<div align="center">

# Nexus

**An AI canvas workspace — ask, generate and arrange every result as a card on one infinite board.**

[![Next.js](https://img.shields.io/badge/Next.js-16-000000?style=flat-square&logo=next.js)](https://nextjs.org)
[![React](https://img.shields.io/badge/React-19-61DAFB?style=flat-square&logo=react&logoColor=black)](https://react.dev)
[![TypeScript](https://img.shields.io/badge/TypeScript-strict-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org)
[![Prisma](https://img.shields.io/badge/Prisma-7-2D3748?style=flat-square&logo=prisma)](https://www.prisma.io)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Neon-4169E1?style=flat-square&logo=postgresql&logoColor=white)](https://neon.tech)
[![pgvector](https://img.shields.io/badge/pgvector-HNSW-4169E1?style=flat-square&logo=postgresql&logoColor=white)](https://github.com/pgvector/pgvector)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-v4-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white)](https://tailwindcss.com)
[![AI SDK](https://img.shields.io/badge/AI_SDK-v5-000000?style=flat-square&logo=vercel)](https://ai-sdk.dev)
[![Gemini](https://img.shields.io/badge/Google-Gemini-4285F4?style=flat-square&logo=googlegemini&logoColor=white)](https://ai.google.dev)
![CI](https://github.com/Metenchuk/Nexus-ai/actions/workflows/ci.yml/badge.svg)

### [**Live Demo**](https://nexus-ai-chi-orcin.vercel.app) · [Report Bug](https://github.com/Metenchuk/Nexus-ai/issues) · [Request Feature](https://github.com/Metenchuk/Nexus-ai/issues)

</div>

---

## Overview

Most AI apps are a chat log — you ask, you scroll, you lose the answer.

**Nexus** treats every AI result as a **card on an infinite canvas**. A web search, a generated chart, an AI image, a document analysis — each one becomes a block you can move, link, expand and reuse. Draw an arrow from one card to another and the second one inherits the first one's content as context: that is the graph of your thinking, not a transcript of it.

Built as a portfolio project to demonstrate production patterns end-to-end — server-first data fetching with React Server Components, streaming AI responses, RAG over documents with pgvector, type-safe model output with Zod, optimistic UI, and a clean feature-based architecture.

**What makes it worth a look:**

- **Infinite canvas** built on React Flow — pan, zoom, drag, connect, minimap, persisted positions, node virtualization
- **Streaming answers** that type themselves directly into a card while it sits on the board
- **Context graph** — an edge between two cards feeds one card's output into the other's prompt
- **Cited web search** — Serper results grounded through Gemini, with clickable source chips and a Postgres query cache
- **Document RAG** — PDF/DOCX/TXT chunked, embedded and searched with pgvector; citations jump the preview to the exact page
- **AI image studio** — parallel generation plus background removal, both on one Gemini image model
- **Typed AI output** — `generateObject` + Zod schemas, so charts and recommendations arrive as validated JSON, never parsed text
- **PDF export** — the whole canvas exported as a real, selectable-text PDF built from state, not a screenshot

---

## Screenshots

### The canvas

| Canvas board | Command palette |
| :---: | :---: |
| ![Canvas](public/canvas.png) | ![Command palette](public/command-palette.png) |

### AI blocks

| Web answer | Image studio |
| :---: | :---: |
| ![Web answer](public/web-answer.png) | ![Image studio](public/image-studio.png) |

| Document analysis | Recommendations |
| :---: | :---: |
| ![Document analysis](public/document-analysis.png) | ![Recommendations](public/recommendations.png) |

### Workspace

| Library | Files |
| :---: | :---: |
| ![Library](public/library.png) | ![Files](public/files.png) |

| History | Settings |
| :---: | :---: |
| ![History](public/history.png) | ![Settings](public/settings.png) |

---

## Features

### Canvas — the core of the app

- Infinite board with pan, zoom, fit-view and a minimap, powered by **React Flow**
- Five card types, each its own custom node: **Web answer**, **Chart**, **AI image**, **Document**, **Recommendations**
- **Optimistic creation** — a card appears instantly with a temporary id and a `PENDING` status, then swaps for the real record from the server; it rolls back if the request fails
- **Streaming into the node** — answers are read chunk by chunk and written straight into the card while it sits on the board
- **Context graph** — connect two cards and the target card's prompt is enriched with the source card's content on the server side
- **Node virtualization** — only cards inside the viewport render, so the board stays smooth with many cards
- Drag positions are saved with a debounced `PATCH`, so a reload restores the exact layout
- Hover toolbar per card: regenerate, duplicate, expand, delete, and *turn into chart* on web answers
- Keyboard: `⌘K` for the palette, `Delete` to remove a selection, `Esc` to close, space-drag to pan
- **Interactive landing** — the homepage is a live read-only canvas: drag demo cards and connect them to see context inheritance before signing up

### Command palette

- One input, four modes: search the web, generate a chart, create an image, analyze a document
- The default mode comes from user settings, so `Enter` alone does the right thing
- Every submission is written to the `Prompt` table — that table *is* the History page

### Web answer

- Serper returns the top five results; titles and snippets are passed to Gemini as grounding context
- The model marks every fact with `[1]`, `[2]`, and the client swaps those markers for clickable domain chips
- **Continue** appends a new paragraph, **follow-up** asks within the same sources, **Save** stars the card
- Search results are cached in Postgres for an hour, keyed by the normalised query, cutting external API calls significantly

### Image studio

- Parallel generation — the variations slider controls how many images are produced in one `Promise.all` on the server
- **Remove background** runs on the same Gemini image model via a multimodal editing message, no third-party service
- Bytes go to UploadThing; the database stores only the URL
- Style and ratio default to the values from user settings

### Document analysis (RAG)

- Drag-and-drop upload of PDF, DOCX or TXT through UploadThing
- `pdf-parse` extracts text **page by page**, `mammoth` handles DOCX
- Text is chunked with overlap, embedded, and stored in **pgvector** with an HNSW index
- **Q&A** embeds the question and retrieves the top matching chunks by cosine distance; **Summary** and **Key points** run over the full text
- Each chunk keeps its page number, so a `[p.8]` marker in an answer scrolls the preview to that page
- Answers are cached in the card, so a second click costs nothing

### Recommendations

- Collects every card on the canvas and asks Gemini for next actions via `generateObject` with a Zod schema
- Category is a `z.enum`, so the model physically cannot return a value the filters do not know
- `why?` reveals the reasoning the model returned in the same object — no second request
- Filters live in the URL through **nuqs**, so a filtered view is shareable

### Workspace

- **Library** — canvas grid with previews drawn from real card types and coordinates, server-side search, filters, rename, duplicate (cards *and* edges in one transaction) and delete
- **Files** — everything uploaded, storage broken down by type, preview panel, download, delete from storage and database, and *Ask this file* to open it as a card
- **History** — prompts grouped by day, click to focus the exact card on its canvas, rerun to repeat a prompt, clear all
- **Settings** — profile and avatar, model and language preferences that the API routes actually read, accent gradient applied through CSS variables, live integration status and real monthly usage counters

### Cross-cutting

- **Better Auth** with Google, GitHub and email + password; middleware protects the whole app group with a return-to redirect
- **Custom toast system** — a Zustand store plus a portal, callable from anywhere without a provider
- **Custom SVG spinner** and a branded 404 page — zero UI dependencies for feedback states
- **Usage limits** — AI routes check the monthly counter and return `429` before calling the model
- **PDF export** — `jspdf` loaded via dynamic import, so it never sits in the initial bundle

---

## Tech Stack

### Core

| Layer | Technology |
| --- | --- |
| Framework | Next.js 16 (App Router, RSC, Route Handlers) |
| UI | React 19, Tailwind CSS v4 |
| Language | TypeScript (strict) |
| Database | PostgreSQL (Neon) + Prisma 7 + pgvector |
| Auth | Better Auth (Google, GitHub, credentials) |
| Deployment | Vercel |

### AI

| Purpose | Technology |
| --- | --- |
| SDK | Vercel AI SDK v5 (`streamText`, `generateText`, `generateObject`, `embed`, `embedMany`) |
| Text model | Google Gemini (fast / balanced / deep tiers) |
| Image model | Gemini image model — generation and editing |
| Embeddings | Gemini embedding model, stored in pgvector |
| Web search | Serper (Google Search API) |
| Schema validation | Zod 4 — shared between forms and model output |

### Frontend

| Purpose | Technology |
| --- | --- |
| Canvas | React Flow (`@xyflow/react`) |
| Client state | Zustand (canvas, toasts) |
| URL state | nuqs (filters, search, pagination) |
| Forms | React Hook Form + Zod resolvers |
| File uploads | UploadThing |
| PDF export | jspdf (dynamic import) |
| Icons | Lucide React |
| Dates | date-fns |
| Utilities | use-debounce, nanoid |

### Server

| Purpose | Technology |
| --- | --- |
| Document parsing | pdf-parse (per page), mammoth (DOCX) |
| Vector search | pgvector with an HNSW index (cosine distance) |
| Object storage | UploadThing |
| Search cache | PostgreSQL table with a one-hour TTL |
| Rate limiting | Monthly usage counters per user |

---

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│                     Next.js App Router                    │
│                                                           │
│   Server Components              Client Components        │
│   ──────────────────             ──────────────────       │
│   features/*/server/       →     features/*/components/   │
│   Prisma queries                 React Flow · Zustand     │
│   initial data                   streaming · optimistic   │
└───────────────┬──────────────────────────┬───────────────┘
                │                          │
                │ Route Handlers           │ fetch
                ▼                          ▼
┌──────────────────────────────────────────────────────────┐
│                       app/api/*                           │
│   auth · canvas · cards · edges · search · chart           │
│   image · documents · recommendations · files             │
│   history · settings · account · uploadthing              │
└───┬──────────────┬───────────────┬───────────────┬───────┘
    │              │               │               │
    ▼              ▼               ▼               ▼
┌────────┐  ┌────────────┐  ┌───────────┐  ┌─────────────┐
│ Prisma │  │   Gemini   │  │  Serper   │  │ UploadThing │
│ Neon + │  │  AI SDK v5 │  │  search   │  │   storage   │
│pgvector│  │            │  │           │  │             │
└────────┘  └────────────┘  └───────────┘  └─────────────┘
```

**Feature-based structure.** Every domain owns three folders: `server/` for React Server Components and Prisma queries, `components/` for the client layer, and `hooks/` for the network and state logic that connects them. The boundary between server and client is explicit and visible in the file path.

**Why these choices:**

| Decision | Reason |
| --- | --- |
| One `Card` model with a `Json` data field | Five card types share one table; the type decides the shape of `data` and which node renders it. No polymorphic joins. |
| Zustand as the single source of truth on the canvas | React Flow nodes are derived from the store with `useMemo`, so there is no second state to keep in sync. |
| Context resolved on the server | The generation route looks up incoming edges itself — the client never assembles prompts, and the logic lives in one place. |
| pgvector added only when needed | Started without vectors — Gemini's context window fit whole documents. Added RAG once documents outgrew the window, and kept a hybrid: retrieval only for Q&A, full text for summaries. |
| `Unsupported("vector")` + raw SQL | Prisma has no native vector type, so the column is declared `Unsupported` and read/written via `$queryRaw` with the `<=>` cosine-distance operator. |
| `generateObject` for structured output | Charts and recommendations are validated by Zod before they reach React. No `JSON.parse` in a `try/catch`. |
| Pre-generated ids for canvas duplicate | `createMany` is one round-trip instead of N, but returns no ids — so ids are generated up front with `randomUUID` to keep the edge mapping, all inside a transaction. |
| Search cache in Postgres | An hourly TTL on a normalised query key saves API quota without adding Redis to the stack. |

---

## Database Schema

| Model | Purpose |
| --- | --- |
| `User` · `Session` · `Account` · `Verification` | Better Auth tables — profile, sessions, OAuth links, credentials |
| `Settings` | Accent, text model, answer language, default mode, canvas grid |
| `Canvas` | A board — one user can have many |
| `Card` | Any AI block: type, status, position, prompt and a `Json` payload |
| `CardEdge` | A directed link between two cards — the context graph |
| `Chunk` | A document chunk with its page number and a `vector(768)` embedding |
| `Prompt` | Every submission, powering the History page and rerun |
| `File` | Uploads — key, URL, size, kind, status, extracted text, pages |
| `Usage` | Monthly counters per user: text requests, images, indexed pages |
| `SearchCache` | Serper results with an hourly TTL, keyed by the query |

**Card types:** `WEB` · `CHART` · `IMAGE` · `DOC` · `RECS`
**Card statuses:** `PENDING` · `STREAMING` · `DONE` · `ERROR`

---

## Getting Started

### Prerequisites

- Node.js 20+
- A PostgreSQL database with the `vector` extension — a free [Neon](https://neon.tech) project works
- A [Google AI Studio](https://aistudio.google.com/apikey) key
- A [Serper](https://serper.dev) key (2,500 free searches)
- An [UploadThing](https://uploadthing.com) app
- Google and GitHub OAuth applications

### 1. Clone and install

```bash
git clone https://github.com/Metenchuk/nexus-ai.git
cd nexus-ai
npm install
```

### 2. Environment variables

Create a `.env` file in the project root:

```dotenv
DATABASE_URL="postgresql://..."
DIRECT_URL="postgresql://..."

AUTH_SECRET="run: npx auth secret"
AUTH_URL="http://localhost:3000"

GOOGLE_CLIENT_ID=""
GOOGLE_CLIENT_SECRET=""
GITHUB_CLIENT_ID=""
GITHUB_CLIENT_SECRET=""

GEMINI_API_KEY=""
SERPER_API_KEY=""
UPLOADTHING_TOKEN=""
```

### 3. Set up the database

```bash
npx prisma migrate dev
npx prisma generate
```

The `vector` extension is created by the migration. If your provider needs it enabled manually, run `CREATE EXTENSION IF NOT EXISTS vector;` once, then re-run the migration.

### 4. Run

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

---

## Scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Start the development server |
| `npm run build` | Type-check and build for production |
| `npm run start` | Run the production build |
| `npm run lint` | Lint with ESLint |
| `npm run test` | Run unit tests (Vitest) |
| `npm run test:watch` | Run tests in watch mode |
| `npx prisma studio` | Inspect the database |
| `npx prisma migrate dev` | Apply schema changes |

---

## Deployment

1. Push the repository to GitHub and import it into [Vercel](https://vercel.com)
2. Add every environment variable from the list above, setting `AUTH_URL` to the production domain
3. Add the production callback URLs to the Google and GitHub OAuth apps:
   `https://nexus-ai-chi-orcin.vercel.app/api/auth/callback/google` and `.../github`
4. `prisma generate` runs automatically during the build
5. Run `npx prisma migrate deploy` against the production database

---

## Roadmap

- [x] **pgvector retrieval** — chunk and embed documents, search with an HNSW cosine index
- [x] **PDF export** — the canvas exported as a real, selectable-text document
- [x] **Test suite** — Vitest unit tests for the canvas store, wired into GitHub Actions CI
- [ ] **Realtime collaboration** — shared canvases with live cursors
- [ ] **Card templates** — reusable prompt chains saved as presets
- [ ] **Mobile canvas gestures** — pinch to zoom, long-press toolbar

---

## Author

**Nazar Metenchuk**

[GitHub](https://github.com/Metenchuk) · [LinkedIn](https://linkedin.com/in/nazar-metenchuk)

<div align="center">

If this project caught your eye, a star would mean a lot.

</div>
