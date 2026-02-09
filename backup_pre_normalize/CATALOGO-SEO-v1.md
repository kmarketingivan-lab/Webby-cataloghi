# ═══════════════════════════════════════════════════════════════════════════════
# CATALOGO SEO v1.0
# ═══════════════════════════════════════════════════════════════════════════════
#
# GUIDA COMPLETA ALL'OTTIMIZZAZIONE SEO PER APPLICAZIONI WEB
# Technical SEO, Metadata, Structured Data, Next.js Implementation
#
# Data creazione: 2026-01-26
# Ultima modifica: 2026-01-26
#
# ═══════════════════════════════════════════════════════════════════════════════

"""
INDICE:
═══════════════════════════════════════════════════════════════════════════════
1.  SEO Fundamentals
2.  Next.js Metadata API
3.  Structured Data (JSON-LD)
4.  Sitemap & Robots.txt
5.  URL Structure & Routing
6.  Open Graph & Social Media
7.  International SEO (i18n/hreflang)
8.  Image SEO
9.  Core Web Vitals & Performance SEO
10. Semantic HTML & Accessibility
11. Internal Linking & Navigation
12. E-commerce SEO
13. Blog & Content SEO
14. Local Business SEO
15. SEO Monitoring & Audit
═══════════════════════════════════════════════════════════════════════════════
"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 1: SEO FUNDAMENTALS
# ═══════════════════════════════════════════════════════════════════════════════

SEO_FUNDAMENTALS = """

## 1.1 Come Funziona Google

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ GOOGLE: CRAWLING → INDEXING → RANKING                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ 1. CRAWLING (Googlebot)                                                    │
│    ├── Scopre nuove pagine seguendo link                                   │
│    ├── Richiede pagine ai server                                           │
│    ├── Legge robots.txt per permessi                                       │
│    └── Renderizza JavaScript (con limiti)                                  │
│                                                                             │
│ 2. INDEXING                                                                 │
│    ├── Analizza contenuto della pagina                                     │
│    ├── Estrae testo, immagini, video                                       │
│    ├── Processa metadata e structured data                                 │
│    └── Archivia in Google Index                                            │
│                                                                             │
│ 3. RANKING                                                                  │
│    ├── Determina rilevanza per query                                       │
│    ├── Valuta qualità e autorevolezza                                      │
│    ├── Considera user experience (Core Web Vitals)                         │
│    └── Personalizza risultati per utente                                   │
│                                                                             │
│ FATTORI DI RANKING PRINCIPALI (2025):                                       │
│ ├── Contenuto di qualità e rilevante                                       │
│ ├── Backlinks (link da altri siti)                                         │
│ ├── Core Web Vitals (LCP, INP, CLS)                                        │
│ ├── Mobile-friendliness                                                    │
│ ├── HTTPS                                                                  │
│ ├── Page Experience                                                        │
│ └── E-E-A-T (Experience, Expertise, Authority, Trust)                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 1.2 SPAs e SEO: La Sfida

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ PROBLEMA: Single Page Applications                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ Client-Side Rendering (CSR)                                                │
│                                                                             │
│   Browser Request                                                           │
│        │                                                                   │
│        ▼                                                                   │
│   ┌─────────────────┐                                                      │
│   │ Empty HTML Shell│  ← Googlebot vede QUESTO!                            │
│   │ <div id="root"> │                                                      │
│   │ </div>          │                                                      │
│   └─────────────────┘                                                      │
│        │                                                                   │
│        ▼ JavaScript Download & Execute                                     │
│   ┌─────────────────┐                                                      │
│   │ Full Content    │  ← L'utente vede QUESTO                              │
│   │ <h1>Products</h1>│                                                      │
│   │ <p>Description..</p>│                                                   │
│   └─────────────────┘                                                      │
│                                                                             │
│ PROBLEMI:                                                                   │
│ ├── Googlebot può non attendere JavaScript                                 │
│ ├── Contenuto non indicizzato correttamente                                │
│ ├── Meta tags dinamici non letti                                           │
│ └── Page speed peggiore (Time to Content)                                  │
│                                                                             │
│ SOLUZIONE: Server-Side Rendering (SSR) o Static Generation (SSG)           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 1.3 Next.js: La Soluzione

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ NEXT.JS RENDERING STRATEGIES                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ SSG (Static Site Generation)                                               │
│ ├── HTML generato a BUILD time                                             │
│ ├── Servito da CDN (velocissimo)                                           │
│ ├── Ideale per: blog, docs, landing pages                                  │
│ └── SEO: ⭐⭐⭐⭐⭐ (perfetto per crawlers)                                    │
│                                                                             │
│ ISR (Incremental Static Regeneration)                                      │
│ ├── HTML statico + rigenerazione periodica                                 │
│ ├── Contenuto sempre fresh senza rebuild                                   │
│ ├── Ideale per: e-commerce, news, cataloghi                                │
│ └── SEO: ⭐⭐⭐⭐⭐ (statico + aggiornato)                                     │
│                                                                             │
│ SSR (Server-Side Rendering)                                                │
│ ├── HTML generato ad ogni REQUEST                                          │
│ ├── Sempre contenuto fresh                                                 │
│ ├── Ideale per: dashboard personalizzate                                   │
│ └── SEO: ⭐⭐⭐⭐ (buono, ma più lento di SSG)                                │
│                                                                             │
│ CSR (Client-Side Rendering)                                                │
│ ├── HTML generato nel browser                                              │
│ ├── Interattività immediata                                                │
│ ├── Ideale per: app private, dashboard                                     │
│ └── SEO: ⭐⭐ (problematico per indicizzazione)                              │
│                                                                             │
│ RACCOMANDAZIONE: Usa SSG/ISR per pagine pubbliche, CSR per aree private   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 1.4 SEO Checklist Essenziale

```typescript
// Checklist SEO per ogni pagina

/*
✅ METADATA
├── Title tag unico (50-60 caratteri)
├── Meta description unica (150-160 caratteri)
├── Canonical URL definito
└── Open Graph tags completi

✅ CONTENUTO
├── H1 unico per pagina
├── Struttura headings corretta (H1 > H2 > H3)
├── Contenuto originale e di qualità
├── Alt text per tutte le immagini
└── Internal links rilevanti

✅ TECHNICAL
├── HTTPS attivo
├── Mobile-responsive
├── Core Web Vitals ottimizzati
├── Sitemap.xml presente
├── Robots.txt configurato
└── Structured data implementato

✅ URL
├── URL parlanti (slugs descrittivi)
├── No duplicate content
├── Redirect 301 per URL cambiate
└── Trailing slash consistente
*/
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 2: NEXT.JS METADATA API
# ═══════════════════════════════════════════════════════════════════════════════

NEXTJS_METADATA_API = """

## 2.1 Static Metadata

```typescript
// app/layout.tsx - Root layout metadata
import type { Metadata } from 'next';

export const metadata: Metadata = {
  // Base metadata (inherited by all pages)
  metadataBase: new URL('https://www.example.com'),
  
  // Title template
  title: {
    default: 'My Website',
    template: '%s | My Website', // %s = page title
  },
  
  // Default description
  description: 'Description of my website for search engines.',
  
  // Keywords (meno importanti oggi, ma utili)
  keywords: ['keyword1', 'keyword2', 'keyword3'],
  
  // Authors
  authors: [
    { name: 'Author Name', url: 'https://author-site.com' },
  ],
  
  // Creator
  creator: 'Company Name',
  
  // Publisher
  publisher: 'Company Name',
  
  // Robots
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1,
    },
  },
  
  // Icons
  icons: {
    icon: '/favicon.ico',
    shortcut: '/favicon-16x16.png',
    apple: '/apple-touch-icon.png',
  },
  
  // Manifest (PWA)
  manifest: '/manifest.json',
  
  // Open Graph (default)
  openGraph: {
    type: 'website',
    locale: 'it_IT',
    url: 'https://www.example.com',
    siteName: 'My Website',
    title: 'My Website',
    description: 'Description for social sharing',
    images: [
      {
        url: '/og-image.jpg',
        width: 1200,
        height: 630,
        alt: 'My Website',
      },
    ],
  },
  
  // Twitter
  twitter: {
    card: 'summary_large_image',
    site: '@username',
    creator: '@username',
  },
  
  // Verification
  verification: {
    google: 'google-site-verification-code',
    yandex: 'yandex-verification-code',
    yahoo: 'yahoo-verification-code',
  },
  
  // Alternates (for i18n)
  alternates: {
    canonical: 'https://www.example.com',
    languages: {
      'en-US': 'https://www.example.com/en',
      'it-IT': 'https://www.example.com/it',
    },
  },
  
  // Category
  category: 'technology',
};

export default function RootLayout({ children }) {
  return (
    <html lang="it">
      <body>{children}</body>
    </html>
  );
}
```

## 2.2 Page-Specific Metadata

```typescript
// app/products/page.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Products', // Diventa "Products | My Website"
  description: 'Browse our products catalog with detailed descriptions.',
  
  openGraph: {
    title: 'Our Products',
    description: 'Discover our amazing products',
    images: ['/products-og.jpg'],
  },
  
  alternates: {
    canonical: '/products',
  },
};

export default function ProductsPage() {
  return <ProductsList />;
}
```

## 2.3 Dynamic Metadata

```typescript
// app/products/[slug]/page.tsx
import type { Metadata, ResolvingMetadata } from 'next';
import { notFound } from 'next/navigation';

interface Props {
  params: Promise<{ slug: string }>;
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>;
}

// Generate metadata dinamicamente
export async function generateMetadata(
  { params, searchParams }: Props,
  parent: ResolvingMetadata
): Promise<Metadata> {
  const { slug } = await params;
  
  // Fetch product data
  const product = await getProduct(slug);
  
  if (!product) {
    return {
      title: 'Product Not Found',
    };
  }
  
  // Resolve parent metadata (per ereditare)
  const previousImages = (await parent).openGraph?.images || [];
  
  return {
    title: product.name,
    description: product.description.substring(0, 160),
    
    openGraph: {
      title: product.name,
      description: product.description,
      images: [product.image, ...previousImages],
      type: 'website',
    },
    
    twitter: {
      card: 'summary_large_image',
      title: product.name,
      description: product.description,
      images: [product.image],
    },
    
    alternates: {
      canonical: `/products/${slug}`,
    },
    
    // Product specific robots
    robots: {
      index: product.isPublished,
      follow: true,
    },
  };
}

// Generate static params per SSG
export async function generateStaticParams() {
  const products = await getAllProducts();
  
  return products.map((product) => ({
    slug: product.slug,
  }));
}

export default async function ProductPage({ params }: Props) {
  const { slug } = await params;
  const product = await getProduct(slug);
  
  if (!product) {
    notFound();
  }
  
  return <ProductDetails product={product} />;
}
```

## 2.4 Metadata per Blog Posts

```typescript
// app/blog/[slug]/page.tsx
import type { Metadata } from 'next';

export async function generateMetadata({ params }): Promise<Metadata> {
  const { slug } = await params;
  const post = await getPost(slug);
  
  if (!post) {
    return { title: 'Post Not Found' };
  }
  
  return {
    title: post.title,
    description: post.excerpt,
    
    authors: [{ name: post.author.name }],
    
    openGraph: {
      title: post.title,
      description: post.excerpt,
      type: 'article',
      publishedTime: post.publishedAt,
      modifiedTime: post.updatedAt,
      authors: [post.author.name],
      tags: post.tags,
      images: [
        {
          url: post.coverImage,
          width: 1200,
          height: 630,
          alt: post.title,
        },
      ],
    },
    
    twitter: {
      card: 'summary_large_image',
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
    },
    
    alternates: {
      canonical: `/blog/${slug}`,
    },
  };
}

