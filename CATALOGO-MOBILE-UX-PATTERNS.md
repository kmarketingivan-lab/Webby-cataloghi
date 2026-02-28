# CATALOGO-MOBILE-UX-PATTERNS

CATALOGO MOBILE UX PATTERNS - Next.js 14 + Tailwind
§ MOBILE-FIRST PRINCIPLES
Design Philosophy
tsx
// components/MobileFirstLayout.tsx
export default function MobileFirstLayout() {
  return (
    <div className="min-h-screen">
      {/* Base styles for mobile (no media queries) */}
      <main className="px-4 py-6">
        <h1 className="text-2xl font-bold">Mobile First Content</h1>
        <p className="mt-2 text-gray-600">Base styling works on 320px+</p>
      </main>
      
      {/* Enhancements for larger screens */}
      <div className="hidden md:block">
        Enhanced desktop features
      </div>
    </div>
  );
}
Content Prioritization
tsx
// components/ContentPriority.tsx
export default function ContentPriority() {
  return (
    <div className="space-y-4">
      {/* Primary content - visible on all devices */}
      <section className="order-1">
        <h2 className="text-xl font-semibold">Essential Info</h2>
        <p>Most important content first</p>
      </section>
      
      {/* Secondary content - mobile optimized */}
      <section className="order-2 md:order-3">
        <h3 className="text-lg">Supporting Details</h3>
        <p className="text-sm">Additional context</p>
      </section>
      
      {/* Tertiary content - hidden/rearranged on mobile */}
      <aside className="order-3 md:order-2 hidden md:block">
        <p>Non-essential content</p>
      </aside>
    </div>
  );
}
Progressive Enhancement
tsx
// components/ProgressiveEnhancement.tsx
'use client';

import { useEffect, useState } from 'react';

export default function ProgressiveEnhancement() {
  const [supportsTouch, setSupportsTouch] = useState(false);
  
  useEffect(() => {
    setSupportsTouch('ontouchstart' in window);
  }, []);
  
  return (
    <div>
      {/* Basic functionality for all devices */}
      <button className="px-4 py-2 bg-blue-500 text-white rounded">
        Basic Button
      </button>
      
      {/* Enhanced for touch devices */}
      {supportsTouch && (
        <div className="mt-4 p-2 bg-green-100 rounded">
          Touch gestures enabled
        </div>
      )}
      
      {/* Enhanced for larger screens */}
      <div className="hidden lg:block mt-4">
        Desktop-only enhancements
      </div>
    </div>
  );
}
Performance Considerations
tsx
// app/layout.tsx (partial)
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Mobile Optimized',
  description: 'Performance-first mobile site',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="it" className="scroll-smooth">
      <head>
        {/* Critical CSS inlined */}
        <style>{`
          .critical-banner {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
          }
        `}</style>
      </head>
      <body className="antialiased">
        {children}
      </body>
    </html>
  );
}
§ RESPONSIVE BREAKPOINTS
Tailwind Breakpoint System
tsx
// components/ResponsiveExample.tsx
export default function ResponsiveExample() {
  return (
    <div className="p-4">
      {/* Default: mobile */}
      <div className="text-sm p-2 bg-gray-100">
        Mobile: Always visible
      </div>
      
      {/* sm: 640px+ */}
      <div className="hidden sm:block text-base p-3 bg-blue-100">
        Small tablets: 640px+
      </div>
      
      {/* md: 768px+ */}
      <div className="hidden md:block text-lg p-4 bg-green-100">
        Tablets: 768px+
      </div>
      
      {/* lg: 1024px+ */}
      <div className="hidden lg:block text-xl p-5 bg-yellow-100">
        Laptops: 1024px+
      </div>
      
      {/* xl: 1280px+ */}
      <div className="hidden xl:block text-2xl p-6 bg-red-100">
        Desktops: 1280px+
      </div>
      
      {/* 2xl: 1536px+ */}
      <div className="hidden 2xl:block text-3xl p-7 bg-purple-100">
        Large screens: 1536px+
      </div>
    </div>
  );
}
Custom Breakpoints
javascript
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
      'xs': '375px',    // iPhone SE and up
      'sm': '640px',    // Small tablets
      'md': '768px',    // Tablets
      'lg': '1024px',   // Laptops
      'xl': '1280px',   // Desktops
      '2xl': '1536px',  // Large screens
      'tablet': {'min': '640px', 'max': '1023px'},
      'desktop-only': {'min': '1024px'},
      'portrait': {'raw': '(orientation: portrait)'},
      'landscape': {'raw': '(orientation: landscape)'},
    },
  },
};
Container Queries
tsx
// components/ContainerQueryCard.tsx
'use client';

import { useRef, useEffect, useState } from 'react';

