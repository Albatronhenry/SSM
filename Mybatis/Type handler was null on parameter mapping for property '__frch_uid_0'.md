### [Type handler was null on parameter mapping for property '__frch_uid_0'](https://bbs.csdn.net/topics/392146710)

异常如下:

nested exception is org.apache.ibatis.exceptions.PersistenceException: 
### Error querying database.  Cause: java.lang.IllegalStateException: Type handler was null on parameter mapping for 
property '__frch_uid_0'.  It was either not specified and/or could not be found for the javaType / jdbcType combination specified.
### Cause: java.lang.IllegalStateException: Type handler was null on parameter mapping for property '__frch_uid_0'.  It was either 
not specified and/or could not be found for the javaType / jdbcType combination specified.

原因:mapping映射的字段类型给错,有的字段在 mybatis-mapper.xml中声明了,却未传相应属性值,或者传了相应属性值,xml文件语句中却没有相关字段导致
