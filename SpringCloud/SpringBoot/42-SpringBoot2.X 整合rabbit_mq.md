### [SpringBoot2.X 整合rabbitMq]([SpringBoot2.x整合RabbitMQ（完整版）_孔明的博客-CSDN博客](https://blog.csdn.net/fan521dan/article/details/104912794))

因项目需要，需要针对rabbitMq进行消息收发，故进行实现，本文采用topic模式实现。

### 1.pom依赖

```xml
        <!-- rabbitMQ -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

### 2.相关配置

#### 2.1 yml配置

```yml
spring:
  rabbitmq:
    addresses: ${spring.cloud.stream.binders.zk_rabbit.environment.rabbitmq.host}
    username: ${spring.cloud.stream.binders.zk_rabbit.environment.rabbitmq.username}
    password: ${spring.cloud.stream.binders.zk_rabbit.environment.rabbitmq.password}
  ##使用使用 Spring Cloud Stream来消费rabbitMQ的消息
  cloud:
    stream:
      binders:
        zk_rabbit:
          type: rabbit
          environment:
            rabbitmq:
              #地址环境
              host: 127.0.0.1:31027
              username: admin
              password: password
      bindings:
        ############权限接口消费者############
        #角色消费者-第一位
        roleTopicInput:
          destination: role_topic
          group: Common_PlatformYY
        #菜单消费者-第二位
        menuTopicInput:
          destination: menu_topic
          group: Common_PlatformYY
        #按钮消费者-第三位
        buttonTopicInput:
          destination: button_topic
          group: Common_PlatformYY
        #角色菜单按钮消费者-第四位
        roleMenuTopicInput:
          destination: role_menu_topic
          group: Common_PlatformYY
        #数据权限消费者-第五位
        permissionTopicInput:
          destination: permission_topic
          group: Common_PlatformYY
#处理权限数据后业务系统推送消息到消息平台
resultTopic:
    queue: result_topic
    exchange: result_topic
    routingKey: result_topic
```

#### 2.2 先在rabbitMq前台配置exchange及queue

![image-20210524202307927](https://raw.githubusercontent.com/Albatronhenry/UploadFile/master/pic/image-20210524202307927.png)



### 3.生产者实现

#### 3.1 配置类 ResultTopicConfig

```java
package com.***.abu.ds.common.rabbitmq.consumer;

import lombok.Getter;
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author linzyc
 * @desc 处理结果消息主题
 * @date 2021-05-24
 */
@Getter
@Configuration
public class ResultTopicConfig {


    /** queue name */
    @Value("${resultTopic.queue}")
    private String queueName;

    /** routing key */
    @Value("${resultTopic.routingKey}")
    private String routingKey;

    /** exchange */
    @Value("${resultTopic.queue}")
    private String exchange;

    /** create queue */
    @Bean
    public Queue queueMessage() {
        return new Queue(queueName);
    }

    /**
     * declare one exchange*/
    @Bean
    TopicExchange exchange() {
        return new TopicExchange(exchange);
    }

    /**
     *  queue & exchange binding assign routingKey
     * @param queueMessage
     * @param exchange
     * @return
     */
    @Bean
    Binding bindingExchangeMessage(Queue queueMessage, TopicExchange exchange) {
        return BindingBuilder.bind(queueMessage).to(exchange).with(routingKey);
    }
}
```

#### 3.2 业务代码实现部分

```java
import org.springframework.amqp.rabbit.core.RabbitTemplate;

@Service
@Slf4j
public class BaseCommonServiceImpl implements BaseCommonServiceI {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    private ResultTopicConfig resultTopicConfig;
    
     /**
     * 将处理后的数据结果反馈到消息平台
     * @param type
     * @param appId
     * @param successLs
     * @param failLs
     */
    @Override
    @SneakyThrows
    public void sendResultMsgToPlatForm(String type, String appId,List<String> successLs, List<String> failLs) {
        String updateTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMddHHmmss"));
        String msg = mapper.writeValueAsString(new JxdsResult(type,updateTime,appId,successLs,failLs));
        rabbitTemplate.convertAndSend(resultTopicConfig.getExchange(), resultTopicConfig.getRoutingKey(),msg);
        log.info("处理结果消息发送平台成功，消息对应exchange为：{}，对应routingKey为：{}，对应消息体为：{}",
                resultTopicConfig.getExchange(), resultTopicConfig.getRoutingKey(),msg);
    }
    
}
```

### 4.发送成功即可在rabbitMq界面看到总数增加

![image-20210524200306299](https://raw.githubusercontent.com/Albatronhenry/UploadFile/master/pic/image-20210524200306299.png)



### 5.消费者实现

#### 5.1采用spring cloud stream实现数据消费

##### RoleTopicReceiverI.java

```java
package com.***.abu.ds.common.rabbitmq.service;


import org.springframework.cloud.stream.annotation.Input;
import org.springframework.messaging.SubscribableChannel;

/**
 * 角色消费者接口
 * @author linzyc
 * @since 2021-05-15
 */
public interface RoleTopicReceiverI {
    String ROLE_TOPIC_INPUT= "roleTopicInput";

    @Input(RoleTopicReceiverI.ROLE_TOPIC_INPUT)
    SubscribableChannel roleTopicInput();
}

```

##### RoleTopicConsumer.java

```java
package com.***.abu.ds.common.rabbitmq.consumer;

import com.crux.common.exception.BusinessException;
import com..***..abu.ds.common.rabbitmq.service.RoleTopicReceiverI;
import com..***..abu.ds.common.util.DataUtil;
import com..***..abu.ds.service.province.common.BaseCommonServiceI;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Map;

/**
 * 角色消费者
 * @author linzyc
 * @since 2021-05-15
 */
@Component
@EnableBinding(value = {RoleTopicReceiverI.class})
@Slf4j
public class RoleTopicConsumer {

    @Autowired
    private BaseCommonServiceI baseCommonServiceI;

    @StreamListener(RoleTopicReceiverI.ROLE_TOPIC_INPUT)
    public void getBasMsg(String message) {
        log.info("jxsync-接收到角色消息：" + message);
        try {
            /** 1--正式环境访问环境，通过MQ获取的角色扩展数据 */
            Map<String, Object> msgMap = DataUtil.json2Object( message, Map.class);
            List list = (List) msgMap.get("APPIDS");
            if (list.contains(EnablePermission.YYZW)){
                String updateTime = (String) msgMap.get("UPDATE_TIME");
                baseCommonServiceI.syncPermissionInfo(EnablePermission.YYZW, updateTime, EnablePermission.ROLE);
            }else{
                log.info("sync-当前这批MQ没有YYZW的数据");
            }
        }catch (Exception e) {
            log.error("sync-接收RoleTopicConsumer-MQ异常"+e.getMessage(), e);
            throw new BusinessException("sync-接收RoleTopicConsumer"+e);
        }
    }
}
```