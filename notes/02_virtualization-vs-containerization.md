# 02. Virtualization vs Containerization

## Virtualization (Virtual Machines)
A **Virtual Machine (VM)** runs a full copy of an operating system on top of a **Hypervisor**, which sits on the host machine's hardware.

**Structure:**
```
Host Hardware
   └── Hypervisor (VMware, VirtualBox, Hyper-V)
         ├── VM 1: Guest OS + Libraries + App
         ├── VM 2: Guest OS + Libraries + App
         └── VM 3: Guest OS + Libraries + App
```

**Characteristics:**
- Each VM includes its **own full OS** (kernel, drivers, etc.)
- Heavier in size (GBs)
- Slower to boot (minutes)
- Strong isolation (separate kernels)
- Good when you need to run different OS types (e.g., Windows + Linux VMs on the same host)

## Containerization
A **container** shares the **host machine's OS kernel** but keeps the application isolated in its own filesystem, process space, and network.

**Structure:**
```
Host Hardware
   └── Host OS (with Docker Engine)
         ├── Container 1: App + Libraries
         ├── Container 2: App + Libraries
         └── Container 3: App + Libraries
```

**Characteristics:**
- No separate guest OS — shares host kernel
- Lightweight (MBs)
- Boots in seconds
- Slightly less isolated than VMs (shares kernel) but sufficient for most use cases
- Ideal for microservices and fast, scalable deployments

## Key Difference (Quick Comparison)

| Aspect | Virtual Machine | Container |
|---|---|---|
| OS | Full guest OS per VM | Shares host OS kernel |
| Size | GBs | MBs |
| Boot Time | Minutes | Seconds |
| Isolation | Strong (hardware-level) | Process-level |
| Performance | Heavier | Lightweight, near-native |
| Use Case | Running multiple OS types | Running multiple apps/services fast |

## Takeaway
Virtualization virtualizes the **hardware** (each VM gets its own OS).
Containerization virtualizes the **OS** (each container shares the OS but isolates the app).

This is why containers are faster, smaller, and better suited for modern microservices-based architectures.
