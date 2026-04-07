When providing the information about a specific circuit design, you will return the recommended cable size result by using the lookup tables in the attached Excel spreadsheet. 

# Explanation about the data in the Excel spreadsheet.

The attached Excel file contains several lookup tables to select the smallest cable size for an electrical circuit based on various design parameters.

Each lookup table contains the following information that specify the common design parameters of the electrical circuit:

- Supply voltage (ex. 480V)
- Load Size Unit (ex. Ampere, kW, kVA, or hp)
- Cable Length Unit (ex. Meter, Feet)

The table entries are based on these 3 parameters

1. The table row specify the load sizes (values of first table column ordered from top-to-bottom and small-to-large). The row number starts at 1 with the first load size value.
2. The table column specify the cable size (values of first table row ordered from left-to-right and small-to-large). The column number started at 1 with the first cable size value.
3. the table cell specifies the cable length. For each row, the cable length is ordered from left-to-right and small-to-large. All blank cells are ignored.

## Step 1. Select the Proper Worksheet (Do Not Skip)

Choose the lookup table based on the type of electrical load based on the common parameter:

- Load unit (Ampere, kW, kVA, or hp)
- Length unit (Meters or Feet)
- Supply voltage (e.g. 400 V, 480 V)

These parameters determine which table is valid.
Do **not** interpolate between tables.

Use one table only
- No interpolation
- No switching tables midway

## Step 2. Select the Correct Load Row

- Look at the ordered load column (first column)
- Select the **first row whose load value is ≥ the given load**
- **You MUST remember the table row number.**

Examples:

- given 30 kW -> select 30 kW (**first** load value that is **equal to or greater** than the given load)
- given 32 A -> select 35 A (**first** load value that is e**qual to or greater** than the given load)
- given 5.5 kW -> select 5.5 kW (**first** load value that is **equal to or greater** than the given load)

## Step 3. Select Cable Length CELL

- Stay in **the same selected row number**. Do **not** jump rows. Scan **left‑to‑right**. Ignore all blank cell. Select **the first cell value ≥ the given cable length**
- Do **not** interpret header meanings
- **You MUST remember the column number** of the selected cable length cell.

## Step 4. Return Result

Return **only**:

- Worksheet used
- Selected load size and its row number.
- Selected cable size and its column number.
- Selected cable length in the table cell which is the intersection of row number and column number.

**Emphasize** the selected load size and its row number. 
**Emphasize** the selected cable size and its column number.
**Emphasize** the selected cable length.

# Workout Examples below

Example 1, given the circuit design:
- Load Size Unit: Amperes
- Supply voltage: 480V
- Load Size: 32 Amp
- Cable length: 500 m

Answer:
- Worksheet : **Feeder**
- Selected Load size: **35 amp**, Row Number: **7**
- Selected Cable Size: **2 AWG**, Column Number: **6**
- Selected Cable length: **621 m**

Example 2, given the circuit design:
- Load Size Unit: kW
- Supply voltage: 400V
- Load Size: 10 kW
- Cable length: 200 m

Answer:
- Worksheet : **Motor**
- Selected Load Size: **11 kW**, Row number : **11**
- Selected Cable Size: **10 mm2**, Column number : **4**
- Selected Cable length: **275 m**

# Provide the answer for these given circuit designs:

Provide answers only. Don't show intermediate reasoning.

Given a circuit design:
- Load Size Unit: Ampere
- Supply voltage: 480V
- Load Size: 130 Amp
- Cable length: 600 m

Answer:


Given a circuit design:
- Load Size Unit: kW
- Supply voltage: 400 V
- Load Size: 25 kW
- Cable length: 600 m

Answer: