# The Assignment: "Trade Shipment Anomaly Detective"

#### You MAY use AI tools (ChatGPT, Copilot, Claude, etc.) — but read the [note at the bottom](#-a-note-on-using-ai).

---

## The Problem

You work for an Indian exporter that does **200+ shipments/month** to 15 countries. Their operations team currently reviews shipment data in Excel to catch problems — pricing anomalies, suspicious patterns, route inefficiencies, and compliance risks. They miss things constantly. Issues are caught weeks later during audits, costing lakhs in penalties and lost margins.

**Build an AI-powered anomaly detection system that ingests shipment data and automatically surfaces problems, patterns, and actionable recommendations.**

---

## The Data

You don't have real data. You'll **generate synthetic data** using a Python script. This is Part 1 of the assignment.

### Table 1: `shipments.csv` (500 rows)

> **Schema definition:** [`schemas/shipments_schema.csv`](schemas/shipments_schema.csv)

### Table 2: `buyers.csv` (8-10 rows)

> **Schema definition:** [`schemas/buyers_schema.csv`](schemas/buyers_schema.csv)


### Table 3: `product_catalog.csv` (15-20 rows)

> **Schema definition:** [`schemas/product_catalog_schema.csv`](schemas/product_catalog_schema.csv)

### Table 4: `routes.csv` (20-25 rows)

> **Schema definition:** [`schemas/routes_schema.csv`](schemas/routes_schema.csv)


---

## What You Must Plant in the Data

Your data generator must inject **at least 15 anomalies** across these 6 categories. **You decide the specifics** — be creative and make them realistic.

> **Full anomaly category definitions:** [`schemas/anomaly_categories.csv`](schemas/anomaly_categories.csv)

**Document every anomaly you plant** in a separate file `planted_anomalies.json`:

> **Example format:** [`schemas/planted_anomalies_example.json`](schemas/planted_anomalies_example.json)

```json
[
  {
    "anomaly_id": "PLANTED-001",
    "shipment_id": "SHP-2025-0127",
    "category": "compliance",
    "description": "HS code 84713000 (computers) assigned to Cotton T-shirts",
    "why_this_matters": "Wrong HS code leads to wrong duty calculation and potential customs penalty of ₹50K-₹2L"
  }
]
```

---

## What You Must Build

### Layer 1: Rule-Based Detection

Hard-coded checks that don't need ML or LLMs:

- `total_fob ≠ quantity × unit_price` (math validation)
- `drawback_amount` claimed when `customs_status = rejected`
- `payment_status = received` but `days_to_payment` is null
- `freight_cost = 0` when `incoterm = CIF` (CIF means seller pays freight)
- Insurance amount sanity check against FOB value

### Layer 2: Statistical Detection

Anomalies that need data analysis:

- Price outliers per product (unit price outside normal range from product catalog)
- Transit time outliers per route (outside range from routes table)
- Freight cost outliers per route + container type
- Payment behavior change per buyer (compare `days_to_payment` against `avg_payment_days` from buyers table)
- Volume spikes per buyer or per country per month

**You choose the statistical method.** Z-scores, IQR, rolling averages, Isolation Forest, whatever you think fits. **Justify your choice.**

### Layer 3: LLM-Powered Detection

Things that need reasoning:

- HS code vs. product description mismatch (ask the LLM: "Does HS code 84713000 correctly classify 'Cotton T-shirts 100% knitted'?")
- Cross-shipment pattern analysis (feed the LLM a buyer's recent shipments and ask: "Do you see any concerning patterns?")
- Generate the executive summary (see Output section)

**Key constraint:** You should NOT send all 500 rows to the LLM. That's expensive, slow, and lazy. Only send what Layers 1 and 2 can't handle. **Document how many LLM calls your system makes and why.**

---

## Output

### 1. Anomaly Report (`anomaly_report.json`)

> **Full schema:** [`schemas/anomaly_report_schema.json`](schemas/anomaly_report_schema.json)

```json
{
  "analysis_summary": {
    "total_shipments_analyzed": 500,
    "anomalies_detected": 18,
    "by_severity": {"critical": 0, "warning": 0, "info": 0},
    "by_category": {"pricing": 0, "compliance": 0, "route": 0, "payment": 0, "volume": 0, "cross_field": 0},
    "by_detection_layer": {"rule_based": 0, "statistical": 0, "llm_semantic": 0}
  },
  "anomalies": [
    {
      "anomaly_id": "ANO-001",
      "shipment_id": "SHP-2025-0127",
      "category": "compliance",
      "severity": "critical | warning | info",
      "detection_layer": "rule_based | statistical | llm_semantic",
      "description": "Human-readable description of what's wrong",
      "evidence": {
        "field": "hs_code",
        "actual_value": "84713000",
        "expected_value_or_range": "6109xxxx",
        "reference_data": "Product catalog shows HS 61091000 for Cotton T-shirts"
      },
      "business_impact": "What could go wrong if this isn't fixed",
      "recommended_action": "What should the operations team do"
    }
  ]
}
```

### 2. Executive Summary (`executive_summary.md`)

Use an LLM to generate a **1-page executive summary** from the anomaly report. Written for a non-technical Operations Head. Should include:

- Top 3 most urgent issues
- Trends (e.g., "Buyer X payment behavior deteriorating over 3 months")
- Cost implications (estimated penalty risk, savings opportunities)
- Recommended immediate actions

### 3. Detection Accuracy (`accuracy_report.json`)

Compare your system's detected anomalies against your `planted_anomalies.json`:

> **Full schema:** [`schemas/accuracy_report_schema.json`](schemas/accuracy_report_schema.json)

```json
{
  "planted_anomalies": 15,
  "detected_correctly": 0,
  "missed": 0,
  "false_positives": 0,
  "precision": 0.0,
  "recall": 0.0,
  "f1_score": 0.0,
  "missed_anomalies": ["PLANTED-003", "PLANTED-011"],
  "false_positive_details": [
    {
      "anomaly_id": "ANO-009",
      "shipment_id": "SHP-2025-0301",
      "why_flagged": "...",
      "why_its_actually_fine": "..."
    }
  ]
}
```

---

## Deployment

Deploy the complete system as a **live, accessible web app**. Use any free platform:

- **Streamlit Community Cloud** (free, easiest)
- **Hugging Face Spaces** (free)
- **Render / Railway** (free tier)
- **Any other** platform you prefer

### The deployed app must:

1. Show a **dashboard** with summary stats (total shipments, anomalies by category, severity breakdown)
2. Have an **anomaly table** — sortable, filterable list of all detected anomalies
3. Allow clicking an anomaly to see **full details** (evidence, impact, recommendation)
4. Display the **executive summary**
5. Have a **"Run Analysis"** button that re-runs the detection pipeline (so we can see it's not just hardcoded output)

**Bonus (optional but impressive):** Allow uploading a different CSV and running the analysis on new data.

**Submit the live URL.**

---

## What You Must Submit

```
your-submission/
├── data/
│   ├── shipments.csv
│   ├── buyers.csv
│   ├── product_catalog.csv
│   ├── routes.csv
│   └── planted_anomalies.json
├── src/
│   ├── data_generator.py
│   ├── rule_engine.py
│   ├── statistical_detector.py
│   ├── llm_detector.py
│   ├── report_generator.py
│   └── app.py                    # Streamlit/Gradio app
├── output/
│   ├── anomaly_report.json
│   ├── executive_summary.md
│   └── accuracy_report.json
├── DESIGN_DECISIONS.md
├── requirements.txt
└── README.md                     # Setup + deployed URL
```

---

## `DESIGN_DECISIONS.md` — THE MOST IMPORTANT FILE

Answer these questions. Be honest. Short answers are fine if they're clear.

1. **What anomalies did you plant and why?** Why are they realistic? What would each one cost an exporter if it went undetected?

2. **What statistical method(s) did you use in Layer 2 and why?** What did you consider and reject?

3. **What exactly did you send to the LLM?** How many calls? What was the total token usage / cost? Why did you draw the line between Layer 2 and Layer 3 where you did?

4. **Show one prompt that didn't work** and explain how you iterated to fix it. Include the bad prompt, what it got wrong, and the improved version.

5. **What are your precision and recall numbers?** Why did the system miss what it missed? Why did it false-positive what it false-positived?

6. **What are 3 anomaly types your system CAN'T catch** but would matter in real trade? How would you solve them with more time/data?

7. **If this ran on 50,000 shipments instead of 500**, what breaks? What would you change architecturally?

---

## Evaluation Criteria

> **Full criteria table:** [`schemas/evaluation_criteria.csv`](schemas/evaluation_criteria.csv)

---

## A Note on Using AI

**Use AI. Seriously.** Use ChatGPT, Copilot, Claude, whatever helps you. That's a real-world skill.

**But here's what we'll check:**

We will ask you to **walk us through your code in a 15-minute call**. We'll ask things like:

- *"Why did you use IQR instead of Z-scores here?"*
- *"What does this prompt do and why did you structure it this way?"*
- *"If I change this buyer's payment history, what happens?"*
- *"This anomaly was a false positive — why?"*

If you can't answer these, it doesn't matter how polished your code is.

**The goal isn't to test if you can avoid AI. The goal is to test if you can THINK WITH AI as a tool, not use AI as a replacement for thinking.**

Good luck. Build something cool. 🚀