export default async function BlogPost({ params }) {
  const { slug } = await params;
  const post = await getPost(slug);
  
  if (!post) {
    notFound();
  }
  
  return (
    <article>
      <h1>{post.title}</h1>
      <time dateTime={post.publishedAt}>{formatDate(post.publishedAt)}</time>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}
```

## 2.5 Title Best Practices

```typescript
// Title tag optimization

// ❌ BAD
export const metadata = {
  title: 'Home', // Troppo generico
  title: 'Best Products Ever - Buy Now - Shop Online - Free Shipping - Best Deals', // Keyword stuffing
};

// ✅ GOOD
export const metadata = {
  title: 'Handmade Leather Bags | Artisan Collection', // Descrittivo, 50-60 chars
};

// Title template pattern
// layout.tsx
export const metadata = {
  title: {
    template: '%s | Brand Name',
    default: 'Brand Name - Tagline',
  },
};

// page.tsx
export const metadata = {
  title: 'Products', // → "Products | Brand Name"
};

// Lunghezze raccomandate:
// - Title: 50-60 caratteri (Google taglia dopo ~60)
// - Description: 150-160 caratteri
// - OG Title: può essere più lungo
// - OG Description: 200-300 caratteri ok
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 3: STRUCTURED DATA (JSON-LD)
# ═══════════════════════════════════════════════════════════════════════════════

STRUCTURED_DATA = """

## 3.1 Overview Structured Data

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ STRUCTURED DATA - SCHEMA.ORG                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ COS'È                                                                       │
│ Markup standardizzato che aiuta i motori di ricerca a capire               │
│ il contenuto delle pagine in modo più preciso.                              │
│                                                                             │
│ BENEFICI                                                                    │
│ ├── Rich Results (snippet arricchiti in SERP)                              │
│ ├── Knowledge Graph integration                                            │
│ ├── Voice search optimization                                              │
│ ├── AI search visibility                                                   │
│ └── CTR migliorato (+20-30% riportato)                                     │
│                                                                             │
│ FORMATI                                                                     │
│ ├── JSON-LD (Raccomandato da Google) ✅                                     │
│ ├── Microdata (inline nell'HTML)                                           │
│ └── RDFa (attributi HTML)                                                  │
│                                                                             │
│ SCHEMA TYPES COMUNI                                                         │
│ ├── Organization     - Info azienda                                        │
│ ├── WebSite          - Sito con searchbox                                  │
│ ├── Article          - Blog posts, news                                    │
│ ├── Product          - E-commerce products                                 │
│ ├── BreadcrumbList   - Navigation breadcrumbs                              │
│ ├── FAQPage          - Domande frequenti                                   │
│ ├── HowTo            - Tutorial step-by-step                               │
│ ├── LocalBusiness    - Attività locali                                     │
│ ├── Event            - Eventi                                              │
│ └── Review           - Recensioni                                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 3.2 JSON-LD Base Implementation

```typescript
// lib/seo/structured-data.ts
import { Organization, WebSite, WithContext } from 'schema-dts';

// Organization schema
export function getOrganizationSchema(): WithContext<Organization> {
  return {
    '@context': 'https://schema.org',
    '@type': 'Organization',
    name: 'Company Name',
    url: 'https://www.example.com',
    logo: 'https://www.example.com/logo.png',
    description: 'Company description',
    foundingDate: '2020',
    founders: [
      {
        '@type': 'Person',
        name: 'Founder Name',
      },
    ],
    address: {
      '@type': 'PostalAddress',
      streetAddress: 'Via Roma 1',
      addressLocality: 'Milano',
      addressRegion: 'MI',
      postalCode: '20100',
      addressCountry: 'IT',
    },
    contactPoint: {
      '@type': 'ContactPoint',
      telephone: '+39-02-1234567',
      contactType: 'customer service',
      availableLanguage: ['Italian', 'English'],
    },
    sameAs: [
      'https://www.facebook.com/company',
      'https://www.twitter.com/company',
      'https://www.linkedin.com/company/company',
      'https://www.instagram.com/company',
    ],
  };
}

// WebSite schema with SearchAction
export function getWebSiteSchema(): WithContext<WebSite> {
  return {
    '@context': 'https://schema.org',
    '@type': 'WebSite',
    name: 'Website Name',
    url: 'https://www.example.com',
    potentialAction: {
      '@type': 'SearchAction',
      target: {
        '@type': 'EntryPoint',
        urlTemplate: 'https://www.example.com/search?q={search_term_string}',
      },
      'query-input': 'required name=search_term_string',
    },
  };
}

// Component per inserire JSON-LD
// components/JsonLd.tsx
export function JsonLd<T extends object>({ data }: { data: T }) {
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(data) }}
    />
  );
}

// Uso in layout.tsx
import { JsonLd } from '@/components/JsonLd';
import { getOrganizationSchema, getWebSiteSchema } from '@/lib/seo/structured-data';

export default function RootLayout({ children }) {
  return (
    <html lang="it">
      <head>
        <JsonLd data={getOrganizationSchema()} />
        <JsonLd data={getWebSiteSchema()} />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

## 3.3 Article Schema (Blog Posts)

```typescript
// lib/seo/article-schema.ts
import { Article, WithContext } from 'schema-dts';

interface ArticleSchemaProps {
  title: string;
  description: string;
  url: string;
  image: string;
  datePublished: string;
  dateModified: string;
  author: {
    name: string;
    url?: string;
  };
}

export function getArticleSchema(props: ArticleSchemaProps): WithContext<Article> {
  return {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: props.title,
    description: props.description,
    image: props.image,
    datePublished: props.datePublished,
    dateModified: props.dateModified,
    author: {
      '@type': 'Person',
      name: props.author.name,
      url: props.author.url,
    },
    publisher: {
      '@type': 'Organization',
      name: 'Company Name',
      logo: {
        '@type': 'ImageObject',
        url: 'https://www.example.com/logo.png',
      },
    },
    mainEntityOfPage: {
      '@type': 'WebPage',
      '@id': props.url,
    },
  };
}

// Uso in blog post page
// app/blog/[slug]/page.tsx
export default async function BlogPost({ params }) {
  const post = await getPost(params.slug);
  
  const articleSchema = getArticleSchema({
    title: post.title,
    description: post.excerpt,
    url: `https://www.example.com/blog/${params.slug}`,
    image: post.coverImage,
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
    author: {
      name: post.author.name,
      url: post.author.url,
    },
  });
  
  return (
    <>
      <JsonLd data={articleSchema} />
      <article>
        <h1>{post.title}</h1>
        {/* content */}
      </article>
    </>
  );
}
```

## 3.4 Product Schema (E-commerce)

```typescript
// lib/seo/product-schema.ts
import { Product, WithContext } from 'schema-dts';

interface ProductSchemaProps {
  name: string;
  description: string;
  image: string[];
  sku: string;
  brand: string;
  price: number;
  currency: string;
  availability: 'InStock' | 'OutOfStock' | 'PreOrder';
  url: string;
  rating?: {
    value: number;
    count: number;
  };
}

export function getProductSchema(props: ProductSchemaProps): WithContext<Product> {
  return {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: props.name,
    description: props.description,
    image: props.image,
    sku: props.sku,
    brand: {
      '@type': 'Brand',
      name: props.brand,
    },
    offers: {
      '@type': 'Offer',
      url: props.url,
      priceCurrency: props.currency,
      price: props.price,
      availability: `https://schema.org/${props.availability}`,
      seller: {
        '@type': 'Organization',
        name: 'Store Name',
      },
    },
    ...(props.rating && {
      aggregateRating: {
        '@type': 'AggregateRating',
        ratingValue: props.rating.value,
        reviewCount: props.rating.count,
      },
    }),
  };
}

// Uso in product page
export default async function ProductPage({ params }) {
  const product = await getProduct(params.slug);
  
  const productSchema = getProductSchema({
    name: product.name,
    description: product.description,
    image: product.images,
    sku: product.sku,
    brand: product.brand,
    price: product.price,
    currency: 'EUR',
    availability: product.inStock ? 'InStock' : 'OutOfStock',
    url: `https://www.example.com/products/${params.slug}`,
    rating: product.rating,
  });
  
  return (
    <>
      <JsonLd data={productSchema} />
      <ProductDetails product={product} />
    </>
  );
}
```

## 3.5 BreadcrumbList Schema

```typescript
// lib/seo/breadcrumb-schema.ts
import { BreadcrumbList, WithContext } from 'schema-dts';

interface BreadcrumbItem {
  name: string;
  url: string;
}

export function getBreadcrumbSchema(items: BreadcrumbItem[]): WithContext<BreadcrumbList> {
  return {
    '@context': 'https://schema.org',
    '@type': 'BreadcrumbList',
    itemListElement: items.map((item, index) => ({
      '@type': 'ListItem',
      position: index + 1,
      name: item.name,
      item: item.url,
    })),
  };
}

// components/Breadcrumbs.tsx
interface BreadcrumbsProps {
  items: BreadcrumbItem[];
}

export function Breadcrumbs({ items }: BreadcrumbsProps) {
  const schema = getBreadcrumbSchema(items);
  
  return (
    <>
      <JsonLd data={schema} />
      <nav aria-label="Breadcrumb">
        <ol className="flex items-center space-x-2">
          {items.map((item, index) => (
            <li key={item.url} className="flex items-center">
              {index > 0 && <span className="mx-2">/</span>}
              {index === items.length - 1 ? (
                <span aria-current="page">{item.name}</span>
              ) : (
                <a href={item.url}>{item.name}</a>
              )}
            </li>
          ))}
        </ol>
      </nav>
    </>
  );
}

// Uso
<Breadcrumbs
  items={[
    { name: 'Home', url: 'https://www.example.com' },
    { name: 'Products', url: 'https://www.example.com/products' },
    { name: 'Category', url: 'https://www.example.com/products/category' },
    { name: product.name, url: `https://www.example.com/products/${product.slug}` },
  ]}
/>
```

## 3.6 FAQPage Schema

```typescript
// lib/seo/faq-schema.ts
import { FAQPage, WithContext } from 'schema-dts';

interface FAQItem {
  question: string;
  answer: string;
}

export function getFAQSchema(faqs: FAQItem[]): WithContext<FAQPage> {
  return {
    '@context': 'https://schema.org',
    '@type': 'FAQPage',
    mainEntity: faqs.map((faq) => ({
      '@type': 'Question',
      name: faq.question,
      acceptedAnswer: {
        '@type': 'Answer',
        text: faq.answer,
      },
    })),
  };
}

// components/FAQ.tsx
export function FAQ({ faqs }: { faqs: FAQItem[] }) {
  const schema = getFAQSchema(faqs);
  
  return (
    <>
      <JsonLd data={schema} />
      <section>
        <h2>Domande Frequenti</h2>
        <dl>
          {faqs.map((faq, index) => (
            <div key={index}>
              <dt className="font-bold">{faq.question}</dt>
              <dd className="mt-2">{faq.answer}</dd>
            </div>
          ))}
        </dl>
      </section>
    </>
  );
}
```

## 3.7 LocalBusiness Schema

```typescript
// lib/seo/local-business-schema.ts
import { LocalBusiness, WithContext } from 'schema-dts';

export function getLocalBusinessSchema(): WithContext<LocalBusiness> {
  return {
    '@context': 'https://schema.org',
    '@type': 'LocalBusiness',
    '@id': 'https://www.example.com/#localbusiness',
    name: 'Business Name',
    image: 'https://www.example.com/store-photo.jpg',
    url: 'https://www.example.com',
    telephone: '+39-02-1234567',
    email: 'info@example.com',
    address: {
      '@type': 'PostalAddress',
      streetAddress: 'Via Roma 1',
      addressLocality: 'Milano',
      addressRegion: 'MI',
      postalCode: '20100',
      addressCountry: 'IT',
    },
    geo: {
      '@type': 'GeoCoordinates',
      latitude: 45.4642,
      longitude: 9.1900,
    },
    openingHoursSpecification: [
      {
        '@type': 'OpeningHoursSpecification',
        dayOfWeek: ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'],
        opens: '09:00',
        closes: '18:00',
      },
      {
        '@type': 'OpeningHoursSpecification',
        dayOfWeek: ['Saturday'],
        opens: '10:00',
        closes: '14:00',
      },
    ],
    priceRange: '€€',
    sameAs: [
      'https://www.facebook.com/business',
      'https://www.instagram.com/business',
    ],
  };
}
```

## 3.8 Validazione Structured Data

```typescript
// Tools per validare structured data:

// 1. Google Rich Results Test
// https://search.google.com/test/rich-results

// 2. Schema Markup Validator
// https://validator.schema.org/

// 3. Script per validazione in development
// scripts/validate-schema.ts
import Ajv from 'ajv';

async function validateSchema(data: object) {
  // Validate JSON structure
  const ajv = new Ajv();
  
  // Check required fields based on @type
  const type = data['@type'];
  
  const requiredFields: Record<string, string[]> = {
    Article: ['headline', 'author', 'datePublished'],
    Product: ['name', 'offers'],
    FAQPage: ['mainEntity'],
    LocalBusiness: ['name', 'address'],
  };
  
  const required = requiredFields[type] || [];
  const missing = required.filter((field) => !(field in data));
  
  if (missing.length > 0) {
    console.warn(`Missing required fields for ${type}:`, missing);
    return false;
  }
  
  return true;
}

// Test nel browser
if (process.env.NODE_ENV === 'development') {
  const scripts = document.querySelectorAll('script[type="application/ld+json"]');
  scripts.forEach((script) => {
    try {
      const data = JSON.parse(script.textContent || '');
      console.log('Schema validated:', data['@type']);
    } catch (e) {
      console.error('Invalid JSON-LD:', e);
    }
  });
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 4: SITEMAP & ROBOTS.TXT
# ═══════════════════════════════════════════════════════════════════════════════

SITEMAP_AND_ROBOTS = """

## 4.1 Sitemap.xml in Next.js

```typescript
// app/sitemap.ts - Static sitemap
import type { MetadataRoute } from 'next';

export default function sitemap(): MetadataRoute.Sitemap {
  const baseUrl = 'https://www.example.com';
  
  return [
    {
      url: baseUrl,
      lastModified: new Date(),
      changeFrequency: 'daily',
      priority: 1,
    },
    {
      url: `${baseUrl}/about`,
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.8,
    },
    {
      url: `${baseUrl}/products`,
      lastModified: new Date(),
      changeFrequency: 'weekly',
      priority: 0.9,
    },
    {
      url: `${baseUrl}/blog`,
      lastModified: new Date(),
      changeFrequency: 'daily',
      priority: 0.8,
    },
    {
      url: `${baseUrl}/contact`,
      lastModified: new Date(),
      changeFrequency: 'yearly',
      priority: 0.5,
    },
  ];
}
```

## 4.2 Dynamic Sitemap

```typescript
// app/sitemap.ts - Dynamic sitemap con dati dal database
import type { MetadataRoute } from 'next';
import { prisma } from '@/lib/db';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = 'https://www.example.com';
  
  // Fetch all products
  const products = await prisma.product.findMany({
    where: { published: true },
    select: { slug: true, updatedAt: true },
  });
  
  // Fetch all blog posts
  const posts = await prisma.post.findMany({
    where: { published: true },
    select: { slug: true, updatedAt: true },
  });
  
  // Fetch all categories
  const categories = await prisma.category.findMany({
    select: { slug: true, updatedAt: true },
  });
  
  // Static pages
  const staticPages: MetadataRoute.Sitemap = [
    {
      url: baseUrl,
      lastModified: new Date(),
      changeFrequency: 'daily',
      priority: 1,
    },
    {
      url: `${baseUrl}/about`,
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.8,
    },
    {
      url: `${baseUrl}/contact`,
      lastModified: new Date(),
      changeFrequency: 'yearly',
      priority: 0.5,
    },
  ];
  
  // Product pages
  const productPages: MetadataRoute.Sitemap = products.map((product) => ({
    url: `${baseUrl}/products/${product.slug}`,
    lastModified: product.updatedAt,
    changeFrequency: 'weekly',
    priority: 0.8,
  }));
  
  // Blog pages
  const blogPages: MetadataRoute.Sitemap = posts.map((post) => ({
    url: `${baseUrl}/blog/${post.slug}`,
    lastModified: post.updatedAt,
    changeFrequency: 'weekly',
    priority: 0.7,
  }));
  
  // Category pages
  const categoryPages: MetadataRoute.Sitemap = categories.map((category) => ({
    url: `${baseUrl}/products/category/${category.slug}`,
    lastModified: category.updatedAt,
    changeFrequency: 'weekly',
    priority: 0.6,
  }));
  
  return [
    ...staticPages,
    ...productPages,
    ...blogPages,
    ...categoryPages,
  ];
}
```

## 4.3 Sitemap Index per Siti Grandi

```typescript
// Per siti con >50,000 URL, usare sitemap index

// app/sitemap.ts
import type { MetadataRoute } from 'next';

export async function generateSitemaps() {
  // Calcola quanti sitemap servono
  const productCount = await prisma.product.count();
  const sitemapsNeeded = Math.ceil(productCount / 50000);
  
  return Array.from({ length: sitemapsNeeded }, (_, i) => ({ id: i }));
}

export default async function sitemap({
  id,
}: {
  id: number;
}): Promise<MetadataRoute.Sitemap> {
  const baseUrl = 'https://www.example.com';
  const limit = 50000;
  const offset = id * limit;
  
  const products = await prisma.product.findMany({
    where: { published: true },
    select: { slug: true, updatedAt: true },
    skip: offset,
    take: limit,
  });
  
  return products.map((product) => ({
    url: `${baseUrl}/products/${product.slug}`,
    lastModified: product.updatedAt,
  }));
}

// Questo genera:
// /sitemap/0.xml
// /sitemap/1.xml
// etc.
```

## 4.4 Sitemap con Immagini e Video

```typescript
// app/sitemap.ts - Con immagini
import type { MetadataRoute } from 'next';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = 'https://www.example.com';
  const products = await getProducts();
  
  return products.map((product) => ({
    url: `${baseUrl}/products/${product.slug}`,
    lastModified: product.updatedAt,
    changeFrequency: 'weekly',
    priority: 0.8,
    images: product.images.map((img) => img.url),
    // Per video:
    // videos: [{
    //   title: 'Product Video',
    //   thumbnail_loc: 'https://example.com/thumb.jpg',
    //   description: 'Product description',
    //   content_loc: 'https://example.com/video.mp4',
    //   duration: 120,
    // }],
  }));
}
```

## 4.5 Robots.txt

```typescript
// app/robots.ts
import type { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  const baseUrl = 'https://www.example.com';
  
  return {
    rules: [
      {
        userAgent: '*',
        allow: '/',
        disallow: [
          '/admin/',
          '/api/',
          '/private/',
          '/_next/',
          '/checkout/',
          '/cart/',
          '/account/',
          '/*.json$',
        ],
      },
      {
        userAgent: 'GPTBot',
        disallow: ['/'], // Block AI crawlers se necessario
      },
    ],
    sitemap: `${baseUrl}/sitemap.xml`,
    host: baseUrl,
  };
}

// Output:
// User-Agent: *
// Allow: /
// Disallow: /admin/
// Disallow: /api/
// Disallow: /private/
// ...
// 
// User-Agent: GPTBot
// Disallow: /
//
// Sitemap: https://www.example.com/sitemap.xml
// Host: https://www.example.com
```

## 4.6 Environment-Based Robots

```typescript
// app/robots.ts - Different rules per environment
import type { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  const baseUrl = process.env.NEXT_PUBLIC_BASE_URL || 'https://www.example.com';
  const isProduction = process.env.NODE_ENV === 'production';
  
  // Block all crawlers on staging/preview
  if (!isProduction || baseUrl.includes('staging') || baseUrl.includes('preview')) {
    return {
      rules: {
        userAgent: '*',
        disallow: '/',
      },
    };
  }
  
  // Production rules
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: ['/admin/', '/api/', '/private/'],
    },
    sitemap: `${baseUrl}/sitemap.xml`,
  };
}
```

## 4.7 next-sitemap Package

```bash
# Alternative: usare next-sitemap per configurazione avanzata
pnpm add next-sitemap
```

```javascript
// next-sitemap.config.js
/** @type {import('next-sitemap').IConfig} */
module.exports = {
  siteUrl: process.env.SITE_URL || 'https://www.example.com',
  generateRobotsTxt: true,
  generateIndexSitemap: true,
  sitemapSize: 7000,
  changefreq: 'daily',
  priority: 0.7,
  
  exclude: [
    '/admin/*',
    '/api/*',
    '/private/*',
    '/checkout',
    '/cart',
    '/account/*',
  ],
  
  alternateRefs: [
    { href: 'https://www.example.com', hreflang: 'it' },
    { href: 'https://en.example.com', hreflang: 'en' },
  ],
  
  robotsTxtOptions: {
    additionalSitemaps: [
      'https://www.example.com/server-sitemap.xml', // Dynamic sitemap
    ],
    policies: [
      { userAgent: '*', allow: '/' },
      { userAgent: 'GPTBot', disallow: '/' },
    ],
  },
  
  transform: async (config, path) => {
    // Custom priority per path
    let priority = config.priority;
    
    if (path === '/') {
      priority = 1.0;
    } else if (path.startsWith('/products/')) {
      priority = 0.8;
    } else if (path.startsWith('/blog/')) {
      priority = 0.7;
    }
    
    return {
      loc: path,
      changefreq: config.changefreq,
      priority,
      lastmod: config.autoLastmod ? new Date().toISOString() : undefined,
    };
  },
  
  additionalPaths: async (config) => {
    // Aggiungi URL dinamici
    const products = await fetchAllProducts();
    
    return products.map((product) => ({
      loc: `/products/${product.slug}`,
      lastmod: product.updatedAt,
      changefreq: 'weekly',
      priority: 0.8,
    }));
  },
};
```

```json
// package.json
{
  "scripts": {
    "build": "next build",
    "postbuild": "next-sitemap"
  }
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 5: URL STRUCTURE & ROUTING
# ═══════════════════════════════════════════════════════════════════════════════

URL_STRUCTURE_AND_ROUTING = """

## 5.1 URL Best Practices

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ URL OPTIMIZATION                                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ ✅ BUONE PRATICHE                                                           │
│ ├── URL parlanti e descrittivi                                             │
│ │   /products/handmade-leather-bag                                         │
│ ├── Parole chiave rilevanti nel path                                       │
│ ├── Lowercase (minuscole)                                                  │
│ ├── Trattini per separare parole (non underscore)                          │
│ ├── Brevi e concisi (sotto 75 caratteri)                                   │
│ └── Struttura gerarchica logica                                            │
│     /category/subcategory/product-name                                     │
│                                                                             │
│ ❌ CATTIVE PRATICHE                                                         │
│ ├── ID numerici senza contesto                                             │
│ │   /products/12345                                                        │
│ ├── Query strings complesse                                                │
│ │   /search?id=123&ref=abc&utm=xyz                                         │
│ ├── Caratteri speciali                                                     │
│ │   /products/bag%20leather                                                │
│ ├── UPPERCASE                                                              │
│ │   /Products/LEATHER-BAG                                                  │
│ ├── Underscore                                                             │
│ │   /products/leather_bag                                                  │
│ └── URL troppo lunghi                                                      │
│     /shop/all-products/bags/leather/handmade/italian-crafted-genuine-...   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 5.2 Next.js Routing SEO-Friendly

```typescript
// app/products/[slug]/page.tsx
// URL: /products/handmade-leather-bag

// Slug generation utility
// lib/utils/slug.ts
export function generateSlug(text: string): string {
  return text
    .toLowerCase()
    .normalize('NFD')
    .replace(/[\u0300-\u036f]/g, '') // Rimuove accenti
    .replace(/[^a-z0-9]+/g, '-')     // Sostituisce caratteri speciali con -
    .replace(/^-+|-+$/g, '')         // Rimuove - iniziali e finali
    .substring(0, 60);               // Limita lunghezza
}

// Esempio di utilizzo
const title = "Borsa in Pelle Artigianale Italiana";
const slug = generateSlug(title); // "borsa-in-pelle-artigianale-italiana"
```

## 5.3 Canonical URLs

```typescript
// app/products/[slug]/page.tsx
import type { Metadata } from 'next';

export async function generateMetadata({ params }): Promise<Metadata> {
  const { slug } = await params;
  
  return {
    alternates: {
      canonical: `https://www.example.com/products/${slug}`,
    },
  };
}

// Per pagine con query params (filtering, sorting)
// app/products/page.tsx
export async function generateMetadata({ searchParams }): Promise<Metadata> {
  // Canonical sempre SENZA query params
  // Questo evita duplicate content per /products?sort=price, /products?filter=leather, etc.
  return {
    alternates: {
      canonical: 'https://www.example.com/products',
    },
  };
}

// Layout-level canonical base
// app/layout.tsx
export const metadata: Metadata = {
  metadataBase: new URL('https://www.example.com'),
  alternates: {
    canonical: './', // Relative to current path
  },
};
```

## 5.4 Trailing Slash Consistency

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Scelta: con o senza trailing slash (scegli UNO e mantienilo)
  trailingSlash: false, // /products (raccomandato)
  // trailingSlash: true, // /products/
};

module.exports = nextConfig;
```

## 5.5 Redirects per URL Changes

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  async redirects() {
    return [
      // Redirect permanente (301) - cambio URL definitivo
      {
        source: '/old-page',
        destination: '/new-page',
        permanent: true,
      },
      // Redirect con wildcard
      {
        source: '/blog/:slug*',
        destination: '/articles/:slug*',
        permanent: true,
      },
      // Redirect con regex
      {
        source: '/product/:id(\\d+)',
        destination: '/products/:id',
        permanent: true,
      },
      // Redirect basato su locale
      {
        source: '/en/about',
        destination: '/about',
        permanent: true,
        locale: false,
      },
    ];
  },
};

