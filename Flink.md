# Flink
## 1. Flink 简介
### Flink 是什么？
* Apache Flink 是一个**框架**和**分布式**处理引擎，用于对**无界和有界数据流**进行**状态**计算



### 为什么选择 Flink？
* 传统的数据架构只能处理固定的数据，无法不断添加新数据计算
* 目标：
    - 低延迟
    - 高吞吐
    - 结果的准确性和良好的容错性



### 哪些行业需要处理流数据
* 对实时性要求比较高的
* 电商和市场营销
    - 数据报表、广告投放、业务流程需要
* 物联网（IOT）
    - 传感器实时数据采集和显示、实时报警，交通运输业
* 电信业
    - 基站流量调配
* 银行和金融业
    - 实时结算和通知推送，实时检测异常行为



### Flink 的主要特点
* 事件驱动
* 基于流的世界观
    - 在 Flink 的世界观中，一切都是由流组成的，离线数据是有界的流，实时数据是无界的流
* 分层 API
    - 越顶层越抽象，表达含义越简明，使用越灵活
    - 越底层越具体，表达能力越丰富，使用越灵活
* 其他特点
    - 支持事件时间（event-time）和处理时间（processing-time）语义
    - 精确一次（exactly-once）的状态一致性保证
    - 低延迟，每秒处理数百万个事件，毫秒级延迟
    - 与众多常用存储系统的连接
    - 高可用，动态扩展，实现 7 * 24 小时全天候运行




### Flink vs Spark Streaming
* 数据模型
    - spark 采用 RDD 模型，spark streaming 的 DStream 实际上也就是一组组小批数据 RDD 的集合
    - flink 基本数据模型是数据流，以及事件（Event）序列
* 运行时架构
    - spark 是批计算，将 DAG 划分为不同的 stage，一个完成后才可以计算下一个
    - flink 是标准的流执行模式，一个事件在一个节点处理完后可以直接发往下一个节点进行处理



