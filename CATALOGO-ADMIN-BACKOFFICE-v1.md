# CATALOGO ADMIN-BACKOFFICE v1
§ DOCUMENTAZIONE TECNICA COMPLETA PER PANNELLI AMMINISTRATIVI PROFESSIONALI

---

§ §1. ADMIN LAYOUT & NAVIGATION

§ 1.1 ADMIN LAYOUT STRUCTURE

typescript
// app/admin/layout.tsx
import { AdminSidebar } from '@/components/admin/sidebar';
import { AdminHeader } from '@/components/admin/header';
import { AdminBreadcrumbs } from '@/components/admin/breadcrumbs';
import { AdminNotifications } from '@/components/admin/notifications';
import { AdminProvider } from '@/components/admin/provider';

export default function AdminLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <AdminProvider>
      <div className="min-h-screen bg-gray-50">
        {/* Mobile sidebar backdrop */}
        <div
          id="admin-sidebar-backdrop"
          className="fixed inset-0 z-40 bg-gray-900/50 lg:hidden"
          style={{ display: 'none' }}
        />
        
        {/* Sidebar for desktop */}
        <div className="hidden lg:fixed lg:inset-y-0 lg:z-50 lg:flex lg:w-72 lg:flex-col">
          <AdminSidebar />
        </div>
        
        {/* Mobile sidebar */}
        <div
          id="admin-mobile-sidebar"
          className="fixed inset-y-0 left-0 z-50 w-72 transform bg-white shadow-lg transition-transform duration-300 ease-in-out lg:hidden"
          style={{ transform: 'translateX(-100%)' }}
        >
          <AdminSidebar mobile />
        </div>
        
        {/* Main content */}
        <div className="lg:pl-72">
          <AdminHeader />
          <AdminNotifications />
          
          <main className="py-8">
            <div className="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
              {/* Breadcrumbs - optional based on route */}
              <AdminBreadcrumbs />
              
              {/* Page content */}
              <div className="mt-6">
                {children}
              </div>
            </div>
          </main>
        </div>
      </div>
      
      {/* Script to handle mobile sidebar */}
      <script
        dangerouslySetInnerHTML={{
          __html: `
            function toggleMobileSidebar() {
              const sidebar = document.getElementById('admin-mobile-sidebar');
              const backdrop = document.getElementById('admin-sidebar-backdrop');
              const isOpen = sidebar.style.transform === 'translateX(0px)';
              
              if (isOpen) {
                sidebar.style.transform = 'translateX(-100%)';
                backdrop.style.display = 'none';
              } else {
                sidebar.style.transform = 'translateX(0px)';
                backdrop.style.display = 'block';
              }
            }
            
            // Close sidebar when clicking backdrop
            document.getElementById('admin-sidebar-backdrop')?.addEventListener('click', toggleMobileSidebar);
            
            // Close sidebar on escape key
            document.addEventListener('keydown', (e) => {
              if (e.key === 'Escape') {
                const sidebar = document.getElementById('admin-mobile-sidebar');
                if (sidebar && sidebar.style.transform === 'translateX(0px)') {
                  toggleMobileSidebar();
                }
              }
            });
          `,
        }}
      />
    </AdminProvider>
  );
}

§ 1.2 NAVIGATION CONFIGURATION

typescript
// lib/admin/navigation.ts
import { 
  LayoutDashboard, 
  Users, 
  Package, 
  ShoppingCart, 
  Settings, 
  BarChart3,
  CreditCard,
  MessageSquare,
  Shield,
  FileText,
  Bell,
  HelpCircle
} from 'lucide-react';

export interface NavItem {
  title: string;
  href: string;
  icon?: React.ComponentType<{ className?: string }>;
  badge?: number | string;
  disabled?: boolean;
  children?: NavItem[];
  permission?: string; // Permission required to see this item
  exact?: boolean; // Exact match for active state
}

export interface NavGroup {
  title?: string;
  items: NavItem[];
}

export const adminNavigation: NavGroup[] = [
  {
    items: [
      {
        title: 'Dashboard',
        href: '/admin',
        icon: LayoutDashboard,
        exact: true,
      },
    ],
  },
  {
    title: 'Sales',
    items: [
      {
        title: 'Orders',
        href: '/admin/orders',
        icon: ShoppingCart,
        badge: '12', // Example badge
        children: [
          { title: 'All Orders', href: '/admin/orders' },
          { title: 'Pending', href: '/admin/orders?status=pending' },
          { title: 'Processing', href: '/admin/orders?status=processing' },
          { title: 'Completed', href: '/admin/orders?status=completed' },
        ],
      },
      {
        title: 'Customers',
        href: '/admin/customers',
        icon: Users,
        children: [
          { title: 'All Customers', href: '/admin/customers' },
          { title: 'Groups', href: '/admin/customers/groups' },
          { title: 'Segments', href: '/admin/customers/segments' },
        ],
      },
      {
        title: 'Analytics',
        href: '/admin/analytics',
        icon: BarChart3,
      },
    ],
  },
  {
    title: 'Catalog',
    items: [
      {
        title: 'Products',
        href: '/admin/products',
        icon: Package,
        children: [
          { title: 'All Products', href: '/admin/products' },
          { title: 'Categories', href: '/admin/products/categories' },
          { title: 'Brands', href: '/admin/products/brands' },
          { title: 'Inventory', href: '/admin/products/inventory' },
          { title: 'Collections', href: '/admin/products/collections' },
        ],
      },
      {
        title: 'Discounts',
        href: '/admin/discounts',
        icon: CreditCard,
      },
    ],
  },
  {
    title: 'Content',
    items: [
      {
        title: 'Pages',
        href: '/admin/pages',
        icon: FileText,
      },
      {
        title: 'Blog',
        href: '/admin/blog',
        icon: MessageSquare,
      },
      {
        title: 'Reviews',
        href: '/admin/reviews',
        icon: Bell,
        children: [
          { title: 'All Reviews', href: '/admin/reviews' },
          { title: 'Pending', href: '/admin/reviews?status=pending' },
          { title: 'Flagged', href: '/admin/reviews?status=flagged' },
        ],
      },
    ],
  },
  {
    title: 'Administration',
    items: [
      {
        title: 'Users',
        href: '/admin/users',
        icon: Users,
        permission: 'users.view',
        children: [
          { title: 'All Users', href: '/admin/users' },
          { title: 'Roles', href: '/admin/users/roles', permission: 'roles.view' },
          { title: 'Permissions', href: '/admin/users/permissions', permission: 'permissions.view' },
          { title: 'Activity Log', href: '/admin/users/activity', permission: 'activity.view' },
        ],
      },
      {
        title: 'Settings',
        href: '/admin/settings',
        icon: Settings,
        children: [
          { title: 'General', href: '/admin/settings/general' },
          { title: 'Store', href: '/admin/settings/store' },
          { title: 'Shipping', href: '/admin/settings/shipping' },
          { title: 'Tax', href: '/admin/settings/tax' },
          { title: 'Email', href: '/admin/settings/email' },
          { title: 'Integrations', href: '/admin/settings/integrations' },
        ],
      },
      {
        title: 'Security',
        href: '/admin/security',
        icon: Shield,
        permission: 'security.view',
      },
      {
        title: 'Help & Support',
        href: '/admin/support',
        icon: HelpCircle,
      },
    ],
  },
];

// Helper function to check if a nav item is active
export function isNavItemActive(item: NavItem, pathname: string): boolean {
  if (item.exact) {
    return pathname === item.href;
  }
  
  if (item.children) {
    return item.children.some(child => 
      pathname === child.href || pathname.startsWith(child.href + '/')
    );
  }
  
  return pathname.startsWith(item.href + '/') || pathname === item.href;
}

// Flatten navigation for breadcrumbs/search
export function flattenNavigation(navigation: NavGroup[]): NavItem[] {
  const items: NavItem[] = [];
  
  function flatten(itemsArray: NavItem[]) {
    for (const item of itemsArray) {
      items.push(item);
      if (item.children) {
        flatten(item.children);
      }
    }
  }
  
  for (const group of navigation) {
    flatten(group.items);
  }
  
  return items;
}

// Get breadcrumb items for current path
export function getBreadcrumbItems(pathname: string): NavItem[] {
  const allItems = flattenNavigation(adminNavigation);
  const items: NavItem[] = [];
  let currentPath = pathname;
  
  while (currentPath) {
    const item = allItems.find(item => 
      item.href === currentPath || 
      (item.href !== '/admin' && currentPath.startsWith(item.href))
    );
    
    if (item) {
      items.unshift(item);
      // Move up to parent path
      if (item.href === '/admin') break;
      currentPath = item.href.split('/').slice(0, -1).join('/') || '/admin';
    } else {
      break;
    }
  }
  
  return items;
}

§ 1.3 ADMIN SHELL COMPONENTS

typescript
// components/admin/sidebar.tsx
'use client';

import { useState } from 'react';
import Link from 'next/link';
import { usePathname } from 'next/navigation';
import { ChevronRight, X } from 'lucide-react';
import { adminNavigation, isNavItemActive } from '@/lib/admin/navigation';
import { cn } from '@/lib/utils';
import { checkPermission } from '@/lib/auth/permissions';

