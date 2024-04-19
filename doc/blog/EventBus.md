# Guava EvenBus源码解读

[toc]

### 发布订阅模型

* 通过发布订阅中心建立事件发布者和事件订阅者间的通信体系。发布者和订阅者不直接通信，而是将要发布的消息交     由发布订阅中心管理，订阅者按需订阅中心中的消息，订阅者将在事件发生时被事件处理中心主动唤醒。

* 发布者的发布动作和订阅者的订阅动作相互独立，消息派发由发布订阅中心负责，生产者、订阅者完全解耦的。

<img src=".\assets\image-20240410205859440.png" alt="image-20240410205859440" style="zoom: 45%;" />

### Guava EventBus概述

* `EventBus`为轻量级、进程内事件驱动组件，提供同步以及异步方式的事件驱动机制。

    

    <img src=".\assets\v2-60d8d22597f8ccfbec74773ddf1bfc23_r.jpg" alt="img" style="zoom:70%;" />

* 组成：`EventBus` 、`AsyncEventBus`提供同步和异步的事件处理机制；`Event`、`DeadEvent`事件和无订阅者的事件；`Subscriber`、`SynchronizedSubscriber`根据`handler`方法生成的线程不安全订阅者和线程安全订阅者；`perThreadDispatcher, legacyAsyncDispatcher`线程独立派发器和线程共享派发器，用于派发任务给`Subscriber`；`SubscriberRegistry`订阅者注册器，用户根据`Listener`生成`Subscriber`并注入到容器中。

    

* 工作流程：定义事件订阅者`Listener`，包含事件发生时要执行的操作`handler`。将`Listener`注册到`EventBus`时，将每个`handler`封装成`Subscriber`对象，加入所等待事件的集合`Subscribers`中。当某事件发生时，等待在该事件集合`Subscribers`中的`Subscriber`对象将被调用，完成事件的处理。

<img src=".\assets\v2-8bd35d1da57ca7650a34fd8ab7852c6d_r.jpg" alt="img" style="zoom:50%;" />

### 事件注册



<img src=".\assets\v2-be424160e247db0099d3d4c1005a758e_r.jpg" alt="img" style="zoom:50%;" />

##### `register`

* 注册方法`register`：构建缓存`<key=event.class, value=[Subscriber(handler)]>`，将==listener及其父类==定义的`handler`封装成`Subscriber`对象，加入所等待事件的集合`Subscribers`中。

  ```java
  void register(Object listener) {
      // 将listener及其父类定义的全部handler方法包换成Subscriber类型，
      // 并以Map形式返回,key为等待的事件类型，value为等待该事件的多个hadler构成的Subscriber set集合
      // <key=envnt.class, value=[Subscriber(handler)]>
      Multimap<Class<?>, Subscriber> listenerMethods = findAllSubscribers(listener);
  
      for (Entry<Class<?>, Collection<Subscriber>> entry : listenerMethods.asMap().entrySet()) {
        Class<?> eventType = entry.getKey();
        Collection<Subscriber> eventMethodsInListener = entry.getValue();
  
        // 将新加入的Subscriber添加到所等待事件对应的Subscribers集合中
        // CopyOnWrite集合通过读写分离，写操作同步保证并发写入安全
        CopyOnWriteArraySet<Subscriber> eventSubscribers = subscribers.get(eventType);
        // 该事件未被注册过，创建新的Subscribers集合
        if (eventSubscribers == null) {
          CopyOnWriteArraySet<Subscriber> newSet = new CopyOnWriteArraySet<>();
          // putIfAbsent: 不存在才添加; firstNonNull:获得参数列表中第一个非null值
          // 双重检查防止两个线程同时发现event未注册，一个线程完成注册后，另一个线程用新的set覆盖，导致第一个线程注册Subscriber丢失
          eventSubscribers =
              MoreObjects.firstNonNull(subscribers.putIfAbsent(eventType, newSet), newSet);
        }
  
        eventSubscribers.addAll(eventMethodsInListener);
      }
  }
  ```



