# GenStage

在本章节中, 我们将研究 GenStage, 它的作用是什么, 以及我们如何在应用程序中如何利用它.

<!-- TOC -->

- [GenStage](#genstage)
    - [介绍](#%E4%BB%8B%E7%BB%8D)
    - [消费者和生产者](#%E6%B6%88%E8%B4%B9%E8%80%85%E5%92%8C%E7%94%9F%E4%BA%A7%E8%80%85)
    - [入门](#%E5%85%A5%E9%97%A8)

<!-- /TOC -->

## 介绍

那么, 什么是 GenStage? 从官方文档中, 它是一个"elixir 的一种规范以及工作流", 但是, 这对我们来说意味什么呢?

这意味着, GenStage 提供了一种方式, 使我们可以在隔离的进程中, 定义一种通过独立的步骤(或者阶段)组成的工作流; 如果你在此前使用过管道, 那其中的一些概念应该比较熟悉.

为了更好理解其的工作原理, 让我们看一个简单的生产者-消费者流程:
```
[A] -> [B] -> [C]
```

在上面的示例中, 我们有三个阶段: `A` 是生产者, `B` 是生产者-消费者, 以及 `C` 是消费者. `A` 产生值供 `B` 消费, `B` 执行一些操作, 返回一个新的值, 并被消费者 `C` 接收; `stage` 的角色很重要, 我们会在下一章节探讨.

虽然我们的例子是一对一的生产者至消费者, 但是任何特定的 `stage` 都可能有多个生产者和多个消费者.

为了更好的说明这些概念, 我们将使用 `GenStage` 构建一个管道, 但是首先, 让我们更深入探索一下 `GenStage` 中的每个角色.

## 消费者和生产者

正如我们看到的, stage 中的角色很重要, `GenStage` 规范承认以下三个角色:

* `:producer` - 来源. 生产者等待消费者的需求, 并响应所请求的事件.
* `:producer_consumer` - 既是来源也是消费者. 生产者-消费者既可以响应其他消费者的需求, 也可以向生产者请求事件.
* `:consumer` - 消费者. 消费者请求并接收来自生产者的数据.

注意, 我们的生产者**等待**需求? 通过 `GenStage`, 我们的消费者向上游发送需求, 并处理来自生产者的数据. 这种促进机制称为背压. 当消费者繁忙时, 背压使生产者承受的压力不会过大.

现在, 我们已经介绍过了 `GenStage` 中的角色, 下面开始构建我们的应用程序.

## 入门

在示例中, 我们将构建一个生成数字的 `GenStage` 应用程序, 其整理偶数, 并将其最终打印出来.

对于我们的应用来说, 我们会使用到全部三种 `GenStage` 角色. 生产者将负责计数和生成数字. 使用生产者-消费者来过滤出偶数, 然后对下游的需求做出响应. 最后, 我们将构建一个消费者来现实

For our application we’ll use all three GenStage roles. Our producer will be responsible for counting and emitting numbers. We’ll use a producer-consumer to filter out only the even numbers and later respond to demand from downstream. Last we’ll build a consumer to display the remaining numbers for us.