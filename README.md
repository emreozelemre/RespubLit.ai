# RespubLit.ai

RespubLit.ai is an AI-powered computational platform for research evaluation and peer review. It provides two main workflows:

- Reviewer discovery: proposal → ranked reviewer shortlist

- Review report evaluation: review text → structured quality assessment

The platform is designed as a decision-support system and consistently follows a human-in-the-loop principle.

## Architecture

RespubLit.ai uses a modular architecture with separate components for proposal understanding, retrieval/ranking, and review-text assessment. Inference is implemented on the llama.cpp stack using GGUF models, to support flexible deployment across heterogeneous hardware and stable high-throughput operation.

## Models

### Tallulah (Mistral NeMo 12B, LoRA fine-tuned)
Tallulah is the central interpretive model used in the reviewer discovery workflow for concept extraction, reranking, and final reviewer fit analysis.

### Semantic embedding layer (configurable 3B/4B/7B embedding models)
Used for semantic embeddings, similarity-based retrieval and early-stage semantic ranking. The embedding model can vary across peer-review campaigns depending on domain coverage needs, throughput constraints, and operational context.

### Tallulah review report evaluation layer (Mistral NeMo 12B, LoRA fine-tuned)
A separate fine-tuned module used to evaluate the quality of peer-review reports (not the proposal). It assesses technical rigor, constructive feedback, overall quality, relevance to abstract, bias signals (e.g., sentiment bias, score–text misalignment), grammar and typos, signals consistent with LLM-generated or fabricated reports.

A lightweight, public demonstration model of this function is available for testing here:
https://huggingface.co/emreozelemre/RespubLitRRE-LoRA

# Reviewer discovery workflow

### Concept extraction (Tallulah)
Proposal text is parsed and converted into a concept-level representation used to guide downstream retrieval and ranking.

### Semantic retrieval (embedding layer)
The system retrieves relevant scholarly evidence using embeddings and similarity-based search.

### Candidate construction and enrichment
Candidate reviewers are derived from retrieved evidence and enriched with bibliographic/affiliation metadata.

### Shortlist filtering and COI screening
The candidate pool is reduced via structured ranking and operational filters, including conflict-of-interest screening.

### Final reranking (Tallulah)
Tallulah performs final fit assessment and reranks shortlisted candidates.

### Output generation
Results are exported in structured formats suitable for campaign operations and audit.

## Tallulah Review report evaluation workflow

A second fine-tuned Mistral NeMo 12B module is used specifically for review-text assessment. Inputs are peer-review reports (optionally paired with the proposal abstract for alignment checks). Outputs are structured quality assessments designed for meta-review, quality control, audit, and bias monitoring. The object of analysis is the review text. The review report evaluation module is purpose-built for heavy-duty tasks, capable of auditing thousands of submitted review reports at scale.

## Data sources

RespubLit.ai relies on open scholarly databases: Local OpenAlex dataset (for high-throughput local retrieval) and external APIs for enrichment and COI detection: OpenAlex, Crossref, PubMed, arXiv and Semantic Scholar.

Multi-source enrichment is important for COI detection because reliable screening typically requires cross-checking identifiers, affiliations, publication metadata, and co-authorship traces across multiple sources. For large-scale campaigns, API usage may require subscription or higher-tier access depending on rate limits and concurrency.

## Throughput and scale

The system is designed for heavy-duty campaign usage and can process thousands of proposals in a single run.

Typical per-proposal processing time depends on hardware, proposal length, and enrichment settings:

workstation-scale: ~<1 minute to ~3 minutes per proposal

AI cluster deployments (parallel execution): can reach ~15 seconds per proposal under favorable conditions

At campaign scale, throughput is primarily constrained by compute, memory, and external API capacity.

## TScope of public documentation

This repository documents the system at the level of workflow structure, model roles, data sources, and operational interfaces. Internal calibration logic (scoring formulas, thresholds, prompting, and tuning) is intentionally not disclosed.
