## 第二章  创建和销毁对象
### 1. 考虑用静态工厂方法代替构造器
#### 什么是静态工厂方法
在Java中获取一个类的实例的方法可以使用`new`关键字，通过构造函数来实现对象的创建。
``` java
Date date = new Date();
```
除此之外还有另外获取实例的方法，如：
``` java
Calendar calendar = Calendar.getInstance();

// OR
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```
用一个静态方法来对外提供自身实例的方法，为所说的`静态工程方法(Static Factory method)`

#### 使用静态工厂方法的优势
##### 第一大优势在于，它们有名字
沟槽器的参数本身没有确切地描述正被返回的对象，那么具有适当名称的静态工厂会更容易使用，产生的客户端代码也更`易于阅读`。
一个类智能有一个带有指定签名的构造器，导致铜鼓哦参数列表区分不同的构造器，再导致用户该如何使用构造器，可能出现调用错误的构造器，而且如果没有文档，对这些构造器的代码阅读十分困难。
当一个类需要多个带相同其阿明的构造器时，用静态工厂方法代替构造器，并慎重地选择名称以便突出它们之间的区别。

##### 第二大优势在于，不必在每次调用它们的时候都创建一个对象
使得不可变类可以使用预先构建好的实例，或者将构建好的实例缓存起来，进行重复利用，从而避免创建不必要的重复对象。
静态工厂方法能够为重复的调用返回相同对象，这样有助于类总嗯你跟严格控制在某个时刻哪些实例应该存在。这种类被称作实例受控的类。
实例受控的类可以确保它是一个Singleton，或者不可实例化的。使得不可变类可以确保不会存在两个相等的实例，客户端可以使用==操作符来代替equals(Object)方法，可以提升性能 

##### 第三大优势在于，可以返回原返回类型的任何子类型的对象
在选择返回对象的类时有了更大的灵活性。
构造方法只能返回确切的自身类型，而静态工厂方法则能够更加灵活，可以根据需要方便地返回任何它的子类型的实例。
``` java 
Class Person {
    public static Person getInstance(){
        return new Person();
        // 这里可以改为 return new Player() / Cooker()
    }
}
Class Player extends Person{
}
Class Cooker extends Person{
}
```
Person 类的静态工厂方法可以返回 Person 的实例，也可以根据需要返回它的子类 Player 或者 Cooker。当然只是举个例子。

Java8之前，接口不能有静态方法。按照惯例，名为Type的接口的静态工厂方法被放在一个不可实例化的伴随类中。比如，Java的Collection框架有45个接口的使用的实现，提供类似不可变集合，同步集合。几乎所有实现都是通过一个不可实例化类的静态工厂方法导出的。返回的对象的类都是不公开的。
自Java8起，接口不能包含静态方法的限制被消除了，所以通常没有理由为接口通过不可实例化的伴随类。许多本来会在这种级别的公共静态成员应该被放在接口本身中。但是，请注意，仍然有必要将这些静态方法背后的大部分实现代码放到单独的包私有类中。这是因为 Java 8 要求接口的所有静态成员都是公共的。Java 9 允许私有静态方法，但是静态字段和静态成员类仍然需要是公共的。

##### 第四大优势在于，返回对象的类可以随调用的不同而变化，作为输入参数的函数
返回任何声明的类型的子类型都是允许的。返回对象的类也可以因版本而异。

比如EnumSet类没有公共的构造器，只有静态工程。在OpenJDK实现中，它们返回两个自雷的一个实例，这却绝育底层enum类型的大小。如果它有64或者更少的元素，如同大多数枚举类型做的一样，静态工厂返回一个long类型的`RegularEnumSet`实例。如果enum类型有65或者更多的元素，工厂返回一个long数组的`JumboEnumSet`实例。
这些对于客户端是不可见的。如果`RegularEnumSet`不再为销项enum类型提供性能优势，可能会在未来的版本中被消除，而不会产生不良影响。当然，如果证明EnumSet有益于性能，未来版本可以添加更多的EnumSet实现。客户端不知道耶不关心它们从工厂活的的对象的类，只关心它是EnumSet的子类。

