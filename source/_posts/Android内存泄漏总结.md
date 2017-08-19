title: Android内存泄漏总结
date: 2016-03-14 12:58:50
tags:
---



## 前言
对于内存泄漏，我想大家在开发中肯定都遇到过，只不过内存泄漏对我们来说并不是可见的，因为它是在堆中活动，而要想检测程序中是否有内存泄漏的产生，通常我们可以借助[LeakCanary][1]、MAT等工具来检测应用程序是否存在内存泄漏，MAT是一款强大的内存分析工具，功能繁多而复杂，而LeakCanary则是由Square开源的一款轻量第三方内存泄漏检测工具，当它检测到程序中有内存泄漏的产生时，它将以最直观的方式告诉我们该内存泄漏是由谁产生的和该内存泄漏导致谁泄漏了而不能回收，供我们复查。
### 内存的分配
Java 程序运行时的内存分配策略有三种,分别是静态分配,栈式分配,和堆式分配，对应的，三种存储策略使用的内存空间主要分别是静态存储区（也称方法区）、栈区和堆区。

 - 静态存储区（方法区）：主要存放静态数据、全局 static 数据和常量。这块内存在程序编译时就已经分配好，并且在程序整个运行期间都存在。
 - 栈区 ：当方法被执行时，方法体内的局部变量都在栈上创建，并在方法执行结束时这些局部变量所持有的内存将会自动被释放。因为栈内存分配运算内置于处理器的指令集中，效率很高，但是分配的内存容量有限。
 - 堆区 ： 又称动态内存分配，通常就是指在程序运行时直接 new 出来的内存。这部分内存在不使用时将会由 Java 垃圾回收器来负责回收。


栈与堆的区别：
在方法体内定义的（局部变量）一些基本类型的变量和对象的引用变量都是在方法的栈内存中分配的。当在一段方法块中定义一个变量时，Java就会在栈中为该变量分配内存空间，当超过该变量的作用域后，该变量也就无效了，分配给它的内存空间也将被释放掉，该内存空间可以被重新使用。
堆内存用来存放所有由new创建的对象（包括该对象其中的所有成员变量）和数组。在堆中分配的内存，将由Java垃圾回收器来自动管理。在堆中产生了一个数组或者对象后，还可以在栈中定义一个特殊的变量，这个变量的取值等于数组或者对象在堆内存中的首地址，这个特殊的变量就是我们上面说的引用变量。我们可以通过这个引用变量来访问堆中的对象或者数组。
举个例子:

    public class Sample() {
        int s1 = 0;
        Sample mSample1 = new Sample();
    
        public void method() {
            int s2 = 1;
            Sample mSample2 = new Sample();
        }
    }
    
    Sample mSample3 = new Sample();
    
Sample 类的局部变量 s2 和引用变量 mSample2 都是存在于栈中，但 mSample2 指向的对象是存在于堆上的。
mSample3 指向的对象实体存放在堆上，包括这个对象的所有成员变量 s1 和 mSample1，而它自己存在于栈中。

结论：
局部变量的基本数据类型和引用存储于栈中，引用的对象实体存储于堆中。—— 因为它们属于方法中的变量，生命周期随方法而结束。
成员变量全部存储与堆中（包括基本数据类型，引用和引用的对象实体）—— 因为它们属于类，类对象终究是要被new出来使用的。

### 引用
静态域的变量会持有Activity的引用。在垃圾回收过程中，你可以对一个对象有很多的强引用。当这些强引用的个数总和为零的时候，垃圾回收器就会释放掉它。
弱引用，就是一种不增加引用总数的持有引用方式。垃圾回收期是否决定要回收一个对象，只取决于它是否还存在强引用。所以说，如果我们将我们的 Activity 持有为弱引用，一旦我们发现弱引用持有的对象已经被销毁了，那么这个 Activity 就已经被垃圾回收器回收了。否则，那可以大概确定这个 Activity 已经被泄露了。

#### 强引用(Strong Reference)
强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。如下：

    Object o=new Object();   //  强引用  
当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。如果不使用时，要通过如下方式来弱化引用，如下：

    o=null;     // 帮助垃圾收集器回收此对象  
 显式地设置o为null，或超出对象的生命周期范围，则gc认为该对象不存在引用，这时就可以回收这个对象。具体什么时候收集这要取决于gc的算法。（关于GC的知识会在以后总结）
 
