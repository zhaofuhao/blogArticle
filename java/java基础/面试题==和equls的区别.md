##  java面试题 ==和equls的区别

### ==运算符

`==运算符`可以使用在基本数据类型变量和引用数据类型变量中

1. 如果比较的是基本数据类型变量，比较两个变量保存的数据是否相等(类型不一定相同)

```java
       //值类型比较
        int i =10;
        int j =10;
        double d =10.0;
        System.out.println(j==d);//true
        System.out.println(i==j); //true
```

2. 如果比较的是引用数据类型变量 比较两个对象的地址值是否相同 即引用是否指向同一个实体

```java
       //引用类型比较
        Customer c1 = new Customer("Tom", 21);
        Customer c2 = new Customer("Tom", 21);
        System.out.println(c1==c2);//false
```



### equals方法

`equals`是一个方法而非运算符 只能适用于引用数据类型 值类型想使用的话得需要使用对应的包装类

1. Object类中定义的`equals`方法作用和`==`相同

   定义一个Customer类没有重写equals方法

```java
public class Customer {
    private String name;
    private int age;

    public Customer() {
    }

    public Customer(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

测试

```java
Customer c1 = new Customer("Tom", 21);
Customer c2 = new Customer("Tom", 21);
System.out.println(c1==c2);//false
```

2. string Date File 包装类都重写了Object equals方法 重写后比较的是两个对象实体内容是否相同

```java
String s1= new String("zhaofuhao");
String s2 = new String("zhaofuhao");
System.out.println(s1==s2); //true
```

> 总结：
>
> 1. 在object类中 ==和equls作用相同
> 2. 如果子类重写了equls方法 如果地址值相同就返回true 如果地址值不同但是内容相同也返回true
> 3. equals是一个方法而非运算符 只能适用于引用数据类型 值类型想使用的话得需要使用对应的包装类
> 4. ==是一个运算符可以使用在基本数据类型变量和引用数据类型变量中

原文连接https://nwjshm.cn/archives/11.html

