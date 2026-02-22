# Section 3: Storing the Data

**Exam Weight: ~25%**

One of the highest-weighted sections. Covers selecting the right storage technology
for each use case, managing data lakes, and designing unified data platforms.

## Subsections

| Subsection | Topic | File |
|------------|-------|------|
| 3.1 | [Selecting storage systems](./3.1-selecting-storage-systems.md) | Choosing managed GCP storage services |
| 3.3 | [Using a data lake](./3.3-using-a-data-lake.md) | Data lake management, discovery, access, cost |
| 3.4 | [Designing for a data platform](./3.4-designing-for-a-data-platform.md) | Unified platforms, federated governance |

> **Note:** Subsection 3.2 is not included in the renewal exam guide.

## Key Themes

- **Service selection**: Matching storage characteristics (latency, consistency,
  scalability, query pattern) to use case
- **Data lake management**: Configuring Dataplex for access control, discovery, and cost
- **Federated governance**: Governing distributed data across teams and platforms
- **BigLake**: Unified storage layer over Cloud Storage and BigQuery

## Storage Selection Quick Reference

| Use Case | Recommended Service |
|----------|-------------------|
| Analytical queries, DW | BigQuery |
| Time-series, high-throughput NoSQL | Bigtable |
| Globally distributed OLTP (relational) | Spanner |
| Regional OLTP (relational) | Cloud SQL or AlloyDB |
| Unstructured files, data lake | Cloud Storage |
| Document store, mobile | Firestore |
| In-memory caching | Memorystore |
| Unified lakehouse | BigLake |

## Section Progress

- [ ] 3.1 Selecting storage systems
- [ ] 3.3 Using a data lake
- [ ] 3.4 Designing for a data platform
