## 第三章 普适于所有对象的方法
### 10. 覆盖equals时请遵守通用约定
重写`equals`方法看起来简单，但有很多方式会出错，而且后果很严重。最简单避免问题的方法是不要去重写`equals`方法，在每个类的实例只和自己相等的情况下。
#### 如果符合下列的任何条款，是正确的做法
1. 每个类的实例本质上是唯一的
>“Each instance of the class is inherently unique. This is true for classes such as Thread that represent active entities rather than values. The equals implementation provided by Object has exactly the right behavior for these classes.”

2. 类不需要提供一个“逻辑相等的测试”
>“There is no need for the class to provide a “logical equality” test. For example, java.util.regex.Pattern could have overridden equals to check whether two Pattern instances represented exactly the same regular expression, but the designers didn’t think that clients would need or want this functionality. Under these circumstances, the equals implementation inherited from Object is ideal.”

3. 父类已经重写了`equals`，父类的行为也适用于这个子类。
>“A superclass has already overridden equals, and the superclass behavior is appropriate for this class. For example, most Set implementations inherit their equals implementation from AbstractSet, List implementations from AbstractList, and Map implementations from AbstractMap.”

4. 类是私有的或包私有的，那么你可以确认它的`equals`方法将永远不会被调用
>“The class is private or package-private, and you are certain that its equals method will never be invoked. If you are extremely risk-averse, you can override the equals method to ensure that it isn’t invoked accidentally”
``` java 
@Override public boolean equals(Object o) {
    throw new AssertionError(); // Method is never called
}
```

#### 所以什么情况下适合重写`equals`
当一个类有一个与纯粹的对象标识有所不同的逻辑相等的概念，并且父类没有重写`equals`。
>“It is when a class has a notion of logical equality that differs from mere object identity and a superclass has not already overridden equals. ”
*value classes*通常是这种情况。值类只是表示值的类，例如`Integer`或者`String`。

对于值类，通常使用`equals`是为了比较逻辑上是否相等，而不是是否引用同一个对象。

有一种值类不需要重写`equals`方法是使用实例控制来保证只有一个对象存在。Enum类型归于这一类。对于这些类，逻辑相同和对象一致是一样的。

#### 重写`equals`时需要遵循的公约
`equals`方法实现了一种*equivalence relation*。它有这些属性：
* *Reflexive*：对于任何非空引用值`x`，`x.equals(x)`必须返回`true`
* *Symmetric*：对于任何非空引用值`x`和`y`，当且仅当`y.equals(x)`返回`true`时，`x.equals(y)`必须返回`true`
* *Transitive*：对于任何费控引用值`x`，`y`，`z`，如果`x.equals(y)`返回`true`，并且`y.equals(z)`返回`true`，那么`x.equals(z)`必须返回`true`
* *Consistent*：对于任何非空引用值`x`和`y`，对此调用`x.equals(y)`必须一致地返回`true`或者一致地返回`false`，前提是不修改`equals`比较中使用的信息
* 对于任何非空对象`x`，`x.equals(null)`必须返回`false`

用*John Donne*的话来说，没有一个类是孤岛，类的实例经常被传递给另一个类，而这个的基础是遵守`equals`约定。虽然看起来很可怕，但如果不遵守，会让程序变得不稳定或直接崩溃，而且很难进行定位错误的代码。

#### 细说公约
*equivalence relation*简单地说是一个将一组元素分割成元素相等的子集的操作符。这些子集被认为是*equivalence classes*。

##### Reflexivity
一个对象必须等于它自己。很难想象违背了会发生什么。
##### Symmetry
任何两个对象必须对于它们是否相等达成一致。

一旦你违反了这个约定，你就不知道当面对你的对象时，其他对象将会有什么表现。

##### Transitivity
如果 a == b 并且 b == c， 那么a == c。

>“There is no way to extend an instantiable class and add a value component while preserving the equals contract, unless you’re willing to forgo the benefits of object-oriented abstraction.”

java.sql.Timestamp类扩展了java.util.Date，Timestamp的`equals`实现违反了对称性，如果Timestamp和Date对象都在相同的集合中被使用或者混在一起，会造成难以预测的结果。

##### Consistency
如果两个对象相等，他们必须一致保持相等，除非某一方或者全部被修改了。
换句话说，可变的对象在任何时候可以和任何对象相等，但是不可变对象不行。
>“Whether or not a class is immutable, do not write an equals method that depends on unreliable resources.”

##### Non-nullity
所有对象都与`null`不相等。

