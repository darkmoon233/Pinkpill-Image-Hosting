# webAssemblyPerformanceAnalysis

## 1. `benchmark` 概念

>In computing, a benchmark is the act of running a computer program, a set of programs, or other operations, in order to assess the relative performance of an object, normally by running a number of standard tests and trials against it. The term benchmark is also commonly utilized for the purposes of elaborately designed benchmarking programs themselves.  
Benchmarking is usually associated with assessing performance characteristics of computer hardware, for example, the floating point operation performance of a CPU, but there are circumstances when the technique is also applicable to software. Software benchmarks are, for example, run against compilers or database management systems (DBMS).  
Benchmarks provide a method of comparing the performance of various subsystems across different chip/system architectures.  
-- from `wikipedia`

在计算中,基准测试是运行一个计算机程序,一组计算机程序或其他操作的行为,以便评估对象的相对性能,通常是通过对其运行许多标准测试和试验.基准这一术语也常用于精心设计的基准程序本身.基准测试通常与评估计算机硬件的性能特征相关联,例如,CPU的浮点运算性能,但是存在该技术也适用于软件的情况.例如,软件基准测试针对编译器或数据库管理系统（DBMS）运行.基准测试提供了一种比较不同芯片/系统架构中各种子系统性能的方法.

## 2. `benchmark`测试目的

通过对wasm的性能与c++、Python等的性能对比,判断其是否能够在性能上对比其他脚本语言存在优势,从而替代一部分脚本语言的功能.

## 3. `benchmark` 测试

### 3.1 `Emscripten` 内置的 Benchmark

Emscripten拥有全面的测试套件,几乎涵盖了所有Emscripten功能.这些测试对于开发人员来说是一个很好的资源,因为它们提供了大多数功能的实际示例,并且已知传递主分支（并且几乎总是在传入分支上）.除了正确性测试之外,您还可以运行基准测试.

Emscripten提供了一套测试套件,其中涵盖了各方面的测试,尤其是包含一整套benchmark测试,但是这个测试套件是完全封装的,很难进行拆解和重新处理.

#### 3.1.2 测试环境

    操作系统：Windows10 企业版

    内存：16 GB

    CPU：Intel core i7-8700 3.20GHZ

    Emscripten版本：1.38.39

    Node.js版本：v10.16.0

    clang版本：6.0.1

#### 3.1.3 基于nodejs的wasm benchmark

Emscripten允许使用不同的JS引擎进行wasm的运行,这里选用了基于V8的nodejs（选用nodejs是因为他在使用时非常的方便）.V8引擎完全支持webAssembly.

在本次测试中,共有16项测试用例,每一项用例运行了5次,取其平均值作为统计分析数据.

下图为基于nodejs的wasm benchmark数据.

![wasm](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/EmscriptenWasmBenchmark.png)

#### 3.1.4 基于clang的c++ benchmark

为了与wasm的性能进行对比,选用了对应benchmark用例的c++版本,使用clang进行编译运行.

在本次测试中,共有16项测试用例,每一项用例运行了5次,取其平均值作为统计分析数据.

下图为基于clang的c++ benchmark数据.

![c++](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/EmscriptenC++Benchmark.png)

#### 3.1.5 Emscripten测试套件下的wasm性能分析

单纯的运行时长不能反映运行效率,现将以每个用例中的c++运行时间为基准1,衡量对应的wasm的运行时间.对比数据下：

![rate](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/EmscriptenBenchmarkRate.png)

