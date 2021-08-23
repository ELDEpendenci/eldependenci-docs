---
description: 此功能採用了鏈式dsl設計編輯/新增訊息。
---

# 訊息編輯

{% hint style="info" %}
你可以直接參閱 [Javadocs](https://eric2788.github.io/ELDependenci/) 來查看教學。
{% endhint %}

```java
@Commander(
        name = "test",
        description = "test command",
        alias = {"tes", "te"},
        playerOnly = true
)
public class TestCommand implements CommandNode {

    @Inject
    private MessageService service;

    @Override
    public void execute(CommandSender commandSender) {
        var player = (Player)commandSender;
        var message = 
                service.edit("&athis is a text!!")
                .add("&6extra!!")
                .nextLine()
                .add("&eafter nextline!")
                .command("/test")
                .suggest("/test")
                .url("https://github.com/eric2788")
                .hoverText(new Text("hover text!!"))
                .build();
        player.spigot().sendMessage(message);
    }

}

```

