# 优先选择原生类型而非包装类型

Java的类型系统由2部分组成，一个是原生类型，如：`int`，`double`和`boolean`；一个是引用类型，如：`String`和`List`。每一个原生类型都有一个对应的引用类型，称作包装类型。  `int `，` double `和` boolean`对应的包装类型是`Integer`，`Double`和`Boolean`。

正如条款6中提到的，自动装箱和自动拆箱使得原生类型和包装类型之间的区别变得模糊，但并没有擦除它们之间的区别。这两者之间是有真正区别的，重要的是你要时刻留意你用的是哪一种，并在两者之间仔细选择。

原生类型和包装类型的区别只要有3点。首先，原生类型只有它的值，然而包装后的原生类型除了它的值之外，还有其它的标识。换句话说，2个包装类型实例可以有相同的值和不同的标识。第二，原生类型仅仅有一个功能值，然而每一个包装类型除了对应原生类型的功能值外，还有一个非功能值——`null`。最后，原生类型比包装类型更节省时间和空间。如果你不小心的话，这三种差异会给你带来真正的麻烦。

考虑下面的比较器，它设计的目的是对`Integer`类型的值进行升序排序。回想一下，`Comparator`的`compare`方法返回一个负数、零或正数，具体取决于它的第一个参数是小于、等于还是大于第二个参数。在实践中你不需要编写这个比较器，因为它实现了整数的自然排序，但它是一个有趣的例子：

```java
// Broken comparator - can you spot the flaw?
Comparator<Integer> naturalOrder = (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```

这个比较器看起来应该可以工作，它将通过许多测试。比如，它可以和`Collections`的`sort`方法一起使用，正确地对一个百万元素的列表进行排序，无论这个列表是否包含重复元素。但这个比较器存在严重缺陷。为了让你相信，仅仅打印`naturalOrder.compare(new Integer(42), new Integer(42)) `的值。这两个整数实例表示相同的值（42），因此这个表达式的值应该是0，但它是1，这表明第一个整数值大于第二个整数值！

那么问题是什么呢？`naturalOrder`中的第一个测试工作正常。对表达式`i < j`求值会使`i`和`j`引用的`Integer`实例自动拆箱；也就是说，它提取了它们的原生值。然后表达式求值就是检查第一个整数值是否小于第二个整数值。但假设它不是。然后下一个测试计算表达式`i==j`，它是对2个对象引用进行了一致性比较。如果`i`和`j`引用了相同整数值的不同`Integer`实例，那么这个比较会返回`false`，然后这个比较器会错误地返回1，表示第一个`Integer`的值大于第二个。**对包装类型使用`==`操作符几乎总是错误的**。

在实践中，你需要一个比较器来描述类型的自然排序，你应该直接调用` Comparator.naturalOrder() `。如果你自己写一个比较器，那么你应该使用比较器的共造方法，或者对原生类型使用静态的比较方法（条款14）。也就是说，你可以通过添加两个局部变量来存储与包装类型`Integer`参数相对应的原生的`int`值，并对这些变量执行所有的比较，从而修复损坏的比较器中的问题。这可以避免错误地相等性比较：

```java
Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
	int i = iBoxed, j = jBoxed; // Auto-unboxing
	return i < j ? -1 : (i == j ? 0 : 1);
};
```

接下来，考虑一下这个有趣的小程序：

```java
public class Unbelievable {
	static Integer i;
	public static void main(String[] args) {
		if (i == 42)
			System.out.println("Unbelievable");
	}
}
```

是的，它不会打印`Unbelievable`，但是它打印的结果还是很奇怪。它在计算表达式`i == 42`时抛出`NullPointerException`。问题就在于，`i`是`Integer`，而不是`int`的，而且像所有的非常量的对象引用字段一样，它的初始值是`null`。

当程序计算表达式`i == 42`时，它是在比较`Integere`和`int`。在几乎所有情况下，**当你把原生类型和该原生类型的包装类型放在一个操作中混合使用时，该包装类型会自动拆箱**。如果一个空对象引用自动拆箱，那么你将得到一个`NullPointerException`。正如这个程序所展示的，他几乎可以在任何地方发生。修复这个问题非常简单，只需将`i`声明为`int`而不是`Integer`。

最后，考虑条款6里的第24页中的程序：

```java
// Hideously slow program! Can you spot the object creation?
public static void main(String[] args) {
	Long sum = 0L;
	for (long i = 0; i < Integer.MAX_VALUE; i++) {
		sum += i;
	}
	System.out.println(sum);
}
```

这个程序比它本来的速度慢得多，因为它意外地将一个局部变量`sum`声明为包装类型`Long`，而不是原生类型`long`。程序编译时没有错误或警告，该变量被反复地拆箱和装箱，导致明显的性能下降。

在本条款中讨论三个程序中，问题都是一样的：程序员忽略了原生类型和包装类型之间的区别，并承受了后果。在前两个程序中，后果就是是彻底的失败；在第三个，是严重的性能问题。