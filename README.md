# Counting Bloom Filter

**This is the file you may want to see: [Counting-Bloom-Filter-Overview-and-Introduction.ipynb](https://github.com/josealvarez97/Counting-Bloom-Filter/tree/master/Introduction%20and%20Implementation%20to%20a%20Counting%20Bloom%20Filter)**


But if you really, really, really don't want to open that beautiful file, here are some excerpts of what is in there (although I *highly recommend* taking a look at it, that's just better).

## Implementation of Counting Bloom Filter (Excerpt)

```python
# Import the necessary libraries
import random
import string
import math


# IMPLEMENTATION OF COUNTING BLOOM FILTER    
    
# Empty hash table generator
def empty_hash_table(N):
    return [0 for n in range(N)]

# BLOOM FILTER GENERATOR
# p stands for false positive rate
# n stands for the number of items to store.
def empty_bloom_filter(p,n):
    
    
    # Calculate the corresponding memory size m
    m = - math.floor((n * math.log(p))/(math.log(2)**2))
    
    # Create the bloom filter based on this parameters
    bfilter_table = empty_hash_table(m)
    
    return bfilter_table
   
# Returns the required number of hash functions k based
# a given false positive ratio p
def required_k(p):
    # Calculate the required number of hash functions
    # for the desired p.
    k = (-1)* round(math.log(p,2))
    
    return k 

# Inserts an item bloom filter an item.
def add_to_bloom_filter(hash_table, item, hash_functions):
    N = len(hash_table)
    
    for i in range(len(hash_functions)):
        index = hash_functions[i](item) % N    
        hash_table[index] += 1
    
    return hash_table


def query(hash_table, item, hash_functions):
    N = len(hash_table)
    
    for i in range(len(hash_functions)):
        index = hash_functions[i](item) % N  
        
        if hash_table[index] == 0:
            return False
        
    return True
    
    # return true if the item has already been stored in the hash_table


def remove(hash_table, item, hash_functions):
    if not query(hash_table, item, hash_functions):
        raise ValueError()

    N = len(hash_table)
    
    for i in range(len(hash_functions)):
        index = hash_functions[i](item) % N
        if hash_table[index] != 0:
            hash_table[index] -= 1
    
    return hash_table


# IMPLEMENTATION OF HASH FUNCTIONS

# Produces a hash function based on a template by considering a template as a parameter
def make_hash_function(offset_size):
    
    # HASH FUNCTION TEMPLATE
    def hash_function(string):
        # If we need an offset for the input, we add it.
        # As we're dealing with strings, we'll just add "*"
        # as many times as necessary.
        for i in range(offset_size):
            string += "*"

        # Compute the hash code
        ans = 0
        for i in range(len(string)):
            ans = 31*ans + ord(string[i])
            
        return ans # Inpired in Hurner's Rule
        
    return hash_function

# Generates a list of "different" hash functions
def hash_functions_generator(k):
    
    hashes_list = []
    
    for offset_size_i in range(0,k):
        hash_func = make_hash_function(offset_size_i)
        hashes_list.append(hash_func)
        
    return hashes_list


# HELPER FUNCTIONS / UTILITIES

# Random Words Generator
def randomword(length):
    return ''.join(random.choice(string.ascii_lowercase) for i in range(length))

# List of random words generator
def randomwords_list(list_length, word_length):
    rword_list = []
    
    for i in range(list_length):
        rword = randomword(word_length)
        rword_list.append(rword)
        
    return rword_list
```


## Description of Implemented Hash Functions

In order to produce $k$ hash functions, a *hash_functions_generator* is used. This function, produces $k$ hash functions based on a single template of a hash function for strings (that will be described later). The key is that in order to produce $k$ different functions based on the template, the generator applies an increasing offset that affects the input of the template hash function prior to execute its core algorithm. To make each of these points clear, we will start by describing the template hash function and afterwards the *hash_functions_generator*. 

### Template Hash Function:

The function computes a polynomial whose coefficients are the ASCII integer values of the chars in the input string. In other words, given the polynomial

![](https://latex.codecogs.com/gif.latex?%24%24p%28x%29%20%3D%20%5Csum_%7Bi%3D0%7D%5E%7Bn%7D%20a_ix%5Ei%20%3D%20a_0&plus;a_1x&plus;a_2x%5E2&plus;...&plus;a_nx%5En%24%24)

We define $a_i$ as the value corresponding to the $i_{th}$ char in the input string. $x$ is an arbitrary integer (for this implementation, $x=31$).

However, in order to make this polynomial feasable to compute, a new sequence of constants is defined as follows:

![](https://latex.codecogs.com/gif.latex?%24%24b_n%3A%3Da_n%24%24%5C%5C%20%24%24b_%7Bn-1%7D%3A%3Da_%7Bn-1%7D&plus;b_%7Bn%7Dx_0%24%24%5C%5C%20%24%24.%24%24%5C%5C%20%24%24.%24%24%5C%5C%20%24%24.%24%24%5C%5C%20%24%24b_%7B0%7D%3A%3Da_%7B0%7D&plus;b_%7B1%7Dx_0%24%24)

**Where $b_0$ is then the value of $p(x_0)$.** This function is inspired in the algorithm for computing polynomials known as [Hurner's Rule](https://en.wikipedia.org/wiki/Horner%27s_method).

### Producing several hash functions from the template hash function 

Under the premise that by having a good hash function with a wide output, there will not be correlation between similar inputs of such a hash, one can use such hash function to generate multiple *different* hash functions by passking $k$ different initial values based on the original input. For example, if the input were numbers, for number $8$, we would pass $8,9,...,8+k-1$ to the hash function to simulate the output of many $k$ hash functions. In the case of strings, we may append a dummy value (e.g., "*"), that we will call an offset, $k-1$ times (which is what the generator in this implementation does). 

Here is the excerpt of the code that produces a different hash function, from a template, by considering an offset:


```python
# Produces a hash function based on a template by considering a template as a parameter
def make_hash_function(offset_size):
    
    # HASH FUNCTION TEMPLATE
    def hash_function(string):
        # If we need an offset for the input, we add it.
        # As we're dealing with strings, we'll just add "*"
        # as many times as necessary.
        for i in range(offset_size):
            string += "*"

        # Compute the hash code
        ans = 0
        for i in range(len(string)):
            ans = 31*ans + ord(string[i])
            
        return ans # Inpired in Hurner's Rule
        
    return hash_function
```

And here is the excerpt of the code that models the generator. It sends offsets of sizes $i=0$ to $i=k-1$ to the *make_hash_function* method and returns all the results in a python list

```python
# Generates a list of "different" hash functions
def hash_functions_generator(k):
    
    hashes_list = []
    
    for offset_size_i in range(0,k):
        hash_func = make_hash_function(offset_size_i)
        hashes_list.append(hash_func)
        
    return hashes_list
```

**Below is a demonstration that the different generated hash functions produce values that are *"disparate enough"* for $k=5$ and $input =$"$HelloWorld!$".** 


## Other interesting and *relevant* stuff in the [actual main jupyter notebook (Counting-Bloom-Filter-Overview-and-Introduction.ipynb)](https://github.com/josealvarez97/Counting-Bloom-Filter/tree/master/Introduction%20and%20Implementation%20to%20a%20Counting%20Bloom%20Filter).

* It contains an introduction of my own and complexity analysis of bloom filters and, in particular, counting bloom filters.
* It has a lot of graphs testing (and proving) empirical performance.
