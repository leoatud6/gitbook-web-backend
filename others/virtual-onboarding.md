# Cloud 云原生

 公有云 + 私有云 + 混合云

 考虑到安全问题，需要慎用公有云。

  ECS： elastic cloud server

  Electerm : ET : terminal tools



 google cloud --> leoatud6 --> test-instance --> monthly cost = $15

 Server connection port: 22

 public access ip: 35.245.0.28  

 云上的设置：安全组（firewall）和端口的设置。 

 停机后还会继续收费，因为磁盘会占用空间。

** 私有IP**： 建立集群的时候使用。一般不会改变。服务器之间的访问。

** 公网IP**：每次启动可能会改变。

![](<../.gitbook/assets/image (348).png>)

![](<../.gitbook/assets/image (347).png>)

**VPC：** virtual private connection (私有网络 、 专有网络)-->换分网段

 VPC提供的是**隔离**区域，不同VPC之间沟通需要external IP.

![ 后面是代表不变的，如果是24，那就只有256台机器。](<../.gitbook/assets/image (349).png>)

![](<../.gitbook/assets/image (350).png>)

 其中255.255.255.255 --> boardcast广播地址， 192.0.0.0 localhost两个不可用。

```bash
sudo apt-get install nginx
systemctl start nginx
whereis nginx
# use browser to access ip:80 --> check if nginx is running
cd /usr/share/nginx
curl 127.0.0.1:80






```



