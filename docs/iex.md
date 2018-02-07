# IEx 辅助函数

<!-- TOC -->

- [IEx 辅助函数](#iex-%E8%BE%85%E5%8A%A9%E5%87%BD%E6%95%B0)
  - [概述](#%E6%A6%82%E8%BF%B0)
    - [自动补全](#%E8%87%AA%E5%8A%A8%E8%A1%A5%E5%85%A8)
    - [.iex.exs](#iexexs)
    - [h](#h)
    - [i](#i)
    - [r](#r)
    - [s](#s)
    - [t](#t)

<!-- /TOC -->

## 概述
当你开始使用 elixir 时, iex 是你的好伙伴. 它是一个 `REPL`, 同时包含了很多高级特性, 可以保证的在探索新的代码或者开发时更轻松. 我们将在本节介绍一些内置的辅助函数.

### 自动补全
当你在 shell 中工作时, 你可能经常发现自己在使用一个并不熟悉的新模块. 想要了解可以使用些什么, 自动完成可能会帮到你. 只需键入模块名称, 并跟着一个 `.`, 接着按 `tab`:
```shell
iex> Map. # press Tab
delete/2             drop/2               equal?/2
fetch!/2             fetch/2              from_struct/1
get/2                get/3                get_and_update!/3
get_and_update/3     get_lazy/3           has_key?/2
keys/1               merge/2              merge/3
new/0                new/1                new/2
pop/2                pop/3                pop_lazy/3
put/3                put_new/3            put_new_lazy/3
replace!/3           replace/3            split/2
take/2               to_list/1            update!/3
update/4             values/1
```

现在我们可以知道有多少个函数以及函数对应的元数了.

### .iex.exs
每次启动 iex 时, 它都会查找 `.iex.exs` 配置文件. 如果没有在当前目录找到, iex 就会使用用户家目录中的 (`~/.iex.exs`) 作为后备.

在 iex 启动时, 我们会使用这个文件中定义的配置项和代码. 例如, 我们想在 iex 中使用某些辅助函数, 我们可以编辑 `.iex.exs` 文件.

让我们添加一个具有几个辅助函数的模块:
```elixir
defmodule IExHelpers do
  def whats_this?(term) when is_nil(term), do: "Type: Nil"
  def whats_this?(term) when is_binary(term), do: "Type: Binary"
  def whats_this?(term) when is_boolean(term), do: "Type: Boolean"
  def whats_this?(term) when is_atom(term), do: "Type: Atom"
  def whats_this?(_term), do: "Type: Unknown"
end
```

现在, 当我们执行 iex, 我们从一开始就可以使用 `IExHelpers` 模块, 打开 iex 让我们试试:
```shell
$ iex
20 [9.1] [source] [64-bit] [smp:8:8] [async-threads:10] [hipe] [kernel-poll:false] [dtrace]

Interactive Elixir (1.5.2) - press Ctrl+C to exit (type h() ENTER for help)
iex> IExHelpers.whats_this?("a string")
"Type: Binary"
iex> IExHelpers.whats_this?(%{})
"Type: Unknown"
iex> IExHelpers.whats_this?(:test)
"Type: Atom"
```

正如我们看到的那样, 不需要做任何额外的 `require` 或者 `import` 动作来导入辅助函数, iex 为我们处理了.

### h

`h` 是 elixir 提供给我们最有用的工具之一. 由于语言对文档超一流的支持, 任何代码的文档都可以使用这个辅助函数获取. 看看下面的例子:
```shell
iex> h Enum
                                      Enum

Provides a set of algorithms that enumerate over enumerables according to the
Enumerable protocol.

┃ iex> Enum.map([1, 2, 3], fn(x) -> x * 2 end)
┃ [2, 4, 6]

Some particular types, like maps, yield a specific format on enumeration. For
example, the argument is always a {key, value} tuple for maps:

┃ iex> map = %{a: 1, b: 2}
┃ iex> Enum.map(map, fn {k, v} -> {k, v * 2} end)
┃ [a: 2, b: 4]

Note that the functions in the Enum module are eager: they always start the
enumeration of the given enumerable. The Stream module allows lazy enumeration
of enumerables and provides infinite streams.

Since the majority of the functions in Enum enumerate the whole enumerable and
return a list as result, infinite streams need to be carefully used with such
functions, as they can potentially run forever. For example:

┃ Enum.each Stream.cycle([1, 2, 3]), &IO.puts(&1)
```

现在可以将它和自动补全结合起来, 想象一下我们第一次对 `Map` 的探索:
```shell
iex> h Map
                                      Map

A set of functions for working with maps.

Maps are key-value stores where keys can be any value and are compared using
the match operator (===). Maps can be created with the %{} special form defined
in the Kernel.SpecialForms module.

iex> Map.
delete/2             drop/2               equal?/2
fetch!/2             fetch/2              from_struct/1
get/2                get/3                get_and_update!/3
get_and_update/3     get_lazy/3           has_key?/2
keys/1               merge/2              merge/3
new/0                new/1                new/2
pop/2                pop/3                pop_lazy/3
put/3                put_new/3            put_new_lazy/3
split/2              take/2               to_list/1
update!/3            update/4             values/1

iex> h Map.merge/2
                             def merge(map1, map2)

Merges two maps into one.

All keys in map2 will be added to map1, overriding any existing one.

If you have a struct and you would like to merge a set of keys into the struct,
do not use this function, as it would merge all keys on the right side into the
struct, even if the key is not part of the struct. Instead, use
Kernel.struct/2.

Examples

┃ iex> Map.merge(%{a: 1, b: 2}, %{a: 3, d: 4})
┃ %{a: 3, b: 2, d: 4}
```

正如我们看到的, 我们不仅仅可以查看模块提供了哪些函数, 还可以访问各个函数的文档, 其中很多包含用例演示.

### i
让我们利用新掌握的 `h` 来学习更多关于 `i` 辅助函数的知识:
```
iex> h i

                              def i(term \\ v(-1))

Prints information about the data type of any given term.

If no argument is given, the value of the previous expression is used.

## Examples

    iex> i(1..5)

Will print:

    Term
      1..5
    Data type
      Range
    Description
      This is a struct. Structs are maps with a __struct__ key.
    Reference modules
      Range, Map

iex> i Map
Term
  Map
Data type
  Atom
Module bytecode
  /usr/local/Cellar/elixir/1.6.1/bin/../lib/elixir/ebin/Elixir.Map.beam
Source
  /private/tmp/elixir-20180130-42559-1d5vx7w/elixir-1.6.1/lib/elixir/lib/map.ex
Version
  [234303838320399652689109978883853316190]
Compile options
  []
Description
  Use h(Map) to access its documentation.
  Call Map.module_info() to access metadata.
Raw representation
  :"Elixir.Map"
Reference modules
  Module, Atom
Implemented protocols
  IEx.Info, Inspect, String.Chars, List.Chars
```

现在我们可以看到一些 `Map` 的信息, 包括源代码的存储位置, 模块的引用等. 这对研究自定义的, 外部数据和新功能等十分有用.

孤立的看头部信息是无价值的, 但是在更高的层次我们可以手机到相关的信息:

* 它是原子数据类型
* 源代码的位置
* 版本和编译选项
* 基本描述
* 如何访问它
* 它引用了哪些模块

这给了我们很多帮助, 总比盲目的进行要好.

### r
如果需要重编译部分模块时, 我们可以使用 `r` 辅助函数.

比如, 修改了部分代码, 接着我们想执行一下刚刚修改的函数. 要做到这点, 我们需要保存修改, 并使用 `r` 进行重编译:
```elixir
iex> r MyProject
warning: redefining module MyProject (current version loaded from _build/dev/lib/my_project/ebin/Elixir.MyProject.beam)
  lib/my_project.ex:1

{:reloaded, MyProject, [MyProject]}
```

### s
使用 `s` 辅助函数, 可以获取模块或者函数的类型规格信息. 我们可以是用它来了解模块或者函数的可用功能:
```elixir
iex> s Map.merge/2
@spec merge(map(), map()) :: map()

# it also works on entire modules
iex> s Map
@spec get(map(), key(), value()) :: value()
@spec put(map(), key(), value()) :: map()
# ...
@spec get(map(), key()) :: value()
@spec get_and_update!(map(), key(), (value() -> {get, value()})) :: {get, map()} | no_return() when get: term()
```

### t
`t` 辅助函数揭示给定模块的可用类型:
```elixir
iex> t Map
@type key() :: any()
@type value() :: any()
```

现在我们知道 `Map` 在其实现中定义了 `key` 和 `value` 类型. 如果我们查看 `Map` 的源码:
```elixir
defmodule Map do
# ...
  @type key :: any
  @type value :: any
# ...
```

这是一个简单的示例, 说明了实现 `key` 和 `value` 的值可以是任意的类型, 了解这些信息这对我们很有用.

通过以上内置的帮助函数, 我们可以轻松探索源码, 同时学习更多程序是如何运行的细节. iex 是非常有用且强大的工具, 令开发人员如虎添翼. 通过 elixir 提供的这些工具, 使得研究和构建代码变得更有趣.