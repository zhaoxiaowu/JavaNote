# 第一章 走入并行世界

## 几个概念

**1、同步（Synchronous）和异步（Asynchronous）**

通常用来形容一次方法调用。同步方法调用一旦开始，必须等到方法调用返回后，才能继续后续的行为。异步方法调用更像是一个消息传递，一旦开始，方法调用就会立即返回，调用者可以继续后续的操作，异步方法通常会在另外一个线程中“真实”的执行。

**2、并发（Concurrency）和并行（Parallelism）**

表示多个任务一起执行。并发偏重于多个任务**交替**执行，而多个任务之间可能还是串行的，只是对于外部观察者来说，看起来像是同时执行的。并行指的是真正意义上的“同时执行”。

**3、临界区**

表示一种公共资源或者共享数据，可以被多个线程使用，但是一次只能有一个线程使用，一旦被占用，其他线程想使用就必须等待。

**4、阻塞（Blocking）和非阻塞（Non-Blocking）**

形容多线程间的互相影响。比如一个线程占用了临界区资源，其他线程就必须等待，导致线程挂起，叫做阻塞。非阻塞强调没有一个线程可以妨碍其他线程执行。

**5、死锁（Deadlock）、饥饿（Starvation）和活锁（Livelock）**

死锁：两个或多个线程都在等待对方释放锁，导致线程都处于阻塞状态。

饥饿：一个或多个线程由于种种原因无法获得所需要的资源，导致一直无法执行。

活锁：线程为了彼此间的相应而互相礼让，导致资源不断在两个线程间跳动，却没有一个线程正常执行。

## 并发级别

1、阻塞

线程无法执行时进入阻塞状态。

2、无饥饿（Starvation-Free）

保证线程都有机会执行。

3、无障碍（Obstruction-Free）

乐观策略，认为多个线程之间的冲突几率不大，所有线程都无障碍的执行，检测到冲突后进行回滚。

4、无锁（Lock-Free）

包含一个无穷循环，在这个循环中，线程会不断尝试修改共享变量。无锁的并行总能保证有一个线程在有限步内是可以完成的。

5、无等待（Wait-Free）

要求所有的线程必须在有限步内完成。

## 回到Java：JMM

Java内存模型（JMM）的关键技术点都是围绕着多线程的原子性、可见性和有序性来建立的。

1、原子性（Atomicity）

指一个操作是不可中断的。

2、可见性（Visibility）

指一个线程修改了某一个共享变量的值时，其他线程是否能够立即知道这个修改。

3、有序性（Ordering）

程序在执行时，可能会发生指令重排，指令重排后的指令与原指令顺序未必一致。

注意：对于一个线程来说，他看到的指令执行顺序一定是一致的，指令重排不会使串行的语义逻辑发生问题。

4、哪些指令不能重排：Happen-Before规则

（1）程序顺序原则：一个线程内保证语义的串行性。

（2）volatile规则：volatile变量的写先于读发生，这保证了volatile变量的可见性。

（3）锁规则：解锁（unlock）必然发生在随后的加锁（lock）前。

（4）传递性：A先于B，B先于C，那么A必然先于C。

（5）线程的start（）方法先于它的每一个动作。

（6）线程的所有操作先于线程的终结。

（7）线程的中断（interrupt（））先于被中断线程的代码。

（8）对象的构造函数的执行、结束先于finalize（）方法。

# 第二章 Java并行程序基础

线程和进程的关系：进程是线程的容器，进程可以容纳若干个线程。

线程的生命周期：

![](../images/20181120173640764.jpeg)

## 2.2 线程的基本操作

### 1、新建线程

Java使用Thread类代表线程，所有的线程对象都必须是Thread类或其子类的实例。Java可以用三种方法来创建线程：

（1）继承Thread类创建线程

通过定义Thread类的子类，并重写该类的run（）方法，该方法的方法体就是线程需要完成的任务，run（）方法也称为线程执行体。然后创建Thread子类的实例，也就是创建了线程对象。最后启动线程，即调用线程的start（）方法。

```java
public class myThread1 extends Thread {

    public static void main(String[] args) {

        myThread1 t1=new myThread1();

        t1.start();

    }

    @Override

    public void run() {

        System.out.println("hello");
    }

}
```

（2）实现Runnable接口创建线程

Runnable是一个单方法接口，它只有一个run（）方法。这也是最常用的方法。

```java
public class myThread2 implements Runnable {


    public static void main(String[] args) {


        Thread t2 = new Thread(new myThread2());

        t2.start();


    }

    @Override

    public void run() {

        System.out.println("hello");

    }


}
```

（3）使用Callable和Future创建线程

***\*和Runnable接口不一样，Callable接口提供了一个call（）方法作为线程执行体，call()方法比run()方法\*******\*功能要强大。\****

Callable接口是一个泛型接口，call()方法可以有返回值，其返回值的类型就是传递进来的类型。

call()方法可以声明抛出异常

Java5提供了Future接口来代表Callable接口里call()方法的返回值，并且为Future接口提供了一个实现类FutureTask，这个实现类既实现了Future接口，还实现了Runnable接口，因此可以作为Thread类的target。在Future接口里定义了几个公共方法来控制它关联的Callable任务。

创建并启动有返回值的线程的步骤如下：

(1)创建Callable接口的实现类，并实现call()方法，然后创建该实现类的实例（从java8开始可以直接使用Lambda表达式创建Callable对象）。

(2)使用FutureTask类来包装Callable对象，该FutureTask对象封装了Callable对象的call()方法的返回值

(3)使用FutureTask对象作为Thread对象的target创建并启动线程（因为FutureTask实现了Runnable接口）