##### 第五大优势在于，当编写包含方法的类时，返回对象的类不需要存在。
```
“ Such flexible static factory methods form the basis of service provider frameworks, like the Java Database Connectivity API (JDBC).”

摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 
```
在服务提供框架中有三个基本组件：服务接口（代表了一种实现），服务提供者用来注册实现的提供者注册API，一个客户端用来活的服务实例的服务访问API。
服务访问API允许客户端以特定标准的方式选择一种实现。如果没有这个条件，API返回一个默认实现的实例，或者允许客户端循环使用所有可用的实现。
服务提供者框架的第四个可选组件是服务提供者接口，在JDBC中，连接扮演服务接口DriverManager的角色。DriverManager.registerDriver是提供注册的API，DriverManager.getConnection是服务访问API，驱动程序是服务提供者接口。
服务提供者框架模式有许多变体。依赖注入框架可以看作是强大的服务提供者。

#### 只提供静态工厂方法的限制
##### 没有`public`或者`protected`修饰的构造器的类不能被子类化
>“For example, it is impossible to subclass any of the convenience implementation classes in the Collections Framework. ”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

##### 静态工厂方法难以被程序员找到
常用的静态工厂方法（不够全面）：
* from - 一个参数的类型转换方法，返回一个与这个类型相符的实例 `Date d = Date.from(instant);`
* of - 接受多个参数的聚合方法，返回一个这个类型的合并后的实例 `Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KEING);`
* valueOf - 代替from和of更冗长的方法 `BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);`
* instance 或者 getInstance - 返回一个被参数描述的实例，但不能说有相同值 `stachWalker luke = StackWalker.getInstance(options);`
* create 或者 newInstance - 与instance和getInstance类似，除了这个方法保证每次调用都返回一个新的实例 `Object newArray = Array.newInstance(classObject, arrayLen);`
* getType - 类似于getInstance， 但被使用在工厂方法位于不同的类中。Type是工厂方法返回的对象。`FileStore fs = Files.getFileStore(path);`
* newType - 类似于newInstance，但被用于工厂方法位于不同的类中。Type是工厂方法返回的对象。 `BufferedReader br = Files.newBufferedReader(path);`
* type - getType和newType的简洁的替代方式。 `List<Complaint> litany = Collections.list(legacyLitany); `

### 2.当面对多个参数的构造函数时，考虑使用构建器
#### constructor pattern
对于有必填项又有可选项的对象，我们传统的做法是使用可伸缩构造函数（根据参数的数量，不断添加构造函数）。
>“the telescoping constructor pattern works, but it is hard to write client code when there are many parameters, and harder still to read it.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

#### JavaBeans pattern
>“A second alternative when you’re faced with many optional parameters in a constructor is the JavaBeans pattern, in which you call a parameterless constructor to create the object and then call setter methods to set each required parameter and each optional parameter of interest”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 
>
>“Unfortunately, the JavaBeans pattern has serious disadvantages of its own. Because construction is split across multiple calls, a JavaBean may be in an inconsistent state partway through its construction. ”
>“A related disadvantage is that the JavaBeans pattern precludes the possibility of making a class immutable (Item 17) and requires added effort on the part of the programmer to ensure thread safety.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

#### Builder pattern

>“Instead of making the desired object directly, the client calls a constructor (or static factory) with all of the required parameters and gets a builder object. Then the client calls setter-like methods on the builder object to set each optional parameter of interest. Finally, the client calls a parameterless build method to generate the object, which is typically immutable.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

``` java
// Builder Pattern
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val)
            { calories = val;      return this; }
        public Builder fat(int val)
            { fat = val;           return this; }
        public Builder sodium(int val)
            { sodium = val;        return this; }
        public Builder carbohydrate(int val)
            { carbohydrate = val;  return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}

// when using it 
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
        .calories(100).sodium(35).carbohydrate(27).build();
```
>“This client code is easy to write and, more importantly, easy to read. The Builder pattern simulates named optional parameters as found in Python and Scala.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

> “Check invariants involving multiple parameters in the constructor invoked by the build method. To ensure these invariants against attack, do the checks on object fields after copying parameters from the builder (Item 50). If a check fails, throw an IllegalArgumentException (Item 72) whose detail message indicates which parameters are invalid (Item 75).”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

