# MCP + A2A Multi-Agent Customer Support System

This project implements a multi-agent customer support system using:

- **MCP (Model Context Protocol)** for database-backed tools (customers, tickets, billing context),
- **LangGraph** for **A2A (Agent-to-Agent)** coordination between a Router, Customer Data Agent, and Support Agent,
- A shared state structure for message passing and explicit state transitions.

The core of the implementation lives in two notebooks:

- `mcp_server.ipynb` – MCP server & database setup 
- `a2a_agents.ipynb` – LangGraph-based A2A orchestration that calls MCP tools

The database is a simple SQLite file, `support.db`, created and populated via `database_setup.py`.

---

## 1. Prerequisites

- **Python**: 3.10 or 3.11
- **pip**: latest version
- **virtualenv / venv**: standard Python `venv` module
- **Jupyter** (or JupyterLab) to run the notebooks

---

## 2. Install Dependencies

Create a file named requirements.txt in the project root with (at least) the following content:

```bash
# Core HTTP + MCP + web server
flask
flask-cors
requests

# Jupyter / notebook support
notebook
jupyterlab

# LangGraph (A2A orchestration)
langgraph

# Optional: environment loading, if you later use .env files
python-dotenv
```

Then install the dependencies:
```bash
pip install -r requirements.txt
```

---

## 3. Initialize / Reset the Database

The project uses a SQLite database file called support.db.

To create or reset it, run:

```bash
python database_setup.py
```

This script:

Creates the support.db file,

Initializes the customers and tickets tables,

Inserts some sample data for testing the MCP tools and A2A flows.

---

## 4. Run the MCP Server (`mcp_server.ipynb`)

The MCP server exposes database tools (e.g., get_customer, list_customers, get_customer_history) over HTTP using SSE and the MCP JSON-RPC protocol.

1. Open mcp_server.ipynb.

2. Run all cells (Kernel → Restart & Run All).

The notebook will:

- Start a Flask app on http://127.0.0.1:5000/mcp,

- Implement MCP endpoints

- Wrap database interactions (via support.db) inside MCP tools.

3. Keep this notebook running.

The A2A agent notebook will call this MCP endpoint via HTTP + SSE.

---

## 5. Run the LangGraph A2A Agents (`a2a_agents.ipynb`)

In a second terminal (or another Jupyter session), with the same virtual environment activated.

1. Open the A2A orchestration notebook (e.g., a2a_agents.ipynb).

2. Run all cells.

This notebook defines:

- A shared SupportState structure (TypedDict) that agents read/write,

- Three agents as LangGraph nodes:

    - Router – classifies scenario, extracts customer ID, routes between specialists,

    - CustomerDataAgent – calls MCP tools to fetch customer info / premium lists / billing context,

    - SupportAgent – creates human-readable support responses for each scenario.

- Conditional routing logic:

    - route_decision(state) decides which node to call next (CustomerDataAgent, SupportAgent, or END),

    - Agent-to-agent coordination for:

        - Scenario 1: Account help with a specific customer ID,

        - Scenario 2: Cancellation + billing issues with negotiation and billing context,

        - Scenario 3: High-priority ticket status for all premium customers.

3. At the bottom of the notebook, there are test queries. 

Running this cell will:

- Invoke the LangGraph graph for each scenario,

- Call MCP tools (through the HTTP/SSE client wrapper),

- Print:

    - the final support response, and
    
    - the A2A trace, showing the exact path taken: Router → CustomerDataAgent → Router → SupportAgent → Router → END.