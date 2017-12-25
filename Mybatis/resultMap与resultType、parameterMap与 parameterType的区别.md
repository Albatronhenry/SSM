 ### resultMap 与 resultType、parameterMap 与  parameterType的区别



` resultMap ` & ` resultType `
       

      两者都是表示查询结果集与java对象之间的一种关系，处理查询结果集，映射到java对象。

      ` resultMap `表示将查询结果集中的列一一映射到bean对象的各个属性。映射的查询结果集中的列标签可以根据需要灵活变化，并且，在映射关系中，还可以通过typeHandler设置实现查询结果值的类型转换，比如布尔型与0/1的类型转换。

例如：

```xml

<resultMaptype="hdu.terence.bean.Message"id="MessageResult"> 

    <!--存放Dao值--><!--type是和数据库对应的bean类名Message-->

    <idcolumn="id"jdbcType="INTEGER"property="id"/><!--主键标签-->

    <resultcolumn="COMMAND"jdbcType="VARCHAR"property="command"/>

    <resultcolumn="DESCRIPTION"jdbcType="VARCHAR"property="description"/>

    <resultcolumn="CONTENT"jdbcType="VARCHAR"property="content"/>

  </resultMap>

 ```   
 ```xml

  <selectid="queryMessageList"parameterType="hdu.terence.bean.Message"resultMap="MessageResult">

    SELECTID,COMMAND,DESCRIPTION,CONTENT FROM message WHERE 1=1      

    <iftest="command!=null and!&quot;&quot;.equals(command.trim())">

    andCOMMAND=#{command}

    </if>

    <iftest="description!=null and!&quot;&quot;.equals(description.trim())">

    andDESCRIPTION like '%' #{description} '%'

    </if> 

  </select>

 ```

      resultType 表示的是bean中的对象类，此时可以省略掉resultMap标签的映射，但是必须保证查询结果集中的属性 和 bean对象类中的属性是一一对应的，此时大小写不敏感，但是有限制。

以下是resultType的写法，将其值设置成对应的java类上即可。不需要上述resultMap的映射关系。

```xml
  <selectid="queryMessageList"parameterType="hdu.terence.bean.Message"   resultType=" hdu.terence.bean.Message ">

    SELECTID,COMMAND,DESCRIPTION,CONTENT FROM message WHERE 1=1      

    <iftest="command!=null and!&quot;&quot;.equals(command.trim())">

    andCOMMAND=#{command}

    </if>

    <iftest="description!=null and!&quot;&quot;.equals(description.trim())">

    andDESCRIPTION like '%' #{description} '%'

    </if> 

  </select>
```

--------------------------

ParameterMap(不推荐) & parameterType
      

       ParameterMap和resultMap类似，表示将查询结果集中列值的类型一一映射到java对象属性的类型上，在开发过程中不推荐这种方式。

       一般使用parameterType直接将查询结果列值类型自动对应到java对象属性类型上，不再配置映射关系一一对应，例如上述代码中下划线部分表示将查询结果类型自动对应到hdu.terence.bean.Message的Bean对象属性类型。

     

Mybatis家族历史
     

    Mybatis出生于GoogleCode，使用的这两个名字叫做resultType和parameterType。

     以前的版本叫做iBatis，出生于Apache，以前这两个配置叫做resultClass和parrameterClass，根据这种命名也应该知道这种映射都和java类有关。

 

#{}和${}的使用


resultMap和ParameterMap书写拼写要使用#{}，resultType 和parameterType类型使用${}，使用例子如下：

` Select ID，COMMAND from Message where COMMAND=#{command} `

` Select ID，COMMAND from Message where COMMAND=‘${command}’ `

前者解析为：

         `   Select ID，COMMAND from Message where COMMAND=？具有预编译效果 `

后者解析为：

         `   Select ID，COMMAND from Message where COMMAND=段子   不具有预编译效果 `

 

但是，例如当页面向后台传递一个列名（属性名）的时候，是不希望被预编译出一个？的，此时要用到$格式；

如：加上` order by${param} `，此时` param `是一个列名。

 

#{}和 ognl表达式
     
![](http://img.blog.csdn.net/20170307212306148?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQ1NETl9UZXJlbmNl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


 [Reference website](http://blog.csdn.net/csdn_terence/article/details/60779889)
