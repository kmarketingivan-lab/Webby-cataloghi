# CATALOGO-ADMIN-BACKOFFICE

ESPANSIONE CATALOGO-ADMIN-BACKOFFICE
§ DASHBOARD ANALYTICS ADMIN
1. Prisma Schema Aggiornamento
prisma
// schema.prisma - Aggiungi queste tabelle

model AnalyticsMetric {
  id        String   @id @default(cuid())
  date      DateTime @unique
  revenue   Decimal  @default(0)
  users     Int      @default(0)
  orders    Int      @default(0)
  conversions Decimal @default(0)
  avgOrderValue Decimal @default(0)
  createdAt DateTime @default(now())
  
  @@index([date])
}

model AdminDashboardPreference {
  id        String   @id @default(cuid())
  userId    String   @unique
  period    String   @default("30d") // today, 7d, 30d, 90d, ytd
  widgets   Json     // Configurazione widget personalizzata
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@map("admin_dashboard_preferences")
}
2. Widget KPI con Recharts
tsx
// app/admin/dashboard/components/KpiDashboard.tsx
"use client";

import { useState, useEffect } from 'react';
import {
  LineChart,
  Line,
  BarChart,
  Bar,
  AreaChart,
  Area,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
  Legend
} from 'recharts';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Skeleton } from '@/components/ui/skeleton';
import { TrendingUp, TrendingDown, Users, ShoppingCart, DollarSign, Target } from 'lucide-react';

type Period = 'today' | '7d' | '30d' | '90d' | 'ytd';

interface KpiData {
  name: string;
  value: number;
  previousValue: number;
  change: number;
  isPositive: boolean;
  sparkline: number[];
}

