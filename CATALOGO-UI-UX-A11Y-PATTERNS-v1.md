# CATALOGO UI/UX - A11Y ACCESSIBILITY PATTERNS v1

> **Versione**: 1.0
> **Data**: 2026-01-27
> **Ambito**: Componenti accessibili WCAG 2.1 AA, ARIA patterns, keyboard navigation

---

§ 1. MODAL/DIALOG

§ ARIA SPECIFICATION

| Proprietà        | Valore         | Obbligatorio |
| ---------------- | -------------- | ------------ |
| role             | dialog         | ✅            |
| aria-modal       | true           | ✅            |
| aria-labelledby  | id titolo      | ✅            |
| aria-describedby | id descrizione | ⚠️           |

§ KEYBOARD NAVIGATION

| Tasto     | Azione         | Note        |
| --------- | -------------- | ----------- |
| Tab       | Next focus     | Focus trap  |
| Shift+Tab | Previous focus | Focus trap  |
| Escape    | Close dialog   | Sempre      |
| Enter     | Attiva         | Button/link |

§ FOCUS MANAGEMENT

| Evento | Comportamento        |
| ------ | -------------------- |
| Open   | Focus primo elemento |
| Close  | Restore focus        |
| Trap   | Focus limitato       |

§ IMPLEMENTAZIONE

typescript
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

§ TESTING CHECKLIST

| Test       | Metodo | Expected        |
| ---------- | ------ | --------------- |
| Focus trap | Tab    | Resta nel modal |
| Escape     | ESC    | Chiude          |
| SR         | NVDA   | Annuncia dialog |

---

§ 2. DROPDOWN MENU

§ ARIA SPECIFICATION

| Proprietà     | Valore  | Obbligatorio |
| ------------- | ------- | ------------ |
| role          | menu    | ✅            |
| aria-haspopup | true    | ✅            |
| aria-expanded | boolean | ✅            |

§ KEYBOARD NAVIGATION

| Tasto     | Azione    |
| --------- | --------- |
| Enter     | Open      |
| ArrowDown | Next item |
| ArrowUp   | Prev item |
| Escape    | Close     |

§ IMPLEMENTAZIONE

typescript
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

---

§ 3. TABS

§ ARIA SPECIFICATION

| Proprietà     | Valore   | Obbligatorio |
| ------------- | -------- | ------------ |
| role          | tablist  | ✅            |
| role          | tab      | ✅            |
| role          | tabpanel | ✅            |
| aria-selected | boolean  | ✅            |

§ KEYBOARD NAVIGATION

| Tasto      | Azione   |
| ---------- | -------- |
| ArrowRight | Next tab |
| ArrowLeft  | Prev tab |
| Enter      | Activate |

§ IMPLEMENTAZIONE

typescript
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

---

§ 4. ACCORDION

§ ARIA SPECIFICATION

| Proprietà     | Valore   | Obbligatorio |
| ------------- | -------- | ------------ |
| aria-expanded | boolean  | ✅            |
| aria-controls | id panel | ✅            |

§ KEYBOARD NAVIGATION

| Tasto | Azione |
| ----- | ------ |
| Enter | Toggle |
| Space | Toggle |

§ IMPLEMENTAZIONE

typescript
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

---

§ 5. TOOLTIP

§ ARIA SPECIFICATION

| Proprietà        | Valore     | Obbligatorio |
| ---------------- | ---------- | ------------ |
| role             | tooltip    | ✅            |
| aria-describedby | id tooltip | ✅            |

§ IMPLEMENTAZIONE

typescript
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

---

§ 6. TOAST/NOTIFICATION

§ ARIA SPECIFICATION

| Proprietà | Valore | Obbligatorio |
| --------- | ------ | ------------ |
| role      | status | ✅            |
| aria-live | polite | ✅            |

§ IMPLEMENTAZIONE

typescript
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

---

§ 7. AUTOCOMPLETE/COMBOBOX

§ ARIA SPECIFICATION

| Proprietà          | Valore  | Obbligatorio |
| ------------------ | ------- | ------------ |
| role               | combobox| ✅            |
| aria-expanded      | boolean | ✅            |
| aria-autocomplete  | list    | ✅            |
| aria-activedescendant | id   | ⚠️           |

§ IMPLEMENTAZIONE

typescript
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

---

§ 8. DATE PICKER

§ ARIA SPECIFICATION

| Proprietà  | Valore          | Obbligatorio |
| ---------- | --------------- | ------------ |
| aria-label | "Select date"   | ✅            |
| role       | grid (calendar) | ✅            |

§ IMPLEMENTAZIONE

typescript
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

---

§ 9. SLIDER/RANGE

§ ARIA SPECIFICATION

| Proprietà      | Valore | Obbligatorio |
| -------------- | ------ | ------------ |
| role           | slider | ✅            |
| aria-valuemin  | number | ✅            |
| aria-valuemax  | number | ✅            |
| aria-valuenow  | number | ✅            |
| aria-valuetext | string | ⚠️           |

§ IMPLEMENTAZIONE

typescript
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

---

§ 10. TOGGLE/SWITCH

§ ARIA SPECIFICATION

| Proprietà    | Valore  | Obbligatorio |
| ------------ | ------- | ------------ |
| role         | switch  | ✅            |
| aria-checked | boolean | ✅            |

§ IMPLEMENTAZIONE

typescript
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

---

§ 11. BREADCRUMB

§ ARIA SPECIFICATION

| Proprietà    | Valore      | Obbligatorio |
| ------------ | ----------- | ------------ |
| aria-label   | "Breadcrumb"| ✅            |
| aria-current | "page"      | ✅ (last)    |

§ IMPLEMENTAZIONE

typescript
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

---

§ 12. PAGINATION

§ ARIA SPECIFICATION

| Proprietà  | Valore       | Obbligatorio |
| ---------- | ------------ | ------------ |
| aria-label | "Pagination" | ✅            |
| aria-current | "page"     | ✅ (active)  |

§ IMPLEMENTAZIONE

typescript
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

---

§ 13. DATA TABLE (SORTABLE)

§ ARIA SPECIFICATION

| Proprietà   | Valore               | Obbligatorio |
| ----------- | -------------------- | ------------ |
| role        | table                | ✅            |
| aria-sort   | ascending/descending | ✅ (sortable)|
| scope       | col/row              | ✅            |

§ IMPLEMENTAZIONE

typescript
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

---

§ 14. TREE VIEW

§ ARIA SPECIFICATION

| Proprietà     | Valore  | Obbligatorio |
| ------------- | ------- | ------------ |
| role          | tree    | ✅            |
| role          | treeitem| ✅            |
| role          | group   | ✅ (nested)  |
| aria-expanded | boolean | ✅            |

§ IMPLEMENTAZIONE

typescript
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

---

§ 15. CAROUSEL

§ ARIA SPECIFICATION

| Proprietà            | Valore     | Obbligatorio |
| -------------------- | ---------- | ------------ |
| aria-roledescription | "carousel" | ✅            |
| aria-label           | slide info | ✅            |
| aria-live            | polite     | ⚠️           |

§ IMPLEMENTAZIONE

typescript
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

---

§ SKIP NAVIGATION LINK

§ OBIETTIVO
Saltare direttamente al contenuto principale.

§ IMPLEMENTAZIONE

html
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

---

§ LANDMARK REGIONS

§ HTML5 LANDMARKS

html
<header>...</header>     <!-- banner -->
<nav>...</nav>           <!-- navigation -->
<main>...</main>         <!-- main -->
<aside>...</aside>       <!-- complementary -->
<footer>...</footer>     <!-- contentinfo -->

§ ARIA ALTERNATIVE

html
<div role="banner">...</div>
<div role="navigation">...</div>
<div role="main">...</div>
<div role="complementary">...</div>
<div role="contentinfo">...</div>

---

§ A11Y TOKEN SYSTEM

typescript
export const a11yTokens = {
  focusRing: '2px solid #005fcc',
  focusOffset: '2px',
  minTouchTarget: '44px',
  contrastRatio: 4.5,
  largeTextContrastRatio: 3,
  reducedMotionDuration: '0ms',
};

---

§ ACCESSIBILITY CHECKLIST

§ UI
- [ ] Contrast ratio ≥ 4.5:1 (normal text)
- [ ] Contrast ratio ≥ 3:1 (large text)
- [ ] Touch target ≥ 44px × 44px
- [ ] Focus visible on all interactive elements
- [ ] No keyboard traps

§ SEMANTICA
- [ ] Heading hierarchy (h1 → h6)
- [ ] Landmark roles defined
- [ ] Labels associated with form controls
- [ ] Alt text for images

§ ARIA
- [ ] No ARIA when native HTML works
- [ ] ARIA roles match behavior
- [ ] State attributes synchronized (aria-expanded, aria-checked, etc.)
- [ ] Live regions for dynamic content

§ KEYBOARD
- [ ] Tab order logical
- [ ] All functionality accessible via keyboard
- [ ] Escape closes modals/menus
- [ ] Arrow keys for composite widgets

§ SCREEN READER
- [ ] Tested with NVDA/VoiceOver
- [ ] Meaningful announcements
- [ ] No duplicate announcements
- [ ] Form errors announced

---

§ WCAG 2.1 AA COMPLIANCE MATRIX

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

8. WCAG 2.1 AA COMPLETE CHECKLIST
Perceivable (Principle 1)

1.1.1 Non-text Content (Level A)
✅ Status: Required
Test Method: Audit all images, icons, charts, and form image buttons. Decorative images must have null alt (alt="") or be hidden from assistive tech (e.g., aria-hidden="true"). Informative images must have concise, descriptive alt text. Complex images (charts, diagrams) require long descriptions in text nearby or linked via aria-describedby.
Code Example (Next.js Image):

tsx
Copia
Scarica
import Image from 'next/image';
import React from 'react';

