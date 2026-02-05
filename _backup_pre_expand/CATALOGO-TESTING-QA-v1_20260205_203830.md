# ═══════════════════════════════════════════════════════════════════════════════
# CATALOGO TESTING & QA v1.0
# ═══════════════════════════════════════════════════════════════════════════════
#
# GUIDA COMPLETA AL TESTING E QUALITY ASSURANCE
# Unit Testing, Integration Testing, E2E Testing, Best Practices
#
# Data creazione: 2026-01-26
# Ultima modifica: 2026-01-26
#
# ═══════════════════════════════════════════════════════════════════════════════

"""
INDICE:
═══════════════════════════════════════════════════════════════════════════════
1.  Testing Fundamentals
2.  Test Environment Setup (Vitest)
3.  Unit Testing - React Components
4.  Unit Testing - Functions & Utilities
5.  Unit Testing - Custom Hooks
6.  Integration Testing - API Routes
7.  Integration Testing - Database
8.  E2E Testing - Playwright Setup
9.  E2E Testing - User Flows
10. Mocking Strategies
11. Test Data Management
12. Code Coverage
13. CI/CD Integration
14. QA Checklist Pre-Release
═══════════════════════════════════════════════════════════════════════════════
"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 1: TESTING FUNDAMENTALS
# ═══════════════════════════════════════════════════════════════════════════════

TESTING_FUNDAMENTALS = """

## 1.1 Testing Pyramid

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ TESTING PYRAMID                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                           ▲                                                 │
│                          /│\\                                               │
│                         / │ \\                                              │
│                        /  │  \\     E2E Tests                               │
│                       /   │   \\    - Slow, expensive                       │
│                      /    │    \\   - Test critical user journeys           │
│                     /     │     \\  - 5-10% of test suite                   │
│                    /──────┼──────\\                                         │
│                   /       │       \\                                        │
│                  /        │        \\   Integration Tests                   │
│                 /         │         \\  - Test component interactions       │
│                /          │          \\ - API + DB integration              │
│               /           │           \\- 20-30% of test suite              │
│              /────────────┼────────────\\                                   │
│             /             │             \\                                  │
│            /              │              \\  Unit Tests                     │
│           /               │               \\ - Fast, isolated               │
│          /                │                \\- Test single functions        │
│         /                 │                 \\- 60-70% of test suite        │
│        ▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔                               │
│                                                                             │
│ MORE ──────────────────────────────────────────────────────────────── LESS  │
│ FAST                                                                   FAST │
│ ISOLATED                                                           ISOLATED │
│ CHEAP                                                                 CHEAP │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 1.2 Test Types

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ TEST TYPES OVERVIEW                                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ UNIT TESTS                                                                  │
│ ├── Scope: Single function, component, or module                           │
│ ├── Dependencies: Mocked                                                   │
│ ├── Speed: Very fast (ms)                                                  │
│ ├── Tools: Vitest, Jest, React Testing Library                             │
│ └── Examples:                                                               │
│     ├── Pure function calculations                                         │
│     ├── Component rendering                                                │
│     └── Custom hook logic                                                  │
│                                                                             │
│ INTEGRATION TESTS                                                           │
│ ├── Scope: Multiple units working together                                 │
│ ├── Dependencies: Real or partially mocked                                 │
│ ├── Speed: Medium (100ms - 1s)                                             │
│ ├── Tools: Vitest, Supertest, MSW                                          │
│ └── Examples:                                                               │
│     ├── API route with database                                            │
│     ├── Form submission flow                                               │
│     └── Component with context providers                                   │
│                                                                             │
│ E2E TESTS                                                                   │
│ ├── Scope: Full application flow                                           │
│ ├── Dependencies: Real (browser, server, DB)                               │
│ ├── Speed: Slow (seconds - minutes)                                        │
│ ├── Tools: Playwright, Cypress                                             │
│ └── Examples:                                                               │
│     ├── User registration flow                                             │
│     ├── Checkout process                                                   │
│     └── Authentication flows                                               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 1.3 AAA Pattern (Arrange-Act-Assert)

```typescript
// Struttura standard per tutti i test
describe('calculateTotal', () => {
  it('should calculate total with tax', () => {
    // ARRANGE - Setup test data and conditions
    const items = [
      { price: 100, quantity: 2 },
      { price: 50, quantity: 1 }
    ];
    const taxRate = 0.1;

    // ACT - Execute the code under test
    const result = calculateTotal(items, taxRate);

    // ASSERT - Verify the expected outcome
    expect(result).toBe(275); // (200 + 50) * 1.1
  });
});
```

## 1.4 Test Naming Conventions

```typescript
// Pattern: "should [expected behavior] when [condition]"

// ✅ GOOD: Descriptive, behavior-focused names
describe('UserService', () => {
  describe('createUser', () => {
    it('should create a new user when valid data is provided', () => {});
    it('should throw ValidationError when email is invalid', () => {});
    it('should hash password before saving to database', () => {});
    it('should send welcome email after user creation', () => {});
  });
});

// ❌ BAD: Vague, implementation-focused names
describe('UserService', () => {
  it('test createUser', () => {}); // Vague
  it('works', () => {}); // Non descriptive
  it('should call bcrypt.hash', () => {}); // Implementation detail
});
```

## 1.5 What to Test vs What Not to Test

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ WHAT TO TEST                              WHAT NOT TO TEST                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ ✅ Business logic                         ❌ Third-party libraries          │
│ ✅ User interactions                      ❌ Framework internals            │
│ ✅ Edge cases and error handling          ❌ Simple getters/setters         │
│ ✅ Critical user journeys                 ❌ CSS styling (unless critical)  │
│ ✅ Data transformations                   ❌ Implementation details         │
│ ✅ API contracts                          ❌ Private methods directly       │
│ ✅ Accessibility (a11y)                   ❌ Constants and config files     │
│ ✅ Security-sensitive code                ❌ Generated code                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 2: TEST ENVIRONMENT SETUP (VITEST)
# ═══════════════════════════════════════════════════════════════════════════════

VITEST_SETUP = """

## 2.1 Vitest Installation

```bash
# Install Vitest and related packages
pnpm add -D vitest @vitest/ui @vitest/coverage-v8

# React Testing Library
pnpm add -D @testing-library/react @testing-library/jest-dom @testing-library/user-event

# DOM environment
pnpm add -D jsdom

# Optional: For testing hooks
pnpm add -D @testing-library/react-hooks
```

## 2.2 Vitest Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    // Environment
    environment: 'jsdom',
    
    // Globals (describe, it, expect disponibili senza import)
    globals: true,
    
    // Setup files
    setupFiles: ['./src/test/setup.ts'],
    
    // Include patterns
    include: ['src/**/*.{test,spec}.{js,mjs,cjs,ts,mts,cts,jsx,tsx}'],
    
    // Exclude patterns
    exclude: [
      'node_modules',
      'dist',
      '.next',
      'coverage',
      '**/*.e2e.{test,spec}.*' // E2E tests run separately
    ],
    
    // Coverage configuration
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      reportsDirectory: './coverage',
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.d.ts',
        '**/*.config.*',
        '**/index.ts', // Barrel files
        'src/types/',
      ],
      // Thresholds
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 80,
        statements: 80
      }
    },
    
    // Reporters
    reporters: ['default', 'html'],
    
    // Timeouts
    testTimeout: 10000,
    hookTimeout: 10000,
    
    // Watch mode exclude
    watchExclude: ['node_modules', 'dist'],
    
    // Pool options for parallel execution
    pool: 'threads',
    poolOptions: {
      threads: {
        singleThread: false,
        isolate: true
      }
    }
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@/components': path.resolve(__dirname, './src/components'),
      '@/lib': path.resolve(__dirname, './src/lib'),
      '@/hooks': path.resolve(__dirname, './src/hooks'),
      '@/test': path.resolve(__dirname, './src/test')
    }
  }
});
```

## 2.3 Test Setup File

```typescript
// src/test/setup.ts
import { expect, afterEach, vi } from 'vitest';
import { cleanup } from '@testing-library/react';
import * as matchers from '@testing-library/jest-dom/matchers';

// Extend Vitest's expect with jest-dom matchers
expect.extend(matchers);

// Cleanup after each test
afterEach(() => {
  cleanup();
  vi.clearAllMocks();
  vi.clearAllTimers();
});

// Mock IntersectionObserver
const mockIntersectionObserver = vi.fn();
mockIntersectionObserver.mockReturnValue({
  observe: () => null,
  unobserve: () => null,
  disconnect: () => null,
});
window.IntersectionObserver = mockIntersectionObserver;

// Mock ResizeObserver
window.ResizeObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
}));

// Mock matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
});

// Mock scrollTo
window.scrollTo = vi.fn();

// Mock localStorage
const localStorageMock = {
  getItem: vi.fn(),
  setItem: vi.fn(),
  removeItem: vi.fn(),
  clear: vi.fn(),
};
Object.defineProperty(window, 'localStorage', { value: localStorageMock });

// Suppress console errors in tests (optional)
// vi.spyOn(console, 'error').mockImplementation(() => {});
```

## 2.4 Package.json Scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:watch": "vitest --watch",
    "test:ui": "vitest --ui",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:ci": "vitest run --coverage --reporter=junit --outputFile=test-results.xml"
  }
}
```

## 2.5 TypeScript Configuration for Tests

```json
// tsconfig.json - Aggiungi types per Vitest
{
  "compilerOptions": {
    "types": ["vitest/globals", "@testing-library/jest-dom"]
  }
}
```

```typescript
// src/test/types.d.ts
/// <reference types="vitest/globals" />
/// <reference types="@testing-library/jest-dom" />

import type { TestingLibraryMatchers } from '@testing-library/jest-dom/matchers';

declare module 'vitest' {
  interface Assertion<T = any> extends TestingLibraryMatchers<T, void> {}
  interface AsymmetricMatchersContaining extends TestingLibraryMatchers<any, void> {}
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 3: UNIT TESTING - REACT COMPONENTS
# ═══════════════════════════════════════════════════════════════════════════════

UNIT_TESTING_COMPONENTS = """

## 3.1 Component Testing Basics

```typescript
// components/Button/Button.tsx
interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'danger';
  disabled?: boolean;
  loading?: boolean;
  onClick?: () => void;
}

export function Button({
  children,
  variant = 'primary',
  disabled = false,
  loading = false,
  onClick
}: ButtonProps) {
  return (
    <button
      className={`btn btn-${variant}`}
      disabled={disabled || loading}
      onClick={onClick}
      aria-busy={loading}
    >
      {loading ? <Spinner /> : children}
    </button>
  );
}
```

```typescript
// components/Button/Button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './Button';

describe('Button', () => {
  // Test rendering
  it('should render children correctly', () => {
    render(<Button>Click me</Button>);
    
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });

  // Test variants
  it('should apply correct variant class', () => {
    render(<Button variant="danger">Delete</Button>);
    
    expect(screen.getByRole('button')).toHaveClass('btn-danger');
  });

  // Test disabled state
  it('should be disabled when disabled prop is true', () => {
    render(<Button disabled>Submit</Button>);
    
    expect(screen.getByRole('button')).toBeDisabled();
  });

  // Test loading state
  it('should show spinner and be disabled when loading', () => {
    render(<Button loading>Submit</Button>);
    
    const button = screen.getByRole('button');
    expect(button).toBeDisabled();
    expect(button).toHaveAttribute('aria-busy', 'true');
    expect(screen.queryByText('Submit')).not.toBeInTheDocument();
  });

  // Test click handler
  it('should call onClick when clicked', async () => {
    const user = userEvent.setup();
    const handleClick = vi.fn();
    
    render(<Button onClick={handleClick}>Click me</Button>);
    
    await user.click(screen.getByRole('button'));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  // Test disabled click
  it('should not call onClick when disabled', async () => {
    const user = userEvent.setup();
    const handleClick = vi.fn();
    
    render(<Button disabled onClick={handleClick}>Click me</Button>);
    
    await user.click(screen.getByRole('button'));
    
    expect(handleClick).not.toHaveBeenCalled();
  });
});
```

## 3.2 Query Priority (React Testing Library)

```typescript
// Priority order for queries (most to least preferred)
// https://testing-library.com/docs/queries/about#priority

/*
┌─────────────────────────────────────────────────────────────────────────────┐
│ QUERY PRIORITY                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ 1. ACCESSIBLE QUERIES (preferred - how users find elements)                │
│    ├── getByRole        → buttons, links, headings, etc.                   │
│    ├── getByLabelText   → form fields with labels                          │
│    ├── getByPlaceholderText → inputs with placeholder                      │
│    ├── getByText        → non-interactive elements                         │
│    └── getByDisplayValue → filled form elements                            │
│                                                                             │
│ 2. SEMANTIC QUERIES                                                        │
│    ├── getByAltText     → images                                           │
│    └── getByTitle       → title attribute                                  │
│                                                                             │
│ 3. TEST IDS (escape hatch - use when others don't work)                    │
│    └── getByTestId      → data-testid attribute                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
*/

// ✅ PREFERRED: Use accessible queries
const submitButton = screen.getByRole('button', { name: /submit/i });
const emailInput = screen.getByLabelText(/email/i);
const heading = screen.getByRole('heading', { name: /welcome/i, level: 1 });
const link = screen.getByRole('link', { name: /learn more/i });

// ⚠️ ACCEPTABLE: When accessible queries don't work
const logo = screen.getByAltText('Company logo');
const helpIcon = screen.getByTitle('Help');

// 🔶 ESCAPE HATCH: Last resort
const complexWidget = screen.getByTestId('date-picker');
```

## 3.3 Query Variants (get/query/find)

```typescript
/*
┌─────────────────────────────────────────────────────────────────────────────┐
│ QUERY VARIANTS                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ Prefix   │ No Match  │ 1 Match │ >1 Match │ Async? │ Use Case             │
│ ─────────┼───────────┼─────────┼──────────┼────────┼───────────────────── │
│ getBy    │ Throw     │ Return  │ Throw    │ No     │ Element SHOULD exist │
│ queryBy  │ null      │ Return  │ Throw    │ No     │ Element may NOT exist│
│ findBy   │ Throw     │ Return  │ Throw    │ Yes    │ Element will APPEAR  │
│ getAllBy │ Throw     │ Array   │ Array    │ No     │ Multiple elements    │
│ queryAllBy│ []       │ Array   │ Array    │ No     │ Zero or more elements│
│ findAllBy│ Throw     │ Array   │ Array    │ Yes    │ Multiple async       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
*/

// Use getBy when element should be present
const button = screen.getByRole('button');

// Use queryBy to test element is NOT present
expect(screen.queryByText('Error')).not.toBeInTheDocument();

// Use findBy for async elements (waits up to 1s by default)
const userData = await screen.findByText('John Doe');

// Multiple elements
const listItems = screen.getAllByRole('listitem');
expect(listItems).toHaveLength(5);
```

## 3.4 User Interactions with userEvent

```typescript
import userEvent from '@testing-library/user-event';

describe('Form Component', () => {
  it('should handle form submission', async () => {
    // Always setup userEvent at the start
    const user = userEvent.setup();
    const onSubmit = vi.fn();
    
    render(<LoginForm onSubmit={onSubmit} />);
    
    // Type in inputs
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    
    // Click submit
    await user.click(screen.getByRole('button', { name: /sign in/i }));
    
    expect(onSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123'
    });
  });

  it('should handle keyboard navigation', async () => {
    const user = userEvent.setup();
    
    render(<Form />);
    
    // Tab through form elements
    await user.tab();
    expect(screen.getByLabelText(/name/i)).toHaveFocus();
    
    await user.tab();
    expect(screen.getByLabelText(/email/i)).toHaveFocus();
    
    // Press Enter to submit
    await user.keyboard('{Enter}');
  });

  it('should handle select dropdown', async () => {
    const user = userEvent.setup();
    
    render(<CountrySelect />);
    
    await user.selectOptions(
      screen.getByRole('combobox'),
      screen.getByRole('option', { name: 'Italy' })
    );
    
    expect(screen.getByRole('combobox')).toHaveValue('IT');
  });

  it('should handle checkbox', async () => {
    const user = userEvent.setup();
    
    render(<TermsCheckbox />);
    
    const checkbox = screen.getByRole('checkbox', { name: /accept terms/i });
    expect(checkbox).not.toBeChecked();
    
    await user.click(checkbox);
    expect(checkbox).toBeChecked();
  });
});
```

