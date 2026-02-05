# CATALOGO UI/UX - A11Y ACCESSIBILITY PATTERNS v1

> **Versione**: 1.0
> **Data**: 2026-01-27
> **Ambito**: Componenti accessibili WCAG 2.1 AA, ARIA patterns, keyboard navigation

---

## 1. MODAL/DIALOG

### ARIA Specification

| Proprietà        | Valore         | Obbligatorio |
| ---------------- | -------------- | ------------ |
| role             | dialog         | ✅            |
| aria-modal       | true           | ✅            |
| aria-labelledby  | id titolo      | ✅            |
| aria-describedby | id descrizione | ⚠️           |

### Keyboard Navigation

| Tasto     | Azione         | Note        |
| --------- | -------------- | ----------- |
| Tab       | Next focus     | Focus trap  |
| Shift+Tab | Previous focus | Focus trap  |
| Escape    | Close dialog   | Sempre      |
| Enter     | Attiva         | Button/link |

### Focus Management

| Evento | Comportamento        |
| ------ | -------------------- |
| Open   | Focus primo elemento |
| Close  | Restore focus        |
| Trap   | Focus limitato       |

### Implementazione

```typescript
import React, { useEffect, useRef } from 'react';

interface ModalProps {
  open: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
}

export const Modal: React.FC<ModalProps> = ({ open, onClose, title, children }) => {
  const ref = useRef<HTMLDivElement>(null);
  const prevFocus = useRef<HTMLElement | null>(null);

  useEffect(() => {
    if (open) prevFocus.current = document.activeElement as HTMLElement;
  }, [open]);

  useEffect(() => {
    if (!open) return;
    const handler = (e: KeyboardEvent) => {
      if (e.key === 'Escape') onClose();
    };
    document.addEventListener('keydown', handler);
    ref.current?.focus();
    return () => {
      document.removeEventListener('keydown', handler);
      prevFocus.current?.focus();
    };
  }, [open, onClose]);

  if (!open) return null;

  return (
    <div role="dialog" aria-modal="true" aria-labelledby="modal-title">
      <div tabIndex={-1} ref={ref}>
        <h2 id="modal-title">{title}</h2>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>
  );
};
```

### Testing Checklist

| Test       | Metodo | Expected        |
| ---------- | ------ | --------------- |
| Focus trap | Tab    | Resta nel modal |
| Escape     | ESC    | Chiude          |
| SR         | NVDA   | Annuncia dialog |

---

## 2. DROPDOWN MENU

### ARIA Specification

| Proprietà     | Valore  | Obbligatorio |
| ------------- | ------- | ------------ |
| role          | menu    | ✅            |
| aria-haspopup | true    | ✅            |
| aria-expanded | boolean | ✅            |

### Keyboard Navigation

| Tasto     | Azione    |
| --------- | --------- |
| Enter     | Open      |
| ArrowDown | Next item |
| ArrowUp   | Prev item |
| Escape    | Close     |

### Implementazione

```typescript
import React, { useState, useRef } from 'react';

export const Dropdown: React.FC = () => {
  const [open, setOpen] = useState(false);
  const btnRef = useRef<HTMLButtonElement>(null);

  return (
    <div>
      <button
        ref={btnRef}
        aria-haspopup="true"
        aria-expanded={open}
        onClick={() => setOpen(!open)}
      >
        Menu
      </button>
      {open && (
        <ul role="menu">
          <li role="menuitem" tabIndex={0}>Item 1</li>
          <li role="menuitem" tabIndex={0}>Item 2</li>
        </ul>
      )}
    </div>
  );
};
```

---

## 3. TABS

### ARIA Specification

| Proprietà     | Valore   | Obbligatorio |
| ------------- | -------- | ------------ |
| role          | tablist  | ✅            |
| role          | tab      | ✅            |
| role          | tabpanel | ✅            |
| aria-selected | boolean  | ✅            |

### Keyboard Navigation

| Tasto      | Azione   |
| ---------- | -------- |
| ArrowRight | Next tab |
| ArrowLeft  | Prev tab |
| Enter      | Activate |

### Implementazione

```typescript
import React, { useState } from 'react';

export const Tabs: React.FC = () => {
  const [active, setActive] = useState(0);

  return (
    <div>
      <div role="tablist">
        {[0, 1].map((i) => (
          <button
            key={i}
            role="tab"
            aria-selected={active === i}
            onClick={() => setActive(i)}
          >
            Tab {i + 1}
          </button>
        ))}
      </div>
      <div role="tabpanel">Content {active + 1}</div>
    </div>
  );
};
```

