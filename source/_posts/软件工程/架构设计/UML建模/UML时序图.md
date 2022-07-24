---
title: UML时序图
date: 2022-07-24 21:22:53
tags: UML时序图
categories: 软件工程
---

[plantUML时序图](https://plantuml.com/zh/sequence-diagram)

时序图（Sequence Diagram）是显示对象之间交互的图，这些对象是按时间顺序排列的。顺序图中显示的是参与交互的对象及其对象之间消息交互的顺序。时序图中包括的建模元素主要有：对象（Actor）、生命线（Lifeline）、控制焦点（Focus of control）、消息（Message）等等。

## 一、时序图的组成元素

1. 角色（Actor）：系统角色，可以是人、机器、其他系统、子系统。一般是时序图时间顺序的发起点。
2. 对象（Object）：对象，在不同的构图软件中，有不同的角色。比如在常用的绘图软件plantuml中，可以有 participant、queue、database 等。该角色一般是描述一个对象、一个模块、一个系统（**时序图也分粒度和层级**）。对象位于时序图的顶部，以一个矩形表示。
3. 生命线（LifeLine）：时序图中每个对象和底部中心都有一条垂直的虚线，这就是对象的生命线（时间线），以一条垂直的虚线表示，对象间的消息存在与二条虚线间。
4. 激活期（Activation）：又名控制焦点，它代表时序图在对象时间线上某段时期执行的操作，以一个很窄的矩形表示。
5. 消息（Message）：表示对象之间发送的信息。消息分为四种类型。
	- 同步消息(Synchronous Message)消息的发送者把控制传递给消息的接收者，然后停止活动，等待消息的接收者放弃或者返回控制。用来表示同步的意义，以一条实线和实心箭头表示。
	- 异步消息(Asynchronous Message)消息发送者通过消息把信号传递给消息的接收者，然后继续自己的活动，不等待接受者返回消息或者控制。异步消息的接收者和发送者是并发工作的，以一条实线和空心箭头表示。
	- 返回消息(Return Message)返回消息表示从过程调用返回，以小于号和虚线表示。
	- 自关联消息 表示方法的自身调用或者一个对象内的一个方法调用另外一个方法。
6. 组合片段：用来解决交互执行的条件和方式，它允许在序列图中直接表示逻辑组件，通过指定条件、子进程的应用区域，为任何生命线的任何部分定义特殊条件和子进程。组合片段有十几种，最常用的是 ALT 片段，同属来说就是 if-else 的条件判断组合。

## 二、基本绘制

关键字activate和deactivate用来表示参与者的生命活动。一旦参与者被激活，它的生命线就会显示出来。destroy表示一个参与者的生命线的终结。

```js
@startuml
skinparam SequenceReferenceFontColor yellow
skinparam sequence {
  ArrowColor ForestGreen
  LifeLineBorderColor ForestGreen
  LifeLineBackgroundColor ForestGreen
  
  ParticipantBackgroundColor ForestGreen
  ParticipantBorderColor ForestGreen
  ParticipantFontColor white

  ActorBackgroundColor ForestGreen
  ActorBorderColor ForestGreen
  ActorFontColor ForestGreen
}
' 消息自动编号
autonumber
' 自动激活打开
' autoactive on

actor User
' 同步消息
User -> A: DoWork
activate A

A -> B: << createRequest >>
activate B

' 异步消息
' 激活子函数
B ->> B: doSth
activate B
' 子函数执行完后回来
' return
deactivate B

B -> C: DoWork
activate C
C --> B: WorkDone
destroy C

B --> A: RequestCreated
deactivate B

A -> User: Done
deactivate A

@enduml
```

![](%E6%88%AA%E5%B1%8F2022-07-24%2020.47.12.png)

message的文本颜色修改，目前只有很傻的方式，无法统一设置：

```js
<color:Red>认证接受</color>
```

## 三、组合片段

| 片段类型     | 名称  | 说明                                                                                                                                 |
| -------- | --- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Opt      | 选项  | 包含一个可能发生或可能不发生的序列。可以在临界中指定序列发 生的条件。                                                                                                |
| Alt      | 抉择  | 包含一个片段列表,这些片段包含备选消息序列。在任何场合 下只发生一个序列。 可以在每个片段中设置-个临界来指示该片段可以运行的条件。else 的临界指示其他任何临界都不为True时应运行的片段。如果所有临界都为 False并且没有else ,则不执行任何片段。 |
| Loop     | 循环  | 片段重复一定次数。 可以在临界中指示片段重复的条件。Loop组合片段具有“Min"和"Max"属性，它们指示片段可以重复的最小和最大次数。默认值是无限制。                                                      |
| Break    | 中断  | 如果执行此片段，则放弃序列的其余部分。可以使用临界来指示发生中断的条                                                                                                 |
| Par      | 并行  | 并行处理。片段中的事件可以交错。                                                                                                                   |
| Critical | 关键  | 用在Par或Seq片段中。指示此片段中的消息不得与其他消息交错。                                                                                                   |
| Seq      | 弱顺序 | 有两个或更多操作数片段。涉及同一生命线的消息必须以片段的顺序发生。如果消息涉及的生命线不同,来自不同片段的消息可能会并行交错。                                                                    |
| Strict   | 强顺序 | 有两个或更多操作数片段。这些片段必须按给定顺序发生。                                                                                                         |
| Consider | 考虑  | 指定此片段描述的消息列表。其他消息可发生在运行的系统中 ,但对此描述来说意义不大                                                                                           |
| Ignore   | 忽略  | 此片段未描述的消息列表。这些消息可发生在运行的系统中 ,但对此描述来说意义不大。                                                                                           |
| Assert   | 断言  | 操作数片段指定唯一有效的序列。 通常用在Consider或Ignore片段中。                                                                                            |
| Neg      | 否定  | 此片段中显示的序列不得发生。通常用在Consider或Ignore月段中。                                                                                              |

```js
@startuml

skinparam SequenceReferenceFontColor yellow
skinparam sequence {
  ArrowColor ForestGreen
  LifeLineBorderColor ForestGreen
  LifeLineBackgroundColor ForestGreen
  
  ParticipantBackgroundColor ForestGreen
  ParticipantBorderColor ForestGreen
  ParticipantFontColor white

  ActorBackgroundColor ForestGreen
  ActorBorderColor ForestGreen
  ActorFontColor ForestGreen

  ' GroupBackgroundColor ForestGreen
  ' GroupBodyBackgroundColor ForestGreen
  GroupFontColor white
  GroupHeaderFontColor ForestGreen
  GroupBorderColor ForestGreen
}
' 消息自动编号 改变编号的字体颜色
autonumber "<font color=ForestGreen>"
' 自动激活打开
' autoactive on

Alice -> Bob: <color:ForestGreen>认证请求</color>

alt 成功情况
    Bob -> Alice: <color:ForestGreen>认证接受</color>
else 某种失败情况
    Bob -> Alice: <color:ForestGreen>认证失败</color>
    ' 分组
    group 我自己的标签
    Alice -> Log : <color:ForestGreen>开始记录攻击日志</color>
        loop 1000次
            Alice -> Bob: <color:ForestGreen>DNS 攻击</color>
        end
    Alice -> Log : <color:ForestGreen>结束记录攻击日志</color>
    end
else 另一种失败
   Bob -> Alice: <color:ForestGreen>请重复</color>
end

@enduml
```

![](%E6%88%AA%E5%B1%8F2022-07-24%2021.18.43.png)

## 四、绘图样式规范

```js
@startuml

skinparam SequenceReferenceFontColor yellow
skinparam sequence {
  ArrowColor ForestGreen
  LifeLineBorderColor ForestGreen
  LifeLineBackgroundColor ForestGreen
  
  ParticipantBackgroundColor ForestGreen
  ParticipantBorderColor ForestGreen
  ParticipantFontColor white

  ActorBackgroundColor ForestGreen
  ActorBorderColor ForestGreen
  ActorFontColor ForestGreen

  ' GroupBackgroundColor ForestGreen
  ' GroupBodyBackgroundColor ForestGreen
  GroupFontColor white
  GroupHeaderFontColor ForestGreen
  GroupBorderColor ForestGreen
}
' 消息自动编号 改变编号的字体颜色
autonumber "<font color=ForestGreen>"
' 自动激活打开
' autoactive on
```
