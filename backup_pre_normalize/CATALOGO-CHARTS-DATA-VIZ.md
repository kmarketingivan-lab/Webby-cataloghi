# CATALOGO-CHARTS-DATA-VIZ

CATALOGO-CHARTS-DATA-VIZ per Next.js 14 + Recharts/Tremor
§ RECHARTS COMPLETO
Tutti i chart types con esempi
tsx
// components/recharts/AllCharts.tsx
'use client';

import React from 'react';
import {
  AreaChart, Area,
  BarChart, Bar,
  LineChart, Line,
  ScatterChart, Scatter, ZAxis,
  PieChart, Pie, Cell,
  RadarChart, Radar, PolarGrid, PolarAngleAxis, PolarRadiusAxis,
  Treemap,
  Sankey,
  FunnelChart, Funnel,
  ComposedChart,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  Legend,
  ResponsiveContainer
} from 'recharts';

const areaData = [
  { name: 'Jan', uv: 4000, pv: 2400, amt: 2400 },
  { name: 'Feb', uv: 3000, pv: 1398, amt: 2210 },
  { name: 'Mar', uv: 2000, pv: 9800, amt: 2290 },
];

const barData = [
  { name: 'A', value: 400 },
  { name: 'B', value: 300 },
  { name: 'C', value: 300 },
];

const pieData = [
  { name: 'Group A', value: 400 },
  { name: 'Group B', value: 300 },
];

const COLORS = ['#0088FE', '#00C49F', '#FFBB28', '#FF8042'];

export function RechartsAreaChart() {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <AreaChart data={areaData}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="name" />
        <YAxis />
        <Tooltip />
        <Area type="monotone" dataKey="uv" stroke="#8884d8" fill="#8884d8" />
      </AreaChart>
    </ResponsiveContainer>
  );
}

export function RechartsBarChartStacked() {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <BarChart data={areaData}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="name" />
        <YAxis />
        <Tooltip />
        <Legend />
        <Bar dataKey="pv" stackId="a" fill="#8884d8" />
        <Bar dataKey="uv" stackId="a" fill="#82ca9d" />
      </BarChart>
    </ResponsiveContainer>
  );
}

export function RechartsPieChartWithCustomizedLabel() {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <PieChart>
        <Pie
          data={pieData}
          cx="50%"
          cy="50%"
          labelLine={false}
          label={({ name, percent }) => `${name}: ${(percent * 100).toFixed(0)}%`}
          outerRadius={80}
          fill="#8884d8"
          dataKey="value"
        >
          {pieData.map((entry, index) => (
            <Cell key={`cell-${index}`} fill={COLORS[index % COLORS.length]} />
          ))}
        </Pie>
        <Tooltip />
      </PieChart>
    </ResponsiveContainer>
  );
}

export function RechartsRadarChartExample() {
  const data = [
    { subject: 'Math', A: 120, B: 110, fullMark: 150 },
    { subject: 'Chinese', A: 98, B: 130, fullMark: 150 },
    { subject: 'English', A: 86, B: 130, fullMark: 150 },
  ];

  return (
    <ResponsiveContainer width="100%" height={300}>
      <RadarChart cx="50%" cy="50%" outerRadius="80%" data={data}>
        <PolarGrid />
        <PolarAngleAxis dataKey="subject" />
        <PolarRadiusAxis />
        <Radar name="Mike" dataKey="A" stroke="#8884d8" fill="#8884d8" fillOpacity={0.6} />
        <Tooltip />
      </RadarChart>
    </ResponsiveContainer>
  );
}

export function RechartsTreemapExample() {
  const data = [
    { name: 'Axis', size: 100 },
    { name: 'Grid', size: 80 },
    { name: 'Line', size: 200 },
  ];

  return (
    <ResponsiveContainer width="100%" height={300}>
      <Treemap
        data={data}
        dataKey="size"
        aspectRatio={4/3}
        stroke="#fff"
        fill="#8884d8"
      />
    </ResponsiveContainer>
  );
}

export function RechartsComposedChartExample() {
  const data = [
    { name: 'Page A', uv: 590, pv: 800, amt: 1400 },
    { name: 'Page B', uv: 868, pv: 967, amt: 1506 },
  ];

  return (
    <ResponsiveContainer width="100%" height={300}>
      <ComposedChart data={data}>
        <CartesianGrid stroke="#f5f5f5" />
        <XAxis dataKey="name" />
        <YAxis />
        <Tooltip />
        <Legend />
        <Bar dataKey="pv" barSize={20} fill="#413ea0" />
        <Line type="monotone" dataKey="uv" stroke="#ff7300" />
      </ComposedChart>
    </ResponsiveContainer>
  );
}
Custom Tooltips
tsx
// components/recharts/CustomTooltips.tsx
'use client';

