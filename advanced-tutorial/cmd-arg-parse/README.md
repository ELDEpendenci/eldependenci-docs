# 指令參數解析

{% hint style="warning" %}
在開始本教程之前，請先細閱[快速開始](../../quick-start.md)。
{% endhint %}

以下為一個計算器指令。

```java
@Commander(
        name = "calculate",
        description = "test calculate command",
        alias = {"cal", "c"}
)
public class TestCalculateCommand implements CommandNode {

    @Override
    public void execute(CommandSender commandSender) {
    }
}
```

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

```java
@Commander(
        name = "minus",
        description = "minus command",
        alias = {"reduce", "m"}
)
public class TestCalculateMinusCommand implements CommandNode {

    @CommandArg(order = 0)
    private int one;

    @CommandArg(order = 1)
    private int two;

    @Override
    public void execute(CommandSender commandSender) {
        commandSender.sendMessage(one+" - "+two+" = "+(one - two));
    }
}
```

聰明的你應該知道這是一個 加法指令與減法指令，接受的參數為兩個 integer 數值。他們的指令使用方式顯示將會是

* /calculate add \<one> \<two>
* /calculate minus \<one> \<two>

{% hint style="info" %}
CommandArg 的參數 order 將決定其參數順序。
{% endhint %}

接下來，在 CommandArgs 上修改一些

```java
    @CommandArg(order = 0, labels = {"first value"})
    private int one;

    @CommandArg(order = 1, labels = {"second value"}, optional = true)
    private int two = 22;
```

這樣，其指令使用方式顯示將會變成

* /calculate add \<first value> \[second value]
* /calculate minus \<first value> \[second value]

{% hint style="info" %}
\<arg> 參數為必填參數，\[arg] 參數為 可選參數 (optional = true)，而因為初始化了數值 two 為 22, 因此 當 指令輸入者沒有輸入參數 two 的時候，它的默認數值將為 22。
{% endhint %}

## 餘下參數 <a href="#remainargs" id="remainargs"></a>

在被標註的指令參數解析之後，餘下的指令參數將被忽略。若果想獲取它們，只需要聲明一個 List\<String> 變數並標註為 @RemainArgs

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

    @RemainArgs
    private List<String> args;

    @Override
    public void execute(CommandSender commandSender) {
        commandSender.sendMessage(one+" + "+two+" = "+(one + two));
        commandSender.sendMessage("remainArgs: "+args.toString());
    }
}
```

當你輸入 `/calculate add 1 1 a b c d e` 的時候，輸出的結果將為&#x20;

> 1 + 1 = 2
>
> remainArgs: \[a, b, c, d, e]

{% hint style="danger" %}
被標註 RemainArgs 的變數必須為 List\<String> , 否則會報錯。
{% endhint %}
