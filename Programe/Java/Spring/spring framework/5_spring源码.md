# 源码本地启动

## spring源码下载

地址:[https://github.com/spring-projects/spring-framework/tree/5.0.x](https://github.com/spring-projects/spring-framework/tree/5.0.x)

查看源码中gradle/wrapper/gradle-wrapper.properties中定义的gradle版本

![image-20210215001939965](https://gitee.com/Zeebrary/PicBed/raw/master/img/image-20210215001939965.png)

## gradle配置

下载地址：[https://services.gradle.org/distributions](https://services.gradle.org/distributions)

选择对应版本,本人直接使用浏览器下载，速度慢的离谱,右键复制下载链接，使用某雷下载会快很多。

下载完成之后，我们需要解压 Gradle 到指定的目录，例如“`F:\Programe\Gradle\gradle-6.8.1`”。

- 配置系统变量。

右键计算机 -> 属性 -> 高级系统设置 -> 环境变量，在系统变量区域，先点击“新建”输入变量名为：GRADLE_HOME，变量值为：`F:\Programe\Gradle\gradle-6.8.1`（根据自己的路径填写）；再找到 Path 环境变量，新增配置“`%GRADLE_HOME%\bin`”。

到这里 Gradle 的安装就已经完成了，接下来我们使用命令行执行工具，来测试一下 Gradle 安装是否成功。

打开一个新的 cmd 命令窗口，输入命令 `gradle -v`，如果出现版本消息，则说明配置成功

- Gradle 加速

和 Maven 的配置相同，我们可以给 Gradle 配置一个阿里的数据源，加速项目的构建（加上下载 Jar 包），找到配置文件 init.gradle，我的目录在 `F:\Programe\Gradle\gradle-6.8.1`，如果没有找到则新建一个 init.gradle 文件，之后添加如下配置：

```json
allprojects {
   repositories {
       maven {
           url "http://maven.aliyun.com/nexus/content/groups/public"
       }
   }
}
```

导入idea,ctrl+alt+s 搜索gradle,设置IDEA中gradle配置为使用我们自己下的gradle路径:`F:\Programe\Gradle\gradle-6.8.1`

开始build,等待idea构建完成，大概需要半个小时左右，等待即可。