# 										Spring AOP框架

### 1 AOP的发展过程

#### 1.1 AOP走向现实

##### 1.1.1 静态AOP时代

​	静态AOP，即第一代AOP，以最初的AspectJ为杰出代表，特点是，相应的横切关注点以Aspect形式实现之后，会通过特定的编译器，将实现后Aspect编译并织入到系统的静态类中。

​	静态AOP的优点是，Aspect直接以字节码的形式编译到Java类中，Java虚拟机可以像通常一样加载Java类运行（因为编译完成的Aspect是完全符合Java类的规范的），不会对整个系统的运行造成任何的性能损失。缺点是灵活性不够，如果横切关注点需要改变织入到系统的位置，就需要重新修改Aspect定义文件，然后使用编译器重新编译Aspect并重新织入到系统中。

##### 1.1.2 动态AOP时代

​	动态AOP，即第二代AOP，该时代的AOP框架或产品，大都通过Java语言提供的各种动态特性来实现Aspect织入到当前系统的过程。第二代AOP的AOL大都采用Java语言实现，与静态AOP最大的不同是，AOP的织入过程在系统运行开始之后进行，而不是预先编译到系统类中，而且织入信息大都采用外部XML文件格式保存，可以在调整织入点以及织入逻辑单元的同时，不必变更系统其它模块，甚至在系统运行的时候，也可以动态更改织入逻辑。

​	但动态AOP在引入灵活性以及易用性的同时，也会不可避免地引入相应的性能问题。因为动态AOP的实现产品大都在类加载或者系统运行期间，采用对系统字节码进行操作的方式来完成Aspect到系统的织入，难免会造成一定的运行时性能损失。

#### 1.2 Java平台上的AOP实现机制

##### 1.2.1 动态代理

​	JDK1.3之后，引入了动态代理机制，可以在运行期间，为相应的接口动态生成对应的代理对象。所以，我们可以将横切关注点逻辑封装到动态代理的InvocationHandler中，然后在系统运行期间，根据横切关注点需要织入的模块位置，将横切逻辑织入到相应的代理类中。

​	这种方式实现的唯一缺点或者说优点就是，所有需要织入横切关注点逻辑的模块类都得实现相应的接口，因为动态代理机制只针对接口有效。动态代理是在运行期间使用反射，相对于编译后的静态类的执行，性能上稍逊一些。

​	Spring AOP默认情况下采用这种机制实现AOP机能。

##### 1.2.2 动态字节码增强

​	我们可以为需要织入横切逻辑的模块类在运行期间，通过动态字节码增强技术，为这些系统模块类生成相应的子类，而将横切逻辑加到这些子类中，让应用程序在执行期间使用的是这些动态生成的子类，从而达到将横切逻辑织入系统的目的。不足之处在于，如果需要扩展的类以及类中的实例方法等声明为final的话，则无法对其进行子类化的扩展。

​	Spring AOP在无法采用动态代理机制进行AOP功能扩展的时候，会使用CGLIB库的动态字节码增强支持来实现AOP的功能扩展。

##### 1.2.3 自定义类加载器

​	我们可以通过自定义类加载器的方式完成横切逻辑到系统的织入，自定义类加载器通过读取外部文件规定的织入规则和必要信息，在加载class文件期间就可以将横切逻辑添加到系统模块类的现有逻辑中，然后将改动后的class交给Java虚拟机运行。

##### 1.2.3 AOL扩展

​	AOL扩展是最强大、也最难掌握的一种方式，AspectJ就属于这种方式。

#### 1.3 AOP的组成

##### 1.3.1 Joinpoint

​	在系统运行之前，AOP的功能模块都需要织入OOP的功能模块中。所以，要进行这种织入过程，我们需要知道在系统的哪些执行点上进行织入操作，这些将要在其之上进行织入操作的系统执行点就称之为JoinPoint。

##### 1.3.2 Pointcut

​	Pointcut代表的是Joinpoint的表述方式。将横切逻辑织入到当前系统的过程中，需要参照Pointcut规定的Jointpoint信息，才可以知道应该往系统的哪些Jointpoint上织入横切逻辑。

