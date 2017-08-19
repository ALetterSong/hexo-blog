title: Java反射与注解
date: 2016-05-29 10:36:01
tags:
---

## 反射
### 了解 Java 中的反射
 什么是 Java 的反射
   Java 反射是可以让我们**(1)在运行时获取类的函数、属性、父类、接口等Class内部信息**的机制。通过反射还可以让我们**(2)在运行期实例化对象，调用方法，通过调用get/set方法获取变量的值，即使方法或属性是私有的**也可以通过反射的形式调用。

编译时我们**对于类的内部信息不可知**，必须得到运行时才能获取类的具体信息。比如ORM框架，在运行时才能够获取类中的各个属性，然后通过反射的形式获取其属性名和值，存入数据库。这也是反射比较经典应用场景之一。  

还有一个比较常见的场景就是我们要**使用的类在运行时才会确定**，这个时候我们不能在编译期就使用它，因此只能通过反射的形式来使用在运行时才存在的类(该类符合某种特定的规范，例如JDBC)，这是反射用得比较多的场景。

 Class 类
   那么既然反射是操作 Class 信息的，Class 又是什么呢?    
   ![这里写图片描述](https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tech/reflection/image/arch.png)  
   当我们编写完一个 Java 项目之后，所有的 Java 文件都会被编译成一个.class 文件，这些 Class 对象承载了这个类型的父类、接口、构造函数、方法、属性等原始信息，这些 class 文件在程序运行时会被 ClassLoader 加载到虚拟机中。当一个类被加载以后，Java 虚拟机就会在内存中自动产生一个 Class 对象。我们通过 new 的形式创建对象实际上就是通过这些 Class 来创建，只是这个过程对于我们是不透明的而已。          
### 反射 Class 以及构造对象
 获取 Class 对象   
   在你想检查一个类的信息之前，你首先需要获取类的 Class 对象。Java 中的所有类型包括基本类型，即使是数组都有与之关联的 Class 类的对象。如果你在编译期知道一个类的名字的话，那么你可以使用如下的方式获取一个类的 Class 对象。

    Class<?> myObjectClass = MyObject.class;

   如果你已经得到了某个对象，但是你想获取这个对象的 Class 对象，那么你可以通过下面的方法得到:
   

    Student me = new Student("mr.simple");
    Class<?> clazz = me.getClass();
       

   如果你在编译期获取不到目标类型，但是你知道它的完整类路径，那么你可以通过如下的形式来获取 Class 对象:


    Class<?> myObjectClass=Class.forName("com.simple.User");

 
在使用 Class.forName()方法时，你必须提供一个类的全名，这个全名包括类所在的包的名字。例如 User 类位于 com.simple 包，那么他的完整类路径就是 com.simple.User。     
如果在调用 Class.forName()方法时，没有在编译路径下(classpath)找到对应的类，那么将会抛出 ClassNotFoundException。     

