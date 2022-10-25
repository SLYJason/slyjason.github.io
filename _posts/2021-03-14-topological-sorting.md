---
layout: post
title: "Topological Sorting"
subtitle: "graph theory"
date: 2021-03-14
author: "Luyi"
header-img: "img/in-post/2021-03-14-topological-sorting/post-bg.jpg"
tags:
    - Algorithms
---

### Real Work Challenge
Many real world problems can be modelled as a graph with directed edges where some event must occur before others. 
Things like school class prerequisites, program dependencies, event scheduling, assembly instructions etc. 
For example, considering the following scenario: a task scheduling system have `11` tasks which is labeled from `0` to `10`, you got an assignment need to finish the task `9` and would like to know in order to finish it, you need to finish its dependencies first. In such case, task `6` and `7` needs to be finished, then in order to finish task `6` task `3` need to finished etc. Same with task `7` need to get its dependencies task `3` and task `5` finished first. 
Therefore, we have task order `2 -> 0 -> 1 -> 4 -> 5 -> 3 -> 6 -> 7 -> 9`, the dependency paths are colored with blue:
![Figure_1](/img/in-post/2021-03-14-topological-sorting/Figure_1.svg)

### Topological Sort Definition
By looking at the above dedicated example, we have a sense now about what is ordering looks like in a dependency graph which is called **Topological Sorting**. 
From [Wikipedia](https://en.wikipedia.org/wiki/Topological_sorting): in computer science, a topological sort or topological ordering of a directed graph is a linear ordering of its vertices such that for every directed edge `uv` from a vertex `u` to vertex `v`, `u` comes before `v` in the ordering. Precisely, a topological sort is a graph traversal in which each node v is visited only after all its dependencies are visited.
Worth to note here, this graph should be **[directed acyclic graph (DAG)](https://en.wikipedia.org/wiki/Directed_acyclic_graph)**, i.e. has directed edges and no cycles in the graph. And also note the topological ordering is not **unique**, i.e. can have multiple ordering result.

### Approach 1 - DFS
#### How it works
Using depth search fist (DFS) will explore the node as deep as possible, which means the end node of the DFS call stack has the most dependencies. Thus, maintaining a states that specify which node is **unvisited**, **visiting** and **visited** using an array:
1. `unvisited`: node has not been reached by DFS.
2. `visiting`: node is current in the DFS call stack, but not return to the callback step.
3. `visited`: node already visited, no need to search again.

For the following example, start from node `1`, at the end of DFS, node `9` will be `visited`. And later in the callback of DFS visiting node `6`, `3`, `0` will be changed from `visiting` to `visited`.
![Figure_2](/img/in-post/2021-03-14-topological-sorting/Figure_2.svg)

Then the algorithms runs as following:
1. Pick an `unvisited` node called `u` and search its neighbors `v` using DFS. There has 3 conditions when visit `v`:
    * If node `v` is `unvisited` then we search `v` using DFS and if all DFS complete callback to `u`. And in the callback step, mark the node `v` as `visited` and add it to the topological sorting result.
    * If node `v` is `visiting` then we find a cycle, so the topological sorting does not exist.
    * If node `v` is `visited` then no need to search again.
2. If all neighbors `v` of node `u` are all `visited`, we can mark `u` is `visited` too and add it into the result.

#### Implementation
In the following implementation, `nums` is total tasks, `edges` is an array of edge array and each `edge[i, j]` means `j -> i`. Worth to note here if the graph has cycles, we will return an empty array.
```java
    public int[] topologicalSorting(int nums, int[][] edges) {
        // step 1: build the graph.
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < nums; i++) graph.add(new ArrayList<>());
        for (int[] prerequisite : edges) {
            graph.get(prerequisite[1]).add(prerequisite[0]);
        }

        // step 2: dfs + backtracking.
        int[] states = new int[nums]; // 0: unvisited, 1: visiting, 2: visited.
        LinkedList<Integer> res = new LinkedList<>();
        for (int i = 0; i < nums; i++) {
            if (hasCyclic(i, states, graph, res)) return new int[0];
        }

        return res.stream().mapToInt(n -> n).toArray();
    }

    /**
     * A helper function to determine the graph has a cycle or not.
     * @param curr current starting node.
     * @param states maintain each node unvisited, visiting, visited states.
     * @param graph the DAG graph representation.
     * @param res the final topological sort result.
     * @return True has a cycle, False otherwise.
     */
    private boolean hasCyclic(int curr, int[] states, List<List<Integer>> graph, LinkedList<Integer> res) {
        if (states[curr] == 1) return true;
        if (states[curr] == 2) return false;
        states[curr] = 1;

        for (int neighbor : graph.get(curr)) {
            if (hasCyclic(neighbor, states, graph, res)) return true;
        }
        states[curr] = 2;
        res.addFirst(curr); // add the nodes in reverse order
        return false;
    }
```

### Approach 2 - BFS
#### How it works
Use breadth search first (BFS) to explore the neighbor nodes first in the graph we need to know the neighbor node dependency which called `in_degree`. Consider the following example, task `3` has two dependencies `1` and `2` so `in_degree = 2`, similar to task `4` has only 1 dependency so `in_degree = 1`.
![Figure_3](/img/in-post/2021-03-14-topological-sorting/Figure_3.svg)

So overall we need to know each node `in_degree` and remove the dependencies as we do BFS, here is the algorithm:
1. Build the `in_degree` of each node in the graph using the edges and push the nodes has 0 `in_degree` into a queue.
2. Pick any node from the queue, adding this node to the topological sorting result. Searching the neighbor nodes and the `in_degree` of the neighbor node should minus 1.
3. Push any neighbor node that has 0 `in_degree` into the queue.
4. Continue the process until the queue is empty.

#### Implementation
```java
    public int[] topologicalSorting(int nums, int[][] edges) {
        // step 1: build the graph and in_degree.
        List<Integer> res = new ArrayList<>();
        List<List<Integer>> graph = new ArrayList<>();
        int[] in_degree = new int[nums];
        for (int i = 0; i < nums; i++) graph.add(new ArrayList<>());
        for (int[] p : edges) {
            in_degree[p[0]]++;
            graph.get(p[1]).add(p[0]);
        }

        // step 2: bfs.
        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < nums; i++) {
            if (in_degree[i] == 0) queue.offer(i); // in_degree with 0 added to the queue, means this value has zero dependency.
        }
        while (!queue.isEmpty()) {
            int task = queue.poll();
            res.add(task);
            for (int neighbor : graph.get(task)) {
                if (--in_degree[neighbor] == 0) {
                    queue.offer(neighbor);
                }
            }
        }
        return res.size() == nums ? res.stream().mapToInt(n -> n).toArray() : new int[0];
    }
```

### Conclusion
Topological sorting is very powerful graph algorithm and there are many applications use it. This article gives the core concepts of how it works using both DFS and BFS in a directed acyclic graph (DAG) graph. The real world scenarios maybe more complicated so encourage you continue to learn.

### Reference
Here is several useful links:
* [Youtube: Topological Sort](https://www.youtube.com/watch?v=eL-KzMXSXXI&ab_channel=WilliamFiset).
* [LeetCode: Course Schedule](https://leetcode.com/problems/course-schedule/).
* [LeetCode: Course Schedule II](https://leetcode.com/problems/course-schedule-ii/).