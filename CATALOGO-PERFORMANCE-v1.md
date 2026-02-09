# ═══════════════════════════════════════════════════════════════════════════════
# CATALOGO PERFORMANCE v1.0
# ═══════════════════════════════════════════════════════════════════════════════
#
§ GUIDA COMPLETA ALL'OTTIMIZZAZIONE DELLE PERFORMANCE WEB
# Core Web Vitals, Image Optimization, Caching, Bundle Optimization
#
# Data creazione: 2026-01-26
# Ultima modifica: 2026-01-26
#
# ═══════════════════════════════════════════════════════════════════════════════

"""
INDICE:
═══════════════════════════════════════════════════════════════════════════════
1.  Core Web Vitals Overview
2.  LCP - Largest Contentful Paint
3.  INP - Interaction to Next Paint
4.  CLS - Cumulative Layout Shift
5.  Image Optimization
6.  JavaScript Optimization
7.  CSS Optimization
8.  Font Optimization
9.  Caching Strategies
10. CDN & Edge Computing
11. Server-Side Rendering & Static Generation
12. Database & API Performance
13. Performance Monitoring
14. Performance Budgets
15. Checklist Performance Pre-Deployment
═══════════════════════════════════════════════════════════════════════════════
"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 1: CORE WEB VITALS OVERVIEW
# ═══════════════════════════════════════════════════════════════════════════════

CORE_WEB_VITALS_OVERVIEW = """

§ 1.1 CORE WEB VITALS 2025

┌─────────────────────────────────────────────────────────────────────────────┐
│ CORE WEB VITALS - METRICHE 2025                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐              │
│ │      LCP        │  │      INP        │  │      CLS        │              │
│ │   Loading       │  │  Interactivity  │  │ Visual Stability│              │
│ └────────┬────────┘  └────────┬────────┘  └────────┬────────┘              │
│          │                    │                    │                       │
│  ┌───────┴───────┐    ┌───────┴───────┐    ┌───────┴───────┐              │
│  │ GOOD: ≤2.5s   │    │ GOOD: ≤200ms  │    │ GOOD: ≤0.1    │              │
│  │ NEEDS: 2.5-4s │    │ NEEDS: 200-500│    │ NEEDS: 0.1-0.25│              │
│  │ POOR: >4s     │    │ POOR: >500ms  │    │ POOR: >0.25   │              │
│  └───────────────┘    └───────────────┘    └───────────────┘              │
│                                                                             │
│ NOTA: INP ha sostituito FID (First Input Delay) a Marzo 2024               │
│                                                                             │
│ Misurazione: 75° percentile dei dati utente reali (CrUX)                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 1.2 METRICHE SUPPLEMENTARI

┌─────────────────────────────────────────────────────────────────────────────┐
│ METRICHE SUPPLEMENTARI                                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ TTFB (Time to First Byte)                                                  │
│ ├── Tempo fino al primo byte dal server                                    │
│ ├── Target: < 800ms                                                        │
│ └── Indica problemi server, network, o CDN                                 │
│                                                                             │
│ FCP (First Contentful Paint)                                               │
│ ├── Tempo fino al primo contenuto visibile                                 │
│ ├── Target: < 1.8s                                                         │
│ └── Indica problemi di render-blocking resources                           │
│                                                                             │
│ TBT (Total Blocking Time)                                                  │
│ ├── Tempo totale di blocco main thread tra FCP e TTI                       │
│ ├── Target: < 200ms                                                        │
│ └── Lab metric proxy per INP                                               │
│                                                                             │
│ TTI (Time to Interactive)                                                  │
│ ├── Tempo fino alla piena interattività                                    │
│ ├── Target: < 3.8s                                                         │
│ └── Quando la pagina risponde in < 50ms                                    │
│                                                                             │
│ Speed Index                                                                 │
│ ├── Velocità di rendering visivo del contenuto                             │
│ ├── Target: < 3.4s                                                         │
│ └── Misura progressione visuale                                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 1.3 IMPATTO BUSINESS DELLE PERFORMANCE

┌─────────────────────────────────────────────────────────────────────────────┐
│ PERFORMANCE → BUSINESS IMPACT                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ CONVERSION RATE                                                             │
│ ├── +100ms load time → -7% conversioni (Amazon)                            │
│ ├── Poor → Good CWV → +25% conversion rate                                 │
│ └── 1s delay → -11% page views                                             │
│                                                                             │
│ BOUNCE RATE                                                                 │
│ ├── 53% utenti mobile abbandonano se >3s                                   │
│ ├── Good CWV → -35% bounce rate                                            │
│ └── Ogni secondo aggiuntivo → +32% bounce rate                             │
│                                                                             │
│ SEO RANKING                                                                 │
│ ├── Core Web Vitals = ~10-15% ranking signals                              │
│ ├── Good CWV → +8-15% visibility boost                                     │
│ └── Mobile-first indexing prioritizza performance                          │
│                                                                             │
│ REVENUE                                                                     │
│ ├── 1s improvement → +2% revenue (Walmart)                                 │
│ ├── Poor performance → -30% revenue potential                              │
│ └── Good CWV → +30% engagement                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 1.4 STRUMENTI DI MISURAZIONE

typescript
// Field Data (Real User Monitoring)
// - Chrome User Experience Report (CrUX)
// - Google Search Console
// - PageSpeed Insights (field data section)
// - web-vitals library

// Lab Data (Controlled Testing)
// - Lighthouse
// - Chrome DevTools Performance panel
// - WebPageTest
// - PageSpeed Insights (lab data section)

// Installazione web-vitals
// pnpm add web-vitals

// lib/analytics/web-vitals.ts
import { onCLS, onINP, onLCP, onFCP, onTTFB, Metric } from 'web-vitals';

type MetricHandler = (metric: Metric) => void;

// Invia a analytics
function sendToAnalytics(metric: Metric) {
  const body = JSON.stringify({
    name: metric.name,
    value: metric.value,
    rating: metric.rating,
    delta: metric.delta,
    id: metric.id,
    navigationType: metric.navigationType,
    // Additional context
    url: window.location.href,
    timestamp: Date.now(),
  });

  // Use sendBeacon for reliability
  if (navigator.sendBeacon) {
    navigator.sendBeacon('/api/analytics/vitals', body);
  } else {
    fetch('/api/analytics/vitals', {
      body,
      method: 'POST',
      keepalive: true,
    });
  }
}

// Inizializza tracking
export function initWebVitals() {
  onCLS(sendToAnalytics);
  onINP(sendToAnalytics);
  onLCP(sendToAnalytics);
  onFCP(sendToAnalytics);
  onTTFB(sendToAnalytics);
}

// Console logging per development
export function logWebVitals() {
  const logMetric: MetricHandler = (metric) => {
    console.log(`[${metric.name}]`, {
      value: metric.value.toFixed(2),
      rating: metric.rating,
      delta: metric.delta.toFixed(2),
    });
  };

  onCLS(logMetric);
  onINP(logMetric);
  onLCP(logMetric);
  onFCP(logMetric);
  onTTFB(logMetric);
}

typescript
// app/layout.tsx - Next.js App Router
'use client';

import { useEffect } from 'react';
import { initWebVitals, logWebVitals } from '@/lib/analytics/web-vitals';

export function WebVitalsReporter() {
  useEffect(() => {
    if (process.env.NODE_ENV === 'production') {
      initWebVitals();
    } else {
      logWebVitals();
    }
  }, []);

  return null;
}

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 2: LCP - LARGEST CONTENTFUL PAINT
# ═══════════════════════════════════════════════════════════════════════════════

LCP_OPTIMIZATION = """

§ 2.1 COS'È LCP

┌─────────────────────────────────────────────────────────────────────────────┐
│ LCP - LARGEST CONTENTFUL PAINT                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ DEFINIZIONE                                                                 │
│ Tempo necessario per renderizzare l'elemento di contenuto più grande       │
│ visibile nel viewport.                                                      │
│                                                                             │
│ ELEMENTI LCP CANDIDATI                                                      │
│ ├── <img> elements                                                         │
│ ├── <image> inside <svg>                                                   │
│ ├── <video> poster image (or first frame)                                  │
│ ├── Background images via url()                                            │
│ ├── Block-level elements with text nodes                                   │
│ └── <canvas> elements                                                      │
│                                                                             │
│ THRESHOLDS                                                                  │
│ ├── GOOD:    ≤ 2.5 seconds                                                 │
│ ├── NEEDS:   2.5 - 4.0 seconds                                             │
│ └── POOR:    > 4.0 seconds                                                 │
│                                                                             │
│ FATTORI CHE INFLUENZANO LCP                                                │
│ ├── Server response time (TTFB)                                            │
│ ├── Render-blocking resources (CSS, JS)                                    │
│ ├── Resource load time (images, fonts)                                     │
│ └── Client-side rendering delay                                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 2.2 OTTIMIZZAZIONE SERVER RESPONSE TIME

typescript
// Ridurre TTFB

// 1. Utilizzare CDN
// Configurazione Vercel (automatic)
// Configurazione Cloudflare

// 2. Edge Functions per contenuto dinamico
// app/api/products/route.ts
import { NextRequest, NextResponse } from 'next/server';

export const runtime = 'edge'; // Run at edge locations

export async function GET(request: NextRequest) {
  // Cache at edge
  const response = NextResponse.json(await getProducts());
  
  response.headers.set(
    'Cache-Control',
    'public, s-maxage=60, stale-while-revalidate=300'
  );
  
  return response;
}

// 3. Database connection pooling
// lib/db.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient({
  log: process.env.NODE_ENV === 'development' ? ['query'] : [],
});

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}

// 4. Streaming SSR
// app/products/page.tsx
import { Suspense } from 'react';

export default function ProductsPage() {
  return (
    <div>
      <h1>Products</h1>
      <Suspense fallback={<ProductsSkeleton />}>
        <ProductsList />
      </Suspense>
    </div>
  );
}

// Immediate response, stream content as ready
async function ProductsList() {
  const products = await fetchProducts();
  return <ProductGrid products={products} />;
}

§ 2.3 ELIMINARE RENDER-BLOCKING RESOURCES

typescript
// next.config.js - Ottimizzazioni automatiche Next.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Automatic CSS optimization
  optimizeFonts: true,
  
  // Experimental features
  experimental: {
    // Optimize CSS loading
    optimizeCss: true,
  },
};

module.exports = nextConfig;

html
<!-- Critical CSS inline -->
<head>
  <!-- Critical CSS inline per above-the-fold -->
  <style>
    /* Critical styles only */
    .hero { ... }
    .nav { ... }
  </style>
  
  <!-- Non-critical CSS deferred -->
  <link rel="preload" href="/styles/main.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
  <noscript><link rel="stylesheet" href="/styles/main.css"></noscript>
</head>

typescript
// Script loading strategies - Next.js
import Script from 'next/script';

export default function Layout({ children }) {
  return (
    <>
      {children}
      
      {/* After page is interactive */}
      <Script
        src="https://analytics.example.com/script.js"
        strategy="afterInteractive"
      />
      
      {/* On browser idle */}
      <Script
        src="https://chat-widget.example.com/widget.js"
        strategy="lazyOnload"
      />
      
      {/* Worker thread (experimental) */}
      <Script
        src="https://heavy-analytics.example.com/track.js"
        strategy="worker"
      />
    </>
  );
}

§ 2.4 OTTIMIZZAZIONE LCP ELEMENT

typescript
// components/HeroImage.tsx
import Image from 'next/image';

export function HeroImage() {
  return (
    <Image
      src="/hero.jpg"
      alt="Hero"
      width={1200}
      height={600}
      // CRITICAL: priority per LCP element
      priority
      // Placeholder blur mentre carica
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRg..."
      // Responsive sizes
      sizes="100vw"
      // Quality optimization
      quality={85}
    />
  );
}

// Per immagini above-the-fold (LCP candidates)
// - Usare priority={true}
// - Mai usare loading="lazy"
// - Preload manuale se necessario

html
<!-- Preload LCP image -->
<head>
  <link 
    rel="preload" 
    as="image" 
    href="/hero.webp"
    imagesrcset="/hero-400.webp 400w, /hero-800.webp 800w, /hero-1200.webp 1200w"
    imagesizes="100vw"
  />
  
  <!-- Preconnect to image CDN -->
  <link rel="preconnect" href="https://images.example.com" />
  <link rel="dns-prefetch" href="https://images.example.com" />
</head>

§ 2.5 RESOURCE HINTS

typescript
// app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html>
      <head>
        {/* Preconnect: stabilisce connessione anticipata */}
        <link rel="preconnect" href="https://fonts.googleapis.com" />
        <link rel="preconnect" href="https://cdn.example.com" crossOrigin="anonymous" />
        
        {/* DNS Prefetch: risolve DNS in anticipo */}
        <link rel="dns-prefetch" href="https://api.example.com" />
        
        {/* Preload: carica risorse critiche */}
        <link 
          rel="preload" 
          href="/fonts/inter-var.woff2" 
          as="font" 
          type="font/woff2"
          crossOrigin="anonymous"
        />
        
        {/* Prefetch: carica risorse per navigazione futura */}
        <link rel="prefetch" href="/about" />
        
        {/* Modulepreload: preload ES modules */}
        <link rel="modulepreload" href="/_next/static/chunks/main.js" />
      </head>
      <body>{children}</body>
    </html>
  );
}

§ 2.6 FETCH PRIORITY API

typescript
// Priorità di fetch per risorse critiche

// HTML attribute
<img src="/hero.jpg" fetchpriority="high" />
<img src="/below-fold.jpg" fetchpriority="low" />

// Next.js Image con priority
<Image
  src="/hero.jpg"
  priority // Aggiunge fetchpriority="high"
  alt="Hero"
/>

// Script priority
<script src="/critical.js" fetchpriority="high"></script>
<script src="/analytics.js" fetchpriority="low"></script>

// Fetch API
fetch('/api/critical-data', { priority: 'high' });
fetch('/api/analytics', { priority: 'low' });

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 3: INP - INTERACTION TO NEXT PAINT
# ═══════════════════════════════════════════════════════════════════════════════

INP_OPTIMIZATION = """

§ 3.1 COS'È INP

