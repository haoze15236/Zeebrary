# JRebel
## 安装和使用

**安装教程：**[https://blog.csdn.net/qq_40722827/article/details/99976121](https://blog.csdn.net/qq_40722827/article/details/99976121)
**使用教程：**[https://www.jianshu.com/p/58b1a7551ad7

## 错误
**1. unable ping server localhost:1099**
a. 查看tomcat版本与jdk版本是否一致
b. 查看tomcat启动vm options 与JRebel 的vm options 是否一致[https://blog.csdn.net/zixiao217/article/details/104577095](https://blog.csdn.net/zixiao217/article/details/104577095)
此错误的原因是由于文件路径中存在中文,及时采用B方案解决问题之后,jrebel由于无法写入jrebel.log文件,依然无法热部署成功。
笔者最后把修改win10用户名中文修改成英文之后,热部署成功。