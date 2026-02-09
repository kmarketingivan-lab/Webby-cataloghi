# ============================================================================
# CATALOGO DESIGN TOKEN SYSTEM v1.0 - SPECIFICA COMPLETA
# ============================================================================
# TIPO: SPECIFICA DETERMINISTICA
# STANDARD: W3C DTCG 2025.10 (Design Tokens Format Module)
# TARGET: Generazione automatica UI/UX per piattaforme web
# AFFIDABILITÀ TARGET: 95%
# DATA: Gennaio 2026
# ============================================================================

"""
ISTRUZIONI PER IL MODELLO AI:

1. Questo documento è la SPECIFICA UFFICIALE per i Design Tokens
2. OGNI token DEVE seguire ESATTAMENTE la struttura W3C DTCG 2025.10
3. NON inventare tipi di token non definiti qui
4. USA SEMPRE il formato JSON con prefisso $ ($value, $type, $description)
5. I colori DEVONO usare il formato colorSpace + components (NON hex diretto)
6. Le dimensioni DEVONO essere oggetti con value + unit
7. Per alias usa la sintassi {path.to.token}
8. SEGUI l'architettura a 3 livelli: Primitive → Semantic → Component
"""

# ============================================================================
# SEZIONE 1: STRUTTURA FILE E FORMATO
# ============================================================================

## 1.1 File Format

- **Formato**: JSON
- **Estensione**: `.tokens` o `.tokens.json`
- **Media Type**: `application/design-tokens+json`
- **Encoding**: UTF-8
- **Schema**: `https://www.designtokens.org/schemas/2025.10/format.json`

## 1.2 Proprietà Riservate (Prefisso $)

| Proprietà | Obbligatoria | Descrizione |
|-----------|--------------|-------------|
| `$value` | SÌ | Valore del token |
| `$type` | NO* | Tipo del token (ereditabile) |
| `$description` | NO | Descrizione testuale |
| `$deprecated` | NO | true/false/string con motivo |
| `$extensions` | NO | Dati vendor-specific |

*Se non specificato, DEVE essere ereditato dal gruppo padre o determinato dall'alias.

## 1.3 Caratteri Proibiti nei Nomi

- `{` e `}` (usati per alias)
- `.` (usato per path)
- Nomi che iniziano con `$`

## 1.4 Esempio Struttura Base

```json
{
  "$schema": "https://www.designtokens.org/schemas/2025.10/format.json",
  "color": {
    "$type": "color",
    "primitive": { ... },
    "semantic": { ... }
  },
  "spacing": {
    "$type": "dimension",
    "primitive": { ... },
    "semantic": { ... }
  }
}
```

# ============================================================================
# SEZIONE 2: TIPI PRIMITIVI
# ============================================================================

## 2.1 COLOR

Rappresenta un colore. Supporta spazi colore moderni (sRGB, Display P3, Oklch, etc).

### Struttura OBBLIGATORIA:
```json
{
  "token-name": {
    "$type": "color",
    "$value": {
      "colorSpace": "srgb",
      "components": [0.2, 0.4, 0.8],
      "alpha": 1,
      "hex": "#3366cc"
    }
  }
}
```

### Campi:
| Campo | Tipo | Obbligatorio | Descrizione |
|-------|------|--------------|-------------|
| colorSpace | string | SÌ | "srgb", "display-p3", "oklch", etc |
| components | array[3] | SÌ | Valori 0-1 per sRGB, specifici per altri |
| alpha | number | NO | Opacità 0-1 (default: 1) |
| hex | string | NO | Fallback esadecimale per compatibilità |

### Spazi Colore Supportati:
- `srgb` - Standard RGB (components: [r, g, b] 0-1)
- `display-p3` - Wide gamut (components: [r, g, b] 0-1)
- `oklch` - Perceptually uniform (components: [l, c, h] 0-1, 0-0.4, 0-360)

## 2.2 DIMENSION

Rappresenta una misura (spacing, sizing, border-width, etc).

### Struttura OBBLIGATORIA:
```json
{
  "spacing-md": {
    "$type": "dimension",
    "$value": {
      "value": 16,
      "unit": "px"
    }
  }
}
```

### Campi:
| Campo | Tipo | Obbligatorio | Valori Ammessi |
|-------|------|--------------|----------------|
| value | number | SÌ | Qualsiasi numero |
| unit | string | SÌ | "px" o "rem" |

### Note:
- `px` = pixel ideale (convertito in dp su Android, pt su iOS)
- `rem` = relativo al font-size base (16px default)

## 2.3 FONT FAMILY

Rappresenta una famiglia di font.

### Struttura:
```json
{
  "font-primary": {
    "$type": "fontFamily",
    "$value": ["Inter", "Helvetica Neue", "sans-serif"]
  }
}
```

### Formati Ammessi:
- Stringa singola: `"Inter"`
- Array di fallback: `["Inter", "Arial", "sans-serif"]`

## 2.4 FONT WEIGHT

Rappresenta il peso del font.

### Struttura:
```json
{
  "weight-bold": {
    "$type": "fontWeight",
    "$value": 700
  }
}
```

### Valori Ammessi:
| Numerico | Alias String |
|----------|--------------|
| 100 | "thin", "hairline" |
| 200 | "extra-light", "ultra-light" |
| 300 | "light" |
| 400 | "normal", "regular", "book" |
| 500 | "medium" |
| 600 | "semi-bold", "demi-bold" |
| 700 | "bold" |
| 800 | "extra-bold", "ultra-bold" |
| 900 | "black", "heavy" |
| 950 | "extra-black", "ultra-black" |

## 2.5 DURATION

Rappresenta una durata temporale (animazioni).

### Struttura OBBLIGATORIA:
```json
{
  "duration-fast": {
    "$type": "duration",
    "$value": {
      "value": 150,
      "unit": "ms"
    }
  }
}
```

### Unità Ammesse:
- `ms` - millisecondi
- `s` - secondi

## 2.6 CUBIC BÉZIER

Rappresenta una curva di easing per animazioni.

### Struttura:
```json
{
  "ease-out": {
    "$type": "cubicBezier",
    "$value": [0, 0, 0.2, 1]
  }
}
```

### Formato:
Array di 4 numeri: `[P1x, P1y, P2x, P2y]`
- P1x, P2x: range [0, 1]
- P1y, P2y: range [-∞, ∞]

## 2.7 NUMBER

Rappresenta un numero generico (line-height, aspect-ratio, etc).

### Struttura:
```json
{
  "line-height-normal": {
    "$type": "number",
    "$value": 1.5
  }
}
```

# ============================================================================
# SEZIONE 3: TIPI COMPOSITI
# ============================================================================

## 3.1 TYPOGRAPHY

Combina proprietà tipografiche correlate.

### Struttura OBBLIGATORIA:
```json
{
  "heading-1": {
    "$type": "typography",
    "$value": {
      "fontFamily": ["Inter", "sans-serif"],
      "fontSize": { "value": 32, "unit": "px" },
      "fontWeight": 700,
      "letterSpacing": { "value": -0.5, "unit": "px" },
      "lineHeight": 1.2
    }
  }
}
```

### Campi:
| Campo | Tipo | Obbligatorio |
|-------|------|--------------|
| fontFamily | fontFamily | SÌ |
| fontSize | dimension | SÌ |
| fontWeight | fontWeight | SÌ |
| letterSpacing | dimension | NO |
| lineHeight | number | NO |

## 3.2 SHADOW

Rappresenta un'ombra (box-shadow, drop-shadow).

### Struttura Singola:
```json
{
  "shadow-sm": {
    "$type": "shadow",
    "$value": {
      "color": {
        "colorSpace": "srgb",
        "components": [0, 0, 0],
        "alpha": 0.1
      },
      "offsetX": { "value": 0, "unit": "px" },
      "offsetY": { "value": 1, "unit": "px" },
      "blur": { "value": 2, "unit": "px" },
      "spread": { "value": 0, "unit": "px" }
    }
  }
}
```

### Struttura Multipla (Layered):
```json
{
  "shadow-lg": {
    "$type": "shadow",
    "$value": [
      {
        "color": { "colorSpace": "srgb", "components": [0, 0, 0], "alpha": 0.1 },
        "offsetX": { "value": 0, "unit": "px" },
        "offsetY": { "value": 10, "unit": "px" },
        "blur": { "value": 15, "unit": "px" },
        "spread": { "value": -3, "unit": "px" }
      },
      {
        "color": { "colorSpace": "srgb", "components": [0, 0, 0], "alpha": 0.1 },
        "offsetX": { "value": 0, "unit": "px" },
        "offsetY": { "value": 4, "unit": "px" },
        "blur": { "value": 6, "unit": "px" },
        "spread": { "value": -4, "unit": "px" }
      }
    ]
  }
}
```

### Campi Shadow Object:
| Campo | Tipo | Obbligatorio |
|-------|------|--------------|
| color | color | SÌ |
| offsetX | dimension | SÌ |
| offsetY | dimension | SÌ |
| blur | dimension | SÌ |
| spread | dimension | SÌ |
| inset | boolean | NO (default: false) |

## 3.3 BORDER

Rappresenta uno stile di bordo.

### Struttura:
```json
{
  "border-default": {
    "$type": "border",
    "$value": {
      "color": {
        "colorSpace": "srgb",
        "components": [0.878, 0.878, 0.878]
      },
      "width": { "value": 1, "unit": "px" },
      "style": "solid"
    }
  }
}
```

### Campi:
| Campo | Tipo | Obbligatorio |
|-------|------|--------------|
| color | color | SÌ |
| width | dimension | SÌ |
| style | strokeStyle | SÌ |

