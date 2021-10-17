# JAX-RS2

```java
@Consumer ::: 接受的内容request
@Producer ::: 返回的内容response


@GET
@Path("{id : \\d+}")   //还有regex限制必须为num
@Produces({MediaType.APPLICATION_JSON})
User getUser(@PathParam("id") Long id);


@XmlRootElement //Java API for XML binding
@XmlAccessorType(XmlAccessType.FIELD)

    @XmlElement(name = "username")
    private String name;
    
    @JsonProperty("username")
    private String name;


```





![](<../.gitbook/assets/image (225).png>)





```java
@Context   //injects an instance of a supported resource
@HeaderParam
@CookieParam
@MatrixParam
@QueryParam
@PathParam

//Resource Methods
@HttpMethod
@GET
@POST
@PUT
@DELETE
@PATCH
@HEAD
@OPTIONS

@Path

@FormParam
@HeaderParam

@Consumer
@Producer(MediaType.APPLICATION_JSON)

@PreMatching



```



```java
//Resource may return:::
void    //with 204 code
Response    //    
GenericEntity


Filter
Interceptor






```

## @context = securityContext

![](<../.gitbook/assets/image (226).png>)

![](<../.gitbook/assets/image (227).png>)

![](<../.gitbook/assets/image (228).png>)

![](<../.gitbook/assets/image (229).png>)

![](<../.gitbook/assets/image (230).png>)

### Context Negotiation 

@Producer

@Consumer

### Process Request

Provider

Response.readEntity()

MessageBodyWriter

![](<../.gitbook/assets/image (231).png>)

![](<../.gitbook/assets/image (232).png>)

`@Priority(Priorities.AUTHENTICATION) :::表示最高的优先级`

`@Priority(Priorities.HEADER_DECORTOER) :::`

`@Priority(Priorities.USER) :::`

1. Filter
2. Interceptor (ReaderXXX  + WriterXXX + ContentEncoder)

@NameBinding

### 同步VS异步通讯（主要聊异步）

![](<../.gitbook/assets/image (233).png>)

* long polling
* streaming
* web-hook async
* SSE： server send event (基于H5的异步通讯)
* WebSocket

### Rest Client

* Client
* WebTarget
* Invocation

```java
Client:::
    not recommended
        --> memory leak problem
```

![Need to config first before using the Client](<../.gitbook/assets/image (234).png>)

```java
WebTarget:::
    DSL: domain specific language
    
```













