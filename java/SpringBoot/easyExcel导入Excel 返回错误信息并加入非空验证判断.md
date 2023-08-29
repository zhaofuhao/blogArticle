## easyExcel导入Excel 返回错误信息所属行，并加入非空验证判断 

> 在项目中肯定会存在一些excel导入功能，存在的问题：导入的时候类型转换异常，如果只抛出错误异常的话，用户也看不懂错误信息，不如给用户提示是那一行的第几列的数据有异常。废话不多说 上代码

导入的框架是`easyExcel`： https://easyexcel.opensource.alibaba.com/

### 1. 返回错误信息所属行功能

ExcelListener 监听器代码：

```java
/**
 * @author ：扫地僧
 * @date ：2023/08/29 0029 13:35
 * @version: V1.0
 * @slogan: 天下风云出我辈，一入代码岁月催
 * @description: 通用EasyExcel 监听器
 **/
@Slf4j
public final class ExcelListener<T> extends AnalysisEventListener<T> {

    /**
     * 自定义用于暂时存储data
     * 可以通过实例获取该值
     */
    private List<T> datas = new ArrayList<>();

    /**
     * 每解析一行都会回调invoke()方法
     * @param data  读取后的数据对象
     * @param context 内容
     */
    @Override
    public void invoke(T data, AnalysisContext context) {
       
            datas.add(data);
      
    }

    /**
     * 读取完后操作
     * @param context
     */
    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
     log.info("所有数据读取完成");
    }

    /**
     * 异常方法 (类型转换异常也会执行此方法)  （读取一行抛出异常也会执行此方法)
     *
     * @param exception
     * @param context
     * @throws Exception
     */
    @Override
    public void onException(Exception exception, AnalysisContext context) {
        log.info("有异常");
        // 如果是某一个单元格的转换异常 能获取到具体行号
        // 如果要获取头的信息 配合invokeHeadMap使用
        if (exception instanceof ExcelDataConvertException) {
            ExcelDataConvertException excelDataConvertException = (ExcelDataConvertException)exception;
            log.error("第{}行，第{}列解析异常，数据为:{}", excelDataConvertException.getRowIndex(),
                    excelDataConvertException.getColumnIndex(), excelDataConvertException.getCellData());
            throw new RuntimeException("第"+excelDataConvertException.getRowIndex()+"行" +
                    "，第" + (excelDataConvertException.getColumnIndex() + 1) + "列读取错误");
        }
    }

    /**
     * 返回数据
     * @return 返回读取的数据集合
     **/
    public List<T> getDatas() {
        return datas;
    }
}
```

> 我将监听器的类型定义成泛型的好处是 无论做那个表的导入功能 只需要这一个监听器即可，具体的业务方法交给了Service
>
> 注意：`RuntimeException` 异常是java的运行时异常，如果公司有专门定义的异常类 替换就可以

实体类代码

```java
@Data
@TableName("sys_test")
public class SysTestEntity implements Serializable {
    private static final long serialVersionUID = 1L;

    /**
     * id
     */
    @TableId(type = IdType.ASSIGN_ID)
    private String id;

    /**
     * 姓名
     */
    @ExcelProperty("姓名")
    private String name;

    /**
     * 年龄
     */
    @ExcelProperty("年龄")
    private Integer age;

    /**
     * 手机号
     */
    @ExcelProperty("手机号")
    private Integer phone;

    /**
     * 工资
     */
    @ExcelProperty("工资")
    private BigDecimal salary;

    /**
     * 生日
     */
    @JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @ExcelProperty("生日")
    private Date birthday;

}
```

控制器代码

```java
@RestController
@RequestMapping("test/systest")
public class SysTestController {
    @Autowired
    private SysTestService sysTestService;
    /**
 * 模版导入
 * @param file
 * @return
 * @throws IOException
 */
    @PostMapping("upload")
    @ResponseBody
    public Result upload(MultipartFile file) throws IOException {
            //使用泛型指定实体类
            ExcelListener<SysTestEntity> excelListener = new ExcelListener<>();
            //读取数据
           EasyExcel.read(file.getInputStream(),SysTestEntity.class,excelListener).headRowNumber(1).sheet(0).doRead();
            //获取读取的数据
            List<SysTestEntity> list = excelListener.getDatas();
             //使用批量添加方法
            sysTestService.saveBatch(list);
            return ResultUtil.success("导入成功");
    }
}
```

### 实现非空校验

> 非空校验实现思路：
>
> 1.  自定义注解，定义一下错误信息
>
> 2. 自定义解析器，通过反射获取类的信息，根据注解去做校验，如果输入为空就抛出异常

自定义注解

