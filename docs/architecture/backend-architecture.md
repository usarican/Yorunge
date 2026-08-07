# Yörünge — Backend Mimari ve Teknik Tasarım Dokümanı

> **Doküman:** `backend-architecture.md`  
> **Sürüm:** v1.0  
> **Konum:** `docs/architecture/backend-architecture.md`  
> **Hedef:** Yörünge platformunun backend altyapısını, 4 eksenli değerlendirme motorunu, LangGraph tabanlı yapay zeka ajan orkestrasyonunu, Supabase PostgreSQL veri şemasını, RAG (Knowledge) hattını, Docker kapsayıcı yapısını ve Coolify VPS canlıya alım süreçlerini teknik standartlarda tanımlamak.

---

## 1. Genel Mimari Bakış

Yörünge Backend, **Modüler Monolit (Modular Monolith)** mimarisinde tasarlanmıştır. Erken aşamada gereksiz mikroservis karmaşıklığından kaçınılırken, yapay zeka orkestrasyonu, AST kod analizi ve web-hook dinleyicileri net servis sınırları ile izole edilmiştir.

```
                                ┌───────────────────────────────────────────┐
                                │             NEXT.JS FRONTEND              │
                                └─────────────────────┬─────────────────────┘
                                                      │ REST / SSE / WS
                                                      ▼
┌───────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                       FASTAPI BACKEND SERVICE                                     │
│                                                                                                   │
│  ┌───────────────────────┐  ┌───────────────────────┐  ┌───────────────────────┐                  │
│  │   Onboarding Router   │  │   Interview Router    │  │  Roadmap & Task Auth  │                  │
│  └───────────┬───────────┘  └───────────┬───────────┘  └───────────┬───────────┘                  │
│              │                          │                          │                              │
│              ▼                          ▼                          ▼                              │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                                  LANGGRAPH AGENT ORCHESTRATION                              │  │
│  │  ┌──────────────────────┐  ┌──────────────────────┐  ┌───────────────────────────────────┐  │  │
│  │  │ AST Analysis Subgraph│  │ Adaptive Scenario    │  │ 4-Axis Evaluation Aggregator      │  │  │
│  │  └──────────────────────┘  │ Interview Subgraph   │  └───────────────────────────────────┘  │  │
│  │  ┌──────────────────────┐  └──────────────────────┘  ┌───────────────────────────────────┐  │  │
│  │  │ GitHub PR Agent      │                            │ Android Knowledge RAG Subgraph    │  │  │
│  │  └──────────────────────┘                            └───────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────┬──────────────────────────────────────────────┘  │
│                                                 │                                                 │
│  ┌──────────────────────────────────────────────┴──────────────────────────────────────────────┐  │
│  │                                 SERVICES & INTEGRATION LAYER                                │  │
│  │  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐    ┌───────────────┐  │  │
│  │  │ GitHub API Client│    │ AST Code Parser  │    │ Vector / Embed   │    │ Crypto / Auth │  │  │
│  │  └──────────────────┘    └──────────────────┘    └──────────────────┘    └───────────────┘  │  │
│  └──────────────────────────────────────────────┬──────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┼─────────────────────────────────────────────────┘
                                                  │ AsyncPG / SQLAlchemy 2.0
                                                  ▼
                                ┌───────────────────────────────────────────┐
                                │         SUPABASE MANAGED POSTGRES         │
                                └───────────────────────────────────────────┘
```

---

## 2. Teknoloji Yığını (Backend Tech Stack)

| Bileşen | Seçilen Teknoloji | Seçim Gerekçesi |
| :--- | :--- | :--- |
| **Dil & Runtime** | Python 3.12 | Zengin AI/LLM kütüphaneleri, LangChain/LangGraph ekosistemi, tip desteği (`typing`). |
| **Web Framework** | FastAPI | Yüksek performanslı asenkron (async/await) yapı, Pydantic v2 doğrulaması, otomatik OpenAPI dokümantasyonu. |
| **Agent Framework** | LangGraph | State tabanlı, döngüsel (cyclical) ajan akışları, duraklatma/devam ettirme (human-in-the-loop), sub-graph modülerliği. |
| **Veritabanı** | Supabase (Managed Postgres) | Şifreli veri saklama, otomatik yedekleme, PostgreSQL 15+, pgvector eklentisi ile vektör arama desteği. |
| **ORM / DB Sürücü** | SQLAlchemy 2.0 (Async) + AsyncPG | Tip güvenli asenkron DB erişimi, migration yönetimi (Alembic). |
| **Async Task Worker** | Arq / Celery + Redis | Ağır AST parsing ve uzun süren RAG indeksleme işlemlerini arkaplanda yürütme. |
| **Dağıtım / Deployment** | Docker + Coolify (Hetzner VPS) | Git push ile otomatik deploy, Let's Encrypt SSL, container orchestration. |