* **Pointcut有以下的表述方式：**直接指定Jointpoint所在方法名称、正则表达式、使用特定的Pointcut表述语言
* **Pointcut运算：**Pointcut与Pointcut之间可以执行逻辑运算，以得到我们想要的更为确切的Pointcut表述。

##### 1.3.3 Advice

​	Advice是单一横切关注点逻辑的载体，它代表将会织入到Joinpoint的横切逻辑。

* **Before Advice：**是在Jointpoint指定位置之前执行的Advice类型。通常，它不会中断程序执行流程。一般可以使用Before Advice做一些系统的初始化工作，比如设置系统初始值，获取必要系统资源等。

* **After Advice：**After Advice就是在相应连接点之后执行的Advice，可细分为以下三种：

  * After returning Advice：只有当前Jointpoint处执行流程正常完成后，After returning Advice才会执行。比如方法执行正常返回而没有抛出异常。

  * After throwing Advice：又称Throws Advice，只有在当前Jointpoint执行过程中抛出异常的情况下，才会执行。

  * After Advice：After (Finally) Advice，该类型Advice不管Jointpoint处执行流程是正常终了还是抛出异常都会执行。


  ![15](C:\Users\wangxin\Desktop\pics\15.png)

* **Around Advice：**Around Advice对附加其上的Jointpoint进行包裹，可以在Jointpoint之前和之后都指定相应的逻辑，甚至于中断或忽略Jointpoint处原来程序流程的执行。

* **Introduction：**Introduction不是根据横切逻辑在Jointpoint处的执行时机来区分的，而是根据它可以完成的功能而区别于其他Advice类型。

##### 1.3.4 Aspect

​	Aspect是对系统中的横切关注点逻辑进行模块化封装的AOP概念实体。通常情况下，Aspect可以包含多个Pointcut以及相关Advice定义。

##### 1.3.5 织入和织入器

​	织入(Weaving)过程是连接AOP和OOP的桥梁，只有经过织入过程之后，以Aspect模块化的横切关注点才会集成到OOP的现存系统中。Spring AOP使用一组类来完成最终的织入操作，ProxyFactory类则是Spring AOP中最通用的织入器(Weaver)。

##### 1.3.6 目标对象

​	符合Pointcut所指定的条件，将在织入过程中被织入横切逻辑的对象，称为目标对象(Target Object)。

![16](C:\Users\wangxin\Desktop\pics\16.png)



### 2 Spring AOP概述及其实现机制

#### 2.1 Spring AOP的实现机制

​	Spring AOP属于第二代AOP，采用动态代理机制和字节码生成技术实现。默认情况下，如果Spring AOP发现目标对象实现了相应Interface，则采用动态代理机制为其生成代理对象实例。而如果目标对象没有实现任何Interface，Spring AOP会尝试使用一个称为CGLIB（Code Generation Library）的开源的动态字节码生成类库，为目标对象生成动态的代理对象实例。

​	**动态代理相关见另一篇review，这里只记录了动态字节码**

​	使用动态字节码生成技术扩展对象行为的原理是，我们可以对目标对象进行集成扩展，为其生成相应的子类，而子类可以通过覆写来扩展父类的行为，只要将横切逻辑的实现放到子类中，然后让系统使用扩展后的目标对象的子类，就可以达到与代理模式相同的结果了。

![17](C:\Users\wangxin\Desktop\pics\17.png)

​	但是使用继承的方式来扩展对象定义，也不能像静态代理模式那样，为每个不同类型的目标对象都单独创建相应的扩展子类，所以要借助于CGLIB这样的动态字节码生成库，在系统运行期间动态地为目标对象生成相应的扩展子类。下面演示CGLIB的使用：

**目标类：**

```java
public class Requestable{
    public void request(){
        System.out.println("request in Requestable without implement any interface");
    }
}
```

​	要对Requestable类进行扩展，首先需要实现一个net.sf.cglib.proxy.Callback。不过更多的时候，我们会直接使用net.sf.cglib.proxy.MethodInterceptor接口（MethodInterceptor扩展了Callback接口）。

