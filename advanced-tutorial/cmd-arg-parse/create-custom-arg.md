---
description: 創建自定義參數解析。
---

# 創建自定義參數

創建參數解析器如下

{% tabs %}
{% tab title="使用 Bukkit 掛接" %}
```java
@ELDBukkit(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    public void bindServices(ServiceCollection serviceCollection) {
    }

    @Override
    protected void manageProvider(BukkitManagerProvider provider) {
        var parser = provider.getArgumentManager(); //參數解析器
        // 創建參數解析
        parser.registerParser(Integer.class, (iterator, commandSender, argParser) -> {
            try{
                return Integer.parseInt(iterator.next());
            }catch (NumberFormatException e){
                throw new ArgumentParseException("不是有效的 integer."); 
            }
        });
    }
}
```
{% endtab %}

{% tab title="使用 Bungee 掛接" %}
```java
@ELDBungee(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBungeePlugin {

    @Override
    public void bindServices(ServiceCollection serviceCollection) {
    }

    @Override
    protected void manageProvider(BungeeManagerProvider provider) {
        var parser = provider.getArgumentManager(); //參數解析器
        // 創建參數解析
        parser.registerParser(Integer.class, (iterator, commandSender, argParser) -> {
            try{
                return Integer.parseInt(iterator.next());
            }catch (NumberFormatException e){
                throw new ArgumentParseException("不是有效的 integer."); 
            }
        });
    }
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
拋出 ArgumentParseException 並不會於後台報錯，只會停止執行指令並以訊息提醒指令輸入者。
{% endhint %}

上方程式碼為創建 Integer (整數) 的參數解析，以用於解析參數為 integer 的時候使用，例如

```java
@Commander(
        name = "add",
        description = "calculate add command",
        alias = {"ad", "plus", "a"}
)
public class TestCalculateAddCommand implements CommandNode {

    @CommandArg(order = 0)
    private int one;

    @CommandArg(order = 1)
    private int two;

    @Override
    public void execute(CommandSender commandSender) {
        commandSender.sendMessage(one+" + "+two+" = "+(one + two));
    }
}
```

為了能以多重方式解析一個類別，註冊指令參數解析時可以標明 identifier (標識文字)

{% tabs %}
{% tab title="使用 Bukkit 掛接" %}
```java
@ELDBukkit(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    public void bindServices(ServiceCollection serviceCollection) {
    }

    @Override
    protected void manageProvider(BukkitManagerProvider provider) {
        var parser = provider.getArgumentManager(); //參數解析器
        // 創建參數解析, identifier (標識文字) 為 message
        parser.registerParser(String.class, "message", (iterator, commandSender, p) -> {
            StringBuilder builder = new StringBuilder();
            iterator.forEachRemaining(s -> builder.append(s).append(" "));
            return builder.toString();
        });
    }
}
```
{% endtab %}

{% tab title="使用 Bungee 掛接" %}
```java
@ELDBungee(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBungeePlugin {

    @Override
    public void bindServices(ServiceCollection serviceCollection) {
    }

    @Override
    protected void manageProvider(BungeeManagerProvider provider) {
        var parser = provider.getArgumentManager(); //參數解析器
        // 創建參數解析, identifier (標識文字) 為 message
        parser.registerParser(String.class, "message", (iterator, commandSender, p) -> {
            StringBuilder builder = new StringBuilder();
            iterator.forEachRemaining(s -> builder.append(s).append(" "));
            return builder.toString();
        });
    }
}
```
{% endtab %}
{% endtabs %}

這樣，此標識的解析將不同於普通 String 類 只解析一個參數，而是把剩餘的參數連成一則訊息串。\
其用法例子:

```java
@Commander(
        name = "say",
        description = "test say command",
        alias = {"sa", "s"}
)
public class TestSayCommand implements CommandNode {

    @CommandArg(order = 0, identifier = "message")
    private String message;

    @Override
    public void execute(CommandSender commandSender) {
        Bukkit.broadcastMessage(commandSender.getName()+" says: "+message);
    }
}
```

這樣，當輸入 `/say this is a text` 的時候, message 的 數值將為 `"this is a text"`

## 創建自定義參數時使用參數解析 <a href="#arg-parser" id="arg-parser"></a>

假設你創建一個自定義參數為 `org.bukkit.Location` , 你會發現其參數 x y z 將會是 double 類別，這樣你就可以在內部使用參數解析。

```java
argumentManager.registerParser(Location.class, (args, sender, parser) -> {
            World world;
            if (!(sender instanceof Player)) {
                world = Bukkit.getWorld(args.next());
            } else {
                world = ((Player) sender).getWorld();
            }
            if (world == null) {
                throw new ArgumentParseException("&c未知世界");
            }
            var x = parser.tryParse(Double.class, args, sender); //內部解析
            var y = parser.tryParse(Double.class, args, sender); //內部解析
            var z = parser.tryParse(Double.class, args, sender); //內部解析
            return new Location(world, x, y, z);
        });
```

{% hint style="info" %}
你還可以把 ArgParserService 注入到 可進行依賴注入的實例中。
{% endhint %}
