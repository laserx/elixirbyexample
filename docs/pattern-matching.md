# 模式匹配
模式匹配是 elixir 中一个强大的部分. 其允许我们匹配简单的值, 数据结构, 甚至函数. 在这一课, 我们将会看到模式匹配是如何使用的.

<!-- TOC -->

- [模式匹配](#%E6%A8%A1%E5%BC%8F%E5%8C%B9%E9%85%8D)
    - [匹配操作符](#%E5%8C%B9%E9%85%8D%E6%93%8D%E4%BD%9C%E7%AC%A6)
    - [锁定操作符](#%E9%94%81%E5%AE%9A%E6%93%8D%E4%BD%9C%E7%AC%A6)

<!-- /TOC -->

## 匹配操作符
在 elixir 中, `=` 实质上是匹配操作符, 媲美代数中的等号. 通过它将整个表达式转换为一个等式, 并使 elixir 匹配左右两边的值. 如果匹配成功, 将返回等式的值. 不然, 将抛出异常, 让我们看一看:
```elixir
iex> x = 1
1
```

现在, 让我们尝试一些简单的匹配:
```elixir
iex> 1 = x
1
iex> 2 = x
** (MatchError) no match of right hand side value: 1
```

再让我们尝试匹配一些我们学习过的集合:
```elixir
# Lists
iex> list = [1, 2, 3]
iex> [1, 2, 3] = list
[1, 2, 3]
iex> [] = list
** (MatchError) no match of right hand side value: [1, 2, 3]

iex> [1 | tail] = list
[1, 2, 3]
iex> tail
[2, 3]
iex> [2 | _] = list
** (MatchError) no match of right hand side value: [1, 2, 3]

# Tuples
iex> {:ok, value} = {:ok, "Successful!"}
{:ok, "Successful!"}
iex> value
"Successful!"
iex> {:ok, value} = {:error}
** (MatchError) no match of right hand side value: {:error}
```

## 锁定操作符
当匹配的左侧包含变量的时候, 匹配操作符执行赋值. 在某些情况下, 并不期望重新绑定变量的行为, 为此, 我们可以使用锁定操作符: `^`.

当我们锁定一个变量的值时, 我们匹配已存在的值, 而不是重绑定一个新的值. 我们来看一下它是如何工作的:
```elixir
iex> x = 1
1
iex> ^x = 2
** (MatchError) no match of right hand side value: 2
iex> {x, ^x} = {2, 1}
{2, 1}
iex> x
2
```

elixir 1.2 引入了对图键和函数子句进行锁定功能:
```elixir
iex> key = "hello"
"hello"
iex> %{^key => value} = %{"hello" => "world"}
%{"hello" => "world"}
iex> value
"world"
iex> %{^key => value} = %{:hello => "world"}
** (MatchError) no match of right hand side value: %{hello: "world"}
```

一个函数子句锁定的示例:
```elixir
iex> greeting = "Hello"
"Hello"
iex> greet = fn
...>   (^greeting, name) -> "Hi #{name}"
...>   (greeting, name) -> "#{greeting}, #{name}"
...> end
#Function<12.54118792/2 in :erl_eval.expr/5>
iex> greet.("Hello", "Sean")
"Hi Sean"
iex> greet.("Mornin'", "Sean")
"Mornin', Sean"
```