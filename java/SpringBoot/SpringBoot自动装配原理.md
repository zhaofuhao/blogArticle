## SpringBoot自动装配原理

自动装配，简单来说就是自动吧第三方的bean装配到ioc容器中 不需要我们去在去写bean配置

在springboot 主启动类上加上@SpringBootApplication注解就可以实现自动装配

@SpringbootApplication是一个复合注解，真正实现自动装配的注解是@EnableAutoConfiguration

自动装配的实现主要依靠三个核心关键技术。

引入Starter启动依赖组件的时候，这个组件里面必须要包含@Configuration配置类，在这个配置类里面通过@Bean注解声明需要装配到IOC容器的Bean对象。

这个配置类是放在第三方的 jar 包里面，然后通过 SpringBoot 中的约定优于配置 思想，把这个配置类的全路径放在 classpath:/META-INF/spring.factories 文件中。 

这样 SpringBoot 就可以知道第三方 jar 包里面的配置类的位置，这个步骤主要是 用到了 Spring 里面的 SpringFactoriesLoader 来完成的。

SpringBoot  拿到所第三方  jar  包里面声明的配置类以后，再通过  Spring  提供的 ImportSelector 接口，实现对这些配置类的动态加载。

