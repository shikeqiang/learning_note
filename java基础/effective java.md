# 1. 考虑使用静态工厂方法替代构造方法

​	一个类允许客户端获取其实例的传统方式是提供一个公共构造方法。 其实还有另一种技术应该成为每个程序员工具箱的一部分。 一个类可以提供一个公共静态工厂方法，它只是一个返回类实例的静态方法。 下面是一个 `Boolean` 简单的例子（`boolean` 基本类型的包装类）。 此方法将 `boolean` 基本类型转换为 `Boolean` 对象引用：

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

​	类可以为其客户端提供静态工厂方法，而不是公共构造方法。提供静态工厂方法而不是公共构造方法有优点也有缺点。

## 优点：

​	**静态工厂方法的一个优点是，可以自定义名字。** 如果构造方法的参数本身并不描述被返回的对象，则具有精心选择名称的静态工厂更易于使用，并且生成的客户端代码更易于阅读。 例如，返回一个可能为素数的 `BigInteger` 的构造方法 `BigInteger(int，int，Random)` 可以更好地表示为名为 `BigInteger.probablePrime` 的静态工厂方法。 

​	**静态工厂方法的第二个优点是，与构造方法不同，它们不需要每次调用时都创建一个新对象。** 这允许不可变的类 （详见第 17 条）使用预先构建的实例，或者在构造时缓存实例，并反复分配它们以避免创建不必要的重复对象。`Boolean.valueof(boolean)` 方法说明了这种方法：它从不创建对象。这种技术类似于 `Flyweight` 模式。如果经常请求等价对象，那么它可以极大地提高性能，特别是如果在创建它们非常昂贵的情况下。

　　静态工厂方法从重复调用返回相同对象的能力允许类保持在任何时候存在的实例的严格控制。这样做的类被称为实例控制（instance-controlled）。编写实例控制类的原因有很多。实例控制允许一个类来保证它是一个单例（详见第 3 条）项或不可实例化的（详见第 4 条）。同时,它允许一个不可变的值类（详见第 17 条）保证不存在两个相同的实例：当且仅当 `a == b` 时 `a.equals(b)`。这是享元模式的基础[Gamma95]。`Enum` 类型（详见第 34 条）提供了这个保证。

　　**静态工厂方法的第三个优点是，与构造方法不同，它们可以返回其返回类型的任何子类型的对象。**

​	**静态工厂的第四个优点是返回对象的类可以根据输入参数的不同而不同。** 声明的返回类型的任何子类都是允许的。 返回对象的类也可以随每次发布而不同。

　　`EnumSet` 类（详见第 36 条）没有公共构造方法，只有静态工厂。 在 OpenJDK 实现中，它们根据底层枚举类型的大小返回两个子类中的一个的实例：如果大多数枚举类型具有 64 个或更少的元素，静态工厂将返回一个 `RegularEnumSet` 实例， 返回一个 `long` 类型；如果枚举类型具有六十五个或更多元素，则工厂将返回一个 `JumboEnumSet` 实例，返回一个 `long` 类型的数组。

## 缺点：

​	**只提供静态工厂方法的主要限制是，没有公共或受保护构造方法的类不能被子类化。**

​	**静态工厂方法的第二个缺点是，程序员很难找到它们，没有专门的API文档。**

下面是一些静态工厂方法的常用名称。以下清单并非完整：

- from —— A 类型转换方法，它接受单个参数并返回此类型的相应实例，例如：**Date d = Date.from(instant)**;
- of —— 一个聚合方法，接受多个参数并返回该类型的实例，并把他们合并在一起，例如：**Set faceCards = EnumSet.of(JACK, QUEEN, KING)**;
- valueOf —— from 和 to 更为详细的替代 方式，例如：**BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE)**;
- instance 或 getinstance —— 返回一个由其参数 (如果有的话) 描述的实例，但不能说它具有相同的值，例如：**StackWalker luke = StackWalker.getInstance(options)**;
- create 或 newInstance —— 与 instance 或 getInstance 类似，除了该方法保证每个调用返回一个新的实例，例如：**Object newArray = Array.newInstance(classObject, arrayLen)**;
- getType —— 与 getInstance 类似，但是如果在工厂方法中不同的类中使用。**Type** 是工厂方法返回的对象类型，例如：**FileStore fs = Files.getFileStore(path)**;
- newType —— 与 newInstance 类似，但是如果在工厂方法中不同的类中使用。Type 是工厂方法返回的对象类型，例如：**BufferedReader br = Files.newBufferedReader(path)**;
- type —— getType 和 newType 简洁的替代方式，例如：**List litany = Collections.list(legacyLitany)**;

　　总之，静态工厂方法和公共构造方法都有它们的用途，并且了解它们的相对优点是值得的。通常，静态工厂更可取，因此避免在没有考虑静态工厂的情况下提供公共构造方法。



















参照：<http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/>