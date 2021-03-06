## Cron

cron 组件 是 elves计划任务功能模块，基于Quartz框架实现的。 实现了elves 计划任务的添加和删除，根据计划任务内容定时向scheduler模块发起任务。

# 安装过程

## 编译

```
cd elves-scheduler
chmod +x ./control
./control build                                                 #二进制版本可以忽略编译过程
```

## 配置

```
mv conf/conf.properties.example conf/conf.properties            #复制配置文件
vim conf/conf.properties                                        #编辑配置文件
```

**./conf/conf.properties**

```
#Zookeeper Config
zookeeper.host=127.0.0.1                #Zookeeper地址
zookeeper.outTime=10000                 #Zookeeper超时时间        
zookeeper.root=/elves                   #Zookeeper ROOT地址

#MQ Basic Config

mq.ip       = 127.0.0.1                 #RabbitMQ IP
mq.port     = 5672                      #RabbitMQ 端口
mq.user     = admin                     #RABBITMQ 账号
mq.password =                           #RABBITMQ 密码       
mq.exchange = elves                     #Exchange 名称 


#jdbc conf
jdbc.type=mysql
jdbc.driver=com.mysql.jdbc.Driver
jdbc.pool.init=1
jdbc.pool.minIdle=3
jdbc.pool.maxActive=20
jdbc.testSql=SELECT 'x' FROM DUAL
jdbc.url=jdbc\:mysql\://127.0.0.1\:3306/elves_cron?characterEncoding=UTF-8&amp;useOldAliasMetadataBehavior=true&amp;zeroDateTimeBehavior=convertToNull
jdbc.username=mysql
jdbc.password=mysql
```

## 脚本参数

** ./control**

```
build|pack|start|stop|restart|status|version

build   : 运行后将执行mvn pakcge , 最终构建成至 bin
pack    : 将本模块打包(不包含配置文件与日志文件)
start   : 以nohup形式启动elves-{module}
stop    : 关闭elves-{module}
restart : 执行 stop & start
status  : 查看elves-{module}的运行状态
version : 查看当前模块的版本
```

## 组件-Quart介绍

