---
layout: post
title: "Functions with Timeouts and Multiprocessing in Python"
menutitle: "Functions with Timeouts and Multiprocessing in Python"
date: 2020-08-10 22:22:00 +0000
tags: functions multiprocessing timeouts python time out Windows Mac OS X
category: Quant Dev
author: am
published: true
redirect_from: "/2020-08-10-functions-with-timeouts-and-multiprocessing-in-python/"
language: EN
comments: true
---

## Contents
{:.no_toc}

* This will become a table of contents (this text will be scraped).
{:toc}

# Problem

There are often cases in quant finance where we may want to call a function which becomes useless if it
takes too long to evaluate e.g. in rapidly moving markets a tick-based strategy may fail to evaluate fast enough
and could need a time based kill switch

# Why this is hard in Python
This problem becomes quite tricky, particularly on Windows machines where there is not option to use `signal`.
Furthermore, the issue can be complicated by the need to remain agile between python versions.

There is then a further issue that `multiprocessing` can be particularly annoying with the requirement to protect
target functions inside `if __name__ == '__main__`


# Approach
My approach is to use `multiprocess.Process` to run the function and then kill it if it breaches a timeout threshold.

There are two ways to approach this, one is with `time.sleep` where we can do something like populate a mutable object and then kill the process irrespective of whether it has finished. The more robust way is to use `multiprocess.Queue` which has a timeout built into it.


## Simple Start

The following is a way to put these initial ideas into practice, we can play with the `timeout` as a proof of concept

```python
import time
import multiprocessing as mp
import multiprocessing.queues as mpq

from typing import Tuple, Callable, Dict


def _lemmiwinks(func: Callable, args: Tuple[object], kwargs: Dict[str, object], q: mp.Queue):
    """lemmiwinks crawls into the unknown"""
    q.put(func(*args, **kwargs))


def foo(x):
    time.sleep(x)
    return x


if __name__ == '__main__':
    x = 2
    timeout = 1

    q_worker = mp.Queue()
    proc = mp.Process(target=_lemmiwinks, args=(foo, (x,), {}, q_worker))
    proc.start()
    try:
        res = q_worker.get(timeout=timeout)
        print(res)
    except mpq.Empty:
        proc.terminate()
        print('Timeout!')
```

We can create a `TimeoutError` class to handle the timeouts like

```python
class TimeoutError(Exception):

    def __init__(self, func, timeout):
        self.t = timeout
        self.fname = func.__name__

    def __str__(self):
            return f"function '{self.fname}' timed out after {self.t}s"
```

Now I actually found a few cases where the `proc` failed to terminate with an `AttributeError`. Secondly, we kind of waht to make sure it terminates all the time anyway so I use the following

```python
    try:
        return q_worker.get(timeout=timeout)
    except mpq.Empty:
        raise TimeoutError(func, timeout)
    finally:
        try:
            proc.terminate()
        except:
            pass
```

## Avoiding AttributeError when doing multiprocessing

One of the most annoying things when doing multiprocessing in python is the need to protect all the functions
and routines using

```python
<<functions>>

if '__main__' == __name__:`
    <<parallel spawning>>
```

To get around this we can use `dill` to compress the function into a string and pass that instead with `dill.dumps(func)` and `dill.loads(func_string)(*args, **kwargs)`

The only real drawback of doing this is that now we need to declare all our imports that are useful to the function inside the function like

```python
def foo(x):
    import time
    time.sleep(x)
    return x
```


## As a Decorator

For single calls then it makes sense to create a decorator like below

```python
import multiprocessing as mp
import multiprocessing.queues as mpq
import functools
import dill

from typing import Tuple, Callable, Dict, Optional, Iterable, List

class TimeoutError(Exception):

    def __init__(self, func, timeout):
        self.t = timeout
        self.fname = func.__name__

    def __str__(self):
            return f"function '{self.fname}' timed out after {self.t}s"


def _lemmiwinks(func: Callable, args: Tuple[object], kwargs: Dict[str, object], q: mp.Queue):
    """lemmiwinks crawls into the unknown"""
    q.put(dill.loads(func)(*args, **kwargs))


def killer_call(func: Callable = None, timeout: int = 10) -> Callable:
    """
    Single function call with a timeout

    Args:
        func: the function
        timeout: The timeout in seconds
    """

    if not isinstance(timeout, int):
        raise ValueError(f'timeout needs to be an int. Got: {timeout}')

    if func is None:
        return functools.partial(killer_call, timeout=timeout)

    @functools.wraps(killer_call)
    def _inners(*args, **kwargs) -> object:
        q_worker = mp.Queue()
        proc = mp.Process(target=_lemmiwinks, args=(dill.dumps(func), args, kwargs, q_worker))
        proc.start()
        try:
            return q_worker.get(timeout=timeout)
        except mpq.Empty:
            raise TimeoutError(func, timeout)
        finally:
            try:
                proc.terminate()
            except:
                pass
    return _inners
