title: jpa查询+poi写入excel
author: carl-zhu
tags: []
categories: []
date: 2021-11-04 19:28:00
---
## 目的
任务目的是利用jpa查询数据库中的数据，并利用poi库将其写入excel表中。

### poi
poiApache公司利用Java编写的免费的跨平台Java APi，能够创建和维护office openXML和OLE2复合格式文档的功能，能够对Word、Excel、PowePoint等文件进行读写操作（感兴趣可以自行检索，网上对这些的介绍比我详细多了）。
### 搭建过程
#### 创建工作薄
poi对excel的操作是按照工作薄、页、行、列的顺序进行操作的，所以在所有操作之前，要先新建一个工作薄。
在poi中有三种工作薄提供，以下是三种格式的区别，推荐使用是XSSDWorkbook
~~~
HSSFWorkbook ：2003版excel  以.xls结尾，最多只有65536（2^16）行，256（2^8）列。
XSSFWorkbook ：2007版的excel 以.xlsx结尾，最多只有1048576（2^20）行,16384（2^14）列。
SXSSFWorkbook :支持低内存占用的操作方式 以.xlsx结尾
~~~
创建一个工作薄
~~~
HSSFWorkbook wb=new HSSFWorkbook();
XSSFWorkbook wb=new XSSFWorkbook();
SXSSFWorkbook wb=new SXSSFWorkbook();
~~~

#### 在工作薄中创建页
所谓的页（不知道是不是这么称呼），就是excel左下角一页一页的那个东东。一个excel里可以有多页内容。
~~~
wb: 工作薄的名称

Sheet sheet=wb.createSheet("sheet name")
or
XSSFSheet sheet=wb.creatSheet("sheet name")

tip：第一种的方式更加通用，第二种还需要加上特定的类型。
~~~

#### 创建行、列
在poi中，sheet创建好之后就是对行列的创建，创建的方式为
~~~
Row row=sheet.createRow(i); i表示第几行
Cell cell=row.createCell(j); j表示第几列
~~~
#### 举例说明
要使用poi的方法，首先要引入相关的依赖，在pom.xml中加入如下依赖
~~~
<dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-scratchpad</artifactId>
            <version>3.5-beta6</version>
        </dependency>

        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-contrib</artifactId>
            <version>3.5-beta6</version>
        </dependency>
~~~
测试的demo，循环写入6000行，10列的数据。

~~~
public class test {
    //保存地址
    String path="D:\\blog";
    @Test
    public void testbignum() throws IOException {
        //开始时间
        long begtime=System.currentTimeMillis();

        Workbook workbook=new HSSFWorkbook();
        Sheet sheet=workbook.createSheet();
        //写入
        for (int rowNum=0;rowNum<65500;rowNum++){
            Row row=sheet.createRow(rowNum);
            for (int cellNum=0;cellNum<10;cellNum++){
                Cell cell=row.createCell(cellNum);
                cell.setCellValue(cellNum);
            }
        }
        System.out.println("ovwe");
        //创建输出流
        FileOutputStream fileOutputStream = new FileOutputStream(path + "12.xls");
        //在工作薄中写入流
        workbook.write(fileOutputStream);
        fileOutputStream.close();
        //结束时间
        long endT=System.currentTimeMillis();
        System.out.println((double)(endT-begtime)/1000);
    }
}
~~~
此时你应该可以在D/blog 文件夹下看到你新建的一个excel文件，里面是一些数字。以上是一个小的demo，演示一下写入的过程。下面是两个常见的情况。

**一**、需要你在查询数据库之后将数据导出，不需要写成响应式。
~~~
//在此，我们如此分析，其实就是两个小操作，查询+写入excel
//查询的结果是一个类的list，按照上述的方法进行写入就行

假定：
1、实体类写好
2、dao层写好，并含有List<entity class name> findAll();
3、server层写好，含有一个返回了List<entity class name> findAll()；的函数。（不一定就是叫findAll，名字随意）

