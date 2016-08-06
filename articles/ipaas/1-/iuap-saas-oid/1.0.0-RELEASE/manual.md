#对象OID组件概述#

## 业务需求 ##

在数据持久化时，业务上通常需要生成全局唯一的对象标识ID，在应用水平扩展时，多个JVM中生成的唯一标识保证不冲突。传统的UUID能保证唯一性，但在数据量很大之后，数据查询效率会下降，所以需要一种能保证全局唯一并且可以保证读写效率的对象标识生成方案。

在多租户的SaaS应用中，在数据的唯一标识上，希望在保证全局唯一的条件下，通过PK直观的区分开不同的租户数据，对ID的统一生成提出了需求。

## 解决方案 ##

iuap-saas-oid组件提供20位全局唯一标识生成方案，基于UAPOID算法通过schema、顺序号（包括0-9和a-z）等生成唯一ID，同时利用本地缓存和数据库机制实现高效和高可用。

## 功能说明 ##

1. 提供全局唯一的20位ID生成API；
2. 支持扩展8位标识与租户的映射；
3. 支持本地缓存；

# 整体设计 #

20位ID生成方案为8位租户标识+12位流水号，流水号的生成依赖数据库和动态数据源组件。多个JAVA进程同时生成OID时，需要首先从数据库中取回一定的取值范围为本JVM使用，在获取范围时候锁表。取值范围可以在配置文件中配置，

# 使用说明 #

## 组件配置 ##

### Maven依赖配置 ###

	<dependency>
		<groupId>com.yonyou.iuap</groupId>
		<artifactId>iuap-saas-oid</artifactId>
		<version>${iuap.saas.modules.version}</version>
	</dependency>

${iuap.saas.modules.version} 为在pom.xml中定义的需要引入组件的version。

### Spring配置 ###

    <bean id="oidJdbcService" class="com.yonyou.iuap.oid.OidJdbcService">
    	<property name="jdbcTemplate" ref="jdbcTemplate" />
    	<property name="stepSize" value="5000" />
    </bean>
    
    <bean id="oidSmService" class="com.yonyou.iuap.oid.OidSchemaMappingService">
    	<property name="schemasProvider" ref="schemasProvider" />
    </bean>

stepSize为一个java进程一次性取出流水范围的值。

###数据库准备###

	CREATE TABLE pub_oid(
		schemacode VARCHAR(8) NOT NULL,
		oidbase VARCHAR(20) NOT NULL,
		id VARCHAR(36) NOT NULL,
		ts TIMESTAMP NULL,PRIMARY KEY (id)
	)ENGINE=InnoDB DEFAULT CHARSET=utf8;


## 方法调用 ##

###生成唯一的OID###
	@Test
	public void testWithException() {
		InvocationInfoProxy.setTenantid("tenant01");
		String id = OidGenerator.getInstance().genOid(InvocationInfoProxy.getTenantid());
		System.out.println(id);
		
	}

###批量生成###

	OidGenerator.getInstance().genBatchOids(InvocationInfoProxy.getTenantid(), 100);

