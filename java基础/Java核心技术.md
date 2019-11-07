# Java核心技术

## 一、java的基础设计结构

### 1.数据类型

​	Java 是 -种强类型语言。这就意味着必须为每一个变量声明一种类型: 在 Java 中， -共有 **8种基本类型** （ primitive type ), 其中有 4 种整型、 2 种浮点类型、 **1 种用于表示 Unicode 编码的字符单元的字符类型 char**和 1 种用于表示真值的 boolean 类型。 

==注意==：Java 有一个能够表示任意精度的算术包，通常称为“ 大数值”（bignumber。) 虽然被称为**大数值，但它并不是一种新的 Java 类型， 而是一个 Java 对象。**

#### 1.1、整型

​	整型用于表示没有小数部分的数值， 它允许是负数。 Java 提供了 4 种整型，具体内容如表 3-1 所示。![1544750128207](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/1544750128207.png)

**byte 和 short 类型主要用于特定的应用场合， 例如， 底层的文件处理或者需要控制占用存储空间量的大数组。**  	==在 Java 中， 整型的范围与运行 Java 代码的机器无关。== 由于 Java 程序必须保证在所有机器上都能够得到相同的运行结果， 所以**各种数据类型的取值范围必须固定。** 

​	长整型数值有一个后缀 L 或 l( 如 4000000000L。) 十六进制数值有一个前缀 Ox 或 0X (如OxCAFEL)。 八进制有一个前缀 0  。**从 Java 7 开始， 加上前缀 0b 或 0B 就可以写二进制数。** 从 Java 7 开始，还可以为数字字面量加下划线， 如用 1_000_000表示一百万。 Java 编译器会去除这些下划线 。

#### 1.2、浮点类型

​	浮点类型用于表示有小数部分的数值。在 Java 中有两种浮点类型，具体内容如表 3-2 所示。 ![1544750643866](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/1544750643866.png)

​	double 表示这种类型的数值精度是 float 类型的两倍（有人称之为双精度数值)。绝大部分应用程序都采用 double 类型。实际上，只有很少的情况适合使用 float 类型，例如，需要单精度数据的库， 或者需要存储大量数据。**float 类型的数值有一个后缀 F 或 f (例如， 3.14F)。**没有后缀 F 的浮点数值（如 3.14 ) 默认为 double 类型。当然，也可以在浮点数值后面添加后缀 D 或 d (例如， 3.14D)。 

==注意：== 可以使用十六进制表示浮点数值。例如，0.125=2—3 可以表示成 0xl.0p-3。在十六进制表示法中， 使用 p 表示指数， 而不是 e。 **注意， 尾数采用十六进制， 指数采用十进制。**指数的基数是 2， 而不是 10。 

#### 1.3、char类型

​	char 类型原本用于表示单个字符。char 类型的字面量值要用==**单引号**==括起来。 例如： W 是编码值为 65 所对应的字符常量。它与 "A" 不同，"A" 是包含一个字符 A 的字符串, char 类型的值可以表示为十六进制值，其范围从 \u0000 到 \Uffff 。![1544751354753](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/1544751354753.png)

#### 1.4、boolean类型

​	boolean (布尔）类型有两个值： false 和 true, 用来判定逻辑条件。整型值和布尔值之间不能进行相互转换。 

### 2.变量

​	声明一个变量之后，**必须用赋值语句对变量进行显式初始化**， 千万不要使用未初始化的变量。 如：int i=10,而不是int i;然后就直接输出 i 了。

​	在 Java 中， 利用关键字 final 指示常量关键字 final 表示这个变量只能被赋值一次。一旦被赋值之后， 就不能够再更改了**。习惯上,常量名使用全大写 。** 在 Java 中，经常希望某个常量可以在一个类中的多个方法中使用，通常将这些常量称为类常量。可以**使用关键字 static final设置一个类常量。**类常量的定义位于 main方法的外部。 因此， 在同一个类的其他方法中也可以使用这个常量。 而且， 如果一个常量被声明为 **public， 那么其他类的方法也可以使用这个常量。**  

### 3.运算符

​	在 Java 中，使用算术运算符 +、-、 *、 / 表示加、减、 乘、 除运算。 **当参与 / 运算的两个操作数都是整数时， 表示整数除法；否则， 表示浮点除法。** 整数的求余操作（有时称为取模 )用 ％ 表示。 例如， 15/2 等于 ，7 15%2 等于 1, 15.0/2 等于 7.50。**需要注意**， 整数被 0 除将会产生一个异常， 而浮点数被 0 除将会得到无穷大或 NaN 结果。 

#### 3.1、数值类型之间的转换 

![1544754201229](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/1544754201229.png)

