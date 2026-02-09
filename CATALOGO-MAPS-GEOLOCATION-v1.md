# CATALOGO MAPS-GEOLOCATION v1
§ DOCUMENTAZIONE TECNICA COMPLETA PER MAPS E GEOLOCATION IN REACT/NEXT.JS

---

§ §1. MAP LIBRARY COMPARISON

§ **TABELLA COMPARATIVA DETTAGLIATA**

| Library | Cost | Features | Bundle Size | Performance | Customization | Best For | Worst For |
|---------|------|----------|-------------|-------------|---------------|----------|-----------|
| **Mapbox GL JS** | Free: 50k loads/mo<br>Paid: $0.50-5/1k loads | ⭐⭐⭐⭐⭐<br>3D, vector tiles, custom styles, GL shaders | 200kB gzipped<br>+ WASM runtime | Excellent (GPU) | Very high | Production apps, beautiful maps, custom styling | Budget projects, simple needs |
| **Leaflet** | Free (MIT) | ⭐⭐⭐⭐<br>Raster tiles, plugins, markers, polygons | 40kB gzipped<br>No dependencies | Good (CPU) | High (via plugins) | Simple maps, lightweight, OSS purists | 3D maps, vector tiles |
| **Google Maps** | $0.007/load (first 100k/mo free) | ⭐⭐⭐⭐⭐<br>Street View, Places, Directions, Street View | SDK varies<br>~500kB total | Excellent | Medium (Google style) | Google ecosystem, business apps | Cost-sensitive, OSS requirements |
| **MapLibre GL** | Free (BSD) | ⭐⭐⭐⭐<br>Fork of Mapbox GL 1.x, vector tiles | 200kB gzipped<br>Similar to Mapbox | Very good | High (Mapbox-like) | Mapbox alternative, OSS vector maps | Latest Mapbox features |
| **react-map-gl** | Free (MIT) | ⭐⭐⭐⭐<br>React wrapper for Mapbox | Depends on Mapbox | Excellent | High (React native) | React + Mapbox integration | Non-React apps |

§ **DECISION TREE**

Need vector tiles & 3D? → Mapbox GL JS / MapLibre
Need lightweight & simple? → Leaflet
Already using Google services? → Google Maps
React app with Mapbox? → react-map-gl
Budget constraints? → Leaflet or MapLibre

§ **RACCOMANDAZIONE FINALE**

typescript
// Per la maggior parte delle applicazioni Next.js:
const RECOMMENDED_STACK = {
  primary: 'Mapbox GL JS', // Per bellezza e features
  fallback: 'Leaflet', // Per bundle size concerns
  geocoding: 'Mapbox Geocoding', // Built-in con Mapbox
  places: 'Google Places API', // Migliore autocomplete
  directions: 'Mapbox Directions', // Con Mapbox
  
  // Alternative OSS:
  ossAlternative: 'MapLibre GL + OpenStreetMap',
  
  // Per SSR/Next.js:
  dynamicImport: true, // Import dinamico per bundle splitting
  lazyLoading: true, // Lazy load maps
}

---

§ §2. MAPBOX SETUP

§ 2.1 INSTALLATION & CONFIGURATION

typescript
// lib/maps/mapbox-client.ts
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

// Configuration singleton
class MapboxClient {
  private static instance: MapboxClient;
  private isInitialized = false;
  
  private constructor() {
    // Initialize Mapbox
    const accessToken = process.env.NEXT_PUBLIC_MAPBOX_TOKEN;
    
    if (!accessToken) {
      console.warn(
        'Mapbox access token not found. ' +
        'Set NEXT_PUBLIC_MAPBOX_TOKEN environment variable.'
      );
      return;
    }
    
    mapboxgl.accessToken = accessToken;
    
    // Set RTL text plugin for internationalization
    mapboxgl.setRTLTextPlugin(
      'https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-rtl-text/v0.2.3/mapbox-gl-rtl-text.js',
      null,
      true // Lazy load the plugin
    );
    
    this.isInitialized = true;
  }
  
  static getInstance(): MapboxClient {
    if (!MapboxClient.instance) {
      MapboxClient.instance = new MapboxClient();
    }
    return MapboxClient.instance;
  }
  
  isReady(): boolean {
    return this.isInitialized;
  }
  
  getAccessToken(): string | undefined {
    return mapboxgl.accessToken;
  }
  
  // Helper to check if Mapbox is available
  static isSupported(): boolean {
    return mapboxgl.supported();
  }
  
  // Get supported language
  static getLanguage(): string {
    return navigator?.language || 'en';
  }
}

// Export singleton
export const mapboxClient = MapboxClient.getInstance();

// CSS overrides per Mapbox
export const mapboxStyles = `
  .mapboxgl-map {
    font-family: inherit;
  }
  
  .mapboxgl-popup-content {
    border-radius: 8px;
    padding: 16px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  }
  
  .mapboxgl-popup-close-button {
    font-size: 20px;
    padding: 8px;
    color: #666;
    
    &:hover {
      color: #000;
      background-color: transparent;
    }
  }
  
  .mapboxgl-ctrl-group {
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  }
  
  .mapboxgl-ctrl-group button {
    width: 36px;
    height: 36px;
    
    &:hover {
      background-color: #f5f5f5;
    }
  }
`;

// Map styles configuration
export const MAP_STYLES = {
  streets: 'mapbox://styles/mapbox/streets-v12',
  outdoors: 'mapbox://styles/mapbox/outdoors-v12',
  light: 'mapbox://styles/mapbox/light-v11',
  dark: 'mapbox://styles/mapbox/dark-v11',
  satellite: 'mapbox://styles/mapbox/satellite-v9',
  satelliteStreets: 'mapbox://styles/mapbox/satellite-streets-v12',
  
  // Custom styles
  customLight: process.env.NEXT_PUBLIC_MAPBOX_CUSTOM_LIGHT_STYLE,
  customDark: process.env.NEXT_PUBLIC_MAPBOX_CUSTOM_DARK_STYLE,
} as const;

export type MapStyle = keyof typeof MAP_STYLES;

typescript
// components/maps/MapboxProvider.tsx - Per SSR/CSR handling
'use client';

import { ReactNode, useEffect, useState } from 'react';
import dynamic from 'next/dynamic';
import { mapboxClient } from '@/lib/maps/mapbox-client';

interface MapboxProviderProps {
  children: ReactNode;
  fallback?: ReactNode;
}

// Dynamic import per Mapbox (per evitare bundle nel server)
const MapboxProviderComponent = dynamic(
  () => import('@/components/maps/MapboxProviderClient'),
  {
    ssr: false,
    loading: () => (
      <div className="flex h-full items-center justify-center">
        <div className="text-gray-500">Loading map...</div>
      </div>
    ),
  }
);

