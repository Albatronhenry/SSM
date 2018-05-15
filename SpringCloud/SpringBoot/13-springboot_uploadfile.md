[springboot_uploadfile](http://412887952-qq-com.iteye.com/blog/2293385)
---------------------
* 注:本次使用springboot版本为2.x

#### 单文件上传

* 1.创建相应maven (web thymeleaf)项目
* 2.创建页面
```html
<!DOCTYPE html>

<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org"

      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Y-@</title>
    </head>
    <body>
       <form method="POST" enctype="multipart/form-data" action="/upload"> 
           <p>文件：<input type="file" name="file" /></p>
           <p><input type="submit" value="上传" /></p>
       </form>
    </body>
</html>
```
* 3.创建FileUploadController
```java
package com.yonyou.uploadfile.controller;

import java.io.BufferedOutputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.File;
import java.io.IOException;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.multipart.MultipartFile;

@Controller
public class FileUploadController {
	@RequestMapping("/file")
	public String File() {
		return "/file";
	}

	@RequestMapping("/upload")
	@ResponseBody
	public String uploadfile(@RequestParam("file") MultipartFile file) {
		if (!file.isEmpty()) {
			try {
				//文件上传存放路径
				File filesrc = new File("d:/upload/"+file.getOriginalFilename());
				//判断路径是否存在,不存在则创建
				if(!filesrc.getParentFile().exists()){
					filesrc.getParentFile().mkdirs();
				}
				//把文件读取到指定路径下
				BufferedOutputStream bo = new BufferedOutputStream(
						new FileOutputStream(filesrc));
				bo.write(file.getBytes());
				bo.flush();
				bo.close();
			} catch (FileNotFoundException e) {

				e.printStackTrace();

				return "上传失败," + e.getMessage();

			} catch (IOException e) {

				e.printStackTrace();

				return "上传失败," + e.getMessage();

			}

			return "上传成功";

		} else {

			return "上传失败，因为文件是空的.";

		}
	}
}
```
* 4.然后就可以访问：http://localhost:8080/file 进行测试了，如果没有指定文件上传的路径,默认是在工程的根路径下,这里指定为` d:/upload/ `
#### [解决最大上传文件大小问题](https://blog.csdn.net/u010429286/article/details/54381705)
* SpringBoot做文件上传时出现了The field file exceeds its maximum permitted size of 1048576 bytes.错误，显示文件的大小超出了允许的范围。
* 查看了官方文档，原来Spring Boot工程嵌入的tomcat限制了请求的文件大小，这一点在Spring Boot的官方文档中有说明，
* 每个文件的配置最大为1Mb，单次请求的文件的总数不能大于10Mb。要更改这个默认值需要在配置文件（如application.properties）中加入两个配置

```poperties
# springboot 1.3.x 
#multipart.maxFileSize=100Mb
#multipart.maxRequestSize=1000Mb

# springboot 1.4x
#spring.http.multipart.maxFileSize=100Mb
#spring.http.multipart.maxRequestSize=1000Mb

# springboot 2.0X
spring.servlet.multipart.max-file-size = 10Mb    
spring.http.multipart.max-request-size=100Mb    
```

