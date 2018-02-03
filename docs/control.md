# 流程控制
在这一课我们将看看 elixir 中可用的流程控制方法.

## if 和 unless
在了解 `if/2` 前, 如果你有使用 Ruby 的经验, 你可能对 `unless/2` 比较熟悉. 在 elixir 中, 他们的行为和 Ruby 中一样, 不过, `if/2` 和 `unless/2` 是宏定义, 而不是语言结构; 你可以在[Kernel module](https://hexdocs.pm/elixir/Kernel.html)中找到他们的实现方法.

需要注意的是, 在 elixir 中, 只有 `nil` 和布尔值 `false` 为假值.
```elixir
iex> if String.valid?("Hello") do
...>   "Valid string!"
...> else
...>   "Invalid string."
...> end
"Valid string!"

iex> if "a string value" do
...>   "Truthy"
...> end
"Truthy"
```

`unless/2` 的用法和 `if/2` 一样, 只是其对否定生效:
```elixir
iex> unless is_integer("hello") do
...>   "Not an Int"
...> end
"Not an Int"
```