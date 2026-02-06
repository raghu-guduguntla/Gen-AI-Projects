# Fact-Checking Agent for Instagram Reels — Built on n8n

This repository contains an **n8n workflow** that takes an **Instagram Reel URL**, extracts the audio, transcribes it, identifies factual claims, verifies them using live web search, and generates a **human-readable authenticity report**.

## What This Workflow Does (High-Level)

1. Accepts an **Instagram Reel URL** via a webhook
2. Downloads the reel metadata and media using **Apify**
3. Extracts and uploads audio to **AssemblyAI**
4. Transcribes spoken content
5. Uses an **AI agent** to:

   * Identify factual claims
   * Search the web for verification
   * Assign verdicts (TRUE / FALSE / PARTIALLY_TRUE / UNVERIFIABLE)
6. Produces:

   * Claim-level verdicts + reasoning + sources
   * An overall authenticity score
7. Returns a **clean HTML report** to the user

---

## Input & Output

### Input (Webhook)

```json
{
  "reelUrl": "https://www.instagram.com/reel/XXXXXXXX/"
}
```

### Output

* A **rendered HTML report** showing:

  * Total claims
  * Verdicts per claim
  * Explanations and sources
  * Overall authenticity summary

---

## Node-by-Node Breakdown

### 1. Webhook (Trigger)

**Node:** `Webhook`

* Entry point of the workflow
* Accepts a POST request containing `reelUrl`
* Makes the workflow usable from:

  * A frontend
  * Postman
  * Browser form
  * Another automation

---

### 2. Edit Fields

**Node:** `Edit Fields`

* Extracts `reelUrl` from webhook body
* Adds a timestamp
* Normalizes input for downstream nodes

---

### 3. Workflow Configuration

**Node:** `Workflow Configuration` (Set)

* Central place to store:

  * Reel URL
  * Execution timestamp
* Makes values easily reusable across nodes

---

### 4. Download Instagram Reel Metadata

**Node:** `Download Instagram Reel (Apify metadata)`

* Calls **Apify’s Instagram Reel Scraper**
* Fetches:

  * Video metadata
  * Media URLs
* Uses Apify API

⚠️ **You must replace the Apify API token with your own**

---

### 5. Extract Media URL

**Node:** `Extract Media URL` (Code)

* Safely extracts the reel’s video/audio URL
* Handles variations in Apify response fields
* Throws a clear error if media URL is missing

---

### 6. Download Reel Media

**Node:** `Download Reel Media (File)`

* Downloads the actual reel media file
* Outputs binary data for transcription

---

### 7. AssemblyAI – Upload

**Node:** `AssemblyAI: Upload`

* Uploads reel audio/video to AssemblyAI
* Returns a hosted `upload_url`

⚠️ **Replace AssemblyAI API key with your own**

---

### 8. AssemblyAI – Create Transcript

**Node:** `AssemblyAI: Create Transcript`

* Starts transcription job
* Enables punctuation for cleaner text

---

### 9. Wait + Poll Loop

**Nodes:**

* `Wait (poll)`

* `AssemblyAI: Get Transcript`

* `If Not Completed`

* Polls AssemblyAI until transcription is complete

* Loops automatically if status is `queued` or `processing`

* Proceeds only when transcript is ready

---

### 10. Transcript (Final)

**Node:** `Transcript (final)` (Set)

* Extracts final transcript text
* Attaches reel URL for context
* This is the **input to the AI agent**

---

### 11. Fact-Checking AI Agent

**Node:** `Fact-Checking AI Agent`

This is the **core intelligence** of the workflow.

The agent:

* Receives the transcript
* Identifies factual claims
* Uses tools to verify them
* Produces structured results

#### Connected AI Components

**Language Model**

* `OpenAI Chat Model`

**Tool**

* `Google Search Tool` (SERP API)

**Output Parser**

* `Structured Output Parser`

The agent is instructed to:

* Use multiple trusted sources
* Provide verdicts:

  * TRUE
  * FALSE
  * PARTIALLY_TRUE
  * UNVERIFIABLE
* Return machine-readable JSON

---

### 12. Format Final Report

**Node:** `Format Final Report` (Set)

* Restructures AI output into a clean report format
* Prepares data for rendering

---

### 13. HTML Report Generator

**Node:** `Code in JavaScript`

* Converts structured data into a **styled HTML page**
* Adds:

  * Verdict badges
  * Source links
  * Explanation sections
* Escapes unsafe content
* Mobile-friendly layout

---

### 14. Respond to Webhook

**Node:** `Respond to Webhook`

* Sends the HTML report back to the requester
* Final user-visible output

---

## How to Use This Workflow

### Step 1: Import

* Download the workflow JSON
* In n8n → **Import workflow**

### Step 2: Add Credentials

You must configure:

* **Apify API Key**
* **AssemblyAI API Key**
* **OpenAI API Key**
* **SERP API Key**

### Step 3: Activate Webhook

* Copy webhook URL
* Send POST request with `reelUrl`

---

## Customization Ideas

* Replace Instagram with **YouTube Shorts / TikTok**
* Swap Google Search tool for **Perplexity / Tavily**
* Store reports in:

  * Notion
  * Supabase
  * Google Sheets
* Add confidence thresholds (e.g. flag below 50%)

---

## Limitations

* Private Instagram reels won’t work
* Claims that lack public data may be marked **UNVERIFIABLE**
* Accuracy depends on:

  * Search results
  * Source quality
  * Transcript quality

---

## Why This Workflow Is Useful

* Fights misinformation at scale
* Demonstrates **agentic AI patterns in n8n**
* Great reference for:

  * LLM + tools
  * Structured outputs
  * Polling async APIs
  * HTML generation inside workflows

---

If you want, I can also:

* Add a **diagram**
* Rewrite this as a **GitHub-polished README**
* Create a **lite version** for beginners
* Convert this into a **product demo narrative**

Just tell me 👍
