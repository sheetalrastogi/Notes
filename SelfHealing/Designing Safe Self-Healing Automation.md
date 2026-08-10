Source:  https://www.ministryoftesting.com/insights/when-ai-maintains-your-tests-designing-safe-self-healing-automation

Summary: When AI Maintains Your Tests - Designing Safe Self-Healing Automation

AI-assisted self-healing automation can reduce the maintenance burden of UI automation by automatically recovering from broken locators while keeping test execution safe, predictable, and transparent. The key principle is that AI should be a fallback mechanism, not the primary automation engine.

**Key Problems Addressed**
 - Most UI automation failures are caused by locator changes, not actual application defects.
 - Teams spend significant effort fixing broken locators instead of validating business functionality.
 - As applications evolve, locator maintenance becomes a continuous and costly cycle.
 - Excessive flaky failures reduce confidence in automation results.


**Core Design Principle**
AI as Controlled Fallback
 - Use deterministic automation first
 - Invoke AI only when locator lookup fails
 - Validate AI-generated locator before execution
 - Log and review healing actions

❌ Do not let AI drive automation directly

This preserves predictability and debuggability while providing resilience against UI changes.

**Self-Healing Workflow**
```text
Test Starts
      ↓
Try Original Locator
      ↓
Locator Found?
      │
      ├─ Yes ──► Execute Test Step
      │
      └─ No
            ↓
       Check Cache
            ↓
       Healed Locator Found?
            │
            ├─ Yes ──► Reuse Locator
            │
            └─ No
                  ↓
          Parse Locator Signals
                  ↓
          Detect Context
                  ↓
          Prune UI Hierarchy
                  ↓
          Invoke LLM
                  ↓
          Generate Locator
                  ↓
          Validation Layer
                  ↓
          Execute Action
                  ↓
          Cache & Log Result
                  ↓
          Continue Test
```


# Major Components of the Healing Framework

| Component | Purpose |
|:----------|:--------|
| Deterministic Execution | Normal locator-based automation |
| Cache Layer | Reuse previously healed locators |
| Locator Parser | Extract text, type, action, and context |
| Context Detection | Identify correct screen/workflow |
| XML Pruning | Reduce UI hierarchy complexity |
| LLM Generator | Generate alternative locator |
| Validation Layer | Verify locator safety and uniqueness |
| Execution Layer | Execute action using healed locator |
| Logging Layer | Record healing decisions and outcomes |

# Locator Healing Pipeline

Step 1. Locator Parsing

Extracts:

Element text
Element class
Action type
Screen context
Element type

Example:

Continue Button
      ↓
Text = Continue
Class = Button
Action = Click
Context = Checkout

Step 2. Context Detection

Uses:

Locator names
Test flow
Screen semantics

Purpose:

Continue Button

may exist on multiple screens.   Context helps select the correct element.

# Step 3. XML Pruning

Before AI:

500+ UI Elements


After pruning:

Only relevant candidates


Benefits:

Lower latency
Lower cost
Better accuracy

# Step 4. LLM Locator Generation

AI receives:

Parsed attributes
Context
Action type
Pruned UI hierarchy

Rules:

Match intended action
Match correct context
Prefer stable attributes
Return EMPTY if uncertain

# Step 5. Validation Layer

Generated locator must pass:

Validation CheckLocator resolves to one element
Element is visible
Element is enabled
Element is clickable
Matches action type
Matches current UI state

AI proposes.

Framework decides.

This is the system's primary safety gate.

# Step 6. Execution


After validation:
```text
Use Healed Locator
      ↓
Perform Action
      ↓
Verify Outcome
```

Execution success is not guaranteed even after healing. Runtime validation is still required.

# Step 7. Cache and Logging

Successful healed locators are:

Cached for future use
Logged for traceability
Audited for review

This avoids repeated AI calls for the same UI change.

Locator Update Strategies

| Strategy | Description | Risk |
|:---------|:------------|:-----|
| Runtime Only | Use healed locator during execution only | Low |
| Human Review | Create PR for engineer approval | Medium |
| Auto Merge | Commit locator automatically | High |

The article recommends starting with Runtime Only Healing and gradually introducing more automation.

Limitations
- Similar elements may confuse the model.
- Complex parent-child locator relationships are difficult.
- Prompt changes may impact existing scenarios.
- LLM calls add execution time and cost.
- Context quality directly affects healing accuracy.


# Real-World Example

A button label changed from:

Continue

to

Next


More than 40 tests failed.

The healing system automatically recovered most tests, but some failures remained because the model selected a similar element from a different workflow.

Key lesson:

Better context was more valuable than a smarter model.

Key Lessons Learned
1	AI should be a fallback, not a replacement	
2	Context matters more than model capability	
3	Structured signals reduce ambiguity	
4	Prompt engineering requires engineering discipline	
5	Validation layers are mandatory	
6	Transparency and logging build trust	
7	Success depends on system design, not AI alone	


Takeaway for Selenium/Test Automation Architects

```text
Prefer Deterministic Locator
         ↓
If Failed
         ↓
Use AI-Assisted Healing
         ↓
Validate Healed Locator
         ↓
Execute Action
         ↓
Verify Business Outcome
         ↓
Log & Cache
```

The article's central message is that successful self-healing automation is achieved through strong guardrails, validation, context awareness, caching, and human oversight, not by blindly trusting AI-generated locators.
