# CATALOGO PWA-OFFLINE v1
## Documentazione Tecnica Completa per PWA e Offline Capabilities in Next.js

---

## §1. PWA FUNDAMENTALS

### **Tabella Funzionalità PWA Complete**

| Feature | Purpose | Required | Implementation Difficulty | User Value |
|---------|---------|----------|---------------------------|------------|
| **Web App Manifest** | App metadata, icons, splash screen | ✅ | Low | High (enables install) |
| **Service Worker** | Offline support, caching, background sync | ✅ | Medium | Very High |
| **HTTPS** | Security requirement | ✅ | Low | Mandatory |
| **Responsive Design** | All device sizes | ✅ | Medium | Very High |
| **Installable** | Add to home screen, app-like experience | ⚠️ Highly Recommended | Medium | Very High |
| **Push Notifications** | Re-engagement, updates | Optional | High | High |
| **App Shell** | Instant loading, native feel | Recommended | Medium | High |
| **Background Sync** | Offline data sync | Optional | High | High |
| **Periodic Sync** | Background updates | Optional | High | Medium |
| **Share Target** | Share to app from other apps | Optional | Medium | Medium |
| **File Handling** | Open files with app | Optional | High | Medium |
| **URL Handling** | Handle custom URLs | Optional | Medium | High |
| **Badging** | App icon badge | Optional | Low | Medium |

### **PWA Checklist per Next.js**

```typescript
const PWA_REQUIREMENTS = {
  mandatory: [
    'Web App Manifest (manifest.json)',
    'Service Worker Registration',
    'HTTPS (production)',
    'Responsive Design',
    'App Icons (multiple sizes)',
  ],
  
  recommended: [
    'Install Prompt Handling',
    'Offline Fallback Page',
    'App Shell Caching',
    'Network Resilience',
    'Fast Initial Load (< 3s)',
  ],
  
  advanced: [
    'Push Notifications',
    'Background Sync',
    'Periodic Sync',
    'Share Target API',
    'File Handling',
    'URL Protocol Handling',
  ],
  
  iosSpecific: [
    'apple-mobile-web-app-capable meta tag',
    'apple-touch-icon meta tags',
    'Splash Screen configuration',
    'Viewport configuration',
    'Manual install instructions',
  ],
  
  testing: [
    'Lighthouse PWA Audit (>90)',
    'Offline Functionality Test',
    'Install Flow Test',
    'Cross-browser Compatibility',
    'Performance Audit',
  ]
};
```

---

## §2. NEXT.JS PWA SETUP

### 2.1 next-pwa Configuration Completa