## 3.5 Testing Async Components

```typescript
// components/UserProfile.tsx
export function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function fetchUser() {
      try {
        const data = await api.getUser(userId);
        setUser(data);
      } catch (err) {
        setError('Failed to load user');
      } finally {
        setLoading(false);
      }
    }
    fetchUser();
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div role="alert">{error}</div>;
  return <div>Hello, {user?.name}</div>;
}
```

```typescript
// components/UserProfile.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { UserProfile } from './UserProfile';
import { api } from '@/lib/api';

// Mock the API module
vi.mock('@/lib/api');

describe('UserProfile', () => {
  const mockUser = { id: '1', name: 'John Doe', email: 'john@example.com' };

  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should show loading state initially', () => {
    vi.mocked(api.getUser).mockImplementation(() => new Promise(() => {})); // Never resolves
    
    render(<UserProfile userId="1" />);
    
    expect(screen.getByText(/loading/i)).toBeInTheDocument();
  });

  it('should display user data after loading', async () => {
    vi.mocked(api.getUser).mockResolvedValue(mockUser);
    
    render(<UserProfile userId="1" />);
    
    // Wait for user name to appear
    await waitFor(() => {
      expect(screen.getByText(/john doe/i)).toBeInTheDocument();
    });
    
    // Loading should be gone
    expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
  });

  it('should display error message on fetch failure', async () => {
    vi.mocked(api.getUser).mockRejectedValue(new Error('Network error'));
    
    render(<UserProfile userId="1" />);
    
    // Use findBy for async elements
    const alert = await screen.findByRole('alert');
    expect(alert).toHaveTextContent(/failed to load user/i);
  });

  it('should fetch new user when userId changes', async () => {
    vi.mocked(api.getUser).mockResolvedValue(mockUser);
    
    const { rerender } = render(<UserProfile userId="1" />);
    await screen.findByText(/john doe/i);
    
    const newUser = { id: '2', name: 'Jane Doe', email: 'jane@example.com' };
    vi.mocked(api.getUser).mockResolvedValue(newUser);
    
    rerender(<UserProfile userId="2" />);
    
    await waitFor(() => {
      expect(screen.getByText(/jane doe/i)).toBeInTheDocument();
    });
    
    expect(api.getUser).toHaveBeenCalledTimes(2);
  });
});
```

## 3.6 Testing Components with Context

```typescript
// test/test-utils.tsx
import { render, RenderOptions } from '@testing-library/react';
import { ReactElement } from 'react';
import { ThemeProvider } from '@/contexts/ThemeContext';
import { AuthProvider } from '@/contexts/AuthContext';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

// Custom render with all providers
interface WrapperProps {
  children: React.ReactNode;
}

function AllTheProviders({ children }: WrapperProps) {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
        gcTime: 0,
      },
    },
  });

  return (
    <QueryClientProvider client={queryClient}>
      <AuthProvider>
        <ThemeProvider>
          {children}
        </ThemeProvider>
      </AuthProvider>
    </QueryClientProvider>
  );
}

// Custom render that includes providers
function customRender(
  ui: ReactElement,
  options?: Omit<RenderOptions, 'wrapper'>
) {
  return render(ui, { wrapper: AllTheProviders, ...options });
}

// Re-export everything from testing-library
export * from '@testing-library/react';
export { customRender as render };
```

```typescript
// Usage in tests
import { render, screen } from '@/test/test-utils';
import { Dashboard } from './Dashboard';

describe('Dashboard', () => {
  it('should render with all providers', () => {
    // Uses custom render with providers
    render(<Dashboard />);
    
    expect(screen.getByRole('main')).toBeInTheDocument();
  });
});
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 4: UNIT TESTING - FUNCTIONS & UTILITIES
# ═══════════════════════════════════════════════════════════════════════════════

UNIT_TESTING_FUNCTIONS = """

## 4.1 Testing Pure Functions

```typescript
// lib/utils/formatters.ts
export function formatCurrency(
  amount: number, 
  currency: string = 'EUR',
  locale: string = 'it-IT'
): string {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency
  }).format(amount);
}

export function formatDate(
  date: Date | string,
  format: 'short' | 'long' | 'relative' = 'short'
): string {
  const d = typeof date === 'string' ? new Date(date) : date;
  
  switch (format) {
    case 'short':
      return d.toLocaleDateString('it-IT');
    case 'long':
      return d.toLocaleDateString('it-IT', { 
        weekday: 'long', 
        year: 'numeric', 
        month: 'long', 
        day: 'numeric' 
      });
    case 'relative':
      return getRelativeTime(d);
    default:
      return d.toLocaleDateString('it-IT');
  }
}

export function slugify(text: string): string {
  return text
    .toLowerCase()
    .normalize('NFD')
    .replace(/[\u0300-\u036f]/g, '')
    .replace(/[^a-z0-9]+/g, '-')
    .replace(/^-|-$/g, '');
}
```

```typescript
// lib/utils/formatters.test.ts
import { describe, it, expect } from 'vitest';
import { formatCurrency, formatDate, slugify } from './formatters';

describe('formatCurrency', () => {
  it('should format EUR correctly in Italian locale', () => {
    expect(formatCurrency(1234.56)).toBe('1.234,56 €');
  });

  it('should format USD correctly', () => {
    expect(formatCurrency(1234.56, 'USD', 'en-US')).toBe('$1,234.56');
  });

  it('should handle zero', () => {
    expect(formatCurrency(0)).toBe('0,00 €');
  });

  it('should handle negative numbers', () => {
    expect(formatCurrency(-99.99)).toBe('-99,99 €');
  });

  it('should round to 2 decimal places', () => {
    expect(formatCurrency(10.999)).toBe('11,00 €');
  });
});

describe('formatDate', () => {
  const testDate = new Date('2025-03-15T10:30:00');

  it('should format date in short format', () => {
    expect(formatDate(testDate, 'short')).toBe('15/03/2025');
  });

  it('should format date in long format', () => {
    expect(formatDate(testDate, 'long')).toContain('15');
    expect(formatDate(testDate, 'long')).toContain('marzo');
    expect(formatDate(testDate, 'long')).toContain('2025');
  });

  it('should accept ISO string input', () => {
    expect(formatDate('2025-03-15', 'short')).toBe('15/03/2025');
  });

  it('should use short format by default', () => {
    expect(formatDate(testDate)).toBe(formatDate(testDate, 'short'));
  });
});

describe('slugify', () => {
  it('should convert to lowercase', () => {
    expect(slugify('Hello World')).toBe('hello-world');
  });

  it('should replace spaces with hyphens', () => {
    expect(slugify('hello world')).toBe('hello-world');
  });

  it('should remove special characters', () => {
    expect(slugify('Hello! World?')).toBe('hello-world');
  });

  it('should handle accented characters', () => {
    expect(slugify('Caffè Latte')).toBe('caffe-latte');
  });

  it('should handle multiple spaces', () => {
    expect(slugify('hello   world')).toBe('hello-world');
  });

  it('should trim leading and trailing hyphens', () => {
    expect(slugify('  hello world  ')).toBe('hello-world');
  });

  it('should handle empty string', () => {
    expect(slugify('')).toBe('');
  });
});
```

## 4.2 Testing Functions with Dependencies

```typescript
// lib/services/priceCalculator.ts
interface PriceConfig {
  taxRate: number;
  discountThreshold: number;
  discountRate: number;
}

interface CartItem {
  price: number;
  quantity: number;
}

export function calculateSubtotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

export function calculateDiscount(
  subtotal: number,
  config: PriceConfig
): number {
  if (subtotal >= config.discountThreshold) {
    return subtotal * config.discountRate;
  }
  return 0;
}

export function calculateTax(
  amount: number,
  config: PriceConfig
): number {
  return amount * config.taxRate;
}

export function calculateTotal(
  items: CartItem[],
  config: PriceConfig
): {
  subtotal: number;
  discount: number;
  tax: number;
  total: number;
} {
  const subtotal = calculateSubtotal(items);
  const discount = calculateDiscount(subtotal, config);
  const taxableAmount = subtotal - discount;
  const tax = calculateTax(taxableAmount, config);
  const total = taxableAmount + tax;

  return { subtotal, discount, tax, total };
}
```

```typescript
// lib/services/priceCalculator.test.ts
import { describe, it, expect } from 'vitest';
import {
  calculateSubtotal,
  calculateDiscount,
  calculateTax,
  calculateTotal
} from './priceCalculator';

describe('PriceCalculator', () => {
  const defaultConfig = {
    taxRate: 0.22, // 22% IVA
    discountThreshold: 100,
    discountRate: 0.1 // 10% discount
  };

  describe('calculateSubtotal', () => {
    it('should calculate subtotal for single item', () => {
      const items = [{ price: 50, quantity: 2 }];
      expect(calculateSubtotal(items)).toBe(100);
    });

    it('should calculate subtotal for multiple items', () => {
      const items = [
        { price: 10, quantity: 3 },
        { price: 25, quantity: 2 }
      ];
      expect(calculateSubtotal(items)).toBe(80);
    });

    it('should return 0 for empty cart', () => {
      expect(calculateSubtotal([])).toBe(0);
    });
  });

  describe('calculateDiscount', () => {
    it('should apply discount when subtotal meets threshold', () => {
      expect(calculateDiscount(100, defaultConfig)).toBe(10);
    });

    it('should apply discount when subtotal exceeds threshold', () => {
      expect(calculateDiscount(200, defaultConfig)).toBe(20);
    });

    it('should not apply discount below threshold', () => {
      expect(calculateDiscount(99, defaultConfig)).toBe(0);
    });
  });

  describe('calculateTax', () => {
    it('should calculate tax correctly', () => {
      expect(calculateTax(100, defaultConfig)).toBe(22);
    });

    it('should handle decimal amounts', () => {
      expect(calculateTax(50.5, defaultConfig)).toBeCloseTo(11.11, 2);
    });
  });

  describe('calculateTotal', () => {
    it('should calculate total without discount', () => {
      const items = [{ price: 40, quantity: 2 }]; // subtotal: 80
      const result = calculateTotal(items, defaultConfig);

      expect(result).toEqual({
        subtotal: 80,
        discount: 0,
        tax: 17.6, // 80 * 0.22
        total: 97.6
      });
    });

    it('should calculate total with discount', () => {
      const items = [{ price: 50, quantity: 3 }]; // subtotal: 150
      const result = calculateTotal(items, defaultConfig);

      expect(result.subtotal).toBe(150);
      expect(result.discount).toBe(15); // 10% of 150
      expect(result.tax).toBeCloseTo(29.7, 2); // 22% of 135
      expect(result.total).toBeCloseTo(164.7, 2);
    });

    it('should handle empty cart', () => {
      const result = calculateTotal([], defaultConfig);

      expect(result).toEqual({
        subtotal: 0,
        discount: 0,
        tax: 0,
        total: 0
      });
    });
  });
});
```

## 4.3 Testing Error Handling

```typescript
// lib/validators.ts
export class ValidationError extends Error {
  constructor(
    message: string,
    public field: string,
    public code: string
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}

export function validateEmail(email: string): string {
  if (!email) {
    throw new ValidationError('Email is required', 'email', 'REQUIRED');
  }
  
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(email)) {
    throw new ValidationError('Invalid email format', 'email', 'INVALID_FORMAT');
  }
  
  return email.toLowerCase().trim();
}

export function validatePassword(password: string): void {
  const errors: string[] = [];
  
  if (password.length < 12) {
    errors.push('Password must be at least 12 characters');
  }
  if (!/[A-Z]/.test(password)) {
    errors.push('Password must contain uppercase letter');
  }
  if (!/[a-z]/.test(password)) {
    errors.push('Password must contain lowercase letter');
  }
  if (!/[0-9]/.test(password)) {
    errors.push('Password must contain number');
  }
  
  if (errors.length > 0) {
    throw new ValidationError(
      errors.join('. '),
      'password',
      'INVALID_PASSWORD'
    );
  }
}
```

```typescript
// lib/validators.test.ts
import { describe, it, expect } from 'vitest';
import { validateEmail, validatePassword, ValidationError } from './validators';

describe('validateEmail', () => {
  it('should return normalized email for valid input', () => {
    expect(validateEmail('Test@Example.COM')).toBe('test@example.com');
  });

  it('should throw ValidationError for empty email', () => {
    expect(() => validateEmail('')).toThrow(ValidationError);
    expect(() => validateEmail('')).toThrow('Email is required');
  });

  it('should throw with correct error properties', () => {
    try {
      validateEmail('');
    } catch (error) {
      expect(error).toBeInstanceOf(ValidationError);
      expect((error as ValidationError).field).toBe('email');
      expect((error as ValidationError).code).toBe('REQUIRED');
    }
  });

  it('should throw for invalid email format', () => {
    const invalidEmails = [
      'notanemail',
      'missing@domain',
      '@nodomain.com',
      'spaces in@email.com'
    ];

    invalidEmails.forEach(email => {
      expect(() => validateEmail(email)).toThrow('Invalid email format');
    });
  });
});

describe('validatePassword', () => {
  it('should not throw for valid password', () => {
    expect(() => validatePassword('ValidPass123!')).not.toThrow();
  });

  it('should throw for password too short', () => {
    expect(() => validatePassword('Short1A')).toThrow(/at least 12 characters/);
  });

  it('should throw for missing uppercase', () => {
    expect(() => validatePassword('alllowercase123')).toThrow(/uppercase/);
  });

  it('should throw for missing lowercase', () => {
    expect(() => validatePassword('ALLUPPERCASE123')).toThrow(/lowercase/);
  });

  it('should throw for missing number', () => {
    expect(() => validatePassword('NoNumbersHere!')).toThrow(/number/);
  });

  it('should include all validation errors in message', () => {
    expect(() => validatePassword('short')).toThrow(
      expect.objectContaining({
        message: expect.stringContaining('at least 12 characters'),
        field: 'password',
        code: 'INVALID_PASSWORD'
      })
    );
  });
});
```

## 4.4 Testing Async Functions

```typescript
// lib/api/userService.ts
interface User {
  id: string;
  name: string;
  email: string;
}

export async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  
  if (!response.ok) {
    if (response.status === 404) {
      throw new Error('User not found');
    }
    throw new Error('Failed to fetch user');
  }
  
  return response.json();
}

export async function createUser(data: Omit<User, 'id'>): Promise<User> {
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || 'Failed to create user');
  }
  
  return response.json();
}
```

```typescript
// lib/api/userService.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { fetchUser, createUser } from './userService';

// Mock global fetch
const mockFetch = vi.fn();
global.fetch = mockFetch;

