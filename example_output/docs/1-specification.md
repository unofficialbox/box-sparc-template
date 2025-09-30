# SPARC Framework: Box Demo Web Application

## Step 1: Specification

### Project Overview

**Project Name:** Box Demo Web Applications

**Project Goal:** Develop a demo application that replicates the look and feel of the Box Webapp, enabling Solutions Engineers to demonstrate Box's core file management capabilities in a lightweight, easily deployable environment.

**Target Audience:** 
- Full-stack developers
- Designers
- Solutions Engineers (non-technical users who will run the application locally)

**Context:** This application serves as a demonstration tool that mimics the Box web interface, allowing teams to showcase Box's features without requiring full production infrastructure. The focus is on visual fidelity and ease of use rather than complete feature parity.

---

### Functional Requirements

Based on the provided requirements and the reference screenshot of the Box webapp, the application must implement the following features:

#### FR-1: Left Sidebar Navigation
**Priority:** Critical  
**Description:** A fixed left sidebar containing:
- Box logo at the top
- "All Files" menu option (primary navigation item)
- Visual styling matching Box's blue gradient background (#0061D5)
- Responsive hover states and selection indicators

**Acceptance Criteria:**
- Sidebar is always visible and fixed to the left edge
- Logo is properly sized (minimum 50px width as per style guide)
- "All Files" option is clearly labeled with appropriate icon
- Selected state uses Box Blue (#0061D5)

#### FR-2: File/Folder List View (Center Panel)
**Priority:** Critical  
**Description:** The main content area displaying folder contents with:
- Breadcrumb navigation showing current folder path
- Column headers: NAME, UPDATED, SIZE
- Sortable columns (especially UPDATED with sort indicator)
- Row items showing:
  - File/folder icon (yellow folder icon, red PDF icon, etc.)
  - File/folder name
  - Last updated date and user
  - File size (for files) or item count (for folders)
- View toggle buttons (grid/list view)
- Search bar at the top
- "New" and "Share" action buttons in the header

**Acceptance Criteria:**
- Displays hierarchical folder structure
- Supports navigation into folders via click
- Shows appropriate icons for different content types
- Implements sorting by clicking column headers
- Displays proper date formatting
- Shows file sizes in appropriate units (KB, MB, GB)

#### FR-3: Right Sidebar Details Panel
**Priority:** Critical  
**Description:** A collapsible right sidebar showing detailed information about selected items:
- Toggle between "Sharing" and "Details" tabs
- Details tab includes:
  - Sync to Desktop toggle
  - Classification section with "Add" link
  - Folder Properties:
    - Description field
    - Owner name
    - Created timestamp
    - Modified timestamp
    - Size information
  - Metadata section with "Add" link
  - Workflows section

**Acceptance Criteria:**
- Panel toggles between Sharing and Details views
- All metadata fields are clearly labeled
- Timestamps use consistent date formatting
- Empty states show helpful placeholder text
- "Add" links are interactive

#### FR-4: Context Menu (Right-Click)
**Priority:** High  
**Description:** Right-click functionality on files and folders showing contextual actions:
- Common actions: Open, Download, Share, Delete, Rename, Copy, Move
- Additional options based on item type
- Keyboard shortcut indicators where applicable

**Acceptance Criteria:**
- Menu appears on right-click
- Menu positioned near cursor
- Actions are contextually appropriate
- Menu closes on outside click or action selection
- Keyboard navigation support (Esc to close)

---

### Non-Functional Requirements

#### NFR-1: Lightweight Architecture
**Description:** Minimize dependencies and resource consumption
- No Kubernetes orchestration required
- No Docker containerization required
- No external database systems (use file-based or in-memory storage)
- No Redis or other caching layers
- Target memory footprint: < 200MB RAM when running
- Fast startup time: < 5 seconds

#### NFR-2: Ease of Deployment
**Description:** Enable non-technical users to run the application
- Single command startup (e.g., `npm start` or `./run.sh`)
- Clear README with step-by-step instructions
- Environment variable configuration via `.env` file
- Automatic dependency installation
- Graceful error messages with troubleshooting tips

#### NFR-3: High Performance
**Description:** Ensure responsive user experience
- Initial page load: < 2 seconds
- API response time: < 500ms for folder listing
- Smooth UI interactions (60fps animations)
- Efficient rendering for lists with 100+ items
- Lazy loading for images and large datasets

#### NFR-4: Box Integration
**Description:** Authenticate and interact with Box APIs
- Use Client Credentials Grant (CCG) authentication
- Subject type: "user"
- Support for:
  - Folder listing
  - File metadata retrieval
  - Folder navigation
  - File operations (download, upload - optional for v1)

#### NFR-5: Demo Application Status
**Description:** Acknowledge this is a demonstration tool
- Not production-ready (no comprehensive error handling required)
- Simplified security model
- Mock data acceptable where Box API integration is complex
- Focus on visual presentation over business logic

#### NFR-6: Browser Compatibility
**Description:** Support modern browsers
- Chrome/Edge (Chromium) - primary target
- Firefox - secondary support
- Safari - best effort
- No IE11 support required

---

### User Scenarios (Jobs-to-be-Done Framework)

#### Scenario 1: Navigate Folder Structure
**As a** Solutions Engineer demonstrating Box  
**I want to** browse through folders and subfolders  
**So that** I can show clients how Box organizes content hierarchically

**User Flow:**
1. User opens application to default "All Files" view
2. User sees list of top-level folders and files
3. User clicks on a folder row
4. Application navigates into folder, updating breadcrumb and file list
5. User clicks breadcrumb to navigate back up the hierarchy

#### Scenario 2: View File/Folder Details
**As a** Solutions Engineer demonstrating Box  
**I want to** display detailed metadata about files and folders  
**So that** I can highlight Box's metadata and governance capabilities

**User Flow:**
1. User clicks on a file or folder in the list
2. Right sidebar highlights the "Details" tab
3. Details panel shows comprehensive information:
   - Owner, dates, size
   - Classification (with ability to add)
   - Metadata fields
   - Workflows
4. User clicks "Add" to demonstrate classification/metadata features

#### Scenario 3: Perform Quick Actions
**As a** Solutions Engineer demonstrating Box  
**I want to** right-click on files to show available actions  
**So that** I can demonstrate Box's productivity features

**User Flow:**
1. User right-clicks on a file or folder
2. Context menu appears with relevant actions
3. User hovers over options to see descriptions
4. User clicks an action or presses Esc to close

#### Scenario 4: Search for Content
**As a** Solutions Engineer demonstrating Box  
**I want to** search for files and folders by name  
**So that** I can show how quickly users can find content

**User Flow:**
1. User clicks search bar at top
2. User types search query
3. Results filter in real-time
4. User clicks result to open/navigate

#### Scenario 5: Create Branch for Modifications
**As a** user  
**I want to** create a branch of the existing approved branch  
**So that** I can create modifications without impacting other users

**User Flow:**
1. User accesses version control or branching feature (if implemented)
2. User selects "Create Branch" option
3. User provides branch name
4. System creates isolated copy for modifications
5. User can switch between branches

*Note: This scenario may require extended implementation in later phases.*

---

### UI/UX Guidelines

#### Design Principles
Based on the Box Brand Style Guide and reference screenshot:

1. **Visual Consistency with Box Brand**
   - Use Box Blue (#0061D5) as primary brand color
   - Implement Box Gray (#151F26) for text
   - White (#FFFFFF) backgrounds for content areas
   - Typography: Inter Display for headers, Inter for body text
   - Sentence case for all labels and headings

2. **Layout Structure**
   - **Left Sidebar:** 232px width, Box Blue gradient background
   - **Main Content:** Flexible width, white background
   - **Right Sidebar:** ~320px width, collapsible, white background with subtle border
   - **Top Bar:** 64px height, contains search, user profile, and actions

3. **Component Design**
   - **File/Folder Icons:** 
     - Folders: Yellow (#FFD700) folder icon
     - PDFs: Red (#ED3757) with PDF badge
     - Other file types: Appropriate colored icons
   - **Buttons:**
     - Primary: Box Blue background, white text
     - Secondary: White background, Box Blue border and text
     - Minimum height: 36px, border-radius: 4px
   - **Tables:**
     - Row height: 48px
     - Hover state: Light gray background (#F7F7F7)
     - Selected state: Light blue background (#F3F9FE)
     - Borders: 1px solid #E8E8E8

4. **Typography Hierarchy**
   - Page titles: Inter Display SemiBold, 24px
   - Section headers: Inter Display SemiBold, 18px
   - Body text: Inter Regular, 14px
   - Metadata labels: Inter Regular, 12px, #767676

5. **Interactive States**
   - Hover: Subtle background color change
   - Active/Selected: Box Blue accent or background
   - Disabled: 40% opacity
   - Focus: 2px Box Blue outline

6. **Spacing System**
   - Base unit: 8px
   - Small spacing: 8px
   - Medium spacing: 16px
   - Large spacing: 24px
   - XL spacing: 32px

7. **Accessibility**
   - WCAG 2.1 AA compliance for color contrast
   - Keyboard navigation support
   - ARIA labels for interactive elements
   - Focus indicators on all interactive elements

#### Validation Process with Playwright
At each development milestone:
1. Navigate to the application using `mcp__playwright__browser_navigate`
2. Take screenshots using `mcp__playwright__browser_take_screenshot`
3. Compare against reference screenshot (`1_box_webapp.png`)
4. Verify DOM structure using `mcp__playwright__browser_snapshot`
5. Check console for errors using `mcp__playwright__browser_console_messages`
6. Test responsive behavior with `mcp__playwright__browser_resize`
7. Validate interactions (clicks, hovers) using appropriate Playwright tools

---

### Technical Constraints

#### Backend
- **Language:** TypeScript
- **Runtime:** Node.js (v18+)
- **SDK:** `box-typescript-sdk-gen`
- **Framework:** Minimal framework (Express.js or similar)
- **Authentication:** Client Credentials Grant (CCG) with subject_type "user"

#### Frontend
- **Language:** TypeScript
- **Framework:** React.js (v18+)
- **Build Tool:** Vite.js
- **Styling:** CSS Modules or Styled Components (TBD in architecture phase)
- **State Management:** React Context or minimal library (Zustand/Jotai)

#### Development Environment
- **Package Manager:** npm or yarn
- **Node Version:** 18+ (LTS)
- **Platform:** macOS optimized (primary target)
- **Memory Constraint:** < 200MB for running application

---

### File Structure Proposal

```
box-vibezzz/
├── frontend/                  # React + Vite frontend
│   ├── src/
│   │   ├── components/       # Reusable UI components
│   │   │   ├── Sidebar/
│   │   │   │   ├── LeftSidebar.tsx
│   │   │   │   └── RightSidebar.tsx
│   │   │   ├── FileList/
│   │   │   │   ├── FileListView.tsx
│   │   │   │   ├── FileRow.tsx
│   │   │   │   └── ColumnHeader.tsx
│   │   │   ├── Header/
│   │   │   │   ├── TopBar.tsx
│   │   │   │   └── SearchBar.tsx
│   │   │   ├── ContextMenu/
│   │   │   │   └── ContextMenu.tsx
│   │   │   └── common/
│   │   │       ├── Button.tsx
│   │   │       ├── Icon.tsx
│   │   │       └── Toggle.tsx
│   │   ├── pages/
│   │   │   └── FileBrowser.tsx
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
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── public/
│   │   └── assets/
│   │       └── icons/
│   ├── index.html
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts
├── backend/                   # Node.js + TypeScript backend
│   ├── src/
│   │   ├── routes/
│   │   │   └── files.routes.ts
│   │   ├── services/
│   │   │   └── box.service.ts
│   │   ├── middleware/
│   │   │   └── auth.middleware.ts
│   │   ├── types/
│   │   │   └── box.types.ts
│   │   ├── config/
│   │   │   └── box.config.ts
│   │   └── server.ts
│   ├── package.json
│   └── tsconfig.json
├── docs/                      # SPARC documentation
│   ├── 1-specification.md    # This document
│   ├── 2-pseudocode.md       # To be created
│   ├── 3-architecture.md     # To be created
│   ├── 4-refinement.md       # To be created
│   └── 5-completion.md       # To be created
├── context/
│   ├── screenshots/
│   │   └── 1_box_webapp.png
│   └── style/
│       └── box_style_guide.md
├── scripts/
│   ├── setup.sh              # Initial setup script
│   └── start.sh              # Application startup script
├── .env.example              # Environment variable template
├── .gitignore
├── README.md                 # User-facing documentation
├── AGENTS.md
└── STARTING_PROMPT.md
```

---

### Assumptions

1. **Box API Access**
   - Developers will provide their own Box Client Credentials (Client ID & Client Secret)
   - Developers have access to a Box Enterprise or Business account
   - CCG authentication with user subject is properly configured in Box Admin Console

2. **User Technical Level**
   - Solutions Engineers can execute basic command-line operations (navigate directories, run scripts)
   - Solutions Engineers have Node.js installed or can follow installation instructions
   - Solutions Engineers have basic understanding of environment variables

3. **Development Environment**
   - Primary development on macOS
   - Moderately powered laptop (8GB+ RAM, dual-core+ processor)
   - Cannot consume significant memory (keep under 200MB)

4. **Scope Limitations**
   - This is a demonstration tool, not a production application
   - Not all Box features need to be implemented
   - Visual fidelity is more important than feature completeness
   - Mock data is acceptable for complex features

5. **Authentication Flow**
   - User will be authenticated as a specific Box user via CCG
   - No user login UI required (authentication happens server-side)
   - Single user session (no multi-tenancy required)

6. **Browser Environment**
   - Users will access via modern browsers (Chrome, Firefox, Safari)
   - No mobile responsiveness required for v1
   - Desktop viewport: 1280px+ width

7. **Performance**
   - Typical folder contains < 100 items
   - No infinite scroll required for v1
   - Network latency to Box API: < 1 second

---

### Reflection

#### Why These Requirements?

**Functional Requirements Rationale:**
The four core functional requirements (left sidebar, file list, right details panel, context menu) were derived directly from the Box webapp screenshot and represent the minimum viable interface for demonstrating Box's file management capabilities. These elements are the most visually recognizable aspects of the Box interface and provide the foundation for common demonstration scenarios.

**Non-Functional Requirements Rationale:**
The constraints around lightweight deployment (no K8s, Docker, databases) directly address the target user: Solutions Engineers who need to quickly spin up demos on their laptops. This audience requires simplicity over sophistication. The memory and performance constraints ensure the app can run alongside other demo tools (presentations, video conferencing) without degrading the laptop experience.

**Technical Stack Rationale:**
TypeScript provides type safety critical for SDK integration and reduces runtime errors. React + Vite offers fast development iteration and excellent developer experience. The `box-typescript-sdk-gen` ensures we're using Box's latest, officially supported SDK. Avoiding heavy frameworks keeps the application lean.

#### Potential Challenges & Mitigation

**Challenge 1: Visual Fidelity vs. Development Time**
- *Risk:* Perfectly replicating Box's UI could be time-consuming
- *Mitigation:* Focus on "close enough" visual match; use CSS variables to easily adjust styles; consider using Box UI Elements library for certain components (though this may add weight)

**Challenge 2: Box API Rate Limiting**
- *Risk:* Frequent API calls during demos could hit rate limits
- *Mitigation:* Implement simple in-memory caching for folder listings; add cache invalidation on actions; display cached data age to users

**Challenge 3: CCG Authentication Setup**
- *Risk:* Non-technical users may struggle with Box Admin Console configuration
- *Mitigation:* Provide detailed step-by-step guide with screenshots in README; include troubleshooting section; validate credentials on startup with helpful error messages

**Challenge 4: Cross-Platform Compatibility**
- *Risk:* While macOS is primary target, some users may be on Windows
- *Mitigation:* Write platform-agnostic TypeScript; test startup scripts on both platforms; use Node.js built-ins over platform-specific commands

**Challenge 5: Keeping Bundle Size Small**
- *Risk:* Dependencies can quickly balloon
- *Mitigation:* Audit dependencies regularly; use tree-shaking with Vite; consider preact as React alternative if bundle size becomes an issue; lazy load components

#### Alternative Approaches Considered

**1. Full-Stack Framework (Next.js) vs. Separate Frontend/Backend**
- *Chosen:* Separate frontend (React+Vite) and backend (Node+Express)
- *Alternative:* Next.js for SSR and API routes
- *Reasoning:* Next.js adds complexity and build overhead. Separate services provide clearer separation of concerns and simpler deployment model. For a demo app, we don't need SSR benefits.

**2. Mock Data vs. Live Box Integration**
- *Chosen:* Live Box integration with optional mock fallback
- *Alternative:* Fully mocked data without Box SDK
- *Reasoning:* Live integration provides authentic demonstration experience and allows showing real customer data. Mock fallback ensures app works without credentials for development.

**3. CSS Approach: Styled Components vs. CSS Modules vs. Tailwind**
- *Chosen:* TBD in Architecture phase, leaning toward CSS Modules
- *Alternatives:* 
  - Styled Components: Runtime overhead, larger bundle
  - Tailwind: Requires build step, verbose class names
  - CSS Modules: Scoped styles, no runtime, minimal bundle impact
- *Reasoning:* CSS Modules offer best balance of developer experience, performance, and bundle size for this use case.

**4. State Management: Context API vs. Redux vs. Zustand**
- *Chosen:* React Context for simple state, Zustand if complexity grows
- *Alternatives:*
  - Redux: Too heavy for demo app
  - MobX: Adds complexity
  - Zustand: Lightweight, simple API
- *Reasoning:* Context API sufficient for file navigation and selection state. Zustand available if state becomes more complex.

#### How This Supports Project Goals

Each specification element ladders up to the core goal: **creating a demo app that looks and feels like Box webapp**.

- **Visual Requirements:** Directly address "looks like Box" through strict adherence to Box Brand Guidelines
- **Functional Requirements:** Ensure "feels like Box" by implementing core interaction patterns users expect
- **Non-Functional Requirements:** Enable Solutions Engineers to "easily deploy and run" on their laptops
- **User Scenarios:** Define the demonstration flows Solutions Engineers will actually perform

The specification balances **visual authenticity** (detailed design guidelines, brand colors) with **practical constraints** (lightweight, no complex infrastructure) to create a tool that's both impressive in demos and practical to deploy.

#### Next Steps

With this comprehensive specification in place, we can proceed to:
1. **Pseudocode (Step 2):** Translate these requirements into high-level code structures
2. **Architecture (Step 3):** Make concrete technology choices and define system interactions
3. **Refinement (Step 4):** Optimize and iterate on the design
4. **Completion (Step 5):** Implement and test the application

The specification provides a clear blueprint that reduces ambiguity and ensures all stakeholders (developers, designers, Solutions Engineers) have a shared understanding of what will be built.
