## 文件结构
gRPC APIs应使用proto3 IDL定义在`.proto`文件中。

文件结构应该将更高层级和更重要的定义放在其它项目之前。在每个proto文件中，每个适用的部分应该遵循以下顺序：
*   版权和许可证提示，如果需要。
*   Proto的`syntax`、`package`、`option`和`import`语句，顺序分先后。
*   API概述文档以供读者为阅读文件剩余内容做好准备。
*   API proto的`service`定义，按其重要性降序排列。
*   用于RPC请求和响应的`message`定义，遵照与对应方法一致的顺序。若有请求消息，则其必须在它对应的响应消息之前。
*   资源`message`定义。父资源的定义必须在子资源之前。

若一个proto文件包含了完整的API接口，则它应该以该API来命名：

API      | Proto
---------|---------------
Library  | library.proto
Calendar | calendar.proto

大型.proto文件可以被分成多个文件。服务、资源信息、请求/响应消息应该按需被放到不同的文件里。

我们建议将同一个服务的请求和响应放在同一个文件里。可以考虑将该文件命名为`<enclosed service name>.proto`。对只包含资源的proto文件，可以考虑将其命名为`resources.proto`。

### Proto文件名
proto文件名应该使用lower_case_underscore_separated_names并且必须使用`.proto`的扩展名。举个例子：service_controller.proto。

### Proto可选项
为了能跨APIs生成一致的的客户端库，API开发者必须在他们的`.proto`文件中使用一致的proto可选项。遵循本份指南的API定义必须使用以下文件级别的proto可选项：

```
syntax = "proto3";

// 包的名称应该以公司名称开头，以主版本号结尾。
package google.abc.xyz.v1;

// 该选项规定了将C#代码里使用的命名空间。一般proto包默认使用PascalCased版本，当包名的每个部分都由单个单词组成时PascalCased版本能正常使用。
// 例如，一个名为`google.shopping.pets.v1`的包会使用名为`Google.Shopping.Pets.V1`的C#命名空间。
// 但是，如果包名某个部分由多个单词组成，就需要指定该选项，以免只有首字母大写。例如，谷歌宠物中心的API若有包叫做`google.shopping.petstore.v1`则意味着在C#中命名空间将是`Google.Shopping.Petstore.V1`。相反的，应该使用该选项将它正确大写化为`Google.Shopping.PetStore.V1`。
//
// 更多关于C#或.NET字母大写规则的细节，请参阅[框架设计指南](https://msdn.microsoft.com/en-us/library/ms229043)。
//
// 关于字母大写有个特殊情况：当使用了缩略词时，需要将缩略词全部大写。例如，`IOStream` 和 `OSVersion`, 而不能写成 `IoStream` 和 `OsVersion`。在API里要谨慎使用这些词，因为proto并不知道哪个词是缩写、哪个不是。比如命名空间中有`OSLogin` ，要是在同名信息里生成一个叫做`OsDetails`的类，就会导致不一致。若能确保消息或字段名中不会出现缩略词，那么使用常规的PascalCase是安全的。
//
// 对于预发布版本，Alpha/Beta应该以大写字母开头，例如应该是"V1Beta1"，而不是"V1beta1"。
option csharp_namespace = "Google.Abc.Xyz.V1";

// 该选项让proto编译器能在包名而不是外部类中生成Java代码（见下）。通过减少一层名称嵌套能简化开发体验，并且可以和大多数不支持外部类的编程语言保持一致。
option java_multiple_files = true;

// Java外部类的名称应该是大写驼峰式。该类只用于保存proto的描述符，所以开发者无需直接使用它。
option java_outer_classname = "XyzProto";

// Java包名称必须是带有合适前缀的proto包名。
option java_package = "com.google.abc.xyz.v1";

// 包生成的合理的Objective-C前缀。它应该最少有3个字符、全部大写并且约定使用包名的缩写。使用简短但足够独特的名称以确保不会和将来可能出现的名称冲突。'GPB'作为保留字代表protocol buffer本身。
option objc_class_prefix = "GABCX";

// 该选项指定将会在PHP代码里使用的命名空间。proto包默认使用PascalCased版本，若包名由单个单词组成时能正常使用。
// 例如，包名为"google.shopping.pets.v1"，在PHP命名空间下会变成"Google\\Shopping\\Pets\\V1"。
// 然而，若包名某个部分由多个单词组成，就需要指定该选项，以免只有首字母大写。例如，谷歌宠物中心的API可能有一个包叫做`google.shopping.petstore.v1`，其在PHP命名空间下会变成`Google\\Shopping\\Petstore\\V1`。相反的，应该使用该选项将它正确大写化为`Google\\Shopping\\PetStore\\V1`。
//
// 对于预发布版本，Alpha/Beta不该是以大写开头，例如应该是"V1beta1"，而不是"V1Beta1"。注意这和C#命名空间中的预发布版本不同。
//
option php_namespace = "Google\\Abc\\Xyz\\V1";
```