module.exports = nextConfig;
```

## 5.6 Dynamic Redirects

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  
  // Redirect vecchi URL prodotto
  if (pathname.startsWith('/shop/')) {
    const newPath = pathname.replace('/shop/', '/products/');
    return NextResponse.redirect(new URL(newPath, request.url), 301);
  }
  
  // Force lowercase URLs
  if (pathname !== pathname.toLowerCase()) {
    return NextResponse.redirect(
      new URL(pathname.toLowerCase(), request.url),
      301
    );
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

## 5.7 Pagination SEO

```typescript
// app/blog/page.tsx
import type { Metadata } from 'next';

interface BlogPageProps {
  searchParams: Promise<{ page?: string }>;
}

export async function generateMetadata({ searchParams }: BlogPageProps): Promise<Metadata> {
  const { page } = await searchParams;
  const currentPage = parseInt(page || '1');
  
  return {
    title: currentPage > 1 ? `Blog - Pagina ${currentPage}` : 'Blog',
    
    alternates: {
      // Canonical SENZA page param per pagina 1
      canonical: currentPage === 1 
        ? 'https://www.example.com/blog' 
        : `https://www.example.com/blog?page=${currentPage}`,
    },
    
    // Altri tag utili per pagination
    other: {
      // Prev/Next per crawlers (deprecated ma ancora utili)
      ...(currentPage > 1 && {
        'link rel="prev"': `https://www.example.com/blog?page=${currentPage - 1}`,
      }),
    },
  };
}

// Component con link prev/next
function Pagination({ currentPage, totalPages }: { currentPage: number; totalPages: number }) {
  return (
    <nav aria-label="Pagination">
      {currentPage > 1 && (
        <Link href={`/blog?page=${currentPage - 1}`} rel="prev">
          Previous
        </Link>
      )}
      
      {/* Page numbers */}
      {Array.from({ length: totalPages }, (_, i) => i + 1).map((page) => (
        <Link
          key={page}
          href={`/blog?page=${page}`}
          aria-current={page === currentPage ? 'page' : undefined}
        >
          {page}
        </Link>
      ))}
      
      {currentPage < totalPages && (
        <Link href={`/blog?page=${currentPage + 1}`} rel="next">
          Next
        </Link>
      )}
    </nav>
  );
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 6: OPEN GRAPH & SOCIAL MEDIA
# ═══════════════════════════════════════════════════════════════════════════════

OPEN_GRAPH_AND_SOCIAL = """

## 6.1 Open Graph Protocol

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ OPEN GRAPH META TAGS                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ Open Graph (OG) controlla come i contenuti appaiono quando condivisi       │
│ su social media (Facebook, LinkedIn, WhatsApp, etc.)                        │
│                                                                             │
│ TAG ESSENZIALI                                                              │
│ ├── og:title         - Titolo della pagina                                 │
│ ├── og:description   - Descrizione breve                                   │
│ ├── og:image         - Immagine di anteprima                               │
│ ├── og:url           - URL canonico                                        │
│ └── og:type          - Tipo di contenuto                                   │
│                                                                             │
│ OG:TYPE VALUES                                                              │
│ ├── website          - Homepage, landing pages                             │
│ ├── article          - Blog posts, news                                    │
│ ├── product          - Pagine prodotto                                     │
│ ├── profile          - Profili utente                                      │
│ └── video.other      - Pagine video                                        │
│                                                                             │
│ IMAGE SPECIFICATIONS                                                        │
│ ├── Dimensioni: 1200x630 px (rapporto 1.91:1)                             │
│ ├── Min: 600x315 px                                                        │
│ ├── Max file size: 8MB                                                     │
│ ├── Formato: JPG, PNG, GIF                                                 │
│ └── Importante: Testo leggibile anche se ridimensionato                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 6.2 Open Graph in Next.js

```typescript
// app/layout.tsx - Default OG per tutto il sito
import type { Metadata } from 'next';

export const metadata: Metadata = {
  openGraph: {
    type: 'website',
    locale: 'it_IT',
    url: 'https://www.example.com',
    siteName: 'My Website',
    title: {
      default: 'My Website',
      template: '%s | My Website',
    },
    description: 'Description for social sharing',
    images: [
      {
        url: '/og-default.jpg',
        width: 1200,
        height: 630,
        alt: 'My Website',
        type: 'image/jpeg',
      },
    ],
  },
};
```

## 6.3 Page-Specific Open Graph

```typescript
// app/blog/[slug]/page.tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await getPost(params.slug);
  
  return {
    openGraph: {
      type: 'article',
      title: post.title,
      description: post.excerpt,
      url: `https://www.example.com/blog/${params.slug}`,
      publishedTime: post.publishedAt,
      modifiedTime: post.updatedAt,
      authors: [post.author.name],
      tags: post.tags,
      section: post.category,
      images: [
        {
          url: post.coverImage,
          width: 1200,
          height: 630,
          alt: post.title,
        },
        // Immagine quadrata per alcuni social
        {
          url: post.thumbnailImage,
          width: 600,
          height: 600,
          alt: post.title,
        },
      ],
    },
  };
}
```

## 6.4 Twitter Cards

```typescript
// app/layout.tsx
export const metadata: Metadata = {
  twitter: {
    card: 'summary_large_image', // o 'summary' per immagini piccole
    site: '@siteusername',       // Account Twitter del sito
    creator: '@creatorusername', // Account Twitter dell'autore
    title: 'My Website',
    description: 'Description for Twitter',
    images: ['/twitter-image.jpg'],
  },
};

// Per pagina specifica
// app/blog/[slug]/page.tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await getPost(params.slug);
  
  return {
    twitter: {
      card: 'summary_large_image',
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
      creator: post.author.twitterHandle,
    },
  };
}
```

## 6.5 Dynamic OG Images

```typescript
// app/api/og/route.tsx
import { ImageResponse } from 'next/og';
import { NextRequest } from 'next/server';

export const runtime = 'edge';

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const title = searchParams.get('title') || 'My Website';
  const description = searchParams.get('description') || '';
  
  return new ImageResponse(
    (
      <div
        style={{
          height: '100%',
          width: '100%',
          display: 'flex',
          flexDirection: 'column',
          alignItems: 'center',
          justifyContent: 'center',
          backgroundColor: '#1a1a2e',
          padding: '40px',
        }}
      >
        <div
          style={{
            display: 'flex',
            flexDirection: 'column',
            alignItems: 'flex-start',
            width: '100%',
            maxWidth: '900px',
          }}
        >
          {/* Logo */}
          <img
            src="https://www.example.com/logo-white.png"
            width={150}
            height={50}
            style={{ marginBottom: '40px' }}
          />
          
          {/* Title */}
          <div
            style={{
              fontSize: '60px',
              fontWeight: 'bold',
              color: 'white',
              lineHeight: 1.2,
              marginBottom: '20px',
            }}
          >
            {title}
          </div>
          
          {/* Description */}
          {description && (
            <div
              style={{
                fontSize: '28px',
                color: '#a0a0a0',
                lineHeight: 1.4,
              }}
            >
              {description}
            </div>
          )}
        </div>
      </div>
    ),
    {
      width: 1200,
      height: 630,
    }
  );
}

// Uso nei metadata
export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await getPost(params.slug);
  
  const ogImageUrl = new URL('/api/og', 'https://www.example.com');
  ogImageUrl.searchParams.set('title', post.title);
  ogImageUrl.searchParams.set('description', post.excerpt);
  
  return {
    openGraph: {
      images: [
        {
          url: ogImageUrl.toString(),
          width: 1200,
          height: 630,
          alt: post.title,
        },
      ],
    },
  };
}
```

## 6.6 Testing Social Previews

```typescript
// Tools per testare le preview social:

// 1. Facebook Sharing Debugger
// https://developers.facebook.com/tools/debug/

// 2. Twitter Card Validator
// https://cards-dev.twitter.com/validator

// 3. LinkedIn Post Inspector
// https://www.linkedin.com/post-inspector/

// 4. Open Graph Preview (locale)
// Usa estensioni browser come "SEO META in 1 CLICK"

// Script per verificare OG tags
// scripts/check-og-tags.ts
async function checkOGTags(url: string) {
  const response = await fetch(url);
  const html = await response.text();
  
  const ogTags = {
    title: html.match(/<meta property="og:title" content="([^"]+)"/)?.[1],
    description: html.match(/<meta property="og:description" content="([^"]+)"/)?.[1],
    image: html.match(/<meta property="og:image" content="([^"]+)"/)?.[1],
    url: html.match(/<meta property="og:url" content="([^"]+)"/)?.[1],
    type: html.match(/<meta property="og:type" content="([^"]+)"/)?.[1],
  };
  
  console.log('Open Graph Tags:', ogTags);
  
  // Validazione
  if (!ogTags.title) console.warn('Missing og:title');
  if (!ogTags.description) console.warn('Missing og:description');
  if (!ogTags.image) console.warn('Missing og:image');
  
  return ogTags;
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 7: INTERNATIONAL SEO (i18n/hreflang)
# ═══════════════════════════════════════════════════════════════════════════════

INTERNATIONAL_SEO = """

## 7.1 Strategie i18n

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ STRATEGIE URL INTERNAZIONALI                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ 1. SUBPATH (Raccomandato)                                                  │
│    example.com/it/products                                                 │
│    example.com/en/products                                                 │
│    example.com/de/products                                                 │
│    ✅ Facile da implementare                                               │
│    ✅ Tutti i contenuti sotto un dominio                                   │
│    ✅ SEO consolidato                                                      │
│                                                                             │
│ 2. SUBDOMAINS                                                               │
│    it.example.com/products                                                 │
│    en.example.com/products                                                 │
│    de.example.com/products                                                 │
│    ⚠️ SEO separato per ogni subdomain                                     │
│    ✅ Separazione server geografica possibile                              │
│                                                                             │
│ 3. COUNTRY-CODE TOP LEVEL DOMAINS (ccTLD)                                  │
│    example.it/products                                                     │
│    example.co.uk/products                                                  │
│    example.de/products                                                     │
│    ❌ Costoso (più domini)                                                 │
│    ❌ SEO da costruire per ogni dominio                                    │
│    ✅ Forte segnale geografico                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 7.2 Next.js i18n Configuration

```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  i18n: {
    locales: ['it', 'en', 'de', 'fr', 'es'],
    defaultLocale: 'it',
    localeDetection: true, // Auto-detect da browser
  },
};

module.exports = nextConfig;

// Per App Router (Next.js 13+), usa route segments
// app/[locale]/layout.tsx
// app/[locale]/page.tsx
// app/[locale]/products/page.tsx
```

## 7.3 App Router i18n Setup

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { match } from '@formatjs/intl-localematcher';
import Negotiator from 'negotiator';

const locales = ['it', 'en', 'de', 'fr', 'es'];
const defaultLocale = 'it';

function getLocale(request: NextRequest): string {
  const negotiatorHeaders: Record<string, string> = {};
  request.headers.forEach((value, key) => (negotiatorHeaders[key] = value));
  
  const languages = new Negotiator({ headers: negotiatorHeaders }).languages();
  return match(languages, locales, defaultLocale);
}

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  
  // Check if pathname has locale
  const pathnameHasLocale = locales.some(
    (locale) => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  );
  
  if (pathnameHasLocale) return;
  
  // Redirect to locale path
  const locale = getLocale(request);
  request.nextUrl.pathname = `/${locale}${pathname}`;
  
  return NextResponse.redirect(request.nextUrl);
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico|.*\\..*).*)'],
};
```

## 7.4 Hreflang Tags

```typescript
// app/[locale]/layout.tsx
import type { Metadata } from 'next';

interface LocaleLayoutProps {
  params: Promise<{ locale: string }>;
  children: React.ReactNode;
}

export async function generateMetadata({ params }: LocaleLayoutProps): Promise<Metadata> {
  const { locale } = await params;
  const baseUrl = 'https://www.example.com';
  
  return {
    alternates: {
      canonical: `${baseUrl}/${locale}`,
      languages: {
        'it': `${baseUrl}/it`,
        'en': `${baseUrl}/en`,
        'de': `${baseUrl}/de`,
        'fr': `${baseUrl}/fr`,
        'es': `${baseUrl}/es`,
        'x-default': `${baseUrl}/it`, // Default fallback
      },
    },
  };
}

// Output HTML:
// <link rel="alternate" hreflang="it" href="https://www.example.com/it" />
// <link rel="alternate" hreflang="en" href="https://www.example.com/en" />
// <link rel="alternate" hreflang="de" href="https://www.example.com/de" />
// <link rel="alternate" hreflang="x-default" href="https://www.example.com/it" />
```

## 7.5 Dynamic Hreflang per Pagine

```typescript
// app/[locale]/products/[slug]/page.tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const { locale, slug } = await params;
  const baseUrl = 'https://www.example.com';
  
  // Fetch product per verificare che esista in tutte le lingue
  const product = await getProduct(slug, locale);
  const availableLocales = await getProductAvailableLocales(slug);
  
  // Genera hreflang solo per lingue dove il prodotto esiste
  const languages: Record<string, string> = {};
  
  for (const loc of availableLocales) {
    languages[loc] = `${baseUrl}/${loc}/products/${slug}`;
  }
  
  // Aggiungi x-default
  languages['x-default'] = `${baseUrl}/it/products/${slug}`;
  
  return {
    title: product.name,
    description: product.description,
    alternates: {
      canonical: `${baseUrl}/${locale}/products/${slug}`,
      languages,
    },
  };
}
```

## 7.6 Hreflang in Sitemap

```typescript
// app/sitemap.ts
import type { MetadataRoute } from 'next';

const locales = ['it', 'en', 'de', 'fr', 'es'];
const baseUrl = 'https://www.example.com';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const products = await getAllProducts();
  
  // Generate entries with all language alternates
  const productEntries = products.flatMap((product) => {
    return locales.map((locale) => ({
      url: `${baseUrl}/${locale}/products/${product.slug}`,
      lastModified: product.updatedAt,
      changeFrequency: 'weekly' as const,
      priority: 0.8,
      alternates: {
        languages: Object.fromEntries(
          locales.map((loc) => [loc, `${baseUrl}/${loc}/products/${product.slug}`])
        ),
      },
    }));
  });
  
  // Static pages per ogni lingua
  const staticPages = ['', '/about', '/contact'];
  
  const staticEntries = staticPages.flatMap((page) => {
    return locales.map((locale) => ({
      url: `${baseUrl}/${locale}${page}`,
      lastModified: new Date(),
      changeFrequency: 'monthly' as const,
      priority: page === '' ? 1.0 : 0.8,
      alternates: {
        languages: Object.fromEntries(
          locales.map((loc) => [loc, `${baseUrl}/${loc}${page}`])
        ),
      },
    }));
  });
  
  return [...staticEntries, ...productEntries];
}
```

## 7.7 Language Switcher SEO-Friendly

```typescript
// components/LanguageSwitcher.tsx
'use client';

import { usePathname, useRouter } from 'next/navigation';
import { useParams } from 'next/navigation';

const locales = [
  { code: 'it', name: 'Italiano', flag: '🇮🇹' },
  { code: 'en', name: 'English', flag: '🇬🇧' },
  { code: 'de', name: 'Deutsch', flag: '🇩🇪' },
  { code: 'fr', name: 'Français', flag: '🇫🇷' },
  { code: 'es', name: 'Español', flag: '🇪🇸' },
];

