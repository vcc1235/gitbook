## gitLab 的安装

- 创建或者修改 **yum**源

  ~~~ sh
  vim /etc/yum.repos.d/gitlab-ce.repo
  ~~~

- *gitlab-ce.repo*内容

  ~~~ sh
  name=Gitlab CE Repository
  baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
  gpgcheck=0
  enabled=1
  ~~~

- 更新**yum**缓存

  ~~~ sh
  yum makecache
  ~~~

- 安装 **gitlab-ce**

  ~~~sh
  yum install gitlab-ce
  ~~~

- 修改配置文件

  ~~~shell
  vim /etc/gitlab/gitlab.rb
  ~~~

- *gitlab.rb*文件

  ~~~shell
  external_url http://192.168.0.120:3005  # URL on which GitLab will be reachable
  ## 下面gitlab_rails 配置 如  gitlab['time_zone']='Beijing'
  gitlab_email_from   邮箱地址
  gitlab_email_display_name   昵称 如：GitLab
  ## SMTP 配置
  smtp_enable  true
  smtp_address  smtp服务地址 如：smtp.163.com
  smtp_port 465
  smtp_user_name  发送端邮箱地址 对应平台
  smtp_password   授权码
  smtp_domain   smtp服务地址
  smtp_authentication   login
  user['git_user_email']  与发送地址一样
  ~~~

- 重新加载配置

  ~~~shell
  gitlab-ctl reconfigure
  ~~~

- 重新启动

  ~~~shell
  gitlab-ctl restart
  ~~~

- 查看日志

  ~~~shell
  gitlab-ctl tail
  ~~~

  