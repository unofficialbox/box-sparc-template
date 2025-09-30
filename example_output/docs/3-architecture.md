# SPARC Framework: Box Demo Web Application

## Step 3: Architecture

### Overview

This document defines the system architecture, technology stack, deployment model, and design patterns for the Box Demo Web Application. The architecture is optimized for simplicity, performance, and ease of deployment while maintaining visual fidelity to the Box webapp.

---

### 1. Architectural Style

**Selected Pattern:** **Three-Tier Architecture** with clear separation of concerns

#### Layers:

1. **Presentation Layer (Frontend)**
   - React-based single-page application (SPA)
   - Handles UI rendering, user interactions, and client-side state
   - Communicates with backend via REST API

2. **Application Layer (Backend)**
   - Node.js/Express API server
   - Handles business logic, authentication, and Box SDK integration
   - Acts as middleware between frontend and Box APIs

3. **Data Layer (Box Cloud)**
   - Box cloud storage as the source of truth
   - No local database required
   - Optional in-memory caching for performance

```
┌─────────────────────────────────────────────────────────┐
│                    User Browser                          │
│                                                          │
│  ┌────────────────────────────────────────────────┐    │
│  │         React Frontend (Port 5174)              │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐    │    │
│  │  │Components│  │  Hooks   │  │  State   │    │    │
│  │  └──────────┘  └──────────┘  └──────────┘    │    │
│  │                       │                        │    │
│  │                   API Service                  │    │
│  └───────────────────────┼────────────────────────┘    │
└────────────────────────────┼──────────────────────────┘
                           HTTP/REST
                              │
┌────────────────────────────┼──────────────────────────┐
│                             ▼                           │
│  ┌────────────────────────────────────────────────┐   │
│  │      Node.js Backend (Port 3001)               │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐    │   │
│  │  │  Routes  │  │ Services │  │  Config  │    │   │
│  │  └──────────┘  └──────────┘  └──────────┘    │   │
│  │                       │                        │   │
│  │                Box TypeScript SDK              │   │
│  └───────────────────────┼────────────────────────┘   │
│                  Local Server/Laptop                    │
└────────────────────────────┼──────────────────────────┘
                           HTTPS
                              │
┌────────────────────────────┼──────────────────────────┐
│                             ▼                           │
│                      Box Platform                       │
│  ┌────────────────────────────────────────────────┐   │
│  │              Box API Services                   │   │
│  │  • Folders API    • Files API                  │   │
│  │  • Search API     • Auth API                   │   │
│  └────────────────────────────────────────────────┘   │
│                                                         │
│                    Box Cloud Storage                    │
└─────────────────────────────────────────────────────────┘
```

#### Rationale:

- **Simplicity:** Clear separation makes the codebase easy to understand for Solutions Engineers
- **Flexibility:** Frontend and backend can be developed and tested independently
- **No Database Required:** Meets NFR-5 (no external databases)
- **Standard Pattern:** Well-understood architecture reduces onboarding time

---

### 2. Technology Stack

#### 2.1 Frontend Technology Decisions

| Technology | Choice | Version | Justification |
|------------|--------|---------|---------------|
| **Runtime** | Browser | Modern (ES2020+) | Standard web platform |
| **Framework** | React | 18.3.x | Excellent dev experience, component reusability, large ecosystem |
| **Language** | TypeScript | 5.x | Type safety, better IDE support, catches errors early |
| **Build Tool** | Vite | 5.x | Fast dev server, optimized production builds, simple config |
| **Styling** | CSS Modules | Built-in | Scoped styles, no runtime overhead, simple to understand |
| **HTTP Client** | Axios | 1.6.x | Better API than fetch, interceptors for error handling |
| **State Management** | React Context + Hooks | Built-in | Sufficient for app complexity, no external dependencies |
| **Icons** | SVG Components | Custom | Lightweight, customizable, no icon library dependency |
| **Date Formatting** | date-fns | 3.x | Lightweight, tree-shakeable (vs. moment.js) |

