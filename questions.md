# Questions that will need an answer

## How does pickle work? What makes something pickleable?

Well basically it takes an object, serialize it and save it to a compressed binary.
Something is not pickleable when it is not serializable. The problem that we were
experiencing was related to that:

```python
class A:

    def __init__(self):
        self.pointer = cpointer() # lets say that this is some c pointer, an address, not a value

a = A()

pickle.dump(A) # works, class itself contains the constructor, not the pointer
pickle.dump(a) # fails, instance contains the pointer
```

So what have we learned? If possible minimize pointers and make them as local as possible

## How does ONNX work? What makes something ONNX compatible?

## How do dataclasses work? Everything about dataclass, property, field

## How to use the python debugger from the terminal

## How to use a profiler preferably from the terminal
