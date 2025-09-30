# SPARC Framework: Box Demo Web Application

## Step 2: Pseudocode

### Overview

This document provides high-level pseudocode serving as a development roadmap for the Box Demo Web Application. The pseudocode is organized by application layer (frontend, backend) and component, with inline comments explaining the purpose and logic.

---

### Frontend Pseudocode

#### 2.1 Main Application Entry Point

**File:** `frontend/src/main.tsx`

```typescript
// Main application entry point
IMPORT React from 'react'
IMPORT ReactDOM from 'react-dom/client'
IMPORT App from './App'
IMPORT './styles/globals.css'

// Initialize React application
FUNCTION main():
    GET root_element FROM document.getElementById('root')
    CREATE react_root FROM ReactDOM.createRoot(root_element)
    RENDER <App /> INTO react_root
END FUNCTION

EXECUTE main()
```

---

#### 2.2 Root App Component

**File:** `frontend/src/App.tsx`

```typescript
// Root application component
IMPORT FileBrowser FROM './pages/FileBrowser'
IMPORT { BoxProvider } FROM './hooks/useBox'

FUNCTION App():
    RETURN:
        <BoxProvider>
            <div className="app-container">
                <FileBrowser />
            </div>
        </BoxProvider>
END FUNCTION

EXPORT App
```

---

#### 2.3 File Browser Page

**File:** `frontend/src/pages/FileBrowser.tsx`

```typescript
// Main file browser page component
IMPORT LeftSidebar, TopBar, FileListView, RightSidebar, ContextMenu
IMPORT useFileNavigation, useBox

FUNCTION FileBrowser():
    // State management
    STATE current_folder_id = "0"  // Root folder
    STATE selected_item = null
    STATE context_menu = { visible: false, x: 0, y: 0, item: null }
    STATE view_mode = "list"  // or "grid"
    STATE sort_config = { column: "updated", direction: "desc" }
    
    // Custom hooks
    { folders, files, loading, error } = useBox(current_folder_id)
    { navigateToFolder, navigateUp, breadcrumbs } = useFileNavigation()
    
    // Event handlers
    FUNCTION handleFolderClick(folder_id):
        SET current_folder_id = folder_id
        navigateToFolder(folder_id)
        CLEAR selected_item
    END FUNCTION
    
    FUNCTION handleItemSelect(item):
        SET selected_item = item
    END FUNCTION
    
    FUNCTION handleRightClick(event, item):
        PREVENT event.default_action
        SET context_menu = {
            visible: true,
            x: event.clientX,
            y: event.clientY,
            item: item
        }
    END FUNCTION
    
    FUNCTION handleContextMenuClose():
        SET context_menu.visible = false
    END FUNCTION
    
    FUNCTION handleSort(column):
        IF sort_config.column == column:
            TOGGLE sort_config.direction
        ELSE:
            SET sort_config.column = column
            SET sort_config.direction = "desc"
        END IF
    END FUNCTION
    
    FUNCTION handleSearch(query):
        // Filter files/folders by name
        RETURN folders.FILTER(f => f.name.includes(query))
            .CONCAT(files.FILTER(f => f.name.includes(query)))
    END FUNCTION
    
    // Render layout
    RETURN:
        <div className="file-browser">
            <LeftSidebar 
                activeItem="all-files"
            />
            
            <div className="main-content">
                <TopBar 
                    breadcrumbs={breadcrumbs}
                    onSearch={handleSearch}
                    onBreadcrumbClick={navigateToFolder}
                />
                
                <FileListView
                    folders={folders}
                    files={files}
                    loading={loading}
                    error={error}
                    selectedItem={selected_item}
                    viewMode={view_mode}
                    sortConfig={sort_config}
                    onFolderClick={handleFolderClick}
                    onItemSelect={handleItemSelect}
                    onRightClick={handleRightClick}
                    onSort={handleSort}
                />
            </div>
            
            <RightSidebar
                selectedItem={selected_item}
                visible={selected_item !== null}
            />
            
            {context_menu.visible AND (
                <ContextMenu
                    x={context_menu.x}
                    y={context_menu.y}
                    item={context_menu.item}
                    onClose={handleContextMenuClose}
                />
            )}
        </div>
END FUNCTION

EXPORT FileBrowser
```

---

#### 2.4 Left Sidebar Component

**File:** `frontend/src/components/Sidebar/LeftSidebar.tsx`

