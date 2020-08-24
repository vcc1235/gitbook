#### Docker

- Dockerfile      镜像配置文件

  ~~~ sh
  FROM centos:7
  WORKDIR /usr
  RUN mkdir /usr/local/java 
  ADD jdk /usr/local/java 
  ENV JAVA_HOME /usr/local/java
  ENV JRE_HOME ${JAVA_HOME}/jre
  ENV CLASSPATH .:${JAVA_HOME}/lib:${JRE_HOME}/lib
  ENV PATH ${JAVA_HOME}/bin:$PATH
  ~~~

  ~~~ sh
  ## 生成镜像文件
  docker build -t jdk:8 .
  ~~~

- 创建容器

  ~~~ sh
  # 文件软连接
  -v 载体目录(文件):容器目录(文件)
  --name 容器名称
  -p 载体端口号:容器端口号
  ~~~

  ~~~ sh
  docker run -it --name xxx -v 宿机目录(文件):容器目录(文件) --privileged -p 宿机端口:容器端口 -d 镜像
  ~~~

  

