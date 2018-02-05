# 集合

列表, 元组, 关键字列表和映射.

<!-- TOC -->

- [集合](#%E9%9B%86%E5%90%88)
    - [列表](#%E5%88%97%E8%A1%A8)
        - [列表拼接](#%E5%88%97%E8%A1%A8%E6%8B%BC%E6%8E%A5)
        - [列表减法](#%E5%88%97%E8%A1%A8%E5%87%8F%E6%B3%95)
        - [头/尾](#%E5%A4%B4%E5%B0%BE)
    - [元组](#%E5%85%83%E7%BB%84)
    - [关键字列表](#%E5%85%B3%E9%94%AE%E5%AD%97%E5%88%97%E8%A1%A8)
    - [映射](#%E6%98%A0%E5%B0%84)

<!-- /TOC -->

## 列表
列表是值的简单集合, 值可以是任意类型; 列表可以包含重复的值:
```elixir
iex> [3.14, :pie, "Apple"]
[3.14, :pie, "Apple"]
```

elixir 内部使用链表实现列表. 这意味着获取列表长度是一个复杂度 O(n) 的操作. 因此, 向列表头部添加值要比向尾部添加更快.
```elixir
iex> list = [3.14, :pie, "Apple"]
[3.14, :pie, "Apple"]
iex> ["π"] ++ list
["π", 3.14, :pie, "Apple"]
iex> list ++ ["Cherry"]
[3.14, :pie, "Apple", "Cherry"]
```

### 列表拼接
列表拼接使用 `++/2` 操作符:
```elixir
iex> [1, 2] ++ [3, 4, 1]
[1, 2, 3, 4, 1]
```
请注意上面 `++/2` 使用的格式: 在 elixir (以及 erlang, elixir 在其之上构建) 中, 一个函数或者操作符包含2个组件, 其名称(这里是 `++`) 和其元数. 元数是 elixir 核心的一部分. 其表示函数参数个数(在这个示例中为2). 元数和函数名使用 `/` 拼接. 我们会在后续的示例中讨论这个问题; 现在了解这些概念有助于理解这些符号.

### 列表减法
列表减法使用 `--/2` 操作符; 减去的值不存在于现有列表中是安全的:
```elixir
iex> ["foo", :bar, 42] -- [42, "bar"]
["foo", :bar]
```

请注意重复的值, 每一个右侧的值, 左侧出现的第一个值会被移除:
```elixir
iex> [1,2,2,3,2,3] -- [1,2,3,2]
[2, 3]
```
**注意**: 列表减法使用严格比较来匹配值.

### 头/尾
当我们使用列表, 通常使用列表的头部和尾部数据. 头部指的是列表的第一个元素, 而尾部是除去头部元素之外的一个列表. elixir 提供了2个有用的函数, `hd` 和 `tl`, 用于处理这样的情况:
```elixir
iex> hd [3.14, :pie, "Apple"]
3.14
iex> tl [3.14, :pie, "Apple"]
[:pie, "Apple"]
```

除了上述函数之外, 也可以使用模式匹配和列表构造函数操作符 `|` 来分隔列表为头部和尾部. 我们会在后面的课程了解这种模式:
```elixir
iex> [head | tail] = [3.14, :pie, "Apple"]
[3.14, :pie, "Apple"]
iex> head
3.14
iex> tail
[:pie, "Apple"]
```

## 元组
元组和列表类似, 但是其存储于连续的内存中, 这使得获取元组长度很快, 但是修改开销较大; 新的元组必须完全复制至内存中. 元祖使用 `{` 定义:
```elixir
iex> {3.14, :pie, "Apple"}
{3.14, :pie, "Apple"}
```
元组通常作为函数返回额外信息的一种方式; 当我们接触模式匹配后, 其用途会更加明显:
```elixir
iex> File.read("path/to/existing/file")
{:ok, "... contents ..."}
iex> File.read("path/to/unknown/file")
{:error, :enoent}
```

## 关键字列表
关键字列表和映射是 elixir 中的关联集合, 关键字列表是一个由包含两个元素的元组组成的特殊列表, 其中的元组第一个元素是原子; 其性能和列表相当:
```elixir
iex> [foo: "bar", hello: "world"]
[foo: "bar", hello: "world"]
iex> [{:foo, "bar"}, {:hello, "world"}]
[foo: "bar", hello: "world"]
```

关键字列表的三个特征显示其重要性:

* 键为原子
* 键可排序
* 键可以不唯一

基于以上原因, 关键字列表普遍使用于为函数传递选项.

## 映射
在 elixir 中, 映射是我们最常用的键-值存储. 不像关键字列表, 映射允许任意类型的值作为键, 同时不可排序. 你可以使用 `%{}` 语法定义映射:
```elixir
iex> map = %{:foo => "bar", "hello" => :world}
%{:foo => "bar", "hello" => :world}
iex> map[:foo]
"bar"
iex> map["hello"]
:world
```

自 elixir 1.2 开始, 变量可以作为映射的键:
```elixir
iex> key = "hello"
"hello"
iex> %{key => "world"}
%{"hello" => "world"}
```

如果一个重复的键添加至映射中, 其将会覆盖之前的值:
```elixir
iex> %{:foo => "bar", :foo => "hello world"}
%{foo: "hello world"}
```

从上面的输出我们可以看到, 对于只包含原子键的映射有一个特殊的语法:
```elixir
iex> %{foo: "bar", hello: "world"}
%{foo: "bar", hello: "world"}
iex> %{foo: "bar", hello: "world"} == %{:foo => "bar", :hello => "world"}
true
```

另一方面, 还有一个特殊的语法用来访问原子键的值:
```elixir
iex> map = %{foo: "bar", hello: "world"}
%{foo: "bar", hello: "world"}
iex> map.hello
"world"
```

映射还提供了更新自身属性的一种语法这一有趣的特性:
```elixir
iex> map = %{foo: "bar", hello: "world"}
%{foo: "bar", hello: "world"}
iex> %{map | foo: "baz"}
%{foo: "baz", hello: "world"}
```