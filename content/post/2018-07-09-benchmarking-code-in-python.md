---
title: Benchmarking Code in Python
author: ~
date: '2018-07-09'
slug: benchmarking-code-in-python
categories: []
tags: [python, benchmarking, pandas, numpy, ufunc, datascience]
---

When I first started programming I started to get the hang of `for-loops` and I basically applied them to everything that was iterable. Upon learning more about efficient coding for data science, I learned about vectorization (mostly through R and pySpark), so I wanted to showcase a simple case study as to when vectorization reigns supreme over `for-loops`.  

<!--more-->

Although I will state that `for-loops` do allow the user to be a little more intimate with the function call and I do encourage everyone to become efficient at loops :)

# Load Packages
First we will load the appropriate packages, in this case we will be using `numpy`, `matplotlib` and `timeit`. I will provide a `requirements.txt` file for people utilizing `virtualenv` but if you're using `anaconda` the packages should already come pre-installed. 

+ [numpy](http://www.numpy.org/) - used to showcase the standard library for data enthusiast in python
+ [matplotlib](https://matplotlib.org/) - plotting library to showcase differences in methods
+ [timeit](https://docs.python.org/3/library/timeit.html) - package we will use to showcase difference in loops and vectorization through benchmarking 

```python
import numpy as np
import timeit
import matplotlib.pyplot as plt
# For ggplot2 style plots
plt.style.use('ggplot')

# Set random seed for reproducibility
np.random.seed(42)
```
Notice how I set the random seed here, this will be important in order for this walkthrough to be reproducibile. Although experimenting with different random seeds might produce different results!


# Define Looping Function

First I copy-pasted the function David created so that I can use it in this script. It is a simple function that produces the summation of an array. This is the way many people are taught when first learning python and programming. 

```python
def vector_sum(vector):
    # takes vector and returns the sum of its elements
    s = 0
    for i in vector:
        s = s + i
    return s
```

# Define vectorization

The operation that we are going to be vectorization doesn't need to be defined as a function since it utilizes two `ufunc` functions from `numpy`. 

## ufunc functions

This is an important concept especially when utilizing `numpy` for data analysis (so all the time!). But many of these built-in operations are vectorized to perform fast and efficient processes. 

You can read more [here](https://docs.scipy.org/doc/numpy/reference/ufuncs.html).

But if I were to give a TLDR; if you need to do an operation on data that doesn't require too many conditional statements use vectorized operations. 

The operation is written as such:

```python
np.add.reduce(vector)
```

**NOTE**: Note upon further reading if the array has dimensions WxL where W is greater than 1, than `np.sum()` is a more appropriate function to use for our desired outcome. 

# Check to see if both operations are equal

I'm going to create an array of length 100 with random values, and show that the `vector_sum` and `numpy` `ufunc` operations produce the same result. 

```python
my_array = np.random.rand(100)
```

Let's check the array size first, this will be important to understand in a later section discussing benchmarking. 

```python
len(my_array)
```


    100

Here I'll be utilizing the function on `my_array` and printing out the output with 5 significant values. 

```python
print('{0: .5f}'.format(vector_sum(my_array)))
```

     47.01807
     
     
Dope! Now I will use the `ufunc` operations and hopefully the results will be the same. 

```python
print('{0: .5f}'.format(np.add.reduce(my_array)))
```


     47.01807
     
As we can see we got the same results! 

The important aspect of this blog post is the time it takes to produce our results. For a data set with 100 values optimizing will most likely not be important, but when you start playing with larger data sets this will be important to produce fast and succinct data analysis. 


# Benchmark with timeit

Now I will be using the `timeit` function where I will include the two operations as strings and include the parameter `globals = globals()`. This will recognize my current python environment (otherwise I would have to recreate everything within the function call which would be pretty tedious). 

I will tell it the number of iterations to do (in this case we will do 1000). 

```python
print(
    # Include function call in timeit
    timeit.timeit('vector_sum(my_array)',
    # Include global environment 
    globals = globals(), 
    # Number of iterations
    number = 1000))
```

    0.019325109002238605
    
Now the `ufunc` operation:

```python
print(
    timeit.timeit('np.add.reduce(my_array)', 
    globals = globals(), 
    number = 1000))
```

    0.004530509999312926
    

# Bootstrapping Benchmarking Methods

Its already pretty obvious that the `numpy` operations performs significantly better than the `vector_sum` function, but to drive the point home I will do a loop that will do these operations on randomly generated arrays. The arrays will vary in size starting with an array of size 100 and ending with an array of size 10000. 

Then I will input the indices and the time it took to execute the operations into a dictionary which I will then plot to see the difference in time for the varying arrays. 

Note that the arrays generated at each loop are different since the random generator is being using on each operation seperately. In future iterations I would like to have the same arrays to give a more concrete comparison. 


```python
vector_dict = {'vectorized': [], 
              'for_loop': [],
              'index': []}
```
Ironically I will be using a `for-loop` to append each respective key in the dictionary. Since they are being appended in a list inside a dictionary, the order will be kept. When iterating over a dictionary be careful because it will output in random order (Until Python 3.7 from what I've heard).

```python
for i in range(100, 10000, 200):
    # Create index 
    vector_dict['index'].append(i)
    # Create the array that will contain all time values for numpy operations
    vector_dict['vectorized'].append(
      timeit.timeit(
        'np.add.reduce(np.random.rand(i))'.format(i), 
        globals = globals(), 
        number = 1000))
    # Create the array that will contain all time values for `for-loop` function
    vector_dict['for_loop'].append(
      timeit.timeit(
        'vector_sum(np.random.rand(i))'.format(i), 
        globals = globals(), 
        number = 1000))
```
So each key will have the values in a list from the first value (100) to the last (10000) in increments of 200 so 50 values overall. 


# Plotting 

Next we will visualize the difference by ploting our index as our x-axis and y-axis containing the time it took to calculate that n-dimension array. 

```python
fig, ax = plt.subplots()

ax.set_facecolor("#fafafa")
plt.plot(vector_dict['index'], vector_dict['vectorized'], 'r')
plt.plot(vector_dict['index'], vector_dict['for_loop'], 'g')
plt.legend(('Vectorized Operation', 'For Loop'))
plt.xlabel('Length of Randomly Created Arrays')
plt.ylabel('Time for operation (1000 iterations for each array)')
plt.show()
```

![](https://i.imgur.com/0sXn7AJ.png)

Now we can notice the huge difference in operations and the time it took to calculate them over different sized arrays. That cray-cray!

# Conclusions

I wanted to create this blog post because I'll often see beginners being taught a certain way of writing code, and its important to learn these methods. But we have to remember that there are other methods and operations that will significantly improve the run time of your code. 

The benefits of vectorization are evident when utilizing tools like R and pySpark, and might not be as clear when learning python. For most of my python experience learning data science, I didn't read too much documentation going over vectorization although it did feel like it was implied. 

So for a beginner like myself being able to see the differences in performance of methodologies can help reduce running time, but also understanding of more succinct data science practices.  

If you have any questions, suggestions or corrections reach out to me!