# azkaban 调度框架
## 一 概述
### 1.1 什么是 Azkaban

  Azkaban 是由 Linkedin 公司推出的一个批量工作流任务调度器，主要用于在一个工作流
内以一个特定的顺序运行一组工作和流程，它的配置是通过简单的 key:value 对的方式，通
过配置中的 Dependencies 来设置依赖关系。Azkaban 使用 job 配置文件建立任务之间的依赖
关系，并提供一个易于使用的 web 用户界面维护和跟踪你的工作流。

### 1.2 为什么需要工作流调度系统

1）一个完整的数据分析系统通常都是由大量任务单元组成：
Shell 脚本程序，Java 程序，MapReduce 程序、Hive 脚本等
2）各任务单元之间存在时间先后及前后依赖关系
3）为了很好地组织起这样的复杂执行计划，需要一个工作流调度系统来调度执行；
例如，我们可能有这样一个需求，某个业务系统每天产生 20G 原始数据，我们每天都
要对其进行处理，处理步骤如下所示：
1) 通过 Hadoop 先将原始数据上传到 HDFS 上（HDFS 的操作）；
2) 使用 MapReduce 对原始数据进行清洗（MapReduce 的操作）；
3) 将清洗后的数据导入到 hive 表中（hive 的导入操作）；
4) 对 Hive 中多个表的数据进行 JOIN 处理，得到一张 hive 的明细表（创建中间表）；
5) 通过对明细表的统计和分析，得到结果报表信息（hive 的查询操作）；

### 1.3 Azkaban 特点

1) 兼容任何版本的 hadoop
2) 易于使用的 Web 用户界面
3) 简单的工作流的上传
4) 方便设置任务之间的关系
5) 调度工作流
6) 模块化和可插拔的插件机制
7) 认证/授权(权限的工作)
8) 能够杀死并重新启动工作流
9) 有关失败和成功的电子邮件提醒

### 1.4 常见工作流调度系统
1）简单的任务调度：直接使用 crontab 实现；
2）复杂的任务调度：开发调度平台或使用现成的开源调度系统，比如 ooize、azkaban 等

### 1.5 Azkaban 的架构

Azkaban 由三个关键组件构成：

1) AzkabanWebServer：AzkabanWebServer 是整个 Azkaban 工作流系统的主要管理者，
它用户登录认证、负责 project 管理、定时执行工作流、跟踪工作流执行进度等一
系列任务。
2) AzkabanExecutorServer：负责具体的工作流的提交、执行，它们通过 mysql 数据库
来协调任务的执行。
3) 关系型数据库（MySQL）：存储大部分执行流状态，AzkabanWebServer 和
AzkabanExecutorServer 都需要访问数据库。

### 1.6 Azkaban 下载地址
下载地址:http://azkaban.github.io/downloads.html

# 二 Azkaban 安装部署
## 2.1 安装前准备
1) 将 Azkaban Web 服务器、Azkaban 执行服务器、Azkaban 的 sql 执行脚本及 MySQL 安
装包拷贝到 hadoop102 虚拟机/opt/software 目录下
a) azkaban-web-server-2.5.0.tar.gz
b) azkaban-executor-server-2.5.0.tar.gz
c) azkaban-sql-script-2.5.0.tar.gz
d) mysql-libs.zip
  
  
2) 选择 Mysql 作为 Azkaban 数据库，因为 Azkaban 建立了一些 Mysql 连接增强功能，以
方便 Azkaban 设置。并增强服务可靠性。

## 2.2 安装 Azkaban
 tar -zxvf azkaban-web-server-2.5.0.tar.gz -C /opt/module/azkaban/
 tar -zxvf azkaban-executor-server2.5.0.tar.gz -C /opt/module/azkaban/
 tar -zxvf azkaban-sql-script-2.5.0.tar.gz-C /opt/module/azkaban/
 
 mysql> create database azkaban;
 mysql> use azkaban;
 mysql> source /opt/module/azkaban/azkaban-2.5.0/create-all-sql2.5.0.sql

## 2.3 生成密钥对和证书
Keytool 是 java 数据证书的管理工具，使用户能够管理自己的公/私钥对及相关证书。
-keystore 指定密钥库的名称及位置(产生的各类信息将存在.keystore 文件中)
-genkey(或者-genkeypair) 生成密钥对
-alias 为生成的密钥对指定别名，如果没有默认是 mykey
-keyalg 指定密钥的算法 RSA/DSA 默认是 DSA