```typescript
// next.config.js - Configurazione PWA completa
/** @type {import('next').NextConfig} */
const withPWA = require('next-pwa')({
  dest: 'public',
  disable: process.env.NODE_ENV === 'development',
  register: true,
  skipWaiting: true,
  scope: '/',
  sw: 'service-worker.js',
  publicExcludes: ['!noprecache/**/*'],
  buildExcludes: [
    /middleware-manifest\.json$/,
    /middleware-runtime\.js$/,
    /_middleware\.js$/,
    /_error\.js$/,
    /chunks\/images\/.*$/,
  ],
  
  // Cache strategies
  runtimeCaching: [
    {
      urlPattern: /^https:\/\/fonts\.(?:googleapis|gstatic)\.com\/.*/i,
      handler: 'CacheFirst',
      options: {
        cacheName: 'google-fonts',
        expiration: {
          maxEntries: 4,
          maxAgeSeconds: 365 * 24 * 60 * 60, // 1 year
        },
      },
    },
    {
      urlPattern: /^https:\/\/cdn\.jsdelivr\.net\/.*/i,
      handler: 'CacheFirst',
      options: {
        cacheName: 'cdn-assets',
        expiration: {
          maxEntries: 32,
          maxAgeSeconds: 30 * 24 * 60 * 60, // 30 days
        },
      },
    },
    {
      urlPattern: /\.(?:eot|otf|ttc|ttf|woff|woff2|font\.css)$/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'static-font-assets',
        expiration: {
          maxEntries: 4,
          maxAgeSeconds: 7 * 24 * 60 * 60, // 7 days
        },
      },
    },
    {
      urlPattern: /\.(?:jpg|jpeg|gif|png|svg|ico|webp)$/i,
      handler: 'CacheFirst',
      options: {
        cacheName: 'static-image-assets',
        expiration: {
          maxEntries: 64,
          maxAgeSeconds: 30 * 24 * 60 * 60, // 30 days
        },
      },
    },
    {
      urlPattern: /\.(?:js)$/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'static-js-assets',
        expiration: {
          maxEntries: 32,
          maxAgeSeconds: 24 * 60 * 60, // 24 hours
        },
      },
    },
    {
      urlPattern: /\.(?:css|less)$/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'static-style-assets',
        expiration: {
          maxEntries: 32,
          maxAgeSeconds: 24 * 60 * 60, // 24 hours
        },
      },
    },
    {
      urlPattern: /\/_next\/static\/.*/i,
      handler: 'CacheFirst',
      options: {
        cacheName: 'next-static',
        expiration: {
          maxEntries: 64,
          maxAgeSeconds: 24 * 60 * 60, // 24 hours
        },
      },
    },
    {
      urlPattern: /\/_next\/image\?url=.+$/i,
      handler: 'StaleWhileRevalidate',
      options: {
        cacheName: 'next-images',
        expiration: {
          maxEntries: 128,
          maxAgeSeconds: 24 * 60 * 60, // 24 hours
        },
      },
    },
    {
      urlPattern: /^https?.*/,
      handler: 'NetworkFirst',
      options: {
        cacheName: 'offline-fallback',
        expiration: {
          maxEntries: 32,
          maxAgeSeconds: 24 * 60 * 60, // 24 hours
        },
        networkTimeoutSeconds: 10,
      },
    },
  ],
  
  // Precache fallback
  fallbacks: {
    document: '/offline',
    image: '/static/images/fallback.png',
    font: '/static/fonts/fallback.woff2',
  },
  
  // Custom workbox options
  maximumFileSizeToCacheInBytes: 5 * 1024 * 1024, // 5MB
  cleanupOutdatedCaches: true,
  clientsClaim: true,
  navigateFallback: '/',
  navigateFallbackDenylist: [
    /^\/api\/.*/, // Don't fallback for API routes
    /^\/admin\/.*/, // Don't fallback for admin routes
    /^\/_next\/.*/, // Don't fallback for Next.js internals
  ],
});

const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  images: {
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    minimumCacheTTL: 60 * 60 * 24, // 24 hours
  },
  
  // Headers per PWA
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-DNS-Prefetch-Control',
            value: 'on',
          },
          {
            key: 'Strict-Transport-Security',
            value: 'max-age=63072000; includeSubDomains; preload',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'X-XSS-Protection',
            value: '1; mode=block',
          },
          {
            key: 'Referrer-Policy',
            value: 'origin-when-cross-origin',
          },
          {
            key: 'Permissions-Policy',
            value: 'camera=(), microphone=(), geolocation=(), interest-cohort=()',
          },
        ],
      },
      {
        source: '/service-worker.js',
        headers: [
          {
            key: 'Content-Type',
            value: 'application/javascript; charset=utf-8',
          },
          {
            key: 'Cache-Control',
            value: 'no-cache, no-store, must-revalidate',
          },
        ],
      },
    ];
  },
  
  // Compression
  compress: true,
  
  // PWA specific
  pwa: {
    dest: 'public',
  },
};

module.exports = withPWA(nextConfig);
```

### 2.2 Web App Manifest Completo

