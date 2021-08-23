---
description: After v0.0.6.
---

# Dynamic Arguments

Dynamic Argument is for a command argument which can have more than one type.  
Examples:

```java
@Commander(
        name = "sleep",
        description = "sleep command"
)
public class TestSleepCommand implements CommandNode {


    // this arg can be either long or string type
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
DynamicArg must be an **Object type** or it will throw error.
{% endhint %}



