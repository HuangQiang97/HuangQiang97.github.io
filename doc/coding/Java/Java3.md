# 多线程

### 多线程基础

1：把一个任务称为一个进程，某些进程内部还需要同时执行多个子任务，把子任务称为线程。一个进程可以包含一个或多个线程，但至少会有一个线程。操作系统调度的最小任务单位其实不是进程，而是线程。多线程编程的特点在于：多线程经常需要读写共享数据，并且需要同步

2：实现多任务的方法，有以下几种：

* 多进程模式（每个进程只有一个线程）

* 多线程模式（一个进程有多个线程）

* 多进程＋多线程模式（复杂度最高）：

3：    多进程与多线程比较：

- 创建进程比创建线程开销大，尤其是在Windows系统上；
- 进程间通信比线程间通信要慢，因为线程间通信就是读写同一个变量，速度很快。
- 多进程稳定性比多线程高，因为在多进程的情况下，一个进程崩溃不会影响其他进程，而在多线程的情况下，任何一个线程崩溃会直接导致整个进程崩溃。

4：Java语言内置了多线程支持：当Java程序启动的时候，实际上是启动了一个JVM进程，然后，JVM启动主线程来执行`main()`方法。在`main()`方法中，我们又可以启动其他线程。

### 创建多线程

1：方法一：从`Thread`派生一个自定义类，然后覆写`run()`方法：

```java
public class Main {
    public static void main(String[] args) {
        Thread t = new MyThread();
        // 启动新线程,start()方法会在内部自动调用实例的run()方法
        // 直接调用run()方法，相当于调用了一个普通的Java方法，当前线程并没有任何改变，也不会启动新线程。上述代码实际上是在main()方法内部又调用了run()方法，打印语句是在main线程中执行的，没有任何新线程被创建。必须调用Thread实例的start()方法才能启动新线程，一个线程对象只能调用一次start()方法。
        // main线程和t线程就开始同时运行了，并且由操作系统调度，程序本身无法确定线程的调度顺序。
        t.start(); 
    }
}

class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("start new thread!");
    }
}

// 简写为
public class Main {
    public static void main(String[] args) {
        Thread t = new Thread(){
            @Override
            public void run() {
                System.out.println("start new thread!");
            }
        };
        t.start(); 
    }
}

```

方法二：创建`Thread`实例时，传入一个`Runnable`实例：

```
public class Main {
    public static void main(String[] args) {
        Thread t = new Thread(new MyRunnable());
        t.start(); // 启动新线程
    }
}

class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("start new thread!");
    }
}

// 简写为：
public class Main {
    public static void main(String[] args) {
        Thread t = new Thread(() -> {
            System.out.println("start new thread!");
        });
        t.start(); // 启动新线程
    }
}

```

###  线程的状态

1：Java线程的状态有以下几种：

- New：新创建的线程，尚未执行；
- Runnable：运行中的线程，正在执行`run()`方法的Java代码；
- Blocked：运行中的线程，因为某些操作被阻塞而挂起，离开执行队列；
- Waiting：运行中的线程，因为某些操作在等待中，仍在执行队列；
- Timed Waiting：运行中的线程，因为执行`sleep()`方法正在计时等待；
- Terminated：线程已终止，因为`run()`方法执行完毕。

当线程启动后，它可以在`Runnable`、`Blocked`、`Waiting`和`Timed Waiting`这几个状态之间切换，直到最后变成`Terminated`状态，线程终止。

2：线程终止的原因有：

- 线程正常终止：`run()`方法执行到`return`语句返回；
- 线程意外终止：`run()`方法因为未捕获的异常导致线程终止；
- 对某个线程的`Thread`实例调用`stop()`方法强制终止（强烈不推荐使用）。

3：当`main`线程对线程对象`t`调用`join()`方法时，主线程将等待变量`t`表示的线程运行结束，即`join`就是指等待该线程结束，然后才继续往下执行自身线程。`join(long)`的重载方法也可以指定一个等待时间，超过等待时间后就不再继续等待。对已经运行结束的线程调用`join()`方法会立刻返回。主线程在等待子线程结束，这个时候主线程是在waiting状态。

### 中断线程