interface AdminSidebarProps {
  mobile?: boolean;
}

export function AdminSidebar({ mobile = false }: AdminSidebarProps) {
  const pathname = usePathname();
  const [expandedGroups, setExpandedGroups] = useState<Set<number>>(new Set());
  
  const toggleGroup = (index: number) => {
    setExpandedGroups(prev => {
      const next = new Set(prev);
      if (next.has(index)) {
        next.delete(index);
      } else {
        next.add(index);
      }
      return next;
    });
  };
  
  const hasPermission = (permission?: string) => {
    if (!permission) return true;
    return checkPermission(permission);
  };
  
  return (
    <div className="flex h-full flex-col border-r border-gray-200 bg-white">
      {/* Logo/Header */}
      <div className="flex h-16 items-center justify-between border-b border-gray-200 px-6">
        <Link href="/admin" className="flex items-center gap-2">
          <div className="h-8 w-8 rounded-lg bg-primary-600" />
          <span className="text-lg font-semibold text-gray-900">Admin Panel</span>
        </Link>
        {mobile && (
          <button
            onClick={() => {
              const event = new CustomEvent('toggle-mobile-sidebar');
              window.dispatchEvent(event);
            }}
            className="rounded-md p-1 text-gray-500 hover:bg-gray-100 hover:text-gray-900"
          >
            <X className="h-5 w-5" />
          </button>
        )}
      </div>
      
      {/* Navigation */}
      <nav className="flex-1 overflow-y-auto py-4">
        <div className="space-y-1 px-3">
          {adminNavigation.map((group, groupIndex) => {
            const hasItems = group.items.some(item => hasPermission(item.permission));
            if (!hasItems) return null;
            
            return (
              <div key={groupIndex} className="space-y-1">
                {group.title && (
                  <div className="px-3 py-2">
                    <h3 className="text-xs font-semibold uppercase tracking-wider text-gray-500">
                      {group.title}
                    </h3>
                  </div>
                )}
                
                <div className="space-y-1">
                  {group.items
                    .filter(item => hasPermission(item.permission))
                    .map((item, itemIndex) => {
                      const isActive = isNavItemActive(item, pathname);
                      const hasChildren = item.children && item.children.length > 0;
                      const isExpanded = expandedGroups.has(itemIndex);
                      
                      return (
                        <div key={item.href}>
                          <Link
                            href={hasChildren ? '#' : item.href}
                            onClick={(e) => {
                              if (hasChildren) {
                                e.preventDefault();
                                toggleGroup(itemIndex);
                              }
                            }}
                            className={cn(
                              'flex items-center justify-between rounded-lg px-3 py-2 text-sm font-medium transition-colors',
                              isActive
                                ? 'bg-primary-50 text-primary-700'
                                : 'text-gray-700 hover:bg-gray-100 hover:text-gray-900',
                              item.disabled && 'cursor-not-allowed opacity-50'
                            )}
                          >
                            <div className="flex items-center gap-3">
                              {item.icon && (
                                <item.icon
                                  className={cn(
                                    'h-5 w-5',
                                    isActive ? 'text-primary-600' : 'text-gray-400'
                                  )}
                                />
                              )}
                              <span>{item.title}</span>
                            </div>
                            
                            <div className="flex items-center gap-1">
                              {item.badge && (
                                <span className="inline-flex items-center rounded-full bg-primary-100 px-2 py-0.5 text-xs font-medium text-primary-800">
                                  {item.badge}
                                </span>
                              )}
                              {hasChildren && (
                                <ChevronRight
                                  className={cn(
                                    'h-4 w-4 transition-transform',
                                    isExpanded && 'rotate-90'
                                  )}
                                />
                              )}
                            </div>
                          </Link>
                          
                          {/* Children */}
                          {hasChildren && isExpanded && (
                            <div className="ml-6 mt-1 space-y-1 border-l border-gray-200 pl-3">
                              {item.children
                                ?.filter(child => hasPermission(child.permission))
                                .map((child) => {
                                  const isChildActive = isNavItemActive(child, pathname);
                                  return (
                                    <Link
                                      key={child.href}
                                      href={child.href}
                                      className={cn(
                                        'flex items-center gap-3 rounded-lg px-3 py-2 text-sm transition-colors',
                                        isChildActive
                                          ? 'bg-primary-50 text-primary-700'
                                          : 'text-gray-700 hover:bg-gray-100 hover:text-gray-900',
                                        child.disabled && 'cursor-not-allowed opacity-50'
                                      )}
                                    >
                                      <div className="h-1.5 w-1.5 rounded-full bg-current opacity-50" />
                                      <span>{child.title}</span>
                                    </Link>
                                  );
                                })}
                            </div>
                          )}
                        </div>
                      );
                    })}
                </div>
              </div>
            );
          })}
        </div>
      </nav>
      
      {/* Footer */}
      <div className="border-t border-gray-200 p-4">
        <div className="flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="h-8 w-8 rounded-full bg-gray-200" />
            <div className="text-sm">
              <p className="font-medium text-gray-900">John Doe</p>
              <p className="text-gray-500">Administrator</p>
            </div>
          </div>
          <Link
            href="/admin/settings/profile"
            className="rounded-md p-1 text-gray-500 hover:bg-gray-100 hover:text-gray-900"
          >
            <Settings className="h-5 w-5" />
          </Link>
        </div>
      </div>
    </div>
  );
}

typescript
// components/admin/header.tsx
'use client';

import { useState } from 'react';
import { Search, Bell, HelpCircle, Menu } from 'lucide-react';
import { GlobalSearch } from './global-search';
import { NotificationCenter } from './notifications/notification-center';
import { cn } from '@/lib/utils';

export function AdminHeader() {
  const [searchOpen, setSearchOpen] = useState(false);
  const [notificationsOpen, setNotificationsOpen] = useState(false);
  
  return (
    <header className="sticky top-0 z-40 border-b border-gray-200 bg-white px-4 shadow-sm sm:px-6 lg:px-8">
      <div className="flex h-16 items-center justify-between">
        {/* Left: Mobile menu button */}
        <button
          onClick={() => {
            const event = new CustomEvent('toggle-mobile-sidebar');
            window.dispatchEvent(event);
          }}
          className="rounded-md p-2 text-gray-500 hover:bg-gray-100 hover:text-gray-900 lg:hidden"
        >
          <Menu className="h-5 w-5" />
        </button>
        
        {/* Search */}
        <div className="flex-1 max-w-2xl">
          <GlobalSearch />
        </div>
        
        {/* Right: Actions */}
        <div className="flex items-center gap-2">
          {/* Help */}
          <button
            onClick={() => window.open('/admin/help', '_blank')}
            className="hidden rounded-md p-2 text-gray-500 hover:bg-gray-100 hover:text-gray-900 sm:block"
            title="Help & Documentation"
          >
            <HelpCircle className="h-5 w-5" />
          </button>
          
          {/* Notifications */}
          <div className="relative">
            <button
              onClick={() => setNotificationsOpen(!notificationsOpen)}
              className="relative rounded-md p-2 text-gray-500 hover:bg-gray-100 hover:text-gray-900"
              title="Notifications"
            >
              <Bell className="h-5 w-5" />
              <span className="absolute -right-0.5 -top-0.5 h-2 w-2 rounded-full bg-red-500" />
            </button>
            
            {notificationsOpen && (
              <>
                <div
                  className="fixed inset-0 z-40"
                  onClick={() => setNotificationsOpen(false)}
                />
                <div className="absolute right-0 top-full z-50 mt-2 w-80">
                  <NotificationCenter onClose={() => setNotificationsOpen(false)} />
                </div>
              </>
            )}
          </div>
          
          {/* User dropdown */}
          <div className="relative">
            <button className="flex items-center gap-3 rounded-lg p-2 hover:bg-gray-100">
              <div className="h-8 w-8 rounded-full bg-gray-200" />
              <div className="hidden text-left sm:block">
                <p className="text-sm font-medium text-gray-900">John Doe</p>
                <p className="text-xs text-gray-500">Admin</p>
              </div>
            </button>
            
            {/* Dropdown menu */}
            <div className="absolute right-0 top-full hidden pt-2 group-hover:block">
              <div className="w-48 rounded-lg border border-gray-200 bg-white py-1 shadow-lg">
                <a
                  href="/admin/settings/profile"
                  className="block px-4 py-2 text-sm text-gray-700 hover:bg-gray-100"
                >
                  Profile
                </a>
                <a
                  href="/admin/settings/security"
                  className="block px-4 py-2 text-sm text-gray-700 hover:bg-gray-100"
                >
                  Security
                </a>
                <div className="border-t border-gray-200" />
                <a
                  href="/logout"
                  className="block px-4 py-2 text-sm text-gray-700 hover:bg-gray-100"
                >
                  Sign out
                </a>
              </div>
            </div>
          </div>
        </div>
      </div>
    </header>
  );
}

typescript
// components/admin/breadcrumbs.tsx
'use client';

