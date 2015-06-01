title: "Speedup Python"
date: 2015-05-31 19:35:01
tags:
---

Python is not known as a very fast language, but you have some options to speed up your programs.

<!-- more -->

这两天在看[Rust lang](http://www.rust-lang.org), 基本是因为首页的一句话所吸引：

> Rust is a systems programming language that runs blazingly fast, prevents nearly all segfaults, and guarantees thread safety.

其实包括之前看[Julia](http://julialang.org)和[Dlang](http://dlang.org/index.html)，都是被他们的效率所吸引。Python用久了，难免对速度有点盲目的向往。但是语言设计很多时候都是一些trade-off的问题，执行效率和生产效率往往是不可得兼的。但这并不意味着选择Python就一定意味着牺牲执行效率。这里简单列几个可选的方案。

1. [pypy](http://pypy.org): "PyPy is a fast, compliant alternative implementation of the Python language". 但如果代码中大量使用了numpy等使用Python C API的package，往往是无法兼容的。
2. [numba](http://numba.pydata.org): 依然借助于JIT，在CPython中就可以使用，相对简单，对Numpy有一定支持。
3. [Cython](http://cython.org): 用这家伙需要写"python风格的C代码"，Cython实际上会自动生成相应的C代码。
4. Python C API: 对于C中高手或者相对简单的环境，当然可以直接用ctypes。值得一提的是不一定只用C，像Rust能便宜成C兼容的类库，自然也可以通过Python C API使用。
5. [cffi](https://cffi.readthedocs.org/en/latest/)，[swig](http://www.swig.org)等。

对于以上方法来说，并不意味着有必要尽可能多的使用，甚至在某些环境下效率反而有所降低。简单使用的话建议numba，深入了解可以从Cython的文档开始。
