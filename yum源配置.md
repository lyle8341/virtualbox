#yum

+ vi 命令编辑 /etc/yum.repos.d/CentOS-Base.repo 文件，将其中的 mirrorlist 行用 # 号注释掉，并将 baseurl 行取消注释
+ 将http://mirrorlist.centos.org替换为http://mirrors.aliyun.com