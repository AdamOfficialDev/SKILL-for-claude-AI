# DAVID — State Management Reference (Scanner W)

Anti-patterns and best practices for Redux, Zustand, Context, and Pinia. Load when state management code is present.

---

## React Context Anti-Patterns

### The Re-render Problem

```jsx
// ❌ New object created every render — ALL consumers re-render every time
function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  return (
    <AppContext.Provider value={{ user, setUser, theme, setTheme }}>
      {children}
    </AppContext.Provider>
  );
}

// ✅ Split contexts by update frequency
function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  const value = useMemo(() => ({ user, setUser }), [user]);
  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
}

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const value = useMemo(() => ({ theme, setTheme }), [theme]);
  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}
```

### Context for Server State (Wrong Tool)

```jsx
// ❌ Using Context to cache API data — reinventing React Query
const DataContext = createContext(null);

function DataProvider({ children }) {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    setLoading(true);
    fetch('/api/users').then(r => r.json()).then(data => {
      setUsers(data);
      setLoading(false);
    });
  }, []);
  
  return <DataContext.Provider value={{ users, loading }}>{children}</DataContext.Provider>;
}

// ✅ Use React Query for server state
function UserList() {
  const { data: users, isLoading } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then(r => r.json()),
  });
  // Automatic caching, refetching, loading states, error states, pagination
}
```

---

## Redux Anti-Patterns

### Unmemoized Selectors

```javascript
// ❌ New array/object created every time — breaks memoization, causes re-renders
const activeUsers = useSelector(state =>
  state.users.filter(u => u.active)           // New array reference every render
);

const userStats = useSelector(state => ({
  count: state.users.length,                  // New object reference every render
  active: state.users.filter(u => u.active).length,
}));

// ✅ Memoized with reselect
import { createSelector } from '@reduxjs/toolkit';

const selectUsers = state => state.users;
const selectActiveUsers = createSelector(
  selectUsers,
  users => users.filter(u => u.active)  // Only recomputes when users array changes
);

const selectUserStats = createSelector(
  selectUsers,
  users => ({
    count: users.length,
    active: users.filter(u => u.active).length,
  })
);

// Usage
const activeUsers = useSelector(selectActiveUsers);
const stats = useSelector(selectUserStats);
```

### Derived State Stored in Redux

```javascript
// ❌ Derived data stored in Redux — gets out of sync
const counterSlice = createSlice({
  name: 'counter',
  initialState: { items: [], count: 0, isEmpty: true },
  reducers: {
    addItem(state, action) {
      state.items.push(action.payload);
      state.count = state.items.length;      // Derived — unnecessary
      state.isEmpty = state.items.length === 0; // Derived — unnecessary
    }
  }
});

// ✅ Only store primitive state, compute derived in selectors
const counterSlice = createSlice({
  name: 'counter',
  initialState: { items: [] },
  reducers: {
    addItem(state, action) { state.items.push(action.payload); }
  }
});
// Selectors:
const selectCount = state => state.counter.items.length;
const selectIsEmpty = state => state.counter.items.length === 0;
```

### Server State in Redux

```javascript
// ❌ Using Redux to manage API loading/data — too much boilerplate, wrong tool
const usersSlice = createSlice({
  name: 'users',
  initialState: { data: [], loading: false, error: null },
  reducers: {
    fetchStart(state) { state.loading = true; },
    fetchSuccess(state, action) { state.data = action.payload; state.loading = false; },
    fetchError(state, action) { state.error = action.payload; state.loading = false; },
  }
});

// ✅ Redux Toolkit Query (RTK Query) — purpose-built for this
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  endpoints: build => ({
    getUsers: build.query({ query: () => '/users' }),
    createUser: build.mutation({ query: body => ({ url: '/users', method: 'POST', body }) }),
  }),
});

// OR React Query — even simpler
const { data: users, isLoading } = useQuery({ queryKey: ['users'], queryFn: getUsers });
```

---

## Zustand Anti-Patterns

```javascript
// ❌ Selecting entire store — re-renders on ANY store change
const store = useAppStore();
const { user, settings, cart } = store;

// ✅ Select only what you need — each selector is independent
const user     = useAppStore(state => state.user);
const settings = useAppStore(state => state.settings);
const cart     = useAppStore(state => state.cart);

// ✅ Select multiple with shallow comparison
import { shallow } from 'zustand/shallow';
const { user, cart } = useAppStore(
  state => ({ user: state.user, cart: state.cart }),
  shallow  // Prevents re-render if user and cart are the same references
);

// ❌ Mutating state directly (Zustand uses Immer by default in create with immer)
const useStore = create((set) => ({
  count: 0,
  increment: () => set(state => { state.count++; }),  // Direct mutation — needs immer
}));

// ✅ Return new state object
const useStore = create((set) => ({
  count: 0,
  increment: () => set(state => ({ count: state.count + 1 })),
}));
```

---

## When to Use What

| Scenario | Tool |
|----------|------|
| Server data (API responses) | React Query / SWR / RTK Query |
| Shared UI state (modal open, theme) | Zustand / Jotai / small Context |
| Complex client state with actions | Redux Toolkit |
| Simple local component state | useState / useReducer |
| Form state | react-hook-form |
| URL state (tabs, filters, pagination) | URL search params |
| Rarely-changing global config | Context (no frequent updates) |