```java
public class RequestCtrlCallback implements MethodInterceptor{
    private static final Log logger = LogFactory.getLog(RequestCtrlCallback.class);
    public Object intercept(Object object,Method method,Object[] args,MethodProxy proxy) throws Throwable{
        if(method.getName().equals("request")){
            //do something
            return Proxy.invokeSuper(object,args);
        }
        return null;
    }  
}

```

​	这样，RequestCtrlCallback就实现了对request()方法请求进行访问控制的逻辑。现在我们通过CGLIB的Enhancer为目标对象动态地生成一个子类，并将RequestCtrlCallback中的横切逻辑附加到该子类中，代码如下所示：

```java
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(Requestable.class);
enhancer.setCallback(new RequestCtrlCallback());

Requestable proxy = (Requestable)enhancer.create();
proxy.request();
```

​	通过为enhancer指定需要生成的子类对应的父类，以及Callback实现，enhancer最终为我们生成了需要的代理对象实例。使用CGLIB对类进行扩展的唯一限制就是无法对final方法进行覆写。



### 3 Spring AOP一代

​	**在动态代理和CGLIB的支持下，Spring AOP框架的实现经过了两代。从Spring的AOP框架第一次发布，到Spring 2.0发布之前的AOP实现，是Spring第一代AOP实现。Spring 2.0发布后的AOP实现是第二代。**

#### 3.1 Spring AOP中的Joinpoint																																																																																																																																																																										

​	在Spring AOP中，仅支持方法级别的Joinpoint，更确切地说，只支持方法执行类型的Joinpoint。

#### 3.2 Spring AOP中的Pointcut

​	Spring中以接口定义**org.springframework.aop.Pointcut**作为其AOP框架中所有Pointcut的最顶层抽象，该接口定义了两个方法用来帮助捕捉系统中的相应Joinpoint，并提供了一个TruePointcut类型的实例。***如果Pointcut类型为TruePointcut，默认会对系统中的所有对象，以及对象上所有被支持的Joinpoint进行匹配***。org.springframework.aop.Pointcut接口定义如以下代码所示：

```java
public interface Pointcut{
    ClassFilter getClassFilter();//类过滤器，用于匹配将被织入操作的对象
    MethodMatcher getMethodMatcher();//方法匹配器，匹配被织入操作的方法
    Pointcut TRUE = TruePointcut.INSTANCE;
}
```

​	ClassFilter接口的作用是对Joinpoint所处的对象进行Class级别的类型匹配，定义如下：

```java
public interface ClassFilter{
    boolean matches(Class clazz);
    ClassFilter TRUE = TrueClassFilter.INSTANCE;
}
```

​	当织入的目标对象的Class类型与Pointcut所规定的类型相符时，matches方法将会返回true，否则返回false，即意味着不会对这个类型的目标对象进行织入操作。

​	比如我们仅希望对系统中Foo类型的类执行织入，则可以如下这样定义ClassFilter：

```java
public class FooClassFilter{
    public boolean matches(Class clazz){
        return Foo.class.equals(clazz);
    }
}
```

​	如果类型对我们所捕捉的Joinpoint无所谓，那么Pointcut中使用的ClassFilter可以直接使用“ClassFilter TRUE = TrueClassFilter.INSTANCE；”。当Pointcut中返回的ClassFilter类型为该类型实例时，Pointcut的匹配将会针对系统中所有的目标类以及他们的实例进行。

​	MethodMatcher定义如下：

```java
public interface MethodMatcher{
    boolean matches(Method method,Class targetClass);
    boolean isRuntime();
    boolean matches(Methos methos,Class targetClass,Object[] args);
    MethodMatcher TRUE = TrueMethodMatcher.INSTANCE;
}
```

​	MethodMatcher通过重载，定义了两个matches方法，而这两个方法的分界线就是isRuntime()方法。在对对象具体方法进行拦截的时候，可以忽略每次方法执行的时候调用者传入的参数，也可以每次都检查这些方法调用参数，以强化拦截条件。

