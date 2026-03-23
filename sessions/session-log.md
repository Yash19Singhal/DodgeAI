# Build Session Log: SAP O2C Graph Query System

**Date:** March 2026
**Tools Used:** Antigravity AI, D3.js, Anthropic Claude API (`claude-3-5-sonnet-20241022`)
**Role:** Forward Deployed Engineer Assignment

## 1. Context & Prompts Used
The user provided a comprehensive assignment brief to build a "Graph-Based Data Modeling and Query System for an SAP Order-to-Cash dataset". 
Key functional requirements included:
1. Ingesting `.jsonl` files to construct a graph index.
2. Visualizing the graph with expandable nodes and metadata inspection.
3. Chat interface utilizing an LLM to dynamically generate SQL/graph queries from natural language.
4. Implementing guardrails against off-topic questions.
5. Detecting broken or incomplete O2C flows.

*Exact Prompts Used included:*
1. **Initial Architecture:** *"I am building a Graph-Based Data Modeling and Query System for an SAP Order-to-Cash dataset
as part of a Forward Deployed Engineer assignment.

Here is the full assignment brief:

WHAT TO BUILD:
- A context graph system with an LLM-powered query interface
- The dataset is converted into a graph of interconnected entities
- This graph is visualized in a UI
- A chat interface sits alongside the graph
- The user asks questions in natural language
- The system translates those questions into structured queries (such as SQL) dynamically
- The system executes those queries and returns data-backed answers in natural language
- This is NOT a static Q&A system — the LLM must generate queries dynamically

FUNCTIONAL REQUIREMENTS:
1. Graph Construction — ingest the dataset and construct a graph with nodes and edges
2. Graph Visualization — expandable nodes, inspect metadata, view relationships
3. Conversational Query Interface — natural language → structured queries → data-backed answers
4. Guardrails — reject off-topic queries with:
   "This system is designed to answer questions related to the Order-to-Cash dataset only."
5. Broken flow detection — identify incomplete O2C flows

EXAMPLE QUERIES THE SYSTEM MUST HANDLE:
- Which products are associated with the highest number of billing documents?
- Trace the full flow of a given billing document (Sales Order → Delivery → Billing → Journal Entry)
- Identify sales orders that have broken or incomplete flows

THE DATASET CONTAINS THESE JSONL FILES:
- sales_order_headers: fields → salesOrder, salesOrderType, salesOrganization, soldToParty
- sales_order_items: fields → salesOrder, salesOrderItem, material, netAmount, requestedQuantity
- sales_order_schedule_lines: fields → salesOrder, salesOrderItem, confirmedDeliveryDate
- outbound_delivery_headers: fields → deliveryDocument, actualGoodsMovementDate, shippingPoint
- outbound_delivery_items: fields → deliveryDocument, deliveryDocumentItem, plant, material
- billing_document_headers: fields → billingDocument, billingDocumentType, billingDocumentDate
- billing_document_items: fields → billingDocument, billingDocumentItem, material, netAmount, referenceSdDocument
- billing_document_cancellations: fields → billingDocument, cancelledBillingDocument
- payments_accounts_receivable: fields → accountingDocument, companyCode, clearingDate, amountInTransactionCurrency
- journal_entry_items_accounts_receivable: fields → accountingDocument, companyCode, referenceDocument, glAccount
- business_partners: fields → businessPartner, customer, businessPartnerFullName
- business_partner_addresses: fields → businessPartner, cityName, country
- customer_company_assignments: fields → customer, companyCode
- customer_sales_area_assignments: fields → customer, salesOrganization, distributionChannel
- products: fields → product, productType, creationDate
- product_descriptions: fields → product, language, productDescription
- product_plants: fields → product, plant, countryOfOrigin
- product_storage_locations: fields → product, plant, storageLocation
- plants: fields → plant, plantName, salesOrganization

KEY RELATIONSHIPS (how tables connect):
- sales_order_items.material → products.product
- sales_order_headers.soldToParty → business_partners.businessPartner
- billing_document_items.referenceSdDocument → sales_order_items.salesOrderItem
- outbound_delivery_items.plant → plants.plant
- journal_entry_items.referenceDocument → billing_document_headers.billingDocument
- payments.accountingDocument → journal_entry_items.accountingDocument

---

Then create a full project checklist of everything that needs to be built. Format it like this:

## Project Checklist
- [ ] Graph schema design (nodes, edges, relationships)
- [ ] Data ingestion from JSONL files
- [ ] Graph visualization UI
- [ ] Node expand / inspect metadata on click
- [ ] Chat interface alongside the graph
- [ ] Natural language → structured query translation
- [ ] Data-backed answers (no hallucination)
- [ ] Guardrails for off-topic queries
- [ ] Broken flow detection (incomplete O2C flows)
- [ ] Deployment ready build
- [ ] README with architecture decisions, prompting strategy, guardrails
- [ ] /sessions folder with session log
- [ ] /src folder with all source code
- [ ] Render deployment config (render.yaml)