```typescript
// Left navigation sidebar
IMPORT Icon FROM '../common/Icon'

INTERFACE LeftSidebarProps:
    activeItem: string

FUNCTION LeftSidebar({ activeItem }):
    // Navigation items configuration
    CONSTANT nav_items = [
        { id: "all-files", label: "All Files", icon: "folder" }
        // Additional items can be added here
    ]
    
    RETURN:
        <aside className="left-sidebar">
            <div className="sidebar-header">
                <img src="/assets/box-logo.svg" alt="Box" className="logo" />
            </div>
            
            <nav className="sidebar-nav">
                FOR EACH item IN nav_items:
                    <div 
                        className={`nav-item ${item.id === activeItem ? 'active' : ''}`}
                        key={item.id}
                    >
                        <Icon name={item.icon} />
                        <span>{item.label}</span>
                    </div>
                END FOR
            </nav>
        </aside>
END FUNCTION

EXPORT LeftSidebar
```

---

#### 2.5 Top Bar Component

**File:** `frontend/src/components/Header/TopBar.tsx`

```typescript
// Top navigation bar with breadcrumbs and actions
IMPORT SearchBar FROM './SearchBar'
IMPORT Button FROM '../common/Button'

INTERFACE TopBarProps:
    breadcrumbs: Array<{ id: string, name: string }>
    onSearch: (query: string) => void
    onBreadcrumbClick: (id: string) => void

FUNCTION TopBar({ breadcrumbs, onSearch, onBreadcrumbClick }):
    STATE user_profile = { name: "User", avatar: "/assets/avatar.png" }
    
    RETURN:
        <header className="top-bar">
            <div className="breadcrumb-section">
                FOR EACH (crumb, index) IN breadcrumbs:
                    <span 
                        className="breadcrumb-item"
                        onClick={() => onBreadcrumbClick(crumb.id)}
                    >
                        {crumb.name}
                    </span>
                    
                    IF index < breadcrumbs.length - 1:
                        <span className="breadcrumb-separator">›</span>
                    END IF
                END FOR
            </div>
            
            <div className="top-bar-actions">
                <SearchBar onSearch={onSearch} />
                
                <Button variant="secondary">New</Button>
                <Button variant="primary">Share</Button>
                
                <div className="user-profile">
                    <img src={user_profile.avatar} alt={user_profile.name} />
                </div>
            </div>
        </header>
END FUNCTION

EXPORT TopBar
```

---

#### 2.6 File List View Component

**File:** `frontend/src/components/FileList/FileListView.tsx`

```typescript
// Main file/folder list component
IMPORT FileRow FROM './FileRow'
IMPORT ColumnHeader FROM './ColumnHeader'

INTERFACE FileListViewProps:
    folders: Array<Folder>
    files: Array<File>
    loading: boolean
    error: Error | null
    selectedItem: Folder | File | null
    viewMode: "list" | "grid"
    sortConfig: { column: string, direction: "asc" | "desc" }
    onFolderClick: (id: string) => void
    onItemSelect: (item: Folder | File) => void
    onRightClick: (event: MouseEvent, item: Folder | File) => void
    onSort: (column: string) => void

FUNCTION FileListView(props):
    // Combine and sort items
    FUNCTION getSortedItems():
        CONST all_items = [...props.folders, ...props.files]
        
        RETURN all_items.SORT((a, b) => {
            CONST value_a = a[props.sortConfig.column]
            CONST value_b = b[props.sortConfig.column]
            
            IF props.sortConfig.direction === "asc":
                RETURN value_a > value_b ? 1 : -1
            ELSE:
                RETURN value_a < value_b ? 1 : -1
        })
    END FUNCTION
    
    CONST sorted_items = getSortedItems()
    
    // Render states
    IF props.loading:
        RETURN <div className="loading">Loading...</div>
    END IF
    
    IF props.error:
        RETURN <div className="error">Error: {props.error.message}</div>
    END IF
    
    IF sorted_items.length === 0:
        RETURN <div className="empty-state">This folder is empty</div>
    END IF
    
    // Render list
    RETURN:
        <div className="file-list-view">
            <div className="view-controls">
                <button onClick={() => setViewMode("list")}>List</button>
                <button onClick={() => setViewMode("grid")}>Grid</button>
            </div>
            
            <table className="file-table">
                <thead>
                    <tr>
                        <ColumnHeader 
                            label="NAME" 
                            column="name"
                            sortConfig={props.sortConfig}
                            onSort={props.onSort}
                        />
                        <ColumnHeader 
                            label="UPDATED" 
                            column="modified_at"
                            sortConfig={props.sortConfig}
                            onSort={props.onSort}
                        />
                        <ColumnHeader 
                            label="SIZE" 
                            column="size"
                            sortConfig={props.sortConfig}
                            onSort={props.onSort}
                        />
                    </tr>
                </thead>
                
                <tbody>
                    FOR EACH item IN sorted_items:
                        <FileRow
                            key={item.id}
                            item={item}
                            isSelected={props.selectedItem?.id === item.id}
                            onClick={() => {
                                IF item.type === "folder":
                                    props.onFolderClick(item.id)
                                ELSE:
                                    props.onItemSelect(item)
                                END IF
                            }}
                            onContextMenu={(e) => props.onRightClick(e, item)}
                        />
                    END FOR
                </tbody>
            </table>
        </div>
END FUNCTION

EXPORT FileListView
```

