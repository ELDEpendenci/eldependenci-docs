---
description: This page are going to teach you how to create your first plugin via ELD.
---

# Quick Start

{% hint style="info" %}
Before the continue reading, make sure you have understanding with these keywords:

* Configuration - Mostly mean to YAML
* Arguments - Mostly mean to command arguments
* Service - Mostly mean to interface, and also API
* Singleton - An Instance which keep only single one. If it's use in implementing interface, commonly will not be inject.
{% endhint %}

If you are using maven, you can follow the xml instruction as below.

```markup
<repositories>
    <repository>
        <id>nexus</id>
        <url>https://nexus.chu77.xyz/repository/plugins/</url>
    </repository>
</repositories>
```

```markup
<dependencies>
    <dependency>
        <groupId>org.eldependenci</groupId>
        <artifactId>eldependenci-framework</artifactId>
        <version>{最新版本}</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

At first you need to create two classes, the first is a registry class which for registering commands and listeners.

```java
public class TesterRegistry implements ComponentsRegistry {


    @Override
    public void registerCommand(CommandRegistry commandRegistry) { // register command
    }

    @Override
    public void registerListeners(ListenerRegistry listenerRegistry) { //register listeners
    }

}
```

The second if a lifecycle class which to replcae the feature of JavaPlugin \(to injectable lifecycle instance\)

```java
public class TesterLifeCycle implements ELDLifeCycle {

    @Override
    public void onEnable(JavaPlugin javaPlugin) {
        javaPlugin.getLogger().info("hello world!"); // output hello world!
    }

    @Override
    public void onDisable(JavaPlugin javaPlugin) {
        javaPlugin.getLogger().info("plugin disabled!"); //output plugin disabled!
    }
}
```

{% hint style="danger" %}
LifeCycle class and  Registry class must be no arg constructor.
{% endhint %}

Start to create your main class.

```java
@ELDPlugin(
        registry = TesterRegistry.class, //point to registry class
        lifeCycle = TesterLifeCycle.class //point to lifecycle class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        // the place to bind service, bind singleton and add configuration
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {
        // operation before lifecycle
    }
}
```

{% hint style="success" %}
Finally, create plugin.yml and point your main into your class which extended ELDBukkitPlugin, then your first plugin will be done.
{% endhint %}

## Command Creation <a id="create-command"></a>

Below examples showing how to create sub commands

```java
@Commander(
        name = "test",
        description = "test command",
        alias = {"tes", "te"}
)
public class TestCommand implements CommandNode {

    @Override
    public void execute(CommandSender commandSender) {
    }

}
```

```java
@Commander(
        name = "one",
        description = "test one command"
)
public class TestOneCommand implements CommandNode {

    //command arg, the display name will be "string"
    //command order is 0
    @CommandArg(order = 0, labels = {"string"})
    private String value;

    @Override
    public void execute(CommandSender commandSender) {
        commandSender.sendMessage("this is one command with value "+value);
    }
}
```

```java
@Commander(
        name = "two",
        description = "two command"
)
public class TestTwoCommand implements CommandNode {

    @CommandArg(order = 0)
    private int number;

    @Override
    public void execute(CommandSender commandSender) {
        commandSender.sendMessage("this is two command with number "+number);
    }
}
```

Finally, define the parent-child relationship from registry class.

{% code title="TesterRegistry.java" %}
```java
public class TesterRegistry implements ComponentsRegistry {


    @Override
    public void registerCommand(CommandRegistry commandRegistry) {
        commandRegistry.command(TestCommand.class, c -> {
            
            c.command(TestOneCommand.class);
            
            c.command(TestTwoCommand.class);
            
        });
    }

    @Override
    public void registerListeners(ListenerRegistry listenerRegistry) {
    }


}
```
{% endcode %}

## Event Listener \(ELD ver\) <a id="event-listener"></a>

You can instantly register bukkit listener into your registry class, but you can also register an eld listener which shows like below.

```java
public class TestELDListener implements ELDListener {
    
