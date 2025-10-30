# ELIC - Requirements for Architectural Assessment

**Document Purpose:** Define requirements for AI-powered tender documentation tool (authority-side) for Codex/Claude Code to assess architectural feasibility and recommend implementation approach.

**Target:** Romanian contracting authorities creating public procurement tenders  
**Distribution:** ADR (Autoritatea pentru Digitalizarea României)  
**Timeline:** Demo in 10 days (if feasibility confirmed)  
**Assessment Deadline:** 5 November 2025

---

## Executive Summary

### What We're Building
Context-adaptive questionnaire that generates compliant Romanian public procurement tender documentation (starting with Caiet de Sarcini + Specificații Tehnice). Copilot-style UX with multi-source knowledge base (legislation, templates, best practices, anti-patterns).

### What We Need From You
1. **Architectural recommendation** - RAG strategy, context adaptation approach, model selection
2. **Feasibility assessment** - Can this be built in 10 days for demo?
3. **Risk identification** - What are the top 3-5 failure modes?
4. **Cost estimation** - Token economics per document (target: €2-10)
5. **Model strategy** - Romanian language quality + upgrade path

### Strategic Context
- **User has 10 days** to build demo if feasibility confirmed (5 Nov gate decision)
- **Customer is ADR** (institutional buyer, not individual municipalities)
- **Validation path:** Demo → ADR pitch → Institutional pilot (not field user testing)
- **Kill criteria:** If unfeasible, pivot/kill before building

---

## 1. Functional Requirements

### 1.1 Core Functionality

**Context-Adaptive Questionnaire**
- System presents dynamic questionnaire based on 6 context dimensions (see Section 1.2)
- Questions adapt to project characteristics (Works vs Services vs Goods)
- Generated output: SEAP-compliant tender documentation

**Multi-Section Generation (MVP Scope)**
- **Section 1:** Caiet de Sarcini (requirements specification) - ~15,000 words
- **Section 2:** Specificații Tehnice (technical specifications) - ~15,000 words
- Total output: ~30,000 words per tender document

**Copilot-Style UX**
- **Left pane:** Main questionnaire workflow (sequential questions)
- **Right pane:** On-demand assistance (click "?" → context → suggestions)
- User can: Accept suggestion / Edit manually / Request regeneration

**Compliance Validation**
- Flag restrictive requirements (per ANAP guidance)
- Highlight potential legal violations (Legea 98/2016, Legea 101/2016)
- Warn about common mistakes (from ANAP "cele mai frecvente observații")

**Output Format**
- SEAP-compatible format (manual export acceptable for demo)
- Proper document structure, section numbering, cross-references
- Romanian language, formal legislative style

### 1.2 Context Dimensions (6 Required)

System must adapt questionnaire and output based on these dimensions:

| Dimension | Values | Impact on Output |
|-----------|--------|------------------|
| **Project Type** | Works / Services / Goods | Different sections required, technical terminology changes |
| **CPV Code** | 45* (construction) / 72* (IT) / etc. | Adjusts technical specs, compliance rules |
| **Contract Value** | <€140k / €140k-€5M / >€5M | Procedure type, guarantees, timelines differ |
| **Funding Source** | EU / National / Local | Reporting requirements, DNSH analysis inclusion |
| **Authority Type** | Central / Local / Agency | Approval workflows, delegation levels |
| **Urgency** | Standard / Accelerated | Timeline modifications, justification requirements |

**Question for Assessment:** With 6 dimensions × 3-5 values each (729-7,776 combinations), is hard-coded rule-based adaptation viable, or do we need dynamic LLM routing?

### 1.3 Section-Level Operations

**Sequential Generation with Locking**
1. User completes questionnaire for Section 1 (Caiet de Sarcini)
2. System generates Section 1 → User validates → Lock
3. User completes questionnaire for Section 2 (Specificații Tehnice)
4. System generates Section 2 using Section 1 as context → Validate → Lock

