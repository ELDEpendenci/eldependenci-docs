# 指令標註與分支架構

{% hint style="warning" %}
在開始本教程之前，請先細閱[快速開始](../quick-start.md)。
{% endhint %}

## 指令標註 <a id="annotation"></a>

完整的指令標註參數如下

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Commander {

    String name(); //名稱 (必填)

    String description(); //介紹 (必填)

    boolean playerOnly() default false; // 僅限玩家

    String permission() default ""; // 權限

    String[] alias() default {}; // 別名

}
```

{% hint style="warning" %}
playerOnly 並不會自動把 commandSender 轉換成 玩家類別，你仍然需要手動進行形態轉換。
{% endhint %}

## 分支架構 <a id="subcommands"></a>

在本框架，創建分支指令可謂極其簡單。在大型指令系統下，本框架有助你細分各個支點的功能。

```java
public class TesterRegistry implements ComponentsRegistry {

    // 本框架允許你無限創建分支指令。
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