>“The Builder pattern is well suited to class hierarchies. Use a parallel hierarchy of builders, each nested in the corresponding class. Abstract classes have abstract builders; concrete classes have concrete builders. ”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

``` java
// Builder pattern for class hierarchies
public abstract class Pizza {
   public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
   final Set<Topping> toppings;

   abstract static class Builder<T extends Builder<T>> {
      EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
      public T addTopping(Topping topping) {
         toppings.add(Objects.requireNonNull(topping));
         return self();
      }

      abstract Pizza build();
      // Subclasses must override this method to return "this"
      protected abstract T self();
   }
   Pizza(Builder<?> builder) {
      toppings = builder.toppings.clone(); // See Item  50
   }
}

//SubClass
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;”
    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }

        @Override protected Builder self() { return this; }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}

public class Calzone extends Pizza {
    private final boolean sauceInside;
    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // Default

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override public Calzone build() {
            return new Calzone(this);
        }

        @Override protected Builder self() { return this; }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}

//For use
NyPizza pizza = new NyPizza.Builder(SMALL)
        .addTopping(SAUSAGE).addTopping(ONION).build();
Calzone calzone = new Calzone.Builder()
        .addTopping(HAM).sauceInside().build();
```
Builder模式显得比较灵活。

---
为了创建一个对象，必须先创建它的构建器。实际使用时，它的小号可以忽视，但是对于性能为关键的场景下，这可能会是一个问题。

代码显得更加冗长。只有在有足够多的参数时，才值得使用。在考虑使用`contractor`还是`Builder`时，如果未来会添加更多的参数，如先使用构造函数，以后切换到构建器时，构造函数或静态工厂方法会很难处理，所以最好一开始就使用构建器。

--- 
>“In summary, the Builder pattern is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters, especially if many of the parameters are optional or of identical type. Client code is much easier to read and write with builders than with telescoping constructors, and builders are much safer than JavaBeans.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

### 3.用私有构造函数或者枚举类型强化单例属性
>“Making a class a singleton can make it difficult to test its clients because it’s impossible to substitute a mock implementation for a singleton unless it implements an interface that serves as its type.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

>“There are two common ways to implement singletons. Both are based on keeping the constructor private and exporting a public static member to provide access to the sole instance. ”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 
有两种常用方式来实现单例。都基于保持保持构造函数私有化和导出一个公用的静态成员来提供访问到唯一的实例。

#### 第一个例子
``` java
“// Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
}

```
私有化构造函数纸杯初始化公共静态不变的Elvis.INSTANCE是调用一次。
##### 优势
1. API更清晰的知道类是单例的，公共静态字段是final的
2. 更加简单。

#### 第二个例子
``` java
// Singleton with static factory
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
}

```
公共成员是一个静态工厂方法。
都调用Elvis.getInstancd来活的相同对象的引用，而且不会有其他Elvis实例将被创建。
##### 主要优势
1. 可以灵活地调整类是否是单例的，而不用改变API
2. 可以写一个 *generic singleton factory*， 如果应用需要的话
3. 可以使用方法引用。例如Elvis::instance

>To make a singleton class that uses either of these approaches serializable (Chapter 12), it is not sufficient merely to add implements Serializable to its declaration. To maintain the singleton guarantee, declare all instance fields transient and provide a readResolve method (Item 89). Otherwise, each time a serialized instance is deserialized, a new instance will be created, leading, in the case of our example, to spurious Elvis sightings.To prevent this from happening, add this readResolve method to the Elvis class
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

``` java
// readResolve method to preserve singleton property
private Object readResolve() {
     // Return the one true Elvis and let the garbage collector
     // take care of the Elvis impersonator.
    return INSTANCE;
}
```

#### 第三个方法
声明一个单元素枚举
``` java
// Enum singleton - the preferred approach
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```

> This approach is similar to the public field approach, but it is more concise, provides the serialization machinery for free, and provides an ironclad guarantee against multiple instantiation, even in the face of sophisticated serialization or reflection attacks. This approach may feel a bit unnatural, but a single-element enum type is often the best way to implement a singleton. Note that you can’t use this approach if your singleton must extend a superclass other than Enum (though you can declare an enum to implement interfaces).
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