## 3.4 STROKE STYLE

Può essere stringa o oggetto.

### Valori Stringa:
`"solid"`, `"dashed"`, `"dotted"`, `"double"`, `"groove"`, `"ridge"`, `"outset"`, `"inset"`

### Valore Oggetto (custom dash):
```json
{
  "style": {
    "dashArray": [
      { "value": 4, "unit": "px" },
      { "value": 2, "unit": "px" }
    ],
    "lineCap": "round"
  }
}
```

## 3.5 GRADIENT

Rappresenta un gradiente di colori.

### Struttura:
```json
{
  "gradient-primary": {
    "$type": "gradient",
    "$value": [
      {
        "color": { "colorSpace": "srgb", "components": [0.2, 0.4, 0.9] },
        "position": 0
      },
      {
        "color": { "colorSpace": "srgb", "components": [0.6, 0.2, 0.8] },
        "position": 1
      }
    ]
  }
}
```

### Campi Stop:
| Campo | Tipo | Obbligatorio |
|-------|------|--------------|
| color | color | SÌ |
| position | number | SÌ (0-1) |

## 3.6 TRANSITION

Rappresenta una transizione animata.

### Struttura:
```json
{
  "transition-default": {
    "$type": "transition",
    "$value": {
      "duration": { "value": 200, "unit": "ms" },
      "delay": { "value": 0, "unit": "ms" },
      "timingFunction": [0.4, 0, 0.2, 1]
    }
  }
}
```


# ============================================================================
# SEZIONE 4: GRUPPI E ALIAS
# ============================================================================

## 4.1 Gruppi

I gruppi organizzano i token in strutture gerarchiche.

### Struttura:
```json
{
  "color": {
    "$type": "color",
    "$description": "Palette colori del sistema",
    "primary": {
      "$value": { "colorSpace": "srgb", "components": [0.2, 0.4, 0.9] }
    },
    "secondary": {
      "$value": { "colorSpace": "srgb", "components": [0.6, 0.2, 0.8] }
    }
  }
}
```

### Ereditarietà $type:
Il `$type` definito su un gruppo viene ereditato da tutti i token figli.

## 4.2 Alias (Reference)

Un token può referenziare il valore di un altro token.

### Sintassi:
```json
{
  "color": {
    "brand": {
      "$type": "color",
      "$value": { "colorSpace": "srgb", "components": [0.2, 0.4, 0.9] }
    }
  },
  "semantic": {
    "primary": {
      "$type": "color",
      "$value": "{color.brand}"
    }
  }
}
```

### Regole:
- Usa `{path.to.token}` per referenziare
- Il path usa `.` come separatore
- Gli alias possono essere concatenati (a → b → c)
- VIETATI i riferimenti circolari

## 4.3 Estensione Gruppi ($extends)

Un gruppo può ereditare da un altro.

### Struttura:
```json
{
  "button": {
    "$type": "color",
    "background": {
      "$value": { "colorSpace": "srgb", "components": [0.2, 0.4, 0.9] }
    }
  },
  "button-primary": {
    "$extends": "{button}",
    "background": {
      "$value": { "colorSpace": "srgb", "components": [0.9, 0.2, 0.4] }
    }
  }
}
```

### Comportamento:
- **Eredità**: Tutti i token del gruppo referenziato vengono copiati
- **Override**: Token locali con stesso path sovrascrivono quelli ereditati
- **Aggiunta**: Token locali nuovi vengono aggiunti

# ============================================================================
# SEZIONE 5: ARCHITETTURA TOKEN A 3 LIVELLI
# ============================================================================

## 5.1 Panoramica

L'architettura a 3 livelli garantisce scalabilità e manutenibilità:

```
┌─────────────────────────────────────────────────────────────┐
│                    COMPONENT TOKENS                          │
│   (button.background, input.border, card.shadow)            │
│   ↓ referenziano                                             │
├─────────────────────────────────────────────────────────────┤
│                    SEMANTIC TOKENS                           │
│   (surface.primary, text.default, border.default)           │
│   ↓ referenziano                                             │
├─────────────────────────────────────────────────────────────┤
│                    PRIMITIVE TOKENS                          │
│   (blue.500, gray.100, spacing.4, font.size.lg)             │
│   (valori raw - NON usare direttamente nei componenti)      │
└─────────────────────────────────────────────────────────────┘
```

## 5.2 Livello 1: PRIMITIVE TOKENS

### Definizione:
Token che definiscono valori RAW senza significato semantico.
NON devono essere usati direttamente nei componenti UI.

### Naming Convention:
`[property].[scale]` oppure `[color-name].[intensity]`

### Scala Colori Primitiva STANDARD:

```json
{
  "primitive": {
    "$description": "Raw values - DO NOT use directly in components",
    "color": {
      "$type": "color",
      "gray": {
        "50":  { "$value": { "colorSpace": "srgb", "components": [0.976, 0.980, 0.984], "hex": "#f9fafb" }},
        "100": { "$value": { "colorSpace": "srgb", "components": [0.945, 0.953, 0.961], "hex": "#f1f3f5" }},
        "200": { "$value": { "colorSpace": "srgb", "components": [0.906, 0.914, 0.925], "hex": "#e7e9ec" }},
        "300": { "$value": { "colorSpace": "srgb", "components": [0.831, 0.843, 0.859], "hex": "#d4d7db" }},
        "400": { "$value": { "colorSpace": "srgb", "components": [0.616, 0.639, 0.671], "hex": "#9da3ab" }},
        "500": { "$value": { "colorSpace": "srgb", "components": [0.424, 0.455, 0.502], "hex": "#6c7480" }},
        "600": { "$value": { "colorSpace": "srgb", "components": [0.314, 0.345, 0.392], "hex": "#505864" }},
        "700": { "$value": { "colorSpace": "srgb", "components": [0.231, 0.259, 0.306], "hex": "#3b424e" }},
        "800": { "$value": { "colorSpace": "srgb", "components": [0.161, 0.184, 0.227], "hex": "#292f3a" }},
        "900": { "$value": { "colorSpace": "srgb", "components": [0.106, 0.122, 0.161], "hex": "#1b1f29" }},
        "950": { "$value": { "colorSpace": "srgb", "components": [0.059, 0.071, 0.102], "hex": "#0f121a" }}
      },
      "blue": {
        "50":  { "$value": { "colorSpace": "srgb", "components": [0.937, 0.965, 1.000], "hex": "#eff6ff" }},
        "100": { "$value": { "colorSpace": "srgb", "components": [0.859, 0.918, 0.996], "hex": "#dbeafe" }},
        "200": { "$value": { "colorSpace": "srgb", "components": [0.749, 0.859, 0.996], "hex": "#bfdbfe" }},
        "300": { "$value": { "colorSpace": "srgb", "components": [0.576, 0.773, 0.992], "hex": "#93c5fd" }},
        "400": { "$value": { "colorSpace": "srgb", "components": [0.376, 0.647, 0.976], "hex": "#60a5fa" }},
        "500": { "$value": { "colorSpace": "srgb", "components": [0.231, 0.506, 0.969], "hex": "#3b82f6" }},
        "600": { "$value": { "colorSpace": "srgb", "components": [0.145, 0.388, 0.922], "hex": "#2563eb" }},
        "700": { "$value": { "colorSpace": "srgb", "components": [0.114, 0.306, 0.847], "hex": "#1d4ed8" }},
        "800": { "$value": { "colorSpace": "srgb", "components": [0.114, 0.255, 0.690], "hex": "#1e40af" }},
        "900": { "$value": { "colorSpace": "srgb", "components": [0.114, 0.224, 0.549], "hex": "#1e3a8c" }},
        "950": { "$value": { "colorSpace": "srgb", "components": [0.090, 0.145, 0.357], "hex": "#17255b" }}
      },
      "green": {
        "50":  { "$value": { "colorSpace": "srgb", "components": [0.941, 0.992, 0.957], "hex": "#f0fdf4" }},
        "100": { "$value": { "colorSpace": "srgb", "components": [0.863, 0.988, 0.906], "hex": "#dcfce7" }},
        "200": { "$value": { "colorSpace": "srgb", "components": [0.733, 0.969, 0.816], "hex": "#bbf7d0" }},
        "300": { "$value": { "colorSpace": "srgb", "components": [0.525, 0.929, 0.671], "hex": "#86efab" }},
        "400": { "$value": { "colorSpace": "srgb", "components": [0.286, 0.851, 0.506], "hex": "#4ade80" }},
        "500": { "$value": { "colorSpace": "srgb", "components": [0.133, 0.773, 0.369], "hex": "#22c55e" }},
        "600": { "$value": { "colorSpace": "srgb", "components": [0.086, 0.639, 0.290], "hex": "#16a34a" }},
        "700": { "$value": { "colorSpace": "srgb", "components": [0.082, 0.502, 0.247], "hex": "#15803f" }},
        "800": { "$value": { "colorSpace": "srgb", "components": [0.086, 0.396, 0.231], "hex": "#16653b" }},
        "900": { "$value": { "colorSpace": "srgb", "components": [0.078, 0.325, 0.208], "hex": "#145334" }},
        "950": { "$value": { "colorSpace": "srgb", "components": [0.031, 0.180, 0.106], "hex": "#082e1b" }}
      },
      "red": {
        "50":  { "$value": { "colorSpace": "srgb", "components": [0.996, 0.945, 0.945], "hex": "#fef2f2" }},
        "100": { "$value": { "colorSpace": "srgb", "components": [0.996, 0.886, 0.886], "hex": "#fee2e2" }},
        "200": { "$value": { "colorSpace": "srgb", "components": [0.996, 0.792, 0.792], "hex": "#fecaca" }},
        "300": { "$value": { "colorSpace": "srgb", "components": [0.988, 0.647, 0.647], "hex": "#fca5a5" }},
        "400": { "$value": { "colorSpace": "srgb", "components": [0.973, 0.443, 0.443], "hex": "#f87171" }},
        "500": { "$value": { "colorSpace": "srgb", "components": [0.937, 0.267, 0.267], "hex": "#ef4444" }},
        "600": { "$value": { "colorSpace": "srgb", "components": [0.863, 0.149, 0.149], "hex": "#dc2626" }},
        "700": { "$value": { "colorSpace": "srgb", "components": [0.725, 0.110, 0.110], "hex": "#b91c1c" }},
        "800": { "$value": { "colorSpace": "srgb", "components": [0.600, 0.106, 0.106], "hex": "#991b1b" }},
        "900": { "$value": { "colorSpace": "srgb", "components": [0.498, 0.114, 0.114], "hex": "#7f1d1d" }},
        "950": { "$value": { "colorSpace": "srgb", "components": [0.271, 0.051, 0.051], "hex": "#450d0d" }}
      },
      "amber": {
        "50":  { "$value": { "colorSpace": "srgb", "components": [1.000, 0.984, 0.922], "hex": "#fffbeb" }},
        "100": { "$value": { "colorSpace": "srgb", "components": [0.996, 0.953, 0.780], "hex": "#fef3c7" }},
        "200": { "$value": { "colorSpace": "srgb", "components": [0.992, 0.902, 0.545], "hex": "#fde68a" }},
        "300": { "$value": { "colorSpace": "srgb", "components": [0.988, 0.827, 0.302], "hex": "#fcd34d" }},
        "400": { "$value": { "colorSpace": "srgb", "components": [0.984, 0.733, 0.145], "hex": "#fbbf24" }},
        "500": { "$value": { "colorSpace": "srgb", "components": [0.961, 0.620, 0.043], "hex": "#f59e0b" }},
        "600": { "$value": { "colorSpace": "srgb", "components": [0.851, 0.471, 0.027], "hex": "#d97706" }},
        "700": { "$value": { "colorSpace": "srgb", "components": [0.706, 0.337, 0.031], "hex": "#b45308" }},
        "800": { "$value": { "colorSpace": "srgb", "components": [0.573, 0.255, 0.055], "hex": "#92410e" }},
        "900": { "$value": { "colorSpace": "srgb", "components": [0.471, 0.212, 0.059], "hex": "#78360f" }},
        "950": { "$value": { "colorSpace": "srgb", "components": [0.271, 0.106, 0.020], "hex": "#451b05" }}
      },
      "white": { "$value": { "colorSpace": "srgb", "components": [1, 1, 1], "hex": "#ffffff" }},
      "black": { "$value": { "colorSpace": "srgb", "components": [0, 0, 0], "hex": "#000000" }}
    }
  }
}
```

