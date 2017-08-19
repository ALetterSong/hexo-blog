title: Java多线程与同步机制
date: 2016-01-11 14:27:11
tags:
---

> 部分资料来自[并发编程网][1]

#### 一、多线程优点和实现方法
优点：

 - 资源利用率更好
 - 程序设计在某些情况下更简单
 - 程序响应更快
 
 线程状态图
![线程状态图][2]

Java 中实现多线程有两种方法：继承`Thread`类、实现`Runnable`接口，在程序开发中只要是多线程，肯定永远以实现`Runnable`接口为主，因为实现`Runnable`接口相比继承`Thread`类有如下优势：

 - 可以避免由于 Java 的单继承特性而带来的局限；
 - 增强程序的健壮性，代码能够被多个线程共享，代码与数据是独立的；
 - 适合多个相同程序代码的线程区处理同一资源的情况。
下面以典型的买票程序（基本都是以这个为例子）为例，来说明二者的区别。
首先通过继承 Thread 类实现，代码如下：

	    class MyThread extends Thread{  
	        private int ticket = 5;  
	        public void run(){  
	            for (int i=0;i<10;i++)  
	            {  
	                if(ticket > 0){  
	                    System.out.println("ticket = " + ticket--);  
	                }  
	            }  
	        }  
	    }  
	    
	    public class ThreadDemo{  
	        public static void main(String[] args){  
	            new MyThread().start();  
	            new MyThread().start();  
	            new MyThread().start();  
	        }  
	    } 
    
下面是通过实现 Runnable 接口实现的多线程程序，代码如下：


    class MyThread implements Runnable{  
        private int ticket = 5;  
        public void run(){  
            for (int i=0;i<10;i++)  
            {  
                if(ticket > 0){  
                    System.out.println("ticket = " + ticket--);  
                }  
            }  
        }  
    }  
    
    public class RunnableDemo{  
        public static void main(String[] args){  
            MyThread my = new MyThread();  
            new Thread(my).start();  
            new Thread(my).start();  
            new Thread(my).start();  
        }  
    }
    
 - 在第二种方法（Runnable）中，ticket输出的顺序并不是54321，这是因为线程执行的时机难以预测，`ticket--`并不是原子操作。
 - 在第一种方法中，我们new了3个`Thread`对象，即三个线程分别执行三个对象中的代码，因此便是三个线程去独立地完成卖票的任务；而在第二种方法中，我们同样也new了3个`Thread`对象，但只有一个`Runnable`对象，3 个 `Thread `对象共享这个 `Runnable`对象中的代码，因此，便会出现3 个线程共同完成卖票任务的结果。如果我们new出3个`Runnable`对象，作为参数分别传入3个`Thread`对象中，那么3个线程便会独立执行各自`Runnable` 对象中的代码，即3个线程各自卖5张票。
 - 在第二种方法中，由于 3 个`Thread`对象共同执行一个`Runnable`对象中的代码，因此可能会造成线程的不安全，比如可能ticket会输出 -1（如果我们`System.out....`语句前加上线程休眠操作，该情况将很有可能出现），这种情况的出现是由于，一个线程在判断ticket为1>0后，还没有来得及减 1，另一个线程已经将ticket减1，变为了0，那么接下来之前的线程再将 ticket 减1，便得到了-1。这就需要加入同步操作（即互斥锁），确保同一时刻只有一个线程在执行每次for循环中的操作。而在第一种方法中，并不需要加入同步操作，因为每个线程执行自己`Thread`对象中的代码，不存在多个线程共同执行同一个方法的情况。
 
#### 二、线程安全
在同一程序中运行多个线程本身不会导致问题，问题在于多个线程访问了相同的资源。如，同一内存区（变量，数组，或对象）、系统（数据库，web services等）或文件。实际上，这些问题只有在一或多个线程向这些资源做了写操作时才有可能发生，只要资源没有发生变化,多个线程读取相同的资源就是安全的。

    public class Counter {
        protected long count = 0;
        public void add(long value){
            this.count = this.count + value;   
        }
    }
两个线程分别加了2和3到count变量上，两个线程执行结束后 count 变量的值应该等于5。然而由于两个线程是交叉执行的，两个线程从内存中读出的初始值都是 0。然后各自加了2和3，并分别写回内存。最终的值并不是期望的5，而是最后写回内存的那个线程的值，上面例子中最后写回内存的是线程A，但实际中也可能是线程 B。如果没有采用合适的同步机制，线程间的交叉执行情况就无法预料。
当两个线程竞争同一资源时，如果对资源的访问顺序敏感，就称存在竞态条件。导致竞态条件发生的代码区称作临界区。上例中add()方法就是一个临界区,它会产生竞态条件。在临界区中使用适当的同步就可以避免竞态条件。

#### 三、Java同步块
Java 同步关键字（synchronized）
(1)实例方法同步

    public synchronized void add(int value){
     this.count += value;
     }

