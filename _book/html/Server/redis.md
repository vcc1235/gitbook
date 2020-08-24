## Redis 

#### 文件

- bin 文件

  ~~~ shell
  redis-benchmark    ## 用于redis 性能测试
  redis-check-aof    ## 用于redis 修复工作
  redis-check-rdb    ## 用于redis 数据修复
  redis-cli          ## 模拟redis客户端，进行连接redis,操作redis
  redis-sentinel     ## redis的哨兵模式，用于解决redis高可用性解决方案
  redis-server       ## redis 服务  启动
  ~~~

- etc 

  ~~~ shell
  redis.conf      ## redis 配置文件
  sentinel.conf    ## 哨兵模式配置文件
  ~~~

- var

  ~~~ shell
  dump.rdb   ## redis数据库配置文件
  redis.log  ## redis 日志文件
  ~~~

#### 操作

- 启动

  ~~~ shell
  ### 全局启动
  service redis-server start
  ### 自定义启动
  redis-server 指定配置文件 --port 端口号 
  ~~~

#### 配置文件

- redis.conf

  ~~~ shell
  bind 127.0.0.1  ## 绑定可访问主机地址 例如：0.0.0.0 指的所有主机都可以访问
  daemonize  yes ## redis默认不是守护进程方式运行  修改配置启动守护进程
  pidfile  /usr/local/redis/redis.pid  ### 指定pid文件路径
  port 6379  ### 指定redis启动端口号
  timeout 300 ## 客户端闲置多长时间关闭连接 如果为0 则永不关闭
  tcp-keeplive 0 ## 检测客户端网络中断时间间隔 如果为0 则永不检测
  loglevel verbose ## 指定日志记录级别  debug verbose notice warning
  logfile  stdout  ## 日志记录方式 默认标准输出
  databases 16 ##设置数据库的数量 默认值16 
  save 300 10 ## 表示300秒内有10个更改就将数据同步到数据文件
   ##指定存储至本地数据库时是否压缩数据，默认为yes，redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变得巨大：
   rdbcompassion yes
   ## 指定本地数据库文件名，默认值为dump.rdb：
   dbfilename dump.rdb
   ## 指定本地数据库存放目录
   dir ./
   ## 设置当本机为slave服务时，设置master服务的IP地址及端口，在redis启动时，它会自动从master进行数据同步
   slaveof <masterip><masterport>
   ## 当master服务设置了密码保护时，slave服务连接master的密码
   masterauth <master-password>
   ## 设置redis连接密码，如果配置了连接密码，客户端在连接redis时需要通过auth <password>命令提供密码，默认关闭
   requirepass 123456
   ## 设置同一时间最大客户端连接数，默认无限制即0
   maxclients 128
   ## 指定redis最大内存限制，redis在启动时会把数据加载到内存中，达到最大内存后，redis会先尝试清除已到期或即将到期的key，当次方法处理后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制， 会把key存放内存，value会存放在swap区
   maxmemory <bytes>
   ## 设置缓存过期策略
   maxmemory-policy noeviction
   ## 指定是否在每次更新操作后进行日志记录，redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内数据丢失。因为redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内置存在于内存中。默认为no
   appendonly no
   ## 指定更新日志文件名，默认为appendonly.aof
   appendfilename appendonly.aof
   ## 指定更新日志条件 no:表示等操作系统进行数据缓存同步到磁盘（快) always: 表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全） everysec:表示每秒同步一次（折中，默认值）
   appendfsync everysec
   
   ## 指定是否启用虚拟内存机制，默认值为no，简单介绍一下，VM机制将数据分页存放，由redis将访问量较小的页即冷数据 swap到磁盘上，访问多的页面由磁盘自动换出到内存中
   vm-enabled no
   ## 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个redis实例共享
   vm-swap-file /tmp/redis.swap
   ## 将所有大于vm-max-memory的数据存入虚拟内存，无论vm-max-memory设置多小，所有索引数据都是内存存储的（redis的索引数据就是keys），也就是说，当vm-max-memory设置为0的时候，其实是所有value都存在于磁盘。默认值为 0
   vm-max-memory 0
   ## redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是根据存储的数据大小来设定的，作者建议如果储存很多小对象，page大小最好设置为32或者64bytes；如果存储很多大对象，则可以使用更大的page，如果不确定，就使用默认值
   vm-page-size 32
   ## 设置swap文件中page数量，由于页表（一种表示页面空闲或使用的bitmap）是放在内存中的，在磁盘上每8个pages将消耗1byte的内存
   vm-pages 134217728
   ## 设置访问swap文件的线程数，最好不要超过机器的核数，如果设置为0，那么所有对swap文件的操作都是串行的，可能会造成长时间的延迟。默认值为4
   vm-max-threads 4
   ## 设置在客户端应答时，是否把较小的包含并为一个包发送，默认为开启
   glueoutputbuf yes
   ## 指定在超过一定数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
   hash-max-zipmap-entries 64
   hash-max-zipmap-value 512
   ## 指定是否激活重置hash，默认开启
   activerehashing yes
   ## 指定包含其他配置文件，可以在同一主机上多个redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
   include /path/to/local.conf
  ~~~

  