# 字符串
字符串, 字符列表, 字位和码位.

<!-- TOC -->

- [字符串](#%E5%AD%97%E7%AC%A6%E4%B8%B2)
    - [字符串](#%E5%AD%97%E7%AC%A6%E4%B8%B2)
    - [字符串列表](#%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%88%97%E8%A1%A8)
    - [字位 & 码位](#%E5%AD%97%E4%BD%8D-%E7%A0%81%E4%BD%8D)
    - [字符串函数](#%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%87%BD%E6%95%B0)
        - [`length/1`](#length1)
        - [`replace/3`](#replace3)
        - [`duplicate/2`](#duplicate2)
        - [`split/2`](#split2)
    - [练习](#%E7%BB%83%E4%B9%A0)
        - [异序词](#%E5%BC%82%E5%BA%8F%E8%AF%8D)

<!-- /TOC -->

## 字符串
elixir 中的字符串不过是字节序列. 我们看这个例子:
```elixir
iex> string = <<104,101,108,108,111>>
"hello"
iex> string <> <<0>>
<<104, 101, 108, 108, 111, 0>>
```

通过将字符串与 `0` 字节拼接, iex 将字符串显示为二进制, 因为其已经不再是一个有效的字符串了. 这个技巧可以帮助我们快速的查看任何字符串的底层字节.

> **注意**: 使用 `<<>>` 意味着我们正在告诉编译器, 符号内的元素是字节.

## 字符串列表
在 elixir 内部, 用字节序列而不是字符数组来表示字符串. elixir 同样具有一个 字符列表类型 (字符列表). 字符串使用 `"` 包围, 字符列表使用 `'` 包围.

不同之处在于, 字符列表中的每个值都是一个字符的 Unicode 码位, 而在二进制中, 码位被编码为 UTF-8, 让我深入看看:
```elixir
iex(5)> 'hełło'
[104, 101, 322, 322, 111]
iex(6)> "hełło" <> <<0>>
<<104, 101, 197, 130, 197, 130, 111, 0>>
```

`322` 是 `ł` 的 Unicode 码位, 但当它用 UTF-8 编码时, 为两个字节 `197`, `130`.

当在使用 elixir 编程时, 我们通常使用字符串, 而不是字符列表. 而内置支持字符列表的主要原因是在某些 erlang 模块中字符列表是必须的.

更多相关信息, 请查阅官方[入门指南](https://elixir-lang.org/getting-started/binaries-strings-and-char-lists.html).

## 字位 & 码位
码位只是简单的 Unicode 字符, 由一个或多个字节组成, 具体依赖于 UTF-8 编码. 在 US ASCII 字符集之外的字符将始终编码为多字节. 例如, 带有波浪符或者声调符 (`á, ñ, è`) 的拉丁文字通常被编码为2个字节. 来自亚洲语言的字符通常被编码为3到4个字节. 字位由多个码位组成, 表现为单个字符.

`String` 模块已经提供了2个函数来获取字位和码位, 分别是 `graphemes/1` 和 `codepoints/1`. 让我们观察一下:
```elixir
iex> string = "\u0061\u0301"
"á"

iex> String.codepoints string
["a", "́"]

iex> String.graphemes string
["á"]
```

## 字符串函数
让我们回顾一下 `String` 模块中一些重要和有用的函数. 在这里我们只覆盖可用函数的一小部分. 要查阅完整的函数列表, 请访问官方 [String](https://hexdocs.pm/elixir/String.html) 文档.

### `length/1`
返回字符串中字位数.
```elixir
String.length "Hello"
5
```

### `replace/3`
返回一个新的字符串, 用新的字符替换掉字符串中当前的字符.
```elixir
iex> String.replace("Hello", "e", "a")
"Hallo"
```

### `duplicate/2`
返回重复 n 次的新字符串.
```elixir
iex> String.duplicate("Oh my ", 3)
"Oh my Oh my Oh my "
```

### `split/2`
返回一个按照模式分割后的列表.
```elixir
iex> String.split("Hello World", " ")
["Hello", "World"]
```

## 练习
通过一个简单的练习来证明我们已经掌握了字符串的操作.

### 异序词
假设 A 和 B 是异序词, 如果通过重排 A 或者 B 可以保证他们一致, 例如:
* A = super
* B = perus

如果我们重排字符串 A 中的字符, 可以得到字符串 B, 反之亦然.

那么, 我们应该如何使用 elixir 判断两个字符串是异序词呢? 最简单的方法是将每一个字符串的字位按照字母顺序排序, 再检测列表是否相等. 让我们试试:
```elixir
defmodule Anagram do
  def anagrams?(a, b) when is_binary(a) and is_binary(b) do
    sort_string(a) == sort_string(b)
  end

  def sort_string(string) do
    string
    |> String.downcase()
    |> String.graphemes()
    |> Enum.sort()
  end
end
```

首先, 观察一下函数 `anagrams?/2`. 我们检测了接受到的参数是否为二进制. 在 elixir 中可以使用这种方式来检测参数是否为字符串.

之后, 我们调用一个按字母顺序排序的函数. 第一步是先将字符串小写, 接着, 使用 `String.graphemes/1` 获取字符串的字位列表, 最后, 使用 `Enum.sort/1` 对列表排序. 很直接, 不是吗?

让我们看下在 iex 中的输出:
```elixir
iex> Anagram.anagrams?("Hello", "ohell")
true

iex> Anagram.anagrams?("María", "íMara")
true

iex> Anagram.anagrams?(3, 5)
** (FunctionClauseError) no function clause matching in Anagram.anagrams?/2

    The following arguments were given to Anagram.anagrams?/2:

        # 1
        3

        # 2
        5

    iex:11: Anagram.anagrams?/2
```

正如你看到的那样, 最后的 `anagrams?` 调用导致了 `FunctionClauseError`, 这个异常告诉了我们, 在模块中不存在接收两个非二进制参数的函数, 这正是我们想要的, 只接收两个字符串, 而不是其他的类型.