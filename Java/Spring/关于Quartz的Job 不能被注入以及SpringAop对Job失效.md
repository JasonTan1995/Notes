### 关于Quartz的Job 不能被注入以及SpringAop对Job失效

#### Problem(问题)

​        最近在工作遇到需要对Quartz的Job进行异常后将异常记录到数据库的操作，第一反应就想到了使用Spring的AOP，利用AfterThrowing来完成这个操作。理想是美好的，但现实却是骨感的。研究了好久都不生效。研究的过程发现居然还不能依赖注入，注入到的testService是空的。

​        切面类（Aspect）

```java
@Aspect
public class ErrorLogAop {

    @Pointcut("execution(public * com.aspect.quartzdemo.job.*.*(..))")
    //@Pointcut("execution(public * com.aspect.quartzdemo.service.*.*(..))")
    public void joinPointExpression(){}


    @Before("joinPointExpression()")
    public void before(){
        System.out.println("start before");
    }

}
```

​      Job(任务类)

```java
public class TimeJob implements Job {

    @Autowired
    private TestService testService;
    
    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        testService.test();
        System.out.println(new Date());
    }
}
```



#### Environment（环境）

​     该篇文章围绕的是Springboot与Quartz一起使用时发生的问题。

​    JDK： 1.8

​     SpringBoot：2.0.4.RELEASE

​     Quartz：spring-boot-starter-quartz（由于Springboot2.0后官方主动与Quartz整合了，并推出了启动器，所以不需要以前那么繁琐的配置了，什么工厂什么实例一堆的。现在Scheduler，Springboot直接帮我们生成好了，我们只需要依赖注入就可以了。）



####  Solution(解法)

​     最后根据几个大神的帮助，解决了这个问题。主要是因为Job的实例是由Quartz进行管理的，因而Spring管理的实例并不能在Quartz实例Job的过程进行任何操作。下面介绍两种解决的方案。

#####    1. 使用JobListener监听任务内的操作

> Listeners are objects that you create to perform actions based on events occurring within the scheduler. As you can probably guess, **TriggerListeners** receive events related to triggers, and **JobListeners** receive events related to jobs.
>
> Trigger-related events include: trigger firings, trigger mis-firings (discussed in the “Triggers” section of this document), and trigger completions (the jobs fired off by the trigger is finished).

官方文档的意思大概是:监听器是在调度器中基于事件机制执行操作的对象，我们大概可以猜到，触发监听器是接收跟触发器相关的事件，任务监听器是跟接收跟任务有关的事件。

### org.quartz.TriggerListener implement：

​    跟触发器有关的事件包括：触发器被触发，触发器触发失败以及触发器触发完成（触发器完成后作业任务开始运行）

```java
public interface TriggerListener {  

    public String getName();  

    public void triggerFired(Trigger trigger, JobExecutionContext context);  

    public boolean vetoJobExecution(Trigger trigger, JobExecutionContext context);  

    public void triggerMisfired(Trigger trigger);  

    public void triggerComplete(Trigger trigger, JobExecutionContext context,  
    int triggerInstructionCode);
}  
```

### org.quartz.JobListener implement：

   跟作业任务相关的事件：job即将被执行的通知和job执行完成的通知事件（其中包括出现异常的通知，而且还能捕获到异常的信息）这个就很符合我的想法了^_^。

 ```java
public interface JobListener {  

    public String getName();  

    public void jobToBeExecuted(JobExecutionContext context);  

    public void jobExecutionVetoed(JobExecutionContext context);  

    public void jobWasExecuted(JobExecutionContext context,  
    JobExecutionException jobException);  

}  
 ```

那么怎么使用自定义的监听器呢？来看看官方文档是怎么说的

> To create a listener, simply create an object that implements the org.quartz.TriggerListener and/or org.quartz.JobListener interface. Listeners are then registered with the scheduler during run time, and must be given a name (or rather, they must advertise their own name via their getName() method).
>
> For your convenience, tather than implementing those interfaces, your class could also extend the class JobListenerSupport or TriggerListenerSupport and simply override the events you’re interested in.

