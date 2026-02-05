# CATALOGO UI/UX - DESIGN SYSTEM FUNDAMENTALS v1

> **Versione**: 1.0
> **Data**: 2026-01-27
> **Ambito**: Design Tokens, Component Hierarchy, Spacing, Typography, Colors, Breakpoints, Tailwind Config

---

## 1. DESIGN TOKENS

### 1.1 Token Categories Table

| Categoria              | Token Name            | Valore Esempio 1 | Valore Esempio 2 | Valore Esempio 3 | Formato      |
| ---------------------- | --------------------- | ---------------- | ---------------- | ---------------- | ------------ |
| Color Primary          | --color-primary-500   | #3B82F6          | #2563EB          | #1D4ED8          | CSS Variable |
| Color Secondary        | --color-secondary-500 | #8B5CF6          | #7C3AED          | #6D28D9          | CSS Variable |
| Color Semantic         | --color-success       | #16A34A          | #15803D          | #166534          | CSS Variable |
| Color Neutral          | --color-gray-500      | #6B7280          | #4B5563          | #374151          | CSS Variable |
| Typography Font Family | --font-sans           | Inter            | system-ui        | sans-serif       | CSS Variable |
| Typography Font Size   | --text-body           | 16px             | 14px             | 18px             | CSS Variable |
| Typography Line Height | --line-body           | 24px             | 20px             | 28px             | CSS Variable |
| Typography Weight      | --font-weight-regular | 400              | 500              | 600              | CSS Variable |
| Spacing                | --space-4             | 16px             | 1rem             | 0.5rem           | CSS Variable |
| Sizing                 | --size-avatar-md      | 40px             | 48px             | 56px             | CSS Variable |
| Border Radius          | --radius-md           | 8px              | 12px             | 16px             | CSS Variable |
| Border Width           | --border-default      | 1px              | 2px              | 4px              | CSS Variable |
| Shadow                 | --shadow-soft         | 0 2px 8px        | 0 4px 16px       | 0 8px 24px       | CSS Variable |
| Opacity                | --opacity-disabled    | 0.4              | 0.6              | 0.8              | CSS Variable |
| Z-Index                | --z-modal             | 40               | 50               | 60               | CSS Variable |

### 1.2 Token Implementation

```typescript
// tokens.ts
export const tokens = {
  colors: {
    primary: {
      50: '#EFF6FF',
      100: '#DBEAFE',
      200: '#BFDBFE',
      300: '#93C5FD',
      400: '#60A5FA',
      500: '#3B82F6',
      600: '#2563EB',
      700: '#1D4ED8',
      800: '#1E40AF',
      900: '#1E3A8A',
    },
    semantic: {
      success: '#16A34A',
      warning: '#F59E0B',
      error: '#DC2626',
      info: '#0EA5E9',
    },
  },
  spacing: {
    0: '0px',
    1: '0.25rem',
    2: '0.5rem',
    3: '0.75rem',
    4: '1rem',
    5: '1.25rem',
    6: '1.5rem',
    8: '2rem',
    10: '2.5rem',
    12: '3rem',
    16: '4rem',
    20: '5rem',
    24: '6rem',
  },
  typography: {
    fontFamily: {
      sans: ['Inter', 'system-ui', 'sans-serif'],
      mono: ['JetBrains Mono', 'monospace'],
    },
    fontSize: {
      display2xl: '4.5rem',
      displayXl: '3.75rem',
      displayLg: '3rem',
      h1: '2.25rem',
      h2: '1.875rem',
      h3: '1.5rem',
      h4: '1.25rem',
      bodyLg: '1.125rem',
      body: '1rem',
      bodySm: '0.875rem',
      caption: '0.75rem',
    },
    lineHeight: {
      tight: '1.1',
      snug: '1.25',
      normal: '1.5',
      relaxed: '1.6',
    },
    fontWeight: {
      regular: 400,
      medium: 500,
      semibold: 600,
      bold: 700,
    },
  },
} as const;

export type Tokens = typeof tokens;
```

---

## 2. COMPONENT HIERARCHY (ATOMIC DESIGN)

### 2.1 Hierarchy Table

