# Veridion POC -- Entity Resolution Summary

## Data Cleanup & Context

When I first opened the dataset, the raw CSV export couldn't be evaluated directly. Several rows had bad line wraps and extra delimiters bleeding into the final column. Before jumping into candidate matching, I normalized the structure and tied every record back to its original input row key so every match or flag remains fully traceable.

In total, the benchmark covers **592 input companies** evaluated against **3,041 total rows** (2,951 Veridion candidate profiles plus 90 external reference records pulled from corporate registries and official filings where the candidate profiles were incomplete or ambiguous).

## Matching Logic

Relying strictly on string similarity for company names leads to false positives (like confusing a holding parent with a local operating branch). To avoid this, I evaluated candidates using a clear hierarchy:

1. **Legal Name & Registration Number:** The gold standard for unambiguous identification.
2. **Country & Jurisdiction:** A country discrepancy is an immediate red flag—usually signaling a foreign branch, a parent entity, or a completely unrelated business.
3. **Address Details:** Checking street, city, and postal code to differentiate registered headquarters from regional sites or branch offices.
4. **Website Domain:** Good for supporting context, but a shared brand domain alone doesn't prove two entities are the exact same legal unit.

Whenever a profile looked close on name but failed on legal entity structure or address alignment, I kept it in **Manual Review** rather than forcing an unsafe match.

## Key Results

Here is the high-level breakdown across the **592 input companies** in the final workbook (`Veridion_POC_Submission_Final_v3.xlsx`):

| Outcome | Companies | Share | 
| ----- | ----- | ----- | 
| **Veridion Profile Match** | 251 | 42.4% | 
| **External Legal Match** | 81 | 13.7% | 
| **Total Confirmed Matches** | **332** | **56.1%** | 
| **Manual Review Required** | 260 | 43.9% | 
| **Total Input** | **592** | **100.0%** | 

* **56.1% of companies (332 total)** are resolved: 251 matched directly to a strong Veridion profile, and 81 were verified using official external sources (such as trade registers and official annual filings).
* **43.9% of companies (260 total)** remain in Manual Review, where ambiguity prevents automatic linking without risk.

At the individual candidate row level (3,041 rows in `Entity Resolution`):

* **332 Match** decisions (251 Veridion candidate profiles + 81 external references)
* **345 Manual Review** decisions (340 Veridion candidate profiles + 5 external references)
* **2,360 Unmatched** candidate profiles
* **4 Evidence-only** reference records

## Triage for the Manual Review Queue

To help procurement and data teams tackle the **260 unresolved companies**, I grouped them by root cause in the `MR_Triage_Backlog` tab:

* **Website / Domain Missing or Unverified (188 companies / 188 rows):** Name matches, but there is no verified domain to confirm the match safely.
* **Address / Location Discrepancy (58 companies / 58 rows):** Name aligns, but postcodes, cities, or street addresses clash with official records.
* **Group Structure / Status Issues (13 companies / 15 rows):** Complex setups requiring human validation (e.g., historical mergers, parent vs. local branch distinction).
* **General Candidate Review (1 company / 1 row):** Single-case spot check against local public registers.

## Notable Edge Cases

* **ELGIGANTEN GROSSIST A/S:** This historical Danish entity was dissolved following a merger. The active Elgiganten entity profile is not identical to this dissolved legal record, so they were kept separate to avoid linking inactive vendor accounts.
* **3Step IT:** The Norwegian (*3 Step IT AS*) and Swedish (*3 Step IT Sweden AB*) entities are distinct legal entities with their own tax IDs. Matching both to the Finnish group parent profile would obscure local operational contracts.
* **DNB Bank Karl Johan:** "Karl Johan" refers to a specific branch office location in Oslo, while *DNB Bank ASA* is the underlying legal entity. The location was flagged as a branch site under the main bank rather than a standalone corporate entity.
* **Sinal A/S:** *ENERGI HOBRO TELE A/S* is listed as a registered secondary name for Sinal A/S. While this supports the external reference, it doesn't mean brand profiles like Eniig or Norlys should be blanket-merged into Sinal.
* **Daily Jurrat:** Name matches closely, but country and address details did not line up. Kept in Manual Review to prevent false positive matching.

## Priority Cases Verified via External Sources

Four high-priority manual review cases were resolved using official registry data:

* **ISS Facility Services A/S:** Danish annual reports confirm CVR `14406042` and the primary address (*Gyngemose Parkvej 50, 2860 Søborg*). The candidate Veridion profile at a secondary address stays in Manual Review, while the external reference serves as the resolved match.
* **Vertiv Energy Private Limited:** Indian tribunal documents and statutory filings confirmed the legal identity and Thane (400604) facility.
* **Tongyu Communication Inc.:** Official corporate filings confirmed the legal name and registered office in Zhongshan (reconciling the input Guzhen location).
* **Telekom Malaysia Berhad:** Corporate disclosures confirmed the registered office on Jalan Pantai Baharu, Kuala Lumpur (highlighting an outdated postcode on the input file).

## Data Quality Insights & Next Steps

Analyzing the candidate dataset revealed several recurring patterns to address in production:

* **822 Country Mismatches:** Candidates often map to foreign parent companies. Strict country-matching rules should be enforced upfront.
* **671 Missing Websites:** Avoid auto-populating missing domains with third-party business directories or generic group sites.
* **1,380 Missing Legal Names:** Official registered legal names should be enriched directly from primary registries, keeping trade names in a secondary field.
* **180 Reused Veridion IDs:** Repeated candidate IDs across different input vendors must be audited so separate suppliers aren't accidentally merged in the ERP.

### Business Value

A clean entity mapping is critical for accurate spend analytics and vendor consolidation. Without reliable legal entity resolution, ERP spend analysis risks aggregating unrelated entities or splitting spend across duplicated vendor masters.

### Methodology Note

This review used spreadsheet automation and structured search routines to compare candidate records against public registries. While all 592 input companies have been assigned an outcome, individual candidate rows flagged for manual review should be verified against fresh registry extracts prior to ERP master data updates.
