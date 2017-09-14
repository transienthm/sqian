## 优势：为什么搭建私有的仓库管理
1. 从本地或内网下载依赖包，提高效率
2. 提供第三方库的下载
3. 发布组内的组件, Snapshot and Release
4. 保证依赖库的版本统一

##  安装配置过程
1. 下载安装包
nexus官网地址http://www.sonatype.org/nexus/, 有内嵌Jetty的bundle版本，可以独立运行。

2. 解压文件
$ tar -xvf nexus-2.11.1-01-bundle.tar.gz

得到两个目录 nexus-2.11.1-01 和 sonatype-work，执行文件/配置文件保存在 nexus-2.11.1-01 目录, jar包文件保存在 sonatype-work 目录

3. 安装
本配置将nexus安装到了/usr/local/目录下
$ cp -r nexus-2.11.1-01 /usr/local/
$ cp -r sonatype-work /usr/local/
4. 使用admin帐户进行配置

Nexus默认配置了三个帐户: admin, deployment and anonymous。及时修改admin和deployment的默认密码

配代了三个proxy repository: 如果有其他第三库下载不到，可将对应的repository加入proxy

管理用户及其读写权限可使用admin帐户配置

ps: 默认Nexus对releases版本库，deploy policy设置为disabled，要布置发布则修改为Allow

5. 配置Nexus作为系统服务
```
sudo adduser --home /opt/nexus --disabled-login --disabled-password nexus
确保 nexus 安装目录的所有者为 nexus

$ cd /etc/init.d
$ chkconfig --add nexus
$ chkconfig --levels 345 nexus on
$ service nexus start
Starting Sonatype Nexus...
$ tail -f /usr/local/nexus/logs/wrapper.log
Change the owner and group of the directories used by the repository manager, including nexus-work configured in nexus.properties defaulting to sonatype-work/nexus, to the nexus user that will run the application. (Refer to the reference)
```
## 开发使用

1. setting 设置
server提供身份验证，id名称要与pom.xml中的发布对应
```
<settings>
    <servers>
        <server>
            <id>nexus</id>
            <username>username</username>
            <password>password</password>
        </server>
        <server>
            <id>snapshots</id>
            <username>username</username>
            <password>password</password>
        </server>
        <server>
            <id>releases</id>
            <username>username</username>
            <password>password</password>
        </server>
    </servers>
    <mirrors>
        <mirror>
        <!--This sends everything else to /public -->
            <id>nexus</id>
            <mirrorOf>*</mirrorOf>
            <url>http://localhost:8081/nexus/content/groups/public</url>
        </mirror>
    </mirrors>

    <profiles>
        <profile>
            <id>nexus</id>
            <!--Enable snapshots for the built in central repo to direct -->
            <!--all requests to nexus via the mirror -->
            <repositories>
                <repository>
                <id>central</id>
                <url>http://central</url>
                <releases><enabled>true</enabled></releases>
                <snapshots><enabled>true</enabled></snapshots>
                </repository>
            </repositories>

            <pluginRepositories>
                <pluginRepository>
                    <id>central</id>
                    <url>http://central</url>
                    <releases><enabled>true</enabled></releases>
                    <snapshots><enabled>true</enabled></snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>

    <activeProfiles>
    <!--make the profile active all the time -->
        <activeProfile>nexus</activeProfile>
    </activeProfiles>
</settings>
```
2. pom.xml设置

用于将包发布到 snapshots 和 releases
```
<distributionManagement>
    <snapshotRepository>
        <id>snapshots</id>
        <url>http://localhost:8081/nexus/content/repositories/snapshots</url>
    </snapshotRepository>

    <repository>
        <id>releases</id>
        <url>http://localhost:8081/nexus/content/repositories/releases</url>
    </repository>
</distributionManagement>
```

> 参考资料
使用Nexus替代Center https://books.sonatype.com/nexus-book/reference/config-maven.html
配置作为系统服务 https://books.sonatype.com/nexus-book/reference/install-sect-service.html
用户验证 https://books.sonatype.com/nexus-book/reference/_adding_credentials_to_your_maven_settings.html
