# Hibernate

本身是对JDBC的light weight封装，充当持久层

{% embed url="https://hibernate.cfg.xml" %}

![xml 代替 real mapping](<../.gitbook/assets/image (2).png>)

![Old way to do the connection, heavy weight](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FDPaV4UNjNzohh2UiC8v3%2Ffile.png?alt=media)



## Session

1. get from sessionFactory
2. session.beginTransaction();
3. session.save(obj);
4. session.close();

## Cache (if cache exists, no need to query)

1. hashmap storage
2. 一级缓存: session level (save, update, load, get, list, lock)
3. 二级缓存: sessionFactory level

