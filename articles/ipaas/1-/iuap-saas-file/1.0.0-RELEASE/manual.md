#文件组件SaaS版概述#

## 业务需求 ##
iuap-file组件在面对多租户类型的SaaS应用需求时，会遇到很多限制，如租户文件数据的隔离。需要平台提供在多租户SaaS应用场景下对文件资源操作和管理的方案，提供SaaS版的文件组件，简单的实现文件的多租户管理。

## 解决方案 ##

iuap-saas-file在原有的文件组件基础上，增加了针对多租户的支持。以配置的方式实现租户和OSS Bucket路径的映射，实现了租户数据逻辑上的隔离，同时支持文件的直传、权限控制等。

## 功能说明 ##
1.	支持文件的上传、下载、删除操作；
2.	支持获取文件下载URL；
3.	支持阿里云OSS的文件直传和回调；
4.	支持阿里云OSS不同bucket权限的文件URL获取；
5.	支持对阿里云OSS请求的安全校验；
6.	支持多租户对OSS bucket目录的映射

# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，依赖FastDFS和阿里云OSS的客户端SDK,其对外提供的依赖方式如下：

	<dependency>
		<groupId>com.yonyou.iuap</groupId>
		<artifactId>iuap-saas-file</artifactId>
		<version>${iuap.saas.modules.version}</version>
	</dependency>

${iuap.saas.modules.version} 为平台在maven私服上发布的组件的version。


## 调用流程 ##
<img src="/images/file_structure.jpg"/>

组件封装了三种文件系统，并暴露出了常用的文件操作接口，用户通过这些接口，去操作组件所管理的对应文件系统。


# 使用说明 #

## 组件配置 ##

与基本的文件组件相同,SaaS版文件组件需要修改属性配置文件，指定存储类型。组件通过SaasFileManager类下的静态方法，完成对多租户情境下文件资源管理的支持。

### 属性文件配置 ###

**在application.properties文件中配置工程所使用的文件存储系统**

	#使用哪种文件存储系统（如AliOss阿里云）
    storeType=AliOss
    #storeType=Local
    #storeType=FastDfs

	#默认存储Bucket(private权限)
    defaultBucket=your private bucket
	#默认存储Bucket(read权限)
    defaultBucketRead=your read bucket
	#默认存储Bucket(Full权限)
	defaultBucketFull=your full bucket
    
    #使用阿里云文件系统，配置阿里云endpoint、accessKeyId、accessKeySecret
    endpoint=oss-cn-beijing.aliyuncs.com
    accessKeyId=your accessKeyId
    accessKeySecret=your accessKeySecret
    #接受阿里云上传回调的主机ip和端口，该主机需要外网能直接访问（直传功能）
    callbackTarget=http://localhost
    #处理上传回调的servlet截取的地址，直传成功后阿里云oss将会对上面配置的主机地址发送url为改配置url的post请求（直传功能）
    callbackUrl=/oss
    #需要上传回调所带的参数，上面配置的的请求将会携带callbackBody配置的参数（直传功能）
    callbackBody=filename=${object}&bucket=${bucket}&size=${size}

在工程上配置属性文件时，可以根据使用的存储类型，配置需要的部分。


## 使用方式 ##

###阿里云OSS###

**普通方式**

组件提供的API，操作阿里云oss，所有的api都通过SaasFileManager类静态调用，与原始组件相比各个方法的参数多需要一个String类型的租户ID，例子：SaasFileManager.uploadFile("yourtenantID","yourbucket","test.txt",content)。
    
	//上传，参数（bucket名，文件名，上传文件字节数组），返回（上传文件的文件名）
    String uploadFile(String bucketName,String tenantID, String fileName,byte[] fileContent)

 	//下载，参数（bucket名，文件名），返回（下载文件的字节数组）
 	byte[] downLoadFile(String bucketName,String tenantID,String fileName) ;

	//删除，参数（bucket名，文件名），返回（删除是否成功标志）
    boolean deleteFile(String bucketName,String tenantID,String fileName)

    //获取文件url，参数（bucket名，文件名，过期时间），返回（上传文件的url）目前expired参数只支持阿里云oss私有bucket，其他系统可置为0
	String getUrl(String bucketName,String tenantID, String fileName,int expired)
	
	//获取图片url，参数（bucket名，文件名，过期时间），返回（图片的url）与getUrl的区别是文件名可以使用类似example.jpg@100h_100w这种格式产生略缩图，）目前expired参数只支持阿里云oss私有bucket，其他系统可置为0
	String getImgUrl(String bucketName,String tenantID,String fileName,int expired)

