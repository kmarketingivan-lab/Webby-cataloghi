# CATALOGO PWA-OFFLINE v1
§ DOCUMENTAZIONE TECNICA COMPLETA PER PWA E OFFLINE CAPABILITIES IN NEXT.JS

---

§ §1. PWA FUNDAMENTALS

§ **TABELLA FUNZIONALITÀ PWA COMPLETE**

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

§ **PWA CHECKLIST PER NEXT.JS**

typescript
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

---

§ §2. NEXT.JS PWA SETUP

§ 2.1 NEXT-PWA CONFIGURATION COMPLETA

typescript
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

§ 2.2 WEB APP MANIFEST COMPLETO

json
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

§ 2.3 ICON GENERATION SCRIPT

typescript
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

---

§ §3. SERVICE WORKER E OFFLINE SUPPORT

§ 3.1 CUSTOM SERVICE WORKER CON OFFLINE SUPPORT AVANZATO

typescript
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

§ 3.2 OFFLINE PAGE COMPONENT

typescript
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

---

§ §4. HOOKS E UTILITIES PWA

§ 4.1 USEONLINESTATUS HOOK

typescript
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


---

## PWA-SERVICE-WORKER-COMPLETE

### Panoramica
Implementazione completa del Service Worker per PWA con caching strategies, background sync, push notifications e offline fallback pages.

### Implementazione Completa

```typescript
// public/sw.ts — Service Worker con Workbox
import { precacheAndRoute, cleanupOutdatedCaches } from "workbox-precaching";
import { registerRoute, NavigationRoute, Route } from "workbox-routing";
import {
  StaleWhileRevalidate,
  CacheFirst,
  NetworkFirst,
  NetworkOnly,
} from "workbox-strategies";
import { ExpirationPlugin } from "workbox-expiration";
import { CacheableResponsePlugin } from "workbox-cacheable-response";
import { BackgroundSyncPlugin } from "workbox-background-sync";

declare const self: ServiceWorkerGlobalScope;

// ============================================================
// PRECACHE STATIC ASSETS
// ============================================================
precacheAndRoute(self.__WB_MANIFEST);
cleanupOutdatedCaches();

// ============================================================
// NAVIGATION — NetworkFirst with offline fallback
// ============================================================
const navigationHandler = new NetworkFirst({
  cacheName: "pages-cache",
  networkTimeoutSeconds: 3,
  plugins: [
    new CacheableResponsePlugin({ statuses: [0, 200] }),
    new ExpirationPlugin({ maxEntries: 50, maxAgeSeconds: 24 * 60 * 60 }),
  ],
});

const navigationRoute = new NavigationRoute(navigationHandler, {
  denylist: [/\/api\//, /\/_next\//, /\/admin\//],
});
registerRoute(navigationRoute);

// ============================================================
// STATIC ASSETS — CacheFirst
// ============================================================
registerRoute(
  ({ request }) =>
    request.destination === "style" ||
    request.destination === "script" ||
    request.destination === "font",
  new CacheFirst({
    cacheName: "static-assets",
    plugins: [
      new CacheableResponsePlugin({ statuses: [0, 200] }),
      new ExpirationPlugin({
        maxEntries: 100,
        maxAgeSeconds: 30 * 24 * 60 * 60,
      }),
    ],
  })
);

// ============================================================
// IMAGES — CacheFirst with size limit
// ============================================================
registerRoute(
  ({ request }) => request.destination === "image",
  new CacheFirst({
    cacheName: "images-cache",
    plugins: [
      new CacheableResponsePlugin({ statuses: [0, 200] }),
      new ExpirationPlugin({
        maxEntries: 200,
        maxAgeSeconds: 7 * 24 * 60 * 60,
        purgeOnQuotaError: true,
      }),
    ],
  })
);

// ============================================================
// API CALLS — NetworkFirst with cache fallback
// ============================================================
registerRoute(
  ({ url }) => url.pathname.startsWith("/api/") && !url.pathname.includes("/auth"),
  new NetworkFirst({
    cacheName: "api-cache",
    networkTimeoutSeconds: 5,
    plugins: [
      new CacheableResponsePlugin({ statuses: [0, 200] }),
      new ExpirationPlugin({
        maxEntries: 100,
        maxAgeSeconds: 5 * 60,
      }),
    ],
  })
);

// ============================================================
// BACKGROUND SYNC — Queue failed mutations
// ============================================================
const bgSyncPlugin = new BackgroundSyncPlugin("mutation-queue", {
  maxRetentionTime: 24 * 60, // 24 hours
  onSync: async ({ queue }) => {
    let entry;
    while ((entry = await queue.shiftRequest())) {
      try {
        await fetch(entry.request.clone());
        console.log("[SW] Background sync successful:", entry.request.url);
      } catch (error) {
        console.error("[SW] Background sync failed:", error);
        await queue.unshiftRequest(entry);
        throw error;
      }
    }
  },
});

registerRoute(
  ({ url, request }) =>
    url.pathname.startsWith("/api/") &&
    (request.method === "POST" || request.method === "PUT" || request.method === "PATCH"),
  new NetworkOnly({ plugins: [bgSyncPlugin] }),
  "POST"
);

registerRoute(
  ({ url, request }) =>
    url.pathname.startsWith("/api/") &&
    (request.method === "POST" || request.method === "PUT" || request.method === "PATCH"),
  new NetworkOnly({ plugins: [bgSyncPlugin] }),
  "PUT"
);

// ============================================================
// PUSH NOTIFICATIONS
// ============================================================
self.addEventListener("push", (event) => {
  const data = event.data?.json() ?? {
    title: "New Notification",
    body: "You have a new notification",
    icon: "/icons/icon-192.png",
    badge: "/icons/badge-72.png",
  };

  event.waitUntil(
    self.registration.showNotification(data.title, {
      body: data.body,
      icon: data.icon ?? "/icons/icon-192.png",
      badge: data.badge ?? "/icons/badge-72.png",
      image: data.image,
      data: data.data,
      actions: data.actions ?? [
        { action: "open", title: "Open" },
        { action: "dismiss", title: "Dismiss" },
      ],
      tag: data.tag ?? "default",
      renotify: data.renotify ?? false,
      vibrate: [200, 100, 200],
    })
  );
});

self.addEventListener("notificationclick", (event) => {
  event.notification.close();

  const url = event.notification.data?.url ?? "/";

  if (event.action === "dismiss") return;

  event.waitUntil(
    self.clients.matchAll({ type: "window" }).then((clients) => {
      const existingClient = clients.find(
        (c) => c.url === url && "focus" in c
      );

      if (existingClient) {
        return existingClient.focus();
      }

      return self.clients.openWindow(url);
    })
  );
});

// ============================================================
// OFFLINE FALLBACK
// ============================================================
self.addEventListener("fetch", (event) => {
  if (event.request.mode === "navigate") {
    event.respondWith(
      fetch(event.request).catch(() =>
        caches.match("/offline") ?? new Response("Offline", { status: 503 })
      )
    );
  }
});
```

