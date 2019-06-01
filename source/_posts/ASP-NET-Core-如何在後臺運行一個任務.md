---
title: ASP.NET Core . 如何在後臺運行一個任務
date: 2019-05-27 00:45:03
tags:
- C#
categories:
- ASP.NET CORE
---

# ASP.NET Core . 如何在後臺運行一個任務
<!--more-->
在大部分程序中一般都會需要用到後臺任務， 比如定時更新緩存或更新某些狀態.

以調用微信公眾號的Api為例， 經常會用到access_token，官方文檔這樣描述：“是公眾號的全局唯一接口調用憑據，有效期目前為2個小時，需定時刷新，重復獲取將導致上次獲取的access_token失效，建議公眾號開發者使用中控服務器統一獲取和刷新Access_token，其他業務邏輯服務器所使用的access_token均來自於該中控服務器，不應該各自去刷新，否則容易造成沖突，導致access_token覆蓋而影響業務。

Microsoft示例提供了有關此常見使用模式的幾個示例IHostedService。

* 定時後台任務
* 在後台任務中使用範圍服務
* 排隊的後台任務

## 實現方式
　　ASP.NET Core 在2.0的時候就提供了一個名為IHostedService的接口，我們要做的只有兩件事：

　　　　1. 實現它。

　　　　2. 將這個接口實現註冊到依賴註入服務中。

### 實現IHostedService的接口

由以下程式碼 ，一個是這個服務啟動的時候做的操作，另一個則是停止的時候。

```csharp
public interface IHostedService
{
    Task StartAsync(CancellationToken cancellationToken);
    Task StopAsync(CancellationToken cancellationToken);
}
```

在 ASP.NET Core 2.1中， 提供了一個名為 BackgroundService 的類，它在 Microsoft.Extensions.Hosting 命名空間中 .
是繼承自 IHostedService, IDisposable ， 它相當於是幫我們寫好了一些“通用”的邏輯， 而我們只需要繼承並實現它的 ExecuteAsync 即可。


###實現Cache 背景執行範例 ：
```
public class XXXCache : BackgroundService
{
    private readonly IMemoryCache _memoryCache;
   

    public XXXCache(IMemoryCache memoryCache)
    {
        _memoryCache = memoryCache;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        LoggerHelper.Instance.Info(“Cache is starting.”);

        stoppingToken.Register(() =>
            LoggerHelper.Instance.Info(“Cache is stopping.”));

        while (!stoppingToken.IsCancellationRequested)
        {
            LoggerHelper.Instance.Info(“Cache task doing background update.”);
            var data = await GetCurrencyFromDomainService();
             _memoryCache.Set(key, data);

            await Task.Delay(TimeSpan.FromSeconds(1000);
        }

        LoggerHelper.Instance.Info($”Cache background task is stopping.”);
    }

public override void Dispose()
{
    _memoryCache.Dispose();
}

```

在Startup的ConfigureServices中註冊這個服務，如下代碼所示：

```
#region Register MemoryCache

services.AddHostedService<XXXCache>();

#endregion
```


[Microsoft 微服務中使用 IHostedService 和 BackgroundService 類別實作背景工作](https://docs.microsoft.com/zh-tw/dotnet/standard/microservices-architecture/multi-container-microservice-net-applications/background-tasks-with-ihostedservice)