# Node.js

## 什么是Node.js?

Node.js是一个基于Chrome V8引擎的JavaScript运行环境。Node.js有以下作用

1. 前端开发环境

   **Webpack**：配合编译前端项目

   **Npm插件**:绑定在Node.js中,

2. 服务端动态编程语言

   **Server**:Node.js可以搭建前端web Server

## Node.js安装

[Node.js中文网](http://nodejs.cn/download/)

## Node.js使用

```shell
# 查看版本
node -v
# 进入node;相当于进入chrome浏览器console窗口
node
```



# Npm使用

## 什么是Npm?

Npm是个包管理器，可以找到各种各样的插件

## Npm使用

```shell
#查看版本
npm -v
#初始化
npm init
#局部安装
npm install 
#全局安装
npm install -g
#npm添加用户
npm adduser
#推送代码到npm平台
npm publish
#设置npm镜像
npm config set registry https://registry.npmjs.org/
#设置使用淘宝镜像
npm config set registry https://registry.npm.taobao.org/
#删除包
npm unpublish [包名] -force
```



# Yarn使用