# 元编程

元编程是使用代码生产代码的过程. 在 elixir 中, 元编程提供给我们扩展语言使其适应我们的需要, 以及动态改变代码的能力. 我们将首先看看 elixir 是如何在底层实现它的, 以及如何修改它, 最后, 我们可以使用以上的知识来扩展它.

**注意**: 元编程是非常复杂的, 只有在必要的时候才使用它. 过度使用肯定会造成代码难以理解和调试的后果.

<!-- TOC -->

- [元编程](#%E5%85%83%E7%BC%96%E7%A8%8B)
    - [Quote](#quote)
    - [Unquote](#unquote)

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

在第一例子中, 我们引用了变量 `denominator`, 所以抽象语法树包含的元组中
In the first example our variable denominator is quoted so the resulting AST includes a tuple for accessing the variable. In the unquote/1 example the resulting code includes the value of denominator instead.

