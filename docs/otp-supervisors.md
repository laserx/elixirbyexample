# OTP Supervisors

Supervisors 是一种特殊的进程, 其主要目的是: 监控其他的进程. supervisors 可以自动重启异常的子进程, 使我们可以创建高容错的应用程序.

<!-- TOC -->

- [OTP Supervisors](#otp-supervisors)
    - [配置](#%E9%85%8D%E7%BD%AE)
        - [策略](#%E7%AD%96%E7%95%A5)
        - [重新启动](#%E9%87%8D%E6%96%B0%E5%90%AF%E5%8A%A8)
        - [嵌套](#%E5%B5%8C%E5%A5%97)
    - [Task Supervisor](#task-supervisor)
        - [设置](#%E8%AE%BE%E7%BD%AE)
        - [任务监控](#%E4%BB%BB%E5%8A%A1%E7%9B%91%E6%8E%A7)

<!-- /TOC -->

## 配置

Supervisors 的神奇魔法力量在 `Supervisor.start_link/2` 函数中. 除了启动我们的 supervisor 和 子进程外, 其允许我们定义 supervisor 使用何种策略来管理子进程.

子进程是通过一个列表和导入自模块 `Supervisor.Spec` 中的函数 `worker/3` 定义. 函数 `worker/3` 接收一个模块名称, 参数和一系列的选项值, 在初始化时, `worker/3` 会调用 `start_link/3` 并传入参数.

让我们使用 [OTP 并发](otp_concurrency.md)中的 `SimpleQueue` 来试一下:
```elixir
import Supervisor.Spec

children = [
  worker(SimpleQueue, [], name: SimpleQueue)
]

{:ok, pid} = Supervisor.start_link(children, strategy: :one_for_one)
```

如果我们的进程发生了崩溃或者被终止, Supervisor 会自动的将其重启, 就像什么都没发生一样.


### 策略

supervisors 现在有4种不同的重启策略可以使用:
* `:one_for_one` - 只重新启动失败的子进程.
* `:one_for_all` - 在失败时重新启动所有的子进程.
* `:rest_for_one` - 重启失败的进程以及其之后启动所有进程.
* `:simple_one_for_one` - 最适合动态附着的子进程. Supervisor 规范定义只能包含一个子进程, 但是, 该子进程可以 `spawn` 多次. 该策略旨在用于需要动态启动和停止受监控的子进程时.

### 重新启动

还有几种处理子进程崩溃的方法:
* `:permanent` - 子进程始终重新启动
* `:temporary` - 子进程永远不会重新启动
* `:transient` - 子进程只有在异常终止时重新启动

这不是必要的选项值, 默认为 `:permanent`.

### 嵌套

除了 `worker` 进程, 我们还可以监管 supervisors 来建立 supervisor tree. 唯一的区别就是将 `worker/3` 替换为 `supervisor/3`:
```elixir
import Supervisor.Spec

children = [
  supervisor(ExampleApp.ConnectionSupervisor, [[name: ExampleApp.ConnectionSupervisor]]),
  worker(SimpleQueue, [[], [name: SimpleQueue]])
]

{:ok, pid} = Supervisor.start_link(children, strategy: :one_for_one)
```

## Task Supervisor

任务有自己专属的 Supervisor, `Task.Supervisor`. 设计用于动态的创建任务, 该 Supervisor 内部使用 `simple_one_for_one` 策略.

### 设置
`Task.Supervisor` 使用上于其他的 supervisors 没有区别:
```elixir
import Supervisor.Spec

children = [
  supervisor(Task.Supervisor, [[name: ExampleApp.TaskSupervisor, restart: :transient]])
]

{:ok, pid} = Supervisor.start_link(children, strategy: :one_for_one)
```

`Supervisor` 和 `Task.Supervisor` 最主要的区别在于, `Task.Supervisor` 默认的重启策略是 `:temporary` (任务永远不会重新启动).

### 任务监控
我们可以在 supervisor 启动时, 使用函数 `start_child/2` 创建一个被监控的任务:
```elixir
{:ok, pid} = Task.Supervisor.start_child(ExampleApp.TaskSupervisor, fn -> background_work end)
```

如果我们的任务过早的崩溃, 其将为我们重启任务. 这可以在处理接入连接或者处理后台任务时, 可能特别的有用.
