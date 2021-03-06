《《《 [返回首页](../README.md)       <br/>
《《《 [上一节](00_Comparison_and_Bounds.md)

### 可比较的

接口 `Comparable <T>` 包含一个可用于比较一个对象与另一个对象的方法：

```java
interface Comparable<T> {
  public int compareTo(T o);
}
```

根据参数是小于，等于还是大于给定对象，`compareTo` 方法返回一个负值，零值或正值。 当一个类实现 `Comparable` 时，由该接口指定的顺序被称为该类的自然顺
序。

通常，属于类的对象只能与属于同一类的对象进行比较。 例如，`Integer` 实现了 `Comparable <Integer>`：

```java
Integer int0 = 0;
Integer int1 = 1;
assert int0.compareTo(int1) < 0;
```

比较返回一个负数，因为 `0` 在数字顺序下先于 `1`。 同样，`String` 实现了 `Comparable <String>`：

```java
String str0 = "zero";
String str1 = "one";
assert str0.compareTo(str1) > 0;
```

这个比较返回一个正数，因为在字母顺序下“零”跟在“一”之后。

接口的类型参数允许在编译时捕获无意义的比较：

```java
Integer i = 0;
String s = "one";
assert i.compareTo(s) < 0; // 编译错误
```

您可以将一个整数与一个整数或一个字符串与一个字符串进行比较，但试图将一个整数与一个字符串进行比较会发出编译时错误。

任意数字类型之间不支持比较：

```java
Number m = new Integer(2);
Number n = new Double(3.14);
assert m.compareTo(n) < 0; // 编译错误
```

这里的比较是非法的，因为 `Number` 类没有实现 `Comparable` 接口。

通常使用 `equals`，我们要求两个对象相等，当且仅当它们相同时：

```java
x.equals(y) if and only if x.compareTo(y) == 0
```

在这种情况下，我们说自然顺序与平等一致。

建议您在设计课程时选择与平等一致的自然排序。如果在集合框架中使用接口 `SortedSet` 或 `SortedMap`，这两个函数使用自然排序来比较项目，这一点尤其重要。
如果两个比较相同的项目被添加到一个有序集合中，则只有一个将被存储，即使这两个项目不相等;排序地图也是如此。 （您也可以使用第 `3.4` 节中描述的比较器指定
不同的排序以用于排序集合或排序映射;但在这种情况下，指定排序应该再次与等号一致）。

`Java` 核心库中几乎每个具有自然排序的类都与 `equals` 相一致。 `java.math.BigDecimal` 是一个例外，它将具有相同值但精度不同的两个小数进行比较，例如 
`4.0` 和 `4.00`。 `3.3` 节给出了另一个与自然顺序不相等的类的例子。

比较不同于平等，因为它不接受空参数。 如果 `x` 不为空，则为 `x.equals(null)` 必须返回 `false`，而 `x.compareTo(null)` 必须抛出 
`NullPointerException`。

我们使用标准成语进行比较，写入 `x.compareTo(y)<0` 而不是 `x <y`，并且写入 `x.compareTo(y)<= 0 `而不是 `x <= y`。 使用其中的最后一个，甚至在诸如 
`java.math.BigDecimal之` 类的类型上自然排序与 `equals` 不一致是非常明智的。

**可比契约** `Comparable <T>` 接口的契约指定了三个属性。 这些属性是使用符号函数定义的，函数的定义使得 `sgn(x)` 返回 `-1`,`0` 或 `1`，具体取决于 
`x` 是负数，零还是正数。
        
首先，比较是反对称的。 颠倒参数的顺序颠倒了结果：

```java
sgn(x.compareTo(y)) == -sgn(y.compareTo(x))
```

这概括了数字的属性：`x <y` 当且仅当 `y> x`。 当且仅当 `y.compareTo(x)` 引发异常时，还需要 `x.compareTo(y)` 引发异常。

其次，比较是传递性的。 如果一个值小于第二个，而第二个小于第三个，那么第一个小于第三个：

```java
if x.compareTo(y) < 0 and y.compareTo(z) < 0 then x.compareTo(z) < 0
```

这概括了数字的属性：如果 `x <y` 且 `y <z`，则 `x <z`。

第三，比较是一致的。 如果两个值相同，则它们与第三个值的比较方式相同：

```java
if x.compareTo(y) == 0 then sgn(x.compareTo(z)) == sgn(y.compareTo(z))
```

这概括了数字的属性：如果 `x == y`，那么 `x <z` 当且仅当 `y <z`。 据推测，如果 `x.compareTo(y)== 0`，那么 `x.compareTo(z)` 会引发异常，当且仅当 
`y.compareTo(z)` 引发异常时，也需要这样做，尽管没有明确说明。

强烈建议比较与平等兼容：

```java
x.equals(y) if and only if x.compareTo(y) == 0
```

正如我们前面看到的，一些特殊的类，如 `java.math.BigDecimal` 违反了这个约束。

然而，总是要求比较具有反思性。 每个价值都与自身相同：

```java
x.compareTo（x）== 0
```

这是从第一个要求开始的，因为把 `x` 和 `y` 相同就给了我们`sgn(x.compareTo(x))== -sgn(x.compareTo(x))`。

**注意这一点！** 值得指出的是比较定义中的微妙之处。

这是比较两个整数的正确方法：

```java
class Integer implements Comparable<Integer> {
  ...
  public int compareTo(Integer that) {
    return this.value < that.value ? -1 :
    this.value == that.value ? 0 : 1 ;
  }
...
}
```

条件表达式根据接收方是小于，等于还是大于参数返回 `-1`,`0` 或 `1`。 你可能会认为下面的代码也会起作用，因为如果接收者小于参数，该方法被允许返回任何负
数。

```java
class Integer implements Comparable<Integer> {
...
  public int compareTo(Integer that) {
    // bad implementation -- don't do it this way!
    return this.value - that.value;
  }
...
}
```

但是当溢出时，这段代码可能会给出错误的答案。 例如，当比较大的负值和大的正值时，差异可能会大于可存储在整数 `Integer.MAX_VALUE` 中的最大值。

《《《 [下一节](02_Maximum_of_a_Collection.md)      <br/>
《《《 [返回首页](../README.md)