┌─────────────────────────────────────────────────────────────────────────────┐
│ INP - INTERACTION TO NEXT PAINT                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ DEFINIZIONE                                                                 │
│ Misura la reattività della pagina durante TUTTA la sessione utente,        │
│ tracciando il delay più lungo tra input utente e update visuale.           │
│                                                                             │
│ INTERAZIONI MISURATE                                                        │
│ ├── Click (mouse, trackpad, touchscreen)                                   │
│ ├── Tap (touchscreen)                                                      │
│ ├── Key press (physical/virtual keyboard)                                  │
│ └── Context menu (right-click)                                             │
│                                                                             │
│ COMPONENTI DI INP                                                           │
│ ├── Input Delay: tempo da evento a handler execution                       │
│ ├── Processing Time: tempo di esecuzione handler                           │
│ └── Presentation Delay: tempo da handler a next paint                      │
│                                                                             │
│ THRESHOLDS                                                                  │
│ ├── GOOD:    ≤ 200ms                                                       │
│ ├── NEEDS:   200 - 500ms                                                   │
│ └── POOR:    > 500ms                                                       │
│                                                                             │
│ vs FID (deprecated Marzo 2024)                                             │
│ ├── FID: solo prima interazione                                            │
│ └── INP: tutte le interazioni, riporta la peggiore                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 3.2 OTTIMIZZARE INPUT DELAY

typescript
// Il main thread deve essere libero per rispondere agli input
// Problema: Long Tasks (>50ms) bloccano gli input

// 1. Code Splitting - Caricare solo il codice necessario
// Next.js dynamic imports
import dynamic from 'next/dynamic';

// Carica componente solo quando necessario
const HeavyChart = dynamic(() => import('./HeavyChart'), {
  loading: () => <ChartSkeleton />,
  ssr: false, // Solo client-side
});

// Carica su interazione
const Modal = dynamic(() => import('./Modal'));

function ProductPage() {
  const [showModal, setShowModal] = useState(false);
  
  return (
    <>
      <button onClick={() => setShowModal(true)}>
        Open Details
      </button>
      {showModal && <Modal />}
    </>
  );
}

// 2. Lazy loading di librerie pesanti
async function handleExport() {
  // Carica xlsx solo quando serve
  const XLSX = await import('xlsx');
  const workbook = XLSX.utils.book_new();
  // ...
}

§ 3.3 OTTIMIZZARE PROCESSING TIME

typescript
// 1. Debounce e Throttle per eventi frequenti
import { useCallback, useMemo } from 'react';
import debounce from 'lodash/debounce';
import throttle from 'lodash/throttle';

function SearchInput() {
  const [query, setQuery] = useState('');
  
  // Debounce: aspetta 300ms dopo l'ultimo input
  const debouncedSearch = useMemo(
    () => debounce((q: string) => {
      fetchSearchResults(q);
    }, 300),
    []
  );
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    setQuery(value); // UI update immediato
    debouncedSearch(value); // API call debounced
  };
  
  return <input value={query} onChange={handleChange} />;
}

// Throttle per scroll/resize
function useScrollPosition() {
  const [scrollY, setScrollY] = useState(0);
  
  useEffect(() => {
    const handleScroll = throttle(() => {
      setScrollY(window.scrollY);
    }, 100); // Max 10 updates/second
    
    window.addEventListener('scroll', handleScroll, { passive: true });
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);
  
  return scrollY;
}

typescript
// 2. Yield to Main Thread - Permetti al browser di rispondere
// Usa scheduler.yield() o setTimeout per spezzare long tasks

// Modern approach con scheduler.yield
async function processLargeArray(items: Item[]) {
  for (let i = 0; i < items.length; i++) {
    processItem(items[i]);
    
    // Yield ogni 5 items per permettere input handling
    if (i % 5 === 0) {
      await scheduler.yield?.() ?? new Promise(r => setTimeout(r, 0));
    }
  }
}

// Alternative: requestIdleCallback
function processInBackground(items: Item[]) {
  let index = 0;
  
  function processChunk(deadline: IdleDeadline) {
    while (index < items.length && deadline.timeRemaining() > 0) {
      processItem(items[index]);
      index++;
    }
    
    if (index < items.length) {
      requestIdleCallback(processChunk);
    }
  }
  
  requestIdleCallback(processChunk);
}

// 3. Web Workers per heavy computation
// workers/dataProcessor.worker.ts
self.onmessage = (event) => {
  const { data } = event;
  const result = heavyComputation(data);
  self.postMessage(result);
};

// Component
function DataProcessor() {
  const workerRef = useRef<Worker>();
  
  useEffect(() => {
    workerRef.current = new Worker(
      new URL('../workers/dataProcessor.worker.ts', import.meta.url)
    );
    
    workerRef.current.onmessage = (event) => {
      setResults(event.data);
    };
    
    return () => workerRef.current?.terminate();
  }, []);
  
  const processData = (data: any) => {
    workerRef.current?.postMessage(data);
  };
}

§ 3.4 OTTIMIZZARE REACT RENDERING

typescript
// 1. useMemo per calcoli costosi
function ProductList({ products, filters }) {
  // Memoizza calcoli pesanti
  const filteredProducts = useMemo(() => {
    return products
      .filter(p => matchesFilters(p, filters))
      .sort((a, b) => a.price - b.price);
  }, [products, filters]);
  
  return <Grid items={filteredProducts} />;
}

// 2. useCallback per event handlers stabili
function ProductCard({ product, onAddToCart }) {
  // Handler stabile, non causa re-render figli
  const handleClick = useCallback(() => {
    onAddToCart(product.id);
  }, [product.id, onAddToCart]);
  
  return (
    <Card>
      <AddButton onClick={handleClick} />
    </Card>
  );
}

// 3. React.memo per componenti pure
const ProductCard = memo(function ProductCard({ product }) {
  return (
    <div className="card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>{product.price}</p>
    </div>
  );
});

// 4. Virtualization per liste lunghe
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null);
  
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50, // Altezza stimata item
    overscan: 5, // Items extra renderizzati
  });
  
  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: virtualItem.start,
              height: virtualItem.size,
            }}
          >
            <ItemRow item={items[virtualItem.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}

§ 3.5 OTTIMIZZARE EVENT HANDLERS

typescript
// 1. Passive event listeners per scroll/touch
useEffect(() => {
  const handleScroll = () => {
    // Non chiama preventDefault(), quindi può essere passive
    updateScrollPosition();
  };
  
  window.addEventListener('scroll', handleScroll, { passive: true });
  return () => window.removeEventListener('scroll', handleScroll);
}, []);

// 2. Event delegation
function List({ items }) {
  // Un solo handler per tutta la lista
  const handleClick = (e: React.MouseEvent) => {
    const target = e.target as HTMLElement;
    const itemId = target.closest('[data-item-id]')?.getAttribute('data-item-id');
    
    if (itemId) {
      handleItemClick(itemId);
    }
  };
  
  return (
    <ul onClick={handleClick}>
      {items.map(item => (
        <li key={item.id} data-item-id={item.id}>
          {item.name}
        </li>
      ))}
    </ul>
  );
}

// 3. Avoid layout thrashing
function updateElements(elements: HTMLElement[]) {
  // ❌ BAD: read-write-read-write causes layout thrashing
  elements.forEach(el => {
    const height = el.offsetHeight; // READ (forces layout)
    el.style.height = height * 2 + 'px'; // WRITE
  });
  
  // ✅ GOOD: batch reads, then batch writes
  const heights = elements.map(el => el.offsetHeight); // All reads
  elements.forEach((el, i) => {
    el.style.height = heights[i] * 2 + 'px'; // All writes
  });
}

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 4: CLS - CUMULATIVE LAYOUT SHIFT
# ═══════════════════════════════════════════════════════════════════════════════

CLS_OPTIMIZATION = """

§ 4.1 COS'È CLS

┌─────────────────────────────────────────────────────────────────────────────┐
│ CLS - CUMULATIVE LAYOUT SHIFT                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ DEFINIZIONE                                                                 │
│ Misura la stabilità visiva quantificando quanto il contenuto si sposta     │
│ inaspettatamente durante il caricamento della pagina.                       │
│                                                                             │
│ FORMULA                                                                     │
│ CLS = Σ (impact fraction × distance fraction)                              │
│                                                                             │
│ ├── Impact Fraction: % viewport interessata dallo shift                    │
│ └── Distance Fraction: distanza dello shift / altezza viewport             │
│                                                                             │
│ THRESHOLDS                                                                  │
│ ├── GOOD:    ≤ 0.1                                                         │
│ ├── NEEDS:   0.1 - 0.25                                                    │
│ └── POOR:    > 0.25                                                        │
│                                                                             │
│ CAUSE COMUNI                                                                │
│ ├── Immagini senza dimensioni                                              │
│ ├── Ads, embeds, iframes senza spazio riservato                            │
│ ├── Contenuto iniettato dinamicamente                                      │
│ ├── Font che causano FOIT/FOUT                                             │
│ └── Animazioni che triggherano layout                                      │
│                                                                             │
│ NOTA: Layout shifts causati da user interaction (click) sono esclusi       │
│       dal calcolo (entro 500ms dall'input)                                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 4.2 DIMENSIONI ESPLICITE PER MEDIA

typescript
// ✅ ALWAYS: Specificare width e height per immagini

// Next.js Image - dimensioni automatiche per local imports
import Image from 'next/image';
import heroImage from '../public/hero.jpg';

function Hero() {
  return (
    <Image
      src={heroImage}
      alt="Hero"
      // width e height inferiti automaticamente
      placeholder="blur" // blurDataURL generato automaticamente
    />
  );
}

// Next.js Image - remote images richiedono dimensioni
function ProductImage({ product }) {
  return (
    <Image
      src={product.imageUrl}
      alt={product.name}
      width={400}
      height={300}
      // oppure fill con container sized
    />
  );
}

// Fill mode - responsive con aspect ratio preservato
function ResponsiveImage() {
  return (
    <div className="relative aspect-video w-full">
      <Image
        src="/banner.jpg"
        alt="Banner"
        fill
        sizes="100vw"
        className="object-cover"
      />
    </div>
  );
}

css
/* CSS per mantenere aspect ratio */

/* Modern approach con aspect-ratio */
.image-container {
  aspect-ratio: 16 / 9;
  width: 100%;
}

.image-container img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* Fallback per browser vecchi */
.image-container-legacy {
  position: relative;
  width: 100%;
  padding-bottom: 56.25%; /* 16:9 ratio */
}

.image-container-legacy img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

§ 4.3 RISERVARE SPAZIO PER CONTENUTO DINAMICO

typescript
// Skeleton loaders per contenuto in caricamento
function ProductCardSkeleton() {
  return (
    <div className="animate-pulse">
      {/* Immagine placeholder con dimensioni fisse */}
      <div className="bg-gray-200 aspect-square w-full rounded-lg" />
      
      {/* Title placeholder */}
      <div className="mt-4 h-4 bg-gray-200 rounded w-3/4" />
      
      {/* Price placeholder */}
      <div className="mt-2 h-4 bg-gray-200 rounded w-1/4" />
    </div>
  );
}

// Container con min-height per evitare collapse
function DynamicContent() {
  const [content, setContent] = useState(null);
  
  return (
    <div 
      className="min-h-[200px]" // Riserva spazio minimo
      style={{ contain: 'layout' }} // CSS containment
    >
      {content ? (
        <ActualContent data={content} />
      ) : (
        <ContentSkeleton />
      )}
    </div>
  );
}

typescript
// Ads e embeds - sempre riservare spazio
function AdSlot({ width, height }) {
  return (
    <div 
      className="ad-container"
      style={{ 
        width: `${width}px`, 
        height: `${height}px`,
        minHeight: `${height}px`, // Prevent collapse
        contain: 'strict', // Isola dal resto della pagina
      }}
    >
      {/* Ad caricato async */}
      <AdComponent />
    </div>
  );
}

// YouTube embed con aspect ratio
function YouTubeEmbed({ videoId }) {
  return (
    <div className="relative aspect-video w-full">
      <iframe
        src={`https://www.youtube.com/embed/${videoId}`}
        className="absolute inset-0 w-full h-full"
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
        allowFullScreen
      />
    </div>
  );
}

§ 4.4 FONT LOADING OPTIMIZATION

typescript
// next.config.js - Font optimization automatica
/** @type {import('next').NextConfig} */
const nextConfig = {
  optimizeFonts: true, // Default true
};

// app/layout.tsx - next/font (recommended)
import { Inter, Roboto_Mono } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap', // Mostra fallback, poi swap
  variable: '--font-inter',
});

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-roboto-mono',
});

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={`${inter.variable} ${robotoMono.variable}`}>
      <body className={inter.className}>{children}</body>
    </html>
  );
}

css
/* Font fallback system per ridurre layout shift */

/* Definire font-family con fallback che matcha le metriche */
:root {
  --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', 
               Roboto, 'Helvetica Neue', Arial, sans-serif;
}

/* Usare size-adjust per matching metriche */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-var.woff2') format('woff2');
  font-weight: 100 900;
  font-display: swap;
  /* Opzionale: aggiustare metriche per match fallback */
  size-adjust: 100%;
  ascent-override: 90%;
  descent-override: 20%;
  line-gap-override: 0%;
}

/* Font display values:
   - swap: mostra fallback, poi swap (può causare FOUT)
   - optional: mostra fallback, swap solo se font carica veloce
   - fallback: come optional ma con timeout più breve
   - block: nasconde testo brevemente (FOIT), poi mostra
*/

§ 4.5 EVITARE CONTENUTO INIETTATO ABOVE THE FOLD

typescript
// ❌ BAD: Banner iniettato che spinge giù il contenuto
function Page() {
  const [showBanner, setShowBanner] = useState(false);
  
  useEffect(() => {
    checkIfUserQualifiesForBanner().then(setShowBanner);
  }, []);
  
  return (
    <>
      {showBanner && <PromoBanner />} {/* Causa CLS! */}
      <Header />
      <MainContent />
    </>
  );
}

// ✅ GOOD: Riservare spazio per banner potenziale
function Page() {
  const [bannerState, setBannerState] = useState<'loading' | 'show' | 'hide'>('loading');
  
  useEffect(() => {
    checkIfUserQualifiesForBanner().then(show => 
      setBannerState(show ? 'show' : 'hide')
    );
  }, []);
  
  return (
    <>
      {/* Spazio sempre riservato, contenuto condizionale */}
      <div className={bannerState === 'hide' ? 'hidden' : 'min-h-[60px]'}>
        {bannerState === 'show' && <PromoBanner />}
      </div>
      <Header />
      <MainContent />
    </>
  );
}

// ✅ BETTER: Banner in posizione non-intrusive
function Page() {
  return (
    <>
      <Header />
      <MainContent />
      {/* Banner in fondo o sticky non causa CLS */}
      <StickyBottomBanner />
    </>
  );
}

§ 4.6 CSS CONTAINMENT

css
/* CSS Containment per isolare layout shifts */

/* contain: layout - isola calcoli layout */
.dynamic-section {
  contain: layout;
}

/* contain: paint - isola rendering */
.heavy-animation {
  contain: paint;
}

/* contain: strict - massima isolazione */
.third-party-widget {
  contain: strict;
  width: 300px;
  height: 250px;
}

/* content-visibility per lazy rendering */
.below-fold-section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px; /* Stima altezza */
}

/* Esempio pratico: lista lunga */
.product-list-item {
  contain: content;
  content-visibility: auto;
  contain-intrinsic-size: 0 120px;
}

§ 4.7 ANIMAZIONI PERFORMANTI

css
/* ❌ Animazioni che triggherano layout */
.bad-animation {
  animation: expand 0.3s ease;
}

@keyframes expand {
  from { height: 0; margin-top: 0; }
  to { height: 100px; margin-top: 20px; }
}

/* ✅ Animazioni che usano solo transform e opacity */
.good-animation {
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { 
    opacity: 0; 
    transform: translateY(-10px); 
  }
  to { 
    opacity: 1; 
    transform: translateY(0); 
  }
}

/* Properties che NON triggherano layout:
   - transform
   - opacity
   - filter
   
   Properties che TRIGGHERANO layout (evitare in animazioni):
   - width, height
   - margin, padding
   - top, left, right, bottom
   - font-size
   - border-width
*/

/* Usare will-change con cautela */
.animated-element {
  will-change: transform, opacity;
}

/* Rimuovere will-change dopo animazione */
.animated-element.animation-done {
  will-change: auto;
}

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 5: IMAGE OPTIMIZATION
# ═══════════════════════════════════════════════════════════════════════════════

IMAGE_OPTIMIZATION = """

§ 5.1 FORMATI IMMAGINE MODERNI

┌─────────────────────────────────────────────────────────────────────────────┐
│ FORMATI IMMAGINE - CONFRONTO                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ FORMATO    │ COMPRESSIONE │ TRASPARENZA │ ANIMAZIONE │ SUPPORT BROWSER     │
│ ───────────┼──────────────┼─────────────┼────────────┼──────────────────── │
│ JPEG       │ Lossy        │ No          │ No         │ 100%                │
│ PNG        │ Lossless     │ Si          │ No         │ 100%                │
│ GIF        │ Lossless     │ Si          │ Si         │ 100%                │
│ WebP       │ Both         │ Si          │ Si         │ 97%+                │
│ AVIF       │ Both         │ Si          │ Si         │ 92%+                │
│ ───────────┴──────────────┴─────────────┴────────────┴──────────────────── │
│                                                                             │
│ RACCOMANDAZIONI:                                                            │
│ ├── Foto: AVIF > WebP > JPEG                                               │
│ ├── Graphics/Icons: SVG (vector) o WebP                                    │
│ ├── Screenshots: WebP o PNG                                                │
│ └── Animazioni: WebP/AVIF > MP4 video > GIF                               │
│                                                                             │
│ RISPARMIO TIPICO vs JPEG:                                                   │
│ ├── WebP: 25-35% più piccolo                                               │
│ └── AVIF: 50% più piccolo                                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 5.2 NEXT.JS IMAGE COMPONENT

typescript
// next.config.js - Configurazione immagini
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    // Domini remoti autorizzati
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'cdn.example.com',
        pathname: '/images/**',
      },
      {
        protocol: 'https',
        hostname: '*.cloudinary.com',
      },
    ],
    
    // Formati output (default: webp)
    formats: ['image/avif', 'image/webp'],
    
    // Dimensioni generate
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    
    // Loader custom (es. Cloudinary)
    // loader: 'custom',
    // loaderFile: './lib/image-loader.ts',
  },
};

