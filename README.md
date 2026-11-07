# 📊 AI Financial Data Analyst Agent

An autonomous AI agent built with **n8n** that queries a PostgreSQL database using natural language, analyzes financial trends, and generates executive-grade reports with deterministic, hallucination-free visual charts.

This project was built to analyze U.S. Federal Government financial data (Gross Cost, Earned Revenue, and Net Cost) but can be easily adapted to any business SQL database.

## 🚀 Features

* **Natural Language to SQL:** Uses an AI Agent to translate user questions into valid PostgreSQL queries.
* **Model-Agnostic Architecture:** Designed to work flawlessly on free-tier and open-source models (via OpenRouter, Groq, or Gemini) without relying on fragile multi-turn tool calling.
* **Deterministic Chart Generation:** Bypasses LLM URL hallucinations by decoupling the charting logic from the agent. The LLM outputs a hidden JSON data block, which a custom JavaScript node parses to build a perfect QuickChart.io visualization.
* **Executive Formatting:** Automatically structures outputs into professional Markdown reports featuring Executive Summaries, Key Figures tables, and Strategic Recommendations.

## 🧠 The Engineering Challenge & Solution

**The Problem:** Standard LLM function-calling often fails when asked to generate image URLs (like QuickChart). Smaller or free-tier models tend to ignore the tool execution step, hallucinate invalid JSON structures, or output broken, unencoded URLs with raw syntax directly into the chat.

**The Solution (Decoupled Generation):** 
Instead of forcing the LLM to execute a charting tool, this workflow isolates responsibilities:
1. **The AI Agent** executes the SQL query, analyzes the numbers, writes the report, and appends a hidden `<CHART_DATA>` JSON block at the bottom of its response.
2. **A Post-Processing Code Node** uses Regex to intercept the response, extracts the JSON block, strictly formats and URL-encodes a Chart.js configuration, and dynamically injects the rendered Markdown image back into the text.

This guarantees a **100% reliable chart generation** on every run, regardless of which LLM provider is used.

## 🏗️ Architecture

1. **Trigger:** User submits a prompt via the n8n Chat interface.
2. **AI Agent Node:** Analyzes the prompt and decides to use the `query_database` tool.
3. **PostgreSQL Tool:** Executes the `SELECT` statement and returns ground-truth financial data.
4. **AI Generation:** The LLM synthesizes the SQL output into a structured report + a hidden `<CHART_DATA>` block.
5. **Code Node (JS):** Extracts the data block, encodes a QuickChart URL, and formats the final Markdown response.

## 🛠️ Prerequisites

* **n8n:** A local or cloud-hosted instance of [n8n](https://n8n.io/).
* **PostgreSQL:** A database containing your dataset.
* **LLM API Key:** An API key for OpenRouter, Google Gemini, Groq, or OpenAI.

## 📦 Setup & Installation

### 1. Database Setup
Create a PostgreSQL table named `federal_financials` and populate it with your data.

```sql
CREATE TABLE federal_financials (
    id SERIAL PRIMARY KEY,
    record_date DATE,
    fiscal_year INT,
    agency_name VARCHAR(255),
    gross_cost_billions NUMERIC(10,2),
    earned_revenue_billions NUMERIC(10,2),
    net_cost_billions NUMERIC(10,2)
);
```

### 2. Import the Workflow
1. Download the `AI_Data_Analyst_Agent.json` file from this repository.
2. Open your n8n workspace, click **Add Workflow**, and select **Import from File**.
3. Upload the `.json` file.

### 3. Configure Credentials
* Double-click the **PostgreSQL tool** and connect it to your database.
* Double-click the **Chat Model node** and enter your API key (e.g., OpenRouter, Gemini). 

## 💬 Usage Example

Start a chat session in n8n and try the following prompt:

> *"What was the Department of Energy's net cost over the past 5 years? Include a trend chart and write an executive report."*

**Expected Output:**
The agent will query the database, calculate the YoY growth, provide strategic recommendations, and embed a fully rendered trend line chart comparing the last 5 years of data.

## 🔮 Future Improvements

* **Dynamic Chart Types:** Update the JavaScript node to dynamically switch between `bar`, `line`, and `pie` charts based on a parameter provided by the LLM.
* **Export to PDF:** Add a downstream branch to convert the Markdown output to PDF and email it to stakeholders.
* **Mathematical Verification Agent:** Introduce a secondary validation step to mathematically verify the LLM's percentage-change calculations before delivering the report.

---
*Built as a portfolio project demonstrating robust AI workflow engineering, prompt constraints, and deterministic tool extraction.*
