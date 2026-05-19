# Parallelized Sudoku Solvers

![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![C++](https://img.shields.io/badge/C++-00599C?style=for-the-badge&logo=c%2B%2B&logoColor=white)
![Parallel Computing](https://img.shields.io/badge/Parallel_Computing-pthreads-informational?style=for-the-badge)

A comparative study of three Sudoku solving algorithms, each implemented in both **sequential** and **parallelized** forms to benchmark how concurrency affects performance across varying board sizes, difficulty levels, and algorithms. 

- Board Sizes: **9×9**, **16×16**, and **25×25**
- Difficulties: **Easy**, **Medium**, and **Hard**

📄 [Read the Report](attachments/Parallelized_Sudoku_Solvers.pdf) 📁 [View All Input Files](data/)

### Algorithms

| # | Algorithm | Language | Sequential | Parallelized |
|---|-----------|----------|------------|--------------|
| 1 | [Brute Force (Backtracking)](#1-brute-force) | Java | ✅ | ✅ |
| 2 | [Dancing Links (Algorithm X)](#2-dancing-links) | Java | ✅ | ✅ |
| 3 | [Propagation-Cross-Search](#3-propagation-cross-search) | C++ | ✅ | ✅ |

### Key Challenges

- **Thread Overhead vs. Speedup:** Parallelizing an already-fast sequential solver can introduce more overhead than benefit, especially on smaller boards.
- **Synchronization:** Managing shared state, avoiding deadlocks, and preventing thread conflicts across backtracking-based search trees.
- **Algorithm-Specific Parallelism:** Each technique required a different strategy for distributing work across threads.

## Prerequisites

| Tool | Check | Purpose |
|------|-------|---------|
| Java | `java -version` | Brute Force & Dancing Links |
| g++ (C++17) | `g++ --version` | Propagation-Cross-Search |
| Clang | `clang --version` | Propagation-Cross-Search |
| Git | `git --version` | Cloning the repo |

<details>
<summary><strong>Installation Instructions</strong></summary>

#### g++ (GCC)
- **Linux:** `sudo apt-get install g++`
- **macOS:** `xcode-select --install`
- **Windows:** Install [MinGW](https://www.mingw-w64.org/) or [Cygwin](https://cygwin.com/)

#### Git
- **Linux:** `sudo apt-get install git`
- **macOS:** `brew install git`
- **Windows:** [Git for Windows](https://gitforwindows.org/)

#### Java
- **Linux:** `sudo apt-get install default-jdk`
- **macOS:** `brew install openjdk`
- **Windows:** [Oracle JDK](https://www.oracle.com/java/technologies/downloads/) or [AdoptOpenJDK](https://adoptium.net/)

#### Clang
- **Linux:** `sudo apt-get install clang`
- **macOS:** `brew install llvm`
- **Windows:** [LLVM](https://releases.llvm.org/download.html)

</details>

#### Clone the Repo

```sh
git clone https://github.com/johnmichael-kane/Parallelized-Sudoku-Solvers.git
cd Parallelized-Sudoku-Solvers
```

## Algorithms Implemented

### 1. Brute Force

A backtracking search that tries every possible number placement, undoing and retrying when a conflict is found.

**Parallelization Strategy:** Multiple threads each explore separate subtrees of the search space, with the first thread to find a valid solution triggering a shared completion flag.

- [Sequential & Parallelized Branch](BruteForce/)
- Adapted: [bryanesmith/Sudoku-solver](https://github.com/bryanesmith/Sudoku-solver/blob/master/SudokuPuzzle.cpp)

---

### 2. Dancing Links

An implementation of Donald Knuth's Algorithm X using the Dancing Links (DLX) technique to solve Sudoku as an **exact cover problem**. Each constraint maps to a matrix column; each legal number placement maps to a matrix row.

**Parallelization Strategy:** An initial BFS generates `n` partial board states, one per thread. Each thread independently runs the full DLX solver on its board; the first to find a solution sets a shared flag and all threads return.

- [Sequential Implementation](DancingLinks/sequential)
- [Parallelized Implementation](DancingLinks/parallel)
- Adapted: [gkaranikas/dancing-links](https://github.com/gkaranikas/dancing-links/tree/master)

#### Key Source Files

| File | Role |
|------|------|
| `Cell.java` | Represents a grid cell (row, column, assigned value) |
| `ColumnObject.java` | DLX column node with cover/uncover operations |
| `DancingLinkObject.java` | DLX node with directional links and column reference |
| `DancingLinkSolver.java` | Recursive backtracking search; selects column with fewest possibilities first |
| `ExactMatrix.java` | Converts Sudoku grid to exact cover matrix |
| `Sudoku.java` | Entry point (reads input, initializes matrix, invokes solver) |

#### Run (Sequential)

```sh
cd DancingLinks/sequential
javac Sudoku.java
java Sudoku <FILE_NAME>
```

#### Run (Parallel)

```sh
cd DancingLinks/parallel
javac Sudoku.java
java Sudoku <FILE_NAME> <NUM_THREADS>
```

<details>
<summary><strong>Sample Output (16×16 Solved Board)</strong></summary>

```
Solved Sudoku:
Time taken: 47 milliseconds

| 01  03  16  05 | 14  02  11  15 | 07  06  08  04 | 10  09  13  12 |
| 14  07  12  11 | 13  09  08  10 | 03  02  01  16 | 04  06  05  15 |
...
```

</details>

<details>
<summary><strong>Sample Output (25×25 Solved Board)</strong></summary>

```
Solved Sudoku:
Time taken: 283 milliseconds

| 23  18  20  08  15 | 22  21  06  17  03 | 13  09  05  11  10 | 24  19  14  07  04 | 12  25  16  01  02 |
...
```

</details>

---

### 3. Propagation-Cross-Search

A C++ solver combining three progressive techniques before falling back to search:

1. **Constraint Propagation:** Eliminate candidates using row/column/box constraints
2. **Cross-Referencing:** Identify forced placements by intersecting possibilities
3. **Depth-First Search:** applied only when propagation can't resolve remaining cells

**Parallelization Points:** Candidate checks, possible-position scanning, and DFS (on hard boards) are conditionally parallelized across the `Game` and `PossibleGrid` components.

- [Sequential Implementation](PropagateCrossSearch/sequential)
- [Parallelized Implementation](PropagateCrossSearch/parallel)
- Adapted: [anthemEdge/Sudoku-Solver](https://github.com/anthemEdge/Sudoku-Solver)

#### Run

```sh
# Navigate to your chosen implementation
cd PropagateCrossSearch/sequential   # or /parallel

# Compile
g++ -std=c++17 -g -o SudokuSolver SudokuSolver.cpp Game.cpp Grid.cpp PossibleGrid.cpp

# Run (follow console prompts for filename and thread count)
./SudokuSolver
```

## Troubleshooting

- Navigate directories with `cd <folder>` and `cd ..`
- If you hit C++ compilation errors, confirm g++ supports C++17: `g++ --version`
- On Windows with pthreads issues, verify your MinGW/Cygwin threading flags

## Improvement Roadmap

- [ ] Standardize input format across all three algorithms
- [ ] Refactor for consistent code structure and unified Sudoku output formatting
- [ ] Optimize solver efficiency and reduce thread overhead
- [ ] Expand benchmarking with recorded sequential vs. parallel runtime comparisons
- [ ] Update report to reflect post-optimization results
- [ ] Clean up and consolidate branch structure

## Contributors

| Name | GitHub |
|------|--------|
| Soleil Cordray | [@soleil-cordray](https://github.com/soleil-cordray) |
| JohnMichael Kane | [@johnmichael-kane](https://github.com/johnmichael-kane) |
| Abdullah AL Hinaey | [@AbdullahALX](https://github.com/AbdullahALX) |
