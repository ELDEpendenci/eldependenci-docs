---
description: >-
  Injecting Singleton has been taught from Quick Start, so here will focus on
  services only.
---

# Injecting Custom Services

{% hint style="warning" %}
Please make sure you have read the [Quick Start](../../quick-start.md) before the continuous reading.
{% endhint %}

Create an interface

```java
public interface IService {

    void doSomething(CommandSender sender);
    
}
```

and make a class to implement it.

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

Finally, register on main class.

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

Then, you can use it in any injected instances.

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