1：方法一：中断一个线程只需要在其他线程中对目标线程调用`interrupt()`方法，`interrupt()`方法仅仅向`t`线程发出了“中断请求”，至于`t`线程是否能立刻响应,要看具体代码。目标线程需要反复检测自身状态是否是interrupted状态``while (! isInterrupted()）``，如果是，就立刻结束运行。标线程检测到`isInterrupted()`为`true`或者捕获了`InterruptedException`都应该立刻结束自身线程。`interrupt()`并不具备阻塞效果，传递完信号后直接继续执行下一句。

如果线程处于正常运行状态，调用``interrupt()``会向目标线程传递被中断信号，目标线程自行处理中断信号，不会出现抛出异常。Interrupt一般用于清理，他在接收到中断信号后自行处理，不强制要求立刻推出，被中断可以选择不退出。

如果线程处于等待状态，例如，`t.join()`会让`main`线程进入等待状态，此时，如果对`main`线程调用`interrupt()`，`join()`方法会立刻结束等待并抛出`InterruptedException`，因此，目标线程只要捕获到`join()`方法抛出的`InterruptedException`，就说明有其他线程对其调用了`interrupt()`方法，如果捕获了该方法的这个异常。依然可以根据代码正常运行。但是通常情况下该线程应该执行资源清理然后立刻结束运行。当前线程被``interrupt()`,并不影响由它创建的子线程的正常运行。

```
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread t = new MyThread();
        t.start();
        t.interrupt(); // 中断t线程
        t.join(); // 等待t线程结束
    }
}

class MyThread extends Thread {
    public void run() {
        Thread hello = new HelloThread();
        hello.start(); // 启动hello线程
        try {
            hello.join(); // 等待hello线程结束，等待状态被`interrupt()`会抛出异常。
        } catch (InterruptedException e) {
        }
        hello.interrupt(); //正常运行状态被`interrupt()`会正常结束，无异常产生。
    }
}

class HelloThread extends Thread {
    public void run() {
        while (!isInterrupted()) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
            	//若线程在在wait,sleep,occupied状态时，调用了它的interrupt()方法，那么它的“中断状态”会被清除，也就是isInterrupted() 会返回false，并且会收到一个InterruptedException异常。如果没有break的话，while会条件会一直返回true，就死循环了
            	// 可以通过Thread.currentThread.interrupt()恢复中断标志位true
            	break; 
            }
        }
    }
}

```



2：方法二：一个常用的中断线程的方法是设置标志位。我们通常会用一个`running`标志位来标识线程是否应该继续运行，在外部线程中，通过把`HelloThread.running`置为`false`，就可以让线程结束：

```
public class Main {
    public static void main(String[] args)  throws InterruptedException {
        HelloThread t = new HelloThread();
        t.start();
        t.running = false; // 标志位置为false
    }
}

class HelloThread extends Thread {
	// 线程间共享变量需要使用volatile关键字标记，确保每个线程都能读取到更新后的变量值。
    // 在Java虚拟机中，变量的值保存在主内存中，但是，当线程访问变量时，它会先获取一个副本，并保存在自己的工作内存中。如果线程修改了变量的值，虚拟机会在某个时刻把修改后的值回写到主内存，但是，这个时间是不确定的！这会导致如果一个线程更新了某个变量，另一个线程读取的值可能还是更新前的,这就造成了多线程之间共享的变量不一致。
    // volatile关键字的目的是告诉虚拟机：每次访问变量时，总是获取主内存的最新值；每次修改变量后，立刻回写到主内存。当一个线程修改了某个共享变量的值，其他线程能够立刻看到修改后的值
    public volatile boolean running = true;
    public void run() {
        while (running) {
        }
    }
}