1）生成 keystore 的密码及相应信息的密钥库
```
 keytool -keystore keystore -alias jetty -
genkey -keyalg RSA
输入密钥库口令:
再次输入新口令:
您的名字与姓氏是什么?
 [Unknown]:
您的组织单位名称是什么?
 [Unknown]:
您的组织名称是什么?
 [Unknown]:
您所在的城市或区域名称是什么?
 [Unknown]:
您所在的省/市/自治区名称是什么?
 [Unknown]:
该单位的双字母国家/地区代码是什么?
 [Unknown]:
CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown 是否
正确?
 [否]: y
输入 <jetty> 的密钥口令
 (如果和密钥库口令相同, 按回车):
 再次输入新口令:
```
注意：
密钥库的密码至少必须 6 个字符，可以是纯数字或者字母或者数字和字母的组合等等
密钥库的密码最好和<jetty> 的密钥相同，方便记忆
2）将 keystore 拷贝到 azkaban web 服务器根目录中
  
  ## 2.4 时间同步配置
  先配置好服务器节点上的时区
1） 如果在/usr/share/zoneinfo/这个目录下不存在时区配置文件 Asia/Shanghai，就要用
tzselect 生成
## 2.5 配置文件
### 2.5.1 Web 服务器配置
1）进入 azkaban web 服务器安装目录 conf 目录，打开 azkaban.properties 文件
```
#Azkaban Personalization Settings
#服务器 UI 名称,用于服务器上方显示的名字
azkaban.name=Test
#描述
azkaban.label=My Local Azkaban
#UI 颜色
azkaban.color=#FF3601
azkaban.default.servlet.path=/index
#默认 web server 存放 web 文件的目录
web.resource.dir=/opt/module/azkaban/server/web/
#默认时区,已改为亚洲/上海 默认为美国
default.timezone.id=Asia/Shanghai
#Azkaban UserManager class
user.manager.class=azkaban.user.XmlUserManager
#用户权限管理默认类（绝对路径）
user.manager.xml.file=/opt/module/azkaban/server/conf/azkaban-users.xml
#Loader for projects
#global 配置文件所在位置（绝对路径）
executor.global.properties=/opt/module/azkaban/executor/conf/global.pro
perties
azkaban.project.dir=projects
#数据库类型
database.type=mysql
#端口号
mysql.port=3306
#数据库连接 IP
mysql.host=hadoop102
#数据库实例名
mysql.database=azkaban
#数据库用户名
mysql.user=root
#数据库密码
mysql.password=000000
#最大连接数
mysql.numconnections=100
# Velocity dev mode
velocity.dev.mode=false
# Azkaban Jetty server properties.
# Jetty 服务器属性.
#最大线程数
jetty.maxThreads=25
#Jetty SSL 端口
jetty.ssl.port=8443
#Jetty 端口
jetty.port=8081
#SSL 文件名（绝对路径）
jetty.keystore=/opt/module/azkaban/server/keystore
#SSL 文件密码
jetty.password=000000
#Jetty 主密码与 keystore 文件相同
jetty.keypassword=000000
#SSL 文件名（绝对路径）
jetty.truststore=/opt/module/azkaban/server/keystore
#SSL 文件密码
jetty.trustpassword=000000
# Azkaban Executor settings
executor.port=12321
# mail settings
mail.sender=
mail.host=
job.failure.email=
job.success.email=
lockdown.create.projects=false
cache.directory=cache
```

3）web 服务器用户配置
在 azkaban web 服务器安装目录 conf 目录，按照如下配置修改 azkaban-users.xml 文件，
增加管理员用户
 vim azkaban-users.xml
 ```
<azkaban-users>
<user username="azkaban" password="azkaban" roles="admin"
groups="azkaban" />
<user username="metrics" password="metrics" roles="metrics"/>
<user username="admin" password="admin" roles="admin,metrics"/>
<role name="admin" permissions="ADMIN" />
<role name="metrics" permissions="METRICS"/>
</azkaban-users>

```
# 其他excutor  配置类似

## 启动
 bin/azkaban-executor-start.sh
 bin/azkaban-web-start.sh

# 三 Azkaban 实战
## 3.1 单一 job 案例
```
 vim first.job
 #first.job
type=command
command=echo 'this is my first job'
```
* 打包成 zip 格式
```
zip first.zip first.job
```
