# 批量命令行执行工具\(Bumblebee\)

## 简介

Bumblebee\(大黄蜂\)运维工具：基于Elves开源自动化运维开发平台实现的一款远程命令行执行工具，bumblebee分为Manager、Client、Script三部分。

## Manager

java-web项目，作为管理端和服务端，作为后台管理端可以进行权限分配和为客户端提供接口。  
![](/assets/bbb-index.png)

## Client

c\#编写的一款c/s客户端程序，调用bumblebee-manager提供的接口实现服务器的远程命令执行，并显示结果展示,最新版支持反射SHELL控制机器。

![](/assets/bbb-client.png)

## Script

python编写，依照Elves-App SDK开发，是Elves-Agent端执行的脚本文件，所有的命令通过Elves调用各个服务器上的脚本，执行并回复结果。

## 特征

* Client端命令全部通过manager端作为代理，调用Elves执行并返回数据，并提供命令操作日志
* Manager端提供指令黑名单，避免危险命令操作
* Manager端同样提供发送命令节点功能
* Manager端提供用户授权功能，用户只能操作授权的机器
* Manager端提供机器列表界面（可以自定义查询Elves机器在线状态）
* Manager端接口调用安全起见，加入签名验证

## 开源地址

[https://github.com/gy-games/bumblebee](https://github.com/gy-games/bumblebee)





