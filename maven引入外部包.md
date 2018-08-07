### [maven 引入外部jar包的几种方式](https://blog.csdn.net/sheng_666v/article/details/79115908)
```
方式1：dependency 本地jar包
<dependency>
        <groupId>com.hope.cloud</groupId>  <!--自定义-->
        <artifactId>cloud</artifactId>    <!--自定义-->
        <version>1.0</version> <!--自定义-->
        <scope>system</scope> <!--system，类似provided，需要显式提供依赖的jar以后，Maven就不会在Repository中查找它-->
        <systemPath>${basedir}/lib/cloud.jar</systemPath> <!--项目根目录下的lib文件夹下-->
 </dependency>

方式2：编译阶段指定外部lib
<plugin>
     <artifactId>maven-compiler-plugin</artifactId>
     <version>2.3.2</version>
     <configuration>
     <source>1.8</source>
     <target>1.8</target>
     <encoding>UTF-8</encoding>
     <compilerArguments>
     <extdirs>lib</extdirs><!--指定外部lib-->
     </compilerArguments>
     </configuration>
</plugin>

方式3：将外部jar打入本地maven仓库
cmd 进入jar包所在路径，执行以下命令
mvn install:install-file -Dfile=cloud.jar -DgroupId=com.hope.cloud -DartifactId=cloud -Dversion=1.0 -Dpackaging=jar
引入依赖
<dependency>
    <groupId>com.hope.cloud</groupId>
    <artifactId>cloud</artifactId>
    <version>1.0</version>
 </dependency>
