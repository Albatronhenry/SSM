### springboot2.X整合poi

[项目需求，需要导出excel文档]()

#### pom依赖：
```xml
  <!-- poi -->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>3.17</version>
        </dependency>

        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>3.17</version>
        </dependency>

```
#### controller类：
```java
@PostMapping("/exportExcel")
	public void exportExcel(@RequestBody Map<String,Object> map, HttpServletResponse response){	
				String user_code = contextUtils.getContextInfo().getUsercode();
				String rg_code = contextUtils.getContextInfo().getRg_code();
				String set_year = contextUtils.getContextInfo().getSet_year();
				Map methodsmap = iMethodsService.commonCall(map,user_code,rg_code,set_year);										
				//组装导出文件
				XSSFWorkbook wb = null;
				OutputStream os = null;
				try {
					// excel 构建
					wb = new XSSFWorkbook();
					XSSFSheet sheet = wb.createSheet("excel导出");
					// 表头列
					XSSFRow row = sheet.createRow(0);
					List cellLs = (List) JSONArray.parseArray((String) methodsmap.get("data"));
					if (cellLs != null && cellLs.size() > 0) {
						for (int i = 0, l = cellLs.size(); i < l; i++) {
							Map<String, String> dataCell = (Map) cellLs.get(i);
							// 数据列
							XSSFRow _row = sheet.createRow(i + 1);
							int j = 0;
							for (Map.Entry<String, String> entry : dataCell.entrySet()) {
								if (i == 0) {
									String mapKey = entry.getKey();
									// 拼装表头列
									row.createCell(j).setCellValue(mapKey);
								}
								String mapValue = entry.getValue();
								_row.createCell(j).setCellValue(mapValue);
								j++;
							}
						}
						// 设置Header并且输出文件
						String fileName = "数据交互客户端excel导出";
						response.setHeader("Content-Disposition","attachment; filename=" + new String((fileName + ".xlxs").getBytes(), "iso-8859-1"));
						response.setContentType("application/vnd.ms-excel");
						response.setCharacterEncoding("UTF-8");
						os = response.getOutputStream();
						wb.write(os);
						os.flush();
					} 
				}catch (Exception e) {
					e.printStackTrace();
				} finally {
					try {
						wb.close();
						os.close();
					} catch (IOException e) {
					}
				}				
	}

```

#### vue前端：
```js
//Excel导出
  if( vm.entity.type === "1") {
      downloadExcel("/methods/exportExcel", "数据交互客户端excel导出.xlsx",  vm.entity);
      vm.reset();
      vm.getMethodParam=false;
 }


 import {isEmpty,downloadExcel } from "../utils/checkUtils";

/**
 * 下载Excel
 * @param {下载路径} url
 * @param {下载文件名} fileName
 * @param {参数} param 对象，后端用@RequestBody接收
 */
export const  downloadExcel = (url, fileName, param)  => {
    if(!url) {
        return;
    }
    let axios = require('axios');
    axios({
        method: "post",
        url: url,
        data : param,
        responseType: "blob",
        headers: {
            "Content-Type": "application/json"
        }
    }).then(res => {
        //new Blob([res])中不加data就会返回下图中[objece objece]内容（少取一层）
        const blob = new Blob([res.data]);
        const elink = document.createElement("a");
        elink.download = fileName;
        elink.style.display = "none";
        elink.href = URL.createObjectURL(blob);
        document.body.appendChild(elink);
        elink.click();
        URL.revokeObjectURL(elink.href);
        document.body.removeChild(elink);
    }).catch(error => {
        let vm = this;
        vm.$message.error( "文件下载出错");
        console.log("文件下载出错", error);
    });
}

```

### 进阶（主要知识点：合并单元格并保存边线）

#### controller类：

