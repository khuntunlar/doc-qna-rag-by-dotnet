# 🤖 DocQnA — RAG-Based Document Q&A App

> Ask natural language questions over your uploaded PDF documents using AI — powered by Retrieval-Augmented Generation (RAG).

---

## ✨ Features

### 🚀 V2.0 Highlights

- 🧠 **Conversation-Aware Q&A** — Follow-up questions understand previous context
- ⚡ **Batch Embeddings** — Faster ingestion using NVIDIA NIM batch API with fallback
- 🗂️ **Multi-Document Q&A** — Ask questions across entire collections
- 🎤 **Voice Input** — Speak your question using Web Speech API (Chrome)
- 📥 **Export Chat** — Download conversations as Markdown or PDF

---

### ✅ Core Features

- 🔐 **JWT Authentication** — Secure login, refresh tokens, auto-refresh
- 📄 **PDF Upload** — Drag & drop with validation and status tracking
- 🧠 **Full RAG Pipeline** — Extract → Chunk → Embed → Store
- 💬 **AI Q&A (RAG)** — Grounded answers from your documents
- 🌊 **Streaming Responses** — Real-time token streaming via SSE
- 📚 **Source Attribution** — Chunk-level citations with relevance score
- 📜 **Chat History** — Persisted conversations with stats and management
- 🗂️ **Collections** — Group documents and manage them easily
- 🔒 **User Isolation** — Complete data separation per user
- 📱 **Responsive UI** — Works across devices
- 🛡️ **Error Handling + Validation** — Clean API responses + FluentValidation

---




## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  React 18 + TypeScript                  │
│     MUI styled() · Zustand · Axios · react-hot-toast   │
└────────────────────────┬────────────────────────────────┘
                         │ HTTP / REST / SSE
┌────────────────────────▼────────────────────────────────┐
│              ASP.NET Core 8 Web API                     │
│   JWT Auth · EF Core · FluentValidation · Serilog      │
│         ExceptionMiddleware · HealthChecks              │
└──────┬──────────────────────────────┬───────────────────┘
       │                              │
┌──────▼──────┐               ┌───────▼────────────────────┐
│  PostgreSQL │               │      RAG Pipeline           │
│  (Metadata) │               │                             │
│  · Users    │               │  PDF → PdfPig → Extract     │
│  · Documents│               │      → Sliding Window Chunk │
│  · Chat     │               │      → NVIDIA NIMBatch Embed│
│  · History  │               │      → Qdrant Store         │
│  · Collections              │                             │
└─────────────┘               │  Query → Embed→Multi-Search │
                              │        (Qdrant)→Re-rank->   │
                              |        Llama → Stream       |
                         ┌────▼────┐  ┌───────▼──────┐
                         │ Qdrant  │  │ NVIDIA NIM   │
                         │ Vectors │  │ Llama 4 +    │
                         │  DB     │  │ nv-embedqa   │
                         └─────────┘  └──────────────┘
