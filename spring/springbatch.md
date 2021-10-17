---
description: 大数据量并行处理框架，用于数据离线迁移或处理
---

# SpringBatch

**SpringBatch **是一个**大数据量**的**并行**处理框架。通常用于数据的**离线迁移**，和数据处理，⽀持事务、并发、流程、监控、纵向和横向扩展，提供统⼀的接⼝管理和任务管理;SpringBatch是SpringSource和埃森哲为了统一业界并行处理标准为广大开发者提供方便开发的一套框架。

* SpringBatch 本身提供了_重试，异常处理，跳过，重启、任务处理统计，资源管理_等特性，这些特性开发者看重他的主要原因;
* SpringBatch 是一个**轻量级**的**批处理**框架;
* SpringBatch **结构分层**，业务与处理策略、结构分离;
* 任务的运行的实例状态，执行数据，参数都会落地到数据库;

![](<../.gitbook/assets/image (122).png>)



* Infrastureture:
* Core
* Application

#### Job & Steps:

One job have many steps



```java
@EnableBatchProcessing
```

## QianFeng Video

![](<../.gitbook/assets/image (235).png>)

* Chunk based processing
* Declarative I/O
* start/stop/restart
* retry/skip

```java
@Configuration
@EnableBatchProcessing
public class JobConfiguration {

    //use task to enable batch job
    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    //use steps to execute the task
    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    //create tasks
    @Bean
    public Job helloWorldJob(){
        return jobBuilderFactory.get("helloWorldJob")
                .start(step1())
                .build();
    }

    @Bean
    public Step step1(){
        return stepBuilderFactory.get("step1")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(
                            StepContribution stepContribution,
                            ChunkContext chunkContext) throws Exception {
                        //real task here
                        System.out.println("this is the real task" + " hello world");
                        //need to return this step
                        return RepeatStatus.FINISHED;
                    }
                }).build();
    }
}
```

![ 与数据库相连需要一些settings](<../.gitbook/assets/image (236).png>)

![default auto gen those tables](<../.gitbook/assets/image (237).png>)

### Step:

itemReader / itemProcessor / itemWriter

**JobInstant**: 每次执行都有一个instance，但是不一定成功

**JobExecution**: 代表job执行过程，同一个job重新执行都有新的execution

**JobParamters**: 同一个任务可以有不同的JobInstance

**JobStep**: 每一个Job有不同的steps

**ExecutionContext**: 执行上下文，期间可以共享step，也可以获取一些数据，共享一些数据。

```java
@Configuration
@EnableBatchProcessing
public class JobDemo {

    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job jobDemoJob(){
        return jobBuilderFactory.get("jobDemoJob")
                .start(step1())
                .next(step2())
                .next(step3())
                .build();
    }

    @Bean
    public Step step1(){
        return stepBuilderFactory.get("step1")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                        System.out.println("Step1");
                        return RepeatStatus.FINISHED;
                    }
                }).build();
    }
}



//自由组合
return jobBuilderFactory.get("jobDemoJob")
        .start(step1())
        .on("COMPLETED")
        .to(step2())
        .from(step2())
        .on("COMPLETED")
        .stopAndRestart(step3())
        .end().build();
```

### Flow:

 **多个step的集合，**可以被多个Job复用。使用**FlowBuilder**来创建。

 将step放在flow里面，然后flow再放进job里面，同样的道理。

![执行完的job数据会写进DB，再次执行的时候已经completed，无法进行](<../.gitbook/assets/image (238).png>)



### Split: 并发执行

```java
//concurently execute, not ordered
      
@Bean
public Job splitDemoJob(){
return jobBuilderFactory.get("splitDemoJob")
        .start(splitDemoFlow1())
        .split(new SimpleAsyncTaskExecutor())
        .add(splitDemoFlow2())
        .end().build();
}
```

### JobExecutionDecider: 决策器

implements JobExecutionDecider

```java
public class MyDecider implements JobExecutionDecider {
    
    private int count;
    
    @Override
    public FlowExecutionStatus decide(JobExecution jobExecution, StepExecution stepExecution) {
        count++;
        if(count % 2 == 0)
            return new FlowExecutionStatus("even");
        else
            return new FlowExecutionStatus("odd");
    }
}


//inside configuration file
@Bean
public JobExecutionDecider myDecider(){
    return new MyDecider();
}

@Bean
public Job deciderDemoJob(){
    return jobBuilderFactory.get("deciderDemoStep")
            .start(splitDemoStep2())
            .next(myDecider())
            .from(myDecider()).on("even").to(splitDemoStep1())
            .from(myDecider()).on("odd").to(splitDemoStep3())
            .end().build();
}


```

### Job的嵌套，child job依赖parent job to execute

 JobStepBuilder(new StepBuilder("childJob2"))

```java
@Autowired
private Job childJobOne;

@Autowired
private JobLauncher launcher;

private Step childJob2(JobRepository repository, PlatformTransactionManager manager){
    return new JobStepBuilder(new StepBuilder("childJob2"))
            .job(childJobOne)
            .launcher(launcher)
            .repository(repository)
            .transactionManager(manager)
            .build();
}


#properties files
spring.batch.job.names=parentJob
```





























































