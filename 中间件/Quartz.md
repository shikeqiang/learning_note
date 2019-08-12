# 一、简介

​	任务调度框架，主要组件是调度器、任务和触发器(生效时间)，

- JobDetail：实现类和业务逻辑信息，通过JobBuilder的newJob方法创建一个任务；
- Trigger：触发器，时间触发规则，主要有SimpleTrigger(定频率执行任务)和CronTrigger(可以执行更复杂的任务)
- Scheduler：调度器，主要有start等方法,与调度程序交互的主要API。
- Job：接口，只有一个execute方法，
- JobBuilder：定义或者创建JobDetail实例，即Job实例
- JobStore：保存Job数据
- Calendar：一个Trigger可以喝多个Calendar关联，以排除或包含某些时间点；
- 监听器：JobListener、TriggerListener、SchedulerListener;

主要用到的设计模式

Builder、Factory、组件模式、链式写法。

# 二、主要成员

### Job和JobDetail

​	Job接口只有一个execute方法，在里面编写业务逻辑；

```java
void execute(JobExecutionContext context) throws JobExecutionException;
```

Job实例在Quartz中的生命周期

​	每次调度器执行job时，它在调用execute方法前会创建一个新的job实例，当调用完成后，关联的job对象实例会被释放，释放的实例会被垃圾回收机制回收。

​	JobDetail为Job实例提供了很多属性，以及jobDataMap成员变量属性，它用来存储特定Job实例的状态信息，调度器需要借助JobDetail对象来添加Job实例。

JobDetail重要属性

- name
- jobClass
- group
- jobDataMap

```java
public class HelloScheduler {
    public static void main(String[] args) throws SchedulerException {
      // 创建一个JobDetail实例，将该实例与HelloJob Class绑定,需要指定唯一标识Identity，链式写法
        JobDetail jobDetail= JobBuilder.newJob(HelloJob.class)
                .withIdentity("myJob","group1").build();
        System.out.println("jobDetail's name:"+jobDetail.getKey().getName());
        System.out.println("jobDetail's group:"+jobDetail.getKey().getGroup());
        System.out.println("jobDetail's jobClass:"+jobDetail.getJobClass().getName());
        // 创建一个Trigger实例，定义该job立即执行，并且每隔两秒钟重复执行一次，直到永远
        Trigger trigger= (Trigger) TriggerBuilder
                .newTrigger()
                .withIdentity("myTrigger","group1") // 唯一标识
                .startNow() //立即执行
                .withSchedule(
                        SimpleScheduleBuilder.simpleSchedule()
                        .withIntervalInSeconds(2).repeatForever())
                .build();
        // 创建Scheduler实例,通过工厂模式创建
        SchedulerFactory factory=new StdSchedulerFactory();
        Scheduler scheduler=factory.getScheduler();
        scheduler.start();
        // 打印当前的执行时间
        Date date = new Date();
        SimpleDateFormat sf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println("HelloScheduler 当前时间：" + sf.format(date));
        scheduler.scheduleJob(jobDetail,trigger);
    }
}
```

### JobExecutionContext

​	当Scheduler调用一个Job，就会将JobExecutionContext传递给Job的execute()方法，**Job能通过JobExecutionContext对象访问到Quartz运行时候的环境以及Job本身的明细数据**。想要传参就要通过JobExecutionContext进行。

### JobDataMap

​	在运行任务调度时，JobDataMap存储在JobExecutionContext中，非常方便获取；JobDataMap可以用来装载任何可序列化的数据对象，当job实例对象被执行时这些参数对象会传递给它，JobDataMap实现了JDK的Map接口，并且添加了一些非常方便的方法用来存取基本数据类型。

获取JobDataMap的两种方式：

- 从Map中直接获取(先在JobDetail和Trigger中定义，然后通过JobExecutionContext获取)
- 定义三个成员变量，这三个成员变量对应Scheduler中的三个key，通过setter和jobExecutionContext.getMergedJobDataMap()获取

### Trigger

​	Quartz的触发器，用来告诉调度程序作业，什么时候触发，即Trigger对象是用来触发执行Job的。

触发器通用属性：

- JobKey

  表示Job实例的标识，触发器被触发时，该指定的Job实例会被执行，可以通过JobKey获取跟这个Trigger绑定到一起的JobDetail信息。

