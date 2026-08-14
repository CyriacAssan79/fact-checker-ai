# 🔎 Fact-Checker AI

> **Evidence-based automatic fact-checking system built with NLP, Information Retrieval and Natural Language Inference (NLI).**

Fact-Checker AI is a prototype designed to automatically verify factual claims by retrieving relevant evidence from a Wikipedia corpus, reranking candidate passages, and applying Natural Language Inference.

Given a claim, the system produces one of three FEVER-style labels:

- ✅ `SUPPORTS` — the evidence supports the claim
- ❌ `REFUTES` — the evidence contradicts the claim
- ❓ `NOT ENOUGH INFO` — the available evidence is insufficient to decide

The project was developed as a **final project for a GenAI & Machine Learning training program** and focuses on building an end-to-end fact-checking pipeline rather than relying on a single classification model.

---

## 🚀 Pipeline

```text
                    ┌──────────────────┐
                    │      Claim       │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Dense Retrieval  │
                    │      FAISS       │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Passage Retrieval│
                    │     SQLite       │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │    Reranking     │
                    │  Cross-Encoder   │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │      NLI         │
                    │ DeBERTa-based    │
                    └────────┬─────────┘
                             │
                             ▼
                 ┌─────────────────────────┐
                 │  Verdict + Evidence +   │
                 │ Confidence information  │
                 └─────────────────────────┘
```

### 🔄 Processing Flow

The system follows a multi-stage verification pipeline:

`Claim → Dense Retrieval → Passage Retrieval → Reranking → NLI → Final Verdict`

1. **Claim embedding** — The input claim is transformed into a semantic vector representation.
2. **Dense retrieval** — FAISS searches for semantically similar documents and passages in the indexed Wikipedia corpus.
3. **Passage retrieval** — Retrieved document identifiers are mapped to their corresponding passages using SQLite.
4. **Reranking** — A Cross-Encoder evaluates the relevance of candidate passages and reorders them.
5. **Natural Language Inference** — The NLI model compares the claim with the retrieved evidence.
6. **FEVER label conversion** — The NLI output is converted into a FEVER-style verdict.
7. **Evidence presentation** — The application displays the verdict, confidence information and supporting evidence.

## 🧩 Main Components

| Component | Technology / Model | Role |
|---|---|---|
| Claim & passage embeddings | `sentence-transformers/all-MiniLM-L6-v2` | Semantic representation |
| Dense retrieval | FAISS | Fast vector similarity search |
| Metadata store | SQLite | Passage and document metadata |
| Reranking | `cross-encoder/ms-marco-MiniLM-L-6-v2` | Candidate relevance scoring |
| Verification | `cross-encoder/nli-deberta-v3-base` | Natural Language Inference |
| Interface | Streamlit | Interactive fact-checking application |
| Evaluation | FEVER + custom metrics | Retrieval and verification evaluation |

## 🎯 Project Objective

The objective of this project is to explore how a complete:

`Retrieval → Reranking → NLI`

pipeline can be used to automatically verify factual claims using external evidence.

Unlike a simple text classifier, the system attempts to:

- retrieve relevant evidence;
- rank the retrieved passages;
- compare claims against evidence;
- produce a FEVER-style verdict;
- expose the evidence used for the decision;
- provide confidence and intermediate scores.

This approach makes the verification process more evidence-based and interpretable.

## 📊 Evaluation

The system was evaluated using the FEVER development set and the currently indexed Wikipedia subset.

### Latest End-to-End Evaluation

- FEVER development examples loaded: 19,998
- Target evaluation cap: 50 evaluable examples
- Actually evaluated: 24
- Skipped: 19,974

### Results

| Metric | Result |
|---|---|
| Correct labels | 18 / 24 |
| Accuracy | 75.00% |
| Evidence Recall | 54.17% |
| Evidence Precision | 3.34% |
| FEVER Score | 41.67% |
| Macro F1 | 0.5211 |
| Expected Calibration Error (ECE) | 0.1333 |

### ⚠️ Evaluation Scope