example测试类


	/**
	 * 阿里云上传下载删除测试
	 * 测试前请设置application.properties的storeType=AliOss
	 * 参数"your bucket"为阿里云oss的bucket名，请更换为您的bucket名
	 * @throws Exception
	 */
	@Test
	public void testAliUpload() throws Exception {
		String filename;
		//借用文件流生成文件的二进制数组
		LocalClient client =LocalClient.getInstance();
		//参数为测试文件路径;		
		byte[] content =client.download("/etc/filetest/test.txt");
		//==================准备工作完毕=============================
		
		//阿里云上传
		filename=SaasFileManager.uploadFile("your bucket","your tenantID","test.txt",content);
		//阿里云下载
		byte[] downloadContent=SaasFileManager.downLoadFile("your bucket","your tenantID",filename);
		//获取文件url
		String url = SaasFileManager.getUrl("your bucket","your tenantID", filename, 60);
		//获取图片url(将上传文件路径改为图片)
		//String imgurl = SaasFileManager.getImgUrl("your bucket","your tenantID", filename, 60);
		//删除
		boolean flag=SaasFileManager.deleteFile("your bucket","your tenantID",filename);
		
		System.out.println(filename);
		System.out.println(url);
		//System.out.println(imgurl);
		System.out.println("删除状态"+flag);
		//Assert.isTrue(flag);
	}	



**文件直传**

直传实现与普通文件组件类似，多加入了租户id参数，用来代表不同的租户。更多内容可以参见示例工程example_iuap_saas_file


直传功能目前只支持阿里云oss，该功能使文件上传请求和流量不再经过应用服务器而直接交给oss处理，直传功能由阿里云官方文档推荐的框架搭建(相关连接：<a href="https://help.aliyun.com/document_detail/31927.html?spm=5176.doc31923.2.3.VjMHHN">web端直传</a>)

流程

用户请求->Servlet获取签名->js装配获得的签名并上传文件->oss上传文件后向Servlet发送回调->Servlet对oss回调请求处理后向oss发送处理结果->oss向用户返回上传结果。

首先建立一个Servlet类继承SaasCallbackServer，通过该Servlet装配oss直传需要的签名数据，并处理oss只直传完毕后的回调
	    
	@WebServlet(asyncSupported = true,name = "Oss",urlPatterns = { "/oss" })
	public class OssDirectServlet extends SaasCallbackServer {
		
		/**
		 * 
		 */
		private static final long serialVersionUID = 1L;
	
		protected void doPost(HttpServletRequest request, HttpServletResponse response)
				throws ServletException, IOException {
			String ossCallbackBody = GetPostBody(request.getInputStream(), Integer.parseInt(request.getHeader("content-length")));//获取回调
			boolean ret = VerifyOSSCallbackRequest(request, ossCallbackBody);//验证回调
			
			if (ret)
			{
				//验证通过进行操作
				Map<String, Object> map = getUrlParams(ossCallbackBody);
				String filename=(String) map.get("filename");//获取存储的文件名
				String bucket=(String) map.get("bucket");//获取存储的bucket
				System.out.println(filename);
				System.out.println(bucket);
				
				response(request, response, "{\"Status\":\"OK\"}", HttpServletResponse.SC_OK);
	
			}
			else
			{
				response(request, response, "{\"Status\":\"verdify not ok\"}", HttpServletResponse.SC_BAD_REQUEST);
			}
		}
	
	}


通过js进行上传
	    function init(){
	
		//提交
		//获取oss签名参数
	    var ret = get_signature()
		//装配发送到oss文件上传url   
	    var formData = new FormData(); 
	    var file=$("#file").prop('files');
	   
	    var filepath=$("#file").val();
	    var arr=filepath.split('\\');
	    var filename=arr[arr.length-1];//===================获得本地文件名（需要你提供）
	
	    if (ret == true)
	    {
	    	//装配oss签名参数
	    	formData.append("name",filename);
	    	formData.append("key", key);
	    	formData.append("policy", policyBase64);
	    	formData.append("OSSAccessKeyId", accessid);
	    	formData.append("success_action_status", '200');
	    	formData.append("callback", callbackbody);
	    	formData.append("signature", signature);
	    	formData.append("file",document.getElementById('file').files[0])//===================获得文件数据（需要你提供）
	    }
		//发送文件上传请求   
		$.ajax({  
	        url: host,  
	        type: 'POST',  
	        data: formData,  
	        async: false,  
	        contentType: false, //必须
	    	processData: false, //必须
	        success: function (returndata) {  
	            alert(returndata);  
	        },  
	        error: function (returndata) {  
	            alert(returndata);  
	        }  
	    });  
	}
	
	//发送到应用服务器 获取oss签名参数
	function send_request(){
		var obj
		$.ajax({
			type : 'GET',
			async : false,
			url :'http://localhost/example_iuap_saas_file/oss?tenantid='+document.getElementById('tenantid').value,//==========租户id（需要你提供）
			success : function(data){
			obj=$.parseJSON(data);
			} 
		});
		return obj;
	}
	
	//获取到签名参数后将参数填入变量待用
	function get_signature()
	{
	    //可以判断当前expire是否超过了当前时间,如果超过了当前时间,就重新取一下.3s 做为缓冲
	    expire = 0
	    now = timestamp = Date.parse(new Date()) / 1000; 
	    console.log('get_signature ...');
	    console.log('expire:' + expire.toString());
	    console.log('now:', + now.toString())
	    if (expire < now + 3)
	    {
	        console.log('get new sign')
	        var obj = send_request(obj)
	        policyBase64 = obj['policy']
	        accessid = obj['accessid']
	        signature = obj['signature']
	        expire = parseInt(obj['expire'])
	        callbackbody = obj['callback'] 
	        host =obj['host']
	        key = obj['perfix']+'${filename}'
	        return true;
	    }
	    return false;
	};
	
	$("#submit").attr("onclick","init();");

