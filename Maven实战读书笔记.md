### 2. Maven的安装和配置

#### Maven环境变量配置

1. 配置JDK环境（略）
2. 下载Maven并解压缩
3. 新建系统变量M2_HOME，值为Maven解压后的目录
4. 添加Path变量：%M2_HOME%\bin


#### 打印Java系统属性和环境变量

```shell
mvn help:system
```

#### 设置HTTP代理

> 目测用不到

```xml
<proxies>
	<proxy>
  		<id>my-proxy</id>
      	<active>true</active>
      	<protocol>http</protocol>
      	<host>218.14.227.194</host>
      	<port>3212</port>
      	<!--
  		<username>* * *</username>
  		<password>* * *</password>
		-->
	</proxy>
</proxies>
```

#### Maven安装最佳实践

* 设置MAVEN_OPTS环境变量，值为-Xms128m -Xmx512m，以增大Maven运行内存
* 配置用户范围的settings.xml，该xml位于用户目录下的.m2/settings.xml，避免设置全局后影响系统中其他用户，并便于Maven升级
* 不要使用IDE内嵌的Maven，避免各种环境下版本不同造成的不稳定

### 3. Maven使用入门

#### 基本流程

1. 编写POM

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project ...>
   	<modelVersion>当前POM模型的版本，目前只能是4.0.0</modelVersion>
     	<groupId>项目所在组</groupId>
     	<artifactId>项目所在组中的唯一ID</artifactId>
     	<version>项目当前版本，SNAPSHOT表示还处于开发中</version>
     	<name>声明一个对于用户更为友好的项目名称</name>
   </project>
   ```

2. 编写主干代码

   * 遵循Maven的约定，将主代码放到src/main/java/目录下

   * mvn clean compile：清理输出目录target/，编译项目主代码，clean、compile等均属于Maven插件

   * 由于历史原因，compiler插件默认只支持编译Java 1.3(书上所言，不知道现在是几)，需要进行配置：

     ```xml
     <build>
     	<plugins>
       		<plugin>
           		<groupId>org.apache.maven.plugins</groupId>
               	<artifactId>maven-compiler-plugin</artifactId>
               	<configuration>
               		<source>1.5</source>
                   	<target>1.5</target>
               	</configuration>
           	</plugin>
       	</plugins>
     </build>
     ```

     ​

3. 编写测试代码

   * 需要添加相关的测试包依赖如JUnit
   * 测试类放在src/test/java目录下

4. 打包和运行

   * mvn clean package：打包

   * mvn clean install：运行

   * 生成可执行的jar文件（默认打包生成的jar并没有将带有main方法的类信息添加到mainifest中）

     ```xml
     <puglin>
     	<groupId>org.apache.maven.plugins</groupId>
       	<artifactId>maven-shade-plugin</artifactId>
       	<version>1.2.1</version>
       	<executions>
           	<execution>
           		<phase>package</phase>
               	<goals>
               		<goal>shade</goal>
               	</goals>
               	<configuration>
                   	<transformers>
                       	<transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                           	<mainClass>com.juvenxu.mvnbook.helloworld.HelloWorld</mainClass>
                       	</transformer>
                   	</transformers>
               	</configuration>
           	</execution>
       	</executions>
     </puglin>
     ```

5. 使用Archetype生成项目骨架

   mvn archetype:generate

   有很多可用的Archetype以供选择，默认为maven-archetype-quickstart

### 5. 坐标和依赖

#### 坐标

* groupId
* artifactId
* version
* packaging
* classifier

前三个是必须定义的，packaing是可选的（默认为jar），classifier是不能直接定义的。

#### 依赖

* groupId、artifacId、version：依赖的基本坐标
* type：依赖的类型，一般不必声明，默认为jar
* scope：依赖的范围
* optional：以来是否可选
* exclusions：排除传递性依赖

##### 依赖范围

依赖范围就是用来控制依赖与三种classpath（编译、测试、运行）的关系

* compile

  编译依赖范围，默认值。对于三种classpath都有效

* test

  测试依赖范围，只对于测试classpath有效，在编译主代码或运行项目时无法使用此类依赖，如JUnit。

* provided

  已提供依赖范围，对于编译和测试classpath有效，但在运行时无效。如servlet-api，编译和测试项目时需要该依赖，但在运行项目时，容器已经提供，不需要重复引入。

* runtime

  运行时依赖，对于测试和运行classpath有效，编译主代码时无效

* system

  系统依赖范围，类似与provided，但是指定本机系统的依赖，如引入环境变量

* import

##### 传递性依赖

项目直接定义的依赖为第一依赖，传递性依赖则是第一依赖的依赖，可称第二依赖。

* 当第二依赖的scope为compile时，第二依赖的scope等于第一依赖的scope
* 如果第二依赖的scope是test，则不会引入该第二依赖
* 第二依赖的scope为provided时，只有当第一依赖也为provided才会引入
* 第二依赖的scope为runtime时，基本与第一依赖scope一致，但若第一依赖为compile，第二依赖为runtime

###### 依赖调节

如果不同的第一依赖引入了相同的第N依赖，路径最近者优先，其次第一声明者优先

##### 可选依赖

> 用途不明

```xml
<dependency>
	<groupId>mysql</groupId>
  	<artifactId>mysql-connector-java</artifactId>
  	<version>5.1.10</version>
  	<optional>true</optional>
