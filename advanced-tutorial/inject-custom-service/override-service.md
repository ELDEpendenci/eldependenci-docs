---
description: 此為版本 v0.0.6 後的功能。
---

# 覆蓋原有服務

在某些情況下, 你可能需要讓其他插件師註冊你的服務，以進行掛接操作。\
版本 v0.0.6 後提供了 覆蓋原有服務類，該服務必須繼承 Overridable。\
我們以類似 Vault 的中介方式作為範例。

### 以下為經濟服務中介插件 <a href="#econ-middleware" id="econ-middleware"></a>

首先定義經濟服務

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

實作默認的經濟服務 (這邊的默認將拋出沒有註冊的錯誤)

```java
public class DefaultEconomyService implements EconomyService {

    @Override
    public Result giveMoney(Player player, double money) {
        throw new IllegalStateException("未註冊任何的經濟插件!");
    }

    @Override
    public Result takeMoney(Player player, double money) {
        throw new IllegalStateException("未註冊任何的經濟插件!");
    }

    @Override
    public Result setMoney(Player player, double money) {
        throw new IllegalStateException("未註冊任何的經濟插件!");
    }
}
```

最後在主類註冊

```java
    @Override
    public void bindServices(ServiceCollection serviceCollection) {
        // 註冊經濟服務
        serviceCollection.bindService(EconomyService.class, DefaultEconomyService.class);
    }
```

### 經濟服務提供插件 (經濟插件) <a href="#econ-provider" id="econ-provider"></a>

首先掛接經濟服務中介插件，然後實作自己的經濟服務。

```java
public class MyEconomyService implements EconomyService{
    
    // 僅為範例，真正的經濟插件必須有 資料 I/O 作為離線儲存。
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

最後，到主類註冊。

```java
    @Override
    public void bindServices(ServiceCollection serviceCollection) {
        // 這邊需要使用覆蓋註冊，否則將被忽略
        serviceCollection.overrideService(EconomyService.class, MyEconomyService.class);
    }
```

### 使用經濟服務的插件 (任何) <a href="#econ-user" id="econ-user"></a>

最後由其他插件掛接經濟服務中介插件並注入經濟服務並使用，範例如下

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
            player.sendMessage("未知商品: "+product);
            return;
        }
        var result = economyService.takeMoney(player, price);
        if (result == EconomyService.Result.SUCCESS){
            player.sendMessage("商品購買成功");
            // 給予商品
        }else if (result == EconomyService.Result.FAILED){
            player.sendMessage("商品購買失敗，金錢不足!");
        }
    }
    
    public void sellProduct(Player player, String product){
        var price = pricing.get(product);
        if (price == null) {
            player.sendMessage("未知商品: "+product);
            return;
        }
        var result = economyService.giveMoney(player, price);
        if (result == EconomyService.Result.SUCCESS){
            player.sendMessage("商品出售成功");
            // 收回商品
        }else if (result == EconomyService.Result.FAILED){
            player.sendMessage("商品出售失敗!");
        }
    }
}
```

當然，別忘了任何使用依賴注入的單例都需要註冊。

```java
    @Override
    public void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.addSingleton(ShopManager.class);
    }
```

{% hint style="success" %}
在註冊 ShopManager 的同時，它也將可被其他單例注入使用。&#x20;
{% endhint %}





