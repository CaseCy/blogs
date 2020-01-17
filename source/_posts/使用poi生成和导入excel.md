---
title: " 使用poi生成和导入Excel\t\t"
tags:
  - 杂
url: 125.html
id: 125
comments: false
categories:
  - 杂
date: 2018-10-19 09:48:46
---
参考：[POI通用导出Excel](http://blog.csdn.net/houxuehan/article/details/50960259)
***
## POI的导出Excel的方式和性能
POI操作EXCEL对象 
HSSF：操作Excel 97(.xls)格式 
XSSF：操作Excel 2007 OOXML (.xlsx)格式，操作EXCEL内存占用高于HSSF 
SXSSF:从POI3.8 beta3开始支持，基于XSSF，低内存占用。

使用POI的HSSF对象，生成Excel 97(.xls)格式，生成的EXCEL不经过压缩直接导出。 
线上问题：负载服务器转发请求到应用服务器阻塞,以及内存溢出 。 
如果系统存在大数据量报表导出，则考虑使用POI的SXSSF进行EXCEL操作。

HSSF生成的Excel 97(.xls)格式本身就有每个sheet页不能超过65536条的限制。 
XSSF生成Excel 2007 OOXML (.xlsx)格式，条数增加了，但是导出过程中，内存占用率却高于HSSF. 
SXSSF是自3.8-beta3版本后，基于XSSF提供的低内存占用的操作EXCEL对象。其原理是可以设置或者手动将内存中的EXCEL行写到硬盘中，这样内存中只保存了少量的EXCEL行进行操作。

EXCEL的压缩率特别高，能达到80%，12M的文件压缩后才2M左右。 如果未经过压缩、不仅会占用用户带宽，且会导致负载服务器（apache）和应用服务器之间，长时间占用连接(二进制流转发)，导致负载服务器请求阻塞，不能提供服务。

一定要注意文件流的关闭

防止前台（页面）连续触发导出EXCEL
***
## 代码实现
```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.streaming.SXSSFWorkbook;
import org.apache.poi.xssf.usermodel.XSSFRichTextString;
------------------------------------------------------------------------------------
Workbook workbook = new SXSSFWorkbook(-1); //参数为大小，-1时为不限制，不写默认是120
Sheet sheet = workbook.createSheet(fileName); //filname为sheet的名字
Row row = sheet.createRow(0); //创建第0行
Cell cell = row.createCell(0); //在第0行中创建第0列,确定一个单元格
cell.setCellValue("单元格的值");//给上面创建的单元格设置值
---------------------------------------------------------------------------------
sheet.setColumnWidth(j,length);//设置列的宽度，j为第几列，length为长度，中文一般是用列标题的长度.getBytes().length*256
sheet.autoSizeColumn(int i,boolean a); //自动设置列宽度。这个对中文的支持不是很好
---------------------------------------------------------------------------------
//将生成的Excel通过response返回，这里一定要记得关闭workbook这个流资源
OutputStream os = response.getOutputStream();
            response.setContentType("application/msexcel");
            response.setHeader("Content-Disposition", "attachment;"
                    + " filename="
                    + new String(fileName.getBytes(), "ISO-8859-1")+".xls");
            workbook.write(os);
        } catch (IOException e)
```
自动调整列宽度，中文（仅供参考）
```java
    //自动调整列宽度
    private static void autoSetColumWidth(Sheet sheet){
        //获取第1列和第0列比较
        Row row1 = sheet.getRow(sheet.getFirstRowNum()+1);
        for (int j = 0; j < sheet.getRow(sheet.getFirstRowNum()).getLastCellNum(); j++) {
            Cell cell = row1.getCell(j);
            String stringCellValue = cell.getStringCellValue();
            if(sheet.getColumnWidth(j)<stringCellValue.getBytes().length*SIZE_VALUE){
                sheet.setColumnWidth(j,stringCellValue.getBytes().length*SIZE_VALUE);
            }
        }
    }
-----------------------------------------------------------------------------
    //自动调整列宽度
    private static void autoSetColumWidth(Sheet sheet,int[] len){
        for (int i = 0; i < len.length; i++) {
            sheet.setColumnWidth(i,len[i]);
        }
    }

```

## POI设置样式
参考：[POI设置Excel样式](http://blog.csdn.net/npp616/article/details/8546737)
　　　[POI设置Excel单元格样式](http://blog.csdn.net/spp_1987/article/details/13769043)
```java
CellStyle cellStyle = workbook.createCellStyle();
Font font = workbook.createFont();
font.setBold(true);
font.setFontName("黑体");
font.setFontHeightInPoints((short)11);  //设置字体大小
cellStyle.setFont(font);

CellStyle bodyStyle = workbook.createCellStyle();
bodyStyle.setBorderBottom(BorderStyle.THIN);
bodyStyle.setBorderLeft(BorderStyle.THIN);
bodyStyle.setBorderRight(BorderStyle.THIN);
bodyStyle.setBorderTop(BorderStyle.THIN);
```
## 使用POI导入Excel
参考：[Java+poi Excel导入](http://blog.csdn.net/lp1791803611/article/details/52351333)
　　　[使用POI读取Excel文件](http://blog.csdn.net/slience_perseverance/article/details/8228157)