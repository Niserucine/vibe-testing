# Example Vibe Test: No-Code Automation Platform

This is a complete example of a vibe test case. Adapt the structure to your own system.

---

## VT-1: No-Code User Connects Shopify to QuickBooks

### Persona

**Maya** — Owner of a 12-person e-commerce business. Not a developer. Comfortable with tools like Zapier, Make.com, and Shopify admin panels. Has never written code or used a terminal.

### Environment

- **Deployment:** Cloud-hosted SaaS instance
- **Runners:** Platform runner only (control-plane sandbox)
- **Access:** Web browser → gateway API
- **No local runner, no SSH targets, no CLI**

### Goal

> "When a new order comes in on Shopify, automatically create an invoice in QuickBooks, send a confirmation email via SendGrid, and log the order in our Google Sheet."

### Scenario

#### Step 1: Maya signs up and enters the web UI

She opens the app in a browser, authenticates via Google OAuth, and lands in her workspace.

**Primitives:**
- `gateway-api-spec.md`: JWT authentication, workspace creation
- `storage-and-workspace.md`: Workspace initialization

**Questions:**
- Q1.1: Does the gateway spec define a signup/workspace-creation flow, or only authenticated access to existing workspaces?
- Q1.2: Is there a web UI spec? The gateway defines REST/WebSocket/gRPC — is there a frontend that Maya uses?

#### Step 2: Maya describes her automation in plain language

She types her goal in a chat interface. The agent receives the intent and begins decomposition.

**Primitives:**
- `main-agent.md`: Agent receives the intent
- `llm-capability-layer.md`: LLM understanding of the request
- `agent-host.md`: PlanProposer decomposes into a plan

**Questions:**
- Q2.1: How does the agent discover available connectors for Shopify/QuickBooks/SendGrid/Sheets?
- Q2.2: Can the plan DAG express "on trigger X, do A → B → C" as a persistent automation, or only as a one-shot run?
- Q2.3: How does Maya go from "I described it in English" to "it runs forever as a service"?

#### Step 3: Agent installs connectors

The agent determines that four connectors are needed, searches the registry, installs them.

**Primitives:**
- `registry.md`: Search, install
- `connectors.md`: ConnectorDefinition, AuthDefinition
- `policy-spec.md`: Installation may require confirmation

**Questions:**
- Q3.1: Does the install flow handle OAuth2 consent? The connectors spec defines `AuthDefinition::OAuth2` but who initiates the browser redirect for Maya to grant access?
- Q3.2: If Maya needs to paste an API key (e.g., SendGrid), how does the agent prompt her? Is there an interactive credential collection capability?
- Q3.3: The registry validates config against a schema. Who fills in the config — the agent or Maya?

#### Step 4: Agent builds the automation plan

The agent proposes a plan DAG: trigger → invoice → email → log.

**Primitives:**
- `run-execution-semantics.md`: PlanNode, DAG structure
- `capability-schemas.md`: connector invocation I/O
- `connectors.md`: Action references

**Questions:**
- Q5.1: Is this plan a one-shot run or a persistent service? Maya wants it to fire on every new order.
- Q5.2: What happens if the QuickBooks API returns a transient error? Is there retry logic per node?
- Q5.3: Can Maya see the plan before it runs? Is there an approval gate?

#### Step 5: Agent deploys the automation

The session becomes a persistent service.

**Primitives:**
- `crystallization.md`: Session → persistent artifact
- `code-promotion.md`: Build/test/deploy pipeline
- `flow-deployment-model.md`: Supervisor, deployment target

**Questions:**
- Q6.1: Crystallization mines executed traces. But Maya's automation was a proposed plan, not a trace. Does crystallization handle proposed plans?
- Q6.2: The promotion pipeline has 7 steps. For a no-code user, all should be invisible. Is there a shortcut?
- Q6.3: After deployment, how does Maya monitor her automation?

### Spec Coverage Matrix

| Spec Doc | Steps Hit | Coverage |
|----------|-----------|----------|
| `flow-as-durable-actor.md` | 4,5 | Partial — persistent automation concept unclear |
| `capability-schemas.md` | 4 | Connector invocation covered |
| `registry.md` | 3 | Search/install covered |
| `connectors.md` | 3,4 | Core covered; OAuth UX gap |
| `run-execution-semantics.md` | 4 | Plan DAG covered; template reuse unclear |
| `crystallization.md` | 5 | Trace mining covered; plan-first path unclear |
| `code-promotion.md` | 5 | Pipeline covered; no-code shortcut missing |
| `gateway-api-spec.md` | 1 | Auth covered; signup flow gap |
| `storage-and-workspace.md` | 1 | Workspace init covered |
| `cli-spec.md` | — | Not exercised (Maya uses web UI) |
| `memory-module.md` | — | Not exercised |

---

## Expected Output

After simulation, the gap report would look like:

### BLOCKING

| ID | Gap | Affected Steps | Recommended Fix |
|----|-----|---------------|-----------------|
| G-B1 | No web UI spec — Maya needs browser access | 1 | New: `web-ui-spec.md` |
| G-B2 | No persistent automation concept — one-shot runs only | 4, 5 | Add automation template to `flow-as-durable-actor.md` |
| G-B3 | OAuth2 consent flow has no owner | 3 | Add browser redirect handling to `connectors.md` |

### DEGRADED

| ID | Gap | Affected Steps | Workaround |
|----|-----|---------------|-----------|
| G-D1 | No one-click deploy for no-code users | 5 | Use full 7-step pipeline with agent automation |
| G-D2 | No monitoring dashboard spec | 5 | Query events via API manually |

### COSMETIC

| ID | Gap | Affected Steps |
|----|-----|---------------|
| G-C1 | No automation template gallery | 2 |