(4)调用FutureTask对象的get()方法来获得子线程执行结束后的返回值

```java
public class myThread3 implements Callable<String> {

    @Override

    public String call() throws Exception {

        return "hello"

    }


    public static void main(String[] args) {

        FutureTask<String> futureTask = new FutureTask<>(new myThread3());

        Thread t3 = new Thread(futureTask);

        t3.start();

        try {

            System.out.println(futureTask.get());

        } catch (Exception e) {

            e.printStackTrace();

        }

    }
```

### 2、终止线程

可以使用stop（）方法来强制终止线程，但是强烈不推荐使用，有可能会引起未知错误。

建议使用退出标志来终止线程：一般来说，当run方法执行完后，线程就会退出。但有时run方法是永远不会结束的。如在服务端程序中使用线程进行监听客户端请求，或是其他的需要循环处理的任务。 在这种情况下，一般是将这些任务放在一个循环中，如while循环。如果想让循环永远运行下去，可以使用while（true）{……}来处理。但要想使 while循环在某一特定条件下退出，最直接的方法就是设一个boolean类型的标志，并通过设置这个标志为true或false来控制while循环是否退出。

### 3、线程中断

线程中断并不会使线程立即退出，而是给线程发送一个通知，线程间接收到通知后如何处理，由目标线程自行决定。

```java
public void Thread.interrupt()              //通知目标线程中断，即设置中断标志位。

public boolean Thread.isInterrupted()       //通过检查中断标志位判断目标线程是否被中断

public static boolean Thread.interrupted()  //判断当前线程是否中断，并清除中断标志位状态
```

一个简单的例子：

```java
public class InterruptTest {
    public static void main(String[] args) throws InterruptedException {

        Thread t1=new Thread(){

            @Override

            public void run() {

                while (true) {

                    if (Thread.currentThread().isInterrupted()) {

                        System.out.println("Interruted!");

                        break;

                    }

                    Thread.yield();

                }

            }

        };

        t1.start();

        t1.interrupt();

    }

}
```

Thread.sleep() 方法会让当前线程休眠若干时间，当线程在sleep()休眠时，如果被中断，就会产生InterruptedException异常。需要捕获异常进行处理。

### 4、等待（wait）和通知（notify）

等待wait() 方法和通知notify() 方法并不是Thread类中的，而是输出Object类。任何对象都可以调用这两个方法。

当在一个对象实例上调用了obj.wait() 方法后，当前线程就会在这个对象上等待（进入等待队列），直到有其他线程调用了obj.notify() 方法，它就从等待队列中随机唤醒一个线程（notifyAll() 会唤醒所有线程）。

Object.wait() 方法和notify() 方法必须包含在对应的synchronized语句中，在执行方法前首先需要获得目标对象的一个监视器。

```java
public class SimpleWN {

    final static Object object = new Object();

    public static class T1 extends Thread {

        @Override

        public void run() {

            synchronized (object) {

                System.out.println(System.currentTimeMillis() + ":T1 start!");

                try {

                    System.out.println(System.currentTimeMillis()+":T1 wait for object");

                    object.wait();

                } catch (InterruptedException e) {

                    e.printStackTrace();

                }

            }

            System.out.println(System.currentTimeMillis()+":T1 end!");

        }

    }

    public static class T2 extends Thread {

        @Override

        public void run() {

            synchronized (object) {

                System.out.println(System.currentTimeMillis() + ":T2 start!notify one thread");

                object.notify();

                System.out.println(System.currentTimeMillis()+":T2 end!");

                try {

                    Thread.sleep(3000);

                } catch (InterruptedException e) {

                    e.printStackTrace();

                }

            }

        }

    }
    public static void main(String[] args) {

        Thread t1 = new T1();

        Thread t2 = new T2();

        t1.start();t2.start();

    }

}
```

输出如下：

```
1553926016134:T1 start!

1553926016134:T1 wait for object

1553926016140:T2 start!notify one thread

1553926016140:T2 end!

1553926019141:T1 end!
```

T1先申请锁，wait() 方法执行后，T1进行等待，释放锁。T2获得锁，执行notify() 方法，然后休眠三秒后释放锁。T1得到通知后开始尝试获得锁，但是在三秒之后才能获得锁执行。

wait() 和sleep() 方法的重要区别是wait() 会释放目标对象的锁，而sleep() 方法不会释放任何资源。

### 5、挂起（suspend）和继续执行（resume）线程

方法已废弃，不推荐使用，因为挂起时不释放任何锁资源。

### 6、等待线程结束（join）和谦让（yeild）

一个线程的输入可能依赖于另一个线程或多个线程的输出，需要等待依赖线程执行完毕，才能继续执行，使用join() 方法实现这个功能。

```java
public class JoinMain {

    public volatile static int i=0;

    public static class AddThread extends Thread {

        @Override

        public void run() {

            while (i < 1000000){

                i++;

            };

        }

    }


    public static void main(String[] args) throws InterruptedException {

        AddThread at = new AddThread();

        at.start();

        at.join();//join方法的本质是让调用线程wait() 方法在当前线程对象实例上

        System.out.println(i);

    }

}
```

在主函数中，如果不使用join（）方法等待AddThread，那么得到的i很可能是0或者一个非常小的数字。因为AddThread还没开始执行，i的值就已经被输出了。但在使用join（）方法后，主线程等待AddThread执行完毕才执行，输出为1000000。join() 方法的本质是让调用线程wait（）方法在当前线程对象实例上。

Thread.yeild() 方法是一个静态方法，一旦执行，会使当前线程让出CPU。但其之后还会重新参与CPU资源的争夺。

