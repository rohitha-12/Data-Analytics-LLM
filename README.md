# Data-Analytics-LLM
# Nicaragua IMF Financial Analysis — LLM Case Study

**Student reference number:** 24137331  
**Country and report:** Nicaragua, 2025 Article IV Consultation  
**Primary document:** IMF Country Report No. 26/014 (January 2026)  
**Execution environment:** Google Colab (Python)

## 1. Overview

This notebook analyses the IMF's *Nicaragua: 2025 Article IV Consultation—Press Release; and Staff Report*. It combines PDF text extraction, section-level summarisation, structured financial-indicator extraction, retrieval-augmented generation (RAG), automated evaluation, and a four-panel dashboard. An optional experiment compares generic prompting with strict system instructions.

The notebook uses **`gemini-3.5-flash`**, as specified in its configuration, for text generation and LLM-as-a-Judge assessments. It uses a separate, locally run Sentence Transformers model for dense retrieval embeddings.

## 2. Files

| File | Purpose |
|---|---|
| `24137331_.ipynb` | Main Google Colab notebook; contains 13 Python code cells and saved execution outputs. |
| `Nicaragua.pdf` | Source IMF report (105 pages). |
| `README.md` | Setup, execution, output and methodology guide. |

The notebook generates its own processed data, JSON results, vector index and dashboard. These generated files are **not required before the first run**.

## 3. Running the Notebook in Google Colab

1. Open **`24137331_.ipynb`** in Google Colab.
2. Upload **`Nicaragua.pdf`** to the Colab session so its path is exactly **`/content/Nicaragua.pdf`**. The filename is case-sensitive.
3. Create a Gemini API key through Google's supported Gemini API access method. In Colab, open **Secrets** (key icon), create a secret named **`GeminiAPI`**, paste the key, and enable notebook access. Do **not** paste the key into code or commit it to a repository.
4. Run the notebook's code cells **in order, from Cell 1 through Cell 13**. The notebook includes three Markdown headings between code sections; those headings are not counted as code cells.
5. Review saved results in **`/content/imf_outputs/`**. Download any files needed for submission before the Colab runtime is deleted.

Cell 1 installs `google-genai`, `pypdf` and `tiktoken`; Cell 9 installs `sentence-transformers` and `faiss-cpu`. The workflow also uses `pydantic`, `numpy`, `pandas` and `matplotlib`. If Colab reports a dependency conflict after installation, resolve it and restart the runtime before rerunning the cells.

**API usage:** Gemini calls require an accessible model and available quota. The notebook uses sequential requests, caching and retries to reduce repeated requests; quotas or API availability can still interrupt a fresh run.

## 4. Notebook Structure

| Code cell(s) | Task | Main operation |
|---|---|---|
| 1–2 | Task 1 | Configure Gemini securely; extract and clean the 105-page PDF. |
| 3 | Task 1 | Build overlapping token-aware chunks and identify report sections. |
| 4–5 | Task 1 | Produce section/executive summaries and evaluate them using a separate judge prompt. |
| 6 | Task 2 | Count seven financial themes and extract economic strengths and vulnerabilities. |
| 7–8 | Task 2 | Extract seven JSON indicators, handle missing data, extract trends and audit results. |
| 9 | Task 3 | Generate local MiniLM embeddings and create a FAISS search index. |
| 10 | Task 3 | Retrieve evidence, answer financial questions and evaluate grounded responses. |
| 11 | Task 4 | Prepare annual indicators and use `pd.melt()` for tidy data. |
| 12 | Task 4 | Generate a four-panel financial dashboard at 320 DPI. |
| 13 | Extra credit | Compare generic prompting against strict system instructions. |

## 5. Main Methods

- **PDF processing:** `pypdf`, with cleaning and retained physical PDF-page references.
- **Chunking:** `tiktoken`, with a target of **450 tokens** and **50-token overlap**.
- **Summarisation:** Gemini generates short section summaries and seven executive-summary points.
- **Structured extraction:** Pydantic schemas validate the seven requested JSON fields. Unavailable indicators may be represented by `null`; the overall capital adequacy ratio is not treated as the Tier 1 capital ratio.
- **RAG:** `sentence-transformers/all-MiniLM-L6-v2` creates local dense vectors; FAISS `IndexFlatL2` retrieves matching passages. Gemini answers from retrieved passages and records source references.
- **Visualisation:** Pandas (`pd.melt`) and Matplotlib produce theme, economic-trend, evaluation-score and banking-indicator charts.
- **Evaluation:** Separate judge prompts assess summarisation, numerical extraction and RAG responses. These scores are automated assessments, not independently verified ground truth.

