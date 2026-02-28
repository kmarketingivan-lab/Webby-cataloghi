# CATALOGO-WEBSITE-TEMPLATE-v1

> **Dominio**: Website/CMS dinamico con gestione contenuti
> **Stack**: Next.js 14, TypeScript, Prisma, tRPC
> **Versione**: 1.0
> **Data**: 2026-01-29

---

§ 1. INDICE

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

§ 1. PANORAMICA

§ 1.1 OBIETTIVO
Sito web dinamico con funzionalità CMS per gestione contenuti, menu, form contatti e impostazioni del sito.

§ 1.2 FUNZIONALITÀ PRINCIPALI

| Funzionalità | Descrizione |
|--------------|-------------|
| **Gestione Pagine** | CRUD pagine con slug, status (DRAFT/PUBLISHED), SEO |
| **Menu Dinamici** | Menu gerarchici con drag & drop reordering |
| **Form Contatti** | Submissions con status, spam detection, email notifications |
| **Site Settings** | Configurazione globale sito (nome, descrizione, logo) |

§ 1.3 STRUTTURA FILE

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

§ 2. SCHEMA DATABASE

§ 2.1 FILE: `PRISMA/SCHEMA-WEBSITE.PRISMA`

prisma
// Schema database per CMS website
// Modelli: Page, MenuItem, ContactSubmission, SiteSetting

§ 2.2 MODELLI PRINCIPALI

| Modello | Campi Chiave | Relazioni |
|---------|--------------|-----------|
| `Page` | id, title, slug, content, status, showInNav | - |
| `MenuItem` | id, label, url, location, order, parentId | self-relation (parent/children) |
| `ContactSubmission` | id, name, email, message, status | - |
| `SiteSetting` | key, value | - |

---

§ 3. SERVICES LAYER

§ 3.1 PAGESERVICE

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

typescript
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

§ 3.2 MENUSERVICE

**File**: `src/server/services/menu-service.ts`

| Metodo | Parametri | Return | Descrizione |
|--------|-----------|--------|-------------|
| `getMenu` | `location` | `MenuItem[]` | Menu per posizione |
| `createItem` | `CreateMenuItemInput` | `MenuItem` | Crea voce menu |
| `updateItem` | `itemId, UpdateMenuItemInput` | `MenuItem` | Aggiorna voce |
| `deleteItem` | `itemId` | `void` | Elimina voce |
| `reorder` | `items[]` | `void` | Riordina voci |

typescript
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

§ 3.3 CONTACTSERVICE

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

typescript
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

§ 3.4 SETTINGSSERVICE

**File**: `src/server/services/settings-service.ts`

| Metodo | Parametri | Return | Descrizione |
|--------|-----------|--------|-------------|
| `get<T>` | `key` | `T \| null` | Legge setting |
| `set<T>` | `key, value` | `void` | Scrive setting |
| `getAll` | - | `Record<string, any>` | Tutte le settings |
| `getSiteInfo` | - | `SiteInfo` | Info sito aggregate |
| `updateSiteInfo` | `Partial<SiteInfo>` | `SiteInfo` | Aggiorna info sito |

typescript
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

---

§ 4. API LAYER

§ 4.1 ROUTER TRPC

**File**: `src/server/trpc/routers/website.ts`

| Endpoint | Tipo | Input | Descrizione |
|----------|------|-------|-------------|
| `getPage` | Query | `string` (slug) | Recupera pagina |
| `createPage` | Mutation | `{ title, content }` | Crea pagina |
| `updatePage` | Mutation | `{ id, title, content }` | Aggiorna pagina |
| `deletePage` | Mutation | `string` (id) | Elimina pagina |

typescript
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

§ 4.2 VALIDAZIONI ZOD

**File**: `src/lib/validations/website.ts`

| Schema | Campi | Uso |
|--------|-------|-----|
| `pageSchema` | title, content | Validazione pagine |
| `menuSchema` | label, url? | Validazione menu |
| `contactSchema` | name, email, message | Validazione form contatto |

typescript
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

---

§ 5. COMPONENTI WEBSITE

§ 5.1 MAPPA COMPONENTI

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

§ 5.2 HEADER

**File**: `src/components/website/header.tsx`

tsx
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

§ 5.3 FOOTER

**File**: `src/components/website/footer.tsx`

tsx
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

§ 5.4 HEROSECTION

**File**: `src/components/website/hero-section.tsx`

tsx
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

§ 5.5 CONTACTFORM

**File**: `src/components/website/contact-form.tsx`

tsx
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

---

§ 6. PAGINE PUBBLICHE

§ 6.1 STRUTTURA ROUTE

| Route | File | Descrizione |
|-------|------|-------------|
| `/` | `src/app/(website)/page.tsx` | Homepage |
| `/[...slug]` | `src/app/(website)/[...slug]/page.tsx` | Pagine dinamiche |
| `/contact` | `src/app/(website)/contact/page.tsx` | Pagina contatti |
| Layout | `src/app/(website)/layout.tsx` | Layout wrapper |

§ 6.2 HOMEPAGE

**File**: `src/app/(website)/page.tsx`

tsx
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

§ 6.3 PAGINE DINAMICHE

**File**: `src/app/(website)/[...slug]/page.tsx`

tsx
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

§ 6.4 LAYOUT

**File**: `src/app/(website)/layout.tsx`

tsx
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

---

§ 7. PANNELLO ADMIN

§ 7.1 STRUTTURA ROUTE ADMIN

| Route | File | Descrizione |
|-------|------|-------------|
| `/admin/pages` | `pages/page.tsx` | Lista pagine |
| `/admin/pages/new` | `pages/new/page.tsx` | Crea pagina |
| `/admin/pages/[id]/edit` | `pages/[id]/edit/page.tsx` | Modifica pagina |
| `/admin/menu` | `menu/page.tsx` | Gestione menu |
| `/admin/submissions` | `submissions/page.tsx` | Lista submissions |
| `/admin/settings/site` | `settings/site/page.tsx` | Impostazioni sito |

§ 7.2 LISTA PAGINE

**File**: `src/app/admin/pages/page.tsx`

tsx
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

§ 7.3 CREA PAGINA

**File**: `src/app/admin/pages/new/page.tsx`

tsx
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

§ 7.4 SECTION BUILDER

**File**: `src/components/admin/pages/section-builder.tsx`

tsx
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

§ 7.5 SITE SETTINGS

**File**: `src/app/admin/settings/site/page.tsx`

tsx
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

---

§ 8. TESTING

§ 8.1 TEST COMPONENTI

**File**: `tests/website.test.ts`

typescript
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

---

§ 10. CHECKLIST IMPLEMENTAZIONE

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