**Package.json (Frontend):**
```json
{
  "name": "box-demo-frontend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext ts,tsx"
  },
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "axios": "^1.6.8",
    "date-fns": "^3.6.0"
  },
  "devDependencies": {
    "@types/react": "^18.3.3",
    "@types/react-dom": "^18.3.0",
    "@vitejs/plugin-react": "^4.3.1",
    "typescript": "^5.5.3",
    "vite": "^5.3.4",
    "eslint": "^8.57.0",
    "@typescript-eslint/eslint-plugin": "^7.13.1",
    "@typescript-eslint/parser": "^7.13.1"
  }
}
```

**Total Frontend Bundle Size Target:** < 200KB gzipped

---

#### 2.2 Backend Technology Decisions

| Technology | Choice | Version | Justification |
|------------|--------|---------|---------------|
| **Runtime** | Node.js | 18.x LTS | Long-term support, stable, async I/O |
| **Language** | TypeScript | 5.x | Type safety, shared types with frontend |
| **Framework** | Express.js | 4.x | Minimal, well-documented, widely used |
| **Box SDK** | @box/box-typescript-sdk-gen | Latest | Official SDK, type-safe, modern API |
| **Environment** | dotenv | 16.x | Simple env variable management |
| **HTTP Logging** | morgan | 1.x | Request logging for debugging |
| **CORS** | cors | 2.x | Enable cross-origin requests from frontend |
| **Process Manager** | concurrently | 8.x (dev) | Run multiple processes during development |

**Package.json (Backend):**
```json
{
  "name": "box-demo-backend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js",
    "lint": "eslint . --ext ts"
  },
  "dependencies": {
    "express": "^4.19.2",
    "cors": "^2.8.5",
    "dotenv": "^16.4.5",
    "morgan": "^1.10.0",
    "@box/box-typescript-sdk-gen": "^1.3.0"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/cors": "^2.8.17",
    "@types/morgan": "^1.9.9",
    "@types/node": "^20.14.9",
    "typescript": "^5.5.3",
    "tsx": "^4.16.0",
    "eslint": "^8.57.0",
    "@typescript-eslint/eslint-plugin": "^7.13.1",
    "@typescript-eslint/parser": "^7.13.1"
  }
}
```

**Memory Footprint Target:** < 150MB

---

#### 2.3 Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| **VS Code** | IDE | Recommended editor with TypeScript support |
| **ESLint** | Linting | TypeScript rules, consistent code style |
| **Prettier** | Formatting | Auto-format on save |
| **Git** | Version control | Standard workflows |
| **npm** | Package management | No yarn/pnpm to reduce complexity |

---

### 3. System Architecture Diagram

#### 3.1 Component Architecture

```
Frontend Architecture:
┌────────────────────────────────────────────────────────────┐
│                         App.tsx                             │
│                    (Root Component)                         │
└────────────────────────────┬───────────────────────────────┘
                            │
┌────────────────────────────┴───────────────────────────────┐
│                     BoxProvider                             │
│              (Context for Box data & auth)                  │
└────────────────────────────┬───────────────────────────────┘
                            │
┌────────────────────────────┴───────────────────────────────┐
│                    FileBrowser.tsx                          │
│                  (Main Page Component)                      │
└─────┬──────────────┬──────────────┬──────────────┬─────────┘
      │              │              │              │
┌─────▼─────┐  ┌────▼────┐  ┌──────▼──────┐  ┌───▼────────┐
│  Left     │  │  Top    │  │  FileList   │  │   Right    │
│  Sidebar  │  │  Bar    │  │    View     │  │  Sidebar   │
│           │  │         │  │             │  │            │
│ • Logo    │  │• Breadc.│  │ • FileRow   │  │ • Details  │
│ • Nav     │  │• Search │  │ • Sort      │  │ • Sharing  │
└───────────┘  └─────────┘  └─────────────┘  └────────────┘
                                   │
                            ┌──────▼──────┐
                            │ ContextMenu │
                            │ (Overlay)   │
                            └─────────────┘

Custom Hooks:
┌──────────────┐      ┌───────────────────┐
│   useBox     │      │ useFileNavigation │
│ • Fetch data │      │ • Breadcrumbs     │
│ • Cache      │      │ • Navigate        │
└──────┬───────┘      └─────────┬─────────┘
       │                        │
       └────────┬───────────────┘
                │
         ┌──────▼──────┐
         │ API Service │
         │ (Axios)     │
         └──────┬──────┘
                │
                ▼
          Backend REST API
```