</dependency>
<dependency>
	<groupId>postgresql</groupId>
  	<artifactId>postgresql</artifactId>
  	<version>8.4-701.jdbc3</version>
  	<optional>true</optional>
</dependency>
```

#### 最佳实践

1. 排除依赖

   < 没看懂。。。 >

2. 归类依赖

   使用属性定义依赖的版本，如：

   ```xml
   <properties>
   	<springframework.version>2.5.6</springframework.version>
   </properties>
   <denpendencies>
   	<dependency>
     		...
         	<version>${springframework.version}</version>
     	</dependency>
   </denpendencies>
   ```

3. 优化依赖

   可以运行 mvn dependency:list 查看当前项目已解析的依赖，通过 mvn dependency:tree 查看当前项目的依赖树，通过 mvn dependency:analyze 查看使用但未声明的依赖与声明但未使用的依赖

### 仓库

编辑settings.xml，设置仓库地址：

```xml
<settings>
	<localRepository>D:\java\repository\</localRepository>
</settings>
```

在%M2_HOME%/bin/maven-model-builder-3.0.jar中的pom-4.0.0.xml里配置了中央仓库的位置。可如下在pom.xml中配置远程仓库：

```xml
<repositories>
	<repository>
      	<id>jboss</id>
      	<name>JBoss Repository</name>
      	<url>http://repository.jboss.com/maven2/</url>
      	<releases>
          	<enabled>true</enabled>
      	</releases>
      	<snapshots>
          	<enabled>true</enabled>
          	<updatePolicy>daily</updatePolicy>
          	<checksumPolicy>ignore</checksumPolicy>
      	</snapshots>
      	<layout>default</layout>
  	</repository>
</repositories>
```

updatePolicy - 配置Maven从远程仓库检查更新的频率，默认值为daily：

* daily：每天检查一次
* never：从不检查更新
* always：每次构件都检查更新
* interval:x：每个x分钟检查一次

##### 在settings.xml中配置仓库认证信息：

```xml
<servers>
	<server>
      	<id>my-proj</id>
      	<username>repo-user</username>
      	<password>repo-pwd</password>
  	</server>
</servers>
```

##### 在pom.xml中设置部署至远程仓库

```xml
<distributionManagement>
	<repository>
      	<id>proj-releases</id>
      	<name>Proj Release Repository</name>
      	<url>http://192.168.1.100/content/repositories/proj-releases</url>
  	</repository>
  	<snapshotRepository>
      	<id>proj-releases</id>
      	<name>Proj Release Repository</name>
      	<url>http://192.168.1.100/content/repositories/proj-snapshots</url>
  	</snapshotRepository>