export function LanguageSwitcher() {
  const pathname = usePathname();
  const router = useRouter();
  const params = useParams();
  const currentLocale = params.locale as string;
  
  const switchLocale = (newLocale: string) => {
    // Replace current locale in path
    const newPath = pathname.replace(`/${currentLocale}`, `/${newLocale}`);
    router.push(newPath);
  };
  
  return (
    <nav aria-label="Language selector">
      <ul className="flex space-x-2">
        {locales.map((locale) => (
          <li key={locale.code}>
            <button
              onClick={() => switchLocale(locale.code)}
              aria-current={locale.code === currentLocale ? 'true' : undefined}
              lang={locale.code}
              hrefLang={locale.code}
              className={locale.code === currentLocale ? 'font-bold' : ''}
            >
              {locale.flag} {locale.name}
            </button>
          </li>
        ))}
      </ul>
    </nav>
  );
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 8: IMAGE SEO
# ═══════════════════════════════════════════════════════════════════════════════

IMAGE_SEO = """

## 8.1 Image Optimization per SEO

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ IMAGE SEO BEST PRACTICES                                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ ALT TEXT                                                                    │
│ ├── Descrivi cosa mostra l'immagine                                        │
│ ├── Includi keyword quando naturale                                        │
│ ├── Max 125 caratteri                                                      │
│ ├── No "immagine di..." o "foto di..."                                     │
│ └── Alt vuoto per immagini decorative                                      │
│                                                                             │
│ FILE NAMING                                                                 │
│ ├── Nomi descrittivi: handmade-leather-bag.jpg                             │
│ ├── Trattini per separare parole                                           │
│ ├── Lowercase                                                              │
│ └── No: IMG_12345.jpg, DSC00123.png                                        │
│                                                                             │
│ FORMATO E DIMENSIONI                                                        │
│ ├── WebP/AVIF preferiti (smaller, same quality)                            │
│ ├── JPEG per foto                                                          │
│ ├── PNG per grafiche/trasparenze                                           │
│ ├── SVG per icone/loghi                                                    │
│ └── Dimensioni appropriate per uso                                         │
│                                                                             │
│ LAZY LOADING                                                                │
│ ├── loading="lazy" per immagini below-the-fold                             │
│ ├── No lazy load per LCP image (hero)                                      │
│ └── Next.js Image gestisce automaticamente                                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 8.2 Next.js Image Component

```typescript
// components/ProductImage.tsx
import Image from 'next/image';

interface ProductImageProps {
  src: string;
  alt: string;
  productName: string;
  priority?: boolean;
}

export function ProductImage({ src, alt, productName, priority = false }: ProductImageProps) {
  return (
    <figure>
      <Image
        src={src}
        alt={alt || `${productName} - product image`}
        width={800}
        height={600}
        priority={priority} // true per LCP image
        placeholder="blur"
        blurDataURL="/placeholder.jpg" // o genera con plaiceholder
        sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
        className="rounded-lg"
      />
      <figcaption className="sr-only">{alt}</figcaption>
    </figure>
  );
}

// Hero image (above the fold - priorità massima)
<Image
  src="/hero-image.jpg"
  alt="Handcrafted leather bags collection"
  width={1920}
  height={1080}
  priority={true} // No lazy loading!
  fetchPriority="high"
/>

// Gallery images (below the fold - lazy load)
<Image
  src="/product-gallery-1.jpg"
  alt="Bag interior detail"
  width={400}
  height={300}
  loading="lazy"
/>
```

## 8.3 Alt Text Best Practices

```typescript
// ✅ BUONI esempi di alt text
const goodAltExamples = {
  product: 'Brown leather messenger bag with brass buckles',
  person: 'Chef Maria preparing homemade pasta',
  infographic: 'Chart showing sales growth from 2020 to 2024',
  screenshot: 'Dashboard showing analytics overview',
  logo: 'Company Name logo', // Per logo principale
  decorative: '', // Vuoto per immagini decorative
};

// ❌ CATTIVI esempi di alt text
const badAltExamples = {
  vague: 'Image', // Troppo generico
  redundant: 'Photo of a bag', // "Photo of" non necessario
  keyword_stuffing: 'leather bag best leather bag buy leather bag cheap', // Spam
  too_long: 'This is a beautiful handcrafted Italian leather bag made by artisans...', // >125 chars
  filename: 'IMG_20240115_123456.jpg', // File name non descrittivo
};

// Component con alt text generator
interface ImageWithSEOProps {
  src: string;
  productName: string;
  productType: string;
  variant?: string;
  width: number;
  height: number;
}

export function ImageWithSEO({ 
  src, 
  productName, 
  productType, 
  variant,
  width,
  height 
}: ImageWithSEOProps) {
  // Generate descriptive alt text
  const altText = variant 
    ? `${productName} ${productType} - ${variant}`
    : `${productName} ${productType}`;
  
  return (
    <Image
      src={src}
      alt={altText}
      width={width}
      height={height}
      title={productName} // Tooltip on hover
    />
  );
}
```

## 8.4 Image Sitemap

```typescript
// Sitemap con immagini per Google Images
// app/sitemap.ts
import type { MetadataRoute } from 'next';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const products = await getAllProducts();
  
  return products.map((product) => ({
    url: `https://www.example.com/products/${product.slug}`,
    lastModified: product.updatedAt,
    images: product.images.map((img) => img.url),
    // Alternativa con più dettagli:
    // images: product.images.map((img) => ({
    //   url: img.url,
    //   title: img.alt,
    //   caption: img.description,
    // })),
  }));
}
```

## 8.5 Structured Data per Immagini

```typescript
// Product con immagini multiple per Google Images
function getProductSchemaWithImages(product: Product) {
  return {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    description: product.description,
    image: product.images.map((img) => ({
      '@type': 'ImageObject',
      url: img.url,
      width: img.width,
      height: img.height,
      caption: img.alt,
    })),
    // ... altri campi
  };
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 9: CORE WEB VITALS & PERFORMANCE SEO
# ═══════════════════════════════════════════════════════════════════════════════

CORE_WEB_VITALS_SEO = """

## 9.1 Performance come Ranking Factor

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ CORE WEB VITALS E SEO                                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ Google usa Core Web Vitals come segnale di ranking dal 2021.               │
│ INP ha sostituito FID come metrica nel Marzo 2024.                         │
│                                                                             │
│ METRICHE (2025)                                                             │
│ ├── LCP (Largest Contentful Paint) ≤ 2.5s                                  │
│ ├── INP (Interaction to Next Paint) ≤ 200ms                                │
│ └── CLS (Cumulative Layout Shift) ≤ 0.1                                    │
│                                                                             │
│ IMPATTO SEO                                                                 │
│ ├── 10-15% peso nel ranking (stimato)                                      │
│ ├── Può fare differenza tra pagina 1 e pagina 2                            │
│ ├── Migliora user experience → bounce rate più basso                       │
│ └── Mobile-first indexing: performance mobile critica                      │
│                                                                             │
│ COME GOOGLE MISURA                                                          │
│ ├── Chrome User Experience Report (CrUX)                                   │
│ ├── Dati reali da utenti Chrome                                            │
│ ├── 75th percentile degli utenti                                           │
│ └── Dati aggregati per 28 giorni                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 9.2 LCP Optimization per SEO

```typescript
// Ottimizzazioni LCP critiche per SEO

// 1. Preload hero image
// app/layout.tsx
export const metadata: Metadata = {
  // Preconnect a domini esterni
  other: {
    'link rel="preconnect" href="https://cdn.example.com"': '',
    'link rel="dns-prefetch" href="https://cdn.example.com"': '',
  },
};

// 2. Priority per LCP element
// components/HeroSection.tsx
import Image from 'next/image';

export function HeroSection() {
  return (
    <section>
      <Image
        src="/hero.jpg"
        alt="Hero description"
        width={1920}
        height={1080}
        priority // Critical for LCP!
        fetchPriority="high"
      />
    </section>
  );
}

// 3. Inline critical CSS
// next.config.js
module.exports = {
  experimental: {
    optimizeCss: true, // Abilita ottimizzazione CSS
  },
};
```

## 9.3 INP Optimization per SEO

```typescript
// INP = Interaction to Next Paint
// Ottimizza la reattività della pagina

// 1. Evita long tasks - usa dynamic imports
import dynamic from 'next/dynamic';

// Lazy load componenti pesanti
const HeavyChart = dynamic(() => import('./HeavyChart'), {
  loading: () => <ChartSkeleton />,
  ssr: false,
});

// 2. Usa Web Workers per calcoli pesanti
// lib/worker.ts
if (typeof window !== 'undefined') {
  const worker = new Worker(new URL('./heavy-calc.worker.ts', import.meta.url));
  
  worker.postMessage({ data: largeDataset });
  worker.onmessage = (e) => {
    setResult(e.data);
  };
}

// 3. Debounce/throttle eventi frequenti
import { useDebouncedCallback } from 'use-debounce';

function SearchInput() {
  const debouncedSearch = useDebouncedCallback(
    (value) => search(value),
    300
  );
  
  return (
    <input onChange={(e) => debouncedSearch(e.target.value)} />
  );
}

// 4. React optimization
import { memo, useMemo, useCallback } from 'react';

const ExpensiveComponent = memo(function ExpensiveComponent({ data }) {
  const processedData = useMemo(() => processData(data), [data]);
  const handleClick = useCallback(() => doSomething(), []);
  
  return <div onClick={handleClick}>{processedData}</div>;
});
```

## 9.4 CLS Optimization per SEO

```typescript
// CLS = Cumulative Layout Shift
// Stabilità visiva della pagina

// 1. Sempre dimensioni esplicite per media
<Image
  src="/product.jpg"
  alt="Product"
  width={400}  // Sempre specificare!
  height={300} // Sempre specificare!
/>

// 2. Skeleton loaders per contenuti dinamici
function ProductSkeleton() {
  return (
    <div className="animate-pulse">
      <div className="h-64 w-full bg-gray-200 rounded" />
      <div className="h-6 w-3/4 bg-gray-200 rounded mt-4" />
      <div className="h-4 w-1/2 bg-gray-200 rounded mt-2" />
    </div>
  );
}

// 3. Riservare spazio per ads/embeds
function AdSlot() {
  return (
    <div 
      className="ad-container"
      style={{ minHeight: '250px' }} // Altezza minima riservata
    >
      {/* Ad loads here */}
    </div>
  );
}

// 4. Font loading senza FOUT/FOIT
// app/layout.tsx
import { Inter } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap', // Mostra fallback subito
  preload: true,
});

export default function RootLayout({ children }) {
  return (
    <html className={inter.className}>
      <body>{children}</body>
    </html>
  );
}
```

## 9.5 PageSpeed Insights Integration

```typescript
// Script per monitoring PageSpeed in CI/CD
// scripts/check-pagespeed.ts
import psi from 'psi';

const PAGES_TO_TEST = [
  { url: 'https://www.example.com/', name: 'Homepage' },
  { url: 'https://www.example.com/products', name: 'Products' },
  { url: 'https://www.example.com/blog', name: 'Blog' },
];

const THRESHOLDS = {
  performance: 90,
  accessibility: 95,
  bestPractices: 90,
  seo: 95,
};

async function runPageSpeedTest(url: string) {
  const result = await psi(url, {
    strategy: 'mobile', // Mobile-first!
    key: process.env.PAGESPEED_API_KEY,
  });
  
  return {
    performance: result.data.lighthouseResult.categories.performance.score * 100,
    accessibility: result.data.lighthouseResult.categories.accessibility.score * 100,
    bestPractices: result.data.lighthouseResult.categories['best-practices'].score * 100,
    seo: result.data.lighthouseResult.categories.seo.score * 100,
    coreWebVitals: {
      lcp: result.data.lighthouseResult.audits['largest-contentful-paint'].displayValue,
      cls: result.data.lighthouseResult.audits['cumulative-layout-shift'].displayValue,
      // INP non disponibile in lab test, usa field data
    },
  };
}

async function main() {
  for (const page of PAGES_TO_TEST) {
    console.log(`Testing: ${page.name}`);
    const scores = await runPageSpeedTest(page.url);
    
    console.log(`  Performance: ${scores.performance}`);
    console.log(`  SEO: ${scores.seo}`);
    console.log(`  LCP: ${scores.coreWebVitals.lcp}`);
    console.log(`  CLS: ${scores.coreWebVitals.cls}`);
    
    // Fail CI if below threshold
    if (scores.performance < THRESHOLDS.performance) {
      console.error(`❌ Performance below threshold: ${scores.performance} < ${THRESHOLDS.performance}`);
      process.exit(1);
    }
    
    if (scores.seo < THRESHOLDS.seo) {
      console.error(`❌ SEO score below threshold: ${scores.seo} < ${THRESHOLDS.seo}`);
      process.exit(1);
    }
  }
  
  console.log('✅ All pages passed performance checks');
}

main().catch(console.error);
```

"""



# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 10: SEMANTIC HTML & ACCESSIBILITY
# ═══════════════════════════════════════════════════════════════════════════════

SEMANTIC_HTML_ACCESSIBILITY = """

## 10.1 HTML Semantico per SEO

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ SEMANTIC HTML E SEO                                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ HTML semantico aiuta i motori di ricerca a capire la struttura             │
│ e la gerarchia del contenuto della pagina.                                  │
│                                                                             │
│ ELEMENTI STRUTTURALI                                                        │
│ ├── <header>    - Intestazione pagina/sezione                              │
│ ├── <nav>       - Navigazione principale                                   │
│ ├── <main>      - Contenuto principale (uno per pagina)                    │
│ ├── <article>   - Contenuto indipendente (blog post, prodotto)             │
│ ├── <section>   - Sezione tematica                                         │
│ ├── <aside>     - Contenuto correlato (sidebar)                            │
│ └── <footer>    - Footer pagina/sezione                                    │
│                                                                             │
│ HEADINGS (H1-H6)                                                            │
│ ├── H1 unico per pagina (titolo principale)                                │
│ ├── Gerarchia logica: H1 > H2 > H3 (no skip levels)                        │
│ ├── Keywords naturali negli headings                                       │
│ └── Descrittivi, non generici                                              │
│                                                                             │
│ ALTRI ELEMENTI SEMANTICI                                                    │
│ ├── <time datetime="">  - Date/orari                                       │
│ ├── <address>           - Info contatto                                    │
│ ├── <figure>/<figcaption> - Immagini con didascalia                        │
│ └── <mark>, <strong>, <em> - Enfasi semantica                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 10.2 Page Structure Template

```tsx
// app/blog/[slug]/page.tsx - Struttura semantica corretta

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug);
  
  return (
    <>
      {/* Header di pagina */}
      <header className="page-header">
        <nav aria-label="Breadcrumb">
          <Breadcrumbs items={[
            { name: 'Home', url: '/' },
            { name: 'Blog', url: '/blog' },
            { name: post.title, url: `/blog/${post.slug}` },
          ]} />
        </nav>
      </header>
      
      {/* Contenuto principale */}
      <main id="main-content">
        <article itemScope itemType="https://schema.org/BlogPosting">
          {/* Header dell'articolo */}
          <header>
            <h1 itemProp="headline">{post.title}</h1>
            
            <p className="meta">
              Pubblicato il{' '}
              <time dateTime={post.publishedAt} itemProp="datePublished">
                {formatDate(post.publishedAt)}
              </time>
              {' '}da{' '}
              <address itemProp="author" itemScope itemType="https://schema.org/Person">
                <a rel="author" href={`/authors/${post.author.slug}`} itemProp="url">
                  <span itemProp="name">{post.author.name}</span>
                </a>
              </address>
            </p>
          </header>
          
          {/* Immagine di copertina */}
          <figure>
            <Image
              src={post.coverImage}
              alt={post.coverImageAlt}
              width={1200}
              height={630}
              priority
              itemProp="image"
            />
            {post.coverImageCaption && (
              <figcaption>{post.coverImageCaption}</figcaption>
            )}
          </figure>
          
          {/* Contenuto dell'articolo */}
          <section itemProp="articleBody">
            <div dangerouslySetInnerHTML={{ __html: post.content }} />
          </section>
          
          {/* Footer dell'articolo */}
          <footer>
            <section aria-labelledby="tags-heading">
              <h2 id="tags-heading" className="sr-only">Tags</h2>
              <ul className="tags" itemProp="keywords">
                {post.tags.map((tag) => (
                  <li key={tag}>
                    <a href={`/blog/tag/${tag}`}>{tag}</a>
                  </li>
                ))}
              </ul>
            </section>
          </footer>
        </article>
        
        {/* Articoli correlati */}
        <aside aria-labelledby="related-heading">
          <h2 id="related-heading">Articoli correlati</h2>
          <RelatedPosts postId={post.id} />
        </aside>
      </main>
    </>
  );
}
```

## 10.3 Heading Structure

```tsx
// ✅ CORRETTA gerarchia headings
function ProductPage({ product }) {
  return (
    <main>
      <h1>{product.name}</h1>                    {/* H1 unico */}
      
      <section>
        <h2>Descrizione</h2>                     {/* H2 */}
        <p>{product.description}</p>
        
        <h3>Caratteristiche</h3>                 {/* H3 sotto H2 */}
        <ul>{/* ... */}</ul>
        
        <h3>Specifiche tecniche</h3>
        <ul>{/* ... */}</ul>
      </section>
      
      <section>
        <h2>Recensioni</h2>
        {product.reviews.map((review) => (
          <article key={review.id}>
            <h3>{review.title}</h3>              {/* H3 sotto H2 */}
            <p>{review.content}</p>
          </article>
        ))}
      </section>
    </main>
  );
}

// ❌ ERRATA gerarchia headings
function BadProductPage({ product }) {
  return (
    <main>
      <h2>{product.name}</h2>                    {/* NO H1! */}
      <h4>Descrizione</h4>                       {/* Skip da H2 a H4 */}
      <h1>Caratteristiche</h1>                   {/* H1 duplicato */}
      <h6>Specifiche</h6>                        {/* Skip levels */}
    </main>
  );
}
```

## 10.4 Accessibility per SEO

```tsx
// L'accessibilità migliora anche il SEO

// 1. Skip links per navigazione
// components/SkipLinks.tsx
export function SkipLinks() {
  return (
    <a
      href="#main-content"
      className="sr-only focus:not-sr-only focus:absolute focus:top-0 focus:left-0 focus:z-50 focus:p-4 focus:bg-white"
    >
      Salta al contenuto principale
    </a>
  );
}

// 2. ARIA labels per navigazione
function Navigation() {
  return (
    <nav aria-label="Navigazione principale">
      <ul role="list">
        <li><a href="/">Home</a></li>
        <li><a href="/products">Prodotti</a></li>
        <li><a href="/about">Chi siamo</a></li>
      </ul>
    </nav>
  );
}

// 3. Form labels
function SearchForm() {
  return (
    <form role="search" aria-label="Cerca nel sito">
      <label htmlFor="search-input" className="sr-only">
        Cerca
      </label>
      <input
        id="search-input"
        type="search"
        name="q"
        placeholder="Cerca prodotti..."
        aria-describedby="search-hint"
      />
      <span id="search-hint" className="sr-only">
        Inserisci un termine di ricerca
      </span>
      <button type="submit" aria-label="Avvia ricerca">
        <SearchIcon aria-hidden="true" />
      </button>
    </form>
  );
}

// 4. Immagini accessibili
function AccessibleImage({ src, alt, decorative = false }) {
  if (decorative) {
    return <img src={src} alt="" role="presentation" />;
  }
  
  return <img src={src} alt={alt} />;
}

// 5. Tabelle accessibili
function DataTable({ data, caption }) {
  return (
    <table>
      <caption>{caption}</caption>
      <thead>
        <tr>
          <th scope="col">Nome</th>
          <th scope="col">Prezzo</th>
          <th scope="col">Stock</th>
        </tr>
      </thead>
      <tbody>
        {data.map((item) => (
          <tr key={item.id}>
            <th scope="row">{item.name}</th>
            <td>{item.price}</td>
            <td>{item.stock}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

## 10.5 Rich Content Markup

```tsx
// Markup per contenuti rich che aiutano SEO

// 1. Citazioni
function Quote({ text, author, source }) {
  return (
    <figure>
      <blockquote cite={source}>
        <p>{text}</p>
      </blockquote>
      <figcaption>
        — <cite>{author}</cite>
      </figcaption>
    </figure>
  );
}

// 2. Liste di definizioni
function Glossary({ terms }) {
  return (
    <dl>
      {terms.map((term) => (
        <div key={term.name}>
          <dt><strong>{term.name}</strong></dt>
          <dd>{term.definition}</dd>
        </div>
      ))}
    </dl>
  );
}

// 3. Contenuto con tempo
function Event({ event }) {
  return (
    <article>
      <h3>{event.title}</h3>
      <p>
        <time dateTime={event.startDate}>
          {formatDate(event.startDate)}
        </time>
        {' - '}
        <time dateTime={event.endDate}>
          {formatDate(event.endDate)}
        </time>
      </p>
      <address>
        {event.location.address}
      </address>
    </article>
  );
}

// 4. Codice con syntax highlighting
function CodeBlock({ code, language }) {
  return (
    <figure>
      <pre>
        <code className={`language-${language}`}>
          {code}
        </code>
      </pre>
      <figcaption>Esempio di codice {language}</figcaption>
    </figure>
  );
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 11: INTERNAL LINKING & NAVIGATION
# ═══════════════════════════════════════════════════════════════════════════════

INTERNAL_LINKING = """

## 11.1 Strategia Internal Linking

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ INTERNAL LINKING PER SEO                                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ BENEFICI                                                                    │
│ ├── Distribuisce "link equity" tra le pagine                               │
│ ├── Aiuta crawlers a scoprire contenuti                                    │
│ ├── Stabilisce gerarchia del sito                                          │
│ ├── Migliora user experience                                               │
│ └── Aumenta tempo on-site                                                  │
│                                                                             │
│ BEST PRACTICES                                                              │
│ ├── Anchor text descrittivi (no "clicca qui")                              │
│ ├── Link a pagine rilevanti                                                │
│ ├── No troppi link per pagina (max ~100)                                   │
│ ├── Prioritizza link nel content                                           │
│ └── Mantieni struttura gerarchica                                          │
│                                                                             │
│ STRUTTURA A SILO                                                            │
│                                                                             │
│              [Homepage]                                                     │
│              /    |    \                                                    │
│       [Cat A] [Cat B] [Cat C]                                              │
│        / | \    / \    / | \                                               │
│      [P1][P2][P3][P4][P5][P6][P7]                                          │
│                                                                             │
│ Le pagine dello stesso silo si linkano tra loro.                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 11.2 Anchor Text Best Practices

```tsx
// ✅ BUONI anchor text
<Link href="/leather-bags">borse in pelle artigianali</Link>
<Link href="/blog/leather-care">come curare la pelle</Link>
<Link href="/products/messenger-bag">borsa messenger marrone</Link>

// ❌ CATTIVI anchor text
<Link href="/leather-bags">clicca qui</Link>        // Generico
<Link href="/leather-bags">scopri di più</Link>     // Non descrittivo
<Link href="/leather-bags">link</Link>              // Inutile
<Link href="/leather-bags">                         // URL come anchor
  https://example.com/leather-bags
</Link>

// Component per link con anchor text appropriato
interface InternalLinkProps {
  href: string;
  children: React.ReactNode;
  title?: string;
}

export function InternalLink({ href, children, title }: InternalLinkProps) {
  return (
    <Link href={href} title={title}>
      {children}
    </Link>
  );
}
```

## 11.3 Related Content Component

```tsx
// components/RelatedContent.tsx
import Link from 'next/link';

interface RelatedItem {
  slug: string;
  title: string;
  excerpt: string;
  image?: string;
}

interface RelatedContentProps {
  items: RelatedItem[];
  type: 'products' | 'posts' | 'categories';
  title?: string;
}

export function RelatedContent({ items, type, title = 'Potrebbe interessarti' }: RelatedContentProps) {
  const basePath = {
    products: '/products',
    posts: '/blog',
    categories: '/categories',
  }[type];
  
  return (
    <aside aria-labelledby="related-content-heading">
      <h2 id="related-content-heading">{title}</h2>
      <ul className="grid grid-cols-1 md:grid-cols-3 gap-4">
        {items.map((item) => (
          <li key={item.slug}>
            <Link 
              href={`${basePath}/${item.slug}`}
              className="block p-4 border rounded hover:shadow"
            >
              {item.image && (
                <Image src={item.image} alt="" width={200} height={150} />
              )}
              <h3>{item.title}</h3>
              <p>{item.excerpt}</p>
            </Link>
          </li>
        ))}
      </ul>
    </aside>
  );
}

// Uso
<RelatedContent
  items={relatedProducts}
  type="products"
  title="Prodotti correlati"
/>
```

## 11.4 Breadcrumbs Navigation

```tsx
// components/Breadcrumbs.tsx
import Link from 'next/link';
import { getBreadcrumbSchema } from '@/lib/seo/breadcrumb-schema';
import { JsonLd } from './JsonLd';

interface BreadcrumbItem {
  name: string;
  url: string;
}

interface BreadcrumbsProps {
  items: BreadcrumbItem[];
}

export function Breadcrumbs({ items }: BreadcrumbsProps) {
  const schema = getBreadcrumbSchema(items);
  
  return (
    <>
      <JsonLd data={schema} />
      <nav aria-label="Breadcrumb" className="text-sm">
        <ol className="flex items-center space-x-2" itemScope itemType="https://schema.org/BreadcrumbList">
          {items.map((item, index) => {
            const isLast = index === items.length - 1;
            
            return (
              <li 
                key={item.url}
                className="flex items-center"
                itemProp="itemListElement"
                itemScope
                itemType="https://schema.org/ListItem"
              >
                {index > 0 && (
                  <span className="mx-2 text-gray-400" aria-hidden="true">/</span>
                )}
                
                {isLast ? (
                  <span 
                    aria-current="page" 
                    itemProp="name"
                    className="text-gray-600"
                  >
                    {item.name}
                  </span>
                ) : (
                  <Link 
                    href={item.url}
                    itemProp="item"
                    className="text-blue-600 hover:underline"
                  >
                    <span itemProp="name">{item.name}</span>
                  </Link>
                )}
                
                <meta itemProp="position" content={String(index + 1)} />
              </li>
            );
          })}
        </ol>
      </nav>
    </>
  );
}
```

## 11.5 Auto-Linking nel Content

```tsx
// lib/utils/auto-link.ts
// Automaticamente linka parole chiave nel contenuto

interface KeywordLink {
  keyword: string;
  url: string;
  title?: string;
}

export function autoLinkContent(content: string, links: KeywordLink[]): string {
  let result = content;
  
  // Ordina per lunghezza keyword (più lunghe prima)
  const sortedLinks = [...links].sort((a, b) => b.keyword.length - a.keyword.length);
  
  for (const link of sortedLinks) {
    // Match solo parole intere, case insensitive
    const regex = new RegExp(`\\b(${escapeRegex(link.keyword)})\\b`, 'gi');
    
    // Linka solo la prima occorrenza
    let linked = false;
    result = result.replace(regex, (match) => {
      if (linked) return match;
      linked = true;
      return `<a href="${link.url}" title="${link.title || ''}">${match}</a>`;
    });
  }
  
  return result;
}

function escapeRegex(string: string): string {
  return string.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
}

// Uso nel blog post
const keywordLinks: KeywordLink[] = [
  { keyword: 'borse in pelle', url: '/products/category/leather-bags', title: 'Scopri le nostre borse in pelle' },
  { keyword: 'artigianato italiano', url: '/about/craftsmanship', title: 'La nostra tradizione artigianale' },
  { keyword: 'cura della pelle', url: '/blog/leather-care-guide', title: 'Guida alla cura della pelle' },
];

const linkedContent = autoLinkContent(post.content, keywordLinks);
```

## 11.6 Footer Links SEO

```tsx
// components/Footer.tsx
export function Footer() {
  return (
    <footer className="bg-gray-900 text-white py-12">
      <div className="container mx-auto grid grid-cols-1 md:grid-cols-4 gap-8">
        {/* Chi siamo */}
        <section aria-labelledby="footer-about">
          <h2 id="footer-about" className="font-bold mb-4">Chi Siamo</h2>
          <p className="text-sm text-gray-400">
            Artigiani della pelle dal 1990...
          </p>
        </section>
        
        {/* Categorie - Link SEO importanti */}
        <nav aria-labelledby="footer-categories">
          <h2 id="footer-categories" className="font-bold mb-4">Categorie</h2>
          <ul className="space-y-2">
            <li><Link href="/products/category/bags">Borse</Link></li>
            <li><Link href="/products/category/wallets">Portafogli</Link></li>
            <li><Link href="/products/category/belts">Cinture</Link></li>
            <li><Link href="/products/category/accessories">Accessori</Link></li>
          </ul>
        </nav>
        
        {/* Informazioni */}
        <nav aria-labelledby="footer-info">
          <h2 id="footer-info" className="font-bold mb-4">Informazioni</h2>
          <ul className="space-y-2">
            <li><Link href="/shipping">Spedizioni</Link></li>
            <li><Link href="/returns">Resi</Link></li>
            <li><Link href="/faq">FAQ</Link></li>
            <li><Link href="/contact">Contatti</Link></li>
          </ul>
        </nav>
        
        {/* Risorse - Content SEO */}
        <nav aria-labelledby="footer-resources">
          <h2 id="footer-resources" className="font-bold mb-4">Risorse</h2>
          <ul className="space-y-2">
            <li><Link href="/blog">Blog</Link></li>
            <li><Link href="/guides/leather-care">Guida Cura Pelle</Link></li>
            <li><Link href="/guides/sizing">Guida Taglie</Link></li>
            <li><Link href="/about/craftsmanship">Il Nostro Artigianato</Link></li>
          </ul>
        </nav>
      </div>
      
      {/* Legal - nofollow per link esterni */}
      <div className="border-t border-gray-800 mt-8 pt-8">
        <nav aria-label="Link legali">
          <ul className="flex space-x-4 justify-center text-sm text-gray-400">
            <li><Link href="/privacy">Privacy Policy</Link></li>
            <li><Link href="/terms">Termini e Condizioni</Link></li>
            <li><Link href="/cookies">Cookie Policy</Link></li>
          </ul>
        </nav>
      </div>
    </footer>
  );
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 12: E-COMMERCE SEO
# ═══════════════════════════════════════════════════════════════════════════════

ECOMMERCE_SEO = """

## 12.1 Product Page SEO

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ E-COMMERCE SEO ESSENTIALS                                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ PAGINA PRODOTTO                                                             │
│ ├── Title: [Nome Prodotto] | [Brand] | [Categoria]                         │
│ ├── H1: Nome prodotto con variante                                         │
│ ├── Description unica (no duplicati tra varianti)                          │
│ ├── Product Schema con prezzo, disponibilità, reviews                      │
│ ├── Immagini multiple con alt text descrittivo                             │
│ ├── Breadcrumbs                                                            │
│ └── Recensioni con Review Schema                                           │
│                                                                             │
│ PAGINA CATEGORIA                                                            │
│ ├── Title: [Categoria] | [Subcategoria] - [Brand]                          │
│ ├── H1: Nome categoria                                                     │
│ ├── Testo introduttivo (200-500 parole)                                    │
│ ├── Filtri crawlabili (no JavaScript-only)                                 │
│ ├── CollectionPage Schema                                                  │
│ └── Pagination SEO-friendly                                                │
│                                                                             │
│ PROBLEMI COMUNI                                                             │
│ ├── Duplicate content tra varianti                                         │
│ ├── Faceted navigation → infinite URLs                                     │
│ ├── Out of stock pages                                                     │
│ └── Thin content su pagine categoria                                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 12.2 Product Page Implementation

```tsx
// app/products/[slug]/page.tsx
import type { Metadata } from 'next';
import { getProductSchema } from '@/lib/seo/product-schema';
import { getBreadcrumbSchema } from '@/lib/seo/breadcrumb-schema';
import { JsonLd } from '@/components/JsonLd';

export async function generateMetadata({ params }): Promise<Metadata> {
  const product = await getProduct(params.slug);
  
  return {
    title: `${product.name} | ${product.brand} | ${product.category}`,
    description: product.shortDescription.substring(0, 160),
    
    openGraph: {
      type: 'product',
      title: product.name,
      description: product.shortDescription,
      images: product.images.map((img) => ({
        url: img.url,
        width: 1200,
        height: 630,
        alt: `${product.name} - ${img.alt}`,
      })),
    },
    
    alternates: {
      canonical: `/products/${params.slug}`,
    },
    
    robots: {
      index: product.isPublished && product.inStock,
      follow: true,
    },
  };
}

export default async function ProductPage({ params }) {
  const product = await getProduct(params.slug);
  
  const productSchema = getProductSchema({
    name: product.name,
    description: product.description,
    image: product.images.map((i) => i.url),
    sku: product.sku,
    brand: product.brand,
    price: product.price,
    currency: 'EUR',
    availability: product.inStock ? 'InStock' : 'OutOfStock',
    url: `https://www.example.com/products/${params.slug}`,
    rating: product.rating,
  });
  
  const breadcrumbSchema = getBreadcrumbSchema([
    { name: 'Home', url: 'https://www.example.com' },
    { name: product.categoryName, url: `https://www.example.com/category/${product.categorySlug}` },
    { name: product.name, url: `https://www.example.com/products/${params.slug}` },
  ]);
  
  return (
    <>
      <JsonLd data={productSchema} />
      <JsonLd data={breadcrumbSchema} />
      
      <main>
        <Breadcrumbs items={[/* ... */]} />
        
        <article itemScope itemType="https://schema.org/Product">
          <header>
            <h1 itemProp="name">{product.name}</h1>
            <p itemProp="brand" itemScope itemType="https://schema.org/Brand">
              <span itemProp="name">{product.brand}</span>
            </p>
          </header>
          
          {/* Gallery con alt text SEO */}
          <ProductGallery images={product.images} productName={product.name} />
          
          {/* Prezzo e disponibilità */}
          <div itemProp="offers" itemScope itemType="https://schema.org/Offer">
            <span itemProp="priceCurrency" content="EUR">€</span>
            <span itemProp="price" content={product.price}>{product.price}</span>
            <link itemProp="availability" href={`https://schema.org/${product.inStock ? 'InStock' : 'OutOfStock'}`} />
          </div>
          
          {/* Description ricca */}
          <section>
            <h2>Descrizione</h2>
            <div itemProp="description" dangerouslySetInnerHTML={{ __html: product.description }} />
          </section>
          
          {/* Specifiche */}
          <section>
            <h2>Specifiche</h2>
            <ProductSpecs specs={product.specs} />
          </section>
          
          {/* Reviews con schema */}
          <section>
            <h2>Recensioni ({product.reviewCount})</h2>
            <ProductReviews reviews={product.reviews} productId={product.id} />
          </section>
        </article>
        
        {/* Related products */}
        <RelatedProducts categoryId={product.categoryId} currentProductId={product.id} />
      </main>
    </>
  );
}
```

## 12.3 Category Page SEO

```tsx
// app/category/[slug]/page.tsx
import type { Metadata } from 'next';