## 2.3 volatile与Java内存模型（JMM）

volatile定义：Java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致地更新，线程应该确保通过排他锁单独获得这个变量。

实现原理：有volatile变量修饰的共享变量进行写操作时，转变为汇编代码时会增加一行**Lock**前缀的指令，该指令会做两件事情：

将当前处理器缓存行的数据写回到系统内存；这个写回内存的操作使其他CPU里缓存了该内存地址的数据无效。

当用volatile（易变的，不稳定的）关键字**声明一个变量**时，就等于告诉虚拟机，这个变量极有可能被某些程序或者线程修改。为了**确保这个变量被修改后，所有线程都能“看到”这个改动**，虚拟机就必须保证这个变量的可见性等特点。

```java
public class Visibility {

    private static volatile boolean ready;//如果不使用volatile关键字，那么ReaderThread线程无法看到主线程的修改，导致ReaderThread线程永远无法退出

    private static int number;

    private static class ReaderThread extends Thread {

        @Override

        public void run() {

            while (!ready);

            System.out.println(number);

        }

    }

    public static void main(String[] args) throws InterruptedException {

        new ReaderThread().start();

        Thread.sleep(1000);

        number=42;

        ready = true;

        Thread.sleep(1000);

    }

}
```

关键字volatile并不能代替锁，也无法保证复合操作（如i++）的原子性。

## 2.4 分门别类的管理：线程组

相同功能的线程可以放到一个线程组里。

```java
public class ThreadGroupTest implements Runnable {

    public static void main(String[] args) {

        ThreadGroup tg = new ThreadGroup("PrintGroup");//建立线程组

        Thread t1 = new Thread(tg, new ThreadGroupTest(), "T1");//创建线程时指定所属的线程组

        Thread t2 = new Thread(tg, new ThreadGroupTest(), "T2");

        t1.start();
        
        t2.start();
        
        System.out.println(tg.activeCount());//获得活动线程的总数

        tg.list();

    }

    @Override

    public void run() {

        String groupAndName = Thread.currentThread().getThreadGroup().getName() +

                "-" + Thread.currentThread().getName();

        while (true) {

            System.out.println("i'm " + groupAndName);

            try {

                Thread.sleep(3000);

            } catch (InterruptedException e) {

                e.printStackTrace();

            }

        }

    }

}
```

## 2.5 驻守后台：守护线程（Daemon）

守护线程是一种特殊的线程，在后台完成一些系统性的服务。

```java
public class DaemonTest {

    public static class myDaemon extends Thread {

        @Override

        public void run() {

            while (true) {

                System.out.println("I am alive");

                try {

                    Thread.sleep(1000);

                } catch (InterruptedException e) {

                    e.printStackTrace();

                }

            }

        }

    }


    public static void main(String[] args) throws InterruptedException {

        Thread t = new myDaemon();
        
        t.setDaemon(true);//把线程设置为守护线程，必须在start()之前

        t.start();

        Thread.sleep(2000);

    }

}
```

当一个Java应用中只有守护线程时，虚拟机就会自动退出。上述代码中若不把线程设置为守护线程，那t线程就会不停打印输出。

## 2.6 先做重要的事：线程优先级

在Java中，使用1到10表示线程优先级，数字越大优先级越高。使用setPriority（）方法对线程进行设置，高优先级的线程倾向于更快的完成。

## 2.7 线程安全的概念与关键字synchronized

synchronized的用法：

（1）指定加锁对象：给指定**对象**加锁，进入同步代码前要获得给定对象的锁

```java
public class AccountingSync implements Runnable {

    static AccountingSync instance = new AccountingSync();

    static int i=0;

    @Override//将关键字synchronized作用于一个给定对象instance
             //因此，每次线程进入被关键字synchronized包裹的代码段时，就会请求instance实例的锁。
    public void run() {

        for (int j = 0; j < 100000; j++) {
            
            synchronized (instance) {

                i++;

            }

        }

    }

    public static void main(String[] args) throws InterruptedException {

        Thread t1 = new Thread(instance);

        Thread t2 = new Thread(instance);

        t1.start();t2.start();

        t1.join();t2.join();

        System.out.println(i);

    }
}
```

（2）直接作用于实例方法：相当于对当前对象**实例**加锁，进入同步代码前要获得当前实例的锁。

```java
    public synchronized void increase() {   
        i++;
    }

    @Override
    public void run() {
        for (int j = 0; j < 100000; j++) 
            increase();
        }
    }
//其他代码同上
```

（3）直接作用于静态方法：相当于对当前**类**加锁，进入同步代码前要获得当前类的锁。

```java
public class AccountingSync3 implements Runnable {
    static int i=0;
    public static synchronized void increase() {
        i++;
    }
    
    @Override
    public void run() {
        for (int j = 0; j < 100000; j++) 
            increase();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new AccountingSync3());//注意，只用对当前类加锁后才能使用这种方式
//        因为如果只是对方法加锁时t1和t2线程指向了不同的对象实例。
   
        Thread t2 = new Thread(new AccountingSync3());
        t1.start();t2.start();

        t1.join();t2.join();
        
        System.out.println(i);
    }
}
```

# 第三章 JDK并发包

## 3.1 多线程的团队协作：同步控制

同步控制是并发程序必不可少的重要手段。

### 1、关键字synchronized的功能扩展：重入锁

重入锁可以完全替代关键字synchronized。重入锁使用java.util.concurrent.locks.ReentrantLock类来实现。

```java
public class ReenterLock implements Runnable {

    public static ReentrantLock lock = new ReentrantLock();
    public static int i=0;

    @Override
    public void run() {

        for (int j = 0; j < 100000; j++) {

            lock.lock();
            try {
                i++;

            }finally {
                
                lock.unlock();
            }
        }
    }
}
```