```

---

## 🛠️ Tech Stack

| Layer          | Technology                                             | Purpose                            |
| -------------- | ------------------------------------------------------ | ---------------------------------- |
| Frontend       | React 18 + TypeScript                                  | UI framework                       |
| Styling        | MUI `styled()` utility                                 | Component styling — no inline sx   |
| State          | Zustand                                                | Global auth state                  |
| HTTP           | Axios + interceptors                                   | API calls, 401 auto-refresh        |
| Notifications  | react-hot-toast                                        | User feedback toasts               |
| Markdown       | react-markdown                                         | Render LLM markdown answers        |
| Backend        | ASP.NET Core 8 Web API                                 | REST + SSE API                     |
| Auth           | JWT Bearer + BCrypt                                    | Secure authentication              |
| Validation     | FluentValidation                                       | Request validation                 |
| ORM            | EF Core 8 + PostgreSQL                                 | Relational data storage            |
| PDF Parsing    | PdfPig                                                 | Text extraction from PDFs          |
| Chunking       | Custom sliding window                                  | 2000-char chunks, 200-char overlap |
| Embeddings     | NVIDIA NIM (`nvidia/nv-embedqa-e5-v5`)                 | 1024-dim vector embeddings         |
| LLM            | NVIDIA NIM (`meta/llama-4-maverick-17b-128e-instruct`) | Answer generation                  |
| Streaming      | Server-Sent Events (SSE)                               | Token-by-token streaming           |
| Vector DB      | Qdrant (gRPC port 6334)                                | Cosine similarity search           |
| Logging        | Serilog                                                | Structured request logging         |
| Health         | ASP.NET Health Checks                                  | PostgreSQL + Qdrant monitoring     |
| Infrastructure | Docker + Docker Compose                                | PostgreSQL + Qdrant containers     |

---

## 🏆 What Makes This Production-Grade?

- ✅ Context-aware AI (not just single-turn Q&A)
- ✅ Multi-document reasoning across collections
- ✅ Optimized embedding pipeline (batch + fallback)
- ✅ Real-time streaming UX (SSE)
- ✅ Voice-enabled interaction
- ✅ Exportable conversations (PDF/Markdown)
- ✅ Clean architecture (separation of concerns)
- ✅ Error handling + validation + logging
- ✅ Scalable vector search (Qdrant)

> This is not a demo — it's a full-stack AI system.

---

### 1. Clone the Repository

```bash
cd doc-qna-rag-by-dotnet
```

### 2. Start Docker Containers

```bash
docker-compose up -d
```

| Service          | Port                     | Dashboard                       |
| ---------------- | ------------------------ | ------------------------------- |
| PostgreSQL 16    | 5432                     | Connect via DBeaver             |
| Qdrant Vector DB | 6333 (REST), 6334 (gRPC) | http://localhost:6333/dashboard |

### 3. Configure the Backend

Create `DocQnA.API/appsettings.Development.json` — **gitignored, never commit:**

```json
{
  "Nvidianim": {
    "ApiKey": "nvapi-your-key-here",
    "BaseUrl": "https://integrate.api.nvidia.com/v1",
    "ChatModel": "meta/llama-4-maverick-17b-128e-instruct",
    "EmbeddingModel": "nvidia/nv-embedqa-e5-v5"
  }
}
```

Verify `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5432;Database=docqna_db;Username=docqna_user;Password=docqna_pass123"
  },
  "Jwt": {
    "SecretKey": "your-generated-secret-key",
    "Issuer": "DocQnA",
    "Audience": "DocQnA",
    "ExpiryMinutes": "60"
  },
  "Qdrant": {
    "Endpoint": "http://localhost:6333",
    "VectorSize": 1024
  }
}
```

### 4. Run the Backend

```bash
cd DocQnA.API
dotnet run
```

- Swagger UI → `http://localhost:5000/swagger`
- Health Check → `http://localhost:5000/health`

> EF Core migrations run automatically on startup.

### 5. Run the Frontend

```bash
cd doc-qna-client
npm install
npm run dev
```

App → `http://localhost:5173`

---

## 📁 Project Structure

```
DocQnA/
│
├── DocQnA.API/                        # ASP.NET Core 8 Backend
│   ├── Controllers/
│   │   ├── AuthController.cs          # Register, Login, Refresh, Logout
│   │   ├── DocumentController.cs      # Upload, List, Delete, Status
│   │   ├── QnAController.cs           # Ask, AskStream (SSE), History CRUD
│   │   └── CollectionController.cs    # Collections CRUD + doc management
│   ├── Services/
│   │   ├── AuthService.cs             # JWT auth business logic
│   │   ├── TokenService.cs            # JWT + refresh token generation
│   │   ├── DocumentService.cs         # Upload + Qdrant cleanup on delete
│   │   ├── IngestionService.cs        # RAG pipeline orchestrator
│   │   ├── PdfExtractorService.cs     # PDF → raw text (PdfPig)
│   │   ├── TextChunkerService.cs      # Sliding window chunker
│   │   ├── NimService.cs              # NVIDIA NIM embeddings + LLM + streaming
│   │   ├── QdrantService.cs           # Vector store CRUD (gRPC)
│   │   ├── QnAService.cs              # Q&A + streaming + history
│   │   └── CollectionService.cs       # Collections business logic
│   ├── Models/
│   │   ├── User.cs
│   │   ├── Document.cs
│   │   ├── ChatMessage.cs
│   │   ├── Collection.cs
│   │   └── CollectionDocument.cs      # Join table
│   ├── DTOs/
│   │   ├── AuthDTOs.cs
│   │   ├── DocumentDTOs.cs
│   │   ├── QnADTOs.cs
│   │   └── CollectionDTOs.cs
│   ├── Validators/
│   │   ├── AuthValidators.cs          # FluentValidation rules
│   │   ├── QnAValidators.cs
│   │   └── CollectionValidators.cs
│   ├── Middleware/
│   │   └── ExceptionMiddleware.cs     # Global error handling
│   ├── Infrastructure/
│   │   ├── AppDbContext.cs
│   │   └── Migrations/
│   ├── Extensions/
│   │   └── ClaimsPrincipalExtensions.cs
│   └── Program.cs
│
├── doc-qna-client/                    # React + TypeScript Frontend
│   └── src/
│       ├── pages/
│       │   ├── LoginPage.tsx
│       │   ├── RegisterPage.tsx
│       │   ├── DashboardPage.tsx      # Upload + document management
│       │   ├── ChatPage.tsx           # Streaming chat + history sidebar
│       │   ├── HistoryPage.tsx        # Full history + stats + individual delete
│       │   └── CollectionsPage.tsx    # Collections CRUD
│       ├── components/
│       │   ├── DocumentUploader.tsx   # Drag & drop, PDF-only with rejection toast
│       │   ├── DocumentList.tsx       # List with status chips + chat button
│       │   ├── SourceViewer.tsx       # Collapsible source chunks
│       │   ├── ProtectedRoute.tsx     # JWT-guarded routes
│       │   ├── ErrorBoundary.tsx      # React error boundary
│       │   ├── skeletons/
│       │   │   ├── DocumentSkeleton.tsx
│       │   │   ├── HistorySkeleton.tsx
│       │   │   └── CollectionSkeleton.tsx
│       │   └── styles/
│       │       ├── AuthStyles.ts
│       │       ├── DocumentStyles.ts  # NavPrimaryButton + NavDangerButton
│       │       ├── ChatStyles.ts
│       │       ├── HistoryStyles.ts
│       │       └── CollectionStyles.ts
│       ├── api/
│       │   ├── authApi.ts             # Axios + 401 interceptor + token refresh
│       │   ├── documentApi.ts
│       │   ├── qnaApi.ts              # ask + askStream (SSE) + history CRUD
│       │   └── collectionApi.ts
│       ├── store/
│       │   └── authStore.ts           # Zustand auth state
│       ├── hooks/
│       │   └── usePageTitle.ts        # Dynamic page titles
│       └── types/
│           └── index.ts               # All TypeScript interfaces
│
├── docker-compose.yml
└── README.md
```