##### `findAllSubscribers`

* 获取注册`listener`及其父类定义的所有`handler`方法，并以`<key=event.class, value=[Subscriber(handler)]>`形式返回。

  ```java
  private Multimap<Class<?>, Subscriber> findAllSubscribers(Object listener) {
      Multimap<Class<?>, Subscriber> methodsInListener = HashMultimap.create();
      Class<?> clazz = listener.getClass();
      // 将listener及其父类的全部handler方法构建<envent.class,[Subscriber(handler)]>
      for (Method method : getAnnotatedMethods(clazz)) {
        Class<?>[] parameterTypes = method.getParameterTypes();
        Class<?> eventType = parameterTypes[0];
        methodsInListener.put(eventType, Subscriber.create(bus, listener, method));
      }
      return methodsInListener;
  }
  ```



##### `getAnnotatedMethodsNotCached`

* `getAnnotatedMethods`方法将从`<listener.class,[Method]>`缓存`subscriberMethodsCache`获取当前`listener`对象及其父类所有标注有`@Subscribe`的`handler`方法。

  ```java
  private static ImmutableList<Method> getAnnotatedMethods(Class<?> clazz) {
      try {
        return subscriberMethodsCache.getUnchecked(clazz);
      } catch (UncheckedExecutionException e) {
        throwIfUnchecked(e.getCause());
        throw e;
      }
  }
  private static final LoadingCache<Class<?>, ImmutableList<Method>> subscriberMethodsCache =
    CacheBuilder.newBuilder()
        .weakKeys()
        .build(
            new CacheLoader<Class<?>, ImmutableList<Method>>() {
              @Override
              public ImmutableList<Method> load(Class<?> concreteClass) throws Exception {
                return getAnnotatedMethodsNotCached(concreteClass);
              }
            });
  ```
  
  
  
  `subscriberMethodsCache`声明为`static`变量，作为缓存只会在类加载时被加载一次，在类加载时会触发初始化方法`getAnnotatedMethodsNotCached`
  
  ```java
  private static ImmutableList<Method> getAnnotatedMethodsNotCached(Class<?> clazz) {
      //  获得listener自身的class、以及所有父类黑和接口
      //  返回值中子类在父类之前，只有子类的hanlder方法被注册，保障事件触发时，调用的是子类方法，保证多态特性
      Set<? extends Class<?>> supertypes = TypeToken.of(clazz).getTypes().rawTypes();
      Map<MethodIdentifier, Method> identifiers = Maps.newHashMap();
      for (Class<?> supertype : supertypes) {
        // 自身方法，不包含继承方法
        for (Method method : supertype.getDeclaredMethods()) {
          // 查找有`@Subscribe`标注的方法，并加入<method>
          if (method.isAnnotationPresent(Subscribe.class) && !method.isSynthetic()) {
            MethodIdentifier ident = new MethodIdentifier(method);
            // 保证被重写方法只加载一次，MethodIdentifier使用(methodName,parmList)作为唯一标识
            // 重写方法的MethodIdentifier对象相等，保证只有子类被重写方法被注册，维持多态特性
            if (!identifiers.containsKey(ident)) {
              identifiers.put(ident, method);
            }
          }
        }
      }
      return ImmutableList.copyOf(identifiers.values());
  }
  ```



##### `Subscriber`

