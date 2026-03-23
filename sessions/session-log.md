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
1. **Initial Architecture:** *"I am building a Graph-Based Data Modeling and Query System for an SAP Order-to-Cash dataset as part of a Forward Deployed Engineer assignment... A context graph system with an LLM-powered query interface. The dataset is converted into a graph of interconnected entities. This graph is visualized in a UI..."*
2. **Data Layer:** *"Now build the data ingestion layer. Read all the JSONL files from the dataset and construct an in-memory graph with [Nodes: SalesOrder, Delivery...] [Edges: HAS_ITEM, FULFILLED_BY...] Save this graph to a JSON file (data/graph.json)."*
3. **Frontend UI:** *"Now build the graph visualization panel using D3.js force-directed layout. Requirements: Dark themed UI, split layout (graph left 70%, chat 30%). Nodes colored by entity type. On node click: show side panel with metadata. On click highlight connected nodes. Support zoom/pan, legend, filter panel."*
4. **LLM Connection:** *"LLM Integration: Use the Anthropic Claude API. Write a detailed system prompt that gives Claude the full schema. Tell Claude to output a JSON query plan first, which frontend executes against in-memory graph, then Claude uses results to answer. Strict guardrail for off-topic."*
5. **Flow Analytics:** *"Now add the broken/incomplete O2C flow detection feature. Build a function analyzeBrokenFlows() that scans all sales orders and classifies each one: COMPLETE, DELIVERED_NOT_BILLED, BILLED_NOT_PAID, CANCELLED, PARTIAL. Display summary stats bar. Clicking a category filters the graph. Wire this into the chat for instant accurate answers."*
6. **Deployment:** *"Now prepare the app for deployment and write the README. Make sure the app works as a fully static site (no backend server required). Create a render.yaml for Render deployment. Write README.md with exact sections [Overview, Architecture, Schema, Guardrails, Broken Flow, Setup, Example Queries]."*

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
