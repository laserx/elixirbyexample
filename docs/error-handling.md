# 异常处理
虽然返回 `{:error, reason}` 元组更为常见, 但 elixir 支持异常. 在本章中, 我们会探索如何处理错误以及我们可用的不同处理机制.

一般来说, 在 elixir 约定是创建的函数 (`example/1`) 能够返回 `{:ok, result}` 和 `{:error, reason}`, 同时会有另一个函数与其类似 (`example!/1`), 而这个函数返回数据或者抛出异常.

本章我们着重聚焦于后者.

<!-- TOC -->

- [异常处理](#%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86)
    - [异常处理](#%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86)
    - [after](#after)
    - [创建异常类型](#%E5%88%9B%E5%BB%BA%E5%BC%82%E5%B8%B8%E7%B1%BB%E5%9E%8B)
    - [Throws](#throws)
    - [退出](#%E9%80%80%E5%87%BA)

<!-- /TOC -->

## 异常处理
在学习处理异常之前, 我们需要先掌握如何抛出异常, 最简单的方法是使用 `raise/1`:
```elixir
iex> raise "Oh no!"
** (RuntimeError) Oh no!
```

如果希望指定异常类型和消息, 可以使用 `raise/2`:
```elixir
iex> raise ArgumentError, message: "the argument value is invalid"
** (ArgumentError) the argument value is invalid
```

当知道可能发生异常时, 我们可以使用 `try/rescue` 和模式匹配来处理:
```elixir
iex> try do
...>   raise "Oh no!"
...> rescue
...>   e in RuntimeError -> IO.puts("An error occurred: " <> e.message)
...> end
An error occurred: Oh no!
:ok
```

可以在一次 `rescue` 中匹配多种异常:
```elixir
try do
  opts
  |> Keyword.fetch!(:source_file)
  |> File.read!()
rescue
  e in KeyError -> IO.puts("missing :source_file option")
  e in File.Error -> IO.puts("unable to read source file")
end
```

## after
有时, 可能需要在无论异常与否, 都需要执行某些动作在 `try/rescue` 之后. 为此, 可以使用 `try/after`. 如果你熟悉 ruby, 这和 `begin/rescue/ensure` 或 java 中的 `try/catch/finally` 类似:
```elixir
iex> try do
...>   raise "Oh no!"
...> rescue
...>   e in RuntimeError -> IO.puts("An error occurred: " <> e.message)
...> after
...>   IO.puts "The end!"
...> end
An error occurred: Oh no!
The end!
:ok
```

这经常使用在文件或者连接打开的情况, 确保其正常关闭:
```elixir
{:ok, file} = File.open("example.json")

try do
  # Do hazardous work
after
  File.close(file)
end
```

## 创建异常类型
虽然 elixir 内置了一些类似于 `RuntimeError` 这样的错误类型, 但当我们需要某些特定的类型时, elixir 给与我们创建新异常类型的的能力. 用宏 `defexception/1` 创建一个新的异常类型是很轻松的, 其接受一个 `:message` 选项, 并将其设置为默认的异常消息:
```elixir
defmodule ExampleError do
  defexception message: "an example error has occurred"
end
```

让我们使用一下新创建的异常:
```elixir
iex> try do
...>   raise ExampleError
...> rescue
...>   e in ExampleError -> e
...> end
%ExampleError{message: "an example error has occurred"}
```

## Throws
另一种 elixir 处理错误的机制是 `throw` 和 `catch`. 在实践中, 这种处理方式很少出现在新的 elixir 代码中, 但了解和理解这种方式是很重要的.

函数 `throw/1` 提供给我们以特定值退出执行的能力, 同时, 我们可以 `catch` 并使用该值:
```elixir
iex> try do
...>   for x <- 0..10 do
...>     if x == 5, do: throw(x)
...>     IO.puts(x)
...>   end
...> catch
...>   x -> "Caught: #{x}"
...> end
0
1
2
3
4
"Caught: 5"
```

如上所述, `throw/catch` 是非常不常见的, 当库无法提供足够的 API 时, 通常以 `stopgaps` 的方式退出.

## 退出
最后一种 elixir 提供的异常机制是退出. 退出信号发生在一个进程死亡时, 是 elixir 容错性重要的一部分.

要显式的退出, 可以使用 `exit/1`:
```elixir
iex> spawn_link fn -> exit("oh no") end
** (EXIT from #PID<0.101.0>) evaluator process exited with reason: "oh no"
```

虽然可以使用 `try/catch` 捕获退出信号但这样做是十分罕见的. 在几乎所有的情况下, 由 `supervisor` 处理进程退出是最有利的:
```elixir
iex> try do
...>   exit "oh no!"
...> catch
...>   :exit, _ -> "exit blocked"
...> end
"exit blocked"
```