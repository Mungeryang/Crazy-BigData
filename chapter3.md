## Linux集群部署

集群部署的关键是三台虚拟机的安装与配置，为后序Hadoop开发搭建一个基本的开发环境。

**JDK部署**

编写shell脚本实现jdk一键安装部署

jdk.install.sh

```shell
#!/bin/bash
#提示安装
echo "开始安装jdk"
sleep 1
#删除系统自带自带的jdk
oldjava=`rpm -qa | grep java`
for old in ${oldjava}
do 
	#echo $old
	rpm -e --nodesps $old
done
#创建安装目录
java_path=$(/export/server)
if[ ! -d $java_path ]
then
	mkdir -p $java_path
fi
#解压jdk安装包
tar -zxvf jdk-8u241-linux-x64.tar.gz -C $java_path
#设置java环境
JAVA_HOME="/export/server/jdk1.8.0_241"
grep "JAVA_HOME" /etc.profile
if[ $? -ne 0 ]
then
	#JAVA_HOME
	echo "-------------java_home----------------"
	echo 'export JAVA_HOME=/export/server/jdk1.8.0_241' >> /etc/profile
	#PATH
	echo "-------------java_path----------------"
	echo 'export PATH=:$JAVA_HOME/bin:$PATH' >> /etc/profile
fi
#加载环境变量
sleep 1
source /etc/profile
#提示安装成功
echo "java安装成功!"
java -version
```

**mysql部署**

```shell
#解压mysql安装包
cd /export/server
tar -zxvf mysql-5.7.44-linux-glibc2.12-x86_64.tar.gz -C /export/server/
#重命名
cd /export/server/
cp mysql-5.7.44-linux-glibc2.12-x86_64 mysql-5.7.44

#添加用户组
[root@bogon server]# groupadd mysql
[root@bogon server]# useradd -r -g mysql mysql

#修改目录权限
[root@bogon server]# chown -R mysql:mysql /export/server/mysql-5.7.44/

#配置mysql服务
[root@bogon server]# cp /export/server/mysql-5.7.44/support-files/mysql.server /etc/init.d/mysql

#修改mysql配置文件
[root@bogon server]# vim /etc/init.d/mysql
basedir=/export/server/mysql-5.7.44
datadir=/export/server/mysql-5.7.44/data
#修改my.cnf配置文件
[root@bogon server]# vim /etc/my.cnf
[client]
        port=3306
        default-character-set=utf8
        [mysqld]
        basedir=/export/server/mysql-5.7.44
        datadir=/export/server/mysql-5.7.44/data
        port=3306
        character-set-server=utf8
default_storage_engine=InnoDB

#初始化mysql
[root@bogon server]# /export/server/mysql-5.7.44/bin/mysqld --defaults-file=/etc/my.cnf --initialize --user=mysql --basedir=/export/server/mysql-5.7.44 --datadir=/export/server/mysql-5.7.44/data
--------
2024-01-11T02:01:28.956628Z 1 [Note] A temporary password is generated for root@localhost: ejWq+6JpQy*x
--------
#启动mysql服务
[root@bogon server]# service mysql start
#登录mysql
[root@bogon server]# /export/server/mysql-5.7.44/bin/mysql -uroot -p
Enter password:ejWq+6JpQy*x
#修改密码并配置开启远程访问权限
mysql> set password=password('123456');
mysql> GRANT ALL PRIVILEGES ON *.*  TO 'root'@'%' IDENTIFIED BY '123456';
mysql> flush privileges;
mysql> exit

#配置mysql环境变量
[root@bogon server]# vim /etc/proflie
export MYSQL_HOME=/export/server/mysql-5.7.44
export PATH=$PATH:$MYSQL_HOME/bin

#启动mysql服务并开启开机自动登录
[root@bogon etc]# source profile
[root@bogon etc]# chkconfig --add mysql
[root@bogon etc]# chkconfig mysql on

#关闭防火墙
[root@bogon etc]# systemctl stop firewalld.service
[root@bogon etc]# systemctl disable firewalld.service

#修改selinux权限
[root@bogon etc]# cd /etc/selinux/
[root@bogon selinux]# vim config
SELINUX=disabled

#mysql服务配置完成！！
```

### 服务器准备

### 三台虚拟机创建

### 设置三台虚拟机内存

### 配置MAC地址

node1：00:0C:29:4A:08:6B

node2：00:0C:29:BE:A5:16

node3：00:0C:29:4A:08:6B

### 配置IP地址

```shell
[root@localhost /]# cd etc/sysconfig/network-scripts/
[root@localhost network-scripts]# vim ifcfg-ens32 /ens33
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens32"
UUID="933b55d4-3ec2-4470-be03-07b3d44d5f2b"
DEVICE="ens32"
ONBOOT="yes"
IPADDR="192.168.88.162" #集群项目只修改ipaddr即可
GATEWAY="192.168.88.2"
NETMASK="255.255.255.0"
DNS1="8.8.8.8"
DNS2="114.114.114.114"
IPV6_PRIVACY="no"
[root@localhost network-scripts]# systemctl restart network
```



### CRT连接三台虚拟机

### 设置主机名和域名映射

```shell
[root@node1 ~]# vim /etc/hostname
  node1
  node2
  node3
[root@node1 ~]# vim /etc/hosts
192.168.88.161 node1 node1.itcast.cn
192.168.88.162 node2 node2.itcast.cn
192.168.88.163 node3 node3.itcast.cn

```

### 关闭三台虚拟机防火墙和selinux

```shell
systemctl stop firewalld.service
systemctl disable firewalld.service
```



### 三台虚拟机设置免密登录

明文与密文

#### 公钥和私钥

公钥用来加密，私钥用来解密

~~~shell
ssh-keygen -r rsa
ssh-copy-id
~~~



### 三台虚拟机时钟同步

~~~shell
crontab -e
*/1 * * * * /usr/sbin/ntpdate_ntp4.aliyun.com;

~~~

### 三台虚拟机JDK环境配置