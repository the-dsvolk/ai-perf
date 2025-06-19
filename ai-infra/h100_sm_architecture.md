# NVIDIA H100 Streaming Multiprocessor Architecture

## Color Palette & Style Guide

The following color scheme is used consistently across all H100 system diagrams for visual clarity and component identification:


## H100 Streaming Multiprocessor Overview

This comprehensive guide illustrates the internal architecture of a single Streaming Multiprocessor (SM) in the NVIDIA H100 GPU, showing the complete data flow, execution pipeline, and memory hierarchy.

### High-Level SM Architecture

```mermaid
flowchart TB
    subgraph "H100 Streaming Multiprocessor (SM) - Complete Architecture"
        direction TB
        
        %% Instruction Management Layer
        subgraph "Instruction Management"
            direction LR
            ICache[<font size="10">Instruction Cache<br/>128KB</font>]
            IBuffer[<font size="10">Instruction Buffer<br/>Multiple Levels</font>]
            IDecode[<font size="10">Instruction Decode<br/>4-way decode</font>]
        end

        %% Warp Scheduling Layer
        subgraph "Warp Scheduling Layer"
            direction LR
            WS1[<font size="10">Warp Scheduler 1<br/>16 Active Warps</font>]
            WS2[<font size="10">Warp Scheduler 2<br/>16 Active Warps</font>]
            WS3[<font size="10">Warp Scheduler 3<br/>16 Active Warps</font>]
            WS4[<font size="10">Warp Scheduler 4<br/>16 Active Warps</font>]
            DispatchUnit[<font size="10">Dispatch Unit<br/>Instruction Distribution</font>]
        end

        %% Execution Units Layer
        subgraph "Execution Units"
            direction LR
            
            subgraph "Integer & FP Units"
                direction TB
                INT1[<font size="10">INT32 Units<br/>16x ALU</font>]
                FP32_1[<font size="10">FP32 Units<br/>16x CUDA Cores</font>]
                FP64_1[<font size="10">FP64 Units<br/>8x Double Precision</font>]
            end
            
            subgraph "Tensor Processing"
                direction TB
                TC1[<font size="10">Tensor Core Gen 4<br/>Matrix Engine 1</font>]
                TC2[<font size="10">Tensor Core Gen 4<br/>Matrix Engine 2</font>]
                TC3[<font size="10">Tensor Core Gen 4<br/>Matrix Engine 3</font>]
                TC4[<font size="10">Tensor Core Gen 4<br/>Matrix Engine 4</font>]
            end
            
            subgraph "Special Functions"
                direction TB
                SFU1[<font size="10">Special Function Unit 1<br/>Transcendental Math</font>]
                SFU2[<font size="10">Special Function Unit 2<br/>Advanced Math</font>]
                RT1[<font size="10">RT Core Gen 3<br/>Ray-Triangle Intersection</font>]
                RT2[<font size="10">RT Core Gen 3<br/>BVH Traversal</font>]
            end
        end

        %% Memory Subsystem
        subgraph "Memory Subsystem"
            direction LR
            
            subgraph "Register Files"
                direction TB
                RF1[<font size="10">Register File Bank 1<br/>64KB - Warp 1-16</font>]
                RF2[<font size="10">Register File Bank 2<br/>64KB - Warp 17-32</font>]
                RF3[<font size="10">Register File Bank 3<br/>64KB - Warp 33-48</font>]
                RF4[<font size="10">Register File Bank 4<br/>64KB - Warp 49-64</font>]
            end
            
            subgraph "Cache & Shared Memory"
                direction TB
                L1Cache[<font size="10">L1 Data Cache<br/>192KB Configurable</font>]
                SharedMem[<font size="10">Shared Memory<br/>256KB Configurable</font>]
                TexCache[<font size="10">Texture Cache<br/>96KB</font>]
                ConstCache[<font size="10">Constant Cache<br/>64KB</font>]
            end
            
            subgraph "Load/Store & Atomics"
                direction TB
                LSU1[<font size="10">Load/Store Unit 1<br/>Memory Operations</font>]
                LSU2[<font size="10">Load/Store Unit 2<br/>Memory Operations</font>]
                AtomicUnit[<font size="10">Atomic Unit<br/>Atomic Operations</font>]
                CoalescingUnit[<font size="10">Coalescing Unit<br/>Memory Access Pattern</font>]
            end
        end

        %% External Memory Interface
        subgraph "External Memory Interface"
            direction LR
            L2Cache[<font size="10">L2 Cache Slice<br/>6MB per SM</font>]
            MemCtrl[<font size="10">Memory Controller<br/>HBM3 Interface</font>]
            HBM[<font size="10">HBM3 Memory<br/>80GB, 3TB/s</font>]
        end

        %% Connections - Instruction Flow
        ICache --> IBuffer
        IBuffer --> IDecode
        IDecode --> WS1 & WS2 & WS3 & WS4
        WS1 & WS2 & WS3 & WS4 --> DispatchUnit

        %% Connections - Execution Flow
        DispatchUnit --> INT1 & FP32_1 & FP64_1
        DispatchUnit --> TC1 & TC2 & TC3 & TC4
        DispatchUnit --> SFU1 & SFU2 & RT1 & RT2
        DispatchUnit --> LSU1 & LSU2 & AtomicUnit

        %% Connections - Register Access
        INT1 & FP32_1 & FP64_1 --> RF1 & RF2 & RF3 & RF4
        TC1 & TC2 & TC3 & TC4 --> RF1 & RF2 & RF3 & RF4
        SFU1 & SFU2 --> RF1 & RF2 & RF3 & RF4

        %% Connections - Memory Hierarchy
        LSU1 & LSU2 --> CoalescingUnit
        CoalescingUnit --> L1Cache & SharedMem & TexCache & ConstCache
        L1Cache & SharedMem --> L2Cache
        AtomicUnit --> L2Cache
        L2Cache --> MemCtrl
        MemCtrl --> HBM
    end

    %% Apply Styles
    style ICache fill:#9966ff,stroke:#333,stroke-width:2px,color:#fff
    style IBuffer fill:#9966ff,stroke:#333,stroke-width:2px,color:#fff
    style IDecode fill:#9966ff,stroke:#333,stroke-width:2px,color:#fff
    style WS1 fill:#9966ff,stroke:#333,stroke-width:2px,color:#fff
    style WS2 fill:#9966ff,stroke:#333,stroke-width:2px,color:#fff
    style WS3 fill:#9966ff,stroke:#333,stroke-width:2px,color:#fff
    style WS4 fill:#9966ff,stroke:#333,stroke-width:2px,color:#fff
    style DispatchUnit fill:#9966ff,stroke:#333,stroke-width:2px,color:#fff
    style INT1 fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style FP32_1 fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style FP64_1 fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style SFU1 fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style SFU2 fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style TC1 fill:#ff6600,stroke:#333,stroke-width:2px,color:#fff
    style TC2 fill:#ff6600,stroke:#333,stroke-width:2px,color:#fff
    style TC3 fill:#ff6600,stroke:#333,stroke-width:2px,color:#fff
    style TC4 fill:#ff6600,stroke:#333,stroke-width:2px,color:#fff
    style RT1 fill:#cc6600,stroke:#333,stroke-width:2px,color:#fff
    style RT2 fill:#cc6600,stroke:#333,stroke-width:2px,color:#fff
    style RF1 fill:#66ccff,stroke:#333,stroke-width:2px,color:#000
    style RF2 fill:#66ccff,stroke:#333,stroke-width:2px,color:#000
    style RF3 fill:#66ccff,stroke:#333,stroke-width:2px,color:#000
    style RF4 fill:#66ccff,stroke:#333,stroke-width:2px,color:#000
    style SharedMem fill:#66ccff,stroke:#333,stroke-width:2px,color:#000
    style L1Cache fill:#ff9966,stroke:#333,stroke-width:2px,color:#000
    style TexCache fill:#ff9966,stroke:#333,stroke-width:2px,color:#000
    style ConstCache fill:#ff9966,stroke:#333,stroke-width:2px,color:#000
    style L2Cache fill:#ff9966,stroke:#333,stroke-width:2px,color:#000
    style LSU1 fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style LSU2 fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style AtomicUnit fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style CoalescingUnit fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style MemCtrl fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style HBM fill:#663399,stroke:#333,stroke-width:2px,color:#fff
```