### Varianti e Configurazioni

```typescript
// lib/pwa/install-prompt.tsx
"use client";

import { useState, useEffect, useCallback } from "react";
import { Button } from "@/components/ui/button";
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { Download, X, Smartphone } from "lucide-react";

// ============================================================
// INSTALL PROMPT HOOK
// ============================================================
interface BeforeInstallPromptEvent extends Event {
  prompt: () => Promise<void>;
  userChoice: Promise<{ outcome: "accepted" | "dismissed" }>;
}

export function useInstallPrompt() {
  const [deferredPrompt, setDeferredPrompt] =
    useState<BeforeInstallPromptEvent | null>(null);
  const [isInstalled, setIsInstalled] = useState(false);
  const [isIOS, setIsIOS] = useState(false);

  useEffect(() => {
    // Check if already installed
    if (window.matchMedia("(display-mode: standalone)").matches) {
      setIsInstalled(true);
      return;
    }

    // iOS detection
    const ua = navigator.userAgent;
    setIsIOS(/iPad|iPhone|iPod/.test(ua) && !(window as any).MSStream);

    const handler = (e: Event) => {
      e.preventDefault();
      setDeferredPrompt(e as BeforeInstallPromptEvent);
    };

    window.addEventListener("beforeinstallprompt", handler);
    window.addEventListener("appinstalled", () => setIsInstalled(true));

    return () => window.removeEventListener("beforeinstallprompt", handler);
  }, []);

  const install = useCallback(async () => {
    if (!deferredPrompt) return false;

    await deferredPrompt.prompt();
    const { outcome } = await deferredPrompt.userChoice;
    setDeferredPrompt(null);
    return outcome === "accepted";
  }, [deferredPrompt]);

  return {
    canInstall: !!deferredPrompt,
    isInstalled,
    isIOS,
    install,
  };
}

// ============================================================
// INSTALL BANNER
// ============================================================
export function InstallBanner() {
  const { canInstall, isInstalled, isIOS, install } = useInstallPrompt();
  const [dismissed, setDismissed] = useState(false);

  if (isInstalled || dismissed) return null;
  if (!canInstall && !isIOS) return null;

  return (
    <Card className="fixed bottom-4 left-4 right-4 z-50 mx-auto max-w-lg shadow-lg md:left-auto md:right-4 md:w-96">
      <CardHeader className="pb-2 relative">
        <button
          onClick={() => setDismissed(true)}
          className="absolute right-4 top-4 text-muted-foreground hover:text-foreground"
        >
          <X className="h-4 w-4" />
        </button>
        <CardTitle className="flex items-center gap-2 text-base">
          <Smartphone className="h-5 w-5" />
          Install App
        </CardTitle>
        <CardDescription className="text-xs">
          Get a faster, native-like experience
        </CardDescription>
      </CardHeader>
      <CardContent className="pb-4">
        {isIOS ? (
          <p className="text-sm text-muted-foreground">
            Tap the share button{" "}
            <span className="inline-block rotate-90">⎋</span> then
            &quot;Add to Home Screen&quot;
          </p>
        ) : (
          <Button
            onClick={install}
            size="sm"
            className="w-full"
          >
            <Download className="mr-2 h-4 w-4" />
            Install
          </Button>
        )}
      </CardContent>
    </Card>
  );
}
```

