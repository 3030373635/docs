## 核心概念

- Flow：流（Flow）是 Prefect 中的工作流单元，由多个任务组成。每个 Flow 可以定义任务之间的依赖关系，并控制任务的执行顺序。
- Task：任务（Task）是 Flow 中的基本单元，代表数据处理管道中的一个步骤。任务可以是任何计算操作，如读取数据、数据转换、模型训练等。
- State：状态（State）表示任务或 Flow 的当前执行状态，例如 “Running”、”Success”、”Failed” 等。状态管理有助于监控和调试工作流。
- Storage：存储（Storage）定义了 Flow 的存储位置，可以是本地文件系统、GitHub、S3 等。存储配置决定了 Flow 如何被加载和执行。
- Run Config：运行配置（Run Config）定义了 Flow 的执行环境，例如在本地运行、在 Kubernetes 集群中运行等。
- Agent：代理（Agent）是负责监听 Prefect 服务器并执行 Flow 的进程。代理可以在本地或云端运行，支持多种环境。
- 调度器（Scheduler）：调度器是负责实际执行任务的组件。Prefect 提供了多种调度器选项，包括本地调度器、分布式调度器、Kubernetes 调度器等。可以根据项目的需求选择合适的调度器。