## Detailed Component Architecture

### Tensor Core Architecture (4th Generation)

```mermaid
flowchart TB
    subgraph "H100 Tensor Core - 4th Generation"
        direction TB
        
        %% Input Processing
        subgraph "Input Processing"
            direction LR
            MatrixA[Matrix A Input<br/>FP16/BF16/FP8/INT8]
            MatrixB[Matrix B Input<br/>FP16/BF16/FP8/INT8]
            AccumC[Accumulator C<br/>FP32/FP16/BF16]
        end

        %% Core Processing
        subgraph "Matrix Processing Engine"
            direction TB
            MMA1[Matrix Multiply Unit 1<br/>16x16 Operations]
            MMA2[Matrix Multiply Unit 2<br/>16x16 Operations]
            MMA3[Matrix Multiply Unit 3<br/>16x16 Operations]
            MMA4[Matrix Multiply Unit 4<br/>16x16 Operations]
            
            AccumulatorArray[Accumulator Array<br/>256x FP32 Accumulators]
            
            SparsityEngine[Sparsity Engine<br/>2:4 Structured Sparsity]
            CompressionUnit[Compression Unit<br/>Data Compression]
        end

        %% Output Processing
        subgraph "Output Processing"
            direction LR
            TypeConvert[Type Conversion<br/>FP32→FP16/BF16]
            OutputBuffer[Output Buffer<br/>Result Staging]
            WriteBack[Write Back Unit<br/>Register File]
        end

        %% Data Flow
        MatrixA --> MMA1 & MMA2
        MatrixB --> MMA1 & MMA3
        MatrixA --> SparsityEngine
        MatrixB --> CompressionUnit
        SparsityEngine --> MMA1 & MMA2 & MMA3 & MMA4
        CompressionUnit --> MMA1 & MMA2 & MMA3 & MMA4
        
        MMA1 & MMA2 & MMA3 & MMA4 --> AccumulatorArray
        AccumC --> AccumulatorArray
        AccumulatorArray --> TypeConvert
        TypeConvert --> OutputBuffer
        OutputBuffer --> WriteBack
    end

    %% Apply Styles
    style MatrixA fill:#66ccff,stroke:#333,stroke-width:2px,color:#000
    style MatrixB fill:#66ccff,stroke:#333,stroke-width:2px,color:#000
    style AccumC fill:#66ccff,stroke:#333,stroke-width:2px,color:#000
    style MMA1 fill:#ff6600,stroke:#333,stroke-width:2px,color:#fff
    style MMA2 fill:#ff6600,stroke:#333,stroke-width:2px,color:#fff
    style MMA3 fill:#ff6600,stroke:#333,stroke-width:2px,color:#fff
    style MMA4 fill:#ff6600,stroke:#333,stroke-width:2px,color:#fff
    style AccumulatorArray fill:#ff6600,stroke:#333,stroke-width:2px,color:#fff
    style SparsityEngine fill:#cc6600,stroke:#333,stroke-width:2px,color:#fff
    style CompressionUnit fill:#cc6600,stroke:#333,stroke-width:2px,color:#fff
    style TypeConvert fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style OutputBuffer fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style WriteBack fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
```

