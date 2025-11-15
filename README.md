# Rearc Data Quest – End-to-End Data Pipeline

This repository contains my full implementation of the Rearc Data Quest, including S3 ingestion, API sourcing, data analytics, and an automated AWS serverless pipeline deployed using Terraform.

---

## 📌 Project Structure

rearc-data-quest/
├── src/
│ ├── part1_sync_bls.py
│ ├── part2_fetch_population.py
│ └── rearc_data_quest_part3.ipynb
│
├── lambda_ingest/ # Lambda that runs Part 1 + Part 2
│ ├── main.py
│ ├── part1_sync_bls.py
│ ├── part2_fetch_population.py
│ └── (third-party dependencies)
│
├── lambda_analytics/ # Lambda that runs Part 3 queries
│ └── main.py
│
├── terraform/
│ ├── main.tf
│ ├── terraform.tfvars
│ └── .terraform.lock.hcl
│
├── requirements.txt
├── .gitignore
└── README.md



---

## 🧩 Part 1 — BLS Data → S3 Sync

**Goal:**  
Fetch all `.txt` time-series files from the BLS Productivity dataset and keep them synchronized in S3.

**Highlights:**
- Automatically discovers all files (no hard-coding)
- Downloads only new/changed files
- Pushes to S3 bucket  
- Handles 403 errors by using a valid `User-Agent`
- Scripts reused inside Lambda

**Code:**  
`src/part1_sync_bls.py`

---

## 🧩 Part 2 — Population API → S3

**Goal:**  
Fetch U.S. population data from:
https://honolulu-api.datausa.io/tesseract/data.jsonrecords?cube=acs_yg_total_population_1&drilldowns=Year,Nation&measures=Population


**Highlights:**
- Stores *all* years available (not only 2013–2018)
- Normalizes JSON into simple `{year, population}` rows
- Saves result in S3 at:  
  `rearc-data-quest/population/us_population_all_years.json`

**Code:**  
`src/part2_fetch_population.py`

---

## 🧩 Part 3 — Analytics (Notebook)

**Goal:**  
Use Pandas to answer three analytical questions:

### 1️⃣ Population Mean & Std Dev (2013–2018)
Computed from the API dataset.

### 2️⃣ Best Year for Each Series
For every `series_id`:
- Sum Q01–Q04 for each year
- Pick the year with the highest total

Resulting dataframe includes:
series_id | year | year_sum


### 3️⃣ Population + Value Join  
For:
series_id = PRS30006032 and period = Q01


Joined with population data for the matching year.

**Notebook:**  
`src/rearc_data_quest_part3.ipynb`

---

## 🧩 Part 4 — Automated Pipeline (Terraform)

This is the full AWS pipeline orchestrating the quest.

### ✅ Resources created
- **S3 Bucket** (pre-existing, injected via variables)
- **Lambda #1: Ingest Function**
  - Runs Part 1 and Part 2
  - Scheduled daily via EventBridge
- **SQS Queue**
  - Triggered when new population JSON is uploaded
- **Lambda #2: Analytics Function**
  - Reads BLS + population data
  - Executes analytics from Part 3
  - Logs results to CloudWatch

### 🔧 Deployment
Inside the `terraform/` directory:

```bash
terraform init
terraform plan
terraform apply

Terraform code:
terraform/main.tf

📤 Deliverables Submitted
Part	Deliverable
Part 1	Python script + S3 dataset
Part 2	Python script + S3 JSON
Part 3	Jupyter Notebook (.ipynb) with queries & results
Part 4	Terraform Infrastructure as Code

📝 Notes on AI Usage

I used AI as a helper for:
Debugging Python
Understanding AWS/Terraform syntax
Structuring the project
All final code was tested, fixed, and validated manually.