---

#### 2.7 File Row Component

**File:** `frontend/src/components/FileList/FileRow.tsx`

```typescript
// Individual file/folder row in the list
IMPORT Icon FROM '../common/Icon'

INTERFACE FileRowProps:
    item: Folder | File
    isSelected: boolean
    onClick: () => void
    onContextMenu: (event: MouseEvent) => void

FUNCTION FileRow({ item, isSelected, onClick, onContextMenu }):
    // Get appropriate icon based on item type
    FUNCTION getIcon():
        IF item.type === "folder":
            RETURN "folder"
        ELSE IF item.extension === ".pdf":
            RETURN "pdf"
        ELSE IF item.extension IN [".jpg", ".png", ".gif"]:
            RETURN "image"
        ELSE:
            RETURN "file"
        END IF
    END FUNCTION
    
    // Format file size
    FUNCTION formatSize(bytes):
        IF bytes < 1024:
            RETURN bytes + " B"
        ELSE IF bytes < 1024 * 1024:
            RETURN (bytes / 1024).toFixed(1) + " KB"
        ELSE:
            RETURN (bytes / (1024 * 1024)).toFixed(1) + " MB"
        END IF
    END FUNCTION
    
    // Format date
    FUNCTION formatDate(timestamp):
        CONST date = NEW Date(timestamp)
        CONST month = date.toLocaleString('default', { month: 'short' })
        CONST day = date.getDate()
        CONST year = date.getFullYear()
        RETURN `${month} ${day}, ${year}`
    END FUNCTION
    
    RETURN:
        <tr 
            className={`file-row ${isSelected ? 'selected' : ''}`}
            onClick={onClick}
            onContextMenu={onContextMenu}
        >
            <td className="name-cell">
                <Icon name={getIcon()} />
                <span>{item.name}</span>
            </td>
            
            <td className="updated-cell">
                {formatDate(item.modified_at)} by {item.modified_by.name}
            </td>
            
            <td className="size-cell">
                {item.type === "folder" 
                    ? item.item_collection.total_count + " Files"
                    : formatSize(item.size)
                }
            </td>
        </tr>
END FUNCTION

EXPORT FileRow
```

---

#### 2.8 Right Sidebar Component

**File:** `frontend/src/components/Sidebar/RightSidebar.tsx`

```typescript
// Right details sidebar
IMPORT Toggle FROM '../common/Toggle'

INTERFACE RightSidebarProps:
    selectedItem: Folder | File | null
    visible: boolean

FUNCTION RightSidebar({ selectedItem, visible }):
    STATE active_tab = "details"  // or "sharing"
    STATE sync_enabled = false
    
    IF NOT visible OR selectedItem === null:
        RETURN null
    END IF
    
    // Format timestamp
    FUNCTION formatTimestamp(timestamp):
        CONST date = NEW Date(timestamp)
        CONST month = date.toLocaleString('default', { month: 'short' })
        CONST day = date.getDate()
        CONST year = date.getFullYear()
        CONST time = date.toLocaleTimeString('en-US', { 
            hour: 'numeric', 
            minute: '2-digit',
            hour12: true 
        })
        RETURN `${month} ${day}, ${year}, ${time}`
    END FUNCTION
    
    RETURN:
        <aside className="right-sidebar">
            <div className="sidebar-tabs">
                <button 
                    className={active_tab === "sharing" ? "active" : ""}
                    onClick={() => SET active_tab = "sharing"}
                >
                    Sharing
                </button>
                <button 
                    className={active_tab === "details" ? "active" : ""}
                    onClick={() => SET active_tab = "details"}
                >
                    Details
                </button>
            </div>
            
            <div className="sidebar-content">
                IF active_tab === "details":
                    <div className="details-panel">
                        <div className="sync-section">
                            <label>Sync to Desktop</label>
                            <Toggle 
                                checked={sync_enabled}
                                onChange={(value) => SET sync_enabled = value}
                            />
                        </div>
                        
                        <div className="classification-section">
                            <label>Classification</label>
                            <div className="value">Not classified</div>
                            <a href="#" className="add-link">Add</a>
                        </div>
                        
                        <div className="properties-section">
                            <h3>Folder Properties</h3>
                            
                            <div className="property">
                                <label>Description</label>
                                <input 
                                    type="text" 
                                    placeholder="Enter a description"
                                />
                            </div>
                            
                            <div className="property">
                                <label>Owner</label>
                                <div className="value">{selectedItem.owned_by.name}</div>
                            </div>
                            
                            <div className="property">
                                <label>Created</label>
                                <div className="value">
                                    {formatTimestamp(selectedItem.created_at)}
                                </div>
                            </div>
                            
                            <div className="property">
                                <label>Modified</label>
                                <div className="value">
                                    {formatTimestamp(selectedItem.modified_at)}
                                </div>
                            </div>
                            
                            <div className="property">
                                <label>Size</label>
                                <div className="value">
                                    {selectedItem.type === "folder" 
                                        ? selectedItem.size + " MB"
                                        : formatSize(selectedItem.size)
                                    }
                                </div>
                            </div>
                        </div>
                        
                        <div className="metadata-section">
                            <label>Metadata</label>
                            <div className="empty-state">
                                No metadata applied. Click add to apply metadata to this folder.
                            </div>
                            <a href="#" className="add-link">Add</a>
                        </div>
                        
                        <div className="workflows-section">
                            <label>Workflows</label>
                            <div className="empty-state">
                                No workflows found on this folder.
                            </div>
                        </div>
                    </div>
                ELSE:
                    <div className="sharing-panel">
                        <!-- Sharing UI to be implemented -->
                        <p>Sharing options coming soon...</p>
                    </div>
                END IF
            </div>
        </aside>
END FUNCTION

EXPORT RightSidebar
```

