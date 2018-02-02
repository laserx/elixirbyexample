# 基础

入门, 基础数据类型和操作.

<!-- TOC -->

- [基础](#%E5%9F%BA%E7%A1%80)
    - [入门](#%E5%85%A5%E9%97%A8)
        - [安装 elixir](#%E5%AE%89%E8%A3%85-elixir)
        - [体验交互模式](#%E4%BD%93%E9%AA%8C%E4%BA%A4%E4%BA%92%E6%A8%A1%E5%BC%8F)
    - [基本数据类型](#%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)
        - [整型](#%E6%95%B4%E5%9E%8B)
        - [浮点](#%E6%B5%AE%E7%82%B9)
        - [布尔](#%E5%B8%83%E5%B0%94)
        - [原子](#%E5%8E%9F%E5%AD%90)
        - [字符串](#%E5%AD%97%E7%AC%A6%E4%B8%B2)
    - [基础运算](#%E5%9F%BA%E7%A1%80%E8%BF%90%E7%AE%97)
        - [四则运算](#%E5%9B%9B%E5%88%99%E8%BF%90%E7%AE%97)
        - [布尔运算](#%E5%B8%83%E5%B0%94%E8%BF%90%E7%AE%97)
        - [比较](#%E6%AF%94%E8%BE%83)
        - [字符串模板](#%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%A8%A1%E6%9D%BF)
        - [字符串拼接](#%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%8B%BC%E6%8E%A5)

<!-- /TOC -->

## 入门

### 安装 elixir
不同操作系统的安装说明可以在[官网](elixir-lang.org)的[安装指南](https://elixir-lang.org/install.html)中找到.

安装 elixir 之后, 你可以简单的确认一下安装的版本.
```shell
% elixir -v
Erlang/OTP 20 [erts-9.2] [source] [64-bit] [smp:4:4] [ds:4:4:10] [async-threads:10] [hipe] [kernel-poll:false] [dtrace]

Elixir 1.6.1 (compiled with OTP 20)
```

### 体验交互模式

elixir 自带一个交互工具 IEx, 可以让我们执行 elixir 表达式.

启动它, 只需要执行 iex:
```elixir
% iex
Erlang/OTP 20 [erts-9.2] [source] [64-bit] [smp:4:4] [ds:4:4:10] [async-threads:10] [hipe] [kernel-poll:false] [dtrace]

Interactive Elixir (1.6.1) - press Ctrl+C to exit (type h() ENTER for help)
iex>
```

让我们试着在 iex 中输入几个简单的表达式:
```elixir
iex> 2+3
5
iex> 2+3 == 5
true
iex> String.length("the quick brown fox jumps over the lazy dog")
43
```

如果你还不理解这些表达式, 请不要担心, 但我们希望你能理解大体的含义.

## 基本数据类型

### 整型
```elixir
iex> 255
255
```

elixir 内置支持二进制, 八进制和十六进制数字.
```
iex> 0b0110
6
iex> 0o644
420
iex> 0x1F
31
```

### 浮点
在 elixir 中, 浮点数至少需要一位整数位和一位小数位, 其具有64位双精度, 同时, 支持科学计数法.
```elixir
iex> 3.14
3.14
iex> .14
** (SyntaxError) iex:2: syntax error before: '.'
iex> 1.0e-10
1.0e-10
```

### 布尔
elixir 使用 `true` 和 `false` 表示布尔值; 除了 `false` 和 `nil` 其他所有值都是真值.
```elixir
iex> true
true
iex> false
false
```

### 原子
原子是一个常量, 它的值就是名字本身. 如果你熟悉 `Ruby`, 原子和 `Ruby` 中的符号是一个意思. 
```elixir
iex> :foo
:foo
iex> :foo == :bar
false
```

布尔值 `true` 和 `false` 对照 原子 `:true` 和 `:false` 相等.
```elixir
iex> true |> is_atom 
true
iex> :true |> is_boolean
true
iex> :true === true
true
```

在 elixir 中, 模块的名称也是原子. 例如 `MyApp.MyModule` 就是一个有效的原子, 不论该模块是否声明过.
```elixir
iex> is_atom(MyApp.MyModule)
true
```

原子同样可以用来引用 erlang 中的模块, 包括内置的模块.
```elixir
iex> :crypto.strong_rand_bytes 3
<<23, 104, 108>>
```

### 字符串
elixir 中的字符串使用 `"` 声明, 并使用 utf-8 编码.
```elixir
iex> "Hello"
"Hello"
iex> "dziękuję"
"dziękuję"
```

字符串支持换行符以及转义字符
```elixir
iex> "foo
...> bar"
"foo\nbar"
iex> "foo\nbar"
"foo\nbar"
```

elixir 也包含了更复杂的数据类型, 我们会在接下来集合和函数的教程中学习到关于这方面的知识.

## 基础运算

### 四则运算

elixir 支持 `+`, `-`, `*` 和 `/` 四则运算, 但有一点需要铭记, `/` 的返回值只可能是浮点类型.
```elixir
iex> 2 + 2
4
iex> 2 - 1
1
iex> 2 * 5
10
iex> 10 / 5
2.0
```

当你需要获取整数除法值或者余数时, elixir 提供了两个有用的函数来实现这个功能:
```
iex> div(10, 5)
2
iex> rem(10, 3)
1
```

### 布尔运算
elixir 提供了 `||`, `&&` 和 `!` 布尔操作符. 以上支持任意类型:
```elixir
iex> -20 || true
-20
iex> false || 42
42

iex> 42 && true
true
iex> 42 && nil
nil

iex> !42
false
iex> !false
true
```

同时, 还有三个额外的操作符, 其第一个参数必须是布尔类型的(`true` 或者 `false`):
```elixir
iex> true and 42
42
iex> false or true
true
iex> not false
true
iex> 42 and true
** (ArgumentError) argument error: 42
iex> not 42
** (ArgumentError) argument error
```

### 比较
elixir 内置了 `==`, `!=`, `===`, `!==`, `<=`, `>=`, `<` 和 `>` 以上我们常用的比较操作符.
```elixir
iex> 1 > 2
false
iex> 1 != 2
true
iex> 2 == 2
true
iex> 2 <= 3
true
```

为了严格比较整数和浮点数, 使用 `===`:
```elixir
iex> 2 == 2.0
true
iex> 2 === 2.0
false
```

elixir 中一个重要的特性是任意类型的数据可以比较, 这一点在比较中比较重要, 我们不需要记住其排序, 但是有必要了解这一点:

> number < atom < reference < function < port < pid < tuple < map < list < bitstring

这可能导致一些有趣的比较结果, 无法在别的语言中体会到:
```elixir
iex> :hello > 999
true
iex> {:hello, :world} > [1, 2, 3]
false
```

### 字符串模板
如果你使用过 `Ruby`, 在 elixir 中字符串模板和其类似:
```elixir
iex> name = "Sean"
iex> "Hello #{name}"
"Hello Sean"
```

### 字符串拼接
字符串使用 `<>` 来拼接:
```elixir
iex> name = "Sean"
iex> "Hello " <> name
"Hello Sean"
```