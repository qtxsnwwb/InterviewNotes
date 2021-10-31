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




### 事件驱动型应用
* 事件驱动型应用是一类具有**状态**的应用，它从一个或多个事件流提取数据，并根据到来的事件触发计算、状态更新或其他外部动作
* 事件驱动型应用是在计算存储分离的传统应用基础上进化而来。在传统架构中，应用需要读写远程事务性数据库
* 事件驱动型应用是基于状态化流处理来完成。在该设计中，数据和计算不会分离，应用只需访问本地（内存或磁盘）即可获取数据。系统容错性的实现依赖于定期向远程持久化存储写入checkpoint
* 优势
    - 事件驱动型应用无须查询远程数据库，本地数据访问使得它具有更高的吞吐和更低的延迟。而由于定期向远程持久化存储的 checkpoint 工作可以异步、增量式完成，因此对于正常事件处理的影响甚微。
    - 事件驱动型应用的优势不仅限于本地数据访问。传统分层架构下，通常多个应用会共享同一个数据库，因而任何对数据库自身的更改（如由应用更新或服务扩容导致数据布局发生改变）都需要谨慎协调。而事件驱动型应用，由于只需考虑自身数据，因此在更改数据表示或服务扩容时所需的协调工作将大大减少。




### 数据分析应用
* 数据分析任务需要从原始数据中提取有价值的信息和指标。传统的分析方式通常是利用批查询，或将事件记录下来并给予此有限数据集构建应用来完成。为了得到最新数据的分析结果，必须先将它们加入分析数据集并重新执行查询或运行应用，随后将结果写入存储系统或生成报告
* 借助一些先进的流处理引擎，还可以实时地进行数据分析。和传统模式下读取有限数据集不同，流式查询或应用汇接入实时事件流，并随着事件消费持续产生和更新结果。这些结果数据可能会写入外部数据库系统或以内部状态的形式维护。仪表展示应用可以相应地从外部数据库读取数据或直接查询应用的内部状态
* **流式分析应用的优势**
    - 和批量分析相比，由于流式分析省掉了周期性的数据导入和查询过程，因此从事件中获取指标的延迟更低。不仅如此，批量查询必须处理那些由定期导入和输入有界性导致的人工数据边界，而流式查询则无需考虑该问题
    - 流式分析会简化应用抽象。批量查询的流水线通常由多个独立部件组成，需要周期性地调度提取数据和执行查询。如此复杂的流水线操作起来并不容易，一旦某个组件出错将会影响流水线的后续步骤。而流式分析应用整体运行在 Flink 之类的高端流处理系统之上，涵盖了从数据接入到连续结果计算的所有步骤，因此可以依赖底层引擎提供的故障恢复机制




### 数据管道
* 提取-转换-加载（ETL）是一种在存储系统之间进行数据转换和迁移的常用方法。ETL 作业通常会周期性地触发，将数据从事务型数据库拷贝到分析型数据库或数据仓库
* 数据管道和 ETL 作业的用途相似，都可以转换、丰富数据，并将其从某个存储系统移动到另一个。但数据管道是以持续流模式运行，而非周期性触发。因此它支持从一个不断生成数据的源头读取记录，并将它们以低延迟移动到终点。例如：数据管道可以用来监控文件系统目录中的新文件，并将其数据写入事件日志；另一个应用可能会将事件流物化到数据库或增量构建和优化查询索引
* 优势
    - 和周期性 ETL 作业相比，持续数据管道可以明显降低将数据移动到目的端的延迟。此外，由于它能够持续消费和发送数据，因此用途更广，支持用例更多




## 2. 流处理基础
### 数据并行和任务并行
* 数据并行：将输入数据分组，让同一操作的多个任务并行执行在不同数据子集上。能够将计算负载分配到多个节点上从而允许处理大规模的数据
* 任务并行：让不同算子的任务（基于相同或不同的数据）并行计算。可以更好地利用集群的计算资源




### 数据交换策略
* 转发策略：在发送端任务和接收端任务之间一对一地进行数据传输。如果两端任务运行在同一物理机器上，该交换策略可以避免网络通信
* 广播策略：把每个数据项发往下游算子的全部并行任务。该策略会把数据复制多份且涉及网络通信，因此代价十分昂贵
* 基于键值的策略：根据某一键值属性对数据分区，并保证键值相同的数据项会交由同一任务处理
* 随机策略：将数据均匀分配至算子的所有任务，以实现计算任务的负载均衡




### 延迟和吞吐
* 延迟：处理一个事件所需的时间
* 吞吐：用来衡量系统处理能力（处理速率）的指标，告诉我们系统每单位时间可以处理多少事件
* 降低延迟实际上可以提高吞吐。系统执行操作越快，相同时间内执行的操作数目就会越多。通过并行处理多条数据流，可以在处理更多事件的同时降低延迟




### 窗口
* 转换操作和滚动聚合每次处理一个事件来产生输出并（可能）更新状态。然而，有些操作必须收集并缓冲记录才能计算结果，例如流式 Join 或像是求中位数的整体聚合。为了在在无限数据流上高效地执行这些操作，必须对操作所维持的数据量加以限制。除此之外，窗口操作还支持在数据流上完成一些具有切实语义价值的查询。
* 窗口操作会持续创建一些称为“桶”的有限事件集合，并允许我们基于这些有限集进行计算。事件通常会根据其事件或其他数据属性分配到不同桶中。为了准确定义窗口算子语义，我们需要决定事件如何分配到桶中以及窗口用怎样的频率产生结果
* 窗口类型语义
    - 滚动窗口：将事件分配到长度固定且互不重叠的桶中。在窗口边界通过后，所有事件会发送给计算函数进行处理
    - 滑动窗口：将事件分配到大小固定且允许相互重叠的桶中，意味着每个事件可能会同时属于多个桶
    - 会话窗口：根据会话间隔将事件分为不同的会话，该间隔值定义了会话在关闭前的非活动时间长度




### 时间语义
* 处理时间：当前流处理算子所在机器上的本地时钟时间。基于处理时间的窗口会包含那些恰好在一段时间内到达窗口算子的事件，这里的时间段是按照机器时间测量的
* 事件时间：数据流中事件实际发生的时间，它以附加在数据流中事件的时间戳为依据。这些时间戳通常在事件数据进入流处理管道之前就存在。即使事件有延迟，事件时间窗口也能准确地将事件分配到窗口中，从而反映出真实发生的情况
* 比较：
    - 处理时间窗口能够将延迟降至最低。由于无需考虑迟到或乱序的事件，窗口只需简单地缓冲事件，然后在达到特定事件后立即触发窗口计算即可。因此对于那些更重视处理速度而非准确度的应用，处理事件就能派上用场
    - 如果你需要周期性地实时报告结果而无论其准确定如何，处理时间窗口能够表示数据流自身的真实情况。
    - 虽然处理时间提供了很低的延迟，但它的结果依赖处理速度，具有不确定性
    - 事件时间能保证结果的准确定，并允许你处理延迟甚至无序的事件，但延迟较高
* 水位线
    - 水位线是一个全局进度指标，表示我们确信不会再有延迟事件到来的某个时间点
    - 本质上，水位线提供了一个逻辑时钟，用来通知系统当前的事件时间。当一个算子接收到时间为 T 的水位线，就可以认为不会再收到任何时间戳小于或等于 T 的事件了
    - 水位线无论对于事件时间窗口还是处理乱序事件的算子都很关键。算子一旦收到某个水位线，就相当于接到信号：某个特定时间区间的时间戳已经到齐，可以触发窗口计算或对接收的数据进行排序了




### 状态和一致性模型
* 为了生成结果，函数会在一段时间或基于一定个数的事件来累积状态。有状态算子同时使用传入的事件和内部状态来计算输出
* 一致性模型
    - 至多一次：任务发生故障时既不恢复丢失的状态，也不重放丢失的事件，保证每个事件至多被处理一次。事件可以随意丢弃，没有任何机制来保证结果的正确性
    - 至少一次：所有事件最终都会处理，虽然有些可能会处理多次。如果正确定仅依赖信息的完整度，那重复处理或许可以接受。为了确保至少一次结果语义的正确性，需要想办法从源头或缓冲区中重放事件。持久化事件日志会将所有事件写入永久存储，这样在任务故障时就可以重放它们
    - 精确一次：表示不但没有事件丢失，而且每个事件对于内部状态的更新都只有一次。本质上，精确一次保障意味着应用总会提供正确的结果，就如同故障从未发生过一般。提供精确一次保障是以至少一次保障为前提，因此同样需要数据重放机制
    - 端到端的精确一次：在整个数据处理管道上结果都是正确的。在每个组件都提供自身的保障情况下，整个处理管道上端到端的保障会受制于保障最弱的那个组件




## 3. Flink 运行架构
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

#### Flink 数据传输
* TaskManager 负责将数据从发送任务传输至接收任务。它的网络模块在记录传输前会先将它们收集到缓冲区中。即记录并非逐个发送，而是在缓冲区中以批次形式发送
* 每个 TaskManager 都有一个用于收发数据的网络缓冲池（默认 32 KB）。如果发送端和接收端的任务运行在不同的 TaskManager 进程中，它们就要用到操作系统的**网络栈**进行通信。流式应用需要以**流水线**方式交换数据，因此每对 TaskManager 之间都要维护一个或多个**永久的 TCP 连接**来执行数据交换。对于每一个接收任务，TaskManager 都要提供一个专用的网络缓冲区，用于接收其他任务发来的数据

#### 基于信用值的流量控制
* 通过网络连接逐条发送记录不但低效，还会导致很多额外开销
* 基于信用值的流量控制：接收任务会给发送任务授予一定的信用值，其实就是保留一些用来接收它数据的网络缓冲。一旦发送端收到信用通知，就会在信用值所限定的范围内尽可能多地传输缓冲数据，并会附带上积压量（已经填满准备传输的网络缓冲数目）大小。接收端使用保留的缓冲来处理收到的数据，同时依据各发送端的积压量信息来计算所有相连的发送端在下一轮的信用优先级
* 由于发送端可以在接收端有足够资源时立即传输数据，所以基于信用值的流量控制可以有效降低延迟



### 任务链（Operator Chains）
* Flink 采用了一种称为任务链的优化技术，可以在特定条件下减少本地通信的开销。为了满足任务链的要求，必须将两个或多个算子设为相同的并行度，并通过本地转发（local forward）的方式进行连接
* 条件：
    - 多个算子拥有相同的并行度
    - 通过本地转发通道相连
* **相同并行度**的 **one-to-one** 操作，Flink 这样相连的算子链接在一起形成一个 task，原来的算子成为里面的 subtask
* 并行度相同、并且是 ont-to-one 操作，两个条件缺一不可




