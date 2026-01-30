# Unpack operators

Runnable gist:[https://gist.github.com/inspektral/14763622f1d273791ded8afce78e90d5]()

```python
def sum(*args):
    sum = 0
    for x in args:
        sum += x
    return sum

print(sum(1,2,3))
```

prints `6`,
and this is all fine, but how do those * work?

```python
a = [1,2,3]
print(a, *a)
```

prints `[1, 2, 3] 1 2 3`

So they work in both ways? How? What?
they are called unpack operators, and fine in front of iterables they can unpack them, so things like this are possible:

```python
l1 = [1,2,3,4]
l2 = [5,6,7,8]

l3 = [*l1, *l2]

print(l1, l2, l3)
```

Prints: `[1, 2, 3, 4] [5, 6, 7, 8] [1, 2, 3, 4, 5, 6, 7, 8]`and now the weird part arrives...

```python
a, *b, c = l3
print(a,b,c)
```

Prints: `1 [2, 3, 4, 5, 6, 7] 8` so here is like a pack operator? 

Apparently there are various operations that the CPython interpeter can to UNPACK_EX, when the * is on the left of an assignment like:

```python
a, *b, c = [1,2,3,4]
```

It looks onto the right of the assignement, checks how much stuff there is, assigns it to the non-* variables and the rest to the * variables as a LIST

```python
import dis

def test_assignment():
    l = [1, 2, 3]
    a, *b = l

dis.dis(test_assignment)
```

the *args thing, which is happens before the function call, completely in C, that assigns the arguments to a TUPLE

```python
def test_func(*args):
    return args

dis.dis(test_func)
```

In general, besides the usual differences between tuple and list they are, very similar. It is done like this because the unpack is usually done to manipulate the slice, and to be consistent it is a slice, which is much heavier, on the contrary the *args are usually not modified in place, so a tuple, much more efficient is used
