# Project Brief: Login Anomaly Detection System

## 1. Dataset Card

| Field | Details |
| :--- | :--- |
| **Name** | Login Anomaly Dataset |
| **Publisher and URL** | M. Umair Khattak on Kaggle: [Login Anomaly Dataset](https://www.kaggle.com/datasets/mmumairkhattak/login-anomaly-dataset/data) |
| **Licence (exact name and link)** | Check Kaggle's platform terms / dataset specific open license (e.g., CC0, CC-BY, or Database Open License as listed on Kaggle) |
| **Download date and version or last update** | October 2026, Latest version available on Kaggle |
| **Grain: what is one row?** | A single login attempt by a user (containing session details, timestamps, IP addresses, device info, and anomaly indicators). |
| **Period covered** | Historical logs spanning 12+ months (as required for seasonality and trend analysis). |
| **Size: rows x columns, MB** | $\ge$ 5,000 rows $\times$ multiple columns (exact size depending on downloaded CSV file dimensions). |
| **Key columns: dates, dimensions, measures, candidate target** | **Dates:** Timestamp/Date of login attempt.<br>**Dimensions:** User ID, Country/Location, Device Type, Browser, IP Address.<br>**Measures:** Login duration, failed attempt counts, request rates.<br>**Candidate target:** `Is_Anomaly` or binary flag indicating whether a login attempt is anomalous or fraudulent. |
| **Known quality issues you already spotted** | Potential missing values in optional metadata fields, IP address formatting inconsistencies, or class imbalance in the target anomaly variable. |
| **Other sources you plan to join, if any** | Geo-IP lookup databases (to map IP addresses to countries/coordinates) or threat intelligence blacklists. |

---

## 2. Describe the Business Problem

### 2.1 Organisation, department and stakeholder
* **Organisation:** A global mid-sized e-commerce and digital services platform handling hundreds of thousands of customer authentications daily, where account takeovers and credential stuffing pose severe security and financial risks.
* **Department:** The Fraud Prevention and Identity Security Operations Team (a specialized unit of 6 security analysts).
* **Stakeholder:** The Lead Security Operations Center (SOC) Analyst. They review the dashboard and automated risk feeds daily, tuning alerting thresholds and deciding which high-risk accounts require immediate locking, multi-factor authentication (MFA) step-up enforcement, or manual investigation.

### 2.2 The decision to support
* **Core Decision:** "With this dashboard and analytical engine, the stakeholder will decide which automated security rules and risk thresholds to adjust daily to block malicious login anomalies while minimizing friction for legitimate users."
* **Constraints:** Must operate within a strict privacy and data protection framework (e.g., GDPR/CCPA guidelines regarding user IP and location logging), limited analyst bandwidth (cannot manually review more than 50 high-risk alerts per day), and API response time limits for real-time risk scoring.

### 2.3 Hypotheses
* **H1 (Geographical Improbability):** 
  * *Statement:* More than 70% of successful login anomalies involve an impossible travel scenario (subsequent logins from vastly distant geographic locations within an unrealistically short time window).
  * *Metric:* Percentage of flagged login anomalies originating from distant geo-locations within a <2-hour window.
  * *Expected direction:* High concentration of anomalies violating speed-distance physical limits.
  * *How you test it:* Scatter plots and time-delta calculations between consecutive user login timestamps and geographic coordinates (Lessons 03-05).
  * *If true →* Enforce automatic geo-velocity checking and mandatory step-up MFA for high-velocity shifts.
  * *If false →* Reallocate rule weights toward device fingerprinting and unusual behavioral hours rather than location.

* **H2 (Device & Browser Spoofing):** 
  * *Statement:* Credential stuffing and brute-force attacks predominantly use headless browsers or randomized, uncommon user-agent strings compared to normal traffic.
  * *Metric:* Ratio of anomalous login attempts associated with rare/headless user-agents versus standard consumer browsers.
  * *Expected direction:* Anomalies will show a heavily skewed distribution toward non-standard user-agents.
  * *How you test it:* Categorical aggregation and bar charts grouped by browser type and device fingerprint against the anomaly target variable.
  * *If true →* Automatically block or challenge login requests originating from unrecognized headless user-agents.
  * *If false →* Treat user-agent parameters as secondary signals and rely more heavily on behavioral timing metrics.

* **H3 (Temporal Clustering / Off-Hours Activity):** 
  * *Statement:* Unauthorized login attempts heavily cluster during local off-hours (midnight to 4:00 AM) relative to the user's historical active timezone.
  * *Metric:* Hourly distribution density of anomalous vs. normal login attempts.
  * *Expected direction:* Significant spike in anomaly proportion during local nighttime hours.
  * *How you test it:* Hourly time-series line charts and distribution density plots.
  * *If true →* Apply stricter risk scoring algorithms for login attempts occurring during a user's typical sleeping hours.
  * *If false →* Maintain uniform temporal risk thresholds throughout the 24-hour cycle.

### 2.4 KPIs
1. **Anomaly Detection Rate (ADR):** Total number of correctly flagged malicious login attempts divided by total actual attacks, tracked daily, benchmarked against a target of $\ge 95\%$.
2. **False Positive Rate (FPR):** Percentage of legitimate user logins incorrectly flagged as anomalies, tracked per day, with an acceptable threshold of $< 2\%$ to avoid user friction.
3. **Average Investigation Time:** Mean time taken by SOC analysts to review and resolve a flagged login anomaly alert, measured per shift, targeting under 5 minutes per alert.
4. **Credential Stuffing Volume:** Total count of blocked automated brute-force attempts per hour, grouped by source country/IP network, directly supporting hypothesis H3.

---

## 3. Design the Solution

### 3.1 Solution type
* **Combined Solution (Analytical Engine + Management Dashboard):** The system acts primarily as an **analytical engine** that ingests raw login logs, runs anomaly detection rules and classification models (Block 3), and outputs a prioritized risk score and alert queue. This feeds a **management dashboard** refreshed daily for the Lead SOC Analyst to monitor system performance, track KPIs, and adjust security policies.

### 3.2 General architecture

```mermaid
flowchart LR
    A[Source<br/>Kaggle CSV / Login Logs] --> B[Ingestion<br/>download script, pandas]
    B --> C[Cleaning & quality<br/>types, nulls, duplicates, outliers]
    C --> D[Data model<br/>fact table + dimensions, parquet]
    D --> E[Analytics / engine<br/>KPIs, rules, anomaly model]
    E --> F[Dashboard<br/>Plotly / Streamlit or Power BI]
    F --> G((Stakeholder<br/>SOC Analyst Decision))