export async function generateMetadata({ params, searchParams }): Promise<Metadata> {
  const category = await getCategory(params.slug);
  const page = parseInt(searchParams.page || '1');
  
  const title = page > 1 
    ? `${category.name} - Pagina ${page}` 
    : category.name;
  
  return {
    title: `${title} | Shop Online`,
    description: category.seoDescription || `Scopri la nostra collezione di ${category.name}. ${category.productCount} prodotti disponibili.`,
    
    alternates: {
      // Canonical senza parametri di filtro
      canonical: page === 1 
        ? `/category/${params.slug}` 
        : `/category/${params.slug}?page=${page}`,
    },
    
    // No index per pagine filtrate
    robots: {
      index: !searchParams.filter && !searchParams.sort,
      follow: true,
    },
  };
}

export default async function CategoryPage({ params, searchParams }) {
  const category = await getCategory(params.slug);
  const products = await getCategoryProducts(params.slug, searchParams);
  
  // CollectionPage schema
  const collectionSchema = {
    '@context': 'https://schema.org',
    '@type': 'CollectionPage',
    name: category.name,
    description: category.description,
    url: `https://www.example.com/category/${params.slug}`,
    numberOfItems: category.productCount,
    mainEntity: {
      '@type': 'ItemList',
      itemListElement: products.items.map((product, index) => ({
        '@type': 'ListItem',
        position: index + 1,
        item: {
          '@type': 'Product',
          name: product.name,
          url: `https://www.example.com/products/${product.slug}`,
          image: product.image,
          offers: {
            '@type': 'Offer',
            price: product.price,
            priceCurrency: 'EUR',
          },
        },
      })),
    },
  };
  
  return (
    <>
      <JsonLd data={collectionSchema} />
      
      <main>
        <Breadcrumbs items={[
          { name: 'Home', url: '/' },
          { name: category.name, url: `/category/${params.slug}` },
        ]} />
        
        <header>
          <h1>{category.name}</h1>
          {/* Testo SEO introduttivo */}
          <p>{category.description}</p>
        </header>
        
        {/* Filtri - con URL crawlabili */}
        <aside aria-label="Filtri">
          <FilterSidebar 
            category={params.slug} 
            currentFilters={searchParams}
            useCrawlableUrls={true} // Genera link invece di JS
          />
        </aside>
        
        {/* Product grid */}
        <section aria-label="Prodotti">
          <ProductGrid products={products.items} />
        </section>
        
        {/* Pagination SEO */}
        <Pagination
          currentPage={products.currentPage}
          totalPages={products.totalPages}
          basePath={`/category/${params.slug}`}
        />
        
        {/* Testo SEO aggiuntivo (sotto i prodotti) */}
        {category.seoContent && (
          <section className="mt-8">
            <div dangerouslySetInnerHTML={{ __html: category.seoContent }} />
          </section>
        )}
      </main>
    </>
  );
}
```

## 12.4 Handling Product Variants

```tsx
// Gestione varianti prodotto per evitare duplicate content