```java
import com.yonyougov.report.util.CellStyleUtils;
import com.yonyougov.report.util.NumberUtils;

import org.apache.poi.ss.usermodel.BorderStyle;
import org.apache.poi.ss.util.CellRangeAddress;
import org.apache.poi.xssf.usermodel.XSSFCell;
import org.apache.poi.xssf.usermodel.XSSFCellStyle;
import org.apache.poi.xssf.usermodel.XSSFRow;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
/**
	 * 导出Excel
	 * 
	 * @param form
	 * @param request
	 * @param response
	 */
	@PostMapping("/exportData")
	public void exportData(@RequestBody JzzfYszxForm form, HttpServletRequest request, HttpServletResponse response) {
		XSSFWorkbook wb = null;
		OutputStream os = null;
		try {
			/* 创建文件 */
			wb = new XSSFWorkbook();
			XSSFSheet sheet = wb.createSheet("*******执行情况表");
			XSSFRow row0 = sheet.createRow(0);
			XSSFRow row1 = sheet.createRow(1);
			XSSFRow row2 = sheet.createRow(2);		
			/** 默认数据开始列 */
			int startCellNum = 0;
			List<Map<String, Object>> headers = form.getExcelHeaders();
			List<String> headerKeys = Lists.newArrayList();
			if (headers != null && !headers.isEmpty()) {
				startCellNum = headers.size();
				//设置动态表头风格
				XSSFCellStyle xssfDynamicHeaderCellStyle = CellStyleUtils.dynamicHeaderCellStyle(wb);
				for (int i = 0; i < headers.size(); i++) {
					Map<String, Object> header = headers.get(i);
					String key = (String) header.get("key");// en
					String label = (String) header.get("label");//**单位
					XSSFCell celli =  row0.createCell(i);				
					celli.setCellStyle(xssfDynamicHeaderCellStyle);
					celli.setCellValue(label);
					CellRangeAddress region = new CellRangeAddress(0, 2, i, i);
					sheet.addMergedRegion(region);// 1：开始行 2：结束行 3：开始列 4：结束列
					CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
					headerKeys.add(key);
				}
			}
			
			//设置表头风格
			XSSFCellStyle xssfHeaderCellStyle = CellStyleUtils.fixHeaderCellStyle(wb);
			XSSFCell cell = row0.createCell(startCellNum);
			cell.setCellStyle(xssfHeaderCellStyle);
			cell.setCellValue("预算指标");
			CellRangeAddress region = new CellRangeAddress(0, 0, startCellNum, startCellNum + 2);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			XSSFCell cell1 = row0.createCell(startCellNum + 3);
			cell1.setCellStyle(xssfHeaderCellStyle);
			cell1.setCellValue("用款计划");
			region = new CellRangeAddress(0, 0, startCellNum + 3, startCellNum + 11);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);		
			
			XSSFCell cell2 = row0.createCell(startCellNum + 12);
			cell2.setCellStyle(xssfHeaderCellStyle);
			cell2.setCellValue("支出");
			region = new CellRangeAddress(0, 0, startCellNum + 12, startCellNum + 14);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);		
			
			XSSFCell cell3 = row0.createCell(startCellNum + 15);
			cell3.setCellStyle(xssfHeaderCellStyle);
			cell3.setCellValue("预算结余");
			region = new CellRangeAddress(0, 0, startCellNum + 15, startCellNum + 17);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);		
			
			XSSFCell cell4 = row0.createCell(startCellNum + 18);
			cell4.setCellStyle(xssfHeaderCellStyle);
			cell4.setCellValue("执行进度");
			region = new CellRangeAddress(0, 0, startCellNum + 18, startCellNum + 20);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			// 增加此部分代码，将合并的，没有填值的也设置边框样式	
			for (int j = 0,preColNum = headers.size()+21; j < preColNum; j++) {
				sheet.setColumnWidth(j, 20*256);
			}
			
			XSSFCell cell5 = row1.createCell(startCellNum);
			cell5.setCellStyle(xssfHeaderCellStyle);
			cell5.setCellValue("批复指标");
			region = new CellRangeAddress(1, 2, startCellNum, startCellNum);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			XSSFCell cell6 = row1.createCell(startCellNum +1);
			cell6.setCellStyle(xssfHeaderCellStyle);
			cell6.setCellValue("已用指标");
			region = new CellRangeAddress(1, 2, startCellNum + 1, startCellNum + 1);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			XSSFCell cell7 = row1.createCell(startCellNum +2);
			cell7.setCellStyle(xssfHeaderCellStyle);
			cell7.setCellValue("剩余指标");
			region = new CellRangeAddress(1, 2, startCellNum + 2, startCellNum + 2);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			XSSFCell cell8 = row1.createCell(startCellNum +3);
			cell8.setCellStyle(xssfHeaderCellStyle);
			cell8.setCellValue("已审批计划");
			region = new CellRangeAddress(1, 1, startCellNum + 3, startCellNum + 5);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			XSSFCell cell9 = row1.createCell(startCellNum +6);
			cell9.setCellStyle(xssfHeaderCellStyle);
			cell9.setCellValue("已支用计划");
			region = new CellRangeAddress(1, 1, startCellNum + 6, startCellNum + 8);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			XSSFCell cell10 = row1.createCell(startCellNum +9);
			cell10.setCellStyle(xssfHeaderCellStyle);
			cell10.setCellValue("结余计划");
			region = new CellRangeAddress(1, 1, startCellNum + 9, startCellNum + 11);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			XSSFCell cell11 = row1.createCell(startCellNum +12);
			cell11.setCellStyle(xssfHeaderCellStyle);
			cell11.setCellValue("合计");
			region = new CellRangeAddress(1, 2, startCellNum + 12, startCellNum + 12);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			XSSFCell cell12 = row1.createCell(startCellNum +13);
			cell12.setCellStyle(xssfHeaderCellStyle);
			cell12.setCellValue("直接支付");
			region = new CellRangeAddress(1, 2, startCellNum + 13, startCellNum + 13);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			XSSFCell cell13 = row1.createCell(startCellNum +14);
			cell13.setCellStyle(xssfHeaderCellStyle);
			cell13.setCellValue("授权支付");
			region = new CellRangeAddress(1, 2, startCellNum + 14, startCellNum + 14);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			XSSFCell cell14 = row1.createCell(startCellNum +15);
			cell14.setCellStyle(xssfHeaderCellStyle);
			cell14.setCellValue("合计");
			region = new CellRangeAddress(1, 2, startCellNum + 15, startCellNum + 15);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			XSSFCell cell15 = row1.createCell(startCellNum +16);
			cell15.setCellStyle(xssfHeaderCellStyle);
			cell15.setCellValue("指标结余");
			region = new CellRangeAddress(1, 2, startCellNum + 16, startCellNum + 16);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			XSSFCell cell16 = row1.createCell(startCellNum +17);
			cell16.setCellStyle(xssfHeaderCellStyle);
			cell16.setCellValue("计划结余");
			region = new CellRangeAddress(1, 2, startCellNum + 17, startCellNum + 17);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			XSSFCell cell17 = row1.createCell(startCellNum +18);
			cell17.setCellStyle(xssfHeaderCellStyle);
			cell17.setCellValue("计划占指标");
			region = new CellRangeAddress(1, 2, startCellNum + 18, startCellNum + 18);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			XSSFCell cell18 = row1.createCell(startCellNum +19);
			cell18.setCellStyle(xssfHeaderCellStyle);
			cell18.setCellValue("支出占计划");
			region = new CellRangeAddress(1, 2, startCellNum + 19, startCellNum + 19);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			XSSFCell cell19 = row1.createCell(startCellNum +20);
			cell19.setCellStyle(xssfHeaderCellStyle);
			cell19.setCellValue("支付占用指标");
			region = new CellRangeAddress(1, 2, startCellNum + 20, startCellNum + 20);
			sheet.addMergedRegion(region);
			CellStyleUtils.setFixBorderStyle(BorderStyle.THIN, region, sheet);	
			
			
			XSSFCell cell20 = row2.createCell(startCellNum +3);
			cell20.setCellStyle(xssfHeaderCellStyle);
			cell20.setCellValue("合计");
			XSSFCell cell21 = row2.createCell(startCellNum +4);
			cell21.setCellStyle(xssfHeaderCellStyle);
			cell21.setCellValue("直接支付");
			XSSFCell cell22 = row2.createCell(startCellNum +5);
			cell22.setCellStyle(xssfHeaderCellStyle);
			cell22.setCellValue("授权支付");
			XSSFCell cell23 = row2.createCell(startCellNum +6);
			cell23.setCellStyle(xssfHeaderCellStyle);
			cell23.setCellValue("合计");
			XSSFCell cell24 = row2.createCell(startCellNum +7);
			cell24.setCellStyle(xssfHeaderCellStyle);
			cell24.setCellValue("财政直接支付");
			XSSFCell cell25 = row2.createCell(startCellNum +8);
			cell25.setCellStyle(xssfHeaderCellStyle);
			cell25.setCellValue("财政授权支付");
			XSSFCell cell26 = row2.createCell(startCellNum +9);
			cell26.setCellStyle(xssfHeaderCellStyle);
			cell26.setCellValue("合计");
			XSSFCell cell27 = row2.createCell(startCellNum +10);
			cell27.setCellStyle(xssfHeaderCellStyle);
			cell27.setCellValue("财政直接支付");
			XSSFCell cell28 = row2.createCell(startCellNum +11);
			cell28.setCellStyle(xssfHeaderCellStyle);
			cell28.setCellValue("财政授权支付");

			this.setFormData(form);
			List<Map<String, Object>> datas = jzzfYszxServiceI.loadYSZXData(form);
			for (int i = 0; i < datas.size(); i++) {
				Map<String, Object> data = datas.get(i);
				XSSFRow rowi = sheet.createRow(i + 3);
				//设置表体风格
				XSSFCellStyle xssfDataCellStyle = CellStyleUtils.dataCellStyle(wb, i);
				for (int j = 0; j < headerKeys.size(); j++) {
					String key = headerKeys.get(j);
					String code = key.toUpperCase() + "_CODE";
					String value = (String) data.get(code);
					XSSFCell cellj = rowi.createCell(j);
					cellj.setCellStyle(xssfDataCellStyle);
					cellj.setCellValue(value);
				}
				XSSFCell cell30 = rowi.createCell(startCellNum);
				cell30.setCellStyle(xssfDataCellStyle);
				cell30.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("BUDGET_MONEY"))));
				XSSFCell cell31 = rowi.createCell(startCellNum + 1);
				cell31.setCellStyle(xssfDataCellStyle);
				cell31.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("BUDGET_USE_MONEY"))));
				XSSFCell cell32 = rowi.createCell(startCellNum + 2);
				cell32.setCellStyle(xssfDataCellStyle);
				cell32.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("BUDGET_CANUSE_MONEY"))));
				XSSFCell cell33 = rowi.createCell(startCellNum + 3);
				cell33.setCellStyle(xssfDataCellStyle);
				cell33.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("PLANED_SUM_MONEY"))));
				XSSFCell cell34 = rowi.createCell(startCellNum + 4);
				cell34.setCellStyle(xssfDataCellStyle);
				cell34.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("PLANED_DIR_MONEY"))));
				XSSFCell cell35 = rowi.createCell(startCellNum + 5);
				cell35.setCellStyle(xssfDataCellStyle);
				cell35.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("PLANED_ACC_MONEY"))));
				XSSFCell cell36 = rowi.createCell(startCellNum + 6);
				cell36.setCellStyle(xssfDataCellStyle);
				cell36.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("PLANUSED_SUM_MONEY"))));
				XSSFCell cell37 = rowi.createCell(startCellNum + 7);
				cell37.setCellStyle(xssfDataCellStyle);
				cell37.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("PLANUSED_DIR_MONEY"))));
				XSSFCell cell38 = rowi.createCell(startCellNum + 8);
				cell38.setCellStyle(xssfDataCellStyle);
				cell38.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("PLANUSED_ACC_MONEY"))));
				XSSFCell cell39 = rowi.createCell(startCellNum + 9);
				cell39.setCellStyle(xssfDataCellStyle);
				cell39.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("PLANREMAIN_SUM_MONEY"))));
				XSSFCell cell40 = rowi.createCell(startCellNum + 10);
				cell40.setCellStyle(xssfDataCellStyle);
				cell40.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("PLANREMAIN_DIR_MONEY"))));
				XSSFCell cell41 = rowi.createCell(startCellNum + 11);
				cell41.setCellStyle(xssfDataCellStyle);
				cell41.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("PLANREMAIN_ACC_MONEY"))));
				XSSFCell cell42 = rowi.createCell(startCellNum + 12);
				cell42.setCellStyle(xssfDataCellStyle);
				cell42.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("PAYED_SUM_MONEY"))));
				XSSFCell cell43 = rowi.createCell(startCellNum + 13);
				cell43.setCellStyle(xssfDataCellStyle);
				cell43.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("PAYED_DIR_MONEY"))));
				XSSFCell cell44 = rowi.createCell(startCellNum + 14);
				cell44.setCellStyle(xssfDataCellStyle);
				cell44.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("PAYED_ACC_MONEY"))));
				XSSFCell cell45 = rowi.createCell(startCellNum + 15);
				cell45.setCellStyle(xssfDataCellStyle);
				cell45.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("BUDGETBALANCE_SUM_MONEY"))));
				XSSFCell cell46 = rowi.createCell(startCellNum + 16);
				cell46.setCellStyle(xssfDataCellStyle);
				cell46.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("BUDGETBALANCE_BUDGET_MONEY"))));
				XSSFCell cell47 = rowi.createCell(startCellNum + 17);
				cell47.setCellStyle(xssfDataCellStyle);
				cell47.setCellValue(NumberUtils.parseString("",String.valueOf(data.get("BUDGETBALANCE_PLAN_SUM_MONEY"))));
				XSSFCell cell48 = rowi.createCell(startCellNum + 18);
				cell48.setCellStyle(xssfDataCellStyle);
				cell48.setCellValue(String.valueOf(data.get("BUDGET_USE_PRECENT")));
				XSSFCell cell49 = rowi.createCell(startCellNum + 19);
				cell49.setCellStyle(xssfDataCellStyle);
				cell49.setCellValue(String.valueOf(data.get("PLAN_USE_PRECENT")));
				XSSFCell cell50 = rowi.createCell(startCellNum + 20);
				cell50.setCellStyle(xssfDataCellStyle);
				cell50.setCellValue(String.valueOf(data.get("PAY_BUDGET_PRECENT")));
			}

			String fileName = "*****情况表";
			response.setHeader("Content-Disposition", "attachment; filename=" + new String((fileName + ".xlxs").getBytes(), "iso-8859-1"));
			response.setContentType("application/vnd.ms-excel");
			response.setCharacterEncoding("UTF-8");
			os = response.getOutputStream();
			wb.write(os);
			os.flush();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				wb.close();
				os.close();
			} catch (IOException e) {
			}
		}
	}
```

#### CellStyleUtils封装工具类：

```java
package com.*****.report.util;

import org.apache.poi.ss.usermodel.BorderStyle;
import org.apache.poi.ss.usermodel.FillPatternType;
import org.apache.poi.ss.usermodel.HorizontalAlignment;
import org.apache.poi.ss.usermodel.IndexedColors;
import org.apache.poi.ss.util.CellRangeAddress;
import org.apache.poi.ss.util.RegionUtil;
import org.apache.poi.xssf.usermodel.XSSFCellStyle;
import org.apache.poi.xssf.usermodel.XSSFFont;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

public class CellStyleUtils {
	//动态表头风格
		public static XSSFCellStyle dynamicHeaderCellStyle(XSSFWorkbook wb) {
			XSSFCellStyle xssfCellStyle = wb.createCellStyle();
			xssfCellStyle.setFillForegroundColor(IndexedColors.WHITE.getIndex());
			xssfCellStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);
			// 生成字体
			XSSFFont font = wb.createFont();
			xssfCellStyle.setFont(font);
			xssfCellStyle.setBorderTop(BorderStyle.THIN);
			xssfCellStyle.setBorderLeft(BorderStyle.THIN);
			xssfCellStyle.setBorderRight(BorderStyle.THIN);
			xssfCellStyle.setBorderBottom(BorderStyle.THIN);
			// 自动换行
			xssfCellStyle.setWrapText(true);
			// 水平居中
			xssfCellStyle.setAlignment(HorizontalAlignment.CENTER);
			// 垂直居中
			xssfCellStyle.setVerticalAlignment(xssfCellStyle.getVerticalAlignmentEnum().CENTER);
			return xssfCellStyle;
		}
	//固定表头风格
	public static XSSFCellStyle fixHeaderCellStyle(XSSFWorkbook wb) {
		XSSFCellStyle xssfCellStyle = wb.createCellStyle();
		xssfCellStyle.setFillForegroundColor(IndexedColors.WHITE.getIndex());// 灰色
		xssfCellStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);
		// 生成字体
		XSSFFont font = wb.createFont();
		xssfCellStyle.setFont(font);
		xssfCellStyle.setBorderTop(BorderStyle.THIN);
		xssfCellStyle.setBorderLeft(BorderStyle.THIN);
		xssfCellStyle.setBorderRight(BorderStyle.THIN);
		xssfCellStyle.setBorderBottom(BorderStyle.THIN);
		// 自动换行
		xssfCellStyle.setWrapText(true);
		// 水平居中
		xssfCellStyle.setAlignment(HorizontalAlignment.CENTER);
		// 垂直居中
		xssfCellStyle.setVerticalAlignment(xssfCellStyle.getVerticalAlignmentEnum().CENTER);
		return xssfCellStyle;
	}
	//表体风格
	public static XSSFCellStyle dataCellStyle(XSSFWorkbook wb,int index) {
		XSSFCellStyle xssfCellStyle = wb.createCellStyle();
/*		short color = 22;//白色
		if(index%2==0) {
			color = 80;//浅灰
		}*/
		xssfCellStyle.setFillForegroundColor(IndexedColors.WHITE.getIndex());
		xssfCellStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);
		// 生成字体
		XSSFFont font = wb.createFont();
		xssfCellStyle.setFont(font);
		xssfCellStyle.setBorderTop(BorderStyle.THIN);
		xssfCellStyle.setBorderLeft(BorderStyle.THIN);
		xssfCellStyle.setBorderRight(BorderStyle.THIN);
		xssfCellStyle.setBorderBottom(BorderStyle.THIN);
		// 自动换行
		xssfCellStyle.setWrapText(true);
		// 水平居中
		xssfCellStyle.setAlignment(HorizontalAlignment.LEFT);
		// 垂直居中
		xssfCellStyle.setVerticalAlignment(xssfCellStyle.getVerticalAlignmentEnum().CENTER);
		return xssfCellStyle;
	}
	/**
	* 合并单元格后出现边框有的消失的解决方法
	* @param borderStyle
	* @param region
	* @param sheet
	*/
	public static void setFixBorderStyle(BorderStyle border, CellRangeAddress region, XSSFSheet sheet){
		RegionUtil.setBorderBottom(border, region, sheet); //下边框
		RegionUtil.setBorderLeft(border, region, sheet); //左边框
		RegionUtil.setBorderRight(border, region, sheet); //右边框
		RegionUtil.setBorderTop(border, region, sheet); //上边框
	}
	
}

```

