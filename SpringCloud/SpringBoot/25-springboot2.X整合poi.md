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
