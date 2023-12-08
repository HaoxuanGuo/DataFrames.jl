# 入门指南

## 安装

DataFrames包通过Julia包系统提供，可以使用以下命令进行安装：

```julia
using Pkg
Pkg.add("DataFrames")
```

在本教程的其余部分，我们假设你已经安装了DataFrames包，并且已经输入了`using DataFrames`，将所有相关的变量引入到你当前的命名空间。

!!! 注意

    默认情况下，DataFrames.jl在Jupyter Notebook中显示数据帧时，行和列的数量分别限制为25和100。你可以通过改变`ENV["DATAFRAMES_COLUMNS"]`和`ENV["DATAFRAMES_ROWS"]`变量的值来覆盖这种行为，以保持输出的最大列数和行数。如果这些数字等于或小于0，将打印所有的列或行。

    或者，你可能想要在每个Julia会话中，通过某个Jupyter内核文件设置数据帧的最大打印行数为`100`，最大打印列数为`1000`（数字`100`和`1000`只是例子，可以进行调整）。在这种情况下，向这个Jupyter内核文件的`"env"`变量中添加一个`"DATAFRAME_COLUMNS": "1000", "DATAFRAMES_ROWS": "100"`条目。关于Jupyter内核的位置和规范的信息，请查看[这里](https://jupyter-client.readthedocs.io/en/stable/kernels.html)。

    [PrettyTables.jl](https://github.com/ronisbr/PrettyTables.jl)包在Jupyter notebook中渲染`DataFrame`。用户可以通过向`show`函数传递关键字参数`kwargs...`来自定义输出：`show(stdout, MIME("text/html"), df; kwargs...)`，其中`df`是`DataFrame`。此处可以使用PrettyTables.jl在HTML后端支持的任何参数。因此，例如，如果用户想要在Jupyter中将所有小于0的数字的颜色改为红色，他们可以在`using PrettyTables`后执行：`show(stdout, MIME("text/html"), df; highlighters = hl_lt(0, HtmlDecoration(color = "red")))`。关于可用选项的更多信息，请查看[PrettyTables.jl文档](https://ronisbr.github.io/PrettyTables.jl/stable/man/usage/)。

## `DataFrame` 类型

`DataFrame`类型的对象表示一个数据表，作为一系列向量，每个向量对应一列或变量。构造`DataFrame`的最简单方式是使用关键字参数或对传递列向量：

```jldoctest dataframe
julia> using DataFrames

julia> DataFrame(a=1:4, b=["M", "F", "F", "M"]) # 关键字参数构造器
4×2 DataFrame
 Row │ a      b
     │ Int64  String
─────┼───────────────
   1 │     1  M
   2 │     2  F
   3 │     3  F
   4 │     4  M
```

以下是其他常用的构造数据帧的方法：

```jldoctest dataframe
julia> DataFrame((a=[1, 2], b=[3, 4])) # 从命名元组的向量构造Tables.jl表
2×2 DataFrame
 Row │ a      b
     │ Int64  Int64
─────┼──────────────
   1 │     1      3
   2 │     2      4

julia> DataFrame([(a=1, b=0), (a=2, b=0)]) # 从命名元组的向量构造Tables.jl表
2×2 DataFrame
 Row │ a      b
     │ Int64  Int64
─────┼──────────────
   1 │     1      0
   2 │     2      0

julia> DataFrame("a" => 1:2, "b" => 0) # Pair构造器
2×2 DataFrame
 Row │ a      b
     │ Int64  Int64
─────┼──────────────
   1 │     1      0
   2 │     2      0

julia> DataFrame([:a => 1:2, :b => 0]) # Pairs向量构造器
2×2 DataFrame
 Row │ a      b
     │ Int64  Int64
─────┼──────────────
   1 │     1      0
   2 │     2      0

julia> DataFrame(Dict(:a => 1:2, :b => 0)) # 字典构造器
2×2 DataFrame
 Row │ a      b
     │ Int64  Int64
─────┼──────────────
   1 │     1      0
   2 │     2      0

julia> DataFrame([[1, 2], [0, 0]], [:a, :b]) # 向量的向量构造器
2×2 DataFrame
 Row │ a      b
     │ Int64  Int64
─────┼──────────────
   1 │     1      0
   2 │     2      0

julia> DataFrame([1 0; 2 0], :auto) # 矩阵构造器
2×2 DataFrame
 Row │ x1     x2
     │ Int64  Int64
─────┼──────────────
   1 │     1      0
   2 │     2      0
```

列可以直接（即，不复制）用`df.col`，`df."col"`，`df[!, :col]`或`df[!, "col"]`提取（这个规则适用于从数据帧获取数据，而不是向数据帧写入数据）。后两种语法更灵活，因为它们允许传递一个保存列名的变量，而不仅仅是一个字面名。注意，列名可以是符号（写成`:col`，`:var"col"`或`Symbol("col")`）或字符串（写成`"col"`）。在`df."col"`和`:var"col"`的形式中，使用`进行字符串插值不起作用。列也可以通过指定它们的位置的整数索引来提取。

由于`df[!, :col]`不会复制，改变这种语法返回的列向量的元素将影响存储在原始`df`中的值。要获取列的副本，使用`df[:, :col]`：改变这种语法返回的向量不会改变`df`。

```jldoctest dataframe
julia> df = DataFrame(A=1:4, B=["M", "F", "F", "M"])
4×2 DataFrame
 Row │ A      B
     │ Int64  String
─────┼───────────────
   1 │     1  M
   2 │     2  F
   3 │     3  F
   4 │     4  M

julia> df.A
4-element Vector{Int64}:
 1
 2
 3
 4

julia> df."A"
4-element Vector{Int64}:
 1
 2
 3
 4

julia> df.A === df[!, :A]
true

julia> df.A === df[:, :A]
false

julia> df.A == df[:, :A]
true

julia> df.A === df[!, "A"]
true

julia> df.A === df[:, "A"]
false

julia> df.A == df[:, "A"]
true

julia> df.A === df[!, 1]
true

julia> df.A === df[:, 1]
false

julia> df.A == df[:, 1]
true

julia> firstcolumn = :A
:A

julia> df[!, firstcolumn] === df.A
true

julia> df[:, firstcolumn] === df.A
false

julia> df[:, firstcolumn] == df.A
true
```

列名可以使用`names`函数获取为字符串：

```jldoctest dataframe
julia> names(df)
2-element Vector{String}:
 "A"
 "B"
```

你也可以通过传递一个列选择器条件作为第二个参数来过滤列名。在[`names`](@ref)文档字符串中，你可以找到关于所有可用条件的详细列表。这里我们给出一些选定的例子：

```jldoctest dataframe
julia> names(df, r"A") # 正则表达式选择器
1-element Vector{String}:
 "A"

julia> names(df, Int) # 使用列元素类型的选择器
1-element Vector{String}:
 "A"

julia> names(df, Not(:B)) # 保留除:B以外的所有列的选择器
1-element Vector{String}:
 "A"
```

要以`Symbol`形式获取列名，使用`propertynames`函数：

```jldoctest dataframe
julia> propertynames(df)
2-element Vector{Symbol}:
 :A
 :B
```

!!! 注意

    DataFrames.jl允许使用`Symbol`（如`:A`）和字符串（如`"A"`）进行所有列索引操作，以便于使用。然而，使用`Symbol`稍微快一些，通常应该被优先选择，除非通过字符串操作生成它们。

### 按列构造

也可以从一个空的`DataFrame`开始，一列一列地添加：

```jldoctest dataframe
julia> df = DataFrame()
0×0 DataFrame

julia> df.A = 1:8
1:8

julia> df[:, :B] = ["M", "F", "F", "M", "F", "M", "M", "F"]
8-element Vector{String}:
 "M"
 "F"
 "F"
 "M"
 "F"
 "M"
 "M"
 "F"

julia> df[!, :C] .= 0
8-element Vector{Int64}:
 0
 0
 0
 0
 0
 0
 0
 0

julia> df
8×3 DataFrame
 Row │ A      B       C
     │ Int64  String  Int64
─────┼──────────────────────
   1 │     1  M           0
   2 │     2  F           0
   3 │     3  F           0
   4 │     4  M           0
   5 │     5  F           0
   6 │     6  M           0
   7 │     7  M           0
   8 │     8  F           0
```

我们以这种方式构建的`DataFrame`有8行和3列。这可以使用`size`函数进行检查：

```jldoctest dataframe
julia> size(df, 1)
8

julia> size(df, 2)
3

julia> size(df)
(8, 3)
```

在上述例子中，注意到表达式`df[!, :C] .= 0`通过广播一个标量来创建了数据框中的新列。

在设置数据框的列时，`df[!, :C]`和`df.C`的语法是等价的，它们会替换（或创建）`df`中的`:C`列。这与使用`df[:, :C]`设置数据框中的列不同，后者如果列已经存在的话，会就地更新列的内容。

这里有一个例子展示这种差异。让我们尝试将`:B`列改为二元变量。

```jldoctest dataframe
julia> df[:, :B] = df.B .== "F"
ERROR: MethodError: Cannot `convert` an object of type Bool to an object of type String

julia> df[:, :B] .= df.B .== "F"
ERROR: MethodError: Cannot `convert` an object of type Bool to an object of type String
```

上述操作没有成功，因为当你使用`:`作为行选择器时，`:B`列是就地更新的，它只支持存储字符串。

另一方面，下面的操作可以：

```jldoctest dataframe
julia> df.B = df.B .== "F"
8-element BitVector:
 0
 1
 1
 0
 1
 0
 0
 1

julia> df
8×3 DataFrame
 Row │ A      B      C
     │ Int64  Bool   Int64
─────┼─────────────────────
   1 │     1  false      0
   2 │     2   true      0
   3 │     3   true      0
   4 │     4  false      0
   5 │     5   true      0
   6 │     6  false      0
   7 │     7  false      0
   8 │     8   true      0
```

如你所见，因为我们在赋值的右侧使用了`df.B`，所以`:B`列被替换了。如果我们使用`df[!, :B]`或者我们使用广播赋值`.=`，将会达到同样的效果。

在手册的[Indexing](@ref)部分，你可以找到所有关于所有可用索引选项的详细信息。

### 逐行构建

也可以逐行填充`DataFrame`。让我们构建一个空的数据框，有两列（注意第一列只能包含整数，第二列只能包含字符串）：

```jldoctest dataframe
julia> df = DataFrame(A=Int[], B=String[])
0×2 DataFrame
 Row │ A      B
     │ Int64  String
─────┴───────────────
```

然后可以添加元组或向量作为行，其中元素的顺序与列的顺序匹配。要在数据框的末尾添加新行，使用[`push!`](@ref)：

```jldoctest dataframe
julia> push!(df, (1, "M"))
1×2 DataFrame
 Row │ A      B
     │ Int64  String
─────┼───────────────
   1 │     1  M

julia> push!(df, [2, "N"])
2×2 DataFrame
 Row │ A      B
     │ Int64  String
─────┼───────────────
   1 │     1  M
   2 │     2  N
```

行也可以作为`Dict`添加，其中字典的键匹配列的名称：

```jldoctest dataframe
julia> push!(df, Dict(:B => "F", :A => 3))
3×2 DataFrame
 Row │ A      B
     │ Int64  String
─────┼───────────────
   1 │     1  M
   2 │     2  N
   3 │     3  F
```

请注意，逐行构建`DataFrame`的性能明显低于一次性构建，或者按列构建。对于许多用例，这可能无关紧要，但对于非常大的`DataFrame`，这可能是一个考虑因素。

如果你想在数据框的开始处添加行，使用[`pushfirst!`](@ref)，要在任意位置插入行，使用[`insert!`](@ref)。

你也可以使用[`append!`](@ref)和[`prepend!`](@ref)函数将整个表添加到数据框中。

### 从另一种表类型构建

DataFrames支持[Tables.jl](https://github.com/JuliaData/Tables.jl)接口，用于与表格数据交互。这意味着`DataFrame`可以作为任何期望Tables.jl接口输入的包的"源"（文件格式包，数据操作包等）。`DataFrame`也可以是任何Tables.jl接口输入的接收器。一些示例用途是：

```julia
df = DataFrame(a=[1, 2, 3], b=[:a, :b, :c])

# 把 DataFrame 写入 CSV 文件
CSV.write("dataframe.csv", df)

# 把 DataFrame 存储在 SQLite 数据库表中
SQLite.load!(df, db, "dataframe_table")

# 通过 Query.jl 包转换 DataFrame
df = df |> @map({a=_.a + 1, _.b}) |> DataFrame
```

支持[Tables.jl](https://github.com/JuliaData/Tables.jl)接口的特定常见集合的一个例子是`NamedTuple`的向量：

```jldoctest dataframe
julia> v = [(a=1, b=2), (a=3, b=4)]
2-element Vector{NamedTuple{(:a, :b), Tuple{Int64, Int64}}}:
 (a = 1, b = 2)
 (a = 3, b = 4)

julia> df = DataFrame(v)
2×2 DataFrame
 Row │ a      b
     │ Int64  Int64
─────┼──────────────
   1 │     1      2
   2 │     3      4
```

你也可以很容易地把数据框转换回`NamedTuple`的向量：

```jldoctest dataframe
julia> using Tables

julia> Tables.rowtable(df)
2-element Vector{NamedTuple{(:a, :b), Tuple{Int64, Int64}}}:
 (a = 1, b = 2)
 (a = 3, b = 4)
```
