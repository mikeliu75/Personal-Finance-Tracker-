# Financial Tracker — Specification

## 1. Concept & Vision

A clean, personal finance tracker that transforms receipt images into structured spending data. The user uploads a receipt photo, the system extracts the key details, and the results are organized into a clear dashboard showing what was spent, when, and on what type. Designed for Hong Kong civil servants tracking daily expenses — functional, fast, no clutter.

---

## 2. Design Language

**Aesthetic direction:** Clean government-style utility — structured, readable, trustworthy. Think Inland Revenue department forms meets modern web app. White space, clear hierarchy, minimal decoration.

**Color palette:**
- Background: `#F5F6FA` (light grey-blue)
- Surface/Cards: `#FFFFFF`
- Primary: `#1A73E8` (professional blue)
- Secondary: `#34A853` (green for income/positive)
- Accent/Danger: `#EA4335` (red for expenses)
- Text primary: `#202124`
- Text secondary: `#5F6368`
- Border: `#E8EAED`

**Typography:**
- Headings: `Inter`, weight 600-700
- Body: `Inter`, weight 400
- Numbers/data: `JetBrains Mono` (monospace for amounts)
- Fallback: system-ui, sans-serif

**Spatial system:**
- Base unit: 8px
- Card padding: 24px
- Section gaps: 32px
- Border radius: 8px (cards), 4px (buttons/inputs)

**Motion philosophy:**
- Minimal and purposeful — 150ms ease-out for state changes
- No decorative animations
- Upload progress: smooth linear bar fill

---

## 3. Layout & Structure

```
┌─────────────────────────────────────────────────────────┐
│  Header: "Expense Tracker" + Total Spend this month      │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────────────┐  ┌─────────────────────────┐   │
│  │   Upload Receipt    │  │   Quick Stats           │   │
│  │   Drop zone / btn   │  │   Total / Count / Avg   │   │
│  └─────────────────────┘  └─────────────────────────┘   │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────────────┐  ┌─────────────────────────┐   │
│  │  Pie Chart          │  │  Per-Day Bar Chart       │   │
│  │  Spending by Type    │  │  Daily spending bars     │   │
│  └─────────────────────┘  └─────────────────────────┘   │
├─────────────────────────────────────────────────────────┤
│  Transaction List (sortable table)                      │
│  Date | Type | Description | Amount                     │
└─────────────────────────────────────────────────────────┘
```

**Responsive:** Single column on mobile, 2-column grid for charts on desktop.

---

## 4. Features & Interactions

### Upload Receipt
- Drag-and-drop zone or file picker
- Accepts: JPG, PNG, WebP
- On upload: show progress bar, then parse with MiniMax VL API
- On success: auto-populate form fields (editable before confirm)
- On error: show error message with retry option
- Supported receipt types: restaurant, retail, grocery, transport, entertainment

### Transaction Form
Fields:
- **Receipt** — file name (display only after upload)
- **Spending Type** — dropdown: Dining, Transport, Shopping, Entertainment, Utilities, Healthcare, Other
- **Date** — date picker, defaults to today
- **Amount** — number input (HKD), 2 decimal places
- **Description** — optional text field for notes

Submit: adds to transaction list, saves to localStorage, updates charts.

### Transaction List
- Shows all recorded transactions in reverse chronological order
- Columns: Date, Type, Description, Amount
- Sortable by clicking column headers
- Each row has a delete (×) button
- Amounts color-coded: red for all expenses (since all are expenses)

### Summary Charts
**Pie Chart — Spending by Type:**
- Categories as slices with distinct colors
- Legend with category names + percentages
- Hover shows category total
- Library: Chart.js (CDN)

**Bar Chart — Per-Day Spending:**
- X-axis: dates in current view range
- Y-axis: total spend per day (HKD)
- Bars color-coded by dominant spending type that day
- 7-day rolling window by default

### Export
- "Export JSON" button at top
- Downloads `transactions_YYYY-MM-DD.json` with full transaction list + summary
- Format: `{ transactions: [...], summary: { byType: {...}, byDay: {...} }, exportedAt: "..." }`

### Data Persistence
- All transactions stored in `localStorage` as JSON
- Key: `expense_tracker_transactions`
- Loaded on page start; saved on every add/delete

---

## 5. Component Inventory

### Upload Drop Zone
- Default: dashed border `#E8EAED`, icon + "Drop receipt here or click to upload"
- Drag-over: border `#1A73E8`, background `#E8F1FD`
- Uploading: progress bar inside, file name shown
- Success: green checkmark, file name
- Error: red border, error message, "Try Again" button

### Transaction Row
- Default: white background, bottom border
- Hover: slight grey background `#F8F9FA`
- Delete button (×): appears on hover, red on hover

### Stat Card
- White card, subtle shadow
- Large number (monospace), label below
- Icon left-aligned

### Buttons
- Primary: `#1A73E8` background, white text, 8px radius
- Secondary: white background, `#1A73E8` border + text
- Disabled: `#E8EAED` background, `#9AA0A6` text

### Charts
- White card background with 24px padding
- Chart.js with custom colors matching palette
- Responsive, aspect ratio maintained

---

## 6. Technical Approach

**Stack:** Single HTML file — vanilla JS, CSS, Chart.js (CDN)

**Receipt parsing:** MiniMax VL API via `analyze_image.js` script → extracted text → form auto-fill

**Data model:**
```json
{
  "transactions": [
    {
      "id": "TRX001",
      "date": "2026-04-18",
      "spendingType": "Dining",
      "description": "IPPUDO (Queensway Plaza)",
      "amount": 268.00,
      "receiptRef": "receipt_20260418.jpg",
      "createdAt": "2026-05-24T03:25:00Z"
    }
  ],
  "summary": {
    "total": 268.00,
    "byType": { "Dining": 268.00 },
    "byDay": { "2026-04-18": 268.00 }
  },
  "exportedAt": "2026-05-24T03:25:00Z"
}
```

**Spending types (with colors):**
- Dining: `#EA4335` (red)
- Transport: `#FBBC04` (yellow)
- Shopping: `#34A853` (green)
- Entertainment: `#9334E6` (purple)
- Utilities: `#1A73E8` (blue)
- Healthcare: `#00ACC1` (cyan)
- Other: `#9E9E9E` (grey)

**Key functions:**
- `parseReceipt(imageData)` — calls VL API, returns structured data
- `addTransaction(tx)` — validates, saves to localStorage, re-renders
- `deleteTransaction(id)` — removes from list, re-renders
- `renderCharts()` — updates pie + bar charts with current data
- `exportJSON()` — generates and downloads JSON file

**localStorage schema:**
```json
{
  "version": 1,
  "transactions": [...]
}
```