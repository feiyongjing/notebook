# mysql字符问题
~~~shell
# 执行以下命令显示使用的字符集编码
show variables like 'character%';
# 结果如下
+--------------------------+------------+
| Variable_name            | Value      |
+--------------------------+------------+
| character_set_client     | utf8mb4    |
| character_set_connection | utf8mb4    |
| character_set_database   | utf8mb4    |
| character_set_filesystem | binary     |
| character_set_results    | utf8mb4    |
| character_set_server     | utf8mb4    |
| character_set_system     | utf8       |
+--------------------------+------------+
# character_set_client 是客户端进程使用的编码
# character_set_connection 是客户端连接所使用的编码
# character_set_database 是磁盘上数据库的数据使用的编码
# character_set_filesystem 是磁盘文件系统使用的编码
# character_set_results 是客户端查询服务端的结果显示使用的编码
# character_set_server 是mysql服务端进程使用的编码
# character_set_system  是mysql服务端的服务器操作系统使用的编码

# 出现乱码的问题情况，客户端使用的编码、客户端mysql连接工具、服务端mysql使用的编码保持一致，并且服务端和服务端的操作系统环境变量支持该编码
~~~

### 查看mysql的客户端链接库函数，在mysql服务端执行 nm /usr/lib64/mysql/libmysqlclient.a | grep mysql_init 查看具体的函数
~~~shell
# nm 命令用于列出目标文件的符号表内容。目标文件可以是执行文件、目标文件（.o）、共享库（.so）等。符号表包含了程序中各种变量、函数等标识符的信息，这些信息对于理解和分析程序的执行流程非常有用。
nm [options] [file...]
#-g：仅显示外部符号（global symbols）。
#-u：仅显示未定义的符号。
#-C：解码（demangle）低级别的符号名称到用户级别的名称，使其更易于理解。
#-n 或 --numeric-sort：按照符号地址排序输出。
#-p：按照符号表中的顺序显示，不排序。
#-l：显示符号所在的源代码行号。
~~~

# 查看mysql的头文件
~~~shell
find / -name mysql.h
# 一般是这个目录/usr/include/mysql/mysql.h
~~~

# 常见API，参考官网：https://www.mysqlzh.com/doc/196/114.html
### STDCALL是一个宏定义，用于指定函数的调用约定。在C和C++编程中，调用约定决定了函数的参数如何被传递到栈上，以及谁负责清理栈：调用者还是被调用者。STDCALL是“standard calling convention”的缩写，它是Windows API中使用的默认调用约定之一，也被称为__stdcall。__stdcall广泛用于Windows API函数和某些类型的回调函数，因为它可以生成比__cdecl更紧凑的代码。由于被调用函数自身清理栈，所以每个函数调用的代码大小会减少，这在包含大量函数调用的程序中尤其有益。在不同的编译器和平台上，STDCALL可能有不同的实现方式。例如，在Microsoft Visual C++中，它通常被定义为__stdcall。在其他平台或编译器中，如果不支持__stdcall，STDCALL可能被定义为空，这意味着它会退回到默认的调用约定。
~~~c
// 初始化mysql对象
// 如果参数mysql是NULL指针，该函数将分配、初始化、并返回新对象。否则，将初始化对象，并返回对象的地址。内存不足时返回NULL
MYSQL *      STDCALL mysql_init(MYSQL *mysql);

// 错误处理
unsigned int STDCALL mysql_errno(MYSQL *mysql);
const char * STDCALL mysql_error(MYSQL *mysql);

