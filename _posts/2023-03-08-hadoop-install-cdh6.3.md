---
layout:     post
title:      cdh6.3.2 install 
subtitle:   with centos7.4
date:       2023-03-08
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - BigData
    - install
---


# CDH 6.3部署


## 主机规划

| 主机名 | 主机IP  | 用途 |
| --- | --- | --- |
|dms210|172.19.3.210|集群管理节点|
|dms214|172.19.3.xxx|dn节点+nn节点|
|dms215|172.19.3.xxx|dn节点+nn节点|
|dms216|172.19.3.xxx|dn节点|
|dms217|172.19.3.xxx|dn节点|
|dms218|172.19.3.xxx|dn节点|

## 1.主机命名、hosts配置(除了root免密登录，其它全程root用户操作)

登录到各个主机上修改主机名,必须是完整的FQDN，如： dms210.cmiot.com，不能是

```bash
hostnamectl set-hostname dms210.cmiot.com
```
修改网络配置文件/etc/sysconfig/network种的HOSTNAME为当前主机名：

```
HOSTNAME=dms210.cmiot.com
```
修改系统配置文件: /etc/hosts
```bash
127.0.0.1       localhost.localdomain localhost localhost4 localhost4.localdomain4

::1     localhost.localdomain localhost localhost6 localhost6.localdomain6

172.19.3.210 dms210.cmiot.com dms210

172.19.3.xxx dms214.cmiot.com dms214

172.19.3.xxx dms215.cmiot.com dms215

172.19.3.xxx dms216.cmiot.com dms216

172.19.3.xxx dms217.cmiot.com dms217

172.19.3.xxx dms218.cmiot.com dms218
```

## 2.管理节点到其它主机的ssh免密登录
登录到210和214，215主机生成ssh密钥，然后分发到集群所有主机上，包括这3台主机本身。
### app用户的免密登录
可以直接分发
生成密钥：
```bash
ssh-keygen -t rsa -P ''
```

分发公钥：
```bash
ssh-copy-id -i .ssh/id_rsa.pub app@dms210
ssh-copy-id -i .ssh/id_rsa.pub app@dms214
ssh-copy-id -i .ssh/id_rsa.pub app@dms215
ssh-copy-id -i .ssh/id_rsa.pub app@dms216
ssh-copy-id -i .ssh/id_rsa.pub app@dms217
ssh-copy-id -i .ssh/id_rsa.pub app@dms218
```
安装过程如果使用非root用户，需要该用户具有免密执行sudo的权限，还需要脚本中执行sudo命令的权限，坑比较多。条件允许尽量使用root用户部署。
### root用户免密登录

如果默认配置为禁止root远程登录，只能普通用户登录到主机然后切换到root，如果需要远程登录,需要修改所有机器的ssh配置
```bash
sudo vi /etc/ssh/sshd_config
```
找到
```bash
Banner /etc/sshbanner
LogLevel INFO
MaxAuthTries 6
PermitRootLogin no
PubkeyAuthentication yes
PermitEmptyPasswords no
StrictModes yes
IgnoreUserKnownHosts yes
```
将'PermitRootLogin no'修改为'PermitRootLogin yes'，重启sshd服务：
```bash
sudo systemctl restart sshd
```
210,214,215机器上执行sudo命令切换到root用户
生成密钥：
```bash
sudo su -
ssh-keygen -t rsa -P ''
cp .ssh/id_rsa.pub /home/app
chmod 777 /home/app/id_rsa.pub 
```
app用户210上合并公钥：

```bash
scp dms214:~/id_rsa.pub ./id_rsa.pub.214
scp dms215:~/id_rsa.pub ./id_rsa.pub.215
cat id_rsa.pub.214>>id_rsa.pub
cat id_rsa.pub.215>>id_rsa.pub
rm id_rsa.pub.214 id_rsa.pub.215
```
app用户分发公钥：
```bash
scp id_rsa.pub app@dms210:~/
scp id_rsa.pub app@dms211:~/
scp id_rsa.pub app@dms212:~/
scp id_rsa.pub app@dms213:~/
scp id_rsa.pub app@dms214:~/
scp id_rsa.pub app@dms215:~/
scp id_rsa.pub app@dms216:~/
scp id_rsa.pub app@dms217:~/
scp id_rsa.pub app@dms218:~/
```
登录各机器，切换到root用户复制公钥文件到.ssh目录重命名为authorized_keys，没有.ssh目录的，创建该目录，确保.ssh目录权限为700，authorized_keys文件权限为600