---

## 3. Klasör Düzeni (Directory Structure)

Backend kod tabanı `backend/` klasörü altında aşağıdaki yapıda organize edilir:

```
backend/
├── Dockerfile                      # Multi-stage production container tanımı
├── docker-entrypoint.sh            # Alembic migration & Uvicorn startup scripti
├── pyproject.toml                  # Dependency ve tool yapılandırmaları (Poetry/Ruff)
├── alembic.ini                     # DB Migration yapılandırması
├── app/
│   ├── main.py                     # FastAPI uygulama başlatma noktası
│   ├── config.py                   # Pydantic BaseSettings ile ortam değişkenleri
│   ├── api/                        # HTTP API Router katmanı
│   │   ├── v1/
│   │   │   ├── auth.py             # GitHub OAuth & JWT işlemleri
│   │   │   ├── onboarding.py       # 5 adımlı onboarding ve canlı telemetri (SSE)
│   │   │   ├── evaluation.py       # 4 eksenli skor ve detay raporları
│   │   │   ├── interview.py        # Adaptif sistem tasarımı mülakatı (WebSocket / SSE)
│   │   │   ├── roadmap.py          # Kişiselleştirilmiş yol haritası ve görevler
│   │   │   ├── webhooks.py         # GitHub App webhook dinleyici (PR/Commit)
│   │   │   └── knowledge.py        # Android RAG ve soru-cevap servisi
│   ├── agents/                     # LangGraph Ajan Orkestrasyonu
│   │   ├── state.py                # Global Agent State tanımları (Pydantic / TypedDict)
│   │   ├── ast_analyzer.py         # Repo Kotlin / Compose AST analiz sub-graph'ı
│   │   ├── scenario_interview.py   # Adaptif diyalog mülakat sub-graph'ı
│   │   ├── evaluator.py            # 4 eksenli skor birleştirici (Aggregator)
│   │   ├── pr_reviewer.py          # GitHub PR kod inceleme ve feedback ajanı
│   │   └── rag_qa.py               # Güncel Android dokümantasyonu RAG ajanı
│   ├── core/                       # Çekirdek altyapı
│   │   ├── database.py             # Async SQLAlchemy session ve engine
│   │   ├── security.py             # Cryptography (Fernet) ile OAuth token şifreleme
│   │   └── logging.py              # Structlog ile yapılandırılmış loglama
│   ├── db/                         # SQLAlchemy Modelleri & Schema
│   │   ├── base.py                 # Declarative Base
│   │   └── models/                 # DB Tabloları (User, Assessment, Task vb.)
│   ├── schemas/                    # Pydantic REST Request/Response şemaları
│   └── services/                   # Dış servis entegrasyonları
│       ├── github_service.py       # GitHub REST & GraphQL API Client
│       ├── ast_parser_service.py   # Tree-sitter / Tree-sitter-kotlin parser sarmalayıcısı
│       └── vector_service.py       # OpenAI / Voyage embeddings & pgvector araması
└── tests/                          # Pytest test paketleri
```

---

## 4. Veritabanı Şeması (Database ER Schema)

Supabase PostgreSQL üzerinde tutulacak ana tablolar ve ilişkileri aşağıda tanımlanmıştır:

```mermaid
erDiagram
    USERS ||--o{ REPOSITORIES : owns
    USERS ||--o{ ASSESSMENTS : undergoes
    ASSESSMENTS ||--o{ QUIZZES : contains
    ASSESSMENTS ||--o{ INTERVIEWS : triggers
    ASSESSMENTS ||--o1 ROADMAPS : generates
    ROADMAPS ||--o{ ROADMAP_NODES : contains
    ROADMAP_NODES ||--o{ TASKS : assigns
    TASKS ||--o{ TASK_SUBMISSIONS : evaluates

    USERS {
        uuid id PK
        string github_id UK
        string username
        string email
        string encrypted_github_token
        datetime created_at
    }

    ASSESSMENTS {
        uuid id PK
        uuid user_id FK
        string status
        jsonb scores_4_axis
        text rationale_summary
        datetime completed_at
    }

    QUIZZES {
        uuid id PK
        uuid assessment_id FK
        jsonb questions_and_answers
        int score
    }

    INTERVIEWS {
        uuid id PK
        uuid assessment_id FK
        jsonb chat_history
        int system_design_score
        text evaluation_notes
    }

    ROADMAPS {
        uuid id PK
        uuid user_id FK
        string current_level
        jsonb target_skills
    }

    TASKS {
        uuid id PK
        uuid roadmap_node_id FK
        string title
        text description
        string github_issue_url
        string status
    }
```

