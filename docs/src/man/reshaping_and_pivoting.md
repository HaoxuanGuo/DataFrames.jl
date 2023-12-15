# 重塑和旋转数据

使用 `stack` 函数将数据从宽格式转换为长格式：

```jldoctest reshape
julia> using DataFrames, CSV

julia> path = joinpath(pkgdir(DataFrames), "docs", "src", "assets", "iris.csv");

julia> iris = CSV.read(path, DataFrame)
150×5 DataFrame
 Row │ SepalLength  SepalWidth  PetalLength  PetalWidth  Species
     │ Float64      Float64     Float64      Float64     String15
─────┼──────────────────────────────────────────────────────────────────
   1 │         5.1         3.5          1.4         0.2  Iris-setosa
   2 │         4.9         3.0          1.4         0.2  Iris-setosa
   3 │         4.7         3.2          1.3         0.2  Iris-setosa
   4 │         4.6         3.1          1.5         0.2  Iris-setosa
   5 │         5.0         3.6          1.4         0.2  Iris-setosa
   6 │         5.4         3.9          1.7         0.4  Iris-setosa
   7 │         4.6         3.4          1.4         0.3  Iris-setosa
   8 │         5.0         3.4          1.5         0.2  Iris-setosa
  ⋮  │      ⋮           ⋮            ⋮           ⋮             ⋮
 144 │         6.8         3.2          5.9         2.3  Iris-virginica
 145 │         6.7         3.3          5.7         2.5  Iris-virginica
 146 │         6.7         3.0          5.2         2.3  Iris-virginica
 147 │         6.3         2.5          5.0         1.9  Iris-virginica
 148 │         6.5         3.0          5.2         2.0  Iris-virginica
 149 │         6.2         3.4          5.4         2.3  Iris-virginica
 150 │         5.9         3.0          5.1         1.8  Iris-virginica
                                                        135 rows omitted

julia> stack(iris, 1:4)
600×3 DataFrame
 Row │ Species         variable     value
     │ String15        String       Float64
─────┼──────────────────────────────────────
   1 │ Iris-setosa     SepalLength      5.1
   2 │ Iris-setosa     SepalLength      4.9
   3 │ Iris-setosa     SepalLength      4.7
   4 │ Iris-setosa     SepalLength      4.6
   5 │ Iris-setosa     SepalLength      5.0
   6 │ Iris-setosa     SepalLength      5.4
   7 │ Iris-setosa     SepalLength      4.6
   8 │ Iris-setosa     SepalLength      5.0
  ⋮  │       ⋮              ⋮          ⋮
 594 │ Iris-virginica  PetalWidth       2.3
 595 │ Iris-virginica  PetalWidth       2.5
 596 │ Iris-virginica  PetalWidth       2.3
 597 │ Iris-virginica  PetalWidth       1.9
 598 │ Iris-virginica  PetalWidth       2.0
 599 │ Iris-virginica  PetalWidth       2.3
 600 │ Iris-virginica  PetalWidth       1.8
                            585 rows omitted
```

`stack` 的第二个可选参数表示要堆叠的列。这些通常被称为测量变量。也可以给出列名：

```jldoctest reshape
julia> stack(iris, [:SepalLength, :SepalWidth, :PetalLength, :PetalWidth])
600×3 DataFrame
 Row │ Species         variable     value
     │ String15        String       Float64
─────┼──────────────────────────────────────
   1 │ Iris-setosa     SepalLength      5.1
   2 │ Iris-setosa     SepalLength      4.9
   3 │ Iris-setosa     SepalLength      4.7
   4 │ Iris-setosa     SepalLength      4.6
   5 │ Iris-setosa     SepalLength      5.0
   6 │ Iris-setosa     SepalLength      5.4
   7 │ Iris-setosa     SepalLength      4.6
   8 │ Iris-setosa     SepalLength      5.0
  ⋮  │       ⋮              ⋮          ⋮
 594 │ Iris-virginica  PetalWidth       2.3
 595 │ Iris-virginica  PetalWidth       2.5
 596 │ Iris-virginica  PetalWidth       2.3
 597 │ Iris-virginica  PetalWidth       1.9
 598 │ Iris-virginica  PetalWidth       2.0
 599 │ Iris-virginica  PetalWidth       2.3
 600 │ Iris-virginica  PetalWidth       1.8
                            585 rows omitted
```