</distributionManagement>
```

#### 快照版本

设定版本号为快照的模块，在发布到仓库服务器中时，Maven会自动为构件打上时间戳，有了该时间戳，Maven就能随时找到仓库中该构件快照版本最新的文件。检查更新频率由updatePolicy控制，强制检查更新可使用命令： **mvn clean install-U**

#### 镜像

如果仓库X可以提供仓库Y存储的所有内容，那么就可以认为X是Y的一个镜像。在settings.xml中可以配置镜像：

```xml
<mirrors>
	<mirror>
      	<id>maven.net.cn</id>
      	<name>one of the central mirrors in China</name>
      	<url>http://maven.net.cn/content/groups/public/</url>
      	<mirrorOf>central</mirrorOf>
  	</mirror>
</mirrors>
```

< mirrorOf >的值代表任何对于该仓库的请求都会转至该镜像。

### 生命周期和插件

#### clean生命周期

1.  **pre-clean** 执行一些清理前需要完成的工作
2.  **clean** 清理上一次构建生成的文件
3.  **post-clean** 执行一些清理后需要完成的工作

#### default生命周期

* **validate**
* **initialize**
* **generate-sources**
* **process-sources**
* **generate-resources**
* **process-resoures**
* **compile**
* **process-classes**
* **generate-test-sources**
* **process-test-sources**
* **test**
* **prepare-package**
* **package**
* **pre-integration-test**
* **integration-test**
* **post-integration-test**
* **verify**
* **install**
* **deploy**

#### site生命周期

* **pre-site**
* **site**
* **post-site**
* **site-deploy**

#### 命令行与生命周期

* **$mvn clean**: pre-clean，clean
* **$mvn test**: validate ~ test
* **$mvn clean install**: pre-clean，clean，validate ~ install
* **$mvn clean deploy site-deploy**: pre-clean、clean，default所有，site所有

#### 插件目标

maven-插件名-plugin，对应目标：插件名:目标名

如：maven-dependency-plugin插件的目标有：dependency:analyze，dependency:tree，dependency:list等。

Maven的生命周期与插件相互绑定，用以完成实际的构建任务。

内置插件如：maven-clean-plugin，maven-site-plugin，maven-resources-plugin，maven-compiler-plugin，maven-surefire-plugin，maven-jar-plugin，maven-install-plugin，maven-deploy-plugin。

可以自定义绑定插件目标：

```xml
<build>
	<plugins>
      	<!--插件坐标声明-->
  		<groupId>org.apache.maven.plugins</groupId>
  		<artifactId>maven-source-plugin</artifactId>
  		<version>2.1.1</version>
  		<executions>
  			<execution>
              	 <!--任务标识-->
    			<id>attach-sources</id>
              	 <!--绑定的生命周期-->
    			<phase>verify</phase>
    			<goals>
                  	 <!--执行的插件任务-->
    				<goal>jar-no-fork</goal>
    			</goals>
              	 <configration>
              	 	<tasks>
                    	<echo>just a test</echo>  	
                   	</tasks>
              	 </configration>
    		</execution>
  		</executions>
  	</plugins>
</build>
```

### 聚合与继承

#### 聚合

创建一个额外的模块，来构建所有模块。该模块本身一般只有pom文件，并且打包方式的值必须为pom：

```xml
<project ...>
	<modelVersion>4.0.0</modelVersion>
  	<groupId>com.juvexu.mvnbook.account</groupId>
  	<artifactId>acct-aggregator</artifactId>
  	<version>1.0.0-SNAPSHOT</version>
  	<packaging>pom</packaging>
  	<name>Account Aggregator</name>
  	<modules>
      	<module>account-email</module>
      	<module>account-persist</module>
	</modules>
</project>
```

每个module的值都是一个当前POM的相对目录，聚合模块与其他模块的目录结构并非一定是要父子类关系，只要在module处以pom所在目录为基准指定正确的路径即可。

#### 继承

一个项目的不同模块可能引用了很多同样的依赖，为了避免重复引入，可以在聚合模块中加入依赖，并让其他模块继承。在需要继承的模块的pom中加入：

```xml
<modelVersion>4.0.0</modelVersion>
<parent>
	<groupId>com.juvenxu.mvnbook.account</groupId>
  	<artifactId>account-parent</artifactId>
  	<version>1.0.0-SNAPSHOT</version>
  	<relativePath>../account-parent/pom.xml</relativePath>
