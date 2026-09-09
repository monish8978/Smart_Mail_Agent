# Smart Mail Agent — Frontend (Web Application)

A modern, high-performance React + Vite + TypeScript web interface for the **Smart Mail Agent (Mail AI Automation Platform)**. Built with a Windows 11 Fluent-inspired aesthetic, Tailwind CSS, Lucide icons, and Framer Motion.

---

## 1. Overview & Architecture

The frontend provides full operational control, live monitoring, human-in-the-loop review, and configuration management for the autonomous email automation system. It caters to two distinct roles:
- **System Administrator (`admin`)**: Manages multi-tenant client registrations, credentials, master automation kill-switches, global & tenant-level LLM model routing, dynamic CRM connector approval lifecycle, SSRF URL allowlisting, and system telemetry.
- **Client / Tenant User (`client`)**: Monitors tenant mailbox telemetry, tracks ticket lifecycle & sentiment analysis, curates tenant RAG knowledge bases, controls blocked keywords, configures email disclaimers, and reviews/edits/approves AI draft replies.

### Tech Stack
- **Framework:** [React 18](https://react.dev/) + [TypeScript](https://www.typescriptlang.org/)
- **Build & Dev Tooling:** [Vite 5](https://vitejs.dev/)
- **Routing:** [React Router v6](https://reactrouter.com/) (configured with base path `/Smart_Mail_agent`)
- **Styling:** [Tailwind CSS v3](https://tailwindcss.com/) with custom dark/light theme tokens and Fluent UI styling
- **Animations:** [Framer Motion](https://www.framer.com/motion/)
- **Icons:** [Lucide React](https://lucide.dev/)
- **Charts:** [Recharts](https://recharts.org/)
- **State & Utilities:** `clsx`, `tailwind-merge`, localStorage session caching

---

## 2. Project Directory Structure

```
Smart_Mail_Agent_FE/
├── index.html                   # HTML entry point with runtime config injection
├── package.json                 # Dependencies & build scripts
├── vite.config.ts               # Vite configuration (base: /Smart_Mail_agent/, proxy rules)
├── tsconfig.json                # TypeScript project configuration
├── tailwind.config.js           # Theme extensions, colors, animations
├── postcss.config.js            # PostCSS plugins
├── public/
│   ├── config.js                # Runtime environment override (window.__APP_CONFIG__)
│   └── vite.svg
├── dist/                        # Production build output
└── src/
    ├── main.tsx                 # React DOM mount point
    ├── App.tsx                  # App layout, theme bootstrap, route registry
    ├── index.css                # Global CSS, Windows 11 design variables, scrollbars
    ├── components/
    │   ├── Layout.tsx           # Global shell (Topbar + Sidebar + Outlet)
    │   ├── Sidebar.tsx          # Collapsible navigation drawer, role filtering, draft counter
    │   ├── Topbar.tsx           # Search, quick status indicators, user profile dropdown
    │   ├── SetupWizard.tsx      # Multi-step client onboarding wizard
    │   ├── LangGraphVisualizer.tsx # Live interactive node visualizer for LangGraph execution
    │   └── EmailBodyWithDisclaimer.tsx # Email renderer with highlighted disclaimer blocks
    ├── lib/
    │   ├── api.ts               # Typed API client interfacing with FastAPI backend
    │   └── utils.ts             # Style mergers (cn) and formatting helpers
    └── pages/
        ├── auth/
        │   ├── Login.tsx        # Authentication & JWT session initialization
        │   ├── Register.tsx     # Client self-registration request
        │   ├── Logout.tsx       # Session tear-down & token eviction
        │   └── ApproveRegistration.tsx # Direct registration validation
        ├── Dashboard.tsx        # High-level KPIs, mail volumes, sentiment distribution
        ├── Inbox.tsx            # Mail monitor, raw/clean view, blocked email queue, paused senders
        ├── Drafts.tsx           # Human-in-the-loop review queue, batch send, inline editor
        ├── AiProcessing.tsx     # Step-by-step pipeline execution visualizer & latency metrics
        ├── Tickets.tsx          # Synced CRM reference status, docket search, manual dispatch
        ├── EmailAccounts.tsx    # IMAP/SMTP credentials setup & connection test
        ├── PayloadConfig.tsx    # Dynamic CRM Connector studio, OAuth 2.0 handshake, live preview
        ├── KnowledgeBase.tsx    # Document upload, chunking inspector, semantic vector testing
        ├── LlmAnalytics.tsx     # LLM latency, token counts, cost tracking, breakdown charts
        ├── LlmConfigs.tsx       # (Admin) Multi-tenant LLM provider routing & model management
        ├── AdminClients.tsx     # (Admin) Client tenant onboarding, approval, feature flags
        ├── Settings.tsx         # Account settings, confidence score threshold, disclaimers, blocked keywords
        ├── ApiTesting.tsx       # Interactive sandbox for testing backend endpoints
        ├── OrderTracking.tsx    # Order/Docket lookup debugger
        └── SystemHealth.tsx     # System uptime and container health monitors
```

---

## 3. Environment & Runtime Configuration

The application uses dynamic client-side configuration injection via `public/config.js`, enabling deployments across different hosts without rebuilding JavaScript bundles.

### `public/config.js`
```javascript
window.__APP_CONFIG__ = {
  API_URL: "http://localhost:8024",
  WS_URL: "ws://localhost:8024/ws"
};
```

When deploying behind an Nginx reverse proxy or domain name, adjust `API_URL` and `WS_URL` in `config.js` directly:
```javascript
window.__APP_CONFIG__ = {
  API_URL: "https://your-domain.com/api",
  WS_URL: "wss://your-domain.com/ws"
};
```

If `window.__APP_CONFIG__` is absent, `src/lib/api.ts` falls back to Vite environment variables (`import.meta.env.VITE_API_URL` / `import.meta.env.VITE_WS_URL`) or defaults to `http://localhost:8024`.

---

## 4. Key Functional Modules

### 4.1 Authentication & Multi-Tenancy (`/login`, `/register`, `AdminClients.tsx`)
- Role-based dashboard view (`admin` vs `client`).
- Sessions persist JWT tokens and user metadata in `localStorage`.
- Role-gated navigation: Admins see client management, LLM model provisioning, and connector approval tooling. Clients only see mailbox telemetry, inbox, review drafts, and their tenant settings.

### 4.2 Inbox & Email Processing Monitor (`/inbox`, `/ai-processing`)
- Live feed of emails processed by the Celery pipeline.
- Visual status pills: `auto_sent`, `ticket_created`, `paused`, `blocked_keyword`, `automation_halted`, `pending_manual_review`.
- Multi-tab management: **All Inboxes**, **Blocked by Keywords**, and **Paused Senders**.
- Manual reply console with automated SMTP dispatch and status clearing.

### 4.3 Drafts & Human-in-the-Loop Review Queue (`/drafts`)
- Dedicated workflow for low-confidence AI replies or accounts configured with `feature_auto_send: false` (Draft Mode).
- Filter by sentiment, intent, confidence score range, client, and search terms.
- Full draft editor: Modify subject and generated response with live validation.
- Actions: Individual **Send**, **Discard**, or **Batch Send** selected / filtered drafts.
- Real-time polling badge in sidebar showing active pending drafts requiring action.

### 4.4 Dynamic System Connectors (`/payloads`)
- Client and Admin studio for managing CRM webhook connectors (`connector_configs`).
- Supports `GET` and `POST` methods, URL allowlist validation, header templates, request payload templating, and JMESPath response mapping.
- **Auth Modes:** Bearer Token, Basic Auth, API Key (Header/Query), and OAuth 2.0 Client Credentials.
- **AI Preview Assistant:** Generate connector configurations from plain text CRM documentation via LLM prompt preview.
- **Lifecycle Actions:** Draft, Request Approval, Admin Approve, Reject, Takedown, Request Deletion, and Regenerate (zero-downtime reconfiguration).

### 4.5 RAG Knowledge Base (`/knowledge`)
- Upload documents (PDF, TXT, Excel/Spreadsheets).
- Semantic lookup sandbox: Query the Qdrant vector database in real-time to inspect retrieval chunks and similarity scores.
- Document management: Delete documents or view indexed chunk counts.

### 4.6 LLM & Model Routing Studio (`/admin/llm-configs`, `/llm-analytics`)
- Configure global defaults and per-tenant LLM provider overrides (`Groq`, `OpenAI`, `Anthropic Claude`, `Google Gemini`, `xAI Grok`, `Azure OpenAI`).
- Test API keys and dynamically fetch available models from provider endpoints.
- Analytics charts: Token consumption, prompt vs completion tokens, estimated USD cost, latency breakdowns, and model usage distribution.

### 4.7 Mailbox Accounts & Settings (`/accounts`, `/settings`)
- Register IMAP/SMTP mail server credentials (host, port, SSL, user, app passwords).
- Configure reply confidence score thresholds (0–100 scale).
- Manage blocked keyword dictionaries and automated disclaimer text snippets appended to outbound replies.

---

## 5. Getting Started (Development & Production)

### Prerequisites
- Node.js (v18 or v20 recommended)
- npm or pnpm
- Running backend instance (default: `http://localhost:8024`)

### Installation
```bash
# Clone or navigate to the frontend directory
cd /home/hyper_is_op/Smart_Mail_Agent_FE

# Install dependencies
npm install
# or
pnpm install
```

### Running Local Development Server
```bash
npm run dev
```
Development server spins up at: `http://localhost:1947/Smart_Mail_agent/` (proxies `/api` and `/ws` to `http://127.0.0.1:8024`).

### Production Build
```bash
npm run build
```
This runs TypeScript checks (`tsc`) and bundles optimized production assets into `dist/`.

### Deployment to Web Server (e.g., Nginx / Apache)
1. Build the production bundle:
   ```bash
   npm run build
   ```
2. Deploy the contents of `dist/` to your web server root (e.g. `/var/www/html/Smart_Mail_Agent`):
   ```bash
   cp -r dist/* /var/www/html/Smart_Mail_Agent/
   ```
3. Ensure `config.js` inside `/var/www/html/Smart_Mail_Agent/config.js` points to the target backend API host.

#### Sample Nginx Configuration Snippet:
```nginx
location /Smart_Mail_agent {
    alias /var/www/html/Smart_Mail_Agent;
    index index.html;
    try_files $uri $uri/ /Smart_Mail_agent/index.html;
}
```

---

## 6. Testing & Quality Assurance
- **TypeScript Compilation:** `npx tsc --noEmit`
- **Linting:** `npm run lint`
- **Build Verification:** `npm run build`
