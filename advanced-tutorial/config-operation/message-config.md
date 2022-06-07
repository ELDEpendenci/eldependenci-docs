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
    @Override
    public void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.addConfiguration(LangConfig.class);
    }
```

### 操作訊息文件 <a href="#operate-lang" id="operate-lang"></a>

要操作訊息文件，只需要呼叫 `getLang()` 的方法，其操作器的方法如下

```java
/**
 * 訊息文件控制器
 */
public interface LangController {

    /**
     * 獲取訊息
     * @param node 節點
     * @return 包含前綴的訊息
     */
    String get(String node);

    /**
     * 獲取訊息，連帶訊息參數 {0} {1} {2}...
     * @param node 節點
     * @param args 訊息參數
     * @return 包含前綴的訊息
     */
    String get(String node, Object... args);


    /**
     * 獲取訊息，連帶訊息參數 %s %d %2.f...
     * @param node 節點
     * @param args 訊息參數
     * @return 包含前綴的訊息
     */
    String getF(String node, Object... args);

    /**
     * 獲取無前綴的訊息
     * @param node 節點
     * @return 無前綴訊息
     */
    String getPure(String node);

    /**
     * 獲取無前綴的訊息，連帶訊息參數 {0} {1} {2}...
     * @param node 節點
     * @param args 訊息參數
     * @return 無前綴訊息
     */
    String getPure(String node, Object... args);


    /**
     * 獲取無前綴的訊息，連帶訊息參數  %s %d %2.f...
     * @param node 節點
     * @param args 訊息參數
     * @return 無前綴訊息
     */
    String getPureF(String node, Object... args);

    /**
     * 獲取訊息列表
     * @param node 節點
     * @return 含前綴訊息列表
     */
    List<String> getList(String node);


    /**
     * 獲取無前綴訊息列表
     * @param node 節點
     * @return 無前綴訊息列表
     */
    List<String> getPureList(String node);

    /**
     * 獲取前綴
     * @return 前綴
     */
    String getPrefix();

}

```

{% hint style="info" %}
以上所有獲取的訊息都已經自動轉換了顏色代碼。
{% endhint %}