describe('userService', () => {
  beforeEach(() => {
    mockFetch.mockReset();
  });

  describe('fetchUser', () => {
    it('should return user data on success', async () => {
      const mockUser = { id: '1', name: 'John', email: 'john@example.com' };
      
      mockFetch.mockResolvedValueOnce({
        ok: true,
        json: async () => mockUser
      });

      const result = await fetchUser('1');
      
      expect(result).toEqual(mockUser);
      expect(mockFetch).toHaveBeenCalledWith('/api/users/1');
    });

    it('should throw "User not found" for 404', async () => {
      mockFetch.mockResolvedValueOnce({
        ok: false,
        status: 404
      });

      await expect(fetchUser('999')).rejects.toThrow('User not found');
    });

    it('should throw generic error for other failures', async () => {
      mockFetch.mockResolvedValueOnce({
        ok: false,
        status: 500
      });

      await expect(fetchUser('1')).rejects.toThrow('Failed to fetch user');
    });

    it('should handle network errors', async () => {
      mockFetch.mockRejectedValueOnce(new Error('Network error'));

      await expect(fetchUser('1')).rejects.toThrow('Network error');
    });
  });

  describe('createUser', () => {
    const newUser = { name: 'Jane', email: 'jane@example.com' };

    it('should create user and return data', async () => {
      const createdUser = { id: '2', ...newUser };
      
      mockFetch.mockResolvedValueOnce({
        ok: true,
        json: async () => createdUser
      });

      const result = await createUser(newUser);
      
      expect(result).toEqual(createdUser);
      expect(mockFetch).toHaveBeenCalledWith('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newUser)
      });
    });

    it('should throw with server error message', async () => {
      mockFetch.mockResolvedValueOnce({
        ok: false,
        json: async () => ({ message: 'Email already exists' })
      });

      await expect(createUser(newUser)).rejects.toThrow('Email already exists');
    });
  });
});
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 5: UNIT TESTING - CUSTOM HOOKS
# ═══════════════════════════════════════════════════════════════════════════════

UNIT_TESTING_HOOKS = """

## 5.1 Testing Custom Hooks with renderHook

```typescript
// hooks/useCounter.ts
import { useState, useCallback } from 'react';

export function useCounter(initialValue: number = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => setCount(c => c + 1), []);
  const decrement = useCallback(() => setCount(c => c - 1), []);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);
  const setValue = useCallback((value: number) => setCount(value), []);

  return { count, increment, decrement, reset, setValue };
}
```

```typescript
// hooks/useCounter.test.ts
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('should initialize with default value', () => {
    const { result } = renderHook(() => useCounter());
    
    expect(result.current.count).toBe(0);
  });

  it('should initialize with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    expect(result.current.count).toBe(10);
  });

  it('should increment counter', () => {
    const { result } = renderHook(() => useCounter(0));
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  it('should decrement counter', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });

  it('should reset to initial value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    act(() => {
      result.current.increment();
      result.current.increment();
    });
    expect(result.current.count).toBe(12);
    
    act(() => {
      result.current.reset();
    });
    expect(result.current.count).toBe(10);
  });

  it('should set specific value', () => {
    const { result } = renderHook(() => useCounter(0));
    
    act(() => {
      result.current.setValue(100);
    });
    
    expect(result.current.count).toBe(100);
  });

  it('should handle prop changes', () => {
    const { result, rerender } = renderHook(
      ({ initial }) => useCounter(initial),
      { initialProps: { initial: 0 } }
    );
    
    expect(result.current.count).toBe(0);
    
    // Change initial value prop
    rerender({ initial: 50 });
    
    // Count shouldn't change, only reset() uses new initial
    expect(result.current.count).toBe(0);
    
    act(() => {
      result.current.reset();
    });
    expect(result.current.count).toBe(50);
  });
});
```

## 5.2 Testing Hooks with Side Effects

```typescript
// hooks/useLocalStorage.ts
import { useState, useEffect, useCallback } from 'react';

export function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T | ((prev: T) => T)) => void, () => void] {
  // Get initial value from localStorage or use default
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  // Update localStorage when value changes
  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(storedValue));
    } catch (error) {
      console.error('Failed to save to localStorage:', error);
    }
  }, [key, storedValue]);

  // Remove from localStorage
  const remove = useCallback(() => {
    window.localStorage.removeItem(key);
    setStoredValue(initialValue);
  }, [key, initialValue]);

  return [storedValue, setStoredValue, remove];
}
```

```typescript
// hooks/useLocalStorage.test.ts
import { renderHook, act } from '@testing-library/react';
import { useLocalStorage } from './useLocalStorage';

describe('useLocalStorage', () => {
  beforeEach(() => {
    localStorage.clear();
    vi.clearAllMocks();
  });

  it('should use initial value when localStorage is empty', () => {
    const { result } = renderHook(() => 
      useLocalStorage('test-key', 'default')
    );

    expect(result.current[0]).toBe('default');
  });

  it('should use value from localStorage if exists', () => {
    localStorage.setItem('test-key', JSON.stringify('stored value'));

    const { result } = renderHook(() => 
      useLocalStorage('test-key', 'default')
    );

    expect(result.current[0]).toBe('stored value');
  });

  it('should update localStorage when value changes', () => {
    const { result } = renderHook(() => 
      useLocalStorage('test-key', 'initial')
    );

    act(() => {
      result.current[1]('new value');
    });

    expect(result.current[0]).toBe('new value');
    expect(localStorage.getItem('test-key')).toBe('"new value"');
  });

  it('should support function updates', () => {
    const { result } = renderHook(() => 
      useLocalStorage('counter', 0)
    );

    act(() => {
      result.current[1](prev => prev + 1);
    });

    expect(result.current[0]).toBe(1);
  });

  it('should remove from localStorage', () => {
    localStorage.setItem('test-key', JSON.stringify('value'));

    const { result } = renderHook(() => 
      useLocalStorage('test-key', 'default')
    );

    expect(result.current[0]).toBe('value');

    act(() => {
      result.current[2](); // remove()
    });

    expect(result.current[0]).toBe('default');
    expect(localStorage.getItem('test-key')).toBeNull();
  });

  it('should handle complex objects', () => {
    const initialValue = { user: 'John', settings: { theme: 'dark' } };

    const { result } = renderHook(() => 
      useLocalStorage('config', initialValue)
    );

    act(() => {
      result.current[1]({ 
        user: 'Jane', 
        settings: { theme: 'light' } 
      });
    });

    expect(result.current[0]).toEqual({
      user: 'Jane',
      settings: { theme: 'light' }
    });
  });
});
```

## 5.3 Testing Async Hooks

```typescript
// hooks/useFetch.ts
import { useState, useEffect, useCallback } from 'react';

interface UseFetchState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
}

interface UseFetchReturn<T> extends UseFetchState<T> {
  refetch: () => Promise<void>;
}

export function useFetch<T>(url: string): UseFetchReturn<T> {
  const [state, setState] = useState<UseFetchState<T>>({
    data: null,
    loading: true,
    error: null
  });

  const fetchData = useCallback(async () => {
    setState(prev => ({ ...prev, loading: true, error: null }));
    
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      const data = await response.json();
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState({ 
        data: null, 
        loading: false, 
        error: error instanceof Error ? error : new Error('Unknown error')
      });
    }
  }, [url]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { ...state, refetch: fetchData };
}
```

```typescript
// hooks/useFetch.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { useFetch } from './useFetch';

const mockFetch = vi.fn();
global.fetch = mockFetch;

describe('useFetch', () => {
  beforeEach(() => {
    mockFetch.mockReset();
  });

  it('should start with loading state', () => {
    mockFetch.mockImplementation(() => new Promise(() => {})); // Never resolves
    
    const { result } = renderHook(() => useFetch('/api/data'));

    expect(result.current.loading).toBe(true);
    expect(result.current.data).toBeNull();
    expect(result.current.error).toBeNull();
  });

  it('should return data on successful fetch', async () => {
    const mockData = { id: 1, name: 'Test' };
    mockFetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockData
    });

    const { result } = renderHook(() => useFetch('/api/data'));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.data).toEqual(mockData);
    expect(result.current.error).toBeNull();
  });

  it('should handle fetch errors', async () => {
    mockFetch.mockResolvedValueOnce({
      ok: false,
      status: 404
    });

    const { result } = renderHook(() => useFetch('/api/data'));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.data).toBeNull();
    expect(result.current.error).toBeInstanceOf(Error);
    expect(result.current.error?.message).toContain('404');
  });

  it('should handle network errors', async () => {
    mockFetch.mockRejectedValueOnce(new Error('Network error'));

    const { result } = renderHook(() => useFetch('/api/data'));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.error?.message).toBe('Network error');
  });

  it('should refetch data when called', async () => {
    const mockData1 = { id: 1 };
    const mockData2 = { id: 2 };

    mockFetch
      .mockResolvedValueOnce({ ok: true, json: async () => mockData1 })
      .mockResolvedValueOnce({ ok: true, json: async () => mockData2 });

    const { result } = renderHook(() => useFetch('/api/data'));

    await waitFor(() => {
      expect(result.current.data).toEqual(mockData1);
    });

    // Trigger refetch
    await result.current.refetch();

    await waitFor(() => {
      expect(result.current.data).toEqual(mockData2);
    });

    expect(mockFetch).toHaveBeenCalledTimes(2);
  });

  it('should refetch when URL changes', async () => {
    mockFetch.mockResolvedValue({
      ok: true,
      json: async () => ({ data: 'test' })
    });

    const { result, rerender } = renderHook(
      ({ url }) => useFetch(url),
      { initialProps: { url: '/api/data/1' } }
    );

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    rerender({ url: '/api/data/2' });

    await waitFor(() => {
      expect(mockFetch).toHaveBeenCalledWith('/api/data/2');
    });
  });
});
```

## 5.4 Testing Hooks with Context

```typescript
// hooks/useAuth.ts
import { useContext } from 'react';
import { AuthContext } from '@/contexts/AuthContext';

export function useAuth() {
  const context = useContext(AuthContext);
  
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  
  return context;
}
```

```typescript
// hooks/useAuth.test.tsx
import { renderHook } from '@testing-library/react';
import { AuthContext, AuthProvider } from '@/contexts/AuthContext';
import { useAuth } from './useAuth';

describe('useAuth', () => {
  it('should throw error when used outside provider', () => {
    // Suppress console.error for this test
    const consoleSpy = vi.spyOn(console, 'error').mockImplementation(() => {});
    
    expect(() => {
      renderHook(() => useAuth());
    }).toThrow('useAuth must be used within AuthProvider');
    
    consoleSpy.mockRestore();
  });

  it('should return auth context value', () => {
    const mockAuthValue = {
      user: { id: '1', name: 'John' },
      isAuthenticated: true,
      login: vi.fn(),
      logout: vi.fn()
    };

    const wrapper = ({ children }: { children: React.ReactNode }) => (
      <AuthContext.Provider value={mockAuthValue}>
        {children}
      </AuthContext.Provider>
    );

    const { result } = renderHook(() => useAuth(), { wrapper });

    expect(result.current.user).toEqual({ id: '1', name: 'John' });
    expect(result.current.isAuthenticated).toBe(true);
    expect(result.current.login).toBeDefined();
    expect(result.current.logout).toBeDefined();
  });
});
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 6: INTEGRATION TESTING - API ROUTES
# ═══════════════════════════════════════════════════════════════════════════════

INTEGRATION_TESTING_API = """

## 6.1 Testing Next.js API Routes

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';
import { z } from 'zod';

const createUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
});

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const data = createUserSchema.parse(body);

    const existingUser = await prisma.user.findUnique({
      where: { email: data.email }
    });

    if (existingUser) {
      return NextResponse.json(
        { error: 'Email already exists' },
        { status: 409 }
      );
    }

    const user = await prisma.user.create({
      data,
      select: { id: true, name: true, email: true, createdAt: true }
    });

    return NextResponse.json(user, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation failed', details: error.errors },
        { status: 400 }
      );
    }
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const page = parseInt(searchParams.get('page') || '1');
  const limit = parseInt(searchParams.get('limit') || '10');

  const users = await prisma.user.findMany({
    skip: (page - 1) * limit,
    take: limit,
    select: { id: true, name: true, email: true, createdAt: true },
    orderBy: { createdAt: 'desc' }
  });

  const total = await prisma.user.count();

  return NextResponse.json({
    data: users,
    pagination: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit)
    }
  });
}
```

```typescript
// app/api/users/route.test.ts
import { describe, it, expect, beforeEach, afterEach } from 'vitest';
import { POST, GET } from './route';
import { prisma } from '@/lib/prisma';
import { NextRequest } from 'next/server';

// Helper to create NextRequest
function createRequest(
  method: string,
  url: string,
  body?: object
): NextRequest {
  const request = new NextRequest(new URL(url, 'http://localhost:3000'), {
    method,
    headers: { 'Content-Type': 'application/json' },
    body: body ? JSON.stringify(body) : undefined
  });
  return request;
}

describe('Users API', () => {
  // Clean up test data
  beforeEach(async () => {
    await prisma.user.deleteMany({
      where: { email: { contains: '@test.com' } }
    });
  });

  afterEach(async () => {
    await prisma.user.deleteMany({
      where: { email: { contains: '@test.com' } }
    });
  });

  describe('POST /api/users', () => {
    it('should create a new user with valid data', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@test.com'
      };

      const request = createRequest('POST', '/api/users', userData);
      const response = await POST(request);
      const data = await response.json();

      expect(response.status).toBe(201);
      expect(data).toMatchObject({
        name: 'John Doe',
        email: 'john@test.com'
      });
      expect(data.id).toBeDefined();
      expect(data.createdAt).toBeDefined();
    });

    it('should return 400 for invalid email', async () => {
      const userData = {
        name: 'John Doe',
        email: 'invalid-email'
      };

      const request = createRequest('POST', '/api/users', userData);
      const response = await POST(request);
      const data = await response.json();

      expect(response.status).toBe(400);
      expect(data.error).toBe('Validation failed');
      expect(data.details).toBeDefined();
    });

    it('should return 400 for missing required fields', async () => {
      const userData = { name: 'John Doe' };

      const request = createRequest('POST', '/api/users', userData);
      const response = await POST(request);

      expect(response.status).toBe(400);
    });

    it('should return 409 for duplicate email', async () => {
      // Create first user
      await prisma.user.create({
        data: { name: 'Existing User', email: 'existing@test.com' }
      });

      // Try to create user with same email
      const request = createRequest('POST', '/api/users', {
        name: 'New User',
        email: 'existing@test.com'
      });
      const response = await POST(request);
      const data = await response.json();

      expect(response.status).toBe(409);
      expect(data.error).toBe('Email already exists');
    });
  });

  describe('GET /api/users', () => {
    beforeEach(async () => {
      // Seed test data
      await prisma.user.createMany({
        data: [
          { name: 'User 1', email: 'user1@test.com' },
          { name: 'User 2', email: 'user2@test.com' },
          { name: 'User 3', email: 'user3@test.com' },
        ]
      });
    });

    it('should return paginated users', async () => {
      const request = createRequest('GET', '/api/users?page=1&limit=2');
      const response = await GET(request);
      const data = await response.json();

      expect(response.status).toBe(200);
      expect(data.data).toHaveLength(2);
      expect(data.pagination).toMatchObject({
        page: 1,
        limit: 2,
        totalPages: expect.any(Number)
      });
    });

    it('should use default pagination', async () => {
      const request = createRequest('GET', '/api/users');
      const response = await GET(request);
      const data = await response.json();

      expect(data.pagination.page).toBe(1);
      expect(data.pagination.limit).toBe(10);
    });

    it('should return users in descending order by createdAt', async () => {
      const request = createRequest('GET', '/api/users');
      const response = await GET(request);
      const data = await response.json();

      const dates = data.data.map((u: any) => new Date(u.createdAt).getTime());
      const sortedDates = [...dates].sort((a, b) => b - a);
      
      expect(dates).toEqual(sortedDates);
    });
  });
});
```

## 6.2 Testing with Supertest (Express)

```typescript
// server/app.ts
import express from 'express';
import { userRouter } from './routes/users';

