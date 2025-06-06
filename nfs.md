# nfs搭建



+ 1.在每台机器上执行
  - yum install nfs-utils
+ 2.在master上执行
  - echo "/nfs/data/ *(insecure,rw,sync,no_root_squash)" > /etc/exports
  - mkdir -p /nfs/data
  - systemctl enable rpcbind
  - systemctl enable nfs-server
  - systemctl start rpcbind
  - systemctl start nfs-server
  - exportfs -r
  
+ 3.配置nfs-client
  - showmount -e <nfs-server节点ip>
  - mkdir -p /nfs/data
  - mount -t nfs nfs_server_ip:/nfs/data /nfs/data