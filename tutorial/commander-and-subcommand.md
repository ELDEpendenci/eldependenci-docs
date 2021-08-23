# Annotation And Sub Command

{% hint style="warning" %}
Please make sure you have read the [Quick Start](../quick-start.md) before the continuous reading.
{% endhint %}

## Annotation <a id="annotation"></a>

Here's the annotation of command

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Commander {

    String name(); // command name (compulsory)

    String description(); //command description (compulsory)

    boolean playerOnly() default false; // command is only player

    String permission() default ""; // command permission

    String[] alias() default {}; // command alias

}
```

{% hint style="warning" %}
playerOnly will not auto cast CommandSender into Player type，you still need to cast it manually.
{% endhint %}

## Sub Command Structure <a id="subcommands"></a>

With this framework, create infinity sub commands are extremely easy.

```java
public class TesterRegistry implements ComponentsRegistry {

    @Override
    public void registerCommand(CommandRegistry commandRegistry) {
        commandRegistry.command(TestCommand.class, c -> {

            c.command(TestSayCommand.class);

            c.command(TestCalculateCommand.class, cc -> {

                cc.command(TestCalculateAddCommand.class);

                cc.command(TestCalculateMinusCommand.class);

            });

            c.command(TestConfigCommand.class, cc -> {

                cc.command(TestConfigCheckCommand.class);

                cc.command(TestConfigEditCommand.class);

                cc.command(TestConfigReloadCommand.class);

            });

            c.command(TestServiceCommand.class, cc -> {

                cc.command(TestServiceByeCommand.class);

                cc.command(TestServiceHelloCommand.class);

            });

            c.command(TestSchedulerCommand.class, cc ->{

                cc.command(TestSchedulerOneCommand.class);

                cc.command(TestSchedulerTwoCommand.class);

            });
        });
    }

    @Override
    public void registerListeners(ListenerRegistry listenerRegistry) {
    }


}
```

