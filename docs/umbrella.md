# 保护伞项目

有时, 一个项目可能会越来越巨大, 实质上真的很大. `mix` 构建工具允许代码分割成多个应用, 同时, 保障我们的 elixir 项目随着其增长更易于管理.

<!-- TOC -->

- [保护伞项目](#%E4%BF%9D%E6%8A%A4%E4%BC%9E%E9%A1%B9%E7%9B%AE)
    - [介绍](#%E4%BB%8B%E7%BB%8D)
    - [子项目](#%E5%AD%90%E9%A1%B9%E7%9B%AE)
    - [IEx](#iex)

<!-- /TOC -->

## 介绍
为了创建一个保护伞项目, 我们只需要像开始创建一个普通的 mix 项目一样, 只不过多传递一个 `--umbrella` 参数. 在这个例子中, 我们将开发一个机器学习工具包的 shell. 为什么是机器学习工具包项目? 为什么不呢? 其是由各种学习算法和工具函数组成的.
```shell
$ mix new machine_learning_toolkit --umbrella

* creating .gitignore
* creating README.md
* creating mix.exs
* creating apps
* creating config
* creating config/config.exs

Your umbrella project was created successfully.
Inside your project, you will find an apps/ directory
where you can create and host many apps:

    cd machine_learning_toolkit
    cd apps
    mix new my_app

Commands like "mix compile" and "mix test" when executed
in the umbrella project root will automatically run
for each application in the apps/ directory.
```

正如你从命令行看到的, mix 为我们创建了一个小型的项目框架, 其包含两个目录:
* `apps/` - 子项目的存储位置
* `config/` - 保护伞项目配置的存储位置

## 子项目
让我们切换到 `machine_learning_toolkit/apps` 目录下, 并且如下使用 mix 创建3个普通的应用程序:
```shell
$ mix new utilities

* creating README.md
* creating .gitignore
* creating mix.exs
* creating config
* creating config/config.exs
* creating lib
* creating lib/utilities.ex
* creating test
* creating test/test_helper.exs
* creating test/utilities_test.exs

Your Mix project was created successfully.
You can use "mix" to compile it, test it, and more:

    cd utilities
    mix test

Run "mix help" for more commands.


$ mix new datasets

* creating README.md
* creating .gitignore
* creating mix.exs
* creating config
* creating config/config.exs
* creating lib
* creating lib/datasets.ex
* creating test
* creating test/test_helper.exs
* creating test/datasets_test.exs

Your Mix project was created successfully.
You can use "mix" to compile it, test it, and more:

    cd datasets
    mix test

Run "mix help" for more commands.

$ mix new svm

* creating README.md
* creating .gitignore
* creating mix.exs
* creating config
* creating config/config.exs
* creating lib
* creating lib/svm.ex
* creating test
* creating test/test_helper.exs
* creating test/svm_test.exs

Your Mix project was created successfully.
You can use "mix" to compile it, test it, and more:

    cd svm
    mix test

Run "mix help" for more commands.
```

现在我们应该具有一个像下面这样的项目树:
```shell
$ tree
.
├── README.md
├── apps
│   ├── datasets
│   │   ├── README.md
│   │   ├── config
│   │   │   └── config.exs
│   │   ├── lib
│   │   │   └── datasets.ex
│   │   ├── mix.exs
│   │   └── test
│   │       ├── datasets_test.exs
│   │       └── test_helper.exs
│   ├── svm
│   │   ├── README.md
│   │   ├── config
│   │   │   └── config.exs
│   │   ├── lib
│   │   │   └── svm.ex
│   │   ├── mix.exs
│   │   └── test
│   │       ├── svm_test.exs
│   │       └── test_helper.exs
│   └── utilities
│       ├── README.md
│       ├── config
│       │   └── config.exs
│       ├── lib
│       │   └── utilities.ex
│       ├── mix.exs
│       └── test
│           ├── test_helper.exs
│           └── utilities_test.exs
├── config
│   └── config.exs
└── mix.exs
```

如果我们切换回保护伞项目的根, 我们可以调用所有的典型命令, 例如编译. 因为所有的子项目都是普通的应用程序, 你可以切换至他们各自的目录下, 像平常一样, 执行 mix 所提供的功能.
```shell
$ mix compile

==> svm
Compiled lib/svm.ex
Generated svm app

==> datasets
Compiled lib/datasets.ex
Generated datasets app

==> utilities
Compiled lib/utilities.ex
Generated utilities app

Consolidated List.Chars
Consolidated Collectable
Consolidated String.Chars
Consolidated Enumerable
Consolidated IEx.Info
Consolidated Inspect
```

## IEx
你可能认为与保护伞项目下的应用程序交互可能会有些许的不同. 不管你信不信, 这是错的! 如果我们切换至项目的顶级目录, 同时, 通过 `iex -S mix` 启动 iex, 我们可以和平时一样与保护伞项目下的所有项目进行交互. 为了下面的简单演示, 让我们调整下 `apps/datasets/lib/datasets.ex` 中的内容.
```elixir
defmodule Datasets do
  def hello do
    IO.puts("Hello, I'm the datasets")
  end
end
```

```shell
$ iex -S mix
Erlang/OTP 20 [erts-9.1] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false] [dtrace]

==> datasets
Compiled lib/datasets.ex
Consolidated List.Chars
Consolidated Collectable
Consolidated String.Chars
Consolidated Enumerable
Consolidated IEx.Info
Consolidated Inspect
Interactive Elixir (1.5.2) - press Ctrl+C to exit (type h() ENTER for help)

iex> Datasets.hello
Hello, I'm the datasets
:ok
```