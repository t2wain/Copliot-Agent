## Task

Analyze the provided drawing document and extract **revision history entries** exclusively from the **drawing title block revision history table**.

---

## Definition: Drawing Revision History Table

The drawing revision history is a structured table located within the **title block at the bottom of the drawing**.

The table contains **exactly eight (8) columns**, ordered left-to-right as follows:

1. **REV** → `rev-no`  
2. **DATE** → `rev-date`  
3. **BY** → `rev-by`  
4. **DESCRIPTION** → `description`  
5. **CHK** → `checked-by`  
6. **ENGR** → `engr-by`  
7. **APPR** → `approve-by`  
8. **DATE** → `approved-date`

Additional rules:
- Revisions are recorded **from bottom to top** (earliest at bottom, latest at top).
- Each row corresponds to **one revision entry**.
- Each revision entry **must conform** to `schema:revision:v1`.

### Visual Column Alignment Rule (Strict)

Each revision row MUST be parsed strictly by **visual left‑to‑right column position** as rendered in the drawing.

- Column boundaries are determined by the table grid or visual alignment, not by text content.
- Each cell maps to exactly one column based solely on its horizontal position.
- Blank cells do NOT shift values into adjacent columns.
- Do NOT realign, merge, or reposition values due to missing or empty cells.

### Blank Cell Handling

If a column cell is visually present but contains no text:
- Output the corresponding field with an empty string (`""`)
- Do NOT omit the field
- Do NOT infer or borrow values from other rows

### Non‑Inference Constraint

Do NOT infer column membership from semantics, labels, or neighboring rows.
Only the visual column position determines field assignment.
Column membership is determined ONLY by visual horizontal position; content meaning is irrelevant.

---

## Extraction Scope (Strict)

✅ Extract data **only** from the title block revision history table.

❌ Ignore all other content, including but not limited to:
- Geometry, views, or models
- Dimensions or annotations
- Notes, callouts, or legends
- BOMs or parts lists
- Revision clouds or tables outside the title block
- Logos, stamps, watermarks, or branding elements unless explicitly inside labeled title block fields

---

## Output Contract (Strict Enforcement)

- Output **exactly one JSON object** per drawing file
- Output **must strictly conform** to `schema:revision-history:v1`
- Do **not** include:
  - Comments
  - Explanations
  - Confidence scores
  - Inferred, derived, or additional fields
- Do **not** alter field names or schema structure

**Any deviation from this contract invalidates the response.**

---

## Schemas

### `schema:revision:v1`

```json
{
  "$id": "schema:revision:v1",
  "type": "object",
  "required": ["rev-no", "rev-date", "rev-by"],
  "properties": {
    "rev-no": { "type": "string" },
    "rev-date": { "type": "string", "format": "date" },
    "rev-by": { "type": "string" },
    "description": { "type": "string" },
    "checked-by": { "type": "string" },
    "engr-by": { "type": "string" },
    "approve-by": { "type": "string" },
    "approved-date": { "type": "string", "format": "date" }
  },
  "additionalProperties": false
}
```

### `schema:revision-history:v1`

```json
{
  "$id": "schema:revision-history:v1",
  "type": "object",
  "required": ["file-name", "revisions"],
  "properties": {
    "file-name": { "type": "string" },
    "revisions": {
      "type": "array",
      "items": { "$ref": "schema:revision:v1" }
    }
  },
  "additionalProperties": false
}
```