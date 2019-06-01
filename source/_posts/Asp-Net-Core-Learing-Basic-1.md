---
title: Asp .Net Core Learing (Basic 1)
date: 2019-06-02 00:58:53
categories:
- ASP.NET CORE
tags:
- ASP.NET CORE
---
# Asp.Net Core (Base )
在Asp.Net.Core中 ，一般創建Web 程式需要一個Host ,`CreateWebHostBuilder`這個方法就能創立一個WebHostBuilder ，
`Build()`完之後Return 一個實現IWebHost接口的實例 (WebHostBuilder)
然後`Run()`就會運行Web程序，直到程式關閉。

```
public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        //表示在程序啟動的時候,我們會調用Startup這個類別.
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>();
}
```
