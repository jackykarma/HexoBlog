---
title: UML类图
date: 2022-07-24 21:22:53
tags: UML类图
categories: UML建模
---

[plantUML类图](https://plantuml.com/zh/class-diagram)

类图（Class Diagram）: 类图是面向对象系统建模中最常用和最重要的图，是定义其它图的基础。类图主要是用来显示系统中的类、接口以及它们之间的静态结构和关系的一种静态模型。

类图的3个基本组件：类名、属性、方法。

## 一、类之间关系

1. 泛化（Generalization)
2. 实现（Realization）
3. 关联（Association)
4. 聚合（Aggregation）
5. 组合(Composition)
6. 依赖(Dependency)

PlantUML箭头线的绘制方法：

```js
<|：空心箭头
<：实心箭头
--：实线 // 通过增加-可以增加线的长度
..：虚线
// 方向
2个--或..改为一个-或.:从垂直排列改为水平方向
```

各种关系的强弱顺序：泛化 = 实现 \> 组合 \> 聚合 \> 关联 \> 依赖。

### 1.1. 泛化

【泛化关系】：是一种继承关系，表示一般与特殊的关系，它指定了子类如何特化父类的所有特征和行为。例如：老虎是动物的一种，即有老虎的特性也有动物的共性。
【箭头指向】：带三角箭头的实线，箭头指向父类。

```js
@startuml

class Animal{}
class Tiger{}
' 仅仅是上下摆放位置的差异
' Tiger --|> Animal
Animal <|-- Tiger

@enduml
```

![](%E6%88%AA%E5%B1%8F2022-07-24%2015.52.34.png)

### 1.2. 实现（Realization）

【实现关系】：是一种类与接口的关系，表示类是接口所有特征和行为的实现.
【箭头指向】：带三角箭头的虚线，箭头指向接口。

![](%E6%88%AA%E5%B1%8F2022-07-24%2017.10.22.png)

```js
@startuml

interface IShape {}
Class Circle {}
IShape <|.. Circle

@enduml
```

### 1.3. 关联（Association)

1. 关联关系：是一种拥有的关系，它使一个类知道另一个类的属性和方法；如：老师与  学生，丈夫与妻子关联可以是双向的，也可以是单向的。双向的关联可以有两个箭头或者没有箭头，单向的关联有一个箭头。
2. 代码体现：成员变量
3. 箭头及指向：带普通箭头的实心线，指向被拥有者。

```js
@startuml

class Teacher {
  -students:Student
}

class Student {
  -courses: Course
}

class Course {}

' Teacher "n" -- "n" Student
' Student "1" --> "n" Course
' 问题：如何增加连接线的长度, 多增加横杠线-
Teacher "n" - "n" Student
Student "1" -> "n" Course

@enduml
```

![](%E6%88%AA%E5%B1%8F2022-07-24%2017.26.59.png)

类之间的静态关系具有多重性：

1. 一对一关联
2. 一对多关联
3. 规定数值关联
4. 可选关联
5. 多对多关联

上图中，老师与学生是双向关联，老师有多名学生，学生也可能有多名老师。但学生与某课程间的关系为单向关联，一名学生可能要上多门课程，课程是个抽象的东西他不拥有学生。

### 1.4. 聚合（Aggregation）

1. 聚合关系：是整体与部分的关系，且部分可以离开整体而单独存在。**聚合关系是关联关系的一种**，是强的关联关系；**关联和聚合在语法上无法区分**，必须考察具体的逻辑关系。
2. 代码体现：成员变量
3. 箭头及指向：带空心菱形的实心线，菱形指向整体；

```js
@startuml

class Car {
  ' 轮胎
  -tyre: Tyre
  -engine: Engine 
}

class Tyre {}
class Engine {}

Car "1" o--> "1" Engine
Car "1" o--> "4" Tyre

@enduml
```

如车和轮胎是整体和部分的关系，轮胎离开车仍然可以存在。如雁群和大雁。

### 1.5. 组合(Composition)

1. 组合关系：是整体与部分的关系，但**部分不能离开整体而单独存在**。如公司和部门是整体和部分的关系，没有公司就不存在部门；如大雁和其翅膀。组合关系是关联关系的一种，是比聚合关系还要强的关系，它要求普通的聚合关系中代表整体的对象负责代表部分的对象的生命周期。
2. 代码体现：成员变量
3. 箭头及指向：带实心菱形的实线，菱形指向整体。

