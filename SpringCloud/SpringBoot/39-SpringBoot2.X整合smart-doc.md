### SpringBoot 2.X 整合smart-doc

1.pom引入插件

```xml
 <!-- smart-doc -->
            <plugin>
                <groupId>com.github.shalousun</groupId>
                <artifactId>smart-doc-maven-plugin</artifactId>
                <version>2.1.1</version>
                <configuration>
                    <configFile>./src/main/resources/smart-doc.json</configFile>
                    <excludes>
                        <exclude>com.google.guava:guava</exclude>
                    </excludes>
                    <includes>
                        <include>com.alibaba:fastjson</include>
                    </includes>
                </configuration>
                <executions>
                    <execution>
                        <phase>compile</phase>
                        <goals>
                            <goal>html</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```

2.classpath-resources路径下添加配置文件smart-doc.json

```json
{
  "serverUrl": "http://{{serverName}}",
  "outPath": "src/main/resources/static/doc",   //对应输出文件路径
  "md5EncryptedHtmlName": true,
  "projectName": "xxx中间服务接口",
  "skipTransientField": true,
  "showAuthor":false,
  "revisionLogs": [
    {
      "version": "1.0",
      "status": "realise",
      "author": "linzyc",
      "remarks": "smart-doc使用接口文档"
    }
  ]
}
```

3.maven进行编译安装

![image-20210415121403798](https://raw.githubusercontent.com/Albatronhenry/UploadFile/master/pic/image-20210415121403798.png)

4.文件生成到配置文件中的指定路径下

![image-20210415122047681](https://raw.githubusercontent.com/Albatronhenry/UploadFile/master/pic/image-20210415122047681.png)

5.使用对应浏览器进行浏览即可

![image-20210415122013477](https://raw.githubusercontent.com/Albatronhenry/UploadFile/master/pic/image-20210415122013477.png)

[几个在线文档生成神器](https://mp.weixin.qq.com/s/C0HDnsX9tclRODkER2f6sA)