module.exports = nextConfig;

typescript
// components/OptimizedImage.tsx
import Image from 'next/image';

interface OptimizedImageProps {
  src: string;
  alt: string;
  priority?: boolean;
  className?: string;
}

// Hero image - above the fold
export function HeroImage({ src, alt }: OptimizedImageProps) {
  return (
    <Image
      src={src}
      alt={alt}
      width={1200}
      height={600}
      priority // Preload, no lazy loading
      quality={85} // Balance quality/size
      sizes="100vw"
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDA..."
    />
  );
}

// Product image - lazy loaded
export function ProductImage({ product }) {
  return (
    <Image
      src={product.image}
      alt={product.name}
      width={400}
      height={400}
      sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
      quality={75}
      className="object-cover rounded-lg"
      // loading="lazy" è default
    />
  );
}

// Responsive fill image
export function BackgroundImage({ src, alt }) {
  return (
    <div className="relative w-full h-[400px]">
      <Image
        src={src}
        alt={alt}
        fill
        sizes="100vw"
        className="object-cover"
        quality={80}
      />
    </div>
  );
}

// Art direction - different images per breakpoint
export function ResponsiveHero() {
  return (
    <picture>
      <source
        media="(min-width: 1024px)"
        srcSet="/hero-desktop.webp"
        type="image/webp"
      />
      <source
        media="(min-width: 768px)"
        srcSet="/hero-tablet.webp"
        type="image/webp"
      />
      <Image
        src="/hero-mobile.webp"
        alt="Hero"
        width={640}
        height={400}
        priority
      />
    </picture>
  );
}

§ 5.3 SIZES ATTRIBUTE BEST PRACTICES

typescript
// sizes indica al browser quale larghezza avrà l'immagine
// prima che CSS sia caricato

// Full width
sizes="100vw"

// Responsive grid
sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 33vw"

// Sidebar image
sizes="(max-width: 768px) 100vw, 300px"

// Fixed width sempre
sizes="400px"

// Esempio pratico con layout
function ProductGrid({ products }) {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      {products.map((product) => (
        <div key={product.id}>
          <Image
            src={product.image}
            alt={product.name}
            width={400}
            height={400}
            // Matches grid: 1col mobile, 2col tablet, 3col desktop
            sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 33vw"
          />
        </div>
      ))}
    </div>
  );
}

§ 5.4 LAZY LOADING E PLACEHOLDER

typescript
// Blur placeholder con blurDataURL
import Image from 'next/image';

// Generare blurDataURL automaticamente per immagini locali
import heroImg from '../public/hero.jpg';

function Hero() {
  return (
    <Image
      src={heroImg}
      alt="Hero"
      placeholder="blur" // blurDataURL generato automaticamente
    />
  );
}

// Generare blurDataURL per immagini remote
// lib/blur-placeholder.ts
import { getPlaiceholder } from 'plaiceholder';

export async function getBlurDataURL(imageUrl: string): Promise<string> {
  try {
    const response = await fetch(imageUrl);
    const buffer = Buffer.from(await response.arrayBuffer());
    const { base64 } = await getPlaiceholder(buffer);
    return base64;
  } catch {
    // Fallback placeholder
    return 'data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAA...';
  }
}

// Usage in page
async function ProductPage({ params }) {
  const product = await getProduct(params.id);
  const blurDataURL = await getBlurDataURL(product.image);
  
  return (
    <Image
      src={product.image}
      alt={product.name}
      width={600}
      height={600}
      placeholder="blur"
      blurDataURL={blurDataURL}
    />
  );
}

§ 5.5 IMAGE CDN E OPTIMIZATION SERVICES

typescript
// Custom loader per Cloudinary
// lib/cloudinary-loader.ts
export default function cloudinaryLoader({ src, width, quality }) {
  const params = [
    'f_auto', // Auto format (webp/avif)
    'c_limit', // Fit within dimensions
    `w_${width}`,
    `q_${quality || 'auto'}`,
  ];
  
  return `https://res.cloudinary.com/your-cloud/image/upload/${params.join(',')}${src}`;
}

// next.config.js
module.exports = {
  images: {
    loader: 'custom',
    loaderFile: './lib/cloudinary-loader.ts',
  },
};

// Custom loader per imgix
export function imgixLoader({ src, width, quality }) {
  const url = new URL(`https://your-domain.imgix.net${src}`);
  url.searchParams.set('w', width.toString());
  url.searchParams.set('q', (quality || 75).toString());
  url.searchParams.set('auto', 'format,compress');
  return url.toString();
}

// Self-hosted con sharp
// Esempio API route per image optimization
// app/api/image/route.ts
import sharp from 'sharp';
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const src = searchParams.get('src');
  const width = parseInt(searchParams.get('w') || '800');
  const quality = parseInt(searchParams.get('q') || '80');
  
  const response = await fetch(src!);
  const buffer = await response.arrayBuffer();
  
  const optimized = await sharp(Buffer.from(buffer))
    .resize(width)
    .webp({ quality })
    .toBuffer();
  
  return new NextResponse(optimized, {
    headers: {
      'Content-Type': 'image/webp',
      'Cache-Control': 'public, max-age=31536000, immutable',
    },
  });
}

§ 5.6 RESPONSIVE IMAGES PATTERN

typescript
// components/ResponsiveImage.tsx
import Image from 'next/image';

interface ImageVariant {
  src: string;
  media?: string;
  type?: string;
}

interface ResponsiveImageProps {
  src: string;
  alt: string;
  width: number;
  height: number;
  variants?: ImageVariant[];
  priority?: boolean;
}

export function ResponsiveImage({
  src,
  alt,
  width,
  height,
  variants,
  priority = false,
}: ResponsiveImageProps) {
  if (variants && variants.length > 0) {
    return (
      <picture>
        {variants.map((variant, index) => (
          <source
            key={index}
            srcSet={variant.src}
            media={variant.media}
            type={variant.type}
          />
        ))}
        <Image
          src={src}
          alt={alt}
          width={width}
          height={height}
          priority={priority}
        />
      </picture>
    );
  }
  
  return (
    <Image
      src={src}
      alt={alt}
      width={width}
      height={height}
      priority={priority}
      sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 33vw"
    />
  );
}

// Usage
<ResponsiveImage
  src="/fallback.jpg"
  alt="Product"
  width={800}
  height={600}
  variants={[
    { src: "/product-desktop.avif", media: "(min-width: 1024px)", type: "image/avif" },
    { src: "/product-desktop.webp", media: "(min-width: 1024px)", type: "image/webp" },
    { src: "/product-mobile.avif", media: "(max-width: 1023px)", type: "image/avif" },
    { src: "/product-mobile.webp", media: "(max-width: 1023px)", type: "image/webp" },
  ]}
/>

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 6: JAVASCRIPT OPTIMIZATION
# ═══════════════════════════════════════════════════════════════════════════════

