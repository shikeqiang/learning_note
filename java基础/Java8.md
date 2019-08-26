# 一、lambda

## 1.基础语法

```java
/*
 * 一、Lambda 表达式的基础语法：
 * 	Java8中引入了一个新的操作符 "->" 该操作符称为箭头操作符或 Lambda 操作符，
 * 	箭头操作符将 Lambda 表达式拆分成两部分：
 * 左侧：Lambda 表达式的参数列表，方法的参数列表
 * 右侧：Lambda 表达式中所需执行的功能， 即 Lambda 体，即方法实现的功能
 *
 * 语法格式一：无参数，无返回值
 * 		() -> System.out.println("Hello Lambda!");  左边为参数，右边是方法的实现
 *
 * 语法格式二：有一个参数，并且无返回值
 * 		(x) -> System.out.println(x)
 *
 * 语法格式三：若只有一个参数，小括号可以省略不写
 * 		x -> System.out.println(x)
 *
 * 语法格式四：有两个以上的参数，有返回值，有多条语句时，lambda表达式要用大括号
 *		Comparator<Integer> com = (x, y) -> {
 *			System.out.println("函数式接口");
 *			return Integer.compare(x, y);
 *		};
 *
 * 语法格式五：若 Lambda 体中只有一条语句， return 和 大括号都可以省略不写
 * 		Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
 *
 * 语法格式六：Lambda 表达式的参数列表的数据类型可以省略不写，因为JVM编译器通过上下文推断出，数据类型，即“类型推断”
 * 		(Integer x, Integer y) -> Integer.compare(x, y);
 *
 * 上联：左右遇一括号省
 * 下联：左侧推断类型省
 * 横批：能省则省
 *
 * 二、Lambda 表达式需要“函数式接口”的支持
 * 函数式接口：接口中只有一个抽象方法的接口，称为函数式接口。
 * 可以使用注解 @FunctionalInterface 修饰,可以检查是否是函数式接口
 */
public class TestLambda2 {

    @Test
    public void test1() {
        int num = 0;        //jdk 1.7 前，局部内部类的变量必须声明是 final，jdk8开始默认是final

        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello World!" + num);
            }
        };

        r.run();

        System.out.println("-------------------------------");
//      匿名内部类通过lambda表达式实现
        Runnable r1 = () -> System.out.println("Hello Lambda!");
        r1.run();
    }

//    有一个参数并且无返回值
    @Test
    public void test2() {
        Consumer<String> con = x -> System.out.println(x);  // 若只有一个参数，小括号可以省略不写
        con.accept("杰哥牛逼！");    // accept方法：Performs this operation on the given argument.
    }

    @Test
    public void test3() {
        Comparator<Integer> com = (x, y) -> {   // 有多条语句时lambda表达式要用大括号
            System.out.println("函数式接口");
            return Integer.compare(x, y);
        };
    }

//  如果只有一条语句，return 和 大括号都可以省略不写
    @Test
    public void test4() {
        Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
    }

    @Test
    public void test5() {
//		String[] strs={"aaa", "bbb", "ccc"};    // 如果不拆开的话，直接初始化也是类型推断
//		strs = {"aaa", "bbb", "ccc"};
        List<String> list = new ArrayList<>();
        show(new HashMap<>());  // jdk8开始可以不写，会进行类型推断
    }

    public void show(Map<String, Integer> map) {
    }

    //需求：对一个数进行运算
    @Test
    public void test6() {
        Integer num = operation(100, (x) -> x * x);
        System.out.println(num);
        System.out.println(operation(200, (y) -> y + 200));
    }

    /**
     * @param num 要计算的数
     * @param mf  函数式接口
     * @return
     */
    public Integer operation(Integer num, MyFun mf) {
        return mf.getValue(num);
    }
}
```

> 作为参数传递 Lambda 表达式:为了将 Lambda 表达式作为参数传递，接收Lambda 表达式的参数类型必须是与该 Lambda 表达式兼容的函数式接口的类型。

例子：

```java
public class TestLambda {
    List<Employee> emps = Arrays.asList(
            new Employee(101, "张三", 18, 9999.99),
            new Employee(102, "李四", 59, 6666.66),
            new Employee(103, "王五", 28, 3333.33),
            new Employee(104, "赵六", 8, 7777.77),
            new Employee(105, "田七", 38, 5555.55)
    );

    @Test
    public void test1() {
        // 过滤，先按年龄比,如果年龄相等就比较姓名，否则比较年龄
        Collections.sort(emps, (e1, e2) -> {
            if (e1.getAge() == e2.getAge()) {
                return e1.getName().compareTo(e2.getName());
            } else {
                return -Integer.compare(e1.getAge(), e2.getAge()); // 加了减号就是倒序
            }
        });

        for (Employee emp : emps) {
            System.out.println(emp);
        }
    }

    // 处理字符串
    @Test
    public void test2() {
        // 去除空格
        String trimStr = strHandler("\t\t\t baichen大帅哥   ", (str) -> str.trim());
        System.out.println(trimStr);
        // 转为大写
        String upper = strHandler("abcdef", (str) -> str.toUpperCase());
        System.out.println(upper);
        // 切割字符串
        String newStr = strHandler("baichen大帅哥", (str) -> str.substring(2, 5));
        System.out.println(newStr);
    }

    //需求：用于处理字符串
    public String strHandler(String str, MyFunction mf) {
        return mf.getValue(str);
    }

    @Test
    public void test3() {
        op(100L, 200L, (x, y) -> x + y);

        op(100L, 200L, (x, y) -> x * y);
    }

    //需求：对于两个 Long 型数据进行处理
    public void op(Long l1, Long l2, MyFunction2<Long, Long> mf) {
        System.out.println(mf.getValue(l1, l2));
    }
}
```