export function MapboxProvider({ children, fallback }: MapboxProviderProps) {
  const [isSupported, setIsSupported] = useState<boolean | null>(null);
  
  useEffect(() => {
    // Check browser support
    const supported = mapboxClient.isReady() && 
      typeof window !== 'undefined' && 
      window.mapboxgl?.supported?.();
    
    setIsSupported(supported ?? false);
  }, []);
  
  if (isSupported === null) {
    return fallback || (
      <div className="flex h-full items-center justify-center">
        <div className="text-gray-500">Checking browser support...</div>
      </div>
    );
  }
  
  if (!isSupported) {
    return fallback || (
      <div className="flex h-full flex-col items-center justify-center gap-4 p-8">
        <div className="text-lg font-semibold text-red-600">
          Map not supported
        </div>
        <p className="text-center text-gray-600">
          Your browser does not support interactive maps.
          <br />
          Please try updating your browser or use a different one.
        </p>
      </div>
    );
  }
  
  return <MapboxProviderComponent>{children}</MapboxProviderComponent>;
}

// Client component per Mapbox
export function MapboxProviderClient({ children }: { children: ReactNode }) {
  // Inietta CSS personalizzati
  useEffect(() => {
    const style = document.createElement('style');
    style.textContent = mapboxStyles;
    document.head.appendChild(style);
    
    return () => {
      document.head.removeChild(style);
    };
  }, []);
  
  return <>{children}</>;
}

§ 2.2 BASIC MAP COMPONENT

typescript
// components/maps/Map.tsx - Componente mappa principale
'use client';

import { useEffect, useRef, useState, useCallback } from 'react';
import mapboxgl from 'mapbox-gl';
import { MapboxProvider } from './MapboxProvider';
import { MAP_STYLES, MapStyle } from '@/lib/maps/mapbox-client';
import { cn } from '@/lib/utils';

// Types
export interface Viewport {
  latitude: number;
  longitude: number;
  zoom: number;
  bearing?: number;
  pitch?: number;
}

export interface MapProps {
  /** Initial viewport */
  initialViewport?: Partial<Viewport>;
  /** Map style */
  style?: MapStyle | string;
  /** Custom Mapbox style URL */
  styleUrl?: string;
  /** Container className */
  className?: string;
  /** Map container dimensions */
  width?: string | number;
  height?: string | number;
  /** Interactive controls */
  interactive?: boolean;
  showNavigation?: boolean;
  showFullscreen?: boolean;
  showScale?: boolean;
  /** Map events */
  onLoad?: (map: mapboxgl.Map) => void;
  onMove?: (viewport: Viewport) => void;
  onClick?: (event: mapboxgl.MapMouseEvent) => void;
  /** Children (markers, popups, etc.) */
  children?: React.ReactNode;
  /** Fit bounds */
  bounds?: mapboxgl.LngLatBoundsLike;
  /** Max bounds */
  maxBounds?: mapboxgl.LngLatBoundsLike;
  /** Min/max zoom */
  minZoom?: number;
  maxZoom?: number;
  /** Padding for fitBounds */
  fitBoundsOptions?: mapboxgl.FitBoundsOptions;
}

export function Map({
  initialViewport = {},
  style = 'streets',
  styleUrl,
  className,
  width = '100%',
  height = '100%',
  interactive = true,
  showNavigation = true,
  showFullscreen = true,
  showScale = true,
  onLoad,
  onMove,
  onClick,
  children,
  bounds,
  maxBounds,
  minZoom = 0,
  maxZoom = 22,
  fitBoundsOptions,
}: MapProps) {
  return (
    <MapboxProvider
      fallback={
        <div
          className={cn('bg-gray-100', className)}
          style={{ width, height }}
        />
      }
    >
      <MapComponent
        initialViewport={initialViewport}
        style={style}
        styleUrl={styleUrl}
        className={className}
        width={width}
        height={height}
        interactive={interactive}
        showNavigation={showNavigation}
        showFullscreen={showFullscreen}
        showScale={showScale}
        onLoad={onLoad}
        onMove={onMove}
        onClick={onClick}
        bounds={bounds}
        maxBounds={maxBounds}
        minZoom={minZoom}
        maxZoom={maxZoom}
        fitBoundsOptions={fitBoundsOptions}
      >
        {children}
      </MapComponent>
    </MapboxProvider>
  );
}

// Internal map component
function MapComponent({
  initialViewport,
  style,
  styleUrl,
  className,
  width,
  height,
  interactive,
  showNavigation,
  showFullscreen,
  showScale,
  onLoad,
  onMove,
  onClick,
  children,
  bounds,
  maxBounds,
  minZoom,
  maxZoom,
  fitBoundsOptions,
}: MapProps) {
  const mapContainer = useRef<HTMLDivElement>(null);
  const mapRef = useRef<mapboxgl.Map | null>(null);
  const [isLoaded, setIsLoaded] = useState(false);
  
  // Default viewport (centered on Rome)
  const defaultViewport: Viewport = {
    latitude: 41.9028,
    longitude: 12.4964,
    zoom: 13,
    ...initialViewport,
  };
  
  // Initialize map
  useEffect(() => {
    if (!mapContainer.current || mapRef.current) return;
    
    const styleToUse = styleUrl || MAP_STYLES[style as MapStyle] || MAP_STYLES.streets;
    
    const map = new mapboxgl.Map({
      container: mapContainer.current,
      style: styleToUse,
      center: [defaultViewport.longitude, defaultViewport.latitude],
      zoom: defaultViewport.zoom,
      bearing: defaultViewport.bearing,
      pitch: defaultViewport.pitch,
      interactive,
      minZoom,
      maxZoom,
      maxBounds,
      attributionControl: false,
      preserveDrawingBuffer: false, // Per performance
      antialias: true, // Smooth rendering
      locale: 'en', // Lingua mappe
    });
    
    // Add controls
    if (showNavigation) {
      map.addControl(new mapboxgl.NavigationControl({ showCompass: true }), 'top-right');
    }
    
    if (showFullscreen) {
      map.addControl(new mapboxgl.FullscreenControl(), 'top-right');
    }
    
    if (showScale) {
      map.addControl(new mapboxgl.ScaleControl({ unit: 'metric' }), 'bottom-left');
    }
    
    // Custom attribution
    map.addControl(
      new mapboxgl.AttributionControl({
        compact: true,
        customAttribution: '© Mapbox, © OpenStreetMap',
      }),
      'bottom-right'
    );
    
    // Handle map load
    map.on('load', () => {
      setIsLoaded(true);
      onLoad?.(map);
      
      // Fit bounds if provided
      if (bounds) {
        map.fitBounds(bounds, fitBoundsOptions);
      }
    });
    
    // Handle move events
    const handleMove = () => {
      const center = map.getCenter();
      const zoom = map.getZoom();
      const bearing = map.getBearing();
      const pitch = map.getPitch();
      
      onMove?.({
        latitude: center.lat,
        longitude: center.lng,
        zoom,
        bearing,
        pitch,
      });
    };
    
    map.on('move', handleMove);
    map.on('moveend', handleMove);
    
    // Handle click events
    if (onClick) {
      map.on('click', onClick);
    }
    
    mapRef.current = map;
    
    return () => {
      if (mapRef.current) {
        mapRef.current.remove();
        mapRef.current = null;
      }
    };
  }, []); // Solo una volta
  
  // Handle bounds updates
  useEffect(() => {
    if (!mapRef.current || !bounds || !isLoaded) return;
    
    mapRef.current.fitBounds(bounds, fitBoundsOptions);
  }, [bounds, isLoaded, fitBoundsOptions]);
  
  // Handle viewport updates
  useEffect(() => {
    if (!mapRef.current || !isLoaded) return;
    
    const map = mapRef.current;
    
    map.flyTo({
      center: [defaultViewport.longitude, defaultViewport.latitude],
      zoom: defaultViewport.zoom,
      bearing: defaultViewport.bearing,
      pitch: defaultViewport.pitch,
      duration: 1000,
    });
  }, [defaultViewport, isLoaded]);
  
  // Handle style updates
  useEffect(() => {
    if (!mapRef.current || !isLoaded) return;
    
    const styleToUse = styleUrl || MAP_STYLES[style as MapStyle] || MAP_STYLES.streets;
    mapRef.current.setStyle(styleToUse);
  }, [style, styleUrl, isLoaded]);
  
  return (
    <div className="relative">
      <div
        ref={mapContainer}
        className={cn('map-container', className)}
        style={{
          width,
          height,
          minHeight: typeof height === 'number' ? `${height}px` : undefined,
        }}
      />
      
      {/* Render children (markers, popups) dopo caricamento */}
      {isLoaded && children && (
        <MapChildrenRenderer map={mapRef.current!}>
          {children}
        </MapChildrenRenderer>
      )}
      
      {/* Loading overlay */}
      {!isLoaded && (
        <div className="absolute inset-0 flex items-center justify-center bg-gray-100/80">
          <div className="flex flex-col items-center gap-2">
            <div className="h-8 w-8 animate-spin rounded-full border-2 border-gray-300 border-t-blue-600" />
            <div className="text-sm text-gray-600">Loading map...</div>
          </div>
        </div>
      )}
    </div>
  );
}

