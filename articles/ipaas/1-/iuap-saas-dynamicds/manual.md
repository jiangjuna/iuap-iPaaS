#动态数据源组件概述#

## 业务需求 ##

多数据源组件iuap-saas-dynamicds是针对SAAS应用中多租户间需要对数据进行隔离的需求推出的。为了保证数据的隔离和安全性，同时考虑数据库异常时回滚的粒度，每个租户需要使用单独的数据库或者数据库实例，而一个租户配置单独一个数据源会导致创建太多的连接池，造成内存和连接的浪费，代价过大。所以需要平台提供统一的多租户应用的数据库连接方案，在保证连接可控的基础上达到按照不同租户连接不同数据源的要求。

同时，数据库的切换和连接池的控制等，要求做到对业务开发透明，应该只通过修改配置文件的方式达到上述目的。


## 解决方案 ##

iuap-saas-dynamicds组件依托iuap平台的多租户方案，根据传递的上下文信息，在基础的数据源之上进行一次路由，从数据源中获取数据库链接时修改租户对应的catalog，达到切换数据库的目的。

同时，动态数据源代理了多个可配置的基础数据源，可以根据全局配置文件，找到租户对应的具体数据源实例和catalog。

组件预留了对全局配置文件的扩展方案，基于Zookeeper做的管控，运行态的JVM中可以监听到全局配置的变化，从而根据生产环境租户的变化，动态的调整数据源和连接信息，达到真正动态增删数据源的效果。 

## 功能说明 ##

1. 实现了对多租户数据的数据库隔离

2. 支持通过只修改配置文件的方式，增加动态数据源功能

3. 支持使用租户上下文信息作为路由条件

4. 通过配置的方式，扩展特殊租户对多数据库的需求

5. 支持mysql和SqlServer


# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，其对外提供的依赖方式如下：

	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-saas-dynamicds</artifactId>
	  <version>${iuap.saas.modules.version}</version>
	</dependency>

${iuap.saas.modules.version} 为平台在maven私服上发布的组件的version。例如1.0.0-RELEASE。

## 使用限制 ##

组件初期的方案对租户和数据库的命名有如下：

1. 租户信息来源于用户中心
2. 租户和数据库一一对应并且租户编码和对应的数据库名相同
3. 上下文中保证传递租户信息

iuap-saas-dynamicds组件预留了扩展接口，在后期的升级扩展中，组件会支持根据配置中心获取租户与数据库之间的关系，同时会支持租户的不同子应用与不同的数据库之间的映射关系等。

# 使用说明 #

## 组件配置 ##

### 配置Maven依赖 ###

**在maven工程中添加对动态数据源组件的依赖**

	<dependency>
		<groupId>com.yonyou.iuap</groupId>
		<artifactId>iuap-saas-dynamicds</artifactId>
		<version>${iuap.saas.modules.version}</version>
	</dependency>

### 配置属性文件 ###

**修改application.properties文件中关于数据库连接的配置**

	jdbc.driver=com.mysql.jdbc.Driver
	jdbc.url=jdbc:mysql://localhost:3306/ec?useUnicode=true&characterEncoding=utf-8
	jdbc.catalog=ec
	jdbc.username=root
	jdbc.password=root
	jdbc.defaultAutoCommit=true
	
	#connection pool settings
	jdbc.pool.minIdle=0
	jdbc.pool.maxIdle=20
	jdbc.pool.maxActive=50
	jdbc.pool.maxWait=30000
	jdbc.pool.initialSize=0
	
	jdbc.pool.testOnBorrow=false
	jdbc.pool.validationInterval=30000
	jdbc.pool.testOnReturn=true
	jdbc.pool.validationQuery=select 1
	
	jdbc.pool.testWhileIdle=true
	jdbc.pool.timeBetweenEvictionRunsMillis=30000
	jdbc.pool.numTestsPerEvictionRun=-1
	
	jdbc.pool.minEvictableIdleTimeMillis=60000
	jdbc.pool.removeAbandoned=true
	jdbc.pool.removeAbandonedTimeout=60

### 配置Spring配置文件 ###