export default function ContainerQueryCard() {
  const containerRef = useRef<HTMLDivElement>(null);
  const [width, setWidth] = useState(0);
  
  useEffect(() => {
    const updateWidth = () => {
      if (containerRef.current) {
        setWidth(containerRef.current.offsetWidth);
      }
    };
    
    updateWidth();
    const observer = new ResizeObserver(updateWidth);
    if (containerRef.current) {
      observer.observe(containerRef.current);
    }
    
    return () => observer.disconnect();
  }, []);
  
  return (
    <div ref={containerRef} className="w-full max-w-md mx-auto">
      <div className={`
        p-4 border rounded-lg
        ${width > 400 ? 'bg-blue-50 grid grid-cols-2 gap-4' : 'bg-gray-50'}
        ${width > 300 ? 'text-base' : 'text-sm'}
      `}>
        <div>
          <h3 className={`${width > 400 ? 'text-lg' : 'text-md'} font-bold`}>
            Responsive Card
          </h3>
          <p className="mt-2">Width: {width}px</p>
        </div>
        {width > 400 && (
          <div className="flex items-center justify-center">
            <div className="w-20 h-20 bg-blue-100 rounded"></div>
          </div>
        )}
      </div>
    </div>
  );
}
When to Use Which
tsx
// components/BreakpointStrategy.tsx
export default function BreakpointStrategy() {
  return (
    <div className="space-y-6 p-4">
      {/* Use viewport queries for layout structure */}
      <div className="block md:flex md:space-x-4">
        <div className="flex-1 p-4 bg-gray-100">Main content</div>
        <div className="flex-1 p-4 bg-gray-200 mt-4 md:mt-0">Sidebar</div>
      </div>
      
      {/* Use container queries for reusable components */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        {[1, 2, 3].map((i) => (
          <div key={i} className="border p-4">
            <p className="text-sm lg:text-base">Adaptive text</p>
          </div>
        ))}
      </div>
      
      {/* Use orientation queries for mobile-specific adjustments */}
      <div className="landscape:hidden portrait:block bg-yellow-100 p-4">
        Portrait mode only content
      </div>
    </div>
  );
}
§ MOBILE NAVIGATION
Bottom Navigation Bar
tsx
// components/BottomNavigation.tsx
'use client';

import { Home, Search, Bell, User, Plus } from 'lucide-react';
import { useState } from 'react';

export default function BottomNavigation() {
  const [active, setActive] = useState('home');
  
  const navItems = [
    { id: 'home', icon: Home, label: 'Home' },
    { id: 'search', icon: Search, label: 'Search' },
    { id: 'add', icon: Plus, label: 'Add' },
    { id: 'notifications', icon: Bell, label: 'Notifications' },
    { id: 'profile', icon: User, label: 'Profile' },
  ];
  
  return (
    <nav className="fixed bottom-0 left-0 right-0 bg-white border-t border-gray-200 pb-safe">
      <div className="flex justify-around items-center h-16">
        {navItems.map((item) => {
          const Icon = item.icon;
          const isActive = active === item.id;
          
          return (
            <button
              key={item.id}
              onClick={() => setActive(item.id)}
              className={`
                flex flex-col items-center justify-center p-2
                ${isActive ? 'text-blue-600' : 'text-gray-500'}
              `}
            >
              <Icon className="w-6 h-6" />
              <span className="text-xs mt-1">{item.label}</span>
              {isActive && (
                <div className="w-1 h-1 bg-blue-600 rounded-full mt-1"></div>
              )}
            </button>
          );
        })}
      </div>
    </nav>
  );
}
Hamburger Menu
tsx
// components/HamburgerMenu.tsx
'use client';

import { Menu, X } from 'lucide-react';
import { useState } from 'react';

export default function HamburgerMenu() {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <div className="relative">
      {/* Hamburger Button */}
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="p-2 rounded-lg hover:bg-gray-100 transition-colors"
        aria-label={isOpen ? 'Close menu' : 'Open menu'}
      >
        {isOpen ? (
          <X className="w-6 h-6" />
        ) : (
          <Menu className="w-6 h-6" />
        )}
      </button>
      
      {/* Dropdown Menu */}
      {isOpen && (
        <>
          {/* Backdrop */}
          <div
            className="fixed inset-0 bg-black/20 z-40"
            onClick={() => setIsOpen(false)}
          />
          
          {/* Menu Panel */}
          <div className="absolute right-0 mt-2 w-56 bg-white rounded-lg shadow-lg z-50 border border-gray-200">
            <div className="py-2">
              <a
                href="#"
                className="flex items-center px-4 py-3 text-gray-700 hover:bg-gray-100"
              >
                Dashboard
              </a>
              <a
                href="#"
                className="flex items-center px-4 py-3 text-gray-700 hover:bg-gray-100"
              >
                Profile
              </a>
              <a
                href="#"
                className="flex items-center px-4 py-3 text-gray-700 hover:bg-gray-100"
              >
                Settings
              </a>
              <hr className="my-2" />
              <a
                href="#"
                className="flex items-center px-4 py-3 text-red-600 hover:bg-gray-100"
              >
                Logout
              </a>
            </div>
          </div>
        </>
      )}
    </div>
  );
}
Slide-out Drawer
tsx
// components/SlideOutDrawer.tsx
'use client';

import { Menu, X } from 'lucide-react';
import { useState, useEffect } from 'react';

export default function SlideOutDrawer() {
  const [isOpen, setIsOpen] = useState(false);
  
  // Prevent body scroll when drawer is open
  useEffect(() => {
    if (isOpen) {
      document.body.style.overflow = 'hidden';
    } else {
      document.body.style.overflow = 'unset';
    }
    return () => {
      document.body.style.overflow = 'unset';
    };
  }, [isOpen]);
  
  // Close on escape key
  useEffect(() => {
    const handleEscape = (e: KeyboardEvent) => {
      if (e.key === 'Escape') setIsOpen(false);
    };
    window.addEventListener('keydown', handleEscape);
    return () => window.removeEventListener('keydown', handleEscape);
  }, []);
  
  return (
    <>
      {/* Trigger Button */}
      <button
        onClick={() => setIsOpen(true)}
        className="p-2 bg-blue-600 text-white rounded-lg"
      >
        <Menu className="w-6 h-6" />
      </button>
      
      {/* Drawer Overlay */}
      <div
        className={`
          fixed inset-0 bg-black z-40 transition-opacity duration-300
          ${isOpen ? 'opacity-50' : 'opacity-0 pointer-events-none'}
        `}
        onClick={() => setIsOpen(false)}
      />
      
      {/* Drawer Panel */}
      <div
        className={`
          fixed top-0 left-0 h-full w-64 bg-white shadow-xl z-50
          transform transition-transform duration-300 ease-in-out
          ${isOpen ? 'translate-x-0' : '-translate-x-full'}
        `}
      >
        {/* Drawer Header */}
        <div className="p-4 border-b flex justify-between items-center">
          <h2 className="text-lg font-semibold">Menu</h2>
          <button
            onClick={() => setIsOpen(false)}
            className="p-1 hover:bg-gray-100 rounded"
          >
            <X className="w-5 h-5" />
          </button>
        </div>
        
        {/* Drawer Content */}
        <nav className="p-4">
          <ul className="space-y-2">
            {['Home', 'Profile', 'Messages', 'Settings', 'Help'].map((item) => (
              <li key={item}>
                <a
                  href="#"
                  className="block px-4 py-3 rounded-lg hover:bg-gray-100"
                >
                  {item}
                </a>
              </li>
            ))}
          </ul>
        </nav>
        
        {/* Drawer Footer */}
        <div className="absolute bottom-0 w-full p-4 border-t">
          <button className="w-full py-2 bg-gray-100 rounded-lg">
            Logout
          </button>
        </div>
      </div>
    </>
  );
}
Tab Bar
tsx
// components/TabBar.tsx
'use client';

