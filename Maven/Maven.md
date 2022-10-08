# Maven

## 1. scope

|                    | compile | test | provided |
| :----------------: | :-----: | :--: | :------: |
|     主程序有效     |   √   |  ×  |    √    |
|    测试程序有效    |   √   |  √  |    √    |
|   参与打包/部署   |   √   |  ×  |    ×    |
| 依赖可传递到子模块 |   √   |  ×  |    ×    |

### 1.1 compile

### 1.2 test

典型：JUnit

### 1.3 provided

典型：servlet-api.jar

### 1.4 system

* maven scope非compile范围的依赖是不能传递的。所以system 引用的jar不会被传递
* 引入自定义jar

```xml
<!--dependencies部分-->
<dependencies>
    <!--按如下方式引入每一个第三方的jar包，其中${project.basedir}指当前项目的根目录-->
    <dependency>
        <groupId>com.test</groupId>
        <artifactId>test</artifactId>
        <scope>system</scope>
        <version>1.0</version>
        <systemPath>${project.basedir}/src/main/resources/lib/test.jar</systemPath>
    </dependency>
</dependencies>
 
<!--如果是打jar包，则需在build的plugins中添加如下配置-->
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <!--值为true是指打包时包含scope为system的第三方Jar包-->
        <includeSystemScope>true</includeSystemScope>
    </configuration>
</plugin>
 
<!--如果是打war包，则需在build的plugins中设置maven-war-plugin插件，否则外部依赖无法打进war包 -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <configuration>
        <webResources>
            <resource>
                <directory>src/main/resources/lib</directory>
                <targetPath>WEB-INF/lib/</targetPath>
                <includes>
                    <include>**/*.jar</include>
                </includes>
            </resource>
        </webResources>
    </configuration>
</plugin>
```

## 2. 依赖的原则（jar冲突）

### 2.1 路径最短优先

### 2.2 路径相同，先声明者优先

## 3.  Maven命令

### 3.1 打包强制刷新snapshot快照

**`-U强制maven更新snapshot ... mvn clean package   -DskipTests -U  -Parm`**

在持续集成服务器上使用怎样的 mvn 命令集成项目，这个问题乍一看答案很显然，不就是 mvn clean install 么？事实上比较好的集成命令会稍微复杂些，下面是一些总结：

不要忘了clean： clean能够保证上一次构建的输出不会影响到本次构建。

使用deploy而不是install： 构建的SNAPSHOT输出应当被自动部署到私有Maven仓库供他人使用，这一点在前面已经详细论述。

使用-U参数： 该参数能强制让Maven检查所有SNAPSHOT依赖更新，确保集成基于最新的状态，如果没有该参数，Maven默认以天为单位检查更新，而持续集成的频率应该比这高很多。

使用-e参数：如果构建出现异常，该参数能让Maven打印完整的stack trace，以方便分析错误原因。

使用-Dmaven.repo.local参数：如果持续集成服务器有很多任务，每个任务都会使用本地仓库，下载依赖至本地仓库，为了避免这种多线程使用本地仓库可能会引起的冲突，可以使用-Dmaven.repo.local=/home/juven/ci/foo-repo/这样的参数为每个任务分配本地仓库。

使用-B参数：该参数表示让Maven使用批处理模式构建项目，能够避免一些需要人工参与交互而造成的挂起状态。

综上，持续集成服务器上的集成命令应该为 mvn clean deploy -B -e -U -Dmaven.repo.local=xxx 。此外，定期清理持续集成服务器的本地Maven仓库也是个很好的习惯，这样可以避免浪费磁盘资源，几乎所有的持续集成服务器软件都支持本地的脚本任务，你可以写一行简单的shell或bat脚本，然后配置以天为单位自动清理仓库。需要注意的是，这么做的前提是你有私有Maven仓库，否则每次都从Internet下载所有依赖会是一场噩梦。

### 3.2 mvn 日志 -X

mvn clean install -X
-X 可以打印出具体的错误信息