import React from 'react';
import { TooltipProps } from 'recharts';
import { ValueType, NameType } from 'recharts/types/component/DefaultTooltipContent';

export function CustomTooltip({
  active,
  payload,
  label,
  currency = '€',
}: TooltipProps<ValueType, NameType> & { currency?: string }) {
  if (active && payload && payload.length) {
    return (
      <div className="bg-white p-4 border border-gray-200 rounded-lg shadow-lg">
        <p className="font-semibold text-gray-800">{label}</p>
        {payload.map((entry, index) => (
          <p key={index} className="text-sm" style={{ color: entry.color }}>
            {entry.name}: {currency} {Number(entry.value).toLocaleString()}
          </p>
        ))}
      </div>
    );
  }
  return null;
}

export function TimeSeriesTooltip({
  active,
  payload,
  label,
  dateFormat = 'MMM dd, yyyy HH:mm',
}: TooltipProps<ValueType, NameType> & { dateFormat?: string }) {
  if (active && payload && payload.length) {
    return (
      <div className="bg-gray-900 text-white p-3 rounded-md text-sm">
        <p className="font-medium mb-1">
          {new Date(label).toLocaleDateString('en-US', {
            month: 'short',
            day: 'numeric',
            year: 'numeric',
            hour: '2-digit',
            minute: '2-digit',
          })}
        </p>
        <div className="space-y-1">
          {payload.map((entry, index) => (
            <div key={index} className="flex items-center justify-between">
              <div className="flex items-center">
                <div
                  className="w-3 h-3 rounded-full mr-2"
                  style={{ backgroundColor: entry.color }}
                />
                <span>{entry.name}</span>
              </div>
              <span className="font-semibold ml-4">
                {typeof entry.value === 'number' ? entry.value.toLocaleString() : entry.value}
              </span>
            </div>
          ))}
        </div>
      </div>
    );
  }
  return null;
}
Custom Legends
tsx
// components/recharts/CustomLegends.tsx
'use client';

import React from 'react';

interface LegendProps {
  payload?: Array<{
    value: string;
    color: string;
    dataKey?: string;
  }>;
  onClick?: (dataKey: string) => void;
  activeKeys?: Set<string>;
}

export function InteractiveLegend({ payload, onClick, activeKeys }: LegendProps) {
  if (!payload) return null;

  return (
    <div className="flex flex-wrap gap-3 justify-center mt-4">
      {payload.map((entry, index) => {
        const isActive = !activeKeys || activeKeys.has(entry.dataKey || entry.value);
        
        return (
          <button
            key={`item-${index}`}
            className={`flex items-center px-3 py-1.5 rounded-full text-sm transition-all ${
              isActive 
                ? 'bg-gray-100 hover:bg-gray-200' 
                : 'opacity-40 bg-gray-50 hover:opacity-70'
            }`}
            onClick={() => onClick?.(entry.dataKey || entry.value)}
          >
            <div
              className="w-3 h-3 rounded-full mr-2"
              style={{ backgroundColor: isActive ? entry.color : '#ccc' }}
            />
            <span className={isActive ? 'text-gray-800' : 'text-gray-500'}>
              {entry.value}
            </span>
          </button>
        );
      })}
    </div>
  );
}

export function CompactLegend({ payload }: LegendProps) {
  if (!payload) return null;

  return (
    <div className="flex items-center justify-center space-x-4 text-sm">
      {payload.map((entry, index) => (
        <div key={`item-${index}`} className="flex items-center">
          <div
            className="w-2 h-2 rounded-full mr-1.5"
            style={{ backgroundColor: entry.color }}
          />
          <span className="text-gray-600">{entry.value}</span>
        </div>
      ))}
    </div>
  );
}
Responsive Containers
tsx
// components/recharts/ResponsiveChartContainer.tsx
'use client';

import React, { useState, useEffect } from 'react';
import { ResponsiveContainer } from 'recharts';

interface ResponsiveChartProps {
  children: React.ReactNode;
  aspectRatio?: number;
  minHeight?: number;
  maxHeight?: number;
  debounceResize?: number;
}