// Helper per renderizzare children con contesto map
function MapChildrenRenderer({
  map,
  children,
}: {
  map: mapboxgl.Map;
  children: React.ReactNode;
}) {
  const [mounted, setMounted] = useState(false);
  
  useEffect(() => {
    setMounted(true);
  }, []);
  
  if (!mounted) return null;
  
  // Qui potresti usare un context provider per passare la map ai children
  return <>{children}</>;
}

// Hook per accedere alla map instance
export function useMap() {
  const [map, setMap] = useState<mapboxgl.Map | null>(null);
  
  useEffect(() => {
    // Potresti usare un context per accedere alla map
    // Questa è un'implementazione semplificata
    const interval = setInterval(() => {
      const mapElement = document.querySelector('.mapboxgl-map');
      if (mapElement && (mapElement as any)._mapboxgl) {
        setMap((mapElement as any)._mapboxgl);
        clearInterval(interval);
      }
    }, 100);
    
    return () => clearInterval(interval);
  }, []);
  
  return map;
}

---

§ §3. MAP FEATURES

§ 3.1 MARKERS

typescript
// components/maps/MapMarker.tsx
'use client';

import { useEffect, useRef, useState } from 'react';
import mapboxgl from 'mapbox-gl';
import { useMap } from './Map';
import { cn } from '@/lib/utils';

export interface MapMarkerProps {
  /** Marker position */
  latitude: number;
  longitude: number;
  
  /** Marker appearance */
  color?: string;
  size?: number;
  icon?: string | React.ReactNode;
  iconUrl?: string;
  iconSize?: [number, number];
  iconAnchor?: [number, number];
  
  /** Popup content */
  popup?: React.ReactNode;
  popupOptions?: mapboxgl.PopupOptions;
  
  /** Interaction */
  draggable?: boolean;
  onClick?: (event: mapboxgl.MapMouseEvent) => void;
  onDragStart?: (event: mapboxgl.MapMouseEvent) => void;
  onDragEnd?: (event: mapboxgl.MapMouseEvent) => void;
  
  /** Class names */
  className?: string;
  popupClassName?: string;
}

export function MapMarker({
  latitude,
  longitude,
  color = '#3b82f6',
  size = 24,
  icon,
  iconUrl,
  iconSize = [24, 24],
  iconAnchor = [12, 24],
  popup,
  popupOptions,
  draggable = false,
  onClick,
  onDragStart,
  onDragEnd,
  className,
  popupClassName,
}: MapMarkerProps) {
  const map = useMap();
  const markerRef = useRef<mapboxgl.Marker | null>(null);
  const popupRef = useRef<mapboxgl.Popup | null>(null);
  const [isDragging, setIsDragging] = useState(false);
  
  // Create/update marker
  useEffect(() => {
    if (!map) return;
    
    // Cleanup existing marker
    if (markerRef.current) {
      markerRef.current.remove();
      markerRef.current = null;
    }
    
    // Create marker element
    const el = document.createElement('div');
    el.className = cn('map-marker', className);
    
    if (iconUrl) {
      // Use image icon
      const img = document.createElement('img');
      img.src = iconUrl;
      img.style.width = `${iconSize[0]}px`;
      img.style.height = `${iconSize[1]}px`;
      img.style.objectFit = 'contain';
      el.appendChild(img);
    } else if (icon && typeof icon === 'string') {
      // Use emoji or text
      el.innerHTML = icon;
      el.style.fontSize = `${size}px`;
      el.style.lineHeight = '1';
    } else {
      // Default marker with color
      el.style.width = `${size}px`;
      el.style.height = `${size}px`;
      el.style.backgroundColor = color;
      el.style.borderRadius = '50%';
      el.style.border = '2px solid white';
      el.style.boxShadow = '0 2px 6px rgba(0, 0, 0, 0.3)';
      el.style.cursor = 'pointer';
      el.style.transition = 'transform 0.2s';
      
      // Add inner dot
      const innerDot = document.createElement('div');
      innerDot.style.width = '40%';
      innerDot.style.height = '40%';
      innerDot.style.backgroundColor = 'white';
      innerDot.style.borderRadius = '50%';
      innerDot.style.position = 'absolute';
      innerDot.style.top = '50%';
      innerDot.style.left = '50%';
      innerDot.style.transform = 'translate(-50%, -50%)';
      el.appendChild(innerDot);
    }
    
    // Create marker
    const marker = new mapboxgl.Marker({
      element: el,
      draggable,
      anchor: iconAnchor as any,
    })
      .setLngLat([longitude, latitude])
      .addTo(map);
    
    // Add click handler
    if (onClick) {
      el.addEventListener('click', (e) => {
        e.stopPropagation();
        const mapEvent = {
          lngLat: marker.getLngLat(),
          originalEvent: e,
          target: marker.getElement(),
        } as unknown as mapboxgl.MapMouseEvent;
        onClick(mapEvent);
      });
    }
    
    // Add drag handlers
    if (draggable) {
      marker.on('dragstart', (e) => {
        setIsDragging(true);
        onDragStart?.(e);
      });
      
      marker.on('dragend', (e) => {
        setIsDragging(false);
        onDragEnd?.(e);
      });
    }
    
    // Create popup if needed
    if (popup) {
      const popupEl = new mapboxgl.Popup({
        closeButton: true,
        closeOnClick: true,
        maxWidth: '300px',
        className: cn('map-popup', popupClassName),
        ...popupOptions,
      });
      
      if (typeof popup === 'string') {
        popupEl.setHTML(popup);
      } else {
        const container = document.createElement('div');
        // In React 18+, userei ReactDOM.createRoot
        // Per semplicità, usiamo innerHTML se popup è string
        popupEl.setDOMContent(container);
      }
      
      marker.setPopup(popupEl);
      popupRef.current = popupEl;
    }
    
    markerRef.current = marker;
    
    return () => {
      if (markerRef.current) {
        markerRef.current.remove();
        markerRef.current = null;
      }
      if (popupRef.current) {
        popupRef.current.remove();
        popupRef.current = null;
      }
    };
  }, [
    map,
    latitude,
    longitude,
    color,
    size,
    icon,
    iconUrl,
    iconSize,
    iconAnchor,
    draggable,
    className,
    popupClassName,
  ]);
  
  // Handle popup updates
  useEffect(() => {
    if (!popup || !markerRef.current || !popupRef.current) return;
    
    if (typeof popup === 'string') {
      popupRef.current.setHTML(popup);
    }
    // Altrimenti, gestisci React content...
  }, [popup]);
  
  // Drag state style
  useEffect(() => {
    if (!markerRef.current) return;
    
    const el = markerRef.current.getElement();
    if (!el) return;
    
    if (isDragging) {
      el.style.transform = 'scale(1.2)';
      el.style.zIndex = '1000';
    } else {
      el.style.transform = 'scale(1)';
      el.style.zIndex = 'auto';
    }
  }, [isDragging]);
  
  return null;
}

