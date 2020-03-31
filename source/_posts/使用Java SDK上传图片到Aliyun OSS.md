### 前言：

​		**在项目中需要上传图片，所以想到使用阿里云的OSS对象存储服务来存储图片，方便项目后续的上线部署。在这里我采用的是Java SDK的方式去上传图片，前端引入Markdown来测试图片上传功能。**



#### 在使用 Java SDK上传图片之前，需要做好几项准备

1、上[阿里云]([https://www.aliyun.com](https://www.aliyun.com/))官网购买对象存储OSS相应的服务.

2、进入OSS控制台，创建Bucket，Bucket是一个独立的存储空间，相当于电脑的磁盘，可以在里面建立各种子目录，如下，Bucket名是`chen-xi`，包含三个子目录.

![bucket-detail](/images/bucket.PNG)

3、根据阿里云推荐的策略创建AccessKey，这是用来验证身份的钥匙，在后续编程时需要用到.

4、我使用的是maven，需要引入依赖，如果是其他方式可以在官方文档里直接下载压缩包导入.

```xml
 <dependency>
      <groupId>com.aliyun.oss</groupId>
      <artifactId>aliyun-sdk-oss</artifactId>
      <version>2.7.0</version>
 </dependency>
```

5、配置application-properties.

```xml
<!-- accessKey值去阿里云官网查询 -->
aliyun.oss.access-key-id=<accessKeyId>		
aliyun.oss.access-key-secret=<accessKeySecret>
aliyun.oss.access-uri=http://chen-xi.oss-cn-beijing.aliyuncs.com
aliyun.oss.endpoint=http://oss-cn-beijing.aliyuncs.com
aliyun.oss.bucket-name=chen-xi
```



#### 使用 Java SDK上传图片：基于Spring Boot项目

**首先创建上传文件的工具类：**

```java
import com.aliyun.oss.OSS;
import com.aliyun.oss.OSSClientBuilder;
import com.aliyun.oss.model.ObjectMetadata;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.io.InputStream;

/**
 * @Author: Mr.Chen
 * @Description:使用Java SDK上传文件到Aliyun OSS
 * @Date:Created in 20:43 2020/3/29
 */
@Service
public class AliyunOssProvider {
	
    /**
     *  通过配置文件注入相应的值
     */
    @Value("${aliyun.oss.access-key-id}")
    private String accessKeyId;
    @Value("${aliyun.oss.access-key-secret}")
    private String accessKeySecret;
    @Value("${aliyun.oss.endpoint}")
    private String endPoint;
    @Value("${aliyun.oss.bucket-name}")
    private String bucketName;
    @Value("${aliyun.oss.access-uri}")
    private String accessUrl;

    OSS client = null;
	
     /**
     * 上传文件
     * @param multipartFile 接收上传文件的接口
     * @param remotePath bucket下的子目录名
     * @return 文件的路径
     * @throws IOException 抛出IO异常
     */
    public String uploadFile(MultipartFile multipartFile, String remotePath) throws IOException {
        //将multipartFile转换成需要的IO流
        InputStream fileContent = multipartFile.getInputStream();
        //获取文件名
        String fileName = multipartFile.getOriginalFilename();
        //修改文件名,保证文件名不重复
        fileName = "tcx_" + System.currentTimeMillis() + fileName.substring(fileName.lastIndexOf("."));
        //定义二级目录
        String remoteFilePath = remotePath.replaceAll("\\\\", "/") + "/";
        //初始化OSS客户端
        client = new OSSClientBuilder().build(endPoint, accessKeyId, accessKeySecret);
    	//获取OSS对象的元数据
        ObjectMetadata objectMetadata = new ObjectMetadata();
        //设置必要的属性
        objectMetadata.setContentLength(fileContent.available());
        objectMetadata.setContentEncoding("utf-8");
        objectMetadata.setCacheControl("no-cache");
        objectMetadata.setHeader("Pragma", "no-cache");
        objectMetadata.setContentType(fileName.substring(fileName.lastIndexOf(".")));
        objectMetadata.setContentDisposition("inline;filename=" + fileName);
        //上传文件
        client.putObject(bucketName, remoteFilePath+fileName , fileContent, objectMetadata);
        //关闭相应的流
        client.shutdown();
        fileContent.close();
        //返回文件路径
        return accessUrl + "/" + remoteFilePath + fileName;
    }
}

```



**其次创建Controller来调用：**

```java
import com.chenxi.community.dto.FileDTO;
import com.chenxi.community.provider.AliyunOssProvider;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.multipart.MultipartHttpServletRequest;

import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

/**
 * @Author: Mr.Chen
 * @Description: 文件上传控制器
 * @Date:Created in 20:43 2020/3/29
 */
@Controller
public class FileUploadController {

    @Autowired
    private AliyunOssProvider aliyunOssProvider;

    @ResponseBody	//返回Json对象，展示到前端
    @RequestMapping(value = "/file/upload")
    public FileDTO upload(HttpServletRequest request) {
        //获取上传文件请求
        MultipartHttpServletRequest multipartRequest = (MultipartHttpServletRequest) request;
        //从前端的元素获取文件对象，此处是Markdown的图片对象
        MultipartFile file = multipartRequest.getFile("editormd-image-file");
        String fileName = null;
        FileDTO fileDTO = new FileDTO();
        //文件不为空
        if (file != null) {
            try {
                //上传图片文件
                fileName = aliyunOssProvider.uploadFile(file, "images");
                //上传成功
                fileDTO.setSuccess(1);
                fileDTO.setUrl(fileName);
            } catch (IOException e) {
                //上传文件出现异常
                fileDTO.setSuccess(0);
                fileDTO.setMessage("上传失败");
            }
        } else {  
            System.out.println("选择的文件为空，请重新尝试上传");
        }
        return fileDTO;
    }
}

```



**Controller里使用到的FileDTO类：**

```java
import lombok.Data;

/**
 * @Author: Mr.Chen
 * @Description: 文件数据传输类
 * @Date:Created in 20:45 2020/3/29
 */
@Data
public class FileDTO {
    /**
     *  0 表示上传失败，1 表示上传成功
     */
    private Integer success;
    /**
     * 提示信息，上传成功或上传失败及错误信息等
     */
   
    private String message;
    /**
     * 文件地址，上传成功时才返回
     */
    private String url;              
}

```



**最后进行测试：前端引入Markdown来测试图片上传功能**

第一步：前端展示

![](/images/markdown.PNG)

第二步：测试用例--本地上传图片

<img src="/images/upload-image1.PNG" style="zoom:110%;" />

<img src="/images/upload-image2.PNG" style="zoom:110%;" />

第三步：验证上传结果

​			markdown编辑器返回图片路径，能正确预览：

<img src="/images/upload-image3.PNG" style="zoom:75%;" />

​			Aliyun OSS的bucket子目录images下，生成了该图片文件：

<img src="/images/upload-success.PNG" style="zoom:75%;" />





*Thank you.*





