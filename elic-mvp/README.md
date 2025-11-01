# ELIC MVP - Romanian Procurement Tender Documentation Tool

AI-powered copilot for Romanian contracting authorities to generate compliant public procurement tender documentation (Caiet de Sarcini).

## 🎯 Project Status

**Phase:** Demo Development (10-day sprint)
**Target Demo Date:** Mid-November 2025
**Scope:** Works projects only, Caiet de Sarcini section
**Strategic Assessment:** [ELIC-Strategic-Assessment.md](../ELIC-Strategic-Assessment.md)

## 📋 Quick Start

### Prerequisites

- Docker & Docker Compose
- Anthropic API key (Claude Sonnet 3.5)
- OpenAI API key (optional, for GPT-4o validation)

### Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/Standivarius/e-licitatie.git
   cd e-licitatie/elic-mvp
   ```

2. **Configure environment variables**
   ```bash
   cp .env.example .env
   # Edit .env and add your API keys
   ```

3. **Start the services**
   ```bash
   docker-compose up -d
   ```

4. **Access the application**
   - Frontend (Streamlit): http://localhost:8501
   - Backend API: http://localhost:8000
   - API Docs: http://localhost:8000/docs
   - ChromaDB: http://localhost:8001

### Development Mode

```bash
# View logs
docker-compose logs -f

# Restart services
docker-compose restart

# Stop services
docker-compose down

# Rebuild after code changes
docker-compose up -d --build
```

## 🏗️ Architecture

### High-Level Design

```
┌─────────────────────────────────────────────────────────┐
│                 STREAMLIT FRONTEND                      │
│  (Questionnaire Flow + Live Preview + Edit/Regen)       │
└────────────┬────────────────────────────────────────────┘
             │
┌────────────▼────────────────────────────────────────────┐
│              FASTAPI BACKEND                            │
│  - Context Builder (3 dimensions → template selection)  │
│  - Section Manager (locking, regeneration)              │
│  - Token Budget Monitor (cost tracking)                 │
└────────┬───────────────────────┬────────────────────────┘
         │                       │
    ┌────▼─────┐          ┌──────▼──────┐
    │ ChromaDB │          │  Claude API │
    │   RAG    │          │  Sonnet 3.5 │
    └────┬─────┘          └──────┬──────┘
         │                       │
    ┌────▼────────────────────────▼──────┐
    │     KNOWLEDGE BASE                 │
    │  - Templates (10 Works tenders)    │
    │  - ANAP Validation Rules           │
    └────────────────────────────────────┘