```typescript
// lib/pwa/offline-indicator.tsx
"use client";

import { useState, useEffect } from "react";
import { WifiOff, Wifi } from "lucide-react";
import { cn } from "@/lib/utils";
import { motion, AnimatePresence } from "framer-motion";

export function OfflineIndicator() {
  const [isOnline, setIsOnline] = useState(true);
  const [showReconnected, setShowReconnected] = useState(false);

  useEffect(() => {
    const handleOnline = () => {
      setIsOnline(true);
      setShowReconnected(true);
      setTimeout(() => setShowReconnected(false), 3000);
    };

    const handleOffline = () => {
      setIsOnline(false);
      setShowReconnected(false);
    };

    setIsOnline(navigator.onLine);

    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);

    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);

  return (
    <AnimatePresence>
      {(!isOnline || showReconnected) && (
        <motion.div
          initial={{ y: -50, opacity: 0 }}
          animate={{ y: 0, opacity: 1 }}
          exit={{ y: -50, opacity: 0 }}
          className={cn(
            "fixed top-0 left-0 right-0 z-[9999] flex items-center justify-center gap-2 py-2 text-sm font-medium",
            isOnline
              ? "bg-green-500 text-white"
              : "bg-yellow-500 text-black"
          )}
        >
          {isOnline ? (
            <>
              <Wifi className="h-4 w-4" />
              Back online
            </>
          ) : (
            <>
              <WifiOff className="h-4 w-4" />
              You are offline — changes will sync when reconnected
            </>
          )}
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

### Edge Cases e Error Handling

```typescript
// lib/pwa/push-subscription.ts
"use client";

// ============================================================
// PUSH NOTIFICATION SUBSCRIPTION MANAGEMENT
// ============================================================
export async function subscribeToPush(): Promise<PushSubscription | null> {
  if (!("serviceWorker" in navigator) || !("PushManager" in window)) {
    console.warn("Push notifications not supported");
    return null;
  }

  const permission = await Notification.requestPermission();
  if (permission !== "granted") {
    console.warn("Notification permission denied");
    return null;
  }

  const registration = await navigator.serviceWorker.ready;

  const existingSubscription = await registration.pushManager.getSubscription();
  if (existingSubscription) {
    return existingSubscription;
  }

  const vapidPublicKey = process.env.NEXT_PUBLIC_VAPID_PUBLIC_KEY;
  if (!vapidPublicKey) {
    console.error("VAPID public key not set");
    return null;
  }

  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: urlBase64ToUint8Array(vapidPublicKey),
  });

  // Send subscription to server
  await fetch("/api/push/subscribe", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(subscription.toJSON()),
  });

  return subscription;
}

export async function unsubscribeFromPush(): Promise<boolean> {
  const registration = await navigator.serviceWorker.ready;
  const subscription = await registration.pushManager.getSubscription();

  if (subscription) {
    await fetch("/api/push/unsubscribe", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ endpoint: subscription.endpoint }),
    });

    return subscription.unsubscribe();
  }

  return false;
}

function urlBase64ToUint8Array(base64String: string): Uint8Array {
  const padding = "=".repeat((4 - (base64String.length % 4)) % 4);
  const base64 = (base64String + padding).replace(/-/g, "+").replace(/_/g, "/");
  const rawData = atob(base64);
  const outputArray = new Uint8Array(rawData.length);
  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i);
  }
  return outputArray;
}

// ============================================================
// CACHE SIZE MANAGEMENT
// ============================================================
export async function getCacheSize(): Promise<{ used: number; quota: number }> {
  if (!("storage" in navigator) || !("estimate" in navigator.storage)) {
    return { used: 0, quota: 0 };
  }

  const estimate = await navigator.storage.estimate();
  return {
    used: estimate.usage ?? 0,
    quota: estimate.quota ?? 0,
  };
}

export async function clearAppCache(): Promise<void> {
  const cacheNames = await caches.keys();
  await Promise.all(cacheNames.map((name) => caches.delete(name)));
}

// ============================================================
// UPDATE PROMPT
// ============================================================
export function useServiceWorkerUpdate() {
  const [updateAvailable, setUpdateAvailable] = useState(false);
  const [registration, setRegistration] = useState<ServiceWorkerRegistration | null>(null);

  useEffect(() => {
    if (!("serviceWorker" in navigator)) return;

    navigator.serviceWorker.ready.then((reg) => {
      setRegistration(reg);

      reg.addEventListener("updatefound", () => {
        const newWorker = reg.installing;
        if (!newWorker) return;

        newWorker.addEventListener("statechange", () => {
          if (newWorker.state === "installed" && navigator.serviceWorker.controller) {
            setUpdateAvailable(true);
          }
        });
      });
    });
  }, []);

  const applyUpdate = () => {
    if (registration?.waiting) {
      registration.waiting.postMessage({ type: "SKIP_WAITING" });
      window.location.reload();
    }
  };

  return { updateAvailable, applyUpdate };
}

import { useState, useEffect } from "react";
```

### Errori Comuni da Evitare
- **Cache stale in development**: Disabilita il SW in development o usa `skipWaiting`
- **Missing HTTPS**: Service Workers funzionano solo su HTTPS (eccetto localhost)
- **Background sync non supportato**: Verifica `'SyncManager' in window` prima di usarlo
- **Push senza VAPID**: Le push notifications richiedono chiavi VAPID configurate

### Checklist di Verifica
- [ ] Il SW precache le risorse statiche essenziali
- [ ] Le pagine usano NetworkFirst con timeout di 3s
- [ ] Le immagini usano CacheFirst con limite di dimensione
- [ ] Il background sync riprova le mutation fallite
- [ ] Le push notification hanno azioni configurabili
- [ ] L'offline indicator mostra lo stato di connessione
- [ ] L'install prompt funziona su Chrome, Edge e iOS
- [ ] L'update prompt notifica gli aggiornamenti disponibili



---

## OFFLINE-DATA-SYNC

### Panoramica
Pattern per sincronizzazione dati offline con IndexedDB, conflict resolution, queue di mutazioni offline e sync automatico al ripristino della connessione.

### Implementazione Completa

```typescript
// lib/pwa/offline-store.ts
import Dexie, { type Table } from "dexie";

