## Nginx

1. **conf**文件夹为配置文件夹

   ![image-20191112103720657](../../img/image-20191112103720657.png)

2. .conf 为服务器配置文件

   * worker_processs 1  ; 

     ~~~ yml
     # 阻塞和非阻塞网络模型
     # 同步阻塞模型，一请求一进(线)程,当进（线）程增加到一定程度后
     # 更多cpu时间浪费到切换一，性能急剧下降，所以负载率不高
     # Nginx 基于时间的非阻塞多路复用(epoll或kquene)模型
     # 一个进程在短时间内可以响应大量的请求
     # 建议值 <= cpu核心数量,一般高于cpu数量不会带好处，也许还有进程切换开销的负面影响
     worker_processes 4 ;
     ~~~

   * events

     ~~~ yaml
      events{
         ## nginx最大的负载量
         ## 并发响应的关键配置值
         ## 每个进程允许的最大同时连接数 work_connections*worker_processes=maxConnection;
         ## 因为一般一个浏览器会同时开两条连接，如果反向代理，nginx到后端服务器的连接也要占用连接数
         #  所以 做静态服务器时，一般 maxClient = work_connections*worker_processes/2
         #  做反向代理时 maxClient = work_connections*worker_processes/4 
         ## 这个值理论上越大越号，但是最多可承受多少请求与配件和网络相关 ，也可最大可打开文件 ，最大可用sockets数量
         worker_connections 1024 ;
         ## 指明使用epoll 或者 kquene（*BSD)
         use epoll ;
         # 备注  要达到超高负载下最好的网络响应能力，还有必要优化与网络相关的liunx内核参数
         multi_accept on; ### 设置一个进程是否同时接受多个网络连接 默认 off
     }
     ~~~
   
* http
  
     ~~~ yaml
      access_log   xxxx   ## 访问日志
     error_log    xxxx   ## 异常日志 
     crit         xxxx   ## 错误级别
     http{
        	include mime.types      ## 引入外部配置文件
     	default_type application/octet-stream ;  ## 默认接收格式
         ## 启用内核复制模式  保持开启达到最快IO效率
         sendfile  on;  
         
        upstream mysvr{
        		server 127.0.0.1:8080 ;
        		server 127.0.0.1:3000 ; ## 热备
        }
        error_page 404 https://xxxx.com ; ## 错误页
        	## 启动时 会在数据包达到一定的大小后再发送数据
         ## 减少网络通信次数 降低阻塞概率 但是会影响响应的及时性
         ## 比较适合文件下载这类大数据包通信场景
         # tcp_nopush on ;
         # tcp_nodelay on|off   on禁用Nagle算法
         ## HTTP1.1 支持持久连接 alive
         keepalive_timeout 30s;
         ## 开启压缩
         gzip on;
         gzip_min_length 100 ;
      gzip_comp_level 4 ;
         gzip_types text/plain text/cess application/json application/x-javascript text/xml application
         include  配置文件 ## 配置文件分开设置
         
     }
   ~~~
   
   * server
   
     ~~~yaml
     server{
     	listen 80 ;        ## 监听端口   http访问端口
   	server_name xxxx.com ;   ### 指定访问 域名
     	rewrite ^(.*) https://$host$1 permanent ;   ### 指向https 操作
     }
     server{
      
         listen 443;  ## 监听端口  https访问端口
         server_name  xxxx.com;  ## 访问主机 可以是域名
      	root /home/www ;  ## 目录   
         access_log  /home/log/nginx/access_log combined; ### 日志位置
         index  index.html index.htm index.php ;   ### 指定默认访问文件
         
         ssl on; ## 开启 ssl验证
         ssl_certificate /certifier/ssl/xxxx.pem ;  ## pem格式证书
         ssl_certificate_key /certifier/ssl/xxxx.key; ## key格式证书
         ssl_session_timeout 5m ;  ## 超时设置
        	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
         ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
         ssl_prefer_server_ciphers on;
         ## 执行路径 
         location / {  ## 默认路径 如：https://xxxx.com/
          	root html ;  ## 指定目录
         }
         
         location /list{   ## 访问如： https://xxxx.com/list  进入
         	root html
         }
         
         location /proxy{    ## 访问如: https://xxxx.com/proxy  进入
         	proxy_pass http://mysvr/proxy;  ## 转发到访问:  http:mysvr/proxy => http://127.0.0.1:8080/proxy 或者 http://127.0.0.1:3000/proxy 中
         	# 可以更改或添加客户端的请求头部信息内容并转发至后端服务器，比如在后端服务器想要获取客户端的真实IP的时 候，就要更改每一个报文的头部，如下：
         	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         	deny 127.0.0.1; ## 拒绝指定ip访问
         	allow 192.168.0.116 ; ## 允许指定ip访问
         }
         location ~ \.php$ {   ### 针对php的访问
         #    root           html;
         #    fastcgi_pass   127.0.0.1:9000;
         #    fastcgi_index  index.php;
         #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
         #    include        fastcgi_params;
         }
         
     }
     ~~~
     
     