* `Subscriber`创建时将根据`handler`方法是否有`@AllowConcurrentEvents`，创建为普通`Subscriber`或者线程安全的`SynchronizedSubscriber`，二者为继承关系，唯一区别是`SynchronizedSubscriber`执行时会加锁，保障线程安全。二者在线程池中通过反射方式执行持有的`listener`的`handler`方法。

  ```java
  class Subscriber {
      private final Executor executor;
      
      private Subscriber(EventBus bus, Object target, Method method) {
        this.bus = bus;
        this.target = checkNotNull(target);
        this.method = method;
        method.setAccessible(true);
        this.executor = bus.executor();
      }
      
      final void dispatchEvent(Object event) {
          // 在线程池中通过反射方式执行持有的`listener`的`handler`方法。
          executor.execute(
                  () -> {
                      try {
                          invokeSubscriberMethod(event);
                      } catch (InvocationTargetException e) {
                          bus.handleSubscriberException(e.getCause(), context(event));
                      }
                  });
      }
      void invokeSubscriberMethod(Object event) throws InvocationTargetException {
        try {
          method.invoke(target, checkNotNull(event));
        } 
        // catch exception
      }
  
      static final class SynchronizedSubscriber extends Subscriber {
          @Override
          void invokeSubscriberMethod(Object event) throws InvocationTargetException {
              // 加锁保证线程安全
              synchronized (this) {
                  super.invokeSubscriberMethod(event);
              }
          }
      }
  }
  ```
  



### 事件分发

<img src=".\assets\v2-66cd1894a173203d1823f7bb619df38a_r.jpg" alt="img" style="zoom:50%;" />

##### `post`

* 当向`EventBus`投递事件时，将找到`<key=event.class, value=[Subscriber(handler)]>`缓存中，用于处理==对应事件及父类事件的Subscriber==，进行事件处理。

  ```java
  public void post(Object event) {
      // 对应事件及父类事件的`Subscriber`
      Iterator<Subscriber> eventSubscribers = subscribers.getSubscribers(event);
      if (eventSubscribers.hasNext()) {
          // 分发任务，完成对应Subscriber调用
        dispatcher.dispatch(event, eventSubscribers);
      } else if (!(event instanceof DeadEvent)) {
        // 无对应Subscriber的事件已DeadEvent重新发布
        // 可以为DeadEvent创建Subscriber兜底处理
        post(new DeadEvent(this, event));
      }
  }
  ```

##### `getSubscribers`

* `getSubscribers`的获取==不仅仅是对应事件的所有Subscriber，还包括当前事件所有父类的Subscriber==，保证事件层级关系能正确处理。

  ```java
  Iterator<Subscriber> getSubscribers(Object event) {
      // flattenHierarchy方法将返回当前事件类型及其父类类型，且当前类型在前，父类在后
      ImmutableSet<Class<?>> eventTypes = flattenHierarchy(event.getClass());
  
      List<Iterator<Subscriber>> subscriberIterators =
          Lists.newArrayListWithCapacity(eventTypes.size());
  	// 获得当前事件和父类事件的全部订阅者
      for (Class<?> eventType : eventTypes) {
        CopyOnWriteArraySet<Subscriber> eventSubscribers = subscribers.get(eventType);
        if (eventSubscribers != null) {
          // eager no-copy snapshot
          subscriberIterators.add(eventSubscribers.iterator());
        }
      }
  
      return Iterators.concat(subscriberIterators.iterator());
  }
  ```

##### `flattenHierarchyCache`

* `flattenHierarchy`方法会到`<key=event.class, value=[event.class, superClass.class]>`缓存`flattenHierarchyCache`中找到当前类及父类类型，因为是静态变量，所以在代码加载的时候就会缓存`Event`所有父类。

  ```java
  static ImmutableSet<Class<?>> flattenHierarchy(Class<?> concreteClass) {
      try {
        return flattenHierarchyCache.getUnchecked(concreteClass);
      } catch (UncheckedExecutionException e) {
        throw Throwables.propagate(e.getCause());
      }
  } 
  private static final LoadingCache<Class<?>, ImmutableSet<Class<?>>> flattenHierarchyCache =
        CacheBuilder.newBuilder()
            .weakKeys()
            .build(
                new CacheLoader<Class<?>, ImmutableSet<Class<?>>>() {
                  // <Class<?>> is actually needed to compile
                  @SuppressWarnings("RedundantTypeArguments")
                  @Override
                  public ImmutableSet<Class<?>> load(Class<?> concreteClass) {
                    return ImmutableSet.<Class<?>>copyOf(
                        // 获得concreteClass及其所有父类，子类在前，父类在后
                        TypeToken.of(concreteClass).getTypes().rawTypes());
                  }
                });
  
  ```



