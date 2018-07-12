# GeneratorMapper
配合Mapper通用插件用Generator生成mapper文件
## MyBatis Geneator 详解
### 配置文件头
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
```
### 根节点generatorConfiguration
> generatorConfiguration节点没有任何属性，直接写节点即可，如下：
```xml
<generatorConfiguration>
	<!-- 具体配置内容 -->
</generatorConfiguration>  
```
### generatorConfiguration子元素
#### properties元素
>元素用于指定一个需要在配置中解析使用的外部属性文件，引入属性文件后，可以在配置中使用 ${property}这种形式的引用，通过这种方式引用属性文件中的属性值。 对于后面需要配置的jdbc信息和targetProject属性会很有用。
这个属性可以通过resource或者url来指定属性文件的位置，这两个属性只能使用其中一个来指定，同时出现会报错。

> * resource：指定classpath下的属性文件，使用类似com/myproject/generatorConfig.properties这样的属性值。
> * url：可以指定文件系统上的特定位置，例如file:///C:/myfolder/generatorConfig.properties

#### classPathEntry元素
>这个元素可以0或多个，不受限制。这个元素的作用是将MBG运行时需要用到的jar包(或zip格式)添加到classpath下。最常见的用法是，当classpath下面没有JDBC驱动的时候，我们通常通过这个属性指定驱动的路径，例如：
```xml
<classPathEntry location="E:\mysql\mysql-connector-java-5.1.29.jar"/>
```

#### context元素
> 在MBG的配置中，至少需要有一个<context>元素。
> <context>元素用于指定生成一组对象的环境。例如指定要连接的数据库，要生成对象的类型和要处理的数据库中的表。运行MBG的时候还可以指定要运行的<context>。
> 该元素只有一个必选属性id，用来唯一确定一个<context>元素，该id属性可以在运行MBG的使用。
>此外还有几个可选属性：

> * defaultModelType:这个属性很重要，这个属性定义了MBG如何生成实体类。这个属性有以下可选值：
> 1. conditional:这是默认值,这个模型和下面的hierarchical类似，除了如果那个单独的类将只包含一个字段，将不会生成一个单独的类。 因此,如果一个表的主键只有一个字段,那么不会为该字段生成单独的实体类,会将该字段合并到基本实体类中。
> 2. flat:该模型为每一张表只生成一个实体类。这个实体类包含表中的所有字段。这种模型最简单，推荐使用。
> 3. hierarchical:如果表有主键,那么该模型会产生一个单独的主键实体类,如果表还有BLOB字段， 则会为表生成一个包含所有BLOB字段的单独的实体类,然后为所有其他的字段生成一个单独的实体类。 MBG会在所有生成的实体类之间维护一个继承关系。

> * targetRuntime:此属性用于指定生成的代码的运行时环境。该属性支持以下可选值：
> 1. MyBatis3:默认的值，生成基于MyBatis3.x以上版本的内容，包括XXXBySample
> 2. MyBatis3Simple:类似MyBatis3，只是不生成XXXBySample

> * introspectedColumnImpl:该参数可以指定扩展org.mybatis.generator.api.IntrospectedColumn该类的实现类。该属性的作用

一般情况下，我们使用如下的配置即可：
<context id="Mysql" defaultModelType="flat">

如果你希望不生成和Example查询有关的内容，那么可以按照如下进行配置:
<context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">

##### context子元素
MBG配置中的其他几个元素，基本上都是<context>的子元素，这些子元素（有严格的配置顺序）包括：
*   `<property>` (0个或多个)
*   `<plugin>` (0个或多个)
*   `<commentGenerator>` (0个或1个)
*   `<jdbcConnection>` (1个)
*   `<javaTypeResolver>` (0个或1个)
*   `<javaModelGenerator>` (1个)
*   `<sqlMapGenerator>` (0个或1个)
*   `<javaClientGenerator>`  (0个或1个)
*   `<table>` (1个或多个)

##### property
其中<property>属性比较特殊，后面讲解的时候都会和父元素一起进行讲解。在讲解<property>属性前，我们先看看什么是分隔符？。

这里通过一个例子说明。假设在Mysql数据库中有一个表名为user info，你没有看错，中间是一个空格，这种情况下如果写出select * from user info这样的语句，肯定是要报错的，在Mysql中的时候我们一般会写成如下的样子:
```sql
select * from `user info`
```
这里的使用的反单引号(`)就是分隔符，分隔符可以用于表名或者列名。

