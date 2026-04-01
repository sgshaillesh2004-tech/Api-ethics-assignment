**Task 1 — Classify and Handle PII Fields**

| Field Name          | Type of PII             | Action                  | Reason                                                                                         |
| ------------------- | ----------------------- | ----------------------- | ---------------------------------------------------------------------------------------------- |
| **full_name**       | Direct PII              | ❌ Drop                  | Directly identifies a person. Not needed for research.                                         |
| **email**           | Direct PII              | ❌ Drop                  | Personally identifiable and sensitive. No research value.                                      |
| **date_of_birth**   | Indirect PII            | 🔒 Mask                 | Can identify individuals when combined with other data. Keep only age or year.                 |
| **zip_code**        | Indirect PII            | 🔒 Mask                 | Can reveal identity in small areas. Keep partial (e.g., first 3 digits).                       |
| **job_title**       | Indirect PII            | 🔁 Pseudonymize         | May identify individuals in small datasets. Replace with category (e.g., “Healthcare Worker”). |
| **diagnosis_notes** | Sensitive Data (Health) | 🔁 Pseudonymize / Clean | May contain personal info. Remove names or identifiers before sharing.                         |


**Task 2 — Audit the API Script for Ethical Compliance**

Violation 1: Hardcoding API Key
Problem:
API key is exposed in the code.
Anyone can misuse it.
Violates security best practices and possibly API Terms of Service.
Fix:

Store API key securely using environment variables.


import os
import requests

API_URL = "https://healthstats-api.example.com/records"
API_KEY = os.getenv("API_KEY")  # Secure way

records = []
for page in range(1, 101):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})
    data = response.json()
    records.extend(data["results"])



Violation 2: Collecting & Storing Excessive Data Without Limits
Problem:
Fetching 100 pages without checking:
API rate limits
User consent
Data minimization principle
Storing all raw data permanently, including PII.
Fix:
Add rate limiting
Collect only required fields
Avoid storing raw sensitive data

import time
import os
import requests

API_URL = "https://healthstats-api.example.com/records"
API_KEY = os.getenv("API_KEY")

records = []

for page in range(1, 101):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})
    
    if response.status_code != 200:
        break
    
    data = response.json()
    
    # Keep only required (non-PII) fields
    for item in data["results"]:
        cleaned_record = {
            "age": item.get("date_of_birth"),  # later convert to age
            "zip_prefix": str(item.get("zip_code"))[:3],
            "job_category": item.get("job_title"),
            "diagnosis": item.get("diagnosis_notes")
        }
        records.append(cleaned_record)
    
    time.sleep(1)  # Respect API rate limits

# Save only cleaned data
save_to_database(records)
