# 常用命令

### Namespace

+ 创建Namespace
  - kubectl create ns xx 
  - yaml方式
    - 新建一个名为my-namespace.yaml文件
      ```
	  apiVersion: v1
	  kind: Namespace
	  metadata:
	    name: xxx
	  ```
	- kubectl apply -f my-namespace.yaml


+ 删除namespace
    + kubectl delete ns 空间名
    + kubectl delete -f my-namespace.yaml


---

### pod(一个pod可以运行多个容器)

+ pod命名规则
  - 1.由小写字母、数字和连字符（-）组成
  - 2.长度在1~253之间
  - 3.首尾不能是连字符（-）

+ 创建pod，运行一个nginx容器，命令行方式
  - kubectl run lyle-nginx --image=nginx:latest

+ 查看pod详情
  - kubectl describe pod <pod-name>
  - 关注 Events 内容
  - kubectl get pod -owide

+ 查看pod日志
  - kubectl logs <pod-name>

+ 删除pod
  - kubectl delete pod <pod-name>


+ 进入容器
  - kubectl exec -it <pod-name> -c <容器名> -- sh


---

### deployment

> deployment负责创建和更新应用程序的实例，使pod拥有多副本，自愈，扩缩容等能力。创建Deployment后，K8s Master将应用程序实例调度到集群的各个节点。如果托管实例的节点关闭或被删除，Deployment控制器会将该实例替换为集群中另一个节点上的实例。


+ kubectl create deployment lyle-nginx --image=nginx:latest
+ kubectl get pods
+ kubectl get deploy
+ 监测
  - kubectl get pods -w
  - watch kubectl get pods


+ 查看deploy信息
  - kubectl get deploy
  
+ **删除deployment**
  - kubectl delete deployment <deploy-name>

+ 查看pod打印的日志
  - kubectl logs lyle-nginx-8647b4f46f-fxjsv


+ 使用exce 在pod的容器中执行命令
  - kubectl exec lyle-nginx-8647b4f46f-fxjsv -- env
  - kubectl exec lyle-nginx-8647b4f46f-fxjsv -- ls
  - kubectl exec lyle-nginx-8647b4f46f-fxjsv -- sh

+ 多副本创建
  - kubectl create deployment lyle-nginx --image=nginx:latest --replicas=3
+ 查看deployment
  - kubectl describe deployment lyle-nginx

+ 编辑部署信息
  - kubectl edit deploy <deploy-name>

+ 扩缩容
  - kubectl scale --replicas=5 deployment <deploy-name>
  - kubectl scale --replicas=1 deployment <deploy-name>


+ 滚动升级
  > 对lyle-tomcat这个deployment进行滚动升级和回滚，将tomcat版本由tomcat:9.0.55升级到tomcat:10.1.11,再回滚到tomcat:9.0.55
  - kubectl set image deployment <deployment-name> <container-name>=<new-image>
  - kubectl set image deployment lyle-tomcat tomcat=tomcat:10.0.11 --record
  - --record: 这个选项会将这个更改记录到 Deployment 的修订历史中。可以通过查看修订历史来追踪更改的详细信息。例如，你可以使用 kubectl rollout history deployment/my-app 来查看历史记录
+ 回滚
  - kubectl rollout history deployment <deploy-name>
  - 回滚到上一个版本
    - kubectl rollout undo deployment lyle-tomcat
  - 回滚到指定版本
    - kubectl rollout undo deployment lyle-tomcat --to-revision=2



### service

> Service是一个抽象层，它定义了一组Pod的逻辑集，并为这些Pod支持外部流量暴露、负载均衡和服务发现。


+ type类型
  - ClusterIP
	- 只能从集群内访问
  - NodePort
    - <NodeIP>:<NodePort>从集群外部访问
  - LoadBalancer
    - 创建一个负载均衡器（如果支持的话），并为service分配一个固定的外部IP
  - ExternalName
    - 通过返回带有该名称的CNAME记录，使用任意名称（由spec中的externalName指定）公开service，不适用代理



+ 创建service
  - kubectl expose deployment <deploy-name> --name=tomcat --port=8080 --type=NodePort (port是service的端口)
  - kubectl get svc
    ```
    [root@k8s-1 ~]# kubectl get pod,svc -owide
    NAME                               READY   STATUS    RESTARTS   AGE    IP             NODE    NOMINATED NODE   READINESS GATES
    pod/lyle-tomcat-7f7d59f6c9-csxmp   1/1     Running   0          116m   172.16.13.81   k8s-3   <none>           <none>
    
    NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE    SELECTOR
    service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          7d1h   <none>
	service/tomcat       NodePort    10.106.31.200   <none>        8080:31802/TCP   92m    app=lyle-tomcat

	访问方式：
	pod:  curl 172.16.13.81:8080
	svc:  curl 10.106.31.200:8080
	node: curl localhost:31802 (会负载均衡到每个pod)
	```


  - ~~kubectl create service nodeport NAME [--tcp=port:targetPort] [--dry-run=server|client|none]~~
    - 新建一个名为 my-ns 的 NodePort 类型 Service
      - kubectl create service nodeport nginx-deploy-service --tcp=80:80
	  - **这种有缺陷，如果service的label和selector与deploy中的label和selector不一致，则无法使用service访问**






+ 使用资源清单
  - kubectl edit svc <service-name>
  - 会生成一个临时文件，如：（"/tmp/kubectl-edit-2083600716.yaml"）
  - kubectl get svc <service-name> -oyaml


+ 各端口解释
  - port: service的虚拟ip，集群内网可以使用 serviceIp:port 访问服务
  - nodePort: service在宿主机上映射的外网访问端口，端口范围必须在30000-32767之间
  - targetPort: pod暴露的端口，一般与pod内部容器暴露的端口一致

### volume

> volume指的是存储卷，包含可被pod中容器访问的数据目录，容器中的文件在磁盘上是临时存放的，当容器崩溃时文件会丢失，同时无法在多个pod中共享文件，volume可以解决这个问题



### ingress

> 它允许在K8s集群中暴露HTTP和HTTPS服务，通过Ingress，可以将流量路由到不同的服务和端点，而无需使用不同的负载均衡器。

+ Ingress和Service区别
  - Service是k8s中抽象的应用程序服务，它公开了一个单一的IP地址和端口，可在集群内部的pod间进行流量路由
  - Ingress是一个资源对象，它提供了对集群外部流量路由的规则。通过一个公共IP地址和端口将流量路由到一个或多个service




















### 易错
+ 多个资源间的逗号后面不能有空格
  - kubectl get svc,deploy,pod -owide
  - kubectl get svc,deploy























