# 比较

本节将DataFrames.jl与Python、R和Stata中的其他数据操作框架进行比较。

可以使用以下代码创建一个示例数据集：

```julia
using DataFrames
using Statistics

df = DataFrame(grp=repeat(1:2, 3), x=6:-1:1, y=4:9, z=[3:7; missing], id='a':'f')
df2 = DataFrame(grp=[1, 3], w=[10, 11])
```

!!! note

    某些操作会改变表格，因此每个操作都假设在原始数据框上进行。

请注意，在下面的比较中，像 `x -> x >= 1` 这样的谓词可以更简洁地写为 `=>(1)`。后一种形式还具有额外的好处，它在每个Julia会话中只编译一次（而 `x -> x >= 1` 每次引入时都会定义一个新的匿名函数）。

## 与Python包pandas的比较

下表比较了DataFrames.jl的主要函数与Python包pandas（版本1.1.0）的对应函数：

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({'grp': [1, 2, 1, 2, 1, 2],
                   'x': range(6, 0, -1),
                   'y': range(4, 10),
                   'z': [3, 4, 5, 6, 7, None]},
                   index = list('abcdef'))
df2 = pd.DataFrame({'grp': [1, 3], 'w': [10, 11]})
```

由于pandas支持多索引，此示例数据框使用 `a` 到 `f` 作为行索引，而不是单独的 `id` 列。

### 访问数据

| 操作                      | pandas                  | DataFrames.jl                      |
|:-------------------------|:------------------------|:-----------------------------------|
| 位置索引单元格           | `df.iloc[1, 1]`         | `df[2, 2]`                         |
| 位置切片行               | `df.iloc[1:3]`          | `df[2:3, :]`                       |
| 位置切片列               | `df.iloc[:, 1:]`        | `df[:, 2:end]`                     |
| 标签索引行               | `df.loc['c']`           | `df[findfirst(==('c'), df.id), :]` |
| 标签索引列               | `df.loc[:, 'x']`        | `df[:, :x]`                        |
| 标签切片列               | `df.loc[:, ['x', 'z']]` | `df[:, [:x, :z]]`                  |
|                          | `df.loc[:, 'x':'z']`    | `df[:, Between(:x, :z)]`           |
| 混合索引                 | `df.loc['c'][1]`        | `df[findfirst(==('c'), df.id), 2]` |

请注意，Julia使用基于1的索引，两端都包含。特殊关键字 `end` 可用于表示最后一个索引。同样，关键字 `begin` 可用于表示第一个索引。

此外，使用 `findfirst` 函数索引数据框时，返回一个单独的 `DataFrameRow` 对象。如果 `id` 不唯一，可以使用 `findall` 函数或布尔索引。它将返回一个包含所有匹配行的 `DataFrame` 对象。下面的两行代码是功能上等价的：

```julia
df[findall(==('c'), df.id), :]
df[df.id .== 'c', :]
```

DataFrames.jl的索引始终产生一致和可预测的返回类型。相比之下，pandas的 `loc` 函数在索引值为 `'c'` 的索引中只有一个值时返回一个 `Series` 对象，而在有多个索引值为 `'c'` 的行时返回一个 `DataFrame` 对象。

### 常见操作

| 操作                     | pandas                                                         | DataFrames.jl                               |
|:-------------------------|:---------------------------------------------------------------|:--------------------------------------------|
| 求多个值的平均值         | `df['z'].mean(skipna = False)`                                 | `mean(df.z)`                                |
|                          | `df['z'].mean()`                                               | `mean(skipmissing(df.z))`                   |
|                          | `df[['z']].agg(['mean'])`                                      | `combine(df, :z => mean ∘ skipmissing)`     |
| 添加新列                | `df.assign(z1 = df['z'] + 1)`                                  | `transform(df, :z => (v -> v .+ 1) => :z1)` |
| 重命名列                | `df.rename(columns = {'x': 'x_new'})`                          | `rename(df, :x => :x_new)`                  |
| 选择和转换列            | `df.assign(x_mean = df['x'].mean())[['x_mean', 'y']]`          | `select(df, :x => mean, :y)`                |
| 排序行                  | `df.sort_values(by = 'x')`                                     | `sort(df, :x)`                              |
|                          | `df.sort_values(by = ['grp', 'x'], ascending = [True, False])` | `sort(df, [:grp, order(:x, rev = true)])`   |
| 删除缺失行              | `df.dropna()`                                                  | `dropmissing(df)`                           |
| 选择唯一行              | `df.drop_duplicates()`                                         | `unique(df)`                                |

请注意，pandas默认情况下在其分析函数中跳过`NaN`值。相比之下，
Julia函数不会跳过`NaN`值。如果需要，在处理之前可以过滤掉
`NaN`值，例如`mean(Iterators.filter(!isnan, x))`。

Pandas使用`NaN`表示缺失数据和浮点数的"not a number"值。
Julia为表示缺失数据定义了一个特殊值`missing`。DataFrames.jl在默认情况下遵守
Julia中有关传播`missing`值的一般规则。如果需要，
可以使用`skipmissing`函数删除缺失数据。
有关更多信息，请参阅[缺失数据](@ref)部分。

此外，pandas在应用函数后保留原始列名。
DataFrames.jl默认情况下在列名后附加后缀。为了简化起见，
上述示例没有在pandas和DataFrames.jl之间同步列名
（可以向`select`、`transform`和`combine`函数传递`renamecols=false`关键字参数以保留旧列名）。

### 可变操作

| 操作              | pandas                                                | DataFrames.jl                                |
|:-------------------|:------------------------------------------------------|:---------------------------------------------|
| 添加新列          | `df['z1'] = df['z'] + 1`                              | `df.z1 = df.z .+ 1`                          |
|                    |                                                       | `transform!(df, :z => (x -> x .+ 1) => :z1)` |
|                    | `df.insert(1, 'const', 10)`                           | `insertcols!(df, 2, :const => 10)`           |
| 重命名列          | `df.rename(columns = {'x': 'x_new'}, inplace = True)` | `rename!(df, :x => :x_new)`                  |
| 排序行            | `df.sort_values(by = 'x', inplace = True)`            | `sort!(df, :x)`                              |
| 删除缺失行        | `df.dropna(inplace = True)`                           | `dropmissing!(df)`                           |
| 选择唯一行        | `df.drop_duplicates(inplace = True)`                  | `unique!(df)`                                |

一般来说，DataFrames.jl 遵循 Julia 的约定，在函数名中使用 `!` 表示修改行为。

### 分组数据和聚合

DataFrames.jl 提供了 `groupby` 函数来对每个组进行操作。`groupby` 的结果是一个 `GroupedDataFrame` 对象，可以使用 `combine`、`transform` 或 `select` 函数对其进行处理。下表列举了一些常见的分组和聚合用法。

| 操作                           | pandas                                                                                 | DataFrames.jl                                        |
|:--------------------------------|:---------------------------------------------------------------------------------------|:-----------------------------------------------------|
| 按组聚合                       | `df.groupby('grp')['x'].mean()`                                                        | `combine(groupby(df, :grp), :x => mean)`             |
| 聚合后重命名列                 | `df.groupby('grp')['x'].mean().rename("my_mean")`                                      | `combine(groupby(df, :grp), :x => mean => :my_mean)` |
| 添加聚合数据作为列             | `df.join(df.groupby('grp')['x'].mean(), on='grp', rsuffix='_mean')`                    | `transform(groupby(df, :grp), :x => mean)`           |
| ...并选择输出列                | `df.join(df.groupby('grp')['x'].mean(), on='grp', rsuffix='_mean')[['grp', 'x_mean']]` | `select(groupby(df, :grp), :id, :x => mean)`         |

请注意，pandas 对于一维结果返回一个 `Series` 对象，除非之后调用 `reset_index`。
相应的 DataFrames.jl 示例返回一个等价的 `DataFrame` 对象。
考虑第一个示例：

```python
>>> df.groupby('grp')['x'].mean()
grp
1    4
2    3
Name: x, dtype: int64
```

在 DataFrames.jl 中，它看起来是这样的：

```julia
julia> combine(groupby(df, :grp), :x => mean)
2×2 DataFrame
 Row │ grp    x_mean
     │ Int64  Float64
