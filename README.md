# Action Engine

**Privacy-first civic action creation, review, and distribution API.**

Part of the [CivicWorks](https://civ.works) ecosystem — designed as a standalone, API-driven service that can be consumed by the CivicWorks/CivicSky media platform or any authorized client.

## Overview

Action Engine enables organizations to create, submit, and distribute civic actions to subscribers based on geolocation and issue affinity. The system is designed for automation-first operation with AI-powered content review and minimal human intervention.

### Core Pipeline

```
Creation → AI Review → [Auto-Approve / Flag Exception] → Distribution → Analytics
```

### Action Sources
- **Native**: Organizations create actions directly via the API
- **Mobilize.us Import**: Bridge for organizations transitioning from Mobilize.us
- **API Ingestion**: Third-party action providers (ACLU, Momsrising, etc.)

### Privacy Architecture
- All infrastructure hosted in EU/Switzerland — zero US-based components
- Subscriber data anonymized with reduced-precision geolocation (city-level)
- Aggregate-only analytics returned to organizations (no PII)
- Designed to resist jurisdictional overreach

## Tech Stack

- **Language**: Python 3.12+
- **Framework**: FastAPI
- **Database**: PostgreSQL (encrypted at rest)
- **AI Review**: Modular — pluggable content moderation pipeline
- **Auth**: OAuth 2.0 / JWT
- **Hosting**: EU/Swiss infrastructure (Hetzner/Scaleway/Infomaniak — TBD)

## Project Structure

```
action-engine/
├── README.md
├── docs/
│   ├── schema.md              # Data model documentation
│   ├── api-spec.md            # API endpoint specification
│   └── architecture.md        # Infrastructure & privacy architecture
├── src/
│   ├── main.py                # FastAPI application entry
│   ├── config.py              # Environment configuration
│   ├── models/                # SQLAlchemy / Pydantic models
│   │   ├── organization.py
│   │   ├── action.py
│   │   ├── issue.py
│   │   ├── subscriber.py
│   │   └── analytics.py
│   ├── routes/                # API endpoints
│   │   ├── actions.py         # CRUD + submission workflow
│   │   ├── organizations.py   # Org management
│   │   ├── issues.py          # Issue taxonomy
│   │   ├── distribution.py    # Distribution targeting
│   │   └── analytics.py       # Org-facing analytics
│   ├── services/              # Business logic
│   │   ├── ai_review.py       # AI content review pipeline
│   │   ├── distribution.py    # Geo + affinity matching
│   │   ├── ingestion.py       # Mobilize.us + external imports
│   │   └── trust_scoring.py   # Org accountability scoring
│   └── middleware/            # Auth, rate limiting, logging
│       ├── auth.py
│       └── privacy.py         # PII scrubbing, audit logging
├── tests/
├── migrations/
├── docker-compose.yml
├── Dockerfile
├── requirements.txt
└── .env.example
```

## Getting Started

```bash
# Clone the repo
git clone https://github.com/CivicWorks/action-engine.git
cd action-engine

# Set up environment
cp .env.example .env
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Run locally
uvicorn src.main:app --reload
```

## Roadmap

### Phase 1 — Demonstrable Prototype
- [ ] Action creation API (native)
- [ ] AI auto-review with exception flagging
- [ ] Action display/feed endpoint
- [ ] Basic geo + issue affinity matching
- [ ] Org trust scoring

### Phase 2 — Functional MVP
- [ ] Mobilize.us ingestion pipeline
- [ ] Org onboarding and management
- [ ] Subscriber notification system
- [ ] Analytics dashboard API
- [ ] EU/Swiss infrastructure deployment

### Phase 3 — Coalition Scale
- [ ] Strategic action plans and workplans
- [ ] Thought leader provisioning
- [ ] Large org/coalition management
- [ ] Advanced engagement catalysts (news, stories, art)

## License

TBD

## Contact

george@civ.works