### Scala Spacing Primitiva (8px base unit):

```json
{
  "primitive": {
    "spacing": {
      "$type": "dimension",
      "0":   { "$value": { "value": 0, "unit": "px" }},
      "px":  { "$value": { "value": 1, "unit": "px" }},
      "0-5": { "$value": { "value": 2, "unit": "px" }},
      "1":   { "$value": { "value": 4, "unit": "px" }},
      "1-5": { "$value": { "value": 6, "unit": "px" }},
      "2":   { "$value": { "value": 8, "unit": "px" }},
      "2-5": { "$value": { "value": 10, "unit": "px" }},
      "3":   { "$value": { "value": 12, "unit": "px" }},
      "3-5": { "$value": { "value": 14, "unit": "px" }},
      "4":   { "$value": { "value": 16, "unit": "px" }},
      "5":   { "$value": { "value": 20, "unit": "px" }},
      "6":   { "$value": { "value": 24, "unit": "px" }},
      "7":   { "$value": { "value": 28, "unit": "px" }},
      "8":   { "$value": { "value": 32, "unit": "px" }},
      "9":   { "$value": { "value": 36, "unit": "px" }},
      "10":  { "$value": { "value": 40, "unit": "px" }},
      "11":  { "$value": { "value": 44, "unit": "px" }},
      "12":  { "$value": { "value": 48, "unit": "px" }},
      "14":  { "$value": { "value": 56, "unit": "px" }},
      "16":  { "$value": { "value": 64, "unit": "px" }},
      "20":  { "$value": { "value": 80, "unit": "px" }},
      "24":  { "$value": { "value": 96, "unit": "px" }},
      "28":  { "$value": { "value": 112, "unit": "px" }},
      "32":  { "$value": { "value": 128, "unit": "px" }},
      "36":  { "$value": { "value": 144, "unit": "px" }},
      "40":  { "$value": { "value": 160, "unit": "px" }},
      "44":  { "$value": { "value": 176, "unit": "px" }},
      "48":  { "$value": { "value": 192, "unit": "px" }},
      "52":  { "$value": { "value": 208, "unit": "px" }},
      "56":  { "$value": { "value": 224, "unit": "px" }},
      "60":  { "$value": { "value": 240, "unit": "px" }},
      "64":  { "$value": { "value": 256, "unit": "px" }},
      "72":  { "$value": { "value": 288, "unit": "px" }},
      "80":  { "$value": { "value": 320, "unit": "px" }},
      "96":  { "$value": { "value": 384, "unit": "px" }}
    }
  }
}
```

### Scala Typography Primitiva (Modular Scale 1.250):

```json
{
  "primitive": {
    "fontSize": {
      "$type": "dimension",
      "xs":   { "$value": { "value": 12, "unit": "px" }},
      "sm":   { "$value": { "value": 14, "unit": "px" }},
      "base": { "$value": { "value": 16, "unit": "px" }},
      "lg":   { "$value": { "value": 18, "unit": "px" }},
      "xl":   { "$value": { "value": 20, "unit": "px" }},
      "2xl":  { "$value": { "value": 24, "unit": "px" }},
      "3xl":  { "$value": { "value": 30, "unit": "px" }},
      "4xl":  { "$value": { "value": 36, "unit": "px" }},
      "5xl":  { "$value": { "value": 48, "unit": "px" }},
      "6xl":  { "$value": { "value": 60, "unit": "px" }},
      "7xl":  { "$value": { "value": 72, "unit": "px" }},
      "8xl":  { "$value": { "value": 96, "unit": "px" }},
      "9xl":  { "$value": { "value": 128, "unit": "px" }}
    },
    "fontFamily": {
      "$type": "fontFamily",
      "sans":  { "$value": ["Inter", "system-ui", "-apple-system", "sans-serif"] },
      "serif": { "$value": ["Georgia", "Cambria", "Times New Roman", "serif"] },
      "mono":  { "$value": ["JetBrains Mono", "Fira Code", "Consolas", "monospace"] }
    },
    "fontWeight": {
      "$type": "fontWeight",
      "thin":       { "$value": 100 },
      "extralight": { "$value": 200 },
      "light":      { "$value": 300 },
      "normal":     { "$value": 400 },
      "medium":     { "$value": 500 },
      "semibold":   { "$value": 600 },
      "bold":       { "$value": 700 },
      "extrabold":  { "$value": 800 },
      "black":      { "$value": 900 }
    },
    "lineHeight": {
      "$type": "number",
      "none":    { "$value": 1 },
      "tight":   { "$value": 1.25 },
      "snug":    { "$value": 1.375 },
      "normal":  { "$value": 1.5 },
      "relaxed": { "$value": 1.625 },
      "loose":   { "$value": 2 }
    },
    "letterSpacing": {
      "$type": "dimension",
      "tighter": { "$value": { "value": -0.05, "unit": "em" }},
      "tight":   { "$value": { "value": -0.025, "unit": "em" }},
      "normal":  { "$value": { "value": 0, "unit": "em" }},
      "wide":    { "$value": { "value": 0.025, "unit": "em" }},
      "wider":   { "$value": { "value": 0.05, "unit": "em" }},
      "widest":  { "$value": { "value": 0.1, "unit": "em" }}
    }
  }
}
```


## 5.3 Livello 2: SEMANTIC TOKENS

### Definizione:
Token che definiscono COME e DOVE usare i primitivi.
Rappresentano intenzioni di design, non valori specifici.

### Naming Convention:
`[category].[subcategory].[variant].[state]`

Dove:
- **category**: surface, text, border, icon, etc
- **subcategory**: primary, secondary, brand, danger, etc
- **variant**: hover, active, disabled, etc (opzionale)
- **state**: light, dark (gestito da theming)

### Semantic Color Tokens STANDARD:

