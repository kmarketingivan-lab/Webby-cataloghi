# CATALOGO-DESIGN-TOKEN-SYSTEM

CATALOGO-DESIGN-TOKEN-SYSTEM per Next.js 14 + Tailwind CSS
§ SEMANTIC COLOR TOKENS
typescript
// tokens/semantic-colors.ts

export const semanticColors = {
  // Surface colors
  surface: {
    default: 'hsl(var(--surface-default))',
    muted: 'hsl(var(--surface-muted))',
    subtle: 'hsl(var(--surface-subtle))',
    inverse: 'hsl(var(--surface-inverse))',
  },
  
  // Text colors
  text: {
    default: 'hsl(var(--text-default))',
    muted: 'hsl(var(--text-muted))',
    subtle: 'hsl(var(--text-subtle))',
    inverse: 'hsl(var(--text-inverse))',
    onColor: 'hsl(var(--text-on-color))',
  },
  
  // Border colors
  border: {
    default: 'hsl(var(--border-default))',
    muted: 'hsl(var(--border-muted))',
    strong: 'hsl(var(--border-strong))',
  },
  
  // State colors
  state: {
    hover: 'hsl(var(--state-hover))',
    active: 'hsl(var(--state-active))',
    focus: 'hsl(var(--state-focus))',
    disabled: 'hsl(var(--state-disabled))',
  },
  
  // Feedback colors
  feedback: {
    success: {
      default: 'hsl(var(--feedback-success))',
      muted: 'hsl(var(--feedback-success-muted))',
      subtle: 'hsl(var(--feedback-success-subtle))',
    },
    warning: {
      default: 'hsl(var(--feedback-warning))',
      muted: 'hsl(var(--feedback-warning-muted))',
      subtle: 'hsl(var(--feedback-warning-subtle))',
    },
    error: {
      default: 'hsl(var(--feedback-error))',
      muted: 'hsl(var(--feedback-error-muted))',
      subtle: 'hsl(var(--feedback-error-subtle))',
    },
    info: {
      default: 'hsl(var(--feedback-info))',
      muted: 'hsl(var(--feedback-info-muted))',
      subtle: 'hsl(var(--feedback-info-subtle))',
    },
  },
} as const;
css
/* app/globals.css */
:root {
  /* Surface colors */
  --surface-default: 0 0% 100%;
  --surface-muted: 210 40% 98%;
  --surface-subtle: 210 40% 96%;
  --surface-inverse: 222 47% 11%;
  
  /* Text colors */
  --text-default: 222 47% 11%;
  --text-muted: 215 25% 27%;
  --text-subtle: 215 16% 47%;
  --text-inverse: 0 0% 100%;
  --text-on-color: 0 0% 100%;
  
  /* Border colors */
  --border-default: 214 32% 91%;
  --border-muted: 210 40% 96%;
  --border-strong: 215 25% 27%;
  
  /* State colors */
  --state-hover: 210 40% 96%;
  --state-active: 210 40% 94%;
  --state-focus: 214 95% 93%;
  --state-disabled: 220 14% 96%;
  
  /* Feedback colors */
  --feedback-success: 142 76% 36%;
  --feedback-success-muted: 142 72% 29%;
  --feedback-success-subtle: 142 70% 95%;
  --feedback-warning: 32 95% 44%;
  --feedback-warning-muted: 32 92% 50%;
  --feedback-warning-subtle: 43 96% 95%;
  --feedback-error: 0 84% 60%;
  --feedback-error-muted: 0 74% 54%;
  --feedback-error-subtle: 0 96% 95%;
  --feedback-info: 199 89% 48%;
  --feedback-info-muted: 199 89% 44%;
  --feedback-info-subtle: 199 90% 95%;
}

.dark {
  --surface-default: 222 47% 11%;
  --surface-muted: 217 33% 17%;
  --surface-subtle: 215 25% 27%;
  --surface-inverse: 0 0% 100%;
  
  --text-default: 210 40% 98%;
  --text-muted: 215 20% 65%;
  --text-subtle: 217 10% 64%;
  --text-inverse: 222 47% 11%;
  --text-on-color: 0 0% 100%;
  
  --border-default: 215 25% 27%;
  --border-muted: 217 33% 17%;
  --border-strong: 214 32% 91%;
  
  --state-hover: 217 33% 17%;
  --state-active: 217 33% 20%;
  --state-focus: 217 33% 22%;
  --state-disabled: 215 25% 27%;
}
§ COMPONENT-LEVEL TOKENS
typescript
// tokens/components.ts

