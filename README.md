
```markdown
# Hybrid-scGPT: Agentic Cell Type Annotation Pipeline with LangGraph

An advanced, multi-agent hybrid architecture designed for single-cell RNA-seq (scRNA-seq) cell type annotation. This pipeline combines the deep representation power of **scGPT** (a foundational transformer model for single-cell biology) with a **LangGraph-driven multi-agent system** that automatically adjudicates low-confidence predictions using domain-specific tool integration and LLM reasoning.

---

## 🚀 System Architecture

The pipeline is designed as an event-driven state machine managed by LangGraph. It runs linear preprocessing and embedding steps before entering a controlled conditional loop to evaluate and patch uncertain model classifications.

### Pipeline Flow Diagram

```mermaid
graph TD
    %% Nodes Configuration
    raw_dataset[📁 1. raw_dataset]
    dataset_processing[⚙️ 2. dataset_processing]
    scgpt[🧬 3. scgpt Embedding]
    prediction[🔮 4. ML Prediction]
    confidence_check[🛡️ 5. confidence_check]
    init_teacher_queue[📋 6. init_teacher_queue]
    prepare_teacher_cell[🔬 7. prepare_teacher_cell]
    teacher_agent[🤖 8. teacher_agent Adjudication]
    merge_teacher[🔀 9. merge_teacher]
    metrics[📊 10. metrics Evaluation]
    END(((🛑 END)))

    %% Sequential Flow
    raw_dataset --> dataset_processing
    dataset_processing --> scgpt
    scgpt --> prediction
    prediction --> confidence_check
    confidence_check --> init_teacher_queue

    %% Conditional Control Routing
    init_teacher_queue -- "route_teacher_loop (Has Low Conf Cells)" --> prepare_teacher_cell
    init_teacher_queue -- "route_teacher_loop (Queue Empty)" --> metrics
    
    %% Processing Loop
    prepare_teacher_cell --> teacher_agent
    teacher_agent --> merge_teacher
    merge_teacher -- "route_teacher_loop" --> prepare_teacher_cell
    merge_teacher -- "route_teacher_loop (Finished Queue)" --> metrics

    %% Terminal Exit
    metrics --> END

    %% Custom Styling
    style raw_dataset fill:#f9f9f9,stroke:#333,stroke-width:1px
    style confidence_check fill:#fff3cd,stroke:#ffc107,stroke-width:2px
    style teacher_agent fill:#d1ecf1,stroke:#17a2b8,stroke-width:2px
    style metrics fill:#d4edda,stroke:#28a745,stroke-width:2px

```

---

## 🛠️ Core Concepts & Components

### 1. Embeddings & Classification Layer (`scGPT` + `MLP`)

* **scGPT Integration:** Reads raw gene expression counts from specialized datasets (e.g., Pollen dataset) and passes data through the scGPT transformer model to derive high-dimensional, biologically expressive embeddings.
* **Classification:** A custom PyTorch Multi-Layer Perceptron (MLP) trained on the generated embeddings outputs soft probability distributions across possible cell types.

### 2. Confidence Gating (`confidence_check`)

* Every single-cell prediction is checked against a configurable accuracy threshold ($\text{Confidence Threshold} = 0.995$).
* Cells meeting or exceeding this threshold are pushed straight through to the final evaluation metrics.
* Cells with sub-threshold probabilities are automatically flagged as **low-confidence targets** and lined up in a sequential review queue.

### 3. Tool-Augmented LLM Adjudication Loop (`teacher_agent`)

Instead of blindly accepting weak neural network classifications, the agent invokes a specialized corrective loop for low-confidence cells:

* **Gene Retrieval Tool (`get_top_expressed_genes`):** Directly queries the active `AnnData` structure to isolate the cell's top 20 most highly expressed genes.
* **CellMarker Database Tool (`query_cellmarker_db`):** Cross-references those specific marker genes directly against a PostgreSQL implementation of the **CellMarker Database** to extract validated, empirical biological tissue markers.
* **LLM Decision Node (`teacher_agent_node`):** A context-aware Language Model acts as an expert biological adjudicator. Armed with the base model's guess, the top gene markers, and the verified database matches, it determines whether to override the MLP model prediction with verified database ground truth or fallback safely to the baseline guess.

---

## 🧬 LangGraph State Graph Implementation

The pipeline state is tracked immutably through a shared `PipelineState` context object containing fields like `adata`, `embeddings`, `confidence_gate`, and `teacher_results`.

```python
# Architecture definition excerpt
graph = StateGraph(PipelineState)

# Node registration
graph.add_node("raw_dataset", raw_dataset_node)
graph.add_node("dataset_processing", dataset_processing_node)
graph.add_node("scgpt", scgpt_node)
graph.add_node("prediction", prediction_node)
graph.add_node("confidence_check", confidence_check_node)
graph.add_node("init_teacher_queue", init_teacher_queue_node)
graph.add_node("prepare_teacher_cell", prepare_teacher_cell_node)
graph.add_node("teacher_agent", teacher_agent_node)
graph.add_node("merge_teacher", merge_teacher_result_node)
graph.add_node("metrics", metrics_node)

# Execution Flow Mapping
graph.set_entry_point("raw_dataset")
graph.add_edge("raw_dataset", "dataset_processing")
graph.add_edge("dataset_processing", "scgpt")
graph.add_edge("scgpt", "prediction")
graph.add_edge("prediction", "confidence_check")
graph.add_edge("confidence_check", "init_teacher_queue")

# Loop routing definitions
graph.add_conditional_edges("init_teacher_queue", route_teacher_loop)
graph.add_edge("prepare_teacher_cell", "teacher_agent")
graph.add_edge("teacher_agent", "merge_teacher")
graph.add_conditional_edges("merge_teacher", route_teacher_loop)

graph.add_edge("metrics", END)

```

---

## 📋 Environment Configuration & Requirements

Before starting the pipeline, configure the following dependencies and environmental keys:

```bash
# Core execution paths
export DATA_DIR="path/to/pollen_dataset"
export MODEL_DIR="path/to/scgpt_checkpoint"

# Database connection for cellmarker validations
export DB_URL="postgresql://user:password@host:port/dbname"

# LLM Adjudication configuration
export OPENAI_API_KEY="your-openai-api-key"

```

### Installation

```bash
pip install torch scanpy scgpt langgraph langchain_core scikit-learn psycopg2-binary gdown

```

---

## 📊 Evaluation Metrics

The pipeline concludes execution at the `metrics` node, generating comparative diagnostic evaluation matrices (Macro/Weighted Precision, Recall, and F1-Scores) comparing the **Raw Baseline Model (scGPT alone)** against the **Hybrid Pipeline (scGPT + Multi-Agent Loop)** to illustrate the accuracy lift achieved via agentic intervention.

```

```