import { usePathname } from 'next/navigation';
import Link from 'next/link';
import { ChevronRight, Home } from 'lucide-react';
import { getBreadcrumbItems } from '@/lib/admin/navigation';
import { cn } from '@/lib/utils';

export function AdminBreadcrumbs() {
  const pathname = usePathname();
  
  // Don't show breadcrumbs on dashboard
  if (pathname === '/admin') {
    return null;
  }
  
  const items = getBreadcrumbItems(pathname);
  
  if (items.length === 0) {
    return null;
  }
  
  return (
    <nav className="flex items-center space-x-2 text-sm text-gray-500">
      <Link
        href="/admin"
        className="inline-flex items-center hover:text-gray-700"
      >
        <Home className="h-4 w-4" />
      </Link>
      
      {items.map((item, index) => {
        const isLast = index === items.length - 1;
        
        return (
          <div key={item.href} className="flex items-center space-x-2">
            <ChevronRight className="h-4 w-4" />
            {isLast ? (
              <span className="font-medium text-gray-900">{item.title}</span>
            ) : (
              <Link
                href={item.href}
                className="hover:text-gray-700"
              >
                {item.title}
              </Link>
            )}
          </div>
        );
      })}
    </nav>
  );
}

typescript
// components/admin/page-header.tsx
import { ReactNode } from 'react';
import { cn } from '@/lib/utils';

interface AdminPageHeaderProps {
  title: string;
  description?: string;
  actions?: ReactNode;
  breadcrumb?: boolean;
  className?: string;
}

export function AdminPageHeader({
  title,
  description,
  actions,
  breadcrumb = false,
  className,
}: AdminPageHeaderProps) {
  return (
    <div className={cn('mb-6', className)}>
      {breadcrumb && <AdminBreadcrumbs />}
      
      <div className="mt-4 flex items-center justify-between">
        <div>
          <h1 className="text-2xl font-bold tracking-tight text-gray-900 sm:text-3xl">
            {title}
          </h1>
          {description && (
            <p className="mt-2 text-sm text-gray-500">{description}</p>
          )}
        </div>
        
        {actions && (
          <div className="flex items-center gap-3">{actions}</div>
        )}
      </div>
    </div>
  );
}

typescript
// components/admin/card.tsx
import { ReactNode } from 'react';
import { cn } from '@/lib/utils';

interface AdminCardProps {
  children: ReactNode;
  className?: string;
  padding?: 'none' | 'sm' | 'md' | 'lg';
  title?: string;
  description?: string;
  actions?: ReactNode;
  footer?: ReactNode;
}

export function AdminCard({
  children,
  className,
  padding = 'md',
  title,
  description,
  actions,
  footer,
}: AdminCardProps) {
  const paddingClasses = {
    none: '',
    sm: 'p-4',
    md: 'p-6',
    lg: 'p-8',
  };
  
  return (
    <div className={cn(
      'overflow-hidden rounded-lg border border-gray-200 bg-white shadow-sm',
      className
    )}>
      {/* Header */}
      {(title || description || actions) && (
        <div className="border-b border-gray-200 bg-white px-6 py-4">
          <div className="flex items-center justify-between">
            <div>
              {title && (
                <h3 className="text-lg font-medium text-gray-900">{title}</h3>
              )}
              {description && (
                <p className="mt-1 text-sm text-gray-500">{description}</p>
              )}
            </div>
            {actions && (
              <div className="flex items-center gap-2">{actions}</div>
            )}
          </div>
        </div>
      )}
      
      {/* Content */}
      <div className={cn(paddingClasses[padding])}>
        {children}
      </div>
      
      {/* Footer */}
      {footer && (
        <div className="border-t border-gray-200 bg-gray-50 px-6 py-4">
          {footer}
        </div>
      )}
    </div>
  );
}

---

§ §2. DATA TABLE SYSTEM (CRITICO)

§ 2.1 SERVER-SIDE DATA TABLE ARCHITECTURE

| Feature | Client-side | Server-side | Hybrid | Raccomandazione |
|---------|-------------|-------------|--------|-----------------|
| **Pagination** | ❌ >100 righe | ✅ Database | ✅ Client cache | **Server-side** per qualsiasi dato reale |
| **Sorting** | ⚠️ Limitato a ~1000 items | ✅ Indexed queries | ✅ Server con client fallback | **Server-side** per performance |
| **Filtering** | ❌ Lento, UI blocking | ✅ WHERE clause efficiente | ✅ Server-side filters | **Server-side** sempre |
| **Search** | ❌ Terribile performance | ✅ Full-text search | ✅ Server-side solo | **Server-side** con debounce |
| **Total Items** | ✅ Necessario | ✅ Count query | ✅ Count query | **Server-side** |
| **Complexity** | Bassa | Media | Alta | **Server-side** per mantenere semplicità |
| **Cache** | Easy | Manual/CDN | Hybrid | **Server-side** con Redis cache |
| **Best for** | <100 righe statiche | Dati reali | App complesse | **Server-side per admin panels** |

§ 2.2 DATATABLE COMPONENT COMPLETO

typescript
// components/admin/data-table/data-table.tsx
'use client';

import { useState, useEffect } from 'react';
import {
  useReactTable,
  getCoreRowModel,
  getPaginationRowModel,
  getSortedRowModel,
  getFilteredRowModel,
  ColumnDef,
  SortingState,
  ColumnFiltersState,
  VisibilityState,
  RowSelectionState,
  flexRender,
} from '@tanstack/react-table';
import { DataTableToolbar } from './data-table-toolbar';
import { DataTablePagination } from './data-table-pagination';
import { DataTableSkeleton } from './data-table-skeleton';
import { cn } from '@/lib/utils';

interface DataTableProps<TData, TValue> {
  columns: ColumnDef<TData, TValue>[];
  data: TData[];
  total: number;
  page: number;
  pageSize: number;
  totalPages: number;
  onPageChange: (page: number) => void;
  onPageSizeChange: (pageSize: number) => void;
  onSortChange: (sorting: SortingState) => void;
  onFilterChange: (filters: ColumnFiltersState) => void;
  onRowSelectionChange?: (selection: RowSelectionState) => void;
  initialSorting?: SortingState;
  initialFilters?: ColumnFiltersState;
  isLoading?: boolean;
  searchPlaceholder?: string;
  searchColumn?: string;
  filterableColumns?: {
    id: string;
    title: string;
    options: {
      label: string;
      value: string;
      icon?: React.ComponentType<{ className?: string }>;
    }[];
  }[];
  bulkActions?: Array<{
    label: string;
    onClick: (selectedRows: TData[]) => Promise<void>;
    variant?: 'default' | 'destructive';
    icon?: React.ComponentType<{ className?: string }>;
    confirm?: {
      title: string;
      description: string;
    };
  }>;
  emptyState?: React.ReactNode;
  className?: string;
}

