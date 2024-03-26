---
layout: distill
title: land permutation puzzle
description: tackling the land permutation puzzle - a quest for minimized borders
tags: algorithms java programming artificialintelligence problemsolving optimization
giscus_comments: true
date: 2024-03-23
featured: false


# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: The Land Permutation Puzzle
    # if a section has subsections, you can add them as follows:
    subsections:
       - name: Problem Definition
    #   - name: Example Child Subsection 2
  - name: Navigating Solutions - BFS, DFS, and IDDFS Explained
    subsections:
        - name: Breadth-First Search (BFS)
        - name: Depth-First Search (DFS)
        - name: Iterative Deepening Depth-First Search (IDDFS)
        - name: Comparassion
  - name: Implementation
    subsections:
        - name: Problem Modeling
        - name: Algorithms 
  - name: Results
  - name: Discussion
  - name: Further Information

# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }
---

## The Land Permutation Puzzle

In the intricate tapestry of computational challenges, the "Land Permutation Problem" stands out. It's a deceivingly straightforward puzzle derived from the artificial intelligence course in computer science degree, involving a matrix of lands owned by various proprietors. The goal is to minimize the number of different owners' borders. This territorial jigsaw requires both insightful analysis and strategic algorithmic application. To understand the intricacies and rules of this puzzle, explore the problem statement on [GitHub](https://github.com/jjginga/LandPermutationProblem).

_The problem as described in the README of the repository is a practical component of my studies._

#### Problem Definition

The task involves a given matrix of NxM houses representing a map of lands of equal dimension with K owners. The goal is to reduce the number of borders between lands of different owners to a number equal to or less than W. A border exists between two houses with different owners. Owners can be represented by colors, letters, or numbers, with numbers being used in our program.

| Component          | Description |
|--------------------|-------------|
| **Objective**      | test(s) ≤ W |
| **Initial State**  | s0 ∈ S      |
| **Possible Actions** | A = {(n, m, n', m') \| lands (n,m) and (n’,m’) are adjacent} |
| **Transition Model** | exe((s, (n, m, n', m'))) = s' where s' is the matrix resulting from exchanging the owner of (n,m) and (n', m'). |
| **Cost**           | -           |
| **Successors**     | succ(s) = {s' \| ∃a ∈ A, s' = exe(s, a)} |
| **Solution**       | sf ∈ S \| test(sf) |
| **Constraints**    | test(s’) ≤ test(s) |

This table succinctly encapsulates the problem's components, providing a clear framework for the "Land Permutation Problem." The constraints ensure that the number of borders in a successor state must not exceed that of the current state, guiding the search towards the objective.


## Navigating Solutions: BFS, DFS, and IDDFS Explained

This problem — minimizing the number of borders between lands with different owners — can be modeled as a graph traversal challenge. Each configuration of the land map, representing territories and their borders, can be considered a node (or vertex) in a graph. The initial state is the root node, and each possible action (e.g., swapping two adjacent territories to potentially reduce borders) leads to a successor state, which is a child node in the graph.

So, to solve this enigmatic puzzle we can use three classic algorithms, each with its own strengths and peculiarities. **Breadth-First Search (BFS)** is like casting a wide net, exploring all neighboring nodes at the current depth before diving deeper. It's methodical, ensuring no stone is left unturned, but its memory consumption can quickly become a concern as the breadth of exploration expands.

In contrast, **Depth-First Search (DFS)** opts for a more tunnel-vision approach, diving deep into the problem space one path at a time. Its memory footprint is lighter, making it nimble and efficient in certain mazes of complexity. However, its laser focus can sometimes be a drawback, as it might miss broader solutions found at shallower depths.

**Iterative Deepening Depth-First Search (IDDFS)**, then, marries the thoroughness of BFS with the memory efficiency of DFS. By incrementally deepening the search depth, IDDFS systematically covers the search space, ensuring completeness without the heavy memory burden associated with BFS.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/search_algorithms.jpg" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

## Breadth-First Search (BFS)

Breadth-First Search (BFS) systematically covers a graph by visiting adjacent vertices in successive levels.

#### How BFS Works:
1. **Initialization**: BFS begins with the root node (initial state) and explores all its neighbors (successor states).
2. **Queue Management**: It employs a queue to keep track of the frontier—the set of all nodes visible but not yet explored.
3. **Uniform Exploration**: At each step, BFS dequeues the next node from the frontier, examines it for a solution, and enqueues all its unvisited neighbors.
4. **Memory Consideration**: The major downside of BFS in complex puzzles like land permutation is its memory usage, which grows exponentially with the number of levels explored.

## Depth-First Search (DFS)

Depth-First Search delves into a graph, extending as far along each path as possible before backtracking to explore other branches.

#### How DFS Operates:
1. **Initialization**: Starting at the root, DFS pushes the initial state onto a stack.
2. **Stack Utilization**: It utilizes a stack to manage the nodes currently being explored.
3. **Depth Exploration**: DFS continuously explores down one branch until it hits a dead end, then backtracks to explore other branches.
4. **Memory Efficiency**: Its stack-based approach limits memory usage to the maximum depth of the search space, unlike BFS's breadth-based memory expansion.
5. **Solution Depth**: While efficient, DFS is not guaranteed to find the shortest solution in terms of moves or transformations.


## Iterative Deepening Depth-First Search (IDDFS)

Iterative Deepening Depth First Search (IDDFS) progressively deepens the reach of Depth-First Search, applying a breadth-like approach to depth-limited searches.

#### How IDDFS Functions:
1. **Depth Iteration**: Starting with a shallow depth limit, IDDFS performs a DFS within this limit. If no solution is found, the limit is increased, and the search is repeated.
2. **Balance of Efficiency**: This approach ensures that the memory advantages of DFS are maintained while achieving the breadth of coverage that BFS offers.
3. **Optimality**: Like BFS, IDDFS will find the optimal solution in terms of the shortest path to the goal.
4. **Revisitation**: Each iteration revisits nodes from previous depths, which is the trade-off for its balance between BFS and DFS advantages.


## Comparassion 

| Algorithm | Time Complexity | Space Complexity | Characteristics |
|-----------|-----------------|------------------|-----------------|
| **BFS**   | O( $$ b^d $$ )          | O( $$ b^d $$ )           | Complete and optimal; explores all neighbors at a given depth before moving deeper. High memory use due to storage of all nodes at the current level. |
| **DFS**   | O( $$ b^m $$ )          | O(bm)            | Not necessarily complete or optimal; explores as far as possible along a branch before backtracking. Lower memory use as it stores only a single path from the root to a leaf node, along with remaining unexplored siblings for each node on the path. |
| **IDDFS** | O( $$ b^d $$ )          | O(bd)            | Combines the advantages of BFS and DFS. Complete and optimal like BFS, but with the memory efficiency of DFS. Iteratively deepens, effectively performing a DFS to a specific depth, then increasing this limit iteratively. |

**Key:**
- **b**: branching factor (the average number of child nodes per node)
- **d**: depth of the shallowest solution
- **m**: maximum depth of the state space (potentially infinite)

This table elucidates the stark contrasts between these search algorithms, particularly in terms of their space and time efficiency. BFS and IDDFS share the same time complexity, reflecting their completeness and ability to find the optimal solution. However, the space complexity of IDDFS is dramatically lower, akin to DFS, making it an appealing choice for problems where space is a limiting factor and completeness is required.


This table elucidates the stark contrasts between these search algorithms, particularly in terms of their space and time efficiency. BFS and IDDFS share the same time complexity, reflecting their completeness and ability to find the optimal solution. However, the space complexity of IDDFS is dramatically lower, akin to DFS, making it an appealing choice for problems where space is a limiting factor and completeness is required.

## Implementation 

To solve this problem I splitted it into two parts:
- a "framework" designed to solve state space search problems where the objective is to minimize some goal that can be expressed numerically and can be reused for other problems. For that purpose we implemented a variety of search techniques including Breadth-First Search (BFS), Depth-First Search (DFS), and Iterative Deepening Depth-First Search (IDDFS), each encapsulated within its own class and implementing a common abstract search technique interface.
- a class that defines the state space of the Land Permutation problem and can be coupled with the "framework".

#### Problem Modeling

The **LandMap** class represents the state space, defining the layout of lands and borders that need to be manipulated. It includes methods for state evaluation (border count), generating successors, and performing swap operations which are fundamental actions of the search algorithms. These actions ensure that only valid moves towards the objective are considered, preventing an increase in the number of borders. 

The **countBorders** function is designed to iterate through each cell of the land map matrix and check for borders, where a border is defined as a side where adjacent cells have different values (representing different owners). This operation is O(N) where N is the number of cells in the matrix because it performs a constant amount of work for each cell: one comparison with its right neighbor and one with its bottom neighbor (except for cells on the rightmost and bottom edges, which have no neighbors in that direction). Without two loops or nested loops per cell that would increase the time complexity.

To **generate the successor states** the border count is tested before creating a new object to ensure the move is beneficial towards achieving the goal of minimizing borders, and this pre-validation step is crucial for maintaining the efficiency of the search algorithms. Each potential swap of adjacent territories in the land map is a candidate move that might lead to a new state; however, not all moves lead to a desirable outcome. By assessing the impact of a swap on the number of borders before actually instantiating a new LandMap object, the algorithm effectively filters out successor states that would not contribute to the solution. This pre-emptive check prevents the unnecessary creation and storage of state objects that do not bring the search any closer to the goal, which would otherwise waste computational resources and potentially slow down the search process due to increased memory usage and garbage collection overhead.

#### Algorithms

In the implementation of the search algorithms we went for a clean and modular approach, going for an algorithm design that are applicable to a broad range of problems beyond just the land permutation challenge. The Breadth-First Search (BFS) class utilizes a **Queue** data structure to ensure a level-order traversal. The Depth-First Search (DFS) class employs a **Stack** to dive deep into the search space, backtracking when necessary. This approach is typical for DFS's deep exploration strategy, which is both memory-efficient and adept at handling problems with vast search spaces.

Iterative Deepening Depth-First Search (IDDFS) marries the strengths of both BFS and DFS by combining the memory efficiency of DFS with the completeness of BFS. IDDFS systematically increases the depth limit, essentially performing a DFS to the current limit, then restarting with a deeper limit, allowing for a progressive exploration that is both thorough and space-conscious. Each of these classes is architected to encapsulate the search logic within its own module, using polymorphism through an interface that defines the structure of a search technique, making the algorithms interchangeable and reusable.

## Results

The following table presents the outcomes of applying different search algorithms to the several instances of the land permutation problem with the w1 goals. Each entry outlines the depth reached, the number of states generated, and the execution time recorded for the corresponding search technique and instance. It's important to note that an execution time marked as ">60s" indicates that the algorithm did not find a solution within the 60-second time limit. For these instances, the depth and generated states reflect the search progress made up to the point of timeout. These metrics are crucial for understanding the performance and efficiency of each algorithm under the constraints of time-limited execution.

| Instance | Algorithm                | Depth | Generated States | Execution Time |
|----------|--------------------------|-------|------------------|----------------|
| 1        | Breath First Search      | 3     | 16               | 0.0045 s       |
| 1        | Depth-First Search       | 3     | 10               | 0.0008 s       |
| 1        | Iterative Deep-First Search | 3     | 40               | 0.0024 s       |
| 2        | Breath First Search      | 2     | 9                | 0.0002 s       |
| 2        | Depth-First Search       | 2     | 5                | 0.0001 s       |
| 2        | Iterative Deep-First Search | 2     | 15               | 0.0004 s       |
| 3        | Breath First Search      | 3     | 5298084          | >60s           |
| 3        | Depth-First Search       | 159   | 7788             | 0.0065 s       |
| 3        | Iterative Deep-First Search | 3     | 5364301          | >60s           |
| 4        | Breath First Search      | 4     | 968              | 0.0007 s       |
| 4        | Depth-First Search       | 10    | 34               | 0.0001 s       |
| 4        | Iterative Deep-First Search | 4     | 1819             | 0.0013 s       |
| 5        | Breath First Search      | 6     | 146233           | 0.2841 s       |
| 5        | Depth-First Search       | 18    | 174              | 0.0002 s       |
| 5        | Iterative Deep-First Search | 6     | 352495           | 0.7041 s       |
| 6        | Breath First Search      | 9     | 14146290         | >60s           |
| 6        | Depth-First Search       | 77    | 679              | 0.0012 s       |
| 6        | Iterative Deep-First Search | 7     | 2336663          | >60s           |
| 7        | Breath First Search      | 3     | 609063           | >60s           |
| 7        | Depth-First Search       | 216   | 5183             | 0.0059 s       |
| 7        | Iterative Deep-First Search | 4     | 8387805          | >60s           |


## Discussion

The execution of search algorithms on the land permutation problem has demonstrated varied outcomes, which reveal the strengths and weaknesses inherent in each method. The Breath First Search (BFS), noted for its exhaustive level-by-level exploration, has proven effective in shallower instances where the breadth of potential states remains manageable. However, its performance notably diminishes as the complexity of the state space increases, often resulting in timeouts. This indicates that while BFS is thorough, its resource consumption makes it less viable for problems with expansive search trees.

Depth-First Search (DFS), on the other hand, showcased its ability to reach deeper into the search space with a significantly lower number of generated states, emphasizing its depth-focused approach. Its memory efficiency shines in instances where the solution lies far from the root, but this comes with the trade-off of potentially overlooking nearer solutions due to its path-focused traversal. The variation in depth and generated states between DFS and BFS highlight the impact of the algorithms' differing exploration strategies.

Iterative Deepening Depth-First Search (IDDFS) strikes a balance between the two, ensuring completeness while retaining memory efficiency. Although IDDFS also faced timeouts in more complex instances, it consistently covered more ground, as indicated by the greater depth reached before the time cap. The iterative nature of IDDFS allows it to comb through the state space methodically, which is particularly beneficial when the solution's depth is unknown, making it a robust choice for a wide range of scenarios.

The results table succinctly captures the essence of these algorithms' performances and provides a clear comparison that aids in selecting the appropriate approach for different instances of the problem. The trade-offs between time complexity, space complexity, and solution optimality become apparent, informing the decision-making process for algorithm selection in problem-solving.

## Further information

[Geeks for Geeks - BFS](https://www.geeksforgeeks.org/breadth-first-search-or-bfs-for-a-graph/)

[Geeks for Geeks - DFS](https://www.geeksforgeeks.org/depth-first-search-or-dfs-for-a-graph/)

[Geeks for Geeks - IDDFS](https://www.geeksforgeeks.org/iterative-deepening-searchids-iterative-deepening-depth-first-searchiddfs/)

Russell, S., & Norvig, P. (2001). _Artificial Intelligence: A Modern Approach (3rd ed)_. [AIMA](https://aima.cs.berkeley.edu/)