注意，所有列可以是不同的类型。类型提升遵循 `vcat` 的规则。

堆叠后的 `DataFrame` 包括所有未指定为堆叠的列。这些列对于每个堆叠的列都会重复。这些通常被称为标识符（id）列。除了 id 列，还有两个额外的列，标签为 `:variable` 和 `:values`，包含列标识符和堆叠的列。

`stack` 的第三个可选参数表示重复的 id 列。这使得指定您希望包含在长格式中的变量更加容易：

```jldoctest reshape
julia> stack(iris, [:SepalLength, :SepalWidth], :Species)
300×3 DataFrame
 Row │ Species         variable     value
     │ String15        String       Float64
─────┼──────────────────────────────────────
   1 │ Iris-setosa     SepalLength      5.1
   2 │ Iris-setosa     SepalLength      4.9
   3 │ Iris-setosa     SepalLength      4.7
   4 │ Iris-setosa     SepalLength      4.6
   5 │ Iris-setosa     SepalLength      5.0
   6 │ Iris-setosa     SepalLength      5.4
   7 │ Iris-setosa     SepalLength      4.6
   8 │ Iris-setosa     SepalLength      5.0
  ⋮  │       ⋮              ⋮          ⋮
 294 │ Iris-virginica  SepalWidth       3.2
 295 │ Iris-virginica  SepalWidth       3.3
 296 │ Iris-virginica  SepalWidth       3.0
 297 │ Iris-virginica  SepalWidth       2.5
 298 │ Iris-virginica  SepalWidth       3.0
 299 │ Iris-virginica  SepalWidth       3.4
 300 │ Iris-virginica  SepalWidth       3.0
                            285 rows omitted
```

如果你更喜欢指定 id 列，那么可以像这样使用 `Not` 和 `stack`：

```jldoctest reshape
julia> stack(iris, Not(:Species))
600×3 DataFrame
 Row │ Species         variable     value
     │ String15        String       Float64
─────┼──────────────────────────────────────
   1 │ Iris-setosa     SepalLength      5.1
   2 │ Iris-setosa     SepalLength      4.9
   3 │ Iris-setosa     SepalLength      4.7
   4 │ Iris-setosa     SepalLength      4.6
   5 │ Iris-setosa     SepalLength      5.0
   6 │ Iris-setosa     SepalLength      5.4
   7 │ Iris-setosa     SepalLength      4.6
   8 │ Iris-setosa     SepalLength      5.0
  ⋮  │       ⋮              ⋮          ⋮
 594 │ Iris-virginica  PetalWidth       2.3
 595 │ Iris-virginica  PetalWidth       2.5
 596 │ Iris-virginica  PetalWidth       2.3
 597 │ Iris-virginica  PetalWidth       1.9
 598 │ Iris-virginica  PetalWidth       2.0
 599 │ Iris-virginica  PetalWidth       2.3
 600 │ Iris-virginica  PetalWidth       1.8
                            585 rows omitted
```

`unstack` 将数据从长格式转换为宽格式。默认情况下需要指定哪些列是 id 变量、列变量名和列值：

这是对上述Julia代码和注释的中文翻译：

