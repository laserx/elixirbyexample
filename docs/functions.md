# 函数

在 elixir 以及大量其他的函数式语言中, 函数是一等公民. 我们将掌握 elixir 中函数的类型, 不同类型的差异, 以及如何使用这些函数.

## 匿名函数
顾名思义, 匿名函数意味其没有名称. 正如我们在枚举中看到的那样, 其通常被传递给另外一个函数. 在 elixir 中声明匿名函数需要使用关键字 `fn` 和 `end`. 通过 `->` 的分隔, 我们可以任意的定义函数的参数以及函数体.

让我们看一个基本示例:
```elixir
iex> sum = fn (a, b) -> a + b end
iex> sum.(2, 3)
5
```

### & 简写操作符
在 elixir 中使用匿名函数是经常性的操作, 为此, 存在简写操作来解决这个问题:
```elixir
iex> sum = &(&1 + &2)
iex> sum.(2, 3)
5
``` 
你可能已经猜到, 在简写版本中, 我们的参数使用 `&1`, `&2`, `&3` 等.

## 模式匹配
elixir 中的模式匹配不仅仅局限于变量中, 正如我们这一章节将要看到的, 其可以应用于函数的签名.

elixir 使用模式匹配来分辨出最先匹配的参数集, 并调用其对于的函数体:
```elixir
iex> handle_result = fn
...>   {:ok, result} -> IO.puts "Handling result..."
...>   {:error} -> IO.puts "An error has occurred!"
...> end

iex> some_result = 1
iex> handle_result.({:ok, some_result})
Handling result...

iex> handle_result.({:error})
An error has occurred!
```

## 命名函数
我们可以声明带有名称的函数, 以便稍后我们可以轻松的引用它们. 命名函数定义于一个模块内使用关键字 `def`. 我们会在下一章节学习更多的关于模块的内容, 现在, 我们只需要专注于命名函数其上.

在一个模块中可以调用另一个模块中定义的函数. 下面是在 elixir 中常用的一种块构建的方式:
```elixir
defmodule Greeter do
  def hello(name) do
    "Hello, " <> name
  end
end

iex> Greeter.hello("Sean")
"Hello, Sean"
```

如果我们的函数体只有一行, 我们可以用 `do:` 让它更短一点:
```elixir
defmodule Greeter do
  def hello(name), do: "Hello, " <> name
end
```

借助于我们对模式匹配的认知, 让我们尝试递归命名函数:
```elixir
defmodule Length do
  def of([]), do: 0
  def of([_ | tail]), do: 1 + of(tail)
end

iex> Length.of []
0
iex> Length.of [1, 2, 3]
3
```

### 函数命名和元数
我们在之前有提到函数的命名是通过函数名和元数(参数个数)组成的. 这意味着你可以这样做:
```elixir
defmodule Greeter2 do
  def hello(), do: "Hello, anonymous person!"   # hello/0
  def hello(name), do: "Hello, " <> name        # hello/1
  def hello(name1, name2), do: "Hello, #{name1} and #{name2}"
                                                # hello/2
end

iex> Greeter2.hello()
"Hello, anonymous person!"
iex> Greeter2.hello("Fred")
"Hello, Fred"
iex> Greeter2.hello("Fred", "Jane")
"Hello, Fred and Jane"
```
我们在注释中列出了函数名. 第一个实现是无参的, 所以称之为 `hello/0`; 第二个实现由一个参数, 称之为 `hello/1`, 以此类推. 不像其他语言中函数的重载, 这些被认为是彼此_不同的_函数.(刚刚描述的模式匹配, 只适用于具有_相同_元数多重定义的函数定义时)

### 私有函数
当我们不希望特定的函数被别的模块所访问, 我们可以将其私有. 私有函数只可以在模块内部被调用. 在 elixir 中我们使用 `defp` 定义私有函数:
```elixir
defmodule Greeter do
  def hello(name), do: phrase <> name
  defp phrase, do: "Hello, "
end

iex> Greeter.hello("Sean")
"Hello, Sean"

iex> Greeter.phrase
** (UndefinedFunctionError) function Greeter.phrase/0 is undefined or private
    Greeter.phrase()
```

### 哨兵
之前在[流程控制](https://elixirschool.com/en/lessons/basics/control-structures)中简短的提到了哨兵子句, 现在我们会看到如何在命名函数中使用他们. 一旦 elixir 匹配了一个函数, 对应其存在的所有哨兵子句都将被验证.
在接下来的示例中, 我们有2个函数具有相同的参数, 我们依靠哨兵子句根据参数类型决定使用哪一个函数:
```elixir
defmodule Greeter do
  def hello(names) when is_list(names) do
    names
    |> Enum.join(", ")
    |> hello
  end

  def hello(name) when is_binary(name) do
    phrase() <> name
  end

  defp phrase, do: "Hello, "
end

iex> Greeter.hello ["Sean", "Steve"]
"Hello, Sean, Steve"
```

### 参数默认值
如果我们希望为参数指定默认值, 可以使用语法 `argument \\ value`:
```elixir
defmodule Greeter do
  def hello(name, language_code \\ "en") do
    phrase(language_code) <> name
  end

  defp phrase("en"), do: "Hello, "
  defp phrase("es"), do: "Hola, "
end

iex> Greeter.hello("Sean", "en")
"Hello, Sean"

iex> Greeter.hello("Sean")
"Hello, Sean"

iex> Greeter.hello("Sean", "es")
"Hola, Sean"
```

当我们把哨兵子句和参数默认值结合在一起时, 会产生一个问题, 让我们看看可能的样子:
```elixir
defmodule Greeter do
  def hello(names, language_code \\ "en") when is_list(names) do
    names
    |> Enum.join(", ")
    |> hello(language_code)
  end

  def hello(name, language_code \\ "en") when is_binary(name) do
    phrase(language_code) <> name
  end

  defp phrase("en"), do: "Hello, "
  defp phrase("es"), do: "Hola, "
end

** (CompileError) iex:31: definitions with multiple clauses and default values require a header. Instead of:

    def foo(:first_clause, b \\ :default) do ... end
    def foo(:second_clause, b) do ... end

one should write:

    def foo(a, b \\ :default)
    def foo(:first_clause, b) do ... end
    def foo(:second_clause, b) do ... end

def hello/2 has multiple clauses and defines defaults in one or more clauses
    iex:31: (module)
```

elixir 不支持在多重匹配函数中设置参数默认值, 这会产生混淆. 解决这个问题, 我们需要添加一个携带参数默认值的顶部函数.
```elixir
defmodule Greeter do
  def hello(names, language_code \\ "en")

  def hello(names, language_code) when is_list(names) do
    names
    |> Enum.join(", ")
    |> hello(language_code)
  end

  def hello(name, language_code) when is_binary(name) do
    phrase(language_code) <> name
  end

  defp phrase("en"), do: "Hello, "
  defp phrase("es"), do: "Hola, "
end

iex> Greeter.hello ["Sean", "Steve"]
"Hello, Sean, Steve"

iex> Greeter.hello ["Sean", "Steve"], "es"
"Hola, Sean, Steve"
```