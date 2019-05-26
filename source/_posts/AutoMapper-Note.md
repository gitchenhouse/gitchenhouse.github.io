---
title: AutoMapper Note
date: 2019-05-26 02:00:28
tags:
---
# AutoMapper 類別之間轉換
### 設定Profile
透過 Profile 可隔離不同的 Class，我們應該要把這些對應關係給集中管理，設定一次所有的開發人員都能夠用，AutoMapper 也有提供集中管理的機制

很簡單，只要實作 AutoMapper.Profile 並覆寫 Configure 方法，把相關的對應關係擺好

```
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

### 指定 Property 的轉換

```
CreateMap<XXXEntity, XXXModel>()
    .ForMember(dest => dest.Id, opt => opt.MapFrom(src => src.MemberId))
```

### Config
名字不同時 config 加入  屬性支持

`AddMemberConfiguration().AddName<SourceToDestinationNameMapperAttributesMember>();`
範例 ：
```
public class Foo
{
    [MapTo("SourceOfBar")]
    public int Bar { get; set; }
}
```

### Initialize
在Program 加入 Init AutoMapper

```
Mapper.Initialize(x => x.AddProfile<XXXProfile>());
```
###  調用
調用 Mapper.Map 方法，就可以產生新的物件

`Mapper.Map<AccountTwViewModel>(account);`
