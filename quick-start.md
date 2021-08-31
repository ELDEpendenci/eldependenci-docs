---
description: 本頁將教你如何創建你的第一個使用 ELD framework 框架開發的插件。
---

# 快速開始

{% hint style="info" %}
在閱讀本教程之前，可以先參閱以下字眼的定義以更易理解教程。

* 文件 - 泛指 YAML
* 參數 - 泛指指令參數
* 服務 - 泛指 界面接口，可用作 插件對外接口 \(API\)
* 單例 - 保持單一的實例。如果用於實現接口，通常不會用於注入。
{% endhint %}

若果你使用 Maven, 你可以依照下列的 文本 掛接 ELDependenci 框架。

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

首先你需要創建兩個 classes， 第一個是 Registry，用於註冊監聽器與指令。

```java
public class TesterRegistry implements ComponentsRegistry {


    @Override
    public void registerCommand(CommandRegistry commandRegistry) { //註冊指令
    }

    @Override
    public void registerListeners(ListenerRegistry listenerRegistry) { //註冊監聽器
    }

}
```

第二個是 LifeCycle, 用於插件的生命週期執行。

```java
public class TesterLifeCycle implements ELDLifeCycle {

    @Override
    public void onEnable(JavaPlugin javaPlugin) {
        javaPlugin.getLogger().info("hello world!"); // 輸出 hello world!
    }

    @Override
    public void onDisable(JavaPlugin javaPlugin) {
        javaPlugin.getLogger().info("plugin disabled!"); //輸出 plugin disabled!
    }
}
```

{% hint style="danger" %}
LifeCycle 類 和 Registry 類 必須為無參數構造器。
{% endhint %}

開始創建你的 Main Class。

```java
@ELDPlugin(
        registry = TesterRegistry.class, //指向註冊class
        lifeCycle = TesterLifeCycle.class //指向生命週期class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        // 綁定服務, 實例及文件的地方
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {
        // 生命週期前的操作
    }
}
```

{% hint style="success" %}
最後，設置好 plugin.yml, 把 main 指向至 繼承 ELDBukkitPlugin 的 class \(ELDTester.java\) , 你的第一個ELD插件就完成了。
{% endhint %}

## 創建指令 <a id="create-command"></a>

以下範例將編寫分支指令。

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

    //指令參數, 在指令幫助將顯示參數名稱為string
    //order 則是 指令參數排序
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

最後，在 Registry class 定義關係。

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

## 事件監聽器 \(ELD 版本\) <a id="event-listener"></a>

在註冊事件監聽器中，你可以直接註冊原版的事件監聽器，也可以註冊 ELD 版本監聽器 ELDListener。

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

最後，在 Registry class 進行註冊。

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
        //註冊原版監聽器
        listenerRegistry.listeners(List.of(
                TestListeners.class // TestListeners implements org.bukkit.Listener
        ));
    }
}
```

## Yaml 文件配置 <a id="yaml"></a>

此框架採用 ORM 設計進行 Yaml 文件配置。假設你有如下文件配置:

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

文件映射物件需設計如下。

```java
@Resource(locate = "config.yml") // 文件位置
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
此框架文件配置採用 jackson-databind-yaml 作為基礎，因此你可以使用任何的[ jackson-annotations](https://www.baeldung.com/jackson-annotations) 以控制映射物件。
{% endhint %}

假若你的文件配置如下，則可使用 Map 裝載。

{% tabs %}
{% tab title="config.yml" %}
```yaml
name: "hello world"
number: 12
bool: true
# 使用 Map 裝載
boxes:
  box1:
    name: "box 1"
    size: 1
    color: RED
  box2:
    name: "box 2"
    size: 2
    color: BLUE
  box3:
    name: "box 3"
    size: 3
    color: ORANGE
# 使用 List 裝載
boxList:
  - name: "box 1"
    size: 1
    color: ORANGE
  - name: "box 2"
    size: 2
    color: ORANGE

```
{% endtab %}

{% tab title="TestConfig.java" %}
```java
@Resource(locate = "config.yml")
public class TestConfig extends Configuration {

    public String name;
    public int number;
    public boolean bool;
    public Map<String, Box> boxes;
    public List<Box> boxList;

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
                ", boxes=" + boxes +
                ", boxList=" + boxList +
                '}';
    }
}
```
{% endtab %}
{% endtabs %}

{% hint style="success" %}
你也可以裝載任何繼承了 `ConfigurationSerializable` 的 Bukkit Object 類型，例如 `ItemStack` 或 `Location` 等等。
{% endhint %}

最後在你的 Main class 中，註冊 映射物件。

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
註冊後，你可把文件映射物件當作可注入實例使用。
{% endhint %}

## 注入實例 <a id="inject-instance"></a>

### 注入文件映射物件 <a id="yaml-inject-instance"></a>

以分支指令為例。

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

    @Inject //注入物件
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

    @Inject // 注入物件
    private TestConfig config;

    @Override
    public void execute(CommandSender commandSender) {
        config.getController().reload();
        commandSender.sendMessage("reload completed");
    }
}
```

以事件監聽器為例

```java
public class TestListeners implements Listener {

    @Inject // 注入物件
    private TestConfig config;

    @EventHandler
    public void onPlayerChat(AsyncPlayerChatEvent e) {
        if (e.getMessage().equals("config")) {
            e.getPlayer().sendMessage(config.toString());
        }
    }


}
```

### 注入自定義單例 \(Singleton\) <a id="inject-singleton"></a>

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

註冊單例

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

使用: 以生命週期為例

```java
public class TesterLifeCycle implements ELDLifeCycle {

    @Inject
    private TesterSingleton singleton;

    @Inject
    private TestConfig config;

    @Override
    public void onEnable(JavaPlugin javaPlugin) {
        singleton.setKey("abc", "fuck");
        singleton.setKey("xyz", "diu");
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

@Inject 除了使用在 instance field 之外，你也可以使用於 constructor \(建構子/構造器\) 之中。

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
你無法使用 @Inject 於 不可注入的實例之中，否則會報錯。
{% endhint %}

## 注入服務 \(Service\) <a id="inject-service"></a>

服務與單例性質基本相同，但其分別在於服務是使用 interface 作為媒介，而 單例 則使用 實例 。  
使用 interface 作為媒介的好處在於避免出現[高耦合](https://ithelp.ithome.com.tw/articles/10191761)的問題，通常適用於作為插件對外接口\(API\), 或是出現不同的實作方式時。

```java
@Commander(
        name = "one",
        description = "one scheduler"
)
public class TestSchedulerOneCommand implements CommandNode {


    @Inject
    private ScheduleService service; // 此為框架內部可使用服務

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
你可以在任何可進行依賴注入的實例中注入可注入實例。

可注入實例包括如下

* 文件映射物件
* 服務 \(Service\)
* 單例 \(Singleton\)
* 插件主類 \(Plugin\)

可進行依賴注入的實例包括如下

* 指令 \(繼承 CommandNode 的 class\)
* 監聽器 \(繼承 Listener 或 ELDListener 的 class\)
* 已註冊實例 \(使用 ServiceCollection 註冊的單例 \[Singleton\]\)
* 已註冊服務 \(使用 ServiceCollection 註冊的服務 \[Service\]\)
* 生命週期 \(繼承 ELDLifeCycle 的 class\)
{% endhint %}



