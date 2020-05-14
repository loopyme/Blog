---
layout:     post
title: Task Graph & Pandas-profiling
subtitle: 任务图与Pandas-profiling
date:       2020-05-14
author:     Loopy
header-img: img/home-bg-geek.jpg
catalog: true
tags:
    - Fun
    - Algorithm
    - Pypi

---


## Problem in Pandas-profiling

I am recently very interested in [`Pandas-profiling`](https://github.com/pandas-profiling/pandas-profiling) and do contribute some codes (besides, it's my first time to use Slack for collaborate).

Simply speaking, `Pandas-profiling`(Hereinafter referred to as `PP`) build reports for data sets. Since main task is clear, `PP`'s works have obvious temporal relation, so it is designed to get the final result directly at the first place.

However, as the functionality expands, this pattern bring some problems:

 1. When user wants some intermediate results, (e.g. get the summery in json instead of HTML report) `PP` would insist to do all the computation.
 2. When user adjust some parameters, (e.g. adjust Chi-square test threshold after read the report) `PP` would also insist to do all the computation. 

The problems may be solved with cache-pipeline, but it will make each step very granular(summary->plotting->render->save), some computations will still be unnecessary. What if only a tiny part like Chi-square threshold of summary should be updated? Maybe only one line will change on the HTML report, but the entire report will still be regenerated.

How can we make computer lazy* and only do those computations are real necessary to improve the performance?

> Unlike many other languages, `Python` is dynamically typed language, and ever more dynamic than some languages like `Java`. As a result, in a python project, functions and tasks often change, that makes feature 'lazy' become more important
> 
> `Dask` is a powerful tool to make computation delayed, but we don't only need delayed, but also lazy --- avoid useless computations


## Solve the problem with Task Graph

Similar to `PP`, the main task of many python projects can be considered as a static collection of atomic tasks with dependencies between input and output, which can be thought of as forming the vertices of a directed acyclic graph, while the directed edges of the graph show the dependencies between tasks.

**That's the Task Graph!**

We can solve the problems with the following techniques:
 - **Up-Propagate**: If some data item is accessed, we need to do the up-propagate and find all upstream tasks and do the correspond tasks. The result of an upstream task may be passed to downstream tasks on completion of the upstream. Data does not stream to downstream task during execution, the upstream task completes its work, terminates, and then its result is passed to the downstream task. The final result of the graph is returned when the last downstream task(s) complete.
 - **Cache**: when a task been computed once(Usually triggered by Up-Propagate), it will automatically cache the result.
 - **Down-Propagate** : If one task is changed, we need to do the down-propagate and find all downstream tasks and clear the correspond task caches. The result of an upstream task is much likely to have effect on all of its' downstream tasks, so the recomputation is necessary. But Down-Propagate will not trigger the recomputation but only clear the result caches, recomputation will happened when this task has been 'Up-Propagated' or accessed.

**Some other interesting things about Task Graph:**
 - We can do parallel scheduling when 'Up-Propagate'
 - In principle, individual tasks must handle any errors, but with Task-Graph manager, we can add error capture there.

## Example on Task Graph

Firstly, we need to figure out the data flow and draw a task graph.

Then, we need to implement task graph on the given tasks collection, consider a project with task graph like this:

<script src="https://cdnjs.cloudflare.com/ajax/libs/mermaid/8.0.0/mermaid.min.js"></script>
<script>mermaid.initialize({startOnLoad:true});</script>
<div class="mermaid">
graph TD

    A{A}
    B{B}
    C{C}
    D{D}
    E{E}

    Task1((Task1))
    Task2((Task2))
    Task3((Task3))
    Task4((Task4))
    Task5((Task5))

    Input1-->Task1
    Task1-->A

    Input1-->Task2
    Input2-->Task2
    Task2-->B
    Task2-->C

    A-->Task3
    B-->Task3
    Task3-->D

    C-->Task4
    Task4-->E

    D-->Task5
    E-->Task5
    Task5-->Output
</div>

This project has 3 inputs, 1 outputs, and A, B, C... are some intermediate results. 

Finally, We can solve the problems we raised before with the following techniques:

 - **Up-Propagate**: If D is accessed, we need to do the up-propagate and find all upstream tasks(Task1,2,3) and do the correspond tasks.
 - **Down-Propagate** : If Task4 is changed, we need to do the down-propagate and find all downstream tasks(Task4,5) and clear the correspond task caches(E,Output)
 - **Cache**: when a task been computed(Usually triggered by Up-Propagate), it will automatically cache the result.

Computer is lazy now! Do all the work dutifully, but do the least!

## Task Graph package

I build a package to make this easier, codes below can describe the task graph in the example section , manage and run the computations:
```python
from task_graph import TaskGraph


#################################
# part A: define the tasks
def Task1(*args):
    # do the task
    return res

# many more tasks ...

#################################
# part B: describe the TaskGraph
tg = TaskGraph()

A = tg.add_task(Task1)(input1)
B_C = tg.add_task(Task2)(input1, input2)
B = tg.add_task("__getitem__")(B_C, 0)
C = tg.add_task("__getitem__")(B_C, 1)
D = tg.add_task(Task3)(A, B)
E = tg.add_task(Task4)(C)
Output = tg.add_task(Task5)(D,E)

res = Output.compute()

#################################
# part C: If C is accessed
print(C) # no task will run, return cached result

#################################
# part D: If Task4 is changed
tg.update_task(E)(new_Task4)(C)

res = Output.compute() # compute new result:recompute new_Task4, Task5
```

See [Task-Graph](https://github.com/loopyme/Task-Graph) for more info about this tool.

## Conclusion

One of the biggest differences in Python from other languages is that the methods (like all other objects) are mutable, the number of tasks and their dependencies may not be known until run time, Task Graph can accommodate that and make computer not only delay-compute but also lazy-compute.

Task Graph is suitable for projects where the whole process and tasks are clear, able to obtain optimization effect especially when many of the intermediate results may be accessed by user, or when some of the tasks are mutable and may be tweaked at run time.