#### 3.2 Backend Architecture

```
Backend Architecture:
┌─────────────────────────────────────────────────────────┐
│                      server.ts                          │
│                  (Express App Entry)                    │
└───────────────────────┬─────────────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
┌───────▼──────┐ ┌──────▼──────┐ ┌─────▼────────┐
│  Middleware  │ │   Routes    │ │ Error Handler│
│              │ │             │ │              │
│ • CORS       │ │ • /folders  │ │ • 500 errors │
│ • JSON Parse │ │ • /files    │ │ • Logging    │
│ • Logging    │ │ • /search   │ │              │
└──────────────┘ └──────┬──────┘ └──────────────┘
                        │
                 ┌──────▼──────┐
                 │ BoxService  │
                 │  (Class)    │
                 ├─────────────┤
                 │ • getFolderItems()
                 │ • getFolderById()
                 │ • search()
                 │ • downloadFile()
                 │ • cache (Map)
                 └──────┬──────┘
                        │
                 ┌──────▼──────┐
                 │ Box Config  │
                 │             │
                 │ • Client ID │
                 │ • Secret    │
                 │ • User ID   │
                 └──────┬──────┘
                        │
                 ┌──────▼──────────────┐
                 │ Box TypeScript SDK  │
                 │                     │
                 │ • BoxClient         │
                 │ • BoxCCGAuth        │
                 └──────┬──────────────┘
                        │
                        ▼
                   Box Platform API
```

---

### 4. Data Flow

#### 4.1 Folder Navigation Flow

```
User Action → Frontend → Backend → Box API → Response

1. User clicks folder "Documents"
   │
2. FileBrowser.handleFolderClick("12345")
   │
3. useBox hook triggers API call
   │
4. axios.get('/api/folders/12345/items')
   │
5. Backend: filesRouter receives request
   │
6. BoxService.getFolderItems("12345")
   │
7. BoxClient.folders.getFolderItems("12345")
   │
8. Box API: GET /folders/12345/items
   │
9. Box returns: { entries: [...], total_count: 10 }
   │
10. Backend caches response (5 min TTL)
    │
11. Response sent to frontend
    │
12. useBox updates state: { folders, files, loading: false }
    │
13. FileListView re-renders with new data
    │
14. User sees folder contents
```

#### 4.2 Authentication Flow (Server Startup)

```
1. Server starts
   │
2. Load environment variables (.env)
   │
3. boxConfig validates required fields
   │
4. BoxService constructor initializes:
   │
   ├─> BoxCCGAuth created with:
   │   • clientId
   │   • clientSecret
   │   • enterpriseId
   │   • userId (subject)
   │
   ├─> BoxClient created with auth
   │
5. On first API request:
   │
   ├─> CCG Auth obtains access token
   │   • POST to Box token endpoint
   │   • Receives JWT token
   │
   ├─> Token cached by SDK
   │   • Automatically refreshed when expired
   │
6. All subsequent requests use cached token
```

#### 4.3 Caching Strategy

