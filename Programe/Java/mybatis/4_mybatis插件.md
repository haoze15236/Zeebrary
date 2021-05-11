# [mybatis-plugins](https://mybatis.org/mybatis-3/zh/configuration.html#plugins)

mybatis预留了4个拓展点来对它的行为做出合理的修改,通过合理配置pluguns插件，即可做到如下内容的修改。

![image-20210511234251752](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210511234251752.png)分页功能

插件一般用于，查询数据分页,性能监控，公共字段统一赋值等。

## 源码解析

# [mybatis-pageHelper](https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md)

测试：

```java
@Test
public void testPageHelper(){
    try (SqlSession session = sqlSessionFactory.openSession(true)) {
        ExpReportLineMapper mapper = session.getMapper(ExpReportLineMapper.class);
        PageHelper.startPage(1,2);
        System.out.println(mapper.selectMap());
        PageHelper.startPage(1,2);
        System.out.println(mapper.selectOne(1));
        PageHelper.startPage(1,2);
        System.out.println(mapper.selectLines(1));
    }
}
```

输出结果：

```
{1={DESCRIPTION=测试1, EXP_REPORT_LINE_ID=1, PAT_RATIO=10, EXP_REPORT_HEADER_ID=1}, 2={DESCRIPTION=测试2, EXP_REPORT_LINE_ID=2, PAT_RATIO=20, EXP_REPORT_HEADER_ID=1}}

BillLine(billLineId=1, billHeaderId=1, description=测试1)ExpReportLine{payRatio=10}

Page{count=true, pageNum=1, pageSize=2, startRow=0, endRow=2, total=4, pages=2, reasonable=false, pageSizeZero=false}[BillLine(billLineId=1, billHeaderId=1, description=测试1)ExpReportLine{payRatio=10}, BillLine(billLineId=2, billHeaderId=1, description=测试2)ExpReportLine{payRatio=20}]
```

可以看到，返回的如果是List集合，会被自动包装到`com.github.pagehelper.Page`类中，所以返回的是Page对象。同时在官方说明中，有提到可以再把Page封装到`com.github.pagehelper.PageInfo`里以获取更多友好的支持。

```java
PageInfo page = new PageInfo(list);
```

