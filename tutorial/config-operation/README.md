---
description: 普通文件配置已經在快速開始敘述過，因此本教程將集中於文件配置操作。
---

# 文件配置操作

{% hint style="warning" %}
Please make sure you have read the [Quick Start](../../quick-start.md) before the continuous reading.
{% endhint %}

所有文件配置映射物件都有一個 method 為 `getController()` , 該操作器用於重載文件配置或儲存文件。

操作如下

```java
@Commander(
        name = "edit",
        description = "config edit command"
)
public class TestConfigEditCommand  implements CommandNode {

    private final Random random = new Random();

    @Inject
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
            config.getController().save(); // 儲存文件
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

    @Inject
    private TestConfig config;

    @Override
    public void execute(CommandSender commandSender) {
        config.getController().reload(); // 重載文件
        commandSender.sendMessage("reload completed");
    }
}
```

{% hint style="danger" %}
請不要嘗試在訊息文件使用 儲存操作 ，否則可能將導致資料丟失。
{% endhint %}

