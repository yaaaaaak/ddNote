#### 实现

MybatisGenerator，简单来讲，就是围绕数据库表生成最表层crud代码的工具。

一个基础实现可以很简单，新建一个maven项目（非maven自行改造），在pom.xml里配置一下数据库依赖和generator插件。

```xml

    <dependencies>
        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.33</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.2</version>
            </plugin>
        </plugins>
    </build>
```

再在项目/resources目录下创建一个generatorConfig.xml文件，往文件里写如下配置即可。配置都有相应注释，有其他需要可以去官网查阅api，这里不多赘述。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!--数据库驱动jar -->
    <classPathEntry
            location="C:\Users\Administrator\.m2\repository\mysql\mysql-connector-java\5.1.33\mysql-connector-java-5.1.33.jar" />

    <context id="Tables" targetRuntime="MyBatis3">
        <!--去除注释 -->
        <commentGenerator>
            <property name="suppressAllComments" value="true" />
        </commentGenerator>

        <!--数据库连接 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://127.0.0.1:3306/test1?characterEncoding=utf8" userId="root"
                        password="123456">
        </jdbcConnection>
        <!--默认false Java type resolver will always use java.math.BigDecimal if
            the database column is of type DECIMAL or NUMERIC. -->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <!--生成实体类 指定包名 以及生成的地址 （可以自定义地址，但是路径不存在不会自动创建 使用Maven生成在target目录下，会自动创建） -->
        <javaModelGenerator targetPackage="model"
                            targetProject="F:/workspaceIdea/testCode/MybatisGenerator/src/main/java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!--生成SQLMAP文件 -->
        <sqlMapGenerator targetPackage="mapper"
                         targetProject="F:/workspaceIdea/testCode/MybatisGenerator/src/main/resources">
            <property name="enableSubPackages" value="false" />
        </sqlMapGenerator>
        <!--生成Dao文件 可以配置 type="XMLMAPPER"生成xml的dao实现 context id="DB2Tables" 修改targetRuntime="MyBatis3" -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="dao"
                             targetProject="F:/workspaceIdea/testCode/MybatisGenerator/src/main/java">
            <property name="enableSubPackages" value="false" />
        </javaClientGenerator>

        <!--对应数据库表 mysql可以加入主键自增 字段命名 忽略某字段等 -->
        <table tableName="tb_video_label" domainObjectName="VideoLabel"
               enableCountByExample="false" enableUpdateByExample="false"
               enableDeleteByExample="false" enableSelectByExample="false"
               selectByExampleQueryId="false" />
    </context>
</generatorConfiguration>
```

#### 其他

吐槽一下。其实Mybatis一直以灵活广受称赞，技术选型选它也是看中这一点。使用了MybatisGenerator之后，微妙地跟一直诟病的Hibernate接近。不过工作需要，有时候项目还是得快速开发，有代码洁癖受不了的话，可以自己另外开一个xml来写sql（比如我）。