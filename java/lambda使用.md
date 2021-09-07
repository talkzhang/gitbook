测试代码说明基本使用：

```java
public static void functionDemo() {
        // Function 接收一个参数，返回一个参数
        Function<String, String> f = t -> t + "Function";
        System.out.println(f.apply("函数式接口："));

        // Predicate 接收一个参数，返回boolean
        System.out.println("函数式接口：Predicate");
        Predicate<String> p = Objects::isNull;
        System.out.println(p.test(""));

        // Supplier 不接收参数，返回一个参数
        Supplier<String> s = () -> "函数式接口：Supplier";
        System.out.println(s.get());

        // Consumer 接收一个参数，不返回
        Consumer<String> c = System.out::println;
        c.accept("函数式接口：Consumer");

        // BiFunction 接收两个参数，返回一个参数
        System.out.println("函数式接口：BiFunction");
        BiFunction<Object, Object, Boolean> b = Objects::equals;
        System.out.println(b.apply("2", 2));

        // 还有比较常见的 Runnable、Comparator等
    }
```