**Editing & Regeneration**
- User can **edit output directly** (manual override)
- User can **request section regeneration** with feedback ("make less restrictive")
- If Section 1 edited after Section 2 generated → Flag Section 2 as "needs regeneration"
- System maintains cross-references automatically (e.g., "As specified in Section 2.3...")

**Critical Requirement:** Must support **diff-based regeneration** (regenerate single section, not entire 30k word document). This is essential for token cost management.

---

## 2. Knowledge Base Architecture

### 2.1 Multi-Source Knowledge Base

System retrieves from 5 knowledge sources:

| Source Type | Content | Usage Pattern |
|-------------|---------|---------------|
| **Legislation** | Legea 98/2016, Legea 101/2016, HG 395/2016, ANAP instructions | Validation: "Is this requirement legal?" |
| **Templates** | 30 successful tenders (10 per vertical: Works/Services/Goods) | Generation: "What should Section X contain?" |
| **Best Practices** | ANAP published guidance documents | Generation: "What's the recommended approach?" |
| **Anti-Patterns** | ANAP "cele mai frecvente observații" (common mistakes) | Validation: "Why is this flagged as problematic?" |
| **Vertical Structures** | Category-specific requirements (CPV-driven) | Adaptation: "How does this differ for IT vs Construction?" |

### 2.2 Retrieval Strategy

**Three retrieval modes:**

1. **Validation queries** - "Is this requirement compliant?"
   - Query: Legislation + Anti-patterns
   - Output: ✅ Compliant / ⚠️ Warning / ❌ Non-compliant with citation

2. **Generation queries** - "What should Section 3.2 contain for Works projects?"
   - Query: Templates + Best practices + Vertical structures
   - Output: Suggested content with sources

3. **Assistance queries** - "Why was this flagged?"
   - Query: Anti-patterns + Legislation
   - Output: Explanation with reference to ANAP guidance

**Question for Assessment:** 
- What's the optimal chunking strategy for Romanian legislation? (By article? By section? Semantic similarity?)
- How to handle cross-references within legislation? (Article 164 refers to Article 165...)
- Should templates be full documents or extracted patterns?

### 2.3 Template Acquisition Strategy

**Phase 1 (Demo - 2-3 days work):**
- Scrape 150 tenders from SEAP (50 per vertical: Works/Services/Goods)
- Use LLM to auto-classify: ✅ Good / ⚠️ Has issues / ❌ Poor quality
  - Criteria: ANAP compliance, no single-bidder, no appeals, completeness
- Manually review top 30 (10 per vertical) for inclusion in knowledge base

**Phase 2 (Post-demo, if ADR interested):**
- Request ADR's "gold standard" templates (institutional knowledge)

**Question for Assessment:** Is LLM-based template classification viable, or too risky for demo? Alternative: Use only ANAP-published examples (lower quantity, higher quality).

---

## 3. Language & Model Strategy

### 3.1 Romanian Language Requirements

**Critical constraint:** Romanian legislative language quality is essential. Formal, precise terminology required.

**Language characteristics:**
- Formal register (legislative/administrative style)
- Fixed terminology (terms of art must be exact)
- Latin-origin vocabulary (but distinct from other Romance languages)
- Structured syntax (articles, alineate, puncte)

**Quality threshold:** Generated text must be indistinguishable from human-written Romanian legislative text. Grammatical errors acceptable if <1%, terminology errors unacceptable.

### 3.2 Model Selection Question

**Current assumption:** Claude Sonnet 4.5 or GPT-4o

**Questions for Assessment:**
1. **Which model has best Romanian legislative language quality?**
   - Test: Paraphrase Article 164 from Legea 98/2016 in Romanian
   - Benchmark: Compare Sonnet 4.5, GPT-4o, GPT-4-turbo

2. **Is fine-tuning needed?**
   - Cost of fine-tuning Romanian-specific model?
   - Alternative: Extended prompts with glossary + few-shot examples?

