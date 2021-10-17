# Commandline

```bash
### from online store project
#General
lsof -i -P -n | grep 8080
lsof -i -P -n | grep LISTEN
Ps -ef | grep nginx

#npm
yarn add node-cron@2.0.3
Npm run dev

#fastdfs
sudo /usr/local/bin/fdfs_storaged /etc/fdfs/storage.conf restart
sudo /usr/local/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart


#nginx
/usr/local/opt/nginx/bin
brew services start nginx

#elasticsearch memory limit start
docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9200:9200 -p 9300:9300 -p 5601:5601 elasticsearch


#free domain
3225db1132.wicp.vip

#change hosts mapping
Sudo nano /etc/hosts
127.0.0.1 localhost
```
