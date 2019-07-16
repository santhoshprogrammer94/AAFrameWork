# AA.FrameWork
AA.Framework is built on the popular open source class library of NET Core

## 开源清单
- [ ] log：Log4net
- [ ] object mapper：automapper
- [ ] orm：dapper（Dapper-FluentMap,Dommel）
- [x] MQ：RabbitMQ.Client
- [ ] Redis：StackExchange.Redis
- [x] message bus：MassTransit
- [x] more.....
## Nuget Packages

Package| nuget
---|---
AA.Dapper | [nuget](https://www.nuget.org/packages/AA.Dapper/)
AA.Log4Net | [nuget](https://www.nuget.org/packages/AA.Log4Net/)
AA.FrameWork  | [nuget](https://www.nuget.org/packages/AA.FrameWork/)
AA.Redis  | [nuget](https://www.nuget.org/packages/AA.Redis/)
AA.AutoMapper  | [nuget](https://www.nuget.org/packages/AA.AutoMapper/)

## AA.Dapper用法
#### 实体映射配置
使用DommelEntityMap<TEntity>类映射属性名称。创建一个派生类，在构造函数中设置Map方法，可指定某个属性映射到数据库列名

```
   public class UserInfoMap : DommelEntityMap<UserInfo>
    {
        public UserInfoMap()
        {
            ToTable("Sys_UserInfo");//映射具体的表名
            Map(p => p.SysNo).IsKey();//指定主键 ,IsIdentity是否自增
            Map(p=>p.GmtCreate).ToColumn("GmtCreateDate"); //属性名和数据库列名 可以不同
            Map(p=>p.LastLoginDate).Ignore();//一些计算属性，可以忽略不需要跟数据库列进行映射
        }
    }
```

使用MapConfiguration.Init方法，把映射类初始化，后续就可以使用了

```
 public static void InitMapCfgs()
        {
            var fluentMapconfig = new List<Action<FluentMapConfiguration>>();
            fluentMapconfig.Add(cfg =>
            {
                cfg.AddMap(new UserInfoMap());
            });
            MapConfiguration.Init(fluentMapconfig);
        }
```

####  开始使用AA.Dapper
##### 使用DapperContext设置数据库连接和数据库类型是sqlserver还是mysql

```
    public class AADapperContext : DapperContext
    {
        public AADapperContext() : base(new NameValueCollection()
        {
            ["aa.dataSource.AaCenter.connectionString"] = "Data Source =.; Initial Catalog = AaCenter;User ID = sa; Password = 123;",
            ["aa.dataSource.AaCenter.provider"] = "SqlServer"
        })
        { }
    }
```
##### 仓储包含了大部分的操作，同时支持Async操作

```
IDapperRepository<UserInfo> userInfoRepository = new DapperRepository<UserInfo>();
```
###### 插入实体

```
 var user = new UserInfo()
            {
                UserName = "chengTian",
                RealName = "成天",
                GmtCreate = DateTime.Now,
                GmtModified = DateTime.Now
            };
  var result = userInfoRepository.Insert(user);
```
###### 修改实体

```
 var user = userInfoRepository.Get(1);
            user.GmtModified = DateTime.Now;
 var result = userInfoRepository.Update(user);
```
###### 获取实体

```
   var user = userInfoRepository.Get(1);//By 主键
   var users = userInfoRepository.GetAll();//所有
   var users = userInfoRepository.Select(p => p.UserName == "chengTian");//谓词
```
###### 删除实体

```
   var user = userInfoRepository.Get(1);
   var result = userInfoRepository.Delete(user);
```
###### 支持Dapper原生操作
操作基本的封装都是单表的操作，可以满足一部分业务开发，有些业务场景编写sql还是比较合适的比如报表和一些复杂的查询，推荐使用原生dapper的操作

```
public class UserInfoRepository : DapperRepository<UserInfo>, IUserInfoRepository
    {
      
        public IEnumerable<UserInfo> QueryAll()
        {
            var result = DapperContext.Current.DataBase.Query<UserInfo>("SELECT * from  [Sys_UserInfo]");//实例
            return result;
        }
    }
```
## AA.Log4Net 用法

1. log4net.config 配置，并将log4net.config的属性->复制到输出目录->设置为->始终复制
1.  Log4NetLogger.Use("配置文件名或者路径+配置文件名"); 例如：log4net.config文件在根目录下，Log4NetLogger.Use("log4net.config");如果在自定义文件中；例如config/log4net.config 则Log4NetLogger.Use("config/log4net.config")
1. ILog log= Logger.Get(typeof(类));
log.Debug("example");



