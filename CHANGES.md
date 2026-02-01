# Code Changes

This document details all modifications made to Mainsail to add the KlipperSearch feature.

## Source Repository

All source code is maintained in the Mainsail fork:
**https://github.com/mikehatch/mainsail** (develop branch)

---

## Files Created

### 1. Search Vuex Store Module

**Location:** `src/store/search/`

#### `src/store/search/types.ts`
Defines TypeScript interfaces for the search feature:

```typescript
export interface SearchResult {
    filename: string      // Config filename (e.g., "printer.cfg")
    filepath: string      // Full path in config directory
    line: number         // Line number where match was found
    content: string      // The matching line content
    context: string[]    // Surrounding lines for context
}

export interface CachedFile {
    content: string      // File contents
    modified: number     // Timestamp for cache invalidation
}

export interface SearchState {
    showDialog: boolean              // Dialog visibility
    query: string                    // Current search query
    results: SearchResult[]          // Search results
    isSearching: boolean            // Loading state
    fileCache: { [key: string]: CachedFile }  // File cache
    targetLine: number | null       // Line to navigate to
}
```

[View full file](https://github.com/mikehatch/mainsail/blob/develop/src/store/search/types.ts)

---

#### `src/store/search/index.ts`
Main store module definition with initial state:

```typescript
export const getDefaultState = (): SearchState => {
    return {
        showDialog: false,
        query: '',
        results: [],
        isSearching: false,
        fileCache: {},
        targetLine: null,
    }
}

export const search: Module<SearchState, any> = {
    namespaced: true,
    state,
    getters,
    actions,
    mutations,
}
```

[View full file](https://github.com/mikehatch/mainsail/blob/develop/src/store/search/index.ts)

---

#### `src/store/search/actions.ts`
Core search logic (158 lines):

**Key Actions:**
- `search(query)` - Main search orchestration with debouncing
- `performSearch(query)` - Searches all config files using regex
- `getConfigFiles()` - Retrieves `.cfg` and `.conf` files from filetree
- `getFileContent(file)` - Fetches and caches file contents from Moonraker
- `openResult(result)` - Opens file in editor and navigates to line

**File Caching Logic:**
```typescript
async getFileContent({ commit, state, rootGetters }, file: FileStateFile) {
    const filePath = file.path || file.filename
    const cached = state.fileCache[filePath]

    // Check cache validity using modification timestamp
    if (cached && file.modified && cached.modified >= file.modified.getTime()) {
        return cached.content
    }

    // Fetch from Moonraker API
    const baseUrl = rootGetters['socket/getUrl']
    const url = `${baseUrl}/server/files/config/${encodeURIComponent(filePath)}`
    const response = await axios.get(url, { responseType: 'text' })

    // Update cache
    commit('setCachedFile', {
        path: filePath,
        file: {
            content: response.data,
            modified: file.modified ? file.modified.getTime() : Date.now(),
        },
    })

    return response.data
}
```

[View full file](https://github.com/mikehatch/mainsail/blob/develop/src/store/search/actions.ts)

---

#### `src/store/search/mutations.ts`
State mutations:

```typescript
export const mutations: MutationTree<SearchState> = {
    setShowDialog(state, payload) {
        state.showDialog = payload
    },
    setQuery(state, payload) {
        state.query = payload
    },
    setResults(state, payload) {
        state.results = payload
    },
    setIsSearching(state, payload) {
        state.isSearching = payload
    },
    setCachedFile(state, { path, file }) {
        state.fileCache[path] = file
    },
    setTargetLine(state, payload) {
        state.targetLine = payload
    },
    reset(state) {
        Object.assign(state, getDefaultState())
    },
}
```

[View full file](https://github.com/mikehatch/mainsail/blob/develop/src/store/search/mutations.ts)

---

#### `src/store/search/getters.ts`
Computed properties:

```typescript
export const getters: GetterTree<SearchState, RootState> = {
    hasResults: (state) => state.results.length > 0,
    resultCount: (state) => state.results.length,
}
```

[View full file](https://github.com/mikehatch/mainsail/blob/develop/src/store/search/getters.ts)

---

### 2. Search Dialog Component

#### `src/components/dialogs/ConfigSearchDialog.vue`
Vue component for the search UI (265 lines):

**Features:**
- Debounced search input (300ms)
- Real-time results as you type
- Keyboard navigation (Enter, Up, Down arrows)
- Highlighted search matches
- Click to open file at specific line

**Template Structure:**
```vue
<v-dialog v-model="showDialog" max-width="700" :fullscreen="isMobile">
    <panel :title="$t('ConfigSearch.Title')">
        <!-- Search Input -->
        <v-text-field
            v-model="searchQuery"
            :label="$t('ConfigSearch.SearchPlaceholder')"
            :loading="isSearching"
            autofocus
            @input="debouncedSearch"
            @keydown.enter="selectFirstResult"
        />

        <!-- Results List -->
        <v-list v-if="hasResults">
            <v-list-item
                v-for="result in results"
                @click="openResult(result)"
            >
                <v-list-item-title>
                    {{ result.filename }}:{{ result.line }}
                </v-list-item-title>
                <v-list-item-subtitle>
                    <code v-html="highlightMatch(result.content)" />
                </v-list-item-subtitle>
            </v-list-item>
        </v-list>
    </panel>
</v-dialog>
```

**Search Highlighting:**
```typescript
highlightMatch(text: string): string {
    if (!this.searchQuery) return this.escapeHtml(text)

    const escapedQuery = this.searchQuery.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
    const regex = new RegExp(`(${escapedQuery})`, 'gi')
    const escaped = this.escapeHtml(text)

    return escaped.replace(regex, '<mark>$1</mark>')
}
```

[View full file](https://github.com/mikehatch/mainsail/blob/develop/src/components/dialogs/ConfigSearchDialog.vue)

---

## Files Modified

### 1. Top Navigation Bar

#### `src/components/TheTopbar.vue`

**Changes:**
1. Added search icon button
2. Added ConfigSearchDialog component
3. Added keyboard shortcut handler (Ctrl+Shift+F)

**Search Button:**
```vue
<v-btn
    v-if="klippyIsConnected"
    tile
    icon
    class="config-search-button"
    :title="$t('ConfigSearch.Title')"
    @click="openConfigSearch">
    <v-icon>{{ mdiMagnify }}</v-icon>
</v-btn>
```

**Keyboard Shortcut:**
```typescript
mounted() {
    // Add keyboard shortcut for config search (Ctrl+Shift+F)
    window.addEventListener('keydown', this.handleKeyboardShortcut)
}

beforeDestroy() {
    window.removeEventListener('keydown', this.handleKeyboardShortcut)
}

handleKeyboardShortcut(event: KeyboardEvent) {
    if ((event.ctrlKey || event.metaKey) && event.shiftKey && event.key === 'F') {
        event.preventDefault()
        this.openConfigSearch()
    }
}

openConfigSearch() {
    this.$store.dispatch('search/openDialog')
}
```

**Imports Added:**
```typescript
import { mdiMagnify } from '@mdi/js'
import ConfigSearchDialog from '@/components/dialogs/ConfigSearchDialog.vue'
```

[View full file](https://github.com/mikehatch/mainsail/blob/develop/src/components/TheTopbar.vue)

---

### 2. Code Editor

#### `src/components/TheEditor.vue`

**Changes:**
1. Added `searchTargetLine` getter to read from search store
2. Added watcher to navigate to line when file opens from search

**Line Navigation:**
```typescript
get searchTargetLine(): number | null {
    return this.$store.state.search?.targetLine ?? null
}

@Watch('show')
onShowChange(newVal: boolean) {
    if (newVal && this.searchTargetLine) {
        // Wait for the editor to be ready and content to load
        this.$nextTick(() => {
            setTimeout(() => {
                if (this.editor && this.searchTargetLine) {
                    this.editor.gotoLine(this.searchTargetLine)
                    this.$store.dispatch('search/clearTargetLine')
                }
            }, 100)
        })
    }
}
```

This watches for the editor dialog to open, and if there's a target line set by the search feature, it navigates to that line using CodeMirror's `gotoLine` method.

[View full file](https://github.com/mikehatch/mainsail/blob/develop/src/components/TheEditor.vue)

---

### 3. Store Registration

#### `src/store/index.ts`

**Added:**
```typescript
import { search } from './search'

// In modules registration:
modules: {
    // ... existing modules
    search,
}
```

[View full file](https://github.com/mikehatch/mainsail/blob/develop/src/store/index.ts)

---

### 4. Root State Types

#### `src/store/types.ts`

**Added imports:**
```typescript
import { FileState } from '@/store/files/types'
import { SearchState } from '@/store/search/types'
```

**Added to RootState interface:**
```typescript
export interface RootState {
    // ... existing properties
    files?: FileState
    search?: SearchState
}
```

[View full file](https://github.com/mikehatch/mainsail/blob/develop/src/store/types.ts)

---

### 5. Translations

#### `src/locales/en.json`

**Added:**
```json
{
    "ConfigSearch": {
        "Title": "Search Config Files",
        "SearchPlaceholder": "Search for text in config files...",
        "TypeToSearch": "Type at least 2 characters to search",
        "NoResults": "No results found",
        "ResultCount": "{count} result(s) found",
        "PressEnter": "Press Enter to open first result"
    }
}
```

[View full file](https://github.com/mikehatch/mainsail/blob/develop/src/locales/en.json)

---

## Architecture Summary

```
User Action (Ctrl+Shift+F or Click Icon)
    ↓
TheTopbar.vue (openConfigSearch)
    ↓
search/openDialog action
    ↓
ConfigSearchDialog opens
    ↓
User types query (debounced 300ms)
    ↓
search/search action
    ↓
search/getConfigFiles
    → Gets file list from rootState.files.filetree
    → Filters .cfg and .conf files
    ↓
search/performSearch
    → For each file:
        → search/getFileContent
            → Check cache (with timestamp validation)
            → If not cached: fetch from Moonraker API
            → Cache result
        → Search content with regex
        → Collect matches with line numbers
    → Return results (max 100)
    ↓
Results displayed in dialog
    ↓
User clicks result
    ↓
search/openResult
    → Set targetLine in store
    → Dispatch editor/openFile
    ↓
TheEditor.vue watches for show + targetLine
    → Calls editor.gotoLine(targetLine)
    → Clears targetLine
```

---

## API Endpoints Used

### Moonraker File API

**List files:**
```
GET /server/files/list?root=config
```
Retrieved from existing `rootState.files.filetree` (already cached by Mainsail)

**Get file content:**
```
GET /server/files/config/{filename}
```
Fetched on-demand and cached with modification timestamp

---

## Performance Optimizations

1. **File Caching**: Files are cached with modification timestamps to avoid re-fetching unchanged files
2. **Debounced Search**: 300ms debounce on search input to reduce unnecessary searches
3. **Result Limiting**: Maximum 100 results to prevent UI overload
4. **Regex Escaping**: User input is escaped to prevent regex errors
5. **Case Insensitive**: Search uses `gi` flags for case-insensitive matching

---

## Testing

The search feature was developed and tested using:
- Local development server (`npm run serve`)
- [virtual-klipper-printer](https://github.com/mainsail-crew/virtual-klipper-printer) Docker environment

---

## Future Enhancements

Potential improvements (not currently implemented):

- [ ] Regular expression search support (advanced mode)
- [ ] Search history / recent searches
- [ ] File type filtering
- [ ] Search within specific directories
- [ ] Replace functionality
- [ ] Server-side indexing for faster searches
- [ ] Search result export

---

## License

All modifications inherit the [GNU General Public License v3.0](LICENSE) from Mainsail.

---

## Links

- **Source Code**: https://github.com/mikehatch/mainsail/tree/develop
- **Base Project**: https://github.com/mainsail-crew/mainsail
- **Installation Guide**: [INSTALL.md](INSTALL.md)
- **Development Guide**: [DEVELOPMENT.md](DEVELOPMENT.md)
