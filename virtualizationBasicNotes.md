# Virtualization Basic Notes

<!-- TOC -->

- [Virtualization Basic Notes](#virtualization-basic-notes)
    - [Memory Management](#memory-management)
        - [`MICRO'15` Large Pages and Lightweight Memory Management in Virtualized Environments](#micro15-large-pages-and-lightweight-memory-management-in-virtualized-environments)
            - [Shortcomings of Large Pages](#shortcomings-of-large-pages)
            - [GLUE (Generalized Large-page Utilization Enhancements)](#glue-generalized-large-page-utilization-enhancements)

<!-- /TOC -->

## Memory Management

### `MICRO'15` Large Pages and Lightweight Memory Management in Virtualized Environments

#### Shortcomings of Large Pages

- large pages can reduce consolidation in real-world cloud deployments where memory resources are over-committed by precluding opportunities for page sharing
- large pages can also interfere with a hypervisor’s ability to perform lightweight guest memory usage monitoring (its ability to effectively allocate and place data in non-uniform memory access systems)
- large pages can hamper operations like agile VM migration

As a result, virtualization software often break large pages into smaller baseline pages (lost opportunity in terms of reducing address translation overheads)

#### GLUE (Generalized Large-page Utilization Enhancements)

- GLUE augments standard TLBs to store information that identifies these contiguous, aligned, but splintered regions
- GLUE then uses TLB speculation to identify the constituent translations
- Small system physical pages are speculated by interpolating around the information stored about a single speculative large-page translation in the TLB