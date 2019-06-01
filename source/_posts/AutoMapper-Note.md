---
title: AutoMapper Note
date: 2019-05-26 02:00:28
tags: C#
categories:
- C# AutoMapper
---
 #AutoMapper 類別之間轉換
C# 在物件類別之間轉換是很常發生的，例如Model 轉 DB 的 Entity , View 轉 Model 等等 ，這些常在物件轉換很常發生，AutoMapper可以輕易解決，轉換型別的問題 。

<!--more-->

 ##1.簡易Ｍapper物件
首先，您需要來源和目標類型。目標類別的設計可能受其所在層的影響，但只要成員的名稱與來源類型的成員匹配，AutoMapper就能發揮最佳效果。如果您有一個名為“FirstName”的來源成員，則會自動將其映射到名為“FirstName”的目標成員。
```csharp
public class TestCass
{
    public string FirstName { get; set; }
}

public class Test2Cass
{
    public string FirstName { get; set; }
}
```

通常的做法為 ：
```csharp
TestCass.FirstName = Test2Cass.FirstName;
```

使用AutoMapper轉化
```csharp
public static void Main(string[] args)
{
    //初始化AutoMapper
    Mapper.Initialize(cfg =>cfg.CreateMap<TestCass,Test2Cass>());
    //設定屬性
    var class1 = new TestCass();
    class1.FirstName = “123”;
    //利用AutoMapper 轉型
    var class2= Mapper.Map<TestCass, Test2Cass>(class1);
    
    // 印出 ： 123
    Console.WriteLine(class2.FirstName);
}
```

左側的類型是源類型，右側的類型是目標類型。要執行映射，請使用靜態或實例Mapper方法，具體取決於靜態或實例初始化：

 ##2.設定Profile
 透過 Profile 可隔離不同的 Class，我們應該要把這些對應關係給集中管理，設定一次所有的開發人員都能夠用，AutoMapper 也有提供集中管理的機制
 很簡單，只要實作 AutoMapper.Profile 並覆寫 Configure 方法，把相關的對應關係擺好

```csharp
public class XXXProfile : Profile
{
    public XXXMappingProfile()
    {
// 設定Config
AddMemberConfiguration().AddName<SourceToDestinationNameMapperAttributesMember>();

            //CreateMap<Souce, Destination>();
            CreateMap<Model, Response>()
            .ForMember(Destination => Destination.Id, option => ption.MapFrom(source => source.Id))
            CreateMap<Response, Model>()
            .ForMember(Destination => Destination.Id, option => option.MapFrom(source => source.Id))
    }
}
```

 ##3.指定 Property 的轉換
若有特殊屬性轉換需求 ，需使用For Member 
範例：
```csharp
CreateMap<XXXEntity, XXXModel>()
    .ForMember(dest => dest.Id, opt => opt.MapFrom(src => src.MemberId))
```

## 4. Config
名字不同時 config 加入 屬性支持

`AddMemberConfiguration().AddName<SourceToDestinationNameMapperAttributesMember>();`
範例 ：
```csharp
public class Foo
{
    [MapTo("SourceOfBar")]
    public int Bar { get; set; }
}
```

 ##5. Initialize Profile
在Program 加入 Init AutoMapper 的 Profile
```csharp
Mapper.Initialize(x => x.AddProfile<XXXProfile>());
```