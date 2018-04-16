springboot简单配置
---------------
基础入门中可以成功启动-->接下来就是创建相关controller包,注意

![bootpck](https://github.com/Albatronhenry/UploadFile/blob/master/pic/bootpck.png)

编写相应的controller类

```java

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController  //扫描
public class HelloworldController {
	
	@Value("${henry.msg}")  //获取context/yml文件中的数据值
	private String msg;
	
	@RequestMapping("/henry")
     public String SayHello(){
    	 return "hello , "+msg;
     }
}


```

两种不同配置文件对应如下,效果相同

![set](https://github.com/Albatronhenry/UploadFile/blob/master/pic/set.png)

![yml](https://github.com/Albatronhenry/UploadFile/blob/master/pic/yml.png)

实现springboot加载图标修改

![banner](https://github.com/Albatronhenry/UploadFile/blob/master/pic/banner.png)

运行效果如下:

![bootrun](https://github.com/Albatronhenry/UploadFile/blob/master/pic/bootrun.png)