export const componentTokens = {
  // Button tokens
  button: {
    primary: {
      background: 'hsl(var(--button-primary-bg))',
      text: 'hsl(var(--button-primary-text))',
      border: 'hsl(var(--button-primary-border))',
      hover: {
        background: 'hsl(var(--button-primary-bg-hover))',
        text: 'hsl(var(--button-primary-text-hover))',
        border: 'hsl(var(--button-primary-border-hover))',
      },
      active: {
        background: 'hsl(var(--button-primary-bg-active))',
        text: 'hsl(var(--button-primary-text-active))',
        border: 'hsl(var(--button-primary-border-active))',
      },
      disabled: {
        background: 'hsl(var(--button-primary-bg-disabled))',
        text: 'hsl(var(--button-primary-text-disabled))',
        border: 'hsl(var(--button-primary-border-disabled))',
      },
    },
    secondary: {
      background: 'hsl(var(--button-secondary-bg))',
      text: 'hsl(var(--button-secondary-text))',
      border: 'hsl(var(--button-secondary-border))',
      hover: {
        background: 'hsl(var(--button-secondary-bg-hover))',
        text: 'hsl(var(--button-secondary-text-hover))',
        border: 'hsl(var(--button-secondary-border-hover))',
      },
    },
    outline: {
      background: 'hsl(var(--button-outline-bg))',
      text: 'hsl(var(--button-outline-text))',
      border: 'hsl(var(--button-outline-border))',
      hover: {
        background: 'hsl(var(--button-outline-bg-hover))',
        text: 'hsl(var(--button-outline-text-hover))',
        border: 'hsl(var(--button-outline-border-hover))',
      },
    },
    ghost: {
      background: 'hsl(var(--button-ghost-bg))',
      text: 'hsl(var(--button-ghost-text))',
      border: 'hsl(var(--button-ghost-border))',
      hover: {
        background: 'hsl(var(--button-ghost-bg-hover))',
        text: 'hsl(var(--button-ghost-text-hover))',
        border: 'hsl(var(--button-ghost-border-hover))',
      },
    },
    destructive: {
      background: 'hsl(var(--button-destructive-bg))',
      text: 'hsl(var(--button-destructive-text))',
      border: 'hsl(var(--button-destructive-border))',
      hover: {
        background: 'hsl(var(--button-destructive-bg-hover))',
        text: 'hsl(var(--button-destructive-text-hover))',
        border: 'hsl(var(--button-destructive-border-hover))',
      },
    },
  },
  
  // Input tokens
  input: {
    default: {
      background: 'hsl(var(--input-bg))',
      text: 'hsl(var(--input-text))',
      border: 'hsl(var(--input-border))',
      placeholder: 'hsl(var(--input-placeholder))',
    },
    focus: {
      background: 'hsl(var(--input-bg-focus))',
      text: 'hsl(var(--input-text-focus))',
      border: 'hsl(var(--input-border-focus))',
      ring: 'hsl(var(--input-ring-focus))',
    },
    error: {
      background: 'hsl(var(--input-bg-error))',
      text: 'hsl(var(--input-text-error))',
      border: 'hsl(var(--input-border-error))',
      ring: 'hsl(var(--input-ring-error))',
    },
    disabled: {
      background: 'hsl(var(--input-bg-disabled))',
      text: 'hsl(var(--input-text-disabled))',
      border: 'hsl(var(--input-border-disabled))',
      placeholder: 'hsl(var(--input-placeholder-disabled))',
    },
  },
  
  // Card tokens
  card: {
    default: {
      background: 'hsl(var(--card-bg))',
      text: 'hsl(var(--card-text))',
      border: 'hsl(var(--card-border))',
      shadow: 'var(--card-shadow)',
    },
    interactive: {
      background: 'hsl(var(--card-bg-interactive))',
      text: 'hsl(var(--card-text-interactive))',
      border: 'hsl(var(--card-border-interactive))',
      shadow: 'var(--card-shadow-interactive)',
      hover: {
        background: 'hsl(var(--card-bg-interactive-hover))',
        text: 'hsl(var(--card-text-interactive-hover))',
        border: 'hsl(var(--card-border-interactive-hover))',
        shadow: 'var(--card-shadow-interactive-hover)',
      },
    },
    selected: {
      background: 'hsl(var(--card-bg-selected))',
      text: 'hsl(var(--card-text-selected))',
      border: 'hsl(var(--card-border-selected))',
      shadow: 'var(--card-shadow-selected)',
    },
  },
  
  // Badge tokens
  badge: {
    default: {
      background: 'hsl(var(--badge-bg))',
      text: 'hsl(var(--badge-text))',
      border: 'hsl(var(--badge-border))',
    },
    primary: {
      background: 'hsl(var(--badge-primary-bg))',
      text: 'hsl(var(--badge-primary-text))',
      border: 'hsl(var(--badge-primary-border))',
    },
    success: {
      background: 'hsl(var(--badge-success-bg))',
      text: 'hsl(var(--badge-success-text))',
      border: 'hsl(var(--badge-success-border))',
    },
    warning: {
      background: 'hsl(var(--badge-warning-bg))',
      text: 'hsl(var(--badge-warning-text))',
      border: 'hsl(var(--badge-warning-border))',
    },
    error: {
      background: 'hsl(var(--badge-error-bg))',
      text: 'hsl(var(--badge-error-text))',
      border: 'hsl(var(--badge-error-border))',
    },
    outline: {
      background: 'hsl(var(--badge-outline-bg))',
      text: 'hsl(var(--badge-outline-text))',
      border: 'hsl(var(--badge-outline-border))',
    },
  },
} as const;
§ ANIMATION TOKENS
typescript
// tokens/animations.ts

