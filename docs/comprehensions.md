# 推导式
列表推导式是 elixir 中循环可枚举数据的语法糖. 在本章我们学习如何使用推导式来迭代和生成.

<!-- TOC -->

- [推导式](#%E6%8E%A8%E5%AF%BC%E5%BC%8F)
    - [基础](#%E5%9F%BA%E7%A1%80)
    - [过滤器](#%E8%BF%87%E6%BB%A4%E5%99%A8)
    - [使用 `:into`](#%E4%BD%BF%E7%94%A8-into)

<!-- /TOC -->

## 基础
推导式通常用来为 `Enum` 和 `Stream` 迭代创造更简洁的语句. 让我们看一个简单的推导式, 并分解它:
```elixir
iex> list = [1, 2, 3, 4, 5]
iex> for x <- list, do: x*x
[1, 4, 9, 16, 25]
```

我们首先注意到的是 `for` 和生成器. 什么是生成器? 生成器是列表推导式中 `x <- [1, 2, 3, 4]` 这段表达式. 其用来生成下一个值.

幸运的是, 推导式不仅仅限制于列表; 实质上, 它可以作用于任意的可枚举值:
```elixir
# Keyword Lists
iex> for {_key, val} <- [one: 1, two: 2, three: 3], do: val
[1, 2, 3]

# Maps
iex> for {k, v} <- %{"a" => "A", "b" => "B"}, do: {k, v}
[{"a", "A"}, {"b", "B"}]

# Binaries
iex> for <<c <- "hello">>, do: <<c>>
["h", "e", "l", "l", "o"]
```

像 elixir 中许多其他的东西一样, 生成器依赖模式匹配将输入和左侧的变量进行比较. 在找到匹配时, 该值将被忽略:
```elixir
iex> for {:ok, val} <- [ok: "Hello", error: "Unknown", ok: "World"], do: val
["Hello", "World"]
```

推导式可以使用多个生成器, 就像嵌套循环一样:
```elixir
iex> list = [1, 2, 3, 4]
iex> for n <- list, times <- 1..n do
...>   String.duplicate("*", times)
...> end
["*", "*", "**", "*", "**", "***", "*", "**", "***", "****"]
```

为了更好的说明循环中发生了什么, 使用 `IO.puts` 将两个生成器的值显示出来:
```elixir
iex> for n <- list, times <- 1..n, do: IO.puts "#{n} - #{times}"
1 - 1
2 - 1
2 - 2
3 - 1
3 - 2
3 - 3
4 - 1
4 - 2
4 - 3
4 - 4
```

列表推导式是语法糖, 应该用在适当的情况下. (请适度服用)

## 过滤器
你可以将过滤器理解为推导式的哨兵. 当一个被过滤的值返回 `false` 或者 `nil` 时, 它将从最终结果的列表中移除掉. 试着循环一个范围值, 且只关注偶数. 我们可以使用 `Integer` 模块中函数 `is_even/1` 判断值是否为偶数:
```elixir
import Integer
iex> for x <- 1..10, is_even(x), do: x
[2, 4, 6, 8, 10]
```

像生成器一样, 我们可以使用多个过滤器. 让我们扩大值的范围, 同时过滤出为偶数同时能被3整除的值:
```elixir
import Integer
iex> for x <- 1..100,
...>   is_even(x),
...>   rem(x, 3) == 0, do: x
[6, 12, 18, 24, 30, 36, 42, 48, 54, 60, 66, 72, 78, 84, 90, 96]
```

## 使用 `:into`
如果我们期望生成除列表以外的东西, 可以使用 `:into` 选项做到这一点. 一般来说, `:into` 接受任意实现了 `Collectable` 协议的结构体. 

让我们使用 `:into` 从关键字列表创建一个映射:
```elixir
iex> for {k, v} <- [one: 1, two: 2, three: 3], into: %{}, do: {k, v}
%{one: 1, three: 3, two: 2}
```

由于二进制实现了 `Collectable`, 我们可以用列表推导式和 `:into` 创建一个字符串:
```elixir
iex> for c <- [72, 101, 108, 108, 111], into: "", do: <<c>>
"Hello"
```

就是这样, 列表推导式是用简洁的方式实现遍历集合的简单方法.