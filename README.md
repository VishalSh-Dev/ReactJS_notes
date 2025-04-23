# 📘 React.js Revision Notes: From Fundamentals to Advanced Concepts

A comprehensive practical guide to React organized for quick revision and deep understanding with hands-on examples.

---

## 1. 🌱 React Fundamentals

### 🔹 Core Concepts

- **Component-Based Architecture**: UI is built using reusable and composable components
- **JSX (JavaScript XML)**: Lets you write HTML-like syntax directly in JS
- **Virtual DOM**: React's efficient way of updating the UI by only re-rendering changed elements
- **One-Way Data Flow**: Data passed from parent to child using props
- **Declarative UI**: Describe what the UI should look like based on current state, not how to change it

### 🔹 Component Types

#### Functional Component (Modern Approach)

```jsx
function Home() {
  return <div>Welcome to Home</div>;
}
```

#### Class Component (Legacy)

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

---

## 2. 🧠 State Management in React

### 🔹 useState Hook

Used in: `CurrencyConverter`, `TodoApp`

```jsx
const [amount, setAmount] = useState(0);
const [fromCurrency, setFromCurrency] = useState("usd");
```

**🚀 Key Points:**
- Returns a pair: current state and a function to update it
- State changes trigger re-renders
- Updates are asynchronous (state updates may be batched)

### 🔹 Props

Used to pass data from parent to child components:

```jsx
function Inputbox({ label, amount, onAmountChange, onCurrencyChange }) {
  return (
    <input value={amount} onChange={onAmountChange} />
  );
}
```

**🚀 Key Points:**
- Props are read-only
- Changes in props cause re-renders
- Props can be any JavaScript value including functions and objects

---

## 3. ⚓ React Hooks

### 🔹 useEffect

Used in: `CurrencyConverter`, `ThemeSwitcher`, `TodoWithContext`

```jsx
useEffect(() => {
  fetch(`https://.../${currency}.json`)
    .then(res => res.json())
    .then(data => setData(data[currency]));
  
  // Cleanup function (optional)
  return () => {
    // Clean up resources or subscriptions
  };
}, [currency]); // Dependency array
```

**🚀 Use for:**
- Fetching API data
- DOM updates and manipulations
- Listening to changes (e.g., localStorage, subscriptions)
- Side effects when dependencies change

### 🔹 useContext

From `TodoWithContext`, `ThemeSwitcher`

```jsx
const useTodoContext = () => {
  const context = useContext(TodoContext);
  if (!context) throw new Error('Must use inside Provider');
  return context;
};
```

**🚀 Key Points:**
- Consumes data from nearest Provider above
- Updates when Provider value changes
- Helps avoid prop drilling

### 🔹 useId

Used for accessibility and unique identifiers:

```jsx
const id = useId();
<label htmlFor={id}>Currency</label>
<input id={id} />
```

**🚀 Key Points:**
- Generates stable unique IDs
- Works with SSR (Server-Side Rendering)
- Ideal for accessibility attributes

---

## 4. 🔁 Custom Hooks

From `useCurrencyInfo.js` in `CurrencyConverter`

```jsx
function useCurrencyInfo(currency) {
  const [data, setData] = useState({});
  useEffect(() => {
    fetch(`https://.../${currency}.json`)
      .then(res => res.json())
      .then(data => setData(data[currency]));
  }, [currency]);
  return data;
}
```

**🚀 Benefits:**
- Reuse logic across components
- Cleaner component code
- Better organization and separation of concerns
- Can combine multiple React hooks

---

## 5. 🌍 Context API

### Creating and Using Context: `ThemeSwitcher`

```jsx
// Create context with default values
export const ThemeContext = createContext({
  themeMode: "light",
  darkTheme: () => {},
  lightTheme: () => {},
});

