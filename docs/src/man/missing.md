# 缺失数据

在Julia中，数据的缺失值使用特殊对象`missing`来表示，它是`Missing`类型的唯一实例。

```jldoctest
julia> missing
missing

julia> typeof(missing)
Missing
```

`Missing`类型允许用户创建包含缺失值的向量和`DataFrame`列。这里我们创建一个包含缺失值的向量，返回的向量的元素类型是`Union{Missing, Int64}`。

```jldoctest missings
julia> x = [1, 2, missing]
3-element Vector{Union{Missing, Int64}}:
 1
 2
  missing

julia> eltype(x)
Union{Missing, Int64}

julia> Union{Missing, Int}
Union{Missing, Int64}

julia> eltype(x) == Union{Missing, Int}
true
```

执行操作时可以使用`skipmissing`来排除`missing`值，它返回一个内存高效的迭代器。

```jldoctest missings
julia> skipmissing(x)
skipmissing(Union{Missing, Int64}[1, 2, missing])
```

`skipmissing`的输出可以直接作为函数的参数传入。例如，我们可以找到所有非缺失值的`sum`，或者将非缺失值`collect`到一个新的无缺失值向量中。

```jldoctest missings
julia> sum(skipmissing(x))
3

julia> collect(skipmissing(x))
2-element Vector{Int64}:
 1
 2
```

函数`coalesce`可以用来将缺失值替换为另一个值（注意点，表示应用到`x`的所有条目）：

```jldoctest missings
julia> coalesce.(x, 0)
3-element Vector{Int64}:
 1
 2
 0
```

函数[`dropmissing`](@ref)和[`dropmissing!`](@ref)可以用来从数据框中删除包含`missing`值的行，并分别创建一个新的`DataFrame`或就地修改原始数据。

```jldoctest missings
julia> using DataFrames

julia> df = DataFrame(i=1:5,
                      x=[missing, 4, missing, 2, 1],
                      y=[missing, missing, "c", "d", "e"])
5×3 DataFrame
 Row │ i      x        y
     │ Int64  Int64?   String?
─────┼─────────────────────────
   1 │     1  missing  missing
   2 │     2        4  missing
   3 │     3  missing  c
   4 │     4        2  d
   5 │     5        1  e

julia> dropmissing(df)
2×3 DataFrame
 Row │ i      x      y
     │ Int64  Int64  String
─────┼──────────────────────
   1 │     4      2  d
   2 │     5      1  e
```

可以指定在哪些列中搜索包含`missing`值的行以进行删除。

```jldoctest missings
julia> dropmissing(df, :x)
3×3 DataFrame
 Row │ i      x      y
     │ Int64  Int64  String?
─────┼───────────────────────
   1 │     2      4  missing
   2 │     4      2  d
   3 │     5      1  e
```

默认情况下，[`dropmissing`](@ref)和[`dropmissing!`](@ref)函数在选定用于行删除的列中保持`Union{T, Missing}`元素类型。要删除`Missing`部分（如果存在），请将`disallowmissing`关键字参数设置为`true`（它将成为未来的默认行为）。

```jldoctest missings
julia> dropmissing(df, disallowmissing=true)
2×3 DataFrame
 Row │ i      x      y
     │ Int64  Int64  String
─────┼──────────────────────
   1 │     4      2  d
   2 │     5      1  e
```

有时，允许或不允许数据框的某些列支持缺失值是有用的。这些操作由[`allowmissing`](@ref)、[`allowmissing!`](@ref)、[`disallowmissing`](@ref)和[`disallowmissing!`](@ref)函数支持。这是一个例子：

```jldoctest missings
julia> df = DataFrame(x=1:3, y=4:6)
3×2 DataFrame
 Row │ x      y
     │ Int64  Int64
─────┼──────────────
   1 │     1      4
   2 │     2      5
   3 │     3      6

julia> allowmissing!(df)
3×2 DataFrame
 Row │ x       y
     │ Int64?  Int64?
─────┼────────────────
   1 │      1       4
   2 │      2       5
   3 │      3       6
```

