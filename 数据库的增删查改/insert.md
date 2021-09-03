~~~c
#include <mysql/mysql.h>
#include <string.h>
#include <stdio.h>

int main(int argc,char* argv[]){
    MYSQL *conn;//结构体
    char *server = "localhost";
    char *user = "root";
    char *password = "W111";
    char *database = "project";//要访问的数据库名称
//    char query[200]="insert into user (id, username, salt,cryptpasswd,pwd,) values (1,'zhaoxin','ttyy','gfgfg','/zhao/')";

    char query[200]="insert into user (id,username,salt,cryptpasswd,pwd) values (2,'xiaotian','uuyy','uyrty','/xiaotian/')";
    int queryResult;
    conn = mysql_init(NULL);//初始化结构体
    if(!mysql_real_connect(conn, server, user, password, database, 0, NULL, 0)) //连接数据库{
        printf("Error connecting to database: %s\n", mysql_error(conn));
    }
    else{
        printf("Connected...\n");
    }

    queryResult = mysql_query(conn, query);//查询结果
    if(queryResult){
        printf("Error making query:%s\n", mysql_error(conn));
    }
    else{
        printf("insert success\n");
    }
    mysql_close(conn);
    return 0;
}
~~~