---

#### 2.9 Context Menu Component

**File:** `frontend/src/components/ContextMenu/ContextMenu.tsx`

```typescript
// Right-click context menu
IMPORT { useEffect } FROM 'react'

INTERFACE ContextMenuProps:
    x: number
    y: number
    item: Folder | File
    onClose: () => void

FUNCTION ContextMenu({ x, y, item, onClose }):
    // Menu items based on item type
    FUNCTION getMenuItems():
        CONST common_items = [
            { id: "open", label: "Open", icon: "open" },
            { id: "download", label: "Download", icon: "download", shortcut: "⌘D" },
            { id: "rename", label: "Rename", icon: "edit" },
            { id: "copy", label: "Copy", icon: "copy", shortcut: "⌘C" },
            { id: "move", label: "Move", icon: "move" },
            { id: "delete", label: "Delete", icon: "trash", shortcut: "⌘⌫" }
        ]
        
        IF item.type === "folder":
            RETURN [
                { id: "open", label: "Open", icon: "open" },
                { id: "share", label: "Share", icon: "share" },
                ...common_items.slice(2)
            ]
        ELSE:
            RETURN [
                { id: "preview", label: "Preview", icon: "eye" },
                { id: "share", label: "Share", icon: "share" },
                ...common_items.slice(1)
            ]
        END IF
    END FUNCTION
    
    // Close on click outside
    EFFECT:
        FUNCTION handleClickOutside(event):
            IF NOT event.target.closest('.context-menu'):
                onClose()
            END IF
        END FUNCTION
        
        document.addEventListener('click', handleClickOutside)
        
        CLEANUP:
            document.removeEventListener('click', handleClickOutside)
    END EFFECT
    
    // Close on Escape key
    EFFECT:
        FUNCTION handleKeyDown(event):
            IF event.key === "Escape":
                onClose()
            END IF
        END FUNCTION
        
        document.addEventListener('keydown', handleKeyDown)
        
        CLEANUP:
            document.removeEventListener('keydown', handleKeyDown)
    END EFFECT
    
    // Handle menu item click
    FUNCTION handleItemClick(action_id):
        SWITCH action_id:
            CASE "open":
                // Open item logic
                BREAK
            CASE "download":
                // Download logic
                BREAK
            CASE "rename":
                // Rename logic
                BREAK
            CASE "share":
                // Share logic
                BREAK
            CASE "delete":
                // Delete logic
                BREAK
            DEFAULT:
                console.log(`Action ${action_id} not implemented`)
        END SWITCH
        
        onClose()
    END FUNCTION
    
    CONST menu_items = getMenuItems()
    
    RETURN:
        <div 
            className="context-menu"
            style={{ left: x, top: y }}
        >
            <ul>
                FOR EACH item IN menu_items:
                    <li 
                        key={item.id}
                        onClick={() => handleItemClick(item.id)}
                    >
                        <Icon name={item.icon} />
                        <span>{item.label}</span>
                        {item.shortcut AND (
                            <span className="shortcut">{item.shortcut}</span>
                        )}
                    </li>
                END FOR
            </ul>
        </div>
END FUNCTION

EXPORT ContextMenu
```

---

#### 2.10 Custom Hook: useBox

**File:** `frontend/src/hooks/useBox.ts`