// ============================================================
// INDEXEDDB SCHEMA
// ============================================================
interface OfflineItem {
  id: string;
  type: string;
  data: Record<string, unknown>;
  syncStatus: "synced" | "pending" | "conflict" | "error";
  localUpdatedAt: number;
  serverUpdatedAt?: number;
  version: number;
}

interface PendingMutation {
  id: string;
  type: "create" | "update" | "delete";
  entityType: string;
  entityId: string;
  payload: Record<string, unknown>;
  createdAt: number;
  retryCount: number;
  maxRetries: number;
  error?: string;
}

class OfflineDatabase extends Dexie {
  items!: Table<OfflineItem>;
  mutations!: Table<PendingMutation>;
  metadata!: Table<{ key: string; value: unknown }>;

  constructor() {
    super("offline-store");

    this.version(1).stores({
      items: "id, type, syncStatus, localUpdatedAt",
      mutations: "id, entityType, entityId, createdAt",
      metadata: "key",
    });
  }
}

export const offlineDB = new OfflineDatabase();

// ============================================================
// OFFLINE DATA MANAGER
// ============================================================
export class OfflineDataManager {
  // ----------------------------------------------------------
  // SAVE DATA LOCALLY
  // ----------------------------------------------------------
  async saveLocal(
    type: string,
    id: string,
    data: Record<string, unknown>
  ): Promise<void> {
    const existing = await offlineDB.items.get(id);

    await offlineDB.items.put({
      id,
      type,
      data,
      syncStatus: "pending",
      localUpdatedAt: Date.now(),
      serverUpdatedAt: existing?.serverUpdatedAt,
      version: (existing?.version ?? 0) + 1,
    });

    // Queue mutation for sync
    await offlineDB.mutations.add({
      id: crypto.randomUUID(),
      type: existing ? "update" : "create",
      entityType: type,
      entityId: id,
      payload: data,
      createdAt: Date.now(),
      retryCount: 0,
      maxRetries: 5,
    });
  }

  // ----------------------------------------------------------
  // DELETE LOCALLY
  // ----------------------------------------------------------
  async deleteLocal(type: string, id: string): Promise<void> {
    await offlineDB.items.update(id, { syncStatus: "pending" });

    await offlineDB.mutations.add({
      id: crypto.randomUUID(),
      type: "delete",
      entityType: type,
      entityId: id,
      payload: {},
      createdAt: Date.now(),
      retryCount: 0,
      maxRetries: 5,
    });
  }

  // ----------------------------------------------------------
  // GET LOCAL DATA
  // ----------------------------------------------------------
  async getLocal<T>(id: string): Promise<T | null> {
    const item = await offlineDB.items.get(id);
    return item ? (item.data as T) : null;
  }

  async getAllLocal<T>(type: string): Promise<T[]> {
    const items = await offlineDB.items
      .where("type")
      .equals(type)
      .toArray();
    return items.map((i) => i.data as T);
  }

  // ----------------------------------------------------------
  // SYNC WITH SERVER
  // ----------------------------------------------------------
  async sync(): Promise<{
    synced: number;
    failed: number;
    conflicts: number;
  }> {
    let synced = 0;
    let failed = 0;
    let conflicts = 0;

    const mutations = await offlineDB.mutations
      .orderBy("createdAt")
      .toArray();

    for (const mutation of mutations) {
      try {
        const result = await this.processMutation(mutation);

        if (result.status === "success") {
          await offlineDB.mutations.delete(mutation.id);
          await offlineDB.items.update(mutation.entityId, {
            syncStatus: "synced",
            serverUpdatedAt: Date.now(),
          });
          synced++;
        } else if (result.status === "conflict") {
          await offlineDB.items.update(mutation.entityId, {
            syncStatus: "conflict",
          });
          conflicts++;
        } else {
          mutation.retryCount++;
          mutation.error = result.error;

          if (mutation.retryCount >= mutation.maxRetries) {
            await offlineDB.items.update(mutation.entityId, {
              syncStatus: "error",
            });
            await offlineDB.mutations.delete(mutation.id);
            failed++;
          } else {
            await offlineDB.mutations.update(mutation.id, mutation);
          }
        }
      } catch (error) {
        failed++;
      }
    }

    return { synced, failed, conflicts };
  }

  // ----------------------------------------------------------
  // PROCESS SINGLE MUTATION
  // ----------------------------------------------------------
  private async processMutation(
    mutation: PendingMutation
  ): Promise<{ status: "success" | "conflict" | "error"; error?: string }> {
    const endpoint = `/api/${mutation.entityType}${
      mutation.type !== "create" ? `/${mutation.entityId}` : ""
    }`;

    const method =
      mutation.type === "create"
        ? "POST"
        : mutation.type === "update"
          ? "PATCH"
          : "DELETE";

    const response = await fetch(endpoint, {
      method,
      headers: {
        "Content-Type": "application/json",
        "X-Offline-Sync": "true",
      },
      body: method !== "DELETE" ? JSON.stringify(mutation.payload) : undefined,
    });

    if (response.ok) {
      return { status: "success" };
    }

    if (response.status === 409) {
      return { status: "conflict" };
    }

    const errorBody = await response.text().catch(() => "Unknown error");
    return { status: "error", error: `${response.status}: ${errorBody}` };
  }

