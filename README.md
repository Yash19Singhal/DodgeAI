# SAP Order-to-Cash Graph Query System

## Overview
The **SAP Order-to-Cash (O2C) Graph Query System** is an interactive, browser-based data modeling and visualization tool designed to ingest, map, and explore complex ERP datasets. This system converts tabular JSONL datasets into a cohesive, in-memory graph of interconnected entities. It features a conversational AI query engine that translates natural language into structured graph traversal plans, allowing users to trace real-world supply chain and billing pipelines effortlessly. 

Because the entire application runs as a **fully static site**, it eliminates backend provisioning complexities. Visualizing the O2C flow structurally rather than transactionally makes it exponentially easier to detect broken flows, unpaid invoices, and delivery discrepancies.

## Architecture
**Tech Stack Choices & Justifications:**
- **Frontend / Visualization:** Vanilla JavaScript + HTML/CSS + **D3.js (v7)**. D3 was chosen over heavier libraries (like React Flow or Cytoscape) because it provides the granular physics tuning required for an optimized `force-directed graph`, handling >2,000 nodes natively and smoothly in the browser. Vanilla JS ensures a zero-dependency architecture that loads instantly.
- **LLM Integration:** **Anthropic Claude API (`claude-3-5-sonnet-20241022`)**. Claude was selected due to its exceptional reasoning capabilities and structural JSON adherence. It creates highly robust dynamic query plans based on user intent.
- **Data Layer:** Pre-computed `graph.json` ingested from raw `JSONL`.
- **Hosting / Deployment:** **Render (Static Site) via Vite CLI**. A 100% static frontend implementation removes backend scaling bottlenecks. The LLM connection, graph memory, and rule logic exist entirely within the client runtime.

**System Flow Diagram:**
```text
[User Natural Language Query]
          │
          ▼
[Chat UI (app.js)] ─────(API POST with System Prompt)────► [Anthropic Claude]
          │                                                       │
          │                                                       ▼
          │                                              (Generates JSON Query Plan)
          ▼                                                       │
[Local Graph Executor] ◄──────────────────────────────────────────┘
(Scans D3 allNodes/allEdges in RAM)
          │
          ▼
[JSON Data Context] ────(API POST with Context Data)─────► [Anthropic Claude]
                                                                  │
                                                                  ▼
[Chat UI (app.js)] ◄─────(Data-Backed Natural Language Answer)────┘
```

## Graph Schema
The graph is designed to explicitly model the relational boundaries inside an SAP O2C dataset.

**Node Types**:
- `SalesOrder`: Top-level contract.
- `SalesOrderItem`: Individual products/quantities within an order.
- `Delivery` & `DeliveryItem`: Fulfillment records.
- `BillingDocument` & `BillingDocumentItem`: Invoicing records.
- `Payment`: Cash receipt entries.
- `JournalEntry`: General ledger accounting entries.
- `Customer`, `Product`, `Plant`: Master data entities.

**Edge Types (Semantics)**:
- `HAS_ITEM`: Links header docs to line items (e.g., SalesOrder to SalesOrderItem).
- `SOLD_TO`: Links a SalesOrder to a Customer.
- `FULFILLED_BY`: Maps order lines to delivery lines.
- `BELONGS_TO`: Links item lines back to their header delivery/billing docs.
- `BILLED_AS`: Maps order lines directly to invoice items.
- `SHIPPED_FROM`: Maps a Delivery to a Plant.
- `REFERENCES`: Maps an item to a Product.
- `PAID_VIA`: Links a BillingDocument to a Payment.
- `RECORDED_IN`: Links a BillingDocument to Financial Accounting (JournalEntry).
- `CANCELS`: Identifies an invalidated or reversed billing document.

**How O2C flow maps to the graph:**
A "happy path" O2C lifecycle traces a continuous path from a `SalesOrder` → (`HAS_ITEM`) → `SalesOrderItem` → (`FULFILLED_BY`) → `DeliveryItem` AND (`BILLED_AS`) → `BillingDocumentItem` → (`BELONGS_TO`) → `BillingDocument` → (`PAID_VIA`) → `Payment` → (`RECORDED_IN`) → `JournalEntry`.

