---
description: create your own command argument parser.
---

# Custom command argument parser

Argument parser creation as below:

```java
@ELDPlugin(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {
        var parser = provider.getArgumentManager(); //argument parser
        // create new class parser
        parser.registerParser(Integer.class, (iterator, commandSender, argParser) -> {
            try{
                return Integer.parseInt(iterator.next());
            }catch (NumberFormatException e){
                throw new ArgumentParseException("not a valid integer."); 
            }
        });
    }
}
```

{% hint style="info" %}
throwing an `ArgumentParseException` will not throw an error to console. It will just stop executing command and notify to command sender.
{% endhint %}

The code showed above is for creating Integer class parser which use for parsing integer argument, like below:

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

You can mark an identifier to create multiple parser with same class.

```java
@ELDPlugin(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {
        var parser = provider.getArgumentManager(); //參數解析器
        // identifier is message
        parser.registerParser(String.class, "message", (iterator, commandSender, p) -> {
            StringBuilder builder = new StringBuilder();
            iterator.forEachRemaining(s -> builder.append(s).append(" "));
            return builder.toString();
        });
    }
}
```

Example as below:

```java
@Commander(
        name = "say",
        description = "test say command",
        alias = {"sa", "s"}
)
public class TestSayCommand implements CommandNode {

    @CommandArg(order = 0, identifier = "message")
    private String message;

    @Override
    public void execute(CommandSender commandSender) {
        Bukkit.broadcastMessage(commandSender.getName()+" says: "+message);
    }
}
```

when you input `/say this is a text` , message argument will be `"this is a text"`

## Using parser inside argument parser creation <a id="arg-parser"></a>

when you create anargument parser like `org.bukkit.Location` , you will find that x y z is a double class，then you can use the parser inside your creation.

```java
argumentManager.registerParser(Location.class, (args, sender, parser) -> {
            World world;
            if (!(sender instanceof Player)) {
                world = Bukkit.getWorld(args.next());
            } else {
                world = ((Player) sender).getWorld();
            }
            if (world == null) {
                throw new ArgumentParseException("&cunknown world");
            }
            var x = parser.tryParse(Double.class, args, sender); //內部解析
            var y = parser.tryParse(Double.class, args, sender); //內部解析
            var z = parser.tryParse(Double.class, args, sender); //內部解析
            return new Location(world, x, y, z);
        });
```

{% hint style="info" %}
you can also inject ArgParserService into your injectable instance.
{% endhint %}

