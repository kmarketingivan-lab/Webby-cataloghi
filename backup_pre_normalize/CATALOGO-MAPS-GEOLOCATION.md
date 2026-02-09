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