Catalogo Template Next.js 14 - Website Templates & Landing Pages
1. Hero Section Patterns
1.1 Centered Hero
tsx
Copia
Scarica
export default function CenteredHero() {
  return (
    <section className="relative overflow-hidden bg-gradient-to-b from-white to-gray-50 dark:from-gray-900 dark:to-gray-950 py-24 sm:py-32">
      <div className="absolute inset-0 bg-grid-slate-100 dark:bg-grid-slate-800/20 [mask-image:linear-gradient(0deg,white,rgba(255,255,255,0.6))]" />
      <div className="container relative mx-auto px-4 sm:px-6 lg:px-8">
        <div className="mx-auto max-w-3xl text-center">
          <div className="mb-6 inline-flex items-center rounded-full bg-blue-100 dark:bg-blue-900/30 px-4 py-2 text-sm font-medium text-blue-800 dark:text-blue-300">
            <span className="mr-2">✨</span> Nuova funzionalità disponibile
          </div>
          <h1 className="text-4xl font-bold tracking-tight text-gray-900 dark:text-white sm:text-6xl">
            Costruisci il tuo prodotto
            <span className="block text-blue-600 dark:text-blue-400">con velocità e precisione</span>
          </h1>
          <p className="mt-6 text-lg leading-8 text-gray-600 dark:text-gray-300">
            Una piattaforma completa per team che vogliono realizzare progetti straordinari.
            Ottieni tutto ciò che ti serve per costruire, lanciare e far crescere il tuo prodotto.
          </p>
          <div className="mt-10 flex flex-col sm:flex-row gap-4 justify-center">
            <button className="rounded-lg bg-blue-600 dark:bg-blue-500 px-6 py-3 text-sm font-semibold text-white shadow-sm hover:bg-blue-500 dark:hover:bg-blue-400 transition-colors focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-600">
              Inizia gratis
            </button>
            <button className="rounded-lg bg-white dark:bg-gray-800 px-6 py-3 text-sm font-semibold text-gray-900 dark:text-white shadow-sm ring-1 ring-inset ring-gray-300 dark:ring-gray-700 hover:bg-gray-50 dark:hover:bg-gray-700 transition-colors">
              <span className="flex items-center justify-center gap-2">
                <svg className="h-5 w-5" fill="currentColor" viewBox="0 0 24 24">
                  <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z" />
                </svg>
                Vedi demo
              </span>
            </button>
          </div>
          <div className="mt-16 flex justify-center">
            <div className="relative rounded-xl border border-gray-200 dark:border-gray-800 bg-white dark:bg-gray-900 p-2 shadow-lg">
              <div className="h-64 w-full bg-gradient-to-r from-blue-500 to-purple-600 rounded-lg flex items-center justify-center">
                <div className="text-white text-center p-4">
                  <div className="text-3xl mb-2">🎨</div>
                  <p className="font-medium">Dashboard Interattiva</p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
  );
}
1.2 Split Layout Hero
tsx
Copia
Scarica
export default function SplitHero() {
  return (
    <section className="relative overflow-hidden bg-white dark:bg-gray-900">
      <div className="container mx-auto px-4 sm:px-6 lg:px-8 py-24">
        <div className="grid lg:grid-cols-2 gap-12 items-center">
          <div>
            <span className="inline-block rounded-full bg-gradient-to-r from-blue-500 to-purple-600 px-4 py-2 text-sm font-semibold text-white mb-6">
              🚀 Più di 10.000 clienti
            </span>
            <h1 className="text-4xl font-bold tracking-tight text-gray-900 dark:text-white sm:text-5xl lg:text-6xl">
              Trasforma le tue idee in
              <span className="block text-transparent bg-clip-text bg-gradient-to-r from-blue-600 to-purple-600">
                prodotti digitali
              </span>
            </h1>
            <p className="mt-6 text-lg text-gray-600 dark:text-gray-300">
              La piattaforma completa per sviluppatori e designer che vogliono creare esperienze
              digitali straordinarie. Tutti gli strumenti di cui hai bisogno in un unico posto.
            </p>
            <div className="mt-10 flex flex-col sm:flex-row gap-4">
              <div className="flex-1">
                <input
                  type="email"
                  placeholder="La tua email"
                  className="w-full rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-800 px-4 py-3 text-gray-900 dark:text-white placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-500"
                />
              </div>
              <button className="rounded-lg bg-gradient-to-r from-blue-600 to-purple-600 px-6 py-3 text-sm font-semibold text-white shadow-lg hover:from-blue-500 hover:to-purple-500 transition-all">
                Inizia gratis
              </button>
            </div>
            <div className="mt-8 flex items-center gap-6">
              <div className="flex -space-x-2">
                {[1, 2, 3, 4].map((i) => (
                  <div
                    key={i}
                    className="h-8 w-8 rounded-full border-2 border-white dark:border-gray-900 bg-gradient-to-br from-blue-400 to-purple-500"
                  />
                ))}
              </div>
              <p className="text-sm text-gray-600 dark:text-gray-400">
                Unisciti a <span className="font-semibold">5.000+</span> professionisti
              </p>
            </div>
          </div>
          <div className="relative">
            <div className="relative rounded-2xl overflow-hidden shadow-2xl">
              <div className="aspect-[4/3] bg-gradient-to-br from-gray-900 to-gray-800">
                <div className="absolute inset-0 flex items-center justify-center">
                  <div className="text-center p-8">
                    <div className="text-6xl mb-4">📊</div>
                    <h3 className="text-2xl font-bold text-white mb-2">Dashboard Live</h3>
                    <p className="text-gray-300">Monitora le metriche in tempo reale</p>
                  </div>
                </div>
              </div>
            </div>
            <div className="absolute -bottom-6 -left-6 h-48 w-48 rounded-full bg-gradient-to-r from-blue-500/20 to-purple-500/20 blur-3xl" />
            <div className="absolute -top-6 -right-6 h-48 w-48 rounded-full bg-gradient-to-r from-purple-500/20 to-pink-500/20 blur-3xl" />
          </div>
        </div>
      </div>
    </section>
  );
}
1.3 Video Background Hero
tsx
Copia
Scarica
export default function VideoHero() {
  return (
    <section className="relative min-h-screen flex items-center justify-center overflow-hidden">
      <div className="absolute inset-0">
        <div className="absolute inset-0 bg-gradient-to-b from-black/70 via-black/50 to-black/70 z-10" />
        <video
          autoPlay
          muted
          loop
          className="absolute inset-0 w-full h-full object-cover"
        >
          <source
            src="https://assets.mixkit.co/videos/preview/mixkit-abstract-geometric-technology-background-33471-large.mp4"
            type="video/mp4"
          />
        </video>
      </div>
      <div className="container relative z-20 mx-auto px-4 sm:px-6 lg:px-8 text-center">
        <h1 className="text-5xl sm:text-7xl font-bold text-white mb-6">
          Esperienze
          <span className="block text-transparent bg-clip-text bg-gradient-to-r from-blue-400 to-purple-500">
            Immersive
          </span>
        </h1>
        <p className="text-xl text-gray-200 max-w-2xl mx-auto mb-10">
          Creiamo esperienze digitali che connettono brand e persone attraverso tecnologia
          all'avanguardia e design innovativo.
        </p>
        <div className="flex flex-col sm:flex-row gap-4 justify-center">
          <button className="px-8 py-4 bg-white text-gray-900 font-semibold rounded-lg hover:bg-gray-100 transition-all transform hover:scale-105">
            Scopri di più
          </button>
          <button className="px-8 py-4 border-2 border-white text-white font-semibold rounded-lg hover:bg-white/10 transition-all">
            Guarda il video
          </button>
        </div>
        <div className="mt-20 grid grid-cols-2 md:grid-cols-4 gap-8">
          {['Design', 'Sviluppo', 'AI', 'Cloud'].map((item) => (
            <div key={item} className="text-center">
              <div className="text-3xl font-bold text-white mb-2">100+</div>
              <div className="text-gray-300">{item}</div>
            </div>
          ))}
        </div>
      </div>
      <div className="absolute bottom-8 left-1/2 transform -translate-x-1/2 z-20">
        <div className="animate-bounce">
          <svg className="h-8 w-8 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 14l-7 7m0 0l-7-7m7 7V3" />
          </svg>
        </div>
      </div>
    </section>
  );
}
1.4 Animated Hero
tsx
Copia
Scarica
'use client';