| Level    | Nome Componente    | Descrizione (max 10 parole) | Props Principali          |
| -------- | ------------------ | --------------------------- | ------------------------- |
| Atom     | Button             | Azione primaria cliccabile  | variant, size, disabled   |
| Atom     | Input              | Campo input testuale        | value, placeholder, error |
| Atom     | Label              | Etichetta campo form        | htmlFor, required         |
| Atom     | Icon               | Icona vettoriale            | name, size, color         |
| Atom     | Spinner            | Indicatore caricamento      | size, color               |
| Atom     | Badge              | Etichetta stato             | variant                   |
| Atom     | Avatar             | Immagine utente             | src, size                 |
| Atom     | Checkbox           | Selezione booleana          | checked, onChange         |
| Atom     | Switch             | Toggle booleano             | checked, disabled         |
| Atom     | Divider            | Separatore visivo           | orientation               |
| Molecule | SearchField        | Input con azione ricerca    | onSearch, placeholder     |
| Molecule | FormField          | Label + Input + Error       | name, control             |
| Molecule | CardHeader         | Titolo e azioni card        | title, actions            |
| Molecule | CardFooter         | Azioni card                 | children                  |
| Molecule | Dropdown           | Menu a discesa              | items, onSelect           |
| Molecule | ModalHeader        | Titolo modale               | title, onClose            |
| Molecule | ModalFooter        | Azioni modale               | confirmText               |
| Molecule | InputGroup         | Input con addon             | prefix, suffix            |
| Molecule | ListItem           | Riga lista                  | icon, label               |
| Molecule | Toast              | Messaggio temporaneo        | variant, message          |
| Organism | Navbar             | Header navigazione          | links, user               |
| Organism | Sidebar            | Navigazione laterale        | items, collapsed          |
| Organism | Footer             | Footer applicazione         | links                     |
| Organism | LoginForm          | Form autenticazione         | onSubmit                  |
| Organism | UserMenu           | Menu utente                 | user, actions             |
| Organism | DataTable          | Tabella dati                | columns, data             |
| Organism | FilterPanel        | Filtri ricerca              | filters, onApply          |
| Organism | NotificationCenter | Lista notifiche             | items                     |
| Organism | DashboardCards     | KPI cards                   | metrics                   |
| Organism | SettingsForm       | Impostazioni utente         | values, onSave            |
| Template | DashboardLayout    | Layout dashboard            | sidebar, children         |
| Template | AuthLayout         | Layout autenticazione       | children                  |
| Template | MarketingLayout    | Layout marketing            | header, footer            |
| Template | SettingsLayout     | Layout impostazioni         | tabs                      |
| Template | EmptyStateLayout   | Layout stato vuoto          | icon, message             |
| Page     | LoginPage          | Pagina login                | onLogin                   |
| Page     | DashboardPage      | Home dashboard              | data                      |
| Page     | SettingsPage       | Impostazioni utente         | sections                  |
| Page     | ProfilePage        | Profilo utente              | user                      |
| Page     | NotFoundPage       | Errore 404                  | redirect                  |

---

## 3. SPACING SCALE

### 3.1 Scale Table

| Token     | Pixels | REM       | Tailwind Class | Use Case                   |
| --------- | ------ | --------- | -------------- | -------------------------- |
| space-0   | 0px    | 0rem      | p-0, m-0       | Reset                      |
| space-px  | 1px    | 0.0625rem | p-px           | Border spacing             |
| space-0.5 | 2px    | 0.125rem  | p-0.5          | Tight icon gaps            |
| space-1   | 4px    | 0.25rem   | p-1            | Inline element spacing     |
| space-2   | 8px    | 0.5rem    | p-2            | Compact component padding  |
| space-3   | 12px   | 0.75rem   | p-3            | Default icon spacing       |
| space-4   | 16px   | 1rem      | p-4            | Standard component padding |
| space-5   | 20px   | 1.25rem   | p-5            | Card padding small         |
| space-6   | 24px   | 1.5rem    | p-6            | Card padding default       |
| space-8   | 32px   | 2rem      | p-8            | Section spacing            |
| space-10  | 40px   | 2.5rem    | p-10           | Large section spacing      |
| space-12  | 48px   | 3rem      | p-12           | Container padding          |
| space-16  | 64px   | 4rem      | p-16           | Page section gap           |
| space-20  | 80px   | 5rem      | p-20           | Hero section spacing       |
| space-24  | 96px   | 6rem      | p-24           | Maximum spacing            |

