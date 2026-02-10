# CSE 160 Exam 1 Quick Reference Cheat Sheet 🔥

## Last-Minute Cram Guide - Read This Tomorrow Morning!

---

## 🎯 THE MOST IMPORTANT FORMULAS

### The Golden Formula
```c
global_id = group_id × local_size + local_id
```
**In code:**
```c
int i = get_group_id(0) * get_local_size(0) + get_local_id(0);
```

### 2D Indexing
```c
row = get_group_id(1) * get_local_size(1) + get_local_id(1);
col = get_group_id(0) * get_local_size(0) + get_local_id(0);
```

### Matrix Access (Row-Major)
```c
index = row * width + col
```

### Grid Sizing (Ceiling Division)
```c
num_groups = (N + block_size - 1) / block_size
```

---

## 📊 KEY OPENCL FUNCTIONS

| Function | What It Returns |
|----------|----------------|
| `get_global_id(d)` | Your unique ID (0 to N-1) |
| `get_local_id(d)` | Your ID in work-group (0 to local_size-1) |
| `get_group_id(d)` | Which work-group (0 to num_groups-1) |
| `get_local_size(d)` | Work-items per work-group |
| `get_global_size(d)` | Total work-items |
| `get_num_groups(d)` | Total work-groups |

---

## 🚧 BARRIER RULES - DON'T DEADLOCK!

### When You NEED a Barrier
If Thread A writes to shared memory and Thread B reads it:
```c
shared[tid] = data[tid];           // Thread A writes
barrier(CLK_LOCAL_MEM_FENCE);      // ⚠️ BARRIER NEEDED HERE
float val = shared[other_tid];     // Thread B reads
```

### The Pattern in Tiling
```c
for (int m = 0; m < numTiles; m++) {
    shared[tid] = data[...];                // Load
    barrier(CLK_LOCAL_MEM_FENCE);          // Wait for all loads
    
    for (int k = 0; k < TILE; k++) {       // Compute
        sum += shared[...];
    }
    
    barrier(CLK_LOCAL_MEM_FENCE);          // Wait before next tile
}
```

### ⚠️ NEVER DO THIS (Deadlock!)
```c
if (tid < 10) {
    barrier(CLK_LOCAL_MEM_FENCE);  // ☠️ Only threads 0-9 hit this!
}
```

### ✅ DO THIS INSTEAD
```c
if (tid < 10) {
    // do work
}
barrier(CLK_LOCAL_MEM_FENCE);  // Everyone hits this
```

---

## 🔲 TILED MATRIX MULTIPLICATION

### The Complete Pattern
```c
__local float As[TILE][TILE];
__local float Bs[TILE][TILE];

int tx = get_local_id(0);
int ty = get_local_id(1);
int Row = get_group_id(1) * TILE + ty;
int Col = get_group_id(0) * TILE + tx;

float sum = 0;

for (int m = 0; m < numTiles; m++) {
    // Load A (my row, columns m*TILE to (m+1)*TILE)
    if (Row < numARows && m*TILE + tx < numAColumns)
        As[ty][tx] = A[Row * numAColumns + m*TILE + tx];
    else
        As[ty][tx] = 0;
    
    // Load B (rows m*TILE to (m+1)*TILE, my column)
    if (m*TILE + ty < numBRows && Col < numBColumns)
        Bs[ty][tx] = B[(m*TILE + ty) * numBColumns + Col];
    else
        Bs[ty][tx] = 0;
    
    barrier(CLK_LOCAL_MEM_FENCE);  // ⚠️ CRITICAL!
    
    // Compute
    for (int k = 0; k < TILE; k++)
        sum += As[ty][k] * Bs[k][tx];
    
    barrier(CLK_LOCAL_MEM_FENCE);  // ⚠️ CRITICAL!
}

if (Row < numCRows && Col < numCColumns)
    C[Row * numCColumns + Col] = sum;
```

### Key Indices to Remember
For tile `m`:
- **Loading A:** `A[Row * numAColumns + m*TILE + tx]`
- **Loading B:** `B[(m*TILE + ty) * numBColumns + Col]`
- **Store in:** `As[ty][tx]` and `Bs[ty][tx]`

---

## 🚀 MEMORY COALESCING

### Good (Coalesced) ✅
```c
int idx = get_global_id(0);
val = array[idx];
```
Thread 0→array[0], Thread 1→array[1], Thread 2→array[2] ... **Sequential!**