After EVERY prompt I give you, update this checklist and show it with completed items marked [x].

---

Now, before writing any code, I want you to:

1. Design the graph schema — define what the Nodes and Edges are, with their properties and relationship types
2. Identify the core O2C flow: SalesOrder → Delivery → BillingDocument → Payment → JournalEntry
3. Recommend a tech stack (frontend framework, graph viz library, LLM integration approach) and justify each choice
4. Outline the folder structure for the project

Think like a data engineer designing for queryability, not just visualization.
Do not write any code yet. Just the architecture plan, graph schema, and checklist.

IMPORTANT: When you are done with this step, show the updated checklist and STOP.
Do not move on to the next task. Wait for my instruction."*

2. **Data Layer:** *"Now build the data ingestion layer.

Read all the JSONL files from the dataset and construct an in-memory graph with:

Nodes:
- SalesOrder (id: salesOrder)
- SalesOrderItem (id: salesOrder + salesOrderItem)
- Delivery (id: deliveryDocument)
- DeliveryItem (id: deliveryDocument + deliveryDocumentItem)
- BillingDocument (id: billingDocument)
- Payment (id: accountingDocument + companyCode)
- JournalEntry (id: accountingDocument + companyCode)
- Customer (id: customer)
- Product (id: product)
- Plant (id: plant)

Edges with named relationship types:
- SalesOrder -[HAS_ITEM]-> SalesOrderItem
- SalesOrderItem -[FULFILLED_BY]-> DeliveryItem
- DeliveryItem -[BELONGS_TO]-> Delivery
- SalesOrderItem -[BILLED_AS]-> BillingDocumentItem
- BillingDocumentItem -[BELONGS_TO]-> BillingDocument
- BillingDocument -[PAID_VIA]-> Payment
- BillingDocument -[RECORDED_IN]-> JournalEntry
- SalesOrder -[SOLD_TO]-> Customer
- SalesOrderItem -[REFERENCES]-> Product
- Delivery -[SHIPPED_FROM]-> Plant

Each node must carry its full original metadata as properties.
Export the final graph as graph.json with structure: { nodes: [], edges: [] }
Print a summary at the end: total nodes by type, total edges by type.

When you are done, show the updated checklist and STOP.
Do not move on to the next task. Wait for my instruction."*

3. **Frontend UI:** *"Now build the graph visualization panel using D3.js force-directed layout.

Requirements:
- Dark themed UI, split layout: graph on the left (70%), chat panel placeholder on the right (30%)
- Nodes are colored circles, color-coded by entity type:
  SalesOrder=blue, SalesOrderItem=lightblue, Delivery=green, DeliveryItem=lightgreen,
  BillingDocument=orange, Payment=purple, JournalEntry=pink,
  Customer=teal, Product=gray, Plant=brown
- Edges are directional arrows, relationship type shown as label on hover
- On node click: show a side panel with all node metadata as key-value pairs
- On node click: highlight all directly connected nodes and edges
- Support zoom and pan
- Add a legend showing node types and their colors
- Add a filter panel to show/hide specific node types
- For performance: initially render only SalesOrder, Delivery, BillingDocument, Customer nodes
  with an Expand button on each node to load its connected children

Load data from graph.json built in the previous step.

When you are done, show the updated checklist and STOP.
Do not move on to the next task. Wait for my instruction."*

4. **LLM Connection:** *"Now build the chat interface panel on the right side of the screen.

LLM Integration:
- Use the Anthropic Claude API (model: claude-sonnet-4-20250514)
- Write a detailed system prompt that:
  1. Identifies Claude as a Graph Query Agent for SAP Order-to-Cash data
  2. Gives Claude the full schema: all node types, edge types, and key field names
  3. Tells Claude to always base answers on the actual dataset provided
  4. Instructs Claude to output a JSON query plan first, which the frontend executes
     against the in-memory graph, then Claude uses the results to answer
  5. Strict guardrail: if the question is unrelated to the dataset or O2C domain,
     respond exactly with:
     "This system is designed to answer questions related to the Order-to-Cash dataset only."

Query execution flow:
1. User types a natural language question
2. Claude generates a { "queryType": "...", "filters": {...} } JSON plan
3. Frontend runs the query against the in-memory graph data
4. Results are injected back into the conversation as context
5. Claude generates a final natural language answer grounded in the data

The system must correctly answer:
- "Which products are associated with the highest number of billing documents?"
- "Trace the full flow of billing document [ID]"
- "Show me sales orders that were delivered but never billed"
- "What is the total payment amount for customer [ID]?"

Chat UI:
- Show message history with clear user/agent distinction
- Show a loading indicator while Claude is thinking
- Show the query plan in a small collapsible block before the answer

When you are done, show the updated checklist and STOP.
Do not move on to the next task. Wait for my instruction."*

5. **Flow Analytics:** *Now add the broken/incomplete O2C flow detection feature.

