# docker-compose

+ 上线
  - docker compose up -d
+ 下线
  - docker compose down
+ 指定启动那些
  - docker compose start a1 a2 a3
+ 指定停止那些
  - docker compose stop x1 x2
+ 扩容
  - docker compose scale x1=3

### wordpress+mysql  
+ 原始方式
  - 1.自定义网络
    - 	
  - 2.wordpress容器
    ```
    docker run -d -p 8080:80 \
	-e WORDPRESS_DB_HOST=mysql \
	-e WORDPRESS_DB_USER=root \
    -e WORDPRESS_DB_PASSWORD=root \
	-e WORDPRESS_DB_NAME=wordpress \
	-v wordpress:/var/www/html \
	--restart always --name wordpress-app \
	--network blog \
	wordpress:latest
    ```
  - 3.mysql容器
    ```
	docker run -d -p 3306:3306 \
	-e MYSQL_ROOT_PASSWORD=root \
	-e MYSQL_DATABASE=wordpress \
	-v mysql-data:/var/lib/mysql \
	-v /app/myconf:/etc/mysql/conf.d \
	--restart always --name mysql \
	--network blog \
	mysql:8.0
	```
+ compose方式
  ```yaml
  name: myblog
  services:
    mysql:
      container_name: mysql
      image: mysql:8.0
      ports:
        - "3306:3306"
      environment:
        - MYSQL_ROOT_PASSWORD=root
        - MYSQL_DATABASE=wordpress
      volumes:
        - mysql-data:/var/lib/mysql
        - /app/myconf:/etc/mysql/conf.d
      restart: always
      networks:
        - blog
  
    wordpress:
      container_name: wordpress
      image: wordpress
      ports:
        - "8080:80"
      environment:
        WORDPRESS_DB_HOST: mysql
        WORDPRESS_DB_USER: root
        WORDPRESS_DB_PASSWORD: root
        WORDPRESS_DB_NAME: wordpress
      volumes:
        - wordpress:/var/www/html
      restart: always
      networks:
        - blog
      depends_on:
        - mysql
  
  volumes:
    mysql-data:
    wordpress:
  
  networks:
    blog:
  ```
+ docker compose -f blog.yaml up -d
  - docker compose多次执行，只会动修改了的容器
+ docker compose -f blog.yaml down

+ 下线并删除卷和镜像
  - docker compose -f blog.yaml down --rmi <local | all> -v
