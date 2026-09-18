# AI Job-Fit Scoring Automation

I got tired of manually reading every job posting and comparing it against my own skills, so I built a small n8n workflow that does it for me.

## What it does

Paste a job description in, and it sends the text to an LLM (Groq's `gpt-oss-120b`) along with my actual skill profile. The model comes back with a score out of 10, a list of what actually matches, and a list of what's missing — and the whole thing gets logged straight into my job tracker spreadsheet automatically.

## How it's built

- **n8n** running locally, triggered manually with a job description pasted in
- **Groq API** does the actual scoring — same model I used in my RAG chatbot project
- **Google Sheets** gets the result appended as a new row, via OAuth2

Five nodes total: trigger → hold the job text → call the API → pull the clean text out of the response → write it to the sheet.

## Sample output

```
Score: 6/10
Matching: Python, HuggingFace Transformers, LangChain, RAG pipelines, XGBoost, Streamlit, LLM experience
Missing: PyTorch, TensorFlow, scikit-learn, database knowledge, cloud experience, strong German proficiency
```

## The debugging part (honestly the real learning here)

This one fought back. Job descriptions sometimes have quotation marks in them like a line that said someone doesn't work *"towards"* but *"with"* their team. Pasting that straight into a JSON request body breaks the JSON, silently, right at that character. Took a while to actually pin down, because the error messages pointed at everything except the real cause.

Fixed it by switching the request body to an expression that builds the JSON with `JSON.stringify()` instead of typing it out by hand that escapes special characters automatically instead of me trying to do it manually and getting it wrong.

Also spent a good chunk of time on Google's OAuth setup creating a Cloud project, enabling the right APIs, setting up a consent screen, adding myself as a test user, generating client credentials. None of it is hard exactly, just a lot of small steps where one wrong click breaks the whole thing.

## Limitations, as of now

- Still a manual trigger no automatic job scraping (most sites like LinkedIn don't allow that anyway)
- One job at a time
- I still type the job title in myself rather than having it pulled from the posting

## Setup

1. Import the workflow JSON into n8n
2. Add your Groq key as a Header Auth credential
3. Connect Google Sheets via OAuth2 (needs Sheets + Drive APIs enabled on a Google Cloud project)
4. Swap the candidate profile text in the HTTP Request node for your own skills