// 建立连接，如果连接成功，返回MYSQL*连接句柄。如果连接失败，返回NULL。对于成功的连接，返回值与第1个参数的值相同。
// mysql参数是mysql_init得到的mysql对象
// host参数必须是主机名或IP地址。如果是NULL或"localhost"，连接将被视为与本地主机的连接。如果操作系统支持套接字（Unix）或命名管道（Windows），将使用它们而不是TCP/IP连接到服务器
// user参数包含用户的MySQL登录ID。如果“user”是NULL或空字符串""，用户将被视为当前用户。在UNIX环境下，它是当前的登录名。在Windows ODBC下，必须明确指定当前用户名。
// passwd参数包含用户的密码。如果“passwd”是NULL，仅会对该用户的（拥有1个空密码字段的）用户表中的条目进行匹配检查。调用mysql_real_connect()之前，不要尝试加密密码，密码加密将由客户端API自动处理
// db参数是数据库名称。如果db为NULL，连接会将默认的数据库设为该值
// port参数不是0，其值将用作TCP/IP连接的端口号
// unix_socket参数不是NULL，该字符串描述了使用的套接字或命名管道
// client_flag的值通常为0，但是，也能将其设置为下述标志的组合，以允许特定功能:
//             CLIENT_COMPRESS           使用压缩协议。
//             CLIENT_FOUND_ROWS         返回发现的行数（匹配的），而不是受影响的行数。
//             CLIENT_IGNORE_SPACE       允许在函数名后使用空格。使所有的函数名成为保留字。
//             CLIENT_INTERACTIVE        关闭连接之前，允许interactive_timeout（取代了wait_timeout）秒的不活动时间。客户端的会话wait_timeout变量被设为会话interactive_timeout变量的值。
//             CLIENT_LOCAL_FILES        允许LOAD DATA LOCAL处理功能。
//             CLIENT_MULTI_STATEMENTS   通知服务器，客户端可能在单个字符串内发送多条语句（由‘;’隔开）。如果未设置该标志，将禁止多语句执行。
//             CLIENT_MULTI_RESULTS      通知服务器，客户端能够处理来自多语句执行或存储程序的多个结果集。如果设置了CLIENT_MULTI_STATEMENTS，将自动设置它。
//             CLIENT_NO_SCHEMA          禁止db_name.tbl_name.col_name语法。它用于ODBC。如果使用了该语法，它会使用分析程序生成错误
//             CLIENT_ODBC               客户端是ODBC客户端。它将mysqld变得对ODBC更为友好。
//             CLIENT_SSL                使用SSL（加密协议）。该选项不应由应用程序设置，它是在客户端库内部设置的。
MYSQL *      STDCALL mysql_real_connect(MYSQL *mysql, const char *host,
                        const char *user,
                        const char *passwd,
                        const char *db,
                        unsigned int port,
                        const char *unix_socket,
                        unsigned long clientflag);

// 执行sql语句(不单单只是指查询sql)，如果执行成功，返回0。如果出现错误，返回非0值。
// 执行由“Null终结的字符串”查询指向的SQL查询。正常情况下，字符串必须包含1条SQL语句，而且不应为语句添加终结分号（‘;’）或“\g”。如果允许多语句执行，字符串可包含多条由分号隔开的语句。
// 如果希望了解查询是否应返回结果集，可使用mysql_field_count()进行检查
// 注意mysql_query 不能执行包含二进制数据的查询，二进制数据可能包含字符 '\0'，如果需要查询二进制数据需要使用mysql_real_query函数
int STDCALL mysql_query(MYSQL *mysql, const char *q);

// 获取sql执行结果集合 MYSQL_RES* 表示结果集合即多行数据，MYSQL_ROW是结果集中的一行数据
// 对于非查询sql执行后，不需要调用mysql_store_result()或mysql_use_result()，但是如果在任何情况下均调用了mysql_store_result()，它也不会导致任何伤害或性能降低。
// 通过检查mysql_store_result()是否返回0，可检测查询是否没有结果集（以后会更多）。如果希望了解查询是否应返回结果集，可使用mysql_field_count()进行检查。
// mysql_store_result()将查询的全部结果读取到客户端，分配1个MYSQL_RES结构，并将结果置于该结构中。如果查询未返回结果集，mysql_store_result()将返回Null指针（例如，如果查询是INSERT语句）。如果读取结果集失败，mysql_store_result()也会返回Null指针。
// 注意一旦完成了对结果集的操作，必须调用mysql_free_result()释放结果集
MYSQL_RES * STDCALL mysql_store_result(MYSQL *mysql);
MYSQL_RES * STDCALL mysql_use_result(MYSQL *mysql)

// 从结果集中获取一行数据，并且内部指针移动到下一行（类似文件内部的读写指针）
// 检索结果集的下一行。在mysql_store_result()之后使用时，如果没有要检索的行，mysql_fetch_row()返回NULL。在mysql_use_result()之后使用时，如果没有要检索的行或出现了错误，mysql_fetch_row()返回NULL。
// 行内值的数目由mysql_num_fields(result)给出。如果行中保存了调用mysql_fetch_row()返回的值，将按照row[0]到row[mysql_num_fields(result)-1]，访问这些值的指针。行中的NULL值由NULL指针指明。
// 可以通过调用mysql_fetch_lengths()来获得行中字段值的长度。对于空字段以及包含NULL的字段，长度为0。通过检查字段值的指针，能够区分它们。如果指针为NULL，字段为NULL，否则字段为空。
MYSQL_ROW   STDCALL mysql_fetch_row(MYSQL_RES *result);

