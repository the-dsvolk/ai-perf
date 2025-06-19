# NVIDIA H100 NVLink Architecture

This diagram illustrates the NVLink connections between GPUs in an NVIDIA H100 system. The H100 uses NVLink 4.0, which provides high-speed GPU-to-GPU communication.

## Complete H100 System Architecture

The following diagram shows the complete H100 system architecture including CPUs, GPUs, PCIe fabric, NVSwitch, and network connectivity.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'fontSize': '22px'}, 'flowchart': {'nodeSpacing': 80, 'rankSpacing': 120}, 'themeCSS': 'svg { max-width: 100% !important; width: 1600px !important; height: 1200px !important; }'}}%%
flowchart LR
    subgraph "H100 System Architecture"
        %% CPUs on the left
        subgraph "CPU Layer"
            direction TB
            CPU1[Intel Xeon 8480C\n56 cores\n2.0/2.9/3.8 GHz]
            CPU2[Intel Xeon 8480C\n56 cores\n2.0/2.9/3.8 GHz]
        end

        %% Memory next to CPUs
        subgraph "Memory Layer"
            direction TB
            MEM1[2TB System Memory]
        end

        %% Storage next to Memory
        subgraph "Storage Layer"
            direction TB
            subgraph "Storage Subsystem\n(Data Cache)"
                STORAGE[8x 3.84TB\nNVMe U.2 SED\nRAID 0]
            end
        end

        %% PCIe bus next to Storage
        PCIe[PCIe Gen5 Fabric]

        %% Network adapters
        subgraph "Network Layer"
            direction TB
            subgraph "Communication Network\n(4x OSFP ports)"
                IB1[ConnectX-7\nInfiniBand 400Gbps]
                IB2[ConnectX-7\nInfiniBand 400Gbps]
                IB3[ConnectX-7\nInfiniBand 400Gbps]
                IB4[ConnectX-7\nInfiniBand 400Gbps]
            end
            
            subgraph "Storage Network\n(2x Dual Port)"
                ETH1[ConnectX-7\nDual Port\n400GbE]
                ETH2[ConnectX-7\nDual Port\n400GbE]
            end
        end

        %% GPUs at the bottom
        subgraph "GPU Layer"
            direction LR
            GPU1[H100 GPU 1\n80GB HBM3]
            GPU2[H100 GPU 2\n80GB HBM3]
            GPU3[H100 GPU 3\n80GB HBM3]
            GPU4[H100 GPU 4\n80GB HBM3]
            GPU5[H100 GPU 5\n80GB HBM3]
            GPU6[H100 GPU 6\n80GB HBM3]
            GPU7[H100 GPU 7\n80GB HBM3]
            GPU8[H100 GPU 8\n80GB HBM3]

            %% NVSwitch
            NVSwitch1[NVSwitch 1\n4th Gen]
            NVSwitch2[NVSwitch 2\n4th Gen]
            NVSwitch3[NVSwitch 3\n4th Gen]
            NVSwitch4[NVSwitch 4\n4th Gen]
        end

        %% Connections from CPUs to Memory
        CPU1 <--> MEM1
        CPU2 <--> MEM1

        %% Connections from Memory to Storage
        MEM1 --> STORAGE

        %% Connections from Storage to PCIe
        STORAGE --> PCIe

        %% Connections from Network to PCIe
        IB1 --> PCIe
        IB2 --> PCIe
        IB3 --> PCIe
        IB4 --> PCIe
        ETH1 --> PCIe
        ETH2 --> PCIe

        %% Connections from GPUs to PCIe
        GPU1 --> PCIe
        GPU2 --> PCIe
        GPU3 --> PCIe
        GPU4 --> PCIe
        GPU5 --> PCIe
        GPU6 --> PCIe
        GPU7 --> PCIe
        GPU8 --> PCIe

        %% GPU to NVSwitch Connections
        GPU1 <-->|"NVLink 4.0\n300 GB/s"| NVSwitch1
        GPU2 <-->|"NVLink 4.0\n300 GB/s"| NVSwitch1
        GPU3 <-->|"NVLink 4.0\n300 GB/s"| NVSwitch2
        GPU4 <-->|"NVLink 4.0\n300 GB/s"| NVSwitch2
        GPU5 <-->|"NVLink 4.0\n300 GB/s"| NVSwitch3
        GPU6 <-->|"NVLink 4.0\n300 GB/s"| NVSwitch3
        GPU7 <-->|"NVLink 4.0\n300 GB/s"| NVSwitch4
        GPU8 <-->|"NVLink 4.0\n300 GB/s"| NVSwitch4

        %% NVSwitch to NVSwitch Connections
        NVSwitch1 <-->|"NVLink 4.0\n300 GB/s"| NVSwitch2
        NVSwitch2 <-->|"NVLink 4.0\n300 GB/s"| NVSwitch3
        NVSwitch3 <-->|"NVLink 4.0\n300 GB/s"| NVSwitch4
    end

    %% Styling with improved text contrast
    classDef cpu fill:#ff9900,stroke:#333,stroke-width:3px,color:#fff,font-weight:bold
    classDef memory fill:#66ccff,stroke:#333,stroke-width:3px,color:#000,font-weight:bold
    classDef gpu fill:#76b900,stroke:#333,stroke-width:3px,color:#fff,font-weight:bold
    classDef nvswitch fill:#0066cc,stroke:#333,stroke-width:3px,color:#fff,font-weight:bold
    classDef infiniband fill:#cc0000,stroke:#333,stroke-width:3px,color:#fff,font-weight:bold
    classDef ethernet fill:#990000,stroke:#333,stroke-width:3px,color:#fff,font-weight:bold
    classDef storage fill:#663399,stroke:#333,stroke-width:3px,color:#fff,font-weight:bold
    classDef pcie fill:#666666,stroke:#333,stroke-width:3px,color:#fff,font-weight:bold

    class CPU1,CPU2 cpu
    class MEM1 memory
    class GPU1,GPU2,GPU3,GPU4,GPU5,GPU6,GPU7,GPU8 gpu
    class NVSwitch1,NVSwitch2,NVSwitch3,NVSwitch4 nvswitch
    class IB1,IB2,IB3,IB4 infiniband
    class ETH1,ETH2 ethernet
    class STORAGE storage
    class PCIe pcie
