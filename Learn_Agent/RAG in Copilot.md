# User Prompt 1 - RAG in Copilot

When I uploaded multiple documents to OneDrive so that Copilot can have access to these document, does Copilot employed a RAG process to only include relevant information into the context window?

# Agent Response 1

Short answer: **yes**—when you upload multiple documents to OneDrive and use Microsoft 365 Copilot, Copilot **does not load all documents into the model’s context window**. Instead, it uses a **Retrieval‑Augmented Generation (RAG)** pipeline to dynamically retrieve **only the most relevant portions** of your files for each question.

Below is a clear, mental‑model view of how this works specifically with **OneDrive / Microsoft 365 Copilot**.

## High‑level flow (what happens under the hood)

When you ask Copilot a question that could involve documents stored in OneDrive:

1. **Your prompt is analyzed**
    - Copilot interprets intent and rewrites the query into optimized search terms.
2. **Relevant content is retrieved (RAG)**
    - Copilot searches across your OneDrive (and other M365 data you have access to) using - Microsoft Graph + semantic indexing.
    - Only **top‑ranked, relevant chunks** of documents are selected—not entire files.
3. **Context is constructed**
    - The retrieved snippets are injected into the LLM prompt alongside:
        - System instructions
        - Your original user message
        - Conversation context
4. **Response generation**
    - The model produces an answer *grounded in the retrieved content*.
    - Source grounding and permission checks are enforced.
5. **Nothing is permanently “loaded”**
    - Each prompt performs retrieval again; documents are **not memorized** or fully cached in the - context window.

This is the classic **enterprise RAG pattern**.

## Why Copilot must use RAG (practical reasons)

### 1. Context window limits

Even though Copilot can reference very large files (hundreds of pages), **the LLM’s working context is still finite**. Microsoft explicitly avoids inserting entire documents at once.

> Large documents are chunked and retrieved selectively so that only relevant excerpts are provided to the model.

### 2. Scale: thousands of files

OneDrive can contain:

- Thousands of documents
- Hundreds of megabytes
- Different formats (DOCX, PDF, PPT, XLSX)

Without RAG, this would be impossible to process efficiently.

### 3. Security & permissions

Copilot retrieves data via Microsoft Graph, so:

- It only accesses files **you are allowed to see**
- Results are filtered by identity and permissions at query time

This is critical in enterprise environments.

## What is actually retrieved?

Copilot does not retrieve:

- Entire documents
- All uploaded files
- Random content from OneDrive

It **does** retrieve:

- Semantically relevant **snippets or chunks**
- Often just a few paragraphs per file
- Usually from **multiple documents**, if relevant

Microsoft Learn explicitly describes this RAG process as:

> Query rewriting → content retrieval → summarization → safety validation

## Important clarification: uploading ≠ preloading

Uploading documents to OneDrive does not mean:

- “Copilot now has all these files in its context window”

What it actually means:

- “Copilot can *retrieve from these files when needed*”

Think of OneDrive as a **searchable knowledge source**, not expanded model memory.

## Example

If you upload:

- 20 project design documents
- 10 meeting summaries
- 5 budget spreadsheets

And you ask:

> “What risks were identified for Phase 2?”

Copilot will:

- Search all 35 files
- Pull only passages related to “risks” and “Phase 2”
- Inject maybe **2–10 small text chunks** into the prompt
- Generate a grounded answer

This behavior is explicitly described as RAG in Microsoft Copilot’s architecture.

## OneDrive Agents and explicit knowledge bases

If you create a **OneDrive Agent** (Copilot Agents / Copilot Studio):

You are still using RAG
You just **narrow and formalize the retrieval scope**
Retrieval is more precise, but the mechanism is unchanged

So even agents do not bypass the context window—they just improve retrieval quality.

## Summary

**Yes—Microsoft 365 Copilot uses RAG with OneDrive documents.**

- **Aspect** : **Behavior**
- All documents loaded? : No
- Relevant snippets retrieved dynamically? : Yes
- Context window bounded? : Always
- Permissions enforced at retrieval time? : Yes
- Content reused across turns? : Re‑retrieved per prompt

<hr>

If you want, I can:

- Compare Copilot RAG vs “manual” RAG systems you may build with Azure OpenAI
- Explain how citation grounding works in Copilot answers
- Walk through what does get remembered across a conversation (vs what doesn’t)

# User Prompt 2 - Citation

Explain how citation grounding works in Copilot answers

# Agent Response 2

## What “citation grounding” means in Copilot

Citation grounding means that:

> Copilot’s answer is explicitly anchored to specific pieces of enterprise content (files, emails, chats, pages), and those anchors are surfaced to you as clickable citations.

In practice, this is how Microsoft ensures Copilot is not “free‑hallucinating,” but instead:

- retrieves trusted content,
- generates an answer from it, and
- shows you where each claim came from.

## The lifecycle of a grounded, cited Copilot answer

Think of citation grounding as **four tightly coupled stages**.

