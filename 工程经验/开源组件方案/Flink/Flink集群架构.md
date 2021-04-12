## flink集群架构

Flink集群架构依旧遵循 Master-Worker这样的一种集群架构模式

节点有两种进程组成，一个 *JobManager* 和一个或者多个 *TaskManager*。

- JobManager就是管理节点，管理整个集群的计算资源，job调度以及Checkpoint协调
- *TaskManager*（也称为 *worker*）执行作业流的 task，并且缓存和交换数据流。
- Client，本地执行应用main方法解析JobGraph对象，并最终将JobGraph提交到JobManager

![The processes involved in executing a Flink dataflow](image/processes.svg)

当 Flink 集群启动后，首先会启动一个 JobManger 和一个或多个的 TaskManager。由 Client 提交任务给 JobManager，JobManager 再调度任务到各个 TaskManager 去执行，然后 TaskManager 将心跳和统计信息汇报给 JobManager。TaskManager 之间以流的形式进行数据的传输。上述三者均为独立的 JVM 进程。

### Graph生成

Client在提交任务的时候，会通过Dag有向无环图来表达用户程序，生成一个执行流图。

这个执行流图其实是分为4层的，慢慢的转换为最终的物理执行图：

StreamGraph -> JobGraph -> ExecutionGraph -> 物理执行图。

- **StreamGraph：**是根据用户通过 Stream API 编写的代码生成的最初的图。用来表示程序的拓扑结构。
- **JobGraph：**StreamGraph经过优化后生成了 JobGraph，提交给 JobManager 的数据结构。主要的优化为，将多个符合条件的节点 chain 在一起作为一个节点，这样可以减少数据在节点之间流动所需要的序列化/反序列化/传输消耗。
- **ExecutionGraph：**JobManager 根据 JobGraph 生成ExecutionGraph。ExecutionGraph是JobGraph的并行化版本，是调度层最核心的数据结构。
- **物理执行图：**JobManager 根据 ExecutionGraph 对 Job 进行调度后，在各个TaskManager 上部署 Task 后形成的“图”，并不是一个具体的数据结构。

![img](image/TB1tA_GJFXXXXapXFXXXXXXXXXX)