```js
@startuml

class Company {
  -department: Department
}

class Department {}
Company "1" *--> "n" Department
@enduml
```

![](%E6%88%AA%E5%B1%8F2022-07-24%2018.01.21.png)

### 1.6. 依赖(Dependency)

1. 依赖关系：是一种使用的关系，即一个类的实现需要另一个类的协助，所以要尽量不使用双向的互相依赖；
2. 代码表现：局部变量、方法的参数或者对静态方法的调用；
3. 箭头及指向：带箭头的虚线，指向被使用者；

```js
@startuml

class ModernPerson {
  void useComputer(Computer computer)
}

class Computer {}
ModernPerson ..> Computer

@enduml
```

![](%E6%88%AA%E5%B1%8F2022-07-24%2018.34.05.png)

## 二、类之间作用关系

在标签的开始或结束位置添加\< 或 \>以表明是哪个对象作用到哪个对象上。

```js
@startuml

class 汽车

发动机 - 汽车 : 驱动 >
汽车 *- 轮子 : 拥有 4 >
汽车 -- 人 : < 所属

@enduml
```

![](%E6%88%AA%E5%B1%8F2022-07-24%2018.44.19.png)

## 三、类属性和方法

定义可访问性：

![](%E6%88%AA%E5%B1%8F2022-07-24%2018.54.35.png)

```js
@startuml

class ConfigNode {
  -id:String
  -children:List<ConfigNode>
  ' 显示指明是field
  -{field}parent:ConfigNode?
  +getParent():ConfigNode    
  ' 显示指明是method
  +{method}getChildrent():List<ConfigNode)
}

@enduml
```

![](%E6%88%AA%E5%B1%8F2022-07-24%2018.52.47.png)

抽象与静态：通过修饰符`{static}`或者`{abstract}`，可以定义静态或者抽象的方法或者属性。

PlantUML默认自动将方法和属性重新分组，你可以自己定义分隔符来重排方法和属性，下面的分隔符都是可用的：`--..==__.`

```js
@startuml
class Foo1 {
  You can use
  several lines
  ..
  as you want
  and group
  ==
  things together.
  __
  You can have as many groups
  as you want
  --
  End of class
}

class User {
  .. Simple Getter ..
  + getName()
  + getAddress()
  .. Some setter ..
  + setName()
  __ private data __
  int age
  -- encrypted --
  String password
}
@enduml
```

![](%E6%88%AA%E5%B1%8F2022-07-24%2018.57.48.png)

## 四、布局

有时候默认布局并不完美...可以使用 together 关键词将某些类进行分组： 布局引擎会尝试将它们捆绑在一起。你也可以使用建立 hidden 链接的方式来强制布局（就是2个类实际没有关系，但是为了放在一块就用hidden连接）。通过这种方式可以调整类之间的摆放位置。

```js
@startuml

' 有时候默认布局并不完美...可以使用 together 关键词将某些类进行分组： 布局引擎会尝试将它们捆绑在一起
' 你也可以使用建立 hidden 链接的方式来强制布局（就是2个类实际没有关系，但是为了放在一块就用hidden连接）
class Bar1
class Bar2
together {
  class Together1
  class Together2
  class Together3
}
Together1 - Together2
Together2 - Together3
Together2 -[hidden]--> Bar1
Bar1 -[hidden]> Bar2

@enduml
```

![](%E6%88%AA%E5%B1%8F2022-07-24%2019.04.49.png)

### 4.1. 改变箭头方向

类之间默认采用两个破折号 -- 显示出垂直 方向的线. 要得到水平方向的可以像这样使用单破折号 (或者点):

```js
@startuml
教室 o- 学生
教室 *-- 椅子
@enduml
```

也可以通过改变倒置链接来改变方向:

```js
@startuml
学生 -o 教室
椅子 --* 教室
@enduml
```

也可通过在箭头内部使用关键字， 例如left, right, up 或者 down，来改变方向。您可以使用缩写形式来表示方向，第一个字符（例如，-d- 而不是 -down-) 或前两个字符 (-do-)。您不应滥用此功能：Graphviz 通常无需调整即可提供良好的结果。

