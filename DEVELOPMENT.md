# Development Guide

This guide covers setting up a development environment for KlipperSearch.

## Prerequisites

- Node.js 20 or higher
- npm
- Git

## File Structure

```
mainsail/src/
├── store/search/
│   ├── index.ts        # Store module definition
│   ├── types.ts        # TypeScript interfaces
│   ├── actions.ts      # Search logic, file fetching, caching
│   ├── mutations.ts    # State mutations
│   └── getters.ts      # Computed properties
├── components/dialogs/
│   └── ConfigSearchDialog.vue  # Search UI component
└── components/
    ├── TheTopbar.vue   # Modified - added search button
    └── TheEditor.vue   # Modified - added line navigation
```

## Setup

### 1. Fork and Clone

```bash
# Fork the repository on GitHub first
# Then clone your fork
git clone https://github.com/YOUR_USERNAME/mainsail.git
cd mainsail
git checkout develop
```

### 2. Install Dependencies

```bash
npm ci
```

### 3. Configure Development Environment

```bash
# Copy the example environment file
cp .env.development.local.example .env.development.local

# Edit .env.development.local with your Moonraker host
# Example:
# VUE_APP_HOSTNAME=localhost
# VUE_APP_PORT=7125
```

### 4. Start Development Server

```bash
npm run serve
```

The development server will start at [http://localhost:8080](http://localhost:8080) and proxy API requests to your configured Moonraker instance.

## Testing

### Option 1: Virtual Klipper Printer (Recommended)

For testing without a physical printer, use the [virtual-klipper-printer](https://github.com/mainsail-crew/virtual-klipper-printer) Docker project:

```bash
# Clone the virtual printer repo
git clone https://github.com/mainsail-crew/virtual-klipper-printer
cd virtual-klipper-printer

# Start the virtual printer
docker-compose up -d

# Moonraker will be available at http://localhost:7125
```

Then configure your `.env.development.local`:
```
VUE_APP_HOSTNAME=localhost
VUE_APP_PORT=7125
```

### Option 2: Physical Printer

If you have a physical printer running Klipper + Moonraker, you can point your development server to it:

```
VUE_APP_HOSTNAME=your-printer-ip
VUE_APP_PORT=80
```

## Code Quality

### Linting

Run ESLint to check for code issues:

```bash
npm run lint
```

Fix auto-fixable issues:

```bash
npm run lint -- --fix
```

### Type Checking

TypeScript types are checked during build. To check manually:

```bash
npm run build
```

## Building

Build the production version:

```bash
npm run build
```

The built files will be in the `dist/` directory.

## Search Feature Architecture

### How It Works

1. **User opens search dialog** (`Ctrl+Shift+F` or click search icon)
2. **ConfigSearchDialog.vue renders** with search input
3. **User types query** (debounced 300ms)
4. **Search action dispatched** to Vuex store
5. **getConfigFiles action** retrieves file list from `rootState.files.filetree`
6. **For each file**:
   - Check cache first (with modification time check)
   - If not cached or stale, fetch via Moonraker API
   - Search file content with regex
7. **Results returned** with filename, line number, content, context
8. **User clicks result** → `openResult` action triggered
9. **Editor opened** at specific line number

### Key Components

#### Vuex Store Module (`src/store/search/`)

- **State**: Query, results, loading state, file cache
- **Actions**:
  - `search(query)` - Main search orchestration
  - `performSearch(query)` - Core search logic
  - `getConfigFiles()` - Retrieves file list from filetree
  - `getFileContent(file)` - Fetches and caches file content
  - `openResult(result)` - Opens file in editor at line
- **Mutations**: State updates
- **Getters**: Computed properties (hasResults, resultCount)

#### UI Component (`ConfigSearchDialog.vue`)

- Search input with debouncing
- Results list with highlighting
- Keyboard navigation (Enter, Up, Down)
- Click to open file

#### Integration Points

- **TheTopbar.vue**: Search button + keyboard shortcut handler
- **TheEditor.vue**: Line navigation via `searchTargetLine` watcher

### API Endpoints Used

- `GET /server/files/list?root=config` - List config files (via existing filetree)
- `GET /server/files/config/{filename}` - Fetch file content

### Caching Strategy

Files are cached with modification timestamps:
```typescript
fileCache: {
  'printer.cfg': {
    content: '...',
    modified: 1234567890
  }
}
```

Cache is invalidated when file's `modified` timestamp is newer than cached version.

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-feature`)
3. Make your changes
4. Run linter (`npm run lint`)
5. Test your changes
6. Commit with conventional commits format
7. Push to your fork
8. Submit a pull request

### Commit Message Format

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add search result highlighting
fix: resolve cache invalidation issue
docs: update development guide
chore: upgrade dependencies
```

## Debugging

### Browser DevTools

1. Open browser console (F12)
2. Check for errors in Console tab
3. Use Vue DevTools extension to inspect:
   - Component state
   - Vuex store state
   - Events

### Common Issues

#### Search returns no results

- Check browser console for API errors
- Verify Moonraker is connected
- Check that config files exist in the filetree

#### Editor doesn't navigate to line

- Check that `searchTargetLine` is set in Vuex state
- Verify `TheEditor.vue` watcher is triggering
- Check CodeMirror `gotoLine` method is available

#### Cache not working

- Check file modification timestamps
- Verify cache invalidation logic in `getFileContent`

## Resources

- [Vue.js 2 Documentation](https://v2.vuejs.org/)
- [Vuex Documentation](https://v3.vuex.vuejs.org/)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [Vuetify 2 Documentation](https://v2.vuetifyjs.com/)
- [Moonraker API Documentation](https://moonraker.readthedocs.io/)
