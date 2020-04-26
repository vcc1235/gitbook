## Maven

#### 仓库

~~~ xml
<repositories>
   <repository>
     <id>com-vcc-mapper</id>
     <url>http://maven.xalete.com/maven</url>
   </repository>
</repositories>
~~~

- 根据sql 文件创建 dao模型  mapper.xml  service 

~~~xml
<dependency>
  <groupId>com.vcc</groupId>
  <artifactId>mapper</artifactId>
  <version>1.0.0</version>
</dependency>
~~~

~~~ java
public static void main(String[] args) {
    String path = Sql.class.getClassLoader().getResource("demo.yml").getPath();
    JavaShell javaShell = new JavaShell(path);
    javaShell.createJavaFile();
    javaShell.createMapperXML();
}
~~~

 **demo.yml**

~~~ yaml
mapper:
  sql: E:\\vcc\\mall.sql    ###  数据库地址
  dao: com.vcc.demo.jinxun.dao      ###  model 模型
  mapper: com.vcc.demo.jinxun.mapper   ###  mapper 地址
  service: com.vcc.demo.jinxun.service  # service 地址
  mapperxml: src/main/resources/mapper/   ### xml 地址
  md: src/main/resource/sql.md    ### md 说明
~~~