  // ----------------------------------------------------------
  // RESOLVE CONFLICT
  // ----------------------------------------------------------
  async resolveConflict(
    id: string,
    resolution: "keep-local" | "keep-server" | "merge",
    mergedData?: Record<string, unknown>
  ): Promise<void> {
    if (resolution === "keep-server") {
      // Fetch server version
      const item = await offlineDB.items.get(id);
      if (!item) return;

      const response = await fetch(`/api/${item.type}/${id}`);
      if (response.ok) {
        const serverData = await response.json();
        await offlineDB.items.update(id, {
          data: serverData.data,
          syncStatus: "synced",
          serverUpdatedAt: Date.now(),
        });
      }
    } else if (resolution === "keep-local") {
      // Re-push local version
      const item = await offlineDB.items.get(id);
      if (!item) return;

      await offlineDB.mutations.add({
        id: crypto.randomUUID(),
        type: "update",
        entityType: item.type,
        entityId: id,
        payload: { ...item.data, _forceOverwrite: true },
        createdAt: Date.now(),
        retryCount: 0,
        maxRetries: 3,
      });

      await offlineDB.items.update(id, { syncStatus: "pending" });
    } else if (resolution === "merge" && mergedData) {
      await offlineDB.items.update(id, {
        data: mergedData,
        syncStatus: "pending",
      });

      const item = await offlineDB.items.get(id);
      if (!item) return;

      await offlineDB.mutations.add({
        id: crypto.randomUUID(),
        type: "update",
        entityType: item.type,
        entityId: id,
        payload: mergedData,
        createdAt: Date.now(),
        retryCount: 0,
        maxRetries: 3,
      });
    }
  }

  // ----------------------------------------------------------
  // CLEAR ALL LOCAL DATA
  // ----------------------------------------------------------
  async clearAll(): Promise<void> {
    await offlineDB.items.clear();
    await offlineDB.mutations.clear();
  }

  // ----------------------------------------------------------
  // GET SYNC STATUS
  // ----------------------------------------------------------
  async getSyncStatus(): Promise<{
    pendingCount: number;
    conflictCount: number;
    errorCount: number;
    lastSyncAt: number | null;
  }> {
    const [pending, conflicts, errors] = await Promise.all([
      offlineDB.items.where("syncStatus").equals("pending").count(),
      offlineDB.items.where("syncStatus").equals("conflict").count(),
      offlineDB.items.where("syncStatus").equals("error").count(),
    ]);

    const lastSync = await offlineDB.metadata.get("lastSyncAt");

    return {
      pendingCount: pending,
      conflictCount: conflicts,
      errorCount: errors,
      lastSyncAt: lastSync?.value as number | null,
    };
  }
}

export const offlineManager = new OfflineDataManager();
```

### Varianti e Configurazioni

```typescript
// hooks/use-offline-sync.ts
"use client";

import { useState, useEffect, useCallback } from "react";
import { offlineManager } from "@/lib/pwa/offline-store";

// ============================================================
// OFFLINE SYNC HOOK
// ============================================================
interface SyncStatus {
  isOnline: boolean;
  isSyncing: boolean;
  pendingCount: number;
  conflictCount: number;
  errorCount: number;
  lastSyncAt: Date | null;
}

export function useOfflineSync(autoSyncInterval = 30_000) {
  const [status, setStatus] = useState<SyncStatus>({
    isOnline: typeof navigator !== "undefined" ? navigator.onLine : true,
    isSyncing: false,
    pendingCount: 0,
    conflictCount: 0,
    errorCount: 0,
    lastSyncAt: null,
  });

  const refreshStatus = useCallback(async () => {
    const syncStatus = await offlineManager.getSyncStatus();
    setStatus((prev) => ({
      ...prev,
      pendingCount: syncStatus.pendingCount,
      conflictCount: syncStatus.conflictCount,
      errorCount: syncStatus.errorCount,
      lastSyncAt: syncStatus.lastSyncAt
        ? new Date(syncStatus.lastSyncAt)
        : null,
    }));
  }, []);

  const sync = useCallback(async () => {
    if (!navigator.onLine || status.isSyncing) return;

    setStatus((prev) => ({ ...prev, isSyncing: true }));
    try {
      const result = await offlineManager.sync();
      console.log(
        `[Sync] Synced: ${result.synced}, Failed: ${result.failed}, Conflicts: ${result.conflicts}`
      );
      await refreshStatus();
    } finally {
      setStatus((prev) => ({ ...prev, isSyncing: false }));
    }
  }, [status.isSyncing, refreshStatus]);

  useEffect(() => {
    const handleOnline = () => {
      setStatus((prev) => ({ ...prev, isOnline: true }));
      sync();
    };
    const handleOffline = () => {
      setStatus((prev) => ({ ...prev, isOnline: false }));
    };

    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);

    refreshStatus();

    const interval = setInterval(() => {
      if (navigator.onLine) sync();
    }, autoSyncInterval);

    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
      clearInterval(interval);
    };
  }, [sync, refreshStatus, autoSyncInterval]);

  return { ...status, sync, refreshStatus };
}
```

```typescript
// components/pwa/sync-status-bar.tsx
"use client";