─────┼────────────────
   1 │     1      4.0
   2 │     2      3.0
```

在 DataFrames.jl 中，`GroupedDataFrame` 对象支持高效的键查找。因此，当需要重复执行查找时，它的性能表现良好。

### 更高级的命令

本节包括更复杂的示例。

| 操作                                  | pandas                                                                       | DataFrames.jl                                             |
|:---------------------------------------|:-----------------------------------------------------------------------------|:------------------------------------------------------------------------|
| 复杂函数                              | `df[['z']].agg(lambda v: np.mean(np.cos(v)))`                                | `combine(df, :z => v -> mean(cos, skipmissing(v)))`                     |
| 聚合多列数据                          | `df.agg({'x': max, 'y': min})`                                               | `combine(df, :x => maximum, :y => minimum)`                             |
|                                        | `df[['x', 'y']].mean()`                                                      | `combine(df, [:x, :y] .=> mean)`                                        |
|                                        | `df.filter(regex=("^x")).mean()`                                             | `combine(df, names(df, r"^x") .=> mean)`                                |
| 应用函数于多个变量                    | `df.assign(x_y_cor = np.corrcoef(df.x, df.y)[0, 1])`                         | `transform(df, [:x, :y] => cor)`                                        |
| 逐行操作                              | `df.assign(x_y_min = df.apply(lambda v: min(v.x, v.y), axis=1))`             | `transform(df, [:x, :y] => ByRow(min))`                                 |
|                                        | `df.assign(x_y_argmax = df.apply(lambda v: df.columns[v.argmax()], axis=1))` | `transform(df, AsTable([:x, :y]) => ByRow(argmax))`                     |
| 以DataFrame作为输入                    | `df.groupby('grp').head(2)`                                                  | `combine(d -> first(d, 2), groupby(df, :grp))`                          |
| 以DataFrame作为输出                    | `df[['x']].agg(lambda x: [min(x), max(x)])`                                  | `combine(df, :x => (x -> (x=[minimum(x), maximum(x)],)) => AsTable)`  |

请注意，pandas在`groupby`之后保留相同的行顺序，而DataFrames.jl在`combine`操作之后按提供的键进行分组，
但`select`和`transform`保留原始的行顺序。

### 连接数据框

DataFrames.jl支持类似关系数据库的连接操作。

| 操作             | pandas                                         | DataFrames.jl                   |
|:----------------------|:-----------------------------------------------|:--------------------------------|
| 内连接            | `pd.merge(df, df2, how = 'inner', on = 'grp')` | `innerjoin(df, df2, on = :grp)` |
| 外连接            | `pd.merge(df, df2, how = 'outer', on = 'grp')` | `outerjoin(df, df2, on = :grp)` |
| 左连接             | `pd.merge(df, df2, how = 'left', on = 'grp')`  | `leftjoin(df, df2, on = :grp)`  |
| 右连接            | `pd.merge(df, df2, how = 'right', on = 'grp')` | `rightjoin(df, df2, on = :grp)` |
| 半连接 (过滤) | `df[df.grp.isin(df2.grp)]`                     | `semijoin(df, df2, on = :grp)`  |
| 反连接 (过滤) | `df[~df.grp.isin(df2.grp)]`                    | `antijoin(df, df2, on = :grp)`  |

对于多列连接，pandas和DataFrames.jl都接受一个数组作为`on`关键字参数。

在半连接和反连接的情况下，pandas中的`isin`函数仍然可以使用，只要连接键以元组的形式[组合起来](https://stackoverflow.com/questions/63660610/how-to-perform-semi-join-with-multiple-columns-in-pandas)。
在DataFrames.jl中，只需在`on`关键字参数中指定连接键的数组即可正常工作。

## 与R包dplyr的比较

下表比较了DataFrames.jl的主要函数与R包dplyr（版本1）的对应关系：

```R
df <- tibble(grp = rep(1:2, 3), x = 6:1, y = 4:9,
             z = c(3:7, NA), id = letters[1:6])