```js
@startuml
foo -left-> dummyLeft
foo -right-> dummyRight
foo -up-> dummyUp
foo -down-> dummyDown
@enduml
```

![](%E6%88%AA%E5%B1%8F2022-07-24%2019.24.04.png)

## 五、关联类或关系类

可以在定义了两个类之间的关系后定义一个 关系类 association class 例如:

```js
@startuml
class Student {
  Name
}
Student "0..*" - "1..*" Course
(Student, Course) .. Enrollment

class Enrollment {
  drop()
  cancel()
}
@enduml
```

![](%E6%88%AA%E5%B1%8F2022-07-24%2019.25.57.png)


## 六、绘图样式规范

```js
' 必须放在class定义之前，否则不会生效
skinparam class {
  ' 定义箭头的颜色
  ArrowColor ForestGreen
  ' 定义箭头上文字颜色
  ArrowFontColor ForestGreen
}

skinparam ClassBackgroundColor lightgray
'可单独设置Header的背景色，却不能设置类名称的颜色
skinparam ClassHeaderBackgroundColor ForestGreen
skinparam ClassFontColor white
skinparam ClassBorderThickness 0
skinparam ClassFontName Courier
skinparam ClassFontSize 14
skinparam ClassArrowColor ForestGreen
skinparam stereotypeCBackgroundColor YellowGreen
' fixme:ClassAttributeFontColor属性的样式为何会覆盖Header（类名称）不合理啊
' skinparam ClassAttributeFontSize 30
' skinparam ClassAttributeFontColor red
```

示例代码：

```js
@startuml

' 必须放在class定义之前，否则不会生效
skinparam class {
  ' 定义箭头的颜色
  ArrowColor ForestGreen
  ' 定义箭头上文字颜色
  ArrowFontColor ForestGreen
}

skinparam ClassBackgroundColor lightgray
'可单独设置Header的背景色，却不能设置类名称的颜色
skinparam ClassHeaderBackgroundColor ForestGreen
skinparam ClassFontColor white
skinparam ClassBorderThickness 0
skinparam ClassFontName Courier
skinparam ClassFontSize 14
skinparam ClassArrowColor ForestGreen
skinparam stereotypeCBackgroundColor YellowGreen
' fixme:ClassAttributeFontColor属性的样式为何会覆盖Header（类名称）不合理啊
' skinparam ClassAttributeFontSize 30
' skinparam ClassAttributeFontColor red

class Test {
  ' 这样写有点麻烦
  +<color:ForestGreen>isOpen:Boolean</color>
  +<color:ForestGreen>isOpen:Boolean</color>
  +<color:ForestGreen>isOpen:Boolean</color>
  +<color:ForestGreen>isOpen:Boolean</color>
  +<color:ForestGreen>isOpen:Boolean</color>
  +<color:ForestGreen>isOpen:Boolean</color>
}

Class Jacky {}

' 依赖关系
Test ..> "1" Jacky

' 泛化关系：继承
Test "1" --|> "many" Alis:一对多
' 组合关系
Test "1" --* "3" Grace:一对多
' 聚合关系
Test "1" --o "1" Moorning:一对一

interface A {
  +<color:ForestGreen>isOpen:Boolean</color>
  +<color:ForestGreen>isOpen:Boolean</color>
  +<color:ForestGreen>isOpen:Boolean</color>
  +<color:ForestGreen>isOpen:Boolean</color>
  +<color:ForestGreen>isOpen:Boolean</color>
  +<color:ForestGreen>isOpen:Boolean</color> 
}

' 有时候默认布局并不完美...可以使用 together 关键词将某些类进行分组： 布局引擎会尝试将它们捆绑在一起
' 你也可以使用建立 hidden 链接的方式来强制布局（就是2个类实际没有关系，但是为了放在一块就用hidden连接）
class Bar1
class Bar2
together {
  class Together1
  class Together2
  class Together3
}
Together1 - Together2
Together2 - Together3
Together2 -[hidden]--> Bar1
Bar1 -[hidden]> Bar2

@enduml
```


![](%E6%88%AA%E5%B1%8F2022-07-24%2015.39.00.png)