* **isRuntime返回false**，表示不会考虑具体Joinpoint的方法参数，这种类型的MethodMatcher称之为**StaticMethodMatcher**。因为不用每次都检查参数，那么对于同样类型的方法匹配结果，就可以在框架内部缓存以提高性能。isRuntime方法返回false表明当前的MethodMatcher为StaticMethodMatcher的时候，只有Boolean matches(Method method,Class targetClass)方法被执行，它的匹配结果将会成为其所属的Pointcut主要依据。
* 当isRuntime方法返回true时，表明该MethodMatcher将会每次都对方法调用的参数进行匹配检查，这种类型的MethodMatcher称之为DynamicMethodMatcher。因为每次都要对方法参数进行检查，无法对匹配的结果进行缓存，所以，匹配效率相对于StaticMethodMatcher来说要差。如果一个MethodMatcher为DynamicMethodMatcher（isRuntime()返回true），并且当方法boolean matches(Method method,Class targetClass);也返回true的时候，三个参数的matches方法将被执行，以进一步检查匹配条件。



​	在MethodMatcher类型的基础上，Pointcut可以分为两类，即StaticMethodMatcherPointcut和DynamicMethodMatcherPointcut。因为StaticMethodMatcherPointcut具有明显的性能优势，所以，Spring为其提供了更多支持。下图给出了Spring AOP中各Pointcut类型之间的一个局部族谱。

​	![18](C:\Users\wangxin\Desktop\pics\18.png)

##### 3.2.1 常见的Pointcut

​	下图给出了较为常用的几种Pointcut实现：

![19](C:\Users\wangxin\Desktop\pics\19.png)

* **NameMatchMethodPointcut**

  这是最简单的Pointcut实现，属于StaticMethodMatcherPointcut的子类，可以根据自身指定的一组方法名称与Joinpoint处的方法的方法名称进行匹配，比如：

  ```java
  NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
  pointcut.setMappedName("matches");
  //或者传入多个方法名
  pointcut.setMappedName(new String[]{"matches","isRuntime"});
  ```

  但是，NameMatchMethodPointcut无法对重载的方法名进行匹配。NameMatchMethodPointcut除了可以指定方法名，以对指定的JoinPoint进行匹配，还可以使用“*”通配符，实现简单的模糊匹配。如下：

  ```java
  pointcut.setMappedName(new String[]{"match*","*matches","mat*es"});
  ```

* **JdkRegexMethodPointcut和Perl5RegexMethodPointcut**

  StaticMethodMatcherPointcut的子类中有一个专门提供基于正则表达式的实现分支，以抽象类AbstractRegexpMethodPointcut为统帅。与NameMatchMethodPointcut相似，AbstractRegexpMethodPointcut声明了pattern和patterns属性，可以指定一个或者多个正则表达式的匹配模式。

  JdkRegexMethodPointcut的实现基于JDK1.4之后引入的JDK标准正则表达式。简单实用实例如下：

  ```java
  JdkRegexMethodPointcut pointcut = new JdkRegexMethodPointcut();
  pointcut.setPattern(".*match.*");
  //or
  pointcut.setPatterns(new String[]{".*match","。*matches"});
  ```

  ***注意：*****使用正则表达式来匹配相应的Jointpoint所处的方法时，正则表达式的匹配模式必须以匹配整个方法签名(Method Signature)的形式指定，而不能像NameMatchMethodPointcut那样仅给出匹配的方法名称。**

  Perl5RegexMethodPointcut的使用和需要注意的问题与JdkRegexMethodPointcut几乎相同。

