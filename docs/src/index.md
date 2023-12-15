# DataFrames.jl

欢迎阅读DataFrames.jl的文档！

本资源旨在教授你如何使用DataFrames.jl包进行表格数据操作所需的一切知识。

你可以查看以下资源来了解DataFrames.jl的更多应用，尤其是与其他包的结合使用（这些资源会与DataFrames.jl的发布版本保持同步更新）：
* 在*Journal of Statistical Software*发表的文章[DataFrames.jl: Flexible and Fast Tabular Data in Julia](https://www.jstatsoft.org/article/view/v107i04)
* [Data Wrangling with DataFrames.jl Cheat Sheet](https://www.ahsmart.com/pub/data-wrangling-with-data-frames-jl-cheat-sheet/)
* [使用Jupyter Notebooks的DataFrames教程](https://github.com/bkamins/Julia-DataFrames-Tutorial/)
* [Julia Academy DataFrames.jl教程](https://github.com/JuliaAcademy/DataFrames)
* [JuliaCon 2019](https://github.com/bkamins/JuliaCon2019-DataFrames-Tutorial)、
  [JuliaCon 2020](https://github.com/bkamins/JuliaCon2020-DataFrames-Tutorial)、
  [JuliaCon 2021](https://github.com/bkamins/JuliaCon2021-DataFrames-Tutorial)、
  [JuliaCon 2022](https://github.com/bkamins/JuliaCon2022-DataFrames-Tutorial)、
  [PyData Global 2020](https://github.com/bkamins/PyDataGlobal2020)以及
  [ODSC Europe 2021](https://github.com/bkamins/ODSC-EUROPE-2021)的教程
* [DataFrames.jl展示](https://github.com/bkamins/DataFrames-Showcase)

如果你更喜欢通过书籍学习DataFrames.jl，可以考虑阅读：
* [Julia for Data Analysis](https://github.com/bkamins/JuliaForDataAnalysis)
* [Julia Data Science](https://juliadatascience.io/)

## 什么是DataFrames.jl？

DataFrames.jl提供了一套用于在Julia中处理表格数据的工具。其设计和功能与Python的[pandas](https://pandas.pydata.org/)和R的`data.frame`、[`data.table`](https://rdatatable.gitlab.io/data.table/)以及[dplyr](https://dplyr.tidyverse.org/)相似，使其成为一个极好的通用数据科学工具。

DataFrames.jl在Julia数据生态系统中起着核心作用，并且与许多不同的库紧密集成。DataFrames.jl并不是Julia中处理表格数据的唯一工具——如下所述，对于某些特定用例，还有一些其他的优秀库——但是它通过熟悉的接口提供了出色的数据整理功能。

要更详细地了解这个工具链，请查看本手册中的教程。新用户可以从[使用DataFrames.jl的第一步](@ref)部分开始。

你可能会发现[DataFramesMeta.jl](https://juliadata.github.io/DataFramesMeta.jl/stable/)包或者本手册中讨论的[数据操作框架](@ref)部分的其他便利包在编写更高级的数据转换时很有帮助，尤其是如果你没有太多的编程经验。这些包提供了类似于R中[dplyr](https://dplyr.tidyverse.org/)的便利语法。

如果你在使用DataFrames.jl时使用元数据，你可能会发现[TableMetadataTools.jl](https://github.com/JuliaData/TableMetadataTools.jl)包很有用。此包定义了几个便利函数，用于执行典型的元数据操作。

## DataFrames.jl和Julia数据生态系统

对于新用户来说，Julia数据生态系统可能是一个难以导航的领域，部分原因是Julia生态系统比其他一些语言更倾向于将功能分布在不同的库中。由于许多来到DataFrames.jl的人刚开始探索Julia数据生态系统，以下是一些提供不同数据科学工具的良好支持的库，以及每个库的特点，以及它们与DataFrames.jl的集成程度的一些注解。

- **统计**
    - [StatsKit.jl](https://github.com/JuliaStats/StatsKit.jl)：一个便利的元包，加载一套用于统计的基本包，包括本节下面提到的那些包和DataFrames.jl本身。
    - [Statistics](https://docs.julialang.org/en/v1/stdlib/Statistics/)：Julia标准库提供了广泛的统计功能，但要访问这些函数，你必须调用`using Statistics`。
    - [LinearAlgebra](https://docs.julialang.org/en/v1/stdlib/LinearAlgebra/)：与`Statistics`一样，许多线性代数特性（分解、求逆等）都存在于你必须加载才能使用的库中。
    - [SparseArrays](https://docs.julialang.org/en/v1/stdlib/SparseArrays/)也在标准库中，但必须加载才能使用。
    - [FreqTables.jl](https://github.com/nalimilan/FreqTables.jl)：创建频率表/交叉制表。与DataFrames.jl紧密集成。
    - [HypothesisTests.jl](https://juliastats.org/HypothesisTests.jl/stable/)：一系列假设检验工具。
    - [GLM.jl](https://juliastats.org/GLM.jl/stable/manual/)：用于估计线性和广义线性模型的工具。与DataFrames.jl紧密集成。
    - [StatsModels.jl](https://juliastats.org/StatsModels.jl/stable/)：用于将异质的`DataFrame`转换为用于线性代数库或不直接支持`DataFrame`的机器学习应用的同质矩阵。会做如将分类变量转换为指标/一键热编码，创建交互项等事情。
    - [MultivariateStats.jl](https://multivariatestatsjl.readthedocs.io/en/stable/index.html)：线性回归、岭回归、PCA、组分分析工具。与DataFrames.jl的集成不够紧密，但可以很容易地与`StatsModels`一起使用。
- **机器学习**
    - [MLJ.jl](https://github.com/alan-turing-institute/MLJ.jl)：如果你更多的是一个应用用户，那么有一个单独的包可以从所有这些不同的库中抽取，并提供一个类似scikit-learn的API：MLJ.jl。MLJ.jl为大量的机器学习算法提供了一个公共接口。
    - [ScikitLearn.jl](https://cstjean.github.io/ScikitLearn.jl/stable/)：围绕Python的完整scikit-learn机器学习库的Julia包装器。与DataFrames.jl的集成不够紧密，但可以通过StatsModels.jl进行结合。
    - [AutoMLPipeline](https://github.com/IBM/AutoMLPipeline.jl)：一个包，使得使用简单表达式创建复杂的ML管道结构变得非常简单。它利用Julia的内置宏编程特性，符号处理，操纵管道表达式，并使得发现机器学习回归和分类的最优结构变得容易。
    - 深度学习：[KNet.jl](https://denizyuret.github.io/Knet.jl/stable/tutorial/#Introduction-to-Knet-1)和[Flux.jl](https://github.com/FluxML/Flux.jl)。
- **绘图**
    - [Plots.jl](http://docs.juliaplots.org/latest/)：强大的、现代的绘图库，语法类似于Python的[matplotlib](https://matplotlib.org/)或R的`plot`。[StatsPlots.jl](http://docs.juliaplots.org/latest/tutorial/#Using-Plot-Recipes-1)为Plots.jl提供了许多标准统计图的配方。
    - [Gadfly.jl](http://gadflyjl.org/stable/)：高级绘图库，具有类似于R的[ggplot](https://ggplot2.tidyverse.org/reference/ggplot.html)的"图形语法"语法。
    - [AlgebraOfGraphics.jl](http://juliaplots.org/AlgebraOfGraphics.jl/stable/)：一个基于[Makie.jl](https://docs.makie.org/stable/)的"图形语法"库。
    - [VegaLite.jl](https://www.queryverse.org/VegaLite.jl/stable/)：高级绘图库，使用不同的"图形语法"语法，并强调交互式图形。
- **数据整理**：
    - [Impute.jl](https://github.com/invenia/Impute.jl)：处理向量、矩阵和表中缺失数据的各种方法。
    - [DataFramesMeta.jl](https://github.com/JuliaData/DataFramesMeta.jl)：为DataFrames.jl提供一系列便利函数，增强了`select`和`transform`，提供了类似于R中[dplyr](https://dplyr.tidyverse.org/)提供的用户体验。
    - [DataFrameMacros.jl](https://github.com/jkrumbiegel/DataFrameMacros.jl)：提供了类似于DataFramesMeta.jl的常见DataFrames.jl函数的宏版本，具有方便的语法，用于一次操作多个列。
    - [Query.jl](https://github.com/queryverse/Query.jl)：Query.jl为数据整理提供了一个单一的框架，可用于多种库，包括DataFrames.jl、其他表格数据库（下面会有更多信息），甚至非表格数据。提供了许多类似于R中dplyr或[LINQ](https://en.wikipedia.org/wiki/Language_Integrated_Query)的便利函数。
    - 在本手册的[数据操作框架](@ref)部分，你可以找到这些包的更多信息。
- **更多！**
    - [Graphs.jl](https://github.com/JuliaGraphs/Graphs.jl)：一个纯Julia的，高性能的网络分析库。可以使用[GraphDataFrameBridge.jl](https://github.com/JuliaGraphs/GraphDataFrameBridge.jl)包将`DataFrame`中的边列表轻松转换为图。
- **IO**：
    - DataFrames.jl可以很好地处理各种格式，包括：
        - CSV文件（使用[CSV.jl](https://github.com/JuliaData/CSV.jl)），
        - Apache Arrow（使用[Arrow.jl](https://github.com/JuliaData/Arrow.jl)）
        - 读取Stata、SAS和SPSS文件（使用[ReadStatTables.jl](https://github.com/junyuan-chen/ReadStatTables.jl)；另外[Queryverse](https://www.queryverse.org/)用户可以选择[StatFiles.jl](https://github.com/queryverse/StatFiles.jl)），
        - Parquet文件
          (使用[Parquet2.jl](https://gitlab.com/ExpandingMan/Parquet2.jl))，
        - 读取R数据文件(.rda, .RData) (使用[RData.jl](https://github.com/JuliaData/RData.jl))。

虽然并非所有这些库都与DataFrames.jl紧密集成，但由于`DataFrame`本质上是对齐的Julia向量的集合，所以很容易(a)从非DataFrames集成的库中提取一个向量，或者(b)使用`Matrix`构造函数或StatsModels.jl将你的表格转换为同质类型的矩阵。

### 其他Julia表格库

DataFrames.jl是一个用于数据操作和整理的很好的通用工具，但并不是所有应用的理想选择。对于有更专业需求的用户，可以考虑使用：

- [TypedTables.jl](https://juliadata.github.io/TypedTables.jl/stable/)：类型稳定的异质表格。当你的表格结构相对稳定并且没有数千个列时，用于提高性能。
- [JuliaDB.jl](https://juliadata.github.io/JuliaDB.jl/stable/)：对于处理无法放入内存的数据的用户，我们建议使用JuliaDB.jl，它为大型数据集提供了更好的性能，并可以处理超出内存的数据操作（Python用户可以将JuliaDB.jl视为Julia版本的[dask](https://dask.org/)）。

请注意，Julia生态系统中的大多数表格数据库（包括DataFrames.jl）都支持一个公共接口（在[Tables.jl](https://github.com/JuliaData/Tables.jl)包中定义）。因此，一些库能够处理各种表格数据结构，使得随着你的需求变化，从一个表格库轻松切换到另一个表格库成为可能。例如，[Query.jl](https://github.com/queryverse/Query.jl)的用户可以使用同一段代码来操作`DataFrame`、`Table`（由TypedTables.jl定义）或JuliaDB表。

## 有问题吗？

如果有你期望DataFrames能做到的事情，但你不知道如何做，请在[Discourse](https://discourse.julialang.org/new-topic?title=[DataFrames%20Question]:%20&body=%23%20Question:%0A%0A%23%20Dataset%20(if%20applicable):%0A%0A%23%20Minimal%20Working%20Example%20(if%20applicable):%0A&category=Domains/Data&tags=question)的Domains/Data中提出问题。此外，你可能还想在[JuliaAcademy](https://juliaacademy.com/p/introduction-to-dataframes-jl)听一下对DataFrames.jl的介绍。

请通过[开启一个问题](https://github.com/JuliaData/DataFrames.jl/issues/new)来报告错误。

你可以在整个文档中跟随**源**链接，直接跳到GitHub上的源文件，以改进文档和函数功能的pull请求。

在提交你的第一个PR之前，请查看[DataFrames的贡献指南](https://github.com/JuliaData/DataFrames.jl/blob/main/CONTRIBUTING.md)！

可以在[发布页面](https://github.com/JuliaData/DataFrames.jl/releases)找到关于特定版本的信息。

## 包手册

```@contents
Pages = ["man/basics.md",
         "man/getting_started.md",
         "man/joins.md",
         "man/split_apply_combine.md",
         "man/reshaping_and_pivoting.md",
         "man/sorting.md",
         "man/categorical.md",
         "man/missing.md",
         "man/comparisons.md",
         "man/querying_frameworks.md"]
Depth = 2
```

## API

只有导出的（即在加载DataFrames.jl包并使用 `using DataFrames` 后，无需使用 `DataFrames.` 限定符即可使用）类型和函数被视为DataFrames.jl包的公共API的一部分。通常，所有这些对象都在此手册中有文档记录（如果有缺少的文档，请在[这里](https://github.com/JuliaData/DataFrames.jl/issues/new)报告问题）。

!!! note

    在可能的情况下，DataFrames.jl避免对公共和记录的API进行破坏性更改。

    下列更改不被视为破坏性更改：

    * 由操作计算的特定浮点值可能随时更改；用户应仅依赖近似的准确性；
    * 在使用Base Julia提供的默认随机数生成器的函数中，计算出的特定随机数可能会在Julia版本之间更改；
    * 如果更改的功能被分类为错误；
    * 如果更改的行为未被记录；两个主要情况是：
      1. 在实现中，某些函数接受的参数范围比记录处理的范围更广 - 更改处理未记录参数的方式不被视为破坏性；
      2. 函数返回的值的类型发生了更改，但仍然遵循文档中指定的契约；例如，如果一个函数记录为返回一个向量，那么将其类型从 `Vector` 更改为 `PooledVector` 不被视为破坏性；
    * 错误行为：抛出异常的代码可以更改抛出的异常类型或停止抛出异常；
    * 显示的更改（如何打印对象）；
    * 更改Base Julia的全局对象的状态，这些对象的状态通常被视为不稳定（例如，全局随机数生成器的状态）。

    所有公共API的部分的类型和函数在进行对它们的破坏性更改或移除它们之前，都保证会经历一个弃用期。

    标准做法是，当DataFrames.jl的一个主要版本发布时（例如，在1.x版本中弃用的功能将在2.0版本中进行更改）。

    在罕见的情况下，可能会在一个小版本中引入破坏性更改。在这种情况下，更改的行为仍然会经历一个小版本的弃用期。可能允许进行这种破坏性更改的情况是（如果可能，仍将避免这种破坏性更改）：

    * 受影响的功能以前在文档中明确标识为可能会更改（例如，在DataFrames.jl 1.4版本发布中，`:note`-style元数据的传播规则就被记录为这样）；
    * 更改处于被分类为错误的边缘（在罕见的情况下，即使某个函数的行为被记录，其对某些参数组合的后果也可能被决定为无意的和不希望的）；
    * 更改是为了调整DataFrames.jl的功能以适应Base Julia的更改。

请注意，虽然Julia允许您访问DataFrames.jl的内部函数或类型，但这些可能在DataFrames.jl的版本之间无预警地更改。特别是，直接访问DataFrames.jl包的公共API部分的类型的字段是不安全的，例如使用 `getfield` 函数。当考虑允许对定义类型的字段进行某些操作时，应该使用适当的导出函数代替。

```@contents
Pages = ["lib/types.md", "lib/functions.md", "lib/indexing.md"]
Depth = 2
```

## 索引

```@index
Pages = ["lib/types.md", "lib/functions.md"]
```
