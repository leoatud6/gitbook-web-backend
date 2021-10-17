# Hystrix (circuit breaker 断路器)

### [https://github.com/Netflix/Hystrix/wiki](https://github.com/Netflix/Hystrix/wiki)

![](<../.gitbook/assets/image (224).png>)



* 1、创建HystrixCommand 或者 HystrixObservableCommand 对象
* 2、执行命令execute()、queue()、observe()、toObservable()
* 3、如果请求结果缓存这个特性被启用，并且缓存命中，则缓存的回应会立即通过一个Observable对象的形式返回
* 4、检查熔断器状态，确定请求线路是否是开路，如果请求线路是开路，Hystrix将不会执行这个命令，而是直接执行getFallback
* 5、如果和当前需要执行的命令相关联的线程池和请求队列，Hystrix将不会执行这个命令，而是直接执行getFallback
* 6、执行HystrixCommand.run()或HystrixObservableCommand.construct()，如果这两个方法执行超时或者执行失败，则执行getFallback()
* 7、Hystrix 会将请求成功，失败，被拒绝或超时信息报告给熔断器，熔断器维护一些用于统计数据用的计数
