# 5. Making RAG Actually Good

**Week 3 - Generative AI and retrieval · Thu · Hard · about 4 hours**

## What you will practise

- Measuring a RAG system instead of eyeballing it
- Chunk size, hybrid search and re-ranking
- Choosing between fine-tuning, RAG and prompting

## Before you start

Finish 04. You need a working RAG pipeline.

## The task

Your pipeline works on the questions you tried. Now find out how good it really is.

1. Write **fifteen test questions** about your documents, with the answer you expect and which document holds it. Include:
   - ten answerable from a single chunk
   - three needing facts from two different documents
   - two your documents do **not** cover, where the right answer is a refusal
2. Run all fifteen and score two things separately:
   - **retrieval**: did the right chunk come back at all?
   - **answer**: was the final answer correct?
   These fail differently and need different fixes. A correct answer from the wrong chunk is luck, not a working system.
3. Change **chunk size**: try 200, 500 and 1000 characters and re-score all fifteen. Small chunks retrieve precisely but lose context; large ones keep context but drag in noise. Record what happens to your numbers.
4. Add **hybrid search**: combine keyword search (BM25) with vector search. Vector search is weak on exact terms - product codes, names, numbers - and keyword search covers exactly that gap.
5. Add a **re-ranker**: retrieve twenty chunks, then reorder them with a cross-encoder and keep the best five.
6. Produce a table: configuration, retrieval score, answer score, latency. Pick a winner and justify it, latency included - a configuration two percent better but four times slower is often the wrong choice.
7. Write a short section: for your domain, would you choose **prompting, RAG, or fine-tuning**? Give the reasoning. RAG suits facts that change and must be cited; fine-tuning suits a fixed style or format; prompting suits everything simple. Most teams reach for fine-tuning far too early.

The two-question split in step 2 is the professional habit here.

## Example

Scoring retrieval and answer separately:

```markdown
| Config | Retrieval @5 | Answer correct | Latency |
| --- | --- | --- | --- |
| 200-char chunks, vector only | 9/15 | 7/15 | 0.4 s |
| 500-char chunks, vector only | 12/15 | 10/15 | 0.4 s |
| 1000-char chunks, vector only | 11/15 | 9/15 | 0.6 s |
| 500-char + hybrid (BM25) | 14/15 | 12/15 | 0.6 s |
| 500-char + hybrid + re-rank | **15/15** | **13/15** | 1.9 s |

Winner for this project: 500-char + hybrid, without the re-ranker. It finds
14 of 15 at 0.6 s; the re-ranker buys one more retrieval and one more answer
for 3x the latency. In an interactive assistant that trade is not worth it.

The two answers still wrong are both multi-document questions - the chunks are
retrieved correctly and the model fails to combine them. That is a generation
problem, not a retrieval one.
```

Note how the last paragraph uses the separated scores to locate the real fault.

## Done when

Tick every line yourself before you submit. If one of them fails, the assignment is not finished.

- [ ] Fifteen test questions exist, including three multi-document and two unanswerable
- [ ] Retrieval and answer quality are scored **separately**
- [ ] Three chunk sizes are compared across all fifteen questions
- [ ] Hybrid keyword-plus-vector search is implemented and scored
- [ ] A re-ranker is implemented and scored
- [ ] A comparison table includes latency, and a winner is justified
- [ ] A written recommendation covers prompting against RAG against fine-tuning for your domain

## How we mark it

| What we look at | Exemplary | Adequate | Needs work |
| --- | --- | --- | --- |
| **Test set** | Fifteen questions covering all three kinds | Ten or more | Few, or all easy |
| **Separated scoring** | Retrieval and answer measured apart, faults located | Both measured | One combined score |
| **Improvements** | Chunk sizes, hybrid and re-ranking all tried | Two tried | One, or none |
| **Recommendation** | Clear reasoning including latency and cost | Some reasoning | Picks the top number |

## How to submit