#### 高质量`equals`方法诀窍
1. 使用==操作符来检查参数是否是该对象的引用。这只是一种性能优化。
2. 使用`instanceof`操作符来检查参数是否具有正确的类型。
3. 将参数转化成正确的类型。因为有`instanceof`测试，所以保证会成功
4. 对垒中每个重要的字段，检查岑石是否与该对象的相应字段匹配。

对于非`float`或`double`的原始类型字段，使用`==`操作符来比较
对于对象引用字段，递归调用`equals`方法
对于`float`字段，使用`Float.compare(float, float)`方法
对于`double`字段，使用`Double.compare(double.double)`方法。
特殊对待`float`和`double`是有必要的，因为存在`Float.Nan`和`-0.0f`，`double`也是相同的原因。

>“ While you could compare float and double fields with the static methods Float.equals and Double.equals, this would entail autoboxing on every comparison, which would have poor performance. ”

>“Some object reference fields may legitimately contain null. To avoid the possibility of a NullPointerException, check such fields for equality using the static method Objects.equals(Object, Object).”

>“The performance of the equals method may be affected by the order in which fields are compared. For best performance, you should first compare fields that are more likely to differ, less expensive to compare, or, ideally, both. You must not compare fields that are not part of an object’s logical state, such as lock fields used to synchronize operations. ”

>“When you are finished writing your equals method, ask yourself three questions: Is it symmetric? Is it transitive? Is it consistent?”

#### 一些最后的警告
* Always override **hashCode** when you override **equals**.
* Don't try to be too clever. 
* Don't subsitute another type of Object in the quals declaration
``` java 
// Broken - parameter type must be Object!
public boolean equals(MyClass o) {
...
}
```

### 11. 覆盖equals时总要覆盖hashCode
>“You must override hashCode in every class that overrides equals.”

>"equal objects must have equal hash codes."

> If two objects are equal according to the `equals(Object)` method, then calling `hashCode` on the two objects must produce the same integer result.

如果不修改`hashCode`方法，对于两个不同的实例，可能在逻辑上是`equals`，但是对象的`hashCode`方法的返回值是两个。

> A good hash function tends to produce unequal hash codes for unequal instances.

>“Luckily it’s not too hard to achieve a fair approximation. Here is a simple recipe:
>1. Declare an int variable named result, and initialize it to the hash code c for the first significant field in your object, as computed in step 2.a. (Recall from Item 10 that a significant field is a field that affects equals comparisons.)
>2. For every remaining significant field f in your object, do the following:
> a. Compute an int hash code c for the field:
>>  i. If the field is of a primitive type, compute Type.hashCode(f), where Type is the boxed primitive class corresponding to f’s type.
>>  ii. If the field is an object reference and this class’s equals method compares the field by recursively invoking equals, recursively invoke hashCode on the field. If a more complex comparison is required, compute a “canonical representation” for this field and invoke hashCode on the canonical representation. If the value of the field is null, use 0 (or some other constant, but 0 is traditional).
>>  iii. If the field is an array, treat it as if each significant element were a separate field. That is, compute a hash code for each significant element by applying these rules recursively, and combine the values per step 2.b. If the array has no significant elements, use a constant, preferably not 0. If all elements are significant, use Arrays.hashCode.
>>  
>b. Combine the hash code c computed in step 2.a into result as follows:
result = 31 * result + c;
>3. Return result.”

> You may exclude *derived field* from the hash code computation.

> You *must* exclude any fields that are not used in `equals` comparisons, or you risk violation the secode provision of the `hashCode` contract.

``` java
// Typical hashCode method
@Override public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}

// One-line hashCode method - mediocre performance
@Override public int hashCode() {
   return Objects.hash(lineNum, prefix, areaCode);
}

// hashCode method with lazily initialized cached hash code
private int hashCode; // Automatically initialized to 0

@Override public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}

```

三个版本：
经典版本：使用乘法的方式逐个处理需要取hash值的字段
单行方法版本：使用`Objects`中的`hash`方法，底层原理和经典版本相同
缓存版本： 第一次调用时，生成hashCode并进行缓存，之后无需重新计算，直接返回即可。

> Do not be tempted to exclude significant fields from the hash code computation to improve performance.

计算hash值的方法可能跑得快了，但会让hash table的性能恶化值不可用的状态。

> Don't provide a detailed specification for the value returned by **hashCode**, soclients can't reasonably depend on it; the gives you the flexibility to change it.


### 12. 每次都覆盖`toString`
> providing a good toString implementation makes your class much more pleasant to use and makes systems using the class easier to debug.

