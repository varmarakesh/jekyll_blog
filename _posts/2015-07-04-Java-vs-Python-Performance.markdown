---
layout: post
title:  "Java vs Python Performance"
date:   2015-07-04 15:34:11
categories: python
description: Java vs Python Performance
comments: True
---
It is well known that Python programs tend to run slower compared to Java, C#.net, C++. This post attempts to demonstrate the fact. I have taken a classic CPU bound problem, generate nth prime number, implemented a python solution and java solution based on the same algorithm.

Java implementation source code is available [at this link] [1]

Python implementation source code is available [at this link] [2] 


Java Single Thread Test took 127 milli seconds.

Python Single Thread Test took 4.8 seconds.

The difference is quite stark, python performs 35 times worse. I've learnt that to perform performance benchmarking of different languages, need to try them on CPU bound problems. Think of CPU bound problems as one that have heavy looping, sorting, floating point operations and they have no IO (databases, file reads, network, etc). If you program contains IO operations, then it is not usually suitable for a good benchmark test because IO time will consume a significant execution time and IO execution time(database call or a web service call) is usually the same regardless of the python or java implementation.

Conclusion
----------
1. It is very hard to beat the complied implementations of these CPU bound, iterative numerical calculations.
2. Python most selling features that evolve around developer productivity. But this kind of a large scale iterative mathematical operations come with a cost. The python solution was run on the cPython, the default implementation of python. However, using cython, pypy or even numpy would offer significant performance benefits.
3. I have include a test for multi-threading also. Since python uses GIL (global interpretor lock), the multi-threaded implementation actually perform worse than the single threaded implementation. However, for Java it is not the case.

[1]: https://bitbucket.org/varmarakesh/fullstack/src/c20824f57249970c75bf457ba7aade7c42f4d3ac/java/threading_test/src/ThreadTest.java?at=master
[2]: https://bitbucket.org/varmarakesh/fullstack/src/c20824f57249970c75bf457ba7aade7c42f4d3ac/python/threading/testprime.py?at=master