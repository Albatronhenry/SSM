[springboot邮件发送](http://www.ityouknow.com/springboot/2017/05/06/springboot-mail.html)
----------------
#### 开发前提: 对应邮箱的pop3/smtp要开启,并获取相应授权码

* 1.创建一个简单的maven Mail项目
* 2.在配置文件中添加:
```properties
spring.mail.host=smtp.qq.com
spring.mail.username=mr.lin1994@foxmail.com  //邮箱账号
spring.mail.password=txqlxxxxxxxxxxsbahd        //授权码
spring.mail.default-encoding=UTF-8          
// [下面三行代码一定要加上,不然会报530的错](https://blog.csdn.net/u011244202/article/details/54809696) 
spring.mail.properties.mail.smtp.auth=true  
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
mail.fromMail.addr=mr.lin1994@foxmail.com    //发送邮件的邮箱账号
```

* 3、编写mailServiceImpl实现类,接口类省略
```java
@Component
public class MailServiceImpl implements MailService{

    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    @Autowired
    private JavaMailSender mailSender;

    @Value("${mail.fromMail.addr}")
    private String from;

    @Override
    public void sendSimpleMail(String to, String subject, String content) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from);
        message.setTo(to);
        message.setSubject(subject);
        message.setText(content);

        try {
            mailSender.send(message);
            logger.info("邮件已经发送。");
        } catch (Exception e) {
            logger.error("发送邮件时发生异常！", e);
        }

    }
}
```
* 4、编写test类进行测试
```java
package com.yonyou.email;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.yonyou.email.interf.MailService;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootEmailApplicationTests {
	 @Autowired
	    private MailService MailService;
	@Test
	public void contextLoads() {
		 MailService.sendSimpleMail("3082849886@qq.com","test simple mail"," hello ,brother , this is simple mail");
		   
	}

}
```
* 发送成功页面
![spring_mail](https://github.com/Albatronhenry/UploadFile/blob/master/pic/spring_mail.png)