该数据表明(运行时间一定程度上能够代表性能）,c++的性能是wasm的50-200倍左右.这样的数据与已知情况下的wasm性能不符,同时Emscripten的官方文档关于benchmark部分的描述非常简单,并没有详细阐述其中的参数设置与结果分析,导致这其中的不合理性难以解释.因此需进行进一步的benchmark测试来分析对应的wasm性能.

### 3.2 自定义benchmark

#### 3.2.2 测试设计

采用c++,wasm,Python和Lua四类进行相同的用例计算,统计平均耗时进行分析.

wasm的生成与运行有两类方式：

- 通过Emscripten套件生成.wasm与对应的.js文件,通过js文件引入wasm文件.
- 第二种是通过clang将c++编译为wasm,再通过其他的编译器后端进行wasm的直接运行（不适用js）.

第二类方法相比于第一类方法有着更好的灵活性,可以自由选择不同wasm runtime,但是通过尝试[intel/wasm-micro-runtime]发现([www://asfda](https://github.com/intel/wasm-micro-runtime))(相比于其他几项runtime,该项目有着第二多的star和最新的更新时间),wasm-micro-runtime对于复杂的c++程序生成的wasm文件不能成功运行,因此只采用了第一种方式进行wasm的性能测试.

每个测试用例在相同环境下运行5次,以平均值（保留三位小数）作为性能评估的依据。

#### 3.2.2 测试环境

    操作系统：Ubuntu 18.042 LTS

    内存：7.8 GB

    CPU：Intel Core i7-8700 CPU 3.20GHz 

    Emscripten版本：1.38.31

    Node.js版本：v8.10.0 

    clang版本：6.0.0-1ubuntu2

    Lua版本: 5.3.3

    Python版本: 3.6.8

#### 3.3.3 测试样例

综合考虑了[benchmark game](https://benchmarksgame-team.pages.debian.net/benchmarksgame/index.html) 和 [scriptorium](https://github.com/r-lyeh-archived/scriptorium)两个项目,设置了Mandelbrot, N-body, Spectralnorm, Fasta, Fankuchen, Binary-trees, Fibonacci 七项测试用例.

1. mandelbrot
  
   mandelbrot分形：对于复平面上的点C,令
   - Z0=C;
   - Z1=Z0^2+C;
   - Z2=Z1^2+C; ......

   本用例要求：
   - 在N-by-N位图上绘制Mandelbrot集[-1.5-i,0.5 + i].以便携式位图格式逐字节写入输出.
   - 计算到第1600项.
  
   mandelbrot图形的样子为![mandelbrot](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/mandelbrot.jpg)

2. n-body

   在物理学中,n体问题是预测一组天体在重力作用下相互作用的个体运动的问题.经典的物理问题可以非正式地陈述如下：鉴于一组天体的准稳定轨道特性（瞬时位置,速度和时间）,预测它们的相互作用力; 因此,预测他们未来所有的真实轨道运动.

   本用例要求：
   - 使用相同的简单辛积分器模拟木星行星的轨道.
   - 天体数量为1000000.

3. spectral-norm

   谱范数是矩阵的最大奇异值.

   本用例要求：
   - 计算无限矩阵A的谱范数,条目a[11] = 1,a[12] = 1/2,a[21] = 1/3,a[13] = 1/4,a[22] = 1/5,a[31] = 1/6等.
   - 矩阵规模为1000.

4. fasta

   FASTA格式是一种用于记录核酸序列或肽序列的文本格式,其中的核酸或氨基酸均以单个字母编码呈现.

   本用例要求：
   - 通过从给定序列复制产生DNA序列
   - 通过2个字母表的加权随机选择产生DNA序列
      - 将选择每个核苷酸的预期概率转换为累积概率
      - 匹配一个随机数与那些累积概率来选择每个核苷酸（使用线性搜索或二分搜索）
      - 每次需要选择核苷酸时,使用给定的朴素的线性同余生成器来计算随机数（不要缓存随机数序列）
   - 序列长度为2500000. 

5. fannkuch-redux

   本用例要求：
   - 采用{1,...,n}的排列,例如：{4,2,1,5,3}.
   - 取第一个元素,这里是4,并颠倒前4个元素的顺序：{5,1,2,4,3}.
   - 重复此操作直到第一个元素为1,因此翻转不会再改变：{3,4,2,1,5},{2,4,3,1,5}{4,2,3,1,5},{1,3,2,4,5}.计算翻转次数,这里5.
   - 为所有n!进行这个排列,并记录任何排列所需的最大翻转次数
   - 序列长度为10.

6. binary-trees

   本用例要求：
   - 构建深度为N的满二叉树
   - 自行定义树节点和树的类
   - 分配树节点,遍历树并统计树节点数目,以及删除节点
   - 二叉树深度为18.

7. Fibonacci

    本用例要求：
    - 使用递归进行计算fibonacci数列的第25项.
    - 计算300次.

#### 3.3.4 benchmark测试结果（以WASM耗时为基准）

##### 3.3.4.1 mandelbrot

|Rank|Language|Time|Relative WASM speed|
|---:|:-------|---:|:----------------:|
|1|C++| 0.073 s.|![x4.12](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/x4.12.svg)|
|2|WASM| 0.299 s.|![100](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/100.svg)|
|3|Lua| 7.613 s.|![4](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/4.svg)|
|4|Python| 9.099 s.|![3](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/3.svg)|

##### 3.3.4.2 n-body

|Rank|Language|Time|Relative WASM speed|
|---:|:-------|---:|:----------------:|
|1|C++| 0.068 s.|![x5.08](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/x5.08.svg)|
|2|WASM| 0.344 s.|![100](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/100.svg)|
|3|Lua| 6.617 s.|![5](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/5.svg)|
|4|Python| 23.871 s.|![1](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/1.svg)|

##### 3.3.4.3 spectral-norm

|Rank|Language|Time|Relative WASM speed|
|---:|:-------|---:|:----------------:|
|1|C++| 0.048 s.|![x8.21](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/x8.21.svg)|
|2|WASM| 0.396 s.|![100](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/100.svg)|
|3|Lua| 7.608 s.|![5](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/5.svg)|
|4|Python| 28.490 s.|![1](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/1.svg)|

##### 3.3.4.4 fasta

|Rank|Language|Time|Relative WASM speed|
|---:|:-------|---:|:----------------:|
|1|C++| 0.984 s.|![x21.02](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/x21.02.svg)|
|2|Lua| 5.705 s.|![x3.62](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/x3.62.svg)|
|3|Python| 16.238 s.|![x1.27](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/x1.27.svg)|
|4|WASM| 20.689 s.|![100](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/100.svg)|

##### 3.3.4.5 fannkuch-redux

|Rank|Language|Time|Relative WASM speed|
|---:|:-------|---:|:----------------:|
|1|C++| 0.150 s.|![x7.54](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/x7.54.svg)|
|2|WASM| 1.129 s.|![100](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/100.svg)|
|3|Lua| 8.060 s.|![14](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/14.svg)|
|4|Python| 13.428 s.|![8](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/8.svg)|

##### 3.3.4.6 binary-trees

|Rank|Language|Time|Relative WASM speed|
|---:|:-------|---:|:----------------:|
|1|WASM| 3.288 s.|![100](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/100.svg)|
|2|C++| 3.473 s.|![94](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/94.svg)|
|3|Python| 29.736 s.|![11](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/11.svg)|
|4|Lua| 33.647 s.|![10](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/10.svg)|

##### 3.3.4.7 Fibonacci

|Rank|Language|Time|Relative WASM speed|
|---:|:-------|---:|:----------------:|
|1|C++| 0.039 s.|![x5.28](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/x5.28.svg)|
|2|WASM| 0.204 s.|![100](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/100.svg)|
|3|Lua| 2.581 s.|![8](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/8.svg)|
|4|Python| 9.690 s.|![2](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkSvg/2.svg)|

#### 3.3.5 benchmark分析

1. 对于fasta测试,wasm的表现出现的明显的波动。需要指明的是,本用例有着的大量的打印到控制台操作(c++ fwrite_unlocked() 函数),wasm的打印时间和它所显示的计算时间是持平的,而其他的语言对于该测试的表现统计时间均小于用于打印的实际时间。为此,我将其中的输出部分进行了删除再进行测试,c++的平均耗时为0.399s,wasm的平均耗时为0.439s,c++耗时为WASM耗时的90.8%,基本符合其他的用例得出的结论,或许可以认为在打印方面,wasm的性能较差。为此我进行了一个简单的测试（输出一个特定字符1000000次）,c++的平均耗时为 1.862 s,wasm的平均耗时为 44.224 s,python的耗时为 19.624 s,可以看出在打印方面wasm的速度确实要慢一些。

2. 以上七项测试可以从一定程度上反映出各类语言的性能情况,所有数据汇总图

![total](https://github.com/darkmoon233/Pinkpill-Image-Hosting/blob/master/webAssemblyAnalysis/benchmarkResult.png)

由图可以得出以下的结论：

- 毫无疑问C++有着远超其他三类语言的性能（它仅在binary-trees中比wasm慢,其他用例下它的速度远超其他）。
- wasm对比C++,C++速度仅为wasm的数倍。
- wasm对比Python和Lua有着明显的性能优势。
- Python和Lua的性能相近。

## 参考

1. <https://emscripten.org/docs/getting_started/test-suite.html>
2. <https://en.wikipedia.org/wiki/Benchmark_(computing)>
3. <https://benchmarksgame-team.pages.debian.net/benchmarksgame/index.html>
4. <https://takahirox.github.io/WebAssembly-benchmark/>
5. <https://github.com/intel/wasm-micro-runtime>
6. <https://github.com/r-lyeh-archived/scriptorium>