import { useState } from 'react';

export default function TabBar() {
  const [activeTab, setActiveTab] = useState('all');
  
  const tabs = [
    { id: 'all', label: 'All' },
    { id: 'favorites', label: 'Favorites' },
    { id: 'recent', label: 'Recent' },
    { id: 'archived', label: 'Archived' },
  ];
  
  return (
    <div className="bg-white border-b">
      {/* Scrollable tab container for mobile */}
      <div className="flex overflow-x-auto no-scrollbar px-4">
        {tabs.map((tab) => (
          <button
            key={tab.id}
            onClick={() => setActiveTab(tab.id)}
            className={`
              flex-shrink-0 px-4 py-3 font-medium text-sm relative
              ${activeTab === tab.id ? 'text-blue-600' : 'text-gray-500'}
            `}
          >
            {tab.label}
            {activeTab === tab.id && (
              <div className="absolute bottom-0 left-0 right-0 h-0.5 bg-blue-600"></div>
            )}
          </button>
        ))}
      </div>
    </div>
  );
}

// Add to globals.css for hiding scrollbar
// .no-scrollbar::-webkit-scrollbar { display: none; }
// .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
Floating Action Button
tsx
// components/FloatingActionButton.tsx
'use client';

import { Plus, MessageCircle, Camera, FileText } from 'lucide-react';
import { useState } from 'react';

export default function FloatingActionButton() {
  const [isOpen, setIsOpen] = useState(false);
  
  const actions = [
    { icon: MessageCircle, label: 'New Chat', color: 'bg-blue-500' },
    { icon: Camera, label: 'Take Photo', color: 'bg-green-500' },
    { icon: FileText, label: 'New Note', color: 'bg-purple-500' },
  ];
  
  return (
    <div className="fixed bottom-6 right-6 z-30">
      {/* Action Buttons */}
      <div className="flex flex-col items-end space-y-3 mb-3">
        {isOpen &&
          actions.map((action, index) => {
            const Icon = action.icon;
            return (
              <div
                key={index}
                className="flex items-center space-x-2"
                style={{
                  transform: `translateY(${-60 * (index + 1)}px)`,
                  transition: 'transform 0.2s',
                }}
              >
                <span className="text-sm bg-black/80 text-white px-2 py-1 rounded">
                  {action.label}
                </span>
                <button
                  className={`${action.color} w-12 h-12 rounded-full flex items-center justify-center text-white shadow-lg hover:shadow-xl transition-shadow`}
                >
                  <Icon className="w-5 h-5" />
                </button>
              </div>
            );
          })}
      </div>
      
      {/* Main FAB */}
      <button
        onClick={() => setIsOpen(!isOpen)}
        className={`
          w-14 h-14 bg-blue-600 rounded-full flex items-center justify-center
          text-white shadow-lg hover:shadow-xl transition-all duration-300
          ${isOpen ? 'rotate-45' : ''}
        `}
        aria-label={isOpen ? 'Close menu' : 'Open menu'}
      >
        <Plus className="w-6 h-6" />
      </button>
    </div>
  );
}
§ MOBILE LAYOUTS
Single Column Layouts
tsx
// components/SingleColumnLayout.tsx
export default function SingleColumnLayout() {
  return (
    <div className="min-h-screen bg-gray-50">
      {/* Header */}
      <header className="sticky top-0 bg-white border-b z-10">
        <div className="px-4 py-3">
          <h1 className="text-xl font-bold">App Name</h1>
        </div>
      </header>
      
      {/* Main Content - Single Column */}
      <main className="px-4 py-6">
        <section className="mb-8">
          <h2 className="text-lg font-semibold mb-4">Section 1</h2>
          <div className="space-y-4">
            {[1, 2, 3].map((i) => (
              <div key={i} className="bg-white p-4 rounded-lg shadow">
                Content item {i}
              </div>
            ))}
          </div>
        </section>
        
        <section className="mb-8">
          <h2 className="text-lg font-semibold mb-4">Section 2</h2>
          <div className="space-y-4">
            {[4, 5, 6].map((i) => (
              <div key={i} className="bg-white p-4 rounded-lg shadow">
                Content item {i}
              </div>
            ))}
          </div>
        </section>
      </main>
      
      {/* Footer */}
      <footer className="bg-white border-t px-4 py-6">
        <p className="text-center text-gray-600">Footer content</p>
      </footer>
    </div>
  );
}
Card-based Layouts
tsx
// components/CardLayout.tsx
export default function CardLayout() {
  const cards = [
    { title: 'Product Design', desc: 'Learn UI/UX principles', color: 'bg-blue-100' },
    { title: 'Web Development', desc: 'Frontend & Backend skills', color: 'bg-green-100' },
    { title: 'Mobile Apps', desc: 'React Native tutorials', color: 'bg-purple-100' },
    { title: 'Data Science', desc: 'Python & ML courses', color: 'bg-yellow-100' },
    { title: 'Marketing', desc: 'Digital marketing strategies', color: 'bg-pink-100' },
    { title: 'Business', desc: 'Startup fundamentals', color: 'bg-indigo-100' },
  ];
  
  return (
    <div className="p-4">
      {/* Masonry-like grid for mobile */}
      <div className="columns-1 gap-4 sm:columns-2 lg:columns-3">
        {cards.map((card, index) => (
          <div
            key={index}
            className={`
              break-inside-avoid mb-4 p-6 rounded-xl shadow-md
              ${card.color} border border-white/50
            `}
          >
            <h3 className="text-lg font-bold mb-2">{card.title}</h3>
            <p className="text-gray-700">{card.desc}</p>
            <button className="mt-4 px-4 py-2 bg-black/10 hover:bg-black/20 rounded-lg transition">
              Explore
            </button>
          </div>
        ))}
      </div>
    </div>
  );
}
List Views
tsx
// components/MobileListView.tsx
'use client';

import { ChevronRight, Star, Calendar, User } from 'lucide-react';
import { useState } from 'react';

interface ListItem {
  id: number;
  title: string;
  subtitle: string;
  date: string;
  user: string;
  rating: number;
  unread: boolean;
}