```jldoctest reshape
julia> iris.id = 1:size(iris, 1)
1:150

julia> longdf = stack(iris, Not([:Species, :id]))
600×4 DataFrame
 Row │ Species         id     variable     value
     │ String15        Int64  String       Float64
─────┼─────────────────────────────────────────────
   1 │ Iris-setosa         1  SepalLength      5.1
   2 │ Iris-setosa         2  SepalLength      4.9
   3 │ Iris-setosa         3  SepalLength      4.7
   4 │ Iris-setosa         4  SepalLength      4.6
   5 │ Iris-setosa         5  SepalLength      5.0
   6 │ Iris-setosa         6  SepalLength      5.4
   7 │ Iris-setosa         7  SepalLength      4.6
   8 │ Iris-setosa         8  SepalLength      5.0
  ⋮  │       ⋮           ⋮         ⋮          ⋮
 594 │ Iris-virginica    144  PetalWidth       2.3
 595 │ Iris-virginica    145  PetalWidth       2.5
 596 │ Iris-virginica    146  PetalWidth       2.3
 597 │ Iris-virginica    147  PetalWidth       1.9
 598 │ Iris-virginica    148  PetalWidth       2.0
 599 │ Iris-virginica    149  PetalWidth       2.3
 600 │ Iris-virginica    150  PetalWidth       1.8
                                   585 rows omitted

julia> unstack(longdf, :id, :variable, :value)
150×5 DataFrame
 Row │ id     SepalLength  SepalWidth  PetalLength  PetalWidth
     │ Int64  Float64?     Float64?    Float64?     Float64?
─────┼─────────────────────────────────────────────────────────
   1 │     1          5.1         3.5          1.4         0.2
   2 │     2          4.9         3.0          1.4         0.2
   3 │     3          4.7         3.2          1.3         0.2
   4 │     4          4.6         3.1          1.5         0.2
   5 │     5          5.0         3.6          1.4         0.2
   6 │     6          5.4         3.9          1.7         0.4
   7 │     7          4.6         3.4          1.4         0.3
   8 │     8          5.0         3.4          1.5         0.2
  ⋮  │   ⋮         ⋮           ⋮            ⋮           ⋮
 144 │   144          6.8         3.2          5.9         2.3
 145 │   145          6.7         3.3          5.7         2.5
 146 │   146          6.7         3.0          5.2         2.3
 147 │   147          6.3         2.5          5.0         1.9
 148 │   148          6.5         3.0          5.2         2.0
 149 │   149          6.2         3.4          5.4         2.3
 150 │   150          5.9         3.0          5.1         1.8
                                               135 rows omitted
```

如果剩余的列是唯一的，你可以跳过 id 变量并使用：

```jldoctest reshape
julia> unstack(longdf, :variable, :value)
150×6 DataFrame
 Row │ Species         id     SepalLength  SepalWidth  PetalLength  PetalWidth ⋯
     │ String15        Int64  Float64?     Float64?    Float64?     Float64?   ⋯
─────┼──────────────────────────────────────────────────────────────────────────
   1 │ Iris-setosa         1          5.1         3.5          1.4         0.2 ⋯
   2 │ Iris-setosa         2          4.9         3.0          1.4         0.2
   3 │ Iris-setosa         3          4.7         3.2          1.3         0.2
   4 │ Iris-setosa         4          4.6         3.1          1.5         0.2
   5 │ Iris-setosa         5          5.0         3.6          1.4         0.2 ⋯
   6 │ Iris-setosa         6          5.4         3.9          1.7         0.4
   7 │ Iris-setosa         7          4.6         3.4          1.4         0.3
   8 │ Iris-setosa         8          5.0         3.4          1.5         0.2
  ⋮  │       ⋮           ⋮         ⋮           ⋮            ⋮           ⋮      ⋱
 144 │ Iris-virginica    144          6.8         3.2          5.9         2.3 ⋯
 145 │ Iris-virginica    145          6.7         3.3          5.7         2.5
 146 │ Iris-virginica    146          6.7         3.0          5.2         2.3
 147 │ Iris-virginica    147          6.3         2.5          5.0         1.9
 148 │ Iris-virginica    148          6.5         3.0          5.2         2.0 ⋯
 149 │ Iris-virginica    149          6.2         3.4          5.4         2.3
 150 │ Iris-virginica    150          5.9         3.0          5.1         1.8
                                                               135 rows omitted
```

