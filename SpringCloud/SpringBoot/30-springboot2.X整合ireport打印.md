# springboot2.X整合ireport打印

背景：因项目需要打印相关单据（批量勾选打印，以及打印不能空白）

------

### POM依赖(公网需加载别的依赖)

```xml
<dependency>
			<groupId>commons-digester</groupId>
			<artifactId>commons-digester</artifactId>
			<version>1.8</version>
		</dependency>

		<dependency>
			<groupId>com.ufgov.gfmis</groupId>
			<artifactId>ifmis-ltreport</artifactId>
			<version>${fasp.version}</version>
		</dependency>

		<dependency>
			<groupId>com.ufgov.gfmis</groupId>
			<artifactId>iText-2.1.7</artifactId>
			<version>${fasp.version}</version>
		</dependency>

		<dependency>
			<groupId>com.lowagie</groupId>
			<artifactId>iTextAsian</artifactId>
			<version>1.0</version>
		</dependency>

		<dependency>
			<groupId>com.ufgov.gfmis</groupId>
			<artifactId>itruscom1.4</artifactId>
			<version>${fasp.version}</version>
		</dependency>

		<dependency>
			<groupId>janino</groupId>
			<artifactId>janino</artifactId>
			<version>${fasp.version}</version>
		</dependency>
		<dependency>
			<groupId>net.sf.jasperreports</groupId>
			<artifactId>jasperreports</artifactId>
			<version>6.9.0-yygov-1.0</version>
			<exclusions>
				<exclusion>
					<groupId>com.lowagie</groupId>
					<artifactId>itext</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>com.yonyougov</groupId>
			<artifactId>jasperreports-ext</artifactId>
			<version>1.0.0</version>
		</dependency>

		<dependency>
			<groupId>net.sf.jasperreports</groupId>
			<artifactId>jasperreports-fonts</artifactId>
			<version>6.9.0</version>
		</dependency>

		<dependency>
			<groupId>org.codehaus.groovy</groupId>
			<artifactId>groovy-all</artifactId>
			<version>2.4.11</version>
		</dependency>

<!-- https://mvnrepository.com/artifact/joda-time/joda-time -->
		<dependency>
    		<groupId>joda-time</groupId>
    		<artifactId>joda-time</artifactId>
   			 <version>${joda-time.version}</version>
		</dependency>
```

### PrintHelper工具类

