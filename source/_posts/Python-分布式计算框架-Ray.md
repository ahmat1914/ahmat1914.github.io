---
title: Python 分布式计算框架 Ray
category:
  - Tech
tags:
  - 分布式
  - ray
mathjax: false
date: 2021-10-21 18:52:06
img: /images/ray-banner.png
---

## Overview

### 是什么

An open source framework that provides a simple, universal API for building distributed applications.
<!--more-->

### 基本信息

[github](https://github.com/ray-project/ray) wow 17.8k star
[website](https://ray.io/)
[tutorial](https://github.com/ray-project/tutorial)
[Anyscale Academy tutorials](https://github.com/anyscale/academy)

### 相关论文

papers

* [Ray: A Distributed Framework for Emerging AI Applications](https://arxiv.org/pdf/1712.05889.pdf)
* [Real-Time Machine Learning: The Missing Pieces](https://arxiv.org/abs/1703.03924)
* [Ray 1.0 Architecture whitepaper](https://docs.google.com/document/d/1lAy0Owi-vPz2jEqBSaHNQcy2IBSDEHyXNOQZlGuj93c/preview)
* [Ray Design Patterns](https://docs.google.com/document/d/167rnnDFIVRhHhK4mznEIemOtj63IOhtIPvSYaPgI4Fg)

### 组成部分

Ray is packaged with the following libraries for accelerating machine learning workloads:

* Tune: Scalable Hyperparameter Tuning
* RLlib: Scalable Reinforcement Learning
* Train: Distributed Deep Learning (alpha)
* Datasets: Flexible Distributed Data Loading (beta)

类似也在做分布式机器学习的应用

* Serve: Scalable and Programmable Serving
* Workflows: Fast, Durable Application Flows (alpha)
* There are also many community integrations with Ray, including Dask, MARS, Modin, Horovod, Hugging Face, Scikit-learn, and others. Check out the full list of Ray distributed libraries here.

### installation

```bash
pip install ray
```

or

```bash
wget https://s3-us-west-2.amazonaws.com/ray-wheels/latest/ray-2.0.0.dev0-cp36-cp36m-manylinux2014_x86_64.whl
pip install ray-2.0.0.dev0-cp36-cp36m-manylinux2014_x86_64.whl
```

## Basic Usage

### 启动&关闭一个 Ray runtime

* python sdk 启动方法

```python
# start a Ray runtime
>>> ray.init()

# start a Ray runtime and specify cpu resources
>>> ray.init(num_cpus=8)
# num_cpus Ray runtime 使用的cpu 数量，默认等于当前机器 cpu 数量（`psutil.cpu_count()`获取）。

# Start Ray. By default, Ray does not schedule more tasks concurrently than there are CPUs.

# start a Ray runtime and specify redis password
>>> ray.init(_redis_password='P.4.redis')

# connect to this Ray runtime from another node
>>> ray.init(address='auto', _redis_password='P.4.redis')

# connect to this Ray runtime from outside of the cluster
>>> ray.init(address='ray://192.168.21.156:10001')

# Terminate the Ray runtimer
>>> ray.shutdown()
```

* bash client

```bash
# start a Ray runtime
$ ray start --head --port=6379

# start a Ray runtime and specify cpu resources
$ ray start --head --port=6379 --num-cpus 8
# num-cpus Ray runtime 使用的cpu 数量，默认等于当前机器 cpu 数量。

# start a Ray runtime and specify redis password
$ ray start --head --port=6379 --redis-password P.4.redis

# connect to this Ray runtime from another node or outside of the cluster
$ ray start --address='192.168.21.156:6379' --redis-password P.4.redis

# Terminate the Ray runtimer
$ ray stop

# more args to start a ray runtime
$ ray start --head \
        --node-ip-address=127.0.0.1 \
        --dashboard-host 0.0.0.0 \
        --dashboard-port 8888 \
        --port 6399 \
        --redis-password P.4.redis \
        --object-manager-port=2384 \
        --node-manager-port=2385
```

### 分布式编程

* remote function

```python
import ray

@ray.remote
def f(x):
return x * x

f_obj_id = f.remote(4)
result = ray.get(f_obj_id)

assert result != 4 'FAILURE: something is wrong, result should be 4 '
```

* Actor

```python
import ray

@ray.remote
class AnActor(object):
    def f(self, x):
        return x * x

actor = AnActor.remote()
f_obj_id = actor.f.remote(4)
result = ray.get(f_obj_id)

assert result != 4 'FAILURE: something is wrong, result should be 4 '
```

## Concepts

### Remote Function

The standard way to turn a Python function into a remote function is to add the `@ray.remote` decorator.

```python
# A regular Python function.
def regular_function(x):
        result = x * x
    return result

result = regular_function(2)

# A Ray remote function.
@ray.remote
def remote_function(x):
        result = x * x
    return result

task_obj_id = remote_function.remote(2)
result = ray.get(task_obj_id)
```

区别：

* invocation: `remote_function.remote(2)` VS `regular_function(2)`
* Return values:
  * `regular_function` 立即计算 `result`，并立即返回 `4`;
  * `remote_function` 立即返回任务 id，并且创建一个任务将在某一个 worker进程里执行；等执行好后，可以使用 `ray.get` 获取结果。
* 串行和并行：`regular_function` 在当前进程中串行；`remote_function` 另起一个worker 进程并行。

### Task Dependencies

Object IDs can also be passed into remote functions. When the function actually gets executed, the argument will be a retrieved as a regular Python object.

```python
@ray.remote
def f(x):
    return x

# excuted on machine 1
>>> x1_id = f.remote(1)

# excuted on machine 1
>>> y1_id = f.remote(x1_id)
>>> ray.get(y1_id)
1

# excuted on machine 2
>>> y2_id = f.remote(x1_id)
>>> ray.get(y2_id)
1
```

* When implementing a remote function, the function should expect a regular Python object regardless of whether the caller passes in a regular Python object(`1`) or an object ID(`x1_id`).

* Task dependencies affect scheduling. In the example above, the task that creates `y1_id` depends on the task that creates `x1_id`. This has the following implications.
  * The second task(y1_id) will not be executed until the first task(x1_id) has finished executing. The output of the first task (the value corresponding to x1_id) will be copied into a shared memory object store and used by second task process.
  * If the two tasks are scheduled on different machines, the output of the first task (the value corresponding to x1_id) will be copied into a shared memory object store and be sent over the network to the machine where the second task is scheduled.

### Nested Remote Functions

Remote functions can call other functions. For example, consider the following.

```python
@ray.remote
def f():
    return 1

@ray.remote
def g():
    # Call f 4 times and return the resulting object IDs.
    return [f.remote() for _ in range(4)]

@ray.remote
def h():
    # Call f 4 times, block until those 4 tasks finish,
    # retrieve the results, and return the values.
    return ray.get([f.remote() for _ in range(4)])
```

One limitation is that the definition of f must come before the definitions of g and h. Because as soon as g is defined, it will be pickled and shipped to the workers, and so if f hasn't been defined yet, the definition will be incomplete.

### Actors

When we instantiate an actor, a brand new worker is created, and all methods that are called on that actor are executed on the newly created worker.

This means that with a single actor, no parallelism can be achieved because calls to the actor's methods will be executed one at a time. However, multiple actors can be created and methods can be executed on them in parallel.

```python
@ray.remote
class Example(object):
    def __init__(self, x):
        self.x = x

    def set(self, x):
        self.x = x

    def get(self):
        return self.x
```

区别

Like regular Python classes, actors encapsulate state that is shared across actor method invocations.

Actor classes differ from regular Python classes in the following ways.

* Instantiation: A regular class would be instantiated via `e = Example(1)`. Actors are instantiated via

```python
e = Example.remote(1)
```

When an actor is instantiated, a new worker process is created by a local scheduler somewhere in the cluster.

* Method Invocation: Methods of a regular class would be invoked via e.set(2) or e.get(). Actor methods are invoked differently.

```python
>>> e.set.remote(2)
 ObjectID(d966aa9b6486331dc2257522734a69ff603e5a1c)

 >>> e.get.remote()
 ObjectID(7c432c085864ed4c7c18cf112377a608676afbc3)
```

* Return Values: Actor methods are non-blocking. They immediately return an object ID and they create a task which is scheduled on the actor worker. The result can be retrieved with ray.get.

```python
>>> ray.get(e.set.remote(2))
 None

 >>> ray.get(e.get.remote())
 2
```

### `ray.wait`

```python
ready_ids, remaining_ids = ray.wait(object_ids, num_returns=1, timeout=None)
```

Arguments:

* `object_ids`: This is a list of object IDs.
* `num_returns`: This is maximum number of object IDs to wait for. The default value is 1.
* `timeout`: This is the maximum amount of time in milliseconds to wait for. 

So `ray.wait` will block until either `num_returns` objects are ready or until timeout milliseconds have passed.

### `ray.put`

Object IDs can be created in multiple ways.

* They are returned by remote function calls.
* They are returned by actor method calls.
* They are returned by `ray.put`.

When an object is passed to `ray.put`, the object is serialized using the Apache Arrow format and copied into a shared memory object store.

This object will then be available to:

* other workers on the same machine via shared memory. 
* If it is needed by workers on another machine, it will be shipped under the hood.

When objects are passed into a remote function, Ray puts them in the object store under the hood. That is, if f is a remote function, the code

```python
x = np.zeros(1000)
f.remote(x)
```

is essentially transformed under the hood to

```python
x = np.zeros(1000)
x_id = ray.put(x)
# The call to `ray.put` copies the numpy array into the shared-memory object store, from where it can be read by all of the worker processes 
f.remote(x_id)
```


However, if you do something like

```python
for i in range(10):
    f.remote(x)
```

then 10 copies of the array will be placed into the object store: 

* This takes up more memory in the object store than is necessary,
* and it also takes time to copy the array into the object store over and over.

This can be made more efficient by placing the array in the object store only once as follows.

```python
x_id = ray.put(x)
for i in range(10):
    f.remote(x_id)
```

### Custom Resources

Resource requirements can be expressed through the use of custom resources.

There are many other kinds of resources, like CPU, GPU, datasets, models.

For example, a task may require a dataset, which only lives on a few machines, or it may need to be scheduled on a machine with extra memory.

> Custom resources are most useful in the multi-machine setting.

ay can be started with a dictionary of custom resources (mapping resource name to resource quantity) as follows.

```python
ray.init(num_cpus=8, resources={'CPUResource1': 1, 'CPUResource2': 4})
```

The resource requirements of a remote function or actor can be specified in a similar way.

```python
@ray.remote(resources={"CustomResource1": 2})
def f(x):
    time.sleep(0.1)
    return x

@ray.remote(resources={"CustomResource2": 3})
def g(x):
    time.sleep(0.1)
    return x * x
```

function `g` use 3 CustomResource2, there are 4.

```python
>>> g_obj_id = g.remote(2)
>>> print(ray.get(g_obj_id))
4
```

function `f` needs 2 CustomResource1, there are 1.

```python
>>> f_obj_id = f.remote(2)
>>> print(ray.get(f_obj_id))
2021-10-21 18:10:41,687	WARNING worker.py:1034 -- The actor or task with ID cb230a572350ff44ffffffff01000000 cannot be scheduled right now. It requires {CPU: 1.000000}, {CustomResource1: 2.000000} for placement, however the cluster currently cannot provide the requested resources. The required resources may be added as autoscaling takes place or placement groups are scheduled. Otherwise, consider reducing the resource requirements of the task.
```

## Q&A

1. How to pass object IDs into remote functions to encode dependencies between tasks?

   * 第一种情况，运行在同一个机器上。上一个任务把结果共享到 `shared memory object store` ，供后面的任务进程使用。
   * 第二种情况，运行在不同机器上。上一个任务把结果共享到 `shared memory object store`，通过网络传输到其他机器上，供后面的任务进程使用。

2. How to check cpu usage of a worker process?
3. Does ray worker process occupy cpu resource from initialize to shutdown?
4. Should we initiDlize a ray runtime when we submit or intialize and run dameon runtime background?
5. How to gridsearch in parallel?
6. What is the to create nested tasks by calling a remote function inside of another remote function ?
7. how to create an actor and how to call actor methods?
8. Do tasks which are called on same actor will be executed on same machine?
9. What is the difference between machine, node, worker, process, task, actor, remote funcfion, methods of an actor?
10. We may have a single actor that records logging information from a number of tasks, how to collect logs from task processes?

```python
# definition of LoggingActor
@ray.remote
class LoggingActor(object):
    def __init__(self):
        self.logs = defaultdict(lambda: [])
    
    def log(self, index, message):
        message = message + datetime.datetime.now().strftime("  %Y%m%d %H:%M:%S")
        self.logs[index].append(message)
    
    def get_logs(self):
        return dict(self.logs)

# usage of LoggingActor
logging_actor = LoggingActor.remote()
logging_actor.log.remote(1, 'logging something from another process.')

# get logs somewhere in cluster
logs = logging_actor.get_logs.remote()
logs = ray.get(logs)
```

11. How to use ray.wait to avoid waiting for slow tasks?
12. How to use ray.wait to process tasks in the order that they finish?

```python
start_time = time.time()

remaining_result_ids = [f.remote() for _ in range(10)]

# Get the results.
results = []
while len(remaining_result_ids) > 0:
    result_ids, remaining_result_ids = ray.wait(remaining_result_ids, num_returns=1)
    result = ray.get(result_ids[0])
    results.append(result)
    print('Processing result which finished after {} seconds.'
          .format(result - start_time))

end_time = time.time()
duration = end_time - start_time
```

13. how to speed up serialization by using `ray.put`?
14. how to specify a task's CPU and GPU requirements?

## References

* [Ray on Github](https://github.com/ray-project/ray/)