默认的toString方法的实现由类名字+@+hashCode的未签名16进制部分构成。
这样的值是简单和易读的，但是和其他相同对象比较时不能提供有效的信息。

> It is recommended that all subclasses override this method

`toString`方法在对象传入`println`，`printf`，时，和String相关联操作时，或者`assert`，或者debugger打印时。即使你从来没有调用对象的`toString`，其他的对象可能会。

> When practival, the toString method should return *all* of the interesting information contained in the object.

> Whether or not you decide to specify the format, you should clearly document your intentions.

``` java
/**
 * Returns the string representation of this phone number.
 * The string consists of twelve characters whose format is
 * "XXX-YYY-ZZZZ", where XXX is the area code, YYY is the
 * prefix, and ZZZZ is the line number. Each of the capital
 * letters represents a single decimal digit.
 *
 * If any of the three parts of this phone number is too small
 * to fill up its field, the field is padded with leading zeros.
 * For example, if the value of the line number is 123, the last
 * four characters of the string representation will be "0123".
 */
@Override public String toString() {
    return String.format("%03d-%03d-%04d",
            areaCode, prefix, lineNum);
}
```

> Whether or not you specify the format, provide programmatic access to the information contained in the value returned by toString.

> It makes no sense to write a toString method in a static utility class. Nor should you write a `toString` methond in most enum types beacause Java provides a perfectly good one for you.
> You should, however, write a `toString` method in any abstract class whose subclasses share a common string representation.

> To recap, override `Object`'s `toString` implementation in every instantiable class you write, unless a superclass has already done so. It makes classes much more pleasant to use and aids in debugging. The `toString` method should return a concise, useful description of the object, in an aesthetically pleasing format.

### 13. 谨慎地覆盖clone
> The `Cloneable` interface was intended as a mixin interface for classes to advertise that they permit cloning.
> Its primary flaw is that it lacks a `clone` methond, and `Object`'s `clone`method is protected.

>It determines the behavior of `Object`'s protected `clone` implementation: if a class implements `Cloneable`, `Object`'s `clone` method returns a field-by-field coly of the object; otherwise it throws `CloneNotSupportedException`. 

> In practice, a class implementing Cloneable is expected to provide a properly functioning public clone method.

>The general contract for the `clone` method is weak. Here it is , copied from the `Object` specification:
>Creates and returns a copy of this object. The precise meaning of "copy" may depend on the class of the object. The general intent is that, for any object `x`, the expression.
>x.clone() != x
>will be true, and the expression
>x.clone().getClass() == x.getClass()
>will be true, but these are not absolute requirements. While it is typically the case that 
>x.clone().equals(x)
>will be true, this is not an absolute requirement.

``` 
没看明白，后续继续看
```

### 14. 考虑实现Comparable
> The `compareTo` method is not declared in `Object`. Rather, it is the sole method in the `Comparable` interface.

> except that it permits order comparisons in addition to simple equality comparisons, and it is generic(通用的).
> By implementing `Comparable`, a class indicates(暗示) that its instances have a *natural ordering*(自然顺序). 

> By implementing `Comparable`, you allow your class to interoperate with all of the mayny generic algorithms and collection implementations that depend on this interface.

> If you are writing a value class with an obvious natural ordering, such as alphabetical(按字母顺序) order, numerical(按数字顺序) order, or chronological(按时间先后顺序) order, you should implement the `Comparable` interface

 ``` java 
public interface Comparable<T> {
    int comparaeTo(T t);
}
```

>The general contact of the compareTo method is similar to that of `equals`:
>Compares this object with the specified object for order.Returns a **negative integer**, **zero**, or a **positive integer** as this object is **less than**, **equal to**, or **greater than** the specified object. Throws `ClassCastException` if the specified object's type prevents it from being compared to this object.

在下面的描述中，`sgn(expression)`表示数学中的符号函数，被定义为：根据传入表达式的值是负数、零或正数，对应返回-1、0或1

> The implementor must ensure that `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))` for all `x` and `y`. (This implies that `x.compareTo(y)` must throw an exception of and only if `y.compareTo(x)` throws an exception.)
> The implementor must also ensure that the relation is thansitive: `(x.compareTo(y) > 0 && y.compareTo(z) > 0)` implies `x.compareTo(z) > 0`.
> Finally, the implemetor must ensure that `x.compareTo(y) == 0` implies that `agn(x.compareTo(z)) == sgn(y.compareTo(z))`, for all `z`.
> It is strongly recommended, but not required, that `(x.compareTo(y) == 0) == (x.equals(y))`. Generally speaking, any class that implements the `Comparable` interface and violates this condition should clearly indicate this fact. The recommened language is "Note: This class has a natural ordering that is inconsistent with `equals`."（强烈建议`(x.compareTo(y) == 0) == (x.equals(y))`成立，但不是必须的，一般兰说，任何实现Comparable接口并违反此条件的类都应该清楚地注明这个事实。推荐使用的表述是“注意：该类的自然顺序与equals不一致。”）