### Memory Hierarchy and Data Flow

```mermaid
flowchart TB
    subgraph "H100 SM Memory Hierarchy"
        direction TB
        
        %% Thread Level
        subgraph "Per-Thread Memory"
            direction LR
            Registers[Registers<br/>255 per thread<br/>32-bit each]
            LocalMem[Local Memory<br/>Spilled registers<br/>In L1/L2]
        end

        %% Warp Level  
        subgraph "Per-Warp Memory"
            direction LR
            WarpRF[Warp Register File<br/>64KB per bank<br/>4 banks total]
            WarpShared[Warp-Shared Data<br/>In Shared Memory]
        end

        %% Block Level
        subgraph "Per-Block Memory"
            direction LR
            BlockShared[Shared Memory<br/>256KB configurable<br/>Low latency]
            BlockLocal[Block Local Memory<br/>In L1 Cache]
        end

        %% SM Level
        subgraph "SM-Wide Memory"
            direction LR
            L1DataCache[L1 Data Cache<br/>192KB configurable<br/>Combined with Shared]
            TextureCache[Texture Cache<br/>96KB<br/>Read-only optimized]
            ConstantCache[Constant Cache<br/>64KB<br/>Broadcast reads]
            InstructionCache[Instruction Cache<br/>128KB<br/>Code storage]
        end

        %% GPU Level
        subgraph "GPU-Wide Memory"
            direction LR
            L2Cache[L2 Cache<br/>6MB per SM<br/>Total 80MB]
            HBM3Memory[HBM3 Memory<br/>80GB total<br/>3TB/s bandwidth]
        end

        %% System Level
        subgraph "System Memory"
            direction LR
            SystemRAM[System RAM<br/>CPU Memory<br/>PCIe Connection]
            NVME[NVMe Storage<br/>High-speed SSD<br/>Data Loading]
        end

        %% Memory Flow Connections
        Registers --> LocalMem
        Registers --> WarpRF
        WarpRF --> WarpShared
        WarpShared --> BlockShared
        BlockShared --> L1DataCache
        BlockLocal --> L1DataCache
        L1DataCache --> L2Cache
        TextureCache --> L2Cache
        ConstantCache --> L2Cache
        InstructionCache --> L2Cache
        L2Cache --> HBM3Memory
        HBM3Memory -.->|PCIe Gen5<br/>128GB/s| SystemRAM
        SystemRAM -.->|NVMe-oF<br/>28GB/s| NVME
    end

    %% Apply Styles
    style Registers fill:#66ccff,stroke:#333,stroke-width:2px,color:#000
    style WarpRF fill:#66ccff,stroke:#333,stroke-width:2px,color:#000
    style LocalMem fill:#66ccff,stroke:#333,stroke-width:2px,color:#000
    style WarpShared fill:#66ccff,stroke:#333,stroke-width:2px,color:#000
    style BlockShared fill:#66ccff,stroke:#333,stroke-width:2px,color:#000
    style BlockLocal fill:#66ccff,stroke:#333,stroke-width:2px,color:#000
    style L1DataCache fill:#ff9966,stroke:#333,stroke-width:2px,color:#000
    style TextureCache fill:#ff9966,stroke:#333,stroke-width:2px,color:#000
    style ConstantCache fill:#ff9966,stroke:#333,stroke-width:2px,color:#000
    style InstructionCache fill:#ff9966,stroke:#333,stroke-width:2px,color:#000
    style L2Cache fill:#ff9966,stroke:#333,stroke-width:2px,color:#000
    style HBM3Memory fill:#663399,stroke:#333,stroke-width:2px,color:#fff
    style SystemRAM fill:#666666,stroke:#333,stroke-width:2px,color:#fff
    style NVME fill:#666666,stroke:#333,stroke-width:2px,color:#fff
```

