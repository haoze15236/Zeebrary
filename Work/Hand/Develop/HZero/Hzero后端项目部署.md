# 数据库初始化

```sql
#创建hzero用户
create user 'hzero'@'%' identified by 'hzero';
```

# 应用搭建

## 下载应用

- 开放平台选配:[https://open.hand-china.com/publish-center/service-match/create](https://open.hand-china.com/publish-center/service-match/create)

下载应用代码到本地之后，导入IDEA,将每个服务都加入到maven里面,等待自动下载依赖

![image-20210225224205433](C:\Users\zee\AppData\Roaming\Typora\typora-user-images\image-20210225224205433.png)

## 启动应用

本地部署教程:[https://open.hand-china.com/document-center/doc/product/10002/10209?doc_id=35263#%E7%89%88%E6%9C%AC%E5%8D%87%E7%BA%A7](https://open.hand-china.com/document-center/doc/product/10002/10209?doc_id=35263#%E7%89%88%E6%9C%AC%E5%8D%87%E7%BA%A7)

# 错误合集

hzero-resource执行groovy脚本初始化数据库时,执行到一半报错某个表不存在：

```mysql
ALTER TABLE hzero_platform.IAM_LABEL ADD FD_LEVEL VARCHAR(32) NOT NULL COMMENT "层级"
```

**问题猜测：**单独拿这个语句到数据库中执行,发现同样的报错,表名修改成小写,执行正常。

**解决办法：**修改数据库设置:`/etc/my.cnf`