### 事件执行

`SubscriberRegistry().post`方法的事件分发器`dispatch`常见的类型可以分为两类：`PerThreadQueuedDispatcher, LegacyAsyncDispatcher`



##### `perThreadDispatch`

* `EventBus`默认分发器，使用`DirectExecutor`线程池，由提交任务的线程去执行任务，属于同步模型，线程间独立分发，线程内`BFS`有序处理。

* 线程间：为每一个调用`SubscriberRegistry().post()`方法的线程使用一个线程独立的队列`queueForThread`，缓存各自待分发的`Subscriber`，线程独立分发调度事件，防止多线程竞争以及容器的并发读写。

* 线程内：使用线程独立`dispatching`变量标记当前线程是否已经开始分发任务，==BFS方式处理事件==，只会在最顶层一处处理`handler`调用。同时线程内`BFS`方式嵌套事件处理，严格的`FIFO`==保证线程内事件执行顺序和发布顺序一致，以及同一事件的多个Subscriber的处理顺序和注册顺序一致==。

* 处理流程：

   -->当前线程分发事件`A`，`A`的`handler`加入队列，设置``dispatching=true``，进入`A`的`handler`处理，

   --> 使用`DirectExecutor`线程池，将由当前线程执行`handler`，假设`handler`中又分发事件`B`，
  
   -->  线程内的`dispatch()`再次被调用于`B`事件分发，此时`dispatching=true`，事件`B`的`handler`将只是加入队列，不会继续`handler`处理，
  
   -->  `dispatch()`方法返回，继续在最顶层处理队列中`handler`调用，
  
   -->  防止`DFS`方式不断调度运行`handler`，造成`SOF`，
  
   -->  `BFS`方式嵌套事件处理，处理线程在处理完事件`A`的所有`handler`后再处理`B`事件，以及同一事件的多个Subscriber的处理顺序和注册顺序一致。

```java
/** 存储当前线程待分发的事件，有界队列，最大值为int的最大值 */
private final ThreadLocal<Queue<Event>> queue = new ThreadLocal<Queue<Event>>() {
  @Override
  protected Queue<Event> initialValue() {
    return Queues.newArrayDeque();
  }
};
/** 标志位标记当前线程是否已经开始分发任务，BFS处理事件，防止不断递归handler造成SOF */
private final ThreadLocal<Boolean> dispatching = new ThreadLocal<Boolean>() {
  @Override
  protected Boolean initialValue() {
    return false;
  }
};
// 分发事件，执行hander处理逻辑
void dispatch(Object event, Iterator<Subscriber> subscribers) {
  checkNotNull(event);
  checkNotNull(subscribers);
  // 当Dispatcher被多线程调用时，如果不设置线程独立队列，将导致队列的并发读写竞争和读写错误
  // 独立线程发布事件，其调度独立，不用等待其他线程的事件处理完成
  Queue<Event> queueForThread = requireNonNull(queue.get());
  queueForThread.offer(new Event(event, subscribers));

  // dispatching用于表示当前线程已经开始分发任务，防止递归分发导致SOF
  // 如果事件A的handler导致了事件B的发布，B在dispatch()中在将B对应Subscriber加入队列后即返回，继续完成已有队列中handle调度
  // BFS方式嵌套事件处理：事件A的handler导致事件B被发布，处理线程在处理完事件A的所有handler后再处理B事件。
  if (!dispatching.get()) {
    dispatching.set(true);
    try {
      Event nextEvent;
      // 初始队列只有一个元素，但是Subscriber的处理可能导致新的事件被分发，队列元素增加，需要循环处理
      while ((nextEvent = queueForThread.poll()) != null) {
        while (nextEvent.subscribers.hasNext()) {
          // 分发当前事件对应的全部Subscriber
          nextEvent.subscribers.next().dispatchEvent(nextEvent.event);
        }
      }
    } finally {
      // 当前线程的全部任务分发完成，清空缓存队列
      dispatching.remove();
      queue.remove();
    }
  }
}
```



