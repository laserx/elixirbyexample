# 测试
测试是软件开发中重要的组成之一. 在本章, 我们将关注如何使用 ExUnit 测试 elixir 代码, 以及一些相关的最佳实践.

<!-- TOC -->

- [测试](#%E6%B5%8B%E8%AF%95)
    - [ExUnit](#exunit)
        - [assert](#assert)
        - [refute](#refute)
        - [assert_raise](#assertraise)
        - [assert_receive](#assertreceive)
        - [capture_io & capture_log](#captureio-capturelog)
    - [Test Setup](#test-setup)
    - [Mocking](#mocking)

<!-- /TOC -->

## ExUnit
ExUnit 是 elixir 内置的测试框架, 其包含了测试我们代码所需的一切. 在开始之前, 需要我们注意的是, 测试是以 elixir 脚本形式实现的, 所以我们需要使用 `.exs` 作为其的文件扩展名. 执行测试之前, 我们需要使用 `ExUnit.start()` 启动 ExUnit, 这通常已经包含在 `test/test_helper.exs` 文件中.

在我们生成上一章的示例项目时, mix 已经帮助我们创建了一个简单的测试, 位于 `test/example_test.exs`:
```elixir
defmodule ExampleTest do
  use ExUnit.Case
  doctest Example

  test "greets the world" do
    assert Example.hello() == :world
  end
end
```

通过执行 `mix test` 测试我们的项目. 可以看到类似的输出:
```shell
..

Finished in 0.03 seconds
2 tests, 0 failures
```

为什么会输出2个测试呢. 让我们看看 `lib/example.ex`. mix 通过 `doctest` 为我们创建了额外的一个测试.
```elixir
defmodule Example do
  @moduledoc """
  Documentation for Example.
  """

  @doc """
  Hello world.

  ## Examples

      iex> Example.hello
      :world

  """
  def hello do
    :world
  end
end
```

### assert
如果你之间编写过测试, 那你可能对 `assert` 比较熟悉; 在某些框架下, `should` 或者 `expect` 具有和 `assert` 类似的功能.

我们使用 `assert` 宏时, 判断表达式是否为 `true`. 如果不是这样, 就会抛出异常, 同时, 测试失败. 为了让测试失败, 我们改写一下代码, 并执行 `mix test`:
```elixir
defmodule ExampleTest do
  use ExUnit.Case
  doctest Example

  test "greets the world" do
    assert Example.hello() == :word
  end
end
```

现在我们会看到一个不一样的输出:
```shell
  1) test greets the world (ExampleTest)
     test/example_test.exs:5
     Assertion with == failed
     code:  assert Example.hello() == :word
     left:  :world
     right: :word
     stacktrace:
       test/example_test.exs:6 (test)

.

Finished in 0.03 seconds
2 tests, 1 failures
```
ExUnit 会向我们报告, 失败的断言是哪一个, 期望值是什么, 实际值又是什么.

### refute
`refute` 与 `assert` 的关系正像 `unless` 与 `if` 一样. 当你想确保语句返回永远是 `false` 时, 使用 `refute`.

### assert_raise
某些情况下, 断言抛出的异常是很有必要的. 我们可以使用 `assert_raise` 来做到这一点. 我们会在下面的 `plug` 章节看到 `assert_raise` 的示例.

### assert_receive
在 elixir 中, 应用由相互之间消息传递的 `actors/processes` 组成, 因此经常会遇到测试消息发送的情况. 由于 ExUnit 运行于自己的进程中, 其可以像其他的进程一样接收消息, 并且使用 `assert_received` 进行断言:
```elixir
defmodule SendingProcess do
  def run(pid) do
    send(pid, :ping)
  end
end

defmodule TestReceive do
  use ExUnit.Case

  test "receives ping" do
    SendingProcess.run(self())
    assert_received :ping
  end
end
```

`assert_received` 不等待消息, 其可以指定超时时间.

### capture_io & capture_log
使用 `ExUnit.CaptureIO` 可以在不修改原始代码的前提下捕获应用的输出. 只需传递生成输出的函数:
```elixir
defmodule OutputTest do
  use ExUnit.Case
  import ExUnit.CaptureIO

  test "outputs Hello World" do
    assert capture_io(fn -> IO.puts("Hello World") end) == "Hello World\n"
  end
end
```

`ExUnit.CaptureLog` 等价于捕获输出至 `Logger`.

## Test Setup
在某些情况下, 可能需要在测试之前进行配置. 可以使用 `setup` 和 `setup_all` 来达到这个目的. `setup` 会在执行每个测试之前执行, `setup_all` 会在整个测试套件执行前执行. 预期返回一个 `{:ok, state}` 这样的元组, 我们可以在测试中使用 `state`.

为了举例, 使用 `setup_all` 调整我们的代码:
```elixir
defmodule ExampleTest do
  use ExUnit.Case
  doctest Example

  setup_all do
    {:ok, recipient: :world}
  end

  test "greets", state do
    assert Example.hello() == state[:recipient]
  end
end
```

## Mocking
在 elixir 中使用 mocking 的简单答复是: 不要 mocking. 你可能不自觉的使用 mocking, 但是社区中有充分的理由不推荐你这么做.

在[这篇文章](http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/)中有大量的讨论. 重点是, 在测试中不依赖于 mocking, 而是通过定义应用之外的接口 (behaviors), 并在客户端程序中实现测试的 mock, 这样裨益良多.

在应用程序中切换实现的最佳方式是将模块作为参数传递, 并设置默认值. 如果这不起作用, 可以使用内置的配置机制. 为了实现 mock, 你无需使用特殊的 mocking 库, 只需要使用 `behaviours` 和 `callbacks`.