你甚至可以跳过传递 `:variable` 和 `:value` 值作为位置参数，因为它们将默认被使用，可以这样写：
```jldoctest reshape
julia> unstack(longdf)
150×6 DataFrame
 Row │ Species         id     SepalLength  SepalWidth  PetalLength  PetalWidth ⋯
     │ String15        Int64  Float64?     Float64?    Float64?     Float64?   ⋯
─────┼──────────────────────────────────────────────────────────────────────────
   1 │ Iris-setosa         1          5.1         3.5          1.4         0.2 ⋯
   2 │ Iris-setosa         2          4.9         3.0          1.4         0.2
   3 │ Iris-setosa         3          4.7         3.2          1.3         0.2
   4 │ Iris-setosa         4          4.6         3.1          1.5         0.2
   5 │ Iris-setosa         5          5.0         3.6          1.4         0.2 ⋯
   6 │ Iris-setosa         6          5.4         3.9          1.7         0.4
   7 │ Iris-setosa         7          4.6         3.4          1.4         0.3
   8 │ Iris-setosa         8          5.0         3.4          1.5         0.2
  ⋮  │       ⋮           ⋮         ⋮           ⋮            ⋮           ⋮      ⋱
 144 │ Iris-virginica    144          6.8         3.2          5.9         2.3 ⋯
 145 │ Iris-virginica    145          6.7         3.3          5.7         2.5
 146 │ Iris-virginica    146          6.7         3.0          5.2         2.3
 147 │ Iris-virginica    147          6.3         2.5          5.0         1.9
 148 │ Iris-virginica    148          6.5         3.0          5.2         2.0 ⋯
 149 │ Iris-virginica    149          6.2         3.4          5.4         2.3
 150 │ Iris-virginica    150          5.9         3.0          5.1         1.8
                                                               135 rows omitted
```

将 `view=true` 传递给 `stack` 会返回一个数据框，其列是原始宽数据框的视图。以下是一个例子：

```jldoctest reshape
julia> stack(iris, view=true)
600×4 DataFrame
 Row │ Species         id     variable     value
     │ String15        Int64  String       Float64
─────┼─────────────────────────────────────────────
   1 │ Iris-setosa         1  SepalLength      5.1
   2 │ Iris-setosa         2  SepalLength      4.9
   3 │ Iris-setosa         3  SepalLength      4.7
   4 │ Iris-setosa         4  SepalLength      4.6
   5 │ Iris-setosa         5  SepalLength      5.0
   6 │ Iris-setosa         6  SepalLength      5.4
   7 │ Iris-setosa         7  SepalLength      4.6
   8 │ Iris-setosa         8  SepalLength      5.0
  ⋮  │       ⋮           ⋮         ⋮          ⋮
 594 │ Iris-virginica    144  PetalWidth       2.3
 595 │ Iris-virginica    145  PetalWidth       2.5
 596 │ Iris-virginica    146  PetalWidth       2.3
 597 │ Iris-virginica    147  PetalWidth       1.9
 598 │ Iris-virginica    148  PetalWidth       2.0
 599 │ Iris-virginica    149  PetalWidth       2.3
 600 │ Iris-virginica    150  PetalWidth       1.8
                                   585 rows omitted
```

这样可以节省内存。为了创建这个视图，定义了几个`AbstractVector`：

`:variable`列 -- `EachRepeatedVector`
这将变量重复N次，其中N是原始AbstractDataFrame的行数。

`:value`列 -- `StackedVector`
这提供了原始列堆叠在一起的视图。

Id列 -- `RepeatedVector`
这将原始列重复N次，其中N是堆叠的列数。

要进行聚合，可以使用split-apply-combine函数与
`unstack`结合，或者在`unstack`中使用`combine`关键字参数。以下是一个例子：

