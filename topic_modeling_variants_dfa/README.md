# Topic Modeling Variants for DFA Scientific Literature

This project applies topic modeling methods to scientific literature related to Dental Fear and Anxiety (DFA) to identify and analyze recurring themes  in the literature.

The following topic-modleing approaches are applied to the DFA scientific literature:

- NMF (TF-IDF) for transparent lexical topics.
- Anchored CorEx for seed-guided thematic steering.
- BERTopic for embedding-based semantic clustering.


## Objective

The objective is to identify recurring themes in DFA scientific literature and compare different topic-modleing approaches in terms of interpretability, domain alignment, and topic diversity.

## Topic Modeling Approaches

NMF (TF-IDF)

Non-negative Matrix Factorization (NMF) is applied to TF–IDF features to identify topics based on patterns of important terms within the literature. It provides a transparent lexical baseline for topic discovery.

Anchored CorEx

Anchored Correlation Explanation (CorEx) uses domain-specific seed terms to guide topic discovery toward concepts relevant to DFA research.

BERTopic

BERTopic uses document embeddings to represent semantic information and groups semantically similar text into topics. This allows realted content to be identified even when different terminology is used.

## Outputs

The pipeline produces:

- Topic word lists for each model in `models/*/*.json`
- Topic cards for interpretation in `reports/topic_cards/<model>/`
- Model evaluation results in `evaluation/*.csv`

---

## Quickstart 

```bash
# Create and activate a virtual env (Linux/macOS)
python -m venv .venv && source .venv/bin/activate

# Install (full stack: pdfminer, CorEx, BERTopic, embeddings, umap, hdbscan)
pip install -U pip
pip install -r requirements-full.txt

# 1) Ingest PDFs → chunks
bash scripts/01_ingest.sh

# 2) Preprocess chunks → cleaned tokens
bash scripts/02_preprocess.sh

# 3) Train NMF, CorEx, BERTopic
bash scripts/03_run_models.sh

# 4) Evaluate (coherence, seed-overlap, diversity + comparison table)
bash scripts/04_evaluate.sh

# 5) Generate topic cards (Markdown)
bash scripts/05_make_report.sh
```

## Repository structure 

```
topic-modeling-dental-fear/
├─ configs/
│  ├─ base.yaml            # paths, chunk sizes, tf-idf, stopwords, random seed
│  ├─ nmf.yaml             # NMF hyperparams (k-range, sparsity, init)
│  ├─ corex.yaml           # CorEx hyperparams (n_topics, anchors, strength)
│  ├─ bertopic.yaml        # Embedding model, UMAP/HDBSCAN, top_n_words
│  └─ seeds.yaml           # Domain seed sets (pain, claustrophobia, etc.)
├─ data/
│  ├─ raw/                 # original PDFs (read-only)
│  ├─ interim/             # chunks.jsonl (post-ingest)
│  └─ processed/           # chunks_tokens.jsonl (preprocessed tokens)
├─ models/
│  ├─ nmf/                 # best_terms.json, grid_results.json
│  ├─ corex/               # topics.json
│  └─ bertopic/            # topics.json, bertopic_model.pkl
├─ evaluation/
│  ├─ extrinsic_overlap.csv    # seed-overlap per model
│  ├─ coherence.csv            # per-topic c_v
│  └─ model_comparison.csv     # aggregate summary
├─ reports/
│  ├─ figures/             # (optional) plots
│  └─ topic_cards/
│     ├─ nmf/              # topic_00.md, topic_01.md, ...
│     ├─ corex/            # topic_00.md, ...
│     └─ bertopic/         # topic_00.md, ...
├─ src/
│  ├─ ingest/
│  │  ├─ pdf_to_text.py    # pdfminer → text per PDF
│  │  └─ chunker.py        # split into ~320-token chunks with overlap
│  ├─ preprocess/
│  │  └─ clean.py          # normalize, lemmatize, stopwords, n-grams
│  ├─ features/
│  │  ├─ tfidf.py          # build TF–IDF + vocab
│  │  └─ seeds.py          # load seed sets for CorEx/extrinsic eval
│  ├─ models/
│  │  ├─ nmf_runner.py     # grid over k, save best terms
│  │  ├─ corex_runner.py   # anchored CorEx with binary features
│  │  └─ bertopic_runner.py# embeddings → UMAP → HDBSCAN → c-TF-IDF labels
│  ├─ eval/
│  │  ├─ extrinsic.py      # seed-overlap
│  │  ├─ coherence.py      # c_v using gensim
│  │  └─ compare.py        # aggregates comparison table
│  ├─ labeling/
│  │  └─ topic_cards.py    # write Markdown cards per topic
│  └─ utils/
│     ├─ io.py             # YAML/JSONL helpers
│     └─ logging.py        # (if present) simple logging config
├─ scripts/
│  ├─ 01_ingest.sh
│  ├─ 02_preprocess.sh
│  ├─ 03_run_models.sh
│  ├─ 04_evaluate.sh
│  └─ 05_make_report.sh
├─ README.md               # (this file)
└─ LICENSE
```

---

## Workflow 

1) **Ingest** 
   - `src/ingest/pdf_to_text.py` extrcats text from the input PDFs.
   - `src/ingest/chunker.py`divides the extracted text into overlapping chunks.
   - Output: `data/interim/chunks.jsonl`

