## 文件架构
  gRPC APIs 应在`.proto`文件里用proto3 IDL定义。

  文件架构应该将高层级和更重要的定义放在其他项目之前。proto文件中每部分须按照以下顺序排列：
  * 版权和许可证，如果需要。
  * `syntax`、`package`、`option`、`import`语句,他们要以此顺序排列。
  * 让读者准备好阅读文件剩余内容的API概述文档。
  * API proto文件的`service`定义要按照其重要性降序排列。
  * RPC请求和响应`消息`的定义顺序应该和相关方法定义的顺序一致。若有请求消息，则其要优先于它的响应消息被定义。
  * 资源`消息`定义。在子资源定义之前，其父资源必须先完成定义。

  若一个proto文件包含了整个API的定义，则它的命名要与API一致：
  
|   API    |     Proto     |
|----------|:-------------:|
| Library  | Library.proto |
| Calendar | Calendar.proto|

  若.proto文件较大则可能会被分成多个文件。其中的服务、资源信息、请求/响应消息应该按需放至不同的文档里去。

  我们建议将同一个服务的请求和响应放在同一个文件里。文件可命名为`<enclosed service name>.proto`。只包含资源的proto文件可命名为`resources.proto`。

### proto文件的名称
  proto文件的名称最好使用`小写单词下划线互相分隔`的方式定义，且必须使用.proto的扩展名。例如：service_controller.proto。

### proto选项
  为了能在不同API中生成一致的的客户端库，API开发者必须在`.proto`文件中使用一致的proto选项。遵循本份指导定义的API须使用以下proto选项：
```
syntax = "proto3";

// 包的名称应该以公司名称开头，以主版本号结尾
package google.abc.xyz.v1;

// 该选项规定了使用在C#代码里的命名空间。一般proto包默认使用PascalCased版本，当包名的每个部分都由单个单词组成时PascalCased版本能正常使用。
// 例如，一个包名`google.shopping.pets.v1`在使用C#命名空间时会转换为`Google.Shopping.Pets.V1`
// 如果包名某个部分由多个单词组成，就需要指定该选项，以免第一个字母变成大写。例如，谷歌宠物中心的API若有包叫做`google.shopping.petstore.v1`，在C#命名空间下包名是`Google.Shopping.Petstore.V1`。所以需要指定该选项将包名改为`Google.Shopping.PetStore.V1`
//
// 更多关于C#或.NET的信息，可在[框架设计指南](https://msdn.microsoft.com/en-us/library/ms229043)中查看。
//
// 有个特殊情况：当使用了缩略词时，需要将缩略词全部大写。例如，`IOStream` 和 `OSVersion`, 而不能写成 `IoStream` 和 `OsVersion`。在API里要谨慎使用这些词，因为proto并不知道哪个词是缩写、哪个不是。比如命名空间中有`OSLogin` ，要是在同名信息里生成一个叫做`OsDetails`的类，就会导致不一致。若能确保消息或字段名中不会出现缩略词，那么使用常规的PascalCase是安全的。
// 
// 对于预发布版本，Alpha/Beta应该以大写字母开头，举个例子应该是"V1Beta1"，而不是"V1beta1"。
option csharp_namespace = "Google.Abc.Xyz.V1";

// 该选项让proto编译器能在包名而不是外部类中生成Java代码（见下）。通过减少一层名称嵌套能简化开发，并且可以和大多数不支持外部类的编程语言保持一致。
option java_multiple_files = true;

// Java外部包类名应该是大写驼峰式。该类只用于保存proto的描述符，所以开发者无需直接使用它。
option java_outer_classname = "XyzProto";

// Java包名称必须是带有合适前缀的proto包名。
option java_package = "com.google.abc.xyz.v1";

// 包生成的合理的Objective-C前缀。最短3个字符、全部大写并且惯例是使用包名的缩写。使用简短但独一无二的名称确保不会和将来可能出现的名称冲突。'GPB'作为保留字代表protocol buffer本身。
option objc_class_prefix = "GABCX";

// 该选项指定使用PHP代码里的命名空间。proto包默认使用PascalCased版本，若包名由单个单词组成时能正常使用。
// 例如，包名为"google.shopping.pets.v1"，在PHP命名空间下会变成"Google\\Shopping\\Pets\\V1"。
// 然而，若包名某个部分由多个单词组成，就需要指定该选项，以免第一个字母变成大写。例如，谷歌宠物中心的API可能有一个包叫做`google.shopping.petstore.v1`，其在PHP命名空间下会变成`Google\\Shopping\\Petstore\\V1`。所以需要指定该选项使包名变成`Google\\Shopping\\PetStore\\V1`
// 对于预发布版本，Alpha/Beta不该是以大写开头，举个例子应该是"V1beta1"，而不是"V1Beta1"。注意这和C#命名空间中的预发布版本不同。
//
option php_namespace = "Google\\Abc\\Xyz\\V1";
```
