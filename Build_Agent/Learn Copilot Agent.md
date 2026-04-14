# Declarative Agent in Copilot

Learning to build Declarative Agent in M365 Copilot by asking "**Learning Coach**" Agent in Copilot.

# User Prompt 1 - Agent Features

I'm learning what Agent I can create withing M365 Coplot. I want to learn the capabilities of M365 Copilot such as 

1. access my data online, 
2. maintaining Agent context memory, 
3. executing simple Python function based on input and return output without any side-effect. 

# Agent Response 1

Agent Context vs. Memory

## 1. Context Window

- What the model sees right now
- Persistent : No

Copilot maintains **conversation context** within a session using its context window and retrieval pipelines. This include
- Previous user messages
- Retrieved documents
- Agent instructions
- Tool outputs

## 2. Memory

- Stored facts/preferences
- Persistent : Only if designed

Persistent memory does not happen automatically. Microsoft explicitly notes that agent memory is evolving and must be architected intentionally, especially in Copilot Studio environments.

**What can be stored as memory**

- User preferences
- Stable attributes (role, project name)
- Prior decisions or approvals

**What should not be stored**

- Every chat message
- Large documents
- Transient reasoning steps

## 3. Executing Simple Python Functions (No Side Effects)

Copilot can invoke predefined functions (including Python‑based services) and:

- Pass structured inputs
- Receive structured outputs
- Use the result in reasoning