JAVASCRIPT_OPTIMIZATION = """

§ 6.1 CODE SPLITTING

typescript
// Next.js automatic code splitting per route
// Ogni page è un chunk separato automaticamente

// Dynamic imports per component-level splitting
import dynamic from 'next/dynamic';

// Lazy load componente pesante
const HeavyChart = dynamic(() => import('@/components/HeavyChart'), {
  loading: () => <ChartSkeleton />,
  ssr: false, // Solo client-side se necessario
});

// Lazy load solo quando visibile
const LazySection = dynamic(() => import('@/components/LazySection'), {
  loading: () => <SectionSkeleton />,
});

function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  
  return (
    <div>
      <Header />
      <QuickStats />
      
      {/* Carica solo quando user clicca */}
      <button onClick={() => setShowChart(true)}>Show Charts</button>
      {showChart && <HeavyChart />}
      
      {/* Intersection Observer per lazy loading */}
      <LazyLoadWrapper>
        <LazySection />
      </LazyLoadWrapper>
    </div>
  );
}

// React.lazy alternativo
import { lazy, Suspense } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <LazyComponent />
    </Suspense>
  );
}

§ 6.2 TREE SHAKING

typescript
// ✅ GOOD: Named imports permettono tree shaking
import { debounce } from 'lodash-es';
import { format } from 'date-fns';

// ❌ BAD: Import intera libreria
import _ from 'lodash';
import moment from 'moment';

// ✅ Usare path imports per lodash se necessario
import debounce from 'lodash/debounce';
import throttle from 'lodash/throttle';

// Verificare che package.json abbia sideEffects corretto
// package.json del tuo progetto
{
  "sideEffects": [
    "*.css",
    "*.scss"
  ]
}

// Per librerie che non supportano tree shaking, usare alternative:
// moment → date-fns (treeshakable)
// lodash → lodash-es o radash
// axios → ky o native fetch

// next.config.js - Ottimizzazioni bundle
const nextConfig = {
  // Modularize imports automatico
  modularizeImports: {
    'lodash': {
      transform: 'lodash/{{member}}',
    },
    '@mui/icons-material': {
      transform: '@mui/icons-material/{{member}}',
    },
    '@heroicons/react/24/outline': {
      transform: '@heroicons/react/24/outline/{{member}}',
    },
  },
};

§ 6.3 BUNDLE ANALYSIS

bash
# Installare bundle analyzer
pnpm add -D @next/bundle-analyzer

# next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // altre config
});

# Eseguire analisi
ANALYZE=true pnpm build

typescript
// Visualizzare cosa è nel bundle
// Cercare:
// 1. Librerie duplicate
// 2. Polyfills non necessari
// 3. Codice non usato
// 4. Librerie troppo grandi

// Esempio output analisi:
// ├── pages/_app.js (150 KB)
// │   ├── react (42 KB)
// │   ├── react-dom (130 KB) ← shared
// │   ├── lodash (71 KB) ← PROBLEMA: importata tutta
// │   └── moment (231 KB) ← PROBLEMA: molto grande

// Soluzioni:
// 1. Sostituire lodash con lodash-es + named imports
// 2. Sostituire moment con date-fns
// 3. Dynamic import per componenti pesanti

§ 6.4 MINIFICATION E COMPRESSION

typescript
// next.config.js - SWC minifier (default in Next.js 12+)
const nextConfig = {
  swcMinify: true, // Default true
  
  // Compress output
  compress: true, // Default true for production
};

// Server-side compression con gzip/brotli
// vercel.json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "Content-Encoding",
          "value": "gzip"
        }
      ]
    }
  ]
}

// Express custom server
import compression from 'compression';

app.use(compression({
  level: 6, // Compression level (0-9)
  threshold: 1024, // Solo per response > 1KB
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  },
}));

§ 6.5 SCRIPT LOADING STRATEGIES

typescript
// Next.js Script component
import Script from 'next/script';

export default function Layout({ children }) {
  return (
    <>
      {/* Prima dell'idratazione - blocca rendering */}
      <Script
        src="/critical-polyfill.js"
        strategy="beforeInteractive"
      />
      
      {children}
      
      {/* Dopo che pagina è interattiva */}
      <Script
        src="https://www.googletagmanager.com/gtag/js?id=GA_ID"
        strategy="afterInteractive"
      />
      
      {/* Durante idle del browser */}
      <Script
        src="https://connect.facebook.net/en_US/sdk.js"
        strategy="lazyOnload"
      />
      
      {/* In web worker (experimental) */}
      <Script
        src="/heavy-analytics.js"
        strategy="worker"
      />
      
      {/* Inline script */}
      <Script id="structured-data" type="application/ld+json">
        {JSON.stringify(structuredData)}
      </Script>
    </>
  );
}

// Callbacks per tracking
<Script
  src="https://example.com/script.js"
  onLoad={() => console.log('Script loaded')}
  onReady={() => console.log('Script ready, can use globals')}
  onError={(e) => console.error('Script failed', e)}
/>

§ 6.6 THIRD-PARTY SCRIPT OPTIMIZATION

typescript
// Facade pattern per third-party pesanti
// components/YouTubeFacade.tsx
import { useState } from 'react';
import Image from 'next/image';

interface YouTubeFacadeProps {
  videoId: string;
  title: string;
}

export function YouTubeFacade({ videoId, title }: YouTubeFacadeProps) {
  const [isLoaded, setIsLoaded] = useState(false);
  
  // Mostra thumbnail finché user non clicca
  if (!isLoaded) {
    return (
      <button
        onClick={() => setIsLoaded(true)}
        className="relative aspect-video w-full group"
        aria-label={`Play video: ${title}`}
      >
        <Image
          src={`https://img.youtube.com/vi/${videoId}/maxresdefault.jpg`}
          alt={title}
          fill
          className="object-cover"
        />
        {/* Play button overlay */}
        <div className="absolute inset-0 flex items-center justify-center">
          <div className="w-16 h-16 bg-red-600 rounded-full flex items-center justify-center group-hover:bg-red-700">
            <svg className="w-8 h-8 text-white" viewBox="0 0 24 24">
              <path fill="currentColor" d="M8 5v14l11-7z" />
            </svg>
          </div>
        </div>
      </button>
    );
  }
  
  // Carica iframe solo dopo click
  return (
    <iframe
      src={`https://www.youtube.com/embed/${videoId}?autoplay=1`}
      title={title}
      className="aspect-video w-full"
      allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
      allowFullScreen
    />
  );
}

// Partytown per spostare scripts in web worker
// next.config.js
const nextConfig = {
  experimental: {
    nextScriptWorkers: true,
  },
};

// pages/_document.tsx
import { Html, Head, Main, NextScript } from 'next/document';

export default function Document() {
  return (
    <Html>
      <Head>
        {/* Partytown si occupa di spostare scripts in worker */}
      </Head>
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  );
}

// Usage con strategy="worker"
<Script
  src="https://www.googletagmanager.com/gtag/js?id=GA_ID"
  strategy="worker"
/>

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 7: CSS OPTIMIZATION
# ═══════════════════════════════════════════════════════════════════════════════

CSS_OPTIMIZATION = """

§ 7.1 CRITICAL CSS

typescript
// Next.js App Router gestisce CSS automaticamente
// - CSS Modules sono automaticamente code-split
// - CSS viene estratto in file separati in production

// Per critical CSS manuale, usare inline styles
// app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html>
      <head>
        {/* Critical CSS inline per above-the-fold */}
        <style dangerouslySetInnerHTML={{ __html: `
          :root { --primary: #0070f3; }
          body { margin: 0; font-family: system-ui; }
          .header { height: 60px; background: white; }
          .hero { min-height: 400px; }
        `}} />
      </head>
      <body>{children}</body>
    </html>
  );
}

// Tool: critters per estrazione automatica critical CSS
// next.config.js
const nextConfig = {
  experimental: {
    optimizeCss: true, // Usa critters
  },
};

§ 7.2 CSS MODULES

typescript
// Automatic code splitting per page/component

// components/Button/Button.module.css
.button {
  padding: 0.5rem 1rem;
  border-radius: 4px;
  font-weight: 500;
}

.primary {
  background: var(--primary);
  color: white;
}

.secondary {
  background: transparent;
  border: 1px solid var(--primary);
  color: var(--primary);
}

// components/Button/Button.tsx
import styles from './Button.module.css';
import { clsx } from 'clsx';

interface ButtonProps {
  variant?: 'primary' | 'secondary';
  children: React.ReactNode;
}

export function Button({ variant = 'primary', children }: ButtonProps) {
  return (
    <button className={clsx(styles.button, styles[variant])}>
      {children}
    </button>
  );
}

§ 7.3 TAILWIND CSS OPTIMIZATION

javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './app/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [],
  // Rimuovi classi non usate in production (default)
  // Mode JIT è default in Tailwind v3+
};

// postcss.config.js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
    // Minificazione CSS in production
    ...(process.env.NODE_ENV === 'production' ? { cssnano: {} } : {}),
  },
};

§ 7.4 ELIMINARE CSS NON UTILIZZATO

bash
# PurgeCSS per rimuovere CSS non usato
pnpm add -D @fullhuman/postcss-purgecss

# postcss.config.js
const purgecss = require('@fullhuman/postcss-purgecss');

module.exports = {
  plugins: [
    require('tailwindcss'),
    require('autoprefixer'),
    ...(process.env.NODE_ENV === 'production'
      ? [
          purgecss({
            content: [
              './app/**/*.{js,ts,jsx,tsx}',
              './components/**/*.{js,ts,jsx,tsx}',
            ],
            defaultExtractor: (content) =>
              content.match(/[\w-/:]+(?<!:)/g) || [],
            safelist: {
              standard: ['html', 'body'],
              // Classi generate dinamicamente
              greedy: [/^data-/, /^aria-/],
            },
          }),
        ]
      : []),
  ],
};

§ 7.5 CSS PERFORMANCE PATTERNS

css
/* Evitare selettori costosi */

/* ❌ BAD: Selettori universali e discendenti profondi */
* { box-sizing: border-box; }
.container div span a { color: blue; }
[class*="icon-"] { display: inline-block; }

/* ✅ GOOD: Selettori semplici e specifici */
*, *::before, *::after { box-sizing: border-box; } /* Meglio specifico */
.nav-link { color: blue; }
.icon { display: inline-block; }

/* Usare CSS Layers per gestire specificità */
@layer base, components, utilities;

@layer base {
  h1 { font-size: 2rem; }
}

@layer components {
  .card { padding: 1rem; }
}

@layer utilities {
  .mt-4 { margin-top: 1rem; }
}

/* Evitare !important */
/* Se necessario, usare cascade layers invece */

/* CSS containment per isolamento */
.widget {
  contain: layout style paint;
}

/* content-visibility per rendering lazy */
.article-section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px;
}

§ 7.6 FONT DISPLAY E LOADING

css
/* Font display strategies */
@font-face {
  font-family: 'CustomFont';
  src: url('/fonts/custom.woff2') format('woff2');
  font-weight: 400;
  font-style: normal;
  font-display: swap; /* FOUT: mostra fallback, poi swap */
  /* font-display: optional; */ /* Meglio per performance */
}

/* Preload font critico */
/* In <head>: */
/* <link rel="preload" href="/fonts/custom.woff2" as="font" type="font/woff2" crossorigin /> */

/* Subset font per ridurre dimensione */
/* Usare tool come glyphhanger per creare subset */
@font-face {
  font-family: 'CustomFont';
  src: url('/fonts/custom-latin.woff2') format('woff2');
  unicode-range: U+0000-00FF; /* Latin base */
}

/* Variable fonts per flessibilità */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-var.woff2') format('woff2');
  font-weight: 100 900; /* Range supportato */
  font-style: normal;
  font-display: swap;
}

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 8: FONT OPTIMIZATION
# ═══════════════════════════════════════════════════════════════════════════════

FONT_OPTIMIZATION = """

§ 8.1 NEXT.JS FONT OPTIMIZATION

typescript
// app/layout.tsx - next/font (Recommended)
import { Inter, Roboto_Mono } from 'next/font/google';

// Google Fonts con ottimizzazione automatica
const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
  // Preload solo weights usati
  weight: ['400', '500', '600', '700'],
});

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-mono',
  weight: ['400', '700'],
});

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={`${inter.variable} ${robotoMono.variable}`}>
      <body className={inter.className}>
        {children}
      </body>
    </html>
  );
}

// Local fonts
import localFont from 'next/font/local';

const customFont = localFont({
  src: [
    {
      path: './fonts/CustomFont-Regular.woff2',
      weight: '400',
      style: 'normal',
    },
    {
      path: './fonts/CustomFont-Bold.woff2',
      weight: '700',
      style: 'normal',
    },
  ],
  display: 'swap',
  variable: '--font-custom',
});

§ 8.2 FONT LOADING STRATEGIES

┌─────────────────────────────────────────────────────────────────────────────┐
│ FONT-DISPLAY VALUES                                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ VALUE        │ BLOCK PERIOD │ SWAP PERIOD │ USE CASE                       │
│ ─────────────┼──────────────┼─────────────┼─────────────────────────────── │
│ auto         │ ~3s          │ Infinite    │ Browser default                │
│ block        │ Short (3s)   │ Infinite    │ Icons, critical branding       │
│ swap         │ 0            │ Infinite    │ Body text (FOUT ok)            │
│ fallback     │ ~100ms       │ ~3s         │ Balance FOIT/FOUT              │
│ optional     │ ~100ms       │ None        │ Best performance, may not load │
│ ─────────────┴──────────────┴─────────────┴─────────────────────────────── │
│                                                                             │
│ RACCOMANDAZIONI:                                                            │
│ ├── Body text: 'swap' (user può leggere subito)                            │
│ ├── Headings: 'swap' o 'fallback'                                          │
│ ├── Icons: 'block' (evita mostrare caratteri sbagliati)                    │
│ └── Non-critical: 'optional' (massima performance)                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

typescript
// Preload critical fonts
// app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html>
      <head>
        {/* Preload font critico */}
        <link
          rel="preload"
          href="/fonts/inter-var.woff2"
          as="font"
          type="font/woff2"
          crossOrigin="anonymous"
        />
        
        {/* Preconnect per Google Fonts (se usate direttamente) */}
        <link rel="preconnect" href="https://fonts.googleapis.com" />
        <link rel="preconnect" href="https://fonts.gstatic.com" crossOrigin="anonymous" />
      </head>
      <body>{children}</body>
    </html>
  );
}

§ 8.3 FONT SUBSETTING

typescript
// Usare subset per ridurre dimensione font
const inter = Inter({
  subsets: ['latin'], // Solo caratteri latini
  // subsets: ['latin', 'latin-ext', 'cyrillic'], // Più subset se necessari
});

// Subset manuale per font custom
// Usare tool: glyphhanger, fonttools, subfont

// glyphhanger CLI
// npx glyphhanger --whitelist="ABCDEFGHIJKLMNOPQRSTUVWXYZ..." --subset=*.woff2

// Unicode ranges per subset
const fontStyles = `
  @font-face {
    font-family: 'CustomFont';
    src: url('/fonts/custom-latin.woff2') format('woff2');
    unicode-range: U+0000-00FF, U+0131, U+0152-0153;
  }
  
  @font-face {
    font-family: 'CustomFont';
    src: url('/fonts/custom-latin-ext.woff2') format('woff2');
    unicode-range: U+0100-024F, U+0259, U+1E00-1EFF;
  }
`;

§ 8.4 VARIABLE FONTS

typescript
// Variable fonts = 1 file per tutti i weights/styles

// Local variable font
import localFont from 'next/font/local';

const interVariable = localFont({
  src: './fonts/Inter-Variable.woff2',
  weight: '100 900', // Range completo
  style: 'normal',
  display: 'swap',
  variable: '--font-inter',
});

// CSS per variable fonts
const variableFontCSS = `
  @font-face {
    font-family: 'Inter';
    src: url('/fonts/Inter-Variable.woff2') format('woff2');
    font-weight: 100 900;
    font-style: normal;
    font-display: swap;
  }
  
  /* Utilizzo con font-variation-settings */
  .custom-weight {
    font-variation-settings: 'wght' 450;
  }
  
  /* O semplicemente font-weight */
  .medium {
    font-weight: 450;
  }
`;

// Vantaggi variable fonts:
// - 1 file invece di multiple (es. 400, 500, 600, 700)
// - Dimensione totale spesso minore
// - Flessibilità per qualsiasi weight

§ 8.5 SYSTEM FONT STACK

css
/* System font stack per massima performance */
.system-font {
  font-family: 
    -apple-system, 
    BlinkMacSystemFont, 
    'Segoe UI', 
    Roboto, 
    'Helvetica Neue', 
    Arial, 
    'Noto Sans', 
    sans-serif,
    'Apple Color Emoji', 
    'Segoe UI Emoji', 
    'Segoe UI Symbol', 
    'Noto Color Emoji';
}

/* Monospace system stack */
.mono {
  font-family:
    ui-monospace,
    SFMono-Regular,
    Menlo,
    Monaco,
    Consolas,
    'Liberation Mono',
    'Courier New',
    monospace;
}

/* Fallback che matcha custom font metrics */
/* Usare fontaine o capsize per calcolare */
@font-face {
  font-family: 'Inter Fallback';
  src: local('Arial');
  ascent-override: 90.44%;
  descent-override: 22.25%;
  line-gap-override: 0%;
  size-adjust: 107.64%;
}

.text {
  font-family: 'Inter', 'Inter Fallback', sans-serif;
}

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 9: CACHING STRATEGIES
# ═══════════════════════════════════════════════════════════════════════════════

CACHING_STRATEGIES = """

§ 9.1 HTTP CACHING OVERVIEW