现在，`df`允许其所有列中存在缺失值。我们可以利用这个事实，并将`df`中的一些值设置为`missing`，例如：

```jldoctest missings
julia> df[1, 1] = missing
missing

julia> df
3×2 DataFrame
 Row │ x        y
     │ Int64?   Int64?
─────┼─────────────────
   1 │ missing       4
   2 │       2       5
   3 │       3       6
```

请注意，可以将列选择器作为第二个位置参数传递给[`allowmissing`](@ref)和[`allowmissing!`](@ref)，以限制只更改我们数据框中的某些列。

现在，让我们执行相反的操作，不允许在`df`中有缺失值。我们知道列`:y`不包含缺失值，所以我们可以使用[`disallowmissing`](@ref)函数，将列选择器作为第二个位置参数传入：

```jldoctest missings
julia> disallowmissing(df, :y)
3×2 DataFrame
 Row │ x        y
     │ Int64?   Int64
─────┼────────────────
   1 │ missing      4
   2 │       2      5
   3 │       3      6
```

这个操作创建了一个新的`DataFrame`。如果我们想就地更新`df`，应该使用[`disallowmissing!`](@ref)函数。

如果我们尝试使用`disallowmissing(df)`在整个数据框中不允许缺失值，我们会得到一个错误。然而，通常在所有实际上不包含它们的列中不允许缺失值，但保持有一些`missing`值的列不变，而无需明确列出它们，这是很有用的。这可以通过传递`error=false`关键字参数来实现：

```jldoctest missings
julia> disallowmissing(df, error=false)
3×2 DataFrame
 Row │ x        y
     │ Int64?   Int64
─────┼────────────────
   1 │ missing      4
   2 │       2      5
   3 │       3      6
```

[Missings.jl](https://github.com/JuliaData/Missings.jl)包提供了一些方便的函数来处理缺失值。

最常用的一个是`passmissing`。它是一个高阶函数，接受一些函数`f`作为其参数，并返回一个新函数，如果其位置参数中有任何`missing`，则返回`missing`，否则将函数`f`应用于这些参数。这个功能与那些不支持将`missing`值作为参数传递的函数结合使用是有用的。例如，尝试`uppercase(missing)`会产生一个错误，而以下的操作是可以的：

```jldoctest missings
julia> passmissing(uppercase)("a")
"A"

julia> passmissing(uppercase)(missing)
missing
```

函数`Missings.replace`返回一个迭代器，它用另一个值替换`missing`元素：

```jldoctest missings
julia> using Missings

julia> Missings.replace(x, 1)
Missings.EachReplaceMissing{Vector{Union{Missing, Int64}}, Int64}(Union{Missing, Int64}[1, 2, missing], 1)

julia> collect(Missings.replace(x, 1))
3-element Vector{Int64}:
 1
 2
 1

julia> collect(Missings.replace(x, 1)) == coalesce.(x, 1)
true
```

函数`nonmissingtype`返回`Union{T, Missing}`中的元素类型`T`。

```jldoctest missings
julia> eltype(x)
Union{Missing, Int64}

julia> nonmissingtype(eltype(x))
Int64
```

`missings`函数构建支持缺失值的`Vector`和`Array`，使用可选的第一个参数来指定元素类型。

```jldoctest missings
julia> missings(1)
1-element Vector{Missing}:
 missing

julia> missings(3)
3-element Vector{Missing}:
 missing
 missing
 missing

julia> missings(1, 3)
1×3 Matrix{Missing}:
 missing  missing  missing

julia> missings(Int, 1, 3)
1×3 Matrix{Union{Missing, Int64}}:
 missing  missing  missing
```

请参阅[Julia手册](https://docs.julialang.org/en/v1/manual/missing/)以获取更多关于缺失值的信息。
