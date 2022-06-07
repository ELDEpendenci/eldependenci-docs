# 框架內置指令參數解析

## 內置參數解析 class 列表一覽

* Integer
* Double
* Long
* Byte
* Short
* Float
* Character
* Boolean
* String
* Player
* OfflinePlayer
* Location
* UUID

{% hint style="info" %}
解析源碼可參考[這裏](https://github.com/eric2788/ELDependenci/blob/4be10baf95b63d53ef8d3f98b7fc5f9ce8309b52/ELDependenci-plugin/src/main/java/com/ericlam/mc/eld/ELDependenci.java#L195)。
{% endhint %}

## 內置標識參數解析列表一覽

### String

* message -> 把所有參數連成訊息串
* sha-256 -> 把參數轉換為 sha256 hex 文字