import { useState, useEffect } from 'react';

const words = ['Design', 'Sviluppo', 'Marketing', 'Analytics'];

export default function AnimatedHero() {
  const [currentWordIndex, setCurrentWordIndex] = useState(0);
  const [fade, setFade] = useState(true);

  useEffect(() => {
    const interval = setInterval(() => {
      setFade(false);
      setTimeout(() => {
        setCurrentWordIndex((prev) => (prev + 1) % words.length);
        setFade(true);
      }, 300);
    }, 2000);
    return () => clearInterval(interval);
  }, []);

  return (
    <section className="min-h-screen flex items-center justify-center bg-gradient-to-br from-gray-50 to-white dark:from-gray-900 dark:to-gray-950">
      <div className="container mx-auto px-4 sm:px-6 lg:px-8">
        <div className="text-center">
          <div className="mb-8">
            <div className="inline-block rounded-full bg-gradient-to-r from-blue-500 to-purple-500 p-1">
              <div className="rounded-full bg-white dark:bg-gray-900 px-4 py-2">
                <span className="text-sm font-semibold bg-gradient-to-r from-blue-600 to-purple-600 bg-clip-text text-transparent">
                  ✨ Piattaforma tutto-in-uno
                </span>
              </div>
            </div>
          </div>
          <h1 className="text-5xl sm:text-7xl font-bold mb-6">
            <span className="block text-gray-900 dark:text-white">Soluzioni per</span>
            <div className="h-24 sm:h-32 mt-4">
              <span
                className={`inline-block text-transparent bg-clip-text bg-gradient-to-r from-blue-600 to-purple-600 transition-all duration-300 ${
                  fade ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-4'
                }`}
              >
                {words[currentWordIndex]}
              </span>
            </div>
          </h1>
          <p className="text-xl text-gray-600 dark:text-gray-300 max-w-2xl mx-auto mb-10">
            Un toolkit completo per trasformare le tue idee in realtà digitali.
            Design, sviluppo, deploy e analytics in un'unica piattaforma.
          </p>
          <div className="flex flex-col sm:flex-row gap-4 justify-center">
            <button className="group relative px-8 py-4 bg-gradient-to-r from-blue-600 to-purple-600 text-white font-semibold rounded-lg overflow-hidden">
              <span className="relative z-10">Inizia ora</span>
              <div className="absolute inset-0 bg-gradient-to-r from-blue-700 to-purple-700 translate-y-full group-hover:translate-y-0 transition-transform duration-300" />
            </button>
            <button className="px-8 py-4 border-2 border-gray-300 dark:border-gray-700 text-gray-700 dark:text-gray-300 font-semibold rounded-lg hover:border-blue-500 hover:text-blue-500 transition-all">
              Prenota demo
            </button>
          </div>
        </div>
        <div className="mt-20 relative">
          <div className="absolute inset-0 bg-gradient-to-r from-blue-500/10 via-purple-500/10 to-pink-500/10 blur-3xl" />
          <div className="relative grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
            {['Dashboard', 'Analytics', 'Team', 'API'].map((feature) => (
              <div
                key={feature}
                className="bg-white/50 dark:bg-gray-800/50 backdrop-blur-sm rounded-xl p-6 border border-gray-200 dark:border-gray-700 hover:border-blue-500 dark:hover:border-blue-500 transition-all hover:scale-105"
              >
                <div className="text-3xl mb-4">📊</div>
                <h3 className="font-semibold text-gray-900 dark:text-white">{feature}</h3>
              </div>
            ))}
          </div>
        </div>
      </div>
    </section>
  );
}
1.5 Gradient Hero
tsx
Copia
Scarica
export default function GradientHero() {
  return (
    <section className="relative min-h-screen flex items-center justify-center overflow-hidden">
      <div className="absolute inset-0 bg-gradient-to-br from-blue-600 via-purple-600 to-pink-600">
        <div className="absolute inset-0 bg-[url('/grid.svg')] opacity-20" />
      </div>
      <div className="relative z-10 container mx-auto px-4 sm:px-6 lg:px-8 text-center">
        <h1 className="text-5xl sm:text-7xl lg:text-8xl font-bold text-white mb-8">
          Innovazione
          <span className="block">Digitale</span>
        </h1>
        <p className="text-xl text-white/80 max-w-2xl mx-auto mb-12">
          Trasformiamo le tue visioni in esperienze digitali memorabili.
          Tecnologia avanzata e design eccellente per risultati straordinari.
        </p>
        <div className="flex flex-col sm:flex-row gap-6 justify-center items-center">
          <button className="px-10 py-5 bg-white text-gray-900 font-bold rounded-full text-lg hover:bg-gray-100 transition-all transform hover:scale-105 shadow-2xl">
            Inizia il progetto
          </button>
          <button className="px-10 py-5 bg-transparent border-2 border-white text-white font-bold rounded-full text-lg hover:bg-white/10 transition-all">
            Scopri di più
          </button>
        </div>
        <div className="mt-24 grid grid-cols-1 md:grid-cols-3 gap-8">
          {[
            { title: 'Progetti completati', value: '500+' },
            { title: 'Clienti soddisfatti', value: '98%' },
            { title: 'Supporto 24/7', value: '∞' },
          ].map((stat) => (
            <div key={stat.title} className="text-center">
              <div className="text-5xl font-bold text-white mb-2">{stat.value}</div>
              <div className="text-white/70">{stat.title}</div>
            </div>
          ))}
        </div>
      </div>
      <div className="absolute bottom-0 left-0 right-0 h-32 bg-gradient-to-t from-white dark:from-gray-900 to-transparent" />
    </section>
  );
}
1.6 Image Left Hero
tsx
Copia
Scarica
export default function ImageLeftHero() {
  return (
    <section className="min-h-screen bg-white dark:bg-gray-900">
      <div className="container mx-auto px-4 sm:px-6 lg:px-8">
        <div className="grid lg:grid-cols-2 gap-16 items-center min-h-screen">
          <div className="relative">
            <div className="relative rounded-3xl overflow-hidden shadow-2xl">
              <div className="aspect-[4/3] bg-gradient-to-br from-blue-500 to-purple-600">
                <div className="absolute inset-0 flex items-center justify-center">
                  <div className="text-center p-8">
                    <div className="text-6xl mb-6">🚀</div>
                    <h3 className="text-3xl font-bold text-white mb-4">Dashboard Avanzata</h3>
                    <p className="text-blue-100">Interfaccia intuitiva e potente</p>
                  </div>
                </div>
              </div>
            </div>
            <div className="absolute -bottom-6 -right-6 w-64 h-64 bg-gradient-to-r from-blue-500/20 to-purple-500/20 rounded-full blur-3xl" />
          </div>
          <div>
            <h2 className="text-sm font-semibold text-blue-600 dark:text-blue-400 uppercase tracking-wide mb-4">
              Piattaforma Completa
            </h2>
            <h1 className="text-4xl sm:text-5xl lg:text-6xl font-bold text-gray-900 dark:text-white mb-6">
              Tutto ciò di cui hai bisogno per
              <span className="block text-blue-600 dark:text-blue-400">crescere online</span>
            </h1>
            <p className="text-lg text-gray-600 dark:text-gray-300 mb-10">
              Una suite completa di strumenti per gestire la tua presenza digitale.
              Dal design allo sviluppo, dall'analisi all'ottimizzazione.
            </p>
            <div className="space-y-6">
              {[
                { icon: '⚡', text: 'Performance ottimizzate' },
                { icon: '🔒', text: 'Sicurezza enterprise' },
                { icon: '📈', text: 'Analytics in tempo reale' },
              ].map((feature, index) => (
                <div key={index} className="flex items-center gap-4">
                  <div className="flex-shrink-0 w-12 h-12 bg-blue-100 dark:bg-blue-900/30 rounded-lg flex items-center justify-center">
                    <span className="text-2xl">{feature.icon}</span>
                  </div>
                  <span className="text-gray-700 dark:text-gray-300">{feature.text}</span>
                </div>
              ))}
            </div>
            <div className="mt-12 flex flex-col sm:flex-row gap-4">
              <button className="flex-1 bg-blue-600 dark:bg-blue-500 text-white font-semibold py-4 px-8 rounded-lg hover:bg-blue-700 dark:hover:bg-blue-600 transition-colors">
                Inizia gratis
              </button>
              <button className="flex-1 border-2 border-gray-300 dark:border-gray-700 text-gray-700 dark:text-gray-300 font-semibold py-4 px-8 rounded-lg hover:border-blue-500 hover:text-blue-500 transition-colors">
                Vedi demo
              </button>
            </div>
          </div>
        </div>
      </div>
    </section>
  );
}
1.7 Image Right Hero
tsx
Copia
Scarica
export default function ImageRightHero() {
  return (
    <section className="relative min-h-screen bg-gradient-to-br from-gray-50 to-white dark:from-gray-900 dark:to-gray-950 overflow-hidden">
      <div className="absolute inset-0 bg-grid-slate-100 dark:bg-grid-slate-800/20" />
      <div className="container relative mx-auto px-4 sm:px-6 lg:px-8 py-24">
        <div className="grid lg:grid-cols-2 gap-16 items-center">
          <div>
            <h1 className="text-4xl sm:text-5xl lg:text-6xl font-bold text-gray-900 dark:text-white mb-6">
              La piattaforma
              <span className="block text-transparent bg-clip-text bg-gradient-to-r from-blue-600 to-purple-600">
                per team moderni
              </span>
            </h1>
            <p className="text-lg text-gray-600 dark:text-gray-300 mb-8">
              Collabora, sviluppa e distribuisci in modo efficiente con strumenti pensati per
              team che vogliono fare la differenza nel mondo digitale.
            </p>
            <div className="space-y-4 mb-10">
              {['Collaborazione in tempo reale', 'Integrazioni illimitate', 'Deploy automatico'].map(
                (item, index) => (
                  <div key={index} className="flex items-center gap-3">
                    <svg className="h-6 w-6 text-green-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fillRule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clipRule="evenodd" />
                    </svg>
                    <span className="text-gray-700 dark:text-gray-300">{item}</span>
                  </div>
                )
              )}
            </div>
            <div className="flex flex-col sm:flex-row gap-4">
              <button className="px-8 py-4 bg-gradient-to-r from-blue-600 to-purple-600 text-white font-semibold rounded-lg hover:from-blue-700 hover:to-purple-700 transition-all shadow-lg hover:shadow-xl">
                Prova gratuitamente
              </button>
              <button className="px-8 py-4 border-2 border-gray-300 dark:border-gray-700 text-gray-700 dark:text-gray-300 font-semibold rounded-lg hover:border-blue-500 hover:text-blue-500 transition-colors">
                Prenota demo
              </button>
            </div>
          </div>
          <div className="relative">
            <div className="relative rounded-3xl overflow-hidden shadow-2xl">
              <div className="aspect-[4/3] bg-gradient-to-tr from-gray-900 to-gray-800">
                <div className="absolute inset-0 flex items-center justify-center p-8">
                  <div className="text-center">
                    <div className="text-6xl mb-6">👥</div>
                    <h3 className="text-3xl font-bold text-white mb-4">Team Collaboration</h3>
                    <p className="text-gray-300">Lavora insieme in tempo reale</p>
                  </div>
                </div>
              </div>
            </div>
            <div className="absolute -top-6 -left-6 w-64 h-64 bg-gradient-to-r from-blue-500/20 to-purple-500/20 rounded-full blur-3xl" />
            <div className="absolute -bottom-6 -right-6 w-64 h-64 bg-gradient-to-r from-purple-500/20 to-pink-500/20 rounded-full blur-3xl" />
          </div>
        </div>
      </div>
    </section>
  );
}
1.8 Minimal Hero
tsx
Copia
Scarica
export default function MinimalHero() {
  return (
    <section className="min-h-screen flex items-center justify-center bg-white dark:bg-gray-900">
      <div className="container mx-auto px-4 sm:px-6 lg:px-8">
        <div className="max-w-2xl mx-auto text-center">
          <h1 className="text-4xl sm:text-5xl lg:text-6xl font-light text-gray-900 dark:text-white mb-8 tracking-tight">
            Semplicità ed
            <span className="block font-normal">eleganza digitale</span>
          </h1>
          <p className="text-lg text-gray-600 dark:text-gray-400 mb-12 leading-relaxed">
            Creiamo esperienze digitali minimali che comunicano con chiarezza ed efficacia.
            Meno è più, quando ogni dettaglio è curato alla perfezione.
          </p>
          <div className="flex flex-col sm:flex-row gap-4 justify-center">
            <button className="px-8 py-3 bg-gray-900 dark:bg-white text-white dark:text-gray-900 font-medium rounded hover:bg-gray-800 dark:hover:bg-gray-100 transition-colors">
              Inizia progetto
            </button>
            <button className="px-8 py-3 border border-gray-300 dark:border-gray-700 text-gray-700 dark:text-gray-300 font-medium rounded hover:border-gray-400 dark:hover:border-gray-600 transition-colors">
              Scopri il processo
            </button>
          </div>
          <div className="mt-24 border-t border-gray-200 dark:border-gray-800 pt-12">
            <div className="grid grid-cols-2 md:grid-cols-4 gap-8">
              {[
                { name: 'Apple', color: 'text-gray-900 dark:text-white' },
                { name: 'Google', color: 'text-gray-900 dark:text-white' },
                { name: 'Microsoft', color: 'text-gray-900 dark:text-white' },
                { name: 'Amazon', color: 'text-gray-900 dark:text-white' },
              ].map((company) => (
                <div key={company.name} className={`text-sm ${company.color} opacity-70`}>
                  {company.name}
                </div>
              ))}
            </div>
          </div>
        </div>
      </div>
    </section>
  );
}
2. Navigation Patterns
2.1 Responsive Navbar with Mobile Menu
tsx
Copia
Scarica
'use client';

