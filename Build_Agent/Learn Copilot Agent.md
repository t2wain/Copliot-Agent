# Declarative Agent in Copilot

Learning to build Declarative Agent in Copilot by asking Copilot for the instruction.

# User Prompt 1

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

# User Prompt 2

Provide an example how I can specify a simple Python function code in my prompt

# Agent Response 2

When you “specify a Python function in a prompt” for M365 Copilot, you are not embedding executable Python.

Instead, you are:

- Describing a function contract
- Allowing Copilot to reason about when to call it
- Expecting a pure, side‑effect‑free computation

Think of it as declaring a calculator, not writing a script.

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

# User Prompt 3

Provide an example how to refer to a file stored in SharePoint within my prompt.

# Agent Response 3

## Core Learning Concept (very important)

- **You never “load” a SharePoint file in a prompt.**
- You **reference** it so Copilot knows what to retrieve during reasoning.

Copilot uses **file references as retrieval hints**, not as memory or attachments.

## Pattern 1

**Referencing a File by Name (Most Common)**

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

## Pattern 2

**Referencing a File by Location (More Precise)**

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

## Pattern 3

**Referencing a File as Context, Not Truth**

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
- Teaches the agent to separate source facts from analysis
- Critical for compliance or audit scenarios

**Key Ideas**

- Advanced prompt hygiene
- Shows maturity in agent design

## Pattern 4

**Conditional File Usage (Best‑in‑Class)**

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

# User Prompt 4

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

## Example 1

**Combined Agent Prompt (File + Python Function)**

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

# User Prompt 5

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

# User Prompt 6

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

# User Prompt 7

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

# User Prompt 8

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

# User Prompt 9

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

# User Prompt 10

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

# Uer Prompt 11

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

# User Prompt 12

How to clearly separate instruction prompt from knowlege base?

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

# User Prompt 13

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

# User Prompt 14

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