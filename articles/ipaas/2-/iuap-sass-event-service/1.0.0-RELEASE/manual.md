# 事件中心使概述

##业务需求

通常情况下，后台的业务系统为了提高系统的可靠性和支持动态扩展，一般会拆分成多个独立应用，应用之间相互独立，因此需要提供业务系统间事件通知机制。

##解决方案

本组件通过rabbitmq进行事件消息队列管理，保证消息服务的可靠性。

## 功能说明

1.	基于消息发布／订阅的方式提供各个系统间的事件通知机制
2.	使用RebbitMQ分布式消息中间件实现消息的订阅发布，保证消息服务的可靠性。
3.  业务系统通过调用事件中心reset服务发送事件。
4.  事件中心收到事件后，调用业务系统发布的reset服务执行事件处理。
5.  业务系统服务无法访问或调用失败时，支持等待一段时间重新发送的机制。

# 整体设计

## 依赖环境 ##

组件采用Maven进行编译和打包发布，其对外提供的依赖方式如下：
```
	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-event-service</artifactId>
	  <version>${iuap.modules.version}</version>
	  <type>war</type>
	</dependency>
```

${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 工作流程

发送方：
1. 业务系统调用事件中心提供的服务发送消息
2. 事件中心接收到消息后将消息放入队列中。

监听方
1. 事件中心监听器接收队列消息后，调用业务注册的url地址执行事件处理。
2. 调用成功后，删除队列中的消息。
3. 调用失败后，间隔一段时间再次尝试发送。
4. 尝试超过一定次数后，记录处理失败日志，删除消息。

# 使用说明

## 事件中心的部署（使用公有服务时，不需要）

事件中心以war包的形式提供，部署的时候需要配置两个配置文件：

** 1. 关于数据库的配置文件：eventcenter-application.properties **

```

#jdbc.url的值是jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8
jdbc.username=
jdbc.password=

```

** 2. 关于MQ的配置文件:eventcenterConfig.properties, **

```

#集群地址配置
mq.address=172.20.14.133:5672

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

#用户名和密码，这两项需要配置，mq.username=admin，mq.password=admin
mq.username=
mq.password=

#消息是否持久化
mq.durable=true

#监听队列的线程数，默认值为1
#mq.customerNum=1

#事件管理节点的服务地址(需要修改)
event.manager.url=http://localhost:8080/event/eventmanager/eventinfo/index.do

```

其中，主要需要配置mq.address与event.manager.url，其它可以直接使用默认值

- mq.address:RabbitmqMQ服务器地址，使用MQ集群时可以配置多个，以","隔开。
	例如：mq.address=172.20.14.133:5672,172.20.14.134:5672

- event.manager.url:当前事件中心服务地址，/eventmanager/eventinfo/index.do不需要修改，将http://localhost:8080/event修改为本地服务器的ip端口及事件中心部署的应用路径。


** 支持通过环境变量传入路径的方式，key值是eventcenter-mqConfig-filePath，即eventcenter-mqConfig-filePath=配置文件路径。 **


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
      <td>监听服务方签名的key</td>
      <td>client_key</td>
      <td>事件中心调用业务监听服务时，根据此key进行加签</td>
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

** 1. sdk与通过事件中心直接调用url的方式，不能同时使用，否则会收到两次事件。 **

** 2. url注册的服务要满足下面API中要求说明 **


### 开发说明

事件中心的服务通知方式，主要是提供发送事件和通知事件服务，应用系统通过一个request请求服务地址进行事件的发送和接收，并且，我们提供了加签功能，以保证公网服务的安全。

#### 事件中心通过调用服务发送事件：

业务系统发起事件时如下方式

** 1.将需要发送事件的内容通过以下地址和参数传递： **

（1）请求地址：https://uas.yyuap.com/event/eventmanager/sendEventMsg.do

（2）请求需要传入的参数：String sourceID，String eventType，String tenantCode，String userObject(事件的消息内容，建议为json格式)。

**  注： 当不使用公共服务时，请将https://uas.yyuap.com/event/替换为实际部署的事件中心服务地址

** 2.进行加签：要调用我们的服务发送事件，需要进行加签，应用系统需要通过邮件向事件中心服务管理人员申请加签的密钥。**

加签时需要配置以下文件：application.properties和authfile.txt，具体参考示例工程。

** application.properties文件中增加如下配置项：**

```

bpm.client.credential.path=D:/etc/authfile.txt

```

** 其中authfile.txt文件配置说明如下： **

```

appId=3c8242c9467d059f9b718a890c9932ca
key=test

```

key配置为事件中心提供的密钥，appId配置为应用系统在事件中心应用注册中的应用编码，注意此处于加签算法中的服务调用参数appid的值应保持一致。上述两个参数为通过邮件向事件中心管理人员申请注册时，由事件中心提供。需要与事件中心的应用注册表中的数据相同。

** 3.具体的加签算法，请参考示例工程，主要核心代码如下：**

```

public class SenderSign {

	/**
	 * HttpClient Post请求签名，最常规的用法
	 *
	 * @throws UAPSecurityException
	 * @throws ClientProtocolException
	 * @throws IOException
	 */
	@Test
	public void testPostSign() throws UAPSecurityException,
			ClientProtocolException, IOException {

		String ts = System.currentTimeMillis() + "";
		String url = "http://localhost:8080/event/eventmanager/sendEventMsg.do?"
				+"appId=ANONYMOUS_CLUSTER"
				+ "&ts=" + ts;

		// 3、使用HttpClient发起http请求
		HttpClient httpClient = new DefaultHttpClient();
		HttpPost httpPost = new HttpPost(url);

		Map<String, String> paramsMap = new HashMap<String,String>();
		paramsMap.put("sourceID", "USER");
		paramsMap.put("eventType", "ADD_AFTER");
		paramsMap.put("userObject", String.valueOf(System.currentTimeMillis()));

		List<NameValuePair> list = new ArrayList<NameValuePair>();
		Iterator iterator = paramsMap.entrySet().iterator();
		while (iterator.hasNext()) {
			Entry<String, String> elem = (Entry<String, String>) iterator
					.next();
			list.add(new BasicNameValuePair(elem.getKey(), elem.getValue()));
		}
		if (list.size() > 0) {
			UrlEncodedFormEntity entity = new UrlEncodedFormEntity(list,
					"UTF-8");
			httpPost.setEntity(entity);
		}

		// 6、根据完整的URL、表单参数Map以及content-length的值生成SignProp对象
		SignProp signProp = SignPropGenerator.genSignProp(url);
		signProp.setPostParamsStr(PostParamsHelper.genParamsStrByMap(paramsMap));
		if(httpPost.getEntity().isChunked() || httpPost.getEntity().getContentLength()<0) {
			signProp.setContentLength(-1);
		} else {
			signProp.setContentLength(httpPost.getEntity().getContentLength());
		}
		// 7、根据SignProp对象构造签名字符串，getSigner时候传入提供服务的类型，以查找不同的客户端证书
		String sign = ClientSignFactory.getSigner("bpm").sign(signProp);
		// 8、将签名字符串加入到http请求的header中
		httpPost.setHeader(AuthConstants.PARAM_DIGEST, sign);

		// 9、发送http post请求，并得到响应结果
		HttpResponse response = httpClient.execute(httpPost);
		String result = "";
		if (response != null) {
			HttpEntity resEntity = response.getEntity();
			if (resEntity != null) {
				result = EntityUtils.toString(resEntity, "UTF-8");
				System.out.println(result);
			}
		}
	}

}

```


#### 业务系统监听事件实现

** 1.在pub_eventlistener中注册对应的url，url需要是完整的请求路径，**

http://localhost:8080/event/eventmanager/status.do.

若是应用系统收到了事件，则需要返回一个json串，告诉我们已经正确收到了事件，格式是：
{"success":"true"}。

获取参数和返回值参考如下代码：
```

	@RequestMapping("/status")
	public @ResponseBody Object getStatus(HttpServletRequest request,HttpServletResponse response){
		Map<String,Object> result = new HashMap<String,Object>();
		String sourceId = request.getHeader("sourceID");
		String eventyType = request.getHeader("eventType");
		String userObject = request.getHeader("userObject");
		JSONObject jsonObj = JSONObject.fromObject(userObject);

    //拿到参数后进行业务处理，处理完后按照以下格式返回success为true
		JSONObject json = new JSONObject();
		json.put("success", "true");
		return json;
	}

```

** 注：这个success为true意味着应用系统收到了消息，若是不返回这个json串，我们会一直发送，一直到应用系统成功接收，若是业务上有异常，应该应用系统自行处理。也就是说，只有你认为需要再次接收这个消息的情况下，才返回false。**


** 2，加签：若是要进行加签，在pub_event_node中注册节点填入client_key，作为我们加签的key值，若是不注册，我们认为不需要加签，直接调用应用系统注册的url。 **

** 3. 在web.xml中注册filter：**

```
      <!-- 事件通知发送服务签名验证 -->
      <filter>
          <filter-name>sendEventMsgSecurityFilter</filter-name>
          <filter-class>com.yonyou.iuap.event.manager.filter.SendEventMsgSecurityFilter</filter-class>
      </filter>
      <filter-mapping>
          <filter-name>sendEventMsgSecurityFilter</filter-name>
          <url-pattern>/eventmanager/sendEventMsg.do</url-pattern>
      </filter-mapping>

```

** 具体com.yonyou.iuap.event.manager.filter.SendEventMsgSecurityFilter实现，参考示例工程。 **

## API接口

### 发送事件服务（事件中心提供）

**请求方法**  

/eventmanager/sendEventMsg.do?appId=nodecode&ts=ts

** 注：使用公网服务时，地址为  https://uas.yyuap.com/event/eventmanager/sendEventMsg.do?appId=nodecode&ts=ts **

**请求方式**  

POST

**请求参数说明**  

appId：注册在事件中心的应用编码

ts：加签时使用，当前的时间戳

post请求参数：
<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
		<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>sourceID</td>
    <td>True</td>
		<td>int</td>
    <td>20</td>
    <td>事件源ID</td>
  </tr>
  <tr>
    <td>eventType</td>
    <td>True</td>
		<td>String</td>
    <td>40</td>
    <td>事件类型编码</td>
  </tr>
   <tr>
    <td>tenantCode</td>
    <td>True</td>
		<td>String</td>
    <td>8</td>
    <td>租户编码</td>
  </tr>
  <tr>
	 <td>userObject</td>
	 <td>False</td>
	 <td>String</td>
	 <td>无</td>
	 <td>事件的消息内容，建议为json格式</td>
  </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，0-失败</td>
  </tr>
  <tr>
    <td>msg</td>
    <td>返成功或错误信息，一般为成功提示或错误描述</td>
  </tr>
</table>

### 接收事件服务（注册监听的业务系统提供）

**请求方法**  

order/userAddlistener.do （由业务系统编写）

**请求方式**  

POST

**请求参数说明**  

post请求参数：
<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
		<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>sourceID</td>
    <td>True</td>
		<td>int</td>
    <td>20</td>
    <td>事件源ID</td>
  </tr>
  <tr>
    <td>eventType</td>
    <td>True</td>
		<td>String</td>
    <td>40</td>
    <td>事件类型编码</td>
  </tr>
   <tr>
    <td>tenantCode</td>
    <td>True</td>
		<td>String</td>
    <td>8</td>
    <td>租户编码</td>
  </tr>
  <tr>
	 <td>userObject</td>
	 <td>False</td>
	 <td>String</td>
	 <td>无</td>
	 <td>事件的消息内容，建议为json格式</td>
  </tr>
</table>

**返回参数说明**  

返回："data":{"success":"true"}。

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>success</td>
    <td>true/false，如果为false，会等待一段时间尝试再次发送</td>
  </tr>
</table>
