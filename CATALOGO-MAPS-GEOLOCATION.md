# CATALOGO-MAPS-GEOLOCATION

Catalogo Maps & Geolocation per Next.js 14 + TypeScript
§ MAPBOX GL JS
React Integration
typescript
// lib/mapbox/mapbox-config.ts
import mapboxgl from 'mapbox-gl';

export const MAPBOX_CONFIG = {
  accessToken: process.env.NEXT_PUBLIC_MAPBOX_TOKEN || '',
  style: 'mapbox://styles/mapbox/streets-v12',
  minZoom: 1,
  maxZoom: 22,
  center: [12.4964, 41.9028] as [number, number], // Roma
  zoom: 10
};

// Inizializza Mapbox
if (typeof window !== 'undefined') {
  mapboxgl.accessToken = MAPBOX_CONFIG.accessToken;
}
typescript
// components/mapbox/MapboxMap.tsx
'use client';

import { useEffect, useRef, useState } from 'react';
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';
import { MAPBOX_CONFIG } from '@/lib/mapbox/mapbox-config';

interface MapboxMapProps {
  center?: [number, number];
  zoom?: number;
  style?: string;
  interactive?: boolean;
  onLoad?: (map: mapboxgl.Map) => void;
}

export default function MapboxMap({
  center = MAPBOX_CONFIG.center,
  zoom = MAPBOX_CONFIG.zoom,
  style = MAPBOX_CONFIG.style,
  interactive = true,
  onLoad
}: MapboxMapProps) {
  const mapContainer = useRef<HTMLDivElement>(null);
  const map = useRef<mapboxgl.Map | null>(null);
  const [isLoaded, setIsLoaded] = useState(false);

  useEffect(() => {
    if (!mapContainer.current || map.current) return;

    map.current = new mapboxgl.Map({
      container: mapContainer.current,
      style,
      center,
      zoom,
      interactive,
      attributionControl: true
    });

    map.current.addControl(new mapboxgl.NavigationControl(), 'top-right');
    map.current.addControl(new mapboxgl.FullscreenControl(), 'top-right');
    map.current.addControl(new mapboxgl.ScaleControl(), 'bottom-left');

    map.current.on('load', () => {
      setIsLoaded(true);
      if (onLoad && map.current) {
        onLoad(map.current);
      }
    });

    return () => {
      if (map.current) {
        map.current.remove();
        map.current = null;
      }
    };
  }, []);

  useEffect(() => {
    if (map.current && isLoaded) {
      map.current.setCenter(center);
      map.current.setZoom(zoom);
    }
  }, [center, zoom, isLoaded]);

  return (
    <div className="relative w-full h-full">
      <div ref={mapContainer} className="absolute inset-0" />
    </div>
  );
}
Custom Markers
typescript
// components/mapbox/CustomMarker.tsx
'use client';

import { useEffect, useRef } from 'react';
import mapboxgl from 'mapbox-gl';
import { createRoot } from 'react-dom/client';

interface CustomMarkerProps {
  map: mapboxgl.Map;
  lngLat: [number, number];
  children: React.ReactNode;
  anchor?: 'center' | 'top' | 'bottom' | 'left' | 'right' | 'top-left' | 'top-right' | 'bottom-left' | 'bottom-right';
  onClick?: () => void;
  className?: string;
}

export function CustomMarker({
  map,
  lngLat,
  children,
  anchor = 'bottom',
  onClick,
  className = ''
}: CustomMarkerProps) {
  const markerRef = useRef<HTMLDivElement | null>(null);
  const markerInstance = useRef<mapboxgl.Marker | null>(null);

  useEffect(() => {
    if (!map) return;

    const el = document.createElement('div');
    el.className = `custom-marker ${className}`;
    
    if (onClick) {
      el.addEventListener('click', onClick);
      el.style.cursor = 'pointer';
    }

    // Render React component into marker element
    const root = createRoot(el);
    root.render(children);

    const marker = new mapboxgl.Marker({
      element: el,
      anchor
    })
      .setLngLat(lngLat)
      .addTo(map);

    markerRef.current = el;
    markerInstance.current = marker;

    return () => {
      if (markerInstance.current) {
        markerInstance.current.remove();
      }
      root.unmount();
    };
  }, [map, lngLat, children, anchor, onClick, className]);

  return null;
}

