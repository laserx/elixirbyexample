# 规格和类型

在本章节我们会学习 `@spec` 和 `@type` 的语法. 首先其是对编写文档进行更多的语法补充, 使其可以用工具进行分析. 其次是帮助我们编写更具有可读性和更易于理解的代码.

<!-- TOC -->

- [规格和类型](#%E8%A7%84%E6%A0%BC%E5%92%8C%E7%B1%BB%E5%9E%8B)
    - [介绍](#%E4%BB%8B%E7%BB%8D)
    - [规格](#%E8%A7%84%E6%A0%BC)
    - [自定义类型](#%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B1%BB%E5%9E%8B)
        - [定义自定义类型](#%E5%AE%9A%E4%B9%89%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B1%BB%E5%9E%8B)
        - [类型的文档](#%E7%B1%BB%E5%9E%8B%E7%9A%84%E6%96%87%E6%A1%A3)

<!-- /TOC -->

## 介绍

期望描述函数接口的需求并不罕见, 可以使用 [`@doc annotation`](documentations.md) 编写文档, 但是, 这只是为其他开发者提供相应的信息, 并不能在编译时检测代码. 为此, elixir 提供了 `@spec` 注释来描述函数的规格, 使其可以被编译器检验.

在某些情况下, 规格肯能变得非常臃肿且复杂, 如果你希望降低复杂度, 你可能会希望引入自定义类型定义, elixir 提供了 `@type` 注释来做到这点. 另一方面, elixir 依然是动态类型语言, 这意味着类型相关的所有信息都会被编译器忽略, 但是, 类型注释可以被其他工具所使用.

## 规格

如果你有 java 或者 ruby 的开发经验, 你可以视规格为接口. 规格定义了函数参数和返回值的类型.

为了定义输入和输出类型, 我们在函数定义之前使用 `@spec` 指令, 将函数名称作为 `params`, 接着是函数的参数列表, 最后用 `::` 将返回值类型与函数列表分开.

观察一下示例:
```elixir
@spec sum_product(integer) :: integer
def sum_product(a) do
  [1, 2, 3]
  |> Enum.map(fn el -> el * a end)
  |> Enum.sum()
end
```

看起来一切正常, 当我们调用时, 可以返回正确的结果. 但是, 函数 `Enum.sum` 返回 `number` 而不是我们在 `@spec` 中期望得到的 `integer`. 这可能成为错误的根源! 所以存在像 `Dialyzer` 这样的代码静态分析工具来帮助找到这种类型的错误, 我们会在后面的章节探讨它.

## 自定义类型

编写规格是很好的方式, 但是, 有时我们的函数处理的是比简单的数字或者集合更复杂的数据结构. 在 `@spec` 中定义的情况下, 其他的开发人员将很难理解, 或者难以调整它. 有时, 函数需要接收大量的参数或者返回复杂的数据, 一个长长的参数列表是代码中许多潜在不良气息之一. 在面向对象语言中, 就像 ruby 或者 java, 我们可以轻松的使用类, 来帮助我们解决这个问题. elixir 中没有类, 但是因为其易于扩展, 所以, 我们可以定义类型.

elixir 开箱提供了一些基础类型, 像 `integer` 或者 `pid`, 可以在[文档](https://hexdocs.pm/elixir/typespecs.html#types-and-their-syntax)中找到完整的可用列表.

### 定义自定义类型

让我们修改下 `sum_times` 函数, 并且引入一些额外的参数:
```elixir
@spec sum_times(integer, %Examples{first: integer, last: integer}) :: integer
def sum_times(a, params) do
  for i <- params.first..params.last do
    i
  end
  |> Enum.map(fn el -> el * a end)
  |> Enum.sum()
  |> round
end
```

我们引入了一个 `Examples` 模块中的结构体, 其包含 `first` 和 `last` 两个字段, 这是 `Range` 模块结构体的简单版本. 我们在[模块](modules.md#结构体)中, 有讨论过 `structs`. 想象一下, 我们需要在很多地方的规格中使用 `Examples` 的结构体, 其编写冗长, 规格复杂, 令人烦躁, 同时, 还可能成为错误的根源. 解决这样的问题可以使用 `@type`.

elixir 提供了三种类型的指令:

* `@type` - 简单, 共有类型. 类型的内部结构是共有的.
* `@typep` - 类型是私有的, 并且只能在其定义的模块内部使用.
* `@opaque` - 类型是共有的, 但是其内部结构是私有的.

定义我们的类型:
```elixir
defmodule Examples do
  defstruct first: nil, last: nil

  @type t(first, last) :: %Examples{first: first, last: last}

  @type t :: %Examples{first: integer, last: integer}
end
```

现在, 我们已经定义了类型 `t(first, last)`, 其是结构体 `%Examples{first: first, last: last}` 的表现形式. 在这一点上, 我们看到, 类型可以接收参数, 但是我们同样定义了类型 `t`, 并且, 这一次其是结构体 `%Examples{first: integer, last: integer}` 的表现形式.

有什么区别? 第一个代表结构体 `Examples`, 其中两个键可以是任意类型. 第二个代表结构体的键类型是 `integer`. 这意味着, 这样的代码:
```elixirr
@spec sum_times(integer, Examples.t()) :: integer
def sum_times(a, params) do
  for i <- params.first..params.last do
    i
  end
  |> Enum.map(fn el -> el * a end)
  |> Enum.sum()
  |> round
end
```

等价于代码:
```elixir
@spec sum_times(integer, Examples.t(integer, integer)) :: integer
def sum_times(a, params) do
  for i <- params.first..params.last do
    i
  end
  |> Enum.map(fn el -> el * a end)
  |> Enum.sum()
  |> round
end
```

### 类型的文档

我们最后要谈论的内容是如何为类型编写文档. 正如我们从[文档](documentations.md)章节学习的那样, 我们可以使用 `@doc` 和 `@moduledoc` 注释来创建函数和模块的文档. 为了文档化我们的类型, 可以使用 `@typedoc`:
```elixir
defmodule Examples do
  @typedoc """
      Type that represents Examples struct with :first as integer and :last as integer.
  """
  @type t :: %Examples{first: integer, last: integer}
end
```

指令 `@typedoc` 与 `@doc` 和 `@moduledoc` 类似.
