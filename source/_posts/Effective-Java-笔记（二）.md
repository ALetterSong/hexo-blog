title: Effective Java 笔记（二）
date: 2015-12-12 15:40:54
tags:
---

> 泛型；枚举和注解；方法；

**泛型**
**NO.23 - NO.29**

**枚举和注解**
[枚举类型（Enumerated Type）][1]很早就出现在编程语言中，它被用来将一组类似的值包含到一种类型当中。而这种枚举类型的名称则会被定义成独一无二的类型描述符，在这一点上和常量的定义相似。不过相比较常量类型，枚举类型可以为申明的变量提供更大的取值范围。
举个例子来说明一下，如果希望为彩虹描绘出七种颜色，你可以在 Java 程序中通过常量定义方式来实现。

     Public static class RainbowColor { 
        // 红橙黄绿青蓝紫七种颜色的常量定义
        public static final int RED = 0; 
        public static final int ORANGE = 1; 
        public static final int YELLOW = 2; 
        public static final int GREEN = 3; 
        public static final int CYAN = 4; 
        public static final int BLUE = 5; 
        public static final int PURPLE = 6; 
     }
使用的时候，你可以在程序中直接引用这些常量。但是，这种方式还是存在着一些问题。
*类型不安全*
由于颜色常量的对应值是整数形，所以程序执行过程中很有可能给颜色变量传入一个任意的整数值，导致出现错误。
*没有命名空间*
由于颜色常量只是类的属性，当你使用的时候不得不通过类来访问。
*一致性差*
因为整形枚举属于编译期常量，所以编译过程完成后，所有客户端和服务器端引用的地方，会直接将整数值写入。这样，当你修改旧的枚举整数值后或者增加新的枚举值后，所有引用地方代码都需要重新编译，否则运行时刻就会出现错误。
*类型无指意性*
由于颜色枚举值仅仅是一些无任何含义的整数值，如果在运行期调试时候，你就会发现日志中有很多魔术数字，但除了程序员本身，其他人很难明白其奥秘。
为了改进 Java 语言在这方面的不足弥补缺陷，5.0 版本 SDK 发布时候，在语言层面上增加了枚举类型。枚举类型的定义也非常的简单，用 enum 关键字加上名称和大括号包含起来的枚举值体即可，例如上面提到的彩虹颜色就可以用新的 enum 方式来重新定义：

    enum RainbowColor { RED, ORANGE, YELLOW, GREEN, CYAN, BLUE, PURPLE }
从上面的定义形式来看，似乎 Java 中的枚举类型很简单，但实际上 Java 语言规范赋予枚举类型的功能非常的强大，它不仅是简单地将整形数值转换成对象，而是将枚举类型定义转变成一个完整功能的类定义。这种类型定义的扩展允许开发者给枚举类型增加任何方法和属性，也可以实现任意的接口。另外，Java 平台也为 Enum 类型提供了高质量的实现，比如默认实现 Comparable 和 Serializable 接口，让开发者一般情况下不用关心这些细节。
回到本文的主题上来，引入枚举类型到底能够给我们开发带来什么样好处呢？一个最直接的益处就是扩大 switch 语句使用范围。5.0 之前，Java 中 switch 的值只能够是简单类型，比如 int、byte、short、char, 有了枚举类型之后，就可以使用对象了。这样一来，程序的控制选择就变得更加的方便，看下面的例子：

     // 定义一周七天的枚举类型			
     public enum WeekDayEnum { Mon, Tue, Wed, Thu, Fri, Sat, Sun } 
    
     // 读取当天的信息
     WeekDayEnum today = readToday(); 
     
     // 根据日期来选择进行活动
     switch(today) { 
      Mon: do something; break; 
      Tue: do something; break; 
      Wed: do something; break; 
      Thu: do something; break; 
      Fri: do something; break; 
      Sat: play sports game; break; 
      Sun: have a rest; break; 
     }
对于这些枚举的日期，JVM 都会在运行期构造成出一个简单的对象实例一一对应。这些对象都有唯一的 identity，类似整形数值一样，switch 语句就根据此来进行执行跳转。
**NO.30 用enum代替int常量**
**NO.31 用实例域代替序数**
**NO.32 用EnumSet代替位域**
**NO.33 用EnumMap代替序数索引**
**NO.34 用接口模拟可伸缩的枚举**

[Java Annotation 及几个常用开源项目注解原理简析][2]

    @Target(ElementType.FIELD)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface ViewBinder {
        int id() default -1;
        String method() default "";
        String type() default "";
    }
@interface是用于自定义注解的，它里面定义的方法的声明不能有参数，也不能抛出异常，并且方法的返回值被限制为简单类型、String、Class、emnus、@interface，和这些类型的数组。

 注解@Target也是用来修饰注解的元注解，它有一个属性ElementType也是枚举类型，值为：ANNOTATION_TYPE，CONSTRUCTOR ，FIELD，LOCAL_VARIABLE，METHOD，PACKAGE，PARAMETER和TYPE，如@Target(ElementType.METHOD) 修饰的注解表示该注解只能用来修饰在方法上。

@RetentionRetention注解表示需要在什么级别保存该注释信息，用于描述注解的生命周期，它有一个RetentionPolicy类型的value，是一个枚举类型，它有以下的几个值：

(1)用@Retention(RetentionPolicy.SOURCE)修饰的注解，指定注解只保留在源文件当中，编译成类文件后就把注解去掉；
(2)用@Retention(RetentionPolicy.CLASS)修饰的注解，指定注解只保留在源文件和编译后的class 文件中，当jvm加载类时就把注解去掉；
(3)用@Retention(RetentionPolicy.RUNTIME )修饰的注解，指定注解可以保留在jvm中，这样就可以使用反射获取信息了。
默认是RUNTIME，这样我们才能在运行的时候通过反射获取并做对应的逻辑处理。
**NO.35 注解优先于命名模式**
**NO.36 坚持使用Override注解**
**NO.37 用标记接口定义类型**

**方法**
**NO.38 检查参数的有效性**
可以使用断言assert进行有效性检查，但assert需要显式开启，不推荐使用断言。
**NO.39 必要时进行保护性拷贝**
**NO.40 谨慎设计方法签名**
(1)谨慎地选择方法的名称
(2)不要过于追求提供便利的方法
(3)避免过长的参数列表（同类型的长参数序列格外有害） 
a.分解成多个方法
b.创建辅助类
c.采用Builder模式
d.对于参数类型，要优先使用接口而不是类
e.对于boolean参数，要优先使用两个元素的枚举类型。
**NO.41 慎用重载**
**NO.42 慎用可变参数**
有时候，有必要编写需要1个或者多个某种类型参数的方法，而不是需要0个或者多个。

    static int min(int firstArg,int...remainingArgs) {
            int min = firstArgs;
            for (int arg : remainingArgs) {
                if (arg < min)
                    min = arg;
            }
            return min;
        }

**NO.43 返回零长度的数组或集合，而不是null**
**NO.44 为所有导出的API元素编写文档注释**



  [1]: https://www.ibm.com/developerworks/cn/java/j-lo-enum/
  [2]: http://www.trinea.cn/android/java-annotation-android-open-source-analysis/