```

## Key Features

- Each H100 GPU has 18 NVLink 4.0 lanes
- Each NVLink 4.0 lane provides 50 GB/s bandwidth
- Each GPU-to-GPU connection uses 6 lanes (300 GB/s)
- Total NVLink bandwidth of up to 900 GB/s per GPU
- Full mesh topology for optimal communication
- Direct GPU-to-GPU communication without CPU intervention
- Support for up to 8 GPUs in a single system

## System Architecture Details

- **CPUs**: 2x Intel Xeon 8480C PCIe Gen5 CPUs
  - 56 cores each
  - 2.0/2.9/3.8 GHz (base/all core turbo/Max turbo)
- **System Memory**: 2TB total
- **GPUs**: 8x NVIDIA H100 GPUs
  - 80GB HBM3 memory each
  - 640GB total GPU memory
- **NVSwitch**: 4x 4th generation NVSwitches
  - 900 GB/s GPU-to-GPU bandwidth
- **Storage**:
  - 8x 3.84 TB NVMe U.2 SED drives
  - Configured in RAID 0 array
- **Network**:
  - **Communication Network** (4x OSFP ports):
    - 8x NVIDIA ConnectX-7 Single Port InfiniBand Cards
    - InfiniBand: Up to 400Gbps
    - Ethernet: 400GbE, 200GbE, 100GbE, 50GbE, 40GbE, 25GbE, 10GbE
  - **Storage Network** (2x Dual Port):
    - 2x NVIDIA ConnectX-7 Dual Port Ethernet Cards
    - Ethernet: 400GbE, 200GbE, 100GbE, 50GbE, 40GbE, 25GbE, 10GbE
    - InfiniBand: Up to 400Gbps
- **PCIe Fabric**: Gen5 PCIe providing 64 GB/s per connection

## Notes

- The diagram shows the exact number of NVLink lanes between each GPU pair
- Each connection represents 6 NVLink lanes, providing 300 GB/s bandwidth
- The total bandwidth per GPU is distributed across multiple connections
- The actual physical layout may vary depending on the specific H100 system configuration
- NVLink 4.0 provides improved power efficiency and reliability compared to previous generations
- The system supports both direct GPU-to-GPU communication via NVLink and CPU-mediated communication via PCIe
- **Communication Network** provides high-speed InfiniBand connectivity for GPU-to-GPU communication across nodes
- **Storage Network** provides dedicated high-speed Ethernet connectivity for storage access
- The storage subsystem uses NVMe U.2 SED drives in a RAID 0 configuration for maximum performance 


## NVLink Topology Details

- Each H100 GPU has 18 NVLink 4.0 lanes
- Each NVLink 4.0 lane provides 50 GB/s bandwidth
- Each GPU-to-GPU connection uses 6 lanes (300 GB/s)
- Total NVLink bandwidth of up to 900 GB/s per GPU
- Full mesh topology for optimal communication
- Direct GPU-to-GPU communication without CPU intervention

### Connection Types


2. **GPU-to-NVSwitch Connections**:
   - Each GPU pair connects to one NVSwitch
   - Each connection provides 300 GB/s bandwidth

3. **NVSwitch-to-NVSwitch Connections**:
   - NVSwitch1 ↔ NVSwitch2
   - NVSwitch2 ↔ NVSwitch3
   - NVSwitch3 ↔ NVSwitch4
   - Enables communication between all GPUs 

## System Infrastructure Architecture

The following diagram illustrates the system infrastructure components including CPUs, Memory, PCI Bus, and Network connectivity.

```mermaid
flowchart LR
    subgraph "System Infrastructure"
        %% CPUs on the left with caches
        subgraph "CPU Layer"
            direction TB
            subgraph "CPU 1"
                CPU1[Intel Xeon 8480C\n56 cores\n2.0/2.9/3.8 GHz]
                L1_1[L1 Cache\n56x 48KB\nper core]
                L2_1[L2 Cache\n56x 2MB\nper core]
                L3_1[L3 Cache\n105MB\nshared]
            end
            subgraph "CPU 2"
                CPU2[Intel Xeon 8480C\n56 cores\n2.0/2.9/3.8 GHz]
                L1_2[L1 Cache\n56x 48KB\nper core]
                L2_2[L2 Cache\n56x 2MB\nper core]
                L3_2[L3 Cache\n105MB\nshared]
            end
        end

        %% Memory next to CPUs
        subgraph "Memory Layer"
            direction TB
            MEM1[2TB System Memory\n8x DDR5-4800\n307.2 GB/s bandwidth]
        end

        %% Storage next to Memory
        subgraph "Storage Layer"
            direction TB
            subgraph "Storage Subsystem\n(Data Cache)"
                STORAGE[8x 3.84TB\nNVMe U.2 SED\nRAID 0\n28 GB/s read\n12 GB/s write]
            end
        end

        %% PCIe bus in the middle
        PCIe[PCIe Gen5 Fabric\n64 GB/s per connection]

        %% Network adapters
        subgraph "Network Layer"
            direction TB
            subgraph "Communication Network\n(4x OSFP ports)"
                IB1[ConnectX-7\nInfiniBand 400Gbps]
                IB2[ConnectX-7\nInfiniBand 400Gbps]
                IB3[ConnectX-7\nInfiniBand 400Gbps]
                IB4[ConnectX-7\nInfiniBand 400Gbps]
            end
            
            subgraph "Storage Network\n(2x Dual Port)"
                ETH1[ConnectX-7\nDual Port\n400GbE]
                ETH2[ConnectX-7\nDual Port\n400GbE]
            end
        end

        %% Connections from CPUs to Memory
        CPU1 <-->|"307.2 GB/s"| MEM1
        CPU2 <-->|"307.2 GB/s"| MEM1

        %% Connections from Memory to Storage
        MEM1 -->|"28 GB/s read\n12 GB/s write"| STORAGE

        %% Connections from Storage to PCIe
        STORAGE -->|"PCIe Gen4 x4\n8 GB/s"| PCIe

        %% Connections from Network to PCIe
        IB1 -->|"PCIe Gen5 x16\n64 GB/s"| PCIe
        IB2 -->|"PCIe Gen5 x16\n64 GB/s"| PCIe
        IB3 -->|"PCIe Gen5 x16\n64 GB/s"| PCIe
        IB4 -->|"PCIe Gen5 x16\n64 GB/s"| PCIe
        ETH1 -->|"PCIe Gen5 x16\n64 GB/s"| PCIe
        ETH2 -->|"PCIe Gen5 x16\n64 GB/s"| PCIe
    end

    %% Styling
    classDef cpu fill:#ff9900,stroke:#333,stroke-width:2px
    classDef cache fill:#ff9966,stroke:#333,stroke-width:2px
    classDef memory fill:#66ccff,stroke:#333,stroke-width:2px
    classDef infiniband fill:#cc0000,stroke:#333,stroke-width:2px
    classDef ethernet fill:#990000,stroke:#333,stroke-width:2px
    classDef storage fill:#663399,stroke:#333,stroke-width:2px
    classDef pcie fill:#666666,stroke:#333,stroke-width:2px

    class CPU1,CPU2 cpu
    class L1_1,L2_1,L3_1,L1_2,L2_2,L3_2 cache
    class MEM1 memory
    class IB1,IB2,IB3,IB4 infiniband
    class ETH1,ETH2 ethernet
    class STORAGE storage
    class PCIe pcie
