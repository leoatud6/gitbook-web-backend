# HTTP

## HTTP2 （vsHTTP1.1）

* binary file VS text communication
* 多路复用代替阻塞(long polling长链接)
* 服务器推送
* compressed header

```markup
server.http2.enabled = true
//need tomcat version >= 8
```

##

## Request

* 请求行: get/post/put/delete
* header: key-value
* body: 只有post的时候才会有

## Socket

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FE2F2K0GbfAnAfjylJQ8U%2Ffile.png?alt=media)

```java
//simple code for socket test
public static void main(String[] args) throws Exception {
    Socket socket = null;
    InputStream is = null;
    OutputStream os = null;

    try {
        socket = new Socket("www.baidu.com",80);
        is = socket.getInputStream();
        os = socket.getOutputStream();
        os.write("GET /subject/about/index.html HTTP/1.1\n".getBytes());
        os.write("HOST:www.baidu.com\n".getBytes());
        os.write("\n".getBytes());

        int i=is.read();
        while(i!=-1){
            System.out.print((char)i);
            i = is.read();
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if(null!=is){
            is.close();
            is=null;
        }
        if(null!=os){
            os.close();
            os=null;
        }
        if(null!=socket){
            socket.close();
            socket=null;
        }
    }
}



//socket server code, should be in while loop
while(true){
    ServerSocket serverSocket;
    Socket socket;
    OutputStream ops;
    try {
        serverSocket = new ServerSocket(8080);
        socket = serverSocket.accept();
        ops = socket.getOutputStream();
        ops.write("".getBytes());
        ops.write("".getBytes());
        ops.write("".getBytes());
        ops.write("".getBytes());
        StringBuffer sb = new StringBuffer();
        
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
    
    }
}
```



![server handle response](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FqLEoipn8OgcBAN6dDCXV%2Ffile.png?alt=media)