6个实心箭头，表示无信息丢失的转换； 有 3 个虚箭头， 表示可能有精度损失的转换。round 方法返回的结果为 long 类型 。 

### 4.字符串

​	**==每个用双引号括起来的字符串都是 String类的一个实例 ，如：String a="a"==**

#### 4.1、子串

String 类的 substring 方法可以从一个较大的字符串提取出一个子串。例如：
​	String greeting = "Hello";
​	String s = greeting.substring(0, 3);
创建了一个由字符“ Hel” 组成的字符串。 即substring(a,b)截取的范围是：[a,b)

#### 4.2、拼接

​	**当将一个字符串与一个非字符串的值进行拼接时，后者被转换成字符串** 

#### 4.3、检测字符串是否相等

​	**==equal()和equalsIgnoreCase()都可以用来比较两个字符串是否相等，区别在于第二个是忽略大小写的。==** 

#### 4.4、==String API==

- char charAt (int index)  返回给定位置的代码单元。除非对底层的代码单元感兴趣， 否则不需要调用这个方法。

-  int codePointAt(int Index)   返回从给定位置开始的码点。

- int offsetByCodePoints(int startlndex, int cpCount)  返回从 startlndex 代码点开始，位移 cpCount 后的码点索引。

-  int compareTo(String other)    按照字典顺序， 如果字符串位于 other 之前， 返回一个负数；如果字符串位于 other 之后，返回一个正数； 如果两个字符串相等，返回 0。

-  IntStream codePoints() 将这个字符串的码点作为一个流返回。调用 toArray 将它们放在一个数组中。

- new String(int[] codePoints, int offset, int count)  用数组中从 offset 开始的 count 个码点构造一个字符串。

- boolean equals(0bject other)    如果字符串与 other 相等， 返回 true。 

- boolean equalsIgnoreCase( String other )   如果字符串与 other 相等 （忽略大小写，) 返回 tme。

- boolean startsWith( String prefix )
  boolean endsWith( String suffix )        如果字符串以 suffix 开头或结尾， 则返回 true。

- int indexOf ( String str ) 

  int indexOf( String str, int fromlndex )
  int indexOf ( int cp)
  int indexOf( int cp, int fromlndex )  返回与字符串 str 或代码点 cp 匹配的第一个子串的开始位置。这个位置从索引 0 或fromlndex 开始计算。 如果在原始串中不存在 str， 返回 -1。

- int 1astIndexOf( String str )
  Int 1astIndexOf ( String str, int fromlndex )
  int lastindexOf( int cp)
  int lastindexOf( int cp, int fromlndex )   返回与字符串 str 或代码点 cp 匹配的最后一个子串的开始位置。 这个位置从原始串尾端或 fromlndex 开始计算。

- int length( )  返回字符串的长度。

- int codePointCount( int startlndex, int endlndex )   返回 startlndex 和 endludex-l 之间的代码点数量。没有配成对的代用字符将计入代码点。

- String replace( CharSequence oldString,CharSequence newString)   返回一个新字符串。这个字符串用newString 代替原始字符串中所有的 oldString。可以用 String 或 StringBuilder 对象作为 CharSequence 参数。

- String substring( int beginlndex )
  String substring(int beginlndex, int endlndex )
  返回一个新字符串。这个字符串包含原始字符串中从 beginlndex 到串尾或 endlndex-l的所有代码单元。

- String toLowerCase( )
  String toUpperCase( )
  返回一个新字符串。 这个字符串将原始字符串中的大写字母改为小写，或者将原始字符串中的所有小写字母改成了大写字母。

- String trim()    返回一个新字符串。这个字符串将删除了原始字符串头部和尾部的空格。

- String join(CharSequence delimiter, CharSequence... elements )    返回一个新字符串， 用给定的定界符连接所有元素。 	

#### 4.5、StringBuilder相关API

![1544757827379](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/1544757827379.png)

### 5.输入输出

#### 5.1	读取输入

​	要想通过控制台进行输人，首先需要构造一个Scanner对象，并与“ 标准输人流”System.in关联。

```Java
Scanner in = new Scanner(System.in);
//nextLine 方法将输入一行。 
//使用 nextLine 方法是因为在输人行中有可能包含空格。 
System.out.print("What is your name? "); 
String name = in.nextLine(); 
//要想读取一个单词(以空白 符作为分隔符 ，) 就调用
String firstName = in.next();
//要想读取一个整数， 就调用 nextlnt 方法。
System.out.print("How old are you? "); 
int age = in.nextlnt();
//与此类似， 要想读取下一个浮点数， 就调用 nextDouble 方法。
```

