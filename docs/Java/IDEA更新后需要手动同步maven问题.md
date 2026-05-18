### IDEA更新后需要手动同步maven问题

更新idea后，发现之前的项目打开后需要手动同步maven项目，比较烦人，经过询问ai后，找到解决方案

在 `C:\Users\xxx\.m2\repository` 中创建 `settings.xml` 配置文件，内容如下：

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">

  <!-- 这里需要替换成具体的 maven 本地仓库地址 -->
  <localRepository>D:\develop\Maven\Repository</localRepository>

  <mirrors>
    <mirror>
      <id>aliyunmaven</id>
      <mirrorOf>*</mirrorOf>
      <name>aliyun</name>
      <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
  </mirrors>

</settings>
```

问题是如何找到的：重新打开 Spring Boot 项目后

发现 项目结构-项目设置-库 中的所有依赖都去 `C:\Users\xxx\.m2\repository` 进行查找，然后报错。
