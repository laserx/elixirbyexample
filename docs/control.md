# 流程控制
在这一课我们将看看 elixir 中可用的流程控制方法.

<!-- TOC -->

- [流程控制](#%E6%B5%81%E7%A8%8B%E6%8E%A7%E5%88%B6)
    - [if & unless](#if-unless)
    - [case](#case)
    - [cond](#cond)
    - [with](#with)

<!-- /TOC -->

## if & unless
在了解 `if/2` 前, 如果你有使用 Ruby 的经验, 你可能对 `unless/2` 比较熟悉. 在 elixir 中, 他们的行为和 Ruby 中一样, 不过, `if/2` 和 `unless/2` 是宏定义, 而不是语言结构; 你可以在[Kernel module](https://hexdocs.pm/elixir/Kernel.html)中找到他们的实现方法.

需要注意的是, 在 elixir 中, 只有 `nil` 和布尔值 `false` 为假值.
```elixir
iex> if String.valid?("Hello") do
...>   "Valid string!"
...> else
...>   "Invalid string."
...> end
"Valid string!"

iex> if "a string value" do
...>   "Truthy"
...> end
"Truthy"
```

`unless/2` 的用法和 `if/2` 一样, 只是其对否定生效:
```elixir
iex> unless is_integer("hello") do
...>   "Not an Int"
...> end
"Not an Int"
```

## case
如果需要匹配多个模式时, 我们可以使用 `case/2`:
```elixir
iex> case {:ok, "Hello World"} do
...>   {:ok, result} -> result
...>   {:error} -> "Uh oh!"
...>   _ -> "Catch all"
...> end
"Hello World"
```

变量 `_` 在 `case/2` 中有着重要的位置. 如果没有 `_`, 当没有匹配结果时, 将会抛出异常:
```elixir
iex> case :even do
...>   :odd -> "Odd"
...> end
** (CaseClauseError) no case clause matching: :even

iex> case :even do
...>   :odd -> "Odd"
...>   _ -> "Not Odd"
...> end
"Not Odd"
```

可以认为 `_` 和 `else` 一样, 可以匹配"所有的其他条件".

由于 `case/2` 依赖于模式匹配, 模式匹配中的所有规则和限制都适用于 `case/2`. 如果你期望和现有已存在的变量进行匹配, 你必须使用锁定操作符 `^/1`:
```elixir
iex> pie = 3.14
 3.14
iex> case "cherry pie" do
...>   ^pie -> "Not so tasty"
...>   pie -> "I bet #{pie} is tasty"
...> end
"I bet cherry pie is tasty"
```

`case/2` 具有一个很好的特性, 其支持哨兵子句:

_这一示例直接来源于 elixir 官方[入门指南](https://elixir-lang.org/getting-started/case-cond-and-if.html#case)._
```elixir
iex> case {1, 2, 3} do
...>   {1, x, 3} when x > 0 ->
...>     "Will match"
...>   _ ->
...>     "Won't match"
...> end
"Will match"
```

可以参考官方文档学习在[哨兵子句](https://hexdocs.pm/elixir/master/guards.html)允许使用的表达式的相关知识.

## cond
当我们需要匹配多种条件而不是值的时候, 可以使用 `cond/1`; 其类似于别的语言中的 `else if` 或者 `elsif`:
_这一示例直接来源于 elixir 官方[入门指南](https://elixir-lang.org/getting-started/case-cond-and-if.html#cond)._
```elixir
iex> cond do
...>   2 + 2 == 5 ->
...>     "This will not be true"
...>   2 * 2 == 3 ->
...>     "Nor this"
...>   1 + 1 == 2 ->
...>     "But this will"
...> end
"But this will"
```

和 `case/2` 一样, `cond/1` 如果没有匹配上任何结果, 将抛出异常. 为了解决这个问题, 我们可以定义一个条件, 保证其为 `true`:
```elixir
iex> cond do
...>   7 + 1 == 0 -> "Incorrect"
...>   true -> "Catch all"
...> end
"Catch all"
```

## with
当使用嵌套的 `case/2` 语句无法清晰的连接起来时, 可以用 `with/1` 来有效的解决这个问题. `with/1` 表达式由关键字, 生成器和最终表达式组成.

我们将在[列表推导](https://elixirschool.com/en/lessons/basics/comprehensions/)中更深入的讨论生成器的相关知识, 现在我们只需要其使用模式匹配来比较 `<-` 的左右两侧.

我们用一个简单的 `with/1` 的示例来看看它能做什么:
```elixir
iex> user = %{first: "Sean", last: "Callan"}
%{first: "Sean", last: "Callan"}
iex> with {:ok, first} <- Map.fetch(user, :first),
...>      {:ok, last} <- Map.fetch(user, :last),
...>      do: last <> ", " <> first
"Callan, Sean"
```

当表达式匹配失败, 将返回未匹配的值:
```elixir
iex> user = %{first: "doomspork"}
%{first: "doomspork"}
iex> with {:ok, first} <- Map.fetch(user, :first),
...>      {:ok, last} <- Map.fetch(user, :last),
...>      do: last <> ", " <> first
:error
```

现在让我们看看在不使用 `with/1` 的情况下, 一个稍大一点的示例我们如何重构它:
```elixir
case Repo.insert(changeset) do
  {:ok, user} ->
    case Guardian.encode_and_sign(user, :token, claims) do
      {:ok, token, full_claims} ->
        important_stuff(token, full_claims)

      error ->
        error
    end

  error ->
    error
end
```

当我们引入 `with/1`, 我们得到的代码更容易理解, 同时代码量也更少:
```elixir
with {:ok, user} <- Repo.insert(changeset),
     {:ok, token, full_claims} <- Guardian.encode_and_sign(user, :token, claims) do
  important_stuff(token, full_claims)
end
```

自 elixir 1.3 后, `with/1` 语句支持 `else`:
```elixir
import Integer

m = %{a: 1, c: 3}

a =
  with {:ok, number} <- Map.fetch(m, :a),
    true <- is_even(number) do
      IO.puts "#{number} divided by 2 is #{div(number, 2)}"
      :even
  else
    :error ->
      IO.puts("We don't have this item in map")
      :error

    _ ->
      IO.puts("It is odd")
      :odd
  end
```

这可以通过提供一个类似 `case` 的模式匹配方式来辅助处理异常. 值将是第一个非匹配的表达式.