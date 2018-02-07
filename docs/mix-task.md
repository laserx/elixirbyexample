# 自定义 mix 任务
为 elixir 项目创建自定义 mix 任务.

<!-- TOC -->

- [自定义 mix 任务](#%E8%87%AA%E5%AE%9A%E4%B9%89-mix-%E4%BB%BB%E5%8A%A1)
    - [简介](#%E7%AE%80%E4%BB%8B)
    - [设置](#%E8%AE%BE%E7%BD%AE)
    - [自定义 mix 任务](#%E8%87%AA%E5%AE%9A%E4%B9%89-mix-%E4%BB%BB%E5%8A%A1)
    - [执行 mix 任务](#%E6%89%A7%E8%A1%8C-mix-%E4%BB%BB%E5%8A%A1)

<!-- /TOC -->

## 简介
通过添加自定义 mix 任务来扩展 elixir 应用功能的想法并不罕见. 在我们学习如何为项目创建特定的 mix 任务前, 让我们看一个已存在的示例:
```shell
$ mix phoenix.new my_phoenix_app

* creating my_phoenix_app/config/config.exs
* creating my_phoenix_app/config/dev.exs
* creating my_phoenix_app/config/prod.exs
* creating my_phoenix_app/config/prod.secret.exs
* creating my_phoenix_app/config/test.exs
* creating my_phoenix_app/lib/my_phoenix_app.ex
* creating my_phoenix_app/lib/my_phoenix_app/endpoint.ex
* creating my_phoenix_app/test/views/error_view_test.exs
...
```

从上面的命令行指令可以看出, `Phoenix` 框架拥有自定义的 mix 任务用以生成新的项目. 我们是否可以为自己的项目创建类似的功能呢? 那么, 好消息是, 我们可以, 同时, elixir 使我们很容易做到这点.

## 设置
首先, 创建一个基础 mix 应用.
```shell
$ mix new hello

* creating README.md
* creating .gitignore
* creating mix.exs
* creating config
* creating config/config.exs
* creating lib
* creating lib/hello.ex
* creating test
* creating test/test_helper.exs
* creating test/hello_test.exs

Your Mix project was created successfully.
You can use "mix" to compile it, test it, and more:

cd hello
mix test

Run "mix help" for more commands.
```

现在, 在 mix 为我们生成的 `lib/hello.ex` 文件中创建一个简单的函数, 使其输出 `Hello, World!`.
```elixir
defmodule Hello do
  @doc """
  Output's `Hello, World!` everytime.
  """
  def say do
    IO.puts("Hello, World!")
  end
end
```

## 自定义 mix 任务
让我们创建自定义的 mix 任务. 第一步, 创建一个文件夹以及一个文件, 路径是 `hello/lib/mix/tasks/hello.ex`. 在这个文件中, 插入7行 elixir 代码.
```elixir
defmodule Mix.Tasks.Hello do
  use Mix.Task

  @shortdoc "Simply runs the Hello.say/0 command."
  def run(_) do
    # calling our Hello.say() function from earlier
    Hello.say()
  end
end
```

注意我们在 `defmodule` 语句的开始位置使用了 `Mix.Tasks`, 在它后面的是命令行中调用的命令名称. 在第二行中, 我们引入了 `use Mix.Task`, 其将 `Mix.Task` 的 behaviour 带入当前的命名空间中. 然后, 我们声明了一个忽略任意参数的 `run` 函数, 在函数中, 调用了 `Hello` 模块的 `say` 函数.

## 执行 mix 任务
让我们检查一下 mix 任务. 只要我们在项目目录中, 它应该能正常工作. 在命令行中, 执行 `mix hello`, 我们应该可以看到:
```shell
$ mix hello
Hello, World!
```

mix 默认对使用者十分友好. 它知道任何人都有拼写错误的可能, 所以, 其使用了种称为字符串模糊匹配的技术来给出建议:
```shell
$ mix hell
** (Mix) The task "hell" could not be found. Did you mean "hello"?
```

你是否注意到我们引入了一个新的模块属性 `@shortdoc`? 这个属性在分发应用的时候十分有用, 例如当用户在命令行中执行 `mix help` 时.
```shell
$ mix help

mix app.start         # Starts all registered apps
...
mix hello             # Simply calls the Hello.say/0 function.
...
```