import { useState } from 'react';

export default function Navigation() {
  const [isOpen, setIsOpen] = useState(false);
  const [isScrolled, setIsScrolled] = useState(false);

  if (typeof window !== 'undefined') {
    window.addEventListener('scroll', () => {
      setIsScrolled(window.scrollY > 10);
    });
  }

  return (
    <nav
      className={`fixed w-full z-50 transition-all duration-300 ${
        isScrolled
          ? 'bg-white/80 dark:bg-gray-900/80 backdrop-blur-md border-b border-gray-200 dark:border-gray-800'
          : 'bg-transparent'
      }`}
    >
      <div className="container mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex items-center justify-between h-16">
          <div className="flex items-center">
            <div className="text-2xl font-bold text-gray-900 dark:text-white">
              Logo
            </div>
            <div className="hidden md:block ml-10">
              <div className="flex items-baseline space-x-4">
                {['Home', 'Features', 'Pricing', 'Blog', 'Contact'].map((item) => (
                  <a
                    key={item}
                    href="#"
                    className="text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400 px-3 py-2 rounded-md text-sm font-medium transition-colors"
                  >
                    {item}
                  </a>
                ))}
              </div>
            </div>
          </div>
          <div className="hidden md:block">
            <div className="flex items-center space-x-4">
              <button className="text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400 px-4 py-2 text-sm font-medium">
                Sign In
              </button>
              <button className="bg-blue-600 dark:bg-blue-500 text-white px-6 py-2 rounded-lg text-sm font-medium hover:bg-blue-700 dark:hover:bg-blue-600 transition-colors">
                Get Started
              </button>
            </div>
          </div>
          <div className="md:hidden">
            <button
              onClick={() => setIsOpen(!isOpen)}
              className="text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400"
            >
              <svg className="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                {isOpen ? (
                  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
                ) : (
                  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4 6h16M4 12h16M4 18h16" />
                )}
              </svg>
            </button>
          </div>
        </div>
      </div>

      {/* Mobile menu */}
      {isOpen && (
        <div className="md:hidden bg-white dark:bg-gray-900 border-t border-gray-200 dark:border-gray-800">
          <div className="px-2 pt-2 pb-3 space-y-1 sm:px-3">
            {['Home', 'Features', 'Pricing', 'Blog', 'Contact'].map((item) => (
              <a
                key={item}
                href="#"
                className="text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400 block px-3 py-2 rounded-md text-base font-medium"
              >
                {item}
              </a>
            ))}
            <div className="pt-4 space-y-2">
              <button className="w-full text-center text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400 px-4 py-2 text-base font-medium">
                Sign In
              </button>
              <button className="w-full bg-blue-600 dark:bg-blue-500 text-white px-6 py-2 rounded-lg text-base font-medium hover:bg-blue-700 dark:hover:bg-blue-600 transition-colors">
                Get Started
              </button>
            </div>
          </div>
        </div>
      )}
    </nav>
  );
}
2.2 Sticky Header with Mega Menu
tsx
Copia
Scarica
'use client';