```

### 守护线程

1： Java程序入口就是由JVM启动`main`线程，`main`线程又可以启动其他线程。当所有线程都运行结束时，JVM退出，进程结束。

如果有一个线程没有退出，JVM进程就不会退出。所以，必须保证所有线程都能及时结束。

2：但是有一种线程的目的就是无限循环，例如，一个定时触发任务的线程，他是为其他线程服务的，当被服务线程结束后，他也应该结束，但由于是死循环无法正常结束，所以可以他把标记为守护线程。守护线程是指为其他线程服务的线程。在JVM中，所有非守护线程都执行完毕后，无论有没有守护线程，虚拟机都会自动退出。JVM退出时，不必关心守护线程是否已结束。

3：使用

方法和普通线程一样，只是在调用`start()`方法前，调用`setDaemon(true)`把该线程标记为守护线程：

```
t.setDaemon(true);
t.start();
```

守护线程不能持有任何需要关闭的资源，例如打开文件等，因为虚拟机退出时，守护线程没有任何机会来关闭文件，这会导致数据丢失。

### 线程同步

1： 多线程模型下，要保证逻辑正确，对共享变量进行读写时，必须保证一组指令以原子方式执行：即某一个线程执行时，其他线程必须等待。通过加锁和解锁的操作，就能保证指令总是在一个线程执行期间，不会有其他线程会进入此指令区间。即使在执行期线程被操作系统中断执行，其他线程也会因为无法获得锁导致无法进入此指令区间。只有执行线程将锁释放后，其他线程才有机会获得锁并执行。这种加锁和解锁之间的代码块我们称之为临界区（Critical Section），任何时候临界区最多只有一个线程能执行。

```ascii
┌───────┐     ┌───────┐
│Thread1  │     │Thread2  │
└───┬───┘     └───┬───┘
    │             │
    │-- lock --   │
    │ILOAD (100)  │
    │IADD         │
    │ISTORE (101) │
    │-- unlock -- │
    │             │-- lock --
    │             │ILOAD (101)
    │             │IADD
    │             │ISTORE (102)
    │             │-- unlock --
    ▼             ▼
```

2：使用`synchronized`关键字对一个对象进行加锁，`synchronized`保证了代码块在任意时刻最多只有一个线程能执行：

```
// 线程在执行各自的synchronized(lock) { ... }代码块时，必须先获得锁，才能进入代码块进行。执行结束后，在synchronized语句块结束会自动释放锁。这样一来，对n变量进行读写就不可能同时进行。
// 缺点是带来了性能下降。因为synchronized代码块无法并发执行。此外，加锁和解锁需要消耗一定的时间，所以，synchronized会降低程序的执行效率。
// 不必担心抛出异常。因为无论是否有异常，都会在synchronized结束处正确释放锁
//  public static final Object lock = new Object();静态字段作为锁。
// synchronized除了加锁外，还具有内存屏障功能，并且强制读取所有共享变量的主内存最新值，退出synchronized时再强制回写主内存（如果有修改），已经包含了volatile的功能，所以n不用加volatile修饰。
synchronized(lock) {
    n = n + 1;
}// 释放锁
```

3：`volatile`和`synchronized`

volatile只保证：读主内存到本地副本；操作本地副本；回写主内存。这3步多个线程可以同时进行，一个线程读取变量后，另一个线程照样可以正常读取，即获取值和回写值时都不会阻塞线程，它只是保证了其他线程能更快的看到修改后的值，只保证线程读取内存的时效的问题，不保证原子性。所以 volatile 不能用于线程同步，只是用于提高程序执行效率。

synchronized除了加锁外，还具有内存屏障功能，并且强制读取所有共享变量的主内存最新值，退出synchronized时再强制回写主内存（如果有修改），已经包含了volatile的功能，所以变量不用额外加volatile修饰。

4：原语

JVM规范定义了几种原子操作：

- 基本类型（`long`和`double`除外）赋值，例如：`int n = m`；
- 引用类型赋值，例如：`List<String> list = anotherList`。

单条原子操作的语句不需要同步，如果是多行赋值语句，就必须保证是同步操作

```
synchronized(this) {
    this.first = first;
    this.last = last;
}
```

### 同步方法

1：如果一个类被设计为允许多线程正确访问，我们就说这个类就是“线程安全”的（thread-safe）

* 一些不变类，例如`String`，`Integer`，`LocalDate`，它们的所有成员变量都是`final`，多线程同时访问时只能读不能写，这些不变类也是线程安全的。

* 类似`Math`这些只提供静态方法，没有成员变量的类，也是线程安全的。

* 大部分类，例如`ArrayList`，都是非线程安全的类，不能在多线程中修改它们。但是，如果所有线程都只读取，不写入，那么`ArrayList`是可以安全地在线程间共享的。

* 没有特殊说明时，一个类默认是非线程安全的。

2：Java程序依靠`synchronized`对线程进行同步，使用`synchronized`的时候，锁住的是哪个对象非常重要。

```
public class Counter {
    private int count = 0;
	public static int lock=Object();
    public void add(int n) {
    	// 常用方法是把synchronized逻辑封装起来，方便调用
        synchronized(this) { // 锁住this即当前实例，这又使得创建多个Counter实例的时候，它们之间互不影响，可以并发执行
            count += n;
        }
    }
    // 等价写法，它表示整个方法都必须用this实例加锁。
    public synchronized void add(int n) { // 锁住this
    	count += n;
	} 
	// static方法，是没有this实例的，锁住的是该类的Class实例,即synchronized(Counter.class) {ops;},一个类只有一个Class实例，所以各个实例无法并发执行。
	public synchronized static void test(int n) {
    	ops;
	}
	// 读取单个变量无需同步，说过是多个就要同步
	public int get() {
        return count;
    }

}