export default function MobileListView() {
  const [items] = useState<ListItem[]>([
    { id: 1, title: 'Project Alpha', subtitle: 'Initial design phase completed', date: 'Today', user: 'Alice', rating: 4.5, unread: true },
    { id: 2, title: 'Website Redesign', subtitle: 'Feedback collected from users', date: 'Yesterday', user: 'Bob', rating: 4.2, unread: true },
    { id: 3, title: 'Mobile App', subtitle: 'Beta testing in progress', date: '2 days ago', user: 'Charlie', rating: 4.8, unread: false },
    { id: 4, title: 'Dashboard Update', subtitle: 'Analytics features added', date: '1 week ago', user: 'Diana', rating: 4.0, unread: false },
    { id: 5, title: 'API Integration', subtitle: 'Third-party services connected', date: '2 weeks ago', user: 'Eve', rating: 3.9, unread: false },
  ]);
  
  const [selected, setSelected] = useState<number | null>(null);
  
  return (
    <div className="bg-gray-50 min-h-screen">
      {/* Search/Filter Bar */}
      <div className="sticky top-0 bg-white p-4 border-b">
        <input
          type="text"
          placeholder="Search items..."
          className="w-full px-4 py-3 bg-gray-100 rounded-lg"
        />
      </div>
      
      {/* List Container */}
      <div className="divide-y divide-gray-200">
        {items.map((item) => (
          <div
            key={item.id}
            className={`
              bg-white p-4 transition-colors
              ${selected === item.id ? 'bg-blue-50' : 'hover:bg-gray-50'}
              ${item.unread ? 'border-l-4 border-blue-500' : ''}
            `}
            onClick={() => setSelected(item.id)}
          >
            <div className="flex items-start justify-between">
              <div className="flex-1">
                {/* Main row */}
                <div className="flex items-center gap-2">
                  <h3 className="font-semibold text-gray-900">{item.title}</h3>
                  {item.unread && (
                    <span className="w-2 h-2 bg-blue-500 rounded-full"></span>
                  )}
                </div>
                
                {/* Subtitle */}
                <p className="text-gray-600 text-sm mt-1">{item.subtitle}</p>
                
                {/* Metadata row */}
                <div className="flex items-center gap-4 mt-3">
                  <span className="flex items-center gap-1 text-xs text-gray-500">
                    <Calendar className="w-3 h-3" />
                    {item.date}
                  </span>
                  <span className="flex items-center gap-1 text-xs text-gray-500">
                    <User className="w-3 h-3" />
                    {item.user}
                  </span>
                  <span className="flex items-center gap-1 text-xs text-gray-500">
                    <Star className="w-3 h-3 fill-yellow-400 text-yellow-400" />
                    {item.rating}
                  </span>
                </div>
              </div>
              
              {/* Action/Indicator */}
              <ChevronRight className="w-5 h-5 text-gray-400" />
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
Infinite Scroll
tsx
// components/InfiniteScroll.tsx
'use client';

import { useEffect, useRef, useState, useCallback } from 'react';
import { Loader2 } from 'lucide-react';

export default function InfiniteScroll() {
  const [items, setItems] = useState<number[]>([]);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);
  const observerRef = useRef<IntersectionObserver>();
  const loadMoreRef = useRef<HTMLDivElement>(null);
  
  // Initial load
  useEffect(() => {
    loadMore();
  }, []);
  
  // Setup intersection observer
  useEffect(() => {
    if (!hasMore || loading) return;
    
    observerRef.current = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) {
          loadMore();
        }
      },
      { threshold: 0.1 }
    );
    
    if (loadMoreRef.current) {
      observerRef.current.observe(loadMoreRef.current);
    }
    
    return () => observerRef.current?.disconnect();
  }, [hasMore, loading]);
  
  const loadMore = useCallback(async () => {
    if (loading || !hasMore) return;
    
    setLoading(true);
    
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 1000));
    
    const newItems = Array.from({ length: 10 }, (_, i) => items.length + i + 1);
    
    setItems(prev => [...prev, ...newItems]);
    
    // Stop after 50 items
    if (items.length + 10 >= 50) {
      setHasMore(false);
    }
    
    setLoading(false);
  }, [items.length, loading, hasMore]);
  
  return (
    <div className="p-4">
      <h1 className="text-2xl font-bold mb-6">Infinite Scroll Demo</h1>
      
      <div className="space-y-4">
        {items.map((item) => (
          <div
            key={item}
            className="bg-white p-6 rounded-lg shadow border"
          >
            <div className="flex items-center justify-between">
              <div>
                <h3 className="font-semibold">Item #{item}</h3>
                <p className="text-gray-600">This is item number {item}</p>
              </div>
              <div className="text-sm text-gray-500">
                {new Date().toLocaleDateString()}
              </div>
            </div>
          </div>
        ))}
      </div>
      
      {/* Loading indicator / trigger */}
      <div ref={loadMoreRef} className="py-8">
        {loading ? (
          <div className="flex items-center justify-center gap-2 text-gray-600">
            <Loader2 className="w-5 h-5 animate-spin" />
            Loading more items...
          </div>
        ) : hasMore ? (
          <div className="text-center text-gray-500">
            Scroll down to load more
          </div>
        ) : (
          <div className="text-center text-gray-500 py-8">
            No more items to load
          </div>
        )}
      </div>
    </div>
  );
}
Pull-to-Refresh Indicator
tsx
// components/PullToRefresh.tsx
'use client';

import { ArrowDown, Check } from 'lucide-react';
import { useEffect, useState, useRef } from 'react';

