## final finalize 和finally 的不同之处

- final是一个修饰符，可以是一个修饰变量、方法和类。如果final修饰变量，该变量的值在初始化后不能被改变。修饰类 表示类不能被继承 修饰方法 表示方法不能被重写
- Java 技术允许使用 finalize() 方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。这个方法是由垃圾收集器在确定这个对象没有被引用时对这个对象调用的，但是什么时候调用 finalize 没有保证。
- finally 是一个关键字，与 try 和 catch 一起用于异常的处理。finally 块一定会被执行，无论在 try 块中是否有发生异常。