```typescript
// Custom hook for Box API integration
IMPORT { useState, useEffect } FROM 'react'
IMPORT api FROM '../services/api'

FUNCTION useBox(folder_id: string):
    STATE folders = []
    STATE files = []
    STATE loading = true
    STATE error = null
    
    // Fetch folder contents
    EFFECT [folder_id]:
        ASYNC FUNCTION fetchFolderContents():
            TRY:
                SET loading = true
                SET error = null
                
                CONST response = AWAIT api.get(`/api/folders/${folder_id}/items`)
                
                // Separate folders and files
                CONST items = response.data.entries
                SET folders = items.FILTER(item => item.type === "folder")
                SET files = items.FILTER(item => item.type === "file")
                
                SET loading = false
            CATCH err:
                SET error = err
                SET loading = false
            END TRY
        END FUNCTION
        
        fetchFolderContents()
    END EFFECT
    
    RETURN { folders, files, loading, error }
END FUNCTION

EXPORT useBox
```

---

#### 2.11 Custom Hook: useFileNavigation

**File:** `frontend/src/hooks/useFileNavigation.ts`

```typescript
// Custom hook for file navigation and breadcrumbs
IMPORT { useState } FROM 'react'
IMPORT api FROM '../services/api'

FUNCTION useFileNavigation():
    STATE breadcrumbs = [{ id: "0", name: "All Files" }]
    STATE navigation_stack = ["0"]
    
    // Navigate to a specific folder
    ASYNC FUNCTION navigateToFolder(folder_id: string):
        TRY:
            // Fetch folder details to get name and path
            CONST response = AWAIT api.get(`/api/folders/${folder_id}`)
            CONST folder = response.data
            
            // Build breadcrumbs from path_collection
            CONST new_breadcrumbs = [{ id: "0", name: "All Files" }]
            
            FOR EACH ancestor IN folder.path_collection.entries:
                IF ancestor.id !== "0":
                    new_breadcrumbs.PUSH({
                        id: ancestor.id,
                        name: ancestor.name
                    })
                END IF
            END FOR
            
            // Add current folder
            new_breadcrumbs.PUSH({
                id: folder.id,
                name: folder.name
            })
            
            SET breadcrumbs = new_breadcrumbs
            navigation_stack.PUSH(folder_id)
        CATCH err:
            console.error("Navigation failed:", err)
        END TRY
    END FUNCTION
    
    // Navigate up one level
    FUNCTION navigateUp():
        IF navigation_stack.length > 1:
            navigation_stack.POP()
            CONST parent_id = navigation_stack[navigation_stack.length - 1]
            navigateToFolder(parent_id)
        END IF
    END FUNCTION
    
    RETURN { navigateToFolder, navigateUp, breadcrumbs }
END FUNCTION

EXPORT useFileNavigation
```

---

#### 2.12 API Service

**File:** `frontend/src/services/api.ts`

```typescript
// API service for backend communication
IMPORT axios FROM 'axios'

// Create axios instance with base configuration
CONST api = axios.create({
    baseURL: import.meta.env.VITE_API_URL || 'http://localhost:3000',
    timeout: 10000,
    headers: {
        'Content-Type': 'application/json'
    }
})

// Request interceptor
api.interceptors.request.use(
    (config) => {
        // Add any auth tokens or custom headers here
        RETURN config
    },
    (error) => {
        RETURN Promise.reject(error)
    }
)

// Response interceptor
api.interceptors.response.use(
    (response) => {
        RETURN response
    },
    (error) => {
        // Handle errors globally
        IF error.response:
            console.error('API Error:', error.response.data.message)
        ELSE IF error.request:
            console.error('Network Error:', error.message)
        ELSE:
            console.error('Error:', error.message)
        END IF
        
        RETURN Promise.reject(error)
    }
)

EXPORT api
```

---

### Backend Pseudocode

#### 2.13 Server Entry Point

**File:** `backend/src/server.ts`

```typescript
// Main server application
IMPORT express FROM 'express'
IMPORT cors FROM 'cors'
IMPORT dotenv FROM 'dotenv'
IMPORT filesRouter FROM './routes/files.routes'

// Load environment variables
dotenv.config()

// Create Express app
CONST app = express()
CONST PORT = process.env.PORT || 3000

// Middleware
app.use(cors())
app.use(express.json())

// Routes
app.use('/api', filesRouter)

// Health check endpoint
app.get('/health', (req, res) => {
    res.json({ status: 'ok', timestamp: new Date().toISOString() })
})

// Error handling middleware
app.use((err, req, res, next) => {
    console.error(err.stack)
    res.status(500).json({ 
        error: 'Internal Server Error',
        message: err.message 
    })
})

// Start server
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`)
})
```

---

#### 2.14 Files Router

**File:** `backend/src/routes/files.routes.ts`

```typescript
// API routes for file/folder operations
IMPORT express FROM 'express'
IMPORT BoxService FROM '../services/box.service'