* **AnnotationMatchingPointcut**

  AnnotationMatchingPointcut根据目标对象中是否存在指定类型的注解来匹配Joinpoint。要使用该类型的Pointcut，首先需要声明相应的注解。下面是示例：

  ```java
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.TYPE)
  public  @interface ClassLevelAnnotation{//用于类层次
  }
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.Method)
  public @interface MethodLevelAnnotation{//用于方法层次
  }
  ```

  ```java
  //上面两个主角的使用示例
  @ClassLevelAnnotation
  public class GenericTargetObject{
      @MethodLevelAnnotation
      public void gMethod1(){
          //
      }
      public void gMethod2(){
          //
      }
  }
  ```

  针对GenericTargetObject类型，不同的AnnotationMatchingPointcut定义会产生不同的匹配行为

  * ​

    ```java
    AnnotationMatchingPointcut pointcut = new AnnotationMatchingPointcut(ClassLevelAnnotation.class)
    ```

    AnnotationMatchingPointcut仅指定类级别的注解，GenericTargetObject类中所有方法执行的时候，将全部匹配，不管该方法有没有指定注解。也可以通过AnnotationMatchingPointcut的静态方法，来构建类级别的注解对应的AnnotationMatchingPointcut实例。

    ```java
    AnnotationMatchingPointcut pointcut = AnnotationMatchingPointcut.forClassAnnotation(ClassLevelAnnotation.class);
    ```

    ​

  * ```java
    AnnotationMatchingPointcut pointcut = AnnotationMatchingPointcut.forMethodAnnotation(MethodLevelAnnotation.class);
    ```

    如果只指定方法级别的注解而忽略类级别的注解，则该Pointcut仅匹配特定的标注了指定注解的方法定义，而忽略其他方法。对于GenericTargetObject，或者其他类中任何方法来说，只要它标注了@MethodLevelAnnotation，则这些方法将都会被匹配并织入相应的横切逻辑。

  * ```java
    AnnotationMatchingPointcut pointcut = new AnnotationMatchingPointcut(ClassLevelAnnotation.class,MethodLevelAnnotation.class);
    ```

    同时限定类级别的注解和方法级别的注解，可以进一步缩小包围圈。现在，只有标注了@ClassLevelAnnotation的类定义中同时标注了@MethodLevelAnnotation的方法才会被匹配，二者缺一不可。

* **ComposablePointcut**

  ComposablePointcut是Spring AOP提供的可以进行Pointcut逻辑运算的Pointcut实现。它可以进行Pointcut之间的并、交运算。

* **ControlFlowPointcut**

  通过ControlFlowPointcut，可以指定，只有当指定类的方法被调用的时候，才对该方法进行拦截，其他地方调用该方法时，不对此方法进行拦截。（具体书上讲的很细，需要用到时应该回头再看看）因为ControlFlowPointcut类型的Pointcut需要在运行期间检查程序的调用栈，而且每次方法调用都需要检查，所以性能比较差。如果不是十分必要，应该尽量避免这种Pointc的使用。

##### 3.2.2 扩展Pointcut（Customize Pointcut）

​	Spring AOP的Pointcut类型可以划分为StaticMethodMatcherPointcut和DynamicMethodMatcherPointcut两类，要实现自定义Pointcut，通常在这两个抽象类的基础上实现相应子类即可。

* **自定义StaticMethodMatcherPointcut**

  StaticMethodMatcherPointcut为其子类提供了如下几个方面的默认实现

  * 默认所有StaticMethodMatcherPointcut的子类的ClassFilter均为ClassFilter.TRUE，即忽略类的类型匹配。如果具体子类需要对目标对象的类型做进一步限制，可以通过public void setClassFilter(ClassFilter clssFilter)方法设置相应的ClassFilter实现。
  * 因为是StaticMethodMatcherPointcut，所以，其MethodMatcher的isRuntime方法返回false，同时三个参数的matches方法抛出UnsupportedOperationException异常，表示该方法不应该被调用到。

* **自定义DynamicMethodMatcherPointcut**

  DynamicMethodMatcherPointcut也为其子类提供了部分便利。

  * getClassFilter()方法返回ClassFilter.TRUE，如果需要对特定的目标对象类型进行限定，子类只要覆写这个方法即可。
  * 对应的MethodMatcher的isRuntime总是返回true，同时StaticMethodMatcherPointcut提供了两个参数的matches方法的实现，默认直接返回true。

  要实现自定义DynamicMethodMatcherPointcut，通常情况下，我们只需要实现三个参数的matches方法逻辑即可。

#### 3.3 Spring AOP中的Advice

![20](C:\Users\wangxin\Desktop\pics\20.png)

​	Advice实现了将被织入到Pointcut规定的Joinpoint处的横切逻辑。在Spring中，Advice按照其自身实例能否在目标对象类所有实例中共享这一标准，可以划分为两大类，即per-class类型的Advice和per-instance类型的Advice。

##### 3.3.1 per-class类型的Advice

​	per-class类型的Advice是指，该类型的Advice的实例可以在目标对象类的所有实例之间共享。这种类型的Advice通常只是提供方法拦截的功能，不会为目标对象类保存任何状态或者添加新的特性。