### 4.用私有构造函数强化非实例化性
针对使用程序类(utility)不是为实例化而设计: 实例化是没有意义的。但是在没有显式构造函数的情况下，编译器提供类一个公共的、无参数的默认构造器。
> “Attempting to enforce noninstantiability by making a class abstract does not work. The class can be subclassed and the subclass instantiated. Furthermore, it misleads the user into thinking the class was designed for inheritance (Item 19). There is, however, a simple idiom to ensure noninstantiability. A default constructor is generated only if a class contains no explicit constructors, so a class can be made noninstantiable by including a private constructor:”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

``` java 
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
    ... // Remainder omitted
}
```
因为显式构造函数是私有的，在类之外不可访问，加的异常只是为了提供一层保险，防止构造函数在类内被意外调用。它保证了类永远不会被实例化。因为这个动作反常规，所以加上非实例化的注释。
也有一个另一个作用，防止了类被子类化。所有构造函数必须调用父类的构造函数，无论显式的还是隐式的，自雷都内有可访问的父类的构造函数。

### 5.优先考虑依赖注入来引用资源
有许多类需要依赖一个或者多个底层资源。比如一个拼写检测器依赖一个字典

#### 静态工具类实现
``` java
// Inappropriate use of static utility - inflexible & untestable!
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {} // Noninstantiable

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

#### 单例实现
``` java
// Inappropriate use of singleton - inflexible & untestable!
public class SpellChecker {
    private final Lexicon dictionary = ...;