export const app = express();
app.use(express.json());
app.use('/api/users', userRouter);
```

```typescript
// server/routes/users.test.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest';
import request from 'supertest';
import { app } from '../app';
import { prisma } from '../lib/prisma';

describe('Users API (Express)', () => {
  beforeAll(async () => {
    // Setup: ensure clean database
    await prisma.$connect();
  });

  afterAll(async () => {
    await prisma.$disconnect();
  });

  beforeEach(async () => {
    await prisma.user.deleteMany({
      where: { email: { contains: '@test.com' } }
    });
  });

  describe('POST /api/users', () => {
    it('should create user and return 201', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({
          name: 'Test User',
          email: 'test@test.com'
        })
        .expect('Content-Type', /json/)
        .expect(201);

      expect(response.body).toHaveProperty('id');
      expect(response.body.name).toBe('Test User');
    });

    it('should validate request body', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({ name: '' })
        .expect(400);

      expect(response.body).toHaveProperty('error');
    });
  });

  describe('GET /api/users/:id', () => {
    it('should return user by id', async () => {
      const user = await prisma.user.create({
        data: { name: 'Get Test', email: 'get@test.com' }
      });

      const response = await request(app)
        .get(`/api/users/${user.id}`)
        .expect(200);

      expect(response.body.id).toBe(user.id);
      expect(response.body.name).toBe('Get Test');
    });

    it('should return 404 for non-existent user', async () => {
      await request(app)
        .get('/api/users/non-existent-id')
        .expect(404);
    });
  });

  describe('PUT /api/users/:id', () => {
    it('should update user', async () => {
      const user = await prisma.user.create({
        data: { name: 'Original', email: 'update@test.com' }
      });

      const response = await request(app)
        .put(`/api/users/${user.id}`)
        .send({ name: 'Updated' })
        .expect(200);

      expect(response.body.name).toBe('Updated');
    });
  });

  describe('DELETE /api/users/:id', () => {
    it('should delete user', async () => {
      const user = await prisma.user.create({
        data: { name: 'Delete Me', email: 'delete@test.com' }
      });

      await request(app)
        .delete(`/api/users/${user.id}`)
        .expect(204);

      const deleted = await prisma.user.findUnique({
        where: { id: user.id }
      });
      expect(deleted).toBeNull();
    });
  });
});
```

## 6.3 Mocking External Services with MSW

```typescript
// test/mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  // Mock external payment API
  http.post('https://api.stripe.com/v1/charges', async ({ request }) => {
    const body = await request.formData();
    const amount = body.get('amount');

    return HttpResponse.json({
      id: 'ch_mock_123',
      amount: Number(amount),
      status: 'succeeded',
      currency: 'eur'
    });
  }),

  // Mock external email service
  http.post('https://api.sendgrid.com/v3/mail/send', () => {
    return HttpResponse.json({ message: 'Email sent' }, { status: 202 });
  }),

  // Mock OAuth provider
  http.get('https://oauth.provider.com/userinfo', () => {
    return HttpResponse.json({
      sub: 'oauth-user-123',
      email: 'oauth@example.com',
      name: 'OAuth User'
    });
  }),

  // Error scenarios
  http.post('https://api.stripe.com/v1/charges', ({ request }) => {
    const url = new URL(request.url);
    if (url.searchParams.get('fail') === 'true') {
      return HttpResponse.json(
        { error: { message: 'Card declined' } },
        { status: 402 }
      );
    }
    return HttpResponse.json({ id: 'ch_123', status: 'succeeded' });
  })
];
```

```typescript
// test/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

```typescript
// test/setup.ts
import { beforeAll, afterEach, afterAll } from 'vitest';
import { server } from './mocks/server';

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

```typescript
// services/payment.test.ts
import { describe, it, expect } from 'vitest';
import { http, HttpResponse } from 'msw';
import { server } from '@/test/mocks/server';
import { processPayment } from './payment';

describe('PaymentService', () => {
  it('should process payment successfully', async () => {
    const result = await processPayment({
      amount: 1000,
      currency: 'eur',
      cardToken: 'tok_123'
    });

    expect(result.status).toBe('succeeded');
    expect(result.chargeId).toBeDefined();
  });

  it('should handle payment failure', async () => {
    // Override handler for this test
    server.use(
      http.post('https://api.stripe.com/v1/charges', () => {
        return HttpResponse.json(
          { error: { message: 'Insufficient funds' } },
          { status: 402 }
        );
      })
    );

    await expect(
      processPayment({ amount: 1000, currency: 'eur', cardToken: 'tok_bad' })
    ).rejects.toThrow('Payment failed: Insufficient funds');
  });

  it('should handle network errors', async () => {
    server.use(
      http.post('https://api.stripe.com/v1/charges', () => {
        return HttpResponse.error();
      })
    );

    await expect(
      processPayment({ amount: 1000, currency: 'eur', cardToken: 'tok_123' })
    ).rejects.toThrow('Network error');
  });
});
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 7: INTEGRATION TESTING - DATABASE
# ═══════════════════════════════════════════════════════════════════════════════

INTEGRATION_TESTING_DATABASE = """

## 7.1 Database Testing Setup

```typescript
// test/db-setup.ts
import { PrismaClient } from '@prisma/client';
import { execSync } from 'child_process';

// Use test database
const TEST_DATABASE_URL = process.env.TEST_DATABASE_URL || 
  'postgresql://postgres:postgres@localhost:5432/myapp_test';

// Create test Prisma client
export const testPrisma = new PrismaClient({
  datasources: {
    db: {
      url: TEST_DATABASE_URL
    }
  }
});

// Setup function - run before all tests
export async function setupTestDatabase() {
  // Reset database schema
  execSync('npx prisma db push --force-reset', {
    env: {
      ...process.env,
      DATABASE_URL: TEST_DATABASE_URL
    }
  });

  // Run seeds if needed
  // await seedTestData();
}

// Teardown function - run after all tests
export async function teardownTestDatabase() {
  await testPrisma.$disconnect();
}

// Clean specific tables between tests
export async function cleanDatabase() {
  // Delete in order respecting foreign keys
  await testPrisma.orderItem.deleteMany();
  await testPrisma.order.deleteMany();
  await testPrisma.product.deleteMany();
  await testPrisma.user.deleteMany();
}

// Transaction-based cleanup (faster)
export async function withCleanDatabase<T>(
  fn: (prisma: PrismaClient) => Promise<T>
): Promise<T> {
  return testPrisma.$transaction(async (tx) => {
    const result = await fn(tx as PrismaClient);
    // Rollback by throwing
    throw { __rollback: true, result };
  }).catch((e) => {
    if (e.__rollback) return e.result;
    throw e;
  });
}
```

## 7.2 Testing Repository Layer

```typescript
// repositories/userRepository.ts
import { PrismaClient, User } from '@prisma/client';

export class UserRepository {
  constructor(private prisma: PrismaClient) {}

  async findById(id: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { id } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { email } });
  }

  async create(data: { name: string; email: string; passwordHash: string }): Promise<User> {
    return this.prisma.user.create({ data });
  }

  async update(id: string, data: Partial<User>): Promise<User> {
    return this.prisma.user.update({
      where: { id },
      data
    });
  }

  async delete(id: string): Promise<void> {
    await this.prisma.user.delete({ where: { id } });
  }

  async findWithOrders(id: string) {
    return this.prisma.user.findUnique({
      where: { id },
      include: {
        orders: {
          include: { items: true }
        }
      }
    });
  }
}
```

```typescript
// repositories/userRepository.test.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest';
import { testPrisma, setupTestDatabase, teardownTestDatabase, cleanDatabase } from '@/test/db-setup';
import { UserRepository } from './userRepository';

describe('UserRepository', () => {
  let userRepo: UserRepository;

  beforeAll(async () => {
    await setupTestDatabase();
    userRepo = new UserRepository(testPrisma);
  });

  afterAll(async () => {
    await teardownTestDatabase();
  });

  beforeEach(async () => {
    await cleanDatabase();
  });

  describe('create', () => {
    it('should create a new user', async () => {
      const user = await userRepo.create({
        name: 'Test User',
        email: 'test@example.com',
        passwordHash: 'hashed_password'
      });

      expect(user.id).toBeDefined();
      expect(user.name).toBe('Test User');
      expect(user.email).toBe('test@example.com');
      expect(user.createdAt).toBeInstanceOf(Date);
    });

    it('should fail for duplicate email', async () => {
      await userRepo.create({
        name: 'First',
        email: 'same@example.com',
        passwordHash: 'hash1'
      });

      await expect(
        userRepo.create({
          name: 'Second',
          email: 'same@example.com',
          passwordHash: 'hash2'
        })
      ).rejects.toThrow(/unique constraint/i);
    });
  });

  describe('findById', () => {
    it('should find existing user', async () => {
      const created = await userRepo.create({
        name: 'Find Me',
        email: 'findme@example.com',
        passwordHash: 'hash'
      });

      const found = await userRepo.findById(created.id);

      expect(found).not.toBeNull();
      expect(found?.id).toBe(created.id);
    });

    it('should return null for non-existent user', async () => {
      const found = await userRepo.findById('non-existent-uuid');
      expect(found).toBeNull();
    });
  });

  describe('update', () => {
    it('should update user fields', async () => {
      const user = await userRepo.create({
        name: 'Original',
        email: 'update@example.com',
        passwordHash: 'hash'
      });

      const updated = await userRepo.update(user.id, { name: 'Updated' });

      expect(updated.name).toBe('Updated');
      expect(updated.email).toBe('update@example.com'); // unchanged
    });

    it('should update timestamps', async () => {
      const user = await userRepo.create({
        name: 'Time Test',
        email: 'time@example.com',
        passwordHash: 'hash'
      });

      // Wait a bit to ensure different timestamp
      await new Promise(resolve => setTimeout(resolve, 100));

      const updated = await userRepo.update(user.id, { name: 'New Name' });

      expect(updated.updatedAt.getTime()).toBeGreaterThan(user.createdAt.getTime());
    });
  });

  describe('delete', () => {
    it('should delete user', async () => {
      const user = await userRepo.create({
        name: 'Delete Me',
        email: 'delete@example.com',
        passwordHash: 'hash'
      });

      await userRepo.delete(user.id);

      const found = await userRepo.findById(user.id);
      expect(found).toBeNull();
    });
  });

  describe('findWithOrders', () => {
    it('should return user with orders and items', async () => {
      // Create user with orders
      const user = await testPrisma.user.create({
        data: {
          name: 'Order User',
          email: 'orders@example.com',
          passwordHash: 'hash',
          orders: {
            create: {
              total: 100,
              status: 'completed',
              items: {
                create: [
                  { productId: 'prod-1', quantity: 2, price: 50 }
                ]
              }
            }
          }
        }
      });

      const result = await userRepo.findWithOrders(user.id);

      expect(result?.orders).toHaveLength(1);
      expect(result?.orders[0].items).toHaveLength(1);
      expect(result?.orders[0].total).toBe(100);
    });
  });
});
```

## 7.3 Testing Complex Queries

```typescript
// repositories/orderRepository.test.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest';
import { testPrisma, setupTestDatabase, teardownTestDatabase, cleanDatabase } from '@/test/db-setup';
import { OrderRepository } from './orderRepository';

describe('OrderRepository - Complex Queries', () => {
  let orderRepo: OrderRepository;

  beforeAll(async () => {
    await setupTestDatabase();
    orderRepo = new OrderRepository(testPrisma);
  });

  afterAll(async () => {
    await teardownTestDatabase();
  });

  beforeEach(async () => {
    await cleanDatabase();
    // Seed test data
    await seedOrderTestData();
  });

  async function seedOrderTestData() {
    const user = await testPrisma.user.create({
      data: {
        name: 'Test User',
        email: 'test@example.com',
        passwordHash: 'hash'
      }
    });

    // Create orders with different dates and statuses
    await testPrisma.order.createMany({
      data: [
        { userId: user.id, total: 100, status: 'completed', createdAt: new Date('2025-01-01') },
        { userId: user.id, total: 200, status: 'completed', createdAt: new Date('2025-01-15') },
        { userId: user.id, total: 150, status: 'pending', createdAt: new Date('2025-02-01') },
        { userId: user.id, total: 300, status: 'completed', createdAt: new Date('2025-02-15') },
      ]
    });
  }

  describe('getOrderStats', () => {
    it('should calculate correct statistics', async () => {
      const stats = await orderRepo.getOrderStats({
        startDate: new Date('2025-01-01'),
        endDate: new Date('2025-12-31')
      });

      expect(stats.totalOrders).toBe(4);
      expect(stats.totalRevenue).toBe(750); // 100+200+150+300
      expect(stats.averageOrderValue).toBe(187.5);
    });

    it('should filter by date range', async () => {
      const stats = await orderRepo.getOrderStats({
        startDate: new Date('2025-01-01'),
        endDate: new Date('2025-01-31')
      });

      expect(stats.totalOrders).toBe(2);
      expect(stats.totalRevenue).toBe(300); // 100+200
    });

    it('should filter by status', async () => {
      const stats = await orderRepo.getOrderStats({
        startDate: new Date('2025-01-01'),
        endDate: new Date('2025-12-31'),
        status: 'completed'
      });

      expect(stats.totalOrders).toBe(3);
      expect(stats.totalRevenue).toBe(600); // 100+200+300
    });
  });

  describe('getTopCustomers', () => {
    it('should return customers ranked by total spent', async () => {
      // Create additional users with orders
      const user2 = await testPrisma.user.create({
        data: {
          name: 'Big Spender',
          email: 'big@example.com',
          passwordHash: 'hash',
          orders: {
            create: { total: 1000, status: 'completed' }
          }
        }
      });

      const topCustomers = await orderRepo.getTopCustomers(5);

      expect(topCustomers[0].email).toBe('big@example.com');
      expect(topCustomers[0].totalSpent).toBe(1000);
    });
  });
});
```

## 7.4 Testing Transactions

```typescript
// services/orderService.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { testPrisma, cleanDatabase } from '@/test/db-setup';
import { OrderService } from './orderService';

describe('OrderService - Transactions', () => {
  let orderService: OrderService;

  beforeEach(async () => {
    await cleanDatabase();
    orderService = new OrderService(testPrisma);

    // Seed product with inventory
    await testPrisma.product.create({
      data: {
        id: 'prod-1',
        name: 'Test Product',
        price: 100,
        inventory: 5
      }
    });

    // Seed user
    await testPrisma.user.create({
      data: {
        id: 'user-1',
        name: 'Test User',
        email: 'test@example.com',
        passwordHash: 'hash'
      }
    });
  });

  it('should create order and decrement inventory atomically', async () => {
    const order = await orderService.createOrder({
      userId: 'user-1',
      items: [{ productId: 'prod-1', quantity: 2 }]
    });

    expect(order.id).toBeDefined();
    expect(order.total).toBe(200);

    // Verify inventory was decremented
    const product = await testPrisma.product.findUnique({
      where: { id: 'prod-1' }
    });
    expect(product?.inventory).toBe(3);
  });

  it('should rollback transaction on insufficient inventory', async () => {
    await expect(
      orderService.createOrder({
        userId: 'user-1',
        items: [{ productId: 'prod-1', quantity: 10 }] // More than available
      })
    ).rejects.toThrow('Insufficient inventory');

    // Verify inventory unchanged
    const product = await testPrisma.product.findUnique({
      where: { id: 'prod-1' }
    });
    expect(product?.inventory).toBe(5);

    // Verify no order was created
    const orders = await testPrisma.order.count();
    expect(orders).toBe(0);
  });

  it('should handle concurrent orders correctly', async () => {
    // Set inventory to 3
    await testPrisma.product.update({
      where: { id: 'prod-1' },
      data: { inventory: 3 }
    });

    // Create two concurrent orders for 2 items each
    const order1Promise = orderService.createOrder({
      userId: 'user-1',
      items: [{ productId: 'prod-1', quantity: 2 }]
    });

    const order2Promise = orderService.createOrder({
      userId: 'user-1',
      items: [{ productId: 'prod-1', quantity: 2 }]
    });

    // One should succeed, one should fail
    const results = await Promise.allSettled([order1Promise, order2Promise]);
    
    const succeeded = results.filter(r => r.status === 'fulfilled');
    const failed = results.filter(r => r.status === 'rejected');

    expect(succeeded.length).toBe(1);
    expect(failed.length).toBe(1);

    // Verify final inventory
    const product = await testPrisma.product.findUnique({
      where: { id: 'prod-1' }
    });
    expect(product?.inventory).toBe(1);
  });
});
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 8: E2E TESTING - PLAYWRIGHT SETUP
# ═══════════════════════════════════════════════════════════════════════════════

E2E_PLAYWRIGHT_SETUP = """