```json
{
  "semantic": {
    "$description": "Intent-based tokens - USE these in components",
    "color": {
      "$type": "color",
      
      "surface": {
        "$description": "Backgrounds and fill colors",
        "default":     { "$value": "{primitive.color.white}" },
        "muted":       { "$value": "{primitive.color.gray.50}" },
        "subtle":      { "$value": "{primitive.color.gray.100}" },
        "emphasis":    { "$value": "{primitive.color.gray.200}" },
        "inverse":     { "$value": "{primitive.color.gray.900}" },
        "brand":       { "$value": "{primitive.color.blue.500}" },
        "brand-muted": { "$value": "{primitive.color.blue.50}" }
      },
      
      "text": {
        "$description": "Text colors",
        "default":     { "$value": "{primitive.color.gray.900}" },
        "muted":       { "$value": "{primitive.color.gray.600}" },
        "subtle":      { "$value": "{primitive.color.gray.500}" },
        "placeholder": { "$value": "{primitive.color.gray.400}" },
        "inverse":     { "$value": "{primitive.color.white}" },
        "brand":       { "$value": "{primitive.color.blue.600}" },
        "link":        { "$value": "{primitive.color.blue.600}" },
        "link-hover":  { "$value": "{primitive.color.blue.700}" }
      },
      
      "border": {
        "$description": "Border colors",
        "default":     { "$value": "{primitive.color.gray.200}" },
        "muted":       { "$value": "{primitive.color.gray.100}" },
        "emphasis":    { "$value": "{primitive.color.gray.300}" },
        "strong":      { "$value": "{primitive.color.gray.400}" },
        "brand":       { "$value": "{primitive.color.blue.500}" },
        "focus":       { "$value": "{primitive.color.blue.500}" }
      },
      
      "icon": {
        "$description": "Icon colors",
        "default":     { "$value": "{primitive.color.gray.600}" },
        "muted":       { "$value": "{primitive.color.gray.400}" },
        "inverse":     { "$value": "{primitive.color.white}" },
        "brand":       { "$value": "{primitive.color.blue.500}" }
      },
      
      "feedback": {
        "$description": "Feedback/status colors",
        "success": {
          "default": { "$value": "{primitive.color.green.500}" },
          "muted":   { "$value": "{primitive.color.green.50}" },
          "text":    { "$value": "{primitive.color.green.700}" }
        },
        "warning": {
          "default": { "$value": "{primitive.color.amber.500}" },
          "muted":   { "$value": "{primitive.color.amber.50}" },
          "text":    { "$value": "{primitive.color.amber.700}" }
        },
        "danger": {
          "default": { "$value": "{primitive.color.red.500}" },
          "muted":   { "$value": "{primitive.color.red.50}" },
          "text":    { "$value": "{primitive.color.red.700}" }
        },
        "info": {
          "default": { "$value": "{primitive.color.blue.500}" },
          "muted":   { "$value": "{primitive.color.blue.50}" },
          "text":    { "$value": "{primitive.color.blue.700}" }
        }
      },
      
      "interactive": {
        "$description": "Interactive element states",
        "hover":    { "$value": "{primitive.color.gray.100}" },
        "active":   { "$value": "{primitive.color.gray.200}" },
        "selected": { "$value": "{primitive.color.blue.100}" },
        "disabled": { "$value": "{primitive.color.gray.100}" }
      }
    }
  }
}
```

### Semantic Spacing Tokens:

```json
{
  "semantic": {
    "spacing": {
      "$type": "dimension",
      
      "inset": {
        "$description": "Padding inside containers",
        "none":  { "$value": "{primitive.spacing.0}" },
        "xs":    { "$value": "{primitive.spacing.1}" },
        "sm":    { "$value": "{primitive.spacing.2}" },
        "md":    { "$value": "{primitive.spacing.4}" },
        "lg":    { "$value": "{primitive.spacing.6}" },
        "xl":    { "$value": "{primitive.spacing.8}" },
        "2xl":   { "$value": "{primitive.spacing.12}" }
      },
      
      "stack": {
        "$description": "Vertical space between elements",
        "none":  { "$value": "{primitive.spacing.0}" },
        "xs":    { "$value": "{primitive.spacing.1}" },
        "sm":    { "$value": "{primitive.spacing.2}" },
        "md":    { "$value": "{primitive.spacing.4}" },
        "lg":    { "$value": "{primitive.spacing.6}" },
        "xl":    { "$value": "{primitive.spacing.8}" },
        "2xl":   { "$value": "{primitive.spacing.12}" },
        "3xl":   { "$value": "{primitive.spacing.16}" }
      },
      
      "inline": {
        "$description": "Horizontal space between elements",
        "none":  { "$value": "{primitive.spacing.0}" },
        "xs":    { "$value": "{primitive.spacing.1}" },
        "sm":    { "$value": "{primitive.spacing.2}" },
        "md":    { "$value": "{primitive.spacing.3}" },
        "lg":    { "$value": "{primitive.spacing.4}" },
        "xl":    { "$value": "{primitive.spacing.6}" }
      },
      
      "gap": {
        "$description": "Grid/flex gap",
        "none":  { "$value": "{primitive.spacing.0}" },
        "xs":    { "$value": "{primitive.spacing.2}" },
        "sm":    { "$value": "{primitive.spacing.3}" },
        "md":    { "$value": "{primitive.spacing.4}" },
        "lg":    { "$value": "{primitive.spacing.6}" },
        "xl":    { "$value": "{primitive.spacing.8}" }
      }
    }
  }
}
```

### Semantic Typography Tokens:

```json
{
  "semantic": {
    "typography": {
      "$type": "typography",
      
      "display": {
        "2xl": {
          "$value": {
            "fontFamily": "{primitive.fontFamily.sans}",
            "fontSize": "{primitive.fontSize.7xl}",
            "fontWeight": "{primitive.fontWeight.bold}",
            "lineHeight": "{primitive.lineHeight.tight}",
            "letterSpacing": "{primitive.letterSpacing.tight}"
          }
        },
        "xl": {
          "$value": {
            "fontFamily": "{primitive.fontFamily.sans}",
            "fontSize": "{primitive.fontSize.6xl}",
            "fontWeight": "{primitive.fontWeight.bold}",
            "lineHeight": "{primitive.lineHeight.tight}",
            "letterSpacing": "{primitive.letterSpacing.tight}"
          }
        },
        "lg": {
          "$value": {
            "fontFamily": "{primitive.fontFamily.sans}",
            "fontSize": "{primitive.fontSize.5xl}",
            "fontWeight": "{primitive.fontWeight.bold}",
            "lineHeight": "{primitive.lineHeight.tight}",
            "letterSpacing": "{primitive.letterSpacing.tight}"
          }
        }
      },
      
      "heading": {
        "h1": {
          "$value": {
            "fontFamily": "{primitive.fontFamily.sans}",
            "fontSize": "{primitive.fontSize.4xl}",
            "fontWeight": "{primitive.fontWeight.bold}",
            "lineHeight": "{primitive.lineHeight.tight}",
            "letterSpacing": "{primitive.letterSpacing.tight}"
          }
        },
        "h2": {
          "$value": {
            "fontFamily": "{primitive.fontFamily.sans}",
            "fontSize": "{primitive.fontSize.3xl}",
            "fontWeight": "{primitive.fontWeight.semibold}",
            "lineHeight": "{primitive.lineHeight.tight}",
            "letterSpacing": "{primitive.letterSpacing.normal}"
          }
        },
        "h3": {
          "$value": {
            "fontFamily": "{primitive.fontFamily.sans}",
            "fontSize": "{primitive.fontSize.2xl}",
            "fontWeight": "{primitive.fontWeight.semibold}",
            "lineHeight": "{primitive.lineHeight.snug}",
            "letterSpacing": "{primitive.letterSpacing.normal}"
          }
        },
        "h4": {
          "$value": {
            "fontFamily": "{primitive.fontFamily.sans}",
            "fontSize": "{primitive.fontSize.xl}",
            "fontWeight": "{primitive.fontWeight.semibold}",
            "lineHeight": "{primitive.lineHeight.snug}",
            "letterSpacing": "{primitive.letterSpacing.normal}"
          }
        },
        "h5": {
          "$value": {
            "fontFamily": "{primitive.fontFamily.sans}",
            "fontSize": "{primitive.fontSize.lg}",
            "fontWeight": "{primitive.fontWeight.medium}",
            "lineHeight": "{primitive.lineHeight.normal}",
            "letterSpacing": "{primitive.letterSpacing.normal}"
          }
        },
        "h6": {
          "$value": {
            "fontFamily": "{primitive.fontFamily.sans}",
            "fontSize": "{primitive.fontSize.base}",
            "fontWeight": "{primitive.fontWeight.medium}",
            "lineHeight": "{primitive.lineHeight.normal}",
            "letterSpacing": "{primitive.letterSpacing.normal}"
          }
        }
      },
      
      "body": {
        "lg": {
          "$value": {
            "fontFamily": "{primitive.fontFamily.sans}",
            "fontSize": "{primitive.fontSize.lg}",
            "fontWeight": "{primitive.fontWeight.normal}",
            "lineHeight": "{primitive.lineHeight.relaxed}",
            "letterSpacing": "{primitive.letterSpacing.normal}"
          }
        },
        "md": {
          "$value": {
            "fontFamily": "{primitive.fontFamily.sans}",
            "fontSize": "{primitive.fontSize.base}",
            "fontWeight": "{primitive.fontWeight.normal}",
            "lineHeight": "{primitive.lineHeight.normal}",
            "letterSpacing": "{primitive.letterSpacing.normal}"
          }
        },
        "sm": {
          "$value": {
            "fontFamily": "{primitive.fontFamily.sans}",
            "fontSize": "{primitive.fontSize.sm}",
            "fontWeight": "{primitive.fontWeight.normal}",
            "lineHeight": "{primitive.lineHeight.normal}",
            "letterSpacing": "{primitive.letterSpacing.normal}"
          }
        }
      },
      
      "label": {
        "lg": {
          "$value": {
            "fontFamily": "{primitive.fontFamily.sans}",
            "fontSize": "{primitive.fontSize.sm}",
            "fontWeight": "{primitive.fontWeight.medium}",
            "lineHeight": "{primitive.lineHeight.tight}",
            "letterSpacing": "{primitive.letterSpacing.normal}"
          }
        },
        "md": {
          "$value": {
            "fontFamily": "{primitive.fontFamily.sans}",
            "fontSize": "{primitive.fontSize.sm}",
            "fontWeight": "{primitive.fontWeight.medium}",
            "lineHeight": "{primitive.lineHeight.tight}",
            "letterSpacing": "{primitive.letterSpacing.normal}"
          }
        },
        "sm": {
          "$value": {
            "fontFamily": "{primitive.fontFamily.sans}",
            "fontSize": "{primitive.fontSize.xs}",
            "fontWeight": "{primitive.fontWeight.medium}",
            "lineHeight": "{primitive.lineHeight.tight}",
            "letterSpacing": "{primitive.letterSpacing.wide}"
          }
        }
      },
      
      "caption": {
        "$value": {
          "fontFamily": "{primitive.fontFamily.sans}",
          "fontSize": "{primitive.fontSize.xs}",
          "fontWeight": "{primitive.fontWeight.normal}",
          "lineHeight": "{primitive.lineHeight.normal}",
          "letterSpacing": "{primitive.letterSpacing.normal}"
        }
      },
      
      "code": {
        "$value": {
          "fontFamily": "{primitive.fontFamily.mono}",
          "fontSize": "{primitive.fontSize.sm}",
          "fontWeight": "{primitive.fontWeight.normal}",
          "lineHeight": "{primitive.lineHeight.normal}",
          "letterSpacing": "{primitive.letterSpacing.normal}"
        }
      }
    }
  }
}
```


