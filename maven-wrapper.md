# maven wrapper


> 使用 Maven Wrapper 时，它会检测当前电脑上的 Maven 版本，如果和项目的 Maven 版本一致，就使用当前系统中的 Maven；如果不一致，Warpper 将会：

+ **自动下载特定版本的 Maven 到 ~\.m2\wrapper\dists 目录下**
+ **自动下载 .jar 到 ~\.m2\repository 目录下。**



+ 命令./mvnw dependency:resolve
  - 已解决依赖的列表


+ 执行mvn package和mvn install时候会下载依赖，这两个命令包含执行了mvn dependency:resolve 