### 高可用性设置
#### TaskManager 故障
* 假设一个 Flink 设置包含了 4 个 TaskManager，每个 TaskManager 有 2 个处理槽，那么一个流式应用最多支持以并行度 8 来运行。如果有一个 TaskManager 出现故障，则可用处理槽的数量就降到了 6 个。这时候 JobManager 就会向 ResourceManager 申请更多的处理槽。若无法完成，JobManager 将无法重启应用，直至有足够数量的可用处理槽。

#### JobManager 故障
* 支持在原 JobManager 消失的情况下将作业的管理职责及元数据迁移到另一个 JobManager
* Flink 的高可用模式是基于能够提供分布式协调和共识服务的 Apache ZooKeeper 来完成的，它在 Flink 中主要用于“领导”选举以及持久且高可用的数据存储
* JobManager 会将 JobGraph 以及全部所需的元数据（例如应用的 JAR 文件）写入一个远程持久化存储系统。此外，JobManager 会将存储位置的路径地址写入 ZooKeeper 的数据存储。在应用执行过程中，JobManager会接收每个任务检查点的状态句柄（存储位置）。在检查点即将完成的时候，如果所有任务已经将各自状态成功写入远程存储，JobManager 就会将状态句柄写入远程存储，并将远程位置的路径地址写入 ZooKeeper。因此所有用于 JobManager 故障恢复的数据都在远程存储上面，而 ZooKeeper 持有这些存储位置的路径




### 事件时间处理
#### 时间戳
* Flink 流式应用处理的所有记录都必须包含时间戳。时间戳将记录和特定时间点进行关联，这些时间点通常是记录所对应事件的发生时间。但实际上应用可以自由选择时间戳的含义，只要保证流记录的时间戳会随着数据流的前进大致递增即可。
* 当 Flink 以事件时间模式处理数据流时，会根据记录的时间戳触发时间相关算子的计算
* Flink 内部此阿勇 8 字节的 Long 值对时间戳进行编码，并将它们以元数据（metadata）的形式附加在记录上，内置算子会将这个 Long 值解析为毫秒精度的 Unix 时间戳

#### 水位线
* 水位线（watermark）用于在事件时间应用中推断每个任务当前的事件时间。基于时间的算子会使用这个时间来触发计算并推动进度前进
* Flink 中水位线是利用一些包含 Long 值时间戳的特殊记录来实现的
* 基本属性：
    - 必须单调递增。这是为了确保任务中的事件时间时钟正确前进，不会倒退
    - 和记录的时间戳存在联系。一个时间戳为 T 的水位线表示，接下来所有记录的时间戳一定都大于 T
* 水位线的意义之一在于它允许控制结果的完整性和延迟。如果水位线和记录的时间戳非常接近，那结果的处理延迟就会很低，因为任务无须等待过多记录就可以触发最终计算。但同时结果的完整性可能会受影响，因为可能有部分相关记录被视为迟到记录，没能参与运算。水位线会增加处理延迟，但同时结果的完整性也会有所提升

#### 水位线传播和事件时间
* Flink 内部将水位线实现为特殊的记录，它们通过算子任务进行接收和发送。任务内部的时间服务会维护一些计时器，它们依靠接收到水位线来激活。这些计时器是由任务在时间服务内注册，并在将来的某个时间点执行计算。
* 当任务接收到一个水位线时会执行以下操作：
    1. 基于水位线记录的时间戳更新内部事件时间时钟
    2. 任务的时间服务会找出所有触发时间小于更新后事件时间的计时器。对于每个到期的计时器，调用回调函数，利用它来执行计算或发出记录
    3. 任务根据更新后的事件时间将水位线发出