## 5.4 Livello 3: COMPONENT TOKENS

### Definizione:
Token specifici per ogni componente UI.
Referenziano SOLO semantic tokens (mai primitivi direttamente).

### Naming Convention:
`[component].[element].[property].[variant].[state]`

### Component Token EXAMPLES:

```json
{
  "component": {
    "$description": "Component-specific tokens - reference semantic tokens only",
    
    "button": {
      "primary": {
        "background": {
          "default":  { "$value": "{semantic.color.surface.brand}" },
          "hover":    { "$value": "{primitive.color.blue.600}" },
          "active":   { "$value": "{primitive.color.blue.700}" },
          "disabled": { "$value": "{primitive.color.gray.300}" }
        },
        "text": {
          "default":  { "$value": "{semantic.color.text.inverse}" },
          "disabled": { "$value": "{primitive.color.gray.500}" }
        },
        "border": {
          "default": { "$value": "{semantic.color.surface.brand}" },
          "focus":   { "$value": "{semantic.color.border.focus}" }
        }
      },
      "secondary": {
        "background": {
          "default":  { "$value": "{semantic.color.surface.default}" },
          "hover":    { "$value": "{semantic.color.interactive.hover}" },
          "active":   { "$value": "{semantic.color.interactive.active}" },
          "disabled": { "$value": "{semantic.color.surface.muted}" }
        },
        "text": {
          "default":  { "$value": "{semantic.color.text.default}" },
          "disabled": { "$value": "{semantic.color.text.subtle}" }
        },
        "border": {
          "default": { "$value": "{semantic.color.border.default}" },
          "focus":   { "$value": "{semantic.color.border.focus}" }
        }
      },
      "ghost": {
        "background": {
          "default":  { "$value": "transparent" },
          "hover":    { "$value": "{semantic.color.interactive.hover}" },
          "active":   { "$value": "{semantic.color.interactive.active}" }
        },
        "text": {
          "default": { "$value": "{semantic.color.text.brand}" }
        }
      },
      "danger": {
        "background": {
          "default": { "$value": "{semantic.color.feedback.danger.default}" },
          "hover":   { "$value": "{primitive.color.red.600}" }
        },
        "text": {
          "default": { "$value": "{semantic.color.text.inverse}" }
        }
      },
      "padding": {
        "sm": { "$value": "{semantic.spacing.inset.xs} {semantic.spacing.inset.sm}" },
        "md": { "$value": "{semantic.spacing.inset.sm} {semantic.spacing.inset.md}" },
        "lg": { "$value": "{semantic.spacing.inset.md} {semantic.spacing.inset.lg}" }
      },
      "borderRadius": {
        "sm": { "$value": { "value": 4, "unit": "px" }},
        "md": { "$value": { "value": 6, "unit": "px" }},
        "lg": { "$value": { "value": 8, "unit": "px" }},
        "full": { "$value": { "value": 9999, "unit": "px" }}
      }
    },
    
    "input": {
      "background": {
        "default":  { "$value": "{semantic.color.surface.default}" },
        "disabled": { "$value": "{semantic.color.surface.muted}" }
      },
      "text": {
        "default":     { "$value": "{semantic.color.text.default}" },
        "placeholder": { "$value": "{semantic.color.text.placeholder}" },
        "disabled":    { "$value": "{semantic.color.text.muted}" }
      },
      "border": {
        "default": { "$value": "{semantic.color.border.default}" },
        "hover":   { "$value": "{semantic.color.border.emphasis}" },
        "focus":   { "$value": "{semantic.color.border.focus}" },
        "error":   { "$value": "{semantic.color.feedback.danger.default}" }
      },
      "padding": {
        "sm": { "$value": "{semantic.spacing.inset.xs} {semantic.spacing.inset.sm}" },
        "md": { "$value": "{semantic.spacing.inset.sm} {semantic.spacing.inset.md}" },
        "lg": { "$value": "{semantic.spacing.inset.md} {semantic.spacing.inset.md}" }
      },
      "borderRadius": { "$value": { "value": 6, "unit": "px" }}
    },
    
    "card": {
      "background": { "$value": "{semantic.color.surface.default}" },
      "border":     { "$value": "{semantic.color.border.default}" },
      "shadow":     { "$value": "{primitive.shadow.sm}" },
      "padding": {
        "sm": { "$value": "{semantic.spacing.inset.sm}" },
        "md": { "$value": "{semantic.spacing.inset.md}" },
        "lg": { "$value": "{semantic.spacing.inset.lg}" }
      },
      "borderRadius": { "$value": { "value": 8, "unit": "px" }}
    },
    
    "modal": {
      "background":      { "$value": "{semantic.color.surface.default}" },
      "overlay":         { "$value": "{primitive.color.black}", "alpha": 0.5 },
      "border":          { "$value": "{semantic.color.border.default}" },
      "shadow":          { "$value": "{primitive.shadow.xl}" },
      "padding":         { "$value": "{semantic.spacing.inset.lg}" },
      "borderRadius":    { "$value": { "value": 12, "unit": "px" }},
      "header-spacing":  { "$value": "{semantic.spacing.stack.md}" },
      "content-spacing": { "$value": "{semantic.spacing.stack.lg}" },
      "footer-spacing":  { "$value": "{semantic.spacing.stack.md}" }
    },
    
    "alert": {
      "success": {
        "background": { "$value": "{semantic.color.feedback.success.muted}" },
        "border":     { "$value": "{semantic.color.feedback.success.default}" },
        "text":       { "$value": "{semantic.color.feedback.success.text}" },
        "icon":       { "$value": "{semantic.color.feedback.success.default}" }
      },
      "warning": {
        "background": { "$value": "{semantic.color.feedback.warning.muted}" },
        "border":     { "$value": "{semantic.color.feedback.warning.default}" },
        "text":       { "$value": "{semantic.color.feedback.warning.text}" },
        "icon":       { "$value": "{semantic.color.feedback.warning.default}" }
      },
      "danger": {
        "background": { "$value": "{semantic.color.feedback.danger.muted}" },
        "border":     { "$value": "{semantic.color.feedback.danger.default}" },
        "text":       { "$value": "{semantic.color.feedback.danger.text}" },
        "icon":       { "$value": "{semantic.color.feedback.danger.default}" }
      },
      "info": {
        "background": { "$value": "{semantic.color.feedback.info.muted}" },
        "border":     { "$value": "{semantic.color.feedback.info.default}" },
        "text":       { "$value": "{semantic.color.feedback.info.text}" },
        "icon":       { "$value": "{semantic.color.feedback.info.default}" }
      },
      "padding":      { "$value": "{semantic.spacing.inset.md}" },
      "borderRadius": { "$value": { "value": 8, "unit": "px" }}
    },
    
    "badge": {
      "primary": {
        "background": { "$value": "{semantic.color.surface.brand}" },
        "text":       { "$value": "{semantic.color.text.inverse}" }
      },
      "secondary": {
        "background": { "$value": "{semantic.color.surface.muted}" },
        "text":       { "$value": "{semantic.color.text.default}" }
      },
      "success": {
        "background": { "$value": "{semantic.color.feedback.success.muted}" },
        "text":       { "$value": "{semantic.color.feedback.success.text}" }
      },
      "warning": {
        "background": { "$value": "{semantic.color.feedback.warning.muted}" },
        "text":       { "$value": "{semantic.color.feedback.warning.text}" }
      },
      "danger": {
        "background": { "$value": "{semantic.color.feedback.danger.muted}" },
        "text":       { "$value": "{semantic.color.feedback.danger.text}" }
      },
      "padding":      { "$value": "{semantic.spacing.inset.xs} {semantic.spacing.inset.sm}" },
      "borderRadius": { "$value": { "value": 9999, "unit": "px" }}
    },
    
    "table": {
      "header": {
        "background": { "$value": "{semantic.color.surface.muted}" },
        "text":       { "$value": "{semantic.color.text.default}" }
      },
      "row": {
        "background": { "$value": "{semantic.color.surface.default}" },
        "hover":      { "$value": "{semantic.color.interactive.hover}" },
        "selected":   { "$value": "{semantic.color.interactive.selected}" }
      },
      "cell": {
        "border":  { "$value": "{semantic.color.border.muted}" },
        "padding": { "$value": "{semantic.spacing.inset.sm} {semantic.spacing.inset.md}" }
      }
    },
    
    "nav": {
      "background": { "$value": "{semantic.color.surface.default}" },
      "border":     { "$value": "{semantic.color.border.default}" },
      "item": {
        "text": {
          "default": { "$value": "{semantic.color.text.muted}" },
          "hover":   { "$value": "{semantic.color.text.default}" },
          "active":  { "$value": "{semantic.color.text.brand}" }
        },
        "background": {
          "default": { "$value": "transparent" },
          "hover":   { "$value": "{semantic.color.interactive.hover}" },
          "active":  { "$value": "{semantic.color.interactive.selected}" }
        }
      }
    },
    
    "tooltip": {
      "background":   { "$value": "{semantic.color.surface.inverse}" },
      "text":         { "$value": "{semantic.color.text.inverse}" },
      "padding":      { "$value": "{semantic.spacing.inset.xs} {semantic.spacing.inset.sm}" },
      "borderRadius": { "$value": { "value": 4, "unit": "px" }}
    },
    
    "avatar": {
      "background": { "$value": "{semantic.color.surface.muted}" },
      "text":       { "$value": "{semantic.color.text.default}" },
      "border":     { "$value": "{semantic.color.border.default}" },
      "size": {
        "xs": { "$value": { "value": 24, "unit": "px" }},
        "sm": { "$value": { "value": 32, "unit": "px" }},
        "md": { "$value": { "value": 40, "unit": "px" }},
        "lg": { "$value": { "value": 48, "unit": "px" }},
        "xl": { "$value": { "value": 64, "unit": "px" }},
        "2xl": { "$value": { "value": 96, "unit": "px" }}
      }
    }
  }
}
```