    @Override
    public void defineNodes(EventListeners eventListeners) {
        // 擁有 player.join.slient 的玩家沒有加入訊息
        eventListeners.listen(PlayerJoinEvent.class)
                .expireAfter(3) // 執行超過三次後逾期
                .filter(e -> e.getPlayer().hasPermission("player.join.silent"))
                .handle(e -> e.setJoinMessage(null));

        // 根據玩家是否擁有 player.chat 權限而發送不同訊息
        eventListeners.listen(AsyncPlayerChatEvent.class)
                .priority(EventPriority.MONITOR)
                .filter(e -> !e.isCancelled()) // ignoreCancelled = true
                .filter(e -> e.getPlayer().hasPermission("player.chat"))
                .fork() // 建立分支點而不採用 handle
                .ifTrue(e -> e.getPlayer().sendMessage("you have player.chat permission"))
                .ifFalse(e -> e.getPlayer().sendMessage("you don't have player.chat permission"));
        
        eventListeners.listen(PlayerQuitEvent.class)
                .filter(e -> e.getPlayer().hasPermission("vip.permission"))
                .fork()
                .ifTrue(this::onPlayerQuitIsVIP) // 使用 lambda method
                .ifFalse(this::onPlayerQuitIsNotVIP);
    }
    
    private void onPlayerQuitIsVIP(PlayerQuitEvent event){
        event.setQuitMessage("vip left the server: "+event.getPlayer().getName());
    }
    
    
    private void onPlayerQuitIsNotVIP(PlayerQuitEvent event){
        event.setQuitMessage("player left the server: "+event.getPlayer().getName());
    }
}
```

Finally register in registry class

```java
public class TesterRegistry implements ComponentsRegistry {


    @Override
    public void registerCommand(CommandRegistry commandRegistry) {
    }

    @Override
    public void registerListeners(ListenerRegistry listenerRegistry) {
        listenerRegistry.ELDListeners(List.of(
                TestELDListener.class
        ));
        //the original bukkit listener
        listenerRegistry.listeners(List.of(
                TestListeners.class // TestListeners implements org.bukkit.Listener
        ));
    }
}
```

## Yaml Configuration <a id="yaml"></a>

This framework are using ORM as configuration based. If you have the yaml below

{% code title="config.yml" %}
```yaml
name: "hello world"
number: 12
bool: true
box:
  name: "box abc"
  size: 20
  color: RED
```
{% endcode %}

The mapping object should set like this.

```java
@Resource(locate = "config.yml") // resource location from both inside jar and outside destination
public class TestConfig extends Configuration {
    public String name;
    public int number;
    public boolean bool;
    public Box box;

    public static class Box {
        public String name;
        public int size;
        public ChatColor color;


        @Override
        public String toString() {
            return "Box{" +
                    "name='" + name + '\'' +
                    ", size=" + size +
                    ", color=" + color +
                    '}';
        }
    }


    @Override
    public String toString() {
        return "TestConfig{" +
                "name='" + name + '\'' +
                ", number=" + number +
                ", bool=" + bool +
                ", box=" + box +
                '}';
    }
}

```

{% hint style="info" %}
This framework are using jackson-databind-yaml as based，so you can use any[ jackson-annotations](https://www.baeldung.com/jackson-annotations) to control your mapper object.
{% endhint %}

Finally add your mapping object class in main.

```java
@ELDPlugin(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.addConfiguration(TestConfig.class); // 註冊文件
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {
    }
}
```

{% hint style="success" %}
You can inject the mapping object after registration.
{% endhint %}

## Instance injection <a id="inject-instance"></a>

### Injecting yaml mapping object <a id="yaml-inject-instance"></a>

example from sub commands

```java
@Commander(
        name = "config",
        description = "config command"
)
public class TestConfigCommand implements CommandNode {
    @Override
    public void execute(CommandSender commandSender) {

    }
}
```

```java
@Commander(
        name = "edit",
        description = "config edit command"
)
public class TestConfigEditCommand  implements CommandNode {

    private final Random random = new Random();

    @Inject //inject config
    private TestConfig config;