##### `LegacyAsyncDispatcher`

* `AsyncEventBus`默认分发器，强制自定义线程池，一般是异步执行任务（如果使用`DirectExecutor`线程池则是同步），多线程共享等待队列，==DFS处理，事件间和事件内无执行顺序保证==。

* 线程间：多个线程通过同一个`Dispatcher`分发的事件将位于同一个队列，但是`ConcurrentLinkedQueue`在高并发环境下，事件整体的出队顺序可能与它们发布顺序不同(非阻塞算法、并发修改、内存一致性效应、线程调度和执行延迟)，==事件处理顺序和发布顺序不一定一致==。

* 线程内：异步处理事件订阅，采用`DFS`方式处理队列事件，如果事件A某个的`handler`触发了事件`B`的发布，线程先处理事件`B`所有的`handler`后，再返回继续处理事件`A`的其它`handler`。同样因为`ConcurrentLinkedQueue`在高并发环境下，出队顺序可能与它们加入队列的顺序不同，==同一个事件的多个handler的执行顺序不一定和注册顺序一致==。

    ```java
    private static final class LegacyAsyncDispatcher extends Dispatcher {
    
        /** 并发安全的队列实例，用于存储多个线程post的事件和对应的订阅者，非严格FIFO */
        private final ConcurrentLinkedQueue<EventWithSubscriber> queue = Queues.newConcurrentLinkedQueue();
    
        @Override
        void dispatch(Object event, Iterator<Subscriber> subscribers) {
          checkNotNull(event);
          // DFS方式处理handler
          // ConcurrentLinkedQueue在高并发环境下,出队顺序可能与入队列的顺序不同，非严格FIFO
          while (subscribers.hasNext()) {
            queue.add(new EventWithSubscriber(event, subscribers.next()));
          }
    
          EventWithSubscriber e;
          while ((e = queue.poll()) != null) {
            e.subscriber.dispatchEvent(e.event);
          }
    }
    ```
    



##### 订阅者执行顺序

* `EventBus`：同一个事件的多个订阅者，谁先注册到`EventBus`谁先执行。如果是在同一个`Listener`中的多个订阅者一起被注册，收到事件的顺序跟方法名有关。

* `AsyncEventBus`同一个事件的多个订阅者，它们的注册顺序跟接收到事件的顺序上没有任何联系，在新的线程中，异步并发的执行自己的任务。

    

### 线程执行

##### 线程池

* 在根据方法创建订阅者时，根据方法是否标注为`@AllowConcurrentEvents`，创建非线程安全的`Subscriber`或者线程安全的`SynchronizedSubscriber`。
* 无论是`Subscriber`还是`SynchronizedSubscriber`他们的`dispatchEvent()`都是将任务提交到线程池执行，通过反射执行方法，而他们的线程池参数来自`EventBus`和`AsyncEventBus`。
* `EventBus`的`Executor`参数默认是`DirectExecutor`，该线程池的作用是让提交任务的线程去执行任务，==同步执行==，`Dispatcher`参数默认为`PerThreadQueuedDispatcher`，线程间事件分发独立。
* `AsyncEventBus`的`Executor`强制用户传入，由用户决定线程池类型，==一般是异步执行==，任务提交线程池后就返回，且`Dispatcher`参数必须为`LegacyAsyncDispatcher`类型，线程间共享等待队列。

