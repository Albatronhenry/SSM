[mybaits中in超过1000处理](https://blog.csdn.net/qq_16765615/article/details/84616376)
-----------

```xml
原有写法：
 <if test=" form.getBis_id() != null and form.getBis_id().size()>0">
            and bis_id in        
        <foreach collection="form.getBis_id()" index="index" item="item" open="(" separator="," close=")">
        	#{item}
        </foreach>
</if>

修改后写法：
 <if test=" form.getBis_id() != null and form.getBis_id().size()>0">
            and bis_id in        
        <foreach collection="form.getBis_id()" index="index" item="item" open="(" separator="," close=")">
        	<if test="(index % 999) == 998"> NULL ) OR "bis_id" IN (</if>#{item}
        </foreach>
</if>


```
