# 事件中心sdk概述

## 业务需求

通常情况下，后台的业务系统为了提高系统的可靠性和支持动态扩展，一般会拆分成多个独立应用，应用之间相互独立，因此需要提供业务系统间事件通知机制。

## 解决方案

本组件通过rabbitmq进行事件消息队列管理，保证消息服务的可靠性。

## 功能说明

1.	基于消息发布／订阅的方式提供各个系统间的事件通知机制
2.	使用RebbitMQ分布式消息中间件实现消息的订阅发布，保证消息服务的可靠性。
3.	保证分布式环境下同一应用多个服务仅会接收一次事件。
5.	同一应用可以注册多个监听，按注册指定的顺序依次执行。
6.  支持执行失败后再次处理。


# 整体设计

## 依赖环境

组件采用Maven进行编译和打包发布，其对外提供的依赖方式如下：

```

		<dependency>
			<groupId>com.yonyou.iuap</groupId>
			<artifactId>iuap-event-sdk</artifactId>
			<version>${iuap.modules.version}<;/version>
		</dependency>

```

${iuap.modules.version} 为平台在maven私服上发布的组件的version。


## 工作流程

发送方：
1. 业务系统在启动时调用事件中心服务获取本系统相关的事件注册信息
2. 业务系统调用SDK提供的接口发送事件。
3. SDK将事件发送给事件中心的消息服务器。

监听方
1. 业务系统在启动时调用事件中心服务获取本系统相关的事件注册信息
2. 业务系统启动监听服务，定时获取消息服务器上的消息
3. 收到消息后，根据消息监听注册的信息，创建监听类实例，执行监听事件处理。
4. 如果处理失败，则通知消息服务器当前消息处理失败，下一次收到后再此尝试处理。
5. 处理成功，通知消息服务器处理成功。删除指定消息。

# 使用说明

##  配置说明

需要在eventConfig.properties中配置MQ及事件中心地址等相关信息。

```

#集群地址配置（可以配置多个以,分隔）
mq.address=172.20.14.133:5672,172.20.14.134:5672

#最大连接数
mq.maxtotal=10

#最大等待毫秒数
mq.maxwait=5000

#最大空闲连接
mq.maxidle=-1

#最小空闲连接
mq.minidle=2

#连接空闲的最小时间，达到此值后空闲连接将被移除
mq.softminevictableidletimemillis=60000

#每过这么多毫秒检查一次连接池中空闲的连接，把空闲时间超过minEvictableTimeMillis毫秒的连接断开，
#直到连接池中的连接数到minIdle为止
mq.timebetweenevictionrunsmillis=60000

#用户名和密码，这两项需要配置
mq.username=admin
mq.password=admin

#消息是否持久化
mq.durable=true

#监听队列的线程数，默认值为1
mq.customerNum=1

#集群名称，最好每个集群配置一个唯一值(对应在事件中心注册的应用编码，node_code)
msg.clusterCode=ANONYMOUS_CLUSTER3

#事件管理节点的服务地址(需要修改为指定的地址，目前使用的是uas.yyuap.com)
event.manager.url=http://uas.yyuap.com/event/eventmanager/eventinfo/index.do


```

**注：eventConfig.properties配置文件也支持通过系统变量和环境变量传入，格式为event-mqConfig-filePath=配置文件路径名。**

##  事件中心注册说明

注册相关事件信息，按以下内容填写，填写好后发给事件中心管理者进行注册。

### 事件注册应用服务表 pub_event_node

<table>
   <tr>
      <td>名称</td>
      <td>代码 </td>
      <td>注释</td>
   </tr>
   <tr>
      <td>应用编码</td>
      <td>node_code</td>
      <td>应用编码，同在事件中心sdk中配置的msg.clusterCode</td>
   </tr>
   <tr>
      <td>应用名称</td>
      <td>node_name</td>
      <td></td>
   </tr>
   <tr>
      <td>描述</td>
      <td>note</td>
      <td></td>
   </tr>
   <tr>
      <td>最后更新时间</td>
      <td>last_time</td>
      <td></td>
   </tr>
   <tr>
      <td>版本</td>
      <td>version</td>
      <td></td>
   </tr>
</table>

### 事件类型pub_eventtype

<table>
   <tr>
      <td>名称</td>
      <td>代码</td>
      <td>注释</td>
   </tr>
   <tr>
      <td>主键</td>
      <td>id</td>
      <td></td>
   </tr>
   <tr>
      <td>事件源编码</td>
      <td>sourceid</td>
      <td>一般用来描述产生事件的对象或单据。例如用户，租户等</td>
   </tr>
   <tr>
      <td>事件源名称</td>
      <td>sourcename</td>
      <td></td>
   </tr>
   <tr>
      <td>事件类型编码</td>
      <td>eventtypecode</td>
      <td>一般用于描述事件对象或单据的具体行为。例如：新增、修改、删除等</td>
   </tr>
   <tr>
      <td>事件类型名称</td>
      <td>eventtypename</td>
      <td></td>
   </tr>
   <tr>
      <td>事件源所属应用</td>
      <td>owner</td>
      <td>产生事件的应用，具体为pub_event_node中的，可以为ALL（全部应用，例如安全日志之类的公共组件的事件）</td>
   </tr>
   <tr>
      <td>详细说明</td>
      <td>note</td>
      <td>主要对事件的消息内容格式做出说明，便于后续监听插件安格式获取数据。</td>
   </tr>
   <tr>
      <td>最后更新时间</td>
      <td>last_time</td>
      <td></td>
   </tr>
   <tr>
      <td>版本</td>
      <td>version</td>
      <td></td>
   </tr>
</table>

###	事件通知业务插件表pub_eventlistener

