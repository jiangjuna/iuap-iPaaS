#分布式锁组件SaaS版概述#

## 业务需求 ##

在基础技术组件iuap-lock组件的基础上，业务上已经可以对某些特殊资源全局加锁，防止同时对同一资源的访问造成的数据不一致性。

在多租户的SaaS应用中，相同类型的资源需要以租户标识来区分，加锁时针对的资源唯一标识也需要携带租户信息，需要平台提供一种统一的方式解决加锁问题。

## 解决方案 ##

iuap-saas-lock在基础的分布式锁组件的基础上，对锁资源追加租户标识，针租户信息和资源标识进行加锁和解锁，提供统一的API，屏蔽了业务开发对于租户信息的考虑。

## 功能说明 ##

在具备分布式锁组件iuap-lock所有功能的基础上，扩展了如下功能：

1. 支持多租户应用的资源全局加锁
2. 预留租户和唯一标识之间的映射扩展

# 使用说明 #

## 组件配置 ##

###Maven配置###

	<dependency>
		<groupId>com.yonyou.iuap</groupId>
		<artifactId>iuap-saas-lock</artifactId>
		<version>${iuap.saas.modules.version}</version>
	</dependency>
${iuap.saas.modules.version} 为平台在maven私服上发布的组件的version。

###属性文件配置###

	#配置锁组件连接zookeeper的url
	zklock.url=127.0.0.1:2181
	#zookeeper集群方式配置示例
	#zklock.url=20.12.6.58:2181,20.12.6.59:2181,20.12.6.60:2181

	#锁存活最大时间，单位秒，如果不配置，不强制删除，如果配置，加锁失败且已有的锁存活时间大于此值，强制删除
	zklock.maxlocktime=3600

###Spring文件配置###

配置连接池的信息，具体配置如下：

	<bean id="zkPoolConfig" class="org.apache.commons.pool2.impl.GenericObjectPoolConfig">
		<property name="maxTotal" value="100"/>
		<property name="maxIdle" value="20"/>
		<property name="maxWaitMillis" value="60000"/>
		
		<property name="timeBetweenEvictionRunsMillis" value="60000"/>
		<property name="numTestsPerEvictionRun" value="-1"/>
		<property name="minEvictableIdleTimeMillis" value="30000"/>
	</bean>

配置租户映射服务类

	<!--租户和schemacode之间的映射服务类-->
    <bean id="lockSmService" class="com.yonyou.iuap.lock.LockSmService"/>

其他配置和基础的iuap-lock的配置方式相同，请参考基础技术组件的iuap-lock对应的技术手册

## 使用方法 ##

###普通加锁解锁###

	@Test
	public void testDisLock() throws InterruptedException {
		String tenantId = "tenant01";
		final String lockpath = "iuapdislockpath_test";
		boolean islocked = false;
		
		try {
			islocked = SaasDisLock.lock(tenantId, lockpath);
			if(islocked){
				System.out.println("加锁成功，耗用时间为:" + time3 + " 毫秒!");
			}
		} catch (Exception e) {
			System.out.println("锁服务调用失败，耗用时间为:" + time3 + " 毫秒!");
		} finally {
			if(islocked){
				SaasDisLock.unlock(tenantId, lockpath);
			} else {
				System.err.println("加锁失败!");
			}
		}
	}


###批量加锁解锁###

	@Test
	public void testBatchDisLockCodeExample() throws InterruptedException {
		String tenantId = "tenant01";
		String lockpaths[] = {"iuapdislockpath_test1","iuapdislockpath_test2","iuapdislockpath_test3"};
		boolean islocked = false;
		try {
			islocked = SaasDisLock.lock(tenantId, lockpaths);
			if(islocked){
				System.out.println("批量加锁成功，业务处理!");
			} else {
				System.out.println("批量加锁失败!");
			}
		} catch (Exception e) {
			System.out.println("加锁操作异常!");
		} finally {
			if(islocked){
				SaasDisLock.unlock(tenantId, lockpaths);
				System.out.println("批量解锁成功!");
			} 
		}
	}