3. **Hybrid approach viable?**
   - Use Sonnet for reasoning + Romanian grammar checker layer?
   - Trade-off: Complexity vs language quality

**Recommendation requested:** Model selection strategy balancing Romanian fluency vs reasoning capability vs cost.

### 3.3 Model Upgrade Path

**Critical requirement:** Easy model swaps as LLMs evolve (Sonnet 4.5 → 5, GPT-4 → 5).

**Architecture principle:** Model-agnostic design
- ✅ Abstraction layer between application logic and LLM API
- ✅ Prompt templates work across Claude/GPT/other
- ✅ Evaluation suite to test new models before production swap
- ❌ No model-specific features (function calling syntax, token limits, etc.)

**Success metric:** Can upgrade from Sonnet 4.5 to Sonnet 5 in <1 day without rewriting prompts.

**Question for Assessment:** What's the architectural pattern for model-agnostic design? Service layer? Adapter pattern?

---

## 4. Cost Management & Token Economics

### 4.1 Token Cost Targets

**Target cost per document:** €2-10 (acceptable for demo)

**Cost drivers:**
- 30,000 word output = ~40,000 tokens output
- Legislation retrieval = ~20,000 tokens input (varies by query)
- Total per document: ~60,000 tokens (first pass)

**With Sonnet 4.5 pricing (~$3/M input, ~$15/M output):**
- First pass: ~€1.00 per document ✅ Within target
- With 5 corrections (no guardrails): ~€5.00 ❌ Approaching limit

### 4.2 Cost Management Strategy (Critical)

**Problem:** Iterative corrections can explode costs (regenerate 30k words × 20 times = 600k tokens output)

**Required guardrails:**

1. **Section-level generation with locking**
   - Generate Section 1 → Validate → Lock → Generate Section 2
   - Correction regenerates ONLY affected section (5k words, not 30k)

2. **Diff-based editing**
   - If user edits Section 3, don't regenerate Sections 1-2
   - Track which sections are "dirty" and need regeneration

