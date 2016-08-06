# Saas版后台任务服务端组件概述 #

## 业务需求 ##

应用程序中经常会用到跑定时任务的需求，比如定时垃圾回收。有关任务调度需求有时候很复杂，如每隔多长时间重复执行，一个任务在不同时间段执行等。

##解决方案##

本组件提供对Quartz的封装，提供独立的任务调度服务。避免业务系统直接配置Quartz的带来的性能消耗和集群环境的并发问题。 业务系统通过Rest服务添加任务，任务调动执行时回调业务系统的URL启动任务。

iuap-saas-dispatch-service组件功能包括添加、删除、暂停、重启任务。不仅提供了外部调用的Rest服务，并且组件本身也有完整的任务配置界面，包括任务调度，日志查询等功能。

## 功能说明 ##

本组件为后台任务组件的saas版本，支持后台任务组件的全部功能，同时支持下面功能
1.  支持按租户区分注册和执行任务
2. 	支持动态数据源
3. 	基于Auth组件认证用户
4.  支持在云工作台中集成任务的管理界面


# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，依赖Quartz框架,其对外提供的依赖方式如下：
```
	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-saas-dispatch-service</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>
```

${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 功能结构 ##

参考后台任务组件的使用文档，

## 部署结构 ##

独立的war包部署，使用云工作台时，两者为独立的web应用，任务的管理界面嵌入云工作台中做为入口使用

任务执行时调用应用服务执行任务


# 使用说明 #

## 组件包说明 ##

参加后台任务组件使用说明

##组件配置##

** 1:数据库配置 **

```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/ec?useUnicode=true&characterEncoding=utf-8
jdbc.catalog=ec
jdbc.username=root
jdbc.password=root
#定义初始连接数
jdbc.initialSize=0
#定义最大连接数
jdbc.maxActive=50
#定义最大空闲
jdbc.maxIdle=20
#定义最小空闲
jdbc.minIdle=0
#定义最长等待时间
jdbc.maxWait=30000

jdbc.pool.minEvictableIdleTimeMillis=60000
jdbc.pool.removeAbandoned=true
jdbc.pool.removeAbandonedTimeout=60

jdbc.pool.testOnBorrow=false
jdbc.pool.validationInterval=30000
jdbc.pool.testOnReturn=true
jdbc.pool.validationQuery=select 1
jdbc.pool.testWhileIdle=true
jdbc.pool.timeBetweenEvictionRunsMillis=30000
jdbc.pool.numTestsPerEvictionRun=-1

#动态dynamicDataSource   默认数据源jdbcDataSource
filejdbcDataSource=dynamicDataSource

```
** 注意，要使用动态数据源特性 ** ，需要设置filejdbcDataSource=dynamicDataSource，否则要设置为filejdbcDataSource=jdbcDataSource

** 2:与云工作台同时使用时，请在工作台的数据库中执行下面脚本 **

** 3:其它请参考后台任务组件的配置说明 **


## 示例工程说明：##

请参考后台任务组件的示例工程

## 开发说明 ##

### 通过Rest任务调度 ###

具体参见后台任务组件的使用说明

### 通过界面进行任务调度 ###

工作台的菜单中打开“任务调动”应用，

界面功能的具体操作过程参见后台任务组件的使用说明，


## API接口 ##

具体参见后台任务组件的使用说明
