Saudi VAT Compliance Analytics — Microsoft Fabric Portfolio Project

An end-to-end data analytics project built on Microsoft Fabric, simulating a Saudi ZATCA VAT compliance monitoring system. The project covers the full data pipeline — from messy, synthetic raw data through cleaning, business-logic modeling, and an interactive Power BI dashboard — and doubles as hands-on preparation for the DP-600 (Fabric Analytics Engineer) certification.

The Business Question

With limited regulatory review capacity, which taxpayers should be prioritized for compliance review — and why?

Every step in this project — from data cleaning to the final dashboard — is built to answer this question directly, not just to demonstrate individual technical skills in isolation.

Background & Domain Context

This project draws on the author's prior professional background: 10 years of experience in taxation in Sudan, applied here to the Saudi VAT (ZATCA) regulatory context. The synthetic dataset and business rules (15% VAT rate, e-invoicing compliance, violation categories) are modeled after Saudi ZATCA's real framework, adapted for a training/portfolio dataset.

Tech Stack
Microsoft Fabric — Lakehouse, Notebooks (PySpark + Spark SQL), Semantic Model (Direct Lake), Power BI Report
Python / pandas / numpy — synthetic dataset generation
SQL & PySpark — data cleaning and transformation
DAX — semantic model measures and calculated tables
Power BI — 4-page interactive dashboard
Architecture — Medallion (Bronze → Silver → Gold)
Bronze (raw CSVs)
   ↓  cleaning, deduplication, standardization (01_bronze_to_silver.ipynb)
Silver (silver_taxpayers, silver_invoices, silver_declarations, silver_violations)
   ↓  business-logic aggregation (02_gold_layer.ipynb)
Gold (gold_taxpayer_compliance_summary, gold_sector_summary)
   ↓
Semantic Model (Direct Lake) → Power BI Dashboard (4 pages)
Dataset

A synthetic, intentionally messy relational dataset simulating Saudi VAT compliance data, generated with Python (pandas/numpy):

Table	Rows (raw)	Description
taxpayers	816	Registered businesses — sector, VAT number, revenue, registration date
invoices	~6,090	E-invoices with VAT amounts, some miscalculated
vat_declarations	2,400	Periodic VAT filings — monthly or quarterly
violations	650	Recorded compliance violations and penalties

Deliberately injected data-quality issues: inconsistent date formats, typo/casing variants in categorical fields, mixed currency string formats, duplicate rows, orphan foreign keys, and a ~4% rate of miscalculated VAT (to simulate real anomaly detection scenarios).

Data Cleaning Highlights (Bronze → Silver)
Multi-format date parsing using COALESCE across 5 known date formats found in the raw data
Category standardization — sector and city typo/casing variants mapped via lookup tables (e.g. Mecca/Makkah merged)
Deduplication using ROW_NUMBER() OVER (PARTITION BY ...), keeping the most recent record per business key
Referential integrity check — invoices referencing a non-existent taxpayer_id are flagged (not silently dropped), simulating real-world system desync
VAT anomaly detection — invoices where vat_amount deviates from the legal 15% of amount_before_vat by more than 0.5 SAR are flagged (vat_anomaly_flag)
Deliberate missingness handling — NULLs that carry real meaning (e.g. a declaration never submitted, a violation never resolved) are preserved rather than imputed; a has_registration_date flag makes data completeness measurable rather than silently invisible
Gold Layer — Business Logic

gold_taxpayer_compliance_summary — one row per taxpayer, combining invoice, declaration, and violation history into a single compliance profile, including:

risk_score — a weighted signal of compliance issues (anomalous invoices ×2, late declarations ×2, missing declarations ×3, violations ×3)
red_flag — a separate indicator for taxpayers with more than 3 recorded violations, since repeat non-compliance is treated as a distinct risk signal from the financial value of any single violation
total_outstanding — real outstanding VAT arrears (متأخرات) per taxpayer

gold_sector_summary — the same signals rolled up by industry sector.

Key Findings
61 of 800 taxpayers (~7.6%) show a high compliance risk score (≥10) and warrant priority review.
Retail (5.43) and Construction (5.15) show the highest average sector risk scores; Technology (4.0) the lowest — consistent with the hypothesis that sectors with more mature digital invoicing systems show stronger compliance.
SAR 1.86M in total outstanding VAT arrears across all taxpayers.
~4% of invoices (240 of ~5,960) show a VAT calculation that doesn't match the legal 15% rate; the net effect across these anomalies is underreported VAT (revenue at risk to ZATCA), not overreported.
6 taxpayers are red-flagged for repeat violations (>3 recorded incidents).
Power BI Dashboard — 4 Pages

Designed with a custom dark navy theme (background 
#0F172A, card surface 
#1E293B) so that risk signals stand out clearly: red (
#F43F5E) marks high-risk/critical values, orange (
#FB923C) marks medium risk, and teal (
#22D3EE) marks normal/neutral values — a consistent color language used across all cards, charts, and tables.

Executive Overview — KPI cards, sector risk chart, top-10 highest-risk taxpayers table (with risk_score conditionally colored), key findings summary
Taxpayer Deep Dive — full interactive table of all 800 taxpayers, filterable by sector and red-flag status
Invoice Anomalies — flagged invoices with Expected VAT / Actual VAT / Difference, filtered to underreported cases (revenue at risk)
Compliance Timeline — violation counts by year and type, color-coded by violation category
Design Decisions Worth Noting
risk_score vs red_flag kept separate, rather than merged into one number — risk_score measures breadth of compliance issues, red_flag specifically measures repeat-offense behavior, since the two represent different kinds of risk worth surfacing independently.
Silver-layer aggregation before joining in the Gold layer — joining raw transaction tables (invoices, declarations, violations) directly on taxpayer_id would cause row multiplication (a fan-out); each source is pre-aggregated to one row per taxpayer first.
Direct Lake limitations encountered and worked around — an attempt to build a DAX calculated date table failed under Direct Lake mode (which doesn't support calculated tables using row-by-row scanning functions like MINX/MAXX); the date dimension was instead built as a physical Delta table (dim_date) via SQL, which Direct Lake can read natively.
Known Simplifications
VAT numbers in this dataset use a 17-character synthetic format rather than the real ZATCA 15-digit standard.
The referential-integrity issue (invoices with non-existent taxpayer IDs) does not occur in ZATCA's real, fully-automated e-invoicing system; it was intentionally injected here to practice detecting and handling data-quality issues common in less mature or migrating systems.
Project Structure
01_bronze_to_silver.ipynb   — raw data ingestion and cleaning (all 4 tables)
02_gold_layer.ipynb         — business-logic aggregation, risk scoring, dim_date
vat_compliance_model        — Fabric semantic model (Direct Lake)
VAT Compliance Dashboard    — 4-page Power BI report
Author

Salma — Data Analyst (Riyadh), PL-300 certified, currently preparing for DP-600 (Fabric Analytics Engineer). Background: 10 years in taxation (Sudan), transitioning into data analytics.
