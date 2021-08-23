---
description: >-
  Injecting with multiple implementations can let you choose the most suitable
  way to do your job.
---

# Injecting with multiple implementations

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

Example with command node:

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
    private Map<String, MyService> myService; // use Map to inject

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

> When you use `/service hello true` , the service will execute via ServiceA, when you use`/service hello false` or `/service hello` , the service will execute via ServiceB.

## Example with scenario <a id="scenario"></a>

let's say you have multiple ways to let user choose how to save your data.

{% code title="config.yaml" %}
```yaml
useMySQL: true
host: "127.0.0.1:3306"
```
{% endcode %}

 Mapping object for config:

```java
@Resource(locate = "config.yml")
public class TestConfig extends Configuration {
    public boolean useMySQL;
    public String host;
}
```

define StorageService

```java
public interface StorageService {
    
    void initialize();

    void save();

}
```

implement YAML storage service

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

implement MySQL storage service

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

register as below

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

Usage from any singleton

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

## Multiple implementations without key <a id="multiple-inject-without-key"></a>

LogService as example:

```java
public interface LogService {

    void log(Logger logger);

}
```

Create multiple implementation.

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

register in main class

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

Usage:

```java
public class TesterLifeCycle implements ELDLifeCycle {

    @Inject
    private Set<LogService> logServices;

    @Override
    public void onEnable(JavaPlugin javaPlugin) {
        // output via all services
        logServices.forEach(l -> l.log(javaPlugin.getLogger()));
    }

    @Override
    public void onDisable(JavaPlugin javaPlugin) {

    }
}

```