</parent>
<artifactId>account-email</artifactId>
<name>Account Email</name>
```

relativePath的默认值为../pom.xml，即默认认为父POM文件在上一层目录下。此处写法意味着聚合模块与本模块处于同一级目录。

**关于如何防止一个简单模块引入了大量非必要的依赖，尚未得知**

> 猜测：在父pom中，定义< dependencyManagement >是会让子模块继承的部分
>
> 有待证实

#### 反应堆

项目的构建顺序与依赖的顺序有关，依赖的项目先于项目构建。依赖关系会将反应堆构成一个有向非循环图，各个模块是该图的节点，依赖关系构成了有向边，这个图不允许出现循环，如果有，maven就会报错。

##### 裁剪反应堆

* -am：同时构建所列模块的依赖模块
* -amd：同时构建依赖于所列模块的模块
* -pl：构建指定的模块，模块间用逗号分隔
* -rf：在完整的反应堆构建顺序基础上指定从哪个模块开始构建

如：

```shel
$ mvn clean install -pl account-email -am
$ mvn clean install -rf account-email
$ mvn clean install -pl account-parent -amd -rf account-email
```

### 使用Maven进行测试

#### maven-surefire-plugin简介

默认情况下，maven-surefire-plugin的test目标会自动执行测试源码路径（默认为src/test/java/）下所有符合一组命名模式的测试类，这组模式为：

- `**/Test*.java`：任何子目录下所有命名以Test开头的Java类
- `**/*Test.java`：任何子目录下所有命名以Test结尾的Java类
- `**/*TestCase.java`：任何子目录下所有命名以TestCase结尾的Java类。

#### 跳过测试

添加参数，如：$mvn package-DskipTests

也可以在配置中设置：

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>2.5</version>
  <configuration>
    <skipTests>true</skipTests>
  </configuration>
</plugin>
```

甚至可以临时跳过测试代码的编译：$mvn package-Dmaven.test.skip=true

或：

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>2.1</version>
  <configuration>
    <!--书中为skip，怀疑笔误-->
    <skipTests>true</skipTests>
  </configuration>
</plugin>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>2.5</version>
  <configuration>
    <skipTests>true</skipTests>
  </configuration>
</plugin>
```

#### 动态指定要运行的测试用例

如果只想运行RandomGeneratorTest用例，则可以使用如下命令：

$mvn test-Dtest=RandomGeneratorTest

使用星号匹配零个或多个字符：

$mvn test-Dtest=Random*Test

使用逗号指定多个测试用例：

$mvn test-Dtest=RandomGeneratorTest,AccountCaptchaServiceTest

也可以结合使用。

test参数的值必须匹配一个或多个测试类，如果maven-surefire-plugin找不到任何匹配的测试类，就会报错并导致构建失败。可以加上参数，表示即使没有任何测试也不要报错：

$mvn test -Dtest -DfailifNoTests=false

#### 包含与排除测试用例

maven-surefire-plugin允许用户通过额外的配置来自定义包含一些其他测试类，或者排除一些符合默认命名模式的测试类。如下，自动运行以Tests结尾的测试类，排除特定测试类：

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>2.5</version>
  <configuration>
    <includes>
      <include>**/*Tests.java</include>
    </includes>
    <excludes>
      <exclude>**/*ServiceTest.java</exclude>
      <exclude>**/TempDaoTest.java</exclude>
    </excludes>
  </configuration>
</plugin>
```

两个星号**用来匹配任意路径，一个星号匹配除路径风格符外的0个或者多个字符。

#### 测试报告

略

#### 运行TestNG测试

TestNG允许用户使用一个名为testng.xml的文件来配置想要运行的测试集合：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<suite name="Suite1" verbose="1">
  <test name="Regression1">
    <classes>
      <class name="com.juvenxu.RandomGeneratorTest"/>
    </classes>
  </test>
</suite>
```

同时在配置maven-surefire-plugin使用该testng.xml：

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>2.5</version>
  <configuration>
    <suiteXmlFiles>
      <suiteXmlFile>testng.xml</suiteXmlFile>
    </suiteXmlFiles>
  </configuration>
</plugin>
```

在TestNG中，可以通过注解将测试方法加入到测试组中：

@Test(groups={"util","medium"})

通过如下配置运行一个或者多个TestNG测试组：

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>2.5</version>
  <configuration>
    <groups>util,medium</groups>
  </configuration>
