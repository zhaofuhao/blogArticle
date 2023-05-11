

## Spring Boot 构建多租户系统 实现动态切换数据源

### 概述

SaaS(Software as a Service)，多租户系统（一套系统，不同租户数据不同） 它只是一种软件架构，从技术角度来说很好实现。主要是运营下去会比较难。



**传统模式下的系统** 

![传统模式系统](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/%E4%BC%A0%E7%BB%9F%E6%A8%A1%E5%BC%8F%E7%B3%BB%E7%BB%9F.png)

**多租户下的系统架构图**

![多租户下的架构](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/%E5%A4%9A%E7%A7%9F%E6%88%B7%E4%B8%8B%E7%9A%84%E6%9E%B6%E6%9E%84.png)

> 多租户的好处  好升级也好维护， 假设我们开发一个应用程序，并且希望这一套程序销售给N个客户用，传统模式下，我们要为N个客户创建 服务器，数据库 并为N个客户部署相同的程序N次。采用多租户了就部署一套

## 实现多租户

### 实现方式

主流的方案有三种

- 方案1：共享数据库 共享数据架构 通过租户id进行区分属于那个租户
- 方案2：共享数据库 多个租户共享数据库 但一个租户一个Schema
- 方案3：独立数据库 一个租户一个数据库(采用)

### 方案3实现

>采用方案3需要创建一个单独的数据库存储所有的租户信息，并存储租户的数据库和数据源信息

1. 难点1：不同租户使用的时候如何进行切换数据库？
2. 难点2：需要动态添加数据源信息

#### 难点1的解决办法

1. 可以通过域名的方式来识别租户 我们可以为每一个租户提供一个二级域名，通过二级域名就可以实现区分租户比如 zuhu1.saas.com，zuhu2.saas.com
2. 可以将租户信息作为请求参数传递给服务端，服务端进行一个识别，如 saas.com?tenantId=tenant1,saas.com?tenantId=tenant2。
3. 可以在请求头Header 设置租户信息，服务端通过解析Header中获取租户信息。

**我采用的是 二级域名+Header设置租户信息 ** 

![image-20230111104257793](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/image-20230111104257793.png)



#### 难点2的解决办法

因为 使用的是`mybatis-plus`框架  官网提供了两个多数据源的框架

多数据源既动态数据源，项目开发逐渐扩大，单个数据源、单一数据源已经无法满足需求项目的支撑需求。

由此延伸了多数据源的扩展，下文提供了两种不同方向的扩展插件。

- `dynamic-datasource` 开源文档付费，属于组织参与者`小锅盖`发起的项目
- `mybatis-mate` 企业级付费授权，资料文档免费

我使用的是`dynamic-datasource`这个框架 文档我入手了



#### 采用的框架

> 我使用的框架
>
> jeecgboot低代码开发框架 jeecgboot集成了dynamic-datasource框架 
>
> 数据库 mysql

#### 数据表准备

```sql

-- 租户表
CREATE TABLE `sys_data_source` (
  `id` varchar(36) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `code` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '租户编码',
  `name` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '租户名称',
  `remark` varchar(200) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '备注',
  `db_type` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '数据库类型',
  `db_driver` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '驱动类',
  `db_url` varchar(500) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '数据源地址',
  `db_name` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '数据库名称',
  `db_username` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '用户名',
  `db_password` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '密码',
  `create_by` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '创建人',
  `create_time` datetime DEFAULT NULL COMMENT '创建日期',
  `update_by` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '更新人',
  `update_time` datetime DEFAULT NULL COMMENT '更新日期',
  `sys_org_code` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '所属部门',
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `uk_sdc_rule_code` (`code`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci ROW_FORMAT=DYNAMIC;
```

#### 代码实现动态添加数据源

>jeecgboot有一个多数据管理的页面 我就基于他那个功能修改了一下

```java

@Override
    public Result saveDataSource(SysDataSource sysDataSource) {
        try {
            long count = checkDbCode(sysDataSource.getCode());
            if (count > 0) {
                return Result.error("数据源编码已存在");
            }
            String dbPassword = sysDataSource.getDbPassword();
            if (StringUtils.isNotBlank(dbPassword)) {
                String encrypt = SecurityUtil.jiami(dbPassword);
                sysDataSource.setDbPassword(encrypt);
            }
            boolean result = save(sysDataSource);
            if (result) {
                //动态创建数据源
                addDynamicDataSource(sysDataSource, dbPassword);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return Result.OK("添加成功！");
    }

/**
     * 动态添加数据源 【注册mybatis动态数据源】
     *
     * @param sysDataSource 添加数据源数据对象
     * @param dbPassword    未加密的密码
     */
    private void addDynamicDataSource(SysDataSource sysDataSource, String dbPassword) {
        DataSourceProperty dataSourceProperty = new DataSourceProperty();
        dataSourceProperty.setUrl(sysDataSource.getDbUrl());
        dataSourceProperty.setPassword(dbPassword);
        dataSourceProperty.setDriverClassName(sysDataSource.getDbDriver());
        dataSourceProperty.setUsername(sysDataSource.getDbUsername());
        DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
        DataSource dataSource = dataSourceCreator.createDataSource(dataSourceProperty);
        try {
            ds.addDataSource(sysDataSource.getCode(), dataSource);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

#### 动态切换数据源

```java
DynamicDataSourceContextHolder.push("数据源名称");//动态切换数据源
```

> 思路：当请求后端接口的时候 通过web拦截器 拦截一下请求头获取租户编码 进行切换

```java
//web相关
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Resource
    private TenantDsInterceptor tenantDsInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //注册租户切换数据源拦截器
        registry.addInterceptor(this.tenantDsInterceptor);
    }
}
----------------------------
/**
 * @author ：扫地僧 租户切换拦截器
 * @date ：2022-12-28 上午 10:40:36
 * @version: V1.0
 * @slogan: 天下风云出我辈，一入代码岁月催
 * @description:
 **/
@Slf4j
@Component
public class TenantDsInterceptor implements HandlerInterceptor {
    @Autowired
    private ISysDataSourceService sysDataSourceService;

    /**
     * 在请求处理前调用
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();
        log.info("经过多数据源Interceptor,当前路径是{}", requestURI);
        String tenantId = request.getHeader("tencode");
        //如果tenantId为空，则使用默认数据源
        if (StringUtils.isNotEmpty(tenantId)){
            log.info("拿到的租户编码{}", tenantId);
            sysDataSourceService.changeDsByTenantId(tenantId);
        }
        return true;
    }

    /**
     * 请求处理之后进行调用，但是在视图被渲染之前（Controller方法调用之后）
     * @param request
     * @param response
     * @param handler
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    /**
     * 在整个请求结束之后被调用，也就是在DispatcherServlet 渲染了对应的视图之后执行（主要是用于进行资源清理工作）
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        //清空当前线程数据源
        DynamicDataSourceContextHolder.clear();
    }
}
```

