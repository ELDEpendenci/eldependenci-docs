---
description: 教程將以擴充插件範本提供的範例為主。
---

# 擴充插件範例詳解

在範例中，你可以看到一個module下有兩個module，分別是 API Module 和 Plugin Module，他們分別是 API 模組 及 伺服器插件模組。\
而伺服器插件模組是依賴於 API 模組的，因此伺服器插件建置的時候會包裝 API 模組以創建完整的伺服器插件。

{% hint style="info" %}
架構大致如下

* API Module - itself
* Plugin Module - itself + API Module
{% endhint %}

## API Module 詳解 <a href="#api-module-explain" id="api-module-explain"></a>

API Module 內的檔案很少，此取決於你提供多少服務 (API) 予你的使用者。以 API Module 為例，此模組只提供了一個服務給使用者，它就是 `ExampleService`。

```java
/**
 * 範例服務
 */
public interface ExampleService {

    /**
     * 做一些很酷的事情!
     */
    void doSomethingCool();

}
```

{% hint style="danger" %}
如果你需要加自定義參數，或者需要返回一個自定義的類型，**請務必使用 interface 作為其類。**\
****那是因為在 API Module 這個提供接口的模組，你應確保檔案內沒有任何實作的類別，以確保遵從[依賴反轉原則](https://medium.com/@f40507777/%E4%BE%9D%E8%B3%B4%E5%8F%8D%E8%BD%89%E5%8E%9F%E5%89%87-dependency-inversion-principle-dip-bc0ba2e3a388)。
{% endhint %}

除此之外，它還有一個 class，**但這個並不是對外服務，而是安裝器**。

```java
/**
 * 我的安裝
 */
public interface MyExampleInstallation {

    /**
     * 放一些 String 進去
     * @param key key
     * @param value value
     */
    void putSomeValue(String key, String value);

}
```

**關於這個我們將在後頁講解。**

## Plugin Module 詳解 <a href="#plugin-module-explain" id="plugin-module-explain"></a>

Plugin Module 是實作 API Module 內所有 服務的 模組，因此需要依賴於 API Module。\
它是實現服務運作的核心，因此他也是最後建置成伺服器插件的模組。

{% hint style="info" %}
在 v0.1.1 之後，出現了幾個除實例和ELD三個組件以外的 class。關於這點我們將在後頁進行講解。
{% endhint %}

`ExampleServiceImpl` 是實現 `ExampleService` 的 class。

```java
public class ExampleServiceImpl implements ExampleService {

    @Override
    public void doSomethingCool() {
        Bukkit.getLogger().info("Hello World!!");
    }
}
```

剩下的三個 classes， 分別是註冊為 ELD 插件所需的三個組件。

{% tabs %}
{% tab title="生命週期" %}
{% code title="TutorialLifeCycle.java" %}
```java
public class TutorialLifeCycle implements ELDLifeCycle {

    @Inject // 實現自我使用而注入
    private ExampleService service;

    @Override
    public void onEnable(JavaPlugin javaPlugin) {
        service.doSomethingCool();
    }

    @Override
    public void onDisable(JavaPlugin javaPlugin) {
        service.doSomethingCool();
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="組件註冊" %}
{% code title="TutorialRegistry.java" %}
```java
public class TutorialRegistry implements ComponentsRegistry {

    @Override
    public void registerCommand(CommandRegistry commandRegistry) {
        // no command
    }

    @Override
    public void registerListeners(ListenerRegistry listenerRegistry) {
        // no listeners
    }

}
```
{% endcode %}
{% endtab %}

{% tab title="插件主類" %}
{% code title="TutorialPlugin.java" %}
```java
@ELDPlugin(
        lifeCycle = TutorialLifeCycle.class,
        registry = TutorialRegistry.class
)
// v0.1.1 後，繼承的是 ELDBukkitAddon, 而不是 ELDBukkitPlugin
public class TutorialPlugin extends ELDBukkitAddon {

    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        // 這裏註冊你的服務
        serviceCollection.bindService(ExampleService.class, ExampleServiceImpl.class);
    }

    @Override
    protected void preAddonInstall(ManagerProvider managerProvider, AddonManager addonManager) {
       // ...這堆東西將在稍後詳解。
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="success" %}
當你建置完成後，會有兩個 jar , 分別是 API Module 的 jar 及 Plugin Module jar 。他們的職責如下:&#x20;

API Module 生成的 jar:  給予 其他插件師 進行掛接與使用

Plugin Module 生成的 jar:  給予 服主 放入伺服器運行
{% endhint %}

## 為什麼要分開模組進行開發？ <a href="#why-separate-template" id="why-separate-template"></a>

這是遵從了  [**基於介面編程**](https://zh.wikipedia.org/wiki/%E5%9F%BA%E4%BA%8E%E6%8E%A5%E5%8F%A3%E7%BC%96%E7%A8%8B)  ****  中一句重要的名言:

> [Program to an interface, not an implementation.](http://teddy-chen-tw.blogspot.com/2010/06/program-to-interface.html)

此方式在一定程度上確保了插件的可維護性，以及讓插件師專注於接口使用而非實作本身。

以範例為例，由於 Plugin Module 是 實現 API Module 所有服務 的 模組，因此當你需要修改實作程式碼的時候將不需要更改由其他插件師掛接並使用的 API Module , 提高可維護性。

