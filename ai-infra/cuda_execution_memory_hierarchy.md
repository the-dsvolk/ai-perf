# CUDA Execution & Memory Hierarchy (Top to Bottom)

## Execution Hierarchy

### Grid
The **entire job** you launch on the GPU. One kernel launch = one grid.
- Contains many **blocks**
- Blocks are **distributed across ALL SMs** by the hardware scheduler
- A grid can have thousands of blocks — they don't all run at once

```
GPU (e.g., 4 SMs)
├── SM 0: Block 0, Block 4, Block 8 ...
├── SM 1: Block 1, Block 5, Block 9 ...
├── SM 2: Block 2, Block 6, Block 10 ...
└── SM 3: Block 3, Block 7, Block 11 ...
         ↑
         One grid's blocks, distributed across all SMs
```

**Key insight**: A single kernel with enough blocks will keep **all SMs busy**. You don't need multiple concurrent kernels — just make your grid large enough.

| GPU | SMs | Minimum blocks to saturate |
|-----|-----|---------------------------|
| H100 | 132 | ≥132 (but more is better) |
| A100 | 108 | ≥108 |
| RTX 4090 | 128 | ≥128 |

In practice, you want **many more blocks than SMs** (e.g., 4-8× more) because:
- Multiple blocks can run per SM (if they fit in registers/shared memory)
- More blocks = more warps = better latency hiding when warps stall

### Block (Thread Block)
A group of threads that:
- Run on the **same SM** (Streaming Multiprocessor)
- Can **share memory** with each other (shared memory)
- Can **synchronize** with each other (`__syncthreads()`)
- Max ~1024 threads per block

### Warp
The **actual execution unit** — 32 threads that execute **in lockstep** (SIMT).
- The GPU doesn't run individual threads — it runs warps
- All 32 threads execute the same instruction at the same time
- If threads diverge (if/else), both paths run serially → performance hit

### Thread
The smallest unit of execution. Each thread has:
- Its own **registers** (private, fastest memory)
- Its own thread ID (`threadIdx.x`)

---

## Memory Hierarchy (Fast → Slow)

| Memory | Scope | Speed | Size |
|--------|-------|-------|------|
| **Registers** | Per thread | Fastest | See clarification below |
| **Shared Memory** | Per block | Very fast | ~48-164 KB per SM |
| **L1/L2 Cache** | Automatic | Fast | MB range |
| **Global Memory (HBM)** | All threads | Slow | GBs |

**→ For a detailed walk-through with sizes, latencies, and throughput:** [Life of a Memory Request: HBM to Register](memory_request_life.html) — how a load travels from HBM to a register, with a diagram and reference table.

---

## Registers: Clarification

| What | Value (typical modern GPU) |
|------|----------------------------|
| **Total registers per SM** | ~65,536 (32-bit registers) |
| **Max registers per thread** | 255 (hardware limit) |

### How it works

The SM has a **fixed pool** of registers (~65K). These are **divided among all active threads** on that SM.

**Example:**
- SM has 65,536 registers
- You launch blocks with 1024 threads total on that SM
- Each thread gets: `65,536 / 1024 = 64 registers`

### The Tradeoff (Occupancy)

| Registers/thread | Threads per SM | Occupancy |
|------------------|----------------|-----------|
| 32 | 2048 | High ✓ |
| 64 | 1024 | Medium |
| 128 | 512 | Low |
| 255 | ~256 | Very low ✗ |

**More registers per thread** → kernel can keep more data in fast memory → faster per thread  
**Fewer registers per thread** → more threads fit → better latency hiding

### Summary

- **255** = max any single thread can use
- **~65K** = total pool per SM, shared by all threads
- Compiler decides how many each thread needs → affects how many threads can run

---

## Warp Scheduler

Each SM has **warp schedulers** that:
1. Pick ready warps (not waiting on memory)
2. Issue instructions to execution units
3. Hide latency by switching between warps instantly

**Key insight**: When one warp stalls (waiting for memory), the scheduler switches to another warp at zero cost — this is how GPUs hide latency.

---

## Quick Mental Model

```
Grid
 └── Block 0, Block 1, Block 2, ...
      └── Warp 0 (threads 0-31), Warp 1 (threads 32-63), ...
           └── Thread 0, Thread 1, ... Thread 31
                └── Registers (private to each thread)
```

**Think of it as**: Grid → Blocks → Warps → Threads → Registers

---

## Kernel Launch: Specifying Grid & Block

When you launch a CUDA kernel, you specify **both** grid and block dimensions:

```cuda
kernel<<<gridDim, blockDim>>>(args);
```

| Parameter | What it specifies | Can be |
|-----------|-------------------|--------|
| `gridDim` | Number of blocks in the grid | 1D, 2D, or 3D |
| `blockDim` | Number of threads per block | 1D, 2D, or 3D |

### Examples

```cuda
// 1D grid, 1D blocks: 256 blocks × 128 threads = 32,768 threads
kernel<<<256, 128>>>(data);

// 2D grid, 1D blocks (e.g., for images)
dim3 grid(32, 32);      // 32×32 = 1024 blocks
dim3 block(256);        // 256 threads per block
kernel<<<grid, block>>>(image);

// 2D grid, 2D blocks (common for image processing)
dim3 grid(width/16, height/16);
dim3 block(16, 16);     // 16×16 = 256 threads per block
kernel<<<grid, block>>>(image);

// 3D grid, 3D blocks (e.g., for volumes)
dim3 grid(8, 8, 8);
dim3 block(8, 8, 8);    // 512 threads per block
kernel<<<grid, block>>>(volume);
```

### Why 2D/3D?

It's just **convenience** for indexing — the GPU doesn't care. A 16×16 block = 256 threads = a 1D block of 256.

But for image processing, it's easier to write:
```cuda
int x = blockIdx.x * blockDim.x + threadIdx.x;
int y = blockIdx.y * blockDim.y + threadIdx.y;
pixel = image[y * width + x];
```

Than to manually compute 2D coords from a 1D index.

---

## References

- [CUDA C Programming Guide - Memory Hierarchy](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#memory-hierarchy)
