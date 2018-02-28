# 并发

并发是 elixir 的主要卖点之一. 感谢 erlang 虚拟机(BEAM), elixir 中的并发比预期中简单. 并发模型依赖于 `actor`, 一个通过消息传递与其他进程通信的轻量级进程.

在本章节我们会专注于 elixir 内置的并发模块, 在下一章, 我们会探讨实现该功能的 otp behaviors.

<!-- TOC -->

- [并发](#%E5%B9%B6%E5%8F%91)
  - [进程](#%E8%BF%9B%E7%A8%8B)
    - [消息传递](#%E6%B6%88%E6%81%AF%E4%BC%A0%E9%80%92)
    - [进程链接](#%E8%BF%9B%E7%A8%8B%E9%93%BE%E6%8E%A5)
    - [进程监控](#%E8%BF%9B%E7%A8%8B%E7%9B%91%E6%8E%A7)
  - [代理](#%E4%BB%A3%E7%90%86)
  - [任务](#%E4%BB%BB%E5%8A%A1)

<!-- /TOC -->

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

### 消息传递

进程间通信依赖于消息传递, 这有2个主要组件: `send/2` 和 `receive`. `send/2` 函数允许我们发送消息至 PID, 我们使用 `receive` 监听匹配的消息. 如果没有找到匹配, 则不间断持续执行.
```elixir
defmodule Example do
  def listen do
    receive do
      {:ok, "hello"} -> IO.puts("World")
    end

    listen
  end
end

iex> pid = spawn(Example, :listen, [])
#PID<0.108.0>

iex> send pid, {:ok, "hello"}
World
{:ok, "hello"}

iex> send pid, :ok
:ok
```

你可能注意到了 `listen/0` 函数是递归的, 这允许我们的进程处理多个消息. 如果没有递归, 我们的进程会在处理完第一条消息后退出.

### 进程链接

`spawn` 有一个已知的问题是, 当一个进程崩溃时, 主进程不知道子进程的状态. 为此, 我们需要使用 `spawn_link` 链接我们的进程. 两个已链接的进程会收到彼此的退出通知:
```elixir
defmodule Example do
  def explode, do: exit(:kaboom)
end

iex> spawn(Example, :explode, [])
#PID<0.66.0>

iex> spawn_link(Example, :explode, [])
** (EXIT from #PID<0.84.0>) shell process exited with reason: :kaboom
```

有时, 我们不希望已链接的进程崩溃当前进程, 为此, 我们需要使用 `Process.flag/2` 捕获退出通知. 其使用 erlang 的 `Process.flag/2` 函数作为 `trap_exit` 的标志位. 当捕获退出时(`trap_exit` 设置为 `true`), 退出信号将作为一个元组消息 `{:EXIT, from_pid, reason}` 被接收.
```elixir
defmodule Example do
  def explode, do: exit(:kaboom)

  def run do
    Process.flag(:trap_exit, true)
    spawn_link(Example, :explode, [])

    receive do
      {:EXIT, from_pid, reason} -> IO.puts("Exit reason: #{reason}")
    end
  end
end

iex> Example.run
Exit reason: kaboom
:ok
```

### 进程监控
如果我们不希望链接两个进程, 但是依然需要保持获知进程状态应该如何做? 为此, 我们可以使用 `spawn_monitor` 进程监控. 如果当我们监控的进程崩溃时, 可以接受到消息, 并不会使当前进程崩溃, 同时也无需显式的捕获退出信号.
```elixir
defmodule Example do
  def explode, do: exit(:kaboom)

  def run do
    {pid, ref} = spawn_monitor(Example, :explode, [])

    receive do
      {:DOWN, ref, :process, from_pid, reason} -> IO.puts("Exit reason: #{reason}")
    end
  end
end

iex> Example.run
Exit reason: kaboom
:ok
```

## 代理

代理是围绕后台进程维护状态的一种抽象. 我们可以从我们应用和节点中的其他进程访问代理. 代理的状态是通过函数的返回值设定的:
```elixir
iex> {:ok, agent} = Agent.start_link(fn -> [1, 2, 3] end)
{:ok, #PID<0.65.0>}

iex> Agent.update(agent, fn (state) -> state ++ [4, 5] end)
:ok

iex> Agent.get(agent, &(&1))
[1, 2, 3, 4, 5]
```

当我们为一个代理命名后, 可以使用命名代替 PID 来引用他:
```elixir
iex> Agent.start_link(fn -> [1, 2, 3] end, name: Numbers)
{:ok, #PID<0.74.0>}

iex> Agent.get(Numbers, &(&1))
[1, 2, 3]
```

## 任务
任务提供了一种在后台执行函数, 并稍后获取其返回值的方法. 在处理开销较大的操作而不阻塞应用程序的执行的情况下, 其十分有价值.
```elixir
defmodule Example do
  def double(x) do
    :timer.sleep(2000)
    x * 2
  end
end

iex> task = Task.async(Example, :double, [2000])
%Task{pid: #PID<0.111.0>, ref: #Reference<0.0.8.200>}

# Do some work

iex> Task.await(task)
4000
```