* **Before Advice**

  Before Advice所实现的横切逻辑将在相应的Joinpoint之前执行，在Before Advice执行完成之后，程序流程将从Joinpoint处继续执行，所以Before Advice通常不会打断程序的执行流程。

  要在Spring中实现Before Advice，通常只需要实现org.springframework.aop.MethodBeforeAdvice接口即可，该接口定义如下：

  ```java
  public interface MethodBeforeAdvice extends BeforeAdvice{
      void before(Method method,Object[] args,Object target) throws Throwable;
  }
  ```

  MethodBeforeAdvice继承了BeforeAdvice，而BeforeAdvice与AOP Alliance的Advice一样，都是标志接口，其中没有定义任何方法（考虑到了将来的可扩展性）。

* **ThrowsAdvice**

  spring中以接口定义org.springframework.aop.ThrowsAdvice对应通常AOP概念中的AfterThrowingAdvice。虽然该接口没有定义任何方法，但是在实现相应的ThrowsAdvice的时候，我们的方法定义需要遵循如下规则：

  ```java
  void afterThrowing([Method,args,target],ThrowableSubclass);
  ```

  其中，[]中的三个参数可以省略。我们可以根据将要拦截的Throwable的不同类型，在同一个ThrowsAdvice中实现多个afterThrowing方法。框架会使用Java反射机制来调用这些方法。

* **AfterReturningAdvice**

  org.springframework.aop.AfterReturningAdvice接口定义了Spring的AfterReturningAdvice，其定义如下：

  ```java
  public interface AfterReturningAdvice extends AfterAdvice{
      void afterReturning(Object returnValue,Method method,Object[] args,Object target) throws Throwable;
  }
  ```

  通过Spring中的AfterReturningAdvice，我们可以访问当前Joinpoint的方法返回值、方法、方法参数以及所在的目标对象。

* **Around Advice**

  Spring中没有直接定义对应Around Advice的实现接口，而是直接采用AOP Alliance的标准接口，即org.aopalliance.intercept.MethodInterceptor，该接口定义如下：

  ```java
  public interface MethodInterceptor extends Interceptor{
  	Object invoke(MethodInvocation invocation) throws Throwable;
  }
  ```

##### 3.3.2 per-instance 类型的Advice

​	与per-class类型的Advice不同，per-instance类型的Advice不会在目标类所有对象实例之间共享，而是会为不同的实例对象保存它们各自的状态及相关逻辑。在Spring AOP中，Introduction就是唯一的一种per-instance型Advice。

​	Introduction可以在不改动目标类定义的情况下，为目标类添加新的属性以及行为。在Spring中，为目标对象添加新的属性和行为必须声明相应的接口以及相应的实现。这样，再通过特定的拦截器将新的接口定义以及实现类中的逻辑附加到目标对象之上。之后，目标对象（确切说是目标对象的代理对象）就拥有了新的状态和行为。这个特定的拦截器就是org.springframework.aop.IntroductionInterceptor，定义如下：

```java
public interface IntroductionInterceptor extends MethodInterceptor,DynamicIntroductionAdvice{
}

public interface DynamicIntroductionAdvice extends Advice{
    boolean implementsInterface(Class intf);
}
```

IntroductionInterceptor继承了MethodInterceptor以及DynamicIntroductionAdvice。通过DynamicIntroductionAdvice，我们可以界定当前的IntroductionInterceptor为哪些接口类提供相应的拦截功能。通过MethodInterceptor，IntroductionInterceptor就可以处理新添加的接口上的方法调用了。



![21](C:\Users\wangxin\Desktop\pics\21.png)

上图给出了Introduction相关的类图结构。

***TODO...:******后面有关DelegatingIntroductionInterceptor和DelegatePerTargetObjectIntroductionInterceptor的使用暂时不太理解，后面再回头看。***

#### 3.4 Spring AOP中的Aspect

​	Advisor代表Spring中的Aspect，但是，与正常的Aspect不同，Advisor通常只持有一个Pointcut和一个Advice。而理论上，Aspect定义中可以有多个Pointcut和多个Advice，所以，我们可以认为Advisor是一种特殊的Aspect。

