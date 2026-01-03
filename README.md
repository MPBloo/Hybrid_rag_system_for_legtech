Designing a Hybrid Symbolic–Semantic RAG System for Legal Decision Support

Exploratory system design under real-world constraints

1. Motivation

Retrieval-Augmented Generation (RAG) systems have shown strong performance on unstructured textual corpora. However, legal documents introduce specific challenges that standard chunk-based RAG pipelines struggle to address:
High lexical redundancy: many articles share similar phrasing with subtle normative differences.
Strong structural constraints: applicability depends on zones, destinations, types of work, and cross-referenced rules.
Low tolerance for irrelevant context: legal professionals require precision, traceability, and explicit citations.
This project explores a hybrid symbolic–semantic retrieval architecture designed to operate under these constraints, with a focus on decision support rather than automation.
The system was developed iteratively in the context of a real-world legal compliance product, which imposed strict requirements on interpretability, controllability, and failure modes.

2. Baseline: Naïve Hybrid RAG and Its Limits
2.1 Initial Approach

The initial system followed a conventional hybrid RAG design:
OCR and text normalization of legal documents / Fixed-size chunking with overlap /Semantic vector search (embeddings) / Keyword-based search with heuristic weighting / Top-k chunk injection into a large language model
This baseline was intentionally simple and served as a reference point for identifying failure modes.

2.2 Observed Failure Modes

In practice, several structural issues emerged:
Semantic redundancy amplification : 
Similar articles located in different sections were retrieved simultaneously, overwhelming the LLM with near-duplicate or irrelevant information (even if the phrasing is alike, some rules apply in certain sections and not other, and those rules were sometimes wrongfully retrieved because the location of the rules was not taken into account).

Loss of normative context : 
Chunking ignored legal structure (zones, applicability conditions), leading to answers that were linguistically relevant but legally invalid.

Context dilution : 
Increasing chunk size to preserve context reduced precision; decreasing it increased hallucinations.

Implicit reasoning leakage : 
The LLM inferred applicability implicitly instead of being constrained explicitly, which is undesirable in legal settings.
These limitations motivated a shift away from purely text-centric retrieval.

3. Design Hypothesis

Legal retrieval requires explicit structural constraints before semantic ranking.
Rather than asking the LLM to infer legal structure from text, the system should: encode explicit applicability rules symbolically, and use semantic similarity within a constrained search space.

This led to a two-layer architecture:
Symbolic filtering via a lightweight knowledge graph
Semantic prioritization via vector search

4. System Overview
4.1 High-Level Architecture

The system separates concerns into three layers:
Structural Layer (Symbolic)
Encodes legal applicability and relationships.

Semantic Layer (Neural)
Handles linguistic variability and relevance ranking.

Generation Layer (LLM)
Produces natural-language explanations grounded in retrieved sources.

This separation is intentional and reflects a design choice to avoid end-to-end black-box behavior.

5. Knowledge Graph Design
5.1 Ontology (Minimal by Design)
The knowledge graph models only decision-relevant entities:
Rule (atomic legal articles)
Zone
Thematique
Destination
Travaux
Tag

Relations encode applicability and scope, such as:
applies to
deals with
is triggered by

The ontology is deliberately shallow. The goal is constraint, not full legal formalization.

5.2 Rationale
Avoid overfitting to a specific legal theory
Keep the graph interpretable by non-technical stakeholders
Allow incremental enrichment without schema explosion
The graph is not used for inference but for search-space restriction.

6. Semantic Indexing of Legal Citations

Each legal rule is indexed semantically with enriched context:
exact citation
normalized textual content
structural tags derived from the graph
human-readable references

Vector search is performed after symbolic filtering, not before.

This ensures:
high recall within a legally valid subset
reduced semantic noise
traceable retrieval paths

7. Question Analysis and Orchestration
7.1 Natural Language Question Parsing

User questions are analyzed to extract:
applicable zone (if explicit)
intended type of work
thematic intent
keywords

This component relies on an LLM and is treated as heuristic and fallible by design.
Error propagation is explicitly acknowledged and mitigated through conservative filtering rules.

7.2 Retrieval Strategy

Hard constraints: zones (strict filtering)
Soft constraints: destinations, works (scoring boosts)
Semantic ranking within constrained candidates
This distinction between hard and soft constraints proved critical in practice.

8. Answer Generation Philosophy

The system does not aim to:
replace legal reasoning
guarantee completeness
provide definitive legal advice

Instead, it generates:
structured answers
explicit citations
confidence indicators as heuristics, not probabilities
The LLM is framed as an explanation layer, not a decision-maker.

9. Known Limitations

This system makes several non-trivial trade-offs:
Heuristic scoring (non-learned)
No formal evaluation benchmark yet
Dependence on LLM-based question parsing
Limited handling of cross-article logical dependencies
Confidence score is a proxy, not a calibrated uncertainty measure
These limitations are accepted consciously to preserve interpretability and control.

10. Research Directions

This work opens several research avenues:
Learning-to-rank over constrained legal corpora
Probabilistic traversal of legal graphs
Uncertainty-aware generation
Human-in-the-loop validation interfaces
Comparative evaluation against end-to-end RAG baselines
The system is best viewed as an experimental design space, not a finalized solution.

11. Conclusion

This project explores a pragmatic hybrid between symbolic structure and neural semantics in a legally constrained domain.
Its primary contribution is not algorithmic novelty, but architectural clarity:
explicit constraints before similarity
structure before generation
explanation before automation

It reflects an attempt to align modern LLM-based systems with domains where correctness, traceability, and control matter more than fluency.

Author Note
This system was developed in the context of a real-world startup project.
Code and data are not publicly released, but the design and reasoning are shared to foster discussion and critique.