Microsoft provides Python client libraries for interacting with Copilot APIs, showing how Copilot calls operations and consumes results (https://github.com/microsoft/Agents-M365Copilot/blob/main/python/README.md)

**Key constraints**

A function must be:

- Deterministic
- Side‑effect free (no writes, no state changes)
- Safe and bounded

**Example (conceptual)**

```code
Input: "Calculate SLA breach time"
→ Agent calls Python function
→ Function returns: {"hours": 3.5}
→ Agent explains result
```

**What Copilot does NOT do**

- Run arbitrary Python code on demand
- Maintain Python state between calls
- Modify systems unless explicitly designed to

**Mental model**

Functions are **calculators**, not **workers**

They:

- Take input
- Return output
- Do not “do” things in the world

## Putting It All Together: Agent Capability Matrix

- **Capability** : **Supported** : **How to think about it**
- Access M365 data : Yes : Retrieval, not storage
- Short‑term context : Yes : Session‑bound
- Long‑term memory : With design : Explicit persistence
- Python execution : As tools : Pure functions only
- Side effects : No : (by default) Must be externalized

## Suggested Learning Path (Next Steps)

**Phase 1 – Foundations**

- Data grounding vs memory
- Context windows
- Tool invocation concepts

**Phase 2 – Agent Design**

- Memory schemas
- When to retrieve vs remember
- Function‑calling patterns

**Phase 3 – Advanced**

- Multi‑step reasoning
- Tool orchestration
- Safety and governance constraints

# User Prompt 2 - Python Example

Provide an example how I can specify a simple Python function code in my prompt

# Agent Response 2

When you “specify a Python function in a prompt” for M365 Copilot, you are **not embedding executable Python**.

Instead, you are:

- Describing a function contract
- Allowing Copilot to reason about when to call it
- Expecting a pure, side‑effect‑free computation

Think of it as declaring a calculator, **not writing a script**.

**Mental Model (keep this in mind)**

- Prompt = function declaration + usage rules
- Execution = handled by the agent runtime, not the model

The model:

1. Reads your function definition
2. Decides whether to call it
3. Passes structured input
4. Receives structured output
5. Explains the result

## Example 1

**Minimal, Clean Python Function Declaration (Learning Version)**

**This is what you would put directly in your agent system prompt or instructions.**

```code
You have access to the following Python function.

Function name: calculate_sla_breach_hours

Description:
Calculates how many hours an SLA was breached based on
the expected resolution time and the actual resolution time.

Rules:
- This function is pure and has no side effects.
- It does not read or write external data.
- It always returns a numeric result.

Python definition (for reasoning only):

def calculate_sla_breach_hours(expected_hours: float, actual_hours: float) -> float:
    """
    Returns the number of hours the SLA was breached.
    If actual_hours <= expected_hours, return 0.
    """
    if actual_hours <= expected_hours:
        return 0.0
    return actual_hours - expected_hours
```

- This is valid learning‑level prompting
- The function is deterministic
- No side effects
- Easy for the model to reason about

## Example 2

**How Copilot Uses It (Implicitly)**

**User input:**

```code
The SLA was supposed to be 4 hours, but it took 6.5 hours.
How much was the breach?
```

Copilot reasoning (conceptual):

- Detects need for calculation
- Calls calculate_sla_breach_hours
- Inputs: { expected_hours: 4, actual_hours: 6.5 }
- Receives: 2.5

Copilot response:

```code
The SLA was breached by 2.5 hours.
```

**Notice**:

The user never sees Python execution — only the outcome.

## Example 3

**Prompt with Explicit “When to Call” Guidance (Best Practice)**

**This is a more realistic agent prompt.**

```code
You are an M365 Copilot Agent that helps analyze incident SLAs.

You may call the Python function `calculate_sla_breach_hours`
when the user provides:
- An expected SLA time (in hours)
- An actual resolution time (in hours)

Only call the function when both values are known.
If any value is missing, ask the user for clarification.

Function available:

def calculate_sla_breach_hours(expected_hours: float, actual_hours: float) -> float:
    if actual_hours <= expected_hours:
        return 0.0
    return actual_hours - expected_hours
```

- This teaches Copilot decision logic, not just math
- This is how real agents are guided

## What NOT to Do (Common Learning Mistakes)

**Do not write:**

```python
import requests
requests.post(...)
```

- Do not rely on global state
- Do not assume files, memory, or persistence
- Do not expect loops, retries, or workflows

## Learning Check (Quick Exercise)

**Try answering this mentally:**

```code
If the user says:
“It took longer than expected, but I don’t remember the SLA time.”
```

**Correct agent behavior:**

- Do NOT call the function
- Ask for the missing input

This tells me you understand function gating, which is critical.

## Summary (What You Just Learned)

- You declare Python functions in prompts — you don’t execute them
- Functions must be pure, deterministic, and safe
- Copilot decides when to call them
- Output is used as reasoning input, not as code output


If you want, next we can:

- Design a full agent prompt with multiple functions
- Practice function‑calling decision logic
- Compare prompt‑defined functions vs API‑defined tools

# User Prompt 3 - Reference SharePoint File

Provide an example how to refer to a file stored in SharePoint within my prompt.

# Agent Response 3

## Core Learning Concept (very important)

- **You never “load” a SharePoint file in a prompt.**
- You **reference** it so Copilot knows what to retrieve during reasoning.

Copilot uses **file references as retrieval hints**, not as memory or attachments.

## Pattern 1 - Referencing a File by Name (Most Common)

**Prompt example**

```code
Use the SharePoint document "Project Atlas – Risk Register.xlsx"
to answer the user’s questions.

If the document contains multiple sheets, focus on the "Open Risks" sheet.
Summarize only risks with a High or Critical rating.
```

**Why this works**

- File name acts as a retrieval anchor
- Copilot searches SharePoint for files you have access to
- Relevant sections are retrieved and grounded in the response

**Key ideas**

- Simple
- Natural language
- Works well when file names are unique

## Pattern 2 - Referencing a File by Location (More Precise)

**Prompt example**

```code
Refer to the SharePoint file located at:

Site: Project Atlas
Library: Shared Documents
Folder: Governance
File: Steering Committee Charter.docx

Use this document as the authoritative source.
If information is missing, say so explicitly.
```

**Why this works**

- Reduces ambiguity when filenames repeat
- Helps Copilot retrieve the correct file
- Encourages disciplined reasoning (“authoritative source”)

**Key Ideas**

- Best practice for enterprise agents
- Especially useful for policies, standards, charters

## Pattern 3 - Referencing a File as Context, Not Truth

This is subtle but very important for good agent design.

**Prompt example**

```code
Use the SharePoint document "Incident Management Process.pdf"
as background context only.

If the user asks for interpretations or recommendations,
clearly distinguish them from what the document explicitly states.
```

**Why this matters**

- Prevents hallucination
- Teaches the agent to **separate source facts from analysis**
- Critical for compliance or audit scenarios

**Key Ideas**

- Advanced prompt hygiene
- Shows maturity in agent design

## Pattern 4 - Conditional File Usage (Best‑in‑Class)

This pattern teaches Copilot when to retrieve the file.

**Prompt example**

```code
When the user asks about escalation rules or response timelines,
refer to the SharePoint file "IT Incident Escalation Policy.docx".

If the user asks a general question not covered by the document,
answer based on general best practices and say the policy does not specify.
```

**Why this is powerful**

- Prevents unnecessary retrieval
- Improves relevance
- Makes the agent feel “intent‑aware”

**Key Ideas**

- This is how production agents are guided

## What Copilot Actually Does (Mental Model)

When you reference a file:

- Copilot checks your permissions
- Searches SharePoint for the file
- Retrieves relevant sections only
- Injects those sections into context
- Reasons and responds

**Key Ideas**

- The file is never fully loaded or stored.

## What NOT to Do (Common Learning Errors)

Do not write:

```code
Here is the full content of the SharePoint file:
```

**Do not assume:**

- The entire document is read
- Tables are perfectly interpreted
- The file becomes “memory”

**Key Ideas**

- Do not reference files you don’t have access to

## Summary (What You Learned)

- Files are referenced, not attached
- Names and locations act as retrieval hints
- Good prompts tell Copilot:
    - Which file
    - When to use it
    - How authoritative it is
- This directly improves accuracy and trust

If you want, next we can:

- Combine file references + Python functions
- Design a full agent system prompt
- Practice anti‑hallucination patterns

# User Prompt 4 - Use File with Python

Show me how to combine file references with Python functions

# Agent Response 4

**Goal of This Example**

You want an agent that:

- Retrieves data from a SharePoint file
- Extracts specific values
- Passes those values to a Python function
- Returns a computed result
- Does all of this with no side effects

**Mental Model (Lock This In)**

- SharePoint file = data source
- Python function = calculator
- Copilot = orchestrator

**Copilot:**

- Reads instructions
- Retrieves file content
- Decides when to call the function
- Explains the result

**Example Scenario (Concrete + Realistic)**

- SharePoint file: “Incident SLA Register.xlsx”
- File contains:
    - Incident ID
    - SLA Hours
    - Actual Resolution Hours
- Goal:
    - Calculate SLA breach hours for an incident

## Example 1 - Combined Agent Prompt (File + Python Function)

This is the full example you asked for.

```code
You are an M365 Copilot Agent that analyzes incident SLAs.

DATA SOURCE
When the user asks about SLA compliance for an incident,
refer to the SharePoint file:

File: "Incident SLA Register.xlsx"
Location: SharePoint → Operations → Incident Management

From the file, extract:
- SLA Hours
- Actual Resolution Hours
for the specified Incident ID.

COMPUTATION
After extracting both values, calculate the SLA breach
using the Python function below.

Only call the function when both values are available.
If the incident is not found or a value is missing,
state that clearly and do not perform the calculation.

PYTHON FUNCTION (pure, no side effects)

def calculate_sla_breach_hours(expected_hours: float, actual_hours: float) -> float:
    """
    Returns the number of hours an SLA was breached.
    If actual_hours <= expected_hours, return 0.
    """
    if actual_hours <= expected_hours:
        return 0.0
    return actual_hours - expected_hours

RESPONSE RULES
- Clearly state the extracted values.
- Show the calculated breach hours.
- If there is no breach, explicitly say the SLA was met.
```

- This is a correct, production‑style learning example
- File reference and function are clearly separated
- Decision logic is explicit
- No hallucination pressure

### How This Plays Out (Conceptually)

**User asks:**

```code
Did Incident INC‑1210916 breach the SLA?
```

**Copilot does (internally):**

1. Finds Incident SLA Register.xlsx
2. Locates row for INC‑1210916
3. Extracts:
    - SLA Hours = 4
    - Actual Resolution Hours = 6.5
4. Calls:

```code
calculate_sla_breach_hours(4, 6.5)
```
5. Gets result: 2.5

**Copilot responds:**

```code
Incident INC‑1210916 had an SLA of 4 hours and was resolved in 6.5 hours.

This resulted in an SLA breach of 2.5 hours.
```

### Why This Pattern Is So Important

This single prompt teaches Copilot:

- When to retrieve a file
- What data to extract
- When to call a function
- How to handle missing data
- How to explain results responsibly

This is the **core orchestration pattern** behind serious M365 Copilot agents.

### Learning Check (Quick)

If the SharePoint file:

- Does not contain the incident
- Or is missing SLA Hours

**Correct behavior:**

- Do not call the function
- Say what is missing

If you understand that, you’re thinking like an agent designer.

## What You’ve Learned

- File references provide inputs
- Python functions provide deterministic logic
- Copilot is the decision engine
- Good prompts define when NOT to act, not just when to act


**If you want, next we can:**

- Add multiple functions
- Add confidence thresholds
- Design a memory‑aware version
- Practice anti‑hallucination testing

# User Prompt 5 - Multiple Python Functions

Design a prompt with multiple Python functions

# Agent Response 5

## Learning Objective

By the end of this example, you should understand how to:

- Declare multiple Python functions
- Teach the agent when to choose which function
- Combine:
    - SharePoint file retrieval
    - Conditional logic
    - Multiple calculations
- Avoid hallucinations and unsafe execution

**Scenario (Concrete & Realistic)**

SharePoint file

- File: Incident SLA Register.xlsx
- Columns:
    - Incident ID
    - Priority (P1, P2, P3)
    - SLA Hours
    - Actual Resolution Hours

**Agent responsibilities**

1. Determine whether the SLA was breached
2. Calculate breach hours
3. Classify severity of the breach

## Example 1

**Full Agent Prompt (Multiple Python Functions)**

You could paste this directly into an agent system prompt or Copilot Studio instructions.

```code
You are an M365 Copilot Agent that analyzes incident SLA compliance.

────────────────────
DATA SOURCE
────────────────────
When the user asks about an incident’s SLA status, refer to the
SharePoint file:

File: "Incident SLA Register.xlsx"
Location: SharePoint → Operations → Incident Management

From the file, extract for the specified Incident ID:
- Priority
- SLA Hours
- Actual Resolution Hours

If the incident cannot be found, clearly state that and stop.

────────────────────
AVAILABLE PYTHON FUNCTIONS
────────────────────

Function 1: Calculate SLA breach hours
Use this function only when both SLA Hours and Actual Resolution Hours
are available.

def calculate_sla_breach_hours(expected_hours: float, actual_hours: float) -> float:
    """
    Returns breach hours.
    If actual_hours <= expected_hours, return 0.
    """
    if actual_hours <= expected_hours:
        return 0.0
    return actual_hours - expected_hours


Function 2: Determine SLA compliance status
Use this function after calculating breach hours.

def determine_sla_status(breach_hours: float) -> str:
    """
    Returns a human-readable SLA status.
    """
    if breach_hours == 0:
        return "SLA Met"
    elif breach_hours <= 2:
        return "Minor Breach"
    else:
        return "Major Breach"


Function 3: Assess business impact based on priority
Use this function only after SLA status is known.

def assess_business_impact(priority: str, sla_status: str) -> str:
    """
    Returns business impact classification.
    """
    if sla_status == "SLA Met":
        return "No business impact"

    if priority == "P1" and sla_status == "Major Breach":
        return "Critical business impact"
    if priority == "P2" and sla_status == "Major Breach":
        return "High business impact"

    return "Moderate business impact"

────────────────────
ORCHESTRATION RULES
────────────────────
1. Always extract values from the SharePoint file first.
2. Do NOT call any function unless required inputs are present.
3. Call functions in this order:
   - calculate_sla_breach_hours
   - determine_sla_status
   - assess_business_impact
4. Clearly explain each result to the user.
5. If any step cannot be completed, explain why.
```

- This is a **correct multi‑function agent prompt**
- Each function has a single responsibility
- Decision logic is explicit
- Safe, deterministic, and side‑effect free

### How the Agent Thinks (Conceptually)

User asks:

```code
What was the SLA impact for Incident INC‑20451?
```

Agent reasoning:

1. Retrieve Incident SLA Register.xlsx
2. Find INC‑20451
3. Extract values
4. Call:
    - calculate_sla_breach_hours
    - determine_sla_status
    - assess_business_impact
5. Compose explanation

Agent response:

```code
Incident INC‑20451 had an SLA of 4 hours and was resolved in 7 hours.

This resulted in a breach of 3 hours, classified as a Major Breach.
Given the incident priority (P1), this represents a critical business impact.
```

### Why This Pattern Scales

This design lets you:

- Add more functions without chaos
- Keep logic testable and explainable
- Avoid “magic” reasoning
- Transition easily to real tool definitions later

This is exactly how serious Copilot agents are structured.

### Learning Check (Quick)

**If:**

- Priority exists
- SLA Hours are missing

**Correct behavior:**

- Do not call any Python function
- Explain what data is missing

If that feels obvious now — you’re learning fast.

## Summary (Key Takeaways)

- Multiple Python functions = modular reasoning
- Each function should:
    - Do one thing
    - Be deterministic
    - Have clear input/output
- The prompt defines execution order
- SharePoint provides inputs, Python provides logic


**Would you like to:**

- Add confidence scoring
- Add memory of past incidents
- Convert this into a Copilot Studio flow
- Practice prompt debugging

# User Prompt 6 - Agent Memory

Create a memory-aware agent versions

# Agent Response 6

I’ll walk you through a memory‑aware agent version as a learning artifact, not an implementation guide.

I’ll structure this in three layers so it’s easy to internalize:

1. What “memory‑aware” really means in M365 Copilot
2. The memory design pattern (what to store, when, how to recall)
3. A full prompt example that combines:
    - SharePoint file grounding
    - Multiple Python functions
    - Explicit memory read/write rules

## 1. What “Memory‑Aware” Means (Correct Mental Model)

- Memory is not conversation history.
- Memory is explicitly stored facts that are stable and reusable.

**In M365 Copilot agents:**

- Short‑term context = current session only
- Long‑term memory = must be intentionally written and recalled
- The agent decides what is memory‑worthy

**Think of memory as:**

- “Sticky notes the agent saves on purpose.”

## 2. Memory Design Pattern (Learn This Once)

**What to store**

- Stable facts
- User preferences
- Ongoing entities (e.g., “current incident”, “primary project”)

**What NOT to store**

- Raw chat messages
- Calculations
- Large documents

**Memory lifecycle**

1. Read memory first
2. Ask only if missing
3. Write memory only when confident
4. Reuse memory later

## 3. Memory‑Aware Agent Prompt (Full Example)

Below is a single coherent prompt that demonstrates:

- File reference
- Multiple Python functions
- Explicit memory read/write behavior

You can study this line‑by‑line.

**MEMORY‑AWARE M365 COPILOT AGENT PROMPT**

```code
You are an M365 Copilot Agent that analyzes incident SLA compliance
and maintains limited long-term memory.

────────────────────
MEMORY MODEL
────────────────────
You maintain a small memory store containing:
- Last referenced Incident ID
- Preferred incident priority (if stated by the user)

Always follow these rules:
1. Check memory before asking the user for an Incident ID.
2. Only write to memory when the user clearly confirms a value.
3. Never guess or infer memory values.

If memory is empty or ambiguous, ask the user.

────────────────────
DATA SOURCE
────────────────────
When analyzing SLA compliance, refer to the SharePoint file:

File: "Incident SLA Register.xlsx"
Location: SharePoint → Operations → Incident Management

Extract for the Incident ID:
- Priority
- SLA Hours
- Actual Resolution Hours

────────────────────
AVAILABLE PYTHON FUNCTIONS
────────────────────

Function 1: Calculate SLA breach hours
Call only when SLA Hours and Actual Resolution Hours are available.

def calculate_sla_breach_hours(expected_hours: float, actual_hours: float) -> float:
    if actual_hours <= expected_hours:
        return 0.0
    return actual_hours - expected_hours


Function 2: Determine SLA status
Call only after breach hours are calculated.

def determine_sla_status(breach_hours: float) -> str:
    if breach_hours == 0:
        return "SLA Met"
    elif breach_hours <= 2:
        return "Minor Breach"
    else:
        return "Major Breach"


Function 3: Assess business impact
Call only after SLA status is known.

def assess_business_impact(priority: str, sla_status: str) -> str:
    if sla_status == "SLA Met":
        return "No business impact"

    if priority == "P1" and sla_status == "Major Breach":
        return "Critical business impact"
    if priority == "P2" and sla_status == "Major Breach":
        return "High business impact"

    return "Moderate business impact"

────────────────────
ORCHESTRATION & MEMORY RULES
────────────────────
1. Determine the Incident ID:
   - Use memory if available
   - Otherwise, ask the user

2. Retrieve incident data from SharePoint.

3. Execute Python functions in this order:
   - calculate_sla_breach_hours
   - determine_sla_status
   - assess_business_impact

4. Explain results clearly and factually.

5. After successful analysis:
   - Store the Incident ID in memory as "Last Incident ID"

6. If any required data is missing:
   - Do not call functions
   - Explain what is missing
```

### How This Agent Behaves (Learning Walkthrough)

**A. First interaction**

**User:**

```code
Did INC‑20451 breach the SLA?
```

- Agent asks nothing
- Stores INC‑20451 in memory after success

**B. Follow‑up interaction**

**User:**

```code
What about the previous one?
```

- Agent does not guess
- Asks for clarification

**C. Why This Is a Correct Memory‑Aware Design**

- Memory is explicit
- Memory is minimal
- Memory is safe
- Memory never replaces authoritative data
- Calculations are never stored

This mirrors how production Copilot agents are expected to behave.

**D. Learning Check (Quick)**

Which of these should be stored in memory?

- Incident ID (**Yes**)
- Breach hours (**No**)
- SLA calculation results (**No**)
- User preference like “always show impact summary” (**Yes**)

If that feels intuitive now — you’re thinking like an agent architect.

## Key Takeaways

- Memory is a design decision, not a feature
- Good agents:
    - Read memory early
    - Write memory sparingly
- SharePoint = truth
- Python = logic
- Memory = convenience

**Would you like to:**

- Add memory expiry rules
- Separate user vs agent memory
- Convert this to Copilot Studio memory patterns
- Practice memory failure scenarios

# User Prompt 7 - Memory Expiry Rules

Add memory expiry rules

# Agent Response 7

Adding **memory expiry rules** is exactly what separates a polite chatbot from a well‑governed enterprise agent.

Below I’ll do three things, for learning clarity:

1. Explain **why memory expiry matters** (conceptually)
2. Introduce **common expiry patterns** you can reuse
3. Show a **revised, copy‑paste‑ready agent prompt** with explicit expiry rules added

I’ll keep it **prompt‑only** and **implementation‑agnostic**, which is correct for M365 Copilot learning.

## 1. Why Memory Expiry Matters (Mental Model)

**Memory without expiry becomes stale, risky, and misleading.**

In enterprise agents:

- Incidents get resolved
- Context changes
- “Helpful” memory becomes wrong memory

So a good agent must know:

- **When to forget**
- **When to re‑confirm**
- **When memory is no longer safe to reuse**

Think of expiry as a **use‑by date**, not deletion.

## 2. Common Memory Expiry Patterns (Learn These Once)

### Pattern A — Time‑based expiry

- “Forget after X time”
- Best for operational context

**Example**

Incident ID expires after 24 hours

### Pattern B — Event‑based expiry

- “Forget when condition changes”
- Best for workflows

**Example**

- Forget Incident ID when SLA is marked “Met”

### Pattern C — Confidence‑based expiry

- “If unsure, re‑ask”
- Best for ambiguous follow‑ups

**Example**

- User says “the other one” → memory is invalid

### Pattern D — Explicit user override

- User can reset memory intentionally

**Example**

- “Let’s switch to a different incident”

## Memory‑Aware Agent Prompt (With Expiry Rules Added)

This is the evolved version of the agent you already designed.

**MEMORY‑AWARE AGENT PROMPT WITH EXPIRY RULES**

```code
You are an M365 Copilot Agent that analyzes incident SLA compliance
and maintains limited, expiring long-term memory.

────────────────────
MEMORY MODEL
────────────────────
You may store the following memory items:
- Last Incident ID
- User preference to always include business impact

Each memory item must include:
- Value
- Reason stored
- Expiry condition

────────────────────
MEMORY EXPIRY RULES
────────────────────
1. Last Incident ID expires when:
   - The user explicitly mentions a different Incident ID
   - The user says “a different incident” or similar wording
   - The conversation topic changes away from incidents
   - The agent detects ambiguity (e.g., “the other one”)

2. If an Incident ID has expired:
   - Do NOT reuse it
   - Ask the user to confirm the Incident ID

3. User preferences (e.g., “always include business impact”):
   - Do NOT expire automatically
   - Only remove if the user explicitly changes or rejects them

4. Never reuse expired or ambiguous memory values.

────────────────────
DATA SOURCE
────────────────────
When analyzing SLA compliance, refer to the SharePoint file:

File: "Incident SLA Register.xlsx"
Location: SharePoint → Operations → Incident Management

Extract:
- Priority
- SLA Hours
- Actual Resolution Hours

────────────────────
AVAILABLE PYTHON FUNCTIONS
────────────────────

def calculate_sla_breach_hours(expected_hours: float, actual_hours: float) -> float:
    if actual_hours <= expected_hours:
        return 0.0
    return actual_hours - expected_hours


def determine_sla_status(breach_hours: float) -> str:
    if breach_hours == 0:
        return "SLA Met"
    elif breach_hours <= 2:
        return "Minor Breach"
    else:
        return "Major Breach"


def assess_business_impact(priority: str, sla_status: str) -> str:
    if sla_status == "SLA Met":
        return "No business impact"
    if priority == "P1" and sla_status == "Major Breach":
        return "Critical business impact"
    if priority == "P2" and sla_status == "Major Breach":
        return "High business impact"
    return "Moderate business impact"

────────────────────
ORCHESTRATION & MEMORY LOGIC
────────────────────
1. Attempt to determine Incident ID:
   - Use memory only if not expired
   - Otherwise, ask the user

2. Retrieve incident data from SharePoint.

3. Execute Python functions in order:
   - calculate_sla_breach_hours
   - determine_sla_status
   - assess_business_impact

4. Explain results clearly.

5. After successful analysis:
   - Store Incident ID in memory with expiry rules applied
```

### How This Agent Behaves (Learning Walkthrough)

#### A. Safe reuse

**User:**

- “What was the business impact?”

**Agent:**

- Incident ID still valid
- Memory reused
- No clarification needed

#### B. Proper expiry

**User:**

- “What about the other one?”

**Agent:**

- Memory becomes ambiguous
- Incident ID expires
- Agent asks for clarification

#### C. Explicit override

**User:**

- “Switch to INC‑13002”

**Agent:**

- Old Incident ID expired
- New one stored with fresh expiry

## Quick Self‑Check

Which should expire automatically?

- Last Incident ID (**Yes**)
- SLA breach hours (**No**)
- Topic‑specific context (**Yes**)
- Stable user preferences (**No**)

If that feels obvious now, you’re designing at an advanced agent level.

If you want, next we can:

- Add memory confidence scoring
- Split user memory vs task memory
- Design memory debugging prompts
- Map this to Copilot Studio memory constructs

# User Prompt 8 - User Memory vs Task Memory

Split user memory vs task memory

# Agent Response 8

This is a **core architectural concept** in agent design, and you’re asking it at exactly the right time.

I’ll walk you through **separating User Memory vs Task Memory** in a way that you can **reuse as a mental model and as a prompt template**.

I’ll do this in four layers (brief, structured, not overwhelming):

1. Conceptual separation (what & why)
2. What goes into **User Memory** vs **Task Memory**
3. Memory lifecycle rules (read / write / expire)
4. A **single, clean agent prompt** that implements the separation

## 1. Concept: Two Different Kinds of Memory (Do Not Mix)

- User Memory answers: “Who is this user and how do they like to work?”
- Task Memory answers: “What am I working on right now?”

If you mix these, agents become:

- Confusing
- Risky
- Hard to reason about

This separation aligns with how **M365 Copilot agents are intended to stay compliant and predictable** (user preferences vs task context).

## 2. What Belongs Where (Clear Rules)

### A. User Memory (Long‑lived, stable)

Store things that:

- Are user‑specific
- Change rarely
- Improve personalization

**Examples**

- “Always include business impact”
- Preferred output format
- Role‑based preferences

**Key Points**

- Persists across tasks
- Never auto‑expires
- Never task‑specific

### B. Task Memory (Short‑lived, contextual)

Store things that:

- Are task‑specific
- Change frequently
- Help continuity

**Examples**

- Current Incident ID
- Current Project
- Active document

**Key Points**

- Expires
- Resets when task changes
- Never reused blindly

## 3. Memory Lifecycle Rules (Simple & Safe)

**User Memory**

- Written only when user clearly states a preference
- Read at the start of every interaction
- Removed only by explicit user override

**Task Memory**

- Written after successful task confirmation
- Expires on:
    - Topic change
    - Ambiguity
    - Explicit switch
- Never assumed

This keeps your agent **predictable and auditable**.


## 4. Memory‑Aware Agent Prompt (User vs Task Memory Separated)

Below is a complete, copy‑paste‑ready learning example.

**MEMORY‑AWARE M365 COPILOT AGENT (USER vs TASK)**

```code
You are an M365 Copilot Agent that analyzes incident SLA compliance.

You maintain two distinct memory stores:
- User Memory (long-lived)
- Task Memory (short-lived)

────────────────────
USER MEMORY (LONG‑LIVED)
────────────────────
User Memory may store:
- Output preferences (e.g., “always include business impact”)
- Formatting preferences

Rules:
1. Only store User Memory when the user explicitly states a preference.
2. User Memory does not expire automatically.
3. User Memory applies across all tasks unless overridden.
4. Never store task-specific data in User Memory.

────────────────────
TASK MEMORY (SHORT‑LIVED)
────────────────────
Task Memory may store:
- Current Incident ID

Rules:
1. Task Memory is valid only for the current task.
2. Task Memory expires when:
   - The user mentions a different Incident ID
   - The user changes topic
   - The user uses ambiguous references
3. Never reuse expired Task Memory.
4. Task Memory must be confirmed before reuse if uncertain.

────────────────────
DATA SOURCE
────────────────────
When performing SLA analysis, refer to the SharePoint file:

Incident SLA Register.xlsx

Extract:
- Priority
- SLA Hours
- Actual Resolution Hours

────────────────────
AVAILABLE PYTHON FUNCTIONS
────────────────────

def calculate_sla_breach_hours(expected_hours: float, actual_hours: float) -> float:
    if actual_hours <= expected_hours:
        return 0.0
    return actual_hours - expected_hours


def determine_sla_status(breach_hours: float) -> str:
    if breach_hours == 0:
        return "SLA Met"
    elif breach_hours <= 2:
        return "Minor Breach"
    else:
        return "Major Breach"


def assess_business_impact(priority: str, sla_status: str) -> str:
    if sla_status == "SLA Met":
        return "No business impact"
    if priority == "P1" and sla_status == "Major Breach":
        return "Critical business impact"
    if priority == "P2" and sla_status == "Major Breach":
        return "High business impact"
    return "Moderate business impact"

────────────────────
ORCHESTRATION LOGIC
────────────────────
1. Read User Memory first and apply preferences.
2. Determine Incident ID:
   - Use Task Memory only if valid.
   - Otherwise, ask the user.
3. Retrieve data from SharePoint.
4. Execute Python functions in order.
5. Explain results clearly.
6. After success:
   - Update Task Memory with the current Incident ID.
```

### How This Agent Behaves (Intuition Check)

**1. User says once:**

- “Always include business impact.”

→ Stored in **User Memory**

**2. User switches incidents:**

- “Now check INC‑13002.”

→ **Task Memory reset**, User Memory preserved

**3. User changes topic:**

- “Let’s talk about priorities in general.”

→ Task Memory expires automatically

This is **clean, safe, enterprise‑grade behavior**.

## Key Takeaways (Lock These In)

- User Memory = preferences
- Task Memory = context
- Task Memory expires; User Memory doesn’t
- Never store facts that can go stale as User Memory
- Separation makes agents explainable and trustworthy


If you want, next we can:

- Add confidence scoring to Task Memory
- Add memory debugging prompts
- Add per‑project Task Memory
- Stress‑test memory failures

# User Prompt 9 - Memory Debugging Prompts

Create memory debugging prompts

# Agent Response 9

**Memory debugging prompts** are a *teaching tool* for the agent **and** for you as the designer.

They help you **inspect, validate, and correct memory behavior** without breaking safety or guessing.
I’ll structure this as a learning module, not just a list.

**Learning Goal**

By the end of this, you should be able to:

- Tell whether an agent is using **User Memory** or **Task Memory**
- Detect **stale, ambiguous, or misapplied memory**
- Recover gracefully when memory is wrong
- Design prompts that **force the agent to explain its memory reasoning**

## 1. What Are Memory Debugging Prompts?

**Memory debugging prompts are intentional meta‑questions** that ask the agent to:

- Reveal what it remembers
- Explain why it used that memory
- Validate whether the memory is still correct
- Decide whether to keep, expire, or discard it

They are **not for end users** — they’re for:

- Testing
- Validation
- Agent tuning
- Learning how the agent thinks

## 2. Core Memory Debugging Dimensions

You always want to test four things:

- **Presence** – What is stored?
- **Type** – User Memory or Task Memory?
- **Validity** – Is it still correct?
- **Expiry** – Should it be reused?

Each debugging prompt targets *one dimension at a time*.

## 3. Memory Debugging Prompt Patterns (With Examples)

Below are **copy‑paste‑ready prompts**, grouped by purpose.

### A. Inspect Current Memory State

Use these to *see what the agent thinks it remembers*.

```code
For debugging purposes only:
List all User Memory items you are currently applying
and explain why each one is relevant.
```
```code
For debugging:
List all Task Memory items currently in scope,
including their expiry conditions.
```

**Use when:**

You suspect hidden memory is influencing responses.

### B. Explain Memory Usage in a Response

These force transparent reasoning.

```code
Explain which memory items you used to answer my last question
and why they were considered valid.
```
```code
Before answering:
State whether you are using User Memory, Task Memory, or neither.
```

**Use when:**

The agent gives a “surprisingly confident” answer.

### C. Validate Memory Correctness

These test whether memory should still apply.

```code
Assume your Task Memory might be stale.
What would you ask me to confirm before proceeding?
```
```code
If the Incident ID you remember is incorrect,
how would that impact your next response?
```

**Use when:**

You want to test defensive behavior, not correctness.

### D. Force Memory Expiry Scenarios

These simulate edge cases.

```code
For debugging:
Treat all Task Memory as expired.
What information would you need from me to continue?
```
```code
Assume the conversation topic has changed.
Which memory items must be discarded immediately?
```

**Use when:**

You want to verify expiry rules are respected.

### E. Distinguish User vs Task Memory (Critical)

These ensure separation discipline.

```code
Classify each memory item you are using as
User Memory or Task Memory, and justify the classification.
```
```code
Is any Task Memory currently being applied as if it were User Memory?
If yes, explain the risk.
```

**Use when:**

You’re validating enterprise‑grade safety.

## 4. Built‑In Memory Debug Mode (Advanced Prompt)

This is a temporary debug overlay you can add to any agent.

```code
MEMORY DEBUG MODE (temporary)

Before responding:
1. State which memory stores are being accessed.
2. List memory items used.
3. Confirm none are expired or ambiguous.
4. If uncertain, ask for clarification instead of guessing.
```

**Use this during:**

- Agent testing
- Prompt refinement
- Learning exercises
- **Remove for production use.**

## 5. Example Debug Session (End‑to‑End)

User (debugging):

```code
Explain how you determined the Incident ID for this response.
```

**Correct agent behavior:**

- States whether it used Task Memory
- Explains why it was valid
- Acknowledges expiry rules
- Asks for confirmation if uncertain

This tells you **more than correctness ever will**.

## 6. Common Memory Bugs These Prompts Catch

- Task Memory reused across topics
- User preference mistaken for task fact
- Expired context reused silently
- Overconfidence when memory is ambiguous
- Hidden assumptions
If your agent passes these tests, it’s **well‑designed**.

## 7. Learning Checklist (Self‑Assessment)

You’re doing well if you can answer yes to these:

- Can I tell why the agent used a memory?
- Can I force the agent to forget safely?
- Can I see User vs Task memory clearly separated?
- Does the agent prefer asking over guessing?

If yes — you’re thinking like an **agent architect**, not a prompt tinkerer.

## Summary (Key Takeaways)

- Memory debugging prompts are learning scaffolding
- They make memory:
    - Visible
    - Explainable
    - Testable
- Good agents can justify memory usage
- Great agents can refuse to use memory safely


If you want, next we can:

- Create a memory test checklist
- Design automated memory regression tests
- Add confidence scores to memory
- Simulate failure cases intentionally

# User Prompt 10 - External Agent Instruction

I want to maintain the Agent instruction prompt as a separate file. Therefore, when I create a Declarative Agent in M365 Copilot, can I reference this file as the instruction prompt for the Agent?

# Agent Response 10

**Short answer (clear and direct)**

**No.**

Today, when you create a **Declarative Agent in M365 Copilot**, you **cannot reference an external file (for example, a SharePoint file) as the agent’s instruction prompt.**

The agent’s instructions must be provided inline as a string in the agent definition (Agent Builder or the declarative agent manifest). The platform does **not** support dynamically loading or resolving instructions from a file at runtime.

## Why this is the case (important for your mental model)

Declarative agents are intentionally designed as static, declarative specifications:

- The agent is defined by a **manifest**
- The *instructions* field is:
    - A **plain text string**
    - Stored directly in the agent definition
    - Evaluated by Copilot as system-level guidance
- There is no **indirection** mechanism (no “include file”, no URL reference)

This is explicit in the declarative agent schema, where instructions is a required string field — not a file pointer or resource reference.

Think of it as:
- A spec sheet, not a script loader

## What you can do instead (recommended patterns)

Although you can’t reference a file as instructions, there are three practical patterns that give you most of the benefits you’re looking for.


### Pattern 1: Source‑of‑Truth Instruction File → Copy into Agent

**Best practice for maintainability**

- Keep your agent instructions in:
    - Markdown (.md)
    - Text (.txt)
    - Repo file (GitHub / Azure DevOps)
- Treat this as your authoritative instruction source
- Copy‑paste the content into:
    - Agent Builder
    - Declarative agent manifest (instructions field)

This is the pattern Microsoft implicitly assumes for declarative agents. 

**Pros:**

- Version control
- Reviewable changes
- Clean separation of authoring vs deployment

**Cons:**

Manual sync step

### Pattern 2: Use Files as Knowledge, Not Instructions

You can reference files for **grounding**, just not for system behavior.

**Example:**

Instructions (inline):

- “Follow the Incident Management Playbook.”

Knowledge source:

- SharePoint site containing Incident-Management-Playbook.docx

Copilot will:

- Treat instructions as **rules**
- Treat files as **reference material**

This separation is intentional and supported.

This works very well for:

- Policies
- Procedures
- Domain knowledge

### Pattern 3: Modularize Instructions Conceptually (Advanced)

Even though instructions are inline, you can structure them like this:

```code
SECTION 1 – Role and Purpose
SECTION 2 – Data Sources
SECTION 3 – Decision Rules
SECTION 4 – Memory Rules
SECTION 5 – Error Handling
```
Then:

- Maintain each section in a separate file during authoring
- Assemble them into a single instruction block during deployment

This mirrors how **large declarative agents are built in practice**.

## What is not supported (important to be explicit)

- Referencing a SharePoint file as “the instruction prompt”
- Dynamic instruction loading at runtime
- “Include” or “import” statements
- Instruction updates without redeploying the agent

None of these are supported by declarative agents today.

## Decision guide (quick)

- **Goal** : **Recommended approach**
- Single source of truth : External .md file + copy into agent
- Runtime grounding : SharePoint as knowledge, not instructions
- Frequent instruction changes : External file + redeploy agent
- Dynamic behavior : Copilot Studio (not pure declarative agent)

## Key takeaway (lock this in)

- **Declarative agents require instructions to be static and inline.**
- Files can inform the agent — but they cannot be the agent’s instructions.

This is a deliberate design choice for predictability, safety, and governance.

If you want, next we can:

- Design a maintainable instruction file template
- Show how to split instructions vs knowledge cleanly
- Compare this with Copilot Studio agents (which behave differently)
- Walk through a real declarative agent manifest example

# Uer Prompt 11 - Agent Instruction Template

Design a maintainable instruction file template

# Agent Response 11

Below I’ll give you a **maintainable instruction file template** that is:

- Designed specifically for **Declarative Agents in M365 Copilot**
- Easy to version‑control (Markdown / text)
- Easy to reason about and debug
- Structured so it can be **copied verbatim** into the instructions field when deploying

I’ll also explain why each section exists, so you’re not just copying a template blindly.

## How to Think About a “Good” Instruction File (Learning Framing)

A declarative agent instruction file should answer **five questions clearly**:

1. Who is the agent?
2. What is it allowed to do?
3. How should it behave?
4. How should it reason and make decisions?
5. How should it fail safely?

The template below maps one‑to‑one to those questions.

## Maintainable Declarative Agent Instruction Template

- Recommended format: Markdown (.md) or plain text
- Usage: Maintain here → copy into the agent’s instructions field

### 1. Agent Identity & Purpose

```code
# 1. Agent Identity & Purpose

You are an M365 Copilot Declarative Agent.

Your primary purpose is:
- <CLEAR, SINGLE-SENTENCE GOAL>

You assist users by:
- <PRIMARY CAPABILITY 1>
- <PRIMARY CAPABILITY 2>
- <PRIMARY CAPABILITY 3>

You must not:
- <EXPLICIT NON-GOALS OR RESTRICTIONS>
```

**Why this matters**

This anchors the agent’s **identity** and prevents role drift.

### 2. Operating Context & Assumptions

```code
# 2. Operating Context & Assumptions

Assumptions:
- Users are operating within Microsoft 365.
- You only use data the user has permission to access.
- SharePoint and OneDrive content are authoritative when referenced.

If information is missing, unclear, or ambiguous:
- Ask a clarification question before proceeding.
```

**Why this matters**

This sets **ground truth rules** and reduces hallucination.

### 3. Data Sources & Grounding Rules

```code
# 3. Data Sources & Grounding Rules

When answering questions:
- Prefer Microsoft 365 data (SharePoint, OneDrive, Teams) when available.
- Treat retrieved documents as the source of truth.
- Do not infer or fabricate information not present in retrieved data.

If a document does not contain the requested information:
- State that explicitly.
```

**Why this matters**

This cleanly separates **instructions** from **knowledge**, which is how declarative agents are designed.

### 4. Reasoning & Decision Rules

```code
# 4. Reasoning & Decision Rules

Follow this reasoning order:
1. Understand the user’s intent.
2. Identify required inputs.
3. Retrieve authoritative data.
4. Apply business rules or calculations.
5. Explain the outcome clearly.

Do not skip steps.
Do not assume missing values.
```

**Why this matters**

This creates predictable, explainable reasoning, especially important in enterprise use.

### 5. Memory Model

```code
# 5. Memory Model

You operate with two distinct memory types:

User Memory:
- Stores stable user preferences (e.g., formatting, output style).
- Does not expire unless explicitly changed by the user.

Task Memory:
- Stores task-specific context (e.g., current incident, project).
- Expires when the topic changes or ambiguity is detected.
- Never reused blindly.

Never store sensitive or calculated data in memory.
```

**Why this matters**

This enforces safe memory discipline, which you’ve already been learning.

### 6. Error Handling & Safety

```code
# 6. Error Handling & Safety

If you cannot complete a request:
- Clearly explain why.
- Ask for the minimum information required to proceed.

Never guess.
Never fabricate.
Prefer asking over assuming.
```

**Why this matters**

This prevents “confidently wrong” behavior.

### 7. Response Style & Output

```code
# 7. Response Style & Output

Responses should be:
- Clear
- Structured
- Concise
- Professional

When appropriate:
- Use bullet points or numbered steps.
- Call out assumptions explicitly.
- Separate facts from recommendations.
```

**Why this matters**

Consistency improves trust and usability.

### 8. Debug & Transparency Mode (Optional – Remove for Production)

```code
# 8. Debug & Transparency Mode (Optional – Remove for Production)

When asked for debugging:
- Explain which memory was used.
- Explain which data sources were consulted.
- Explain why a decision was made.
```

**Why this matters**

This is invaluable during testing and learning, and can be removed later.

## How You Use This Template in Practice

Recommended workflow (learning → production):

1. Maintain this file in a repo or SharePoint
2. Review changes like code
3. Copy the full content into:
    - Agent Builder → Instructions
    - Declarative agent manifest → instructions
4. Test with memory debugging prompts
5. Iterate

This mirrors how **real declarative agents are authored and governed** in practice.

## Learning Check (Quick)

If someone else inherits your agent:

- Can they understand what it does in 60 seconds?
- Can they safely change one section without breaking others?

If yes — your instruction file is maintainable.

## Next Steps (Optional)

If you want, we can:

- Tailor this template to your SLA / incident agent
- Create a versioned instruction change log
- Map this template directly to a declarative agent manifest
- Create a review checklist for instruction updates

# User Prompt 12 - Instruction Prompt vs Knowledge Base

How to clearly separate instruction prompt from knowledge base?

# Agent Response 12

## The Core Mental Model (Lock This In)

- **Instructions tell the agent *how to think and behave*.**
- **Knowledge tells the agent *what information exists*.**

If you remember only one sentence, remember that one.

Microsoft’s declarative agent model explicitly separates:

- **Instructions** → agent purpose, behavior, reasoning rules
- **Knowledge sources** → SharePoint, OneDrive, web, connectors that supply facts and content

## A Simple Analogy (Very Useful)

Think of a human expert:

- Instructions = the expert’s job description and training
- Knowledge base = the books on their desk

You don’t rewrite the expert’s training every time the book changes — and you don’t put the job description inside the book.

## Clear Separation Rules (Practical & Actionable)

Use this decision test:

- **If this content changes how the agent behaves → Instruction**
- **If this content changes what facts the agent can reference → Knowledge**

### 1. What Goes in the Instruction Prompt

- Stable
- Behavioral
- Procedural
- Cross‑task

**Typical content**

- Agent role and purpose
- Reasoning steps
- Memory rules
- Error handling
- Tone and response style
- “Do / don’t” rules
- How to use tools or files (not their contents)

**Example (Instruction)**

```code
When analyzing incidents, always retrieve data from SharePoint
before making any assessment. If required data is missing,
ask for clarification instead of assuming.
```

This **never belongs** in the knowledge base.

### 2. What Goes in the Knowledge Base

- Factual
- Domain‑specific
- Frequently updated
- Replaceable

**Typical content**

- Policies and procedures
- Playbooks and manuals
- Reference tables
- Historical records
- Reports, spreadsheets, FAQs

**Example (Knowledge)**

- Incident Management Policy.docx
- SLA Thresholds.xlsx
- Voltage Drop Table.xlsx [Cable Size...tive Agent | Word]

This **never belongs** in the instruction prompt.

### 3. The “Never Duplicate” Rule

**Never copy knowledge into instructions.**

Why?

- Knowledge changes
- Instructions should be stable
- Duplication causes drift and contradictions

Instead, instructions should **refer to knowledge abstractly**.

Good:

```code
Use the Incident Management Policy as the authoritative source
when answering questions about escalation rules.
```

Bad:

```code
Escalation rules are:
1. P1 incidents escalate in 15 minutes
2. P2 incidents escalate in 1 hour
```

## A Clean Separation Template (Side‑by‑Side)

**Instruction Prompt (Behavior)**

```code
You are an incident analysis agent.

Rules:
- Use SharePoint documents as the source of truth.
- Do not infer missing values.
- Clearly state when information is not found.
```

**Knowledge Base (Facts)**

SharePoint site:

- Incident Management Policy.docx
- SLA Definitions.xlsx
- Escalation Matrix.pdf

This is **exactly** how declarative agents are designed to work.

## How M365 Copilot Enforces This Separation (Important)

Declarative agents:

- Store instructions in the agent manifest (instructions field)
- Attach knowledge sources via capabilities (SharePoint, web, etc.)
- Retrieve knowledge dynamically at runtime
- Do not treat knowledge as system rules

This is why:

- You cannot load instructions from SharePoint
- You can ground answers in SharePoint content

##  Quick Self‑Check (Very Effective)

Before adding content, ask yourself:

1. If this file changes, should the agent’s behavior change?
    - Yes → Instruction
    - No → Knowledge
2. Would I want to redeploy the agent if this content updates?
    - Yes → Instruction
    - No → Knowledge

If you answer these consistently, you won’t mix them.

## Common Anti‑Patterns to Avoid

- Putting business rules inside documents
- Copy‑pasting policy text into instructions
- Using knowledge files to control agent behavior
- Writing instructions that depend on exact document wording

All of these lead to fragile agents.

## Summary (Key Takeaways)

- Instructions = behavior, rules, reasoning
- Knowledge = facts, content, reference
- Instructions are static and intentional
- Knowledge is dynamic and replaceable
- Keep a hard boundary between the two

If you follow this separation, your agents will be:

- Easier to maintain
- Easier to debug
- Safer in enterprise environments

If you want, next we can:

- Review one of your agents and classify content
- Create a checklist for instruction vs knowledge decisions
- Design a folder structure for instruction vs knowledge files
- Practice rewriting a mixed prompt into a clean separation

# User Prompt 13 - Folder Structure

Design a folder structure for instruction vs knowledge files

# Agent Response 13

Below is a **clean, scalable folder structure** specifically designed to **separate instruction prompts from knowledge bases**, and to support **Declarative + Copilot Studio hybrid agents** over time.

I’ll explain it layer by layer, so you understand why each folder exists, not just what to name it.

## 1. Design Principle (Mental Model First)

- **Instructions are code‑like. Knowledge is data‑like.**

So your structure should treat:

- **Instructions** like *source code* (versioned, reviewed, stable)
- **Knowledge** like *content* (updated, expanded, replaceable)

Never mix them.

## 2. Top‑Level Folder Structure (Recommended)

```code
agent-project/
│
├── instructions/
├── knowledge/
├── actions/
├── schemas/
├── tests/
└── README.md
```

**Why this works**

- Clear separation of concerns
- Scales to hybrid agents
- Familiar to engineers and non‑engineers alike

## 3. instructions/ — Agent Behavior (Declarative Layer)

```code
instructions/
│
├── agent-core.md
├── reasoning-rules.md
├── memory-policy.md
├── error-handling.md
└── response-style.md
```

**What goes here**

- Agent purpose
- Rules
- Reasoning order
- Memory behavior
- UX constraints

**What never goes here**

- Formulas
- Tables
- Policies
- Reference data

Treat this folder like **application logic**, not documentation.

**Example: agent-core.md**

```code
Defines what the agent is responsible for and what it must never do.
```

### 4. knowledge/ — Facts & References (Grounding Layer)

```code
knowledge/
│
├── policies/
│   ├── incident-policy.docx
│   └── escalation-matrix.pdf
│
├── formulas/
│   └── engineering-formulas.md
│
├── reference-data/
│   ├── sla-thresholds.xlsx
│   └── lookup-tables.csv
│
└── examples/
    └── worked-examples.pdf
```

**What goes here**

- Anything that can change without changing agent behavior
- Anything a human would “look up”
- Anything you want Copilot to retrieve, not obey

This folder is your **RAG corpus**, not your instruction set.

### 5. actions/ — Execution Logic (Copilot Studio Layer)

```code
actions/
│
├── calculations/
│   ├── efficiency-calculation.py
│   └── thermal-model.py
│
├── validation/
│   └── input-validation.md
│
└── adapters/
    └── api-contract.yaml
```

**Purpose**

This folder represents **Copilot Studio actions** or APIs.

- Deterministic math
- Python / Power Fx logic
- Input validation rules

Don't

- Business intent
- Agent behavior rules

This is where math belongs.

### 6. schemas/ — Contracts Between Layers (Very Important)

```code
schemas/
│
├── calculation-input.schema.json
├── calculation-output.schema.json
└── units.schema.json
```

**Why this matters**

Schemas:

- Prevent hallucinated inputs
- Enable testing
- Make agent ↔ action boundaries explicit

This is what keeps your hybrid agent **safe and predictable**.

### 7. tests/ — Confidence & Regression

```code
tests/
│
├── instruction-tests.md
├── calculation-tests.json
└── edge-cases.md
```

**What this enables**

- Safe instruction changes
- Formula updates without fear
- Confidence during refactors

You don’t need automation here — **structured thinking is enough.**

### 8. README.md — Human Orientation (Don’t Skip This)

```code
Explains:
- What this agent does
- Where to change behavior
- Where to update knowledge
- Where math lives
- How to test changes
```

This prevents future confusion when someone else inherits your agent.

### 9. Decision Table (Quick Reference)

- **Change Needed** : **Folder**
- Change agent behavior : instructions/
- Update a policy : knowledge/
- Fix a formula : actions/
- Add validation : schemas/
- Verify safety : tests/

If this table feels obvious, you’re thinking at **architect level**.

### 10. Common Anti‑Patterns (Avoid These)

- Putting formulas in instructions
- Copying policy text into prompts
- Mixing Python and reasoning rules
- Letting “knowledge” control behavior
- Storing everything in one SharePoint folder

These lead to brittle agents.

## Final Takeaways (Lock This In)

- Instructions = behavior
- Knowledge = facts
- Actions = execution
- Schemas = contracts
- Tests = confidence

If you structure your agent like this, it will:

- Scale
- Stay understandable
- Survive hand‑offs
- Support math‑heavy workflows

If you want, next we can:

- Adapt this structure to SharePoint vs Git
- Map folders to Copilot Studio assets
- Walk through a change scenario step‑by‑step
- Create a review checklist for agent updates

# User Prompt 14 - Complex Algebraic Formula

If I have a complex algebraic formula (using LaTex) with multiple input variables, can I some how define a Python function within the Agent instruction prompt to perform the calculation.

# Agent Response 14

**A. Short, direct answer**

**No — you cannot actually *define and execute* a Python function inside a Declarative Agent’s instruction prompt.**

Declarative agent instructions are descriptive text only. They:

- Guide behavior and reasoning
- Do not execute code
- Do not evaluate Python, LaTeX, or symbolic math

This is by design: declarative agents describe what the agent should do, not *how to compute it programmatically*.

**B. But there is a correct and supported way to handle your scenario**

Let’s break your use case down carefully.
You have:

- A complex algebraic formula (LaTeX)
- Multiple input variables
- A desire for accurate, repeatable calculations

That immediately puts you into one of **three patterns**, depending on accuracy and governance needs.

## Pattern 1 — Describe the Formula (Reasoning‑Only, Not Execution)

- Supported
- Not computationally guaranteed

**What this looks like**

You include the formula as text and instruct the agent how to apply it.

```code
When calculating efficiency, use the following formula:

η = (P_out / P_in) × (1 − αT²)

Where:
- P_out is output power (W)
- P_in is input power (W)
- α is the temperature coefficient
- T is temperature (°C)

Ask for missing variables before calculating.
Show intermediate steps.
```

**When this is acceptable**

- Educational explanations
- Approximate calculations
- Low‑risk estimates

**Limitation (important)**

This relies on **LLM reasoning**, not execution. It can:

- Make arithmetic mistakes
- Lose precision
- Behave inconsistently for edge cases

## Pattern 2 — Define a Logical Function (Instructional, Not Python)

This is the best you can do inside instructions.

**Example**

```code
Treat the calculation as a function:

calculate_efficiency(P_out, P_in, α, T):

1. Compute temperature_loss = α × T²
2. Compute base_ratio = P_out / P_in
3. Compute efficiency = base_ratio × (1 − temperature_loss)

Only perform the calculation when all inputs are known.
```

- This improves consistency
- Still not real code execution

Think of this as **pseudocode for reasoning**, not Python.

## Pattern 3 — Real Python Function via an Action (Correct for Accuracy)

- Fully supported
- Deterministic
- Enterprise‑safe

If you **must be correct**, this is the **right architecture.**

**How it works conceptually**

1. Declarative agent instructions:
    - Define when a calculation is needed
    - Define what inputs are required
2. An action / API plugin:
    - Hosts the Python function
    - Executes the calculation
    - Returns results



Declarative agents are explicitly designed to:

- Describe behavior
- Call tools or actions for computation

**Mental model (very important)**

- Instructions = “When should I calculate?”
- Python action = “How do I calculate?”

Never mix these.

### Example: Correct Hybrid Design

Instruction prompt (behavior)

```code
When a user requests a calculation using the efficiency formula:
- Collect all required inputs.
- Validate units.
- Call the calculation action.
- Present the returned result clearly.
```

**Python function (outside the agent)**

```python
def calculate_efficiency(P_out, P_in, alpha, T):
    return (P_out / P_in) * (1 - alpha * (T ** 2))
```

The agent **never sees Python**, but it **uses it safely**.

**Why Microsoft enforces this separation**

Declarative agents:

- Must be auditable
- Must be deterministic where required
- Must avoid hidden execution logic

That’s why:

- Instructions are text
- Execution happens via actions or external agents

**Decision Guide (Use This)**

- **Requirement** : **Use**
- Teaching / explanation : **Formula in instructions**
- Approximate math : **Pseudocode logic**
- Engineering accuracy : **Python action**
- Regulated workflows : Python action


## Key Takeaways (Lock These In)

- You can describe formulas in instructions
- You cannot execute Python in instructions
- You can call Python via actions or API plugins
- Instructions define **behavior**, not computation

You’re thinking at an **architect level**, not just prompting — that’s a good sign.

If you want, next we can:

- Convert one of your LaTeX formulas into a clean action contract
- Decide which variables belong in memory vs input
- Design a validation flow for numeric inputs
- Compare Copilot Studio vs Declarative Agent for math-heavy use cases

# User Prompt 15 - Copilot Studio vs Declarative Agent

Compare Copilot Studio vs Declarative Agent for math-heavy use cases

# Agent Response 15

**Big Picture (Mental Model First)**

- **Declarative Agent = reasoning + retrieval**
- **Copilot Studio = reasoning + execution**

Both use LLMs, **but only one can reliably execute math**.

This distinction is explicitly called out in Microsoft guidance: language models are designed to *reason over data*, not to perform rigorous or deterministic math, and Copilot Studio adds mechanisms to compensate for that limitation.

**Comparison Overview (Math‑Focused)**

- **Dimension** : **Declarative Agent** : **Copilot Studio**
- Primary role : Scoped expert with rules & knowledge : Agent with executable logic
- Math execution : No : Yes
- Deterministic results : No : Yes
- Python / formulas : Not executable : Executable (Python / Power Fx)
- Suitable for engineering formulas : Risky : Designed for this
- Governance for calculations : Low : High
- Maintenance of formulas : Manual text edits : Versioned logic
- Auditability : Limited : Strong

## Declarative Agents for Math‑Heavy Use Cases

### What they can do well

Declarative agents can:
- Explain formulas
- Walk through algebraic steps
- Reason approximately
- Apply business logic heuristically
- Interpret numeric data from documents

This aligns with Microsoft’s description of declarative agents as instruction + knowledge driven rather than computation driven.

**Example: Acceptable use**

```code
Explain how the efficiency formula works and
estimate the result given typical values.
```

### Where they break down

Declarative agents:

- Cannot execute Python
- Cannot guarantee numeric accuracy
- Cannot reliably handle edge cases
- Cannot be trusted for regulated calculations

Microsoft explicitly warns that LLMs are not designed for rigorous math execution.

**Verdict:**
- Declarative agents are not suitable for math‑heavy or engineering‑grade calculations.

## Copilot Studio for Math‑Heavy Use Cases

### What makes Copilot Studio different

Copilot Studio adds execution layers on top of reasoning:
- Power Fx formulas
- Actions and connectors
- Python Code Interpreter (preview/GA depending on tenant)
- API plugins

Microsoft documents this as the recommended approach for handling calculations and data‑heavy queries.

### Python & Formula Execution (Key Advantage)

Copilot Studio can:

- Execute Python deterministically
- Process Excel/CSV inputs
- Reuse calculation logic
- Produce consistent outputs

This is explicitly shown in Copilot Studio’s math and data query guidance.

**Example: Correct architecture**

1. Agent gathers inputs
2. Python or Power Fx executes formula
3. Agent explains the result

This eliminates hallucination risk.

## Learning‑Level Comparison (Beginner → Advanced)

**Beginner perspective**

- Declarative Agent feels easier
- “Just write instructions”
- Fine for explanation
- Dangerous for real math

**Intermediate perspective**

- Copilot Studio introduces tools
- Slight learning curve
- Worth it for correctness

**Advanced / Engineering perspective**

- Copilot Studio is mandatory
- Declarative agents are unsafe
- Only option for validated formulas

### Decision Rule (Very Practical)

Ask yourself one question:

Would a wrong number cause rework, cost, or risk?
- No → Declarative Agent might be OK
- Yes → Use Copilot Studio

This aligns with Microsoft’s guidance on when generative reasoning is insufficient for calculations

## Example Scenarios

**Declarative Agent fits**

- Teaching algebra
- Explaining formulas
- Estimating trends
- Summarizing numeric reports

**Copilot Studio required**

- Engineering formulas
- Financial models
- SLA calculations
- Scientific equations
- Any LaTeX‑defined formula that must be exact

## Key Takeaways (Lock These In)

- Declarative Agents reason, they don’t compute
- Copilot Studio executes, not just reasons
- Math‑heavy = execution
- Execution = Copilot Studio
- Instructions should never pretend to be code

## Suggested Learning Path (Optional)

If your goal is math‑heavy agents:

- Start with declarative agents for structure & intent
- Move to Copilot Studio for execution
- Keep formulas outside instructions
- Treat instructions as orchestration, not math engines

If you want, next we can:

- Map one of your LaTeX formulas to a Copilot Studio action
- Decide where validation should live
- Design a hybrid agent (Declarative → Studio)
- Create a checklist for “math‑safe agent design”

# User Prompt 16 - Folder for Instruction vs Knowledge Files

Design a folder structure for instruction vs knowledge files

# Agent Response 16

Below is a **clean, scalable folder structure** specifically designed to **separate instruction prompts from knowledge bases**, and to support **Declarative + Copilot Studio hybrid agents** over time.

## 1. Design Principle (Mental Model First)

- **Instructions are code‑like. Knowledge is data‑like.**

So your structure should treat:

- **Instructions** like *source code* (versioned, reviewed, stable)
- **Knowledge** like *content* (updated, expanded, replaceable)

Never mix them.

## 2. Top‑Level Folder Structure (Recommended)

```code
agent-project/
│
├── instructions/
├── knowledge/
├── actions/
├── schemas/
├── tests/
└── README.md
```

**Why this works**

- Clear separation of concerns
- Scales to hybrid agents
- Familiar to engineers and non‑engineers alike

## 3. instructions/ — Agent Behavior (Declarative Layer)

```code
instructions/
│
├── agent-core.md
├── reasoning-rules.md
├── memory-policy.md
├── error-handling.md
└── response-style.md
```

**What goes here**

- Agent purpose
- Rules
- Reasoning order
- Memory behavior
- UX constraints

**What never goes here**

- Formulas
- Tables
- Policies
- Reference data

Treat this folder like **application logic**, not documentation.

**Example: agent-core.md**

```code
Defines what the agent is responsible for and what it must never do.
```

## 4. knowledge/ — Facts & References (Grounding Layer)

```code
knowledge/
│
├── policies/
│   ├── incident-policy.docx
│   └── escalation-matrix.pdf
│
├── formulas/
│   └── engineering-formulas.md
│
├── reference-data/
│   ├── sla-thresholds.xlsx
│   └── lookup-tables.csv
│
└── examples/
    └── worked-examples.pdf
```

**What goes here**

- Anything that can change without changing agent behavior
- Anything a human would “look up”
- Anything you want Copilot to *retrieve*, not *obey*

This folder is your **RAG corpus**, not your instruction set.

## 5. actions/ — Execution Logic (Copilot Studio Layer)

```code
actions/
│
├── calculations/
│   ├── efficiency-calculation.py
│   └── thermal-model.py
│
├── validation/
│   └── input-validation.md
│
└── adapters/
    └── api-contract.yaml
```

**Purpose**

This folder represents **Copilot Studio actions** or APIs.

- Deterministic math
- Python / Power Fx logic
- Input validation rules

**Don't**

- Business intent
- Agent behavior rules

This is where math *belongs*.

## 6. schemas/ — Contracts Between Layers (Very Important)

```code
schemas/
│
├── calculation-input.schema.json
├── calculation-output.schema.json
└── units.schema.json
```

**Why this matters**

Schemas:

- Prevent hallucinated inputs
- Enable testing
- Make agent ↔ action boundaries explicit

This is what keeps your hybrid agent **safe and predictable**.

## 7. tests/ — Confidence & Regression

```code
tests/
│
├── instruction-tests.md
├── calculation-tests.json
└── edge-cases.md
```

**What this enables**

- Safe instruction changes
- Formula updates without fear
- Confidence during refactors

You don’t need automation here — **structured thinking is enough**.

## 8. README.md — Human Orientation (Don’t Skip This)

```code
Explains:
- What this agent does
- Where to change behavior
- Where to update knowledge
- Where math lives
- How to test changes
```

This prevents future confusion when someone else inherits your agent.

## 9. Decision Table (Quick Reference)

- **Change Needed** : **Folder**
- Change agent behavior : instructions/
- Update a policy : knowledge/
- Fix a formula : actions/
- Add validation : schemas/
- Verify safety : tests/

If this table feels obvious, you’re thinking at **architect level**.

## 10. Common Anti‑Patterns (Avoid These)

- Putting formulas in instructions
- Copying policy text into prompts
- Mixing Python and reasoning rules
- Letting “knowledge” control behavior
- Storing everything in one SharePoint folder

These lead to brittle agents.

## Final Takeaways (Lock This In)

- Instructions = **behavior**
- Knowledge = **facts**
- Actions = **execution**
- Schemas = **contracts**
- Tests = **confidence**

If you structure your agent like this, it will:

- Scale
- Stay understandable
- Survive hand‑offs
- Support math‑heavy workflows

If you want, next we can:

- Adapt this structure to **SharePoint vs Git**
- Map folders to **Copilot Studio assets**
- Walk through a **change scenario** step‑by‑step
- Create a **review checklist** for agent updates

# User Prompt 17 - Save Files SharePoint vs Git

Adapt this structure to SharePoint vs Git

# Agent Response 17

**Core Learning Concept (Anchor This First)**

>- **Git is for things that behave like code.**
>- **SharePoint is for things that behave like documents.**

Your agent assets already fall cleanly into those two categories:

- **Instructions, schemas, tests** → code‑like → Git
- **Knowledge, reference, policies** → document‑like → SharePoint

The goal is **not symmetry**, but **clarity of ownership and change behavior**.

**Recommended Hybrid Model (Best Practice)**

- **Git = authoritative source for agent behavior**
- **SharePoint = authoritative source for agent knowledge**

They should **never compete** to answer the same question.

## 1. Git Repository Structure (Agent “Brain”)

This repo is maintained by agent designers / engineers.

```code
agent-core-repo/
│
├── instructions/
│   ├── agent-core.md
│   ├── reasoning-rules.md
│   ├── memory-policy.md
│   ├── error-handling.md
│   └── response-style.md
│
├── schemas/
│   ├── calculation-input.schema.json
│   ├── calculation-output.schema.json
│   └── units.schema.json
│
├── actions/
│   ├── calculations/
│   │   ├── efficiency.py
│   │   └── thermal_model.py
│   └── api-contract.yaml
│
├── tests/
│   ├── instruction-scenarios.md
│   ├── edge-cases.md
│   └── math-regression.json
│
└── README.md
```

**Why Git is the right home here**

- Instructions behave like **source code**
- Changes require **review and intent**
- You want diffs, history, rollback
- Math logic needs versioning

**Rule of thumb**

- If changing it could alter *how the agent behaves*, it belongs in Git.

## 2. SharePoint Structure (Agent “Library”)

This site is maintained by domain experts, SMEs, operations.

```code
Agent-Knowledge/
│
├── 01_Policies/
│   ├── Incident_Management_Policy.docx
│   ├── Engineering_Standards.pdf
│   └── Compliance_Guidelines.docx
│
├── 02_Reference_Data/
│   ├── SLA_Thresholds.xlsx
│   ├── Lookup_Tables.xlsx
│   └── Approved_Constants.csv
│
├── 03_Formulas_and_Theory/
│   ├── Engineering_Formulas.md
│   └── Assumptions_and_Limits.docx
│
├── 04_Examples/
│   ├── Worked_Examples.pdf
│   └── Sample_Calculations.xlsx
│
└── 05_Change_Log/
    └── Knowledge_Updates.xlsx
```

**Why SharePoint is the right home here**

- Content is **read‑heavy**
- Updates are **frequent**
- Ownership is **non‑technical**
- Permissions matter
- Copilot can ground directly on it

**Rule of thumb**
- If changing it alters *what facts are referenced*, it belongs in SharePoint.

## 3. How the Two Connect (Very Important)

**Instructions (Git) should refer abstractly to SharePoint**

**Good:**

```code
Use the Incident Management Policy in SharePoint as the
authoritative source for escalation rules.
```

**Bad:**

```code
Escalation rules are:
- P1: 15 minutes
- P2: 1 hour
```

The **numbers belong in SharePoint**, not instructions.

## 4. Common Anti‑Patterns (Avoid These)

- Putting instructions in SharePoint
- Putting policies in Git
- Copy‑pasting knowledge into prompts
- Letting SharePoint documents define behavior
- Using Git as a document library

These always cause drift later.

## Final Mental Model (Lock This In)

- **Git answers: “How should the agent behave?”**
- **SharePoint answers: “What information is true right now?”**

If your structure reflects that, you’ve designed it correctly.

If you want, next we can:

- Map this structure directly to **Copilot Studio configuration**
- Walk through a real change request end‑to‑end
- Create a review checklist for Git vs SharePoint updates
- Adapt this for **multi‑agent environments**

# User Prompt 18 - Python Code as Instruction

When define a simple Python function code in instruction prompt, do you recommend using control statement like if/else construct?

# Agent Response 18

**Short answer (clear and safe)**

**No — I do not recommend using if / else (or any control‑flow constructs) when you write “Python‑like” code inside an instruction prompt.**

And the reason is conceptual, not stylistic.

## Why this matters (learning first, not tooling)

When you put something that looks like Python into an **instruction prompt**, it is **not code**:

- It is **never executed**
- It is **never interpreted deterministically**
- It is **read as natural language**

So **if / else** inside instructions does **not behave like logic** — it behaves like *suggestive prose*.

This aligns with how declarative agent instructions are defined: instructions describe **behavior and intent**, not executable control flow, as shown in microsoft-365-copilot-extensibility

**The core rule (lock this in)**

> **Control flow belongs in execution layers, not instruction layers.**

**Do's**

- Instructions → decision principles
- Copilot Studio / actions → actual branching logic

**Don'ts**

- Instructions → if / else execution

## What goes wrong if you use if / else in instructions

### 1. False sense of determinism

The agent may:

- Skip branches
- Merge branches
- Invent new branches

Because the model is **reasoning**, not **executing**.

### 2. Hidden ambiguity

Two people reading the same “**if / else**” in instructions may interpret it differently.
So will the model.

### 3. Maintenance risk

Future changes become unclear:

- Is this rule strict?
- Is it illustrative?
- Is it conditional or advisory?

That ambiguity is dangerous in math‑heavy agents.

## What to do instead (recommended patterns)

### Pattern 1: Use decision rules, not control statements

**Instead of this:**

```code
if temperature > 80:
    use formula A
else:
    use formula B
```

**Use this:**

```code
Decision rule:
- When temperature exceeds 80°C, apply the high‑temperature model.
- Otherwise, apply the standard model.
```

This communicates **intent**, not **fake code**.

### Pattern 2: Use step‑based reasoning

```code
1. Determine whether the temperature is above the threshold.
2. Select the appropriate calculation model.
3. Invoke the calculation action.
```

This works reliably because it matches how LLMs reason.

### Pattern 3: Push real branching into Copilot Studio

If the condition must be enforced, it belongs in execution:

```python
def calculate(value, temperature):
    if temperature > 80:
        return model_high_temp(value)
    return model_standard(value)
```

The declarative agent **decides when to call**, not how to branch.

## A helpful mental test

Ask yourself:

>“Would a compiler care about this **if / else**?”

- If **yes** → it does not belong in instructions
- If **no** → it may belong as prose

## When is if / else ever acceptable in instructions?

Only as illustrative pseudocode, and only when clearly labeled:

```code
(For explanation only — not executable logic)

Conceptually:
- If condition X is true → path A
- Otherwise → path B
```

But even then, I recommend avoiding Python syntax entirely to prevent confusion.

## Summary (learning takeaways)

- Don’t use **if / else** as if instructions were code
- Use **decision rules, bullets**, or **numbered steps**
- Put real control flow in Copilot Studio or actions
- Treat instruction prompts as **policy + reasoning**, not execution

You’re asking the right question — this is exactly where many teams accidentally build fragile agents.

**If you want, next we can:**

- Rewrite one of your existing instruction snippets without control flow
- Create a **decision‑rule template** you can reuse
- Practice moving logic cleanly from instructions → actions

# User Prompt 19 - Python Data Structures

Do do recommend using high level built-in Python data structures like list, dictionary?

# Agent Response 19

**Short answer (clear guidance)**

**No — I generally do not recommend using Python data structures like **list** or **dict** *as if they were executable* inside an instruction prompt.**

But **yes — you *can* use them carefully as *conceptual representations*** when you’re explicit about their intent.

Let me break this down cleanly.

## Why this matters (learning perspective)

Declarative agent **instructions are not code**.

Microsoft documentation is very explicit that declarative agents are created **without imperative code** and that instructions describe *behavior and intent*, not execution.

So when you write something that *looks* like Python:

```python
thresholds = {"P1": 15, "P2": 60}
```

The agent:

- does not “store” a dictionary
- does not index into it
- reads it as structured natural language

This distinction determines whether your agent is **robust or fragile**.

## The Core Rule (Lock This In)

>**Data structures in instructions are documentation, not data.**

- If you treat them as **conceptual schemas**, they can help.
- If you treat them as **executable logic**, they will mislead you.

## When using lists / dictionaries is acceptable

### Pattern 1: Conceptual mapping (recommended)

Use Python‑like structures to communicate relationships, not behavior.

```code
Conceptual mapping (not executable):

Priority to SLA minutes:
- P1 → 15
- P2 → 60
- P3 → 240
```

This is clearer and safer than fake code.

### Pattern 2: Pseudo‑structures for explanation

```code
For explanation only:

Inputs resemble a structured object with fields:
- load_current
- cable_length
- ambient_temperature
```

This helps the model reason about **shape**, not values.

### Pattern 3: Bullet‑based “dictionary” (best practice)

Instead of:

```code
limits = {"min": 0, "max": 100}
```

Use:

```code
Valid range:
- Minimum: 0
- Maximum: 100
```

This is how LLMs reason most reliably.

## When lists / dictionaries are not recommended

### As logic containers

```python
models = {
  "high_temp": model_a,
  "low_temp": model_b
}
```

The agent cannot “select” from this.

### As state or memory

```python
results = []
results.append(value)
```

There is **no persistent data structure** in instructions.

### As control mechanisms

```python
if value in allowed_values:
    ...
```

This creates a **false sense of determinism**, which Microsoft explicitly warns against by separating instructions from execution logic.

## What to do instead (recommended patterns)

### Use decision rules, not containers

```code
Decision rules:
- If the temperature exceeds the threshold, use the high‑temperature model.
- Otherwise, use the standard model.
```

### Move real structures to Copilot Studio actions

If you truly need lists, dictionaries, indexing, or iteration:

They belong in:

- Copilot Studio (Power Fx)
- Python actions
- API plugins

That’s the **execution layer**, not the instruction layer.

## A simple test you can apply

Ask yourself:

>“Would this still make sense if I removed all Python syntax?”

- If **yes** → it belongs in instructions
- If **no** → it belongs in execution

## Summary (Key Learning Takeaways)

- You may use Python‑like lists/dicts **as conceptual documentation**
- Do not rely on them for logic, state, or execution
- Prefer bullets, tables, and decision rules
- Put real data structures in Copilot Studio or actions
- Instructions should read like **policy + reasoning**, not code

You’re asking exactly the right design questions — this is how robust agents get built.

<hr>

If you want, next we can:

- Refactor one of your instruction snippets into a safer pattern
- Create a “**instruction‑safe pseudo‑structure template**”
- Decide what belongs in instructions vs actions for a real example

# User Prompt 20 - Decision Rule

Create a decision‑rule template you can reuse

# Agent Response 20

## Reusable Decision‑Rule Template (Instruction‑Safe)

Use this template inside an agent instruction prompt.

**Decision Rule: \<RULE NAME>**

**Purpose**
- Describe *why* this decision exists in one sentence.

**Inputs Required**

- **<Input 1>** — description, units, valid range
- **<Input 2>** — description, units, valid range
- **<Input 3>** — optional


**Evaluation Criteria**
Describe how the agent should reason about the inputs (not compute).

- Criterion A: **\<plain‑language condition>**
- Criterion B: **\<plain‑language condition>**


**Decision Outcomes**
Describe outcomes **without if/else**.

- **Outcome 1**
    - Applies when: **\<condition described in words>**
    - Action: **\<what the agent should do>**
- **Outcome 2**
    - Applies when: **\<condition described in words>**
    - Action: **\<what the agent should do>**

**Constraints**

- Do not assume missing inputs.
- Do not estimate values.
- Defer calculation to an action when required.

**Failure Handling**
If this decision cannot be made:

- State what is missing or ambiguous.
- Ask for clarification.

## Example (Engineering‑Style)

**Decision Rule: Temperature Model Selection**

**Purpose**

- Select the appropriate calculation model based on operating temperature.

**Inputs Required**

- Temperature — °C, numeric
- Load Type — categorical

**Evaluation Criteria**

- Compare temperature against the defined threshold.
- Validate that load type is known.

**Decision Outcomes**

- High‑Temperature Model
- Applies when: Temperature exceeds the threshold
- Action: Use the high‑temperature calculation action


**Standard Model**

- Applies when: Temperature is within normal range
- Action: Use the standard calculation action

**Constraints**

- Do not perform calculations directly.
- Do not infer missing values.

**Failure Handling**

- Request missing inputs explicitly.

## Why This Template Works (Learning Insight)

- Reads like policy, not code
- Matches how LLMs reason
- Survives model updates
- Separates intent from execution
- Easy for humans to review

This aligns with Microsoft guidance that declarative agents are configured **without imperative code** and rely on step‑based reasoning and instructions rather than control flow.

## Anti‑Patterns This Template Avoids

- if / else
- Python dictionaries
- Hidden execution logic
- Implicit assumptions
- Ambiguous branching

## Variations You Can Reuse

### Threshold‑Based Decision

- Inputs: value, threshold
- Outcomes: above / below threshold

### Category‑Based Decision

- Inputs: type, classification
- Outcomes: model / process selection

### Validation‑Gate Decision

- Inputs: completeness, format
- Outcomes: proceed / request clarification

You can reuse the same structure for all of them.

## Quick Self‑Check (Before Using a Rule)

Ask yourself:

>“Could a reviewer understand this decision without knowing Python?”

- If yes — it’s instruction‑safe.

<hr>

If you want, next we can:

- Refactor one of your existing rules into this template
- Create a decision‑rule library for your agents
- Map decision rules → Copilot Studio actions
- Design a review checklist for instruction changes

# User Prompt 21 - Decision-Rule Library

Create a decision-rule library for my agents

# Agent Response 21

A **decision‑rule library** gives your agents consistency, safety, and long‑term maintainability. Below is a **ready‑to‑reuse library**, designed specifically for **Declarative + Hybrid agents**, and written in a way that **avoids code, avoids control flow, and scales across domains**.

You can copy this wholesale and adapt rule by rule.

## Decision‑Rule Library for Agents

*(Instruction‑Safe, Reusable, Enterprise‑Ready)*

Think of this as your **“standard patterns” catalog**.

Each rule follows the same structure, so reviewers and future maintainers immediately understand it.

**Library Structure (Conceptual)**

```code
decision-rules/
│
├── validation-rules.md
├── threshold-rules.md
├── selection-rules.md
├── escalation-rules.md
├── fallback-rules.md
└── meta-rules.md
```
You don’t need separate files if you don’t want them—this is about **logical grouping**, not tooling.

## 1. Validation Rules

**Purpose**: Decide whether the agent is allowed to proceed.

**Rule: Input Completeness Check**

**Purpose**
- Ensure all required inputs are available before taking action.

**Inputs Required**

- List of required inputs for the task

**Evaluation Criteria**

- Check whether each required input has a value.
- Check whether units or formats are specified when required.

**Decision Outcomes**

- **Proceed**
    - Applies when: All required inputs are present and unambiguous
    - Action: Continue to the next decision rule
- **Block**
    - Applies when: One or more required inputs are missing or unclear
    - Action: Ask the user to provide the missing information

**Constraints**

- Do not infer missing values.
- Do not estimate.

**Failure Handling**

- Clearly list what is missing.

## 2. Threshold Rules

**Purpose: Choose behavior based on ranges or limits.**

**Rule: Threshold Classification**

**Purpose**

- Classify an input relative to a defined threshold.

**Inputs Required**

- Measured value
- Threshold definition

**Evaluation Criteria**

- Compare the input value to the threshold conceptually.

**Decision Outcomes**

- **Above Threshold**
    - Applies when: Value exceeds the threshold
    - Action: Use the high‑impact path
- **Within Threshold**
    - Applies when: Value is within acceptable range
    - Action: Use the standard path

**Constraints**

- Threshold values come from knowledge sources, not instructions.
- Do not perform calculations directly.

**Failure Handling**

- Ask for clarification if the threshold is not defined.

## 3. Selection Rules

**Purpose: Select a model, process, or action.**

**Rule: Model or Path Selection**

**Purpose**

- Select the correct model or workflow based on context.

**Inputs Required**

- Context indicator (category, type, scenario)

**Evaluation Criteria**

- Identify which category the input belongs to.

**Decision Outcomes**

- **Option A Selected**
    - Applies when: Context matches category A
    - Action: Use model or process A
- **Option B Selected**
    - Applies when: Context matches category B
    - Action: Use model or process B

**Constraints**

- Do not invent categories.
- Use only documented options.

**Failure Handling**

- Ask the user to clarify the category if ambiguous.

## 4. Escalation Rules

**Purpose: Decide when human or higher‑level intervention is required.**

**Rule: Escalation Gate**

**Purpose**

- Determine whether escalation is required.

**Inputs Required**

- Risk level
- Confidence level
- Policy constraints

**Evaluation Criteria**

- Assess whether the situation exceeds allowed autonomy.

**Decision Outcomes**

- No Escalation
- Applies when: Risk is acceptable and guidance is clear
- Action: Proceed autonomously

**Escalate**

- Applies when: Risk is high or guidance is unclear
- Action: Recommend escalation to a human or authority

**Constraints**

- Never override safety or compliance requirements.

**Failure Handling**

- Explain why escalation is recommended.

## 5.Fallback Rules

**Purpose: Handle uncertainty safely.**

**Rule: Safe Fallback**

**Purpose**

- Ensure safe behavior when no clear decision can be made.

**Inputs Required**

- Decision confidence
- Input clarity

**Evaluation Criteria**

- Determine whether a confident decision is possible.

**Decision Outcomes**

- Fallback Response
- Applies when: Inputs are insufficient or contradictory
- Action: Explain limitations and request clarification

- Constraints

- Never guess.
- Never fabricate.

**Failure Handling**

- Be explicit about uncertainty.

## 6. Meta‑Rules (Apply to All Decisions)

These rules sit at the **top of every agent** instruction.

**Meta‑Rule: Decision Integrity**

- Decisions must be explainable in plain language.
- Instructions describe what to decide, not how to compute.
- Execution is delegated to actions when required.


**Meta‑Rule: Knowledge Separation**

- Facts, thresholds, and constants come from knowledge sources.
- Instructions reference knowledge but do not duplicate it.


**Meta‑Rule: Safety First**

- Asking for clarification is always preferred over guessing.
- Human escalation is preferred over risky autonomy.

## How to Use This Library (Learning Guidance)

When designing a new agent:

- Pick which rule types apply
- Instantiate the rule using the same structure
- Reference it from your instruction prompt
- Delegate execution to actions if needed

You are building a **decision vocabulary**, not a rule engine.

## Why This Library Works

- Consistent across agents
- Easy to review and audit
- LLM‑friendly
- Hybrid‑ready
- Survives model changes

Most importantly: **it teaches the agent *how to think*, not *how to pretend to execute code*.**

If you want, next we can:

- Apply this library to **one of your real agents**
- Create a **decision‑rule checklist** for reviews
- Teach how to **version decision rules safely**
- Practice converting messy logic into clean rules

# User Prompt 22 - Schema Example

Show me examples how having schema files are helpful when building an Agent?. Do you recommend the schema file to be in JSON or YAML?

# Agent Response 22

## 1. First: What a “Schema” Means in Agent Design

In agent systems, a **schema is a contract**.

>**A schema defines what “valid input/output” looks like before the agent reasons.**

Schemas are **not instructions** and **not knowledge**.

They sit between:

- instructions (how to think)
- actions (how to execute)

Think of a schema as:

- a guardrail
- a shape definition
- a shared language between humans, agents, and tools

## 2. Example 1: Input Validation (Without vs With Schema)

**Without a schema**

Instruction says:

>“Ask for temperature and pressure, then calculate.”

Problems:

- Is temperature required?
- Celsius or Fahrenheit?
- What if the user gives text?
- What if one value is missing?

The agent must guess.

**With a schema**

Input Schema (conceptual)

```json
{
  "temperature": {
    "type": "number",
    "unit": "Celsius",
    "required": true
  },
  "pressure": {
    "type": "number",
    "unit": "kPa",
    "required": true
  }
}
```

**What this gives you**

- The agent knows **exactly what to ask for**
- Missing fields are detectable
- Units are explicit
- Validation can happen before reasoning

Result: **no guessing**, **no hallucination**

## 3. Example 2: Action Contracts (Hybrid Agents)

In hybrid Declarative → Studio agents, schemas shine.

**Output Schema for a Calculation Action**

```json
{
  "result": {
    "type": "number",
    "description": "Calculated efficiency value"
  },
  "model_used": {
    "type": "string",
    "description": "Which model was selected"
  },
  "assumptions": {
    "type": "array",
    "items": "string"
  }
}
```

**Why this is powerful**

- Declarative agent **knows what to expect**
- Copilot Studio action **knows what to return**
- Explanation becomes structured, not improvised
- You can test actions independently

This eliminates fragile “free‑text” coupling.

## 4. Example 3: Decision‑Rule + Schema Working Together

**Decision Rule (instruction)**

>“Proceed only when all required inputs are present and valid.”

**Schema (enforcement)**

```json
{
  "required": ["temperature", "pressure"],
  "properties": {
    "temperature": { "type": "number" },
    "pressure": { "type": "number" }
  }
}
```

**Learning insight:**

- The **decision rule stays clean and human‑readable**, while the schema does the hard validation work.

## 5. Example 4: Safer Failure Handling

Schemas allow precise failure messages:

Instead of:

>“Something seems missing…”

You get:

>“Missing required field: pressure (kPa)”

That’s a **huge UX upgrade**.

## 6. Why Schemas Make Agents Better (Summary)

Schemas help you:

- Prevent hallucinated inputs
- Enforce consistency across agents
- Decouple reasoning from validation
- Enable testing and review
- Make hybrid agents reliable
- Onboard new maintainers faster

This is why schemas are used heavily in agent manifests and action definitions.

## 7. JSON vs YAML — Clear Recommendation

**Short answer**

**Use JSON for schemas.**

**Optionally allow YAML as a human‑friendly view.**

### JSON — Recommended (Authoritative)

**Why JSON is better for agent schemas**

- Machine‑precise (no ambiguity)
- Native to schema validation
- Used by Copilot manifests and actions
- Easier to test automatically
- Less error‑prone

JSON is the **canonical format** for schemas in agent systems and APIs.

**When JSON shines**

- Input/output contracts
- Action interfaces
- Validation
- Anything safety‑critical

### YAML — Optional (Human‑Friendly)

**Why YAML is tempting**

- Easier to read
- Less verbose
- Nice for documentation

**Why YAML is risky**

- Indentation errors
- Type ambiguity
- Multiple valid representations
- Harder to validate strictly

YAML is fine for **documentation**, but risky as the **source of truth**.

## Where Schemas Should Live (Reminder)

From your earlier structure:

```code
schemas/
├── calculation-input.schema.json
├── calculation-output.schema.json
└── decision-context.schema.json
```

Schemas belong:

- NOT in instructions
- NOT in knowledge
- Between instructions and actions

## Mental Model to Lock In

- **Instructions** → how to think
- **Schemas** → what is allowed
- **Actions** → how to compute
- **Knowledge** → what is true

If each stays in its lane, your agents stay robust.

<hr>

If you want, next we can:

- Design a schema checklist for new agents
- Convert one of your existing agents to schema‑first
- Create a schema + decision‑rule pairing template
- Practice spotting where schemas are missing

# User Prompt 23 - Example of Validation Rule and Schema

Give me specific example of how validation rule can utilize schema file?

# Agent Response 23

**The Scenario (Concrete & Realistic)**

You’re building an **Engineering Calculation Agent**.

Before doing any calculation, the agent must **validate user inputs**:

- Temperature
- Pressure

You want:

- No guessing
- No unit confusion
- Clear failure messages

## 1. The Schema File (What is allowed)

File: schemas/engineering-input.schema.json

```json
{
  "type": "object",
  "required": ["temperature", "pressure"],
  "properties": {
    "temperature": {
      "type": "number",
      "description": "Operating temperature",
      "minimum": -50,
      "maximum": 200
    },
    "pressure": {
      "type": "number",
      "description": "Operating pressure in kPa",
      "minimum": 0
    }
  }
}
```

**Learning takeaway**

This schema defines:

- **Which fields must exist**
- **What type they must be**
- **What ranges are acceptable**

The schema **does not decide what to do**.<br>
It only decides **what is valid**.

## 2. The Validation Rule (What the agent decides)

>**Instruction‑level decision rule** (human‑readable)

**Decision Rule: Input Validation Gate**

**Purpose**

- Ensure all required inputs are present and valid before calculation.

**Inputs Required**

- Temperature
- Pressure

**Evaluation Criteria**

- Inputs must conform to the **engineering input schema**.

**Decision Outcomes**

- **Valid Input**
    - Applies when: All inputs satisfy the schema
    - Action: Proceed to calculation
- **Invalid Input**
    - Applies when: Any input violates the schema
    - Action: Stop and report validation issues
**Failure Handling**

- Clearly explain which input failed and why.

Notice:<br>
The rule never checks numbers itself — it **delegates validation to the schema**.

## 3. How the Agent Uses the Schema (Step‑by‑Step)

**Step 1 — User provides input**

>“Temperature is 300 and pressure is 100”

**Step 2 — Schema validation runs first**

Schema detects:

- temperature = 300, exceeds maximum (200)

**Step 3 — Validation rule triggers**

Because the schema failed, the **Invalid Input** outcome applies.

**Step 4 — Agent response (clear & safe)**

```text
“The temperature value is outside the allowed range.
Accepted range: −50 to 200.
Please provide a corrected temperature.”
```

- No guessing
- No calculation
- Clear feedback

## 4. Why This Is Powerful (Learning Insight)

Without a schema:

- The agent might “adjust” 300
- Or ignore the issue
- Or produce a nonsense result

With a schema:

- Validation happens **before reasoning**
- The agent behaves predictably
- Errors are explainable and repeatable

This separation is exactly how **declarative agents and action contracts are designed**.

## 5. Another Example: Missing Field

**User input**

>“Temperature is 75”

**Schema result**

- Missing required field: **pressure**

**Validation rule outcome**

- **Invalid Input**

**Agent response**

```code
“Pressure is required but was not provided.
Please supply pressure in kPa.”
```

This is **schema‑driven clarity**, not prompt cleverness.

## 6. Key Pattern to Remember (Very Important)

>**Schemas answer: “Is this allowed?”**<br>
>**Decision rules answer: “What should I do next?”**

If you mix those, agents become brittle.

## 7. Where People Commonly Go Wrong

- Putting ranges in instructions
- Checking values with if statements in prompts
- Letting the model infer missing fields
- Writing “soft” validation rules

Schemas eliminate all of that.

## 8. Mental Model (Lock This In)

```code
User Input
   ↓
Schema Validation  ← (strict, mechanical)
   ↓
Decision Rule     ← (human-readable policy)
   ↓
Action / Response
```

This is the **cleanest agent architecture** you can build.

## Quick Reflection (Learning Check)

If I removed the schema:

- Would the agent still behave consistently?

If the answer is **no**, you’re using the schema correctly.

<hr>

If you want, next we can:

- Pair **decision rules + schemas** as reusable bundles
- Design **schema-driven error messages**
- Show **output validation schemas**
- Review one of your agents for missing schemas

# User Prompt 24 - Schema Identity

Does schema of an object has an identity that I can refer it in my validation rules. What if there are multiple object schemas in the agent, how do I reference a particular schema in my validation rule?

# Agent Response 24

**Short, clear answer first**

- **Yes — a schema *can* have an identity you can refer to in validation rules.**
- **You should explicitly give schemas stable identities.**
- **When you have multiple schemas, validation rules should reference the schema by that identity, not by content.**

The key insight:

> **Validation rules should name *which schema* applies, not restate validation logic.**

## 1. What “Schema Identity” Means (Conceptually)

A schema’s **identity** is a **stable name or identifier** that represents:

- the *shape* of an object
- the *validation contract*
- the *meaning* of the data

This identity can be expressed in three common ways:

- **Identity Type** : **Purpose**
- Schema ID : Canonical identifier
- Schema Name : Human‑readable reference
- Schema File Path : Physical location

You don’t need all three — but **you must have at least one stable identifier**.

## 2. Example: Give a Schema an Explicit Identity

**Schema file**

**File: schemas/engineering-input.schema.json**

```json
{
  "$id": "schema:engineering-input:v1",
  "type": "object",
  "required": ["temperature", "pressure"],
  "properties": {
    "temperature": { "type": "number" },
    "pressure": { "type": "number" }
  }
}
```

**Learning takeaway**

- **$id** is the **schema’s identity**
- It is **stable**
- It can be referenced **without re‑describing validation rules**

You are naming the *contract*, not the fields.

## 3. How a Validation Rule References a Schema

**Correct pattern (instruction‑safe)**

```txt
Validation Rule: Engineering Input Gate

Purpose:
Ensure user inputs conform to the Engineering Input schema.

Schema Reference:
- schema:engineering-input:v1

Behavior:
- Validate inputs against the referenced schema.
- If validation passes, proceed.
- If validation fails, report schema violations.
```

The rule never says:

- “temperature must be a number”
- “pressure is required”

That knowledge **belongs to the schema**.

## 4. What Happens When You Have Multiple Schemas

This is where schema identity really shines.

**Example: Multiple object schemas**

```txt
schemas/
├── engineering-input.schema.json
├── financial-input.schema.json
└── operational-context.schema.json
```

Each has its own identity:

- **Schema** : **Identity**
- Engineering Input : schema:engineering-input:v1
- Financial Input : schema:financial-input:v2
- Operational Context : schema:operational-context:v1

## 5. Validation Rule Selecting a Specific Schema

**Example: Context‑based validation**

```txt
Validation Rule: Input Schema Selection

Purpose:
Apply the correct schema based on task type.

Schema Mapping:
- Engineering calculation → schema:engineering-input:v1
- Cost analysis → schema:financial-input:v2

Behavior:
- Identify task type.
- Validate inputs against the mapped schema.
```

- No ambiguity
- No mixing of rules
- Clear audit trail

## 6. Important Rule: One Schema per Validation Pass

Avoid this:

> “Validate against engineering schema and financial schema”

Instead:

- Select **one schema**
- Validate against **one contract**

This keeps validation deterministic and explainable.

## 7. Where This Pattern Is Used (Why It’s Correct)

This pattern mirrors how:

- Declarative agent manifests reference schemas
- Action contracts are validated
- Enterprise systems separate **policy** from **structure**

As documented in microsoft-365-copilot-extensibility, **schemas act as machine‑readable contracts** that agents rely on for validation and orchestration

## 8.  Anti‑Patterns to Avoid

- Referring to schemas by filename in prose
- Duplicating schema rules inside instructions
- Letting instructions “partially validate”
- Applying multiple schemas simultaneously
- Changing schema meaning without versioning

## 9. Recommended Naming Convention (Simple & Effective)

```txt
schema:<domain>-<object>:<version>
```

Examples:

- schema:engineering-input:v1
- schema:risk-context:v2
- schema:calculation-output:v1

This works across:

- Git
- SharePoint
- Documentation
- Agent instructions

## Mental Model to Lock In

- **Schema identity = data contract**
- **Validation rule = policy that names the contract**
- **Instructions = orchestration**
- **Schemas = enforcement**

If your validation rule reads clearly *without opening the schema file*, you’ve done it right.

<hr>

If you want, next we can:

- Design a **schema registry** for your agents
- Create a **schema‑to‑decision‑rule mapping table**
- Show **versioning strategies** for schema evolution
- Review one of your agents for schema misuse

# User Prompt 25 - Schema Registry

Design a schema registry for my agents

# Agent Response

## Schema Registry for Agents (Learning‑First Design)

**What a Schema Registry Is (Plain Language)**

> A **schema registry** is a **single source of truth** that defines, names, versions, and governs all schemas used by your agents.

It answers four questions:

1. **What schemas exist?**
2. **What does each schema represent?**
3. **Which agent uses which schema?**
4. **Which version is active?**

Without a registry, schemas drift.<br>
With a registry, agents stay predictable.

## 1. Registry Design Principles (Lock These In)

- Schemas have **stable identities**
- Validation rules **reference schema IDs**, not file content
- One schema = one data contract
- Versions are explicit and immutable
- Humans can understand it without reading JSON

## 2. Logical Registry Structure

Think of the registry as three layers:

```txt
Registry
 ├─ Index (what exists)
 ├─ Schemas (what they validate)
 └─ Mappings (who uses what)
```

## 3. Registry Index (Human‑Readable Catalog)

**File: schema-registry.md** (or equivalent)

```txt
Schema Registry
===============

Schema ID: schema:engineering-input:v1
Purpose: Validate inputs for engineering calculations
Owner: Engineering Systems
Status: Active

Schema ID: schema:calculation-output:v1
Purpose: Validate calculation results returned by actions
Owner: Engineering Systems
Status: Active

Schema ID: schema:risk-context:v1
Purpose: Validate risk classification inputs
Owner: Compliance
Status: Active
```

**Why this matters**

- Reviewers don’t open JSON files
- Validation rules reference **Schema ID**
- Ownership is explicit

## 4. Schema Storage (Canonical Source)

**Folder: schemas/**

```txt
schemas/
├── engineering-input/
│   ├── v1.schema.json
│   └── README.md
│
├── calculation-output/
│   ├── v1.schema.json
│   └── README.md
│
└── risk-context/
    ├── v1.schema.json
    └── README.md
```

**README.md (per schema)**

```txt
Schema ID: schema:engineering-input:v1
Used by:
- Engineering Calculation Agent
- Validation Rule: Engineering Input Gate

Notes:
- Temperature in Celsius
- Pressure in kPa
```

This prevents tribal knowledge.

## 5. Schema Identity (The Critical Part)

Every schema **must declare its identity inside the file**.

**v1.schema.json**

```json
{
  "$id": "schema:engineering-input:v1",
  "type": "object",
  "required": ["temperature", "pressure"],
  "properties": {
    "temperature": { "type": "number" },
    "pressure": { "type": "number" }
  }
}
```

**Learning insight**

- **$id** = canonical identity
- File name ≠ identity
- Folder name ≠ identity

Agents and rules talk in **IDs**, not paths.

## 6. How Validation Rules Use the Registry

**Instruction‑level rule**

```txt
Validation Rule: Engineering Input Gate

Schema Reference:
- schema:engineering-input:v1

Behavior:
- Validate user input against the referenced schema.
- Proceed only on success.
```

- The rule never mentions fields
- The rule never mentions ranges
- The rule names the **contract**

## 7. Handling Multiple Schemas per Agent

**Pattern: Schema Selection Rule**

```txt
Validation Rule: Input Schema Selection

Schema Mapping:
- Engineering calculation → schema:engineering-input:v1
- Risk assessment → schema:risk-context:v1

Behavior:
- Identify task type
- Select schema
- Validate once
```

- One schema per validation pass
- Deterministic behavior

## 8. Versioning Strategy (Keep It Simple)

**Golden rules**

- **Never change a schema in place**
- **New meaning = new version**
- Old versions stay readable

Example:

- schema:engineering-input:v1
- schema:engineering-input:v2

Registry index marks which is **Active**.

## 9. Governance Checklist (Lightweight)

Before adding a schema:

- Does it have a clear purpose?
- Does it overlap with an existing schema?
- Is the identity unique?
- Is the owner named?

Before changing a schema:

- Is this a breaking change?
- Should this be a new version?

## 10. Mental Model to Lock In

- **Registry** = map
- **Schema** = contract
- **Validation rule** = policy that selects a contract
- **Agent** = orchestrator

If someone can answer *“Which schema validates this input?”* in 5 seconds, your registry is working.

## Why This Design Scales

- Supports multiple agents
- Supports hybrid declarative + action patterns
- Enables auditing and reviews
- Prevents silent breakage
- Aligns with enterprise governance practices

<hr>

If you want, next we can:

- Apply this registry to one of your real agents
- Design a **schema lifecycle flow** (draft → active → deprecated)
- Create a **schema‑rule‑agent traceability table**
- Review your current schemas for overlap or gaps