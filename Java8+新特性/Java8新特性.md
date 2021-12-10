## 简介
Java 8 (又称为 jdk 1.8) 是 Java 语言开发的一个主要版本，也是LTS（ long-term support）官方长期支持版本。Java 8 是oracle公司于2014年3月发布，可以看成是自Java 5 以来最具革命性的版本。Java 8为Java语言、编译器、类库、开发工具与JVM带来了大量新特性。 
 
    代码更少(增加了新的语法：Lambda 表达式)  
    强大的 Stream API  
    速度更快  
    最大化减少空指针异常：Optional  
    Nashorn引擎，允许在JVM上运行JS应用  
    便于并行  
- - -
## 清单
    1.接口的新特性  
    2.注解的新特性  
    3.集合的底层源码实现  
    4.新日期时间的API  
    5.Optional类的使用  
    6.Lambda 表达式(Lambda Expressions)  
    7.Stream API  
- - -
## 详细
### 接口的新特性
Java 8中，你可以为接口添加静态方法和默认方法。从技术角度来说，这是完全合法的，只是它看起来违反了接口作为一个抽象定义的理念。

    1.静态方法
    2.默认方法

* 静态方法  
> 静态方法使用 ***static*** 关键字修饰。可以通过接口直接调用静态方法，并执行其方法体。我们经常在相互一起使用的类中使用静态方法。你可以在标准库中找到像Collection/Collections或者Path/Paths这样成对的接口和类。
```
public interface A {
    public static void method() {
	    System.out.println(“hello lambda!");
    }
}

```
* 默认方法
> 默认方法使用 ***default*** 关键字修饰。可以通过实现类对象来调用。我们在已有的接口中提供新方法的同时，还保持了与旧版本代码的兼容性。
比如：java 8 API中对Collection、List、Comparator等接口提供了丰富的默认方法。
```
public interface A {
    public default void method1() {
	    System.out.println("北京");
    }
    default String method2() {
        return "上海";
    }
}
```
> ***"类优先"*** 原则  
若一个接口中定义了一个默认方法，而另外一个父类或接口中又定义了一个同名的方法时:  
1.选择父类中的方法。如果一个父类提供了具体的实现，那么接口中具有相同名称和参数的默认方法会被忽略  
2.接口冲突。如果一个父接口提供一个默认方法，而另一个接口也提供了一个具有相同名称和参数列表的方法（不管方法是否是默认方法），那么实现类必须覆盖该方法来解决冲突  
```
interface MyFunc {
    default String getName() {
        return "Hello Java8!";
    }
}

interface Named {
    default String getName() {
        return "Hello Java!";
    }
}

class MyClass implements MyFunc, Named {
    public String getName() {
        return Named.super.getName();
    }
}
```
- - -
### 注解的新特性
Java 8对注解处理提供了两点改进：可重复的注解及可用于类型的注解。此外，反射也得到了加强，在Java8中能够得到方法参数的名称。这会简化标注在方法参数上的注解。

    1.可重复注解 （Repeating annotations）
    2.类型注解

* 可重复注解
> 自从Java 5 引入注解以来，这个特性变得非常流行并且被广泛使用。然而，注解的一个使用限制是，在同一个位置，相同的注解不能声明两次。Java 8解除了这个规则并且引入了重复注解的概念，它允许相同的注解在同一个位置声明多次。重复注解本身在定义时需要使用@Repeatable注解修饰。实际上，这个并不算是语言的改变，而是一个编译器欺骗，底层技术保持不变。  
使用方式注意两点：1.在需要重复的注解上声明@Repeatable，设置其成员值为包含其的注解（如：XXX.class），2.设置需要重复的注解的Target和Retention等元注解与包含其的注解相同。也就是要想定义重复注解，必须给它定义的容器类，还要使用 @Repeatable 注解修饰一下
```
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.lang.reflect.Method;

import static java.lang.annotation.ElementType.*;

/**
 * @author: panlingfeng
 * @createDate: 2021/12/9 11:01 上午
 */
public class Demo {
    public static void main(String[] args) throws NoSuchMethodException {
        Class<Demo> clazz = Demo.class;
        Method method = clazz.getMethod("show");
        // 获取方法上的注解
        RepetitionAnnotation[] ras = method.getAnnotationsByType(RepetitionAnnotation.class);
        for (RepetitionAnnotation repetitionAnnotation : ras) {
            System.out.println(repetitionAnnotation.value());
        }
    }

    @RepetitionAnnotation("Hello")
    @RepetitionAnnotation("World")
    public void show() {
    }
}


@Repeatable(RepetitionAnnotations.class)
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.RUNTIME)
@interface RepetitionAnnotation {
    String value() default "ling";
}

/**
 * 容器类
 */
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.RUNTIME)
@interface RepetitionAnnotations {
    RepetitionAnnotation[] value();
}
```

执行输出

    Hello
    World

* 类型注解
> Java 8 扩展了注解的使用场景。现在，它几乎可以对Java中任何元素进行注解：局部变量、泛型、超类、甚至是函数的异常声明。就是扩展了 @Target 枚举类型，添加了***TYPE_PARAMETER、TYPE_USE***

    ElementType.TYPE_PARAMETER 表示该注解能写在类型变量的声明语句中（如：泛型声明）
    ElementType.TYPE_USE 表示该注解能写在使用类型的任何语句中
    
```
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.util.ArrayList;
import java.util.Collection;

public class Annotations {
    @Retention(RetentionPolicy.RUNTIME)
    @Target({ElementType.TYPE_USE, ElementType.TYPE_PARAMETER})
    public @interface NonEmpty {
    }

    public static class Holder<@NonEmpty T> extends @NonEmpty Object {
        public void method() throws @NonEmpty Exception {
        }
    }

    @SuppressWarnings("unused")
    public static void main(String[] args) {
        final Holder<String> holder = new @NonEmpty Holder<String>();
        @NonEmpty Collection<@NonEmpty String> strings = new ArrayList<>();
    }
}
```