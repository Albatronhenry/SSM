### [springboot2.X关于jpa查询出来的数据总是重复](https://blog.csdn.net/qq_23434557/article/details/88390065#jpa_list_1)

项目部署测试发现，每次调用后台查询（使用jpa 利用 EntityManager 写自定义原生 sql 语句的时候，返回自定义entity类中），主键重复导致查询list的数据总是重复第一条数据

解决方案：自定义entity实体类增加id属性，同时修改原生sql语句
```java

△ 实体类增加id字段

@Entity
@ToString
@Data
public class JobsParamsUnionEntity  implements Serializable{
	/**
	 * id 处理jpa查询数据重复
	 */
	@Id
	private Long id;
  
  ...其余省略
}
```

```java
//oracle语句写法 rownum AS id，已验证
@Query(value = "select rownum AS id,p.p_code as p_code,p.p_value as p_value,p.table_name as table_name,j.j_time as j_time,j.m_id as m_id,j.db_id as db_id from cli_job j LEFT JOIN cli_job_param p ON p.j_id = j.j_id where  p.created_by=?1 and p.set_year=?2 and p.rg_code=?3 and p.j_id=?4 ",nativeQuery = true)
//mysql语句写法  @rownum \\:= @rownum +1 AS id,未验证
@Query(value = "select @rownum \\:= @rownum +1 AS id,p.p_code as p_code,p.p_value as p_value,p.table_name as table_name,j.j_time as j_time,j.m_id as m_id,j.db_id as db_id from cli_job j LEFT JOIN cli_job_param p ON p.j_id = j.j_id where  p.created_by=?1 and p.set_year=?2 and p.rg_code=?3 and p.j_id=?4 ",nativeQuery = true)

```