```

| 操作                | dplyr                          | DataFrames.jl                          |
|:-------------------------|:-------------------------------|:---------------------------------------|
| 多值归约   | `summarize(df, mean(x))`       | `combine(df, :x => mean)`              |
| 添加新列          | `mutate(df, x_mean = mean(x))` | `transform(df, :x => mean => :x_mean)` |
| 重命名列           | `rename(df, x_new = x)`        | `rename(df, :x => :x_new)`             |
| 选择列             | `select(df, x, y)`             | `select(df, :x, :y)`                   |
| 选择并转换列 | `transmute(df, mean(x), y)`    | `select(df, :x => mean, :y)`           |
| 选择行                | `filter(df, x >= 1)`           | `subset(df, :x => ByRow(x -> x >= 1))` |
| 排序行                | `arrange(df, x)`               | `sort(df, :x)`                         |

与dplyr类似，这些函数中的一些可以应用于分组的数据框，此时它们按组进行操作：

| 操作                | dplyr                                      | DataFrames.jl                               |
|:-------------------------|:-------------------------------------------|:--------------------------------------------|
| 多值归约   | `summarize(group_by(df, grp), mean(x))`    | `combine(groupby(df, :grp), :x => mean)`    |
| 添加新列          | `mutate(group_by(df, grp), mean(x))`       | `transform(groupby(df, :grp), :x => mean)`  |
| 选择并转换列 | `transmute(group_by(df, grp), mean(x), y)` | `select(groupby(df, :grp), :x => mean, :y)` |

下表比较了更高级的命令：

| 操作                 | dplyr                                                     | DataFrames.jl                                                              |
|:--------------------------|:----------------------------------------------------------|:---------------------------------------------------------------------------|
| 复杂函数          | `summarize(df, mean(x, na.rm = T))`                       | `combine(df, :x => x -> mean(skipmissing(x)))`                             |
| 转换多列数据 | `summarize(df, max(x), min(y))`                           | `combine(df, :x => maximum,  :y => minimum)`                               |
|                           | `summarize(df, across(c(x, y), mean))`                    | `combine(df, [:x, :y] .=> mean)`                                           |
|                           | `summarize(df, across(starts_with("x"), mean))`           | `combine(df, names(df, r"^x") .=> mean)`                                   |
|                           | `summarize(df, across(c(x, y), list(max, min)))`          | `combine(df, ([:x, :y] .=> [maximum minimum])...)`                         |
| 多变量函数     | `mutate(df, cor(x, y))`                                   | `transform(df, [:x, :y] => cor)`                                           |
| 逐行操作                  | `mutate(rowwise(df), min(x, y))`                          | `transform(df, [:x, :y] => ByRow(min))`                                    |
|                           | `mutate(rowwise(df), which.max(c_across(matches("^x"))))` | `transform(df, AsTable(r"^x") => ByRow(argmax))`                           |
| 以DataFrame作为输入        | `summarize(df, head(across(), 2))`                        | `combine(d -> first(d, 2), df)`                                            |
| 以DataFrame作为输出       | `summarize(df, tibble(value = c(min(x), max(x))))`        | `combine(df, :x => (x -> (value = [minimum(x), maximum(x)],)) => AsTable)` |


## 与 R 包 data.table 的比较

下表比较了 DataFrames.jl 的主要函数与 R 包 data.table（版本 1.14.1）的对应关系。

```R
library(data.table)
df  <- data.table(grp = rep(1:2, 3), x = 6:1, y = 4:9,
                  z = c(3:7, NA), id = letters[1:6])