// 各个线程可以并发执行
new Thread(() -> {
     new Counter().add();
}).start();
new Thread(() -> {
     new Counter().add();
}).start();

////////////////////////////////////////
 public class Counter {
    private int count = 0;
	public static int lock=Object();
    public void add(int n) {
    	// 常用方法是把synchronized逻辑封装起来，方便调用
        synchronized(Counter.lock) { // 锁住静态变量，由于静态变量属于类本身，各个实例共享一个静态变量，这使得创建多个Counter实例的时候无法并发执行
            count += n;
        }
    }
}

// 各个线程无法并发执行
new Thread(() -> {
     Counter().add();
}).start();
new Thread(() -> {
     Counter().dec();
}).start();

```

3：同步只保证多线程执行的synchronized块是依次执行，最终状态对不对还取决于你的逻辑。比如要等所有线程操作完毕再去读取的结果才是正确的，如果各个线程正在执行，直接读取结果就会之错误的。

### 死锁

1：Java的线程锁是可重入的锁：JVM允许同一个线程重复获取同一个锁，这种能被同一个线程反复获取的锁，就叫做可重入锁，原因是：不是方法获取锁，是线程获取锁，一个线程获取锁后就可以在线程里面的方法中传递。获取锁的时候，不但要判断是否是第一次获取，还要记录这是第几次获取。每获取一次锁，记录+1，每退出`synchronized`块，记录-1，减到0的时候，才会真正释放锁。

```
public class Counter {
    private int count = 0;

    public synchronized void add(int n) {
        if (n < 0) {
        // 不会柱塞
            dec(-n);
        } else {
            count += n;
        }
    }

    public synchronized void dec(int n) {
        count += n;
    }
}
```

2：两个线程各自持有不同的锁，然后各自试图获取对方手里的锁，造成了双方无限等待下去，这就是死锁。死锁发生后，没有任何机制能解除死锁，只能强制结束JVM进程。解决方法是：线程获取锁的顺序要一致。即严格按照先获取`lockA`，再获取`lockB`的顺序。

```
public void add(int m) {
    synchronized(lockA) { // 获得lockA的锁
        this.value += m;
        synchronized(lockB) { // 获得lockB的锁
            this.another += m;
        } // 释放lockB的锁
    } // 释放lockA的锁
}

// 可能发生死锁
public void dec(int m) {
    synchronized(lockB) { // 获得lockB的锁
        this.another -= m;
        synchronized(lockA) { // 获得lockA的锁
            this.value -= m;
        } // 释放lockA的锁
    } // 释放lockB的锁
}

线程1：进入add()，获得lockA；
线程2：进入dec()，获得lockB。
随后：
线程1：准备获得lockB，失败，等待中；
线程2：准备获得lockA，失败，等待中。

// 不会发生死锁
public void dec(int m) {
    synchronized(lockA) { // 获得lockA的锁
        this.value -= m;
        synchronized(lockB) { // 获得lockB的锁
            this.another -= m;
        } // 释放lockB的锁
    } // 释放lockA的锁
}


