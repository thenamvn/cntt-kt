# Visualizations for Thesis

## Figure 1: High-Level System Architecture
This diagram illustrates the "Hub-and-Spoke" topology where the Central Aggregator coordinates the training without accessing the Banking Nodes' private data.

```mermaid
graph TD
    subgraph "Central Aggregator (Cloud/Server)"
        GlobalModel[("Global Model State\n(Weights W_t)")]
        Aggregator{{"Aggregation Engine\n(FedAvg)"}}
    end

    subgraph "Secure Communication Channel (SSL/TLS)"
        Downlink1[Broadcast Weights]
        Downlink2[Broadcast Weights]
        Downlink3[Broadcast Weights]
        Uplink1[Upload Model Updates]
        Uplink2[Upload Model Updates]
        Uplink3[Upload Model Updates]
    end

    subgraph "Banking Node A"
        LocalDB_A[("Private Data A")]
        LocalTrainer_A["Local Training\n(SGD/Adam)"]
    end

    subgraph "Banking Node B"
        LocalDB_B[("Private Data B")]
        LocalTrainer_B["Local Training\n(SGD/Adam)"]
    end

    subgraph "Banking Node C"
        LocalDB_C[("Private Data C")]
        LocalTrainer_C["Local Training\n(SGD/Adam)"]
    end

    GlobalModel --> Aggregator
    Aggregator --"Round t"--> Downlink1 & Downlink2 & Downlink3
    Downlink1 --> LocalTrainer_A
    Downlink2 --> LocalTrainer_B
    Downlink3 --> LocalTrainer_C

    LocalDB_A --> LocalTrainer_A
    LocalDB_B --> LocalTrainer_B
    LocalDB_C --> LocalTrainer_C

    LocalTrainer_A --> Uplink1
    LocalTrainer_B --> Uplink2
    LocalTrainer_C --> Uplink3

    Uplink1 & Uplink2 & Uplink3 --> Aggregator
    Aggregator --"Update"--> GlobalModel
```

## Figure 2: Data Harmonization Pipeline
This diagram details how heterogeneous raw data from different banks is transformed into a standardized feature vector before entering the training loop.

```mermaid
flowchart LR
    rawDB[("Raw Banking DB\n(Oracle/Postgres)")] -->|Extract| cleaning[("Data Cleaning\n(Null Handling)")]
    
    subgraph "Feature Engineering (Standardization Schema)"
        direction TB
        cleaning --> feat1["Transaction Amount\n(Log Normalization)"]
        cleaning --> feat2["Time/Date\n(Cyclical Encoding sin/cos)"]
        cleaning --> feat3["Merchant Category\n(Embedding Mapping)"]
        cleaning --> feat4["Missing Fields\n(Sentinel Value Filling)"]
    end

    feat1 & feat2 & feat3 & feat4 --> vector{{"Standardized\nFeature Vector (X)"}}
    
    vector -->|Input| neuralNet["Local Neural Network\n(Input Layer)"]
    
    style vector fill:#f9f,stroke:#333,stroke-width:2px
```

## Figure 3: Sequence Diagram of a Training Round
This sequence diagram shows the chronological flow of operations for a single Federated Learning round.

```mermaid
sequenceDiagram
    participant Server as Central Aggregator
    participant BankA as Bank Node A
    participant BankB as Bank Node B

    Note over Server: Initialize Round t
    Server->>BankA: Broadcast Global Weights (W_t)
    Server->>BankB: Broadcast Global Weights (W_t)

    rect rgb(240, 248, 255)
        Note left of BankA: Local Phase
        BankA->>BankA: Load W_t into Local Model
        BankA->>BankA: Local Training (E epochs) on Dataset D_a
        BankA->>BankA: Compute New Weights W_a
    end

    rect rgb(240, 248, 255)
        Note right of BankB: Local Phase
        BankB->>BankB: Load W_t into Local Model
        BankB->>BankB: Local Training (E epochs) on Dataset D_b
        BankB->>BankB: Compute New Weights W_b
    end

    BankA->>Server: Upload Update (W_a)
    BankB->>Server: Upload Update (W_b)

    Note over Server: Aggregation Phase
    Server->>Server: W_new = (n_a/N)*W_a + (n_b/N)*W_b
    Server->>Server: Update Global Model State
    Note over Server: Round t+1 Ready
```

## Figure 4: Neural Network Architecture for Fraud Detection
Illustrating the specific definitions of layer discussed in the methodology.

```mermaid
graph LR
    Input[("Input Layer\n(Feature Vector)")] --> Dense1["Dense Layer\n(128 units, ReLU)"]
    Dense1 --> Drop1["Dropout (0.2)"]
    Drop1 --> Dense2["Dense Layer\n(64 units, ReLU)"]
    Dense2 --> Output["Output Layer\n(1 unit, Sigmoid)"]
    Output --> Prediction{{"Prob (0.0 - 1.0)"}}

    style Input fill:#e1f5fe
    style Output fill:#fff9c4
    style Prediction fill:#ffe0b2
```
