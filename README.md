# Warden Process Watcher

![Warden](http://spetz.github.io/img/warden_logo.png)

**OPEN SOURCE & CROSS-PLATFORM TOOL FOR SIMPLIFIED MONITORING**

**[getwarden.net](http://getwarden.net)**

|Branch             |Build status                                                  
|-------------------|-----------------------------------------------------
|master             |[![master branch build status](https://api.travis-ci.org/warden-stack/Warden.Watchers.Process.svg?branch=master)](https://travis-ci.org/warden-stack/Warden.Watchers.Process)
|develop            |[![develop branch build status](https://api.travis-ci.org/warden-stack/Warden.Watchers.Process.svg?branch=develop)](https://travis-ci.org/warden-stack/Warden.Watchers.Process/branches)

**ProcessWatcher** can be used for process monitoring also including the remote machines.

### Installation:

Available as a **[NuGet package](https://www.nuget.org/packages/Warden.Watchers.Process)**. 
```
dotnet add package Warden.Watchers.Process
```

### Configuration:

 - **DoesNotHaveToBeResponding()** - return a valid result even if the existing process is not responding.
 - **EnsureThat()** - predicate containing a received *ProcessInfo* that has to be met in order to return a valid result.
 - **EnsureThatAsync()** - async predicate containing a received *ProcessInfo* that has to be met in order to return a valid result.
 - **WithProcessServiceProvider()** - provide a  custom *IProcessService* which is responsible for checking the process status. 

**ProcessWatcher** can be configured by using the **ProcessWatcherConfiguration** class or via the lambda expression passed to a specialized constructor.

Example of configuring the watcher via provided configuration class:
```csharp
var configuration = ProcessWatcherConfiguration
    .Create("mongod", machineName: "MyRemoteMachine")
    .EnsureThat(process => process.Id == 1000)
    .Build();
var processWatcher = ProcessWatcher.Create("Process watcher", configuration);

var wardenConfiguration = WardenConfiguration
    .Create()
    .AddWatcher(processWatcher)
    //Configure other watchers, hooks etc.
```

Example of adding the watcher directly to the **Warden** via one of the extension methods:
```csharp
var wardenConfiguration = WardenConfiguration
    .Create()
    .AddProcessWatcher("mongod", cfg =>
    {
        cfg.EnsureThat(process => process.Id == 1000)
    })
    //Configure other watchers, hooks etc.
```

Please note that you may either use the lambda expression for configuring the watcher or pass the configuration instance directly. You may also configure the **hooks** by using another lambda expression available in the extension methods.

### Check result type:
**ProcessWatcher** provides a custom **ProcessWatcherCheckResult** type which contains additional values.

```csharp
public class ProcessWatcherCheckResult : WatcherCheckResult
{
    public ProcessInfo ProcessInfo { get; }
}
```
### Custom interfaces:
```csharp
public interface IProcessService
{
    Task<ProcessInfo> GetProcessInfoAsync(string name);
}

public class ProcessInfo
{
    public int Id { get; }
    public string Name { get; }
    public string Machine { get; set; }
    public bool Exists { get; }
    public bool Responding{ get; }
}
```

**IProcessService** is responsible for checking process status. It can be configured via the *WithProcessServiceProvider()* method. By default it is based on the **[System.Diagnostics.Process](https://msdn.microsoft.com/en-us//library/system.diagnostics.process(v=vs.110).aspx)**.