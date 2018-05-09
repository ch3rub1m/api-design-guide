## 文档
  本章是为API添加内部文档的指南。大部分API也包含了概述、教程和高级参考文档， 但这些不在本指南的讨论范围内。想获取更多有关API、资源、方法命名的信息，请查看[命名约定](https://git.garena.com/shopee/space/api-design-guide/blob/36c7960d6005269ea8d6de1ce0d1db632a589ac2/Naming_Conventions.md)

### 注释格式
  `.proto`文件中使用Protocol Buffers常用的`//`添加注释。
  ```
  // Creates a shelf in the library, and returns the new Shelf.
rpc CreateShelf(CreateShelfRequest) returns (Shelf) {
  option (google.api.http) = { post: "/v1/shelves" body: "shelf" };
}
  ```
### 服务配置中添加注释
  你可以在YAML服务配置文件中添加内部文档，作为给`.proto`文件添加文档注释的替代方法。如果在内部文档和`.proto`文件中记录了同一个元素，则以内部文档为准。
```
documentation:
  summary: Gets and lists social activities
  overview: A simple example service that lets you get and list possible social activities
  rules:
  - selector: google.social.Social.GetActivity
    description: Gets a social activity. If the activity does not exist, returns Code.NOT_FOUND.
...
```
  如果不同的服务使用了同一个`.proto`文件但希望能提供特定服务的文档，则可用添加内部文档的方法。你可以在YAML中可以添加`overview`来更详细地描述API。但一般更推荐将文档注释添加到`.proto`文件里。

  和`.proto`注释一样，可以在YAML文件的注释中使用Markdown来提供其他格式。

### API描述
  API描述是一个以动词开头的短语，描述了可以此API能做什么。 在`.proto`文件中，API描述是作为注释添加到相关`service`上去的，例如：
  ```
  // Manages books and shelves in a simple digital library.
    service LibraryService {
    ...
    }
  ```
 API描述的其他示例：
 * Shares updates, photos, videos, and more with your friends around the world.
 * Accesses a cloud-hosted machine learning service that makes it easy to build smart apps that respond to streams of data.

### 资源描述
  资源描述是一个句子，描述了该资源代表了什么。如果要添加更多信息，使用其他句子。在`.proto`文件中，资源猫描述作为注释添加到对应消息上，例如：
  ```
  // A book resource in the Library API.
    message Book {
    ...
    }
  ```
  资源描述的其他示例：
  * A task on the user's to-do list. Each task has a unique priority.
  * An event on the user's calendar.

### 字段和参数描述
  描述一个字段或参数定义的名词短语，一些示例：
  * The number of topics in this series.
  * The accuracy of the latitude and longitude coordinates, in meters. Must be non-negative.
  * Flag governing whether attachment URL values are returned for submission resources in this series. The default value for series.insert is true.
  * The container for voting information. Present only when voting information is recorded.
  * Not currently used or deprecated.

  字段和参数描述要遵循以下规则：
  * 必须描述清楚边界条件（即清楚什么是有效的、什么是无效的。开发者可能会误用服务，且不会阅读底层代码来弄清楚）
  * 必须指定所有的默认值或默认行为（即服务器在未提供值的时候会做什么）。
  * 当字段或参数是字符串时（例如名称或路径），需要描述语法、允许的字符和需要的编码格式。例如：
    * 1-255 characters in the set [A-a0-9]
    * A valid URL path string starting with / that follows the RFC 2332 conventions. Max length is 500 characters.
  * 如果可以的话，给字段或参数提供一个示例值。
  * 当字段是必须的、仅输入、仅输出时，必须在字段描述的开头说明。默认所有字段和参数都是可选的。例如：
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

### 方法描述
  方法描述是一个描述方法的作用和方法操作对象的句子。通常以第三人称[现在时的动词](https://developers.google.com/internal/style/reference-verbs)开头（即以's'结尾的动词）。如果需要添加更多详情，则使用其他句子。一些示例：
  * Lists calendar events for the authenticated user.
  * Updates a calendar event with the data included in the request.
  * Deletes a location record from the authenticated user's location history.
  * Creates or updates a location record in the authenticated user's location history using the data included in the request. If a location resource already exists with the same timestamp value, the data provided overwrites the existing data.

### 所有描述
  确保每个描述简洁完整且容易理解。不能仅仅只重申显而易见的信息，例如`series.insert`方法的描述不能只是'Inserts a series'。虽然你的命名信息很丰富，但读者阅读你的描述时想要获取更多详细信息。如果你不确定描述中要写什么，试着回答以下的问题：
  * 它是什么？
  * 成功或失败后会做什么？什么会导致失败？怎样导致失败的？
  * 它是幂等的吗？
  * 单位是什么？（例如：米、度、像素）
  * 接受值的范围？
  * 副作用是什么？
  * 如何使用？
  * 常见错误是什么？
  * 是否一直存在？（例如： "Container for voting information. Present only when voting information is recorded."）
  * 是否有默认设置？

### 惯例
  本节列出了文本描述和文档的一些使用惯例。例如，用'ID'表示标识符（全部大写），而不是'Id'或'id'；用'JSON'，而不是'Json'或'json'。所有字段/参数使用`code font`格式。字符串要使用引号。
  * ID
  * JSON
  * RPC
  * REST
  * `property_name`或`string_literal`
  * `true`/`false`

### 要求的级别
  使用术语：must, must not, required, shall, shall not, should, should not, recommended, may, 和 optional来表达预期或要求的级别。

  以上词语的含义在[RFC 2119](https://www.ietf.org/rfc/rfc2119.txt)中有定义。您可能会将RFC摘要里的声明写入你的API文档。

  确定哪个术语符合你的要求，同时为开发人员提供灵活性。在API中其他技术选项工作时，不要使用绝对术语例如must。

### 语言风格
  和命名约定一样，在写注释时建议使用简洁一致的词语和风格，让母语非英语的读者容易理解。因此要避免行话、俚语、复杂的隐喻、流行文化或其他不容易理解的内容。使用友好专业的风格，并尽可能保持注释的简洁。请记住，大部分读者只想知道如何使用API，而不是阅读你的文档！