```jldoctest reshape
julia> using Statistics

julia> d = stack(iris, Not(:Species))
750×3 DataFrame
 Row │ Species         variable     value
     │ String15        String       Float64
─────┼──────────────────────────────────────
   1 │ Iris-setosa     SepalLength      5.1
   2 │ Iris-setosa     SepalLength      4.9
   3 │ Iris-setosa     SepalLength      4.7
   4 │ Iris-setosa     SepalLength      4.6
   5 │ Iris-setosa     SepalLength      5.0
   6 │ Iris-setosa     SepalLength      5.4
   7 │ Iris-setosa     SepalLength      4.6
   8 │ Iris-setosa     SepalLength      5.0
  ⋮  │       ⋮              ⋮          ⋮
 744 │ Iris-virginica  id             144.0
 745 │ Iris-virginica  id             145.0
 746 │ Iris-virginica  id             146.0
 747 │ Iris-virginica  id             147.0
 748 │ Iris-virginica  id             148.0
 749 │ Iris-virginica  id             149.0
 750 │ Iris-virginica  id             150.0
                            735 rows omitted

julia> agg = combine(groupby(d, [:variable, :Species]), :value => mean => :vmean)
15×3 DataFrame
 Row │ variable     Species          vmean
     │ String       String15         Float64
─────┼───────────────────────────────────────
   1 │ SepalLength  Iris-setosa        5.006
   2 │ SepalLength  Iris-versicolor    5.936
   3 │ SepalLength  Iris-virginica     6.588
   4 │ SepalWidth   Iris-setosa        3.418
   5 │ SepalWidth   Iris-versicolor    2.77
   6 │ SepalWidth   Iris-virginica     2.974
   7 │ PetalLength  Iris-setosa        1.464
   8 │ PetalLength  Iris-versicolor    4.26
   9 │ PetalLength  Iris-virginica     5.552
  10 │ PetalWidth   Iris-setosa        0.244
  11 │ PetalWidth   Iris-versicolor    1.326
  12 │ PetalWidth   Iris-virginica     2.026
  13 │ id           Iris-setosa       25.5
  14 │ id           Iris-versicolor   75.5
  15 │ id           Iris-virginica   125.5

julia> unstack(agg, :variable, :Species, :vmean)
5×4 DataFrame
 Row │ variable     Iris-setosa  Iris-versicolor  Iris-virginica
     │ String       Float64?     Float64?         Float64?
─────┼───────────────────────────────────────────────────────────
   1 │ SepalLength        5.006            5.936           6.588
   2 │ SepalWidth         3.418            2.77            2.974
   3 │ PetalLength        1.464            4.26            5.552
   4 │ PetalWidth         0.244            1.326           2.026
   5 │ id                25.5             75.5           125.5

julia> unstack(d, :variable, :Species, :value, combine=mean)
5×4 DataFrame
 Row │ variable     Iris-setosa  Iris-versicolor  Iris-virginica
     │ String       Float64?     Float64?         Float64?
─────┼───────────────────────────────────────────────────────────
   1 │ SepalLength        5.006            5.936           6.588
   2 │ SepalWidth         3.418            2.77            2.974
   3 │ PetalLength        1.464            4.26            5.552
   4 │ PetalWidth         0.244            1.326           2.026
   5 │ id                25.5             75.5           125.5
```

要将`AbstractDataFrame`转置，可以使用[`permutedims`](@ref)函数。

```jldoctest reshape
julia> df1 = DataFrame(a=["x", "y"], b=[1.0, 2.0], c=[3, 4], d=[true, false])
2×4 DataFrame
 Row │ a       b        c      d
     │ String  Float64  Int64  Bool
─────┼───────────────────────────────
   1 │ x           1.0      3   true
   2 │ y           2.0      4  false

julia> permutedims(df1, 1)
3×3 DataFrame
 Row │ a       x        y
     │ String  Float64  Float64
─────┼──────────────────────────
   1 │ b           1.0      2.0
   2 │ c           3.0      4.0
   3 │ d           1.0      0.0
```

请注意，原始`df`中由`src_colnames`索引的列在转置结果中成为列名，原始列名成为新列。这通常用于具有同类元素类型的列，因为其他列的元素类型是对所有转置列的`promote_type`结果。还要注意，默认情况下，从原始`df`的列名创建的新列与`src_namescol`的名称相同。可选的位置参数`dest_namescol`可以改变这一点：

```jldoctest reshape
julia> df2 = DataFrame(a=["x", "y"], b=[1, "two"], c=[3, 4], d=[true, false])
2×4 DataFrame
 Row │ a       b    c      d
     │ String  Any  Int64  Bool
─────┼───────────────────────────
   1 │ x       1        3   true
   2 │ y       two      4  false

julia> permutedims(df2, 1, "different_name")
3×3 DataFrame
 Row │ different_name  x     y
     │ String          Any   Any
─────┼─────────────────────────────
   1 │ b               1     two
   2 │ c               3     4
   3 │ d               true  false
```