┌─────────────────────────────────────────────────────────────────────────────┐
│ HTTP CACHE HEADERS                                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ Cache-Control directives:                                                   │
│                                                                             │
│ ├── max-age=<seconds>                                                      │
│ │   Tempo massimo di cache (in secondi)                                    │
│ │                                                                          │
│ ├── s-maxage=<seconds>                                                     │
│ │   Come max-age ma per cache condivise (CDN, proxy)                       │
│ │                                                                          │
│ ├── public                                                                 │
│ │   Può essere cached da qualsiasi cache                                   │
│ │                                                                          │
│ ├── private                                                                │
│ │   Solo cache browser, non CDN/proxy                                      │
│ │                                                                          │
│ ├── no-cache                                                               │
│ │   Deve rivalidare con server prima di usare cache                        │
│ │                                                                          │
│ ├── no-store                                                               │
│ │   Non salvare in cache (dati sensibili)                                  │
│ │                                                                          │
│ ├── must-revalidate                                                        │
│ │   Se stale, DEVE rivalidare prima di usare                               │
│ │                                                                          │
│ ├── stale-while-revalidate=<seconds>                                       │
│ │   Usa stale mentre ricarica in background                                │
│ │                                                                          │
│ └── immutable                                                              │
│     Non cambia mai, non rivalidare                                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 9.2 NEXT.JS CACHING

typescript
// app/api/products/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  const products = await fetchProducts();
  
  return NextResponse.json(products, {
    headers: {
      // Cache per 60s, stale-while-revalidate per 5 min
      'Cache-Control': 'public, s-maxage=60, stale-while-revalidate=300',
    },
  });
}

// Route segment config per caching
// app/products/page.tsx
export const revalidate = 60; // ISR: rivalidare ogni 60 secondi
export const dynamic = 'force-static'; // Forza static generation

// oppure
export const dynamic = 'force-dynamic'; // Sempre fresh
export const revalidate = 0; // No cache

// fetch con cache options
async function getProduct(id: string) {
  const res = await fetch(`${API_URL}/products/${id}`, {
    // Cache strategy
    cache: 'force-cache', // Default: cache indefinitamente
    // cache: 'no-store', // Mai cache
    
    // Oppure time-based revalidation
    next: { revalidate: 60 }, // Rivalidare ogni 60s
    
    // Tag-based revalidation
    next: { tags: ['products', `product-${id}`] },
  });
  
  return res.json();
}

// Revalidation on-demand
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';

export async function POST(request: Request) {
  const { path, tag, secret } = await request.json();
  
  // Verify secret
  if (secret !== process.env.REVALIDATION_SECRET) {
    return new Response('Invalid secret', { status: 401 });
  }
  
  if (path) {
    revalidatePath(path);
  }
  
  if (tag) {
    revalidateTag(tag);
  }
  
  return Response.json({ revalidated: true });
}

§ 9.3 STATIC ASSETS CACHING

typescript
// next.config.js - Headers per static assets
const nextConfig = {
  async headers() {
    return [
      {
        // Immagini ottimizzate Next.js
        source: '/_next/image/:path*',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=31536000, immutable',
          },
        ],
      },
      {
        // Static files (JS, CSS con content hash)
        source: '/_next/static/:path*',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=31536000, immutable',
          },
        ],
      },
      {
        // Fonts
        source: '/fonts/:path*',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=31536000, immutable',
          },
        ],
      },
      {
        // HTML pages - cache breve
        source: '/:path*',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=0, must-revalidate',
          },
        ],
      },
    ];
  },
};

§ 9.4 SERVICE WORKER CACHING

typescript
// Service Worker per caching avanzato
// public/sw.js
const CACHE_NAME = 'app-cache-v1';
const STATIC_ASSETS = [
  '/',
  '/offline',
  '/manifest.json',
];

// Install: cache static assets
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll(STATIC_ASSETS);
    })
  );
});

// Activate: cleanup old caches
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames
          .filter((name) => name !== CACHE_NAME)
          .map((name) => caches.delete(name))
      );
    })
  );
});

// Fetch: stale-while-revalidate strategy
self.addEventListener('fetch', (event) => {
  // Solo GET requests
  if (event.request.method !== 'GET') return;
  
  event.respondWith(
    caches.open(CACHE_NAME).then(async (cache) => {
      const cachedResponse = await cache.match(event.request);
      
      // Fetch in background
      const fetchPromise = fetch(event.request).then((networkResponse) => {
        if (networkResponse.ok) {
          cache.put(event.request, networkResponse.clone());
        }
        return networkResponse;
      });
      
      // Return cached or wait for network
      return cachedResponse || fetchPromise;
    })
  );
});

// Workbox per SW più semplice
// npm install workbox-webpack-plugin
// next.config.js with next-pwa
const withPWA = require('next-pwa')({
  dest: 'public',
  disable: process.env.NODE_ENV === 'development',
  runtimeCaching: [
    {
      urlPattern: /^https:\/\/api\.example\.com\/.*/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'api-cache',
        expiration: {
          maxEntries: 100,
          maxAgeSeconds: 60 * 60 * 24, // 24 hours
        },
      },
    },
  ],
});

module.exports = withPWA({
  // next config
});

§ 9.5 REACT QUERY / SWR CACHING

typescript
// SWR - Stale-While-Revalidate per client-side
import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(res => res.json());

function Products() {
  const { data, error, isLoading, mutate } = useSWR(
    '/api/products',
    fetcher,
    {
      // Cache per 5 minuti
      dedupingInterval: 5 * 60 * 1000,
      
      // Rivalidazione automatica
      revalidateOnFocus: true,
      revalidateOnReconnect: true,
      refreshInterval: 30000, // Ogni 30s
      
      // Fallback data (da SSR o cache)
      fallbackData: initialData,
    }
  );
  
  if (error) return <div>Error loading</div>;
  if (isLoading) return <div>Loading...</div>;
  
  return <ProductList products={data} />;
}

// SWR con mutations
function useProducts() {
  const { data, mutate } = useSWR('/api/products', fetcher);
  
  const addProduct = async (newProduct) => {
    // Optimistic update
    mutate(
      [...(data || []), newProduct],
      false // Don't revalidate yet
    );
    
    // Send to server
    await fetch('/api/products', {
      method: 'POST',
      body: JSON.stringify(newProduct),
    });
    
    // Revalidate
    mutate();
  };
  
  return { products: data, addProduct };
}

// TanStack Query (React Query)
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

function Products() {
  const queryClient = useQueryClient();
  
  const { data, isLoading, error } = useQuery({
    queryKey: ['products'],
    queryFn: () => fetch('/api/products').then(res => res.json()),
    staleTime: 5 * 60 * 1000, // 5 min
    gcTime: 30 * 60 * 1000, // Cache per 30 min (ex cacheTime)
  });
  
  const addMutation = useMutation({
    mutationFn: (newProduct) =>
      fetch('/api/products', {
        method: 'POST',
        body: JSON.stringify(newProduct),
      }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products'] });
    },
  });
  
  return <ProductList products={data} onAdd={addMutation.mutate} />;
}

§ 9.6 REDIS CACHING

typescript
// lib/redis.ts
import { Redis } from '@upstash/redis';

export const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!,
});

// Cache wrapper
export async function cached<T>(
  key: string,
  fetcher: () => Promise<T>,
  ttlSeconds = 60
): Promise<T> {
  // Check cache
  const cached = await redis.get<T>(key);
  if (cached) {
    return cached;
  }
  
  // Fetch fresh
  const fresh = await fetcher();
  
  // Store in cache
  await redis.setex(key, ttlSeconds, fresh);
  
  return fresh;
}

// Usage
// app/api/products/route.ts
export async function GET() {
  const products = await cached(
    'products:all',
    () => fetchProductsFromDB(),
    300 // 5 min
  );
  
  return Response.json(products);
}

// Cache invalidation
export async function invalidateProductCache(productId?: string) {
  if (productId) {
    await redis.del(`product:${productId}`);
  }
  await redis.del('products:all');
}

// Pattern per invalidation granulare
// Usare tags/prefixes
await redis.set('products:featured', data);
await redis.set('products:new', data);
await redis.set('products:sale', data);

// Invalidate all products
const keys = await redis.keys('products:*');
if (keys.length > 0) {
  await redis.del(...keys);
}

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 10: CDN & EDGE COMPUTING
# ═══════════════════════════════════════════════════════════════════════════════

CDN_AND_EDGE = """

§ 10.1 CDN OVERVIEW

┌─────────────────────────────────────────────────────────────────────────────┐
│ CDN - CONTENT DELIVERY NETWORK                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                        ┌─────────────┐                                     │
│                        │   Origin    │                                     │
│                        │   Server    │                                     │
│                        └──────┬──────┘                                     │
│                               │                                            │
│              ┌────────────────┼────────────────┐                           │
│              │                │                │                           │
│        ┌─────▼─────┐   ┌─────▼─────┐   ┌─────▼─────┐                      │
│        │   Edge    │   │   Edge    │   │   Edge    │                      │
│        │   EU      │   │   US      │   │   Asia    │                      │
│        └─────┬─────┘   └─────┬─────┘   └─────┬─────┘                      │
│              │                │                │                           │
│        ┌─────▼─────┐   ┌─────▼─────┐   ┌─────▼─────┐                      │
│        │   Users   │   │   Users   │   │   Users   │                      │
│        │   EU      │   │   US      │   │   Asia    │                      │
│        └───────────┘   └───────────┘   └───────────┘                      │
│                                                                             │
│ BENEFICI:                                                                   │
│ ├── Riduzione latenza (contenuto più vicino all'utente)                    │
│ ├── Riduzione carico sul server origin                                     │
│ ├── Protezione DDoS                                                        │
│ ├── HTTPS automatico                                                       │
│ └── Compressione automatica                                                │
│                                                                             │
│ PROVIDERS POPOLARI:                                                         │
│ ├── Vercel (integrato con Next.js)                                         │
│ ├── Cloudflare                                                             │
│ ├── AWS CloudFront                                                         │
│ ├── Fastly                                                                 │
│ └── Akamai                                                                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 10.2 EDGE FUNCTIONS

typescript
// Vercel Edge Functions
// middleware.ts (runs at edge)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export const config = {
  matcher: ['/api/:path*', '/dashboard/:path*'],
};

export function middleware(request: NextRequest) {
  // A/B Testing
  const bucket = Math.random() < 0.5 ? 'a' : 'b';
  const response = NextResponse.next();
  response.cookies.set('ab-bucket', bucket);
  
  // Geolocation based routing
  const country = request.geo?.country || 'US';
  if (country === 'IT') {
    return NextResponse.redirect(new URL('/it', request.url));
  }
  
  // Rate limiting check
  const ip = request.ip ?? '127.0.0.1';
  // Check rate limit in edge KV store
  
  return response;
}

// Edge API Route
// app/api/geo/route.ts
import { NextRequest, NextResponse } from 'next/server';

export const runtime = 'edge'; // Run at edge

export async function GET(request: NextRequest) {
  const { geo, ip } = request;
  
  return NextResponse.json({
    country: geo?.country,
    city: geo?.city,
    region: geo?.region,
    ip,
  });
}

§ 10.3 EDGE CACHING CONFIGURATION

typescript
// Vercel configuration
// vercel.json
{
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "s-maxage=60, stale-while-revalidate=300"
        }
      ]
    }
  ],
  "regions": ["iad1", "sfo1", "cdg1", "hnd1"], // Deploy to specific regions
}

// Cloudflare Workers
// wrangler.toml
name = "my-worker"
main = "src/index.ts"
compatibility_date = "2024-01-01"

[vars]
MY_VAR = "my-value"

// src/index.ts
export default {
  async fetch(request, env, ctx) {
    const cache = caches.default;
    
    // Check cache
    let response = await cache.match(request);
    if (response) {
      return response;
    }
    
    // Fetch from origin
    response = await fetch(request);
    
    // Clone and cache
    ctx.waitUntil(cache.put(request, response.clone()));
    
    return response;
  },
};

§ 10.4 EDGE DATABASE

typescript
// Vercel KV (Redis at edge)
import { kv } from '@vercel/kv';

// Store data
await kv.set('user:1', { name: 'John', email: 'john@example.com' });

// Get data
const user = await kv.get('user:1');

// With expiration
await kv.setex('session:abc', 3600, { userId: 1 });

// Increment
await kv.incr('page:views');

// Vercel Postgres
import { sql } from '@vercel/postgres';

export async function getProducts() {
  const { rows } = await sql`SELECT * FROM products WHERE active = true`;
  return rows;
}

// Cloudflare D1 (SQLite at edge)
export default {
  async fetch(request, env) {
    const { results } = await env.DB.prepare(
      'SELECT * FROM products WHERE active = ?'
    ).bind(1).all();
    
    return Response.json(results);
  },
};

// Turso (libSQL at edge)
import { createClient } from '@libsql/client';

const client = createClient({
  url: process.env.TURSO_DATABASE_URL,
  authToken: process.env.TURSO_AUTH_TOKEN,
});

const result = await client.execute('SELECT * FROM products');

§ 10.5 IMAGE CDN

typescript
// Cloudinary configuration
// lib/cloudinary-loader.ts
export default function cloudinaryLoader({
  src,
  width,
  quality,
}: {
  src: string;
  width: number;
  quality?: number;
}) {
  const params = [
    'f_auto', // Auto format
    'c_fill', // Crop mode
    `w_${width}`,
    `q_${quality || 'auto'}`,
  ];
  
  const baseUrl = 'https://res.cloudinary.com/your-cloud/image/upload';
  return `${baseUrl}/${params.join(',')}${src}`;
}

// next.config.js
module.exports = {
  images: {
    loader: 'custom',
    loaderFile: './lib/cloudinary-loader.ts',
  },
};

// Imgix
export function imgixLoader({ src, width, quality }) {
  const url = new URL(`https://your-domain.imgix.net${src}`);
  const params = url.searchParams;
  
  params.set('w', width.toString());
  params.set('q', (quality || 75).toString());
  params.set('auto', 'format,compress');
  params.set('fit', 'max');
  
  return url.toString();
}

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 11: SERVER-SIDE RENDERING & STATIC GENERATION
# ═══════════════════════════════════════════════════════════════════════════════

SSR_AND_SSG = """

§ 11.1 RENDERING STRATEGIES OVERVIEW

