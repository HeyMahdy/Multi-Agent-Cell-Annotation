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