### Bad (Not Coalesced) ❌
```c
int idx = get_global_id(0);
val = array[idx * 100];
```
Thread 0→array[0], Thread 1→array[100], Thread 2→array[200] ... **Jumps!**

### The Math
- Coalescing happens within **subgroups** (usually 32 threads)
- Max fetch: **128 bytes**
- Example: `128 bytes / 4 bytes per int = 32 ints can coalesce`

---

## ⚡ COMPUTE vs MEMORY BOUND

### The 3-Step Process

**Step 1: Calculate Operational Intensity**
```
OI = FLOPs / Bytes Accessed
```

**Step 2: Calculate Limits**
```
Compute Limit = Peak FLOPs (given)
Memory Limit = Memory Bandwidth × OI
```

**Step 3: Determine Bottleneck**
```
Performance = min(Compute Limit, Memory Limit)

If Memory Limit < Compute Limit → MEMORY BOUND
If Compute Limit < Memory Limit → COMPUTE BOUND
```

### Example
- Peak = 100 GFLOPS, Bandwidth = 200 GB/s
- Operation: 2 FLOPs per 8 bytes

```
OI = 2/8 = 0.25 FLOPs/byte
Memory Limit = 200 × 0.25 = 50 GFLOPS
Compute Limit = 100 GFLOPS
min(50, 100) = 50 → MEMORY BOUND
```

---

## 🌳 REDUCTION PATTERN

```c
__local int shared[BLOCK_SIZE];

shared[tid] = input[global_id];
barrier(CLK_LOCAL_MEM_FENCE);

for (int stride = 1; stride < BLOCK_SIZE; stride *= 2) {
    if (tid % (2 * stride) == 0) {
        shared[tid] += shared[tid + stride];
    }
    barrier(CLK_LOCAL_MEM_FENCE);
}

if (tid == 0)
    output[group_id] = shared[0];
```

**Key Points:**
- Stride doubles: 1, 2, 4, 8, 16...
- Active threads: `tid % (2*stride) == 0`
- **NOT** `tid % stride == 0` ← Common mistake!

---

## 🔄 COMMON OPERATIONS

### Transpose
```c
// A is H×W → B is W×H
int row = get_global_id(1);
int col = get_global_id(0);
B[col * H + row] = A[row * W + col];
```

### Rotate Right 90°
```c
input_row = width - output_col - 1;
input_col = output_row;
```

### Rotate Left 90°
```c
input_row = output_col;
input_col = height - output_row - 1;
```

---

## 📐 MATRIX DIMENSIONS

For **C = A × B** where:
- A is **M×K**
- B is **K×N**  
- C is **M×N**

### Access Patterns
```c
A[row][k]  → A[row * K + k]      // Stride = K (num columns)
B[k][col]  → B[k * N + col]      // Stride = N (num columns)
C[row][col] → C[row * N + col]   // Stride = N (num columns)
```

**Remember:** Stride = number of COLUMNS (width)

---

## ⚠️ TOP 10 MISTAKES TO AVOID

### 1. Ceiling Division
❌ `num_groups = N / block_size`  
✅ `num_groups = (N + block_size - 1) / block_size`

### 2. Reduction Stride
❌ `if (tid % stride == 0)`  
✅ `if (tid % (2 * stride) == 0)`

### 3. Barrier in Conditional
❌ `if (...) { barrier(...); }`  
✅ `barrier(...)` outside the if

### 4. Missing Boundary Check
❌ `C[row * width + col] = ...`  
✅ `if (row < H && col < W) { C[...] = ...; }`

### 5. Wrong Matrix Stride
❌ `A[row * M + col]` for A(M×K)  
✅ `A[row * K + col]` ← Use num COLUMNS

### 6. Confusing Dimensions
❌ `row = get_global_id(0)`  
✅ `row = get_global_id(1)` ← Dimension 1 is Y

### 7. Missing Tile Padding
❌ `As[ty][tx] = A[...]` without bounds check  
✅ `if (...) As[ty][tx] = A[...]; else As[ty][tx] = 0;`

### 8. Tiling Index
❌ `A[Row * numAColumns + m + tx]`  
✅ `A[Row * numAColumns + m*TILE + tx]`

