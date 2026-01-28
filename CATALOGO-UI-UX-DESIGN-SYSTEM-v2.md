# CATALOGO UI/UX - DESIGN SYSTEM FUNDAMENTALS v2

> **Versione**: 2.0
> **Data**: 2026-01-27
> **Ambito**: Design Tokens, Component Hierarchy, Spacing, Typography, Colors, Animation, Dark Mode, Responsive, Icons, Accessibility
> **Sezioni**: 1-18 (12 originali + 6 espansione)

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


---

## 13. ANIMATION & MOTION SYSTEM

### 13.1 Duration Tokens

| Token | Value | CSS Variable | Use Case |
|-------|-------|--------------|----------|
| instant | 0ms | --duration-instant | Immediate feedback |
| faster | 75ms | --duration-faster | Micro hover states |
| fast | 150ms | --duration-fast | Button interactions |
| normal | 200ms | --duration-normal | Standard transitions |
| slow | 300ms | --duration-slow | Panel slides |
| slower | 500ms | --duration-slower | Page transitions |
| slowest | 700ms | --duration-slowest | Complex sequences |

### 13.2 Easing Tokens

| Token | CSS Value | Use Case | Feel |
|-------|-----------|----------|------|
| ease-linear | linear | Progress bars | Constant |
| ease-in | cubic-bezier(0.4, 0, 1, 1) | Exit animations | Accelerating |
| ease-out | cubic-bezier(0, 0, 0.2, 1) | Enter animations | Decelerating |
| ease-in-out | cubic-bezier(0.4, 0, 0.2, 1) | Move animations | Smooth |
| ease-spring | cubic-bezier(0.34, 1.56, 0.64, 1) | Bouncy elements | Playful |
| ease-bounce | cubic-bezier(0.68, -0.55, 0.265, 1.55) | Attention grab | Fun |

### 13.3 Framer Motion Variants

```typescript
import { Variants, Transition } from 'framer-motion';

export const transitions = {
  fast: { duration: 0.15, ease: [0, 0, 0.2, 1] } as Transition,
  normal: { duration: 0.2, ease: [0.4, 0, 0.2, 1] } as Transition,
  spring: { type: 'spring', stiffness: 400, damping: 30 } as Transition,
};

export const fadeIn: Variants = {
  hidden: { opacity: 0 },
  visible: { opacity: 1, transition: transitions.normal },
  exit: { opacity: 0, transition: transitions.fast },
};

export const slideUp: Variants = {
  hidden: { opacity: 0, y: 16 },
  visible: { opacity: 1, y: 0, transition: transitions.normal },
  exit: { opacity: 0, y: -8, transition: transitions.fast },
};

export const scaleIn: Variants = {
  hidden: { opacity: 0, scale: 0.95 },
  visible: { opacity: 1, scale: 1, transition: transitions.spring },
  exit: { opacity: 0, scale: 0.98, transition: transitions.fast },
};

export const staggerContainer: Variants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: { staggerChildren: 0.05, delayChildren: 0.1 },
  },
};
```

### 13.4 Reduced Motion Support

```typescript
import { useEffect, useState } from 'react';

export function useReducedMotion(): boolean {
  const [prefersReducedMotion, setPrefersReducedMotion] = useState(false);

  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
    setPrefersReducedMotion(mediaQuery.matches);

    const handleChange = (event: MediaQueryListEvent) => {
      setPrefersReducedMotion(event.matches);
    };

    mediaQuery.addEventListener('change', handleChange);
    return () => mediaQuery.removeEventListener('change', handleChange);
  }, []);

  return prefersReducedMotion;
}
```