// Approccio 1: URL canonico alla variante principale
// /products/tshirt-blue → canonical: /products/tshirt
// /products/tshirt-red → canonical: /products/tshirt

export async function generateMetadata({ params }): Promise<Metadata> {
  const product = await getProduct(params.slug);
  
  return {
    title: product.name,
    alternates: {
      canonical: `/products/${product.parentSlug || product.slug}`,
    },
  };
}

// Approccio 2: Tutte le varianti su una pagina con selector
// /products/tshirt → page con dropdown colore
function ProductPage({ product }) {
  const [selectedVariant, setSelectedVariant] = useState(product.variants[0]);
  
  return (
    <div>
      <h1>{product.name}</h1>
      
      {/* Variant selector - non cambia URL */}
      <select 
        value={selectedVariant.id}
        onChange={(e) => {
          const variant = product.variants.find(v => v.id === e.target.value);
          setSelectedVariant(variant);
        }}
        aria-label="Seleziona variante"
      >
        {product.variants.map((variant) => (
          <option key={variant.id} value={variant.id}>
            {variant.name} - €{variant.price}
          </option>
        ))}
      </select>
      
      {/* Immagine variante */}
      <Image src={selectedVariant.image} alt={`${product.name} - ${selectedVariant.name}`} />
    </div>
  );
}

// Approccio 3: Parametri per varianti (con noindex)
// /products/tshirt?color=blue → noindex
// /products/tshirt → index (canonical)

export async function generateMetadata({ params, searchParams }): Promise<Metadata> {
  const hasVariantParams = searchParams.color || searchParams.size;
  
  return {
    robots: {
      index: !hasVariantParams, // noindex se ha params variante
      follow: true,
    },
    alternates: {
      canonical: `/products/${params.slug}`, // sempre senza params
    },
  };
}
```

## 12.5 Out of Stock Products

```tsx
// Gestione prodotti esauriti per SEO

export async function generateMetadata({ params }): Promise<Metadata> {
  const product = await getProduct(params.slug);
  
  if (!product) {
    return { title: 'Prodotto non trovato' };
  }
  
  // Prodotto esaurito temporaneamente: mantieni indicizzato
  if (!product.inStock && product.willRestock) {
    return {
      title: `${product.name} (Temporaneamente esaurito)`,
      robots: { index: true, follow: true },
    };
  }
  
  // Prodotto discontinued: noindex o redirect
  if (product.discontinued) {
    return {
      title: product.name,
      robots: { index: false, follow: true }, // noindex
    };
  }
  
  return {
    title: product.name,
    robots: { index: true, follow: true },
  };
}

// Redirect per prodotti discontinued (in next.config.js o middleware)
// Se il prodotto è stato rimpiazzato, redirect 301 al nuovo
async function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname;
  
  if (pathname.startsWith('/products/')) {
    const slug = pathname.replace('/products/', '');
    const redirect = await getProductRedirect(slug);
    
    if (redirect) {
      return NextResponse.redirect(
        new URL(redirect.newUrl, request.url),
        redirect.permanent ? 301 : 302
      );
    }
  }
  
  return NextResponse.next();
}
```

## 12.6 Review Schema Implementation

```tsx
// components/ProductReviews.tsx
import { JsonLd } from './JsonLd';

interface Review {
  id: string;
  author: string;
  rating: number;
  title: string;
  content: string;
  date: string;
}

interface ProductReviewsProps {
  reviews: Review[];
  productName: string;
  productUrl: string;
}

export function ProductReviews({ reviews, productName, productUrl }: ProductReviewsProps) {
  // Calcola aggregated rating
  const averageRating = reviews.reduce((sum, r) => sum + r.rating, 0) / reviews.length;
  
  // Review schema per ogni recensione
  const reviewSchemas = reviews.map((review) => ({
    '@context': 'https://schema.org',
    '@type': 'Review',
    itemReviewed: {
      '@type': 'Product',
      name: productName,
      url: productUrl,
    },
    reviewRating: {
      '@type': 'Rating',
      ratingValue: review.rating,
      bestRating: 5,
      worstRating: 1,
    },
    author: {
      '@type': 'Person',
      name: review.author,
    },
    datePublished: review.date,
    reviewBody: review.content,
    name: review.title,
  }));
  
  // Aggregate rating schema
  const aggregateSchema = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: productName,
    aggregateRating: {
      '@type': 'AggregateRating',
      ratingValue: averageRating.toFixed(1),
      reviewCount: reviews.length,
      bestRating: 5,
      worstRating: 1,
    },
  };
  
  return (
    <>
      <JsonLd data={aggregateSchema} />
      
      {/* Rating summary */}
      <div className="flex items-center mb-4">
        <StarRating rating={averageRating} />
        <span className="ml-2">
          {averageRating.toFixed(1)} su 5 ({reviews.length} recensioni)
        </span>
      </div>
      
      {/* Individual reviews */}
      <ul className="space-y-4">
        {reviews.map((review) => (
          <li key={review.id}>
            <JsonLd data={reviewSchemas.find(s => s.name === review.title)} />
            <article>
              <header className="flex items-center">
                <StarRating rating={review.rating} />
                <h3 className="ml-2 font-bold">{review.title}</h3>
              </header>
              <p className="text-sm text-gray-500">
                di {review.author} - <time dateTime={review.date}>{formatDate(review.date)}</time>
              </p>
              <p className="mt-2">{review.content}</p>
            </article>
          </li>
        ))}
      </ul>
    </>
  );
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 13: BLOG & CONTENT SEO
# ═══════════════════════════════════════════════════════════════════════════════

BLOG_CONTENT_SEO = """

## 13.1 Blog Post SEO Checklist

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ BLOG POST SEO CHECKLIST                                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ PRE-PUBBLICAZIONE                                                           │
│ ├── ☐ Keyword research completata                                          │
│ ├── ☐ Title con keyword principale (50-60 chars)                           │
│ ├── ☐ Meta description unica e compelling (150-160 chars)                  │
│ ├── ☐ URL slug ottimizzato (/blog/keyword-slug)                            │
│ ├── ☐ H1 unico con keyword                                                 │
│ ├── ☐ Struttura H2/H3 logica                                               │
│ ├── ☐ Internal links a contenuti correlati                                 │
│ ├── ☐ External links a fonti autorevoli                                    │
│ ├── ☐ Immagini con alt text descrittivo                                    │
│ ├── ☐ Featured image per social (1200x630)                                 │
│ └── ☐ Article schema implementato                                          │
│                                                                             │
│ CONTENUTO                                                                   │
│ ├── ☐ Minimo 1500 parole per guide complete                                │
│ ├── ☐ Keyword density 1-2% (naturale)                                      │
│ ├── ☐ Keyword nel primo paragrafo                                          │
│ ├── ☐ LSI keywords (termini correlati)                                     │
│ ├── ☐ Paragrafi brevi (max 3-4 frasi)                                      │
│ ├── ☐ Liste puntate dove appropriato                                       │
│ ├── ☐ Table of Contents per post lunghi                                    │
│ └── ☐ FAQ section con FAQPage schema                                       │
│                                                                             │
│ POST-PUBBLICAZIONE                                                          │
│ ├── ☐ Submetti a Google Search Console                                     │
│ ├── ☐ Condividi su social                                                  │
│ ├── ☐ Internal linking da post esistenti                                   │
│ └── ☐ Monitora rankings e CTR                                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 13.2 Complete Blog Post Template

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from 'next';
import { getArticleSchema } from '@/lib/seo/article-schema';
import { getFAQSchema } from '@/lib/seo/faq-schema';

export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await getPost(params.slug);
  
  return {
    title: post.seoTitle || post.title,
    description: post.seoDescription || post.excerpt,
    
    authors: [{ name: post.author.name, url: post.author.url }],
    
    openGraph: {
      type: 'article',
      title: post.seoTitle || post.title,
      description: post.seoDescription || post.excerpt,
      url: `https://www.example.com/blog/${params.slug}`,
      publishedTime: post.publishedAt,
      modifiedTime: post.updatedAt,
      authors: [post.author.name],
      tags: post.tags,
      section: post.category,
      images: [
        {
          url: post.featuredImage,
          width: 1200,
          height: 630,
          alt: post.featuredImageAlt || post.title,
        },
      ],
    },
    
    twitter: {
      card: 'summary_large_image',
      title: post.seoTitle || post.title,
      description: post.seoDescription || post.excerpt,
      images: [post.featuredImage],
      creator: post.author.twitter,
    },
    
    alternates: {
      canonical: `/blog/${params.slug}`,
    },
  };
}

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug);
  const relatedPosts = await getRelatedPosts(post.id, post.tags);
  
  // Structured Data
  const articleSchema = getArticleSchema({
    title: post.title,
    description: post.excerpt,
    url: `https://www.example.com/blog/${params.slug}`,
    image: post.featuredImage,
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
    author: {
      name: post.author.name,
      url: post.author.url,
    },
  });
  
  const faqSchema = post.faqs?.length > 0 ? getFAQSchema(post.faqs) : null;
  
  // Reading time
  const readingTime = calculateReadingTime(post.content);
  
  return (
    <>
      <JsonLd data={articleSchema} />
      {faqSchema && <JsonLd data={faqSchema} />}
      
      <main>
        <Breadcrumbs items={[
          { name: 'Home', url: '/' },
          { name: 'Blog', url: '/blog' },
          { name: post.category, url: `/blog/category/${post.categorySlug}` },
          { name: post.title, url: `/blog/${params.slug}` },
        ]} />
        
        <article itemScope itemType="https://schema.org/BlogPosting">
          <header className="mb-8">
            {/* H1 con keyword */}
            <h1 itemProp="headline" className="text-4xl font-bold">
              {post.title}
            </h1>
            
            {/* Meta info */}
            <div className="flex items-center mt-4 text-gray-600">
              <address className="not-italic" itemProp="author" itemScope itemType="https://schema.org/Person">
                <a rel="author" href={`/authors/${post.author.slug}`} itemProp="url">
                  <Image 
                    src={post.author.avatar} 
                    alt={post.author.name}
                    width={40}
                    height={40}
                    className="rounded-full mr-2"
                  />
                  <span itemProp="name">{post.author.name}</span>
                </a>
              </address>
              
              <span className="mx-2">•</span>
              
              <time dateTime={post.publishedAt} itemProp="datePublished">
                {formatDate(post.publishedAt)}
              </time>
              
              <span className="mx-2">•</span>
              
              <span>{readingTime} min lettura</span>
            </div>
            
            {/* Tags */}
            <ul className="flex gap-2 mt-4" itemProp="keywords">
              {post.tags.map((tag) => (
                <li key={tag}>
                  <a href={`/blog/tag/${tag}`} className="text-sm bg-gray-100 px-2 py-1 rounded">
                    {tag}
                  </a>
                </li>
              ))}
            </ul>
          </header>
          
          {/* Featured Image */}
          <figure className="mb-8">
            <Image
              src={post.featuredImage}
              alt={post.featuredImageAlt || post.title}
              width={1200}
              height={630}
              priority
              itemProp="image"
            />
            {post.featuredImageCaption && (
              <figcaption className="text-sm text-gray-500 mt-2">
                {post.featuredImageCaption}
              </figcaption>
            )}
          </figure>
          
          {/* Table of Contents */}
          {post.toc && (
            <nav aria-labelledby="toc-heading" className="mb-8 p-4 bg-gray-50 rounded">
              <h2 id="toc-heading" className="font-bold mb-2">Indice dei contenuti</h2>
              <TableOfContents items={post.toc} />
            </nav>
          )}
          
          {/* Article Body */}
          <div 
            itemProp="articleBody" 
            className="prose prose-lg max-w-none"
            dangerouslySetInnerHTML={{ __html: post.content }}
          />
          
          {/* FAQ Section */}
          {post.faqs?.length > 0 && (
            <section className="mt-12">
              <h2>Domande Frequenti</h2>
              <FAQ faqs={post.faqs} />
            </section>
          )}
          
          {/* Author Box */}
          <aside className="mt-12 p-6 bg-gray-50 rounded">
            <h2 className="font-bold mb-4">Sull'autore</h2>
            <AuthorBox author={post.author} />
          </aside>
        </article>
        
        {/* Related Posts */}
        <aside className="mt-12">
          <h2 className="text-2xl font-bold mb-6">Articoli correlati</h2>
          <RelatedPosts posts={relatedPosts} />
        </aside>
        
        {/* Comments with nofollow */}
        <section className="mt-12">
          <h2 className="text-2xl font-bold mb-6">Commenti</h2>
          <Comments postId={post.id} />
        </section>
      </main>
    </>
  );
}
```

## 13.3 Table of Contents Component

```tsx
// components/TableOfContents.tsx
interface TOCItem {
  id: string;
  text: string;
  level: number;
}

export function TableOfContents({ items }: { items: TOCItem[] }) {
  return (
    <ol className="space-y-2">
      {items.map((item, index) => (
        <li 
          key={item.id}
          style={{ marginLeft: `${(item.level - 2) * 16}px` }}
        >
          <a 
            href={`#${item.id}`}
            className="text-blue-600 hover:underline"
          >
            {item.text}
          </a>
        </li>
      ))}
    </ol>
  );
}

// Utility per generare TOC dal content
export function generateTOC(htmlContent: string): TOCItem[] {
  const headingRegex = /<h([2-4])[^>]*id="([^"]*)"[^>]*>([^<]*)<\/h[2-4]>/g;
  const items: TOCItem[] = [];
  
  let match;
  while ((match = headingRegex.exec(htmlContent)) !== null) {
    items.push({
      level: parseInt(match[1]),
      id: match[2],
      text: match[3],
    });
  }
  
  return items;
}
```

## 13.4 Blog Category & Tag Pages

```tsx
// app/blog/category/[slug]/page.tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const category = await getCategory(params.slug);
  
  return {
    title: `${category.name} - Articoli e Guide`,
    description: category.description || `Tutti gli articoli nella categoria ${category.name}`,
    
    alternates: {
      canonical: `/blog/category/${params.slug}`,
    },
  };
}

export default async function CategoryPage({ params, searchParams }) {
  const category = await getCategory(params.slug);
  const posts = await getCategoryPosts(params.slug, searchParams.page);
  
  return (
    <main>
      <header>
        <h1>Articoli su {category.name}</h1>
        <p>{category.description}</p>
      </header>
      
      <PostGrid posts={posts.items} />
      
      <Pagination 
        currentPage={posts.currentPage}
        totalPages={posts.totalPages}
        basePath={`/blog/category/${params.slug}`}
      />
    </main>
  );
}

// app/blog/tag/[tag]/page.tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  return {
    title: `Articoli su #${params.tag}`,
    description: `Tutti gli articoli taggati con ${params.tag}`,
    
    // Tag pages: index but consider noindex se pochi contenuti
    robots: {
      index: true,
      follow: true,
    },
    
    alternates: {
      canonical: `/blog/tag/${params.tag}`,
    },
  };
}
```

## 13.5 Author Pages

```tsx
// app/authors/[slug]/page.tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const author = await getAuthor(params.slug);
  
  return {
    title: `${author.name} - Articoli e Bio`,
    description: author.bio.substring(0, 160),
    
    alternates: {
      canonical: `/authors/${params.slug}`,
    },
  };
}

export default async function AuthorPage({ params }) {
  const author = await getAuthor(params.slug);
  const posts = await getAuthorPosts(author.id);
  
  // Person schema
  const personSchema = {
    '@context': 'https://schema.org',
    '@type': 'Person',
    name: author.name,
    url: `https://www.example.com/authors/${params.slug}`,
    image: author.avatar,
    description: author.bio,
    sameAs: [
      author.twitter,
      author.linkedin,
      author.github,
    ].filter(Boolean),
    jobTitle: author.jobTitle,
    worksFor: {
      '@type': 'Organization',
      name: 'Company Name',
    },
  };
  
  return (
    <>
      <JsonLd data={personSchema} />
      
      <main>
        <header className="flex items-center gap-6 mb-8">
          <Image
            src={author.avatar}
            alt={author.name}
            width={120}
            height={120}
            className="rounded-full"
          />
          <div>
            <h1>{author.name}</h1>
            <p className="text-gray-600">{author.jobTitle}</p>
            <div className="flex gap-2 mt-2">
              {author.twitter && (
                <a href={author.twitter} rel="noopener noreferrer" aria-label="Twitter">
                  <TwitterIcon />
                </a>
              )}
              {author.linkedin && (
                <a href={author.linkedin} rel="noopener noreferrer" aria-label="LinkedIn">
                  <LinkedInIcon />
                </a>
              )}
            </div>
          </div>
        </header>
        
        <section>
          <h2>Bio</h2>
          <p>{author.bio}</p>
        </section>
        
        <section>
          <h2>Articoli di {author.name}</h2>
          <PostGrid posts={posts} />
        </section>
      </main>
    </>
  );
}
```

"""



# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 14: LOCAL BUSINESS SEO
# ═══════════════════════════════════════════════════════════════════════════════

LOCAL_BUSINESS_SEO = """

## 14.1 Local SEO Fundamentals

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ LOCAL SEO OVERVIEW                                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ Local SEO ottimizza la visibilità per ricerche geografiche:                │
│ "ristorante vicino a me", "idraulico Milano", etc.                         │
│                                                                             │
│ ELEMENTI CHIAVE                                                             │
│ ├── Google Business Profile (ex Google My Business)                        │
│ ├── NAP consistency (Name, Address, Phone)                                 │
│ ├── LocalBusiness Schema                                                   │
│ ├── Citazioni locali (directory, Yelp, Pagine Gialle)                      │
│ ├── Reviews locali                                                         │
│ └── Contenuto localizzato                                                  │
│                                                                             │
│ LOCAL PACK (3-Pack)                                                         │
│ ┌─────────────────────────────────────┐                                    │
│ │ 🗺️ Mappa                            │                                    │
│ │                                     │                                    │
│ │ 📍 Business 1    ⭐⭐⭐⭐⭐ (123)      │                                    │
│ │ 📍 Business 2    ⭐⭐⭐⭐ (89)        │                                    │
│ │ 📍 Business 3    ⭐⭐⭐⭐⭐ (201)      │                                    │
│ └─────────────────────────────────────┘                                    │
│                                                                             │
│ Apparire nel Local Pack richiede:                                          │
│ ├── Google Business Profile ottimizzato                                    │
│ ├── Buone reviews                                                          │
│ ├── NAP consistency                                                        │
│ └── LocalBusiness Schema sul sito                                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 14.2 LocalBusiness Schema

```typescript
// lib/seo/local-business-schema.ts
import { LocalBusiness, WithContext } from 'schema-dts';

interface LocalBusinessProps {
  name: string;
  description: string;
  image: string;
  phone: string;
  email: string;
  address: {
    street: string;
    city: string;
    region: string;
    postalCode: string;
    country: string;
  };
  geo: {
    latitude: number;
    longitude: number;
  };
  openingHours: {
    dayOfWeek: string[];
    opens: string;
    closes: string;
  }[];
  priceRange: string;
  url: string;
  socialProfiles: string[];
}

export function getLocalBusinessSchema(props: LocalBusinessProps): WithContext<LocalBusiness> {
  return {
    '@context': 'https://schema.org',
    '@type': 'LocalBusiness',
    '@id': `${props.url}#localbusiness`,
    name: props.name,
    description: props.description,
    image: props.image,
    telephone: props.phone,
    email: props.email,
    url: props.url,
    address: {
      '@type': 'PostalAddress',
      streetAddress: props.address.street,
      addressLocality: props.address.city,
      addressRegion: props.address.region,
      postalCode: props.address.postalCode,
      addressCountry: props.address.country,
    },
    geo: {
      '@type': 'GeoCoordinates',
      latitude: props.geo.latitude,
      longitude: props.geo.longitude,
    },
    openingHoursSpecification: props.openingHours.map((hours) => ({
      '@type': 'OpeningHoursSpecification',
      dayOfWeek: hours.dayOfWeek,
      opens: hours.opens,
      closes: hours.closes,
    })),
    priceRange: props.priceRange,
    sameAs: props.socialProfiles,
  };
}

// Tipi specifici di LocalBusiness
export function getRestaurantSchema(props: LocalBusinessProps & {
  servesCuisine: string[];
  menu: string;
  acceptsReservations: boolean;
}) {
  return {
    '@context': 'https://schema.org',
    '@type': 'Restaurant',
    ...getLocalBusinessSchema(props),
    servesCuisine: props.servesCuisine,
    menu: props.menu,
    acceptsReservations: props.acceptsReservations,
  };
}

export function getStoreSchema(props: LocalBusinessProps & {
  paymentAccepted: string[];
  currenciesAccepted: string;
}) {
  return {
    '@context': 'https://schema.org',
    '@type': 'Store',
    ...getLocalBusinessSchema(props),
    paymentAccepted: props.paymentAccepted.join(', '),
    currenciesAccepted: props.currenciesAccepted,
  };
}
```

## 14.3 NAP Consistency Component

```tsx
// components/BusinessInfo.tsx
// Mostra NAP in modo consistente su tutto il sito

interface BusinessInfoProps {
  showMap?: boolean;
  variant?: 'full' | 'compact' | 'footer';
}

const BUSINESS_INFO = {
  name: 'Company Name Srl',
  address: {
    street: 'Via Roma 123',
    city: 'Milano',
    region: 'MI',
    postalCode: '20100',
    country: 'Italia',
  },
  phone: '+39 02 1234567',
  phoneFormatted: '02 123 4567',
  email: 'info@example.com',
  geo: {
    latitude: 45.4642,
    longitude: 9.1900,
  },
};

export function BusinessInfo({ showMap = false, variant = 'full' }: BusinessInfoProps) {
  const localBusinessSchema = getLocalBusinessSchema({
    name: BUSINESS_INFO.name,
    // ... altri campi
  });
  
  if (variant === 'footer') {
    return (
      <address itemScope itemType="https://schema.org/LocalBusiness" className="not-italic">
        <JsonLd data={localBusinessSchema} />
        <span itemProp="name" className="font-bold">{BUSINESS_INFO.name}</span>
        <br />
        <span itemProp="address" itemScope itemType="https://schema.org/PostalAddress">
          <span itemProp="streetAddress">{BUSINESS_INFO.address.street}</span>,{' '}
          <span itemProp="postalCode">{BUSINESS_INFO.address.postalCode}</span>{' '}
          <span itemProp="addressLocality">{BUSINESS_INFO.address.city}</span>{' '}
          (<span itemProp="addressRegion">{BUSINESS_INFO.address.region}</span>)
        </span>
        <br />
        Tel: <a href={`tel:${BUSINESS_INFO.phone}`} itemProp="telephone">{BUSINESS_INFO.phoneFormatted}</a>
        <br />
        Email: <a href={`mailto:${BUSINESS_INFO.email}`} itemProp="email">{BUSINESS_INFO.email}</a>
      </address>
    );
  }
  
  return (
    <div itemScope itemType="https://schema.org/LocalBusiness">
      <JsonLd data={localBusinessSchema} />
      
      <h2 itemProp="name">{BUSINESS_INFO.name}</h2>
      
      <address itemProp="address" itemScope itemType="https://schema.org/PostalAddress" className="not-italic">
        <p>
          <span itemProp="streetAddress">{BUSINESS_INFO.address.street}</span>
        </p>
        <p>
          <span itemProp="postalCode">{BUSINESS_INFO.address.postalCode}</span>{' '}
          <span itemProp="addressLocality">{BUSINESS_INFO.address.city}</span>{' '}
          (<span itemProp="addressRegion">{BUSINESS_INFO.address.region}</span>)
        </p>
        <p>
          <span itemProp="addressCountry">{BUSINESS_INFO.address.country}</span>
        </p>
      </address>
      
      <p>
        <strong>Telefono:</strong>{' '}
        <a href={`tel:${BUSINESS_INFO.phone}`} itemProp="telephone">
          {BUSINESS_INFO.phoneFormatted}
        </a>
      </p>
      
      <p>
        <strong>Email:</strong>{' '}
        <a href={`mailto:${BUSINESS_INFO.email}`} itemProp="email">
          {BUSINESS_INFO.email}
        </a>
      </p>
      
      {showMap && (
        <div className="mt-4">
          <iframe
            src={`https://maps.google.com/maps?q=${BUSINESS_INFO.geo.latitude},${BUSINESS_INFO.geo.longitude}&output=embed`}
            width="100%"
            height="300"
            style={{ border: 0 }}
            allowFullScreen
            loading="lazy"
            referrerPolicy="no-referrer-when-downgrade"
            title={`Mappa ${BUSINESS_INFO.name}`}
          />
        </div>
      )}
    </div>
  );
}
```

## 14.4 Contact Page SEO

```tsx
// app/contact/page.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Contattaci | Nome Azienda Milano',
  description: 'Contatta Nome Azienda a Milano. Telefono: 02 123 4567. Email: info@example.com. Via Roma 123, 20100 Milano.',
  
  openGraph: {
    title: 'Contattaci',
    description: 'Contatta Nome Azienda a Milano',
  },
  
  alternates: {
    canonical: '/contact',
  },
};

export default function ContactPage() {
  const contactPageSchema = {
    '@context': 'https://schema.org',
    '@type': 'ContactPage',
    name: 'Contattaci',
    description: 'Pagina contatti di Nome Azienda',
    mainEntity: {
      '@type': 'LocalBusiness',
      name: 'Nome Azienda Srl',
      telephone: '+39 02 1234567',
      email: 'info@example.com',
      address: {
        '@type': 'PostalAddress',
        streetAddress: 'Via Roma 123',
        addressLocality: 'Milano',
        postalCode: '20100',
        addressCountry: 'IT',
      },
    },
  };
  
  return (
    <>
      <JsonLd data={contactPageSchema} />
      
      <main>
        <h1>Contattaci</h1>
        
        <div className="grid md:grid-cols-2 gap-8">
          {/* Info azienda con NAP */}
          <section>
            <h2>I Nostri Contatti</h2>
            <BusinessInfo showMap={true} />
            
            {/* Orari apertura */}
            <section className="mt-6">
              <h3>Orari di Apertura</h3>
              <table>
                <tbody>
                  <tr>
                    <td>Lunedì - Venerdì</td>
                    <td>09:00 - 18:00</td>
                  </tr>
                  <tr>
                    <td>Sabato</td>
                    <td>10:00 - 14:00</td>
                  </tr>
                  <tr>
                    <td>Domenica</td>
                    <td>Chiuso</td>
                  </tr>
                </tbody>
              </table>
            </section>
          </section>
          
          {/* Form contatto */}
          <section>
            <h2>Scrivici un Messaggio</h2>
            <ContactForm />
          </section>
        </div>
      </main>
    </>
  );
}
```

## 14.5 Multi-Location SEO

```tsx
// Per business con più sedi
// app/locations/page.tsx

export const metadata: Metadata = {
  title: 'Le Nostre Sedi | Brand Name',
  description: 'Trova la sede Brand Name più vicina a te. Presenti a Milano, Roma, Napoli e Torino.',
};

export default async function LocationsPage() {
  const locations = await getLocations();
  
  return (
    <main>
      <h1>Le Nostre Sedi</h1>
      
      {locations.map((location) => (
        <article key={location.id} itemScope itemType="https://schema.org/LocalBusiness">
          <h2 itemProp="name">{location.name}</h2>
          <address itemProp="address" itemScope itemType="https://schema.org/PostalAddress">
            <span itemProp="streetAddress">{location.address.street}</span>,{' '}
            <span itemProp="addressLocality">{location.address.city}</span>
          </address>
          <p>
            <a href={`tel:${location.phone}`} itemProp="telephone">{location.phone}</a>
          </p>
          <a href={`/locations/${location.slug}`}>
            Scopri di più →
          </a>
        </article>
      ))}
    </main>
  );
}

// app/locations/[slug]/page.tsx - Pagina singola sede
export async function generateMetadata({ params }): Promise<Metadata> {
  const location = await getLocation(params.slug);
  
  return {
    title: `${location.name} | Brand Name ${location.city}`,
    description: `Visita Brand Name a ${location.city}. ${location.address.street}, ${location.address.postalCode}. Tel: ${location.phone}`,
    
    alternates: {
      canonical: `/locations/${params.slug}`,
    },
  };
}

export default async function LocationPage({ params }) {
  const location = await getLocation(params.slug);
  
  const locationSchema = getLocalBusinessSchema({
    name: `Brand Name ${location.city}`,
    description: location.description,
    // ... altri campi
  });
  
  return (
    <>
      <JsonLd data={locationSchema} />
      
      <main>
        <Breadcrumbs items={[
          { name: 'Home', url: '/' },
          { name: 'Sedi', url: '/locations' },
          { name: location.name, url: `/locations/${params.slug}` },
        ]} />
        
        <h1>{location.name}</h1>
        
        <BusinessInfo location={location} showMap={true} />
        
        {/* Contenuto localizzato */}
        <section>
          <h2>Servizi disponibili a {location.city}</h2>
          {/* ... */}
        </section>
      </main>
    </>
  );
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 15: SEO MONITORING & AUDIT
# ═══════════════════════════════════════════════════════════════════════════════

SEO_MONITORING_AUDIT = """

## 15.1 SEO Monitoring Tools Setup

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ SEO MONITORING STACK                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ ESSENZIALI (Gratuiti)                                                       │
│ ├── Google Search Console                                                  │
│ │   └── Indexing, coverage, CTR, impressions, queries                      │
│ ├── Google Analytics 4                                                     │
│ │   └── Traffic, conversions, user behavior                                │
│ └── Bing Webmaster Tools                                                   │
│     └── Indexing per Bing                                                  │
│                                                                             │
│ TECHNICAL SEO                                                               │
│ ├── Lighthouse (built-in Chrome DevTools)                                  │
│ ├── PageSpeed Insights                                                     │
│ ├── Screaming Frog SEO Spider (free fino a 500 URLs)                       │
│ └── Chrome DevTools (Network, Performance)                                 │
│                                                                             │
│ MONITORAGGIO AVANZATO (Paid)                                               │
│ ├── Ahrefs / SEMrush / Moz                                                 │
│ │   └── Rankings, backlinks, competitor analysis                           │
│ ├── ContentKing / Lumar                                                    │
│ │   └── Monitoraggio real-time technical SEO                               │
│ └── BrightEdge / Conductor                                                 │
│     └── Enterprise SEO platforms                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 15.2 Google Search Console Integration

```typescript
// lib/seo/search-console.ts
// API per estrarre dati da Google Search Console

import { google } from 'googleapis';

const auth = new google.auth.GoogleAuth({
  credentials: JSON.parse(process.env.GOOGLE_SERVICE_ACCOUNT_KEY || '{}'),
  scopes: ['https://www.googleapis.com/auth/webmasters.readonly'],
});

const searchconsole = google.searchconsole({ version: 'v1', auth });

interface SearchAnalyticsQuery {
  siteUrl: string;
  startDate: string;
  endDate: string;
  dimensions?: string[];
  rowLimit?: number;
}

export async function getSearchAnalytics(query: SearchAnalyticsQuery) {
  const response = await searchconsole.searchanalytics.query({
    siteUrl: query.siteUrl,
    requestBody: {
      startDate: query.startDate,
      endDate: query.endDate,
      dimensions: query.dimensions || ['query', 'page'],
      rowLimit: query.rowLimit || 1000,
    },
  });
  
  return response.data.rows?.map((row) => ({
    query: row.keys?.[0],
    page: row.keys?.[1],
    clicks: row.clicks,
    impressions: row.impressions,
    ctr: row.ctr,
    position: row.position,
  })) || [];
}

// Fetch top queries
export async function getTopQueries(siteUrl: string, days: number = 28) {
  const endDate = new Date();
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  return getSearchAnalytics({
    siteUrl,
    startDate: startDate.toISOString().split('T')[0],
    endDate: endDate.toISOString().split('T')[0],
    dimensions: ['query'],
    rowLimit: 100,
  });
}

// Fetch page performance
export async function getPagePerformance(siteUrl: string, pageUrl: string, days: number = 28) {
  const endDate = new Date();
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const response = await searchconsole.searchanalytics.query({
    siteUrl,
    requestBody: {
      startDate: startDate.toISOString().split('T')[0],
      endDate: endDate.toISOString().split('T')[0],
      dimensions: ['date'],
      dimensionFilterGroups: [
        {
          filters: [
            {
              dimension: 'page',
              expression: pageUrl,
            },
          ],
        },
      ],
    },
  });
  
  return response.data.rows;
}
```

## 15.3 SEO Dashboard Component

```tsx
// app/admin/seo-dashboard/page.tsx
import { getTopQueries, getPagePerformance } from '@/lib/seo/search-console';

export default async function SEODashboard() {
  const siteUrl = 'https://www.example.com';
  
  const [topQueries, pagePerformance] = await Promise.all([
    getTopQueries(siteUrl, 28),
    getPagePerformance(siteUrl, '/products', 28),
  ]);
  
  return (
    <div className="p-6">
      <h1 className="text-2xl font-bold mb-6">SEO Dashboard</h1>
      
      {/* Overview Cards */}
      <div className="grid grid-cols-4 gap-4 mb-8">
        <MetricCard
          title="Total Clicks"
          value={topQueries.reduce((sum, q) => sum + q.clicks, 0)}
          change="+12%"
        />
        <MetricCard
          title="Total Impressions"
          value={topQueries.reduce((sum, q) => sum + q.impressions, 0)}
          change="+8%"
        />
        <MetricCard
          title="Average CTR"
          value={`${(topQueries.reduce((sum, q) => sum + q.ctr, 0) / topQueries.length * 100).toFixed(2)}%`}
        />
        <MetricCard
          title="Average Position"
          value={(topQueries.reduce((sum, q) => sum + q.position, 0) / topQueries.length).toFixed(1)}
        />
      </div>
      
      {/* Top Queries Table */}
      <section className="mb-8">
        <h2 className="text-xl font-bold mb-4">Top Queries</h2>
        <table className="w-full">
          <thead>
            <tr>
              <th>Query</th>
              <th>Clicks</th>
              <th>Impressions</th>
              <th>CTR</th>
              <th>Position</th>
            </tr>
          </thead>
          <tbody>
            {topQueries.slice(0, 20).map((query, index) => (
              <tr key={index}>
                <td>{query.query}</td>
                <td>{query.clicks}</td>
                <td>{query.impressions}</td>
                <td>{(query.ctr * 100).toFixed(2)}%</td>
                <td>{query.position?.toFixed(1)}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </section>
      
      {/* Performance Chart */}
      <section>
        <h2 className="text-xl font-bold mb-4">Performance Over Time</h2>
        <PerformanceChart data={pagePerformance} />
      </section>
    </div>
  );
}
```