These results are based on the 24 FEVER examples that were actually evaluable with the currently indexed corpus.

Most skipped examples require gold evidence that is not present in the current Wikipedia subset.

Therefore, the reported metrics should not be interpreted as a full evaluation of the complete FEVER development set.

This limitation highlights an important aspect of evidence-based fact-checking:

> Retrieval quality and corpus coverage directly influence downstream verification performance.

The project's FEVER Score implementation counts an example as correct when:

- the predicted label is correct; and
- at least one gold evidence sentence is successfully retrieved.

Expected Calibration Error (ECE) is also implemented in `evaluation/metrics.py` to monitor confidence calibration.

## 🖥️ Application

The project includes a Streamlit web interface allowing users to enter a claim and obtain a complete verification result.

The application displays:

- Final verdict
- FEVER label
- Confidence information
- Label scores
- Primary evidence
- Source document
- Retrieved passage
- FAISS similarity score
- Reranking score
- NLI details

### Example

Input:

```text
Andrew Kevin Walker is only Chinese.
```

The system processes the claim through the complete retrieval and verification pipeline before displaying the final result.

### Launch the application

```powershell
streamlit run frontend/app.py
```

💡 Screenshots of the Streamlit interface can be added here to provide a visual demonstration of the project.

## 📁 Project Structure

```text
fact-checker-ai/
│
├── api/
│   └── pipeline.py              # Pipeline orchestration
│
├── config/
│   └── setting.py               # Paths, models and configuration
│
├── data/
│   ├── fever/                   # FEVER datasets
│   ├── wikipedia/               # Wikipedia corpus
│   ├── processed/               # Processed chunks
│   └── indexes/                 # FAISS and SQLite indexes
│
├── evaluation/
│   ├── evaluate_pipeline.py     # End-to-end evaluation
│   ├── evaluate_retrieval.py    # Retrieval evaluation
│   ├── metrics.py               # Evaluation metrics
│   └── evaluation.ipynb         # Evaluation notebook
│
├── frontend/
│   ├── app.py                   # Streamlit application
│   └── .streamlit/              # Streamlit configuration
│
├── indexing/
│   ├── preprocess_wikipedia.py  # Wikipedia preprocessing
│   ├── build_embeddings.py      # Embedding generation
│   ├── build_faiss.py           # FAISS index construction
│   └── build_chunk_database.py  # SQLite metadata database
│
├── retrieval/
│   ├── document_retriever.py    # Document retrieval
│   ├── passage_retriever.py     # Passage retrieval
│   └── reranker.py              # Cross-Encoder reranking
│
├── verification/
│   └── verifier.py              # NLI verification
│
├── tests/                       # Unit tests
│
├── utils/                       # Utility modules
│
├── requirements.txt
├── README.md
└── Rapport_Projet_Fact-Checker_AI.pdf
```

## ⚙️ Installation

The project was developed and tested in a dedicated Conda environment named:

```text
fact-checker
```

The environment was tested under WSL.

### 1. Activate the environment

```powershell
conda activate fact-checker
```

### 2. Install dependencies

```powershell
pip install -r requirements.txt
```

The project uses several Hugging Face models. The first execution may therefore download model weights automatically.

An `HF_TOKEN` is optional for the current pipeline, although authentication can improve Hugging Face Hub rate limits and download speed.

## 🏗️ Build the Index

Run the following commands from the repository root.

### 1. Preprocess Wikipedia

```powershell
python indexing/preprocess_wikipedia.py
```

This step prepares the Wikipedia corpus and generates sentence-based chunks.

### 2. Generate embeddings

```powershell
python indexing/build_embeddings.py
```

Embeddings are generated using:

```text
sentence-transformers/all-MiniLM-L6-v2
```

### 3. Build the FAISS index

```powershell
python indexing/build_faiss.py
```

This creates the vector index used for dense retrieval.

### 4. Build the SQLite metadata database

```powershell
python indexing/build_chunk_database.py
```

SQLite stores the metadata required to map retrieved vectors back to their corresponding passages and source documents.

