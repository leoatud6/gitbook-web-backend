# JDBC

## JDBC

1. load driver (class.forname() ) —> reflection / drivermanager
2. driverManager.getConnect()
3. connection.createStatement
4. statement.excuteQuery() —> update
5. resultSet—> getAll back
6. close connection/statement        

    (catch all exceptions)

Threading Pool concept  --> is a container

DataSource

![](<../.gitbook/assets/image (179).png>)







![数据访问层](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FeykyUGRInKnEzLTMsxbS%2Ffile.png?alt=media)





![jdbc interface to multi driver](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FdoPhStrHtKVVCMwJ8EAv%2Ffile.png?alt=media)



## Implementation

```java
DriverManager.registerDriver(new com.mysql.jdbc.Driver());
System.setProperty("jdbc.driver" , "com.mysql.jdbc.Driver");
Class.forName("com.mysql.jdbc.Driver");


File file = new File("src/com/itheima/jdbc/JDBC_Util.java");
Reader reader = new BufferedReader(new FileReader(file));
reader.close();

ps.setCharacterStream(1,reader,(int)file.length());

```

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FVwcLpnFcxikONioDV215%2Ffile.png?alt=media)



![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2Fi0CHT6LNZglh9JT03pSW%2Ffile.png?alt=media)

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FheXNkq4KSN8BeUMltL5D%2Ffile.png?alt=media)