// mysql_field_count获取作用在连接上的最近sql查询的结果的列数
// mysql_num_fields获取结果集中的列数
unsigned int mysql_field_count(MYSQL *mysql);
unsigned int mysql_num_fields(MYSQL_RES *result);

// 对于结果集，返回所有MYSQL_FIELD结构的数组，即表格的字段信息
// 返回一个MYSQL_FIELD结构体数组，通过调用MYSQL_FIELD结构体的name属性可以获取到结果集表格的名称
MYSQL_FIELD * mysql_fetch_fields(MYSQL_RES *result);
MYSQL_FIELD * mysql_fetch_field(MYSQL_RES *result);

// 获取mysql连接使用的字符集（MY_CHARSET_INFO结构体的name属性是字符集名称）
void mysql_get_character_set_info(MYSQL *mysql, MY_CHARSET_INFO *cs);
// 设置mysql连接使用的字符集
int mysql_set_character_set(MYSQL *mysql, char *csname);

// 释放结果集合内存
void        STDCALL mysql_free_result(MYSQL_RES *result);

// 关闭连接
void        STDCALL mysql_close(MYSQL *sock);

// 以下几个API是预处理sql，对于多次执行的语句，预处理执行是一种有效的方式。首先对语句进行解析，为执行作好准备。接下来，在以后使用初始化函数返回的语句句柄执行一次或多次。对于多次执行的语句，预处理执行比直接执行快，主要原因在于，仅对查询执行一次解析操作。在直接执行的情况下，每次执行语句时，均将进行查询。此外，由于每次执行预处理语句时仅需发送参数的数据，从而减少了网络通信量。预处理语句的另一个优点是，它采用了二进制协议，从而使得客户端和服务器之间的数据传输更有效率。
// 创建MYSQL_STMT句柄，准备开始进行预处理sql字符串
MYSQL_STMT *mysql_stmt_init(MYSQL *mysql);
// 为执行操作准备SQL字符串，其中？是占位符(参数标记符)
int mysql_stmt_prepare(MYSQL_STMT *stmt, const char *query, unsigned long length);
// 返回预处理SQL语句中的参数数目
unsigned long mysql_stmt_param_count(MYSQL_STMT *stmt);
// 将内存中的数据与预处理SQL语句中的参数标记符关联起来
my_bool mysql_stmt_bind_param(MYSQL_STMT *stmt, MYSQL_BIND *bind);
// 执行预处理语句
int mysql_stmt_execute(MYSQL_STMT *stmt);
// 关闭预处理语句。此外，mysql_stmt_close()还会取消由“stmt”指向的语句句柄分配
my_bool mysql_stmt_close(MYSQL_STMT *);

~~~

# 连接数据库，执行sql获取结果集例子
### 编译命令如下
~~~shell
gcc mysql-test.c -o mysql-test.o -I/usr/include/mysql -L/usr/lib64/mysql -lmysqlclient
# -I参数指定需要的头文件目录
# -L参数指定需要的库文件目录
# -l参数指定需要的库名称
~~~

# 示例代码如下
~~~c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include "mysql.h"

int main(){
    int i=0;
    // 初始化
    MYSQL * mysql = mysql_init(NULL);
    if(mysql==NULL){
        printf("内存不足导致mysql初始化失败\n");
        return -1;
    }
    printf("mysql init ok\n");

    // 连接数据库
    MYSQL* conn = mysql_real_connect(mysql, "192.168.0.119", "root", "cloudgrow.cn", "wz_shoe", 30306, NULL, 0);
    if(conn==NULL){
        printf("wz_shoe数据库连接失败\n");
        return -1;
    }
    printf("wz_shoe数据库连接成功\n");

    // 执行sql
    char sSql[255] = "select * from test;";
    int ret = mysql_query(conn, sSql);
    if(ret!=0){
        printf("wz_shoe数据库test表格查询失败\n");
        return -1;
    }
    printf("wz_shoe数据库test表格查询成功\n");

    // 方式一：获取查询sql执行后的结果列数
    // unsigned int num = mysql_field_count(conn);

    // 获取sql执行的结果集
    MYSQL_RES* results = mysql_store_result(conn);
    if(results==0){
        printf("wz_shoe数据库test表格查询结果获取失败\n", mysql_error(conn));
        return -1;
    }
    printf("wz_shoe数据库test表格查询结果集\n");

    // 方式二：获取查询结果集列数
    unsigned int num =  mysql_num_fields(results);

    // 解析sql执行的结果集表头
    MYSQL_FIELD* fields = mysql_fetch_fields(results);
    if(fields == NULL){
        printf("wz_shoe数据库test表格查询结果的表格头信息获取失败\n", mysql_error(conn));
        return -1;
    }
    for(i=0;i<num;i++){
        printf("%s\t", fields[i].name);
    }
    printf("\n");

    // 解析sql执行的结果集
    MYSQL_ROW row = NULL;
    while((row = mysql_fetch_row(results))){
        for(i=0;i<num;i++){
            printf("%s\t", row[i]);
        }
        printf("\n");
    }

    // 回收结果集占用的内存
    mysql_free_result(results);

    // 关闭mysql连接
    mysql_close(conn);
    return 0;
}
~~~
---