#### NumberUtils封装工具类

```java
package com.yonyougov.report.util;

import java.math.BigDecimal;
import java.text.DecimalFormat;

import org.apache.commons.lang3.StringUtils;
/**
 * 金额字符串转金额类型展示
 * 入参-->    千分位： ,###,###        123456789.3                 	 			结果-->123,456,789
 * 入参-->    千分位两位小数：   ,###,###.00     123456789.3                  	结果-->123,456,789.30
 * 入参-->    千分位两位小数：   ,###,##0.00     0                  				结果-->0.00
 * 如果传入参数格式为空，默认千分位两位小数： ,###,##0.00
 */
public class NumberUtils {
	public static String parseString(String pattern, String money) {
		if(StringUtils.isBlank(money)||"null".equals(money)) {
			return "";
		}
		BigDecimal bd = new BigDecimal(money);
		if(StringUtils.isBlank(pattern)) {
			pattern = ",###,##0.00";
		}
		DecimalFormat df = new DecimalFormat(pattern);
		return df.format(bd);
	}
}

```

数据表格样式：

![image-20200403195756898](https://github.com/Albatronhenry/UploadFile/blob/master/pic/20200403201051.png)

导出效果样式：

![image-20200403200154522](https://github.com/Albatronhenry/UploadFile/blob/master/pic/20200403201123.png)