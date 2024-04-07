### DelayQueue实现

##### 1，初始化

```java
private final transient ReentrantLock lock = new ReentrantLock();
private final PriorityQueue<E> q = new PriorityQueue<E>();
private Thread leader;
private final Condition available = lock.newCondition();
```



* ` lock = new ReentrantLock()`保证多线程操作队列的安全性。

* `q = new PriorityQueue<E>()`按照到期时间从小到大对元素排序。

* `Thread leader`等待对队首元素操作的leader线程，该线程将被定时休眠后唤醒，最先获取值返回。

* `available = lock.newCondition()`与Leader/Followers模式配合，Followers在此等待，当leader离任时某个Follower被唤醒成为新的leader，非公平获取，线程获取元素的返回顺序和调用顺序不一定一致。。

  保证只有一个leader线程会被定时唤醒，其余Followers线程在`available `上无限休眠，减少不必要的竞争。

* 非公平获取，线程获取元素的返回顺序和调用顺序不一定一致。



##### 2，加入队列

```java
public boolean offer(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        q.offer(e);
        if (q.peek() == e) {
            leader = null;
            available.signal();
        }
        return true;
    } finally {
        lock.unlock();
    }
}
```



* `available.signal()`：如果`e`加入队列后自己为队首元素，证明`e`的到期时间早于向前队首元素$\hat{e}$，而`leader`线程的休眠时间设定为$\hat{e}$的到期时间，所以需要提前唤醒等待中的线程(`leader`和`flowers`线程)，让他们尝试获取`e`或者将自己的休眠时间更新为`e`的到期，保证`e`被及时消费。

  

  举例：->当前队首元素$\hat{e}$过期时间10s，leader线程休眠10s

  ​	  ->1s后过期时间5s的新元素`e`加入队列，成为队首元素
  ​          ->如果不手动唤醒线程，元素`e`只能在9s被主动唤醒的`leader`线程获取，此时`e`已经过期4s

  ​	  ->如果手动唤醒等待线程，被唤醒的线程更新自己的定时休眠时间为5s，`e`将能在5s后被及时消费。

  

* `leader = null`：`leader`是对队首元素操作的线程，处于定时休眠状态，具有更高的获取数据优先级。当`leader`和`flower`线程休眠时，由于二者处于同一个等待队列和柱塞队列，无法保证`available.signal()`唤醒的是`leader`线程，如果被唤醒的是`flower`线程且队首元素`e`未到期，该线程发现存在`leader`线程后将进入永久休眠（`take`方法的第15行），之后只能等待`leader`线程休眠时间到后主动唤醒获取队首元素，但由于`leader`线程休眠时间是按照$\hat{e}$设定的，长于`e`的过期时间，将导致`e`不能及时被消费。

  

  举例：->前队首元素$\hat{e}$过期时间10s，leader线程休眠10s

  ​	   ->1s后过期时间5s的新元素`e`加入队列，成为队首元素，唤醒一个等待线程

  ​	   ->如果不将leader标识置空，且被唤醒的线程是之前的flower线程，唤醒线程将再次进入永久休眠，元素`e`只		能在9s被主动唤醒的`leader`线程获取，此时`e`已经过期4s

     	->如果将leader标识置空，被唤醒的线程将成为新的leader线程，并且更新自己的定时休眠时间为5s，`e`将能		在5s后被及时消费。

  

* 公平性：由于手动调用了`available.signal()`，被唤醒线程不一定是`leader`线程，所以是非公平，方法返回顺序不一定和调用顺序一致，

  

##### 3，获取元素

```java
 // 柱塞方式获得元素
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            E first = q.peek();
            if (first == null)
                available.await();
            else {
                long delay = first.getDelay(NANOSECONDS);
                if (delay <= 0L)
                    return q.poll();
                first = null; // don't retain ref while waiting
                if (leader != null)
                    available.await();
                else {
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        available.awaitNanos(delay);
                    } finally {
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        if (leader == null && q.peek() != null)
            available.signal();
        lock.unlock();
    }
}

```



* `leader = null`：24行leader线程定时唤醒后即完成一届任期，如果自己还是leader将自动卸任进入下一轮leader的竞选，以便其他线程有机会成为新的领导线程并被唤醒。
* 自动连任`leader`，修改进入无限期休眠条件为当前`leaer`不是自己、定时唤醒后不卸任自动连任、返回前再卸任：

```java
public E take() throws InterruptedException {
    //...
    try {
        //...
		// 修改进入无限期休眠条件
        if (leader != null&&leader != Thread.currentThread())
            available.await();
        else {
            Thread thisThread = Thread.currentThread();
            leader = thisThread;
            try {
                available.awaitNanos(delay);
            } finally {
                // 不卸任leader，自动连任
                // if (leader == thisThread)
                //     leader = null;
            }
        }
    } finally {
        // 返回前卸任leader
        leader=null;
        if (q.peek() != null)
            available.signal();
        lock.unlock();
    }
}
```
