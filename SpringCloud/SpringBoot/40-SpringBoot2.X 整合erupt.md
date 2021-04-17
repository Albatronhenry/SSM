### SpringBoot2.X 整合erupt

1. #### pom.xml依赖

```xml
        <!-- erupt -->
        <!--auth-->
        <dependency>
            <groupId>xyz.erupt</groupId>
            <artifactId>erupt-upms</artifactId>
            <version>${erupt.version}</version>
        </dependency>
        <!--security-->
        <dependency>
            <groupId>xyz.erupt</groupId>
            <artifactId>erupt-security</artifactId>
            <version>${erupt.version}</version>
        </dependency>
        <!--WEB-->
        <dependency>
            <groupId>xyz.erupt</groupId>
            <artifactId>erupt-web</artifactId>
            <version>${erupt.version}</version>
        </dependency>
```

2. #### 启动类加上相关注解,不然前台打不开

```java
@ComponentScan({"xyz.erupt","com.xxx"}) // ↓ com.xxx要替换成实际需要扫描的代码包
@EntityScan({"xyz.erupt","com.xxx"})    // ↓ 例如DemoApplication所在的包为 com.example.demo
@EruptScan({"xyz.erupt","com.xxx"})     // → 则：com.xxx → com.example.demo
```

3. #### application.yml 初始化配置自动加载数据库表 oracle为例

```yml
spring:
  jpa:
    open-in-view: true
    hibernate:
      ddl-auto: update
    show-sql: true
    generate-ddl: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.Oracle10gDialect
        cache:
          use_second_level_cache: false
```

4. #### [修改登录界面相关配置为对应应用配置](https://www.yuque.com/erupts/erupt/gtp7iw)

​     4.1 项目位置：/resources/public/app.js 

```js
window.eruptSiteConfig = {
    //标题
    title: "XXXXXXX",
    //描述
    desc: "XXXX接口",
    //logo路径
    logoPath: "erupt.svg",
    //logo文字
    logoText: "XXXXXXXX",
    //是否展示版权信息（1.6.10及以后版本支持）
    copyright: false
};
```

5. 图标文件配置