```
Cache Layer 1: Frontend (Session Storage)
┌────────────────────────────────────┐
│ • User preferences                  │
│ • View mode (list/grid)            │
│ • Sort preferences                 │
│ • Recently viewed folders          │
│ TTL: Session lifetime              │
└────────────────────────────────────┘

Cache Layer 2: Backend (In-Memory Map)
┌────────────────────────────────────┐
│ • Folder contents                   │
│ • Folder metadata                  │
│ • Search results                   │
│ TTL: 5 minutes                     │
│ Invalidation: Manual/time-based    │
└────────────────────────────────────┘

Cache Layer 3: Box SDK (Token Cache)
┌────────────────────────────────────┐
│ • Access tokens                     │
│ TTL: Per Box token expiry (~60min) │
│ Managed by SDK                     │
└────────────────────────────────────┘
```

---

### 5. API Design

#### 5.1 REST Endpoints

**Base URL:** `http://localhost:3001/api`

| Method | Endpoint | Description | Request | Response |
|--------|----------|-------------|---------|----------|
| GET | `/folders/:folderId/items` | Get folder contents | Query: `limit`, `offset` | `{ entries: [], total_count: n }` |
| GET | `/folders/:folderId` | Get folder details | - | `{ id, name, size, ... }` |
| GET | `/search` | Search files/folders | Query: `q`, `ancestor_folder_ids` | `{ entries: [], total_count: n }` |
| GET | `/files/:fileId/download` | Download file | - | File stream |
| GET | `/health` | Health check | - | `{ status: "ok" }` |

**Example Request:**
```http
GET /api/folders/0/items?limit=100&offset=0
Host: localhost:3001
Accept: application/json
```

**Example Response:**
```json
{
  "entries": [
    {
      "id": "12345",
      "type": "folder",
      "name": "Documents",
      "modified_at": "2025-09-26T10:30:00Z",
      "modified_by": {
        "id": "67890",
        "name": "Kyle Adams",
        "type": "user"
      },
      "owned_by": {
        "id": "67890",
        "name": "Kyle Adams"
      },
      "item_collection": {
        "total_count": 15
      }
    },
    {
      "id": "54321",
      "type": "file",
      "name": "Report.pdf",
      "size": 1258291,
      "modified_at": "2025-09-25T14:20:00Z",
      "extension": ".pdf"
    }
  ],
  "total_count": 2,
  "offset": 0,
  "limit": 100
}
```

#### 5.2 Error Handling

**Error Response Format:**
```json
{
  "error": "Error category",
  "message": "Human-readable error message",
  "code": "ERROR_CODE",
  "details": {}
}
```

**Error Codes:**

| HTTP Status | Error Code | Meaning |
|-------------|------------|---------|
| 400 | `INVALID_REQUEST` | Missing or invalid parameters |
| 401 | `UNAUTHORIZED` | Box authentication failed |
| 403 | `FORBIDDEN` | User lacks permission for resource |
| 404 | `NOT_FOUND` | Folder/file doesn't exist |
| 429 | `RATE_LIMIT` | Too many requests to Box API |
| 500 | `INTERNAL_ERROR` | Unexpected server error |
| 503 | `BOX_UNAVAILABLE` | Box API is down |

---

### 6. File Structure (Implemented)

