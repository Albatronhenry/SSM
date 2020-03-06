> # 29-springboot2.X整合FDFS分布式文件系统
>
> [参考1](https://www.cnblogs.com/lr393993507/p/10336547.html)|[参考2](https://blog.csdn.net/MakeLoveWith/article/details/82666062?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task
> )

###### 1.pom.xml依赖

```xml
<!-- fdfs -->
		 <dependency>
            <groupId>com.github.tobato</groupId>
            <artifactId>fastdfs-client</artifactId>
            <version>1.26.2</version>
        </dependency>
```

###### 2.yml配置

```yml
#Fdfs配置
fdfs:
  so-timeout: 1501
  connect-timeout: 2000
  thumb-image:             #缩略图生成参数
    width: 150
    height: 150
  #TrackerList参数,支持多个 x.x.x.62:22122,x.x.x.66:22122
  tracker-list: x.x.x.62:22122
```

> ###### 3.启动类加载，处理启动问题
>
> [整合问题参考](https://blog.csdn.net/qq_34495753/article/details/102851142)
>
> 3.1启动注解加载
>
> ```java
> //增加fdfs相关
> @EnableMBeanExport(registration = RegistrationPolicy.IGNORE_EXISTING)
> @Import(FdfsClientConfig.class)
> public class DfZjsbApplication {
> 	public static void main(String[] args) {
> 		SpringApplication.run(DfZjsbApplication.class, args);
> 	}
> 
> }
> ```
>
> 

###### 4.FDFS文件上传下载删除实现

```java
import java.io.File;
import java.io.IOException;
import java.net.URLEncoder;
import java.util.List;
import java.util.Set;

import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.io.FilenameUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import com.github.tobato.fastdfs.domain.MataData;
import com.github.tobato.fastdfs.domain.StorePath;
import com.github.tobato.fastdfs.proto.storage.DownloadByteArray;
import com.github.tobato.fastdfs.proto.storage.DownloadCallback;
import com.github.tobato.fastdfs.service.FastFileStorageClient;
import com.google.common.collect.Lists;
import com.google.common.collect.Sets;
import com.microsoft.sqlserver.jdbc.StringUtils;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;

/**
 * 附件操作
 * 
 * @author linzyc
 *
 */
@RestController
@RequestMapping("/file")
@Slf4j
@Api(description = "fastdfs接口API")
public class FileController {
	
	@Autowired
	private FastFileStorageClient fastFileStorageClient;
	
	/**
	 * 文件上传(支持多文件)
	 * @throws IOException 
	 */
    @ApiOperation("文件上传")
	@PostMapping(value = "/upload",headers = "content-type=multipart/form-data")
	public Result upload(@RequestParam("file") MultipartFile[] files) throws IOException {
		// TODO 存放文件保存url地址及其他信息到数据库，具体待定
		for(int i = 0,size = files.length;i<size;i++) {
			MultipartFile file = files[i];
			int j = i+1;
			if (file.isEmpty()) {
				return ResultUtil.error("上传失败，选择的第"+j+"文件为空");
			}
			// 原文件名
			String oriFileName = file.getOriginalFilename();
			Set<MataData> metaData = Sets.newHashSet();
			metaData.add(new MataData("name",file.getName()));
			metaData.add(new MataData("description",file.getContentType()));
			StorePath uploadFile = fastFileStorageClient.uploadFile(file.getInputStream(), file.getSize(), FilenameUtils.getExtension(file.getOriginalFilename()), metaData);
			String fileUrl = getResAccessUrl(uploadFile);
			log.info("上传第{}个文件，文件名为：{}，路径为：{} ",j,oriFileName,fileUrl);
		}
		//TODO 具体上传路径、相关信息需要存入数据库，这在业务中实现
		return ResultUtil.success(ResultEnums.SUCCESS, "上传成功");
	}	

	/**
	 * 文件下载
	 * @throws IOException 
	 */
    @ApiOperation("文件下载")
	@PostMapping("/download")
	public void download(@RequestParam("fileUrl") String fileUrl,@RequestParam("oriFileName") String oriFileName, HttpServletResponse response) throws IOException {
		// TODO 具体业务实现定	
		log.info("文件路径为：{} ，原始文件名为：{}",fileUrl,oriFileName);
    	StorePath praseFromUrl = StorePath.praseFromUrl(fileUrl);
		String group = praseFromUrl.getGroup();
		String path = praseFromUrl.getPath();
		DownloadCallback<byte[]> callback = new DownloadByteArray();
		//获取文件
		byte[] bytes = fastFileStorageClient.downloadFile(group, path, callback);
		response.reset();
        //设置相应类型application/octet-stream （注：applicatoin/octet-stream 为通用）
		response.setContentType("applicatoin/octet-stream");
		response.setHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(oriFileName, "UTF-8"));
		ServletOutputStream outputStream = response.getOutputStream();
		outputStream.write(bytes);
		outputStream.close();
	}	

	/**
	 * 附件删除
	 */
    @ApiOperation("文件删除")
	@PostMapping("/delete")
	public Result delete(@RequestParam("fileUrl") String fileUrl) {
    	if(StringUtils.isEmpty(fileUrl)) {
    		return ResultUtil.error("文件路径为空，无法删除");
    	}
    	StorePath praseFromUrl = StorePath.praseFromUrl(fileUrl);
    	fastFileStorageClient.deleteFile(praseFromUrl.getGroup(), praseFromUrl.getPath());
    	return ResultUtil.success(ResultEnums.SUCCESS, "删除成功");
	}
    
   /**
    * 路径转换
    */
	 private String getResAccessUrl(StorePath storePath) {
	        //如果需要拿取地址ip，需引入 FdfsWebServer  String fileUrl = fdfsWebServer.getWebServerUrl() + storePath.getFullPath();
	        String fileUrl = storePath.getFullPath();
	        return fileUrl;
	    }
	 
	/**
	 * 附件查询 --内部方法，加载单据时调用此方法根据主单据单号对应附件
	 */
	  public Result fileQuery(String code) {
			// TODO 具体业务实现时间待定
			List<File> fileLs = Lists.newArrayList();
			return ResultUtil.success(ResultEnums.SUCCESS, fileLs);
		}
}

```