interface AccessibleImageProps {
  src: string;
  alt: string;
  isDecorative?: boolean;
  longDescId?: string;
}

export const AccessibleImage: React.FC<AccessibleImageProps> = ({
  src,
  alt,
  isDecorative = false,
  longDescId
}) => {
  // Decorative images should have empty alt
  const altText = isDecorative ? '' : alt;

  return (
    <>
      <Image
        src={src}
        alt={altText}
        aria-describedby={longDescId}
        width={300}
        height={200}
      />
      {longDescId && (
        <div id={longDescId} className="sr-only">
          {/* Detailed description of complex image */}
          Detailed chart showing quarterly revenue growth...
        </div>
      )}
    </>
  );
};

1.2.1 Audio-only and Video-only (Prerecorded) (Level A)
✅ Status: Required
Test Method: For audio-only content, provide a transcript. For video-only (no audio) content, provide an audio description or text alternative describing the key visual information.

1.2.2 Captions (Prerecorded) (Level A)
✅ Status: Required
Test Method: All pre-recorded video with audio must have accurate, synchronized captions. Test by playing video and verifying captions match dialogue and important sounds.
Code Example:

tsx
Copia
Scarica
'use client';
import React from 'react';

export const VideoPlayer: React.FC = () => {
  return (
    <div className="video-container">
      <video controls aria-label="Demo of accessible form patterns">
        <source src="/demo.mp4" type="video/mp4" />
        <track
          src="/captions_en.vtt"
          kind="captions"
          srcLang="en"
          label="English"
          default
        />
        <track
          src="/descriptions_en.vtt"
          kind="descriptions"
          srcLang="en"
          label="English Descriptions"
        />
        {/* Fallback transcript */}
        <p>
          <a href="/transcript.txt">View video transcript</a>
        </p>
      </video>
    </div>
  );
};

1.2.3 Audio Description or Media Alternative (Prerecorded) (Level A)
✅ Status: Required
Test Method: Provide an audio description track for pre-recorded video where visual information is not conveyed by the main audio, OR provide a full text alternative describing all visual and auditory content.

1.2.4 Captions (Live) (Level AA)
✅ Status: Required
Test Method: Live audio content (webinars, streams) must have real-time captions. Use a certified live captioning service.

1.2.5 Audio Description (Prerecorded) (Level AA)
✅ Status: Required
Test Method: For pre-recorded video, provide an extended audio description where pauses in dialogue are insufficient to convey the full visual story.

1.3.1 Info and Relationships (Level A)
✅ Status: Required
Test Method: Use semantic HTML (<header>, <nav>, <main>, <section>, <article>, <aside>, <footer>, <h1>-<h6>, <ul>, <ol>, <li>, <table>, <th>, <caption>, <figure>, <figcaption>). Programmatically associate labels with form controls (<label for="...">, aria-labelledby). Use ARIA landmarks (role="banner", role="navigation") where appropriate but prioritize native semantics. Verify with browser developer tools' accessibility inspector.

1.3.2 Meaningful Sequence (Level A)
✅ Status: Required
Test Method: Navigate using only the keyboard (Tab, Shift+Tab). The focus order must follow a logical, meaningful sequence matching the visual layout. Use CSS (order, flex-direction) for visual only, not to change reading order. Verify DOM order matches visual flow.

1.3.3 Sensory Characteristics (Level A)
✅ Status: Required
Test Method: Instructions must not rely solely on sensory characteristics like shape, color, size, or visual location. For example, instead of "Click the green button," use "Click the green 'Submit' button."

1.3.4 Orientation (Level AA)
✅ Status: Required
Test Method: Content must not restrict its view and operation to a single display orientation (portrait or landscape) unless essential (e.g., a piano app). Use CSS @media (orientation: portrait/landscape) to ensure content adapts.

1.3.5 Identify Input Purpose (Level AA)
✅ Status: Required
Test Method: For form fields collecting user information (name, email, address, phone), use appropriate autocomplete attribute values (name, email, street-address, tel). This assists users with cognitive disabilities and autofill tools.
Code Example:

tsx
Copia
Scarica
import React from 'react';

export const IdentityInput: React.FC = () => {
  return (
    <form>
      <div>
        <label htmlFor="full-name">Full Name</label>
        <input
          type="text"
          id="full-name"
          name="full-name"
          autoComplete="name"
          required
        />
      </div>
      <div>
        <label htmlFor="email">Email Address</label>
        <input
          type="email"
          id="email"
          name="email"
          autoComplete="email"
          required
        />
      </div>
      <div>
        <label htmlFor="current-password">Password</label>
        <input
          type="password"
          id="current-password"
          name="current-password"
          autoComplete="current-password"
          required
        />
      </div>
    </form>
  );
};

1.4.1 Use of Color (Level A)
✅ Status: Required
Test Method: Color must not be the only visual means of conveying information, indicating an action, prompting a response, or distinguishing a visual element. Test by viewing in grayscale. Ensure error states use both color and an icon/text label.

1.4.2 Audio Control (Level A)
✅ Status: Required
Test Method: If audio plays automatically for more than 3 seconds, provide a mechanism to pause, stop, or control the volume independently of the overall system volume.

1.4.3 Contrast (Minimum) (Level AA)
✅ Status: Required
Test Method: Text and images of text must have a contrast ratio of at least 4.5:1. Large text (≥18.66px normal weight or ≥14px bold) must have a ratio of at least 3:1. Use tools like axe DevTools, browser inspector contrast checker, or standalone color contrast analyzers. Logo text and inactive UI components are exempt.
Code Example:

css
Copia
Scarica
/* CSS Contrast Variables */
:root {
  --color-text: #222222; /* Not #777777 */
  --color-background: #ffffff;
  --color-primary: #0056b3; /* Sufficient contrast on white */
  --color-error: #a82a2a;
}

body {
  color: var(--color-text);
  background-color: var(--color-background);
}

.button-primary {
  background-color: var(--color-primary);
  color: white; /* Contrast ratio > 4.5:1 */
  padding: 0.75rem 1.5rem;
}

.text-large {
  font-size: 1.5rem; /* 24px - 3:1 ratio sufficient if bold */
  font-weight: 700;
}

1.4.4 Resize text (Level AA)
✅ Status: Required
Test Method: Text must be resizable up to 200% using browser zoom or text-only resize settings without loss of content or functionality. Test by using browser zoom (Ctrl/Cmd + +) to 200%. Ensure no horizontal scrolling, overlapping content, or truncated text.

1.4.5 Images of Text (Level AA)
✅ Status: Required
Test Method: Prefer actual text over images of text, unless the image of text is essential (e.g., a logo, a font specimen). If used, ensure the image of text meets contrast requirements (1.4.3) and is scalable.

1.4.10 Reflow (Level AA)
✅ Status: Required
Test Method: Content must be usable at 400% browser zoom (320 CSS pixels wide equivalent) without horizontal scrolling for vertical text, and without requiring two-dimensional scrolling for horizontal text. Avoid fixed-width containers; use relative units (%, rem, em) and CSS Flexbox/Grid.

1.4.11 Non-text Contrast (Level AA)
✅ Status: Required
Test Method: UI components (buttons, form control borders, focus indicators) and graphical objects (icons in charts) must have a contrast ratio of at least 3:1 against adjacent colors. Test focus rings, button borders, and icon states.

1.4.12 Text Spacing (Level AA)
✅ Status: Required
Test Method: Ensure no loss of content or functionality when user overrides text spacing via custom stylesheet: line height to at least 1.5 times font size, paragraph spacing to at least 2 times font size, letter spacing to at least 0.12 times font size, word spacing to at least 0.16 times font size.
Code Example:

css
Copia
Scarica
/* Base styles that support user overrides */
p, li, h1, h2, h3, h4, h5, h6 {
  line-height: 1.5;
  letter-spacing: 0.012em;
  word-spacing: 0.016em;
}

p + p {
  margin-top: 2em;
}

/* Ensure containers can expand */
.button, .card {
  min-height: 44px; /* Accounts for increased line-height */
}

1.4.13 Content on Hover or Focus (Level AA)
✅ Status: Required
Test Method: For additional content (tooltips, sub-menus) shown on hover or keyboard focus: 1) It must be dismissible (e.g., moving pointer away, pressing Escape), 2) It must be hoverable (pointer can move to the content), and 3) It must remain visible until the hover/focus trigger is removed or user dismisses it.

Operable (Principle 2)

2.1.1 Keyboard (Level A)
✅ Status: Required
Test Method: All functionality must be operable through a keyboard interface without requiring specific timings for individual keystrokes. Test using Tab, Shift+Tab, Enter, Space, Arrow keys, and Escape. No keyboard traps.

2.1.2 No Keyboard Trap (Level A)
✅ Status: Required
Test Method: If keyboard focus can be moved to a component (e.g., a modal, a custom widget), it must be possible to move focus away using only the keyboard (usually Tab, Shift+Tab, or Escape). There must be no "keyboard trap."

2.1.4 Character Key Shortcuts (Level A)
✅ Status: Required
Test Method: If a single character key shortcut is implemented (e.g., "S" to search), it must be able to be turned off, remapped, or only active when the relevant component (e.g., a search box) has focus.

2.2.1 Timing Adjustable (Level A)
✅ Status: Required
Test Method: For any time limit (e.g., session timeout, reading timer), provide a way to turn off, adjust, or extend it (minimum 10x extension). Real-time events (auction bids) and essential time limits (parking meter app) are exceptions.

2.2.2 Pause, Stop, Hide (Level A)
✅ Status: Required
Test Method: For moving, blinking, scrolling, or auto-updating information that starts automatically, lasts more than 5 seconds, and is presented in parallel with other content, provide a mechanism for the user to pause, stop, hide, or control the update frequency. Carousels, auto-advancing banners, and news tickers must have pause controls.