```

## System Infrastructure Details

### CPU Configuration
- 2x Intel Xeon 8480C PCIe Gen5 CPUs
  - 56 cores each
  - 2.0/2.9/3.8 GHz (base/all core turbo/Max turbo)
  - Cache Hierarchy:
    - L1 Cache: 48KB per core (56 cores)
    - L2 Cache: 2MB per core (56 cores)
    - L3 Cache: 105MB shared

### Memory Configuration
- 2TB total system memory
- 8x DDR5-4800 DIMMs
- Memory bandwidth: 307.2 GB/s per CPU
- Direct connection to both CPUs

### Storage Configuration
- 8x 3.84 TB NVMe U.2 SED drives
- Configured in RAID 0 array
- Performance:
  - Read bandwidth: 28 GB/s
  - Write bandwidth: 12 GB/s
- Connected to system memory and PCIe fabric
- PCIe Gen4 x4 interface (8 GB/s per drive)

### Network Configuration
1. **Communication Network** (4x OSFP ports):
   - 4x NVIDIA ConnectX-7 Single Port InfiniBand Cards
   - InfiniBand: Up to 400Gbps
   - Ethernet: 400GbE, 200GbE, 100GbE, 50GbE, 40GbE, 25GbE, 10GbE
   - PCIe Gen5 x16 interface (64 GB/s)

2. **Storage Network** (2x Dual Port):
   - 2x NVIDIA ConnectX-7 Dual Port Ethernet Cards
   - Ethernet: 400GbE, 200GbE, 100GbE, 50GbE, 40GbE, 25GbE, 10GbE
   - InfiniBand: Up to 400Gbps
   - PCIe Gen5 x16 interface (64 GB/s)

### PCIe Fabric
- Gen5 PCIe providing 64 GB/s per connection
- Central connection point for all components
- Enables high-speed communication between all system components 