    private SpellChecker(...) {}
    public static INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

上面两个方法都不是很好，因为他们似乎只有一个字典可以使用。实际场景中每种语言都会有自己的字典，特别的字典需要使用特殊的词库。

> “You could try to have SpellChecker support multiple dictionaries by making the dictionary field nonfinal and adding a method to change the dictionary in an existing spell checker, but this would be awkward, error-prone, and unworkable in a concurrent setting. Static utility classes and singletons are inappropriate for classes whose behavior is parameterized by an underlying resource.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

#### 依赖注入实现
>A simple pattern that satisfies this requirement is to pass the resource into the constructor when creating a new instance. This is one form of dependency injection: the dictionary is a dependency of the spell checker and is injected into the spell checker when it is created.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

``` java
“// Dependency injection provides flexibility and testability
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
} 
```

#### 依赖注入升级版
>“A useful variant of the pattern is to pass a resource factory to the constructor. A factory is an object that can be called repeatedly to create instances of a type. Such factories embody the Factory Method pattern [Gamma95]. The Supplier<T> interface, introduced in Java 8, is perfect for representing factories. Methods that take a Supplier<T> on input should typically constrain the factory’s type parameter using a bounded wildcard type (Item 31) to allow the client to pass in a factory that creates any subtype of a specified type. For example, here is a method that makes a mosaic using a client-provided factory to produce each tile:”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

注入工厂方法且支持创建任何具体类型的子类

```java 
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

---
虽然依赖注入很好的改善了灵活性和可测试性，但它会把典型的有成千上万的依赖的大项目弄的乱七八糟。这些乱七八糟的东西几乎可以被清理，通过使用*依赖注入框架*，比如Dagger，Guice，或者Spring。

>“In summary, do not use a singleton or static utility class to implement a class that depends on one or more underlying resources whose behavior affects that of the class, and do not have the class create these resources directly. Instead, pass the resources, or factories to create them, into the constructor (or static factory or builder). This practice, known as dependency injection, will greatly enhance the flexibility, reusability, and testability of a class.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

### 6.避免创建不必要的对象
一个极端的千万不要做的例子是：
``` java
String s = new String("bikini");
```
>“The statement creates a new String instance each time it is executed, and none of those object creations is necessary. The argument to the String constructor ("bikini") is itself a String instance, functionally identical to all of the objects created by the constructor. If this usage occurs in a loop or in a frequently invoked method, millions of String instances can be created needlessly.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

改善后的版本：
``` java
String s = "bikini;
```
这个版本使用一个String的实例，而不是每次执行时创建一个新的。而且，它保证了对象将被其他运行炸相同虚拟机中的代码重复使用。
可以通过使用静态工厂方法（#1）来避免创建不必要的对象。比如，工程方法`Boolean.valueOf(String)`比构造函数`Boolean(String)`[在Java9中被消除]更好。

有一些对象的创建比其他的更加昂贵。如果需要重复创建这样昂贵的对象，建议是缓存它以便重复使用。但不幸的是在创建这样对象时并不明确。

如果要写一个判断String是不是罗马数字，最简单的方式使用正则表达式:
``` java
// Performance can be greatly improved!
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```
>“While String.matches is the easiest way to check if a string matches a regular expression, it’s not suitable for repeated use in performance-critical situations. ”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

>“Creating a Pattern instance is expensive because it requires compiling the regular expression into a finite state machine.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

改善性能的方法： 将正则表达式编译到不可变的Pattern实例中作为类初始化的一部分，缓存它，然后在每次调用`isRomanNumeral`方法中重复使用相同的实例。
``` java
// Reusing expensive object for improved performance
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
            return ROMAN.matcher(s).matches();
    }
}
```
这个版本的改进，提供了如果经常调用时的明显的性能改进，也使得代码更加清楚。

改进后，如果不使用`isRomanNumeral`方法，但在类加载时，`ROMAN`字段仍会被不必要地初始化。可能通过*延迟加载*的方式来避免初始化，在第一次调用`isRomanNumeral`方法时再进行初始化，但并不推荐。
>“As is often the case with lazy initialization, it would complicate the implementation with no measurable performance improvement ”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

另一种创建不必要对象的方式是`autoboxing`。
>“Autoboxing blurs but does not erase the distinction between primitive and boxed primitive types. ”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 
``` java
// Hideously slow! Can you spot the object creation?
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i;

    return sum;
}
```
>“This program gets the right answer, but it is much slower than it should be, due to a one-character typographical error. The variable sum is declared as a Long instead of a long, which means that the program constructs about 231 unnecessary Long instances (roughly one for each time the long i is added to the Long sum). Changing the declaration of sum from Long to long reduces the runtime from 6.3 seconds to 0.59 seconds on my machine. The lesson is clear: prefer primitives to boxed primitives, and watch out for unintentional autoboxing.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

-- 20190127

### 7.消除过期的对象引用
Java的垃圾回收机制会容易留下不需要思考内存管理的印象，但这不是完全正确的。

#### Object manages its own memery
``` java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * Ensure space for at least one more element, roughly
     * doubling the capacity each time the array needs to grow.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
看起来没有问题，也能通过所有测试，但有问题潜伏着。不准确的说，这个程序有内存泄露的问题
>“In extreme cases, such memory leaks can cause disk paging and even program failure with an OutOfMemoryError, but such failures are relatively rare.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

问题点在于：当一个栈增长然后缩小时，那些被pop出的对象将不会被垃圾回收，即时程序使用的栈没有更多引用指向他们。

>“The fix for this sort of problem is simple: null out references once they become obsolete.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

``` java 
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```
使过期引用无效化的另一个好处是如果虽有被错误地取消引用，程序将立即随着`NullPointerException`而失败，而不是安静地做着错误的事。能尽快查出程序的问题。

经历了没有无效化过期引用导致的问题，可能会在程序结束使用使用对象时尽可能快的过度地无效化所有引用。这也是不可取的。
>“Nulling out object references should be the exception rather than the norm.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

>“Generally speaking, whenever a class manages its own memory, the programmer should be alert for memory leaks. ”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 
无论何时一个元素被释放时，这个元素包含的任何对象引用应该被无效化。

#### Caches
>“Once you put an object reference into a cache, it’s easy to forget that it’s there and leave it in the cache long after it becomes irrelevant.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

有不少解决方案来解决这个问题。
##### 使用 WeakHashMap
>“ If you’re lucky enough to implement a cache for which an entry is relevant exactly so long as there are references to its key outside of the cache, represent the cache as a WeakHashMap; entries will be removed automatically after they become obsolete.Remember that WeakHashMap is useful only if the desired lifetime of cache entries is determined by external references to the key, not the value.”
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

