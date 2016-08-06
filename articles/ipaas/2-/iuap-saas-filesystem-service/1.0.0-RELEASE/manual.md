# Saas版附件系统服务组件概述

## 业务需求

提供业务系统与文件服务之间的管理功能，支持单据的上传多个附件的管理功能。


## 功能说明

本组件为附件管理组件的saas版本，支持后台任务组件的全部功能，同时支持下面特性

1. 支持按租户隔离
2. 支持动态数据源
3. 基于Auth组件认证用户


# 整体设计

## 依赖环境

```
<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-saas-filesystem-service</artifactId>
	  <version>${iuap.modules.version}</version>
	  <type>war</type>
	</dependency>
```

${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 功能结构

本组件为附件管理组件的saas版本，目前支持FastDFS，阿里的OSS服务。

# 使用说明

## 配置说明：

** 1. jdbc.properties：数据库配置文件 **

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

** 2. applicationContext-shiro.xml配置 **

认证组件的相关配置，在使用云工作台时，可使用组件的默认配置，不需要修改。如果业务应用的配置与工作台默认配置不同时，需要根据配置手工修改war包中的WEB-INF/classes/applicationContext-shiro.xmlz中的配置。具体配置方式参考认证组件的说明。


** 3. 其它配置，请参考附件管理组件的配置说明 **




## 示例工程说明：##

请参考附件管理组件的示例工程，

## 开发步骤 ##

### 配置说明

1. 本组件的使用方法同附件管理组件，请参考附件管理组件的使用说明。

2. 本组件使用认证组件做为身份认证，需要通过cookies传递用户、token、租户等相关信息，因此需要业务服务同时使用认证组件（保持配置相同），切要于业务服务部署在相同域名下使用。

### API接口说明

参考附件管理组件的使用说明