df2 <- data.table(grp=c(1,3), w = c(10,11))
```

| 操作                               | data.table                                       | DataFrames.jl                                |
|:-----------------------------------|:-------------------------------------------------|:---------------------------------------------|
| 多值归约                           | `df[, .(mean(x))]`                               | `combine(df, :x => mean)`                    |
| 添加新列                           | `df[, x_mean:=mean(x) ]`                         | `transform!(df, :x => mean => :x_mean)`      |
| 重命名列（原地操作）               | `setnames(df, "x", "x_new")`                     | `rename!(df, :x => :x_new)`                  |
| 重命名多列（原地操作）             | `setnames(df, c("x", "y"), c("x_new", "y_new"))` | `rename!(df, [:x, :y] .=> [:x_new, :y_new])` |
| 选择列作为 DataFrame               | `df[, .(x, y)]`                                  | `select(df, :x, :y)`                         |
| 选择列作为向量                     | `df[, x]`                                        | `df[!, :x]`                                  |
| 删除列                             | `df[, -"x"]`                                     | `select(df, Not(:x))`                        |
| 删除列（原地操作）                 | `df[, x:=NULL]`                                  | `select!(df, Not(:x))`                       |
| 删除多列（原地操作）               | `df[, c("x", "y"):=NULL]`                        | `select!(df, Not([:x, :y]))`                 |
| 选择并转换列                       | `df[, .(mean(x), y)]`                            | `select(df, :x => mean, :y)`                 |
| 选择行                             | `df[ x >= 1 ]`                                   | `filter(:x => >=(1), df)`                    |
| 排序行（原地操作）                 | `setorder(df, x)`                                | `sort!(df, :x)`                              |
| 排序行                             | `df[ order(x) ]`                                 | `sort(df, :x)`                               |

### 数据分组和聚合

| 操作                         | data.table                                        | DataFrames.jl                             |
|:----------------------------|:--------------------------------------------------|:------------------------------------------|
| 多值归约                     | `df[, mean(x), by=id ]`                           | `combine(groupby(df, :id), :x => mean)`   |
| 添加新列（原地操作）         | `df[, x_mean:=mean(x), by=id]`                    | `transform!(groupby(df, :id), :x => mean)`|
| 选择并转换列                 | `df[, .(x_mean = mean(x), y), by=id]`             | `select(groupby(df, :id), :x => mean, :y)`|

### 更高级的命令

| 操作                                 | data.table                                                                                 | DataFrames.jl                                                               |
|:--------------------------------------|:-------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------|
| 复杂函数                             | `df[, .(mean(x, na.rm=TRUE)) ]`                                                            | `combine(df, :x => x -> mean(skipmissing(x)))`                              |
| 转换特定行（原地操作）               | `df[x<=0, x:=0]`                                                                           | `df.x[df.x .<= 0] .= 0`                                                     |
| 转换多列                             | `df[, .(max(x), min(y)) ]`                                                                 | `combine(df, :x => maximum, :y => minimum)`                                 |
|                                       | `df[, lapply(.SD, mean), .SDcols = c("x", "y") ]`                                          | `combine(df, [:x, :y] .=> mean)`                                            |
|                                       | `df[, lapply(.SD, mean), .SDcols = patterns("*x") ]`                                       | `combine(df, names(df, r"^x") .=> mean)`                                    |
|                                       | `df[, unlist(lapply(.SD, function(x) c(max=max(x), min=min(x)))), .SDcols = c("x", "y") ]` | `combine(df, ([:x, :y] .=> [maximum minimum])...)`                          |
| 多变量函数                           | `df[, .(cor(x,y)) ]`                                                                       | `transform(df, [:x, :y] => cor)`                                            |
| 逐行操作                             | `df[, min_xy := min(x, y), by = 1:nrow(df)]`                                               | `transform!(df, [:x, :y] => ByRow(min))`                                    |
|                                       | `df[, argmax_xy := which.max(.SD) , .SDcols = patterns("*x"), by = 1:nrow(df) ]`           | `transform!(df, AsTable(r"^x") => ByRow(argmax))`                           |
| 输出为 DataFrame                     | `df[, .SD[1], by=grp]`                                                                     | `combine(groupby(df, :grp), first)`                                         |
| 输出为 DataFrame                     | `df[, .SD[which.max(x)], by=grp]`                                                          | `combine(groupby(df, :grp), sdf -> sdf[argmax(sdf.x), :])`                  |

### 连接数据框

| 操作             | data.table                                      | DataFrames.jl                   |
|:------------------|:------------------------------------------------|:--------------------------------|
| 内连接           | `merge(df, df2, on = "grp")`                    | `innerjoin(df, df2, on = :grp)` |
| 外连接           | `merge(df, df2, all = TRUE, on = "grp")`        | `outerjoin(df, df2, on = :grp)` |
| 左连接           | `merge(df, df2, all.x = TRUE, on = "grp")`      | `leftjoin(df, df2, on = :grp)`  |
| 右连接           | `merge(df, df2, all.y = TRUE, on = "grp")`      | `rightjoin(df, df2, on = :grp)` |
| 反连接（过滤）   | `df[!df2, on = "grp" ]`                         | `antijoin(df, df2, on = :grp)`  |
| 半连接（过滤）   | `merge(df1, df2[, .(grp)])   `                  | `semijoin(df, df2, on = :grp)`  |


## 与 Stata（8 版及以上版本）的比较

下表比较了 DataFrames.jl 的主要功能与 Stata 的对应关系：

| 操作                   | Stata                   | DataFrames.jl                           |
|:-----------------------|:------------------------|:----------------------------------------|
| 多值合并               | `collapse (mean) x`     | `combine(df, :x => mean)`               |
| 添加新列               | `egen x_mean = mean(x)` | `transform!(df, :x => mean => :x_mean)` |
| 重命名列               | `rename x x_new`        | `rename!(df, :x => :x_new)`             |
| 选择列                 | `keep x y`              | `select!(df, :x, :y)`                   |
| 选择行                 | `keep if x >= 1`        | `subset!(df, :x => ByRow(x -> x >= 1))` |
| 排序行                 | `sort x`                | `sort!(df, :x)`                         |

需要注意的是后缀 `!`（例如 `transform!`、`select!` 等）表示操作会直接在原始数据框上进行修改，与 Stata 中的操作方式类似。

其中一些函数可以应用于分组数据框，此时它们将按组进行操作：

| 操作                   | Stata                            | DataFrames.jl                               |
|:-----------------------|:---------------------------------|:--------------------------------------------|
| 添加新列               | `egen x_mean = mean(x), by(grp)` | `transform!(groupby(df, :grp), :x => mean)` |
| 多值合并               | `collapse (mean) x, by(grp)`     | `combine(groupby(df, :grp), :x => mean)`    |

下表比较了更高级的命令：

| 操作                      | Stata                          | DataFrames.jl                                              |
|:--------------------------|:-------------------------------|:-----------------------------------------------------------|
| 转换特定行                | `replace x = 0 if x <= 0`      | `transform(df, :x => (x -> ifelse.(x .<= 0, 0, x)) => :x)` |
| 转换多个列                | `collapse (max) x (min) y`     | `combine(df, :x => maximum,  :y => minimum)`               |
|                           | `collapse (mean) x y`          | `combine(df, [:x, :y] .=> mean)`                           |
|                           | `collapse (mean) x*`           | `combine(df, names(df, r"^x") .=> mean)`                   |
|                           | `collapse (max) x y (min) x y` | `combine(df, ([:x, :y] .=> [maximum minimum])...)`         |
| 多变量函数                | `egen z = corr(x y)`           | `transform!(df, [:x, :y] => cor => :z)`                    |
| 逐行处理                  | `egen z = rowmin(x y)`         | `transform!(df, [:x, :y] => ByRow(min) => :z)`             |