发送上传请求后会在上文中的Servlet类dopost方法处理oss回调，最终oss将结果发送回前台js的ajax回调。

**临时文件url与永久文件url**


oss文件系统能够通过getUrl方法返回文件的url，返回的url有临时和永久两种。该接口如下


public static String getUrl(String bucketName，String tenantID,String fileName,int expired) 

四个参数为ossbucket的权限，租户id，文件名，过期时间。其中bucketName参数对应了配置文件application.properties中所配置的defaultBucket和defaultBucketRead的bucket名称，如果该bucket名称没有配置则默认该bucket的权限为private。如果需要临时url的话bucketName对应的bucket权限应该为private，并且传入expired值为过期时长。如果需要永久的文件url前提是上传文件bucket需要为read或full权限（不推荐使用），设置bucket权限可以在oss控制台(相关连接：<a href="https://help.aliyun.com/document_detail/31885.html?spm=5176.doc31886.6.101.vSWwVY">建立新的bucket</a>)，选择需要修改权限的bucket，Bucket属性标签有读写权限修改。在geturl获取永久链接时expired可以为0。

**略缩图功能**

首先在oss控制台，选择存储略缩图文件的bucket，在左边的菜单中选择图片处理，选择开通图片处理。
然后通过下面的接口可以返回略缩图url：

public static String getImgUrl(String bucketName,String tenantID,String fileName,int expired)

略缩图也是分为临时与永久url，方法与上文相同。通过该方法获得url以后附加类似@100h的参数产生略缩图。

例子：

将图缩略成高度为100，宽度按比例处理。

http://image-demo.img-cn-hangzhou.aliyuncs.com/example.jpg@100h

将图缩略成宽度为100，高度按比例处理。

http://image-demo.img-cn-hangzhou.aliyuncs.com/example.jpg@100w


##常用接口##
**文件组件api接口介绍**

组件接口类SaasFileManager

<table style="border-collapse:collapse">
	<tr>
		<th>方法名</th>
		<th>参数</th>
		<th>返回值</th>
		<th>功能说明</th>
	</tr>

	<tr>
		<td>uploadFile</td>
		<td>
			1. String bucketName（bucket名）<br/>
			2. String tenantID（租户id）<br/>
			3. String fileName（上传文件名）<br/>
			4. byte[] fileContent（上传文件字节数组）<br/>
		</td>
		<td>String（上传后的文件名）</td>
		<td>上传文件</td>
	</tr>
	<tr>
		<td>downLoadFile</td>
		<td>
			1. String bucketName（下载文件所在bucket名）<br/>
			2. String fileName（下载文件名）<br/>
		</td>
		<td>byte[]（下载文件的二进制数组）</td>
		<td>下载文件</td>
	</tr>
	<tr>
		<td>deleteFile</td>
		<td>
			1. String bucketName（bucket名）<br/>
			2. String tenantID（租户id）<br/>
			3. String fileName（要删除的文件名）<br/>
		</td>
		<td>boolean（删除文件是否成功）</td>
		<td>删除文件</td>
	</tr>
	<tr>
		<td>getUrl</td>
		<td>
			1. String bucketName(bucket名)<br/>
			2. String tenantID（租户id）<br/>
			3. String fileName（获取url的文件名<br/>
			4. int expired（单位 秒 ，连接过期时间,目前该参数只支持阿里云oss私有bucket)<br/>
		</td>
		<td>String（文件的url）</td>
		<td>返回文件url</td>
	</tr>
	<tr>
		<td>getImgUrl</td>
		<td>
			1. String bucketName(bucket名)<br/>
			2. String tenantID（租户id）<br/>
			3. String fileName（获取url的文件名）<br/>
			4. int expired（单位 秒 ，连接过期时间,目前该参数只支持阿里云oss私有bucket）<br/>
		</td>
		<td>String（图片的url）</td>
		<td>
			返回图片url <br/>
			使用阿里云oss时在fileName加入类似@100h的参数能生成略缩图<br/>
			使用fdfs时在fileName加入类似@100h的参数能生成略缩图（需要布置nginx支持）<br/>
		</td>
	</tr>
</table>
	
更多详细说明，请参考阿里云的OSS相关的帮助文档。