### 通过 Class 对象构造目标类型的对象	
我们知道，在 java中要构造对象，必须通过该类的构造函数，那么其实反射也是一样一样的。但是它们确实有区别的，通过反射构造对象，我们首先要获取类的 Constructor(构造器)对象，然后通过 Constructor 来创建目标类的对象。还是直接上代码的。     

      private static void classForName() {
            try {
            	// 获取 Class 对象
                Class<?> clz = Class.forName("org.java.advance.reflect.Student");
                // 通过 Class 对象获取 Constructor，Student 的构造函数有一个字符串参数
                // 因此这里需要传递参数的类型 ( Student 类见后面的代码 )
                Constructor<?> constructor = clz.getConstructor(String.class);
                // 通过 Constructor 来创建 Student 对象
                Object obj = constructor.newInstance("mr.simple");
                System.out.println(" obj :  " + obj.toString());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

	
   通过上述代码，我们就可以在运行时通过完整的类名来构建对象。      

注意，当你通过反射获取到Constructor、Method、Field后，在反射调用之前将此对象的 accessible 标志设置为 true，以此来提升反射速度。值为 true 则指示反射的对象在使用时应该取消Java语言访问检查。值为 false 则指示反射的对象应该实施Java语言访问检查。例如 :     

       Constructor<?> constructor = clz.getConstructor(String.class);
       // 设置 Constructor 的 Accessible
       constructor.setAccessible(true);
    
       // 设置 Methohd 的 Accessible
       Method learnMethod = Student.class.getMethod("learn"， String.class);
       learnMethod.setAccessible(true);

### 反射获取类中函数

 获取当前类中定义的方法
   要获取当前类中定义的所有方法可以通过 Class 中的 getDeclaredMethods函数，它会获取到当前类中的 public、default、protected、private的所有方法。而 getDeclaredMethod(String name, Class...<?> parameterTypes)则是获取某个指定的方法。代码示例如下 :  

     private static void showDeclaredMethods() {
          Student student = new Student("mr.simple");
            Method[] methods = student.getClass().getDeclaredMethods();
            for (Method method : methods) {
                System.out.println("declared method name : " + method.getName());
            }
    
            try {
                Method learnMethod = student.getClass().getDeclaredMethod("learn", String.class);
                // 获取方法的参数类型列表
                Class<?>[] paramClasses = learnMethod.getParameterTypes() ;
                for (Class<?> class1 : paramClasses) {
                    System.out.println("learn 方法的参数类型 : " + class1.getName());
                }
                // 是否是 private 函数，属性是否是 private 也可以使用这种方式判断
                System.out.println(learnMethod.getName() + " is private "
                        + Modifier.isPrivate(learnMethod.getModifiers()));
                learnMethod.invoke(student, "java ---> ");
            } catch (Exception e) {
                e.printStackTrace();
            }
        }



 获取当前类、父类中定义的公有方法      
   要获取当前类以及父类中的所有 public 方法可以通过 Class 中的 getMethods 函数，而 getMethod 则是获取某个指定的方法。代码示例如下 : 
   
    private static void showMethods() {
        Student student = new Student("mr.simple");
        // 获取所有方法
        Method[] methods = student.getClass().getMethods();
        for (Method method : methods) {
            System.out.println("method name : " + method.getName());
        }
    
        try {
            // 通过 getMethod 只能获取公有方法，如果获取私有方法则会抛出异常，比如这里就会抛异常
            Method learnMethod = student.getClass().getMethod("learn", String.class);
            // 是否是 private 函数，属性是否是 private 也可以使用这种方式判断
            System.out.println(learnMethod.getName() + " is private " + Modifier.isPrivate(learnMethod.getModifiers()));
            // 调用 learn 函数
            learnMethod.invoke(student, "java");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

 

### 反射获取类中的属性
   获取属性和章节 3 中获取方法是非常相似的，只是从 getMethod 函数换成了 getField，从 getDeclaredMethod 换成了 getDeclaredField 罢了。      

 获取当前类中定义的属性
   要获取当前类中定义的所有属性可以通过 Class 中的 getDeclaredFields 函数，它会获取到当前类中的 public、default、protected、private 的所有属性。而 getDeclaredField 则是获取某个指定的属性。代码示例如下 :  


    private static void showDeclaredFields() {
        Student student = new Student("mr.simple");
        // 获取当前类和父类的所有公有属性
        Field[] publicFields = student.getClass().getDeclaredFields();
        for (Field field : publicFields) {
            System.out.println("declared field name : " + field.getName());
        }
    
        try {
            // 获取当前类和父类的某个公有属性
            Field gradeField = student.getClass().getDeclaredField("mGrade");
            // 获取属性值
            System.out.println(" my grade is : " + gradeField.getInt(student));
            // 设置属性值
            gradeField.set(student, 10);
            System.out.println(" my grade is : " + gradeField.getInt(student));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

  
 获取当前类、父类中定义的公有属性      
   要获取当前类以及父类中的所有 public 属性可以通过 Class 中的 getFields 函数，而 getField 则是获取某个指定的属性。代码示例如下 :  

    private static void showFields() {
        Student student = new Student("mr.simple");
        // 获取当前类和父类的所有公有属性
        Field[] publicFields = student.getClass().getFields();
        for (Field field : publicFields) {
            System.out.println("field name : " + field.getName());
        }
    
        try {
            // 获取当前类和父类的某个公有属性
            Field ageField = student.getClass().getField("mAge");
            System.out.println(" age is : " + ageField.getInt(student));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }




### 反射获取父类与接口
 获取父类
获取 Class 对象的父类。


    Student student = new Student("mr.simple");
    Class<?> superClass = student.getClass().getSuperclass();
    while (superClass != null) {
        System.out.println("Student's super class is : " + superClass.getName());
        // 再获取父类的上一层父类，直到最后的 Object 类，Object 的父类为 null
        superClass = superClass.getSuperclass();
    }
          


 获取接口    
   获取 Class 对象中实现的接口。


    private static void showInterfaces() {
        Student student = new Student("mr.simple");
        Class<?>[] interfaceses = student.getClass().getInterfaces();
        for (Class<?> class1 : interfaceses) {
            System.out.println("Student's interface is : " + class1.getName());
        }
    }
        


### 获取注解信息
在框架开发中，注解加反射的组合使用是最为常见形式的。关于注解方面的知识请参考[注解](http://haoduoyu.cc/2016/05/29/Java反射与注解/#注解)，定义注解时我们会通过@Target 指定该注解能够作用的类型，看如下示例:    


    @Target({
            ElementType.METHOD, ElementType.FIELD, ElementType.TYPE
    })
    @Retention(RetentionPolicy.RUNTIME)
    static @interface Test {

    }
   
上述注解的@target 表示该注解只能用在函数上。通过反射 api 我们也能够获取一个Class对象获取类型、属性、函数等相关的对象，通过这些对象的 getAnnotation 接口获取到对应的注解信息。
   首先我们需要在目标对象上添加上注解，例如 : 
   

    @Test(tag = "Student class Test Annoatation")
    public class Student extends Person implements Examination {
        // 年级
        @Test(tag = "mGrade Test Annotation ")
        int mGrade;
        
        // ......
    }
         	
   然后通过相关的注解函数得到注解信息，如下所示 :   
   

    private static void getAnnotationInfos() {
        Student student = new Student("mr.simple");
        Test classTest = student.getClass().getAnnotation(Test.class);
        System.out.println("class Annotatation tag = " + classTest.tag());
        
        Field field = null;
        try {
            field = student.getClass().getDeclaredField("mGrade");
            Test testAnnotation = field.getAnnotation(Test.class);
            System.out.println("属性的 Test 注解 tag : " + testAnnotation.tag());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
     
  输出结果为 : 

    class Annotatation tag = Student class Test Annoatation
    属性的 Test 注解 tag : mGrade Test Annotation 
## 注解

### Annotation 示例  
Override Annotation  

    @Override
    public void onCreate(Bundle savedInstanceState);
Retrofit Annotation  

    @GET("/users/{username}")
    User getUser(@Path("username") String username);
Butter Knife Annotation  

    @InjectView(R.id.user) EditText username;

Retrofit 为符合 RESTful 规范的网络请求框架  
Butter Knife 为 View 及事件等依赖注入框架  

### Annotation 概念及作用  
 概念  
能够添加到Java源代码的语法元数据。类、方法、变量、参数、包都可以被注解，可用来将信息元数据与程序元素进行关联。

 作用  
a. **标记**，用于告诉编译器一些信息  
b. **编译时动态处理**，如动态生成代码  
c. **运行时动态处理**，如得到注解信息  
这里的三个作用实际对应着后面自定义 Annotation 时说的 @Retention 三种值分别表示的 Annotation   


### Annotation 分类  
 标准 Annotation
标准 Annotation 是指 Java 自带的几个 Annotation。
**@Override** 表示重写函数,
**@Deprecated** 表示不鼓励使用(有更好方式、使用有风险或已不在维护)
**@SuppressWarnings** 表示忽略某项 Warning  
 元 Annotation，@Retention, @Target, @Inherited, @Documented  
元 Annotation 是指用来定义 Annotation 的 Annotation，在后面 Annotation 自定义部分会详细介绍含义  
 自定义 Annotation  
自定义 Annotation 表示自己根据需要定义的 Annotation，定义时需要用到上面的元 Annotation  
这里是一种分类而已，也可以根据作用域分为源码时、编译时、运行时 Annotation，后面在自定义 Annotation 时会具体介绍  

### Annotation 自定义  
 调用

    public class App {

        @MethodInfo(
            author = “trinea.cn+android@gmail.com”,
            date = "2014/02/14",
            version = 2)
        public String getAppName() {
            return "trinea";
        }
    }
这里是调用自定义 Annotation——MethodInfo 的示例。  
MethodInfo Annotation 作用为给方法添加相关信息，包括 author、date、version。  

 定义

    @Documented
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    @Inherited
    public @interface MethodInfo {

        String author() default "trinea@gmail.com";

        String date();

        int version() default 1;
    }
    
这里是 MethodInfo 的实现部分  
(1). 通过 @interface 定义，注解名即为自定义注解名  
(2). 注解配置参数名为注解类的方法名，且：  
a. 所有方法没有方法体，没有参数没有修饰符，实际只允许 public & abstract 修饰符，默认为 public，不允许抛异常  
b. 方法返回值只能是基本类型，String, Class, annotation, enumeration 或者是他们的一维数组  
c. 若只有一个默认属性，可直接用 value() 函数。一个属性都没有表示该 Annotation 为 Mark Annotation  
(3). 可以加 default 表示默认值  

 元 Annotation  
@Documented 是否会保存到 Javadoc 文档中  
@Retention 保留时间，可选值 SOURCE（源码时），CLASS（编译时），RUNTIME（运行时），默认为 CLASS，SOURCE 大都为 Mark Annotation，这类 Annotation 大都用来校验，比如 Override, SuppressWarnings  
@Target 可以用来修饰哪些程序元素，如 TYPE, METHOD, CONSTRUCTOR, FIELD, PARAMETER 等，未标注则表示可修饰所有  
@Inherited 是否可以被继承，默认为 false  

### Annotation 解析  
 运行时 Annotation 解析  
(1) 运行时 Annotation 指 @Retention 为 RUNTIME 的 Annotation，可手动调用下面常用 API 解析

    method.getAnnotation(AnnotationName.class);
    method.getAnnotations();
    method.isAnnotationPresent(AnnotationName.class);
    
其他 @Target 如 Field，Class 方法类似  
getAnnotation(AnnotationName.class) 表示得到该 Target 某个 Annotation 的信息，因为一个 Target 可以被多个 Annotation 修饰  
getAnnotations() 则表示得到该 Target 所有 Annotation  
isAnnotationPresent(AnnotationName.class) 表示该 Target 是否被某个 Annotation 修饰  
(2) 解析示例如下：  

    public static void main(String[] args) {
        try {
            Class cls = Class.forName("cn.trinea.java.test.annotation.App");
            for (Method method : cls.getMethods()) {
                MethodInfo methodInfo = method.getAnnotation(
    MethodInfo.class);
                if (methodInfo != null) {
                    System.out.println("method name:" + method.getName());
                    System.out.println("method author:" + methodInfo.author());
                    System.out.println("method version:" + methodInfo.version());
                    System.out.println("method date:" + methodInfo.date());
                }
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
    
以之前自定义的 MethodInfo 为例，利用 Target（这里是 Method）getAnnotation 函数得到 Annotation 信息，然后就可以调用 Annotation 的方法得到响应属性值  

 编译时 Annotation 解析  
(1) 编译时 Annotation 指 @Retention 为 CLASS 的 Annotation，甴编译器自动解析。需要做的  
a. 自定义类集成自 AbstractProcessor   
b. 重写其中的 process 函数  
这块很多同学不理解，实际是编译器在编译时自动查找所有继承自 AbstractProcessor 的类，然后调用他们的 process 方法去处理  
(2) 假设 MethodInfo 的 @Retention 为 CLASS，解析示例如下：  

    @SupportedAnnotationTypes({ "cn.trinea.java.test.annotation.MethodInfo" })
    public class MethodInfoProcessor extends AbstractProcessor {

        @Override
        public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment env) {
            HashMap<String, String> map = new HashMap<String, String>();
            for (TypeElement te : annotations) {
                for (Element element : env.getElementsAnnotatedWith(te)) {
                    MethodInfo methodInfo = element.getAnnotation(MethodInfo.class);
                    map.put(element.getEnclosingElement().toString(), methodInfo.author());
                }
            }
            return false;
        }
    }
    
SupportedAnnotationTypes 表示这个 Processor 要处理的 Annotation 名字。  
process 函数中参数 annotations 表示待处理的 Annotations，参数 env 表示当前或是之前的运行环境  
process 函数返回值表示这组 annotations 是否被这个 Processor 接受，如果接受后续子的 rocessor 不会再对这个 Annotations 进行处理  

### 几个 Android 开源库 Annotation 原理简析  
 Annotation — Retrofit  
(1) 调用

    @GET("/users/{username}")
    User getUser(@Path("username") String username);
    
(2) 定义  

    @Documented
    @Target(METHOD)
    @Retention(RUNTIME)
    @RestMethod("GET")
    public @interface GET {
      String value();
    }

从定义可看出 Retrofit 的 Get Annotation 是运行时 Annotation，并且只能用于修饰 Method  
(3) 原理  
    
    private void parseMethodAnnotations() {
        for (Annotation methodAnnotation : method.getAnnotations()) {
        Class<? extends Annotation> annotationType = methodAnnotation.annotationType();
        RestMethod methodInfo = null;
      
        for (Annotation innerAnnotation : annotationType.getAnnotations()) {
            if (RestMethod.class == innerAnnotation.annotationType()) {
                methodInfo = (RestMethod) innerAnnotation;
                break;
            }
        }
        ……
        }
    }   
    
[ServiceMethod.java](https://github.com/square/retrofit/blob/parent-2.0.2/retrofit/src/main/java/retrofit2/ServiceMethod.java) 的 parseMethodAnnotations 方法如上，会检查每个方法的每个 Annotation， 看是否被 RestMethod 这个 Annotation 修饰的 Annotation 修饰，这个有点绕，就是是否被 GET、DELETE、POST、PUT、HEAD、PATCH 这些 Annotation 修饰，然后得到 Annotation 信息，在对接口进行动态代理时会掉用到这些 Annotation 信息从而完成调用。  

Retrofit 原理涉及到[动态代理](http://a.codekk.com/detail/Android/Caij/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20Java%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86)，这里原理都只介绍 Annotation，具体原理分析请见 [Android 开源项目实现原理解析](http://a.codekk.com)   

 Annotation — Butter Knife  
(1) 调用

    @InjectView(R.id.user) 
    EditText username;
    
(2) 定义

    @Retention(CLASS) 
    @Target(FIELD)
    public @interface InjectView {
      int value();
    }

可看出 Butter Knife 的 InjectView Annotation 是编译时 Annotation，并且只能用于修饰属性  
(3) 原理  

    @Override 
    public boolean process(Set<? extends TypeElement> elements, RoundEnvironment env) {
        Map<TypeElement, ViewInjector> targetClassMap = findAndParseTargets(env);

        for (Map.Entry<TypeElement, ViewInjector> entry : targetClassMap.entrySet()) {
            TypeElement typeElement = entry.getKey();
            ViewInjector viewInjector = entry.getValue();

            try {
                JavaFileObject jfo = filer.createSourceFile(viewInjector.getFqcn(), typeElement);
                Writer writer = jfo.openWriter();
                writer.write(viewInjector.brewJava());
                writer.flush();
                writer.close();
            } catch (IOException e) {
                error(typeElement, "Unable to write injector for type %s: %s", typeElement, e.getMessage());
            }
        }

        return true;
    }


[ButterKnifeProcessor.java](https://github.com/JakeWharton/butterknife/blob/butterknife-parent-7.0.1/butterknife/src/main/java/butterknife/internal/ButterKnifeProcessor.java) 的 process 方法如上，编译时，在此方法中过滤 InjectView 这个 Annotation 到 targetClassMap 后，会根据 targetClassMap 中元素生成不同的 class 文件到最终的 APK 中，然后在运行时调用 ButterKnife.inject(x) 函数时会到之前编译时生成的类中去找。
这里原理都只介绍 Annotation，具体原理分析请见 [Android 开源项目实现原理解析](http://a.codekk.com)   

[参考资料][1]


  [1]: http://a.codekk.com/