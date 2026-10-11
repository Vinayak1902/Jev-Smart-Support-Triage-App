# Smart Support Triage with Jev

Smart Support Triage with Jev is a lightweight customer-support demo that uses the TypeSafe Jev API to analyze incoming support tickets, route them to the right team, estimate urgency, flag refund risk, and decide when a human review is needed.

The application combines structured AI classification with a simple FastAPI web interface, making it easy to demonstrate how a ticket triage system could work in a real SaaS support workflow.

## What this project is

This project is designed to:

- classify tickets into department buckets such as billing, technical, account, or general
- score how urgent the issue is
- estimate whether the customer is asking for a refund or reversal
- predict whether a human agent should review the case before automated routing
- provide a suggested customer-facing reply, when a Groq API key is configured

In short, this is a small AI-powered support triage system for demo and learning purposes. It is not a full enterprise help desk, but it demonstrates the core decision logic that many customer support platforms use.

## High-Level Design (HLD)

The application follows a simple layered design:

1. Frontend layer
   - A FastAPI app serves an HTML page and static assets from the `templates/` and `static/` directories.
   - The browser sends a ticket to the backend for analysis.

2. API layer
   - `app.py` defines the web server and exposes the `/` and `/api/analyze` endpoints.
   - The `/api/analyze` route accepts ticket text and returns triage results and a suggested reply.

3. Decision service layer
   - `services/jev_service.py` calls the TypeSafe Jev API with a structured ticket prompt.
   - Jev returns department, urgency, refund likelihood, and human-review probability.

4. Routing layer
   - `app.py` contains deterministic business rules that decide whether the ticket should stay in a normal queue or be escalated to a human specialist.
   - Escalation is triggered when the confidence is low or the risk/urgency thresholds are exceeded.

5. Optional reply generation layer
   - `services/response_chain.py` uses LangChain + Groq to draft a concise customer response.
   - If `GROQ_API_KEY` is not configured, the system still works and shows the triage results without the generated reply.

Architecture summary:

```text
Browser / UI
   |
   v
FastAPI app (app.py)
   |
   +--> Ticket analysis via TypeSafe Jev API
   |
   +--> Route decision logic
   |
   +--> Optional response draft via Groq / LangChain
   |
   v
JSON response to UI
```

## Project structure

```text
.
├── app.py                     # FastAPI entry point
├── README.md                 # Project documentation
├── requirements.txt          # Python dependencies
├── demo_tickets.json         # Example tickets for demo/testing
├── .env.example              # Example environment file (if present)
├── services/
│   ├── __init__.py
│   ├── jev_service.py         # TypeSafe API integration
│   └── response_chain.py     # Optional Groq reply generation
├── static/
│   ├── app.js
│   └── style.css
├── templates/
│   └── index.html
└── .gitignore
```

## Prerequisites

- Python 3.10 or newer
- A TypeSafe API key
- Optional: a Groq API key for generated response text

## How to run the project

1. Open a terminal in the project folder.

2. Create and activate a virtual environment:

```bash
python -m venv .venv
```

On Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

On macOS/Linux:

```bash
source .venv/bin/activate
```

3. Install dependencies:

```bash
pip install -r requirements.txt
```

4. Create a `.env` file in the project root with your credentials:

```env
TYPESAFE_API_KEY=your_typesafe_api_key
TYPESAFE_MODEL=jev-latest

# Optional, for generated customer replies
GROQ_API_KEY=your_groq_api_key
GROQ_MODEL=llama-3.1-8b-instant
```

5. Start the app:

```bash
python app.py
```

Or with Uvicorn directly:

```bash
uvicorn app:app --host 0.0.0.0 --port 8080 --reload
```

6. Open the app in your browser:

```text
http://localhost:8080
```

## How it works in practice

- Paste a customer support ticket into the web page.
- The backend sends the ticket to the Jev API.
- Jev returns structured answers about department, urgency, refund likelihood, and human review requirements.
- The app applies internal business rules to route the ticket.
- If configured, the app also drafts a short support reply.

## Notes

- The app is intentionally simple and beginner-friendly.
- The triage decision is still useful even without Groq, because the core classification logic comes from Jev.
- The optional reply generation is intentionally non-blocking; if Groq fails, the triage summary still appears.

## Demo and extension ideas

This project can be extended with:

- persistent ticket storage
- queue prioritization and SLA tracking
- real CRM integration
- a dashboard for team metrics
- model and rule tuning for different business domains

## License

This project is provided as a demo and learning example for customer-support triage workflows.