CONST router = express.Router()
CONST boxService = NEW BoxService()

// Get folder items
router.get('/folders/:folderId/items', ASYNC (req, res) => {
    TRY:
        CONST folder_id = req.params.folderId
        CONST limit = req.query.limit || 100
        CONST offset = req.query.offset || 0
        
        // Fetch folder items from Box
        CONST items = AWAIT boxService.getFolderItems(folder_id, limit, offset)
        
        res.json(items)
    CATCH error:
        res.status(500).json({ 
            error: 'Failed to fetch folder items',
            message: error.message 
        })
    END TRY
})

// Get folder details
router.get('/folders/:folderId', ASYNC (req, res) => {
    TRY:
        CONST folder_id = req.params.folderId
        
        // Fetch folder details from Box
        CONST folder = AWAIT boxService.getFolderById(folder_id)
        
        res.json(folder)
    CATCH error:
        res.status(500).json({ 
            error: 'Failed to fetch folder',
            message: error.message 
        })
    END TRY
})

// Search files/folders
router.get('/search', ASYNC (req, res) => {
    TRY:
        CONST query = req.query.q
        CONST folder_id = req.query.ancestor_folder_ids || '0'
        
        IF NOT query:
            RETURN res.status(400).json({ error: 'Query parameter required' })
        END IF
        
        // Search in Box
        CONST results = AWAIT boxService.search(query, folder_id)
        
        res.json(results)
    CATCH error:
        res.status(500).json({ 
            error: 'Search failed',
            message: error.message 
        })
    END TRY
})

// Download file
router.get('/files/:fileId/download', ASYNC (req, res) => {
    TRY:
        CONST file_id = req.params.fileId
        
        // Get download URL or stream from Box
        CONST download_stream = AWAIT boxService.downloadFile(file_id)
        
        // Pipe to response
        download_stream.pipe(res)
    CATCH error:
        res.status(500).json({ 
            error: 'Download failed',
            message: error.message 
        })
    END TRY
})

EXPORT router
```

---

#### 2.15 Box Service

**File:** `backend/src/services/box.service.ts`

```typescript
// Box SDK integration service
IMPORT { BoxClient } from '@box/box-typescript-sdk-gen'
IMPORT { BoxCCGAuth } from '@box/box-typescript-sdk-gen/auth'
IMPORT boxConfig FROM '../config/box.config'

CLASS BoxService:
    PRIVATE client: BoxClient
    PRIVATE auth: BoxCCGAuth
    
    CONSTRUCTOR():
        // Initialize CCG authentication
        this.auth = NEW BoxCCGAuth({
            clientId: boxConfig.clientId,
            clientSecret: boxConfig.clientSecret,
            enterpriseId: boxConfig.enterpriseId,
            userId: boxConfig.userId
        })
        
        // Create Box client
        this.client = NEW BoxClient({ auth: this.auth })
    END CONSTRUCTOR
    
    // Get folder items
    ASYNC METHOD getFolderItems(folder_id: string, limit: number, offset: number):
        TRY:
            CONST response = AWAIT this.client.folders.getFolderItems(folder_id, {
                queryParams: {
                    limit: limit,
                    offset: offset,
                    fields: [
                        'id', 'name', 'type', 'size', 
                        'modified_at', 'modified_by', 
                        'owned_by', 'path_collection',
                        'item_collection'
                    ]
                }
            })
            
            RETURN response
        CATCH error:
            console.error('Error fetching folder items:', error)
            THROW error
        END TRY
    END METHOD
    
    // Get folder by ID
    ASYNC METHOD getFolderById(folder_id: string):
        TRY:
            CONST response = AWAIT this.client.folders.getFolderById(folder_id, {
                queryParams: {
                    fields: [
                        'id', 'name', 'type', 'size',
                        'created_at', 'modified_at',
                        'owned_by', 'path_collection',
                        'description', 'item_collection'
                    ]
                }
            })
            
            RETURN response
        CATCH error:
            console.error('Error fetching folder:', error)
            THROW error
        END TRY
    END METHOD
    
    // Search for items
    ASYNC METHOD search(query: string, ancestor_folder_id: string):
        TRY:
            CONST response = AWAIT this.client.search.searchForContent({
                query: query,
                ancestorFolderIds: [ancestor_folder_id],
                fields: ['id', 'name', 'type', 'size', 'modified_at']
            })
            
            RETURN response
        CATCH error:
            console.error('Search error:', error)
            THROW error
        END TRY
    END METHOD
    
    // Download file
    ASYNC METHOD downloadFile(file_id: string):
        TRY:
            CONST download_url = AWAIT this.client.files.getFileDownloadUrl(file_id)
            
            // Return stream or URL
            RETURN download_url
        CATCH error:
            console.error('Download error:', error)
            THROW error
        END TRY
    END METHOD
    
    // Cache management (simple in-memory cache for demo)
    PRIVATE cache: Map<string, { data: any, timestamp: number }> = NEW Map()
    PRIVATE CACHE_TTL = 5 * 60 * 1000  // 5 minutes
    
    PRIVATE METHOD getCachedData(key: string):
        CONST cached = this.cache.get(key)
        
        IF cached AND (Date.now() - cached.timestamp < this.CACHE_TTL):
            RETURN cached.data
        END IF
        
        RETURN null
    END METHOD
    
    PRIVATE METHOD setCachedData(key: string, data: any):
        this.cache.set(key, {
            data: data,
            timestamp: Date.now()
        })
    END METHOD
    
