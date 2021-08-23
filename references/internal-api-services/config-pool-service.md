---
description: 此為 v0.0.7 後的功能。
---

# 文件池服務

使用，以指令為例。

```java
@Commander(
        name = "book",
        description = "book command"
)
public class TestBookCommand implements CommandNode {
    @Override
    public void execute(CommandSender commandSender) {

    }
}
```

```java
@Commander(
        name = "check",
        description = "book chceck command"
)
public class TestBookCheckCommand implements CommandNode {

    @Inject
    private ConfigPoolService service;

    // 文件名稱為 id ( 例如 test.yml, id 則為 test )
    @CommandArg(order = 0)
    private String book;

    @Override
    public void execute(CommandSender commandSender) {
        // 檢測文件池是被已經被加到快取
        commandSender.sendMessage("book config cached: "+service.isPoolCached(BookConfig.class));
        // 如果不是，則需要使用 async，否則會拿到 null
        service.getPoolAsync(BookConfig.class).thenRunSync(map -> {
            if (!map.containsKey(book)) {
                commandSender.sendMessage("book "+book+" is not exist.");
                return;
            }
            var bookContent = map.get(book);
            commandSender.sendMessage("id: "+bookContent.getId());
            commandSender.sendMessage("書名: "+ bookContent.title);
            commandSender.sendMessage("作者: "+bookContent.author);
            commandSender.sendMessage("書本簡介: "+bookContent.description);
            commandSender.sendMessage("書本總頁數: "+bookContent.pages);
            commandSender.sendMessage("書本內容: ");
            for (int i = 0; i < bookContent.contents.size(); i++) {
                commandSender.sendMessage(i+1+": "+bookContent.contents.get(i));
            }
        }).join();
    }
}
```

```java
@Commander(
        name = "reload",
        description = "book reload"
)
public class TestBookReloadCommand implements CommandNode {

    @Inject
    private ConfigPoolService service;

    @Override
    public void execute(CommandSender commandSender) {
        // 異步重載文件池
        service.reloadPool(BookConfig.class).whenComplete((v, ex) ->{
            if (ex != null) ex.printStackTrace();

            commandSender.sendMessage("reload completed!");
        });
    }
}
```

{% hint style="info" %}
有關 ConfigPoolService 的其他方法，請參閱 [javadocs](https://eric2788.github.io/ELDependenci/)。
{% endhint %}



