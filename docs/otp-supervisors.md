# OTP Supervisors

Supervisors 是一种特殊的进程, 其主要目的是: 监控其他的进程. supervisors 可以自动重启异常的子进程, 使我们可以创建高容错的应用程序.

<!-- TOC -->

- [OTP Supervisors](#otp-supervisors)
    - [配置](#%E9%85%8D%E7%BD%AE)

<!-- /TOC -->

## 配置

Supervisors 的神奇魔法力量在 `Supervisor.start_link/2` 函数中. 除了启动我们的 supervisor 和 子进程外, 其允许我们定义 supervisor 使用何种策略来管理子进程.

子进程是通过一个列表
Children are defined using a list and the worker/3 function we imported from Supervisor.Spec. The worker/3 function takes a module, arguments, and a set of options. Under the hood worker/3 calls start_link/3 with our arguments during initialization.