export const animationTokens = {
  // Duration
  duration: {
    instant: '0ms',
    fast: '150ms',
    normal: '250ms',
    slow: '350ms',
    slower: '500ms',
    slowest: '700ms',
  },
  
  // Easing functions
  easing: {
    linear: 'linear',
    easeIn: 'cubic-bezier(0.4, 0, 1, 1)',
    easeOut: 'cubic-bezier(0, 0, 0.2, 1)',
    easeInOut: 'cubic-bezier(0.4, 0, 0.2, 1)',
    spring: 'cubic-bezier(0.175, 0.885, 0.32, 1.275)',
    springBouncy: 'cubic-bezier(0.68, -0.55, 0.265, 1.55)',
  },
  
  // Scale values
  scale: {
    xs: 'scale(0.95)',
    sm: 'scale(0.98)',
    md: 'scale(1.02)',
    lg: 'scale(1.05)',
    xl: 'scale(1.1)',
  },
  
  // Opacity values
  opacity: {
    disabled: '0.5',
    hover: '0.8',
    active: '0.7',
    focus: '0.9',
    subtle: '0.6',
    invisible: '0',
    visible: '1',
  },
  
  // Keyframes
  keyframes: {
    fadeIn: {
      '0%': { opacity: '0' },
      '100%': { opacity: '1' },
    },
    fadeOut: {
      '0%': { opacity: '1' },
      '100%': { opacity: '0' },
    },
    slideUp: {
      '0%': { transform: 'translateY(10px)', opacity: '0' },
      '100%': { transform: 'translateY(0)', opacity: '1' },
    },
    slideDown: {
      '0%': { transform: 'translateY(-10px)', opacity: '0' },
      '100%': { transform: 'translateY(0)', opacity: '1' },
    },
    slideInLeft: {
      '0%': { transform: 'translateX(-10px)', opacity: '0' },
      '100%': { transform: 'translateX(0)', opacity: '1' },
    },
    slideInRight: {
      '0%': { transform: 'translateX(10px)', opacity: '0' },
      '100%': { transform: 'translateX(0)', opacity: '1' },
    },
    scaleIn: {
      '0%': { transform: 'scale(0.95)', opacity: '0' },
      '100%': { transform: 'scale(1)', opacity: '1' },
    },
    scaleOut: {
      '0%': { transform: 'scale(1)', opacity: '1' },
      '100%': { transform: 'scale(0.95)', opacity: '0' },
    },
    spin: {
      '0%': { transform: 'rotate(0deg)' },
      '100%': { transform: 'rotate(360deg)' },
    },
    ping: {
      '75%, 100%': {
        transform: 'scale(2)',
        opacity: '0',
      },
    },
    pulse: {
      '0%, 100%': { opacity: '1' },
      '50%': { opacity: '0.5' },
    },
    bounce: {
      '0%, 100%': {
        transform: 'translateY(-25%)',
        animationTimingFunction: 'cubic-bezier(0.8,0,1,1)',
      },
      '50%': {
        transform: 'none',
        animationTimingFunction: 'cubic-bezier(0,0,0.2,1)',
      },
    },
  },
  
  // Animation compositions
  animations: {
    fadeIn: 'fadeIn 150ms var(--ease-out)',
    fadeOut: 'fadeOut 150ms var(--ease-in)',
    slideUp: 'slideUp 200ms var(--ease-out)',
    slideDown: 'slideDown 200ms var(--ease-out)',
    slideInLeft: 'slideInLeft 200ms var(--ease-out)',
    slideInRight: 'slideInRight 200ms var(--ease-out)',
    scaleIn: 'scaleIn 200ms var(--ease-out)',
    scaleOut: 'scaleOut 200ms var(--ease-in)',
    spin: 'spin 1s linear infinite',
    ping: 'ping 1s cubic-bezier(0, 0, 0.2, 1) infinite',
    pulse: 'pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite',
    bounce: 'bounce 1s infinite',
  },
} as const;
§ SHADOW TOKENS
typescript
// tokens/shadows.ts

