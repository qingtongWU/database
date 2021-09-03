~~~c
#include <mysql/mysql.h>                    
#include <string.h>
#include <stdio.h>
#include<stdlib.h>

int findparent_id(char *curfilename,int *parent_id){
    int index=0;
    char parent[10][10]={0};
    MYSQL *conn;
    MYSQL_RES *res;
    MYSQL_ROW row;
    char *server = "localhost";
    char *user = "root";
    char *password = "W111";
    char *database = "project";//要访问的数据库名称
    char query[300] = "select * from document where filename='";
    unsigned int queryRet;
    sprintf(query, "%s%s%s", query, curfilename,"'");//将不同格式的数据连接成一个字符串    

    conn = mysql_init(NULL);//初始化检错
    if(!conn){
        printf("MySQL init failed\n");
        return -1;
    }

    if(!mysql_real_connect(conn, server, user, password, database, 0, NULL, 0)){
        printf("Error connecting to database: %s\n", mysql_error(conn));
        return -1;
    }//判断数据库有没有连接成功
    queryRet = mysql_query(conn, query);//返回查询结果是否正确

    if(0==queryRet){
        res = mysql_store_result(conn);//存储查询结果
        row = mysql_fetch_row(res);//捕捉查询结果
        if(NULL != row){
            strcpy(parent[index],row[1]);
        }
        //没有找到返回-1
        else{
            return -1;
        }
        mysql_free_result(res);
        mysql_close(conn);
        *parent_id=atoi(parent[index]);
        return 0;
    }
    //没有找到返回-1
    else{
        return -1;
    }    
}

int findFileName(int parent_id,char *filename){
    char fname[10][20];
    int index=0;
    MYSQL *conn;
    MYSQL_RES *res;
    MYSQL_ROW row;
    char *server = "localhost";
    char *user = "root";
    char *password = "W111";
    char *database = "project";//要访问的数据库名称
    char query[300] = "select * from document where id=";                                                                                                                                                                        
    unsigned int queryRet;
    sprintf(query, "%s%d", query, parent_id);//将不同格式的数据连接成一个字符串    
    //puts(query);

    conn = mysql_init(NULL);
    if(!conn){
        printf("MySQL init failed\n");
        return -1;
    }
    if(!mysql_real_connect(conn, server, user, password, database, 0, NULL, 0)){
        printf("Error connecting to database: %s\n", mysql_error(conn));
        return -1;
    }
    queryRet = mysql_query(conn, query);
    if(0==queryRet){
        res = mysql_store_result(conn);
        row = mysql_fetch_row(res);
        if(NULL != row){
            sprintf(fname[index],"%s",row[2]);                  
        }
        mysql_free_result(res);
        mysql_close(conn);
        strcpy(filename,fname[index]);
        return 0;
    }
    //没有找到返回-1
    else{
        return -1;
    }

}
int pwd(int id){
    int aid[10]={0};
    int i=0;
    int ret=2;
    int parent_id;
    char filename[10]={0};

    char b1[10]={0};
    findFileName(id,filename);
    strcpy(b1,filename);
    while(1){
        ret= findparent_id(b1,&parent_id);
        if(-1==ret){
            printf("file or directory is not exist,please re-enter filename\n");
            return 0;
        }
        aid[i]=parent_id;
        if(0==parent_id){
            break;
        }
        i++;
        memset(filename,0,sizeof(filename));
        memset(b1,0,sizeof(b1));
        findFileName(parent_id,filename);
        strcpy(b1,filename);
    }
    if(0==i){
        printf("root content\n");
    }
    else{
        memset(filename,0,sizeof(filename));
        char a1[10]={0};
        for(int j=i-1;j>=0;j--){
            findFileName(aid[j],filename);
            strcat(a1,filename);
            strcat(a1,"/");
        }
        puts(a1);
    }
    return 0;
}
int main(int argc,char **argv){
    int b1=atoi(argv[1]);
    pwd(b1);//b1是输入的文件名
    return 0;
}

~~~

