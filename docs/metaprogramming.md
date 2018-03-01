# 元编程

元编程是使用代码生产代码的过程. 在 elixir 中, 元编程提供给我们扩展语言使其适应我们的需要, 以及动态改变代码的能力. 我们将首先看看 elixir 是如何在底层实现它的, 以及如何修改它, 最后, 我们可以使用以上的知识来扩展它.

**注意**: 元编程是非常复杂的, 只有在必要的时候才使用它. 过度使用肯定会造成代码难以理解和调试的后果.

<!-- TOC -->

- [元编程](#%E5%85%83%E7%BC%96%E7%A8%8B)
    - [Quote](#quote)
    - [Unquote](#unquote)
    - [宏](#%E5%AE%8F)
    - [调试](#%E8%B0%83%E8%AF%95)

<!-- /TOC -->

## Quote
元编程的第一步是理解表达式如何表示. 在 elixir 中, 抽象语法树 (AST), 我们代码的内部表现, 是由元组组成的. 该元组包含3部分: 函数名称, 元数据, 函数参数.

为了看到这些内部结构, elixir 提供给我们函数 `quote/2`. 使用 `quote/2`, 我们可以转换 elixir 代码至其底层表现形式:
```elixir
iex> quote do: 42
42
iex> quote do: "Hello"
"Hello"
iex> quote do: :world
:world
iex> quote do: 1 + 2
{:+, [context: Elixir, import: Kernel], [1, 2]}
iex> quote do: if value, do: "True", else: "False"
{:if, [context: Elixir, import: Kernel],
 [{:value, [], Elixir}, [do: "True", else: "False"]]}
```

是否注意到前3个返回值并不是元组? 是因为当使用 `quote` 时, 有5种原语返回他们本身:
```elixir
iex> :atom
:atom
iex> "string"
"string"
iex> 1 # All numbers
1
iex> [1, 2] # Lists
[1, 2]
iex> {"hello", :world} # 2 element tuples
{"hello", :world}
```

## Unquote
现在, 我们可以获取我们代码的内部结构, 但是我们如何修改它? 要注入新的代码或值, 我们可以使用 `unquote/1`. 当我们 `unquote` 表达式时, 当我们 `unquote` 表达式时, 它将执行并注入至抽象语法树中. 为了演示 `unquote/1`, 让我们看看下面的例子:
```elixir
iex> denominator = 2
2
iex> quote do: divide(42, denominator)
{:divide, [], [42, {:denominator, [], Elixir}]}
iex> quote do: divide(42, unquote(denominator))
{:divide, [], [42, 2]}
```

在这个例子中, 我们 `quote` 了变量 `denominator`, 所以结果中的抽象语法树包含了用于访问该变量的元组. 在 `unquote/1` 示例中, 结果代码包含了 `denominator` 的值.

## 宏
一旦我们理解了 `quote/2` 和 `unquote/1`, 我们就可以准备深入研究宏了. 重要的是要记住, 和所有的元编程一样, 请谨慎使用它.

用最简单的术语表述, 宏是特殊的函数, 设计用来返回一个将插入我们应用程序代码中的引用表达式. 想象一下, 宏被替换为引用表达式, 而不是调用一个函数. 通过宏, 我们具有了扩展 elixir 和动态添加代码至我们的应用程序所需的一切.

我们首先使用 `defmacro/2` 定义一个宏, 就像 elixir 许多部分一样, 其本身就是一个宏 (慢慢体会一下). 下面的示例中, 我们会通过宏现实 `unless`, 记住, 我们的宏需要返回一个引用表达式:
```elixir
defmodule OurMacro do
  defmacro unless(expr, do: block) do
    quote do
      if !unquote(expr), do: unquote(block)
    end
  end
end
```

引入我们的模块, 试试我们的宏:
```elixir
iex> require OurMacro
nil
iex> OurMacro.unless true, do: "Hi"
nil
iex> OurMacro.unless false, do: "Hi"
"Hi"
```

因为宏替换应用程序中的代码, 所以我们可以控制它的何时和如何编译. 我们可以在 `Logger` 模块中找到这样的示例. 当 `logging` 被禁用, 就不会有代码注入, 并且最终的应用程序不包含引用和函数调用. 与其他语言的区别是, 即使实现为空, 依然存在函数调用的开销.

为了证明这一点, 我们会编写一段简单的 `logger` 代码, 并可以启用或停用它:
```elixir
defmodule Logger do
  defmacro log(msg) do
    if Application.get_env(:logger, :enabled) do
      quote do
        IO.puts("Logged message: #{unquote(msg)}")
      end
    end
  end
end

defmodule Example do
  require Logger

  def test do
    Logger.log("This is a log message")
  end
end
```

当 `logging` 启用时, 我们的 `test` 函数在代码中看起来是这样的:
```elixir
def test do
  IO.puts("Logged message: #{"This is a log message"}")
end
```

如果我们停用它, 结果代码看起来是这样的:
```elixir
def test do
end
```

## 调试