---

## 4. TYPOGRAPHY SCALE

### 4.1 Type Scale Table

| Token            | Font Size       | Line Height | Letter Spacing | Font Weight | Use Case         |
| ---------------- | --------------- | ----------- | -------------- | ----------- | ---------------- |
| text-display-2xl | 72px / 4.5rem   | 1.1         | -0.02em        | 700         | Hero headlines   |
| text-display-xl  | 60px / 3.75rem  | 1.1         | -0.02em        | 700         | Page titles      |
| text-display-lg  | 48px / 3rem     | 1.1         | -0.015em       | 600         | Section titles   |
| text-h1          | 36px / 2.25rem  | 1.2         | -0.01em        | 600         | H1               |
| text-h2          | 30px / 1.875rem | 1.25        | -0.01em        | 600         | H2               |
| text-h3          | 24px / 1.5rem   | 1.3         | 0em            | 600         | H3               |
| text-h4          | 20px / 1.25rem  | 1.4         | 0em            | 500         | H4               |
| text-body-lg     | 18px / 1.125rem | 1.6         | 0em            | 400         | Lead paragraphs  |
| text-body        | 16px / 1rem     | 1.5         | 0em            | 400         | Body text        |
| text-body-sm     | 14px / 0.875rem | 1.45        | 0em            | 400         | Secondary text   |
| text-caption     | 12px / 0.75rem  | 1.4         | 0.01em         | 400         | Captions, labels |

---

## 5. COLOR SYSTEM

### 5.1 Primary Palette

| Shade       | Hex     | RGB         | HSL          | Use Case           |
| ----------- | ------- | ----------- | ------------ | ------------------ |
| primary-50  | #EFF6FF | 239,246,255 | 214,100%,97% | Backgrounds hover  |
| primary-100 | #DBEAFE | 219,234,254 | 214,95%,93%  | Backgrounds active |
| primary-200 | #BFDBFE | 191,219,254 | 213,97%,87%  | Borders light      |
| primary-300 | #93C5FD | 147,197,253 | 212,96%,78%  | Borders            |
| primary-400 | #60A5FA | 96,165,250  | 213,94%,68%  | Icons secondary    |
| primary-500 | #3B82F6 | 59,130,246  | 217,91%,60%  | Primary default    |
| primary-600 | #2563EB | 37,99,235   | 221,83%,53%  | Primary hover      |
| primary-700 | #1D4ED8 | 29,78,216   | 224,76%,48%  | Primary active     |
| primary-800 | #1E40AF | 30,64,175   | 226,71%,40%  | Text on light      |
| primary-900 | #1E3A8A | 30,58,138   | 224,64%,33%  | Text emphasis      |

### 5.2 Semantic Colors

| Name    | Default | Hover   | Active  | Light BG | Use Case                |
| ------- | ------- | ------- | ------- | -------- | ----------------------- |
| success | #16A34A | #15803D | #166534 | #DCFCE7  | Conferme, completamenti |
| warning | #F59E0B | #D97706 | #B45309 | #FEF3C7  | Avvisi, attenzione      |
| error   | #DC2626 | #B91C1C | #991B1B | #FEE2E2  | Errori, eliminazioni    |
| info    | #0EA5E9 | #0284C7 | #0369A1 | #E0F2FE  | Informazioni, help      |

### 5.3 Neutral Grays

| Shade    | Hex     | Use Case          |
| -------- | ------- | ----------------- |
| gray-50  | #F9FAFB | Page background   |
| gray-100 | #F3F4F6 | Card background   |
| gray-200 | #E5E7EB | Borders, dividers |
| gray-300 | #D1D5DB | Disabled borders  |
| gray-400 | #9CA3AF | Placeholder text  |
| gray-500 | #6B7280 | Secondary text    |
| gray-600 | #4B5563 | Body text         |
| gray-700 | #374151 | Headings          |
| gray-800 | #1F2937 | Primary text      |
| gray-900 | #111827 | Highest contrast  |

---

## 6. BREAKPOINTS

### 6.1 Breakpoint Table

