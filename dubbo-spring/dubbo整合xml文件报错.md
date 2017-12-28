### [dubbo整合xml文件报错](http://blog.csdn.net/zl544434558/article/details/53022776)

dubbo配置文件上会有红叉叉,不影响项目运行，项目中用maven依赖dubbo-2.5.3.jar。错误详情：

```

Multiple annotations found at this line:
    - cvc-complex-type.2.4.c: The matching wildcard is strict, but no declaration can be found for element 'dubbo:reference'.
    - schema_reference.4: Failed to read schema document 'http://code.alibabatech.com/schema/dubbo/dubbo.xsd', because 
     1) could not find the document; 2) the document could not be read; 3) the root element of the document is not <xsd:schema>.

```

* 解决办法： 
  * 1.找到maven仓库下dubbo-2.5.3.jar的目录(window中可以直接到maven本地库中去找到）

    mvn locate dubbo-2.5.3.jar

    会返回类似的路径 
    D:\SoftWareCustomer\Maven\maven_repository\repository\com\alibaba\dubbo\2.5.3

  * 2.在命令行中跳转到dubbo-2.4.9.jar所在目录，解压dubbo-2.5.3.jar文件

    cd D:\SoftWareCustomer\Maven\maven_repository\repository\com\alibaba\dubbo\2.5.3

    mvn jar xf dubbo-2.5.3.jar(也可以直接进入相应目录使用解压软件直接解压)

  * 3.在eclipse中window–->preferences–->XML–->XML Catalog 右边界面

    add按钮后会出现一个新的界面 

    选择file system 将刚才解压文件中

    META-INF/dubbo.xsd选中 
* 需要注意两点 
  * 1）key type 选择 schema location （默认是Namespace name) 
   * 2)key 一定要写http://code.alibabatech.com/schema/dubbo/dubbo.xsd 
      （默认http://code.alibabatech.com/schema/dubbo） 
![dubbo](https://github.com/Albatronhenry/UploadFile/blob/master/pic/dubbo.xsd%E6%96%87%E4%BB%B6%E5%AF%BC%E5%85%A5.png)

* 最后：

弄好之后记得更新这个maven项目，同时项目单击右键--》Vlidate,然后就大功告成了
