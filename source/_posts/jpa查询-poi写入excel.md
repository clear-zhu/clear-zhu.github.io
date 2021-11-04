title: jpa查询+poi写入excel
author: carl-zhu
tags: []
categories: []
date: 2021-11-04 19:28:00
---
## 目的
任务目的是利用jpa查询数据库中的数据，并利用poi库将其写入excel表中。

### poi
### 搭建过程
#### 创建工作薄
poi对excel是按照工作薄、每一页、每一行、每一列进行操作的，所以在所有操作之前，要先新建一个工作薄。
在poi中有三种工作薄。
~~~
HSSFWorkbook ：2003版excel  以.xls结尾
XSSFWorkbook ：2007版的excel 以.xlsx结尾
SXSSFWorkbook :支持低内存占用的操作方式 以.xlsx结尾

~~~