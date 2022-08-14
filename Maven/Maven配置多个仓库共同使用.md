# Maven配置多个仓库共同使用

## 最终的settings.[xml](https://so.csdn.net/so/search?q=xml&spm=1001.2101.3001.7020)文件配置

如果你不想看具体的配置，可以直接把我下面这个配置拿走，修改一下`localRepository`部分对应自己的本地repo(通常为`～/.m2/repository`)，然后将`settings.xml`替换`${MAVEN_HOME}/conf/settings.xml`即可。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 http://maven.apache.org/xsd/settings-1.2.0.xsd">
  <localRepository>/Users/chris/.m2/repository</localRepository>
  <pluginGroups></pluginGroups>
  <proxies></proxies>
  <servers></servers>
  <mirrors></mirrors>
 
  <profiles>
    <profile>
      <id>aliyun</id> 
      <repositories>
        <repository>
          <id>aliyun</id> 
          <url>https://maven.aliyun.com/repository/public</url> 
          <releases>
            <enabled>true</enabled>
          </releases> 
          <snapshots>
            <enabled>true</enabled> 
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </repository>
      </repositories>
    </profile>
    <profile>
      <id>pentaho</id>
      <repositories>
        <repository>
          <id>pentaho</id>
          <url>https://nexus.pentaho.org/content/repositories/omni/</url>
          <releases>
            <enabled>true</enabled>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </repository>
      </repositories>
    </profile>
    <profile>
      <id>repo1</id>
      <repositories>
        <repository>
          <id>repo1</id>
          <url>https://repo1.maven.org/maven2</url>
          <releases>
            <enabled>true</enabled>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </repository>
      </repositories>
    </profile>
    <profile>
      <id>repo2</id>
      <repositories>
        <repository>
          <id>repo2</id>
          <url>https://repo2.maven.org/maven2</url>
          <releases>
            <enabled>true</enabled>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </repository>
      </repositories>
    </profile>
  </profiles>
 
  <activeProfiles>
  <activeProfile>aliyun</activeProfile>
  <activeProfile>pentaho</activeProfile>
  <activeProfile>repo1</activeProfile>
  <activeProfile>repo2</activeProfile>
 
</activeProfiles>
</settings>
```



## settings.xml相关具体配置介绍

### Mirrors标签部分

通常用来定义镜像仓库（repository），指定用来下载对应依赖和插件的仓库地址。使用Mirrors的理由如下：

- 配置一个就近加速的网络仓库；
- 配置一个你本地创建的镜像仓库；

来自Maven官方的Mirrors配置示例如下：

```xml
<settings>
  ...
  <mirrors>
    <mirror>
      <id>other-mirror</id>
      <name>Other Mirror Repository</name>
      <url>https://other-mirror.repo.other-company.com/maven2</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>
  ...
</settings>
```

**在Mirrors部分可以配置多个镜像仓库，但是在该部分配置多个仓库，并不能提供自动查询多个仓库的功能，默认还是取第一个仓库进行查询。**

### Profiles标签部分

settings.xml中的profile部分，包括4个二级子标签：**activation**, **repositories**, **pluginRepositories** and **properties**。profile部分的内容需要在下面进行激活。这里介绍多个镜像共同使用仅介绍**Repositories**部分。

Repositories

配置方式如下，指定repo地址、id：

```xml
  <profiles>
    <profile>
      <id>repo2</id>
      <repositories>
        <repository>
          <id>repo2</id>
          <url>https://repo2.maven.org/maven2</url>
          <releases>
            <enabled>true</enabled>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </repository>
      </repositories>
    </profile>
  </profiles>
```

### Active Profiles标签部分

settings.xml配置文件最后的一部分就是激活公用多个仓库的重点**activeProfiles**。包含多个**activeProfile**元素，每一个activeProfile元素都用来指定一个上部分profile的id，也就是说，每指定一个activeProfile映射，就激活一个profile，将会覆盖其他任何环境配置。

大家从如下例子里，可以看到我配置了aliyun、pentaho、repo1和repo2四个仓库，这样再执行mvn命令，他就会从这四个仓库循环去查找包，第一个找不到就去第二个找。。。

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  ...
<activeProfiles>
  <activeProfile>aliyun</activeProfile>
  <activeProfile>pentaho</activeProfile>
  <activeProfile>repo1</activeProfile>
  <activeProfile>repo2</activeProfile>
</activeProfiles>
</settings>
```



参考: https://maven.apache.org/guides/introduction/introduction-to-profiles.html#profile-order

All profile elements in a POM from active profiles overwrite the global elements with the same name of the POM or extend those in case of collections. In case multiple profiles are active in the same POM or external file, the ones which are defined later take precedence over the ones defined earlier (independent of their profile id and activation order).

你配置中 仓库的优先级是 repo2 > repo1 > pentaho > aliyun
另外， 现在 repo2 应该没有这个地址了吧.