```

### Technology Stack

- **Frontend:** Streamlit (rapid prototyping)
- **Backend:** FastAPI (Python 3.11+)
- **LLM:** Claude Sonnet 3.5 (generation) + GPT-4o (validation)
- **Vector DB:** ChromaDB (RAG retrieval)
- **Container:** Docker + Docker Compose

## 📁 Project Structure

```
elic-mvp/
├── backend/
│   ├── app/
│   │   ├── api/              # FastAPI routes
│   │   ├── core/             # Configuration, LLM providers
│   │   ├── models/           # Pydantic models
│   │   ├── services/         # Business logic
│   │   │   ├── generation.py    # Document generation
│   │   │   ├── validation.py    # ANAP compliance checks
│   │   │   ├── rag.py           # RAG retrieval
│   │   │   └── templates.py     # Template management
│   │   └── utils/            # Helpers
│   ├── data/
│   │   ├── templates/        # Gold-standard tender templates
│   │   ├── rules/            # ANAP validation rules
│   │   └── demo_outputs/     # Pre-generated demo outputs
│   ├── tests/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── main.py
├── frontend/
│   ├── pages/
│   │   ├── 1_questionnaire.py
│   │   ├── 2_preview.py
│   │   └── 3_export.py
│   ├── components/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app.py
├── scripts/
│   ├── scrape_seap.py       # SEAP tender scraping
│   ├── validate_templates.py
│   └── test_romanian_quality.py
├── docker-compose.yml
├── .env.example
└── README.md
```

## 🚀 Demo Scope (10-Day Sprint)

### ✅ In Scope

- **Single vertical:** Works projects only (construction/infrastructure)
- **Single section:** Caiet de Sarcini (~12,000 words)
- **3 context dimensions:**
  - Project Type: Works (fixed)
  - Contract Value: <€140k / €140k-€5M / >€5M
  - Funding Source: EU / National / Local
- **Template-based generation:** 10 gold-standard templates + LLM adaptation
- **Rule-based validation:** ANAP common mistakes (~100-200 rules)
- **Manual edit + regeneration:** Section-level granularity

### ⏸️ Deferred to Post-Demo

- Services and Goods verticals
- Specificații Tehnice (second section)
- Full legislative RAG (using validation-only RAG for demo)
- React dual-pane UX
- LLM-based context routing
- SEAP API integration

## 📊 Success Metrics

### Minimum Demo Criteria

1. ✅ Generate structurally complete Caiet de Sarcini for 3 Works scenarios
2. ✅ Romanian language quality: <5% grammatical errors
3. ✅ ANAP compliance: <10 warnings per document
4. ✅ Token cost: <€1 per document
5. ✅ UX: User can complete flow without help

### Cost Targets

- **Per document:** €0.20 - €0.50 (with corrections)
- **Demo (3 docs):** <€1.50 total
- **Business model:** Charge €5-10/document (95%+ gross margin)

## 🧪 Pre-Decision Tests

**Run before November 5th decision:**

1. **Romanian Language Quality Test** (2 hours)
   ```bash
   python scripts/test_romanian_quality.py
   ```
   - Tests both Claude Sonnet 3.5 and GPT-4o
   - Success criterion: <5% error rate

2. **SEAP Scraping Feasibility** (2 hours)
   ```bash
   python scripts/scrape_seap.py --limit 10
   ```
   - Verify data access and parseability

3. **Cost Estimation Validation** (1 hour)
   - Run one full generation, measure tokens
   - Success criterion: <€2 per document

## 📅 10-Day Timeline

### Days 1-2: Foundation + De-Risking
- [ ] Romanian model testing (GATE: must pass)
- [ ] Scrape 50 Works tenders from SEAP
- [ ] Manual selection of 10 gold-standard templates
- [ ] Extract ANAP rules into JSON
- [ ] Streamlit UI skeleton

### Days 3-4: Core Generation Pipeline
- [ ] Template substitution engine
- [ ] LLM integration (Claude Sonnet 3.5)
- [ ] Prompt engineering for Romanian legislative style
- [ ] Basic validation layer

### Days 5-6: Context Adaptation
- [ ] 3-dimension questionnaire logic
- [ ] Section locking/dirty tracking
- [ ] Token counting and budget monitoring
- [ ] Manual edit vs regeneration workflow

### Days 7-8: Validation + QA
- [ ] Run 10 test cases
- [ ] Romanian grammar checking (LanguageTool)
- [ ] ANAP compliance validation
- [ ] Bug fixes and prompt refinement

### Days 9-10: Demo Preparation
- [ ] Pre-generate 3 demo outputs
- [ ] Polish UI
- [ ] Rehearse demo flow
- [ ] Prepare backup (pre-generated outputs)

## 🔑 Key Design Decisions

### 1. Template-Based Approach (Not Full RAG)

**Why:** Lower risk, proven quality, faster development
- Primary generation uses 10 proven templates from SEAP
- LLM adapts templates to context (not generation from scratch)
- RAG used only for validation (ANAP rules)

### 2. Streamlit (Not React)

**Why:** 3-4 days time savings
- Rapid prototyping for demo
- Good enough UX for ADR presentation
- Can rebuild in React for production

### 3. Rule-Based Context Adaptation (Not LLM Routing)

**Why:** 27 combinations manageable with rules
- Reduced dimensions: 3×3×3 = 27 combinations
- Simple decision tree logic
- Can migrate to LLM routing post-demo

### 4. Model Tiering Strategy

- **Sonnet 3.5:** Document generation (best Romanian quality)
- **GPT-4o:** Validation (structured outputs)
- **Haiku:** UI suggestions (90% cheaper)

## ⚠️ Risk Management

### Top 5 Risks

1. **Romanian language quality unacceptable** (30% prob, CRITICAL)
   - Mitigation: Day 1 testing, fallback to template-copying

2. **Template quality insufficient** (40% prob, HIGH)
   - Mitigation: Manual curation, ANAP examples fallback

3. **Token costs explode** (25% prob, MEDIUM)
   - Mitigation: Prompt caching, section budgets, model tiering

4. **Context adaptation breaks** (20% prob, MEDIUM)
   - Mitigation: Simple rules, manual testing, graceful degradation

5. **Demo technical failure** (35% prob, HIGH)
   - Mitigation: Pre-generated outputs, rehearsal, backup narrative

## 📖 Documentation

- [Strategic Assessment](../ELIC-Strategic-Assessment.md) - Full architectural analysis
- [API Documentation](http://localhost:8000/docs) - Interactive API docs
- [Development Guide](./docs/DEVELOPMENT.md) - Setup and contribution guide

## 🤝 Contributing

**Current Phase:** Internal development (Marius + Claude)
**Post-Demo:** Open for ADR partnership collaboration

## 📄 License

Proprietary - ADR Distribution Partnership Model

## 📞 Contact

**Project Lead:** Marius
**Technical Architecture:** Claude (Sonnet 4.5)
**Target Customer:** ADR (Autoritatea pentru Digitalizarea României)

---

**Strategic Recommendation:** RUN PRE-DECISION TESTS → If pass → GO with reduced scope → Demo to ADR

**Expected Outcome:** 65% probability of ADR interest in pilot phase
