# 项目介绍
mybatis-generator-plugins 是以 mybatis-generator-core 为基础实现的代码生成插件，为我们自动生成DO类、DAO类和_SqlMap.xml文件，并提供增删查改等基本方法。

# 核心类
## org.mybatis.generator.api.ShellRunner
主程序入口

## 继承 org.mybatis.generator.api.PluginAdapter 类
分布式开发的话，由 mybatis generator 插件生成的 实体类 和 *Example 类都必须要序列化
```
<plugin type="com.cc.mybatis.generator.SerializablePlugin"/>
```

## 继承 org.mybatis.generator.internal.DefaultCommentGenerator 类
自定义代码生成器中生成实体类的中文注释
```
<commentGenerator type="com.cc.mybatis.generator.MyCommentGenerator">
    <property name="author" value="chencheng0816@gmail.com"/>
    <property name="dateFormat" value="yyyy/MM/dd"/>
</commentGenerator>
```

## 修改 org.mybatis.generator.config.TableConfiguration 类
自定义<table>属性的默认值，默认不生成 Example类

## org.mybatis.generator.config.xml.MyBatisGeneratorConfigurationParser 的 parseTable 方法
解析并处理<table>各个属性

## org.mybatis.generator.api.IntrospectedTable
calculateJavaClientAttributes(): 自定义DAO文件名
```
if (stringHasValue(tableConfiguration.getMapperName())) {
    sb.append(tableConfiguration.getMapperName());
} else {
    if (stringHasValue(fullyQualifiedTable.getDomainObjectSubPackage())) {
        sb.append(fullyQualifiedTable.getDomainObjectSubPackage());
        sb.append('.');
    }

    // 自定义DAO文件名
    String domainObjectName = fullyQualifiedTable.getDomainObjectName();
    int domainObjectNameLength = domainObjectName.length();
    sb.append(domainObjectName.substring(0, domainObjectNameLength - 2));
    sb.append("DAO");
}
```
calculateMyBatis3XmlMapperFileName(): 自定义map xml文件名
```
if (stringHasValue(tableConfiguration.getMapperName())) {
    String mapperName = tableConfiguration.getMapperName();
    int ind = mapperName.lastIndexOf('.');
    if (ind == -1) {
        sb.append(mapperName);
    } else {
        sb.append(mapperName.substring(ind + 1));
    }
    sb.append(".xml"); //$NON-NLS-1$
} else {
    // 自定义map xml文件名
    String domainObjectName = fullyQualifiedTable.getDomainObjectName();
    int domainObjectNameLength = domainObjectName.length();
    sb.append(domainObjectName.substring(0, domainObjectNameLength - 2));
    sb.append("_SqlMap.xml"); //$NON-NLS-1$
}
```

# 插件打包
## maven打包指令
mvn clean install

# 插件引用
项目添加 mybatis-generator-plugins 包依赖后，修改 resources 路径下的 generatorConfig.xml, Maven 执行命令（mvn mybatis-generator:generate）自动生成代码  
1. 配置依赖及插件 pom.xml 
```
<!-- 添加 mybatis-generator 插件（命令运行方式：进入工程目录执行：mvn mybatis-generator:generate 生成代码） -->
<plugin>
	<groupId>org.mybatis.generator</groupId>
	<artifactId>mybatis-generator-maven-plugin</artifactId>
	<version>1.3.7</version>
	<configuration>
		<!-- 配置文件 -->
		<configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
		<!-- 允许移动和修改 -->
		<verbose>true</verbose>
		<overwrite>true</overwrite>
	</configuration>
	<dependencies>
		<!-- jdbc 依赖 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>${mysql.driver.version}</version>
		</dependency>
		<dependency>
			<groupId>com.cc</groupId>
			<artifactId>mybatis-generator-plugins</artifactId>
			<version>${project.parent.version}</version>
		</dependency>
	</dependencies>
</plugin>
```
2. 配置 generatorConfig.xml 
```
<!-- 引入 jdbc properties 配置文件 -->
<properties resource="properties/mbg.properties"/>

<context id="MySqlTables" targetRuntime="MyBatis3">

	<!-- 生成的Java文件的编码 -->
	<property name="javaFileEncoding" value="UTF-8"/>

	<!-- 格式化java代码 -->
	<property name="javaFormatter" value="org.mybatis.generator.api.dom.DefaultJavaFormatter"/>

	<!-- 格式化XML代码 -->
	<property name="xmlFormatter" value="org.mybatis.generator.api.dom.DefaultXmlFormatter"/>

	<!-- 为生成的Java模型类添加序列化接口 -->
	<plugin type="com.cc.mybatis.generator.SerializablePlugin"/>

	<!-- 为生成的Java模型创建一个toString方法 -->
	<plugin type="org.mybatis.generator.plugins.ToStringPlugin"/>

	<!-- 自定义代码生成器中生成实体类的中文注释 -->
	<commentGenerator type="com.cc.mybatis.generator.MyCommentGenerator">
		<property name="author" value="chencheng0816@gmail.com"/>
		<property name="dateFormat" value="yyyy/MM/dd"/>
	</commentGenerator>

	<!--数据库链接地址账号密码-->
	<jdbcConnection driverClass="${mbg.db.driverClassName}" connectionURL="${mbg.db.url}" userId="${mbg.db.username}" password="${mbg.db.password}">
		<property name="useInformationSchema" value="true"/>
	</jdbcConnection>

	<!-- java类型处理器 用于处理DB中的类型到Java中的类型，默认使用JavaTypeResolverDefaultImpl； 注意一点，默认会先尝试使用Integer，Long，Short等来对应DECIMAL和 NUMERIC数据类型； -->
	<javaTypeResolver>
		<property name="forceBigDecimals" value="false"/>
	</javaTypeResolver>

	<!--生成Model类存放位置-->
	<javaModelGenerator targetPackage="com.cc.model" targetProject="${mbg.path}/java">
		<property name="enableSubPackages" value="false"/>
		<property name="trimStrings" value="true"/>
	</javaModelGenerator>

	<!--生成映射文件存放位置-->
	<sqlMapGenerator targetPackage="mapper" targetProject="${mbg.path}/resources">
		<property name="enableSubPackages" value="false"/>
	</sqlMapGenerator>

	<!--生成Dao类存放位置-->
	<javaClientGenerator targetPackage="com.cc.dao" targetProject="${mbg.path}/java" type="XMLMAPPER">
		<property name="enableSubPackages" value="false"/>
	</javaClientGenerator>

	<!--生成对应表及类名-->
	<table tableName="BE_USER" domainObjectName="UserDO"/>

</context>
```
3. 配置检查及生成代码
```
配置检查：mvn mybatis-generator:help
生成代码：mvn mybatis-generator:generate
```
