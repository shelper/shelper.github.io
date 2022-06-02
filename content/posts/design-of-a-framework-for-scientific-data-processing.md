---
title: design of a framework for scientific data processing
date: 2016-05-29
tags: ["python", "javascript", "research"]
categories: ["programming"]
link:
description:
---

I have beening thinking of building such a framework for a while, I started a little project called [pypeline](http://github.com/shelper/pypeline). The code on github is not up-to-date.
And I have some new thoughts on the framework design. So this article serves as a design documents
and welcomes any comments or suggestions.

Based on my own experience, I often want to process some data or image at multiple stage.
Often, I want to change some parameters of certian algorithm to see how the outcomes are changed.
It becomes cumbersome and difficult to check after saving a lot of intermedia data, with file names like
`para1_value1_para1_value2...`.

I then thought about it, and decide to write a framework like a pipeline. for a multi-step process,
I can connect the processing function of each step and form a pipeline. I decided to use async module
in python and if needed, use a generate as the input, so if I want to check the outcome of a multi-step process,
I can simply write a generate to feed in the input repeatedly, and with the async feature,
I can change the parameters in realtime, and the outcome will change instantaneously or even real time as well.

then the next thing is to make this pipeline easier to use, e.g.,
I used decorator to claim funtions that can be inserted into the pipeline by a single line of code.
I made a pipeline class with implementations of `pop()` and `insert()` for adding and removing functions in the pipeline
I also plan to add input and output check for each functions, so that when I connect two functions,
the framework will check if the output of the first matches the input of the second. I am thinking of
using a separate file like the header file in `c/c++` to describe the input and output of each function.

for instance, I have two functions to connect:

```python
# functions.py
def f1(x, y):
    return x + y, x - y

def f2(a, b):
    return a + b
```

and `x, y, a, b` are all integers,

so I can have a separate file with same name but different extension:

```
# functions.dsc
f1 = {'input': [int, int], 'output': [int, int]}
f2 = {'input': [int, int], 'output': [int]}
```

then when connect these two functions I can do:

```
# pseudo code:
if f1['output'] == f2['input']:
    connect f1 and f2
else:
    raise FuncInputOutputMismatch()
```

for more complicated cases, the pipeline might have branches and inter-connections within branches.
it of course should be one way data flow. it further reminds me the frontend library from facebook that uses one way data flow:
the [React](https://facebook.github.io/react/). There is also a [video](https://www.youtube.com/watch?v=xsSnOQynTHs) on this.

then I did some research and found something called [`hot loading`](https://github.com/gaearon/react-hot-loader)
it also reminds me the [talk](https://www.youtube.com/watch?v=PUv66718DII) given by Bret Victor.
it would be such a fantastic tool if some tool like this can be used for scientific data processing...

So I just wonder if there is already something similar or at least some that i can borrow code/ideas from?
or is this kind of design idea new? i tend to think it is not, I would like to read more to get a better understanding of
such design before i go ahead coding the framework.
