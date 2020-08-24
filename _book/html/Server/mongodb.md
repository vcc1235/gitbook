## Mongodb 安装

- 下载地址  https://www.mongodb.com/download-center/community 

- 选择对应的版本 获取对应版本下载地址 例如：
  
  - https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.6.16.tg
  
- 解压 文件  移动到指定的位置  例如： /usr/local/mongodb

- 创建文件夹   

  ~~~ shell
  ##  创建数据库存放文件夹
  mkdir /usr/local/mongodb/data  
  ##  创建配置文件夹
  mkdir /usr/local/mongodb/etc
  ##  创建日志文件夹
  mkdir /usr/local/mongodb/log
  ~~~

- 设置配置

  ~~~ shell
  ### 进入配置文件夹
  cd /usr/local/mongodb/etc  
  ### 创建配置文件夹
  vim mongodb.conf
  ### 配置文件内容
  
  ## 数据库
  dbpath=/usr/local/mongodb/data
  ## 日志
  logpath=/usr/local/mongodb/log/mongodb.log
  ## 端口号
  port= 27017
  ## 绑定ip访问
  #bind_ip=127.0.0.1  只允许本机访问
  bind_ip=0.0.0.0   #所有主机都可以访问
  # 权限
  auth=false   ## 关闭认证
  ~~~

  