```

The worthwhile part of this is to note that we can pass keyword arguments to the wrapper by declaring `func=None` and then using `functools.partial` to prepare the wrapper on the fly

```python
    if func is None:
        return functools.partial(killer_call, timeout=timeout)
```

## Parallel Calls

Finally, we may also stumble into the issue of daemonic processes not being able to spawn children. I find this error hilarious. This will occur if you try to parallelise a `multiprocessing.Process` within another `multiprocessing.Process` that has the `daemon` attribute set as True

This is always the case in the mapping functions like `multiprocessing.Pool().map()` and all of the `pathos.multiprocessing` library

The requirement is then that we should have a parallel map function for our function killer.

We can do this by modifying the above script to pass the function parameters about via and input queue and an output queue using the `multiprocessing.Queue` objects to their potential.

### A Parallel Map

I will start at the outside as this is how I approached the problem myself.

The function below will create two queues and then put all the iterable arguments into `q_in` with an index associated to their ordering like

```python
    q_in = mp.Queue()
    q_out = mp.Queue()
    sent = [q_in.put((i, x)) for i, x in enumerate(iterable)]
```

We then create the processes that point to some kind of `_queue_mgr` function which we will write later.
The processes in this instance are the ones that are doing the parallelism so we want as many processes as we think sensible given the number of cores on the machine. I tend to use max cores - 2 as I like the extra capacity for background tasks.

```python
    processes = [
        mp.Process(target=_queue_mgr, args=(dill.dumps(func), q_in, q_out, timeout, pid))
        for pid in range(cpus)
    ]
```

After starting the processes (**don't forget this!!**) we then retrieve the contents from the `q_out` as we assume this will
be populated by the `_queue_mgr` function which we have yet to define.

This `.get` is blocking and will act like a join

```python
    print(f'Started {len(processes)} processes')
    for proc in processes:
        proc.start()

    result = [q_out.get() for _ in sent]
```

finally kill off the processes to be sure they don't linger about and sort the results to get a function like the below

```python
def killer_pmap(func: Callable, iterable: Iterable, cpus: int, timeout: int = 4):
    """
    Parallelisation of func across the iterable with a timeout at each evaluation

    Args:
        func: The function
        iterable: The iterable to map func over
        cpus: The number of cpus to use. Default is the use max - 2.
        timeout: kills the func calls if they take longer than this in seconds
    """

    q_in = mp.Queue()
    q_out = mp.Queue()
    sent = [q_in.put((i, x)) for i, x in enumerate(iterable)]

    processes = [
        mp.Process(target=_queue_mgr, args=(dill.dumps(func), q_in, q_out, timeout, pid))
        for pid in range(cpus)
    ]
    print(f'Started {len(processes)} processes')
    for proc in processes:
        proc.start()

    result = [q_out.get() for _ in sent]

    for proc in processes:
        proc.terminate()

    return [x for _, x, in sorted(result)]
```


### Managing the in/out queues

The in/out queues can be managed by looping until `q_in` is depleted of items. Therein the function isn't too dissimilar to the one we previously wrote.

Instead of returning, we make sure to put the contents in the output queue like `q_out.put`.

For clarification we have three queues here: 

 - An input queue: containing all the remaining inputs that haven't been processed
 - An output queue: containing all the results
 - A worker queue: This contains the timeout and will allow us to kill processes that take too long

The full function then becomes

```python
def _queue_mgr(func_str: str, q_in: mp.Queue, q_out: mp.Queue, timeout: int, pid: int) -> object:
    """
    Controls the main workflow of cancelling the function calls that take too long
    in the parallel map

    Args:
        func_str: The function, converted into a string via dill (more stable than pickle)
        q_in: The input queue
        q_out: The output queue
        timeout: The timeout in seconds
        pid: process id
    """
    while not q_in.empty():
        positioning, x  = q_in.get()
        q_worker = mp.Queue()
        proc = mp.Process(target=_lemmiwinks, args=(func_str, (x,), {}, q_worker,))
        proc.start()
        try:
            print(f'[{pid}]: {positioning}: getting')
            res = q_worker.get(timeout=timeout)
            print(f'[{pid}]: {positioning}: got')
            q_out.put((positioning, res))
        except mpq.Empty:
            q_out.put((positioning, None))
            print(f'[{pid}]: {positioning}: timed out ({timeout}s)')
        finally:
            try:
                proc.terminate()
                print(f'[{pid}]: {positioning}: terminated')
            except:
                pass
    print(f'[{pid}]: completed!')
