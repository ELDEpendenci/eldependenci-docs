# 日誌輸出

在正常情況下，你可以透過注入插件實例來獲取 `Logger` 而使用日誌輸出。但有時候你需要透過日誌輸出來 debug 時，如果在處理完畢後不刪走用於其日誌輸出的程式碼，可能會導致洗屏或導致控制台輸出內容過多。

本框架的 `LoggingService` 除了提供一系列基本的日誌輸出操作外，也支援透過 `config.yml` 中檢查是否啟用 `debug logging` 來控制你的日誌輸出。

{% code title="ELDependenci/config.yml" %}
```yaml
sharePluginInstance: false
defaultSingleton: true
fileWalker: TREE
debugLogging: true # 是否啟用日誌輸出
```
{% endcode %}

在 `config.yml` 中， `debugLogging` 的數值默認為 `false`, 在啟用後，你將可以透過以下代碼輸出 debug 日誌。

```java
public class TesterLifeCycle implements ELDLifeCycle {

    @Inject
    private LoggingService loggingService;

    @Override
    public void onEnable(JavaPlugin javaPlugin) {
        var logger = loggingService.getLogger(ELDTester.class);
        logger.debug("this {0} is {1}", "logger", "debug logger");
        logger.debug(new Exception("this is a debug exception"), "this is a debug message");
        logger.info("this is a info message");
        logger.warn("this is a warn message");
    }

    @Override
    public void onDisable(JavaPlugin javaPlugin) {

    }
}

```

在 `config.yml` 啟用 `debugLogging` 後，輸出如下:

![](<../../.gitbook/assets/image (6).png>)