┌─────────────────────────────────────────────────────────────────────────────┐
│ RENDERING STRATEGIES - Next.js App Router                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ STATIC (SSG)                                                                │
│ ├── Generato a build time                                                  │
│ ├── Cached e servito da CDN                                                │
│ ├── Velocissimo (no server computation)                                    │
│ └── Ideale per: blog, docs, landing pages                                  │
│                                                                             │
│ INCREMENTAL STATIC REGENERATION (ISR)                                      │
│ ├── Generato a build time + rigenerato periodicamente                      │
│ ├── Background revalidation                                                │
│ ├── Cache hit while regenerating                                           │
│ └── Ideale per: e-commerce, news, contenuto semi-dinamico                  │
│                                                                             │
│ SERVER-SIDE RENDERING (SSR)                                                │
│ ├── Generato ad ogni request                                               │
│ ├── Sempre fresh                                                           │
│ ├── Più lento (server computation)                                         │
│ └── Ideale per: user dashboard, contenuto personalizzato                   │
│                                                                             │
│ CLIENT-SIDE RENDERING (CSR)                                                 │
│ ├── Shell statica + fetch lato client                                      │
│ ├── Interattività immediata per shell                                      │
│ ├── Loading states per dati                                                │
│ └── Ideale per: app dashboard, dati real-time                              │
│                                                                             │
│ STREAMING SSR                                                               │
│ ├── HTML inviato progressivamente                                          │
│ ├── TTFB migliorato                                                        │
│ ├── Suspense boundaries                                                    │
│ └── Ideale per: pagine con sezioni a velocità diversa                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 11.2 STATIC GENERATION

typescript
// app/blog/page.tsx - Static by default
export default async function BlogPage() {
  // Questa fetch è cached a build time
  const posts = await fetch('https://api.example.com/posts', {
    cache: 'force-cache', // Default
  }).then(res => res.json());
  
  return (
    <div>
      <h1>Blog</h1>
      {posts.map(post => (
        <BlogCard key={post.id} post={post} />
      ))}
    </div>
  );
}

// generateStaticParams per dynamic routes
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts').then(res => res.json());
  
  return posts.map((post) => ({
    slug: post.slug,
  }));
}

export default async function PostPage({ params }: { params: { slug: string } }) {
  const post = await fetch(`https://api.example.com/posts/${params.slug}`).then(res => res.json());
  
  return <article>{post.content}</article>;
}

// Opt-out di dynamic params
export const dynamicParams = false; // 404 per slug non generati

§ 11.3 INCREMENTAL STATIC REGENERATION

typescript
// app/products/page.tsx - ISR
export const revalidate = 60; // Revalidate every 60 seconds

export default async function ProductsPage() {
  const products = await fetch('https://api.example.com/products', {
    next: { revalidate: 60 }, // Oppure qui
  }).then(res => res.json());
  
  return <ProductGrid products={products} />;
}

// On-demand revalidation
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';
import { NextRequest } from 'next/server';

export async function POST(request: NextRequest) {
  const body = await request.json();
  
  // Revalidate by path
  if (body.path) {
    revalidatePath(body.path);
  }
  
  // Revalidate by tag
  if (body.tag) {
    revalidateTag(body.tag);
  }
  
  return Response.json({ revalidated: true, now: Date.now() });
}

// Tag-based revalidation
async function getProduct(id: string) {
  return fetch(`https://api.example.com/products/${id}`, {
    next: { tags: ['products', `product-${id}`] },
  }).then(res => res.json());
}

// Webhook handler per CMS
export async function POST(request: NextRequest) {
  const payload = await request.json();
  
  // CMS ha aggiornato un prodotto
  if (payload.type === 'product.updated') {
    await revalidateTag(`product-${payload.productId}`);
  }
  
  return Response.json({ ok: true });
}

§ 11.4 DYNAMIC RENDERING (SSR)

typescript
// Force dynamic rendering
// app/dashboard/page.tsx
export const dynamic = 'force-dynamic';

export default async function DashboardPage() {
  const session = await getSession();
  const userData = await fetch(`/api/user/${session.userId}`, {
    cache: 'no-store', // No caching
  }).then(res => res.json());
  
  return <Dashboard data={userData} />;
}

// Dynamic functions trigger SSR automatically
// headers(), cookies(), searchParams
import { cookies, headers } from 'next/headers';

export default async function Page() {
  const cookieStore = await cookies();
  const theme = cookieStore.get('theme');
  
  const headersList = await headers();
  const userAgent = headersList.get('user-agent');
  
  return <div>Theme: {theme?.value}</div>;
}

// searchParams also makes page dynamic
export default async function SearchPage({
  searchParams,
}: {
  searchParams: { q?: string };
}) {
  const query = searchParams.q;
  const results = await search(query);
  
  return <SearchResults results={results} />;
}

§ 11.5 STREAMING SSR

typescript
// app/page.tsx - Streaming with Suspense
import { Suspense } from 'react';

export default function Page() {
  return (
    <div>
      {/* Header renders immediately */}
      <Header />
      
      {/* Main content streams in */}
      <Suspense fallback={<HeroSkeleton />}>
        <Hero />
      </Suspense>
      
      {/* Products stream independently */}
      <Suspense fallback={<ProductsSkeleton />}>
        <FeaturedProducts />
      </Suspense>
      
      {/* Reviews stream last */}
      <Suspense fallback={<ReviewsSkeleton />}>
        <CustomerReviews />
      </Suspense>
    </div>
  );
}

// Async component that might be slow
async function CustomerReviews() {
  // Slow API call
  const reviews = await fetch('https://slow-api.com/reviews', {
    cache: 'no-store',
  }).then(res => res.json());
  
  return <ReviewList reviews={reviews} />;
}

// loading.tsx for automatic suspense
// app/products/loading.tsx
export default function Loading() {
  return <ProductsSkeleton />;
}

// Nested layouts stream independently
// app/dashboard/layout.tsx
export default function DashboardLayout({ children }) {
  return (
    <div className="flex">
      {/* Sidebar can be separate suspense */}
      <Suspense fallback={<SidebarSkeleton />}>
        <Sidebar />
      </Suspense>
      
      <main>{children}</main>
    </div>
  );
}

§ 11.6 REACT SERVER COMPONENTS

typescript
// Server Components (default in app router)
// app/products/page.tsx
async function ProductsPage() {
  // Direct database access - no API needed
  const products = await db.product.findMany({
    where: { active: true },
  });
  
  // Heavy computation on server
  const analytics = processAnalytics(products);
  
  // Secrets safe on server
  const apiKey = process.env.SECRET_API_KEY;
  
  return (
    <div>
      <ProductList products={products} />
      <AnalyticsChart data={analytics} />
    </div>
  );
}

// Client Components for interactivity
// components/AddToCartButton.tsx
'use client';

import { useState } from 'react';

export function AddToCartButton({ productId }) {
  const [isAdding, setIsAdding] = useState(false);
  
  const handleAdd = async () => {
    setIsAdding(true);
    await addToCart(productId);
    setIsAdding(false);
  };
  
  return (
    <button onClick={handleAdd} disabled={isAdding}>
      {isAdding ? 'Adding...' : 'Add to Cart'}
    </button>
  );
}

// Mixing Server and Client Components
// app/products/[id]/page.tsx
async function ProductPage({ params }) {
  // Server: fetch product data
  const product = await getProduct(params.id);
  
  return (
    <div>
      {/* Server Component */}
      <ProductDetails product={product} />
      
      {/* Client Component */}
      <AddToCartButton productId={product.id} />
      
      {/* Server Component with Client child */}
      <ProductReviews productId={product.id}>
        <ReviewForm productId={product.id} />
      </ProductReviews>
    </div>
  );
}

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 12: DATABASE & API PERFORMANCE
# ═══════════════════════════════════════════════════════════════════════════════

DATABASE_AND_API_PERFORMANCE = """

§ 12.1 DATABASE QUERY OPTIMIZATION

typescript
// Prisma query optimization

// ❌ BAD: N+1 queries
async function getPostsWithAuthors() {
  const posts = await prisma.post.findMany();
  
  // N queries per gli autori!
  for (const post of posts) {
    post.author = await prisma.user.findUnique({
      where: { id: post.authorId },
    });
  }
  
  return posts;
}

// ✅ GOOD: Include relations
async function getPostsWithAuthors() {
  return prisma.post.findMany({
    include: {
      author: {
        select: {
          id: true,
          name: true,
          avatar: true,
        },
      },
    },
  });
}

// ✅ GOOD: Select solo campi necessari
async function getPostList() {
  return prisma.post.findMany({
    select: {
      id: true,
      title: true,
      excerpt: true,
      publishedAt: true,
      author: {
        select: {
          name: true,
        },
      },
    },
    where: {
      published: true,
    },
    orderBy: {
      publishedAt: 'desc',
    },
    take: 20,
  });
}

// Pagination efficiente
async function getPaginatedPosts(cursor?: string, limit = 20) {
  return prisma.post.findMany({
    take: limit + 1, // Fetch one extra to check if there's more
    ...(cursor && {
      skip: 1, // Skip the cursor
      cursor: { id: cursor },
    }),
    orderBy: { createdAt: 'desc' },
  });
}

§ 12.2 DATABASE INDEXING

sql
-- PostgreSQL indexes per query comuni

-- Index semplice
CREATE INDEX idx_posts_published_at ON posts(published_at DESC)
WHERE published = true;

-- Composite index
CREATE INDEX idx_posts_author_published ON posts(author_id, published_at DESC)
WHERE published = true;

-- Partial index
CREATE INDEX idx_active_products ON products(category_id)
WHERE active = true;

-- GIN index per JSONB
CREATE INDEX idx_products_metadata ON products USING GIN (metadata);

-- Full-text search index
CREATE INDEX idx_posts_search ON posts USING GIN (to_tsvector('english', title || ' ' || content));

typescript
// Prisma schema con indexes
// prisma/schema.prisma
model Post {
  id          String   @id @default(cuid())
  title       String
  content     String
  published   Boolean  @default(false)
  authorId    String
  publishedAt DateTime?
  createdAt   DateTime @default(now())
  
  author User @relation(fields: [authorId], references: [id])
  
  @@index([authorId])
  @@index([published, publishedAt(sort: Desc)])
}

model Product {
  id         String  @id @default(cuid())
  name       String
  categoryId String
  price      Decimal @db.Decimal(10, 2)
  active     Boolean @default(true)
  
  category Category @relation(fields: [categoryId], references: [id])
  
  @@index([categoryId, active])
  @@index([price])
}

§ 12.3 CONNECTION POOLING

typescript
// Prisma connection pooling
// lib/db.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query'] : [],
  });

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}

// Connection string con pooling
// .env
DATABASE_URL="postgresql://user:password@host:5432/db?connection_limit=20&pool_timeout=20"

// Prisma Accelerate per edge
// .env
DATABASE_URL="prisma://accelerate.prisma-data.net/?api_key=..."

// lib/db.ts con Prisma Accelerate
import { PrismaClient } from '@prisma/client';
import { withAccelerate } from '@prisma/extension-accelerate';

export const prisma = new PrismaClient().$extends(withAccelerate());

// Query con caching
const posts = await prisma.post.findMany({
  cacheStrategy: {
    ttl: 60, // 60 seconds
    swr: 300, // stale-while-revalidate for 5 minutes
  },
});

§ 12.4 API RESPONSE OPTIMIZATION

typescript
// Pagination
// app/api/products/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const page = parseInt(searchParams.get('page') || '1');
  const limit = Math.min(parseInt(searchParams.get('limit') || '20'), 100);
  const cursor = searchParams.get('cursor');
  
  // Cursor-based pagination (più efficiente per dataset grandi)
  if (cursor) {
    const products = await prisma.product.findMany({
      take: limit + 1,
      skip: 1,
      cursor: { id: cursor },
      orderBy: { createdAt: 'desc' },
    });
    
    const hasMore = products.length > limit;
    const items = hasMore ? products.slice(0, -1) : products;
    
    return NextResponse.json({
      items,
      nextCursor: hasMore ? items[items.length - 1].id : null,
    });
  }
  
  // Offset-based pagination
  const [products, total] = await Promise.all([
    prisma.product.findMany({
      skip: (page - 1) * limit,
      take: limit,
      orderBy: { createdAt: 'desc' },
    }),
    prisma.product.count(),
  ]);
  
  return NextResponse.json({
    items: products,
    pagination: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit),
    },
  });
}

// Field filtering
export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const fields = searchParams.get('fields')?.split(',');
  
  const select = fields?.reduce((acc, field) => {
    acc[field] = true;
    return acc;
  }, {} as Record<string, boolean>);
  
  const products = await prisma.product.findMany({
    select: select || undefined,
  });
  
  return NextResponse.json(products);
}

// Compression
export async function GET() {
  const data = await getLargeDataset();
  
  return new NextResponse(JSON.stringify(data), {
    headers: {
      'Content-Type': 'application/json',
      'Content-Encoding': 'gzip', // Vercel comprime automaticamente
    },
  });
}

§ 12.5 GRAPHQL PERFORMANCE

typescript
// GraphQL con DataLoader per evitare N+1
import DataLoader from 'dataloader';
import { prisma } from '@/lib/db';

// Create loaders
function createLoaders() {
  return {
    userLoader: new DataLoader(async (ids: readonly string[]) => {
      const users = await prisma.user.findMany({
        where: { id: { in: [...ids] } },
      });
      
      // Ritorna nello stesso ordine degli IDs
      const userMap = new Map(users.map((u) => [u.id, u]));
      return ids.map((id) => userMap.get(id) || null);
    }),
    
    postsByAuthorLoader: new DataLoader(async (authorIds: readonly string[]) => {
      const posts = await prisma.post.findMany({
        where: { authorId: { in: [...authorIds] } },
      });
      
      // Raggruppa per authorId
      const postsByAuthor = new Map<string, typeof posts>();
      for (const post of posts) {
        const existing = postsByAuthor.get(post.authorId) || [];
        existing.push(post);
        postsByAuthor.set(post.authorId, existing);
      }
      
      return authorIds.map((id) => postsByAuthor.get(id) || []);
    }),
  };
}

// Resolver con DataLoader
const resolvers = {
  Query: {
    posts: async () => prisma.post.findMany(),
  },
  Post: {
    author: async (parent, _, context) => {
      return context.loaders.userLoader.load(parent.authorId);
    },
  },
  User: {
    posts: async (parent, _, context) => {
      return context.loaders.postsByAuthorLoader.load(parent.id);
    },
  },
};

// Query complexity limiting
import { createComplexityRule, simpleEstimator } from 'graphql-query-complexity';

const complexityRule = createComplexityRule({
  maximumComplexity: 1000,
  estimators: [simpleEstimator({ defaultComplexity: 1 })],
  onComplete: (complexity) => {
    console.log('Query complexity:', complexity);
  },
});

§ 12.6 API RATE LIMITING

typescript
// Rate limiting con Upstash Redis
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!,
});

// 10 requests per 10 seconds per IP
const ratelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(10, '10 s'),
  analytics: true,
});

// Middleware
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export async function middleware(request: NextRequest) {
  const ip = request.ip ?? '127.0.0.1';
  
  const { success, limit, reset, remaining } = await ratelimit.limit(ip);
  
  if (!success) {
    return new NextResponse('Too Many Requests', {
      status: 429,
      headers: {
        'X-RateLimit-Limit': limit.toString(),
        'X-RateLimit-Remaining': remaining.toString(),
        'X-RateLimit-Reset': reset.toString(),
      },
    });
  }
  
  const response = NextResponse.next();
  response.headers.set('X-RateLimit-Limit', limit.toString());
  response.headers.set('X-RateLimit-Remaining', remaining.toString());
  
  return response;
}

export const config = {
  matcher: '/api/:path*',
};

// Different limits per tier
const rateLimits = {
  free: Ratelimit.slidingWindow(10, '10 s'),
  pro: Ratelimit.slidingWindow(100, '10 s'),
  enterprise: Ratelimit.slidingWindow(1000, '10 s'),
};

async function getRateLimiter(userId: string) {
  const user = await getUser(userId);
  return new Ratelimit({
    redis,
    limiter: rateLimits[user.tier],
    prefix: `ratelimit:${user.tier}`,
  });
}

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 13: PERFORMANCE MONITORING
# ═══════════════════════════════════════════════════════════════════════════════

PERFORMANCE_MONITORING = """