下面继续看<property>支持的属性：

> * autoDelimitKeywords
> * beginningDelimiter
> * endingDelimiter
> * javaFileEncoding
> * javaFormatter
> * xmlFormatter
由于这些属性比较重要，这里一一讲解。

> * 首先是autoDelimitKeywords，当表名或者字段名为SQL关键字的时候，可以设置该属性为true，MBG会自动给表名或字段名添加分隔符。

> * 然后这里继续上面的例子来讲beginningDelimiter和endingDelimiter属性。由于beginningDelimiter和endingDelimiter的默认值为双引号(")，在Mysql中不能这么写，所以还要将这两个默认值改为反单引号(`)，配置如下：
```xml
 <property name="beginningDelimiter" value="`"/>
 <property name="endingDelimiter" value="`"/>  
```
> * 属性javaFileEncoding设置要使用的Java文件的编码，默认使用当前平台的编码，只有当生产的编码需要特殊指定时才需要使用，一般用不到。

> * 最后两个javaFormatter和xmlFormatter属性可能会很有用，如果你想使用模板来定制生成的java文件和xml文件的样式，你可以通过指定这两个属性的值来实现。


##### public元素
该元素可以配置0个或者多个，不受限制。

<plugin>元素用来定义一个插件。插件用于扩展或修改通过MyBatis Generator (MBG)代码生成器生成的代码。
插件将按在配置中配置的顺序执行。

##### jdbcConnection元素

<jdbcConnection>用于指定数据库连接信息，该元素必选，并且只能有一个。

配置该元素只需要注意如果JDBC驱动不在**classpath**下，就需要通过`<classPathEntry>`元素引入jar包，这里**推荐**将jar包放到**classpath**下。

该元素有两个必选属性:

> *   `driverClass`:访问数据库的JDBC驱动程序的完全限定类名
> *   `connectionURL`:访问数据库的JDBC连接URL

该元素还有两个可选属性:

> *   `userId`:访问数据库的用户ID
> *   `password`:访问数据库的密码

此外该元素还可以接受多个`<property>`子元素，这里配置的`<property>`属性都会添加到JDBC驱动的属性中。

这个元素配置起来最容易，这里举个简单例子：
```xml
<jdbcConnection driverClass="com.mysql.jdbc.Driver"
                connectionURL="jdbc:mysql://localhost:3306/test"
                userId="root"
                password="">
</jdbcConnection>
```
##### javaModelGenerator元素

该元素必须配置一个，并且最多一个。

该元素用来控制生成的实体类，根据`<context>`中配置的`defaultModelType`，一个表可能会对应生成多个不同的实体类。一个表对应多个类实际上并不方便，所以前面也推荐使用`flat`，这种情况下一个表对应一个实体类。

该元素只有两个属性，都是必选的。

> *   `targetPackage`:生成实体类存放的包名，一般就是放在该包下。实际还会受到其他配置的影响(`<table>`中会提到)。
> *   `targetProject`:指定目标项目路径，使用的是文件系统的绝对路径。

该元素支持以下几个`<property>`子元素属性：

> *   `constructorBased`:该属性只对`MyBatis3`有效，如果`true`就会使用构造方法入参，如果`false`就会使用`setter`方式。默认为`false`。
> *   `enableSubPackages`:如果`true`，MBG会根据`catalog`和`schema`来生成子包。如果`false`就会直接用`targetPackage`属性。默认为`false`。
> *   `immutable`:该属性用来配置实体类属性是否可变，如果设置为`true`，那么`constructorBased`不管设置成什么，都会使用构造方法入参，并且不会生成`setter`方法。如果为`false`，实体类属性就可以改变。默认为`false`。
> *   `rootClass`:设置所有实体类的基类。如果设置，需要使用类的全限定名称。并且如果MBG能够加载`rootClass`，那么MBG不会覆盖和父类中完全匹配的属性。匹配规则：
    *   属性名完全相同
    *   属性类型相同
    *   属性有`getter`方法
    *   属性有`setter`方法
> *   `trimStrings`:是否对数据库查询结果进行`trim`操作，如果设置为`true`就会生成类似这样`public void setUsername(String username) {this.username = username == null ? null : username.trim();}`的`setter`方法。默认值为`false`。

配置示例如下：
```xml
<javaModelGenerator targetPackage="test.model" targetProject="E:\MyProject\src\main\java">
  	<property name="enableSubPackages" value="true" />
  	<property name="trimStrings" value="true" />
</javaModelGenerator>
```
##### sqlMapGenerator元素

该元素可选，最多配置一个。但是有如下两种必选的特殊情况：

> *   如果`targetRuntime`目标是**iBATIS2**，该元素必须配置一个。
> *   如果`targetRuntime`目标是**MyBatis3**，只有当`<javaClientGenerator>`需要XML时，该元素必须配置一个。 如果没有配置
<javaClientGenerator>，则使用以下的规则：
> * 如果指定了一个`<sqlMapGenerator>`，那么MBG将只生成XML的SQL映射文件和实体类。
>  * 如果没有指定`<sqlMapGenerator>`，那么MBG将只生成实体类。

> * `targetPackage`:生成实体类存放的包名，一般就是放在该包下。实际还会受到其他配置的影响(`<table>`中会提到)。
> *   `targetProject`:指定目标项目路径，使用的是文件系统的绝对路径。

该元素支持`<property>`子元素，只有一个可以配置的属性：

> *   `enableSubPackages`:如果`true`，MBG会根据`catalog`和`schema`来生成子包。如果`false`就会直接用`targetPackage`属性。默认为`false`。

配置示例：
```xml
<sqlMapGenerator targetPackage="test.xml"  targetProject="E:\MyProject\src\main\resources">
  	<property name="enableSubPackages" value="true" />
</sqlMapGenerator>
```
##### javaClientGenerator元素

该元素可选，最多配置一个。如果不配置该元素，就不会生成Mapper接口。
该元素有3个必选属性：

> *   `type`:该属性用于选择一个预定义的客户端代码（可以理解为Mapper接口）生成器，用户可以自定义实现，需要继承`org.mybatis.generator.codegen.AbstractJavaClientGenerator`类，必选有一个默认的构造方法。 该属性提供了以下预定的代码生成器，首先根据`<context>`的`targetRuntime`分成三类：
  
 >  *   **MyBatis3**:
       >  *   **ANNOTATEDMAPPER**:基于注解的Mapper接口，不会有对应的XML映射文件
      >   *   **MIXEDMAPPER**:XML和注解的混合形式，(上面这种情况中的)`SqlProvider`注解方法会被XML替代。
      >   *   **XMLMAPPER**:所有的方法都在XML中，接口调用依赖XML文件。
   
>  *   **MyBatis3Simple**:
      >   *   **ANNOTATEDMAPPER**:基于注解的Mapper接口，不会有对应的XML映射文件
       >  *   **XMLMAPPER**:所有的方法都在XML中，接口调用依赖XML文件。

  >   *   **Ibatis2Java2**或**Ibatis2Java5**:
      >   *   **IBATIS**:生成的对象符合iBATIS的DAO框架（不建议使用）。
      >   *   **GENERIC-CI**:生成的对象将只依赖于SqlMapClient，通过构造方法注入。
      >   *   **GENERIC-SI**:生成的对象将只依赖于SqlMapClient，通过`setter`方法注入。
      >   *   **SPRING**:生成的对象符合Spring的DAO接口

> *   `targetPackage`:生成实体类存放的包名，一般就是放在该包下。实际还会受到其他配置的影响(`<table>`中会提到)。
> *   `targetProject`:指定目标项目路径，使用的是文件系统的绝对路径。

该元素还有一个可选属性：

> *   `implementationPackage`:如果指定了该属性，实现类就会生成在这个包中。

该元素支持`<property>`子元素设置的属性：

> *   `enableSubPackages`
> *   `exampleMethodVisibility`
> *   `methodNameCalculator`
> *   `rootInterface`
> *   `useLegacyBuilder`

> 这几个属性不太常用，具体作用请看完整的文档，这里对`rootInterface`做个简单介绍。`rootInterface`用于指定一个所有生成的接口都继承的父接口。 这个值可以通过`<table>`配置的`rootInterface`属性覆盖。这个属性对于通用Mapper来说，可以让生成的所有接口都继承该接口。

配置示例：
```xml
<javaClientGenerator type="XMLMAPPER" targetPackage="test.dao" targetProject="E:\MyProject\src\main\java"/>
```


##### table元素

该元素至少要配置一个，可以配置多个。
该元素用来配置要通过内省的表。只有配置的才会生成实体类和其他文件。
该元素有一个必选属性：

> *   `tableName`：指定要生成的表名，可以使用[SQL通配符]匹配多个表。

例如要生成全部的表，可以按如下配置：
```xml
<table tableName="%" />
```

该元素包含多个可选属性：
> *  `schema`:数据库的schema,可以使用[SQL通配符](http://www.w3school.com.cn/sql/sql_wildcards.asp)匹配。如果设置了该值，生成SQL的表名会变成如`schema.tableName`的形式。
> *   `catalog`:数据库的catalog，如果设置了该值，生成SQL的表名会变成如`catalog.tableName`的形式。
> *   `alias`:如果指定，这个值会用在生成的select查询SQL的表的别名和列名上。 列名会被别名为 alias_actualColumnName(别名_实际列名) 这种模式。
> *   `domainObjectName`:生成对象的基本名称。如果没有指定，MBG会自动根据表名来生成名称。
> *   `enableXXX`:XXX代表多种SQL方法，该属性用来指定是否生成对应的XXX语句。
> *   `selectByPrimaryKeyQueryId`:DBA跟踪工具会用到，具体请看详细文档。
> *   `selectByExampleQueryId`:DBA跟踪工具会用到，具体请看详细文档。
> *   `modelType`:和`<context>`的`defaultModelType`含义一样，这里可以针对表进行配置，这里的配置会覆盖`<context>`的`defaultModelType`配置。
> *   `escapeWildcards`:这个属性表示当查询列，是否对schema和表名中的SQL通配符 ('_' and '%') 进行转义。 对于某些驱动当schema或表名中包含SQL通配符时（例如，一个表名是MY_TABLE，有一些驱动需要将下划线进行转义）是必须的。默认值是`false`。
> *   `delimitIdentifiers`:是否给标识符增加**分隔符**。默认`false`。当`catalog`,`schema`或`tableName`中包含空白时，默认为`true`。
> *   `delimitAllColumns`:是否对所有列添加**分隔符**。默认`false`。

该元素包含多个可用的`<property>`子元素，可选属性为：

> *   `constructorBased`:和`<javaModelGenerator>`中的属性含义一样。
> *   `ignoreQualifiersAtRuntime`:生成的SQL中的表名将不会包含`schema`和`catalog`前缀。
> *   `immutable`:和`<javaModelGenerator>`中的属性含义一样。
> *   `modelOnly`:此属性用于配置是否为表只生成实体类。如果设置为`true`就不会有Mapper接口。如果配置了`<sqlMapGenerator>`，并且`modelOnly`为`true`，那么XML映射文件中只有实体对象的映射元素(`<resultMap>`)。如果为`true`还会覆盖属性中的`enableXXX`方法，将不会生成任何CRUD方法。
> *   `rootClass`:和`<javaModelGenerator>`中的属性含义一样。
> *   `rootInterface`:和`<javaClientGenerator>`中的属性含义一样。
> *   `runtimeCatalog`:运行时的`catalog`，当生成表和运行环境的表的`catalog`不一样的时候可以使用该属性进行配置。
> *   `runtimeSchema`:运行时的`schema`，当生成表和运行环境的表的`schema`不一样的时候可以使用该属性进行配置。
> *   `runtimeTableName`:运行时的`tableName`，当生成表和运行环境的表的`tableName`不一样的时候可以使用该属性进行配置。
> *   `selectAllOrderByClause`:该属性值会追加到`selectAll`方法后的SQL中，会直接跟`order by`拼接后添加到SQL末尾。
> *   `useActualColumnNames`:如果设置为true,那么MBG会使用从数据库元数据获取的列名作为生成的实体对象的属性。 如果为false(默认值)，MGB将会尝试将返回的名称转换为驼峰形式。 在这两种情况下，可以通过 元素显示指定，在这种情况下将会忽略这个（useActualColumnNames）属性。
> *   `useColumnIndexes`:如果是true,MBG生成resultMaps的时候会使用列的索引,而不是结果中列名的顺序。
> *   `useCompoundPropertyNames`:如果是true,那么MBG生成属性名的时候会将列名和列备注接起来. 这对于那些通过第四代语言自动生成列(例如:FLD22237),但是备注包含有用信息(例如:"customer id")的数据库来说很有用. 在这种情况下,MBG会生成属性名FLD2237_CustomerId。

除了`<property>`子元素外，`<table>`还包含以下子元素：
> * <generatedKey>元素
这个元素最多可以配置一个。
这个元素用来指定自动生成主键的属性（identity字段或者sequences序列）。如果指定这个元素，MBG在生成insert的SQL映射文件中插入一个`<selectKey>`元素。 这个元素**非常重要**，这个元素包含下面两个必选属性：
> *   `column`:生成列的列名。
> *   `sqlStatement`:将返回新值的 SQL 语句。如果这是一个identity列，您可以使用其中一个预定义的的特殊值。预定义值如下：
   >  *   **Cloudscape**
    > *   **DB2**
    > *   **DB2_MF**
    > *   **Derby**
    > *   **HSQLDB**
    > *   **Informix**
    > *   **MySql**
    > *   **SqlServer**
    > *   **SYBASE**
    > *   **JDBC**:这会配置MBG使用MyBatis3支持的JDBC标准的生成key来生成代码。 这是一个独立于数据库获取标识列中的值的方法。

 **重要**: 只有当目标运行为MyBatis3时才会产生正确的代码。 如果与iBATIS2一起使用目标运行时会产生运行时错误的代码。

这个元素还包含两个可选属性：

> *   `identity`:当设置为`true`时,该列会被标记为`identity`列， 并且`<selectKey>`元素会被插入在`insert`后面。 当设置为`false`时，
`<selectKey>`会插入到`insert`之前（通常是序列）。**重要**: 即使您`type`属性指定为`post`，您仍然需要为`identity`列将该参数设置为`true`。 这将标志MBG从插入列表中删除该列。默认值是`false`。
*   `type`:`type=post and identity=true`的时候生成的`<selectKey>`中的`order=AFTER`,当`type=pre`的时候，`identity只能为false`，生成的`<selectKey>`中的`order=BEFORE`。可以这么理解，自动增长的列只有插入到数据库后才能得到ID，所以是`AFTER`,使用序列时，只有先获取序列之后，才能插入数据库，所以是`BEFORE`。

配置示例一：
```xml
<table tableName="user login info" domainObjectName="UserLoginInfo">
  	<generatedKey column="id" sqlStatement="Mysql"/>
</table>
```
对应的生成的结果：
```xml
<insert id="insert" parameterType="test.model.UserLoginInfo">
	<selectKey keyProperty="id" order="BEFORE" resultType="java.lang.Integer">
  		SELECT LAST_INSERT_ID()
	</selectKey>
	insert into `user login info` (Id, username, logindate, loginip)
	values (#{id,jdbcType=INTEGER}, #{username,jdbcType=VARCHAR}, #{logindate,jdbcType=TIMESTAMP}, #{loginip,jdbcType=VARCHAR})
</insert>
```

配置示例二：
```xml
<table tableName="user login info" domainObjectName="UserLoginInfo">
  	<generatedKey column="id" sqlStatement="select SEQ_ID.nextval from dual"/>
</table>
```
对应的生成结果：
```xml
<insert id="insert" parameterType="test.model.UserLoginInfo">
  	<selectKey keyProperty="id" order="BEFORE" resultType="java.lang.Integer">
    	select SEQ_ID.nextval from dual
  	</selectKey>
  	insert into `user login info` (Id, username, logindate, loginip)
  	values (#{id,jdbcType=INTEGER}, #{username,jdbcType=VARCHAR}, #{logindate,jdbcType=TIMESTAMP},#{loginip,jdbcType=VARCHAR})
</insert>
```


配置示例三：
```xml
<table tableName="user login info" domainObjectName="UserLoginInfo">
  	<generatedKey column="id" sqlStatement="JDBC"/>
</table>
```


对应的生成结果：
```xml
  <insert id="insert" keyProperty="id" parameterType="test.model.UserLoginInfo" useGeneratedKeys="true">
    	insert into user login info (username, logindate, loginip)
    	values (#{username,jdbcType=VARCHAR}, #{logindate,jdbcType=TIMESTAMP}, #{loginip,jdbcType=VARCHAR})
  </insert>
```


###### columnRenamingRule元素
该元素最多可以配置一个，使用该元素可以在生成列之前，对列进行重命名。这对那些存在同一前缀的字段想在生成属性名时去除前缀的表非常有用。 例如假设一个表包含以下的列：

> *   `CUST_BUSINESS_NAME`
> *   `CUST_STREET_ADDRESS`
> *   `CUST_CITY`
> *   `CUST_STATE`

生成的所有属性名中如果都包含`CUST`的前缀可能会让人不爽。这些前缀可以通过如下方式定义重命名规则:
```xml
<columnRenamingRule searchString="^CUST_" replaceString="" />
```
注意，在内部，MBG使用`java.util.regex.Matcher.replaceAll`方法实现这个功能。 请参阅有关该方法的文档和在Java中使用正则表达式的例子。

> 当`<columnOverride>`匹配一列时，这个元素（`<columnRenamingRule>`）会被忽略。`<columnOverride>`优先于重命名的规则。

> 该元素有一个必选属性：
> *   `searchString`:定义将被替换的字符串的正则表达式。
该元素有一个可选属性：
> *   `replaceString`:这是一个用来替换搜索字符串列每一个匹配项的字符串。如果没有指定，就会使用空字符串。

关于`<table>`的`<property>`属性`useActualColumnNames`对此的影响可以查看完整文档。

###### columnOverride元素

该元素可选，可以配置多个。该元素从将某些属性默认计算的值更改为指定的值。
> 该元素有一个必选属性:
> *   `column`:要重写的列名。
该元素有多个可选属性：
> *   `property`:要使用的Java属性的名称。如果没有指定，MBG会根据列名生成。 例如，如果一个表的一列名为`STRT_DTE`，MBG会根据`<table>`的`useActualColumnNames`属性生成`STRT_DTE`或`strtDte`。
> *   `javaType`:该列属性值为完全限定的Java类型。如果需要，这可以覆盖由`JavaTypeResolver`计算出的类型。 对某些数据库来说，这是必要的用来处理**“奇怪的”**数据库类型（例如MySql的`unsigned bigint`类型需要映射为`java.lang.Object`)。
> *   `jdbcType`:该列的JDBC类型(`INTEGER, DECIMAL, NUMERIC, VARCHAR`等等)。 如果需要，这可以覆盖由`JavaTypeResolver`计算出的类型。 对某些数据库来说，这是必要的用来处理怪异的JDBC驱动 (例如DB2的`LONGVARCHAR`类型需要为iBATIS 映射为`VARCHAR`)。
> *   `typeHandler`:用户定义的需要用来处理这列的类型处理器。它必须是一个继承iBATIS的`TypeHandler`类或`TypeHandlerCallback`接口（该接口很容易继承）的全限定的类名。如果没有指定或者是空白，iBATIS会用默认的类型处理器来处理类型。**重要**:MBG不会校验这个类型处理器是否存在或者可用。 MGB只是简单的将这个值插入到生成的SQL映射的配置文件中。
> *   `delimitedColumnName`:指定是否应在生成的SQL的列名称上增加**分隔符**。 如果列的名称中包含空格，MGB会自动添加**分隔符**， 所以这个重写只有当列名需要强制为一个合适的名字或者列名是数据库中的保留字时是必要的。

配置示例：
  ```xml
<table schema="DB2ADMIN" tableName="ALLTYPES" >
	<columnOverride column="LONG_VARCHAR_FIELD" javaType="java.lang.String" jdbcType="VARCHAR" />
</table>
```
######  ignoreColumn元素

该元素可选，可以配置多个。
该元素可以用来屏蔽不需要生成的列。
> 该元素有一个必选属性：
> *   `column`:要忽略的列名。
> 该元素还有一个可选属性：
> *   `delimitedColumnName`:匹配列名的时候是否区分大小写。如果为`true`则区分。默认值为`false`，不区分大小写。
