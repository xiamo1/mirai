# Mirai - Messages

## 目录
- [消息系统](#消息系统)
- [消息类型](#消息类型)
- [消息元素](#消息元素)
  - [转义规则](#转义规则)
  - [消息链的 mirai 码](#消息链的-mirai-码)
  - [由 `CodableMessage` 取得 mirai 码字符串](#由-codablemessage-取得-mirai-码字符串)
  - [由 mirai 码字符串取得 `MessageChain` 实例](#由-mirai-码字符串取得-messagechain-实例)
- [消息链](#消息链)
  - [发送消息](#发送消息)
  - [构造消息链](#构造消息链)
  - [元素唯一性](#元素唯一性)
  - [获取消息链中的消息元素](#获取消息链中的消息元素)
- [Mirai 码](#mirai-码)

## 消息系统

在 Contacts 章节提到，要发送消息，使用 `Contact.sendMessage(Message)`。`Message` 架构如下图所示。

[![](https://mermaid.ink/img/eyJjb2RlIjoiY2xhc3NEaWFncmFtXG5cbmNsYXNzIE1lc3NhZ2VDaGFpblxuTWVzc2FnZUNoYWluIDogTGlzdH5TaW5nbGVNZXNzYWdlflxuXG5NZXNzYWdlPHwtLU1lc3NhZ2VDaGFpblxuTWVzc2FnZTx8LS1TaW5nbGVNZXNzYWdlXG5cbk1lc3NhZ2VDaGFpbiBvLS0gU2luZ2xlTWVzc2FnZVxuXG5TaW5nbGVNZXNzYWdlPHwtLU1lc3NhZ2VDb250ZW50XG5TaW5nbGVNZXNzYWdlPHwtLU1lc3NhZ2VNZXRhZGF0YVxuXG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)](https://mermaid-js.github.io/mermaid-live-editor/#/edit/eyJjb2RlIjoiY2xhc3NEaWFncmFtXG5cbmNsYXNzIE1lc3NhZ2VDaGFpblxuTWVzc2FnZUNoYWluIDogTGlzdH5TaW5nbGVNZXNzYWdlflxuXG5NZXNzYWdlPHwtLU1lc3NhZ2VDaGFpblxuTWVzc2FnZTx8LS1TaW5nbGVNZXNzYWdlXG5cbk1lc3NhZ2VDaGFpbiBvLS0gU2luZ2xlTWVzc2FnZVxuXG5TaW5nbGVNZXNzYWdlPHwtLU1lc3NhZ2VDb250ZW50XG5TaW5nbGVNZXNzYWdlPHwtLU1lc3NhZ2VNZXRhZGF0YVxuXG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)

`SingleMessage` 表示单个消息元素，`MessageChain`（消息链） 是 `List<SingleMessage>`。主动发送的消息和从服务器接收消息都是 `MessageChain`。


> 回到 [目录](#目录)

## 消息类型

*单个消息元素（`SingleMessage`）*分为 *消息内容（`MessageContent`）* 和 *消息元数据（`MessageMetadata`）*。

实践中，消息内容和消息元数据会混合存在于消息链中。

> 回到 [目录](#目录)

## 消息元素

Mirai 支持富文本消息。

消息拥有三种转换到字符串的表示方式。

| 方法                      | 解释                                                                                          |
|:-------------------------|:---------------------------------------------------------------------------------------------|
| `serializeToMiraiCode()` | 对应的 Mirai 码. 消息的一种序列化方式，格式为 `[mirai:TYPE:PROP]`，其中 `TYPE` 为消息类型, `PROP` 为属性 |
| `contentToSting()`       | QQ 对话框中以纯文本方式会显示的消息内容。无法用纯文字表示的消息会丢失信息，如任何图片都是 `[图片]`             |
| `toString()`             | Java 对象的 `toString()`，会尽可能包含多的信息用于调试作用，**行为可能不确定**                           |

各类型消息元素及其 `contentToString()` 如下表格所示。

[`PlainText`]: ../mirai-core-api/src/commonMain/kotlin/message/data/PlainText.kt
[`At`]: ../mirai-core-api/src/commonMain/kotlin/message/data/At.kt
[`AtAll`]: ../mirai-core-api/src/commonMain/kotlin/message/data/AtAll.kt
[`Face`]: ../mirai-core-api/src/commonMain/kotlin/message/data/Face.kt
[`PokeMessage`]: ../mirai-core-api/src/commonMain/kotlin/message/data/PokeMessage.kt
[`VipFace`]: ../mirai-core-api/src/commonMain/kotlin/message/data/VipFace.kt
[`Image`]: ../mirai-core-api/src/commonMain/kotlin/message/data/Image.kt
[`FlashImage`]: ../mirai-core-api/src/commonMain/kotlin/message/data/FlashImage.kt
[`MarketFace`]: ../mirai-core-api/src/commonMain/kotlin/message/data/MarketFace.kt
[`MusicShare`]: ../mirai-core-api/src/commonMain/kotlin/message/data/MusicShare.kt

[`MessageSource`]: ../mirai-core-api/src/commonMain/kotlin/message/data/MessageSource.kt
[`QuoteReply`]: ../mirai-core-api/src/commonMain/kotlin/message/data/QuoteReply.kt
[`LightApp`]: ../mirai-core-api/src/commonMain/kotlin/message/data/RichMessage.kt
[`SimpleServiceMessage`]: ../mirai-core-api/src/commonMain/kotlin/message/data/RichMessage.kt
[`Voice`]: ../mirai-core-api/src/commonMain/kotlin/message/data/Voice.kt
[`ForwardMessage`]: ../mirai-core-api/src/commonMain/kotlin/message/data/ForwardMessage.kt

|          消息类型          | 属性                                        | 解释                 | `contentToString()`     |
|:------------------------:|:--------------------------------------------|:--------------------|:------------------------|
|      [`PlainText`]       | `content: String`                           | 纯文本               | `$content`              |
|        [`Image`]         | `imageId: String`                           | 自定义图片            | `[图片]`                 |
|          [`At`]          | `target: Int`                               | 提及某人              | `@$target`              |
|        [`AtAll`]         |                                             | 提及全体成员           | `@全体成员`              |
|         [`Face`]         | `id: Int`                                   | 原生表情              | `[表情对应的中文名]`       |
|      [`FlashImage`]      | `image: Image`                              | 闪照                 | `[闪照]`                 |
|     [`PokeMessage`]      | `name: String`, `pokeType: Int` , `id: Int` | 戳一戳消息（消息非动作） | `[戳一戳]`               |
|       [`VipFace`]        | `kind: VipFace.Kind`, `count: Int`          | VIP 表情             | `[${kind.name}]x$count` |
|       [`LightApp`]       | `content: String`                           | 小程序               | `$content`              |
|        [`Voice`]         | `content: String`                           | 语音                 | `$content`              |
|      [`MarketFace`]      | `id: Int, name: String`                     | 商城表情              | `[表情对应的中文名]`       |
|    [`MessageSource`]     | ...                                         | 消息来源元数据         | *空字符串*                |
|      [`QuoteReply`]      | `source: MessageSource`                     | 引用回复              | *空字符串*               |
|    [`ForwardMessage`]    | ...                                         | 合并转发              | `[转发消息]`             |
| [`SimpleServiceMessage`] | `serviceId: Int, content: String`           | （不稳定）服务消息      | `$content`              |
|      [`MusicShare`]      | ...                                          | (自 2.1) 音乐分享     | `[分享]曲名`              |

**请打开相关消息类型的源码查看用法。**

> 回到 [目录](#目录)

## 消息链

[`MessageChain`]: ../mirai-core-api/src/commonMain/kotlin/message/data/MessageChain.kt
[`SingleMessage`]: ../mirai-core-api/src/commonMain/kotlin/message/data/Message.kt
[`CodableMessage`]: ../mirai-core-api/src/commonMain/kotlin/message/code/CodableMessage.kt

前文已经介绍消息链，这里简略介绍消息链的使用。详细的使用请查看源码内注释。

### 发送消息

在 [Contacts 章节](Contacts.md) 提到，要发送消息使用 `Contact.sendMessage`。`Contact.sendMessage` 的定义是：
```kotlin
 suspend fun sendMessage(message: Message): MessageReceipt<Contact>
```

要发送简单的单元素消息，使用：
```kotlin
contact.sendMessage(PlainText("Hello!"))
```
```java
contact.sendMessage(new PlainText("Hello!"));
```

要发送多元素消息，可将消息使用 `plus` 操作连接：
```kotlin
contact.sendMessage(PlainText("你要的图片是") + Image("/f8f1ab55-bf8e-4236-b55e-955848d7069f")) // 一个纯文本加一个图片
```
```java
contact.sendMessage(new PlainText("你要的图片是：").plus(Image.fromId("/f8f1ab55-bf8e-4236-b55e-955848d7069f"))); // 一个纯文本加一个图片
```

### 构造消息链

更复杂的消息则需要构造为消息链。

#### 在 Kotlin 构造消息链

| 定义                                                     |
|:--------------------------------------------------------|
| `fun Iterable<Messaged>.toMessageChain(): MessageChain` |
| `fun Sequence<Messaged>.toMessageChain(): MessageChain` |
| `fun Array<Message>.toMessageChain(): MessageChain`     |
| `fun Message.toMessageChain(): MessageChain`            |
| `fun messageChainOf(vararg Message): MessageChain`      |
| `fun Message.plus(tail: Message): MessageChain`         |

可以使用如上表格所示的方法构造，或使用 DSL builder。
```
class MessageChainBuilder : MutableList<SingleMessage>, Appendable {
    operator fun Message.unaryPlus()
    operator fun String.unaryPlus()
    fun add(vararg messages: Message)
}
```

```kotlin
val chain = buildMessageChain {
    +PlainText("a")
    +AtAll
    +Image("/f8f1ab55-bf8e-4236-b55e-955848d7069f")
    add(At(123456))
}

// chain 结果是包含 PlainText, AtAll, Image, At 的 MessageChain
```

#### 在 Java 构造消息链

| 定义                                                                  |
|:---------------------------------------------------------------------|
| `public static MessageChain newChain(Iterable<Message> iterable)`    |
| `public static MessageChain newChain(Message iterable...)`           |
| `public static MessageChain newChain(Iterator<Message> iterable...)` |

方法都位于 `net.mamoe.mirai.message.data.MessageUtils`。

使用 `MessageChainBuilder`:
```java
MessageChain chain = new MessageChainBuilder()
    .append(new PlainText("string"))
    .append("string") // 会被构造成 PlainText 再添加, 相当于上一行
    .append(AtAll.INSTANCE)
    .append(Image.fromId("/f8f1ab55-bf8e-4236-b55e-955848d7069f"))
    .build();
```

### 元素唯一性

[`MessageKey`]: ../mirai-core-api/src/commonMain/kotlin/message/data/MessageKey.kt
[`ConstrainSingle`]: ../mirai-core-api/src/commonMain/kotlin/message/data/Message.kt#L273-L287
[`HummerMessage`]: ../mirai-core-api/src/commonMain/kotlin/message/data/HummerMessage.kt

部分元素只能单一存在于消息链中。这样的元素实现接口 [`ConstrainSingle`]。

唯一的元素例如 *消息元数据* [`MessageSource`]，在连接时，新的（右侧）元素会替换旧的（左侧）元素。如：
```kotlin
val source1: MessageSource
val source2: MessageSource

val chain: MessageChain = source1 + source2
// 结果 chain 只包含一个元素，即右侧的 source2。
```

元素唯一性的识别基于 [`MessageKey`]。[`MessageKey`] 拥有多态机制。元素替换时会替换。如 [`HummerMessage`] 的继承关系
```
              MessageContent
                    ↑
              HummerMessage
                    ↑
       +------------+-------------+------------+
       |            |             |            |
 PokeMessage     VipFace      FlashImage      ...
```

当连接一个 [`VipFace`] 到一个 [`MessageChain`] 时，由于 [`VipFace`] 最上层为 `MessageContent`，消息链中第一个 `MessageContent` 会被（保留顺序地）替换为 [`VipFace`]，其他所有 `MessageContent` 都会被删除。
```kotlin
val chain = messageChainOf(quoteReply, plainText, at, atAll) // quoteReply 是 MessageMetadata, 其他三个都是 MessageContent
val result = chain + VipFace(VipFace.AiXin, 1) // VipFace 是 ConstrainSingle，最上层键为 MessageContent，因此替换所有的 MessageContent
// 结果为 [quoteReply, VipFace]
```


### 获取消息链中的消息元素

#### A. 筛选 List
[`MessageChain`] 继承接口 `List<SingleMessage>`。
```kotlin
val image: Image? = chain.filterIsInstance<Image>().firstOrNull()
```
```java
Image image = (Image) chain.stream().filter(Image.class::isInstance).findFirst().orElse(null);
```

在 Kotlin 要获取第一个指定类型实例还可以使用快捷扩展。
```kotlin
val image: Image? = chain.findIsInstance<Image>()
val image: Image = chain.firstIsInstance<Image>() // 不存在时 NoSuchElementException
```

#### B. 获取唯一消息
如果要获取 `ConstrainSingle` 的消息元素，可以快速通过键获得。

```kotlin
val quote: QuoteReply? = chain[QuoteReply] // 类似 Map.get
val quote: QuoteReply = chain.getOrFail(QuoteReply) // 不存在时 NoSuchElementException
```
```java
QuoteReply quote = chain.get(QuoteReply.Key);
```

> 这是因为 `MessageKey` 一般都以消息元素的 `companion object` 实现

#### C. 使用属性委托

可在 Kotlin 使用属性委托。这样的方法与上述方法在性能上等价。

```kotlin
val image: Image by chain // 不存在时 NoSuchElementException
val image: Image? by chain.orNull()
val image: Image? by chain.orElse { /* 返回一个 Image */ }
```

### 序列化

消息可以序列化为 JSON 字符串，使用 `MessageChain.serializeToJsonString` 和 `MessageChain.deserializeFromJsonString`。

## Mirai 码

实现了接口 `CodableMessage` 的消息类型支持 mirai 码表示。

### 转义规则

mirai 码内的属性字符串会被转义。

|   原字符    | 转义结果字符 |
|:----------:|:---------:|
|    `[`     |   `\[`    |
|    `]`     |   `\]`    |
|    `:`     |   `\:`    |
|    `,`     |   `\,`    |
|    `\`     |   `\\`    |
| *换行符 \n* |    `\n`    |
| *换行符 \r* |    `\r`    |

### 消息链的 mirai 码

消息链 [`MessageChain`] 是多个 [`SingleMessage`] 的集合。[`MessageChain`] 也实现 [`CodableMessage`]。在转换为 mirai 码时所有 [`CodableMessage`] 直接相连：
```
val chain = messageChainOf(PlainText("plain"), At(123), AtAll)

chain.serializeToMiraiCode() // "plain[mirai:at:123][mirai:atall]"
```

### 由 `CodableMessage` 取得 mirai 码字符串

通过 `CodableMessage.serializeToMiraiCode()`。

```
val at = At(123)

at.serializeToMiraiCode() // 结果为 `[mirai:at:123]`
```

|          消息类型          | `serializeToMiraiCode()`                         |
|:------------------------:|:-------------------------------------------------|
|      [`PlainText`]       | `$content`                                       |
|        [`Image`]         | `[mirai:image:$imageId]`                         |
|          [`At`]          | `[mirai:at:$target]`                             |
|        [`AtAll`]         | `[mirai:atall]`                                  |
|         [`Face`]         | `[mirai:face:id]`                                |
|      [`FlashImage`]      | `[mirai:flash:${image.imageId}]`                 |
|     [`PokeMessage`]      | `[mirai:poke:$name,$pokeType,$id]`               |
|       [`VipFace`]        | `[mirai:vipface:${kind.id},${kind.name},$count]` |
|       [`LightApp`]       | `[mirai:app:$content]`                           |
| [`SimpleServiceMessage`] | `[mirai:service:$serviceId,$content]`            |

### 由 mirai 码字符串取得 `MessageChain` 实例

```kotlin
val chain = "[mirai:atall]".deserializeMiraiCode()
```
```java
MessageChain chain = MiraiCode.deserializeFromMiraiCode("[mirai:atall]");
```


---------

到这里，你已经完成了 Mirai 所有文档的阅读。现在你已经熟悉了 Mirai，并可以开始使用了。

你可以首先构造 Bot，登录，然后从监听事件起开始创建你的机器人，或从 Bot 获取到指定群主动发送消息。在使用中遇到问题可以参考 Mirai 源码内注释，该注释会包含更多实践上的帮助。

如果你仍然对 Mirai 架构有不明确的地方，欢迎在 [#848](https://github.com/mamoe/mirai/discussions/848) 提出建议，或者直接在 PR 提交你的修改。

> 回到 [目录](#目录)
>
> [回到 Mirai 文档索引](README.md#mirai-core-api-文档)
