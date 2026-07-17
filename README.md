# Azure-Databricks-Lakehouse

## Overview

Developed as part of the NashTech Data Engineering Program, this project implements an end-to-end Azure Lakehouse that processes CSV data through the Medallion Architecture (Bronze → Silver → Gold).

---

# Solution Architecture

```mermaid
flowchart TD

A[CSV Source Files] --> B[Azure Data Factory Pipeline]

B --> C[Landing Zone<br/>ADLS Gen2]

C --> D[Databricks Bronze Notebook]

D --> E[Auto Loader]

E --> F[Bronze Delta Tables]

F --> G[Silver Transformation]

G --> H[Silver Delta Tables]

H --> I[Gold Aggregation]

I --> J[Gold Delta Tables]

J --> K[Analytics / BI / SQL]
```

---

# Medallion Architecture

The project follows the standard Lakehouse Medallion pattern.

```mermaid
flowchart LR

A[Landing CSV Files]

A --> B[Bronze]

B --> C[Silver]

C --> D[Gold]

B -->|"Raw ingestion"| B1[(Delta)]
C -->|"Clean + Deduplicate"| C1[(Delta)]
D -->|"Business Aggregation"| D1[(Delta)]
```

---

# Pipeline Orchestration

Azure Data Factory orchestrates the complete workflow.

```mermaid
flowchart LR

P[ADF Pipeline]

P --> B[Bronze Notebook]

B --> S[Silver Notebook]

S --> G[Gold Notebook]
```

Each notebook receives parameters from ADF.

---

# Parameterized Design

Instead of hardcoding storage paths, the pipeline uses a single parameter:

```
p_trainee_id
```

Example:

```
rookie01
rookie02
rookie03
...
```

ADF dynamically builds storage paths such as:

```
landing/<trainee>/orders/

bronze/<trainee>/orders/

silver/<trainee>/orders/

gold/<trainee>/orders/
```

---

# Shared Training Environment

To reduce Azure costs, infrastructure resources are shared while data remains isolated.

```mermaid
flowchart TB

subgraph Shared Azure Resources

RG[Resource Group]

SA[ADLS Gen2]

DBW[Databricks Workspace]

CL[Shared Cluster]

end

subgraph Trainees

R1[rookie01]

R2[rookie02]

R3[rookie03]

RN[...]

end

R1 --> SA
R2 --> SA
R3 --> SA
RN --> SA

R1 --> DBW
R2 --> DBW
R3 --> DBW
RN --> DBW
```

Each participant works within their own folder hierarchy to prevent conflicts while sharing the same compute resources.

---

# Data Processing Flow

```mermaid
sequenceDiagram

participant User
participant ADF
participant ADLS
participant Databricks
participant Delta

User->>ADF: Run Pipeline

ADF->>Databricks: Execute Bronze Notebook

Databricks->>ADLS: Read CSV Files

Databricks->>Delta: Write Bronze Delta

ADF->>Databricks: Execute Silver Notebook

Databricks->>Delta: Read Bronze

Databricks->>Delta: Clean & Deduplicate

Databricks->>Delta: Write Silver

ADF->>Databricks: Execute Gold Notebook

Databricks->>Delta: Aggregate

Databricks->>Delta: Write Gold
```

---

# Technology Stack

| Component                    | Purpose                                    |
| ---------------------------- | ------------------------------------------ |
| Azure Data Factory           | Pipeline orchestration                     |
| Azure Data Lake Storage Gen2 | Data lake storage                          |
| Azure Databricks             | Distributed data processing                |
| Auto Loader                  | Incremental file ingestion                 |
| Delta Lake                   | Reliable data storage with ACID guarantees |
| PySpark                      | Data transformation                        |
| Parameterized Pipelines      | Reusable workflow orchestration            |

---
