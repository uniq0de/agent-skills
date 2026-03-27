# MCP Tools Development: Complete Guide to Best Practices


## Understanding Tools for Agents {#understanding}

### What Makes Tools Different?

You're not writing APIs for developers—you're designing interaction surfaces for AI agents with limited context and different affordances.

### The Core Challenge

Agents have **limited context** (unlike abundant computer memory). A tool returning ALL contacts forces the agent to read token-by-token through irrelevant information—a waste of precious context space.

**Better approach**: Design `search_contacts` instead of `list_contacts`

---

## Fundamental Design Philosophy

### 1. **Think Like an Agent, Not a Developer**

```
WRONG: Create separate tools for each micro-action
   - list_users
   - list_events  
   - create_event

RIGHT: Consolidate into high-value composite tools
   - schedule_event (finds availability + schedules + reserves room)
```

### 2. **Fewer, More Powerful Tools > Many Specialized Tools**

- Reduces selection ambiguity
- Decreases context overhead
- Enables complex workflows in single tool calls
- Tools should consolidate related multi-step operations

### 3. **Prioritize High-Impact Workflows**

Focus on tools that match real evaluation tasks and genuine use cases—not theoretical scenarios.

---

## Development Workflow {#workflow}

0. Develop tools
1. Generate realistic evaluation tasks (20+ examples)
2. Create verifiable outputs/ground truth
3. Run agents through evaluation loop
4. Collect metrics:
   - Task accuracy
   - Token consumption
   - Tool call count
   - Error rates
   - Runtime


**Strong Evaluation Tasks**:
- "Customer ID 9182 was charged 3x. Find logs, assess impact scope, compile evidence."
- "Schedule meeting for Jane next week, attach project notes, reserve conference room."
- "Sarah Chen cancelled. Prepare retention offer including: (1) why she's leaving (2) best offer (3) risk factors."

**Weak Evaluation Tasks** :
- "Search logs for payment_complete"
- "Find customer by ID 45892"
- "Schedule meeting with jane@acme.corp"

---

## Core Design Principles {#principles}

### Principle 1: **Choose the Right Tools to Build**

**DO:**
- Implement tools for distinct, high-impact workflows
- Consider what agents actually need vs. what the API exposes
- Build search-first tools (e.g., `search_logs` vs. `read_logs`)
- Combine operations intelligently

**DON'T:**
- Wrap every API endpoint as a tool
- Create overlapping tools that confuse agents
- Build tools for edge cases or rare workflows

### Principle 2: **Use Strategic Namespacing**

**Format**: `{service}_{resource}_{action}`

```
GOOD:
  - asana_search_projects
  - slack_send_message
  - github_list_prs

AVOID:
  - get_data
  - process
  - execute
```

**Why it matters:**
- Reduces ambiguity when agents choose tools
- Helps agents understand scope
- Prevents wrong tool selection
- Scales cleanly as tool library grows

### Principle 3: **Return Only High-Signal Context**

**Response Design:**
```
INCLUDE:
  - name
  - semantic IDs (slugs, natural names)
  - 0-indexed IDs
  - similar other fields

EXCLUDE:
  - uuid, mime_type, 256px_image_url
  - Internal technical identifiers
  - Irrelevant metadata
  - Low-context information
```

### Principle 4: **Optimize for Token Efficiency**

Implement pagination, filtering, truncation with sensible defaults:

```python
# Tools should support:
- pagination (limit, offset)
- range selection (start_date, end_date)
- filtering (status, category)
- search targeting (query parameter)
```

**Smart Truncation Guidance:**
```
POOR ERROR:
   "Invalid input"

HELPFUL ERROR:
   "Query returned 5,000+ results. Use date filters or search keywords.
    Example: search_logs(status='failed', start_date='2024-01-01')"
```

### Principle 5: **Craft Exceptional Tool Descriptions**

This is **the most important factor** in tool performance.

**Description Should Include:**
- What the tool does (clear purpose)
- When to use it (explicit use cases)
- When NOT to use it (limitations)
- What each parameter means
- What information is returned
- Important caveats or constraints

**Minimum**: 3-4 sentences per tool (more for complex tools)

**Example: Poor vs. Good**

**POOR:**
```json
{
  "name": "get_stock_price",
  "description": "Gets the stock price for a ticker."
}
```

**GOOD:**
```json
{
  "name": "get_stock_price",
  "description": "Retrieves the current stock price for a given ticker symbol. 
    The ticker must be a valid symbol for publicly traded companies on 
    major US exchanges (NYSE, NASDAQ). Returns the latest trade price in USD. 
    Use when users ask about current stock prices. Does NOT provide company 
    information or historical data.",
  "input_schema": {
    "properties": {
      "ticker": {
        "type": "string",
        "description": "Stock ticker symbol, e.g., 'AAPL' for Apple Inc."
      }
    }
  }
}
```

---

## Implementation Best Practices

### Parameter Design
- Use **unambiguous names**: `user_id` not `user`
- Provide **input examples** for complex parameters
- Include **enum options** for constrained choices
- Set **sensible defaults** for optional parameters

### Response Structure
- Keep response size bounded

### Error Handling
- Return **actionable error messages** with examples
- Never return opaque error codes
- Guide agents toward valid inputs
- Suggest filters/pagination when result too large

---

## Developer Checklist {#checklist}

Before shipping tools, verify:

### Design Phase
- Identified high-impact workflows
- Consolidated related operations
- Avoided overlapping tools
- Chose clear naming scheme with prefixes
- Tool count is reasonable (not 100+ tools)

### Implementation Phase
- Tool descriptions are 3-4+ sentences
- Each parameter is unambiguously named
- Input examples provided for complex schemas
- Response returns only high-signal data
- Response size is bounded (pagination implemented)
- Error messages are actionable
- Truncation guidance helps agents optimize

### Testing Phase
- Created realistic evaluation tasks (20+)
- Ground truth outputs defined
- Built evaluation loop programmatically
- Tested with interleaved thinking enabled
- Measured accuracy, tokens, call counts, errors
- Reviewed agent reasoning transcripts
- Identified confusion patterns

### Optimization Phase
- Refined descriptions based on errors
- Simplified overlapping tools
- Improved response format/structure
- Tested response_format enum if applicable
- Re-evaluated on held-out test set
- Used Claude Code for batch optimization

### Documentation Phase
- Clear usage examples provided
- Parameter constraints documented
- Response format explained
- Edge cases noted
- Tool relationships mapped

---

## 🔍 Common Anti-Patterns to Avoid

| Anti-Pattern | Solution |
|-----------------|-----------|
| List all → agent reads token-by-token | Search/filter → agent targets directly |
| Generic names (get_data) | Namespaced names (service_resource_action) |
| Vague descriptions | Explicit: what, when, when-not, caveats |
| All technical IDs (uuid, mime_type) | Semantic names + 0-indexed IDs |
| Bloated responses | Return only high-signal fields |
| Silent failures | Actionable error messages |
| Hundreds of micro-tools | Fewer, more capable composite tools |
| No input examples | Example calls for complex parameters |
