# 文件池配置

文件池，又名文件組，指一個特定的文件夾內的所有yaml。  
該文件夾內的yaml必須符合統一的格式，以確保文件映射能有效運作。

### 範例

創建文件映射物件，該映射物件必須繼承 `GroupConfiguration`。  
然後，使用 `@GroupResource` 標註你的文件夾位置。 

```java
@GroupResource(folder = "Books")
public class Book extends GroupConfiguration {

    public String title;
    public String author;
    public String description;
    public int pages;
    public List<String> contents;

}
```

`@GroupResource` 有兩個屬性，第一個是必填的 `folder`, 第二個是選填的 `preloads`. 在 `preloads` 中，你可以透過填入 文件名稱 \(可以多個\) 來預先複製 jar 內的文件到插件資料夾內。

```java
@GroupResource(
     folder = "Books",
     preloads = { "internal" } // 假設你 jar 內有 /Books/internal.yml 的文件
)
public class Book extends GroupConfiguration {

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
        serviceCollection.addGroupConfiguration(Book.class); //註冊
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {
       

    }
}
```

然後，便可以開始使用。

### 使用

使用可以透過 `GroupConfig<T>` 進行注入，但在標註上並非使用 `@Inject`，而是使用 `@InjectPool` 。

{% hint style="danger" %}
請勿使用 `@Inject` 注入 `GroupConfig<T>`， 否則會報錯。
{% endhint %}

從上述範例當中，我們註冊了 `Book` 作為 文件池組別，使用則如下範例:

```java
@Commander(
        name = "check",
        description = "book chceck command"
)
public class TestBookCheckCommand implements CommandNode {

    @InjectPool // 使用 @InjectPool 而不是 @Inject
    private GroupConfig<Book> bookGroupConfig; // 注入

    @CommandArg(order = 0)
    private String book;

    @Override
    public void execute(CommandSender commandSender) {
        Optional<Book> gp = bookGroupConfig.findById(book);
        if (gp.isEmpty()){
            commandSender.sendMessage("book "+book+" is not exist.");
            return;
        }
        var bookContent = gp.get();
        commandSender.sendMessage("id: "+bookContent.getId());
        commandSender.sendMessage("書名: "+ bookContent.title);
        commandSender.sendMessage("作者: "+bookContent.author);
        commandSender.sendMessage("書本簡介: "+bookContent.description);
        commandSender.sendMessage("書本總頁數: "+bookContent.pages);
        commandSender.sendMessage("書本內容: ");
        for (int i = 0; i < bookContent.contents.size(); i++) {
            commandSender.sendMessage(i+1+": "+bookContent.contents.get(i));
        }
    }
}

```

`GroupConfig<T>` 採用 DAO pattern，其源碼如下:

```java
public interface GroupConfig<T extends GroupConfiguration> {

    /**
     *
     * @return 所有文件實例
     */
    List<T> findAll();

    /**
     * 根據 id 尋找文件實例
     * @param id 標識 id
     * @return 文件實例
     */
    Optional<T> findById(String id);

    /**
     * 保存一個文件，標識 id 不能為 null
     * @param config 文件實例
     */
    void save(T config);

    /**
     * 透過 id 刪除文件
     * @param id 標識 id
     * @return 刪除成功
     */
    boolean deleteById(String id);

    /**
     * 獲取文件池總數量
     * @return 數量
     */
    long totalSize();

    /**
     * 刪除文件
     * @param config 文件實例，id 不能為 null
     * @return 刪除成功
     */
    boolean delete(T config);

    /**
     * 清楚所有快取
     */
    void fetch();

    /**
     * 清楚指定文件的快取
     * @param id 標識 id
     */
    void fetchById(String id);

}
```

{% hint style="warning" %}
要注意的是，`GroupConfig<T>` 本身的操作並不是異步的，你需要自行實作異步。但它本身亦有快取功能，能大程度上加快二次獲取。

有時候你可能需要手動去獲取特定類型的 `GroupConfig<T>`，這時候你應該使用[文件池服務](../../references/internal-api-services/config-pool-service.md)。
{% endhint %}

### 文件池的讀寫操作\[NEW\]

v0.1.3 之後，文件池將允許刪除，創建及更新指定文件。

`GroupConfig<T>` 在獲取指定文件之後，將會把其儲存到快取，直到被刪除和更新。若要手動清除快取，可以調用 `fetch` 或者 `fetchById` 來清除指定文件。

```java
@Commander(
        name = "reload",
        description = "book reload"
)
public class TestBookReloadCommand implements CommandNode {

    @InjectPool
    private GroupConfig<Book> groupConfig;

    @Override
    public void execute(CommandSender commandSender) {
        groupConfig.fetch(); // 清除快取後，下次獲取將直接從文件加載
        commandSender.sendMessage("reloaded");
    }
}
```

除此之外，你也可以透過 文件池 創建文件。範例如下:

```java
@Commander(
        name = "add",
        description = "add book"
)
public class TestBookAddCommand implements CommandNode {


    @InjectPool
    private GroupConfig<Book> groupConfig;

    @Inject
    private ScheduleService scheduleService; // 如需使用異步，請自行實現

    @Inject
    private ELDTester plugin;

    @CommandArg(order = 1)
    private String id;

    @CommandArg(order = 1, optional = true)
    private String author = "unknown";

    @Override
    public void execute(CommandSender sender) {
        Book book = new Book();
        book.setId(id); // 記得設置標識 id, 否則會報錯
        book.title = "This is a title with id "+id;
        book.author = author;
        book.description = "book with id "+id;
        book.contents = Arrays.asList("this", "is", "a", "content");

        scheduleService
                .runAsync(plugin, () -> groupConfig.save(book))
                .thenRunSync(v -> sender.sendMessage("save completed")).join();
    }
}
```

刪除指定文件的範例則如下:

```java
@Commander(
        name = "delete",
        description = "delete book"
)
public class TestBookDeleteCommand  implements CommandNode {

    @Inject
    private ScheduleService scheduleService;

    @Inject
    private ELDTester plugin;

    @InjectPool
    private GroupConfig<Book> groupConfig;


    @CommandArg(order = 1)
    private String id;

    @Override
    public void execute(CommandSender sender) {
        scheduleService
                .callAsync(plugin, () -> groupConfig.deleteById(id))
                .thenRunSync(result -> sender.sendMessage("delete "+(result ? "success" : "failed"))).join();
    }
}

```

