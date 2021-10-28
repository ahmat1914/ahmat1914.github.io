---
title: Methods of same actor instance will share same process
category:
  - Note
tags:
  - ray
  - 分布式
mathjax: false
date: 2021-10-28 17:34:46
img:
---

Actor - a stateful worker process (an instance of a `@ray.remote` class).

<!--more-->

> with a single actor, no parallelism can be achieved because calls to the actor’s methods will be executed one at a time.

> A task can be stateless (a `@ray.remote` function) or stateful (a method of a `@ray.remote` class(Actor))

Actor 有状态，意味着共用同一个实例。同一个实例提交的不同的 task 就具备了一样的属性，如 进程、元信息。

```python
import os

import names
import ray


@ray.remote(num_cpus=1)
class Worker:
    def __init__(self):
        self.name = names.get_full_name()

    def work(self):
        return (os.getpid(), self.name)


ray.init()
same_worker = Worker.remote()

print(ray.get([same_worker.work.remote() for _ in range(3)]))
ray.shutdown()
```

```bash
2021-10-28 17:38:30,642 INFO services.py:1173 -- View the Ray dashboard at http://127.0.0.1:8265

[(40925, 'Renee Colbeth'), (40925, 'Renee Colbeth'), (40925, 'Renee Colbeth')]
```

相对的，不同的实例有不同的状态。

```python
import os

import names
import ray


@ray.remote(num_cpus=1)
class Worker:
    def __init__(self):
        self.name = names.get_full_name()

    def work(self):
        return (os.getpid(), self.name)


ray.init()

workers = [Worker.remote() for _ in range(3)]
print(ray.get([worker.work.remote() for worker in workers]))
ray.shutdown()

```

```bash
View the Ray dashboard at http://127.0.0.1:8265

[(10752, 'Kathryn Hodge'), (10816, 'Melvin Borden'), (10758, 'Sherry Hampshire')]
```