## 8.1 Playwright Installation

```bash
# Install Playwright
pnpm add -D @playwright/test

# Install browsers
npx playwright install

# Install only specific browsers
npx playwright install chromium
npx playwright install --with-deps chromium firefox webkit
```

## 8.2 Playwright Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  // Test directory
  testDir: './e2e',
  
  // Test file pattern
  testMatch: '**/*.e2e.ts',
  
  // Timeout per test
  timeout: 30 * 1000,
  
  // Expect timeout
  expect: {
    timeout: 5000
  },
  
  // Run tests in parallel
  fullyParallel: true,
  
  // Fail the build on CI if you accidentally left test.only in the source code
  forbidOnly: !!process.env.CI,
  
  // Retry on CI only
  retries: process.env.CI ? 2 : 0,
  
  // Opt out of parallel tests on CI
  workers: process.env.CI ? 1 : undefined,
  
  // Reporter to use
  reporter: [
    ['html', { outputFolder: 'playwright-report' }],
    ['json', { outputFile: 'test-results/results.json' }],
    ['junit', { outputFile: 'test-results/junit.xml' }]
  ],
  
  // Shared settings for all projects
  use: {
    // Base URL to use in actions like `await page.goto('/')`
    baseURL: process.env.PLAYWRIGHT_BASE_URL || 'http://localhost:3000',
    
    // Collect trace when retrying the failed test
    trace: 'on-first-retry',
    
    // Screenshot on failure
    screenshot: 'only-on-failure',
    
    // Video on failure
    video: 'on-first-retry',
    
    // Headless mode
    headless: true,
    
    // Viewport size
    viewport: { width: 1280, height: 720 },
    
    // Accept downloads
    acceptDownloads: true,
  },
  
  // Configure projects for multiple browsers
  projects: [
    // Setup project - runs before all tests
    {
      name: 'setup',
      testMatch: /.*\.setup\.ts/,
    },
    
    // Desktop browsers
    {
      name: 'chromium',
      use: { 
        ...devices['Desktop Chrome'],
        storageState: '.auth/user.json'
      },
      dependencies: ['setup'],
    },
    {
      name: 'firefox',
      use: { 
        ...devices['Desktop Firefox'],
        storageState: '.auth/user.json'
      },
      dependencies: ['setup'],
    },
    {
      name: 'webkit',
      use: { 
        ...devices['Desktop Safari'],
        storageState: '.auth/user.json'
      },
      dependencies: ['setup'],
    },
    
    // Mobile browsers
    {
      name: 'mobile-chrome',
      use: { 
        ...devices['Pixel 5'],
        storageState: '.auth/user.json'
      },
      dependencies: ['setup'],
    },
    {
      name: 'mobile-safari',
      use: { 
        ...devices['iPhone 13'],
        storageState: '.auth/user.json'
      },
      dependencies: ['setup'],
    },
    
    // Unauthenticated tests
    {
      name: 'unauthenticated',
      use: { ...devices['Desktop Chrome'] },
      testMatch: '**/public/**/*.e2e.ts',
    },
  ],
  
  // Run local dev server before starting tests
  webServer: {
    command: 'pnpm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120 * 1000,
  },
});
```

## 8.3 Authentication Setup

```typescript
// e2e/auth.setup.ts
import { test as setup, expect } from '@playwright/test';

const authFile = '.auth/user.json';

setup('authenticate', async ({ page }) => {
  // Navigate to login page
  await page.goto('/login');
  
  // Fill in credentials
  await page.getByLabel('Email').fill(process.env.TEST_USER_EMAIL || 'test@example.com');
  await page.getByLabel('Password').fill(process.env.TEST_USER_PASSWORD || 'TestPassword123!');
  
  // Click login button
  await page.getByRole('button', { name: /sign in/i }).click();
  
  // Wait for redirect to dashboard
  await page.waitForURL('/dashboard');
  
  // Verify logged in
  await expect(page.getByRole('heading', { name: /dashboard/i })).toBeVisible();
  
  // Save authentication state
  await page.context().storageState({ path: authFile });
});

// Admin authentication
const adminAuthFile = '.auth/admin.json';

setup('authenticate as admin', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill(process.env.TEST_ADMIN_EMAIL!);
  await page.getByLabel('Password').fill(process.env.TEST_ADMIN_PASSWORD!);
  await page.getByRole('button', { name: /sign in/i }).click();
  await page.waitForURL('/admin');
  await page.context().storageState({ path: adminAuthFile });
});
```

## 8.4 Page Object Model

```typescript
// e2e/app/BasePage.ts
import { Page, Locator, expect } from '@playwright/test';

export abstract class BasePage {
  readonly page: Page;
  
  constructor(page: Page) {
    this.page = page;
  }
  
  // Common navigation
  async goto(path: string = '') {
    await this.page.goto(path);
  }
  
  // Wait for page load
  async waitForLoad() {
    await this.page.waitForLoadState('networkidle');
  }
  
  // Common selectors
  get header() {
    return this.page.getByRole('banner');
  }
  
  get footer() {
    return this.page.getByRole('contentinfo');
  }
  
  get mainContent() {
    return this.page.getByRole('main');
  }
  
  // Toast notifications
  async expectToast(message: string | RegExp) {
    await expect(this.page.getByRole('alert')).toContainText(message);
  }
  
  // Loading state
  async waitForLoadingToFinish() {
    await this.page.waitForSelector('[data-testid="loading"]', { state: 'hidden' });
  }
}
```

```typescript
// e2e/app/LoginPage.ts
import { Page, expect } from '@playwright/test';
import { BasePage } from './BasePage';

export class LoginPage extends BasePage {
  readonly emailInput = this.page.getByLabel('Email');
  readonly passwordInput = this.page.getByLabel('Password');
  readonly submitButton = this.page.getByRole('button', { name: /sign in/i });
  readonly errorMessage = this.page.getByRole('alert');
  readonly forgotPasswordLink = this.page.getByRole('link', { name: /forgot password/i });
  readonly registerLink = this.page.getByRole('link', { name: /create account/i });
  
  constructor(page: Page) {
    super(page);
  }
  
  async goto() {
    await super.goto('/login');
  }
  
  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }
  
  async expectError(message: string | RegExp) {
    await expect(this.errorMessage).toBeVisible();
    await expect(this.errorMessage).toContainText(message);
  }
  
  async expectRedirectToDashboard() {
    await this.page.waitForURL('/dashboard');
  }
}
```

```typescript
// e2e/app/DashboardPage.ts
import { Page, expect } from '@playwright/test';
import { BasePage } from './BasePage';

export class DashboardPage extends BasePage {
  readonly welcomeHeading = this.page.getByRole('heading', { name: /welcome/i });
  readonly statsCards = this.page.locator('[data-testid="stat-card"]');
  readonly recentActivityList = this.page.getByRole('list', { name: /recent activity/i });
  readonly userMenu = this.page.getByRole('button', { name: /user menu/i });
  readonly logoutButton = this.page.getByRole('menuitem', { name: /logout/i });
  
  constructor(page: Page) {
    super(page);
  }
  
  async goto() {
    await super.goto('/dashboard');
  }
  
  async expectLoaded() {
    await expect(this.welcomeHeading).toBeVisible();
    await expect(this.statsCards.first()).toBeVisible();
  }
  
  async logout() {
    await this.userMenu.click();
    await this.logoutButton.click();
    await this.page.waitForURL('/login');
  }
  
  async getStatValue(statName: string): Promise<string> {
    const card = this.statsCards.filter({ hasText: statName });
    const value = card.locator('[data-testid="stat-value"]');
    return await value.innerText();
  }
}
```

```typescript
// e2e/app/ProductsPage.ts
import { Page, Locator, expect } from '@playwright/test';
import { BasePage } from './BasePage';

export class ProductsPage extends BasePage {
  readonly searchInput = this.page.getByRole('searchbox');
  readonly productGrid = this.page.locator('[data-testid="product-grid"]');
  readonly productCards = this.page.locator('[data-testid="product-card"]');
  readonly sortSelect = this.page.getByRole('combobox', { name: /sort by/i });
  readonly filterPanel = this.page.locator('[data-testid="filter-panel"]');
  readonly noResultsMessage = this.page.getByText(/no products found/i);
  
  constructor(page: Page) {
    super(page);
  }
  
  async goto() {
    await super.goto('/products');
  }
  
  async search(query: string) {
    await this.searchInput.fill(query);
    await this.searchInput.press('Enter');
    await this.waitForLoadingToFinish();
  }
  
  async sortBy(option: 'price-asc' | 'price-desc' | 'name' | 'newest') {
    await this.sortSelect.selectOption(option);
    await this.waitForLoadingToFinish();
  }
  
  async filterByCategory(category: string) {
    await this.filterPanel.getByRole('checkbox', { name: category }).check();
    await this.waitForLoadingToFinish();
  }
  
  async getProductCount(): Promise<number> {
    return await this.productCards.count();
  }
  
  getProductCard(name: string): Locator {
    return this.productCards.filter({ hasText: name });
  }
  