import { useState } from 'react';

const menuItems = {
  Product: ['Features', 'Pricing', 'Integrations', 'API'],
  Solutions: ['Marketing', 'Sales', 'Service', 'Analytics'],
  Resources: ['Blog', 'Documentation', 'Help Center', 'Community'],
  Company: ['About', 'Careers', 'Press', 'Contact'],
};

export default function MegaMenuNavigation() {
  const [activeMenu, setActiveMenu] = useState<string | null>(null);

  return (
    <nav className="sticky top-0 bg-white dark:bg-gray-900 border-b border-gray-200 dark:border-gray-800 z-50">
      <div className="container mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex items-center justify-between h-16">
          <div className="flex items-center">
            <div className="text-xl font-bold text-gray-900 dark:text-white">
              Brand
            </div>
            <div className="hidden lg:flex ml-10 space-x-1">
              {Object.keys(menuItems).map((menu) => (
                <div
                  key={menu}
                  className="relative"
                  onMouseEnter={() => setActiveMenu(menu)}
                  onMouseLeave={() => setActiveMenu(null)}
                >
                  <button className="text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400 px-4 py-2 text-sm font-medium flex items-center gap-1">
                    {menu}
                    <svg className="h-4 w-4" fill="currentColor" viewBox="0 0 20 20">
                      <path fillRule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clipRule="evenodd" />
                    </svg>
                  </button>
                  {activeMenu === menu && (
                    <div className="absolute left-0 w-64 bg-white dark:bg-gray-800 border border-gray-200 dark:border-gray-700 rounded-lg shadow-lg p-4">
                      {menuItems[menu as keyof typeof menuItems].map((item) => (
                        <a
                          key={item}
                          href="#"
                          className="block px-4 py-2 text-sm text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700 rounded-md hover:text-blue-600 dark:hover:text-blue-400"
                        >
                          {item}
                        </a>
                      ))}
                    </div>
                  )}
                </div>
              ))}
            </div>
          </div>
          <div className="flex items-center space-x-4">
            <button className="text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400 px-4 py-2 text-sm font-medium">
              Sign In
            </button>
            <button className="bg-gradient-to-r from-blue-600 to-purple-600 text-white px-6 py-2 rounded-lg text-sm font-medium hover:from-blue-700 hover:to-purple-700 transition-all">
              Get Started
            </button>
          </div>
        </div>
      </div>
    </nav>
  );
}
2.3 Sidebar Navigation
tsx
Copia
Scarica
'use client';