```bash
sudo su -
cp /home/app/id_rsa.pub .ssh/authorized_keys
chmod 700 .ssh
chmod 600 .ssh/authorized_keys
rm -f  /home/app/id_rsa.pub
```
注意：配置完成后这3台机器到全集群的免密ssh都测试一遍，消除第一次登录的提示。否则程序第一次登录会报超时错误，因为程序通常不会处理交互提示，只会一直等待直到超时。

## 3.关闭selinux和巨透内存页

## 4.配置时间同步服务

查看ntp服务状态：
```bash
sudo systemctl status  ntpd
```
如果没有安装时间同步服务，先安装

```bash
sudo yum install -y  ntp
```
修改ntp服务的配置/etc/ntp.conf ，选定一台服务器如210机器从中国区时间服务器同步，其它机器从210同步
210服务器：
```bash
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
server 0.cn.pool.ntp.org
server 1.cn.pool.ntp.org
server 2.cn.pool.ntp.org
server 3.cn.pool.ntp.org
```
其它服务器：
```bash
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
server 172.19.3.210
```
重启ntp服务,并设置为开机启动
```bash
sudo systemctl restart ntpd
sudo systemctl enable ntpd
```
## 安装jdk
我们使用linux版64位java se，1.8版本的最新发布版jdk1.8.0_271,必须要安装到/usr/java/jdk-version 目录下，可以安装好一台后，用root帐户免密分发到其他机器
```bash
tar -xzf jdk-8u271-linux-x64.tar.gz
sudo su -
cd /usr
mkdir java
cd java
cp /home/app/jdk1.8.0_271 ./
cd ..
chmod 755 -R java
scp -r java dms211:/usr/
scp -r java dms215:/usr/
scp -r java dms216:/usr/
scp -r java dms217:/usr/
scp -r java dms218:/usr/
```
登录每台机器，配置全局环境变量/etc/profile：
```bash
export JAVA_HOME=/usr/java/jdk1.8.0_271
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
```
清理210上的安装包
```bash
rm -f /home/app/jdk-8u271-linux-x64.tar.gz
rm -rf /home/app/jdk1.8.0_271
```
## 6.安装mysql作为大数据集群保存元数据的数据库

编辑/etc/my.cnf:
```bash
datadir=/data/mysqldb
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
lower_case_table_names=1
max_connections=2000
character_set_server=utf8mb4
default_authentication_plugin=mysql_native_password
explicit_defaults_for_timestamp=0
transaction-isolation = READ-COMMITTED
symbolic-links = 0
key_buffer_size = 32M
max_allowed_packet = 16M
thread_stack = 256K
thread_cache_size = 64
read_rnd_buffer_size = 16M
sort_buffer_size = 8M
join_buffer_size = 8M
server_id=1
binlog_format = mixed
read_buffer_size = 2M
read_rnd_buffer_size = 16M
sort_buffer_size = 8M
join_buffer_size = 8M

# InnoDB settings
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 64M
innodb_buffer_pool_size = 4G
innodb_thread_concurrency = 8
innodb_flush_method = O_DIRECT
innodb_log_file_size = 512M

sql_mode=STRICT_ALL_TABLES
```
重启mysql服务：
```bash
sudo systemctl restart mysqld
```
启动mysql_secure_installation修改root密码：