---

## 4. ACCORDION

### ARIA Specification

| Proprietà     | Valore   | Obbligatorio |
| ------------- | -------- | ------------ |
| aria-expanded | boolean  | ✅            |
| aria-controls | id panel | ✅            |

### Keyboard Navigation

| Tasto | Azione |
| ----- | ------ |
| Enter | Toggle |
| Space | Toggle |

### Implementazione

```typescript
import React, { useState } from 'react';

export const Accordion: React.FC = () => {
  const [open, setOpen] = useState(false);

  return (
    <div>
      <button
        aria-expanded={open}
        aria-controls="panel-1"
        onClick={() => setOpen(!open)}
      >
        Section
      </button>
      {open && <div id="panel-1">Content</div>}
    </div>
  );
};
```

---

## 5. TOOLTIP

### ARIA Specification

| Proprietà        | Valore     | Obbligatorio |
| ---------------- | ---------- | ------------ |
| role             | tooltip    | ✅            |
| aria-describedby | id tooltip | ✅            |

### Implementazione

```typescript
import React, { useState } from 'react';

export const Tooltip: React.FC<{ text: string }> = ({ text }) => {
  const [show, setShow] = useState(false);

  return (
    <span>
      <button
        aria-describedby="tooltip"
        onFocus={() => setShow(true)}
        onBlur={() => setShow(false)}
        onMouseEnter={() => setShow(true)}
        onMouseLeave={() => setShow(false)}
      >
        Info
      </button>
      {show && <span id="tooltip" role="tooltip">{text}</span>}
    </span>
  );
};
```

---

## 6. TOAST/NOTIFICATION

### ARIA Specification

| Proprietà | Valore | Obbligatorio |
| --------- | ------ | ------------ |
| role      | status | ✅            |
| aria-live | polite | ✅            |

### Implementazione

```typescript
import React, { useEffect, useState } from 'react';

export const Toast: React.FC<{ message: string }> = ({ message }) => {
  const [visible, setVisible] = useState(true);

  useEffect(() => {
    const t = setTimeout(() => setVisible(false), 4000);
    return () => clearTimeout(t);
  }, []);

  return visible ? (
    <div role="status" aria-live="polite">
      {message}
    </div>
  ) : null;
};
```

---

## 7. AUTOCOMPLETE/COMBOBOX

### ARIA Specification

| Proprietà          | Valore  | Obbligatorio |
| ------------------ | ------- | ------------ |
| role               | combobox| ✅            |
| aria-expanded      | boolean | ✅            |
| aria-autocomplete  | list    | ✅            |
| aria-activedescendant | id   | ⚠️           |

### Implementazione

```typescript
import React, { useState } from 'react';

export const Combobox: React.FC = () => {
  const [value, setValue] = useState('');
  const options = ['Apple', 'Banana', 'Cherry'];

  return (
    <div>
      <input
        role="combobox"
        aria-expanded="true"
        aria-autocomplete="list"
        value={value}
        onChange={(e) => setValue(e.target.value)}
      />
      <ul role="listbox">
        {options.filter(o => o.toLowerCase().includes(value.toLowerCase())).map((o) => (
          <li key={o} role="option">{o}</li>
        ))}
      </ul>
    </div>
  );
};
```

---

## 8. DATE PICKER

### ARIA Specification

| Proprietà  | Valore          | Obbligatorio |
| ---------- | --------------- | ------------ |
| aria-label | "Select date"   | ✅            |
| role       | grid (calendar) | ✅            |

### Implementazione

```typescript
import React, { useState } from 'react';

export const DatePicker: React.FC = () => {
  const [date, setDate] = useState('');

  return (
    <label>
      Date
      <input
        type="date"
        aria-label="Select date"
        value={date}
        onChange={(e) => setDate(e.target.value)}
      />
    </label>
  );
};
```

---

## 9. SLIDER/RANGE

### ARIA Specification

| Proprietà      | Valore | Obbligatorio |
| -------------- | ------ | ------------ |
| role           | slider | ✅            |
| aria-valuemin  | number | ✅            |
| aria-valuemax  | number | ✅            |
| aria-valuenow  | number | ✅            |
| aria-valuetext | string | ⚠️           |

### Implementazione