##### 定期清理
>“More commonly, the useful lifetime of a cache entry is less well defined, with entries becoming less valuable over time. Under these circumstances, the cache should occasionally be cleansed of entries that have fallen into disuse. This can be done by a background thread (perhaps a ScheduledThreadPoolExecutor) or as a side effect of adding new entries to the cache. The LinkedHashMap class facilitates the latter approach with its removeEldestEntry method. ”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

##### 直接使用 java.lang.ref

#### 监听者和其他回调
>“If you implement an API where clients register callbacks but don’t deregister them explicitly, they will accumulate unless you take some action. One way to ensure that callbacks are garbage collected promptly is to store only weak references to them, for instance, by storing them only as keys in a WeakHashMap.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

由于内存泄露不能被明显的发现，它们可能在系统中存在多年。也只会在仔细的代码审查或者用调试工具帮助下查看堆。因此，学会在问题发生之前预测这类问题并阻止他们发生是非常有必要的。

### 8.避免使用终结方法和清除方法
>“Finalizers are unpredictable, often dangerous, and generally unnecessary. ”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

在Java9中，finalizers已经被弃用，但它们仍旧被java库所使用。在Java9中代替它的是*cleaner*。

>“Cleaners are less dangerous than finalizers, but still unpredictable, slow, and generally unnecessary.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

#### 缺点

延后完成不只是理论上的问题。类提供的`finalizer`会延迟回收它的实例。
>“Unfortunately, the finalizer thread was running at a lower priority than another application thread, so objects weren’t getting finalized at the rate they became eligible for finalization.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

>“Cleaners are a bit better than finalizers in this regard because class authors have control over their own cleaner threads, but cleaners still run in the background, under the control of the garbage collector, so there can be no guarantee of prompt cleaning.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

>“As a consequence, you should never depend on a finalizer or cleaner to update persistent state.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

>“There is a severe performance penalty for using finalizers and cleaners. ”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

>“Finalizers have a serious security problem: they open your class up to finalizer attacks.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

>“To protect nonfinal classes from finalizer attacks, write a final finalize method that does nothing.”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

>“So what should you do instead of writing a finalizer or cleaner for a class whose objects encapsulate resources that require termination, such as files or threads? Just have your class implement AutoCloseable, and require its clients to invoke the close method on each instance when it is no longer needed, typically using try-with-resources to ensure termination even in the face of exceptions ”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

#### 好处
>“One is to act as a safety net in case the owner of a resource neglects to call its close method. ”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

>“A second legitimate use of cleaners concerns objects with native peers. A native peer is a native (non-Java) object to which a normal object delegates via native methods. ”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

### 9.try-with-resources比try-finally更好
Java类库中含有很多通过执行`close`方法必须手动关闭的资源。如`InputStream`，`OutputStream`,和`java.sql.Connection`。
以往，用`try-finally`的结构可以保证资源将被正确地关闭，即使出现例外。
``` java
// try-finally - No longer the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```
但当它需要增加第二个资源时，会变得糟糕起来。
``` java
try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
```
用`try-finally`的格式，即使正确的代码来关闭资源（上述例子中），依旧有细微的缺点。
如果因为下层物理硬盘的问题，调用`read`时失败了，在调用`close`方法时也失败了，这是第一次异常的信息会被清除。

这些问题使用`try-with-resource`就可以解决。
>“To be usable with this construct, a resource must implement the AutoCloseable interface, which consists of a single void-returning close method. ”
>
>摘录来自: Joshua Bloch. “Effective Java, Third Edition。” Apple Books. 

``` java 
// try-with-resources - the the best way to close resources! 
// improved first example
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(
           new FileReader(path))) {
       return br.readLine();
    }
}

// try-with-resources on multiple resources - short and sweet
// improved second example
static void copy(String src, String dst) throws IOException {
    try (InputStream   in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```

以下版本的`firstLineOfFile`不会抛出异常，但会返回一个默认值如果不能打开或读取文件
``` java 
// try-with-resources with a catch clause
static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(
           new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