~~~
 1、Entity：定义一个名为Students的实体类
 ~~~
 package com.example.demoforme.jpa.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Data
@AllArgsConstructor
@NoArgsConstructor
//表的结构，可以根据自己的表来创建
@Table(name = "students")
public class Students {

  @Id
  private long id;
  private long classId;
  private String name;
  private String gender;
  private long score;
  private String address;
  private String school;

}
 ~~~
2、dao层：新建StudentsDAO文件，编写sql语句。（个人理解，转化成sql的一层）
~~~
package com.example.demoforme.jpa.dao;

import com.example.demoforme.jpa.model.Students;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import org.springframework.lang.Nullable;
import java.util.List;

public interface StudentsDAO extends JpaRepository<Students,String>, JpaSpecificationExecutor<Students> {

    @Nullable
    List<Students> findAll();
}

~~~
3、server层：逻辑实现层，将方法实现
~~~
package com.example.demoforme.server;

import com.alibaba.fastjson.JSON;
import com.example.demoforme.jpa.dao.StudentsDAO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import com.example.demoforme.jpa.model.Students;

import java.util.ArrayList;
import java.util.List;

@Service
public class StudentsService {
    @Autowired
    private StudentsDAO StdDao;

    public List<Students> find(){
        List<Students> all = StdDao.findAll();
        return all;
    }
}
~~~
4、在server层编写一个辅助类，帮我们出来输出excel。
~~~
package com.example.demoforme.server;

import com.example.demoforme.jpa.dao.StudentsDAO;
import com.example.demoforme.jpa.model.Students;
import org.apache.poi.hssf.usermodel.HSSFCell;
import org.apache.poi.hssf.usermodel.HSSFRow;
import org.apache.poi.xssf.streaming.SXSSFWorkbook;
import org.apache.poi.xssf.usermodel.XSSFCell;
import org.apache.poi.xssf.usermodel.XSSFRow;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.swing.filechooser.FileSystemView;
import java.io.File;
import java.io.FileOutputStream;

import java.util.List;

@Service
public class ExcelWService {

    public void createExcel(List<Students> list) {
        //excel标题
        String title = "学生表";
        //excel表名
        String [] headers = {"ID","班级","姓名","性别","分数","地址", "学校"};
        //excel文件名
        //存储路径--获取桌面位置
        FileSystemView view = FileSystemView.getFileSystemView();
        File directory = view.getHomeDirectory();
        System.out.println(directory);
        //存储Excel的路径
        String path = directory+"\\"+".xlsx";
        System.out.println(path);
        try {

            //定义一个Excel表格
            XSSFWorkbook wb = new XSSFWorkbook();  //创建工作薄
            XSSFSheet sheet = wb.createSheet("sheet1"); //创建工作表
            XSSFRow row = sheet.createRow(0); //行
            XSSFCell cell;  //单元格


            for (int i = 0; i < headers.length; i++) {
                XSSFCell hssfCell = row.createCell(i);
                hssfCell.setCellValue(headers[i]);
            }
            for (int i = 0; i <list.size(); i++){
                XSSFRow row1 = sheet.createRow(i +2);
                String str=null;
                for (int j = 0; j < title.length(); j++){
                    //将内容按顺序赋给对应列对象
                    XSSFCell hssfCell = row1.createCell(j);
                    switch (j){
                        case 0:
                             str = String.valueOf(list.get(i).getId());
                             break;
                        case 1:
                            str = String.valueOf(list.get(i).getClassId());
                            break;
                        case 2:
                            str = String.valueOf(list.get(i).getName());
                            break;
                        case 3:
                            str = String.valueOf(list.get(i).getGender());
                            break;
                        case 4:
                            str = String.valueOf(list.get(i).getScore());
                            break;
                        case 5:
                            str = String.valueOf(list.get(i).getAddress());
                            break;
                        case 6:
                            str = String.valueOf(list.get(i).getSchool());
                            break;

                    }
                    hssfCell.setCellValue(str);
                }
            }

            //输出流,下载时候的位置
            FileOutputStream outputStream = new FileOutputStream(path);
            wb.write(outputStream);
            outputStream.flush();
            outputStream.close();
            System.out.println("写入成功");
        } catch (Exception e) {
            System.out.println("写入失败");
            e.printStackTrace();
        }
    }

tip:本程序也是看着网上的同学改的，太久了，记不住了，有看见的同学莫怪。介意的同学联系我，我会删除改正的。
~~~
5、controller层：写一个getmapping在网页调用。
~~~
package com.example.demoforme.controller;

import com.alibaba.fastjson.JSON;
import com.example.demoforme.jpa.model.Students;
import com.example.demoforme.server.ExcelWService;
import com.example.demoforme.server.StudentsService;
import com.example.demoforme.util.ExportExcelUtil;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.OutputStream;
import java.io.UnsupportedEncodingException;
import java.util.List;

@RestController
@RequestMapping("/demo")
public class DemoController {
    @Autowired
    private StudentsService studentsService;