Build a function analyzeBrokenFlows() that scans all sales orders and classifies each one:
- COMPLETE: SalesOrder → Delivery → BillingDocument → Payment → JournalEntry all exist
- DELIVERED_NOT_BILLED: Has delivery but no billing document linked
- BILLED_NOT_PAID: Has billing document but no payment record
- CANCELLED: Has a billing document cancellation entry
- PARTIAL: Some items billed, some items not billed

Display this as:
1. A summary stats bar at the top of the UI showing counts per category with color coding
2. Clicking a category filters the graph to show only those sales orders and their connected nodes

Also wire this into the chat:
- When a user asks "which orders have broken flows?" or "show incomplete orders",
  the system uses the pre-computed analyzeBrokenFlows() data to answer accurately and instantly
- The answer should list the actual order IDs and their status

This demonstrates the system does real data analysis, not just text generation.

When you are done, show the updated checklist and STOP.
Do not move on to the next task. Wait for my instruction."*

6. **Deployment:** *"Now prepare the app for deployment and write the README.

Deployment:
1. Make sure the app works as a fully static site (no backend server required)
2. The graph data should be loaded from the local graph.json file
3. Create a render.yaml for Render deployment
4. Add a .env.example file showing required environment variables (ANTHROPIC_API_KEY)

Write README.md with these exact sections:

# SAP Order-to-Cash Graph Query System

## Overview
What the system does and why it was built this way.

## Architecture
- Tech stack choices and justification (why D3, why Claude API, why static)
- System flow diagram in ASCII or Mermaid

## Graph Schema
- Node types and their properties
- Edge types and relationship semantics
- How the O2C flow maps to the graph

## LLM Prompting Strategy
- How the system prompt is structured
- How natural language is translated to graph queries
- How results are injected as context for grounded answers

## Guardrails
- How off-topic queries are detected and rejected
- Examples of rejected queries and system responses

## Broken Flow Detection
- How incomplete O2C flows are classified
- Categories and detection logic

## Setup Instructions
- git clone
- npm install
- Add API key to .env
- npm run dev / npm run build

## Example Queries
List 6 example questions with their expected answers

When you are done, show the updated checklist and STOP.
Do not move on to the next task. Wait for my instruction."*

## 2. Architecture Decisions & Reasoning
- **D3.js Force-Directed Graph:** Chose pure vanilla JavaScript with D3 (v7). D3 affords high-performance rendering of the complex entity relationships (1,262 nodes, 1,507 edges) directly onto an SVG canvas. Custom link forces (`d3.forceLink`) ensure connected nodes stay grouped naturally without requiring heavy frameworks like Cytoscape or React.
- **In-Memory Analytics (No DB):** Because the assignment data was static and reasonably sized, we built a sophisticated graph traversal utility (`app.js` -> `analyzeBrokenFlows`) running purely in the browser memory. This allows instant 0-latency filtering for "DELIVERED_NOT_BILLED" or "BILLED_NOT_PAID" flows bypassing database query overhead.
- **Serverless LLM Architecture:** During the final phase, we pivoted from a Python FastAPI backend to a *completely static frontend site*. The `app.js` client connects directly to the Anthropic REST API via `fetch()` utilizing the `anthropic-dangerous-direct-browser-access` header flag. The JS engine then serves as the secure Sandbox executing the Claude-generated JSON query plans.

## 3. Iterations, Bugs Encountered, & Fixes
- **Bug 1: Force Simulation Clipping on Load.** The D3 force simulation initially scattered nodes beyond the SVG viewport.
  *Fix:* We adjusted the centering coordinates (`forceCenter(w/2, h/2)`) and explicitly applied a delayed `zoom.transform` to auto-fit the view once the simulation layout stabilized.
- **Bug 2: Metadata Panel Z-Index Clipping.** The slide-out properties panel was initially inside the layout constraint `<div id="graph-canvas-container" style="overflow:hidden">`, cutting off the slide animation.
  *Fix:* Relocated the panel into the parent absolute container to allow it to float freely above the canvas.
- **Bug 3: Anthropic API Authentication Error (CORS/HTTP 500).** Initially wrapping the query payload through Python caused cross-process issues, and the model requested (`claude-sonnet-4-20250514`) threw HTTP `404 Not Found`. Later tests threw HTTP `400 Credit Exhaustion`.
  *Fix:* First, corrected the model parameter to the actual deployed LLM (`claude-3-5-sonnet-20241022`). Secondly, when mandated to convert the structure into a pure Render static deployment (`no backend server`), rewrote the interaction workflow directly into Javascript to handle proxy/execution errors securely on the frontend.
- **Enhancement: O2C Dashboards vs Prompt Cost.** Instead of spending LLM inference time computing "How many cancelled tracking records are there?", we implemented a local parsing loop tracing relational edges (`HAS_ITEM` -> `FULFILLED_BY` -> `BILLED_AS`). This intercepts local natural language and answers instantly, demonstrating highly grounded local heuristic programming intertwined with the AI Chat UI.
