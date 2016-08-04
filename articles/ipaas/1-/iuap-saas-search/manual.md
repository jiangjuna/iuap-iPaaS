#搜索组件SaaS版概述#

## 业务需求 ##
在多租户的需求下，为了保证数据的安全性，租户的数据需要物理隔离。在充分利用硬件的前提下，解决solr数据隔离是一个迫切的需求。

## 解决方案 ##
iuap使用共享solrcloud创建多个collection的方案来解决数据隔离的问题，为租户提供了安全性较高的数据物理隔离方案，同时能够更加充分的利用硬件资源，使得每个solrcloud可以支持更多的租户。


## 功能说明 ##
1. 兼容iuap-search组件的功能；
2. 支持多租户的数据隔离场景；


# 使用说明 #
## maven依赖 ##
	<dependency>
        <artifactId>iuap-saas-search</artifactId>
        <groupId>com.yonyou.iuap</groupId>
        <version>${iuap.saas.modules.version}</version>
    </dependency>

**注意**:${iuap.saas.modules.version} 为平台在maven私服上发布的组件的version。

## 使用方式 ##
1. 在solr example下collection1中的schema.xml添加以下内容

		<field name="tenentid" type="string" indexed="true" stored="false" multiValued="false"/>

2. 新建一个maven工程，导入maven依赖，实现如下代码


		public class Wood extends BaseSolrEntity implements Serializable {
		
		    @Field
		    private String id;
		
		    @Field
		    private String name;
		
		    @Field
		    private List<String> title;
		
		    @Field
		    private double count;
		
		    @Field
		    private Double money;
		
		    @Field("createtime")
		    private Date createTime;
		    
		    //此处省略getter,setter
		}

示例代码如下：

	public class SearchExample{
	    
	    @Test
	    public void testSaasSearch() throws SearchException {
	        System.setProperty(ISaasClient.SOLR_ZK_HOSTS, "192.168.56.101:2181");
	
	        SearchClient cloudSearchClient = new SearchClient();
	        SaasSearchClient saasSearchClient = new SaasSearchClient();
	        saasSearchClient.setSearchClient(cloudSearchClient);
	        CloudSearchClient collZxs = saasSearchClient.createCloudSearchClient("tenantid");

			//查询指定租户对应的collection下的数据
	        Criteria<Wood> criteria = collZxs.createCriteria(Wood.class);
	        System.out.println(criteria.list());
	
	        CloudSearchClient collUap = saasSearchClient.createCloudSearchClient("iuap");
	        criteria = collUap.createCriteria(Wood.class);
	        System.out.println(criteria.list());
	
	    }
	}