```

###  使用wait和notify

1：多线程协调运行的原则就是：当条件不满足时，线程进入等待状态；当条件满足时，线程被唤醒，继续执行任务。

```
class TaskQueue {
    Queue<String> queue = new LinkedList<>();
	// 添加任务
    public synchronized void addTask(String s) {
        this.queue.add(s);
        
        // 对this锁对象调用notify()方法，这个方法会唤醒一个正在this锁等待的线程,然后从wait()方法返回
        // 要在synchronized内部调用
        this.notify(); 
        //使用notifyAll()将唤醒所有当前正在this锁等待的线程，而notify()只会唤醒其中一个（具体哪个依赖操作系统，有一定的随机性）。这是因为可能有多个线程正在wait()中等待，使用notifyAll()将一次性全部唤醒。通常来说，notifyAll()更安全。有些时候，如果我们的代码逻辑考虑不周，用notify()会导致只唤醒了一个线程，而其他线程可能永远等待下去醒不过来了。
        this.notifyAll();

    }
	// 可能死循环
	// 当quene为空，进入死循环，直到添加任务。但是getTask()已经持有this锁，addTask()会被阻塞，所以无法添加任务，无法跳出死循环。
    public synchronized String getTask() {
        while (queue.isEmpty()) {
       
        }
        return queue.remove();
    }
    // 非死循环版本。
    public synchronized String getTask() {
    	// 始终在while循环中wait()，并且每次被唤醒后拿到this锁就必须再次判断，因为可能多个线程竞争一个锁，如果时使用if(queue.isEmpty()){  this.wait();},可能多个线程同时被唤醒，但只有一个线程能获得锁假设为t1，其余线程阻塞，当t1执行完毕释放锁，其余线程再次竞争，但只有一个线程能获取到资源假设为t2，如果使用if，t2获得锁后直接remove(),但这时quene可能为空，出现逻辑错误。所以每次被唤醒后拿到this锁就必须再次判断。
        while (queue.isEmpty()) { 
        // 如果队列为空，线程将执行this.wait()，进入等待状态。
        // wait()方法必须在当前获取的锁对象上调用，这里获取的是this锁，因此调用this.wait()。如果是其他锁：someLock.wait()
        // wait()方法不会返回，直到将来某个时刻，线程从等待状态被其他线程唤醒后，wait()方法才会返回，然后，继续执行下一条语句。
        // 必须在synchronized块中才能调用wait()方法,wait()方法调用时，会释放线程获得的锁，wait()方法返回后，线程又会重新试图获得锁。
            this.wait(); // 释放this锁
            // 重新获取this锁，已唤醒的线程还需要重新获得锁后才能继续执行，当多个线程竞争一个锁后，未获得锁的线程会阻塞，当获得锁的线程释放锁，被阻塞线程再次竞争锁，如此往复。
        }
        return queue.remove();
	}
```

### ReentrantLock

1: `synchronized`关键字用于加锁，但这种锁一是很重，二是获取时必须一直等待，没有额外的尝试机制。`java.util.concurrent.locks`包提供的`ReentrantLock`用于替代`synchronized`加锁

```
public class Counter {
    private final Lock lock = new ReentrantLock();
    private int count;

    public void add(int n) {
        lock.lock();
        // 要考虑异常。
        try {
            count += n;
        } finally {
        // synchronized是Java语言层面提供的语法，所以我们不需要考虑异常，而ReentrantLock是Java代码实现的锁，我们就必须先获取锁,在finally中正确释放锁。
            lock.unlock();
        }
    }
}
```

2:`ReentrantLock`可以尝试获取锁

```
if (lock.tryLock(1, TimeUnit.SECONDS)) {// 尝试获取锁的时候，最多等待1秒。如果1秒后仍未获取到锁，tryLock()返回false，程序自己醒来，然后程序就可以做一些额外处理，而不是无限等待下去。线程在tryLock()失败的时候不会导致死锁。
    try {
        ...
    } finally {
        lock.unlock();
    }
}
```

3：在`ReentrantLock`下使用`Condition`对象来实现`wait`和`notify`的功能。

```
class TaskQueue {
    private final Lock lock = new ReentrantLock();
    // 引用的Condition对象必须从Lock实例的newCondition()返回，这样才能获得一个绑定了Lock实例的Condition实例。
    private final Condition condition = lock.newCondition();
    private Queue<String> queue = new LinkedList<>();

    public void addTask(String s) {
        lock.lock();
        try {
            queue.add(s);
            // 会唤醒某个等待线程
            condition.signal();
            // 会唤醒所有等待线程；
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public String getTask() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                // 会释放当前锁，进入等待状态；
                condition.await();
                // 唤醒线程从await()返回后需要重新获得锁，无法获取就会进入阻塞状态
            }
            return queue.remove();
        } finally {
            lock.unlock();
        }
    }
}

