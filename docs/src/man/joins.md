# 数据库风格的连接

## 连接的介绍

我们经常需要将两个或更多的数据集合并在一起，以提供我们正在研究的主题的完整画面。例如，假设我们有以下两个数据集：

```jldoctest joins
julia> using DataFrames

julia> people = DataFrame(ID=[20, 40], Name=["John Doe", "Jane Doe"])
2×2 DataFrame
 Row │ ID     Name
     │ Int64  String
─────┼─────────────────
   1 │    20  John Doe
   2 │    40  Jane Doe

julia> jobs = DataFrame(ID=[20, 40], Job=["Lawyer", "Doctor"])
2×2 DataFrame
 Row │ ID     Job
     │ Int64  String
─────┼───────────────
   1 │    20  Lawyer
   2 │    40  Doctor
```

我们可能希望使用一个包含每个ID的姓名和工作的更大的数据集。我们可以使用`innerjoin`函数做到这一点：

```jldoctest joins
julia> innerjoin(people, jobs, on = :ID)
2×3 DataFrame
 Row │ ID     Name      Job
     │ Int64  String    String
─────┼─────────────────────────
   1 │    20  John Doe  Lawyer
   2 │    40  Jane Doe  Doctor
```

在关系数据库理论中，这种操作通常被称为连接。用于确定在连接过程中应该合并哪些行的列被称为键。

以下函数提供了执行七种类型的连接：

- `innerjoin`：输出包含在所有传入的数据框中存在的键值的行。
- `leftjoin`：输出包含在第一个（左）参数中存在的键值的行，无论该值是否在第二个（右）参数中存在。
- `rightjoin`：输出包含在第二个（右）参数中存在的键值的行，无论该值是否在第一个（左）参数中存在。
- `outerjoin`：输出包含在任何传入的数据框中存在的键值的行。
- `semijoin`：类似于内连接，但输出仅限于来自第一个（左）参数的列。
- `antijoin`：输出包含在第一个（左）但不在第二个（右）参数中存在的键值的行。与`semijoin`一样，输出仅限于来自第一个（左）参数的列。
- `crossjoin`：输出是来自所有传入数据框的行的笛卡尔积。