§ 13.1 REAL USER MONITORING (RUM)

typescript
// Web Vitals monitoring
// lib/analytics/web-vitals.ts
import { onCLS, onINP, onLCP, onFCP, onTTFB, Metric } from 'web-vitals';

interface VitalsData extends Metric {
  url: string;
  timestamp: number;
  deviceType: 'mobile' | 'tablet' | 'desktop';
  connection?: string;
}

function getDeviceType(): 'mobile' | 'tablet' | 'desktop' {
  if (typeof window === 'undefined') return 'desktop';
  const width = window.innerWidth;
  if (width < 768) return 'mobile';
  if (width < 1024) return 'tablet';
  return 'desktop';
}

function getConnectionType(): string | undefined {
  if (typeof navigator === 'undefined') return undefined;
  return (navigator as any).connection?.effectiveType;
}

function sendToAnalytics(metric: Metric) {
  const data: VitalsData = {
    ...metric,
    url: window.location.href,
    timestamp: Date.now(),
    deviceType: getDeviceType(),
    connection: getConnectionType(),
  };

  // Send via sendBeacon for reliability
  const blob = new Blob([JSON.stringify(data)], { type: 'application/json' });
  
  if (navigator.sendBeacon) {
    navigator.sendBeacon('/api/analytics/vitals', blob);
  } else {
    fetch('/api/analytics/vitals', {
      method: 'POST',
      body: blob,
      keepalive: true,
    });
  }
}

export function initWebVitals() {
  onCLS(sendToAnalytics);
  onINP(sendToAnalytics);
  onLCP(sendToAnalytics);
  onFCP(sendToAnalytics);
  onTTFB(sendToAnalytics);
}

// API endpoint per raccolta dati
// app/api/analytics/vitals/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const data = await request.json();
  
  // Store in database/analytics service
  await storeVitalsData(data);
  
  // Alert se poor performance
  if (
    (data.name === 'LCP' && data.value > 4000) ||
    (data.name === 'INP' && data.value > 500) ||
    (data.name === 'CLS' && data.value > 0.25)
  ) {
    await sendPerformanceAlert(data);
  }
  
  return NextResponse.json({ ok: true });
}

§ 13.2 LIGHTHOUSE CI

yaml
# .github/workflows/lighthouse.yml
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
            http://localhost:3000/about
          budgetPath: ./lighthouse-budget.json
          uploadArtifacts: true
          temporaryPublicStorage: true

json
// lighthouse-budget.json
[
  {
    "path": "/*",
    "timings": [
      {
        "metric": "largest-contentful-paint",
        "budget": 2500
      },
      {
        "metric": "first-contentful-paint",
        "budget": 1800
      },
      {
        "metric": "cumulative-layout-shift",
        "budget": 0.1
      },
      {
        "metric": "total-blocking-time",
        "budget": 200
      }
    ],
    "resourceSizes": [
      {
        "resourceType": "script",
        "budget": 300
      },
      {
        "resourceType": "total",
        "budget": 1000
      }
    ],
    "resourceCounts": [
      {
        "resourceType": "third-party",
        "budget": 10
      }
    ]
  }
]

§ 13.3 PERFORMANCE ALERTING

typescript
// lib/monitoring/alerts.ts
interface PerformanceThresholds {
  lcp: { warning: number; critical: number };
  inp: { warning: number; critical: number };
  cls: { warning: number; critical: number };
}

const thresholds: PerformanceThresholds = {
  lcp: { warning: 2500, critical: 4000 },
  inp: { warning: 200, critical: 500 },
  cls: { warning: 0.1, critical: 0.25 },
};

export async function checkPerformanceAlerts(data: VitalsData) {
  const threshold = thresholds[data.name.toLowerCase() as keyof PerformanceThresholds];
  
  if (!threshold) return;
  
  if (data.value >= threshold.critical) {
    await sendAlert({
      level: 'critical',
      metric: data.name,
      value: data.value,
      threshold: threshold.critical,
      url: data.url,
    });
  } else if (data.value >= threshold.warning) {
    await sendAlert({
      level: 'warning',
      metric: data.name,
      value: data.value,
      threshold: threshold.warning,
      url: data.url,
    });
  }
}

async function sendAlert(alert: Alert) {
  // Slack notification
  await fetch(process.env.SLACK_WEBHOOK_URL!, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      text: `🚨 Performance Alert: ${alert.metric}`,
      blocks: [
        {
          type: 'section',
          text: {
            type: 'mrkdwn',
            text: `*${alert.level.toUpperCase()}*: ${alert.metric} is ${alert.value}ms (threshold: ${alert.threshold}ms)\n*URL*: ${alert.url}`,
          },
        },
      ],
    }),
  });
  
  // PagerDuty for critical
  if (alert.level === 'critical') {
    await triggerPagerDuty(alert);
  }
}

§ 13.4 VERCEL ANALYTICS

typescript
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react';
import { SpeedInsights } from '@vercel/speed-insights/next';

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        {children}
        <Analytics />
        <SpeedInsights />
      </body>
    </html>
  );
}

// Custom events
import { track } from '@vercel/analytics';

// Track user action
track('product_view', {
  productId: product.id,
  category: product.category,
});

// Track conversion
track('purchase', {
  value: order.total,
  items: order.items.length,
});

§ 13.5 CUSTOM PERFORMANCE DASHBOARD

typescript
// app/api/analytics/dashboard/route.ts
import { prisma } from '@/lib/db';
import { NextResponse } from 'next/server';

export async function GET() {
  const now = new Date();
  const last24h = new Date(now.getTime() - 24 * 60 * 60 * 1000);
  const last7d = new Date(now.getTime() - 7 * 24 * 60 * 60 * 1000);
  
  // Aggregate vitals data
  const [vitals24h, vitals7d, topPages] = await Promise.all([
    // Last 24 hours averages
    prisma.webVitals.groupBy({
      by: ['name'],
      _avg: { value: true },
      _p75: { value: true },
      where: { timestamp: { gte: last24h } },
    }),
    
    // Last 7 days trend
    prisma.$queryRaw`
      SELECT 
        name,
        DATE(timestamp) as date,
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY value) as p75
      FROM web_vitals
      WHERE timestamp >= ${last7d}
      GROUP BY name, DATE(timestamp)
      ORDER BY date
    `,
    
    // Slowest pages
    prisma.webVitals.groupBy({
      by: ['url', 'name'],
      _avg: { value: true },
      where: { timestamp: { gte: last24h } },
      orderBy: { _avg: { value: 'desc' } },
      take: 10,
    }),
  ]);
  
  return NextResponse.json({
    current: vitals24h,
    trend: vitals7d,
    slowestPages: topPages,
    timestamp: now.toISOString(),
  });
}

// components/PerformanceDashboard.tsx
'use client';

import { useEffect, useState } from 'react';
import { LineChart, Line, XAxis, YAxis, Tooltip, ResponsiveContainer } from 'recharts';

