## S1: What's software architecture

### Definition

### Explanation

* Module

* Component

* Mapping

## S2: Requirement and architecture

## S3: How to evaluate architecture

## S4: ADD

### Refine input for the next iterations

其实就是确定了引入的新元素的需求，包括：功能、质量和约束。使用的形式可以是： UseCase, Scenario, UtilityTree, Annotation。

### When to finish the iteration

* address all ASRs

* closely related to downstream teams

## S5: Roles of architcts

## S6: How to grow as an architect

### Knowledge

#### Foundation

algorithm/concurrency/distribution/storage/transaction/security/compiler/A.I.

#### Mthods

requirement/architecure/process/cases/patterns


#### Engineering

languages/libraries/tools/infrastructures

#### Domain


## TBD

* Document architecture

* Architecture and requirement

## Q & A

* 为什么产品文档中要包含架构？
> 用于展示系统解决了关注的质量属性

* 组件

是否看作是一类特殊的模块：可以被部署、运行、监控、停止的模块。除此之外，它具有其他类型模块同样的性质。
1) Component = Model+
2) Models are more easily dervied at certain circumstance than components
3) For a certain element, either decompoed to modules or components, do not decompose it to both.
设计过程中，是将元素同时分解成模块和组件，还是只分解成一类？如果是同时得到子模块和组件，后续设计时，是选择模块还是组件？

qa属性有的时候可以委托给模块，有的时候只能在组件之间分配，通过组件的交换完成。





