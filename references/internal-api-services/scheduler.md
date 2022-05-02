---
description: 計時器的 Service 名稱為 ScheduleService
---

# 計時器

### 在 BukkitRunnable 使用依賴注入 <a href="#inject-bukkitrunnable" id="inject-bukkitrunnable"></a>

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

透過使用 injectTask(BukkitRunnable runnable) 進行計時器操作，該 BukkitRunnable 將能使用依賴注入以進行其他操作。

ScheduleService 源碼一覽 (只顯示了計時器部分)

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

### 異步與同步的切換執行與數值呼叫傳遞 <a href="#call-async" id="call-async"></a>

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

### 阻塞運行及等待多個異步事件運行完成

在 `v0.1.5` 版本後，新增了類似 `Promise.all` 的等待多個異步事件完成的方式。演示如下:

```java
@Commander(
        name = "four",
        description = "test scheduler four"
)
public class TestSchedulerFourCommand implements CommandNode {

    @Inject
    private ScheduleService scheduleService;
    @Inject
    private ELDTester plugin;

    // 是否為有序
    @CommandArg(order = 1, labels = "<with order>", optional = true)
    private boolean order = true;

    @Override
    public void execute(CommandSender commandSender) {
        // 異步事件 1
        var one = scheduleService.runAsync(plugin, () -> {
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            commandSender.sendMessage("one!");
        });
        // 異步事件 2
        var two = scheduleService.runAsync(plugin, () -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            commandSender.sendMessage("two!");
        });
        // 異步事件 3
        var three = scheduleService.runAsync(plugin, () -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            commandSender.sendMessage("three!");
        });

        if (order){
            // 有序使用方式
            commandSender.sendMessage("with order..");
            scheduleService.runAsync(plugin, () -> {
                try {
                    one.block(); // 阻塞運行, 因為在 runAsync 裡面所以可以安全使用。
                    two.block(); // 阻塞運行
                    three.block(); // 阻塞運行
                } catch (Throwable e) {
                    e.printStackTrace();
                } finally {
                    commandSender.sendMessage("done!");
                }

            }).join();
        }else{
            commandSender.sendMessage("without order..");
            // 無序運行，異步事件將會迸發運行
            scheduleService.runAllAsync(plugin, List.of(one, two, three)).join();
        }
    }
}
```
