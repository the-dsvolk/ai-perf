# NVIDIA H100 NVLink Topology

## Style Definitions

The following class definitions can be reused across all H100 system diagrams for consistent styling:


This diagram illustrates NVLink connections between GPUs and NVSwitches in an NVIDIA H100 system. **H100 GPUs do NOT connect directly to each other** - they only connect to NVSwitches, which provide the full mesh connectivity.

```mermaid
graph TB
    subgraph "NVIDIA H100 NVSwitch Architecture"
        %% GPUs arranged in pairs
        subgraph "GPU Pair 1"
            GPU1[H100 GPU 1<br/>80GB HBM3]
            GPU2[H100 GPU 2<br/>80GB HBM3]
        end
        
        subgraph "GPU Pair 2"
            GPU3[H100 GPU 3<br/>80GB HBM3]
            GPU4[H100 GPU 4<br/>80GB HBM3]
        end
        
        subgraph "GPU Pair 3"
            GPU5[H100 GPU 5<br/>80GB HBM3]
            GPU6[H100 GPU 6<br/>80GB HBM3]
        end
        
        subgraph "GPU Pair 4"
            GPU7[H100 GPU 7<br/>80GB HBM3]
            GPU8[H100 GPU 8<br/>80GB HBM3]
        end

        %% NVSwitches in the center
        subgraph "NVSwitch Fabric"
            NVSwitch1[NVSwitch 1<br/>4th Gen]
            NVSwitch2[NVSwitch 2<br/>4th Gen]
            NVSwitch3[NVSwitch 3<br/>4th Gen]
            NVSwitch4[NVSwitch 4<br/>4th Gen]
        end

        %% NO DIRECT GPU-TO-GPU CONNECTIONS
        %% Only GPU to NVSwitch connections

        %% Each GPU connects to multiple NVSwitches
        %% GPU1 and GPU2 distribute their 18 lanes across all 4 NVSwitches
        GPU1 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch1
        GPU1 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch2
        GPU1 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch3
        GPU1 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch4
        
        GPU2 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch1
        GPU2 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch2
        GPU2 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch3
        GPU2 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch4

        GPU3 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch1
        GPU3 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch2
        GPU3 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch3
        GPU3 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch4
        
        GPU4 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch1
        GPU4 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch2
        GPU4 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch3
        GPU4 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch4

        GPU5 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch1
        GPU5 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch2
        GPU5 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch3
        GPU5 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch4
        
        GPU6 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch1
        GPU6 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch2
        GPU6 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch3
        GPU6 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch4

        GPU7 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch1
        GPU7 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch2
        GPU7 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch3
        GPU7 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch4
        
        GPU8 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch1
        GPU8 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch2
        GPU8 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch3
        GPU8 <-->|"4-5 NVLink lanes<br/>per connection"| NVSwitch4

        %% NVSwitch to NVSwitch connections for full fabric
        NVSwitch1 <-->|"NVSwitch Fabric<br/>Full Mesh"| NVSwitch2
        NVSwitch1 <-->|"NVSwitch Fabric<br/>Full Mesh"| NVSwitch3
        NVSwitch1 <-->|"NVSwitch Fabric<br/>Full Mesh"| NVSwitch4
        NVSwitch2 <-->|"NVSwitch Fabric<br/>Full Mesh"| NVSwitch3
        NVSwitch2 <-->|"NVSwitch Fabric<br/>Full Mesh"| NVSwitch4
        NVSwitch3 <-->|"NVSwitch Fabric<br/>Full Mesh"| NVSwitch4
    end

    style GPU1 fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style GPU2 fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style GPU3 fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style GPU4 fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style GPU5 fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style GPU6 fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style GPU7 fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style GPU8 fill:#76b900,stroke:#333,stroke-width:2px,color:#fff
    style NVSwitch1 fill:#0066cc,stroke:#333,stroke-width:2px,color:#fff
    style NVSwitch2 fill:#0066cc,stroke:#333,stroke-width:2px,color:#fff
    style NVSwitch3 fill:#0066cc,stroke:#333,stroke-width:2px,color:#fff
    style NVSwitch4 fill:#0066cc,stroke:#333,stroke-width:2px,color:#fff
```

## H100 Architecture

### Key Facts About H100 NVLink Topology:
- Each H100 GPU has **18 NVLink 4.0 ports total**
- All 18 ports connect to NVSwitches (distributed across the 4 NVSwitches)
- Each NVLink 4.0 lane provides **50 GB/s bandwidth**
- Total bandwidth per GPU: **900 GB/s** (18 lanes × 50 GB/s)
- NVSwitches provide **full mesh connectivity** between all GPUs
- 4th generation NVSwitches handle all GPU-to-GPU communication

### Architecture:
1. **8 H100 GPUs** connect only to **4 NVSwitches**
2. Each GPU distributes its 18 NVLink lanes across all 4 NVSwitches
3. Typical distribution: **4-5 lanes per NVSwitch** per GPU
4. NVSwitches form a **full mesh fabric** for GPU-to-GPU communication
5. Any GPU can communicate with any other GPU through the NVSwitch fabric

### Bandwidth Distribution:
- **Per GPU**: 18 NVLink lanes × 50 GB/s = **900 GB/s total**
- **Per NVSwitch connection**: 4-5 lanes × 50 GB/s = **200-250 GB/s**
- **Aggregate system bandwidth**: Massive parallel bandwidth through NVSwitch fabric
- **No bottlenecks**: Full mesh ensures optimal communication paths

### Why NVSwitches Are Essential:
- **Scalability**: Support for 8 GPUs with full mesh connectivity
- **Bandwidth**: No bandwidth reduction compared to direct connections
- **Flexibility**: Dynamic routing and load balancing
- **Reliability**: Redundant paths and fault tolerance
- **Simplicity**: Eliminates complex direct GPU wiring

## Connection Details

### GPU-to-NVSwitch Connections Only:
- **Each GPU** connects to **all 4 NVSwitches**
- **18 NVLink lanes per GPU** distributed across NVSwitches
- **No direct GPU-to-GPU connections**
- NVSwitches handle all inter-GPU communication

### NVSwitch Fabric:
- **4 NVSwitches** form a full mesh fabric
- Each NVSwitch connects to all other NVSwitches
- Provides multiple paths for GPU-to-GPU communication
- Enables optimal bandwidth utilization

### Bandwidth Specifications:
- **NVLink 4.0**: 50 GB/s per lane
- **18 lanes per GPU**: 900 GB/s total per GPU
- **Lane distribution**: 4-5 lanes per NVSwitch per GPU
- **Total system bandwidth**: Massive aggregate through fabric

### Topology Benefits:
- **True full mesh**: Every GPU can communicate with every other GPU
- **No CPU involvement**: Direct GPU-to-GPU transfers through NVSwitch
- **Optimal bandwidth**: No reduction compared to direct connections  
- **Scalable architecture**: Clean, manageable topology
- **High reliability**: Multiple paths and redundancy
- **Load balancing**: NVSwitches optimize traffic routing 