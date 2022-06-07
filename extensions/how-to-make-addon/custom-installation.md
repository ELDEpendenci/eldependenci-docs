---
description: 自定義安裝為 v0.1.1 版本之後的重大更新之一。
---

# 自定義安裝

你還記得剛剛所說過的 安裝器 嗎？

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

在 v0.1.1 版本後，為了擺脫僅限 `ServiceCollection` 所提供的 method 上的安裝，新增了自定義安裝器的功能。

其新增方式在 `TutorialPlugin.java` 可見:

```java
@ELDBukkit(
        lifeCycle = TutorialLifeCycle.class,
        registry = TutorialRegistry.class
)
public class TutorialPlugin extends ELDBukkitPlugin {

    @Override
    public void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.bindService(ExampleService.class, ExampleServiceImpl.class);

        // 取出模組安裝器 (需要依賴 eldependenci-addon)
        AddonInstallation addonManager = serviceCollection.getInstallation(AddonInstallation.class);

        MyExampleInstallationImpl installation = new MyExampleInstallationImpl();
        // 添加安裝器
        addonManager.customInstallation(MyExampleInstallation.class, installation);
        // 安裝 guice module
        addonManager.installModule(new MyExampleModule(installation));
    }

    @Override
    protected void manageProvider(BukkitManagerProvider bukkitManagerProvider) {

    }
}
```

&#x20;這樣，你的使用者就可在掛接你的插件後使用如下方式安裝:

```java
@ELDBukkit(
        lifeCycle = LifeCycle.class,
        registry = Registry.class
)
public class JavaMain extends ELDBukkitPlugin {

    @Override
    public void bindServices(ServiceCollection serviceCollection) {
        // 獲取 該擴充插件 的註冊器
        MyExampleInstallation installation = serviceCollection.getInstallation(MyExampleInstallation.class);
        installation.putSomeValue("hello", "world");
        installation.putSomeValue("good", "work");
    }


    @Override
    protected void manageProvider(BukkitManagerProvider bukkitManagerProvider) {

    }
}
```

## 被註冊後的處理

關於這點，目前可提供的處理方式是用到 guice 的 Module 之中。

```java
/**
 * 這個是 guice module, 可到 guice wiki 查看
 */
public final class MyExampleModule extends AbstractModule {

    private final MyExampleInstallationImpl installation;

    public MyExampleModule(MyExampleInstallationImpl installation) {
        this.installation = installation;
    }

    @Override
    protected void configure() {
        // 把已被註冊的安裝器注入以在其他實例獲取
        bind(MyExampleInstallationImpl.class).toInstance(installation);
    }
}
```

{% hint style="warning" %}
關於 Guice Module 這裏不會詳細講述，請自行到 [guice wiki ](https://github.com/google/guice/wiki)查看。
{% endhint %}

以上的使用方式為把安裝器注入到你的服務之中，範例如下:

```java
public class ExampleServiceImpl implements ExampleService {

    // 獲取安裝了的實例
    @Inject
    private MyExampleInstallationImpl installation;

    @Override
    public void doSomethingCool() {
        Bukkit.getLogger().info("Hello World!!");
        Bukkit.getLogger().info("Registered: ");
        installation.getStringMap().forEach((k, v) -> {
            Bukkit.getLogger().info(k+": "+v);
        });
    }
}
```

