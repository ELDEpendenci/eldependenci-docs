# 文件池服務

文件池服務包含的方法如下:

```java
public interface ConfigPoolService {

    /**
     * 獲取 指定類型 的文件池
     * @param type 指定類型
     * @param <T> 文件組類別
     * @return 文件池
     */
    <T extends GroupConfiguration> GroupConfig<T> getGroupConfig(Class<T> type);

    /**
     * 獲取 指定類型 的語言文件池
     * @param type 指定類型
     * @param <T> 語言文件池類別
     * @return 語言文件池
     */
    <T extends GroupLangConfiguration> GroupLang<T> getGroupLang(Class<T> type);

}
```

相對於直接注入指定類型的 `GroupConfig<T>` 或 `GroupLang<T>` 使用，你可以透過注入文件池服務來獲取動態的文件池類別，在處理未知的文件池類別時非常有用。
