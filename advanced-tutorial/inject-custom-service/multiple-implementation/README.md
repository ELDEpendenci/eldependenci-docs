---
description: 注入多重實作方式可讓你提取最合適的方法來執行你的工作。
---

# 注入多重實作方式

```java
public interface MyService {

    void sayHelloTo(CommandSender sender);

    void sayGoodBye(CommandSender sender);

}
```

```java
public class MyServiceA implements MyService {


    @Override
    public void sayHelloTo(CommandSender sender) {
        sender.sendMessage("hello world A!!!");
    }

    @Override
    public void sayGoodBye(CommandSender sender) {
        sender.sendMessage("good bye A!!!");
    }
}
```

```java
public class MyServiceB implements MyService{

    @Override
    public void sayHelloTo(CommandSender sender) {
        sender.sendMessage("hello world B!!!");
    }

    @Override
    public void sayGoodBye(CommandSender sender) {
        sender.sendMessage("good bye B!!!");
    }

}
```

註冊

```java
@ELDPlugin(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.addServices(MyService.class, Map.of(
                "A", MyServiceA.class,
                "B", MyServiceB.class
        ));
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {

    }
}
```

使用，以指令為例

```java
@Commander(
        name = "service",
        description = "service command"
)
public class TestServiceCommand implements CommandNode {

    @Override
    public void execute(CommandSender commandSender) {
    }


}
```

```java
@Commander(
        name = "bye",
        description = "bye command"
)
public class TestServiceByeCommand implements CommandNode {


    @Inject
    private Map<String, MyService> myService; // 使用map inject

    @CommandArg(order = 0, optional = true)
    private boolean b = false;

    @Override
    public void execute(CommandSender commandSender) {
        myService.get(b ? "B" : "A").sayGoodBye(commandSender);
    }
}
```

```java
@Commander(
        name = "hello",
        description = "hello command"
)
public class TestServiceHelloCommand implements CommandNode {

    @Inject
    private Map<String, MyService> myService;

    @CommandArg(order = 0, optional = true)
    private boolean b = false;

    @Override
    public void execute(CommandSender commandSender) {
        myService.get(b ? "B" : "A").sayHelloTo(commandSender);
    }
}
```

> 當你使用 `/service hello true` 的時候，將使用 ServiceB 來執行，當你輸入 `/service hello false` 或者 `/service hello` 的時候，將使用 ServiceA 來執行

## 使用場景範例 <a href="#scenario" id="scenario"></a>

以下將列出使用多重實作方式的使用範例。

{% code title="config.yaml" %}
```yaml
useMySQL: true
host: "127.0.0.1:3306"
```
{% endcode %}

映射物件如下

```java
@Resource(locate = "config.yml")
public class TestConfig extends Configuration {
    public boolean useMySQL;
    public String host;
}
```

定義儲存服務

```java
public interface StorageService {
    
    void initialize();

    void save();

}
```

實作 yaml 儲存服務

```java
public class YamlStorageService implements StorageService{
    @Override
    public void initialize() {
        System.out.println("initialize with yaml");
    }

    @Override
    public void save() {
        System.out.println("using yaml to save");
    }
}
```

實作 mysql 儲存服務

```java
public class MySQLStorageService implements StorageService{
    
    @Inject
    private TestConfig config;
    
    @Override
    public void initialize() {
        System.out.println("initialize with mysql with host: "+config.host);
    }

    @Override
    public void save() {
        System.out.println("using mysql to save");
    }
}
```

註冊如下

```java
@ELDPlugin(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.addServices(StorageService.class, Map.of(
            "mysql", MySQLStorageService.class, 
            "yaml", YamlStorageService.class
        ));
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {
       
    }
}
```

在某個單例的使用方式

```java
public class StorageManager {
    
    private final StorageService storageService;
    
    @Inject
    public StorageManager(TestConfig config, Map<String, StorageService> serviceMap){
        this.storageService = serviceMap.get(config.useMySQL ? "mysql" : "yaml");
        this.storageService.initialize();
    }
    
    
    public void save(){
        this.storageService.save();
    }
}
```

## 不使用key的狀況下注入多重實作 <a href="#multiple-inject-without-key" id="multiple-inject-without-key"></a>

以 LogService 為例

```java
public interface LogService {

    void log(Logger logger);

}
```

創建多重實作方式。

```java
public class LoggerA  implements LogService{
    @Override
    public void log(Logger logger) {
        logger.info("Logging information by LoggerA");
    }
}
```

```java
public class LoggerB implements LogService {
    @Override
    public void log(Logger logger) {
        logger.info("Logging information by LoggerB");
    }
}
```

```java
public class LoggerC implements LogService{
    @Override
    public void log(Logger logger) {
        logger.info("Logging information by LoggerC");
    }
}
```

在主類註冊

```java
@ELDPlugin(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.addService(LogService.class, LoggerA.class);
        serviceCollection.addService(LogService.class, LoggerB.class);
        serviceCollection.addService(LogService.class, LoggerC.class);
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {
    
    }
}
```

使用範例如下

```java
public class TesterLifeCycle implements ELDLifeCycle {

    @Inject
    private Set<LogService> logServices;

    @Override
    public void onEnable(JavaPlugin javaPlugin) {
        // 輸出全部記錄
        logServices.forEach(l -> l.log(javaPlugin.getLogger()));
    }

    @Override
    public void onDisable(JavaPlugin javaPlugin) {

    }
}

```

