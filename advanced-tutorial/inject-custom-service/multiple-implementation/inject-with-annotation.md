---
description: 透過辨識標註注入在多重注入實例中的指定實例。
---

# 透過辨識標註進行注入

辨識標註有兩種，第一個是命名標註 `@Named` , 第二個是自定義標註。

## 命名標註

### 在 Map 中使用命名標註

我們以上一頁的註冊為例:

```java
    @Override
    public void bindServices(ServiceCollection serviceCollection) {
        serviceCollection.addServices(MyService.class, Map.of(
                "A", MyServiceA.class,
                "B", MyServiceB.class
        ));
    }
```

除了透過注入 `Map` 外，我們可以透過 `@Named(key)` 來獲取指定的實例。

```java
public class AnotherService {
    
    @Named("A") // 註冊時的 key
    @Inject
    private MyService serviceA; // 注入 ServiceA
   
   
    @Named("B") // 註冊時的 key
    @Inject
    private MyService serviceB; // 注入 ServiceB
}
```

### 在 Set 中使用命名標註

因為 Set 在註冊的時候沒有 key, 所以我們必須在定義實例的 class 的時候進行標註。

```java
@Named("A") // 在這裏進行標註
public class MyServiceA implements MyService {


    @Override
    public void sayHelloTo(CommandSender sender) {
        sender.sendMessage("hello world A!!!");
    }

    @Override
    public void sayGoodBye(CommandSender sender) {
        sender.sendMessage("good bye A!!!");
    }
}
```

```java
@Named("B") // 在這裏進行標註
public class MyServiceB implements MyService{

    @Override
    public void sayHelloTo(CommandSender sender) {
        sender.sendMessage("hello world B!!!");
    }

    @Override
    public void sayGoodBye(CommandSender sender) {
        sender.sendMessage("good bye B!!!");
    }

}
```

{% hint style="info" %}
獲取方式與 上述 Map 的例子 相同。
{% endhint %}

## 自定義標註

**自定義標註只能用於 Set 註冊。**首先，為每個特定實例創建一個標註(Annotation)且該標註需要標註(Annotate) `@Qualifier`

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Qualifier // 必須標註這個作為自定義標註
public @interface AService {
}
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Qualifier // 必須標註這個作為自定義標註
public @interface BService {
}
```

然後，跟 `@Named` 一樣在 實例的 class 上標註:

```java
@AService // 在這裏進行標註
public class MyServiceA implements MyService {


    @Override
    public void sayHelloTo(CommandSender sender) {
        sender.sendMessage("hello world A!!!");
    }

    @Override
    public void sayGoodBye(CommandSender sender) {
        sender.sendMessage("good bye A!!!");
    }
}
```

```java
@BService // 在這裏進行標註
public class MyServiceB implements MyService{

    @Override
    public void sayHelloTo(CommandSender sender) {
        sender.sendMessage("hello world B!!!");
    }

    @Override
    public void sayGoodBye(CommandSender sender) {
        sender.sendMessage("good bye B!!!");
    }

}
```

獲取方式則如下:

```java
public class AnotherService {
    
    @AService
    @Inject
    private MyService serviceA; // 注入 ServiceA
   
   
    @BService
    @Inject
    private MyService serviceB; // 注入 ServiceB
}
```