export const shadowTokens = {
  // Elevation levels
  elevation: {
    1: {
      light: '0 1px 2px 0 rgb(0 0 0 / 0.05)',
      dark: '0 1px 2px 0 rgb(0 0 0 / 0.2)',
    },
    2: {
      light: '0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1)',
      dark: '0 1px 3px 0 rgb(0 0 0 / 0.3), 0 1px 2px -1px rgb(0 0 0 / 0.3)',
    },
    3: {
      light: '0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1)',
      dark: '0 4px 6px -1px rgb(0 0 0 / 0.3), 0 2px 4px -2px rgb(0 0 0 / 0.3)',
    },
    4: {
      light: '0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)',
      dark: '0 10px 15px -3px rgb(0 0 0 / 0.3), 0 4px 6px -4px rgb(0 0 0 / 0.3)',
    },
    5: {
      light: '0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)',
      dark: '0 20px 25px -5px rgb(0 0 0 / 0.3), 0 8px 10px -6px rgb(0 0 0 / 0.3)',
    },
  },
  
  // Inner shadows
  inner: {
    sm: 'inset 0 1px 2px 0 rgb(0 0 0 / 0.05)',
    md: 'inset 0 2px 4px 0 rgb(0 0 0 / 0.05)',
    lg: 'inset 0 4px 6px 0 rgb(0 0 0 / 0.1)',
    xl: 'inset 0 8px 10px 0 rgb(0 0 0 / 0.1)',
  },
  
  // Focus rings
  focusRing: {
    default: {
      light: '0 0 0 3px hsl(var(--state-focus) / 0.5)',
      dark: '0 0 0 3px hsl(var(--state-focus) / 0.7)',
    },
    error: {
      light: '0 0 0 3px hsl(var(--feedback-error) / 0.5)',
      dark: '0 0 0 3px hsl(var(--feedback-error) / 0.7)',
    },
    success: {
      light: '0 0 0 3px hsl(var(--feedback-success) / 0.5)',
      dark: '0 0 0 3px hsl(var(--feedback-success) / 0.7)',
    },
    subtle: {
      light: '0 0 0 1px hsl(var(--state-focus) / 0.3)',
      dark: '0 0 0 1px hsl(var(--state-focus) / 0.5)',
    },
  },
  
  // Glow effects
  glow: {
    xs: {
      light: '0 0 4px hsl(var(--feedback-info) / 0.2)',
      dark: '0 0 6px hsl(var(--feedback-info) / 0.4)',
    },
    sm: {
      light: '0 0 8px hsl(var(--feedback-info) / 0.3)',
      dark: '0 0 12px hsl(var(--feedback-info) / 0.5)',
    },
    md: {
      light: '0 0 16px hsl(var(--feedback-info) / 0.3)',
      dark: '0 0 24px hsl(var(--feedback-info) / 0.5)',
    },
    lg: {
      light: '0 0 24px hsl(var(--feedback-info) / 0.4)',
      dark: '0 0 32px hsl(var(--feedback-info) / 0.6)',
    },
    xl: {
      light: '0 0 32px hsl(var(--feedback-info) / 0.5)',
      dark: '0 0 48px hsl(var(--feedback-info) / 0.7)',
    },
  },
  
  // Special shadows
  special: {
    card: '0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1)',
    dropdown: '0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)',
    modal: '0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)',
    toast: '0 25px 50px -12px rgb(0 0 0 / 0.25)',
  },
} as const;
§ TAILWIND INTEGRATION
typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss';

