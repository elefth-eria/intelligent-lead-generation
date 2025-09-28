# Intelligent Lead Generation (Notebook)

A Jupyter Notebook project that builds a lightweight, semi‑automatic **lead directory** from the web. It generates targeted queries, runs web search via SerpApi, filters results, and (optionally) scrapes simple contact signals (like emails) from organization pages.

> Built around modular “agents” so you can swap parts in/out without breaking the rest.

---

## Features
- **KeywordAgent**: expands a topic (optionally with synonyms) and adds country + intent clauses (`"contact us"`, `"about us"`, `"team"`, `"official site"`).
- **SerpApiSearchAgent**: queries Google via SerpApi and de‑duplicates by base domain.
- **FilterAgent**: removes noisy domains (e.g., LinkedIn, Crunchbase, Wikipedia).
- **ContactPageFinder**: small, same‑domain BFS to `/contact`, `/about`, `/team` pages; extracts emails (best‑effort).
- **PersonalizedSearchAgent**: excludes your own name/company/LinkedIn handle from queries and lightly re‑ranks results.
- **UI (ipywidgets)**: quick inputs for profile, topic, country, and max results; displays a table and saves `directory_results.csv`.

---

## Requirements
- Python **3.10+**
- Jupyter Notebook or JupyterLab
- Recommended packages:
  - `requests`, `beautifulsoup4`, `pandas`, `tqdm`, `ipywidgets`
  - `transformers` (optional, only if you want LLM‑based synonyms)
  - `pycountry`
- A valid **SerpApi** key (for Google results)

Install example:
```bash
pip install requests beautifulsoup4 pandas tqdm ipywidgets pycountry
# Only if you want Hugging Face synonyms:
pip install transformers
```

Enable ipywidgets (if needed):
```bash
jupyter nbextension enable --py widgetsnbextension
```

---

## Environment / Secrets
Set your SerpApi key before running the notebook:
```bash
export SERPAPI_API_KEY="your_key_here"
```
Or paste it into the notebook init cell (not recommended for versioned code).

If you use Hugging Face for synonyms and the model is gated/private:
```bash
export HUGGINGFACEHUB_API_TOKEN="hf_xxx"
```

> Never commit real API keys to git.

---

## Quick Start
1. Open the notebook (e.g., `Project_24_9_v3_fixed.ipynb` or `Refined_Version.ipynb`).
2. Run the **setup/imports** cell(s).
3. Run the **initialization** cell (loads agents and reads keys).
4. Run the **UI** cell:
   - Fill **First name**, **Surname**, **Company**, **LinkedIn**, **Topic** (e.g., “startup accelerators”).
   - Choose **Country** (e.g., “Greece”) and **Max results**.
   - Click **Run**. A table appears and `directory_results.csv` is saved locally.

---

## Architecture
```
UI (ipywidgets)
   └─ PersonalizedSearchAgent
         ├─ KeywordAgent
         │     └─ SmartSynonymAgent (optional / can be rule-based)
         ├─ SerpApiSearchAgent
         └─ FilterAgent
   └─ ContactPageFinder (optional email BFS)
```
- **KeywordAgent** builds targeted queries using synonyms (or only the topic), country clause, site bias (`.ccTLD` + `.com`), and intent phrases.
- **SerpApiSearchAgent** executes Google searches via SerpApi (`organic_results`). Results are de‑duplicated by base domain.
- **FilterAgent** removes unwanted sites.
- **PersonalizedSearchAgent** appends negative terms from your profile (name, company, LinkedIn handle) and re‑ranks results (penalizes self‑mentions; small boost for ccTLD).
- **ContactPageFinder** fetches the homepage and a few obvious subpaths. It returns the first email found + the page where it was found.

---

## Performance Tips
- **Synonyms**: If `mistral-7b` on CPU is slow, use a tiny model (e.g., `google/flan-t5-small`) or a **rule‑based fallback**; cap tokens (`max_new_tokens≈24`).
- **Reduce fan‑out**: Set `KeywordAgent.max_queries = 6` (instead of 14).
- **Parallel search**: Run SerpApi queries concurrently (ThreadPoolExecutor); set timeouts (≈8s).
- **Reuse connections**: Use a global `requests.Session` with retries and HTTP keep‑alive.
- **Email BFS**: Use short timeouts (≈4s), small `max_pages` (≈3), and only scrape top N results (e.g., 5) in parallel.

---

## Troubleshooting
- **0 results**: check SerpApi key/credits; try a broad topic; reduce negative terms (personalization).
- **Very slow / stalls**: the synonym model is likely the bottleneck—use the rule‑based agent or a tiny HF model with `max_time`.
- **`Setting pad_token_id to eos_token_id`**: harmless info message; you can set `tokenizer.pad_token = tokenizer.eos_token` to silence it.
- **IndentationError**: ensure the personalization block is inside `PersonalizedSearchAgent.search()`.
- **Emails missing**: increase `max_pages` slightly or manually click the homepage/“Contact” links.

---

## Ethics & Compliance
- Don’t store sensitive personal data without consent.
- Verify emails before outreach; prefer contact forms when requested.