# ============================================================================
# SEZIONE 6: SCALE AGGIUNTIVE STANDARD
# ============================================================================

## 6.1 Border Radius Scale

```json
{
  "primitive": {
    "borderRadius": {
      "$type": "dimension",
      "none":  { "$value": { "value": 0, "unit": "px" }},
      "sm":    { "$value": { "value": 4, "unit": "px" }},
      "md":    { "$value": { "value": 6, "unit": "px" }},
      "lg":    { "$value": { "value": 8, "unit": "px" }},
      "xl":    { "$value": { "value": 12, "unit": "px" }},
      "2xl":   { "$value": { "value": 16, "unit": "px" }},
      "3xl":   { "$value": { "value": 24, "unit": "px" }},
      "full":  { "$value": { "value": 9999, "unit": "px" }}
    }
  }
}
```

## 6.2 Shadow Scale

```json
{
  "primitive": {
    "shadow": {
      "$type": "shadow",
      "none": { "$value": [] },
      "xs": {
        "$value": {
          "color": { "colorSpace": "srgb", "components": [0, 0, 0], "alpha": 0.05 },
          "offsetX": { "value": 0, "unit": "px" },
          "offsetY": { "value": 1, "unit": "px" },
          "blur": { "value": 2, "unit": "px" },
          "spread": { "value": 0, "unit": "px" }
        }
      },
      "sm": {
        "$value": {
          "color": { "colorSpace": "srgb", "components": [0, 0, 0], "alpha": 0.05 },
          "offsetX": { "value": 0, "unit": "px" },
          "offsetY": { "value": 1, "unit": "px" },
          "blur": { "value": 3, "unit": "px" },
          "spread": { "value": 0, "unit": "px" }
        }
      },
      "md": {
        "$value": [
          {
            "color": { "colorSpace": "srgb", "components": [0, 0, 0], "alpha": 0.1 },
            "offsetX": { "value": 0, "unit": "px" },
            "offsetY": { "value": 4, "unit": "px" },
            "blur": { "value": 6, "unit": "px" },
            "spread": { "value": -1, "unit": "px" }
          },
          {
            "color": { "colorSpace": "srgb", "components": [0, 0, 0], "alpha": 0.06 },
            "offsetX": { "value": 0, "unit": "px" },
            "offsetY": { "value": 2, "unit": "px" },
            "blur": { "value": 4, "unit": "px" },
            "spread": { "value": -1, "unit": "px" }
          }
        ]
      },
      "lg": {
        "$value": [
          {
            "color": { "colorSpace": "srgb", "components": [0, 0, 0], "alpha": 0.1 },
            "offsetX": { "value": 0, "unit": "px" },
            "offsetY": { "value": 10, "unit": "px" },
            "blur": { "value": 15, "unit": "px" },
            "spread": { "value": -3, "unit": "px" }
          },
          {
            "color": { "colorSpace": "srgb", "components": [0, 0, 0], "alpha": 0.05 },
            "offsetX": { "value": 0, "unit": "px" },
            "offsetY": { "value": 4, "unit": "px" },
            "blur": { "value": 6, "unit": "px" },
            "spread": { "value": -2, "unit": "px" }
          }
        ]
      },
      "xl": {
        "$value": [
          {
            "color": { "colorSpace": "srgb", "components": [0, 0, 0], "alpha": 0.1 },
            "offsetX": { "value": 0, "unit": "px" },
            "offsetY": { "value": 20, "unit": "px" },
            "blur": { "value": 25, "unit": "px" },
            "spread": { "value": -5, "unit": "px" }
          },
          {
            "color": { "colorSpace": "srgb", "components": [0, 0, 0], "alpha": 0.04 },
            "offsetX": { "value": 0, "unit": "px" },
            "offsetY": { "value": 8, "unit": "px" },
            "blur": { "value": 10, "unit": "px" },
            "spread": { "value": -5, "unit": "px" }
          }
        ]
      },
      "2xl": {
        "$value": {
          "color": { "colorSpace": "srgb", "components": [0, 0, 0], "alpha": 0.25 },
          "offsetX": { "value": 0, "unit": "px" },
          "offsetY": { "value": 25, "unit": "px" },
          "blur": { "value": 50, "unit": "px" },
          "spread": { "value": -12, "unit": "px" }
        }
      },
      "inner": {
        "$value": {
          "color": { "colorSpace": "srgb", "components": [0, 0, 0], "alpha": 0.05 },
          "offsetX": { "value": 0, "unit": "px" },
          "offsetY": { "value": 2, "unit": "px" },
          "blur": { "value": 4, "unit": "px" },
          "spread": { "value": 0, "unit": "px" },
          "inset": true
        }
      }
    }
  }
}
```

## 6.3 Z-Index Scale

```json
{
  "primitive": {
    "zIndex": {
      "$type": "number",
      "auto":     { "$value": "auto" },
      "hide":     { "$value": -1 },
      "base":     { "$value": 0 },
      "raised":   { "$value": 1 },
      "dropdown": { "$value": 10 },
      "sticky":   { "$value": 20 },
      "fixed":    { "$value": 30 },
      "drawer":   { "$value": 40 },
      "modal":    { "$value": 50 },
      "popover":  { "$value": 60 },
      "tooltip":  { "$value": 70 },
      "toast":    { "$value": 80 },
      "max":      { "$value": 9999 }
    }
  }
}
```

## 6.4 Breakpoints Scale

```json
{
  "primitive": {
    "breakpoint": {
      "$type": "dimension",
      "xs":  { "$value": { "value": 0, "unit": "px" }},
      "sm":  { "$value": { "value": 640, "unit": "px" }},
      "md":  { "$value": { "value": 768, "unit": "px" }},
      "lg":  { "$value": { "value": 1024, "unit": "px" }},
      "xl":  { "$value": { "value": 1280, "unit": "px" }},
      "2xl": { "$value": { "value": 1536, "unit": "px" }}
    }
  }
}
```

## 6.5 Animation/Motion Scale

```json
{
  "primitive": {
    "duration": {
      "$type": "duration",
      "instant":  { "$value": { "value": 0, "unit": "ms" }},
      "fastest":  { "$value": { "value": 50, "unit": "ms" }},
      "faster":   { "$value": { "value": 100, "unit": "ms" }},
      "fast":     { "$value": { "value": 150, "unit": "ms" }},
      "normal":   { "$value": { "value": 200, "unit": "ms" }},
      "slow":     { "$value": { "value": 300, "unit": "ms" }},
      "slower":   { "$value": { "value": 400, "unit": "ms" }},
      "slowest":  { "$value": { "value": 500, "unit": "ms" }}
    },
    "easing": {
      "$type": "cubicBezier",
      "linear":      { "$value": [0, 0, 1, 1] },
      "ease":        { "$value": [0.25, 0.1, 0.25, 1] },
      "ease-in":     { "$value": [0.42, 0, 1, 1] },
      "ease-out":    { "$value": [0, 0, 0.58, 1] },
      "ease-in-out": { "$value": [0.42, 0, 0.58, 1] },
      "spring":      { "$value": [0.175, 0.885, 0.32, 1.275] },
      "bounce":      { "$value": [0.68, -0.55, 0.265, 1.55] }
    }
  }
}
```

## 6.6 Opacity Scale

```json
{
  "primitive": {
    "opacity": {
      "$type": "number",
      "0":   { "$value": 0 },
      "5":   { "$value": 0.05 },
      "10":  { "$value": 0.1 },
      "20":  { "$value": 0.2 },
      "25":  { "$value": 0.25 },
      "30":  { "$value": 0.3 },
      "40":  { "$value": 0.4 },
      "50":  { "$value": 0.5 },
      "60":  { "$value": 0.6 },
      "70":  { "$value": 0.7 },
      "75":  { "$value": 0.75 },
      "80":  { "$value": 0.8 },
      "90":  { "$value": 0.9 },
      "95":  { "$value": 0.95 },
      "100": { "$value": 1 }
    }
  }
}
```

# ============================================================================
# SEZIONE 7: THEMING (LIGHT/DARK MODE)
# ============================================================================

## 7.1 Struttura Multi-File per Theming

