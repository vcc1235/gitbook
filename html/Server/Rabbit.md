# RabbitMQ 安装  (erlang作为支撑)

### RabbitMQ 下载地址

 https://www.rabbitmq.com/releases/rabbitmq-server/ 

- 选择  rabbitmq-server-generic 版本
- 由于 liunx版本安装包末尾为xz结尾 需执行  xz -d  rabbitmq-server-generic.**.xz 
- 解压 rabbitmq-server-generic.**.tar  
- 设置环境变量  /etc/profile  ==>  PATH=$PATH:<rabbitmq 路途径>/sbin

### Erlang 下载地址

 http://www.erlang.org/downloads 

- 选择**20** 左右版本即可
- 解压安装包  进入安装包
- 创建安装目录  /usr/local/erlang
- 执行   ./configure --prefix=/usr/local/erlang   
- 执行  make && make install
- 设置环境变量  /etc/profile ==>  PATH=/usr/local/erlang/bin:$PATH
- 使环境变量生效  source /etc/profile

### RabbitMQ操作

| select         | option                                        |
| -------------- | --------------------------------------------- |
| 启动           | rabbitmq-server  -detached                    |
| 查看状态       | rabbitmqctl  status                           |
| 关闭服务       | rabbitmqctl stop                              |
| 启动插件       | rabbitmq-plugins enable rabbitmq_managerment  |
| 查看mq用户     | rabbitmqctl list_users                        |
| 查看用户权限   | rabbitmqctl list_user_permissions guest       |
| 新增用户       | rabbitmqctl add_user admin 123456             |
| 赋予管理员权限 | rabbitmqctl set_user_tags admin administrator |