§ 3.2 MARKER CLUSTERING

typescript
// components/maps/MarkerCluster.tsx
'use client';

import { useEffect, useRef } from 'react';
import mapboxgl from 'mapbox-gl';
import Supercluster from 'supercluster';
import { useMap } from './Map';
import { MapMarker } from './MapMarker';

export interface ClusterPoint {
  id: string | number;
  latitude: number;
  longitude: number;
  properties?: Record<string, any>;
}

export interface MarkerClusterProps {
  /** Array of points to cluster */
  points: ClusterPoint[];
  
  /** Cluster options */
  clusterRadius?: number;
  clusterMaxZoom?: number;
  clusterMinZoom?: number;
  
  /** Marker rendering */
  renderMarker?: (point: ClusterPoint) => React.ReactNode;
  renderCluster?: (cluster: {
    id: number;
    pointCount: number;
    latitude: number;
    longitude: number;
  }) => React.ReactNode;
  
  /** Events */
  onClusterClick?: (cluster: {
    id: number;
    pointCount: number;
    latitude: number;
    longitude: number;
  }) => void;
  onMarkerClick?: (point: ClusterPoint) => void;
}

export function MarkerCluster({
  points,
  clusterRadius = 50,
  clusterMaxZoom = 16,
  clusterMinZoom = 0,
  renderMarker,
  renderCluster,
  onClusterClick,
  onMarkerClick,
}: MarkerClusterProps) {
  const map = useMap();
  const clusterRef = useRef<Supercluster | null>(null);
  const markersRef = useRef<Set<string>>(new Set());
  
  useEffect(() => {
    if (!map) return;
    
    // Initialize supercluster
    const cluster = new Supercluster({
      radius: clusterRadius,
      maxZoom: clusterMaxZoom,
      minZoom: clusterMinZoom,
    });
    
    // Prepare GeoJSON features
    const features = points.map(point => ({
      type: 'Feature' as const,
      geometry: {
        type: 'Point' as const,
        coordinates: [point.longitude, point.latitude],
      },
      properties: {
        id: point.id,
        ...point.properties,
      },
    }));
    
    cluster.load(features);
    clusterRef.current = cluster;
    
    // Update clusters on map move
    const updateClusters = () => {
      if (!clusterRef.current || !map) return;
      
      const zoom = map.getZoom();
      const bounds = map.getBounds();
      
      const bbox: [number, number, number, number] = [
        bounds.getWest(),
        bounds.getSouth(),
        bounds.getEast(),
        bounds.getNorth(),
      ];
      
      const clusters = clusterRef.current.getClusters(bbox, Math.floor(zoom));
      
      // Clear existing markers
      markersRef.current.forEach(id => {
        const el = document.getElementById(`cluster-marker-${id}`);
        if (el) el.remove();
      });
      markersRef.current.clear();
      
      // Add new markers/clusters
      clusters.forEach(cluster => {
        const isCluster = cluster.properties?.cluster;
        const id = cluster.properties?.cluster_id || cluster.properties?.id;
        
        if (isCluster) {
          // Render cluster
          const clusterData = {
            id: cluster.properties.cluster_id,
            pointCount: cluster.properties.point_count,
            latitude: cluster.geometry.coordinates[1],
            longitude: cluster.geometry.coordinates[0],
          };
          
          if (renderCluster) {
            // Custom cluster rendering
            const markerId = `cluster-${clusterData.id}`;
            markersRef.current.add(markerId);
            
            // Qui implementeresti il rendering del cluster
            // Per semplicità, usiamo MapMarker
          } else {
            // Default cluster marker
            const el = document.createElement('div');
            el.id = `cluster-${clusterData.id}`;
            el.className = 'cluster-marker';
            el.innerHTML = `
              <div style="
                width: 40px;
                height: 40px;
                background-color: rgba(59, 130, 246, 0.8);
                border-radius: 50%;
                display: flex;
                align-items: center;
                justify-content: center;
                color: white;
                font-weight: bold;
                font-size: 14px;
                border: 3px solid white;
                box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
                cursor: pointer;
              ">
                ${clusterData.pointCount}
              </div>
            `;
            
            el.addEventListener('click', (e) => {
              e.stopPropagation();
              onClusterClick?.(clusterData);
              
              // Zoom to cluster bounds
              const expansionZoom = clusterRef.current?.getClusterExpansionZoom(
                clusterData.id
              );
              if (expansionZoom) {
                map.flyTo({
                  center: [clusterData.longitude, clusterData.latitude],
                  zoom: expansionZoom,
                  duration: 500,
                });
              }
            });
            
            new mapboxgl.Marker({ element: el })
              .setLngLat([clusterData.longitude, clusterData.latitude])
              .addTo(map);
            
            markersRef.current.add(`cluster-${clusterData.id}`);
          }
        } else {
          // Render single point
          const pointData: ClusterPoint = {
            id: cluster.properties.id,
            latitude: cluster.geometry.coordinates[1],
            longitude: cluster.geometry.coordinates[0],
            properties: cluster.properties,
          };
          
          if (renderMarker) {
            // Custom marker rendering
          } else {
            // Default marker
            const markerId = `marker-${pointData.id}`;
            markersRef.current.add(markerId);
            
            const el = document.createElement('div');
            el.id = markerId;
            el.className = 'single-marker';
            
            el.addEventListener('click', (e) => {
              e.stopPropagation();
              onMarkerClick?.(pointData);
            });
            
            new mapboxgl.Marker({ element: el })
              .setLngLat([pointData.longitude, pointData.latitude])
              .addTo(map);
          }
        }
      });
    };
    
    // Initial update
    updateClusters();
    
    // Update on move
    map.on('move', updateClusters);
    map.on('moveend', updateClusters);
    map.on('zoom', updateClusters);
    
    return () => {
      if (map) {
        map.off('move', updateClusters);
        map.off('moveend', updateClusters);
        map.off('zoom', updateClusters);
      }
      
      // Cleanup markers
      markersRef.current.forEach(id => {
        const el = document.getElementById(id);
        if (el) el.remove();
      });
      markersRef.current.clear();
    };
  }, [
    map,
    points,
    clusterRadius,
    clusterMaxZoom,
    clusterMinZoom,
    renderMarker,
    renderCluster,
    onClusterClick,
    onMarkerClick,
  ]);
  
  return null;
}