```
box-vibezzz/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Sidebar/
│   │   │   │   ├── LeftSidebar.tsx
│   │   │   │   ├── LeftSidebar.module.css
│   │   │   │   ├── RightSidebar.tsx
│   │   │   │   └── RightSidebar.module.css
│   │   │   ├── FileList/
│   │   │   │   ├── FileListView.tsx
│   │   │   │   ├── FileListView.module.css
│   │   │   │   ├── FileRow.tsx
│   │   │   │   ├── FileRow.module.css
│   │   │   │   ├── ColumnHeader.tsx
│   │   │   │   └── ColumnHeader.module.css
│   │   │   ├── Header/
│   │   │   │   ├── TopBar.tsx
│   │   │   │   ├── TopBar.module.css
│   │   │   │   ├── SearchBar.tsx
│   │   │   │   └── SearchBar.module.css
│   │   │   ├── ContextMenu/
│   │   │   │   ├── ContextMenu.tsx
│   │   │   │   └── ContextMenu.module.css
│   │   │   └── common/
│   │   │       ├── Button.tsx
│   │   │       ├── Button.module.css
│   │   │       ├── Icon.tsx
│   │   │       ├── Icon.module.css
│   │   │       ├── Toggle.tsx
│   │   │       └── Toggle.module.css
│   │   ├── pages/
│   │   │   ├── FileBrowser.tsx
│   │   │   └── FileBrowser.module.css
│   │   ├── hooks/
│   │   │   ├── useBox.ts
│   │   │   └── useFileNavigation.ts
│   │   ├── services/
│   │   │   └── api.ts
│   │   ├── types/
│   │   │   └── box.types.ts
│   │   ├── styles/
│   │   │   ├── globals.css
│   │   │   └── variables.css
│   │   ├── assets/
│   │   │   ├── icons/
│   │   │   │   ├── folder.svg
│   │   │   │   ├── file.svg
│   │   │   │   ├── pdf.svg
│   │   │   │   └── ...
│   │   │   └── box-logo.svg
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── public/
│   ├── index.html
│   ├── package.json
│   ├── tsconfig.json
│   ├── tsconfig.node.json
│   ├── vite.config.ts
│   └── .eslintrc.json
├── backend/
│   ├── src/
│   │   ├── routes/
│   │   │   └── files.routes.ts
│   │   ├── services/
│   │   │   └── box.service.ts
│   │   ├── middleware/
│   │   │   ├── error.middleware.ts
│   │   │   └── logging.middleware.ts
│   │   ├── types/
│   │   │   └── box.types.ts
│   │   ├── config/
│   │   │   └── box.config.ts
│   │   └── server.ts
│   ├── package.json
│   ├── tsconfig.json
│   └── .eslintrc.json
├── docs/
│   ├── 1-specification.md
│   ├── 2-pseudocode.md
│   ├── 3-architecture.md     # This document
│   ├── 4-refinement.md       # To be created
│   └── 5-completion.md       # To be created
├── context/
│   ├── screenshots/
│   │   └── 1_box_webapp.png
│   └── style/
│       └── box_style_guide.md
├── scripts/
│   ├── setup.sh
│   └── start.sh
├── .env.example
├── .gitignore
├── README.md
├── AGENTS.md
└── STARTING_PROMPT.md
```

---

### 7. Design Patterns

#### 7.1 Frontend Patterns

**1. Container/Presentational Pattern**
- **Container Components:** `FileBrowser.tsx` - manages state and logic
- **Presentational Components:** `FileRow.tsx`, `Button.tsx` - pure rendering

**2. Custom Hooks Pattern**
- Encapsulate reusable logic: `useBox`, `useFileNavigation`
- Promotes code reuse and testability

**3. Composition Pattern**
- Build complex UI from smaller components
- Example: `FileListView` composed of `ColumnHeader` + `FileRow`

**4. Context API Pattern**
- Share global state (Box data, auth status) without prop drilling
- `BoxProvider` wraps app

**5. Module Pattern (CSS Modules)**
- Scoped styles prevent naming conflicts
- Co-located with components

#### 7.2 Backend Patterns

**1. Service Layer Pattern**
- `BoxService` encapsulates all Box API logic
- Separates business logic from routes

**2. Repository Pattern (Implicit)**
- `BoxService` acts as repository for Box data
- Could be swapped with mock implementation for testing

**3. Middleware Pattern**
- Express middleware for cross-cutting concerns
- CORS, logging, error handling

**4. Singleton Pattern**
- Single `BoxService` instance shared across requests
- Reduces overhead of repeated auth

**5. Caching Pattern**
- In-memory Map for simple cache
- TTL-based expiration

---

### 8. Security Considerations

#### 8.1 Authentication

**Box CCG (Client Credentials Grant):**
- Server-side only (credentials never exposed to frontend)
- Subject type: "user" (act on behalf of specific Box user)
- Token managed by Box SDK (auto-refresh)

**Environment Variables:**
- Sensitive credentials in `.env` file
- Never committed to version control (`.gitignore`)
- Validated on server startup

