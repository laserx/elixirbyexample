# mix
在更深入了解 elixir 前, 我们需要学习 mix 的相关知识. 如果你对 Ruby 比较熟悉, mix 相当于 `bundler`, `RubyGems` 和 `Rake` 的结合. 其是任何 elixir 项目的关键组成, 在这一章节. 我们会探讨 mix 的几个主要功能. 想了解所有的 mix 功能, 可以尝试执行 `mix help`.

到目前为止, 我们一直限制在 `iex` 中学习. 为了构建实质性的项目, 我们需要将代码划分成多个文件来高效的管理它们; mix 通过项目来达成这种需求.

<!-- TOC -->

- [mix](#mix)
    - [新项目](#%E6%96%B0%E9%A1%B9%E7%9B%AE)
    - [交互](#%E4%BA%A4%E4%BA%92)
    - [编译](#%E7%BC%96%E8%AF%91)
    - [依赖管理](#%E4%BE%9D%E8%B5%96%E7%AE%A1%E7%90%86)
    - [环境](#%E7%8E%AF%E5%A2%83)

<!-- /TOC -->

## 新项目
当我们准备创建新项目时, 使用 `mix new` 命令可以轻松的做到这一点. 该功能将生成项目的目录结构和必要的样本文件. 这很简单, 让我们尝试下:
```shell
$ mix new example
```

从输出我们可以观察到, mix 为我们创建了目录和一些样本文件:
```shell
* creating README.md
* creating .gitignore
* creating mix.exs
* creating config
* creating config/config.exs
* creating lib
* creating lib/example.ex
* creating test
* creating test/test_helper.exs
* creating test/example_test.exs
```

在本章节, 我们将注意力聚焦于 `mix.exs`. 在这个文件中, 我们会配置我们的应用, 依赖, 环境和版本. 在你最喜爱的编辑器中打开这个文件, 你会看到像这样的内容 (简洁起见, 注释被移除):
```elixir
defmodule Example.Mixfile do
  use Mix.Project

  def project do
    [
      app: :example,
      version: "0.1.0",
      elixir: "~> 1.5",
      start_permanent: Mix.env() == :prod,
      deps: deps()
    ]
  end

  def application do
    [
      extra_applications: [:logger]
    ]
  end

  defp deps do
    []
  end
end
```

我们看到的第一部分是项目. 在项目中, 我们定义了应用的名称(`app`), 指定了当前的版本(`version`), elixir 的版本(elixir), 最后是我们的依赖(`deps`).

`application` 部分用于生成下面涉及的应用的文件.

## 交互
在应用程序的上下文中使用 iex 是很必要的. 感谢 `mix` 让这一切变得简单. 让我们启动一个 iex 的新会话:
```shell
$ iex -S mix
```

用这种方式启动 iex 将会加载应用和依赖至当前的运行时中.

## 编译
mix 很聪明, 会在必要时自动编译你的变更. 但还是需要明确的编译整个项目. 在本节中, 我们会探讨如何编译项目和编译了什么.

要编译 mix 项目, 我们只需要在根目录执行 `mix compile`:
```shell
$ mix compile
```

我们的项目中没有什么实质性的代码, 所以输出并不令人愉悦, 但理应成功完成:
```
Compiled lib/example.ex
Generated example app
```

当编译项目时, mix 会创建一个 `_build` 的目录来承载我们的组件. 如果我们查看 `_build` 中的内容时, 我们应该会看到已编译的应用: `example.app`.

## 依赖管理
当前的项目还没有任何依赖, 我们马上就会为其添加, 这样, 让我们继续讨论依赖的定义以及拉取.

要添加一个新的依赖, 我们首先需要将其添加到我们的 `mix.exs` 中的 `deps` 部分. 我们的依赖列表由具有两个必备值和一个可选值的元组组成: 由原子定义的包名, 由字符串定义的版本, 和可选的选项.

对于这个例子, 让我们观察一个具有依赖的项目, 像 `phoenix_slim`:
```elixir
def deps do
  [
    {:phoenix, "~> 1.1 or ~> 1.2"},
    {:phoenix_html, "~> 2.3"},
    {:cowboy, "~> 1.0", only: [:dev, :test]},
    {:slime, "~> 0.14"}
  ]
end
```

你可以从上面的依赖中辨别出, 只有在开发环境和测试环境中才需要 `cowboy` 这个依赖.

当定义了项目的依赖后, 还需要执行最后一步: 拉取依赖. 类似于 `bundle install`:
```shell
$ mix deps.get
```

我们已经定义以及拉取了项目的依赖. 这样我们可以在需要的时候添加必要的依赖了.

## 环境
mix 和 `Bundler` 类似, 支持不同的环境. mix 自带三种环境:
* `:dev` - 默认环境.
* `:test` - `mix test` 会使用该环境. 下一章节讨论.
* `:prod` - 用于我们的生产环境.

当前环境可以使用 `Mix.env` 来获取. 当然, 可以通过修改环境 `MIX_ENV` 来调整当前环境:
```shell
$ MIX_ENV=prod mix compile
```
