# Maven

  本地仓库索引： Index-->提高查询速度

 Traditional： 编码-->编译-->Test-->运行-->打包-->部署

 Maven: All in one



> ApacheMaven/ conf/ settings.xml 可以更改Maven的配置





![三种Repository： local, localServer, Internet](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2Fm4kWlsjxGLvC1EtapJXu%2Ffile.png?alt=media)

### Maven Lifecycle

* mvn tomcat: run —>target folder 
* mvn clean —> clean target folder 
* mvn compile —>only src/main path files (not including test path) 
* mvn test —> compile only test path and run mvn package —> var / war 
* mvn install —>put all into local repository (if has) —>相当于执行了Package/test/compile 
* mvn deploy —>发布私服

