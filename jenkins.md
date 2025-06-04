# jenkins部署

+ 1.镜像jenkins拉取
  + docker pull jenkins/jenkins:latest
+ 2.创建共享卷，修改所属组和用户,和容器里相同
  ```shell
  [root@jenkins ~]# mkdir /jenkins
  [root@jenkins ~]# chown 1000:1000 /jenkins
  
  # 这里为什么要改成 1000，是因为容器里是以 jenkins 用户的身份去读写数据，而在容器里jenkins 的 uid 是 1000
  root@7a63da8850b8:/# id jenkins
  uid=1000(jenkins) gid=1000(jenkins) groups=1000(jenkins)
  ```
+ 3.创建创建 jenkins 容器
  > docker run -dit -p 8080:8080 -p 50000:50000 --name jenkins  --privileged=true --restart=always -v /jenkins:/var/jenkins_home jenkins/jenkins:latest

+ 密码 lyle/root123456
+ 地址 http://127.0.0.1

+ jenkins绑定的docker启动参数添加
  > -H tcp://0.0.0.0:2376

+ gitlab要触发jenkins的话，就必须要关闭跨站点伪造请求，web界面里已经没法关闭了，所以要需要做如下设置
  ```shell
  [root@jenkins jenkins]# docker exec -u root -it jenkins /bin/bash
  [root@44a32750c94c /]# vi /usr/local/bin/jenkins.sh
  # 找到exec java那行
  -Dhudson.security.csrf.GlobalCrumbIssuerConfiguration.DISABLE_CSRF_PROTECTION=true 运行跨站请求访问
  
  # 最终效果如下：
  exec java -Duser.home="$JENKINS_HOME" -Dhudson.security.csrf.GlobalCrumbIssuerConfiguration.DISABLE_CSRF_PROTECTION=true "${java_opts_array[@]}" -jar ${JENKINS_WAR} "${jenkins_opts_array[@]}" "$@"
  ```
  - 没有vi命令
	```
	apt-get update
	apt-get install vim
	```
+ docker restart jenkins 


+ 使用jenkins系统工具中配置的maven构建，jar依赖存储的位置在
  > /var/jenkins_home/.m2/repository


### 下载kubectl客户端工具(参考k8s集群里面)
+ 拷贝证书和k8s集群客户端工具到jenkins容器内
+ docker cp admin.conf jenkins:/
  + 先从k8s集群拷贝过来
+ docker cp kubectl jenkins:/
+ 在jenkins容器中测试
  + ./kubectl --kubeconfig=admin.conf get pods -A