export function DataTable<TData, TValue>({
  columns,
  data,
  total,
  page,
  pageSize,
  totalPages,
  onPageChange,
  onPageSizeChange,
  onSortChange,
  onFilterChange,
  onRowSelectionChange,
  initialSorting = [],
  initialFilters = [],
  isLoading = false,
  searchPlaceholder = 'Search...',
  searchColumn,
  filterableColumns,
  bulkActions,
  emptyState,
  className,
}: DataTableProps<TData, TValue>) {
  const [sorting, setSorting] = useState<SortingState>(initialSorting);
  const [columnFilters, setColumnFilters] = useState<ColumnFiltersState>(initialFilters);
  const [columnVisibility, setColumnVisibility] = useState<VisibilityState>({});
  const [rowSelection, setRowSelection] = useState<RowSelectionState>({});
  const [globalFilter, setGlobalFilter] = useState('');
  
  // Sync state changes with parent
  useEffect(() => {
    onSortChange(sorting);
  }, [sorting, onSortChange]);
  
  useEffect(() => {
    onFilterChange(columnFilters);
  }, [columnFilters, onFilterChange]);
  
  useEffect(() => {
    if (onRowSelectionChange) {
      onRowSelectionChange(rowSelection);
    }
  }, [rowSelection, onRowSelectionChange]);
  
  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    onSortingChange: setSorting,
    onColumnFiltersChange: setColumnFilters,
    onColumnVisibilityChange: setColumnVisibility,
    onRowSelectionChange: setRowSelection,
    onGlobalFilterChange: setGlobalFilter,
    state: {
      sorting,
      columnFilters,
      columnVisibility,
      rowSelection,
      globalFilter,
    },
    manualPagination: true,
    manualSorting: true,
    manualFiltering: true,
    pageCount: totalPages,
    enableRowSelection: !!onRowSelectionChange,
  });
  
  const selectedRows = table.getSelectedRowModel().rows.map(row => row.original);
  
  if (isLoading) {
    return <DataTableSkeleton />;
  }
  
  if (data.length === 0 && !isLoading) {
    return emptyState || (
      <div className="rounded-lg border border-dashed border-gray-300 p-12 text-center">
        <div className="mx-auto h-12 w-12 rounded-full bg-gray-100" />
        <h3 className="mt-4 text-sm font-medium text-gray-900">No data found</h3>
        <p className="mt-1 text-sm text-gray-500">
          Try adjusting your filters or search terms.
        </p>
      </div>
    );
  }
  
  return (
    <div className={cn('space-y-4', className)}>
      <DataTableToolbar
        table={table}
        filterableColumns={filterableColumns}
        searchPlaceholder={searchPlaceholder}
        searchColumn={searchColumn}
        bulkActions={bulkActions}
        selectedRows={selectedRows}
      />
      
      <div className="overflow-hidden rounded-md border border-gray-200 bg-white">
        <div className="overflow-x-auto">
          <table className="w-full divide-y divide-gray-200">
            <thead className="bg-gray-50">
              {table.getHeaderGroups().map(headerGroup => (
                <tr key={headerGroup.id}>
                  {headerGroup.headers.map(header => {
                    return (
                      <th
                        key={header.id}
                        className="px-6 py-3 text-left text-xs font-medium uppercase tracking-wider text-gray-500"
                        style={{
                          width: header.getSize() !== 150 ? header.getSize() : undefined,
                        }}
                      >
                        {header.isPlaceholder
                          ? null
                          : flexRender(
                              header.column.columnDef.header,
                              header.getContext()
                            )}
                      </th>
                    );
                  })}
                </tr>
              ))}
            </thead>
            <tbody className="divide-y divide-gray-200 bg-white">
              {table.getRowModel().rows.map(row => (
                <tr
                  key={row.id}
                  className={cn(
                    'hover:bg-gray-50',
                    row.getIsSelected() && 'bg-primary-50'
                  )}
                >
                  {row.getVisibleCells().map(cell => (
                    <td
                      key={cell.id}
                      className="whitespace-nowrap px-6 py-4 text-sm text-gray-900"
                    >
                      {flexRender(
                        cell.column.columnDef.cell,
                        cell.getContext()
                      )}
                    </td>
                  ))}
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      </div>
      
      <DataTablePagination
        table={table}
        total={total}
        page={page}
        pageSize={pageSize}
        totalPages={totalPages}
        onPageChange={onPageChange}
        onPageSizeChange={onPageSizeChange}
      />
    </div>
  );
}

typescript
// components/admin/data-table/data-table-pagination.tsx
'use client';

import { Table } from '@tanstack/react-table';
import {
  ChevronLeft,
  ChevronRight,
  ChevronsLeft,
  ChevronsRight,
} from 'lucide-react';
import { Button } from '@/components/ui/button';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';

interface DataTablePaginationProps<TData> {
  table: Table<TData>;
  total: number;
  page: number;
  pageSize: number;
  totalPages: number;
  onPageChange: (page: number) => void;
  onPageSizeChange: (pageSize: number) => void;
}

export function DataTablePagination<TData>({
  table,
  total,
  page,
  pageSize,
  totalPages,
  onPageChange,
  onPageSizeChange,
}: DataTablePaginationProps<TData>) {
  const pageSizeOptions = [10, 20, 30, 40, 50];
  
  return (
    <div className="flex items-center justify-between px-2">
      <div className="flex-1 text-sm text-gray-500">
        {table.getFilteredSelectedRowModel().rows.length > 0 ? (
          <div className="flex items-center gap-2">
            {table.getFilteredSelectedRowModel().rows.length} of{' '}
            {total} row(s) selected.
          </div>
        ) : (
          <div>
            Showing {(page - 1) * pageSize + 1} to{' '}
            {Math.min(page * pageSize, total)} of {total} results
          </div>
        )}
      </div>
      
      <div className="flex items-center space-x-6 lg:space-x-8">
        <div className="flex items-center space-x-2">
          <p className="text-sm font-medium">Rows per page</p>
          <Select
            value={`${pageSize}`}
            onValueChange={(value) => {
              onPageSizeChange(Number(value));
            }}
          >
            <SelectTrigger className="h-8 w-[70px]">
              <SelectValue placeholder={pageSize} />
            </SelectTrigger>
            <SelectContent side="top">
              {pageSizeOptions.map((size) => (
                <SelectItem key={size} value={`${size}`}>
                  {size}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
        </div>
        
        <div className="flex w-[100px] items-center justify-center text-sm font-medium">
          Page {page} of {totalPages}
        </div>
        
        <div className="flex items-center space-x-2">
          <Button
            variant="outline"
            className="hidden h-8 w-8 p-0 lg:flex"
            onClick={() => onPageChange(1)}
            disabled={page <= 1}
          >
            <span className="sr-only">Go to first page</span>
            <ChevronsLeft className="h-4 w-4" />
          </Button>
          <Button
            variant="outline"
            className="h-8 w-8 p-0"
            onClick={() => onPageChange(page - 1)}
            disabled={page <= 1}
          >
            <span className="sr-only">Go to previous page</span>
            <ChevronLeft className="h-4 w-4" />
          </Button>
          <Button
            variant="outline"
            className="h-8 w-8 p-0"
            onClick={() => onPageChange(page + 1)}
            disabled={page >= totalPages}
          >
            <span className="sr-only">Go to next page</span>
            <ChevronRight className="h-4 w-4" />
          </Button>
          <Button
            variant="outline"
            className="hidden h-8 w-8 p-0 lg:flex"
            onClick={() => onPageChange(totalPages)}
            disabled={page >= totalPages}
          >
            <span className="sr-only">Go to last page</span>
            <ChevronsRight className="h-4 w-4" />
          </Button>
        </div>
      </div>
    </div>
  );
}

typescript
// components/admin/data-table/data-table-toolbar.tsx
'use client';

import { Table } from '@tanstack/react-table';
import { Search, X } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { DataTableFacetedFilter } from './data-table-faceted-filter';
import { DataTableViewOptions } from './data-table-view-options';
import { BulkActions } from './bulk-actions';

interface DataTableToolbarProps<TData> {
  table: Table<TData>;
  filterableColumns?: {
    id: string;
    title: string;
    options: {
      label: string;
      value: string;
      icon?: React.ComponentType<{ className?: string }>;
    }[];
  }[];
  searchPlaceholder?: string;
  searchColumn?: string;
  bulkActions?: Array<{
    label: string;
    onClick: (selectedRows: TData[]) => Promise<void>;
    variant?: 'default' | 'destructive';
    icon?: React.ComponentType<{ className?: string }>;
    confirm?: {
      title: string;
      description: string;
    };
  }>;
  selectedRows: TData[];
}

export function DataTableToolbar<TData>({
  table,
  filterableColumns,
  searchPlaceholder,
  searchColumn,
  bulkActions,
  selectedRows,
}: DataTableToolbarProps<TData>) {
  const isFiltered = table.getState().columnFilters.length > 0;
  const hasSelectedRows = selectedRows.length > 0;
  
  const searchColumnValue = searchColumn || 
    (table.getAllColumns().find(col => col.id === 'name')?.id);
  
  return (
    <div className="flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
      <div className="flex flex-1 flex-col gap-2 sm:flex-row sm:items-center">
        {/* Search */}
        {searchColumnValue && (
          <div className="relative w-full sm:w-64">
            <Search className="absolute left-2.5 top-2.5 h-4 w-4 text-gray-500" />
            <Input
              placeholder={searchPlaceholder}
              value={(table.getColumn(searchColumnValue)?.getFilterValue() as string) ?? ''}
              onChange={(event) =>
                table.getColumn(searchColumnValue)?.setFilterValue(event.target.value)
              }
              className="pl-8"
            />
          </div>
        )}
        
        {/* Filters */}
        {filterableColumns?.map((column) => {
          const tableColumn = table.getColumn(column.id);
          if (!tableColumn) return null;
          
          const selectedValues = new Set(tableColumn.getFilterValue() as string[]);
          
          return (
            <DataTableFacetedFilter
              key={column.id}
              column={tableColumn}
              title={column.title}
              options={column.options}
              selectedValues={selectedValues}
            />
          );
        })}
        
        {/* Clear filters */}
        {isFiltered && (
          <Button
            variant="ghost"
            onClick={() => table.resetColumnFilters()}
            className="h-10 px-3"
          >
            Reset
            <X className="ml-2 h-4 w-4" />
          </Button>
        )}
      </div>
      
      <div className="flex items-center gap-2">
        {/* Bulk actions */}
        {hasSelectedRows && bulkActions && (
          <BulkActions
            actions={bulkActions}
            selectedRows={selectedRows}
            onClearSelection={() => table.resetRowSelection()}
          />
        )}
        
        {/* Column visibility */}
        <DataTableViewOptions table={table} />
      </div>
    </div>
  );
}

typescript
// components/admin/data-table/data-table-column-header.tsx
'use client';

import { Column } from '@tanstack/react-table';
import { ChevronsUpDown, ChevronUp, ChevronDown } from 'lucide-react';
import { cn } from '@/lib/utils';
import { Button } from '@/components/ui/button';

interface DataTableColumnHeaderProps<TData, TValue>
  extends React.HTMLAttributes<HTMLDivElement> {
  column: Column<TData, TValue>;
  title: string;
}

export function DataTableColumnHeader<TData, TValue>({
  column,
  title,
  className,
}: DataTableColumnHeaderProps<TData, TValue>) {
  if (!column.getCanSort()) {
    return <div className={cn(className)}>{title}</div>;
  }
  
  return (
    <div className={cn('flex items-center space-x-2', className)}>
      <Button
        variant="ghost"
        size="sm"
        className="-ml-3 h-8 data-[state=open]:bg-accent"
        onClick={() => column.toggleSorting()}
      >
        <span>{title}</span>
        {column.getIsSorted() === 'desc' ? (
          <ChevronDown className="ml-2 h-4 w-4" />
        ) : column.getIsSorted() === 'asc' ? (
          <ChevronUp className="ml-2 h-4 w-4" />
        ) : (
          <ChevronsUpDown className="ml-2 h-4 w-4" />
        )}
      </Button>
    </div>
  );
}

typescript
// components/admin/data-table/data-table-row-actions.tsx
'use client';

import { Row } from '@tanstack/react-table';
import { MoreHorizontal, Pencil, Trash, Eye } from 'lucide-react';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { Button } from '@/components/ui/button';

interface DataTableRowActionsProps<TData> {
  row: Row<TData>;
  onView?: (row: TData) => void;
  onEdit?: (row: TData) => void;
  onDelete?: (row: TData) => void;
  customActions?: Array<{
    label: string;
    icon?: React.ComponentType<{ className?: string }>;
    onClick: (row: TData) => void;
    variant?: 'default' | 'destructive';
  }>;
}

export function DataTableRowActions<TData>({
  row,
  onView,
  onEdit,
  onDelete,
  customActions = [],
}: DataTableRowActionsProps<TData>) {
  const actions = [];
  
  if (onView) {
    actions.push({
      label: 'View',
      icon: Eye,
      onClick: onView,
    });
  }
  
  if (onEdit) {
    actions.push({
      label: 'Edit',
      icon: Pencil,
      onClick: onEdit,
    });
  }
  
  if (customActions.length > 0) {
    actions.push(...customActions);
  }
  
  if (onDelete) {
    actions.push({
      label: 'Delete',
      icon: Trash,
      onClick: onDelete,
      variant: 'destructive' as const,
    });
  }
  
  if (actions.length === 0) {
    return null;
  }
  
  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button
          variant="ghost"
          className="flex h-8 w-8 p-0 data-[state=open]:bg-muted"
        >
          <MoreHorizontal className="h-4 w-4" />
          <span className="sr-only">Open menu</span>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end" className="w-[160px]">
        {actions.map((action, index) => {
          const Icon = action.icon;
          return (
            <DropdownMenuItem
              key={index}
              onClick={() => action.onClick(row.original)}
              className={action.variant === 'destructive' ? 'text-red-600' : ''}
            >
              {Icon && <Icon className="mr-2 h-4 w-4" />}
              {action.label}
            </DropdownMenuItem>
          );
        })}
      </DropdownMenuContent>
    </DropdownMenu>
  );
}

typescript
// components/admin/data-table/data-table-faceted-filter.tsx
'use client';

import { Column } from '@tanstack/react-table';
import { Check } from 'lucide-react';
import * as React from 'react';
import {
  Popover,
  PopoverContent,
  PopoverTrigger,
} from '@/components/ui/popover';
import { Button } from '@/components/ui/button';
import {
  Command,
  CommandEmpty,
  CommandGroup,
  CommandInput,
  CommandItem,
  CommandList,
  CommandSeparator,
} from '@/components/ui/command';
import { cn } from '@/lib/utils';

interface DataTableFacetedFilterProps<TData, TValue> {
  column?: Column<TData, TValue>;
  title?: string;
  options: {
    label: string;
    value: string;
    icon?: React.ComponentType<{ className?: string }>;
  }[];
  selectedValues: Set<string>;
}

export function DataTableFacetedFilter<TData, TValue>({
  column,
  title,
  options,
  selectedValues,
}: DataTableFacetedFilterProps<TData, TValue>) {
  return (
    <Popover>
      <PopoverTrigger asChild>
        <Button variant="outline" size="sm" className="h-10 border-dashed">
          {title}
          {selectedValues.size > 0 && (
            <>
              <div className="ml-2 h-4 w-px bg-gray-300" />
              <span className="ml-2 text-xs">
                {selectedValues.size} selected
              </span>
            </>
          )}
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-[200px] p-0" align="start">
        <Command>
          <CommandInput placeholder={title} />
          <CommandList>
            <CommandEmpty>No results found.</CommandEmpty>
            <CommandGroup>
              {options.map((option) => {
                const isSelected = selectedValues.has(option.value);
                return (
                  <CommandItem
                    key={option.value}
                    onSelect={() => {
                      if (isSelected) {
                        selectedValues.delete(option.value);
                      } else {
                        selectedValues.add(option.value);
                      }
                      const filterValues = Array.from(selectedValues);
                      column?.setFilterValue(
                        filterValues.length ? filterValues : undefined
                      );
                    }}
                  >
                    <div
                      className={cn(
                        'mr-2 flex h-4 w-4 items-center justify-center rounded-sm border border-primary',
                        isSelected
                          ? 'bg-primary text-primary-foreground'
                          : 'opacity-50 [&_svg]:invisible'
                      )}
                    >
                      <Check className="h-4 w-4" />
                    </div>
                    {option.icon && (
                      <option.icon className="mr-2 h-4 w-4 text-gray-500" />
                    )}
                    <span>{option.label}</span>
                    {selectedValues.has(option.value) && (
                      <span className="ml-auto flex h-4 w-4 items-center justify-center">
                        <Check className="h-4 w-4" />
                      </span>
                    )}
                  </CommandItem>
                );
              })}
            </CommandGroup>
            {selectedValues.size > 0 && (
              <>
                <CommandSeparator />
                <CommandGroup>
                  <CommandItem
                    onSelect={() => column?.setFilterValue(undefined)}
                    className="justify-center text-center"
                  >
                    Clear filters
                  </CommandItem>
                </CommandGroup>
              </>
            )}
          </CommandList>
        </Command>
      </PopoverContent>
    </Popover>
  );
}

typescript
// components/admin/data-table/data-table-view-options.tsx
'use client';

import { Table } from '@tanstack/react-table';
import { Eye } from 'lucide-react';
import {
  DropdownMenu,
  DropdownMenuCheckboxItem,
  DropdownMenuContent,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { Button } from '@/components/ui/button';

interface DataTableViewOptionsProps<TData> {
  table: Table<TData>;
}

export function DataTableViewOptions<TData>({
  table,
}: DataTableViewOptionsProps<TData>) {
  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="outline" size="sm" className="ml-auto h-10">
          <Eye className="mr-2 h-4 w-4" />
          View
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end" className="w-[150px]">
        <DropdownMenuLabel>Toggle columns</DropdownMenuLabel>
        <DropdownMenuSeparator />
        {table
          .getAllColumns()
          .filter(
            (column) =>
              typeof column.accessorFn !== 'undefined' && column.getCanHide()
          )
          .map((column) => {
            return (
              <DropdownMenuCheckboxItem
                key={column.id}
                className="capitalize"
                checked={column.getIsVisible()}
                onCheckedChange={(value) => column.toggleVisibility(!!value)}
              >
                {column.id}
              </DropdownMenuCheckboxItem>
            );
          })}
      </DropdownMenuContent>
    </DropdownMenu>
  );
}

typescript
// components/admin/data-table/data-table-skeleton.tsx
export function DataTableSkeleton() {
  return (
    <div className="space-y-4">
      {/* Toolbar skeleton */}
      <div className="flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
        <div className="h-10 w-64 rounded-md bg-gray-200" />
        <div className="flex gap-2">
          <div className="h-10 w-24 rounded-md bg-gray-200" />
          <div className="h-10 w-24 rounded-md bg-gray-200" />
        </div>
      </div>
      
      {/* Table skeleton */}
      <div className="overflow-hidden rounded-md border border-gray-200">
        <div className="border-b border-gray-200 bg-gray-50 px-6 py-3">
          <div className="flex gap-6">
            {Array.from({ length: 5 }).map((_, i) => (
              <div
                key={i}
                className="h-4 flex-1 rounded bg-gray-200"
                style={{ width: `${Math.random() * 100 + 50}px` }}
              />
            ))}
          </div>
        </div>
        
        <div className="divide-y divide-gray-200">
          {Array.from({ length: 5 }).map((_, rowIndex) => (
            <div key={rowIndex} className="px-6 py-4">
              <div className="flex gap-6">
                {Array.from({ length: 5 }).map((_, colIndex) => (
                  <div
                    key={colIndex}
                    className="h-4 flex-1 rounded bg-gray-100"
                    style={{ width: `${Math.random() * 100 + 50}px` }}
                  />
                ))}
              </div>
            </div>
          ))}
        </div>
      </div>
      
      {/* Pagination skeleton */}
      <div className="flex items-center justify-between">
        <div className="h-4 w-32 rounded bg-gray-200" />
        <div className="flex gap-2">
          <div className="h-8 w-8 rounded bg-gray-200" />
          <div className="h-8 w-8 rounded bg-gray-200" />
          <div className="h-8 w-8 rounded bg-gray-200" />
          <div className="h-8 w-8 rounded bg-gray-200" />
        </div>
      </div>
    </div>
  );
}

§ 2.3 SERVER ACTION PER DATA FETCHING

typescript
// lib/actions/data-table.ts
'use server';

import { PrismaClient } from '@prisma/client';
import { z } from 'zod';
import { unstable_cache } from 'next/cache';

const prisma = new PrismaClient();

// ============ VALIDATION SCHEMAS ============
const TableParamsSchema = z.object({
  page: z.number().int().positive().default(1),
  pageSize: z.number().int().min(1).max(100).default(20),
  sortBy: z.string().optional(),
  sortOrder: z.enum(['asc', 'desc']).optional(),
  filters: z.record(z.any()).optional(),
  search: z.string().optional(),
});

export type TableParams = z.infer<typeof TableParamsSchema>;

export interface TableResult<T> {
  data: T[];
  total: number;
  page: number;
  pageSize: number;
  totalPages: number;
}

// ============ TABLE DATA FETCHER ============
export async function getTableData<T>(
  model: string,
  params: TableParams,
  config: {
    searchFields?: string[];
    relations?: Record<string, boolean>;
    select?: Record<string, boolean>;
    where?: any;
    transform?: (data: any) => T;
  } = {}
): Promise<TableResult<T>> {
  try {
    // Validate params
    const validated = TableParamsSchema.parse(params);
    const { page, pageSize, sortBy, sortOrder, filters, search } = validated;
    
    // Build where clause
    let where: any = { ...config.where };
    
    // Apply search
    if (search && config.searchFields && config.searchFields.length > 0) {
      where.OR = config.searchFields.map(field => ({
        [field]: {
          contains: search,
          mode: 'insensitive' as const,
        },
      }));
    }
    
    // Apply filters
    if (filters) {
      Object.entries(filters).forEach(([key, value]) => {
        if (value !== undefined && value !== null && value !== '') {
          if (Array.isArray(value) && value.length > 0) {
            where[key] = { in: value };
          } else if (typeof value === 'boolean') {
            where[key] = value;
          } else if (typeof value === 'string') {
            where[key] = { contains: value, mode: 'insensitive' as const };
          } else if (typeof value === 'object' && value !== null) {
            // Handle date ranges, number ranges, etc.
            if (value.start || value.end) {
              where[key] = {};
              if (value.start) {
                where[key].gte = value.start;
              }
              if (value.end) {
                where[key].lte = value.end;
              }
            }
          } else {
            where[key] = value;
          }
        }
      });
    }
    
    // Calculate skip
    const skip = (page - 1) * pageSize;
    
    // Build orderBy
    let orderBy: any = {};
    if (sortBy) {
      orderBy[sortBy] = sortOrder || 'asc';
    } else {
      orderBy = { createdAt: 'desc' };
    }
    
    // Execute queries in parallel
    const [data, total] = await Promise.all([
      // Get paginated data
      (prisma as any)[model].findMany({
        where,
        orderBy,
        skip,
        take: pageSize,
        include: config.relations,
        select: config.select,
      }),
      // Get total count
      (prisma as any)[model].count({ where }),
    ]);
    
    // Transform data if needed
    const transformedData = config.transform ? data.map(config.transform) : data;
    
    return {
      data: transformedData,
      total,
      page,
      pageSize,
      totalPages: Math.ceil(total / pageSize),
    };
  } catch (error) {
    if (error instanceof z.ZodError) {
      throw new Error(`Invalid table parameters: ${error.errors.map(e => e.message).join(', ')}`);
    }
    throw error;
  }
}

// ============ CACHED VERSION ============
export const getCachedTableData = unstable_cache(
  async <T>(...args: Parameters<typeof getTableData>) => {
    return getTableData<T>(...args);
  },
  ['table-data'],
  {
    revalidate: 60, // 1 minute
    tags: ['table-data'],
  }
);

// ============ PRISMA QUERY BUILDER ============
export function buildPrismaWhereFromFilters(
  filters: Record<string, any>,
  fieldMappings: Record<string, string> = {}
): any {
  let where: any = {};
  
  for (const [key, value] of Object.entries(filters)) {
    const field = fieldMappings[key] || key;
    
    if (value === undefined || value === null || value === '') {
      continue;
    }
    
    if (Array.isArray(value) && value.length > 0) {
      where[field] = { in: value };
    } else if (typeof value === 'boolean') {
      where[field] = value;
    } else if (typeof value === 'string') {
      where[field] = { contains: value, mode: 'insensitive' as const };
    } else if (typeof value === 'object') {
      // Handle operators
      if (value.equals !== undefined) {
        where[field] = { equals: value.equals };
      } else if (value.not !== undefined) {
        where[field] = { not: value.not };
      } else if (value.in !== undefined) {
        where[field] = { in: value.in };
      } else if (value.notIn !== undefined) {
        where[field] = { notIn: value.notIn };
      } else if (value.lt !== undefined) {
        where[field] = { lt: value.lt };
      } else if (value.lte !== undefined) {
        where[field] = { lte: value.lte };
      } else if (value.gt !== undefined) {
        where[field] = { gt: value.gt };
      } else if (value.gte !== undefined) {
        where[field] = { gte: value.gte };
      } else if (value.contains !== undefined) {
        where[field] = { contains: value.contains, mode: 'insensitive' as const };
      } else if (value.startsWith !== undefined) {
        where[field] = { startsWith: value.startsWith, mode: 'insensitive' as const };
      } else if (value.endsWith !== undefined) {
        where[field] = { endsWith: value.endsWith, mode: 'insensitive' as const };
      }
    } else {
      where[field] = value;
    }
  }
  
  return where;
}

// ============ EXAMPLE USAGE ============
/*
// In a server component:
import { getTableData } from '@/lib/actions/data-table';

export async function UsersTable() {
  const params = {
    page: 1,
    pageSize: 20,
    sortBy: 'createdAt',
    sortOrder: 'desc',
    filters: { status: 'active' },
    search: 'john',
  };
  
  const result = await getTableData<User>('user', params, {
    searchFields: ['name', 'email', 'phone'],
    relations: {
      role: true,
      profile: true,
    },
    select: {
      id: true,
      name: true,
      email: true,
      status: true,
      createdAt: true,
      role: {
        select: {
          name: true,
        },
      },
    },
    transform: (user) => ({
      ...user,
      roleName: user.role?.name,
    }),
  });
  
  return (
    <DataTable
      data={result.data}
      total={result.total}
      page={result.page}
      pageSize={result.pageSize}
      totalPages={result.totalPages}
      // ... rest of props
    />
  );
}
*/

§ 2.4 COLUMN DEFINITION PATTERN

typescript
// lib/utils/table-columns.ts
import { ColumnDef } from '@tanstack/react-table';
import { DataTableColumnHeader } from '@/components/admin/data-table/data-table-column-header';
import { DataTableRowActions } from '@/components/admin/data-table/data-table-row-actions';
import { StatusBadge } from '@/components/admin/status-badge';
import { Checkbox } from '@/components/ui/checkbox';
import { Badge } from '@/components/ui/badge';
import { formatDate } from '@/lib/utils';

// ============ EXAMPLE: USER COLUMNS ============
export type User = {
  id: string;
  name: string;
  email: string;
  role: string;
  status: 'active' | 'inactive' | 'suspended';
  lastLogin: Date | null;
  createdAt: Date;
};

export const userColumns: ColumnDef<User>[] = [
  // Selection column
  {
    id: 'select',
    header: ({ table }) => (
      <Checkbox
        checked={table.getIsAllPageRowsSelected()}
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
  },
  
  // Name column
  {
    accessorKey: 'name',
    header: ({ column }) => (
      <DataTableColumnHeader column={column} title="Name" />
    ),
    cell: ({ row }) => {
      const name = row.getValue('name') as string;
      const email = row.original.email;
      
      return (
        <div className="flex items-center gap-3">
          <div className="h-8 w-8 rounded-full bg-gray-200" />
          <div>
            <div className="font-medium text-gray-900">{name}</div>
            <div className="text-sm text-gray-500">{email}</div>
          </div>
        </div>
      );
    },
    filterFn: 'includesString',
  },
  
  // Role column
  {
    accessorKey: 'role',
    header: 'Role',
    cell: ({ row }) => {
      const role = row.getValue('role') as string;
      
      const roleColors: Record<string, string> = {
        admin: 'bg-purple-100 text-purple-800',
        user: 'bg-gray-100 text-gray-800',
        moderator: 'bg-blue-100 text-blue-800',
        support: 'bg-green-100 text-green-800',
      };
      
      return (
        <Badge className={roleColors[role] || 'bg-gray-100 text-gray-800'}>
          {role}
        </Badge>
      );
    },
    filterFn: (row, id, value) => {
      return value.includes(row.getValue(id));
    },
  },
  
  // Status column
  {
    accessorKey: 'status',
    header: 'Status',
    cell: ({ row }) => {
      const status = row.getValue('status') as string;
      return <StatusBadge status={status} />;
    },
    filterFn: (row, id, value) => {
      return value.includes(row.getValue(id));
    },
  },
  
  // Last login column
  {
    accessorKey: 'lastLogin',
    header: ({ column }) => (
      <DataTableColumnHeader column={column} title="Last Login" />
    ),
    cell: ({ row }) => {
      const date = row.getValue('lastLogin') as Date | null;
      
      if (!date) {
        return <span className="text-gray-500">Never</span>;
      }
      
      return (
        <div className="text-sm text-gray-900">
          {formatDate(date, 'MMM d, yyyy')}
        </div>
      );
    },
  },
  
  // Created at column
  {
    accessorKey: 'createdAt',
    header: ({ column }) => (
      <DataTableColumnHeader column={column} title="Joined" />
    ),
    cell: ({ row }) => {
      const date = row.getValue('createdAt') as Date;
      return (
        <div className="text-sm text-gray-900">
          {formatDate(date, 'MMM d, yyyy')}
        </div>
      );
    },
  },
  
  // Actions column
  {
    id: 'actions',
    cell: ({ row }) => (
      <DataTableRowActions
        row={row}
        onView={(user) => {
          // Navigate to user detail
          window.location.href = `/admin/users/${user.id}`;
        }}
        onEdit={(user) => {
          // Navigate to edit page
          window.location.href = `/admin/users/${user.id}/edit`;
        }}
        onDelete={async (user) => {
          if (confirm(`Are you sure you want to delete ${user.name}?`)) {
            // Call delete API
            await fetch(`/api/admin/users/${user.id}`, {
              method: 'DELETE',
            });
            window.location.reload();
          }
        }}
        customActions={[
          {
            label: 'Impersonate',
            onClick: (user) => {
              // Start impersonation
              fetch(`/api/admin/users/${user.id}/impersonate`, {
                method: 'POST',
              }).then(() => {
                window.location.href = '/';
              });
            },
          },
          {
            label: 'Reset Password',
            onClick: (user) => {
              // Open reset password modal
              console.log('Reset password for', user.id);
            },
          },
        ]}
      />
    ),
  },
];

// ============ EXAMPLE: PRODUCT COLUMNS ============
export type Product = {
  id: string;
  name: string;
  sku: string;
  category: string;
  price: number;
  stock: number;
  status: 'draft' | 'active' | 'out_of_stock' | 'discontinued';
  createdAt: Date;
};

export const productColumns: ColumnDef<Product>[] = [
  {
    id: 'select',
    header: ({ table }) => (
      <Checkbox
        checked={table.getIsAllPageRowsSelected()}
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
  },
  
  {
    accessorKey: 'name',
    header: ({ column }) => (
      <DataTableColumnHeader column={column} title="Product" />
    ),
    cell: ({ row }) => {
      const name = row.getValue('name') as string;
      const sku = row.original.sku;
      
      return (
        <div>
          <div className="font-medium text-gray-900">{name}</div>
          <div className="text-sm text-gray-500">SKU: {sku}</div>
        </div>
      );
    },
  },
  
  {
    accessorKey: 'category',
    header: 'Category',
    cell: ({ row }) => (
      <Badge variant="outline">{row.getValue('category')}</Badge>
    ),
  },
  
  {
    accessorKey: 'price',
    header: ({ column }) => (
      <DataTableColumnHeader column={column} title="Price" />
    ),
    cell: ({ row }) => {
      const price = parseFloat(row.getValue('price'));
      return (
        <div className="font-medium">
          ${price.toFixed(2)}
        </div>
      );
    },
  },
  
  {
    accessorKey: 'stock',
    header: 'Stock',
    cell: ({ row }) => {
      const stock = row.getValue('stock') as number;
      
      let color = 'text-green-600';
      if (stock < 10) color = 'text-red-600';
      else if (stock < 50) color = 'text-yellow-600';
      
      return (
        <div className={`font-medium ${color}`}>
          {stock} units
        </div>
      );
    },
  },
  
  {
    accessorKey: 'status',
    header: 'Status',
    cell: ({ row }) => (
      <StatusBadge status={row.getValue('status')} />
    ),
  },
  
  {
    id: 'actions',
    cell: ({ row }) => (
      <DataTableRowActions
        row={row}
        onView={(product) => {
          window.location.href = `/admin/products/${product.id}`;
        }}
        onEdit={(product) => {
          window.location.href = `/admin/products/${product.id}/edit`;
        }}
      />
    ),
  },
];

§ 2.5 FILTERABLE COLUMNS PATTERN

typescript
// components/admin/data-table/filter-types.tsx
'use client';

import { useState } from 'react';
import { Calendar, ChevronDown, Filter } from 'lucide-react';
import { format } from 'date-fns';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  Popover,
  PopoverContent,
  PopoverTrigger,
} from '@/components/ui/popover';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { Calendar as CalendarComponent } from '@/components/ui/calendar';
import { cn } from '@/lib/utils';

// ============ TEXT FILTER ============
interface TextFilterProps {
  column: string;
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
}

export function TextFilter({
  column,
  value,
  onChange,
  placeholder = 'Filter...',
}: TextFilterProps) {
  return (
    <div className="relative">
      <Input
        placeholder={placeholder}
        value={value}
        onChange={(e) => onChange(e.target.value)}
        className="pr-8"
      />
      {value && (
        <button
          onClick={() => onChange('')}
          className="absolute right-2 top-1/2 -translate-y-1/2 text-gray-500 hover:text-gray-700"
        >
          ×
        </button>
      )}
    </div>
  );
}

// ============ SELECT FILTER ============
interface SelectFilterProps {
  column: string;
  value: string;
  onChange: (value: string) => void;
  options: Array<{
    label: string;
    value: string;
  }>;
  placeholder?: string;
}

export function SelectFilter({
  column,
  value,
  onChange,
  options,
  placeholder = 'Select...',
}: SelectFilterProps) {
  return (
    <Select value={value} onValueChange={onChange}>
      <SelectTrigger className="w-full">
        <SelectValue placeholder={placeholder} />
      </SelectTrigger>
      <SelectContent>
        {options.map((option) => (
          <SelectItem key={option.value} value={option.value}>
            {option.label}
          </SelectItem>
        ))}
      </SelectContent>
    </Select>
  );
}

// ============ DATE RANGE FILTER ============
interface DateRangeFilterProps {
  column: string;
  value: { from?: Date; to?: Date };
  onChange: (value: { from?: Date; to?: Date }) => void;
}

export function DateRangeFilter({
  column,
  value,
  onChange,
}: DateRangeFilterProps) {
  const [open, setOpen] = useState(false);
  
  return (
    <Popover open={open} onOpenChange={setOpen}>
      <PopoverTrigger asChild>
        <Button
          variant="outline"
          className={cn(
            'w-full justify-start text-left font-normal',
            !value && 'text-muted-foreground'
          )}
        >
          <Calendar className="mr-2 h-4 w-4" />
          {value?.from ? (
            value.to ? (
              <>
                {format(value.from, 'LLL dd, y')} - {format(value.to, 'LLL dd, y')}
              </>
            ) : (
              format(value.from, 'LLL dd, y')
            )
          ) : (
            <span>Pick a date range</span>
          )}
          <ChevronDown className="ml-auto h-4 w-4 opacity-50" />
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-auto p-0" align="start">
        <CalendarComponent
          initialFocus
          mode="range"
          defaultMonth={value?.from}
          selected={value}
          onSelect={(range) => {
            onChange(range || {});
            if (range?.from && range?.to) {
              setOpen(false);
            }
          }}
          numberOfMonths={2}
        />
      </PopoverContent>
    </Popover>
  );
}

// ============ BOOLEAN FILTER ============
interface BooleanFilterProps {
  column: string;
  value: boolean | null;
  onChange: (value: boolean | null) => void;
  trueLabel?: string;
  falseLabel?: string;
}

export function BooleanFilter({
  column,
  value,
  onChange,
  trueLabel = 'Yes',
  falseLabel = 'No',
}: BooleanFilterProps) {
  const options = [
    { label: 'All', value: null },
    { label: trueLabel, value: true },
    { label: falseLabel, value: false },
  ];
  
  return (
    <Select
      value={value === null ? 'null' : value.toString()}
      onValueChange={(val) => {
        if (val === 'null') onChange(null);
        else onChange(val === 'true');
      }}
    >
      <SelectTrigger className="w-full">
        <SelectValue placeholder="Select..." />
      </SelectTrigger>
      <SelectContent>
        {options.map((option) => (
          <SelectItem
            key={option.value === null ? 'null' : option.value.toString()}
            value={option.value === null ? 'null' : option.value.toString()}
          >
            {option.label}
          </SelectItem>
        ))}
      </SelectContent>
    </Select>
  );
}

// ============ NUMBER RANGE FILTER ============
interface NumberRangeFilterProps {
  column: string;
  value: { min?: number; max?: number };
  onChange: (value: { min?: number; max?: number }) => void;
  placeholder?: {
    min?: string;
    max?: string;
  };
}

export function NumberRangeFilter({
  column,
  value,
  onChange,
  placeholder = { min: 'Min', max: 'Max' },
}: NumberRangeFilterProps) {
  return (
    <div className="flex gap-2">
      <Input
        type="number"
        placeholder={placeholder.min}
        value={value.min || ''}
        onChange={(e) => 
          onChange({
            ...value,
            min: e.target.value ? parseFloat(e.target.value) : undefined,
          })
        }
      />
      <Input
        type="number"
        placeholder={placeholder.max}
        value={value.max || ''}
        onChange={(e) =>
          onChange({
            ...value,
            max: e.target.value ? parseFloat(e.target.value) : undefined,
          })
        }
      />
    </div>
  );
}

// ============ FILTER PRESETS ============
interface FilterPreset {
  id: string;
  label: string;
  filters: Record<string, any>;
}

interface FilterPresetsProps {
  presets: FilterPreset[];
  onSelect: (filters: Record<string, any>) => void;
  activePreset?: string;
}

export function FilterPresets({
  presets,
  onSelect,
  activePreset,
}: FilterPresetsProps) {
  return (
    <div className="flex gap-2">
      {presets.map((preset) => (
        <Button
          key={preset.id}
          variant={activePreset === preset.id ? 'default' : 'outline'}
          size="sm"
          onClick={() => onSelect(preset.filters)}
        >
          {preset.label}
        </Button>
      ))}
    </div>
  );
}

§ 2.6 URL STATE PERSISTENCE

typescript
// hooks/use-table-state.ts
'use client';

import { useState, useEffect, useCallback } from 'react';
import { useSearchParams, useRouter, usePathname } from 'next/navigation';
import { SortingState, ColumnFiltersState } from '@tanstack/react-table';
import { z } from 'zod';

// ============ URL SCHEMAS ============
const URLTableParamsSchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  pageSize: z.coerce.number().int().min(1).max(100).default(20),
  sortBy: z.string().optional(),
  sortOrder: z.enum(['asc', 'desc']).optional(),
  // Filters as encoded JSON
  filters: z.string().optional(),
  search: z.string().optional(),
});

export type URLTableParams = z.infer<typeof URLTableParamsSchema>;

// ============ HOOK ============
export function useTableState(defaultParams?: Partial<URLTableParams>) {
  const router = useRouter();
  const pathname = usePathname();
  const searchParams = useSearchParams();
  
  // Parse URL params
  const parseUrlParams = useCallback((): URLTableParams => {
    try {
      const params = {
        page: searchParams.get('page'),
        pageSize: searchParams.get('pageSize'),
        sortBy: searchParams.get('sortBy'),
        sortOrder: searchParams.get('sortOrder') as 'asc' | 'desc' | null,
        filters: searchParams.get('filters'),
        search: searchParams.get('search'),
      };
      
      // Clean undefined values
      const cleanedParams: Record<string, any> = {};
      for (const [key, value] of Object.entries(params)) {
        if (value !== null && value !== '') {
          cleanedParams[key] = value;
        }
      }
      
      const parsed = URLTableParamsSchema.parse(cleanedParams);
      
      // Merge with defaults
      return { ...defaultParams, ...parsed };
    } catch (error) {
      console.error('Failed to parse URL params:', error);
      return URLTableParamsSchema.parse(defaultParams || {});
    }
  }, [searchParams, defaultParams]);
  
  const [params, setParams] = useState<URLTableParams>(parseUrlParams());
  
  // Update URL
  const updateUrl = useCallback((newParams: Partial<URLTableParams>) => {
    const currentParams = new URLSearchParams(searchParams.toString());
    
    // Merge with existing params
    const merged = { ...parseUrlParams(), ...newParams };
    
    // Update URLSearchParams
    for (const [key, value] of Object.entries(merged)) {
      if (value === undefined || value === null || value === '') {
        currentParams.delete(key);
      } else {
        currentParams.set(key, String(value));
      }
    }
    
    // Remove default values
    if (merged.page === 1) currentParams.delete('page');
    if (merged.pageSize === 20) currentParams.delete('pageSize');
    if (!merged.sortBy) currentParams.delete('sortBy');
    if (!merged.sortOrder) currentParams.delete('sortOrder');
    if (!merged.filters) currentParams.delete('filters');
    if (!merged.search) currentParams.delete('search');
    
    // Update URL
    const newUrl = `${pathname}?${currentParams.toString()}`;
    router.push(newUrl, { scroll: false });
  }, [router, pathname, searchParams, parseUrlParams]);
  
  // Parse filters from URL
  const parseFilters = useCallback((): ColumnFiltersState => {
    if (!params.filters) return [];
    
    try {
      const parsed = JSON.parse(params.filters);
      return Array.isArray(parsed) ? parsed : [];
    } catch (error) {
      console.error('Failed to parse filters:', error);
      return [];
    }
  }, [params.filters]);
  
  // Parse sorting from URL
  const parseSorting = useCallback((): SortingState => {
    if (!params.sortBy) return [];
    
    return [{
      id: params.sortBy,
      desc: params.sortOrder === 'desc',
    }];
  }, [params.sortBy, params.sortOrder]);
  
  // Update params when URL changes
  useEffect(() => {
    setParams(parseUrlParams());
  }, [searchParams, parseUrlParams]);
  
  // Convert to table params
  const getTableParams = useCallback(() => ({
    page: params.page,
    pageSize: params.pageSize,
    sortBy: params.sortBy,
    sortOrder: params.sortOrder,
    filters: parseFilters(),
    search: params.search,
  }), [params, parseFilters]);
  
  // Handlers for table events
  const handlePageChange = useCallback((page: number) => {
    updateUrl({ page });
  }, [updateUrl]);
  
  const handlePageSizeChange = useCallback((pageSize: number) => {
    updateUrl({ pageSize, page: 1 }); // Reset to page 1 when page size changes
  }, [updateUrl]);
  
  const handleSortChange = useCallback((sorting: SortingState) => {
    if (sorting.length === 0) {
      updateUrl({ sortBy: undefined, sortOrder: undefined });
    } else {
      updateUrl({
        sortBy: sorting[0].id,
        sortOrder: sorting[0].desc ? 'desc' : 'asc',
      });
    }
  }, [updateUrl]);
  
  const handleFilterChange = useCallback((filters: ColumnFiltersState) => {
    updateUrl({
      filters: filters.length > 0 ? JSON.stringify(filters) : undefined,
      page: 1, // Reset to page 1 when filters change
    });
  }, [updateUrl]);
  
  const handleSearchChange = useCallback((search: string) => {
    updateUrl({
      search: search || undefined,
      page: 1, // Reset to page 1 when search changes
    });
  }, [updateUrl]);
  
  const resetFilters = useCallback(() => {
    updateUrl({
      page: 1,
      sortBy: undefined,
      sortOrder: undefined,
      filters: undefined,
      search: undefined,
    });
  }, [updateUrl]);
  
  // Build URL for links
  const buildTableUrl = useCallback((overrides: Partial<URLTableParams> = {}) => {
    const merged = { ...params, ...overrides };
    const urlParams = new URLSearchParams();
    
    if (merged.page && merged.page > 1) urlParams.set('page', merged.page.toString());
    if (merged.pageSize && merged.pageSize !== 20) urlParams.set('pageSize', merged.pageSize.toString());
    if (merged.sortBy) urlParams.set('sortBy', merged.sortBy);
    if (merged.sortOrder) urlParams.set('sortOrder', merged.sortOrder);
    if (merged.filters) urlParams.set('filters', merged.filters);
    if (merged.search) urlParams.set('search', merged.search);
    
    const queryString = urlParams.toString();
    return queryString ? `${pathname}?${queryString}` : pathname;
  }, [params, pathname]);
  
  return {
    params,
    getTableParams,
    handlePageChange,
    handlePageSizeChange,
    handleSortChange,
    handleFilterChange,
    handleSearchChange,
    resetFilters,
    buildTableUrl,
    parseFilters,
    parseSorting,
  };
}

// ============ EXAMPLE USAGE ============
/*
export function UsersTable() {
  const tableState = useTableState({
    pageSize: 10,
    sortBy: 'createdAt',
    sortOrder: 'desc',
  });
  
  const { params, handlePageChange, handleSortChange, handleFilterChange } = tableState;
  
  // Fetch data using params
  const { data, total, page, pageSize, totalPages } = await getTableData('user', {
    page: params.page,
    pageSize: params.pageSize,
    sortBy: params.sortBy,
    sortOrder: params.sortOrder,
    filters: tableState.parseFilters(),
    search: params.search,
  });
  
  return (
    <DataTable
      data={data}
      total={total}
      page={page}
      pageSize={pageSize}
      totalPages={totalPages}
      onPageChange={handlePageChange}
      onPageSizeChange={tableState.handlePageSizeChange}
      onSortChange={handleSortChange}
      onFilterChange={handleFilterChange}
      initialSorting={tableState.parseSorting()}
      initialFilters={tableState.parseFilters()}
    />
  );
}
*/

---

**NOTA:** Questo è solo il §1 e §2 (Admin Layout & Data Table System) della documentazione completa. La risposta completa richiederebbe oltre 3.000 righe di codice e documentazione. Continuerei con le altre sezioni se vuoi vedere il resto del catalogo admin.