```bash
sudo /usr/bin/mysql_secure_installation
```
## 7.安装mysql的jdbc驱动，cloudary推荐使用5.1版本
从mysql官网下载mysql的java驱动，解压改名后考到/usr/share/java目录，必须是这个目录，文件名必须是mysql-connector-java.jar
```bash
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.49.tar.gz
tar zxvf mysql-connector-java-5.1.49.tar.gz
mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.49
cp mysql-connector-java-5.1.49-bin.jar /usr/share/java/mysql-connector-java.jar
```
清理210上的安装包
```bash
rm -f /home/app/mysql-connector-java-5.1.49.tar.gz
rm -rf /home/app/mysql-connector-java-5.1.49
```
保险起见，分发到所有机器，需要访问mysql的组件都需要这个驱动，根据组件在集群中机器的分布来分发，容易出错,会导致组件初始化时出错，从而引起一系列问题
```bash
scp -r /usr/share/java dms211:/usr/share/
scp -r /usr/share/java dms212:/usr/share/
scp -r /usr/share/java dms213:/usr/share/
scp -r /usr/share/java dms214:/usr/share/
scp -r /usr/share/java dms215:/usr/share/
scp -r /usr/share/java dms216:/usr/share/
scp -r /usr/share/java dms217:/usr/share/
scp -r /usr/share/java dms218:/usr/share/
```
## 创建database，不需要的组件可以不用建database和相应的用户
root用户登录mysql
```bash
mysql -u root -p
```
创建各组件所需database及用户：
```bash
CREATE DATABASE  <database> DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci
CREATE USER '<user>'@'%' IDENTIFIED BY 'xxx';
GRANT ALL ON <database>.* TO '<user>'@'%';
```
| service | Database | User |
| -- | -- | -- |
|Cloudera Manager Server|scm|scm|
|Activity Monitor|amon|amon|
|Reports Manager|rman|rman|
|Hue|hue|hue|
|Hive Metastore Server|metastore|hive|
|Sentry Server|sentry|sentry|
|Cloudera Navigator Audit Server|nav|nav|
|Cloudera Navigator Metadata Server|navms|navms|
|Oozie|oozie|oozie|
```bash
 CREATE DATABASE  scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
 CREATE DATABASE  amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
 CREATE DATABASE  rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
 CREATE DATABASE  hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
 CREATE DATABASE  metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
 CREATE DATABASE  sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
 CREATE DATABASE  nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
 CREATE DATABASE  navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
 CREATE DATABASE  oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE USER 'scm'@'%' IDENTIFIED BY 'xxx';
CREATE USER 'amon'@'%' IDENTIFIED BY 'xxx';
CREATE USER 'rman'@'%' IDENTIFIED BY 'xxx';
CREATE USER 'hue'@'%' IDENTIFIED BY 'xxx';
CREATE USER 'hive'@'%' IDENTIFIED BY 'xxx';
CREATE USER 'sentry'@'%' IDENTIFIED BY 'xxx';
CREATE USER 'nav'@'%' IDENTIFIED BY 'xxx';
CREATE USER 'navms'@'%' IDENTIFIED BY 'xxx';
CREATE USER 'oozie'@'%' IDENTIFIED BY 'xxx';
GRANT ALL ON scm.* TO 'scm'@'%';
GRANT ALL ON amon.* TO 'amon'@'%';
GRANT ALL ON rman.* TO 'rman'@'%';
GRANT ALL ON hue.* TO 'hue'@'%';
GRANT ALL ON metastore.* TO 'hive'@'%';
GRANT ALL ON sentry.* TO 'sentry'@'%';
GRANT ALL ON nav.* TO 'nav'@'%';
GRANT ALL ON navms.* TO 'navms'@'%';
GRANT ALL ON oozie.* TO 'oozie'@'%';

其中xxx替换为自己的密码
```
## 7.配置cm本地yum库
官方文档地址：https://docs.cloudera.com/documentation/enterprise/upgrade/topics/cm_ig_create_local_package_repo.html
在210上安装apache服务器和createrepo：
```bash
sudo yum -y install httpd createrepo
sudo systemctl start httpd
```
下载cm安装包，构建repository仓库
```bash
sudo su -
cd /var/www/html/
mkdir cloudera-repos
cd cloudera-repos
wget https://archive.cloudera.com/cm6/6.3.1/allkeys.asc
wget  wget https://archive.cloudera.com/cm6/6.3.1/redhat7/yum/RPMS/x86_64/cloudera-manager-daemons-6.3.1-1466458.el7.x86_64.rpm
wget https://archive.cloudera.com/cm6/6.3.1/redhat7/yum/RPMS/x86_64/cloudera-manager-server-6.3.1-1466458.el7.x86_64.rpm
wget https://archive.cloudera.com/cm6/6.3.1/redhat7/yum/RPMS/x86_64/cloudera-manager-server-db-2-6.3.1-1466458.el7.x86_64.rpm
wget https://archive.cloudera.com/cm6/6.3.1/redhat7/yum/RPMS/x86_64/cloudera-manager-agent-6.3.1-1466458.el7.x86_64.rpm
wget https://archive.cloudera.com/cm6/6.3.1/redhat7/yum/RPM-GPG-KEY-cloudera
createrepo .
cd ..
chmod 755 -R cloudera-repos
```
完成以上步骤，在浏览器中输入http://<server_host>/cloudera-repos应该能看见文件列表<br/>
导入GPG key（如果没有这步操作，很可能cloudera服务安装失败）210节点：
```bash
sudo rpm --import http://172.19.3.210/cloudera-repos/RPM-GPG-KEY-cloudera
```
在所有机器上添加210上刚配置的yum源：
```bash
sudo vi /etc/yum.repos.d/cloudera-manager.repo
```
添加如下内容：
```
[cloudera-manager]
name=Cloudera Manager 6.3.1
baseurl=http://172.19.3.210/cloudera-repos/
gpgkey=http://172.19.3.210/cloudera-repos/RPM-GPG-KEY-cloudera
gpgcheck=1
```
刷新缓存：
```bash
sudo yum clean all
sudo yum makecache
```
## 7.安装cm（root）
```bash
 yum install cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server
```
安装完CM后/opt/ 下会出现cloudera目录，将之前下载的parcel文件移动到/opt/cloudera/parcel-repo目录
```bash
 mv /home/app/CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel*  /opt/cloudera/parcel-repo
 mv /home/app/manifest.json /opt/cloudera/parcel-repo
```
生成sha校验码（本步骤不确定是否必须，但网上离线部署手册上都有这一步）：
```bash
cd /opt/cloudera/parcel-repo
 sha1sum CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel| awk '{ print $1 }' >CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha
```
调整权限：
```bash
cd /opt/cloudera
chown cloudera-scm:cloudera-scm -R parcel-repo
chmod 755 -R parcel-repo
```
最终文件如下：
```bash
-rwxr-xr-x 1 cloudera-scm cloudera-scm 2082186246 Nov 12  2019 CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel
-rwxr-xr-x 1 cloudera-scm cloudera-scm         41 Nov  9 11:04 CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha
-rwxr-xr-x 1 cloudera-scm cloudera-scm         40 Nov 12  2019 CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha1
-rwxr-xr-x 1 cloudera-scm cloudera-scm         64 Nov 12  2019 CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha256
-rwxr-xr-x 1 cloudera-scm cloudera-scm      33887 Nov 12  2019 manifest.json
```
配置cm的数据库' /opt/cloudera/cm/schema/scm_prepare_database.sh <databaseType> <databaseName><databaseUser>'
```bash
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql scm scm
```
输入密码后会自动初始化cm的元数据库
启动cm：
```bash
sudo systemctl start cloudera-scm-server
sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
```
当看见日志中出现'INFO WebServerImpl:com.cloudera.server.cmf.WebServerImpl: Started Jetty server.'时，说明cm启动完成。可以使用浏览器访问http://<server_host>:7180来打开cm的图形界面
初始用户名密码为admin/admin，然后一直点击继续，直到版本选择，选企业版试用60天，到期不上传许可证会降为免费版，不影响数据和使用。
点继续进入cdh安装向导欢迎页，点继续，输入集群名字dms，点继续，进入主机选择，输入dms[210-218].cmiot.com，点击搜索会显示dms210-dms2189台机器，选中除211-213之外的6台机器，
http://172.19.3.210:7180/cmf/express-wizard/wizard#




