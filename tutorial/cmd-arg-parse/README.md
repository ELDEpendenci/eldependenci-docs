# Command Arguments

{% hint style="warning" %}
Please make sure you have read the [Quick Start](../../quick-start.md) before the continuous reading.
{% endhint %}

Below is a calculator command.

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

Well you should know they are plus calculation and minus calculation commands, accept with two integer arguments. The help message will show as below:

* /calculate add &lt;one&gt; &lt;two&gt;
* /calculate minus &lt;one&gt; &lt;two&gt;

{% hint style="info" %}
`order` decided the order of arguments.
{% endhint %}

Now, we try edit something in @CommandArg

```java
    @CommandArg(order = 0, labels = {"first value"})
    private int one;

    @CommandArg(order = 1, labels = {"second value"}, optional = true)
    private int two = 22;
```

The help message will now show as below:

* /calculate add &lt;first value&gt; \[second value\]
* /calculate minus &lt;first value&gt; \[second value\]

{% hint style="info" %}
&lt;arg&gt; is a compulsory argument while \[arg\] is an optional argument \(optional = true\). you can see that property `two` has been assigned as 22, so when command sender didn't input the second value, the default value of second value will be 22.
{% endhint %}

## Remain Arguments <a id="remainargs"></a>

After parsing declared command arguments, the remaining arguments will be ignored. If you want to take them, just declare a variable with type List&lt;String&gt; and annotating with @RemainArgs

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

when you input `/calculate add 1 1 a b c d e` the output result will be

> 1 + 1 = 2
>
> remainArgs: \[a, b, c, d, e\]

{% hint style="danger" %}
the property type with annotated @RemainArgs must be List&lt;String&gt;, or else it will throw an error.
{% endhint %}

