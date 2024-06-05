1.Mysql安装

①下载并上传解压文件,安装在/application/mysql

```
tar xf mysql-5.7.14-linux-glibc2.5-86_64.tar.gz

mkdir /application

mv mysql-5.7.14-linux-glibc2.5-x86_64 /application/mysql

```

②处理原始环境

```
yum remove -y mariadb-libs

yum install -y libaio-devel
```

③设置环境变量

```
vim /etc/profile

export PATH=/application/mysql/bin:$PATH 

source /etc/profile

mysql -V
```

④创建数据路径

```
mkdir -p /data/mysql/3306
```

⑤创建用户并授权

```
useradd -s /sbin/nologin mysql

chown -R mysql.mysql /application/*

chown -R mysql.mysql /data
```

⑥初始化命令

```
mysqld --initialize-insecure --user=mysql --basedir=/application/mysql --datadir=/data/mysql/3306

mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql/3306
```

2.初始化配置文件

①.作用：

​	（1）影响数据库的启动

​	（2）影响到客户端的功能

②.初始化配置的方法

（1）初始化配置文件,例如/etc/my.cnf

（2）启动命令行上进行设置，例如:mysqld_safe和mysqld

（3）预编译时的设置，仅限于编译安装时的设置

③.初始化文件的书写格式

```
[标签]
xxx=xxx
[标签]
xxx=xxx
```

④.配置文件标签归类

服务器端:

[mysqld]

[mysqld_safe]

[server]

客户端: 

[mysql]

[mysqladmin]

[mysqldump]

[client] 

⑤.配置文件设置样板  /etc/my.cnf

```
#服务器端配置
[mysqld]
#用户
user=mysql
#软件安装目录
basedir=/application/mysql
#数据路径
datadir=/data/mysql/3306
#socket文件位置
socket=/tmp/mysql.sock
#服务器id号
server_id=6
#端口号
port=3306
#客户端配置
[mysql]
#socket文件位置
socket=/tmp/mysql.sock
```

⑥配置文件读取顺序

/etc/my.cnf 

/etc/mysql/my.cnf 

/usr/local/mysql/etc/my.cnf 

~/.my.cnf 

配置文件以最后一个为准

⑦强制使用自定义配置文件

​	--defautls-file

```
//强制读取my.test.cnf文件，其他的配置文件都不读取
[root@shell 20:42 ~]#mysqld_safe --defaults-file=/my.test.cnf
```



3.启动数据库

①mysql的启动过程

​	Ⅰ.日常启停

​	mysql.server start	 ---> 	mysqld_safe 	--->	mysqld

​	mysql.service	--->	mysqld

​	Ⅱ.维护性任务

​	mysqld_safe [临时参数] &

​	命令行内临时参数优先级最高



②各种启动方式配置

Ⅰ. sys-v

service mysqld  start启动方式配置

**多实例未验证可用性**

```
cp /application/mysql/support-files/mysql.server  /etc/init.d/mysqld 

service mysqld restart
```

Ⅱ.systemd

systemctl start mysqld启动方式配置

```
cat >/etc/systemd/system/mysqld.service <<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/application/mysql/bin/mysqld --defaults-file=/etc/my.cnf
LimitNOFILE = 5000
EOF
```

```
//在多实例中需要更改 ExecStart指向的my.cnf位置
```



4.MYSQL的连接管理

​	注意：提前应该将用户授权做好

TCP/ IP:

```
mysql> grant all on *.* to root@'192.168.137.%' identified by '123456';
Query OK, 0 rows affected, 1 warning (0.00 sec)

[root@shell 20:53 ~]#mysql -uroot -p -h 192.168.137.100 -P3306
```

Socket:

```
[root@shell 20:56 ~]#mysql -uroot -p -S /tmp/mysql.sock
```

-e 免交互执行sql语句

```
[root@shell 18:22 ~]#mysql -uroot -p -e "show databases;"
Enter password: 
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| wordpress          |
+--------------------+
```

