## Mysql 安装

电脑系统： Mac OS 

安装时问题：
一年前已安装mysql，当时没有配置环境变量
按照教程配置环境变量如下：
> ''' export PATH=$PATH:/usr/local/mysql/bin '''
> ''' source ~/.bash_profile '''

配置环境变量后执行：
> ''' mysql -u root -p '''

执行后输入密码，提示报错，报错信息如下：
> ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)

网上搜索了下，有人解释原因是：
> 有可能是 my.cnf 配置文件中设置了 [mysqld] 的参数 socket ，而没有设置[client]的参数socket
> mysql.sock 文件有什么用：连接localhost通常通过一个Unix域套接字文件进行，一般是/tmp/mysql.sock。如果套接字文件被删除了，本地客户就不能连接。/tmp 文件夹属于临时文件，随时可能被删除。

解决方案：
1. mysql 支持 socket 和 TCP/IP 连接，可以尝试用TCP/IP方式登陆：
> ''' mysql -u root -h 127.0.0.1 -p '''
2. 执行后有新的报错：
> ERROR 2003 (HY000): Can't connect to MySQL server on '127.0.0.1' (61)   
3. 网上分析原因可能是mysql服务的进程没有启动，所以执行命令启动mysql服务
> ''' sudo /etc/init.d/mysqld start '''
> 提示： sudo: /etc/init.d/mysqld: command not found
4. 尝试另一种启动方式
> '''  /usr/local/mysql '''
> '''  sudo ./support-files/mysql.server start '''
5. 启动成功：
> Starting MySQL 
> .Logging to '/usr/local/mysql/data/xxx.local.err'. 
> .. SUCCESS!
6. 再次尝试登入 ''' mysql -u root -p '''
> 登录成功
7. 输入 quit
> 提示Bye，成功登出

Mac中如何kill mysqld进程
1. ''' ps aux | grep mysqld ''' 找到进程
2. ''' sudo kill -9 pid1 pid2''' kill进程


##  初识数据库
1.1 

CREATE TABLE Addressbook
(regist_id    INTEGER      NOT NULL,
name          VARCHAR(128) NOT NULL,
address       VARCHAR(256)  NOT NULL,
tel_no        CHAR(10),
mail_address  CHAR(20),
PRIMARY KEY (regist_id)); 


1.2

ALTER TABLE Addressbook ADD COLUMN  postal_code CHAR(8) NOT NULL;

1.3
drop table Addressbook;

1.4
可以
使用CREAT语句重新创建表，再用INSERT语句插入之前的记录。