| Name | Min Width | Max Width | Tailwind Prefix | Target Devices             |
| ---- | --------- | --------- | --------------- | -------------------------- |
| xs   | 0px       | 639px     | (default)       | Mobile portrait            |
| sm   | 640px     | 767px     | sm:             | Mobile landscape           |
| md   | 768px     | 1023px    | md:             | Tablet portrait            |
| lg   | 1024px    | 1279px    | lg:             | Tablet landscape / Desktop |
| xl   | 1280px    | 1535px    | xl:             | Desktop                    |
| 2xl  | 1536px    | ∞         | 2xl:            | Large desktop              |

### 6.2 Container Widths

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss';

export default {
  theme: {
    container: {
      center: true,
      padding: {
        DEFAULT: '1rem',
        sm: '2rem',
        lg: '4rem',
        xl: '5rem',
        '2xl': '6rem',
      },
      screens: {
        sm: '640px',
        md: '768px',
        lg: '1024px',
        xl: '1280px',
        '2xl': '1400px',
      },
    },
  },
} satisfies Config;
```

---

## 7. TAILWIND CONFIG COMPLETO

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss';

export default {
  content: ['./src/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#EFF6FF',
          100: '#DBEAFE',
          200: '#BFDBFE',
          300: '#93C5FD',
          400: '#60A5FA',
          500: '#3B82F6',
          600: '#2563EB',
          700: '#1D4ED8',
          800: '#1E40AF',
          900: '#1E3A8A',
        },
        success: '#16A34A',
        warning: '#F59E0B',
        error: '#DC2626',
        info: '#0EA5E9',
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
        mono: ['JetBrains Mono', 'monospace'],
      },
      fontSize: {
        'display-2xl': ['4.5rem', { lineHeight: '1.1', letterSpacing: '-0.02em' }],
        'display-xl': ['3.75rem', { lineHeight: '1.1', letterSpacing: '-0.02em' }],
        'display-lg': ['3rem', { lineHeight: '1.1', letterSpacing: '-0.015em' }],
      },
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
        '112': '28rem',
        '128': '32rem',
      },
      borderRadius: {
        '4xl': '2rem',
      },
      boxShadow: {
        soft: '0 2px 15px -3px rgba(0, 0, 0, 0.07), 0 10px 20px -2px rgba(0, 0, 0, 0.04)',
      },
    },
  },
  plugins: [],
} satisfies Config;
```

---

## 8. ACCESSIBILITY FUNDAMENTALS

### 8.1 WCAG Mapping Table

| Area           | Requirement          | Token / Rule     | Valore Specifico        | Standard    |
| -------------- | -------------------- | ---------------- | ----------------------- | ----------- |
| Color Contrast | Text on background   | Contrast ratio   | ≥ 4.5:1                 | WCAG 2.1 AA |
| Color Contrast | Large text           | Contrast ratio   | ≥ 3:1                   | WCAG 2.1 AA |
| Focus Visible  | Interactive elements | Outline          | 2px solid #2563EB       | WCAG 2.1 AA |
| Hit Area       | Touch targets        | Min size         | 44px × 44px             | WCAG 2.1 AA |
| Keyboard       | Navigation           | Tab order        | Logical DOM order       | WCAG 2.1 AA |
| Screen Reader  | Buttons              | aria-label       | Required when icon-only | ARIA        |
| Screen Reader  | Forms                | aria-describedby | Error/help text id      | ARIA        |
| Motion         | Animations           | Reduced motion   | prefers-reduced-motion  | WCAG 2.1 AA |

### 8.2 Focus Styles

```typescript
// focusStyles.ts
import type { CSSProperties } from 'react';

export const focusRing: CSSProperties = {
  outline: '2px solid #2563EB',
  outlineOffset: '2px',
};
```

---

## 9. COMPONENT STATES

### 9.1 State Table

| State    | Background | Border  | Text    | Opacity | Use Case          |
| -------- | ---------- | ------- | ------- | ------- | ----------------- |
| Default  | #FFFFFF    | #E5E7EB | #111827 | 1       | Normal            |
| Hover    | #EFF6FF    | #BFDBFE | #1D4ED8 | 1       | Pointer hover     |
| Active   | #DBEAFE    | #93C5FD | #1E40AF | 1       | Pressed           |
| Focus    | #FFFFFF    | #2563EB | #111827 | 1       | Keyboard focus    |
| Disabled | #F3F4F6    | #E5E7EB | #9CA3AF | 0.6     | Non-interactive   |
| Error    | #FEE2E2    | #DC2626 | #991B1B | 1       | Validation error  |
| Success  | #DCFCE7    | #16A34A | #166534 | 1       | Positive feedback |