const config: Config = {
  darkMode: ['class'],
  content: [
    './pages/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './app/**/*.{ts,tsx}',
    './src/**/*.{ts,tsx}',
  ],
  theme: {
    container: {
      center: true,
      padding: '2rem',
      screens: {
        '2xl': '1400px',
      },
    },
    extend: {
      // Color system
      colors: {
        border: 'hsl(var(--border-default))',
        input: 'hsl(var(--input-border))',
        ring: 'hsl(var(--state-focus))',
        background: 'hsl(var(--surface-default))',
        foreground: 'hsl(var(--text-default))',
        
        // Semantic colors
        surface: {
          DEFAULT: 'hsl(var(--surface-default))',
          muted: 'hsl(var(--surface-muted))',
          subtle: 'hsl(var(--surface-subtle))',
          inverse: 'hsl(var(--surface-inverse))',
        },
        text: {
          DEFAULT: 'hsl(var(--text-default))',
          muted: 'hsl(var(--text-muted))',
          subtle: 'hsl(var(--text-subtle))',
          inverse: 'hsl(var(--text-inverse))',
          onColor: 'hsl(var(--text-on-color))',
        },
        state: {
          hover: 'hsl(var(--state-hover))',
          active: 'hsl(var(--state-active))',
          focus: 'hsl(var(--state-focus))',
          disabled: 'hsl(var(--state-disabled))',
        },
        
        // Feedback colors
        success: {
          DEFAULT: 'hsl(var(--feedback-success))',
          muted: 'hsl(var(--feedback-success-muted))',
          subtle: 'hsl(var(--feedback-success-subtle))',
        },
        warning: {
          DEFAULT: 'hsl(var(--feedback-warning))',
          muted: 'hsl(var(--feedback-warning-muted))',
          subtle: 'hsl(var(--feedback-warning-subtle))',
        },
        error: {
          DEFAULT: 'hsl(var(--feedback-error))',
          muted: 'hsl(var(--feedback-error-muted))',
          subtle: 'hsl(var(--feedback-error-subtle))',
        },
        info: {
          DEFAULT: 'hsl(var(--feedback-info))',
          muted: 'hsl(var(--feedback-info-muted))',
          subtle: 'hsl(var(--feedback-info-subtle))',
        },
        
        // Component colors
        primary: {
          DEFAULT: 'hsl(var(--button-primary-bg))',
          foreground: 'hsl(var(--button-primary-text))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--button-secondary-bg))',
          foreground: 'hsl(var(--button-secondary-text))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--button-destructive-bg))',
          foreground: 'hsl(var(--button-destructive-text))',
        },
        card: {
          DEFAULT: 'hsl(var(--card-bg))',
          foreground: 'hsl(var(--card-text))',
        },
      },
      
      // Border radius
      borderRadius: {
        lg: 'var(--radius-lg)',
        md: 'var(--radius-md)',
        sm: 'var(--radius-sm)',
        xs: 'var(--radius-xs)',
      },
      
      // Animation
      animation: {
        'fade-in': 'fadeIn 150ms var(--ease-out)',
        'fade-out': 'fadeOut 150ms var(--ease-in)',
        'slide-up': 'slideUp 200ms var(--ease-out)',
        'slide-down': 'slideDown 200ms var(--ease-out)',
        'slide-in-left': 'slideInLeft 200ms var(--ease-out)',
        'slide-in-right': 'slideInRight 200ms var(--ease-out)',
        'scale-in': 'scaleIn 200ms var(--ease-out)',
        'scale-out': 'scaleOut 200ms var(--ease-in)',
        spin: 'spin 1s linear infinite',
        ping: 'ping 1s cubic-bezier(0, 0, 0.2, 1) infinite',
        pulse: 'pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite',
        bounce: 'bounce 1s infinite',
      },
      
      // Keyframes
      keyframes: {
        fadeIn: {
          '0%': { opacity: '0' },
          '100%': { opacity: '1' },
        },
        fadeOut: {
          '0%': { opacity: '1' },
          '100%': { opacity: '0' },
        },
        slideUp: {
          '0%': { transform: 'translateY(10px)', opacity: '0' },
          '100%': { transform: 'translateY(0)', opacity: '1' },
        },
        slideDown: {
          '0%': { transform: 'translateY(-10px)', opacity: '0' },
          '100%': { transform: 'translateY(0)', opacity: '1' },
        },
        slideInLeft: {
          '0%': { transform: 'translateX(-10px)', opacity: '0' },
          '100%': { transform: 'translateX(0)', opacity: '1' },
        },
        slideInRight: {
          '0%': { transform: 'translateX(10px)', opacity: '0' },
          '100%': { transform: 'translateX(0)', opacity: '1' },
        },
        scaleIn: {
          '0%': { transform: 'scale(0.95)', opacity: '0' },
          '100%': { transform: 'scale(1)', opacity: '1' },
        },
        scaleOut: {
          '0%': { transform: 'scale(1)', opacity: '1' },
          '100%': { transform: 'scale(0.95)', opacity: '0' },
        },
        spin: {
          '0%': { transform: 'rotate(0deg)' },
          '100%': { transform: 'rotate(360deg)' },
        },
        ping: {
          '75%, 100%': {
            transform: 'scale(2)',
            opacity: '0',
          },
        },
        pulse: {
          '0%, 100%': { opacity: '1' },
          '50%': { opacity: '0.5' },
        },
        bounce: {
          '0%, 100%': {
            transform: 'translateY(-25%)',
            animationTimingFunction: 'cubic-bezier(0.8,0,1,1)',
          },
          '50%': {
            transform: 'none',
            animationTimingFunction: 'cubic-bezier(0,0,0.2,1)',
          },
        },
      },
      
      // Box shadow
      boxShadow: {
        // Elevation
        'elevation-1': '0 1px 2px 0 rgb(0 0 0 / 0.05)',
        'elevation-2': '0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1)',
        'elevation-3': '0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1)',
        'elevation-4': '0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)',
        'elevation-5': '0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)',
        
        // Inner shadows
        'inner-sm': 'inset 0 1px 2px 0 rgb(0 0 0 / 0.05)',
        'inner-md': 'inset 0 2px 4px 0 rgb(0 0 0 / 0.05)',
        'inner-lg': 'inset 0 4px 6px 0 rgb(0 0 0 / 0.1)',
        
        // Focus rings
        'focus-ring': '0 0 0 3px hsl(var(--state-focus) / 0.5)',
        'focus-ring-error': '0 0 0 3px hsl(var(--feedback-error) / 0.5)',
        'focus-ring-success': '0 0 0 3px hsl(var(--feedback-success) / 0.5)',
        'focus-ring-subtle': '0 0 0 1px hsl(var(--state-focus) / 0.3)',
        
        // Special
        card: '0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1)',
        dropdown: '0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)',
        modal: '0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)',
        toast: '0 25px 50px -12px rgb(0 0 0 / 0.25)',
      },
      
      // Opacity
      opacity: {
        disabled: '0.5',
        hover: '0.8',
        active: '0.7',
        focus: '0.9',
        subtle: '0.6',
      },
      
      // Transition
      transitionDuration: {
        instant: '0ms',
        fast: '150ms',
        normal: '250ms',
        slow: '350ms',
        slower: '500ms',
        slowest: '700ms',
      },
      transitionTimingFunction: {
        'ease-in': 'cubic-bezier(0.4, 0, 1, 1)',
        'ease-out': 'cubic-bezier(0, 0, 0.2, 1)',
        'ease-in-out': 'cubic-bezier(0.4, 0, 0.2, 1)',
        spring: 'cubic-bezier(0.175, 0.885, 0.32, 1.275)',
        'spring-bouncy': 'cubic-bezier(0.68, -0.55, 0.265, 1.55)',
      },
    },
  },
  plugins: [
    require('tailwindcss-animate'),
    // Custom plugin for design tokens
    function({ addBase, addComponents, theme }: any) {
      addBase({
        ':root': {
          // Border radius
          '--radius-xs': '0.25rem',
          '--radius-sm': '0.375rem',
          '--radius-md': '0.5rem',
          '--radius-lg': '0.75rem',
          
          // Animation easing
          '--ease-in': theme('transitionTimingFunction.ease-in'),
          '--ease-out': theme('transitionTimingFunction.ease-out'),
          '--ease-in-out': theme('transitionTimingFunction.ease-in-out'),
          '--spring': theme('transitionTimingFunction.spring'),
        },
      });
      
      addComponents({
        // Focus visible utility
        '.focus-visible': {
          'outline': '2px solid hsl(var(--state-focus))',
          'outline-offset': '2px',
        },
        
        // Reduced motion utility
        '@media (prefers-reduced-motion: reduce)': {
          '.motion-reduce': {
            'animation-duration': '1ms !important',
            'animation-iteration-count': '1 !important',
            'transition-duration': '1ms !important',
          },
        },
        
        // High contrast mode
        '@media (prefers-contrast: high)': {
          '.high-contrast': {
            'border-width': '2px',
            '--border-default': '222 47% 11%',
            '--text-default': '222 47% 3%',
          },
        },
      });
    },
  ],
};

