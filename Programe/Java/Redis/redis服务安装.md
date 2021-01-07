# Window

## redis服务开机自启动后台运行

1. 进入 DOS窗口
2. 在进入redis的安装目录
3. 输入：redis-server --service-install redis.windows.conf --loglevel verbose  ( 安装redis服务 )
4. 输入：redis-server --service-start   ( 启动服务 )
5. 输入：redis-server --service-stop （停止服务）