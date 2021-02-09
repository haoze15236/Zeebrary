

# @FunctionalInterface

新增注解，用于限制接口中只存在一个抽象方法-----**Single Abstract Method interfaces**

标注了此注解的类可以包含多个以下方法：

| [default](#default) 修饰的方法 |
| -- |
| Static 修饰的方法 |

```java
@FunctionalInterface
public interface MethodReference<I,O>{
    O print(I input);

    default void showHello(){
        System.out.println("Hello");
    }

    static void showErr(){
        System.out.println("Error");
    }

}
```



# default

新增关键字

**特点：**

- 为接口的方法提供默认实现。

- 可以被实现类重写。

**目的：**

- 解决接口扩展，需要统一改动已经实现接口的类的痛点。

# lambda

本质是返回接口的匿名实现类，会自动匹配接口的**唯一抽象方法**

| 格式                                                         | 用法                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------ |
| (参数类型 参数值A，参数类型 参数值B...)->{语句A;语句B;... return 返回值;} | 标准格式                                               |
| (参数值A,参数值B...)->{语句A;语句B;... return 返回值;}       | 可以省略参数类型                                       |
| 参数A ->                                                     | 当只有一个参数时，可以省略()                           |
| ()->{语句A;语句B;... return 返回值;}                         | 当没有参数时，不能省略()                               |
| 参数A ->返回值                                               | 当实现方法中只有一个返回值时，可省略标准格式中的大括号 |



```java
public class TestLambda {
    public static void main(String[] args) {
        //匿名实现类写法
        MethodReference<String,String> method =new MethodReference<String,String>(){
            @Override
            public String print(String input) {
                return input.toUpperCase();
            }
        };
        //lambda表达式简化写法
        MethodReference<String,String> method2 =(input)-> input.toUpperCase();
        //::接口引用写法
        MethodReference<String,String> method3 =String::toUpperCase;

        System.out.println(method.print("asd"));
        System.out.println(method2.print("asd"));
        System.out.println(method3.print("asd"));
    }
}
```



# ::

方法引用，本质和lambda表达式一样，返回接口的匿名实现类。

| 格式             | 用法               |
| ---------------- | ------------------ |
| Class::new       | 调用类的构造方法   |
| Class::method    | 调用类的方法       |
| instance::method | 调用类的实现的方法 |



# Stream

[https://juejin.im/post/5c3d7c8a51882525dd591ac7#heading-16](https://juejin.im/post/5c3d7c8a51882525dd591ac7#heading-16)

[https://blog.csdn.net/Al_assad/article/details/82356845](https://blog.csdn.net/Al_assad/article/details/82356845)

https://developer.ibm.com/zh/articles/j-lo-java8streamapi/



## Collectors

```java
//LIST 转map
Map<Long,FndCompanyDimensionValue> existsValueList = selectValueList.stream()       .collect(Collectors.toMap(FndCompanyDimensionValue::getDimensionValueId,Function.identity()));

/**
 * List -> Map
 * 需要注意的是：
 * toMap 如果集合对象有重复的key，会报错Duplicate key ....
 *  apple1,apple12的id都为1。
 *  可以用 (k1,k2)->k1 来设置，如果有重复的key,则保留key1,舍弃key2
 */
Map<Integer, Apple> appleMap = appleList.stream().collect(Collectors.toMap(Apple::getId, a -> a,(k1,k2)->k1));

//去重
List<Person> unique = persons.stream().collect(
            Collectors.collectingAndThen(
                    Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(Person::getName))), ArrayList::new)
);
//根据两个字段去重
List<Person> unique = persons.stream().collect(
           Collectors. collectingAndThen(
                    Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(o -> o.getName() + ";" + o.getSex()))), ArrayList::new)
);
//分组
Map<Integer, List<Apple>> groupBy = appleList.stream().collect(Collectors.groupingBy(Apple::getId));

//计算 总金额
BigDecimal totalMoney = appleList.stream().map(Apple::getMoney).reduce(BigDecimal.ZERO, BigDecimal::add);
System.err.println("totalMoney:"+totalMoney); 

//查找流中最大 最小值
Optional<Dish> maxDish = Dish.menu.stream().
      collect(Collectors.maxBy(Comparator.comparing(Dish::getCalories)));
maxDish.ifPresent(System.out::println);

Integer max = list.stream().max((list1,list2) -> list1>list2 ? 1:-1).get();
        
 
Optional<Dish> minDish = Dish.menu.stream().
      collect(Collectors.minBy(Comparator.comparing(Dish::getCalories)));
minDish.ifPresent(System.out::println);
//排序
list.sort(( u1, u2)->u1.getAge().compareTo(u2.getAge()));
```