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