Push the notebook or script to your own repository, open a pull request, then paste the pull request link into the matching lesson in Dnyx Learn. The full steps are in [How to submit](../../README.md#how-to-submit).

## Go deeper (optional, not marked)

- [Ragas - RAG evaluation framework](https://docs.ragas.io/)
- [Cross-encoder re-ranking (sentence-transformers)](https://sbert.net/examples/applications/retrieve_rerank/README.html)
- [BM25 explained](https://en.wikipedia.org/wiki/Okapi_BM25)
- [RAG against fine-tuning (OpenAI cookbook)](https://cookbook.openai.com/examples/fine-tuned_qa/ft_retrieval_augmented_generation_qdrant)

------------------------------------------------------------------------------

# Week 3 – Assignment 05: RAG Evaluation

## Overview

This assignment evaluates a Retrieval-Augmented Generation (RAG) system using an agriculture-focused knowledge base containing 10 PDF documents.

The evaluation compares different chunk sizes, dense vector retrieval, hybrid retrieval using BM25 and vector search, and cross-encoder reranking.

## Dataset and Configuration

- Source documents: 10 agricultural PDF documents
- Total baseline chunks: 1,478
- Embedding model: `all-MiniLM-L6-v2`
- Embedding dimensions: 384
- Vector database: ChromaDB
- Answer generation: Gemini 3.5 Flash Lite
- Baseline chunk size: 500 characters
- Chunk overlap: 50 characters
- Evaluation questions: 15 (13 answerable and 2 unsupported)

## Evaluation Results

### 1. Chunk Size Comparison

| Chunk Size | Total Chunks | Source Recall |
|---|---:|---:|
| 200 | 4,420 | 88.46% |
| 500 | 1,478 | 88.46% |
| 1,000 | 702 | 88.46% |

All three configurations achieved the same source-level recall. The 500-character configuration was retained as a balance between context size and chunk count.

### 2. Retrieval Strategy Comparison

| Retrieval Method | Source Recall | Average Retrieval Latency |
|---|---:|---:|
| Vector-only | 88.46% | Not recorded |
| Hybrid BM25 + Vector | 92.31% | 0.055 seconds |
| Vector + Cross-encoder | 92.31% | 2.321 seconds |

Hybrid retrieval combines keyword-based BM25 search and semantic vector search using Reciprocal Rank Fusion (RRF).

The cross-encoder retrieves 20 vector candidates, reranks them, and returns the top 5.

### 3. Answer Quality

Manual evaluation against 15 reference answers:

| Configuration | Answer Quality |
|---|---:|
| Vector-only baseline | 63.33% |
| Hybrid retrieval | 73.33% |

Scoring: 1 for a fully correct answer, 0.5 for a partially correct answer, and 0 for an incorrect answer or an unnecessary refusal.

Cross-encoder answer quality was not evaluated.

### 4. Key Findings

- Hybrid retrieval improved source-level recall from 88.46% to 92.31%.
- Hybrid answer quality improved from 63.33% to 73.33%.
- Cross-encoder reranking achieved the same recall as hybrid search but had higher latency.
- Multi-document questions remained challenging.
- Unsupported questions were correctly refused.

## Selected Configuration

**Hybrid retrieval using BM25 + vector search with RRF** was selected.

It achieved 92.31% source recall and 73.33% answer quality, with an average retrieval latency of 0.055 seconds in the Colab experiment.

## Prompting vs RAG vs Fine-tuning

- **Prompting:** Controls the model's instructions, response format, and behaviour.
- **RAG:** Retrieves relevant information from external agricultural documents and provides it as context. This is suitable for knowledge that needs to be updated without retraining the model.
- **Fine-tuning:** Adapts model behaviour and domain-specific response patterns but requires training data and computational resources.

**Recommendation:** Use RAG with hybrid retrieval and a carefully designed prompt. Consider fine-tuning later if domain-specific response quality remains inadequate.

## Limitations

1. Evaluation was limited to 15 questions and 10 source documents.
2. Multi-document retrieval still needs improvement.
3. Answer quality was evaluated manually.
4. Cross-encoder answer quality was not measured.
5. Latency measurements were collected in Google Colab.
6. The system has not been evaluated in a production environment.

## Tools and Technologies

- Python
- Google Colab
- Sentence Transformers
- ChromaDB
- BM25 (`rank-bm25`)
- Cross-Encoder
- Gemini API

## How AI Was Used

AI assistance was used for code guidance, debugging, evaluation design, and documentation. Retrieval experiments and reported results were generated and evaluated using the notebook implementation.