PS：因为输入是可见的， 所以 Scanner 类不适用于从控制台读取密码。Java SE 6 特别引入了 Console 类实现这个目的。要想读取一个密码， 可以采用下列代码: 

```
Console cons = System.console(); 
Stringusername=cons.readLine("Username: "); 
char[] passwd = cons.readPassword("Password: "); 
```

为了安全起见，返回的密码存放在一维字符数组中， 而不是字符串中。在对密码进行处理之后， 应该马上用一个填充值覆盖数组元素。采用 Console 对象处理输入不如采用 Scanner 方便。每次只能读取一行输入， 而没有能够读取一个单词或一个数值的方法。

##### Scanner相关API：![image-20181219162839610](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181219162839610-5208119.png)

#### 5.2 格式化输出

​	Java中引入了跟C语言一样的格式化输出：System.out.printf("%8.2f", x);可以用 8 个字符的宽度和小数点后两个字符的精度打印 x。 也就是说， 打印输出一个空格和 7 个字符，原来x=3333.333333333335,现在输出的是：3333.33。

在 printf 中， 可以使用多个参数， 例如: System.out.printf("Hello, %s. Next year, you'll be %d", name, age) ; 

每一个以 % 字符开始的格式说明符都用相应的参数替换。 格式说明符尾部的转换符将指示被格式化的数值类型:f 表示浮点数，s 表示字符串，d 表示十进制整数。

##### 下面是printf所有的转换符：![image-20181219164600421](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181219164600421-5209160.png)

另外， 还可以给出控制格式化输出的各种标志。 例如， 逗号 标志增加了分组的分隔符。

即 Systen.out.printf("%x,.2f", 10000.0 / 3.0); 	打印结果：3, 333.33  ，这里的逗号就是打印语句中增加的。

