---
description: 此為 v0.0.7 後的功能。
---

# 文件池配置 \[NEW\]

文件池，又名文件組，指一個特定的文件夾內的所有yaml。  
該文件夾內的yaml必須符合統一的格式，以確保文件映射能有效運作。

### 範例

創建文件映射物件，該映射物件必須繼承 `GroupConfiguration`。  
然後，使用 `@GroupResource` 標註你的文件夾位置。 

```java
@GroupResource(folder = "Books")
public class BookConfig extends GroupConfiguration {

    public String title;
    public String author;
    public String description;
    public int pages;
    public List<String> contents;

}
```

Yaml 文件組 \(以書本為範例\)

{% code title="normal.yml" %}
```yaml
title: "正常的書名"
author: "正常的作者"
description: "真的很正常的一本書"
pages: 666
contents:
    - "這是一本很正常的書本"
    - "為什麼我會說很正常呢"
    - "因為真的很正常"
    - "正常到你察覺不了他的存在"
    - "oh 當你看到這則訊息的時候代表他重載得很正常。"
```
{% endcode %}

{% code title="test.yml" %}
```yaml
title: "測試書本"
author: "愛測試的作者"
description: "這是一本測試的書"
pages: 9999
contents:
    - "測試的重點"
    - "就是必須被測試"
    - "否則就無法測試"
    - "有測試才有實踐"
```
{% endcode %}

註冊文件池

```java
@ELDPlugin(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {

    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.addGroupConfiguration(BookConfig.class); //註冊
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {
       

    }
}
```

然後，便可以開始使用。

{% hint style="info" %}
有關文件池使用範例請參考[這裏](../../references/internal-api-services/config-pool-service.md)。
{% endhint %}

