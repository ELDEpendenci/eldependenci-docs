---
description: 本頁將使用 Guice 的 wiki 中的範例作為介紹。
---

# 注入服務提供器

這種注入方式是透過繼承 Provider\<T> 去獲取其實例，其代碼如下:

```java
public interface Provider<T> {
  T get();
}
```

範例如下:

```java
public class DatabaseTransactionLogProvider implements Provider<TransactionLog> {
  private final Connection connection;

  @Inject
  public DatabaseTransactionLogProvider(Connection connection) {
    this.connection = connection;
  }

  public TransactionLog get() {
    DatabaseTransactionLog transactionLog = new DatabaseTransactionLog();
    transactionLog.setConnection(connection);
    return transactionLog;
  }
}
```

完成後，到主類註冊。

```java
@ELDPlugin(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {


    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.bindServiceProvider(TransactionLog.class, DatabaseTransactionLogProvider.class);
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {

    }
}
```

## 注入 Provider\<T>

除了直接注入 `TransactionLog` 外, 你也可以直接注入 `Provider<TransactionLog>` 來每次獲取新的實例。

```java
public class LogService {

   @Inject
   private Provider<TransactionLog> logProvider;

   public void log(String message){
      TransactionLog logger = logProvider.get();
      // do something ....
   }
}
```