## 2.常见接口类型

### 四大核心函数式接口(都在java.util.function包中)

![image-20190825211635099](/Users/jack/Desktop/md/images/image-20190825211635099.png)

### 其他接口

![image-20190825211823962](/Users/jack/Desktop/md/images/image-20190825211823962.png)

## 3.方法引用和构造器引用

### 3.1 方法引用

​	若 Lambda 体中的功能，已经有方法提供了实现，可以使用方法引用（可以将方法引用理解为 Lambda 表达式的另外一种表现形式）

主要有三种语法格式：
* 对象的引用 :: 实例方法名
* 类名 :: 静态方法名
* 类名 :: 实例方法名

> 注意：
> *   ==方法引用所引用的方法的参数列表与返回值类型，需要与函数式接口中抽象方法的参数列表和返回值类型保持一致！==
> *   ==若Lambda 的参数列表的第一个参数，是实例方法的调用者，第二个参数(或无参)是实例方法的参数时，格式： ClassName::MethodName(参照Test5)==

### 3.2 构造器引用

​	与函数式接口相结合，自动与函数式接口中方法兼容。可以把构造器引用赋值给定义的方法，与构造器参数
列表要与接口中抽象方法的参数列表一致!构造器的参数列表，需要与函数式接口中参数列表保持一致！

格式：**类名 :: new**

## 3.3 数组引用

格式: type[] :: new

例子：

```java
public class TestMethodRef {
    //数组引用
    @Test
    public void test8() {
        Function<Integer, String[]> fun = (args) -> new String[args];
        String[] strs = fun.apply(10);
        System.out.println(strs.length);
        System.out.println("--------------------------");
        Function<Integer, Employee[]> fun2 = Employee[]::new;
        Employee[] emps = fun2.apply(20);
        System.out.println(emps.length);
    }

    //构造器引用
    @Test
    public void test7() {
//      构造器的参数列表，需要与函数式接口中参数列表保持一致
        Function<String, Employee> fun = Employee::new;// 供给型接口
        BiFunction<String, Integer, Employee> fun2 = Employee::new;
        //有参数的构造器引用
        Function<String, Employee> fun3 = Employee::new;
        Employee employee = fun3.apply("小明");// 调用了只包含了name属性的构造器,根据参数类型判断
        System.out.println(employee);
    }

    //没有参数的构造器引用
    @Test
    public void test6() {
        Supplier<Employee> sup = () -> new Employee();
        System.out.println(sup.get());
        System.out.println("------------------------------------");
        Supplier<Employee> sup2 = Employee::new;
        System.out.println(sup2.get());
    }

    //类名 :: 实例方法名
    @Test
    public void test5() {
        BiPredicate<String, String> bp = (x, y) -> x.equals(y);
        System.out.println(bp.test("abcde", "abcde"));
        System.out.println("-----------------------------------------");
// 若Lambda 的参数列表的第一个参数，是实例方法的调用者，第二个参数(或无参)是实例方法的参数时，
// 格式： ClassName::MethodName(即这里相当于是abc.equals(abc))
        BiPredicate<String, String> bp2 = String::equals;
        System.out.println(bp2.test("abc", "abc"));
        System.out.println("-----------------------------------------");
        Function<Employee, String> fun = (e) -> e.show();
        System.out.println(fun.apply(new Employee()));
        System.out.println("-----------------------------------------");
        Function<Employee, String> fun2 = Employee::show;
        System.out.println(fun2.apply(new Employee()));

    }

    //类名 :: 静态方法名
    @Test
    public void test4() {
        Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
        System.out.println("-------------------------------------");
        Comparator<Integer> com2 = Integer::compare;
    }

    @Test
    public void test3() {
        BiFunction<Double, Double, Double> fun = (x, y) -> Math.max(x, y);
        System.out.println(fun.apply(1.5, 22.2));
        System.out.println("--------------------------------------------------");
        BiFunction<Double, Double, Double> fun2 = Math::max;
        System.out.println(fun2.apply(1.2, 1.5));
    }

    //对象的引用 :: 实例方法名
    @Test
    public void test2() {
        Employee emp = new Employee(101, "张三", 18, 9999.99);
        Supplier<String> sup = () -> emp.getName();
        System.out.println(sup.get());
        System.out.println("----------------------------------");
        Supplier<Integer> sup2 = emp::getAge;// 参数类型和返回值类型要一致
        System.out.println(sup2.get());
    }

    @Test
    public void test1() {
        PrintStream ps = System.out;
        Consumer<String> con = (str) -> ps.println(str);
        con.accept("Hello World！");
        System.out.println("--------------------------------");
        // 方法引用，参数类型和返回体类型要一致(这里是accept方法)println是实例方法
        Consumer<String> con2 = ps::println;
        con2.accept("Hello Java8！");
        Consumer<String> con3 = System.out::println;
        con3.accept("这也是方法引用");
    }
}
```

# 二、Stream API