也可以使用多个标志， 例如，“ %，( .2f ” **使用分组的分隔符并将负数括在括号内。**

##### printf相关标志：

![image-20181219164816704](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181219164816704-5209296.png)

#### 5.3 文件输入与输出

​	要想对文件进行读取， 就需要一个用 File 对象构造一个 Scanner 对象， 如下所示: 

​		Scanner in = new Scanner(Paths.get("niyflle.txt"), "UTF-8"); 

**如果文件名中包含反斜杠符号， 就要记住在每个反斜杠之前再加一个额外的反斜杠:** “ c:\\\mydirectory\\\myfile.txt”c

现在， 就可以利用前面介绍的任何一个 Scanner 方法对文件进行读取。 要想写入文件， 就需要构造一个 PrintWriter 对象。 在构造器中， 只需要提供文件名:

 PrintWriter out = new Printlulriter(“myfile.txt", "UTF-8"); 

如果文件不存在， 创建该文件。 可以像输出到 System.out —样使用 print、 println 以及 printf 命令。 

### 6.大数值

​	java.math 包中的两个很有用的类: **Biglnteger 和 BigDecimal，** 这两个类可以处理包含任意长度数字序列的数值。==Biglnteger 类实现了任意精度的整数运算， BigDecimal 实现了任意精度的浮点数运算。==	

​	使用静态的 valueOf 方法可以将普通的数值转换为大数值:

```Java
		Biglnteger a = Biglnteger.valueOf(100);	
```

遗憾的是，不能使用人们熟悉的算术运算符(如: + 和 * ) 处理大数值。 **而需要使用大数值类中的 add 和 multiply 方法。**

```Java
Biglnteger c = a.add(b); // c = a + b
Biglnteger d = c.nultipiy(b.add(Biglnteger.valueOf(2))); // d = c * (b + 2)
```

##### BigInter相关方法：

![image-20181219175107299](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181219175107299-5213067.png)

### 7.数组

##### 	两种声明方式：int[] a 或者 int a[]

​	**创建一个数字数组时， 所有元素都初始化为 0。boolean 数组的元素会初始化为 false。 对象数组的元素则初始化为一个特殊值null, 这表示这些元素(还)未存放任何对象。**	

如：String[] names = new String[10];  **会创建一个包含 10 个字符串的数组 所有字符串都为 null 。**

#### 7.1 for each循环

​	Java 有一种功能很强的循环结构， 可以用来依次处理数组中的每个元素 (其他类型的元素集合亦可)而不必为指定下标值而分心。

​	语句格式：for (variable : collection) statement

​	**定义一个变量用于暂存集合中的每一个元素， 并执行相应的语句(当然， 也可以是语句块)。==collection 这一集合表达式必须是一个数组或者是一个实现了 Iterable 接口的类对象(例如ArrayList )。==** 		

#### 7.2 数组初始化

多种格式：int[] small Primes = { 2, 3, 5, 7, 11, 13 };

​		   new intD { 17, 19, 23, 29, 31, 37 }

​		   small Primes = new int[] { 17, 19, 23, 29, 31, 37 };

 在 Java 中， 允许数组长度为 0。 在编写一个结果为数组的方法时， 如果碰巧结果 为空， 则这种语法形式就显得非常有用。 此时可以创建一个长度为 0 的数组: new elementType[0]

#### 7.3 数组拷贝

​	==在 Java 中， 允许将一个数组变量拷贝给另一个数组变量。这时， 两个变量将引用同一个数组:==

​	int[] luckyNumbers = smallPrimes;
​	luckyNumbers[5] = 12; // now smallPrimes[5] is also 12	 

![image-20181219194821880](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181219194821880-5220101.png)

 **如果希望将一个数组的所有值拷贝到一个新的数组中去，就要使用 Arrays 类的 copyOf 方法:**

```
int[] copiedLuckyNumbers = Arrays.copyOf(luckyNumbers, luckyNumbers.length);
```

第 2 个参数是新数组的长度。 这个方法通常用来增加数组的大小:

```
luckyNumbers = Arrays.copyOf(luckyNumbers, 2 * luckyNumbers.length);
```

**如果数组元素是数值型， 那么多余的元素将被赋值为 0 ; 如果数组元素是布尔型， 则将赋值为 false。 相反， 如果长度小于原始数组的长度， 则只拷贝最前面的数据元素。**

#### 7.4 数组排序

​	要想对数值型数组进行排序， 可以使用 Arrays 类中的 sort 方法:		

```
int[] a = new int[10000];
Arrays.sort(a)
```

**这个方法使用了优化的快速排序算法。快速排序算法对于大多数数据集合来说都是效率比较高的。**

##### 数组相关API：![image-20181219202433472](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181219202433472-5222273.png)

![image-20181219202445636](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181219202445636-5222285.png)

## 二、对象和类

### 1.对象和变量之间的区别

​	如：Date deadline;    	

**定义了一个对象变量 deadline, 它可以引用Date类型的对象。 但是，一定要认识到: 变量 deadline 不是一个对象， 实际上也没有引用对象。 此时， 不能将任何 Date 方法应用于这个变量上。** 

语句 	s = deadline.toString();	将产生编译错误，必须首先初始化变量deadline,有两种方式：

​	==1.用新构造的对象初始化这个变量:== 	deadline = new Date() ; 

​	==2.让这个变量引用一个已存在的对象:== 	Date birthday = new Date();	**deadline = birthday;** 

![image-20181220095316185](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181220095316185-5270796.png)

##### 一个对象变量并没有实际包含一个对象，而仅仅引用一个对象。

在 Java 中， 任何对象变量的值都是对存储在另外一个地方的一个对象的引用。new 操作符的返回值也是一个引用。

### 2.用户自定义类

1.==Java源文件名必须与 public 类的名字相匹配。 在一个源文件中， 只能有一个公有类， 但可以有任意数目的非公有类。==

2.final 修饰符大都应用于基本(primitive) 类型域， 或不可变(immutable) 类的域(如果类中的每个方法都不会改变其对象， 这种类就是不可变的类。 

​	例子：private final StringBuiIcier evaluations;

​		在 Employee 构造器中会初始化为 ：evaluations = new StringBuilder(); 

​	**final 关键字只是表示存储在 evaluations 变量中的对象引用不会再指示其他 StringBuilder对象。** 	

可以修改：

```
public void giveGoldStar()
{ 
	evaluations.append(LocalDate.now() + ": Gold star!\n");
}
```

### 3.静态域与静态方法

##### 1.在下面两种情况下使用静态方法:

- 一个方法不需要访问对象状态， 其所需参数都是通过显式参数提供(例如: Math.pow )
- 一个方法只需要访问类的静态域(例如: Employee.getNextld)

2.对象实例也可以调用静态方法，但是不推荐使用。

### 4.方法参数

**分为两种：按值调用 (call by value) 表示方法接收的是调用者提供的值。而按引用调用 (call by reference)**
**表示方法接收的是调用者提供的变量地址。**

例子(参数是基本类型)：

```
public static void tripieValue(double x) // doesn't work
{
	x = 3 * x;
}

double percent = 10; 
tripieValue(percent) ;
```

结果：percent还是10，而x变成30

![image-20181220172045933](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181220172045933-5297646.png)

执行过程：

​	1 ) x 被初始化为 percent 值的一个拷贝 (也就 是 10 ) 

​	2) x被乘以3后等于30。但是percent仍然 是 10 ( 如图所示。) 

​	3 ) 这个方法结束之后， 参数变量 X 不再使用。 