// Esempio di marker personalizzato
export function BusinessMarker({ name, color = '#FF6B6B' }: { name: string; color?: string }) {
  return (
    <div className="relative group">
      <div 
        className="w-10 h-10 rounded-full flex items-center justify-center shadow-lg transform transition-transform group-hover:scale-110"
        style={{ backgroundColor: color }}
      >
        <span className="text-white font-bold text-sm">🏪</span>
      </div>
      <div className="absolute bottom-full left-1/2 transform -translate-x-1/2 mb-2 hidden group-hover:block">
        <div className="bg-gray-900 text-white text-xs rounded py-1 px-2 whitespace-nowrap">
          {name}
        </div>
      </div>
    </div>
  );
}
Popups
typescript
// components/mapbox/CustomPopup.tsx
'use client';

import { useEffect, useRef } from 'react';
import mapboxgl from 'mapbox-gl';
import { createRoot } from 'react-dom/client';

interface CustomPopupProps {
  map: mapboxgl.Map;
  lngLat: [number, number];
  children: React.ReactNode;
  closeButton?: boolean;
  closeOnClick?: boolean;
  maxWidth?: string;
  anchor?: mapboxgl.Anchor;
  offset?: number | mapboxgl.PointLike;
  onClose?: () => void;
}

export function CustomPopup({
  map,
  lngLat,
  children,
  closeButton = true,
  closeOnClick = true,
  maxWidth = '240px',
  anchor = 'top',
  offset = 25,
  onClose
}: CustomPopupProps) {
  const popupRef = useRef<mapboxgl.Popup | null>(null);

  useEffect(() => {
    if (!map) return;

    const popup = new mapboxgl.Popup({
      closeButton,
      closeOnClick,
      maxWidth,
      anchor,
      offset
    })
      .setLngLat(lngLat)
      .setHTML('<div id="popup-content"></div>')
      .addTo(map);

    popupRef.current = popup;

    // Render React content into popup
    const contentContainer = popup.getElement().querySelector('#popup-content');
    if (contentContainer) {
      const root = createRoot(contentContainer);
      root.render(children);
    }

    if (onClose) {
      popup.on('close', onClose);
    }

    return () => {
      if (popupRef.current) {
        popupRef.current.remove();
      }
    };
  }, [map, lngLat, children, closeButton, closeOnClick, maxWidth, anchor, offset, onClose]);

  return null;
}

// Esempio di popup contenuto
export function BusinessPopup({ business }: { business: any }) {
  return (
    <div className="p-4 min-w-[200px]">
      <h3 className="font-bold text-lg mb-2">{business.name}</h3>
      <p className="text-gray-600 mb-2">{business.address}</p>
      <div className="flex items-center gap-2 mb-3">
        <span className="text-yellow-500">★</span>
        <span>{business.rating} ({business.reviewCount} recensioni)</span>
      </div>
      <button className="w-full bg-blue-600 text-white py-2 rounded hover:bg-blue-700 transition">
        Dettagli
      </button>
    </div>
  );
}
Clustering
typescript
// components/mapbox/ClusterMap.tsx
'use client';

import { useEffect, useRef, useState } from 'react';
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';
import { MAPBOX_CONFIG } from '@/lib/mapbox/mapbox-config';

interface ClusterMapProps {
  points: Array<{
    id: string;
    coordinates: [number, number];
    properties: Record<string, any>;
  }>;
  onClusterClick?: (features: mapboxgl.MapboxGeoJSONFeature[]) => void;
  onPointClick?: (point: any) => void;
}

