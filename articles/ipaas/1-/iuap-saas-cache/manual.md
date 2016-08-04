#分布式缓存SaaS版组件概述#

## 业务需求 ##

基础的分布式缓存组件iuap-cache提供了缓存的操作API封装、连接池的建立等基础功能。针对多租户的SaaS应用，各个租户间的缓存数据需要适当的隔离，以保证多个租户对相同的业务功能可以缓存不同的业务数据。需要平台统一提供多租户缓存数据的隔离方案，减轻业务开发使用缓存的复杂性。


## 解决方案 ##

iuap 平台从缓存key的粒度控制多租户缓存数据的区分。

iuap-saas-cache组件依托于iuap-cache组件，在原有的基础上，增加了对租户信息的处理。组件要求调用方的参数中，传递租户的编码，以租户信息和缓存基础key合成的标识作为隔离缓存数据的依据。



## 功能说明 ##

iuap-saas-cache组件扩展如下功能：

1.	key级别隔离多租户缓存数据；
2.	兼容iuap-cache的基础操作；


# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，依赖Jedis的2.6.0版本和iuap-cache，其对外提供的依赖方式如下：

	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-saas-cache</artifactId>
	  <version>${iuap.saas.modules.version}</version>
	</dependency>

${iuap.saas.modules.version} 为平台在maven私服上发布的组件的version，如1.0.0-RELEASE。

## 部署结构 ##

Redis本身支持多种语言的客户端来连接，iuap-cache组件利用java语言通过Jedis客户端进行连接，建立连接池并支持哨兵方式的url。Redis中间件通常是配合哨兵进行集群部署。

iuap-saas-cache组件依托于iuap-cache，部署结构上保持相同。

# 使用说明 #

## 组件配置 ##

**1:在属性文件中，配置redis的连接url，根据业务不同的需要，可以配置成单机方式、集群方式、分片方式等**

	单机方式示例：
	redis.url=direct://localhost:6379?poolName=mypool&poolSize=100
	
	集群示例：
	redis.url=sentinel://20.12.6.57:26379,20.12.6.58:26379,20.12.6.59:26379?poolName=mypool&masterName=mymaster&poolSize=100
	
	分片示例：
	redis.shardedurl=direct://20.12.6.57:6379,20.12.6.58:6379,20.12.6.59:6379?poolName=mypool&masterName=mymaster&poolSize=100

**2:Spring的配置文件中，声明iuap-saas-cache中的bean**

	<bean id="redisPool" class="com.yonyou.iuap.cache.redis.RedisPoolFactory" scope="prototype" factory-method="createJedisPool">
		<constructor-arg value="${redis.url}" />
	</bean>
	
	<bean id="jedisTemplate" class="org.springside.modules.nosql.redis.JedisTemplate">
		<constructor-arg ref="redisPool"></constructor-arg>
	</bean> 
	
    <bean id="cacheManager" class="com.yonyou.iuap.cache.CacheManager">
        <property name="jedisTemplate" ref="jedisTemplate"/>
    </bean>	

    <bean id="saasCacheManager" class="com.yonyou.iuap.cache.SaasCacheManager">
        <property name="cacheManager" ref="cacheManager"/>
    </bean>	

**3:工程中引入对iuap-saas-cache组件的依赖**

	<dependency>
		<groupId>com.yonyou.iuap</groupId>
		<artifactId>iuap-saas-cache</artifactId>
		<version>${iuap.saas.modules.version}</version>
	</dependency>	

${iuap.saas.modules.version}为在pom.xml中定义的引用iuap-saas-cache的版本。

**4:代码中调用组件提供的API，操作分布式缓存**

	/**
	 * 普通存取删除测试,get,set,remove,exists
	 * 
	 * @throws Exception
	 */
	@Test
	public void testSimpleCache() throws Exception {
		String key = "testKey";
		final String tenantId = "tenant01";
		saasCacheMgr.set(tenantId, key, "test");
		
		Assert.isTrue(saasCacheMgr.exists(tenantId, key));
		
		String cacheValue = (String)saasCacheMgr.get(tenantId, key);
		System.out.println(cacheValue);
		
		saasCacheMgr.removeCache(tenantId, key);
		
		Assert.isNull(saasCacheMgr.get(tenantId, key));
	}