</plugin>
```

#### 重用测试代码

从命令行运行mvn package的时候，Maven会将项目的主代码及资源文件打包，默认的打包行为是不会包含测试代码的。当需要对测试类进行打包时，需要显式配置：

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-jar-plugin</artifactId>
  <version>2.2</version>
  <executions>
    <execution>
      <goals>
        <goal>test-jar</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

使用测试包作为依赖：

```xml
<dependency>
  <groupId>com.juvenxu.account</groupId>
  <artifactId>account-captcha</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <type>test-jar</type>
  <scope>test</scope>
</dependency>
```

### 灵活的构建

使用Maven属性归类依赖：

```xml
<properties>
  <springframework.version>2.5.6</springframework.version>
</properties>
<dependencies>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>${springframework.version}</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>${springframework.version}</version>
  </dependency>
  ...
</dependencies>
```

Maven有6类属性：

- 内置属性：主要有两个常用的内置属性——`${basedir}`表示项目根目录，即包含pom.xml文件的目录；`${version}`表示项目版本。

- POM属性：用户可以使用该类属性引用POM文件中对应元素的值，如：

  - `${project.build.sourceDirectory}`：项目主源码目录
  - `${project.build.testSrouceDirectory}`：项目测试代码目录
  - `${project.build.directory}`：项目构建输出目录
  - `${project.outputDirectory}`：项目主代码编译输出目录
  - `${project.testOutputDirectory}`：项目测试代码编译输出目录
  - `${project.groupId}`：项目的groupId
  - `${project.artifactId}`：项目的artifactId
  - `${project.version}`：项目的version，与`${version}`等价
  - `${project.build.finalName}`：项目打包输出文件的名称

- 自定义属性：用户可以在POM的`<properties>`下自定义Maven属性

  ```xml
  <project>
  ...
    <properties>
      <my.prop>hello</my.prop>
    </properties>
  ...
  </project>
  ```

  然后在POM其他地方使用${my.prop}时会被替换成为hello

- Settings属性：与POM属性同理，用户使用以settings.开头的属性引用settings.xml文件中XML元素的值，如常用的${settings.localRepository}指向用户本地仓库中的地址。

- Java系统属性：如{user.home}指向了用户目录。用户可以使用mvn help:system 查看所有的Java系统属性。

- 所有的环境变量都可以使用以env.开头的Maven属性引用。例如${env.JAVA_HOME}指代了Java_HOME环境变量的值。

#### 构建环境的差异

配置文件中，使用Maven属性：

```properties
database.jdbc.driverClass = ${db.driver}
database.jdbc.connectionURL = ${db.url}
database.jdbc.username = ${db.username}
database.jdbc.password = ${db.password}
```

在POM中定义属性：

```xml
<profiles>
  <profile>
    <id>dev</id>
    <properties>
      <db.driver>com.mysql.jdbc.Driver</db.driver>
      <db.url>jdbc:mysql://192.168.1.100:3306/test</db.url>
      <db.username>dev</db.username>
      <db.password>dev-pwd</db.password>
    </properties>
  </profile>
</profiles>
```

Maven属性默认只有在POM中才会被解析。资源文件的处理其实是maven-resources-plugin做的事情，只要通过一些简单的配置，就能够解析资源文件中的Maven属性，即开启资源过滤：

```xml
<resources>
  <resource>
    <directory>${project.basedir}/src/main/resources</directory>
    <filtering>true</filtering>
  </resource>
</resources>
```

```xml
<testResources>
  <testResource>
    <directory>${project.baesdir}/src/test/resources</directory>
    <filtering>true</filtering>
  </testResource>
</testResources>
```

Maven允许用户声明多个资源文件，并且为每个资源目录提供不同的过滤配置：

```xml
<resources>
  <resource>
    <directory>src/main/resources</directory>
    <filtering>true</filtering>
  </resource>
  <resource>
    <directory>src/main/sql</directory>
    <filtering>false</filtering>
  </resource>