2) **Preprocess**  
   - `src/preprocess/clean.py` normalizes and preprocesses the text, including lemmatization, stopword removal, and n-gram handling.   
   - Output: `data/processed/chunks_tokens.jsonl`

3) **Modeling** 
   - **NMF:** `src/models/nmf_runner.py` applies NMF to TF-IDF features. 
   - **Anchored CorEx:** `src/models/corex_runner.py` applies CorEx using anchors defined in `configs/seeds.yaml`  
   - **BERTopic:** `src/models/bertopic_runner.py` uses embeddings, UMAP, HDBSCAN, and c‑TF‑IDF for topic generation.
   - Outputs are stored under `models/*/*.json`.

4) **Evaluation**
   - `src/eval/extrinsic.py` evaluates seed overlap. 
   - `src/eval/coherence.py` calculates topic coherence.
   - `src/eval/compare.py` generates the model comparison.
   - Outputs are stored under `evaluation/`.

5) **Reporting** 
   - `src/labeling/topic_cards.py` generates topic cards for interpretation.
   - Output: `reports/topic_cards/<model>/`

---

## Configuration and Parameters

Configuration files for preprocessing and each topic-modeling appraoch are stored in `configs/`.

### Preprocessing

`configs/base.yaml` contains settings for:

- text chunking
- TF–IDF features
- n-grams
- stopwords
- random seed

### NMF

`configs/nmf.yaml` contains parameters for NMF, including:

- number of topics
- initialization
- regularization
- maximum iterations

### Anchored CorEx

`configs/corex.yaml` contains parameters for CorEx, including:

- number of topics
- use of anchors
- anchor strength

Domain seed terms are defined in `configs/seeds.yaml`.

### BERTopic

`configs/bertopic.yaml` contains parameters for:

- embedding model
- UMAP
- HDBSCAN
- number of topics
- number of representative terms

---


## Evaluation

The topic-modeling approaches are compared using the following metrics:


- **Coherence (c_v):** measures the semantic consistency of the terms within a topic Results are stored in evaluation/coherence.csv.
- **Seed overlap:** measures alignment between discovered topics and curated DFA domain seed terms. Results are stored in `evaluation/extrinsic_overlap.csv`.
- **Term diversity:** measures the uniqueness of terms across topics and provides an indication of topic redundancy.

Aggregated model-level results are stored in `evaluation/model_comparison.csv`.


## Results 

The experimental run used three DFA-related PDFs, resulting in 20 text chunks after preprocessing.

Three topic-modeling approaches were evaluated:

- **NMF** using TF–IDF features and grid search, with a best topic count of `k = 3`.
- **Anchored CorEx** using binary term features and domain-specific seeds.
- **BERTopic** using `all-MiniLM-L6-v2` embeddings, UMAP, HDBSCAN, and c-TF–IDF.

### Model-level comparison

| Model     | Seed Overlap (↑) | Mean Coherence c_v (↑) | Term Diversity (↑) |
|-----------|-------------------|-------------------------|--------------------|
| BERTopic  | **6**             | **0.6917**              | **0.9333**         |
| CorEx     | 4                 | 0.5621                  | **1.0000**         |
| NMF       | 0                 | 0.3956                  | 0.3333             |

For this experimental run, BERTopic produced the highest mean coherence and seed overlap, while CorEx produced the highest term diversity.

---

### Per-Topic coherence (c_v)

Coherence (c_v) is roughly on **[0, 1]**, where **>0.5 is decent**, **>0.6 good**, **>0.7 strong** (corpus-dependent).

**NMF (k = 3)**
- Topic 0: **0.3956**
- Topic 1: **0.3956**
- Topic 2: **0.3956**

**Anchored CorEx (n_topics per config; seed-guided)**
- Topic 0: **0.5093**
- Topic 1: **0.8044**  ← strongest coherent factor
- Topic 2: **0.4653**
- Topic 3: **0.4461**
- Topic 4: **0.5853**

**BERTopic (HDBSCAN inferred k = 3)**
- Topic 0: **0.7528**
- Topic 1: **0.5644**
- Topic 2: **0.7579**

Full results are available in `evaluation/coherence.csv` and `evaluation/model_comparison`.csv.

---

### Reading topics

Human-readable topic cards generated by each model are available at:

- `reports/topic_cards/nmf/topic_*.md`
- `reports/topic_cards/corex/topic_*.md`
- `reports/topic_cards/bertopic/topic_*.md`

Each topic card contains the representative terms for a topic and can be used to support interpretation and labeling.

---

## Troubleshooting 

`pdfminer` not found

Install the project dependencies:

pip install -r requirements-full.txt

Then rerun:

bash scripts/01_ingest.sh

** CorEx `.A1` attribute error**

Ensure that the expected binary sparse matrix representation is passed to CorEx.

**BERTopic `IsADirectoryError`**

Ensure that the BERTopic model is saved to a file path, such as:

bertopic_model.pkl

rather than to a directory.

**Empty Vocabulary** 

Check the preprocessing configuration and confirm that tokens remain after stopword removal and other preprocessing steps.


## Reproducibility

- Project dependencies are specified in `requirements-full.txt`.
- Random seed settings are defined in `configs/base.yaml`.
- Reproducing the results across environments requires consistent versions of the relevant Python packages, including scikit-learn, UMAP, HDBSCAN, sentence-transformers, and BERTopic.

---

## License and Citation

- See `LICENSE` for licensing information.

- When using this project in research, cite the underlying libraries and the scientific literature used as input to the analysis.

---