```

If we now run the parallel example we will get something like

```python
>>> def foo(x):
...     import time
...     time.sleep(x)
...     return x
>>> killer_pmap(foo, range(6))
[0, 1, 2, 3, None, None]
```

where we return `None` values. I leave it to the user to change this / adapt it for generic 'fail' values.


# Full code

Putting it all together we have the full module like

```python
import multiprocessing as mp
import multiprocessing.queues as mpq
import functools
import dill

from typing import Tuple, Callable, Dict, Optional, Iterable, List

class TimeoutError(Exception):

    def __init__(self, func, timeout):
        self.t = timeout
        self.fname = func.__name__

    def __str__(self):
            return f"function '{self.fname}' timed out after {self.t}s"


def _lemmiwinks(func: Callable, args: Tuple[object], kwargs: Dict[str, object], q: mp.Queue):
    """lemmiwinks crawls into the unknown"""
    q.put(dill.loads(func)(*args, **kwargs))


def killer_call(func: Callable = None, timeout: int = 10) -> Callable:
    """
    Single function call with a timeout

    Args:
        func: the function
        timeout: The timeout in seconds
    """

    if not isinstance(timeout, int):
        raise ValueError(f'timeout needs to be an int. Got: {timeout}')

    if func is None:
        return functools.partial(killer_call, timeout=timeout)

    @functools.wraps(killer_call)
    def _inners(*args, **kwargs) -> object:
        q_worker = mp.Queue()
        proc = mp.Process(target=_lemmiwinks, args=(dill.dumps(func), args, kwargs, q_worker))
        proc.start()
        try:
            return q_worker.get(timeout=timeout)
        except mpq.Empty:
            raise TimeoutError(func, timeout)
        finally:
            try:
                proc.terminate()
            except:
                pass
    return _inners


def _queue_mgr(func_str: str, q_in: mp.Queue, q_out: mp.Queue, timeout: int, pid: int) -> object:
    """
    Controls the main workflow of cancelling the function calls that take too long
    in the parallel map

    Args:
        func_str: The function, converted into a string via dill (more stable than pickle)
        q_in: The input queue
        q_out: The output queue
        timeout: The timeout in seconds
        pid: process id
    """
    while not q_in.empty():
        positioning, x  = q_in.get()
        q_worker = mp.Queue()
        proc = mp.Process(target=_lemmiwinks, args=(func_str, (x,), {}, q_worker,))
        proc.start()
        try:
            print(f'[{pid}]: {positioning}: getting')
            res = q_worker.get(timeout=timeout)
            print(f'[{pid}]: {positioning}: got')
            q_out.put((positioning, res))
        except mpq.Empty:
            q_out.put((positioning, None))
            print(f'[{pid}]: {positioning}: timed out ({timeout}s)')
        finally:
            try:
                proc.terminate()
                print(f'[{pid}]: {positioning}: terminated')
            except:
                pass
    print(f'[{pid}]: completed!')


def killer_pmap(func: Callable, iterable: Iterable, cpus: Optional[int] = None, timeout: int = 4):
    """
    Parallelisation of func across the iterable with a timeout at each evaluation

    Args:
        func: The function
        iterable: The iterable to map func over
        cpus: The number of cpus to use. Default is the use max - 2.
        timeout: kills the func calls if they take longer than this in seconds
    """

    if cpus is None:
        cpus = max(mp.cpu_count() - 2, 1)
        if cpus == 1:
            raise ValueError('Not enough CPUs to parallelise. You only have 1 CPU!')
        else:
            print(f'Optimising for {cpus} processors')

    q_in = mp.Queue()
    q_out = mp.Queue()
    sent = [q_in.put((i, x)) for i, x in enumerate(iterable)]

    processes = [
        mp.Process(target=_queue_mgr, args=(dill.dumps(func), q_in, q_out, timeout, pid))
        for pid in range(cpus)
    ]
    print(f'Started {len(processes)} processes')
    for proc in processes:
        proc.start()

    result = [q_out.get() for _ in sent]

    for proc in processes:
        proc.terminate()

    return [x for _, x, in sorted(result)]

if __name__ == '__main__':
    @killer_call(timeout=4)
    def bar(x):
        import time
        time.sleep(x)
        return x

    try:
        bar(10)
    except TimeoutError as e:
        assert str(e) == "function 'bar' timed out after 4s"

    assert bar(2) == 2

    def foo(x):
        import time
        time.sleep(x)
        return x

    assert killer_pmap(foo, range(6)) == [0, 1, 2, 3, None, None]
```

# References

- [Timeout on a Function Call](https://stackoverflow.com/questions/492519/timeout-on-a-function-call/63349156#63349156)