例子(参数是引用类型)：

```
public static void tripieSalary(Employee x) // works
{
	x.raiseSa1ary(200) ; 
	}
	
harry = new Employee(. . .); 
tripieSalary(harry) ;
```

 结果：harry变大了200倍

![image-20181220172244660](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181220172244660-5297764.png)

执行过程：

​	1 ) X 被初始化为 harry 值的拷贝， 这里是一个对象的引用。
 	2 ) raiseSalary 方法应用于这个对象引用。 x 和 harry 同时引用的那个 Employee 对象的薪金提高了 200%。
​	3 ) 方法结束后， 参数变量 x 不再使用。 当然， 对象变量 harry 继续引用那个薪金增至3倍的雇员对象 (如图 所示 )。 

##### Java 程序设计语言对对象采用的不是引用调用， 实际上， 对象引用是按值传递的。

##### Java中方法参数使用情况：

- **一个方法不能修改一个基本数据类型的参数 (即数值型或布尔型)。**
- **一个方法可以改变一个对象参数的状态。**
- **一个方法不能让对象参数引用一个新的对象。**

说明：因为引用类型传递的是对象的地址，所以只是引用了这个地址，而不是调用，所以不能引用一个新的对象，

​	   当用引用类型参数的时候，只是拷贝了传进来的对象的地址，执行完方法后会不再引用。

### 5.方法构造

**1.如果没有写构造器，系统会默认提供一个无参构造器，这个构造器将所有的实例域设置为默认值。 于是， 实例域中的数值型数据设置为 0、 布尔型数据设置为 false、 所有对象变量将设置为 null。**

**2.在一个构造器里面，可以通过this(…)调用另一个构造器，但是这个this语句要放在构造器的第一行。**

**3.还可以通过初始化块初始化数据，无论调用那个构造器，都会先初始化块里面的代码。**==即首先运行初始化块，再执行构造器主体。==

##### 4.调用构造器的具体步骤：

​	**1 ) 所有数据域被初始化为默认值(0、 false 或 null 。)**
​	**2 ) 按照在类声明中出现的次序， 依次执行所有域初始化语句和初始化块。**	

​	**3 ) 如果构造器第一行调用了第二个构造器， 则执行第二个构造器主体**
​	**4 ) 执行这个构造器的主体.**

## 三、继承

**1.如果子类想调用父类的方法，需要通过super这个关键字调用，即super.methodName()调用。注意，super 不是一个对象的引用， 不能将 super 赋给另一个对象变量， 它只是一个指示编译器调用超类方法的特殊关键字。**

##### 2.this和super作用：

​	**this：一是引用隐式参数 二是调用该类其他的构造器；**

​	**super：一是调用超类的方法， 二是调用超类的构造器。** 

##### 3.使用super调用构造器的语句必须是子类构造器的第一句语句。

4.在 Java 中， 子类数组的引用可以转换成超类数组的引用， 而不需要采用强制类型转换。

##### 	Object 类中的 getClass( ) 方法将会返回一个 Class 类型的实例。

##### 5.方法调用过程：

​	假设调用 x.f(args，) 隐式参数 x 声明为类 C 的一个对象。下面为详细的调用过程：

​	1 ) **编译器査看对象的声明类型和方法名**。假设调用 x.f(param)， 且隐式参数 **x 声明为 C 类的对象**。需要注意的是: ==有可能存在多个名字为f, 但参数类型不一样的方法==。例如，可能存在方法 f(int) 和方法f(String)。 **编译器将会一一列举所有 C 类中名为 f 的方法和其超类中访问属性为 public 且名为 f 的方法 (超类的私有方法不可访问)。** 

​	至此， 编译器已获得所有可能被调用的候选方法。 

2 ) 接下来， 编译器将査看调用方法时提供的参数类型。 如果在所有名为 f 的方法中存在 一个与提供的参数类型完全匹配， 就选择这个方法。这个过程被称为==**重载解析(overloading resolution )。**== 例如， 对于调用 x.f(“ Hello ” )来说， 编译器将会挑选 f(String)， 而不是 f(int) 。由于允许类型转换(int 可以转换成 double, Manager 可以转换成 Employee，等等，) 所以这 个过程可能很复杂。 **如果编译器没有找到与参数类型匹配的方法， 或者发现经过类型转换后 有多个方法与之匹配， 就会报告一个错误。** 

​	至此， 编译器已获得需要调用的方法名字和参数类型。 

6.**如果是private方法、static方法、final方法或者构造器， 那么编译器将可以准确地知道应该调用哪个方法， 我们将这种调用方式称为==静态绑定(static binding )。==** 与此对应的是， 调用的方法依赖于隐式参数的实际类型， 并且在运行时实现动态绑定。如：有多个继承关系的类，并且类中有多个构造方法，调用的时候要判断是哪个构造方法。像上面的第五点那样。如果是用的**super**关键字，那么搜索的是超类的方法。