export default function KpiDashboard() {
  const [period, setPeriod] = useState<Period>('30d');
  const [kpis, setKpis] = useState<KpiData[]>([]);
  const [chartData, setChartData] = useState<any[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchDashboardData();
  }, [period]);

  const fetchDashboardData = async () => {
    setLoading(true);
    try {
      const response = await fetch(`/api/admin/analytics?period=${period}`);
      const data = await response.json();
      
      setKpis([
        {
          name: 'Revenue',
          value: data.revenue.current,
          previousValue: data.revenue.previous,
          change: calculateChange(data.revenue.current, data.revenue.previous),
          isPositive: data.revenue.current >= data.revenue.previous,
          sparkline: data.revenueTrend
        },
        {
          name: 'Users',
          value: data.users.current,
          previousValue: data.users.previous,
          change: calculateChange(data.users.current, data.users.previous),
          isPositive: data.users.current >= data.users.previous,
          sparkline: data.usersTrend
        },
        {
          name: 'Orders',
          value: data.orders.current,
          previousValue: data.orders.previous,
          change: calculateChange(data.orders.current, data.orders.previous),
          isPositive: data.orders.current >= data.orders.previous,
          sparkline: data.ordersTrend
        },
        {
          name: 'Conversions',
          value: data.conversions.current,
          previousValue: data.conversions.previous,
          change: calculateChange(data.conversions.current, data.conversions.previous),
          isPositive: data.conversions.current >= data.conversions.previous,
          sparkline: data.conversionsTrend
        }
      ]);
      
      setChartData(data.chartData);
    } catch (error) {
      console.error('Error fetching analytics:', error);
    } finally {
      setLoading(false);
    }
  };

  const calculateChange = (current: number, previous: number): number => {
    if (previous === 0) return 100;
    return ((current - previous) / previous) * 100;
  };

  const KpiCard = ({ kpi }: { kpi: KpiData }) => (
    <Card>
      <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
        <CardTitle className="text-sm font-medium">{kpi.name}</CardTitle>
        {kpi.name === 'Revenue' && <DollarSign className="h-4 w-4 text-muted-foreground" />}
        {kpi.name === 'Users' && <Users className="h-4 w-4 text-muted-foreground" />}
        {kpi.name === 'Orders' && <ShoppingCart className="h-4 w-4 text-muted-foreground" />}
        {kpi.name === 'Conversions' && <Target className="h-4 w-4 text-muted-foreground" />}
      </CardHeader>
      <CardContent>
        <div className="text-2xl font-bold">
          {kpi.name === 'Revenue' ? `$${kpi.value.toLocaleString()}` : kpi.value.toLocaleString()}
        </div>
        <div className="flex items-center text-xs">
          {kpi.isPositive ? (
            <TrendingUp className="mr-1 h-4 w-4 text-green-500" />
          ) : (
            <TrendingDown className="mr-1 h-4 w-4 text-red-500" />
          )}
          <span className={kpi.isPositive ? 'text-green-500' : 'text-red-500'}>
            {kpi.change.toFixed(1)}%
          </span>
          <span className="text-muted-foreground ml-1">vs. previous period</span>
        </div>
        <div className="mt-4 h-[60px]">
          <ResponsiveContainer width="100%" height="100%">
            <LineChart data={kpi.sparkline.map((v, i) => ({ value: v }))}>
              <Line 
                type="monotone" 
                dataKey="value" 
                stroke={kpi.isPositive ? "#10b981" : "#ef4444"} 
                strokeWidth={2}
                dot={false}
              />
            </LineChart>
          </ResponsiveContainer>
        </div>
      </CardContent>
    </Card>
  );

  if (loading) {
    return (
      <div className="space-y-4">
        <div className="flex justify-between items-center">
          <Skeleton className="h-10 w-32" />
          <Skeleton className="h-10 w-40" />
        </div>
        <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
          {[...Array(4)].map((_, i) => (
            <Skeleton key={i} className="h-48 w-full" />
          ))}
        </div>
      </div>
    );
  }

  return (
    <div className="space-y-4">
      <div className="flex justify-between items-center">
        <h2 className="text-3xl font-bold tracking-tight">Dashboard Analytics</h2>
        <Select value={period} onValueChange={(value: Period) => setPeriod(value)}>
          <SelectTrigger className="w-[180px]">
            <SelectValue placeholder="Select period" />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="today">Today</SelectItem>
            <SelectItem value="7d">Last 7 days</SelectItem>
            <SelectItem value="30d">Last 30 days</SelectItem>
            <SelectItem value="90d">Last 90 days</SelectItem>
            <SelectItem value="ytd">Year to Date</SelectItem>
          </SelectContent>
        </Select>
      </div>
      
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
        {kpis.map((kpi) => (
          <KpiCard key={kpi.name} kpi={kpi} />
        ))}
      </div>
      
      <Card className="col-span-4">
        <CardHeader>
          <CardTitle>Performance Overview</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="h-[400px]">
            <ResponsiveContainer width="100%" height="100%">
              <AreaChart data={chartData}>
                <CartesianGrid strokeDasharray="3 3" />
                <XAxis dataKey="date" />
                <YAxis />
                <Tooltip />
                <Legend />
                <Area type="monotone" dataKey="revenue" stackId="1" stroke="#8884d8" fill="#8884d8" />
                <Area type="monotone" dataKey="orders" stackId="1" stroke="#82ca9d" fill="#82ca9d" />
                <Area type="monotone" dataKey="users" stackId="1" stroke="#ffc658" fill="#ffc658" />
              </AreaChart>
            </ResponsiveContainer>
          </div>
        </CardContent>
      </Card>
    </div>
  );
}
§ AUDIT LOG SYSTEM
1. Schema Prisma per Audit Logs
prisma
// schema.prisma
model AuditLog {
  id          String   @id @default(cuid())
  action      String   // CREATE, UPDATE, DELETE, LOGIN, etc.
  entityType  String   // User, Order, Product, etc.
  entityId    String?
  userId      String?
  userEmail   String?
  userRole    String?
  
  oldValue    Json?
  newValue    Json?
  changes     Json?    // Differenze specifiche
  
  ipAddress   String?
  userAgent   String?
  
  timestamp   DateTime @default(now())
  
  // Metadata
  description String?
  tags        String[]
  
  @@index([action])
  @@index([entityType, entityId])
  @@index([userId])
  @@index([timestamp])
  @@index([userRole])
}