2.3.1 Three Flashes or Below Threshold (Level A)
✅ Status: Required
Test Method: Web pages must not contain anything that flashes more than three times in any one-second period, or the flash is below the general flash and red flash thresholds. Use tools to measure flash rate and area.

2.4.1 Bypass Blocks (Level A)
✅ Status: Required
Test Method: Provide a "Skip to Main Content" link (or multiple skip links) at the top of the page, visible on keyboard focus, allowing keyboard users to bypass repetitive content (navigation).

2.4.2 Page Titled (Level A)
✅ Status: Required
Test Method: Each web page must have a descriptive, unique <title> element that identifies its primary purpose and, when applicable, its relation to the larger site. Use the format "Primary Purpose - Site Name". Test in browser tab.
Code Example (Next.js):

tsx
Copia
Scarica
// app/page.tsx or app/layout.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Dashboard Overview - Accessible Platform v2',
  description: 'User dashboard showing key metrics and tasks.',
};

2.4.3 Focus Order (Level A)
✅ Status: Required
Test Method: Navigate sequentially (Tab) through the page. The focus order must be logical and preserve meaning and operability. Use tabindex="0" sparingly; avoid tabindex values greater than 0.

2.4.4 Link Purpose (In Context) (Level A)
✅ Status: Required
Test Method: The purpose of each link must be determinable from its link text alone, or from its link text combined with its programmatically determined link context (e.g., via aria-label, aria-labelledby, or preceding heading/sentence). Avoid "click here" or "read more".

2.4.5 Multiple Ways (Level AA)
✅ Status: Required
Test Method: Provide multiple ways to locate a web page within a set of pages (site), such as site search, table of contents, site map, or comprehensive navigation.

2.4.6 Headings and Labels (Level AA)
✅ Status: Required
Test Method: Headings and form labels must be descriptive and clear, accurately describing the topic or purpose of the content/control they relate to.

2.4.7 Focus Visible (Level AA)
✅ Status: Required
Test Method: Any keyboard-operable UI must have a visible focus indicator. The default browser focus ring must not be removed (outline: none) without providing a high-contrast, clearly distinguishable custom focus style.
Code Example:

css
Copia
Scarica
/* Never do this: */
*:focus {
  outline: none;
}

/* Correct approach: */
*:focus {
  outline: 3px solid #0056b3; /* High contrast color */
  outline-offset: 2px;
  border-radius: 1px;
}

/* For custom components, ensure contrast */
.button:focus-visible {
  box-shadow: 0 0 0 3px #ffffff, 0 0 0 6px #0056b3;
}

2.5.1 Pointer Gestures (Level A)
✅ Status: Required
Test Method: All functionality that uses multipoint or path-based gestures (pinch-to-zoom, swiping) must have a single-pointer alternative (e.g., +/- buttons, arrow buttons).

2.5.2 Pointer Cancellation (Level A)
✅ Status: Required
Test Method: For functionality operated with a single pointer (click, tap, long press), the down-event (e.g., onMouseDown, onPointerDown) must not execute the function. The action should be triggered on the up-event (onClick, onPointerUp), and there must be a mechanism to abort (moving pointer away) or undo (after triggering).

2.5.3 Label in Name (Level A)
✅ Status: Required
Test Method: For UI components with a visible label (text, icon+text), the accessible name (from aria-label, aria-labelledby, or default text content) must contain the visible label text. Test using browser accessibility inspector to compare "Name" with visible text.

2.5.4 Motion Actuation (Level A)
✅ Status: Required
Test Method: Functionality triggered by device motion (shake, tilt) or user gesture (pan) must be able to be turned off, and an alternative interface (button) must be provided to achieve the same function.

2.5.5 Target Size (Enhanced) (Level AAA)
⚠️ Status: Advisory for AA (Required for AAA)
Test Method: Touch targets (buttons, links, form controls) should be at least 44 by 44 CSS pixels. If a smaller target is used, ensure equivalent spaced padding brings the effective touch area to 44px. Test on touch devices.

Understandable (Principle 3)

3.1.1 Language of Page (Level A)
✅ Status: Required
Test Method: The default human language of each page must be programmatically set using the lang attribute on the <html> element (e.g., <html lang="en">).

3.1.2 Language of Parts (Level AA)
✅ Status: Required
Test Method: For passages or phrases in a language different from the page's default, identify the language using the lang attribute on a containing element (e.g., <span lang="fr">Bonjour</span>).

3.2.1 On Focus (Level A)
✅ Status: Required
Test Method: When any component receives focus, it must not automatically trigger a change of context (e.g., submitting a form, opening a new window, shifting focus elsewhere). Focus events should prepare for user input (e.g., expand a combo box list) but not change context without user initiation.

3.2.2 On Input (Level A)
✅ Status: Required
Test Method: Changing the setting of any user interface component (select, checkbox, radio button) must not automatically cause a change of context unless the user has been advised of the behavior before using the component (e.g., a label stating "Selecting an option will submit the form").

3.2.3 Consistent Navigation (Level AA)
✅ Status: Required
Test Method: Navigational mechanisms (menus, sidebars, skip links) that are repeated across multiple pages must appear in the same relative order each time they appear, unless a change is initiated by the user.

3.2.4 Consistent Identification (Level AA)
✅ Status: Required
Test Method: UI components with the same functionality across multiple pages must be identified consistently (same accessible name, same icon+text combination).

3.3.1 Error Identification (Level A)
✅ Status: Required
Test Method: If an input error is automatically detected, the error item must be identified and described to the user in text. The error must be programmatically associated with the field (aria-describedby, aria-errormessage).

3.3.2 Labels or Instructions (Level A)
✅ Status: Required
Test Method: Labels or clear instructions must be provided when content requires user input (e.g., "Date in MM/DD/YYYY format").

3.3.3 Error Suggestion (Level AA)
✅ Status: Required
Test Method: If an input error is detected and known suggestions for correction exist, provide the suggestions to the user (e.g., "The entered date is in the future. Did you mean [suggested date]?").

3.3.4 Error Prevention (Legal, Financial, Data) (Level AA)
✅ Status: Required
Test Method: For pages that cause legal commitments, financial transactions, or modify/user-controllable data, at least one of the following must be true: 1) Submissions are reversible, 2) Data is checked for errors and user can correct, or 3) A mechanism exists to review, confirm, and correct information before finalizing.

Robust (Principle 4)

4.1.1 Parsing (Level A)
✅ Status: Required
Test Method: HTML must be well-formed with no duplicate id attributes, no missing opening/closing tags (except where allowed), and elements must be nested according to their specifications. Validate with W3C validator and automated tools (axe).

4.1.2 Name, Role, Value (Level A)
✅ Status: Required
Test Method: For all custom UI components (custom buttons, sliders, dialogs), ensure the Name (accessible name), Role (role attribute), State/Property (aria-checked, aria-expanded, aria-pressed), and Value (aria-valuenow, aria-valuetext) are programmatically set and updated.

4.1.3 Status Messages (Level AA)
✅ Status: Required
Test Method: Status messages (e.g., form submission success, search results loaded, progress indicator updates) that do not receive focus must be presented to assistive technology via a live region (aria-live) or an aria-alertdialog role, without unnecessarily interrupting the user.

WCAG 2.1 AA Success Criteria Table
Success Criterion	Level	Priority	Test Method	Pass/Fail
1.1.1 Non-text Content	A	P1	Alt text audit, screen reader	Pass
1.2.1 Audio/Video-only	A	P1	Transcript/description check	Pass
1.2.2 Captions (Prerecorded)	A	P1	Play video, verify captions	Pass
1.2.3 Audio Description	A	P1	Check for description track or alternative	Pass
1.2.4 Captions (Live)	AA	P1	Verify live captioning service	Pass
1.2.5 Audio Description (Extended)	AA	P2	Check for extended description	Fail*
1.3.1 Info and Relationships	A	P1	Semantics check (HTML/ARIA)	Pass
1.3.2 Meaningful Sequence	A	P1	Keyboard navigation order	Pass
1.3.3 Sensory Characteristics	A	P1	Content review for shape/color/location	Pass
1.3.4 Orientation	AA	P2	CSS orientation media query test	Pass
1.3.5 Identify Input Purpose	AA	P2	Check autocomplete attributes	Pass
1.4.1 Use of Color	A	P1	Grayscale test, color dependency check	Pass
1.4.2 Audio Control	A	P1	Auto-play audio test	Pass
1.4.3 Contrast (Minimum)	AA	P1	Color contrast analyzer	Pass
1.4.4 Resize Text	AA	P1	200% browser zoom test	Pass
1.4.5 Images of Text	AA	P2	Audit images containing text	Pass
1.4.10 Reflow	AA	P1	400% zoom, no horizontal scroll	Pass
1.4.11 Non-text Contrast	AA	P1	UI component contrast check	Pass
1.4.12 Text Spacing	AA	P2	Apply user style overrides	Pass
1.4.13 Content on Hover/Focus	AA	P2	Test tooltip dismissal/hover	Pass
2.1.1 Keyboard	A	P1	Full keyboard operation test	Pass
2.1.2 No Keyboard Trap	A	P1	Tab through modals/custom widgets	Pass
2.1.4 Character Key Shortcuts	A	P2	Check for single-key shortcuts	Pass
2.2.1 Timing Adjustable	A	P1	Check time-limited content	Pass
2.2.2 Pause, Stop, Hide	A	P1	Test moving content controls	Pass
2.3.1 Three Flashes or Below	A	P1	Flash rate/area analysis	Pass
2.4.1 Bypass Blocks	A	P1	Verify skip link presence/function	Pass
2.4.2 Page Titled	A	P1	Check <title> element	Pass
2.4.3 Focus Order	A	P1	Sequential tab navigation	Pass
2.4.4 Link Purpose (In Context)	A	P1	Audit link text clarity	Pass
2.4.5 Multiple Ways	AA	P2	Check for sitemap/search/nav	Pass
2.4.6 Headings and Labels	AA	P1	Review heading/label clarity	Pass
2.4.7 Focus Visible	AA	P1	Check focus indicator on all controls	Pass
2.5.1 Pointer Gestures	A	P1	Check for single-pointer alternatives	Pass
2.5.2 Pointer Cancellation	A	P1	Test down-event vs up-event	Pass
2.5.3 Label in Name	A	P1	Compare visible vs accessible name	Pass
2.5.4 Motion Actuation	A	P2	Check motion-triggered functions	Pass
2.5.5 Target Size	AAA	P3	Measure touch target size	Fail*
3.1.1 Language of Page	A	P1	Check <html lang> attribute	Pass
3.1.2 Language of Parts	AA	P2	Check language changes marked	Pass
3.2.1 On Focus	A	P1	Test focus-triggered context changes	Pass
3.2.2 On Input	A	P1	Test input-triggered context changes	Pass
3.2.3 Consistent Navigation	AA	P2	Check navigation consistency	Pass
3.2.4 Consistent Identification	AA	P2	Check component identification	Pass
3.3.1 Error Identification	A	P1	Test error messaging & association	Pass
3.3.2 Labels/Instructions	A	P1	Review label/instruction clarity	Pass
3.3.3 Error Suggestion	AA	P1	Check for correction suggestions	Pass
3.3.4 Error Prevention	AA	P1	Review critical submission flows	Pass
4.1.1 Parsing	A	P1	HTML validation	Pass
4.1.2 Name, Role, Value	A	P1	Custom widget audit	Pass
4.1.3 Status Messages	AA	P1	Verify status announcements (live regions)	Pass