export function ResponsiveChartContainer({
  children,
  aspectRatio = 16 / 9,
  minHeight = 200,
  maxHeight = 600,
  debounceResize = 150,
}: ResponsiveChartProps) {
  const [dimensions, setDimensions] = useState({ width: 0, height: 0 });

  useEffect(() => {
    const calculateDimensions = () => {
      const container = document.getElementById('chart-container');
      if (container) {
        const width = container.clientWidth;
        const height = Math.min(
          Math.max(width / aspectRatio, minHeight),
          maxHeight
        );
        setDimensions({ width, height });
      }
    };

    calculateDimensions();

    let timeoutId: NodeJS.Timeout;
    const handleResize = () => {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(calculateDimensions, debounceResize);
    };

    window.addEventListener('resize', handleResize);
    return () => {
      window.removeEventListener('resize', handleResize);
      clearTimeout(timeoutId);
    };
  }, [aspectRatio, minHeight, maxHeight, debounceResize]);

  return (
    <div id="chart-container" className="w-full">
      <ResponsiveContainer width="100%" height={dimensions.height}>
        {children}
      </ResponsiveContainer>
    </div>
  );
}

// Hook per breakpoints responsive
export function useChartResponsive() {
  const [breakpoint, setBreakpoint] = useState<'sm' | 'md' | 'lg' | 'xl'>('lg');

  useEffect(() => {
    const handleResize = () => {
      const width = window.innerWidth;
      if (width < 640) setBreakpoint('sm');
      else if (width < 768) setBreakpoint('md');
      else if (width < 1024) setBreakpoint('lg');
      else setBreakpoint('xl');
    };

    handleResize();
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  const chartConfig = {
    sm: { barSize: 10, fontSize: 10, legend: false },
    md: { barSize: 15, fontSize: 11, legend: true },
    lg: { barSize: 20, fontSize: 12, legend: true },
    xl: { barSize: 25, fontSize: 14, legend: true },
  };

  return {
    breakpoint,
    config: chartConfig[breakpoint],
    isMobile: breakpoint === 'sm' || breakpoint === 'md',
  };
}
Animations
tsx
// components/recharts/AnimatedCharts.tsx
'use client';

import React, { useState, useEffect } from 'react';
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, Area } from 'recharts';

interface AnimatedDataPoint {
  time: number;
  value: number;
}

export function AnimatedLineChart() {
  const [data, setData] = useState<AnimatedDataPoint[]>([]);
  const [animationProgress, setAnimationProgress] = useState(0);

  useEffect(() => {
    const generateData = () => {
      const now = Date.now();
      const newData: AnimatedDataPoint[] = [];
      for (let i = 0; i < 20; i++) {
        newData.push({
          time: now - (19 - i) * 3600000,
          value: Math.random() * 100 + 50,
        });
      }
      setData(newData);
    };

    generateData();
  }, []);

  useEffect(() => {
    const interval = setInterval(() => {
      setData(prev => {
        const newData = [...prev.slice(1)];
        newData.push({
          time: Date.now(),
          value: Math.random() * 100 + 50,
        });
        return newData;
      });
    }, 2000);

    return () => clearInterval(interval);
  }, []);

  // Animazione di entrata
  const animatedData = data.map((point, index) => ({
    ...point,
    animatedValue: point.value * animationProgress,
  }));

  useEffect(() => {
    const timer = setTimeout(() => {
      if (animationProgress < 1) {
        setAnimationProgress(prev => Math.min(prev + 0.05, 1));
      }
    }, 50);

    return () => clearTimeout(timer);
  }, [animationProgress]);

  return (
    <div className="relative">
      <LineChart width={600} height={300} data={animatedData}>
        <CartesianGrid strokeDasharray="3 3" stroke="#f0f0f0" />
        <XAxis 
          dataKey="time" 
          tickFormatter={(time) => new Date(time).getHours() + ':00'}
        />
        <YAxis />
        <Tooltip 
          labelFormatter={(time) => new Date(time).toLocaleTimeString()}
        />
        <Line
          type="monotone"
          dataKey="animatedValue"
          stroke="#8884d8"
          strokeWidth={2}
          dot={{ r: 4 }}
          activeDot={{ r: 6 }}
          animationDuration={1000}
          animationEasing="ease-in-out"
        />
      </LineChart>
      
      {animationProgress < 1 && (
        <div className="absolute inset-0 flex items-center justify-center bg-white/80">
          <div className="text-gray-600">Loading animation...</div>
        </div>
      )}
    </div>
  );
}

export function StaggeredBarAnimation() {
  const [activeBars, setActiveBars] = useState<number[]>([]);

  useEffect(() => {
    const timer = setTimeout(() => {
      if (activeBars.length < 5) {
        setActiveBars(prev => [...prev, prev.length]);
      }
    }, 300);

    return () => clearTimeout(timer);
  }, [activeBars]);

  const data = [
    { name: 'A', value: 400 },
    { name: 'B', value: 300 },
    { name: 'C', value: 200 },
    { name: 'D', value: 278 },
    { name: 'E', value: 189 },
  ];

  return (
    <div className="flex items-end h-48 space-x-2 p-4">
      {data.map((item, index) => (
        <div key={item.name} className="flex flex-col items-center">
          <div
            className={`w-12 bg-blue-500 rounded-t transition-all duration-500 ${
              activeBars.includes(index) 
                ? 'opacity-100' 
                : 'opacity-0 h-0'
            }`}
            style={{
              height: activeBars.includes(index) 
                ? `${(item.value / 500) * 100}%` 
                : '0%',
              transitionDelay: `${index * 100}ms`,
            }}
          />
          <div className="mt-2 text-sm">{item.name}</div>
        </div>
      ))}
    </div>
  );
}
Brush per Zoom
tsx
// components/recharts/BrushZoom.tsx
'use client';

import React, { useState } from 'react';
import {
  LineChart,
  Line,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  Brush,
  AreaChart,
  Area,
  ResponsiveContainer,
} from 'recharts';

const generateTimeSeriesData = () => {
  const data = [];
  let value = 100;
  const now = Date.now();
  
  for (let i = 0; i < 365; i++) {
    const date = new Date(now - (364 - i) * 24 * 3600 * 1000);
    value += (Math.random() - 0.5) * 20;
    value = Math.max(50, Math.min(200, value));
    
    data.push({
      date: date.getTime(),
      value: Math.round(value),
      volume: Math.round(Math.random() * 1000 + 500),
    });
  }
  
  return data;
};

export function BrushZoomChart() {
  const [data] = useState(generateTimeSeriesData());
  const [zoomDomain, setZoomDomain] = useState<[number, number]>([0, 100]);
  
  const handleBrushChange = (domain: { startIndex?: number; endIndex?: number }) => {
    if (domain.startIndex !== undefined && domain.endIndex !== undefined) {
      setZoomDomain([domain.startIndex, domain.endIndex]);
    }
  };
  
  const zoomedData = data.slice(zoomDomain[0], zoomDomain[1] + 1);

  return (
    <div className="space-y-6">
      {/* Chart principale con zoom */}
      <div className="h-80">
        <ResponsiveContainer width="100%" height="100%">
          <AreaChart data={zoomedData}>
            <CartesianGrid strokeDasharray="3 3" stroke="#f0f0f0" />
            <XAxis
              dataKey="date"
              tickFormatter={(date) => new Date(date).toLocaleDateString('en-US', {
                month: 'short',
                day: 'numeric',
              })}
            />
            <YAxis />
            <Tooltip
              labelFormatter={(date) => new Date(date).toLocaleDateString()}
            />
            <Area
              type="monotone"
              dataKey="value"
              stroke="#8884d8"
              fill="#8884d8"
              fillOpacity={0.3}
            />
            <Line
              type="monotone"
              dataKey="volume"
              stroke="#82ca9d"
              strokeWidth={1}
              dot={false}
              yAxisId={1}
            />
            <YAxis yAxisId={1} orientation="right" />
          </AreaChart>
        </ResponsiveContainer>
      </div>
      
      {/* Brush per lo zoom */}
      <div className="h-24">
        <ResponsiveContainer width="100%" height="100%">
          <LineChart data={data}>
            <CartesianGrid strokeDasharray="3 3" stroke="#f5f5f5" />
            <XAxis
              dataKey="date"
              tick={false}
              axisLine={false}
            />
            <YAxis hide />
            <Line
              type="monotone"
              dataKey="value"
              stroke="#8884d8"
              strokeWidth={1}
              dot={false}
            />
            <Brush
              dataKey="date"
              height={30}
              stroke="#8884d8"
              startIndex={zoomDomain[0]}
              endIndex={zoomDomain[1]}
              onChange={handleBrushChange}
              tickFormatter={(date) => new Date(date).toLocaleDateString('MMM dd')}
            />
          </LineChart>
        </ResponsiveContainer>
      </div>
      
      {/* Controlli zoom */}
      <div className="flex items-center justify-between">
        <div className="text-sm text-gray-600">
          Showing {zoomedData.length} of {data.length} days
        </div>
        <div className="flex space-x-2">
          <button
            onClick={() => setZoomDomain([0, 30])}
            className="px-3 py-1 text-sm bg-gray-100 rounded hover:bg-gray-200"
          >
            Last 30 days
          </button>
          <button
            onClick={() => setZoomDomain([0, 90])}
            className="px-3 py-1 text-sm bg-gray-100 rounded hover:bg-gray-200"
          >
            Last 90 days
          </button>
          <button
            onClick={() => setZoomDomain([0, data.length - 1])}
            className="px-3 py-1 text-sm bg-gray-100 rounded hover:bg-gray-200"
          >
            All time
          </button>
        </div>
      </div>
    </div>
  );
}

export function DualBrushChart() {
  const [data] = useState(generateTimeSeriesData());
  const [brushDomain, setBrushDomain] = useState<[number, number]>([0.3, 0.7]);

  return (
    <div className="h-96">
      <ResponsiveContainer width="100%" height="100%">
        <AreaChart data={data}>
          <CartesianGrid strokeDasharray="3 3" />
          <XAxis
            dataKey="date"
            tickFormatter={(date) => new Date(date).toLocaleDateString()}
          />
          <YAxis />
          <Tooltip />
          <Area
            type="monotone"
            dataKey="value"
            stroke="#8884d8"
            fill="#8884d8"
            fillOpacity={0.3}
          />
          <Brush
            dataKey="date"
            height={30}
            stroke="#8884d8"
            startIndex={Math.floor(data.length * brushDomain[0])}
            endIndex={Math.floor(data.length * brushDomain[1])}
            onChange={(domain) => {
              if (domain.startIndex !== undefined && domain.endIndex !== undefined) {
                setBrushDomain([
                  domain.startIndex / data.length,
                  domain.endIndex / data.length,
                ]);
              }
            }}
          >
            <AreaChart>
              <Area
                type="monotone"
                dataKey="value"
                stroke="#8884d8"
                fill="#8884d8"
                fillOpacity={0.6}
              />
            </AreaChart>
          </Brush>
        </AreaChart>
      </ResponsiveContainer>
    </div>
  );
}
§ TREMOR COMPONENTS
Card, Metric, Text Components
tsx
// components/tremor/TremorBase.tsx
'use client';

import React from 'react';
import {
  Card,
  Metric,
  Text,
  Title,
  Subtitle,
  Badge,
  Button,
  Icon,
} from '@tremor/react';
import {
  ArrowUpIcon,
  ArrowDownIcon,
  CashIcon,
  UsersIcon,
  ShoppingCartIcon,
  TrendingUpIcon,
} from '@heroicons/react/outline';

export function KPICard({
  title,
  value,
  prefix = '',
  suffix = '',
  change,
  changeType = 'neutral',
  loading = false,
}: {
  title: string;
  value: number | string;
  prefix?: string;
  suffix?: string;
  change?: number;
  changeType?: 'positive' | 'negative' | 'neutral';
  loading?: boolean;
}) {
  const getChangeColor = () => {
    switch (changeType) {
      case 'positive': return 'text-emerald-700';
      case 'negative': return 'text-red-700';
      default: return 'text-gray-700';
    }
  };

  const getChangeIcon = () => {
    switch (changeType) {
      case 'positive': return ArrowUpIcon;
      case 'negative': return ArrowDownIcon;
      default: return TrendingUpIcon;
    }
  };

  const ChangeIcon = getChangeIcon();

  return (
    <Card className="max-w-xs">
      <Text>{title}</Text>
      {loading ? (
        <div className="h-8 bg-gray-200 animate-pulse rounded mt-2" />
      ) : (
        <Metric>
          {prefix}
          {typeof value === 'number' 
            ? new Intl.NumberFormat().format(value)
            : value
          }
          {suffix}
        </Metric>
      )}
      {change !== undefined && (
        <div className={`flex items-center mt-2 ${getChangeColor()}`}>
          <ChangeIcon className="h-4 w-4 mr-1" />
          <Text>
            {change > 0 ? '+' : ''}{change}%
          </Text>
          <Text className="ml-2 text-gray-600">vs last month</Text>
        </div>
      )}
    </Card>
  );
}

export function MetricGrid() {
  const metrics = [
    {
      title: 'Revenue',
      value: 234567,
      prefix: '$',
      change: 12.5,
      changeType: 'positive' as const,
      icon: CashIcon,
    },
    {
      title: 'Active Users',
      value: 5432,
      change: -3.2,
      changeType: 'negative' as const,
      icon: UsersIcon,
    },
    {
      title: 'Conversions',
      value: 456,
      change: 8.9,
      changeType: 'positive' as const,
      icon: ShoppingCartIcon,
    },
  ];

  return (
    <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
      {metrics.map((metric, index) => {
        const IconComponent = metric.icon;
        return (
          <Card key={index} className="relative overflow-hidden">
            <div className="flex items-start justify-between">
              <div>
                <Text className="text-gray-600">{metric.title}</Text>
                <Metric className="mt-2">
                  {metric.prefix}
                  {new Intl.NumberFormat().format(metric.value)}
                </Metric>
                <div className={`flex items-center mt-4 ${
                  metric.changeType === 'positive' 
                    ? 'text-emerald-600' 
                    : 'text-red-600'
                }`}>
                  {metric.changeType === 'positive' 
                    ? <ArrowUpIcon className="h-4 w-4 mr-1" />
                    : <ArrowDownIcon className="h-4 w-4 mr-1" />
                  }
                  <Text>
                    {metric.change > 0 ? '+' : ''}{metric.change}%
                  </Text>
                </div>
              </div>
              <div className="p-3 bg-blue-50 rounded-lg">
                <IconComponent className="h-6 w-6 text-blue-600" />
              </div>
            </div>
            <div className="absolute bottom-0 left-0 right-0 h-1 bg-gradient-to-r from-blue-500 to-purple-500" />
          </Card>
        );
      })}
    </div>
  );
}

export function ComparisonCard() {
  return (
    <Card className="max-w-md">
      <div className="flex items-center justify-between">
        <div>
          <Title>Monthly Recurring Revenue</Title>
          <Subtitle>Last 30 days vs previous period</Subtitle>
        </div>
        <Badge color="emerald" size="lg">
          On Track
        </Badge>
      </div>
      
      <div className="mt-6 space-y-4">
        <div>
          <div className="flex justify-between mb-1">
            <Text>Current Period</Text>
            <Text className="font-semibold">$45,231.89</Text>
          </div>
          <div className="h-2 bg-gray-200 rounded-full overflow-hidden">
            <div 
              className="h-full bg-emerald-500 rounded-full" 
              style={{ width: '75%' }}
            />
          </div>
        </div>
        
        <div>
          <div className="flex justify-between mb-1">
            <Text>Previous Period</Text>
            <Text className="font-semibold">$38,456.12</Text>
          </div>
          <div className="h-2 bg-gray-200 rounded-full overflow-hidden">
            <div 
              className="h-full bg-blue-500 rounded-full" 
              style={{ width: '65%' }}
            />
          </div>
        </div>
      </div>
      
      <div className="mt-6 pt-6 border-t border-gray-200">
        <div className="flex items-center justify-between">
          <div>
            <Text className="text-gray-600">Growth</Text>
            <Metric className="text-emerald-600">+17.6%</Metric>
          </div>
          <Button size="xs" variant="light" icon={ArrowUpIcon}>
            View Details
          </Button>
        </div>
      </div>
    </Card>
  );
}
AreaChart, BarChart, LineChart, DonutChart
tsx
// components/tremor/TremorCharts.tsx
'use client';

import React from 'react';
import {
  AreaChart as TremorAreaChart,
  BarChart as TremorBarChart,
  LineChart as TremorLineChart,
  DonutChart as TremorDonutChart,
  Card,
  Title,
  Subtitle,
  Text,
} from '@tremor/react';

const chartdata = [
  { date: 'Jan 23', Sales: 2890, Profit: 2400, Customers: 1398 },
  { date: 'Feb 23', Sales: 1890, Profit: 1398, Customers: 9800 },
  { date: 'Mar 23', Sales: 3890, Profit: 2980, Customers: 3908 },
  { date: 'Apr 23', Sales: 3490, Profit: 4300, Customers: 4800 },
  { date: 'May 23', Sales: 4490, Profit: 4100, Customers: 3800 },
  { date: 'Jun 23', Sales: 2490, Profit: 2400, Customers: 1908 },
];

const categories = ['Sales', 'Profit', 'Customers'];

const valueFormatter = (number: number) => 
  `$${Intl.NumberFormat('us').format(number).toString()}`;

const donutData = [
  { name: 'New York', sales: 9800 },
  { name: 'London', sales: 4567 },
  { name: 'Hong Kong', sales: 3908 },
  { name: 'San Francisco', sales: 2400 },
  { name: 'Singapore', sales: 1908 },
  { name: 'Zurich', sales: 1398 },
];

export function TremorAreaChartExample() {
  return (
    <Card>
      <Title>Performance Overview</Title>
      <Subtitle>Monthly sales and profit trends</Subtitle>
      <TremorAreaChart
        className="mt-6 h-72"
        data={chartdata}
        index="date"
        categories={categories}
        colors={['blue', 'emerald', 'amber']}
        valueFormatter={valueFormatter}
        showLegend={true}
        showGridLines={true}
        showAnimation={true}
        animationDuration={1000}
      />
    </Card>
  );
}

export function TremorBarChartStacked() {
  return (
    <Card>
      <Title>Quarterly Performance</Title>
      <Subtitle>Revenue by product category</Subtitle>
      <TremorBarChart
        className="mt-6"
        data={chartdata}
        index="date"
        categories={categories}
        colors={['blue', 'emerald', 'amber']}
        valueFormatter={valueFormatter}
        stack={true}
        showLegend={true}
        showAnimation={true}
      />
    </Card>
  );
}

export function TremorLineChartMultiple() {
  return (
    <Card>
      <Title>Growth Comparison</Title>
      <Subtitle>Year-over-year metrics</Subtitle>
      <TremorLineChart
        className="mt-6 h-72"
        data={chartdata}
        index="date"
        categories={categories}
        colors={['blue', 'emerald', 'amber']}
        valueFormatter={valueFormatter}
        curveType="natural"
        connectNulls={true}
        showLegend={true}
        showAnimation={true}
      />
    </Card>
  );
}

export function TremorDonutChartExample() {
  const valueFormatter = (number: number) => 
    `$${Intl.NumberFormat('us').format(number).toString()}`;

  return (
    <Card className="max-w-lg">
      <Title>Sales Distribution</Title>
      <Subtitle>By region</Subtitle>
      <TremorDonutChart
        className="mt-6"
        data={donutData}
        category="sales"
        index="name"
        valueFormatter={valueFormatter}
        colors={['blue', 'cyan', 'indigo', 'violet', 'fuchsia']}
        variant="donut"
        label="Total Sales"
        showLabel={true}
        showAnimation={true}
        onValueChange={(v) => console.log(v)}
      />
      <div className="mt-6 flex items-center justify-center space-x-4">
        {donutData.map((item) => (
          <div key={item.name} className="flex items-center">
            <div className="w-3 h-3 rounded-full bg-blue-500 mr-2" />
            <Text>{item.name}</Text>
          </div>
        ))}
      </div>
    </Card>
  );
}

export function TremorComparisonChart() {
  const comparisonData = [
    { month: 'Jan', '2023': 4000, '2024': 4400 },
    { month: 'Feb', '2023': 3000, '2024': 4200 },
    { month: 'Mar', '2023': 2000, '2024': 3800 },
    { month: 'Apr', '2023': 2780, '2024': 3900 },
    { month: 'May', '2023': 1890, '2024': 3500 },
    { month: 'Jun', '2023': 2390, '2024': 4100 },
  ];

  return (
    <Card>
      <div className="md:flex justify-between">
        <div>
          <Title>Year-over-Year Growth</Title>
          <Subtitle>Monthly revenue comparison</Subtitle>
        </div>
        <div className="flex space-x-4 mt-4 md:mt-0">
          <div className="flex items-center">
            <div className="w-3 h-3 rounded-full bg-blue-500 mr-2" />
            <Text>2023</Text>
          </div>
          <div className="flex items-center">
            <div className="w-3 h-3 rounded-full bg-emerald-500 mr-2" />
            <Text>2024</Text>
          </div>
        </div>
      </div>
      <TremorBarChart
        className="mt-6 h-60"
        data={comparisonData}
        index="month"
        categories={['2023', '2024']}
        colors={['blue', 'emerald']}
       