```typescript
import React, { useState } from 'react';

export const Slider: React.FC = () => {
  const [value, setValue] = useState(50);

  return (
    <input
      type="range"
      role="slider"
      aria-valuemin={0}
      aria-valuemax={100}
      aria-valuenow={value}
      aria-valuetext={`${value}%`}
      value={value}
      onChange={(e) => setValue(Number(e.target.value))}
    />
  );
};
```

---

## 10. TOGGLE/SWITCH

### ARIA Specification

| Proprietà    | Valore  | Obbligatorio |
| ------------ | ------- | ------------ |
| role         | switch  | ✅            |
| aria-checked | boolean | ✅            |

### Implementazione

```typescript
import React, { useState } from 'react';

export const Switch: React.FC = () => {
  const [on, setOn] = useState(false);

  return (
    <button
      role="switch"
      aria-checked={on}
      onClick={() => setOn(!on)}
    >
      {on ? 'On' : 'Off'}
    </button>
  );
};
```

---

## 11. BREADCRUMB

### ARIA Specification

| Proprietà    | Valore      | Obbligatorio |
| ------------ | ----------- | ------------ |
| aria-label   | "Breadcrumb"| ✅            |
| aria-current | "page"      | ✅ (last)    |

### Implementazione

```typescript
import React from 'react';

export const Breadcrumb: React.FC = () => (
  <nav aria-label="Breadcrumb">
    <ol>
      <li><a href="/">Home</a></li>
      <li><a href="/products">Products</a></li>
      <li aria-current="page">Current Page</li>
    </ol>
  </nav>
);
```

---

## 12. PAGINATION

### ARIA Specification

| Proprietà  | Valore       | Obbligatorio |
| ---------- | ------------ | ------------ |
| aria-label | "Pagination" | ✅            |
| aria-current | "page"     | ✅ (active)  |

### Implementazione

```typescript
import React, { useState } from 'react';

export const Pagination: React.FC = () => {
  const [page, setPage] = useState(1);

  return (
    <nav aria-label="Pagination">
      <button onClick={() => setPage(page - 1)} disabled={page === 1}>Prev</button>
      <span aria-current="page">{page}</span>
      <button onClick={() => setPage(page + 1)}>Next</button>
    </nav>
  );
};
```

---

## 13. DATA TABLE (SORTABLE)

### ARIA Specification

| Proprietà   | Valore               | Obbligatorio |
| ----------- | -------------------- | ------------ |
| role        | table                | ✅            |
| aria-sort   | ascending/descending | ✅ (sortable)|
| scope       | col/row              | ✅            |

### Implementazione

```typescript
import React, { useState } from 'react';

interface Row { name: string; }

export const Table: React.FC = () => {
  const [rows, setRows] = useState<Row[]>([{ name: 'Bob' }, { name: 'Alice' }]);
  const [sortDir, setSortDir] = useState<'ascending' | 'descending'>('ascending');

  const sort = () => {
    const newDir = sortDir === 'ascending' ? 'descending' : 'ascending';
    setSortDir(newDir);
    setRows([...rows].sort((a, b) => 
      newDir === 'ascending' 
        ? a.name.localeCompare(b.name) 
        : b.name.localeCompare(a.name)
    ));
  };

  return (
    <table role="table">
      <thead>
        <tr>
          <th scope="col" aria-sort={sortDir}>
            <button onClick={sort}>Name</button>
          </th>
        </tr>
      </thead>
      <tbody>
        {rows.map((r) => <tr key={r.name}><td>{r.name}</td></tr>)}
      </tbody>
    </table>
  );
};
```

---

## 14. TREE VIEW

### ARIA Specification

| Proprietà     | Valore  | Obbligatorio |
| ------------- | ------- | ------------ |
| role          | tree    | ✅            |
| role          | treeitem| ✅            |
| role          | group   | ✅ (nested)  |
| aria-expanded | boolean | ✅            |

### Implementazione

```typescript
import React, { useState } from 'react';

export const Tree: React.FC = () => {
  const [open, setOpen] = useState(false);

  return (
    <ul role="tree">
      <li role="treeitem" aria-expanded={open}>
        <button onClick={() => setOpen(!open)}>Folder</button>
        {open && (
          <ul role="group">
            <li role="treeitem">File 1</li>
            <li role="treeitem">File 2</li>
          </ul>
        )}
      </li>
    </ul>
  );
};
```

---