*P1=Critical, P2=High, P3=Medium. *Fail indicates common failures or AAA-level not required for AA conformance.

9. FORM ACCESSIBILITY DEEP DIVE
Label Association

Every form control must have a programmatically associated label. The <label> element with htmlFor is the strongest association.

tsx
Copia
Scarica
import React, { useState } from 'react';

// 1. Explicit label association
export const BasicLabel: React.FC = () => {
  return (
    <div>
      <label htmlFor="username-input">Username</label>
      <input type="text" id="username-input" name="username" />
    </div>
  );
};

// 2. Using aria-labelledby for complex labels or grouping
export const AriaLabelledByExample: React.FC = () => {
  return (
    <div>
      <div id="billing-heading">Billing Address</div>
      <div id="billing-instructions">Please enter your full billing address.</div>
      <input
        type="text"
        aria-labelledby="billing-heading billing-instructions"
        aria-describedby="billing-format"
      />
      <div id="billing-format" className="sr-only">
        Street, City, Postal Code
      </div>
    </div>
  );
};

// 3. aria-label for cases where visible label is not present
export const IconButtonExample: React.FC = () => {
  return (
    <button aria-label="Close notification panel">
      <span aria-hidden="true">×</span>
    </button>
  );
};
Error Messages & Validation

Errors must be announced to screen readers immediately and be programmatically associated with the field.

tsx
Copia
Scarica
'use client';
import React, { useState } from 'react';

interface FieldError {
  id: string;
  message: string;
}

export const AccessibleFormWithErrors: React.FC = () => {
  const [email, setEmail] = useState('');
  const [errors, setErrors] = useState<FieldError[]>([]);

  const validateEmail = (value: string): string | null => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!value) return 'Email is required.';
    if (!emailRegex.test(value)) return 'Please enter a valid email address.';
    return null;
  };

  const handleBlur = (e: React.FocusEvent<HTMLInputElement>) => {
    const { id, value } = e.target;
    const errorMessage = validateEmail(value);
    
    setErrors(prev => {
      const otherErrors = prev.filter(err => err.id !== id);
      if (errorMessage) {
        return [...otherErrors, { id, message: errorMessage }];
      }
      return otherErrors;
    });
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // Validation logic
  };

  const emailError = errors.find(err => err.id === 'email');
  const emailDescribedBy = [
    'email-help',
    emailError ? 'email-error' : undefined
  ].filter(Boolean).join(' ');

  return (
    <form onSubmit={handleSubmit} noValidate aria-labelledby="form-heading">
      <h2 id="form-heading">Account Signup</h2>
      
      <div>
        <label htmlFor="email">Email Address*</label>
        <input
          type="email"
          id="email"
          name="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          onBlur={handleBlur}
          aria-required="true"
          aria-invalid={!!emailError}
          aria-describedby={emailDescribedBy}
          autoComplete="email"
        />
        <div id="email-help" className="help-text">
          We'll never share your email.
        </div>
        {emailError && (
          <div
            id="email-error"
            role="alert"
            aria-live="polite"
            className="error-message"
          >
            {emailError.message}
          </div>
        )}
      </div>

      <button type="submit">Sign Up</button>
    </form>
  );
};
Required Fields & Visual Indicators

Always pair the visual asterisk (*) with aria-required="true". Do not rely on color alone.

tsx
Copia
Scarica
import React from 'react';

export const RequiredFieldExample: React.FC = () => {
  return (
    <div className="field-group">
      <label htmlFor="full-name">
        Full Name <span className="required-indicator" aria-hidden="true">*</span>
        <span className="sr-only">(required)</span>
      </label>
      <input
        type="text"
        id="full-name"
        aria-required="true"
        required // HTML5 required for native validation fallback
      />
    </div>
  );
};
Fieldset & Legend for Grouped Inputs

Use <fieldset> and <legend> to group related inputs (radio buttons, checkboxes).

tsx
Copia
Scarica
import React from 'react';

export const PaymentMethodFieldset: React.FC = () => {
  return (
    <fieldset>
      <legend>
        <h3>Payment Method</h3>
      </legend>
      
      <div className="radio-group">
        <input type="radio" id="credit-card" name="payment" value="card" />
        <label htmlFor="credit-card">Credit Card</label>
      </div>
      
      <div className="radio-group">
        <input type="radio" id="paypal" name="payment" value="paypal" />
        <label htmlFor="paypal">PayPal</label>
      </div>
      
      <div className="radio-group">
        <input type="radio" id="bank-transfer" name="payment" value="bank" />
        <label htmlFor="bank-transfer">Bank Transfer</label>
      </div>
    </fieldset>
  );
};
Autocomplete Attributes

Use correct autocomplete values for all relevant form fields to assist users with cognitive disabilities and autofill.

tsx
Copia
Scarica
import React from 'react';

export const AddressForm: React.FC = () => {
  return (
    <form>
      <div>
        <label htmlFor="address-line1">Street Address</label>
        <input
          type="text"
          id="address-line1"
          name="address-line1"
          autoComplete="street-address"
        />
      </div>
      <div>
        <label htmlFor="city">City</label>
        <input
          type="text"
          id="city"
          name="city"
          autoComplete="address-level2"
        />
      </div>
      <div>
        <label htmlFor="postal-code">Postal Code</label>
        <input
          type="text"
          id="postal-code"
          name="postal-code"
          autoComplete="postal-code"
        />
      </div>
      <div>
        <label htmlFor="country">Country</label>
        <input
          type="text"
          id="country"
          name="country"
          autoComplete="country-name"
        />
      </div>
      <div>
        <label htmlFor="phone">Phone Number</label>
        <input
          type="tel"
          id="phone"
          name="phone"
          autoComplete="tel"
        />
      </div>
    </form>
  );
};
Custom Select/Combobox Accessibility

Building a custom select requires implementing ARIA 1.2 combobox pattern.

tsx
Copia
Scarica
'use client';
import React, { useState, useRef, useEffect } from 'react';

interface Option {
  value: string;
  label: string;
}

interface AccessibleComboboxProps {
  id: string;
  label: string;
  options: Option[];
  defaultValue?: string;
}