    @Autowired
    private ExcelWService excelWService;

    @GetMapping("/get")
    public String getdemo() {
        List<Students> list = studentsService.find();
        excelWService.createExcel(list);
        return "success";
    }
}
~~~
运行，在网页中输入端口号+地址,可以在桌面上看到有一个新的excel文件。打开，与idea中的数据库中的表进行对比。完全一样，大功告成（标题是自己定的）。

![upload successful](/images/poi-w-6.png)

![upload successful](/images/poi-w-7.png)

**二**、上述的方法可以输出我们想要的结果,但是，有的小伙伴可能或注意到，在下载的时候，并没有像平时我们下载东西一样会有一个完成的提示。如下一样。
![upload successful](/images/poi-w-8.png)
所以，我们要借助另外一个东东，响应式编写方式（理解的不深，不敢细讲），所以直接上代码吧。
1、生成实体类，和上面的第一步一样。
2、生成dao层，和上面的一样。
3、和上面的一样。
4、也和上面的一样，但是需要改一部分。（以下是别人的代码，作者看到的话提醒我一下）
~~~
package com.example.demoforme.util;


import org.apache.poi.hssf.usermodel.HSSFCell;
import org.apache.poi.hssf.usermodel.HSSFCellStyle;
import org.apache.poi.hssf.usermodel.HSSFRow;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.ss.util.CellRangeAddress;
import org.apache.poi.xssf.streaming.SXSSFWorkbook;

/**
 * 导出报表
 */
public class ExportExcelUtil {
    /**
     * @param title
     * @param headers
     * @param values
     * @return
     */
    public static HSSFWorkbook getHSSFWorkbook(String title,String headers[],String values[][]){

        //工作薄
        HSSFWorkbook wb=new HSSFWorkbook();
        //sheet
        Sheet st=wb.createSheet(title);

        //创建标题合并行
        st.addMergedRegion(new CellRangeAddress(0,(short)0,0,(short)headers.length - 1));

        //设置标题样式
        HSSFCellStyle style=wb.createCellStyle();
        style.setAlignment(HorizontalAlignment.CENTER); //居中格式
        style.setVerticalAlignment(VerticalAlignment.CENTER);
        //设置标题字体
        Font titleFont = wb.createFont();
        titleFont.setFontHeightInPoints((short) 14);
        style.setFont(titleFont);

        //设置值表头样式 设置表头居中
        HSSFCellStyle hssfCellStyle = wb.createCellStyle();
        hssfCellStyle.setAlignment(HorizontalAlignment.CENTER);   //设置居中样式
        hssfCellStyle.setBorderBottom(BorderStyle.THIN);
        hssfCellStyle.setBorderLeft(BorderStyle.THIN);
        hssfCellStyle.setBorderRight(BorderStyle.THIN);
        hssfCellStyle.setBorderTop(BorderStyle.THIN);

        //设置表内容样式
        //创建单元格，并设置值表头 设置表头居中
        HSSFCellStyle style1 = wb.createCellStyle();
        style1.setBorderBottom(BorderStyle.THIN);
        style1.setBorderLeft(BorderStyle.THIN);
        style1.setBorderRight(BorderStyle.THIN);
        style1.setBorderTop(BorderStyle.THIN);

        //产生标题行
        HSSFRow hssfRow = (HSSFRow) st.createRow(0);
        HSSFCell cell = hssfRow.createCell(0);
        cell.setCellValue(title);
        cell.setCellStyle(style);

        //产生表头
        HSSFRow row1 = (HSSFRow) st.createRow(1);
        for (int i = 0; i < headers.length; i++) {
            HSSFCell hssfCell = row1.createCell(i);
            hssfCell.setCellValue(headers[i]);
            hssfCell.setCellStyle(hssfCellStyle);
        }

        //创建内容
        for (int i = 0; i <values.length; i++){
            row1 = (HSSFRow) st.createRow(i +2);
            for (int j = 0; j < values[i].length; j++){
                //将内容按顺序赋给对应列对象
                HSSFCell hssfCell = row1.createCell(j);
                hssfCell.setCellValue(values[i][j]);
                hssfCell.setCellStyle(style1);
            }
        }
        return wb;
    }
}

~~~
5.编写一个响应式请求和回应。
~~~
package com.example.demoforme.controller;

import com.alibaba.fastjson.JSON;
import com.example.demoforme.jpa.model.Students;
import com.example.demoforme.server.ExcelWService;
import com.example.demoforme.server.StudentsService;
import com.example.demoforme.util.ExportExcelUtil;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.OutputStream;
import java.io.UnsupportedEncodingException;
import java.util.List;

@RestController
@RequestMapping("/demo")
public class DemoController {
    @Autowired
    private StudentsService studentsService;

