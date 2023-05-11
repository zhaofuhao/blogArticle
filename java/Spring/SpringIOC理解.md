## SpringIOC的理解

IOC（Inversion of Control 即控制反转）将对象交给Spring容器管理.

![IOC控制反转](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/IOC%E6%8E%A7%E5%88%B6%E5%8F%8D%E8%BD%AC.png)

**和传统方式区别**

![image-20220827111927869](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/image-20220827111927869.png)

>Spring提供了IOC容器，来控制对象的创建，无论你创建对象，处理对象之间的依赖关系，对象的创建时间还是对象的创建数量，都是spring提供IOC容器上配置对象的信息就可以了。

### IOC容器管理对象的好处

1. 由IOC容器帮对象找相应的依赖思想并注入，并不是由对象主动去找
2. 资源集中管理，实现资源的可配置和易管理
3. 降低了使用资源双方的依赖程度，松耦合
4. 解决了Dao和Service的强耦合。