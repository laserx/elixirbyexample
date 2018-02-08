# erlang 互操作性
elixir 在Erlang VM (BEAM) 之上构建其中的一个额外好处是可以利用大量已经存在的库. 互操作性使我们可以在 elixir 代码中使用 erlang 的三方库和标准库. 在本章节中, 我们将探讨如何使用 erlang 中的标准库和三方包.

<!-- TOC -->

- [erlang 互操作性](#erlang-%E4%BA%92%E6%93%8D%E4%BD%9C%E6%80%A7)
    - [标准库](#%E6%A0%87%E5%87%86%E5%BA%93)
    - [erlang 三方包](#erlang-%E4%B8%89%E6%96%B9%E5%8C%85)
    - [显著差异](#%E6%98%BE%E8%91%97%E5%B7%AE%E5%BC%82)
        - [原子](#%E5%8E%9F%E5%AD%90)
        - [字符串](#%E5%AD%97%E7%AC%A6%E4%B8%B2)
        - [变量](#%E5%8F%98%E9%87%8F)

<!-- /TOC -->

## 标准库
在我们项目中任意的 elixir 代码都可以访问绝大多数 erlang 的标准库. erlnag 的模块由小写的原子表示, 例如 `:os` 和 `:timer`.

来使用 `:timer.tc` 计算函数的运行时间:
```elixir
defmodule Example do
  def timed(fun, args) do
    {time, result} = :timer.tc(fun, args)
    IO.puts("Time: #{time} μs")
    IO.puts("Result: #{result}")
  end
end

iex> Example.timed(fn (n) -> (n * n) * n end, [100])
Time: 8 μs
Result: 1000000
```

有关模块的完整列表, 可以访问 [erlang 参考手册](http://erlang.org/doc/apps/stdlib/)

## erlang 三方包
在之前的章节中, 我们介绍了使用 mix 管理项目依赖. 可以用同样的方式来管理 erlang 库的依赖. 如果 erlang 的库并没有推送至 [Hex](https://hex.pm/), 可以直接引用 git 仓库来替代:
```elixir
def deps do
  [{:png, github: "yuce/png"}]
end
```

现在我们可以使用 erlang 的库:
```elixir
png =
  :png.create(%{:size => {30, 30}, :mode => {:indexed, 8}, :file => file, :palette => palette})
```

## 显著差异
现在我们已经知道如何在 elixir 中使用 erlang, 接着, 我们要介绍一下来自 erlang 互操作性的问题.

### 原子
erlang 中的原子看起来和 elixir 中的原子很相似, 只是没有冒号 (`:`). 其是由小写字符串和下划线组成的:
elixir:
```elixir
:example
```

erlang:
```erlang
example.
```

### 字符串
在 elixir 中, 我们讨论字符串时, 实质是指 UTF-8 编码的二进制. 在 erlang 中, 字符串也是使用双引号包围, 但是讨论的实质是字符列表:
elixir:
```elixir
iex> is_list('Example')
true
iex> is_list("Example")
false
iex> is_binary("Example")
true
iex> <<"Example">> === "Example"
true
```

erlang:
```erlang
1> is_list('Example').
false
2> is_list("Example").
true
3> is_binary("Example").
false
4> is_binary(<<"Example">>).
true
```

需要十分注意的是, 很多较老的 erlang 库可能不支持二进制, 所以我们需要转换 elixir 的字符串至字符列表. 还好使用函数 `to_charlist/1` 转换起来很简单:
```elixir
iex> :string.words("Hello World")
** (FunctionClauseError) no function clause matching in :string.strip_left/2

    The following arguments were given to :string.strip_left/2:

        # 1
        "Hello World"

        # 2
        32

    (stdlib) string.erl:1661: :string.strip_left/2
    (stdlib) string.erl:1659: :string.strip/3
    (stdlib) string.erl:1597: :string.words/2

iex> "Hello World" |> to_charlist |> :string.words
2
```

### 变量
elixir:
```elixir
iex> x = 10
10

iex> x1 = x + 10
20
```

erlang:
```erlang
1> X = 10.
10

2> X1 = X + 1.
11
```

就这些! 在我们的 elixir 应用中充分利用现有的 erlang 库是很简单而有效的, 同时还让我们的可用库数量翻了一倍.
