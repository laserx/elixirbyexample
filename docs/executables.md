# 可执行
在 elixir 中构建可执行文件, 可以使用 `escript`. `escript` 生成可以在安装了 erlang 的任意系统中执行的可执行文件.

<!-- TOC -->

- [可执行](#%E5%8F%AF%E6%89%A7%E8%A1%8C)
  - [入门](#%E5%85%A5%E9%97%A8)
  - [解析参数](#%E8%A7%A3%E6%9E%90%E5%8F%82%E6%95%B0)
  - [构建](#%E6%9E%84%E5%BB%BA)

<!-- /TOC -->

## 入门
使用 escript 创建一个可执行文件我们只需要做几件事: 实现 `main/1`函数, 更新 `Mixfile`.

我们将创建一个模块作为可执行文件的入口, 在这里我们会实现 `main/1` 函数:
```elixir
defmodule ExampleApp.CLI do
  def main(args \\ []) do
    # Do stuff
  end
end
```

下一步, 我们需要更新 `Mixfile` 使其包含 `:escript`, 并指定项目中的 `:main_module`:
```elixir
defmodule ExampleApp.Mixfile do
  def project do
    [app: :example_app, version: "0.0.1", escript: escript()]
  end

  defp escript do
    [main_module: ExampleApp.CLI]
  end
end
```

## 解析参数
随着应用程序创建完成, 我们可以继续进行解析命令行参数的工作. 使用 elixir 中的 `OptionParser.parse/2` 和 `:switches` 选项表明标志为布尔类型:
```elixir
defmodule ExampleApp.CLI do
  def main(args \\ []) do
    args
    |> parse_args
    |> response
    |> IO.puts()
  end

  defp parse_args(args) do
    {opts, word, _} =
      args
      |> OptionParser.parse(switches: [upcase: :boolean])

    {opts, List.to_string(word)}
  end

  defp response({opts, word}) do
    if opts[:upcase], do: String.upcase(word), else: word
  end
end
```

## 构建
当我们使用 escript 配置完应用程序后, 使用 mix 构建可执行文件是一件十分轻松的事情:
```shell
$ mix escript.build
```

让我们试一试:
```shell
$ ./example_app --upcase Hello
HELLO

$ ./example_app Hi
Hi
```

好了, 我们使用 escript 构建了我们第一个可执行程序.
