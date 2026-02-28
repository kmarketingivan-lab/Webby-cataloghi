# CATALOGO-UI-UX-DESIGN-SYSTEM-v2

CATALOGO-UI-UX-DESIGN-SYSTEM per Next.js 14
§ DESIGN SYSTEM FOUNDATIONS
Purpose and Benefits

Il Design System unifica l'esperienza utente e lo sviluppo attraverso componenti riutilizzabili, garantendo coerenza, accessibilità e manutenibilità.

Benefici principali:

Consistenza: Esperienza utente uniforme across prodotti

Efficienza: Riduzione del tempo di sviluppo del 40-60%

Accessibilità: Conformità WCAG 2.1 AA integrata

Scalabilità: Facile onboarding per nuovi sviluppatori

Quality Assurance: Test centralizzati e standardizzati

typescript
// design-system.config.ts
export const designSystemConfig = {
  name: 'Catalogo Design System',
  version: '1.0.0',
  foundation: {
    principles: ['Consistency', 'Accessibility', 'Efficiency', 'Clarity'],
    compliance: ['WCAG 2.1 AA', 'GDPR', 'Web Standards'],
    browserSupport: {
      evergreen: 'last 2 versions',
      ie: 'none'
    }
  }
};
Team Collaboration

Ruoli e responsabilità:

Design Lead: Definisce principi visivi e UX

System Architect: Struttura token e componenti

Frontend Lead: Implementa componenti core

Accessibility Specialist: Garantisce conformità WCAG

Product Manager: Allinea roadmap con bisogni prodotto

Workflow collaborativo:

text
Design → Tokenizzazione → Prototipo → Sviluppo → Documentazione → Release
Documentation Requirements

Documentazione obbligatoria per ogni componente:

Descrizione e use case

API completa con TypeScript

Varianti e stati

Accessibilità (ARIA, keyboard nav)

Comportamento responsive

Dos & Don'ts

Esempi d'uso reali

Versioning Strategy

Semantic Versioning (SemVer):

MAJOR: Breaking changes API

MINOR: Nuove features retrocompatibili

PATCH: Bug fixes e miglioramenti minori

Release Channels:

typescript
// release-channels.config.ts
export const releaseChannels = {
  alpha: { audience: 'internal team', stability: 'unstable' },
  beta: { audience: 'selected partners', stability: 'testing' },
  rc: { audience: 'wider team', stability: 'stable' },
  stable: { audience: 'all consumers', stability: 'production' }
};
§ DESIGN TOKENS ARCHITECTURE
Token Hierarchy

Architettura a 3 livelli:

typescript
// tokens/token-hierarchy.ts
export const tokenHierarchy = {
  // Livello 1: Token primitivi (valori assoluti)
  global: {
    color: {
      blue: {
        50: '#eff6ff',
        100: '#dbeafe',
        // ... fino a 900
      },
      spacing: {
        unit: 4, // 4px base unit
        1: '4px',
        2: '8px',
        // ... scale
      }
    }
  },
  
  // Livello 2: Token semantici (significato contestuale)
  semantic: {
    color: {
      background: {
        primary: '{color.blue.500}',
        danger: '{color.red.500}',
      },
      text: {
        primary: '{color.gray.900}',
        secondary: '{color.gray.600}',
      }
    }
  },
  
  // Livello 3: Token component-specific
  component: {
    button: {
      background: {
        primary: '{semantic.color.background.primary}',
        hover: '{semantic.color.background.primary-dark}',
      }
    }
  }
};
Token Naming Conventions

Sistema di naming basato su design-tokens.org:

typescript
// naming-conventions.ts
export const namingConventions = {
  structure: '{category}.{type}.{item}.{state}.{property}',
  examples: [
    'color.background.primary.hover', // Stato hover
    'size.font.heading.large', // Dimensione
    'spacing.padding.button.small', // Spacing specifico
    'border.radius.card.default' // Proprietà
  ],
  rules: [
    'Usare camelCase per tutti i token',
    'Seguire ordine gerarchico',
    'Evitare nomi basati su valori',
    'Priorità semantica su valori'
  ]
};
Token Storage Format

Formato JSON standardizzato con Style Dictionary:

json
// tokens/global.json
{
  "color": {
    "blue": {
      "50": { "value": "#eff6ff", "type": "color" },
      "100": { "value": "#dbeafe", "type": "color" }
    }
  },
  "spacing": {
    "unit": { "value": 4, "type": "spacing" },
    "1": { "value": "{spacing.unit}px", "type": "spacing" },
    "2": { "value": "{spacing.unit * 2}px", "type": "spacing" }
  },
  "font": {
    "family": {
      "sans": { 
        "value": ["Inter", "system-ui", "sans-serif"],
        "type": "fontFamily" 
      }
    }
  }
}

Configurazione Style Dictionary:

javascript
// style-dictionary.config.js
module.exports = {
  source: ['tokens/**/*.json'],
  platforms: {
    css: {
      transformGroup: 'css',
      buildPath: 'styles/tokens/',
      files: [{
        destination: '_variables.css',
        format: 'css/variables'
      }]
    },
    ts: {
      transformGroup: 'js',
      buildPath: 'tokens/generated/',
      files: [{
        destination: 'tokens.ts',
        format: 'typescript/es6-declarations'
      }]
    },
    ios: {
      // ... configurazione per iOS
    },
    android: {
      // ... configurazione per Android
    }
  }
};
Cross-Platform Tokens

Token system per multiple piattaforme:

typescript
// tokens/platform-specific.ts
export const platformTokens = {
  shared: {
    // Token comuni a tutte le piattaforme
    color: {
      brand: { value: '#0066cc' }
    }
  },
  web: {
    // Token specifici web
    shadow: {
      elevation: {
        low: { value: '0 1px 3px rgba(0,0,0,0.12)' }
      }
    }
  },
  mobile: {
    // Token specifici mobile
    shadow: {
      elevation: {
        low: { 
          ios: { value: '{shadow.color} 0px 0.5px 2px' },
          android: { value: '{shadow.color} 0px 1px 2px' }
        }
      }
    }
  }
};
§ COMPONENT LIBRARY STRUCTURE
Atomic Design Levels

Implementazione dei 5 livelli di Atomic Design:

text
src/components/
├── atoms/           # Componenti base indivisibili
│   ├── Button/
│   ├── Icon/
│   └── Text/
├── molecules/       # Combinazioni di atoms
│   ├── FormField/
│   ├── Card/
│   └── Alert/
├── organisms/       # Sezioni complesse
│   ├── Header/
│   ├── Footer/
│   └── Sidebar/
├── templates/       # Layout strutturati
│   ├── Dashboard/
│   ├── AuthLayout/
│   └── ProductPage/
└── pages/          # Pagine specifiche (Next.js)
    ├── HomePage/
    └── ProductDetail/
Component Categories

Categorizzazione per funzione:

typescript
// components/categories.ts
export const componentCategories = {
  layout: ['Grid', 'Container', 'Flex', 'Box', 'Stack'],
  navigation: ['Button', 'Link', 'Breadcrumb', 'Pagination', 'Tabs'],
  forms: ['Input', 'Select', 'Checkbox', 'Radio', 'Form', 'Label'],
  feedback: ['Alert', 'Toast', 'Spinner', 'Progress', 'Skeleton'],
  dataDisplay: ['Table', 'List', 'Card', 'Badge', 'Chip', 'Avatar'],
  overlay: ['Modal', 'Drawer', 'Popover', 'Tooltip', 'Dropdown'],
  typography: ['Heading', 'Text', 'Label', 'Code', 'Blockquote'],
  media: ['Image', 'Icon', 'Video', 'Figure']
};
Component API Design

Pattern standard per component props:

typescript
// components/button/types.ts
export interface BaseComponentProps {
  /** Identificatore unico per testing */
  'data-testid'?: string;
  /** Classi CSS aggiuntive */
  className?: string;
  /** Stili inline */
  style?: React.CSSProperties;
  /** Elemento DOM da utilizzare come root */
  as?: React.ElementType;
  /** Handler eventi */
  onClick?: React.MouseEventHandler;
  /** Accessibilità */
  'aria-label'?: string;
}

export interface ButtonProps extends BaseComponentProps {
  /** Tipo di bottone */
  variant?: 'primary' | 'secondary' | 'ghost' | 'danger';
  /** Dimensione */
  size?: 'sm' | 'md' | 'lg';
  /** Stato di caricamento */
  loading?: boolean;
  /** Bottone disabilitato */
  disabled?: boolean;
  /** Icona prima del testo */
  startIcon?: React.ReactNode;
  /** Icona dopo il testo */
  endIcon?: React.ReactNode;
  /** Contenuto del bottone */
  children: React.ReactNode;
  /** Tipo HTML button */
  type?: 'button' | 'submit' | 'reset';
  /** URL per link button */
  href?: string;
}

// Pattern di default props
export const defaultButtonProps: Partial<ButtonProps> = {
  variant: 'primary',
  size: 'md',
  type: 'button',
  disabled: false,
  loading: false
};
Props Standardization

Props comuni a tutti i componenti:

typescript
// components/shared/types.ts
export type SpacingProps = {
  /** Margin top */
  mt?: SpacingToken;
  /** Margin bottom */
  mb?: SpacingToken;
  /** Padding vertical */
  py?: SpacingToken;
  // ... tutte le direzioni
};

export type LayoutProps = {
  /** Width */
  w?: string | number;
  /** Height */
  h?: string | number;
  /** Display property */
  display?: 'block' | 'inline' | 'flex' | 'grid';
  /** Flexbox alignment */
  alignItems?: 'flex-start' | 'center' | 'flex-end';
  /** Grid template columns */
  gridTemplateColumns?: string;
};

export type ResponsiveProps<T> = {
  /** Valore per mobile (<768px) */
  mobile?: T;
  /** Valore per tablet (768px-1024px) */
  tablet?: T;
  /** Valore per desktop (>1024px) */
  desktop?: T;
};

// Utility type per combinare tutti i props standard
export type StandardComponentProps = 
  BaseComponentProps & 
  SpacingProps & 
  LayoutProps & 
  Partial<Record<keyof ResponsiveProps<any>, any>>;
§ TYPOGRAPHY SYSTEM
Font Stack

Sistema gerarchico di font:

typescript
// typography/font-stack.ts
export const fontStacks = {
  sans: [
    'Inter',
    '-apple-system',
    'BlinkMacSystemFont',
    'Segoe UI',
    'Roboto',
    'Helvetica Neue',
    'Arial',
    'sans-serif'
  ],
  serif: [
    'Georgia',
    'Times New Roman',
    'serif'
  ],
  mono: [
    'SFMono-Regular',
    'Menlo',
    'Monaco',
    'Consolas',
    'Liberation Mono',
    'Courier New',
    'monospace'
  ],
  display: [
    'Inter Display',
    ...fontStacks.sans
  ]
};

// Configurazione Next.js Font Optimization
import { Inter, Roboto_Mono } from 'next/font/google';

export const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
  fallback: ['system-ui', 'sans-serif']
});

export const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-roboto-mono',
  fallback: ['monospace']
});
Type Scale

Scala tipografica modulare (1.25 ratio):

typescript
// typography/type-scale.ts
export const typeScale = {
  // Scala per display text
  display: {
    '2xl': {
      fontSize: '4.5rem',      // 72px
      lineHeight: 1.1,
      letterSpacing: '-0.02em',
      fontWeight: 700
    },
    xl: {
      fontSize: '3.75rem',     // 60px
      lineHeight: 1.1,
      letterSpacing: '-0.02em',
      fontWeight: 700
    },
    lg: {
      fontSize: '3rem',        // 48px
      lineHeight: 1.2,
      letterSpacing: '-0.01em',
      fontWeight: 700
    },
    md: {
      fontSize: '2.25rem',     // 36px
      lineHeight: 1.2,
      letterSpacing: '-0.01em',
      fontWeight: 600
    },
    sm: {
      fontSize: '1.875rem',    // 30px
      lineHeight: 1.2,
      letterSpacing: '-0.01em',
      fontWeight: 600
    },
    xs: {
      fontSize: '1.5rem',      // 24px
      lineHeight: 1.3,
      letterSpacing: '0em',
      fontWeight: 600
    }
  },
  
  // Scala per body text
  text: {
    xl: {
      fontSize: '1.25rem',     // 20px
      lineHeight: 1.5,
      letterSpacing: '0em',
      fontWeight: 400
    },
    lg: {
      fontSize: '1.125rem',    // 18px
      lineHeight: 1.5,
      letterSpacing: '0em',
      fontWeight: 400
    },
    md: {
      fontSize: '1rem',        // 16px
      lineHeight: 1.5,
      letterSpacing: '0em',
      fontWeight: 400
    },
    sm: {
      fontSize: '0.875rem',    // 14px
      lineHeight: 1.5,
      letterSpacing: '0.01em',
      fontWeight: 400
    },
    xs: {
      fontSize: '0.75rem',     // 12px
      lineHeight: 1.5,
      letterSpacing: '0.02em',
      fontWeight: 400
    }
  }
};
Line Heights

Sistema di line-height basato su unità relative:

typescript
// typography/line-heights.ts
export const lineHeights = {
  // Valori relativi (senza unità)
  tight: 1.1,      // Per titoli
  snug: 1.25,     // Per sottotitoli
  normal: 1.5,    // Per body text
  relaxed: 1.75,  // Per testo lungo
  loose: 2,       // Per poesia o citazioni
  
  // Mapping per componenti
  heading: {
    h1: 'tight',
    h2: 'snug',
    h3: 'normal'
  },
  body: {
    paragraph: 'relaxed',
    label: 'normal',
    caption: 'tight'
  }
};
Letter Spacing

Tracking values per leggibilità:

