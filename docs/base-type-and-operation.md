# 基本类型和操作

## 整型和浮点型
在 elixir 中, 整型的行为类似于 python3, 当进行除法运算时, 返回值为浮点数.
```elixir
iex(1)> 3/3
1.0
iex(2)> 5/2
2.5
```

如果期望获得整除值可以使用 `div/2`, 获得余数可以使用 `rem/2`
```elixir
iex(1)> div(5, 3)
1
iex(2)> div 1_000_000, 3 # 整型可以用下划线格式化
333333
iex(3)> rem 3, 2
1
```

在上面的例子中 elixir 允许在类似的情况下不适用圆括号, 例如
```elixir
iex(1)> IO.puts div 5,2
2
:ok
```

elixir 内置支持二进制, 八进制和十六进制
```elixir
iex(1)> 0b11  # 二进制
3
iex(2)> 0o711 # 八进制
457
iex(3)> 0x1f  # 十六进制
31
```

elixir 中的浮点数小数点前至少有一位, 浮点数支持用 `e` 标记的指数值
```elixir
iex(1)> .23
** (SyntaxError) iex:1: syntax error before: '.'
iex(2)> 2.4e10
2.4e10
```

## 布尔类型
在 elixir 中, 支持 `true` 和 `false` 作为布尔值, 可以使用 `is_boolean/1` 判断值是否为布尔类型.
```elixir
iex(1)> is_boolean(true)
true
iex(2)> is_boolean(:false) # atom 会在接下来提及
true
```

elixir 支持 `||`, `&&` 和 `!` 以上的布尔操作. 其接受任意类型的参数
```elixir
iex(1)> 1 || 0
1
iex(2)>is_boolean(:not_boolean) || IO.puts "run here"
run here
:ok
```

同时, elixir 还支持 `and`, `or` 和 `not`, 以上的布尔操作, 第一个参数必须为布尔类型, 否则会抛出异常
```elixir
iex(1)> true and IO.puts "hello"
hello
:ok
iex(2)> false or is_atom(:atom)
true
iex(3)> not true
false
iex(4)> 1 and true
** (BadBooleanError) expected a boolean on left-side of "and", got: 1
```

## atom
原子类型在一种常量, 他的值就是自身的命名, 在某些语言中, 也称为 `symbols`

```elixir
iex(1)> :hello
:hello
iex(2)> :hello == :world
false
iex(5)> :true == true
true
iex(6)> is_boolean(:true)
true
iex(7)> is_boolean(true)
true
```

布尔型 `true` 和 `false` 实质上是 `atom`.

还有一种称为别名的结构体, 表现上使用大写字母开头, 其本质也是 `atom`

```elixir
iex(1)> is_atom(Python)
true
```

## 字符串

在 elixir 中, 字符串只能使用 `"` 声明, 并使用 utf-8 编码. 同时, 支持值的插入. 同时, 支持使用 `<>` 进行字符串拼接.
```elixir
iex(1)> "你好, elixir"
"你好, elixir"
iex(2)> name = "phoenix"
"phoenix"
iex(3)> "#{name} framework"
"phoenix framework"
iex(4)> "hello" <> " world"
"hello world"
```

字符串在 elixir 中本质是字符序列, 这部分稍后展开.

## 比较
elixir 提供了一下比较运算符: `==`, `===`, `!=`, `!==`, `<=`, `>=`, `<` 和 `>`.

```elixir
iex(1)> 1 > 2
false
iex(2)> 1 < 3
true
iex(3)> 1 == "1"
false
iex(4)> true == "true"
false
iex(5)> 1 === "1"
false
iex(6)> 3 == 3.0
true
iex(7)> 3 === 3.0
false
```

elixir 有一个比较重要的特性, 那就是任意两个类型之间可以比较, 按照以下优先级:

> number < atom < reference < function < port < pid < tuple < map < list < bitstring

该特性只需了解, 实际使用中并没有特别大的作用.