* 一个任务会为它的每个输入分区都维护一个分区水位线。当收到某个分区传来的水位线后，任务会以接收值和当前值中较大的那个去更新对应分区水位线的值。随后，任务会把事件时间时钟调整为所有分区水位线中最小的那个值。如果事件时间时钟向前推动，任务会先处理因此而触发的所有计时器，之后才会把对应的水位线发往所有连接的输出分区，以实现事件时间到全部下游任务的广播。
![水位线传播](https://cdn.jsdelivr.net/gh/qtxsnwwb/image-hosting@master/20210522/水位线传播.2k6g75wtga40.png)

#### 时间戳分配和水位线生成
* 时间戳和水位线通常都是在数据流刚刚进入流处理应用的时候分配和生成的。由于不同的应用会选择不同的时间戳，而水位线依赖于时间戳和数据流本身的特征，所以应用必须显式地分配时间戳和生成水位线。
* Flink DataStream 应用可以通过三种方式完成该工作：
    - 在数据源完成
    - 周期分配器
    - 定点分配器




### 状态管理
* 函数里所有需要任务去维护并用来计算结果的数据都属于任务的状态，可以把状态想象成任务的业务逻辑所需要访问的本地或实例变量
![状态管理](https://cdn.jsdelivr.net/gh/qtxsnwwb/image-hosting@master/20210522/状态管理.6rzp03s8ar40.png)

#### 算子状态
* 算子状态的作用范围限定为算子任务，由同一并行任务所处理的所有数据都可以访问到相同的状态
* 状态对于同一子任务而言是共享的
* 算子状态不能由相同或不同算子的另一个子任务访问
* 算子状态数据结构：
    - 列表状态（List state）：将状态表示为一组数据的列表
    - 联合列表状态（Union list state）：也将状态表示为数据的列表。它与常规列表状态的区别在于，在发生故障时，或者从保存点（savepoint）启动应用程序时如何恢复
    - 广播状态（Broadcast state）：如果一个算子有多项任务，而它的每项任务状态又都相同，那么这种特殊情况最适合应用广播状态
    ![算子状态](https://cdn.jsdelivr.net/gh/qtxsnwwb/image-hosting@master/20210522/算子状态.a5wp7mixkog.png)

#### 键值分区状态
* 键值分区状态是根据输入数据流中定义的键（key）来维护和访问的
* Flink 为每个 key 维护一个状态实例，并将具有相同键的所有数据，都分区到同一个算子任务中，这个任务会维护和处理这个 key 对应的状态
* 当任务处理一条数据时，它会自动将状态的访问范围限定为当前数据的 key
* 键值分区状态数据结构：
    - 值状态（Value state）：将状态表示为单个的值
    - 列表状态（List state）：将状态表示为一组数据的列表
    - 映射状态（Map state）：将状态表示为一组 Key-Value 对
    - 聚合状态（Reducing state & Aggregating state）：将状态表示为一个用于聚合操作的列表
    ![键值分区状态](https://cdn.jsdelivr.net/gh/qtxsnwwb/image-hosting@master/20210522/键值分区状态.yjincd3gk74.png)

#### 状态后端
* 为了保证快速访问状态，每个并行任务都会把状态维护在本地。状态具体的存储、访问和维护是由一个名为状态后端的**可插拔组件**来决定。
* 状态后端主要负责两件事：**本地状态管理**和**将状态以检查点的形式写入远程存储**
    - 对于本地状态管理，状态后端会存储所有键值分区状态，并保证能将状态访问范围正确地限制在当前键值。Flink 提供的一类状态后端会把键值分区状态作为对象，以内存数据结构的形式存在 JVM 堆中
    - 另一类状态后端会把状态对象序列化后存到 RocksDB 中，RocksDB 负责将它们写到本地磁盘上
    - 前者状态访问更快一些，但会受到内存大小的限制；后者状态访问会慢一些，但允许状态变得很大
* 状态后端负责将任务状态以检查点形式写入远程持久化存储，该远程存储可能是一个分布式文件系统，也可能是某个数据库系统

#### 有状态算子的扩缩容
* 对于有状态算子，该变并行度会复杂很多，因为我们需要把状态重新分组，分配到与之前数量不等的并行任务上
* Flink 对不同类型的状态提供了四种扩缩容模式
    - **带有键值分区状态的算子在扩缩容时会根据新的任务数量对键值重新分区**。为了降低状态在不同任务之间迁移的必要成本，Flink 不会对单独的键值实施再分配，而是会把所有键值分为不同的键值组（key group）。每个键值组都包含了部分键值，Flink 以此为单位把键值分配给不同任务。
    - **带有算子列表状态的算子在扩缩容时会对列表中的条目进行重新分配**。所有并行算子任务的列表条目会被统一收集起来，随后均匀分配到更少或更多的任务之上。如果列表条目的数量小于算子新设置的并行度，部分任务在启动时的状态就可能为空。
    - **带有算子联合列表状态的算子会在扩缩容时把状态列表的全部条目广播到全部任务上**。随后由任务自己决定哪些条目该保留，哪些该丢弃。
    - **带有算子广播状态的算子在扩缩容时会把状态拷贝到全部新任务上**。这样做的原因是广播状态能确保所有任务的状态相同。在缩容的情况下，由于状态经过复制不会丢失，我们可以简单地停掉多出的任务。




### 检查点
#### 一致性检查点
* Flink 的故障恢复机制需要基于应用状态的一致性检查点
* 有状态的流式应用的一致性检查点是在所有任务处理完等量的原始输入后对全部任务状态进行的一个拷贝

#### 从一致性检查点中恢复
* 在流式应用执行过程中，Flink 会周期性地为应用状态生成检查点。一旦发生故障，Flink 会利用最新的检查点将应用状态恢复到某个一致性的点并重启处理进程
* 应用恢复步骤：
    1. 重启整个应用
    2. 利用最新的检查点重置任务状态
    3. 恢复所有任务的处理
* 如果所有算子都将它们全部的状态写入检查点并从中恢复，并且所有输入流的消费位置都能重置到检查点生成那一刻，那么该检查点和恢复机制就能为整个应用的状态提供**精确一次**的一致性保障

#### Flink 检查点算法
* Flink 的检查点是基于 Chandy-Lamport 分布式快照算法实现的。该算法不会暂停整个应用，而是会把**生成检查点的过程和处理过程分离**，这样在部分任务持久化状态的过程中，其他任务还可以继续执行
* Flink 的检查点算法中会用到一类名为检查点分隔符（checkpoint barrier）的特殊记录。和水位线类似，这些检查点分隔符会通过数据源算子注入到常规的记录流中。相对其他记录，它们在流中的位置无法提前或延后。为了标识所属的检查点，每个检查点分隔符都会带有一个检查点编号，这样就把一条数据流从逻辑上分成了两个部分。所有先于分隔符的记录所引起的状态更改都会被包含在分隔符所对应的检查点之中，而所有晚于分隔符的记录所引起的状态更改都会被纳入之后的检查点中

#### 检查点对性能的影响
* 任务在将其状态存入检查点的过程中，会处于阻塞状态，此时的输入会进入缓冲区。按照 Flink 的设计，是由状态后端负责生成检查点，因此任务的状态的具体拷贝过程完全取决于状态后端的实现
* 我们还可以对分隔符对齐这一步进行调整，以降低检查点算法对处理延迟的影响。对于那些需要极低延迟且能容忍至少一次状态保障的应用，可以通过配置让 Flink 在分隔符对齐的过程中不缓冲那些已收到分隔符所对应分区的记录，而是直接处理它们。待所有的检查点分隔符都到达以后，算子才将状态存入检查点，这时候状态可能会包含一些由本应出现在下一次检查点的记录所引起的改动。一旦出现故障，这些记录会被重复处理，而这意味着检查点只能提供至少一次而非精确一次的一致性保障




### 保存点
* 保存点的生成算法和检查点完全一样，因此可以把保存点看做包含一些额外元数据的检查点
* 保存点的生成不是由 Flink 自动完成，而是需要由用户（或外部调度器）显式触发。同时，Flink 也不会自动清理保存点

#### 保存点的使用
* 给定一个应用和一个兼容的保存点，我们可以从该保存点启动应用。相比于检查点，将应用从某个保存点启动还能做更多事情：
    - 从保存点启动一个不同但相互兼容的应用。这意味着你可以修复应用的一些逻辑 bug，然后在数据流来源的支持范围内下尽可能多地重新处理输入事件，以此来修复结果。应用修改还可用于 A/B 测试或需要不同业务逻辑的假想场景。应用和保存点必须相互兼容，只有这样应用才能加载保存点内的状态
    - 用不同的并行度启动原应用，从而实现应用的扩缩容
    - 在另一个集群上启动相同的应用。这允许你把应用迁移到一个新的 Flink 版本，或是一个不同的集群或数据中心
    - 利用保存点暂停某个应用，稍后再把它启动起来。这样可以为更高优先级的应用腾出集群资源，或者在输入数据不连续的情况下及时释放资源
    - 为保存点设置不同版本并将应用状态归档



## 4. DataStream API
### 构建 Flink 流式应用
* 构建 Flink 流式应用步骤：
    1. 设置执行环境
    2. 从数据源中读取一条或多条流
    3. 通过一系列流式转换来实现应用逻辑
    4. 选择性地将结果输出到一个或多个数据汇中
    5. 执行程序




### 转换操作
* 流式转换以一个或多个数据流为输入，并将它们转换成一个或多个输出流
* 完成一个 DataStream API 程序在本质上可以归结为：**通过组合不同的转换来创建一个满足应用逻辑的 Dataflow 图**
* 转换操作分为 4 类：
    - 作用于单个事件的基本转换
    - 针对相同键值事件的 KeyedStream 转换
    - 将多条数据流合并为一条或将一条数据流拆分成多条流的转换
    - 对流中的事件进行重新组织的分发转换

#### 基本转换
* 基本转换会单独处理每个事件，每条输出记录都由单条输入记录所生成
* **Map**
    - 通过调用DataStream.map() 方法可以指定 map 转换产生一个新的 DataStream
    - 该转换将每个到来的事件传给一个用户自定义的映射器，后者针对每个输入只会返回一个输出事件
* **Filter**
    - 利用一个作用在流中每条输入事件上的布尔条件来决定事件的去留：如果返回值为 true，那么它会保留输入事件并将其转发到输出，否则它会把事件丢弃
    - 通过调用 DataStream.filter() 方法可以指定 filter 转换产生一个数据类型不变的 DataStream
* **FlatMap**
    - 可以对每个输入事件产生零个、一个或多个输出事件
    - flatMap 可以看做是 filter 和 map 的泛化，能够实现后两者的操作

#### 基于 KeyedStream 的转换
* KeyedStream 抽象可以从逻辑上将事件按照键值分配到多条独立的子流中
* 作用于 KeyedStream 的状态化转换可以对当前处理事件的键值所对应上下文中的状态进行读写。这意味着所有键值相同的事件可以访问相同的状态，因此它们可以被一并处理
* **keyBy**
    - keyBy 转换通过指定键值的方式将一个 DataStream 转化为 KeyedStream
    - 流中的事件会根据各自键值被分到不同的分区，有着相同键值的事件一定会在后续算子的同一个任务上处理。虽然键值不同的事件也可能会在同一个任务上处理，但任务函数所能访问的键值分区状态始终会被约束在当前事件键值的范围内
* **滚动聚合**
    - 滚动聚合转换作用于 KeyedStream 上，它将生成一个包含聚合结果（例如求和、最小值、最大值等）的 DataStream
    - 滚动聚合算子会对每一个遇到过的键值保存一个聚合结果，每当有新事件到来，该算子都会更新相应的聚合结果，并将其以事件的形式发送出去
    - 滚动聚合方法：
        - sum()：滚动计算输入流中指定字段的和
        - min()：滚动计算输入流中指定字段的最小值
        - max()：滚动计算输入流中指定字段的最大值
        - minBy()：滚动计算输入流中迄今为止最小值，返回该值所在事件
        - maxBy()：滚动计算输入流中迄今为止最大值，返回该值所在事件
    - 多个滚动聚合方法无法组合使用，每次只能计算一个
    - 滚动聚合算子会为每个处理过的键值维持一个状态。由于这些状态不会被自动清理，所以该算子只能用于键值域有限的流
* **Reduce**
    - reduce 转换是滚动聚合转换的泛化
    - reduce 转换将一个 ReduceFunction 应用在一个 KeyedStream 上，每个到来事件都会和 reduce 结果进行一次组合，从而产生一个新的 DataStream
    - reduce 转换不会改变数据类型，因此输出流的类型会永远和输入流保持一致
    - 只能用于键值域有限的流

#### 多流转换
* **Union**
    - DataStream.union() 方法可以合并两条或多条类型相同的 DataStream，生成一个新的类型相同的 DataStream，这样后续的转换操作就可以对所有输入流中的元素统一处理
    - union 执行过程中，来自两条流的事件会以 FIFO（先进先出）的方式合并，其顺序无法得到任何保证
    - union 算子不会对数据进行去重，每个输入消息都会被发往下游算子
* **Connect，coMap，coFlatMap**
    - DataStream.connect() 方法接收一个 DataStream 并返回一个 ConnectedStreams 对象，该对象表示两个联结起来（connected）的流
    - ConnectedStreams 对象提供了 map() 和 flatMap() 方法，它们分别接收一个 CoMapFunction 和一个 CoFlatMapFunction 作为参数
    - 两个函数都是以两条输入流的类型外加输出流的类型作为其类型参数，它们为两条输入流定义了各自的处理方法
    - CoMapFunction 和 CoFlatMapFunction 内方法的调用顺序无法控制。一旦对应流中有事件到来，系统就需要调用对应的方法
    - 默认情况下，connect() 方法不会使两条输入流的事件之间产生任何关联，因此所有事件都会随机分配给算子实例。该行为会产生不确定的结果，而这往往并不是我们希望看到的。为了在 ConnectedStreams 上实现确定性的转换，connect() 可以与 keyBy() 和 broadcast() 结合使用
* **Split 和 Select**
    - split 转换是 union 转换的逆操作。它将输入流分割成两条或多条**类型和输入流相同的输出流**
    - 每一个到来的事件都可以被发往零个、一个或多个输出流。因此，split 也可以用来过滤或复制事件
    - DataStream.split() 方法会返回一个 SplitStream 对象，它提供的 select() 方法可以让我们通过指定输出名称的方式从 SplitStream 中选择一条或多条流

#### 分发转换
* **随机**
    - 利用 DataStream.shuffle() 方法实现随机数据交换策略。该方法会依照均匀分布随机地将记录发往后继算子的并行任务
* **轮流**
    - rebalance() 方法会将输入流中的事件以轮流方法均匀分配给后继任务
* **重调**
    - rescale() 也会以轮流方式对事件进行分发，但分发目标仅限于部分后继任务
    - 本质上看，重调分区策略为发送端和接收端任务不等的情况提供了一种轻量级的负载均衡方法。当接收端任务远大于发送端任务的时候，该方法会更有效，反之亦然
    - rebalance() 和 rescale() 的本质不同体现在**生成任务连接的方式**。rebalance() 会在所有发送任务和接收任务之间建立通信通道；而 rescale() 中每个发送任务只会和下游算子的部分任务建立通道
* **广播**
    - broadcast() 方法会将输入流中的事件复制并发往所有下游算子的并行任务
* **全局**
    - global() 方法会将输入流中的所有事件发往下游算子的第一个并行任务

> 使用此分区策略时需小心所有事件发往同一任务可能会影响程序性能

* **自定义**
    - 利用 partitionCustom() 方法自己定义分区策略。该方法接收一个 Partitioner 对象，可在其中实现分区逻辑，定义分区需要参照的字段或键值位置




### 设置并行度
* 每个算子都会产生一个或多个并行任务。每个任务负责处理算子的部分输入流。算子并行化任务的数目称为该算子的并行度。它决定了算子处理的并行化程度以及能够处理的数据规模
* 默认情况下，应用内所有算子的并行度都会被设置为应用执行环境的并行度。而环境的并行度（即所有算子的默认并行度）则会根据应用启动时所处的上下文自动初始化
* 如果应用是在一个本地执行环境中运行，并行度会设置为 CPU 的线程数目。如果应用是提交到 Flink 集群运行，那么除非提交客户端明确指定，否则环境并行度将设置为集群默认并行度
* 一般情况下，最好将算子并行度设置为随环境默认并行度变化的值，这样就可以通过提交客户端来轻易调整并行度，从而实现应用的扩缩容




### 类型
#### 支持的数据类型
* 原始类型
* Java 和 Scala 元组
    - Java 元组实现：最多可包含 25 个字段，每个字段长度都对应一个单独的实现类——Tuple1、Tuple2，直到 Tuple25。元组中的各个字段可以使用公有字段名称（f0、f1、f2 等）访问，也可以通过 getField(int pos) 方法基于位置访问，位置下标从 0 开始
    - Java 元组是可变的，因此可以为其字段重新复制，Scala 不可
* Scala 样例类
* POJO（包括 Apache Avro 生成的类）
    - Flink 会分析那些不属于任何一类的数据类型，并尝试将它们作为 POJO 类型进行处理。如果一个类满足如下条件，Flink 就会将它看做 POJO：
        - 是一个公有类
        - 有一个公有的无参默认构造函数
        - 所有字段都是公有的或提供了相应的 getter 及 setter 方法。这些方法需要遵循默认的命名规范
        - 所有字段类型都必须是 Flink 所支持的
* 一些特殊类型
    - Flink 支持多种具有特殊用途的类型，例如原始或对象类型的数组，Java 的 ArrayList、HashMap 及 Enum，Hadoop 的 Writable 类型等




### 定义键值和引用字段
#### 字段位置
* 针对元组数据类型，可以简单地使用元组相应元素的字段位置来定义键值
```scala
val input: DataStream[(Int, String, Long)] = ...
val keyed = input.keyBy(1)
```

#### 字段表达式
* 使用基于字符串的字段表达式，可用于元组、POJO 以及样例类，同时还支持选择嵌套的字段
```scala
case class SensorReading(id:String, timestamp:Long, temperature:Double)
val sensorStream: DataStream[SensorReading] = ...
val keyedSensors = sensorStream.keyBy("id")
```

#### 键值选择器
* 使用 KeySelector 函数，可以从输入事件中提取键值
```scala
val input: DataStream[(Int, Int)] = ...
val keyedStream = input.keyBy(value => math.max(value._1, value._2))
```




### 实现函数
#### 函数类
* Flink 中所有用户自定义函数（如 MapFunction、FilterFunction 及 ProcessFunction）的接口都是以接口或抽象类的形式对外暴露
* 我们可以通过实现接口或继承抽象类的方式实现函数
* 当程序提交执行时，所有参数对象都会利用 Java 自身的序列化机制进行序列化，然后发送到对应算子的所有并行任务上。这样在对象反序列化后，全部配置值都可以保留

> Flink 会利用 Java 序列化机制将所有函数对象序列化后发送到对应的工作进程。用户函数中的全部内容都必须是可序列化的
> 如果有函数需要一个无法序列化的对象实例，可以选择使用富函数（rich function），在 open() 方法中将其初始化或者覆盖 Java 的序列化反序列化方法

#### 富函数
* 作用：在函数处理第一条记录之前进行一些初始化工作或是取得函数执行相关的上下文信息
* DataStream API 中所有的转换函数都有对应的富函数。富函数的使用位置和普通函数以及 Lambda 函数相同。它们可以像普通函数类一样接收参数。富函数的命名规则是以 Rich 开头，后面跟着普通转换函数的名字，例如：RichMapFunction、RichFlatMapFunction 等
* 富函数方法：
    - open()：富函数中的初始化方法。它在每个任务首次调用转换方法（如 filter 或 map）前调用一次。open() **常用于那些只需进行一次的设置工作**
    - close()：富函数中的终止方法，会在每个任务最后一次调用转换方法后调用一次，**常用于清理和释放资源**
* 利用 getRuntimeContext() 方法访问函数的 RuntimeContext。在 RuntimeContext 中能够获取到一些信息，例如函数的并行度，函数所在的子任务的编号以及执行函数的任务名称。同时提供了访问分区状态的方法




## 5. 基于时间和窗口的算子
### 配置时间特性
* 时间特性是 StreamExecutionEnvironment 的一个属性，它可以接收以下值：
    - ProcessingTime：指定算子根据处理机器的系统时钟决定数据流当前的时间。处理时间窗口基于机器时间触发，它可以涵盖触发时间点之前到达算子的任意元素。通常情况下，**在窗口算子中使用处理时间会导致不确定的结果**，这是因为窗口内容取决于元素到达的速率。在该配置下，由于处理任务无须依靠等待水位线来驱动事件时间前进，所以可以提供极低的延迟
    - EventTime：指定算子根据数据自身包含的信息决定当前事件。每个事件时间都带有一个时间戳，系统的逻辑时间是由水位线来定义。时间戳或是在数据进入处理管道之前就已经存在其中，或是需要由应用在数据源处分配。只有依靠水位线声明某个时间间隔内所有时间戳都已接收时，事件时间窗口才会触发。即便事件乱序到达，事件时间窗口也会计算出确定的结果。窗口结果不会取决于数据流的读取或处理速度
    - IngestionTime：指定每个接收的记录都把数据源算子的处理时间作为事件时间的时间戳，并自动生成水位线。IngestionTime 是 EventTime 和 ProcessingTime 的混合体，它表示事件进入流处理引擎的事件。和事件时间相比，摄入时间（Ingestion time）的价值不大，因为它的性能和事件时间类似，但却无法提供确定的结果

#### 分配时间戳和生成水位线
* 每个事件都需要关联一个时间戳，该时间戳通常用来表示事件的实际发生时间；此外事件时间数据流还需要携带水位线，以供算子推断当前事件时间
* DataStream API 中提供了 TimestampAssigner 接口，用于从已读入流式应用的元素中提取时间戳。通常情况下，应该在数据源函数后面立即调用时间戳分配器，因为大多数分配器在生成水位线的时候都会做出一些有关元素顺序相对时间戳的假设。由于元素的读取过程通常都是并行的，所以**一切引起 Flink 跨并行数据流分区进行重新分发的操作（例如改变并行度，keyBy() 或显式重新分发）都会导致元素的时间戳发生乱序**
* 最佳做法就是尽可能靠近数据源的地方，甚至是 SourceFunction 内部，分配时间戳并生成水位线。根据用例的不同，如果某些初始化的过滤或其他转换操作不会引起元素的重新分发，那么可以考虑在分配时间戳之前就使用它们
* 时间戳分配器的工作原理和其他转换算子类似。它们会作用在数据流的元素上面，生成一条带有时间戳和水位线的新数据流。时间戳分配器不会改变 DataStream 的数据类型

##### 周期性水位线分配器
* 周期性水位线分配器的含义是我们指示系统以固定的机器时间间隔来发出水位线并推动事件时间前进
* 默认的时间间隔为 200 毫秒，但可以用 ExecutionConfig.setAutoWatermarkInterval() 方法对其进行配置
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
// 每5秒生成一次水位线
env.getConfig.setAutoWatermarkInterval(5000)
```
* 代码中 Flink 会每隔 5 秒调用一次 AssignerWithPeriodWatermarks 中的 getCurrentWatermark() 方法。如果该方法的返回值非空，且它的时间戳大于上一个水位线的时间戳，那么算子就会发出一个新的水位线。**这项检查对于保证事件时间持续递增十分必要，一旦检查失败将不会生成水位线**
* DataStream API 内置了两个针对常见情况的周期性水位线时间戳分配器：
    - 如果输入的元素的时间戳是单调增加的，则可以使用一个简便方法 assignAscendingTimeStamps。基于时间戳不会回退的事实，该方法使用当前时间戳生成水位线
```scala
val stream: DataStream[SensorReading] = ...
val withTimestampsAndWatermarks = stream.assignAscendingTimestamps(e => e.timestamp)
```
    - 如果知道输入流中的延迟（任意新到元素和已到时间戳最大元素之间的时间差）上限，针对这种情况，Flink 提供了 BoundedOutOfOrdernessTimeStampExtractor，它接收一个表示最大预期延迟的参数
```scala
val stream: DataStream[SensorReading] = ...
val output = stream.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor[SensorReading](Time.seconds(10))(e => .timestamp))
```

##### 定点水位线分配器
* 有时候输入流中会包含一些用于指示系统进度的特殊元组或标记。Flink 为此类情形以及可根据输入元素生成水位线的情形提供了 AssignerWithPunctuatedWatermarks 接口。该接口中的  checkAndGetNextWatermark() 方法会在针对每个事件的 extractTimestamp() 方法后立即调用。它可以决定是否生成一个新的水位线。如果该方法返回一个非空、且大于之前值的水位线，算子就会将这个新水位线发出

#### 水位线、延迟及完整性问题
* 水位线可用于平衡延迟和结果的完整性。它们控制着在执行某些计算（例如完成窗口计算并发出结果）前需要等待数据到达的时间。**基于事件时间的算子使用水位线来判断输入记录的完整度以及自身的操作进度**。根据收到的水位线，算子会计算一个所有相关输入记录都已接收完毕的预期时间点




### 处理函数
* DataStream API 提供了一组相对底层的转换——处理函数，可以访问记录的时间戳和水位线，并支持在将来某个特定时间触发的计时器。此外，处理函数的副输出功能还允许将记录发送到多个输出流中
* 处理函数常被用于构建事件驱动型应用，或实现一些内置窗口及转换无法实现的自定义逻辑。例如，大多数 Flink SQL 所支持的算子都是利用处理函数实现的
* Flink 提供了 8 种不同的处理函数：
    - ProcessFunction
    - KeyedProcessFunction
    - CoProcessFunction
    - ProcessJoinFunction
    - BroadcastProcessFunction
    - KeyedBroadcastProcessFunction
    - ProcessWindowFunction
    - ProcessAllWindowFunction
    - 下述以 KeyedProcessFunction 为例
* KeyedProcessFunction 作用于 KeyedStream 之上，会针对流中的每条记录调用一次，并返回零个、一个或多个记录
* 所有处理函数都实现了 RichFunction 接口，支持 open()、close()、getRuntimeContext() 等方法
* KeyedProcessFunction 还提供了 2 个方法：
    - processElement(v: IN, ctx: Context, out: Collector[OUT])：针对流中的每条记录调用一次，可以在方法中将结果记录传递给 Collector 发送出去。Context 对象是让处理函数与众不同的精华所在，可以通过它访问时间戳、当前记录的键值以及 TimerService。此外，Context 还支持将结果发送到副输出
    - onTimer(timestamp: Long, ctx: OnTimerContext, out: Collector[OUT])：是一个回调函数，会在之前注册的计时器触发时被调用。timestamp 参数给出了所触发计时器的时间戳，Collector 可用来发出记录。OnTimerContext 能够提供和 processElement() 方法中的 Context 对象相同的服务，此外，它还会返回触发计时器的时间域（处理时间还是事件时间）

#### 时间服务和计时器
* Context 和 OnTimerContext 对象中的 TimeService 提供了以下方法：
    - currentProcessingTime()：Long 返回当前的处理时间
    - currentWatermark()：Long 返回当前水位线的时间戳
    - registerProcessingTimeTimer(timestamp: Long)：Unit 针对当前键值注册一个处理时间计时器。当执行机器的处理时间到达给定的时间戳时，该计时器就会触发
    - registerEventTimeTimer(timestamp: Long)：Unit 针对当前键值注册一个事件时间计时器。当更新后的水位线时间戳大于或等于计时器的时间戳时，它就会触发
    - deleteProcessingTimeTimer(timestamp: Long)：Unit 针对当前键值删除一个注册过的处理时间计时器。如果该计时器不存在，则方法不会有任何作用
    - deleteEventTimeTimer(timestamp: Long)：Unit 针对当前键值删除一个注册过的事件事件计时器。如果该计时器不存在，则方法不会有任何作用
* 计时器触发时会调用 onTimer() 回调函数。系统对于 processElement() 和 onTimer() 两个方法的调用是同步的，这样可以防止并发访问和操作状态
* 对于每个键值和时间戳只能注册一个计时器，即每个键值可以有多个计时器，但具体到每个时间戳就只能有一个。默认情况下，KeyedProcessFunction 会将全部计时器的时间戳放到堆中的一个优先队列里，同时也可以配置 RocksDB 状态后端来存放计时器

#### 向副输出发送数据
* 大多数 DataStream API 的算子都只有一个输出，即只能生成一条某个数据类型的结果流。而处理函数提供的副输出功能允许从同一函数发出多条数据流，且它们的类型可以不同。
* 每个副输出都由一个 OutputTag[X]对象标识，其中 X 是副输出结果流的类型。处理函数可以利用 Context 对象将记录发送至一个或多个副输出

#### CoProcessFunction
* 针对有两个输入的底层操作，DataStream API 提供了 CoProcessFunction，它提供了一对作用在每个输入上的转换方法——processElement1() 和 processElement2()。在被调用时都会传入一个 Context 对象，用于访问当前元素或计时器时间戳、TimerService 及副输出。
* CoProcessFunction 同样提供了 onTimer() 回调方法




### 窗口算子
* 窗口算子提供了一种基于有限大小的桶对事件进行分组，并对这些桶中的有限内容进行计算的方法

#### 定义窗口算子
* 窗口算子可以用在键值分区或非键值分区的数据流上。用于键值分区窗口的算子可以并行计算，而非键值分区窗口只能单线程处理
* 新建一个窗口算子需要指定两个窗口组件：
    1. 一个用于决定输入流中的元素该如何划分的窗口分配器（window assigner）。窗口分配器会产生一个 WindowedStream（如果用在非键值分区的 DataStream 上则是 AllWindowedStream）
    2. 一个作用于 WindowedStream（或 AllWindowedStream）上，用于处理分配到窗口中元素的窗口函数

#### 内置窗口分配器
* 基于时间的窗口分配器会根据元素事件时间的时间戳或当前处理时间将其分配到一个或多个窗口。每个时间窗口都有一个开始时间戳和一个结束时间戳
* 所有内置的窗口分配器都提供了一个默认的触发器，一旦（处理或事件）时间超过了窗口的结束时间就会触发窗口计算
* 窗口会随着系统首次为其分配元素而创建，Flink 永远不会对空窗口执行计算
* Flink 内置窗口分配器所创建的窗口类型为 TimeWindow。该窗口类型实际上表示两个时间戳之间的时间区间（左闭右开）。它对外提供了获取窗口边界、检查窗口是否相交以及合并重叠窗口等方法
* **滚动窗口**
    - DataStream API 针对事件时间和处理时间的滚动窗口分别提供了对应的分配器——TumblingEventTimeWindows 和TumblingProcessingTimeWindows
    - 滚动窗口分配器只接收一个参数：以时间单元表示的窗口大小。它可以利用分配器的 of(Time size) 方法指定。时间间隔允许以毫秒、秒、分钟、小时或天数来表示

```scala
val sensorData: DataStream[SensorReading] = ...
val avgTemp = sensorData
			.keyBy(_.id)
			// 将读数按照 1 秒事件时间窗口分组
			.window(TumblingEventTimeWindows.of(Time.seconds(1)))
			.process(new TemperatureAverager)
```

* **滑动窗口**
    - 需要指定窗口大小以及用于定义新窗口开始频率的滑动间隔
    - 如果滑动间隔小于窗口大小，则窗口会出现重叠，此时元素会被分配给多个窗口；如果滑动间隔大于窗口大小，则一些元素可能不会分配给任何窗口，因此可能会被直接丢弃

```scala
// 事件时间滑动窗口分配器
val slidingAvgTemp = sensorData
					.keyBy(_.id)
					// 每隔 15 分钟创建 1 小时的事件时间窗口
					.window(SlidingEventTimeWindows.of(Time.hour(1), Time.minutes(15)))
					.process(new TemperatureAverager)
```

* **会话窗口**
    - 会话窗口将元素放入长度可变且不重叠的窗口中。会话窗口的边界由非活动间隔，即持续没有收到记录的时间间隔来定义
    - 由于会话窗口的开始和结束都取决于接收的元素，所以窗口分配器无法实时将所有元素分配到正确的窗口。事实上，SessionWindows 分配器会将每个到来的元素映射到一个它自己的窗口中。该窗口的起始时间是元素的时间戳，大小为会话间隔。随后分配器会将所有范围存在重叠的窗口合并

```scala
// 事件时间会话窗口分配器
val sessionWindows = sensorData
					.keyBy(_.id)
					// 创建 15 分钟间隔的事件时间会话窗口
			.window(EventTimeSessionWindows.withGap(Time.minutes(15)))
					process(...)
```

#### 在窗口上应用函数
* 可用于窗口的函数类型有两种：
    1. 增量聚合函数。它的应用场景是窗口内以状态形式存储某个值且需要根据每个加入窗口的元素对该值进行更新。此类函数通常会十分节省空间且最终会将聚合值作为单个结果发送出去，如 ReduceFunction 和 AggregateFunction
    2. 全量窗口函数。它会收集窗口内的所有元素，并在执行计算时对它们进行遍历。虽然全量窗口函数通常需要占用更多空间，但它和增量聚合函数相比，支持更复杂的逻辑，如 ProcessWindowFunction
* **ReduceFunction**
    - ReduceFunction 接收两个同类型的值并将它们组合生成一个类型不变的值。当被用在窗口化数据流上时，ReduceFunction 会对分配给窗口的元素进行增量聚合。窗口只需要存储当前聚合结果，一个和 ReduceFunction 的输入及输出类型都相同的值。每当收到一个新元素，算子都会以该元素和从窗口状态取出的当前聚合值为参数调用 ReduceFunction，随后会用 ReduceFunction 的结果替换窗口状态

```scala
val minTempPerWindow: DataStream[(String, Double)] = sensorData
	.map(r => (r.id, r.temperature))
	.keyBy(_._1)
	.timeWindow(Time.seconds(15))
	.reduce((r1, r2) => (r1._1, r1._2.min(r2._2)))
```

* **AggregateFunction**
    - 和 ReduceFunction 类似，AggregateFunction 也会以增量方式应用于窗口内的元素，其状态也只有一个值
    - 下列代码展示了如何用 AggregateFunction 计算每个窗口内传感器读数的平均温度。其累加器负责维护不断变化的温度总和及数量，getResult() 方法用来计算平均值

```scala
val avgTempPerWindow: DataStream[(String, Double)] = sensorData
	.map(r => (r.id, r.temperature))
	.keyBy(_._1)
	.timeWindow(Time.seconds(15))
	.aggregate(new AvgTempFunction)

// 用于计算每个传感器平均温度的 AggregateFunction
// 累加器用于保存温度总和及事件数量
class AvgTempFunction extends AggregateFunction[(String, Double), (String, Double, Int), (String, Double)]{
	override def createAccumulator() = {
		("", 0.0, 0)
	}
	override def add(in: (String, Double), acc: (String, Double, Int)) = {
		(in._1, in._2 + acc._2, 1 + acc._3)
	}
	override def getResult(acc: (String, Double, Int)) = {
		(acc._1, acc._2 / acc._3)
	}
	override def merge(acc1: (String, Double, Int), acc2: (String, Double, Int)) = {
		(acc1._1, acc1._2 + acc2._2, acc1._3 + acc2._3)
	}
}
```

* **ProcessWindowFunction**
    - ReduceFunction 和 AggregateFunction 都是对分配到窗口的事件进行增量计算。然而有些时候我们需要访问窗口内的所有元素来执行一些更加复杂的计算，例如计算窗口内数据的中值或出现频率最高的值。
    - ProcessWindowFunction 可以对窗口内容执行任意计算
    - process() 方法在被调用时会传入窗口的键值、一个用于访问窗口内元素的 Iterator 以及一个用于发出结果的 Collector。此外，该方法和其他处理方法一样都有一个 Context 参数。ProcessWindowFunction 的 Context 对象可以访问窗口的元数据，当前处理时间和水位线，用于管理单个窗口和每个键值全局状态的状态存储以及用于发出数据的副输出
    - ProcessWindowFunction 中的 Context 对象具有一些特有功能，即访问单个窗口的状态及每个键值的全局状态。
        - 其中单个窗口的状态指的是当前正在计算的窗口实例的状态，而全局状态指的是不属于任何一个窗口的键值分区状态。
        - 单个窗口状态用于维护同一窗口内多次调用 process() 方法所需共享的信息，这种多次调用可能是由于配置了允许数据迟到或使用了自定义触发器
        - 使用了单个窗口状态的 ProcessWindowFunction 需要实现 clear() 方法，在窗口清除前清理仅供当前窗口使用的状态
        - 全局状态可用于在键值相同的多个窗口之间共享信息
    - 在系统内部，**由 ProcessWindowFunction 处理的窗口会将已分配的事件存储在 ListState 中**。通过将所有事件收集起来且提供对于窗口元数据及其他一些特性的访问和使用，ProcessWindowFunction 的应用场景比 ReduceFunction 和 AggregateFunction 更加广泛。但和执行增量聚合的窗口相比，**收集全部事件的窗口其状态要大得多**

```scala
public abstract class ProcessWindowFunction<IN, OUT, KEY, W extends Window> extends AbstractRichFunction {
	// 对窗口执行计算
	void process(KEY key, Context ctx, Iterable<IN> vals, Collector<OUT> out) throws Exception;
	// 在窗口清除时删除自定义的单个窗口状态
	public void clear(Context ctx) throws Exception {}
	// 保存窗口元数据的上下文
	public abstract class Context implements Serializable {
		// 返回窗口的元数据
		public abstract W window();
		// 返回当前处理时间
		public abstract long currentProcessingTime();
		// 返回当前事件时间水位线
		public abstract long currentWatermark();
		// 用于单个窗口状态的访问器
		public abstract KeyedStateStore windowState();
		// 用于每个键值全局状态的访问器
		public abstract KeyedStateStore globalState();
		// 向 OutputTag 标识的副输出发送记录
		public abstract <X> void output(OutputTag<X> outputTag, X value);
	}
}
```

* **增量聚合与 ProcessWindowFunction**
    - ProcessWindowFunction 和增量聚合函数相比通常需要在状态中保存更多数据
    - 如果可用增量聚合表示逻辑但还需要访问窗口元数据，则可以将 ReduceFunction 或 AggregateFunction 与功能更强的 ProcessWindowFunction 组合使用
    - 可以对分配给窗口的元素立即执行聚合，随后当窗口触发器触发时，再将聚合后的结果传给 ProcessWindowFunction。这样传递给 ProcessWindowFunction.process() 方法的 Iterable 参数内将只有一个值，即增量聚合的结果
    - DataStream API 中，实现上述过程的途径是将 ProcessWindowFunction 作为 reduce() 或 Aggregate() 方法的第二个参数

```scala
val minMaxTempPerWindow: DataStream[MinMaxTemp] = sensorData
	.map(r => (r.id, r.temperature, r.temperature))
	.keyBy(_._1)
	.timeWindow(Time.seconds(5))
	.reduce(
		// 增量计算最低和最高温度
		(r1: (String, Double, Double), r2: (String, Double, Double)) => {
			(r1._1, r1._2.min(r2._2), r1._3.max(r2._3))
		},
		// 在 ProcessWindowFunction 中计算最终结果
		new AssignWindowEndProcessFunction()
	)
```

#### 自定义窗口算子
* DataStream API 对外暴露了自定义窗口算子的接口和方法，你可以实现自己的**分配器（assigner）**、**触发器（trigger）**以及**移除器（evictor）**
* 当一个元素进入窗口算子时会被移交给 WindowAssigner，该分配器决定了元素应该被放入哪几个窗口中。如果目标窗口不存在，则会创建它
* 如果为窗口算子配置的是增量聚合函数，那么新加入的元素会立即执行聚合，其结果会作为窗口内容存储。如果窗口算子没有配置增量聚合函数，那么新加入的元素会附加到一个用于存储所有窗口分配元素的 ListState 上
* 每个元素在加入窗口后还会被传递至该窗口的触发器。**触发器定义了窗口何时准备好执行计算，何时需要清除自身及保存的内容**。触发器可以根据已分配的元素或注册的计时器（类似处理函数）来决定在某些特定时刻执行计算或清除窗口中的内容
* 触发器成功触发后的行为取决于窗口算子所配置的函数。
    - 如果算子只是**配置了一个增量聚合函数**，就会发出当前聚合结果。
    - 如果算子只**包含一个全量窗口函数**，那么该函数将一次性作用于窗口内的所有元素上，之后便会发出结果。
    - 如果算子**同时拥有一个增量聚合函数和一个全量窗口函数**，那么后者将作用于前者产生的聚合值上，之后便会发出结果
* 移除器作为一个可选组件，允许在 ProcessWindowFunction 调用之前或之后引入。它可以用来**从窗口中删除已经收集的元素**。由于**需要遍历所有元素**，移除器只有在未指定增量聚合函数的时候才能使用

##### 窗口的生命周期
* 窗口会在 WindowAssigner 首次向它分配元素时创建。因此，**每个窗口至少会有一个元素**
* 窗口内的状态由以下几部分组成：
    - 窗口内容：包含分配给窗口的元素，或当窗口算子配置了 ReduceFunction 或 AggregateFunction 时增量聚合所得到的结果
    - 窗口对象：WindowAssigner 会返回零个、一个或多个窗口对象。窗口算子会根据返回的对象对元素进行分组。因此窗口对象中保存着用于区分窗口的信息。每个窗口对象都有一个结束时间戳，它**定义了可以安全删除窗口及其状态的时间点**
    - 触发器计时器：可以在触发器中注册计时器，用于在将来某个时间点触发回调（如对窗口进行计算或清理其内容）。这些计时器由窗口算子负责维护
    - 触发器中的自定义状态
* 窗口算子会在窗口结束时间（由窗口对象中的结束时间戳定义）到达时**删除窗口**。该时间是处理时间还是事件时间语义取决于 WindowAssigner.isEventTime() 方法的返回值
* 当窗口需要删除时，窗口算子会自动清除窗口内容并丢弃窗口对象

> 自定义触发器状态和触发器计时器不会被清除，因为这些状态对于窗口算子而言是不可见的。所以为了避免状态泄露，触发器需要在 Trigger.clear() 方法中清除自身所有状态

##### 窗口分配器
* WindowAssigner 用于决定将到来的元素分配给哪些窗口。每个元素可以被加到零个、一个或多个窗口中。
* WindowAssigner 接口：

```scala
public abstract class WindowAssigner<T, W extends Window> implements Serializable {
	// 返回元素分配的目标窗口集合
	public abstract Collection<W> assignWindows(T element, long timestamp, WindowAssignerContext context);
	// 返回 WindowAssigner 的默认触发器
	public abstract Trigger<T, W> getDefaultTrigger(StreamExecutionEnvironment env);
	// 返回 WindowAssigner 中窗口的 TypeSerializer
	public abstract TypeSerializer<W> getWindowSerializer(ExecutionConfig executionConfig);
	// 表明此分配器是否创建基于事件时间的窗口
	public abstract boolean isEventTime();
	// 用于访问当前处理时间的上下文
	public abstract static class WindowAssignerContext {
		// 返回当前处理时间
		public abstract long getCurrentProcessingTime();
	}
}
```

##### 触发器
* 触发器用于定义何时对窗口进行计算并发出结果
* 触发条件可以是时间，也可以是某些特定的数据条件，如元素数量或某些观测到的元素值
* 每次调用触发器都会生成一个 TriggerResult，它用于决定窗口接下来的行为，可以是以下值之一：
    - CONTINUE：什么都不做
    - FIRE：如果窗口算子配置了 ProcessWindowFunction，就会调用该函数并发出结果；如果窗口只包含一个增量聚合函数，则直接发出当前聚合结果。窗口状态不会发生任何变化
    - PURGE：完全清除窗口内容，并删除窗口自身及其元数据。同时，调用 ProcessWindowFunction.clear() 方法来清理那些自定义的单个窗口状态
    - FIRE_AND_PURGE：先进行窗口计算（FIRE），随后删除所有状态及元数据（PURGE）
* Trigger 接口：

```scala
public abstract class Trigger<T, W extends Window> implements Serializable {
	// 每当有元素添加到窗口时都会调用
	TriggerResult onElement(T element, long timestamp, W window, TriggerContext ctx);
	// 在处理时间计时器触发时调用
	public abstract TriggerResult onProcessingTime(long timestamp, W window, TriggerContext ctx);
	//在事件时间计时器触发时调用
	public abstract TriggerResult onEventTime(long timestamp, W window, TriggerContext ctx);
	// 如果触发器支持合并触发器状态则返回 true
	public boolean canMerge();
	// 当多个窗口合并为一个窗口且需要合并触发器状态时调用
	public void onMerge(W window, OnMergeContext ctx);
	// 在触发器中清除那些为给定窗口保存的状态，该方法会在清除窗口时调用
	public abstract void clear(W window, TriggerContext ctx);
}

// 用于触发器中方法的上下文对象，使其可以注册计时器回调并处理状态
public interface TriggerContext {
	// 返回当前处理时间
	long getCurrentProcessingTime();
	// 返回当前水位线时间
	long getCurrentWatermark();
	// 注册一个处理时间计时器
	void registerProcessingTimeTimer(long time);
	// 注册一个事件时间计时器
	void registerEventTimeTimer(long time);
	// 删除一个处理时间计时器
	void deleteProcessingTimeTimer(long time);
	// 删除一个事件时间计时器
	void deleteEventTimeTimer(long time);
	// 获取一个作用域为触发器键值和当前窗口的状态对象
	<S extends State> S getPartitionedState(StateDescriptor<S, ?> stateDescriptor);
}

// 用于 Trigger.onMerge() 方法的 TriggerContext 扩展
public interface OnMergeContext extends TriggerContext {
	// 合并触发器中的单个窗口状态，目标状态自身需要合并
	void mergePartitionedState(StateDescriptor<S, ?> stateDescriptor);
}
```

* 当在触发器中使用了单个窗口状态时，需要保证它们会随着窗口删除而被正确地清理，否则窗口算子的状态会越积越多，最终可能导致你的应用在某个时间出现故障。为了在删除窗口时彻底清理状态，触发器的 clear() 方法需要删除全部自定义的单个窗口状态并使用 TriggerContext 对象删除所有处理时间和事件时间计时器。由于在删除窗口后不会调用计时器回调方法，所以无法在其中清理状态

##### 移除器
* Evictor 接口：

```scala
public interface Evictor<T, W extends Window> extends Serializable {
	// 选择性地移除元素。在窗口函数之前调用
	void evictBefore(Iterable<TimestampedValue<T>> elements, int size, W window, EvictorContext evictorContext);
	// 选择性地移除元素。在窗口函数之后调用
	void evictAfter(Iterable<TimestampedValue<T>> elements, int size, W window, EvictorContext evictorContext);
	// 用于移除器内方法的上下文对象
	interface EvictorContext {
		// 返回当前处理时间
		long getCurrentProcessingTime();
		// 返回当前事件时间水位线
		long getCurrentWatermark();
	}
}
```

* 移除器常用于 GlobalWindow，它支持清理部分窗口内容而不必完全清除整个窗口状态

> **GlobalWindows 分配器**
> GlobalWindows 分配器会将所有元素映射到一个全局窗口中。它默认的触发器是 NeverTrigger，该触发器永远不会触发。因此 GlobalWindow 分配器需要一个自定义的触发器，可能还需要一个移除器来有选择性地将元素从窗口状态中删除




### 基于时间的双流 Join
* Flink DataStream API 中内置有两个可以根据时间条件对数据流进行 Join 的算子：基于间隔的 Join 和基于窗口的 Join

#### 基于间隔的 Join
* 基于间隔的 Join 会对两条流中拥有相同键值以及彼此之间时间戳不超过某一指定间隔的事件进行 Join
* Join 成功的事件对会发送给 ProcessJoinFunction。下界和上界分别由负时间间隔和正时间间隔来定义，例如 between(Time.hour(-1), Time.minute(15))。在满足下界值小于上界值的前提下，可以任意对它们赋值
* 基于间隔的 Join 需要同时对双流的记录进行缓冲

#### 基于窗口的 Join
* 用到 Flink 中的窗口机制。其原理是将两条输入流中的元素分配到公共窗口中并在窗口完成时进行 Join
* 两条输入流都会根据各自的键值属性进行分区，公共窗口分配器会将二者的事件映射到公共窗口内。当窗口的计时器触发时，算子会遍历两个输入中元素的每个组合（叉乘积）去调用 JoinFunction。由于两条流中的事件会被映射到同一个窗口中，因此该过程中的触发器和移除器与常规窗口算子中的完全相同




### 处理迟到数据
* 迟到是指元素到达算子后，它本应参与贡献的计算已经执行完毕。在事件时间窗口算子的环境下，如果事件到达算子时窗口分配器为其分配的窗口已经因为算子水位线超过了它的结束时间而计算完毕，那么该事件就被认为是迟到的
* DataStream API 应对迟到数据策略：
    - 简单地将其丢弃
    - 将迟到事件重定向到单独的数据流中
    - 根据迟到事件更新并发出计算结果

#### 丢弃迟到事件
* 事件时间窗口的默认行为

#### 重定向迟到事件
* 利用副输出将迟到事件重定向到另一个 DataStream，就可以对它们进行后续处理或利用常规的数据汇函数将其写出。迟到数据可通过定期的回填操作集成到流式应用的结果中
* 处理函数可通过比较事件时间的时间戳和当前水位线来识别迟到事件，并使用常规的副输出 API 将其发出

#### 基于迟到事件更新结果
* 策略：对不完整的结果进行重新计算并发出更新
* 在使用事件时间窗口时，可以制定一个名为延迟容忍度的额外时间段。配置了该属性的窗口算子在水位线超过窗口的结束时间戳之后不会立即删除窗口，而是会将窗口继续保留该延迟容忍度的事件。在这段额外时间内到达的迟到元素会像按时到达的元素一样交给触发器处理。当水位线超过了窗口结束时间加延迟容忍度的间隔，窗口才会被最终删除，伺候所有的迟到元素都将直接丢弃




## 6. 有状态算子和应用
* 由于数据会随时间以流式到来，大多数复杂一些的操作都需要存储部分数据或中间结果
* 很多 Flink 内置的 DataStream 算子、数据源以及数据汇都是有状态的，它们需要对数据记录进行缓冲或者对中间结果及元数据加以维护
* 状态化流处理会在故障恢复、内存管理以及流式应用维护等很多方面对流处理引擎产生影响

### 实现有状态函数
#### 在 RuntimeContext 中声明键值分区状态
* Flink 为键值分区状态提供了很多原语。状态原语定义了单个键值对应的状态结构。它的选择取决于函数和状态的交互方式。同时，由于每个状态后端都为这些原语提供了自己的实现，该选择还会影响函数的性能。
* Flink 目前支持以下状态原语：
    - ValueState[T]：用于保存类型为 T 的单个值
    - ListState[T]：用于保存类型为 T 的元素列表
    - MapState[K, V]：用于保存一组键到值的映射
    - ReducingState[T]：提供了和 ListState[T] 相同的方法，但它的 ReducingState.add(value: T) 方法会立刻返回一个使用 ReduceFunction 聚合后的值
    - AggregatingState[I, O]: 和ReducingState 行为类似，但它使用了更加通用的 AggregateFunction 来聚合内部的值

#### 通过 ListCheckpointed 接口实现算子列表状态
* 在函数中使用算子列表状态，需要实现 ListCheckpointed 接口。该接口不像 ValueState 或 ListState那样直接在状态后端注册，而是需要将算子状态实现为成员变量并通过接口提供的回调函数与状态后端进行交互
* ListCheckpointed 接口提供了两个方法：
    - snapshotState()：会在 Flink 触发为有状态函数生成检查点时调用
    - restoreState()：会在初始化函数状态时调用，该过程可能发生在作业启动或故障恢复的情况下

##### 使用联结的广播状态
* 流式应用的一个常见需求是将相同信息发送到函数的所有并行实例上，并将它们作为可恢复的状态进行维护。在 Flink 中，这种状态称为广播状态
* 在两条数据流上应用带有广播状态的函数需要三个步骤：
    - 调用 DataStream.broadcast() 方法创建一个 BroadcastStream 并提供一个或多个 MapStateDescriptor 对象。每个描述符都会为将来用于 BroadcastStream 的函数定义一个单独的广播状态
    - 将 BroadcastStream 和一个 DataStream 或 KeyedStream 联结起来。必须将 BroadcastStream 作为参数传给 connect() 方法
    - 在联结后的数据流上应用一个函数。根据另一条流是否已经按键值分区，该函数可能是 KeyedBroadcastProcessFunction 或 BroadcastProcessFunction

#### 使用 CheckpointedFunction 接口
* CheckpointedFunction 是用于指定有状态函数的最底层接口。它提供了用于注册和维护键值分区状态以算子状态的钩子函数（hook），同时也是唯一支持使用算子联合列表状态的接口
* CheckpointedFunction 接口定义了两个方法：
    - initializeState()：会在创建 CheckpointedFunction 的并行实例时被调用。触发时机是应用启动或由于故障而重启任务
    - snapshotState()：会在生成检查点之前调用。目的是确保检查点开始之前所有状态对象都已更新完毕。该方法还可以结合 CheckpointedListener 接口使用，在检查点同步阶段将数据一致性地写入外部存储中

> 我们在注册任意一个状态时，都要提供一个函数范围内唯一的名称。在函数注册状态过程中，状态存储首先会利用给定名称检查状态后端中是否存在一个为当前函数注册过的同名状态，并尝试用它对状态进行初始化。如果是重启任务（无论由于故障还是从保存点恢复）的情况，Flink 就会用保存的数据初始化状态；如果应用不是从检查点或保存点启动，那状态就会初始化为空

#### 接收检查点完成通知
* **频繁地同步是分布式系统产生性能瓶颈的主要原因**。Flink 的设计旨在减少同步点的数量，其内部的检查点是基于和数据一起流动的分隔符来实现的，因此可以避免对应用所有算子实施全局同步
* 得益于该检查点机制，Flink 有着非常好的性能表现。但也意味着除了生成检查点的几个逻辑时间点外，应用程序的状态无法做到强一致。了解检查点的完成情况非常重要
* 在所有算子任务都成功将其状态写入检查点存储后，整体的检查点才算创建成功。因此，只有 JobManager 才能对此作出判断。算子为了感知检查点创建成功，可以实现 CheckpointedListener 接口。该接口提供的 notifyCheckpointComplete(long checkpointId) 方法，会在 JobManager 将检查点注册为已完成时被调用




### 为有状态的应用开启故障恢复
* Flink 为有状态的应用创建一致性检查点的机制：在所有算子都处理到应用输入流的某一特定位置时，为全部内置或用户定义的有状态函数基于该时间点创建一个状态快照。为了支持应用容错，JobManager 会以固定间隔创建检查点
* 检查点间隔是影响常规处理期间创建检查点的开销以及故障恢复所需时间的一个重要参数，较短的间隔会为常规处理带来较大的开销，但由于恢复时要重新处理的数据量较小，所以恢复速度会更快
* Flink 还为检查点行为提供了其他一些可供调节的配置选项，例如，一致性保障（精确一次或至少一次）的选择，可同时生成的检查点的数目以及用来取消长时间运行检查点的超时时间，以及多个和状态后端相关的选项




### 确保有状态应用的可维护性
* 在应用运行较长一段时间后，其状态就会变得成本十分昂贵，甚至无法重新计算。同时，我们也需要对长时间运行的应用进行一些维护。例如，修复 Bug，添加、删除或调整功能，或针对不同的数据到来速率算子的并行度
* Flink 利用保存点机制来对应用及其状态进行维护，但它需要初始版本应用的全部有状态算子都指定好两个参数，才可以在未来正常工作。这两个参数是**算子唯一标识**和**最大并行度**

> 算子的唯一标识和最大并行度会被固化到保存点中，不可更改。如果新应用中这两个参数发生了变化，则无法从之前生成的保存点启动。
> 一旦你修改了算子标识或最大并行度则无法从保存点启动应用，只能选择丢弃状态从头开始运行

#### 指定算子唯一标识
* 为应用中的每个算子指定唯一标识，该标识会作为元数据和算子的实际状态一起写入保存点。当应用从保存点启动时，会利用这些标识将保存点中的状态映射到目标应用对应的算子。只有当目标应用的算子标识和保存点中的算子标识相同时，状态才能顺利恢复
* 如果没有为有状态应用的算子显式指定标识，那么在更新应用时就会受到诸多限制

#### 为使用键值分区状态的算子定义最大并行度
* 算子的最大并行度参数定义了算子在对键值状态进行分割时，所能用到的键值组数量。该数量限制了键值分区状态可以被扩展到的最大并行任务数
* 可以通过 StreamExecutionEnvironment 为应用的所有算子设置最大并行度或利用算子的 setMaxParallelism() 方法为每个算子单独设置




### 有状态应用的性能及鲁棒性
* 算子和状态的交互会对应用的鲁棒性及性能产生一定影响。这些影响的原因是多方面的，例如，状态后端的选择（影响本地状态如何存储和执行快照），检查点算法的配置以及应用状态大小等

#### 选择状态后端
* 状态后端负责存储每个状态实例的本地状态，并在生成检查点时将它们写入远程持久化存储。由于本地状态的维护及写入检查点的方式多种多样，所以状态后端被设计为“可插拔的”，两个应用可以选择不同的状态后端实现来维护其状态
* 状态后端的选择会影响有状态应用的鲁棒性及性能
* 每一种状态后端都为不同的状态原语（如 ValueState、ListState 和 MapState）提供了不同的实现
* Flink 提供了三种状态后端：
    - MemoryStateBackend：将状态以常规对象的方式存储在 TaskManager 进程的 JVM 堆里。如果某个任务实例的状态变得很大，那么它所在的 JVM 连同所有运行在该 JVM 之上的任务实例都可能由于 OutOfMemoryError 而终止。此外，该方法可能由于堆中放置了过多常驻内存的对象而引发垃圾回收停顿问题。在生成检查点时，MemoryStateBackend 会将状态发送至 JobManager 并保存到它的堆内存中。因此 JobManager 的内存需要装得下应用的全部状态。因为内存具有易失性，所以一旦 JobManager 出现故障，状态就会丢失。由于存在这些限制，**建议仅将 MemoryStateBackend 用于开发和调试**
    - FsStateBackend：和MemoryStateBackend 一样，将本地状态保存在 TaskManager 的 JVM 堆内。但它不会在创建检查点时将状态存到 JobManager 的易失内存中，而是会将它们**写入远程持久化文件系统**。因此，FsStateBackend 既让本地访问享有内存的速度，又可以支持故障容错。但它同样会受到 TaskManager 内存大小的限制，并且也可能导致垃圾回收停顿问题
    - RocksDBStateBackend：会把全部状态存到本地 RocksDB 实例中。RocksDB 是一个嵌入式键值存储（key-value store），它可以将数据保存到**本地磁盘**上。为了从 RocksDB 中读写数据，系统需要对数据进行序列化和反序列化。RocksDBStateBackend 同样会将状态以检查点形式写入远程持久化文件系统。因为它能够将数据写入磁盘，且支持增量检查点，所以对于状态非常大的应用是一个很好的选择。然而，对于磁盘的读写和序列化反序列化对象的开销使得它和在内存中维护状态比起来，读写性能会偏低

```scala
// 为应用配置 RocksDBStateBackend
val env = StreamExecutionEnvironment.getExecutionEnvironment
val checkpointPath: String = ???
// 远程文件系统检查点配置路径
val backend = new RocksDBStateBackend(checkpointPath)
// 配置状态后端
env.setStateBackend(backend)
```

#### 选择状态原语
* 有状态算子（无论是内置的还是用户自定义的）的性能取决于多个方面，包括状态的数据类型，应用的状态后端以及所选的状态原语
* 对于在读写状态时涉及对象序列化和反序列化的状态后端，状态原语的选择将对应用性能产生决定性的影响
* ValueState 需要在更新和访问时分别进行完整的序列化和反序列化。在构造用于数据访问的 Iterable 对象之前，RocksDBStateBackend 的 ListState 需要将它所有的列表条目反序列化。但向 ListState 中添加一个值的操作会相对轻量级一些，因为它只会序列化新添加的值。RocksDBStateBackend 的 MapState 允许按照每个键对其数据值进行读写，并且只有那些读写的键和数据值才需要进行序列化。在遍历 MapState 的条目集时，状态后端会从 RocksDB 中预取出序列化好的所有条目，并只有在实际访问某个键或数据值的时候才会将其反序列化

#### 防止状态泄露
* 流式应用经常会被设计成需要长年累月地连续运行。应用状态如果不断增加，总有一天会变得过大并“杀死”应用。为了防止应用资源逐渐耗尽，关键要控制算子状态大小。由于对状态的处理会直接影响算子语义，所以 Flink 无法通过自动清理状态来释放资源。所有有状态算子都要控制自身状态大小，确保它们不会无限制增长
* 导致状态增长的一个常见原因是键值状态的键值域不断发生变化。在该场景下，有状态函数所接收记录的键值只有一段特定时间的活跃期，此后就再也不会收到。随着键值空间的不断变化，状态中那些过期的旧键值会变得毫无价值。该问题的解决方案是从状态中删除那些过期的键值。然而，具有键值分区状态的函数只有在收到某键值的记录时才能访问该键值的状态。很多情况下，函数不会知道某条记录是否是该键值所对应的最后一条。因此它根本无法准确移除某一键值的状态
* 在设计和实现有状态算子的时候，需要把应用需求和输入数据的属性（如键值域）都考虑在内。如果你的应用需要用到键值域不断变化的键值分区状态，那么必须要确保能够对那些无用的状态进行清除。该工作可以通过注册针对未来某个时间点的计时器来完成。和状态类似，计时器也会注册在当前活动键值的上下文中。计时器在触发时，会调用回调方法并加载计时器键值的上下文。因此在回调方法内可以获得当前键值状态的完整访问权限并将其清除




### 更新有状态应用
* 很多时候我们需要对一个长时间运行的有状态的流式应用进行 Bug 修复或业务逻辑调整。这往往要求我们在不丢失状态的前提下对当前运行的应用进行版本更新
* 当应用从保存点启动时，它的算子会使用算子标识和状态名称从保存点中查找对应的状态进行初始化。从保存点兼容性的角度看，应用可以通过以下三种方式进行更新：
    1. 在不对已有状态进行更改或删除的前提下更新或扩展应用逻辑，包括向应用中添加有状态或无状态的算子
    2. 从应用中移除某个状态
    3. 通过改变状态原语或数据类型来修改已有算子的状态

#### 保持现有状态更新应用
* 如果应用在更新时不会删除或改变已有状态，那么它一定是保存点兼容的，并且能够从旧版本的保存点启动
* 如果你向应用中添加了新的状态算子或已有算子增加了状态，那么在应用从保存点启动时，这些状态都会被初始化为空

#### 从应用中删除状态
* 删除操作针对的可以是一个完整的有状态算子，也可以是函数中的某个状态。当新版本的应用从一个旧版本的保存点启动时，保存点中的部分状态将无法映射到重启的应用中。如果算子的唯一标识或状态名称发生了改变，也会出现这种情况
* 为了避免保存点中的状态丢失，Flink 在默认情况下不允许那些无法将保存点中的状态全部恢复的应用启动，可以禁用这一安全检查。

#### 修改算子的状态
* 两种方法对状态进行修改：
    - 通过更改状态的数据类型，例如将 ValueState[Int] 改为 ValueState[Double]
    - 通过更改状态原语类型，例如将 ValueState[List[String]] 改为 ListState[String]

#### 可查询式状态
* 很多处理应用需要将它们的结果与其他应用分享。常见的分享模式是先把结果写入数据库或键值存储中，再由其他应用从这些存储中获取结果
* 为了解决以往需要外部数据存储才能分享数据的问题，Apache Flink 提供了可查询式状态功能。在 Flink 中，任何键值分区状态都可以作为可查询式状态暴露给外部应用，就像一个只读的键值存储一样。有状态的流式应用可以按照正常流程处理事件，并在可查询状态中对其中间或最终结果进行存储和更新。外部应用可以在流式应用运行过程中访问某一键值的状态
* 可查询式状态无法应对所有需要外部数据存储的场景。原因之一：它只有在应用运行过程中才可以访问。如果应用正在因为错误而重启、正在进行扩缩容或正在迁移至其他集群，那么可查询式状态将无法访问

#### 可查询式状态服务的架构及启动方式
* Flink 的可查询式状态服务包含三个进程：
    - QueryableStateClient 用于外部系统提交查询及获取结果
    - QueryableStateClientProxy：用于接收并响应客户端请求。该客户端代理需要在每个 TaskManager 上面都运行一个实例。由于键值分区状态会分布在算子所有并行实例上面，所以代理需要识别请求键值对应的状态所在的 TaskManager。该信息可以从负责键值组分配的 JobManager 上面获得，代理在一次请求过后就会将它们缓存下来。客户端代理从各自 TaskManager 的状态服务器上取得状态，然后把结果返给客户端
    - QueryableStateServer：用于处理客户端代理的请求。状态服务器同样需要运行在每个 TaskManager 上面。它会根据查询的键值从本地状态后端取得状态，然后将其返回给提交请求的客户端代理

![可查询式状态服务架构](https://cdn.jsdelivr.net/gh/qtxsnwwb/image-hosting@master/20210522/可查询式状态服务架构.2xefargn6ji0.png)

* 为了在 Flink 设置中启用可查询式状态服务（即在 TaskManager 中启动客户端代理和服务器线程），需要将 *flink-queryable-state-runtime* JAR 文件放到 TaskManager 进程的 Classpath 中。为此，可以直接将 JAR 文件从 Flink 安装路径的 *./opt* 目录拷贝到 *./lib* 目录中。如果 Classpath 中存在该 JAR 文件，可查询式状态的线程就会自动启动，以响应客户端请求
* 正确配置后，会在 TaskManager 的日志中看到以下信息：

```
Started the Queryable State Proxy Server @ ...
```

* 客户端代理和服务器所使用的端口以及一些额外的参数可以在 *./conf/flink-conf.yaml* 文件中配置

#### 对外暴露可查询式状态
* 实现一个支持可查询式状态的流式应用非常简单。要做的就是定义一个具有键值分区状态的函数，然后在获取状态引用之前调用 StateDescriptor 的 setQueryable(String) 方法。这样，目标状态就变为可查询的了

```scala
// 将键值分区状态设置为可查询的
override def open(parameters: Configuration): Unit = {
	// 创建状态描述符
	val lastTempDescriptor = new ValueStateDescriptor[Double]("lastTemp", classOf[Double])
	// 启用可查询式状态并设置其外部标识符
	lastTempDescriptor.setQueryable("lastTemperature")
	// 获得状态引用
	lastTempState = getRuntimeContext.getState[Double](lastTempDescriptor)
}
```

#### 从外部系统查询状态
* 所有基于 JVM 的应用都可以使用 QueryableStateClient 对运行中 Flink 的可查询式状态进行查询。这个类由 *flink-queryable-state-client-java* 依赖提供
* 为了初始化 QueryableStateClient，需要提供任意一个 TaskManager 的主机名以及其上可查询式状态客户端代理的监听端口。客户端代理的默认监听端口是 9067，可以在 *./conf/flink-conf.yaml* 文件中对它进行配置：

```
val client: QueryableStateClient = new QueryableStateClient(tmHostname, proxyPort)
```

* 在得到一个状态客户端对象之后，就可以调用它的 getKvState() 方法来查询应用的状态。该方法需要接收几个参数，包括：当前运行应用的 JobID，状态标识符，所需状态的键值，键值的 TypeInformation 以及可查询式状态的 StateDescriptor。其中的 JobID 可以通过 REST API、Web UI 或者日志文件得到。getKvState() 方法返回一个 CompletableFuture[S]，其中 S 是状态类得到。因此，客户端可以同时发出多个异步请求并等待期返回结果




## 7. 读写外部系统
### 应用的一致性保障
* Flink 的检查点和恢复机制结合可重置的数据源连接器能够确保应用不会丢失数据。但由于在前一次成功的检查点（故障恢复时的回退位置）后发出的数据会被再次发送，所以应用可能会发出两次结果。因此，**可重置的数据源以及 Flink 的恢复机制虽然可以为应用状态提供精确一次的一致性保障，但无法提供端到端的精确一次保障**
* 应用若想提供端到端的精确一次性保障，需要一些特殊的数据汇连接器。根据情况不同，这些连接器可以使用两种技术来实现精确一次保障：
    - **幂等性写**
        - 幂等操作可以多次执行，但只会引起一次改变。例如，将相同的键值对插入一个哈希映射就是一个幂等操作，因为在首次将键值对插入映射中后，目标键值对就已经存在，此时无论该操作重复几次都不会改变这个映射。
        - 追加操作就不是幂等的，因为多次追加某个元素会导致它出现多次
        - 幂等性写操作可以在不改变结果的前提下多次执行，因此可以在一定程度上减轻 Flink 检查点机制所带来的重复结果的影响
    - **事务性写**
        - 基本思路是只有在上次成功的检查点之前计算的结果才会被写入外部数据汇系统。该行为可以提供端到端的精确一次保障。因为在发生故障后，应用会被重置到上一个检查点，而接收系统不会收到任何在该检查点之后生成的结果
        - 通过只在检查点完成后写入数据，事务性写虽然不会像幂等性写那样出现重放过程中的不一致现象，但会增加一定延迟，因为结果只有在检查点完成后才对外可见
        - Flink 提供了两个构件来实现事务性的数据汇连接器：WAL（write-ahead log，写前日志）数据汇和 2PC（two-phase commit，两阶段提交）数据汇
    
#### WAL 数据汇
* WAL 数据汇会将所有结果记录写入应用状态，并在收到检查点完成通知后将它们发送到数据汇系统
* 然而，WAL 数据汇无法 100% 提供精确一次保障，此外还会导致应用状态大小增加以及接收系统需要处理一次次的“波峰式”写入

#### 2PC 数据汇
* 2PC 数据汇需要数据汇系统支持事务或提供可用来模拟事务的构件
* 每次生成检查点，数据汇都会开启一次事务并将全部收到的记录附加到该事务中，即将它们写入接收系统但先不提交。直到收到检查点完成通知后，数据汇才会通过提交事务真正写入结果。该机制需要数据汇在故障恢复后能够提交某检查点完成前开启的事务
* 2PC 协议需要基于 Flink 现有的检查点机制来完成。检查点分隔符可认为是开启新事务的通知，所有算子完成各自检查点的通知可看做是提交股票，而来自 JobManager 的检查点创建成功的消息其实是提交事务的指令。和 WAL 数据汇不同，2PC 数据汇可依赖数据汇系统和数据汇的实现来完成精确一次的结果输出
* 此外，2PC 数据汇可以持续平稳地将记录写入接收系统，而不会像 WAL 数据汇那样经历周期性的“波峰式”写入




## 8. Flink 和流式应用运维
### 运行并管理流式应用
* 为了对主进程、工作进程以及应用进行监控，Flink 对外公开了以下接口：
    1. 一个用于提交和控制应用的命令行客户端工具
    2. 一套用于命令行客户端和 Web UI 的底层 REST API。它可以供用户或脚本访问，还可用于获取所有系统及应用指标并作为提交和管理应用的服务端点
    3. 一个用于提交有关 Flink 集群和当前运行应用详细信息及指标的 Web UI。它同时提供了基本的应用提交和管理功能