import { useOfflineSync } from "@/hooks/use-offline-sync";
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";
import {
  Cloud,
  CloudOff,
  RefreshCw,
  AlertTriangle,
  CheckCircle2,
  Loader2,
} from "lucide-react";
import { cn } from "@/lib/utils";
import { formatDistanceToNow } from "date-fns";

export function SyncStatusBar() {
  const {
    isOnline,
    isSyncing,
    pendingCount,
    conflictCount,
    errorCount,
    lastSyncAt,
    sync,
  } = useOfflineSync();

  const hasIssues = conflictCount > 0 || errorCount > 0;
  const hasPending = pendingCount > 0;

  return (
    <div className="flex items-center gap-3 text-sm">
      {/* Connection status */}
      <div className="flex items-center gap-1.5">
        {isOnline ? (
          <Cloud className="h-4 w-4 text-green-500" />
        ) : (
          <CloudOff className="h-4 w-4 text-yellow-500" />
        )}
        <span className={cn("text-xs", isOnline ? "text-green-600" : "text-yellow-600")}>
          {isOnline ? "Online" : "Offline"}
        </span>
      </div>

      {/* Pending changes */}
      {hasPending && (
        <Badge variant="secondary" className="gap-1">
          <RefreshCw className="h-3 w-3" />
          {pendingCount} pending
        </Badge>
      )}

      {/* Conflicts */}
      {conflictCount > 0 && (
        <Badge variant="destructive" className="gap-1">
          <AlertTriangle className="h-3 w-3" />
          {conflictCount} conflicts
        </Badge>
      )}

      {/* Sync button */}
      {isOnline && (hasPending || hasIssues) && (
        <Button
          variant="ghost"
          size="sm"
          onClick={sync}
          disabled={isSyncing}
          className="h-7 gap-1 px-2"
        >
          {isSyncing ? (
            <Loader2 className="h-3 w-3 animate-spin" />
          ) : (
            <RefreshCw className="h-3 w-3" />
          )}
          Sync
        </Button>
      )}

      {/* Last sync time */}
      {!hasPending && !hasIssues && lastSyncAt && (
        <span className="flex items-center gap-1 text-xs text-muted-foreground">
          <CheckCircle2 className="h-3 w-3 text-green-500" />
          Synced {formatDistanceToNow(lastSyncAt, { addSuffix: true })}
        </span>
      )}
    </div>
  );
}
```

### Edge Cases e Error Handling

```typescript
// lib/pwa/manifest-generator.ts

// ============================================================
// WEB APP MANIFEST
// ============================================================
export interface ManifestConfig {
  name: string;
  shortName: string;
  description: string;
  themeColor: string;
  backgroundColor: string;
  display: "standalone" | "fullscreen" | "minimal-ui" | "browser";
  orientation?: "portrait" | "landscape" | "any";
  startUrl?: string;
  scope?: string;
  categories?: string[];
}

export function generateManifest(config: ManifestConfig) {
  return {
    name: config.name,
    short_name: config.shortName,
    description: config.description,
    theme_color: config.themeColor,
    background_color: config.backgroundColor,
    display: config.display,
    orientation: config.orientation ?? "any",
    start_url: config.startUrl ?? "/",
    scope: config.scope ?? "/",
    categories: config.categories ?? [],
    icons: [
      { src: "/icons/icon-72.png", sizes: "72x72", type: "image/png" },
      { src: "/icons/icon-96.png", sizes: "96x96", type: "image/png" },
      { src: "/icons/icon-128.png", sizes: "128x128", type: "image/png" },
      { src: "/icons/icon-144.png", sizes: "144x144", type: "image/png" },
      { src: "/icons/icon-152.png", sizes: "152x152", type: "image/png" },
      { src: "/icons/icon-192.png", sizes: "192x192", type: "image/png" },
      { src: "/icons/icon-384.png", sizes: "384x384", type: "image/png" },
      { src: "/icons/icon-512.png", sizes: "512x512", type: "image/png" },
      { src: "/icons/icon-512-maskable.png", sizes: "512x512", type: "image/png", purpose: "maskable" },
    ],
    screenshots: [
      {
        src: "/screenshots/desktop.png",
        sizes: "1280x720",
        type: "image/png",
        form_factor: "wide",
        label: "Desktop view",
      },
      {
        src: "/screenshots/mobile.png",
        sizes: "750x1334",
        type: "image/png",
        form_factor: "narrow",
        label: "Mobile view",
      },
    ],
    shortcuts: [
      {
        name: "Dashboard",
        short_name: "Dashboard",
        url: "/dashboard",
        icons: [{ src: "/icons/shortcut-dashboard.png", sizes: "96x96" }],
      },
      {
        name: "New Item",
        short_name: "New",
        url: "/new",
        icons: [{ src: "/icons/shortcut-new.png", sizes: "96x96" }],
      },
    ],
    share_target: {
      action: "/share-target",
      method: "POST",
      enctype: "multipart/form-data",
      params: {
        title: "title",
        text: "text",
        url: "url",
        files: [
          { name: "media", accept: ["image/*", "video/*"] },
        ],
      },
    },
  };
}
```

### Errori Comuni da Evitare
- **IndexedDB quota exceeded**: Gestisci `QuotaExceededError` e pulisci dati vecchi
- **Conflict resolution silente**: Mostra sempre i conflitti all'utente per scelta manuale
- **Sync loop**: Non sincronizzare se gia in corso (`isSyncing` check)
- **Missing maskable icon**: Android richiede un'icona con `purpose: "maskable"` per adattarsi

### Checklist di Verifica
- [ ] Il database offline usa Dexie.js con schema versionato
- [ ] Le mutazioni pending vengono sincronizzate al ritorno online
- [ ] I conflitti mostrano opzioni: keep local, keep server, merge
- [ ] Lo status bar mostra pending, conflicts e ultimo sync
- [ ] Il manifest ha tutte le dimensioni di icona necessarie
- [ ] L'icona maskable ha safe area sufficiente
- [ ] Lo share target funziona per condivisione da altre app



---

## PWA-PERFORMANCE-OPTIMIZATION

### Panoramica
Ottimizzazioni PWA per performance: app shell pattern, resource hints, cache warming, lazy loading con prefetch e metrics di performance.

### Implementazione Completa

```typescript
// lib/pwa/app-shell.tsx
"use client";

