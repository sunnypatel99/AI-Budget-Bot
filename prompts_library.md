# AI Prompt Library

This document contains all Claude AI prompts used in the AI Budget Bot system, including prompt engineering strategies and lessons learned during development.

---

## Table of Contents

1. [Transaction Extraction Prompt](#1-transaction-extraction-prompt)
2. [Weekly Budget Report Prompt](#2-weekly-budget-report-prompt)
3. [Monthly Budget Report Prompt](#3-monthly-budget-report-prompt)
4. [Prompt Engineering Best Practices](#prompt-engineering-best-practices)
5. [Lessons Learned](#lessons-learned)

---

## 1. Transaction Extraction Prompt

**Purpose:** Extract transaction details from credit card notification emails and categorize spending automatically.

**Workflow:** Daily Transaction Processing

**Model:** Claude Sonnet 4

**Input Variables:**
- `$json.Subject` - Email subject line
- `$json.snippet` - Email body content

### Prompt:

```
Extract transaction details from this credit card notification email. Return ONLY valid JSON with no explanations.

Email Subject: {{ $json.Subject }}
Email Content: {{ $json.snippet }}

Required JSON format:
{
  "transaction_date": "YYYY-MM-DD",
  "merchant": "merchant name",
  "amount": 0.00,
  "category": "pick ONE from list"
}

Category must be EXACTLY one of these:
Housing-Rent, Housing-Utilities, Housing-Internet, Groceries, Dining, Transportation-Gas, Transportation-Rideshare, Transportation-Maintenance, Healthcare, Personal Care, Shopping-Clothing, Shopping-Electronics, Shopping-Home, Shopping-General, Entertainment, Subscriptions, Travel, Education, Gifts, Other

Rules:
- For transaction_date, use TODAY'S date in YYYY-MM-DD format
- Extract merchant from subject line (clean up truncated names, e.g., 'CLAUDE.AI SUBSCRIPTI' becomes 'Claude AI')
- Extract amount from subject line as a number
- Match category based on merchant (e.g., Claude AI subscription = Subscriptions)
- If unclear category, use Other
```

### Key Design Decisions:

1. **Exact Category List**: Provides exhaustive list to prevent inconsistent naming (e.g., "Groceries" vs "Grocery Store")
2. **JSON-Only Output**: Explicitly requests no explanatory text to simplify parsing
3. **Merchant Name Cleaning**: Instructs AI to clean truncated merchant names common in email notifications
4. **Fallback Category**: Uses "Other" for unclear transactions instead of guessing
5. **Date Strategy**: Uses current date since email arrival closely matches transaction date for credit cards

### Post-Processing:

After receiving Claude's response, a Code node:
- Removes markdown formatting (```json blocks)
- Parses JSON
- Validates required fields
- Converts Gmail's `internalDate` to proper YYYY-MM-DD format
- Ensures amount is numeric

---

## 2. Weekly Budget Report Prompt

**Purpose:** Generate intelligent weekly budget analysis with spending insights and recommendations.

**Workflow:** Weekly Finance Report (runs every Sunday)

**Model:** Claude Sonnet 4

**Input Variables:**
- Budget targets (calculated in Code node based on paycheck count)
- Pre-calculated spending totals (NOT calculated by AI)
- Transaction details for context
- Date context (day of month, days remaining)

### Prompt:

```
You are a personal finance analyst. Generate a weekly finance report.

**Budget Context:**
- Gross weekly income: $X
- Auto-deducted (not in Airtable): $Y/week ($Z savings + $Z investments)
- Remaining for tracking: $X/week
- This is a [4 or 5]-paycheck month
- Fixed housing costs: $X/month (Rent $X + Internet $X + Utilities ~$X + Water ~$X)
- These are tracked but excluded from non-housing budget analysis

**Weekly Budget (Non-Housing Necessities):**
- Necessities (food, gas, healthcare, etc.): $X/week
- Guilt-free spending: $X/week

**Monthly Budget (Non-Housing Necessities):**
- Necessities: $X/month
- Guilt-free: $X/month

**PRE-CALCULATED SPENDING (Use these exact numbers - do NOT recalculate):**

**Last 7 Days:**
- Necessities (non-housing): $X
- Guilt-free: $X
- Housing: $X
- Total transactions: X

**Month-to-Date:**
- Necessities (non-housing): $X
- Guilt-free: $X
- Housing: $X
- Total transactions: X

**Context:**
- Day X of the month
- X days remaining in month
- Report date: [date]

**Transaction Details (for context only - amounts already calculated above):**
Last 7 Days: [JSON array of transactions with merchant, category, amount]

Month-to-Date: [JSON array of transactions with merchant, category, amount]

Generate a report with TWO sections using the PRE-CALCULATED numbers above (do not add/calculate yourself):

**SECTION 1 - Last 7 Days Review:**
1. Report the exact spending: Necessities $X vs $X budget, Guilt-free $X vs $X budget
2. Calculate variance (over/under budget)
3. Highlight 3-5 specific transactions with merchant names and amounts (e.g., "Taco Bell $3, Chick-fil-A $1.59"), describe spending patterns, but do NOT make lifestyle assumptions (e.g., don't assume rideshare = no car), do not create totals

**SECTION 2 - Month-to-Date Progress:**
1. Report the exact spending: Necessities $X vs $X budget, Guilt-free $X vs $X budget
2. Calculate remaining budget
3. Pace analysis: "You're X days into the month, should have spent ~$X based on daily pace, actually spent the amounts above"
4. On track / overspending / underspending assessment
5. Highlight few notable transactions with merchant names and amounts, describe patterns without making lifestyle assumptions

Note housing separately if any charges this period.

Format with emojis, bold text, clear sections. Be direct and analytical. IMPORTANT: Use ONLY the pre-calculated numbers provided at the top - do NOT attempt to add, sum, or calculate any transaction amounts yourself. For patterns and observations, describe what you see without calculating new totals. Keep total report under 1800 characters.
```

### Key Design Decisions:

1. **Pre-Calculated Totals**: All math done in Code node before Claude sees it (LLMs are bad at math)
2. **Dual Time Periods**: Shows both weekly snapshot AND month-to-date progress
3. **Explicit Math Warning**: Multiple reminders NOT to calculate totals
4. **Transaction Highlighting**: Encourages specific examples (merchant + amount) for context
5. **No Lifestyle Assumptions**: Prevents incorrect inferences (e.g., Uber usage ≠ no car)
6. **Character Limit**: Discord has 2000 char limit, so 1800 leaves buffer
7. **Direct Tone**: "No sugar coating" ensures honest budget feedback

### Budget Calculation Strategy:

Weekly and monthly budgets adjust based on paycheck count:
- 4-paycheck months: Lower weekly/monthly targets
- 5-paycheck months: Higher targets (extra paycheck cushion)

This prevents unfair budget pressure in shorter months.

---

## 3. Monthly Budget Report Prompt

**Purpose:** Comprehensive monthly financial analysis with spending patterns and actionable recommendations.

**Workflow:** Monthly Finance Report (runs on 1st of each month)

**Model:** Claude Sonnet 4

**Input Variables:**
- Previous month's name
- Paycheck count for that month
- Pre-calculated category totals
- Full transaction list for pattern analysis

### Prompt:

```
You are a personal finance analyst. Generate a monthly finance report for [Month Name].

**Income & Budget for [Month Name]:**
- Gross income: X paychecks × $X = $X
- Auto-deducted (not in Airtable): $X ($X/week savings + $X/week investments)
- Remaining for tracking: $X
- Fixed housing costs: $X (Rent $X + Internet $X + Utilities ~$X + Water ~$X)
- Non-housing necessities budget: $X (60% of income minus housing)
- Guilt-free budget: $X (20% of income)

**PRE-CALCULATED SPENDING (Use these exact numbers - do NOT recalculate):**
- Housing: $X (expected ~$X)
- Necessities (non-housing): $X (budget: $X)
- Guilt-free: $X (budget: $X)
- Total transactions: X

**Transaction Details (for context only - amounts already calculated above):**
[JSON array of all transactions with merchant, category, amount]

Generate a comprehensive monthly report using the PRE-CALCULATED numbers above (do not add/calculate yourself):

1. Housing costs: Report $X vs expected $X
2. Non-housing necessities: Report $X vs $X budget
3. Guilt-free spending: Report $X vs $X budget
4. Total budget performance summary (total over/under)
5. Top 3-5 notable merchants or observations from transaction list (describe patterns, do NOT calculate totals)
6. Biggest wins and problem areas
7. Actionable recommendations for next month

Format with emojis, bold text, clear sections. Be direct and analytical - no sugar coating. Use ONLY the pre-calculated numbers provided at the top. Do NOT attempt to add, sum, or calculate any transaction amounts yourself - only use the totals already given. For merchant patterns, describe what you observe without calculating new totals. Keep total report under 1800 characters.
```

### Key Design Decisions:

1. **Variable Income Handling**: Budget adjusts for 4 vs 5 paycheck months automatically
2. **Housing Separation**: Fixed housing costs tracked separately from variable necessities
3. **Performance Summary**: Requires both category breakdown AND total variance
4. **Pattern Analysis**: Asks for merchant-level insights without doing new calculations
5. **Forward-Looking**: Includes actionable recommendations for next month
6. **Wins & Problems**: Balances positive reinforcement with problem identification

---

## Prompt Engineering Best Practices

### 1. **Don't Rely on LLMs for Math**

**Problem:** Large language models approximate numbers rather than calculating precisely.

**Solution:** Pre-calculate all totals, sums, and variances in Code nodes. Only ask Claude to:
- Report the numbers you give it
- Calculate simple variance (provided number - budget)
- Analyze patterns (qualitative, not quantitative)

---

### 2. **Be Explicit About Category Mapping**

**Problem:** Without exact categories, AI creates variations ("Groceries" vs "Grocery Shopping" vs "Food - Groceries").

**Solution:** Provide exhaustive list with exact spelling requirements.

**Implementation:**
- List all categories in prompt
- Require "EXACTLY one of these"
- Provide examples for ambiguous cases
- Include a catch-all ("Other") for unclear items

---

### 3. **Prevent Hallucinated Insights**

**Problem:** LLMs will confidently make assumptions based on limited data.

**Solution:** Explicitly forbid lifestyle assumptions and require fact-based observations.

**Example Instructions:**
- ✅ "Describe what was purchased"
- ✅ "Note patterns in the transaction list"
- ❌ Don't allow "You took Uber, so you don't own a car"
- ❌ Don't allow "Lots of fast food means you're stressed"

---

### 4. **Manage Output Length**

**Problem:** Discord messages have 2000 character limit. Claude can be verbose.

**Solution:**
- Set explicit character limit (1800 to leave buffer)
- Request "concise but comprehensive"
- Structure with sections to ensure key info isn't cut off
- Test and iterate on prompt to find balance

---

### 5. **Structured Output with JSON**

**Problem:** Parsing unstructured text responses is error-prone.

**Solution:** Request JSON-only output for data extraction tasks.

**Best Practices:**
- Show exact JSON schema required
- Request "ONLY valid JSON with no explanations"
- Provide example output
- Handle markdown formatting in post-processing (```json blocks)

---

### 6. **Context Window Management**

**Problem:** Large transaction lists can exceed context limits or add unnecessary cost.

**Solution:**
- For weekly reports: Send simplified transaction list (merchant, category, amount only)
- For monthly reports: Same approach - don't send entire database
- Pre-calculate aggregates before sending to AI
- Balance detail with token efficiency

---

## Lessons Learned

### 1. **Iteration is Key**

Initial prompts were too trusting of AI's math abilities. After discovering incorrect totals, pivoted to pre-calculation strategy.

### 2. **Specificity Prevents Ambiguity**

Early prompts like "categorize this transaction" led to 30+ unique category variations. Explicit category list reduced this to zero variance.

### 3. **Character Limits Matter**

First reports exceeded Discord's limit. Adding "under 1800 characters" reduced message failures from ~40% to 0%.

### 4. **Direct Tone Works Better**

Requesting "encouraging but honest" led to sugar-coated reports. Switching to "direct and analytical - no sugar coating" produced more actionable insights.

### 5. **Transaction Examples Add Value**

Reports that only showed totals felt generic. Adding "highlight 3-5 specific transactions" made reports feel personalized and actionable.

### 6. **Separate Data from Analysis**

Initially sent all raw transaction data and asked Claude to calculate AND analyze. Separating these into:
1. Code node calculates
2. Claude analyzes pre-calculated numbers

...improved both accuracy and report quality.

---

## Prompt Templates for Adaptation

### Basic Transaction Extraction Template

```
Extract [data type] from this [source]. Return ONLY valid JSON.

Input: {{ variable }}

Required JSON format:
{
  "field1": "type",
  "field2": 0.00
}

[Field] must be EXACTLY one of these:
[List all valid options]

Rules:
- [Specific extraction rules]
- [Edge case handling]
- [Fallback behavior]
```

### Basic Analysis Report Template

```
You are a [role]. Generate a [report type].

**Context:**
- [Key background information]
- [Budget/target information]

**PRE-CALCULATED DATA (Use these exact numbers):**
- [Metric 1]: $X
- [Metric 2]: $X

**Raw Data (for context only):**
[Simplified data structure]

Generate a report with:
1. [Required section 1]
2. [Required section 2]
3. [Required section 3]

Format: [formatting requirements]
Tone: [tone requirements]
Length: [character/word limit]
IMPORTANT: [Critical constraints]
```

---

## Future Prompt Enhancements

Potential improvements being considered:

1. **Trend Analysis**: Add month-over-month comparison prompts
2. **Predictive Insights**: "Based on current pace, projected end-of-month spending"
3. **Seasonal Adjustments**: Recognize holidays, travel months, etc.
4. **Goal Tracking**: Incorporate savings goals and progress
5. **Anomaly Detection**: Flag unusual spending patterns automatically

---

**Note:** All prompts use Claude Sonnet 4 (`claude-sonnet-4-20250514`). Other models may require prompt adjustments for optimal performance.