## 6. Outputs

All paths below are relative to **`/content/imf_outputs/`**.

| File | Contents |
|---|---|
| `nicaragua_cleaned.txt` | Cleaned PDF text, including page markers. |
| `task1_chunks.json` | Overlapping source chunks and page metadata. |
| `task1_sections.json` | Identified report sections. |
| `task1_section_summaries.json` | Individual section summaries. |
| `task1_summaries.json` | Section and executive summaries. |
| `task1_section_audits.json` | Section-level evaluation results. |
| `task1_results.json` | Consolidated Task 1 outputs. |
| `task2_themes_qualitative.json` | Theme frequencies, strengths and vulnerabilities. |
| `task2_core_indicators.json` | Seven required numerical/identification fields. |
| `task2_indicator_evidence.json` | Evidence, reporting periods and missing-value notes. |
| `task2_temporal_trends.json` | Annual financial trends. |
| `task2_results.json` | Consolidated Task 2 outputs and indicator audits. |
| `nicaragua_faiss.index` | Locally built FAISS vector index. |
| `nicaragua_rag_metadata.json` | Retrieval passages and index-cache metadata. |
| `task3_rag_results.json` | Retrieved evidence, answers and RAG evaluations. |
| `task4_financial_trends.csv` | Wide-format financial indicators. |
| `task4_tidy_indicators.csv` | Long-format indicators created with `pd.melt()`. |
| `task4_evaluation_scores.csv` | Evaluation metrics for the dashboard. |
| `nicaragua_dashboard.png` | Four-panel dashboard exported at **320 DPI**. |
| `task13_prompt_benchmark.json` | Prompt-comparison responses, latencies and scores. |
| `cache/` | Reusable API-response caches. |

## 7. Recorded Notebook Results

The uploaded executed notebook reports:

- **105 PDF pages**, approximately **39,338 extracted words** and **188 source chunks**.
- **32 section summaries** and **7 executive-summary bullet points**.
- **7 financial themes**, with **6 strengths** and **6 vulnerabilities**.
- **421 FAISS-indexed retrieval passages**, each represented by a **384-dimensional embedding**.
- **3 RAG questions** tested; **2 answered** and **1 marked as unsupported by retrieved evidence**.
- A **four-panel dashboard** saved at **320 DPI**.
- **4 prompt-benchmark trials** (two generic and two strict-instruction trials).

These are the notebook's **saved execution results**, not measurements from a new independent run.

## 8. Limitations and Interpretation

1. **Retrieval coverage:** The RAG question about the countercyclical capital buffer was not answered because the relevant passage was not retrieved, even though the recommendation appears in the IMF source. This is a retrieval failure and should not be presented as an absence of evidence in the report.
2. **Automated judge scores:** Several evaluations received 5/5, but such scores do not establish that the complete source was retrieved or that every assertion is independently verified.
3. **Mixed observation periods:** Economic indicators refer to different years or months. The JSON values should be interpreted with their associated reporting periods; unavailable Tier 1 capital data are represented as `null`.
4. **Prompt benchmarking:** The two prompt variants use different temporal guidance, and only two trials were run per variant. The reported accuracy comparison is exploratory rather than conclusive.
5. **Reproducibility:** Model/API behaviour and quota availability may change. The notebook's section boundaries are tailored to the provided Nicaragua PDF rather than arbitrary IMF documents.

## 9. Data Source and Acknowledgement

International Monetary Fund (2026). *Nicaragua: 2025 Article IV Consultation—Press Release; and Staff Report*. IMF Country Report No. 26/014.

The source document and its numerical findings belong to the IMF. This notebook and its generated outputs should be interpreted as an academic coding case study rather than financial advice. Include source attribution, your own critical analysis and any academic-integrity disclosures required by your course.