    @Override
    public void execute(CommandSender commandSender) {
        config.bool = random.nextBoolean();
        config.name = UUID.randomUUID().toString();
        config.number = random.nextInt();
        config.box = new TestConfig.Box();
        config.box.color = ChatColor.values()[random.nextInt(ChatColor.values().length)];
        config.box.name = UUID.randomUUID().toString()+" box";
        config.box.size = random.nextInt();

        try {
            config.getController().save();
            commandSender.sendMessage("save completed");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
@Commander(
        name = "reload",
        description = "config reload command"
)
public class TestConfigReloadCommand implements CommandNode {

    @Inject // inject config
    private TestConfig config;

    @Override
    public void execute(CommandSender commandSender) {
        config.getController().reload();
        commandSender.sendMessage("reload completed");
    }
}
```

example from listeners

```java
public class TestListeners implements Listener {

    @Inject // inject config
    private TestConfig config;

    @EventHandler
    public void onPlayerChat(AsyncPlayerChatEvent e) {
        if (e.getMessage().equals("config")) {
            e.getPlayer().sendMessage(config.toString());
        }
    }


}
```

### Injecting your own singleton <a id="inject-singleton"></a>

```java
public class TesterSingleton {

    private final Map<String, String> collection = new HashMap<>();

    public void setKey(String key, String value){
        this.collection.put(key, value);
    }

    public String getString(){
        return this.collection.toString();
    }
}
```

register singleton

```java
@ELDPlugin(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.addSingleton(TesterSingleton.class);
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {
        
    }
}
```

example from lifecycle

```java
public class TesterLifeCycle implements ELDLifeCycle {

    @Inject
    private TesterSingleton singleton;

    @Inject
    private TestConfig config;

    @Override
    public void onEnable(JavaPlugin javaPlugin) {
        singleton.setKey("abc", "foo");
        singleton.setKey("xyz", "bar");
        javaPlugin.getLogger().info(singleton.getString());
        javaPlugin.getLogger().info(config.toString());
    }

    @Override
    public void onDisable(JavaPlugin javaPlugin) {
        javaPlugin.getLogger().info(singleton.getString());
        javaPlugin.getLogger().info(config.toString());
    }
}
```

@Inject can also use in constructor.

```java
public class TestManager {

    private final TesterSingleton singleton;
    
    @Inject
    public TestManager(TesterSingleton singleton){
        this.singleton = singleton;
        singleton.setKey("start", "started a insertion in constructor");
    }

    public void doSomething(){
        System.out.println(singleton.getString())
    }
   
}
```

{% hint style="danger" %}
You can't use @inject on uninjectable instance or it will throw an error.
{% endhint %}

## Injecting Service <a id="inject-service"></a>

Services and Singleton are different that services are using interface while singleton are using instance. Using interface can avoid the high coupling problem and commonly use for API, or with different implementations.

```java
@Commander(
        name = "one",
        description = "one scheduler"
)
public class TestSchedulerOneCommand implements CommandNode {


    @Inject
    private ScheduleService service; // this is a service from framework

    @Override
    public void execute(CommandSender commandSender) {
        commandSender.sendMessage("wait for 5 secs");
        service.injectTask(new BukkitRunnable() {

            @Inject
            private ScheduleService service;

            @Override
            public void run() {
                commandSender.sendMessage("scheduler service instance inside bukkit runnable is "+(service == null ? "null" : "not null !"));
            }

        }).asynchronous(false)
        .timeout(100L)
        .run(ELDTester.getProvidingPlugin(ELDTester.class));
    }
}
```

## 

{% hint style="success" %}
You can inject instance from any instance which allow injection.

injectable instances as below

* yaml mapping object
* Service
* Singleton

instance which allow injection as below

* Command \(which extended Command Node\)
* Listeners \(which extended Listener or ELDListener\)
* Singleton \(which registered from ServiceCollection\)
* Services \(which registered from ServiceCollection\)
* LifeCycle \(which extended ELDLifeCycle\)
{% endhint %}