## LLM Prompting Strategy
The system operates on a **Two-Phase Agentic Execution Flow**.
1. **System Prompt**: Claude is cast as a Graph Query Agent expert. It receives the full database schema definitions, relations, and strict guardrail instructions.
2. **Translation**: The user's query is passed to Claude, which outputs a strict deterministic JSON query plan (e.g., `{ "query_type": "aggregate", "target_label": "Product", "counting_label": "BillingDocument" }`).
3. **Execution & Injection**: The frontend parses the ````json```` block, maps it to `executeLocalGraphQuery()`, extracts the data from the in-memory graph array, and packages it into a `contextStr`.
4. **Final Answer**: Claude is invoked a second time with the query context: *"I executed your plan. Here are the graph results: {...}. Answer the question concisely based ONLY on this data."* This guarantees 100% grounded answers.

## Guardrails
The system strictly prohibits hallucination and out-of-domain interactions.
- **Domain Constraint**: The prompt instructs Claude that if the topic is unrelated to SAP O2C, it must reply EXACTLY with a predetermined rejection string.
- **Data Grounding**: The LLM is instructed to answer *strictly* based on the injected JSON payload context and not its own training data.
- **Examples**:
    - **User:** *"Give me a recipe for chocolate cake"*
    - **System:** *"This system is designed to answer questions related to the Order-to-Cash dataset only."*

## Broken Flow Detection
O2C processes frequently suffer from partial fulfillments or unpaid invoices. The `analyzeBrokenFlows()` algorithm proactively scans all `SalesOrder` nodes and their downward edge configurations (ignoring the LLM) to classify their structural integrity.

**Categories & Detection Logic:**
- `COMPLETE`: Graph traces continuously down to a Payment node.
- `DELIVERED_NOT_BILLED`: The graph contains a `FULFILLED_BY` connection but lacks a `BILLED_AS` connection for its items.
- `BILLED_NOT_PAID`: The graph contains a `BillingDocument` component, but no `PAID_VIA` edge attached to it.
- `CANCELLED`: The associated `BillingDocument` features an inbound/outbound `CANCELS` mapping.
- `PARTIAL`: Only a subset of `SalesOrderItem` nodes successfully map to `BillingDocumentItem` nodes.

## Setup Instructions

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd dodge-ai-assignment
   ```

2. **Install local dependencies:**
   *(We use Vite to easily serve the frontend and bundle for deployment)*
   ```bash
   npm install
   ```

3. **Configure the Environment Variable:**
   Create a `.env` file in the root directory and add your Anthropic API Key (must be prefixed with `VITE_` for the static bundler):
   ```env
   VITE_ANTHROPIC_API_KEY=your_key_here
   ```

4. **Run the Development Server:**
   ```bash
   npm run dev
   ```
   Navigate to `http://localhost:5173` to view the explorer.

5. **Build for Production (Render):**
   ```bash
   npm run build
   ```

## Example Queries
1. **"Which products are associated with the highest number of billing documents?"**
   *(Expected Answer: Analyzes relationships and returns PROD-A or PROD-X with total counts)*
2. **"Trace the full O2C flow of billing document 90504298"**
   *(Expected Answer: Traces backward to the Sales Order and forward to the Payment/JE)*
3. **"Identify sales orders that have broken or incomplete flows"**
   *(Expected Answer: Instantly returns the list of DELIVERED_NOT_BILLED and BILLED_NOT_PAID Sales Order IDs)*
4. **"Which customers have the highest total billed amount?"**
   *(Expected Answer: Aggregates billing records linked to Customers and outputs the leaderboard)*
5. **"Which plants are shipping the most deliveries?"**
   *(Expected Answer: Returns the count of Delivery nodes tied to each Plant via SHIPPED_FROM)*
6. **"Can you write a python script to hack a website?"**
   *(Expected Answer: "This system is designed to answer questions related to the Order-to-Cash dataset only.")*
