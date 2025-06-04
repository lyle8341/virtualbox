# harbor


+ 密码默认: admin/Harbor12345

+ docker-compose安装
  + 1.下载[docker-compose](https://github.com/docker/compose/releases/tag/v2.36.2)
  + 2.mv 第一步下载的文件 /usr/local/bin/docker-compose
  + 3.sudo chmod +x /usr/local/bin/docker-compose
  + 4.全局可用
    > sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
  + 3.验证是否成功
    > docker-compose --version
+ 相当于 stop;rm 两个同时执行
  > docker-compose down -v
+ **重启harbor**
  - 在/root/harbor/docker-compose.yml路径下执行命令
  > docker-compose up -d
+ 	

---
+ 浏览器访问harbor
  + https://localhost:52443/
  + NAT端口转发到harbor虚拟机的443端口
+ Log into Harbor from the Docker client.
  + docker login 10.0.2.152
  + 报错就用下面的方式解决即可（--insecure-registry）


### docker login遇到问题 
> Error response from daemon: Get "https://10.0.2.152/v2/": tls: failed to verify certificate: x509: cannot validate certificate for 10.0.2.152 because it doesn't contain any IP SANs
+ 修改文件
  > vim /usr/lib/systemd/system/docker.service
  ```shell
  # 在Service配置组里添加 --insecure-registry https://10.0.2.152
  ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry https://10.0.2.152
  ```
+ 重启
  ```shell
  systemctl daemon-reload
  systemctl restart docker
  ```
