# GenStage

在本章节中, 我们将研究 GenStage, 它的作用是什么, 以及我们如何在应用程序中如何利用它.

<!-- TOC -->

- [GenStage](#genstage)
    - [介绍](#%E4%BB%8B%E7%BB%8D)
    - [消费者和生产者](#%E6%B6%88%E8%B4%B9%E8%80%85%E5%92%8C%E7%94%9F%E4%BA%A7%E8%80%85)
    - [入门](#%E5%85%A5%E9%97%A8)
    - [生产者](#%E7%94%9F%E4%BA%A7%E8%80%85)
    - [生产者-消费者](#%E7%94%9F%E4%BA%A7%E8%80%85-%E6%B6%88%E8%B4%B9%E8%80%85)
    - [消费者](#%E6%B6%88%E8%B4%B9%E8%80%85)
    - [集成](#%E9%9B%86%E6%88%90)
    - [多个生产者或者消费者](#%E5%A4%9A%E4%B8%AA%E7%94%9F%E4%BA%A7%E8%80%85%E6%88%96%E8%80%85%E6%B6%88%E8%B4%B9%E8%80%85)
    - [用例](#%E7%94%A8%E4%BE%8B)

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

对于我们的应用来说, 我们会使用到全部三种 `GenStage` 角色. 生产者将负责计数和生成数字. 使用生产者-消费者来过滤出偶数, 然后对下游的需求做出响应. 最后, 我们将构建一个消费者来显示剩余数字.

我们将从生成具有 supervision tree 的项目开始:
```shell
$ mix new genstage_example --sup
$ cd genstage_example
```

更新 `mix.exs` 中的依赖, 其中添加 `gen_stage`:
```elixir
defp deps do
  [
    {:gen_stage, "~> 0.11"},
  ]
end
```

在进一步研究前, 我们需要拉取并编译依赖:
```shell
$ mix do deps.get, compile
```

现在我们准备构建生产者!

## 生产者
我们 `GenStage` 应用程序的第一步是创建生产者, 正如我们之前讨论的那样, 我们想要创建一个生产者, 使其生成数字流. 让我们先创建生产者对应的文件:
```shell
$ mkdir lib/genstage_example
$ touch lib/genstage_example/producer.ex
```

现在我们添加代码:
```elixir
defmodule GenstageExample.Producer do
  use GenStage

  def start_link(initial \\ 0) do
    GenStage.start_link(__MODULE__, initial, name: __MODULE__)
  end

  def init(counter), do: {:producer, counter}

  def handle_demand(demand, state) do
    events = Enum.to_list(state..(state + demand - 1))
    {:noreply, events, state + demand}
  end
end
```

这里需要注意的两个部分是 `init/1` 和 `handle_demand/2`. 在 `init/1` 中, 我们设置初始状态, 就像我们在 `GenServer` 中做的那样, 但是更重要的是, 我们将其标记为 producer. 来自函数 `init/1` 的响应是由 `GenStage` 依赖什么来分类我们的进程还决定的.

函数 `handle_demand/2` 是主要定义生产者的地方. 其必须由所有的 `GenStage` 实现. 在这里, 我们返回一组消费者所需求的数字, 同时增加计数器计数. 来自消费者的需求, 上面代码中的 `demand`, 预设为一个对应他们可处理事件数量的整数了; 其默认值为1000.

## 生产者-消费者

现在, 我们拥有了一个数字生成生产者, 接着让我们研究下生产者-消费者. 我们希望从生产者请求数字, 过滤技术, 并响应需求.
```shell
$ touch lib/genstage_example/producer_consumer.ex
```

更新文件, 像示例代码这样:
```elixir
defmodule GenstageExample.ProducerConsumer do
  use GenStage

  require Integer

  def start_link do
    GenStage.start_link(__MODULE__, :state_doesnt_matter, name: __MODULE__)
  end

  def init(state) do
    {:producer_consumer, state, subscribe_to: [GenstageExample.Producer]}
  end

  def handle_events(events, _from, state) do
    numbers =
      events
      |> Enum.filter(&Integer.is_even/1)

    {:noreply, numbers, state}
  end
end
```

你可能注意到我们的生产者-消费者在 `init/1` 中引入了一个新的选项值, 以及一个新的方法: `handle_events/3`. 通过 `subscribe_to` 选项, 我们指定 `GenStage` 与特定的生产者进行通信.

`handle_events/3` 函数是我们的主力, 在这里, 我们接收传入事件, 处理他们, 并返回转换后的集. 正如我们将会看到的, 消费者的实现与之类似, 但是主要的不同在于 `handle_events/3` 的返回和它的使用方式. 当我们标记进程为生产者-消费者时, 元组的第二个参数, 在这个例子中的 `numbers`, 是用于满足下游消费者需求的. 在消费者中, 这个值是被丢弃的.

## 消费者
最后, 但并不是无关紧要的是消费者, 让我们开始吧:
```shell
$ touch lib/genstage_example/consumer.ex
```

由于消费者和生产者-消费者很相似, 我们的代码看起来不会有特别大的不同:

```elixir
defmodule GenstageExample.Consumer do
  use GenStage

  def start_link do
    GenStage.start_link(__MODULE__, :state_doesnt_matter)
  end

  def init(state) do
    {:consumer, state, subscribe_to: [GenstageExample.ProducerConsumer]}
  end

  def handle_events(events, _from, state) do
    for event <- events do
      IO.inspect({self(), event, state})
    end

    # As a consumer we never emit events
    {:noreply, [], state}
  end
end
```

正如我们上一节提到的, 我们的消费者不发出事件, 所以我们元组的第二个值将被丢弃掉.

## 集成

现在, 我们已经构建了生产者, 生产者-消费者以及消费者, 我已经准备好将他们连接在一起.

首先, 我们打开 `lib/genstage_example/application.ex`, 并且添加新的进程至 supervisor tree:
```elixir
def start(_type, _args) do
  import Supervisor.Spec, warn: false

  children = [
    worker(GenstageExample.Producer, [0]),
    worker(GenstageExample.ProducerConsumer, []),
    worker(GenstageExample.Consumer, [])
  ]

  opts = [strategy: :one_for_one, name: GenstageExample.Supervisor]
  Supervisor.start_link(children, opts)
end
```

如果所做的一切正确, 我们可以执行项目, 并且应该可以看到所有工作正常:
```shell
$ mix run --no-halt
{#PID<0.109.0>, 2, :state_doesnt_matter}
{#PID<0.109.0>, 4, :state_doesnt_matter}
{#PID<0.109.0>, 6, :state_doesnt_matter}
...
{#PID<0.109.0>, 229062, :state_doesnt_matter}
{#PID<0.109.0>, 229064, :state_doesnt_matter}
{#PID<0.109.0>, 229066, :state_doesnt_matter}
```

我们做到了! 正如我们预期的一样, 应用程序只输出偶数, 并且工作起来**非常快**.

在此, 我们拥有一个工作管道, 生产者生成数字, 生产者-消费者丢弃奇数, 消费者显示结果并继续这个流程.

## 多个生产者或者消费者

在介绍中, 我们有提到可以存在多个生产者或消费者, 让我们看看这一点.

如果我们检查示例代码的 `IO.inspect/1` 的输出, 我们可以看到每一个事件是通过一个 PID 处理的. 让我们对 `lib/genstage_example/application.ex` 做些调整, 使其适配多个 worker:
```elixir
children = [
  worker(GenstageExample.Producer, [0]),
  worker(GenstageExample.ProducerConsumer, []),
  worker(GenstageExample.Consumer, [], id: 1),
  worker(GenstageExample.Consumer, [], id: 2)
]
```

现在我们配置了两个消费者, 让我们看看执行应用程序后, 我们看到的输出:
```elixir
$ mix run --no-halt
{#PID<0.120.0>, 2, :state_doesnt_matter}
{#PID<0.121.0>, 4, :state_doesnt_matter}
{#PID<0.120.0>, 6, :state_doesnt_matter}
{#PID<0.120.0>, 8, :state_doesnt_matter}
...
{#PID<0.120.0>, 86478, :state_doesnt_matter}
{#PID<0.121.0>, 87338, :state_doesnt_matter}
{#PID<0.120.0>, 86480, :state_doesnt_matter}
{#PID<0.120.0>, 86482, :state_doesnt_matter}
```

正如你看到的那样, 通过简单的添加一行代码, 并且指定消费者的 ID, 现在我们有多个 PID.

## 用例
现在, 我们已经介绍了 `GenStage`, 并且构建了第一个示例应用程序, `GenStage` 有什么现实中的用例吗?

* 数据转换管道 - 生产者不是简单的数字生成器, 我们可以从数据库或其他的源, 例如 Apache 的 Kafka 生成事件. 通过组合生产者-消费者和消费者, 我们可以处理, 排序, 分类以及当可用时, 存储指标.
* 工作队列 - 由于事件可以是任意类型, 我们可以生产由一系列消费者来完成的工作.
* 事件处理 - 类似于数据管道, 我们可以接受, 处理, 排序以及执行由来源事件实时触发动作.

以上只是 `GenStage` 可以实现的**几种**工作而已.