## 15. CAROUSEL

### ARIA Specification

| Proprietà            | Valore     | Obbligatorio |
| -------------------- | ---------- | ------------ |
| aria-roledescription | "carousel" | ✅            |
| aria-label           | slide info | ✅            |
| aria-live            | polite     | ⚠️           |

### Implementazione

```typescript
import React, { useState } from 'react';

export const Carousel: React.FC = () => {
  const slides = ['Slide A', 'Slide B', 'Slide C'];
  const [index, setIndex] = useState(0);

  return (
    <div aria-roledescription="carousel" aria-label="Image carousel">
      <div aria-live="polite">
        <span aria-label={`Slide ${index + 1} of ${slides.length}`}>
          {slides[index]}
        </span>
      </div>
      <button onClick={() => setIndex((index - 1 + slides.length) % slides.length)}>
        Previous
      </button>
      <button onClick={() => setIndex((index + 1) % slides.length)}>
        Next
      </button>
    </div>
  );
};
```

---

## SKIP NAVIGATION LINK

### Obiettivo
Saltare direttamente al contenuto principale.

### Implementazione

```html
<a href="#main" class="skip-link">Skip to main content</a>
<main id="main">...</main>

<style>
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px;
  z-index: 100;
}
.skip-link:focus {
  top: 0;
}
</style>
```

---

## LANDMARK REGIONS

### HTML5 Landmarks

```html
<header>...</header>     <!-- banner -->
<nav>...</nav>           <!-- navigation -->
<main>...</main>         <!-- main -->
<aside>...</aside>       <!-- complementary -->
<footer>...</footer>     <!-- contentinfo -->
```

### ARIA Alternative

```html
<div role="banner">...</div>
<div role="navigation">...</div>
<div role="main">...</div>
<div role="complementary">...</div>
<div role="contentinfo">...</div>
```

---

## A11Y TOKEN SYSTEM

```typescript
export const a11yTokens = {
  focusRing: '2px solid #005fcc',
  focusOffset: '2px',
  minTouchTarget: '44px',
  contrastRatio: 4.5,
  largeTextContrastRatio: 3,
  reducedMotionDuration: '0ms',
};
```

---

## ACCESSIBILITY CHECKLIST

### UI
- [ ] Contrast ratio ≥ 4.5:1 (normal text)
- [ ] Contrast ratio ≥ 3:1 (large text)
- [ ] Touch target ≥ 44px × 44px
- [ ] Focus visible on all interactive elements
- [ ] No keyboard traps

### Semantica
- [ ] Heading hierarchy (h1 → h6)
- [ ] Landmark roles defined
- [ ] Labels associated with form controls
- [ ] Alt text for images

### ARIA
- [ ] No ARIA when native HTML works
- [ ] ARIA roles match behavior
- [ ] State attributes synchronized (aria-expanded, aria-checked, etc.)
- [ ] Live regions for dynamic content

### Keyboard
- [ ] Tab order logical
- [ ] All functionality accessible via keyboard
- [ ] Escape closes modals/menus
- [ ] Arrow keys for composite widgets

### Screen Reader
- [ ] Tested with NVDA/VoiceOver
- [ ] Meaningful announcements
- [ ] No duplicate announcements
- [ ] Form errors announced

---

## WCAG 2.1 AA COMPLIANCE MATRIX

| Criterio | Requisito | Verifica |
|----------|-----------|----------|
| 1.1.1 | Non-text Content | Alt text presente |
| 1.3.1 | Info and Relationships | Semantic HTML |
| 1.4.3 | Contrast (Minimum) | 4.5:1 ratio |
| 1.4.4 | Resize Text | 200% zoom |
| 2.1.1 | Keyboard | Full keyboard access |
| 2.1.2 | No Keyboard Trap | Escape works |
| 2.4.1 | Bypass Blocks | Skip links |
| 2.4.3 | Focus Order | Logical sequence |
| 2.4.4 | Link Purpose | Clear link text |
| 2.4.6 | Headings and Labels | Descriptive |
| 2.4.7 | Focus Visible | Visible focus |
| 3.1.1 | Language of Page | lang attribute |
| 3.2.1 | On Focus | No unexpected changes |
| 3.3.1 | Error Identification | Errors described |
| 3.3.2 | Labels or Instructions | Form labels |
| 4.1.1 | Parsing | Valid HTML |
| 4.1.2 | Name, Role, Value | ARIA correct |
