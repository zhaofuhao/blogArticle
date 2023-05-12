## 解决Spring中报错 required a single bean, but 2 were found

> 在写多租户案例的时候 遇到个小错误

![5a9b251c2c65ccea.png](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/LightPicture/2023/05/5a9b251c2c65ccea.png)

**启动后报错：required a single bean, but 2 were found** 

**大致意思就是：找到了两个bean** 

###  错误分析

1. 如果一个接口被两个类继承并注入到Bean中会报错这种错 但是我就继承了一个 而且报错的两个Bean一个是实现类一个是接口
2. 它下面也提示了可以用@Primary注解 我也测试了这种错误也可以解决 到这没结束

### 错误原因

1. 因为我的Mapper类是用`@MapperScan` 注解扫描的但是我标椎错范围了

```java
@SpringBootApplication
@MapperScan(basePackages = { "com.nwjshm.multitenancy.**"}) //**后面没有加mapper包 所以他默认扫描全部了
public class MultiTenancyDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(MultiTenancyDemoApplication.class, args);
	}

}
```

```java
@SpringBootApplication
@MapperScan(basePackages = { "com.nwjshm.multitenancy.**.mapper"}) //**后面没有加mapper包 所以他默认扫描全部了
public class MultiTenancyDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(MultiTenancyDemoApplication.class, args);
	}

}
```