    @Autowired
    private ExcelWService excelWService;

    @GetMapping("/export")
    public void export(HttpServletRequest request, HttpServletResponse response){
        //excel标题
        String title = "学生表";
        //excel表名
        String [] headers = {"ID","班级","姓名","性别","分数","地址", "学校"};
        //excel文件名
        String fileName = title + System.currentTimeMillis()+".xls";

        List<Students> list = studentsService.find();

        //excel元素
        String content[][] = new String[list.size()][6];
        for (int i = 0; i < list.size(); i++) {
            content[i] = new String[headers.length];
            content[i][0]=String.valueOf(list.get(i).getId());
            content[i][1] = String.valueOf(list.get(i).getClassId());
            content[i][2] = list.get(i).getName();
            content[i][3] = list.get(i).getGender();
            content[i][4] = String.valueOf(list.get(i).getScore());
            content[i][5] = list.get(i).getAddress();
            content[i][6] = list.get(i).getSchool();
        }

        //创建HSSFWorkbook
        HSSFWorkbook wb = ExportExcelUtil.getHSSFWorkbook(title, headers, content);

        try {
            this.setResponseHeader(response, fileName);
            OutputStream os = response.getOutputStream();
            wb.write(os);
            os.flush();
            os.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        //发送响应流方法
    }
    public void setResponseHeader(HttpServletResponse response, String fileName) {
        try {
            try {
                fileName = new String(fileName.getBytes(), "ISO8859-1");
            } catch (UnsupportedEncodingException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
            response.setContentType("application/octet-stream;charset=ISO8859-1");
            response.setHeader("Content-Disposition", "attachment;filename=" + fileName);
            response.addHeader("Pargam", "no-cache");
            response.addHeader("Cache-Control", "no-cache");
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

~~~
运行代码，有如下的结果

![upload successful](/images/poi-w-9.png)
谢谢大家的观看，晚安啦！