---

## 🧠 RAG Pipeline

### Ingestion (runs in background after PDF upload)

```
1. EXTRACT   PdfPig reads all pages → raw text
2. CHUNK     Sliding window → 2000-char chunks, 200-char overlap
3. EMBED     Each chunk → NVIDIA NIM (nv-embedqa-e5-v5) → 1024-dim float vector
4. STORE     Vectors + chunk text stored in Qdrant (one collection per document)
5. READY     Document status updated to "ready" in PostgreSQL
```

### Query (streaming via SSE)

```
1. EMBED     Question → NVIDIA NIM → 1024-dim vector
2. SEARCH    Cosine similarity in Qdrant → top 5 chunks (score threshold: 0.3)
3. SOURCES   Sources sent to frontend immediately via SSE
4. PROMPT    System prompt + context chunks + user question assembled
5. STREAM    Llama via NVIDIA NIM → tokens streamed via SSE
6. DISPLAY   Frontend renders tokens live with blinking cursor
7. SAVE      Full Q&A saved to ChatMessages table
```

---

## 🔐 Security

- `appsettings.Development.json` is gitignored — API keys never committed
- Passwords hashed with BCrypt
- JWT access tokens expire in 60 minutes
- Refresh tokens rotate on every login (7-day expiry)
- 401 responses trigger automatic token refresh in Axios interceptor
- Documents and vectors isolated per user at DB and Qdrant level
- FluentValidation on all request DTOs
- Global exception middleware returns clean JSON (no stack traces in production)

---

## 🐳 Docker

```bash
docker-compose up -d    # start PostgreSQL + Qdrant
docker-compose down     # stop
docker logs docqna_postgres
docker logs docqna_qdrant
```

| Service       | Port                     | Notes                                           |
| ------------- | ------------------------ | ----------------------------------------------- |
| PostgreSQL 16 | 5432                     | User data, documents, chat history, collections |
| Qdrant        | 6333 (REST), 6334 (gRPC) | Vector embeddings, one collection per document  |

---



| Feature | Impact |
|--------|--------|
| Conversation Context | Enables follow-up questions |
| Batch Embeddings | ~10x faster ingestion |
| Multi-Document Q&A | Cross-document intelligence |
| Voice Input | Hands-free interaction |
| Export Chat | Shareable outputs |

---


🚀 Focused on building production-grade AI systems using RAG, vector search, and modern full-stack architecture.

**Stack:** `React` · `TypeScript` · `MUI` · `ASP.NET Core 8` · `EF Core` · `FluentValidation` · `NVIDIA NIM` · `Qdrant` · `Docker` · `SSE`

---

## 📄 License

MIT License
