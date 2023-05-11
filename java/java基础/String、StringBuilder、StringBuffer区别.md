## Stirng、StringBuilder、StringBuffer的区别

- String不可变，内部char数组被private final修饰不可变，线程安全
- StringBuilder可变，线程不安全，可以使用append进行拼接字符串
- StringBuffer可变，线程安全，采用synchronized进行同步，操作和StringBuilder一样‘
  - 假如StringBuffer出现在循环体中即使没有出现线程竞争，频繁进行同步互斥会带来性能开销，所以synchronize进行了锁粗化，将锁的范围扩大到整个操作，假如是单线程使用stringBuffer，也就是不存在线程竞争，synchronizd会进行锁消除

