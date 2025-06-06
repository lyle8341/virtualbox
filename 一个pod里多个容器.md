# 一个pod包含多个容器

+ 1.创建文件multi-containers-in-pod.yaml
+ 2.编辑内容
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      run: multi-in-one
    name: lyle-multi-in-pod
  spec:
    containers:
    - name: nginx-m
      image: latest
      ports:
      - containerPort: 80
    - name: tomcat-m
      image: tomcat:9.0.55
  ```
  
+ 3.创建pod
  - kubectl apply -f multi-containers-in-pod.yaml
  
  
+ 4.查看pod信息
  ```
  [root@k8s-1 ~]# kubectl get pod -owide
  NAME                           READY   STATUS    RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
  lyle-multi-in-pod              2/2     Running   0          2m54s   172.16.13.83     k8s-3   <none>           <none>

  READY 2表示里面有两个容器  
  ```
  
+ 5.进入容器
  - kubectl exec -it lyle-multi-in-pod -c nginx-m -- sh
  - kubectl exec -it lyle-multi-in-pod -c tomcat-m -- sh

+ 6.查看容器日志
  - kubectl logs lyle-multi-in-pod （查看的是资源清单上第一个容器的日志）
  - kubectl logs lyle-multi-in-pod -c nginx-m
  - kubectl logs lyle-multi-in-pod -c tomcat-m