与关键字synchronized相比，重入锁有着显示的操作过程。必须手动指示何时加锁，何时释放锁。因此，重入锁对逻辑控制的灵活性要远远优于synchronized。

重入锁是可以反复进入的，一个线程可以多次获得同一把锁，但是要记得释放相同次数。

重入锁的高级功能：

（1）中断响应

对于关键字synchronized来说，如果一个线程在等待锁，那么结果只有两种情况，要么获得这把锁继续执行，要么保持等待。而重入锁则使得线程在等待锁的时候可以被中断。这种情况对于处理死锁是有一定帮助的。

```java
public class IntLock implements Runnable {
    public static ReentrantLock lock1 = new ReentrantLock();
    public static ReentrantLock lock2 = new ReentrantLock();

    int lock;
    public IntLock(int lock) {
        this.lock = lock;
    }

    @Override
    public void run() {
        try {
            if (lock == 1) {
                lock1.lockInterruptibly();//这个请求锁的方法可以对中断进行相应
                Thread.sleep(500);
                lock2.lockInterruptibly();
                } else {
                lock2.lockInterruptibly();
                Thread.sleep(500);
                lock1.lockInterruptibly();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            if (lock1.isHeldByCurrentThread())//查询当前线程是否保持该锁
                lock1.unlock();
            if (lock2.isHeldByCurrentThread())
                lock2.unlock();
            System.out.println(Thread.currentThread().getId()+"线程退出");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        IntLock r1 = new IntLock(1);
        IntLock r2 = new IntLock(2);

        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);

        t1.start();t2.start()
        Thread.sleep(1000);
        
        t2.interrupt();//中断t2
    }
}
```

线程t1和t2启动后，由于互相请求对方持有的锁造成死锁。如果不中断t2，那么会无限期等待下去。中断t2后，t2会放弃对lock1的申请，同时释放已获得的lock2，从而使得t1获得lock2顺利执行下去。

中断后两个线程都会退出，但真正完成工作的只有t1。

（2）锁申请等待限时

除了等待外部通知以外，要避免死锁还有另一种方法：限时等待。可以使用tryLock() 方法进行一次限时的等待。

```java
public class TimeLock implements Runnable {
    
    public static ReentrantLock lock = new ReentrantLock();

    @Override
    public void run() {
        try {
            if (lock.tryLock(5, TimeUnit.SECONDS)) {
                Thread.sleep(6000);
            } else {
                System.out.println("get Lock failed");
            }

        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) {
        TimeLock t1 = new TimeLock();
        
        Thread thread1 = new Thread(t1);
        Thread thread2 = new Thread(t1);

        thread1.start();
        thread2.start();
    }
}
```

tryLock() 方法接受两个参数，等待时长和计时单位。由于占用锁的线程会持有锁长达6秒，故另一个线程无法在5秒之内获得锁，请求失败。

ReentrantLock.tryLock()不带参数时，线程会尝试获得锁，申请成功返回true，申请失败则会立即返回false，不在进行等待。

（3）公平锁

使用synchronized关键字进行锁控制时，产生的锁是非公平的，就是说会在等待队列中随机挑选一个获得锁。

重入锁允许对公平性进行设置，构造函数如下：

```java
public ReentrantLock(boolean fair)
```

当参数为true时，表示锁是公平的。公平锁按照时间先后顺序，保证不会产生饥饿现象。但是实现成本较高，性能比较地下，默认状况下为非公平的。

### 2、重入锁的好搭档：Condition

与Object.wait() 和notify() 作用大致相同。

```java
public class ReenterLockCondition implements Runnable {

    public static ReentrantLock lock = new ReentrantLock();
    
    //通过newCondition方法生成一个与当前重入锁绑定的Condition实例。
    public static Condition condition = lock.newCondition();

    @Override
    public void run() {
       try {
            lock.lock();
            //await方法会使当前线程等待在Condition上，同时释放当前锁

            condition.await();
            System.out.println("Thread is going on");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {

        ReenterLockCondition reenterLockCondition = new ReenterLockCondition();

        Thread thread = new Thread(reenterLockCondition);
        thread.start();

        Thread.sleep(2000);
        
        //在signal方法调用时，要求线程先获得相关的锁
        lock.lock();

        //调用signal方法，系统会从当前Condition对象的等待队列中唤醒一个线程
        //同理，signalAll方法会唤醒所有线程
        condition.signal();

        //在signal方法调用后，需要释放相关的锁，让给被唤醒的线程
        //若没有释放锁，线程thread无法继续执行
        lock.unlock();
    }
}
```

### 3、允许多个线程同时访问：信号量（Semaphore）

信号量是对锁的扩展。无论是内部锁synchronized还是重入锁ReentrantLock，一次都只允许一个线程访问一个资源，而信号量可以指定多个线程，同时访问一个资源。

```java
//信号量的构造函数，构造时必须制定准入数，即同时能申请多少个许可
public Semaphore(int permits)
public Semaphore(int permits,boolean fair)  //第二个参数可以指定是否公平

//信号量的方法
public void acquire()  //尝试获得一个准入许可
public void acquireUninterruptibly()  //不响应中断
public boolean tryAcquire()  //尝试获得一个许可，成功返回true，失败返回false，不等待
public boolean tryAcquire(long timeout, TimeUnit unit)  //等待限时
public void release()  //用于访问结束后释放一个许可
```

一个简单的例子：