```java
public class EventBus {
    // 默认是`DirectExecutor`
    private final Executor executor;
    // 默认是`PerThreadQueuedDispatcher`
    private final Dispatcher dispatcher;
}

public class AsyncEventBus extends EventBus {
    // `Executor`强制用户传入，`Dispatcher`参数默认为`LegacyAsyncDispatcher`类型。
  public AsyncEventBus(Executor executor) {
    super("default", executor, Dispatcher.legacyAsync(), LoggingHandler.INSTANCE);
  }
}

class Subscriber {
    // EventBus.executor
    private final Executor executor;
  	private final Method method;
    final Object target;
    
    final void dispatchEvent(Object event) {
        // 将任务提交到线程池执行
        executor.execute(
                () -> {
                    try {
                        invokeSubscriberMethod(event);
                    } catch (InvocationTargetException e) {
                        bus.handleSubscriberException(e.getCause(), context(event));
                    }
                });
    }
    void invokeSubscriberMethod(Object event) throws InvocationTargetException {
       //通过反射执行handler方法中的逻辑
      try {
        method.invoke(target, checkNotNull(event));
      } 
      // catch exception
    }

    static final class SynchronizedSubscriber extends Subscriber {
        @Override
        void invokeSubscriberMethod(Object event) throws InvocationTargetException {
            // // 加锁保证线程安全
            synchronized (this) {
                super.invokeSubscriberMethod(event);
            }
        }
    }
}
```

##### 并发模型

* 整体的并发模型由：`EventBus` 、`AsyncEventBus`；`Executor`；`Subscriber`、`SynchronizedSubscriber`；`PerThreadQueuedDispatcher`、`LegacyAsyncDispatcher`共同决定。
* 事件总线：` EventBus`默认使用`DirectExecutor`，`Subscriber`将在发布事件的同一个线程中被调用，同步执行。
    `AsyncEventBus`使用自定义线程池来处理事件，允许事件处理异步进行。
* 线程模型：`DirectExecutor`为单线程处理，即使使用 `AsyncEventBus`，也不能实现并行处理。使用多线程线程池时，`EventBus `和`AsyncEventBus` 可以利用多个线程进行事件处理。
* 订阅者类型：根据方法是否标注为`@AllowConcurrentEvents`，创建`Subscriber`或者`SynchronizedSubscriber`。二者的区别在于`SynchronizedSubscriber`通过加锁保证`handler`并发安全。
* 事件分发方式：`PerThreadQueuedDispatcher`线程间事件分发独立，`BFS`方式执行；`LegacyAsyncDispatcher`线程间共享事件分发等待队列，`DFS`方式执行
* ` EventBus`强制绑定`DirectExecutor`和`PerThreadQueuedDispatcher`；`AsyncEventBus`强制绑定`LegacyAsyncDispatcher`。上述三个因素各自独立，事件总线、线程模型、订阅者类型三者共同影响并发模型。

### 优缺点

* 简单易上手。
* 局限于进程内使用，无法实现跨进程处理。
* 基于内存，且没有任何持久化机制。
* 不支持设置同一消息的订阅者消费顺序，默认按照注册顺序执行。
* 不支持消息过滤。

### Spring Event对比

##### 工作流程

​	--> 定义事件`ApplicationEvent`及对应的`ApplicationListener`；

​	--> 当事件发生时，通过`ApplicationEventPublisher`接口实现类的`publistEvent`方法发布事件；

​	--> `publistEvent`方法通过`ApplicationEventMulticaster`接口实现类的`multicastEvent`方法广播事件；

​	-->`multicastEvent`方法找到事件的全部`Listener`，并通过`invokeListener`执行他们定义的事件处理逻辑；

​	-->`invokeListener`方法将通过`ApplicationListenerMethodAdapter`的`onApplicationEvent`执行定义的事件处理逻辑，这里的`ApplicationListenerMethodAdapter`是`handler`方法的适配器，负责统一`handler`方法的调用；

​	-->最后`onApplicationEvent`将通过`processEvent`方法使用反射调用，真正执行`handler`逻辑。



##### 过滤特性