#### 软引用(Soft Reference)
如果一个对象只具有软引用，则内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。 

    String str=new String("abc");                                     // 强引用  
    SoftReference<String> softRef=new SoftReference<String>(str);     // 软引用   


#### 弱引用(Weak Reference)
 弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。

    String str=new String("abc");      
    WeakReference<String> abcWeakRef = new WeakReference<String>(str);  
    str=null; 

 

## 为什么会产生内存泄漏？
当一个对象已经不需要再使用了，本该被回收时，而有另外一个正在使用的对象持有它的引用从而导致它不能被回收，这导致本该被回收的对象不能被回收而停留在堆内存中，这就产生了内存泄漏。
## 内存泄漏对程序的影响？
内存泄漏是造成应用程序OOM的主要原因之一！我们知道Android系统为每个应用程序分配的内存有限，而当一个应用中产生的内存泄漏比较多时，这就难免会导致应用所需要的内存超过这个系统分配的内存限额，这就造成了内存溢出而导致应用Crash。

## Android中常见的内存泄漏
### 单例造成的内存泄漏
由于单例的静态特性使得单例的生命周期和应用的生命周期一样长，这就说明了如果一个对象已经不需要使用了，而单例对象还持有该对象的引用，那么这个对象将不能被正常回收，这就导致了内存泄漏。 
内存泄露：生命周期更长的静态变量持有旧context而导致activity无法释放造成泄漏！（因此静态变量是很容易因此内存泄露的！）
如下这个典例：

    public class AppManager {
        private static AppManager instance;
        private Context context;
        private AppManager(Context context) {
            this.context = context;
        }
        public static AppManager getInstance(Context context) {
            if (instance != null) {
                instance = new AppManager(context);
            }
            return instance;
        }
    }
    
这是一个普通的单例模式，当创建这个单例的时候，由于需要传入一个Context，所以这个Context的生命周期的长短至关重要： 
1、传入的是Application的Context：这将没有任何问题，因为单例的生命周期和Application的一样长 
2、传入的是Activity的Context：当这个Context所对应的Activity退出时，由于该Context和Activity的生命周期一样长（Activity间接继承于Context），所以当前Activity退出时它的内存并不会被回收，因为单例对象持有该Activity的引用。 
所以正确的单例应该修改为下面这种方式：

    public class AppManager {
        private static AppManager instance;
        private Context context;
        private AppManager(Context context) {
            this.context = context.getApplicationContext();
        }
        public static AppManager getInstance(Context context) {
            if (instance != null) {
                instance = new AppManager(context);
            }
            return instance;
        }
    }
这样不管传入什么Context最终将使用Application的Context，而单例的生命周期和应用的一样长，这样就防止了内存泄漏
### 非静态内部类创建静态实例造成的内存泄漏
有的时候我们可能会在启动频繁的Activity中，为了避免重复创建相同的数据资源，可能会出现这种写法：

    public class MainActivity extends AppCompatActivity {
        private static TestResource mResource = null;
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            if(mManager == null){
                mManager = new TestResource();
            }
            //...
        }
        class TestResource {
            //...
        }
    }
这样就在Activity内部创建了一个非静态内部类的单例，每次启动Activity时都会使用该单例的数据，这样虽然避免了资源的重复创建，不过这种写法却会造成内存泄漏，因为非静态内部类默认会持有外部类的引用，而又使用了该非静态内部类创建了一个静态的实例，该实例的生命周期和应用的一样长，这就导致了该静态实例一直会持有该Activity的引用，导致Activity的内存资源不能正常回收。正确的做法为： 
将该内部类设为静态内部类或将该内部类抽取出来封装成一个单例，如果需要使用Context，请使用ApplicationContext

### Handler造成的内存泄漏
Handler的使用造成的内存泄漏问题应该说最为常见了，平时在处理网络任务或者封装一些请求回调等api都应该会借助Handler来处理，对于Handler的使用代码编写一不规范即有可能造成内存泄漏，如下示例：

    public class MainActivity extends AppCompatActivity {
        private Handler mHandler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                //...
            }
        };
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            loadData();
        }
        private void loadData(){
            //...request
            Message message = Message.obtain();
            mHandler.sendMessage(message);
        }
    }