### 1. Retrieval: selecting candidate sources (RAG starts here)

Before Copilot generates any text:

- Your prompt is **rewritten and optimized** for retrieval
- Microsoft Graph + semantic index search your M365 data (OneDrive, SharePoint, Teams, - Outlook, etc.)
- The system retrieves:
    - a small number of **high‑relevance chunks**
    - from one or more documents
    - each chunk carries **metadata** (file ID, location, permissions)

Important:

- Citations are **only possible** for content retrieved during this step.
- If something is not retrieved, it cannot be cited later.

### 2. Context injection: marking sources as “grounding content”

The retrieved snippets are injected into the LLM prompt **with structure**, not as plain text.

Conceptually, the model sees something like:

```txt
[System instructions]

[Grounding source #1]
Source ID: S1
Document: Budget_FY26.xlsx
Excerpt: "...Phase 2 costs are projected at $4.2M..."

[Grounding source #2]
Source ID: S2
Document: Risk_Register.docx
Excerpt: "...Primary delivery risk is vendor lead time..."

[User prompt]
"What risks could impact Phase 2 delivery?"
```

Key points:

- Each snippet is **labeled with a source ID**
- The model is instructed to **use these sources preferentially**
- The model is told to **track which source supports which statement**

This is a Microsoft‑specific extension on top of standard RAG.

### 3. Generation with attribution tracking

As the LLM generates the answer, it performs **implicit attribution tagging**:

- For each sentence (or clause), the model determines:
    - “Which grounding source supports this?”
- The system records:
    - Sentence → Source ID mapping

There are three possible outcomes for each statement:

- **Outcome** : **What Copilot does**
- Supported by retrieved source : Generates statement + citation
- Supported by multiple sources : Generates statement + multiple citations
- Not supported : Either omits it or labels it as general knowledge (often uncited)

This step is why Copilot answers sometimes:

- avoid strong claims
- phrase uncertainty explicitly
- omit guesses that cannot be grounded

### 4. Post‑processing: citation rendering

After generation:

- Source IDs are translated into human‑readable citations
- Citations are rendered as:
    - document name
    - page / slide / section (when available)
    - clickable link (respecting permissions)

Example output:

> Vendor lead time is a key risk for Phase 2 and could delay delivery.<br>
> **(Risk_Register.docx)**

Behind the scenes:

- The citation is not added manually
- It is emitted by the system based on that attribution map

## Why citations sometimes feel “coarse”

You may notice:

- A citation applies to a whole paragraph
- Not every sentence has its own marker

This is intentional:

- Copilot uses block‑level grounding, not token‑level
- It groups adjacent statements supported by the same source
- This improves readability and reduces citation noise

## When Copilot will not show citations

Even in Microsoft 365 Copilot, citations are conditional.

You typically **won’t see citations** when:

1. **Answer is model general knowledge**
    - e.g. “What is critical path in project management?”
2. **Instructional or creative tasks**
    - “Rewrite this email more politely”
    - “Create an agenda”
3. **Reasoning over retrieved content**
    - The retrieved data is used for thinking, but not quoted directly
    - Some intermediate reasoning steps are not cited
4. **The retrieval confidence is low**
    - Copilot may hedge or respond with uncertainty instead

## Why citations increase answer quality (and safety)

Citation grounding enables several enterprise safeguards:

**Factual anchoring**

The model is constrained to:
- say only what retrieved sources reasonably support
- avoid confident hallucinations

**Permission enforcement**
- You only ever see citations to content you already have access to
- Even the existence of restricted docs is hidden

**Auditing & trust**

Users can:

- click through
- validate claims
- challenge incorrect interpretations

This is why Copilot is accepted in regulated environments.

## Common misconceptions (clearing these up)

**“Copilot reads the whole document and then cites it”**

No.

- Only retrieved chunks are read
- Citations reflect those chunks, not the entire file

**“Citations are added after the answer is written”**

No.

- The model generates with awareness of source IDs
- Citations are integral, not cosmetic

**“If a document is cited, it means Copilot fully understands it”**

No.

- Citations mean relevance, not completeness
- Missing nuance often means the relevant chunk was not retrieved

## How to improve citation quality as a user

If you care about citations being precise and useful:

1. **Ask source‑aware questions**
    - “Based on the risk register…”
    - “According to the Phase 2 design doc…”
2. **Narrow scope**
    - Fewer documents → better grounding
3. **Ask for confirmation**
    - “Which document supports this?”
    - “Cite the source for each risk”

These prompts encourage tighter retrieval and attribution.

## One‑sentence takeaway

> **Copilot citations are produced by tracking which retrieved document snippets directly support each generated statement, and only those sources are attached as citations in the final answer.**

If you want, next I can:

- Walk through a concrete OneDrive example step‑by‑step
- Explain differences between Copilot citations vs academic citations
- Compare Copilot grounding vs custom RAG pipelines in Azure OpenAI

Just tell me how deep you want to go.