---
{"dg-publish":true,"permalink":"/whoami/","title":"RISC-V Sv39 MMU RTL Design","tags":["RISCV","MMU","RTL","Sv39","TLB","gardenEntry"],"noteIcon":"","created":"2025-01-11T17:46:52.255+05:30","updated":"2025-01-12T17:17:28.246+05:30"}
---



# RISC-V Sv39 MMU RTL Design Documentation

This document provides a detailed overview of the RTL design for a RISC-V Sv39 Memory Management Unit (MMU). The MMU design consists of the following components:

- **Instruction Translation Lookaside Buffer (ITLB)**
- **Data Translation Lookaside Buffer (DTLB)**
- **Main Unified Translation Lookaside Buffer (UTLB)**
- **Page Table Walk Unit (PTW)**

## 1. Overview

The RISC-V Sv39 MMU supports a 39-bit virtual address space and a 64-bit physical address space. It implements a three-level page table structure and is responsible for virtual-to-physical address translation and memory protection. The MMU is designed for high-performance virtual memory management with hardware support for TLB caching.

---

## 2. Components Description

### 2.1 Instruction Translation Lookaside Buffer (ITLB)

**Purpose:**  
The ITLB caches recently translated instruction addresses to reduce page table walks for instruction fetches.

**Key Features:**  
- Directly interfaced with the instruction fetch unit.  
- Holds a small number of instruction page translations.  
- Lookup based on the virtual address (VPN).  
- On a miss, it forwards the request to the **Unified TLB (UTLB)**.

**RTL Signals:**  
- `itlb_req`: Instruction fetch request.  
- `itlb_vaddr`: Virtual address for lookup.  
- `itlb_hit`: Hit/miss status.  
- `itlb_paddr`: Translated physical address on a hit.  

---

### 2.2 Data Translation Lookaside Buffer (DTLB)

**Purpose:**  
The DTLB caches recently translated data addresses for load/store operations.

**Key Features:**  
- Directly interfaced with the load/store unit.  
- Holds a small number of data page translations.  
- Lookup based on the virtual address (VPN).  
- On a miss, it forwards the request to the **Unified TLB (UTLB)**.

**RTL Signals:**  
- `dtlb_req`: Load/store request.  
- `dtlb_vaddr`: Virtual address for lookup.  
- `dtlb_hit`: Hit/miss status.  
- `dtlb_paddr`: Translated physical address on a hit.  

---

### 2.3 Unified Translation Lookaside Buffer (UTLB)

**Purpose:**  
The Unified TLB (UTLB) serves as the main MMU cache and handles misses from both the ITLB and DTLB.

**Key Features:**  
- Larger capacity compared to ITLB and DTLB.  
- Contains translations for both instruction and data pages.  
- Handles multiple page sizes: 4KB, 2MB, and 1GB.  
- On a miss, it triggers a **Page Table Walk (PTW)**.  

**RTL Signals:**  
- `utlb_req`: Request from ITLB/DTLB.  
- `utlb_vaddr`: Virtual address for lookup.  
- `utlb_hit`: Hit/miss status.  
- `utlb_paddr`: Translated physical address on a hit.  

---

### 2.4 Page Table Walk Unit (PTW)

**Purpose:**  
The PTW performs hardware page table walks when a miss occurs in the UTLB.

**Key Features:**  
- Follows the Sv39 page table structure.  
- Walks up to 3 levels of page tables.  
- Accesses memory for page table entries (PTEs).  
- Caches results in the UTLB upon completion.  

**RTL Signals:**  
- `ptw_start`: PTW start signal triggered by a TLB miss.  
- `ptw_vaddr`: Virtual address for page table walk.  
- `ptw_pte`: Resulting PTE from the walk.  
- `ptw_done`: PTW completion signal.  

---

## 3. Address Translation Flow

1. **Instruction/Data Fetch:**  
   - ITLB/DTLB receives a virtual address.  
   - If a hit occurs, the physical address is returned immediately.  
   - If a miss occurs, the request is forwarded to the UTLB.  

2. **Unified TLB Lookup:**  
   - If a hit occurs in the UTLB, the physical address is returned.  
   - If a miss occurs, a page table walk is triggered.  

3. **Page Table Walk (PTW):**  
   - PTW fetches the PTEs from memory.  
   - The resolved translation is cached in the UTLB.  

---

## 4. RTL Design Considerations

### 4.1 Page Size Support
- 4KB pages  
- 2MB pages (huge pages)  
- 1GB pages (super pages)  

### 4.2 TLB Replacement Policy
- ITLB and DTLB use an LRU (Least Recently Used) policy.  
- UTLB may use a set-associative design with LRU.  

### 4.3 Memory Protection
- PTE permissions are enforced: `R`, `W`, `X`, and `U` bits.  
- Page faults triggered on invalid access.  

---

## 5. RTL Integration and Verification

### 5.1 Integration
- The MMU interfaces with the instruction fetch unit, load/store unit, and memory subsystem.  
- Clock and reset signals are synchronized across all MMU components.  

### 5.2 Verification
- **Unit Testing:** Individual testing of ITLB, DTLB, UTLB, and PTW units.  
- **System Testing:** Full MMU verification with a RISC-V core.  
- **Coverage:** Code and functional coverage metrics for TLB hits/misses, PTW correctness, and permission enforcement.  

---

## 6. Conclusion
The RISC-V Sv39 MMU design described here is optimized for performance with a hierarchical TLB structure and a hardware page table walk unit. This design ensures efficient virtual-to-physical address translation while adhering to the RISC-V privileged specification.

---