# 一个连接mysql的客户端例子
### 编译命令如下
~~~shell
gcc mysql-client.c -o mysql-client.o -I/usr/include/mysql -L/usr/lib64/mysql -lmysqlclient
# -I参数指定需要的头文件目录
# -L参数指定需要的库文件目录
# -l参数指定需要的库名称
~~~

### 连接mysql的客户端，示例代码如下
~~~c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#include "mysql.h"

int main(){
    int i=0;
    // 初始化
    MYSQL * mysql = mysql_init(NULL);
    if(mysql==NULL){
        printf("内存不足导致mysql初始化失败\n");
        return -1;
    }
    printf("mysql init ok\n");

    // 连接数据库
    MYSQL* conn = mysql_real_connect(mysql, "192.168.0.119", "root", "cloudgrow.cn", "wz_shoe", 30306, NULL, 0);
    if(conn==NULL){
        printf("wz_shoe数据库连接失败\n");
        return -1;
    }
    printf("wz_shoe数据库连接成功\n");

    MY_CHARSET_INFO cs;
    mysql_get_character_set_info(mysql, &cs);
    printf("mysql连接默认使用的字符集是：%s\n", cs.name);
    // 修改连接mysql使用的字符集
    mysql_set_character_set(conn, "utf8");

    mysql_get_character_set_info(mysql, &cs);
    printf("mysql连接修改字符集后使用的字符集是：%s\n", cs.name);
    char sSql[1024];
    char* p;

    while(1){
        // 打印mysql命令行标志
        write(STDOUT_FILENO, "mysql> ", strlen("mysql> "));

        // 读取输入的sql
        memset(sSql, 0x00, sizeof(sSql));
        read(STDIN_FILENO, sSql, sizeof(sSql));
        // 去除sql末尾的分号
        p = strrchr(sSql, ';');
        if(p!=NULL){
            *p = '\0';
        }

        // 处理开头的回车
        if(sSql[0]=='\n'){
            continue;
        }

        // 处理sql开头的空格
        for(i=0;i<sizeof(sSql);i++){
            if(sSql[i]!=' '){
                break;
            }
        }
        memmove(sSql, sSql+i, sizeof(sSql)-i+1);

        if(strncasecmp(sSql, "exit", 4) == 0 || strncasecmp(sSql, "quit", 4) == 0){
            mysql_close(conn);
            exit(0);
        }

        int ret = mysql_query(conn, sSql);
        if(ret!=0){
            printf("%s\n", mysql_error(conn));
            continue;
        }

        if(strncasecmp(sSql, "select", 6) != 0){
            // 非查询语句返回数据变更行数
            printf("Query OK, %ld row affected (0.02 sec)\n", mysql_affected_rows(conn));
            continue;
        }

        // 获取sql执行的结果集
        MYSQL_RES* results = mysql_store_result(conn);
        if(results==0){
            printf("%s\n", mysql_error(conn));
            continue;
        }

        // 获取查询结果集列数
        unsigned int num =  mysql_num_fields(results);

        // 解析sql执行的结果集表头
        MYSQL_FIELD* fields = mysql_fetch_fields(results);
        if(fields == NULL){
            printf("%s\n", mysql_error(conn));
            mysql_free_result(results);
            continue;
        }
        for(i=0;i<num;i++){
            printf("%s\t", fields[i].name);
        }
        printf("\n");

        // 解析sql执行的结果集
        MYSQL_ROW row = NULL;
        while((row = mysql_fetch_row(results))){
            for(i=0;i<num;i++){
                printf("%s\t", row[i]);
            }
            printf("\n");
        }

        // 回收结果集占用的内存
        mysql_free_result(results);
    }

    mysql_close(conn);
    return 0;
}
~~~
