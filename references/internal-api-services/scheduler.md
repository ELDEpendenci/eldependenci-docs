---
description: 計時器的 Service 名稱為 ScheduleService
---

# 計時器

### 在 BukkitRunnable 使用依賴注入 <a id="inject-bukkitrunnable"></a>

```java
@Commander(
        name = "one",
        description = "one scheduler"
)
public class TestSchedulerOneCommand implements CommandNode {


    @Inject
    private ScheduleService service;

    @Override
    public void execute(CommandSender commandSender) {
        commandSender.sendMessage("wait for 5 secs");
        service.injectTask(new BukkitRunnable() {

            @Inject // 使用依賴注入
            private ScheduleService service;

            @Override
            public void run() {
                commandSender.sendMessage("scheduler service instance inside bukkit runnable is "+(service == null ? "null" : "not null !"));
            }

        }).asynchronous(false) // 是否異步
        .timeout(100L) // 延遲
        .run(ELDTester.getProvidingPlugin(ELDTester.class));
    }
}
```

透過使用 injectTask\(BukkitRunnable runnable\) 進行計時器操作，該 BukkitRunnable 將能使用依賴注入以進行其他操作。

ScheduleService 源碼一覽 \(只顯示了計時器部分\)

```java
public interface ScheduleService {
    ScheduleService.ScheduleFactory injectTask(BukkitRunnable var1); //注入 task

    public interface ScheduleFactory {
        ScheduleService.ScheduleFactory asynchronous(boolean var1); // 異步

        ScheduleService.ScheduleFactory interval(long var1); //間隔

        ScheduleService.ScheduleFactory timeout(long var1); //延遲

        BukkitTask run(Plugin var1); // 運行
    }
}
```

### 異步與同步的切換執行與數值呼叫傳遞 <a id="call-async"></a>

有時候你需要從異步獲取數值然後放到同步執行。這種操作在java通常使用 CompletableFuture ，然而 CompletableFuture 對 編寫 bukkit 插件並不友好。此功能採用的是完全的 BukkitRunable, 對 編寫 bukkit 插件是完全兼容的。

```java
@Commander(
        name = "two",
        description = "two scheduler"
)
public class TestSchedulerTwoCommand implements CommandNode {


    @Inject
    private ScheduleService service;

    @Override
    public void execute(CommandSender commandSender) {
        commandSender.sendMessage("starting scheduler");
        service.callAsync(ELDTester.getProvidingPlugin(ELDTester.class), () -> {
            commandSender.sendMessage("sleep 5 seconds in async");
            Thread.sleep(5000); // 暫停執行五秒
            return "abc";
        }).thenApplySync(result -> {
            commandSender.sendMessage("get the result "+result+" in sync!");
            commandSender.sendMessage("sending number 123");
            return 123;
        }).thenApplyAsync(re -> {
            commandSender.sendMessage("received "+re+" in async!");
            commandSender.sendMessage("add 123 into "+re+" with 5 secs");
            Thread.sleep(5000); // 暫停執行五秒
            return re + 123;
        }).thenRunSync(res -> {
            commandSender.sendMessage("received "+res+" in sync!");
            commandSender.sendMessage("the final result is "+res);
        }).join();
    }
}
```

在上述的範例中，所有的 async 和 sync 都是使用了 BukkitRunnable 執行，在異步使用 `Thread.sleep(long)` 也沒有影響到 Bukkit main thread 的運行。



