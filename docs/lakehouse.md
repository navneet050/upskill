
### Object-storage-centric architecture
* this makes sure data is generic and can be processed using different processing engines like- spark, trino, photon etc.

### Open table formats
* this ensures open format related features are available

### Metadata-driven query planning
```
Huge Dataset
     ↓
Metadata narrows partitions
     ↓
Statistics narrow files
     ↓
Parquet narrows columns
     ↓
Predicate pushdown narrows pages
     ↓
Vectorized engine processes efficiently
```
### Decoupled compute/storage
* ensures, you are paying only what you need. Fex. in traditional systems you will get both storage and CPUs. But here if you want to increase storage add more only that.
* 
### Unified analytics + AI platform

### Streaming + batch unification

### Governance