### 4.1. 4 Eksenli Skor Yapısı (`scores_4_axis` JSONB)
```json
{
  "kotlin_idioms": {
    "score": 78,
    "level": "Mid",
    "evidence": ["Data class / Sealed interface doğru kullanımı", "Coroutines Flow kullanımı yetersiz"]
  },
  "jetpack_compose": {
    "score": 65,
    "level": "Mid",
    "evidence": ["Unnecessary recompositions tespit edildi", "Side-effect API (LaunchedEffect) düzgün kullanılmış"]
  },
  "software_principles": {
    "score": 82,
    "level": "Senior",
    "evidence": ["Clean Architecture katmanları net", "Unit test coverage %40 seviyesinde"]
  },
  "system_design": {
    "score": 55,
    "level": "Junior-Mid",
    "evidence": ["Offline-first senkronizasyon stratejisinde eksikler", "Cache invalidation senaryosu zayıf"]
  }
}
```

---

## 5. LangGraph Ajan Altyapısı (AI Orchestration)

Tüm yapay zeka akışları `LangGraph` üzerinde durum makineleri (State Graph) olarak kurgulanmıştır.

### 5.1. Onboarding & Analiz Akışı (AST Analysis -> Quiz -> Interview -> Report)

```mermaid
graph TD
    Start([Kullanıcı Repolarını & CV Seçti]) --> ASTNode[AST Code Parser Node]
    ASTNode --> CVNode[CV & Domain Keyword Extractor Node]
    CVNode --> QuestionGenNode[Kişiselleştirilmiş 8-12 Quiz Üretim Node'u]
    QuestionGenNode --> QuizWait[Quiz Yanıtlarını Bekle (Human-in-the-loop)]
    QuizWait --> ScoreCheck{Quiz + AST Skoru >= Mid mi?}
    ScoreCheck -- Evet --> InterviewNode[Adaptif Sistem Tasarımı Diyalog Node'u]
    ScoreCheck -- Hayır (Junior) --> AggregatorNode[4 Eksenli Skor Birleştirici Node]
    InterviewNode --> AggregatorNode
    AggregatorNode --> RoadmapNode[Kişiselleştirilmiş Yol Haritası Üretim Node'u]
    RoadmapNode --> End([Onboarding Raporu Hazır])
```

---

## 6. Docker & Dağıtım Mimarisi (Coolify VPS Setup)

### 6.1. Backend Multi-Stage `Dockerfile` (`backend/Dockerfile`)

```dockerfile
# Build Stage
FROM python:3.12-slim AS builder

WORKDIR /app

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=off \
    PIP_DISABLE_PIP_VERSION_CHECK=on

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --prefix=/install -r requirements.txt

# Final Stage
FROM python:3.12-slim AS runner

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    curl \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /install /usr/local
COPY . /app

EXPOSE 8000

ENTRYPOINT ["/bin/sh", "docker-entrypoint.sh"]
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

### 6.2. `docker-entrypoint.sh`
```bash
#!/bin/sh
set -e

echo "Veritabanı migration'ları uygulanıyor (Alembic)..."
alembic upgrade head

echo "FastAPI sunucusu başlatılıyor..."
exec "$@"
```

### 6.3. Yerel Orkestrasyon (`docker-compose.yml`)
```yaml
version: '3.8'

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql+asyncpg://postgres:postgres@db:5432/yorunge
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - GITHUB_CLIENT_ID=${GITHUB_CLIENT_ID}
      - GITHUB_CLIENT_SECRET=${GITHUB_CLIENT_SECRET}
      - ENCRYPTION_KEY=${ENCRYPTION_KEY}
    volumes:
      - ./backend:/app
    depends_on:
      - db

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=yorunge
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

---

## 7. Güvenlik ve Performans Kuralları

1. **OAuth Token Şifreleme:** Kullanıcının GitHub OAuth token'ı Supabase Postgres üzerinde asla düz metin (plain text) saklanmaz. `cryptography.fernet.Fernet` ile AES-256 algoritmasıyla şifrelenir.
2. **API Rate Limiting:** External LLM maliyetlerini korumak için mülakat ve quiz üretme endpoint'lerine `slowapi` ile IP ve User-ID bazlı rate-limiting (örn. dakikada max 10 istek) uygulanır.
3. **Canlı Telemetri (SSE):** Onboarding Adım 2'deki kod analizi telemetri akışı HTTP SSE (`EventSourceResponse`) üzerinden aktarılır. Connection timeout'lara karşı her 15 saniyede bir `keep-alive` ping atılır.
