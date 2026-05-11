# Razor模版生成

## 使用用例

```csharp
SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    ConnectionString = Config.ConnectionString,
    DbType = DbType.SqlServer,
    IsAutoCloseConnection = true,
    ConfigureExternalServices = new ConfigureExternalServices()
    {
        RazorService = new RazorService() // 需要自定义 RazorService 类
    }
});

var templte = RazorFirst.DefaultRazorClassTemplate; // 内置模板，可自行修改
db.DbFirst.UseRazorAnalysis(templte).CreateClassFile("c:\\Demo\\Razor\\");
```

`RazorService` 在 .NET Framework 和 .NET Core 中略有差异，见下方示例。

## .NET Framework

创建 `RazorService` 需要安装 `RazorEngine 3.10.0.0`。

```csharp
using RazorEngine;
using RazorEngine.Templating;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace SqlSugar.DbFirstExtensions
{
    public class RazorService : IRazorService
    {
        public List<KeyValuePair<string, string>> GetClassStringList(string razorTemplate, List<RazorTableInfo> model)
        {
            if (model != null && model.Any())
            {
                var result = new List<KeyValuePair<string, string>>();
                foreach (var item in model)
                {
                    try
                    {
                        item.ClassName = item.DbTableName; // 格式化类名
                        string key = "RazorService.GetClassStringList" + razorTemplate.Length;
                        var classString = Engine.Razor.RunCompile(razorTemplate, key, item.GetType(), item);
                        result.Add(new KeyValuePair<string, string>(item.ClassName, classString));
                    }
                    catch (Exception ex)
                    {
                        new Exception(item.DbTableName + " error ." + ex.Message);
                    }
                }
                return result;
            }
            else
            {
                return new List<KeyValuePair<string, string>>();
            }
        }
    }
}
```

## .NET Core / .NET 5 / .NET 6

创建 `RazorService` 需要安装 `RazorEngine.NetCore 3.1`。

```csharp
using RazorEngine;
using RazorEngine.Templating;
using SqlSugar;
using System;
using System.Collections.Generic;
using System.Linq;

namespace DbFirstRazorTest
{
    class Program
    {
        static void Main(string[] args)
        {
            SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
            {
                ConnectionString = "server=.;uid=sa;pwd=sasa;database=SQLSUGAR4XTEST",
                DbType = DbType.SqlServer,
                IsAutoCloseConnection = true,
                ConfigureExternalServices = new ConfigureExternalServices()
                {
                    RazorService = new RazorService()
                }
            });
            db.DbFirst.UseRazorAnalysis(RazorFirst.DefaultRazorClassTemplate).CreateClassFile("c:\\Demo\\Razor\\");
        }
    }

    public class RazorService : IRazorService
    {
        public List<KeyValuePair<string, string>> GetClassStringList(string razorTemplate, List<RazorTableInfo> model)
        {
            if (model != null && model.Any())
            {
                var result = new List<KeyValuePair<string, string>>();
                foreach (var item in model)
                {
                    try
                    {
                        item.ClassName = item.DbTableName; // 格式化类名
                        string key = "RazorService.GetClassStringList" + razorTemplate.Length;
                        var classString = Engine.Razor.RunCompile(razorTemplate, key, item.GetType(), item);
                        result.Add(new KeyValuePair<string, string>(item.ClassName, classString));
                    }
                    catch (Exception ex)
                    {
                        new Exception(item.DbTableName + " error ." + ex.Message);
                    }
                }
                return result;
            }
            else
            {
                return new List<KeyValuePair<string, string>>();
            }
        }
    }
}
```