§ 3.3 POLYGONS & SHAPES

typescript
// components/maps/MapPolygon.tsx
'use client';

import { useEffect, useRef } from 'react';
import mapboxgl from 'mapbox-gl';
import { useMap } from './Map';

export interface MapPolygonProps {
  /** Polygon coordinates (first and last should be same) */
  coordinates: [number, number][][]; // Array of rings
  
  /** Styling */
  fillColor?: string;
  fillOpacity?: number;
  strokeColor?: string;
  strokeWidth?: number;
  strokeOpacity?: number;
  
  /** Interaction */
  interactive?: boolean;
  onClick?: (event: mapboxgl.MapMouseEvent) => void;
  onMouseEnter?: (event: mapboxgl.MapMouseEvent) => void;
  onMouseLeave?: (event: mapboxgl.MapMouseEvent) => void;
  
  /** Layer ID (for updates) */
  layerId?: string;
  sourceId?: string;
}

export function MapPolygon({
  coordinates,
  fillColor = '#3b82f6',
  fillOpacity = 0.5,
  strokeColor = '#1d4ed8',
  strokeWidth = 2,
  strokeOpacity = 0.8,
  interactive = true,
  onClick,
  onMouseEnter,
  onMouseLeave,
  layerId = 'polygon-layer',
  sourceId = 'polygon-source',
}: MapPolygonProps) {
  const map = useMap();
  const layerIdRef = useRef(layerId);
  const sourceIdRef = useRef(sourceId);
  
  useEffect(() => {
    if (!map) return;
    
    // Ensure map is loaded
    if (!map.isStyleLoaded()) {
      map.once('load', () => addPolygon());
      return;
    }
    
    addPolygon();
    
    function addPolygon() {
      // Remove existing layer/source
      if (map.getLayer(layerIdRef.current)) {
        map.removeLayer(layerIdRef.current);
      }
      if (map.getSource(sourceIdRef.current)) {
        map.removeSource(sourceIdRef.current);
      }
      
      // Create GeoJSON
      const geoJson: GeoJSON.FeatureCollection = {
        type: 'FeatureCollection',
        features: [
          {
            type: 'Feature',
            geometry: {
              type: 'Polygon',
              coordinates,
            },
            properties: {},
          },
        ],
      };
      
      // Add source
      map.addSource(sourceIdRef.current, {
        type: 'geojson',
        data: geoJson,
      });
      
      // Add fill layer
      map.addLayer({
        id: layerIdRef.current,
        type: 'fill',
        source: sourceIdRef.current,
        paint: {
          'fill-color': fillColor,
          'fill-opacity': fillOpacity,
        },
      });
      
      // Add stroke layer
      map.addLayer({
        id: `${layerIdRef.current}-stroke`,
        type: 'line',
        source: sourceIdRef.current,
        paint: {
          'line-color': strokeColor,
          'line-width': strokeWidth,
          'line-opacity': strokeOpacity,
        },
      });
      
      // Add interaction
      if (interactive) {
        // Cursor changes
        map.on('mouseenter', layerIdRef.current, (e) => {
          map.getCanvas().style.cursor = 'pointer';
          onMouseEnter?.(e);
          
          // Highlight on hover
          map.setPaintProperty(
            layerIdRef.current,
            'fill-color',
            fillColor === '#3b82f6' ? '#2563eb' : fillColor
          );
        });
        
        map.on('mouseleave', layerIdRef.current, (e) => {
          map.getCanvas().style.cursor = '';
          onMouseLeave?.(e);
          
          // Reset color
          map.setPaintProperty(
            layerIdRef.current,
            'fill-color',
            fillColor
          );
        });
        
        if (onClick) {
          map.on('click', layerIdRef.current, onClick);
        }
      }
    }
    
    return () => {
      if (!map) return;
      
      if (map.getLayer(layerIdRef.current)) {
        map.removeLayer(layerIdRef.current);
      }
      if (map.getLayer(`${layerIdRef.current}-stroke`)) {
        map.removeLayer(`${layerIdRef.current}-stroke`);
      }
      if (map.getSource(sourceIdRef.current)) {
        map.removeSource(sourceIdRef.current);
      }
      
      // Remove event listeners
      map.off('mouseenter', layerIdRef.current);
      map.off('mouseleave', layerIdRef.current);
      map.off('click', layerIdRef.current);
    };
  }, [
    map,
    coordinates,
    fillColor,
    fillOpacity,
    strokeColor,
    strokeWidth,
    strokeOpacity,
    interactive,
    onClick,
    onMouseEnter,
    onMouseLeave,
  ]);
  
  return null;
}

---

§ §4. GEOLOCATION API

§ 4.1 GEOLOCATION HOOK

typescript
// hooks/use-geolocation.ts
'use client';

import { useState, useEffect, useCallback } from 'react';

export interface GeolocationPosition {
  latitude: number;
  longitude: number;
  accuracy: number;
  altitude: number | null;
  altitudeAccuracy: number | null;
  heading: number | null;
  speed: number | null;
  timestamp: number;
}

export interface GeolocationOptions {
  enableHighAccuracy?: boolean;
  timeout?: number;
  maximumAge?: number;
  watch?: boolean;
}

export interface GeolocationState {
  position: GeolocationPosition | null;
  error: GeolocationPositionError | null;
  isLoading: boolean;
  permission: PermissionState | null;
}

export interface GeolocationPositionError {
  code: number;
  message: string;
  PERMISSION_DENIED: number;
  POSITION_UNAVAILABLE: number;
  TIMEOUT: number;
}

