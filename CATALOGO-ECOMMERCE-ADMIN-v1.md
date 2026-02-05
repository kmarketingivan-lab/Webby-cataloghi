# CATALOGO-ECOMMERCE-ADMIN-v1

> **Dominio**: E-Commerce
> **Stack**: Next.js, React, TypeScript, Tailwind CSS, shadcn/ui, tRPC, Zod
> **Versione**: 1.0
> **Data**: 2026-02-04

---

## 1. INDICE

| # | Sezione | Path |
|---|---------|------|
| 0 | [Setup](#0-setup) | - |
| 1 | [src/app/admin/layout.tsx](#1-src-app-admin-layout-tsx) | `src/app/admin/layout.tsx` |
| 2 | [src/app/admin/page.tsx](#2-src-app-admin-page-tsx) | `src/app/admin/page.tsx` |
| 3 | [src/components/admin/sidebar.tsx](#3-src-components-admin-sidebar-tsx) | `src/components/admin/sidebar.tsx` |
| 4 | [src/components/admin/header.tsx](#4-src-components-admin-header-tsx) | `src/components/admin/header.tsx` |
| 5 | [src/components/admin/data-table.tsx](#5-src-components-admin-data-table-tsx) | `src/components/admin/data-table.tsx` |
| 6 | [src/components/admin/stats-card.tsx](#6-src-components-admin-stats-card-tsx) | `src/components/admin/stats-card.tsx` |
| 7 | [src/components/admin/chart-card.tsx](#7-src-components-admin-chart-card-tsx) | `src/components/admin/chart-card.tsx` |
| 8 | [src/app/admin/products/page.tsx](#8-src-app-admin-products-page-tsx) | `src/app/admin/products/page.tsx` |
| 9 | [src/app/admin/products/new/page.tsx](#9-src-app-admin-products-new-page-tsx) | `src/app/admin/products/new/page.tsx` |
| 10 | [src/app/admin/products/[id]/page.tsx](#10-src-app-admin-products-id-page-tsx) | `src/app/admin/products/[id]/page.tsx` |
| 11 | [src/components/admin/products/product-form.tsx](#11-src-components-admin-products-product-form-tsx) | `src/components/admin/products/product-form.tsx` |
| 12 | [src/app/admin/orders/page.tsx](#12-src-app-admin-orders-page-tsx) | `src/app/admin/orders/page.tsx` |
| 13 | [src/app/admin/orders/[id]/page.tsx](#13-src-app-admin-orders-id-page-tsx) | `src/app/admin/orders/[id]/page.tsx` |
| 14 | [src/components/admin/orders/order-actions.tsx](#14-src-components-admin-orders-order-actions-tsx) | `src/components/admin/orders/order-actions.tsx` |
| 15 | [src/app/admin/customers/page.tsx](#15-src-app-admin-customers-page-tsx) | `src/app/admin/customers/page.tsx` |
| 16 | [src/app/admin/customers/[id]/page.tsx](#16-src-app-admin-customers-id-page-tsx) | `src/app/admin/customers/[id]/page.tsx` |
| 17 | [src/app/admin/categories/page.tsx](#17-src-app-admin-categories-page-tsx) | `src/app/admin/categories/page.tsx` |

---

## 0. Setup

Prima di iniziare con i file, è fondamentale che tu abbia configurato il tuo progetto Next.js con le seguenti librerie. Se non le hai, installale:

```bash
# Shadcn/ui (segui la guida per l'inizializzazione: npx shadcn-ui@latest init)
npx shadcn-ui@latest add button card input dropdown-menu dialog table checkbox select textarea label toast sonner alert-dialog calendar date-picker form radio-group separator switch tabs badge avatar command tooltip
# Altre dipendenze
npm install clsx tailwind-merge lucide-react recharts react-hook-form zod @hookform/resolvers @tanstack/react-table @tanstack/react-query @trpc/client @trpc/react-query @trpc/server @trpc/next next-auth
# Per drag-and-drop (se necessario, per le categorie)
npm install @hello-pangea/dnd # (o react-beautiful-dnd, ma è meno mantenuto)
```

Assicurati di avere un file `globals.css` che importi i layer di Tailwind e i CSS di Shadcn/ui.

---

## 1. src/app/admin/layout.tsx

```typescript
// src/app/admin/layout.tsx
'use client';

import { useState, createContext, useContext, useEffect } from 'react';
import { usePathname } from 'next/navigation';
import { Sidebar } from '@/components/admin/sidebar';
import { Header } from '@/components/admin/header';
import { Breadcrumb, BreadcrumbItem, BreadcrumbLink, BreadcrumbList, BreadcrumbSeparator } from '@/components/ui/breadcrumb';
import { ADMIN_NAV_ITEMS } from '@/lib/admin/constants';
import { Sheet, SheetContent, SheetTrigger } from '@/components/ui/sheet';
import { Button } from '@/components/ui/button';
import { Menu } from 'lucide-react';
import { Toaster } from '@/components/ui/sonner';

interface SidebarContextType {
  isCollapsed: boolean;
  toggleCollapse: () => void;
}

const SidebarContext = createContext<SidebarContextType | undefined>(undefined);

export const useSidebar = () => {
  const context = useContext(SidebarContext);
  if (!context) {
    throw new Error('useSidebar must be used within a SidebarProvider');
  }
  return context;
};

export default function AdminLayout({ children }: { children: React.ReactNode }) {
  const [isCollapsed, setIsCollapsed] = useState(false);
  const [isMobileMenuOpen, setIsMobileMenuOpen] = useState(false);
  const pathname = usePathname();

  const toggleCollapse = () => setIsCollapsed(!isCollapsed);

  const generateBreadcrumbs = (path: string) => {
    const pathSegments = path.split('/').filter(segment => segment && segment !== 'admin');
    const breadcrumbs = [{ label: 'Dashboard', href: '/admin' }];

    let currentPath = '/admin';
    pathSegments.forEach((segment, index) => {
      currentPath += `/${segment}`;
      const navItem = ADMIN_NAV_ITEMS.find(item => item.href === currentPath);
      const label = navItem?.title || segment.charAt(0).toUpperCase() + segment.slice(1).replace(/-/g, ' ');
      breadcrumbs.push({ label, href: currentPath });
    });
    return breadcrumbs;
  };

  const breadcrumbs = generateBreadcrumbs(pathname);

  useEffect(() => {
    // Close mobile menu on route change
    setIsMobileMenuOpen(false);
  }, [pathname]);

  return (
    <SidebarContext.Provider value={{ isCollapsed, toggleCollapse }}>
      <div className="flex min-h-screen bg-gray-50 dark:bg-gray-950">
        {/* Desktop Sidebar */}
        <div className={`hidden lg:block transition-all duration-300 ${isCollapsed ? 'w-[70px]' : 'w-[260px]'}`}>
          <Sidebar isCollapsed={isCollapsed} />
        </div>

        {/* Mobile Sidebar */}
        <Sheet open={isMobileMenuOpen} onOpenChange={setIsMobileMenuOpen}>
          <SheetContent side="left" className="p-0 w-[260px]">
            <Sidebar isCollapsed={false} onLinkClick={() => setIsMobileMenuOpen(false)} />
          </SheetContent>
        </Sheet>

        <div className="flex-1 flex flex-col">
          <Header onMobileMenuToggle={() => setIsMobileMenuOpen(true)} />
          <main className="flex-1 p-4 lg:p-6 overflow-auto">
            <div className="mb-6 flex items-center justify-between">
              <Breadcrumb>
                <BreadcrumbList>
                  {breadcrumbs.map((crumb, index) => (
                    <BreadcrumbItem key={crumb.href}>
                      <BreadcrumbLink href={crumb.href} className={index === breadcrumbs.length - 1 ? 'font-semibold' : ''}>
                        {crumb.label}
                      </BreadcrumbLink>
                      {index < breadcrumbs.length - 1 && <BreadcrumbSeparator />}
                    </BreadcrumbItem>
                  ))}
                </BreadcrumbList>
              </Breadcrumb>
              <div className="lg:hidden">
                <SheetTrigger asChild>
                  <Button variant="outline" size="icon">
                    <Menu className="h-4 w-4" />
                  </Button>
                </SheetTrigger>
              </div>
            </div>
            {children}
          </main>
        </div>
      </div>
      <Toaster />
    </SidebarContext.Provider>
  );
}
```

---

## 2. src/app/admin/page.tsx

```typescript
// src/app/admin/page.tsx
'use client';

import { StatsCard } from '@/components/admin/stats-card';
import { ChartCard } from '@/components/admin/chart-card';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table';
import { Badge } from '@/components/ui/badge';
import { DollarSign, ShoppingCart, Users, TrendingUp, Package } from 'lucide-react';
import { trpc } from '@/server/trpc/client';
import { formatCurrency } from '@/lib/utils';
import Link from 'next/link';
import { Skeleton } from '@/components/ui/skeleton';

export default function AdminDashboardPage() {
  const { data: dashboardStats, isLoading: isLoadingStats, isError: isErrorStats } = trpc.admin.getDashboardStats.useQuery({ dateRange: 'last30days' });
  const { data: revenueChart, isLoading: isLoadingChart, isError: isErrorChart } = trpc.admin.getRevenueChart.useQuery({ dateRange: 'last30days' });
  const { data: recentOrders, isLoading: isLoadingOrders, isError: isErrorOrders } = trpc.admin.getRecentOrders.useQuery({ limit: 5 });
  const { data: topProducts, isLoading: isLoadingTopProducts, isError: isErrorTopProducts } = trpc.admin.getTopProducts.useQuery({ limit: 5 });
  const { data: lowStockProducts, isLoading: isLoadingLowStock, isError: isErrorLowStock } = trpc.admin.getLowStockProducts.useQuery();

  return (
    <div className="grid gap-6">
      <h1 className="text-3xl font-bold">Dashboard</h1>

      {/* KPI Cards */}
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
        {isLoadingStats ? (
          Array.from({ length: 4 }).map((_, i) => <Skeleton key={i} className="h-[120px] w-full" />)
        ) : isErrorStats ? (
          <div className="col-span-4 text-red-500">Error loading dashboard stats.</div>
        ) : (
          <>
            <StatsCard
              title="Total Revenue"
              value={formatCurrency(dashboardStats?.totalRevenue || 0)}
              change={`${dashboardStats?.revenueChange || 0}%`}
              changeType={(dashboardStats?.revenueChange || 0) >= 0 ? 'positive' : 'negative'}
              icon={<DollarSign className="h-4 w-4 text-muted-foreground" />}
            />
            <StatsCard
              title="Total Orders"
              value={dashboardStats?.totalOrders.toString() || '0'}
              change={`${dashboardStats?.ordersChange || 0}%`}
              changeType={(dashboardStats?.ordersChange || 0) >= 0 ? 'positive' : 'negative'}
              icon={<ShoppingCart className="h-4 w-4 text-muted-foreground" />}
            />
            <StatsCard
              title="New Customers"
              value={dashboardStats?.newCustomers.toString() || '0'}
              change={`${dashboardStats?.customersChange || 0}%`}
              changeType={(dashboardStats?.customersChange || 0) >= 0 ? 'positive' : 'negative'}
              icon={<Users className="h-4 w-4 text-muted-foreground" />}
            />
            <StatsCard
              title="Conversion Rate"
              value={`${dashboardStats?.conversionRate || 0}%`}
              change={`${dashboardStats?.conversionChange || 0}%`}
              changeType={(dashboardStats?.conversionChange || 0) >= 0 ? 'positive' : 'negative'}
              icon={<TrendingUp className="h-4 w-4 text-muted-foreground" />}
            />
          </>
        )}
      </div>

      {/* Revenue Chart & Recent Orders */}
      <div className="grid gap-6 lg:grid-cols-3">
        <div className="lg:col-span-2">
          {isLoadingChart ? (
            <Skeleton className="h-[350px] w-full" />
          ) : isErrorChart ? (
            <Card className="h-[350px] flex items-center justify-center text-red-500">Error loading revenue chart.</Card>
          ) : (
            <ChartCard
              title="Revenue (Last 30 Days)"
              data={revenueChart || []}
              dataKeyX="date"
              dataKeyY="revenue"
              chartType="line"
            />
          )}
        </div>
        <Card>
          <CardHeader>
            <CardTitle>Recent Orders</CardTitle>
            <CardDescription>Latest orders from your store.</CardDescription>
          </CardHeader>
          <CardContent>
            {isLoadingOrders ? (
              <div className="space-y-4">
                {Array.from({ length: 5 }).map((_, i) => (
                  <div key={i} className="flex items-center space-x-4">
                    <Skeleton className="h-8 w-8 rounded-full" />
                    <div className="flex-1 space-y-1">
                      <Skeleton className="h-4 w-3/4" />
                      <Skeleton className="h-3 w-1/2" />
                    </div>
                  </div>
                ))}
              </div>
            ) : isErrorOrders ? (
              <div className="text-red-500">Error loading recent orders.</div>
            ) : recentOrders && recentOrders.length > 0 ? (
              <Table>
                <TableHeader>
                  <TableRow>
                    <TableHead>Order ID</TableHead>
                    <TableHead>Customer</TableHead>
                    <TableHead>Total</TableHead>
                    <TableHead>Status</TableHead>
                  </TableRow>
                </TableHeader>
                <TableBody>
                  {recentOrders.map((order) => (
                    <TableRow key={order.id}>
                      <TableCell>
                        <Link href={`/admin/orders/${order.id}`} className="hover:underline">
                          #{order.id.substring(0, 8)}
                        </Link>
                      </TableCell>
                      <TableCell>{order.customerName}</TableCell>
                      <TableCell>{formatCurrency(order.total)}</TableCell>
                      <TableCell>
                        <Badge variant={order.status === 'completed' ? 'success' : order.status === 'pending' ? 'warning' : 'destructive'}>
                          {order.status}
                        </Badge>
                      </TableCell>
                    </TableRow>
                  ))}
                </TableBody>
              </Table>
            ) : (
              <p className="text-muted-foreground">No recent orders.</p>
            )}
          </CardContent>
        </Card>
      </div>

      {/* Top Products & Low Stock Alerts */}
      <div className="grid gap-6 lg:grid-cols-2">
        <Card>
          <CardHeader>
            <CardTitle>Top Products</CardTitle>
            <CardDescription>Best-selling products by quantity.</CardDescription>
          </CardHeader>
          <CardContent>
            {isLoadingTopProducts ? (
              <div className="space-y-4">
                {Array.from({ length: 5 }).map((_, i) => (
                  <div key={i} className="flex items-center space-x-4">
                    <Skeleton className="h-10 w-10 rounded-md" />
                    <div className="flex-1 space-y-1">
                      <Skeleton className="h-4 w-3/4" />
                      <Skeleton className="h-3 w-1/2" />
                    </div>
                  </div>
                ))}
              </div>
            ) : isErrorTopProducts ? (
              <div className="text-red-500">Error loading top products.</div>
            ) : topProducts && topProducts.length > 0 ? (
              <Table>
                <TableHeader>
                  <TableRow>
                    <TableHead>Product</TableHead>
                    <TableHead>Sales</TableHead>
                    <TableHead>Revenue</TableHead>
                  </TableRow>
                </TableHeader>
                <TableBody>
                  {topProducts.map((product) => (
                    <TableRow key={product.id}>
                      <TableCell className="font-medium">{product.name}</TableCell>
                      <TableCell>{product.sales}</TableCell>
                      <TableCell>{formatCurrency(product.revenue)}</TableCell>
                    </TableRow>
                  ))}
                </TableBody>
              </Table>
            ) : (
              <p className="text-muted-foreground">No top products yet.</p>
            )}
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle>Low Stock Alerts</CardTitle>
            <CardDescription>Products running low on inventory.</CardDescription>
          </CardHeader>
          <CardContent>
            {isLoadingLowStock ? (
              <div className="space-y-4">
                {Array.from({ length: 5 }).map((_, i) => (
                  <div key={i} className="flex items-center space-x-4">
                    <Skeleton className="h-10 w-10 rounded-md" />
                    <div className="flex-1 space-y-1">
                      <Skeleton className="h-4 w-3/4" />
                      <Skeleton className="h-3 w-1/2" />
                    </div>
                  </div>
                ))}
              </div>
            ) : isErrorLowStock ? (
              <div className="text-red-500">Error loading low stock products.</div>
            ) : lowStockProducts && lowStockProducts.length > 0 ? (
              <Table>
                <TableHeader>
                  <TableRow>
                    <TableHead>Product</TableHead>
                    <TableHead>SKU</TableHead>
                    <TableHead>Stock</TableHead>
                  </TableRow>
                </TableHeader>
                <TableBody>
                  {lowStockProducts.map((product) => (
                    <TableRow key={product.id}>
                      <TableCell className="font-medium">{product.name}</TableCell>
                      <TableCell>{product.sku}</TableCell>
                      <TableCell className="text-red-500 font-semibold">{product.stock}</TableCell>
                    </TableRow>
                  ))}
                </TableBody>
              </Table>
            ) : (
              <p className="text-muted-foreground">No products are currently low on stock. Good job!</p>
            )}
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
```

---

## 3. src/components/admin/sidebar.tsx

```typescript
// src/components/admin/sidebar.tsx
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';
import { ADMIN_NAV_ITEMS } from '@/lib/admin/constants';
import { ChevronDown, ChevronRight, Store, PanelLeftClose, PanelLeftOpen } from 'lucide-react';
import { cn } from '@/lib/utils';
import { Button } from '@/components/ui/button';
import { useSidebar } from '@/app/admin/layout';
import { useState } from 'react';

interface SidebarProps {
  isCollapsed: boolean;
  onLinkClick?: () => void; // For mobile to close sheet
}

interface NavItemProps {
  item: typeof ADMIN_NAV_ITEMS[0];
  isCollapsed: boolean;
  level?: number;
  onLinkClick?: () => void;
}

const NavItem: React.FC<NavItemProps> = ({ item, isCollapsed, level = 0, onLinkClick }) => {
  const pathname = usePathname();
  const isActive = pathname.startsWith(item.href);
  const [isOpen, setIsOpen] = useState(isActive);

  const hasSubItems = item.subItems && item.subItems.length > 0;

  const handleToggle = (e: React.MouseEvent) => {
    e.preventDefault();
    setIsOpen(!isOpen);
  };

  const paddingLeft = level * 16 + 16; // 16px per level + base padding

  return (
    <li className="relative">
      <Link
        href={item.href}
        onClick={(e) => {
          if (hasSubItems) {
            handleToggle(e);
          } else {
            onLinkClick?.();
          }
        }}
        className={cn(
          'flex items-center gap-3 rounded-md px-3 py-2 text-sm font-medium transition-colors hover:bg-muted',
          isActive ? 'bg-muted text-primary' : 'text-muted-foreground',
          isCollapsed ? 'justify-center' : '',
          level > 0 ? 'ml-4' : ''
        )}
        style={{ paddingLeft: isCollapsed ? 'auto' : `${paddingLeft}px` }}
      >
        {item.icon && <item.icon className={cn('h-5 w-5', isCollapsed ? 'mr-0' : 'mr-2')} />}
        {!isCollapsed && (
          <>
            <span>{item.title}</span>
            {hasSubItems && (
              <span className="ml-auto">
                {isOpen ? <ChevronDown className="h-4 w-4" /> : <ChevronRight className="h-4 w-4" />}
              </span>
            )}
          </>
        )}
      </Link>
      {hasSubItems && isOpen && !isCollapsed && (
        <ul className="mt-1 space-y-1">
          {item.subItems?.map((subItem) => (
            <NavItem key={subItem.href} item={subItem} isCollapsed={isCollapsed} level={level + 1} onLinkClick={onLinkClick} />
          ))}
        </ul>
      )}
    </li>
  );
};

export function Sidebar({ isCollapsed, onLinkClick }: SidebarProps) {
  const { toggleCollapse } = useSidebar();

  return (
    <aside className={cn(
      "flex h-full flex-col border-r bg-background transition-all duration-300",
      isCollapsed ? 'w-[70px]' : 'w-[260px]'
    )}>
      <div className="flex h-16 items-center border-b px-4 lg:px-6">
        <Link href="/admin" className="flex items-center gap-2 font-semibold">
          <Store className="h-6 w-6" />
          {!isCollapsed && <span className="text-lg">Admin Panel</span>}
        </Link>
      </div>
      <nav className="flex-1 overflow-auto py-4">
        <ul className="grid gap-1 px-2 lg:px-4">
          {ADMIN_NAV_ITEMS.map((item) => (
            <NavItem key={item.href} item={item} isCollapsed={isCollapsed} onLinkClick={onLinkClick} />
          ))}
        </ul>
      </nav>
      <div className="mt-auto border-t p-2 lg:p-4 hidden lg:block">
        <Button
          variant="ghost"
          size="icon"
          className="w-full justify-center"
          onClick={toggleCollapse}
          aria-label={isCollapsed ? 'Expand sidebar' : 'Collapse sidebar'}
        >
          {isCollapsed ? <PanelLeftOpen className="h-5 w-5" /> : <PanelLeftClose className="h-5 w-5" />}
        </Button>
      </div>
    </aside>
  );
}
```

---

## 4. src/components/admin/header.tsx

```typescript
// src/components/admin/header.tsx
'use client';

import { Input } from '@/components/ui/input';
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuLabel, DropdownMenuSeparator, DropdownMenuTrigger } from '@/components/ui/dropdown-menu';
import { Button } from '@/components/ui/button';
import { Bell, Search, User, Menu } from 'lucide-react';
import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar';
import { useSidebar } from '@/app/admin/layout';
import { useEffect, useState } from 'react';
import { useToast } from '@/components/ui/use-toast'; // Assuming you have shadcn toast setup
import { useRouter } from 'next/navigation';
import { useSession, signOut } from 'next-auth/react'; // Assuming NextAuth.js

interface HeaderProps {
  onMobileMenuToggle: () => void;
}

export function Header({ onMobileMenuToggle }: HeaderProps) {
  const { isCollapsed } = useSidebar();
  const { toast } = useToast();
  const router = useRouter();
  const { data: session } = useSession(); // Get session info

  const handleSearch = (query: string) => {
    if (query.trim()) {
      toast({
        title: 'Search initiated',
        description: `Searching for "${query}"... (Not implemented yet)`,
      });
      // In a real app, navigate to a search results page or filter current view
    }
  };

  const handleLogout = async () => {
    await signOut({ callbackUrl: '/' }); // Redirect to home after logout
  };

  // Cmd+K search shortcut
  useEffect(() => {
    const handleKeyDown = (event: KeyboardEvent) => {
      if ((event.metaKey || event.ctrlKey) && event.key === 'k') {
        event.preventDefault();
        // Focus search input or open a search dialog
        const searchInput = document.getElementById('admin-search-input');
        if (searchInput) {
          searchInput.focus();
        }
      }
    };

    window.addEventListener('keydown', handleKeyDown);
    return () => window.removeEventListener('keydown', handleKeyDown);
  }, []);

  return (
    <header className="sticky top-0 z-30 flex h-16 items-center gap-4 border-b bg-background px-4 lg:px-6">
      <Button
        variant="ghost"
        size="icon"
        className="lg:hidden"
        onClick={onMobileMenuToggle}
        aria-label="Toggle mobile menu"
      >
        <Menu className="h-5 w-5" />
      </Button>

      <div className="relative flex-1 max-w-md">
        <Search className="absolute left-2.5 top-2.5 h-4 w-4 text-muted-foreground" />
        <Input
          id="admin-search-input"
          type="search"
          placeholder="Search products, orders, customers... (Cmd+K)"
          className="w-full rounded-lg bg-background pl-8 md:w-[200px] lg:w-[300px]"
          onKeyDown={(e) => {
            if (e.key === 'Enter') {
              handleSearch(e.currentTarget.value);
            }
          }}
        />
      </div>

      <div className="ml-auto flex items-center gap-4">
        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="ghost" size="icon" className="relative">
              <Bell className="h-5 w-5" />
              <span className="absolute -top-1 -right-1 flex h-3 w-3 items-center justify-center rounded-full bg-red-500 text-[10px] text-white">
                3
              </span>
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end" className="w-64">
            <DropdownMenuLabel>Notifications</DropdownMenuLabel>
            <DropdownMenuSeparator />
            <DropdownMenuItem>New order #1234</DropdownMenuItem>
            <DropdownMenuItem>Product 'T-Shirt' low stock</DropdownMenuItem>
            <DropdownMenuItem>Customer 'John Doe' registered</DropdownMenuItem>
            <DropdownMenuSeparator />
            <DropdownMenuItem>View all notifications</DropdownMenuItem>
          </DropdownMenuContent>
        </DropdownMenu>

        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="ghost" className="relative h-8 w-8 rounded-full">
              <Avatar className="h-8 w-8">
                <AvatarImage src={session?.user?.image || "https://github.com/shadcn.png"} alt="User Avatar" />
                <AvatarFallback>{session?.user?.name?.charAt(0) || 'AD'}</AvatarFallback>
              </Avatar>
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end">
            <DropdownMenuLabel>
              {session?.user?.name || 'Admin User'}
              <p className="text-xs text-muted-foreground">{session?.user?.email || 'admin@example.com'}</p>
            </DropdownMenuLabel>
            <DropdownMenuSeparator />
            <DropdownMenuItem onClick={() => router.push('/admin/settings/general')}>Settings</DropdownMenuItem>
            <DropdownMenuItem>Support</DropdownMenuItem>
            <DropdownMenuSeparator />
            <DropdownMenuItem onClick={handleLogout}>Logout</DropdownMenuItem>
          </DropdownMenuContent>
        </DropdownMenu>
      </div>
    </header>
  );
}
```

---

## 5. src/components/admin/data-table.tsx

```typescript
// src/components/admin/data-table.tsx
'use client';

import * as React from 'react';
import {
  ColumnDef,
  ColumnFiltersState,
  SortingState,
  VisibilityState,
  flexRender,
  getCoreRowModel,
  getFilteredRowModel,
  getPaginationRowModel,
  getSortedRowModel,
  useReactTable,
} from '@tanstack/react-table';
import { ArrowUpDown, ChevronDown, FileDown, PlusCircle } from 'lucide-react';

import { Button } from '@/components/ui/button';
import { Checkbox } from '@/components/ui/checkbox';
import {
  DropdownMenu,
  DropdownMenuCheckboxItem,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { Input } from '@/components/ui/input';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table';
import { Skeleton } from '@/components/ui/skeleton';
import { toast } from 'sonner';

interface DataTableProps<TData, TValue> {
  columns: ColumnDef<TData, TValue>[];
  data: TData[];
  totalCount: number;
  pageSize: number;
  pageIndex: number;
  onPaginationChange: (pagination: { pageIndex: number; pageSize: number }) => void;
  onSortingChange: (sorting: SortingState) => void;
  onFiltersChange: (filters: ColumnFiltersState) => void;
  onRowSelectionChange: (selectedRowIds: Record<string, boolean>) => void;
  loading: boolean;
  filterColumnId?: string; // Column to apply global filter on
  bulkActions?: {
    label: string;
    action: (selectedIds: string[]) => void;
    icon?: React.ElementType;
    variant?: 'default' | 'destructive';
  }[];
  onCreateNew?: () => void;
  exportCsv?: (selectedIds: string[]) => Promise<string>; // Returns CSV URL
}

export function DataTable<TData extends { id: string }, TValue>({
  columns,
  data,
  totalCount,
  pageSize,
  pageIndex,
  onPaginationChange,
  onSortingChange,
  onFiltersChange,
  onRowSelectionChange,
  loading,
  filterColumnId,
  bulkActions,
  onCreateNew,
  exportCsv,
}: DataTableProps<TData, TValue>) {
  const [sorting, setSorting] = React.useState<SortingState>([]);
  const [columnFilters, setColumnFilters] = React.useState<ColumnFiltersState>([]);
  const [columnVisibility, setColumnVisibility] = React.useState<VisibilityState>({});
  const [rowSelection, setRowSelection] = React.useState({});

  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    getPaginationRowModel: getPaginationRowModel(), // Client-side pagination for UI, but server-side data fetch
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    onSortingChange: (updater) => {
      const newSorting = typeof updater === 'function' ? updater(sorting) : updater;
      setSorting(newSorting);
      onSortingChange(newSorting);
    },
    onColumnFiltersChange: (updater) => {
      const newFilters = typeof updater === 'function' ? updater(columnFilters) : updater;
      setColumnFilters(newFilters);
      onFiltersChange(newFilters);
    },
    onColumnVisibilityChange: setColumnVisibility,
    onRowSelectionChange: (updater) => {
      const newSelection = typeof updater === 'function' ? updater(rowSelection) : updater;
      setRowSelection(newSelection);
      onRowSelectionChange(newSelection);
    },
    state: {
      sorting,
      columnFilters,
      columnVisibility,
      rowSelection,
      pagination: {
        pageIndex,
        pageSize,
      },
    },
    manualPagination: true,
    manualSorting: true,
    manualFiltering: true,
    pageCount: Math.ceil(totalCount / pageSize),
  });

  const selectedRowIds = table.getSelectedRowModel().rows.map((row) => row.original.id);
  const hasSelectedRows = selectedRowIds.length > 0;

  const handleExportCsv = async () => {
    if (!exportCsv) return;
    try {
      toast.loading('Exporting data...');
      const url = await exportCsv(hasSelectedRows ? selectedRowIds : []);
      if (url) {
        window.open(url, '_blank');
        toast.success('Data exported successfully!');
      } else {
        toast.error('Export failed: No URL returned.');
      }
    } catch (error) {
      console.error('CSV Export error:', error);
      toast.error('Failed to export data.');
    } finally {
      toast.dismiss();
    }
  };

  return (
    <div className="w-full">
      <div className="flex items-center py-4 gap-2">
        {filterColumnId && (
          <Input
            placeholder={`Filter ${filterColumnId}...`}
            value={(table.getColumn(filterColumnId)?.getFilterValue() as string) ?? ''}
            onChange={(event) =>
              table.getColumn(filterColumnId)?.setFilterValue(event.target.value)
            }
            className="max-w-sm"
          />
        )}

        {hasSelectedRows && bulkActions && bulkActions.length > 0 && (
          <DropdownMenu>
            <DropdownMenuTrigger asChild>
              <Button variant="outline" className="ml-auto">
                Bulk Actions ({selectedRowIds.length}) <ChevronDown className="ml-2 h-4 w-4" />
              </Button>
            </DropdownMenuTrigger>
            <DropdownMenuContent align="end">
              {bulkActions.map((action, index) => (
                <DropdownMenuItem key={index} onClick={() => action.action(selectedRowIds)}>
                  {action.icon && <action.icon className="mr-2 h-4 w-4" />}
                  {action.label}
                </DropdownMenuItem>
              ))}
            </DropdownMenuContent>
          </DropdownMenu>
        )}

        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="outline" className="ml-auto">
              Columns <ChevronDown className="ml-2 h-4 w-4" />
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end">
            {table
              .getAllColumns()
              .filter((column) => column.getCanHide())
              .map((column) => {
                return (
                  <DropdownMenuCheckboxItem
                    key={column.id}
                    className="capitalize"
                    checked={column.getIsVisible()}
                    onCheckedChange={(value) =>
                      column.toggleVisibility(!!value)
                    }
                  >
                    {column.id}
                  </DropdownMenuCheckboxItem>
                );
              })}
          </DropdownMenuContent>
        </DropdownMenu>

        {exportCsv && (
          <Button variant="outline" onClick={handleExportCsv}>
            <FileDown className="mr-2 h-4 w-4" /> Export CSV
          </Button>
        )}

        {onCreateNew && (
          <Button onClick={onCreateNew}>
            <PlusCircle className="mr-2 h-4 w-4" /> Create New
          </Button>
        )}
      </div>

      <div className="rounded-md border">
        <Table>
          <TableHeader>
            {table.getHeaderGroups().map((headerGroup) => (
              <TableRow key={headerGroup.id}>
                {headerGroup.headers.map((header) => {
                  return (
                    <TableHead key={header.id}>
                      {header.isPlaceholder
                        ? null
                        : flexRender(
                            header.column.columnDef.header,
                            header.getContext()
                          )}
                    </TableHead>
                  );
                })}
              </TableRow>
            ))}
          </TableHeader>
          <TableBody>
            {loading ? (
              Array.from({ length: pageSize }).map((_, rowIndex) => (
                <TableRow key={rowIndex}>
                  {table.getAllColumns().map((column, colIndex) => (
                    <TableCell key={colIndex}>
                      <Skeleton className="h-6 w-full" />
                    </TableCell>
                  ))}
                </TableRow>
              ))
            ) : table.getRowModel().rows?.length ? (
              table.getRowModel().rows.map((row) => (
                <TableRow
                  key={row.id}
                  data-state={row.getIsSelected() && 'selected'}
                >
                  {row.getVisibleCells().map((cell) => (
                    <TableCell key={cell.id}>
                      {flexRender(cell.column.columnDef.cell, cell.getContext())}
                    </TableCell>
                  ))}
                </TableRow>
              ))
            ) : (
              <TableRow>
                <TableCell colSpan={columns.length} className="h-24 text-center">
                  No results.
                </TableCell>
              </TableRow>
            )}
          </TableBody>
        </Table>
      </div>

      <div className="flex items-center justify-end space-x-2 py-4">
        <div className="flex-1 text-sm text-muted-foreground">
          {table.getFilteredSelectedRowModel().rows.length} of{' '}
          {table.getFilteredRowModel().rows.length} row(s) selected.
        </div>
        <div className="space-x-2">
          <Button
            variant="outline"
            size="sm"
            onClick={() => onPaginationChange({ pageIndex: pageIndex - 1, pageSize })}
            disabled={pageIndex === 0 || loading}
          >
            Previous
          </Button>
          <Button
            variant="outline"
            size="sm"
            onClick={() => onPaginationChange({ pageIndex: pageIndex + 1, pageSize })}
            disabled={pageIndex >= table.getPageCount() - 1 || loading}
          >
            Next
          </Button>
        </div>
      </div>
    </div>
  );
}

// Helper for sortable columns
export const createSortableColumn = <TData, TValue>(
  id: string,
  header: string,
  accessorKey?: keyof TData | string,
  cell?: ColumnDef<TData, TValue>['cell']
): ColumnDef<TData, TValue> => ({
  accessorKey: accessorKey || id,
  id,
  header: ({ column }) => (
    <Button
      variant="ghost"
      onClick={() => column.toggleSorting(column.getIsSorted() === 'asc')}
    >
      {header}
      <ArrowUpDown className="ml-2 h-4 w-4" />
    </Button>
  ),
  cell,
});

// Helper for selection column
export const selectionColumn: ColumnDef<any> = {
  id: 'select',
  header: ({ table }) => (
    <Checkbox
      checked={
        table.getIsAllPageRowsSelected() ||
        (table.getIsSomePageRowsSelected() && 'indeterminate')
      }
      onCheckedChange={(value) => table.toggleAllPageRowsSelected(!!value)}
      aria-label="Select all"
    />
  ),
  cell: ({ row }) => (
    <Checkbox
      checked={row.getIsSelected()}
      onCheckedChange={(value) => row.toggleSelected(!!value)}
      aria-label="Select row"
    />
  ),
  enableSorting: false,
  enableHiding: false,
};
```

---

## 6. src/components/admin/stats-card.tsx

```typescript
// src/components/admin/stats-card.tsx
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { cn } from '@/lib/utils';
import { ArrowDown, ArrowUp } from 'lucide-react';
import React from 'react';

interface StatsCardProps {
  title: string;
  value: string;
  change: string;
  changeType: 'positive' | 'negative' | 'neutral';
  icon: React.ReactNode;
}

export function StatsCard({ title, value, change, changeType, icon }: StatsCardProps) {
  const changeColorClass = changeType === 'positive' ? 'text-green-500' : changeType === 'negative' ? 'text-red-500' : 'text-muted-foreground';
  const ChangeIcon = changeType === 'positive' ? ArrowUp : ArrowDown;

  return (
    <Card>
      <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
        <CardTitle className="text-sm font-medium">{title}</CardTitle>
        {icon}
      </CardHeader>
      <CardContent>
        <div className="text-2xl font-bold">{value}</div>
        <p className={cn("text-xs", changeColorClass, "flex items-center gap-1 mt-1")}>
          {changeType !== 'neutral' && <ChangeIcon className="h-3 w-3" />}
          {change}
          <span className="text-muted-foreground ml-1">from last month</span>
        </p>
      </CardContent>
    </Card>
  );
}
```

---

## 7. src/components/admin/chart-card.tsx

```typescript
// src/components/admin/chart-card.tsx
'use client';

import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import {
  ResponsiveContainer,
  LineChart,
  Line,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  Legend,
  BarChart,
  Bar,
} from 'recharts';
import { Skeleton } from '@/components/ui/skeleton';

interface ChartCardProps {
  title: string;
  data: any[];
  dataKeyX: string;
  dataKeyY: string;
  chartType?: 'line' | 'bar';
  isLoading?: boolean;
  isError?: boolean;
}

export function ChartCard({ title, data, dataKeyX, dataKeyY, chartType = 'line', isLoading, isError }: ChartCardProps) {
  if (isLoading) {
    return <Skeleton className="h-[350px] w-full" />;
  }

  if (isError) {
    return (
      <Card className="h-[350px] flex items-center justify-center">
        <p className="text-red-500">Error loading chart data.</p>
      </Card>
    );
  }

  return (
    <Card>
      <CardHeader>
        <CardTitle>{title}</CardTitle>
      </CardHeader>
      <CardContent>
        <div className="h-[300px]">
          <ResponsiveContainer width="100%" height="100%">
            {chartType === 'line' ? (
              <LineChart
                data={data}
                margin={{
                  top: 5,
                  right: 10,
                  left: 10,
                  bottom: 0,
                }}
              >
                <CartesianGrid strokeDasharray="3 3" className="stroke-muted" />
                <XAxis dataKey={dataKeyX} stroke="#888888" fontSize={12} tickLine={false} axisLine={false} />
                <YAxis stroke="#888888" fontSize={12} tickLine={false} axisLine={false} tickFormatter={(value) => `$${value}`} />
                <Tooltip
                  cursor={{ strokeDasharray: '3 3' }}
                  contentStyle={{ backgroundColor: 'hsl(var(--background))', border: '1px solid hsl(var(--border))', borderRadius: 'var(--radius)' }}
                  labelStyle={{ color: 'hsl(var(--foreground))' }}
                  itemStyle={{ color: 'hsl(var(--foreground))' }}
                />
                <Legend />
                <Line type="monotone" dataKey={dataKeyY} stroke="hsl(var(--primary))" activeDot={{ r: 8 }} />
              </LineChart>
            ) : (
              <BarChart
                data={data}
                margin={{
                  top: 5,
                  right: 10,
                  left: 10,
                  bottom: 0,
                }}
              >
                <CartesianGrid strokeDasharray="3 3" className="stroke-muted" />
                <XAxis dataKey={dataKeyX} stroke="#888888" fontSize={12} tickLine={false} axisLine={false} />
                <YAxis stroke="#888888" fontSize={12} tickLine={false} axisLine={false} tickFormatter={(value) => `$${value}`} />
                <Tooltip
                  cursor={{ fill: 'hsl(var(--muted))' }}
                  contentStyle={{ backgroundColor: 'hsl(var(--background))', border: '1px solid hsl(var(--border))', borderRadius: 'var(--radius)' }}
                  labelStyle={{ color: 'hsl(var(--foreground))' }}
                  itemStyle={{ color: 'hsl(var(--foreground))' }}
                />
                <Legend />
                <Bar dataKey={dataKeyY} fill="hsl(var(--primary))" radius={[4, 4, 0, 0]} />
              </BarChart>
            )}
          </ResponsiveContainer>
        </div>
      </CardContent>
    </Card>
  );
}
```

---

## 8. src/app/admin/products/page.tsx

```typescript
// src/app/admin/products/page.tsx
'use client';

import { useState } from 'react';
import { ColumnDef } from '@tanstack/react-table';
import { DataTable, createSortableColumn, selectionColumn } from '@/components/admin/data-table';
import { trpc } from '@/server/trpc/client';
import { Product } from '@/server/services/admin-service'; // Assuming Product type
import { formatCurrency } from '@/lib/utils';
import { Badge } from '@/components/ui/badge';
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuLabel, DropdownMenuSeparator, DropdownMenuTrigger } from '@/components/ui/dropdown-menu';
import { Button } from '@/components/ui/button';
import { MoreHorizontal, Edit, Trash } from 'lucide-react';
import Link from 'next/link';
import { useRouter } from 'next/navigation';
import { toast } from 'sonner';
import { AlertDialog, AlertDialogAction, AlertDialogCancel, AlertDialogContent, AlertDialogDescription, AlertDialogFooter, AlertDialogHeader, AlertDialogTitle, AlertDialogTrigger } from '@/components/ui/alert-dialog';

export default function ProductsPage() {
  const router = useRouter();
  const [pagination, setPagination] = useState({ pageIndex: 0, pageSize: 10 });
  const [sorting, setSorting] = useState([]);
  const [columnFilters, setColumnFilters] = useState([]);
  const [rowSelection, setRowSelection] = useState({});

  const { data, isLoading, isError, refetch } = trpc.admin.getProducts.useQuery({
    page: pagination.pageIndex + 1,
    limit: pagination.pageSize,
    sortBy: sorting.length > 0 ? sorting[0].id : undefined,
    sortOrder: sorting.length > 0 ? (sorting[0].desc ? 'desc' : 'asc') : undefined,
    filters: columnFilters.reduce((acc, filter) => ({ ...acc, [filter.id]: filter.value }), {}),
  });

  const products = data?.products || [];
  const totalCount = data?.totalCount || 0;

  const deleteProductMutation = trpc.admin.deleteProduct.useMutation({
    onSuccess: () => {
      toast.success('Product deleted successfully!');
      refetch();
    },
    onError: (error) => {
      toast.error('Failed to delete product.', { description: error.message });
    },
  });

  const handleDelete = (id: string) => {
    deleteProductMutation.mutate(id);
  };

  const columns: ColumnDef<Product>[] = [
    selectionColumn,
    createSortableColumn('name', 'Product Name', 'name', ({ row }) => (
      <Link href={`/admin/products/${row.original.id}`} className="font-medium hover:underline">
        {row.getValue('name')}
      </Link>
    )),
    createSortableColumn('sku', 'SKU', 'sku'),
    createSortableColumn('category', 'Category', 'category'),
    createSortableColumn('price', 'Price', 'price', ({ row }) => formatCurrency(row.getValue('price'))),
    createSortableColumn('stock', 'Stock', 'stock', ({ row }) => (
      <Badge variant={row.original.stock < 10 ? 'destructive' : 'outline'}>
        {row.getValue('stock')}
      </Badge>
    )),
    createSortableColumn('status', 'Status', 'status', ({ row }) => (
      <Badge variant={row.getValue('status') === 'published' ? 'success' : 'secondary'}>
        {row.getValue('status')}
      </Badge>
    )),
    {
      id: 'actions',
      enableHiding: false,
      cell: ({ row }) => (
        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="ghost" className="h-8 w-8 p-0">
              <span className="sr-only">Open menu</span>
              <MoreHorizontal className="h-4 w-4" />
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end">
            <DropdownMenuLabel>Actions</DropdownMenuLabel>
            <DropdownMenuItem onClick={() => router.push(`/admin/products/${row.original.id}`)}>
              <Edit className="mr-2 h-4 w-4" /> Edit
            </DropdownMenuItem>
            <DropdownMenuSeparator />
            <AlertDialog>
              <AlertDialogTrigger asChild>
                <DropdownMenuItem onSelect={(e) => e.preventDefault()} className="text-red-600">
                  <Trash className="mr-2 h-4 w-4" /> Delete
                </DropdownMenuItem>
              </AlertDialogTrigger>
              <AlertDialogContent>
                <AlertDialogHeader>
                  <AlertDialogTitle>Are you absolutely sure?</AlertDialogTitle>
                  <AlertDialogDescription>
                    This action cannot be undone. This will permanently delete the product &quot;{row.original.name}&quot;
                    and remove its data from our servers.
                  </AlertDialogDescription>
                </AlertDialogHeader>
                <AlertDialogFooter>
                  <AlertDialogCancel>Cancel</AlertDialogCancel>
                  <AlertDialogAction onClick={() => handleDelete(row.original.id)} className="bg-red-600 hover:bg-red-700">
                    Delete
                  </AlertDialogAction>
                </AlertDialogFooter>
              </AlertDialogContent>
            </AlertDialog>
          </DropdownMenuContent>
        </DropdownMenu>
      ),
    },
  ];

  const bulkDeleteAction = {
    label: 'Delete Selected',
    action: (ids: string[]) => {
      toast.info(`Deleting ${ids.length} products... (Not fully implemented)`);
      // Implement bulk delete mutation here
    },
    icon: Trash,
    variant: 'destructive' as const,
  };

  return (
    <div className="space-y-6">
      <h1 className="text-3xl font-bold">Products</h1>
      <DataTable
        columns={columns}
        data={products}
        totalCount={totalCount}
        pageSize={pagination.pageSize}
        pageIndex={pagination.pageIndex}
        onPaginationChange={setPagination}
        onSortingChange={setSorting}
        onFiltersChange={setColumnFilters}
        onRowSelectionChange={setRowSelection}
        loading={isLoading}
        filterColumnId="name"
        bulkActions={[bulkDeleteAction]}
        onCreateNew={() => router.push('/admin/products/new')}
        exportCsv={async (ids) => {
          // Mock CSV export
          toast.info(`Exporting ${ids.length || 'all'} products...`);
          await new Promise(resolve => setTimeout(resolve, 1000));
          return 'https://example.com/mock-products-export.csv';
        }}
      />
    </div>
  );
}
```

---

## 9. src/app/admin/products/new/page.tsx

```typescript
// src/app/admin/products/new/page.tsx
import { ProductForm } from '@/components/admin/products/product-form';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';

export default function CreateProductPage() {
  return (
    <div className="space-y-6">
      <h1 className="text-3xl font-bold">Create New Product</h1>
      <Card>
        <CardHeader>
          <CardTitle>Product Details</CardTitle>
        </CardHeader>
        <CardContent>
          <ProductForm />
        </CardContent>
      </Card>
    </div>
  );
}
```

---

## 10. src/app/admin/products/[id]/page.tsx

```typescript
// src/app/admin/products/[id]/page.tsx
'use client';

import { ProductForm } from '@/components/admin/products/product-form';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { trpc } from '@/server/trpc/client';
import { Skeleton } from '@/components/ui/skeleton';

export default function EditProductPage({ params }: { params: { id: string } }) {
  const { id } = params;
  const { data: product, isLoading, isError } = trpc.admin.getProductById.useQuery(id);

  if (isLoading) {
    return (
      <div className="space-y-6">
        <Skeleton className="h-10 w-64" />
        <Card>
          <CardHeader><Skeleton className="h-6 w-48" /></CardHeader>
          <CardContent><Skeleton className="h-[800px] w-full" /></CardContent>
        </Card>
      </div>
    );
  }

  if (isError || !product) {
    return (
      <div className="space-y-6">
        <h1 className="text-3xl font-bold">Product Not Found</h1>
        <p className="text-red-500">Could not load product with ID: {id}</p>
      </div>
    );
  }

  return (
    <div className="space-y-6">
      <h1 className="text-3xl font-bold">Edit Product: {product.name}</h1>
      <Card>
        <CardHeader>
          <CardTitle>Product Details</CardTitle>
        </CardHeader>
        <CardContent>
          <ProductForm initialData={product} />
        </CardContent>
      </Card>
    </div>
  );
}
```

---

## 11. src/components/admin/products/product-form.tsx

```typescript
// src/components/admin/products/product-form.tsx
'use client';

import { useForm, useFieldArray } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import * as z from 'zod';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { Form, FormControl, FormDescription, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Switch } from '@/components/ui/switch';
import { Checkbox } from '@/components/ui/checkbox';
import { PlusCircle, Trash2, UploadCloud } from 'lucide-react';
import { Product } from '@/server/services/admin-service'; // Assuming Product type
import { trpc } from '@/server/trpc/client';
import { toast } from 'sonner';
import { useRouter } from 'next/navigation';
import { useEffect, useState } from 'react';
import Image from 'next/image';
import { Badge } from '@/components/ui/badge';

const productFormSchema = z.object({
  name: z.string().min(3, { message: 'Product name must be at least 3 characters.' }),
  slug: z.string().min(3, { message: 'Slug must be at least 3 characters.' }),
  description: z.string().optional(),
  price: z.coerce.number().min(0.01, { message: 'Price must be positive.' }),
  compareAtPrice: z.coerce.number().optional().nullable(),
  costPerItem: z.coerce.number().optional().nullable(),
  sku: z.string().optional(),
  barcode: z.string().optional(),
  stock: z.coerce.number().int().min(0, { message: 'Stock cannot be negative.' }),
  trackInventory: z.boolean().default(true),
  status: z.enum(['draft', 'published']),
  category: z.string().min(1, { message: 'Please select a category.' }),
  images: z.array(z.string().url()).optional(), // Array of image URLs
  variants: z.array(
    z.object({
      id: z.string().optional(), // For existing variants
      optionName: z.string().min(1, 'Option name is required'),
      optionValues: z.array(
        z.object({
          value: z.string().min(1, 'Value is required'),
          sku: z.string().optional(),
          price: z.coerce.number().min(0).optional(),
          stock: z.coerce.number().int().min(0).optional(),
        })
      ).min(1, 'At least one option value is required'),
    })
  ).optional(),
  seoTitle: z.string().optional(),
  seoDescription: z.string().optional(),
});

type ProductFormValues = z.infer<typeof productFormSchema>;

interface ProductFormProps {
  initialData?: Product;
}

export function ProductForm({ initialData }: ProductFormProps) {
  const router = useRouter();
  const form = useForm<ProductFormValues>({
    resolver: zodResolver(productFormSchema),
    defaultValues: initialData || {
      name: '',
      slug: '',
      description: '',
      price: 0.01,
      compareAtPrice: null,
      costPerItem: null,
      sku: '',
      barcode: '',
      stock: 0,
      trackInventory: true,
      status: 'draft',
      category: '',
      images: [],
      variants: [],
      seoTitle: '',
      seoDescription: '',
    },
  });

  const { fields: variantFields, append: appendVariant, remove: removeVariant } = useFieldArray({
    control: form.control,
    name: 'variants',
  });

  const { data: categories, isLoading: isLoadingCategories } = trpc.admin.getCategories.useQuery();

  const createProductMutation = trpc.admin.createProduct.useMutation({
    onSuccess: (newProduct) => {
      toast.success('Product created successfully!');
      router.push(`/admin/products/${newProduct.id}`);
    },
    onError: (error) => {
      toast.error('Failed to create product.', { description: error.message });
    },
  });

  const updateProductMutation = trpc.admin.updateProduct.useMutation({
    onSuccess: (updatedProduct) => {
      toast.success('Product updated successfully!');
      router.push(`/admin/products/${updatedProduct.id}`);
    },
    onError: (error) => {
      toast.error('Failed to update product.', { description: error.message });
    },
  });

  const onSubmit = (values: ProductFormValues) => {
    if (initialData) {
      updateProductMutation.mutate({ id: initialData.id, ...values });
    } else {
      createProductMutation.mutate(values);
    }
  };

  const isSubmitting = createProductMutation.isLoading || updateProductMutation.isLoading;

  // Auto-generate slug
  useEffect(() => {
    const subscription = form.watch((value, { name }) => {
      if (name === 'name' && value.name) {
        form.setValue('slug', value.name.toLowerCase().replace(/[^a-z0-9]+/g, '-').replace(/^-*|-*$/g, ''), { shouldValidate: true });
      }
    });
    return () => subscription.unsubscribe();
  }, [form]);

  // Mock image upload
  const handleImageUpload = (event: React.ChangeEvent<HTMLInputElement>) => {
    const files = event.target.files;
    if (files && files.length > 0) {
      const newImages = Array.from(files).map(file => URL.createObjectURL(file));
      form.setValue('images', [...(form.getValues('images') || []), ...newImages]);
      toast.success(`${files.length} image(s) uploaded (mock)!`);
    }
  };

  const removeImage = (index: number) => {
    const currentImages = form.getValues('images') || [];
    form.setValue('images', currentImages.filter((_, i) => i !== index));
  };

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="grid gap-6 lg:grid-cols-3">
        <div className="lg:col-span-2 space-y-6">
          {/* Basic Info */}
          <Card>
            <CardHeader>
              <CardTitle>Basic Information</CardTitle>
              <CardDescription>Name, description, and slug of your product.</CardDescription>
            </CardHeader>
            <CardContent className="grid gap-4">
              <FormField
                control={form.control}
                name="name"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Product Name</FormLabel>
                    <FormControl>
                      <Input placeholder="e.g., Stylish T-Shirt" {...field} />
                    </FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />
              <FormField
                control={form.control}
                name="slug"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Slug</FormLabel>
                    <FormControl>
                      <Input placeholder="e.g., stylish-t-shirt" {...field} />
                    </FormControl>
                    <FormDescription>Unique URL handle for the product.</FormDescription>
                    <FormMessage />
                  </FormItem>
                )}
              />
              <FormField
                control={form.control}
                name="description"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Description</FormLabel>
                    <FormControl>
                      <Textarea placeholder="Detailed product description..." rows={5} {...field} />
                    </FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />
            </CardContent>
          </Card>

          {/* Pricing */}
          <Card>
            <CardHeader>
              <CardTitle>Pricing</CardTitle>
              <CardDescription>Set the price and cost for your product.</CardDescription>
            </CardHeader>
            <CardContent className="grid gap-4 md:grid-cols-2">
              <FormField
                control={form.control}
                name="price"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Price ($)</FormLabel>
                    <FormControl>
                      <Input type="number" step="0.01" {...field} />
                    </FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />
              <FormField
                control={form.control}
                name="compareAtPrice"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Compare-at price ($)</FormLabel>
                    <FormControl>
                      <Input type="number" step="0.01" placeholder="Optional" {...field} value={field.value === null ? '' : field.value} />
                    </FormControl>
                    <FormDescription>Original price for discount display.</FormDescription>
                    <FormMessage />
                  </FormItem>
                )}
              />
              <FormField
                control={form.control}
                name="costPerItem"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Cost per item ($)</FormLabel>
                    <FormControl>
                      <Input type="number" step="0.01" placeholder="Optional" {...field} value={field.value === null ? '' : field.value} />
                    </FormControl>
                    <FormDescription>Used to calculate profit margin.</FormDescription>
                    <FormMessage />
                  </FormItem>
                )}
              />
            </CardContent>
          </Card>

          {/* Inventory */}
          <Card>
            <CardHeader>
              <CardTitle>Inventory</CardTitle>
              <CardDescription>Manage stock levels and tracking.</CardDescription>
            </CardHeader>
            <CardContent className="grid gap-4 md:grid-cols-2">
              <FormField
                control={form.control}
                name="sku"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>SKU (Stock Keeping Unit)</FormLabel>
                    <FormControl>
                      <Input placeholder="e.g., TSHIRT-RED-M" {...field} />
                    </FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />
              <FormField
                control={form.control}
                name="barcode"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Barcode (ISBN, UPC, GTIN, etc.)</FormLabel>
                    <FormControl>
                      <Input placeholder="e.g., 1234567890123" {...field} />
                    </FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />
              <FormField
                control={form.control}
                name="trackInventory"
                render={({ field }) => (
                  <FormItem className="flex flex-row items-center justify-between rounded-lg border p-4 col-span-2">
                    <div className="space-y-0.5">
                      <FormLabel className="text-base">Track inventory</FormLabel>
                      <FormDescription>
                        Automatically manage stock levels for this product.
                      </FormDescription>
                    </div>
                    <FormControl>
                      <Switch checked={field.value} onCheckedChange={field.onChange} />
                    </FormControl>
                  </FormItem>
                )}
              />
              {form.watch('trackInventory') && (
                <FormField
                  control={form.control}
                  name="stock"
                  render={({ field }) => (
                    <FormItem className="col-span-2">
                      <FormLabel>Quantity in Stock</FormLabel>
                      <FormControl>
                        <Input type="number" {...field} />
                      </FormControl>
                      <FormMessage />
                    </FormItem>
                  )}
                />
              )}
            </CardContent>
          </Card>

          {/* Variants Builder */}
          <Card>
            <CardHeader>
              <CardTitle>Variants</CardTitle>
              <CardDescription>Add options like size or color for your product.</CardDescription>
            </CardHeader>
            <CardContent className="grid gap-4">
              {variantFields.map((variant, index) => (
                <div key={variant.id} className="border p-4 rounded-md relative">
                  <Button
                    type="button"
                    variant="destructive"
                    size="icon"
                    className="absolute top-2 right-2 h-6 w-6"
                    onClick={() => removeVariant(index)}
                  >
                    <Trash2 className="h-4 w-4" />
                  </Button>
                  <FormField
                    control={form.control}
                    name={`variants.${index}.optionName`}
                    render={({ field }) => (
                      <FormItem className="mb-4">
                        <FormLabel>Option Name (e.g., Color, Size)</FormLabel>
                        <FormControl>
                          <Input {...field} placeholder="e.g., Color" />
                        </FormControl>
                        <FormMessage />
                      </FormItem>
                    )}
                  />
                  <FormLabel className="mb-2 block">Option Values</FormLabel>
                  <div className="grid gap-2">
                    {form.watch(`variants.${index}.optionValues`).map((val, valIndex) => (
                      <div key={valIndex} className="flex items-center gap-2">
                        <FormField
                          control={form.control}
                          name={`variants.${index}.optionValues.${valIndex}.value`}
                          render={({ field }) => (
                            <FormItem className="flex-1">
                              <FormControl>
                                <Input {...field} placeholder="e.g., Red, Small" />
                              </FormControl>
                              <FormMessage />
                            </FormItem>
                          )}
                        />
                        <FormField
                          control={form.control}
                          name={`variants.${index}.optionValues.${valIndex}.sku`}
                          render={({ field }) => (
                            <FormItem className="w-24">
                              <FormControl>
                                <Input {...field} placeholder="SKU" />
                              </FormControl>
                            </FormItem>
                          )}
                        />
                        <FormField
                          control={form.control}
                          name={`variants.${index}.optionValues.${valIndex}.price`}
                          render={({ field }) => (
                            <FormItem className="w-24">
                              <FormControl>
                                <Input type="number" step="0.01" {...field} placeholder="Price" value={field.value === undefined ? '' : field.value} />
                              </FormControl>
                            </FormItem>
                          )}
                        />
                        <FormField
                          control={form.control}
                          name={`variants.${index}.optionValues.${valIndex}.stock`}
                          render={({ field }) => (
                            <FormItem className="w-24">
                              <FormControl>
                                <Input type="number" {...field} placeholder="Stock" value={field.value === undefined ? '' : field.value} />
                              </FormControl>
                            </FormItem>
                          )}
                        />
                        <Button
                          type="button"
                          variant="ghost"
                          size="icon"
                          onClick={() => {
                            const currentValues = form.getValues(`variants.${index}.optionValues`);
                            form.setValue(`variants.${index}.optionValues`, currentValues.filter((_, i) => i !== valIndex));
                          }}
                        >
                          <Trash2 className="h-4 w-4" />
                        </Button>
                      </div>
                    ))}
                    <Button
                      type="button"
                      variant="outline"
                      size="sm"
                      onClick={() => form.setValue(`variants.${index}.optionValues`, [...form.getValues(`variants.${index}.optionValues`), { value: '' }])}
                      className="mt-2"
                    >
                      <PlusCircle className="mr-2 h-4 w-4" /> Add Value
                    </Button>
                  </div>
                </div>
              ))}
              <Button
                type="button"
                variant="outline"
                onClick={() => appendVariant({ optionName: '', optionValues: [{ value: '' }] })}
              >
                <PlusCircle className="mr-2 h-4 w-4" /> Add Another Option
              </Button>
            </CardContent>
          </Card>

          {/* Images Upload */}
          <Card>
            <CardHeader>
              <CardTitle>Product Images</CardTitle>
              <CardDescription>Upload images for your product.</CardDescription>
            </CardHeader>
            <CardContent>
              <FormField
                control={form.control}
                name="images"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Images</FormLabel>
                    <FormControl>
                      <div className="grid gap-4">
                        <div className="flex items-center justify-center w-full">
                          <Label
                            htmlFor="image-upload"
                            className="flex flex-col items-center justify-center w-full h-32 border-2 border-dashed rounded-lg cursor-pointer bg-muted hover:bg-muted/80"
                          >
                            <div className="flex flex-col items-center justify-center pt-5 pb-6">
                              <UploadCloud className="w-8 h-8 mb-3 text-muted-foreground" />
                              <p className="mb-2 text-sm text-muted-foreground">
                                <span className="font-semibold">Click to upload</span> or drag and drop
                              </p>
                              <p className="text-xs text-muted-foreground">SVG, PNG, JPG or GIF (MAX. 800x400px)</p>
                            </div>
                            <Input id="image-upload" type="file" className="hidden" multiple onChange={handleImageUpload} />
                          </Label>
                        </div>
                        <div className="grid grid-cols-3 gap-4">
                          {field.value?.map((image, index) => (
                            <div key={index} className="relative group">
                              <Image src={image} alt={`Product image ${index + 1}`} width={150} height={150} className="rounded-md object-cover aspect-square" />
                              <Button
                                type="button"
                                variant="destructive"
                                size="icon"
                                className="absolute top-1 right-1 h-6 w-6 opacity-0 group-hover:opacity-100 transition-opacity"
                                onClick={() => removeImage(index)}
                              >
                                <Trash2 className="h-4 w-4" />
                              </Button>
                            </div>
                          ))}
                        </div>
                      </div>
                    </FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />
            </CardContent>
          </Card>

          {/* SEO Fields */}
          <Card>
            <CardHeader>
              <CardTitle>Search Engine Optimization</CardTitle>
              <CardDescription>Improve your product's visibility on search engines.</CardDescription>
            </CardHeader>
            <CardContent className="grid gap-4">
              <FormField
                control={form.control}
                name="seoTitle"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>SEO Title</FormLabel>
                    <FormControl>
                      <Input placeholder="Meta Title" {...field} />
                    </FormControl>
                    <FormDescription>A concise title for search engine results.</FormDescription>
                    <FormMessage />
                  </FormItem>
                )}
              />
              <FormField
                control={form.control}
                name="seoDescription"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>SEO Description</FormLabel>
                    <FormControl>
                      <Textarea placeholder="Meta Description" rows={3} {...field} />
                    </FormControl>
                    <FormDescription>A brief summary of the product for search engines.</FormDescription>
                    <FormMessage />
                  </FormItem>
                )}
              />
            </CardContent>
          </Card>
        </div>

        {/* Sidebar (Status, Category) */}
        <div className="lg:col-span-1 space-y-6">
          <Card>
            <CardHeader>
              <CardTitle>Status</CardTitle>
            </CardHeader>
            <CardContent>
              <FormField
                control={form.control}
                name="status"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Product Status</FormLabel>
                    <Select onValueChange={field.onChange} defaultValue={field.value}>
                      <FormControl>
                        <SelectTrigger>
                          <SelectValue placeholder="Select status" />
                        </SelectTrigger>
                      </FormControl>
                      <SelectContent>
                        <SelectItem value="draft">Draft</SelectItem>
                        <SelectItem value="published">Published</SelectItem>
                      </SelectContent>
                    </Select>
                    <FormDescription>Set the visibility of your product.</FormDescription>
                    <FormMessage />
                  </FormItem>
                )}
              />
            </CardContent>
          </Card>

          <Card>
            <CardHeader>
              <CardTitle>Category</CardTitle>
            </CardHeader>
            <CardContent>
              <FormField
                control={form.control}
                name="category"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Category</FormLabel>
                    <Select onValueChange={field.onChange} defaultValue={field.value}>
                      <FormControl>
                        <SelectTrigger>
                          <SelectValue placeholder="Select a category" />
                        </SelectTrigger>
                      </FormControl>
                      <SelectContent>
                        {isLoadingCategories ? (
                          <SelectItem value="" disabled>Loading categories...</SelectItem>
                        ) : (
                          categories?.map((cat) => (
                            <SelectItem key={cat.id} value={cat.name}>
                              {cat.name}
                            </SelectItem>
                          ))
                        )}
                      </SelectContent>
                    </Select>
                    <FormMessage />
                  </FormItem>
                )}
              />
            </CardContent>
          </Card>
        </div>

        <div className="lg:col-span-3 flex justify-end gap-2">
          <Button type="button" variant="outline" onClick={() => router.back()}>Cancel</Button>
          <Button type="submit" disabled={isSubmitting}>
            {isSubmitting ? 'Saving...' : initialData ? 'Save Changes' : 'Create Product'}
          </Button>
        </div>
      </form>
    </Form>
  );
}
```

---

## 12. src/app/admin/orders/page.tsx

```typescript
// src/app/admin/orders/page.tsx
'use client';

import { useState } from 'react';
import { ColumnDef } from '@tanstack/react-table';
import { DataTable, createSortableColumn, selectionColumn } from '@/components/admin/data-table';
import { trpc } from '@/server/trpc/client';
import { Order } from '@/server/services/admin-service'; // Assuming Order type
import { formatCurrency, formatDate } from '@/lib/utils';
import { Badge } from '@/components/ui/badge';
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuLabel, DropdownMenuSeparator, DropdownMenuTrigger } from '@/components/ui/dropdown-menu';
import { Button } from '@/components/ui/button';
import { MoreHorizontal, Eye, Truck, XCircle, RefreshCcw } from 'lucide-react';
import Link from 'next/link';
import { useRouter } from 'next/navigation';
import { toast } from 'sonner';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';

export default function OrdersPage() {
  const router = useRouter();
  const [pagination, setPagination] = useState({ pageIndex: 0, pageSize: 10 });
  const [sorting, setSorting] = useState([]);
  const [columnFilters, setColumnFilters] = useState([]);
  const [rowSelection, setRowSelection] = useState({});
  const [statusFilter, setStatusFilter] = useState<string>('all');

  const { data, isLoading, isError, refetch } = trpc.admin.getOrders.useQuery({
    page: pagination.pageIndex + 1,
    limit: pagination.pageSize,
    sortBy: sorting.length > 0 ? sorting[0].id : undefined,
    sortOrder: sorting.length > 0 ? (sorting[0].desc ? 'desc' : 'asc') : undefined,
    filters: {
      ...columnFilters.reduce((acc, filter) => ({ ...acc, [filter.id]: filter.value }), {}),
      status: statusFilter === 'all' ? undefined : statusFilter,
    },
  });

  const orders = data?.orders || [];
  const totalCount = data?.totalCount || 0;

  const updateOrderStatusMutation = trpc.admin.updateOrderStatus.useMutation({
    onSuccess: () => {
      toast.success('Order status updated!');
      refetch();
    },
    onError: (error) => {
      toast.error('Failed to update order status.', { description: error.message });
    },
  });

  const handleUpdateStatus = (orderId: string, status: string) => {
    updateOrderStatusMutation.mutate({ orderId, status });
  };

  const columns: ColumnDef<Order>[] = [
    selectionColumn,
    createSortableColumn('id', 'Order ID', 'id', ({ row }) => (
      <Link href={`/admin/orders/${row.original.id}`} className="font-medium hover:underline">
        #{row.original.id.substring(0, 8)}
      </Link>
    )),
    createSortableColumn('customerName', 'Customer', 'customerName'),
    createSortableColumn('orderDate', 'Date', 'orderDate', ({ row }) => formatDate(row.getValue('orderDate'))),
    createSortableColumn('total', 'Total', 'total', ({ row }) => formatCurrency(row.getValue('total'))),
    createSortableColumn('status', 'Status', 'status', ({ row }) => {
      const status = row.getValue('status');
      let variant: 'default' | 'secondary' | 'destructive' | 'success' | 'warning' = 'secondary';
      switch (status) {
        case 'pending': variant = 'warning'; break;
        case 'processing': variant = 'default'; break;
        case 'shipped': variant = 'default'; break;
        case 'delivered': variant = 'success'; break;
        case 'cancelled': variant = 'destructive'; break;
        case 'refunded': variant = 'destructive'; break;
      }
      return <Badge variant={variant}>{status}</Badge>;
    }),
    {
      id: 'actions',
      enableHiding: false,
      cell: ({ row }) => (
        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="ghost" className="h-8 w-8 p-0">
              <span className="sr-only">Open menu</span>
              <MoreHorizontal className="h-4 w-4" />
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end">
            <DropdownMenuLabel>Actions</DropdownMenuLabel>
            <DropdownMenuItem onClick={() => router.push(`/admin/orders/${row.original.id}`)}>
              <Eye className="mr-2 h-4 w-4" /> View Details
            </DropdownMenuItem>
            {row.original.status === 'pending' && (
              <DropdownMenuItem onClick={() => handleUpdateStatus(row.original.id, 'processing')}>
                <RefreshCcw className="mr-2 h-4 w-4" /> Mark as Processing
              </DropdownMenuItem>
            )}
            {(row.original.status === 'pending' || row.original.status === 'processing') && (
              <DropdownMenuItem onClick={() => handleUpdateStatus(row.original.id, 'shipped')}>
                <Truck className="mr-2 h-4 w-4" /> Mark as Shipped
              </DropdownMenuItem>
            )}
            {row.original.status !== 'cancelled' && row.original.status !== 'refunded' && (
              <>
                <DropdownMenuSeparator />
                <DropdownMenuItem onClick={() => handleUpdateStatus(row.original.id, 'cancelled')} className="text-red-600">
                  <XCircle className="mr-2 h-4 w-4" /> Cancel Order
                </DropdownMenuItem>
              </>
            )}
          </DropdownMenuContent>
        </DropdownMenu>
      ),
    },
  ];

  const bulkShipAction = {
    label: 'Mark Selected as Shipped',
    action: (ids: string[]) => {
      toast.info(`Marking ${ids.length} orders as shipped... (Not fully implemented)`);
      // Implement bulk ship mutation here
    },
    icon: Truck,
  };

  return (
    <div className="space-y-6">
      <h1 className="text-3xl font-bold">Orders</h1>
      <div className="flex items-center gap-2">
        <Select value={statusFilter} onValueChange={setStatusFilter}>
          <SelectTrigger className="w-[180px]">
            <SelectValue placeholder="Filter by Status" />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="all">All Statuses</SelectItem>
            <SelectItem value="pending">Pending</SelectItem>
            <SelectItem value="processing">Processing</SelectItem>
            <SelectItem value="shipped">Shipped</SelectItem>
            <SelectItem value="delivered">Delivered</SelectItem>
            <SelectItem value="cancelled">Cancelled</SelectItem>
            <SelectItem value="refunded">Refunded</SelectItem>
          </SelectContent>
        </Select>
      </div>
      <DataTable
        columns={columns}
        data={orders}
        totalCount={totalCount}
        pageSize={pagination.pageSize}
        pageIndex={pagination.pageIndex}
        onPaginationChange={setPagination}
        onSortingChange={setSorting}
        onFiltersChange={setColumnFilters}
        onRowSelectionChange={setRowSelection}
        loading={isLoading}
        filterColumnId="customerName"
        bulkActions={[bulkShipAction]}
        exportCsv={async (ids) => {
          toast.info(`Exporting ${ids.length || 'all'} orders...`);
          await new Promise(resolve => setTimeout(resolve, 1000));
          return 'https://example.com/mock-orders-export.csv';
        }}
      />
    </div>
  );
}
```

---

## 13. src/app/admin/orders/[id]/page.tsx

```typescript
// src/app/admin/orders/[id]/page.tsx
'use client';

import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Separator } from '@/components/ui/separator';
import { trpc } from '@/server/trpc/client';
import { formatCurrency, formatDate } from '@/lib/utils';
import { Badge } from '@/components/ui/badge';
import { Skeleton } from '@/components/ui/skeleton';
import { OrderActions } from '@/components/admin/orders/order-actions';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table';
import Link from 'next/link';
import { Package, Truck, CheckCircle, XCircle, RefreshCcw, DollarSign } from 'lucide-react';

export default function OrderDetailPage({ params }: { params: { id: string } }) {
  const { id } = params;
  const { data: order, isLoading, isError, refetch } = trpc.admin.getOrderById.useQuery(id);

  if (isLoading) {
    return (
      <div className="space-y-6">
        <Skeleton className="h-10 w-64" />
        <div className="grid gap-6 lg:grid-cols-3">
          <div className="lg:col-span-2 space-y-6">
            <Skeleton className="h-[200px]" />
            <Skeleton className="h-[300px]" />
          </div>
          <div className="lg:col-span-1 space-y-6">
            <Skeleton className="h-[150px]" />
            <Skeleton className="h-[250px]" />
          </div>
        </div>
      </div>
    );
  }

  if (isError || !order) {
    return (
      <div className="space-y-6">
        <h1 className="text-3xl font-bold">Order Not Found</h1>
        <p className="text-red-500">Could not load order with ID: {id}</p>
      </div>
    );
  }

  const getStatusBadgeVariant = (status: string) => {
    switch (status) {
      case 'pending': return 'warning';
      case 'processing': return 'default';
      case 'shipped': return 'default';
      case 'delivered': return 'success';
      case 'cancelled': return 'destructive';
      case 'refunded': return 'destructive';
      default: return 'secondary';
    }
  };

  const getTimelineIcon = (status: string) => {
    switch (status) {
      case 'pending': return <Package className="h-4 w-4 text-muted-foreground" />;
      case 'processing': return <RefreshCcw className="h-4 w-4 text-blue-500" />;
      case 'shipped': return <Truck className="h-4 w-4 text-yellow-500" />;
      case 'delivered': return <CheckCircle className="h-4 w-4 text-green-500" />;
      case 'cancelled': return <XCircle className="h-4 w-4 text-red-500" />;
      case 'refunded': return <DollarSign className="h-4 w-4 text-red-500" />;
      default: return <Package className="h-4 w-4 text-muted-foreground" />;
    }
  };

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-3xl font-bold">Order #{order.id.substring(0, 8)}</h1>
        <OrderActions orderId={order.id} currentStatus={order.status} onActionSuccess={refetch} />
      </div>

      <div className="grid gap-6 lg:grid-cols-3">
        {/* Order Info & Items */}
        <div className="lg:col-span-2 space-y-6">
          <Card>
            <CardHeader>
              <CardTitle>Order Summary</CardTitle>
              <CardDescription>Details about the order.</CardDescription>
            </CardHeader>
            <CardContent className="grid gap-4">
              <div className="grid grid-cols-2 gap-2">
                <div>
                  <p className="text-sm text-muted-foreground">Order Date</p>
                  <p className="font-medium">{formatDate(order.orderDate)}</p>
                </div>
                <div>
                  <p className="text-sm text-muted-foreground">Status</p>
                  <Badge variant={getStatusBadgeVariant(order.status)}>{order.status}</Badge>
                </div>
                <div>
                  <p className="text-sm text-muted-foreground">Total Amount</p>
                  <p className="font-medium">{formatCurrency(order.total)}</p>
                </div>
                <div>
                  <p className="text-sm text-muted-foreground">Payment Status</p>
                  <Badge variant={order.paymentStatus === 'paid' ? 'success' : 'warning'}>{order.paymentStatus}</Badge>
                </div>
              </div>
              <Separator />
              <div>
                <h3 className="font-semibold mb-2">Shipping Address</h3>
                <p>{order.shippingAddress.name}</p>
                <p>{order.shippingAddress.street}</p>
                <p>{order.shippingAddress.city}, {order.shippingAddress.state} {order.shippingAddress.zip}</p>
                <p>{order.shippingAddress.country}</p>
              </div>
              <div>
                <h3 className="font-semibold mb-2">Billing Address</h3>
                <p>{order.billingAddress.name}</p>
                <p>{order.billingAddress.street}</p>
                <p>{order.billingAddress.city}, {order.billingAddress.state} {order.billingAddress.zip}</p>
                <p>{order.billingAddress.country}</p>
              </div>
            </CardContent>
          </Card>

          <Card>
            <CardHeader>
              <CardTitle>Order Items</CardTitle>
            </CardHeader>
            <CardContent>
              <Table>
                <TableHeader>
                  <TableRow>
                    <TableHead>Product</TableHead>
                    <TableHead>SKU</TableHead>
                    <TableHead>Quantity</TableHead>
                    <TableHead className="text-right">Price</TableHead>
                    <TableHead className="text-right">Total</TableHead>
                  </TableRow>
                </TableHeader>
                <TableBody>
                  {order.items.map((item) => (
                    <TableRow key={item.productId}>
                      <TableCell className="font-medium">{item.productName}</TableCell>
                      <TableCell>{item.sku}</TableCell>
                      <TableCell>{item.quantity}</TableCell>
                      <TableCell className="text-right">{formatCurrency(item.price)}</TableCell>
                      <TableCell className="text-right">{formatCurrency(item.price * item.quantity)}</TableCell>
                    </TableRow>
                  ))}
                  <TableRow className="font-semibold">
                    <TableCell colSpan={4} className="text-right">Subtotal</TableCell>
                    <TableCell className="text-right">{formatCurrency(order.subtotal)}</TableCell>
                  </TableRow>
                  <TableRow>
                    <TableCell colSpan={4} className="text-right">Shipping</TableCell>
                    <TableCell className="text-right">{formatCurrency(order.shippingCost)}</TableCell>
                  </TableRow>
                  <TableRow>
                    <TableCell colSpan={4} className="text-right">Tax</TableCell>
                    <TableCell className="text-right">{formatCurrency(order.taxAmount)}</TableCell>
                  </TableRow>
                  <TableRow className="font-bold text-lg">
                    <TableCell colSpan={4} className="text-right">Order Total</TableCell>
                    <TableCell className="text-right">{formatCurrency(order.total)}</TableCell>
                  </TableRow>
                </TableBody>
              </Table>
            </CardContent>
          </Card>
        </div>

        {/* Customer Info & Timeline */}
        <div className="lg:col-span-1 space-y-6">
          <Card>
            <CardHeader>
              <CardTitle>Customer Information</CardTitle>
            </CardHeader>
            <CardContent>
              <p className="font-medium">{order.customerName}</p>
              <p className="text-sm text-muted-foreground">{order.customerEmail}</p>
              <Link href={`/admin/customers/${order.customerId}`} className="text-sm text-primary hover:underline mt-2 block">
                View Customer Profile
              </Link>
            </CardContent>
          </Card>

          <Card>
            <CardHeader>
              <CardTitle>Order Timeline</CardTitle>
            </CardHeader>
            <CardContent>
              <ol className="relative border-l border-gray-200 dark:border-gray-700 ml-4">
                {order.timeline.map((event, index) => (
                  <li key={index} className="mb-6 ml-6">
                    <span className="absolute flex items-center justify-center w-6 h-6 bg-primary-foreground rounded-full -left-3 ring-8 ring-background">
                      {getTimelineIcon(event.status)}
                    </span>
                    <h3 className="flex items-center mb-1 text-lg font-semibold text-foreground">
                      {event.status.charAt(0).toUpperCase() + event.status.slice(1)}
                    </h3>
                    <time className="block mb-2 text-sm font-normal leading-none text-muted-foreground">
                      {formatDate(event.timestamp, { dateStyle: 'medium', timeStyle: 'short' })}
                    </time>
                    <p className="text-base font-normal text-muted-foreground">{event.description}</p>
                  </li>
                ))}
              </ol>
            </CardContent>
          </Card>
        </div>
      </div>
    </div>
  );
}
```

---

## 14. src/components/admin/orders/order-actions.tsx

```typescript
// src/components/admin/orders/order-actions.tsx
'use client';

import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuLabel, DropdownMenuSeparator, DropdownMenuTrigger } from '@/components/ui/dropdown-menu';
import { Button } from '@/components/ui/button';
import { MoreHorizontal, Truck, CheckCircle, XCircle, RefreshCcw, DollarSign } from 'lucide-react';
import { AlertDialog, AlertDialogAction, AlertDialogCancel, AlertDialogContent, AlertDialogDescription, AlertDialogFooter, AlertDialogHeader, AlertDialogTitle, AlertDialogTrigger } from '@/components/ui/alert-dialog';
import { useState } from 'react';
import { trpc } from '@/server/trpc/client';
import { toast } from 'sonner';

interface OrderActionsProps {
  orderId: string;
  currentStatus: string;
  onActionSuccess?: () => void;
}

export function OrderActions({ orderId, currentStatus, onActionSuccess }: OrderActionsProps) {
  const [dialogOpen, setDialogOpen] = useState(false);
  const [actionType, setActionType] = useState<'cancel' | 'refund' | null>(null);

  const updateOrderStatusMutation = trpc.admin.updateOrderStatus.useMutation({
    onSuccess: () => {
      toast.success('Order status updated successfully!');
      onActionSuccess?.();
    },
    onError: (error) => {
      toast.error('Failed to update order status.', { description: error.message });
    },
  });

  const refundOrderMutation = trpc.admin.refundOrder.useMutation({
    onSuccess: () => {
      toast.success('Order refunded successfully!');
      onActionSuccess?.();
    },
    onError: (error) => {
      toast.error('Failed to refund order.', { description: error.message });
    },
  });

  const handleAction = (status: string) => {
    updateOrderStatusMutation.mutate({ orderId, status });
  };

  const handleConfirmAction = () => {
    if (actionType === 'cancel') {
      handleAction('cancelled');
    } else if (actionType === 'refund') {
      refundOrderMutation.mutate(orderId);
    }
    setDialogOpen(false);
    setActionType(null);
  };

  const openConfirmDialog = (type: 'cancel' | 'refund') => {
    setActionType(type);
    setDialogOpen(true);
  };

  const isProcessing = updateOrderStatusMutation.isLoading || refundOrderMutation.isLoading;

  return (
    <>
      <DropdownMenu>
        <DropdownMenuTrigger asChild>
          <Button variant="outline" disabled={isProcessing}>
            Actions <MoreHorizontal className="ml-2 h-4 w-4" />
          </Button>
        </DropdownMenuTrigger>
        <DropdownMenuContent align="end">
          <DropdownMenuLabel>Order Actions</DropdownMenuLabel>
          {currentStatus === 'pending' && (
            <DropdownMenuItem onClick={() => handleAction('processing')} disabled={isProcessing}>
              <RefreshCcw className="mr-2 h-4 w-4" /> Mark as Processing
            </DropdownMenuItem>
          )}
          {(currentStatus === 'pending' || currentStatus === 'processing') && (
            <DropdownMenuItem onClick={() => handleAction('shipped')} disabled={isProcessing}>
              <Truck className="mr-2 h-4 w-4" /> Mark as Shipped
            </DropdownMenuItem>
          )}
          {currentStatus === 'shipped' && (
            <DropdownMenuItem onClick={() => handleAction('delivered')} disabled={isProcessing}>
              <CheckCircle className="mr-2 h-4 w-4" /> Mark as Delivered
            </DropdownMenuItem>
          )}
          {(currentStatus === 'pending' || currentStatus === 'processing' || currentStatus === 'shipped') && (
            <>
              <DropdownMenuSeparator />
              <DropdownMenuItem onClick={() => openConfirmDialog('cancel')} className="text-red-600" disabled={isProcessing}>
                <XCircle className="mr-2 h-4 w-4" /> Cancel Order
              </DropdownMenuItem>
            </>
          )}
          {(currentStatus === 'delivered' || currentStatus === 'shipped') && (
            <DropdownMenuItem onClick={() => openConfirmDialog('refund')} className="text-red-600" disabled={isProcessing}>
              <DollarSign className="mr-2 h-4 w-4" /> Refund Order
            </DropdownMenuItem>
          )}
        </DropdownMenuContent>
      </DropdownMenu>

      <AlertDialog open={dialogOpen} onOpenChange={setDialogOpen}>
        <AlertDialogContent>
          <AlertDialogHeader>
            <AlertDialogTitle>Are you absolutely sure?</AlertDialogTitle>
            <AlertDialogDescription>
              This action cannot be undone. This will {actionType === 'cancel' ? 'cancel the order' : 'process a refund for the order'}.
            </AlertDialogDescription>
          </AlertDialogHeader>
          <AlertDialogFooter>
            <AlertDialogCancel disabled={isProcessing}>Cancel</AlertDialogCancel>
            <AlertDialogAction onClick={handleConfirmAction} disabled={isProcessing} className={actionType === 'cancel' ? 'bg-red-600 hover:bg-red-700' : ''}>
              {isProcessing ? 'Processing...' : `Confirm ${actionType === 'cancel' ? 'Cancellation' : 'Refund'}`}
            </AlertDialogAction>
          </AlertDialogFooter>
        </AlertDialogContent>
      </AlertDialog>
    </>
  );
}
```

---

## 15. src/app/admin/customers/page.tsx

```typescript
// src/app/admin/customers/page.tsx
'use client';

import { useState } from 'react';
import { ColumnDef } from '@tanstack/react-table';
import { DataTable, createSortableColumn, selectionColumn } from '@/components/admin/data-table';
import { trpc } from '@/server/trpc/client';
import { Customer } from '@/server/services/admin-service'; // Assuming Customer type
import { formatCurrency, formatDate } from '@/lib/utils';
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuLabel, DropdownMenuSeparator, DropdownMenuTrigger } from '@/components/ui/dropdown-menu';
import { Button } from '@/components/ui/button';
import { MoreHorizontal, Eye, Mail, Trash } from 'lucide-react';
import Link from 'next/link';
import { useRouter } from 'next/navigation';
import { toast } from 'sonner';
import { AlertDialog, AlertDialogAction, AlertDialogCancel, AlertDialogContent, AlertDialogDescription, AlertDialogFooter, AlertDialogHeader, AlertDialogTitle, AlertDialogTrigger } from '@/components/ui/alert-dialog';

export default function CustomersPage() {
  const router = useRouter();
  const [pagination, setPagination] = useState({ pageIndex: 0, pageSize: 10 });
  const [sorting, setSorting] = useState([]);
  const [columnFilters, setColumnFilters] = useState([]);
  const [rowSelection, setRowSelection] = useState({});

  const { data, isLoading, isError, refetch } = trpc.admin.getCustomers.useQuery({
    page: pagination.pageIndex + 1,
    limit: pagination.pageSize,
    sortBy: sorting.length > 0 ? sorting[0].id : undefined,
    sortOrder: sorting.length > 0 ? (sorting[0].desc ? 'desc' : 'asc') : undefined,
    filters: columnFilters.reduce((acc, filter) => ({ ...acc, [filter.id]: filter.value }), {}),
  });

  const customers = data?.customers || [];
  const totalCount = data?.totalCount || 0;

  const deleteCustomerMutation = trpc.admin.deleteCustomer.useMutation({ // Assuming a deleteCustomer mutation exists
    onSuccess: () => {
      toast.success('Customer deleted successfully!');
      refetch();
    },
    onError: (error) => {
      toast.error('Failed to delete customer.', { description: error.message });
    },
  });

  const handleDelete = (id: string) => {
    deleteCustomerMutation.mutate(id);
  };

  const columns: ColumnDef<Customer>[] = [
    selectionColumn,
    createSortableColumn('name', 'Customer Name', 'name', ({ row }) => (
      <Link href={`/admin/customers/${row.original.id}`} className="font-medium hover:underline">
        {row.getValue('name')}
      </Link>
    )),
    createSortableColumn('email', 'Email', 'email'),
    createSortableColumn('totalOrders', 'Total Orders', 'totalOrders'),
    createSortableColumn('totalSpent', 'Total Spent', 'totalSpent', ({ row }) => formatCurrency(row.getValue('totalSpent'))),
    createSortableColumn('lastOrderDate', 'Last Order', 'lastOrderDate', ({ row }) => row.getValue('lastOrderDate') ? formatDate(row.getValue('lastOrderDate')) : 'N/A'),
    {
      id: 'actions',
      enableHiding: false,
      cell: ({ row }) => (
        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="ghost" className="h-8 w-8 p-0">
              <span className="sr-only">Open menu</span>
              <MoreHorizontal className="h-4 w-4" />
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end">
            <DropdownMenuLabel>Actions</DropdownMenuLabel>
            <DropdownMenuItem onClick={() => router.push(`/admin/customers/${row.original.id}`)}>
              <Eye className="mr-2 h-4 w-4" /> View Details
            </DropdownMenuItem>
            <DropdownMenuItem onClick={() => window.open(`mailto:${row.original.email}`)}>
              <Mail className="mr-2 h-4 w-4" /> Send Email
            </DropdownMenuItem>
            <DropdownMenuSeparator />
            <AlertDialog>
              <AlertDialogTrigger asChild>
                <DropdownMenuItem onSelect={(e) => e.preventDefault()} className="text-red-600">
                  <Trash className="mr-2 h-4 w-4" /> Delete
                </DropdownMenuItem>
              </AlertDialogTrigger>
              <AlertDialogContent>
                <AlertDialogHeader>
                  <AlertDialogTitle>Are you absolutely sure?</AlertDialogTitle>
                  <AlertDialogDescription>
                    This action cannot be undone. This will permanently delete the customer &quot;{row.original.name}&quot;
                    and remove their data from our servers.
                  </AlertDialogDescription>
                </AlertDialogHeader>
                <AlertDialogFooter>
                  <AlertDialogCancel>Cancel</AlertDialogCancel>
                  <AlertDialogAction onClick={() => handleDelete(row.original.id)} className="bg-red-600 hover:bg-red-700">
                    Delete
                  </AlertDialogAction>
                </AlertDialogFooter>
              </AlertDialogContent>
            </AlertDialog>
          </DropdownMenuContent>
        </DropdownMenu>
      ),
    },
  ];

  const bulkDeleteAction = {
    label: 'Delete Selected',
    action: (ids: string[]) => {
      toast.info(`Deleting ${ids.length} customers... (Not fully implemented)`);
      // Implement bulk delete mutation here
    },
    icon: Trash,
    variant: 'destructive' as const,
  };

  return (
    <div className="space-y-6">
      <h1 className="text-3xl font-bold">Customers</h1>
      <DataTable
        columns={columns}
        data={customers}
        totalCount={totalCount}
        pageSize={pagination.pageSize}
        pageIndex={pagination.pageIndex}
        onPaginationChange={setPagination}
        onSortingChange={setSorting}
        onFiltersChange={setColumnFilters}
        onRowSelectionChange={setRowSelection}
        loading={isLoading}
        filterColumnId="name"
        bulkActions={[bulkDeleteAction]}
        exportCsv={async (ids) => {
          toast.info(`Exporting ${ids.length || 'all'} customers...`);
          await new Promise(resolve => setTimeout(resolve, 1000));
          return 'https://example.com/mock-customers-export.csv';
        }}
      />
    </div>
  );
}
```

---

## 16. src/app/admin/customers/[id]/page.tsx

```typescript
// src/app/admin/customers/[id]/page.tsx
'use client';

import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Separator } from '@/components/ui/separator';
import { trpc } from '@/server/trpc/client';
import { formatCurrency, formatDate } from '@/lib/utils';
import { Skeleton } from '@/components/ui/skeleton';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table';
import Link from 'next/link';
import { Mail, Phone, MapPin } from 'lucide-react';

export default function CustomerDetailPage({ params }: { params: { id: string } }) {
  const { id } = params;
  const { data: customer, isLoading: isLoadingCustomer, isError: isErrorCustomer } = trpc.admin.getCustomerById.useQuery(id);
  const { data: customerOrders, isLoading: isLoadingOrders, isError: isErrorOrders } = trpc.admin.getOrders.useQuery({ customerId: id, limit: 10 }); // Assuming getOrders can filter by customerId

  if (isLoadingCustomer) {
    return (
      <div className="space-y-6">
        <Skeleton className="h-10 w-64" />
        <div className="grid gap-6 lg:grid-cols-3">
          <div className="lg:col-span-1 space-y-6">
            <Skeleton className="h-[250px]" />
          </div>
          <div className="lg:col-span-2 space-y-6">
            <Skeleton className="h-[400px]" />
          </div>
        </div>
      </div>
    );
  }

  if (isErrorCustomer || !customer) {
    return (
      <div className="space-y-6">
        <h1 className="text-3xl font-bold">Customer Not Found</h1>
        <p className="text-red-500">Could not load customer with ID: {id}</p>
      </div>
    );
  }

  return (
    <div className="space-y-6">
      <h1 className="text-3xl font-bold">Customer: {customer.name}</h1>

      <div className="grid gap-6 lg:grid-cols-3">
        {/* Customer Info */}
        <div className="lg:col-span-1 space-y-6">
          <Card>
            <CardHeader>
              <CardTitle>Customer Details</CardTitle>
              <CardDescription>General information about the customer.</CardDescription>
            </CardHeader>
            <CardContent className="grid gap-4">
              <div className="flex items-center gap-2">
                <Mail className="h-4 w-4 text-muted-foreground" />
                <p className="font-medium">{customer.email}</p>
              </div>
              {customer.phone && (
                <div className="flex items-center gap-2">
                  <Phone className="h-4 w-4 text-muted-foreground" />
                  <p className="font-medium">{customer.phone}</p>
                </div>
              )}
              {customer.address && (
                <div className="flex items-start gap-2">
                  <MapPin className="h-4 w-4 text-muted-foreground mt-1" />
                  <div>
                    <p>{customer.address.street}</p>
                    <p>{customer.address.city}, {customer.address.state} {customer.address.zip}</p>
                    <p>{customer.address.country}</p>
                  </div>
                </div>
              )}
              <Separator />
              <div>
                <p className="text-sm text-muted-foreground">Total Orders</p>
                <p className="font-medium">{customer.totalOrders}</p>
              </div>
              <div>
                <p className="text-sm text-muted-foreground">Total Spent</p>
                <p className="font-medium">{formatCurrency(customer.totalSpent)}</p>
              </div>
              <div>
                <p className="text-sm text-muted-foreground">Member Since</p>
                <p className="font-medium">{formatDate(customer.memberSince)}</p>
              </div>
            </CardContent>
          </Card>
        </div>

        {/* Orders History */}
        <div className="lg:col-span-2 space-y-6">
          <Card>
            <CardHeader>
              <CardTitle>Order History</CardTitle>
              <CardDescription>Recent orders placed by this customer.</CardDescription>
            </CardHeader>
            <CardContent>
              {isLoadingOrders ? (
                <div className="space-y-4">
                  {Array.from({ length: 5 }).map((_, i) => (
                    <div key={i} className="flex items-center space-x-4">
                      <Skeleton className="h-8 w-8 rounded-full" />
                      <div className="flex-1 space-y-1">
                        <Skeleton className="h-4 w-3/4" />
                        <Skeleton className="h-3 w-1/2" />
                      </div>
                    </div>
                  ))}
                </div>
              ) : isErrorOrders ? (
                <div className="text-red-500">Error loading customer orders.</div>
              ) : customerOrders && customerOrders.orders.length > 0 ? (
                <Table>
                  <TableHeader>
                    <TableRow>
                      <TableHead>Order ID</TableHead>
                      <TableHead>Date</TableHead>
                      <TableHead>Total</TableHead>
                      <TableHead>Status</TableHead>
                      <TableHead className="text-right">Actions</TableHead>
                    </TableRow>
                  </TableHeader>
                  <TableBody>
                    {customerOrders.orders.map((order) => (
                      <TableRow key={order.id}>
                        <TableCell>
                          <Link href={`/admin/orders/${order.id}`} className="hover:underline">
                            #{order.id.substring(0, 8)}
                          </Link>
                        </TableCell>
                        <TableCell>{formatDate(order.orderDate)}</TableCell>
                        <TableCell>{formatCurrency(order.total)}</TableCell>
                        <TableCell>{order.status}</TableCell>
                        <TableCell className="text-right">
                          <Link href={`/admin/orders/${order.id}`} className="text-primary hover:underline text-sm">
                            View
                          </Link>
                        </TableCell>
                      </TableRow>
                    ))}
                  </TableBody>
                </Table>
              ) : (
                <p className="text-muted-foreground">No orders found for this customer.</p>
              )}
            </CardContent>
          </Card>
        </div>
      </div>
    </div>
  );
}
```

---

## 17. src/app/admin/categories/page.tsx

```typescript
// src/app/admin/categories/page.tsx
'use client';

import { useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { PlusCircle, Edit, Trash2, ChevronRight, ChevronDown } from 'lucide-react';
import { trpc } from '@/server/trpc/client';
import { Category } from '@/server/services/admin-service'; // Assuming Category type
import { toast } from 'sonner';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import * as z from 'zod';
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import { Textarea } from '@/components/ui/textarea';
import { AlertDialog, AlertDialogAction, AlertDialogCancel, AlertDialogContent, AlertDialogDescription, AlertDialogFooter, AlertDialogHeader, AlertDialogTitle, AlertDialogTrigger } from '@/components/ui/alert-dialog';
import { Skeleton } from '@/components/ui/skeleton';
// For drag-and-drop, we'll use a simplified approach or a placeholder.
// A full dnd implementation (e.g., with @hello-pangea/dnd) would be too many lines.

const categoryFormSchema = z.object({
  id: z.string().optional(),
  name: z.string().min(2, { message: 'Category name must be at least 2 characters.' }),
  slug: z.string().min(2, { message: 'Slug must be at least 2 characters.' }),
  description: z.string().optional(),
  parentId: z.string().nullable().optional(),
});

type CategoryFormValues = z.infer<typeof categoryFormSchema>;

interface CategoryTreeItemProps {
  category: Category;
  level: number;
  onSelect: (category: Category) => void;
  onDelete: (id: string, name: string) => void;
  selectedCategoryId: string | null;
}

const CategoryTreeItem: React.FC<CategoryTreeItemProps> = ({ category, level, onSelect, onDelete, selectedCategoryId }) => {
  const [isOpen, setIsOpen] = useState(false);
  const isSelected = selectedCategoryId === category.id;

  return (
    <li className="py-1">
      <div
        className={`flex items-center gap-2 p-2 rounded-md cursor-pointer hover:bg-muted ${isSelected ? 'bg-muted text-primary' : ''}`}
        style={{ paddingLeft: `${level * 20 + 8}px` }}
      >
        {category.subCategories && category.subCategories.length > 0 && (
          <Button variant="ghost" size="icon" className="h-6 w-6" onClick={() => setIsOpen(!isOpen)}>
            {isOpen ? <ChevronDown className="h-4 w-4" /> : <ChevronRight className="h-4 w-4" />}
          </Button>
        )}
        <span onClick={() => onSelect(category)} className="flex-1">
          {category.name}
        </span>
        <Button variant="ghost" size="sm" onClick={() => onSelect(category)}>
          <Edit className="h-4 w-4" />
        </Button>
        <Button variant="ghost" size="sm" className="text-red-500" onClick={() => onDelete(category.id, category.name)}>
          <Trash2 className="h-4 w-4" />
        </Button>
      </div>
      {isOpen && category.subCategories && category.subCategories.length > 0 && (
        <ul className="ml-4">
          {category.subCategories.map((subCat) => (
            <CategoryTreeItem
              key={subCat.id}
              category={subCat}
              level={level + 1}
              onSelect={onSelect}
              onDelete={onDelete}

---
_Modello: gemini-2.5-flash (Google AI Studio) | Token: 33282_