Per supportare light/dark mode, usa file separati per ogni tema:

```
tokens/
├── primitive.tokens.json      # Valori raw condivisi
├── semantic.light.tokens.json # Semantic tokens per light mode
├── semantic.dark.tokens.json  # Semantic tokens per dark mode
└── component.tokens.json      # Component tokens (condivisi)
```

## 7.2 Semantic Tokens - DARK MODE

```json
{
  "$schema": "https://www.designtokens.org/schemas/2025.10/format.json",
  "$description": "Dark mode semantic tokens",
  
  "semantic": {
    "color": {
      "$type": "color",
      
      "surface": {
        "default":     { "$value": "{primitive.color.gray.900}" },
        "muted":       { "$value": "{primitive.color.gray.800}" },
        "subtle":      { "$value": "{primitive.color.gray.700}" },
        "emphasis":    { "$value": "{primitive.color.gray.600}" },
        "inverse":     { "$value": "{primitive.color.gray.50}" },
        "brand":       { "$value": "{primitive.color.blue.500}" },
        "brand-muted": { "$value": "{primitive.color.blue.950}" }
      },
      
      "text": {
        "default":     { "$value": "{primitive.color.gray.50}" },
        "muted":       { "$value": "{primitive.color.gray.300}" },
        "subtle":      { "$value": "{primitive.color.gray.400}" },
        "placeholder": { "$value": "{primitive.color.gray.500}" },
        "inverse":     { "$value": "{primitive.color.gray.900}" },
        "brand":       { "$value": "{primitive.color.blue.400}" },
        "link":        { "$value": "{primitive.color.blue.400}" },
        "link-hover":  { "$value": "{primitive.color.blue.300}" }
      },
      
      "border": {
        "default":     { "$value": "{primitive.color.gray.700}" },
        "muted":       { "$value": "{primitive.color.gray.800}" },
        "emphasis":    { "$value": "{primitive.color.gray.600}" },
        "strong":      { "$value": "{primitive.color.gray.500}" },
        "brand":       { "$value": "{primitive.color.blue.500}" },
        "focus":       { "$value": "{primitive.color.blue.400}" }
      },
      
      "icon": {
        "default":     { "$value": "{primitive.color.gray.300}" },
        "muted":       { "$value": "{primitive.color.gray.500}" },
        "inverse":     { "$value": "{primitive.color.gray.900}" },
        "brand":       { "$value": "{primitive.color.blue.400}" }
      },
      
      "feedback": {
        "success": {
          "default": { "$value": "{primitive.color.green.400}" },
          "muted":   { "$value": "{primitive.color.green.950}" },
          "text":    { "$value": "{primitive.color.green.300}" }
        },
        "warning": {
          "default": { "$value": "{primitive.color.amber.400}" },
          "muted":   { "$value": "{primitive.color.amber.950}" },
          "text":    { "$value": "{primitive.color.amber.300}" }
        },
        "danger": {
          "default": { "$value": "{primitive.color.red.400}" },
          "muted":   { "$value": "{primitive.color.red.950}" },
          "text":    { "$value": "{primitive.color.red.300}" }
        },
        "info": {
          "default": { "$value": "{primitive.color.blue.400}" },
          "muted":   { "$value": "{primitive.color.blue.950}" },
          "text":    { "$value": "{primitive.color.blue.300}" }
        }
      },
      
      "interactive": {
        "hover":    { "$value": "{primitive.color.gray.800}" },
        "active":   { "$value": "{primitive.color.gray.700}" },
        "selected": { "$value": "{primitive.color.blue.900}" },
        "disabled": { "$value": "{primitive.color.gray.800}" }
      }
    }
  }
}
```

## 7.3 CSS Output per Theming

```css
/* Light mode (default) */
:root {
  --color-surface-default: #ffffff;
  --color-surface-muted: #f9fafb;
  --color-text-default: #1b1f29;
  --color-text-muted: #505864;
  --color-border-default: #e7e9ec;
  /* ... altri token ... */
}

/* Dark mode */
:root[data-theme="dark"],
.dark {
  --color-surface-default: #1b1f29;
  --color-surface-muted: #292f3a;
  --color-text-default: #f9fafb;
  --color-text-muted: #d4d7db;
  --color-border-default: #3b424e;
  /* ... altri token ... */
}

/* System preference */
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    --color-surface-default: #1b1f29;
    /* ... dark mode tokens ... */
  }
}
```


# ============================================================================
# SEZIONE 8: INTEGRAZIONE TAILWIND CSS
# ============================================================================

## 8.1 Tailwind CSS 4 @theme Directive

Tailwind CSS 4 introduce la direttiva `@theme` per definire tokens direttamente in CSS:

```css
@theme {
  /* Colors - Primitive */
  --color-gray-50: #f9fafb;
  --color-gray-100: #f1f3f5;
  --color-gray-200: #e7e9ec;
  --color-gray-300: #d4d7db;
  --color-gray-400: #9da3ab;
  --color-gray-500: #6c7480;
  --color-gray-600: #505864;
  --color-gray-700: #3b424e;
  --color-gray-800: #292f3a;
  --color-gray-900: #1b1f29;
  --color-gray-950: #0f121a;
  
  --color-blue-50: #eff6ff;
  --color-blue-500: #3b82f6;
  --color-blue-600: #2563eb;
  --color-blue-700: #1d4ed8;
  
  /* Spacing */
  --spacing-0: 0px;
  --spacing-1: 4px;
  --spacing-2: 8px;
  --spacing-3: 12px;
  --spacing-4: 16px;
  --spacing-6: 24px;
  --spacing-8: 32px;
  
  /* Border Radius */
  --radius-sm: 4px;
  --radius-md: 6px;
  --radius-lg: 8px;
  --radius-xl: 12px;
  --radius-full: 9999px;
  
  /* Font Family */
  --font-sans: "Inter", system-ui, -apple-system, sans-serif;
  --font-mono: "JetBrains Mono", "Fira Code", monospace;
  
  /* Font Size */
  --text-xs: 12px;
  --text-sm: 14px;
  --text-base: 16px;
  --text-lg: 18px;
  --text-xl: 20px;
  --text-2xl: 24px;
  --text-3xl: 30px;
  --text-4xl: 36px;
  
  /* Line Height */
  --leading-tight: 1.25;
  --leading-snug: 1.375;
  --leading-normal: 1.5;
  --leading-relaxed: 1.625;
  
  /* Shadows */
  --shadow-sm: 0 1px 2px rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1);
  --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1);
  
  /* Duration */
  --duration-fast: 150ms;
  --duration-normal: 200ms;
  --duration-slow: 300ms;
  
  /* Easing */
  --ease-out: cubic-bezier(0, 0, 0.2, 1);
  --ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
}
```