export function useGeolocation(options: GeolocationOptions = {}) {
  const {
    enableHighAccuracy = true,
    timeout = 10000,
    maximumAge = 0,
    watch = false,
  } = options;
  
  const [state, setState] = useState<GeolocationState>({
    position: null,
    error: null,
    isLoading: false,
    permission: null,
  });
  
  const [watchId, setWatchId] = useState<number | null>(null);
  
  // Check permission
  const checkPermission = useCallback(async () => {
    if (!navigator?.permissions?.query) {
      setState(prev => ({ ...prev, permission: 'prompt' as PermissionState }));
      return;
    }
    
    try {
      const result = await navigator.permissions.query({ name: 'geolocation' as PermissionName });
      setState(prev => ({ ...prev, permission: result.state }));
      
      result.onchange = () => {
        setState(prev => ({ ...prev, permission: result.state }));
      };
    } catch (error) {
      console.warn('Geolocation permission API not supported:', error);
      setState(prev => ({ ...prev, permission: 'prompt' as PermissionState }));
    }
  }, []);
  
  // Get current position
  const getPosition = useCallback(() => {
    if (!navigator.geolocation) {
      const error: GeolocationPositionError = {
        code: 2,
        message: 'Geolocation is not supported by this browser',
        PERMISSION_DENIED: 1,
        POSITION_UNAVAILABLE: 2,
        TIMEOUT: 3,
      };
      setState(prev => ({ ...prev, error, isLoading: false }));
      return Promise.reject(error);
    }
    
    setState(prev => ({ ...prev, isLoading: true, error: null }));
    
    return new Promise<GeolocationPosition>((resolve, reject) => {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          const pos: GeolocationPosition = {
            latitude: position.coords.latitude,
            longitude: position.coords.longitude,
            accuracy: position.coords.accuracy,
            altitude: position.coords.altitude,
            altitudeAccuracy: position.coords.altitudeAccuracy,
            heading: position.coords.heading,
            speed: position.coords.speed,
            timestamp: position.timestamp,
          };
          
          setState(prev => ({
            ...prev,
            position: pos,
            isLoading: false,
            error: null,
          }));
          
          resolve(pos);
        },
        (error) => {
          const positionError: GeolocationPositionError = {
            code: error.code,
            message: error.message,
            PERMISSION_DENIED: 1,
            POSITION_UNAVAILABLE: 2,
            TIMEOUT: 3,
          };
          
          setState(prev => ({
            ...prev,
            error: positionError,
            isLoading: false,
          }));
          
          reject(positionError);
        },
        { enableHighAccuracy, timeout, maximumAge }
      );
    });
  }, [enableHighAccuracy, timeout, maximumAge]);
  
  // Watch position
  const startWatching = useCallback(() => {
    if (!navigator.geolocation) {
      console.warn('Geolocation not supported');
      return;
    }
    
    if (watchId !== null) {
      console.warn('Already watching position');
      return;
    }
    
    setState(prev => ({ ...prev, isLoading: true }));
    
    const id = navigator.geolocation.watchPosition(
      (position) => {
        const pos: GeolocationPosition = {
          latitude: position.coords.latitude,
          longitude: position.coords.longitude,
          accuracy: position.coords.accuracy,
          altitude: position.coords.altitude,
          altitudeAccuracy: position.coords.altitudeAccuracy,
          heading: position.coords.heading,
          speed: position.coords.speed,
          timestamp: position.timestamp,
        };
        
        setState(prev => ({
          ...prev,
          position: pos,
          isLoading: false,
          error: null,
        }));
      },
      (error) => {
        const positionError: GeolocationPositionError = {
          code: error.code,
          message: error.message,
          PERMISSION_DENIED: 1,
          POSITION_UNAVAILABLE: 2,
          TIMEOUT: 3,
        };
        
        setState(prev => ({
          ...prev,
          error: positionError,
          isLoading: false,
        }));
      },
      { enableHighAccuracy, timeout, maximumAge }
    );
    
    setWatchId(id);
  }, [enableHighAccuracy, timeout, maximumAge, watchId]);
  
  // Stop watching
  const stopWatching = useCallback(() => {
    if (watchId !== null) {
      navigator.geolocation.clearWatch(watchId);
      setWatchId(null);
    }
  }, [watchId]);
  
  // Request permission (indirect, via getPosition)
  const requestPermission = useCallback(async () => {
    try {
      const position = await getPosition();
      return { granted: true, position };
    } catch (error: any) {
      return { granted: false, error };
    }
  }, [getPosition]);
  
  // Initialize
  useEffect(() => {
    checkPermission();
    
    if (watch) {
      startWatching();
    } else {
      getPosition();
    }
    
    return () => {
      if (watchId !== null) {
        stopWatching();
      }
    };
  }, [watch]);
  
  return {
    ...state,
    getPosition,
    startWatching,
    stopWatching,
    requestPermission,
    isWatching: watchId !== null,
  };
}

§ 4.2 GEOLOCATION COMPONENT

typescript
// components/maps/GeolocationButton.tsx
'use client';

import { useState } from 'react';
import { useGeolocation } from '@/hooks/use-geolocation';
import { Map } from './Map';
import { MapMarker } from './MapMarker';
import { cn } from '@/lib/utils';
import { Target, Loader2, MapPin, AlertCircle } from 'lucide-react';

interface GeolocationButtonProps {
  onLocationFound?: (position: { latitude: number; longitude: number }) => void;
  onError?: (error: any) => void;
  className?: string;
  showMap?: boolean;
  mapHeight?: string | number;
}

export function GeolocationButton({
  onLocationFound,
  onError,
  className,
  showMap = false,
  mapHeight = 300,
}: GeolocationButtonProps) {
  const {
    position,
    error,
    isLoading,
    permission,
    getPosition,
    requestPermission,
  } = useGeolocation();
  
  const [showDetails, setShowDetails] = useState(false);
  
  const handleClick = async () => {
    try {
      const result = await requestPermission();
      if (result.granted && result.position) {
        onLocationFound?.({
          latitude: result.position.latitude,
          longitude: result.position.longitude,
        });
      } else {
        onError?.(result.error);
      }
    } catch (error) {
      onError?.(error);
    }
  };
  
  const getStatusText = () => {
    if (isLoading) return 'Detecting location...';
    if (error) {
      switch (error.code) {
        case 1: return 'Location permission denied';
        case 2: return 'Location unavailable';
        case 3: return 'Location request timed out';
        default: return 'Location error';
      }
    }
    if (position) return 'Location found';
    
    switch (permission) {
      case 'granted': return 'Get my location';
      case 'denied': return 'Location blocked';
      case 'prompt': return 'Share location';
      default: return 'Get location';
    }
  };
  
  const getIcon = () => {
    if (isLoading) return <Loader2 className="h-5 w-5 animate-spin" />;
    if (error) return <AlertCircle className="h-5 w-5 text-red-500" />;
    if (position) return <MapPin className="h-5 w-5 text-green-500" />;
    return <Target className="h-5 w-5" />;
  };
  
  const isDisabled = permission === 'denied' || isLoading;
  
  return (
    <div className={cn('space-y-4', className)}>
      <button
        onClick={handleClick}
        disabled={isDisabled}
        className={cn(
          'flex items-center justify-center gap-3 rounded-lg px-4 py-3 text-sm font-medium transition-colors',
          'bg-blue-600 text-white hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed',
          'shadow-sm hover:shadow-md'
        )}
      >
        {getIcon()}
        <span>{getStatusText()}</span>
      </button>
      
      {error && (
        <div className="rounded-lg bg-red-50 p-3 text-sm text-red-800">
          <div className="flex items-start gap-2">
            <AlertCircle className="mt-0.5 h-4 w-4 flex-shrink-0" />
            <div>
              <p className="font-medium">Unable to get location</p>
              <p className="mt-1 text-red-700">{error.message}</p>
              {error.code === 1 && (
                <p className="mt-1 text-sm">
                  Please enable location permissions in your browser settings.
                </p>
              )}
            </div>
          </div>
        </div>
      )}
      
      {position && (
        <div className="space-y-3">
          <button
            onClick={() => setShowDetails(!showDetails)}
            className="text-sm text-blue-600 hover:text-blue-800"
          >
            {showDetails ? 'Hide details' : 'Show details'}
          </button>
          
          {showDetails && (
            <div className="rounded-lg bg-gray-50 p-4 text-sm">
              <div className="grid grid-cols-2 gap-4">
                <div>
                  <div className="font-medium text-gray-700">Latitude</div>
                  <div className="text-gray-900">{position.latitude.toFixed(6)}</div>
                </div>
                <div>
                  <div className="font-medium text-gray-700">Longitude</div>
                  <div className="text-gray-900">{position.longitude.toFixed(6)}</div>
                </div>
                <div>
                  <div className="font-medium text-gray-700">Accuracy</div>
                  <div className="text-gray-900">{Math.round(position.accuracy)} meters</div>
                </div>
                <div>
                  <div className="font-medium text-gray-700">Altitude</div>
                  <div className="text-gray-900">
                    {position.altitude ? `${position.altitude.toFixed(1)}m` : 'N/A'}
                  </div>
                </div>
              </div>
            </div>
          )}
          
          {showMap && position && (
            <div className="mt-4 overflow-hidden rounded-lg border border-gray-200">
              <Map
                initialViewport={{
                  latitude: position.latitude,
                  longitude: position.longitude,
                  zoom: 15,
                }}
                height={mapHeight}
                showNavigation
              >
                <MapMarker
                  latitude={position.latitude}
                  longitude={position.longitude}
                  color="#ef4444"
                  popup={
                    <div className="p-2">
                      <div className="font-medium">Your Location</div>
                      <div className="text-sm text-gray-600">
                        Accuracy: {Math.round(position.accuracy)}m
                      </div>
                    </div>
                  }
                />
              </Map>
            </div>
          )}
        </div>
      )}
    </div>
  );
}

