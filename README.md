# Pump Doc AI

**AI-based extraction of technical parameters from equipment documents, and a maintenance assistant for field technicians.**

A self-initiated proof of concept, built with a **local open-source LLM**, so no documents leave the machine.

> ⚠️ All manufacturers, models, datasheets and manuals in this project are **fictional** and were generated for practice. No real company or manufacturer data is used.

![Rule-based vs AI extraction on one datasheet](docs/showcase.png)

---

## The problem

Water and infrastructure operators manage thousands of technical documents from many manufacturers. Each one uses its own layout, language and units. Today, technical parameters are often typed into the asset database by hand, and technicians search through long manuals to find maintenance procedures.

This project explores two AI solutions:

1. **Document extraction:** read equipment datasheets and extract technical parameters into a structured, database-ready format, normalized to standard units.
2. **Maintenance assistant (RAG):** answer technicians' questions from the equipment manuals with step-by-step instructions and a source reference for every step.

## Results: document extraction

Evaluated on 12 datasheets from 3 manufacturers with different layouts, 19 parameters each (228 values), against a hand-checked answer key.

| Method | Accuracy |
|---|---|
| Rule-based baseline (regular expressions, 5 fields) | **80.0 %** |
| Local LLM (same 5 fields) | **100 %** |
| Local LLM (all 19 fields) | **100 %** (228 / 228) |

Speed: about **9 seconds per document** on a single T4 GPU.

**How the LLM reached 100 %**

| Iteration | Change | Result |
|---|---|---|
| 1 | Pydantic schema with optional fields | Model skipped most fields (5/19 on one document) |
| 2 | All fields required + field list in the prompt | 98.2 % |
| 3 | Clearer description for *seal type* + "value only" rule | **100 %** |

A bug in the evaluation itself was also found and fixed (numpy integers were compared as text).

**Why the rule-based baseline fails**

- Different units: **l/s** instead of m³/h and **min⁻¹** instead of rpm → 0 % for flow and speed on one manufacturer
- False positives: in model names like *HV-K220-**4A***, "4A" is read as a current of 4 A
- Every new manufacturer would need new rules, so it does not scale

## Maintenance assistant (RAG): early version

A technician asks a question in plain language, e.g. *"Seal leakage alarm on my pump. What should I do?"*

1. The manuals are split into sections and embedded locally (`nomic-embed-text`).
2. The search is limited to the manual of the pump in question.
3. Sections referenced inside the retrieved text (e.g. *"change oil (4.2)"*) are added automatically.
4. The local LLM writes numbered steps with exact values and a citation like `[MN01 §4.2]` for every step.

An evaluation set of 36 questions with known answers and sections is included. Evaluating and improving this part is the next step.

## Dataset

| Folder | Content |
|---|---|
| `data/raw/datasheets/` | 12 one-page datasheets: clean table, bilingual German/English grid with hp and l/s, and specs hidden in running text |
| `data/raw/manuals/` | 4 installation, operation and maintenance manuals |
| `data/raw/scans/` | 4 datasheets as noisy, rotated scans (no text layer) |
| `data/ground_truth/` | Correct values for all datasheets + 36 question/answer pairs for the manuals |

## Tech stack

| Tool | Purpose |
|---|---|
| **PyMuPDF** | Read text and render pages from PDFs |
| **Ollama + Qwen2.5-7B** | Local open-source LLM for extraction and answers |
| **Pydantic** | Define the output schema and validate the LLM's answers |
| **nomic-embed-text** | Local embeddings for searching the manuals |
| **pandas / NumPy** | Evaluation and similarity search |

## How to run

1. Open `notebooks/pump_doc_ai.ipynb` in Google Colab.
2. Select **Runtime → Change runtime type → T4 GPU**.
3. Copy the `data/` folder to `MyDrive/pump-doc-ai/` in your Google Drive.
4. Run the cells from top to bottom.

## Limitations & next steps

- **Scanned documents:** the 4 scans have no text layer and are not processed yet. Next step: OpenCV preprocessing (deskew, denoise, thresholding) + OCR, or a vision-language model, evaluated against the same answer key.
- **RAG evaluation:** run the 36-question evaluation and improve retrieval and answer completeness.
- **Scale:** tested on a small synthetic dataset; real documents will have more layouts and noise.
- **Phase 2:** move the code into a Python package with a FastAPI backend and a Streamlit demo.

## Author

**Mohammed Almallahi**
