---
layout: post
title:  "Python Global Interpreter Lock"
date:   2015-05-28 15:34:11
categories: python
comments: True
---
Python uses GIL (Global interpreter lock). GIL ensures that python process can only process 1 instruction at a time, regardless the number of cores that CPU is using. This has been the behavior of the default implementation of python, cPython.

However, this could be overcome by using alternative implementations such as cython and by using mutliprocessing module (instead of threading).

To understand the impact of GIL, I ran this program on a multi-core CPU laptop. This is a classic CPU bound problem, generate the nth prime. Think of CPU bound problem as ones that has heavy looping, floating point operations and no IO (databases, disk operations, network, etc).

The full source code is available [at this link] [link]
{% highlight python %}
import math
import threading, time
from functools import wraps

def timefn(fn):
    @wraps(fn)
    def measure_time(*args, **kwargs):
        t1 = time.time()
        result = fn(*args, **kwargs)
        t2 = time.time()
        print ("@timefn:" + fn.func_name + " took " + str(t2 - t1) + " seconds")
        return result
    return measure_time

def is_prime(num):
    for j in range(2,int(math.sqrt(num)+1)):
        if (num % j) == 0:
            return False
    return True

def prime(nth):
    i = 0
    num = 2
    while i < nth:
        if is_prime(num):
            i += 1
            if i == nth:
                print('The ' + str(nth) + 'th prime number is: ' + str(num))
        num += 1

@timefn
def singleThreadtest():
    for count in range(12):
        prime(10000)


@timefn
def multiThreadtest():
    threads = []
    for count in range(12):
        t = threading.Thread(target = prime, args=[10000])
        threads.append(t)
        t.start()

    for t in threads:
        t.join()

if __name__ == "__main__":
    singleThreadtest()
    multiThreadtest()
{% endhighlight %}

@timefn:singleThreadtest took 4.8399810791 seconds.

@timefn:multiThreadtest took 5.75388002396 seconds.

Although, generally speaking, it is a bit suprising to hear that mutli-threaded version performs slower than the single-threaded version, however in python GIL world, it should not be much of a surprise. Because, only single core of CPU is used and only 1 instruction is executed at a given time. So, implementing threading in this case creates overhead of managing threads and introduces waits for shared resources.

#### Conclusion

1. If your program is a CPU bound, think twice about implementing threading. At the least, experiment using cython or multi-processing modules.
2. If your program is IO-bound, then multi-threading will generate real speeds. You can go thru my post title "Oracle AWR Using Python". I've tested multi-threading there and performance benefits are exponential.


[link]: https://bitbucket.org/varmarakesh/fullstack/src/0c08ed6ea9e8a7c02ba702775725f60e37dddde4/python/threading/?at=master