export function PerformanceDashboard() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch('/api/analytics/dashboard')
      .then(res => res.json())
      .then(setData);
  }, []);
  
  if (!data) return <div>Loading...</div>;
  
  return (
    <div className="grid grid-cols-3 gap-4">
      {/* Current vitals cards */}
      <VitalsCard name="LCP" value={data.current.lcp} threshold={2500} />
      <VitalsCard name="INP" value={data.current.inp} threshold={200} />
      <VitalsCard name="CLS" value={data.current.cls} threshold={0.1} />
      
      {/* Trend chart */}
      <div className="col-span-3">
        <ResponsiveContainer width="100%" height={300}>
          <LineChart data={data.trend}>
            <XAxis dataKey="date" />
            <YAxis />
            <Tooltip />
            <Line dataKey="lcp" stroke="#8884d8" name="LCP" />
            <Line dataKey="inp" stroke="#82ca9d" name="INP" />
          </LineChart>
        </ResponsiveContainer>
      </div>
      
      {/* Slowest pages */}
      <div className="col-span-3">
        <h3>Slowest Pages</h3>
        <table>
          <thead>
            <tr>
              <th>URL</th>
              <th>LCP</th>
              <th>INP</th>
              <th>CLS</th>
            </tr>
          </thead>
          <tbody>
            {data.slowestPages.map((page) => (
              <tr key={page.url}>
                <td>{page.url}</td>
                <td>{page.lcp}ms</td>
                <td>{page.inp}ms</td>
                <td>{page.cls}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 14: PERFORMANCE BUDGETS
# ═══════════════════════════════════════════════════════════════════════════════

PERFORMANCE_BUDGETS = """

§ 14.1 DEFINIRE PERFORMANCE BUDGETS

┌─────────────────────────────────────────────────────────────────────────────┐
│ PERFORMANCE BUDGETS                                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ CORE WEB VITALS TARGETS                                                     │
│ ├── LCP:  < 2.5s (75th percentile)                                         │
│ ├── INP:  < 200ms (75th percentile)                                        │
│ └── CLS:  < 0.1 (75th percentile)                                          │
│                                                                             │
│ RESOURCE BUDGETS                                                            │
│ ├── Total page weight:        < 1MB (compressed)                           │
│ ├── JavaScript bundle:        < 300KB (compressed)                         │
│ ├── CSS:                      < 100KB                                      │
│ ├── Images per page:          < 500KB (compressed)                         │
│ ├── Fonts:                    < 100KB                                      │
│ └── Third-party scripts:      < 100KB                                      │
│                                                                             │
│ TIMING BUDGETS                                                              │
│ ├── TTFB:                     < 800ms                                      │
│ ├── FCP:                      < 1.8s                                       │
│ ├── Time to Interactive:      < 3.8s                                       │
│ └── Speed Index:              < 3.4s                                       │
│                                                                             │
│ REQUEST BUDGETS                                                             │
│ ├── Total requests:           < 50                                         │
│ ├── Third-party requests:     < 10                                         │
│ └── Fonts:                    < 5 files                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 14.2 IMPLEMENTARE BUDGET CHECKS

typescript
// scripts/check-bundle-size.ts
import { execSync } from 'child_process';
import fs from 'fs';
import path from 'path';

interface BundleBudget {
  name: string;
  maxSize: number; // in KB
}

const budgets: BundleBudget[] = [
  { name: 'main', maxSize: 150 },
  { name: 'vendor', maxSize: 200 },
  { name: 'pages/_app', maxSize: 50 },
];

function checkBundleSizes() {
  const buildDir = path.join(process.cwd(), '.next');
  const buildManifest = JSON.parse(
    fs.readFileSync(path.join(buildDir, 'build-manifest.json'), 'utf8')
  );
  
  const violations: string[] = [];
  
  for (const budget of budgets) {
    const files = findBundleFiles(buildDir, budget.name);
    const totalSize = files.reduce((sum, file) => {
      const stats = fs.statSync(file);
      return sum + stats.size;
    }, 0);
    
    const sizeKB = totalSize / 1024;
    
    if (sizeKB > budget.maxSize) {
      violations.push(
        `${budget.name}: ${sizeKB.toFixed(2)}KB exceeds budget of ${budget.maxSize}KB`
      );
    } else {
      console.log(`✅ ${budget.name}: ${sizeKB.toFixed(2)}KB / ${budget.maxSize}KB`);
    }
  }
  
  if (violations.length > 0) {
    console.error('\n❌ Bundle size violations:');
    violations.forEach((v) => console.error(`  - ${v}`));
    process.exit(1);
  }
  
  console.log('\n✅ All bundles within budget!');
}

checkBundleSizes();

json
// package.json
{
  "scripts": {
    "build": "next build",
    "postbuild": "npm run check:bundle && npm run check:lighthouse",
    "check:bundle": "ts-node scripts/check-bundle-size.ts",
    "check:lighthouse": "lhci autorun"
  }
}

§ 14.3 BUNDLE ANALYZER INTEGRATION

typescript
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

/** @type {import('next').NextConfig} */
const nextConfig = {
  // Webpack bundle size warnings
  experimental: {
    webpackBuildWorker: true,
  },
  
  webpack: (config, { isServer }) => {
    if (!isServer) {
      // Performance hints
      config.performance = {
        hints: 'warning',
        maxAssetSize: 300 * 1024, // 300KB
        maxEntrypointSize: 300 * 1024,
      };
    }
    return config;
  },
};

module.exports = withBundleAnalyzer(nextConfig);

yaml
# .github/workflows/bundle-check.yml
name: Bundle Size Check

on:
  pull_request:
    branches: [main]

jobs:
  bundle-size:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Build
        run: npm run build
        
      - name: Analyze bundle
        run: ANALYZE=true npm run build
        
      - name: Check bundle size
        uses: preactjs/compressed-size-action@v2
        with:
          repo-token: "${{ secrets.GITHUB_TOKEN }}"
          pattern: ".next/static/**/*.js"
          compression: "gzip"

§ 14.4 AUTOMATED PERFORMANCE TESTING IN CI

yaml
# .github/workflows/performance.yml
name: Performance Tests

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
        
      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v11
        with:
          configPath: './lighthouserc.json'
          uploadArtifacts: true
          
      - name: Comment PR with results
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const results = JSON.parse(fs.readFileSync('lhci-results.json'));
            
            const comment = `## Lighthouse Results
            
            | Metric | Score |
            |--------|-------|
            | Performance | ${results.performance} |
            | LCP | ${results.lcp}ms |
            | INP | ${results.inp}ms |
            | CLS | ${results.cls} |
            `;
            
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: comment
            });

json
// lighthouserc.json
{
  "ci": {
    "collect": {
      "url": [
        "http://localhost:3000/",
        "http://localhost:3000/products",
        "http://localhost:3000/about"
      ],
      "numberOfRuns": 3,
      "startServerCommand": "npm run start"
    },
    "assert": {
      "preset": "lighthouse:recommended",
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.9 }],
        "largest-contentful-paint": ["error", { "maxNumericValue": 2500 }],
        "cumulative-layout-shift": ["error", { "maxNumericValue": 0.1 }],
        "total-blocking-time": ["error", { "maxNumericValue": 200 }],
        "first-contentful-paint": ["warn", { "maxNumericValue": 1800 }]
      }
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 15: CHECKLIST PERFORMANCE PRE-DEPLOYMENT
# ═══════════════════════════════════════════════════════════════════════════════

PERFORMANCE_CHECKLIST = """

§ 15.1 CHECKLIST COMPLETA

┌─────────────────────────────────────────────────────────────────────────────┐
│ PERFORMANCE CHECKLIST PRE-DEPLOYMENT                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ ▢ CORE WEB VITALS                                                          │
│   ▢ LCP < 2.5s su mobile e desktop                                         │
│   ▢ INP < 200ms                                                            │
│   ▢ CLS < 0.1                                                              │
│   ▢ Testato con dati reali (CrUX) se disponibili                           │
│   ▢ Testato su connessioni lente (3G)                                      │
│                                                                             │
│ ▢ IMMAGINI                                                                  │
│   ▢ Tutte le immagini hanno width e height                                 │
│   ▢ Formato WebP/AVIF per immagini principali                              │
│   ▢ Lazy loading per immagini below-the-fold                               │
│   ▢ priority={true} per LCP image                                          │
│   ▢ sizes attribute appropriato                                            │
│   ▢ Placeholder blur per immagini grandi                                   │
│                                                                             │
│ ▢ JAVASCRIPT                                                                │
│   ▢ Bundle principale < 300KB (gzipped)                                    │
│   ▢ Code splitting implementato                                            │
│   ▢ Dynamic imports per componenti pesanti                                 │
│   ▢ No unused dependencies                                                 │
│   ▢ Third-party scripts caricati con strategy appropriata                  │
│   ▢ No blocking scripts nel <head>                                         │
│                                                                             │
│ ▢ CSS                                                                       │
│   ▢ Critical CSS inline                                                    │
│   ▢ CSS non usato rimosso                                                  │
│   ▢ No @import in CSS (usa bundler)                                        │
│   ▢ Fonts con font-display: swap                                           │
│                                                                             │
│ ▢ FONTS                                                                     │
│   ▢ Subset per caratteri usati                                             │
│   ▢ Preload per font critici                                               │
│   ▢ Variable fonts se multipli weights                                     │
│   ▢ Fallback font con metriche simili                                      │
│                                                                             │
│ ▢ CACHING                                                                   │
│   ▢ Static assets con immutable cache                                      │
│   ▢ API responses con cache headers appropriati                            │
│   ▢ Service worker per offline support (se necessario)                     │
│   ▢ CDN configurato correttamente                                          │
│                                                                             │
│ ▢ SERVER                                                                    │
│   ▢ TTFB < 800ms                                                           │
│   ▢ Gzip/Brotli compression attivo                                         │
│   ▢ HTTP/2 o HTTP/3 abilitato                                              │
│   ▢ Database queries ottimizzate                                           │
│   ▢ Connection pooling configurato                                         │
│                                                                             │
│ ▢ RENDERING                                                                 │
│   ▢ Server Components per contenuto statico                                │
│   ▢ Streaming per contenuto lento                                          │
│   ▢ ISR per contenuto semi-dinamico                                        │
│   ▢ Suspense boundaries appropriati                                        │
│                                                                             │
│ ▢ MONITORING                                                                │
│   ▢ Web Vitals tracking attivo                                             │
│   ▢ Error tracking configurato                                             │
│   ▢ Performance alerts impostati                                           │
│   ▢ Lighthouse CI in pipeline                                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 15.2 QUICK FIXES COMUNI

typescript
// Problemi comuni e soluzioni rapide

// ❌ PROBLEMA: LCP lento per hero image
// ✅ SOLUZIONE:
<Image
  src="/hero.jpg"
  priority // Preload
  fetchPriority="high"
  sizes="100vw"
/>

// ❌ PROBLEMA: CLS da immagini senza dimensioni
// ✅ SOLUZIONE:
<Image
  src={src}
  width={800}
  height={600}
  // oppure
  fill
  className="aspect-video"
/>

// ❌ PROBLEMA: INP lento per handler pesanti
// ✅ SOLUZIONE:
const handleClick = async () => {
  // Feedback visivo immediato
  setLoading(true);
  
  // Heavy work in next tick
  await new Promise(r => setTimeout(r, 0));
  await heavyComputation();
  
  setLoading(false);
};

// ❌ PROBLEMA: Bundle troppo grande
// ✅ SOLUZIONE:
// 1. Analizza bundle
ANALYZE=true npm run build

// 2. Dynamic import per componenti pesanti
const HeavyComponent = dynamic(() => import('./Heavy'), {
  loading: () => <Skeleton />,
});

// 3. Sostituisci librerie pesanti
// moment → date-fns
// lodash → lodash-es + named imports

// ❌ PROBLEMA: Fonts causano layout shift
// ✅ SOLUZIONE:
const font = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
});

// ❌ PROBLEMA: Third-party scripts bloccano
// ✅ SOLUZIONE:
<Script
  src="https://example.com/widget.js"
  strategy="lazyOnload" // O afterInteractive
/>

§ 15.3 PERFORMANCE AUDIT TEMPLATE

typescript
// scripts/performance-audit.ts
interface AuditResult {
  category: string;
  item: string;
  status: 'pass' | 'warning' | 'fail';
  value?: string | number;
  recommendation?: string;
}

async function runPerformanceAudit(): Promise<AuditResult[]> {
  const results: AuditResult[] = [];
  
  // 1. Build analysis
  const buildStats = await analyzeBuild();
  results.push({
    category: 'Bundle',
    item: 'Total JS size',
    status: buildStats.jsSize < 300000 ? 'pass' : 'fail',
    value: `${(buildStats.jsSize / 1024).toFixed(0)}KB`,
    recommendation: buildStats.jsSize >= 300000 
      ? 'Consider code splitting and removing unused dependencies'
      : undefined,
  });
  
  // 2. Lighthouse audit
  const lighthouse = await runLighthouse();
  results.push({
    category: 'Core Web Vitals',
    item: 'LCP',
    status: lighthouse.lcp < 2500 ? 'pass' : lighthouse.lcp < 4000 ? 'warning' : 'fail',
    value: `${lighthouse.lcp}ms`,
  });
  
  // 3. Image audit
  const images = await auditImages();
  results.push({
    category: 'Images',
    item: 'Missing dimensions',
    status: images.missingDimensions === 0 ? 'pass' : 'fail',
    value: `${images.missingDimensions} images`,
    recommendation: 'Add width and height to all images',
  });
  
  // 4. Font audit
  const fonts = await auditFonts();
  results.push({
    category: 'Fonts',
    item: 'Font display',
    status: fonts.allHaveSwap ? 'pass' : 'warning',
    recommendation: 'Use font-display: swap for all fonts',
  });
  
  return results;
}

// Generate report
async function generateAuditReport() {
  const results = await runPerformanceAudit();
  
  console.log('\n📊 PERFORMANCE AUDIT REPORT\n');
  console.log('═'.repeat(60));
  
  let currentCategory = '';
  for (const result of results) {
    if (result.category !== currentCategory) {
      currentCategory = result.category;
      console.log(`\n## ${currentCategory}\n`);
    }
    
    const icon = result.status === 'pass' ? '✅' : result.status === 'warning' ? '⚠️' : '❌';
    console.log(`${icon} ${result.item}: ${result.value || result.status}`);
    
    if (result.recommendation) {
      console.log(`   💡 ${result.recommendation}`);
    }
  }
  
  const passCount = results.filter(r => r.status === 'pass').length;
  const total = results.length;
  
  console.log('\n═'.repeat(60));
  console.log(`\nOverall: ${passCount}/${total} checks passed`);
  
  if (passCount < total) {
    process.exit(1);
  }
}

generateAuditReport();

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ APPENDICE: QUICK REFERENCE
# ═══════════════════════════════════════════════════════════════════════════════

APPENDIX_QUICK_REFERENCE = """

§ A.1 COMANDI UTILI

bash
# Build & Analyze
pnpm build                      # Production build
ANALYZE=true pnpm build         # Bundle analyzer
pnpm run lint                   # Lint check

# Lighthouse
npx lighthouse https://example.com --view
npx lhci autorun               # Lighthouse CI

# Performance testing
npx autocannon -c 100 http://localhost:3000  # Load test
npx clinic doctor -- node server.js          # Node.js profiling

# Bundle check
npx source-map-explorer .next/static/chunks/*.js
npx bundlephobia <package-name>             # Check package size

§ A.2 CORE WEB VITALS THRESHOLDS

┌─────────────────────────────────────────────────────────────────────────────┐
│ METRIC         │ GOOD          │ NEEDS IMPROVEMENT │ POOR                   │
├────────────────┼───────────────┼───────────────────┼────────────────────────┤
│ LCP            │ ≤ 2.5s        │ 2.5s - 4s         │ > 4s                   │
│ INP            │ ≤ 200ms       │ 200ms - 500ms     │ > 500ms                │
│ CLS            │ ≤ 0.1         │ 0.1 - 0.25        │ > 0.25                 │
│ TTFB           │ ≤ 800ms       │ 800ms - 1800ms    │ > 1800ms               │
│ FCP            │ ≤ 1.8s        │ 1.8s - 3s         │ > 3s                   │
│ TBT            │ ≤ 200ms       │ 200ms - 600ms     │ > 600ms                │
└─────────────────────────────────────────────────────────────────────────────┘

§ A.3 RESOURCE BUDGETS TEMPLATE

json
{
  "budgets": {
    "javascript": {
      "total": "300KB",
      "firstParty": "200KB",
      "thirdParty": "100KB"
    },
    "css": {
      "total": "100KB",
      "critical": "15KB"
    },
    "images": {
      "hero": "200KB",
      "thumbnail": "50KB"
    },
    "fonts": {
      "total": "100KB",
      "perFont": "50KB"
    },
    "requests": {
      "total": 50,
      "thirdParty": 10
    }
  }
}

§ A.4 IMAGE OPTIMIZATION CHEATSHEET

typescript
// LCP Image (hero/above-fold)
<Image
  src="/hero.jpg"
  alt="Hero"
  width={1200}
  height={600}
  priority
  quality={85}
  sizes="100vw"
/>

// Product Image (lazy)
<Image
  src={product.image}
  alt={product.name}
  width={400}
  height={400}
  sizes="(max-width: 768px) 100vw, 33vw"
/>

// Background Image (fill)
<div className="relative aspect-video">
  <Image
    src="/bg.jpg"
    alt=""
    fill
    className="object-cover"
  />
</div>

§ A.5 CACHE-CONTROL CHEATSHEET

# Static assets (hashed filenames)
Cache-Control: public, max-age=31536000, immutable

# API responses (short cache + SWR)
Cache-Control: public, s-maxage=60, stale-while-revalidate=300

# HTML pages (no cache, must revalidate)
Cache-Control: public, max-age=0, must-revalidate

# Private data (user-specific)
Cache-Control: private, no-cache

# Sensitive data (no storage)
Cache-Control: no-store

§ A.6 PERFORMANCE MONITORING SETUP

typescript
// Minimal web-vitals setup
import { onCLS, onINP, onLCP } from 'web-vitals';

export function reportWebVitals() {
  onCLS(console.log);
  onINP(console.log);
  onLCP(console.log);
}

// With analytics
function sendToAnalytics(metric) {
  fetch('/api/vitals', {
    method: 'POST',
    body: JSON.stringify(metric),
    keepalive: true,
  });
}

onCLS(sendToAnalytics);
onINP(sendToAnalytics);
onLCP(sendToAnalytics);

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ FINE CATALOGO
# ═══════════════════════════════════════════════════════════════════════════════

"""
═══════════════════════════════════════════════════════════════════════════════
CATALOGO-PERFORMANCE-v1.md - COMPLETO
═══════════════════════════════════════════════════════════════════════════════

Sezioni incluse:
1.  Core Web Vitals Overview
2.  LCP - Largest Contentful Paint
3.  INP - Interaction to Next Paint
4.  CLS - Cumulative Layout Shift
5.  Image Optimization
6.  JavaScript Optimization
7.  CSS Optimization
8.  Font Optimization
9.  Caching Strategies
10. CDN & Edge Computing
11. Server-Side Rendering & Static Generation
12. Database & API Performance
13. Performance Monitoring
14. Performance Budgets
15. Checklist Performance Pre-Deployment

Appendice:
- Quick Reference Commands
- Core Web Vitals Thresholds
- Resource Budgets Template
- Image Optimization Cheatsheet
- Cache-Control Cheatsheet
- Performance Monitoring Setup

═══════════════════════════════════════════════════════════════════════════════
"""
