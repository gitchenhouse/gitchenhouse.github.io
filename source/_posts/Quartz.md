---
title: Quartz
date: 2019-06-11 23:40:40
tags: Job
---
# Quartz.NET
Quartz.NET是一個開源的作業排程框架，是OpenSymphony 的 Quartz API的.NET移植，它用C#寫成，可用於winform和asp.net應用中。它提供了巨大的靈活性而不犧牲簡單性。你能夠用它來為執行一個作業而建立簡單的或複雜的排程。它有很多特徵，如：資料庫支援，叢集，外掛，支援cron-like表示式等等。
在Quartz.NET有一個叫做quartz.properties的配置檔案，它允許你修改框架執行時環境。預設是使用Quartz.dll裡面的quartz.properties檔案

##  Quartz.Net的五大構件
1. 調度器：Scheduler
2. 作業任務：Job
3. 觸發器： Trigger
4. 線程池： SimpleThreadPool
5. 作業持久化：JobStore

1. 創建Scheduler的兩種方式
(1). 直接通過`StdSchedulerFactory`類的`GetDefaultScheduler`方法創建
(2).先創建`StdSchedulerFactory`,然後通過`GetScheduler`方法創建.該方式可以在實體化`StdSchedulerFactory`的時候配置一些額外的信息，比如：配置SimpleThreadPool的個數、RemoteScheduler的遠程控制、數據庫的持久化等。

```csharp
var scheduler = await new StdSchedulerFactory(quartzSettings).GetScheduler();
var scheduler = await StdSchedulerFactory.GetDefaultScheduler();
```

2. Scheduler的簡單封裝
　　這裡提供兩種思路，一種是單例的模式封裝，另一種是利用線程槽的模式封裝
　　(1). 單例模式：是指無論多少個用戶訪問，都只有一個實例，在web端上常用（詳見：MySchedulerFactory類）
　　(2). 線程槽模式：是指單個用戶的單次鏈接，在未斷開連接之前，只有一個實例，下次重新連接，實例將重新創建（詳見：MySchedulerFactory2類）
3. Scheduler的基本方法：

(1). 開啟：Start

(2). 關閉：ShutDown

(3). 暫停job或Trigger：PauseAll、PauseJob、PauseJobs、PauseTrigger、PauseTriggers

(4). 恢復job或Trigger：ResumeAll、ResumeJob、ResumeJobs、ResumeTrigger、ResumeTriggers

(5). 將job和trigger加入Scheduler中：ScheduleJob

(6). 添加Job：AddJob
### Job詳解
1. 幾個類型
　　①. JobBuilder：用來創建JobDetail。

　　②. IJob：具體作業任務需要實現該接口，並實現裡面的方法

　　③. IJobDetail：用來定義工作實例

2. job的創建有兩種形式：

　　①.Create的泛型方式：寫起來代碼簡潔方便。

　　②.反射+OfType的方式：用於後期動態綁定，通過程序集的反射


```csharp
1               // 1 (Create的泛型方式) 
2              IJobDetail job1 =JobBuilder.Create<HelloJob2>()
3                      .UsingJobData( " name " , " ypf " )
4                      .UsingJobData( " age " , " 12 " )
5                      .WithIdentity ( " job1 " , " myJob1 " )
6                      .WithDescription( " 我是用來對該job進行描述的")
7                     .StoreDurably( true )
8                      .Build();
9  
10              // 2 (反射+OfType的方式)
11              // 通過反射來創建類
12              var type = Assembly.Load( " QuartzDemo " ).CreateInstance( " QuartzDemo.HelloJob2 " );
13              // OfType的方式加載類型
14              IJobDetail job2 = JobBuilder.Create().OfType(type.GetType())
15                                      .UsingJobData( " name " , " ypf " )
16                                      .UsingJobData( " age " , " 12 " )
17                                      .StoreDurably( true )
18                                      .Build();
```
### 常用的幾個方法

　　①.UsingJobData：給Job添加一些附加值，存儲在JobDataMap裡，可以在具體的Job中獲取。（通過context.JobDetail.JobDataMap獲取）

　　②.StoreDurably：讓該job持久化，不被銷毀.(默認情況下為false，即job沒有對應的trigger的話，job就被銷毀)

　　③.WithIdentity：身份標記，給job起個名稱，便於和Trigger關聯的時候使用.

　　④.WithDescription：用來對job進行描述，並沒有什麼實際作用

```csharpß

       ///  <summary> 
        /// Job詳解
         ///  </summary> 
        public  static  void JobShow()
        {
            // 1.創建Schedule 
            IScheduler scheduler = StdSchedulerFactory.GetDefaultScheduler();
            
            // 2.創建Job
             // 2.1 (Create的泛型方式) 
            IJobDetail job1 = JobBuilder.Create<HelloJob2> ()
                    .UsingJobData( " name " , " ypf " )
                    .UsingJobData( " age " , " 12 " )
                    .WithIdentity( " job1 " , " myJob1 " )
                    .WithDescription( " 我是用來對該job進行描述的" )
                    .StoreDurably( true )
                    .Build();

            // 2.2 (反射+OfType的方式)
             // 通過反射來創建類
            var type = Assembly.Load( " QuartzDemo " ).CreateInstance( " QuartzDemo.HelloJob2 " );
             // OfType的方式加載類型 
            IJobDetail job2 = JobBuilder. Create().OfType(type.GetType())
                                    .UsingJobData( " name " , " ypf " )
                                    .UsingJobData( " age " , " 12 " )
                                    .StoreDurably( true )
                                    .Build();
            IJobDetail job3 = JobBuilder.Create(type.GetType())
                                 .UsingJobData( " name " , " ypf " )
                                 .UsingJobData( " age " , " 12 " )
                                 .StoreDurably( true )
                                 .Build();

            // 3.創建Trigger 
            ITrigger trigger = TriggerBuilder.Create().WithSimpleSchedule(x => x.WithIntervalInSeconds( 1 ).RepeatForever()).Build();

            // 4.將Job和Trigger加入調度器中
             // scheduler.ScheduleJob(job1, trigger);
             // scheduler.ScheduleJob(job2, trigger); 
            scheduler.ScheduleJob(job3, trigger);

            // 5.開始調度
            scheduler.Start();
        }
     ///  <summary> 
    /// 實現IJob接口
     ///  </summary> 
    class HelloJob2 : IJob
    {
        void IJob.Execute(IJobExecutionContext context)
        {
            var name = context.JobDetail.JobDataMap[ " name " ];
             var age = context.JobDetail.JobDataMap[ " age " ];

            Console.WriteLine( " name值為：{0}，age值為：{1} " , name, age);
        }
    }
```