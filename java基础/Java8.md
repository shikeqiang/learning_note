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

### 四大核心函数式接口

![image-20190825211635099](/Users/jack/Desktop/md/images/image-20190825211635099.png)

### 其他接口

![image-20190825211823962](/Users/jack/Desktop/md/images/image-20190825211823962.png)





