# 桌面

1. **Fences-栅栏式桌面整理工具**

   收费软件，国内官网:[http://www.fences.vip/](http://www.fences.vip/)

   电脑启动时有时会出现桌面加载慢的问题,主要是Win10采用了延迟技术，这个技术主要是为触摸而设计的，如果想加快快捷方式的显示速度，我们可以取消Win10的延迟启动。
   1、首先打开注册表编辑器。可以通过Windows徽标键+R来打开运行窗口，输入“regedit”然后按回车键来启动。也可以直接在Win10预览版的开始菜单中进行搜索。

   2、依次在注册表左侧中展开如下路径：HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Serialize，大家很有可能找不到最后的Serialize项，没关系，我们可以在Explorer下面新建该项。直接在Explorer项上点击鼠标右键，然后选择“新建”——“项”，然后将其命名为“Serialize”即可。

   3、在Serialize项右侧窗口中，通过点击鼠标右键，新建一个DWORD(32位)值，名称为”StartupDelayInMSec“（不包含引号），然后通过双击该键值将其改为”0“（不包含引号）后，确定后重启即可。

2. **TranslucentTB-任务栏透明**

   可直接从microsfot store 下载中文汉化版