export const AccessibleCombobox: React.FC<AccessibleComboboxProps> = ({
  id,
  label,
  options,
  defaultValue = ''
}) => {
  const [isOpen, setIsOpen] = useState(false);
  const [selectedValue, setSelectedValue] = useState(defaultValue);
  const [filteredOptions, setFilteredOptions] = useState(options);
  const [activeIndex, setActiveIndex] = useState(-1);
  const inputRef = useRef<HTMLInputElement>(null);
  const listboxRef = useRef<HTMLUListElement>(null);

  const selectedLabel = options.find(opt => opt.value === selectedValue)?.label || '';

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    setSelectedValue(value);
    
    const filtered = options.filter(opt =>
      opt.label.toLowerCase().includes(value.toLowerCase())
    );
    setFilteredOptions(filtered);
    setIsOpen(true);
    setActiveIndex(-1);
  };

  const handleOptionSelect = (option: Option) => {

/>

        {/* Modal container */}
        <div className="fixed inset-0 overflow-y-auto">
          <div className="flex min-h-full items-center justify-center p-4">
            <div
              ref={modalRef}
              className={`relative w-full ${sizeClasses[size]} transform rounded-lg bg-white shadow-xl transition-all`}
              role="dialog"
              aria-modal="true"
              aria-labelledby={titleId}
              aria-describedby={descriptionId}
            >
              {/* Header */}
              <div className="flex items-center justify-between border-b border-gray-200 p-6">
                <h2
                  id={titleId}
                  className="text-xl font-semibold text-gray-900"
                >
                  {title}
                </h2>
                {!hideCloseButton && (
                  <button
                    type="button"
                    className="ml-4 rounded-md text-gray-400 hover:text-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-500"
                    onClick={onClose}
                    aria-label={`Close ${title} dialog`}
                  >
                    <span className="sr-only">{closeButtonText}</span>
                    <span aria-hidden="true">&times;</span>
                  </button>
                )}
              </div>

              {/* Content */}
              <div className="p-6">
                <div id={descriptionId} className="sr-only">
                  {`${title} dialog content`}
                </div>
                {children}
              </div>

              {/* Footer */}
              <div className="border-t border-gray-200 bg-gray-50 px-6 py-4">
                <div className="flex justify-end gap-3">
                  {!hideCloseButton && (
                    <button
                      type="button"
                      className="rounded-md bg-white px-4 py-2 text-sm font-medium text-gray-700 shadow-sm hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
                      onClick={onClose}
                    >
                      {closeButtonText}
                    </button>
                  )}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      {/* Hidden live region for announcements */}
      <div
        id="a11y-announcer"
        aria-live="assertive"
        aria-atomic="true"
        className="sr-only"

### UI UX A11Y PATTERNS - Advanced Implementation Pattern #1

```typescript
// lib/ui-ux-a11y-patterns/pattern-1.ts
import { z } from "zod";

interface ServiceConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  batchSize: number;
  debug: boolean;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  batchSize: z.number().min(1).max(1000).default(50),
  debug: z.boolean().default(false),
});

export class UIUXA11YPATTERNSService1 {
  private config: ServiceConfig;
  private cache: Map<string, { data: unknown; expiresAt: number }> = new Map();
  private metrics = { operations: 0, errors: 0, avgDuration: 0 };

  constructor(config: Partial<ServiceConfig> = {}) {
    this.config = ConfigSchema.parse(config) as ServiceConfig;
  }

  async execute<TInput, TOutput>(
    operation: string,
    input: TInput,
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    this.metrics.operations++;
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);
        const result = await handler(input);
        clearTimeout(timeoutId);

        const duration = Date.now() - startTime;
        this.metrics.avgDuration =
          (this.metrics.avgDuration * (this.metrics.operations - 1) + duration) /
          this.metrics.operations;

        return {
          success: true,
          data: result,
          duration,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        this.metrics.errors++;

        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Operation failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }

        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  async executeBatch<TInput, TOutput>(
    operation: string,
    inputs: TInput[],
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput[]>> {
    const startTime = Date.now();
    const results: TOutput[] = [];
    const errors: string[] = [];

    for (let i = 0; i < inputs.length; i += this.config.batchSize) {
      const batch = inputs.slice(i, i + this.config.batchSize);
      const batchResults = await Promise.allSettled(
        batch.map((input) => this.execute(operation, input, handler))
      );

      for (const result of batchResults) {
        if (result.status === "fulfilled" && result.value.success && result.value.data) {
          results.push(result.value.data);
        } else if (result.status === "rejected") {
          errors.push(result.reason?.message || "Batch item failed");
        }
      }
    }

    return {
      success: errors.length === 0,
      data: results,
      error: errors.length > 0 ? errors.length + " items failed" : undefined,
      duration: Date.now() - startTime,
      retries: 0,
      timestamp: new Date(),
    };
  }

  getCachedOrFetch<T>(key: string, fetcher: () => Promise<T>, ttlMs: number = 60000): Promise<T> {
    const cached = this.cache.get(key);
    if (cached && Date.now() < cached.expiresAt) {
      return Promise.resolve(cached.data as T);
    }
    return fetcher().then((data) => {
      this.cache.set(key, { data, expiresAt: Date.now() + ttlMs });
      return data;
    });
  }

  getMetrics() {
    return {
      ...this.metrics,
      errorRate: this.metrics.operations > 0
        ? ((this.metrics.errors / this.metrics.operations) * 100).toFixed(2) + "%"
        : "0%",
      cacheSize: this.cache.size,
    };
  }
}
```

```typescript
// components/ui-ux-a11y-patterns/Manager1.tsx
"use client";

import { useState, useEffect, useCallback, useMemo } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import { Loader2, Plus, Search, RefreshCw, Trash2, Edit, ChevronLeft, ChevronRight } from "lucide-react";

interface Item {
  id: string;
  name: string;
  status: "active" | "inactive" | "pending";
  category: string;
  createdAt: string;
  updatedAt: string;
}

interface ManagerProps {
  initialItems?: Item[];
  apiEndpoint?: string;
  pageSize?: number;
}

export function Manager1({
  initialItems = [],
  apiEndpoint = "/api/items",
  pageSize = 10,
}: ManagerProps) {
  const [items, setItems] = useState<Item[]>(initialItems);
  const [loading, setLoading] = useState(false);
  const [search, setSearch] = useState("");
  const [statusFilter, setStatusFilter] = useState<string>("all");
  const [page, setPage] = useState(1);

  const fetchItems = useCallback(async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams({ page: String(page), limit: String(pageSize) });
      if (search) params.set("search", search);
      if (statusFilter !== "all") params.set("status", statusFilter);

      const response = await fetch(apiEndpoint + "?" + params);
      if (!response.ok) throw new Error("Failed to fetch");
      const data = await response.json();
      setItems(data.items || data.data || []);
    } catch (error) {
      console.error("Fetch error:", error);
    } finally {
      setLoading(false);
    }
  }, [apiEndpoint, page, pageSize, search, statusFilter]);

  useEffect(() => { fetchItems(); }, [fetchItems]);

  const filteredItems = useMemo(() => {
    return items.filter((item) => {
      const matchesSearch = !search || item.name.toLowerCase().includes(search.toLowerCase());
      const matchesStatus = statusFilter === "all" || item.status === statusFilter;
      return matchesSearch && matchesStatus;
    });
  }, [items, search, statusFilter]);

  const paginatedItems = filteredItems.slice((page - 1) * pageSize, page * pageSize);
  const totalPages = Math.ceil(filteredItems.length / pageSize);

  const handleDelete = useCallback(async (id: string) => {
    try {
      const response = await fetch(apiEndpoint + "/" + id, { method: "DELETE" });
      if (!response.ok) throw new Error("Delete failed");
      setItems((prev) => prev.filter((item) => item.id !== id));
    } catch (error) {
      console.error("Delete error:", error);
    }
  }, [apiEndpoint]);

  const statusColors: Record<string, string> = {
    active: "bg-green-100 text-green-800",
    inactive: "bg-gray-100 text-gray-800",
    pending: "bg-yellow-100 text-yellow-800",
  };

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle>Item Manager</CardTitle>
          <div className="flex items-center gap-2">
            <Button variant="outline" size="sm" onClick={fetchItems} disabled={loading}>
              <RefreshCw className={"h-4 w-4 " + (loading ? "animate-spin" : "")} />
            </Button>
            <Button size="sm"><Plus className="h-4 w-4 mr-1" /> Add New</Button>
          </div>
        </div>
        <div className="flex gap-2 mt-4">
          <div className="relative flex-1">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
            <Input
              placeholder="Search..."
              value={search}
              onChange={(e) => { setSearch(e.target.value); setPage(1); }}
              className="pl-9"
            />
          </div>
          <select
            value={statusFilter}
            onChange={(e) => { setStatusFilter(e.target.value); setPage(1); }}
            className="border rounded px-3 py-2 text-sm"
          >
            <option value="all">All Status</option>
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
            <option value="pending">Pending</option>
          </select>
        </div>
      </CardHeader>
      <CardContent>
        {loading ? (
          <div className="flex items-center justify-center py-12">
            <Loader2 className="h-6 w-6 animate-spin text-muted-foreground" />
          </div>
        ) : paginatedItems.length === 0 ? (
          <p className="text-center py-12 text-muted-foreground">No items found</p>
        ) : (
          <div className="space-y-2">
            {paginatedItems.map((item) => (
              <div key={item.id} className="flex items-center justify-between p-3 border rounded-lg hover:bg-muted/50 transition-colors">
                <div>
                  <p className="font-medium">{item.name}</p>
                  <p className="text-xs text-muted-foreground">
                    {item.category} - {new Date(item.createdAt).toLocaleDateString()}
                  </p>
                </div>
                <div className="flex items-center gap-2">
                  <Badge className={statusColors[item.status] || ""}>{item.status}</Badge>
                  <Button variant="ghost" size="sm"><Edit className="h-4 w-4" /></Button>
                  <Button variant="ghost" size="sm" onClick={() => handleDelete(item.id)}>
                    <Trash2 className="h-4 w-4 text-red-500" />
                  </Button>
                </div>
              </div>
            ))}
          </div>
        )}

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4 pt-4 border-t">
            <p className="text-sm text-muted-foreground">Page {page} of {totalPages}</p>
            <div className="flex gap-1">
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.max(1, p - 1))} disabled={page === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.min(totalPages, p + 1))} disabled={page === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```


### UI UX A11Y PATTERNS - Advanced Implementation Pattern #2

```typescript
// lib/ui-ux-a11y-patterns/pattern-2.ts
import { z } from "zod";

interface ServiceConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  batchSize: number;
  debug: boolean;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  batchSize: z.number().min(1).max(1000).default(50),
  debug: z.boolean().default(false),
});

export class UIUXA11YPATTERNSService2 {
  private config: ServiceConfig;
  private cache: Map<string, { data: unknown; expiresAt: number }> = new Map();
  private metrics = { operations: 0, errors: 0, avgDuration: 0 };

  constructor(config: Partial<ServiceConfig> = {}) {
    this.config = ConfigSchema.parse(config) as ServiceConfig;
  }

  async execute<TInput, TOutput>(
    operation: string,
    input: TInput,
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    this.metrics.operations++;
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);
        const result = await handler(input);
        clearTimeout(timeoutId);

        const duration = Date.now() - startTime;
        this.metrics.avgDuration =
          (this.metrics.avgDuration * (this.metrics.operations - 1) + duration) /
          this.metrics.operations;

        return {
          success: true,
          data: result,
          duration,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        this.metrics.errors++;

        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Operation failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }

        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  async executeBatch<TInput, TOutput>(
    operation: string,
    inputs: TInput[],
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput[]>> {
    const startTime = Date.now();
    const results: TOutput[] = [];
    const errors: string[] = [];

    for (let i = 0; i < inputs.length; i += this.config.batchSize) {
      const batch = inputs.slice(i, i + this.config.batchSize);
      const batchResults = await Promise.allSettled(
        batch.map((input) => this.execute(operation, input, handler))
      );

      for (const result of batchResults) {
        if (result.status === "fulfilled" && result.value.success && result.value.data) {
          results.push(result.value.data);
        } else if (result.status === "rejected") {
          errors.push(result.reason?.message || "Batch item failed");
        }
      }
    }

    return {
      success: errors.length === 0,
      data: results,
      error: errors.length > 0 ? errors.length + " items failed" : undefined,
      duration: Date.now() - startTime,
      retries: 0,
      timestamp: new Date(),
    };
  }

  getCachedOrFetch<T>(key: string, fetcher: () => Promise<T>, ttlMs: number = 60000): Promise<T> {
    const cached = this.cache.get(key);
    if (cached && Date.now() < cached.expiresAt) {
      return Promise.resolve(cached.data as T);
    }
    return fetcher().then((data) => {
      this.cache.set(key, { data, expiresAt: Date.now() + ttlMs });
      return data;
    });
  }

  getMetrics() {
    return {
      ...this.metrics,
      errorRate: this.metrics.operations > 0
        ? ((this.metrics.errors / this.metrics.operations) * 100).toFixed(2) + "%"
        : "0%",
      cacheSize: this.cache.size,
    };
  }
}
```

```typescript
// components/ui-ux-a11y-patterns/Manager2.tsx
"use client";

import { useState, useEffect, useCallback, useMemo } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import { Loader2, Plus, Search, RefreshCw, Trash2, Edit, ChevronLeft, ChevronRight } from "lucide-react";

interface Item {
  id: string;
  name: string;
  status: "active" | "inactive" | "pending";
  category: string;
  createdAt: string;
  updatedAt: string;
}

interface ManagerProps {
  initialItems?: Item[];
  apiEndpoint?: string;
  pageSize?: number;
}

export function Manager2({
  initialItems = [],
  apiEndpoint = "/api/items",
  pageSize = 10,
}: ManagerProps) {
  const [items, setItems] = useState<Item[]>(initialItems);
  const [loading, setLoading] = useState(false);
  const [search, setSearch] = useState("");
  const [statusFilter, setStatusFilter] = useState<string>("all");
  const [page, setPage] = useState(1);

  const fetchItems = useCallback(async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams({ page: String(page), limit: String(pageSize) });
      if (search) params.set("search", search);
      if (statusFilter !== "all") params.set("status", statusFilter);

      const response = await fetch(apiEndpoint + "?" + params);
      if (!response.ok) throw new Error("Failed to fetch");
      const data = await response.json();
      setItems(data.items || data.data || []);
    } catch (error) {
      console.error("Fetch error:", error);
    } finally {
      setLoading(false);
    }
  }, [apiEndpoint, page, pageSize, search, statusFilter]);

  useEffect(() => { fetchItems(); }, [fetchItems]);

  const filteredItems = useMemo(() => {
    return items.filter((item) => {
      const matchesSearch = !search || item.name.toLowerCase().includes(search.toLowerCase());
      const matchesStatus = statusFilter === "all" || item.status === statusFilter;
      return matchesSearch && matchesStatus;
    });
  }, [items, search, statusFilter]);

  const paginatedItems = filteredItems.slice((page - 1) * pageSize, page * pageSize);
  const totalPages = Math.ceil(filteredItems.length / pageSize);

  const handleDelete = useCallback(async (id: string) => {
    try {
      const response = await fetch(apiEndpoint + "/" + id, { method: "DELETE" });
      if (!response.ok) throw new Error("Delete failed");
      setItems((prev) => prev.filter((item) => item.id !== id));
    } catch (error) {
      console.error("Delete error:", error);
    }
  }, [apiEndpoint]);

  const statusColors: Record<string, string> = {
    active: "bg-green-100 text-green-800",
    inactive: "bg-gray-100 text-gray-800",
    pending: "bg-yellow-100 text-yellow-800",
  };

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle>Item Manager</CardTitle>
          <div className="flex items-center gap-2">
            <Button variant="outline" size="sm" onClick={fetchItems} disabled={loading}>
              <RefreshCw className={"h-4 w-4 " + (loading ? "animate-spin" : "")} />
            </Button>
            <Button size="sm"><Plus className="h-4 w-4 mr-1" /> Add New</Button>
          </div>
        </div>
        <div className="flex gap-2 mt-4">
          <div className="relative flex-1">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
            <Input
              placeholder="Search..."
              value={search}
              onChange={(e) => { setSearch(e.target.value); setPage(1); }}
              className="pl-9"
            />
          </div>
          <select
            value={statusFilter}
            onChange={(e) => { setStatusFilter(e.target.value); setPage(1); }}
            className="border rounded px-3 py-2 text-sm"
          >
            <option value="all">All Status</option>
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
            <option value="pending">Pending</option>
          </select>
        </div>
      </CardHeader>
      <CardContent>
        {loading ? (
          <div className="flex items-center justify-center py-12">
            <Loader2 className="h-6 w-6 animate-spin text-muted-foreground" />
          </div>
        ) : paginatedItems.length === 0 ? (
          <p className="text-center py-12 text-muted-foreground">No items found</p>
        ) : (
          <div className="space-y-2">
            {paginatedItems.map((item) => (
              <div key={item.id} className="flex items-center justify-between p-3 border rounded-lg hover:bg-muted/50 transition-colors">
                <div>
                  <p className="font-medium">{item.name}</p>
                  <p className="text-xs text-muted-foreground">
                    {item.category} - {new Date(item.createdAt).toLocaleDateString()}
                  </p>
                </div>
                <div className="flex items-center gap-2">
                  <Badge className={statusColors[item.status] || ""}>{item.status}</Badge>
                  <Button variant="ghost" size="sm"><Edit className="h-4 w-4" /></Button>
                  <Button variant="ghost" size="sm" onClick={() => handleDelete(item.id)}>
                    <Trash2 className="h-4 w-4 text-red-500" />
                  </Button>
                </div>
              </div>
            ))}
          </div>
        )}

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4 pt-4 border-t">
            <p className="text-sm text-muted-foreground">Page {page} of {totalPages}</p>
            <div className="flex gap-1">
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.max(1, p - 1))} disabled={page === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.min(totalPages, p + 1))} disabled={page === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```


### UI UX A11Y PATTERNS - Advanced Implementation Pattern #3

```typescript
// lib/ui-ux-a11y-patterns/pattern-3.ts
import { z } from "zod";

interface ServiceConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  batchSize: number;
  debug: boolean;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  batchSize: z.number().min(1).max(1000).default(50),
  debug: z.boolean().default(false),
});

export class UIUXA11YPATTERNSService3 {
  private config: ServiceConfig;
  private cache: Map<string, { data: unknown; expiresAt: number }> = new Map();
  private metrics = { operations: 0, errors: 0, avgDuration: 0 };

  constructor(config: Partial<ServiceConfig> = {}) {
    this.config = ConfigSchema.parse(config) as ServiceConfig;
  }

  async execute<TInput, TOutput>(
    operation: string,
    input: TInput,
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    this.metrics.operations++;
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);
        const result = await handler(input);
        clearTimeout(timeoutId);

        const duration = Date.now() - startTime;
        this.metrics.avgDuration =
          (this.metrics.avgDuration * (this.metrics.operations - 1) + duration) /
          this.metrics.operations;

        return {
          success: true,
          data: result,
          duration,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        this.metrics.errors++;

        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Operation failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }

        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  async executeBatch<TInput, TOutput>(
    operation: string,
    inputs: TInput[],
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput[]>> {
    const startTime = Date.now();
    const results: TOutput[] = [];
    const errors: string[] = [];

    for (let i = 0; i < inputs.length; i += this.config.batchSize) {
      const batch = inputs.slice(i, i + this.config.batchSize);
      const batchResults = await Promise.allSettled(
        batch.map((input) => this.execute(operation, input, handler))
      );

      for (const result of batchResults) {
        if (result.status === "fulfilled" && result.value.success && result.value.data) {
          results.push(result.value.data);
        } else if (result.status === "rejected") {
          errors.push(result.reason?.message || "Batch item failed");
        }
      }
    }

    return {
      success: errors.length === 0,
      data: results,
      error: errors.length > 0 ? errors.length + " items failed" : undefined,
      duration: Date.now() - startTime,
      retries: 0,
      timestamp: new Date(),
    };
  }

  getCachedOrFetch<T>(key: string, fetcher: () => Promise<T>, ttlMs: number = 60000): Promise<T> {
    const cached = this.cache.get(key);
    if (cached && Date.now() < cached.expiresAt) {
      return Promise.resolve(cached.data as T);
    }
    return fetcher().then((data) => {
      this.cache.set(key, { data, expiresAt: Date.now() + ttlMs });
      return data;
    });
  }

  getMetrics() {
    return {
      ...this.metrics,
      errorRate: this.metrics.operations > 0
        ? ((this.metrics.errors / this.metrics.operations) * 100).toFixed(2) + "%"
        : "0%",
      cacheSize: this.cache.size,
    };
  }
}
```

```typescript
// components/ui-ux-a11y-patterns/Manager3.tsx
"use client";

import { useState, useEffect, useCallback, useMemo } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import { Loader2, Plus, Search, RefreshCw, Trash2, Edit, ChevronLeft, ChevronRight } from "lucide-react";

interface Item {
  id: string;
  name: string;
  status: "active" | "inactive" | "pending";
  category: string;
  createdAt: string;
  updatedAt: string;
}

interface ManagerProps {
  initialItems?: Item[];
  apiEndpoint?: string;
  pageSize?: number;
}

export function Manager3({
  initialItems = [],
  apiEndpoint = "/api/items",
  pageSize = 10,
}: ManagerProps) {
  const [items, setItems] = useState<Item[]>(initialItems);
  const [loading, setLoading] = useState(false);
  const [search, setSearch] = useState("");
  const [statusFilter, setStatusFilter] = useState<string>("all");
  const [page, setPage] = useState(1);

  const fetchItems = useCallback(async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams({ page: String(page), limit: String(pageSize) });
      if (search) params.set("search", search);
      if (statusFilter !== "all") params.set("status", statusFilter);

      const response = await fetch(apiEndpoint + "?" + params);
      if (!response.ok) throw new Error("Failed to fetch");
      const data = await response.json();
      setItems(data.items || data.data || []);
    } catch (error) {
      console.error("Fetch error:", error);
    } finally {
      setLoading(false);
    }
  }, [apiEndpoint, page, pageSize, search, statusFilter]);

  useEffect(() => { fetchItems(); }, [fetchItems]);

  const filteredItems = useMemo(() => {
    return items.filter((item) => {
      const matchesSearch = !search || item.name.toLowerCase().includes(search.toLowerCase());
      const matchesStatus = statusFilter === "all" || item.status === statusFilter;
      return matchesSearch && matchesStatus;
    });
  }, [items, search, statusFilter]);

  const paginatedItems = filteredItems.slice((page - 1) * pageSize, page * pageSize);
  const totalPages = Math.ceil(filteredItems.length / pageSize);

  const handleDelete = useCallback(async (id: string) => {
    try {
      const response = await fetch(apiEndpoint + "/" + id, { method: "DELETE" });
      if (!response.ok) throw new Error("Delete failed");
      setItems((prev) => prev.filter((item) => item.id !== id));
    } catch (error) {
      console.error("Delete error:", error);
    }
  }, [apiEndpoint]);

  const statusColors: Record<string, string> = {
    active: "bg-green-100 text-green-800",
    inactive: "bg-gray-100 text-gray-800",
    pending: "bg-yellow-100 text-yellow-800",
  };

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle>Item Manager</CardTitle>
          <div className="flex items-center gap-2">
            <Button variant="outline" size="sm" onClick={fetchItems} disabled={loading}>
              <RefreshCw className={"h-4 w-4 " + (loading ? "animate-spin" : "")} />
            </Button>
            <Button size="sm"><Plus className="h-4 w-4 mr-1" /> Add New</Button>
          </div>
        </div>
        <div className="flex gap-2 mt-4">
          <div className="relative flex-1">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
            <Input
              placeholder="Search..."
              value={search}
              onChange={(e) => { setSearch(e.target.value); setPage(1); }}
              className="pl-9"
            />
          </div>
          <select
            value={statusFilter}
            onChange={(e) => { setStatusFilter(e.target.value); setPage(1); }}
            className="border rounded px-3 py-2 text-sm"
          >
            <option value="all">All Status</option>
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
            <option value="pending">Pending</option>
          </select>
        </div>
      </CardHeader>
      <CardContent>
        {loading ? (
          <div className="flex items-center justify-center py-12">
            <Loader2 className="h-6 w-6 animate-spin text-muted-foreground" />
          </div>
        ) : paginatedItems.length === 0 ? (
          <p className="text-center py-12 text-muted-foreground">No items found</p>
        ) : (
          <div className="space-y-2">
            {paginatedItems.map((item) => (
              <div key={item.id} className="flex items-center justify-between p-3 border rounded-lg hover:bg-muted/50 transition-colors">
                <div>
                  <p className="font-medium">{item.name}</p>
                  <p className="text-xs text-muted-foreground">
                    {item.category} - {new Date(item.createdAt).toLocaleDateString()}
                  </p>
                </div>
                <div className="flex items-center gap-2">
                  <Badge className={statusColors[item.status] || ""}>{item.status}</Badge>
                  <Button variant="ghost" size="sm"><Edit className="h-4 w-4" /></Button>
                  <Button variant="ghost" size="sm" onClick={() => handleDelete(item.id)}>
                    <Trash2 className="h-4 w-4 text-red-500" />
                  </Button>
                </div>
              </div>
            ))}
          </div>
        )}

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4 pt-4 border-t">
            <p className="text-sm text-muted-foreground">Page {page} of {totalPages}</p>
            <div className="flex gap-1">
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.max(1, p - 1))} disabled={page === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.min(totalPages, p + 1))} disabled={page === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```


### UI UX A11Y PATTERNS - Advanced Implementation Pattern #4

```typescript
// lib/ui-ux-a11y-patterns/pattern-4.ts
import { z } from "zod";

interface ServiceConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  batchSize: number;
  debug: boolean;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  batchSize: z.number().min(1).max(1000).default(50),
  debug: z.boolean().default(false),
});

export class UIUXA11YPATTERNSService4 {
  private config: ServiceConfig;
  private cache: Map<string, { data: unknown; expiresAt: number }> = new Map();
  private metrics = { operations: 0, errors: 0, avgDuration: 0 };

  constructor(config: Partial<ServiceConfig> = {}) {
    this.config = ConfigSchema.parse(config) as ServiceConfig;
  }

  async execute<TInput, TOutput>(
    operation: string,
    input: TInput,
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    this.metrics.operations++;
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);
        const result = await handler(input);
        clearTimeout(timeoutId);

        const duration = Date.now() - startTime;
        this.metrics.avgDuration =
          (this.metrics.avgDuration * (this.metrics.operations - 1) + duration) /
          this.metrics.operations;

        return {
          success: true,
          data: result,
          duration,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        this.metrics.errors++;

        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Operation failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }

        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  async executeBatch<TInput, TOutput>(
    operation: string,
    inputs: TInput[],
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput[]>> {
    const startTime = Date.now();
    const results: TOutput[] = [];
    const errors: string[] = [];

    for (let i = 0; i < inputs.length; i += this.config.batchSize) {
      const batch = inputs.slice(i, i + this.config.batchSize);
      const batchResults = await Promise.allSettled(
        batch.map((input) => this.execute(operation, input, handler))
      );

      for (const result of batchResults) {
        if (result.status === "fulfilled" && result.value.success && result.value.data) {
          results.push(result.value.data);
        } else if (result.status === "rejected") {
          errors.push(result.reason?.message || "Batch item failed");
        }
      }
    }

    return {
      success: errors.length === 0,
      data: results,
      error: errors.length > 0 ? errors.length + " items failed" : undefined,
      duration: Date.now() - startTime,
      retries: 0,
      timestamp: new Date(),
    };
  }

  getCachedOrFetch<T>(key: string, fetcher: () => Promise<T>, ttlMs: number = 60000): Promise<T> {
    const cached = this.cache.get(key);
    if (cached && Date.now() < cached.expiresAt) {
      return Promise.resolve(cached.data as T);
    }
    return fetcher().then((data) => {
      this.cache.set(key, { data, expiresAt: Date.now() + ttlMs });
      return data;
    });
  }

  getMetrics() {
    return {
      ...this.metrics,
      errorRate: this.metrics.operations > 0
        ? ((this.metrics.errors / this.metrics.operations) * 100).toFixed(2) + "%"
        : "0%",
      cacheSize: this.cache.size,
    };
  }
}
```

```typescript
// components/ui-ux-a11y-patterns/Manager4.tsx
"use client";

import { useState, useEffect, useCallback, useMemo } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import { Loader2, Plus, Search, RefreshCw, Trash2, Edit, ChevronLeft, ChevronRight } from "lucide-react";

interface Item {
  id: string;
  name: string;
  status: "active" | "inactive" | "pending";
  category: string;
  createdAt: string;
  updatedAt: string;
}

interface ManagerProps {
  initialItems?: Item[];
  apiEndpoint?: string;
  pageSize?: number;
}

export function Manager4({
  initialItems = [],
  apiEndpoint = "/api/items",
  pageSize = 10,
}: ManagerProps) {
  const [items, setItems] = useState<Item[]>(initialItems);
  const [loading, setLoading] = useState(false);
  const [search, setSearch] = useState("");
  const [statusFilter, setStatusFilter] = useState<string>("all");
  const [page, setPage] = useState(1);

  const fetchItems = useCallback(async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams({ page: String(page), limit: String(pageSize) });
      if (search) params.set("search", search);
      if (statusFilter !== "all") params.set("status", statusFilter);

      const response = await fetch(apiEndpoint + "?" + params);
      if (!response.ok) throw new Error("Failed to fetch");
      const data = await response.json();
      setItems(data.items || data.data || []);
    } catch (error) {
      console.error("Fetch error:", error);
    } finally {
      setLoading(false);
    }
  }, [apiEndpoint, page, pageSize, search, statusFilter]);

  useEffect(() => { fetchItems(); }, [fetchItems]);

  const filteredItems = useMemo(() => {
    return items.filter((item) => {
      const matchesSearch = !search || item.name.toLowerCase().includes(search.toLowerCase());
      const matchesStatus = statusFilter === "all" || item.status === statusFilter;
      return matchesSearch && matchesStatus;
    });
  }, [items, search, statusFilter]);

  const paginatedItems = filteredItems.slice((page - 1) * pageSize, page * pageSize);
  const totalPages = Math.ceil(filteredItems.length / pageSize);

  const handleDelete = useCallback(async (id: string) => {
    try {
      const response = await fetch(apiEndpoint + "/" + id, { method: "DELETE" });
      if (!response.ok) throw new Error("Delete failed");
      setItems((prev) => prev.filter((item) => item.id !== id));
    } catch (error) {
      console.error("Delete error:", error);
    }
  }, [apiEndpoint]);

  const statusColors: Record<string, string> = {
    active: "bg-green-100 text-green-800",
    inactive: "bg-gray-100 text-gray-800",
    pending: "bg-yellow-100 text-yellow-800",
  };

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle>Item Manager</CardTitle>
          <div className="flex items-center gap-2">
            <Button variant="outline" size="sm" onClick={fetchItems} disabled={loading}>
              <RefreshCw className={"h-4 w-4 " + (loading ? "animate-spin" : "")} />
            </Button>
            <Button size="sm"><Plus className="h-4 w-4 mr-1" /> Add New</Button>
          </div>
        </div>
        <div className="flex gap-2 mt-4">
          <div className="relative flex-1">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
            <Input
              placeholder="Search..."
              value={search}
              onChange={(e) => { setSearch(e.target.value); setPage(1); }}
              className="pl-9"
            />
          </div>
          <select
            value={statusFilter}
            onChange={(e) => { setStatusFilter(e.target.value); setPage(1); }}
            className="border rounded px-3 py-2 text-sm"
          >
            <option value="all">All Status</option>
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
            <option value="pending">Pending</option>
          </select>
        </div>
      </CardHeader>
      <CardContent>
        {loading ? (
          <div className="flex items-center justify-center py-12">
            <Loader2 className="h-6 w-6 animate-spin text-muted-foreground" />
          </div>
        ) : paginatedItems.length === 0 ? (
          <p className="text-center py-12 text-muted-foreground">No items found</p>
        ) : (
          <div className="space-y-2">
            {paginatedItems.map((item) => (
              <div key={item.id} className="flex items-center justify-between p-3 border rounded-lg hover:bg-muted/50 transition-colors">
                <div>
                  <p className="font-medium">{item.name}</p>
                  <p className="text-xs text-muted-foreground">
                    {item.category} - {new Date(item.createdAt).toLocaleDateString()}
                  </p>
                </div>
                <div className="flex items-center gap-2">
                  <Badge className={statusColors[item.status] || ""}>{item.status}</Badge>
                  <Button variant="ghost" size="sm"><Edit className="h-4 w-4" /></Button>
                  <Button variant="ghost" size="sm" onClick={() => handleDelete(item.id)}>
                    <Trash2 className="h-4 w-4 text-red-500" />
                  </Button>
                </div>
              </div>
            ))}
          </div>
        )}

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4 pt-4 border-t">
            <p className="text-sm text-muted-foreground">Page {page} of {totalPages}</p>
            <div className="flex gap-1">
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.max(1, p - 1))} disabled={page === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.min(totalPages, p + 1))} disabled={page === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```


### UI UX A11Y PATTERNS - Advanced Implementation Pattern #5

```typescript
// lib/ui-ux-a11y-patterns/pattern-5.ts
import { z } from "zod";

interface ServiceConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  batchSize: number;
  debug: boolean;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  batchSize: z.number().min(1).max(1000).default(50),
  debug: z.boolean().default(false),
});

export class UIUXA11YPATTERNSService5 {
  private config: ServiceConfig;
  private cache: Map<string, { data: unknown; expiresAt: number }> = new Map();
  private metrics = { operations: 0, errors: 0, avgDuration: 0 };

  constructor(config: Partial<ServiceConfig> = {}) {
    this.config = ConfigSchema.parse(config) as ServiceConfig;
  }

  async execute<TInput, TOutput>(
    operation: string,
    input: TInput,
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    this.metrics.operations++;
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);
        const result = await handler(input);
        clearTimeout(timeoutId);

        const duration = Date.now() - startTime;
        this.metrics.avgDuration =
          (this.metrics.avgDuration * (this.metrics.operations - 1) + duration) /
          this.metrics.operations;

        return {
          success: true,
          data: result,
          duration,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        this.metrics.errors++;

        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Operation failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }

        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  async executeBatch<TInput, TOutput>(
    operation: string,
    inputs: TInput[],
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput[]>> {
    const startTime = Date.now();
    const results: TOutput[] = [];
    const errors: string[] = [];

    for (let i = 0; i < inputs.length; i += this.config.batchSize) {
      const batch = inputs.slice(i, i + this.config.batchSize);
      const batchResults = await Promise.allSettled(
        batch.map((input) => this.execute(operation, input, handler))
      );

      for (const result of batchResults) {
        if (result.status === "fulfilled" && result.value.success && result.value.data) {
          results.push(result.value.data);
        } else if (result.status === "rejected") {
          errors.push(result.reason?.message || "Batch item failed");
        }
      }
    }

    return {
      success: errors.length === 0,
      data: results,
      error: errors.length > 0 ? errors.length + " items failed" : undefined,
      duration: Date.now() - startTime,
      retries: 0,
      timestamp: new Date(),
    };
  }

  getCachedOrFetch<T>(key: string, fetcher: () => Promise<T>, ttlMs: number = 60000): Promise<T> {
    const cached = this.cache.get(key);
    if (cached && Date.now() < cached.expiresAt) {
      return Promise.resolve(cached.data as T);
    }
    return fetcher().then((data) => {
      this.cache.set(key, { data, expiresAt: Date.now() + ttlMs });
      return data;
    });
  }

  getMetrics() {
    return {
      ...this.metrics,
      errorRate: this.metrics.operations > 0
        ? ((this.metrics.errors / this.metrics.operations) * 100).toFixed(2) + "%"
        : "0%",
      cacheSize: this.cache.size,
    };
  }
}
```

```typescript
// components/ui-ux-a11y-patterns/Manager5.tsx
"use client";

import { useState, useEffect, useCallback, useMemo } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import { Loader2, Plus, Search, RefreshCw, Trash2, Edit, ChevronLeft, ChevronRight } from "lucide-react";

interface Item {
  id: string;
  name: string;
  status: "active" | "inactive" | "pending";
  category: string;
  createdAt: string;
  updatedAt: string;
}

interface ManagerProps {
  initialItems?: Item[];
  apiEndpoint?: string;
  pageSize?: number;
}

export function Manager5({
  initialItems = [],
  apiEndpoint = "/api/items",
  pageSize = 10,
}: ManagerProps) {
  const [items, setItems] = useState<Item[]>(initialItems);
  const [loading, setLoading] = useState(false);
  const [search, setSearch] = useState("");
  const [statusFilter, setStatusFilter] = useState<string>("all");
  const [page, setPage] = useState(1);

  const fetchItems = useCallback(async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams({ page: String(page), limit: String(pageSize) });
      if (search) params.set("search", search);
      if (statusFilter !== "all") params.set("status", statusFilter);

      const response = await fetch(apiEndpoint + "?" + params);
      if (!response.ok) throw new Error("Failed to fetch");
      const data = await response.json();
      setItems(data.items || data.data || []);
    } catch (error) {
      console.error("Fetch error:", error);
    } finally {
      setLoading(false);
    }
  }, [apiEndpoint, page, pageSize, search, statusFilter]);

  useEffect(() => { fetchItems(); }, [fetchItems]);

  const filteredItems = useMemo(() => {
    return items.filter((item) => {
      const matchesSearch = !search || item.name.toLowerCase().includes(search.toLowerCase());
      const matchesStatus = statusFilter === "all" || item.status === statusFilter;
      return matchesSearch && matchesStatus;
    });
  }, [items, search, statusFilter]);

  const paginatedItems = filteredItems.slice((page - 1) * pageSize, page * pageSize);
  const totalPages = Math.ceil(filteredItems.length / pageSize);

  const handleDelete = useCallback(async (id: string) => {
    try {
      const response = await fetch(apiEndpoint + "/" + id, { method: "DELETE" });
      if (!response.ok) throw new Error("Delete failed");
      setItems((prev) => prev.filter((item) => item.id !== id));
    } catch (error) {
      console.error("Delete error:", error);
    }
  }, [apiEndpoint]);

  const statusColors: Record<string, string> = {
    active: "bg-green-100 text-green-800",
    inactive: "bg-gray-100 text-gray-800",
    pending: "bg-yellow-100 text-yellow-800",
  };

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle>Item Manager</CardTitle>
          <div className="flex items-center gap-2">
            <Button variant="outline" size="sm" onClick={fetchItems} disabled={loading}>
              <RefreshCw className={"h-4 w-4 " + (loading ? "animate-spin" : "")} />
            </Button>
            <Button size="sm"><Plus className="h-4 w-4 mr-1" /> Add New</Button>
          </div>
        </div>
        <div className="flex gap-2 mt-4">
          <div className="relative flex-1">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
            <Input
              placeholder="Search..."
              value={search}
              onChange={(e) => { setSearch(e.target.value); setPage(1); }}
              className="pl-9"
            />
          </div>
          <select
            value={statusFilter}
            onChange={(e) => { setStatusFilter(e.target.value); setPage(1); }}
            className="border rounded px-3 py-2 text-sm"
          >
            <option value="all">All Status</option>
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
            <option value="pending">Pending</option>
          </select>
        </div>
      </CardHeader>
      <CardContent>
        {loading ? (
          <div className="flex items-center justify-center py-12">
            <Loader2 className="h-6 w-6 animate-spin text-muted-foreground" />
          </div>
        ) : paginatedItems.length === 0 ? (
          <p className="text-center py-12 text-muted-foreground">No items found</p>
        ) : (
          <div className="space-y-2">
            {paginatedItems.map((item) => (
              <div key={item.id} className="flex items-center justify-between p-3 border rounded-lg hover:bg-muted/50 transition-colors">
                <div>
                  <p className="font-medium">{item.name}</p>
                  <p className="text-xs text-muted-foreground">
                    {item.category} - {new Date(item.createdAt).toLocaleDateString()}
                  </p>
                </div>
                <div className="flex items-center gap-2">
                  <Badge className={statusColors[item.status] || ""}>{item.status}</Badge>
                  <Button variant="ghost" size="sm"><Edit className="h-4 w-4" /></Button>
                  <Button variant="ghost" size="sm" onClick={() => handleDelete(item.id)}>
                    <Trash2 className="h-4 w-4 text-red-500" />
                  </Button>
                </div>
              </div>
            ))}
          </div>
        )}

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4 pt-4 border-t">
            <p className="text-sm text-muted-foreground">Page {page} of {totalPages}</p>
            <div className="flex gap-1">
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.max(1, p - 1))} disabled={page === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.min(totalPages, p + 1))} disabled={page === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```