> `compareTo` doesn't have to work across objects of different types: when confrinted with objects of different types, `compareTo` is permitted to throw `ClassCastException`.

如同`hashCode`的约定，如果打破了，将破坏其他依赖计算哈希值的使用，一个类违反了`compareTo`约点，会弄坏其他依赖比较的类。
> Classes that depend on comparison include the sorted collections `TreeSet` and `TreeMap` and the utility classes `Collections` and `Arrays`, which contain searching and sorting algorithms.

这三个规定的一个结果是：`compareTo`方法所施加的相等性测试必须遵守同等条件所施加的相同限制：`reflexivity`，`symmetry`和`transitivity`。
因此，会出现相同的警告：除非你将放弃面向对象的抽象的好处，否则无法保留`compareTo`约定的情况下使用继承一个可实例化的类同时新增一个字段。

如果想加一个字段到实现了`Comparable`的类，不要继承它；另外写一个没有关联的类包含上一个类的实例对象。

举个例子，`BigDecimal`类，它的`compareTo`方法和`equals`是不一致的。
如果创建一个空的`HashSet`实例，然后`new BigDecimal("1.0")`和`new BigDecimal("1.00")`，这个集合中将包含两个元素。因为两个`BigDecimal`实例被添加到集合中是不相等的（使用`equals`方法比较）。
如果使用`TreeSet`执行相同的步骤，这个结合只会包含一个元素，因为两个`BigDecimal`实例是相等的（通过`compareTo`方法比较）。

> To compare object reference fields, invoke the compareTo method recursively. If a field does not implement Comparable or you need a nonstandard ordering, use a Comparator instead. 

``` java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITVE_ORDER.compare(s. cis.s);
    }
    
    ... // Reminder omitted
}
```

注意到`CaseInsensitiveString`引用只能被其他`CaseInsensitiveString`引用比较。

Java7中，静态方法`compare`已经被添加到所有Java的`boxed primitive class`.

> Use of the relational operators < and > in compareTo methods is verbose and error-prone and no longer recommended.

如果一个类有多个重要的字段，你比较的顺序就比较关键了。
从最重要的字段开始，逐个校验。
`PhoneNumber`类的`compareTo`方法：

``` java 
// Multiple-field Comparable with primitive fields
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0) {
        result = Short.compare(prefix, pn.prefix);
        if (result == 0) {
            result = Short.compare(lineNum, pn.lineNum);
        }   
    }
    return result;
}
```

在Java8中，`Comparator`接口配备了一组比较构造函数，能流畅地构造比较器。这些比较器可以在实现`compareTo`方法时被使用。
许多程序更倾向考虑这种方式，因为它带来良好的性能开销。
当使用这种方式时，请考虑使用Java的*静态导入*工具，以便您可以通过简单的名称来引用静态比较器构造方法，以简化和简洁。

``` java
// Comparable with comparator construction methods
private static final Comparator<PhoneNumber> COMPARATOR = comparingInt((PhoneNumber pn) -> pn.areaCode).thenComparingInt(pn -> prefix).thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```
这种实现方式在类初始化时创建了一个比较器，使用了两个比较构造函数。
第一种是`comparingInt`。
第二种是`themComparingInt`。

有和`comparingInt`和`thenComparingInt`类似的给基本类型`long`和`double`的方法。`int`版本也可以给短整型使用，如`short`。`double`版本也可以被`float`使用。这就覆盖了所有Java的数字的基本类型。

名称为`comparing`的静态方法优良中重载方式
1. 需要key的提取器，然后使用key的自然排序
2. 需要一个key的提取器和一个用来比较获取的key的比较器

叫`thenComparing`方法有三种重载实例方法：
1. 只需要一个比较器，用来提供次要的排序方式
2. 只需要一个key的提取器，使用key的自然排序作为次要排序
3. 同时需要key的提取器和用来比较提取的key的比较器

#### Summary
无论何时你实现一个需要有排序性质的值类，你应该实现`Comparable`接口，让它的实例可以被轻易地排序，搜索和在基于比较的集合中使用。在`compareTo`方法的视线中比较字段值时，避免使用`<`和`>`操作符，应该使用包装类中的静态比较方法或`Comparator`接口中的比较构造方法。