// Aggiungi questa tabella per retention policy
model AuditLogRetention {
  id            String   @id @default(cuid())
  policyType    String   // TEMPORAL, SIZE_BASED, COMPRESSION
  daysToKeep    Int      @default(90)
  maxSizeMB     Int      @default(1024)
  compressAfter Int      @default(30) // giorni
  isActive      Boolean  @default(true)
  lastCleanup   DateTime?
  createdAt     DateTime @default(now())
}
2. AuditLogTable Component
tsx
// app/admin/audit/components/AuditLogTable.tsx
"use client";

import { useState, useEffect } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Badge } from '@/components/ui/badge';
import { Checkbox } from '@/components/ui/checkbox';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@/components/ui/dialog';
import { Search, Filter, Download, Eye, MoreVertical, Calendar } from 'lucide-react';
import { DatePicker } from '@/components/ui/date-picker';
import { format } from 'date-fns';

type AuditLog = {
  id: string;
  action: string;
  entityType: string;
  entityId: string;
  userEmail: string;
  userRole: string;
  description: string;
  timestamp: Date;
  ipAddress?: string;
  tags: string[];
  changes?: Record<string, { old: any; new: any }>;
};

type FilterState = {
  search: string;
  action: string;
  entityType: string;
  userRole: string;
  dateFrom: Date | null;
  dateTo: Date | null;
};