---

§ §5. ADDRESS & GEOCODING

§ 5.1 ADDRESS AUTOCOMPLETE (GOOGLE PLACES)

typescript
// lib/maps/geocoding.ts
import { z } from 'zod';

export interface GeocodingResult {
  placeId: string;
  formattedAddress: string;
  street?: string;
  city?: string;
  state?: string;
  postalCode?: string;
  country?: string;
  latitude: number;
  longitude: number;
  viewport: {
    northeast: { lat: number; lng: number };
    southwest: { lat: number; lng: number };
  };
  types: string[];
}

export interface AutocompletePrediction {
  placeId: string;
  description: string;
  structuredFormatting?: {
    mainText: string;
    secondaryText: string;
  };
  types: string[];
}

// Validation schemas
const GeocodingResultSchema = z.object({
  placeId: z.string(),
  formattedAddress: z.string(),
  street: z.string().optional(),
  city: z.string().optional(),
  state: z.string().optional(),
  postalCode: z.string().optional(),
  country: z.string().optional(),
  latitude: z.number(),
  longitude: z.number(),
  viewport: z.object({
    northeast: z.object({ lat: z.number(), lng: z.number() }),
    southwest: z.object({ lat: z.number(), lng: z.number() }),
  }),
  types: z.array(z.string()),
});

const AutocompletePredictionSchema = z.object({
  placeId: z.string(),
  description: z.string(),
  structuredFormatting: z.object({
    mainText: z.string(),
    secondaryText: z.string(),
  }).optional(),
  types: z.array(z.string()),
});

// Google Places API implementation
export class GoogleGeocodingService {
  private apiKey: string;
  private sessionToken: string;
  
  constructor(apiKey: string) {
    this.apiKey = apiKey;
    this.sessionToken = this.generateSessionToken();
  }
  
  private generateSessionToken(): string {
    return Math.random().toString(36).substring(2) + Date.now().toString(36);
  }
  
  private async makeRequest<T>(
    endpoint: string,
    params: Record<string, string>
  ): Promise<T> {
    const url = new URL(`https://maps.googleapis.com/maps/api/${endpoint}`);
    
    Object.entries(params).forEach(([key, value]) => {
      url.searchParams.append(key, value);
    });
    
    url.searchParams.append('key', this.apiKey);
    
    const response = await fetch(url.toString());
    
    if (!response.ok) {
      throw new Error(`Google API error: ${response.status}`);
    }
    
    const data = await response.json();
    
    if (data.status !== 'OK' && data.status !== 'ZERO_RESULTS') {
      throw new Error(`Google API error: ${data.status} - ${data.error_message || ''}`);
    }
    
    return data as T;
  }
  
  async autocomplete(
    input: string,
    options?: {
      types?: string[];
      location?: { lat: number; lng: number };
      radius?: number;
      language?: string;
      components?: Record<string, string[]>;
    }
  ): Promise<AutocompletePrediction[]> {
    const params: Record<string, string> = {
      input,
      sessiontoken: this.sessionToken,
    };
    
    if (options?.types) {
      params.types = options.types.join('|');
    }
    
    if (options?.location) {
      params.location = `${options.location.lat},${options.location.lng}`;
    }
    
    if (options?.radius) {
      params.radius = options.radius.toString();
    }
    
    if (options?.language) {
      params.language = options.language;
    }
    
    if (options?.components) {
      params.components = Object.entries(options.components)
        .map(([key, values]) => `${key}:${values.join('|')}`)
        .join('|');
    }
    
    const data = await this.makeRequest<{
      predictions: any[];
      status: string;
    }>('place/autocomplete/json', params);
    
    return data.predictions.map(prediction => {
      const result: AutocompletePrediction = {
        placeId: prediction.place_id,
        description: prediction.description,
        types: prediction.types || [],
      };
      
      if (prediction.structured_formatting) {
        result.structuredFormatting = {
          mainText: prediction.structured_formatting.main_text,
          secondaryText: prediction.structured_formatting.secondary_text,
        };
      }
      
      return AutocompletePredictionSchema.parse(result);
    });
  }
  
  async geocode(address: string): Promise<GeocodingResult[]> {
    const data = await this.makeRequest<{
      results: any[];
      status: string;
    }>('geocode/json', {
      address,
      language: 'en',
    });
    
    return data.results.map(result => {
      const addressComponents = result.address_components || [];
      
      const getComponent = (types: string[]) => {
        const component = addressComponents.find((comp: any) =>
          types.some(type => comp.types.includes(type))
        );
        return component?.long_name;
      };
      
      const geocodingResult: GeocodingResult = {
        placeId: result.place_id,
        formattedAddress: result.formatted_address,
        street: getComponent(['street_address', 'route']),
        city: getComponent(['locality', 'sublocality']),
        state: getComponent(['administrative_area_level_1']),
        postalCode: getComponent(['postal_code']),
        country: getComponent(['country']),
        latitude: result.geometry.location.lat,
        longitude: result.geometry.location.lng,
        viewport: {
          northeast: result.geometry.viewport.northeast,
          southwest: result.geometry.viewport.southwest,
        },
        types: result.types || [],
      };
      
      return GeocodingResultSchema.parse(geocodingResult);
    });
  }
  