END CLASS

EXPORT BoxService
```

---

#### 2.16 Box Configuration

**File:** `backend/src/config/box.config.ts`

```typescript
// Box SDK configuration
IMPORT dotenv FROM 'dotenv'

dotenv.config()

INTERFACE BoxConfig:
    clientId: string
    clientSecret: string
    enterpriseId: string
    userId: string

CONST boxConfig: BoxConfig = {
    clientId: process.env.BOX_CLIENT_ID || '',
    clientSecret: process.env.BOX_CLIENT_SECRET || '',
    enterpriseId: process.env.BOX_ENTERPRISE_ID || '',
    userId: process.env.BOX_USER_ID || ''
}

// Validate configuration
FUNCTION validateConfig():
    CONST required_fields = ['clientId', 'clientSecret', 'enterpriseId', 'userId']
    
    FOR EACH field IN required_fields:
        IF NOT boxConfig[field]:
            THROW NEW Error(`Missing required Box configuration: ${field}`)
        END IF
    END FOR
END FUNCTION

validateConfig()

EXPORT boxConfig
```

---

### Shared Types

#### 2.17 Box Type Definitions

**File:** `frontend/src/types/box.types.ts` (mirrored in backend)

```typescript
// Shared type definitions for Box entities

INTERFACE User:
    id: string
    type: "user"
    name: string
    login: string

INTERFACE PathItem:
    id: string
    name: string
    type: "folder"

INTERFACE PathCollection:
    total_count: number
    entries: Array<PathItem>

INTERFACE ItemCollection:
    total_count: number
    entries: Array<any>  // Can be files or folders

INTERFACE BaseItem:
    id: string
    type: "file" | "folder"
    name: string
    created_at: string
    modified_at: string
    owned_by: User
    modified_by: User
    path_collection: PathCollection

INTERFACE Folder EXTENDS BaseItem:
    type: "folder"
    item_collection: ItemCollection
    size: number  // Total size of folder contents
    description?: string

INTERFACE File EXTENDS BaseItem:
    type: "file"
    size: number
    extension: string
    file_version: {
        id: string
        sha1: string
    }

TYPE BoxItem = Folder | File

EXPORT { User, PathItem, PathCollection, Folder, File, BoxItem }
```

---

### Application Startup Flow

#### 2.18 Startup Scripts

**File:** `scripts/setup.sh`

```bash
#!/bin/bash
# Initial setup script

ECHO "Setting up Box Demo Web Application..."

# Check Node.js version
REQUIRED_NODE_VERSION=18
CURRENT_NODE_VERSION=$(node -v | cut -d'v' -f2 | cut -d'.' -f1)

IF CURRENT_NODE_VERSION < REQUIRED_NODE_VERSION:
    ECHO "Error: Node.js version $REQUIRED_NODE_VERSION or higher required"
    EXIT 1
END IF

# Check for .env file
IF NOT EXISTS .env:
    ECHO "Creating .env file from template..."
    COPY .env.example .env
    ECHO "Please update .env with your Box credentials"
    EXIT 1
END IF

# Install backend dependencies
ECHO "Installing backend dependencies..."
CD backend
npm install
CD ..

# Install frontend dependencies
ECHO "Installing frontend dependencies..."
CD frontend
npm install
CD ..

ECHO "Setup complete! Run './scripts/start.sh' to start the application"
```

**File:** `scripts/start.sh`

```bash
#!/bin/bash
# Application startup script

ECHO "Starting Box Demo Web Application..."

# Start backend server in background
ECHO "Starting backend server..."
CD backend
npm run dev &
BACKEND_PID=$!
CD ..

# Wait for backend to be ready
SLEEP 2

# Start frontend dev server
ECHO "Starting frontend..."
CD frontend
npm run dev &
FRONTEND_PID=$!
CD ..

ECHO "Application started!"
ECHO "Frontend: http://localhost:5174"
ECHO "Backend: http://localhost:3001"
ECHO ""
ECHO "Press Ctrl+C to stop both servers"