export default function AuditLogTable() {
  const [logs, setLogs] = useState<AuditLog[]>([]);
  const [filteredLogs, setFilteredLogs] = useState<AuditLog[]>([]);
  const [selectedLogs, setSelectedLogs] = useState<string[]>([]);
  const [loading, setLoading] = useState(true);
  const [filters, setFilters] = useState<FilterState>({
    search: '',
    action: 'all',
    entityType: 'all',
    userRole: 'all',
    dateFrom: null,
    dateTo: null,
  });
  const [page, setPage] = useState(1);
  const [totalPages, setTotalPages] = useState(1);

  useEffect(() => {
    fetchLogs();
  }, [page, filters]);

  useEffect(() => {
    applyFilters();
  }, [logs, filters]);

  const fetchLogs = async () => {
    setLoading(true);
    try {
      const queryParams = new URLSearchParams({
        page: page.toString(),
        ...(filters.action !== 'all' && { action: filters.action }),
        ...(filters.entityType !== 'all' && { entityType: filters.entityType }),
        ...(filters.userRole !== 'all' && { userRole: filters.userRole }),
        ...(filters.dateFrom && { dateFrom: filters.dateFrom.toISOString() }),
        ...(filters.dateTo && { dateTo: filters.dateTo.toISOString() }),
        ...(filters.search && { search: filters.search }),
      });

      const response = await fetch(`/api/admin/audit-logs?${queryParams}`);
      const data = await response.json();
      setLogs(data.logs);
      setTotalPages(data.totalPages);
    } catch (error) {
      console.error('Error fetching audit logs:', error);
    } finally {
      setLoading(false);
    }
  };

  const applyFilters = () => {
    let filtered = [...logs];

    if (filters.search) {
      const searchLower = filters.search.toLowerCase();
      filtered = filtered.filter(log =>
        log.userEmail?.toLowerCase().includes(searchLower) ||
        log.entityId?.toLowerCase().includes(searchLower) ||
        log.description?.toLowerCase().includes(searchLower) ||
        log.tags.some(tag => tag.toLowerCase().includes(searchLower))
      );
    }

    if (filters.action !== 'all') {
      filtered = filtered.filter(log => log.action === filters.action);
    }

    if (filters.entityType !== 'all') {
      filtered = filtered.filter(log => log.entityType === filters.entityType);
    }

    if (filters.userRole !== 'all') {
      filtered = filtered.filter(log => log.userRole === filters.userRole);
    }

    if (filters.dateFrom) {
      filtered = filtered.filter(log => new Date(log.timestamp) >= filters.dateFrom!);
    }

    if (filters.dateTo) {
      const endDate = new Date(filters.dateTo);
      endDate.setHours(23, 59, 59, 999);
      filtered = filtered.filter(log => new Date(log.timestamp) <= endDate);
    }

    setFilteredLogs(filtered);
  };

  const exportLogs = async (format: 'csv' | 'json') => {
    const response = await fetch(`/api/admin/audit-logs/export?format=${format}`);
    const blob = await response.blob();
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `audit-logs-${new Date().toISOString()}.${format}`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
  };

  const getActionColor = (action: string) => {
    const colors: Record<string, string> = {
      CREATE: 'bg-green-100 text-green-800',
      UPDATE: 'bg-blue-100 text-blue-800',
      DELETE: 'bg-red-100 text-red-800',
      LOGIN: 'bg-purple-100 text-purple-800',
      LOGOUT: 'bg-gray-100 text-gray-800',
    };
    return colors[action] || 'bg-gray-100 text-gray-800';
  };

  const toggleSelectAll = (checked: boolean) => {
    if (checked) {
      setSelectedLogs(filteredLogs.map(log => log.id));
    } else {
      setSelectedLogs([]);
    }
  };

  const toggleSelectLog = (logId: string) => {
    setSelectedLogs(prev =>
      prev.includes(logId)
        ? prev.filter(id => id !== logId)
        : [...prev, logId]
    );
  };

  const clearOldLogs = async () => {
    if (confirm('Are you sure you want to clear logs older than 90 days?')) {
      await fetch('/api/admin/audit-logs/cleanup', { method: 'POST' });
      fetchLogs();
    }
  };

  return (
    <div className="space-y-4">
      <div className="flex justify-between items-center">
        <CardTitle>Audit Logs</CardTitle>
        <div className="flex gap-2">
          <Button variant="outline" onClick={() => exportLogs('csv')}>
            <Download className="mr-2 h-4 w-4" />
            Export CSV
          </Button>
          <Button variant="outline" onClick={() => exportLogs('json')}>
            <Download className="mr-2 h-4 w-4" />
            Export JSON
          </Button>
          <Button variant="destructive" onClick={clearOldLogs}>
            Cleanup Old Logs
          </Button>
        </div>
      </div>

      {/* Filtri */}
      <Card>
        <CardContent className="pt-6">
          <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
            <div className="relative">
              <Search className="absolute left-3 top-3 h-4 w-4 text-muted-foreground" />
              <Input
                placeholder="Search logs..."
                className="pl-9"
                value={filters.search}
                onChange={(e) => setFilters({ ...filters, search: e.target.value })}
              />
            </div>
            
            <Select value={filters.action} onValueChange={(value) => setFilters({ ...filters, action: value })}>
              <SelectTrigger>
                <SelectValue placeholder="Action" />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="all">All Actions</SelectItem>
                <SelectItem value="CREATE">Create</SelectItem>
                <SelectItem value="UPDATE">Update</SelectItem>
                <SelectItem value="DELETE">Delete</SelectItem>
                <SelectItem value="LOGIN">Login</SelectItem>
                <SelectItem value="LOGOUT">Logout</SelectItem>
              </SelectContent>
            </Select>

            <Select value={filters.entityType} onValueChange={(value) => setFilters({ ...filters, entityType: value })}>
              <SelectTrigger>
                <SelectValue placeholder="Entity Type" />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="all">All Entities</SelectItem>
                <SelectItem value="User">Users</SelectItem>
                <SelectItem value="Order">Orders</SelectItem>
                <SelectItem value="Product">Products</SelectItem>
                <SelectItem value="Category">Categories</SelectItem>
              </SelectContent>
            </Select>

            <div className="flex gap-2">
              <DatePicker
                date={filters.dateFrom}
                onSelect={(date) => setFilters({ ...filters, dateFrom: date })}
                placeholder="From date"
              />
              <DatePicker
                date={filters.dateTo}
                onSelect={(date) => setFilters({ ...filters, dateTo: date })}
                placeholder="To date"
              />
            </div>
          </div>
        </CardContent>
      </Card>

      {/* Tabella */}
      <Card>
        <CardContent className="p-0">
          <Table>
            <TableHeader>
              <TableRow>
                <TableHead className="w-12">
                  <Checkbox
                    checked={selectedLogs.length === filteredLogs.length && filteredLogs.length > 0}
                    onCheckedChange={toggleSelectAll}
                  />
                </TableHead>
                <TableHead>Action</TableHead>
                <TableHead>User</TableHead>
                <TableHead>Entity</TableHead>
                <TableHead>Description</TableHead>
                <TableHead>Date</TableHead>
                <TableHead>IP</TableHead>
                <TableHead className="w-20"></TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              {loading ? (
                <TableRow>
                  <TableCell colSpan={8} className="text-center py-8">
                    Loading...
                  </TableCell>
                </TableRow>
              ) : filteredLogs.length === 0 ? (
                <TableRow>
                  <TableCell colSpan={8} className="text-center py-8">
                    No audit logs found
                  </TableCell>
                </TableRow>
              ) : (
                filteredLogs.map((log) => (
                  <TableRow key={log.id}>
                    <TableCell>
                      <Checkbox
                        checked={selectedLogs.includes(log.id)}
                        onCheckedChange={() => toggleSelectLog(log.id)}
                      />
                    </TableCell>
                    <TableCell>
                      <Badge className={getActionColor(log.action)}>
                        {log.action}
                      </Badge>
                    </TableCell>
                    <TableCell>
                      <div>
                        <div className="font-medium">{log.userEmail}</div>
                        <div className="text-sm text-muted-foreground">{log.userRole}</div>
                      </div>
                    </TableCell>
                    <TableCell>
                      <div>
                        <div>{log.entityType}</div>
                        <div className="text-sm text-muted-foreground">{log.entityId}</div>
                      </div>
                    </TableCell>
                    <TableCell className="max-w-xs truncate">
                      {log.description}
                    </TableCell>
                    <TableCell>
                      {format(new Date(log.timestamp), 'MMM dd, yyyy HH:mm')}
                    </TableCell>
                    <TableCell>
                      {log.ipAddress || '-'}
                    </TableCell>
                    <TableCell>
                      <Dialog>
                        <DropdownMenu>
                          <DropdownMenuTrigger asChild>
                            <Button variant="ghost" size="icon">
                              <MoreVertical className="h-4 w-4" />
                            </Button>
                          </DropdownMenuTrigger>
                          <DropdownMenuContent align="end">
                            <DialogTrigger asChild>
                              <DropdownMenuItem>
                                <Eye className="mr-2 h-4 w-4" />
                                View Details
                              </DropdownMenuItem>
                            </DialogTrigger>
                          </DropdownMenuContent>
                        </DropdownMenu>
                        
                        <DialogContent className="max-w-4xl">
                          <DialogHeader>
                            <DialogTitle>Audit Log Details</DialogTitle>
                          </DialogHeader>
                          <div className="space-y-4">
                            <div className="grid grid-cols-2 gap-4">
                              <div>
                                <h4 className="font-semibold mb-2">User Information</h4>
                                <div className="space-y-1">
                                  <div>Email: {log.userEmail}</div>
                                  <div>Role: {log.userRole}</div>
                                  <div>IP: {log.ipAddress || 'N/A'}</div>
                                </div>
                              </div>
                              <div>
                                <h4 className="font-semibold mb-2">Action Details</h4>
                                <div className="space-y-1">
                                  <div>Action: {log.action}</div>
                                  <div>Entity: {log.entityType} ({log.entityId})</div>
                                  <div>Time: {format(new Date(log.timestamp), 'PPpp')}</div>
                                </div>
                              </div>
                            </div>
                            
                            {log.changes && (
                              <div>
                                <h4 className="font-semibold mb-2">Changes</h4>
                                <div className="bg-muted rounded-lg p-4">
                                  <pre className="text-sm overflow-auto">
                                    {JSON.stringify(log.changes, null, 2)}
                                  </pre>
                                </div>
                              </div>
                            )}
                            
                            {log.tags.length > 0 && (
                              <div>
                                <h4 className="font-semibold mb-2">Tags</h4>
                                <div className="flex flex-wrap gap-2">
                                  {log.tags.map((tag) => (
                                    <Badge key={tag} variant="secondary">
                                      {tag}
                                    </Badge>
                                  ))}
                                </div>
                              </div>
                            )}
                          </div>
                        </DialogContent>
                      </Dialog>
                    </TableCell>
                  </TableRow>
                ))
              )}
            </TableBody>
          </Table>
        </CardContent>
      </Card>
    </div>
  );
}
§ BULK ACTIONS
1. Pattern per Selezione Multipla
tsx
// components/bulk/BulkActionsProvider.tsx
"use client";