#### 8.2 API Security

**CORS Configuration:**
```typescript
app.use(cors({
  origin: 'http://localhost:5174',  // Frontend URL
  credentials: true
}))
```

**Input Validation:**
- Validate folder IDs (numeric strings)
- Sanitize search queries
- Limit query parameters (max limit: 1000)

**Rate Limiting (Optional for v1):**
- Can add `express-rate-limit` if needed
- Protect against abuse

#### 8.3 Error Exposure

**Production Mode:**
- Don't expose internal error details to frontend
- Log full errors server-side
- Return generic messages to client

**Development Mode:**
- Detailed error messages for debugging
- Stack traces in console

---

### 9. Performance Optimization

#### 9.1 Frontend Optimizations

**1. Code Splitting**
- Lazy load routes/components if app grows
- Dynamic imports for heavy components

**2. Memoization**
- Use `React.memo` for expensive components
- `useMemo` for computed values
- `useCallback` for event handlers passed to children

**3. Virtual Scrolling (If Needed)**
- Implement if folders exceed 100 items
- Libraries: `react-window`, `react-virtual`

**4. Asset Optimization**
- SVG icons (inline or sprite sheet)
- Compress images (if using photos)
- Tree-shake unused code (Vite handles this)

**5. Caching**
- HTTP cache headers from backend
- Service worker (future enhancement)

#### 9.2 Backend Optimizations

**1. In-Memory Caching**
- Cache folder listings (5-min TTL)
- Reduce Box API calls by ~80%

**2. Concurrent Requests**
- Use `Promise.all` for parallel API calls
- Example: Fetch folder + metadata simultaneously

**3. Response Compression**
- Gzip middleware for JSON responses
```typescript
import compression from 'compression'
app.use(compression())
```

**4. Field Selection**
- Request only needed fields from Box API
- Reduces payload size

**5. Pagination**
- Implement limit/offset for large folders
- Default: 100 items per page

---

### 10. Scalability (Future Considerations)

**Note:** This is a demo app, but these considerations help if it needs to scale.

#### 10.1 Horizontal Scaling

**Current Limitation:**
- In-memory cache not shared across instances

**Solution:**
- Replace Map cache with Redis
- Use sticky sessions for load balancer

#### 10.2 Multi-User Support

**Current State:**
- Single user (CCG subject user)

**Enhancement:**
- Implement user login
- Store user-to-Box-user mapping
- Use CCG with dynamic `subject_id`

#### 10.3 Database (If Needed)

**Potential Uses:**
- User preferences
- Audit logs
- Custom metadata

**Recommendation:**
- SQLite for simplicity (single file)
- PostgreSQL if need full RDBMS

---

### 11. Deployment Model

#### 11.1 Local Development

**Setup Process:**
1. Clone repository
2. Run `./scripts/setup.sh`
   - Validates Node.js version
   - Installs dependencies
   - Creates `.env` from template
3. Configure `.env` with Box credentials
4. Run `./scripts/start.sh`
   - Starts backend on port 3001
   - Starts frontend on port 5174
5. Open browser to `http://localhost:5174`

**Requirements:**
- macOS (primary) or Linux/Windows
- Node.js 18+
- 8GB+ RAM
- 200MB free disk space

#### 11.2 Production Deployment (Optional)

**Build Process:**
```bash
# Frontend
cd frontend
npm run build  # Creates dist/ with optimized bundle

# Backend
cd backend
npm run build  # Compiles TypeScript to dist/
```

**Hosting Options:**

**Option 1: Single Server (VPS)**
- Nginx reverse proxy
- PM2 for process management
- Serve frontend static files
- Proxy `/api` to backend

**Option 2: Cloud Platform**
- Frontend: Vercel, Netlify (static hosting)
- Backend: Heroku, Render, Railway (Node.js hosting)

