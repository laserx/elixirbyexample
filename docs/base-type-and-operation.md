# 基本类型和操作符

## 整型和浮点型
在 elixir 中, 整型的行为类似于 python3, 当进行除法运算时, 返回值为浮点数.
```
iex(1)> 3/3
1.0
iex(2)> 5/2
2.5
```

如果期望获得整除值
```
iex(1)> div(5, 3)
1
iex(2)> div 1_000_000, 3 # 整型可以用下划线格式化
333333
```

在上面的例子中 elixir 允许在类似的情况下不适用圆括号, 例如
```
iex(1)> IO.puts div 5,2
2
:ok
```

elixir 内置支持二进制, 八进制和十六进制
```
iex(1)> 0b11  # 二进制
3
iex(2)> 0o711 # 八进制
457
iex(3)> 0x1f  # 十六进制
31
```

elixir 中的浮点数小数点前至少有一位, 浮点数支持用 `e` 标记的指数值
```
iex(1)> .23
** (SyntaxError) iex:1: syntax error before: '.'
iex(2)> 2.4e10
2.4e10
```

## 布尔类型
在 elixir 中, 支持 `true` 和 `false` 作为布尔值, 可以使用 `is_boolean/1` 判断值是否为布尔类型.
```
iex(1)> is_boolean(true)
true
iex(2)> is_boolean(:false) # atom 会在接下来提及
true
```

## atom