import React, { createContext, useContext, useState, ReactNode } from 'react';
import { toast } from 'sonner';

interface BulkAction<T = any> {
  id: string;
  label: string;
  handler: (selectedIds: string[], data?: T[]) => Promise<void>;
  requiresConfirmation?: boolean;
  confirmationMessage?: string;
  variant?: 'default' | 'destructive' | 'outline';
}

interface BulkActionsContextType {
  selectedIds: string[];
  selectedData: any[];
  isSelecting: boolean;
  totalItems: number;
  setSelectedIds: (ids: string[]) => void;
  setSelectedData: (data: any[]) => void;
  toggleSelectAll: (allIds: string[], allData?: any[]) => void;
  toggleSelectItem: (id: string, data?: any) => void;
  clearSelection: () => void;
  executeBulkAction: (action: BulkAction) => Promise<void>;
  registerActions: (actions: BulkAction[]) => void;
  availableActions: BulkAction[];
}

const BulkActionsContext = createContext<BulkActionsContextType | undefined>(undefined);

export function BulkActionsProvider({ children }: { children: ReactNode }) {
  const [selectedIds, setSelectedIds] = useState<string[]>([]);
  const [selectedData, setSelectedData] = useState<any[]>([]);
  const [availableActions, setAvailableActions] = useState<BulkAction[]>([]);

  const toggleSelectAll = (allIds: string[], allData?: any[]) => {
    if (selectedIds.length === allIds.length) {
      clearSelection();
    } else {
      setSelectedIds(allIds);
      if (allData) setSelectedData(allData);
    }
  };

  const toggleSelectItem = (id: string, data?: any) => {
    setSelectedIds(prev =>
      prev.includes(id)
        ? prev.filter(itemId => itemId !== id)
        : [...prev, id]
    );
    
    if (data) {
      setSelectedData(prev =>
        prev.some(item => item.id === id)
          ? prev.filter(item => item.id !== id)
          : [...prev, data]
      );
    }
  };

  const clearSelection = () => {
    setSelectedIds([]);
    setSelectedData([]);
  };

  const executeBulkAction = async (action: BulkAction) => {
    if (selectedIds.length === 0) {
      toast.error('No items selected');
      return;
    }

    try {
      await action.handler(selectedIds, selectedData);
      toast.success(`Action "${action.label}" completed successfully`);
      clearSelection();
    } catch (error) {
      toast.error(`Failed to execute "${action.label}"`);
      console.error('Bulk action error:', error);
    }
  };

  const registerActions = (actions: BulkAction[]) => {
    setAvailableActions(actions);
  };

  return (
    <BulkActionsContext.Provider
      value={{
        selectedIds,
        selectedData,
        isSelecting: selectedIds.length > 0,
        totalItems: selectedIds.length,
        setSelectedIds,
        setSelectedData,
        toggleSelectAll,
        toggleSelectItem,
        clearSelection,
        executeBulkAction,
        registerActions,
        availableActions,
      }}
    >
      {children}
    </BulkActionsContext.Provider>
  );
}