## 2. Flink 运行架构
### Flink 运行时的组件
#### 任务提交流程（宏观）
![任务提交流程（宏观）](https://cdn.jsdelivr.net/gh/qtxsnwwb/image-hosting@master/20210522/任务提交流程（宏观）.3jcp3ce2j3w0.png)

#### 1. 作业管理器（JobManager）
* 控制一个应用程序执行的主进程，也就是说，每个应用程序都会被一个不同的 JobManager 所控制执行
* JobManager 会先接收到要执行的应用程序，这个应用程序会包括：作业图（JobGraph）、逻辑数据流图（logical dataflow graph）和打包了所有的类、库和其他资源的 JAR 包
* JobManager 会把 JobGraph 转换成一个物理层面的数据流图，这个图被叫做“执行图”（ExecutionGraph），包含了所有可以并发执行的任务
* JobManager 会向资源管理器（ResourceManager）请求执行任务必要的资源，也就是任务管理器（TaskManager）上的插槽（slot）。一旦获取到了足够的资源，就会将执行图分发到真正运行它们的 TaskManager 上。而在运行过程中，JobManager 会负责所有需要中央协调的操作，比如说检查点（checkpoints）的协调



#### 2. 任务管理器（TaskManager）
* Flink 中的工作进程。通常在 Flink 中会有多个 TaskManager 运行，每一个 TaskManager 都包含了一定数量的插槽（slots）。插槽的数量限制了 TaskManager 能够执行的任务数量
* 启动之后，TaskManager 会向资源管理器注册它的插槽；收到资源管理器的指令后，TaskManager 就会将一个或者多个插槽提供给 JobManager 调用。JobManager 就可以向插槽分配任务（tasks）来执行了
* 在执行过程中，一个 TaskManager 可以跟其他运行同一应用程序的 TaskManager 交换数据



#### 3. 资源管理器（ResourceManager）
* 主要负责管理任务管理器（TaskManager）的插槽（slot），TaskManager 插槽是 Flink 中定义的处理资源单元
* Flink 为不同的环境和资源管理工具提供了不同资源管理器，比如 YARN、Mesos、K8s，以及 standalone 部署
* 当 JobManager 申请插槽资源时，ResourceManager 会将有空闲插槽的 TaskManager 分配给 JobManager。如果 ResourceManager 没有足够的插槽来满足 JobManager 的请求，它还可以向资源提供平台发起会话，以提供启动 TaskManager 进程的容器



#### 4. 分发器（Dispatcher）
* 可以跨作业运行，它为应用提交提供了 REST 接口
* 当一个应用被提交执行时，分发器就会启动并将应用移交给一个 JobManager
* Dispatcher 也会启动一个 Web UI，用来方便地展示和监控作业执行的信息
* Dispatcher 在架构中可能并不是必需的，这取决于应用提交运行的方式



### 任务提交流程（YARN）
![任务提交流程（YARN）](https://cdn.jsdelivr.net/gh/qtxsnwwb/image-hosting@master/20210522/任务提交流程（YARN）.7khan19egc80.png)

### 任务调度原理
![任务调度原理](https://cdn.jsdelivr.net/gh/qtxsnwwb/image-hosting@master/20210522/任务调度原理.46fxhgzyz4o0.png)




### 并行度
* 一个特定算子的子任务（subtask）的个数被称之为其并行度（parallelism）
* 一般情况下，一个 stream 的并行度，可以认为就是其所有算子中最大的并行度



### TaskManager 和 Slots
* Flink 中每一个 TaskManager 都是一个 JVM 进程，它可能会在独立的线程上执行一个或多个子任务
* **为了控制一个 TaskManager 能接收多少个 task**，TaskManager 通过 task slot 来进行控制（一个 TaskManager 至少有一个 slot）
* 默认情况下，Flink 允许子任务共享 slot，即使它们是不同任务的子任务。这样的结果是，一个 slot 可以保存作业的整个管道
* Task Slot 是静态的概念，是指 TaskManager 具有的并发执行能力



### 程序与数据流（DataFlow）
* 所有的 Flink 程序都是由三部分组成的：Source、Transformation 和 Sink
* Source 负责读取数据源，Transformation 利用各种算子进行处理加工，Sink 负责输出
* 在运行时，Flink 上运行的程序会被映射成“逻辑数据流”（dataflows），它包含了这三部分
* 每一个 dataflow 以一个或多个 sources 开始，以一个或多个 sinks 结束。dataflow 类似于任意的有向无环图（DAG）
* 在大部分情况下，程序中的转换运算（transformations）跟 dataflow 中的算子（operator）是一一对应的关系



### 执行图（ExecutionGraph）
* Flink 中的执行图可以分成四层：StreamGraph -> JobGraph -> ExecutionGraph -> 物理执行图
* StreamGraph：是根据用户通过 Stream API 编写的代码生成的最初的图。用来表示程序的拓扑结构
* JobGraph：StreamGraph 经过优化后生成了 JobGraph，提交给 JobManager 的数据结构。主要的优化为，将多个符合条件的节点 chain 在一起作为一个节点
* ExecutionGraph：JobManager 根据 JobGraph 生成 ExecutionGraph。ExecutionGraph 是 JobGraph 的并行化版本，是调度层最核心的数据结构
* 物理执行图：JobManager 根据 ExecutionGraph 对 Job 进行调度后，在各个 TaskManager 上部署 Task 后形成的“图”，并不是一个具体的数据结构




### 数据传输形式
* 一个程序中，不同的算子可能具有不同的并行度
* 算子之间传输数据的形式可以是 one-to-one（forwarding）的模式也可以是 redistributing 的模式，具体是哪一种形式，取决于算子的种类
* One-to-one：stream 维护着分区以及元素的顺序（比如 source 和 map 之间）。这意味着 map 算子的子任务看到的元素的个数以及顺序跟 source 算子的子任务生产的元素的个数、顺序相同。map、filter、flatMap 等算子都是 one-to-one 的对应关系
* Redistributing：stream 的分区会发生改变。每一个算子的子任务依据所选择的 transformation 发送数据到不同的目标任务。例如，keyBy 基于 hashCode 重分区、而 broadcast 和 rebalance 会随机重新分区，这些算子都会引起 redistribute 过程，而 redistribute 过程就类似于 Spark 中的 shuffle 过程



### 任务链（Operator Chains）
* Flink 采用了一种称为任务链的优化技术，可以在特定条件下减少本地通信的开销。为了满足任务链的要求，必须将两个或多个算子设为相同的并行度，并通过本地转发（local forward）的方式进行连接
* **相同并行度**的 **one-to-one** 操作，Flink 这样相连的算子链接在一起形成一个 task，原来的算子成为里面的 subtask
* 并行度相同、并且是 ont-to-one 操作，两个条件缺一不可