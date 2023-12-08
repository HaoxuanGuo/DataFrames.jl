# [分类数据](@id man-categorical)

我们经常需要处理数据框中的某些列，这些列只包含少数几个级别的数据：

```jldoctest categorical
julia> v = ["Group A", "Group A", "Group A", "Group B", "Group B", "Group B"]
6-element Vector{String}:
 "Group A"
 "Group A"
 "Group A"
 "Group B"
 "Group B"
 "Group B"
```

在`Vector`中使用的朴素编码将此向量的每个条目表示为一个完整的字符串。相比之下，我们可以通过将字符串替换为对少数级别的索引，更有效地表示数据。这样做有两个好处。第一，这种向量往往会使用更少的内存。第二，可以使用`groupby`函数有效地对其进行分组。

有两种常见的类型允许执行级别池化：
* 来自PooledArrays.jl的`PooledVector`；
* 来自CategoricalArrays.jl的`CategoricalVector`。

`PooledVector`和`CategoricalVector`之间的区别如下：
* `PooledVector`适用于数据压缩是唯一目标的情况；
* `CategoricalVector`被设计为同时提供全面的支持
   用于处理分类变量，无论是无序的
  （名义变量）还是有序的类别（序数变量），但代价是
  只允许`AbstractString`、`AbstractChar`或`Number`元素类型
  （可选地与`Missing`联合）。

当数组中的唯一值（级别）应该尊重有意义的排序时，`CategoricalVector`特别有用，例如在打印表格、绘制图表或拟合回归模型时。CategoricalArrays.jl提供了设置和检索此顺序以及根据它比较值的函数。相反，`PooledVector`类型实际上是`Vector`的替代品，几乎没有用户可见的差异，除了更低的内存使用和更高的性能。

下面我们展示了使用CategoricalArrays.jl的一些选定示例。
有关分类数组的更多信息，请参阅[CategoricalArrays.jl文档](https://categoricalarrays.juliadata.org/stable/)包。
也请注意，本节我们只讨论向量，因为我们正在考虑数据框上下文。然而，一般来说，这两个包都允许处理任何维度的数组。

要跟随下面的例子，你需要先安装CategoricalArrays.jl包。

```jldoctest categorical
julia> using CategoricalArrays

julia> cv = categorical(v)
6-element CategoricalArray{String,1,UInt32}:
 "Group A"
 "Group A"
 "Group A"
 "Group B"
 "Group B"
 "Group B"
```

`CategoricalVectors`支持缺失值。

```jldoctest categorical
julia> cv = categorical(["Group A", missing, "Group A",
                         "Group B", "Group B", missing])
6-element CategoricalArray{Union{Missing, String},1,UInt32}:
 "Group A"
 missing
 "Group A"
 "Group B"
 "Group B"
 missing
```

除了有效地表示重复数据外，`CategoricalArray`
类型还允许我们使用`levels`函数随时有效地确定变量的允许级别（注意，级别可能实际上在数据中使用或未使用）：

```jldoctest categorical
julia> levels(cv)
2-element Vector{String}:
 "Group A"
 "Group B"
```

`levels!`函数还允许改变级别出现的顺序，这在显示目的或处理有序变量时可能有用。

```jldoctest categorical
julia> levels!(cv, ["Group B", "Group A"])
6-element CategoricalArray{Union{Missing, String},1,UInt32}:
 "Group A"
 missing
 "Group A"
 "Group B"
 "Group B"
 missing

julia> levels(cv)
2-element Vector{String}:
 "Group B"
 "Group A"

julia> sort(cv)
6-element CategoricalArray{Union{Missing, String},1,UInt32}:
 "Group B"
 "Group B"
 "Group A"
 "Group A"
 missing
 missing
```

默认情况下，`CategoricalVector`能够表示``2^{32}``个不同的级别。你可以通过调用`compress`函数使用更少的内存：

```jldoctest categorical
julia> cv = compress(cv)
6-element CategoricalArray{Union{Missing, String},1,UInt8}:
 "Group A"
 missing
 "Group A"
 "Group B"
 "Group B"
 missing

```

`categorical`函数还接受一个关键字参数`compress`，当设置为`true`时，等同于在新向量上调用`compress`：

```jldoctest categorical
julia> cv1 = categorical(["A", "B"], compress=true)
2-element CategoricalArray{String,1,UInt8}:
 "A"
 "B"
```

如果`ordered`关键字参数设置为`true`，结果
`CategoricalVector`将是有序的，这意味着它的级别可以被测试
为顺序（而不是抛出错误）：

```jldoctest categorical
julia> cv2 = categorical(["A", "B"], ordered=true)
2-element CategoricalArray{String,1,UInt32}:
 "A"
 "B"

julia> cv1[1] < cv1[2]
ERROR: ArgumentError: Unordered CategoricalValue objects cannot be tested for order using <. Use isless instead, or call the ordered! function on the parent array to change this

julia> cv2[1] < cv2[2]
true
```

你可以使用`isordered`函数检查`CategoricalVector`是否有序，并使用`ordered!`函数在有序和无序之间切换。

```jldoctest categorical
julia> isordered(cv1)
false

julia> ordered!(cv1, true)
2-element CategoricalArray{String,1,UInt8}:
 "A"
 "B"

julia> isordered(cv1)
true

julia> cv1[1] < cv1[2]
true
```