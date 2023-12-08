# 导入和导出数据 (I/O)

## CSV 文件

对于从 CSV 和其他分隔符文本文件读取和写入表格数据，可以使用 [CSV.jl](https://github.com/JuliaData/CSV.jl) 包。

如果你之前没有使用过 CSV.jl 包，那么你可能需要先安装它：
```julia
using Pkg
Pkg.add("CSV")
```

CSV.jl 的函数不会自动加载，必须导入到会话中。
```julia
using CSV
```

现在可以从路径 `input` 的 CSV 文件中读取数据集：
```julia
DataFrame(CSV.File(input))
```

可以将 `DataFrame` 写入到路径 `output` 的 CSV 文件中：
```julia
df = DataFrame(x=1, y=2)
CSV.write(output, df)
```

CSV 函数的行为可以通过关键字参数进行调整。更多信息请参阅 `?CSV.File`，`?CSV.read` 和 `?CSV.write`，或查看在线 [CSV.jl 文档](https://juliadata.github.io/CSV.jl/stable/)。

在简单的情况下，当 CSV.jl 的编译延迟可能成为问题时，可以考虑使用 Julia 标准库的 `DelimitedFiles` 模块。以下是一个示例，展示如何读入数据并进行后处理：

```jldoctest readdlm
julia> using DelimitedFiles, DataFrames

julia> path = joinpath(pkgdir(DataFrames), "docs", "src", "assets", "iris.csv");

julia> data, header = readdlm(path, ',', header=true);

julia> iris_raw = DataFrame(data, vec(header))
150×5 DataFrame
 Row │ SepalLength  SepalWidth  PetalLength  PetalWidth  Species
     │ Any          Any         Any          Any         Any
─────┼──────────────────────────────────────────────────────────────────
   1 │ 5.1          3.5         1.4          0.2         Iris-setosa
   2 │ 4.9          3.0         1.4          0.2         Iris-setosa
   3 │ 4.7          3.2         1.3          0.2         Iris-setosa
   4 │ 4.6          3.1         1.5          0.2         Iris-setosa
   5 │ 5.0          3.6         1.4          0.2         Iris-setosa
   6 │ 5.4          3.9         1.7          0.4         Iris-setosa
   7 │ 4.6          3.4         1.4          0.3         Iris-setosa
   8 │ 5.0          3.4         1.5          0.2         Iris-setosa
  ⋮  │      ⋮           ⋮            ⋮           ⋮             ⋮
 144 │ 6.8          3.2         5.9          2.3         Iris-virginica
 145 │ 6.7          3.3         5.7          2.5         Iris-virginica
 146 │ 6.7          3.0         5.2          2.3         Iris-virginica
 147 │ 6.3          2.5         5.0          1.9         Iris-virginica
 148 │ 6.5          3.0         5.2          2.0         Iris-virginica
 149 │ 6.2          3.4         5.4          2.3         Iris-virginica
 150 │ 5.9          3.0         5.1          1.8         Iris-virginica
                                                        135 rows omitted

julia> iris = identity.(iris_raw)
150×5 DataFrame
 Row │ SepalLength  SepalWidth  PetalLength  PetalWidth  Species
     │ Float64      Float64     Float64      Float64     SubStrin…
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
```

请注意，在我们的例子中：
* `header` 是一个 `Matrix`，因此我们必须将 `vec(header)` 传递给 `DataFrame` 构造器；
* 我们对 `iris_raw` 数据框进行了 `identity` 函数的广播，以执行 `iris_raw` 列的 `eltype` 的缩小；原因是 `readdlm` 函数读入的数据存储在 `data` `Matrix` 中，所以 `iris_raw` 中的所有列最初都有相同的 `eltype` -- 在这种情况下，必须是 `Any`，因为某些列是数值型，而某些列是字符串。

所有这些操作（以及更多）都可以由 CSV.jl 自动处理。

类似地，你可以使用 `DelimitedFiles` 模块的 `writedlm` 函数来保存数据框，例如：

```julia
writedlm("test.csv", Iterators.flatten(([names(iris)], eachrow(iris))), ',')
```

如你所见，将 `iris` 转换成适合 `writedlm` 函数的正确输入，以便你可以创建具有预期格式的 CSV 文件，这是不容易的。因此，CSV.jl 是写入存储在数据框中的数据的 CSV 文件的首选包。

## 其他格式

以下包支持读取和写入其他数据格式（非详尽列表）：
* Apache Arrow（包括 Feather v2）：[Arrow.jl](https://github.com/JuliaData/Arrow.jl)
* Apache Feather（v1）：[Feather.jl](https://github.com/JuliaData/Feather.jl)
* Apache Avro：[Avro.jl](https://github.com/JuliaData/Avro.jl)
* JSON：[JSONTables.jl](https://github.com/JuliaData/JSONTables.jl)
* Parquet：[Parquet2.jl](https://gitlab.com/ExpandingMan/Parquet2.jl)
* Stata，SAS 和 SPSS：[ReadStatTables.jl](https://github.com/junyuan-chen/ReadStatTables.jl)
  （或者 [Queryverse](https://www.queryverse.org/) 用户可以选择 [StatFiles.jl](https://github.com/queryverse/StatFiles.jl)）
* 读取 R 数据文件 (.rda, .RData)：[RData.jl](https://github.com/JuliaData/RData.jl)
* Microsoft Excel (XLSX)：[XLSX.jl](https://github.com/felipenoris/XLSX.jl)
* 复制/粘贴到剪贴板，用于发送数据到电子表格和从电子表格获取数据：[ClipData.jl](https://github.com/pdeffebach/ClipData.jl)