// 释放锁，等待指定时间后，如果还没有被其他线程通过signal()或signalAll()唤醒，可以自己醒来,并返回false;如果时间限定内被唤醒返回true,无论是被唤醒还是自动醒来，都会从等待队列进入同步队列，等待获取lock，在获得lock前会保持阻塞。
if (condition.await(1, TimeUnit.SECOND)) {
    // 被其他线程唤醒
    
} else {
    // 指定时间内没有被其他线程唤醒
}
```

4：一个condition内部维护一个等待队列(不带头结点的链式队列)，一个同步队列（双向队列）。所有调用condition.await方法的线程会程释放lock然后加入到等待队列中，并且线程状态转换为等待状态,直至被signal/signalAll（doSignal方法只会对等待队列的头节点进行操作，而doSignalAll等待队列中的每一个节点都移入到同步队列中，即“通知”当前调用condition.await()方法的每一个线程。）后会使得当前线程从等待队列中移至到同步队列中去，调用condition的signal的前提条件是当前调用signal的线程已经获取了lock，调用condition的signal或者signalAll方法可以将等待队列中等待时间最长的节点移动到同步队列中，按照等待队列是先进先出（FIFO）的，所以等待队列的头节点必然会是等待时间最长的节点，也就是每次调用condition的signal方法是将头节点移动到同步队列中，使得该节点能够有机会获得lock，直到获得了lock后才会从await方法返回，或者在等待时被中断会做中断处理,也即退出await方法必须是已经获得了condition引用（关联）的lock。

可以多次调用lock.newCondition()方法创建多个condition对象，也就是一个lock可以持有多个等待队列和多个同步队列，所以多个同步队列间存在竞争关系。

线程awaitThread先通过lock.lock()方法获取锁成功后调用了condition.await方法进入等待队列，而另一个线程signalThread通过lock.lock()方法获取锁成功后调用了condition.signal或者signalAll方法，使得线程awaitThread能够有机会移入到同步队列中，当其他线程释放lock后使得线程awaitThread能够有机会获取lock，从而使得线程awaitThread能够从await方法中退出执行后续操作。如果awaitThread获取lock失败会重新进入到同步队列的尾部。

### ReadWriteLock

1：`ReadWriteLock`可以解决这个问题，它保证：

- 只允许一个线程写入（其他线程既不能写入也不能读取）；
- 没有写入时，多个线程允许同时读（提高性能）。

把读写操作分别用读锁和写锁来加锁，在读取时，多个线程可以同时获得读锁，这样就大大提高了并发读的执行效率。适用条件是同一个数据，有大量线程读取，但仅有少数线程修改。

```
 private final ReadWriteLock rwlock = new ReentrantReadWriteLock();
 private final Lock rlock = rwlock.readLock();
 private final Lock wlock = rwlock.writeLock();
 public void inc(int index) {
     wlock.lock(); // 加写锁
     try {
         // 对于写入原子操作不加锁是多线程安全的，但是加锁的目的是保证逻辑正确，比如连续写入多个，可能中途被阻断
     	///
     } finally {
     	wlock.unlock(); // 释放写锁
     }
 }

