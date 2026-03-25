# 🎯 Logic Log: Lead Sniper Workflow

## ⚙️ Handling GitHub API Rate Limits

**The Constraint:**
GitHub's API enforces strict rate limits depending on your authentication state:
* **Unauthenticated:** 60 requests/hour
* **Authenticated:** 5,000 requests/hour

**Implementation & Strategy:**
To ensure reliable execution without hitting limits, I implemented the following architecture:
* **Authentication:** Authenticated using a GitHub Personal Access Token (PAT).
* **Headers Configured (HTTP Node):**
    * `Authorization: Bearer <TOKEN>`
    * `Accept: application/vnd.github+json`
* **Pagination Control:** Restricted API payload sizes using the query parameter `?per_page=10`.
* **Pacing:** Utilized a **Loop Over Items** node rather than bulk processing to pace the API requests safely.

> **Outcome:** This setup eliminated rate limit errors entirely, ensuring smooth and reliable execution across all users.

---

## 🛠️ Problems Faced & Solutions

### 1. The `[object Object]` Output
* **Problem:** The AI output was returning a nested JSON object rather than a clean text string for the pitch.
* **Solution:** Bypassed the object wrapper by extracting the precise text field using exact notation: 
    `{{ $json["output"][0]["content"][0]["text"] }}`

### 2. The `pairedItemInvalidIndex` Error
* **Problem:** Referencing `$node[...]` inside a loop caused a data mismatch with the paired items, breaking the execution cycle.
* **Solution:** Switched to referencing the local item data using `{{ $json[...] }}`, which remains contextually accurate inside the loop.

### 3. Invalid JSON in Slack Node
* **Problem:** Mixing raw JSON structure with dynamic expressions in the Slack node led to syntax errors and broken payloads.
* **Solution:** * Changed the node configuration to **"Using Fields Below"** (leveraging UI mapping instead of raw JSON).
    * Unified the output using a single JavaScript expression: `{{ "text" + value }}`.

### 4. Order Mismatch (Wrong Pitch Assigned)
* **Problem:** Data lost alignment across nodes, resulting in users receiving the wrong personalized pitches.
* **Solution:** Restructured the node architecture to enforce strict data lineage. The corrected, linear flow is:
    `Loop` ➡️ `Profile` ➡️ `IF` ➡️ `AI` ➡️ `Slack`
