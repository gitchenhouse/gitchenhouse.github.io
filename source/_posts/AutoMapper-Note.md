---
title: AutoMapper Note
date: 2019-05-26 02:00:28
tags: 
- C#
categories:
- C#
---
#  AutoMapper 類別之間轉換
C# 在物件類別之間轉換是很常發生的，例如Model 轉 DB 的 Entity , View 轉 Model 等等 ，這些常在物件轉換很常發生，AutoMapper可以輕易解決，轉換型別的問題 。

<!--more-->

##  簡易Ｍapper物件
首先，您需要來源和目標類型。目標類別的設計可能受其所在層的影響，但只要成員的名稱與來源類型的成員匹配，AutoMapper就能發揮最佳效果。如果您有一個名為“FirstName”的來源屬性，則會自動將其Mapping到名為“FirstName”的目標成員。
```csharp
public class TestCass
{
    public string FirstName { get; set; }
    public int Id { get; set; }
    public decimal Age {get ;set; }

}

public class Test2Cass
{
    public string FirstName { get; set; }
    public int Number { get; set; }
    public int Age {get ;set; }


}
```

通常的做法為 ：
```csharp
TestCass.FirstName = Test2Cass.FirstName;
```

使用AutoMapper轉換，以下範例將TessClass設定為來源屬性，Ｔest2Class設定為目標屬性
```csharp
public static void Main(string[] args)
{
    //初始化AutoMapper                     Ｓource  Destination
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

左側的類型是來源類型，右側的類型是目標類型。要執行Ｍapping，請使用靜態或實例Mapper方法，具體取決於靜態或實例初始化

## 設定Profile
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
            CreateMap<TestCass, Test2Cass>()
            .ForMember(Destination => Destination.Number, option => ption.MapFrom(source => source.Id))
            CreateMap<Test2Cass, TestCass>()
            .ForMember(Destination => Destination.Id, option => option.MapFrom(source => source.Number))
    }
}
```
接著在Program 加入 Init AutoMapper 的 Profile
```csharp
Mapper.Initialize(x => x.AddProfile<XXXProfile>());
```
或者在Method中，重Config 設定
```csharp
  public void test(TessClass source)
        {
            var configuration = new MapperConfiguration(cfg =>
            {
                cfg.AddProfile<XXXProfile>();
            });
            var mapper = configuration.CreateMapper();
            var mappedClass2 = mapper.Map<TestCass, Test2Cass>(source);
        }
```

## 指定 Property 的轉換
若有特殊屬性轉換需求 ，需使用For Member
範例：
```csharp
CreateMap<TestCass, Test2Cass>()
    .ForMember(dest => dest.Age, opt => opt.MapFrom(src =>             Convert.ToDecimal(src.Age)))
```

## Config
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

若有特殊需求，想隔離其他需要Ｍapping物件，可以這樣做：
```csharp
    var source = new TestClass();
    var config = new MapperConfiguration(cfg =>
    {
         cfg.CreateMap<TestClass, TestClass2>()
         .ForMember(dest => dest.FirstName, s => s.MapFrom(src => src.FirstName))
         .ForMember(dest => dest.Id, s => s.MapFrom(src => src.Number));
    });
    var mapper = config.CreateMapper();
    var mappedTestClass2 = mapper.Map<TestClass, TestClass2>(source);
```