export default function ClusterMap({ points, onClusterClick, onPointClick }: ClusterMapProps) {
  const mapContainer = useRef<HTMLDivElement>(null);
  const map = useRef<mapboxgl.Map | null>(null);
  const [mapLoaded, setMapLoaded] = useState(false);

  useEffect(() => {
    if (!mapContainer.current) return;

    map.current = new mapboxgl.Map({
      container: mapContainer.current,
      style: MAPBOX_CONFIG.style,
      center: MAPBOX_CONFIG.center,
      zoom: MAPBOX_CONFIG.zoom
    });

    map.current.on('load', () => {
      setMapLoaded(true);
      
      // Aggiungi source dati
      map.current!.addSource('points', {
        type: 'geojson',
        data: {
          type: 'FeatureCollection',
          features: points.map(point => ({
            type: 'Feature',
            geometry: {
              type: 'Point',
              coordinates: point.coordinates
            },
            properties: {
              id: point.id,
              ...point.properties
            }
          }))
        },
        cluster: true,
        clusterMaxZoom: 14,
        clusterRadius: 50
      });

      // Layer per cluster
      map.current!.addLayer({
        id: 'clusters',
        type: 'circle',
        source: 'points',
        filter: ['has', 'point_count'],
        paint: {
          'circle-color': [
            'step',
            ['get', 'point_count'],
            '#51bbd6',
            10,
            '#f1f075',
            30,
            '#f28cb1'
          ],
          'circle-radius': [
            'step',
            ['get', 'point_count'],
            20,
            10,
            30,
            30,
            40
          ]
        }
      });

      // Layer per conteggio cluster
      map.current!.addLayer({
        id: 'cluster-count',
        type: 'symbol',
        source: 'points',
        filter: ['has', 'point_count'],
        layout: {
          'text-field': ['get', 'point_count_abbreviated'],
          'text-font': ['DIN Offc Pro Medium', 'Arial Unicode MS Bold'],
          'text-size': 12
        }
      });

      // Layer per punti singoli
      map.current!.addLayer({
        id: 'unclustered-point',
        type: 'circle',
        source: 'points',
        filter: ['!', ['has', 'point_count']],
        paint: {
          'circle-color': '#11b4da',
          'circle-radius': 8,
          'circle-stroke-width': 2,
          'circle-stroke-color': '#fff'
        }
      });

      // Gestione click sui cluster
      map.current!.on('click', 'clusters', (e) => {
        const features = map.current!.queryRenderedFeatures(e.point, {
          layers: ['clusters']
        });
        
        if (features[0]) {
          const clusterId = features[0].properties!.cluster_id;
          const source = map.current!.getSource('points') as mapboxgl.GeoJSONSource;
          
          source.getClusterExpansionZoom(clusterId, (err, zoom) => {
            if (err || !map.current) return;

            const coordinates = (features[0].geometry as any).coordinates;
            
            map.current.easeTo({
              center: coordinates,
              zoom: zoom
            });

            if (onClusterClick) {
              source.getClusterLeaves(clusterId, 25, 0, (err, leaves) => {
                if (!err && leaves) {
                  onClusterClick(leaves);
                }
              });
            }
          });
        }
      });

      // Gestione click sui punti singoli
      map.current!.on('click', 'unclustered-point', (e) => {
        if (!e.features?.[0]) return;
        
        const feature = e.features[0];
        const coordinates = feature.geometry!.coordinates.slice() as [number, number];
        const properties = feature.properties;
        
        if (onPointClick) {
          onPointClick({
            coordinates,
            ...properties
          });
        }
      });

      // Cambia cursore
      map.current!.on('mouseenter', ['clusters', 'unclustered-point'], () => {
        map.current!.getCanvas().style.cursor = 'pointer';
      });

      map.current!.on('mouseleave', ['clusters', 'unclustered-point'], () => {
        map.current!.getCanvas().style.cursor = '';
      });
    });

    return () => {
      if (map.current) {
        map.current.remove();
      }
    };
  }, []);

  // Aggiorna dati quando cambiano i punti
  useEffect(() => {
    if (!mapLoaded || !map.current) return;

    const source = map.current.getSource('points') as mapboxgl.GeoJSONSource;
    if (source) {
      source.setData({
        type: 'FeatureCollection',
        features: points.map(point => ({
          type: 'Feature',
          geometry: {
            type: 'Point',
            coordinates: point.coordinates
          },
          properties: {
            id: point.id,
            ...point.properties
          }
        }))
      });
    }
  }, [points, mapLoaded]);

  return <div ref={mapContainer} className="w-full h-full" />;
}
Layers e Sources
typescript
// lib/mapbox/layers.ts
import mapboxgl from 'mapbox-gl';

export interface LayerConfig {
  id: string;
  type: 'fill' | 'line' | 'circle' | 'symbol' | 'raster' | 'heatmap';
  source: string | mapboxgl.GeoJSONSourceRaw;
  paint?: mapboxgl.AnyPaint;
  layout?: mapboxgl.AnyLayout;
  filter?: any[];
  minzoom?: number;
  maxzoom?: number;
}

export interface SourceConfig {
  id: string;
  type: 'vector' | 'raster' | 'geojson' | 'image' | 'video';
  url?: string;
  tiles?: string[];
  data?: any;
}

// Layer manager
export class MapboxLayerManager {
  private map: mapboxgl.Map;
  private layers: Map<string, LayerConfig> = new Map();
  private sources: Map<string, SourceConfig> = new Map();

  constructor(map: mapboxgl.Map) {
    this.map = map;
  }