这种创建Handler的方式会造成内存泄漏，由于mHandler是Handler的非静态匿名内部类的实例，所以它持有外部类Activity的引用，我们知道消息队列是在一个Looper线程中不断轮询处理消息，那么当这个Activity退出时消息队列中还有未处理的消息或者正在处理消息，而消息队列中的Message持有mHandler实例的引用，mHandler又持有Activity的引用，所以导致该Activity的内存资源无法及时回收，引发内存泄漏，所以另外一种做法为：

    public class MainActivity extends AppCompatActivity {
        private MyHandler mHandler = new MyHandler(this);
        private TextView mTextView ;
        private static class MyHandler extends Handler {
            private WeakReference<Context> reference;
            public MyHandler(Context context) {
                reference = new WeakReference<>(context);
            }
            @Override
            public void handleMessage(Message msg) {
                MainActivity activity = (MainActivity) reference.get();
                if(activity != null){
                    activity.mTextView.setText("");
                }
            }
        }
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            mTextView = (TextView)findViewById(R.id.textview);
            loadData();
        }
    
        private void loadData() {
            //...request
            Message message = Message.obtain();
            mHandler.sendMessage(message);
        }
    }
建一个静态Handler内部类，然后对Handler持有的对象使用弱引用，垃圾回收器在回收的时候，是会忽视掉弱引用的，所以包含它的Activity会被正常清理掉。这样在回收时也可以回收Handler持有的对象，这样虽然避免了Activity泄漏，不过Looper线程的消息队列中还是可能会有待处理的消息，所以我们在Activity的Destroy时或者Stop时应该移除消息队列中的消息，更准确的做法如下：

    public class MainActivity extends AppCompatActivity {
        private MyHandler mHandler = new MyHandler(this);
        private TextView mTextView ;
        private static class MyHandler extends Handler {
            private WeakReference<Context> reference;
            public MyHandler(Context context) {
                reference = new WeakReference<>(context);
            }
            @Override
            public void handleMessage(Message msg) {
                MainActivity activity = (MainActivity) reference.get();
                if(activity != null){
                    activity.mTextView.setText("");
                }
            }
        }
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            mTextView = (TextView)findViewById(R.id.textview);
            loadData();
        }
    
        private void loadData() {
            //...request
            Message message = Message.obtain();
            mHandler.sendMessage(message);
        }
    
        @Override
        protected void onDestroy() {
            super.onDestroy();
            mHandler.removeCallbacksAndMessages(null);
        }
    }
使用mHandler.removeCallbacksAndMessages(null);是移除消息队列中所有消息和所有的Runnable。当然也可以使用mHandler.removeCallbacks();或mHandler.removeMessages();来移除指定的Runnable和Message。

### 线程造成的内存泄漏
对于线程造成的内存泄漏，也是平时比较常见的，如下这两个示例可能每个人都这样写过：

    //——————test1
            new AsyncTask<Void, Void, Void>() {
                @Override
                protected Void doInBackground(Void... params) {
                    SystemClock.sleep(10000);
                    return null;
                }
            }.execute();
    //——————test2
            new Thread(new Runnable() {
                @Override
                public void run() {
                    SystemClock.sleep(10000);
                }
            }).start();
上面的异步任务和Runnable都是一个匿名内部类，因此它们对当前Activity都有一个隐式引用。如果Activity在销毁之前，任务还未完成， 
那么将导致Activity的内存资源无法回收，造成内存泄漏。正确的做法还是使用静态内部类的方式，如下：

    static class MyAsyncTask extends AsyncTask<Void, Void, Void> {
            private WeakReference<Context> weakReference;
    
            public MyAsyncTask(Context context) {
                weakReference = new WeakReference<>(context);
            }
    
            @Override
            protected Void doInBackground(Void... params) {
                SystemClock.sleep(10000);
                return null;
            }
    
            @Override
            protected void onPostExecute(Void aVoid) {
                super.onPostExecute(aVoid);
                MainActivity activity = (MainActivity) weakReference.get();
                if (activity != null) {
                    //...
                }
            }
        }
        static class MyRunnable implements Runnable{
            @Override
            public void run() {
                SystemClock.sleep(10000);
            }
        }
    //——————
        new Thread(new MyRunnable()).start();
        new MyAsyncTask(this).execute();
        
这样就避免了Activity的内存资源泄漏，当然在Activity销毁时候也应该取消相应的任务AsyncTask的cancel()方法，避免任务在后台执行浪费资源。

### 资源未关闭造成的内存泄漏
对于使用了BraodcastReceiver，ContentObserver，File，Cursor，Stream，Bitmap等资源的使用，应该在Activity销毁时及时关闭或者注销，否则这些资源将不会被回收，造成内存泄漏。

