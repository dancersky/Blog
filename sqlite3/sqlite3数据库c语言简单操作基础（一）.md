### sqlite3数据库c语言简单操作基础（一）

**概述**：sqlite3数据库是一个比较精简的数据库操作库，在嵌入式设备上，因其轻量级，大多使用的就是这货了。当然它的使用也是比较简单的，通过几个基础的API就可以完成一些基本的操作了。最近使用这货，所以做一下学习笔记，省得以后用到又的查资料麻烦。这篇笔记就主要是数据库的创建还有基础的几个API介绍。后面的话会具体记录一下数据库的增删改查，以及事务部分。

**使用测试环境：**
	系统linux：ubuntu14.04

**1，sqlite3库下载地址**
官网地址：[https://www.sqlite.org/download.html](https://www.sqlite.org/download.html).

**2，四个基础API介绍**
```c
int sqlite3_open(
  const char *filename,   /* Database filename */
  sqlite3 **ppDb          /* OUT: SQLite db handle */
);
/*
打开/创建数据库文件的API,第一个参数是文件路径及名字，第二个参数是sqlite3操作句柄。在使用该API时，如果我们打开的数据库文件不存在就创建一个，并且会返回一个数据库操作句柄，保存到我们输入的第二个参数。操作成功，返回值为SQLITE_OK.
*/

int sqlite3_exec(
  sqlite3*,                                  /* An open database */
  const char *sql,                           /* SQL to be evaluated */
  int (*callback)(void*,int,char**,char**),  /* Callback function */
  void *,                                    /* 1st argument to callback */
  char **errmsg                              /* Error msg written here */
);
/*
这个API就是主要执行sql语句的，第一个参数，打开的数据库操作句柄。第二个参数，sql语句，就是我们要执行的命令。第三个参数，回调函数。第四个参数，传入回调函数的参数。第五个参数，保存操作失败时的错误信息。操作成功，返回值为SQLITE_OK.
*/

void sqlite3_free（void *）;
/*
这个API就是释放申请的动态内存了，在上一个API操作中，假设出现操作失败，错误信息保存在最后一个参数中，它的内存是动态申请的，这时候我们就要通过这个函数就释放内存了。
*/

int sqlite3_close（sqlite3 *）;
/*
这个API就是关闭数据库的操作，第一个参数就是sqlite3操作句柄。一般在我们结束或出错时调用来关闭数据库。
*/
```
上面说到了sql语句，对于sql语句的语法可能要自己了解一下，这里放一个网址：
[http://www.runoob.com/sqlite/sqlite-insert.html](http://www.runoob.com/sqlite/sqlite-insert.html).

**3，数据库文件及表创建**
```c
#include <sqlite3.h>
#include <stdio.h>

#define TABLE      "student"

int main(void) {
    
    sqlite3 *db;
    char *err_msg = NULL;
    /*打开或创建数据库test.db文件*/
    int rc = sqlite3_open("test.db", &db);
    if (rc != SQLITE_OK) {
        printf("open database test.db failed\n");
        sqlite3_close(db);
        return 1;
    }
    
    /*
    此sql语句意思，如果student这张表不存在就创建student表，
	表的格式为主键id int类型、name 字符串类型、uuid int类型、uuid 唯一性 不可重复
	*/
    char *sql =  "CREATE TABLE IF NOT EXISTS student (\
                [id] INTEGER PRIMARY KEY AUTOINCREMENT,\
                [name] TXT,\
                [uuid] INTEGER,\
                UNIQUE([uuid])\
                );";
	/*执行上述sql语句*/
    rc = sqlite3_exec(db, sql, 0, 0, &err_msg);
    if (rc != SQLITE_OK ) {
        printf("SQL error: %s\n", err_msg);
        sqlite3_free(err_msg);        
        sqlite3_close(db);
        return 1;
    } 
    /*关闭数据库*/
    sqlite3_close(db);
    return 0;
}
```

