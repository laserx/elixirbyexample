# 并发

并发是 elixir 的主要卖点之一. 感谢 erlang 虚拟机(BEAM), elixir 中的并发比预期中简单. 并发模型依赖于 `actor`, 一个通过消息传递与其他进程通信的轻量级进程.

在本章节我们会专注于 elixir 内置的并发模块, 在下一章, 我们会探讨实现该功能的 otp behaviors.

## 进程

进程在 erlang 虚拟机中是轻量并且可以在所有 cpu 上运行. 虽然看起来像系统线程, 但他们更简单, 同时在一个 elixir 应用中有数千个并发进程并不罕见.

创建一个新的进程最简单的方法是使用 `spawn`, 其接收一个匿名或者命名函数, 当我们创建一个新的进程时, 其返回一个进程标识符或者 PID, 其在我们的应用中是唯一的标识. 

现在, 我们将创建一个模块, 并定义一个我们将要运行的函数:
```elixir
defmodule Example do
  def add(a, b) do
    IO.puts(a + b)
  end
end

iex> Example.add(2, 3)
5
:ok
```

异步执行该函数, 我们使用 `spawn/3`:
```elixir
iex> spawn(Example, :add, [2, 3])
5
#PID<0.80.0>
```

