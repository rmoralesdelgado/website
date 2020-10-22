+++
categories = ["nuggets", ]
author = "Raul Morales Delgado"
title = "Concatenating in Numpy with r_ and c_"
description = "In Numpy, the `r_` and `c_` class objects are utilities that allow users to concatenate objects using slice notation. In this post, I explain how to make the most out of them by modifying its string notation and I measure their performance for concatenating and for reshaping."
date = "2020-10-22"
toc = true
tags = ["python", "numpy", ]
+++

In Numpy, the [`r_`](https://numpy.org/doc/stable/reference/generated/numpy.r_.html) and [`c_`](https://numpy.org/doc/stable/reference/generated/numpy.c_.html) class objects are utilities that allow users to concatenate objects using slice notation. Specifically, `c_` is just a shortcut for a particular syntax of `r_`,  as will be explained later on.


## 1. The Syntax
The syntax of `r_` is as follows:
```python
import numpy as np

np.r_[<string>: str, <index expression>: Union[np.ndarray, int, float] [, Union[np.ndarray, int, float]]]
```

Where `<string>` is a `str` object that consists of up to three comma-separated integers that describe how the concatenation will take place. A `str` object of the form `'x,y,z'` can be interpreted as:
* `x`: the axis on which concatenation will take place. `0` means `axis=0` — the first axis—, `1` means `axis=1`, and so on. `-1` stands for the last available axis.
* `y`: the minimum number of dimensions to force each entry into. That is, all to-be-concatenated objects will be upgraded to be `y`-dimensional.
* `z`: is only used when arrays with less than `y` dimensions are provided. In such case, `z` determines *how* the array will be upgraded to be `y`-dimensional. For instance, if you provide a 1-dimensional array with N values — i.e. shape `(N,)` —, and if `y=2` — thus enforcing two dimensions —, then `z=0` will place the `1`s in the upgraded shape *after* `N` — the new array will be of shape `(N,1)`. `z=-1` indicates the opposite — the resulting upgraded array will be of shape `(1,N)`.

Practical explanation of the above will be presented later on.

The `<index expression>` can be one or several Numpy arrays or scalars. Note that lists and nested lists of scalars are also accepted; however, note that lists and nested lists with more than one item will be interpreted as arrays. This will be shown later on as well.

As previously stated, `c_` is just a shorcut expression for:
```python
np.r_['-1,2,0', <index expression>: Union[np.ndarray, int, float] [, Union[np.ndarray, int, float]]]
```

Therefore, the syntax of `c_` is just:
```python
np.c_[<index expression>: Union[np.ndarray, int, float] [, Union[np.ndarray, int, float]]]
```


## 2. Default Concatenation
Suppose we have two 2-dimensional arrays of shape `(3,5)` — 3 rows, 5 columns:
```python
a, b = np.random.randint(100, size=(3,5)), np.random.randint(100, size=(3,5))
print(a, b, sep="\n")
```

```text
[[78  1 15 10 91]
    [59 98 62 41 72]
    [65 82 69 39 82]]
[[10 77 75 37 33]
    [ 1 62 69 20 13]
    [10 32  0 89 20]]
```

`r_` can be used to concatenate on the first axis — axis 0; to stack vertically:
```python
np.r_[a, b]
```

```text
array([[78,  1, 15, 10, 91],
        [59, 98, 62, 41, 72],
        [65, 82, 69, 39, 82],
        [10, 77, 75, 37, 33],
        [ 1, 62, 69, 20, 13],
        [10, 32,  0, 89, 20]])
```

And `c_` can be used to concatenate on the second axis — axis 1; to stack horizontally —:
```python
np.c_[a, b]
```

```text
array([[78,  1, 15, 10, 91, 10, 77, 75, 37, 33],
        [59, 98, 62, 41, 72,  1, 62, 69, 20, 13],
        [65, 82, 69, 39, 82, 10, 32,  0, 89, 20]])
```

Until here, both `r_` and `c_` can be used as a convenience instead of the well known [`np.concatenate`](https://numpy.org/doc/stable/reference/generated/numpy.concatenate.html) function.


## 3. Modifying the String Parameters
Because the explanation for the `<string>` object is a bit confusing, let's start with an example to show what it does. Below, `r_` is used to concatenate two arrays of shape `(3,)`.
```python
np.r_[np.array([1,2,3]), np.array([4,5,6])]
```

```text
array([1, 2, 3, 4, 5, 6])
```

As you may have noted above, `<string>` is optional because its default value — when not specified — is `'0,1,-1'` for `r_` and `'-1,2,0'` for `c_`. Let's run again the example above with `r_`'s default string:
```python
np.r_['0,1,-1', np.array([1,2,3]), np.array([4,5,6])]
```

```text
array([1, 2, 3, 4, 5, 6])
```

As you can see, it returns the same array, which in both cases is of shape `(1,N)` — 1 row, N values.

Let's now modify the string to understand what it actually does. Below, `r_` is used again to concatenate two arrays of shape `(3,)`, but this time the string will be `'-1,2,0'`. This string will do the following:
* `-1`: stack objects on the last axis — i.e. columns for 2-dimensional arrays; stack vertically
* `2`: force each object to have two dimensions minimum — this forces 1-dimensional arrays to have at least two dimensions
* `0`: Upgrades 1-dimensional arrays to `2`-dimensional ones, by placing the `1`s of its new shape at the end of the shape tuple. In our case, a `(3,)` array shape will become a `(3,1)` array shape.

```python
np.r_['-1,2,0', np.array([1,2,3]), np.array([4,5,6])]
```

```text
array([[1, 4],
        [2, 5],
        [3, 6]])
```

Above, each array is upgraded to a `(3,1)` shape — for instance, `array([1,2,3])` upgrades into `array([[1],[2],[3]])`; 3 rows, 1 column. After this reshaping is performed on both arrays, they are then concatenated on the last available axis, the second axis. Thus, `[1]` merges with `[4]`, and so on.

If the last value, `z`, is changed from `0` to `-1`, then `1`s will fill in in the new shape at the beginning of the shape tuple. For instance, a `(3,)` shape will be transformed into a `(1,3)` — 1 row, 3 columns. In this case, all elements will be concatenated in a single *sequence*:
```python
np.r_['-1,2,-1', np.array([1,2,3]), np.array([4,5,6])]
```

```text
array([[1, 2, 3, 4, 5, 6]])
```

When the objects in the index expression are already 2-dimensional arrays with shape `(1,3)`, then the second comma-separated value, `y=2` becomes redundant and the last value, `z=-1` is disregarded. This is shown below: two `(1,3)`-shaped arrays are provided for different `z` values:
```python
np.r_['-1,2,0', np.array([[1,2,3]]), np.array([[4,5,6]])]
```

```text
array([[1, 2, 3, 4, 5, 6]])
```

```python
np.r_['-1,2,99', np.array([[1,2,3]]), np.array([[4,5,6]])]
```

```text
array([[1, 2, 3, 4, 5, 6]])
```


## 4. Reshaping as Well
The `'-1,2,0` is a frequently used routine when concatenating arrays. As stated at the beginning, because of this, the `np.c_`  class is a shortcut equivalent to `np.r_['-1,2,0', index expression]`. This particular expression can be very useful to transform one-dimensional arrays to column vectors:
```python
future_vector = np.arange(10)
future_vector
```

```text
array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
```

```python
np.c_[future_vector]
```

```text
array([[0],
        [1],
        [2],
        [3],
        [4],
        [5],
        [6],
        [7],
        [8],
        [9]])
```

Which, as previously explained, is equivalent to using:
```python
np.r_['-1,2,0', future_vector]
```

```text
array([[0],
        [1],
        [2],
        [3],
        [4],
        [5],
        [6],
        [7],
        [8],
        [9]])
```


## 5. Not Only Arrays
`.r_` and `.c_` can be used to concatenate more than just arrays — they can be used to concatenate lists or individual scalars too. See the following example:
```python
np.c_[np.array([[1,2,3]]), [[10]], 100, [1000], np.array([[4,5]]), np.array([[7,8]])]
```

```text
array([[   1,    2,    3,   10,  100, 1000,    4,    5,    7,    8]])
```

Above, `c_` was used to concatenate both arrays, scalars, lists and nested lists. Note, however, how the following example fails:
```python
np.c_[np.array([[1,2,3]]), [[10]], 100, [1000], np.array([[4,5]]), np.array([7,8])]
```

```text
---------------------------------------------------------------------------

ValueError                                Traceback (most recent call last)

<ipython-input-15-4a609585e2dd> in <module>
----> 1 np.c_[np.array([[1,2,3]]), [[10]], 100, [1000], np.array([[4,5]]), np.array([7,8])]


~/anaconda3/lib/python3.7/site-packages/numpy/lib/index_tricks.py in __getitem__(self, key)
    404                 objs[k] = objs[k].astype(final_dtype)
    405 
--> 406         res = self.concatenate(tuple(objs), axis=axis)
    407 
    408         if matrix:


<__array_function__ internals> in concatenate(*args, **kwargs)


ValueError: all the input array dimensions for the concatenation axis must match exactly, but along dimension 0, the array at index 0 has size 1 and the array at index 5 has size 2
```

It fails because `np.array([7,8])` is an array of shape `(2,)`. This array, under the default string ,`-1,2,0`, is transformed into an array of shape `(2,1)`. However, the remaining arrays are of shape `(1,N)` — only 1 row and N columns. Hence, the error message: `array at index 5 has size 2`; for this to work, `all the input array dimensions for the concatenation axis must match exactly`. In other words, all arrays need to be of shape `(1,N)`.

One last thing to note here is how single-item lists, and single-item nested lists, are interpreted as plain scalars. If more than one item is provided per list, then they are interpreted *as if* they were arrays. See the following examples.
```python
np.c_[np.array([[1,2,3]]), [[10, 20]], 100, [1000, 1020], np.array([[4,5]]), np.array([[7,8]])]
```

```text
---------------------------------------------------------------------------

ValueError                                Traceback (most recent call last)

<ipython-input-16-cd20602a7251> in <module>
----> 1 np.c_[np.array([[1,2,3]]), [[10, 20]], 100, [1000, 1020], np.array([[4,5]]), np.array([[7,8]])]


~/anaconda3/lib/python3.7/site-packages/numpy/lib/index_tricks.py in __getitem__(self, key)
    404                 objs[k] = objs[k].astype(final_dtype)
    405 
--> 406         res = self.concatenate(tuple(objs), axis=axis)
    407 
    408         if matrix:


<__array_function__ internals> in concatenate(*args, **kwargs)


ValueError: all the input array dimensions for the concatenation axis must match exactly, but along dimension 0, the array at index 0 has size 1 and the array at index 3 has size 2
```

In the example above, all arrays are now of shape `(1,N)`, but as the error message suggests, the `array at index 3 has size 2` in its first dimension — the `[1000, 1020]` item is interpreted as one of shape `(2,)`. Even if you try making this a `(2,1)` shape (e.g. `[[1000], [1020]]`), it will still fail. If you make this a `(1,2)`-shape array, then this will work:
```python
np.c_[np.array([[1,2,3]]), [[10, 20]], 100, [[1000, 1020]], np.array([[4,5]]), np.array([[7,8]])]
```

```text
array([[   1,    2,    3,   10,   20,  100, 1000, 1020,    4,    5,    7,
            8]])
```


## 6. Performance
When talking about performance of these classes, it is interesting what they deliver when concatenating and when reshaping. In concatenating, `r_` and `c_` are slightly faster than using Numpy's `concatenate` function. When reshaping, Numpy's `reshape` is **significantly** faster than using either class.

This is a performance measure for concatenating two `(1000,1000)`-shaped arrays:
```python
size = (1_000, 1_000)
a_big, b_big = np.random.randint(100, size=size), np.random.randint(100, size=size)
```

```python
%%timeit
np.concatenate((a_big, b_big), axis = 0)
```

```text
4.27 ms ± 193 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```

```python
%%timeit
np.r_[a_big, b_big]
```

```text
3.89 ms ± 37.1 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```

As seen above, `r_` performs slightly faster.

Below is a performance measure for reshaping a 1-dimensional array into a column vector:
```python
future_vector_big = np.arange(1_000_000)
```

```python
%%timeit
np.reshape(future_vector_big, (-1,1))
```

```text
1.97 µs ± 430 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)
```

```python
%%timeit
np.c_[future_vector_big]
```

```text
2.15 ms ± 85.8 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```

As shown above, `c_` performs 1,000 times slower than using `reshape`.

I hope this post was of some help — I made it because I spent quite some time understanding how to use these two classes — `r_` and `c_` — and when they are actually handy to use. 

Please let me know your thoughts in the comments!

## 7. Acknowledgements

* [NumPy.org > Docs > NumPy v1.19 Manual > NumPy Reference > Routines > Indexing routines > `numpy.r_`](https://numpy.org/doc/stable/reference/generated/numpy.r_.html)
* [NumPy.org > Docs > NumPy v1.19 Manual > NumPy Reference > Routines > Indexing routines > `numpy.c_`](https://numpy.org/doc/stable/reference/generated/numpy.c_.html)
* [NumPy.org > Docs > NumPy v1.19 Manual > NumPy Reference > Routines > Array manipulation routines > `numpy.concatenate`](https://numpy.org/doc/stable/reference/generated/numpy.concatenate.html)
