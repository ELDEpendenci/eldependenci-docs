---
description: 本頁將比較原版的編寫方式與掛接ELD後的編寫方式。
---

# 與原版開發方式的比較

## 分支指令與參數解析

假設你需要編寫如下的分支指令

* /test say &lt;message&gt; - 發送訊息
* /test calculate add &lt;one&gt; \[two\] - 計算加法，第二個數值如果不輸入則為 0
* /test calculate minus &lt;one&gt; \[two\] - 計算減法，第二個數值如果不輸入則為 0

如果你使用原版的編寫方式，那麼其代碼將類似如下:

```java
public class CalculateCommand implements CommandExecutor {

    private boolean showHelp(CommandSender sender){
        sender.sendMessage("/test calculate add <one> [two]");
        sender.sendMessage("/test calculate minus <one> [two]");
        sender.sendMessage("/test say <message>");
        return true;
    }

    @Override
    public boolean onCommand(CommandSender commandSender, Command command, String s, String[] strings) {
        if (strings.length < 2) return showHelp(commandSender);
        switch (strings[0].toLowerCase(Locale.ROOT)){
            case "say":
                String message = String.join(" ", Arrays.copyOfRange(strings, 1, strings.length));
                Bukkit.broadcastMessage(commandSender.getName()+" say: "+message);
                break;
            case "calculate":
                if (strings.length < 3) return showHelp(commandSender);
                try{
                    int one = Integer.parseInt(strings[2]);
                    int two = Integer.parseInt(strings.length > 3 ? strings[3] : "0"); // 默認是 0
                    String msg;
                    switch (strings[1].toLowerCase(Locale.ROOT)){
                        case "add":
                            msg = one + " + " + two + " = " + (one + two);
                            break;
                        case "minus":
                            msg = one + " - " + two + " = " + (one - two);
                            break;
                        default:
                            return showHelp(commandSender);
                    }
                    commandSender.sendMessage(msg);
                }catch (NumberFormatException e){
                    commandSender.sendMessage("not a number!");
                }
                break;
            default:
                showHelp(commandSender);
                break;
        }

        return true;
    }
}
```

{% hint style="info" %}
試想想之後如果你還有更多分支指令要加入，且每個分支指令的參數需求也不一樣，那麼在這個class內，你可能要繼續新增更多的 switch cases； 再假設一個分支指令內又有幾個分支指令, 那麼一個 switch case 內可能又要新增一個 switch ，如此往復。
{% endhint %}

以下是使用本框架所編寫的代碼。本框架採用了一個指令一個class的模式，從設計上是這樣的:

{% tabs %}
{% tab title="TestCommand.java" %}
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
{% endtab %}

{% tab title="TestSayCommand.java" %}
```java
@Commander(
        name = "say",
        description = "test say command",
        alias = {"sa", "s"}
)
public class TestSayCommand implements CommandNode {

    @CommandArg(order = 0, identifier = "message") // 此參數解析將會把所有輸入參數轉變成一則訊息
    private String message;

    @Override
    public void execute(CommandSender commandSender) {
        Bukkit.broadcastMessage(commandSender.getName()+" says: "+message);
    }
}
```
{% endtab %}

{% tab title="TestCalculateCommand.java" %}
```java
@Commander(
        name = "calculate",
        description = "test calculate command",
        alias = {"cal", "c"}
)
public class TestCalculateCommand implements CommandNode {

    @Override
    public void execute(CommandSender commandSender) {
    }
}

```
{% endtab %}

{% tab title="TestCalculateAddCommand.java" %}
```java
@Commander(
        name = "add",
        description = "calculate add command",
        alias = {"ad", "plus", "a"}
)
public class TestCalculateAddCommand implements CommandNode {

    @CommandArg(order = 0)
    private int one;

    @CommandArg(order = 1, optional = true)
    private int two = 0;

    @Override
    public void execute(CommandSender commandSender) {
        commandSender.sendMessage(one+" + "+two+" = "+(one + two));
    }
}
```
{% endtab %}

{% tab title="TestCalculateMinusCommand.java" %}
```java
@Commander(
        name = "minus",
        description = "minus command",
        alias = {"reduce", "m"}
)
public class TestCalculateMinusCommand implements CommandNode {

    @CommandArg(order = 0)
    private int one;

    @CommandArg(order = 1, optional = true)
    private int two = 0;

    @Override
    public void execute(CommandSender commandSender) {
        commandSender.sendMessage(one+" - "+two+" = "+(one - two));
    }
}
```
{% endtab %}
{% endtabs %}

在上述的代碼當中，你應該也發現了使用本框架編寫指令時的第二個特點，也就是**指令參數解析**。  
指令參數解析可讓你在執行指令時省略將輸入的指令參數轉變成其他實例的功夫，助你更方便的編寫指令。你也可以在本框架中註冊自己的指令參數解析，供給自己甚至他人使用。

在細分指令之後，還要把他們連接起來，形成樹狀關係。實現方式也很簡單:

```java
public class TesterRegistry implements ComponentsRegistry {

    @Override
    public void registerCommand(CommandRegistry commandRegistry) { // 註冊指令
        commandRegistry.command(TestCommand.class, c -> {

            c.command(TestSayCommand.class);

            c.command(TestCalculateCommand.class, cc -> {

                cc.command(TestCalculateAddCommand.class);

                cc.command(TestCalculateMinusCommand.class);

            });
            
        });
    }

    @Override
    public void registerListeners(ListenerRegistry listenerRegistry) {
          // 註冊監聽器
    }


}
```

就這樣，創建大型的分支指令就完成了。

## YAML 文件處理

假設你有如下文件