```java
public class SemapDemo implements Runnable {
    
    //声明一个包含5个许可的信号量
    final Semaphore semp = new Semaphore(5);
    
    @Override
    public void run() {
        try {
            //申请信号量
            semp.acquire();
            Thread.sleep(2000);
            System.out.println(Thread.currentThread().getId()+":done!");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            //离开前要释放信号量
            semp.release();
        }
    }

    public static void main(String[] args) {
        
        ExecutorService exec = Executors.newFixedThreadPool(20);
        final SemapDemo demo = new SemapDemo();

        //同时开启20个线程，但每次只能有5个线程进行输出
        for (int i = 0; i < 20; i++) {
            exec.submit(demo);
        }
    }
}
```

### 4、ReadWriteLock读写锁

读写分离锁能有效地帮助减少锁竞争，提升系统性能。

一般来说，读与读之间不互斥，可以并行操作；读与写、写与写之间仍会互相阻塞。

```java
public class ReadWriteLockDemo {

//    private static Lock lock = new ReentrantLock();
    private static ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    private static Lock readLock = readWriteLock.readLock();    //读取锁
    private static Lock writeLock = readWriteLock.writeLock();  //写入锁
    
    private int value;
    
    public Object handleRead(Lock lock) throws InterruptedException {
        try {
            lock.lock();               //模拟读操作
            Thread.sleep(1000);  //读操作的耗时越多，读写锁的优势就越明显
            return value;
        }finally {
            lock.unlock();
        }
    }

    public void handleWrite(Lock lock, int index) throws InterruptedException {
        try {
            lock.lock();               //模拟写操作
            Thread.sleep(1000);  //写操作的耗时越多，读写锁的优势就越明显
            value = index;
        }finally {
            lock.unlock();
        }
    }
    
    public static void main(String[] args) {
        
        final ReadWriteLockDemo demo = new ReadWriteLockDemo();
        
        Runnable readRunable=new Runnable() {
            @Override
            public void run() {
                try {
                    demo.handleRead(readLock);
//                    demo.handleRead(lock);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };

        Runnable writeRunable =new Runnable() {
            @Override
            public void run() {
                try {
                    demo.handleWrite(writeLock,new Random().nextInt());
//                    demo.handleWrite(lock, new Random().nextInt());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
        };

        for (int i = 0; i < 18; i++) {
            new Thread(readRunable).start();
        }

        for (int i = 18; i < 20; i++) {
            new Thread(writeRunable).start();
        }
    }
}
```

以上程序使用读写锁时2秒多就能执行完， 因为读操作是并行的，而使用重入锁时需要20多秒（读操作也会阻塞）。

### 5、倒计数器：CountDownLatch

通常用来控制线程等待，可以让某一个线程等待直到倒计数器结束再开始执行。

```java
public class CountDownLatchDemo implements Runnable {

    //创建一个倒计数器的实例，参数为计数器的计数个数
    //参数为10表示需要10个线程完成任务后等待在CountDownLatch上的线程才能继续执行
    static final CountDownLatch end = new CountDownLatch(10);
    static final CountDownLatchDemo demo = new CountDownLatchDemo();

    @Override
    public void run() {
        try {
            //模拟线程执行时间
            Thread.sleep(new Random().nextInt(10)*1000);
            System.out.println("check complete");

            //一个线程已经完成了任务，倒计数器减一
            end.countDown();

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {

        ExecutorService exec = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 10; i++) {
            exec.submit(demo);
        }

        //要求主线程等待所有检查任务全部完成后，才能继续执行
        end.await();    
        System.out.println("go!");
        exec.shutdown();
    }
}
```

### 6、循环栅栏：CyclicBarrier

类似CountDownLatch，但功能更强大。比如，把计数器设置为10，那么在凑齐第一批10个线程后，一起开始执行任务，直到任务完成，然后计数器就会归零，接着凑齐下一批10个线程。

### 7、线程阻塞工具类：LockSupport

可以在线程内任意位置让线程阻塞。

## 3.2 线程复用：线程池

### 1、什么是线程池

为了避免系统频繁地创建和销毁线程，我们可以让创建的线程复用。在使用线程池后，创建线程变成了从线程池中获得空闲线程，关闭线程变成了向线程池中归还线程（类似数据库连接池）。

### 2、JDK对线程池的支持

Executor框架提供了各种类型的线程池，主要有以下工厂方法：

public static ExecutorService newFixedThreadPool(int nThreads) 
创建固定数目线程的线程池。多于可用线程的任务会被保存在任务队列中，等待线程空闲后处理，

public static ExecutorService newSingleThreadExecutor() 
创建一个单线程化的Executor。

public static ExecutorService newCachedThreadPool() 
创建一个可缓存的线程池，调用execute将重用以前构造的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。

public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) 
创建一个支持定时及周期性的任务执行的线程池，多数情况下可用来替代Timer类。

一个固定大小的线程池的例子：

```java
public class ThreadPoolDemo {

    public static class MyTask implements Runnable {
        @Override
        public void run() {
            System.out.println(System.currentTimeMillis() + ":Thread ID:" + Thread.currentThread().getId());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        
        MyTask myTask = new MyTask();
        ExecutorService es = Executors.newFixedThreadPool(5);
        
        for (int i = 0; i < 10; i++) {
            es.submit(myTask);
        }
    }
}
```

另一个需要注意的是方法是newScheduledThreadPool() 。它可以根据时间需要对线程进行调度。主要方法有：

schedule(Runnable command,long delay, TimeUnit unit)

创建并执行在给定延迟后启用的一次性操作

scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnitunit)

创建并执行一个在给定初始延迟后首次启用的定期操作，后续操作具有给定的周期；也就是将在 initialDelay 后开始执行，然后在initialDelay+period 后执行，接着在 initialDelay + 2 * period 后执行，依此类推

scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)

创建并执行一个在给定初始延迟后首次启用的定期操作，随后，在每一次执行终止和下一次执行开始之间都存在给定的延

```java
public class ScheduledExecutorServiceDemo {

    public static void main(String[] args) 

        ScheduledExecutorService ses = Executors.newScheduledThreadPool(10);
        //如果前面的调度没有完成，那么调度不会启动,也就是说，如果任务执行时间大于间隔时间，在上一个任务执行完成后就会紧接着执行下一个任务
        //而如果使用ses.scheduleWithFixedDelay()方法，那么在上一个任务完成后会在固定的间隔时间之后开始下一个任务
        ses.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                    System.out.println(System.currentTimeMillis()/1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },0,2,TimeUnit.SECONDS);
    }
}
```

### 3、核心线程池的内部实现

对于核心的几个线程池，如newFiexedThreadPool、newSingleThreadExecutor和newCachedThreadPool，都只是对ThreadPoolExecutor类的封装。ThreadPoolExecutor类的构造方法如下：

```java
    public ThreadPoolExecutor(int corePoolSize,//指定了线程池中的线程数量
                              int maximumPoolSize,//最大线程数量
                              long keepAliveTime,//当线程池中的线程数量超过corePoolSize时，多余线程的存活时间
                              TimeUnit unit,//时间单位
                              BlockingQueue<Runnable> workQueue,//任务队列
                              ThreadFactory threadFactory,//线程工厂，用于创建线程
                              RejectedExecutionHandler handler//拒绝策略。当任务太多来不及处理时，如何拒绝任务。)
```

### 4、超负荷了怎么办：拒绝策略

ThreadPoolExecutor的最后一个参数制定了拒绝策略，这是系统超负荷运行时的补救措施。JDK内置了四种拒绝策略：

1、AbortPolicy策略

该策略直接抛出异常，阻止系统工作

2、CallerRunsPolicy策略

只要线程池未关闭，该策略直接在调用者线程中运行当前被丢弃的任务。显然这样不会真的丢弃任务，但是，调用者线程性能可能急剧下降。

3、DiscardOledestPolicy策略

丢弃最老的一个请求任务，也就是丢弃一个即将被执行的任务，并尝试再次提交当前任务。

4、DiscardPolicy策略

默默的丢弃无法处理的任务，不予任何处理。

内置策略均实现了RejectedExecutionHandler接口，若以上策略无法满足需求，可以自己扩展接口实现自定义策略。

```java
public class RejectThreadPoolDemo {
    
    public static class MyTask implements Runnable {

        @Override
        public void run() {
            System.out.println(System.currentTimeMillis() + ":Thread ID:" +
                    Thread.currentThread().getId());
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {

        MyTask task = new MyTask();
        //自定义线程池
        ExecutorService es = new ThreadPoolExecutor(5, 5,
                0L, TimeUnit.MILLISECONDS,
                new LinkedBlockingDeque<Runnable>(10),
                Executors.defaultThreadFactory(),
                new RejectedExecutionHandler() {

                    @Override//实现接口方法
                    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
                        System.out.println(r.toString()+" is discard");
                    }
                });

        for (int i = 0; i < Integer.MAX_VALUE; i++) {
            es.submit(task);
            Thread.sleep(10);
        }
    }
}
```

### 5、自定义创建线程：ThreadFactory

ThreadFactory是一个接口，他只有一个方法：Thread newThread（Runable r）

当线程需要新建线程时，就会调用这个方法。

我们可以自定义ThreadFactory，实现自己的线程创建方法。

```java
    public static void main(String[] args) throws InterruptedException {

        MyTask task = new MyTask();
        ExecutorService es = new ThreadPoolExecutor(5, 5,
                0L, TimeUnit.MILLISECONDS,
                new SynchronousQueue<Runnable>(),
                new ThreadFactory() {
                    @Override//自定义线程工厂
                    public Thread newThread(Runnable r) {
                        Thread t = new Thread(r);
                        t.setDaemon(true);
                        System.out.println("create " + t);
                        return t;
                    }
                });

        for (int i = 0; i < 5; i++) {
            es.submit(task);
        }
        Thread.sleep(2000);
    }
}
```

## 3.3 不要重复发明轮子：JDK的并发容器

### 1、好用的工具类：并发集合简介

JDK提供的并发容器大部分在java.util.concurrent下：

ConcurrentHashMap：一个高效并发的HashMap。是线程安全的。

CopyOnWriteArrayList：一个List，在读多于写的场合性能远远优于Vector。

ConcurrentLinkedQueue：高效的并发队列，使用链表实现。

ConcurrentSkipListMap：跳表的并发实现。

# 第四章 锁的优化及注意事项

# 4.1 有助于提高锁性能的几点建议

**1、减少锁持有时间**

即只在必要的时候进行同步。

**2、减小锁粒度**

如ConcurrentHashMap中并不是对整个HashMap进行加锁，而是对其分段，每段分别加锁。

注：JDK1.8以后ConcurrentHashMap取消了Segment分段锁，采用CAS和synchronized来保证并发安全。数据结构跟HashMap1.8的结构类似，数组+链表/红黑二叉树。synchronized只锁定当前链表或红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，效率又提升N倍。

**3、用读写分离锁来替换独占锁**

读操作和读操作可以并行，读写操作才需要加锁。

**4、锁分离**