- StartTime

  表示触发器的时间表首次被触发的时间，它的值的类型是Java.util.Date.

- EndTime

  表示触发器的不在被触发的时间，它的值的类型是Java.util.Date.

#### SimpleTrigger

​	**在一个指定时间段内执行一次作业任务，或是在指定的时间间隔内多次执行作业任务。**

- 重复次数可以为0，正整数或是SimpleTrigger.REPEAT_INDEFINITELY

- 重复执行间隔必须为0或长整数
- 一旦被指定了endTime参数，那么它会覆盖重复次数参数的效果

```java
SimpleTrigger trigger = (SimpleTrigger) TriggerBuilder.newTrigger()
        .withIdentity("mySimpleTrigger", "group1")
        .startAt(date)      // 时间优先于次数
        .withSchedule(SimpleScheduleBuilder
                .simpleSchedule().withIntervalInSeconds(2)      // 每隔2秒
                .withRepeatCount(SimpleTrigger.REPEAT_INDEFINITELY))    // 执行无数次,可以修改为具体的次数
        .build();
```

#### CronTrigger

​	基于日历的作业调度器(比如：每个月的某天某时某分执行任务)，而不是像SimpleTrigger那样精确指定间隔时间，比SimpleTriigger更常用。

##### Cron表达式

​	用于配置CronTrigger实例，是由7个子表达式组成的字符串，描述了时间表的详细信息。

​	格式：\[秒] \[分] \[小时] [日] \[月] \[周] [年]

![img_0903](/Users/jack/Desktop/md/images/img_0903.png)

![img_0904](/Users/jack/Desktop/md/images/img_0904.png)

> \#  表示第几的意思, 逗号 表示或 ，/ 表示每隔多久触发

![img_0905](/Users/jack/Desktop/md/images/img_0905.png)

- 'L'和'W'可以一起组合使用
- 周字段英文字母不区分大小写，即MON与mon相同
- [在线工具](http://cron.qqe2.com/)

### Scheduler-工厂模式

​	所有Scheduler实例都是由SchedulerFactory来创建。

![img_0906](/Users/jack/Desktop/md/images/img_0906.png)

![img_0907](/Users/jack/Desktop/md/images/img_0907.png)



#### StdSchedulerFactory

- 使用一组参数(Java.util.Properties)来创建和初始化Quartz调度器
- 配置参数一般存储在quartz.properties中
- 调用getScheduler方法就能创建和初始化调度器对象

#### Scheduler的主要函数

- Date scheduleJob(JobDetail jobDetail, Trigger trigger) throws SchedulerException;
- void start()
- void standby()        调用之后可以重启
- void shutdown()    调用之后无法重启，可以传入true/false，当传入参数为true时，表示等待所有正在执行的job执行完毕后，再关闭scheduler。传入false时表示直接关闭scheduler

### quartz.properties

​	如果项目中没有自定义quartz.properties，则会读取jar包中的quartz.properties。

#### 组成部分

- 调度器属性

  > org.quartz.scheduler.instanceName: 用来区分特定的调度器实例，可以按照功能用途来给调度器起名
  >
  > org.quartz.scheduler.instanceId: 和上面的一样，也允许任何字符串，但这个值必须是在所有调度器实例中是唯一的，尤其是在一个集群当中，作为集群的唯一key。假如你想Quartz帮你生成这个值的话，可以设置为AUTO。

- 线程池属性

  > threadCount：线程大小，必须大于0
  >
  > threadPriority：线程的优先级
  >
  > org.quartz.threadPool.class：可以自定义实现线程池，配置到这个属性

- 作业存储设置

  描述了在调度器实例的生命周期中，Job和Trigger信息是如何被存储的，是保存到内存或者DB

- 插件配置

  满足特定需求用到的Quartz插件的配置。

默认的配置文件：

```properties
org.quartz.scheduler.instanceName: DefaultQuartzScheduler
org.quartz.scheduler.rmi.export: false
org.quartz.scheduler.rmi.proxy: false
org.quartz.scheduler.wrapJobExecutionInUserTransaction: false

org.quartz.threadPool.class: org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount: 10
org.quartz.threadPool.threadPriority: 5
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread: true

org.quartz.jobStore.misfireThreshold: 60000

org.quartz.jobStore.class: org.quartz.simpl.RAMJobStore
```









参照：[慕课Quartz课程](https://www.imooc.com/learn/846)



