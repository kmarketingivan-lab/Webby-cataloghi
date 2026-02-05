# CATALOGO-WEBSITE-TEMPLATE-v1

> **Dominio**: Website/CMS dinamico con gestione contenuti
> **Stack**: Next.js 14, TypeScript, Prisma, tRPC
> **Versione**: 1.0
> **Data**: 2026-01-29

---

## 1. INDICE

| Sezione | Descrizione |
|---------|-------------|
| [1. Panoramica](#1-panoramica) | Obiettivi e architettura generale |
| [2. Schema Database](#2-schema-database) | Modelli Prisma per CMS |
| [3. Services Layer](#3-services-layer) | Business logic per pagine, menu, contatti, settings |
| [4. API Layer](#4-api-layer) | Router tRPC e validazioni Zod |
| [5. Componenti Website](#5-componenti-website) | Header, Footer, Navigation, Sezioni landing |
| [6. Pagine Pubbliche](#6-pagine-pubbliche) | Homepage, pagine dinamiche, contatti |
| [7. Pannello Admin](#7-pannello-admin) | CRUD pagine, menu, submissions, settings |
| [8. Testing](#8-testing) | Test del modulo website |

---

## 1. PANORAMICA

### 1.1 Obiettivo
Sito web dinamico con funzionalità CMS per gestione contenuti, menu, form contatti e impostazioni del sito.

### 1.2 Funzionalità Principali

| Funzionalità | Descrizione |
|--------------|-------------|
| **Gestione Pagine** | CRUD pagine con slug, status (DRAFT/PUBLISHED), SEO |
| **Menu Dinamici** | Menu gerarchici con drag & drop reordering |
| **Form Contatti** | Submissions con status, spam detection, email notifications |
| **Site Settings** | Configurazione globale sito (nome, descrizione, logo) |

### 1.3 Struttura File

| Layer | Path | Descrizione |
|-------|------|-------------|
| Database | `prisma/schema-website.prisma` | Schema CMS |
| Services | `src/server/services/` | Business logic |
| API | `src/server/trpc/routers/website.ts` | Endpoints tRPC |
| Validations | `src/lib/validations/website.ts` | Schema Zod |
| Components | `src/components/website/` | UI components |
| Pages (Public) | `src/app/(website)/` | Route pubbliche |
| Pages (Admin) | `src/app/admin/` | Pannello gestione |

---

## 2. SCHEMA DATABASE

### 2.1 File: `prisma/schema-website.prisma`

```prisma
// Schema database per CMS website
// Modelli: Page, MenuItem, ContactSubmission, SiteSetting
```

### 2.2 Modelli Principali

| Modello | Campi Chiave | Relazioni |
|---------|--------------|-----------|
| `Page` | id, title, slug, content, status, showInNav | - |
| `MenuItem` | id, label, url, location, order, parentId | self-relation (parent/children) |
| `ContactSubmission` | id, name, email, message, status | - |
| `SiteSetting` | key, value | - |

---

## 3. SERVICES LAYER

### 3.1 PageService

**File**: `src/server/services/page-service.ts`

| Metodo | Parametri | Return | Descrizione |
|--------|-----------|--------|-------------|
| `create` | `CreatePageInput` | `Page` | Crea nuova pagina |
| `update` | `pageId, UpdatePageInput` | `Page` | Aggiorna pagina |
| `delete` | `pageId` | `void` | Elimina pagina |
| `getById` | `id` | `Page \| null` | Trova per ID |
| `getBySlug` | `slug` | `Page \| null` | Trova per slug |
| `list` | `PageListParams?` | `PaginatedResult<Page>` | Lista paginata |
| `getPublished` | - | `Page[]` | Solo pubblicate |
| `getNavigation` | - | `Page[]` | Per menu navigazione |
| `publish` | `pageId` | `Page` | Pubblica pagina |
| `unpublish` | `pageId` | `Page` | Riporta a draft |
| `generateSlug` | `title` | `string` | Genera slug da titolo |
| `renderContent` | `markdown` | `string` | Render markdown → HTML |

```typescript
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

export class PageService {
  async create(data: CreatePageInput): Promise<Page> {
    return prisma.page.create({ data });
  }

  async update(pageId: string, data: UpdatePageInput): Promise<Page> {
    return prisma.page.update({ where: { id: pageId }, data });
  }

  async delete(pageId: string): Promise<void> {
    await prisma.page.delete({ where: { id: pageId } });
  }

  async getById(id: string): Promise<Page | null> {
    return prisma.page.findUnique({ where: { id } });
  }

  async getBySlug(slug: string): Promise<Page | null> {
    return prisma.page.findUnique({ where: { slug } });
  }

  async list(params?: PageListParams): Promise<PaginatedResult<Page>> {
    const pages = await prisma.page.findMany({...params });
    return { pages, totalCount: pages.length };
  }

  async getPublished(): Promise<Page[]> {
    return prisma.page.findMany({ where: { status: 'PUBLISHED' } });
  }

  async getNavigation(): Promise<Page[]> {
    return prisma.page.findMany({ where: { showInNav: true } });
  }

  async publish(pageId: string): Promise<Page> {
    return prisma.page.update({ where: { id: pageId }, data: { status: 'PUBLISHED' } });
  }

  async unpublish(pageId: string): Promise<Page> {
    return prisma.page.update({ where: { id: pageId }, data: { status: 'DRAFT' } });
  }

  async generateSlug(title: string): Promise<string> {
    // implementazione della generazione dello slug
  }

  async renderContent(markdown: string): Promise<string> {
    // implementazione della renderizzazione del contenuto
  }
}
```

### 3.2 MenuService

**File**: `src/server/services/menu-service.ts`

| Metodo | Parametri | Return | Descrizione |
|--------|-----------|--------|-------------|
| `getMenu` | `location` | `MenuItem[]` | Menu per posizione |
| `createItem` | `CreateMenuItemInput` | `MenuItem` | Crea voce menu |
| `updateItem` | `itemId, UpdateMenuItemInput` | `MenuItem` | Aggiorna voce |
| `deleteItem` | `itemId` | `void` | Elimina voce |
| `reorder` | `items[]` | `void` | Riordina voci |

```typescript
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

export class MenuService {
  async getMenu(location: MenuLocation): Promise<MenuItem[]> {
    return prisma.menuItem.findMany({ where: { location } });
  }

  async createItem(data: CreateMenuItemInput): Promise<MenuItem> {
    return prisma.menuItem.create({ data });
  }

  async updateItem(itemId: string, data: UpdateMenuItemInput): Promise<MenuItem> {
    return prisma.menuItem.update({ where: { id: itemId }, data });
  }

  async deleteItem(itemId: string): Promise<void> {
    await prisma.menuItem.delete({ where: { id: itemId } });
  }

  async reorder(items: { id: string; order: number; parentId?: string }[]): Promise<void> {
    // implementazione del reordering degli elementi del menu
  }
}
```

### 3.3 ContactService

**File**: `src/server/services/contact-service.ts`

| Metodo | Parametri | Return | Descrizione |
|--------|-----------|--------|-------------|
| `submit` | `ContactSubmissionInput` | `ContactSubmission` | Invia form |
| `list` | `SubmissionListParams?` | `PaginatedResult` | Lista submissions |
| `getById` | `id` | `ContactSubmission \| null` | Dettaglio |
| `updateStatus` | `id, status` | `ContactSubmission` | Cambia status |
| `delete` | `id` | `void` | Elimina |
| `markAsSpam` | `id` | `void` | Segna come spam |
| `sendConfirmationEmail` | `submission` | `void` | Email conferma utente |
| `sendNotificationEmail` | `submission` | `void` | Email notifica admin |

```typescript
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

export class ContactService {
  async submit(data: ContactSubmissionInput): Promise<ContactSubmission> {
    return prisma.contactSubmission.create({ data });
  }

  async list(params?: SubmissionListParams): Promise<PaginatedResult<ContactSubmission>> {
    const submissions = await prisma.contactSubmission.findMany({...params });
    return { submissions, totalCount: submissions.length };
  }

  async getById(id: string): Promise<ContactSubmission | null> {
    return prisma.contactSubmission.findUnique({ where: { id } });
  }

  async updateStatus(id: string, status: SubmissionStatus): Promise<ContactSubmission> {
    return prisma.contactSubmission.update({ where: { id }, data: { status } });
  }

  async delete(id: string): Promise<void> {
    await prisma.contactSubmission.delete({ where: { id } });
  }

  async markAsSpam(id: string): Promise<void> {
    await prisma.contactSubmission.update({ where: { id }, data: { status: 'SPAM' } });
  }

  async sendConfirmationEmail(submission: ContactSubmission): Promise<void> {
    // implementazione dell'invio dell'email di conferma
  }

  async sendNotificationEmail(submission: ContactSubmission): Promise<void> {
    // implementazione dell'invio dell'email di notifica
  }
}
```

### 3.4 SettingsService

**File**: `src/server/services/settings-service.ts`

| Metodo | Parametri | Return | Descrizione |
|--------|-----------|--------|-------------|
| `get<T>` | `key` | `T \| null` | Legge setting |
| `set<T>` | `key, value` | `void` | Scrive setting |
| `getAll` | - | `Record<string, any>` | Tutte le settings |
| `getSiteInfo` | - | `SiteInfo` | Info sito aggregate |
| `updateSiteInfo` | `Partial<SiteInfo>` | `SiteInfo` | Aggiorna info sito |

```typescript
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

export class SettingsService {
  async get<T>(key: string): Promise<T | null> {
    const setting = await prisma.siteSetting.findUnique({ where: { key } });
    return setting? setting.value : null;
  }

  async set<T>(key: string, value: T): Promise<void> {
    await prisma.siteSetting.upsert({ where: { key }, create: { key, value }, update: { value } });
  }

  async getAll(): Promise<Record<string, any>> {
    const settings = await prisma.siteSetting.findMany();
    return settings.reduce((acc, setting) => ({...acc, [setting.key]: setting.value }), {});
  }

  async getSiteInfo(): Promise<SiteInfo> {
    // implementazione del recupero delle informazioni del sito
  }

  async updateSiteInfo(data: Partial<SiteInfo>): Promise<SiteInfo> {
    // implementazione dell'aggiornamento delle informazioni del sito
  }
}
```

---

## 4. API LAYER

### 4.1 Router tRPC

**File**: `src/server/trpc/routers/website.ts`

| Endpoint | Tipo | Input | Descrizione |
|----------|------|-------|-------------|
| `getPage` | Query | `string` (slug) | Recupera pagina |
| `createPage` | Mutation | `{ title, content }` | Crea pagina |
| `updatePage` | Mutation | `{ id, title, content }` | Aggiorna pagina |
| `deletePage` | Mutation | `string` (id) | Elimina pagina |

```typescript
import * as trpc from '@trpc/server';
import * as trpcNext from '@trpc/server/adapters/next';
import { z } from 'zod';

export const appRouter = trpc.router()
 .query('getPage', {
    input: z.string(),
    async resolve({ input }) {
      // implementazione del recupero della pagina
    },
  })
 .mutation('createPage', {
    input: z.object({ title: z.string(), content: z.string() }),
    async resolve({ input }) {
      // implementazione della creazione della pagina
    },
  })
  //... altre query e mutation
```

### 4.2 Validazioni Zod

**File**: `src/lib/validations/website.ts`

| Schema | Campi | Uso |
|--------|-------|-----|
| `pageSchema` | title, content | Validazione pagine |
| `menuSchema` | label, url? | Validazione menu |
| `contactSchema` | name, email, message | Validazione form contatto |

```typescript
import { z } from 'zod';

export const pageSchema = z.object({
  title: z.string(),
  content: z.string(),
});

export const menuSchema = z.object({
  label: z.string(),
  url: z.string().optional(),
});

export const contactSchema = z.object({
  name: z.string(),
  email: z.string().email(),
  message: z.string(),
});
```

---

## 5. COMPONENTI WEBSITE

### 5.1 Mappa Componenti

| Componente | File | Descrizione |
|------------|------|-------------|
| **Header** | `header.tsx` | Navbar principale con logo e menu |
| **Footer** | `footer.tsx` | Footer con link e copyright |
| **Navigation** | `navigation.tsx` | Menu navigazione desktop |
| **MobileMenu** | `mobile-menu.tsx` | Menu hamburger mobile |
| **HeroSection** | `hero-section.tsx` | Banner principale landing |
| **FeaturesSection** | `features-section.tsx` | Griglia features |
| **TestimonialsSection** | `testimonials-section.tsx` | Slider testimonial |
| **TeamSection** | `team-section.tsx` | Griglia team members |
| **CtaSection** | `cta-section.tsx` | Call to action |
| **FaqSection** | `faq-section.tsx` | Accordion FAQ |
| **ContactForm** | `contact-form.tsx` | Form contatti con validazione |
| **ContactInfo** | `contact-info.tsx` | Info contatto (indirizzo, tel, email) |
| **Map** | `map.tsx` | Embed Google Maps |

### 5.2 Header

**File**: `src/components/website/header.tsx`

```tsx
import React from 'react';

const Header = () => {
  return (
    <header>
      <nav>
        <ul>
          <li><a href="#">Home</a></li>
          <li><a href="#">About</a></li>
          <li><a href="#">Contact</a></li>
        </ul>
      </nav>
    </header>
  );
};

export default Header;
```

### 5.3 Footer

**File**: `src/components/website/footer.tsx`

```tsx
import React from 'react';

const Footer = () => {
  return (
    <footer>
      <p>&copy; 2023 My Website</p>
      <ul>
        <li><a href="#">Terms of Service</a></li>
        <li><a href="#">Privacy Policy</a></li>
      </ul>
    </footer>
  );
};

export default Footer;
```

### 5.4 HeroSection

**File**: `src/components/website/hero-section.tsx`

```tsx
import React from 'react';

const HeroSection = () => {
  return (
    <section>
      <h1>Welcome to my website</h1>
      <p>This is a hero section</p>
      <button>Learn more</button>
    </section>
  );
};

export default HeroSection;
```

### 5.5 ContactForm

**File**: `src/components/website/contact-form.tsx`

```tsx
import React, { useState } from 'react';

const ContactForm = () => {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [message, setMessage] = useState('');

  const handleSubmit = (event) => {
    event.preventDefault();
    // invio del form
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Name:
        <input type="text" value={name} onChange={(event) => setName(event.target.value)} />
      </label>
      <label>
        Email:
        <input type="email" value={email} onChange={(event) => setEmail(event.target.value)} />
      </label>
      <label>
        Message:
        <textarea value={message} onChange={(event) => setMessage(event.target.value)} />
      </label>
      <button>Send</button>
    </form>
  );
};

export default ContactForm;
```

---

## 6. PAGINE PUBBLICHE

### 6.1 Struttura Route

| Route | File | Descrizione |
|-------|------|-------------|
| `/` | `src/app/(website)/page.tsx` | Homepage |
| `/[...slug]` | `src/app/(website)/[...slug]/page.tsx` | Pagine dinamiche |
| `/contact` | `src/app/(website)/contact/page.tsx` | Pagina contatti |
| Layout | `src/app/(website)/layout.tsx` | Layout wrapper |

### 6.2 Homepage

**File**: `src/app/(website)/page.tsx`

```tsx
import React from 'react';

const Page = () => {
  return (
    <div>
      <h1>Welcome to my website</h1>
      <p>This is the homepage</p>
    </div>
  );
};

export default Page;
```

### 6.3 Pagine Dinamiche

**File**: `src/app/(website)/[...slug]/page.tsx`

```tsx
import React from 'react';

const Page = () => {
  return (
    <div>
      <h1>Welcome to my website</h1>
      <p>This is a dynamic page</p>
    </div>
  );
};

export default Page;
```

### 6.4 Layout

**File**: `src/app/(website)/layout.tsx`

```tsx
import React from 'react';
import Header from '../components/website/header';
import Footer from '../components/website/footer';

const Layout = ({ children }) => {
  return (
    <div>
      <Header />
      {children}
      <Footer />
    </div>
  );
};

export default Layout;
```

---

## 7. PANNELLO ADMIN

### 7.1 Struttura Route Admin

| Route | File | Descrizione |
|-------|------|-------------|
| `/admin/pages` | `pages/page.tsx` | Lista pagine |
| `/admin/pages/new` | `pages/new/page.tsx` | Crea pagina |
| `/admin/pages/[id]/edit` | `pages/[id]/edit/page.tsx` | Modifica pagina |
| `/admin/menu` | `menu/page.tsx` | Gestione menu |
| `/admin/submissions` | `submissions/page.tsx` | Lista submissions |
| `/admin/settings/site` | `settings/site/page.tsx` | Impostazioni sito |

### 7.2 Lista Pagine

**File**: `src/app/admin/pages/page.tsx`

```tsx
import React, { useState, useEffect } from 'react';
import { useSession } from '../context/session';

const Pages = () => {
  const [pages, setPages] = useState([]);
  const session = useSession();

  useEffect(() => {
    const fetchPages = async () => {
      const response = await fetch('/api/pages');
      const data = await response.json();
      setPages(data);
    };
    fetchPages();
  }, []);

  return (
    <div>
      <h1>Pages</h1>
      <ul>
        {pages.map((page) => (
          <li key={page.id}>
            <a href={`/admin/pages/${page.id}/edit`}>{page.title}</a>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default Pages;
```

### 7.3 Crea Pagina

**File**: `src/app/admin/pages/new/page.tsx`

```tsx
import React, { useState } from 'react';

const NewPage = () => {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');

  const handleSubmit = (event) => {
    event.preventDefault();
    // creazione della pagina
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Title:
        <input type="text" value={title} onChange={(event) => setTitle(event.target.value)} />
      </label>
      <label>
        Content:
        <textarea value={content} onChange={(event) => setContent(event.target.value)} />
      </label>
      <button>Create page</button>
    </form>
  );
};

export default NewPage;
```

### 7.4 Section Builder

**File**: `src/components/admin/pages/section-builder.tsx`

```tsx
import React, { useState } from 'react';

const SectionBuilder = () => {
  const [sections, setSections] = useState([]);

  const handleAddSection = () => {
    setSections([...sections, { type: 'text', content: '' }]);
  };

  const handleRemoveSection = (index) => {
    setSections(sections.filter((section, i) => i !== index));
  };

  return (
    <div>
      {sections.map((section, index) => (
        <div key={index}>
          <textarea
            value={section.content}
            onChange={(event) => setSections(sections.map((s, i) =>
              (i === index ? {...s, content: event.target.value } : s)
            ))}
          />
          <button onClick={() => handleRemoveSection(index)}>Remove</button>
        </div>
      ))}
      <button onClick={handleAddSection}>Add section</button>
    </div>
  );
};

export default SectionBuilder;
```

### 7.5 Site Settings

**File**: `src/app/admin/settings/site/page.tsx`

```tsx
import React, { useState, useEffect } from 'react';

const SiteSettings = () => {
  const [siteName, setSiteName] = useState('');
  const [siteDescription, setSiteDescription] = useState('');

  useEffect(() => {
    const fetchSiteSettings = async () => {
      const response = await fetch('/api/settings');
      const data = await response.json();
      setSiteName(data.siteName);
      setSiteDescription(data.siteDescription);
    };
    fetchSiteSettings();
  }, []);

  const handleSubmit = (event) => {
    event.preventDefault();
    // aggiornamento delle impostazioni del sito
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Site name:
        <input type="text" value={siteName} onChange={(event) => setSiteName(event.target.value)} />
      </label>
      <label>
        Site description:
        <textarea value={siteDescription} onChange={(event) => setSiteDescription(event.target.value)} />
      </label>
      <button>Save settings</button>
    </form>
  );
};

export default SiteSettings;
```

---

## 8. TESTING

### 8.1 Test Componenti

**File**: `tests/website.test.ts`

```typescript
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react';
import { Page } from '../components/website/page';

describe('Page component', () => {
  it('renders correctly', () => {
    const { container } = render(<Page />);
    expect(container).toMatchSnapshot();
  });

  it('handles click on link', async () => {
    const { getByText } = render(<Page />);
    const link = getByText('Learn more');
    fireEvent.click(link);
    await waitFor(() => expect(window.location.href).toBe('https://example.com'));
  });
});
```

---

## 10. CHECKLIST IMPLEMENTAZIONE

| Step | Descrizione | Status |
|------|-------------|--------|
| 1 | Creare schema Prisma `schema-website.prisma` | ⬜ |
| 2 | Implementare PageService completo | ⬜ |
| 3 | Implementare MenuService con reorder | ⬜ |
| 4 | Implementare ContactService con email | ⬜ |
| 5 | Implementare SettingsService | ⬜ |
| 6 | Creare router tRPC website | ⬜ |
| 7 | Creare validazioni Zod | ⬜ |
| 8 | Implementare componenti UI website | ⬜ |
| 9 | Creare pagine pubbliche con layout | ⬜ |
| 10 | Implementare pannello admin CRUD | ⬜ |
| 11 | Aggiungere Section Builder | ⬜ |
| 12 | Scrivere test componenti | ⬜ |

---

_Integrato da: 17-OUTPUT-WEBSITE-COMPLETO.md_
_Data integrazione: 2026-01-29 14:51_
_Generato con: Gemini 2.5 Flash / DeepSeek R1_