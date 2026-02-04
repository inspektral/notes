---
layout: default
title: pathlib
parent: Python
---

# pathlib

Let's try to understand how this works, I feel like it is quite relevant if we want to deal with files.

Before there was os to deal with this kind of stuff, and it treated paths as strings. Now we can use Path and treat them as objects.
In pathlib it's a tiny bit slower, but it manages platforms and a lot of other things, way more powerful an basically the go to, at least since python 3.12

```python

```

Let's start by creating a path and using the join `/` operator

```python
from pathlib import Path

base_dir = Path('/home/tito')
python_dir = base_dir / 'test'
this_file = python_dir / 'path.py'

print(base_dir)
print(python_dir)
print(this_file)
```

prints: `/home/tito`
prints: `/home/tito/test`
prints: `/home/tito/test/path.py`
Now let's check some utils:

- `name` returns the filename
- `stem` returns filename without extension
- `suffix` returns file extension
- `parent` returns the directory containing the file

```python
print(this_file.name)
print(this_file.stem)
print(this_file.suffix)
print(this_file.parent)
```

prints: `path.py`
prints: `path`
prints: `.py`
prints: `home\tito\test`**n **
