# gitlab安装

+ 地址：http://127.0.0.1
+ 密码：root/root123456

+ 1. docker pull beginor/gitlab-ce
+ 2. 创建共享卷
  ```
  [root@jenkins ~]# mkdir -p /data/gitlab/etc/ /data/gitlab/log/ /data/gitlab/data
  [root@jenkins ~]# chmod 777 /data/gitlab/etc/ /data/gitlab/log/ /data/gitlab/data/
  ```

+ 3. 启动容器
  > docker run -itd --name=gitlab --restart=always --privileged=true   -p 8443:443  -p 80:80 -p 222:22 -v  /data/gitlab/etc:/etc/gitlab -v  /data/gitlab/log:/var/log/gitlab -v  /data/gitlab/data:/var/opt/gitlab  beginor/gitlab-ce
  - 切记:这里的端口要设置成80，要不push项目会提示没有报错，如果宿主机端口被占用，需要把这个端口腾出来
	
+ 4. 关闭容器修改配置文件(/data/gitlab/etc/gitlab.rb)
  - external_url 'http://192.168.112.10'
  - gitlab_rails[‘gitlab_ssh_host’] = '192.168.112.10'
  - gitlab_rails[gitlab_shell_ssh_port] = 222
  - 修改host
    - /data/gitlab/data/gitlab-rails/etc/gitlab.yml
  - 参数作用
    - grep -Ev "^$|^[#;]" /data/gitlab/etc/gitlab.rb
    - ![http选项的地址](images/http.png)
    - ![ssh选项的地址](images/ssh.png)

+ 5. 测试
  - git branch -m master
  
  
  
  
  
  
[Gitlab+Jenkins+Docker+Harbor+K8s集群搭建CICD平台](https://www.cnblogs.com/misakivv/p/18075229#tid-KyFtcz)  