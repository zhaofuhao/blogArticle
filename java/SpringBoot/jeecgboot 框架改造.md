## jeecgboot 框架改造

> 上篇 [jeecgboot 改造@Dict注解 实现可以翻译list集合 实体 以及map集合](https://nwjshm.cn/archives/36.html) 文章中留个续集 

```java
@Slf4j
@Component
public class JeecgUtils {
    @Autowired
    public RedisTemplate redisTemplate;

    @Lazy
    @Autowired
    private CommonAPI commonApi;



    /**
     *  翻译字典文本
     * @param code
     * @param text
     * @param table
     * @param key
     * @return
     */

    public  String translateDictValue(String code, String text, String table, String key) {
        if(oConvertUtils.isEmpty(key)) {
            return null;
        }
        StringBuffer textValue=new StringBuffer();
        String[] keys = key.split(",");
        for (String k : keys) {
            String tmpValue = null;
            log.debug(" 字典 key : "+ k);
            if (k.trim().length() == 0) {
                continue; //跳过循环
            }
            if (!StringUtils.isEmpty(table)){
                log.debug("--DictAspect------dicTable="+ table+" ,dicText= "+text+" ,dicCode="+code);
                String keyString = String.format("sys:cache:dictTable::SimpleKey [%s,%s,%s,%s]",table,text,code,k.trim());
                if (redisTemplate.hasKey(keyString)){
                    try {
                        tmpValue = oConvertUtils.getString(redisTemplate.opsForValue().get(keyString));
                    } catch (Exception e) {
                        log.warn(e.getMessage());
                    }
                }else {
                    tmpValue= commonApi.translateDictFromTable(table,text,code,k.trim());
                }
            }else {
                String keyString = String.format("sys:cache:dict::%s:%s",code,k.trim());
                if (redisTemplate.hasKey(keyString)){
                    try {
                        tmpValue = oConvertUtils.getString(redisTemplate.opsForValue().get(keyString));
                    } catch (Exception e) {
                        log.warn(e.getMessage());
                    }
                }else {
                    tmpValue = commonApi.translateDict(code, k.trim());
                }
            }


            if (tmpValue != null) {
                if (!"".equals(textValue.toString())) {
                    textValue.append(",");
                }
                textValue.append(tmpValue);
            }

        }
        return textValue.toString();
    }


}
```

