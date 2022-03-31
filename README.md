---
description: ELDependenci 是基於依賴注入的 PaperSpigot 插件開發框架。
---

# 歡迎使用 ELDependenci

{% hint style="success" %}
本框架目前正在為伺服器團隊提供開發協助和咨詢，如有興趣，歡迎[聯絡](https://discord.gg/S5HNtQqzXe)。
{% endhint %}

{% hint style="info" %}
&#x20;從 v0.1.5 起，本插件將不再支援 1.16.5 或以下的版本，且採用 jdk16 運行。
{% endhint %}

### 前言 <a href="#intro" id="intro"></a>

本框架模仿了 ASP .NET 的大量使用依賴注入的方式，從實例注入，服務注入到文檔映射物件的注入。所有的注入實例可在生命週期，指令，事件監聽器以及所有已注入的實例通用。此外，服務注入提供了基於接口編程的API開發環境，在實現 SOLID 原則的時候更為便利，並增加可擴展性與可維護性。

### 優勢 <a href="#pros" id="pros"></a>

在依賴注入的優勢下，你將不需要

* 創建大量的 static 變數
* 在實例中初始化其他實例，造成高耦合問題
* 大量從構造器中注入單例

此外，除了依賴注入外，此框架也創建了與bukkit原版不同的編寫方式。其優勢在於

* 極為簡單的分支指令創建方式
* 指令參數預解析
* 訂閱事件並進行過濾處理
* 極為簡單的文件YAML配置操作(使用物件關聯)
* 極為簡單地處理大量統一格式的文件



### 本文檔將帶你走進 <a href="#index" id="index"></a>

* 如何使用這個框架創建你的第一個 spigot 插件
* 如何創建指令，監聽器與設置文件
* 如何註冊你的第一個實例
* 如何使用框架編寫你的第一個API (Service)
* 框架內置 API (Service) 使用教學



{% hint style="warning" %}
**在閱讀本文檔之前，請先注意以下事項**

* 本框架以 Guice 作為 DI 的主要依賴庫
* 本框架以 Paper 1.17.1 插件作為編寫基礎
* 本框架採用 jdk16 運行
{% endhint %}

{% hint style="info" %}
#### 除此之外，你可能需要先了解

* [何謂依賴注入](https://matthung0807.blogspot.com/2019/08/java-dependency-injection.html)
* [bukkit 插件基本開發知識](https://bukkit.gamepedia.com/Plugin\_Tutorial)
* [Maven 的使用教學](https://kentyeh.github.io/mavenStartup/)
{% endhint %}

****