cron 组件,各种计划任务的调度基于Quart框架实现（官方网站：[http://www.quartz-scheduler.org]()）

Quartz是一套非常高效的轻量级框架，这里提供一下Quartz的cron规则语法，如果想对Quartz有深入的了解，请自己查询相关资料学习。

| 位置 | 时间域 | 允许值 | 特殊值 |
| :--- | :--- | :--- | :--- |
| 1 | 秒 | 0-59 | , - \* / |
| 2 | 分钟 | 0-59 | , - \* / |
| 3 | 小时 | 0-23 | , - \* / |
| 4 | 日期 | 1-31 | , - \* ? / L W C |
| 5 | 月份 | 1-12 | , - \* / |
| 6 | 星期 | 1-7 | , - \* ? / L C \# |
| 7 | 年份（可选） | 1970－2099 | , - \* / |

## 数据库

con模块计划任务的存储使用mqsql实现，下面是SQL语句：

    CREATE TABLE `task_cron` (
      `id` varchar(16) NOT NULL COMMENT '主键Id',
      `agent_ip` varchar(15) NOT NULL COMMENT '客户端IP',
      `mode` enum('sap','sanp') NOT NULL COMMENT '模式(sap,sanp)',
      `app` varchar(32) NOT NULL COMMENT '模块',
      `func` varchar(32) NOT NULL COMMENT '方法',
      `param` text COMMENT '参数',
      `timeout` int(11) DEFAULT '0' COMMENT '超时时间',
      `proxy` varchar(15) DEFAULT NULL COMMENT '代理器',
      `rule` varchar(30) NOT NULL COMMENT '规则(quartz表达式)',
      `flag` tinyint(1) NOT NULL DEFAULT '0' COMMENT '状态(0:暂停,1:正常)',
      `create_time` datetime NOT NULL COMMENT '任务接收时间',
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='计划任务表'

## 组件服务

cron模块主要对openapi模块提供计划任务的操作接口，具体如下：

**RoutingKey : \*.cron**

### 服务提供列表

| **服务** | **类型** | **注释** |
| :--- | :--- | :--- |
| cronResult | rpc.cast | 接收Cron任务执行结果 |
| createCron | rpc.call | 添加Cron计划任务 |
| startCron | rpc.call | 开启Cron计划任务 |
| stopCron | rpc.call | 停止Cron计划任务 |
| deleteCron | rpc.call | 删除Cron计划任务 |
| cronList | rpc.call | 计划任务列表信息 |
| cronDetail | rpc.call | 计划任务详情 |

##### 服务使用列表

| **组件** | **服务** | **类型** | **注释** |
| :--- | :--- | :--- | :--- |
| scheduler | asyncJob | cast | 发起异步任务 |

### 服务提供详情

##### cronResult：

```
 接收消息：
 {
    "mqkey":"{组件}.cron.cronResult",
    "mqtype":"cast",
    "mqbody":{
        "flag"："true"
        "error":""
        "result":{
            "id":"9ad6af3b2e5d4c2d",
            "worker_flag":"1",
            "worker_message":"hello word!",
            "worker_costtime":"74"
        }
    }
}
```

##### createCron：

```
接收消息：
{
    "mqkey":"{组件}.cron.createCron",
    "mqtype":"call.88499CCA100F219",
    "mqbody":{
        "ip":"192.168.6.116",
        "mode":"NP",
        "app":"testApp",
        "func":"test",
        "param":"",
        "timeout":0,
        "proxy":"",
        "rule":"0/5 * * * * ?"
    }
}

回复消息："发送RoutingKey:88499CCA100F219"
{
    "mqkey":"cron.{组件}",
    "mqtype":"cast",
    "mqbody":{
        "flag": "true",
        "error": "",
        "id":"99499CCA100F214"
    }
}
```

##### startCron：

```
接收消息：
{
    "mqkey":"{组件}.cron.startCron",
    "mqtype":"call.198499CCA100F219",
    "mqbody":{
        "id":"DF499CCA100FABC"
    }
}

回复消息："发送RoutingKey:198499CCA100F219"
{
    "mqkey":"cron.{组件}",
    "mqtype":"cast",
    "mqbody":{
        "flag": "true",
        "error": ""
    }
}
```

##### stopCron：

```
接收消息：
{
    "mqkey":"{组件}.cron.stopCron",
    "mqtype":"call.88499CCA100FAAA",
    "mqbody":{
        "id":"88499CCA100FABC"
    }
}

回复消息："发送RoutingKey:88499CCA100FAAA"
{
    "mqkey":"cron.{组件}",
    "mqtype":"cast",
    "mqbody":{
        "flag": "true",
        "error": ""
    }
}
```

**deleteCron：**

```
接收消息：
{
    "mqkey":"{组件}.cron.deleteCron",
    "mqtype":"call.88499CCA100FAAA",
    "mqbody":{
        "id":"88499CCA100FABC"
    }
}

回复消息："发送RoutingKey:88499CCA100FAAA"
{
    "mqkey":"cron.{组件}",
    "mqtype":"cast",
    "mqbody":{
        "flag": "true",
        "error": ""
    }
}
```

##### cronList：

```
接收消息：
{
    "mqkey":"{组件}.cron.cronList",
    "mqtype":"call.88499CCA100FFFF",
    "mqbody":{
        "ip":"192.168.1.1"
    }
}

回复消息："发送RoutingKey:88499CCA100FFFF"
{
    "mqkey":"cron.{组件}",
    "mqtype":"cast",
    "mqbody":{
        "flag":"true",
        "error":"",
        "result":[
            "1ad6af3b2e5d4c2w",
            "2ad6af3b2e5d4c2e",
            "scd6af3b2e5d4c1d"
        ]
    }
}
```

##### cronDetail：

```
接收消息:
{
    "mqkey":"openapi.crn.cronDetail",
    "mqtype":"call.88499CCA100FAAA",
    "mqbody"{
        "id":"884DDCCA100F220"
    }
}

回复消息："发送RoutingKey:88499CCA100FAAA"
{
    "mqkey":"openapi.crn",
    "mqtype":"cast",
    "mqbody":{
        "flag":"true",
        "error":"",
        "result":{
            "ip":"127.0.0.1",
            "app":"apptest",
            "func":"hello",
            "param":"",
            "mode":"NP",
            "proxy":"",
            "rule":"0/5 * * * * ?",
            "last_exec_time":"2017-05-16 18:08:00",
            "last_exec_result" :{
                "flag": "true",
                "error":"",
                "result":{
                    "worker_flag"    :"1",
                    "worker_message" :"hello word!",
                    "worker_costtime":"100"
                }
            }
        }
    }
}
```