  async addToCart(productName: string) {
    const card = this.getProductCard(productName);
    await card.getByRole('button', { name: /add to cart/i }).click();
    await this.expectToast(/added to cart/i);
  }
}
```

## 8.5 Package.json Scripts

```json
{
  "scripts": {
    "e2e": "playwright test",
    "e2e:headed": "playwright test --headed",
    "e2e:ui": "playwright test --ui",
    "e2e:debug": "playwright test --debug",
    "e2e:chromium": "playwright test --project=chromium",
    "e2e:firefox": "playwright test --project=firefox",
    "e2e:mobile": "playwright test --project=mobile-chrome",
    "e2e:report": "playwright show-report"
  }
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 9: E2E TESTING - USER FLOWS
# ═══════════════════════════════════════════════════════════════════════════════

E2E_USER_FLOWS = """

## 9.1 Authentication Flow Tests

```typescript
// e2e/auth/login.e2e.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../app/LoginPage';
import { DashboardPage } from '../app/DashboardPage';

test.describe('Login Flow', () => {
  let loginPage: LoginPage;
  
  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    await loginPage.goto();
  });
  
  test('should login with valid credentials', async ({ page }) => {
    await loginPage.login('test@example.com', 'ValidPassword123!');
    await loginPage.expectRedirectToDashboard();
    
    const dashboardPage = new DashboardPage(page);
    await dashboardPage.expectLoaded();
  });
  
  test('should show error for invalid credentials', async () => {
    await loginPage.login('test@example.com', 'WrongPassword');
    await loginPage.expectError(/invalid email or password/i);
  });
  
  test('should show validation errors for empty fields', async () => {
    await loginPage.submitButton.click();
    
    await expect(loginPage.page.getByText(/email is required/i)).toBeVisible();
    await expect(loginPage.page.getByText(/password is required/i)).toBeVisible();
  });
  
  test('should redirect authenticated users to dashboard', async ({ page }) => {
    // This test uses authenticated context from setup
    test.use({ storageState: '.auth/user.json' });
    
    await page.goto('/login');
    await expect(page).toHaveURL('/dashboard');
  });
  
  test('should handle password visibility toggle', async () => {
    await loginPage.passwordInput.fill('MyPassword123');
    
    // Initially password is hidden
    await expect(loginPage.passwordInput).toHaveAttribute('type', 'password');
    
    // Click show password button
    await loginPage.page.getByRole('button', { name: /show password/i }).click();
    await expect(loginPage.passwordInput).toHaveAttribute('type', 'text');
  });
});
```

```typescript
// e2e/auth/registration.e2e.ts
import { test, expect } from '@playwright/test';

test.describe('Registration Flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/register');
  });
  
  test('should register new user successfully', async ({ page }) => {
    const uniqueEmail = `test-${Date.now()}@example.com`;
    
    await page.getByLabel('Name').fill('New User');
    await page.getByLabel('Email').fill(uniqueEmail);
    await page.getByLabel('Password').fill('SecurePassword123!');
    await page.getByLabel('Confirm Password').fill('SecurePassword123!');
    await page.getByRole('checkbox', { name: /accept terms/i }).check();
    
    await page.getByRole('button', { name: /create account/i }).click();
    
    // Should redirect to verification or dashboard
    await expect(page).toHaveURL(/\/(verify-email|dashboard)/);
  });
  
  test('should validate password requirements', async ({ page }) => {
    await page.getByLabel('Password').fill('weak');
    await page.getByLabel('Password').blur();
    
    await expect(page.getByText(/at least 12 characters/i)).toBeVisible();
    await expect(page.getByText(/uppercase letter/i)).toBeVisible();
    await expect(page.getByText(/number/i)).toBeVisible();
  });
  
  test('should validate password confirmation match', async ({ page }) => {
    await page.getByLabel('Password').fill('SecurePassword123!');
    await page.getByLabel('Confirm Password').fill('DifferentPassword123!');
    await page.getByRole('button', { name: /create account/i }).click();
    
    await expect(page.getByText(/passwords do not match/i)).toBeVisible();
  });
  
  test('should show error for existing email', async ({ page }) => {
    await page.getByLabel('Name').fill('Existing User');
    await page.getByLabel('Email').fill('existing@example.com');
    await page.getByLabel('Password').fill('SecurePassword123!');
    await page.getByLabel('Confirm Password').fill('SecurePassword123!');
    await page.getByRole('checkbox', { name: /accept terms/i }).check();
    
    await page.getByRole('button', { name: /create account/i }).click();
    
    await expect(page.getByText(/email already registered/i)).toBeVisible();
  });
});
```

## 9.2 E-Commerce Flow Tests

```typescript
// e2e/checkout/purchase-flow.e2e.ts
import { test, expect } from '@playwright/test';
import { ProductsPage } from '../app/ProductsPage';
import { CartPage } from '../app/CartPage';
import { CheckoutPage } from '../app/CheckoutPage';

test.describe('Purchase Flow', () => {
  test('should complete a purchase successfully', async ({ page }) => {
    // 1. Browse products
    const productsPage = new ProductsPage(page);
    await productsPage.goto();
    
    // 2. Add items to cart
    await productsPage.addToCart('Premium Widget');
    await productsPage.addToCart('Basic Gadget');
    
    // 3. Go to cart
    await page.getByRole('link', { name: /cart/i }).click();
    const cartPage = new CartPage(page);
    
    // Verify cart contents
    await expect(cartPage.cartItems).toHaveCount(2);
    
    // 4. Update quantity
    await cartPage.updateQuantity('Premium Widget', 2);
    
    // 5. Proceed to checkout
    await cartPage.proceedToCheckout();
    
    const checkoutPage = new CheckoutPage(page);
    
    // 6. Fill shipping info
    await checkoutPage.fillShippingInfo({
      name: 'John Doe',
      address: '123 Main St',
      city: 'Milan',
      postalCode: '20100',
      country: 'Italy'
    });
    
    // 7. Select shipping method
    await checkoutPage.selectShipping('express');
    
    // 8. Fill payment info (use test card)
    await checkoutPage.fillPaymentInfo({
      cardNumber: '4242424242424242',
      expiry: '12/26',
      cvc: '123'
    });
    
    // 9. Review order
    await checkoutPage.reviewOrder();
    await expect(checkoutPage.orderSummary).toBeVisible();
    
    // 10. Place order
    await checkoutPage.placeOrder();
    
    // 11. Verify success
    await expect(page).toHaveURL(/\/order-confirmation/);
    await expect(page.getByText(/thank you for your order/i)).toBeVisible();
    await expect(page.getByTestId('order-number')).toBeVisible();
  });
  
  test('should handle out of stock items', async ({ page }) => {
    const productsPage = new ProductsPage(page);
    await productsPage.goto();
    
    // Product marked as out of stock
    const outOfStockProduct = productsPage.getProductCard('Sold Out Item');
    await expect(outOfStockProduct.getByText(/out of stock/i)).toBeVisible();
    await expect(outOfStockProduct.getByRole('button', { name: /add to cart/i })).toBeDisabled();
  });
  
  test('should apply discount code', async ({ page }) => {
    const cartPage = new CartPage(page);
    await page.goto('/cart');
    
    // Apply discount code
    await cartPage.applyDiscountCode('SAVE10');
    
    // Verify discount applied
    await expect(cartPage.discountLine).toContainText('-10%');
    await cartPage.expectToast(/discount applied/i);
  });
  
  test('should handle payment failure', async ({ page }) => {
    // Setup: Add item and go to checkout
    await page.goto('/cart');
    
    const checkoutPage = new CheckoutPage(page);
    await page.goto('/checkout');
    
    // Use declined test card
    await checkoutPage.fillPaymentInfo({
      cardNumber: '4000000000000002', // Decline card
      expiry: '12/26',
      cvc: '123'
    });
    
    await checkoutPage.placeOrder();
    
    // Should show error
    await expect(page.getByText(/payment declined/i)).toBeVisible();
    await expect(page).toHaveURL(/\/checkout/); // Stay on checkout
  });
});
```

## 9.3 Form Submission Tests

```typescript
// e2e/forms/contact-form.e2e.ts
import { test, expect } from '@playwright/test';

test.describe('Contact Form', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/contact');
  });
  
  test('should submit contact form successfully', async ({ page }) => {
    await page.getByLabel('Name').fill('John Doe');
    await page.getByLabel('Email').fill('john@example.com');
    await page.getByLabel('Subject').selectOption('support');
    await page.getByLabel('Message').fill('I need help with my order #12345');
    
    await page.getByRole('button', { name: /send message/i }).click();
    
    // Wait for success message
    await expect(page.getByRole('alert')).toContainText(/message sent/i);
    
    // Form should be reset
    await expect(page.getByLabel('Name')).toHaveValue('');
  });
  
  test('should validate required fields', async ({ page }) => {
    await page.getByRole('button', { name: /send message/i }).click();
    
    await expect(page.getByText(/name is required/i)).toBeVisible();
    await expect(page.getByText(/email is required/i)).toBeVisible();
    await expect(page.getByText(/message is required/i)).toBeVisible();
  });
  
  test('should handle file upload', async ({ page }) => {
    await page.getByLabel('Name').fill('John Doe');
    await page.getByLabel('Email').fill('john@example.com');
    await page.getByLabel('Message').fill('See attached screenshot');
    
    // Upload file
    const fileInput = page.getByLabel('Attachments');
    await fileInput.setInputFiles({
      name: 'screenshot.png',
      mimeType: 'image/png',
      buffer: Buffer.from('fake-image-content')
    });
    
    // Verify file is shown
    await expect(page.getByText('screenshot.png')).toBeVisible();
    
    await page.getByRole('button', { name: /send message/i }).click();
    await expect(page.getByRole('alert')).toContainText(/message sent/i);
  });
});
```

## 9.4 Accessibility Tests

```typescript
// e2e/a11y/accessibility.e2e.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Accessibility', () => {
  test('homepage should have no accessibility violations', async ({ page }) => {
    await page.goto('/');
    
    const results = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21aa'])
      .analyze();
    
    expect(results.violations).toEqual([]);
  });
  
  test('login page should have no accessibility violations', async ({ page }) => {
    await page.goto('/login');
    
    const results = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa'])
      .exclude('[data-testid="third-party-widget"]') // Exclude third-party widgets
      .analyze();
    
    expect(results.violations).toEqual([]);
  });
  
  test('should be keyboard navigable', async ({ page }) => {
    await page.goto('/');
    
    // Tab through interactive elements
    await page.keyboard.press('Tab');
    await expect(page.getByRole('link', { name: /skip to content/i })).toBeFocused();
    
    await page.keyboard.press('Tab');
    await expect(page.getByRole('link', { name: /home/i })).toBeFocused();
    
    // Press Enter to activate link
    await page.keyboard.press('Enter');
    await expect(page).toHaveURL('/');
  });
  
  test('modal should trap focus', async ({ page }) => {
    await page.goto('/products');
    
    // Open modal
    await page.getByRole('button', { name: /quick view/i }).first().click();
    
    const modal = page.getByRole('dialog');
    await expect(modal).toBeVisible();
    
    // First focusable element in modal should be focused
    const closeButton = modal.getByRole('button', { name: /close/i });
    await expect(closeButton).toBeFocused();
    
    // Tab should stay within modal
    for (let i = 0; i < 10; i++) {
      await page.keyboard.press('Tab');
    }
    
    // Focus should still be within modal
    const focusedElement = page.locator(':focus');
    await expect(focusedElement).toBeDescendantOf(modal);
    
    // Escape should close modal
    await page.keyboard.press('Escape');
    await expect(modal).not.toBeVisible();
  });
});
```

## 9.5 Visual Regression Tests

```typescript
// e2e/visual/visual-regression.e2e.ts
import { test, expect } from '@playwright/test';

test.describe('Visual Regression', () => {
  test('homepage should match snapshot', async ({ page }) => {
    await page.goto('/');
    await page.waitForLoadState('networkidle');
    
    // Hide dynamic content
    await page.evaluate(() => {
      document.querySelectorAll('[data-testid="timestamp"]').forEach(el => {
        el.textContent = 'XX:XX:XX';
      });
    });
    
    await expect(page).toHaveScreenshot('homepage.png', {
      maxDiffPixels: 100,
      animations: 'disabled'
    });
  });
  
  test('product card should match snapshot', async ({ page }) => {
    await page.goto('/products');
    
    const productCard = page.locator('[data-testid="product-card"]').first();
    
    await expect(productCard).toHaveScreenshot('product-card.png', {
      maxDiffPixelRatio: 0.02
    });
  });
  
  test('responsive layouts', async ({ page }) => {
    await page.goto('/');
    
    // Desktop
    await page.setViewportSize({ width: 1920, height: 1080 });
    await expect(page).toHaveScreenshot('homepage-desktop.png');
    
    // Tablet
    await page.setViewportSize({ width: 768, height: 1024 });
    await expect(page).toHaveScreenshot('homepage-tablet.png');
    
    // Mobile
    await page.setViewportSize({ width: 375, height: 667 });
    await expect(page).toHaveScreenshot('homepage-mobile.png');
  });
  
  test('dark mode', async ({ page }) => {
    await page.goto('/');
    
    // Enable dark mode
    await page.getByRole('button', { name: /toggle theme/i }).click();
    
    await expect(page).toHaveScreenshot('homepage-dark.png');
  });
});
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 10: MOCKING STRATEGIES
# ═══════════════════════════════════════════════════════════════════════════════

MOCKING_STRATEGIES = """

## 10.1 Mocking Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ MOCKING TYPES                                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ MOCK                                                                        │
│ ├── Fake implementation that tracks calls                                  │
│ ├── Can specify return values                                              │
│ └── vi.fn(), vi.mock()                                                     │
│                                                                             │
│ SPY                                                                         │
│ ├── Wraps real implementation                                              │
│ ├── Tracks calls while preserving behavior                                 │
│ └── vi.spyOn()                                                             │
│                                                                             │
│ STUB                                                                        │
│ ├── Replaces function with canned response                                 │
│ ├── No call tracking                                                       │
│ └── Used for simple replacements                                           │
│                                                                             │
│ FAKE                                                                        │
│ ├── Working implementation (simplified)                                    │
│ ├── In-memory database, fake API server                                    │
│ └── MSW, test containers                                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 10.2 Vitest Mocking Basics

```typescript
// Basic function mock
import { vi, describe, it, expect } from 'vitest';

describe('Mocking Basics', () => {
  it('should create a mock function', () => {
    const mockFn = vi.fn();
    
    mockFn('arg1', 'arg2');
    mockFn('arg3');
    
    expect(mockFn).toHaveBeenCalledTimes(2);
    expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
    expect(mockFn).toHaveBeenLastCalledWith('arg3');
  });
  
  it('should mock return values', () => {
    const mockFn = vi.fn()
      .mockReturnValue('default')
      .mockReturnValueOnce('first call')
      .mockReturnValueOnce('second call');
    
    expect(mockFn()).toBe('first call');
    expect(mockFn()).toBe('second call');
    expect(mockFn()).toBe('default');
  });
  
  it('should mock async functions', async () => {
    const mockAsync = vi.fn()
      .mockResolvedValue('success')
      .mockResolvedValueOnce('first')
      .mockRejectedValueOnce(new Error('Failed'));
    
    await expect(mockAsync()).rejects.toThrow('Failed');
    expect(await mockAsync()).toBe('first');
    expect(await mockAsync()).toBe('success');
  });
  
  it('should mock implementation', () => {
    const mockFn = vi.fn((x: number) => x * 2);
    
    expect(mockFn(5)).toBe(10);
    expect(mockFn).toHaveBeenCalledWith(5);
    
    // Change implementation
    mockFn.mockImplementation((x: number) => x * 3);
    expect(mockFn(5)).toBe(15);
  });
});
```

## 10.3 Module Mocking

```typescript
// lib/api.ts (module to mock)
export async function fetchUsers() {
  const response = await fetch('/api/users');
  return response.json();
}

export async function createUser(data: UserData) {
  const response = await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify(data)
  });
  return response.json();
}
```

```typescript
// __tests__/userList.test.ts
import { vi, describe, it, expect, beforeEach } from 'vitest';
import { render, screen, waitFor } from '@testing-library/react';
import { UserList } from '@/components/UserList';

// Mock the entire module
vi.mock('@/lib/api', () => ({
  fetchUsers: vi.fn(),
  createUser: vi.fn()
}));

// Import after mocking
import { fetchUsers, createUser } from '@/lib/api';

describe('UserList', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });
  
  it('should display users', async () => {
    const mockUsers = [
      { id: '1', name: 'John' },
      { id: '2', name: 'Jane' }
    ];
    
    vi.mocked(fetchUsers).mockResolvedValue(mockUsers);
    
    render(<UserList />);
    
    await waitFor(() => {
      expect(screen.getByText('John')).toBeInTheDocument();
      expect(screen.getByText('Jane')).toBeInTheDocument();
    });
    
    expect(fetchUsers).toHaveBeenCalledTimes(1);
  });
  
  it('should handle error state', async () => {
    vi.mocked(fetchUsers).mockRejectedValue(new Error('API Error'));
    
    render(<UserList />);
    
    await waitFor(() => {
      expect(screen.getByText(/error loading users/i)).toBeInTheDocument();
    });
  });
});
```

```typescript
// Partial module mock - keep some real implementations
vi.mock('@/lib/api', async () => {
  const actual = await vi.importActual('@/lib/api');
  return {
    ...actual,
    fetchUsers: vi.fn(), // Only mock this
    // createUser uses real implementation
  };
});
```

## 10.4 Spying on Methods

```typescript
import { vi, describe, it, expect, beforeEach, afterEach } from 'vitest';

describe('Spy Examples', () => {
  it('should spy on object methods', () => {
    const calculator = {
      add: (a: number, b: number) => a + b,
      multiply: (a: number, b: number) => a * b
    };
    
    const addSpy = vi.spyOn(calculator, 'add');
    
    // Real implementation still runs
    expect(calculator.add(2, 3)).toBe(5);
    expect(addSpy).toHaveBeenCalledWith(2, 3);
    
    // Can override temporarily
    addSpy.mockReturnValueOnce(100);
    expect(calculator.add(2, 3)).toBe(100);
    
    // Original behavior restored
    expect(calculator.add(2, 3)).toBe(5);
  });
  
  it('should spy on console methods', () => {
    const consoleSpy = vi.spyOn(console, 'error').mockImplementation(() => {});
    
    console.error('Test error');
    
    expect(consoleSpy).toHaveBeenCalledWith('Test error');
    
    consoleSpy.mockRestore();
  });
  
  it('should spy on Date', () => {
    const mockDate = new Date('2025-06-15T10:00:00Z');
    vi.setSystemTime(mockDate);
    
    expect(new Date().toISOString()).toBe('2025-06-15T10:00:00.000Z');
    expect(Date.now()).toBe(mockDate.getTime());
    
    vi.useRealTimers();
  });
});
```

## 10.5 Timer Mocking

```typescript
import { vi, describe, it, expect, beforeEach, afterEach } from 'vitest';

describe('Timer Mocking', () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });
  
  afterEach(() => {
    vi.useRealTimers();
  });
  
  it('should mock setTimeout', () => {
    const callback = vi.fn();
    
    setTimeout(callback, 1000);
    
    expect(callback).not.toHaveBeenCalled();
    
    vi.advanceTimersByTime(1000);
    
    expect(callback).toHaveBeenCalledTimes(1);
  });
  
  it('should mock setInterval', () => {
    const callback = vi.fn();
    
    setInterval(callback, 1000);
    
    vi.advanceTimersByTime(3500);
    
    expect(callback).toHaveBeenCalledTimes(3);
  });
  
  it('should run all pending timers', () => {
    const callback1 = vi.fn();
    const callback2 = vi.fn();
    
    setTimeout(callback1, 1000);
    setTimeout(callback2, 5000);
    
    vi.runAllTimers();
    
    expect(callback1).toHaveBeenCalled();
    expect(callback2).toHaveBeenCalled();
  });
  
  it('should test debounce function', async () => {
    const mockFn = vi.fn();
    const debouncedFn = debounce(mockFn, 500);
    
    debouncedFn();
    debouncedFn();
    debouncedFn();
    
    expect(mockFn).not.toHaveBeenCalled();
    
    vi.advanceTimersByTime(500);
    
    expect(mockFn).toHaveBeenCalledTimes(1);
  });
});
```

## 10.6 Environment Variables Mocking

```typescript
import { vi, describe, it, expect, beforeEach, afterEach } from 'vitest';

describe('Environment Variables', () => {
  const originalEnv = process.env;
  
  beforeEach(() => {
    vi.resetModules();
    process.env = { ...originalEnv };
  });
  
  afterEach(() => {
    process.env = originalEnv;
  });
  
  it('should mock environment variables', async () => {
    process.env.API_URL = 'https://test-api.example.com';
    process.env.NODE_ENV = 'test';
    
    // Re-import module to pick up new env vars
    const { getApiUrl } = await import('@/lib/config');
    
    expect(getApiUrl()).toBe('https://test-api.example.com');
  });
  
  it('should test different environments', async () => {
    process.env.NODE_ENV = 'production';
    
    const { isProduction } = await import('@/lib/config');
    
    expect(isProduction()).toBe(true);
  });
});
```

## 10.7 React Context Mocking

```typescript
// Mocking context for tests
import { vi, describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { AuthContext } from '@/contexts/AuthContext';
import { ProtectedComponent } from '@/components/ProtectedComponent';

describe('Context Mocking', () => {
  const mockAuthContext = {
    user: { id: '1', name: 'Test User', email: 'test@example.com' },
    isAuthenticated: true,
    login: vi.fn(),
    logout: vi.fn(),
    isLoading: false
  };
  
  it('should render with mocked auth context', () => {
    render(
      <AuthContext.Provider value={mockAuthContext}>
        <ProtectedComponent />
      </AuthContext.Provider>
    );
    
    expect(screen.getByText(/welcome, test user/i)).toBeInTheDocument();
  });
  
  it('should render unauthenticated state', () => {
    render(
      <AuthContext.Provider value={{ ...mockAuthContext, isAuthenticated: false, user: null }}>
        <ProtectedComponent />
      </AuthContext.Provider>
    );
    
    expect(screen.getByText(/please log in/i)).toBeInTheDocument();
  });
  
  it('should handle login action', async () => {
    const loginMock = vi.fn().mockResolvedValue(undefined);
    
    render(
      <AuthContext.Provider value={{ ...mockAuthContext, isAuthenticated: false, login: loginMock }}>
        <LoginButton />
      </AuthContext.Provider>
    );
    
    await userEvent.click(screen.getByRole('button', { name: /login/i }));
    
    expect(loginMock).toHaveBeenCalled();
  });
});
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 11: TEST DATA MANAGEMENT
# ═══════════════════════════════════════════════════════════════════════════════

TEST_DATA_MANAGEMENT = """

## 11.1 Factory Functions

```typescript
// test/factories/userFactory.ts
import { faker } from '@faker-js/faker';

interface User {
  id: string;
  name: string;
  email: string;
  role: 'user' | 'admin' | 'editor';
  createdAt: Date;
  updatedAt: Date;
}

type UserOverrides = Partial<User>;

// Factory function
export function createUser(overrides: UserOverrides = {}): User {
  return {
    id: faker.string.uuid(),
    name: faker.person.fullName(),
    email: faker.internet.email().toLowerCase(),
    role: 'user',
    createdAt: faker.date.past(),
    updatedAt: new Date(),
    ...overrides
  };
}

// Create multiple users
export function createUsers(count: number, overrides: UserOverrides = {}): User[] {
  return Array.from({ length: count }, () => createUser(overrides));
}

// Specific user types
export function createAdmin(overrides: UserOverrides = {}): User {
  return createUser({ ...overrides, role: 'admin' });
}

export function createEditor(overrides: UserOverrides = {}): User {
  return createUser({ ...overrides, role: 'editor' });
}
```

```typescript
// test/factories/productFactory.ts
import { faker } from '@faker-js/faker';

interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  inventory: number;
  category: string;
  images: string[];
  isActive: boolean;
}

