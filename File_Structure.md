## 文件架构
  gRPC APIs 应在.proto文件里用proto3 IDL定义。

  文件架构应该放在较高层级上，且要定义地比低层级和不重要项目更重要。proto文件中每部分须按照以下顺序排列：
  * 版权和许可证，如果需要。
  * syntax、package、option、import语句要以此顺序排列。
  * 用于解释文件其他部分的API概述文档。
  * API原型service的定义要按照重要性降序排列。
  * RPC请求和响应message的定义。该顺序和其相关方法定义的顺序一致。每个请求消息都会先于其响应消息。
  * 资源message定义。在子资源定义之前，其父资源必须先完成定义。

  若一个proto文件包含了整个API接口，则它的命名要与API一致：
  * Library
    * Library.proto
  * Calendar
    * Calendar.proto
  若.proto文件较大则可能会被分成多个文件。那么其中的服务、资源信息、请求/响应消息就要分别放到需要的文件里去。

  我们建议将一个服务和它相关的请求、响应放在一个文件里。可以这样命名<enclosed service name>.proto。只包含资源的proto文件可命名为resources.proto。

### proto文件的名称
  proto文件的名称最好使用小写单词，单词之间使用下划线分隔，且必须使用.proto的扩展名。例如：service_controller.proto。

### proto选项
  为了能通过不同API生成一样的的客户端库，API开发者必须在.proto文件中使用一致的proto选项。遵循本份指导定义的API须使用以下文件级proto选项：
  ```
  syntax = "proto3";

// 包的名称应该以公司名称开头，以主版本号结尾
package google.abc.xyz.v1;

// 该选项规定了使用C#时的命名空间。一般proto包默认使用PascalCased版本，当包名的每个部分都由单个单词组成时能正常使用。
// 例如，一个包名`google.shopping.pets.v1`在使用C#命名空间时成为`Google.Shopping.Pets.V1`
// 然而，若包名某个部分由多个单词组成，就需要指定该选项，以免第一个字母变成大写。例如，谷歌宠物中心的API可能有一个包叫做`google.shopping.petstore.v1`，其在C#命名空间下会变成`Google.Shopping.Petstore.V1`。所以需要指定该选项使包名变成`Google.Shopping.PetStore.V1`
//
// 更多关于C#或.NET的信息，可在【框架设计指南】中查看(https://msdn.microsoft.com/en-us/library/ms229043)。
//
// 有个特殊情况：当使用了缩略词时，需要将缩略词全部大写。例如，`IOStream` 和 `OSVersion`, 而不能写成 `IoStream` 和 `OsVersion`。在API里要谨慎使用这些词，因为proto并不知道哪个词是缩写、哪个不是。比如命名空间中有`OSLogin` ，要是在同名信息里生成一个叫做`OsDetails`的类，就会导致冲突。若你能确保消息或字段名中不会出现缩略词，那么使用常规的PascalCase是安全的。
// 
// 对于预发布版本，Alpha/Beta应该开头大写，举个例子应该是"V1Beta1"，而不是"V1beta1"。
option csharp_namespace = "Google.Abc.Xyz.V1";

// 该选项让proto编译器能在包名内生成Java代码（见下），而非在外部类内。通过减少一层名称嵌套能简化开发，并且可以和大多数不支持外部类的编程语言保持一致。
option java_multiple_files = true;

// Java外部包类名应该是大写驼峰式。该类只用于xxxxxx，所以开发者无需直接操作它。！
option java_outer_classname = "XyzProto";

// Java包名称必须是带有合适前缀的proto包名。
option java_package = "com.google.abc.xyz.v1";

// 包可以给Objective-C生成合适的前缀。最少要3字长，全部大写，并且包名写成缩写。使用简短但独一无二的名称确保不会和将来可能出现的名称冲突。'GPB' 保留给protocol buffer去实现本身。
option objc_class_prefix = "GABCX";

// 该选项指定了PHP中使用的命名空间。proto包默认使用PascalCased版本，若包名由单个单词组成时能正常使用。
// 例如，包名为"google.shopping.pets.v1"，在PHP命名空间下会变成"Google\\Shopping\\Pets\\V1"。
// 然而，若包名某个部分由多个单词组成，就需要指定该选项，以免第一个字母变成大写。例如，谷歌宠物中心的API可能有一个包叫做`google.shopping.petstore.v1`，其在PHP命名空间下会变成`Google\\Shopping\\Petstore\\V1`。所以需要指定该选项使包名变成`Google\\Shopping\\PetStore\\V1`
// 对于预发布版本，Alpha/Beta不该是以大写开头，举个例子应该是"V1beta1"，而不是"V1Beta1"。注意这和C#命名空间中的预发布版本不同。
//
option php_namespace = "Google\\Abc\\Xyz\\V1";
  ```
