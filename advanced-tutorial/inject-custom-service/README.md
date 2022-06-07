---
description: 注入實例已經在快速開始說過，因此本教程只集中與注入服務 (Service)
---

# 注入自定義服務

{% hint style="warning" %}
在開始本教程之前，請先細閱[快速開始](../../quick-start.md)。
{% endhint %}

創建一個 interface

```java
public interface IService {

    void doSomething(CommandSender sender);
    
}
```

然後創建一個 class 實作它

```java
public class IServiceImpl implements IService{
    
    @Inject
    private TestConfig config; // 別忘了任何註冊單例/服務均可使用依賴注入!
    
    @Override
    public void doSomething(CommandSender sender) {
        sender.sendMessage(config.name);
    }
}
```

最後到 main class註冊

```java
@ELDPlugin(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.bindService(IService.class, IServiceImpl.class);
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {
    }
}
```

這樣，你便可以在任何地方使用它。

```java
@Commander(
        name = "service",
        description = "service command"
)
public class TestServiceCommand implements CommandNode {

    @Inject
    private IService service;
    
    @Override
    public void execute(CommandSender commandSender) {
        service.doSomething(commandSender);
    }


}

```

{% hint style="info" %}
**目前所有被註冊的服務/實例全都默認為 單例。**如果你不想全部都設置為單例，你可到 `config.yml` 將 `defaultSingleton` 設置為 false, 然後單獨地在你欲單例的 實例 class 上標註 `@Singleton`
{% endhint %}