## 🧪 Run the Evaluation

To evaluate the complete pipeline:

```powershell
python evaluation/evaluate_pipeline.py
```

For retrieval-specific evaluation:

```powershell
python evaluation/evaluate_retrieval.py
```

## 📦 Current Artifacts

The latest local build contains:

- 109 Wikipedia JSONL files
- 747,028 processed chunks
- 96,406 distinct indexed documents in SQLite
- 1,460 NumPy embedding batches
- Embedding dimension: 384
- Embedding model: `sentence-transformers/all-MiniLM-L6-v2`
- FAISS index: `data/indexes/faiss.index`
- SQLite database: `data/indexes/chunks.db`

Large generated artifacts such as raw Wikipedia data, processed chunks, embeddings, FAISS indexes and SQLite indexes are intentionally excluded from version control.

## ⚠️ Limitations

This project is an experimental prototype and currently has several limitations:

- The indexed Wikipedia corpus is a configured subset/snapshot, not a guaranteed complete FEVER Wikipedia dump.
- `indexing/preprocess_wikipedia.py` currently uses `MAX_FILES = 2` for the current preprocessing configuration.
- The latest evaluation contains only 24 evaluable examples because many FEVER gold evidence items are outside the indexed corpus.
- Evidence retrieval quality depends strongly on corpus coverage, chunking strategy and embedding quality.
- NLI confidence is not fully calibrated.
- Running the complete indexing pipeline can require significant CPU, RAM and disk resources.
- Reproducing the exact evaluation requires the same corpus configuration and model versions.
- The current system is a prototype and should not be considered a replacement for professional human fact-checking.

## 🔭 Future Improvements

Potential improvements include:

- [ ] Build and evaluate against a larger, explicitly documented Wikipedia/FEVER corpus
- [ ] Improve hybrid sparse + dense retrieval with BM25 and semantic search
- [ ] Tune chunk size and overlap to improve evidence recall
- [ ] Add stronger claim-aware reranking
- [ ] Improve evidence aggregation and sentence selection
- [ ] Calibrate NLI confidence
- [ ] Expand automated regression tests
- [ ] Add a reproducible benchmark configuration
- [ ] Improve the Streamlit interface
- [ ] Add source links and richer evidence visualization

## 🧠 Skills & Technologies

This project allowed me to work on several areas of modern AI and NLP:

- Artificial Intelligence & Machine Learning
- Natural Language Processing (NLP)
- Natural Language Inference (NLI)
- Machine Learning
- Information Retrieval
- Semantic Search
- Vector Search
- Retrieval-Augmented Generation concepts
- Model evaluation
- Confidence calibration

### Technologies

- Python
- PyTorch
- Hugging Face Transformers
- Sentence Transformers
- FAISS
- SQLite
- Streamlit
- NumPy
- FEVER Dataset

## 💡 Key Learning

One of the main lessons from this project was that:

> Building an effective AI verification system is not only about choosing a powerful model. The quality, coverage and retrieval of the underlying evidence corpus are equally important.
>
> A strong NLI model cannot compensate for evidence that was never retrieved.

This project therefore gave me practical experience with the complete AI pipeline:

```text
Data
 ↓
Preprocessing
 ↓
Embeddings
 ↓
Retrieval
 ↓
Reranking
 ↓
NLI
 ↓
Evaluation
 ↓
Analysis of limitations
```

## 📄 Project Report

A detailed project report is available in the repository:

📘 [Rapport_Projet_Fact-Checker_AI.pdf](Rapport_Projet_Fact-Checker_AI.pdf)

The report provides additional information about:

- Project objectives
- Methodology
- Architecture
- Models used
- Dataset
- Implementation
- Evaluation
- Results
- Limitations
- Future improvements

## 👤 Author

**Cyriac ASSAN**

Master 2 — Big Data Analytics
AI & Machine Learning

Interested in:

- Artificial Intelligence
- Machine Learning
- Data Science
- Natural Language Processing
- Generative AI

## 📜 License

This project is intended primarily for educational, research and demonstration purposes.
