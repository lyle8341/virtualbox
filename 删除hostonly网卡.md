# 删除宿主机上的HOST ONLY


> 在使用VirtualBox虚拟机时会自动创建一个VirtualBox Host-Only Ethernet Adapter网络链接，如果禁用还会创建第二个。


+ 删除方式
  + 1.进入到virtualBox安装目录
    > E:\Program Files\Oracle\VirtualBox
  + 2.执行命令
    > vboxmanage list hostonlyifs
  + 3.删除命令
    > vboxmanage hostonlyif remove "<Name>"
    
    即: `vboxmanage hostonlyif remove "VirtualBox Host-Only Ethernet Adapter"`
  + > 如果你有三个名称完全一样的VirtualBox Host-Only Ethernet Adapter，那么会默认删除第一个，再次执行命令会删除第二个