typescript
// typography/letter-spacing.ts
export const letterSpacing = {
  tighter: '-0.05em',
  tight: '-0.025em',
  normal: '0em',
  wide: '0.025em',
  wider: '0.05em',
  widest: '0.1em',
  
  // Applicazioni specifiche
  applications: {
    uppercase: 'wide',      // Per testo in maiuscolo
    smallCaps: 'tight',     // Per small caps
    displayHeadings: 'tighter', // Per display text
    code: 'normal'          // Per codice
  }
};
Font Weights

Gerarchia dei pesi font:

typescript
// typography/font-weights.ts
export const fontWeights = {
  thin: 100,
  extralight: 200,
  light: 300,
  normal: 400,
  medium: 500,
  semibold: 600,
  bold: 700,
  extrabold: 800,
  black: 900,
  
  // Semantic mapping
  semantic: {
    display: 'bold',
    heading: 'semibold',
    subheading: 'medium',
    body: 'normal',
    label: 'medium',
    caption: 'normal',
    strong: 'bold',
    emphasis: 'semibold'
  }
};
Responsive Typography

Sistema tipografico responsive:

typescript
// typography/responsive.ts
export const responsiveTypography = {
  // Breakpoints
  breakpoints: {
    mobile: '320px',
    tablet: '768px',
    desktop: '1024px',
    wide: '1440px'
  },
  
  // Fluid typography scaling
  fluidScale: (minSize: number, maxSize: number) => {
    const minViewport = 320;
    const maxViewport = 1440;
    
    return `
      font-size: ${minSize}px;
      @media (min-width: ${minViewport}px) {
        font-size: calc(${minSize}px + (${maxSize} - ${minSize}) * 
          ((100vw - ${minViewport}px) / (${maxViewport} - ${minViewport})));
      }
      @media (min-width: ${maxViewport}px) {
        font-size: ${maxSize}px;
      }
    `;
  },
  
  // Predefined fluid scales
  scales: {
    display: {
      '2xl': fluidScale(48, 72),
      xl: fluidScale(36, 60),
      lg: fluidScale(32, 48)
    },
    text: {
      xl: fluidScale(16, 20),
      lg: fluidScale(14, 18),
      md: fluidScale(13, 16)
    }
  }
};

// Implementazione componente con Typography
// components/atoms/Text/Text.tsx
import React from 'react';
import { typeScale, fontWeights, lineHeights } from '../../../typography';

interface TextProps {
  variant?: keyof typeof typeScale.text;
  as?: 'p' | 'span' | 'div';
  weight?: keyof typeof fontWeights;
  lineHeight?: keyof typeof lineHeights;
  children: React.ReactNode;
}

export const Text: React.FC<TextProps> = ({
  variant = 'md',
  as: Component = 'p',
  weight = 'normal',
  lineHeight = 'normal',
  children,
  ...props
}) => {
  const styles = {
    ...typeScale.text[variant],
    fontWeight: fontWeights[weight],
    lineHeight: lineHeights[lineHeight]
  };

  return (
    <Component style={styles} {...props}>
      {children}
    </Component>
  );
};
§ COLOR SYSTEM
Color Palette Generation

Sistema di generazione palette basato su HSL:

typescript
// colors/palette-generator.ts
export interface ColorStop {
  name: string;
  value: string;
  contrastColor: string;
}

export const generateColorPalette = (
  baseColor: string,
  steps: number = 10
): ColorStop[] => {
  const baseHsl = hexToHsl(baseColor);
  const palette: ColorStop[] = [];
  
  for (let i = 0; i < steps; i++) {
    // Calcola luminosità in percentuale
    const lightness = 100 - (i * (100 / steps));
    const color = `hsl(${baseHsl.h}, ${baseHsl.s}%, ${lightness}%)`;
    
    palette.push({
      name: `${i * 100}`,
      value: hslToHex(color),
      contrastColor: lightness > 50 ? '#000000' : '#FFFFFF'
    });
  }
  
  return palette;
};

// Palette completa
export const colorPalette = {
  // Primary colors
  primary: generateColorPalette('#0066CC'),
  
  // Neutral colors
  neutral: {
    0: '#FFFFFF',
    50: '#F9FAFB',
    100: '#F3F4F6',
    200: '#E5E7EB',
    300: '#D1D5DB',
    400: '#9CA3AF',
    500: '#6B7280',
    600: '#4B5563',
    700: '#374151',
    800: '#1F2937',
    900: '#111827',
    950: '#030712'
  },
  
  // Semantic colors
  success: generateColorPalette('#10B981'),
  warning: generateColorPalette('#F59E0B'),
  error: generateColorPalette('#EF4444'),
  info: generateColorPalette('#3B82F6'),
  
  // Extended palette
  extended: {
    purple: generateColorPalette('#8B5CF6'),
    pink: generateColorPalette('#EC4899'),
    orange: generateColorPalette('#F97316'),
    teal: generateColorPalette('#14B8A6')
  }
};
Semantic Colors

Sistema di colori semantici:

typescript
// colors/semantic-colors.ts
export const semanticColors = {
  // Background colors
  background: {
    primary: 'var(--color-neutral-0)',
    secondary: 'var(--color-neutral-50)',
    tertiary: 'var(--color-neutral-100)',
    inverse: 'var(--color-neutral-900)'
  },
  
  // Text colors
  text: {
    primary: 'var(--color-neutral-900)',
    secondary: 'var(--color-neutral-700)',
    tertiary: 'var(--color-neutral-500)',
    disabled: 'var(--color-neutral-400)',
    inverse: 'var(--color-neutral-0)',
    link: 'var(--color-primary-600)',
    linkHover: 'var(--color-primary-700)',
    
    // Semantic text colors
    success: 'var(--color-success-700)',
    warning: 'var(--color-warning-700)',
    error: 'var(--color-error-700)',
    info: 'var(--color-info-700)'
  },
  
  // Border colors
  border: {
    default: 'var(--color-neutral-200)',
    strong: 'var(--color-neutral-300)',
    focus: 'var(--color-primary-500)',
    error: 'var(--color-error-500)',
    success: 'var(--color-success-500)'
  },
  
  // Interactive states
  interactive: {
    primary: {
      default: 'var(--color-primary-600)',
      hover: 'var(--color-primary-700)',
      active: 'var(--color-primary-800)',
      disabled: 'var(--color-neutral-300)'
    },
    secondary: {
      default: 'var(--color-neutral-200)',
      hover: 'var(--color-neutral-300)',
      active: 'var(--color-neutral-400)',
      disabled: 'var(--color-neutral-100)'
    }
  },
  
  // Status colors
  status: {
    success: {
      background: 'var(--color-success-50)',
      text: 'var(--color-success-700)',
      border: 'var(--color-success-200)',
      icon: 'var(--color-success-500)'
    },
    warning: {
      background: 'var(--color-warning-50)',
      text: 'var(--color-warning-700)',
      border: 'var(--color-warning-200)',
      icon: 'var(--color-warning-500)'
    },
    error: {
      background: 'var(--color-error-50)',
      text: 'var(--color-error-700)',
      border: 'var(--color-error-200)',
      icon: 'var(--color-error-500)'
    },
    info: {
      background: 'var(--color-info-50)',
      text: 'var(--color-info-700)',
      border: 'var(--color-info-200)',
      icon: 'var(--color-info-500)'
    }
  }
};
Dark Mode Colors

Sistema di colori per dark mode:

typescript
// colors/dark-mode.ts
export const darkModeColors = {
  // Invertiamo la scala per dark mode
  background: {
    primary: 'var(--color-neutral-950)',
    secondary: 'var(--color-neutral-900)',
    tertiary: 'var(--color-neutral-800)',
    inverse: 'var(--color-neutral-0)'
  },
  
  text: {
    primary: 'var(--color-neutral-50)',
    secondary: 'var(--color-neutral-300)',
    tertiary: 'var(--color-neutral-500)',
    disabled: 'var(--color-neutral-600)',
    inverse: 'var(--color-neutral-950)',
    link: 'var(--color-primary-400)',
    linkHover: 'var(--color-primary-300)',
    
    success: 'var(--color-success-400)',
    warning: 'var(--color-warning-400)',
    error: 'var(--color-error-400)',
    info: 'var(--color-info-400)'
  },
  
  border: {
    default: 'var(--color-neutral-800)',
    strong: 'var(--color-neutral-700)',
    focus: 'var(--color-primary-500)',
    error: 'var(--color-error-500)',
    success: 'var(--color-success-500)'
  },
  
  interactive: {
    primary: {
      default: 'var(--color-primary-500)',
      hover: 'var(--color-primary-400)',
      active: 'var(--color-primary-300)',
      disabled: 'var(--color-neutral-700)'
    },
    secondary: {
      default: 'var(--color-neutral-800)',
      hover: 'var(--color-neutral-700)',
      active: 'var(--color-neutral-600)',
      disabled: 'var(--color-neutral-800)'
    }
  }
};

// Hook per gestione dark mode
// hooks/useDarkMode.ts
import { useEffect, useState } from 'react';

export const useDarkMode = () => {
  const [isDarkMode, setIsDarkMode] = useState(false);
  
  useEffect(() => {
    // Controlla system preference
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    setIsDarkMode(mediaQuery.matches);
    
    const handler = (e: MediaQueryListEvent) => setIsDarkMode(e.matches);
    mediaQuery.addEventListener('change', handler);
    
    return () => mediaQuery.removeEventListener('change', handler);
  }, []);
  
  return { isDarkMode, setIsDarkMode };
};
Accessible Color Combinations

Sistema per garantire accessibilità:

typescript
// colors/accessibility.ts
export interface ColorContrast {
  normalText: boolean;  // AA 4.5:1 per testo normale
  largeText: boolean;   // AA 3:1 per testo grande
  uiComponents: boolean; // AA 3:1 per componenti UI
}

export const checkContrast = (
  foreground: string,
  background: string
): ColorContrast => {
  const contrast = calculateContrastRatio(foreground, background);
  
  return {
    normalText: contrast >= 4.5,
    largeText: contrast >= 3,
    uiComponents: contrast >= 3
  };
};

export const accessibleColorPairs = {
  // Coppie pre-testate per accessibilità
  primary: [
    { foreground: 'neutral-900', background: 'neutral-0', rating: 'AAA' },
    { foreground: 'primary-600', background: 'neutral-0', rating: 'AA' },
    { foreground: 'neutral-0', background: 'primary-600', rating: 'AAA' }
  ],
  
  // Sistema di fallback automatico
  getAccessiblePair: (foregroundToken: string, backgroundToken: string) => {
    const pair = accessibleColorPairs.primary.find(
      p => p.foreground === foregroundToken && p.background === backgroundToken
    );
    
    if (!pair) {
      // Trova alternativa accessibile
      return {
        foreground: 'neutral-900',
        background: 'neutral-0',
        rating: 'AAA',
        originalPair: { foregroundToken, backgroundToken }
      };
    }
    
    return pair;
  }
};

// Utility per calcolo contrast ratio
const calculateContrastRatio = (color1: string, color2: string): number => {
  const lum1 = calculateRelativeLuminance(color1);
  const lum2 = calculateRelativeLuminance(color2);
  
  const brightest = Math.max(lum1, lum2);
  const darkest = Math.min(lum1, lum2);
  
  return (brightest + 0.05) / (darkest + 0.05);
};
Color in Context

Applicazione contestuale dei colori:

typescript
// colors/contextual-colors.ts
export const contextualColors = {
  // Data visualization
  dataViz: {
    categorical: [
      'primary-500',
      'success-500',
      'warning-500',
      'error-500',
      'info-500',
      'purple-500'
    ],
    sequential: {
      blue: ['primary-100', 'primary-300', 'primary-500', 'primary-700', 'primary-900'],
      green: ['success-100', 'success-300', 'success-500', 'success-700', 'success-900']
    },
    diverging: {
      redBlue: [
        'error-700', 'error-500', 'error-300',
        'neutral-100',
        'primary-300', 'primary-500', 'primary-700'
      ]
    }
  },
  
  // User interface states
  uiStates: {
    hover: {
      lighten: 0.1,  // Lighten by 10%
      darken: 0.1    // Darken by 10% for dark mode
    },
    active: {
      lighten: 0.2,
      darken: 0.2
    },
    disabled: {
      opacity: 0.5,
      saturation: 0.5
    }
  },
  
  // Component-specific colors
  components: {
    button: {
      variants: {
        primary: {
          background: 'interactive.primary.default',
          text: 'neutral-0',
          border: 'transparent',
          hover: 'interactive.primary.hover'
        },
        secondary: {
          background: 'transparent',
          text: 'text.primary',
          border: 'border.default',
          hover: 'background.tertiary'
        }
      }
    },
    card: {
      background: 'background.secondary',
      border: 'border.default',
      hover: 'background.tertiary'
    }
  }
};
§ SPACING SYSTEM
Base Unit

Unità base 4px (0.25rem) per consistenza:

typescript
// spacing/base-unit.ts
export const BASE_UNIT = 4; // 4px
export const BASE_REM = 0.25; // 0.25rem (assumendo 16px base font-size)

export const spacingSystem = {
  // Convertitore unità
  px: (value: number) => `${value * BASE_UNIT}px`,
  rem: (value: number) => `${value * BASE_REM}rem`,
  em: (value: number) => `${value * BASE_REM}em`,
  
  // Valori comuni
  common: {
    none: 0,
    auto: 'auto',
    full: '100%',
    screen: '100vw',
    min: 'min-content',
    max: 'max-content'
  }
};
Spacing Scale

Scala spaziale 8-point con progressione geometrica:

