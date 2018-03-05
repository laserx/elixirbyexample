# 协议

在本章节, 我们将接触协议, 学习它是什么, 以及在 elixir 中如何使用.

<!-- TOC -->

- [协议](#%E5%8D%8F%E8%AE%AE)
    - [什么是协议](#%E4%BB%80%E4%B9%88%E6%98%AF%E5%8D%8F%E8%AE%AE)
    - [实现一个协议](#%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AA%E5%8D%8F%E8%AE%AE)

<!-- /TOC -->

## 什么是协议

那么, 什么是协议? 协议是实现 elixir 多态性的一种手段. erlang 的一个痛点是为一个新定义的类型扩展已存在的 API. 为了避免这一点, 在 elixir 中, 函数是基于值的类型动态调度的. elixir 内置了几种协议, 例如, `String.Chars` 协议对我们之前使用过的函数 `to_string/1` 负责. 让我们通过一个简单的例子仔细研究下 `to_string/1`:
```elixir
iex> to_string(5)
"5"
iex> to_string(12.4)
"12.4"
iex> to_string("foo")
"foo"
```

正如你看到的, 我们对多种类型调用函数, 并证明其对这些类型都有效. 如果我们对元组(或者任意没有实现 `String.Chars` 的类型)调用函数 `to_string/1` 会发生什么? 让我们看一下:
```elixir
to_string({:foo})
** (Protocol.UndefinedError) protocol String.Chars not implemented for {:foo}
    (elixir) lib/string/chars.ex:3: String.Chars.impl_for!/1
    (elixir) lib/string/chars.ex:17: String.Chars.to_string/1
```

我们收到了协议错误, 因为元组没有实现 `String.Chars`. 在下一节, 我们将为元组实现 `String.Chars` 协议.

## 实现一个协议

我们看到元组并未实现 `to_string/1`, 那让我们为元组添加一下. 要创建协议的实现, 我们要使用 `defimpl` 和要实现的协议, 并提供 `:for` 选项, 以及实现的类型. 让我们看一下:
```elixir
defimpl String.Chars, for: Tuple do
  def to_string(tuple) do
    interior =
      tuple
      |> Tuple.to_list()
      |> Enum.map(&Kernel.to_string/1)
      |> Enum.join(", ")

    "{#{interior}}"
  end
end
```

如果我们复制上述代码至 iex, 我们现在应该可以调用元组上的 `to_string/1`, 而且不会出现错误:
```shell
iex> to_string({3.14, "apple", :pie})
"{3.14, apple, pie}"
```

我们知道了如何实现一个协议, 但是如何定义一个新的协议呢? 下面的例子, 我们将实现 `to_atom/1`. 让我们看看如何使用 `defprotocol` 实现这一点:
```elixir
defprotocol AsAtom do
  def to_atom(data)
end

defimpl AsAtom, for: Atom do
  def to_atom(atom), do: atom
end

defimpl AsAtom, for: BitString do
  defdelegate to_atom(string), to: String
end

defimpl AsAtom, for: List do
  defdelegate to_atom(list), to: List
end

defimpl AsAtom, for: Map do
  def to_atom(map), do: List.first(Map.keys(map))
end
```

这里, 定义了我们的协议和其预期的函数 `to_atom/1`, 以及几种类型的实现. 现在, 我们可以使用该协议, 让我们在 iex 中尝试一下:
```shell
iex> import AsAtom
AsAtom
iex> to_atom("string")
:string
iex> to_atom(:an_atom)
:an_atom
iex> to_atom([1, 2])
:"\x01\x02"
iex> to_atom(%{foo: "bar"})
:foo
```

值得注意的是, 虽然在代码结构的最下方是映射, 但是其不与映射共享协议的实现. 映射是不可枚举的, 其不可访问.

正如我们所看到的, 协议是实现多态的有效方式.
