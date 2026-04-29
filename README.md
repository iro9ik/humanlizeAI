# HumanizeAI

A text transformation API that converts AI-generated text into natural, human-like writing. Built as a school project, the system exposes a REST API that accepts any AI-generated paragraph and returns a rewritten version with reduced structural overlap and more natural phrasing.

---

## Overview

HumanizeAI is a three-part platform:

- **Frontend** — user interface for submitting and viewing transformed text
- **Backend** — server that routes requests between the frontend and the model API
- **Model API** — this repository, a FastAPI service powered by a fine-tuned T5 model

This repository contains the model API component, built and served from Google Colab using ngrok as a public tunnel.

---

## Model

**Ateeqq/Text-Rewriter-Paraphraser**

- Architecture: T5-base (Text-to-Text Transfer Transformer)
- Parameters: 223 million
- Training data: 430,000 paraphrase examples
- Source: HuggingFace Hub

The model generates four diverse candidate rewrites per sentence using grouped beam search with a diversity penalty, then selects the candidate with the lowest lexical overlap against the original input. This strategy consistently achieves 40-50% lexical diversity across test samples.

---

## Results

| Metric | Value |
|---|---|
| Average lexical overlap | 53% |
| Average lexical diversity | 47% |
| Good rewrites (overlap < 60%) | 6 out of 10 samples |
| BERTScore F1 | 93.5 |
| Perplexity | 3.92 |

---

## API Reference

### Base URL

```
https://your-ngrok-url.ngrok-free.app
```

### Endpoints

**GET /**

Returns server status and model name.

**GET /health**

Returns device info (CPU or CUDA).

**POST /transform**

Transforms AI-generated text into human-like text.

Request body:

```json
{
  "text": "The system autonomously synthesizes context-aware linguistic patterns.",
  "num_beams": 8
}
```

Response:

```json
{
  "original": "The system autonomously synthesizes context-aware linguistic patterns.",
  "transformed": "In order to generate coherent output, the system automatically synthesizes linguistic patterns.",
  "sentences_processed": 1,
  "model": "Ateeqq/Text-Rewriter-Paraphraser"
}
```

**Parameters:**

| Field | Type | Default | Description |
|---|---|---|---|
| text | string | required | Input text, max 5000 characters |
| num_beams | integer | 8 | Beam search width. Higher = better quality, slower |

---

## How to Run

### Requirements

- Google Colab account (free tier works)
- ngrok account (free) — get your token at dashboard.ngrok.com

### Steps

1. Open `HumanizeAI_v7_fixed.ipynb` in Google Colab
2. Set runtime type to GPU: Runtime > Change runtime type > T4 GPU
3. Paste your ngrok token in Cell 5 where it says `YOUR_NGROK_TOKEN_HERE`
4. Run cells 1 through 5 in order
5. Copy the public URL printed at the end of Cell 5
6. Share the URL with the backend team as the hook endpoint

### Startup time

| Step | Time |
|---|---|
| Install dependencies | 1-2 minutes |
| Load model | 1-2 minutes |
| Quality check and graphs | 3-5 minutes |
| API ready | immediate |

Each new Colab session generates a new ngrok URL. Update the backend configuration with the new URL each time.

---

## Repository Structure

```
HumanizeAI/
├── HumanizeAI_v7_fixed.ipynb     # Main notebook — model + API
├── test_api.py                   # Local script to test the running API
└── README.md
```

**Local test script usage:**

```bash
py test_api.py "Your AI-generated text goes here in quotes."
```

---

## Architecture

```
User / Frontend
      |
      | HTTP POST /transform
      v
FastAPI Endpoint (ngrok tunnel)
      |
      v
NLTK Sentence Tokenizer
      |
      | per sentence
      v
T5 Model — generates 4 candidates
      |
      v
Best Candidate Selector (lowest overlap)
      |
      v
Rejoined paragraph returned
```

---

## Evaluation Graphs

Running Cell 3 and Cell 4 in the notebook exports three PNG files:

- `v7_overlap.png` — lexical overlap per sample with color coding
- `v7_summary.png` — pie chart of rewrite quality distribution
- `v7_architecture.png` — pipeline diagram and model statistics

---

## Team

| Role | Responsibility |
|---|---|
| Model & API | Finding, evaluating, and serving the transformation model |
| Frontend | User interface for text input and output display |
| Backend | Server routing between frontend and model API |

---

## Notes

The API is stateless. Each request is independent and the model processes text entirely on the Colab GPU. Response time is approximately 2-4 seconds per sentence depending on the num_beams parameter.

The Colab session remains active as long as the notebook is running. If the session disconnects, rerun cells 2 through 5 to restore the API with a new ngrok URL.