  async reverseGeocode(
    latitude: number,
    longitude: number
  ): Promise<GeocodingResult[]> {
    const data = await this.makeRequest<{
      results: any[];
      status: string;
    }>('geocode/json', {
      latlng: `${latitude},${longitude}`,
      language: 'en',
    });
    
    return data.results.map(result => {
      const addressComponents = result.address_components || [];
      
      const getComponent = (types: string[]) => {
        const component = addressComponents.find((comp: any) =>
          types.some(type => comp.types.includes(type))
        );
        return component?.long_name;
      };
      
      return GeocodingResultSchema.parse({
        placeId: result.place_id,
        formattedAddress: result.formatted_address,
        street: getComponent(['street_address', 'route']),
        city: getComponent(['locality', 'sublocality']),
        state: getComponent(['administrative_area_level_1']),
        postalCode: getComponent(['postal_code']),
        country: getComponent(['country']),
        latitude: result.geometry.location.lat,
        longitude: result.geometry.location.lng,
        viewport: {
          northeast: result.geometry.viewport.northeast,
          southwest: result.geometry.viewport.southwest,
        },
        types: result.types || [],
      });
    });
  }
  
  async getPlaceDetails(placeId: string): Promise<GeocodingResult> {
    const data = await this.makeRequest<{
      result: any;
      status: string;
    }>('place/details/json', {
      place_id: placeId,
      fields: 'address_components,formatted_address,geometry,place_id,types',
      sessiontoken: this.sessionToken,
    });
    
    const result = data.result;
    const addressComponents = result.address_components || [];
    
    const getComponent = (types: string[]) => {
      const component = addressComponents.find((comp: any) =>
        types.some(type => comp.types.includes(type))
      );
      return component?.long_name;
    };
    
    return GeocodingResultSchema.parse({
      placeId: result.place_id,
      formattedAddress: result.formatted_address,
      street: getComponent(['street_address', 'route']),
      city: getComponent(['locality', 'sublocality']),
      state: getComponent(['administrative_area_level_1']),
      postalCode: getComponent(['postal_code']),
      country: getComponent(['country']),
      latitude: result.geometry.location.lat,
      longitude: result.geometry.location.lng,
      viewport: {
        northeast: result.geometry.viewport.northeast,
        southwest: result.geometry.viewport.southwest,
      },
      types: result.types || [],
    });
  }
}

// Mapbox implementation (alternativa a Google)
export class MapboxGeocodingService {
  private apiKey: string;
  
  constructor(apiKey: string) {
    this.apiKey = apiKey;
  }
  
  async forwardGeocode(
    query: string,
    options?: {
      types?: string[];
      proximity?: { longitude: number; latitude: number };
      language?: string;
      limit?: number;
    }
  ): Promise<GeocodingResult[]> {
    const url = new URL('https://api.mapbox.com/geocoding/v5/mapbox.places/' + encodeURIComponent(query) + '.json');
    
    url.searchParams.append('access_token', this.apiKey);
    
    if (options?.types) {
      url.searchParams.append('types', options.types.join(','));
    }
    
    if (options?.proximity) {
      url.searchParams.append('proximity', `${options.proximity.longitude},${options.proximity.latitude}`);
    }
    
    if (options?.language) {
      url.searchParams.append('language', options.language);
    }
    
    if (options?.limit) {
      url.searchParams.append('limit', options.limit.toString());
    }
    
    const response = await fetch(url.toString());
    
    if (!response.ok) {
      throw new Error(`Mapbox API error: ${response.status}`);
    }
    
    const data = await response.json();
    
    return data.features.map((feature: any) => {
      const context = feature.context || [];
      
      const getContext = (id: string) => {
        const item = context.find((c: any) => c.id.startsWith(id));
        return item?.text;
      };
      
      return GeocodingResultSchema.parse({
        placeId: feature.id,
        formattedAddress: feature.place_name,
        street: feature.text,
        city: getContext('place') || getContext('locality'),
        state: getContext('region'),
        postalCode: getContext('postcode'),
        country: getContext('country'),
        latitude: feature.center[1],
        longitude: feature.center[0],
        viewport: {
          northeast: {
            lat: feature.bbox?.[3] || feature.center[1] + 0.01,
            lng: feature.bbox?.[2] || feature.center[0] + 0.01,
          },
          southwest: {
            lat: feature.bbox?.[1] || feature.center[1] - 0.01,
            lng: feature.bbox?.[0] || feature.center[0] - 0.01,
          },
        },
        types: feature.place_type || [],
      });
    });
  }
  
  async reverseGeocode(
    longitude: number,
    latitude: number,
    options?: {
      types?: string[];
      limit?: number;
    }
  ): Promise<GeocodingResult[]> {
    const url = new URL(`https://api.mapbox.com/geocoding/v5/mapbox.places/${longitude},${latitude}.json`);
    
    url.searchParams.append('access_token', this.apiKey);
    
    if (options?.types) {
      url.searchParams.append('types', options.types.join(','));
    }
    
    if (options?.limit) {
      url.searchParams.append('limit', options.limit.toString());
    }
    
    const response = await fetch(url.toString());
    
    if (!response.ok) {
      throw new Error(`Mapbox API error: ${response.status}`);
    }
    
    const data = await response.json();
    
    return data.features.map((feature: any) => {
      const context = feature.context || [];
      
      const getContext = (id: string) => {
        const item = context.find((c: any) => c.id.startsWith(id));
        return item?.text;
      };
      
      return GeocodingResultSchema.parse({
        placeId: feature.id,
        formattedAddress: feature.place_name,
        street: feature.text,
        city: getContext('place') || getContext('locality'),
        state: getContext('region'),
        postalCode: getContext('postcode'),
        country: getContext('country'),
        latitude: feature.center[1],
        longitude: feature.center[0],
        viewport: {
          northeast: {
            lat: feature.center[1] + 0.01,
            lng: feature.center[0] + 0.01,
          },
          southwest: {
            lat: feature.center[1] - 0.01,
            lng: feature.center[0] - 0.01,
          },
        },
        types: feature.place_type || [],
      });
    });
  }
}

// Factory per scegliere il provider
export function createGeocodingService(provider: 'google' | 'mapbox') {
  switch (provider) {
    case 'google':
      const googleKey = process.env.NEXT_PUBLIC_GOOGLE_MAPS_API_KEY;
      if (!googleKey) {
        throw new Error('Google Maps API key not found');
      }
      return new GoogleGeocodingService(googleKey);
    
    case 'mapbox':
      const mapboxKey = process.env.NEXT_PUBLIC_MAPBOX_TOKEN;
      if (!mapboxKey) {
        throw new Error('Mapbox access token not found');
      }
      return new MapboxGeocodingService(mapboxKey);
    
    default:
      throw new Error(`Unknown geocoding provider: ${provider}`);
  }
}

---

**NOTA:** Questo è solo il §1-§5 (Overview, Mapbox Setup, Features, Geolocation, Geocoding) della documentazione completa. Continuerei con:

- §6: Location-based Features (Store Locator, Distance Calculation, Geofencing)
- §7: Map Search & Filtering
- §8: Maps Performance Optimization
- §9: Complete UI Components Library
- §10: Full Checklist

Il codice include:
- Setup Mapbox completo con TypeScript
- Componenti mappa modulari e riutilizzabili
- Geolocation hook con permission handling
- Geocoding service con Google e Mapbox
- Pattern per SSR e lazy loading
- Error handling completo
- Type safety per tutte le API