## 一些建议
1、对 Activity 等组件的引用应该控制在 Activity 的生命周期之内；如果不能就考虑使用getApplicationContext或者getApplication，以避免Activity被外部长生命周期的对象引用而泄露。
2、在涉及到Context时先考虑ApplicationContext，当然它并不是万能的，对于有些地方则必须使用Activity的Context，对于Application，Service，Activity三者的Context的应用场景如下： 
![内存泄漏][2]
其中：NO1表示Application和Service可以启动一个Activity，不过需要创建一个新的task任务队列。而对于Dialog而言，只有在Activity中才能创建 。
3、尽量不要在静态变量或者静态内部类中使用非静态外部成员变量（包括context)，即使要使用，也要考虑适时把外部成员变量置空。对于需要在静态内部类中使用非静态外部成员变量（如：Context、View)，可以在静态内部类中使用弱引用来引用外部类的变量来避免内存泄漏 。
4、对于生命周期比Activity长的内部类对象，并且内部类中使用了外部类的成员变量，可以这样做避免内存泄漏：

 - 将内部类改为静态内部类
 - 静态内部类中使用弱引用来引用外部类的成员变量

5、在 Java 的实现过程中，也要考虑其对象释放，最好的方法是在不使用某对象时，显式地将此对象赋值为 null，比如使用完Bitmap后先调用recycle()，再赋为null,清空对图片等资源有直接引用或者间接引用的数组（使用 array.clear() ; array = null）等，最好遵循谁创建谁释放的原则。
6、保持对对象生命周期的敏感，特别注意单例、静态对象、全局性集合等的生命周期。
7、Handler的持有的引用对象最好使用弱引用，资源释放时也可以清空Handler里面的消息。比如在Activity onStop或者onDestroy的时候，取消掉该Handler对象的Message和Runnable。
8、正确关闭资源，对于使用了BraodcastReceiver，ContentObserver，File，游标Cursor，Stream，Bitmap等资源的使用，应该在Activity销毁时及时关闭或者注销。



参考资料：

[10 条提升Android性能的建议](https://realm.io/cn/news/droidcon-farber-improving-android-app-performance/)
[用LeakCanary检测内存泄漏](https://realm.io/cn/news/droidcon-ricau-memory-leaks-leakcanary/)
[Android性能优化之常见的内存泄漏](http://blog.csdn.net/u010687392/article/details/49909477)
[内存泄露从入门到精通三部曲之基础知识篇][3]
[内存泄露从入门到精通三部曲之排查方法篇][4]
[内存泄露从入门到精通三部曲之常见原因与用户实践][5]
[Android 内存泄漏总结](https://yq.aliyun.com/articles/3009)
[译文：理解Java中的弱引用](http://droidyue.com/blog/2014/10/12/understanding-weakreference-in-java/)
[Java 7之基础- 强引用、弱引用、软引用、虚引用](http://blog.csdn.net/mazhimazh/article/details/19752475)


  [1]: https://github.com/square/leakcanary
  [2]: http://7xoz2q.com1.z0.glb.clouddn.com/%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F
  [3]: http://bugly.qq.com/bbs/forum.php?mod=viewthread&tid=21&highlight=%E5%86%85%E5%AD%98%E6%B3%84%E9%9C%B2%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E7%B2%BE%E9%80%9A%E4%B8%89%E9%83%A8%E6%9B%B2%E4%B9%8B%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E7%AF%87
  [4]: http://bugly.qq.com/bbs/forum.php?mod=viewthread&tid=62&highlight=%E5%86%85%E5%AD%98%E6%B3%84%E9%9C%B2%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E7%B2%BE%E9%80%9A%E4%B8%89%E9%83%A8%E6%9B%B2%E4%B9%8B%E6%8E%92%E6%9F%A5%E6%96%B9%E6%B3%95%E7%AF%87
  [5]: http://bugly.qq.com/bbs/forum.php?mod=viewthread&tid=125&highlight=%E5%86%85%E5%AD%98%E6%B3%84%E9%9C%B2%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E7%B2%BE%E9%80%9A%E4%B8%89%E9%83%A8%E6%9B%B2%E4%B9%8B%E5%B8%B8%E8%A7%81%E5%8E%9F%E5%9B%A0%E4%B8%8E%E7%94%A8%E6%88%B7%E5%AE%9E%E8%B7%B5