
# Json转换

## 概念

### JSONObject

 json对象，就是一个键对应一个值，使用的是大括号{ }，如：{key:value}

### JSONArray

 json数组，使用中括号[ ],只不过数组里面的项也是json键值对格式的Json对象中添加的是键值对，JSONArray中添加的是Json对象



## List`<Bean>`转Json字符串

```java
JSONArray jsonArray = new JSONArray();
List<Bean>.forEach(bean -> {
      JSONObject jsonObject = new JSONObject();
      jsonObject.put("param1" , bean.getParam1());
      jsonObject.put("param2",bean.getParam2());
      jsonArray.add(jsonObject);
});
```



## Json字符串转List`<Bean>`

```java
if (Strings.isNullOrEmpty(this.JsonStr)) {
    return Collections.EMPTY_LIST;
}
JSONArray objects = JSONUtil.parseArray(this.JsonStr);
List<Bean> beans = JSONUtil.toList(objects, Bean.class);
return beans;
```