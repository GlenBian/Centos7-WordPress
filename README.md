# Centos7.4 64位 配置WordPress个人博客

------
> * 操作系统：Centos7.4 64位
> * 服务器：httpd
> * 数据库：MariaDB
> * 博客平台：WordPress
> * 搭载平台：腾讯云
> * 公网IP：***100.100.100.100***

------

#### 1.安装必要的软件
```
yum install httpd -y   #安装apache
yum install php -y     #安装PHP
yum install mariadb-server mariadb    #安装mariadb
```
#### 2.接下来关闭防火墙
```
systemctl stop  firewalld.service     #关闭防火墙
setenforce 0             #关闭selinux
systemctl restart httpd       #重启apache
```
> 使用`systemctl restart httpd`重启httpd服务的时候可能会输出错误，所以建议使用`ps -aux`查看httpd的进程ID，先用`kill PID`来杀死进程，再输入`httpd`打开Apache的http网络服务。

#### 3.测试PHP是否安装成功
```
vim  /var/www/html/info.php    #创建一个文件
```
输入如下内容
``` PHP
<?php
phpinfo();
?>
```
> 然后在任意一个浏览器中输入ip地址***100.100.100.100/info.php***，打开界面如果有PHP信息出现，那就是安装成功了。

![此处输入图片的描述][1]


#### 4.配置mariadb
> mariadb是mysql的一部分，由于mysql存在不开源的风险，所有这里使用MariaDB而不是MySQL；

这里先要启动Mysql服务
```
systemctl start mariadb.service
mysql_secure_installation       #进入配置  
Enter current password for root (enter for none):       #输入原始root密码，空密码就回车
Set root password? [Y/n]        #是否设置root密码
New password:       #输入新密码 ***********
Re-enter new password:       #再次输入
Password updated successfully!
Reloading privilege tables..
 ... Success!
 Remove anonymous users? [Y/n]        #是否移除匿名用户
 ... Success!
Disallow root login remotely? [Y/n]       #是否禁止远程root登陆
 ... skipping.
Remove test database and access to it? [Y/n]       #是否删除测试数据库
Reload privilege tables now? [Y/n]           #重新载入
 ... Success!
```
#### 5.安装WordPress
```
yum -y install wget unzip net-tools         #安装解压工具
wget http://wordpress.org/latest.zip        #下载wordpress压缩包
unzip -q latest.zip              #解压
cp -rf wordpress/* /var/www/html/         #将wordpress目录里的文件复制到html目录里
```
修改目录权限
```
chown -R apache:apache /var/www/html/
chmod -R 755 /var/www/html/
mkdir -p /var/www/html/wp-content/uploads
chown -R :apache /var/www/html/wp-content/uploads
```
编辑配置文件,修改数据名，用户和密码，这些是之前配置数据库时创建的：
```
cd /var/www/html
cp wp-config-sample.php wp-config.php 
vim wp-config.php
```
关键内容如下
``` PHP
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', '***********');

/** MySQL hostname */
define('DB_HOST', 'localhost');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');
```

#### 6.配置数据库
进入MySQL
```
mysql -uroot --password='***********' # 上一步中输入的密码
```
创建一个名为***wordPress***的数据库
``` MySQL
CREATE DATABASE wordpress;
```
MySQL 部分设置完了，我们退出 MySQL 环境：
``` MySQL
exit
```
#### 7.进行测试
在浏览器中输入***http://100.100.100.100/wordpress/index.php***,按照步骤进行设置如图：

![此处输入图片的描述][2]


  [1]: http://122.152.223.142/myImages/1-0.png
  [2]: http://122.152.223.142/myImages/1-1.jpeg
  
#### 8.some bugs

> 在进行一些数据库的配置后，要重启服务
```
systemctl restart httpd
systemctl restart mariadb.service
```
> wordpress 打开后使用电子邮箱的时候可能会报错`可能原因：您的主机禁用了mail()函数`解决办法如下
```
yum install sendmail # 重新安装 sendmail 组件
```

作者 [@GlenBian][3]     
2018 年 08月 14日   
