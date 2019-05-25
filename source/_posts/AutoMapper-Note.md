---
title: AutoMapper Note
date: 2019-05-26 02:00:28
tags:
---
# AutoMapper 類別之間轉換
透過 Profile 可隔離不同的 Class

```
public class XXXProfile : Profile
{
    public XXXMappingProfile()
    {
// 設定Config
AddMemberConfiguration().AddName<SourceToDestinationNameMapperAttributesMember>();

			//CreateMap<Souce, Destination>();
            CreateMap<Model, Response>();
    } 
}
```

指定 Property 的轉換

```
CreateMap<XXXEntity, XXXModel>()
    .ForMember(dest => dest.Id, opt => opt.MapFrom(src => src.MemberId))
```


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