有关更多信息，请参见[SQL连接的维基百科页面](https://en.wikipedia.org/wiki/Join_(SQL))。

以下是不同类型的连接的例子：

```jldoctest joins
julia> jobs = DataFrame(ID=[20, 60], Job=["Lawyer", "Astronaut"])
2×2 DataFrame
 Row │ ID     Job
     │ Int64  String
─────┼──────────────────
   1 │    20  Lawyer
   2 │    60  Astronaut

julia> innerjoin(people, jobs, on = :ID)
1×3 DataFrame
 Row │ ID     Name      Job
     │ Int64  String    String
─────┼─────────────────────────
   1 │    20  John Doe  Lawyer

julia> leftjoin(people, jobs, on = :ID)
2×3 DataFrame
 Row │ ID     Name      Job
     │ Int64  String    String?
─────┼──────────────────────────
   1 │    20  John Doe  Lawyer
   2 │    40  Jane Doe  missing

julia> rightjoin(people, jobs, on = :ID)
2×3 DataFrame
 Row │ ID     Name      Job
     │ Int64  String?   String
─────┼────────────────────────────
   1 │    20  John Doe  Lawyer
   2 │    60  missing   Astronaut

julia> outerjoin(people, jobs, on = :ID)
3×3 DataFrame
 Row │ ID     Name      Job
     │ Int64  String?   String?
─────┼────────────────────────────
   1 │    20  John Doe  Lawyer
   2 │    40  Jane Doe  missing
   3 │    60  missing   Astronaut

julia> semijoin(people, jobs, on = :ID)
1×2 DataFrame
 Row │ ID     Name
     │ Int64  String
─────┼─────────────────
   1 │    20  John Doe

julia> antijoin(people, jobs, on = :ID)
1×2 DataFrame
 Row │ ID     Name
     │ Int64  String
─────┼─────────────────
   1 │    40  Jane Doe
```

交叉连接是唯一一种不使用`on`键的连接：

```jldoctest joins
julia> crossjoin(people, jobs, makeunique = true)
4×4 DataFrame
 Row │ ID     Name      ID_1   Job
     │ Int64  String    Int64  String
─────┼───────────────────────────────────
   1 │    20  John Doe     20  Lawyer
   2 │    20  John Doe     60  Astronaut
   3 │    40  Jane Doe     20  Lawyer
   4 │    40  Jane Doe     60  Astronaut
```

## 键值比较和浮点值

两个或更多数据框的键值使用`isequal`函数进行比较。这与Julia Base中的`Set`和`Dict`类型保持一致。

不建议使用浮点数作为键：浮点比较可能会出现令人惊讶和不可预测的结果。如果你确实使用了浮点键，请注意，默认情况下，当键包含`-0.0`（负零）或`NaN`值时，会引发错误。以下是一个例子：

```jldoctest joins
julia> innerjoin(DataFrame(id=[-0.0]), DataFrame(id=[0.0]), on=:id)
ERROR: ArgumentError: Currently for numeric values `NaN` and `-0.0` in their real or imaginary components are not allowed. Such value was found in column :id in left data frame. Use CategoricalArrays.jl to wrap these values in a CategoricalVector to perform the requested join.
```

可以通过将键值包装在[categorical](@ref man-categorical)向量中来覆盖这一点。

## 基于不同名称的关键列进行连接

为了在左表和右表中的关键字拥有不同名称的情况下连接数据框，你可以传递 `left => right` 对作为 `on` 参数：

```jldoctest joins
julia> a = DataFrame(ID=[20, 40], Name=["John Doe", "Jane Doe"])
2×2 DataFrame
 Row │ ID     Name
     │ Int64  String
─────┼─────────────────
   1 │    20  John Doe
   2 │    40  Jane Doe

julia> b = DataFrame(IDNew=[20, 40], Job=["Lawyer", "Doctor"])
2×2 DataFrame
 Row │ IDNew  Job
     │ Int64  String
─────┼───────────────
   1 │    20  Lawyer
   2 │    40  Doctor

julia> innerjoin(a, b, on = :ID => :IDNew)
2×3 DataFrame
 Row │ ID     Name      Job
     │ Int64  String    String
─────┼─────────────────────────
   1 │    20  John Doe  Lawyer
   2 │    40  Jane Doe  Doctor
```

这是另一个包含多列的例子：

```jldoctest joins
julia> a = DataFrame(City=["Amsterdam", "London", "London", "New York", "New York"],
                     Job=["Lawyer", "Lawyer", "Lawyer", "Doctor", "Doctor"],
                     Category=[1, 2, 3, 4, 5])
5×3 DataFrame
 Row │ City       Job     Category
     │ String     String  Int64
─────┼─────────────────────────────
   1 │ Amsterdam  Lawyer         1
   2 │ London     Lawyer         2
   3 │ London     Lawyer         3
   4 │ New York   Doctor         4
   5 │ New York   Doctor         5

julia> b = DataFrame(Location=["Amsterdam", "London", "London", "New York", "New York"],
                     Work=["Lawyer", "Lawyer", "Lawyer", "Doctor", "Doctor"],
                     Name=["a", "b", "c", "d", "e"])
5×3 DataFrame
 Row │ Location   Work    Name
     │ String     String  String
─────┼───────────────────────────
   1 │ Amsterdam  Lawyer  a
   2 │ London     Lawyer  b
   3 │ London     Lawyer  c
   4 │ New York   Doctor  d
   5 │ New York   Doctor  e

julia> innerjoin(a, b, on = [:City => :Location, :Job => :Work])
9×4 DataFrame
 Row │ City       Job     Category  Name
     │ String     String  Int64     String
─────┼─────────────────────────────────────
   1 │ Amsterdam  Lawyer         1  a
   2 │ London     Lawyer         2  b
   3 │ London     Lawyer         3  b
   4 │ London     Lawyer         2  c
   5 │ London     Lawyer         3  c
   6 │ New York   Doctor         4  d
   7 │ New York   Doctor         5  d
   8 │ New York   Doctor         4  e
   9 │ New York   Doctor         5  e
```

## 处理重复键和追踪源数据框

此外，注意到在最后的连接中，第2行和第3行在两个被连接的 `DataFrame` 中的 `on` 变量上具有相同的值。在这种情况下，`innerjoin`，`outerjoin`，`leftjoin` 和 `rightjoin` 将生成所有匹配行的组合。在我们的示例中，第2行到第5行是作为结果创建的。对于两个被连接的 `DataFrame` 中的第4行和第5行，也可以观察到相同的行为。

为了检查作为 `on` 参数传递的列在每个输入数据框中定义了唯一键（根据 `isequal`），你可以将 `validate` 关键字参数设置为一个两元素的元组或一对 `Bool` 值，每个元素表示是否对相应的数据框运行检查。以下是上述连接操作的示例：

```jldoctest joins
julia> innerjoin(a, b, on = [(:City => :Location), (:Job => :Work)], validate=(true, true))
ERROR: ArgumentError: Merge key(s) are not unique in both df1 and df2. df1 contains 2 duplicate keys: (City = "London", Job = "Lawyer") and (City = "New York", Job = "Doctor"). df2 contains 2 duplicate keys: (Location = "London", Work = "Lawyer") and (Location = "New York", Work = "Doctor").
```

最后，使用 `source` 关键字参数，你可以向结果数据框添加一列，指示给定行仅出现在左边，右边还是两个数据框中。这是一个例子：

```jldoctest joins
julia> a = DataFrame(ID=[20, 40], Name=["John", "Jane"])
2×2 DataFrame
 Row │ ID     Name
     │ Int64  String
─────┼───────────────
   1 │    20  John
   2 │    40  Jane

julia> b = DataFrame(ID=[20, 60], Job=["Lawyer", "Doctor"])
2×2 DataFrame
 Row │ ID     Job
     │ Int64  String
─────┼───────────────
   1 │    20  Lawyer
   2 │    60  Doctor

julia> outerjoin(a, b, on=:ID, validate=(true, true), source=:source)
3×4 DataFrame
 Row │ ID     Name     Job      source
     │ Int64  String?  String?  String
─────┼─────────────────────────────────────
   1 │    20  John     Lawyer   both
   2 │    40  Jane     missing  left_only
   3 │    60  missing  Doctor   right_only
```

注意，这次我们也使用了 `validate` 关键字参数，它没有产生错误，因为在两个源数据框中定义的键都是唯一的。

## 重命名连接的列

你经常需要追踪源数据框。这个特性可以通过 `renamecols` 关键字参数来实现：

```jldoctest joins
julia> innerjoin(a, b, on=:ID, renamecols = "_left" => "_right")
1×3 DataFrame
 Row │ ID     Name_left  Job_right
     │ Int64  String     String
─────┼─────────────────────────────
   1 │    20  John       Lawyer
```

在上面的例子中，我们给左表的非键列添加了 `"_left"` 后缀，给右表的非键列添加了 `"_right"` 后缀。

另外，也可以传入一个用于转换列名的函数：

```jldoctest joins
julia> innerjoin(a, b, on=:ID, renamecols = lowercase => uppercase)
1×3 DataFrame
 Row │ ID     name    JOB
     │ Int64  String  String
─────┼───────────────────────
   1 │    20  John    Lawyer

```

## 在连接中匹配缺失值

默认情况下，当你尝试在一个有 `missing` 值的键上执行连接操作时，会出现错误：

```jldoctest joins
julia> df1 = DataFrame(id=[1, missing, 3], a=1:3)
3×2 DataFrame
 Row │ id       a
     │ Int64?   Int64
─────┼────────────────
   1 │       1      1
   2 │ missing      2
   3 │       3      3

julia> df2 = DataFrame(id=[1, 2, missing], b=1:3)
3×2 DataFrame
 Row │ id       b
     │ Int64?   Int64
─────┼────────────────
   1 │       1      1
   2 │       2      2
   3 │ missing      3

julia> innerjoin(df1, df2, on=:id)
ERROR: ArgumentError: Missing values in key columns are not allowed when matchmissing == :error. `missing` found in column :id in left data frame.
```

如果你希望 `missing` 值被视为相等，可以传入 `matchmissing=:equal` 关键字参数：

```jldoctest joins
julia> innerjoin(df1, df2, on=:id, matchmissing=:equal)
2×3 DataFrame
 Row │ id       a      b
     │ Int64?   Int64  Int64
─────┼───────────────────────
   1 │       1      1      1
   2 │ missing      2      3
```

或者你可能希望删除所有包含 `missing` 值的行。在这种情况下，传入 `matchmissing=:notequal`：

```jldoctest joins
julia> innerjoin(df1, df2, on=:id, matchmissing=:notequal)
1×3 DataFrame
 Row │ id      a      b
     │ Int64?  Int64  Int64
─────┼──────────────────────
   1 │      1      1      1
```

## 指定连接结果中的行顺序

默认情况下，连接操作产生的行顺序是未定义的：

```jldoctest joins
julia> df_left = DataFrame(id=[1, 2, 4, 5], left=1:4)
4×2 DataFrame
 Row │ id     left
     │ Int64  Int64
─────┼──────────────
   1 │     1      1
   2 │     2      2
   3 │     4      3
   4 │     5      4

julia> df_right = DataFrame(id=[2, 1, 3, 6, 7], right=1:5)
5×2 DataFrame
 Row │ id     right
     │ Int64  Int64
─────┼──────────────
   1 │     2      1
   2 │     1      2
   3 │     3      3
   4 │     6      4
   5 │     7      5

julia> outerjoin(df_left, df_right, on=:id)
7×3 DataFrame
 Row │ id     left     right
     │ Int64  Int64?   Int64?
─────┼─────────────────────────
   1 │     2        2        1
   2 │     1        1        2
   3 │     4        3  missing
   4 │     5        4  missing
   5 │     3  missing        3
   6 │     6  missing        4
   7 │     7  missing        5
```

如果你希望结果保持左表的行顺序，可以传入 `order=:left` 关键字参数：

```jldoctest joins
julia> outerjoin(df_left, df_right, on=:id, order=:left)
7×3 DataFrame
 Row │ id     left     right
     │ Int64  Int64?   Int64?
─────┼─────────────────────────
   1 │     1        1        2
   2 │     2        2        1
   3 │     4        3  missing
   4 │     5        4  missing
   5 │     3  missing        3
   6 │     6  missing        4
   7 │     7  missing        5
```

注意，在这种情况下，左表中缺失的键会被放在存在的键之后。

类似地，`order=:right` 会保持右表的顺序（并将不在其中的键放在末尾）：

```jldoctest joins
julia> outerjoin(df_left, df_right, on=:id, order=:right)
7×3 DataFrame
 Row │ id     left     right
     │ Int64  Int64?   Int64?
─────┼─────────────────────────
   1 │     2        2        1
   2 │     1        1        2
   3 │     3  missing        3
   4 │     6  missing        4
   5 │     7  missing        5
   6 │     4        3  missing
   7 │     5        4  missing
```

## 原地左连接

一个常见的操作是从参考表中添加数据到主表。可以使用 `leftjoin!` 函数进行此类原地更新。在这种情况下，左表会通过右表中的匹配行进行原地更新。

```jldoctest joins
julia> main = DataFrame(id=1:4, main=1:4)
4×2 DataFrame
 Row │ id     main
     │ Int64  Int64
─────┼──────────────
   1 │     1      1
   2 │     2      2
   3 │     3      3
   4 │     4      4

julia> leftjoin!(main, DataFrame(id=[2, 4], info=["a", "b"]), on=:id);

julia> main
4×3 DataFrame
 Row │ id     main   info
     │ Int64  Int64  String?
─────┼───────────────────────
   1 │     1      1  missing
   2 │     2      2  a
   3 │     3      3  missing
   4 │     4      4  b
```

请注意，在这种情况下，左表的行顺序和数量不会改变。因此，特别的，右表中不允许有重复的键：

```
julia> leftjoin!(main, DataFrame(id=[2, 2], info_bad=["a", "b"]), on=:id)
ERROR: ArgumentError: duplicate rows found in right table
```
