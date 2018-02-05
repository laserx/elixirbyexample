# 印记

使用和创建印记.

<!-- TOC -->

- [印记](#%E5%8D%B0%E8%AE%B0)
    - [印记概述](#%E5%8D%B0%E8%AE%B0%E6%A6%82%E8%BF%B0)
        - [字符列表](#%E5%AD%97%E7%AC%A6%E5%88%97%E8%A1%A8)
        - [正则表达式](#%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)
        - [字符串](#%E5%AD%97%E7%AC%A6%E4%B8%B2)
        - [单词表](#%E5%8D%95%E8%AF%8D%E8%A1%A8)
        - [NaiveDateTime](#naivedatetime)
    - [创建印记](#%E5%88%9B%E5%BB%BA%E5%8D%B0%E8%AE%B0)

<!-- /TOC -->

## 印记概述
elixir 提供了一套替代语法用来表示和操作字面量. 印记使用 `~` 开头, 后面跟着一个字符. 虽然 elixir 核心内置了一些印记, 但当我们需要对语言进行扩展时, elixir 提供了创建属于我们自己的印记的可能.

内置可用印记列表:
* `~C` 生成一个 **不处理** 转义和插值的字符列表
* `~c` 生成一个 **处理** 转义和插值的字符列表
* `~R` 生成一个 **不处理** 转义和插值的正则表达式
* `~r` 生成一个 **处理** 转义和插值的正则表达式
* `~S` 生成一个 **不处理** 转义和插值的字符串
* `~s` 生成一个 **处理** 转义和插值的字符串
* `~W` 生成一个 **不处理** 转义和插值的单词表
* `~w` 生成一个 **处理** 转义和插值的单词表
* `~N` 生成一个 `NaiveDateTime` 结构体

可用分隔符列表:
* `<...>` 一对尖括号
* `{...}` 一对花括号
* `[...]` 一对中括号
* `(...)` 一对圆括号
* `|...|` 一对竖线
* `/.../` 一对斜线
* `"..."` 一对双引号
* `'...'` 一对单引号

### 字符列表
印记 `~c` 和 `~C` 分别生成字符列表. 例如:
```elixir
iex> ~c/2 + 7 = #{2 + 7}/
'2 + 7 = 9'

iex> ~C/2 + 7 = #{2 + 7}/
'2 + 7 = \#{2 + 7}'
```

我们可以看到小写的 `~c` 中插值被计算, 而大写的 `~C` 没有发生. 我们会发现在内置印记中, 这种大小写相对立的情况是普遍的.

### 正则表达式

印记 `~r` 和 `~R` 用来处理正则表达式. 我们动态的创建和使用在正则函数中. 例如:
```elixir
iex> re = ~r/elixir/
~r/elixir/

iex> "Elixir" =~ re
false

iex> "elixir" =~ re
true
```

我们可以看到, 在第一次正则匹配是, `Elixir` 与正则表达式不匹配. 这是因为首字母的大写. 由于 elixir 支持 PCRE, 我们可以在印记后添加 `i`, 来取消大小写敏感.
```elixir
iex> re = ~r/elixir/i
~r/elixir/i

iex> "Elixir" =~ re
true

iex> "elixir" =~ re
true
```

另外, elixir 提供了一套构建在 erlang 的正则表达式库之上的 [Regex](https://hexdocs.pm/elixir/Regex.html) API. 让我们用正则印记实现 `Regex.split/2`:
```elixir
iex> string = "100_000_000"
"100_000_000"

iex> Regex.split(~r/_/, string)
["100", "000", "000"]
```

如我们看到的一样. 字符串 `"100_000_000"` 由于印记 `~r/_/` 被 `_` 分割. 函数 `Regex.split/2` 返回了一个列表.

### 字符串
印记 `~s` 和 `~S` 用来生成字符串数据, 例如:
```elixir
iex> ~s/the cat in the hat on the mat/
"the cat in the hat on the mat"

iex> ~S/the cat in the hat on the mat/
"the cat in the hat on the mat"
```

有什么不同吗? 实质上不同之处类似于字符列表印记, 答案就是插值和转义字符的的使用, 我们再看看另一个示例:
```elixir
iex> ~s/welcome to elixir #{String.downcase "school"}/
"welcome to elixir school"

iex> ~S/welcome to elixir #{String.downcase "school"}/
"welcome to elixir \#{String.downcase \"school\"}"
```

### 单词表
单词表印记很有用, 它可以节省敲击代码的时间, 减少代码的复杂度. 看看这个例子:
```elixir
iex> ~w/i love elixir school/
["i", "love", "elixir", "school"]

iex> ~W/i love elixir school/
["i", "love", "elixir", "school"]
```

我们可以看到在分隔符之间键入的内容被空格分隔成列表, 虽然这两个例子没有任何区别. 让我们再看看包含插值和转义字符的例子:
```elixir
iex> ~w/i love #{'e'}lixir school/
["i", "love", "elixir", "school"]

iex> ~W/i love #{'e'}lixir school/
["i", "love", "\#{'e'}lixir", "school"]
```

### NaiveDateTime
[NaiveDateTime](https://hexdocs.pm/elixir/NaiveDateTime.html) 可以用来快速创建一个结构体来表示**不包含**时区信息的 `DateTime`.

在大多数情况下, 我们应该避免直接创建 `NaiveDateTime` 结构体. 但是, 在模式匹配中其非常有用, 例如:
```elixir
iex> NaiveDateTime.from_iso8601("2015-01-23 23:50:07") == {:ok, ~N[2015-01-23 23:50:07]}
```

## 创建印记
elixir 的目标之一是成为一种可扩展的语言. 你不应该因为可以轻松的创建属于自己的印记而感到惊讶. 在接下来的示例中, 我们将创建一个字符串大写转换的印记. 由于在 elixir 内核中, 已经存在了一个 (`String.upcase/1`) 函数, 我们将围绕这个函数, 将其封装为印记.
```elixir
iex> defmodule MySigils do
...>   def sigil_u(string, []), do: String.upcase(string)
...> end

iex> import MySigils
nil

iex> ~u/elixir school/
ELIXIR SCHOOL
```

首先, 我们定义一个名为 `MySigils` 的模块, 并在模块中, 创建一个名为 `sigil_u` 的函数. 我们可以使用 `~u`, 因为现有的印记空间中不存在这个印记. `_u` 表明我们希望使用 `u` 作为 `~` 后的字符. 函数定义必须接收两个参数, 一个输入, 一个列表.