```java
package com.*****.ywsp.util;

import java.io.ByteArrayOutputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.util.List;
import java.util.Map;

import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.core.io.ClassPathResource;
import org.springframework.util.ClassUtils;

import com.*****.ywsp.common.constants.PrintConstants;
import com.*****.ywsp.common.exception.BaseException;

import lombok.extern.slf4j.Slf4j;
import net.sf.jasperreports.engine.JRDataSource;
import net.sf.jasperreports.engine.JRException;
import net.sf.jasperreports.engine.JasperFillManager;
import net.sf.jasperreports.engine.JasperPrint;
import net.sf.jasperreports.engine.JasperRunManager;
import net.sf.jasperreports.engine.data.JRBeanCollectionDataSource;
import net.sf.jasperreports.engine.export.JRPdfExporter;
import net.sf.jasperreports.export.SimpleExporterInput;
import net.sf.jasperreports.export.SimpleOutputStreamExporterOutput;

@Slf4j
public class PrintHelper {

	/**
	 *jdbc数据源打印(模版SQL数据源方式)
	 * 
	 * @param req
	 * @param res
	 * @param params
	 * @param fileName
	 * @param connection
	 * @throws IOException
	 * @throws JRException
	 */
	public static void printPdf(HttpServletRequest req, HttpServletResponse res, Map<String, Object> params,
			String fileName, Connection connection) throws IOException, JRException {
		log.info("开始打印");
		req.setCharacterEncoding("utf-8");
		res.setCharacterEncoding("utf-8");
		res.setContentType("text/html");
		res.setContentType("application/pdf");
		ClassPathResource resource = new ClassPathResource(PrintConstants.JASPER_BASE_PATH + fileName);
		if (!resource.exists()) {
			throw new BaseException("模板-" + fileName + "不存在！");
		}
		InputStream is = resource.getInputStream();
		ServletOutputStream out = res.getOutputStream();
		JasperRunManager.runReportToPdfStream(is, out, params, connection);
		out.flush();
		out.close();
	}
	
	/**
	 * javabean数据源打印
	 * 
	 * @param req
	 * @param res
	 * @param params
	 * @param fileName
	 * @param jRBeanCollectionDataSource
	 * @throws IOException
	 * @throws JRException
	 */
	public static void printPdfByJavaBean(HttpServletRequest req, HttpServletResponse res, Map<String, Object> params,
			String fileName, JRBeanCollectionDataSource jRBeanCollectionDataSource) throws IOException, JRException {
		log.info("开始打印");
		req.setCharacterEncoding("utf-8");
		res.setCharacterEncoding("utf-8");
		res.setContentType("text/html");
		res.setContentType("application/pdf");
		ClassPathResource resource = new ClassPathResource(PrintConstants.JASPER_BASE_PATH + fileName);
		if (!resource.exists()) {
			throw new BaseException("模板-" + fileName + "不存在！");
		}
		InputStream is = resource.getInputStream();
		ServletOutputStream out = res.getOutputStream();
		JasperRunManager.runReportToPdfStream(is, out, params, jRBeanCollectionDataSource);
		out.flush();
		out.close();
		log.info("打印结束");
	}
	
	
	/**
	 * javabean数据源打印(批量)
	 * 
	 * @param req
	 * @param res
	 * @param params
	 * @param fileName
	 * @param jRBeanCollectionDataSource
	 * @throws IOException
	 * @throws JRException
	 */
	public static void printBatchPdfByJavaBean(HttpServletRequest req, HttpServletResponse res, 
			List<JasperPrint> jasperPrintList) throws IOException, JRException {
		log.info("开始打印");
		ByteArrayOutputStream baos = new ByteArrayOutputStream();
        JRPdfExporter exporter = new JRPdfExporter();
        exporter.setExporterInput(SimpleExporterInput.getInstance(jasperPrintList));
        exporter.setExporterOutput(new SimpleOutputStreamExporterOutput(baos));
        exporter.exportReport();
        byte[] bytes = baos.toByteArray();
        // 写出文件的类型
        res.setContentType("application/pdf;charset=UTF-8");
        baos.close();
		 ServletOutputStream out = res.getOutputStream();
        out.write(bytes);
        out.flush();              		
		log.info("打印结束");
	}
	
	
	
	public static JasperPrint getJasperPrint(List<?> obj, String fileName, Map<String, Object> params) throws FileNotFoundException {
	        JasperPrint jasperPrint = null;
	        JRDataSource source = null;
	        String url =ClassUtils.getDefaultClassLoader().getResource("").getPath()+PrintConstants.JASPER_BASE_PATH + fileName;
	        try {
	          source = new JRBeanCollectionDataSource(obj);
	          jasperPrint = JasperFillManager.fillReport(url, params, source);
	          return jasperPrint;
	        } catch (JRException e) {
	          log.error("打印异常 {}",e);
	        }
	        return null;
	      }
}

```

### printController实现