export function createProduct(overrides: Partial<Product> = {}): Product {
  return {
    id: faker.string.uuid(),
    name: faker.commerce.productName(),
    description: faker.commerce.productDescription(),
    price: parseFloat(faker.commerce.price({ min: 10, max: 500 })),
    inventory: faker.number.int({ min: 0, max: 100 }),
    category: faker.commerce.department(),
    images: [faker.image.url(), faker.image.url()],
    isActive: true,
    ...overrides
  };
}

// Product states
export function createOutOfStockProduct(): Product {
  return createProduct({ inventory: 0, isActive: true });
}

export function createInactiveProduct(): Product {
  return createProduct({ isActive: false });
}

export function createExpensiveProduct(): Product {
  return createProduct({ price: 9999.99 });
}
```

```typescript
// test/factories/orderFactory.ts
import { faker } from '@faker-js/faker';
import { createProduct } from './productFactory';
import { createUser } from './userFactory';

interface OrderItem {
  productId: string;
  quantity: number;
  price: number;
}

interface Order {
  id: string;
  userId: string;
  items: OrderItem[];
  subtotal: number;
  tax: number;
  total: number;
  status: 'pending' | 'processing' | 'shipped' | 'delivered' | 'cancelled';
  createdAt: Date;
}

export function createOrderItem(overrides: Partial<OrderItem> = {}): OrderItem {
  const price = parseFloat(faker.commerce.price());
  return {
    productId: faker.string.uuid(),
    quantity: faker.number.int({ min: 1, max: 5 }),
    price,
    ...overrides
  };
}

export function createOrder(overrides: Partial<Order> = {}): Order {
  const items = overrides.items || [createOrderItem(), createOrderItem()];
  const subtotal = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  const tax = subtotal * 0.22;
  
  return {
    id: faker.string.uuid(),
    userId: faker.string.uuid(),
    items,
    subtotal,
    tax,
    total: subtotal + tax,
    status: 'pending',
    createdAt: new Date(),
    ...overrides
  };
}

// Complete order scenario
export function createCompletedOrder(): Order {
  return createOrder({ status: 'delivered' });
}

export function createCancelledOrder(): Order {
  return createOrder({ status: 'cancelled' });
}
```

## 11.2 Fixtures

```typescript
// test/fixtures/users.ts
export const testUsers = {
  regularUser: {
    id: 'user-regular-1',
    name: 'Regular User',
    email: 'regular@test.com',
    role: 'user' as const
  },
  adminUser: {
    id: 'user-admin-1',
    name: 'Admin User',
    email: 'admin@test.com',
    role: 'admin' as const
  },
  editorUser: {
    id: 'user-editor-1',
    name: 'Editor User',
    email: 'editor@test.com',
    role: 'editor' as const
  }
};

// test/fixtures/products.ts
export const testProducts = {
  widget: {
    id: 'prod-widget-1',
    name: 'Premium Widget',
    price: 99.99,
    inventory: 50,
    category: 'Widgets'
  },
  gadget: {
    id: 'prod-gadget-1',
    name: 'Super Gadget',
    price: 149.99,
    inventory: 25,
    category: 'Gadgets'
  },
  outOfStock: {
    id: 'prod-oos-1',
    name: 'Sold Out Item',
    price: 29.99,
    inventory: 0,
    category: 'Misc'
  }
};
```

## 11.3 Test Builders

```typescript
// test/builders/UserBuilder.ts
import { faker } from '@faker-js/faker';

interface User {
  id: string;
  name: string;
  email: string;
  role: 'user' | 'admin' | 'editor';
  isVerified: boolean;
  preferences: {
    newsletter: boolean;
    notifications: boolean;
  };
}

export class UserBuilder {
  private user: User;
  
  constructor() {
    this.user = {
      id: faker.string.uuid(),
      name: faker.person.fullName(),
      email: faker.internet.email().toLowerCase(),
      role: 'user',
      isVerified: true,
      preferences: {
        newsletter: false,
        notifications: true
      }
    };
  }
  
  withId(id: string): this {
    this.user.id = id;
    return this;
  }
  
  withName(name: string): this {
    this.user.name = name;
    return this;
  }
  
  withEmail(email: string): this {
    this.user.email = email;
    return this;
  }
  
  asAdmin(): this {
    this.user.role = 'admin';
    return this;
  }
  
  asEditor(): this {
    this.user.role = 'editor';
    return this;
  }
  
  unverified(): this {
    this.user.isVerified = false;
    return this;
  }
  
  withNewsletter(): this {
    this.user.preferences.newsletter = true;
    return this;
  }
  
  withoutNotifications(): this {
    this.user.preferences.notifications = false;
    return this;
  }
  
  build(): User {
    return { ...this.user };
  }
}

// Usage
const adminUser = new UserBuilder()
  .withName('Admin User')
  .asAdmin()
  .withNewsletter()
  .build();

const unverifiedUser = new UserBuilder()
  .unverified()
  .withoutNotifications()
  .build();
```

## 11.4 Database Seeding

```typescript
// test/seed.ts
import { PrismaClient } from '@prisma/client';
import { createUser, createUsers } from './factories/userFactory';
import { createProduct } from './factories/productFactory';
import bcrypt from 'bcrypt';

const prisma = new PrismaClient();

export async function seedTestDatabase() {
  // Clear existing data
  await prisma.orderItem.deleteMany();
  await prisma.order.deleteMany();
  await prisma.product.deleteMany();
  await prisma.user.deleteMany();
  
  // Create admin user
  const adminPassword = await bcrypt.hash('AdminPassword123!', 10);
  await prisma.user.create({
    data: {
      id: 'admin-1',
      name: 'Test Admin',
      email: 'admin@test.com',
      passwordHash: adminPassword,
      role: 'admin',
      isVerified: true
    }
  });
  
  // Create regular test user
  const userPassword = await bcrypt.hash('UserPassword123!', 10);
  await prisma.user.create({
    data: {
      id: 'user-1',
      name: 'Test User',
      email: 'user@test.com',
      passwordHash: userPassword,
      role: 'user',
      isVerified: true
    }
  });
  
  // Create products
  const products = [
    { id: 'prod-1', name: 'Widget A', price: 99.99, inventory: 100 },
    { id: 'prod-2', name: 'Widget B', price: 149.99, inventory: 50 },
    { id: 'prod-3', name: 'Gadget C', price: 199.99, inventory: 25 },
    { id: 'prod-4', name: 'Out of Stock', price: 49.99, inventory: 0 },
  ];
  
  await prisma.product.createMany({ data: products });
  
  console.log('✅ Test database seeded');
}

export async function cleanupTestDatabase() {
  await prisma.orderItem.deleteMany();
  await prisma.order.deleteMany();
  await prisma.product.deleteMany();
  await prisma.user.deleteMany();
  
  console.log('✅ Test database cleaned');
}