```yaml
name: "hello world"
number: 12
bool: true
box:
  name: "box abc"
  size: 20
  color: RED
```

在原版的開發環境下，你需要進行這些步驟以複製文件到插件資料夾，並使用 `FileConfiguration` 來的方法來獲取數值。

```java
public class ELDTester extends JavaPlugin {

    private FileConfiguration config;

    @Override
    public void onEnable() {
        // 提取並複製到資料夾
        File configFile = new File(getDataFolder(), "config.yml");
        if (!configFile.exists()) saveResource("config.yml", true);
        config = YamlConfiguration.loadConfiguration(configFile);

        // 使用
        getLogger().info(config.getString("name"));
        getLogger().info("number: "+(config.getInt("number") * 2));
        if (config.getBoolean("bool")) getLogger().info("it is true");
        ConfigurationSection section = config.getConfigurationSection("box");
        getLogger().info("box:");
        getLogger().info(section.getString("name"));
        getLogger().info("size: "+section.getInt("size"));
        getServer().getConsoleSender().sendMessage(ChatColor.valueOf(section.getString("color"))+"the color is "+section.getString("color"));
    }


}
```

看上去並不像太困難和太麻煩，但我們擁有更便利的方式來助你處理文件。  
本框架採用了[物件映射關聯](https://zh.wikipedia.org/wiki/%E5%AF%B9%E8%B1%A1%E5%85%B3%E7%B3%BB%E6%98%A0%E5%B0%84)的方式處理文件，將使你在YAML的使用上變得更簡單:

```java
@Resource(locate = "config.yml")
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

從上述的 class 中，你不難看出這個 class 每一個屬性都代表了 config.yml 中的路徑，且已經定義了該屬性的類型。因此在經過註冊後，你可以直接注入並使用這個實例來直接存取 config.yml 中的所有內容。  
例如:

```java
@Commander(
        name = "check",
        description = "config check command"
)
public class TestConfigCheckCommand implements CommandNode {

    @Inject // 此為依賴注入，也是本框架的一個重點之一
    private TestConfig config;

    @Override
    public void execute(CommandSender commandSender) {
        commandSender.sendMessage(config.toString()); // 打印出config的所有內容
    }
}
```

至於註冊，也是極其簡單:

```java
@ELDPlugin(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.addConfiguration(TestConfig.class); // 這就是註冊了
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {

    }
}
```

## 依賴注入

我相信各位寫過大型插件的人，都經常會用到這類的獲取方法

```java
public class ELDTester extends JavaPlugin {

    private static ConfigManager configManager;
    private static DatabaseManager databaseManager;
    private static PlayerManager playerManager;


    public static ConfigManager getConfigManager() {
        return configManager;
    }

    public static PlayerManager getPlayerManager() {
        return playerManager;
    }

    public static DatabaseManager getDatabaseManager() {
        return databaseManager;
    }

    @Override
    public void onEnable() {
       configManager = new ConfigManager();
       databaseManager = new DatabaseManager(configManager);
       playerManager = new PlayerManager(databaseManager);
    }

}
```

不管你使用了何種方式讓你從 Main Class 以外去獲取這三個 Manager, 有時候你都會發現 Main class 成為了獲取各種 Manager class 的一個集中管理器。在 Main class 中，你會初始化文件，指令，監聽器，也會初始各種各樣的自定義的 Manager class, 而每一個 Manager class 可能都會依賴某些在 Main class 中初始化的實例。

透過依賴注入的方式，你除了可以不經過 Main class 就能注入你的 class 所需要的依賴之外，也能在修改依賴時避免了對其他 class 的修改，提高可維護性。

以下為一個簡單的例子: 在指令中注入使用依賴注入

```java
public interface I18nService {

    void sendMessage(Player player, String path);

    void switchLanguage(Player player, String lang);

}
```

```java
public class I18nServiceImpl implements I18nService{

    private final Map<UUID, String> languageMap = new HashMap<>(); // 語言儲存資料庫，你應實作離線儲存

    @Inject // 所有被注入的實例中也可以注入其他實例
    private Map<String, TesterMultiLang> multiLangMap;

    // 根據玩家的語言發送訊息
    @Override
    public void sendMessage(Player player, String path) {
        String lang = languageMap.getOrDefault(player.getUniqueId(), "en-us");
        TesterMultiLang langConfig = multiLangMap.get(lang);
        if (langConfig == null){
            player.sendMessage("由於沒有你的語言 (".concat(lang).concat("), 因此使用回默認語言 en-us"));
            langConfig = multiLangMap.get("en-us");
        }
        player.sendMessage(langConfig.getLang().get(path)); // 發送玩家所使用的語言的訊息
    }

    // 切換語言
    @Override
    public void switchLanguage(Player player, String lang) {
        languageMap.put(player.getUniqueId(), lang);
    }
}

```

{% hint style="info" %}
上面所提供的範例為本框架內的多語言文件功能，詳情可以參閱[這裏](advanced-tutorial/config-operation/i18n.md#shi-yong-wei-i18n)。
{% endhint %}

在指令中注入:

```java
@Commander(
        name = "lang",
        description = "test language command"
)
public class TestLanguageCommand implements CommandNode {


    @Inject
    private I18nService i18nService;

    @CommandArg(order = 0)
    private Player player;

    @CommandArg(order = 1)
    private String path;

    @Override
    public void execute(CommandSender commandSender) {
        i18nService.sendMessage(player, path);
    }
}

```

註冊:

```java
@ELDPlugin(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.addMultipleLanguages(TesterMultiLang.class);
        serviceCollection.bindService(I18nService.class, I18nServiceImpl.class);
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {

    }
}
```