![image-20210415193803152](https://raw.githubusercontent.com/Albatronhenry/UploadFile/master/pic/image-20210415193803152.png)

6.登录 http://ip:port/                   默认用户、密码    erupt/erupt



7.代理实现控制

```java
package com.***.abu.ffii.entity.erupt.user;

import lombok.Getter;
import lombok.Setter;
import xyz.erupt.annotation.Erupt;
import xyz.erupt.annotation.EruptField;
import xyz.erupt.annotation.sub_erupt.Power;
import xyz.erupt.annotation.sub_field.Edit;
import xyz.erupt.annotation.sub_field.EditType;
import xyz.erupt.annotation.sub_field.View;
import xyz.erupt.annotation.sub_field.sub_edit.InputType;
import xyz.erupt.annotation.sub_field.sub_edit.Search;
import xyz.erupt.jpa.model.BaseModel;

import javax.persistence.Entity;
import javax.persistence.Table;

/**
 * @author linzyc
 * @desc 用户管理
 * @date 2021-04-15
 */
@Getter
@Setter
@Entity
@Table(name = "ffii_mock_user")
@Erupt(name = "用户管理",
        power = @Power(importable = true,export = true, add = true, delete = true,edit = true,query = true),
        /* 代理操作类 */
        dataProxy = {FiscalUserEruptDataProxy.class}
)
public class FiscalUserErupt extends BaseModel{

    @EruptField(
            views = @View(title = "用户账号"),
            edit = @Edit(title = "用户账号", notNull = true, search = @Search)
    )
    private String code;

    @EruptField(
            views = @View(title = "区划"),
            edit = @Edit(title = "区划", notNull = true, search = @Search)
    )
    private String mofDivCode;

    @EruptField(
            views = @View(title = "密码"),
            edit = @Edit(title = "密码",
                    type = EditType.INPUT, notNull = true,
                    inputType = @InputType(type = "password")
            )
    )
    private String password;

}
```

增加代理实现类：

```java
package com.***.abu.ffii.entity.erupt.user;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.***.abu.ffii.common.config.FfiiConfig;
import com.***.abu.ffii.common.constants.RequestConstants;
import com.***.abu.ffii.service.fiscal.setting.ISettingService;
import com.***.abu.ffii.util.RsaUtils;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.web.client.HttpClientErrorException;
import org.springframework.web.client.RestTemplate;
import xyz.erupt.annotation.fun.DataProxy;
import xyz.erupt.core.exception.EruptApiErrorTip;

import java.util.Collection;
import java.util.Iterator;
import java.util.Map;

/**
 * @author linzyc
 * @desc ...
 * @date 2021-04-16
 */
@Slf4j
@Service
public class FiscalUserEruptDataProxy implements DataProxy<FiscalUserErupt> {

    @Autowired
    private FfiiConfig ffiiConf;

    @Autowired
    private ISettingService iSettingService;

    @Autowired
    private RestTemplate restTemplate;

    private static final ObjectMapper jsonMapper = new ObjectMapper();

    @Override
    public void beforeAdd(FiscalUserErupt fiscalUser) {
        checkAfter(fiscalUser,"添加");
    }

    @Override
    public void beforeUpdate(FiscalUserErupt fiscalUser) {
        checkAfter(fiscalUser,"更新");
    }

    /**
     * 查询数据后进行脱敏
     * @param list
     */
    @Override
    public void afterFetch(Collection<Map<String, Object>> list) {
        list.forEach( item ->{
                    Iterator<Map.Entry<String, Object>> iterator = item.entrySet().iterator();
                    while(iterator.hasNext()){
                        Map.Entry<String, Object> next = iterator.next();
                        if(RequestConstants.GRANT_TYPE.equals(next.getKey())){
                            next.setValue(RequestConstants.HIDE_PASSWORD);
                        }
                    }
                }
        );
    }

    /***
     * 编辑查看密码脱敏
     * @param fiscalUser
     */
    @Override
    public  void editBehavior(FiscalUserErupt fiscalUser) {
        fiscalUser.setPassword(RequestConstants.HIDE_PASSWORD);
    }

    /**
     * 用户校验
     * @param fiscalUser
     * @param method
     */
    private void checkAfter(FiscalUserErupt fiscalUser, String method){
        /** 加用户之前调用token接口，看用户是否正常 */
        String userCode = fiscalUser.getCode();
        String password = fiscalUser.getPassword();
        if(checkToken(userCode,password)){
            throw new EruptApiErrorTip(userCode+"用户"+method+"失败，请核实用户编码或密码后重试");
        };
        if(Boolean.parseBoolean(iSettingService.getSettingParam().getIsCipher())) {
            password = RsaUtils.encryptByPublicKey(password, ffiiConf.getPublicKey());
            fiscalUser.setPassword(password);
        }
    }


    /**
     * 校验增加用户的有效性
     * @param userCode 用户编码
     * @param password 密码
     * @return boolean true 验证失败 false 验证成功
     */
    @SneakyThrows
    private boolean checkToken(String userCode, String password) {
        String url = iSettingService.getSettingParam().getFisSerName().concat(RequestConstants.GRANT_URI);
        MultiValueMap<String, String> requestBody = new LinkedMultiValueMap<>();
        requestBody.add("scope", RequestConstants.SCOPE);
        requestBody.add("password", password);
        requestBody.add("username", userCode);
        requestBody.add("grant_type", RequestConstants.GRANT_TYPE);
        HttpHeaders requestHeaders = new HttpHeaders();
        requestHeaders.add(RequestConstants.TOKEN_AUTH, RequestConstants.DEFAULT_AUTH_HEADER);
        HttpEntity<MultiValueMap> requestEntity = new HttpEntity<>(requestBody, requestHeaders);
        ResponseEntity<String> responseEntity = null;
        try{
            responseEntity = restTemplate.postForEntity(url, requestEntity, String.class);
        }catch (HttpClientErrorException e){
            log.error("增加用户时，用户编码或密码错误导致的认证异常 {}",e.getMessage());
            return true;
        }
        Map<String, Object> map = jsonMapper.readValue(responseEntity.getBody(), Map.class);
        if(map.get("access_token") == null){
            return true;
        };
        return false;
    }
}

```