---

## 10. MOTION SYSTEM

### 10.1 Motion Tokens Table

| Token        | Duration | Easing                     | Use Case        |
| ------------ | -------- | -------------------------- | --------------- |
| motion-fast  | 100ms    | ease-out                   | Hover feedback  |
| motion-base  | 200ms    | ease-in-out                | UI transitions  |
| motion-slow  | 300ms    | ease-in-out                | Modals, drawers |
| motion-enter | 250ms    | cubic-bezier(0.16,1,0.3,1) | Enter animation |
| motion-exit  | 200ms    | cubic-bezier(0.7,0,0.84,0) | Exit animation  |

### 10.2 Reduced Motion Handling

```typescript
// motion.ts
export const prefersReducedMotion =
  typeof window !== 'undefined' &&
  window.matchMedia('(prefers-reduced-motion: reduce)').matches;

export const transitionDuration = prefersReducedMotion ? '0ms' : '200ms';
```

---

## 11. COMPONENT IMPLEMENTATION EXAMPLE

### 11.1 Button Component

```typescript
// Button.tsx
import React from 'react';
import clsx from 'clsx';

type ButtonVariant = 'primary' | 'secondary' | 'danger';
type ButtonSize = 'sm' | 'md' | 'lg';

export interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant: ButtonVariant;
  size: ButtonSize;
}

export const Button: React.FC<ButtonProps> = ({
  variant,
  size,
  disabled,
  children,
  ...props
}) => {
  const classes = clsx(
    'inline-flex items-center justify-center font-medium rounded-md transition-colors',
    {
      'bg-primary-500 text-white hover:bg-primary-600 active:bg-primary-700': variant === 'primary',
      'bg-gray-100 text-gray-800 hover:bg-gray-200 active:bg-gray-300': variant === 'secondary',
      'bg-error text-white hover:bg-red-700 active:bg-red-800': variant === 'danger',
      'h-8 px-3 text-sm': size === 'sm',
      'h-10 px-4 text-base': size === 'md',
      'h-12 px-6 text-lg': size === 'lg',
      'opacity-60 cursor-not-allowed': disabled,
    }
  );

  return (
    <button
      type="button"
      className={classes}
      disabled={disabled}
      {...props}
    >
      {children}
    </button>
  );
};
```

---

## 12. THEMING STRATEGY

### 12.1 Theme Tokens Table

| Theme         | Background | Text Primary | Primary Color | Border  |
| ------------- | ---------- | ------------ | ------------- | ------- |
| Light         | #FFFFFF    | #111827      | #3B82F6       | #E5E7EB |
| Dark          | #111827    | #F9FAFB      | #60A5FA       | #374151 |
| High Contrast | #000000    | #FFFFFF      | #FFFF00       | #FFFFFF |

### 12.2 Theme Provider

```typescript
// ThemeProvider.tsx
import React, { createContext, useContext } from 'react';

export type Theme = 'light' | 'dark' | 'high-contrast';

export interface ThemeContextValue {
  theme: Theme;
}

const ThemeContext = createContext<ThemeContextValue>({ theme: 'light' });

export const useTheme = (): ThemeContextValue => useContext(ThemeContext);

export const ThemeProvider: React.FC<{
  theme: Theme;
  children: React.ReactNode;
}> = ({ theme, children }) => (
  <ThemeContext.Provider value={{ theme }}>
    <div data-theme={theme}>{children}</div>
  </ThemeContext.Provider>
);
```

---

## CHECKLIST DESIGN SYSTEM

```
□ Design Tokens definiti e documentati
□ Atomic Design hierarchy implementata
□ Spacing scale coerente (4px base)
□ Typography scale con font loading ottimizzato
□ Color system con semantic colors
□ Breakpoints mobile-first
□ Tailwind config estesa
□ WCAG 2.1 AA compliance
□ Component states definiti
□ Motion system con reduced-motion
□ Theme provider configurato
```
