# CoalIntel AI

**AI-Powered Mining Reporting & Document Intelligence for CMPDI/CIL subsidiaries**

Smart India Hackathon 2026 · Problem Statement **SIH26023** · Theme: Smart Automation · Category: Software
Team **AK LABS** (Team ID 49976)

Demo video: https://youtu.be/R2fXiT5TBXs

---

## The problem

Geological, mining and production reports in Coal India Limited (CIL) and its subsidiaries are still compiled largely by hand from scanned PDFs, spreadsheets and archives. This makes reports slow, dependent on individual experience, prone to typing errors, and hard to trace back to a source page.

## Our solution

CoalIntel AI is a single Gradio web app with the three modules the problem statement asks for. All modules share one document store. The prototype runs on free Google Colab (T4 GPU) with open-source models, so it needs no paid API.

### Module 1: Automated report generation
- Reads PDF, Excel and CSV files (pdfplumber, PyMuPDF, pandas).
- Pages without a text layer go through Tesseract OCR.
- Stores the **source file and page (or sheet) for every extracted value**.
- Runs five validation checks: missing values, negative values, out-of-range values, duplicate rows, and a consistency check that the **CIL total equals the sum of its eight subsidiaries**.
- Generates a DOCX report (python-docx) with the data table, source page and validation status for each value.

### Module 2: Word cloud and topic identification
- TF-IDF extracts keywords (top 80 feed the word cloud).
- LDA with 5 topics groups the same text chunks into themes.
- On our test documents the themes were mine safety and rescue, exploration and drilling, coking coal and captive mining, employee welfare, and production and projects.

### Module 3: AI question answering with citations
- Documents are split into 800-character chunks and embedded with MiniLM.
- A FAISS index retrieves the top 6 chunks per question.
- Qwen2.5-1.5B answers only from the retrieved text and cites the source file and page.
- If the evidence is missing, it replies **"Not found in the documents."**
- Company-year production questions are answered from the validated table built in Module 1, because the small model sometimes misread flattened table text.

## Pipeline

```
Document upload (PDF / Excel / CSV / scans)
        |
Ingestion and parsing (text and tables with source file + page)
        |
  +-----------------+------------------+-------------------+
  | Extract+validate| Keywords+topics  | Chunk+retrieve    |
  | Module 1        | Module 2         | Module 3          |
  +-----------------+------------------+-------------------+
        |
Validation and grounding (rule checks, answers limited to retrieved text)
        |
Output: DOCX report, word cloud, cited answer
        |
Gradio UI (3 tabs)
```

## Results so far

Tested on five public chapters (Ch. 2, 8, 9, 12, 14) of the **Ministry of Coal Annual Report 2022-23** (about 250 indexed chunks).

| Test | Result |
|---|---|
| Values extracted from the Ch. 9 company-wise production table | 36 (12 entities x 3 financial years), each with page number |
| CIL total = sum of subsidiaries | 3 of 3 years |
| "How much coal did CIL produce in 2021-22?" | 622.63 million tonnes, cited page; matches table extraction |
| Planted errors in a synthetic spreadsheet (negative, blank, 9999, repeated row) | 4 of 4 flagged |
| Out-of-scope question | Refused (1 of 1, manual test) |

**Still pending:** a 12-question Q&A accuracy test and a timed manual-vs-tool report comparison. We do not claim accuracy or time-saving figures until these are measured.

## Tech stack

Python 3 · Google Colab (T4 GPU) · PyMuPDF · pdfplumber · Tesseract OCR · pandas · openpyxl · sentence-transformers (MiniLM) · FAISS · Qwen2.5-1.5B · scikit-learn · wordcloud · python-docx · Gradio

## Getting started

1. Open the notebook in Google Colab (`sih.ipynb`) and set the runtime to **T4 GPU**.
2. Run the setup cell to install dependencies. If you hit a Pillow error, run
   `!pip -q install --force-reinstall --no-deps "pillow>=11.3"`, restart the session, and continue from the next step.
3. Upload your PDF, Excel or CSV files (or the sample chapters).
4. Run the chunking and indexing cells, then launch the Gradio cell.
5. Open the `gradio.live` link and use the three tabs:
   - **Report Generation:** extract, validate, download DOCX
   - **Word Cloud & Topics:** keywords and themes
   - **Ask a Question:** cited answers

## Repository structure

```
.
├── sih.ipynb            # full prototype notebook (all three modules + Gradio UI)
├── sample_data/         # public annual report chapters and synthetic test files
├── docs/                # SIH presentation (PDF) and screenshots
└── README.md
```
Adjust this to match your actual files.

## Known limitations

- The table extractor uses a template rule for the layout we tested. Each new report format needs its own rule; unreadable rows should be flagged for review.
- OCR was tested only on a clean simulated scan, not on old noisy archives.
- A 1.5B model can be weak on complex questions. A larger model can be swapped in without changing the rest of the system.

## Roadmap

- Requirement analysis with subsidiary reporting teams and a catalogue of their report formats
- Archive digitisation with OCR quality checks and image clean-up
- More table templates and validation rules agreed with domain experts
- Pilot on real CIL historical reports to measure accuracy and time saved
- On-premises deployment, role-based access, audit logs
- Hindi support, trend dashboards, larger language models

## Expected benefits (targets, to be confirmed in a pilot)

- About 60% less report preparation time
- 95% or better extraction accuracy on standard formats
- About 70% of repetitive steps automated

## Data and references

- Data: Ministry of Coal Annual Report 2022-23 (public chapters). Synthetic files are used only for validation testing.
- Reimers and Gurevych (2019), *Sentence-BERT*
- Johnson, Douze and Jegou (2019), *Billion-scale similarity search with GPUs* (FAISS)
- Blei, Ng and Jordan (2003), *Latent Dirichlet Allocation*, JMLR
- Documentation of Qwen2.5, Tesseract OCR, Gradio and scikit-learn

## Team

**AK LABS** · Smart India Hackathon 2026

## License

Add a license (for example MIT) before making the repo public.
