# Protocol Buffers v3
这章将会讨论开发者如何使用Protocol Buffers进行API设计。为了简化用户体验和提高运行效率，gRPC应该使用Protocol Buffers 3 （proto 3）来进行API定义。

[Protocol Buffers](https://github.com/google/protobuf)是一门语言中立和平台中立的用来定义数据结构模式以及编程接口的简单接口定义语言（IDL）。它同时支持二进制和文本格式，而且可以在不同的平台上使用很多不同的有线协议（wire protocol）。

Proto3是Protocol Buffers的最新版本，相对于版本2，其有以下的一些改变：

* 原始字段（primitive fields）不再支持`hasField`。未设置值的原始字段将会被赋予一个由语言定义的默认值。
	* 不过仍然可以判断一个字段是否存在。可以使用编译器生成的hasField方法进行测试，又或者是将该字段的值和空值（null）或由实现定义的标记值（sentinel value）进行比较。
* 不再支持用户自定义字段的默认值。
* 枚举定义必须从0开始。
* 不再支持 required 字段。
* 不再支持扩展（extensions），请使用 google.protobuf.Any。
	* `google/protobuf/descriptor.proto` 是个特殊例外，因为要顾及向后兼容性和运行时兼容性。
* 删除了组语法（group） 。

删除这些特性是为了让API的设计变得更加简单，更加稳定，以及性能变得更好。例如，我们将消息记录为日志消息时常常要将消息的一些敏感字段删除，如果这些字段是必须的（required），这个需求就不可能做到了。

查看[Protocol Buffers](https://developers.google.com/protocol-buffers/)获取更多的信息。
		