## 15.4 Automated SEO Audit Script

```typescript
// scripts/seo-audit.ts
import { chromium } from 'playwright';

interface SEOAuditResult {
  url: string;
  title: string | null;
  titleLength: number;
  description: string | null;
  descriptionLength: number;
  h1Count: number;
  h1Text: string | null;
  canonicalUrl: string | null;
  robotsMeta: string | null;
  ogTitle: string | null;
  ogDescription: string | null;
  ogImage: string | null;
  structuredData: object[];
  imagesWithoutAlt: number;
  internalLinks: number;
  externalLinks: number;
  brokenLinks: string[];
  issues: string[];
}

async function auditPage(url: string): Promise<SEOAuditResult> {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  
  await page.goto(url, { waitUntil: 'networkidle' });
  
  const result: SEOAuditResult = await page.evaluate(() => {
    const issues: string[] = [];
    
    // Title
    const title = document.querySelector('title')?.textContent || null;
    const titleLength = title?.length || 0;
    if (!title) issues.push('Missing title tag');
    else if (titleLength < 30) issues.push('Title too short (<30 chars)');
    else if (titleLength > 60) issues.push('Title too long (>60 chars)');
    
    // Description
    const description = document.querySelector('meta[name="description"]')?.getAttribute('content') || null;
    const descriptionLength = description?.length || 0;
    if (!description) issues.push('Missing meta description');
    else if (descriptionLength < 70) issues.push('Meta description too short');
    else if (descriptionLength > 160) issues.push('Meta description too long');
    
    // H1
    const h1Elements = document.querySelectorAll('h1');
    const h1Count = h1Elements.length;
    const h1Text = h1Elements[0]?.textContent || null;
    if (h1Count === 0) issues.push('Missing H1 tag');
    else if (h1Count > 1) issues.push(`Multiple H1 tags (${h1Count})`);
    
    // Canonical
    const canonicalUrl = document.querySelector('link[rel="canonical"]')?.getAttribute('href') || null;
    if (!canonicalUrl) issues.push('Missing canonical URL');
    
    // Robots
    const robotsMeta = document.querySelector('meta[name="robots"]')?.getAttribute('content') || null;
    
    // Open Graph
    const ogTitle = document.querySelector('meta[property="og:title"]')?.getAttribute('content') || null;
    const ogDescription = document.querySelector('meta[property="og:description"]')?.getAttribute('content') || null;
    const ogImage = document.querySelector('meta[property="og:image"]')?.getAttribute('content') || null;
    if (!ogTitle) issues.push('Missing og:title');
    if (!ogImage) issues.push('Missing og:image');
    
    // Structured Data
    const structuredDataScripts = document.querySelectorAll('script[type="application/ld+json"]');
    const structuredData: object[] = [];
    structuredDataScripts.forEach((script) => {
      try {
        structuredData.push(JSON.parse(script.textContent || ''));
      } catch (e) {
        issues.push('Invalid JSON-LD structured data');
      }
    });
    if (structuredData.length === 0) issues.push('No structured data found');
    
    // Images without alt
    const images = document.querySelectorAll('img');
    const imagesWithoutAlt = Array.from(images).filter(img => !img.alt).length;
    if (imagesWithoutAlt > 0) issues.push(`${imagesWithoutAlt} images without alt text`);
    
    // Links
    const links = document.querySelectorAll('a[href]');
    let internalLinks = 0;
    let externalLinks = 0;
    const currentHost = window.location.host;
    
    links.forEach((link) => {
      const href = link.getAttribute('href') || '';
      if (href.startsWith('/') || href.includes(currentHost)) {
        internalLinks++;
      } else if (href.startsWith('http')) {
        externalLinks++;
      }
    });
    
    return {
      url: window.location.href,
      title,
      titleLength,
      description,
      descriptionLength,
      h1Count,
      h1Text,
      canonicalUrl,
      robotsMeta,
      ogTitle,
      ogDescription,
      ogImage,
      structuredData,
      imagesWithoutAlt,
      internalLinks,
      externalLinks,
      brokenLinks: [], // Check separately
      issues,
    };
  });
  
  await browser.close();
  return result;
}

// Run audit on multiple pages
async function runFullAudit(urls: string[]) {
  const results: SEOAuditResult[] = [];
  
  for (const url of urls) {
    console.log(`Auditing: ${url}`);
    try {
      const result = await auditPage(url);
      results.push(result);
    } catch (error) {
      console.error(`Error auditing ${url}:`, error);
    }
  }
  
  // Generate report
  console.log('\n=== SEO AUDIT REPORT ===\n');
  
  for (const result of results) {
    console.log(`URL: ${result.url}`);
    console.log(`Title: ${result.title} (${result.titleLength} chars)`);
    console.log(`H1: ${result.h1Text}`);
    console.log(`Issues: ${result.issues.length}`);
    result.issues.forEach((issue) => console.log(`  - ${issue}`));
    console.log('---');
  }
  
  return results;
}

// Usage
const pagesToAudit = [
  'https://www.example.com',
  'https://www.example.com/products',
  'https://www.example.com/blog',
  'https://www.example.com/contact',
];

runFullAudit(pagesToAudit);
```

## 15.5 Lighthouse CI Integration

```yaml
# .github/workflows/lighthouse-ci.yml
name: Lighthouse CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
      
      - name: Start server
        run: npm run start &
        
      - name: Wait for server
        run: npx wait-on http://localhost:3000
      
      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v11
        with:
          urls: |
            http://localhost:3000
            http://localhost:3000/products
            http://localhost:3000/blog
          budgetPath: ./lighthouse-budget.json
          uploadArtifacts: true
          temporaryPublicStorage: true
```

```json
// lighthouse-budget.json
[
  {
    "path": "/*",
    "resourceSizes": [
      { "resourceType": "total", "budget": 500 },
      { "resourceType": "script", "budget": 200 },
      { "resourceType": "image", "budget": 150 },
      { "resourceType": "stylesheet", "budget": 50 }
    ],
    "resourceCounts": [
      { "resourceType": "script", "budget": 20 },
      { "resourceType": "total", "budget": 50 }
    ],
    "timings": [
      { "metric": "interactive", "budget": 3000 },
      { "metric": "first-contentful-paint", "budget": 1500 },
      { "metric": "largest-contentful-paint", "budget": 2500 }
    ]
  }
]
```

```javascript
// lighthouserc.js
module.exports = {
  ci: {
    collect: {
      numberOfRuns: 3,
      startServerCommand: 'npm run start',
      url: [
        'http://localhost:3000',
        'http://localhost:3000/products',
        'http://localhost:3000/blog',
      ],
    },
    assert: {
      preset: 'lighthouse:recommended',
      assertions: {
        'categories:performance': ['error', { minScore: 0.9 }],
        'categories:accessibility': ['error', { minScore: 0.95 }],
        'categories:best-practices': ['error', { minScore: 0.9 }],
        'categories:seo': ['error', { minScore: 0.95 }],
        
        // SEO specific assertions
        'meta-description': 'error',
        'document-title': 'error',
        'html-has-lang': 'error',
        'link-text': 'warn',
        'crawlable-anchors': 'error',
        'robots-txt': 'error',
        'hreflang': 'warn',
        'canonical': 'error',
        'structured-data': 'warn',
      },
    },
    upload: {
      target: 'temporary-public-storage',
    },
  },
};
```

## 15.6 SEO Checklist Pre-Deployment

```typescript
// scripts/pre-deploy-seo-check.ts

interface SEOCheckResult {
  check: string;
  passed: boolean;
  message: string;
}

async function runPreDeploySEOChecks(): Promise<SEOCheckResult[]> {
  const results: SEOCheckResult[] = [];
  
  // 1. Check robots.txt
  const robotsResponse = await fetch('http://localhost:3000/robots.txt');
  results.push({
    check: 'robots.txt',
    passed: robotsResponse.ok,
    message: robotsResponse.ok ? 'robots.txt is accessible' : 'robots.txt not found',
  });
  
  // 2. Check sitemap.xml
  const sitemapResponse = await fetch('http://localhost:3000/sitemap.xml');
  results.push({
    check: 'sitemap.xml',
    passed: sitemapResponse.ok,
    message: sitemapResponse.ok ? 'sitemap.xml is accessible' : 'sitemap.xml not found',
  });
  
  // 3. Check critical pages have metadata
  const criticalPages = ['/', '/products', '/blog', '/contact'];
  
  for (const page of criticalPages) {
    const response = await fetch(`http://localhost:3000${page}`);
    const html = await response.text();
    
    const hasTitle = html.includes('<title>');
    const hasDescription = html.includes('name="description"');
    const hasOgImage = html.includes('property="og:image"');
    
    results.push({
      check: `Metadata: ${page}`,
      passed: hasTitle && hasDescription && hasOgImage,
      message: `Title: ${hasTitle ? '✓' : '✗'}, Description: ${hasDescription ? '✓' : '✗'}, OG Image: ${hasOgImage ? '✓' : '✗'}`,
    });
  }
  
  // 4. Check for structured data on key pages
  for (const page of criticalPages) {
    const response = await fetch(`http://localhost:3000${page}`);
    const html = await response.text();
    
    const hasStructuredData = html.includes('application/ld+json');
    
    results.push({
      check: `Structured Data: ${page}`,
      passed: hasStructuredData,
      message: hasStructuredData ? 'Has JSON-LD' : 'Missing JSON-LD',
    });
  }
  
  // 5. Check canonical URLs
  for (const page of criticalPages) {
    const response = await fetch(`http://localhost:3000${page}`);
    const html = await response.text();
    
    const hasCanonical = html.includes('rel="canonical"');
    
    results.push({
      check: `Canonical: ${page}`,
      passed: hasCanonical,
      message: hasCanonical ? 'Has canonical' : 'Missing canonical',
    });
  }
  
  return results;
}

async function main() {
  console.log('Running SEO Pre-Deployment Checks...\n');
  
  const results = await runPreDeploySEOChecks();
  
  let allPassed = true;
  
  for (const result of results) {
    const icon = result.passed ? '✅' : '❌';
    console.log(`${icon} ${result.check}: ${result.message}`);
    
    if (!result.passed) {
      allPassed = false;
    }
  }
  
  console.log('\n---');
  
  if (allPassed) {
    console.log('✅ All SEO checks passed!');
    process.exit(0);
  } else {
    console.log('❌ Some SEO checks failed. Please fix before deploying.');
    process.exit(1);
  }
}

main();
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# APPENDICE: QUICK REFERENCE
# ═══════════════════════════════════════════════════════════════════════════════

APPENDIX_QUICK_REFERENCE = """

## A.1 SEO Metadata Quick Reference

```typescript
// Lunghezze raccomandate
const SEO_LENGTHS = {
  title: { min: 30, max: 60 },      // 50-60 chars ideale
  description: { min: 70, max: 160 }, // 150-160 chars ideale
  ogTitle: { min: 30, max: 90 },
  ogDescription: { min: 70, max: 200 },
  altText: { min: 5, max: 125 },
  urlSlug: { max: 75 },
};

// Meta tags essenziali per ogni pagina
const ESSENTIAL_META_TAGS = `
  <title>Page Title | Brand</title>
  <meta name="description" content="Description..." />
  <link rel="canonical" href="https://example.com/page" />
  <meta name="robots" content="index, follow" />
  
  <!-- Open Graph -->
  <meta property="og:title" content="Title" />
  <meta property="og:description" content="Description" />
  <meta property="og:image" content="https://example.com/og.jpg" />
  <meta property="og:url" content="https://example.com/page" />
  <meta property="og:type" content="website" />
  
  <!-- Twitter -->
  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:title" content="Title" />
  <meta name="twitter:description" content="Description" />
  <meta name="twitter:image" content="https://example.com/twitter.jpg" />
`;
```

## A.2 Schema Types Quick Reference

```typescript
// Schema types più comuni
const COMMON_SCHEMA_TYPES = {
  website: {
    '@type': 'WebSite',
    required: ['name', 'url'],
    optional: ['potentialAction (SearchAction)'],
  },
  organization: {
    '@type': 'Organization',
    required: ['name', 'url'],
    optional: ['logo', 'contactPoint', 'sameAs', 'address'],
  },
  localBusiness: {
    '@type': 'LocalBusiness',
    required: ['name', 'address'],
    optional: ['telephone', 'openingHours', 'geo', 'priceRange'],
  },
  product: {
    '@type': 'Product',
    required: ['name'],
    recommended: ['image', 'description', 'offers', 'aggregateRating'],
  },
  article: {
    '@type': 'Article',
    required: ['headline', 'author', 'datePublished'],
    recommended: ['image', 'dateModified', 'publisher'],
  },
  breadcrumbList: {
    '@type': 'BreadcrumbList',
    required: ['itemListElement'],
  },
  faqPage: {
    '@type': 'FAQPage',
    required: ['mainEntity (Question[])'],
  },
};
```

## A.3 robots.txt Rules Quick Reference

```
# Permettere tutto
User-agent: *
Allow: /

# Bloccare tutto
User-agent: *
Disallow: /

# Bloccare directory specifica
Disallow: /admin/
Disallow: /private/

# Bloccare file specifico
Disallow: /secret.html

# Bloccare query strings
Disallow: /*?*

# Bloccare file per estensione
Disallow: /*.pdf$

# Sitemap
Sitemap: https://example.com/sitemap.xml

# Crawl delay (non supportato da Google)
Crawl-delay: 10
```

## A.4 Core Web Vitals Thresholds

```
┌─────────────┬──────────┬───────────┬──────────┐
│ Metric      │ Good     │ Needs Imp │ Poor     │
├─────────────┼──────────┼───────────┼──────────┤
│ LCP         │ ≤2.5s    │ 2.5-4.0s  │ >4.0s    │
│ INP         │ ≤200ms   │ 200-500ms │ >500ms   │
│ CLS         │ ≤0.1     │ 0.1-0.25  │ >0.25    │
└─────────────┴──────────┴───────────┴──────────┘
```

## A.5 Useful SEO Commands

```bash
# Test robots.txt
curl -I https://example.com/robots.txt

# Test sitemap
curl -s https://example.com/sitemap.xml | head -50

# Check HTTP headers
curl -I https://example.com/

# Check redirects
curl -IL https://example.com/old-page

# Lighthouse CLI
npx lighthouse https://example.com --view

# Google PageSpeed Insights API
curl "https://www.googleapis.com/pagespeedonline/v5/runPagespeed?url=https://example.com&strategy=mobile"

# Validate structured data
npx schema-dts-gen validate ./schema.json

# Check mobile friendliness
curl "https://searchconsole.googleapis.com/v1/urlTestingTools/mobileFriendlyTest:run" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com"}'
```

## A.6 SEO Checklist Template

```markdown
# SEO Checklist - [Page Name]

## Technical SEO
- [ ] HTTPS enabled
- [ ] Mobile-responsive
- [ ] Page loads < 3s
- [ ] No broken links
- [ ] robots.txt configured
- [ ] sitemap.xml present
- [ ] Canonical URL set

## On-Page SEO
- [ ] Unique title (50-60 chars)
- [ ] Meta description (150-160 chars)
- [ ] Single H1 tag
- [ ] Heading hierarchy (H1>H2>H3)
- [ ] Alt text on images
- [ ] Internal links
- [ ] External links (nofollow where appropriate)

## Structured Data
- [ ] Organization schema (homepage)
- [ ] BreadcrumbList schema
- [ ] Page-specific schema (Article/Product/etc.)
- [ ] Validated with Google Rich Results Test

## Social Media
- [ ] og:title set
- [ ] og:description set
- [ ] og:image set (1200x630)
- [ ] twitter:card set

## Performance
- [ ] LCP ≤ 2.5s
- [ ] INP ≤ 200ms
- [ ] CLS ≤ 0.1

## Monitoring
- [ ] Google Search Console verified
- [ ] Google Analytics installed
- [ ] Sitemap submitted
- [ ] Core Web Vitals passing
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# FINE CATALOGO SEO
# ═══════════════════════════════════════════════════════════════════════════════

print("""
═══════════════════════════════════════════════════════════════════════════════
CATALOGO SEO v1.0 - COMPLETATO
═══════════════════════════════════════════════════════════════════════════════

SEZIONI INCLUSE:
1.  SEO Fundamentals
2.  Next.js Metadata API
3.  Structured Data (JSON-LD)
4.  Sitemap & Robots.txt
5.  URL Structure & Routing
6.  Open Graph & Social Media
7.  International SEO (i18n/hreflang)
8.  Image SEO
9.  Core Web Vitals & Performance SEO
10. Semantic HTML & Accessibility
11. Internal Linking & Navigation
12. E-commerce SEO
13. Blog & Content SEO
14. Local Business SEO
15. SEO Monitoring & Audit
+   Appendix: Quick Reference

CARATTERISTICHE:
✅ Best practices 2025 aggiornate
✅ INP come nuova metrica Core Web Vitals
✅ Next.js 14+ App Router patterns
✅ Structured Data con schema-dts
✅ TypeScript completo
✅ Component-based architecture
✅ Automated SEO audit scripts
✅ CI/CD integration con Lighthouse

═══════════════════════════════════════════════════════════════════════════════
""")
