# Fact-Checker

Evidence-based automatic fact-checking prototype built on FEVER-style claims and a
Wikipedia corpus indexed with dense embeddings and FAISS.

The system verifies a claim by retrieving candidate passages, reranking them, and
running Natural Language Inference (NLI) to produce one FEVER label:
`SUPPORTS`, `REFUTES`, or `NOT ENOUGH INFO`.

## Architecture

Pipeline:

```text
Claim
-> Dense retrieval with FAISS
-> Passage retrieval from SQLite chunks
-> Cross-Encoder reranking
-> NLI verification
-> FEVER label conversion
-> Final verdict with evidence and confidence
```

Main components:

- FEVER data: claims, labels and gold evidence in `data/fever/`.
- Wikipedia corpus: JSONL wiki files in `data/wikipedia/`.
- Preprocessing: `indexing/preprocess_wikipedia.py` creates sentence windows.
- Chunks: `data/processed/processed_chunks.jsonl`.
- Embeddings: `indexing/build_embeddings.py` uses
  `sentence-transformers/all-MiniLM-L6-v2`.
- FAISS: `indexing/build_faiss.py` builds `data/indexes/faiss.index`.
- Metadata store: `indexing/build_chunk_database.py` builds
  `data/indexes/chunks.db`.
- Retrieval: `retrieval/document_retriever.py` and
  `retrieval/passage_retriever.py`.
- Reranking: `retrieval/reranker.py` uses
  `cross-encoder/ms-marco-MiniLM-L-6-v2`.
- NLI: `verification/verifier.py` uses
  `cross-encoder/nli-deberta-v3-base`.
- Pipeline orchestration: `api/pipeline.py`.
- Web UI: `frontend/app.py`.

## Project Structure

```text
api/                 Pipeline orchestration layer
config/              Shared paths, model names and defaults
data/                FEVER files, Wikipedia files, processed chunks and indexes
evaluation/          Retrieval and end-to-end evaluation scripts
frontend/            Streamlit application and theme config
indexing/            Preprocessing, embeddings, FAISS and SQLite builders
retrieval/           Dense retrieval, passage formatting and reranking
tests/               Lightweight unittest coverage
utils/               Reserved utility modules
verification/        NLI verification and FEVER label conversion
```

## Installation

The requested runtime environment is named `fact-checker`.

```powershell
conda activate fact-checker
pip install -r requirements.txt
```

The project was validated from a dedicated `fact-checker` Conda environment
under WSL. The first model downloads may show a Hugging Face Hub warning when
`HF_TOKEN` is not configured; the pipeline still runs, but authenticated access
can improve rate limits and download speed.

## Execution

Run all commands from the repository root after activating `fact-checker`.

Preprocess Wikipedia:

```powershell
python indexing/preprocess_wikipedia.py
```

Generate embeddings:

```powershell
python indexing/build_embeddings.py
```

Build FAISS:

```powershell
python indexing/build_faiss.py
```

Build SQLite chunks database:

```powershell
python indexing/build_chunk_database.py
```

Evaluate the pipeline:

```powershell
python evaluation/evaluate_pipeline.py
```

Launch Streamlit:

```powershell
streamlit run frontend/app.py
```

## Usage

Open the Streamlit app, enter a claim such as:

```text
Andrew Kevin Walker is only Chinese.
```

Click `Verify claim`. The app displays the final verdict, confidence, label
scores, primary evidence, source document, passage text, FAISS score, rerank
score and NLI details.

## Evaluation

Latest end-to-end evaluation run:

```powershell
python evaluation/evaluate_pipeline.py
```

Run context:

- FEVER development examples loaded: 19,998.
- Distinct indexed documents found in `data/indexes/chunks.db`: 96,406.
- Target evaluation cap: 50 evaluable examples.
- Actually evaluated examples: 24.
- Skipped examples: 19,974.

Results:

- Correct labels: 18 / 24.
- Accuracy: 75.00%.
- Evidence Recall: 54.17%.
- Evidence Precision: 3.34%.
- FEVER Score: 41.67%.
- Macro F1: 0.5211.
- ECE: 0.1333.

The FEVER Score implemented for this project counts an example only when the
predicted label is correct and at least one gold evidence sentence is retrieved.
`evaluation/metrics.py` also includes Expected Calibration Error (ECE).

## Current Artifacts

Observed local artifacts:

- Wikipedia files: 109 JSONL files.
- Processed chunks: 747,028 chunks.
- Distinct indexed documents in the current chunks database: 96,406.
- SQLite database: `data/indexes/chunks.db`.
- FAISS index: `data/indexes/faiss.index`.
- Embeddings: 1,460 NumPy batches, dimension 384.
- Embedding model metadata:
  `sentence-transformers/all-MiniLM-L6-v2`.

## Limits

- The indexed corpus is a subset/configured snapshot, not a guaranteed full
  FEVER Wikipedia dump.
- `indexing/preprocess_wikipedia.py` currently has `MAX_FILES = 2`; this should
  be changed intentionally before rebuilding from raw Wikipedia if a full corpus
  is required.
- Evaluation metrics are reported on the currently indexed subset only. In the
  latest run, only 24 FEVER development examples were evaluable because most
  gold evidence was outside the indexed corpus.
- Retrieval quality depends on chunking, dense embedding quality and corpus
  coverage.
- NLI confidence is not fully calibrated; ECE is available to track this.
- Heavy model and FAISS execution requires the correct environment with FAISS
  installed.

## Perspectives

- Improve document retrieval with hybrid sparse+dense search.
- Tune chunk size and overlap against evidence recall.
- Add stronger reranking or claim-aware evidence selection.
- Calibrate NLI confidence and tune aggregation.
- Evaluate on a larger, explicitly documented FEVER split.
- Add regression tests that run against a tiny local FAISS fixture.