## Comprehensive Technical Specifications

### Execution Units Specifications

#### CUDA Cores (FP32 Units)
- **Count**: 128 CUDA cores per SM (4 blocks × 32 cores)
- **Operations**: Single-precision floating-point, integer
- **Throughput**: 2,048 FP32 operations per clock (fused multiply-add)
- **Clock Speed**: 1.755 GHz boost, 1.41 GHz base
- **Peak Performance**: 3.6 TFlops FP32 per SM

#### Tensor Cores (4th Generation)
- **Count**: 4 Tensor Cores per SM
- **Matrix Size**: 16×16 native, larger via software tiling
- **Data Types Supported**:
  - **FP16**: 256 TFlops per SM
  - **BF16**: 256 TFlops per SM  
  - **TF32**: 128 TFlops per SM
  - **FP8**: 512 TFlops per SM
  - **INT8**: 512 TOps per SM
  - **INT4**: 1024 TOps per SM
- **Sparsity Support**: 2:4 structured sparsity (50% sparsity)
- **New Features**: FP8 support, enhanced sparsity, improved efficiency

#### RT Cores (3rd Generation)
- **Count**: 2 RT cores per SM (132 total per GPU)
- **Function**: Hardware-accelerated ray tracing
- **Operations**: Ray-triangle intersection, BVH traversal
- **Performance**: 2.5× faster than RT Cores Gen 2
- **New Features**: Improved BVH traversal, opacity micromap support

#### Special Function Units (SFUs)
- **Count**: 4 SFUs per SM
- **Functions**: 
  - Transcendental math (sin, cos, log, exp, sqrt)
  - Reciprocal operations
  - Interpolation functions
  - Type conversion operations