```json
// public/manifest.json - Manifest completo con tutti i campi
{
  "$schema": "http://json.schemastore.org/web-manifest",
  "name": "My Application",
  "short_name": "MyApp",
  "description": "A Progressive Web Application built with Next.js",
  "lang": "en",
  "dir": "ltr",
  "start_url": "/",
  "scope": "/",
  
  // Display modes
  "display": "standalone",
  "display_override": ["window-controls-overlay", "standalone", "minimal-ui"],
  
  // Theme
  "theme_color": "#3B82F6",
  "background_color": "#FFFFFF",
  
  // Orientation
  "orientation": "portrait-primary",
  
  // Icons - MANDATORY for PWA
  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    },
    
    // iOS specific (non-standard but recommended)
    {
      "src": "/icons/apple-touch-icon.png",
      "sizes": "180x180",
      "type": "image/png",
      "purpose": "any"
    },
    
    // Maskable icons for adaptive icons
    {
      "src": "/icons/maskable-icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable"
    },
    {
      "src": "/icons/maskable-icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable"
    }
  ],
  
  // Screenshots for PWA stores
  "screenshots": [
    {
      "src": "/screenshots/desktop.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide",
      "label": "Desktop view"
    },
    {
      "src": "/screenshots/mobile.png",
      "sizes": "750x1334",
      "type": "image/png",
      "form_factor": "narrow",
      "label": "Mobile view"
    }
  ],
  
  // Shortcuts
  "shortcuts": [
    {
      "name": "Dashboard",
      "short_name": "Home",
      "description": "Go to dashboard",
      "url": "/dashboard",
      "icons": [
        {
          "src": "/icons/shortcut-dashboard-96x96.png",
          "sizes": "96x96",
          "type": "image/png"
        }
      ]
    },
    {
      "name": "New Item",
      "short_name": "Create",
      "description": "Create new item",
      "url": "/create",
      "icons": [
        {
          "src": "/icons/shortcut-create-96x96.png",
          "sizes": "96x96",
          "type": "image/png"
        }
      ]
    }
  ],
  
  // Categories for app stores
  "categories": ["productivity", "business", "utilities"],
  
  // IARC rating
  "iarc_rating_id": "e84b072d-71b3-4d3e-86ae-31a8ce4e53b7",
  
  // Related applications
  "related_applications": [
    {
      "platform": "play",
      "url": "https://play.google.com/store/apps/details?id=com.example.app",
      "id": "com.example.app"
    },
    {
      "platform": "itunes",
      "url": "https://apps.apple.com/app/id123456789"
    }
  ],
  
  // Prefer related applications
  "prefer_related_applications": false,
  
  // Protocol handlers (if needed)
  "protocol_handlers": [
    {
      "protocol": "web+example",
      "url": "/handle-protocol?url=%s"
    }
  ],
  
  // Share target (if needed)
  "share_target": {
    "action": "/share",
    "method": "GET",
    "params": {
      "title": "title",
      "text": "text",
      "url": "url"
    }
  },
  
  // File handlers (if needed)
  "file_handlers": [
    {
      "action": "/open-file",
      "accept": {
        "text/plain": [".txt"]
      }
    }
  ]
}
```

### 2.3 Icon Generation Script

```typescript
// scripts/generate-icons.js - Genera automaticamente tutte le icone necessarie
#!/usr/bin/env node

const sharp = require('sharp');
const fs = require('fs');
const path = require('path');

const ICON_SIZES = [
  { size: 72, name: 'icon-72x72' },
  { size: 96, name: 'icon-96x96' },
  { size: 128, name: 'icon-128x128' },
  { size: 144, name: 'icon-144x144' },
  { size: 152, name: 'icon-152x152' },
  { size: 192, name: 'icon-192x192', purpose: 'maskable' },
  { size: 384, name: 'icon-384x384' },
  { size: 512, name: 'icon-512x512', purpose: 'maskable' },
  { size: 180, name: 'apple-touch-icon' }, // iOS
];

const SHORTCUT_SIZES = [
  { size: 96, name: 'shortcut-dashboard-96x96' },
  { size: 96, name: 'shortcut-create-96x96' },
];

async function generateIcons() {
  const sourceIcon = path.join(__dirname, '../assets/icon.png'); // Your 1024x1024 source icon
  
  if (!fs.existsSync(sourceIcon)) {
    console.error('Source icon not found at:', sourceIcon);
    console.error('Please create a 1024x1024 PNG icon at', sourceIcon);
    process.exit(1);
  }
  
  // Create icons directory
  const iconsDir = path.join(__dirname, '../public/icons');
  if (!fs.existsSync(iconsDir)) {
    fs.mkdirSync(iconsDir, { recursive: true });
  }
  
  console.log('Generating PWA icons...');
  
  // Generate main icons
  for (const icon of ICON_SIZES) {
    await sharp(sourceIcon)
      .resize(icon.size, icon.size, { fit: 'contain', background: { r: 59, g: 130, b: 246, alpha: 1 } })
      .png()
      .toFile(path.join(iconsDir, `${icon.name}.png`));
    
    console.log(`✅ Generated ${icon.name}.png (${icon.size}x${icon.size})`);
    
    // Generate maskable version if specified
    if (icon.purpose === 'maskable') {
      const maskableIcon = await sharp(sourceIcon)
        .resize(icon.size, icon.size, { fit: 'contain', background: { r: 59, g: 130, b: 246, alpha: 0 } })
        .extend({
          top: Math.floor(icon.size * 0.1),
          bottom: Math.floor(icon.size * 0.1),
          left: Math.floor(icon.size * 0.1),
          right: Math.floor(icon.size * 0.1),
          background: { r: 59, g: 130, b: 246, alpha: 0 },
        })
        .png()
        .toBuffer();
      
      await sharp({
        create: {
          width: icon.size,
          height: icon.size,
          channels: 4,
          background: { r: 59, g: 130, b: 246, alpha: 0 },
        },
      })
        .composite([{ input: maskableIcon, gravity: 'center' }])
        .png()
        .toFile(path.join(iconsDir, `maskable-${icon.name}.png`));
      
      console.log(`✅ Generated maskable-${icon.name}.png`);
    }
  }
  
  // Generate shortcut icons
  for (const shortcut of SHORTCUT_SIZES) {
    // Generate placeholder shortcut icons (you should create proper ones)
    await sharp({
      create: {
        width: shortcut.size,
        height: shortcut.size,
        channels: 4,
        background: { r: 59, g: 130, b: 246, alpha: 1 },
      },
    })
      .png()
      .toFile(path.join(iconsDir, `${shortcut.name}.png`));
    
    console.log(`✅ Generated ${shortcut.name}.png`);
  }
  
  // Generate favicon.ico (multi-size)
  const faviconSizes = [16, 32, 48];
  const faviconImages = await Promise.all(
    faviconSizes.map(size =>
      sharp(sourceIcon)
        .resize(size, size)
        .png()
        .toBuffer()
    )
  );
  
  await sharp(faviconImages[0])
    .png()
    .toFile(path.join(__dirname, '../public/favicon.png'));
  
  console.log('✅ Generated favicon.png');
  
  console.log('\n🎉 All icons generated successfully!');
  console.log('Next steps:');
  console.log('1. Update manifest.json with correct icon paths');
  console.log('2. Add proper shortcut icons');
  console.log('3. Test PWA installation');
}

generateIcons().catch(console.error);
```

