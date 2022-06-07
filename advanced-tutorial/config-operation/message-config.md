---
description: 普通文件配置已經在快速開始敘述過，因此本教程將集中於訊息文件配置。
---

# 訊息文件配置

配置基本相同，唯一不同的是，你並不需要在映射物件聲明任何變數，以及需要在物件上加上 @Prefix 標註。

{% code title="lang.yml" %}
```yaml
prefix: "&a[Plugin]&r"
hello: "&ahello world!"
bye: "&7good bye!"
```
{% endcode %}

映射物件如下

```java
@Prefix(path = "prefix")
@Resource(locate = "lang.yml")
public class LangConfig extends LangConfiguration {
}
```

最後註冊

```java
@ELDPlugin(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.addConfiguration(LangConfig.class);
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {
        
    }
}
```

### 操作訊息文件 <a href="#operate-lang" id="operate-lang"></a>

要操作訊息文件，只需要呼叫 `getLang()` 的方法，其操作器的方法如下

```java
public interface LangController {
    String get(String var1); // 透過路徑獲取訊息

    String getPure(String var1); //透過路徑獲取無前綴訊息

    List<String> getList(String var1); //透過路徑獲取訊息列表

    List<String> getPureList(String var1); //透過路徑獲取無前綴訊息列表

    String getPrefix(); //獲取前綴
}

```

{% hint style="info" %}
以上所有獲取的訊息都已經自動轉換了顏色代碼。
{% endhint %}