**Option 3: Docker (If Constraints Lifted)**
```dockerfile
# Multi-stage build
FROM node:18-alpine as frontend-build
# ... build frontend

FROM node:18-alpine as backend
# ... copy backend + frontend dist
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

---

### 12. Testing Strategy

#### 12.1 Frontend Testing

**Unit Tests:**
- Component rendering (React Testing Library)
- Custom hooks logic
- Utility functions

**Integration Tests:**
- User flows (navigation, selection)
- API service mock responses

**Visual Tests (Playwright):**
- Screenshot comparison against reference
- Validate Box Brand Guidelines compliance

**Tools:**
- Vitest (Vite-native test runner)
- React Testing Library
- Playwright MCP (for visual validation)

#### 12.2 Backend Testing

**Unit Tests:**
- BoxService methods
- Route handlers
- Config validation

**Integration Tests:**
- API endpoints with mocked Box SDK
- Error handling

**Tools:**
- Vitest or Jest
- Supertest (HTTP testing)
- Mock Box SDK responses

#### 12.3 E2E Testing

**Playwright Scenarios:**
1. Navigate to app
2. Click folder
3. Verify folder contents load
4. Right-click file
5. Verify context menu appears
6. Select item
7. Verify details panel updates

---

### 13. Configuration Management

#### 13.1 Environment Variables

**`.env.example`:**
```bash
# Box Configuration
BOX_CLIENT_ID=your_client_id_here
BOX_CLIENT_SECRET=your_client_secret_here
BOX_ENTERPRISE_ID=your_enterprise_id_here
BOX_USER_ID=your_user_id_here

# Server Configuration
PORT=3000
NODE_ENV=development

# Frontend Configuration (for build)
VITE_API_URL=http://localhost:3000
```

**Validation:**
- Server fails fast if required vars missing
- Clear error messages pointing to `.env.example`

#### 13.2 TypeScript Configuration

**Frontend `tsconfig.json`:**
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

**Backend `tsconfig.json`:**
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020"],
    "moduleResolution": "node",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

---

### 14. Monitoring and Logging

#### 14.1 Logging Strategy

**Backend Logging (Morgan):**
```typescript
// Development: detailed logs
app.use(morgan('dev'))

