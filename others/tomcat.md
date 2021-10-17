# Tomcat

## Overview

One server —> have many services 

One service = one container —> have many connectors 

One connector —> have many endpoint 

(一个folder，也就是一个路径，代表context)

#### concepts: 

* [ ] server： start\&stop
* [ ] connector：每个连接对应不同connector
* [ ] service：每个service处理一个connector的请求，多个connector对应一个container
* [ ] container：仅处理请求，一个container可有多个context/web application
* [ ] host：一个engine，多个hosts
* [ ] servlet： 最小单元



![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FrhwSSnLB784A3ude2hfz%2Ffile.png?alt=media)

### Apache Tomcat

![](<../.gitbook/assets/image (183).png>)

```
* /bin - 启动和停止服务等批处理文件. ( *.sh) 文件 (为Unix系统)、 (*.bat) 文件 (for Windows系统)是一个功能性的复制文件. 自从Win32 command-line 开始是一些单一的，缺乏功能的组件, 现在有一些拓展性的功能
* /conf - 配置文件和一些相关的DTD文件. 最重要的是 server.xml. 它是这个容器最主要的配置文件.
* /logs - 日志文件会打印到这里
* /webapps - 这里是你的应用程序部署的地方.
```

![](<../.gitbook/assets/image (184).png>)

> Tomca的心脏是两个组件：Connecter和Container。一个Container可以选择多个Connecter，多个Connector和一个Container就形成了一个Service。Service可以对外提供服务，而Server服务器控制整个Tomcat的生命周期。

```markup
<Server>                                                //顶层类元素，可以包括多个Service   
    <Service>                                           //顶层类元素，可包含一个Engine，多个Connecter
        <Connector>                                     //连接器类元素，代表通信接口
                <Engine>                                //容器类元素，为特定的Service组件处理客户请求，要包含多个Host
                        <Host>                          //容器类元素，为特定的虚拟主机组件处理客户请求，可包含多个Context
                                <Context>               //容器类元素，为特定的Web应用处理所有的客户请求
                                        lalallalal
                                </Context>
                        </Host>
                </Engine>
        </Connector>
    </Service>
</Server>
```

Tomcat VS Nginx(written by Russian: engine X)  --> different focus