export default config;
css
/* app/globals.css - Extended */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    /* Border radius */
    --radius-xs: 0.25rem;
    --

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-14-DESIGN-TOKEN-SYSTEM
Prompt ID: 14 / 48
Parte: 2
Exported: 2026-02-06T13:36:16.439Z
Characters: 4807
════════════════════════════════════════════════════════════

le contrast check for main text
          const textElements = document.querySelectorAll('p, h1, h2, h3, h4, h5, h6');
          let contrastIssues = 0;
          
          textElements.forEach(el => {
            const style = window.getComputedStyle(el);
            const color = style.color;
            const bgColor = style.backgroundColor;
            
            // Convert rgb() to hex for checking
            const rgbToHex = (rgb: string) => {
              const match = rgb.match(/^rgb\((\d+),\s*(\d+),\s*(\d+)\)$/);
              if (!match) return '#000000';
              
              const r = parseInt(match[1]);
              const g = parseInt(match[2]);
              const b = parseInt(match[3]);
              
              return '#' + ((1 << 24) + (r << 16) + (g << 8) + b).toString(16).slice(1);
            };
            
            try {
              const contrast = checkColorContrast(
                rgbToHex(color),
                rgbToHex(bgColor)
              );
              
              if (!contrast.passesAA) {
                contrastIssues++;
              }
            } catch (error) {
              // Skip if colors can't be parsed
            }
          });
          
          return contrastIssues > 0
            ? `Found ${contrastIssues} elements with insufficient color contrast`
            : null;
        },
      },
    ];
    
    const runChecks = () => {
      const newIssues: string[] = [];
      checks.forEach(check => {
        const issue = check.check();
        if (issue) {
          newIssues.push(`${check.name}: ${issue}`);
        }
      });
      setIssues(newIssues);
    };
    
    // Run checks after a delay to allow content to load
    const timeoutId = setTimeout(runChecks, 1000);
    
    // Also run checks on mutations
    const observer = new MutationObserver(runChecks);
    observer.observe(document.body, {
      childList: true,
      subtree: true,
    });
    
    return () => {
      clearTimeout(timeoutId);
      observer.disconnect();
    };
  }, []);
  
  if (process.env.NODE_ENV === 'development' && issues.length > 0) {
    console.warn('Accessibility issues found:', issues);
  }
  
  return <>{children}</>;
}
CONFIGURAZIONE FINALE
typescript
// package.json scripts aggiuntivi
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "tokens:build": "style-dictionary build",
    "tokens:watch": "style-dictionary watch",
    "tokens:sync-figma": "tsx scripts/sync-figma.ts",
    "tokens:validate": "tsx scripts/validate-tokens.ts",
    "tokens:export": "tsx scripts/export-tokens.ts"
  },
  "devDependencies": {
    "style-dictionary": "^3.9.0",
    "tsx": "^4.7.0",
    "tailwindcss-animate": "^1.0.7",
    "figma-js": "^1.16.0",
    "chroma-js": "^2.4.2"
  }
}
typescript
// scripts/validate-tokens.ts
import { checkColorContrast } from '@/tokens/accessibility';
import { semanticColors } from '@/tokens/semantic-colors';