// Run directly
if (require.main === module) {
  seedTestDatabase()
    .then(() => process.exit(0))
    .catch((e) => {
      console.error(e);
      process.exit(1);
    });
}
```

## 11.5 Test Data Isolation

```typescript
// test/helpers/isolation.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// Generate unique test identifiers
export function generateTestId(prefix: string = 'test'): string {
  return `${prefix}-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
}

// Create isolated test data with automatic cleanup
export async function withTestData<T>(
  setup: () => Promise<T>,
  test: (data: T) => Promise<void>
): Promise<void> {
  let data: T | undefined;
  
  try {
    data = await setup();
    await test(data);
  } finally {
    // Cleanup is handled by the test helper or transaction
  }
}

// Transaction-based isolation
export async function runInTransaction<T>(
  fn: (tx: PrismaClient) => Promise<T>
): Promise<T> {
  return prisma.$transaction(async (tx) => {
    const result = await fn(tx as PrismaClient);
    
    // Rollback by throwing
    throw { __rollback: true, result };
  }).catch((e) => {
    if (e.__rollback) return e.result;
    throw e;
  });
}

// Example usage
describe('OrderService', () => {
  it('should create order', async () => {
    await runInTransaction(async (tx) => {
      // All database operations here are rolled back after test
      const user = await tx.user.create({
        data: createUser()
      });
      
      const order = await orderService.createOrder(tx, {
        userId: user.id,
        items: [{ productId: 'prod-1', quantity: 2 }]
      });
      
      expect(order).toBeDefined();
      // Data is automatically rolled back
    });
  });
});
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 12: CODE COVERAGE
# ═══════════════════════════════════════════════════════════════════════════════

CODE_COVERAGE = """

## 12.1 Coverage Configuration

```typescript
// vitest.config.ts - Coverage section
export default defineConfig({
  test: {
    coverage: {
      // Provider: v8 (faster) or istanbul (more accurate)
      provider: 'v8',
      
      // Enable coverage
      enabled: true,
      
      // Output formats
      reporter: [
        'text',           // Console output
        'text-summary',   // Summary in console
        'html',           // HTML report
        'lcov',           // For SonarQube/CI tools
        'json',           // Machine readable
        'cobertura'       // For Azure DevOps
      ],
      
      // Output directory
      reportsDirectory: './coverage',
      
      // Files to include in coverage
      include: [
        'src/**/*.{ts,tsx}',
        '!src/**/*.d.ts',
        '!src/**/*.stories.{ts,tsx}',
      ],
      
      // Files to exclude from coverage
      exclude: [
        'node_modules/',
        'src/test/',
        'src/**/__tests__/',
        'src/**/*.test.{ts,tsx}',
        'src/**/*.spec.{ts,tsx}',
        'src/types/',
        'src/**/index.ts',       // Barrel files
        'src/**/*.config.ts',
        'src/mocks/',
      ],
      
      // Coverage thresholds - fail if below
      thresholds: {
        // Global thresholds
        lines: 80,
        functions: 80,
        branches: 75,
        statements: 80,
        
        // Per-file thresholds (optional)
        // perFile: true,
      },
      
      // Show uncovered lines in output
      all: true,
      
      // Clean coverage results before running
      clean: true,
      
      // Skip coverage for certain files if coverage drops
      skipFull: false,
    }
  }
});
```

## 12.2 Understanding Coverage Metrics

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ COVERAGE METRICS                                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ STATEMENT COVERAGE                                                          │
│ ├── Measures: Lines of code executed                                       │
│ ├── Example: if (x) { doA(); } doB();                                      │
│ │   - if x=true: 100% (both doA and doB executed)                          │
│ │   - if x=false: 66% (only doB executed)                                  │
│ └── Target: 80%+                                                           │
│                                                                             │
│ BRANCH COVERAGE                                                             │
│ ├── Measures: Decision paths taken                                         │
│ ├── Example: if (x && y) { ... }                                           │
│ │   - Need tests where: x=true/y=true, x=false, x=true/y=false            │
│ └── Target: 75%+                                                           │
│                                                                             │
│ FUNCTION COVERAGE                                                           │
│ ├── Measures: Functions that have been called                              │
│ ├── Doesn't check if function logic is fully tested                        │
│ └── Target: 80%+                                                           │
│                                                                             │
│ LINE COVERAGE                                                               │
│ ├── Measures: Executable lines run                                         │
│ ├── Similar to statement but counts lines                                  │
│ └── Target: 80%+                                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 12.3 Coverage-Driven Testing

```typescript
// Example: Identifying untested branches

// Function with multiple branches
export function calculateShipping(
  weight: number,
  destination: 'domestic' | 'international',
  expedited: boolean
): number {
  let baseRate = 0;
  
  // Branch 1: Destination
  if (destination === 'domestic') {
    baseRate = 5.99;
  } else {
    baseRate = 19.99;
  }
  
  // Branch 2: Weight tiers
  if (weight <= 1) {
    // No additional charge
  } else if (weight <= 5) {
    baseRate += 2.99;
  } else if (weight <= 20) {
    baseRate += 9.99;
  } else {
    baseRate += 24.99;
  }
  
  // Branch 3: Expedited
  if (expedited) {
    baseRate *= 1.5;
  }
  
  return Math.round(baseRate * 100) / 100;
}

// Tests for full branch coverage
describe('calculateShipping', () => {
  // Domestic tests
  describe('domestic shipping', () => {
    it('should calculate base rate for light package', () => {
      expect(calculateShipping(0.5, 'domestic', false)).toBe(5.99);
    });
    
    it('should add fee for medium package', () => {
      expect(calculateShipping(3, 'domestic', false)).toBe(8.98);
    });
    
    it('should add fee for heavy package', () => {
      expect(calculateShipping(15, 'domestic', false)).toBe(15.98);
    });
    
    it('should add fee for very heavy package', () => {
      expect(calculateShipping(25, 'domestic', false)).toBe(30.98);
    });
  });
  
  // International tests
  describe('international shipping', () => {
    it('should use international base rate', () => {
      expect(calculateShipping(0.5, 'international', false)).toBe(19.99);
    });
  });
  
  // Expedited tests
  describe('expedited shipping', () => {
    it('should multiply rate by 1.5 for expedited', () => {
      expect(calculateShipping(0.5, 'domestic', true)).toBe(8.99); // 5.99 * 1.5
    });
  });
});
```

## 12.4 Coverage Reports

```bash
# Generate coverage report
pnpm test:coverage

# View HTML report
open coverage/index.html

# Output example:
# ------------|---------|----------|---------|---------|-------------------
# File        | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
# ------------|---------|----------|---------|---------|-------------------
# All files   |   85.71 |    78.57 |   90.00 |   85.71 |
#  src/       |   85.71 |    78.57 |   90.00 |   85.71 |
#   utils.ts  |   100   |    100   |   100   |   100   |
#   api.ts    |   75.00 |    60.00 |   80.00 |   75.00 | 45-52,78
#   auth.ts   |   82.35 |    75.00 |   90.00 |   82.35 | 23-25,89
# ------------|---------|----------|---------|---------|-------------------
```

## 12.5 Istanbul Ignore Comments

```typescript
// Ignoring specific lines/blocks from coverage

// Ignore next line
/* istanbul ignore next */
if (process.env.NODE_ENV === 'development') {
  console.log('Debug mode');
}

// Ignore else branch
/* istanbul ignore else */
if (condition) {
  // This is tested
} else {
  // This edge case is hard to reproduce in tests
}

// Ignore entire function
/* istanbul ignore next */
function debugHelper() {
  // Development-only helper
}

// Ignore specific condition
if (/* istanbul ignore next */ typeof window === 'undefined') {
  // SSR-only code
}

// Note: Use sparingly! Most code should be testable.
// Valid use cases:
// - Development-only code
// - Defensive programming (unreachable branches)
// - Platform-specific code
// - Error boundaries
```

## 12.6 Coverage in CI

```yaml
# .github/workflows/test.yml
name: Test with Coverage

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install
      
      - name: Run tests with coverage
        run: pnpm test:coverage
      
      - name: Check coverage thresholds
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% is below 80% threshold"
            exit 1
          fi
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true
          token: ${{ secrets.CODECOV_TOKEN }}
      
      - name: Upload coverage artifact
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 13: CI/CD INTEGRATION
# ═══════════════════════════════════════════════════════════════════════════════

CI_CD_INTEGRATION = """

## 13.1 Complete CI/CD Testing Pipeline

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  NODE_VERSION: '20'
  PNPM_VERSION: '8'

jobs:
  # ====== LINT & TYPE CHECK ======
  lint:
    name: Lint & Type Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Run ESLint
        run: pnpm lint
      
      - name: Run TypeScript check
        run: pnpm tsc --noEmit

  # ====== UNIT & INTEGRATION TESTS ======
  test:
    name: Unit & Integration Tests
    runs-on: ubuntu-latest
    needs: lint
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test_db
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      
      redis:
        image: redis:7
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Setup database
        run: pnpm prisma db push
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test_db
      
      - name: Run tests
        run: pnpm test:ci
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379
          NODE_ENV: test
      
      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage/lcov.info
          token: ${{ secrets.CODECOV_TOKEN }}
      
      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: |
            coverage/
            test-results/

  # ====== E2E TESTS ======
  e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: test
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test_db
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Install Playwright browsers
        run: pnpm exec playwright install --with-deps chromium
      
      - name: Setup database
        run: |
          pnpm prisma db push
          pnpm db:seed:test
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test_db
      
      - name: Build application
        run: pnpm build
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test_db
      
      - name: Run E2E tests
        run: pnpm e2e:ci
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test_db
          PLAYWRIGHT_BASE_URL: http://localhost:3000
          TEST_USER_EMAIL: user@test.com
          TEST_USER_PASSWORD: TestPassword123!
      
      - name: Upload Playwright report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
      
      - name: Upload test screenshots
        uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: e2e-screenshots
          path: test-results/

  # ====== SECURITY SCAN ======
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Run npm audit
        run: pnpm audit --audit-level=high
        continue-on-error: true
      
      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        continue-on-error: true
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  # ====== REPORT STATUS ======
  report:
    name: Report Status
    runs-on: ubuntu-latest
    needs: [lint, test, e2e, security]
    if: always()
    steps:
      - name: Check job status
        run: |
          if [[ "${{ needs.lint.result }}" == "failure" ]] || \
             [[ "${{ needs.test.result }}" == "failure" ]] || \
             [[ "${{ needs.e2e.result }}" == "failure" ]]; then
            echo "One or more jobs failed"
            exit 1
          fi
```

## 13.2 Package.json CI Scripts

```json
{
  "scripts": {
    "lint": "eslint . --ext .ts,.tsx --max-warnings 0",
    "lint:fix": "eslint . --ext .ts,.tsx --fix",
    "type-check": "tsc --noEmit",
    
    "test": "vitest",
    "test:watch": "vitest --watch",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage",
    "test:ci": "vitest run --coverage --reporter=junit --outputFile=test-results/junit.xml",
    
    "e2e": "playwright test",
    "e2e:ui": "playwright test --ui",
    "e2e:headed": "playwright test --headed",
    "e2e:ci": "playwright test --project=chromium",
    "e2e:report": "playwright show-report",
    
    "db:seed:test": "tsx scripts/seed-test.ts",
    
    "ci": "pnpm lint && pnpm type-check && pnpm test:ci"
  }
}
```

## 13.3 Pre-commit Hooks

```bash
# Install husky
pnpm add -D husky lint-staged
pnpm exec husky init
```

```javascript
// .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

pnpm lint-staged
```

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "vitest related --run"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

```javascript
// .husky/pre-push
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

pnpm test:ci
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 14: QA CHECKLIST PRE-RELEASE
# ═══════════════════════════════════════════════════════════════════════════════

QA_CHECKLIST = """

## 14.1 Pre-Release QA Checklist

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ PRE-RELEASE QA CHECKLIST                                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ AUTOMATED TESTING                                                           │
│ □ All unit tests passing                                                   │
│ □ All integration tests passing                                            │
│ □ All E2E tests passing                                                    │
│ □ Code coverage meets thresholds (80%+ recommended)                        │
│ □ No skipped tests (or documented reasons for skips)                       │
│ □ No flaky tests                                                           │
│                                                                             │
│ CODE QUALITY                                                                │
│ □ No ESLint errors or warnings                                             │
│ □ No TypeScript errors                                                     │
│ □ No console.log statements in production code                             │
│ □ No TODO/FIXME comments for this release                                  │
│ □ Code review completed and approved                                       │
│                                                                             │
│ SECURITY                                                                    │
│ □ npm audit shows no high/critical vulnerabilities                         │
│ □ No secrets in code or logs                                               │
│ □ Authentication flows tested                                              │
│ □ Authorization (RBAC) tested                                              │
│ □ Input validation tested                                                  │
│ □ XSS prevention verified                                                  │
│ □ CSRF protection tested                                                   │
│                                                                             │
│ FUNCTIONALITY                                                               │
│ □ All new features tested manually                                         │
│ □ Regression testing completed                                             │
│ □ Edge cases tested                                                        │
│ □ Error handling tested                                                    │
│ □ Loading states verified                                                  │
│ □ Empty states verified                                                    │
│                                                                             │
│ PERFORMANCE                                                                 │
│ □ Lighthouse score meets targets                                           │
│ □ Core Web Vitals within acceptable range                                  │
│ □ No memory leaks detected                                                 │
│ □ API response times acceptable                                            │
│ □ Large data sets tested                                                   │
│                                                                             │
│ CROSS-BROWSER/DEVICE                                                        │
│ □ Tested on Chrome (latest)                                                │
│ □ Tested on Firefox (latest)                                               │
│ □ Tested on Safari (latest)                                                │
│ □ Tested on mobile (iOS Safari, Chrome Android)                            │
│ □ Responsive design verified                                               │
│                                                                             │
│ ACCESSIBILITY                                                               │
│ □ Axe accessibility scan passes                                            │
│ □ Keyboard navigation works                                                │
│ □ Screen reader tested                                                     │
│ □ Color contrast meets WCAG AA                                             │
│ □ Focus indicators visible                                                 │
│                                                                             │
│ DOCUMENTATION                                                               │
│ □ README updated                                                           │
│ □ API documentation updated                                                │
│ □ Changelog updated                                                        │
│ □ Release notes prepared                                                   │
│                                                                             │
│ DEPLOYMENT                                                                  │
│ □ Database migrations tested                                               │
│ □ Environment variables documented                                         │
│ □ Rollback plan prepared                                                   │
│ □ Monitoring alerts configured                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 14.2 Bug Severity Classification

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ BUG SEVERITY LEVELS                                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ CRITICAL (P0) - Release Blocker                                            │
│ ├── Application crash                                                      │
│ ├── Data loss or corruption                                                │
│ ├── Security vulnerability                                                 │
│ ├── Core functionality broken                                              │
│ └── Response: Fix immediately, no release until resolved                   │
│                                                                             │
│ HIGH (P1) - Major Issue                                                    │
│ ├── Feature completely broken                                              │
│ ├── Significant user impact                                                │
│ ├── No workaround available                                                │
│ └── Response: Fix before release if possible                               │
│                                                                             │
│ MEDIUM (P2) - Minor Issue                                                  │
│ ├── Feature partially broken                                               │
│ ├── Workaround available                                                   │
│ ├── Minor user impact                                                      │
│ └── Response: Can release, schedule fix for next sprint                    │
│                                                                             │
│ LOW (P3) - Trivial Issue                                                   │
│ ├── Cosmetic issues                                                        │
│ ├── Minor inconvenience                                                    │
│ ├── Edge case scenarios                                                    │
│ └── Response: Add to backlog                                               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 14.3 Test Case Template

```markdown
# Test Case: TC-001 - User Login

## Overview
- **Module:** Authentication
- **Priority:** High
- **Type:** Functional
- **Automated:** Yes

## Preconditions
- User account exists in system
- User is not logged in
- Application is accessible

## Test Steps

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Navigate to /login | Login page loads with email and password fields |
| 2 | Enter valid email | Email is accepted |
| 3 | Enter valid password | Password is masked |
| 4 | Click "Sign In" button | Loading indicator appears |
| 5 | Wait for response | Redirected to dashboard |

## Expected Results
- User is logged in
- Session is created
- Dashboard displays user name
- Auth token stored securely

## Test Data
- Email: test@example.com
- Password: ValidPassword123!

## Notes
- Test with remember me checked/unchecked
- Test with invalid credentials
- Test rate limiting after failed attempts
```

## 14.4 Release Sign-off

```typescript
// scripts/release-checklist.ts
interface ReleaseChecklist {
  version: string;
  date: string;
  checks: {
    category: string;
    items: {
      name: string;
      status: 'pass' | 'fail' | 'skip' | 'pending';
      notes?: string;
    }[];
  }[];
  signoffs: {
    role: string;
    name: string;
    date?: string;
    approved: boolean;
  }[];
}

const releaseChecklist: ReleaseChecklist = {
  version: '1.2.0',
  date: '2025-06-15',
  checks: [
    {
      category: 'Automated Testing',
      items: [
        { name: 'Unit tests', status: 'pass' },
        { name: 'Integration tests', status: 'pass' },
        { name: 'E2E tests', status: 'pass' },
        { name: 'Coverage > 80%', status: 'pass', notes: 'Current: 85%' }
      ]
    },
    {
      category: 'Security',
      items: [
        { name: 'npm audit', status: 'pass' },
        { name: 'Snyk scan', status: 'pass' },
        { name: 'Auth testing', status: 'pass' }
      ]
    },
    {
      category: 'Performance',
      items: [
        { name: 'Lighthouse > 90', status: 'pass', notes: 'Score: 94' },
        { name: 'Core Web Vitals', status: 'pass' }
      ]
    }
  ],
  signoffs: [
    { role: 'QA Lead', name: 'Jane Smith', date: '2025-06-14', approved: true },
    { role: 'Tech Lead', name: 'John Doe', date: '2025-06-14', approved: true },
    { role: 'Product Owner', name: 'Alice Johnson', approved: false }
  ]
};

// Check if release can proceed
function canRelease(checklist: ReleaseChecklist): boolean {
  const allChecksPassed = checklist.checks
    .flatMap(c => c.items)
    .every(item => item.status === 'pass' || item.status === 'skip');
  
  const allSignoffsApproved = checklist.signoffs
    .every(s => s.approved);
  
  return allChecksPassed && allSignoffsApproved;
}

console.log('Release approved:', canRelease(releaseChecklist));
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# APPENDICE: QUICK REFERENCE
# ═══════════════════════════════════════════════════════════════════════════════

QUICK_REFERENCE = """

## Testing Quick Reference Card

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ TESTING QUICK REFERENCE                                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ VITEST COMMANDS                                                             │
│ pnpm test                    → Run tests in watch mode                     │
│ pnpm test:run                → Run tests once                              │
│ pnpm test:ui                 → Open Vitest UI                              │
│ pnpm test:coverage           → Generate coverage report                    │
│ vitest related src/file.ts  → Run tests related to file                   │
│                                                                             │
│ PLAYWRIGHT COMMANDS                                                         │
│ pnpm e2e                     → Run all E2E tests                           │
│ pnpm e2e:ui                  → Open Playwright UI                          │
│ pnpm e2e:headed              → Run with browser visible                    │
│ pnpm e2e:debug               → Debug mode                                  │
│ npx playwright codegen       → Generate tests by recording                 │
│                                                                             │
│ REACT TESTING LIBRARY QUERIES (Priority)                                   │
│ 1. getByRole()              → Accessible queries (buttons, links)          │
│ 2. getByLabelText()         → Form inputs                                  │
│ 3. getByPlaceholderText()   → Inputs with placeholder                     │
│ 4. getByText()              → Non-interactive elements                     │
│ 5. getByTestId()            → Escape hatch (data-testid)                  │
│                                                                             │
│ QUERY VARIANTS                                                              │
│ getBy    → Throws if not found         (element exists)                    │
│ queryBy  → Returns null if not found   (element may not exist)             │
│ findBy   → Async, waits for element    (element will appear)               │
│                                                                             │
│ VITEST MOCKING                                                              │
│ vi.fn()                      → Create mock function                        │
│ vi.mock('module')            → Mock entire module                          │
│ vi.spyOn(obj, 'method')      → Spy on method                              │
│ vi.mocked(fn)                → TypeScript helper for mocked fn            │
│ vi.useFakeTimers()           → Mock timers                                │
│ vi.advanceTimersByTime(ms)   → Advance fake timers                        │
│                                                                             │
│ COMMON ASSERTIONS                                                           │
│ expect(x).toBe(y)            → Strict equality                             │
│ expect(x).toEqual(y)         → Deep equality                               │
│ expect(x).toBeTruthy()       → Truthy check                               │
│ expect(fn).toHaveBeenCalled()→ Function was called                        │
│ expect(fn).toThrow()         → Function throws                             │
│ expect(el).toBeInTheDocument()→ RTL: Element exists                       │
│ expect(el).toBeVisible()     → RTL: Element visible                        │
│ expect(el).toHaveTextContent()→ RTL: Text content                         │
│                                                                             │
│ TEST STRUCTURE                                                              │
│ describe('Component', () => {                                              │
│   beforeEach(() => { /* setup */ });                                       │
│   afterEach(() => { /* cleanup */ });                                      │
│   it('should do X when Y', () => {                                         │
│     // Arrange                                                             │
│     // Act                                                                 │
│     // Assert                                                              │
│   });                                                                      │
│ });                                                                        │
│                                                                             │
│ COVERAGE TARGETS                                                            │
│ Lines:      80%+                                                           │
│ Functions:  80%+                                                           │
│ Branches:   75%+                                                           │
│ Statements: 80%+                                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

"""

# ═══════════════════════════════════════════════════════════════════════════════
# FINE CATALOGO TESTING & QA
# ═══════════════════════════════════════════════════════════════════════════════