3. **Prompt caching** (Claude supports this)
   - Cache retrieved legislation chunks per session
   - Reuse cached context across sections (don't re-retrieve)

4. **Token budgets per section**
   - Allocate max tokens per section (e.g., 8,000 output tokens for Caiet de Sarcini)
   - If exceeded, flag for manual intervention

5. **Model tiering** (nice-to-have)
   - Use Haiku for simple tasks (validation, UI suggestions)
   - Use Sonnet for quality-critical tasks (document generation)

**Success metric:** 5 correction iterations should cost <€3 (vs €5 without guardrails).

**Question for Assessment:** What's the architecture for section-level locking + diff-based regeneration? State management approach?

### 4.3 Token Monitoring

**Requirement:** System must track token usage per document, per session, per section.

**Use cases:**
- Cost estimation for ADR pitch ("€5 per tender" vs "€50 per tender")
- Model comparison (Sonnet vs GPT on same task)
- Optimization (identify which sections consume most tokens)

**Question for Assessment:** Claude API provides token counts. What's the logging/monitoring architecture?

---

## 5. Validation & Quality Assurance

### 5.1 Compliance Validation Strategy

**Problem:** How do we know generated caiet de sarcini is legally compliant?

**Three-layer validation:**

**Layer 1: Automated rule checking (must-have for demo)**
- Check against ANAP "cele mai frecvente observații" (common mistakes)
- Flag obvious violations: restrictive language, missing sections, invalid timelines
- Output: ✅ Pass / ⚠️ Warning / ❌ Fail with citation

**Layer 2: LLM-based validation (nice-to-have)**
- Ask LLM: "Does this requirement comply with Article 164?"
- Cross-check generated content against legislation
- Output: Confidence score + explanation

**Layer 3: Human validation (required for demo)**
- Manual review by user (Marius) before ADR presentation
- Acceptable error rate: <5% (not zero, but low enough for demo)

**Question for Assessment:** Is Layer 2 (LLM self-validation) reliable, or does it hallucinate compliance?

### 5.2 Quality Metrics

**Success criteria for demo:**
- ✅ **Functional:** Demo runs without crashing for 3 scenarios (Works/Services/Goods)
- ✅ **Quality:** Generated documents have <5% compliance errors (human-reviewed)
- ✅ **UX:** ADR representative can navigate without instructions
- ✅ **Value:** ADR says "This would save X hours per tender"

**Minimum acceptable quality:** 
- All required sections present and properly structured
- Romanian language grammatically correct (minor errors acceptable)
- No obvious legal violations (restrictive requirements, missing mandatories)

**Question for Assessment:** What automated tests can ensure quality threshold met before demo?

---

## 6. Multi-Section Coherence

### 6.1 Cross-Section Dependencies

**Problem:** Caiet de Sarcini (Section 1) and Specificații Tehnice (Section 2) must be coherent.

**Requirements:**
- Section 2 references Section 1 ("As specified in Caiet de Sarcini, Section 2.3...")
- Technical specs derive from requirements (waterfall logic)
- If Section 1 changes, Section 2 must be flagged as "needs regeneration"

**Example coherence requirement:**
- Caiet de Sarcini specifies: "Contractor must use concrete grade C30/37"
- Specificații Tehnice must detail: "Concrete grade C30/37 per SR EN 206..."
- If user changes to C25/30 in Section 1, Section 2 must update

**Question for Assessment:** How to maintain coherence? 
- Generate Section 2 with Section 1 as full context? (Token heavy)
- Extract key requirements from Section 1 into structured format? (Lighter)
- Track dependencies explicitly? (Complex state management)

### 6.2 Cross-Reference Management

**Requirement:** System must automatically generate and maintain cross-references.

**Examples:**
- "See Section 3.2.1 for technical specifications"
- "As defined in Annex 1"
- "In accordance with requirements in Caiet de Sarcini"

**If user adds new section or reorders, references must update automatically.**

**Question for Assessment:** Document structure representation? DOM-like tree? Plain text with regex? Structured format (JSON/XML)?

---

## 7. Demo Strategy & Success Metrics

### 7.1 Demo Scope

**Three scripted scenarios (must work flawlessly):**

1. **Works project** (Construction/Infrastructure)
   - Context: Road modernization, €2.5M, EU funding, CPV 45233140
   - Sections: Caiet de Sarcini + Specificații Tehnice
   - Demo flow: Complete questionnaire → Generate → Show output

2. **Services project** (IT/Consulting)
   - Context: Software development, €500k, National funding, CPV 72212000
   - Sections: Caiet de Sarcini + Specificații Tehnice
   - Demo flow: Complete questionnaire → Generate → Show adaptation

3. **Goods project** (Equipment/Supplies)
   - Context: Medical equipment, €1M, Local funding, CPV 33100000
   - Sections: Caiet de Sarcini + Specificații Tehnice
   - Demo flow: Complete questionnaire → Generate → Show compliance validation

**One wildcard scenario (nice-to-have):**
- ADR brings their own tender parameters
- System attempts generation (may degrade gracefully if edge case)
- Shows flexibility vs just scripted demos

### 7.2 Demo Success Criteria

**Minimum success (required):**
- ✅ All 3 scripted demos run without crashes
- ✅ Generated documents are structurally complete (all sections present)
- ✅ <5% compliance errors (pre-validated before demo)
- ✅ Romanian language is grammatically acceptable
- ✅ UX is navigable (ADR person doesn't get stuck)

**Stretch success (nice-to-have):**
- ✅ Wildcard scenario works (ADR's own tender)
- ✅ ADR expresses interest in pilot ("We want to test this")
- ✅ Token costs are <€3 per document (demonstrable efficiency)

**Kill criteria (grounds for NO-GO at Gate 1):**
- ❌ Cannot generate structurally complete documents in 10 days
- ❌ Romanian language quality is unusable (>10% errors)
- ❌ Token costs exceed €20 per document (uneconomical)
- ❌ RAG retrieval doesn't work (legislation chunks don't retrieve correctly)

### 7.3 Demo Data Preparation

**Required assets:**
- 3 × questionnaire answer sets (pre-filled for scripted demos)
- 3 × expected outputs (pre-generated, validated for quality)
- Knowledge base populated: Legislation + 30 templates + ANAP guidance + Anti-patterns
- Test harness to verify demo scenarios before ADR presentation

**Question for Assessment:** What's the data preparation timeline? Can knowledge base be populated in 2-3 days?

---

## 8. Error Handling & Failure Modes

### 8.1 Correction Workflow

**User finds error in generated Section 3. What happens?**

**Option 1: Manual edit (must-have)**
- User clicks "Edit" → Inline text editor → Save
- System marks section as "manually edited" (no auto-regeneration)
- User can still request regeneration (discards edits)

**Option 2: Regeneration with feedback (must-have)**
- User highlights problem text → Clicks "Regenerate with feedback"
- Provides instruction: "Make this less restrictive" / "Add technical details"
- System regenerates ONLY affected section (not entire document)

**Option 3: Copilot suggestion (nice-to-have)**
- User clicks "?" next to problematic text
- Copilot offers 2-3 alternative phrasings
- User selects one or ignores

**Question for Assessment:** What's the state management for tracking edits vs regenerations? How to merge manual edits with LLM output?

### 8.2 Top 5 Failure Modes

**Identify these failure modes and provide mitigation strategies:**

| Failure Mode | Detection | Mitigation |
|--------------|-----------|------------|
| **RAG retrieval fails** | Legislation query returns no chunks | Fallback: Use pre-loaded common articles. Error: "Could not retrieve specific legislation" |
| **Context adaptation breaks** | Questionnaire doesn't adapt for valid input | Fallback: Use generic questionnaire. Log error for investigation |
| **Romanian language unusable** | Generated text has >10% grammatical errors | Detection: Grammar checker. Mitigation: Switch model or add grammar layer |
| **Token costs explode** | Single document costs >€20 | Detection: Token counter. Mitigation: Abort generation, simplify requirements |
| **SEAP format invalid** | Output doesn't match required structure | Detection: Schema validator. Mitigation: Regenerate with stricter template |

**Question for Assessment:** Are these the right failure modes? What else should we monitor?

### 8.3 Graceful Degradation

**Requirement:** System must handle failures without crashing.

**Examples:**
- If RAG fails → Use fallback template-based generation (lower quality, but works)
- If token budget exceeded → Truncate section and flag for manual completion
- If validation flags 50+ errors → Abort and ask user to simplify requirements
- If model API down → Queue request and notify user of delay

**Demo philosophy:** Better to show "This is an edge case we're still handling" than crash.

---

## 9. Model Evaluation Framework

### 9.1 A/B Model Testing

**Requirement:** System must support comparing multiple models on same task.

**Use cases:**
- Compare Sonnet 4.5 vs GPT-4o on Romanian legislative text quality
- Measure token costs: Sonnet vs GPT on identical task
- Evaluate latency: Which model is faster?

**Evaluation harness:**
```
test_cases = [
    {
        "input": "Works project, €2M, EU funding, CPV 45000000",
        "expected_sections": ["Introducere", "Caiet de Sarcini", "Specificații"],
        "compliance_checks": ["No restrictive language", "ANAP compliant"]
    },
    # 10-20 test cases
]

models = ["claude-sonnet-4.5", "gpt-4o", "gpt-4-turbo"]

for model in models:
    for test in test_cases:
        output = generate_caiet(model, test.input)
        score = evaluate(output, test.expected_sections, test.compliance_checks)
        log(model, test, score, cost_in_tokens, latency)

# Output: Model comparison table
```

**Metrics to measure:**
- Section completeness (% of required sections generated)
- Compliance rate (% of ANAP rules followed)
- Romanian language quality (grammar checker score)
- Token cost per document
- Latency (seconds to generate)

**Question for Assessment:** Is this evaluation approach sound? What other metrics matter?

### 9.2 Regression Testing

**Requirement:** When upgrading models (Sonnet 4.5 → 5), ensure quality doesn't degrade.

**Process:**
1. Run evaluation harness on new model
2. Compare scores to baseline (current model)
3. If quality improvement + cost acceptable → Upgrade
4. If quality degradation → Keep current model

**Question for Assessment:** What's the architecture for model versioning? Can we run old/new models in parallel during evaluation?

---

## 10. Constraints & Non-Requirements

### 10.1 Out of Scope for Demo

**Do NOT build for demo:**
- ❌ Multi-tenancy (single demo tenant sufficient)
- ❌ Authentication/authorization (open access for demo)
- ❌ Payment integration (cost tracking only)
- ❌ Production-grade error handling (graceful degradation sufficient)
- ❌ Scalability (demo handles 1 concurrent user)
- ❌ Mobile responsiveness (desktop only)
- ❌ Full SEAP API integration (manual export acceptable)

**Can be added post-demo if ADR interested.**

### 10.2 Timeline Constraint

**Hard deadline:** 10 days from 5 Nov (if Gate 1 = GO)
- Days 1-2: Knowledge base preparation + architecture setup
- Days 3-5: Core generation pipeline (RAG + LLM)
- Days 6-7: Copilot UX + validation layer
- Days 8-9: Testing + refinement
- Day 10: Demo preparation (scripts, pre-validation)

**Question for Assessment:** Is 10 days realistic for functional demo? What's the critical path?

### 10.3 Technology Constraints

**Preferred stack (but flexible):**
- Claude Sonnet 4.5 API (or GPT-4o if better Romanian)
- Python backend (FastAPI / Flask)
- React frontend (or simpler: Streamlit for rapid prototyping)
- Vector database for RAG (Pinecone / Chroma / Weaviate)

**Question for Assessment:** Is this stack reasonable for 10-day demo? Should we use Streamlit for faster development?

---

## 11. Architectural Questions for Assessment

**Please provide recommendations on:**

### 11.1 RAG Architecture
1. **Chunking strategy** - By article? By section? Semantic similarity? Hybrid?
2. **Retrieval approach** - Dense retrieval (embeddings)? Sparse (BM25)? Hybrid?
3. **Context window management** - How to fit 30k words output + legislation chunks + conversation history?
4. **Caching strategy** - Prompt caching (Claude)? Vector DB caching? Session state?

### 11.2 Context Adaptation
1. **Rule-based vs dynamic** - Hard-coded rules for 6 dimensions (729-7,776 combinations) OR dynamic LLM routing?
2. **Questionnaire structure** - Decision tree? Conditional logic? State machine?
3. **Scalability** - If we add 7th dimension, how difficult to extend?

### 11.3 Model Strategy
1. **Romanian language quality** - Which model is best? Sonnet 4.5, GPT-4o, GPT-4-turbo?
2. **Cost optimization** - Model tiering (Haiku/Sonnet)? Prompt compression? Context pruning?
3. **Upgrade path** - Abstraction layer design? Adapter pattern? Service layer?

### 11.4 State Management
1. **Section locking** - How to track which sections are locked/dirty/needs-regeneration?
2. **Diff-based regeneration** - How to regenerate single section without losing context from others?
3. **Cross-references** - Document structure representation (DOM? JSON? Plain text)?

### 11.5 Validation Strategy
1. **LLM self-validation** - Reliable? Or prone to hallucination?
2. **Automated compliance checks** - Rule-based (regex)? LLM-based? Hybrid?
3. **Quality gates** - At what point do we abort generation (too many errors)?

### 11.6 Demo Viability
1. **10-day timeline** - Realistic? What's the critical path?
2. **Knowledge base prep** - Can we scrape + classify 150 tenders in 2-3 days?
3. **Risk mitigation** - What's the biggest technical risk? How to de-risk early?

---

## 12. Success Criteria for This Assessment

**We need from you:**

1. **Architecture recommendation** (high-level design, not implementation)
2. **Feasibility verdict** - GO (buildable in 10 days) / NO-GO (not feasible) / NEEDS-PROTOTYPING (spike risks first)
3. **Risk identification** - Top 3-5 failure modes + mitigation strategies
4. **Cost estimation** - Token economics (realistic range, not precise)
5. **Model recommendation** - Which model(s) for Romanian legislative text?
6. **Critical path** - What must be built first? What can be deferred?

**Format:** Markdown document, strategic recommendations (not code)

**Deadline:** 5 November 2025 (to inform Gate 1 decision)

---

## 13. Background Context

**User profile:**
- Marius, technical founder, execution-proven (OEPH, AIGov projects)
- NO direct public procurement experience (couch research + anecdotal signals)
- Validation-first approach (no building before feasibility confirmed)
- ADR access (distribution channel via institutional contact)

**Market context:**
- 45% single-bidder rate in Romanian tenders (low competition)
- 58% appeal/correction rate on Works contracts (poor quality)
- Root cause: Poor tender documentation from contracting authorities
- Solution: Help authorities create better tenders (not help bidders respond)

**Strategic pivot:**
- FROM: Bidder-side tool (suppliers preparing bids)
- TO: Authority-side tool (authorities creating tenders)
- WHY: Construction > deconstruction, ADR distribution, API bypass value

**Kill criteria (if any of these true, project stops):**
- Cannot be built in 10 days for functional demo
- Romanian language quality unusable (>10% errors)
- Token costs exceed €20 per document (uneconomical)
- RAG architecture doesn't work for Romanian legislation

---

## Appendices

### Appendix A: Sample Caiet de Sarcini Structure

**Typical structure (8,000+ lines, 30,000 words):**

1. Informații Generale (General Information)
   - Autoritate contractantă (Contracting authority)
   - Situația actuală (Current situation)
   - Sursa de finanțare (Funding source)

2. Identificarea Proiectului (Project Identification)
   - Strategie / Necesitate / Oportunitate
   - Scopul proiectului (Project scope)
   - Obiective (Objectives)

3. Descrierea Serviciilor (Service Description)
   - Stabilirea criteriilor, cerințelor și etapelor
   - Expertiza tehnică (Technical expertise)
   - Studiul de fezabilitate (Feasibility study)

4. Planificare, Implementare și Management
   - Management de contract
   - Notificări și comunicare
   - Personalul și baza tehnico-materială

5. Ipoteze și Riscuri (Assumptions and Risks)
   - Ipoteze (Assumptions)
   - Riscuri (Risks)

6. Anexe (Annexes)
   - Registrul riscurilor (Risk register)
   - Cerințe privind studiul geotehnic
   - Livrabile (Deliverables)

### Appendix B: Key Legislation References

**Primary legislation:**
- Legea 98/2016 (Public Procurement Law)
- Legea 101/2016 (Remedies Law)
- HG 395/2016 (Implementation Regulations)

**ANAP guidance:**
- ANAP Instructions (various, 2016-2024)
- "Cele mai frecvente observații" (Common mistakes document)
- Template examples (published by ANAP)

**EU directives:**
- Directive 2014/24/EU (Public Procurement)

### Appendix C: ANAP Common Mistakes (Sample)

**From "cele mai frecvente observații" document:**
- Restrictive language (limiting competition unnecessarily)
- Missing mandatory sections (e.g., risk register, implementation timeline)
- Contradictory requirements (Section A says X, Section B says not-X)
- Invalid timelines (e.g., 10 days for complex tender response)
- Overly specific technical requirements (brand names, not "or equivalent")
- Missing DNSH analysis (for EU-funded projects)
- Incorrect CPV codes (misclassifying project type)

**These are gold for validation rules.**

---

**End of Requirements Document**

**Contact:** Marius (via Claude chat)  
**Date:** 30 October 2025  
**Version:** 1.0