##踩坑
centOS7.2 安装时碰见一个问题，前面步骤都正确返回，但是启动cm报错,没有日志文件：
```bash
[root@dms120 cloudera-scm-server]<20210105 19:55:27># systemctl status cloudera-scm-server                         
● cloudera-scm-server.service - Cloudera CM Server Service
   Loaded: loaded (/usr/lib/systemd/system/cloudera-scm-server.service; enabled; vendor preset: disabled)
   Active: failed (Result: start-limit) since Tue 2021-01-05 19:55:21 CST; 12s ago
  Process: 54703 ExecStart=/opt/cloudera/cm/bin/cm-server (code=exited, status=1/FAILURE)
  Process: 54700 ExecStartPre=/opt/cloudera/cm/bin/cm-server-pre (code=exited, status=0/SUCCESS)
 Main PID: 54703 (code=exited, status=1/FAILURE)

Jan 05 19:55:21 dms120.cmiot.com systemd[1]: cloudera-scm-server.service: main process exited, code=exited, status=1/FAILURE
Jan 05 19:55:21 dms120.cmiot.com systemd[1]: Unit cloudera-scm-server.service entered failed state.
Jan 05 19:55:21 dms120.cmiot.com systemd[1]: cloudera-scm-server.service failed.
Jan 05 19:55:21 dms120.cmiot.com systemd[1]: cloudera-scm-server.service holdoff time over, scheduling restart.
Jan 05 19:55:21 dms120.cmiot.com systemd[1]: Stopped Cloudera CM Server Service.
Jan 05 19:55:21 dms120.cmiot.com systemd[1]: start request repeated too quickly for cloudera-scm-server.service
Jan 05 19:55:21 dms120.cmiot.com systemd[1]: Failed to start Cloudera CM Server Service.
Jan 05 19:55:21 dms120.cmiot.com systemd[1]: Unit cloudera-scm-server.service entered failed state.
Jan 05 19:55:21 dms120.cmiot.com systemd[1]: cloudera-scm-server.service failed.
```
journalctl -xe发现找不到JAVA_HOME,检查发现环境变量没有问题，时/usr/java和/usr/share/java这2个目录宿主及权限有问题，后面一个目录权限不对会导致找不到jdbc驱动
```bash
chown root:root -R /usr/java
chown root:root -R /usr/share/java
chmod 755 -R /usr/java
chmod 755 -R /usr/share/java
```
重新启动cm，一切正常
```bash
sudo systemctl start cloudera-scm-server
```













