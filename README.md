# RespubLit.ai: Core Evaluation Engine Architecture

RespubLit.ai is a high-performance, local-first evaluation pipeline engineered to address the systemic challenges of modern research funding administration. The platform operates across two primary workflows:
* **Phase I** matches complex academic research proposals with highly qualified, conflict-free expert reviewers, specifically targeting a final roster of the 60 best-fit candidates.
* **Phase II** conducts automated, large-scale quality audits on the submitted peer-review reports.

Optimized for local infrastructure, the system leverages hardware-accelerated C++ execution to run advanced language models efficiently at scale.

![System Architecture Diagram](docs/images/system-architecture.png)
*(Note: Add your architecture diagram image here)*

## Key Technical Capabilities

* **Hardware-Accelerated LLM Integration:** The pipeline utilizes `llama.cpp` to execute advanced GGUF models directly on the GPU. It leverages custom LoRA adapters trained on state-of-the-art base architectures, utilizing Mistral-NeMo for the PeerAtlas 12B extraction, reranking, and report-auditing modules, alongside Qwen for dense semantic embeddings.
* **Modular Semantic Engine:** The architecture is highly flexible, allowing the semantic engine to seamlessly swap between 3B, 4B, and 7B embedding models. This ensures the pipeline can be tailored to the specific context, computational constraints, and scale demands of any given review campaign.
* **Local-First Data and Multi-Source Enrichment:** To ensure strict data privacy and rapid execution, the system queries massive local OpenAlex datasets via local Parquet files, utilizing Tantivy and DuckDB Full-Text Search (FTS) indices for millisecond retrieval times. For robust Conflict of Interest (COI) screening, it cross-checks identifiers, affiliations, and co-authorship traces by federating searches across external APIs including OpenAlex, Crossref, PubMed, arXiv, and Semantic Scholar.
* **Advanced COI Detection:** The system models complex academic networks using `networkx` to compute graph distances between applicants and candidates, flagging potential conflicts based on shared institutions, historical co-authorships, and overlapping grant funding.

---

## Phase I: Reviewer Selection and Discovery Pipeline

1. **Document Parsing and Metadata Extraction:** The engine ingests raw proposal documents (PDF, DOCX) and utilizes GROBID (or fallback heuristics) to extract structured text, reference bibliographies, and applicant metadata.
2. **Applicant Resolution and Profiling:** Applicants are resolved against the local OpenAlex database to accurately identify their academic footprint and historical research domains.
3. **LLM-Driven Concept Extraction:** The proposal text is fed into the locally hosted PeerAtlas LLM, which autonomously identifies the primary scientific discipline, core concepts, and methodological approaches. Concepts are rigorously validated against the proposal's actual text to prevent LLM hallucinations.
4. **Hybrid Literature Retrieval:** The engine generates diverse search queries to retrieve a broad pool of relevant academic works, utilizing Reciprocal Rank Fusion (RRF) to combine traditional keyword searches with dense semantic vector searches.
5. **Candidate Harvesting and Scoring:** Authors of the top-ranked works are extracted to form a candidate pool and scored using a weighted algorithm that evaluates semantic text similarity, recent publication productivity, and academic graph centrality. Strict domain coherence gates filter out incompatible disciplines.
6. **Strict COI Auditing:** The surviving shortlist undergoes a rigorous COI audit evaluating direct co-authorship history, institutional overlaps, competitive conflicts, and shared funding awards.
7. **Final LLM Adjudication and Diversity Enforcement:** The conflict-free shortlist is sent back to the PeerAtlas LLM, which acts as a final panel judge to assign a precision fit-score before enforcing diversity constraints to prevent institutional domination, ultimately yielding the top 60 expert reviewers.

---

## Phase II: PeerAtlas Review Report Evaluation Workflow

![Review Report Auditing Flowchart](docs/images/report-evaluation-flowchart.png)
*(Note: Add your Phase 2 flowchart image here)*

While Phase I solves the bottleneck of finding the right experts, Phase II solves the challenge of auditing their work at scale. As reviews arrive in high-volume batches, this module acts as an automated meta-reviewer to ensure quality control, fairness, and compliance.

* **Dedicated Assessment Model:** Instead of using a generalized LLM, Phase II utilizes a second, distinctly fine-tuned Mistral-NeMo 12B module from the PeerAtlas family. This model is heavily specialized for critique-text assessment rather than concept extraction.
* **Contextual Inputs and Structured Outputs:** The model ingests the raw text of the submitted peer-review reports, optionally paired with the original proposal abstract for strict alignment checks. It generates structured quality assessments designed to plug directly into funding organizations' meta-review systems for audit trails and monitoring.
* **The Tripartite Evaluation Framework:** Every review is instantly evaluated across three primary dimensions:
    1. **Technical Substance and Rigor:** The system checks if the reviewer actively engaged with the core scientific concepts of the proposal, assessing whether the feedback is highly relevant and technically rigorous, or if it relies on generic, shallow observations.
    2. **Integrity and Authenticity:** As a crucial security layer, the module scans the text for latent bias signals (e.g., unfair tone or demographic assumptions). It also employs forensic detection to flag synthetic, AI-generated fabrications, ensuring human authorship.
    3. **Linguistic Polish:** The system performs a surface-level audit, catching basic grammatical errors, typos, and formatting issues to ensure the feedback is professional.

> **Try it out:** For testing and evaluation purposes, a publicly available, smaller version of the Review Report Evaluation (RRE) model can be found at: [emreozelemre/RespubLitRRE](https://github.com/emreozelemre/RespubLitRRE)

---

## Throughput and Scalability

The system is engineered for heavy-duty campaign usage and is capable of processing thousands of proposals in a single continuous run. Throughput is primarily constrained by compute availability, memory, and external API rate limits, which may require higher-tier API subscriptions for massive-scale concurrency.

* **Workstation-Scale Deployments:** Processes at approximately `<1` to `3` minutes per proposal.
* **AI Cluster Deployments:** With parallel execution environments, throughput can reach `~15 seconds` per proposal under favorable conditions.

---

## Scope of Public Documentation

This repository documents the system at the level of workflow structure, model roles, data sources, and operational interfaces. 

> **Proprietary Notice:** To protect core intellectual property, internal calibration logic—including specific scoring formulas, threshold values, LLM prompt engineering, and LoRA tuning parameters—is intentionally not disclosed.