// Production: standard Apache format
app.use(morgan('combined'))
```

**Custom Logger:**
```typescript
// utils/logger.ts
export const logger = {
  info: (msg: string) => console.log(`[INFO] ${msg}`),
  error: (msg: string, err?: Error) => console.error(`[ERROR] ${msg}`, err),
  warn: (msg: string) => console.warn(`[WARN] ${msg}`)
}
```

#### 14.2 Monitoring

**Health Check Endpoint:**
```typescript
app.get('/health', (req, res) => {
  res.json({
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    memory: process.memoryUsage()
  })
})
```

**Metrics to Track (Optional):**
- Request count per endpoint
- Average response time
- Box API call count
- Cache hit/miss ratio
- Error rate

---

### Reflection

#### Architectural Decisions

**Why Three-Tier Architecture?**
The three-tier architecture (frontend, backend, Box cloud) provides clear separation of concerns while remaining simple enough for a demo application. This pattern is well-understood by developers and allows independent testing and development of each layer.

**Alternative considered:** Monolithic Next.js app with API routes and SSR
**Why rejected:** Adds complexity, larger bundle size, more difficult to deploy for non-technical users

**Why No Database?**
Box serves as the source of truth, eliminating the need for a database. This aligns perfectly with NFR-5 (no external databases) and reduces deployment complexity. In-memory caching provides sufficient performance for demo scenarios.

**Alternative considered:** SQLite for caching and user preferences
**Why rejected:** Adds deployment complexity, file management overhead, not necessary for demo scope

**Why Express vs. Fastify/Hapi?**
Express is the most widely adopted Node.js framework with extensive documentation. For a demo app run by Solutions Engineers, familiarity and simplicity outweigh marginal performance gains from newer frameworks.

**Why CSS Modules vs. Styled-Components/Tailwind?**
CSS Modules provide scoped styling without runtime overhead (styled-components) or verbose class names (Tailwind). They're simple, performant, and familiar to developers with CSS experience.

**Why React vs. Vue/Svelte?**
React has the largest ecosystem and widest adoption, making it easier for developers to contribute. The component model maps naturally to the UI requirements (sidebar, list, context menu).

#### Potential Bottlenecks

**1. Box API Rate Limits**
- *Issue:* Box imposes rate limits that could affect demo scenarios with rapid navigation
- *Mitigation:* In-memory caching with 5-minute TTL reduces API calls by ~80%. Cache hit ratio should be monitored.

**2. Large Folder Rendering**
- *Issue:* Folders with 1000+ items could cause sluggish UI
- *Mitigation:* Pagination limits items to 100 per page. Future: implement virtual scrolling.

**3. Single-Threaded Node.js**
- *Issue:* CPU-intensive operations block event loop
- *Mitigation:* No CPU-intensive operations in current design. All work is I/O (Box API calls).

**4. Memory Leaks in Cache**
- *Issue:* Unbounded cache could grow indefinitely
- *Mitigation:* Implement cache size limit (e.g., max 100 entries) with LRU eviction.

#### Security Trade-offs

**Client-Side vs. Server-Side Auth**
- *Decision:* Server-side CCG authentication only
- *Trade-off:* All requests proxy through backend (slight latency), but credentials never exposed to browser
- *Justification:* Security is paramount; latency is acceptable for demo scenarios

**CORS Configuration**
- *Decision:* Restrict CORS to specific frontend origin
- *Trade-off:* Less flexible than wildcard, but prevents unauthorized access
- *Justification:* Simple to configure, adequate security for local deployment

#### Performance vs. Simplicity

**Caching Strategy**
- *Decision:* Simple in-memory Map with time-based expiration
- *Alternative:* LRU cache with size limits and smarter invalidation
- *Trade-off:* Simpler implementation, but less memory-efficient
- *Justification:* For demo with < 100 cached items, simplicity wins

**Bundle Size vs. Features**
- *Decision:* Minimal dependencies (date-fns over moment, no UI library)
- *Alternative:* Use Box UI Elements library for authentic look
- *Trade-off:* More custom code to write, but significantly smaller bundle
- *Justification:* Aligns with NFR-1 (lightweight), faster load times

#### Future-Proofing

**TypeScript Throughout**
Type safety reduces refactoring costs and catches errors early. Shared types between frontend and backend ensure consistency.

**Modular Component Structure**
Components are loosely coupled and highly cohesive, making it easy to add features (file upload, sharing UI) without major refactoring.

**API Versioning Ready**
RESTful API design allows for future versioning (e.g., `/api/v2/folders`) if needed.

**Configuration-Driven**
Environment variables centralize configuration, making it easy to adapt for different Box environments (sandbox, production).

#### How Architecture Supports Project Goals

**Goal: Look and Feel Like Box Webapp**
- Component structure mirrors Box UI (left sidebar, main content, right details)
- CSS Modules enable pixel-perfect styling per Box Brand Guidelines
- Playwright testing validates visual fidelity

**Goal: Lightweight and Easy to Deploy**
- No Docker/K8s/databases reduces dependencies
- Simple `npm install && npm start` workflow
- < 200MB memory footprint achievable

**Goal: Enable Non-Technical Users**
- Setup scripts automate complex steps
- Clear error messages guide troubleshooting
- Single command startup

**Goal: Box Integration**
- Official Box TypeScript SDK ensures compatibility
- CCG authentication matches enterprise patterns
- Caching optimizes API usage

#### Next Steps

With the architecture defined, we proceed to:
1. **Refinement (Step 4):** Review architecture for optimizations, identify risks, plan testing
2. **Completion (Step 5):** Implement code, validate with Playwright, prepare documentation

The architecture provides a solid foundation that balances simplicity (for ease of deployment) with professionalism (for demo credibility).