import { Suspense, ReactNode } from "react";
import dynamic from "next/dynamic";

// ============================================================
// APP SHELL PATTERN — cached shell, dynamic content
// ============================================================
interface AppShellProps {
  header: ReactNode;
  sidebar?: ReactNode;
  children: ReactNode;
  footer?: ReactNode;
}

export function AppShell({ header, sidebar, children, footer }: AppShellProps) {
  return (
    <div className="flex min-h-screen flex-col">
      {/* Static header — always cached */}
      <header className="sticky top-0 z-50 border-b bg-background">{header}</header>

      <div className="flex flex-1">
        {/* Static sidebar — always cached */}
        {sidebar && (
          <aside className="hidden w-64 shrink-0 border-r lg:block">{sidebar}</aside>
        )}

        {/* Dynamic content — loaded from network */}
        <main className="flex-1 overflow-auto">
          <Suspense
            fallback={
              <div className="flex items-center justify-center p-8">
                <div className="h-8 w-8 animate-spin rounded-full border-4 border-primary border-t-transparent" />
              </div>
            }
          >
            {children}
          </Suspense>
        </main>
      </div>

      {/* Static footer — always cached */}
      {footer && <footer className="border-t bg-muted/50">{footer}</footer>}
    </div>
  );
}

// ============================================================
// RESOURCE HINTS — preconnect, prefetch, preload
// ============================================================
export function ResourceHints() {
  return (
    <>
      {/* Preconnect to external origins */}
      <link rel="preconnect" href="https://fonts.googleapis.com" />
      <link rel="preconnect" href="https://fonts.gstatic.com" crossOrigin="" />
      <link rel="preconnect" href="https://cdn.example.com" />

      {/* DNS prefetch for analytics */}
      <link rel="dns-prefetch" href="https://www.google-analytics.com" />

      {/* Preload critical font */}
      <link
        rel="preload"
        href="/fonts/inter-var.woff2"
        as="font"
        type="font/woff2"
        crossOrigin="anonymous"
      />
    </>
  );
}

// ============================================================
// ROUTE PREFETCHING
// ============================================================
export function usePrefetchRoute(href: string): () => void {
  return () => {
    const link = document.createElement("link");
    link.rel = "prefetch";
    link.href = href;
    link.as = "document";
    document.head.appendChild(link);
  };
}

// ============================================================
// PERFORMANCE METRICS
// ============================================================
interface PerformanceMetrics {
  fcp: number | null;
  lcp: number | null;
  fid: number | null;
  cls: number | null;
  ttfb: number | null;
  inp: number | null;
}

export function measureWebVitals(
  onReport: (metrics: Partial<PerformanceMetrics>) => void
): void {
  if (typeof window === "undefined") return;

  // First Contentful Paint
  const fcpObserver = new PerformanceObserver((list) => {
    const entries = list.getEntriesByName("first-contentful-paint");
    if (entries.length > 0) {
      onReport({ fcp: entries[0].startTime });
    }
  });
  fcpObserver.observe({ type: "paint", buffered: true });

  // Largest Contentful Paint
  const lcpObserver = new PerformanceObserver((list) => {
    const entries = list.getEntries();
    const lastEntry = entries[entries.length - 1];
    if (lastEntry) {
      onReport({ lcp: lastEntry.startTime });
    }
  });
  lcpObserver.observe({ type: "largest-contentful-paint", buffered: true });

  // Cumulative Layout Shift
  let clsValue = 0;
  const clsObserver = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      if (!(entry as any).hadRecentInput) {
        clsValue += (entry as any).value;
      }
    }
    onReport({ cls: clsValue });
  });
  clsObserver.observe({ type: "layout-shift", buffered: true });

  // Time to First Byte
  const navEntry = performance.getEntriesByType("navigation")[0] as PerformanceNavigationTiming;
  if (navEntry) {
    onReport({ ttfb: navEntry.responseStart - navEntry.requestStart });
  }
}

// ============================================================
// CACHE WARMING ON IDLE
// ============================================================
export function warmCacheOnIdle(urls: string[]): void {
  if (typeof window === "undefined") return;

  const warm = () => {
    if ("requestIdleCallback" in window) {
      requestIdleCallback(
        () => {
          urls.forEach((url) => {
            const link = document.createElement("link");
            link.rel = "prefetch";
            link.href = url;
            document.head.appendChild(link);
          });
        },
        { timeout: 2000 }
      );
    }
  };

  if (document.readyState === "complete") {
    warm();
  } else {
    window.addEventListener("load", warm);
  }
}
```

### Varianti e Configurazioni

```typescript
// next.config.ts — PWA configuration with next-pwa
import type { NextConfig } from "next";
import withPWA from "next-pwa";