  addSource(source: SourceConfig) {
    if (this.map.getSource(source.id)) {
      console.warn(`Source ${source.id} already exists`);
      return;
    }

    this.sources.set(source.id, source);
    
    switch (source.type) {
      case 'geojson':
        this.map.addSource(source.id, {
          type: 'geojson',
          data: source.data
        });
        break;
      case 'vector':
        this.map.addSource(source.id, {
          type: 'vector',
          url: source.url,
          tiles: source.tiles
        });
        break;
      case 'raster':
        this.map.addSource(source.id, {
          type: 'raster',
          url: source.url,
          tiles: source.tiles
        });
        break;
    }
  }

  addLayer(layer: LayerConfig) {
    if (this.map.getLayer(layer.id)) {
      console.warn(`Layer ${layer.id} already exists`);
      return;
    }

    this.layers.set(layer.id, layer);
    
    this.map.addLayer({
      id: layer.id,
      type: layer.type,
      source: typeof layer.source === 'string' ? layer.source : layer.source,
      paint: layer.paint || {},
      layout: layer.layout || {},
      filter: layer.filter,
      minzoom: layer.minzoom,
      maxzoom: layer.maxzoom
    });
  }

  removeLayer(layerId: string) {
    if (this.map.getLayer(layerId)) {
      this.map.removeLayer(layerId);
      this.layers.delete(layerId);
    }
  }

  removeSource(sourceId: string) {
    if (this.map.getSource(sourceId)) {
      // Rimuovi prima tutti i layer che usano questa source
      this.layers.forEach((layer, layerId) => {
        if (layer.source === sourceId) {
          this.removeLayer(layerId);
        }
      });

      this.map.removeSource(sourceId);
      this.sources.delete(sourceId);
    }
  }

  updateLayerVisibility(layerId: string, visible: boolean) {
    if (this.map.getLayer(layerId)) {
      this.map.setLayoutProperty(layerId, 'visibility', visible ? 'visible' : 'none');
    }
  }

  updateLayerFilter(layerId: string, filter: any[]) {
    if (this.map.getLayer(layerId)) {
      this.map.setFilter(layerId, filter);
    }
  }

  getLayer(layerId: string): LayerConfig | undefined {
    return this.layers.get(layerId);
  }

  getSource(sourceId: string): SourceConfig | undefined {
    return this.sources.get(sourceId);
  }
}

// Esempi di layer preconfigurati
export const PREDEFINED_LAYERS = {
  heatmap: (id: string, source: string): LayerConfig => ({
    id,
    type: 'heatmap',
    source,
    paint: {
      'heatmap-weight': ['interpolate', ['linear'], ['get', 'intensity'], 0, 0, 6, 1],
      'heatmap-intensity': ['interpolate', ['linear'], ['zoom'], 0, 1, 9, 3],
      'heatmap-color': [
        'interpolate',
        ['linear'],
        ['heatmap-density'],
        0, 'rgba(33,102,172,0)',
        0.2, 'rgb(103,169,207)',
        0.4, 'rgb(209,229,240)',
        0.6, 'rgb(253,219,199)',
        0.8, 'rgb(239,138,98)',
        1, 'rgb(178,24,43)'
      ],
      'heatmap-radius': ['interpolate', ['linear'], ['zoom'], 0, 2, 9, 20],
      'heatmap-opacity': ['interpolate', ['linear'], ['zoom'], 7, 1, 9, 0]
    }
  }),

  choropleth: (id: string, source: string, property: string): LayerConfig => ({
    id,
    type: 'fill',
    source,
    paint: {
      'fill-color': [
        'interpolate',
        ['linear'],
        ['get', property],
        0, '#f7fbff',
        10, '#deebf7',
        20, '#c6dbef',
        50, '#9ecae1',
        100, '#6baed6',
        200, '#4292c6',
        500, '#2171b5',
        1000, '#08519c',
        2000, '#08306b'
      ],
      'fill-opacity': 0.75,
      'fill-outline-color': '#fff'
    }
  })
};
3D Buildings
typescript
// components/mapbox/ThreeDBuildings.tsx
'use client';

import { useEffect } from 'react';
import mapboxgl from 'mapbox-gl';

interface ThreeDBuildingsProps {
  map: mapboxgl.Map;
  enabled: boolean;
}