## 8.2 tailwind.config.js (Pre-v4)

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: 'class', // o 'media'
  theme: {
    colors: {
      transparent: 'transparent',
      current: 'currentColor',
      white: '#ffffff',
      black: '#000000',
      gray: {
        50: '#f9fafb',
        100: '#f1f3f5',
        200: '#e7e9ec',
        300: '#d4d7db',
        400: '#9da3ab',
        500: '#6c7480',
        600: '#505864',
        700: '#3b424e',
        800: '#292f3a',
        900: '#1b1f29',
        950: '#0f121a',
      },
      blue: {
        50: '#eff6ff',
        100: '#dbeafe',
        200: '#bfdbfe',
        300: '#93c5fd',
        400: '#60a5fa',
        500: '#3b82f6',
        600: '#2563eb',
        700: '#1d4ed8',
        800: '#1e40af',
        900: '#1e3a8c',
        950: '#17255b',
      },
      green: {
        50: '#f0fdf4',
        500: '#22c55e',
        600: '#16a34a',
        700: '#15803f',
      },
      red: {
        50: '#fef2f2',
        500: '#ef4444',
        600: '#dc2626',
        700: '#b91c1c',
      },
      amber: {
        50: '#fffbeb',
        500: '#f59e0b',
        600: '#d97706',
        700: '#b45308',
      },
    },
    spacing: {
      0: '0px',
      px: '1px',
      0.5: '2px',
      1: '4px',
      1.5: '6px',
      2: '8px',
      2.5: '10px',
      3: '12px',
      3.5: '14px',
      4: '16px',
      5: '20px',
      6: '24px',
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
    },
    borderRadius: {
      none: '0px',
      sm: '4px',
      DEFAULT: '6px',
      md: '6px',
      lg: '8px',
      xl: '12px',
      '2xl': '16px',
      '3xl': '24px',
      full: '9999px',
    },
    fontFamily: {
      sans: ['Inter', 'system-ui', '-apple-system', 'sans-serif'],
      serif: ['Georgia', 'Cambria', 'Times New Roman', 'serif'],
      mono: ['JetBrains Mono', 'Fira Code', 'Consolas', 'monospace'],
    },
    fontSize: {
      xs: ['12px', { lineHeight: '16px' }],
      sm: ['14px', { lineHeight: '20px' }],
      base: ['16px', { lineHeight: '24px' }],
      lg: ['18px', { lineHeight: '28px' }],
      xl: ['20px', { lineHeight: '28px' }],
      '2xl': ['24px', { lineHeight: '32px' }],
      '3xl': ['30px', { lineHeight: '36px' }],
      '4xl': ['36px', { lineHeight: '40px' }],
      '5xl': ['48px', { lineHeight: '1' }],
      '6xl': ['60px', { lineHeight: '1' }],
      '7xl': ['72px', { lineHeight: '1' }],
      '8xl': ['96px', { lineHeight: '1' }],
      '9xl': ['128px', { lineHeight: '1' }],
    },
    fontWeight: {
      thin: '100',
      extralight: '200',
      light: '300',
      normal: '400',
      medium: '500',
      semibold: '600',
      bold: '700',
      extrabold: '800',
      black: '900',
    },
    boxShadow: {
      xs: '0 1px 2px rgb(0 0 0 / 0.05)',
      sm: '0 1px 3px rgb(0 0 0 / 0.1), 0 1px 2px rgb(0 0 0 / 0.06)',
      DEFAULT: '0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -1px rgb(0 0 0 / 0.06)',
      md: '0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -1px rgb(0 0 0 / 0.06)',
      lg: '0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -2px rgb(0 0 0 / 0.05)',
      xl: '0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -5px rgb(0 0 0 / 0.04)',
      '2xl': '0 25px 50px -12px rgb(0 0 0 / 0.25)',
      inner: 'inset 0 2px 4px rgb(0 0 0 / 0.05)',
      none: 'none',
    },
    transitionDuration: {
      0: '0ms',
      75: '75ms',
      100: '100ms',
      150: '150ms',
      200: '200ms',
      300: '300ms',
      500: '500ms',
      700: '700ms',
      1000: '1000ms',
    },
    transitionTimingFunction: {
      linear: 'linear',
      in: 'cubic-bezier(0.4, 0, 1, 1)',
      out: 'cubic-bezier(0, 0, 0.2, 1)',
      'in-out': 'cubic-bezier(0.4, 0, 0.2, 1)',
    },
    zIndex: {
      auto: 'auto',
      0: '0',
      10: '10',
      20: '20',
      30: '30',
      40: '40',
      50: '50',
    },
    screens: {
      sm: '640px',
      md: '768px',
      lg: '1024px',
      xl: '1280px',
      '2xl': '1536px',
    },
    extend: {
      // Estensioni personalizzate qui
    },
  },
  plugins: [],
};
```

## 8.3 CSS Variables per Semantic Tokens

```css
/* globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    /* Surface colors */
    --surface-default: theme('colors.white');
    --surface-muted: theme('colors.gray.50');
    --surface-subtle: theme('colors.gray.100');
    --surface-emphasis: theme('colors.gray.200');
    --surface-inverse: theme('colors.gray.900');
    --surface-brand: theme('colors.blue.500');
    
    /* Text colors */
    --text-default: theme('colors.gray.900');
    --text-muted: theme('colors.gray.600');
    --text-subtle: theme('colors.gray.500');
    --text-placeholder: theme('colors.gray.400');
    --text-inverse: theme('colors.white');
    --text-brand: theme('colors.blue.600');
    
    /* Border colors */
    --border-default: theme('colors.gray.200');
    --border-muted: theme('colors.gray.100');
    --border-emphasis: theme('colors.gray.300');
    --border-focus: theme('colors.blue.500');
    
    /* Feedback colors */
    --feedback-success: theme('colors.green.500');
    --feedback-success-muted: theme('colors.green.50');
    --feedback-warning: theme('colors.amber.500');
    --feedback-warning-muted: theme('colors.amber.50');
    --feedback-danger: theme('colors.red.500');
    --feedback-danger-muted: theme('colors.red.50');
    --feedback-info: theme('colors.blue.500');
    --feedback-info-muted: theme('colors.blue.50');
  }
  
  .dark {
    --surface-default: theme('colors.gray.900');
    --surface-muted: theme('colors.gray.800');
    --surface-subtle: theme('colors.gray.700');
    --surface-emphasis: theme('colors.gray.600');
    --surface-inverse: theme('colors.gray.50');
    
    --text-default: theme('colors.gray.50');
    --text-muted: theme('colors.gray.300');
    --text-subtle: theme('colors.gray.400');
    --text-placeholder: theme('colors.gray.500');
    --text-inverse: theme('colors.gray.900');
    --text-brand: theme('colors.blue.400');
    
    --border-default: theme('colors.gray.700');
    --border-muted: theme('colors.gray.800');
    --border-emphasis: theme('colors.gray.600');
    --border-focus: theme('colors.blue.400');
    
    --feedback-success: theme('colors.green.400');
    --feedback-success-muted: #052e16;
    --feedback-warning: theme('colors.amber.400');
    --feedback-warning-muted: #451a03;
    --feedback-danger: theme('colors.red.400');
    --feedback-danger-muted: #450a0a;
    --feedback-info: theme('colors.blue.400');
    --feedback-info-muted: #172554;
  }
}
```

# ============================================================================
# SEZIONE 9: REGOLE DI VALIDAZIONE
# ============================================================================

## 9.1 Regole OBBLIGATORIE

Per garantire affidabilità al 95%, OGNI token DEVE:

### R1: Struttura Valida
```
✅ CORRETTO:
{
  "token-name": {
    "$value": ...,
    "$type": "..."
  }
}

❌ ERRORE:
{
  "token-name": {
    "value": ...,    // Manca $
    "type": "..."    // Manca $
  }
}
```

### R2: Tipi Corretti
```
✅ VALIDI: "color", "dimension", "fontFamily", "fontWeight", 
          "duration", "cubicBezier", "number", "typography",
          "shadow", "border", "gradient", "transition"

❌ INVALIDI: "size", "spacing", "font", "style", "animation"
```

### R3: Formato Colori
```
✅ CORRETTO:
{
  "$value": {
    "colorSpace": "srgb",
    "components": [0.2, 0.4, 0.8]
  }
}

❌ ERRORE:
{
  "$value": "#3366cc"           // No hex diretto
  "$value": "rgb(51, 102, 204)" // No CSS string
}
```

### R4: Formato Dimensioni
```
✅ CORRETTO:
{
  "$value": {
    "value": 16,
    "unit": "px"
  }
}

❌ ERRORE:
{
  "$value": "16px"    // No CSS string
  "$value": 16        // No numero raw
}
```

### R5: Alias Validi
```
✅ CORRETTO:
{
  "$value": "{primitive.color.blue.500}"
}

❌ ERRORE:
{
  "$value": "primitive.color.blue.500"   // Mancano {}
  "$value": "{blue.500}"                  // Path incompleto
  "$value": "{primitive/color/blue/500}" // Usa / invece di .
}
```

### R6: Nomi Token Validi
```
✅ VALIDI: 
"color-primary", "spacing-md", "button-background"

❌ INVALIDI:
"$color"        // Inizia con $
"color.primary" // Contiene .
"color{test}"   // Contiene {}
```

## 9.2 Checklist Pre-Deploy

Prima di usare i token in produzione, verificare:

- [ ] Tutti i token hanno `$value` definito
- [ ] I colori usano formato `colorSpace` + `components`
- [ ] Le dimensioni usano formato `value` + `unit`
- [ ] Gli alias puntano a token esistenti
- [ ] Non ci sono riferimenti circolari
- [ ] I nomi non contengono caratteri proibiti
- [ ] I tipi sono tra quelli ammessi dal W3C DTCG
- [ ] Light/Dark mode hanno gli stessi percorsi token

# ============================================================================
# SEZIONE 10: TOOL E CONVERSIONE
# ============================================================================

## 10.1 Style Dictionary v4

Style Dictionary è lo strumento standard per convertire i token DTCG in output platform-specific:

```bash
npm install style-dictionary@4
```

### config.json:
```json
{
  "source": ["tokens/**/*.tokens.json"],
  "platforms": {
    "css": {
      "transformGroup": "css",
      "buildPath": "build/css/",
      "files": [
        {
          "destination": "variables.css",
          "format": "css/variables"
        }
      ]
    },
    "js": {
      "transformGroup": "js",
      "buildPath": "build/js/",
      "files": [
        {
          "destination": "tokens.js",
          "format": "javascript/es6"
        }
      ]
    },
    "scss": {
      "transformGroup": "scss",
      "buildPath": "build/scss/",
      "files": [
        {
          "destination": "_variables.scss",
          "format": "scss/variables"
        }
      ]
    }
  }
}
```

## 10.2 Tokens Studio for Figma

Per sincronizzare token tra Figma e codice:

1. Installa plugin "Tokens Studio for Figma"
2. Configura sync con Git repository
3. Esporta in formato DTCG
4. Usa Style Dictionary per build

## 10.3 Output Esempio

### CSS Variables:
```css
:root {
  --color-primitive-gray-50: #f9fafb;
  --color-primitive-gray-500: #6c7480;
  --color-primitive-blue-500: #3b82f6;
  --color-semantic-surface-default: var(--color-primitive-white);
  --color-semantic-text-default: var(--color-primitive-gray-900);
  --spacing-primitive-4: 16px;
  --spacing-semantic-inset-md: var(--spacing-primitive-4);
}
```

### JavaScript ES6:
```javascript
export const colorPrimitiveGray50 = "#f9fafb";
export const colorPrimitiveGray500 = "#6c7480";
export const colorPrimitiveBlue500 = "#3b82f6";
export const spacingPrimitive4 = "16px";
```

# ============================================================================
# FINE SPECIFICA
# ============================================================================

"""
RIEPILOGO PER IL MODELLO AI:

1. USA SEMPRE questo documento come riferimento per i design tokens
2. RISPETTA il formato W3C DTCG 2025.10
3. SEGUI l'architettura a 3 livelli: Primitive → Semantic → Component
4. GENERA token solo dei tipi definiti qui
5. APPLICA le regole di validazione prima dell'output
6. USA le scale standard (8px spacing, modular type scale)
7. SUPPORTA light/dark mode con la struttura indicata
8. INTEGRA con Tailwind CSS seguendo gli esempi

AFFIDABILITÀ TARGET: 95% di output corretti e consistenti
"""