意思大概是：创建监听器只需要实现JobListener或者TriggerListener接口就能实现了，监听器会向调度器进行注册，而且还必须给监听器一个名字。（它会自动在getName（）这个方法中获取）。

为了方便我们，我们可以选择只继承JobListenerSupport或TriggerListenerSupport来重写我们所需要的方法。

但是官方给出来的例子比较模糊，我提供一个算是比较简单但又比较全的栗子吧。

##### Implement JobListener interface:(Create a Class implement JobListener interface)

```java 
public class MyJobListener implements JobListener {

    @Override
    public String getName() {
        return "myJobListener";
    }

    @Override
    public void jobToBeExecuted(JobExecutionContext context) {
        System.out.println("start Job");
    }

    @Override
    public void jobExecutionVetoed(JobExecutionContext context) {
        System.out.println("Job was executed");
    }

    @Override
    public void jobWasExecuted(JobExecutionContext context, JobExecutionException jobException) {
        System.out.println("Job is Done");
        System.out.println("Job is Error");
    }
}
```

##### JobConfiguration:(Listeners are  registered with the scheduler during run time)

```java
@Configuration
public class JobConfig {

    @Autowired private Scheduler scheduler;

    @PostConstruct
    public void addListener() throws SchedulerException {
        MyJobListener myJobListener = new MyJobListener();
        // Add the All Job to the Scheduler(全部)
        scheduler.getListenerManager().addJobListener(myJobListener, allJobs());
        // Adding a JobListener that is interested in all jobs of a particular group（一个）
        scheduler.getListenerManager().addJobListener(myJobListener, jobGroupEquals("myJobGroup"));
        // Adding a JobListener that is interested in all jobs of two particular groups（两个）
scheduler.getListenerManager().addJobListener(myJobListener,or(jobGroupEquals("myJobGroup"),                   jobGroupEquals("yourGroup")));
    }

    @Bean
    public JobDetail myJobDetail() {
        return JobBuilder.newJob(MyJob.class).withIdentity("MyJob")
                .storeDurably().build();
    }

    @Bean
    public Trigger myJobTrigger() {
  CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder.cronSchedule("0/2 * * * * ? *");
return TriggerBuilder.newTrigger().forJob(myJobDetail()).withIdentity("MyTrigger").withSchedule(cronScheduleBuilder).build();
    }
}    
```

运行，看效果，收工。TriggerListener也是一样这么使用的。

> Listeners are not used by most users of Quartz, but are handy when application requirements create the need for the notification of events, without the Job itself having to explicitly notify the application.

官方也说其实很少人使用（哈哈哈）。我就用到了，确实也挺便利的。

#####  2. 把Job的实例交给Spring管理

​       当我看到这个解决方案的时候，我深深的感觉到自己对于Spring知识的薄弱了。来看看吧。

       - Quartz提供了JobFactory接口，让我们可以自定义实现创建Job的逻辑。
       - 通过实现JobFactory接口，在实例化Job以后，在通过ApplicationContext将Job所需要的属性注入即可。
       - 在Spring与Quartz集成时，用到了org.springframework.scheduling.quartz.SchedulerFactoryBean这个类。

  ```java
// Get Scheduler instance from SchedulerFactory.
        try {
            this.scheduler = createScheduler(schedulerFactory, this.schedulerName);
            populateSchedulerContext();

            if (!this.jobFactorySet && !(this.scheduler instanceof RemoteScheduler)) {
                // Use AdaptableJobFactory as default for a local Scheduler, unless when
                // explicitly given a null value through the "jobFactory" bean property.
              |--  this.jobFactory = new AdaptableJobFactory(); --|
                  //这里是重点。
            }
            if (this.jobFactory != null) {
                if (this.jobFactory instanceof SchedulerContextAware) {
                    ((SchedulerContextAware) this.jobFactory).setSchedulerContext(this.scheduler.getContext());
                }
                this.scheduler.setJobFactory(this.jobFactory);
            }
        }
  ```