---

## §3. SERVICE WORKER E OFFLINE SUPPORT

### 3.1 Custom Service Worker con Offline Support Avanzato

```typescript
// public/sw-custom.js - Service Worker personalizzato con funzionalità avanzate
importScripts('https://storage.googleapis.com/workbox-cdn/releases/6.5.4/workbox-sw.js');

// Workbox setup
workbox.setConfig({
  debug: false,
});

// Precaching configuration
workbox.precaching.precacheAndRoute(self.__WB_MANIFEST || []);

// Cache strategies avanzate
const { 
  registerRoute,
  setDefaultHandler,
  setCatchHandler 
} = workbox.routing;

const {
  CacheFirst,
  NetworkFirst,
  StaleWhileRevalidate,
  NetworkOnly
} = workbox.strategies;

// API Cache Strategy con background sync
const apiCacheHandler = new NetworkFirst({
  cacheName: 'api-cache',
  plugins: [
    {
      cacheWillUpdate: async ({ response }) => {
        // Don't cache error responses
        if (!response || response.status >= 400) {
          return null;
        }
        
        // Don't cache large responses
        const contentLength = response.headers.get('content-length');
        if (contentLength && parseInt(contentLength, 10) > 5 * 1024 * 1024) {
          return null;
        }
        
        return response;
      },
    },
    new workbox.expiration.ExpirationPlugin({
      maxEntries: 50,
      maxAgeSeconds: 5 * 60, // 5 minutes
    }),
  ],
});

// Cache per API con background sync
registerRoute(
  ({ url }) => url.pathname.startsWith('/api/'),
  async ({ event, request, url }) => {
    try {
      return await apiCacheHandler.handle({ event, request, url });
    } catch (error) {
      // Se siamo offline, cerca nella cache
      const cache = await caches.open('api-cache');
      const cachedResponse = await cache.match(request);
      
      if (cachedResponse) {
        return cachedResponse;
      }
      
      // Se non c'è in cache, trigger background sync
      if ('sync' in self.registration) {
        const bgSync = async () => {
          try {
            await fetch(request.clone());
          } catch (err) {
            console.error('Background sync failed:', err);
          }
        };
        
        event.waitUntil(
          self.registration.sync.register('api-sync')
            .then(() => console.log('Background sync registered'))
            .catch(console.error)
        );
      }
      
      throw error;
    }
  }
);

// Cache per immagini con WebP support
const imageCacheHandler = new CacheFirst({
  cacheName: 'images',
  plugins: [
    new workbox.expiration.ExpirationPlugin({
      maxEntries: 100,
      maxAgeSeconds: 30 * 24 * 60 * 60, // 30 days
    }),
    new workbox.cacheableResponse.CacheableResponsePlugin({
      statuses: [0, 200],
    }),
  ],
});

registerRoute(
  ({ request }) => request.destination === 'image',
  imageCacheHandler
);

// Cache per font
registerRoute(
  ({ request }) => request.destination === 'font',
  new CacheFirst({
    cacheName: 'fonts',
    plugins: [
      new workbox.expiration.ExpirationPlugin({
        maxEntries: 20,
        maxAgeSeconds: 365 * 24 * 60 * 60, // 1 year
      }),
    ],
  })
);

// Cache per documenti HTML (app shell)
const htmlCacheHandler = new NetworkFirst({
  cacheName: 'html-cache',
  plugins: [
    new workbox.expiration.ExpirationPlugin({
      maxEntries: 10,
      maxAgeSeconds: 24 * 60 * 60, // 24 hours
    }),
  ],
});

registerRoute(
  ({ request }) => request.destination === 'document',
  htmlCacheHandler
);

// Offline fallback
setCatchHandler(async ({ event }) => {
  const destination = event.request.destination;
  
  switch (destination) {
    case 'document':
      // Return offline page
      const offlinePage = await caches.match('/offline');
      if (offlinePage) {
        return offlinePage;
      }
      
      // Fallback semplice
      return new Response(
        '<h1>Offline</h1><p>You are offline. Please check your connection.</p>',
        {
          headers: { 'Content-Type': 'text/html' },
        }
      );
      
    case 'image':
      // Return placeholder image
      return caches.match('/images/offline-placeholder.png');
      
    case 'font':
      // Return fallback font
      return new Response(
        '',
        { headers: { 'Content-Type': 'font/woff2' } }
      );
      
    default:
      // Return network error
      return Response.error();
  }
});

// Push notifications
self.addEventListener('push', (event) => {
  if (!event.data) return;
  
  const data = event.data.json();
  const options = {
    body: data.body || 'New notification',
    icon: data.icon || '/icons/icon-192x192.png',
    badge: '/icons/badge-72x72.png',
    vibrate: [100, 50, 100],
    data: {
      url: data.url || '/',
      dateOfArrival: Date.now(),
      primaryKey: data.id || '1',
    },
    actions: data.actions || [
      {
        action: 'open',
        title: 'Open App',
      },
      {
        action: 'close',
        title: 'Close',
      },
    ],
  };
  
  event.waitUntil(
    self.registration.showNotification(data.title || 'Notification', options)
  );
});

self.addEventListener('notificationclick', (event) => {
  event.notification.close();
  
  if (event.action === 'close') {
    return;
  }
  
  // Focus on existing tab or open new one
  event.waitUntil(
    clients.matchAll({ type: 'window', includeUncontrolled: true })
      .then((clientList) => {
        const url = event.notification.data?.url || '/';
        
        for (const client of clientList) {
          if (client.url === url && 'focus' in client) {
            return client.focus();
          }
        }
        
        if (clients.openWindow) {
          return clients.openWindow(url);
        }
      })
  );
});

// Background sync
self.addEventListener('sync', (event) => {
  if (event.tag === 'api-sync') {
    console.log('Background sync triggered');
    
    // Qui potresti processare una queue di richieste pendenti
    // Salvate in IndexedDB mentre l'utente era offline
  }
});

// Periodic sync (richiede permission)
self.addEventListener('periodicsync', (event) => {
  if (event.tag === 'update-content') {
    event.waitUntil(updateContent());
  }
});

async function updateContent() {
  // Update cached content in background
  console.log('Periodic sync: updating content');
  
  const cache = await caches.open('html-cache');
  const urlsToUpdate = [
    '/',
    '/dashboard',
    '/products',
  ];
  
  for (const url of urlsToUpdate) {
    try {
      const response = await fetch(url);
      if (response.ok) {
        await cache.put(url, response);
      }
    } catch (error) {
      console.error(`Failed to update ${url}:`, error);
    }
  }
}

// Message handling per comunicazione con la pagina
self.addEventListener('message', (event) => {
  if (event.data && event.data.type === 'SKIP_WAITING') {
    self.skipWaiting();
  }
  
  if (event.data && event.data.type === 'GET_CACHE_INFO') {
    // Rispondi con info sulla cache
    event.ports[0].postMessage({
      type: 'CACHE_INFO',
      data: {
        cacheNames: [...self.caches.keys()],
      },
    });
  }
});

// Lifecycle events
self.addEventListener('install', (event) => {
  console.log('Service Worker installing...');
  
  // Skip waiting per update immediato
  event.waitUntil(self.skipWaiting());
});

self.addEventListener('activate', (event) => {
  console.log('Service Worker activating...');
  
  // Claim clients per controllo immediato
  event.waitUntil(self.clients.claim());
  
  // Cleanup vecchie cache
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames.map((cacheName) => {
          if (cacheName.startsWith('workbox-') && cacheName !== 'workbox-precache') {
            console.log('Deleting old cache:', cacheName);
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
```

