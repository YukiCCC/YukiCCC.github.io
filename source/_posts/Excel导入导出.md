---
title: Excel导入导出
date: 2023-09-14 11:30:44
tags: Spring
category: Java
---

## Excel导入导出

### 1. 简介

#### 1.1 实现方式

1. Apache POI

    POI（Poor Obfuscation Implementation）是Apache提供的操作ms office文档的API，主要针对excel进行操作。

    官网 https://poi.apache.org Apache维护

2. EasyPOI

    easypoi功能如同名字easy，主打的功能就是容易，让一个没见接触过poi的人员就可以方便的写出Excel导出、Excel导入。

    通过简单的注解和模板语言(熟悉的表达式语法)，完成以前复杂的写法。

    官网 http://doc.wupaas.com/docs/easypoi 个人维护

3. EasyExcel

    EasyExcel是一个基于Java的、快速、简洁、解决大文件内存溢出的Excel处理工具。
    他能让你在不用考虑性能、内存的等因素的情况下，快速完成Excel的读、写等功能。

    官网 https://easyexcel.opensource.alibaba.com 阿里维护

#### 1.2 Excel简介

excel的基础元素：

- 工作簿 workbook
    就是一个文件

- 工作表 sheet
    属于工作簿

- 行 row
    属于工作表

- 单元格 cell

    属于行，如C2，表示第二行第三列

### 2. 用法

#### 2.1 Apache POI

步骤：

1. 添加依赖

    ```xml
    <!--poi-->
    <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi-ooxml</artifactId>
      <version>5.2.3</version>
    </dependency>
    ```

2. 基本用法

    ```java
    @Test
    public void write() throws Exception {
    	// 1、创建工作簿
    	Workbook workbook = new HSSFWorkbook();
    	// 2、创建工作表
    	Sheet sheet = workbook.createSheet("wanho");// 指定工作表名
    	// 3、创建行
    	Row row = sheet.createRow(2); // 创建第3行，索引从0开始
    	// 4、创建单元格
    	Cell cell = row.createCell(2); // 创建第3行第3列
    	cell.setCellValue("南京万和"); // 设置单元格内容
    	// 5.输出到硬盘
    	workbook.write(new FileOutputStream("/Users/txy/Desktop/test.xls"));
    	System.out.println("写入成功！");
    	workbook.close();
    }
    
    @Test
    public void read() throws Exception {
    	// 1、读取工作簿
    	Workbook workbook = new HSSFWorkbook(new FileInputStream("/Users/txy/Desktop/test.xls"));
    	// 2、读取工作表
    	Sheet sheet = workbook.getSheet("wanho"); // 根据名称读取工作表
    	// Sheet sheet = workbook.getSheetAt(0); //根据索引读取工作表，索引从0开始
    	// 3、读取行
    	Row row = sheet.getRow(2); // 读取第3行
    	// 4、读取单元格
    	Cell cell = row.getCell(2); // 读取第3行第3列
    	System.out.println("第3行第3列内容为：" + cell.getStringCellValue());
    	workbook.close();
    }
    ```

#### 2.2 EasyPOI

步骤：

1. 添加依赖

    ```xml
    <!--easypoi-->
    <dependency>
      <groupId>cn.afterturn</groupId>
      <artifactId>easypoi-spring-boot-starter</artifactId>
      <version>4.4.0</version>
    </dependency>
    ```

2. 基本用法

    @Excel 标注在属性上，对Excel列进行描述

    工具类：

    - ExcelExportUtil  导出Excel
    - ExcelImportUtil 导入Excel

#### 2.3 EasyExcel

步骤：

1. 添加依赖

    ```xml
    <!--easyexcel-->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>easyexcel</artifactId>
      <version>3.3.2</version>
    </dependency>
    ```

2. 基本用法

    常用注解

    - @ExcelProperty 标注在属性上，对Excel列进行描述
    - @ColumnWidth  标注在类和属性上，设置列宽
    - @ContentStyle  标注在类上，设置内容样式
    - @HeadFontStyle  标注在类上，设置表头样式
    - @DateTimeFormat  标注在属性上，设置日期的格式
    - .....

    工具类：

    - EasyExcel.write()  导出Excel

    - EasyExcel.read() 导入Excel