**5:更多API操作和配置方式，请参考缓存对应的示例工程(DevTool/examples/example\_iuap\_cache)**

iuap-saas-cache中的API使用方式和基础组件的使用相似，在原有参数基础上多增加了租户信息的参数，开发人员从上下文中获取信息传入即可。如果使用的是iuap平台提供的上下文方案，获取方式如下：

	String tenantId = InvocationInfoProxy.getTenantid();


## 常用接口 ##

- SaasCacheManager

<table style="border-collapse:collapse">
	<thead>
		<tr>
			<th>方法名</th>
			<th>参数</th>
			<th>返回值</th>
			<th>说明</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>set(final String tenantId, final String key, final T value)</td>
			<td>String tenantId(租户标识), String key（缓存key）, T value（缓存对象，允许序列化）</td>
			<td>void</td>
			<td>放置键值对对应的缓存</td>
		</tr>
		<tr>
			<td>setex(final String tenantId, final String key, final T value，final int timeout)</td>
			<td>String tenantId(租户标识), String key（缓存key）, T value（缓存对象，允许序列化），int timeout（缓存的有效期）</td>
			<td>void</td>
			<td>放置键值对对应的缓存并设置缓存有效期，单位为秒</td>
		</tr>
		<tr>
			<td>expire(final String tenantId, final String key, final int timeout)</td>
			<td>String tenantId(租户标识), final String key（缓存key）, final int timeout（缓存的有效期）</td>
			<td>void</td>
			<td>修改缓存信息的有效期</td>
		</tr>
		<tr>
			<td>exists(final String tenantId, final String key)</td>
			<td>String tenantId(租户标识), final String key（缓存key）</td>
			<td>Boolean（是否存在的标志）</td>
			<td>判断键值为key的缓存是否存在</td>
		</tr>
		<tr>
			<td>get(final String tenantId, final String key)</td>
			<td>String tenantId(租户标识), final String key（缓存key）</td>
			<td>T extends Serializable（声明的返回类型对象）</td>
			<td>获取对应键值的缓存对象</td>
		</tr>
		<tr>
			<td>hget(final String tenantId, final String key, final String fieldName)</td>
			<td>String tenantId(租户标识), final String key（缓存Map的key），final String fieldName（Map下某个属性的key）</td>
			<td>T extends Serializable（声明的返回类型对象）</td>
			<td>获取HashMap中对应的子key的缓存值</td>
		</tr>
		<tr>
			<td>hmget(final String tenantId, final String key, final String... fieldName)</td>
			<td>String tenantId(租户标识), final String key（缓存Map的key）, final String... fieldName（Map下若干属性的key）</td>
			<td>List</td>
			<td>获取HashMap中对应的多个子key的缓存值集合</td>
		</tr>
		<tr>
			<td>hexists(final String tenantId, final String key, final String field)</td>
			<td>String tenantId(租户标识), final String key（缓存Map的key），final String field（Map下某个属性的key）</td>
			<td>Boolean（是否存在的标志）</td>
			<td>判断HashMap中对应的key是否存在</td>
		</tr>
		<tr>
			<td>hset(final String tenantId, final String key, final String fieldName, final T value)</td>
			<td>String tenantId(租户标识), final String key（缓存Map的key），final String fieldName（Map下某个属性的key），final T value（要放置的缓存对象）</td>
			<td>void</td>
			<td>放置HashMap中的属性和值</td>
		</tr>
		<tr>
			<td>removeCache(final String tenantId, String key)</td>
			<td>String tenantId(租户标识), String key（缓存的key）</td>
			<td>void</td>
			<td>删除key对应的缓存信息</td>
		</tr>
		<tr>
			<td>hdel(final String tenantId, String key, String field)</td>
			<td>String tenantId(租户标识), String key（缓存Map的key），String field（Map下某个属性的key）</td>
			<td>void</td>
			<td>删除Map下键值为filed的属性</td>
		</tr>
	</tbody>
</table>