##### 7.在覆盖一个方法的时候， 子类方法不能低于超类方法的可见性。

##### 8.抽象类

​	包含一个或多个抽象方法的类本身必须被声明为抽象类，抽象类还可以包含具体数据和具体方法，抽象类也可以不含抽象方法，抽象类不能被实例化。

##### 9.修饰符访问范围：

​	**public>protected>default>private**，其中，protected对本包和所有子类可见，default对本包可见。

##### 10.Object：

1.**equals**

​	Object 类中的 equals 方法用于检测一个对象是否等于另外一个对象。在 Object 类中， 这个方法将判断两个对象是否具有相同的引用。如果两个对象具有相同的引用， 它们一定是相等的。

![image-20181221152813435](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181221152813435-5377293.png)

**2.hashCode方法**

​	散列码(hashcode) 是由对象导出的一个整型值。散列码是没有规律的。如果x和y是两个不同的对象， x.hashCode( ) 与 y.hashCode( ) 基本上不会相同。

由于 hashCode 方法定义在 Object 类中， 因此每个对象都有一个默认的散列码， 其值为对象的存储地址。	

例子：

```java
String s = "Ok";
StringBuilder sb = new StringBuilder(s); 
System.out.println(s.hashCode() + " " + sb.hashCode()); 
String t = new String("Ok");
StringBuilder tb = new StringBuilder(1); 
System.out.println(t.hashCode() + " " + tb.hashCode());
```

结果：![image-20181221155257121](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181221155257121-5378777.png)

字符串 s 与 t 拥有相同的散列码， 这是因为字符串的散列码是由内容导出的。 而字符串缓冲 sb 与 tb 却有着不同的散列码， 这是因为在 StringBuffer 类中没有定义hashCode 方法， 它的散列码是由 Object 类的默认 hashCode 方法导出的对象存储地址。

**如果重新定义 equals 方法， 就必须重新定义 hashCode 方法， 以便用户可以将对象插人到散列表中**

hashCode 方法应该返回一个整型数值(也可以是负数)， 并合理地组合实例域的散列码 ,以便能够让各个不同的对象产生的散列码更加均匀。![image-20181221160144364](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181221160144364-5379304.png)

##### 11.泛型数组列表：

 toArray 方法将数组元素拷贝到一个数组中：

```
X[] a = new X[List.size()]; 
list.toArray(a) ;
```

相关API：

• void set(int index，E obj)  设置数组列表指定位置的元素值， 这个操作将覆盖这个位置的原有内容。

​	参数:	 index 位置(必须介于0 ~ size()-l 之间)	obj 新的值

•E get(int index) 	获得指定位置的元素值。 

​	参数: index 获得的元素位置(必须介于 0 ~ size()-l 之间) 

•void add(int index,E obj) 	向后移动元素， 以便插入元素。
 	参数: index 插入位置 (必须介于 0 〜 size()- l 之间) 		obj 新元素 

•E remove(int index) 	删除一个元素， 并将后面的元素向前移动。 被删除的元素由返回值返回。 

​	参数: index 被删除的元素位置(必须介于 0 〜 size()-l 之间) 

##### 12.Integer相关API：

![image-20181221174132203](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181221174132203-5385292.png)

![image-20181221174144197](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181221174144197-5385304.png)

##### 13.Enum相关API：![image-20181221174828306](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181221174828306-5385708.png)

## 四、反射(https://mp.weixin.qq.com/s/kNwLps_lulIYNhDwz0V1OQ)

##### 	使用的前提条件：必须先得到代码的字节码的Class，Class类用于表示.class文件（字节码)

##### 	注意：在运行期间，一个类，只有一个Class对象产生。

##### 	能够分析类能力的程序称为反射。有以下的几个作用：

- 在运行时分析类的能力 。 
- 在运行时查看对象， 例如， 编写一个 toString 方法供所有类使用。
- 实现通用的数组操作代码。
- 利用 Method 对象， 这个对象很像中的函数指针。 

**JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。**

==要想解剖一个类，必须先要获取到该类的字节码文件对象。而解剖使用的就是Class类中的方法.所以先要获取到每一个字节码文件对应的Class类型的对象。==

反射就是把java类中的各种成分映射成一个个的Java对象

例如：一个类有：成员变量、方法、构造方法、包等等信息，利用反射技术可以对一个类进行解剖，把个个组成部分映射成一个个对象。

（其实：一个类中这些成员方法、构造方法、在加入类中都有一个类来描述）