```java
package com.*****.ywsp.controller;

import java.io.IOException;
import java.sql.SQLException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.lang3.StringUtils;
import org.joda.time.DateTime;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.google.common.collect.Lists;
import com.google.common.collect.Maps;
import com.*****.ywsp.domain.dto.zjsb.FundcsDetail;
import com.*****.ywsp.domain.entity.sys.CurrentUserInfo;
import com.*****.ywsp.mapper.zjsb.FundcsDetailMapper;
import com.*****.ywsp.util.ContextUtil;
import com.*****.ywsp.util.PrintHelper;

import net.sf.jasperreports.engine.JRException;
import net.sf.jasperreports.engine.JasperPrint;


@RestController
@RequestMapping("/zjsb/print")
public class printController {	

	@Autowired
	private FundcsDetailMapper fundcsDetailMapper;
	
	@Autowired
	private ContextUtil contextUtil;
    
    /**
	 * 打印（模版SQL数据源方式）
	 * 
	 * @param data
	 * @throws JRException
	 * @throws IOException
	 * @throws SQLException
	 */
	@PostMapping("/printPDF")
	public void printPDF(@RequestBody List<AccAccountApply> data, HttpServletRequest req, HttpServletResponse res) throws IOException, JRException, SQLException {
		/** 参数前台单选传入一条数据的apply_id */
		Map<String, Object> params = Maps.newHashMap();
		String fileName = "accountmanager.jasper";
		if (data != null && !data.isEmpty()) {
			//renderPrintData(data);
			//不管选多少条，取第一条查询
			AccAccountApply accAccountApply = data.get(0);
			String applyId = accAccountApply.getApplyId();
			params.put("apply_id", applyId);
		}
		PrintHelper.printPdf(req, res, params, fileName, dataSource.getConnection());
	}
	
    /**
     * 资金拨款申请书-打印(支持批量) javaBean实现批量打印
     * 参考 https://blog.csdn.net/dhj199181/article/details/83112142
     * @param data
     * @throws JRException
     * @throws IOException
     * @throws SQLException
     */
    @GetMapping("/printPDF")
    public void batchPrintPDF(/*@RequestBody List<String> ids, */HttpServletRequest req, HttpServletResponse res) throws IOException, JRException, SQLException {	    	   	
    	CurrentUserInfo currentUserInfo = contextUtil.getCurrentUserInfo();    	
  		String rgCode = currentUserInfo.getRgCode();
    	String setYear = currentUserInfo.getSetYear();
    	String rgCode = currentUserInfo.getRgCode();
    	String setYear = currentUserInfo.getSetYear();
    	JasperPrint jasperPrint = null;
        List<JasperPrint> jasperPrintList = Lists.newArrayList();      
        //根据所选主单循环
    	for(String id : ids) {	
    		List<FundcsDetail> fdLs = fundcsDetailMapper.selectList(new QueryWrapper<FundcsDetail>()
    				.eq("bill_id", id).eq("rg_code", rgCode).eq("set_year", setYear));
    		/*查询明细封装数据*/
    		String fileName = "ywsp-zjsbsqs.jasper";
    		Map<String, Object> params = Maps.newHashMap();
    		List<FundcsDetail> data = Lists.newArrayList();
    		if(fdLs != null && fdLs.size() > 0) {
    			FundcsDetail fd = fdLs.get(0);
    			Date date = fd.getCreateTime();
    			String yyyyMMdd = DateTime.now().toString("yyyy年MM月dd日");
    			if(date != null) {
    				SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日");
    				yyyyMMdd = sdf.format(date);
    			}  			
    			params.put("enName", fd.getEnName());
    			params.put("mbName", fd.getMbName());
    			params.put("billNo", fd.getBillCode());
    			//TODO 待实现
    			params.put("applyTotalMoney", "12344.00");
    			params.put("budgetTotalMoney", "4243.09");
    			params.put("budgetTotalAviMoney", "21123422");
    			params.put("yyyyMMdd", yyyyMMdd);    
    			String str = "-";
    			for(FundcsDetail fdd : fdLs) {
    				FundcsDetail tmp = new FundcsDetail();
    				tmp.setBlName(StringUtils.join(fdd.getBlCode()).concat(str).concat(fdd.getBlName()));
    				tmp.setBsName(StringUtils.join(fdd.getBsCode()).concat(str).concat(fdd.getBsName()));
    				tmp.setBisName(StringUtils.join(fdd.getBisCode()).concat(str).concat(fdd.getBisName()));
    				tmp.setGovBsiName(StringUtils.join(fdd.getGovBsiCode()).concat(str).concat(fdd.getGovBsiName()));
    				tmp.setBsiName(StringUtils.join(fdd.getBsiCode()).concat(str).concat(fdd.getBsiName()));
    				tmp.setBlName(StringUtils.join(fdd.getBlCode()).concat(str).concat(fdd.getBlName()));
    				tmp.setMkName(StringUtils.join(fdd.getMkCode()).concat(str).concat(fdd.getMkName()));
    				tmp.setApplyMoney(fdd.getApplyMoney());
    				tmp.setBudgetMoney(fdd.getBudgetMoney());
    				tmp.setBudgetAviMoney(fdd.getBudgetAviMoney());
    				tmp.setPkName(StringUtils.join(fdd.getPkCode()).concat(str).concat(fdd.getPkName()));
    				tmp.setSmName(StringUtils.join(fdd.getSmCode()).concat(str).concat(fdd.getSmName()));
    				data.add(tmp);
    			}
    			int size = fdLs.size();
    			int lastData = size%15;
                //根据模版实现特定算法，补全detail区区底部区域的中间空白
    			if(lastData<=8) {
    				for(int i = 0;i<(9-lastData);i++) {
    					FundcsDetail tmp = new FundcsDetail();
    					data.add(tmp);
    				}
    			}else {
    				for(int i = 0;i<(24-lastData);i++) {
    					FundcsDetail tmp = new FundcsDetail();
    					data.add(tmp);
    				}        	
    			}   			
    		} 	
    		jasperPrint = PrintHelper.getJasperPrint(data,fileName,params);		
    		jasperPrintList.add(jasperPrint);
    	}   	
    	PrintHelper.printBatchPdfByJavaBean(req, res, jasperPrintList);  	
    }      
}

```