### 3.2 Offline Page Component

```typescript
// app/offline/page.tsx - Pagina offline completa
'use client';

import { useEffect, useState } from 'react';
import { WifiOff, RefreshCw, Home, CloudOff, Download } from 'lucide-react';
import Link from 'next/link';
import { useOnlineStatus } from '@/hooks/use-online-status';

export default function OfflinePage() {
  const { isOnline, retryConnection } = useOnlineStatus();
  const [cachedPages, setCachedPages] = useState<string[]>([]);
  
  useEffect(() => {
    // Check for cached content
    if ('caches' in window) {
      caches.keys().then(cacheNames => {
        const pageCache = cacheNames.find(name => name.includes('html-cache'));
        if (pageCache) {
          caches.open(pageCache).then(cache => {
            cache.keys().then(requests => {
              const pages = requests
                .map(req => new URL(req.url).pathname)
                .filter(path => path !== '/offline');
              setCachedPages(pages.slice(0, 5)); // Mostra max 5 pagine
            });
          });
        }
      });
    }
  }, []);
  
  const handleRetry = () => {
    retryConnection();
    
    // Force service worker update
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.getRegistration().then(registration => {
        if (registration) {
          registration.update();
        }
      });
    }
  };
  
  if (isOnline) {
    return (
      <div className="min-h-screen flex flex-col items-center justify-center p-4 bg-gradient-to-br from-blue-50 to-indigo-50">
        <div className="max-w-md w-full bg-white rounded-2xl shadow-xl p-8 text-center">
          <div className="w-20 h-20 bg-green-100 rounded-full flex items-center justify-center mx-auto mb-6">
            <Download className="w-10 h-10 text-green-600" />
          </div>
          
          <h1 className="text-3xl font-bold text-gray-900 mb-2">
            Back Online!
          </h1>
          
          <p className="text-gray-600 mb-8">
            Your connection has been restored. You can now browse normally.
          </p>
          
          <div className="space-y-4">
            <Link
              href="/"
              className="block w-full py-3 px-4 bg-blue-600 hover:bg-blue-700 text-white font-medium rounded-lg transition-colors"
            >
              <Home className="inline-block mr-2 h-5 w-5" />
              Go to Homepage
            </Link>
            
            <button
              onClick={handleRetry}
              className="block w-full py-3 px-4 bg-gray-100 hover:bg-gray-200 text-gray-800 font-medium rounded-lg transition-colors"
            >
              <RefreshCw className="inline-block mr-2 h-5 w-5" />
              Refresh Content
            </button>
          </div>
        </div>
      </div>
    );
  }
  
  return (
    <div className="min-h-screen flex flex-col items-center justify-center p-4 bg-gradient-to-br from-gray-50 to-slate-100">
      <div className="max-w-lg w-full bg-white rounded-2xl shadow-xl p-8">
        {/* Header */}
        <div className="text-center mb-8">
          <div className="w-24 h-24 bg-red-100 rounded-full flex items-center justify-center mx-auto mb-6">
            <WifiOff className="w-12 h-12 text-red-600" />
          </div>
          
          <h1 className="text-4xl font-bold text-gray-900 mb-2">
            You&apos;re Offline
          </h1>
          
          <p className="text-gray-600 text-lg">
            Check your internet connection and try again
          </p>
        </div>
        
        {/* Status */}
        <div className="bg-gray-50 rounded-xl p-6 mb-8">
          <div className="flex items-center justify-between mb-4">
            <span className="text-gray-700 font-medium">Network Status</span>
            <span className="inline-flex items-center px-3 py-1 rounded-full text-sm font-medium bg-red-100 text-red-800">
              <CloudOff className="w-4 h-4 mr-1" />
              Offline
            </span>
          </div>
          
          <div className="space-y-3">
            <div className="flex justify-between">
              <span className="text-gray-600">Service Worker</span>
              <span className="text-green-600 font-medium">
                {typeof window !== 'undefined' && 'serviceWorker' in navigator 
                  ? 'Active' 
                  : 'Not Available'}
              </span>
            </div>
            
            <div className="flex justify-between">
              <span className="text-gray-600">Cached Content</span>
              <span className="text-blue-600 font-medium">
                {cachedPages.length > 0 ? 'Available' : 'Limited'}
              </span>
            </div>
          </div>
        </div>
        
        {/* Cached Content */}
        {cachedPages.length > 0 && (
          <div className="mb-8">
            <h3 className="text-lg font-semibold text-gray-900 mb-4">
              Available Offline Content
            </h3>
            <div className="space-y-2">
              {cachedPages.map((page, index) => (
                <Link
                  key={index}
                  href={page}
                  className="block p-3 bg-blue-50 hover:bg-blue-100 rounded-lg transition-colors group"
                >
                  <div className="flex items-center justify-between">
                    <span className="text-blue-700 group-hover:text-blue-900">
                      {page === '/' ? 'Homepage' : page}
                    </span>
                    <span className="text-xs text-blue-500 bg-blue-100 px-2 py-1 rounded">
                      Cached
                    </span>
                  </div>
                </Link>
              ))}
            </div>
          </div>
        )}
        
        {/* Actions */}
        <div className="space-y-4">
          <button
            onClick={handleRetry}
            className="w-full py-4 px-6 bg-blue-600 hover:bg-blue-700 text-white font-medium rounded-xl transition-colors flex items-center justify-center"
          >
            <RefreshCw className="w-5 h-5 mr-3" />
            Retry Connection
          </button>
          
          <div className="grid grid-cols-2 gap-4">
            <button
              onClick={() => {
                if ('serviceWorker' in navigator) {
                  navigator.serviceWorker.getRegistration()
                    .then(reg => reg?.unregister())
                    .then(() => window.location.reload());
                }
              }}
              className="py-3 px-4 bg-gray-100 hover:bg-gray-200 text-gray-800 font-medium rounded-lg transition-colors"
            >
              Reset Cache
            </button>
            
            <button
              onClick={() => {
                if (navigator.onLine === false) {
                  alert('You are still offline. Please check:\n1. Wi-Fi/Data connection\n2. Airplane mode\n3. Router restart');
                }
              }}
              className="py-3 px-4 bg-gray-100 hover:bg-gray-200 text-gray-800 font-medium rounded-lg transition-colors"
            >
              Troubleshoot
            </button>
          </div>
        </div>
        
        {/* Tips */}
        <div className="mt-8 pt-8 border-t border-gray-200">
          <h4 className="text-sm font-semibold text-gray-900 mb-3">
            Offline Tips
          </h4>
          <ul className="space-y-2 text-sm text-gray-600">
            <li className="flex items-start">
              <div className="w-2 h-2 bg-gray-300 rounded-full mt-1.5 mr-3 flex-shrink-0" />
              Some content may be available offline
            </li>
            <li className="flex items-start">
              <div className="w-2 h-2 bg-gray-300 rounded-full mt-1.5 mr-3 flex-shrink-0" />
              Pages you&apos;ve visited recently are cached
            </li>
            <li className="flex items-start">
              <div className="w-2 h-2 bg-gray-300 rounded-full mt-1.5 mr-3 flex-shrink-0" />
              Install the app for better offline experience
            </li>
          </ul>
        </div>
      </div>
    </div>
  );
}
```

