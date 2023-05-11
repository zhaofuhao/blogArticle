##  jeecgboot 改造@Dict注解 实现可以翻译list集合 实体 以及map集合

> jeecgBoot框架的@dict字典翻译注解挺好用的 支持翻译数据字典和数据库 但是他默认的@dict注解只支持分页翻译 其他的是不支持的 上源码

```java
/**
     * 本方法针对返回对象为Result 的IPage的分页列表数据进行动态字典注入
     * 字典注入实现 通过对实体类添加注解@dict 来标识需要的字典内容,字典分为单字典code即可 ，table字典 code table text配合使用与原来jeecg的用法相同
     * 示例为SysUser   字段为sex 添加了注解@Dict(dicCode = "sex") 会在字典服务立马查出来对应的text 然后在请求list的时候将这个字典text，已字段名称加_dictText形式返回到前端
     * 例输入当前返回值的就会多出一个sex_dictText字段
     * {
     *      sex:1,
     *      sex_dictText:"男"
     * }
     * 前端直接取值sext_dictText在table里面无需再进行前端的字典转换了
     *  customRender:function (text) {
     *               if(text==1){
     *                 return "男";
     *               }else if(text==2){
     *                 return "女";
     *               }else{
     *                 return text;
     *               }
     *             }
     *             目前vue是这么进行字典渲染到table上的多了就很麻烦了 这个直接在服务端渲染完成前端可以直接用
     * @param result
     */
    private Object parseDictText(Object result) {
        if (result instanceof Result) {
            if (((Result) result).getResult() instanceof IPage) {
                //取出结果集
              List<Object> records=((IPage) ((Result) result).getResult()).getRecords();
                //update-begin--Author:zyf -- Date:20220606 ----for：【VUEN-1230】 判断是否含有字典注解,没有注解返回-----
                Boolean hasDict= checkHasDict(records);
                if(!hasDict){
                    return result;
                }
                List<JSONObject> dictText = getDictText(records);
                ((IPage) ((Result) result).getResult()).setRecords(dictText);
            }
            else {
                //取出结果集
               Object object=  (Object) ((Result) result).getResult();
                Class<?> aClass = ((Result) result).getResult().getClass();
                if ("java.util.HashMap".equals(aClass.getName())){
                    //将map转换为集合在转换为map
                   Map<String, Object> mapset = new HashMap<>();
                    Map<String,Object> map=  (Map<String,Object>) object;
                    for (String s : map.keySet()) {
                        Object a1 = map.get(s);
                        List<Object> records=new ArrayList<>();
                        Class<?> aClass1 = a1.getClass();
                        if ("java.util.ArrayList".equals(aClass1.getName())){
                            records = (ArrayList)map.get(s);
                        }else{
                         records = oConvertUtils.objToList(a1, Object.class);
                        }
                        Boolean hasDict= checkHasDict(records);
                        if(!hasDict){
                            return result;
                        }
                        List<JSONObject> dictText = getDictText(records);
                        mapset.put(s,dictText);
                    }
                    ((Result) result).setResult(mapset);
                }else {
                    List<Object> records = oConvertUtils.objToList(object, Object.class);
                    //update-begin--Author:zyf -- Date:20220606 ----for：【VUEN-1230】 判断是否含有字典注解,没有注解返回-----
                    Boolean hasDict= checkHasDict(records);
                    if(!hasDict){
                        return result;
                    }
                    List<JSONObject> dictText = getDictText(records);
                    ((Result) result).setResult(dictText);
                }
        }
        }
        return result;
    }
/**
传入集合 翻译字典值
**/
private List<JSONObject> getDictText(List<Object> records){
        List<JSONObject> items = new ArrayList<>();
        //step.1 筛选出加了 Dict 注解的字段列表
        List<Field> dictFieldList = new ArrayList<>();
        // 字典数据列表， key = 字典code，value=数据列表
        Map<String, List<String>> dataListMap = new HashMap<>(5);


        log.debug(" __ 进入字典翻译切面 DictAspect —— " );
        //update-end--Author:zyf -- Date:20220606 ----for：【VUEN-1230】 判断是否含有字典注解,没有注解返回-----
        for (Object record : records) {
            String json="{}";
            try {
                //update-begin--Author:zyf -- Date:20220531 ----for：【issues/#3629】 DictAspect Jackson序列化报错-----
                //解决@JsonFormat注解解析不了的问题详见SysAnnouncement类的@JsonFormat
                json = objectMapper.writeValueAsString(record);
                //update-end--Author:zyf -- Date:20220531 ----for：【issues/#3629】 DictAspect Jackson序列化报错-----
            } catch (JsonProcessingException e) {
                log.error("json解析失败"+e.getMessage(),e);
            }
            //update-begin--Author:scott -- Date:20211223 ----for：【issues/3303】restcontroller返回json数据后key顺序错乱 -----
            JSONObject item = JSONObject.parseObject(json, Feature.OrderedField);
            //update-end--Author:scott -- Date:20211223 ----for：【issues/3303】restcontroller返回json数据后key顺序错乱 -----

            //update-begin--Author:scott -- Date:20190603 ----for：解决继承实体字段无法翻译问题------
            //for (Field field : record.getClass().getDeclaredFields()) {
            // 遍历所有字段，把字典Code取出来，放到 map 里
            for (Field field : oConvertUtils.getAllFields(record)) {
                String value = item.getString(field.getName());
                if (oConvertUtils.isEmpty(value)) {
                    continue;
                }
                //update-end--Author:scott  -- Date:20190603 ----for：解决继承实体字段无法翻译问题------
                if (field.getAnnotation(Dict.class) != null) {
                    if (!dictFieldList.contains(field)) {
                        dictFieldList.add(field);
                    }
                    String code = field.getAnnotation(Dict.class).dicCode();
                    String text = field.getAnnotation(Dict.class).dicText();
                    String table = field.getAnnotation(Dict.class).dictTable();

                    List<String> dataList;
                    String dictCode = code;
                    if (!StringUtils.isEmpty(table)) {
                        dictCode = String.format("%s,%s,%s", table, text, code);
                    }
                    dataList = dataListMap.computeIfAbsent(dictCode, k -> new ArrayList<>());
                    this.listAddAllDeduplicate(dataList, Arrays.asList(value.split(",")));
                }
                //date类型默认转换string格式化日期
                //update-begin--Author:zyf -- Date:20220531 ----for：【issues/#3629】 DictAspect Jackson序列化报错-----
                //if (JAVA_UTIL_DATE.equals(field.getType().getName())&&field.getAnnotation(JsonFormat.class)==null&&item.get(field.getName())!=null){
                //SimpleDateFormat aDate=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                // item.put(field.getName(), aDate.format(new Date((Long) item.get(field.getName()))));
                //}
                //update-end--Author:zyf -- Date:20220531 ----for：【issues/#3629】 DictAspect Jackson序列化报错-----
            }
            items.add(item);
        }

        //step.2 调用翻译方法，一次性翻译
        Map<String, List<DictModel>> translText = this.translateAllDict(dataListMap);

        //step.3 将翻译结果填充到返回结果里
        for (JSONObject record : items) {
            for (Field field : dictFieldList) {
                String code = field.getAnnotation(Dict.class).dicCode();
                String text = field.getAnnotation(Dict.class).dicText();
                String table = field.getAnnotation(Dict.class).dictTable();

                String fieldDictCode = code;
                if (!StringUtils.isEmpty(table)) {
                    fieldDictCode = String.format("%s,%s,%s", table, text, code);
                }

                String value = record.getString(field.getName());
                if (oConvertUtils.isNotEmpty(value)) {
                    List<DictModel> dictModels = translText.get(fieldDictCode);
                    if(dictModels==null || dictModels.size()==0){
                        continue;
                    }

                    String textValue = this.translDictText(dictModels, value);
                    log.debug(" 字典Val : " + textValue);
                    log.debug(" __翻译字典字段__ " + field.getName() + CommonConstant.DICT_TEXT_SUFFIX + "： " + textValue);

                    // TODO-sun 测试输出，待删
                    log.debug(" ---- dictCode: " + fieldDictCode);
                    log.debug(" ---- value: " + value);
                    log.debug(" ----- text: " + textValue);
                    log.debug(" ---- dictModels: " + JSON.toJSONString(dictModels));

                    record.put(field.getName() + CommonConstant.DICT_TEXT_SUFFIX, textValue);
                }
            }
        }
        return items;
    }
```



> 虽然可以支持 list集合和map 还有实体 如果集合里面又套了一层集合是不支持的 具体的解决办法下期再讲