- **Throughput**: 32 operations per clock per SFU
- **Precision**: Full IEEE 754 compliance

### Memory Subsystem Specifications

#### Register File
- **Total Capacity**: 256KB per SM (4 banks × 64KB)
- **Access Pattern**: 4-way banked for conflict-free access
- **Registers per Thread**: Up to 255 32-bit registers
- **Bandwidth**: 64KB per clock cycle
- **Latency**: 1 clock cycle

#### Shared Memory
- **Capacity**: 256KB per SM (configurable with L1)
- **Bank Count**: 32 banks for conflict-free access
- **Bank Width**: 32 bits (4 bytes)
- **Bandwidth**: 256KB per clock cycle
- **Latency**: ~20 clock cycles
- **Access Patterns**: Supports broadcasts, multicasts

#### L1 Data Cache
- **Capacity**: 192KB per SM (configurable with shared memory)
- **Configuration Options**:
  - 128KB L1 + 128KB shared memory
  - 96KB L1 + 160KB shared memory  
  - 64KB L1 + 192KB shared memory
- **Cache Line**: 128 bytes
- **Bandwidth**: 192KB per clock cycle
- **Latency**: ~80 clock cycles

#### Texture Cache
- **Capacity**: 96KB per SM
- **Purpose**: Optimized for 2D/3D spatial locality
- **Features**: Hardware filtering, format conversion
- **Bandwidth**: High throughput for texture operations
- **Access Pattern**: Optimized for graphics workloads

#### Constant Cache
- **Capacity**: 64KB per SM
- **Purpose**: Read-only data with broadcast capability
- **Access Pattern**: Single read broadcasts to entire warp
- **Latency**: ~20 clock cycles for cached data
- **Use Cases**: Shader constants, lookup tables

### Warp Scheduling and Execution

#### Warp Management
- **Warps per SM**: 64 warps maximum (2,048 threads)
- **Active Warps**: 64 warps can be resident simultaneously
- **Warp Schedulers**: 4 schedulers per SM
- **Dispatch Width**: Each scheduler can issue 1 instruction per clock
- **Warp Size**: 32 threads per warp (industry standard)

#### Instruction Pipeline
- **Pipeline Depth**: ~20 stages for arithmetic operations
- **Instruction Issue**: 4 instructions per clock (1 per scheduler)
- **Instruction Types**:
  - Arithmetic: Integer, FP32, FP64
  - Memory: Load, store, atomic operations
  - Control: Branch, predicate, barrier
  - Special: Tensor operations, RT operations

#### Latency Hiding
- **Thread Context**: Zero-overhead context switching
- **Latency Tolerance**: 200+ clock cycles hidden
- **Occupancy**: Up to 2,048 active threads per SM
- **Warp Switching**: Single clock cycle warp switch

### Power and Efficiency Specifications

#### Power Consumption
- **TDP**: 700W total GPU power
- **Power per SM**: ~5.3W per SM (132 SMs total)
- **Voltage**: Multiple voltage domains for optimization
- **Power States**: Multiple P-states for dynamic power management

#### Efficiency Improvements
- **Manufacturing Process**: TSMC 4nm (N4)
- **Transistor Count**: 80 billion transistors
- **Die Size**: 814 mm²
- **Energy Efficiency**: 2.5× improvement over A100
- **Clock Gating**: Fine-grained power management
- **Voltage Scaling**: Dynamic voltage and frequency scaling

## Performance Characteristics

### Computational Performance
- **FP32 Performance**: 60 TFlops (boost clock)
- **Tensor Performance**: 4,000 TFlops (FP8 sparse)
- **RT Performance**: 456 RT Cores operations
- **Memory Bandwidth**: 3TB/s HBM3
- **PCIe Bandwidth**: 128GB/s (PCIe Gen5 x16)

### Scalability Metrics
- **Thread Scalability**: 2,048 threads per SM
- **Memory Scalability**: 3TB/s peak bandwidth
- **Compute Scalability**: 132 SMs total
- **Network Scalability**: 900GB/s NVLink per GPU

### Use Case Optimization
- **AI Training**: Optimized for large language models
- **AI Inference**: High throughput, low latency
- **Scientific Computing**: Double precision, large memory
- **Graphics Rendering**: RT cores, rasterization
- **Data Analytics**: High memory bandwidth, parallel processing 