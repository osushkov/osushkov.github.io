---
layout: post
title: Problem with Python Lambdas
---

Python is a pretty nice language in many ways. It allows very fast prototyping and experimenting with algorithms. However, it's dynamic nature
and some language design decisions can really cause some bizzare bugs. I ran into this interesting example recently relating to lambdas, variable
capture, and variable scoping. At first glance the below snippets should all produce the same output.

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
  print("{} {}".format(i, funcs[i]()))
```

Snippet B:

```python
for j in range(len(funcs)):
  print("{} {}".format(j, funcs[j]()))
```

Snippet C:

```python
def stuff():
  for i in range(len(funcs)):
    print("{} {}".format(i, funcs[i]()))

stuff()
```

One might expect that the output would be the following:
```
1 1
2 2
3 3
4 4
```

However, due to the fact that Python lambda variable capture is by-reference and the weird variable scoping rules, only *Snippet A* works "as expeced".
When I ran into this it was causing a relatively subtle bug and took a fair amount of time to find. Now I know.

