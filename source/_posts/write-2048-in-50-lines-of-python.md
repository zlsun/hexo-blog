title: 50 行Python代码实现 2048
tags:
  - '2048'
categories:
  - Python
date: 2016-03-10 16:42:00
---

# 代码

```py
#!/bin/python3
from random import choice
from os import system
from readchar import readchar, readkey
from colorama import init
from termcolor import colored

N = 4
SEED = [2] * 9 + [4]
FGS = ['white', 'green', 'yellow', 'blue', 'cyan', 'magenta', 'red']
TERM = (24, 80)
OFFSET = (TERM[0] // 2 - 2, TERM[1] // 2 - 10)

pos          = lambda y, x: '\x1b[%d;%dH' % (y, x)
color        = lambda i: colored('%4d' % i, FGS[len('{:b}'.format(i)) % len(FGS)] if i else 'grey')
formatted    = lambda m: '\n'.join(pos(y, OFFSET[1]) + ' '.join(color(i) for i in l) for l, y in zip(m, range(OFFSET[0], OFFSET[0] + 4)))
combine      = lambda l: ([l[0] * 2] + combine(l[2:]) if l[0] == l[1] else [l[0]] + combine(l[1:])) if len(l) >= 2 else l
expand       = lambda l: [l[i] if i < len(l) else 0 for i in range(N)]
merge_left   = lambda l: expand(combine(list(filter(bool, list(l)))))
merge_right  = lambda l: merge_left(l[::-1])[::-1]
left         = lambda m: list(map(merge_left, m))
right        = lambda m: list(map(merge_right, m))
up           = lambda m: list(map(list, zip(*left(zip(*m)))))
down         = lambda m: list(map(list, zip(*right(zip(*m)))))
add_num_impl = lambda m, p: m[p[0]].__setitem__(p[1], choice(SEED))
add_num      = lambda m: add_num_impl(m, choice([(x, y) for x in range(N) for y in range(N) if not m[x][y]]))
win          = lambda m: 2048 in sum(m, [])
gameover     = lambda m: all(m == t(m) for t in trans.values())
draw         = lambda m: system("clear") or print(formatted(m))

trans = {'a': left, 'd': right, 'w': up, 's': down}
m = [[0] * N for _ in range(N)]
init()
add_num(m)
draw(m)
while True:
    while True:
        move = readkey()
        if move in list(trans.keys()) + ['q']:
            break
    if move == 'q':
        break
    n = trans[move](m)
    if n != m:
        add_num(n)
    m = n
    draw(m)
    if win(m):
        print('\n' + colored('(^_^) You Win!'.center(TERM[1]), 'yellow'))
        break
    elif gameover(m):
        print('\n' + colored('(＞﹏＜) Game Over!'.center(TERM[1]), 'red'))
        break
```

## 截图

![](/images/write-2048-in-50-lines-of-python/1.png)
![](/images/write-2048-in-50-lines-of-python/2.png)