typescript
// spacing/spacing-scale.ts
export const spacingScale = {
  // Scale lineare per valori piccoli (0-8)
  0: '0px',
  0.5: '2px',   // 0.5 * 4px
  1: '4px',     // 1 * 4px
  1.5: '6px',   // 1.5 * 4px
  2: '8px',     // 2 * 4px
  2.5: '10px',  // 2.5 * 4px
  3: '12px',    // 3 * 4px
  3.5: '14px',  // 3.5 * 4px
  4: '16px',    // 4 * 4px
  
  // Scala geometrica per valori maggiori
  5: '20px',    // 5 * 4px
  6: '24px',    // 6 * 4px
  7: '28px',
  8: '32px',
  9: '36px',
  10: '40px',
  11: '44px',
  12: '48px',
  14: '56px',
  16: '64px',
  20: '80px',
  24: '96px',
  28: '112px',
  32: '128px',
  36: '144px',
  40: '160px',
  44: '176px',
  48: '192px',
  52: '208px',
  56: '224px',
  60: '240px',
  64: '256px',
  72: '288px',
  80: '320px',
  96: '384px',
  
  // Relative units
  '1/2': '50%',
  '1/3': '33.333%',
  '2/3': '66.666%',
  '1/4': '25%',
  '2/4': '50%',
  '3/4': '75%',
  '1/5': '20%',
  '2/5': '40%',
  '3/5': '60%',
  '4/5': '80%',
  '1/6': '16.666%',
  '2/6': '33.333%',
  '3/6': '50%',
  '4/6': '66.666%',
  '5/6': '83.333%',
  '1/12': '8.333%',
  '2/12': '16.666%',
  '3/12': '25%',
  '4/12': '33.333%',
  '5/12': '41.666%',
  '6/12': '50%',
  '7/12': '58.333%',
  '8/12': '66.666%',
  '9/12': '75%',
  '10/12': '83.333%',
  '11/12': '91.666%'
};
Component Spacing

Spacing specifico per componenti:

