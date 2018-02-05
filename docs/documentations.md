# 文档
elixir 代码文档化.

<!-- TOC -->

- [文档](#%E6%96%87%E6%A1%A3)
    - [注释](#%E6%B3%A8%E9%87%8A)
        - [行内文档](#%E8%A1%8C%E5%86%85%E6%96%87%E6%A1%A3)
        - [文档化模块](#%E6%96%87%E6%A1%A3%E5%8C%96%E6%A8%A1%E5%9D%97)
        - [文档化函数](#%E6%96%87%E6%A1%A3%E5%8C%96%E5%87%BD%E6%95%B0)
    - [ExDoc](#exdoc)

<!-- /TOC -->

## 注释
代码的注释量, 文档的质量, 在编程的世界中依然是个有争论的问题. 但是, 我们都认同文档对我们自己以及使用我们代码的人都是很重要的.

elixir 将文档视作 _一等公民_, 为我们的项目提供了大量的功能来访问和生成文档. elixir 核心为我们提供了许多不同的属性来标注代码. 让我们看看这3种方式:
* `#` - 行内注释
* `@moduledoc` - 模块注释
* `@doc` - 函数注释

### 行内文档
行内注释可能是为代码添加注释最简单的方法了. 类似 Ruby 和 Python, elixir 的行内注释使用 `#` 标记. 也就是井号.

观察这个 elixir 脚本 (greeting.exs):
```elixir
# Outputs 'Hello, chum.' to the console.
IO.puts("Hello, " <> "chum.")
```

当 elixir 执行脚本时, 会忽略掉 `#` 后这行的所有代码, 将其视为无用的数据. 添加注释对代码的操作和性能不会产生影响, 但是, 别的程序员并不明确了解代码实现了些什么的情况下, 可以通过阅读你的注释来了解它. 注意不要滥用行内注释! 凌乱的代码会不受欢迎. 请适度使用.

### 文档化模块
`@moduledoc` 注释器允许添加模块级别的内联文档. 它通常位于文件顶部的 `defmodule` 声明之后. 下面的例子显示了在 `@moduledoc` 装饰器中的一行注释:
```elixir
defmodule Greeter do
  @moduledoc """
  Provides a function `hello/1` to greet a human
  """

  def hello(name) do
    "Hello, " <> name
  end
end
```

我们可以在 iex 中通过 `h` 帮助函数来获取模块的文档.
```elixir
iex> c("greeter.ex")
[Greeter]

iex> h Greeter

                Greeter

Provides a function hello/1 to greet a human
```

### 文档化函数
就像 elixir 提供给我们添加模块级注释一样, 同样具有类似的功能用来注释函数. `@doc` 注释器允许添加函数级别的内联文档. `@doc` 注释器位于被注释的函数之上.
```elixir
defmodule Greeter do
  @moduledoc """
  ...
  """

  @doc """
  Prints a hello message

  ## Parameters

    - name: String that represents the name of the person.

  ## Examples

      iex> Greeter.hello("Sean")
      "Hello, Sean"

      iex> Greeter.hello("pete")
      "Hello, pete"

  """
  @spec hello(String.t()) :: String.t()
  def hello(name) do
    "Hello, " <> name
  end
end
```

再次回到 iex 中, 当我们在模块名前的函数上使用帮助命令(`h`), 我们可以看到:
```
iex> c("greeter.ex")
[Greeter]

iex> h Greeter.hello

                def hello(name)

Prints a hello message

Parameters

  • name: String that represents the name of the person.

Examples

    iex> Greeter.hello("Sean")
    "Hello, Sean"

    iex> Greeter.hello("pete")
    "Hello, pete"

iex>
```
你是否注意到我们可以在文档中使用标记语言, 并且在命令行中渲染出来? 除了很酷以外, 这还是 elixir 生态系统中一个新的补充, 当你看到通过 ExDoc 动态生成 HTML 格式文档的时候, 你会觉得更有趣.

**注意**: `@spec` 注释是用来进行代码静态分析的. 要学习更多信息, 请查看[规范和类型](https://elixirschool.com/en/lessons/advanced/typespec)章节.

## ExDoc