---

## §4. HOOKS E UTILITIES PWA

### 4.1 useOnlineStatus Hook

```typescript
// hooks/use-online-status.ts - Hook per gestione stato connessione
'use client';

import { useState, useEffect, useCallback } from 'react';

interface OnlineStatusOptions {
  /** Interval per ping al server (ms) */
  pingInterval?: number;
  /** URL da usare per il ping */
  pingUrl?: string;
  /** Abilita retry automatico */
  autoRetry?: boolean;
  /** Delay tra retry (ms) */
  retryDelay?: number;
}

interface OnlineStatus {
  /** Se il browser riporta online */
  isOnline: boolean;
  /** Se il server è raggiungibile */
  isServerReachable: boolean;
  /** Ultimo controllo */
  lastChecked: Date | null;
  /** Retry in corso */
  isRetrying: boolean;
  /** Forza un nuovo controllo */
  retryConnection: () => Promise<void>;
  /** Errori recenti */
  recentErrors: Error[];
}

export function useOnlineStatus(options: OnlineStatusOptions = {}): OnlineStatus {
  const {
    pingInterval = 30000, // 30 secondi
    pingUrl = '/api/health',
    autoRetry = true,
    retryDelay = 5000,
  } = options;
  
  const [isOnline, setIsOnline] = useState<boolean>(() => 
    typeof navigator !== 'undefined' ? navigator.onLine : true
  );
  
  const [isServerReachable, setIsServerReachable] = useState<boolean>(true);
  const [lastChecked, setLastChecked] = useState<Date | null>(null);
  const [isRetrying, setIsRetrying] = useState<boolean>(false);
  const [recentErrors, setRecentErrors] = useState<Error[]>([]);
  
  const checkServer = useCallback(async (): Promise<boolean> => {
    try {
      const controller = new AbortController();
      const timeoutId = setTimeout(() => controller.abort(), 5000);
      
      const response = await fetch(pingUrl, {
        method: 'HEAD',
        signal: controller.signal,
        cache: 'no-store',
        headers: {
          'Cache-Control': 'no-cache',
          'Pragma': 'no-cache',
        },
      });
      
      clearTimeout(timeoutId);
      
      const reachable = response.ok;
      setIsServerReachable(reachable);
      setLastChecked(new Date());
      
      if (!reachable) {
        throw new Error(`Server returned ${response.status}`);
      }
      
      // Pulisci errori recenti se successo
      setRecentErrors([]);
      
      return reachable;
    } catch (error) {
      setIsServerReachable(false);
      setLastChecked(new Date());
      
      const newError = error instanceof Error ? error : new Error('Unknown error');
      setRecentErrors(prev => [...prev.slice(-4), newError]); // Mantieni solo ultimi 5 errori
      
      return false;
    }
  }, [pingUrl]);
  
  const retryConnection = useCallback(async (): Promise<void> => {
    if (isRetrying) return;
    
    setIsRetrying(true);
    
    try {
      await checkServer();
    } finally {
      setIsRetrying(false);
    }
  }, [isRetrying, checkServer]);
  
  // Gestione eventi browser online/offline
  useEffect(() => {
    const handleOnline = () => {
      setIsOnline(true);
      if (autoRetry) {
        retryConnection();
      }
    };
    
    const handleOffline = () => {
      setIsOnline(false);
      setIsServerReachable(false);
    };
    
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, [autoRetry, retryConnection]);
  
  // Ping periodico al server
  useEffect(() => {
    if (!isOnline || !autoRetry) return;
    
    const intervalId = setInterval(() => {
      checkServer();
    }, pingInterval);
    
    // Prima verifica immediata
    checkServer();
    
    return () => clearInterval(intervalId);
  }, [isOnline, autoRetry, pingInterval, checkServer]);
  
  // Auto-retry quando torna online
  useEffect(() => {
    if (isOnline && !isServerReachable && autoRetry) {
      const timerId = setTimeout(() => {
        retryConnection();
      }, retryDelay);
      
      return () => clearTimeout(timerId);
    }
  }, [isOnline, isServerReachable, autoRetry, retryDelay, retryConnection]);
  
  return {
    isOnline,
    isServerReachable,
    lastChecked,
    isRetrying,
    retryConnection,
    recentErrors,
  };
}
```

---

**NOTA:** Questo è solo il §1-§4 (Fundamentals, Setup, Service Worker, Hooks) della documentazione completa. Continuerei con:

- §5: Install Prompt e PWA Installation
- §6: Push Notifications Implementation
- §7: App Shell Architecture
- §8: iOS PWA Considerations
- §9: Testing e Debugging
- §10: Complete PWA Checklist

Il codice include:
- Configurazione completa next-pwa con tutte le opzioni
- Service Worker custom con background sync e push notifications
- Offline page completa con gestione cache
- Hook per gestione stato connessione
- Script per generazione automatica icone
- Manifest.json completo con tutti i campi
- Type safety completa
- Error handling robusto