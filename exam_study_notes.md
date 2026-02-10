# CSE 160 Exam 1 Study Notes 📚

## Table of Contents
1. [OpenCL Execution Model](#1-opencl-execution-model-)
2. [Thread Indexing & Math](#2-thread-indexing--math-)
3. [Memory Hierarchy](#3-memory-hierarchy-)
4. [Synchronization & Barriers](#4-synchronization--barriers-)
5. [Tiled Matrix Multiplication](#5-tiled-matrix-multiplication-)
6. [Memory Coalescing](#6-memory-coalescing-)
7. [Performance Analysis](#7-performance-analysis-)
8. [Common Patterns](#8-common-patterns-)
9. [Quick Reference Formulas](#9-quick-reference-formulas-)
10. [Common Pitfalls](#10-common-pitfalls-)

---

## 1. OpenCL Execution Model 🎯

### Understanding the Hierarchy

Think of OpenCL execution like organizing a massive project:

**Work-Items (Individual Workers)**
- The smallest unit - like a single person on a team
- Each one runs the same code but on different data
- Has a unique ID so it knows which data to process
- Example: If you're processing 1000 pixels, you might have 1000 work-items, each handling one pixel

**Work-Groups (Teams)**
- A collection of work-items that work together
- They can share a common workspace (local memory)
- They can communicate and synchronize with each other
- Typically 64, 128, or 256 work-items per work-group
- Example: Your 1000 work-items might be organized into 8 work-groups of 128 workers each

**NDRange (The Entire Workforce)**
- The complete grid of all work-items
- Can be 1D (a line), 2D (a grid), or 3D (a cube)
- Total size = all the work that needs to be done

### Key Functions - What Each One Tells You

```c
get_global_id(0)    // "What's my unique ID among ALL workers?"
                    // Returns: 0, 1, 2, ..., N-1
                    
get_local_id(0)     // "What's my position within MY team?"
                    // Returns: 0, 1, 2, ..., (team_size - 1)
                    
get_group_id(0)     // "Which team am I on?"
                    // Returns: 0, 1, 2, ..., (num_teams - 1)
                    
get_local_size(0)   // "How many people are on my team?"
                    // Returns: the work-group size (e.g., 128)
                    
get_global_size(0)  // "How many total workers are there?"
                    // Returns: total number of work-items
                    
get_num_groups(0)   // "How many teams total?"
                    // Returns: number of work-groups
```

### Why This Matters

Understanding these IDs is crucial because:
- **Global ID** tells you WHICH piece of data you should process
- **Local ID** helps you coordinate with your team members
- **Group ID** identifies your team's location in the grid

---

## 2. Thread Indexing & Math 📊

### The Golden Formula (MOST IMPORTANT!)

```c
global_id = get_group_id(dim) × get_local_size(dim) + get_local_id(dim)
```

**What this means in plain English:**
- Which team am I on? (`get_group_id`)
- How big is each team? (`get_local_size`)
- What's my position in my team? (`get_local_id`)
- So my overall position = (team number × team size) + my position in team

### Example Walkthrough

Let's say you have:
- Work-group size = 16 (each team has 16 people)
- You're in work-group 3 (team 3)
- Your local ID is 5 (you're the 6th person on your team, counting from 0)

Your global ID = 3 × 16 + 5 = 48 + 5 = **53**

This means you process element 53 of the array!

### 1D Indexing (Arrays)

```c
// Simple version - OpenCL calculates for you
int i = get_global_id(0);
data[i] = ...;  // Process element i

// Manual version - you calculate it
int i = get_group_id(0) * get_local_size(0) + get_local_id(0);
data[i] = ...;  // Same thing
```

### 2D Indexing (Matrices/Images)

For 2D data, you need both a row and column:

```c
// Row comes from dimension 1 (Y dimension)
int row = get_group_id(1) * get_local_size(1) + get_local_id(1);

// Column comes from dimension 0 (X dimension)  
int col = get_group_id(0) * get_local_size(0) + get_local_id(0);

// Convert 2D position to 1D array index (row-major order)
int index = row * width + col;
```

**Why row-major?** Because in C, matrices are stored row-by-row in memory:
```
Matrix:     Memory layout:
[1 2 3]     [1, 2, 3, 4, 5, 6, 7, 8, 9]
[4 5 6]
[7 8 9]
```

To access element at (row=1, col=2):
- Skip entire first row: `row * width = 1 * 3 = 3 elements`
- Then move over `col = 2` more
- Index = 3 + 2 = 5, which is element `6` ✓

### Calculating Grid Dimensions

**Problem:** I have 7200 elements to process. Each work-group has 128 work-items. How many work-groups do I need?

**Solution:**
```c
num_groups = ceil(7200 / 128) = ceil(56.25) = 57
```

**In C code (no ceiling function):**
```c
num_groups = (7200 + 128 - 1) / 128;  // Integer math gives ceiling
```

**If each work-item processes K elements:**
```c
total_work_items_needed = N / K;
num_groups = (total_work_items_needed + local_size - 1) / local_size;
```

Example: 7200 elements, each work-item processes 4, local_size = 128
```c
work_items_needed = 7200 / 4 = 1800
num_groups = (1800 + 128 - 1) / 128 = 1927/128 = 15 (integer division)
```

---

## 3. Memory Hierarchy 💾

### The Speed Pyramid

```
    FASTEST
    ┌─────────────┐
    │  Private    │  Each work-item's registers
    │  Memory     │  Tiny but blazing fast
    └─────────────┘
         ↓
    ┌─────────────┐
    │   Local     │  Shared by work-group
    │   Memory    │  ~100x faster than global
    │ (__local)   │  16-64 KB per work-group
    └─────────────┘
         ↓
    ┌─────────────┐
    │   Global    │  Accessible by all
    │   Memory    │  Huge but slow
    │ (__global)  │  Main data storage
    └─────────────┘
         ↓
    ┌─────────────┐
    │  Constant   │  Read-only
    │   Memory    │  Cached
    │(__constant) │  For parameters
    └─────────────┘
    SLOWEST
```

### How to Use Each Type

**Private Memory (Automatic)**
```c
float temp = 5.0f;  // Automatically stored in registers
int count = 0;       // Super fast, but only YOU can see it
```

**Local Memory (Declare with `__local`)**
```c
__local float shared_data[256];  // Shared by all work-items in group
// All team members can read/write this
```

**Global Memory (Pass as pointer)**
```c
__kernel void myKernel(__global float *data) {
    // Everyone can access this, but it's slow
}
```

**Constant Memory (Read-only parameters)**
```c
__kernel void myKernel(__global float *data, __constant int *params) {
    // params are the same for everyone, cached for speed
}
```

### Memory Size Limits (Know These for Exam!)

Typical constraints:
- **Local memory per work-group:** 16-64 KB
- **Work-items per work-group:** Max 256-1024
- **Work-groups per compute unit:** Max 8-40

### Calculating Memory Usage

**Example:** Will this kernel work?
```c
// Work-group size: 1024
// Each work-item loads 10 floats into local memory
// Limit: 64 KB per work-group

Bytes per work-item = 10 floats × 4 bytes/float = 40 bytes
Total = 1024 work-items × 40 bytes = 40,960 bytes = 40 KB

40 KB < 64 KB ✓ YES, it will work!
```

---

## 4. Synchronization & Barriers 🚧

### What is a Barrier?

A barrier is a **checkpoint** where all work-items in a work-group must wait until everyone arrives before anyone can continue.

Think of it like a team meeting: Nobody starts the next task until everyone finishes the current one.

### Why Do We Need Barriers?

**The Problem:** Race conditions!

```c
// Thread 0 writes
shared[0] = 100;

// Thread 1 reads
float val = shared[0];  // What value? Could be anything! 😱
```

**The Solution:** Barrier!

```c
// Thread 0 writes
shared[0] = 100;

barrier(CLK_LOCAL_MEM_FENCE);  // Everyone wait!

// Thread 1 reads
float val = shared[0];  // Now guaranteed to be 100 ✓
```

### When Do You Need a Barrier?

**Simple Rule:** If one work-item writes to shared memory and another reads it, you need a barrier in between.

Ask yourself:
1. Does work-item A write to shared memory?
2. Does work-item B read that same location?
3. Could B read before A finishes writing?

If yes to all three → **You need a barrier!**

### Barrier Syntax

```c
barrier(CLK_LOCAL_MEM_FENCE);   // For __local memory
barrier(CLK_GLOBAL_MEM_FENCE);  // For __global memory
```

### The Classic Tiling Pattern

```c
for (int m = 0; m < numTiles; m++) {
    // Step 1: Everyone loads their piece of data
    shared[ty][tx] = global_data[...];
    
    // BARRIER: Wait for all loads to finish!
    barrier(CLK_LOCAL_MEM_FENCE);
    
    // Step 2: Everyone computes using the shared data
    for (int k = 0; k < TILE_SIZE; k++) {
        sum += shared[ty][k] * shared[k][tx];
    }
    
    // BARRIER: Wait before loading next tile!
    barrier(CLK_LOCAL_MEM_FENCE);
}
```

**Why two barriers?**
1. First barrier: Make sure everyone finishes loading before anyone starts reading
2. Second barrier: Make sure everyone finishes computing before we overwrite the data for the next tile

### The Deadlock Trap ☠️

**NEVER DO THIS:**
```c
if (get_local_id(0) < 10) {
    barrier(CLK_LOCAL_MEM_FENCE);  // ☠️ DEADLOCK!
}
```

**Why?** Threads 0-9 wait at the barrier for threads 10+, but threads 10+ never reach the barrier. Everyone waits forever!

**The Fix:**
```c
if (get_local_id(0) < 10) {
    // Do special work
}
barrier(CLK_LOCAL_MEM_FENCE);  // Everyone hits this
```

### Practice: Where Do Barriers Go?

```c
__local int shared[256];
int tid = get_local_id(0);

// Load data
shared[tid] = input[get_global_id(0)];

// ⚠️ BARRIER NEEDED HERE! (After write, before read)
barrier(CLK_LOCAL_MEM_FENCE);

// Read neighbor's data
if (tid > 0) {
    int neighbor = shared[tid - 1];  // Reading what previous thread wrote
}
```

---

## 5. Tiled Matrix Multiplication 🔲

### Why Tiling?

**The Problem:** Regular matrix multiplication is SLOW because we read from global memory too much.

For C = A × B, to compute one element C[i][j]:
```c
// Naive approach - slow!
float sum = 0;
for (int k = 0; k < K; k++) {
    sum += A[i][k] * B[k][j];  // Every iteration reads from slow global memory
}
```

If K = 1000, we do 1000 slow memory reads per output element! 😱

**The Solution:** Break matrices into tiles, load tiles into fast local memory, reuse data!

### The Tiling Strategy

1. Divide matrix into TILE_SIZE × TILE_SIZE blocks
2. Load one tile of A and one tile of B into local memory (one slow read each)
3. Compute partial result using those tiles (many fast reads from local memory)
4. Move to next tiles and repeat
5. Sum all partial results

### Visual Understanding

For C = A × B where A is 4×4 and B is 4×4, with TILE_SIZE = 2:

```
    A           B           C
[1 2|3 4]   [1 2|3 4]   [? ?|? ?]
[5 6|7 8] × [5 6|7 8] = [? ?|? ?]
----+----   ----+----   ----+----
[9 0|1 2]   [9 0|1 2]   [? ?|? ?]
[3 4|5 6]   [3 4|5 6]   [? ?|? ?]
```

To compute top-left 2×2 of C, we need:
- Iteration 0: Left 2 columns of A, Top 2 rows of B
- Iteration 1: Right 2 columns of A, Bottom 2 rows of B

### The Code Pattern

```c
#define TILE_WIDTH 16

__kernel void matmul(__global float *A, 
                     __global float *B, 
                     __global float *C,
                     int numARows, int numAColumns,
                     int numBRows, int numBColumns) {
    
    // Shared memory for tiles
    __local float As[TILE_WIDTH][TILE_WIDTH];
    __local float Bs[TILE_WIDTH][TILE_WIDTH];
    
    // Thread IDs
    int tx = get_local_id(0);   // 0 to 15
    int ty = get_local_id(1);   // 0 to 15
    int bx = get_group_id(0);   // Which tile column
    int by = get_group_id(1);   // Which tile row
    
    // Which output element am I computing?
    int Row = by * TILE_WIDTH + ty;
    int Col = bx * TILE_WIDTH + tx;
    
    float Pvalue = 0.0f;
    
    // How many tiles do we need?
    int numTiles = (numAColumns + TILE_WIDTH - 1) / TILE_WIDTH;
    
    // Loop through tiles
    for (int m = 0; m < numTiles; m++) {
        
        // Load tile of A (my row, columns m*TILE to (m+1)*TILE)
        if (Row < numARows && m * TILE_WIDTH + tx < numAColumns) {
            As[ty][tx] = A[Row * numAColumns + m * TILE_WIDTH + tx];
        } else {
            As[ty][tx] = 0.0f;  // Padding for edge cases
        }
        
        // Load tile of B (rows m*TILE to (m+1)*TILE, my column)
        if (m * TILE_WIDTH + ty < numBRows && Col < numBColumns) {
            Bs[ty][tx] = B[(m * TILE_WIDTH + ty) * numBColumns + Col];
        } else {
            Bs[ty][tx] = 0.0f;  // Padding
        }
        
        // Wait for all loads to complete
        barrier(CLK_LOCAL_MEM_FENCE);
        
        // Compute partial dot product using this tile
        for (int k = 0; k < TILE_WIDTH; k++) {
            Pvalue += As[ty][k] * Bs[k][tx];
        }
        
        // Wait before loading next tile
        barrier(CLK_LOCAL_MEM_FENCE);
    }
    
    // Write final result
    if (Row < numCRows && Col < numCColumns) {
        C[Row * numBColumns + Col] = Pvalue;
    }
}
```

### Understanding the Indices

**For tile `m`, loading A:**
- We want row `Row` of A (stays constant)
- We want columns from `m*TILE_WIDTH` to `(m+1)*TILE_WIDTH`
- Each thread loads one element: `A[Row * numAColumns + m*TILE_WIDTH + tx]`
- Stored in: `As[ty][tx]`

**For tile `m`, loading B:**
- We want column `Col` of B (stays constant)
- We want rows from `m*TILE_WIDTH` to `(m+1)*TILE_WIDTH`
- Each thread loads one element: `B[(m*TILE_WIDTH + ty) * numBColumns + Col]`
- Stored in: `Bs[ty][tx]`

**Why this indexing?**
- `As[ty][tx]` stores the element for output row `ty` (within tile)
- `Bs[ty][tx]` stores the element for output col `tx` (within tile)
- When computing: `As[ty][k] * Bs[k][tx]` gives us the contribution from column `k`

### Rectangular Tiles

If TILE_WIDTH_X ≠ TILE_WIDTH_Y:

```c
#define TILE_WIDTH_X 32  // Columns
#define TILE_WIDTH_Y 16  // Rows

__local float As[TILE_WIDTH_Y][TILE_WIDTH_X];
__local float Bs[TILE_WIDTH_X][TILE_WIDTH_Y];  // Note: flipped!

int Row = by * TILE_WIDTH_Y + ty;
int Col = bx * TILE_WIDTH_X + tx;
```

### Why Two Barriers?

```c
// Load tiles
As[ty][tx] = A[...];
Bs[ty][tx] = B[...];

barrier(CLK_LOCAL_MEM_FENCE);  // ⚠️ Barrier 1: Wait for ALL loads

// Compute using tiles
for (int k = 0; k < TILE_WIDTH; k++) {
    Pvalue += As[ty][k] * Bs[k][tx];
}

barrier(CLK_LOCAL_MEM_FENCE);  // ⚠️ Barrier 2: Wait before next tile
```

**Barrier 1:** Everyone must finish writing to `As` and `Bs` before anyone reads from them
**Barrier 2:** Everyone must finish reading from `As` and `Bs` before we overwrite them with the next tile

---

## 6. Memory Coalescing 🚀

### What is Coalescing?

When consecutive threads access consecutive memory locations, the GPU can fetch all that data in ONE memory transaction instead of many separate transactions.

**Think of it like this:**
- Bad: Sending 32 individual packages (32 transactions)
- Good: Sending one big box with 32 items (1 transaction)

### Coalesced (FAST ✓)

```c
int idx = get_global_id(0);
float value = array[idx];
```

What happens:
```
Thread 0 → array[0]  ┐
Thread 1 → array[1]  │
Thread 2 → array[2]  ├─ All fetched in ONE transaction!
Thread 3 → array[3]  │
   ...               ┘
```

### Not Coalesced (SLOW ✗)

```c
int idx = get_global_id(0);
float value = array[idx * 100];
```

What happens:
```
Thread 0 → array[0]    ← Transaction 1
Thread 1 → array[100]  ← Transaction 2
Thread 2 → array[200]  ← Transaction 3
Thread 3 → array[300]  ← Transaction 4
   ...
```

Every access needs a separate transaction! 😱

### The Rules

Coalescing happens when:
1. Consecutive thread IDs access consecutive addresses
2. Within a subgroup (usually 32 threads)
3. Addresses are properly aligned

### Calculating Coalesced Accesses

**Given:**
- Maximum fetch size: 128 bytes
- Element size: 4 bytes (int or float)
- Subgroup size: 32 threads

**How many elements can coalesce?**
```
128 bytes / 4 bytes = 32 elements
```

So if 32 consecutive threads access 32 consecutive ints → ONE transaction!

### Example from Exam

```c
// local_size(0) = 4, local_size(1) = 16
int idx = 16 * get_local_id(1) + get_local_id(0);
float val = array[idx];
```

Thread (0, 0) → array[16*0 + 0] = array[0]
Thread (1, 0) → array[16*0 + 1] = array[1]
Thread (2, 0) → array[16*0 + 2] = array[2]
Thread (3, 0) → array[16*0 + 3] = array[3]

✓ These 4 consecutive threads access consecutive addresses!

Thread (0, 1) → array[16*1 + 0] = array[16]
Thread (1, 1) → array[16*1 + 1] = array[17]

✓ Next 4 are also consecutive!

### 2D Access Patterns

**Row-major access (usually coalesced):**
```c
int row = get_global_id(1);
int col = get_global_id(0);
int idx = row * width + col;
data[idx] = ...;
```

If threads (0,0), (1,0), (2,0), (3,0) execute together:
- They access row 0, columns 0, 1, 2, 3
- Consecutive columns → consecutive memory → ✓ Coalesced!

**Column-major access (usually NOT coalesced):**
```c
int row = get_global_id(1);
int col = get_global_id(0);
int idx = col * height + row;
data[idx] = ...;
```

If threads (0,0), (1,0), (2,0), (3,0) execute together:
- They access rows 0, 1, 2, 3 of column 0
- Rows are far apart in memory → ✗ Not coalesced!

### The Transpose Problem

When transposing, you can have EITHER:
- Coalesced reads + non-coalesced writes, OR
- Non-coalesced reads + coalesced writes

You can't have both! Solution: Use local memory to help:

```c
// Read coalesced
__local float tile[16][16];
tile[ty][tx] = input[row * width + col];  // Coalesced read

barrier(CLK_LOCAL_MEM_FENCE);

// Write coalesced (indices are flipped)
output[col * height + row] = tile[ty][tx];  // Coalesced write
```

---

## 7. Performance Analysis ⚡

### Compute Bound vs Memory Bound

Every GPU operation is limited by one of two things:
1. **Compute Bound:** How fast can we do math?
2. **Memory Bound:** How fast can we move data?

The slower one determines your actual performance.

### Step-by-Step Analysis

**Step 1: Calculate Operational Intensity (OI)**

```
OI = Floating Point Operations / Bytes Accessed
```

This tells you: "For every byte I move, how much math do I do?"

**Step 2: Calculate the Limits**

```
Compute Limit = Peak FLOPS (given in problem)
Memory Limit = Memory Bandwidth (GB/s) × Operational Intensity
```

**Step 3: Determine Bottleneck**

```
Actual Performance = min(Compute Limit, Memory Limit)

If Memory Limit < Compute Limit → MEMORY BOUND
If Compute Limit < Memory Limit → COMPUTE BOUND
```

### Example Problem

**Given:**
- Peak performance: 2 TFLOPS = 2000 GFLOPS
- Memory bandwidth: 256 GB/s
- Operation: Naive matrix multiplication
  - 2 memory reads (8 bytes each) per FMA (multiply-add = 2 FLOPs)

**Solution:**

**Step 1: Calculate OI**
```
FLOPs = 2 (one multiply, one add)
Bytes = 2 reads × 8 bytes = 16 bytes
OI = 2 FLOPs / 16 bytes = 0.125 FLOPs/byte
```

**Step 2: Calculate Limits**
```
Compute Limit = 2000 GFLOPS
Memory Limit = 256 GB/s × 0.125 FLOPs/byte = 32 GFLOPS
```

**Step 3: Determine Bottleneck**
```
Actual = min(2000, 32) = 32 GFLOPS
32 < 2000 → MEMORY BOUND! 🐌
```

We can only achieve 32 GFLOPS even though the hardware can do 2000 GFLOPS!

### The Roofline Model

```
Performance
    ↑
    │         ┌─────────────── Compute Bound (Flat roof)
2000│         │
    │         │
    │       ╱ │
    │     ╱   │
 32 ├───╱─────┤ Memory Bound (Sloped roof)
    │ ╱       │
    │╱        │
    └─────────┴──────────────→ Operational Intensity
    0       0.125
```

Your actual performance hits the "roof" - whichever is lower!

### How to Improve

**If Memory Bound:**
- Use tiling to reuse data from local memory
- Improve memory coalescing
- Reduce memory accesses
- Increase operational intensity

**If Compute Bound:**
- Optimize arithmetic operations
- Use faster algorithms
- Better utilize ALUs

### Practice Problem

**Given:**
- Peak: 100 GFLOPS
- Bandwidth: 200 GB/s
- Operation: 2 FLOPs per 8 bytes

**Your turn:**
```
OI = 2 / 8 = 0.25 FLOPs/byte
Memory Limit = 200 × 0.25 = 50 GFLOPS
Compute Limit = 100 GFLOPS
min(50, 100) = 50 → MEMORY BOUND
```

---

## 8. Common Patterns 🔄

### Pattern 1: Simple Vector Addition

```c
__kernel void vecAdd(__global float *A,
                     __global float *B,
                     __global float *C,
                     int N) {
    int i = get_global_id(0);
    
    if (i < N) {
        C[i] = A[i] + B[i];
    }
}

// Host code:
// size_t global = N;
// size_t local = 256;
```

**Key points:**
- Each work-item processes ONE element
- Boundary check: `if (i < N)`
- Perfectly coalesced if threads execute in order

---

### Pattern 2: Reduction (Sum)

**Goal:** Sum all elements in an array

**Strategy:** Tree-based reduction
```
[1, 2, 3, 4, 5, 6, 7, 8]
 │  │  │  │  │  │  │  │
 └─3┘  └─7┘  └11┘  └15┘   stride=1
    │     │     │     │
    └──10─┘     └──26─┘    stride=2
         │           │
         └─────36────┘       stride=4
```

**Code:**
```c
__kernel void reduce(__global int *input,
                     __global int *output) {
    __local int shared[256];
    
    int tid = get_local_id(0);
    int i = get_global_id(0);
    
    // Load into shared memory
    shared[tid] = input[i];
    barrier(CLK_LOCAL_MEM_FENCE);
    
    // Reduction in shared memory
    for (int stride = 1; stride < get_local_size(0); stride *= 2) {
        if (tid % (2 * stride) == 0) {
            shared[tid] += shared[tid + stride];
        }
        barrier(CLK_LOCAL_MEM_FENCE);
    }
    
    // Write result
    if (tid == 0) {
        output[get_group_id(0)] = shared[0];
    }
}
```

**Key points:**
- Stride DOUBLES each iteration: 1, 2, 4, 8, 16, ...
- Active threads: `tid % (2*stride) == 0`
- Barrier after each reduction step
- Each work-group produces one partial sum
- Need second pass to sum the partial sums

---

### Pattern 3: Transpose

**Goal:** Convert A (M×N) to B (N×M)

```c
__kernel void transpose(__global float *A,
                        __global float *B,
                        int numARows, int numACols) {
    int row = get_global_id(1);
    int col = get_global_id(0);
    
    if (row < numARows && col < numACols) {
        // Read from A[row][col]
        // Write to B[col][row]
        B[col * numARows + row] = A[row * numACols + col];
    }
}
```

**Key insight:** Swap row and column in the output!

---

### Pattern 4: Image Rotation

**Rotate Right 90°:**
```c
// Output position → Input position
input_row = width - 1 - output_col;
input_col = output_row;

output[output_row * height + output_col] = 
    input[input_row * width + input_col];
```

**Rotate Left 90°:**
```c
input_row = output_col;
input_col = height - 1 - output_row;
```

**Visual:**
```
Original:      Rotate Right:   Rotate Left:
[1 2 3]        [7 4 1]         [3 6 9]
[4 5 6]        [8 5 2]         [2 5 8]
[7 8 9]        [9 6 3]         [1 4 7]
```

---

### Pattern 5: Thread Coarsening

**Goal:** Each thread processes multiple elements

```c
#define COARSE_FACTOR 4

__kernel void vecAdd(__global float *in1,
                     __global float *in2,
                     __global float *out,
                     int len) {
    // Starting index for this thread
    int start = (get_group_id(0) * get_local_size(0) + get_local_id(0)) * COARSE_FACTOR;
    
    // Process COARSE_FACTOR elements
    for (int i = 0; i < COARSE_FACTOR; i++) {
        int idx = start + i;
        if (idx < len) {
            out[idx] = in1[idx] + in2[idx];
        }
    }
}
```

**Grid size calculation:**
```c
work_items_needed = (len + COARSE_FACTOR - 1) / COARSE_FACTOR;
num_groups = (work_items_needed + local_size - 1) / local_size;
```

**Benefits:**
- Better memory coalescing
- Reduced overhead
- Better register usage

---

## 9. Quick Reference Formulas 📋

### Indexing Formulas

```c
// 1D global index
global_id = get_group_id(0) * get_local_size(0) + get_local_id(0);

// 2D global indices
row = get_group_id(1) * get_local_size(1) + get_local_id(1);
col = get_group_id(0) * get_local_size(0) + get_local_id(0);

// 2D to 1D conversion
index = row * width + col;  // Row-major
index = col * height + row; // Column-major
```

### Grid Sizing

```c
// Simple case
num_groups = (N + block_size - 1) / block_size;

// With coarsening
work_items_needed = (N + coarse_factor - 1) / coarse_factor;
num_groups = (work_items_needed + block_size - 1) / block_size;
```

### Matrix Operations

```c
// Matrix multiply: C = A × B
// A is M×K, B is K×N, C is M×N

// Access A[i][k]
A[i * K + k]

// Access B[k][j]
B[k * N + j]

// Access C[i][j]
C[i * N + j]

// Transpose: B = A^T
// A is M×N, B is N×M
B[col * M + row] = A[row * N + col];
```

### Tiling

```c
// Number of tiles needed
num_tiles = (common_dim + TILE_WIDTH - 1) / TILE_WIDTH;

// Loading A for tile m
A[Row * numAColumns + m * TILE_WIDTH + tx]

// Loading B for tile m
B[(m * TILE_WIDTH + ty) * numBColumns + Col]
```

### Performance

```c
// Operational Intensity
OI = FLOPs / Bytes

// Memory performance limit
Memory_Limit = Bandwidth × OI

// Actual performance
Performance = min(Peak_FLOPs, Memory_Limit)
```

---

## 10. Common Pitfalls ⚠️

### 1. Barrier in Conditional

**❌ WRONG:**
```c
if (tid < 10) {
    barrier(CLK_LOCAL_MEM_FENCE);  // Deadlock!
}
```

**✅ RIGHT:**
```c
if (tid < 10) {
    // do work
}
barrier(CLK_LOCAL_MEM_FENCE);  // Everyone hits this
```

**Why?** ALL threads must hit the same barrier or you get deadlock.

---

### 2. Missing Barriers

**❌ WRONG:**
```c
shared[tid] = data[tid];
float val = shared[tid + 1];  // Race condition!
```

**✅ RIGHT:**
```c
shared[tid] = data[tid];
barrier(CLK_LOCAL_MEM_FENCE);
float val = shared[tid + 1];  // Safe now
```

---

### 3. Forgetting Boundary Checks

**❌ WRONG:**
```c
C[row * width + col] = result;  // Might be out of bounds!
```

**✅ RIGHT:**
```c
if (row < numRows && col < numCols) {
    C[row * width + col] = result;
}
```

---

### 4. Wrong Stride in Reduction

**❌ WRONG:**
```c
for (int stride = 1; stride < SIZE; stride *= 2) {
    if (tid % stride == 0) {  // WRONG!
        shared[tid] += shared[tid + stride];
    }
}
```

**✅ RIGHT:**
```c
for (int stride = 1; stride < SIZE; stride *= 2) {
    if (tid % (2 * stride) == 0) {  // Correct!
        shared[tid] += shared[tid + stride];
    }
}
```

**Why?** With `tid % stride`, too many threads are active!

---

### 5. Using Floor Instead of Ceiling

**❌ WRONG:**
```c
num_groups = N / block_size;  // Might miss elements!
```

**✅ RIGHT:**
```c
num_groups = (N + block_size - 1) / block_size;  // Ceiling division
```

**Why?** If N=1000 and block_size=256:
- Wrong: 1000/256 = 3 groups = 768 threads (missing 232 elements!)
- Right: (1000+255)/256 = 4 groups = 1024 threads ✓

---

### 6. Wrong Matrix Stride

**❌ WRONG:**
```c
// For A (M×K)
A[row * M + col]  // Using number of ROWS
```

**✅ RIGHT:**
```c
// For A (M×K)
A[row * K + col]  // Using number of COLUMNS
```

**Rule:** Stride = number of COLUMNS (width of the matrix)

---

### 7. Confusing Dimensions

**❌ WRONG:**
```c
int row = get_global_id(0);  // Dimension 0
int col = get_global_id(1);  // Dimension 1
```

**✅ RIGHT:**
```c
int row = get_global_id(1);  // Dimension 1 (Y)
int col = get_global_id(0);  // Dimension 0 (X)
```

**Remember:** Dimension 0 = X = columns, Dimension 1 = Y = rows

---

### 8. Barrier Placement in Loops

**❌ WRONG:**
```c
for (int m = 0; m < numTiles; m++) {
    barrier(CLK_LOCAL_MEM_FENCE);  // Too early!
    shared[tid] = data[m];
    // compute
}
```

**✅ RIGHT:**
```c
for (int m = 0; m < numTiles; m++) {
    shared[tid] = data[m];
    barrier(CLK_LOCAL_MEM_FENCE);  // After write!
    // compute
    barrier(CLK_LOCAL_MEM_FENCE);  // Before next iteration!
}
```

---

### 9. Incorrect Tiling Indices

**❌ WRONG:**
```c
// Loading A for tile m
As[ty][tx] = A[Row * numAColumns + m + tx];  // Missing TILE_WIDTH!
```

**✅ RIGHT:**
```c
As[ty][tx] = A[Row * numAColumns + m * TILE_WIDTH + tx];
```

---

### 10. Not Padding in Tiling

**❌ WRONG:**
```c
As[ty][tx] = A[Row * numAColumns + m * TILE_WIDTH + tx];
// What if we're past the edge of the matrix?
```

**✅ RIGHT:**
```c
if (Row < numARows && m * TILE_WIDTH + tx < numAColumns) {
    As[ty][tx] = A[Row * numAColumns + m * TILE_WIDTH + tx];
} else {
    As[ty][tx] = 0.0f;  // Pad with zero
}
```

---

## Final Exam Tips 💡

### Before the Exam
- [ ] Get 7-8 hours of sleep
- [ ] Eat a good breakfast
- [ ] Review this cheat sheet one more time
- [ ] Arrive 5-10 minutes early

### During the Exam
- [ ] Read EVERY question twice
- [ ] For indexing: Draw a small example (4×4 grid)
- [ ] For barriers: Trace reads and writes
- [ ] For math: Show your work
- [ ] Check boundary conditions
- [ ] If stuck, move on and come back

### Time Management
- Quick scan: 2 minutes
- First pass (easy questions): 20 minutes
- Second pass (medium questions): 30 minutes
- Hard questions: 20 minutes
- Review: 8 minutes

### Common Question Types
1. **Indexing:** Calculate global_id from group_id and local_id
2. **Barriers:** Do we need one? Where does it go?
3. **Tiling:** Fill in the blanks for loading tiles
4. **Performance:** Compute vs memory bound
5. **Patterns:** Reduction stride, transpose indices

---

## YOU'VE GOT THIS! 💪🔥

Remember:
- Trust your preparation
- Stay calm
- Read carefully
- Check your work
- Believe in yourself!

Good luck! 🌟
