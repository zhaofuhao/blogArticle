# EasyExcel 读取Excel数据报错解决

> 这两天做excel导入功能时，遇到个类型转换异常错误，我采用的是阿里巴巴开源的 easyExcel
>
> 地址：https://easyexcel.opensource.alibaba.com/

复刻一下这个bug

实体类如下

```java
@Data

public class DemoData {
    private String id;
    @ExcelProperty(value ="姓名")
    private String  date;
    @ExcelProperty(value ="金额")
    private BigDecimal money;
}
```

具体如何读的代码就不展现了，就是根据官网文档写的。

excel表如下

![image-20230809215420962](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/image-20230809215420962.png)

问题就是因为excel的格式问题，数字类型出现了逗号导致系统读取的时候因为这个逗号报错了，把格式改一下就可以了