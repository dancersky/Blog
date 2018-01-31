### sqlite3数据库C语言简单操作基础之数据的增删改（二）

**概述**：前面记录了sqlite3数据库c接口的一些基础API的功能介绍以及数据库文件创建及表的创建，今天主要就是记录一下数据库数据的增删改，为啥这里没说查，后面会单独做一个查的笔记。毕竟查可能就有点不一样了，它是有返回值的，所以就分类到下次记录吧。

**1，插入数据**
	主要还是看sql语句语法，插入数据用到的关键字就是**INSERT**，它的语法知识可以简单的概括为两种。

```
INSERT INTO 表名称 VALUES (值1, 值2,....)
INSERT INTO 表名称(列1, 列2,...) VALUES (值1, 值2,....)
```

***下面是一个承接上篇的一张学生表的数据插入代码；***

```c
/*插入数据到数据库*/
int insert_data(sqlite3 *db)
{
	char *sql = "INSERT INTO student (name,uuid) VALUES('Alice', 17531000);" 
                "INSERT INTO student (name,uuid) VALUES('Bob', 17531001);" 
                "INSERT INTO student (name,uuid) VALUES('Sky', 17531002);" 
                "INSERT INTO student (name,uuid) VALUES('Born', 17531003);" 
                "INSERT INTO student (name,uuid) VALUES('Jason', 17531004);" 
                "INSERT INTO student (name,uuid) VALUES('Mike', 17531005);" 
                "INSERT INTO student (name,uuid) VALUES('Tisa', 17531006);" 
                "INSERT INTO student (name,uuid) VALUES('Rush', 17531007);";
	char *err_msg = NULL;
	int rc = sqlite3_exec(db, sql, 0, 0, &err_msg);
    
    if (rc != SQLITE_OK ) { 
        fprintf(stderr, "SQL error: %s\n", err_msg);
        sqlite3_free(err_msg);         
        return -1;
    } 
    return 0;
}
```
***插入数据后我们可以看到数据库的数据为：***
 ![image](http://img.blog.csdn.net/20180112110229629?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRGFuY2VyX19Ta3k=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


**2，更新数据**
	主要还是看sql语句语法，更新数据用到的关键字就是**UPDATE**，它的语法知识如下。

```
UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值
```

***下面是一个承接上篇的一张学生表的数据更新代码；***

```c
/*更新数据到数据库*/
int update_data(sqlite3 *db)
{
	char *sql = "UPDATE student SET name = 'Alice-c' WHERE uuid = 17531000;" 
				"UPDATE student SET name = 'Bob-c' WHERE uuid = 17531001;";
	char *err_msg = NULL;
	int rc = sqlite3_exec(db, sql, 0, 0, &err_msg);
    
    if (rc != SQLITE_OK ) { 
        fprintf(stderr, "SQL error: %s\n", err_msg);
        sqlite3_free(err_msg);        
        return -1;
    } 
    return 0;
}
```
***更新数据后我们可以看到数据库的数据为：***

![这里写图片描述](http://img.blog.csdn.net/20180112110449164?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRGFuY2VyX19Ta3k=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**3，删除数据**
	主要还是看sql语句语法，删除数据用到的关键字就是**DELETE**，它的语法知识如下。

```
DELETE FROM 表名称 WHERE 列名称 = 值
```

***下面是一个承接上篇的一张学生表的数据删除代码；***
```c
/*删除数据库数据*/
int delete_data(sqlite3 *db)
{
	char *sql = "DELETE FROM student WHERE uuid = 17531000;" 
				"DELETE FROM student WHERE uuid = 17531001;"
				"DELETE FROM student WHERE uuid = 17531002;"
				"DELETE FROM student WHERE uuid = 17531003;";
	char *err_msg = NULL;
	int rc = sqlite3_exec(db, sql, 0, 0, &err_msg);
    
    if (rc != SQLITE_OK ) { 
        fprintf(stderr, "SQL error: %s\n", err_msg);
        sqlite3_free(err_msg);        
        return -1;
    } 
    return 0;
}
```
***删除数据后我们可以看到数据库的数据为：***
![这里写图片描述](http://img.blog.csdn.net/20180112110822621?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRGFuY2VyX19Ta3k=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**4，最后**
	查看数据库文件的软件地址：[http://sqlitebrowser.org/](http://sqlitebrowser.org/).
	我自己写了一个sqlite3的操作demo，代码下载用cmake编译一下就可以跑了，一般满足基本需求了，如果需要的话可以下载：http://download.csdn.net/download/dancer__sky/10200497.

 