#### 打印模版文件，均用ireport5制作

[accountmanager.jasper](https://github.com/Albatronhenry/UploadFile/blob/master/pic/accountmanager.jasper)

[ywsp-zjsbsqs.jasper](https://github.com/Albatronhenry/UploadFile/blob/master/pic/ywsp-zjsbsqs.jasper)

#### 实现效果：

![123](https://github.com/Albatronhenry/UploadFile/blob/master/pic/20200403203338.png)



### 进阶（ireport使用补充）

------

#### ireport常用设置

- > 1.新建datasource(数据库连接方式)
  > 	1.1如果没有相应数据库的驱动（红色）
  > 		工具--选项--classpath--add jar--选择对应jar包导入即可
  > 2.文件--新建--blank A4
  > 3.自动换行
  > 	在可能换行的字段属性面板（右侧）勾选 Stretch With Overflow
  > 	将可能换行的字段的所在行全选，在属性面板里Stretch Type设置为Relative To Tallest Object，
  > 	Print Repeated Values勾选，Print When Detail Overflows 勾选。

数据源选择javabean，代码中创建相应javabean打印

> 报错：https://www.iteye.com/blog/layznet-502832
> Ireport+jasperreports+OpenReports，在用OpenReports产生报表时出现异常ERROR ReportRunAction -net.sf.jasperreports.engine.JRRuntimeException:could not load the following font:
> pdfFontName:Helvetica
> pdfEncoding:UniGB-UCS2-H
> 解决方法
> 1）将iTextAsian.jar和iTextAsianCmaps.jar置于l项目lib中
> 2）将模板设计中的文本框的属性中，在font栏中做如下设置：
>    Font Name: 宋体（或其他如楷体）
>    Pdf font name:STSong-Light
>    Pdf encoding: UniGB-UCS2-H
>    Pdf Embeded: 打勾
> 做以上设置后就OK了

网上说把要显示中文的文本框做如上设置，我的例子中只显示数字的文本框就没去做如上设置，结果还是生成pdf时还是出现异常，把这个文本框做如上设置后就OK了。疑惑的就是为什么我的显示数字的文本框不按中文的设置就出问题呢...

> 附：iTextAsianCmaps.jar下载
> http://prdownloads.sourceforge.net/itextpdf/iTextAsianCmaps.jar