# Trap Ctrl+C to kill both processes
TRAP "kill $BACKEND_PID $FRONTEND_PID; exit" INT

# Wait for processes
WAIT
```

---

### Reflection

#### Pseudocode Design Decisions

**1. Component Granularity**
The pseudocode is organized into discrete, focused components rather than monolithic structures. This aligns with React best practices and makes the codebase easier to understand and maintain. Each component has a single, clear responsibility.

**2. State Management Approach**
We use React hooks (useState, useEffect) and custom hooks (useBox, useFileNavigation) for state management rather than introducing Redux or other heavyweight solutions. This keeps the application lightweight and reduces the learning curve for developers unfamiliar with complex state management libraries.

The decision to use custom hooks centralizes Box API logic and navigation state, making it reusable across components while keeping the component code clean.

**3. API Layer Separation**
The frontend communicates with the backend through a dedicated API service (`api.ts`) using Axios. This abstraction allows us to:
- Centralize error handling
- Add authentication headers consistently
- Mock API responses during development
- Switch between different backend implementations easily

**4. Box SDK Integration**
The backend uses a service class (`BoxService`) to encapsulate all Box API interactions. This design:
- Centralizes authentication logic
- Provides a consistent interface for Box operations
- Enables simple in-memory caching for improved performance
- Makes it easy to add error handling and retry logic

**5. Type Safety**
Shared TypeScript interfaces ensure type consistency between frontend and backend. This reduces runtime errors and improves developer experience with autocomplete and type checking.

#### Potential Challenges

**Challenge 1: Real-time Updates**
- *Issue:* The current design doesn't handle real-time updates when files/folders change
- *Solution:* Implement periodic polling or WebSocket connection if needed. For a demo app, manual refresh is acceptable.

**Challenge 2: Large Folder Performance**
- *Issue:* Folders with thousands of items could cause performance issues
- *Solution:* Implement pagination on both frontend and backend. The pseudocode includes limit/offset parameters as a foundation.

**Challenge 3: Caching Strategy**
- *Issue:* Simple in-memory cache is not shared across server restarts or multiple instances
- *Solution:* For a demo app, this is acceptable. Document cache limitations in README. Consider localStorage on frontend for user preferences.

**Challenge 4: Error Handling**
- *Issue:* Box API errors need to be user-friendly
- *Solution:* Add error boundary components in React and map Box API errors to friendly messages in the backend service.

#### Alternative Approaches Considered

**1. Server-Side Rendering (SSR)**
- *Considered:* Using Next.js for SSR
- *Chosen:* Client-side React + separate backend
- *Reason:* Simpler deployment model, faster development, no need for SSR benefits in a demo app

**2. GraphQL vs REST**
- *Considered:* GraphQL for flexible data fetching
- *Chosen:* REST API
- *Reason:* Simpler to implement, Box SDK is REST-based, no complex query requirements

**3. Monorepo vs Separate Repos**
- *Considered:* Separate repos for frontend/backend
- *Chosen:* Monorepo structure
- *Reason:* Easier for developers to work on both simultaneously, shared types, simpler deployment for demo purposes

**4. WebSocket vs Polling vs No Real-time**
- *Considered:* WebSocket for real-time updates
- *Chosen:* No real-time (manual refresh)
- *Reason:* Reduces complexity, adequate for demo scenarios, can be added later if needed

#### How Pseudocode Maps to Specification

Each functional requirement from Step 1 (Specification) has corresponding pseudocode:

- **FR-1 (Left Sidebar):** `LeftSidebar.tsx` (Section 2.4)
- **FR-2 (File List):** `FileListView.tsx`, `FileRow.tsx`, `TopBar.tsx` (Sections 2.6, 2.7, 2.5)
- **FR-3 (Right Sidebar):** `RightSidebar.tsx` (Section 2.8)
- **FR-4 (Context Menu):** `ContextMenu.tsx` (Section 2.9)

Non-functional requirements are addressed through:
- **NFR-1 (Lightweight):** Minimal dependencies, no databases, simple caching
- **NFR-2 (Easy Deployment):** Startup scripts (Section 2.18)
- **NFR-3 (Performance):** In-memory caching, efficient API calls
- **NFR-4 (Box Integration):** `BoxService` with CCG auth (Section 2.15)

#### Next Steps

With pseudocode in place, we can proceed to:
1. **Architecture (Step 3):** Make concrete technology selections, define deployment model, create system diagrams
2. **Refinement (Step 4):** Review pseudocode for optimizations, add error handling details, refine caching strategy
3. **Completion (Step 5):** Implement actual code based on pseudocode, test, and deploy

The pseudocode serves as a comprehensive blueprint that developers can follow to implement the application with minimal ambiguity.
