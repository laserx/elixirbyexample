# Behaviours

我们在上一章节学习了类型规格的相关内容, 在本章节, 我们会学习如何要求模块实现这些规范. 在 elixir 中, 这个功能被称为 behaviours.

<!-- TOC -->

- [Behaviours](#behaviours)
    - [使用](#%E4%BD%BF%E7%94%A8)
    - [定义一个 behaviour](#%E5%AE%9A%E4%B9%89%E4%B8%80%E4%B8%AA-behaviour)
    - [使用 behaviours](#%E4%BD%BF%E7%94%A8-behaviours)

<!-- /TOC -->

## 使用

有时, 你希望让模块共享一个公共的 API, 在 elixir 中, 解决的办法是使用 behaviours. Behaviours 履行两个主要角色:

* 定义一组必须实现的函数
* 检测该组函数是否实质上被实现了

elixir 中内置了部分 behaviours, 例如 `GenServer`, 但是在本章节中, 我们主要关注于如何创建自己的 behaviours.

## 定义一个 behaviour

为了更好的理解 behaviours, 让我们为 worker 模块实现一个 behaviour. workers 预计将实现两个函数: `init/1` 和 `perform/2`.

为了达成这一点, 我们将使用 `@callback` 指令, 该指令语法类似于 `@spec`, 其定义了一个**必需的**函数; 对宏来说, 我们可以使用 `@macrocallback`, 让我们为 worker 指定 `init/1` 和 `perform/2` 函数.

```elixir
defmodule Example.Worker do
  @callback init(state :: term) :: {:ok, new_state :: term} | {:error, reason :: term}
  @callback perform(args :: term, state :: term) ::
              {:ok, result :: term, new_state :: term}
              | {:error, reason :: term, new_state :: term}
end
```

这里, 我们将 `init/1` 定义为接收任意值并返回一个 `{:ok, state}` 或者 `{:error, reason}` 元组. 这是一个非常标准的初始化. `perform/2` 函数将随着初始化, 接收一些 worker 的参数, 我们期望函数 `perform/2` 像 GenServers一样, 可以返回 `{:ok, result, state}` 或者 `{:error, reason, state}`.

## 使用 behaviours

现在, 我们已经定义了 behaviour, 可以使用它来创建共享相同的公共 API 的各种模块了. 添加 behaviour 至模块, 最简单的方法是使用 `@behaviour` 属性.

让我们为现有模块添加入刚刚创建的 behaviour, 使其下载远程文件, 并将其保存至本地:
```elixir
defmodule Example.Downloader do
  @behaviour Example.Worker

  def init(opts), do: {:ok, opts}

  def perform(url, opts) do
    url
    |> HTTPoison.get!()
    |> Map.fetch(:body)
    |> write_file(opts[:path])
    |> respond(opts)
  end

  defp write_file(:error, _), do: {:error, :missing_body}

  defp write_file({:ok, contents}, path) do
    path
    |> Path.expand()
    |> File.write(contents)
  end

  defp respond(:ok, opts), do: {:ok, opts[:path], opts}
  defp respond({:error, reason}, opts), do: {:error, reason, opts}
end
```

或者, 压缩一系列文件的 worker 如何? 这也没问题:
```elixir
defmodule Example.Compressor do
  @behaviour Example.Worker

  def init(opts), do: {:ok, opts}

  def perform(payload, opts) do
    payload
    |> compress
    |> respond(opts)
  end

  defp compress({name, files}), do: :zip.create(name, files)

  defp respond({:ok, path}, opts), do: {:ok, path, opts}
  defp respond({:error, reason}, opts), do: {:error, reason, opts}
end
```

虽然执行的工作不同, 但是面向共公的 API 不是, 并且, 利用这些模块的任意代码都可以与他们进行交互, 并知道他们会按照预期进行相应. 这使得我们具有创建任意数量的 workers, 处理不同的任务, 但都符合相同公共 API 的能力.

如果我们添加了一个 behaviour, 但是没有实现所有必须的函数, 则会引起编译时告警. 我们通过修改 `Example.Compressor`, 将其中的 `init/1` 函数移除, 观察一下:
```elixir
defmodule Example.Compressor do
  @behaviour Example.Worker

  def perform(payload, opts) do
    payload
    |> compress
    |> respond(opts)
  end

  defp compress({name, files}), do: :zip.create(name, files)

  defp respond({:ok, path}, opts), do: {:ok, path, opts}
  defp respond({:error, reason}, opts), do: {:error, reason, opts}
end
```

现在, 当我们编译代码时, 我们会收到如下的警告:
```elixir
lib/example/compressor.ex:1: warning: undefined behaviour function init/1 (for behaviour Example.Worker)
Compiled lib/example/compressor.ex
```

就是这样! 现在, 我们已经可以创建和共享 behaviour.