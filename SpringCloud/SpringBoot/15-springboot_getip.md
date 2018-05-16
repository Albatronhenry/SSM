[springboot_getip](https://www.cnblogs.com/mr-wuxiansheng/p/7773121.html)
---------------------
* 1.创建一个maven(web)项目
* 2.创建ipUtil工具类
```java
package com.example.demo.util;

import java.net.InetAddress;
import java.net.UnknownHostException;

import javax.servlet.http.HttpServletRequest;

public class IpUtil {
    public static String getIpAddr(HttpServletRequest request) {
        String ipAddress = null;
        try {
            ipAddress = request.getHeader("x-forwarded-for");
            if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
                ipAddress = request.getHeader("Proxy-Client-IP");
            }
            if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
                ipAddress = request.getHeader("WL-Proxy-Client-IP");
            }
            if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
                ipAddress = request.getRemoteAddr();
                if (ipAddress.equals("127.0.0.1")) {
                    // 根据网卡取本机配置的IP
                    InetAddress inet = null;
                    try {
                        inet = InetAddress.getLocalHost();
                    } catch (UnknownHostException e) {
                        e.printStackTrace();
                    }
                    ipAddress = inet.getHostAddress();
                }
            }
            // 对于通过多个代理的情况，第一个IP为客户端真实IP,多个IP按照','分割
            if (ipAddress != null && ipAddress.length() > 15) { // "***.***.***.***".length()
                                                                // = 15
                if (ipAddress.indexOf(",") > 0) {
                    ipAddress = ipAddress.substring(0, ipAddress.indexOf(","));
                }
            }
        } catch (Exception e) {
            ipAddress="";
        }
        // ipAddress = this.getRequest().getRemoteAddr();
       System.err.println(ipAddress);
        return ipAddress;
    }
}
```
### 知识点

通过request.getRemoteAddr()的方法获取的IP实际上是代理服务器的地址，并不是客户端的IP地址。

#### 请求头的意思

    X-Forwarded-For

这是一个 Squid 开发的字段，只有在通过了HTTP代理或者负载均衡服务器时才会添加该项。

格式为X-Forwarded-For:client1,proxy1,proxy2，一般情况下，第一个ip为客户端真实ip，后面的为经过的代理服务器ip。现在大部分的代理都会加上这个请求头。

    Proxy-Client-IP/WL- Proxy-Client-IP

这个一般是经过apache http服务器的请求才会有，用apache http做代理时一般会加上Proxy-Client-IP请求头，而WL-Proxy-Client-IP是他的weblogic插件加上的头。

    HTTP_CLIENT_IP

有些代理服务器会加上此请求头。

    X-Real-IP
    nginx代理一般会加上此请求头。

#### 几点要注意

    这些请求头都不是http协议里的标准请求头，也就是说这个是各个代理服务器自己规定的表示客户端地址的请求头。如果哪天有一个代理服务器软件用oooo-client-ip这个请求头代表客户端请求，那上面的代码就不行了。

    这些请求头不是代理服务器一定会带上的，网络上的很多匿名代理就没有这些请求头，所以获取到的客户端ip不一定是真实的客户端ip。代理服务器一般都可以自定义请求头设置。

    即使请求经过的代理都会按自己的规范附上代理请求头，上面的代码也不能确保获得的一定是客户端ip。不同的网络架构，判断请求头的顺序是不一样的。

    最重要的一点，请求头都是可以伪造的。如果一些对客户端校验较严格的应用（比如投票）要获取客户端ip，应该直接使用ip=request.getRemoteAddr()，虽然获取到的可能是代理的ip而不是客户端的ip，但这个获取到的ip基本上是不可能伪造的，也就杜绝了刷票的可能。(有分析说arp欺骗+syn有可能伪造此ip，如果真的可以，这是所有基于TCP协议都存在的漏洞)，这个ip是tcp连接里的ip。


* 3.创建controller类
```java
package com.example.demo.controller;

import javax.servlet.http.HttpServletRequest;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

import com.example.demo.util.IpUtil;

@Controller
public class IpController {
	@RequestMapping(value = "/getIp")
    @ResponseBody
    public String getIp(HttpServletRequest request) {
        return IpUtil.getIpAddr(request);
    }
}
```
* 4.启动访问相应链接即可
