# 管道操作符
管道操作符 `|>` 将一个表达式的结果传递给另一个表达式作为其第一个参数.

## 介绍
编程可能变得混乱. 实际上, 混乱的函数嵌套调用会导致逻辑难以梳理. 思考下面的嵌套函数:
```shell
foo(bar(baz(new_function(other_function()))))
```

在这里, 我们传递 `other_function/0` 的返回值给 `new_function/1`, 以及, `new_function/1` 传递至 `baz/1`, `baz/1` 至 `bar/1`, 最终, `bar/1` 至 `foo/1`. elixir 通过管道操作符, 解决了语法混乱的问题. 管道操作符 `|>` _将一个表达式的结果, 传递给下一个表达式_. 让我们再看看同样的代码使用管道操作符重写后的样子.
```shell
other_function() |> new_function() |> baz() |> bar() |> foo()
```
管道将左侧表达式的结果传递给右侧的表达式.

## 示例
下面的这组示例, 我们将使用 elixir 的 `String` 模块.

* 字符串序列化(松散化)
    ```elixir
    iex> "Elixir rocks" |> String.split()
    ["Elixir", "rocks"]
    ```

* 大写所有字符并序列化
    ```elixir
    iex> "Elixir rocks" |> String.upcase() |> String.split()
    ["ELIXIR", "ROCKS"]
    ```
* 句尾检测
    ```elixir
    iex> "elixir" |> String.ends_with?("ixir")
    true
    ```

## 最佳实践

如果函数的元数大于1, 那样必须使用括号包围参数. 这对 elixir 是无关紧要的, 但是对别程序员来说这容易误解你的代码. 这对管道操作符来说, 确实很重要. 比如, 看下我们第三个例子, 移除 `String.ends_with?/2` 的括号, 我们会收到如下的警告.
```elixir
iex> "elixir" |> String.ends_with? "ixir"
warning: parentheses are required when piping into a function call. For example:

  foo 1 |> bar 2 |> baz 3

is ambiguous and should be written as

  foo(1) |> bar(2) |> baz(3)

true
```