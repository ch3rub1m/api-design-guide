# 文档
本章是关于如何为API添加内部文档的指南。大部分API也包含了概述、教程和高级参考文档，但这些不在本指南的讨论范围内。想获取更多有关API、资源、方法命名的信息，请查看[命名约定](Naming_Conventions.md)一章。

## 注释格式
`.proto`文件中使用Protocol Buffers常用的`//`来添加注释。
```
// Creates a shelf in the library, and returns the new Shelf.
rpc CreateShelf(CreateShelfRequest) returns (Shelf) {
  option (google.api.http) = { post: "/v1/shelves" body: "shelf" };
}
```
## 服务配置中添加注释
你可以在YAML服务配置文件中添加内部文档，以代替在`.proto`文件中添加文档注释。如果在内部文档和`.proto`文件中对同一个元素进行了描述，则以内部文档为准。
```
documentation:
  summary: Gets and lists social activities
  overview: A simple example service that lets you get and list possible social activities
  rules:
  - selector: google.social.Social.GetActivity
    description: Gets a social activity. If the activity does not exist, returns Code.NOT_FOUND.
...
```
如果不同的服务使用了同一个`.proto`文件, 但你希望能为每个服务提供特定的文档，则可用这种添加内部文档的方法。YAML文档注释法允许你在API描述里面添加更加详细的`overview`部分。但是，我们一般更推荐在`.proto`文件中添加文档注释。

和`.proto`注释一样，你可以使用Markdown为YAML文件注释提供额外的样式。

## API描述
API描述是一个以动词开头的短语，描述了此API能做什么。 在`.proto`文件中，API描述作为注释添加到对应`service`上，如下所示：

```
// Manages books and shelves in a simple digital library.
service LibraryService {
...
}
```

API描述的其他示例：
*   Shares updates, photos, videos, and more with your friends around the world.
*   Accesses a cloud-hosted machine learning service that makes it easy to build smart apps that respond to streams of data.

## 资源描述
资源描述是一个局部句子，说明该资源指的是什么。如果要添加更多信息，请使用额外的句子。在`.proto`文件中，资源描述作为注释添加到对应消息类型上，如下所示：
```
// A book resource in the Library API.
message Book {
...
}
```

资源描述的其他示例：
*   A task on the user's to-do list. Each task has a unique priority.
*   An event on the user's calendar.

## 字段和参数描述
描述字段或参数的名词短语，一些例子：

*   The number of topics in this series.
*   The accuracy of the latitude and longitude coordinates, in meters. Must be non-negative.
*   Flag governing whether attachment URL values are returned for submission resources in this series. The default value for series.insert is true.
*   The container for voting information. Present only when voting information is recorded.
*   Not currently used or deprecated.

字段和参数描述要遵循以下规则：

*   必须描述清楚边界条件（即要清楚地说明什么是有效的、什么是无效的。谨记开发者是一定会误用你的服务的，且不会阅读底层代码来弄清楚那些不明了的信息。）
*   必须指定所有的默认值或默认行为（即在未提供值的时候服务器会做什么）。
*   当字段或参数是字符串时（例如名称或路径），需要说明其遵循的语法、允许的字符和需要的编码格式。例如：
    *   1-255 characters in the set `[A-a0-9]`
    *   A valid URL path string starting with / that follows the RFC 2332 conventions. Max length is 500 characters.
*   如果可以的话，给字段或参数提供一个示例值。
*   如果该字段是`Required`、`Input only`或`Output only`的，都必须在字段描述的开头说明。默认所有字段和参数都是可选的。例如：

```
message Table {
  // Required. The resource name of the table.
  string name = 1;
  // Input only. Whether to dry run the table creation.
  bool dryrun = 2;
  // Output only. The timestamp when the table was created. Assigned by
  // the server.
  Timestamp create_time = 3;
  // The display name of the table.
  string display_name = 4;
}
```

## 方法描述
方法描述是一个描述方法的作用和方法操作对象的句子。通常以第三人称[现在时的动词](https://developers.google.com/internal/style/reference-verbs)开头（即以's'结尾的动词）。如果需要添加更多详情，则使用其他句子。一些示例：

*   Lists calendar events for the authenticated user.
*   Updates a calendar event with the data included in the request.
*   Deletes a location record from the authenticated user's location history.
*   Creates or updates a location record in the authenticated user's location history using the data included in the request. If a location resource already exists with the same timestamp value, the data provided overwrites the existing data.

## 描述的检查清单
你要确保每个描述都是简短但完整的，同时也要可以被那些不太了解你的API的用户看懂。在大多数情况下，不能只是重申显而易见的信息，例如`series.insert`方法的描述不能只是"Inserts a series"。虽然你的命名应该已经表明它是做什么的了，但大多数读者之所以会阅读你的描述是因为他们想要获取更多的详细信息。如果你不确定描述中要写什么，试着回答以下相关的问题：

*   它是什么？
*   成功时会做什么？失败时会做什么？怎样原因会导致失败？
*   它是幂等的吗？
*   单位是什么？（例如：米、度、像素。）
*   接受值的范围？范围是开区间还是闭区间？
*   副作用是什么？
*   如何使用？
*   常见错误是什么？
*   是否会一直存在？（例如： "Container for voting information. Present only when voting information is recorded."）
*   是否有默认设置？

## 约定
本部分列出了文本描述和文档的一些使用约定。例如，用'ID'表示标识符（全部大写），而不是'Id'或'id'；用'JSON'，而不是'Json'或'json'。所有字段/参数使用`code font`格式。字符串要使用引号。

*   ID
*   JSON
*   RPC
*   REST
*   `property_name`或`string_literal`
*   `true`/`false`

## 要求级别

使用这些术语：must, must not, required, shall, shall not, should, should not, recommended, may, 和 optional来表达预期或要求的级别。

以上词语的含义在[RFC 2119](https://www.ietf.org/rfc/rfc2119.txt)中有定义。您可能想要将RFC摘要里的声明写入你的API文档。

在确定哪个术语符合你的需求的同时也要为开发人员提供灵活性。在API中如果其他选项在技术上也是可行的，就不要使用像must这样绝对的语气。

## 语言风格
和[命名约定](Naming_Conventions.md)一样，在写注释时建议使用简单一致的词汇和风格，让非英语母语的读者容易理解。因此要避免使用行话、俚语、复杂的隐喻、流行文化或其他不容易理解的内容。使用友好专业的风格，并尽可能保持注释的简洁。请记住，大部分读者只想知道如何使用API，而不是阅读你的文档！