export const useBulkActions = () => {
  const context = useContext(BulkActionsContext);
  if (!context) {
    throw new Error('useBulkActions must be used within BulkActionsProvider');
  }
  return context;
};
2. Bulk Actions Toolbar
tsx
// components/bulk/BulkActionsToolbar.tsx
"use client";

import { useState } from 'react';
import { Button } from '@/components/ui/button';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogFooter,
} from '@/components/ui/dialog';
import { Progress } from '@/components/ui/progress';
import { Check, MoreHorizontal, Trash2, Download, RefreshCw, Ban, CheckCircle } from 'lucide-react';
import { useBulkActions } from './BulkActionsProvider';
import { Badge } from '@/components/ui/badge';

export function BulkActionsToolbar() {
  const {
    isSelecting,
    totalItems,
    clearSelection,
    executeBulkAction,
    availableActions,
  } = useBulkActions();

  const [showConfirmDialog, setShowConfirmDialog] = useState(false);
  const [pendingAction, setPendingAction] = useState<any>(null);
  const [progress, setProgress] = useState(0);
  const [isProcessing, setIsProcessing] = useState(false);

  if (!isSelecting) return null;

  const handleActionClick = (action: any) => {
    if (action.requiresConfirmation) {
      setPendingAction(action);
      setShowConfirmDialog(true);
    } else {
      executeAction(action);
    }
  };

  const executeAction = async (action: any) => {
    setIsProcessing(true);
    setProgress(0);
    
    // Simulate progress for long operations
    const progressInterval = setInterval(() => {
      setProgress(prev => Math.min(prev + 10, 90));
    }, 500);

    try {
      await executeBulkAction(action);
      clearInterval(progressInterval);
      setProgress(100);
      
      setTimeout(() => {
        setIsProcessing(false);
        setShowConfirmDialog(false);
        setProgress(0);
      }, 500);
    } catch (error) {
      clearInterval(progressInterval);
      setIsProcessing(false);
    }
  };

  const getActionIcon = (label: string) => {
    switch (label.toLowerCase()) {
      case 'delete':
        return <Trash2 className="mr-2 h-4 w-4" />;
      case 'export':
        return <Download className="mr-2 h-4 w-4" />;
      case 'activate':
        return <CheckCircle className="mr-2 h-4 w-4" />;
      case 'deactivate':
        return <Ban className="mr-2 h-4 w-4" />;
      case 'refresh':
        return <RefreshCw className="mr-2 h-4 w-4" />;
      default:
        return <MoreHorizontal className="mr-2 h-4 w-4" />;
    }
  };

  return (
    <>
      <div className="fixed bottom-4 left-1/2 transform -translate-x-1/2 z-50">
        <div className="bg-card border rounded-lg shadow-lg p-4 flex items-center gap-4 min-w-[400px]">
          <div className="flex items-center gap-2">
            <Badge variant="secondary" className="h-8 w-8 rounded-full">
              {totalItems}
            </Badge>
            <span className="text-sm font-medium">
              {totalItems} item{totalItems !== 1 ? 's' : ''} selected
            </span>
          </div>
          
          <div className="flex-1">
            <div className="flex items-center gap-2">
              {availableActions.slice(0, 3).map((action) => (
                <Button
                  key={action.id}
                  variant={action.variant === 'destructive' ? 'destructive' : 'secondary'}
                  size="sm"
                  onClick={() => handleActionClick(action)}
                  disabled={isProcessing}
                >
                  {getActionIcon(action.label)}
                  {action.label}
                </Button>
              ))}
              
              {availableActions.length > 3 && (
                <DropdownMenu>
                  <DropdownMenuTrigger asChild>
                    <Button variant="outline" size="sm" disabled={isProcessing}>
                      <MoreHorizontal className="h-4 w-4" />
                    </Button>
                  </DropdownMenuTrigger>
                  <DropdownMenuContent>
                    {availableActions.slice(3).map((action) => (
                      <DropdownMenuItem
                        key={action.id}
                        onClick={() => handleActionClick(action)}
                        className={action.variant === 'destructive' ? 'text-destructive' : ''}
                      >
                        {getActionIcon(action.label)}
                        {action.label}
                      </DropdownMenuItem>
                    ))}
                  </DropdownMenuContent>
                </DropdownMenu>
              )}
            </div>
          </div>
          
          <Button
            variant="ghost"
            size="sm"
            onClick={clearSelection}
            disabled={isProcessing}
          >
            <Check className="mr-2 h-4 w-4" />
            Done
          </Button>
        </div>
      </div>

      <Dialog open={showConfirmDialog} onOpenChange={setShowConfirmDialog}>
        <DialogContent>
          <DialogHeader>
            <DialogTitle>Confirm Bulk Action</DialogTitle>
          </DialogHeader>
          
          {isProcessing ? (
            <div className="space-y-4">
              <div className="text-center">
                <p className="text-sm text-muted-foreground mb-2">
                  Processing {totalItems} items...
                </p>
                <Progress value={progress} className="w-full" />
                <p className="text-xs text-muted-foreground mt-2">
                  {Math.round(progress)}% complete
                </p>
              </div>
            </div>
          ) : (
            <>
              <div className="space-y-4">
                <p>
                  {pendingAction?.confirmationMessage || 
                    `Are you sure you want to ${pendingAction?.label?.toLowerCase()} ${totalItems} items?`}
                </p>
                
                <div className="bg-muted rounded-lg p-4">
                  <h4 className="font-medium mb-2">Preview of selected items:</h4>
                  <ul className="text-sm space-y-1 max-h-32 overflow-y-auto">
                    {/* Mostra i primi 5 elementi selezionati */}
                    {Array.from({ length: Math.min(totalItems, 5) }).map((_, i) => (
                      <li key={i} className="truncate">
                        Item {i + 1}
                      </li>
                    ))}
                    {totalItems > 5 && (
                      <li className="text-muted-foreground">
                        ...and {totalItems - 5} more
                      </li>
                    )}
                  </ul>
                </div>
                
                {pendingAction?.variant === 'destructive' && (
                  <div className="bg-destructive/10 border border-destructive/20 rounded-lg p-3">
                    <p className="text-sm text-destructive">
                      ⚠️ This action cannot be undone. Please proceed with caution.
                    </p>
                  </div>
                )}
              </div>
              
              <DialogFooter>
                <Button
                  variant="outline"
                  onClick={() => setShowConfirmDialog(false)}
                  disabled={isProcessing}
                >
                  Cancel
                </Button>
                <Button
                  variant={pendingAction?.variant === 'destructive' ? 'destructive' : 'default'}
                  onClick={() => pendingAction && executeAction(pendingAction)}
                  disabled={isProcessing}
                >
                  Confirm {pendingAction?.label}
                </Button>
              </DialogFooter>
            </>
          )}
        </DialogContent>
      </Dialog>
    </>
  );

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-01-ADMIN-BACKOFFICE
Prompt ID: 1 / 48
Parte: 2
Exported: 2026-02-06T11:24:55.968Z
Characters: 937
════════════════════════════════════════════════════════════

s: true,
          category: { select: { name: true } },
        },
        take: 5,
      }),

      // Ricerca categorie
      prisma.category.findMany({
        where: {
          name: { contains: query, mode: 'insensitive' },
        },
        select: {
          id: true,
          name: true,
          slug: true,
          productCount: true,
        },
        take: 5,
      }),
    ]);

    // Mappa i risultati in un formato standardizzato
    const results = [
      ...users.map(user => ({
        id: user.id,
        type: 'user' as const,
        title: user.name || user.email,
        description: user.email,
        url: `/admin/users/${user.id}`,
        icon: 'user',
        metadata: {
          role: user.role,
          createdAt: user.createdAt,
        },
        score: 1.0,
      })),

      ...orders.map(order => ({
        id: order.id,
        type: 'order' as const,
        title: `Order #${order