export function ThreeDBuildings({ map, enabled }: ThreeDBuildingsProps) {
  useEffect(() => {
    if (!map || !enabled) return;

    // Aggiungi layer 3D buildings
    map.addLayer({
      id: '3d-buildings',
      source: 'composite',
      'source-layer': 'building',
      filter: ['==', 'extrude', 'true'],
      type: 'fill-extrusion',
      minzoom: 15,
      paint: {
        'fill-extrusion-color': [
          'interpolate',
          ['linear'],
          ['get', 'height'],
          0, 'hsl(0, 0%, 95%)',
          50, 'hsl(0, 0%, 86%)',
          100, 'hsl(0, 0%, 80%)'
        ],
        'fill-extrusion-height': [
          'interpolate',
          ['linear'],
          ['zoom'],
          15,
          0,
          15.05,
          ['get', 'height']
        ],
        'fill-extrusion-base': [
          'case',
          ['has', 'min_height'],
          ['get', 'min_height'],
          0
        ],
        'fill-extrusion-opacity': 0.6
      }
    });

    // Aggiungi terreno 3D
    map.addSource('mapbox-dem', {
      type: 'raster-dem',
      url: 'mapbox://mapbox.mapbox-terrain-dem-v1',
      tileSize: 512,
      maxzoom: 14
    });

    map.setTerrain({ source: 'mapbox-dem', exaggeration: 1.5 });

    // Abilita sky layer (opzionale)
    map.addLayer({
      id: 'sky',
      type: 'sky',
      paint: {
        'sky-type': 'atmosphere',
        'sky-atmosphere-sun': [0.0, 0.0],
        'sky-atmosphere-sun-intensity': 15
      }
    });

    return () => {
      if (map.getLayer('3d-buildings')) {
        map.removeLayer('3d-buildings');
      }
      if (map.getLayer('sky')) {
        map.removeLayer('sky');
      }
      if (map.getTerrain()) {
        map.setTerrain(null);
      }
    };
  }, [map, enabled]);

  return null;
}

// Controller per 3D buildings
export function ThreeDController({ map }: { map: mapboxgl.Map | null }) {
  const toggle3D = () => {
    if (!map) return;
    
    const enabled = map.getLayer('3d-buildings') !== undefined;
    
    if (enabled) {
      if (map.getLayer('3d-buildings')) map.removeLayer('3d-buildings');
      if (map.getLayer('sky')) map.removeLayer('sky');
      if (map.getTerrain()) map.setTerrain(null);
    } else {
      // Implementa l'aggiunta dei layer 3D
    }
  };

  const adjustPitch = (pitch: number) => {
    if (!map) return;
    map.easeTo({ pitch });
  };

  return (
    <div className="absolute top-4 right-4 bg-white rounded-lg shadow-lg p-3">
      <div className="space-y-2">
        <button
          onClick={toggle3D}
          className="w-full bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700"
        >
          Toggle 3D
        </button>
        <div className="space-y-1">
          <label className="block text-sm font-medium">Pitch:</label>
          <input
            type="range"
            min="0"
            max="60"
            defaultValue="0"
            onChange={(e) => adjustPitch(parseInt(e.target.value))}
            className="w-full"
          />
        </div>
      </div>
    </div>
  );
}
Directions API
typescript
// lib/mapbox/directions.ts
import mapboxgl from 'mapbox-gl';
import { MAPBOX_CONFIG } from './mapbox-config';

export interface RoutePoint {
  lngLat: [number, number];
  name?: string;
}

export interface RouteOptions {
  profile?: 'driving' | 'walking' | 'cycling' | 'driving-traffic';
  alternatives?: boolean;
  geometries?: 'geojson' | 'polyline' | 'polyline6';
  overview?: 'full' | 'simplified' | 'false';
  steps?: boolean;
  language?: string;
}

export class MapboxDirections {
  private map: mapboxgl.Map;
  private routeLayerId = 'route-layer';
  private routeSourceId = 'route-source';

  constructor(map: mapboxgl.Map) {
    this.map = map;
  }

  async calculateRoute(points: RoutePoint[], options: RouteOptions = {}) {
    const coordinates = points.map(p => p.lngLat.join(',')).join(';');
    const profile = options.profile || 'driving';
    
    const query = new URLSearchParams({
      geometries: options.geometries || 'geojson',
      overview: options.overview || 'simplified',
      steps: String(options.steps || false),
      alternatives: String(options.alternatives || false),
      language: options.language || 'it'
    });

    try {
      const response = await fetch(
        `https://api.mapbox.com/directions/v5/mapbox/${profile}/${coordinates}?${query}`,
        {
          headers: {
            'Authorization': `Bearer ${MAPBOX_CONFIG.accessToken}`
          }
        }
      );

      const data = await response.json();
      
      if (data.routes && data.routes.length > 0) {
        this.drawRoute(data.routes[0]);
        return data.routes[0];
      }
      
      return null;
    } catch (error) {
      console.error('Error calculating route:', error);
      return null;
    }
  }