function validateTokenContrast() {
  console.log('🔍 Validating color contrast ratios...\n');
  
  const tests = [
    {
      name: 'Text on Surface Default',
      foreground: semanticColors.text.default,
      background: semanticColors.surface.default,
      requiredRatio: 4.5,
    },
    {
      name: 'Text Muted on Surface Default',
      foreground: semanticColors.text.muted,
      background: semanticColors.surface.default,
      requiredRatio: 4.5,
    },
    {
      name: 'Primary Button Text on Background',
      foreground: 'hsl(var(--button-primary-text))',
      background: 'hsl(var(--button-primary-bg))',
      requiredRatio: 4.5,
    },
  ];
  
  let allPassed = true;
  
  tests.forEach(test => {
    try {
      const result = checkColorContrast(test.foreground, test.background);
      const passed = result.ratio >= test.requiredRatio;
      
      console.log(
        `${passed ? '✅' : '❌'} ${test.name}:`,
        `Ratio: ${result.ratio.toFixed(2)}`,
        `(Required: ${test.requiredRatio})`,
        passed ? 'PASS' : 'FAIL'
      );
      
      if (!passed) {
        allPassed = false;
      }
    } catch (error) {
      console.log(`⚠️  ${test.name}: Could not validate`);
    }
  });
  
  console.log(`\n${allPassed ? '🎉 All tests passed!' : '❌ Some tests failed.'}`);
  process.exit(allPassed ? 0 : 1);
}

