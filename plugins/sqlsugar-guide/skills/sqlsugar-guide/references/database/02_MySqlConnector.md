# MySqlConnector

## 说明

MySqlConnector 是 MySQL 的优化版连接器，性能更好。

## NuGet
- .NET Core：直接使用 `DbType.MySql` 即可，高版本自带
- .NET Framework：需要安装 MySqlConnector 相关 DLL

- **DbType**: `DbType.MySqlConnector`

## 注意事项
找不到 DLL 时配置自定义程序集：
```csharp
InstanceFactory.CustomAssemblies = new System.Reflection.Assembly[] {
    typeof(SqlSugar.MySqlConnector.MySqlProvider).Assembly
};  //扔在程序启动时
```