</resources>
```

使用命令行激活profile，Maven在构建项目的时候使用profile中属性值替换数据库配置文件中的属性引用：

```shell
$mvn clean install -Pdev
```

#### 激活profile

##### 命令行激活

激活dev-x和dev-y两个profile：

```shell
$mvn clean install -Pdev-x,dev-y
```

##### settings文件显式激活

```xml
<settings>
...
  <activeProfiles>
    <activeProfile>dev-x</activeProfile>
  </activeProfiles>
...
</settings>
```

##### 系统属性激活

某系统属性存在时激活profile

```xml
<profiles>
  <profile>
    <activation>
      <property>
        <name>test</name>
      </property>
    </activation>
    ...
  </profile>
</profiles>
```

某系统属性存在且确定时激活profile

```xml
<profiles>
  <profile>
    <activation>
      <property>
        <name>test</name>
        <value>x</value>
      </property>
    </activation>
    ...
  </profile>
</profiles>
```

命令行也可以声明系统属性，例如：

```shell
$mvn clean install -Dtest=x
```

##### 操作系统环境激活

```xml
<profiles>
  <profile>
    <activation>
      <os>
        <name>Windows XP</name>
        <family>Windows</family>
        <arch>x86</arch>
        <version>5.1.2600</version>
      </os>
    </activation>
    ...
  </profile>
</profiles>
```

family包括Windows，UNIX和Mac等，其他几项可以通过查看环境中的系统属性os.name，os.arch，os.version获得。

##### 文件存在与否激活

```xml
<profiles>
  <profile>
    <activation>
      <file>
        <missing>x.properties</missing>
        <exists>y.properties</exists>
      </file>
    </activation>
    ...
  </profile>
</profiles>
```

##### 默认激活

```xml
<profiles>
  <profile>
    <id>dev</id>
    <activation>
      <activeByDefault>true</activeByDefault>
    </activation>
    ...
  </profile>
</profiles>
```

可以通过maven-help-plugin的一个目标来了解当前激活的profile：

```shell
$mvn help:active-profiles
$mvn help:all-profiles
```

#### profile的种类

根据具体的需要，可以在一下位置声明profile：

- pom.xml：只对当前项目生效
- 用户settings.xml：对本机该用户所有的Maven项目有效
- 全局settings.xml：对本级所有的Maven项目有效。
- profiles.xml：单独文件，Maven3已移除。

POM中的profile可使用的元素：

```xml
<project>
  <repositories></repositories>
  <pluginRepositories></pluginRepositories>
  <distributionManagement></distributionManagement>
  <dependencies></dependencies>
  <dependencyManagement></dependencyManagement>
  <modules></modules>
  <properties></properties>
  <reporting></reporting>
  <build>
    <plugins></plugins>
    <defaultGoal></defaultGoal>
    <resources></resources>
    <testResources></testResources>
    <finalName></finalName>
  </build>
</project>
```

POM外部的只能声明`repositories`，`pluginRepositories`，`properties`

#### Web资源过滤

除了打包后位于classpath中的资源文件，还有一类资源文件，默认源码位于src/main/webapp/下，经打包后位于WAR包的根目录。有时候希望对这类资源文件也使用Maven属性，如为不同的客户使用不一样的logo或者css主题：

```xml
<profiles>
  <profile>
    <id>client-a</id>
    <properties>
      <client.logo>a.jpg</client.logo>
      <client.theme>red</client.theme>
    </properties>
  </profile>
  <profile>
    <id>client-b</id>
    <properties>
      <client.logo>b.jpg</client.logo>
      <client.theme>blue</client.theme>
    </properties>
  </profile>
</profiles>
```

配置maven-war-plugin对src/main/webapp/这一web资源目录开启过滤：

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-war-plugin</artifactId>
  <version>2.1-beta-1</version>
  <configuration>
    <webResources>
      <resource>
        <filtering>true</filtering>
        <directory>src/main/webapp</directory>
        <includes>
          <include>* */*.css</include>
          <include>* */*.js</include>
        </includes>
      </resource>
    </webResources>
  </configuration>
</plugin>
```

配置完成后可以激活某个profile进行构建

### 生成项目站点

使用maven-site-plugin，运行`$mvn site`即可生成一个最简单的站点，位于target/site/目录下。

使用stage目标发布到临时目录下：`$mvn site:stage -DstagingDirectory=D:\tmp`