typescript
// spacing/component-spacing.ts
export const componentSpacing = {
  // Button spacing
  button: {
    padding: {
      sm: { x: 'spacing-3', y: 'spacing-1.5' },
      md: { x: 'spacing-4', y: 'spacing-2' },
      lg: { x: 'spacing-5', y: 'spacing

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-36-UI-UX-DESIGN-SYSTEM
Prompt ID: 36 / 48
Parte: 2
Exported: 2026-02-06T13:29:05.758Z
Characters: 1089
════════════════════════════════════════════════════════════

,
      borderColor: 'var(--color-primary-500)',
      color: 'var(--color-primary-700)'
    },
    indeterminate: {
      background: 'var(--color-neutral-100)',
      borderColor: 'var(--color-neutral-400)'
    }
  },
  
  // Stati di visibilità
  visibility: {
    hidden: {
      opacity: 0,
      visibility: 'hidden',
      pointerEvents: 'none'
    },
    visible: {
      opacity: 1,
      visibility: 'visible',
      pointerEvents: 'auto'
    },
    collapsed: {
      height: 0,
      overflow: 'hidden',
      opacity: 0
    },
    expanded: {
      height: 'auto',
      opacity: 1
    }
  },
  
  // Utility per combinare stati
  combineStates: (...states: string[]) => {
    const combined: Record<string, any> = {};
    
    states.forEach(state => {
      const stateConfig = (stateVariants as any)[state];
      if (stateConfig) {
        Object.assign(combined, stateConfig);
      }
    });
    
    return combined;
  }
};

// CSS Animations per stati
export const stateAnimations = {
  '@keyframes spin': {
    from: { transform: 'rotate(0deg)' },
    to: { transform


---

## DESIGN TOKEN SYSTEM — IMPLEMENTAZIONE COMPLETA

### Panoramica
Sistema di design token completo con CSS custom properties, configurazione Tailwind,
color scales, typography, spacing, shadows e animazioni.

### Implementazione Completa — Tailwind Config

```typescript
// tailwind.config.ts — Design Token System

import type { Config } from "tailwindcss";
import { fontFamily } from "tailwindcss/defaultTheme";

const config: Config = {
  darkMode: ["class"],
  content: [
    "./pages/**/*.{ts,tsx}",
    "./components/**/*.{ts,tsx}",
    "./app/**/*.{ts,tsx}",
    "./src/**/*.{ts,tsx}",
  ],
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: {
        "2xl": "1400px",
      },
    },
    extend: {
      // ── Color Tokens ──
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
          50: "hsl(var(--primary-50))",
          100: "hsl(var(--primary-100))",
          200: "hsl(var(--primary-200))",
          300: "hsl(var(--primary-300))",
          400: "hsl(var(--primary-400))",
          500: "hsl(var(--primary-500))",
          600: "hsl(var(--primary-600))",
          700: "hsl(var(--primary-700))",
          800: "hsl(var(--primary-800))",
          900: "hsl(var(--primary-900))",
          950: "hsl(var(--primary-950))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
        success: {
          DEFAULT: "hsl(var(--success))",
          foreground: "hsl(var(--success-foreground))",
        },
        warning: {
          DEFAULT: "hsl(var(--warning))",
          foreground: "hsl(var(--warning-foreground))",
        },
        info: {
          DEFAULT: "hsl(var(--info))",
          foreground: "hsl(var(--info-foreground))",
        },
        sidebar: {
          DEFAULT: "hsl(var(--sidebar-background))",
          foreground: "hsl(var(--sidebar-foreground))",
          primary: "hsl(var(--sidebar-primary))",
          "primary-foreground": "hsl(var(--sidebar-primary-foreground))",
          accent: "hsl(var(--sidebar-accent))",
          "accent-foreground": "hsl(var(--sidebar-accent-foreground))",
          border: "hsl(var(--sidebar-border))",
          ring: "hsl(var(--sidebar-ring))",
        },
      },

      // ── Typography Tokens ──
      fontFamily: {
        sans: ["var(--font-sans)", ...fontFamily.sans],
        heading: ["var(--font-heading)", ...fontFamily.sans],
        mono: ["var(--font-mono)", ...fontFamily.mono],
      },
      fontSize: {
        "2xs": ["0.625rem", { lineHeight: "0.875rem" }],
        xs: ["0.75rem", { lineHeight: "1rem" }],
        sm: ["0.875rem", { lineHeight: "1.25rem" }],
        base: ["1rem", { lineHeight: "1.5rem" }],
        lg: ["1.125rem", { lineHeight: "1.75rem" }],
        xl: ["1.25rem", { lineHeight: "1.75rem" }],
        "2xl": ["1.5rem", { lineHeight: "2rem" }],
        "3xl": ["1.875rem", { lineHeight: "2.25rem" }],
        "4xl": ["2.25rem", { lineHeight: "2.5rem" }],
        "5xl": ["3rem", { lineHeight: "1.15" }],
        "6xl": ["3.75rem", { lineHeight: "1.1" }],
        "7xl": ["4.5rem", { lineHeight: "1.05" }],
      },
      letterSpacing: {
        tighter: "-0.04em",
        tight: "-0.02em",
        normal: "0em",
        wide: "0.02em",
        wider: "0.04em",
        widest: "0.08em",
      },

      // ── Spacing Tokens (4px grid) ──
      spacing: {
        "4.5": "1.125rem",
        "5.5": "1.375rem",
        "13": "3.25rem",
        "15": "3.75rem",
        "18": "4.5rem",
        "22": "5.5rem",
        "26": "6.5rem",
        "30": "7.5rem",
      },

      // ── Border Radius Tokens ──
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
        xl: "calc(var(--radius) + 4px)",
        "2xl": "calc(var(--radius) + 8px)",
      },

      // ── Shadow Tokens ──
      boxShadow: {
        "elevation-1": "0 1px 2px 0 rgb(0 0 0 / 0.05)",
        "elevation-2": "0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1)",
        "elevation-3": "0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1)",
        "elevation-4": "0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)",
        "elevation-5": "0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)",
        "inner-sm": "inset 0 1px 2px 0 rgb(0 0 0 / 0.05)",
        "inner-md": "inset 0 2px 4px 0 rgb(0 0 0 / 0.1)",
        glow: "0 0 15px -3px var(--tw-shadow-color)",
        "glow-lg": "0 0 30px -5px var(--tw-shadow-color)",
      },

      // ── Animation Tokens ──
      keyframes: {
        "accordion-down": {
          from: { height: "0" },
          to: { height: "var(--radix-accordion-content-height)" },
        },
        "accordion-up": {
          from: { height: "var(--radix-accordion-content-height)" },
          to: { height: "0" },
        },
        "fade-in": {
          from: { opacity: "0" },
          to: { opacity: "1" },
        },
        "fade-out": {
          from: { opacity: "1" },
          to: { opacity: "0" },
        },
        "slide-in-from-top": {
          from: { transform: "translateY(-100%)" },
          to: { transform: "translateY(0)" },
        },
        "slide-in-from-bottom": {
          from: { transform: "translateY(100%)" },
          to: { transform: "translateY(0)" },
        },
        "slide-in-from-left": {
          from: { transform: "translateX(-100%)" },
          to: { transform: "translateX(0)" },
        },
        "slide-in-from-right": {
          from: { transform: "translateX(100%)" },
          to: { transform: "translateX(0)" },
        },
        "scale-in": {
          from: { transform: "scale(0.95)", opacity: "0" },
          to: { transform: "scale(1)", opacity: "1" },
        },
        "spin-slow": {
          from: { transform: "rotate(0deg)" },
          to: { transform: "rotate(360deg)" },
        },
        shimmer: {
          "0%": { backgroundPosition: "-200% 0" },
          "100%": { backgroundPosition: "200% 0" },
        },
        pulse: {
          "0%, 100%": { opacity: "1" },
          "50%": { opacity: "0.5" },
        },
        bounce: {
          "0%, 100%": { transform: "translateY(0)" },
          "50%": { transform: "translateY(-10%)" },
        },
      },
      animation: {
        "accordion-down": "accordion-down 0.2s ease-out",
        "accordion-up": "accordion-up 0.2s ease-out",
        "fade-in": "fade-in 0.2s ease-out",
        "fade-out": "fade-out 0.2s ease-out",
        "slide-in-top": "slide-in-from-top 0.3s ease-out",
        "slide-in-bottom": "slide-in-from-bottom 0.3s ease-out",
        "slide-in-left": "slide-in-from-left 0.3s ease-out",
        "slide-in-right": "slide-in-from-right 0.3s ease-out",
        "scale-in": "scale-in 0.2s ease-out",
        "spin-slow": "spin-slow 3s linear infinite",
        shimmer: "shimmer 2s linear infinite",
        "pulse-slow": "pulse 3s ease-in-out infinite",
        "bounce-gentle": "bounce 2s ease-in-out infinite",
      },

      // ── Transition Tokens ──
      transitionDuration: {
        "150": "150ms",
        "200": "200ms",
        "250": "250ms",
        "300": "300ms",
        "400": "400ms",
        "500": "500ms",
      },
      transitionTimingFunction: {
        "ease-spring": "cubic-bezier(0.16, 1, 0.3, 1)",
        "ease-bounce": "cubic-bezier(0.34, 1.56, 0.64, 1)",
        "ease-smooth": "cubic-bezier(0.4, 0, 0.2, 1)",
      },
    },
  },
  plugins: [
    require("tailwindcss-animate"),
    require("@tailwindcss/typography"),
    require("@tailwindcss/container-queries"),
  ],
};

export default config;
```

### CSS Custom Properties — globals.css

```css
/* app/globals.css — CSS Token Variables */

@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    /* ── Color Tokens: Light Mode ── */
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 243 75% 59%;
    --primary-foreground: 210 40% 98%;
    --primary-50: 245 100% 97%;
    --primary-100: 244 100% 94%;
    --primary-200: 243 97% 88%;
    --primary-300: 243 95% 80%;
    --primary-400: 243 90% 70%;
    --primary-500: 243 75% 59%;
    --primary-600: 243 75% 49%;
    --primary-700: 243 72% 41%;
    --primary-800: 243 68% 33%;
    --primary-900: 243 60% 27%;
    --primary-950: 243 55% 17%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --success: 142 76% 36%;
    --success-foreground: 0 0% 100%;
    --warning: 38 92% 50%;
    --warning-foreground: 0 0% 0%;
    --info: 199 89% 48%;
    --info-foreground: 0 0% 100%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 243 75% 59%;
    --radius: 0.5rem;

    /* ── Sidebar ── */
    --sidebar-background: 0 0% 98%;
    --sidebar-foreground: 240 5.3% 26.1%;
    --sidebar-primary: 243 75% 59%;
    --sidebar-primary-foreground: 0 0% 100%;
    --sidebar-accent: 240 4.8% 95.9%;
    --sidebar-accent-foreground: 240 5.9% 10%;
    --sidebar-border: 220 13% 91%;
    --sidebar-ring: 243 75% 59%;

    /* ── Font Tokens ── */
    --font-sans: "Inter", system-ui, -apple-system, sans-serif;
    --font-heading: "Cal Sans", "Inter", sans-serif;
    --font-mono: "JetBrains Mono", "Fira Code", monospace;

    /* ── Z-Index Scale ── */
    --z-dropdown: 50;
    --z-sticky: 100;
    --z-overlay: 200;
    --z-modal: 300;
    --z-popover: 400;
    --z-tooltip: 500;
    --z-toast: 600;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;
    --primary: 243 75% 65%;
    --primary-foreground: 222.2 84% 4.9%;
    --primary-50: 243 30% 12%;
    --primary-100: 243 35% 16%;
    --primary-200: 243 40% 22%;
    --primary-300: 243 50% 32%;
    --primary-400: 243 60% 45%;
    --primary-500: 243 75% 59%;
    --primary-600: 243 75% 65%;
    --primary-700: 243 80% 72%;
    --primary-800: 243 85% 80%;
    --primary-900: 243 90% 88%;
    --primary-950: 243 95% 94%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 50.6%;
    --destructive-foreground: 210 40% 98%;
    --success: 142 71% 45%;
    --success-foreground: 0 0% 100%;
    --warning: 38 92% 60%;
    --warning-foreground: 0 0% 0%;
    --info: 199 89% 58%;
    --info-foreground: 0 0% 100%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 243 75% 65%;
    --sidebar-background: 222 47% 6%;
    --sidebar-foreground: 210 40% 90%;
    --sidebar-primary: 243 75% 65%;
    --sidebar-primary-foreground: 0 0% 100%;
    --sidebar-accent: 222 40% 12%;
    --sidebar-accent-foreground: 210 40% 90%;
    --sidebar-border: 222 30% 14%;
    --sidebar-ring: 243 75% 65%;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
    font-feature-settings: "rlig" 1, "calt" 1;
  }
}

/* ── Scrollbar styling ── */
@layer utilities {
  .scrollbar-thin {
    scrollbar-width: thin;
    scrollbar-color: hsl(var(--muted)) transparent;
  }
  .scrollbar-thin::-webkit-scrollbar {
    width: 6px;
    height: 6px;
  }
  .scrollbar-thin::-webkit-scrollbar-track {
    background: transparent;
  }
  .scrollbar-thin::-webkit-scrollbar-thumb {
    background-color: hsl(var(--muted-foreground) / 0.3);
    border-radius: 3px;
  }

  .text-balance {
    text-wrap: balance;
  }
}
```

### Theme Provider e Dark Mode

```typescript
// components/theme/ThemeProvider.tsx — Next-themes provider

"use client";

import * as React from "react";
import { ThemeProvider as NextThemesProvider } from "next-themes";
import { type ThemeProviderProps } from "next-themes";

export function ThemeProvider({ children, ...props }: ThemeProviderProps) {
  return <NextThemesProvider {...props}>{children}</NextThemesProvider>;
}

// app/layout.tsx usage:
// <ThemeProvider
//   attribute="class"
//   defaultTheme="system"
//   enableSystem
//   disableTransitionOnChange
// >
//   {children}
// </ThemeProvider>
```

```typescript
// components/theme/ThemeToggle.tsx — Dark mode toggle

"use client";

import { useTheme } from "next-themes";
import { Button } from "@/components/ui/button";
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu";
import { Moon, Sun, Monitor } from "lucide-react";

export function ThemeToggle() {
  const { setTheme, theme } = useTheme();

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="ghost" size="icon" className="relative">
          <Sun className="h-5 w-5 rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
          <Moon className="absolute h-5 w-5 rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
          <span className="sr-only">Cambia tema</span>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end">
        <DropdownMenuItem onClick={() => setTheme("light")}>
          <Sun className="mr-2 h-4 w-4" />
          Chiaro
          {theme === "light" && <span className="ml-auto text-xs">Attivo</span>}
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => setTheme("dark")}>
          <Moon className="mr-2 h-4 w-4" />
          Scuro
          {theme === "dark" && <span className="ml-auto text-xs">Attivo</span>}
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => setTheme("system")}>
          <Monitor className="mr-2 h-4 w-4" />
          Sistema
          {theme === "system" && <span className="ml-auto text-xs">Attivo</span>}
        </DropdownMenuItem>
      </DropdownMenuContent>
    </DropdownMenu>
  );
}
```

### Font Loading con next/font

```typescript
// app/layout.tsx — Font loading

import { Inter } from "next/font/google";
import localFont from "next/font/local";

const inter = Inter({
  subsets: ["latin"],
  variable: "--font-sans",
  display: "swap",
});

const calSans = localFont({
  src: "../public/fonts/CalSans-SemiBold.woff2",
  variable: "--font-heading",
  display: "swap",
});

const jetBrainsMono = localFont({
  src: "../public/fonts/JetBrainsMono-Regular.woff2",
  variable: "--font-mono",
  display: "swap",
});

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="it" suppressHydrationWarning>
      <body
        className={`${inter.variable} ${calSans.variable} ${jetBrainsMono.variable} font-sans antialiased`}
      >
        {children}
      </body>
    </html>
  );
}
```

## COMPONENT VARIANTS CON CVA

### Panoramica
Pattern per varianti di componenti con class-variance-authority (CVA),
Tailwind merge e slot composition per Button, Input, Badge, Card e Alert.

### Implementazione Completa

```typescript
// lib/variants.ts — Centralized component variants

import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

// ── Button Variants ──
export const buttonVariants = cva(
  "inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        outline: "border border-input bg-background hover:bg-accent hover:text-accent-foreground",
        secondary: "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
        success: "bg-success text-success-foreground hover:bg-success/90",
        warning: "bg-warning text-warning-foreground hover:bg-warning/90",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
        xl: "h-12 rounded-lg px-10 text-base",
        icon: "h-10 w-10",
        "icon-sm": "h-8 w-8",
        "icon-lg": "h-12 w-12",
      },
      fullWidth: {
        true: "w-full",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);

export type ButtonVariants = VariantProps<typeof buttonVariants>;

// ── Badge Variants ──
export const badgeVariants = cva(
  "inline-flex items-center rounded-full border px-2.5 py-0.5 text-xs font-semibold transition-colors focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2",
  {
    variants: {
      variant: {
        default: "border-transparent bg-primary text-primary-foreground",
        secondary: "border-transparent bg-secondary text-secondary-foreground",
        destructive: "border-transparent bg-destructive text-destructive-foreground",
        outline: "text-foreground",
        success: "border-transparent bg-success/10 text-success",
        warning: "border-transparent bg-warning/10 text-warning",
        info: "border-transparent bg-info/10 text-info",
      },
      size: {
        default: "px-2.5 py-0.5 text-xs",
        sm: "px-2 py-0 text-2xs",
        lg: "px-3 py-1 text-sm",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);

export type BadgeVariants = VariantProps<typeof badgeVariants>;

// ── Input Variants ──
export const inputVariants = cva(
  "flex w-full rounded-md border bg-background text-sm ring-offset-background file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "border-input",
        error: "border-destructive focus-visible:ring-destructive",
        success: "border-success focus-visible:ring-success",
      },
      inputSize: {
        default: "h-10 px-3 py-2",
        sm: "h-9 px-3 py-1 text-xs",
        lg: "h-12 px-4 py-3 text-base",
      },
    },
    defaultVariants: {
      variant: "default",
      inputSize: "default",
    },
  }
);

export type InputVariants = VariantProps<typeof inputVariants>;

// ── Alert Variants ──
export const alertVariants = cva(
  "relative w-full rounded-lg border p-4 [&>svg~*]:pl-7 [&>svg+div]:translate-y-[-3px] [&>svg]:absolute [&>svg]:left-4 [&>svg]:top-4 [&>svg]:text-foreground",
  {
    variants: {
      variant: {
        default: "bg-background text-foreground",
        destructive: "border-destructive/50 text-destructive dark:border-destructive [&>svg]:text-destructive",
        success: "border-success/50 bg-success/5 text-success dark:border-success [&>svg]:text-success",
        warning: "border-warning/50 bg-warning/5 text-warning dark:border-warning [&>svg]:text-warning",
        info: "border-info/50 bg-info/5 text-info dark:border-info [&>svg]:text-info",
      },
    },
    defaultVariants: {
      variant: "default",
    },
  }
);

export type AlertVariants = VariantProps<typeof alertVariants>;

// ── Card Variants ──
export const cardVariants = cva("rounded-lg border bg-card text-card-foreground", {
  variants: {
    variant: {
      default: "shadow-sm",
      elevated: "shadow-elevation-3",
      outline: "shadow-none",
      ghost: "border-transparent shadow-none bg-transparent",
      interactive: "shadow-sm transition-all hover:shadow-elevation-3 hover:-translate-y-0.5 cursor-pointer",
    },
    padding: {
      none: "",
      sm: "p-4",
      default: "p-6",
      lg: "p-8",
    },
  },
  defaultVariants: {
    variant: "default",
    padding: "default",
  },
});

export type CardVariants = VariantProps<typeof cardVariants>;
```

### cn utility

```typescript
// lib/utils.ts — Utility functions

import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// Format date
export function formatDate(
  date: Date | string,
  locale = "it-IT",
  options?: Intl.DateTimeFormatOptions
): string {
  return new Intl.DateTimeFormat(locale, {
    dateStyle: "medium",
    ...options,
  }).format(new Date(date));
}

// Format relative time
export function formatRelativeTime(date: Date | string): string {
  const now = new Date();
  const then = new Date(date);
  const diffMs = now.getTime() - then.getTime();
  const diffSeconds = Math.floor(diffMs / 1000);
  const diffMinutes = Math.floor(diffSeconds / 60);
  const diffHours = Math.floor(diffMinutes / 60);
  const diffDays = Math.floor(diffHours / 24);

  if (diffSeconds < 60) return "ora";
  if (diffMinutes < 60) return `${diffMinutes}m fa`;
  if (diffHours < 24) return `${diffHours}h fa`;
  if (diffDays < 7) return `${diffDays}g fa`;

  return formatDate(date);
}

// Truncate text
export function truncate(text: string, length: number): string {
  if (text.length <= length) return text;
  return text.substring(0, length).trimEnd() + "...";
}
```

## RESPONSIVE DESIGN SYSTEM

### Panoramica
Sistema responsive completo con container queries, breakpoint hooks,
e componenti responsive-aware.

### Implementazione Completa

```typescript
// hooks/use-media-query.ts — Responsive hooks

"use client";

import { useState, useEffect } from "react";

export function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(false);

  useEffect(() => {
    const media = window.matchMedia(query);
    setMatches(media.matches);

    const listener = (event: MediaQueryListEvent) => {
      setMatches(event.matches);
    };

    media.addEventListener("change", listener);
    return () => media.removeEventListener("change", listener);
  }, [query]);

  return matches;
}

// Breakpoint-specific hooks
export function useIsMobile(): boolean {
  return useMediaQuery("(max-width: 767px)");
}

export function useIsTablet(): boolean {
  return useMediaQuery("(min-width: 768px) and (max-width: 1023px)");
}

export function useIsDesktop(): boolean {
  return useMediaQuery("(min-width: 1024px)");
}

export function useBreakpoint(): "mobile" | "tablet" | "desktop" | "wide" {
  const isMobile = useMediaQuery("(max-width: 767px)");
  const isTablet = useMediaQuery("(min-width: 768px) and (max-width: 1023px)");
  const isDesktop = useMediaQuery("(min-width: 1024px) and (max-width: 1279px)");

  if (isMobile) return "mobile";
  if (isTablet) return "tablet";
  if (isDesktop) return "desktop";
  return "wide";
}

// Reduce motion preference
export function usePrefersReducedMotion(): boolean {
  return useMediaQuery("(prefers-reduced-motion: reduce)");
}

// Color scheme preference
export function usePrefersColorScheme(): "light" | "dark" | "no-preference" {
  const prefersDark = useMediaQuery("(prefers-color-scheme: dark)");
  const prefersLight = useMediaQuery("(prefers-color-scheme: light)");

  if (prefersDark) return "dark";
  if (prefersLight) return "light";
  return "no-preference";
}
```

```typescript
// components/layout/ResponsiveContainer.tsx — Responsive container

import { cn } from "@/lib/utils";

interface ResponsiveContainerProps {
  children: React.ReactNode;
  className?: string;
  as?: "div" | "section" | "main" | "article";
  maxWidth?: "sm" | "md" | "lg" | "xl" | "2xl" | "full";
  padding?: "none" | "sm" | "default" | "lg";
}

const maxWidthClasses = {
  sm: "max-w-screen-sm",
  md: "max-w-screen-md",
  lg: "max-w-screen-lg",
  xl: "max-w-screen-xl",
  "2xl": "max-w-screen-2xl",
  full: "max-w-full",
};

const paddingClasses = {
  none: "",
  sm: "px-4 sm:px-6",
  default: "px-4 sm:px-6 lg:px-8",
  lg: "px-6 sm:px-8 lg:px-12",
};

export function ResponsiveContainer({
  children,
  className,
  as: Component = "div",
  maxWidth = "xl",
  padding = "default",
}: ResponsiveContainerProps) {
  return (
    <Component
      className={cn(
        "mx-auto w-full",
        maxWidthClasses[maxWidth],
        paddingClasses[padding],
        className
      )}
    >
      {children}
    </Component>
  );
}
```

## ICON SYSTEM CON LUCIDE-REACT

### Implementazione Completa

```typescript
// components/ui/icon.tsx — Icon system component

import { cn } from "@/lib/utils";
import { type LucideIcon, type LucideProps } from "lucide-react";
import { cva, type VariantProps } from "class-variance-authority";

const iconVariants = cva("shrink-0", {
  variants: {
    size: {
      xs: "h-3 w-3",
      sm: "h-4 w-4",
      default: "h-5 w-5",
      lg: "h-6 w-6",
      xl: "h-8 w-8",
      "2xl": "h-10 w-10",
    },
    variant: {
      default: "",
      muted: "text-muted-foreground",
      primary: "text-primary",
      destructive: "text-destructive",
      success: "text-success",
      warning: "text-warning",
    },
  },
  defaultVariants: {
    size: "default",
    variant: "default",
  },
});

export interface IconProps
  extends Omit<LucideProps, "size">,
    VariantProps<typeof iconVariants> {
  icon: LucideIcon;
}

export function Icon({
  icon: LucideIcon,
  size,
  variant,
  className,
  ...props
}: IconProps) {
  return (
    <LucideIcon
      className={cn(iconVariants({ size, variant }), className)}
      {...props}
    />
  );
}

// Icon Button component
interface IconButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  icon: LucideIcon;
  label: string;
  size?: "sm" | "default" | "lg";
  variant?: "default" | "ghost" | "outline";
}

export function IconButton({
  icon: LucideIcon,
  label,
  size = "default",
  variant = "ghost",
  className,
  ...props
}: IconButtonProps) {
  const sizeClasses = {
    sm: "h-8 w-8 [&>svg]:h-4 [&>svg]:w-4",
    default: "h-10 w-10 [&>svg]:h-5 [&>svg]:w-5",
    lg: "h-12 w-12 [&>svg]:h-6 [&>svg]:w-6",
  };

  const variantClasses = {
    default: "bg-primary text-primary-foreground hover:bg-primary/90",
    ghost: "hover:bg-accent hover:text-accent-foreground",
    outline: "border border-input bg-background hover:bg-accent",
  };

  return (
    <button
      type="button"
      className={cn(
        "inline-flex items-center justify-center rounded-md transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring",
        sizeClasses[size],
        variantClasses[variant],
        className
      )}
      aria-label={label}
      {...props}
    >
      <LucideIcon />
    </button>
  );
}
```

### Errori Comuni da Evitare
- Non usare valori hardcoded per colori — sempre riferirsi ai token CSS
- Non dimenticare il dark mode per ogni componente custom
- Non usare `px` per spacing — sempre il sistema a griglia 4px di Tailwind
- Non dimenticare `motion-safe` / `motion-reduce` per le animazioni
- Non importare font direttamente con `@font-face` — usare `next/font`

### Checklist di Verifica
- [ ] Design tokens definiti come CSS custom properties
- [ ] Tailwind config esteso con tutti i token
- [ ] Dark mode funzionante con next-themes
- [ ] Font loading ottimizzato con next/font
- [ ] CVA variants per tutti i componenti base
- [ ] Responsive hooks disponibili per ogni breakpoint
- [ ] Motion preferences rispettate
- [ ] Container queries per componenti adattivi
- [ ] Sistema di ombre/elevation consistente
- [ ] Icon system con sizing scale



## ANIMATION-TOKEN-SYSTEM

### Panoramica
Sistema completo di animation tokens per micro-interazioni, transizioni di pagina e feedback visivi. Integra Framer Motion con il design system per animazioni consistenti.

### Implementazione Completa

```typescript
// lib/design-system/animation-tokens.ts
import { Variants, Transition, TargetAndTransition } from "framer-motion";

// ============================================================
// ANIMATION DURATION TOKENS
// ============================================================
export const duration = {
  instant: 0.1,
  fast: 0.15,
  normal: 0.25,
  slow: 0.4,
  slower: 0.6,
  slowest: 1.0,
} as const;

// ============================================================
// EASING TOKENS
// ============================================================
export const easing = {
  easeInOut: [0.4, 0, 0.2, 1] as [number, number, number, number],
  easeOut: [0, 0, 0.2, 1] as [number, number, number, number],
  easeIn: [0.4, 0, 1, 1] as [number, number, number, number],
  sharp: [0.4, 0, 0.6, 1] as [number, number, number, number],
  bounce: [0.68, -0.55, 0.265, 1.55] as [number, number, number, number],
  spring: { type: "spring" as const, stiffness: 300, damping: 30 },
  springBouncy: { type: "spring" as const, stiffness: 400, damping: 15 },
  springSmooth: { type: "spring" as const, stiffness: 200, damping: 25 },
} as const;

// ============================================================
// TRANSITION PRESETS
// ============================================================
export const transition: Record<string, Transition> = {
  fast: { duration: duration.fast, ease: easing.easeOut },
  normal: { duration: duration.normal, ease: easing.easeInOut },
  slow: { duration: duration.slow, ease: easing.easeInOut },
  spring: easing.spring,
  springBouncy: easing.springBouncy,
  springSmooth: easing.springSmooth,
};

// ============================================================
// VARIANT PRESETS — Page Transitions
// ============================================================
export const pageTransition: Variants = {
  initial: { opacity: 0, y: 20 },
  enter: {
    opacity: 1,
    y: 0,
    transition: { duration: duration.normal, ease: easing.easeOut },
  },
  exit: {
    opacity: 0,
    y: -20,
    transition: { duration: duration.fast, ease: easing.easeIn },
  },
};

export const fadeIn: Variants = {
  initial: { opacity: 0 },
  enter: { opacity: 1, transition: { duration: duration.normal } },
  exit: { opacity: 0, transition: { duration: duration.fast } },
};

export const slideUp: Variants = {
  initial: { opacity: 0, y: 40 },
  enter: {
    opacity: 1,
    y: 0,
    transition: { duration: duration.slow, ease: easing.easeOut },
  },
  exit: {
    opacity: 0,
    y: 40,
    transition: { duration: duration.normal, ease: easing.easeIn },
  },
};

export const slideRight: Variants = {
  initial: { opacity: 0, x: -40 },
  enter: {
    opacity: 1,
    x: 0,
    transition: { duration: duration.slow, ease: easing.easeOut },
  },
  exit: {
    opacity: 0,
    x: 40,
    transition: { duration: duration.normal, ease: easing.easeIn },
  },
};

export const scaleUp: Variants = {
  initial: { opacity: 0, scale: 0.9 },
  enter: {
    opacity: 1,
    scale: 1,
    transition: easing.spring,
  },
  exit: {
    opacity: 0,
    scale: 0.9,
    transition: { duration: duration.fast },
  },
};

// ============================================================
// VARIANT PRESETS — Stagger Children
// ============================================================
export const staggerContainer: Variants = {
  initial: {},
  enter: {
    transition: {
      staggerChildren: 0.08,
      delayChildren: 0.1,
    },
  },
  exit: {
    transition: {
      staggerChildren: 0.05,
      staggerDirection: -1,
    },
  },
};

export const staggerItem: Variants = {
  initial: { opacity: 0, y: 20 },
  enter: {
    opacity: 1,
    y: 0,
    transition: { duration: duration.normal, ease: easing.easeOut },
  },
  exit: {
    opacity: 0,
    y: 10,
    transition: { duration: duration.fast },
  },
};

// ============================================================
// MICRO-INTERACTION PRESETS
// ============================================================
export const buttonTap: TargetAndTransition = {
  scale: 0.97,
  transition: { duration: duration.instant },
};

export const buttonHover: TargetAndTransition = {
  scale: 1.02,
  transition: easing.spring,
};

export const cardHover: TargetAndTransition = {
  y: -4,
  boxShadow: "0 12px 24px rgba(0,0,0,0.1)",
  transition: { duration: duration.normal, ease: easing.easeOut },
};

export const pulseAnimation: Variants = {
  initial: { scale: 1 },
  pulse: {
    scale: [1, 1.05, 1],
    transition: {
      duration: 1.5,
      repeat: Infinity,
      ease: "easeInOut",
    },
  },
};

export const shakeAnimation: Variants = {
  initial: { x: 0 },
  shake: {
    x: [-10, 10, -8, 8, -4, 4, 0],
    transition: { duration: 0.5 },
  },
};

export const spinAnimation: Variants = {
  initial: { rotate: 0 },
  spin: {
    rotate: 360,
    transition: {
      duration: 1,
      repeat: Infinity,
      ease: "linear",
    },
  },
};
```

### Varianti e Configurazioni

```typescript
// components/ui/animated-layout.tsx
"use client";

import { motion, AnimatePresence, LayoutGroup } from "framer-motion";
import { ReactNode } from "react";
import {
  pageTransition,
  fadeIn,
  staggerContainer,
  staggerItem,
  transition,
} from "@/lib/design-system/animation-tokens";

// ============================================================
// ANIMATED PAGE WRAPPER
// ============================================================
interface AnimatedPageProps {
  children: ReactNode;
  className?: string;
  variant?: "fade" | "slide" | "scale";
}

export function AnimatedPage({
  children,
  className,
  variant = "slide",
}: AnimatedPageProps) {
  const variants = variant === "fade" ? fadeIn : pageTransition;

  return (
    <motion.div
      initial="initial"
      animate="enter"
      exit="exit"
      variants={variants}
      className={className}
    >
      {children}
    </motion.div>
  );
}

// ============================================================
// STAGGER LIST
// ============================================================
interface StaggerListProps {
  children: ReactNode;
  className?: string;
  delay?: number;
}

export function StaggerList({
  children,
  className,
  delay = 0.1,
}: StaggerListProps) {
  return (
    <motion.div
      initial="initial"
      animate="enter"
      exit="exit"
      variants={{
        ...staggerContainer,
        enter: {
          transition: {
            staggerChildren: 0.08,
            delayChildren: delay,
          },
        },
      }}
      className={className}
    >
      {children}
    </motion.div>
  );
}

export function StaggerItem({
  children,
  className,
}: {
  children: ReactNode;
  className?: string;
}) {
  return (
    <motion.div variants={staggerItem} className={className}>
      {children}
    </motion.div>
  );
}

// ============================================================
// ANIMATED PRESENCE WRAPPER for route transitions
// ============================================================
interface RouteTransitionProps {
  children: ReactNode;
  routeKey: string;
}

export function RouteTransition({ children, routeKey }: RouteTransitionProps) {
  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={routeKey}
        initial="initial"
        animate="enter"
        exit="exit"
        variants={pageTransition}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
}

// ============================================================
// LAYOUT ANIMATION WRAPPER
// ============================================================
interface AnimatedListProps<T> {
  items: T[];
  keyExtractor: (item: T) => string;
  renderItem: (item: T) => ReactNode;
  className?: string;
}

export function AnimatedList<T>({
  items,
  keyExtractor,
  renderItem,
  className,
}: AnimatedListProps<T>) {
  return (
    <LayoutGroup>
      <div className={className}>
        <AnimatePresence mode="popLayout">
          {items.map((item) => (
            <motion.div
              key={keyExtractor(item)}
              layout
              initial={{ opacity: 0, scale: 0.8 }}
              animate={{ opacity: 1, scale: 1 }}
              exit={{ opacity: 0, scale: 0.8 }}
              transition={transition.spring}
            >
              {renderItem(item)}
            </motion.div>
          ))}
        </AnimatePresence>
      </div>
    </LayoutGroup>
  );
}

// ============================================================
// SCROLL REVEAL
// ============================================================
interface ScrollRevealProps {
  children: ReactNode;
  className?: string;
  direction?: "up" | "down" | "left" | "right";
  delay?: number;
}

export function ScrollReveal({
  children,
  className,
  direction = "up",
  delay = 0,
}: ScrollRevealProps) {
  const directionMap = {
    up: { y: 40 },
    down: { y: -40 },
    left: { x: 40 },
    right: { x: -40 },
  };

  return (
    <motion.div
      initial={{ opacity: 0, ...directionMap[direction] }}
      whileInView={{ opacity: 1, x: 0, y: 0 }}
      viewport={{ once: true, margin: "-100px" }}
      transition={{
        duration: 0.6,
        delay,
        ease: [0.4, 0, 0.2, 1],
      }}
      className={className}
    >
      {children}
    </motion.div>
  );
}

// ============================================================
// NUMBER COUNTER ANIMATION
// ============================================================
interface AnimatedCounterProps {
  from: number;
  to: number;
  duration?: number;
  className?: string;
  formatter?: (value: number) => string;
}

export function AnimatedCounter({
  from,
  to,
  duration = 1.5,
  className,
  formatter = (v) => Math.round(v).toLocaleString(),
}: AnimatedCounterProps) {
  return (
    <motion.span
      className={className}
      initial={{ opacity: 0 }}
      whileInView={{ opacity: 1 }}
      viewport={{ once: true }}
    >
      <motion.span
        initial={{ "--count": from } as any}
        whileInView={{ "--count": to } as any}
        viewport={{ once: true }}
        transition={{ duration, ease: "easeOut" }}
      >
        {formatter(to)}
      </motion.span>
    </motion.span>
  );
}
```

### Edge Cases e Error Handling

```typescript
// lib/design-system/animation-utils.ts
"use client";

import { useReducedMotion } from "framer-motion";
import { Variants, Transition } from "framer-motion";

// ============================================================
// REDUCED MOTION SUPPORT
// ============================================================
export function useAccessibleAnimation(
  variants: Variants,
  reducedVariants?: Variants
): Variants {
  const shouldReduce = useReducedMotion();

  if (shouldReduce) {
    return (
      reducedVariants ?? {
        initial: { opacity: 0 },
        enter: { opacity: 1, transition: { duration: 0.01 } },
        exit: { opacity: 0, transition: { duration: 0.01 } },
      }
    );
  }

  return variants;
}

// ============================================================
// SAFE ANIMATION WRAPPER — prevents animations on SSR
// ============================================================
import { useEffect, useState, ReactNode } from "react";
import { motion, AnimatePresence } from "framer-motion";

export function SafeAnimatePresence({
  children,
  ...props
}: {
  children: ReactNode;
  mode?: "wait" | "sync" | "popLayout";
}) {
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);
  }, []);

  if (!mounted) {
    return <>{children}</>;
  }

  return <AnimatePresence {...props}>{children}</AnimatePresence>;
}

// ============================================================
// PERFORMANCE — Prevent layout thrashing with will-change
// ============================================================
export const performanceTransition: Transition = {
  type: "tween",
  duration: 0.25,
  ease: [0.4, 0, 0.2, 1],
};

export const gpuAcceleratedStyle = {
  willChange: "transform, opacity",
  transform: "translateZ(0)",
} as const;

// ============================================================
// ANIMATION DEBUGGER — Development only
// ============================================================
export function useAnimationDebug(label: string) {
  if (process.env.NODE_ENV !== "development") return {};

  return {
    onAnimationStart: () => console.log(`[Animation] ${label}: start`),
    onAnimationComplete: () => console.log(`[Animation] ${label}: complete`),
  };
}

// ============================================================
// GESTURE SPRING CONFIG — Prevents overshoot on mobile
// ============================================================
export const mobileSpring: Transition = {
  type: "spring",
  stiffness: 400,
  damping: 40,
  mass: 0.8,
};

export const desktopSpring: Transition = {
  type: "spring",
  stiffness: 300,
  damping: 25,
};

export function usePlatformSpring(): Transition {
  const [isMobile, setIsMobile] = useState(false);

  useEffect(() => {
    setIsMobile(window.matchMedia("(max-width: 768px)").matches);
  }, []);

  return isMobile ? mobileSpring : desktopSpring;
}
```

### Errori Comuni da Evitare
- **Layout thrashing**: Non animare `width`/`height` direttamente — usa `scale` o `transform`
- **Missing `will-change`**: Aggiungi `willChange: "transform"` per animazioni frequenti
- **SSR hydration mismatch**: Wrappa AnimatePresence in un check di mount client-side
- **Reduced motion ignorato**: Usa sempre `useReducedMotion()` e fornisci varianti accessibili

### Checklist di Verifica
- [ ] Tutti i token sono esportati da un unico entry point
- [ ] Le animazioni rispettano `prefers-reduced-motion`
- [ ] Le page transitions usano `AnimatePresence` con `mode="wait"`
- [ ] Le stagger list hanno delay progressivi ragionevoli (< 100ms)
- [ ] Le animazioni GPU-accelerated usano `transform`/`opacity` (non `top`/`left`)
- [ ] Il layout animation usa `LayoutGroup` per contesti multipli

---

## SPACING-AND-LAYOUT-SYSTEM

### Panoramica
Sistema di spacing e layout basato su una griglia a 4px con utility classes custom Tailwind, container responsive e pattern di layout comuni per un design consistente.

### Implementazione Completa

```typescript
// tailwind.config.ts — spacing and layout extensions
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./components/**/*.{ts,tsx}",
    "./app/**/*.{ts,tsx}",
  ],
  theme: {
    extend: {
      // 4px grid spacing system
      spacing: {
        "0.5": "2px",
        "1": "4px",
        "1.5": "6px",
        "2": "8px",
        "2.5": "10px",
        "3": "12px",
        "3.5": "14px",
        "4": "16px",
        "5": "20px",
        "6": "24px",
        "7": "28px",
        "8": "32px",
        "9": "36px",
        "10": "40px",
        "11": "44px",
        "12": "48px",
        "14": "56px",
        "16": "64px",
        "18": "72px",
        "20": "80px",
        "24": "96px",
        "28": "112px",
        "32": "128px",
        "36": "144px",
        "40": "160px",
        "44": "176px",
        "48": "192px",
        "52": "208px",
        "56": "224px",
        "60": "240px",
        "64": "256px",
        "72": "288px",
        "80": "320px",
        "96": "384px",
      },
      // Container sizes
      maxWidth: {
        "8xl": "88rem",
        "9xl": "96rem",
        "10xl": "104rem",
        content: "65ch",
        prose: "75ch",
      },
      // Section spacing
      padding: {
        section: "clamp(3rem, 8vw, 6rem)",
      },
      margin: {
        section: "clamp(3rem, 8vw, 6rem)",
      },
      // Grid template areas
      gridTemplateColumns: {
        sidebar: "280px 1fr",
        "sidebar-lg": "320px 1fr",
        "auto-fill-sm": "repeat(auto-fill, minmax(200px, 1fr))",
        "auto-fill-md": "repeat(auto-fill, minmax(280px, 1fr))",
        "auto-fill-lg": "repeat(auto-fill, minmax(360px, 1fr))",
      },
      // Z-index scale
      zIndex: {
        dropdown: "1000",
        sticky: "1020",
        fixed: "1030",
        backdrop: "1040",
        modal: "1050",
        popover: "1060",
        tooltip: "1070",
        toast: "1080",
      },
    },
  },
  plugins: [],
};

export default config;
```

```typescript
// components/layout/page-layout.tsx
"use client";

import { cn } from "@/lib/utils";
import { ReactNode } from "react";

// ============================================================
// SECTION CONTAINER — responsive padding with max-width
// ============================================================
interface SectionProps {
  children: ReactNode;
  className?: string;
  as?: "section" | "div" | "main" | "article";
  size?: "sm" | "md" | "lg" | "xl" | "full";
  padding?: "none" | "sm" | "md" | "lg";
}

const sectionSizes = {
  sm: "max-w-3xl",
  md: "max-w-5xl",
  lg: "max-w-7xl",
  xl: "max-w-9xl",
  full: "max-w-full",
};

const sectionPadding = {
  none: "",
  sm: "px-4 py-8 sm:px-6 sm:py-12",
  md: "px-4 py-12 sm:px-6 lg:px-8 sm:py-16 lg:py-20",
  lg: "px-4 py-16 sm:px-6 lg:px-8 sm:py-24 lg:py-32",
};

export function Section({
  children,
  className,
  as: Component = "section",
  size = "lg",
  padding = "md",
}: SectionProps) {
  return (
    <Component
      className={cn(
        "mx-auto w-full",
        sectionSizes[size],
        sectionPadding[padding],
        className
      )}
    >
      {children}
    </Component>
  );
}

// ============================================================
// STACK — Vertical spacing component
// ============================================================
interface StackProps {
  children: ReactNode;
  className?: string;
  gap?: "xs" | "sm" | "md" | "lg" | "xl";
  as?: "div" | "section" | "ul" | "ol" | "nav";
}

const stackGaps = {
  xs: "gap-1",
  sm: "gap-2",
  md: "gap-4",
  lg: "gap-6",
  xl: "gap-8",
};

export function Stack({
  children,
  className,
  gap = "md",
  as: Component = "div",
}: StackProps) {
  return (
    <Component className={cn("flex flex-col", stackGaps[gap], className)}>
      {children}
    </Component>
  );
}

// ============================================================
// CLUSTER — Horizontal wrapping layout
// ============================================================
interface ClusterProps {
  children: ReactNode;
  className?: string;
  gap?: "xs" | "sm" | "md" | "lg" | "xl";
  justify?: "start" | "center" | "end" | "between";
  align?: "start" | "center" | "end" | "baseline";
}

const justifyMap = {
  start: "justify-start",
  center: "justify-center",
  end: "justify-end",
  between: "justify-between",
};

const alignMap = {
  start: "items-start",
  center: "items-center",
  end: "items-end",
  baseline: "items-baseline",
};

export function Cluster({
  children,
  className,
  gap = "md",
  justify = "start",
  align = "center",
}: ClusterProps) {
  return (
    <div
      className={cn(
        "flex flex-wrap",
        stackGaps[gap],
        justifyMap[justify],
        alignMap[align],
        className
      )}
    >
      {children}
    </div>
  );
}

// ============================================================
// SIDEBAR LAYOUT
// ============================================================
interface SidebarLayoutProps {
  children: ReactNode;
  sidebar: ReactNode;
  className?: string;
  sidebarPosition?: "left" | "right";
  sidebarWidth?: string;
  breakpoint?: "md" | "lg" | "xl";
}

export function SidebarLayout({
  children,
  sidebar,
  className,
  sidebarPosition = "left",
  sidebarWidth = "280px",
  breakpoint = "lg",
}: SidebarLayoutProps) {
  const bpClass = {
    md: "md:grid md:grid-cols-sidebar",
    lg: "lg:grid lg:grid-cols-sidebar",
    xl: "xl:grid xl:grid-cols-sidebar",
  };

  return (
    <div
      className={cn("flex flex-col gap-6", bpClass[breakpoint], className)}
      style={
        { "--sidebar-width": sidebarWidth } as React.CSSProperties
      }
    >
      {sidebarPosition === "left" ? (
        <>
          <aside className="shrink-0">{sidebar}</aside>
          <main className="min-w-0">{children}</main>
        </>
      ) : (
        <>
          <main className="min-w-0">{children}</main>
          <aside className="shrink-0">{sidebar}</aside>
        </>
      )}
    </div>
  );
}

// ============================================================
// MASONRY GRID
// ============================================================
interface MasonryGridProps {
  children: ReactNode;
  columns?: { default: number; md?: number; lg?: number };
  gap?: string;
  className?: string;
}

export function MasonryGrid({
  children,
  columns = { default: 1, md: 2, lg: 3 },
  gap = "1.5rem",
  className,
}: MasonryGridProps) {
  return (
    <div
      className={cn("columns-1", className, {
        "md:columns-2": columns.md === 2,
        "md:columns-3": columns.md === 3,
        "lg:columns-2": columns.lg === 2,
        "lg:columns-3": columns.lg === 3,
        "lg:columns-4": columns.lg === 4,
      })}
      style={{ columnGap: gap }}
    >
      {children}
    </div>
  );
}
```

### Edge Cases e Error Handling

```typescript
// lib/design-system/layout-utils.ts
"use client";

import { useEffect, useState, useCallback, RefObject } from "react";

// ============================================================
// CONTAINER QUERY HOOK — responsive without media queries
// ============================================================
export function useContainerSize(ref: RefObject<HTMLElement>) {
  const [size, setSize] = useState({ width: 0, height: 0 });

  useEffect(() => {
    const el = ref.current;
    if (!el) return;

    const observer = new ResizeObserver((entries) => {
      for (const entry of entries) {
        const { width, height } = entry.contentRect;
        setSize({ width, height });
      }
    });

    observer.observe(el);
    return () => observer.disconnect();
  }, [ref]);

  return size;
}

// ============================================================
// SCROLL LOCK — prevents body scroll when modal/drawer open
// ============================================================
export function useScrollLock(locked: boolean) {
  useEffect(() => {
    if (!locked) return;

    const scrollY = window.scrollY;
    const body = document.body;
    const originalStyles = {
      overflow: body.style.overflow,
      position: body.style.position,
      top: body.style.top,
      width: body.style.width,
    };

    body.style.overflow = "hidden";
    body.style.position = "fixed";
    body.style.top = `-${scrollY}px`;
    body.style.width = "100%";

    return () => {
      body.style.overflow = originalStyles.overflow;
      body.style.position = originalStyles.position;
      body.style.top = originalStyles.top;
      body.style.width = originalStyles.width;
      window.scrollTo(0, scrollY);
    };
  }, [locked]);
}

// ============================================================
// VIEWPORT SIZE — SSR-safe window dimensions
// ============================================================
export function useViewportSize() {
  const [size, setSize] = useState({
    width: typeof window !== "undefined" ? window.innerWidth : 0,
    height: typeof window !== "undefined" ? window.innerHeight : 0,
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };

    window.addEventListener("resize", handleResize);
    handleResize();
    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return size;
}

// ============================================================
// STICKY OFFSET — accounts for fixed headers
// ============================================================
export function useStickyOffset(headerSelector = "[data-header]") {
  const [offset, setOffset] = useState(0);

  useEffect(() => {
    const header = document.querySelector(headerSelector);
    if (!header) return;

    const observer = new ResizeObserver((entries) => {
      for (const entry of entries) {
        setOffset(entry.contentRect.height);
      }
    });

    observer.observe(header);
    return () => observer.disconnect();
  }, [headerSelector]);

  return offset;
}

// ============================================================
// INTERSECTION OBSERVER — trigger animations on scroll
// ============================================================
export function useInView(
  ref: RefObject<HTMLElement>,
  options?: IntersectionObserverInit
) {
  const [inView, setInView] = useState(false);

  useEffect(() => {
    const el = ref.current;
    if (!el) return;

    const observer = new IntersectionObserver(([entry]) => {
      setInView(entry.isIntersecting);
    }, options);

    observer.observe(el);
    return () => observer.disconnect();
  }, [ref, options]);

  return inView;
}
```

### Errori Comuni da Evitare
- **Spacing inconsistente**: Usa sempre i token di spacing (multipli di 4px), mai valori arbitrari come `13px`
- **Nested containers**: Non nestare più `Section` con padding — usa `padding="none"` per sezioni interne
- **z-index wars**: Usa sempre la scala di z-index definita, mai valori come `z-[9999]`
- **Mobile overflow**: Aggiungi `min-w-0` su flex children per prevenire overflow su mobile

### Checklist di Verifica
- [ ] Tutti gli spacing usano la scala a 4px
- [ ] I container hanno max-width appropriati per ogni breakpoint
- [ ] La sidebar è collassabile su mobile
- [ ] La MasonryGrid gestisce layout shift con CSS columns
- [ ] Lo scroll lock preserva la posizione di scroll corrente
- [ ] I layout sono testati a 320px, 768px, 1024px, 1440px

---

## COLOR-SYSTEM-AND-THEMING

### Panoramica
Sistema colori completo con supporto per brand customization, semantic tokens, accessible contrast ratios, e generazione dinamica di palette.

### Implementazione Completa

```typescript
// lib/design-system/color-system.ts

// ============================================================
// COLOR PALETTE GENERATOR
// ============================================================
interface HSL {
  h: number;
  s: number;
  l: number;
}

function hslToString({ h, s, l }: HSL): string {
  return `${h} ${s}% ${l}%`;
}

export function generatePalette(baseHue: number): Record<string, string> {
  return {
    "50": hslToString({ h: baseHue, s: 95, l: 97 }),
    "100": hslToString({ h: baseHue, s: 90, l: 93 }),
    "200": hslToString({ h: baseHue, s: 85, l: 85 }),
    "300": hslToString({ h: baseHue, s: 78, l: 73 }),
    "400": hslToString({ h: baseHue, s: 70, l: 58 }),
    "500": hslToString({ h: baseHue, s: 65, l: 48 }),
    "600": hslToString({ h: baseHue, s: 70, l: 40 }),
    "700": hslToString({ h: baseHue, s: 72, l: 33 }),
    "800": hslToString({ h: baseHue, s: 68, l: 26 }),
    "900": hslToString({ h: baseHue, s: 65, l: 20 }),
    "950": hslToString({ h: baseHue, s: 75, l: 10 }),
  };
}

// ============================================================
// SEMANTIC COLOR TOKENS
// ============================================================
export interface SemanticColors {
  background: string;
  foreground: string;
  card: string;
  cardForeground: string;
  popover: string;
  popoverForeground: string;
  primary: string;
  primaryForeground: string;
  secondary: string;
  secondaryForeground: string;
  muted: string;
  mutedForeground: string;
  accent: string;
  accentForeground: string;
  destructive: string;
  destructiveForeground: string;
  success: string;
  successForeground: string;
  warning: string;
  warningForeground: string;
  info: string;
  infoForeground: string;
  border: string;
  input: string;
  ring: string;
}

export const lightTheme: SemanticColors = {
  background: "0 0% 100%",
  foreground: "222.2 84% 4.9%",
  card: "0 0% 100%",
  cardForeground: "222.2 84% 4.9%",
  popover: "0 0% 100%",
  popoverForeground: "222.2 84% 4.9%",
  primary: "221.2 83.2% 53.3%",
  primaryForeground: "210 40% 98%",
  secondary: "210 40% 96%",
  secondaryForeground: "222.2 47.4% 11.2%",
  muted: "210 40% 96%",
  mutedForeground: "215.4 16.3% 46.9%",
  accent: "210 40% 96%",
  accentForeground: "222.2 47.4% 11.2%",
  destructive: "0 84.2% 60.2%",
  destructiveForeground: "210 40% 98%",
  success: "142 76% 36%",
  successForeground: "0 0% 100%",
  warning: "38 92% 50%",
  warningForeground: "0 0% 0%",
  info: "199 89% 48%",
  infoForeground: "0 0% 100%",
  border: "214.3 31.8% 91.4%",
  input: "214.3 31.8% 91.4%",
  ring: "221.2 83.2% 53.3%",
};

export const darkTheme: SemanticColors = {
  background: "222.2 84% 4.9%",
  foreground: "210 40% 98%",
  card: "222.2 84% 4.9%",
  cardForeground: "210 40% 98%",
  popover: "222.2 84% 4.9%",
  popoverForeground: "210 40% 98%",
  primary: "217.2 91.2% 59.8%",
  primaryForeground: "222.2 47.4% 11.2%",
  secondary: "217.2 32.6% 17.5%",
  secondaryForeground: "210 40% 98%",
  muted: "217.2 32.6% 17.5%",
  mutedForeground: "215 20.2% 65.1%",
  accent: "217.2 32.6% 17.5%",
  accentForeground: "210 40% 98%",
  destructive: "0 62.8% 30.6%",
  destructiveForeground: "210 40% 98%",
  success: "142 70% 24%",
  successForeground: "210 40% 98%",
  warning: "38 80% 40%",
  warningForeground: "210 40% 98%",
  info: "199 80% 35%",
  infoForeground: "210 40% 98%",
  border: "217.2 32.6% 17.5%",
  input: "217.2 32.6% 17.5%",
  ring: "224.3 76.3% 48%",
};

// ============================================================
// DYNAMIC THEME GENERATOR
// ============================================================
export function generateTheme(brandHue: number): {
  light: SemanticColors;
  dark: SemanticColors;
} {
  const palette = generatePalette(brandHue);

  return {
    light: {
      ...lightTheme,
      primary: palette["500"],
      primaryForeground: "0 0% 100%",
      ring: palette["500"],
      accent: palette["100"],
      accentForeground: palette["900"],
    },
    dark: {
      ...darkTheme,
      primary: palette["400"],
      primaryForeground: palette["950"],
      ring: palette["400"],
      accent: palette["800"],
      accentForeground: palette["100"],
    },
  };
}

// ============================================================
// APPLY THEME TO DOM
// ============================================================
export function applyTheme(
  theme: SemanticColors,
  element: HTMLElement = document.documentElement
) {
  Object.entries(theme).forEach(([key, value]) => {
    const cssVar = `--${key.replace(/([A-Z])/g, "-$1").toLowerCase()}`;
    element.style.setProperty(cssVar, value);
  });
}

// ============================================================
// CONTRAST CHECKER — WCAG 2.1 compliance
// ============================================================
function luminance(r: number, g: number, b: number): number {
  const [rs, gs, bs] = [r, g, b].map((c) => {
    c = c / 255;
    return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4);
  });
  return 0.2126 * rs + 0.7152 * gs + 0.0722 * bs;
}

export function contrastRatio(
  fg: [number, number, number],
  bg: [number, number, number]
): number {
  const l1 = luminance(...fg);
  const l2 = luminance(...bg);
  const lighter = Math.max(l1, l2);
  const darker = Math.min(l1, l2);
  return (lighter + 0.05) / (darker + 0.05);
}

export function meetsWCAG(
  fg: [number, number, number],
  bg: [number, number, number],
  level: "AA" | "AAA" = "AA",
  isLargeText = false
): boolean {
  const ratio = contrastRatio(fg, bg);
  if (level === "AAA") return isLargeText ? ratio >= 4.5 : ratio >= 7;
  return isLargeText ? ratio >= 3 : ratio >= 4.5;
}
```

### Varianti e Configurazioni

```typescript
// components/ui/theme-customizer.tsx
"use client";

import { useState, useEffect } from "react";
import { Button } from "@/components/ui/button";
import { Slider } from "@/components/ui/slider";
import { Label } from "@/components/ui/label";
import {
  Sheet,
  SheetContent,
  SheetDescription,
  SheetHeader,
  SheetTitle,
  SheetTrigger,
} from "@/components/ui/sheet";
import { Paintbrush } from "lucide-react";
import { generateTheme, applyTheme } from "@/lib/design-system/color-system";
import { useTheme } from "next-themes";

const presetThemes = [
  { name: "Blue", hue: 221, color: "#3b82f6" },
  { name: "Purple", hue: 262, color: "#8b5cf6" },
  { name: "Green", hue: 142, color: "#22c55e" },
  { name: "Orange", hue: 25, color: "#f97316" },
  { name: "Rose", hue: 346, color: "#f43f5e" },
  { name: "Cyan", hue: 192, color: "#06b6d4" },
] as const;

export function ThemeCustomizer() {
  const { resolvedTheme } = useTheme();
  const [brandHue, setBrandHue] = useState(221);
  const [borderRadius, setBorderRadius] = useState(0.5);

  useEffect(() => {
    const theme = generateTheme(brandHue);
    const activeTheme = resolvedTheme === "dark" ? theme.dark : theme.light;
    applyTheme(activeTheme);
    document.documentElement.style.setProperty(
      "--radius",
      `${borderRadius}rem`
    );
  }, [brandHue, borderRadius, resolvedTheme]);

  return (
    <Sheet>
      <SheetTrigger asChild>
        <Button variant="outline" size="icon">
          <Paintbrush className="h-4 w-4" />
        </Button>
      </SheetTrigger>
      <SheetContent>
        <SheetHeader>
          <SheetTitle>Customize Theme</SheetTitle>
          <SheetDescription>
            Adjust colors and styling to match your brand.
          </SheetDescription>
        </SheetHeader>
        <div className="space-y-6 py-6">
          <div className="space-y-2">
            <Label>Brand Color</Label>
            <div className="grid grid-cols-3 gap-2">
              {presetThemes.map((preset) => (
                <button
                  key={preset.name}
                  onClick={() => setBrandHue(preset.hue)}
                  className="flex items-center gap-2 rounded-md border p-2 hover:bg-accent"
                  style={{
                    borderColor:
                      brandHue === preset.hue
                        ? preset.color
                        : "transparent",
                  }}
                >
                  <div
                    className="h-4 w-4 rounded-full"
                    style={{ backgroundColor: preset.color }}
                  />
                  <span className="text-xs">{preset.name}</span>
                </button>
              ))}
            </div>
          </div>

          <div className="space-y-2">
            <Label>Custom Hue ({brandHue})</Label>
            <Slider
              value={[brandHue]}
              onValueChange={([v]) => setBrandHue(v)}
              min={0}
              max={360}
              step={1}
              className="w-full"
            />
            <div
              className="h-8 rounded-md"
              style={{
                background: `linear-gradient(to right,
                  hsl(0, 70%, 50%), hsl(60, 70%, 50%),
                  hsl(120, 70%, 50%), hsl(180, 70%, 50%),
                  hsl(240, 70%, 50%), hsl(300, 70%, 50%),
                  hsl(360, 70%, 50%))`,
              }}
            />
          </div>

          <div className="space-y-2">
            <Label>Border Radius ({borderRadius}rem)</Label>
            <Slider
              value={[borderRadius]}
              onValueChange={([v]) => setBorderRadius(v)}
              min={0}
              max={1}
              step={0.05}
              className="w-full"
            />
            <div className="flex gap-2">
              {[0, 0.25, 0.5, 0.75, 1].map((r) => (
                <div
                  key={r}
                  className="h-8 w-8 border-2 border-primary bg-primary/20"
                  style={{ borderRadius: `${r}rem` }}
                />
              ))}
            </div>
          </div>

          <Button
            variant="outline"
            className="w-full"
            onClick={() => {
              setBrandHue(221);
              setBorderRadius(0.5);
            }}
          >
            Reset to Default
          </Button>
        </div>
      </SheetContent>
    </Sheet>
  );
}
```

### Edge Cases e Error Handling

```typescript
// lib/design-system/color-utils.ts

// ============================================================
// HSL PARSING AND CONVERSION
// ============================================================
export function parseHSL(hslString: string): { h: number; s: number; l: number } | null {
  const match = hslString.match(
    /^(\d+\.?\d*)\s+(\d+\.?\d*)%\s+(\d+\.?\d*)%$/
  );
  if (!match) return null;
  return {
    h: parseFloat(match[1]),
    s: parseFloat(match[2]),
    l: parseFloat(match[3]),
  };
}

export function hslToRGB(
  h: number,
  s: number,
  l: number
): [number, number, number] {
  s /= 100;
  l /= 100;
  const c = (1 - Math.abs(2 * l - 1)) * s;
  const x = c * (1 - Math.abs(((h / 60) % 2) - 1));
  const m = l - c / 2;

  let r = 0, g = 0, b = 0;
  if (h < 60) { r = c; g = x; }
  else if (h < 120) { r = x; g = c; }
  else if (h < 180) { g = c; b = x; }
  else if (h < 240) { g = x; b = c; }
  else if (h < 300) { r = x; b = c; }
  else { r = c; b = x; }

  return [
    Math.round((r + m) * 255),
    Math.round((g + m) * 255),
    Math.round((b + m) * 255),
  ];
}

export function rgbToHex(r: number, g: number, b: number): string {
  return `#${[r, g, b].map((c) => c.toString(16).padStart(2, "0")).join("")}`;
}

// ============================================================
// ALPHA CHANNEL SUPPORT
// ============================================================
export function withAlpha(hslVar: string, alpha: number): string {
  return `hsl(${hslVar} / ${alpha})`;
}

// ============================================================
// AUTO FOREGROUND — choose black or white based on background
// ============================================================
export function autoForeground(
  bgH: number,
  bgS: number,
  bgL: number
): string {
  const [r, g, b] = hslToRGB(bgH, bgS, bgL);
  const lum = 0.2126 * (r / 255) + 0.7152 * (g / 255) + 0.0722 * (b / 255);
  return lum > 0.5 ? "0 0% 0%" : "0 0% 100%";
}

// ============================================================
// THEME PERSISTENCE
// ============================================================
const THEME_STORAGE_KEY = "app-custom-theme";

export function saveCustomTheme(config: { brandHue: number; borderRadius: number }) {
  if (typeof window === "undefined") return;
  localStorage.setItem(THEME_STORAGE_KEY, JSON.stringify(config));
}

export function loadCustomTheme(): { brandHue: number; borderRadius: number } | null {
  if (typeof window === "undefined") return null;
  const stored = localStorage.getItem(THEME_STORAGE_KEY);
  if (!stored) return null;
  try {
    const parsed = JSON.parse(stored);
    if (typeof parsed.brandHue === "number" && typeof parsed.borderRadius === "number") {
      return parsed;
    }
    return null;
  } catch {
    return null;
  }
}
```

### Errori Comuni da Evitare
- **Contrast insufficiente**: Verifica sempre il contrast ratio WCAG AA (4.5:1 per testo normale)
- **HSL string format errato**: shadcn/ui usa `"221.2 83.2% 53.3%"` senza `hsl()` wrapper
- **Theme flash**: Carica il tema da localStorage PRIMA del rendering per evitare il flash
- **Dark mode dimenticato**: Ogni colore semantico deve avere una variante light e dark

### Checklist di Verifica
- [ ] Tutti i colori semantici hanno varianti light e dark
- [ ] Il contrast ratio è >= 4.5:1 per testo normale
- [ ] I colori di brand sono personalizzabili senza toccare il codice
- [ ] Il tema viene persistito in localStorage
- [ ] Le transizioni di tema sono smooth (niente flash)
- [ ] Il color system funziona con SSR (no window access in server components)

---

## TYPOGRAPHY-SYSTEM

### Panoramica
Sistema tipografico completo con scale modulari, font loading ottimizzato, responsive typography e prose styles per contenuti editoriali.

### Implementazione Completa

```typescript
// lib/design-system/typography.ts
import { cva, type VariantProps } from "class-variance-authority";

// ============================================================
// TYPOGRAPHY SCALE — based on 1.250 (Major Third) ratio
// ============================================================
export const typographyScale = {
  "display-2xl": "text-[4.5rem] leading-[1.1] tracking-[-0.02em]",
  "display-xl": "text-[3.75rem] leading-[1.1] tracking-[-0.02em]",
  "display-lg": "text-[3rem] leading-[1.1] tracking-[-0.02em]",
  "display-md": "text-[2.25rem] leading-[1.2] tracking-[-0.02em]",
  "display-sm": "text-[1.875rem] leading-[1.2] tracking-[-0.01em]",
  "display-xs": "text-[1.5rem] leading-[1.2] tracking-[-0.01em]",
  "text-xl": "text-[1.25rem] leading-[1.5] tracking-[-0.01em]",
  "text-lg": "text-[1.125rem] leading-[1.5] tracking-[-0.01em]",
  "text-md": "text-[1rem] leading-[1.5]",
  "text-sm": "text-[0.875rem] leading-[1.5]",
  "text-xs": "text-[0.75rem] leading-[1.5]",
} as const;

// ============================================================
// HEADING COMPONENT
// ============================================================
export const headingVariants = cva("font-bold text-foreground", {
  variants: {
    level: {
      h1: "text-4xl sm:text-5xl lg:text-6xl tracking-tight leading-[1.1]",
      h2: "text-3xl sm:text-4xl lg:text-5xl tracking-tight leading-[1.15]",
      h3: "text-2xl sm:text-3xl tracking-tight leading-[1.2]",
      h4: "text-xl sm:text-2xl tracking-tight leading-[1.25]",
      h5: "text-lg sm:text-xl leading-[1.3]",
      h6: "text-base sm:text-lg leading-[1.4]",
    },
    weight: {
      normal: "font-normal",
      medium: "font-medium",
      semibold: "font-semibold",
      bold: "font-bold",
      extrabold: "font-extrabold",
    },
    align: {
      left: "text-left",
      center: "text-center",
      right: "text-right",
    },
  },
  defaultVariants: {
    level: "h2",
    weight: "bold",
    align: "left",
  },
});

export type HeadingVariantsProps = VariantProps<typeof headingVariants>;

// ============================================================
// TEXT COMPONENT
// ============================================================
export const textVariants = cva("text-foreground", {
  variants: {
    size: {
      xs: "text-xs",
      sm: "text-sm",
      base: "text-base",
      lg: "text-lg",
      xl: "text-xl",
    },
    weight: {
      normal: "font-normal",
      medium: "font-medium",
      semibold: "font-semibold",
      bold: "font-bold",
    },
    color: {
      default: "text-foreground",
      muted: "text-muted-foreground",
      primary: "text-primary",
      destructive: "text-destructive",
      success: "text-green-600 dark:text-green-400",
      warning: "text-yellow-600 dark:text-yellow-400",
    },
    leading: {
      tight: "leading-tight",
      snug: "leading-snug",
      normal: "leading-normal",
      relaxed: "leading-relaxed",
      loose: "leading-loose",
    },
  },
  defaultVariants: {
    size: "base",
    weight: "normal",
    color: "default",
    leading: "normal",
  },
});

export type TextVariantsProps = VariantProps<typeof textVariants>;
```

```typescript
// components/ui/typography.tsx
"use client";

import { cn } from "@/lib/utils";
import { ReactNode, createElement } from "react";
import {
  headingVariants,
  textVariants,
  type HeadingVariantsProps,
  type TextVariantsProps,
} from "@/lib/design-system/typography";

// ============================================================
// HEADING COMPONENT
// ============================================================
interface HeadingProps extends HeadingVariantsProps {
  children: ReactNode;
  className?: string;
  as?: "h1" | "h2" | "h3" | "h4" | "h5" | "h6" | "p" | "span";
  id?: string;
}

export function Heading({
  children,
  className,
  level = "h2",
  weight,
  align,
  as,
  id,
}: HeadingProps) {
  const Component = as ?? level ?? "h2";
  return createElement(
    Component,
    {
      className: cn(headingVariants({ level, weight, align }), className),
      id,
    },
    children
  );
}

// ============================================================
// TEXT COMPONENT
// ============================================================
interface TextProps extends TextVariantsProps {
  children: ReactNode;
  className?: string;
  as?: "p" | "span" | "div" | "label" | "small" | "strong" | "em";
}

export function Text({
  children,
  className,
  size,
  weight,
  color,
  leading,
  as: Component = "p",
}: TextProps) {
  return (
    <Component
      className={cn(textVariants({ size, weight, color, leading }), className)}
    >
      {children}
    </Component>
  );
}

// ============================================================
// PROSE — for rich content / CMS content
// ============================================================
interface ProseProps {
  children: ReactNode;
  className?: string;
  size?: "sm" | "base" | "lg" | "xl";
}

const proseSizes = {
  sm: "prose-sm",
  base: "prose-base",
  lg: "prose-lg",
  xl: "prose-xl",
};

export function Prose({ children, className, size = "base" }: ProseProps) {
  return (
    <div
      className={cn(
        "prose dark:prose-invert max-w-none",
        proseSizes[size],
        "prose-headings:font-bold prose-headings:tracking-tight",
        "prose-a:text-primary prose-a:no-underline hover:prose-a:underline",
        "prose-code:rounded prose-code:bg-muted prose-code:px-1.5 prose-code:py-0.5",
        "prose-code:before:content-none prose-code:after:content-none",
        "prose-pre:bg-muted prose-pre:border prose-pre:border-border",
        "prose-img:rounded-lg prose-img:border prose-img:border-border",
        "prose-blockquote:border-l-primary prose-blockquote:not-italic",
        "prose-th:text-left prose-th:font-semibold",
        "prose-td:border-t prose-td:border-border",
        className
      )}
    >
      {children}
    </div>
  );
}

// ============================================================
// TRUNCATE — single or multi-line
// ============================================================
interface TruncateProps {
  children: ReactNode;
  lines?: number;
  className?: string;
}

export function Truncate({ children, lines = 1, className }: TruncateProps) {
  if (lines === 1) {
    return (
      <span className={cn("truncate block", className)}>{children}</span>
    );
  }

  return (
    <span
      className={cn("line-clamp-[var(--lines)] block", className)}
      style={{ "--lines": lines } as React.CSSProperties}
    >
      {children}
    </span>
  );
}

// ============================================================
// GRADIENT TEXT
// ============================================================
interface GradientTextProps {
  children: ReactNode;
  className?: string;
  from?: string;
  to?: string;
  via?: string;
}

export function GradientText({
  children,
  className,
  from = "from-primary",
  to = "to-purple-500",
  via,
}: GradientTextProps) {
  return (
    <span
      className={cn(
        "bg-gradient-to-r bg-clip-text text-transparent",
        from,
        via,
        to,
        className
      )}
    >
      {children}
    </span>
  );
}

// ============================================================
// CODE — inline code styling
// ============================================================
export function InlineCode({
  children,
  className,
}: {
  children: ReactNode;
  className?: string;
}) {
  return (
    <code
      className={cn(
        "rounded bg-muted px-1.5 py-0.5 text-sm font-mono font-medium",
        className
      )}
    >
      {children}
    </code>
  );
}

// ============================================================
// KBD — keyboard shortcut styling
// ============================================================
export function Kbd({
  children,
  className,
}: {
  children: ReactNode;
  className?: string;
}) {
  return (
    <kbd
      className={cn(
        "inline-flex items-center rounded border border-border bg-muted px-1.5 py-0.5",
        "text-xs font-mono font-medium text-muted-foreground",
        "shadow-[0_1px_0_1px_rgba(0,0,0,0.04)]",
        className
      )}
    >
      {children}
    </kbd>
  );
}
```

### Errori Comuni da Evitare
- **Font flash (FOIT/FOUT)**: Usa `next/font` con `display: "swap"` e un fallback font con `adjustFontFallback`
- **Responsive text con clamp errato**: Non usare `clamp()` con viewport units sotto 320px
- **Prose dark mode**: Ricorda `dark:prose-invert` per il contenuto editoriale
- **Heading hierarchy**: Non saltare livelli heading (h1 -> h3) — usa `as` prop per separare semantica da stile

### Checklist di Verifica
- [ ] La scala tipografica è fluida e leggibile a tutti i breakpoint
- [ ] I font sono caricati con `next/font` per performance ottimale
- [ ] Il prose component gestisce correttamente dark mode
- [ ] I componenti di testo supportano la prop `as` per flessibilita semantica
- [ ] Il line-height è adeguato per ogni size (almeno 1.5 per body text)
- [ ] Il truncate multi-line funziona con `line-clamp`



---

## FORM-DESIGN-PATTERNS

### Panoramica
Pattern di design per form consistenti: input groups, form layouts, inline validation feedback, stepper forms multi-step con progress indicator integrato nel design system.

### Implementazione Completa

```typescript
// components/ui/form-field.tsx
"use client";

import { cn } from "@/lib/utils";
import { ReactNode, forwardRef, useId } from "react";
import { Label } from "@/components/ui/label";
import { Input } from "@/components/ui/input";
import { AlertCircle, CheckCircle2, Info } from "lucide-react";

// ============================================================
// FORM FIELD — label + input + error/hint wrapper
// ============================================================
interface FormFieldProps {
  label: string;
  name: string;
  error?: string;
  hint?: string;
  required?: boolean;
  children?: ReactNode;
  className?: string;
}

export function FormField({
  label,
  name,
  error,
  hint,
  required,
  children,
  className,
}: FormFieldProps) {
  const id = useId();
  const errorId = `${id}-error`;
  const hintId = `${id}-hint`;

  return (
    <div className={cn("space-y-2", className)}>
      <Label htmlFor={id} className="flex items-center gap-1">
        {label}
        {required && <span className="text-destructive">*</span>}
      </Label>
      <div className="relative">
        {children ?? (
          <Input
            id={id}
            name={name}
            aria-invalid={!!error}
            aria-describedby={error ? errorId : hint ? hintId : undefined}
            className={cn(
              error && "border-destructive focus-visible:ring-destructive"
            )}
          />
        )}
        {error && (
          <AlertCircle className="absolute right-3 top-1/2 h-4 w-4 -translate-y-1/2 text-destructive" />
        )}
      </div>
      {error && (
        <p id={errorId} className="text-sm text-destructive flex items-center gap-1">
          <AlertCircle className="h-3 w-3" />
          {error}
        </p>
      )}
      {!error && hint && (
        <p id={hintId} className="text-sm text-muted-foreground flex items-center gap-1">
          <Info className="h-3 w-3" />
          {hint}
        </p>
      )}
    </div>
  );
}

// ============================================================
// INPUT GROUP — prefix/suffix support
// ============================================================
interface InputGroupProps {
  prefix?: ReactNode;
  suffix?: ReactNode;
  children: ReactNode;
  className?: string;
}

export function InputGroup({
  prefix,
  suffix,
  children,
  className,
}: InputGroupProps) {
  return (
    <div className={cn("relative flex items-center", className)}>
      {prefix && (
        <div className="pointer-events-none absolute left-3 flex items-center text-muted-foreground">
          {prefix}
        </div>
      )}
      <div className={cn("w-full", prefix && "pl-10", suffix && "pr-10")}>
        {children}
      </div>
      {suffix && (
        <div className="pointer-events-none absolute right-3 flex items-center text-muted-foreground">
          {suffix}
        </div>
      )}
    </div>
  );
}

// ============================================================
// FORM SECTION — group related fields with heading
// ============================================================
interface FormSectionProps {
  title: string;
  description?: string;
  children: ReactNode;
  className?: string;
}

export function FormSection({
  title,
  description,
  children,
  className,
}: FormSectionProps) {
  return (
    <fieldset className={cn("space-y-4", className)}>
      <div className="space-y-1">
        <legend className="text-lg font-semibold">{title}</legend>
        {description && (
          <p className="text-sm text-muted-foreground">{description}</p>
        )}
      </div>
      <div className="space-y-4 border-l-2 border-border pl-4">
        {children}
      </div>
    </fieldset>
  );
}

// ============================================================
// MULTI-STEP FORM — stepper with progress indicator
// ============================================================
interface Step {
  id: string;
  title: string;
  description?: string;
}

interface StepperProps {
  steps: Step[];
  currentStep: number;
  className?: string;
}

export function Stepper({ steps, currentStep, className }: StepperProps) {
  return (
    <nav aria-label="Form progress" className={cn("w-full", className)}>
      <ol className="flex items-center justify-between">
        {steps.map((step, index) => {
          const isCompleted = index < currentStep;
          const isCurrent = index === currentStep;

          return (
            <li
              key={step.id}
              className={cn("flex items-center", index < steps.length - 1 && "flex-1")}
            >
              <div className="flex flex-col items-center gap-1">
                <div
                  className={cn(
                    "flex h-8 w-8 items-center justify-center rounded-full text-sm font-medium transition-colors",
                    isCompleted && "bg-primary text-primary-foreground",
                    isCurrent && "border-2 border-primary text-primary",
                    !isCompleted && !isCurrent && "border-2 border-muted text-muted-foreground"
                  )}
                  aria-current={isCurrent ? "step" : undefined}
                >
                  {isCompleted ? (
                    <CheckCircle2 className="h-5 w-5" />
                  ) : (
                    index + 1
                  )}
                </div>
                <span className={cn(
                  "text-xs font-medium hidden sm:block",
                  isCurrent ? "text-foreground" : "text-muted-foreground"
                )}>
                  {step.title}
                </span>
              </div>
              {index < steps.length - 1 && (
                <div
                  className={cn(
                    "mx-2 h-0.5 flex-1 transition-colors",
                    isCompleted ? "bg-primary" : "bg-muted"
                  )}
                />
              )}
            </li>
          );
        })}
      </ol>
    </nav>
  );
}
```

### Errori Comuni da Evitare
- **aria-invalid mancante**: Aggiungi sempre `aria-invalid` e `aria-describedby` per gli errori
- **Label non collegata**: Usa `htmlFor` con `id` matching sull'input
- **Focus trap in stepper**: Ogni step deve essere navigabile con Tab
- **Error message non screen-reader friendly**: Usa `role="alert"` per errori live

### Checklist di Verifica
- [ ] Ogni campo ha Label, hint e error message slot
- [ ] I campi required sono indicati visivamente e con `aria-required`
- [ ] Il multi-step form preserva lo stato tra gli step
- [ ] Il focus si sposta correttamente tra gli step
- [ ] Le validazioni inline non bloccano la navigazione tra step
