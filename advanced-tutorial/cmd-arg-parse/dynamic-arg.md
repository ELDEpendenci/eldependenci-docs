---
description: 此為版本 v0.0.6 後的功能。
---

# 多形態參數

多形態參數是為了應對一個參數將可以擁有多形態的轉換。\
範例如下:

```java
@Commander(
        name = "sleep",
        description = "sleep command"
)
public class TestSleepCommand implements CommandNode {


    // 此參數擁有 long 和 string 兩個形態
    @DynamicArg(order = 0, types = { Long.class, String.class })
    private Object seconds;

    @Inject
    private ScheduleService scheduleService;

    @Override
    public void execute(CommandSender commandSender) {
        if (seconds instanceof Long){
            commandSender.sendMessage("sleeping...");
            var time = (Long) seconds;
            scheduleService.injectTask(new BukkitRunnable() {
                @Override
                public void run() {
                    commandSender.sendMessage("you wake up!");
                    commandSender.sendMessage("you just slept "+seconds+" seconds!");
                }
            }).timeout(time * 20L).run(ELDTester.getPlugin(ELDTester.class));
        }else if (seconds instanceof String){
            var time = (String) seconds;
            switch (time.toLowerCase()){
                case "never":
                    commandSender.sendMessage("you don't sleep ? okay you never sleep");
                    return;
                case "forever":
                    commandSender.sendMessage("you sleep forever ? that's not good!");
                    return;
                default:
                    commandSender.sendMessage("oh sorry what is "+time+" ? I don't understand.");
            }
        }
    }
}
```

{% hint style="danger" %}
多形態參數必須為 Object 類型，否則會報錯。
{% endhint %}
