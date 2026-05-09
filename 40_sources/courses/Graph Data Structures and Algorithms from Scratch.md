---
tags:
  - algorithms
  - graph
  - data-structures
feature: 
type: course
author: "[[Matthew Singer]]"
source: 
---
# Graph Data Structures and Algorithms from Scratch

## Graph Theory

a graph is a collection of **nodes** (vertices) connected by **edges**
not every pair of vertices would have an edge between them
a graph with 0 edges is called disconnected
**undirective graph** where edges don't have direction
**directive graph** where edges have direction
**weighted graphs** is when each edge has a weight 
exists also **directed wighted graphs**
a **neighbour** of a node is something where you can get from one node to another

A graph G is an ordered pair of edges and vertices (V, E)
V is the set of vertices
E is the set of Edges:
- edges are described as (u, v)
- there being an edge from u to v is the same as saying v is neighbour of u
${\emptyset}$  is the empty set (${\emptyset}, {\emptyset}$) is the empty graph 

**subgraph** is a part of a graph
$H = (V', E')$
$V' \subseteq V$
$E' \subseteq E$

a **path** is a subgraph where all the vertices are connected
a path is a list of edges where at the end of every pair the next pair start with the same edge

a **cycle** is a subgraph that has a path from v to v for all v in V

**self edges** is when a vertex is connected to itself by an edge

## Graph Data Structure

##### Adjacency Matrix
you have a matrix that is the cardinality of V (number of edges $|V|$ )
Make a $|V|$ by $|V|$ matrix, called $A$
$\forall (v_{i,}v_{j)}\in E$, set $A_{[i][j]}= 1$

$$
\begin{array}{ccccccc}
  &  & A & \text{---} & C & \text{---} & B \\
  &  &   &             & | &            & | \\
  &  &   &             & D & \text{---} &   \\
  &  &   &             & | &            &   \\
  &  &   &             & E &            &   \\
\end{array}
$$



$$
\begin{array}{c|ccccc}
   & A & B & C & D & E \\ \hline
A & 0 & 0 & 1 & 0 & 0 \\
B & 0 & 0 & 1 & 1 & 0 \\
C & 1 & 1 & 0 & 1 & 1 \\
D & 0 & 1 & 1 & 0 & 0 \\
E & 0 & 0 & 1 & 0 & 0
\end{array}


$$
This matrix represents where you can go in 1 step
if I multiply this matrix for itself than I will get a matrix where all the ones represented places you could get to in 2 steps
##### Adjacency List
make a list of $|V|$ lists (usually linked list)
add each edge $v_i,v_j$ to the lists $A[j]$
```
A: [C]
B: [C, D]
C: [A, B, D, E]
D: [B, C]
E: [C]
```

###### Programmatically
for a matrix create a 0 filled matrix of the size required.
than for each pairs (a,b) go to `matrix[a][b]` and set 1

##### Space complexity
an **adjacency matrix** is inefficient for sparse (doesn't have a lot of edges) graphs.
the **adjacency matrix** takes $n^2$ where $n$ is the number of edges
the **adjacency list** takes $n$

##### Time complexity
to find if two vertices are connected in the list must iterate all the list while the matrix it's an instant look up, but if I want to produce a list of all neighbours of a specific vertex **adjacency list** already has the answer.

## Graph Properties
### Degree
the degree of a vertex is the number of edges containing it
in a directed graph *in-degree* is the number of edges coming in, and *out-degree* is the number of edges going out
the minimum degree and maximum degree of a graph are the degree of the node with the smallest or larges degree
###### Programmatically
to find the *in-degree* programmatically in a adjacency list we must find the number of occurrences of the vertex we are looking for, in the adjacency matrix we should count how many 1s are in the index of the vertex we are looking for for every row.
to find the *out-degree* in an adjacency list we count the size of the array at the given index
in an adjacency matrix we count the 1s of the array at the given index

### Connectedness
a graph is connected if there a path from every vertex to every other vertex

###### Programmatically

### Cut edge
an edge that, if removed, disconnects the graph
an edge that is not in a cycle

### Completeness
a graph is complete if there is an edge from every vertex to every vertex
###### Programmatically
for every row in the adjacency list contains all the other vertices

### Cyclic
a graph is cyclic if it has a cycle in it

### Trees
a graph is a tree if it has no cycles and is connected

## Search Algorithm
- finding files
- navigation
- currency conversion
#### Breadth-first search
goes level by level
- will always find the shortest path
##### BFS on weighted graphs
- "shortest path" means something different (sum of the )
- can convert weighted graphs to non-weighted
- Dijkstra's algorithm
##### Dijkstra's algorithm
- keep track of shortest know distance to each node
- from the node woth thw shortest distance from the start point, re-calculate everything next to it (if this node has a shorter path, remember that)
- once you have found the target, trace the shortest path
used queue FIFO
#### depth-first search
one path to the end and back
used stack LIFO
[[graphdatastructuresandalgorithmsfromscratchslides1702303715158.pdf]]

### NetworkX
python library
https://networkx.org/en/

### JGraphT
https://jgrapht.org/

#### Topological Sort
creates an ordering of the vertices in a directed graph
before a vertex appears all of its ancestors (a node v's ancestor is any node that appear before v in any path)
###### Kahn's Algorithm
while there are still nodes left to process:
- initialise a queue
- for all nodes with in-degree 0:
	- add node to queue
	- remove all edges from that node
	- mark those nodes as processed
- 