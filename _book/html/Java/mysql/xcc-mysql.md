## Mysql 与 Entity

### 仓库

~~~ xml
<repositories>
   <repository>
     <id>com-vcc-mapper</id>
     <url>http://maven.xalete.com/maven</url>
   </repository>
</repositories>
~~~

##### 根据sql 文件创建 dao模型  mapper.xml  service 

~~~ xml
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
~~~ yaml
    mapper:
    sql: E:\\vcc\\mall.sql    ###  数据库地址
    dao: com.vcc.demo.jinxun.dao      ###  model 模型
    mapper: com.vcc.demo.jinxun.mapper   ###  mapper 地址
    service: com.vcc.demo.jinxun.service  # service 地址
    mapperxml: src/main/resources/mapper/   ### xml 地址
    md: src/main/resource/sql.md    ### md 说明
~~~

##### 根据model 来创建 表

~~~ xml
<dependency>
<groupId>com.xcc.imx</groupId>
<artifactId>mysql</artifactId>
<version>1.0.0</version>
</dependency>
~~~
1.  启动添加指定包名 
	~~~ java
	@SpringBootApplication(scanBasePackages = {"com.xcc.sql"})
	~~~

2. 配置 yml

   ~~~ yaml
   mybatis:
     model:
       pack: com.xcc.xos.main.entity   ## 指定model报名
     configuration:
       log-impl: org.apache.ibatis.logging.stdout.StdOutImpl  ### sql 语句打印 可不要
     config-locations: classpath*:com/xcc/sql/mapper/*.xml   ### 指定mapper.xml 
   ~~~

3. model  注解

   ~~~ java
   package com.xcc.xos.main.entity;
   import ...
   /**
    * @description: 文件模型
    * @author: 123
    * @create: 2020-08-05 16:52
    **/
   @Data
   @XTable(name = "file")  // 表名称
   public class FileEntity {
       
       // name 字段名   type  字段类型    length 字段长度    isKey 是否为主键   isNull 是否可以为空  
       // isAutoIncrement  是否自增   commonValue 注释
       @XColumn(name = "id", type = MysqlTypeConstant.INT, length = 32, isKey = true, isNull = false, isAutoIncrement = true, commonValue = "id")
       private Integer id;
       @XColumn(name = "name", type = MysqlTypeConstant.VARCHAR, isNull = false, commonValue = "文件名称")
       private String name;
       @XColumn(name = "type", type = MysqlTypeConstant.VARCHAR, isNull = false, commonValue = "文件类型")
       private String type;
       @XColumn(name = "url", type = MysqlTypeConstant.VARCHAR, commonValue = "文件url")
       private String url;
       @XColumn(name = "parent", type = MysqlTypeConstant.INT, isNull = false, commonValue = "父类id")
       private Integer parent;
       @XColumn(name = "is_file", type = MysqlTypeConstant.TINYINT, length = 1, isNull = false, commonValue = "是否是文件")
       private boolean file;
       @XColumn(name = "is_status", type = MysqlTypeConstant.TINYINT, length = 1, isNull = false, commonValue = "是否是私有文件")
       private boolean status;
       @XColumn(name = "create_time", type = MysqlTypeConstant.DATETIME, commonValue = "创建时间")
       private Date createTime;
       //  isAutoUpdateTime  时间是否自动更新
       @XColumn(name = "update_time", type = MysqlTypeConstant.DATETIME, commonValue = "更新时间", isAutoUpdateTime = true)
       private Date updateTime;
   
   }
   ~~~

4. 启动即可更具model 生成对应的表

5. 自带一些数据库操作

   ~~~ Java
   public interface IAbstractTableService<T> {
       int insertOne(T var1);
   
       int insertMany(List<T> var1);
   
       T getOne(LinkedHashMap<String, String> var1);
   
       int getCount(LinkedHashMap<String, String> var1);
   
       List<T> getList(int var1, int var2);
   
       List<T> getList(int var1, int var2, String var3);
   
       List<T> getList(LinkedHashMap<String, String> var1, int var2, int var3);
   
       List<T> getList(LinkedHashMap<String, String> var1, int var2, int var3, String var4);
   
       List<T> getList(LinkedHashMap<String, String> var1);
   
       List<T> getList(LinkedHashMap<String, String> var1, String var2);
   
       int update(T var1);
   
       int update(LinkedHashMap<String, Object> var1, LinkedHashMap<String, String> var2);
   
       int delete(LinkedHashMap<String, String> var1);
   }  // 接口类
   @Service("IAbstractTableService")
   public class AbstractTableServiceImpl<T> implements IAbstractTableService<T> {  // 实现类 ， 使用时继承即可
   	// 子类必须重写该方法  返回对应model 类 	
       public Class<T> getTypeClass() {
           return null;
       }
   
   }
   ~~~

   