public int[] get() {
    rlock.lock(); // 加读锁
    try {
    	// 允许多个并发读取
    	// 对于一次读取多个变量，如果读的时候如果没有加锁，读到一半可能会有线程写入数据 从而导致读取的数据出错，
    } finally {
    	rlock.unlock(); // 释放读锁
    }
}
```

###  StampedLock

1：对于`ReadWriteLock`如果有线程正在读，写线程需要等待读线程释放锁后才能获取写锁，即读的过程中不允许写，这是一种悲观的读锁。

`StampedLock`和`ReadWriteLock`相比，改进之处在于：读的过程中也允许获取写锁后写入！读的数据就可能不一致，所以，需要一点额外的代码来判断读的过程中是否有写入，这种读锁是一种乐观锁。

2：乐观锁：乐观地估计读的过程中大概率不会有写入，因此被称为乐观锁。

悲观锁：读的过程中拒绝有写入，也就是写入必须等待。

显然乐观锁的并发效率更高，但一旦有小概率的写入导致读取的数据不一致，需要能检测出来，再读一遍就行。

3：写入的加锁是完全一样的，不同的是读取。注意到首先我们通过`tryOptimisticRead()`获取一个乐观读锁，并返回版本号。接着进行读取，读取完成后，我们通过`validate()`去验证版本号，如果在读取过程中没有写入，版本号不变，验证成功，我们就可以放心地继续后续操作。如果在读取过程中有写入，版本号会发生变化，验证将失败。在失败的时候，我们再通过获取悲观读锁再次读取。由于写入的概率不高，程序在绝大部分情况下可以通过乐观读锁获取数据，极少数情况下使用悲观读锁获取数据。

```
	private final StampedLock stampedLock = new StampedLock();

    private double x;
    private double y;

    public void move(double deltaX, double deltaY) {
    	// 写锁互斥，一个线程获取到了，另一个线程就只能在stampedLock.writeLock()等着
        long stamp = stampedLock.writeLock(); // 获取写锁
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            stampedLock.unlockWrite(stamp); // 释放写锁
        }
    }

    public double distanceFromOrigin() {
        long stamp = stampedLock.tryOptimisticRead(); // 获得一个乐观读锁
        // 注意下面两行代码不是原子操作
        // 假设x,y = (100,200)
        double currentX = x;
        // 此处已读取到x=100，但x,y可能被写线程修改为(300,400)
        double currentY = y;
        // 此处已读取到y，如果没有写入，读取是正确的(100,200)
        // 如果有写入，读取是错误的(100,400)
        if (!stampedLock.validate(stamp)) { // 检查乐观读锁后是否有其他写锁发生，
            stamp = stampedLock.readLock(); // 获取一个悲观读锁，保证x,y读值配对。不会读到一部分写入前的数据一部分写入后的数据
            try {
                currentX = x;
                currentY = y;
            } finally {
                stampedLock.unlockRead(stamp); // 释放悲观读锁
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }
```

4：把读锁细分为乐观读和悲观读，能进一步提升并发效率。但这也是有代价的：一是代码更加复杂，二是`StampedLock`是不可重入锁，不能在一个线程中反复获取同一个锁。

对于悲观锁：读写不能同时进行，一般来说写锁的优先级要高于读锁，read-write-lock假定读很多几乎不会间断，假设现在存在大量读取，如果突然来个写锁，那么只需等当前正在读的释放读锁后，写就立刻获得写锁，其它后续读都得等，不然在一直都有读的情况下，永远写不了。

对于乐观锁：读写可以同时进行。乐观锁其实不上锁，只检查版本号，也不必释放，它的目的是把read-write-lock的read加读锁这一步给去了，因为绝大多数情况下没有写，不需要加读锁，当发现读的时候发生了写，数据前后版本不一致，获取悲观锁，保证读取数据逻辑正确性。

### concurrent

1：对`List`、`Map`、`Set`、`Deque`等，`java.util.concurrent`包也提供了并发集合类

| interface | non-thread-safe         | thread-safe                              |
| :-------- | :---------------------- | :--------------------------------------- |
| List      | ArrayList               | CopyOnWriteArrayList                     |
| Map       | HashMap                 | ConcurrentHashMap                        |
| Set       | HashSet / TreeSet       | CopyOnWriteArraySet                      |
| Queue     | ArrayDeque / LinkedList | ArrayBlockingQueue / LinkedBlockingQueue |
| Deque     | ArrayDeque / LinkedList | LinkedBlockingDeque                      |

使用这些并发集合与使用非线程安全的集合类完全相同，所有的同步和加锁的逻辑都在集合内部实现，对外部调用者来说，只需要正常按接口引用，其他代码和原来的非线程安全代码完全一样

2：把一个旧的非安全集合转换为线程安全集合

```
Map unsafeMap = new HashMap();
Map threadSafeMap = Collections.synchronizedMap(unsafeMap);
```

但是它实际上是用一个包装类包装了非线程安全的`Map`，然后对所有读写方法都用`synchronized`加锁，这样获得的线程安全集合的性能比`java.util.concurrent`集合要低很多，所以不推荐使用。

### Atomic

1：一组原子操作的封装类，如`AtomicInteger`，原子操作实现了无锁的线程安全；适用于计数器，累加器等。子操作可以看做最小的执行单位，该操作在执行完毕前不会被任何其他任务或事件打断。最底层基于汇编语言实现，总体性能比Synchronized高很多。

2：Atomic类是通过无锁（lock-free）的方式实现的线程安全（thread-safe）访问。它的主要原理是利用了CAS：Compare and Set。CAS是指，在这个操作中，如果`AtomicInteger`的当前值是`prev`，那么就更新为`next`，返回`true`。如果`AtomicInteger`的当前值不是`prev`，就什么也不干，返回`false`。通过CAS操作并配合`do ... while`循环，即使其他线程修改了`AtomicInteger`的值，最终的结果也是正确的。

```
public int incrementAndGet(AtomicInteger var) {
    int prev, next;
    do {
        prev = var.get();
        next = prev + 1;
    } while ( ! var.compareAndSet(prev, next)); //如果var的值不是prev证明var之后被修改过放弃加一。
    return next;
}
```