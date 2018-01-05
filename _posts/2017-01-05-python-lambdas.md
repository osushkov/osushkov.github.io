---
layout: post
title: Problem with Python Lambdas
---

Python is a pretty nice language in some ways. It allows very fast prototyping and experimenting with algorithms. However, it's dynamic nature
and some language design decisions can really cause some bizzare bugs. I ran into this interesting example recently relating to lambdas, variable
capture, and variable scoping. At first the below snippets should all produce the same output.

Common code:
```python
my_list = np.array([1,2,3,4])
funcs = []

for i, v in enumerate(my_list.tolist()):
  funcs.append(lambda: my_list[i])
```

Snippet A:
```python
for i in range(len(funcs)):
  print("value A: {} {}".format(i, funcs[i]()))
```

Snippet B:
```python
for j in range(len(funcs)):
  print("value B: {} {}".format(j, funcs[j]()))
```

Snippet C:
```python
def stuff():
  for i in range(len(funcs)):
    print("value C: {} {}".format(i, funcs[i]()))

stuff()
```