如图是类的正常加载过程：反射的原理在与class对象。

（熟悉一下加载的时候：Class对象的由来是将class文件读入内存，并为之创建一个Class对象。）![image-20181223154838544](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181223154838544-5551318.png)

### 1.Class类

​	在程序运行期间， Java 运行时系统始终为所有的对象维护一个被称为运行时的类型标识。这个信息跟踪着每个对象所属的类。 虚拟机利用运行时类型信息选择相应的方法执行。	

​	保存这些信息的类被称为Class, **Object 类中的 getClass( ) 方法将会返回一个 Class 类型的实例。**==最常用的 Class 方法是 getName。这个方法将返回类的名字。调用静态方法 forName 获得类名对应的 Class 对象。==

如：

```Java
User e;
Class c1=e.getClass();
//前面是打印这个类的名字，后面是打印这个类的普通方法
System.out.println(e.getClass().getName() + " " + e.getName());
//如果类在一个包里面，包的名字也作为类名字的一部分
public class RandomTest {
    public static void main(String []args) {
        String className = "java.util.Random";
        try {
            //调用静态方法 forName 获得类名对应的 Class 对象。如果类名保存在字符串中， 并可在运行中改变， 就可以使用这个方法。 当然， 这个方法 只有在 dassName 是类名或接口名时才能够执行。否则， forName 方法将抛出一个 checked exception ( 已检查异常)。
            Class c1 = Class.forName(className);
            System.out.println(c1.getName());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
//结果：java.util.Random
```

 一个 Class 对象实际上表示的是一个类型， 而这个类型未必一定是一种类。 例如，int 不是类， 但 int.class 是一个 Class 类型的对象。==Class类实际上是一个泛型类。==

**==可以用newlnstance( )来动态地创建一个类的实例，如下面这个例子创建了一个与 e 具有相同类类型的实例。==**

```
e.getClass0.newlnstance();
```

**newlnstance 方法调用默认的构造器 (没有参数的构造器)初始化新创建的对象。 如果这个类没有默认的构造器， 就会抛出一个异常。**

**将 forName 与 newlnstance 配合起来使用， 可以根据存储在字符串中的类名创建一个对象：**

```
String s = "java.util.Random";
Object m = Class.forName(s).newlnstance();
```

##### 反射相关API：

​	•static Class forName(String className) 	返回描述类名为 className 的 Class 对象。 

​	•Object newlnstance() 	返回这个类的一个新实例。 

​	•Object newlnstance(Object[]args) 构造一个这个构造器所属类的新实例。 参数: args 这是提供给构造器的参数。 

​	• void printStackTrace() 	将 Throwable 对象和栈的轨迹输出到标准错误流。 	

##### 在java.lang.reflect 包中有三个类 Field、Method 和 Constructor 分别用于描述类的域、方法和构造器。

​	**这三个类都有一个叫做 getName 的方法， 用来返回项目的名称。 Field 类有一个 getType 方法， 用来返回描述域所属类型的 Class 对象。 Method 和 Constructor 类有能够报告参数类型的方法， Method 类还有一个可以报告返回类型的方法。还可以利用 java.lang.reflect 包中的 Modifier 类的静态方法分析getModifiers 返回的整型数值。**

##### Class相关API：

• Field[]   getFields() 
• Filed[]   getDeclaredFields() 
​	getFields 方法将返回一个包含 Field 对象的数组， 这些对象记录了这个类或其超类的公有域	                                                       getDeclaredField 方法也将返回包含 Field 对象的数组， 这些对象记录了这个 类的全部域。 如果类中没有域， 或者 Class 对象描述的是基本类型或数组类型， 这些 方法将返回一个长度为 0 的数组。 

• Method[] getMethods() 

• Method[]getDeclareMethods() 

​	返回包含 Method 对象的数组: getMethods 将返回所有的公有方法， 包括从超类继承 来的公有方法; getDeclaredMethods 返回这个类或接口的全部方法， 但不包括由超类 继承了的方法。 

• Constructor[]  getConstructors() 

• Constructor[]  getDeclaredConstructors() 

​	返回包含 Constructor (构造器)对象的数组， 其中包含了 Class 对象所描述的类的所有公有构造 器(getConstructors) 或所有构造器(getDeclaredConstructors。) 	

##### Constructor相关API：

•Class	 getDeclaringClass( ) 	返冋一个用于描述类中定义的构造器、 方法或域的 Class 对象。 

•Class[] 	getExceptionTypes( ) (在Constructor 和Method 类中) 		返回一个用于描述方法抛出的异常类型的 Class 对象数组。 

