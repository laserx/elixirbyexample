# IEx 辅助函数

<!-- TOC -->

- [IEx 辅助函数](#iex-%E8%BE%85%E5%8A%A9%E5%87%BD%E6%95%B0)
    - [概述](#%E6%A6%82%E8%BF%B0)
        - [自动补全](#%E8%87%AA%E5%8A%A8%E8%A1%A5%E5%85%A8)
        - [.iex.exs](#iexexs)
        - [`h`](#h)

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

### `h`

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

正如我们看到的, 我们不仅仅可以查看模块提供了哪些函数, 还可以访问各个函数的文档, 很多包含用例演示.