import { useState } from 'react';

const navigation = [
  { name: 'Dashboard', icon: '🏠', current: true },
  { name: 'Projects', icon: '📁', current: false },
  { name: 'Team', icon: '👥', current: false },
  { name: 'Calendar', icon: '📅', current: false },
  { name: 'Documents', icon: '📄', current: false },
  { name: 'Reports', icon: '📊', current: false },
  { name: 'Settings', icon: '⚙️', current: false },
];

export default function SidebarNavigation() {
  const [sidebarOpen, setSidebarOpen] = useState(true);

  return (

§ 9. STRATEGY/DECISION TABLES

§ 9.1 DECISION TABLE PER GESTIONE PAGINE

| Condizione | Azione |
|------------|--------|
| Pagina non esiste | Crea nuova pagina |
| Pagina esiste, ma non è pubblicata | Aggiorna pagina e pubblica |
| Pagina esiste e è pubblicata | Aggiorna pagina |

§ 9.2 DECISION TABLE PER MENU DINAMICI

| Condizione | Azione |
|------------|--------|
| Menu non esiste | Crea nuovo menu |
| Menu esiste, ma non ha elementi | Aggiungi elementi al menu |
| Menu esiste e ha elementi | Aggiorna elementi del menu |

§ 10. BEST PRACTICES

§ 10.1 ✅ DO

* Utilizzare tipi TypeScript per garantire la sicurezza dei dati
* Utilizzare Prisma per gestire il database
* Utilizzare tRPC per gestire le API
* Utilizzare componenti UI personalizzati per migliorare l'esperienza utente

§ 10.2 ❌ DON'T

* Non utilizzare tipi TypeScript per garantire la sicurezza dei dati
* Non utilizzare Prisma per gestire il database
* Non utilizzare tRPC per gestire le API
* Non utilizzare componenti UI personalizzati per migliorare l'esperienza utente

§ 11. PERFORMANCE CONSIDERATIONS

§ 11.1 OTTIMIZZAZIONE DEL DATABASE

* Utilizzare indici per migliorare la velocità di query
* Utilizzare caching per ridurre il carico del database
* Utilizzare query ottimizzate per ridurre il tempo di esecuzione

§ 11.2 OTTIMIZZAZIONE DEL CODICE

* Utilizzare funzioni pure per migliorare la velocità di esecuzione
* Utilizzare memoizzazione per ridurre il carico del codice
* Utilizzare codice ottimizzato per ridurre il tempo di esecuzione

§ 12. TESTING PATTERNS (VITEST)

§ 12.1 TEST PER COMPONENTI PRINCIPALI

typescript
import { render, fireEvent, waitFor } from '@testing-library/react';
import { PageComponent } from '../components/page';

describe('PageComponent', () => {
  it('rende il titolo della pagina', () => {
    const { getByText } = render(<PageComponent title="Titolo della pagina" />);
    expect(getByText('Titolo della pagina')).toBeInTheDocument();
  });

  it('rende il contenuto della pagina', () => {
    const { getByText } = render(<PageComponent content="Contenuto della pagina" />);
    expect(getByText('Contenuto della pagina')).toBeInTheDocument();
  });
});

§ 12.2 TEST PER API

typescript
import { createTRPCClient } from '@trpc/client';
import { createTRPCServer } from '@trpc/server';
import { z } from 'zod';
import { apiRouter } from '../server/trpc/routers/api';

describe('apiRouter', () => {
  it('rende il risultato della query', async () => {
    const client = createTRPCClient();
    const server = createTRPCServer({
      router: apiRouter,
      createContext: () => ({}),
    });

    const result = await client.query('api.example', { input: 'input' });
    expect(result).toBe('risultato');
  });
});

§ 13. COMMON PITFALLS & TROUBLESHOOTING

§ 13.1 PROBLEMI COMUNI

* Errore di tipo: assicurarsi di utilizzare tipi TypeScript corretti
* Errore di database: assicurarsi di utilizzare Prisma correttamente
* Errore di API: assicurarsi di utilizzare tRPC correttamente

§ 13.2 SOLUZIONI COMUNI

* Utilizzare il debugger per identificare l'origine dell'errore
* Utilizzare la documentazione per risolvere problemi comuni
* Utilizzare la community per chiedere aiuto

§ 14. MIGRATION/UPGRADE PATTERNS

§ 14.1 MIGRAZIONE DEL DATABASE

* Utilizzare Prisma per eseguire la migrazione del database
* Utilizzare script di migrazione per eseguire la migrazione del database

§ 14.2 UPGRADE DEL CODICE

* Utilizzare TypeScript per eseguire l'upgrade del codice
* Utilizzare funzioni di upgrade per eseguire l'upgrade del codice

§ 15. EDGE CASES

§ 15.1 GESTIONE DI ERRORI

* Utilizzare try-catch per gestire gli errori
* Utilizzare error handling per gestire gli errori

§ 15.2 GESTIONE DI INPUT INVALIDI

* Utilizzare validazione per gestire input invalidi
* Utilizzare error handling per gestire input invalidi

§ 16. ERROR HANDLING

§ 16.1 GESTIONE DI ERRORI

* Utilizzare try-catch per gestire gli errori
* Utilizzare error handling per gestire gli errori

§ 16.2 GESTIONE DI ECCEZIONI

* Utilizzare try-catch per gestire le eccezioni
* Utilizzare error handling per gestire le eccezioni

§ 17. CODE ORGANIZATION

§ 17.1 ORGANIZZAZIONE DEL CODICE

* Utilizzare una struttura di directory per organizzare il codice
* Utilizzare moduli per organizzare il codice

§ 17.2 GESTIONE DI DIPENDENZE

* Utilizzare dipendenze per gestire le dipendenze
* Utilizzare gestione di dipendenze per gestire le dipendenze

§ 18. SECURITY

§ 18.1 GESTIONE DI PASSWORD

* Utilizzare hashing per gestire le password
* Utilizzare salting per gestire le password

§ 18.2 GESTIONE DI AUTENTICAZIONE

* Utilizzare autenticazione per gestire l'accesso
* Utilizzare autorizzazione per gestire l'accesso

§ 19. DEPLOYMENT

§ 19.1 DEPLOY SU SERVER

* Utilizzare un server per deployare l'applicazione
* Utilizzare un servizio di deploy per deployare l'applicazione

§ 19.2 DEPLOY SU CLOUD

* Utilizzare un servizio cloud per deployare l'applicazione
* Utilizzare un servizio di deploy per deployare l'applicazione

§ 20. MONITORING

§ 20.1 MONITORAGGIO DELL'APPLICAZIONE

* Utilizzare strumenti di monitoraggio per monitorare l'applicazione
* Utilizzare log per monitorare l'applicazione

§ 20.2 MONITORAGGIO DEL SERVER

* Utilizzare strumenti di monitoraggio per monitorare il server
* Utilizzare log per monitorare il server

§ 21. OPTIMIZATION

§ 21.1 OTTIMIZZAZIONE DELL'APPLICAZIONE

* Utilizzare strumenti di ottimizzazione per ottimizzare l'applicazione
* Utilizzare tecniche di ottimizzazione per ottimizzare l'applicazione

§ 21.2 OTTIMIZZAZIONE DEL SERVER

* Utilizzare strumenti di ottimizzazione per ottimizzare il server
* Utilizzare tecniche di ottimizzazione per ottimizzare il server

---

## LANDING-PAGE-SECTIONS

### Panoramica
Componenti riutilizzabili per sezioni di landing page con hero, features, pricing, testimonials, CTA e FAQ.

### Implementazione Completa

```typescript
// components/landing/hero-section.tsx
"use client";

import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { ArrowRight, Play, Star } from "lucide-react";
import { cn } from "@/lib/utils";

interface HeroSectionProps {
  badge?: string;
  title: string;
  titleHighlight?: string;
  subtitle: string;
  primaryCTA: { label: string; href: string };
  secondaryCTA?: { label: string; href: string; icon?: "play" | "arrow" };
  socialProof?: {
    avatars: string[];
    text: string;
    rating?: number;
  };
  image?: {
    src: string;
    alt: string;
    width: number;
    height: number;
  };
  className?: string;
}

export function HeroSection({
  badge,
  title,
  titleHighlight,
  subtitle,
  primaryCTA,
  secondaryCTA,
  socialProof,
  image,
  className,
}: HeroSectionProps) {
  const titleParts = titleHighlight ? title.split(titleHighlight) : [title];

  return (
    <section className={cn("relative overflow-hidden py-20 md:py-32", className)}>
      {/* Background gradient */}
      <div className="absolute inset-0 -z-10 bg-[radial-gradient(45%_40%_at_50%_60%,var(--tw-gradient-from)_0%,transparent_100%)] from-primary/5" />

      <div className="container mx-auto px-4">
        <div className="mx-auto max-w-4xl text-center">
          {badge && (
            <Badge variant="secondary" className="mb-6 px-4 py-1.5 text-sm">
              {badge}
            </Badge>
          )}

          <h1 className="text-4xl font-bold tracking-tight sm:text-5xl md:text-6xl lg:text-7xl">
            {titleHighlight ? (
              <>
                {titleParts[0]}
                <span className="bg-gradient-to-r from-primary to-primary/60 bg-clip-text text-transparent">
                  {titleHighlight}
                </span>
                {titleParts[1]}
              </>
            ) : (
              title
            )}
          </h1>

          <p className="mx-auto mt-6 max-w-2xl text-lg text-muted-foreground md:text-xl">
            {subtitle}
          </p>

          <div className="mt-10 flex flex-col items-center justify-center gap-4 sm:flex-row">
            <Button asChild size="lg" className="min-w-[200px] text-base">
              <a href={primaryCTA.href}>
                {primaryCTA.label}
                <ArrowRight className="ml-2 h-5 w-5" />
              </a>
            </Button>
            {secondaryCTA && (
              <Button asChild variant="outline" size="lg" className="min-w-[200px] text-base">
                <a href={secondaryCTA.href}>
                  {secondaryCTA.icon === "play" && <Play className="mr-2 h-5 w-5" />}
                  {secondaryCTA.label}
                  {secondaryCTA.icon === "arrow" && <ArrowRight className="ml-2 h-5 w-5" />}
                </a>
              </Button>
            )}
          </div>

          {socialProof && (
            <div className="mt-10 flex items-center justify-center gap-3">
              <div className="flex -space-x-2">
                {socialProof.avatars.slice(0, 5).map((avatar, i) => (
                  <img
                    key={i}
                    src={avatar}
                    alt=""
                    className="h-8 w-8 rounded-full border-2 border-background"
                  />
                ))}
              </div>
              <div className="text-left">
                {socialProof.rating && (
                  <div className="flex items-center gap-0.5">
                    {Array.from({ length: 5 }).map((_, i) => (
                      <Star
                        key={i}
                        className={cn(
                          "h-3.5 w-3.5",
                          i < socialProof.rating! ? "fill-yellow-400 text-yellow-400" : "text-muted"
                        )}
                      />
                    ))}
                  </div>
                )}
                <p className="text-sm text-muted-foreground">{socialProof.text}</p>
              </div>
            </div>
          )}
        </div>

        {image && (
          <div className="mx-auto mt-16 max-w-5xl">
            <div className="overflow-hidden rounded-xl border shadow-2xl">
              <img
                src={image.src}
                alt={image.alt}
                width={image.width}
                height={image.height}
                className="w-full"
              />
            </div>
          </div>
        )}
      </div>
    </section>
  );
}
```

```typescript
// components/landing/pricing-section.tsx
"use client";

import { useState } from "react";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { Switch } from "@/components/ui/switch";
import { Check, X, HelpCircle } from "lucide-react";
import { cn } from "@/lib/utils";
import {
  Tooltip,
  TooltipContent,
  TooltipProvider,
  TooltipTrigger,
} from "@/components/ui/tooltip";

interface PricingFeature {
  name: string;
  included: boolean | string;
  tooltip?: string;
}

interface PricingPlan {
  id: string;
  name: string;
  description: string;
  price: { monthly: number; yearly: number };
  features: PricingFeature[];
  cta: { label: string; href: string };
  popular?: boolean;
  badge?: string;
}

interface PricingSectionProps {
  title?: string;
  subtitle?: string;
  plans: PricingPlan[];
  currency?: string;
  className?: string;
}

export function PricingSection({
  title = "Simple, transparent pricing",
  subtitle = "Choose the plan that works for you. All plans include a 14-day free trial.",
  plans,
  currency = "$",
  className,
}: PricingSectionProps) {
  const [annual, setAnnual] = useState(true);

  return (
    <section className={cn("py-20", className)}>
      <div className="container mx-auto px-4">
        <div className="mx-auto max-w-2xl text-center">
          <h2 className="text-3xl font-bold tracking-tight sm:text-4xl">{title}</h2>
          <p className="mt-4 text-lg text-muted-foreground">{subtitle}</p>

          <div className="mt-8 flex items-center justify-center gap-3">
            <span className={cn("text-sm", !annual && "font-semibold")}>Monthly</span>
            <Switch checked={annual} onCheckedChange={setAnnual} />
            <span className={cn("text-sm", annual && "font-semibold")}>
              Yearly
              <Badge variant="secondary" className="ml-2">Save 20%</Badge>
            </span>
          </div>
        </div>

        <div className="mx-auto mt-12 grid max-w-5xl gap-8 md:grid-cols-3">
          {plans.map((plan) => {
            const price = annual ? plan.price.yearly : plan.price.monthly;
            const period = annual ? "/yr" : "/mo";

            return (
              <div
                key={plan.id}
                className={cn(
                  "relative flex flex-col rounded-xl border p-6",
                  plan.popular && "border-primary shadow-lg shadow-primary/10 scale-105"
                )}
              >
                {plan.badge && (
                  <Badge className="absolute -top-3 left-1/2 -translate-x-1/2">
                    {plan.badge}
                  </Badge>
                )}

                <div className="mb-6">
                  <h3 className="text-lg font-semibold">{plan.name}</h3>
                  <p className="mt-1 text-sm text-muted-foreground">{plan.description}</p>
                </div>

                <div className="mb-6">
                  <span className="text-4xl font-bold">
                    {currency}{price}
                  </span>
                  <span className="text-muted-foreground">{period}</span>
                </div>

                <Button
                  asChild
                  variant={plan.popular ? "default" : "outline"}
                  className="mb-6 w-full"
                >
                  <a href={plan.cta.href}>{plan.cta.label}</a>
                </Button>

                <ul className="flex-1 space-y-3">
                  {plan.features.map((feature) => (
                    <li key={feature.name} className="flex items-start gap-2 text-sm">
                      {feature.included === true ? (
                        <Check className="mt-0.5 h-4 w-4 shrink-0 text-green-500" />
                      ) : feature.included === false ? (
                        <X className="mt-0.5 h-4 w-4 shrink-0 text-muted-foreground/40" />
                      ) : (
                        <Check className="mt-0.5 h-4 w-4 shrink-0 text-green-500" />
                      )}
                      <span className={cn(feature.included === false && "text-muted-foreground/60")}>
                        {typeof feature.included === "string" ? feature.included : feature.name}
                      </span>
                      {feature.tooltip && (
                        <TooltipProvider>
                          <Tooltip>
                            <TooltipTrigger>
                              <HelpCircle className="h-3.5 w-3.5 text-muted-foreground" />
                            </TooltipTrigger>
                            <TooltipContent>
                              <p className="max-w-xs text-xs">{feature.tooltip}</p>
                            </TooltipContent>
                          </Tooltip>
                        </TooltipProvider>
                      )}
                    </li>
                  ))}
                </ul>
              </div>
            );
          })}
        </div>
      </div>
    </section>
  );
}
```

```typescript
// components/landing/faq-section.tsx
"use client";

import { useState } from "react";
import { cn } from "@/lib/utils";
import { ChevronDown } from "lucide-react";

interface FAQItem {
  question: string;
  answer: string;
}

interface FAQSectionProps {
  title?: string;
  subtitle?: string;
  items: FAQItem[];
  className?: string;
}

export function FAQSection({
  title = "Frequently asked questions",
  subtitle,
  items,
  className,
}: FAQSectionProps) {
  const [openIndex, setOpenIndex] = useState<number | null>(null);

  return (
    <section className={cn("py-20", className)}>
      <div className="container mx-auto max-w-3xl px-4">
        <div className="mb-12 text-center">
          <h2 className="text-3xl font-bold tracking-tight sm:text-4xl">{title}</h2>
          {subtitle && <p className="mt-4 text-lg text-muted-foreground">{subtitle}</p>}
        </div>

        <div className="divide-y">
          {items.map((item, index) => {
            const isOpen = openIndex === index;
            return (
              <div key={index} className="py-4">
                <button
                  onClick={() => setOpenIndex(isOpen ? null : index)}
                  className="flex w-full items-center justify-between text-left"
                  aria-expanded={isOpen}
                >
                  <span className="text-base font-medium">{item.question}</span>
                  <ChevronDown
                    className={cn(
                      "h-5 w-5 shrink-0 text-muted-foreground transition-transform duration-200",
                      isOpen && "rotate-180"
                    )}
                  />
                </button>
                <div
                  className={cn(
                    "grid transition-all duration-200",
                    isOpen ? "grid-rows-[1fr] opacity-100" : "grid-rows-[0fr] opacity-0"
                  )}
                >
                  <div className="overflow-hidden">
                    <p className="pt-3 text-muted-foreground">{item.answer}</p>
                  </div>
                </div>
              </div>
            );
          })}
        </div>
      </div>
    </section>
  );
}
```

### Checklist di Verifica
- [ ] L'hero ha un CTA primario e uno secondario chiari
- [ ] Il pricing supporta toggle monthly/yearly con sconto visibile
- [ ] Le FAQ usano accordion con animazione smooth
- [ ] Il social proof ha avatar, rating e testo
- [ ] Tutte le sezioni sono responsive e accessibili


### WEBSITE TEMPLATE - Advanced Implementation Pattern #1

```typescript
// lib/website-template/pattern-1.ts
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

export class WEBSITETEMPLATEService1 {
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
// components/website-template/Manager1.tsx
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


### WEBSITE TEMPLATE - Advanced Implementation Pattern #2

```typescript
// lib/website-template/pattern-2.ts
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

export class WEBSITETEMPLATEService2 {
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
// components/website-template/Manager2.tsx
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


### WEBSITE TEMPLATE - Advanced Implementation Pattern #3

```typescript
// lib/website-template/pattern-3.ts
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

export class WEBSITETEMPLATEService3 {
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
// components/website-template/Manager3.tsx
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
