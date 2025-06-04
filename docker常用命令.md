# docker常用命令

+ docker ps显示信息不全，解决方法
  - docker ps --no-trunc
  
+ 只显示容器ID
  - docker ps -q
  
+ 根据条件过滤显示的容器
  - docker ps -f "status=exited"

+ 显示名称包含 my_container 的容器
  - docker ps -f "name=my_container"  
  
+ 格式化输出
  - docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"

+ 删除容器
  - docker rm 容器id

+ 删除镜像
  - docker rmi 镜像id

+ docker执行Dockerfile文件中的内容，在其之下maven打包时候，依赖下载的位置（在安装docker的服务器上执行）
  - find / -name "*.m2*" -type d
  - find . -name "*.jar" -printf "%f\n" (最后的 -printf "%f\n" 表示只输出文件名，而不包含路径)


### Dockerfile编写
+ 方式一（每次都会下载jar依赖）
```dockerfile
# 第一阶段构建镜像
FROM openjdk:17-jdk-slim AS build
COPY pom.xml mvnw ./
COPY .mvn .mvn
RUN chmod +x ./mvnw
RUN ./mvnw dependency:resolve
COPY src src
RUN ./mvnw package

# 第二阶段构建镜像
FROM openjdk:17-jdk-slim
WORKDIR demo
# COPY --from=<builder> <src>... <dest>
# <builder>是之前构建的镜像的名称或ID。<src>...是要从构建的镜像中复制的源文件或目录，可以是多个。<dest>是目标路径。
COPY --from=build target/*.jar demo.jar
ENTRYPOINT ["java", "-jar", "demo.jar"]
```

+ 方式二
  + 先使用本地的maven进行编译成jar
  + 在直接使用jar
```dockerfile
FROM openjdk:17-jdk-slim
WORKDIR demo
COPY target/*.jar demo.jar
ENTRYPOINT ["java", "-jar", "demo.jar"]
```