注意在方法声明中同步（synchronized）关键字。这告诉 Java 该方法是同步的。
Java 实例方法同步是同步在拥有该方法的对象上。这样，每个实例其方法同步都同步在不同的对象上，即该方法所属的实例。只有一个线程能够在实例方法同步块中运行。如果有多个实例存在，那么一个线程一次可以在一个实例同步块中执行操作。一个实例一个线程。
(2)静态方法同步

    public static synchronized void add(int value){
     count += value;
     }
同样，这里 synchronized 关键字告诉 Java 这个方法是同步的。

静态方法的同步是指同步在该方法所在的类对象上。因为在 Java 虚拟机中一个类只能对应一个类对象，所以同时只允许一个线程执行同一个类中的静态同步方法。

对于不同类中的静态同步方法，一个线程可以执行每个类中的静态同步方法而无需等待。不管类中的那个静态同步方法被调用，一个类只能由一个线程同时执行。

(3)实例方法中的同步块

    public void add(int value){
        synchronized(this){
           this.count += value;
        }
      }
      
      
示例使用 Java 同步块构造器来标记一块代码是同步的。该代码在执行时和同步方法一样。

注意 Java 同步块构造器用括号将对象括起来。在上例中，使用了“this”，即为调用 add 方法的实例本身。在同步构造器中用括号括起来的对象叫做监视器对象。上述代码使用监视器对象同步，同步实例方法使用调用方法本身的实例作为监视器对象。

一次只有一个线程能够在同步于同一个监视器对象的 Java 方法内执行。

下面两个例子都同步他们所调用的实例对象上，因此他们在同步的执行效果上是等效的。

    public class MyClass {
    
    public synchronized void log1(String msg1, String msg2){
           log.writeln(msg1);
           log.writeln(msg2);
        }
    
    public void log2(String msg1, String msg2){
           synchronized(this){
              log.writeln(msg1);
              log.writeln(msg2);
           }
        }
      }
      
在上例中，每次只有一个线程能够在两个同步块中任意一个方法内执行。

如果第二个同步块不是同步在 this 实例对象上，那么两个方法可以被线程同时执行。


(4)静态方法中的同步块

    public class MyClass {
        public static synchronized void log1(String msg1, String msg2){
           log.writeln(msg1);
           log.writeln(msg2);
        }
    
        public static void log2(String msg1, String msg2){
           synchronized(MyClass.class){
              log.writeln(msg1);
              log.writeln(msg2);
           }
        }
      }
      
      
#### 四、常用方法
*start()*： 它的作用是启动一个新线程，新线程会执行相应的`run()`方法。`start()`不能被重复调用。
*run()*： 和普通的成员方法一样，可以被重复调用。单独调用`run()`的话，会在当前线程中执行`run()`，而并不会启动新线程.
*wait()*:  让当前线程进入等待状态，同时，`wait()`也会让当前线程释放它所持有的锁。
*notify()* 和 *notifyAll()*:唤醒当前对象上的等待线程；`notify()`是唤醒单个线程，而`notifyAll()`是唤醒所有的线程。
*yield()*： 暂停当前正在执行的线程对象，并执行其他线程。
*sleep()*: 让当前线程休眠，即当前线程会从运行状态进入到休眠(阻塞)状态。
*join()*: 让一个线程B“加入”到另外一个线程A的尾部。在A执行完毕之前,B不能工作。
*interrupt()*: 中断本线程。

#### 五、示例

    public class ThreadTest {
        
    
    	public static void main(String[] args) {
    		MyThread m1 = new MyThread(1);
    		MyThread m2 = new MyThread(2);
    		m1.start();
    		m2.start();
    	}
    }
    
    final class MyThread extends Thread {
    	private int val;
    	protected  static final Object monitor = new Object();
    	public MyThread(int v) {
    		val = v;
    	}
    	//非线程安全的,锁定的是调用这个同步方法对象。也就是说，当一个对象m1在不同的线程中执行这个同步方法时，
    	//它们之间会形成互斥，达到同步的效果。
    	//但是这个对象所属的Class所产生的另一对象m2却可以任意调用这个被加了synchronized关键字的方法。
    	//如果是静态方法，则可以实现同步
    	public synchronized void print1(int v) {
    		for (int i = 0; i < 100; i++) {
    			System.out.print(v);
    		}
    	}
    
    	public void print2(int v) {
    		//线程安全
    		synchronized (MyThread.class) {
    			for (int i = 0; i < 100; i++) {
    				System.out.print(v);
    			}
    		}
    	}
    	
    	public void print3(int v) {
    		//线程安全
    		synchronized (monitor) {
    			for (int i = 0; i < 100; i++) {
    				System.out.print(v);
    			}
    		}
    	}
    
    	public void run() {
    		print1(val);
    //		print2(val);
    //		print3(val);
    		
    	}
    }


  [1]: http://ifeve.com/java-concurrency-thread-directory/
  [2]: http://static.oschina.net/uploads/space/2013/0621/174442_0BNr_182175.jpg