将读写锁的思想进一步延伸，就是锁分离。例如：LinkedBlockingQueue中，take() 和put() 函数分别实现从队列中取数据和往队列中增加数据的功能。虽然两个函数都对队列做了修改，但LinkedBlockingQueue是基于链表的，两个操作分别位于队列的首端和尾端，两者并不冲突。因此，在JDK的实现中，用两把不同的锁分离了这两个函数，是两者在真正意义上实现了并发。

**5、锁粗化**

通常情况下，要求每个线程在使用完公共资源后，应该立即释放锁。但是，如果对同一个锁不停地进行请求、同步和释放，也会消耗系统资源。因此，虚拟机在遇到一连串连续地对同一个锁不断进行请求和释放的操作时，便会把所有的锁操作整合成对锁的一次请求，从而减少对锁的请求同步的次数，这个操作叫做锁的粗化。

## 4.2 Java虚拟机对锁优化所作出的努力

**4.2.1 锁偏向**

核心思想是：如果一个线程获得了锁，那么锁就进入偏向模式，当这个线程再次请求锁时，无须再做任何同步操作。对于几乎没有锁竞争的场合，偏向锁的优化效果较好；在竞争激烈的成和，其效果不佳。

**4.2.2 轻量级锁**

如果偏向锁失败，虚拟机并不会立即挂起线程，而是使用一种称为轻量级锁的优化手段。轻量级锁只是简单地将对象头部作为指针指向持有锁的线程堆栈的内部，来判断一个线程是否持有对象锁。如果线程获得轻量级锁成功，则可以顺利进入临界区。如果轻量级锁加锁失败，则表示其他线程抢先争夺到了锁，那么当前线程的锁请求就膨胀为重量级锁。

**4.2.3 自旋锁**

锁膨胀后，为了避免线程在操作系统层面挂起，虚拟机会使用自旋锁。系统假设在不久的将来，线程可以获得这把锁。因此，虚拟机会让当前线程做几个空循环（这也是自旋的含义），在经过若干次循环后，如果可以得到锁，就进入临界区，否则才会挂起。

**4.2.4 锁消除**

锁消除是一种更彻底的锁优化。Java虚拟机在JIT编译时，通过对运行上下文的扫描去除掉不可能存在共享资源竞争的锁。如：在一个不存在并发竞争的场合使用了Vector。

## 4.3 人手一支笔：ThreadLocal

除了控制资源的访问外，我们还可以通过增加资源来保证所有对象的线程安全。比如，让100个人填写个人信息表，如果只有一支笔，所有人就得挨个填写。可以准备100支笔，人手一支，那么很快就能完成。

**1、ThreadLocal的简单使用**

ThreadLocal是一个线程的局部变量，只有当前线程可以访问。既然只有当前线程可以访问，自然是线程安全的。

```java
public class ThreadLocalDemo {

    static ThreadLocal<SimpleDateFormat> t1 = new ThreadLocal<>();
//    private static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-ddHH:mm:ss");
    public static class parseDate implements Runnable {

        int i=0;
        public parseDate(int i) {
            this.i = i;
        }
        
        @Override
        public void run() {
            try {
                if (t1.get() == null) {
                    t1.set(new SimpleDateFormat("yyyy-MM-ddHH:mm:ss"));
                }             
                Date date = t1.get().parse("2019-04-01 19:29:" + i % 60);
                System.out.println(i+":"+date);
            } catch (ParseException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        ExecutorService es = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 1000; i++) {
            es.execute(new parseDate(i));
        }
    }
}
```

使用ThreadLocal的前提是在应用层面上为每一个线程分配了不同的对象，如果在应用上为每一个线程分配了相同的对象实例，那么ThreadLocal也不能保证线程安全。

## 4.4 无锁

对于并发控制而言，锁是一种悲观的策略，它总是假设每一次的临界区操作会产生冲突。而无锁是一种乐观的策略，它会假设对资源的访问是没有冲突的。无锁策略使用一种叫做**比较交换（CAS，Compare And Swap）**的技术来鉴别线程冲突。

**1、与众不同的并发策略：比较交换**

与锁相比，比较交换会使程序看起来复杂一些，但是它对死锁问题天然免疫，线程间的相互影响也较小。更重要的是，无锁策略没有锁竞争和线程间调度带来的系统开销，性能更优越。

CAS算法的过程：它包含三个参数CAS（V,E,N)，其中，V表示要更新的变量，E表示预期值，N表示新值。仅当要更新的变量V等于预期值E时，才会将V设置为新值N，如果V与E不同，说明已有其他线程做了更新，则当前线程什么也不做。最后，CAS返回当前V的真实值。当多个线程同时使用CAS操作一个变量时，只有一个会胜出并成功更新变量，其余均会失败。失败的进行不会被挂起，仅是被告知失败，并且允许再次尝试。

**2、无锁的线程安全整数：AtomicInteger**

JDK并发包中有一个atomic包，里面实现了一些直接使用CAS操作的线程安全的类型。

其中，最常用的一个类就是AtomicInteger，可以把它看做一个整数。与Integer不同，它是可变的，并且是线程安全的。对其进行的任何操作都是用CAS指令进行的。

AtomicInteger的使用方法很简单

```java
public class AtomicIntegerDemo {

    static AtomicInteger i = new AtomicInteger();
    public static class AddThread implements Runnable {

        @Override
        public void run() {
            for (int j = 0; j < 10000; j++) {
                i.incrementAndGet();//当前值加1，并返回新值
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread[] ts = new Thread[10];
        for (int j = 0; j < 10; j++) {
            ts[j] = new Thread(new AddThread());
        }

        for (int j = 0; j < 10; j++) {
            ts[j].start();
        }

        for (int j = 0; j < 10; j++) {
            ts[j].join();
        }

        System.out.println(i);
    }
}
```

