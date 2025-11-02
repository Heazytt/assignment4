# Graph Algorithms: SCC, Topological Sort, and DAG Shortest Paths

## Project Overview

This project implements and analyzes three fundamental graph algorithms on directed graphs:
1. **Strongly Connected Components (SCC)** using Tarjan's algorithm
2. **Topological Sorting** using Kahn's algorithm
3. **DAG Shortest/Longest Paths** for path analysis and critical path computation

The implementation includes comprehensive metrics collection and performance analysis across various graph structures.

---

## Table of Contents

- [Data Summary](#data-summary)
- [Algorithm Implementation](#algorithm-implementation)
- [Results and Metrics](#results-and-metrics)
- [Performance Analysis](#performance-analysis)
- [Conclusions and Recommendations](#conclusions-and-recommendations)
- [Usage](#usage)
- [Project Structure](#project-structure)

---

## Data Summary

### Dataset Overview

The project uses 9 test graphs organized into three size categories:

| Dataset | Size | Nodes (n) | Edges (m) | Density | Weight Model | Source | Description |
|---------|------|-----------|-----------|---------|--------------|--------|-------------|
| **Small Graphs** |
| s1_dag | S | 6 | 6 | 0.20 | edge | 0 | Simple DAG |
| s2_cycle | S | 6 | 7 | 0.23 | edge | 0 | Graph with cycles |
| s3_mixed | S | 8 | 10 | 0.18 | edge | 0 | Mixed structure |
| **Medium Graphs** |
| m1_mix | M | 12 | 17 | 0.13 | edge | 0 | Mixed with cycles |
| m2_dense | M | 15 | 45 | 0.21 | edge | 0 | Dense graph |
| m3_scc | M | 20 | 30 | 0.08 | edge | 0 | Multiple SCCs |
| **Large Graphs** |
| l1_sparse | L | 25 | 30 | 0.05 | edge | 0 | Sparse long chain |
| l2_medium | L | 50 | 120 | 0.05 | edge | 0 | Medium density |
| l3_dense | L | 100 | 500 | 0.05 | edge | 0 | Dense large graph |

### Density Formula Derivation

**Formula:** `density = m / (n × (n-1))`

#### For Directed Graphs

**Maximum Possible Edges:**
- Each vertex can have an edge to any other vertex (except itself)
- From vertex *i*, there can be edges to *(n-1)* other vertices
- Total vertices: *n*
- **Maximum edges:** `m_max = n × (n-1)`

**Density Definition:**
```
density = (actual number of edges) / (maximum possible edges)
density = m / (n × (n-1))
```

#### Value Range

- **Minimum:** `density = 0` (no edges, empty graph)
- **Maximum:** `density = 1` (complete graph, all possible edges present)

#### Example

For a graph with `n = 6` vertices:
- Maximum edges: `6 × 5 = 30`
- If `m = 6` edges: `density = 6/30 = 0.20` (20% dense)

#### Comparison with Undirected Graphs

For **undirected graphs**, the formula differs because edges are bidirectional:
```
density_undirected = 2m / (n × (n-1))
```

However, in this project all graphs are **directed**, so we use the standard formula without the factor of 2.

#### Interpretation

- **Sparse graph:** `density < 0.1` — few connections, linear behavior `O(n)`
- **Medium graph:** `0.1 ≤ density ≤ 0.3` — moderate connectivity
- **Dense graph:** `density > 0.3` — many connections, quadratic tendency `O(n²)`

**Weight Model:** All datasets use edge-based weights (weights assigned to edges, not nodes).

---

## Algorithm Implementation

### 1. Tarjan's SCC Algorithm

**Purpose:** Finds all strongly connected components in a directed graph.

**Complexity:**
- Time: O(n + m)
- Space: O(n)

**Key Metrics Tracked:**
- `dfsVisits`: Number of nodes visited during DFS
- `dfsEdges`: Number of edges explored during DFS

**Implementation Highlights:**
```java
- Single-pass DFS traversal
- Uses low-link values for SCC detection
- Stack-based component extraction
- Tracks visited nodes and explored edges
```

**Output:**
- List of strongly connected components
- Component index mapping for each node
- DFS traversal statistics

---

### 2. Kahn's Topological Sort

**Purpose:** Produces a linear ordering of vertices in a DAG where all edges point forward.

**Complexity:**
- Time: O(n + m)
- Space: O(n)

**Key Metrics Tracked:**
- `pushes`: Number of nodes added to the queue
- `pops`: Number of nodes removed from the queue

**Implementation Highlights:**
```java
- BFS-based approach using queue
- In-degree tracking for node ordering
- Works on condensation graph (DAG from SCC)
- Efficient with queue operations
```

**Output:**
- Topologically sorted node sequence
- Queue operation statistics

---

### 3. DAG Shortest Paths

**Purpose:** Computes shortest paths from source to all reachable nodes in a DAG.

**Complexity:**
- Time: O(n + m)
- Space: O(n)

**Key Metrics Tracked:**
- `relaxations`: Number of successful edge relaxations

**Implementation Highlights:**
```java
- Processes nodes in topological order
- Single-pass distance computation
- Optimal for DAGs (no need for Dijkstra/Bellman-Ford)
- Tracks path reconstruction via parent pointers
```

**Output:**
- Distance array from source
- Farthest reachable node
- Complete path to farthest node

---

### 4. DAG Longest Path (Critical Path)

**Purpose:** Finds the longest path in a DAG (critical path analysis).

**Complexity:**
- Time: O(n + m)
- Space: O(n)

**Implementation Highlights:**
```java
- Negates weights and applies shortest path
- Useful for project scheduling (CPM)
- Identifies critical path length
- Returns complete critical path sequence
```

**Output:**
- Longest path length
- Critical path node sequence

---

## Results and Metrics

### Expected Performance Characteristics

#### SCC Algorithm (Tarjan)

| Graph Type | DFS Visits | DFS Edges | Expected SCCs | Notes |
|------------|------------|-----------|---------------|-------|
| Pure DAG | = n | ≤ m | n (all singleton) | Each node is its own SCC |
| Single Cycle | = n | = m | 1 | All nodes in one SCC |
| Mixed | = n | ≤ m | 1 to n | Depends on cycle structure |

**Key Observations:**
- DFS always visits all nodes exactly once: `dfsVisits = n`
- Edge exploration: `dfsEdges ≤ m` (may terminate early on back edges)
- Number of SCCs inversely correlates with cycle density

---

#### Topological Sort (Kahn)

| Graph Structure | Pushes | Pops | Ratio |
|-----------------|--------|------|-------|
| Linear chain | n | n | 1.0 |
| Tree/Star | n | n | 1.0 |
| Dense DAG | > n | n | > 1.0 |

**Key Observations:**
- `pops = n` always (each node processed once)
- `pushes ≥ n` (minimum when no parallelism)
- `pushes > n` indicates multiple nodes with in-degree 0 simultaneously
- High push/pop ratio suggests high parallelism potential

---

#### DAG Shortest Paths

| Graph Density | Relaxations | Ratio to m | Notes |
|---------------|-------------|------------|-------|
| Sparse (chain) | ≈ m | ~1.0 | Most edges improve paths |
| Medium | 0.5m - 0.8m | 0.5-0.8 | Moderate improvements |
| Dense | < 0.5m | < 0.5 | Many redundant edges |

**Key Observations:**
- Relaxations indicate "useful" edges for shortest paths
- Lower relaxation ratio in dense graphs (redundant edges)
- Topological order ensures optimal single-pass processing

---

## Performance Analysis

### Bottleneck Analysis

#### 1. SCC Detection (Tarjan's Algorithm)

**Time Complexity:** O(n + m)

**Bottlenecks:**
- **DFS Recursion Depth:** Stack overflow risk for deep graphs
  - Mitigated by: Iterative DFS implementation (if needed)
  - Critical for: Long chains (e.g., l1_sparse with 25+ nodes)

- **Stack Operations:** Push/pop for each node
  - Impact: Minimal for modern hardware
  - Scales linearly with n

- **Edge Traversal:** Must explore all edges
  - Impact: Dominates for dense graphs
  - Dense graphs (m ≈ n²) are most expensive

**Structure Effects:**
- **Sparse graphs (m ≈ n):** Optimal performance, O(n)
- **Dense graphs (m ≈ n²):** Edge traversal dominates, O(n²)
- **Many small SCCs:** More overhead from component extraction
- **Few large SCCs:** Less overhead, faster processing

---

#### 2. Topological Sort (Kahn's Algorithm)

**Time Complexity:** O(n + m)

**Bottlenecks:**
- **In-degree Calculation:** O(m) preprocessing
  - Must scan all edges once
  - Unavoidable but single-pass

- **Queue Operations:** O(n) total pushes and pops
  - Extremely fast (ArrayDeque)
  - Negligible overhead

- **Edge Scanning:** O(m) during node processing
  - Each edge examined exactly once
  - Linear in edge count

**Structure Effects:**
- **Linear DAG:** Optimal, minimal queue size (1 node)
- **Star/Tree DAG:** Multiple simultaneous zeros, larger queue
- **Wide DAG:** High parallelism potential (many nodes ready)
- **Dense DAG:** More edge processing per node

**Performance Characteristics:**
```
Best case:  Linear chain → minimal memory, sequential processing
Worst case: Complete DAG → maximal queue size, all edges processed
```

---

#### 3. DAG Shortest Paths

**Time Complexity:** O(n + m)

**Bottlenecks:**
- **Topological Ordering:** Required preprocessing
  - O(n + m) from Kahn's algorithm
  - Can reuse if multiple queries

- **Distance Array Initialization:** O(n)
  - Negligible for small n
  - Cache-friendly sequential access

- **Edge Relaxation:** O(m) worst case
  - Actually O(relaxations) ≤ O(m)
  - Fewer in practice due to topological order

**Structure Effects:**
- **Sparse graphs:** Fast, fewer relaxations
  - Chain graph: Each edge useful, relaxations ≈ m
  - Minimal overhead

- **Dense graphs:** More edges, but diminishing returns
  - Many edges don't improve paths
  - Relaxation ratio < 0.5 for high density
  - Still O(m) but with lower constant

- **Graph depth:** Affects path lengths
  - Deep graphs: Longer critical paths
  - Wide graphs: More parallel paths

**Optimization Opportunities:**
- Early termination if target found
- Skip unreachable nodes (dist = ∞)
- Parallel relaxation (lock-free for disjoint paths)

---

### Effect of Graph Structure

#### Density Impact

| Density Range | SCC Time | Topo Time | SSSP Time | Notes |
|---------------|----------|-----------|-----------|-------|
| Sparse (< 0.1) | Fast | Fast | Fast | Linear behavior, O(n) |
| Medium (0.1-0.3) | Moderate | Moderate | Moderate | Mixed O(n + m) |
| Dense (> 0.3) | Slow | Slow | Moderate | Quadratic tendency |

**Key Insights:**
- SCC and Topo scale linearly with m (edge-bound)
- SSSP benefits from topological order (constant-factor speedup)
- Dense graphs require more memory (adjacency lists longer)

---

#### SCC Size Distribution Impact

| SCC Pattern | Effect on Condensation | Topo Performance | SSSP Performance |
|-------------|------------------------|------------------|------------------|
| All singleton (pure DAG) | No reduction, n' = n | Standard | Standard |
| Few large SCCs | Significant reduction | Faster | Faster |
| Many small SCCs | Moderate reduction | Moderate improvement | Moderate improvement |
| Single SCC (whole graph) | Maximal reduction, n' = 1 | Trivial | Single node |

**Performance Formula:**
```
Speedup = n / n'
where n' = number of SCCs in condensation graph
```

**Example:**
- Original graph: n=100, m=500
- After SCC: n'=10 SCCs, m'=50 edges
- Speedup: 10× for topological sort and shortest paths

---

#### Graph Topology Patterns

**Chain Graph (Linear):**
- SCC: n components, O(n) time
- Topo: Simple sequential ordering
- SSSP: All edges on critical path, max relaxations

**Star Graph (Hub and Spokes):**
- SCC: n components (if DAG)
- Topo: Hub first or last, parallel branches
- SSSP: Minimal relaxations (radial paths)

**Tree Graph:**
- SCC: n components
- Topo: Level-order traversal
- SSSP: Optimal, no redundant edges

**Dense Mesh:**
- SCC: Varies by cycle structure
- Topo: Many edges to process
- SSSP: Low relaxation ratio (redundant paths)

---

## Conclusions and Recommendations

### When to Use Each Algorithm

#### Tarjan's SCC Algorithm

**Use When:**
- Need to detect cycles in directed graphs
- Simplifying graph structure via condensation
- Analyzing graph connectivity patterns
- Preprocessing for DAG algorithms

**Best For:**
- Medium to large graphs with cycles
- Dependency analysis (build systems, package managers)
- Detecting deadlocks in concurrent systems

**Avoid When:**
- Graph is already known to be a DAG (unnecessary overhead)
- Graph is too large to fit in memory (requires complete traversal)

**Practical Recommendations:**
- Run once and cache results for multiple queries
- Use iterative DFS for very deep graphs (avoid stack overflow)
- Combine with condensation for significant speedup on cyclic graphs

---

#### Kahn's Topological Sort

**Use When:**
- Need a valid execution order for dependencies
- Processing DAG in linear order
- Detecting cycles (empty result = cycle exists)
- Task scheduling with precedence constraints

**Best For:**
- Small to medium DAGs
- Build systems and compilation order
- Course prerequisite planning
- Project scheduling

**Avoid When:**
- Graph has cycles (will fail or return partial ordering)
- Need all possible orderings (use DFS-based for backtracking)

**Practical Recommendations:**
- Check if `output.size() == n` to detect cycles
- Use for parallel task scheduling (queue size indicates parallelism)
- Combine with SCC: first find components, then sort condensation
- ArrayDeque is optimal (constant-time push/pop)

---

#### DAG Shortest/Longest Paths

**Use When:**
- Computing optimal paths in acyclic graphs
- Critical path method (CPM) for project management
- Longest path for scheduling with durations
- Single-source shortest paths where Dijkstra is overkill

**Best For:**
- Project scheduling (PERT/CPM)
- Task dependency analysis with costs
- Resource allocation with precedence
- Any DAG path optimization problem

**Avoid When:**
- Graph has cycles (undefined for cycles)
- Need all-pairs shortest paths (use Floyd-Warshall)
- Negative cycles present (though DAGs can't have cycles)

**Practical Recommendations:**
- **Critical Performance Factor:** Always process in topological order
- **Single-pass guarantee:** O(n + m) vs O(n log n + m) for Dijkstra
- **Longest path:** Negate weights, find shortest, negate result
- **Multiple queries:** Cache topological order, reuse for different sources
- **Memory optimization:** Use int[] for distances if weights are small

---

### Algorithm Comparison

| Algorithm | Time | Space | Input | Output | Main Use Case |
|-----------|------|-------|-------|--------|---------------|
| Tarjan SCC | O(n+m) | O(n) | Any directed | Components | Cycle detection |
| Kahn Topo | O(n+m) | O(n) | DAG | Linear order | Dependency resolution |
| DAG SSSP | O(n+m) | O(n) | Weighted DAG | Distance array | Path optimization |
| DAG Longest | O(n+m) | O(n) | Weighted DAG | Critical path | Project scheduling |

---

### Practical Recommendations

#### For Small Graphs (n < 100)

- **All algorithms perform instantly** (microseconds)
- Prioritize code simplicity over micro-optimizations
- Use for educational purposes and algorithm understanding
- Suitable for real-time applications

#### For Medium Graphs (100 ≤ n < 10,000)

- **Performance becomes measurable** (milliseconds)
- Cache topological order for multiple shortest-path queries
- Consider condensation to reduce graph size
- Monitor memory usage for dense graphs

#### For Large Graphs (n ≥ 10,000)

- **Performance critical** (seconds to minutes)
- **Mandatory:** Use SCC + condensation preprocessing
- Consider parallel implementations for independent components
- Use iterative DFS to avoid stack overflow
- Profile to identify bottlenecks (edge traversal vs. node processing)

#### For Very Large Graphs (n > 1,000,000)

- Consider external memory algorithms (graph too large for RAM)
- Streaming algorithms for single-pass processing
- Distributed graph processing (Apache Giraph, GraphX)
- Approximate algorithms if exact solution not required

---

### Optimization Strategies

#### 1. **Preprocessing with SCC**
```
Graph with cycles → Tarjan SCC → Condensation DAG → Topo Sort → SSSP
Speedup: O(n/n') where n' = number of SCCs
```

#### 2. **Cache Topological Order**
```
If running multiple shortest-path queries on same DAG:
- Compute topological order once: O(n+m)
- Reuse for each query: O(n+m) per query
- Savings: Avoid O(n+m) recomputation per query
```

#### 3. **Early Termination**
```
If looking for path to specific target:
- Stop relaxation when target is processed
- Savings: Up to 50% for targets in middle of topological order
```

#### 4. **Memory Optimization**
```
- Use primitive arrays (int[], long[]) instead of objects
- ArrayDeque instead of LinkedList for queue
- Avoid boxing/unboxing (use long instead of Long)
```

#### 5. **Parallel Processing**
```
- Independent SCCs can be processed in parallel
- Topological sort: process all zero-in-degree nodes simultaneously
- SSSP: relax edges from same topological level in parallel
```

---

### Pattern Selection Guide

| Scenario | Recommended Approach | Reasoning |
|----------|----------------------|-----------|
| Graph may have cycles | SCC → Condensation → Topo | Handles cycles, creates DAG |
| Known DAG | Direct Kahn Topo | Fastest, no SCC needed |
| Single shortest path query | Topo → SSSP | Standard pipeline |
| Multiple path queries | Cache Topo, reuse for SSSP | Avoid recomputation |
| Critical path analysis | Topo → Longest Path | Project scheduling |
| Dependency resolution | Kahn Topo only | Simple, efficient |
| Cycle detection | Tarjan SCC | Fast, returns components |

---

## Usage

### Prerequisites

- Java 11 or higher
- Maven 3.6+

### Building the Project

```bash
mvn clean compile
```

### Running All Graphs

```bash
mvn exec:java -Dexec.mainClass="org.example.App"
```

### Running a Specific Graph

Edit `src/main/java/org/example/App.java`:

```java

String selectedFile = "s1_dag.json"; 
```

Available datasets:
- Small: `s1_dag.json`, `s2_cycle.json`, `s3_mixed.json`
- Medium: `m1_mix.json`, `m2_dense.json`, `m3_scc.json`
- Large: `l1_sparse.json`, `l2_medium.json`, `l3_dense.json`

Then run:
```bash
mvn compile exec:java -Dexec.mainClass="org.example.App"
```

### Output Files

- **Metrics CSV:** `./out/metrics.csv` - Contains detailed performance metrics
- **Console Output:** Summary of each algorithm's results

---

## Project Structure

```
Assignment4/
├── pom.xml                           
├── README.md                         
├── data/                             
│   ├── s1_dag.json                  
│   ├── s2_cycle.json                
│   ├── s3_mixed.json                
│   ├── m1_mix.json                  
│   ├── m2_dense.json                
│   ├── m3_scc.json                  
│   ├── l1_sparse.json               
│   ├── l2_medium.json               
│   └── l3_dense.json                
├── src/
│   ├── main/
│   │   └── java/
│   │       ├── graph/
│   │       │   ├── common/
│   │       │   │   ├── Graph.java            
│   │       │   │   ├── JsonGraphReader.java 
│   │       │   │   └── WeightedDAG.java     
│   │       │   ├── scc/
│   │       │   │   ├── TarjanSCC.java       
│   │       │   │   └── Condensation.java    
│   │       │   ├── topo/
│   │       │   │   └── KahnTopo.java        
│   │       │   └── dagsp/
│   │       │       ├── DagShortestPaths.java 
│   │       │       └── DagLongestPath.java   
│   │       └── org/example/
│   │           └── App.java                  
│   └── test/
│       └── java/
│           └── graph/
│               ├── scc/
│               │   └── TarjanSCCTest.java    
│               ├── topo/
│               │   └── KahnTopoTest.java     
│               └── dagsp/
│                   ├── DagShortestPathsTest.java   
│                   └── DagLongestPathTest.java    
└── out/
    └── metrics.csv                   
```

---

## Testing

Run all tests:
```bash
mvn test
```

Run specific test:
```bash
mvn test -Dtest=TarjanSCCTest
mvn test -Dtest=KahnTopoTest
mvn test -Dtest=DagShortestPathsTest
mvn test -Dtest=DagLongestPathTest
```

---

## Example Output

```
[s1_dag.json] SCC: count=6, sizes=[1, 1, 1, 1, 1, 1]
[s1_dag.json] Topo: [5, 4, 3, 2, 1, 0]
[s1_dag.json] SSSP: sourceComp=c5, farthest=c0, dist=7, path=[c5->c4->c2->c1->c0]
[s1_dag.json] Critical: length=9, path=[c5->c3->c2->c1->c0]
```





