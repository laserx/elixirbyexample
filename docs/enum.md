# 枚举
用来枚举可枚举值的一套算法.

<!-- TOC -->

- [枚举](#%E6%9E%9A%E4%B8%BE)
    - [枚举](#%E6%9E%9A%E4%B8%BE)
        - [all?](#all)
        - [any?](#any)
        - [chunk_every](#chunkevery)
        - [chunk_by](#chunkby)
        - [map_every](#mapevery)
        - [each](#each)
        - [map](#map)
        - [min](#min)
        - [max](#max)
        - [reduce](#reduce)
        - [sort](#sort)
        - [uniq_by](#uniqby)

<!-- /TOC -->

## 枚举
枚举模块提供了超过70种函数来处理可枚举值. 在上一课程中我们学的所有集合, 除了元组, 都是可枚举的.

这一课我们只会覆盖小部分的可用函数, 然而, 我们可以自行学习其他的函数. 现在, 让我们在 `iex` 中做一点点小尝试.
```elixir
iex> Enum.__info__(:functions) |> Enum.each(fn({function, arity}) ->
...>   IO.puts "#{function}/#{arity}"
...> end)
all?/1
all?/2
any?/1
any?/2
at/2
at/3
...
```

如上, 枚举提供了大量的函数, 同时, 这又是显而易见的. 其是函数式编程的核心, 是非常有用的功能. 眼见为实, 通过它结合 elixir 其他的特性, 枚举提供给开发者难以置信的可操作性.

有关枚举的完整文档, 可以查看官方 [Enum](https://hexdocs.pm/elixir/Enum.html) 文档; 惰性枚举可以使用 [Stream](https://hexdocs.pm/elixir/Stream.html) 模块.

### all?
当使用 `all?/2`, 和大部分枚举方法一样, 我们提供一个函数来应用于集合的元素之上. 在这个例子中, 整个集合必须判定为 `true`, 否则, 将返回 `false`:
```elixir
iex> Enum.all?(["foo", "bar", "hello"], fn(s) -> String.length(s) == 3 end)
false
iex> Enum.all?(["foo", "bar", "hello"], fn(s) -> String.length(s) > 1 end)
true
```

### any?
不像 `all?/2`, `any?/2` 只要集合中有1个元素判定为 `true`, 就会返回 `true`:
```elixir
iex> Enum.any?(["foo", "bar", "hello"], fn(s) -> String.length(s) == 5 end)
true
```

### chunk_every
如果你需要将集合分割成多个小的分组, 你可以使用 `chunk_every/2` 这个函数:
```elixir
iex> Enum.chunk_every([1, 2, 3, 4, 5, 6], 2)
[[1, 2], [3, 4], [5, 6]]
```

还有一个具有更多元数的函数 `chunk_every/4`, 但是在这里我们不去探究, 如需了解相关内容, 可以参考[官方文档](https://hexdocs.pm/elixir/Enum.html#chunk_every/4)

### chunk_by
当我们需要将集合分组而不仅仅是根据分组的大小时, 我们可以使用函数 `chunk_by/2`, 其接受一个可枚举值和一个函数, 当函数的返回值发生变化时, 就会创建一个新的分组:
```elixir
iex> Enum.chunk_by(["one", "two", "three", "four", "five"], fn(x) -> String.length(x) end)
[["one", "two"], ["three"], ["four", "five"]]
iex> Enum.chunk_by(["one", "two", "three", "four", "five", "six"], fn(x) -> String.length(x) end)
[["one", "two"], ["three"], ["four", "five"], ["six"]]
```

### map_every
有时将集合分组不能满足我们的需求, 在某些情况下, `map_every/3` 对需要改变第 n 个值的场景很适用, 注意, 该方法总会处理第一个值:
```elixir
# Apply function every three items
iex> Enum.map_every([1, 2, 3, 4, 5, 6, 7, 8], 3, fn x -> x + 1000 end)
[1001, 2, 3, 1004, 5, 6, 1007, 8]
```

### each
当只需要迭代一个集合而不产生新值的情况下, 我们可以使用函数 `each/2`:
```elixir
iex> Enum.each(["one", "two", "three"], fn(s) -> IO.puts(s) end)
one
two
three
:ok
```
**注意**: 函数 `each/2` 会返回一个 `:ok` 的原子.

### map
需要将函数应用于每一个值同时生成一个新的集合, 可以使用函数 `map/2`:
```elixir
iex> Enum.map([0, 1, 2, 3], fn(x) -> x - 1 end)
[-1, 0, 1, 2]
```

### min
函数 `min/1` 用来查找集合中的最小值:
```elixir
iex> Enum.min([5, 3, 0, -1])
-1
```

`min/2` 的作用和 `min/1` 相同, 但是当可枚举值为空时, 函数允许我们指定一个函数来返回最小值:
```elixir 
iex> Enum.min([], fn -> :foo end)
:foo
```

### max
函数 `max/1` 返回集合中最大值:
```elixir
iex> Enum.max([5, 3, 0, -1])
5
```

`max/2` 类似于 `min/2`:
```elixir
Enum.max([], fn -> :bar end)
:bar
```

### reduce
使用函数 `reduce/3`, 我们可以将集合提炼为一个值. 为此我们可以为函数提供一个可选的累加器(在这个例子中是 `10`); 如果不提供累加器, 则直接使用第一个值作为累加器:
```elixir
iex> Enum.reduce([1, 2, 3], 10, fn(x, acc) -> x + acc end)
16

iex> Enum.reduce([1, 2, 3], fn(x, acc) -> x + acc end)
6

iex> Enum.reduce(["a","b","c"], "1", fn(x,acc)-> x <> acc end)
"cba1"
```

### sort
通过两个排序函数, 我们可以轻松的为集合排序.

`sort/1` 使用 erlang 的排序方式(term ordering??)来确定排序顺序:
```elixir
iex> Enum.sort([5, 6, 1, 3, -1, 4])
[-1, 1, 3, 4, 5, 6]

iex> Enum.sort([:foo, "bar", Enum, -1, 4])
[-1, 4, Enum, :foo, "bar"]
```

另外, `sort/2` 允许我们自己提供排序函数:
```elixir
# with our function
iex> Enum.sort([%{:val => 4}, %{:val => 1}], fn(x, y) -> x[:val] > y[:val] end)
[%{val: 4}, %{val: 1}]

# without
iex> Enum.sort([%{:count => 4}, %{:count => 1}])
[%{count: 1}, %{count: 4}]
```

### uniq_by
我们可以使用 `uniq_by/2` 来移除集合中的重复值:
```elixir
iex> Enum.uniq_by([1, 2, 3, 2, 1, 1, 1, 1, 1], fn x -> x end)
[1, 2, 3]
```