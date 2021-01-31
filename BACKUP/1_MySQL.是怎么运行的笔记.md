# [MySQL 是怎么运行的笔记](https://github.com/EruDev/blog/issues/1)

## Chapter 01. 初识 MySQL

1. MySQL 采用客户端/服务器架构
2. 客户端与服务器通信的方式
    - TCP/IP；
    - 命名管道或内存；
    - UNIX 域套接字。
3. 处理查询请求
    - 连接管理：主要负责连接的建立与信息的认证
    - 解析与优化：主要进行查询缓存、语法解析、查询优化
    - 存储引擎：主要负责读取和写入底层表中的数据

## Chapter 02. 系统变量

1. 启动选项可以在命令行中直接指定，或者在配置文件中修改。
2. 修改系统变量的方式
    - 在服务器启动时添加相应的启动选项修改；
    - SET [GLOBALE|SESSION] 系统变量名 = 值；
    - SET[@@(GLOBAL|SESSION).] 系统变量名 = 值；
3. 查看的系统变量：`SHOW [GLOBAL|SESSION] VARIABLES [LIKE 匹配的模式]`;  `show variables like 'max_conn%'`