- 由于将Scheduler交给Spring生成， SchedulerFactoryBean有个jobFactory属性 而且jobFactory是实现SchedulerContextAware的类还要继承AdaptableJobFactory。
- 在Spirng-context-support  jar包下org.springframework.scheduling.quartz包中有个SpringBeanJobFactory的类继承了AdaptableJobFactory实现AdaptableJobFactory，spring会默认使用这个给jobFactory，我们可以继承SpringBeanJobFactory重写他的createJobInstance方法


```java
public class JobBeanFactory extends SpringBeanJobFactory {

    @Nullable
    private String[] ignoredUnknownProperties;

    @Nullable
    private SchedulerContext schedulerContext;


    private final BeanFactory beanFactory;


    JobBeanFactory(BeanFactory beanFactory){
        this.beanFactory = beanFactory;
    }


    public void setIgnoredUnknownProperties(String... ignoredUnknownProperties) {
        this.ignoredUnknownProperties = ignoredUnknownProperties;
    }

    @Override
    public void setSchedulerContext(SchedulerContext schedulerContext) {
        this.schedulerContext = schedulerContext;
    }


    @Override
    protected Object createJobInstance(TriggerFiredBundle bundle) throws Exception {
        Class<?> jobClass = bundle.getJobDetail().getJobClass();
        Object job = beanFactory.getBean(jobClass);
        if (isEligibleForPropertyPopulation(job)) {
            BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(job);
            MutablePropertyValues pvs = new MutablePropertyValues();
            if (this.schedulerContext != null) {
                pvs.addPropertyValues(this.schedulerContext);
            }
            pvs.addPropertyValues(bundle.getJobDetail().getJobDataMap());
            pvs.addPropertyValues(bundle.getTrigger().getJobDataMap());
            if (this.ignoredUnknownProperties != null) {
                for (String propName : this.ignoredUnknownProperties) {
                    if (pvs.contains(propName) && !bw.isWritableProperty(propName)) {
                        pvs.removePropertyValue(propName);
                    }
                }
                bw.setPropertyValues(pvs);
            }
            else {
                bw.setPropertyValues(pvs, true);
            }
        }
        return job;
    }
}
```

当Spring在加载配置文件时，如果配置文件中有Bean实现了ApplicationContextAware接口时,Spring会自动调用setApplicationContext方法,我们可以通过这个获取Spring上下文然后在创建Job时让Job自动注入到Spring容器中

在JobConfig中:

```java
@Configuration
public class JobConfig implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    @Bean
    public JobDetail myJobDetail() {
        return JobBuilder.newJob(MyJob.class).withIdentity("MyJob")
                .storeDurably().build();
    }

    @Bean
    public Trigger myJobTrigger() {
        CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder.cronSchedule("0/2 * * * * ? *");
        return TriggerBuilder.newTrigger().forJob(myJobDetail()).withIdentity("MyTrigger").withSchedule(cronScheduleBuilder).build();
    }
    
     @Bean
    public SchedulerFactoryBeanCustomizer schedulerFactoryBeanCustomizer() {
        return schedulerFactoryBean -> schedulerFactoryBean.setJobFactory(new JobBeanFactory(applicationContext));
    }


    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

之后就可以使用Spring的AOP了(记得把切面类也要交给Spring管理)。

```java
@Aspect
@Component
public class ErrorLogAop {

    @Pointcut("execution(public * com.aspect.quartzdemo.job.*.*(..))")
    //@Pointcut("execution(public * com.aspect.quartzdemo.service.*.*(..))")
    public void joinPointExpression(){}


    @Before("joinPointExpression()")
    public void before(){
        System.out.println("start before");
    }

}
```

Job也需要加上注解@Component

```java
@Component
public class MyJob implements Job {


    @Override
    public void execute(JobExecutionContext context) {
        System.out.println("hello");
        System.out.println(this);

    }
}
```

运行，看效果，收工（哈哈）。

#### 总结

​        首先感谢提供解决方案的各位大神，这个问题让我觉得自己还有很多的不足，很多知识都是一知半解会用就行的感觉，继续努力吧。

参考资料：

https://blog.csdn.net/a67474506/article/details/38402059

https://www.cnblogs.com/daxin/p/3608320.html

https://xuzongbao.gitbooks.io/quartz/content/