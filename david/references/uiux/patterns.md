# DAVID — UI/UX Audit: Specialized UX Patterns

Extended specialized pattern library for Scanner O.
Load for: search UX, data tables, file upload, onboarding, wizards, real-time, offline, media, keyboard, data viz, clipboard, drag-and-drop, trust signals.

## Table of Contents
1. [Search UX Patterns](#search-ux-patterns)
2. [Table / Data Grid UX](#table--data-grid-ux)
3. [File Upload UX](#file-upload-ux)
4. [Onboarding UX Patterns](#onboarding-ux-patterns)
5. [Multi-Step Wizard UX](#multi-step-wizard-ux-enhanced)
6. [Real-Time UX Patterns](#real-time-ux-patterns)
7. [Offline / Connectivity UX](#offline--connectivity-ux)
8. [Image & Media UX](#image--media-ux)
9. [Keyboard & Power User UX](#keyboard--power-user-ux)
10. [Data Visualization UX](#data-visualization-ux)
11. [Copy to Clipboard UX](#copy-to-clipboard-ux)
12. [Drag and Drop UX](#drag-and-drop-ux)
13. [Trust Signal UX](#trust-signal-ux)

---

## Search UX Patterns

### Search Input Requirements

```jsx
// ✅ Complete search input implementation
function SearchInput({ onSearch, placeholder = "Search..." }) {
  const [query, setQuery] = useState('');
  const [isFocused, setIsFocused] = useState(false);
  const debouncedSearch = useMemo(() => debounce(onSearch, 300), [onSearch]);

  function handleChange(e) {
    setQuery(e.target.value);
    debouncedSearch(e.target.value);
  }

  function handleClear() {
    setQuery('');
    onSearch('');
  }

  return (
    <div role="search">
      <label htmlFor="search" className="sr-only">Search</label>
      <div className="relative">
        <Search size={16} className="absolute left-3 top-1/2 -translate-y-1/2 text-gray-400" aria-hidden />
        <input
          id="search"
          type="search"           {/* Enables "x" clear button natively + correct keyboard */}
          inputMode="search"
          value={query}
          onChange={handleChange}
          onFocus={() => setIsFocused(true)}
          onBlur={() => setIsFocused(false)}
          placeholder={placeholder}
          autoComplete="off"
          autoCorrect="off"
          spellCheck={false}
          className="pl-9 pr-9 w-full"
        />
        {query && (
          <button
            onClick={handleClear}
            aria-label="Clear search"
            className="absolute right-3 top-1/2 -translate-y-1/2"
          >
            <X size={14} />
          </button>
        )}
      </div>
    </div>
  );
}
```

### Search Results States

```jsx
// All 4 states required for search:
function SearchResults({ query, results, isLoading }) {
  // State 1: Empty query — show suggestions or recent searches
  if (!query) return <RecentSearches />;

  // State 2: Loading
  if (isLoading) return <SearchResultsSkeleton count={5} />;

  // State 3: No results — specific, helpful
  if (results.length === 0) return (
    <div className="text-center py-12">
      <Search className="mx-auto h-10 w-10 text-gray-300" />
      <p className="mt-3 font-medium">No results for "{query}"</p>
      <p className="mt-1 text-sm text-gray-500">
        Try different keywords or{' '}
        <button onClick={clearFilters} className="text-blue-600">clear filters</button>
      </p>
    </div>
  );

  // State 4: Results — highlight matching text
  return (
    <ul>
      {results.map(result => (
        <li key={result.id}>
          <HighlightedText text={result.title} highlight={query} />
        </li>
      ))}
    </ul>
  );
}

// Highlight matching text in results
function HighlightedText({ text, highlight }) {
  if (!highlight) return <span>{text}</span>;
  const parts = text.split(new RegExp(`(${highlight})`, 'gi'));
  return (
    <span>
      {parts.map((part, i) =>
        part.toLowerCase() === highlight.toLowerCase()
          ? <mark key={i} className="bg-yellow-100 text-yellow-900">{part}</mark>
          : <span key={i}>{part}</span>
      )}
    </span>
  );
}
```

---

## Table / Data Grid UX

### Sortable Table Pattern

```jsx
// ✅ Accessible sortable table
function DataTable({ columns, rows }) {
  const [sort, setSort] = useState({ column: null, direction: 'asc' });

  function handleSort(column) {
    setSort(prev => ({
      column,
      direction: prev.column === column && prev.direction === 'asc' ? 'desc' : 'asc'
    }));
  }

  return (
    <table>
      <thead>
        <tr>
          {columns.map(col => (
            <th
              key={col.key}
              aria-sort={
                sort.column === col.key
                  ? sort.direction === 'asc' ? 'ascending' : 'descending'
                  : 'none'
              }
            >
              <button
                onClick={() => handleSort(col.key)}
                className="flex items-center gap-1"
              >
                {col.label}
                <SortIcon
                  active={sort.column === col.key}
                  direction={sort.direction}
                />
              </button>
            </th>
          ))}
        </tr>
      </thead>
      <tbody>
        {sortedRows.map(row => (
          <tr key={row.id} className="hover:bg-gray-50 dark:hover:bg-gray-800/50">
            {columns.map(col => (
              <td key={col.key}>{row[col.key]}</td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

### Table Empty, Loading, Error States

```jsx
function TableBody({ isLoading, error, rows, columns, onRetry }) {
  if (isLoading) return (
    <tbody aria-busy="true" aria-label="Loading data">
      {[...Array(5)].map((_, i) => (
        <tr key={i} className="animate-pulse">
          {columns.map(col => (
            <td key={col.key}>
              <div className="h-4 bg-gray-200 dark:bg-gray-700 rounded w-3/4" />
            </td>
          ))}
        </tr>
      ))}
    </tbody>
  );

  if (error) return (
    <tbody>
      <tr>
        <td colSpan={columns.length} className="text-center py-12">
          <AlertCircle className="mx-auto mb-2 text-red-400" />
          <p className="text-sm text-gray-600">Failed to load data</p>
          <button onClick={onRetry} className="mt-2 btn-secondary btn-sm">Try again</button>
        </td>
      </tr>
    </tbody>
  );

  if (!rows.length) return (
    <tbody>
      <tr>
        <td colSpan={columns.length} className="text-center py-12">
          <p className="text-sm text-gray-500">No records found</p>
        </td>
      </tr>
    </tbody>
  );

  return <tbody>{/* normal rows */}</tbody>;
}
```

---

## File Upload UX

### Drag and Drop Zone

```jsx
// ✅ Complete file upload UX
function FileUploadZone({ onFilesSelected, accept, maxSizeMB = 10 }) {
  const [isDragging, setIsDragging] = useState(false);
  const [error, setError] = useState(null);
  const [previews, setPreviews] = useState([]);

  function validateFile(file) {
    if (maxSizeMB && file.size > maxSizeMB * 1024 * 1024) {
      return `File too large. Maximum size is ${maxSizeMB}MB`;
    }
    if (accept && !accept.split(',').some(t => file.type.match(t.trim()))) {
      return `Invalid file type. Accepted: ${accept}`;
    }
    return null;
  }

  function handleFiles(files) {
    const file = files[0];
    const validationError = validateFile(file);
    if (validationError) {
      setError(validationError);
      return;
    }
    setError(null);
    setPreviews([{ name: file.name, size: file.size, url: URL.createObjectURL(file) }]);
    onFilesSelected(files);
  }

  return (
    <div>
      <div
        role="button"
        tabIndex={0}
        aria-label="Upload file — drag and drop or click to browse"
        onDragOver={e => { e.preventDefault(); setIsDragging(true); }}
        onDragLeave={() => setIsDragging(false)}
        onDrop={e => { e.preventDefault(); setIsDragging(false); handleFiles(e.dataTransfer.files); }}
        onClick={() => fileInputRef.current?.click()}
        onKeyDown={e => e.key === 'Enter' && fileInputRef.current?.click()}
        className={`border-2 border-dashed rounded-xl p-8 text-center cursor-pointer transition-colors
          ${isDragging
            ? 'border-blue-500 bg-blue-50 dark:bg-blue-950'
            : 'border-gray-300 dark:border-gray-600 hover:border-gray-400'
          }`}
      >
        <Upload className="mx-auto mb-3 text-gray-400" size={32} />
        <p className="font-medium text-gray-700 dark:text-gray-300">
          Drop files here, or <span className="text-blue-600">browse</span>
        </p>
        <p className="mt-1 text-sm text-gray-500">
          {accept} up to {maxSizeMB}MB
        </p>
      </div>

      {error && (
        <p role="alert" className="mt-2 text-sm text-red-600">{error}</p>
      )}

      {previews.map(file => (
        <div key={file.name} className="mt-3 flex items-center gap-3 p-3 border rounded-lg">
          <FileIcon type={file.name.split('.').pop()} />
          <div className="flex-1 min-w-0">
            <p className="text-sm font-medium truncate">{file.name}</p>
            <p className="text-xs text-gray-500">{formatBytes(file.size)}</p>
          </div>
          <button onClick={() => setPreviews([])} aria-label="Remove file">
            <X size={16} />
          </button>
        </div>
      ))}

      <input
        ref={fileInputRef}
        type="file"
        accept={accept}
        onChange={e => handleFiles(e.target.files)}
        className="sr-only"
        tabIndex={-1}
      />
    </div>
  );
}
```

---

## Onboarding UX Patterns

### First-Run Experience Requirements

```jsx
// ✅ Contextual onboarding — show at first use, not before
function EmptyDashboard({ isFirstRun, onCreateFirst }) {
  if (!isFirstRun) return <EmptyState message="No data yet" />;

  return (
    <div className="max-w-lg mx-auto text-center py-16">
      {/* Welcome moment */}
      <div className="text-5xl mb-4">👋</div>
      <h1 className="text-2xl font-bold">Welcome to [App Name]</h1>
      <p className="mt-2 text-gray-500">
        You're all set. Let's create your first project to get started.
      </p>

      {/* Primary action — ONE clear next step */}
      <button onClick={onCreateFirst} className="mt-8 btn-primary btn-lg">
        Create your first project
      </button>

      {/* Optional secondary — for users who prefer to explore */}
      <button className="mt-3 btn-ghost text-sm">
        Take a quick tour instead
      </button>

      {/* Trust: "how long will this take" */}
      <p className="mt-4 text-xs text-gray-400">Takes about 2 minutes</p>
    </div>
  );
}
```

### Checklist-Style Onboarding

```jsx
// ✅ Onboarding progress with clear milestones
function OnboardingChecklist({ steps, completedSteps }) {
  const progress = (completedSteps.length / steps.length) * 100;

  return (
    <div className="bg-blue-50 dark:bg-blue-950 rounded-xl p-6">
      <div className="flex items-center justify-between mb-4">
        <h3 className="font-semibold">Get started ({completedSteps.length}/{steps.length})</h3>
        <button
          onClick={() => localStorage.setItem('onboarding-dismissed', 'true')}
          className="text-sm text-gray-500 hover:text-gray-700"
        >
          Dismiss
        </button>
      </div>

      {/* Progress bar */}
      <div className="w-full bg-blue-100 rounded-full h-1.5 mb-4">
        <div
          className="bg-blue-600 h-1.5 rounded-full transition-all duration-500"
          style={{ width: `${progress}%` }}
        />
      </div>

      {steps.map(step => {
        const isDone = completedSteps.includes(step.id);
        return (
          <div key={step.id} className="flex items-start gap-3 mb-3">
            <div className={`mt-0.5 rounded-full w-5 h-5 flex-shrink-0 flex items-center justify-center
              ${isDone ? 'bg-blue-600' : 'border-2 border-gray-300'}`}>
              {isDone && <Check size={12} className="text-white" />}
            </div>
            <div>
              <p className={`text-sm font-medium ${isDone ? 'line-through text-gray-400' : ''}`}>
                {step.label}
              </p>
              {!isDone && (
                <a href={step.href} className="text-xs text-blue-600 mt-0.5 block">
                  {step.cta} →
                </a>
              )}
            </div>
          </div>
        );
      })}
    </div>
  );
}
```

---

## Multi-Step Wizard UX (Enhanced)

### Progress Indicator Patterns

```jsx
// ✅ Linear step indicator — for sequential, non-skippable flows
function StepIndicator({ steps, currentStep }) {
  return (
    <nav aria-label="Progress">
      <ol className="flex items-center">
        {steps.map((step, i) => {
          const status = i < currentStep ? 'complete' : i === currentStep ? 'current' : 'upcoming';
          return (
            <li key={i} className="relative flex-1">
              {/* Connector line */}
              {i < steps.length - 1 && (
                <div className={`absolute top-4 left-1/2 w-full h-0.5
                  ${status === 'complete' ? 'bg-blue-600' : 'bg-gray-200'}`}
                />
              )}
              <div className="relative flex flex-col items-center">
                {/* Step circle */}
                <div className={`w-8 h-8 rounded-full flex items-center justify-center text-sm font-medium z-10
                  ${status === 'complete' ? 'bg-blue-600 text-white' :
                    status === 'current' ? 'border-2 border-blue-600 text-blue-600' :
                    'border-2 border-gray-300 text-gray-400'}`}
                  aria-current={status === 'current' ? 'step' : undefined}
                >
                  {status === 'complete' ? <Check size={16} /> : i + 1}
                </div>
                <span className={`mt-2 text-xs font-medium
                  ${status === 'current' ? 'text-blue-600' : 'text-gray-500'}`}>
                  {step.label}
                </span>
              </div>
            </li>
          );
        })}
      </ol>
    </nav>
  );
}
```

### Auto-Save in Wizards

```jsx
// ✅ Auto-save with status feedback
function useWizardAutoSave(formData, stepId) {
  const [saveStatus, setSaveStatus] = useState('idle'); // idle | saving | saved | error

  const debouncedSave = useMemo(() => debounce(async (data) => {
    setSaveStatus('saving');
    try {
      await api.saveDraft({ stepId, data });
      setSaveStatus('saved');
      setTimeout(() => setSaveStatus('idle'), 2000);
    } catch {
      setSaveStatus('error');
    }
  }, 1000), [stepId]);

  useEffect(() => {
    if (Object.keys(formData).length) debouncedSave(formData);
  }, [formData]);

  return saveStatus;
}

// In wizard component:
const saveStatus = useWizardAutoSave(formData, currentStep);
{saveStatus === 'saving' && <span className="text-xs text-gray-400">Saving draft...</span>}
{saveStatus === 'saved'  && <span className="text-xs text-green-600">Draft saved ✓</span>}
{saveStatus === 'error'  && <span className="text-xs text-red-600">Couldn't save draft</span>}
```

---

## Real-Time UX Patterns

### Live Update Indicators

```jsx
// ✅ Real-time data with freshness indicator
function LiveDashboard() {
  const [lastUpdated, setLastUpdated] = useState(new Date());
  const [isConnected, setIsConnected] = useState(true);

  return (
    <div>
      {/* Connection status — subtle, non-intrusive */}
      <div className="flex items-center gap-1.5 text-xs text-gray-400">
        <div className={`w-1.5 h-1.5 rounded-full ${isConnected ? 'bg-green-500' : 'bg-red-400'}`} />
        {isConnected
          ? `Live · Updated ${formatRelativeTime(lastUpdated)}`
          : 'Reconnecting...'
        }
      </div>
      {/* Dashboard content */}
    </div>
  );
}

// ✅ Typing indicator (chat/collab)
function TypingIndicator({ typingUsers }) {
  if (!typingUsers.length) return null;

  const text = typingUsers.length === 1
    ? `${typingUsers[0]} is typing...`
    : typingUsers.length === 2
    ? `${typingUsers[0]} and ${typingUsers[1]} are typing...`
    : `${typingUsers.length} people are typing...`;

  return (
    <p className="text-xs text-gray-500 italic" aria-live="polite" aria-atomic="true">
      {text}
    </p>
  );
}
```

---

## Offline / Connectivity UX

```jsx
// ✅ Connectivity-aware UI
function ConnectivityBanner() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const handleOnline  = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    window.addEventListener('online',  handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online',  handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  if (isOnline) return null;

  return (
    <div
      role="alert"
      aria-live="assertive"
      className="fixed top-0 inset-x-0 z-50 bg-yellow-50 border-b border-yellow-200 px-4 py-2"
    >
      <div className="flex items-center gap-2 text-sm text-yellow-800">
        <WifiOff size={16} />
        <span>You're offline. Changes will sync when you reconnect.</span>
      </div>
    </div>
  );
}
```

---

## Image & Media UX

### Progressive Image Loading

```jsx
// ✅ Blur-up progressive loading pattern
function ProgressiveImage({ src, placeholderSrc, alt, ...props }) {
  const [loaded, setLoaded] = useState(false);

  return (
    <div className="relative overflow-hidden">
      {/* Blurred placeholder */}
      <img
        src={placeholderSrc}      // Tiny 20px version
        alt=""
        aria-hidden="true"
        className={`absolute inset-0 w-full h-full object-cover scale-110 blur-sm
          transition-opacity duration-300 ${loaded ? 'opacity-0' : 'opacity-100'}`}
      />
      {/* Full quality image */}
      <img
        src={src}
        alt={alt}
        onLoad={() => setLoaded(true)}
        className={`relative w-full h-full object-cover
          transition-opacity duration-300 ${loaded ? 'opacity-100' : 'opacity-0'}`}
        {...props}
      />
    </div>
  );
}

// ✅ Broken image fallback
function SafeImage({ src, alt, fallback = '/placeholder.svg', ...props }) {
  const [error, setError] = useState(false);
  return (
    <img
      src={error ? fallback : src}
      alt={alt}
      onError={() => setError(true)}
      {...props}
    />
  );
}
```

---

## Keyboard & Power User UX

### Command Palette Pattern

```jsx
// ✅ Command palette — K shortcut standard
function CommandPalette({ commands, isOpen, onClose }) {
  const [query, setQuery] = useState('');
  const [selected, setSelected] = useState(0);

  const filtered = commands.filter(c =>
    c.label.toLowerCase().includes(query.toLowerCase())
  );

  function handleKeyDown(e) {
    switch(e.key) {
      case 'ArrowDown': e.preventDefault(); setSelected(s => Math.min(s + 1, filtered.length - 1)); break;
      case 'ArrowUp':   e.preventDefault(); setSelected(s => Math.max(s - 1, 0)); break;
      case 'Enter':     filtered[selected]?.action(); onClose(); break;
      case 'Escape':    onClose(); break;
    }
  }

  if (!isOpen) return null;

  return (
    <div role="dialog" aria-label="Command palette" aria-modal="true">
      {/* Backdrop */}
      <div className="fixed inset-0 bg-black/40 z-40" onClick={onClose} />

      {/* Palette */}
      <div className="fixed top-1/4 left-1/2 -translate-x-1/2 w-full max-w-xl bg-white dark:bg-gray-900 rounded-xl shadow-2xl z-50 overflow-hidden">
        <input
          autoFocus
          type="search"
          value={query}
          onChange={e => { setQuery(e.target.value); setSelected(0); }}
          onKeyDown={handleKeyDown}
          placeholder="Search commands..."
          className="w-full px-4 py-3 text-base border-b outline-none"
          aria-label="Search commands"
          aria-autocomplete="list"
          aria-controls="command-list"
          aria-activedescendant={`cmd-${selected}`}
        />
        <ul id="command-list" role="listbox" className="max-h-80 overflow-auto py-2">
          {filtered.map((cmd, i) => (
            <li
              key={cmd.id}
              id={`cmd-${i}`}
              role="option"
              aria-selected={i === selected}
              onClick={() => { cmd.action(); onClose(); }}
              className={`flex items-center gap-3 px-4 py-2.5 cursor-pointer text-sm
                ${i === selected ? 'bg-blue-50 dark:bg-blue-900/30 text-blue-700' : ''}`}
            >
              {cmd.icon && <cmd.icon size={16} className="text-gray-400" />}
              <span>{cmd.label}</span>
              {cmd.shortcut && <kbd className="ml-auto text-xs text-gray-400">{cmd.shortcut}</kbd>}
            </li>
          ))}
          {!filtered.length && (
            <li className="px-4 py-8 text-center text-sm text-gray-400">No commands found</li>
          )}
        </ul>
      </div>
    </div>
  );
}
```

---

## Data Visualization UX

### Chart Accessibility & Readability

```jsx
// ✅ Accessible chart with fallback data table
function AccessibleChart({ data, title }) {
  return (
    <figure>
      <figcaption className="text-sm font-medium text-gray-700 mb-3">{title}</figcaption>

      {/* Visual chart for sighted users */}
      <div aria-hidden="true">
        <LineChart data={data} />
      </div>

      {/* Data table for screen readers + keyboard users */}
      <details className="mt-4">
        <summary className="text-xs text-blue-600 cursor-pointer">View as table</summary>
        <table className="mt-2 w-full text-xs">
          <thead>
            <tr>
              <th>Date</th>
              <th>Value</th>
            </tr>
          </thead>
          <tbody>
            {data.map(d => (
              <tr key={d.date}>
                <td>{d.date}</td>
                <td>{d.value}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </details>
    </figure>
  );
}
```

### Color-Blind Safe Palette

```javascript
// ✅ Color-blind safe palette for charts (works for deuteranopia, protanopia, tritanopia)
const CHART_COLORS = {
  blue:   '#0077BB',  // Safe
  orange: '#EE7733',  // Safe
  teal:   '#009988',  // Safe
  red:    '#CC3311',  // Safe (not paired with green)
  purple: '#AA3377',  // Safe
  grey:   '#BBBBBB',  // Always safe
  // ❌ NEVER pair: red + green (most common form of color blindness)
};

// ✅ Pattern + color (double encoding for maximum safety)
// Use dashed lines, dotted lines, different markers in addition to color
```

---

## Copy to Clipboard UX

```jsx
// ✅ Complete clipboard UX with feedback
function CopyButton({ text, label = "Copy" }) {
  const [status, setStatus] = useState('idle'); // idle | copied | error

  async function handleCopy() {
    try {
      await navigator.clipboard.writeText(text);
      setStatus('copied');
      setTimeout(() => setStatus('idle'), 2000);
    } catch {
      // Fallback for older browsers
      const el = document.createElement('textarea');
      el.value = text;
      document.body.appendChild(el);
      el.select();
      document.execCommand('copy');
      document.body.removeChild(el);
      setStatus('copied');
      setTimeout(() => setStatus('idle'), 2000);
    }
  }

  return (
    <button
      onClick={handleCopy}
      aria-label={status === 'copied' ? 'Copied to clipboard' : `Copy ${label} to clipboard`}
      className="flex items-center gap-1.5 text-sm"
    >
      {status === 'copied'
        ? <><Check size={14} className="text-green-600" /> Copied!</>
        : <><Copy size={14} /> {label}</>
      }
    </button>
  );
}
```

---

## Drag and Drop UX

```jsx
// ✅ Accessible drag and drop with keyboard fallback
function DraggableList({ items, onReorder }) {
  // Visual DnD for mouse users
  // Keyboard fallback for accessibility
  const [draggedItem, setDraggedItem] = useState(null);
  const [selectedForMove, setSelectedForMove] = useState(null);

  return (
    <ul>
      {items.map((item, i) => (
        <li
          key={item.id}
          draggable
          onDragStart={() => setDraggedItem(item.id)}
          onDragOver={e => { e.preventDefault(); }}
          onDrop={() => onReorder(draggedItem, item.id)}
          className="flex items-center gap-3"
        >
          {/* Drag handle — explicit, not whole row */}
          <button
            aria-label={`Drag to reorder ${item.label}`}
            className="cursor-grab text-gray-400 hover:text-gray-600"
          >
            <GripVertical size={16} />
          </button>

          {/* Keyboard reorder */}
          <button
            aria-label={selectedForMove === item.id
              ? `Moving ${item.label} — press Up/Down to position`
              : `Move ${item.label}`}
            onClick={() => setSelectedForMove(
              selectedForMove === item.id ? null : item.id
            )}
          >
            <ArrowUpDown size={14} />
          </button>

          {item.label}
        </li>
      ))}
    </ul>
  );
}
```

---

## Trust Signal UX

```jsx
// ✅ Trust signals at the right moments
function CheckoutTrustBar() {
  return (
    <div className="flex flex-wrap gap-4 justify-center text-sm text-gray-600 py-4">
      <div className="flex items-center gap-1.5">
        <Lock size={14} className="text-green-600" />
        <span>256-bit SSL encryption</span>
      </div>
      <div className="flex items-center gap-1.5">
        <Shield size={14} className="text-green-600" />
        <span>Secure payment</span>
      </div>
      <div className="flex items-center gap-1.5">
        <RotateCcw size={14} className="text-blue-600" />
        <span>30-day money-back guarantee</span>
      </div>
    </div>
  );
}

// ✅ Social proof — show near primary CTA for max impact
function PricingCard({ plan }) {
  return (
    <div>
      <h3>{plan.name}</h3>
      <p>{plan.price}</p>

      {/* Social proof immediately before CTA */}
      <p className="text-xs text-gray-500 mb-3">
        Joined by {plan.customerCount.toLocaleString()}+ teams
      </p>

      <button className="btn-primary w-full">Get started</button>
    </div>
  );
}
```

