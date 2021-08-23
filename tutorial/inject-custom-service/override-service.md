---
description: After v0.0.6.
---

# Overriding Services

In some situation, you would like to let developers implement your services.  
We provided overriding services after v0.0.6, the implementation must extend `Overridable`.  
Here's the example shows the working flow like `Vault` .

### Economy Service Middleware <a id="econ-middleware"></a>

First define an economy service

```java
public interface EconomyService extends Overridable {

    Result giveMoney(Player player, double money);

    Result takeMoney(Player player, double money);

    Result setMoney(Player player, double money);

    enum Result {
        SUCCESS, FAILED
    }
}
```

implement a default economy service. In this example, we throw an exception that developers not implement any class.

```java
public class DefaultEconomyService implements EconomyService {

    @Override
    public Result giveMoney(Player player, double money) {
        throw new IllegalStateException("not implement any class!");
    }

    @Override
    public Result takeMoney(Player player, double money) {
        throw new IllegalStateException("not implement any class!");
    }

    @Override
    public Result setMoney(Player player, double money) {
        throw new IllegalStateException("not implement any class!");
    }
}
```

Register in main class

```java
    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        // register economy service
        serviceCollection.bindService(EconomyService.class, DefaultEconomyService.class);
    }
```

### Economy Service Provider \(Economy Plugin\) <a id="econ-provider"></a>

First make the economy service middleware plugin as your dependency for your plugin, then implement the Economy Service.

```java
public class MyEconomyService implements EconomyService{
    
    // only for demo, you should use I/O for offline
    private final Map<Player, Double> economy = new ConcurrentHashMap<>();
    
    @Override
    public Result giveMoney(Player player, double money) {
        var balance = economy.getOrDefault(player, 0.0);
        economy.put(player, balance + money);
        return Result.SUCCESS;
    }

    @Override
    public Result takeMoney(Player player, double money) {
        var balance = economy.getOrDefault(player, 0.0);
        if (balance < money) return Result.FAILED;
        economy.put(player, balance - money);
        return Result.SUCCESS;
    }

    @Override
    public Result setMoney(Player player, double money) {
        economy.put(player, money);
        return Result.SUCCESS;
    }
}
```

Finally, register in main class.

```java
    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        // you need to ues overrideService method or it will be ignored.
        serviceCollection.overrideService(EconomyService.class, MyEconomyService.class);
    }
```

### The plugin that using the economy service \(Any\) <a id="econ-user"></a>

Finally,`EconomyService` will be injected by other instances.

```java
public class ShopManager {
    
    @Inject
    private EconomyService economyService;
    
    private final Map<String, Double> pricing = new ConcurrentHashMap<>();
    
    public void addProduct(String product, double price){
        this.pricing.put(product, price);
    }
    
    public void removeProduct(String product){
        this.pricing.remove(product);
    }
    
    public void buyProduct(Player player, String product){
        var price = pricing.get(product);
        if (price == null) {
            player.sendMessage("Unknown Product: "+product);
            return;
        }
        var result = economyService.takeMoney(player, price);
        if (result == EconomyService.Result.SUCCESS){
            player.sendMessage("Purchased successfully.");
            // 給予商品
        }else if (result == EconomyService.Result.FAILED){
            player.sendMessage("Purchased failed, insufficient money.");
        }
    }
    
    public void sellProduct(Player player, String product){
        var price = pricing.get(product);
        if (price == null) {
            player.sendMessage("Unknown Product: "+product);
            return;
        }
        var result = economyService.giveMoney(player, price);
        if (result == EconomyService.Result.SUCCESS){
            player.sendMessage("Sold successfully.");
            // 收回商品
        }else if (result == EconomyService.Result.FAILED){
            player.sendMessage("sold failed.");
        }
    }
}
```

And of course! You need to register it via main class.

```java
    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.addSingleton(ShopManager.class);
    }
```

{% hint style="success" %}
After registering ShopManager, it can be injected by other instance.
{% endhint %}