  private drawRoute(route: any) {
    // Rimuovi layer esistente
    if (this.map.getLayer(this.routeLayerId)) {
      this.map.removeLayer(this.routeLayerId);
    }
    
    if (this.map.getSource(this.routeSourceId)) {
      this.map.removeSource(this.routeSourceId);
    }

    // Aggiungi source
    this.map.addSource(this.routeSourceId, {
      type: 'geojson',
      data: {
        type: 'Feature',
        properties: {},
        geometry: route.geometry
      }
    });

    // Aggiungi layer
    this.map.addLayer({
      id: this.routeLayerId,
      type: 'line',
      source: this.routeSourceId,
      layout: {
        'line-join': 'round',
        'line-cap': 'round'
      },
      paint: {
        'line-color': '#3b82f6',
        'line-width': 4,
        'line-opacity': 0.8
      }
    });

    // Aggiungi marker per i punti
    route.legs.forEach((leg: any, index: number) => {
      const steps = leg.steps;
      steps.forEach((step: any, stepIndex: number) => {
        if (stepIndex % 3 === 0) { // Ogni 3 step aggiungi un marker
          const coordinates = step.maneuver.location;
          
          new mapboxgl.Marker({
            color: '#3b82f6',
            scale: 0.7
          })
            .setLngLat(coordinates)
            .setPopup(new mapboxgl.Popup().setHTML(`<div>Step ${stepIndex}</div>`))
            .addTo(this.map);
        }
      });
    });
  }

  clearRoute() {
    if (this.map.getLayer(this.routeLayerId)) {
      this.map.removeLayer(this.routeLayerId);
    }
    
    if (this.map.getSource(this.routeSourceId)) {
      this.map.removeSource(this.routeSourceId);
    }
  }

  async getOptimizedRoute(points: RoutePoint[], options: RouteOptions = {}) {
    // Per route ottimizzate (TSP)
    const coordinates = points.map(p => p.lngLat.join(',')).join(';');
    
    try {
      const response = await fetch(
        `https://api.mapbox.com/optimized-trips/v1/mapbox/${options.profile || 'driving'}/${coordinates}?roundtrip=true&source=first&destination=last&geometries=geojson`,
        {
          headers: {
            'Authorization': `Bearer ${MAPBOX_CONFIG.accessToken}`
          }
        }
      );

      const data = await response.json();
      return data;
    } catch (error) {
      console.error('Error calculating optimized route:', error);
      return null;
    }
  }
}
§ GOOGLE MAPS
@react-google-maps/api
typescript
// lib/google-maps/google-maps-config.ts
export const GOOGLE_MAPS_CONFIG = {
  apiKey: process.env.NEXT_PUBLIC_GOOGLE_MAPS_API_KEY || '',
  libraries: ['places', 'geometry', 'visualization'] as (
    | 'places'
    | 'drawing'
    | 'geometry'
    | 'visualization'
    | 'localContext'
  )[],
  defaultCenter: { lat: 41.9028, lng: 12.4964 },
  defaultZoom: 12,
  mapId: process.env.NEXT_PUBLIC_GOOGLE_MAPS_MAP_ID
};
typescript
// components/google-maps/GoogleMap.tsx
'use client';

import { useState, useCallback, useRef } from 'react';
import { 
  GoogleMap as GoogleMapComponent,
  LoadScript,
  Marker,
  InfoWindow,
  Polyline,
  Polygon,
  Circle
} from '@react-google-maps/api';
import { GOOGLE_MAPS_CONFIG } from '@/lib/google-maps/google-maps-config';

const containerStyle = {
  width: '100%',
  height: '100%'
};

interface GoogleMapProps {
  center?: google.maps.LatLngLiteral;
  zoom?: number;
  markers?: Array<{
    position: google.maps.LatLngLiteral;
    title?: string;
    icon?: string;
    onClick?: () => void;
  }>;
  polylines?: Array<{
    path: google.maps.LatLngLiteral[];
    strokeColor?: string;
    strokeWeight?: number;
  }>;
  onLoad?: (map: google.maps.Map) => void;
  onUnmount?: (map: google.maps.Map) => void;
  onClick?: (e: google.maps.MapMouseEvent) => void;
}

