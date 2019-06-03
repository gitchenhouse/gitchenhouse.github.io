---
title: Asp .Net Core Learing (Basic 1)
date: 2019-06-02 00:58:53
categories:
- ASP.NET CORE
tags:
- ASP.NET CORE
---
# Asp.Net Core 
在Asp.Net.Core中，一般創建Web 程式需要一個Host ,`CreateWebHostBuilder`這個方法就能創立一個WebHostBuilder ，`Build()`完之後Return 一個實現IWebHost接口的實例 (WebHostBuilder)
然後`Run()`就會運行Web程序，直到程式關閉。

```csharp
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

Asp.Net Core 的執行順序:  Main -> ConfigureServices -> Configure
`StartUp` 程式真正的切入點

 1. ConfigureServices
ConfigureServices方法，把各種服務 Register 到Container中，並配置Services，Container 用Dependency Injection 方式注入Service。
2.  Configure 
Configure 是asp.net core程序用來具體指定如何處理每個http請求的,例如我們可以讓這個程序知道我使用mvc來處理http請求,那就調用app.UseMvc()這個方法就行，參數的實例都是從 WebHost 注入進來，可依需求增減需要的參數。
## Request Pipeline, Middleware
* Request Pipeline : 處理http requests並返回responses的代碼就組成了request pipeline
* MiddleWare : 我們可以做的就是使用一些程序來配置那些請求管道request pipeline以便處理requests和responses。每層中間件接到請求後都可以直接返回或者調用下一個中間件。 比如處理驗證( authentication)的程序,Middleware 就可以作為攔截使用 。一個比較好的例子就是: 在第一層調用authentication驗證中間件, 如果驗證失敗, 那麼直接返回一個表示請求未授權的response 。
```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler();
    }
    app.UseMvc();
}
```
1.app.UseDeveloperExceptionPage() ;就是一個middleware,當exception發生的時候,這段程序就會處理它.而判斷`env.isDevelopment()` 表示,這個middleware只會在Development環境下被調用.
2.處理異常的middleware後邊調用app.UseMvc(),所以處理異常的middleware可以在把request交給mvc之間就處理異常,更重要的是它還可以捕獲並處理返回MVC相關代碼執行中的異常.

[image:AC9CA58B-8D81-4998-9104-B2DCAE8BA009-280-00001139E73D94C2/螢幕快照 2019-06-02 下午2.31.42.png]

## Controller
Web api推薦使用attribute-based  的Controller. 
這種基於屬性配置的路由可以配置Controller或者Action級別, URI會根據Http method然後被匹配到一個controller裡具體的action上. 
常用的Http Method有: 
Get查詢
POST創建
PUT整體修改更新
PATCH部分更新
DELETE刪除

用`[Route("api/[controller]")]` ,它使得整個Controller下面所有action的URI前綴變成了"/api/product",其中`[controller]`表示xxxController.cs中的xxx.
 [.NET Core gRPC with Visual Studio 2019 and .NET Core 3 | CK’s Notepad](https://blog.kevinyang.net/2019/04/08/grpc-chat-server/)