和AtomicInteger类似的类还有：AtomicLong用来代表Long型数据；AtomicBoolean表示Boolean型数据；AtomicReference表示对象引用。

**3、无锁的对象引用：AtomicReference**

它可以保证修改对象引用时的线程安全性。但是有一个问题，如果获取对象当前数据后，在准备修改为新值前，对象的值被其他线程连续修改了两次，而经过这两次修改后，对象的值又恢复了旧值，那么，当前线程就无法正确判定这个对象是否被修改过。

比如下面这个场景：商家为了挽留客户，决定为贵宾卡里小于20元的客户一次性赠送20元，条件是，每一个客户只能赠送一次。

如果在赠予金额到账的同时，客户进行了一次消费，使得总金额小于20元，并且消费后的金额等于赠予金额之前的金额，那么后台进行就会误以为这个账户还没有进行赠予，出错。下面的程序模拟了这个场景：

```java
public class AtomicReferenceDemo {

    static AtomicReference<Integer> money = new AtomicReference<Integer>();

    public static void main(String[] args) {
        money.set(19);
        //模拟多个线程同时更新后台数据库，为用户充值

        for (int i = 0; i < 3; i++) {
            new Thread() {
                @Override
                public void run() {
                    while (true) {
                        while (true) {
                            Integer m = money.get();
                            if (m < 20) {
                                if (money.compareAndSet(m, m + 20)) {
                                    System.out.println("余额小于20元，充值成功，余额：" + money.get() + "元");
                                    break;
                                }
                            } else {
                                //System.out.println("余额大于20元，无须充值");
                                break;
                            }
                        }
                    }
                }
            }.start();
        }

        //用户消费线程，模拟消费行为
        new Thread() {
            @Override
            public void run() {
                for (int i = 0; i < 100; i++) {
                    while (true) {
                        Integer m = money.get();
                        if (m > 10) {
                            System.out.println("大于10元");
                            if (money.compareAndSet(m, m - 10)) {
                                System.out.println("成功消费10元，余额：" + money.get());
                                break;
                            }
                        } else {
                            System.out.println("没有足够的金额");
                            break;
                        }
                    }
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }.start();
    }
}
```

JDK使用AtomicStampedReference就可以解决这个问题。

**4、带有时间戳的对象引用：**AtomicStampedReference

AtomicReference无法解决上述问题的原因是对象在修改过程中丢失了状态信息，对象值本身与状态画上了等号，因此，只需记录对象在修改过程中的状态值就能解决这个问题。

AtomicStampedReference内部不仅维护了对象值，还维护了一个时间戳（或者叫版本戳，实际上它可以使任何一个整数来表达状态值）。只要时间戳发生变化，就能防止不恰当的写入。主要方法如下：

```java
//构造方法, 传入引用和戳
public AtomicStampedReference(V initialRef, int initialStamp)

//返回引用
public V getReference()
//返回版本戳
public int getStamp()

//如果当前引用 等于 预期值并且 当前版本戳等于预期版本戳, 将更新新的引用和新的版本戳到内存
public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp)

//如果当前引用 等于 预期引用, 将更新新的版本戳到内存
public boolean attemptStamp(V expectedReference, int newStamp)

//设置当前引用的新引用和版本戳
public void set(V newReference, int newStamp) 
```

解决前面的问题（又称为ABA问题）：

```java
public class AtomicStampedReferenceDemo {

    static AtomicStampedReference<Integer> money = new AtomicStampedReference<>(19,0);
    public static void main(String[] args) {
        //模拟多个线程同时更新后台数据库，为用户充值
        for (int i = 0; i < 3; i++) {
            final int timeStamp = money.getStamp();
            new Thread() {
                @Override
                public void run() {
                    while (true) {
                        while (true) {
                            Integer m = money.getReference();
                            if (m < 20) {
                                if (money.compareAndSet(m, m + 20,timeStamp,timeStamp+1)) {
                                    System.out.println("余额小于20元，充值成功，余额：" + money.getReference() + "元");
                                    break;
                                }
                            } else {
                                //System.out.println("余额大于20元，无须充值");
                                break;
                            }
                        }
                    }
                }
            }.start();
        }
        
        //用户消费线程，模拟消费行为
        new Thread() {
            @Override
            public void run() {
                for (int i = 0; i < 100; i++) {
                    while (true) {
                        int timeStamp = money.getStamp();
                        Integer m = money.getReference();
                        if (m > 10) {
                            System.out.println("大于10元");
                            if (money.compareAndSet(m, m - 10,timeStamp,timeStamp+1)) {
                                System.out.println("成功消费10元，余额：" + money.getReference());
                                break;
                            }
                        } else {
                            System.out.println("没有足够的金额");
                            break;
                        }
                    }
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }.start();
    }
}
```

**5、数组也能无锁：AtomicIntegerArray**

JDK除了提供基本数据类型以外，还提供**AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray**等复合结构。

一个简单的例子：

```java
public class AtomicIntegerArrayDemo {

    static AtomicIntegerArray array = new AtomicIntegerArray(10);
    public static class AddThread implements Runnable {
        @Override
        public void run() {
            //对数组内10个元素每个累加1000次
            for (int i = 0; i < 10000; i++) {
                array.getAndIncrement(i % array.length());
            }
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        //创建10个线程
        Thread[] ts = new Thread[10];
        for (int i = 0; i < 10; i++) {
            ts[i] = new Thread(new AddThread());
        }

        for (int i = 0; i < 10; i++) {
            ts[i].start();
        }

        for (int i = 0; i < 10; i++) {
            ts[i].join();
        }

        System.out.println(array);
    }
}
```