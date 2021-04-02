---
title: "遥测存档"
weight: 85
---

<!-- {{< synopsis-archiving >}} -->

保存设备的测量值，并以原始或处理过的形式使用。

<!--more-->

## 挑战

为了提供洞察力，IoT 解决方案需要支持对其产生的信息进行实时、批处理和预测分析。在利用历史数据时，每种分析模式都能获得更好的信息。_同时_，未来还有目前未被理解的分析方法，因此 IoT 解决方案必须以尽可能灵活的方式存档数据，以满足未来的需求.

## 解决方案

IoT 解决方案存储原始未处理过的传感器数据，并支持对这些原始样本进行有序的重放，从而确保业务可以获得最新且不断发展的洞察集合。存储和重放能力(store-and-replay)应使历史原始样本看起来几乎就像样本按正常时间序列到达一样。

下图中显示的"遥测存档"设计可以提供此功能。

![Telemetry Archiving Architecture](archiving.png)
([PPTx](/iot-atlas-patterns.pptx))

### 步骤说明

1. 该设备从远离 IoT 解决方案的环境中运行的传感器获得测量值。
2. 设备向包含测量值的主题`telemetry/deviceID`发布消息。此消息通过传输协议发送到服务器提供的协议端点。
3. 服务器可以对消息应用一条或多条[规则]({{< ref "/glossary/vocabulary#规则rule" >}})，从而对部分或全部[消息]({{< ref "/glossary/vocabulary#消息message" >}})的测量值数据执行细粒度的路由。这些规则会将消息至少分发到一个处理路径**`(4)`**和一个裸存储路径**`(5)`**。
4. 消息处理路径按照解决方案中其他组件的需求进行基本的计算，并存储处理后的结果。
5. 原始消息存储路径将原始消息保存下来，并支持对原始消息的有序重放
6. 未来组件可以从某个时间点开始读取原始消息，并且将消息重放至`telemetry/deviceID/replay`主题。解决方案会对重放消息进行必要的处理。

## 考虑点

实施此设计时，请考虑以下问题：

#### 重放记录是否是下游处理所必须的？

简单来说**是的**。大多数解决方案对这个问题的回答都应该是**肯定**的，因为通过如下这些方式，重放未经处理的原始传感器数据可以使得 IoT 解决方案支持 IoT 解决方案洞察力的演进：

- 基础计算的更新，支持对历史数据进行良好的分析
- 创建新的处理记录类型
- 实现新的预期之外的特性
- 从根本上创建新的客户数据视角

这种考虑的一个例子[如下](#重放数据示例)

#### 确保已保存消息的时间顺序是否是重要的？

如果 **是**: `待补充`

如果 **不是**: `待补充`

## 示例

#### 重放数据示例

一个物理站点正在进行电能([kWh](https://en.wikipedia.org/wiki/Kilowatt_hour))消耗的监控。每 30 秒对电能传感器进行采样，每分钟报告一次采样数据。原始未经处理的消息到达时，会被存储并且一个“15 分钟均值”过程会自动计算所消耗电能的 15 分钟平均值。计算结果会以记录的形式存储在解决方案的已处理记录库中。然后，这些新处理的记录将被其他分析流程和 IoT 解决方案的用户界面所使用。

后续解决方案的用户明确要求获得每 5 分钟电力消耗最大值的记录。为了实现这一功能，我们实施了一个新的“5 分钟最大值”过程。这个新过程会重放历史未处理记录，并在每 5 分钟历史间隔上计算最大值。一旦完成，每个计算结果将被存储为一个新的已处理记录类型。

如果不保留原始样本，解决方案将只能对实施了“5 分钟最大值”功能后到达的数据进行计算。最重要的是，如果没有原始样本，用户将无法使用"5 分钟最大值"这个功能去分析这个功能实现之前的物理站点的数据