### 9. Missing Second Barrier
```c
for (m = 0; m < numTiles; m++) {
    // load
    barrier(...);     // ✅ Have this
    // compute
    barrier(...);     // ✅ DON'T FORGET THIS!
}
```

### 10. Wrong Fence Type
❌ `barrier(CLK_GLOBAL_MEM_FENCE)` for local memory  
✅ `barrier(CLK_LOCAL_MEM_FENCE)` for __local

---

## 💾 MEMORY SIZE CALCULATIONS

```
Memory needed = work_items × bytes_per_item

Example:
- Work-group size: 1024
- Each loads 10 floats: 10 × 4 = 40 bytes
- Total: 1024 × 40 = 40,960 bytes = 40 KB

If limit is 64 KB → ✅ OK
If limit is 32 KB → ❌ EXCEEDS
```

---

## 🔢 THREAD COARSENING

```c
#define ELEMENTS_PER_THREAD 4

int start = get_global_id(0) * ELEMENTS_PER_THREAD;

for (int i = 0; i < ELEMENTS_PER_THREAD; i++) {
    if (start + i < N) {
        out[start + i] = in[start + i];
    }
}
```

**Grid size:**
```c
work_items = (N + EPT - 1) / EPT;
num_groups = (work_items + local_size - 1) / local_size;
```

---

## ✍️ EXAM PROBLEM-SOLVING CHECKLIST

### For Indexing Problems
- [ ] Draw a 4×4 example
- [ ] Calculate for thread 0
- [ ] Calculate for thread 1  
- [ ] Verify the pattern

### For Barrier Problems
- [ ] Who writes to shared memory?
- [ ] Who reads from shared memory?
- [ ] Does write happen before read?
- [ ] Do ALL threads hit the barrier?

### For Tiling Problems
- [ ] Identify tx, ty (local) vs Row, Col (global)
- [ ] Check As[ty][tx] loads correct Row data
- [ ] Check Bs[ty][tx] loads correct Col data
- [ ] Verify both barriers are present
- [ ] Check boundary conditions

### For Performance Problems
- [ ] Calculate OI = FLOPs / Bytes
- [ ] Calculate Memory Limit = BW × OI
- [ ] Compare with Compute Limit
- [ ] Take minimum → answer

---

## 🎯 EXAM DAY STRATEGY

### Time Budget (80 minutes)
- **Quick scan:** 2 min
- **Easy questions:** 20 min
- **Medium questions:** 30 min
- **Hard questions:** 20 min
- **Review:** 8 min

### During the Exam
1. **Read each question TWICE**
2. **For code:** Trace execution for threads 0, 1, and last
3. **For math:** Show your work
4. **Stuck?** Move on, come back later
5. **Double-check scantron bubbles!**

### The Night Before
- [ ] Review this cheat sheet
- [ ] Get 7-8 hours sleep
- [ ] Don't cram - trust your prep

### Tomorrow Morning
- [ ] Eat good breakfast
- [ ] Read this cheat sheet once more
- [ ] Arrive 5-10 minutes early
- [ ] Stay calm and confident

---

## 💡 FINAL TIPS

### If You Blank on Something
- **Indexing?** Think: global = group × size + local
- **Barriers?** Ask: Does A write what B reads?
- **Tiling?** Remember: As[ty][tx] for Row, Bs for Col
- **Performance?** Calculate both limits, take minimum

### Common Sense Checks
- Does my answer make sense?
- Did I use ceiling division?
- Are my array bounds correct?
- Did I check all boundary cases?

---

## 🔥 YOU'VE GOT THIS!

**Remember:**
- You've prepared well
- Trust yourself
- Stay calm
- Read carefully
- Check your work

**Believe in yourself! You're going to do GREAT! 💪**

---

## Quick Reference: Most Common Patterns

### Vector Add
```c
int i = get_global_id(0);
if (i < N) out[i] = a[i] + b[i];
```

### Reduction
```c
for (stride = 1; stride < SIZE; stride *= 2)
    if (tid % (2*stride) == 0)
        shared[tid] += shared[tid + stride];
```

### Transpose
```c
B[col*H + row] = A[row*W + col];
```

### Tiling Load
```c
As[ty][tx] = A[Row * numAColumns + m*TILE + tx];
Bs[ty][tx] = B[(m*TILE + ty) * numBColumns + Col];
```

---

**Last thought:** If you know these patterns, you'll ace the exam. Good luck! 🌟
