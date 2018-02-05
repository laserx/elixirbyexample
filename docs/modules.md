# 模块
我们从以往的经验得出, 把所有的函数放在一个文件或者作用域下是不合理的. 在这一章我们会就少如何分组函数, 以及定义一种被称为结构体的专有映射来更有效地组织我们的代码.

<!-- TOC -->

- [模块](#%E6%A8%A1%E5%9D%97)
    - [模块](#%E6%A8%A1%E5%9D%97)
        - [模块属性](#%E6%A8%A1%E5%9D%97%E5%B1%9E%E6%80%A7)
    - [结构体](#%E7%BB%93%E6%9E%84%E4%BD%93)
    - [组合](#%E7%BB%84%E5%90%88)
        - [alias](#alias)
        - [import](#import)
            - [过滤](#%E8%BF%87%E6%BB%A4)
        - [require](#require)
        - [use](#use)

<!-- /TOC -->

## 模块
模块允许我们将函数组织于一个命名空间下. 除了将函数分组, 其还允许我们定义在[函数章节](https://elixirschool.com/en/lessons/basics/functions/)中提及的命名函数和私有函数.

让我们看一个基本的示例:
```elixir
defmodule Example do
  def greeting(name) do
    "Hello #{name}."
  end
end

iex> Example.greeting "Sean"
"Hello Sean."
```

可以在 elixir 中嵌套模块, 允许你进一步的功能化命名空间.
```elixir
defmodule Example.Greetings do
  def morning(name) do
    "Good morning #{name}."
  end

  def evening(name) do
    "Good night #{name}."
  end
end

iex> Example.Greetings.morning "Sean"
"Good morning Sean."
```

### 模块属性
模块属性在 elixir 中通常用来作为常量. 让我们看一个简单的例子:
```elixir
defmodule Example do
  @greeting "Hello"

  def greeting(name) do
    ~s(#{@greeting} #{name}.)
  end
end
```

请注意 elixir 中的保留属性. 以下3个是最常见的:
* `moduledoc` - 当前模块文档
* `doc` - 函数与宏文档
* `behaviour` - 使用 OTP 或用户定义的行为

## 结构体
结构体是带有键和默认值的特殊映射, 结构体必须在模块中定义, 结构体的命名来源于模块名. 一个模块中仅包含结构体定义是非常常见的.

我们使用 `defstruct` 接着一系列字段和默认值的关键字列表来定义结构体:
```elixir
defmodule Example.User do
  defstruct name: "Sean", roles: []
end
```

让我们试着创建一些结构体:
```elixir
iex> %Example.User{}
%Example.User{name: "Sean", roles: []}

iex> %Example.User{name: "Steve"}
%Example.User{name: "Steve", roles: []}

iex> %Example.User{name: "Steve", roles: [:admin, :owner]}
%Example.User{name: "Steve", roles: [:admin, :owner]}
```

更新结构体和更新映射一样:
```elixir
iex> steve = %Example.User{name: "Steve", roles: [:admin, :owner]}
%Example.User{name: "Steve", roles: [:admin, :owner]}
iex> sean = %{steve | name: "Sean"}
%Example.User{name: "Sean", roles: [:admin, :owner]}
```

更重要的是, 你可以将结构体与映射进行匹配:
```elixir
iex> %{name: "Sean"} = sean
%Example.User{name: "Sean", roles: [:admin, :owner]}
```

## 组合
我们已经知道如何创建模块和结构体, 接着让我们学习一下如何通过组合向结构体添加功能. elixir 提供给我们不同的方式来与别的模块进行交互.

### alias
elixir 允许我们为模块添加别名; 在代码中这种用法很常见:
```elixir
defmodule Sayings.Greetings do
  def basic(name), do: "Hi, #{name}"
end

defmodule Example do
  alias Sayings.Greetings

  def greeting(name), do: Greetings.basic(name)
end

# Without alias

defmodule Example do
  def greeting(name), do: Sayings.Greetings.basic(name)
end
```

如果两个别名产生冲突或者希望完全自定义别名, 可以使用 `:as`:
```elixir
defmodule Example do
  alias Sayings.Greetings, as: Hi

  def print_message(name), do: Hi.basic(name)
end
```

甚至可以一次引入多个模块别名:
```elixir
defmodule Example do
  alias Sayings.{Greetings, Farewells}
end
```

### import
如果我们想要导入函数和宏, 而不是模块别名时, 可以使用 `import/`:
```elixir
iex> last([1, 2, 3])
** (CompileError) iex:9: undefined function last/1
iex> import List
nil
iex> last([1, 2, 3])
3
```

#### 过滤
默认情况下, 会导入所有的函数的宏, 但是可以通过 `:only` 和 `:except` 来进行过滤.

要导入特定的函数和宏时, 我们必须为 `:only` 和 `:except` 提供明确的'函数名/元数'对. 让我们试着只导入 `last/1` 这一个函数:
```elixir
iex> import List, only: [last: 1]
iex> first([1, 2, 3])
** (CompileError) iex:13: undefined function first/1
iex> last([1, 2, 3])
3
```

如果我们导入除了 `last/1` 以外的所有内容, 再试试之前的示例:
```elixir
iex> import List, except: [last: 1]
nil
iex> first([1, 2, 3])
1
iex> last([1, 2, 3])
** (CompileError) iex:3: undefined function last/1
```

另外, 除了 '函数名/元数'对之外, 还有 `:functions` 和 `:macros` 这两个特殊原子, 分别对应只导入函数和只导入宏:
```elixir
import List, only: :functions
import List, only: :macros
```

### require
虽然 `require/2` 的使用频率很低, 但是其依然十分重要. 需求一个模块来确保其的宏被编译和加载. 当我们访问模块的宏时, 这很有有用:
```elixir
defmodule Example do
  require SuperMacros

  SuperMacros.do_stuff
end
```

如果尝试调用一个未加载的宏时, elixir 会抛出异常.

### use
通过 `use` 宏, 可以允许另一个模块修改当前模块的定义. 当我们在代码中调用 `use` 时, 实际上是调用由模块提供的 `__using__/1` 回调. 宏 `__using__/1` 的结果将成为我们模块定义的一部分. 为了更好的理解其是如何工作的, 让我们看看下面的例子:
```elixir
defmodule Hello do
  defmacro __using__(_opts) do
    quote do
      def hello(name), do: "Hi, #{name}"
    end
  end
end
```

在这里我们创建了一个 `Hello` 模块, 定义了回调函数 `__using__/1`, 并在其中定义了函数 `hello/1`. 让我们创建一个新的模块, 这样我们可以尝试我们的新代码:
```elixir
defmodule Example do
  use Hello
end
```

如果我们在 iex 中尝试一下代码, 可以看到在 `Example` 模块中存在有 `hello/1` 方法.
```elixir
iex> Example.hello("Sean")
"Hi, Sean"
```

这里可以观察到, `use` 调用 `Hello` 中的回调函数 `__using__/1`, 然后将结果代码添加到我们的模块中. 我们已经验证了一个简单的示例, 现在让我们更新一下代码, 看看 `__using__/1` 如何支持选项. 让我们通过添加一个 `greeting` 选项来实现这一步:
```elixir
defmodule Hello do
  defmacro __using__(opts) do
    greeting = Keyword.get(opts, :greeting, "Hi")

    quote do
      def hello(name), do: unquote(greeting) <> ", " <> name
    end
  end
end
```

让我们更新一下 `Example` 模块来包含新添加的 `greeting` 选项:
```elixir
defmodule Example do
  use Hello, greeting: "Hola"
end
```

当我们尝试在 iex 中调用是, 可以看到 `greeting` 产生了变化:
```
iex> Example.hello("Sean")
"Hola, Sean"
```

以上示例表明了 `use` 在 elixir 工具链中是多么不可思议的强大. 在后续 elixir 的学习中, 请留意 `use`, 其中有一个用例 `use ExUnit.Case, async: true` 你肯定会用到.

**注意**: `quote`, `alias`, `use`, `require` 是我们在元编程中使用的宏.
