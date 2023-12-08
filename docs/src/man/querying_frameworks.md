# 数据操作框架

有三个框架提供了方便的方法来操作 `DataFrame`：DataFramesMeta.jl，DataFrameMacros.jl和Query.jl。它们实现了类似于[dplyr](https://dplyr.tidyverse.org/)或[LINQ](https://en.wikipedia.org/wiki/Language_Integrated_Query)的功能。

## DataFramesMeta.jl

[DataFramesMeta.jl](https://github.com/JuliaStats/DataFramesMeta.jl)包提供了一个便捷且快速的基于宏的接口来处理 `DataFrame`。下面的指南适用于DataFramesMeta.jl的0.10.0版本。

首先安装DataFramesMeta.jl包：

```julia
using Pkg
Pkg.add("DataFramesMeta")
```

该包的主要优点是它通过宏`@transform`，`@select`，`@combine`等提供了更便捷的语法来进行转换函数 `transform`，`select`和`combine`。

DataFramesMeta.jl还从[Chain.jl](https://github.com/jkrumbiegel/Chain.jl)中重新导出了`@chain`宏，允许用户把一个转换的输出作为另一个转换的输入，就像在R中的`|>`和`%>%`一样。

下面我们展示了一些使用该包的选定示例。

首先我们使用逻辑条件对源数据框的行进行子集选择，并选择其两列，重命名其中一列：

```jldoctest dataframesmeta
julia> using DataFramesMeta

julia> df = DataFrame(name=["John", "Sally", "Roger"],
                      age=[54.0, 34.0, 79.0],
                      children=[0, 2, 4])
3×3 DataFrame
 Row │ name    age      children
     │ String  Float64  Int64
─────┼───────────────────────────
   1 │ John       54.0         0
   2 │ Sally      34.0         2
   3 │ Roger      79.0         4

julia> @chain df begin
           @rsubset :age > 40 
           @select(:number_of_children = :children, :name)
       end
2×2 DataFrame
 Row │ number_of_children  name
     │ Int64               String
─────┼────────────────────────────
   1 │                  0  John
   2 │                  4  Roger
```

在下面的示例中，我们展示了DataFramesMeta.jl也支持分割-应用-组合模式：

```jldoctest dataframesmeta
julia> df = DataFrame(key=repeat(1:3, 4), value=1:12)
12×2 DataFrame
 Row │ key    value
     │ Int64  Int64
─────┼──────────────
   1 │     1      1
   2 │     2      2
   3 │     3      3
   4 │     1      4
   5 │     2      5
   6 │     3      6
   7 │     1      7
   8 │     2      8
   9 │     3      9
  10 │     1     10
  11 │     2     11
  12 │     3     12

julia> @chain df begin
           @rsubset :value > 3 
           @by(:key, :min = minimum(:value), :max = maximum(:value))
           @select(:key, :range = :max - :min)
        end
3×2 DataFrame
 Row │ key    range
     │ Int64  Int64
─────┼──────────────
   1 │     1      6
   2 │     2      6
   3 │     3      6

julia> @chain df begin
           groupby(:key)
           @transform :value0 = :value .- minimum(:value)
       end
12×3 DataFrame
 Row │ key    value  value0
     │ Int64  Int64  Int64
─────┼──────────────────────
   1 │     1      1       0
   2 │     2      2       0
   3 │     3      3       0
   4 │     1      4       3
   5 │     2      5       3
   6 │     3      6       3
   7 │     1      7       6
   8 │     2      8       6
   9 │     3      9       6
  10 │     1     10       9
  11 │     2     11       9
  12 │     3     12       9
```

你可以在[DataFramesMeta.jl GitHub页面](https://github.com/JuliaData/DataFramesMeta.jl)上找到更多关于如何使用此包的详细信息。

## DataFrameMacros.jl

[DataFrameMacros.jl](https://github.com/jkrumbiegel/DataFrameMacros.jl) 是DataFramesMeta.jl的替代品，它额外关注于便捷地同时转换多列的解决方案。下面的指南适用于DataFrameMacros.jl的0.3版本。

首先，安装DataFrameMacros.jl包：

```julia
using Pkg
Pkg.add("DataFrameMacros")
```

在DataFrameMacros.jl中，除了`@combine`宏，默认情况下所有宏都是按行处理的。还有一个`@groupby`宏，它允许使用与`@transform`相同的语法在运行时创建分组列，以便在不需要两次编写它们的情况下按新列进行分组。

在下面的示例中，你还可以看到DataFrameMacros.jl的多列功能，其中`mean`一次应用于通过`r"age"`正则表达式选择的两个年龄列。然后使用`"{}"`快捷方式生成新的列名，该快捷方式将转换的列名插入到字符串中。

```jldoctest dataframemacros
julia> using DataFrames, DataFrameMacros, Chain, Statistics

julia> df = DataFrame(name=["John", "Sally", "Roger"],
                      age=[54.0, 34.0, 79.0],
                      children=[0, 2, 4])
3×3 DataFrame
 Row │ name    age      children 
     │ String  Float64  Int64    
─────┼───────────────────────────
   1 │ John       54.0         0
   2 │ Sally      34.0         2
   3 │ Roger      79.0         4

julia> @chain df begin
           @transform :age_months = :age * 12
           @groupby :has_child = :children > 0
           @combine "mean_{}" = mean({r"age"})
       end
2×3 DataFrame
 Row │ has_child  mean_age  mean_age_months 
     │ Bool       Float64   Float64         
─────┼──────────────────────────────────────
   1 │     false      54.0            648.0
   2 │      true      56.5            678.0
```

还有一种能力是将一组多列作为一个单元引用，例如对它们进行聚合，使用`{{ }}`语法。在下面的示例中，将第一季度与其他三个季度的最大值进行比较：

```jldoctest dataframemacros
julia> df = DataFrame(q1 = [12.0, 0.4, 42.7],
                      q2 = [6.4, 2.3, 40.9],
                      q3 = [9.5, 0.2, 13.6],
                      q4 = [6.3, 5.4, 39.3])
3×4 DataFrame
 Row │ q1       q2       q3       q4      
     │ Float64  Float64  Float64  Float64 
─────┼────────────────────────────────────
   1 │    12.0      6.4      9.5      6.3
   2 │     0.4      2.3      0.2      5.4
   3 │    42.7     40.9     13.6     39.3

julia> @transform df :q1_best = :q1 > maximum({{Not(:q1)}})
3×5 DataFrame
 Row │ q1       q2       q3       q4       q1_best 
     │ Float64  Float64  Float64  Float64  Bool    
─────┼─────────────────────────────────────────────
   1 │    12.0      6.4      9.5      6.3     true
   2 │     0.4      2.3      0.2      5.4    false
   3 │    42.7     40.9     13.6     39.3     true
```

## Query.jl

[Query.jl](https://github.com/queryverse/Query.jl)包为`DataFrame`（以及许多其他数据结构）提供了高级数据操作功能。本节提供了该包的简短介绍，[Query.jl文档](http://www.queryverse.org/Query.jl/stable/)提供了包的更全面的文档。这里的指示适用于Query.jl的1.0.0版本。

首先，安装Query.jl包：

```julia
using Pkg
Pkg.add("Query")
```

查询以`@from`宏开始，并由一系列查询命令组成。Query.jl提供了可以过滤，投射，连接，展平和分组`DataFrame`数据的命令。查询可以返回一个迭代器，或者可以将查询结果实体化到各种数据结构中，包括新的`DataFrame`。

一个简单的查询示例如下：

```jldoctest query
julia> using DataFrames, Query

julia> df = DataFrame(name=["John", "Sally", "Roger"],
                      age=[54.0, 34.0, 79.0],
                      children=[0, 2, 4])
3×3 DataFrame
 Row │ name    age      children
     │ String  Float64  Int64
─────┼───────────────────────────
   1 │ John       54.0         0
   2 │ Sally      34.0         2
   3 │ Roger      79.0         4

julia> q1 = @from i in df begin
            @where i.age > 40
            @select {number_of_children=i.children, i.name}
            @collect DataFrame
       end
2×2 DataFrame
 Row │ number_of_children  name
     │ Int64               String
─────┼────────────────────────────
   1 │                  0  John
   2 │                  4  Roger
```

查询以`@from`宏开始。第一个参数`i`是将在后续查询命令中用于引用单个行的范围变量的名称。下一个参数`df`是你想要查询的数据源。此查询中的`@where`命令将通过应用过滤条件`i.age > 40`来过滤源数据。这将过滤掉任何`age`列不大于40的行。然后，`@select`命令将源数据的列投射到新的列结构上。这里的示例应用了三个特定的修改：1) 它只保留源`DataFrame`中的列子集，即转换后的数据不会包含`age`列；2) 它更改了选定的两列的顺序；和3) 它将选定的一列从`children`重命名为`number_of_children`。示例查询使用`{}`语法来实现这一点。Query.jl表达式中的`{}`实例化一个新的[NamedTuple](https://github.com/blackrock/NamedTuples.jl)，即它是编写`@NT(number_of_children=>i.children, name=>i.name)`的快捷方式。`@collect`语句确定查询返回的数据结构。在此示例中，结果以`DataFrame`的形式返回。

没有`@collect`语句的查询返回一个标准的Julia迭代器，可以用任何可以处理迭代器的正常Julia语言结构来使用。以下代码为查询结果返回一个Julia迭代器：

```jldoctest query
julia> q2 = @from i in df begin
                   @where i.age > 40
                   @select {number_of_children=i.children, i.name}
              end; # suppress printing the iterator type

```

可以使用标准的Julia `for`语句遍历结果：

```jldoctest query
julia> total_children = 0
0

julia> for i in q2
           global total_children += i.number_of_children
       end

julia> total_children
4

```

或者可以使用理解（comprehension）来提取一部分行的名称：

```jldoctest query
julia> y = [i.name for i in q2 if i.number_of_children > 0]
1-element Vector{String}:
 "Roger"

```

最后一个示例（只提取名称并应用第二个过滤器）当然可以完全表示为查询表达式：

```jldoctest query
julia> q3 = @from i in df begin
            @where i.age > 40 && i.children > 0
            @select i.name
            @collect
       end
1-element Vector{String}:
 "Roger"

```

以`@collect`语句结束但没有特定类型的查询将查询结果实体化为数组。还要注意`@select`语句中的区别：前面的查询都在`@select`语句中使用`{}`语法将结果投射到表格格式。最后一个查询在`@select`语句中只从每行中选择一个值。

这些例子只是使用[Query.jl](https://github.com/queryverse/Query.jl)可以做的事情的冰山一角，有兴趣的读者可以参考[Query.jl文档](http://www.queryverse.org/Query.jl/stable/)以获取更多信息。
