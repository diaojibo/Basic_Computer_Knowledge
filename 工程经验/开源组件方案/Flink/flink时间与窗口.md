## flink时间与窗口

对于flink进行流实时计算，必定涉及到时间

在flink中定义了三类时间：

- 事件时间（Event Time）：即事件实际发生的时间。
- 处理时间（Processing Time）：事件被处理的时间。
- 进入时间（Ingestion Time）：事件进入流处理框架的时间

![img](image/3267951-5374e8f6fb1d2c8e)

上图很好的概括了这一层关系