* `Spring Event`支持在`handler`定义时指定过滤条件，当条件为真时才会执行具体的`handler`逻辑。

    ```java
    @NoArgsConstructor
    @AllArgsConstructor
    @Data
    public class CustomerEvent {
        private String name;
    }
    
    @Component("springListener")
    public class SpringListener {
        
        /**
         * 监听 CustomerEvent 类型事件，但是需要满足condition条件
         */
        @EventListener(condition = "#event.getName().equals('xxx')")
        public void processEvent(CustomerEvent event) {
            System.out.println("process  CustomerEvent, name:" + event.getName());
        }
    }
    
    ```

    

* 底层实现上使用`ApplicationListenerMethodAdapter`作为`handler`方法的适配器，当事件被触发时，先判断条件是否满足，之后再执行方法。

    ```java
    public class ApplicationListenerMethodAdapter {
    	private final Method method; // processEvent
    	@Nullable
    	private final String condition; // #event.getName().equals('xxx')
        
    	public void processEvent(ApplicationEvent event) {
            Object[] args = resolveArguments(event);
            // condition是否成立判断
            if (shouldHandle(event, args)) {
                Object result = doInvoke(args);
                if (result != null) {
                    handleResult(result);
                }
                else {
                    logger.trace("No result object given - no result to handle");
                }
            }
    }
    ```

    

##### 顺序特性

* `Spring Event`支持在某事件发生时，为他的全部`handler`指定执行顺序，当时间触发时多个`handler`将按照指定顺序执行。

    ```java
    @Component
    public class EventListenerBean {
        @EventListener
        @Order(1)
        public void handleFirst(CustomEvent event) {
            System.out.println("first");
        }
    
        @EventListener
        @Order(2)
        public void handleSecond(CustomEvent event) {
            System.out.println("second");
        }
    }
    
    ```

    

* 其底层实现是在事件发生时，在`ApplicationEventMulticaster`实现类的`multicastEvent`方法中找到事件对应的全部`Listener`后，对他们按照指定的顺序排序，再执行`invokeListener`调用

    ```java
    public class SimpleApplicationEventMulticaster{
        // 广播事件，获取该事件的全部Listener并逐个调用
        public void multicastEvent(ApplicationEvent event, @Nullable ResolvableType eventType) {
        // do something
        for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
            if (executor != null && listener.supportsAsyncExecution()) {
                try {
                    executor.execute(() -> invokeListener(listener, event));
                }
                catch (RejectedExecutionException ex) {
                    // Probably on shutdown -> invoke listener locally instead
                    invokeListener(listener, event);
                }
            }
            else {
                invokeListener(listener, event);
            }
        }
    }
        
    public abstract class AbstractApplicationEventMulticaster{
        // 获得事件的全部Listener
        protected Collection<ApplicationListener<?>> getApplicationListeners(
            ApplicationEvent event, ResolvableType eventType) {
            // do something
            return retrieveApplicationListeners(eventType, sourceType, newRetriever);
        }
        
        // 从注册的所有监听器中找出那些对给定事件感兴趣的监听器
        private Collection<ApplicationListener<?>> retrieveApplicationListeners(
    			ResolvableType eventType, @Nullable Class<?> sourceType, @Nullable CachedListenerRetriever retriever) {
            // do something
            // 按照@Order指定顺序排序
            AnnotationAwareOrderComparator.sort(allListeners);
            return allListeners;
    }
    
    ```

    

### 参考

* [Guava EventBus](https://github.com/google/guava/blob/master/guava/src/com/google/common/eventbus)
* [一文读懂Guava EventBus（订阅\发布事件）](https://zhuanlan.zhihu.com/p/606516662)
* [Guava EventBus的具体使用以及源码解析](https://www.cnblogs.com/knqiufan/p/17479060.html)
* [guava eventbus 原理+源码分析](https://www.cnblogs.com/tele-share/p/14258352.html)
* [EventBus VS Spring Event](https://www.cnblogs.com/shoren/p/eventBus_springEvent.html)
* [业务解耦工具：Spring Event用法和源码分析](https://zhuanlan.zhihu.com/p/613266415)