```

---

## 14. DARK MODE IMPLEMENTATION

### 14.1 Color Token Mapping

| Token | Light Mode | Dark Mode | High Contrast |
|-------|------------|-----------|---------------|
| --background | #FFFFFF | #0A0A0A | #000000 |
| --foreground | #0A0A0A | #FAFAFA | #FFFFFF |
| --card | #FFFFFF | #18181B | #0A0A0A |
| --card-foreground | #0A0A0A | #FAFAFA | #FFFFFF |
| --primary | #2563EB | #3B82F6 | #60A5FA |
| --primary-foreground | #FFFFFF | #FFFFFF | #000000 |
| --secondary | #F4F4F5 | #27272A | #18181B |
| --muted | #F4F4F5 | #27272A | #18181B |
| --muted-foreground | #71717A | #A1A1AA | #D4D4D8 |
| --border | #E4E4E7 | #27272A | #3F3F46 |
| --input | #E4E4E7 | #27272A | #3F3F46 |
| --ring | #2563EB | #3B82F6 | #60A5FA |

### 14.2 CSS Variables Setup

```css
/* app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 0 0% 3.9%;
    --card: 0 0% 100%;
    --card-foreground: 0 0% 3.9%;
    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --border: 214.3 31.8% 91.4%;
    --ring: 221.2 83.2% 53.3%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 0 0% 3.9%;
    --foreground: 0 0% 98%;
    --card: 0 0% 7.8%;
    --card-foreground: 0 0% 98%;
    --primary: 217.2 91.2% 59.8%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --border: 217.2 32.6% 17.5%;
    --ring: 224.3 76.3% 48%;
  }
}
```

### 14.3 Theme Provider (next-themes)

```typescript
// providers/theme-provider.tsx
'use client';

import * as React from 'react';
import { ThemeProvider as NextThemesProvider } from 'next-themes';

export function ThemeProvider({ children, ...props }: { children: React.ReactNode }) {
  return (
    <NextThemesProvider
      attribute="class"
      defaultTheme="system"
      enableSystem
      disableTransitionOnChange
      storageKey="theme"
      {...props}
    >
      {children}
    </NextThemesProvider>
  );
}
```

### 14.4 Theme Toggle Component

```typescript
// components/theme-toggle.tsx
'use client';

import { Moon, Sun, Monitor } from 'lucide-react';
import { useTheme } from 'next-themes';
import { Button } from '@/components/ui/button';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';

export function ThemeToggle() {
  const { setTheme } = useTheme();

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="ghost" size="icon">
          <Sun className="h-5 w-5 rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
          <Moon className="absolute h-5 w-5 rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
          <span className="sr-only">Toggle theme</span>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end">
        <DropdownMenuItem onClick={() => setTheme('light')}>
          <Sun className="mr-2 h-4 w-4" /> Light
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => setTheme('dark')}>
          <Moon className="mr-2 h-4 w-4" /> Dark
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => setTheme('system')}>
          <Monitor className="mr-2 h-4 w-4" /> System
        </DropdownMenuItem>
      </DropdownMenuContent>
    </DropdownMenu>
  );
}
```

---

## 15. RESPONSIVE PATTERNS

### 15.1 Breakpoint Tokens

| Token | Min Width | Max Width | Tailwind | Target |
|-------|-----------|-----------|----------|--------|
| xs | 0px | 479px | default | Small mobile |
| sm | 480px | 639px | sm: | Mobile |
| md | 640px | 767px | md: | Large mobile |
| lg | 768px | 1023px | lg: | Tablet |
| xl | 1024px | 1279px | xl: | Desktop |
| 2xl | 1280px | ∞ | 2xl: | Large desktop |

### 15.2 Responsive Hook

```typescript
// hooks/use-media-query.ts
import { useEffect, useState } from 'react';

type Breakpoint = 'xs' | 'sm' | 'md' | 'lg' | 'xl' | '2xl';

const breakpoints: Record<Breakpoint, string> = {
  xs: '(max-width: 479px)',
  sm: '(min-width: 480px)',
  md: '(min-width: 640px)',
  lg: '(min-width: 768px)',
  xl: '(min-width: 1024px)',
  '2xl': '(min-width: 1280px)',
};

export function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(false);

  useEffect(() => {
    const media = window.matchMedia(query);
    setMatches(media.matches);
    const listener = (e: MediaQueryListEvent) => setMatches(e.matches);
    media.addEventListener('change', listener);
    return () => media.removeEventListener('change', listener);
  }, [query]);

  return matches;
}

export function useBreakpoint(breakpoint: Breakpoint): boolean {
  return useMediaQuery(breakpoints[breakpoint]);
}
```

### 15.3 Container Query Setup

```typescript
// components/adaptive-card.tsx
export function AdaptiveCard({ children }: { children: React.ReactNode }) {
  return (
    <div className="@container">
      <div className="
        flex flex-col gap-3
        @xs:flex-row @xs:items-center @xs:gap-4
        @sm:gap-6
        @md:p-6
      ">
        {children}
      </div>
    </div>
  );
}
```

---

## 16. ICON SYSTEM

### 16.1 Icon Library Comparison

| Library | Icons | Size | Tree-shake | Style | License |
|---------|-------|------|------------|-------|---------|
| Lucide React | 1,400+ | 0KB base | ✅ | Outline 24px | ISC |
| Heroicons | 450+ | 0KB base | ✅ | Outline/Solid | MIT |
| Radix Icons | 300+ | 0KB base | ✅ | Outline 15px | MIT |
| Phosphor | 7,000+ | 0KB base | ✅ | 6 weights | MIT |
| Tabler Icons | 4,000+ | 0KB base | ✅ | Outline | MIT |

### 16.2 Icon Component Wrapper

```typescript
// components/ui/icon.tsx
import { LucideIcon, LucideProps } from 'lucide-react';
import { cn } from '@/lib/utils';

type IconSize = 'xs' | 'sm' | 'md' | 'lg' | 'xl';

const sizeMap: Record<IconSize, { class: string; strokeWidth: number }> = {
  xs: { class: 'h-3 w-3', strokeWidth: 2.5 },
  sm: { class: 'h-4 w-4', strokeWidth: 2 },
  md: { class: 'h-5 w-5', strokeWidth: 2 },
  lg: { class: 'h-6 w-6', strokeWidth: 1.75 },
  xl: { class: 'h-8 w-8', strokeWidth: 1.5 },
};

export function Icon({ 
  icon: IconComponent, 
  size = 'md', 
  className,
  label,
  ...props 
}: { icon: LucideIcon; size?: IconSize; label?: string } & LucideProps) {
  const { class: sizeClass, strokeWidth } = sizeMap[size];
  
  return (
    <IconComponent 
      className={cn(sizeClass, className)} 
      strokeWidth={strokeWidth}
      aria-label={label}
      aria-hidden={!label}
      {...props} 
    />
  );
}
```

---

## 17. ACCESSIBILITY TOKENS

### 17.1 A11y Token Reference

| Token | Value | Purpose | WCAG Criterion |
|-------|-------|---------|----------------|
| --focus-ring-width | 2px | Focus indicator width | 2.4.7 |
| --focus-ring-offset | 2px | Focus ring gap | 2.4.7 |
| --focus-ring-color | hsl(var(--ring)) | Focus ring color | 2.4.7 |
| --min-touch-target | 44px | Minimum touch area | 2.5.5 |
| --contrast-text-normal | 4.5:1 | Normal text contrast | 1.4.3 AA |
| --contrast-text-large | 3:1 | Large text contrast | 1.4.3 AA |
| --contrast-ui | 3:1 | UI component contrast | 1.4.11 |

### 17.2 Focus Ring Implementation

```typescript
// lib/focus.ts
import { cn } from '@/lib/utils';

export const focusRing = cn(
  'focus-visible:outline-none',
  'focus-visible:ring-2',
  'focus-visible:ring-ring',
  'focus-visible:ring-offset-2',
  'focus-visible:ring-offset-background'
);

export const focusRingInset = cn(
  'focus-visible:outline-none',
  'focus-visible:ring-2',
  'focus-visible:ring-ring',
  'focus-visible:ring-inset'
);
```

---

## 18. DESIGN SYSTEM EXPANSION CHECKLIST

```
ANIMATION & MOTION
□ Duration tokens defined (instant → slowest)
□ Easing functions documented
□ Framer Motion variants created
□ Tailwind keyframes configured
□ Reduced motion support implemented
□ Exit animations defined

DARK MODE
□ Color tokens for both themes
□ CSS variables in :root and .dark
□ Theme provider configured
□ No flash on initial load
□ System preference detection
□ Persistent theme storage

RESPONSIVE
□ Breakpoint tokens documented
□ useMediaQuery hook available
□ Container queries configured
□ Mobile-first approach
□ Touch targets ≥44px on mobile

ICONS
□ Icon library selected
□ Icon wrapper component
□ Consistent sizing scale
□ Accessibility labels

ACCESSIBILITY
□ Focus ring tokens defined
□ Focus styles applied globally
□ Contrast ratios documented
□ Touch targets verified
□ Reduced motion respected
```
