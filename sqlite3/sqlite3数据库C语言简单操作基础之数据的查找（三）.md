### sqlite3数据库C语言简单操作基础之数据的查找（三）

**概述**：sqlite3数据库的创建，增删改都说过了，今天就是数据库的查找，也算是数据库比较核心的应用。如果用之前的API执行，因查找数据库有返回值，也就导致使用sqlite_exec()函数时要写回调函数，我觉得这样子效率不高，编写麻烦，我这边就用几个新的API去做。这几个新的API其实就是sqlite_exec（）函数的分解版。

**1，分解版API介绍**

```c
int sqlite3_prepare_v2(
  sqlite3 *db,            /* Database handle ---数据库操作句柄*/
  const char *zSql,       /* SQL statement, UTF-8 encoded --- UTF8编码sql语句*/
  int nByte,              /* Maximum length of zSql in bytes. ---sql语句长度*/
  sqlite3_stmt **ppStmt,  /* OUT: Statement handle ---准备好的二进制执行语句句柄*/
  const char **pzTail     /* OUT: Pointer to unused portion of zSql ---指向sql语句中未使用的语句*/
);
/*
	这个函数主要作用就是将一条sql语句转换为sqlite3可执行二进制sql语句并存入sqlite3_stmt类型数据中，也就是sql语句的准备过程。
*/

int sqlite3_step(sqlite3_stmt*);
/*
	这个函数主要就是执行我们准备好的二进制sql语句，执行成功等状态通过返回值判断。
	SQLITE_BUSY 意味着数据库引擎无法获取执行其工作所需的数据库锁定；
	SQLITE_DONE 表示语句已成功执行；
	SQLITE_ROW 如果正在执行的SQL语句返回任何数据，那么每当调用者准备好处理一行新的数据时返回该值；
	SQLITE_ERROR 意味着发生了一个运行时错误；
	SQLITE_MISUSE 意味着这个例程被不恰当的调用；
*/
int sqlite3_column_int(sqlite3_stmt*, int iCol);
const unsigned char *sqlite3_column_text(sqlite3_stmt*, int iCol);
/*
	这些函数主要是在执行完sqlite3_step后，获取一行数据中某一列的值。这个值可能是int,text等类型。那就使用对应的类型函数去获取值。详细的官网地址：http://www.sqlite.org/c3ref/column_blob.html;
*/

int sqlite3_finalize（sqlite3_stmt * pStmt）
/*
	释放二进制sql执行语句，避免内存泄漏。
*/
```



**2，查找数据**

```
主要还是看sql语句语法，插入数据用到的关键字就是**SELECT**，它的语法知识可以简单的概括为两种。
```

```
SELECT 列名称 FROM 表名称
SELECT * FROM 表名称
后面还可加WHERE 条件 如 SELECT * FROM student WHERE name = 'SKy'
```

***下面是一个查找数据个数的代码：***

```c
/*查找数据个数*/
int sqlite_find_count(sqlite3 *db)
{
	/*查找名字为Sky的个数*/
	char *sql = "select count(*) from  student where name = 'Sky';";
	sqlite3_stmt *stmt = NULL;
	/*将sql语句转换为sqlite3可识别的语句，返回指针到stmt*/
	int res = sqlite3_prepare_v2(db, sql, strlen(sql), &stmt, NULL);
	if (SQLITE_OK != res || NULL == stmt) {
		goto err1;
	}
	/*执行准备好的sqlite3语句*/
	res = sqlite3_step(stmt);
	if (res != SQLITE_ROW) {
		goto err2;
	}
	int count = sqlite3_column_int(stmt, 0);
	if (count < 0) {
		goto err2;
	}
	printf("count = %d\n", count);
	sqlite3_finalize(stmt);
	return count;
err2:
	sqlite3_finalize(stmt);
err1:
	return -1;
}
```

***下面是一个查找数据并取出数据代码：***

```c
/*查找数据并取出数据*/
int sqlite_find_parse(sqlite3 *db)
{
	/*查找name为Sky的数据*/
	char *sql = "select * from  student where name = 'Sky';";
	sqlite3_stmt *stmt = NULL;
	/*将sql语句转换为sqlite3可识别的语句，返回指针到stmt*/
	int res = sqlite3_prepare_v2(db, sql, strlen(sql), &stmt, NULL);
	if (SQLITE_OK != res || NULL == stmt) {
		goto err1;
	}
	/*执行准备好的sqlite3语句*/
	while (SQLITE_ROW == sqlite3_step(stmt)) {
		printf("name: %s, uuid: %u\n",\
		sqlite3_column_text(stmt, 1),\
		sqlite3_column_int(stmt, 2));
	}
	sqlite3_finalize(stmt);
	return 0;
err2:
	sqlite3_finalize(stmt);
err1:
	return -1;
}
```



**3，整个demo源码与运行结果**

***demo***

```c
#include <stdio.h>
#include <stdint.h>
#include <string.h>
#include <time.h>
#include <pthread.h>
#include "sqlite/sqlite3.h"


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
                "INSERT INTO student (name,uuid) VALUES('Sky', 17531007);";
	char *err_msg = NULL;
	int rc = sqlite3_exec(db, sql, 0, 0, &err_msg);
    
    if (rc != SQLITE_OK ) { 
        fprintf(stderr, "SQL error: %s\n", err_msg);
        sqlite3_free(err_msg);         
        return -1;
    } 
    return 0;
}

/*查找数据个数*/
int sqlite_find_count(sqlite3 *db)
{
	/*查找名字为Sky的个数*/
	char *sql = "select count(*) from  student where name = 'Sky';";
	sqlite3_stmt *stmt = NULL;
	/*将sql语句转换为sqlite3可识别的语句，返回指针到stmt*/
	int res = sqlite3_prepare_v2(db, sql, strlen(sql), &stmt, NULL);
	if (SQLITE_OK != res || NULL == stmt) {
		goto err1;
	}
	/*执行准备好的sqlite3语句*/
	res = sqlite3_step(stmt);
	if (res != SQLITE_ROW) {
		goto err2;
	}
	int count = sqlite3_column_int(stmt, 0);
	if (count < 0) {
		goto err2;
	}
	printf("count = %d\n", count);
	sqlite3_finalize(stmt);
	return count;
err2:
	sqlite3_finalize(stmt);
err1:
	return -1;
}

/*查找数据并取出数据*/
int sqlite_find_parse(sqlite3 *db)
{
	/*查找name为Sky的数据*/
	char *sql = "select * from  student where name = 'Sky';";
	sqlite3_stmt *stmt = NULL;
	/*将sql语句转换为sqlite3可识别的语句，返回指针到stmt*/
	int res = sqlite3_prepare_v2(db, sql, strlen(sql), &stmt, NULL);
	if (SQLITE_OK != res || NULL == stmt) {
		goto err1;
	}
	/*执行准备好的sqlite3语句*/
	while (SQLITE_ROW == sqlite3_step(stmt)) {
		printf("name: %s, uuid: %u\n",\
		sqlite3_column_text(stmt, 1),\
		sqlite3_column_int(stmt, 2));
	}
	sqlite3_finalize(stmt);
	return 0;
err2:
	sqlite3_finalize(stmt);
err1:
	return -1;
}

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

	insert_data(db);
	sqlite_find_count(db);
	sqlite_find_parse(db);
    /*关闭数据库*/
    sqlite3_close(db);
    return 0;
}
```

***运行结果如下：***

```
sky@ubuntu:~/Study/sqlite3/build$ ./sqlite_test 
count = 2
name: Sky, uuid: 17531002
name: Sky, uuid: 17531007
```