const nextConfig: NextConfig = {
  reactStrictMode: true,
  images: {
    formats: ["image/avif", "image/webp"],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048],
    imageSizes: [16, 32, 48, 64, 96, 128, 256],
  },
  experimental: {
    optimizeCss: true,
  },
  headers: async () => [
    {
      source: "/(.*)",
      headers: [
        { key: "X-Content-Type-Options", value: "nosniff" },
        { key: "X-Frame-Options", value: "DENY" },
        { key: "Referrer-Policy", value: "strict-origin-when-cross-origin" },
      ],
    },
    {
      source: "/sw.js",
      headers: [
        { key: "Cache-Control", value: "no-cache, no-store, must-revalidate" },
        { key: "Content-Type", value: "application/javascript; charset=utf-8" },
      ],
    },
    {
      source: "/_next/static/(.*)",
      headers: [
        { key: "Cache-Control", value: "public, max-age=31536000, immutable" },
      ],
    },
  ],
};

const config = withPWA({
  dest: "public",
  register: true,
  skipWaiting: true,
  disable: process.env.NODE_ENV === "development",
  runtimeCaching: [
    {
      urlPattern: /^https:\/\/fonts\.(googleapis|gstatic)\.com\/.*/i,
      handler: "CacheFirst",
      options: {
        cacheName: "google-fonts",
        expiration: { maxEntries: 20, maxAgeSeconds: 365 * 24 * 60 * 60 },
      },
    },
    {
      urlPattern: /\.(?:jpg|jpeg|gif|png|svg|ico|webp|avif)$/i,
      handler: "CacheFirst",
      options: {
        cacheName: "static-images",
        expiration: { maxEntries: 200, maxAgeSeconds: 30 * 24 * 60 * 60 },
      },
    },
    {
      urlPattern: /^https:\/\/api\..*\/v\d+\/.*/i,
      handler: "NetworkFirst",
      options: {
        cacheName: "api-responses",
        networkTimeoutSeconds: 10,
        expiration: { maxEntries: 50, maxAgeSeconds: 5 * 60 },
      },
    },
  ],
})(nextConfig);

export default config;
```

### Edge Cases e Error Handling

```typescript
// lib/pwa/storage-manager.ts
"use client";

// ============================================================
// STORAGE QUOTA MANAGEMENT
// ============================================================
export async function checkStorageQuota(): Promise<{
  used: string;
  available: string;
  percentUsed: number;
  isLow: boolean;
}> {
  if (!("storage" in navigator) || !("estimate" in navigator.storage)) {
    return { used: "N/A", available: "N/A", percentUsed: 0, isLow: false };
  }

  const estimate = await navigator.storage.estimate();
  const used = estimate.usage ?? 0;
  const quota = estimate.quota ?? 0;
  const percentUsed = quota > 0 ? (used / quota) * 100 : 0;

  return {
    used: formatBytes(used),
    available: formatBytes(quota - used),
    percentUsed: Math.round(percentUsed * 10) / 10,
    isLow: percentUsed > 80,
  };
}

function formatBytes(bytes: number): string {
  if (bytes === 0) return "0 B";
  const sizes = ["B", "KB", "MB", "GB"];
  const i = Math.floor(Math.log(bytes) / Math.log(1024));
  return `${(bytes / Math.pow(1024, i)).toFixed(1)} ${sizes[i]}`;
}

// ============================================================
// REQUEST PERSISTENT STORAGE
// ============================================================
export async function requestPersistentStorage(): Promise<boolean> {
  if (!("storage" in navigator) || !("persist" in navigator.storage)) {
    return false;
  }

  const isPersisted = await navigator.storage.persisted();
  if (isPersisted) return true;

  return navigator.storage.persist();
}

// ============================================================
// CLEANUP OLD CACHES
// ============================================================
export async function cleanupOldCaches(keepCacheNames: string[]): Promise<void> {
  const cacheNames = await caches.keys();
  await Promise.all(
    cacheNames
      .filter((name) => !keepCacheNames.includes(name))
      .map((name) => caches.delete(name))
  );
}
```

### Errori Comuni da Evitare
- **SW in development**: Disabilita il Service Worker in development per evitare caching stale
- **Missing scope**: Definisci lo scope del manifest correttamente per evitare conflitti
- **Cache-Control su sw.js**: Il SW deve avere `no-cache` per permettere aggiornamenti
- **Quota exceeded**: Monitora lo storage e pulisci cache vecchie proattivamente

### Checklist di Verifica
- [ ] L'app shell (header, sidebar, footer) e cachata separatamente dal contenuto
- [ ] I resource hints (preconnect, prefetch) sono configurati per origini esterne
- [ ] Le Web Vitals (FCP, LCP, CLS, TTFB) sono misurate e reportate
- [ ] Il cache warming carica risorse critiche durante idle time
- [ ] Il next.config include runtime caching per fonts, images e API
- [ ] Lo storage quota e monitorato con warning sopra l'80%
- [ ] Il persistent storage e richiesto per dati critici offline