<table>
   <tr>
      <td>名称</td>
      <td>代码</td>
      <td>注释</td>
   </tr>
   <tr>
      <td>主键</td>
      <td>id</td>
      <td></td>
   </tr>
   <tr>
      <td>事件类型主键</td>
      <td>event_type_id</td>
      <td></td>
   </tr>
   <tr>
      <td>插件名称</td>
      <td>name</td>
      <td></td>
   </tr>
   <tr>
      <td>插件名称多语编码</td>
      <td>namei18n</td>
      <td></td>
   </tr>
   <tr>
      <td>插件全类名</td>
      <td>implclassname</td>
      <td>使用SDK时注册，为sdk调用的事件处理插件的全类名，插件需要实现IBusinessEventListener接口</td>
   </tr>
   <tr>
      <td>服务请求地址url</td>
      <td>url</td>
      <td>不使用SDK时注册，为事件发生时调用的监听处理服务地址，需要满足下面要求</td>
   </tr>
   <tr>
      <td>插件所属应用</td>
      <td>owner</td>
      <td>事件监听插件运行的产品</td>
   </tr>
   <tr>
      <td>插件功能备注</td>
      <td>note</td>
      <td></td>
   </tr>
   <tr>
      <td>插件执行顺序号</td>
      <td>operindex</td>
      <td>同一业务系统内顺序号唯一，同一业务系统内事件按插件顺序执行</td>
   </tr>
   <tr>
      <td>系统节点</td>
      <td>node_code</td>
      <td></td>
   </tr>
   <tr>
      <td>是否启用</td>
      <td>enabled</td>
      <td>Y/N</td>
   </tr>
   <tr>
      <td>最后更新时间</td>
      <td>last_time</td>
      <td></td>
   </tr>
   <tr>
      <td>版本</td>
      <td>version</td>
      <td></td>
   </tr>
</table>

** 注意：**

1.	上述注册信息通过统一提供的事件中心进行注册。

2.	业务系统在启动时，调用事件中心的服务获取本系统关心的事件插件注册信息，同时启动监听服务。

3.	当事件中心注册新的事件时，会通知业务系统的事件插件信息缓存进行更新。

## 开发说明

** 1.服务启动 **

在应用服务启动时调用EventDispatcher.startListener()，启动事件监听线程，如果只发送事件可以不用启动该服务。

** 2.监听类实现IBussinessListener **

```
public class LocalEventPluginImpl implements IBussinessListener{
	/**
	 * 事件响应
	 */
	public void doAction(BusinessEvent businessEvent) {
		System.out.println("doAction, time="+(System.currentTimeMillis()-Long.valueOf(String.valueOf(businessEvent.getUserObject())))+",event="+businessEvent);
	}

	public void doBackAction(BusinessEvent businessEvent) {
		System.out.println("doBackAction, event="+businessEvent);
	}
}
```


## 使用示例

假定一个事件用户新增前事件：

** 1.	新增处理插件 **

需要实现IBussinessListener或者ISecurityBussinessListener接口，新增一个处理插件：
com.yonyou.iuap.event.test.UserPluginImpl


**	2.注册监听 **
由于注册中心还没有添加页面功能，需要手动在数据库中注册事件，请把要注册的信息发给事件中心管理者进行注册。

注册节点信息，加粗文字与eventConfig.properties中的集群名称保持一致；

```

INSERT INTO pub_event_node (node_code, node_name, note, last_time, version) VALUES ('<b>ANONYMOUS_CLUSTER</b>', '测试节点', null, '2016-03-02 18:18:23', 1);

```

注册事件类型

```

INSERT INTO pub_eventtype (id, sourceid, sourcename, sourcenamei18n, eventtypecode, eventtypename, eventtypenamei18n, owner, note, last_time, version) VALUES (1, '<b>USER</b>', '用户', 'USER', '<b>ADD_AFTER</b>', '新增后', 'ADD_AFTER', 'ALL', null, null, 1);

```

注册事件监听插件

```

INSERT INTO pub_eventlistener (id, event_type_id, name, namei18n, implclassname, owner, note, operindex, node_code, enabled, last_time, version) VALUES (1, 1, '测试用户新增', 'testUserAdd', '<b>com.yonyou.iuap.event.test.UserPluginImpl</b>', 'ALL', null, 1, '<b>ANONYMOUS_CLUSTER</b>', 'Y', null, 1);

```

**	3.事件触发 **
```

    BusinessEvent event = new BusinessEvent("USER","ADD_AFTER",
						"tenantCode"/*租户信息*/,
						userObject/*事件发送的数据,建议是json串*/);
		EventDispatcher.fireEvent(event);

```

## API接口

### 发送事件

**描述**  

发送事件

**请求方法**  

com.yonyou.iuap.event.EventDispatcher.fireEvent(event)

**请求方式**  

工具类直接调用  

**请求参数说明**  

com.yonyou.iuap.event.common.BusinessEvent 这个实体类的对象event，该对象里的具体描述如下：

<table>
	<tr>
		<td>参数字段</td>
		<td>必选</td>
		<td>类型</td>
		<td>长度限制</td>
		<td>说明</td>
	</tr>
	<tr>
		<td>sourceId</td>
		<td>true</td>
		<td>String</td>
		<td></td>
		<td>事件源编码</td>
	</tr>
	<tr>
		<td>eventType</td>
		<td>true</td>
		<td>String</td>
		<td></td>
		<td>事件类型编码</td>
	</tr>
	<tr>
		<td>tenantCode</td>
		<td>false</td>
		<td>String</td>
		<td></td>
		<td>租户编码</td>
	</tr>
	<tr>
		<td>userObject</td>
		<td>true</td>
		<td>Serializable</td>
		<td></td>
		<td>要发送的用户数据，建议为json数据</td>
	</tr>
</table>

**返回参数说明**  

无