export default function GoogleMap({
  center = GOOGLE_MAPS_CONFIG.defaultCenter,
  zoom = GOOGLE_MAPS_CONFIG.defaultZoom,
  markers = [],
  polylines = [],
  onLoad,
  onUnmount,
  onClick
}: GoogleMapProps) {
  const [selectedMarker, setSelectedMarker] = useState<number | null>(null);
  const mapRef = useRef<google.maps.Map | null>(null);

  const handleLoad = useCallback((map: google.maps.Map) => {
    mapRef.current = map;
    if (onLoad) onLoad(map);
  }, [onLoad]);

  const handleUnmount = useCallback((map: google.maps.Map) => {
    mapRef.current = null;
    if (onUnmount) onUnmount(map);
  }, [onUnmount]);

  return (
    <LoadScript
      googleMapsApiKey={GOOGLE_MAPS_CONFIG.apiKey}
      libraries={GOOGLE_MAPS_CONFIG.libraries}
      mapIds={GOOGLE_MAPS_CONFIG.mapId ? [GOOGLE_MAPS_CONFIG.mapId] : undefined}
    >
      <GoogleMapComponent
        mapContainerStyle={containerStyle}
        center={center}
        zoom={zoom}
        onLoad={handleLoad}
        onUnmount={handleUnmount}
        onClick={onClick}
        options={{
          mapId: GOOGLE_MAPS_CONFIG.mapId,
          disableDefaultUI: false,
          zoomControl: true,
          streetViewControl: true,
          mapTypeControl: true,
          fullscreenControl: true
        }}
      >
        {markers.map((marker, index) => (
          <Marker
            key={index}
            position={marker.position}
            title={marker.title}
            icon={marker.icon}
            onClick={() => {
              setSelectedMarker(index);
              if (marker.onClick) marker.onClick();
            }}
          />
        ))}

        {selectedMarker !== null && markers[selectedMarker] && (
          <InfoWindow
            position={markers[selectedMarker].position}
            onCloseClick={() => setSelectedMarker(null)}
          >
            <div>
              <h3 className="font-bold">{markers[selectedMarker].title}</h3>
              <p>Position: {markers[selectedMarker].position.lat}, {markers[selectedMarker].position.lng}</p>
            </div>
          </InfoWindow>
        )}

        {polylines.map((polyline, index) => (
          <Polyline
            key={index}
            path={polyline.path}
            options={{
              strokeColor: polyline.strokeColor || '#FF0000',
              strokeOpacity: 0.8,
              strokeWeight: polyline.strokeWeight || 2
            }}
          />
        ))}
      </GoogleMapComponent>
    </LoadScript>
  );
}
Custom Overlays
typescript
// components/google-maps/CustomOverlay.tsx
'use client';

import { useEffect, useRef } from 'react';
import { OverlayView } from '@react-google-maps/api';

interface CustomOverlayProps {
  position: google.maps.LatLngLiteral;
  mapPaneName?: 'floatPane' | 'mapPane' | 'markerLayer' | 'overlayLayer' | 'floatShadow';
  children: React.ReactNode;
  onLoad?: (overlay: google.maps.OverlayView) => void;
  onUnmount?: (overlay: google.maps.OverlayView) => void;
}

export function CustomOverlay({
  position,
  mapPaneName = 'overlayLayer',
  children,
  onLoad,
  onUnmount
}: CustomOverlayProps) {
  return (
    <OverlayView
      position={position}
      mapPaneName={mapPaneName}
      getPixelPositionOffset={(width, height) => ({
        x: -(width / 2),
        y: -(height / 2)
      })}
      onLoad={onLoad}
      onUnmount={onUnmount}
    >
      {children}
    </OverlayView>
  );
}

// Overlay personalizzato con HTML/CSS
export function HTMLOverlay({ 
  position, 
  content,
  className = ''
}: { 
  position: google.maps.LatLngLiteral;
  content: string;
  className?: string;
}) {
  return (
    <CustomOverlay position={position}>
      <div className={`bg-white rounded-lg shadow-lg p-4 ${className}`}>
        <div dangerouslySetInnerHTML={{ __html: content }} />
      </div>
    </CustomOverlay>
  );
}

