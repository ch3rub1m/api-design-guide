# 错误
本章概述了Google APIs的错误模型，并且提供一个规范来指导开发者如何正确地生成和处理错误。

Google APIs 使用一个简单的和协议无关的错误模型，这允许我们可以在不同的APIs，例如gRPC和HTTP，以及不同的错误上下文（例如，异步，归并，和工作流错误）里面提供一致性的体验。

## 错误模型
Google APIs的错误模型是由 [google.rpc.Status](https://github.com/googleapis/googleapis/blob/master/google/rpc/status.proto) 进行定义的，当API发生错误时一个`Status`的实例会被返回给调用者。以下这个代码片段展示了错误模型的总体设计思路：

```
package google.rpc;

message Status {
	// 一个容易被客户端进行处理的错误码。
	// 实际的错误码在 google.rpc.Code 里面定义
	int32 code = 1;

	// 面向开发人员的可读性高的英文错误信息
	// 这个错误信息应该同时说明错误的原因以及提供一个可操作的处理错误的方法
	string message = 2;

	// 额外的错误信息，这些错误信息可以被客户端代码用来处理这个错误，
	// 例如告诉客户端隔多长时间再次尝试或者提供一个帮助链接
	repeated google.protobuf.Any details = 3;
}
```
和大多数Google APIs的设计一样， 我们的错误处理同样遵循面向资源的设计原则。我们会使用一个较小的标准错误集合来对应一个较大的资源集合。举个例子，对于`NOT_FOUND`错误，我们不会给不同的资源定义不同的错误，而是统一给客户端返回一个带有标准`google.rpc.Code.NOT_FOUND`错误码以及指明特定的资源没有被找到的错误。因为我们的错误状态集合比较小，这样可以减少API文档的复杂性，同时客户端的代码也可以使用更加惯用的映射来减少其逻辑的复杂性而且还可以从服务器返回的错误里面获得一些可操作的信息。

## 错误码
Google APIs 一定要使用定义在[google.rpc.Code](https://github.com/googleapis/googleapis/blob/master/google/rpc/code.proto)的标准错误码。单独的APIs应该避免定义多余的错误码，因为开发者基本上不可能为一大堆错误码实现相关的处理逻辑。举个例子，如果平均每个API请求要处理的错误码个数是3个的话就会导致大多数的代码逻辑只为处理错误而存在的，这会使开发者的体验很不好。

## 错误信息
错误信息是用来帮助用户快速并容易地理解和解决API错误的。通常情况下，在书写错误信息是要遵循以下规范：
*   不要假使用户是你API的专家。用户可能是客户端开发者，运营人员，IT工作人员或者是APP的终端用户。
*   不要假使用户知道你API的所有实现或者熟悉错误的上下文（例如日志分析）。
*   如果有可能，错误信息应该被构建为可以让技术人员（不一定是你API的开发者）对你的错误做出响应并且可以改正这个错误。
*   保持错误信息的简要性。如果有需要，为有疑惑的读者提供一个可以提问的，提供反馈的，或者获取更多不可以在错误信息中展示的信息的链接。如果做不到应该为用户提供一个可以展开的错误详情字段。

## 错误详情
Google APIs在[google/rpc/error_details.proto](https://github.com/googleapis/googleapis/blob/master/google/rpc/error_details.proto)里面定义了一套错误详情的标准错误负载。他们涵盖了API错误的最通用的需求，例如配额错误（quota failure）和无效参数错误（invalid parameters）。和错误码一样，错误详情应该尽可能使用这些标准错误负载。

额外的错误详情类型只有在可以帮助应用程序处理错误的时候才应该被引入。如果错误信息依赖于其内容并且只可以被人类来处理的话，应该让开发者手动处理该错误而不是引入新的错误详情类型。

以下是一些错误详情负载的例子：
*   RetryInfo 用于描述什么时候客户端可以重试失败请求，可能会与错误码Code.UNAVAILABLE或者Code.ABORTED一起被返回。
*   QuotaFailure用于描述配额检查失败的原因，可能会与错误码Code.RESOURCE_UNEXHAUSTED一起被返回。
*   BadRequest用于描述客户端请求的违规情况，可能会与错误码Code.INVALID_ARGUMENT一起被返回。

## HTTP 映射
虽然proto3消息有原生的JSON编码，但是Google API平台针对Google的REST APIs使用了不同的错误结构来做到向后兼容。

结构：
```
// Google REST APIs的错误结构。备注：这个结构不是用在其他wire protocol的
message Error {
	// 该消息具有与google.rpc.Status相同的语义。它有一个额外的字段 status字段，用于向后兼容Google API 的客户端库
	message Status {
		// 这个对应着`google.rpc.Status.code`。
		int32 code = 1;
		// 这个对应着`google.rpc.Status.message`。
		string message = 2;
		// 这个是`google.rpc.Status.code`的枚举版本.
		google.rpc.Code status = 4;
		// 这个对应着`google.rpc.Status.details`。
		repeated google.protobuf.Any details = 5;
	}
	// 实际的错误负载。嵌套的消息结构是用来与Google API客户端进行向后兼容的。它同时也让错误对于开发者更加可读。
	Status error = 1;
}
```

例子：
```
{
  "error": {
    "code": 401,
    "message": "Request had invalid credentials.",
    "status": "UNAUTHENTICATED",
    "details": [{
      "@type": "type.googleapis.com/google.rpc.RetryInfo",
      ...
    }]
  }
}
```

## RPC 映射
不同RPC协议映射错误模型的方法不一样。对于[gRPC](http://grpc.io/)来说，错误模型被受支持的语言的生成代码以及运行时的库原生支持。你可以在gRPC的API文档里面获取更多的信息（例如，查看gRPC Java的[io.grpc.Status](https://github.com/grpc/grpc-java/blob/master/core/src/main/java/io/grpc/Status.java)）。

## 客户端库映射
Google客户端库可能会根据语言的不同来对错误进行不同的表示从而和该语言的既定习惯保持一致。例如，[google-cloud-go](https://github.com/GoogleCloudPlatform/google-cloud-go)库将会返回一个实现了和[google.rpc.Status](https://github.com/googleapis/googleapis/blob/master/google/rpc/status.proto)一样接口的错误，但是[google-cloud-java](https://github.com/GoogleCloudPlatform/google-cloud-java)将会抛出一个异常。

## 错误本地化处理
[google.rpc.Status](https://github.com/googleapis/googleapis/blob/master/google/rpc/status.proto)的message字段是面向程序员的，所以必须用英语来编写。

如果需要面向用户的错误信息，请使用 [google.rpc.localizedMessage](https://github.com/googleapis/googleapis/blob/master/google/rpc/error_details.proto#L195:42)作为你的详情字段值。虽然 [google.rpc.LocalizedMessage](https://github.com/googleapis/googleapis/blob/master/google/rpc/error_details.proto)可以被本地化，但是请确保 [google.rpc.Status](https://github.com/googleapis/googleapis/blob/master/google/rpc/status.proto)是用英语写的。

默认情况下，API服务应该使用经过身份认证的用户locale或者HTTP的头部字段Accept-Language来确定本地化的语言。

## 错误处理
以下是包含所有定义在 [google.rpc.Code](https://github.com/googleapis/googleapis/blob/master/google/rpc/code.proto)的错误码和他们出错的简短原因的一个列表，每个列表项包括错误的HTTP code以及RPC code和简短错误描述。你可以通过查看返回的错误码对应的错误描述来相应地对你的请求进行更改。

HTTP | RPC                 | Description
-----|---------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------
200  | OK                  | 没有错误。
400  | INVALID_ARGUMENT.   | 客户端发送的数据包含非法参数。查看错误消息和错误详情来获取更多的信息。
400  | FAILED_PRECONDITION | 现在的系统状态不可以执行当前的请求，例如删除一个非空的目录。
400  | OUT_OF_RANGE        | 客户端指定了一个非法的范围。
401  | UNAUTHENTICATED     | 因为缺失的，失效的或者过期的OAuth令牌，请求未能通过身份认证。
403  | PERMISSION_DENIED   | 客户端没有足够的权限。这可能是因为OAuth令牌没有正确的作用域，或者客户端没有权限，或者是API对客户端代码禁用了。
404  | NOT_FOUND           | 特定的资源没有被找到或者请求因为某些未被公开的原因拒绝（例如白名单）。
409  | ABORTED             | 并发冲突，如读 - 修改 - 写冲突。
409  | ALREADY_EXISTS      | 客户端尝试新建的资源已经存在了。
429  | RESOURCE_EXHAUSTED  | 资源配额不足或达不到速率限制。 客户应该查找google.rpc.QuotaFailure错误详细信息以获取更多信息。
499  | CANCELLED           | 请求被客户端取消了。
500  | DATA_LOSS           | 不可恢复的数据丢失或数据损坏。 客户端应该向用户报告错误。
500  | UNKNOWN             | 未知的服务端出错，通常是由于服务器出现bug了。
500  | INTERNAL            | 服务器内部错误。通常是由于服务器出现bug了。
501  | NOT_IMPLEMENTED     | API方法没有被服务器实现。
503  | UNAVAILABLE         | 服务不可用。通常是由于服务器宕机了。
504  | DEALINE_EXCEED      | 请求超过了截止日期。 只有当调用者设置的截止日期比方法的默认截止日期更短（服务器没能够在截止日期之前处理完请求）并且请求没有在截止日期内完成时，才会发生这种情况。
## 错误重试
客户端应该在发生500和503的时候进行指数退避重试。除非另有说明，否则最小延迟应为1秒。对于429错误，客户端的重试时间最小为30s。对于另外的错误，重试可能不适用-首先确保你的请求具有幂等性，然后查看错误信息来寻求指导。

## 错误传播
如果你的API服务依赖于其他的服务，你不应该盲目地将那些服务的错误传播到你的客户。当翻译错误时，我们建议做以下事情：
*   隐藏技术实现细节和机密信息。
*   调整应该对错误负责的一方。 例如，从另一个服务接收到INVALID_ARGUMENT错误的服务器应该向其调用者返回INTERNAL错误。

## 生成错误
如果你是服务端开发者，你应该生成那些可以提供足够信息来帮助客户端开发者理解并且处理问题的错误。与此同时，您必须意识到用户数据的安全性和私密性，并避免在错误消息和错误详情中公开用户的敏感信息，因为错误通常会被记录下来并可能被其他人访问到。例如，像“客户IP不在128.0.0.0/8白名单中”这样的错误信息会暴露服务端的相关策略信息，用户不应该可以访问到这个信息。

为了生成正确的错误，首先你需要熟悉[google.rpc.Code](https://github.com/googleapis/googleapis/blob/master/google/rpc/code.proto)从而可以再不同的错误环境下选择最适合的错误码。客户端代码可能同时检查多个错误条件，并且返回第一个。

以下列表展示了每个错误码以及一些好的错误信息的例子，列表每一项包括HTTP code, RPC code以及错误信息的例子。

HTTP | RPC                 | Example Error Message
-----|---------------------|----------------------------------------------------------------
400  | INVALID_ARGUMENT    | Request field x.y.z is xxx, expected one of `[yyy, zzz]`.
400  | FAILED_PRECONDITION | Resource xxx is a non-empty directory, so it cannot be deleted.
400  | OUT_OF_RANGE        | Parameter 'age' is out of range `[0, 125]`.
401  | UNAUTHENTICATED     | Invalid authentication credentials.
403  | PERMISSION_DENIED   | Permission 'xxx' denied on file 'yyy'.
404  | NOT_FOUND           | Resource 'xxx' not found.
409  | ABORTED             | Couldn’t acquire lock on resource ‘xxx’.
409  | ALREADY_EXISTS      | Resource 'xxx' already exists.
429  | RESOURCE_EXHAUSTED  | Quota limit 'xxx' exceeded.
499  | CANCELLED           | Request cancelled by the client.
500  | DATA_LOSS           | 查看备注。
500  | UNKNOWN             | 查看备注。
500  | INTERNAL            | 查看备注。
501  | NOT_IMPLEMENTED     | Method 'xxx' not implemented.
503  | UNAVAILABLE         | 查看备注。
504  | DEADLINE_EXCEEDED   | 查看备注。

备注：因为客户端不可以修复服务端错误，所以生成一些额外的错误详情作用不大。为了避免在某些错误条件下泄漏敏感信息，我们推荐不要生成任何的错误信息，只生成google.rpc.DebugInfo的错误详情。DebugInfo通常被设计为用于服务端的日志，而且一定不可以发送给客户端。

Google.rpc包里面定义了一系列标准的错误负载，相对于自定义错误负载，他们更加被推荐使用。 以下列表列出了每个错误代码及其匹配的标准错误负载（如果适用的话）。

HTTP | RPC                 | Recommended Error Detail
-----|---------------------|-------------------------------
400  | INVALID_ARGUMENT    | google.rpc.BadRequest
400  | FAILED_PRECONDITION | google.rpc.PreconditionFailure
400  | OUT_OF_RANGE        | google.rpc.BadRequest
401  | UNAUTHENTICATED     |
403  | PERMISSION_DENIED   |
404  | NOT_FOUND           | google.rpc.ResourceInfo
409  | ABORTED             |
409  | ALREADY_EXISTS      | google.rpc.ResourceInfo
429  | RESOURCE_EXHAUSTED  | google.rpc.QuotaFailure
499  | CANCELLED           |
500  | DATA_LOSS           |
500  | UNKNOWN             |
500  | INTERNAL            |
501  | NOT_IMPLEMENTED     |
503  | UNAVAILABLE         |
504  | DEADLINE_EXCEEDED   |
