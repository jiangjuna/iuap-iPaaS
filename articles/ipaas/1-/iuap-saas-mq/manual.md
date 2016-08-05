#消息队列组件SaaS版概述#

## 业务需求 ##
业务系统中为了提高核心服务的并发性，需要对不是强关联的服务之间进行解耦，使用消息实现彼此异步化，保障核心能力。在应对突发高流量的场景时，也需要有异步消息的机制来实现对请求的削峰填谷，保障核心业务不受影响。

当在不同的系统之间发送消息时，有些场景需要将用户和租户的信息传递到消息的消费者端，需要消息组件在传递基础消息信息的同时，将运行上下文中的一些类似用户登录、所属租户等信息传递下去。

## 解决方案 ##
iuap-saas-mq基于iuap-mq组件和多租户上下文信息传递方案，在提供消息队列功能基础之上，同时实现了对上下文信息自动在生产者和消费者之间传递的功能，开发者只需要关注业务逻辑，实现异步业务。

消费者端接收消息的线程在接收消息的同时，可以从上下文中取出相关的信息，依赖租户信息完成后续的业务动作。

## 功能说明 ##

1.	完全兼容iuap-mq的功能特性；
2.	支持自动加入上下文信息在队列生产者和消费者之间传递

# 整体设计 #

## 依赖环境 ##
iuap-saas-mq组件依赖iuap-mq和spring-rabbit、aliyun-sdk-mns，对外的提供方式如下：

    <dependency>
		<groupId>com.yonyou.iuap</groupId>
		<artifactId>iuap-saas-mq</artifactId>
		<version>${iuap.saas.modules.version}</version>
	</dependency>

${iuap.saas.modules.version}为在pom.xml定义的需要引用组件的版本。


## 处理机制 ##
iuap-saas-mq组件在 iuap-mq的基础上封装了上下文信息，在发送消息之前从系统中获取上下文信息，并和业务信息放到一起组装成json格式的数据发送出去。

接收方先有组件实现的监听器去解析传入进来的json信息，再将信息传递下去。用户只需要继承组件的监听类，并覆写相应的方法，在里面书写系统业务逻辑即可。

**注意:**

组件依赖多租户上下文传递的方案，在消息发送前，需要保证上下文信息中包含租户信息。

# 使用说明 #

## 组件配置 ##

### RabbitMQ方式 ###

spring配置文件内容

-  发送配置

	    <bean id="rabbitMqContextService" class="com.yonyou.iuap.mq.rabbit.RabbitMqContextService">
			<property name="rabbitTemplate" ref="rabbitTemplate"></property>
		</bean>
	
	
- 接收方配置


	    <!-- queue litener  观察 监听模式 当有消息到达时会通知监听在对应的队列上的监听对象-->
		<rabbit:listener-container connection-factory="connectionFactory" acknowledge="auto">
			<rabbit:listener queues="simple_queue" ref="queueLitener"/>
		</rabbit:listener-container>
	    
	    
		<bean id="queueLitener" class="com.yonyou.iuap.mq.rabbitmq.MqSaasTestListener"></bean>   


### 阿里云MNS消息配置 ###

- 发送配置


    	<bean id="mnsAccount" class="com.aliyun.mns.client.CloudAccount">
    		<constructor-arg index="0">           
    			<value>${mns.accesskeyid}</value>       
    		</constructor-arg>       
    		<constructor-arg index="1">           
    			<value>${mns.accesskeysecret}</value>       
    		</constructor-arg>
    		<constructor-arg index="2">           
    			<value>${mns.accountendpoint}</value>       
    		</constructor-arg>		
    	</bean>
    	
    	<bean id="mqService" class="com.yonyou.iuap.mq.mns.AliyunMnsSaasService">
    		<property name="mnsAccount" ref="mnsAccount"></property>
    	</bean>
    	
   ${mns.accesskeyid}  ${mns.accesskeysecret}  ${mns.accountendpoint}   为申请的阿里云的账户信息。
   
   
- 接收配置
		
		<bean id="simpleListener" class="com.yonyou.iuap.mq.mns.MnsSimpleListener">
			<constructor-arg index="0">           
				<value>testali-mq-01</value>       
			</constructor-arg>       
			<constructor-arg index="1">           
				<ref bean="mnsAccount"/>    
			</constructor-arg>
			<constructor-arg index="2">           
				<value>5</value>       
			</constructor-arg>
		</bean>

testali-mq-01为在阿里云上创建的对列名称。

## 调用方式 ##

### RabbitMQ消息中带上下信息 ###

发送消息示例：
     
    @Resource(name="rabbitMqContextService")
	IMqService mqService ;

	/** 发送普通rabbitmq  上下文消息传递*/
	@Test
	public void queueMessageContextTest() throws InterruptedException {
		TestMq t = new TestMq("testmq" + i, "test" + i ) ;
		JsonMapper objectMapper = new JsonMapper();
		String msg = objectMapper.toJson(t) ;
		mqService.sendMsg("iuap-direct-exchange", "simple_queue_key" , msg);
	}
	
	在消息中除了msg这段消息外，iuap组件会自动加入发送消息方所在系统的上下文信息。
		
消息接收方代码实现：

接收方要继承类 com.yonyou.iuap.mq.rabbit.consumer.MqSaasListener ，并覆写 handleMessage() 方法。

在handleMessage方法里面写系统接收到消息后的业务逻辑。
而接收到的上下文信息都放到了 InvocationInfoProxy类里面，可以用 get方法获取。
	
	public class MqSaasTestListener extends MqSaasListener{
		private static Logger logger = LoggerFactory.getLogger(MqSaasTestListener.class);
	
		//接收消息处理具体业务逻辑
		@Override   
		public void handleMessage(String message){
			System.out.println(InvocationInfoProxy.getTenantid());
			System.out.println(InvocationInfoProxy.getUserid()); 
		} 

	}
		
		

### 阿里云MNS传递上下文信息 ###


- 	发送方代码
	
		@Resource(name="mqService") 
		IMqService mqService ;
		
		//发送普通queue 到ali云 
		@Test
		public void queueMessageContext() throws InterruptedException {
			 
			TestMq t = new TestMq("testmq", "test") ;
			JsonMapper objectMapper = new JsonMapper();
			String msg = objectMapper.toJson(t) ;	 
			mqService.sendMsg("testali-mq-01", null, msg);
		}
	


-   接收

	接收方需要实现 iuap组件监听器，并覆写 handleMessage(), 在此方法里面实现系统业务逻辑。
	
	
		public class MnsSimpleListener extends MnsSaasListener {	

			public MnsSimpleListener(String queueName, CloudAccount mnsAccount, int waitSeconds) {
				super(queueName, mnsAccount, waitSeconds);
			} 

			public void handleMessage(String message){
				//根据业务需要处理
				System.out.println(message);
				System.out.println(InvocationInfoProxy.getTenantid());
			}
		}