// Custom SVG Marker
export function SVGCustomMarker({ 
  position,
  color = '#FF0000',
  size = 40
}: {
  position: google.maps.LatLngLiteral;
  color?: string;
  size?: number;
}) {
  return (
    <CustomOverlay position={position}>
      <div style={{ transform: 'translate(-50%, -100%)' }}>
        <svg 
          width={size} 
          height={size} 
          viewBox="0 0 24 24"
          fill={color}
        >
          <path d="M12 2C8.13 2 5 5.13 5 9c0 5.25 7 13 7 13s7-7.75 7-13c0-3.87-3.13-7-7-7zm0 9.5c-1.38 0-2.5-1.12-2.5-2.5s1.12-2.5 2.5-2.5 2.5 1.12 2.5 2.5-1.12 2.5-2.5 2.5z" />
        </svg>
      </div>
    </CustomOverlay>
  );
}
Street View
typescript
// components/google-maps/StreetView.tsx
'use client';

import { useState, useRef, useCallback } from 'react';
import { 
  StreetViewPanorama,
  StreetViewService,
  LoadScript 
} from '@react-google-maps/api';
import { GOOGLE_MAPS_CONFIG } from '@/lib/google-maps/google-maps-config';

interface StreetViewComponentProps {
  position:

════════════════════════════════════════════════════════════
FIGMA CATALOG: BATCH2-08-MAPS-GEOLOCATION
Prompt ID: 8 / 19
Parte: 2
Exported: 2026-02-06T17:32:10.947Z
Characters: 1014
════════════════════════════════════════════════════════════

 }),
  
  black: createCustomIcon({
    iconUrl: '/leaflet/images/marker-icon-black.png',
    shadowUrl: '/leaflet/images/marker-shadow.png'
  }),
  
  yellow: createCustomIcon({
    iconUrl: '/leaflet/images/marker-icon-yellow.png',
    shadowUrl: '/leaflet/images/marker-shadow.png'
  }),
  
  orange: createCustomIcon({
    iconUrl: '/leaflet/images/marker-icon-orange.png',
    shadowUrl: '/leaflet/images/marker-shadow.png'
  }),
  
  grey: createCustomIcon({
    iconUrl: '/leaflet/images/marker-icon-grey.png',
    shadowUrl: '/leaflet/images/marker-shadow.png'
  })
};

// Icone SVG personalizzate
export function createSVGIcon(options: {
  svgString?: string;
  iconSize?: [number, number];
  iconAnchor?: [number, number];
  popupAnchor?: [number, number];
  className?: string;
  color?: string;
  bgColor?: string;
}): L.DivIcon {
  const size = options.iconSize || [30, 30];
  const anchor = options.iconAnchor || [size[0] / 2, size[1]];
  
  const svg = options.svgString || `
    <svg xmlns="http://

## § ADVANCED PATTERNS: MAPS GEOLOCATION

### Server Actions con Validazione

```typescript
// app/actions/maps-geolocation.ts
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


### MAPS GEOLOCATION - Utility Helper #667

```typescript
// lib/utils/maps-geolocation-helper-667.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #849

```typescript
// lib/utils/maps-geolocation-helper-849.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #888

```typescript
// lib/utils/maps-geolocation-helper-888.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #503

```typescript
// lib/utils/maps-geolocation-helper-503.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #763

```typescript
// lib/utils/maps-geolocation-helper-763.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #875

```typescript
// lib/utils/maps-geolocation-helper-875.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #858

```typescript
// lib/utils/maps-geolocation-helper-858.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #79

```typescript
// lib/utils/maps-geolocation-helper-79.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #758

```typescript
// lib/utils/maps-geolocation-helper-758.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #988

```typescript
// lib/utils/maps-geolocation-helper-988.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #640

```typescript
// lib/utils/maps-geolocation-helper-640.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #472

```typescript
// lib/utils/maps-geolocation-helper-472.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #482

```typescript
// lib/utils/maps-geolocation-helper-482.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #478

```typescript
// lib/utils/maps-geolocation-helper-478.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #65

```typescript
// lib/utils/maps-geolocation-helper-65.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #831

```typescript
// lib/utils/maps-geolocation-helper-831.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MAPS GEOLOCATION - Utility Helper #820

```typescript
// lib/utils/maps-geolocation-helper-820.ts
import { z } from "zod";

interface MAPSGEOLOCATIONConfig {
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

export class MAPSGEOLOCATIONProcessor<TInput, TOutput> {
  private config: MAPSGEOLOCATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MAPSGEOLOCATIONConfig> = {}) {
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

  getConfig(): Readonly<MAPSGEOLOCATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```
