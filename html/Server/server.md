## Linux Centos 常用环境安装

#### 引用来源

- 来源:https://oneinstack.com/ 
- 默认安装路径都在  **/usr/local/**

#### 服务

- Liunx+Nginx+MySQL/MongoDB+PHP
- Liunx+Apache+MySQL/MongoDB+PHP
-  Linux + Nginx+ MySQL/MongoDB+ PHP+ Apache 
-  Linux + Nginx+ MySQL/MongoDB+ Tomcat 
-  Linux + Nginx+ PostgreSQL+ PHP 
-  Linux + Apache+ PostgreSQL+ PHP 
-  Linux + Nginx+ MySQL+ HHVM 

~~~ shell
##  Linux + Nginx+ MySQL+ Tomcat+Redis   安装命令
wget -c http://mirrors.linuxeye.com/oneinstack-full.tar.gz && tar xzf oneinstack-full.tar.gz && ./oneinstack/install.sh --nginx_option 1 --tomcat_option 2 --jdk_option 2 --db_option 2 --dbinstallmethod 1 --dbrootpwd oneinstack --redis  --memcached  --reboot 
~~~

~~~ shell
## 也可以选择性安装 
wget -c http://mirrors.linuxeye.com/oneinstack-full.tar.gz && tar xzf oneinstack-full.tar.gz && ./oneinstack/install.sh  
## 执行完会出现对应的安装选择
~~~

