---
usemathjax: true
layout: post
title: "Quirky behavior of numpy.clip"
date: 2022-04-11
background: '/img/bg-index.png'
---

For a change a short post on some quirky `numpy.clip` behavior that surfaced during a recent code review at work. The `numpy.clip` function takes 3 number-like arguments, like `np.clip(x, l, u)` and clips the number $x$ such that the output lies in the interval $[l, u]$, also commonly written like $x \in [l, u]$. 

Another popular way to write $x \in [l, u]$ using inequalities rather than set notation is $l \leq x \leq u$. An alternative, quite natural interface to `numpy.clip` could therefore also have been `np.clip(l, x, u)`. 

The quirky behavior of `numpy.clip` we recently encountered is that the order of the first two arguments does in fact not matter, and both give the same output for all possible inputs. The [documentation](https://numpy.org/doc/stable/reference/generated/numpy.clip.html) specifies that `numpy.clip(x, l, u)` is equivalent to `np.minimum(u, np.maximum(x, l))`, which makes immediately clear why it works both ways. 

One could argue that standard (correct) usage of the clipping method satisfies $l \leq u$, and the clipping method might want to check that. With such a check, the input case $l < u < x$, would be the only one to show a difference between the two different calls `np.clip(x, l, u)` and `np.clip(l, x, u)`. The former would yield `u` while the latter would yield the error that the second argument is larger than the third.
However, the numpy documentation is also clear on that: 

    No check is performed to ensure a_min < a_max.
    
So, what do you think is the best choice? Have no check, such that it supports what is in principle incorrect interface usage? Or have a check, which enforces correct usage, but leads to more work from users that (initially) used the incorrect, yet working alternative interface?


```python
import numpy as np
import itertools
a, b, c = 1, 2, 3

for x, l, u in itertools.permutations([a, b, c]):
    print(x, l, u, ' ---- ', np.clip(x, l, u), ' - ', np.clip(l, x, u))
```

    1 2 3  ----  2  -  2
    1 3 2  ----  2  -  2
    2 1 3  ----  2  -  2
    2 3 1  ----  1  -  1
    3 1 2  ----  2  -  2
    3 2 1  ----  1  -  1
    
-----

## Related posts
None yet

## Want to leave a comment?
Very interested in your comments but still figuring out the most suited approach to this. For now, feel free to send me an [email](mailto:lenseswaenen@gmail.com)