// Provider component
const ThemeProvider = ({ children }) => {
  const [themeMode, setThemeMode] = useState("light");
  
  const darkTheme = () => setThemeMode("dark");
  const lightTheme = () => setThemeMode("light");
  
  return (
    <ThemeContext.Provider value={{ themeMode, darkTheme, lightTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// Usage
<ThemeProvider>
  <App />
</ThemeProvider>
```

**Used for:**
- Theme switching
- Global state (Todos, Auth)
- User preferences
- Internationalization

---

## 6. 🚦 React Router

### Setup Example: `reactRouter`

```jsx
const router = createBrowserRouter([
  {
    path: "/",
    Component: Layout,
    children: [
      { index: true, Component: Home },
      { path: "about", Component: About },
      { path: "contact", Component: Contact },
      { path: "github", Component: Github, loader: getApiinfo },
      { path: "*", Component: () => <h1>404</h1> },
    ],
  },
]);

// Root component
<RouterProvider router={router} />
```

### Navigation Components

```jsx
// Navigation links
<NavLink to="/about" className={({ isActive }) => isActive ? "active" : ""}>
  About
</NavLink>

// Programmatic navigation
import { useNavigate } from 'react-router-dom';

function SomeComponent() {
  const navigate = useNavigate();
  
  const handleClick = () => {
    navigate('/dashboard');
  };
  
  return <button onClick={handleClick}>Go to Dashboard</button>;
}
```

### Route Loaders

```jsx
export const getApiinfo = async () => {
  const response = await fetch("https://randomuser.me/api/");
  return response.json();
};

// Access loader data in component
import { useLoaderData } from 'react-router-dom';

function Github() {
  const data = useLoaderData();
  return <div>{JSON.stringify(data)}</div>;
}
```

---

## 7. 🧰 Redux Toolkit

### Store Setup in `TodoWithRedux`

```jsx
export const store = configureStore({
  reducer: todoReducer
});

// Provide store to app
<Provider store={store}>
  <App />
</Provider>
```

### Slices

```jsx
import { createSlice, nanoid } from '@reduxjs/toolkit';

const todoSlice = createSlice({
  name: "todo",
  initialState: { todos: [] },
  reducers: {
    addTodo: {
      reducer(state, action) {
        state.todos.push(action.payload);
      },
      prepare(todo) {
        return { payload: { id: nanoid(), todo, completed: false } };
      }
    },
    removeTodo(state, action) {
      state.todos = state.todos.filter(todo => todo.id !== action.payload);
    },
    toggleComplete(state, action) {
      const todo = state.todos.find(todo => todo.id === action.payload);
      if (todo) todo.completed = !todo.completed;
    }
  }
});

export const { addTodo, removeTodo, toggleComplete } = todoSlice.actions;
export default todoSlice.reducer;
```

### Using Redux in Components

```jsx
import { useSelector, useDispatch } from 'react-redux';
import { addTodo, removeTodo } from './todoSlice';

function TodoComponent() {
  const todos = useSelector(state => state.todos);
  const dispatch = useDispatch();
  
  const handleAdd = (text) => {
    dispatch(addTodo(text));
  };
  
  const handleRemove = (id) => {
    dispatch(removeTodo(id));
  };
  
  return (
    <div>
      {/* component implementation */}
    </div>
  );
}
```

---

## 8. 💅 UI Libraries - Tailwind CSS

Used in all major projects

```jsx
<div className={`flex px-4 py-3 shadow-sm ${todo.completed ? "bg-green-100" : "bg-gray-100"}`}>
  {todo.task}
</div>
```

**🚀 Advantages:**
- Responsive design with built-in utilities
- Utility-first classes for rapid development
- Easy theming and customization
- Consistent styling system
- Reduced CSS bundle size with PurgeCSS

---

## 9. ✍️ Forms and User Input

### Controlled Components Example

```jsx
const [name, setName] = useState('');
const [email, setEmail] = useState('');

<input
  type="text"
  name="name"
  value={name}
  onChange={(e) => setName(e.target.value)}
/>

<input
  type="email"
  name="email"
  value={email}
  onChange={(e) => setEmail(e.target.value)}
/>
```

### Form Submission

```jsx
const handleSubmit = (e) => {
  e.preventDefault(); // Prevent page refresh
  submitData({ name, email });
};

<form onSubmit={handleSubmit}>
  {/* form elements */}
  <button type="submit">Submit</button>
</form>
```

### Form Validation

```jsx
const [errors, setErrors] = useState({});

const validate = () => {
  let tempErrors = {};
  if (!name) tempErrors.name = "Name is required";
  if (!email) tempErrors.email = "Email is required";
  else if (!/\S+@\S+\.\S+/.test(email)) tempErrors.email = "Email is invalid";
  
  setErrors(tempErrors);
  return Object.keys(tempErrors).length === 0;
};

const handleSubmit = (e) => {
  e.preventDefault();
  if (validate()) {
    submitData({ name, email });
  }
};
```

---

## 10. 📡 API Integration

### Fetch Example

```jsx
fetch("https://randomuser.me/api/")
  .then(res => {
    if (!res.ok) throw new Error('Network response not ok');
    return res.json();
  })
  .then(data => console.log(data))
  .catch(error => console.error('Error fetching data:', error));
```

### With Loading/Error States

```jsx
const [data, setData] = useState(null);
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);

useEffect(() => {
  setLoading(true);
  
  fetch("https://api.example.com/data")
    .then(res => {
      if (!res.ok) throw new Error('Network response failed');
      return res.json();
    })
    .then(data => {
      setData(data);
      setError(null);
    })
    .catch(err => {
      setError(err.message);
      setData(null);
    })
    .finally(() => {
      setLoading(false);
    });
}, []);

// In your JSX
{loading && <p>Loading...</p>}
{error && <p>Error: {error}</p>}
{data && <DisplayData data={data} />}
```

### Using Async/Await

```jsx
const fetchData = async () => {
  try {
    setLoading(true);
    const response = await fetch("https://api.example.com/data");
    if (!response.ok) throw new Error('Network response failed');
    const data = await response.json();
    setData(data);
  } catch (err) {
    setError(err.message);
  } finally {
    setLoading(false);
  }
};

useEffect(() => {
  fetchData();
}, []);
```

---

## 11. 🔍 Best Practices

- **Component Organization**:
  - Keep components small & focused
  - Create container/presentation component separation
  - Use folder structures that scale (by feature or type)

- **Performance Optimization**:
  - Memoize components with `React.memo`
  - Use `useMemo` for expensive calculations
  - Apply `useCallback` for event handlers passed to children
  - Implement virtualization for long lists
  - Add proper keys to lists

- **State Management**:
  - Use local state for component-specific data
  - Apply Context for shared state across component tree
  - Consider Redux for complex global state
  - Avoid prop drilling

- **Code Quality**:
  - Implement PropTypes or TypeScript for type checking
  - Write unit tests for components and hooks
  - Follow naming conventions consistently
  - Use ESLint and Prettier for code formatting

- **Hooks Best Practices**:
  - Always cleanup effects that create subscriptions
  - Don't call hooks conditionally
  - Keep dependency arrays accurate
  - Extract complex logic to custom hooks

---

## 12. 🧊 State Persistence: `localStorage`

```jsx
// Load from localStorage on mount
useEffect(() => {
  const stored = localStorage.getItem("todos");
  if (stored) setTodos(JSON.parse(stored));
}, []);

// Save to localStorage when todos change
useEffect(() => {
  localStorage.setItem("todos", JSON.stringify(todos));
}, [todos]);
```

### Custom Hook for localStorage

```jsx
function useLocalStorage(key, initialValue) {
  // State to store our value
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  // Return a wrapped version of useState's setter function
  const setValue = (value) => {
    try {
      // Allow value to be a function
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue];
}

// Usage
const [todos, setTodos] = useLocalStorage('todos', []);
```

---

## 13. 🚀 Deployment

### GitHub Pages

```json
"homepage": "https://username.github.io/repo-name",
"scripts": {
  "predeploy": "npm run build",
  "deploy": "gh-pages -d dist"
}
```

### Vite Config for GitHub Pages

```js
export default defineConfig({
  base: '/repo-name/',
  plugins: [react()]
})
```

### Environment Variables

```
// .env.development
VITE_API_URL=http://localhost:8080/api

// .env.production
VITE_API_URL=https://api.production.com
```

Accessing in code:
```jsx
const apiUrl = import.meta.env.VITE_API_URL;
```

---

## 14. 🧠 Advanced Hooks & Concepts

### 🔹 useRef Hook

**Purpose:** Access or persist values across renders without triggering re-renders.

```jsx
const inputRef = useRef();

const focusInput = () => {
  inputRef.current.focus();
};

return <input ref={inputRef} type="text" />;
```

**🚀 Use Cases:** 
- DOM manipulation
- Storing previous values 
- Persisting mutable values between renders

### 🔹 useMemo

**Purpose:** Memoize expensive calculations to avoid recalculations on every render.

```jsx
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(num);
}, [num]);
```

**🚀 Use Case:** Optimizing performance by avoiding heavy calculations on each render

### 🔹 useCallback

**Purpose:** Memoize callback functions to prevent unnecessary re-renders in child components.

```jsx
const handleClick = useCallback(() => {
  console.log("Clicked!");
  doSomethingWith(value);
}, [value]);
```

**🚀 Use Case:** Performance optimization when passing functions as props to memoized child components

### 🔹 Error Boundaries

**Purpose:** Catch JavaScript errors in child component tree and display fallback UI.

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    console.error("Error:", error, "Info:", info);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong!</h1>;
    }

    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <ComponentThatMayError />
</ErrorBoundary>
```

**🚀 Use Case:** Production apps for crash-safe rendering

### 🔹 Lazy Loading with React.lazy and Suspense

**Purpose:** Code-splitting to improve initial load performance.

```jsx
const LazyComponent = React.lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  );
}
```

**🚀 Use Case:** Reducing initial bundle size by loading components on demand

### 🔹 React.memo

**Purpose:** Prevent unnecessary re-renders by memoizing the component.

```jsx
const MyComponent = React.memo(function MyComponent({ value }) {
  return <p>{value}</p>;
});
```

**🚀 Use Case:** Performance optimization for components that render often with the same props

### 🔹 PropTypes for Type Checking

```jsx
import PropTypes from 'prop-types';

function UserCard({ name, age }) {
  return <p>{name} is {age} years old</p>;
}

UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
};
```

**Alternative:** Use TypeScript for static type-checking

### 🔹 useReducer for Complex State Logic

```jsx
const initialState = { count: 0 };

const reducer = (state, action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return initialState;
    default:
      return state;
  }
};

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </>
  );
}
```

**🚀 Use Case:** Better for managing complex state with multiple sub-values or when next state depends on previous state

---

## 15. 🔄 React Lifecycle and Component Updates

### Functional Component Lifecycle with Hooks

```jsx
function LifecycleComponent({ id }) {
  // 1. Component Mount (constructor + componentDidMount equivalent)
  useEffect(() => {
    console.log('Component mounted');
    
    // 3. Component Unmount (componentWillUnmount equivalent)
    return () => {
      console.log('Component will unmount');
    };
  }, []);
  
  // 2. Component Update (componentDidUpdate equivalent)
  useEffect(() => {
    if (id) {
      console.log('id changed to:', id);
    }
  }, [id]);
  
  return <div>Component with id: {id}</div>;
}
```

### Component Re-rendering

Understanding when components re-render is crucial:

1. **State changes**: When a component's state changes via setState/useState updater
2. **Props changes**: When a parent re-renders or passes new props
3. **Context changes**: When a context value used by the component changes
4. **Parent re-renders**: Even if props don't change, children re-render when parents do

### Preventing Unnecessary Re-renders

```jsx
// Using React.memo for functional components
const MemoizedComponent = React.memo(
  function MyComponent(props) {
    /* render using props */
  },
  // Optional custom comparison function
  (prevProps, nextProps) => {
    // Return true if you want to skip re-render
    return prevProps.id === nextProps.id;
  }
);

// Using shouldComponentUpdate for class components
class OptimizedComponent extends React.Component {
  shouldComponentUpdate(nextProps, nextState) {
    // Only re-render if id changes
    return this.props.id !== nextProps.id;
  }
  
  render() {
    return <div>{this.props.id}</div>;
  }
}
```

---

## 16. 🧪 Testing React Components

### Setting Up Jest and React Testing Library

```jsx
// Component to test
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p data-testid="count">{count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

### Basic Component Test

```jsx
import { render, screen, fireEvent } from '@testing-library/react';
import Counter from './Counter';

test('counter increments when button is clicked', () => {
  // Render the component
  render(<Counter />);
  
  // Get the elements
  const countElement = screen.getByTestId('count');
  const incrementButton = screen.getByText('Increment');
  
  // Assert initial state
  expect(countElement.textContent).toBe('0');
  
  // Interact with the component
  fireEvent.click(incrementButton);
  
  // Assert the result
  expect(countElement.textContent).toBe('1');
});
```

### Testing Hooks

```jsx
import { renderHook, act } from '@testing-library/react-hooks';
import useCounter from './useCounter';

test('should increment counter', () => {
  const { result } = renderHook(() => useCounter());
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});
```

### Mock API Calls

```jsx
import { render, screen, waitFor } from '@testing-library/react';
import UserProfile from './UserProfile';
import * as api from './api';

// Mock the API module
jest.mock('./api');

test('displays user data when fetched successfully', async () => {
  // Setup the mock return value
  api.fetchUser.mockResolvedValueOnce({ name: 'John Doe', email: 'john@example.com' });
  
  render(<UserProfile userId="123" />);
  
  // Assert loading state
  expect(screen.getByText('Loading...')).toBeInTheDocument();
  
  // Wait for the API call to resolve
  await waitFor(() => {
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });
  
  // Verify the API was called correctly
  expect(api.fetchUser).toHaveBeenCalledWith('123');
});
```

---

## 17. 📱 Responsive React Applications

### Media Queries with CSS-in-JS

```jsx
import styled from 'styled-components';

const ResponsiveContainer = styled.div`
  padding: 10px;
  
  @media (min-width: 768px) {
    padding: 20px;
    display: flex;
  }
  
  @media (min-width: 1024px) {
    padding: 30px;
    max-width: 1200px;
    margin: 0 auto;
  }
`;
```

### Responsive Hooks

```jsx
function useMediaQuery(query) {
  const [matches, setMatches] = useState(false);
  
  useEffect(() => {
    const media = window.matchMedia(query);
    
    // Set initial value
    setMatches(media.matches);
    
    // Define callback
    const listener = () => setMatches(media.matches);
    
    // Register listener
    media.addEventListener('change', listener);
    
    // Cleanup
    return () => media.removeEventListener('change', listener);
  }, [query]);
  
  return matches;
}

// Usage
function ResponsiveComponent() {
  const isMobile = useMediaQuery('(max-width: 767px)');
  const isTablet = useMediaQuery('(min-width: 768px) and (max-width: 1023px)');
  const isDesktop = useMediaQuery('(min-width: 1024px)');
  
  return (
    <div>
      {isMobile && <MobileView />}
      {isTablet && <TabletView />}
      {isDesktop && <DesktopView />}
    </div>
  );
}
```

### Responsive with Tailwind

```jsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <div className="p-4 bg-white shadow">Item 1</div>
  <div className="p-4 bg-white shadow">Item 2</div>
  <div className="p-4 bg-white shadow">Item 3</div>
</div>
```

---

## 18. 🔐 Authentication in React

### JWT Token Authentication

```jsx
// Authentication context
const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  // Check if user is logged in
  useEffect(() => {
    async function loadUserFromToken() {
      const token = localStorage.getItem('token');
      
      if (token) {
        try {
          // Validate token with backend or decrypt JWT
          const userData = await validateToken(token);
          setUser(userData);
        } catch (err) {
          // Token invalid or expired
          localStorage.removeItem('token');
        }
      }
      
      setLoading(false);
    }
    
    loadUserFromToken();
  }, []);
  
  // Login function
  const login = async (credentials) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials)
    });
    
    if (!response.ok) {
      throw new Error('Login failed');
    }
    
    const { token, user } = await response.json();
    
    // Store token in localStorage
    localStorage.setItem('token', token);
    setUser(user);
    
    return user;
  };
  
  // Logout function
  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };
  
  return (
    <AuthContext.Provider value={{ user, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
}

// Custom hook for using auth
export function useAuth() {
  return useContext(AuthContext);
}
```

### Protected Routes

```jsx
function ProtectedRoute({ children }) {
  const { user, loading } = useAuth();
  const navigate = useNavigate();
  
  useEffect(() => {
    if (!loading && !user) {
      navigate('/login', { replace: true });
    }
  }, [user, loading, navigate]);
  
  if (loading) return <div>Loading...</div>;
  
  return user ? children : null;
}

// Usage in routing
<Routes>
  <Route path="/login" element={<LoginPage />} />
  <Route path="/dashboard" element={
    <ProtectedRoute>
      <Dashboard />
    </ProtectedRoute>
  } />
</Routes>
```

---

## 19. 📊 Data Visualization in React

### Using Charts with React-ChartJS-2

```jsx
import { Bar } from 'react-chartjs-2';
import { Chart, registerables } from 'chart.js';

// Register Chart.js components
Chart.register(...registerables);

function BarChart({ data }) {
  const chartData = {
    labels: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'],
    datasets: [
      {
        label: 'Sales 2023',
        data: [12, 19, 3, 5, 2, 3],
        backgroundColor: 'rgba(53, 162, 235, 0.5)',
      },
    ],
  };

  const options = {
    responsive: true,
    plugins: {
      legend: {
        position: 'top',
      },
      title: {
        display: true,
        text: 'Monthly Sales Data',
      },
    },
  };

  return <Bar data={chartData} options={options} />;
}
```

### SVG-based Visualizations

```jsx
function ProgressBar({ value, max = 100 }) {
  const percentage = (value / max) * 100;
  
  return (
    <svg width="200" height="20">
      <rect width="200" height="20" fill="#e0e0e0" rx="10" />
      <rect 
        width={`${percentage * 2}px`} 
        height="20" 
        fill="#4CAF50" 
        rx="10"
      />
      <text
        x="100"
        y="15"
        textAnchor="middle"
        fill="#000000"
        fontSize="12px"
      >
        {`${percentage.toFixed(0)}%`}
      </text>
    </svg>
  );
}
```

---

## 20. 🎛 State Machines with XState

```jsx
import { createMachine, assign } from 'xstate';
import { useMachine } from '@xstate/react';

// Define the state machine
const toggleMachine = createMachine({
  id: 'toggle',
  initial: 'inactive',
  context: {
    count: 0
  },
  states: {
    inactive: {
      on: { TOGGLE: 'active' }
    },
    active: {
      entry: assign({ count: ctx => ctx.count + 1 }),
      on: { TOGGLE: 'inactive' }
    }
  }
});

// Use in a component
function ToggleButton() {
  const [state, send] = useMachine(toggleMachine);
  
  return (
    <div>
      <button onClick={() => send('TOGGLE')}>
        {state.value === 'inactive' ? 'Turn On' : 'Turn Off'}
      </button>
      <p>Toggled {state.context.count} times</p>
      <p>Current state: {state.value}</p>
    </div>
  );
}
```

---

## That's All, Good Luck
