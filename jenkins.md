# jenkins部署


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