**在spring的持久化配置文件中，配置动态数据源bean和依赖的bean**

	<!-- 数据源配置, 使用Tomcat JDBC连接池 -->
	<bean id="jdbcDataSource" class="org.apache.tomcat.jdbc.pool.DataSource" destroy-method="close">
		<!-- Connection Info -->
		<property name="driverClassName" value="${jdbc.driver}" />
		<property name="url" value="${jdbc.url}" />
		<property name="username" value="${jdbc.username}" />
		<property name="password" value="${jdbc.password}" />
		<property name="defaultCatalog" value="${jdbc.catalog}" />
	
		<!-- Connection Pooling Info -->
		<property name="maxActive" value="${jdbc.pool.maxActive}" />
		<property name="maxIdle" value="${jdbc.pool.maxIdle}" />
		<property name="minIdle" value="${jdbc.pool.minIdle}" />
		<property name="maxWait" value="${jdbc.pool.maxWait}" />
		<property name="initialSize" value="${jdbc.pool.initialSize}" />
		
		<property name="testOnBorrow" value="${jdbc.pool.testOnBorrow}"/>
        <property name="validationInterval" value="${jdbc.pool.validationInterval}"/>
        <property name="testOnReturn" value="${jdbc.pool.testOnReturn}"/>
        <!--mysql,sqlserver使用，oracle使用select 1 from dual-->
        <property name="validationQuery" value="${jdbc.pool.validationQuery}"/>
        
        <property name="testWhileIdle" value="${jdbc.pool.testWhileIdle}"/>
        <property name="timeBetweenEvictionRunsMillis" value="${jdbc.pool.timeBetweenEvictionRunsMillis}"/>
        <property name="numTestsPerEvictionRun" value="${jdbc.pool.numTestsPerEvictionRun}"/>
        
		<property name="minEvictableIdleTimeMillis" value="${jdbc.pool.minEvictableIdleTimeMillis}" />
		<property name="removeAbandoned" value="${jdbc.pool.removeAbandoned}" />
		<property name="removeAbandonedTimeout" value="${jdbc.pool.removeAbandonedTimeout}" />
	</bean>
	
	<!--配置动态数据源-->
	<bean id="dynamicDataSource" class="com.yonyou.iuap.dynamicds.ds.DynamicDataSource">
		<property name="targetDataSources">
			<map key-type="java.lang.String">
				<entry key="jdbcDataSource" value-ref="jdbcDataSource" />
			</map>
		</property>
		<property name="defaultTargetDataSource" ref="jdbcDataSource" />
		<property name="dsProvider" ref="dsProvider" />
    </bean>
    
    <bean id="dsProvider" class="com.yonyou.iuap.dynamicds.ds.DefaultDataSourceProvider"/>


## 组件使用 ##

### 数据库准备 ###

在一个mysql数据库实例下，新建多个库，库名可以为tenant01，tenant02等。

### 上下文信息设置 ###

正常情况下，上下文信息又应用的过滤器或者拦截器依据用户登录时候的信息设置到线程绑定变量上，动态数据源组件直接使用即可。

如果是单元测试，可以使用api进行模拟设置：

	InvocationInfoProxy.setTenantid("tenant01");

### 数据库操作 ###

在sping的持久化配置文件中，配置对应的持久化方式，代码中注入服务调用即可。注意，原来配置DataSource的地方，需要配置成dynamicDataSource。

 	<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
		<property name="dataSource" ref="dynamicDataSource"></property>
    </bean>	

单元测试示例：

	@Test
	public void testJdbcTemplateWithDynamicDatasource() throws Exception {
		
		InvocationInfoProxy.setTenantid("teanant01");
		String sql = "select count(*) from mgr_user";
		int num = jdbcTemplate.queryForObject(sql, Integer.class);
		System.out.println(num);
		
		InvocationInfoProxy.setTenantid("tenant02");
		sql = "select count(*) from mgr_user";
		num = jdbcTemplate.queryForObject(sql, Integer.class);
		System.out.println(num);
		
		num = jdbcTemplate.queryForObject(sql, Integer.class);
		System.out.println(num);
		
	}