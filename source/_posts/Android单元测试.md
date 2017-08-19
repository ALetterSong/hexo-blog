title: Android单元测试
date: 2016-04-20 18:18:52
tags:
---


昨天面试的时候被问到了单元测试有关的内容，一直想了解这方面的内容却始终没有学习，今天花时间了解了一下，顺带做个总结。

### 测试基础
软件测试（英语：software testing），描述一种用来促进鉴定软件的正确性、完整性、安全性和质量的过程。
软件测试一般分为白箱测试和黑箱测试。

#### 黑箱测试
黑箱测试（black-box testing），也称黑盒测试，是软件测试方法，测试应用程序的功能，而不是其内部结构或运作。测试者不需具备应用程序的代码、内部结构和编程语言的专门知识。测试者只需知道什么是系统应该做的事，即当键入一个特定的输入，可得到一定的输出。测试案例是依应用系统应该做的功能，照规范、规格或要求等设计。测试者选择有效输入和无效输入来验证是否正确的输出。
此测试方法可适合大部分的软件测试，例如集成测试（integration testing）以及系统测试（system testing）。

#### 白箱测试
白箱测试（white-box testing，又称透明盒测试glass box testing、结构测试structural testing等）是一个测试软件的方法，测试应用程序的内部结构或运作，而不是测试应用程序的功能（即黑箱测试）。在白箱测试时，以编程语言的角度来设计测试案例。测试者输入数据验证数据流在程序中的流动路径，并确定适当的输出，类似测试电路中的节点。
白箱测试可以应用于单元测试（unit testing）、集成测试（integration testing）和系统的软件测试流程，可测试在集成过程中每一单元之间的路径，或者主系统跟子系统中的测试。尽管这种测试的方法可以发现许多的错误或问题，它可能无法检测未使用部分的规范。

### Android单元测试
在计算机编程中，单元测试（英语：Unit Testing）又称为模块测试, 是针对程序模块（软件设计的最小单位）来进行正确性检验的测试工作。程序单元是应用的最小可测试部件。在过程化编程中，一个单元就是单个程序、函数、过程等；对于面向对象编程，最小单元就是方法，包括基类（超类）、抽象类、或者派生类（子类）中的方法。

通常来说，程序员每修改一次程序就会进行最少一次单元测试，在编写程序的过程中前后很可能要进行多次单元测试，以证实程序达到软件规格书要求的工作目标，没有程序错误；虽然单元测试不是什么必须的，但也不坏，这牵涉到项目管理的政策决定。

在Android中，单元测试的本质依旧是验证函数的功能，测试框架也是JUnit。JUnit 是一个 Java 编程语言的单元测试框架。JUnit 在测试驱动的开发方面有很重要的发展，是起源于 JUnit 的一个统称为 xUnit 的单元测试框架之一。JUnit 促进了“先测试后编码”的理念，强调建立测试数据的一段代码，可以先测试，然后再应用。这个方法就好比“测试一点，编码一点，测试一点，编码一点……”，增加了程序员的产量和程序的稳定性，可以减少程序员的压力和花费在排错上的时间。

在Java中，编写代码面对的只有类、对象、函数，编写单元测试时可以在测试工程中创建一个对象出来然后执行其函数进行测试，而在Android中，编写代码需要面对的是组件、控件、生命周期、异步任务、消息传递等，虽然本质是SDK主动执行了一些实例的函数，但创建一个Activity并不能让它执行到resume的状态，因此需要JUnit之外的框架支持。

当前主流的单元测试框架AndroidTest和Robolectric，前者需要运行在Android环境上，后者可以直接运行在JVM上，速度也更快，可以直接由Jenkins周期性执行，无需准备Android环境。因此我们的单元测试基于Robolectric。对于一些测试对象依赖度较高而需要解除依赖的场景，我们可以借助Mock框架。

#### 单元测试
下面演示如何在Android Studio中进行单元测试

(1) 创建如下所示文件夹

![test-1](http://7xq3d5.com1.z0.glb.clouddn.com/test-1.png)

(2) 在 build.gradle 中添加 JUnit 依赖

    testCompile 'junit:junit:4.12'

(3) 创建一个被测试类


    public class Calculator {
    
        public double sum(double a, double b) {
            return a + b;
        }
    
        public double subtract(double a, double b) {
            return a - b;
        }
    
        public double divide(double a, double b) {
            return a / b;
        }
    
        public double multiply(double a, double b) {
            return 404;
        }
    }   


(4) Ctrl+Shift+T , 在打开的对话窗口中，选择JUnit4和"setUp/@Before"，同时为所有的计算器运算生成测试方法。

![test-2](http://7xq3d5.com1.z0.glb.clouddn.com/test2.png)

(5) 生成测试类框架，在框架内填入测试方法

    public class CalculatorTest {
    
        private Calculator mCalculator;
    
        @Before
        public void setUp() throws Exception {
            mCalculator = new Calculator();
        }
    
        @Test
        public void testSum() throws Exception {
            //expected: 6, sum of 1 and 5
            assertEquals(6d, mCalculator.sum(1d, 5d), 0);
        }
    
        @Test
        public void testSubtract() throws Exception {
            assertEquals(1d, mCalculator.subtract(5d, 4d), 0);
        }
    
        @Test
        public void testDivide() throws Exception {
            assertEquals(4d, mCalculator.divide(20d, 5d), 0);
        }
    
        @Test
        public void testMultiply() throws Exception {
            assertEquals(10d, mCalculator.multiply(2d, 5d), 0);
        }
    }


(6) Run 单元测试，可以看到有一个失败了

![test-3](http://7xq3d5.com1.z0.glb.clouddn.com/test-3.png)

#### Espresso UI 测试

Espresso 是Google官方提供的Android UI自动化测试的框架。具体可参照[https://google.github.io/android-testing-support-library/docs/espresso/index.html](https://google.github.io/android-testing-support-library/docs/espresso/index.html)

    @RunWith(AndroidJUnit4.class)
    @LargeTest
    public class UITestInstrumentationTest {
    
        private static final String STRING_TO_BE_TYPED = "Peter";
    
        @Rule
        public ActivityTestRule<UITest> mActivityRule = new ActivityTestRule<>(UITest.class);
    
        @Test
        public void sayHello() {
            //首先，找到ID为editText的view，输入Peter，然后关闭键盘；
            onView(withId(R.id.test_editText)).perform(typeText(STRING_TO_BE_TYPED), closeSoftKeyboard());
            //接下来，点击Say hello!的View，我们没有在布局的XML中为这个Button设置id，因此，通过搜索它上面的文字来找到它；
            onView(withText("Say hello!")).perform(click());
            //最后，将TextView上的文本同预期结果对比，如果一致则测试通过；
            String expectedText = "Hello, " + STRING_TO_BE_TYPED + "!";            
    onView(withId(R.id.test_textView)).check(matches(withText(expectedText))); 
        }
    
    }

参考资料：

[https://zh.wikipedia.org/wiki/单元测试](https://zh.wikipedia.org/wiki/%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95)
[http://tech.meituan.com/Android_unit_test.html](http://tech.meituan.com/Android_unit_test.html)
[http://rexstjohn.com/unit-testing-with-android-studio/](http://rexstjohn.com/unit-testing-with-android-studio/)
[http://www.jianshu.com/p/03118c11c199](http://www.jianshu.com/p/03118c11c199)
