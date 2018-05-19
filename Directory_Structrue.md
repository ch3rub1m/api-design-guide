## 目录结构
  API服务通常用`.proto`文件来定义API接口，用`.yaml`文件来配置API服务。

  每个API在API仓库中必须有一个API目录，该目录包括它的定义文件和构建脚本。

  API目录应该有如下的标准层级：
*   API目录
    *   仓库基础
        *   `BUILD` - 构建文件
        *   `METADATA` - 构建元数据文件
        *   `OWNERS` - API目录所有者
    *   配置文件
        *   `{service}.yaml` - 基准服务配置文件，它是`google.api.Service`的proto消息的YAML表示。
        *   `prod.yaml` - 生产环境差异化服务配置文件。
        *   `staging.yaml` - 仿真环境差异化服务配置文件。
        *   `test.yaml` - 测试环境差异化配置文件。
        *   `local.yaml` - 本地环境差异化服务配置文件。
    *   文档文件
        *   `README.md` - 主要的说明书。它应该包含产品概述、技术描述等等。
        *   `doc/*` - 技术文档文件。它们应该使用Markdown格式。
    *   接口定义
        *   `v[0-9]*/*` - 每个目录包含了一个API的major版本，大体上是proto文件和构建脚本。
        *   `{subapi}/v[0-9]*/* `- 每个`{subapi}`目录包含了一个子API的接口定义。每个子API可能有它们自己独立的major版本。
        *   `type/*` - proto文件，这些文件包含了不同API、相同API的不同版本，或是API和服务具体实现之间共享的类型。`type/*`下定义的类型一旦发布了就不能有破坏兼容性的改动。