export default function PullToRefresh() {
  const [refreshing, setRefreshing] = useState(false);
  const [lastRefresh, setLastRefresh] = useState(new Date());
  const [pullDistance, setPullDistance] = useState(0);
  const [refreshComplete, setRefreshComplete] = useState(false);
  
  const touchStartY = useRef(0);
  const containerRef = useRef<HTMLDivElement>(null);
  
  const handleTouchStart = (e: TouchEvent) => {
    if (window.scrollY === 0) {
      touchStartY.current = e.touches[0].clientY;
    }
  };
  
  const handleTouchMove = (e: TouchEvent) => {
    if (touchStartY.current === 0) return;
    
    const touchY = e.touches[0].clientY;
    const distance = touchY - touchStartY.current;
    
    if (distance > 0 && window.scrollY === 0) {
      e.preventDefault();
      setPullDistance(Math.min(distance, 120));
    }
  };
  
  const handleTouchEnd = async () => {
    if (pullDistance > 80 && !refreshing) {
      setRefreshing(true);
      setRefreshComplete(false);
      
      // Simulate refresh
      await new Promise(resolve => setTimeout(resolve, 1500));
      
      setLastRefresh(new Date());
      setRefreshing(false);
      setRefreshComplete(true);
      
      // Hide success indicator after delay
      setTimeout(() => {
        setRefreshComplete(false);
      }, 1000);
    }
    
    setPullDistance(0);
    touchStartY.current = 0;
  };
  
  useEffect(() => {
    const container = containerRef.current;
    if (!container) return;
    
    container.addEventListener('touchstart', handleTouchStart, { passive: true });
    container.addEventListener('touchmove', handleTouchMove, { passive: false });
    container.addEventListener('touchend', handleTouchEnd);
    
    return () => {
      container.removeEventListener('touchstart', handleTouchStart);
      container.removeEventListener('touchmove', handleTouchMove);
      container.removeEventListener('touchend', handleTouchEnd);
    };
  }, [pullDistance, refreshing]);
  
  return (
    <div ref={containerRef} className="min-h-screen bg-gray-50">
      {/* Pull to refresh indicator */}
      <div
        className="fixed top-0 left-0 right-0 flex items-center justify-center overflow-hidden transition-all duration-300"
        style={{
          height: `${pullDistance}px`,
          opacity: pullDistance > 0 ? 1 : 0,
        }}
      >
        <div className="flex flex-col items-center">
          {refreshing ? (
            <div className="w-8 h-8 border-2 border-blue-500 border-t-transparent rounded-full animate-spin"></div>
          ) : refreshComplete ? (
            <div className="flex items-center gap-2 text-green-600">
              <Check className="w-6 h-6" />
              <span>Updated!</span>
            </div>
          ) : (
            <>
              <ArrowDown
                className={`w-6 h-6 text-blue-500 transition-transform ${
                  pullDistance > 80 ? 'rotate-180' : ''
                }`}
              />
              <span className="text-sm text-gray-600 mt-1">
                {pullDistance > 80 ? 'Release to refresh' : 'Pull down to refresh'}
              </span>
            </>
          )}
        </div>
      </div>
      
      {/* Content */}
      <div className="pt-16 px-4">
        <div className="mb-6">
          <h1 className="text-2xl font-bold">Pull to Refresh Demo</h1>
          <p className="text-gray-600">
            Last updated: {lastRefresh.toLocaleTimeString()}
          </p>
        </div>
        
        <div className="space-y-4">
          {[1, 2, 3, 4, 5].map((i) => (
            <div key={i} className="bg-white p-4 rounded-lg shadow">
              <h3 className="font-semibold">Item {i}</h3>
              <p className="text-gray-600">Some content here...</p>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}
§ MOBILE FORMS
Touch-friendly Inputs
tsx
// components/TouchFriendlyForm.tsx
export default function TouchFriendlyForm() {
  return (
    <form className="space-y-6 p-4">
      {/* Text Input with minimum 44px touch target */}
      <div>
        <label htmlFor="name" className="block text-sm font-medium mb-2">
          Full Name
        </label>
        <input
          id="name"
          type="text"
          className="w-full px-4 py-3.5 text-base rounded-lg border border-gray-300 focus:border-blue-500 focus:ring-2 focus:ring-blue-200"
          style={{ minHeight: '44px' }}
        />
      </div>
      
      {/* Email Input optimized for mobile keyboards */}
      <div>
        <label htmlFor="email" className="block text-sm font-medium mb-2">
          Email Address
        </label>
        <input
          id="email"
          type="email"
          inputMode="email"
          autoComplete="email"
          className="w-full px-4 py-3.5 text-base rounded-lg border border-gray-300 focus:border-blue-500 focus:ring-2 focus:ring-blue-200"
          style={{ minHeight: '44px' }}
        />
      </div>
      
      {/* Telephone Input */}
      <div>
        <label htmlFor="phone" className="block text-sm font-medium mb-2">
          Phone Number
        </label>
        <input
          id="phone"
          type="tel"
          inputMode="tel"
          autoComplete="tel"
          className="w-full px-4 py-3.5 text-base rounded-lg border border-gray-300 focus:border-blue-500 focus:ring-2 focus:ring-blue-200"
          style={{ minHeight: '44px' }}
        />
      </div>
      
      {/* Number Input with spinner removed on mobile */}
      <div>
        <label htmlFor="quantity" className="block text-sm font-medium mb-2">
          Quantity
        </label>
        <input
          id="quantity"
          type="number"
          inputMode="numeric"
          pattern="[0-9]*"
          className="w-full px-4 py-3.5 text-base rounded-lg border border-gray-300 focus:border-blue-500 focus:ring-2 focus:ring-blue-200 [appearance:textfield] [&::-webkit-outer-spin-button]:appearance-none [&::-webkit-inner-spin-button]:appearance-none"
          style={{ minHeight: '44px' }}
        />
      </div>
      
      {/* Date Input */}
      <div>
        <label htmlFor="dob" className="block text-sm font-medium mb-2">
          Date of Birth
        </label>
        <input
          id="dob"
          type="date"

════════════════════════════════════════════════════════════
FIGMA CATALOG: BATCH2-13-MOBILE-UX-PATTERNS
Prompt ID: 13 / 19
Parte: 2
Exported: 2026-02-06T16:38:14.128Z
Characters: 576
════════════════════════════════════════════════════════════

d: 1,
      orderNumber: 'ORD-7890',
      customer: 'Alex Johnson',
      total: '$456.78',
      status: 'shipped',
      items: [
        { id: 1, name: 'Wireless Headphones', quantity: 1, price: '$199.99' },
        { id: 2, name: 'Phone Case', quantity: 2, price: '$128.90' },
        { id: 3, name: 'Screen Protector', quantity: 1, price: '$127.89' },
      ],
      tracking: {
        carrier: 'UPS',
        number: '1Z999AA1234567890',
        estimated: '2024-01-20',
      },
    },
    {
      id: 2,
      orderNumber: 'ORD-7891',
      customer: 'Maria Garcia',

## § ADVANCED PATTERNS: MOBILE UX PATTERNS

### Server Actions con Validazione

```typescript
// app/actions/mobile-ux-patterns.ts
"use server";

import { z } from "zod";
import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";

const ItemSchema = z.object({
  name: z.string().min(2).max(100),
  description: z.string().min(10).max(5000),
  category: z.enum(["general", "premium", "enterprise"]),
  price: z.number().positive().max(999999),
  metadata: z.record(z.string(), z.unknown()).optional(),
  tags: z.array(z.string().max(50)).max(10).optional(),
  isActive: z.boolean().default(true),
});

type ItemInput = z.infer<typeof ItemSchema>;

interface ActionResult<T = unknown> {
  success: boolean;
  data?: T;
  error?: string;
  fieldErrors?: Record<string, string[]>;
}

export async function createItem(formData: FormData): Promise<ActionResult> {
  const raw = {
    name: formData.get("name") as string,
    description: formData.get("description") as string,
    category: formData.get("category") as string,
    price: parseFloat(formData.get("price") as string),
    tags: (formData.get("tags") as string)?.split(",").map((t) => t.trim()).filter(Boolean),
    isActive: formData.get("isActive") === "true",
  };

  const validation = ItemSchema.safeParse(raw);
  if (!validation.success) {
    return {
      success: false,
      error: "Validation failed",
      fieldErrors: validation.error.flatten().fieldErrors as Record<string, string[]>,
    };
  }

  try {
    const { db } = await import("@/lib/db");
    const item = await db.insert("items").values({
      ...validation.data,
      createdAt: new Date(),
      updatedAt: new Date(),
    }).returning();

    revalidatePath("/items");
    return { success: true, data: item[0] };
  } catch (error) {
    console.error("Failed to create item:", error);
    return { success: false, error: "Failed to create item. Please try again." };
  }
}

export async function updateItem(id: string, formData: FormData): Promise<ActionResult> {
  const raw = {
    name: formData.get("name") as string,
    description: formData.get("description") as string,
    category: formData.get("category") as string,
    price: parseFloat(formData.get("price") as string),
    tags: (formData.get("tags") as string)?.split(",").map((t) => t.trim()).filter(Boolean),
    isActive: formData.get("isActive") === "true",
  };

  const validation = ItemSchema.partial().safeParse(raw);
  if (!validation.success) {
    return { success: false, error: "Validation failed", fieldErrors: validation.error.flatten().fieldErrors as Record<string, string[]> };
  }

  try {
    const { db } = await import("@/lib/db");
    const updated = await db.update("items")
      .set({ ...validation.data, updatedAt: new Date() })
      .where({ id })
      .returning();

    if (!updated[0]) return { success: false, error: "Item not found" };

    revalidatePath("/items");
    revalidatePath(`/items/${id}`);
    return { success: true, data: updated[0] };
  } catch (error) {
    console.error("Failed to update item:", error);
    return { success: false, error: "Failed to update item" };
  }
}

export async function deleteItem(id: string): Promise<ActionResult> {
  try {
    const { db } = await import("@/lib/db");
    await db.update("items").set({ deletedAt: new Date() }).where({ id });
    revalidatePath("/items");
    return { success: true };
  } catch (error) {
    console.error("Failed to delete item:", error);
    return { success: false, error: "Failed to delete item" };
  }
}

export async function bulkUpdateItems(
  ids: string[],
  updates: Partial<ItemInput>
): Promise<ActionResult<{ updated: number }>> {
  if (ids.length === 0) return { success: false, error: "No items selected" };
  if (ids.length > 100) return { success: false, error: "Maximum 100 items at once" };

  try {
    const { db } = await import("@/lib/db");
    let updatedCount = 0;
    for (const id of ids) {
      const result = await db.update("items").set({ ...updates, updatedAt: new Date() }).where({ id }).returning();
      if (result[0]) updatedCount++;
    }
    revalidatePath("/items");
    return { success: true, data: { updated: updatedCount } };
  } catch (error) {
    console.error("Bulk update failed:", error);
    return { success: false, error: "Bulk update failed" };
  }
}
```

### Hook useOptimisticList

```typescript
// hooks/useOptimisticList.ts
"use client";

import { useOptimistic, useTransition, useCallback, useState } from "react";

interface ListItem {
  id: string;
  [key: string]: unknown;
}

type OptimisticAction<T> =
  | { type: "add"; item: T }
  | { type: "update"; id: string; updates: Partial<T> }
  | { type: "remove"; id: string }
  | { type: "reorder"; fromIndex: number; toIndex: number };

export function useOptimisticList<T extends ListItem>(initialItems: T[]) {
  const [isPending, startTransition] = useTransition();
  const [error, setError] = useState<string | null>(null);

  const [optimisticItems, dispatch] = useOptimistic<T[], OptimisticAction<T>>(
    initialItems,
    (state, action) => {
      switch (action.type) {
        case "add":
          return [...state, action.item];
        case "update":
          return state.map((item) =>
            item.id === action.id ? { ...item, ...action.updates } : item
          );
        case "remove":
          return state.filter((item) => item.id !== action.id);
        case "reorder": {
          const newState = [...state];
          const [moved] = newState.splice(action.fromIndex, 1);
          newState.splice(action.toIndex, 0, moved);
          return newState;
        }
        default:
          return state;
      }
    }
  );

  const addOptimistic = useCallback(
    (item: T, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "add", item });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to add item");
        }
      });
    },
    [dispatch]
  );

  const updateOptimistic = useCallback(
    (id: string, updates: Partial<T>, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "update", id, updates });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to update item");
        }
      });
    },
    [dispatch]
  );

  const removeOptimistic = useCallback(
    (id: string, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "remove", id });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to remove item");
        }
      });
    },
    [dispatch]
  );

  return {
    items: optimisticItems,
    isPending,
    error,
    addOptimistic,
    updateOptimistic,
    removeOptimistic,
  };
}
```

### Data Table con Sorting, Filtering e Pagination

```typescript
// components/DataTable.tsx
"use client";

import { useState, useMemo, useCallback } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import {
  ChevronLeft,
  ChevronRight,
  ChevronsLeft,
  ChevronsRight,
  ArrowUpDown,
  ArrowUp,
  ArrowDown,
  Search,
  X,
} from "lucide-react";

interface Column<T> {
  key: keyof T & string;
  label: string;
  sortable?: boolean;
  filterable?: boolean;
  render?: (value: T[keyof T], row: T) => React.ReactNode;
  width?: string;
}

interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  pageSize?: number;
  searchable?: boolean;
  selectable?: boolean;
  onRowClick?: (row: T) => void;
  onSelectionChange?: (selected: T[]) => void;
  emptyMessage?: string;
}

export function DataTable<T extends { id: string }>({
  data,
  columns,
  pageSize = 10,
  searchable = true,
  selectable = false,
  onRowClick,
  onSelectionChange,
  emptyMessage = "No data found",
}: DataTableProps<T>) {
  const [currentPage, setCurrentPage] = useState(1);
  const [sortKey, setSortKey] = useState<string | null>(null);
  const [sortDirection, setSortDirection] = useState<"asc" | "desc">("asc");
  const [searchQuery, setSearchQuery] = useState("");
  const [selectedIds, setSelectedIds] = useState<Set<string>>(new Set());
  const [filters, setFilters] = useState<Record<string, string>>({});

  const filteredData = useMemo(() => {
    let result = [...data];

    if (searchQuery) {
      const query = searchQuery.toLowerCase();
      result = result.filter((row) =>
        columns.some((col) => {
          const value = row[col.key];
          return value !== null && value !== undefined && String(value).toLowerCase().includes(query);
        })
      );
    }

    for (const [key, filterValue] of Object.entries(filters)) {
      if (!filterValue) continue;
      result = result.filter((row) => String(row[key as keyof T]).toLowerCase().includes(filterValue.toLowerCase()));
    }

    if (sortKey) {
      result.sort((a, b) => {
        const aVal = a[sortKey as keyof T];
        const bVal = b[sortKey as keyof T];
        if (aVal === bVal) return 0;
        if (aVal === null || aVal === undefined) return 1;
        if (bVal === null || bVal === undefined) return -1;
        const comparison = aVal < bVal ? -1 : 1;
        return sortDirection === "asc" ? comparison : -comparison;
      });
    }
    return result;
  }, [data, searchQuery, filters, sortKey, sortDirection, columns]);

  const totalPages = Math.ceil(filteredData.length / pageSize);
  const paginatedData = filteredData.slice((currentPage - 1) * pageSize, currentPage * pageSize);

  const handleSort = useCallback((key: string) => {
    if (sortKey === key) {
      setSortDirection((prev) => (prev === "asc" ? "desc" : "asc"));
    } else {
      setSortKey(key);
      setSortDirection("asc");
    }
    setCurrentPage(1);
  }, [sortKey]);

  const toggleSelection = useCallback((id: string) => {
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (next.has(id)) next.delete(id); else next.add(id);
      if (onSelectionChange) {
        onSelectionChange(data.filter((item) => next.has(item.id)));
      }
      return next;
    });
  }, [data, onSelectionChange]);

  const toggleAll = useCallback(() => {
    const allIds = paginatedData.map((item) => item.id);
    const allSelected = allIds.every((id) => selectedIds.has(id));
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (allSelected) { allIds.forEach((id) => next.delete(id)); }
      else { allIds.forEach((id) => next.add(id)); }
      if (onSelectionChange) { onSelectionChange(data.filter((item) => next.has(item.id))); }
      return next;
    });
  }, [paginatedData, selectedIds, data, onSelectionChange]);

  const SortIcon = ({ columnKey }: { columnKey: string }) => {
    if (sortKey !== columnKey) return <ArrowUpDown className="h-3 w-3 ml-1 opacity-50" />;
    return sortDirection === "asc" ? <ArrowUp className="h-3 w-3 ml-1" /> : <ArrowDown className="h-3 w-3 ml-1" />;
  };

  return (
    <Card>
      <CardHeader className="pb-3">
        <div className="flex items-center justify-between">
          <CardTitle className="text-lg">
            {filteredData.length} result{filteredData.length !== 1 ? "s" : ""}
          </CardTitle>
          {searchable && (
            <div className="relative w-64">
              <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-muted-foreground" />
              <Input
                placeholder="Search..."
                value={searchQuery}
                onChange={(e) => { setSearchQuery(e.target.value); setCurrentPage(1); }}
                className="pl-9 pr-9"
              />
              {searchQuery && (
                <button onClick={() => setSearchQuery("")} className="absolute right-3 top-1/2 -translate-y-1/2">
                  <X className="h-4 w-4 text-muted-foreground" />
                </button>
              )}
            </div>
          )}
        </div>
        {selectedIds.size > 0 && (
          <div className="flex items-center gap-2 mt-2">
            <Badge variant="secondary">{selectedIds.size} selected</Badge>
            <Button variant="ghost" size="sm" onClick={() => setSelectedIds(new Set())}>Clear</Button>
          </div>
        )}
      </CardHeader>
      <CardContent>
        <div className="overflow-x-auto">
          <table className="w-full text-sm">
            <thead>
              <tr className="border-b bg-muted/50">
                {selectable && (
                  <th className="w-10 py-3 px-3">
                    <input type="checkbox" onChange={toggleAll}
                      checked={paginatedData.length > 0 && paginatedData.every((item) => selectedIds.has(item.id))} />
                  </th>
                )}
                {columns.map((col) => (
                  <th key={col.key} className="text-left py-3 px-3 font-medium" style={{ width: col.width }}>
                    {col.sortable ? (
                      <button onClick={() => handleSort(col.key)} className="flex items-center hover:text-foreground">
                        {col.label} <SortIcon columnKey={col.key} />
                      </button>
                    ) : col.label}
                  </th>
                ))}
              </tr>
            </thead>
            <tbody>
              {paginatedData.length === 0 ? (
                <tr><td colSpan={columns.length + (selectable ? 1 : 0)} className="text-center py-12 text-muted-foreground">{emptyMessage}</td></tr>
              ) : (
                paginatedData.map((row) => (
                  <tr key={row.id} onClick={() => onRowClick?.(row)}
                    className={`border-b transition-colors hover:bg-muted/50 ${onRowClick ? "cursor-pointer" : ""} ${selectedIds.has(row.id) ? "bg-primary/5" : ""}`}>
                    {selectable && (
                      <td className="py-3 px-3" onClick={(e) => e.stopPropagation()}>
                        <input type="checkbox" checked={selectedIds.has(row.id)} onChange={() => toggleSelection(row.id)} />
                      </td>
                    )}
                    {columns.map((col) => (
                      <td key={col.key} className="py-3 px-3">
                        {col.render ? col.render(row[col.key], row) : String(row[col.key] ?? "")}
                      </td>
                    ))}
                  </tr>
                ))
              )}
            </tbody>
          </table>
        </div>

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4">
            <p className="text-sm text-muted-foreground">
              Page {currentPage} of {totalPages}
            </p>
            <div className="flex items-center gap-1">
              <Button variant="outline" size="sm" onClick={() => setCurrentPage(1)} disabled={currentPage === 1}>
                <ChevronsLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage((p) => p - 1)} disabled={currentPage === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage((p) => p + 1)} disabled={currentPage === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage(totalPages)} disabled={currentPage === totalPages}>
                <ChevronsRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```

### API Route con Middleware Pattern

```typescript
// lib/api/middleware.ts
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";

type Handler = (
  req: NextRequest,
  context: { params: Record<string, string>; user?: { id: string; role: string } }
) => Promise<NextResponse>;

type Middleware = (handler: Handler) => Handler;

export function withValidation<T>(schema: z.ZodSchema<T>, source: "body" | "query" = "body"): Middleware {
  return (handler) => async (req, context) => {
    try {
      let data: unknown;
      if (source === "body") {
        data = await req.json();
      } else {
        const searchParams = Object.fromEntries(req.nextUrl.searchParams);
        data = searchParams;
      }
      const parsed = schema.parse(data);
      (req as any).validated = parsed;
      return handler(req, context);
    } catch (error) {
      if (error instanceof z.ZodError) {
        return NextResponse.json(
          { error: "Validation failed", details: error.errors.map((e) => ({ path: e.path.join("."), message: e.message })) },
          { status: 400 }
        );
      }
      return NextResponse.json({ error: "Invalid request body" }, { status: 400 });
    }
  };
}

export function withAuth(requiredRole?: string): Middleware {
  return (handler) => async (req, context) => {
    const authHeader = req.headers.get("authorization");
    if (!authHeader?.startsWith("Bearer ")) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const token = authHeader.slice(7);
    try {
      const { verifyToken } = await import("@/lib/auth");
      const user = await verifyToken(token);
      if (!user) return NextResponse.json({ error: "Invalid token" }, { status: 401 });
      if (requiredRole && user.role !== requiredRole && user.role !== "admin") {
        return NextResponse.json({ error: "Forbidden" }, { status: 403 });
      }
      context.user = user;
      return handler(req, context);
    } catch {
      return NextResponse.json({ error: "Authentication failed" }, { status: 401 });
    }
  };
}

export function withRateLimit(maxRequests: number = 60, windowMs: number = 60000): Middleware {
  const requests = new Map<string, { count: number; resetAt: number }>();

  return (handler) => async (req, context) => {
    const ip = req.headers.get("x-forwarded-for") || req.headers.get("x-real-ip") || "unknown";
    const now = Date.now();
    const entry = requests.get(ip);

    if (!entry || now > entry.resetAt) {
      requests.set(ip, { count: 1, resetAt: now + windowMs });
    } else if (entry.count >= maxRequests) {
      return NextResponse.json(
        { error: "Too many requests" },
        {
          status: 429,
          headers: {
            "X-RateLimit-Limit": maxRequests.toString(),
            "X-RateLimit-Remaining": "0",
            "X-RateLimit-Reset": new Date(entry.resetAt).toISOString(),
            "Retry-After": Math.ceil((entry.resetAt - now) / 1000).toString(),
          },
        }
      );
    } else {
      entry.count++;
    }

    return handler(req, context);
  };
}

export function withErrorHandler(): Middleware {
  return (handler) => async (req, context) => {
    try {
      return await handler(req, context);
    } catch (error) {
      console.error(`[API Error] ${req.method} ${req.url}:`, error);

      if (error instanceof z.ZodError) {
        return NextResponse.json({ error: "Validation error", details: error.errors }, { status: 400 });
      }

      const message = error instanceof Error ? error.message : "Internal server error";
      const status = (error as any).status || 500;
      return NextResponse.json({ error: message }, { status });
    }
  };
}

export function compose(...middlewares: Middleware[]): Middleware {
  return (handler) => {
    let composed = handler;
    for (let i = middlewares.length - 1; i >= 0; i--) {
      composed = middlewares[i](composed);
    }
    return composed;
  };
}

// Esempio d'uso:
// const handler = compose(withErrorHandler(), withAuth("admin"), withRateLimit(30))(async (req, ctx) => {
//   const items = await db.findMany("items", { userId: ctx.user!.id });
//   return NextResponse.json({ items });
// });
```


### MOBILE UX PATTERNS - Utility Helper #585

```typescript
// lib/utils/mobile-ux-patterns-helper-585.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #279

```typescript
// lib/utils/mobile-ux-patterns-helper-279.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #409

```typescript
// lib/utils/mobile-ux-patterns-helper-409.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #254

```typescript
// lib/utils/mobile-ux-patterns-helper-254.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #102

```typescript
// lib/utils/mobile-ux-patterns-helper-102.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #895

```typescript
// lib/utils/mobile-ux-patterns-helper-895.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #486

```typescript
// lib/utils/mobile-ux-patterns-helper-486.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #887

```typescript
// lib/utils/mobile-ux-patterns-helper-887.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #98

```typescript
// lib/utils/mobile-ux-patterns-helper-98.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #698

```typescript
// lib/utils/mobile-ux-patterns-helper-698.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #826

```typescript
// lib/utils/mobile-ux-patterns-helper-826.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #560

```typescript
// lib/utils/mobile-ux-patterns-helper-560.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #963

```typescript
// lib/utils/mobile-ux-patterns-helper-963.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #9

```typescript
// lib/utils/mobile-ux-patterns-helper-9.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #558

```typescript
// lib/utils/mobile-ux-patterns-helper-558.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #262

```typescript
// lib/utils/mobile-ux-patterns-helper-262.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
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

  getConfig(): Readonly<MOBILEUXPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MOBILE UX PATTERNS - Utility Helper #559

```typescript
// lib/utils/mobile-ux-patterns-helper-559.ts
import { z } from "zod";

interface MOBILEUXPATTERNSConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
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
  debug: z.boolean().default(false),
});

export class MOBILEUXPATTERNSProcessor<TInput, TOutput> {
  private config: MOBILEUXPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MOBILEUXPATTERNSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.