​	我们可以将Advisor简单划分为两个分支，一个分支以org.springframework.aop.PointcutAdvisor为首，另一个分支则以org.springframework.aop.Introduction为头儿，如下图所示：

![22](C:\Users\wangxin\Desktop\pics\22.png)

##### 3.4.1 PointcutAdvisor家族

​	实际上，org.springframework.aop.PointcutAdvisor才是真正的定义一个Pointcut和一个Advice的Advisor，大部分的Advisor实现全都是PointcutAdvisor的部下。

![23](C:\Users\wangxin\Desktop\pics\23.png)

下面来看一下几个常用的PointcutAdvisor实现：

* **DefaultPointcutAdvisor**

  ​	DefaultPointcutAdvisor是最通用的PointcutAdvisor实现。除了不能为其制定Introduction类型的Advice之外，剩下的任何类型的Pointcut、任何类型的Advice都可以通过DefaultPointcutAdvisor来使用。我们可以在构造DefaultPointcutAdvisor的时候就明确制定属于当前DefaultPointcutAdvisor实例的Pointcut和Advice，也可以在DefaultPointcutAdvisor实例构造完成后，再通过相应的set方法设置。

  ​	大多数时候，我们会通过IoC容器来注册和使用Spring AOP的各种概念实体。

* **NameMatchMethodPointcutAdvisor**

  ​	NameMatchMethodPointcutAdvisor是细化后的DefaultPointcutAdvisor，它限定了自身可以使用的Pointcut类型为NameMatchMethodPointcutAdvisor，并且外部不可更改。不过，对于使用的advice来说，除了Introduction，其他任何类型的Advice都可以使用。（即只限定Pointcut类型）

  ​	NameMatchMethodPointcutAdvisor内部持有一个NameMatchMethodPointcut类型的Pointcut实例。当通过NameMatchMethodPointcutAdvisor公开的setMappedName和setMappedNames方法设置即将被拦截的方法名称的时候，实际上是在操作NameMatchMethodPointcutAdvisor所持有的这个NameMatchMethodPointcut实例。

* **RegexpMethodPointcutAdvisor**

  ​	与NameMatchMethodPointcutAdvisor类似，RegexpMethodPointcutAdvisor也限定了自身可以使用的Pointcut的类型，即只能通过正则表达式为其设置相应的Pointcut。

  ​	RegexpMethodPointcutAdvisor自身内部持有一个AbstractRegexpMethodPointcut的实例。AbstractRegexpMethodPointcut有两个实现类，即Perl5RegexpMethodPointcut和JdkRegexpMethodPointcut。如果要强制使用Perl5RegexpMethodPointcut，那么可以通过RegexpMethodPointcutAdvisor的塞听Perl(boolean)达成所愿。

* **DefaultBeanFactoryPointcutAdvisor**

  ​	DefaultBeanFactoryPointcutAdvisor是使用比较少的一个Advisor，因为自身绑定到了BeanFactory，所以，要使用DefaultBeanFactoryPointcutAdvisor，我们的应用必须要绑定到Spring的IoC容器。

  ​	DefaultBeanFactoryPointcutAdvisor的作用是，我们可以通过容器中的Advice注册的beanName来关联对应的Advice。只有当对应的Pointcut匹配成功之后，才去实例化对应的Advice，减少了容器启动初期Advisor和Advice之间的耦合性。

##### 3.4.2 IntroductionAdvisor分支

***TODO...:******没怎么理解，还需要重新看***



#### 3.5 Spring AOP的织入

​	**在Spring AOP中，使用类org.springframework.aop.framework.proxyFactory作为织入器。**

##### 3.5.1 ProxyFactory

​	ProxyFactory并非Spring AOP中唯一可用的织入器，而是最基本的一个织入器实现。

​	Spring AOP是基于代理模式的AOP实现，织入过程完成后，会返回织入了横切逻辑的目标对象的代理对象。为ProxyFactory提供必要的原材料之后，ProxyFactory就会返回那个织入完成的代理对象，如以下代码所示：

