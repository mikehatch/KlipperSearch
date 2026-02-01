# Research Findings: Klipper Config Search

## Moonraker File API Capabilities

### Available Endpoints
| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/server/files/list?root=config` | GET | List all config files recursively |
| `/server/files/config/{filename}` | GET | Download file contents |
| `/server/files/get_directory` | GET | List directory contents |

### Key Findings
- **No built-in search**: Moonraker has no grep/search functionality
- **File access**: Can list and read all files in the `config` root
- **Database API**: Key-value store only (LMDB/SQLite), not suitable for full-text search indexing

### Config Root Structure
Klipper config files typically include:
- `printer.cfg` - Main configuration
- `*.cfg` - Additional config includes (macros, hardware, etc.)
- Nested directories for organization

---

## Mainsail Architecture

### Technology Stack
- **Framework**: Vue.js 3 with TypeScript
- **UI Library**: Vuetify (Material Design)
- **Build Tool**: Vite
- **State Management**: Vuex/Pinia (standard Vue patterns)

### File Editor
- Already has syntax highlighting for config files
- Opens files via Moonraker file API
- Could be extended to accept search result navigation

---

## Approach Comparison

### Option 1: Client-Side Search (Simple)

**Architecture:**
```
[Mainsail UI] → [Moonraker File API] → [Config Files]
     ↓
[JavaScript Search in Browser]
```

**Implementation:**
1. Add search component to Mainsail (Vue component)
2. On search: fetch file list from `/server/files/list?root=config`
3. Download each file's content via `/server/files/config/{file}`
4. Perform regex/text search in JavaScript
5. Display results with filename, line number, context
6. Click result → open file in editor at that line

**Pros:**
- No Moonraker modifications needed
- Works with any existing Moonraker installation
- Simpler deployment (just update Mainsail)

**Cons:**
- Slower for large config directories (must download all files)
- Search runs fresh each time (no caching)
- Browser memory usage for large configs

**Mitigation:**
- Cache file contents in browser (localStorage/IndexedDB)
- Only re-fetch changed files (use file modification timestamps)

---

### Option 2: Moonraker Extension (Complex)

**Architecture:**
```
[Mainsail UI] → [Search API] → [Moonraker Extension] → [Search Index]
                                        ↓
                               [Config Files]
```

**Implementation:**
1. Create Moonraker extension (Python)
2. Build index using library like Whoosh or simple inverted index
3. Watch for file changes, update index
4. Expose search API endpoint: `/server/config_search?query=...`
5. Create Mainsail component to call the API

**Pros:**
- Fast searches (pre-indexed)
- Lower bandwidth (only results transferred)
- Could support advanced features (fuzzy search, ranking)

**Cons:**
- Requires Moonraker extension development
- Users must install the extension
- More complex maintenance

---

## Recommendation

**Start with Option 1 (Client-Side Search)** for these reasons:

1. **Faster to MVP**: Can be developed entirely in Mainsail
2. **Easier adoption**: No server-side installation required
3. **Config files are small**: Typical Klipper configs are <100KB total
4. **Validate the UX first**: Prove the feature is useful before optimizing

**Future enhancement path:**
- If performance becomes an issue, migrate to Option 2
- The UI component remains the same, only the data source changes

---

## Next Steps

1. Set up Mainsail development environment
2. Create search UI component (banner or modal)
3. Implement file fetching and caching
4. Build search logic with result highlighting
5. Integrate with existing file editor for navigation
