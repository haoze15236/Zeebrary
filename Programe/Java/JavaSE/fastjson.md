# 基础概念



## JSONObject

继承了`Map<String, Object>`，对应JSON字符串中的`{}`部分。

key一直都是String类型

value有可能是各种对象,所以是Object类型。

## JSONArray

继承了` List<Object>`,对应json字符串中`[ ]`的部分。

Object一般是JSONObject

```json
//此字符串转换成JSONObject之后,Map里面有
//四个key[sign,pid,amount,lines]
//四个value,类型分别是[String,String,int,JSONArray]
{
    "sign": "5d192c66eaabd9a21453bb16b71cb38e",
    "pid": "4d4b05f56d7633aacff351a243390ddf",
    "amount": 1,
    "lines": [
        {
            "lineNumber":"10"
        },
        {
            "lineNumber":"20"
        }]
}
```

## JSON

转换类，用于String - - - JSONObject/JSONArray- - - JavaBean 之间的转换。

# 使用方式

```java
//String--->JSONObject
JSONObject jsonobj = JSON.parseObject(msg);
//
//对于json中[]的部分只能转换成JSONArray对象
JSONArray data = jsonobj.getJSONArray("data");
//JSONArray转对象集合
List<T> requestDataList=data.toJavaList(T.class);
List<T> requestDataList2=JSONArray.parseArray(data.toJSONString(),T.class);
//对象转String
RequestData req = new RequestData("1","test","2","3");
String objson = JSON.toJSONString(req);
```

# 错误合集

```java
com.alibaba.fastjson.JSONException: default constructor not found.
```

原因：反序列化时为对象时，必须要有默认无参的构造函数，否则会报异常: