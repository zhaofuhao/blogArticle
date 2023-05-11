## jeecgBoot框架 实现瘦身打包 lib和程序分开

> 最近项目上线 公司采用的jeecgboot框架 官网:http://www.jeecg.com/  
>
> 打的jar包有几百兆大小 lib占用了大多数 
>
> 分离的好处就是将lib提取出来 减少了打包的速度

### 打包步骤

1. 首先正常打包 将jar包解压 拿出BOOT-INF下的lib文件夹 单独存放

![image-20221116215100648](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/image-20221116215100648.png)

2. 修改pom文件

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<configuration>
				<layout>ZIP</layout>
				<includes>
					<include>
						<groupId>nothing</groupId>
						<artifactId>nothing</artifactId>
					</include>
				</includes>
			</configuration>
			<executions>
				<execution>
					<goals>
						<goal>repackage</goal>
					</goals>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
```



![image-20221116215310618](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/image-20221116215310618.png)

3. 在执行打包命令

4. 运行jar包 运行之前 需要创建这样的目录结构

```text
- config :将resources文件下的文件放到config文件
- lib： lib文件
- jar包
- 运行jar包bat文件 命令:java -Dloader.path=lib文件位置 -jar jar包位置
```



![image-20221116215700077](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/image-20221116215700077.png)

4. 运行jar包 官网步骤是这样写的 但是我运行到这一步的时候报错了 找不到mysql连接的url地址

![img](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/146671329-9431f9fd-e819-4c22-8209-0f7e239d2ba5.png)

### 解决找不到url问题

经过我翻越github的Issues记录时 发现有这个问题 并经过我的尝试解决方法有如下

### 报错的原因

> jeecg-boot-module-demo 项目下有个yml文件 这个项目打的包在lib文件夹下 导致优先级高了 这个yml文件没有配置信息 所以说报错 

### 第一种方法

> 将lib文件下的jeecg-boot-module-demo-3.0.jar 包给删掉错误就解决了

## 第二种方法

> 在system项目中resources文件下 创建config文件夹并将所有的文件放置里面 也可以解决这个问题

### 我采用的办法

> 1. 将jeecg-boot-module-demo 项目下有个yml文件给删除了

**参考文章：** https://github.com/jeecgboot/jeecg-boot/issues/3290  http://doc.jeecg.com/2043890
