# OTP 并发

我们已经研究了 elixir 的并发抽象, 但是有时我们需要更高的控制能力, 为此, 我们转向研究 elixir 内置的 OTP 行为.

在本章节我们专注讨论最庞大的部分: GenServers.

<!-- TOC -->

- [OTP 并发](#otp-%E5%B9%B6%E5%8F%91)
    - [GenServer](#genserver)
    - [同步函数](#%E5%90%8C%E6%AD%A5%E5%87%BD%E6%95%B0)
    - [异步函数](#%E5%BC%82%E6%AD%A5%E5%87%BD%E6%95%B0)

<!-- /TOC -->

## GenServer
OTP server 是一个模块, 其由实现了一系列回调的 GenServer 行为组成. 在基本面上, 一个 GenServer 是一个循环, 该循环每次迭代处理一个请求, 并传递更新状态.

为了演示 GenServer API, 我们将实现一个基础队列来存储和获取值.

要执行我们的 GenServer, 需要启动和初始化它. 在大多数场景下, 我们希望链接进程, 所以我们使用 `GenServer.start_link/3`. 我们传入启动的 GenServer 模块名称, 初始化参数和一套 GenServer 的选项值. 初始化参数会传递至 `GenServer.init/1`, 并通过其的返回值设置初始化状态. 在我们下面的示例中, 参数 (state) 将作为我们的初始化状态:
```elixir
defmodule SimpleQueue do
  use GenServer

  @doc """
  Start our queue and link it.  This is a helper function
  """
  def start_link(state \\ []) do
    GenServer.start_link(__MODULE__, state, name: __MODULE__)
  end

  @doc """
  GenServer.init/1 callback
  """
  def init(state), do: {:ok, state}
end
```

## 同步函数
通常的来说, 需要与 GenServers 进行同步交互, 调用一个函数, 等待它的返回. 处理同步请求, 我们需要实现 ` GenServer.handle_call/3` 回调, 其接收: 请求, 调用者的 PID 和现有的状态; 其预期返回一个元组: `{:reply, response, state}`.

通过模式匹配, 我们可以为不同的请求和状态定义回调. 完整的可接收的返回值列表可以在 [`GenServer.handle_call/3`](https://hexdocs.pm/elixir/GenServer.html#c:handle_call/3) 文档中找到.

为了演示同步请求, 让我们添加显示当前队列和移除值的功能:
```elixir
defmodule SimpleQueue do
  use GenServer

  ### GenServer API

  @doc """
  GenServer.init/1 callback
  """
  def init(state), do: {:ok, state}

  @doc """
  GenServer.handle_call/3 callback
  """
  def handle_call(:dequeue, _from, [value | state]) do
    {:reply, value, state}
  end

  def handle_call(:dequeue, _from, []), do: {:reply, nil, []}

  def handle_call(:queue, _from, state), do: {:reply, state, state}

  ### Client API / Helper functions

  def start_link(state \\ []) do
    GenServer.start_link(__MODULE__, state, name: __MODULE__)
  end

  def queue, do: GenServer.call(__MODULE__, :queue)
  def dequeue, do: GenServer.call(__MODULE__, :dequeue)
end
```

让我们开始执行我们的 `SimpleQueue`, 并测试我们新的 `dequeue` 功能:
```elixir
iex> SimpleQueue.start_link([1, 2, 3])
{:ok, #PID<0.90.0>}
iex> SimpleQueue.dequeue
1
iex> SimpleQueue.dequeue
2
iex> SimpleQueue.queue
[3]
```

## 异步函数
异步请求是通过 `handle_cast/2` 回调来处理的. 其很像 `handle_call/3`, 但是不接收调用者, 同时不返回预期值.

我们将实现 `enqueue` 功能为异步, 更新队列但不阻塞当前的程序执行:
```elixir
defmodule SimpleQueue do
  use GenServer

  ### GenServer API

  @doc """
  GenServer.init/1 callback
  """
  def init(state), do: {:ok, state}

  @doc """
  GenServer.handle_call/3 callback
  """
  def handle_call(:dequeue, _from, [value | state]) do
    {:reply, value, state}
  end

  def handle_call(:dequeue, _from, []), do: {:reply, nil, []}

  def handle_call(:queue, _from, state), do: {:reply, state, state}

  @doc """
  GenServer.handle_cast/2 callback
  """
  def handle_cast({:enqueue, value}, state) do
    {:noreply, state ++ [value]}
  end

  ### Client API / Helper functions

  def start_link(state \\ []) do
    GenServer.start_link(__MODULE__, state, name: __MODULE__)
  end

  def queue, do: GenServer.call(__MODULE__, :queue)
  def enqueue(value), do: GenServer.cast(__MODULE__, {:enqueue, value})
  def dequeue, do: GenServer.call(__MODULE__, :dequeue)
end
```

让我们使用一下我们的新功能:
```elixir
iex> SimpleQueue.start_link([1, 2, 3])
{:ok, #PID<0.100.0>}
iex> SimpleQueue.queue
[1, 2, 3]
iex> SimpleQueue.enqueue(20)
:ok
iex> SimpleQueue.queue
[1, 2, 3, 20]
```

相关详细信息, 请查阅官方 [`GenServer`](https://hexdocs.pm/elixir/GenServer.html#content) 文档.