```java
ProxyFactory weaver = new ProxyFactory(yourTargetObject);
//or
//ProxyFactory weaver = new ProxyFactory();
//weaver.setTarget(task);
Advisor advisor = ...;
weaver.addAdvisor(advisor);
Object proxyObject = weaver.getProxy();
//现在可以使用proxyObject了
```

使用ProxyFactory只需要指定如下两个最基本的东西：

* 第一个是要对其进行织入的目标对象；

* 第二个是将要应用到目标对象的Aspect（在Spring中叫做Advisor）。不过，除了可以指定相应的Advisor之外，还可以使用如下代码，直接指定各种类型的Advice。

  ```java
  weaver.addAdvice(...);
  ```

  * 对于Introduction之外的Advice类型，ProxyFactory内部就会为这些Advice构造相应的Advisor，只不过在为它们构造的Advisor中使用的Pointcut为Pointcut.TRUE，即这些Advice将被应用到系统中所有可识别的Joinpoint处；
  * 如果添加的Advice类型是Introduction类型，则会根据该Introduction的具体类型进行区分：如果是IntroductionInfo的子类实现，因为它本身包含了必要的描述信息，框架内部会为其构造一个DefaultIntroductionAdvisor；而如果是DynamicIntroductionAdvice的子类实现，框架内部将抛出AopConfigException异常（因为无法从DynamicIntroductionAdvice取得必要的目标对象信息）。

  但是，在不同的应用场景下，我们可以指定更多ProxyFactory的控制属性，以便让ProxyFactory帮我们生成必要的代理对象。Spring AOP在使用代理模式实现AOP的过程中采用了动态代理和CGLIB两种机制，分别对实现了某些接口的目标类和没有实现任何接口的目标类进行代理，所以，在使用ProxyFactory对目标类进行代理的时候，会通过ProxyFactory的某些行为控制属性对这两种情况进行区分。

##### 3.5.2 看清ProxyFactory的本质

​	要了解ProxyFactory，我们得先从它的“根”说起，即org.springframework.aop.framework.AopProxy，该接口定义如下：

```java
public interface AopProxy{
    Object getProxy();
    Object getProxy(ClassLoader classLoader);
}
```

​	Spring AOP框架内使用AopProxy对使用的不同的代理实现机制进行了适度的抽象，针对不同的代理实现机制提供相应的AopProxy子类实现。目前，Spring AOP框架内提供了针对JDK的动态代理和CGLIB两种机制的AopProxy实现。

**AopProxy相关结构图：**

![24](C:\Users\wangxin\Desktop\pics\24.png)

​	当前，AopProxy有Cglib2AopProxy和JdkDynamicAopProxy两种实现。因为动态代理需要通过InvocationHandler提供调用拦截，所以，JdkDynamicAopProxy同时实现了InvocationHandler接口。不同AopProxy实现的实例化过程采用工厂模式（确切地说是抽象工厂模式）进行封装，即通过org.springframework.aop.framework.AopProxyFactory进行。AopProxyFactory接口的定义如下所示：

```java
public interface AopProxyFactory{
    AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException;
}
```

AopProxyFactory根据传入的AdvisedSupport实例提供的相关信息，来决定生成什么类型的AopProxy。不过，具体工作会转交给AopProxyFactory的具体实现类。而实际上这个AopProxyFactory实现类现在就一个，即org.springframework.aop.framework.DefaultAopProxyFactory。DefaultAopProxyFactory的实现逻辑很简单，如以下伪代码所示：

```java
if(config.isOptimize()||config.isProxyTargetClass()||hasNoUserSuppliedInterfaces(config)){
    //创建Cglib2AopProxy实例，并返回
}else{
    //创建JDKDynamicAopProxy实例，并返回
}
```

**AdvisedSupport：**

AdvisedSupport其实就是一个生成代理对象所需要的信息的载体，该类相关的类层次图入下：

![25](C:\Users\wangxin\Desktop\pics\25.png)

​	AdvisedSupport所承载的信息可以划分为两类，一类以org.springframework.aop.framework.ProxyConfig为统领，记载生成代理对象的**控制信息**；一类以org.springframework.aop.framework.Advised为旗帜，承载生成代理对象**所需要的必要信息**，如相关目标类Advice、Advisor等。

​	