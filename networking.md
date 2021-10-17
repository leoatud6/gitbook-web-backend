# Networking

![](<.gitbook/assets/image (26).png>)

```bash
nslookup 
traceroute 
nmcli

netstat antp


```

Router一般两个网卡，一个连着WAN，一个连着LAN(192.168.1.1/24)

 软路由：在设备上安装openwrt，可以刷成路由

### Routing Table

![](<.gitbook/assets/image (27).png>)



DNS: localDNS : etc/hosts  ==>映射表

 DDNS： dynamic domain name system: 公网访问nas时，只要记住域名就好，因为local ip是不断变化的，DNS会更新给你。

![](<.gitbook/assets/image (28).png>)

  如果subnet是255.255.255.255，说明只有一个ip address, 唯一的，则是点对点 

 异地组网，蒲公英，内网穿透，组网隧道，路由table



网线有cat-n等级之分，越贵的传输性能越好:

单线复用，两根并在一根上延伸出来一套iptv 

![](<.gitbook/assets/image (29).png>)

![](<.gitbook/assets/image (30).png>)















