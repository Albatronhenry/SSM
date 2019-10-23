### 返回实体相应驼峰命名字段均为null
-----------
```json
{
	"code": 200,
	"msg": "操作成功！",
	"data": {
		"page": {
			"records": [ {
				"id": "3291954347015517240",
				"fromctrlid": "3291954108536544447",
				"toctrlid": "3291954355186785502",
				"en_name": null,
				"bs_name": null,
				"bis_name": null,
				"mk_name": null,
				"pk_name": null,
                                    ...
				"bl_name": null
			}],
			"total": 97,
			"size": 10,
			"current": 1,
			"orders": [],
			"searchCount": true,
			"pages": 10
		},
		"CountTotalMoney": {
			"ROW_COUNT": 97,
			"TOTAL_MONEY": 396290871.39,
			"ROW_ID": 1
		}
	},
	"timestamp": "1571828549091"
}
```
原本写法：
-----------
```xml
<!-- 分页查询数据 -->
    <select id="getPage" resultType="BankAmountEntity">
        SELECT <include refid="searchField"></include> 
        <include refid="queryCondition"></include>        
    </select>

```

原因：字段未映射上

解决办法：将resultType换成resultMap

--------------
```xml

	<resultMap type="com.yonyougov.report.entity.budget.BankAmountEntity" id="bankAmountEntity">
		<result property="id" column="ID" jdbcType="VARCHAR"/>
        <result property="fromctrlid" column="FROMCTRLID" jdbcType="VARCHAR"/>
        <result property="toctrlid" column="TOCTRLID" jdbcType="VARCHAR"/>
        <result property="voucher_no" column="VOUCHER_NO" jdbcType="VARCHAR"/>
        <result property="gov_bsi_name" column="GOV_BSI_NAME" jdbcType="VARCHAR"/>
        <result property="approve_date" column="APPROVE_DATE" jdbcType="VARCHAR"/>
        <result property="bs_name" column="BS_NAME" jdbcType="VARCHAR"/>
                     ...
        <result property="bis_name" column="BIS_NAME" jdbcType="VARCHAR"/>
	</resultMap>

	<!-- 分页查询数据 -->
    <select id="getPage" resultMap="bankAmountEntity">
        SELECT <include refid="searchField"></include> 
        <include refid="queryCondition"></include>        
    </select>
```