•int getModifiers( ) 	返回一个用于描述构造器、 方法或域的修饰符的整型数值。 使用 Modifier 类中的这个 

方法可以分析这个返回值。 

•String getName( ) 	返冋一个用于描述构造器、 方法或域名的字符串。
•Class[ ] getParameterTypes( ) (在 Constructor 和 Method 类中) 

​	返回一个用于描述参数类型的 Class 对象数组。

•Class getReturnType( ) (在 Method 类中) 		返回一个用于描述返H类型的 Class 对象。 

##### Modifier相关API：

•static String toString(int modifiers) 	返回对应 modifiers 中位设置的修饰符的字符串表示。 

•static boolean isAbstract(int modifiers) 

•static boolean isFinal(int modifiers) 

•static boolean islnterface(int modifiers) 

•static boolean isNative(int modifiers) 

•static boolean isPrivate(int modifiers) 

•static boolean isProtected(int modifiers) 

•static boolean isPublic(int modifiers) 

•static boolean isStatic(int modifiers) 

•static boolean isStrict(int modifiers) 

•static boolean isSynchronized(int modifiers) 

•static boolean isVolati1e(int modifiers) 	

这些方法将检测方法名中对应的修饰符在 modffiers 值中的位。

## 五、接口

### 1.接口的特性

1、不能通过new来实例化一个接口；

2、接口可以声明接口的变量，接口变量必须引用实现了接口的类对象；如：

```
Comparable x; 
x = new Employee(. . .); // OK provided Employee implements Comparable
```

3、接口中不能包含实例域或静态方法， 但可以包含常量

### 2.接口示例

#### 1、接口与回调

​	回调(callback) 是一种常见的程序设计模式。在这种模式中， 可以指出某个特定事件发生时应该采取的动作。

​	**在 JavaSE 8 中， 接口可以声明非抽象方法。**

### 3.lambda表达式

​	lambda表达式就是一个代码块， 以及必须传人代码的变量规范。所谓代码变量的规范，即要声明变量对应的类型，基本类型和引用类型都要声明。如：

```Java
//计算两个字符串长度的差值
(String first, String second)-> first.length() - second.length()
```

​	**==lambda 表达式形式: 参数， 箭头(-> ) 以及一个表达式。 如果代码要完成的计算无法放在一个表达式中， 就可以像写方法一样， 把这些代码放在 {}中，并包含显式的 return 语句。==**

```Java
(String first, String second) ->
{ 
	if (first.length() < second.length()) return -1; 
	else if (first.length() > second.length()) return 1;
	else return 0; 
	}
```

**即使 lambda 表达式没有参数， 仍然要提供空括号， 就像无参数方法一样:**

```Java
() -> { 
	for (int i= 100; i>= 0; i--) 
		System.out.println(i); 
}
```

**如果可以推导出一个 lambda 表达式的参数类型， 则可以忽略其类型。**

```Java
//因为在Comparator里面用通过泛型制定了String类型，所以后面的变量可以不用制定类型。
Comparator<String> comp	= (first, second) // Same as (String first, String second)
-> first.length() - second.length();
```

**如果方法只有一 参数， 而且这个参数的类型可以推导得出， 那么甚至还可以省略小括号:**

```Java
ActionListener listener = event ->
System.out.println("The time is " + new Date()"); 
// Instead of (event) -> . . . or (ActionEvent event) -> . . .
```

**无需指定 lambda 表达式的返回类型。 lambda 表达式的返回类型总是会由上下文推导得出。**

```java
//可以在需要 int 类型结果的上下文中使用。
(String first, String second) -> first.length() - second.length()
```

​	lambda 表达式中捕获的变量必须实际上是最终变量 ( effectively final)。实际上的最终变量是指， 这个变量初始化之后就不会再为它赋新值。

​	使用 lambda 表达式的重点是延迟执行 (deferred execution ) 。毕竟， 如果想耍立即执行代码， 完全可以直接执行， 而无需把它包装在一个lambda 表达式中。	

### 4.内部类

使用内部类的原因：

- **内部类方法可以访问该类定义所在的作用域中的数据， 包括私有的数据。** 
- **内部类可以对同一个包中的其他类隐藏起来。** 
- **当想要定义一个回调函数且不想编写大量代码时，使用匿名(anonymous) 内部类比较便捷。** 

##### 内部类的对象总有一个隐式引用， 它指向了创建它的外部类对象。 ![image-20181225093147854](https://learningpics.oss-cn-shenzhen.aliyuncs.com/images/image-20181225093147854-5701508.png)

这个引用在内部类的定义中是不可见的。然而， 为了说明这个概念， 我们将外围类对象的引用称为 outer。 注意，outer不是Java关键字。







































































