validateTokenContrast();
typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  images: {
    formats: ['image/avif', 'image/webp'],
  },
  // For design token optimization
  experimental: {
    optimizeCss: true,
  },
  // Environment variables for Figma sync
  env: {
    FIGMA_TOKEN: process.env.FIGMA_TOKEN,
    FIGMA

## § ADVANCED PATTERNS: DESIGN TOKEN SYSTEM

### Server Actions con Validazione

```typescript
// app/actions/design-token-system.ts
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


### DESIGN TOKEN SYSTEM - Utility Helper #293

```typescript
// lib/utils/design-token-system-helper-293.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #47

```typescript
// lib/utils/design-token-system-helper-47.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #534

```typescript
// lib/utils/design-token-system-helper-534.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #584

```typescript
// lib/utils/design-token-system-helper-584.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #393

```typescript
// lib/utils/design-token-system-helper-393.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #972

```typescript
// lib/utils/design-token-system-helper-972.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #144

```typescript
// lib/utils/design-token-system-helper-144.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #836

```typescript
// lib/utils/design-token-system-helper-836.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #524

```typescript
// lib/utils/design-token-system-helper-524.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #559

```typescript
// lib/utils/design-token-system-helper-559.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #351

```typescript
// lib/utils/design-token-system-helper-351.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #792

```typescript
// lib/utils/design-token-system-helper-792.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #259

```typescript
// lib/utils/design-token-system-helper-259.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #793

```typescript
// lib/utils/design-token-system-helper-793.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #628

```typescript
// lib/utils/design-token-system-helper-628.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #588

```typescript
// lib/utils/design-token-system-helper-588.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #577

```typescript
// lib/utils/design-token-system-helper-577.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DESIGN TOKEN SYSTEM - Utility Helper #249

```typescript
// lib/utils/design-token-system-helper-249.ts
import { z } from "zod";

interface DESIGNTOKENSYSTEMConfig {
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

export class DESIGNTOKENSYSTEMProcessor<TInput, TOutput> {
  private config: DESIGNTOKENSYSTEMConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DESIGNTOKENSYSTEMConfig> = {}) {
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

  getConfig(): Readonly<DESIGNTOKENSYSTEMConfig> {
    return Object.freeze({ ...this.config });
  }
}
```
