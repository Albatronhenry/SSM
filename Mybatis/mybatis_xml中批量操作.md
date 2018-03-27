### mybatis批量操作

oracle下支持执行多条语句，下面3个相同,注意` open ` ,  ` close ` 里面的写法

```xml

<update id="batchUpdate" parameterType="java.util.List">
        <foreach collection="list" item="item" index="index" open="begin" close=";end;" separator=";" > 
            update T_EMP_1 
            <set>       
                age = #{item.age}+1,name=#{item.name}
            </set>
            where id = #{item.id}
        </foreach>
    </update>
    
 ```
 
 ```xml
<update id="batchUpdate" parameterType="java.util.List">
        <foreach collection="list" item="item" index="index" open="begin" close="end;" separator="" > 
            update T_EMP_1 
            <set>       
                age = #{item.age}+1,name=#{item.name}
            </set>
            where id = #{item.id};
        </foreach>
    </update>

    
 ```
 
 ```xml
<update id="batchUpdate" parameterType="java.util.List">
        begin
        <foreach collection="list" item="item" index="index" separator="" > 
            update T_EMP_1 
            <set>       
                age = #{item.age}+1,name=#{item.name}
            </set>
            where id = #{item.id};
        </foreach>
        end;
    </update>
    
 ```
 
 mysql如下:
 
 ```xml
 <update id="batchUpdateStudent" parameterType="List">  
    UPDATE STUDENT SET name = "5566" WHERE id IN  
    <foreach collection="list" item="item" index="index" open="(" separator="," close=")" >  
        #{item}  
    </foreach>  
</update>
 
 ````
 
