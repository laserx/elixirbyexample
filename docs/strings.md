# 字符串
字符串, 字符列表, 字位和码位.

<!-- TOC -->

- [字符串](#%E5%AD%97%E7%AC%A6%E4%B8%B2)
    - [字符串](#%E5%AD%97%E7%AC%A6%E4%B8%B2)
    - [字符串列表](#%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%88%97%E8%A1%A8)

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


When programming in Elixir, we usually use strings, not charlists. The charlist support is mainly included because it is required for some Erlang modules.