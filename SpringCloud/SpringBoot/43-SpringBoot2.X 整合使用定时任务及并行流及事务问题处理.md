### 使用quartz定时任务+stream().parallel()并行流+webClient异步客户端+事务控制

------

背景：因为项目需要，对数据进行上报，但是数据量比较大，故考虑通过定时任务加多线程进行数据上报，同时采用异步请求进行响应。

1.引用webClient

```xml
<!-- webflux -->
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

2.项目中使用及问题描述-主要是异步和定时任务导致的事务问题

```java
    
private volatile static WebClient webClient = null;

    initWebClient(accParam);

    @Describe("创建webClient")
    private void initWebClient(FfiiAccountingParam accParam) {
        if(webClient == null){
            synchronized (this){
                if(webClient == null) {
                    inSerName = accParam.getFinSerName();
                    webClient = WebClient.create(inSerName);
                }
            }
        }
    }

/** 
* 问题：一开始直接在定时任务中调用业务方法并开启事务，然后在事务方法中开启多线程，这样会导致spring通过事务代理失效
* 解决方案：定时任务中，通过并行流 ForkJion进行多线程调用新建的一个handler类的事务注解方法,这样可以保证一个线程里面一个事务，不会出现事务回滚无法回滚的情况
*/
 mofDivs.stream().parallel().forEach(mofDivCode -> iAccountingCornService.doIt(mofDivCode, fiscalYear, pageSize));

    /**业务处理方法*/
    @Transactional(rollbackFor = RuntimeException.class)
    public void doIt(String mofDivCode, String fiscalYear, String pageSize) {
        log.info(" {} ***定时任务当前工作线程号为 -> {}", mofDivCode, Thread.currentThread().getId());
        List<Map<String, Object>> queryData = queryData(mofDivCode, fiscalYear, pageSize);
        if(CollectionUtils.isEmpty(queryData)){
            log.info("{}***{}没有需要进行***上报的数据",fiscalYear,mofDivCode);
            return;
        }
        batchInsertLocal(queryData);
        ping();
        accountingCall(queryData, mofDivCode, fiscalYear);
    }

 @Describe("将待上报数据进行本地更新")
    public void batchInsertLocal(List<Map<String, Object>> data) {
        String insertSql = "insert into ffii_accounting_send(id,mof_div_code,fiscal_year,pay_date,send_time) values (?,?,?,?,?) ";
        jdbcTemplate.batchUpdate(insertSql, new BatchPreparedStatementSetter() {
            @Override
            public void setValues(PreparedStatement ps, int i) throws SQLException {
                Map<String, Object> stringObjectMap = data.get(i);
                ps.setString(1, (String) stringObjectMap.get("ID"));
                ps.setString(2, (String) stringObjectMap.get("MOF_DIV_CODE"));
                ps.setString(3, (String) stringObjectMap.get("FISCAL_YEAR"));
                ps.setString(4, (String) stringObjectMap.get("PAY_DATE"));
                ps.setTimestamp(5, Timestamp.valueOf(LocalDateTime.now()));
            }
            @Override
            public int getBatchSize() {
                return data.size();
            }
        });
    }

    @SneakyThrows
    @Describe("会计核算数据组装上报")
    public void accountingCall(List<Map<String, Object>> queryData, String mofDivCode, String fiscalYear) throws RuntimeException{
        Map<Object, List<Map<String, Object>>> groupData = queryData.stream().collect(Collectors.groupingBy(m -> m.get("AGENCY_CODE")));
        groupData.forEach((k,v)->{
            MaEleAcct maEleAcct = maEleAcctMapper.selectOne(new QueryWrapper<MaEleAcct>()
                    .eq("set_year", fiscalYear).eq("rg_code",mofDivCode).eq("agency_code",k));
            MaEleAgency maEleAgency = maEleAgencyMapper.selectOne(new QueryWrapper<MaEleAgency>()
                    .eq("set_year", fiscalYear).eq("rg_code",mofDivCode).eq("chr_code",k));
            String acctCode = maEleAcct.getChrCode();
            Map param = new ConcurrentHashMap(){{ put("agencyCode", k); put("acctCode", acctCode); put("setYear", fiscalYear);
                put("rgCode", mofDivCode); put("userId","87"); put("userCode","871"); put("roleId","77"); put("roleCode","771");
                put("userName","872"); put("roleName","772"); put("vouList", v); put("agencyTypeCode",maEleAgency.getDivKind());
                put("currentAgencyCode",k); put("sysId", RequestConstants.PAY.toUpperCase()); put("currentAcctCode",acctCode);
                put("transDate", LocalDate.now().toString()); put("billType","02"); put("generate","0"); put("mParam",fixedParam);
            }};
            exchangeDataByAccounting(RequestConstants.ACCOUNTING_URI, HttpMethod.POST, param);
        });
    }


/**
* 这里获取到webClient并进行请求
* response.block是阻塞的 response.subscribe是异步回调的
* 问题：暂不清楚为什么webClient的回调响应抛出异常时，会导致事务不进行回滚，目前采取了别的方案进行处理
更多关于webClient的使用可以查询相关资料进行补充学习，这里只是简单记录
*/
    @SneakyThrows
    @Describe("封装操作会计核算数据接口方法")
    public void exchangeDataByAccounting(String url, HttpMethod httpMethod, Object data) throws RuntimeException {
        log.info("exchangeDataByAccounting中参数url: {}，请求方式httpMethod: {},参数data: {}", url, httpMethod, jsonMapper.writeValueAsString(data));
        Mono<Object> response = AccountingCronTask.getWebClient().method(httpMethod).uri(url)
              .contentType(MediaType.APPLICATION_JSON).body(BodyInserters.fromValue(data)).retrieve().bodyToMono(Object.class);
        response.subscribe(result -> { batchUpdate((Map<String, Object>) result, (Map<String, Object>) data); } );
    }
```

