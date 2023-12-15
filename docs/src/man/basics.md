# 使用DataFrames.jl的第一步

## 设置环境

如果你想使用DataFrames.jl包，你需要首先安装它。
你可以使用以下命令来进行安装：

```julia
julia> using Pkg

julia> Pkg.add("DataFrames")
```

或者

```julia
julia> ] # 按下 ']'

(@v1.9) pkg> add DataFrames
```

如果你想确保一切都按预期工作，你可以运行与DataFrames.jl捆绑的测试，
但请注意，这将需要超过30分钟：

```julia
julia> using Pkg

julia> Pkg.test("DataFrames") # 警告！这将需要超过30分钟。
```

另外，建议使用`status`命令检查你已安装的DataFrames.jl的版本。

```julia
julia> ]

(@v1.9) pkg> status DataFrames
      Status `~\v1.6\Project.toml`
  [a93c6f00] DataFrames v1.5.0
```

在本教程的其余部分，我们将假设你已经安装了DataFrames.jl包，并已经键入`using DataFrames`来加载包：

```jldoctest dataframe
julia> using DataFrames
```

DataFrames.jl提供的最基本的类型是`DataFrame`，其中通常每行被解释为一个观察，每列被解释为一个特征。

!!! 注意 "高级安装配置"

    在构建（预编译）包时，DataFrames.jl会投入额外的时间和努力，以确保在你使用它时它更具响应性。
    然而，在某些情况下，用户可能希望避免这种额外的预编译工作，以减少构建包和后来加载它所需的时间。
    要在你当前的项目中禁用DataFrames.jl的预编译，请按照[PrecompileTools.jl文档](https://julialang.github.io/PrecompileTools.jl/stable/#Package-developers:-reducing-the-cost-of-precompilation-during-development)中给出的指示操作。

## 构造函数和基本实用函数

### 构造函数

在本节中，你将看到使用构造函数创建`DataFrame`的几种方式。你可以在[`DataFrame`](@ref)对象的文档中找到支持的构造函数的详细列表，以及更多的示例。

我们首先创建一个空的`DataFrame`：

```jldoctest dataframe
julia> DataFrame()
0×0 DataFrame
```

现在让我们初始化一个包含多个列的`DataFrame`。基本的方式如下：

```jldoctest dataframe
julia> DataFrame(A=1:3, B=5:7, fixed=1)
3×3 DataFrame
 Row │ A      B      fixed
     │ Int64  Int64  Int64
─────┼─────────────────────
   1 │     1      5      1
   2 │     2      6      1
   3 │     3      7      1
```

注意，使用这个构造函数时，标量（如列`:fixed`的`1`）会自动广播以填充创建的`DataFrame`的所有行。

有时候，你需要创建一个其列名不是有效的Julia标识符的数据帧。在这种情况下，下面的形式，其中`=`被`=>`替换，会很方便：

```jldoctest dataframe
julia> DataFrame("customer age" => [15, 20, 25],
                 "first name" => ["Rohit", "Rahul", "Akshat"])
3×2 DataFrame
 Row │ customer age  first name
     │ Int64         String
─────┼──────────────────────────
   1 │           15  Rohit
   2 │           20  Rahul
   3 │           25  Akshat
```

注意，这次我们将列名作为字符串传递。

你的源数据通常存储在字典中。
只要字典的键是字符串或`Symbol`，你也可以轻松地从中创建`DataFrame`：

```jldoctest dataframe
julia> dict = Dict("customer age" => [15, 20, 25],
                   "first name" => ["Rohit", "Rahul", "Akshat"])
Dict{String, Vector} with 2 entries:
  "first name"   => ["Rohit", "Rahul", "Akshat"]
  "customer age" => [15, 20, 25]

julia> DataFrame(dict)
3×2 DataFrame
 Row │ customer age  first name
     │ Int64         String
─────┼──────────────────────────
   1 │           15  Rohit
   2 │           20  Rahul
   3 │           25  Akshat

julia> dict = Dict(:customer_age => [15, 20, 25],
                   :first_name => ["Rohit", "Rahul", "Akshat"])
Dict{Symbol, Vector} with 2 entries:
  :customer_age => [15, 20, 25]
  :first_name   => ["Rohit", "Rahul", "Akshat"]

julia> DataFrame(dict)
3×2 DataFrame
 Row │ customer_age  first_name
     │ Int64         String
─────┼──────────────────────────
   1 │           15  Rohit
   2 │           20  Rahul
   3 │           25  Akshat
```

使用`Symbol`，例如`:customer_age`，而不是字符串，例如`"customer age"`，来表示列名是首选的，因为它更快。然而，如你在上面的例子中看到的，如果我们的列名包含一个空格，将它作为一个`Symbol`传递并不方便（你必须写成`Symbol("customer age")`，这很冗长），所以使用字符串更方便。

从向量的`NamedTuple`或`NamedTuple`的向量创建`DataFrame`也很常见。下面是这些操作的一些示例：

```jldoctest dataframe
julia> DataFrame((a=[1, 2], b=[3, 4]))
2×2 DataFrame
 Row │ a      b
     │ Int64  Int64
─────┼──────────────
   1 │     1      3
   2 │     2      4

julia> DataFrame([(a=1, b=0), (a=2, b=0)])
2×2 DataFrame
 Row │ a      b
     │ Int64  Int64
─────┼──────────────
   1 │     1      0
   2 │     2      0
```

Sometimes your source data might have a heterogeneous set of columns for each observation.
Here is an example:

```
julia> source = [(type="circle", radius=10), (type="square", side=20)]
2-element Vector{NamedTuple{names, Tuple{String, Int64}} where names}:
 (type = "circle", radius = 10)
 (type = "square", side = 20)
```

If you want to create a data frame from such data containing all columns present in at least
one of the source observations, with a `missing` entry if some column is not present then
you can use `Tables.dictcolumntable` function to help you create the desired data frame:

```
julia> DataFrame(Tables.dictcolumntable(source))
2×3 DataFrame
 Row │ type    radius   side
     │ String  Int64?   Int64?
─────┼──────────────────────────
   1 │ circle       10  missing
   2 │ square  missing       20
```

The role of `Tables.dictcolumntable` is to make sure that the `DataFrame` constructor gets information
about all columns present in the source data and properly instantiates them. If we did not use
this function the `DataFrame` constructor would assume that the first row of data contains the set
of columns present in the source, which would lead to an error in our example:

```
julia> DataFrame(source)
ERROR: type NamedTuple has no field radius
```

Sometimes your source data might have a heterogeneous set of columns for each observation.
Here is an example:

```
julia> source = [(type="circle", radius=10), (type="square", side=20)]
2-element Vector{NamedTuple{names, Tuple{String, Int64}} where names}:
 (type = "circle", radius = 10)
 (type = "square", side = 20)
```

If you want to create a data frame from such data containing all columns present in at least
one of the source observations, with a `missing` entry if some column is not present then
you can use `Tables.dictcolumntable` function to help you create the desired data frame:

```
julia> DataFrame(Tables.dictcolumntable(source))
2×3 DataFrame
 Row │ type    radius   side
     │ String  Int64?   Int64?
─────┼──────────────────────────
   1 │ circle       10  missing
   2 │ square  missing       20
```

The role of `Tables.dictcolumntable` is to make sure that the `DataFrame` constructor gets information
about all columns present in the source data and properly instantiates them. If we did not use
this function the `DataFrame` constructor would assume that the first row of data contains the set
of columns present in the source, which would lead to an error in our example:

```
julia> DataFrame(source)
ERROR: type NamedTuple has no field radius
```

让我们通过展示如何从矩阵创建`DataFrame`来结束构造函数的复习。在这种情况下，你将矩阵作为第一个参数传递。如果第二个参数只是`:auto`，那么列名`x1`，`x2`，...将会自动生成。

```jldoctest dataframe
julia> DataFrame([1 0; 2 0], :auto)
2×2 DataFrame
 Row │ x1     x2
     │ Int64  Int64
─────┼──────────────
   1 │     1      0
   2 │     2      0
```

或者，你可以将列名的向量作为第二个参数传递给`DataFrame`构造函数：

```jldoctest dataframe
julia> mat = [1 2 4 5; 15 58 69 41; 23 21 26 69]
3×4 Matrix{Int64}:
  1   2   4   5
 15  58  69  41
 23  21  26  69

julia> nms = ["a", "b", "c", "d"]
4-element Vector{String}:
 "a"
 "b"
 "c"
 "d"

julia> DataFrame(mat, nms)
3×4 DataFrame
 Row │ a      b      c      d
     │ Int64  Int64  Int64  Int64
─────┼────────────────────────────
   1 │     1      2      4      5
   2 │    15     58     69     41
   3 │    23     21     26     69
```

现在你知道如何从你的Julia会话中已经有的数据创建`DataFrame`。在下一节中，我们将展示如何从磁盘加载数据到`DataFrame`。

### 从CSV文件中读取数据

这里我们关注最常见的场景，即数据存储在硬盘上的CSV格式。

首先确保你已经安装了CSV.jl。你可以按照以下指示进行安装：

```julia
julia> using Pkg

julia> Pkg.add("CSV")
```

为了读取文件，我们将使用`CSV.read`函数。

```jldoctest dataframe
julia> using CSV

julia> path = joinpath(pkgdir(DataFrames), "docs", "src", "assets", "german.csv");

julia> german_ref = CSV.read(path, DataFrame)
1000×10 DataFrame
  Row │ id     Age    Sex      Job    Housing  Saving accounts  Checking accou ⋯
      │ Int64  Int64  String7  Int64  String7  String15         String15       ⋯
──────┼─────────────────────────────────────────────────────────────────────────
    1 │     0     67  male         2  own      NA               little         ⋯
    2 │     1     22  female       2  own      little           moderate
    3 │     2     49  male         1  own      little           NA
    4 │     3     45  male         2  free     little           little
    5 │     4     53  male         2  free     little           little         ⋯
    6 │     5     35  male         1  free     NA               NA
    7 │     6     53  male         2  own      quite rich       NA
    8 │     7     35  male         3  rent     little           moderate
  ⋮   │   ⋮      ⋮       ⋮       ⋮       ⋮            ⋮                ⋮       ⋱
  994 │   993     30  male         3  own      little           little         ⋯
  995 │   994     50  male         2  own      NA               NA
  996 │   995     31  female       1  own      little           NA
  997 │   996     40  male         3  own      little           little
  998 │   997     38  male         2  own      little           NA             ⋯
  999 │   998     23  male         2  free     little           little
 1000 │   999     27  male         2  own      moderate         moderate
                                                  4 columns and 985 rows omitted
```

如你所见，数据框比显示宽度更宽更高，因此它被剪裁，其最右边的4列和中间的985行未被打印。稍后在教程中，我们将讨论如何强制Julia显示整个数据框（如果我们需要的话）。

另外请注意，DataFrames.jl在列名下方显示了列的数据类型。在我们的例子中，它是`Int64`，或者`String7`和`String15`。

让我们在这里提到Julia中标准的`String`类型和例如`String7`或`String15`类型之间的区别。带有数字后缀的类型表示具有固定宽度的字符串（类似于许多数据库提供的`CHAR(N)`类型）。这种字符串的处理速度要比标准的`String`类型快得多（特别是如果你有很多这样的字符串），因为它们的实例不是在堆上分配的。出于这个原因，`CSV.read`默认使用这些固定宽度类型读取狭窄的字符串列。

现在让我们详细解释以下代码块：

```julia
path = joinpath(pkgdir(DataFrames), "docs", "src", "assets", "german.csv");

german_ref = CSV.read(path, DataFrame)
```

- 我们将`german.csv`文件存储在DataFrames.jl仓库中，以便使用户的生活更轻松，避免每次都需要下载它；
- `pkgdir(DataFrames)`给我们提供了到DataFrames.jl包根目录的完整路径。
- 然后从这个目录，我们需要移动到存储`german.csv`文件的目录；我们使用`joinpath`，因为这是在操作系统中独立地组合硬盘上资源的路径的推荐方式（记住，Windows和Unix的路径分隔符不同，它们使用`/`或`\`作为路径分隔符；`joinpath`函数确保我们不会因此遇到问题）；
- 然后我们读取CSV文件；`CSV.read`的第二个参数是`DataFrame`，表示我们想要将文件读入一个`DataFrame`（因为`CSV.read`允许读入许多不同的目标数据格式）。

在继续之前，复制参考数据框：

```jldoctest dataframe
julia> german = copy(german_ref); # 我们复制数据框
```

通过这种方式，即使我们通过修改`german`数据框弄乱了数据，我们也可以轻松地恢复我们的数据。

### 对数据框进行基本操作

要直接提取数据框的列（即不进行复制），你可以使用以下语法之一：
`german.Sex`，`german."Sex"`，`german[!, :Sex]` 或 `german[!, "Sex"]`。

后两种使用索引的语法更为灵活，因为它们允许我们传递一个变量来存储列的名称，而不仅仅是使用 `.` 的语法中的文字名称。

```jldoctest dataframe
julia> german.Sex
1000-element PooledArrays.PooledVector{String7, UInt32, Vector{UInt32}}:
 "male"
 "female"
 "male"
 "male"
 "male"
 "male"
 "male"
 "male"
 "male"
 "male"
 ⋮
 "male"
 "male"
 "male"
 "male"
 "female"
 "male"
 "male"
 "male"
 "male"

julia> colname = "Sex"
"Sex"

julia> german[!, colname]
1000-element PooledArrays.PooledVector{String7, UInt32, Vector{UInt32}}:
 "male"
 "female"
 "male"
 "male"
 "male"
 "male"
 "male"
 "male"
 "male"
 "male"
 ⋮
 "male"
 "male"
 "male"
 "male"
 "female"
 "male"
 "male"
 "male"
 "male"
```

由于 `german.Sex` 在提取数据框的列时不会复制，因此改变此操作返回的向量的元素将影响存储在原始 `german` 数据框中的值。要获取列的*副本*，你可以使用 `german[:, :Sex]` 或 `german[:, "Sex"]`。在这种情况下，改变此操作返回的向量不会影响存储在 `german` 数据框中的数据。

`===` 函数允许我们检查两个表达式是否生成了相同的对象，并确认上述行为：

```jldoctest dataframe
julia> german.Sex === german[!, :Sex]
true

julia> german.Sex === german[:, :Sex]
false
```

你可以使用 `names` 函数获取数据框的列名向量，列名为 `String`：

```jldoctest dataframe
julia> names(german)
10-element Vector{String}:
 "id"
 "Age"
 "Sex"
 "Job"
 "Housing"
 "Saving accounts"
 "Checking account"
 "Credit amount"
 "Duration"
 "Purpose"
```

有时你可能对满足特定条件的列名感兴趣。

例如，你可以通过将此类型作为 `names` 函数的第二个参数来获取具有给定元素类型的列名：

```jldoctest dataframe
julia> names(german, AbstractString)
5-element Vector{String}:
 "Sex"
 "Housing"
 "Saving accounts"
 "Checking account"
 "Purpose"
```

你可以在 [`names`](@ref) 函数的文档中探索更多过滤列名的选项。

如果你想将数据框的列名作为 `Symbol` 获取，可以使用 `propertynames` 函数：

```jldoctest dataframe
julia> propertynames(german)
10-element Vector{Symbol}:
 :id
 :Age
 :Sex
 :Job
 :Housing
 Symbol("Saving accounts")
 Symbol("Checking account")
 Symbol("Credit amount")
 :Duration
 :Purpose
```

如你所见，包含空格的列名作为 `Symbol` 使用并不方便，因为它们需要更多的输入并引入了一些视觉噪声。

如果你对列的元素类型感兴趣，你可以使用 `eachcol(german)` 函数获取数据框列的迭代器。然后，你可以对其进行广播 `eltype` 函数以获得所需的结果：

```jldoctest dataframe
julia> eltype.(eachcol(german))
10-element Vector{DataType}:
 Int64
 Int64
 String7
 Int64
 String7
 String15
 String15
 Int64
 Int64
 String31
```

!!! 注意

    请记住，DataFrames.jl 允许使用 `Symbol`（如 `:id`）和字符串（如 `"id"`）进行所有列索引操作以方便使用。然而，使用 `Symbol` 稍微快一些，但是当列名中存在非标准字符或者你想要操作它们时，字符串更简单。

在我们结束之前，让我们讨论 `empty` 和 `empty!` 函数，这两个函数会从 `DataFrame` 中移除所有行。理解这两个函数行为的差异将帮助你理解 DataFrames.jl 中的函数命名方案。

让我们从使用 `empty` 和 `empty!` 函数的例子开始：

```jldoctest dataframe
julia> empty(german)
0×10 DataFrame
 Row │ id     Age    Sex      Job    Housing  Saving accounts  Checking accoun ⋯
     │ Int64  Int64  String7  Int64  String7  String15         String15        ⋯
─────┴──────────────────────────────────────────────────────────────────────────
                                                               4 columns omitted

julia> german
1000×10 DataFrame
  Row │ id     Age    Sex      Job    Housing  Saving accounts  Checking accou ⋯
      │ Int64  Int64  String7  Int64  String7  String15         String15       ⋯
──────┼─────────────────────────────────────────────────────────────────────────
    1 │     0     67  male         2  own      NA               little         ⋯
    2 │     1     22  female       2  own      little           moderate
    3 │     2     49  male         1  own      little           NA
    4 │     3     45  male         2  free     little           little
    5 │     4     53  male         2  free     little           little         ⋯
    6 │     5     35  male         1  free     NA               NA
    7 │     6     53  male         2  own      quite rich       NA
    8 │     7     35  male         3  rent     little           moderate
  ⋮   │   ⋮      ⋮       ⋮       ⋮       ⋮            ⋮                ⋮       ⋱
  994 │   993     30  male         3  own      little           little         ⋯
  995 │   994     50  male         2  own      NA               NA
  996 │   995     31  female       1  own      little           NA
  997 │   996     40  male         3  own      little           little
  998 │   997     38  male         2  own      little           NA             ⋯
  999 │   998     23  male         2  free     little           little
 1000 │   999     27  male         2  own      moderate         moderate
                                                  4 columns and 985 rows omitted

julia> empty!(german)
0×10 DataFrame
 Row │ id     Age    Sex      Job    Housing  Saving accounts  Checking accoun ⋯
     │ Int64  Int64  String7  Int64  String7  String15         String15        ⋯
─────┴──────────────────────────────────────────────────────────────────────────
                                                               4 columns omitted

julia> german
0×10 DataFrame
 Row │ id     Age    Sex      Job    Housing  Saving accounts  Checking accoun ⋯
     │ Int64  Int64  String7  Int64  String7  String15         String15        ⋯
─────┴──────────────────────────────────────────────────────────────────────────
                                                               4 columns omitted
```

在上述示例中，`empty` 函数创建了一个新的 `DataFrame`，其列名和列元素类型与 `german` 相同，但没有行。另一方面，`empty!` 函数在原地从 `german` 中移除了所有行，并使其每一列都为空。

`empty` 和 `empty!` 函数行为的差异是应用了 Julia 语言中的[风格约定](https://docs.julialang.org/en/v1/manual/variables/#Stylistic-Conventions)。DataFrames.jl 包提供的所有函数都遵循这一约定。

### 获取DataFrame的基本信息

在这一部分，我们将了解如何获取我们的`german` `DataFrame`的基本信息：

`size`函数返回数据框的维度。首先我们恢复`german`数据框，因为我们刚刚在上面清空了它。

```jldoctest dataframe
julia> german = copy(german_ref);

julia> size(german)
(1000, 10)

julia> size(german, 1)
1000

julia> size(german, 2)
10
```

此外，`nrow`和`ncol`函数可以用来获取数据框的行数和列数：
```jldoctest dataframe
julia> nrow(german)
1000

julia> ncol(german)
10
```

要获取数据框中的数据的基本统计信息，使用`describe`函数（查看[`describe`](@ref)的帮助以了解如何自定义显示的统计信息）。

```jldoctest dataframe
julia> describe(german)
10×7 DataFrame
 Row │ variable          mean     min       median  max              nmissing  ⋯
     │ Symbol            Union…   Any       Union…  Any              Int64     ⋯
─────┼──────────────────────────────────────────────────────────────────────────
   1 │ id                499.5    0         499.5   999                     0  ⋯
   2 │ Age               35.546   19        33.0    75                      0
   3 │ Sex                        female            male                    0
   4 │ Job               1.904    0         2.0     3                       0
   5 │ Housing                    free              rent                    0  ⋯
   6 │ Saving accounts            NA                rich                    0
   7 │ Checking account           NA                rich                    0
   8 │ Credit amount     3271.26  250       2319.5  18424                   0
   9 │ Duration          20.903   4         18.0    72                      0  ⋯
  10 │ Purpose                    business          vacation/others         0
                                                                1 column omitted
```

要限制`describe`处理的列，使用`cols`关键字参数，例如：

```jldoctest dataframe
julia> describe(german, cols=1:3)
3×7 DataFrame
 Row │ variable  mean    min     median  max   nmissing  eltype
     │ Symbol    Union…  Any     Union…  Any   Int64     DataType
─────┼────────────────────────────────────────────────────────────
   1 │ id        499.5   0       499.5   999          0  Int64
   2 │ Age       35.546  19      33.0    75           0  Int64
   3 │ Sex               female          male         0  String7
```

默认报告的统计量是平均值、最小值、中位数、最大值、缺失值的数量以及列的元素类型。在计算摘要统计时，会跳过`missing`值。

你可以通过手动调用`show`函数来调整数据框的显示方式：`show(german, allrows=true)`会打印所有行，即使它们无法在屏幕上显示，`show(german, allcols=true)`对列也会做同样的处理，例如：

```jldoctest dataframe
julia> show(german, allcols=true)
1000×10 DataFrame
  Row │ id     Age    Sex      Job    Housing  Saving accounts  Checking account  Credit amount  Duration  Purpose
      │ Int64  Int64  String7  Int64  String7  String15         String15          Int64          Int64     String31
──────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
    1 │     0     67  male         2  own      NA               little                     1169         6  radio/TV
    2 │     1     22  female       2  own      little           moderate                   5951        48  radio/TV
    3 │     2     49  male         1  own      little           NA                         2096        12  education
    4 │     3     45  male         2  free     little           little                     7882        42  furniture/equipment
    5 │     4     53  male         2  free     little           little                     4870        24  car
    6 │     5     35  male         1  free     NA               NA                         9055        36  education
    7 │     6     53  male         2  own      quite rich       NA                         2835        24  furniture/equipment
    8 │     7     35  male         3  rent     little           moderate                   6948        36  car
  ⋮   │   ⋮      ⋮       ⋮       ⋮       ⋮            ⋮                ⋮                ⋮           ⋮               ⋮
  994 │   993     30  male         3  own      little           little                     3959        36  furniture/equipment
  995 │   994     50  male         2  own      NA               NA                         2390        12  car
  996 │   995     31  female       1  own      little           NA                         1736        12  furniture/equipment
  997 │   996     40  male         3  own      little           little                     3857        30  car
  998 │   997     38  male         2  own      little           NA                          804        12  radio/TV
  999 │   998     23  male         2  free     little           little                     1845        45  radio/TV
 1000 │   999     27  male         2  own      moderate         moderate                   4576        45  car
                                                                                                               985 rows omitted
```

直接在单个列上计算描述性统计量非常容易，只需使用在`Statistics`模块中定义的函数：

```jldoctest dataframe
julia> using Statistics

julia> mean(german.Age)
35.546
```

如果我们想对数据框的所有列应用某个函数，我们可以使用`mapcols`函数。它返回一个`DataFrame`，其中源数据框的每一列都通过第一个参数传递的函数进行转换。请注意，`mapcols`保证不会在返回的`DataFrame`中重用`german`的列。如果转换返回其参数，那么在存储之前会先复制它。

```jldoctest dataframe
julia> mapcols(id -> id .^ 2, german)
1000×10 DataFrame
  Row │ id      Age    Sex           Job    Housing   Saving accounts       Ch ⋯
      │ Int64   Int64  String        Int64  String    String                St ⋯
──────┼─────────────────────────────────────────────────────────────────────────
    1 │      0   4489  malemale          4  ownown    NANA                  li ⋯
    2 │      1    484  femalefemale      4  ownown    littlelittle          mo
    3 │      4   2401  malemale          1  ownown    littlelittle          NA
    4 │      9   2025  malemale          4  freefree  littlelittle          li
    5 │     16   2809  malemale          4  freefree  littlelittle          li ⋯
    6 │     25   1225  malemale          1  freefree  NANA                  NA
    7 │     36   2809  malemale          4  ownown    quite richquite rich  NA
    8 │     49   1225  malemale          9  rentrent  littlelittle          mo
  ⋮   │   ⋮       ⋮         ⋮          ⋮       ⋮               ⋮               ⋱
  994 │ 986049    900  malemale          9  ownown    littlelittle          li ⋯
  995 │ 988036   2500  malemale          4  ownown    NANA                  NA
  996 │ 990025    961  femalefemale      1  ownown    littlelittle          NA
  997 │ 992016   1600  malemale          9  ownown    littlelittle          li
  998 │ 994009   1444  malemale          4  ownown    littlelittle          NA ⋯
  999 │ 996004    529  malemale          4  freefree  littlelittle          li
 1000 │ 998001    729  malemale          4  ownown    moderatemoderate      mo
                                                  4 columns and 985 rows omitted
```

如果你想查看数据框的第一行和最后一行，那么你可以使用`first`和`last`函数分别进行操作：

```jldoctest dataframe
julia> first(german, 6)
6×10 DataFrame
 Row │ id     Age    Sex      Job    Housing  Saving accounts  Checking accoun ⋯
     │ Int64  Int64  String7  Int64  String7  String15         String15        ⋯
─────┼──────────────────────────────────────────────────────────────────────────
   1 │     0     67  male         2  own      NA               little          ⋯
   2 │     1     22  female       2  own      little           moderate
   3 │     2     49  male         1  own      little           NA
   4 │     3     45  male         2  free     little           little
   5 │     4     53  male         2  free     little           little          ⋯
   6 │     5     35  male         1  free     NA               NA
                                                               4 columns omitted

julia> last(german, 5)
5×10 DataFrame
 Row │ id     Age    Sex      Job    Housing  Saving accounts  Checking accoun ⋯
     │ Int64  Int64  String7  Int64  String7  String15         String15        ⋯
─────┼──────────────────────────────────────────────────────────────────────────
   1 │   995     31  female       1  own      little           NA              ⋯
   2 │   996     40  male         3  own      little           little
   3 │   997     38  male         2  own      little           NA
   4 │   998     23  male         2  free     little           little
   5 │   999     27  male         2  own      moderate         moderate        ⋯
                                                               4 columns omitted
```

如果不传递行数，使用`first`和`last`将返回数据框中的第一/最后一个`DataFrameRow`。`DataFrameRow`是`AbstractDataFrame`的单行视图。它存储了对父`DataFrame`的引用以及从父数据框中选择的行和列的信息。你可以将`DataFrameRow`看作是可变的`NamedTuple`，即允许更新源数据框，这通常很有用。

```jldoctest dataframe
julia> first(german)
DataFrameRow
 Row │ id     Age    Sex      Job    Housing  Saving accounts  Checking accoun ⋯
     │ Int64  Int64  String7  Int64  String7  String15         String15        ⋯
─────┼──────────────────────────────────────────────────────────────────────────
   1 │     0     67  male         2  own      NA               little          ⋯
                                                               4 columns omitted

julia> last(german)
DataFrameRow
  Row │ id     Age    Sex      Job    Housing  Saving accounts  Checking accou ⋯
      │ Int64  Int64  String7  Int64  String7  String15         String15       ⋯
──────┼─────────────────────────────────────────────────────────────────────────
 1000 │   999     27  male         2  own      moderate         moderate       ⋯
                                                               4 columns omitted
```

## 获取和设置数据框中的数据

### 索引语法

数据框可以类似于矩阵的方式进行索引。在手册的[Indexing](@ref)部分，你可以找到所有可用选项的详细信息。在这里，我们只强调基本的几种。

一般的索引语法是`data_frame[selected_rows, selected_columns]`。请注意，与Julia Base中的矩阵不同，这里总是需要传递行选择器和列选择器。冒号`:`表示应保留所有项目（取决于它的位置是行还是列）。以下是一些例子：

```jldoctest dataframe
julia> german[1:5, [:Sex, :Age]]
5×2 DataFrame
 Row │ Sex      Age
     │ String7  Int64
─────┼────────────────
   1 │ male        67
   2 │ female      22
   3 │ male        49
   4 │ male        45
   5 │ male        53

julia> german[1:5, :]
5×10 DataFrame
 Row │ id     Age    Sex      Job    Housing  Saving accounts  Checking accoun ⋯
     │ Int64  Int64  String7  Int64  String7  String15         String15        ⋯
─────┼──────────────────────────────────────────────────────────────────────────
   1 │     0     67  male         2  own      NA               little          ⋯
   2 │     1     22  female       2  own      little           moderate
   3 │     2     49  male         1  own      little           NA
   4 │     3     45  male         2  free     little           little
   5 │     4     53  male         2  free     little           little          ⋯
                                                               4 columns omitted

julia> german[[1, 6, 15], :]
3×10 DataFrame
 Row │ id     Age    Sex      Job    Housing  Saving accounts  Checking accoun ⋯
     │ Int64  Int64  String7  Int64  String7  String15         String15        ⋯
─────┼──────────────────────────────────────────────────────────────────────────
   1 │     0     67  male         2  own      NA               little          ⋯
   2 │     5     35  male         1  free     NA               NA
   3 │    14     28  female       2  rent     little           little
                                                               4 columns omitted

julia> german[:, [:Age, :Sex]]
1000×2 DataFrame
  Row │ Age    Sex
      │ Int64  String7
──────┼────────────────
    1 │    67  male
    2 │    22  female
    3 │    49  male
    4 │    45  male
    5 │    53  male
    6 │    35  male
    7 │    53  male
    8 │    35  male
  ⋮   │   ⋮       ⋮
  994 │    30  male
  995 │    50  male
  996 │    31  female
  997 │    40  male
  998 │    38  male
  999 │    23  male
 1000 │    27  male
       985 rows omitted
```

注意，`german[!, [:Sex]]`和`german[:, [:Sex]]`返回的是一个数据框对象，而`german[!, :Sex]`和`german[:, :Sex]`返回的是一个向量。在第一种情况下，`[:Sex]`是一个向量，表示结果对象应该是一个数据框。另一方面，`:Sex`是一个单独的`Symbol`，表示应该提取一个单独的列向量。注意，第一种情况需要传递一个向量（不仅仅是任何可迭代的对象），所以例如`german[:, (:Age, :Sex)]`是不允许的，但`german[:, [:Age, :Sex]]`是有效的。下面我们展示这两种操作以强调这个区别：

```jldoctest dataframe
julia> german[!, [:Sex]]
1000×1 DataFrame
  Row │ Sex
      │ String7
──────┼─────────
    1 │ male
    2 │ female
    3 │ male
    4 │ male
    5 │ male
    6 │ male
    7 │ male
    8 │ male
  ⋮   │    ⋮
  994 │ male
  995 │ male
  996 │ female
  997 │ male
  998 │ male
  999 │ male
 1000 │ male
985 rows omitted

julia> german[!, :Sex]
1000-element PooledArrays.PooledVector{String7, UInt32, Vector{UInt32}}:
 "male"
 "female"
 "male"
 "male"
 "male"
 "male"
 "male"
 "male"
 "male"
 "male"
 ⋮
 "male"
 "male"
 "male"
 "male"
 "female"
 "male"
 "male"
 "male"
 "male"
```

如同本教程之前解释的，使用`!`和`:`传递行索引的区别在于`!`不复制列，而`:`从数据框读取数据时会复制。因此，`german[!, [:Sex]]`数据框存储的是与源`german`数据框相同的向量，而`german[:, [:Sex]]`存储的是它的复制品。

`!`选择器通常应该避免使用，因为使用它可能导致难以捕获的错误。然而，当处理非常大的数据框时，它可以用来节省内存和提高操作的性能。

回顾我们已经学到的，要从`german`数据框中获取列`:Age`，你可以执行以下操作：

- 复制向量：`german[:, :Age]`，`german[:, "Age"]`或`german[:, 2]`；
- 获取向量而不复制：`german.Age`，`german."Age"`，`german[!, :Age]`，`german[!, "Age"]`或`german[!, 2]`。

要获取前两列作为`DataFrame`，我们可以如下索引：
- 获取复制的列：`german[:, 1:2]`，`german[:, [:id, :Age]]`，或`german[:, ["id", "Age"]]`；
- 不复制列的情况下重复使用列：`german[!, 1:2]`，`german[!, [:id, :Age]]`，或`german[!, ["id", "Age"]]`。

如果你想获取数据框的单个单元格，使用与获取矩阵单元格相同的语法：

```jldoctest dataframe
julia> german[4, 4]
2
```

### 视图

我们也可以创建一个数据帧的`view`。它通常非常有用，因为它比创建实体化的选择更节省内存。你可以使用`view`函数来创建：

```jldoctest dataframe
julia> view(german, :, 2:5)
1000×4 SubDataFrame
  Row │ Age    Sex      Job    Housing
      │ Int64  String7  Int64  String7
──────┼────────────────────────────────
    1 │    67  male         2  own
    2 │    22  female       2  own
    3 │    49  male         1  own
    4 │    45  male         2  free
    5 │    53  male         2  free
    6 │    35  male         1  free
    7 │    53  male         2  own
    8 │    35  male         3  rent
  ⋮   │   ⋮       ⋮       ⋮       ⋮
  994 │    30  male         3  own
  995 │    50  male         2  own
  996 │    31  female       1  own
  997 │    40  male         3  own
  998 │    38  male         2  own
  999 │    23  male         2  free
 1000 │    27  male         2  own
                       985 rows omitted
```

或者使用`@view`宏：

```jldoctest dataframe
julia> @view german[end:-1:1, [1, 4]]
1000×2 SubDataFrame
  Row │ id     Job
      │ Int64  Int64
──────┼──────────────
    1 │   999      2
    2 │   998      2
    3 │   997      2
    4 │   996      3
    5 │   995      1
    6 │   994      2
    7 │   993      3
    8 │   992      1
  ⋮   │   ⋮      ⋮
  994 │     6      2
  995 │     5      1
  996 │     4      2
  997 │     3      2
  998 │     2      1
  999 │     1      2
 1000 │     0      2
     985 rows omitted
```

同样，我们可以获取数据帧一列的视图：

```jldoctest dataframe
julia> @view german[1:5, 1]
5-element view(::Vector{Int64}, 1:5) with eltype Int64:
 0
 1
 2
 3
 4
```

它的单个单元格：

```jldoctest dataframe
julia> @view german[2, 2]
0-dimensional view(::Vector{Int64}, 2) with eltype Int64:
22
```

或者单独的一行：

```jldoctest dataframe
julia> @view german[3, 2:5]
DataFrameRow
 Row │ Age    Sex      Job    Housing
     │ Int64  String7  Int64  String7
─────┼────────────────────────────────
   3 │    49  male         1  own
```

如你所见，行和列的索引语法与索引完全相同。唯一的区别是我们没有创建一个新的对象，而是创建了一个现有对象的视图。

为了比较索引的性能和创建视图的性能，让我们使用BenchmarkTools.jl包运行以下基准测试（如果你想重新运行这个比较，请安装它）：

```julia
julia> using BenchmarkTools

julia> @btime $german[1:end-1, 1:end-1];
  9.900 μs (44 allocations: 57.56 KiB)

julia> @btime @view $german[1:end-1, 1:end-1];
  67.332 ns (2 allocations: 32 bytes)
```

如你所见，创建一个视图：
- 快一个数量级；
- 分配的内存更少。

视图的缺点是：
- 它指向与其父对象相同的内存（所以改变视图会改变父对象，这有时是不可取的）；
- 一些操作可能稍微慢一些（因为DataFrames.jl需要执行视图的索引到父对象的索引的映射）。

### 更改存储在数据框中的数据

为了展示如何在数据框上执行变异操作，我们首先创建一个`german`数据框的子集：

```jldoctest dataframe
julia> df1 = german[1:6, 2:4]
6×3 DataFrame
 Row │ Age    Sex      Job
     │ Int64  String7  Int64
─────┼───────────────────────
   1 │    67  male         2
   2 │    22  female       2
   3 │    49  male         1
   4 │    45  male         2
   5 │    53  male         2
   6 │    35  male         1
```

在下面的示例中，我们用一个新的向量替换了`df1`数据框中的列`:Age`：

```jldoctest dataframe
julia> val = [80, 85, 98, 95, 78, 89]
6-element Vector{Int64}:
 80
 85
 98
 95
 78
 89

julia> df1.Age = val
6-element Vector{Int64}:
 80
 85
 98
 95
 78
 89

julia> df1
6×3 DataFrame
 Row │ Age    Sex      Job
     │ Int64  String7  Int64
─────┼───────────────────────
   1 │    80  male         2
   2 │    85  female       2
   3 │    98  male         1
   4 │    95  male         2
   5 │    78  male         2
   6 │    89  male         1
```

这是一个非复制操作。只有当`val`向量的长度与`df1`的行数相同时，才能执行此操作，或者作为特殊情况，如果`df1`没有任何列。

```jldoctest dataframe
julia> df1.Age === val # 没有进行复制
true
```

如果在索引中从数据框中选择了一部分行，则会就地执行变异操作，即写入现有向量。下面将行`1:3`中的列`:Job`的值设置为`[2, 4, 6]`：

```jldoctest dataframe
julia> df1[1:3, :Job] = [2, 3, 2]
3-element Vector{Int64}:
 2
 3
 2

julia> df1
6×3 DataFrame
 Row │ Age    Sex      Job
     │ Int64  String7  Int64
─────┼───────────────────────
   1 │    80  male         2
   2 │    85  female       3
   3 │    98  male         2
   4 │    95  male         2
   5 │    78  male         2
   6 │    89  male         1
```

作为特殊规则，使用`!`作为行选择器将替换列而不进行复制（就像上面的`df1.Age = val`示例中一样）。例如，下面我们替换`:Sex`列：

```jldoctest dataframe
julia> df1[!, :Sex] = ["male", "female", "female", "transgender", "female", "male"]
6-element Vector{String}:
 "male"
 "female"
 "female"
 "transgender"
 "female"
 "male"

julia> df1
6×3 DataFrame
 Row │ Age    Sex          Job
     │ Int64  String       Int64
─────┼───────────────────────────
   1 │    80  male             2
   2 │    85  female           3
   3 │    98  female           2
   4 │    95  transgender      2
   5 │    78  female           2
   6 │    89  male             1
```

类似于设置单列的选定行，我们还可以设置数据框的给定行的选定列：

```jldoctest dataframe
julia> df1[3, 1:3] = [78, "male", 4]
3-element Vector{Any}:
 78
   "male"
  4

julia> df1
6×3 DataFrame
 Row │ Age    Sex          Job
     │ Int64  String       Int64
─────┼───────────────────────────
   1 │    80  male             2
   2 │    85  female           3
   3 │    78  male             4
   4 │    95  transgender      2
   5 │    78  female           2
   6 │    89  male             1
```

我们已经提到过`DataFrameRow`可以用于修改其父数据框。下面是一些示例：

```jldoctest dataframe
julia> dfr = df1[2, :] # DataFrameRow，包含df1的第二行和所有列
DataFrameRow
 Row │ Age    Sex     Job
     │ Int64  String  Int64
─────┼──────────────────────
   2 │    85  female      3

julia> dfr.Age = 98 # 将第二行中列`:Age`的值设置为`98`
98

julia> dfr
DataFrameRow
 Row │ Age    Sex     Job
     │ Int64  String  Int64
─────┼──────────────────────
   2 │    98  female      3

julia> dfr[2:3] = ["male", 2] # 设置列`:Sex`和`:Job`中的条目值
2-element Vector{Any}:
  "male"
 2

julia> dfr
DataFrameRow
 Row │ Age    Sex     Job
     │ Int64  String  Int64
─────┼──────────────────────
   2 │    98  male        2
```

这些操作更新了存储在`df1`数据框中的数据。

类似地，视图可以用于更新其父数据框中存储的数据。下面是一些示例：

```jldoctest dataframe
julia> sdf = view(df1, :, 2:3)
6×2 SubDataFrame
 Row │ Sex          Job
     │ String       Int64
─────┼────────────────────
   1 │ male             2
   2 │ male             2
   3 │ male             4
   4 │ transgender      2
   5 │ female           2
   6 │ male             1

julia> sdf[2, :Sex] = "female" # 将第二行中的列`:Sex`的值设置为`female`
"female"

julia> sdf
6×2 SubDataFrame
 Row │ Sex          Job
     │ String       Int64
─────┼────────────────────
   1 │ male             2
   2 │ female           2
   3 │ male             4
   4 │ transgender      2
   5 │ female           2
   6 │ male             1

julia> sdf[6, 1:2] = ["female", 3]
2-element Vector{Any}:
  "female"
 3

julia> sdf
6×2 SubDataFrame
 Row │ Sex          Job
     │ String       Int64
─────┼────────────────────
   1 │ male             2
   2 │ female           2
   3 │ male             4
   4 │ transgender      2
   5 │ female           2
   6 │ female           3
```

在所有这些情况下，`sdf`视图的父级也会被更新。

### 广播赋值

除了普通的赋值，还可以使用`.=`操作进行广播赋值。

在我们继续之前，让我们解释一下在Julia中广播是如何工作的。
执行[broadcasting](https://docs.julialang.org/en/v1/manual/mathematical-operations/#man-dot-operators)的标准语法是使用`.`。例如，与R不同，下面的操作会失败：

```jldoctest dataframe
julia> s = [25, 26, 35, 56]
4-element Vector{Int64}:
 25
 26
 35
 56

julia> s[2:3] = 0
ERROR: ArgumentError: indexed assignment with a single value to possibly many locations is not supported; perhaps use broadcasting `.=` instead?
```

相反，我们必须写成：

```jldoctest dataframe
julia> s[2:3] .= 0
2-element view(::Vector{Int64}, 2:3) with eltype Int64:
 0
 0

julia> s
4-element Vector{Int64}:
 25
  0
  0
 56
```

DataFrames.jl完全支持类似的语法。在这里，由于广播赋值，列`:Age`被一个全新的分配的向量替换：

```jldoctest dataframe
julia> df1[!, :Age] .= [85, 89, 78, 58, 96, 68] # 列`:Age`被一个全新的分配的向量替换
6-element Vector{Int64}:
 85
 89
 78
 58
 96
 68

julia> df1
6×3 DataFrame
 Row │ Age    Sex          Job
     │ Int64  String       Int64
─────┼───────────────────────────
   1 │    85  male             2
   2 │    89  female           2
   3 │    78  male             4
   4 │    58  transgender      2
   5 │    96  female           2
   6 │    68  female           3
```

在上面的例子中，如果使用`:`而不是`!`，将在现有列中进行广播赋值。
就地(in-place)和替换(replace)操作之间的主要区别在于，如果新值与旧值具有不同的类型，则需要替换列。

在下面的示例中，我们操作不存在于`df1`中的列`:Customers`和`:City`。
在这种情况下，使用`!`和`:`是等效的，并且会分配一个新的列：

```jldoctest dataframe
julia> df1[!, :Customers] .= ["Rohit", "Akshat", "Rahul", "Aayush", "Prateek", "Anam"]
6-element Vector{String}:
 "Rohit"
 "Akshat"
 "Rahul"
 "Aayush"
 "Prateek"
 "Anam"

julia> df1[:, :City] .= ["Kanpur", "Lucknow", "Bhuvneshwar", "Jaipur", "Ranchi", "Dehradoon"]
6-element Vector{String}:
 "Kanpur"
 "Lucknow"
 "Bhuvneshwar"
 "Jaipur"
 "Ranchi"
 "Dehradoon"

julia> df1
6×5 DataFrame
 Row │ Age    Sex          Job    Customers  City
     │ Int64  String       Int64  String     String
─────┼───────────────────────────────────────────────────
   1 │    85  male             2  Rohit      Kanpur
   2 │    89  female           2  Akshat     Lucknow
   3 │    78  male             4  Rahul      Bhuvneshwar
   4 │    58  transgender      2  Aayush     Jaipur
   5 │    96  female           2  Prateek    Ranchi
   6 │    68  female           3  Anam       Dehradoon
```

对于`:Age`选择器，广播赋值操作是就地进行的，所以下面的操作会引发错误：

```jldoctest dataframe
julia> df1[:, :Age] .= "Economics"
ERROR: MethodError: Cannot `convert` an object of type String to an object of type Int64
```

我们需要使用`!`，因为它会用一个全新的向量替换旧的向量：

```jldoctest dataframe
julia> df1[!, :Age] .= "Economics"
6-element Vector{String}:
 "Economics"
 "Economics"
 "Economics"
 "Economics"
 "Economics"
 "Economics"

julia> df1
6×5 DataFrame
 Row │ Age        Sex          Job    Customers  City
     │ String     String       Int64  String     String
─────┼───────────────────────────────────────────────────────
   1 │ Economics  male             2  Rohit      Kanpur
   2 │ Economics  female           2  Akshat     Lucknow
   3 │ Economics  male             4  Rahul      Bhuvneshwar
   4 │ Economics  transgender      2  Aayush     Jaipur
   5 │ Economics  female           2  Prateek    Ranchi
   6 │ Economics  female           3  Anam       Dehradoon
```

在DataFrames.jl中，有一些情况下，我们自然地希望实现类似广播的行为，但不允许使用`.`操作。
在这种情况下，为了方便用户，执行所谓的伪广播(pseudo-broadcasting)。
我们已经在`DataFrame`构造函数的示例中看到了它。下面我们展示了在`insertcols!`函数中伪广播的工作方式，
该函数在任意位置向数据框中插入列。

在下面的示例中，我们使用`insertcols!`函数创建了一个名为`:Country`的列。
由于我们传递了一个标量值`"India"`，该列的值被广播到输出数据框的所有行：

```jldoctest dataframe
julia> insertcols!(df1, 1, :Country => "India")
6×6 DataFrame
 Row │ Country  Age        Sex          Job    Customers  City
     │ String   String     String       Int64  String     String
─────┼────────────────────────────────────────────────────────────────
   1 │ India    Economics  male             2  Rohit      Kanpur
   2 │ India    Economics  female           2  Akshat     Lucknow
   3 │ India    Economics  male             4  Rahul      Bhuvneshwar
   4 │ India    Economics  transgender      2  Aayush     Jaipur
   5 │ India    Economics  female           2  Prateek    Ranchi
   6 │ India    Economics  female           3  Anam       Dehradoon
```

您可以将要插入列的位置作为第二个参数传递给`insertcols!`函数：
```
julia> insertcols!(df1, 4, :b => exp(4))
6×7 DataFrame
 Row │ Country  Age        Sex          b        Job    Customers  City        ⋯
     │ String   String     String       Float64  Int64  String     String      ⋯
─────┼──────────────────────────────────────────────────────────────────────────
   1 │ India    Economics  male         54.5982      4  Rohit      Kanpur      ⋯
   2 │ India    Economics  female       54.5982      4  Akshat     Lucknow
   3 │ India    Economics  male         54.5982      4  Rahul      Bhuvneshwar
   4 │ India    Economics  transgender  54.5982      4  Aayush     Jaipur
   5 │ India    Economics  female       54.5982      4  Prateek    Ranchi      ⋯
   6 │ India    Economics  female       54.5982      4  Anam       Dehradoon
```

### Not、Between、Cols和All列选择器

在更复杂的列选择场景中，您可以使用`Not`、`Between`、`Cols`和`All`选择器：
- `Not`选择器（来自[InvertedIndices.jl](https://github.com/mbauman/InvertedIndices.jl)包）允许我们指定要从结果数据框中排除的列。我们可以在`Not`内部放置任何有效的其他列选择器；
- `Between`选择器允许我们指定一系列列（我们可以使用任何单列选择器语法传递起始和停止列）；
- `Cols(...)`选择器选择作为其参数传递的其他选择器的并集；
- `All()`允许我们选择`DataFrame`的所有列；这与传递`:`是相同的；
- 使用正则表达式选择与其名称匹配的列。

让我们给出一些这些选择器的示例。

删除`:Age`列：

```jldoctest dataframe
julia> german[:, Not(:Age)]
1000×9 DataFrame
  Row │ id     Sex      Job    Housing  Saving accounts  Checking account  Cre ⋯
      │ Int64  String7  Int64  String7  String15         String15          Int ⋯
──────┼─────────────────────────────────────────────────────────────────────────
    1 │     0  male         2  own      NA               little                ⋯
    2 │     1  female       2  own      little           moderate
    3 │     2  male         1  own      little           NA
    4 │     3  male         2  free     little           little
    5 │     4  male         2  free     little           little                ⋯
    6 │     5  male         1  free     NA               NA
    7 │     6  male         2  own      quite rich       NA
    8 │     7  male         3  rent     little           moderate
  ⋮   │   ⋮       ⋮       ⋮       ⋮            ⋮                ⋮              ⋱
  994 │   993  male         3  own      little           little                ⋯
  995 │   994  male         2  own      NA               NA
  996 │   995  female       1  own      little           NA
  997 │   996  male         3  own      little           little
  998 │   997  male         2  own      little           NA                    ⋯
  999 │   998  male         2  free     little           little
 1000 │   999  male         2  own      moderate         moderate
                                                  3 columns and 985 rows omitted
```

选择从`:Sex`到`:Housing`的列：

```
julia> german[:, Between(:Sex, :Housing)]
1000×3 DataFrame
  Row │ Sex     Job    Housing
      │ String  Int64  String
──────┼────────────────────────
    1 │ male        2  own
    2 │ female      2  own
    3 │ male        1  own
    4 │ male        2  free
    5 │ male        2  free
    6 │ male        1  free
    7 │ male        2  own
    8 │ male        3  rent
  ⋮   │   ⋮       ⋮       ⋮
  994 │ male        3  own
  995 │ male        2  own
  996 │ female      1  own
  997 │ male        3  own
  998 │ male        2  own
  999 │ male        2  free
 1000 │ male        2  own
               985 rows omitted
```

在下面的示例中，`Cols`选择器选择作为其参数传递的`"Age"`和`Between("Sex", "Job")`选择器的并集：

```jldoctest dataframe
julia> german[:, Cols("Age", Between("Sex", "Job"))]
1000×3 DataFrame
  Row │ Age    Sex      Job
      │ Int64  String7  Int64
──────┼───────────────────────
    1 │    67  male         2
    2 │    22  female       2
    3 │    49  male         1
    4 │    45  male         2
    5 │    53  male         2
    6 │    35  male         1
    7 │    53  male         2
    8 │    35  male         3
  ⋮   │   ⋮       ⋮       ⋮
  994 │    30  male         3
  995 │    50  male         2
  996 │    31  female       1
  997 │    40  male         3
  998 │    38  male         2
  999 │    23  male         2
 1000 │    27  male         2
              985 rows omitted
```

您还可以使用正则表达式`Regex`来选择列。在下面的示例中，我们选择具有其名称中包含`"S"`的列，并使用`Not`来删除第5行：

```jldoctest dataframe
julia> german[Not(5), r"S"]
999×2 DataFrame
 Row │ Sex      Saving accounts
     │ String7  String15
─────┼──────────────────────────
   1 │ male     NA
   2 │ female   little
   3 │ male     little
   4 │ male     little
   5 │ male     NA
   6 │ male     quite rich
   7 │ male     little
   8 │ male     rich
  ⋮  │    ⋮            ⋮
 993 │ male     little
 994 │ male     NA
 995 │ female   little
 996 │ male     little
 997 │ male     little
 998 │ male     little
 999 │ male     moderate
                984 rows omitted
```

## 转换函数的基本用法

在DataFrames.jl中，我们有五个函数可以用来对数据帧的列进行转换：

- `combine`：创建一个新的数据帧，其中包含对源数据帧列应用转换得到的列，可以合并行；
- `select`：创建一个与源数据帧具有相同行数的新数据帧，其中包含对源数据帧列应用转换得到的列；
- `select!`：与`select`相同，但会直接在传入的数据帧上进行更新；
- `transform`：与`select`相同，但会保留数据帧中已存在的列（需要注意的是，这些列可能会被传递给`transform`的转换修改）；
- `transform!`：与`transform`相同，但会直接在传入的数据帧上进行更新。

指定转换的基本方式有以下几种：

- `source_column => transformation => target_column_name`：在这种情况下，将`source_column`作为参数传递给`transformation`函数，并将其存储在`target_column_name`列中。
- `source_column => transformation`：在这种情况下，我们将转换函数应用于`source_column`，目标列名会自动生成。
- `source_column => target_column_name`：将`source_column`重命名为`target_column_name`。
- `source_column`：在结果中保持源列不进行任何转换。

这些规则通常被称为转换迷你语言。

让我们来看一些应用这些规则的示例。

```jldoctest dataframe
julia> using Statistics

julia> combine(german, :Age => mean => :mean_age)
1×1 DataFrame
 Row │ mean_age
     │ Float64
─────┼──────────
   1 │   35.546

julia> select(german, :Age => mean => :mean_age)
1000×1 DataFrame
  Row │ mean_age
      │ Float64
──────┼──────────
    1 │   35.546
    2 │   35.546
    3 │   35.546
    4 │   35.546
    5 │   35.546
    6 │   35.546
    7 │   35.546
    8 │   35.546
  ⋮   │    ⋮
  994 │   35.546
  995 │   35.546
  996 │   35.546
  997 │   35.546
  998 │   35.546
  999 │   35.546
 1000 │   35.546
 985 rows omitted
```

如您所见，在这两种情况下，`mean`函数被应用于`:Age`列，并将结果存储在`:mean_age`列中。`combine`和`select`函数的区别在于，`combine`对数据进行聚合，并根据转换函数返回的行数生成相应的行数。而`select`函数始终保持数据帧中的行数与源数据帧相同。因此，在这种情况下，`mean`函数的结果被广播。

由于`combine`可以根据转换的结果生成任意数量的行，如果我们组合了一些转换，其中一些转换生成向量，而其他转换生成标量，则标量会像在`select`中一样被广播。下面是一个示例：

```jldoctest dataframe
julia> combine(german, :Age => mean => :mean_age, :Housing => unique => :housing)
3×2 DataFrame
 Row │ mean_age  housing
     │ Float64   String7
─────┼───────────────────
   1 │   35.546  own
   2 │   35.546  free
   3 │   35.546  rent
```

请注意，不允许在不同的转换中返回长度不同的向量：

```jldoctest dataframe
julia> combine(german, :Age, :Housing => unique => :Housing)
ERROR: ArgumentError: New columns must have the same length as old columns
```

让我们使用`select`讨论一些其他示例。通常，我们希望将某个函数应用于数据帧的整个列，而不是其各个元素。通常，我们可以使用广播来实现这一点，如下所示：

```jldoctest dataframe
julia> select(german, :Sex => (x -> uppercase.(x)) => :Sex)
1000×1 DataFrame
  Row │ Sex
      │ String
──────┼────────
    1 │ MALE
    2 │ FEMALE
    3 │ MALE
    4 │ MALE
    5 │ MALE
    6 │ MALE
    7 │ MALE
    8 │ MALE
  ⋮   │   ⋮
  994 │ MALE
  995 │ MALE
  996 │ FEMALE
  997 │ MALE
  998 │ MALE
  999 │ MALE
 1000 │ MALE
985 rows omitted
```

这种模式在实践中经常遇到，因此有一个`ByRow`的便捷包装器，用于创建广播变体的函数。在这些示例中，`ByRow`是一种特殊类型，用于选择操作，以表示应将包装的函数应用于选择的每个元素（行）。在这里，我们使用`uppercase`函数将`ByRow`包装器传递给目标列名`:Sex`：

```jldoctest dataframe
julia> select(german, :Sex => ByRow(uppercase) => :SEX)
1000×1 DataFrame
  Row │ SEX
      │ String
──────┼────────
    1 │ MALE
    2 │ FEMALE
    3 │ MALE
    4 │ MALE
    5 │ MALE
    6 │ MALE
    7 │ MALE
    8 │ MALE
  ⋮   │   ⋮
  994 │ MALE
  995 │ MALE
  996 │ FEMALE
  997 │ MALE
  998 │ MALE
  999 │ MALE
 1000 │ MALE
985 rows omitted
```

在这种情况下，我们使用`ByRow`包装器来转换源列`:Age`，并自动生成目标列名：

```jldoctest dataframe
julia> select(german, :Age, :Age => ByRow(sqrt))
1000×2 DataFrame
  Row │ Age    Age_sqrt
      │ Int64  Float64
──────┼─────────────────
    1 │    67   8.18535
    2 │    22   4.69042
    3 │    49   7.0
    4 │    45   6.7082
    5 │    53   7.28011
    6 │    35   5.91608
    7 │    53   7.28011
    8 │    35   5.91608
  ⋮   │   ⋮       ⋮
  994 │    30   5.47723
  995 │    50   7.07107
  996 │    31   5.56776
  997 │    40   6.32456
  998 │    38   6.16441
  999 │    23   4.79583
 1000 │    27   5.19615
        985 rows omitted
```

当我们只传递一个列（不包括`=>`部分）时，我们可以使用在索引中允许的任何列选择器。

在这里，我们从结果数据框中排除列`:Age`：

```jldoctest dataframe
julia> select(german, Not(:Age))
1000×9 DataFrame
  Row │ id     Sex      Job    Housing  Saving accounts  Checking account  Cre ⋯
      │ Int64  String7  Int64  String7  String15         String15          Int ⋯
──────┼─────────────────────────────────────────────────────────────────────────
    1 │     0  male         2  own      NA               little                ⋯
    2 │     1  female       2  own      little           moderate
    3 │     2  male         1  own      little           NA
    4 │     3  male         2  free     little           little
    5 │     4  male         2  free     little           little                ⋯
    6 │     5  male         1  free     NA               NA
    7 │     6  male         2  own      quite rich       NA
    8 │     7  male         3  rent     little           moderate
  ⋮   │   ⋮       ⋮       ⋮       ⋮            ⋮                ⋮              ⋱
  994 │   993  male         3  own      little           little                ⋯
  995 │   994  male         2  own      NA               NA
  996 │   995  female       1  own      little           NA
  997 │   996  male         3  own      little           little
  998 │   997  male         2  own      little           NA                    ⋯
  999 │   998  male         2  free     little           little
 1000 │   999  male         2  own      moderate         moderate
                                                  3 columns and 985 rows omitted
```

在下一个示例中，我们删除列`"Age"`、`"Saving accounts"`、`"Checking account"`、`"Credit amount"`和`"Purpose"`。请注意，这次我们使用字符串列选择器，因为一些列名中包含空格：

```jldoctest dataframe
julia> select(german, Not(["Age", "Saving accounts", "Checking account",
                           "Credit amount", "Purpose"]))
1000×5 DataFrame
  Row │ id     Sex      Job    Housing  Duration
      │ Int64  String7  Int64  String7  Int64
──────┼──────────────────────────────────────────
    1 │     0  male         2  own             6
    2 │     1  female       2  own            48
    3 │     2  male         1  own            12
    4 │     3  male         2  free           42
    5 │     4  male         2  free           24
    6 │     5  male         1  free           36
    7 │     6  male         2  own            24
    8 │     7  male         3  rent           36
  ⋮   │   ⋮       ⋮       ⋮       ⋮        ⋮
  994 │   993  male         3  own            36
  995 │   994  male         2  own            12
  996 │   995  female       1  own            12
  997 │   996  male         3  own            30
  998 │   997  male         2  own            12
  999 │   998  male         2  free           45
 1000 │   999  male         2  own            45
                                 985 rows omitted

```

作为另一个示例，让我们展示一下我们之前使用的`r"S"`正则表达式在`select`中也可以使用：

```jldoctest dataframe
julia> select(german, r"S")
1000×2 DataFrame
  Row │ Sex      Saving accounts
      │ String7  String15
──────┼──────────────────────────
    1 │ male     NA
    2 │ female   little
    3 │ male     little
    4 │ male     little
    5 │ male     little
    6 │ male     NA
    7 │ male     quite rich
    8 │ male     little
  ⋮   │    ⋮            ⋮
  994 │ male     little
  995 │ male     NA
  996 │ female   little
  997 │ male     little
  998 │ male     little
  999 │ male     little
 1000 │ male     moderate
                 985 rows omitted
```

使用`select`或`combine`相对于索引的好处是更容易获取多个列选择器的并集，例如：

```jldoctest dataframe
julia> select(german, r"S", "Job", 1)
1000×4 DataFrame
  Row │ Sex      Saving accounts  Job    id
      │ String7  String15         Int64  Int64
──────┼────────────────────────────────────────
    1 │ male     NA                   2      0
    2 │ female   little               2      1
    3 │ male     little               1      2
    4 │ male     little               2      3
    5 │ male     little               2      4
    6 │ male     NA                   1      5
    7 │ male     quite rich           2      6
    8 │ male     little               3      7
  ⋮   │    ⋮            ⋮           ⋮      ⋮
  994 │ male     little               3    993
  995 │ male     NA                   2    994
  996 │ female   little               1    995
  997 │ male     little               3    996
  998 │ male     little               2    997
  999 │ male     little               2    998
 1000 │ male     moderate             2    999
                               985 rows omitted
```

利用这种灵活性，这里是一种将某些列移动到数据框前面的惯用模式：

```jldoctest dataframe
julia> select(german, "Sex", :)
1000×10 DataFrame
  Row │ Sex      id     Age    Job    Housing  Saving accounts  Checking accou ⋯
      │ String7  Int64  Int64  Int64  String7  String15         String15       ⋯
──────┼─────────────────────────────────────────────────────────────────────────
    1 │ male         0     67      2  own      NA               little         ⋯
    2 │ female       1     22      2  own      little           moderate
    3 │ male         2     49      1  own      little           NA
    4 │ male         3     45      2  free     little           little
    5 │ male         4     53      2  free     little           little         ⋯
    6 │ male         5     35      1  free     NA               NA
    7 │ male         6     53      2  own      quite rich       NA
    8 │ male         7     35      3  rent     little           moderate
  ⋮   │    ⋮       ⋮      ⋮      ⋮       ⋮            ⋮                ⋮       ⋱
  994 │ male       993     30      3  own      little           little         ⋯
  995 │ male       994     50      2  own      NA               NA
  996 │ female     995     31      1  own      little           NA
  997 │ male       996     40      3  own      little           little
  998 │ male       997     38      2  own      little           NA             ⋯
  999 │ male       998     23      2  free     little           little
 1000 │ male       999     27      2  own      moderate         moderate
                                                  4 columns and 985 rows omitted
```

下面，我们只是传递源列和目标列名来重命名它们（没有指定转换部分）：

```jldoctest dataframe
julia> select(german, :Sex => :x1, :Age => :x2)
1000×2 DataFrame
  Row │ x1       x2
      │ String7  Int64
──────┼────────────────
    1 │ male        67
    2 │ female      22
    3 │ male        49
    4 │ male        45
    5 │ male        53
    6 │ male        35
    7 │ male        53
    8 │ male        35
  ⋮   │    ⋮       ⋮
  994 │ male        30
  995 │ male        50
  996 │ female      31
  997 │ male        40
  998 │ male        38
  999 │ male        23
 1000 │ male        27
       985 rows omitted
```

需要注意的是，`select`始终返回一个数据框，即使只选择了一个单独的列，与索引语法相反。比较以下两种情况：

```jldoctest dataframe
julia> select(german, :Age)
1000×1 DataFrame
  Row │ Age
      │ Int64
──────┼───────
    1 │    67
    2 │    22
    3 │    49
    4 │    45
    5 │    53
    6 │    35
    7 │    53
    8 │    35
  ⋮   │   ⋮
  994 │    30
  995 │    50
  996 │    31
  997 │    40
  998 │    38
  999 │    23
 1000 │    27
985 rows omitted

julia> german[:, :Age]
1000-element Vector{Int64}:
 67
 22
 49
 45
 53
 35
 53
 35
 61
 28
  ⋮
 34
 23
 30
 50
 31
 40
 38
 23
 27
```

默认情况下，`select`会复制传递的源数据框的列。为了避免复制，可以传递`copycols=false`关键字参数：

```jldoctest dataframe
julia> df = select(german, :Sex)
1000×1 DataFrame
  Row │ Sex
      │ String7
──────┼─────────
    1 │ male
    2 │ female
    3 │ male
    4 │ male
    5 │ male
    6 │ male
    7 │ male
    8 │ male
  ⋮   │    ⋮
  994 │ male
  995 │ male
  996 │ female
  997 │ male
  998 │ male
  999 │ male
 1000 │ male
985 rows omitted

julia> df.Sex === german.Sex # copy
false

julia> df = select(german, :Sex, copycols=false)
1000×1 DataFrame
  Row │ Sex
      │ String7
──────┼─────────
    1 │ male
    2 │ female
    3 │ male
    4 │ male
    5 │ male
    6 │ male
    7 │ male
    8 │ male
  ⋮   │    ⋮
  994 │ male
  995 │ male
  996 │ female
  997 │ male
  998 │ male
  999 │ male
 1000 │ male
985 rows omitted

julia> df.Sex === german.Sex # no-copy is performed
true
```

要在原地执行选择操作，请使用`select!`：

```jldoctest dataframe
julia> select!(german, Not(:Age));

julia> german
1000×9 DataFrame
  Row │ id     Sex      Job    Housing  Saving accounts  Checking account  Cre ⋯
      │ Int64  String7  Int64  String7  String15         String15          Int ⋯
──────┼─────────────────────────────────────────────────────────────────────────
    1 │     0  male         2  own      NA               little                ⋯
    2 │     1  female       2  own      little           moderate
    3 │     2  male         1  own      little           NA
    4 │     3  male         2  free     little           little
    5 │     4  male         2  free     little           little                ⋯
    6 │     5  male         1  free     NA               NA
    7 │     6  male         2  own      quite rich       NA
    8 │     7  male         3  rent     little           moderate
  ⋮   │   ⋮       ⋮       ⋮       ⋮            ⋮                ⋮              ⋱
  994 │   993  male         3  own      little           little                ⋯
  995 │   994  male         2  own      NA               NA
  996 │   995  female       1  own      little           NA
  997 │   996  male         3  own      little           little
  998 │   997  male         2  own      little           NA                    ⋯
  999 │   998  male         2  free     little           little
 1000 │   999  male         2  own      moderate         moderate
                                                  3 columns and 985 rows omitted
```

正如您所看到的，`german`数据框中的`:Age`列已被删除。

`transform`和`transform!`函数的工作方式与`select`和`select!`完全相同，唯一的区别是它们保留源数据框中存在的所有列。以下是一些示例：

```jldoctest dataframe
julia> german = copy(german_ref);

julia> df = german_ref[1:8, 1:5]
8×5 DataFrame
 Row │ id     Age    Sex      Job    Housing
     │ Int64  Int64  String7  Int64  String7
─────┼───────────────────────────────────────
   1 │     0     67  male         2  own
   2 │     1     22  female       2  own
   3 │     2     49  male         1  own
   4 │     3     45  male         2  free
   5 │     4     53  male         2  free
   6 │     5     35  male         1  free
   7 │     6     53  male         2  own
   8 │     7     35  male         3  rent

julia> transform(df, :Age => maximum)
8×6 DataFrame
 Row │ id     Age    Sex      Job    Housing  Age_maximum
     │ Int64  Int64  String7  Int64  String7  Int64
─────┼────────────────────────────────────────────────────
   1 │     0     67  male         2  own               67
   2 │     1     22  female       2  own               67
   3 │     2     49  male         1  own               67
   4 │     3     45  male         2  free              67
   5 │     4     53  male         2  free              67
   6 │     5     35  male         1  free              67
   7 │     6     53  male         2  own               67
   8 │     7     35  male         3  rent              67
```

在下面的示例中，我们交换存储在`:Sex`列和`:Age`列中的值：

```jldoctest dataframe
julia> transform(german, :Age => :Sex, :Sex => :Age)
1000×10 DataFrame
  Row │ id     Age      Sex    Job    Housing  Saving accounts  Checking accou ⋯
      │ Int64  String7  Int64  Int64  String7  String15         String15       ⋯
──────┼─────────────────────────────────────────────────────────────────────────
    1 │     0  male        67      2  own      NA               little         ⋯
    2 │     1  female      22      2  own      little           moderate
    3 │     2  male        49      1  own      little           NA
    4 │     3  male        45      2  free     little           little
    5 │     4  male        53      2  free     little           little         ⋯
    6 │     5  male        35      1  free     NA               NA
    7 │     6  male        53      2  own      quite rich       NA
    8 │     7  male        35      3  rent     little           moderate
  ⋮   │   ⋮       ⋮       ⋮      ⋮       ⋮            ⋮                ⋮       ⋱
  994 │   993  male        30      3  own      little           little         ⋯
  995 │   994  male        50      2  own      NA               NA
  996 │   995  female      31      1  own      little           NA
  997 │   996  male        40      3  own      little           little
  998 │   997  male        38      2  own      little           NA             ⋯
  999 │   998  male        23      2  free     little           little
 1000 │   999  male        27      2  own      moderate         moderate
                                                  4 columns and 985 rows omitted
```

如果我们将多个源列传递给转换函数，它们将作为连续的位置参数传递。因此，例如下面的`[:Age, :Job] => (+) => :res`转换会计算`+(df1.Age, df1.Job)`（将两列相加），并将结果存储在`:res`列中：

```jldoctest dataframe
julia> select(german, :Age, :Job, [:Age, :Job] => (+) => :res)
1000×3 DataFrame
  Row │ Age    Job    res
      │ Int64  Int64  Int64
──────┼─────────────────────
    1 │    67      2     69
    2 │    22      2     24
    3 │    49      1     50
    4 │    45      2     47
    5 │    53      2     55
    6 │    35      1     36
    7 │    53      2     55
    8 │    35      3     38
  ⋮   │   ⋮      ⋮      ⋮
  994 │    30      3     33
  995 │    50      2     52
  996 │    31      1     32
  997 │    40      3     43
  998 │    38      2     40
  999 │    23      2     25
 1000 │    27      2     29
            985 rows omitted
```

在这个入门教程中给出的示例并没有涵盖转换迷你语言的所有选项。在手册的后面部分，有更高级的示例，特别是展示如何使用`AsTable`操作传递或生成多列的示例（您可能在一些DataFrames.jl的演示中见过）。