```java
/**
 * @author ：扫地僧
 * @date ：2023/08/29 0029 15:00
 * @version: V1.0
 * @slogan: 天下风云出我辈，一入代码岁月催
 * @description: ExcelValid非空验证注解
 **/
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ExcelValid {
    String message() default "导入有未填入的字段";
}
```

自定义检验器

```java
/**
 * @author ：扫地僧
 * @date ：2023/08/29 0029 15:02
 * @version: V1.0
 * @slogan: 天下风云出我辈，一入代码岁月催
 * @description:
 **/
public class ExcelImportValid {
    /**
     * Excel导入字段校验
     *
     * @param object 校验的JavaBean 其属性须有自定义注解
     */
    public static void valid(Object object) {
        Field[] fields = object.getClass().getDeclaredFields();
        for (Field field : fields) {
            //设置可访问
            field.setAccessible(true);
            //属性的值
            Object fieldValue = null;
            try {
                fieldValue = field.get(object);
            } catch (IllegalAccessException e) {
                throw new RuntimeException("导入参数检查失败");
            }
            //是否包含必填校验注解
            boolean isExcelValid = field.isAnnotationPresent(ExcelValid.class);
            if (isExcelValid && Objects.isNull(fieldValue)) {
                System.out.println("导入错误");
                System.out.println(field.getAnnotation(ExcelValid.class).message());
                throw new RuntimeException("NULL"+field.getAnnotation(ExcelValid.class).message());
            }
        }
    }



}
```

实体类加入注解

```java
@Data
@TableName("sys_test")
public class SysTestEntity implements Serializable {
    private static final long serialVersionUID = 1L;


    /**
     * id
     */
    @TableId(type = IdType.ASSIGN_ID)
    private String id;

    /**
     * 姓名
     */
    @ExcelProperty("姓名")
    @ExcelValid(message = "姓名不能为空")
    private String name;

    /**
     * 年龄
     */
    @ExcelProperty("年龄")
    private Integer age;

    /**
     * 手机号
     */
    @ExcelProperty("手机号")
    private Integer phone;

    /**
     * 工资
     */
    @ExcelProperty("工资")
    private BigDecimal salary;

    /**
     * 生日
     */
    @JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @ExcelProperty("生日")
    private Date birthday;

}
```

ExcelListener 监听器代码

```java
/**
 * @author ：扫地僧
 * @date ：2023/08/29 0029 13:35
 * @version: V1.0
 * @slogan: 天下风云出我辈，一入代码岁月催
 * @description: 通用EasyExcel 监听器
 **/
@Slf4j
public final class ExcelListener<T> extends AnalysisEventListener<T> {

    /**
     * 自定义用于暂时存储data
     * 可以通过实例获取该值
     */
    private List<T> datas = new ArrayList<>();

    /**
     * 每解析一行都会回调invoke()方法
     * @param data  读取后的数据对象
     * @param context 内容
     */
    @Override
    public void invoke(T data, AnalysisContext context) {
        //数据存储到list，供批量处理，或后续自己业务逻辑处理。
        try {
            ExcelImportValid.valid(data);
            datas.add(data);
        } catch (Exception e) {
            // 校验失败，处理异常
            System.out.println("校验失败：" + e.getMessage());
            // 可以根据需要采取其他处理措施
            throw new ApiException(e.getMessage());
        }
    }

    /**
     * 读取完后操作
     * @param context
     */
    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
     log.info("所有数据读取完成");
    }

    /**
     * 异常方法 (类型转换异常也会执行此方法)  （读取一行抛出异常也会执行此方法)
     *
     * @param exception
     * @param context
     * @throws Exception
     */
    @Override
    public void onException(Exception exception, AnalysisContext context) {
        log.info("有异常");
        // 如果是某一个单元格的转换异常 能获取到具体行号
        // 如果要获取头的信息 配合invokeHeadMap使用
        if (exception instanceof ExcelDataConvertException) {
            ExcelDataConvertException excelDataConvertException = (ExcelDataConvertException)exception;
            log.error("第{}行，第{}列解析异常，数据为:{}", excelDataConvertException.getRowIndex(),
                    excelDataConvertException.getColumnIndex(), excelDataConvertException.getCellData());
            throw new ApiException("第"+excelDataConvertException.getRowIndex()+"行" +
                    "，第" + (excelDataConvertException.getColumnIndex() + 1) + "列读取错误");
        }
        //抛出非空校验异常
        throw new ApiException(exception.getMessage());
    }

    /**
     * 返回数据
     * @return 返回读取的数据集合
     **/
    public List<T> getDatas() {
        return datas;
    }
}
```

> 有个坑：
>
> invoke方法抛出异常后 系统还是显示导入成功，也打印校验失败错误信息，在我仔细阅读官方文档后发现，抛出异常后会执行onException方法，需要也在onException方法将异常信息抛出去才可以
