# CATALOGO-BOILERPLATE-CONFIG-v1

> **Dominio**: Configurazione progetto Next.js 14 production-ready
> **Stack**: Next.js 14, TypeScript, Prisma, tRPC, Tailwind, Vitest
> **Versione**: 1.0
> **Data**: 2026-01-29

---

## INDICE

| Sezione | Descrizione |
|---------|-------------|
| [1. Panoramica](#1-panoramica) | Obiettivi e struttura configurazioni |
| [2. Package.json](#2-packagejson) | Dipendenze e script |
| [3. TypeScript](#3-typescript) | Configurazione tsconfig.json |
| [4. Tailwind CSS](#4-tailwind-css) | Theme e plugin |
| [5. ESLint & Prettier](#5-eslint--prettier) | Linting e formatting |
| [6. Next.js Config](#6-nextjs-config) | Configurazione framework |
| [7. Vitest](#7-vitest) | Testing configuration |
| [8. Environment](#8-environment) | Variabili ambiente |
| [9. Git & Docker](#9-git--docker) | Gitignore e Docker Compose |

---

## 1. PANORAMICA

### 1.1 Obiettivo
File di configurazione completi e production-ready per un progetto Next.js 14 con stack moderno.

### 1.2 Stack Tecnologico

| Categoria | Tecnologia | Versione |
|-----------|------------|----------|
| **Framework** | Next.js | 14.1.0 |
| **Runtime** | React | 18.2.0 |
| **Language** | TypeScript | 5.3.3 |
| **Database** | Prisma | 5.9.0 |
| **API** | tRPC | 10.45.0 |
| **Styling** | Tailwind CSS | 3.4.1 |
| **Testing** | Vitest | 1.2.1 |
| **Auth** | NextAuth | 5.0.0-beta.4 |

### 1.3 Mappa File Configurazione

| File | Scopo | Sezione |
|------|-------|---------|
| `package.json` | Dipendenze e script | [2](#2-packagejson) |
| `tsconfig.json` | TypeScript compiler | [3](#3-typescript) |
| `tailwind.config.ts` | Theme Tailwind | [4](#4-tailwind-css) |
| `.eslintrc.json` | Regole linting | [5](#5-eslint--prettier) |
| `.prettierrc` | Formattazione codice | [5](#5-eslint--prettier) |
| `next.config.js` | Configurazione Next.js | [6](#6-nextjs-config) |
| `vitest.config.ts` | Testing setup | [7](#7-vitest) |
| `.env.example` | Template variabili | [8](#8-environment) |
| `.gitignore` | File da ignorare | [9](#9-git--docker) |
| `docker-compose.yml` | Servizi Docker | [9](#9-git--docker) |

---

## 2. PACKAGE.JSON

### 2.1 Script Disponibili

| Script | Comando | Descrizione |
|--------|---------|-------------|
| `dev` | `next dev` | Server sviluppo |
| `build` | `next build` | Build produzione |
| `start` | `next start` | Server produzione |
| `lint` | `next lint` | Esegui linting |
| `lint:fix` | `next lint --fix` | Fix automatico lint |
| `type-check` | `tsc --noEmit` | Verifica tipi |
| `test` | `vitest` | Esegui test |
| `test:watch` | `vitest --watch` | Test in watch mode |
| `test:coverage` | `vitest run --coverage` | Coverage report |
| `db:generate` | `prisma generate` | Genera Prisma client |
| `db:push` | `prisma db push` | Push schema a DB |
| `db:migrate` | `prisma migrate dev` | Esegui migration |
| `db:studio` | `prisma studio` | Apri Prisma Studio |
| `db:seed` | `ts-node prisma/seed.ts` | Seed database |

### 2.2 Dipendenze Core

| Package | Versione | Categoria |
|---------|----------|-----------|
| `next` | 14.1.0 | Framework |
| `react` | 18.2.0 | UI |
| `react-dom` | 18.2.0 | UI |
| `@prisma/client` | 5.9.0 | Database |
| `@trpc/client` | 10.45.0 | API |
| `@trpc/server` | 10.45.0 | API |
| `@trpc/react-query` | 10.45.0 | API |
| `@tanstack/react-query` | 5.17.0 | Data fetching |
| `next-auth` | 5.0.0-beta.4 | Auth |
| `zod` | 3.22.4 | Validation |
| `tailwindcss` | 3.4.1 | Styling |

### 2.3 Radix UI Components

| Package | Versione |
|---------|----------|
| `@radix-ui/react-accordion` | 1.1.2 |
| `@radix-ui/react-alert-dialog` | 1.0.5 |
| `@radix-ui/react-avatar` | 1.0.4 |
| `@radix-ui/react-checkbox` | 1.0.4 |
| `@radix-ui/react-dialog` | 1.0.5 |
| `@radix-ui/react-dropdown-menu` | 2.0.6 |
| `@radix-ui/react-label` | 2.0.2 |
| `@radix-ui/react-popover` | 1.0.7 |
| `@radix-ui/react-select` | 1.2.2 |
| `@radix-ui/react-tabs` | 1.0.4 |
| `@radix-ui/react-toast` | 1.1.5 |
| `@radix-ui/react-tooltip` | 1.0.7 |

### 2.4 File Completo

```json
{
  "name": "platform-template",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "lint:fix": "next lint --fix",
    "type-check": "tsc --noEmit",
    "test": "vitest",
    "test:watch": "vitest --watch",
    "test:coverage": "vitest run --coverage",
    "db:generate": "prisma generate",
    "db:push": "prisma db push",
    "db:migrate": "prisma migrate dev",
    "db:studio": "prisma studio",
    "db:seed": "ts-node --compiler-options {\"module\":\"CommonJS\"} prisma/seed.ts",
    "postinstall": "prisma generate"
  },
  "dependencies": {
    "next": "14.1.0",
    "react": "18.2.0",
    "react-dom": "18.2.0",
    "@prisma/client": "5.9.0",
    "@trpc/client": "10.45.0",
    "@trpc/server": "10.45.0",
    "@trpc/react-query": "10.45.0",
    "@trpc/next": "10.45.0",
    "@tanstack/react-query": "5.17.0",
    "next-auth": "5.0.0-beta.4",
    "@auth/prisma-adapter": "1.0.12",
    "bcryptjs": "2.4.3",
    "zod": "3.22.4",
    "tailwindcss": "3.4.1",
    "@radix-ui/react-accordion": "1.1.2",
    "@radix-ui/react-alert-dialog": "1.0.5",
    "@radix-ui/react-avatar": "1.0.4",
    "@radix-ui/react-checkbox": "1.0.4",
    "@radix-ui/react-dialog": "1.0.5",
    "@radix-ui/react-dropdown-menu": "2.0.6",
    "@radix-ui/react-label": "2.0.2",
    "@radix-ui/react-popover": "1.0.7",
    "@radix-ui/react-select": "1.2.2",
    "@radix-ui/react-slot": "1.0.2",
    "@radix-ui/react-tabs": "1.0.4",
    "@radix-ui/react-toast": "1.1.5",
    "@radix-ui/react-tooltip": "1.0.7",
    "class-variance-authority": "0.7.0",
    "clsx": "2.1.0",
    "tailwind-merge": "2.2.0",
    "lucide-react": "0.312.0",
    "react-hook-form": "7.49.3",
    "@hookform/resolvers": "3.3.4",
    "date-fns": "3.2.0",
    "next-themes": "0.2.1",
    "sonner": "1.3.1",
    "resend": "3.1.0",
    "stripe": "14.13.0",
    "@uploadthing/react": "6.2.2",
    "uploadthing": "6.2.2",
    "@t3-oss/env-nextjs": "0.7.1"
  },
  "devDependencies": {
    "typescript": "5.3.3",
    "@types/node": "20.11.0",
    "@types/react": "18.2.48",
    "@types/react-dom": "18.2.18",
    "@types/bcryptjs": "2.4.6",
    "prisma": "5.9.0",
    "eslint": "8.56.0",
    "eslint-config-next": "14.1.0",
    "@typescript-eslint/eslint-plugin": "6.19.0",
    "@typescript-eslint/parser": "6.19.0",
    "vitest": "1.2.1",
    "@testing-library/react": "14.1.2",
    "@testing-library/jest-dom": "6.2.0",
    "@vitejs/plugin-react": "4.2.1",
    "jsdom": "24.0.0",
    "postcss": "8.4.33",
    "autoprefixer": "10.4.17",
    "prettier": "3.2.4",
    "prettier-plugin-tailwindcss": "0.5.11",
    "eslint-plugin-prettier": "5.1.3",
    "eslint-plugin-react": "7.33.2",
    "eslint-plugin-react-hooks": "4.6.0",
    "eslint-plugin-jsx-a11y": "6.8.0",
    "eslint-plugin-import": "2.29.1",
    "eslint-plugin-simple-import-sort": "10.0.0",
    "eslint-plugin-unused-imports": "3.0.0",
    "eslint-plugin-tailwindcss": "3.14.0",
    "eslint-plugin-testing-library": "6.2.0",
    "eslint-plugin-vitest": "0.3.20",
    "ts-node": "10.9.2"
  }
}
```

---

## 3. TYPESCRIPT

### 3.1 Opzioni Compiler

| Opzione | Valore | Descrizione |
|---------|--------|-------------|
| `target` | ES2022 | Target ECMAScript |
| `strict` | true | Strict mode completo |
| `strictNullChecks` | true | Null check rigorosi |
| `noImplicitAny` | true | No implicit any |
| `noUnusedLocals` | true | Errore su variabili non usate |
| `noUnusedParameters` | true | Errore su parametri non usati |
| `moduleResolution` | bundler | Risoluzione moduli |

### 3.2 Path Aliases

| Alias | Path | Uso |
|-------|------|-----|
| `@/*` | `./src/*` | Root src |
| `@/components/*` | `./src/components/*` | Componenti |
| `@/lib/*` | `./src/lib/*` | Utilities |
| `@/hooks/*` | `./src/hooks/*` | Custom hooks |
| `@/types/*` | `./src/types/*` | Type definitions |
| `@/server/*` | `./src/server/*` | Server code |
| `@/styles/*` | `./src/styles/*` | CSS/styles |
| `@/app/*` | `./src/app/*` | App router |
| `@/config/*` | `./src/config/*` | Configurazioni |
| `@/db/*` | `./src/db/*` | Database |
| `@/utils/*` | `./src/utils/*` | Utility functions |
| `@/auth/*` | `./src/auth/*` | Auth modules |

### 3.3 File Completo

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["dom", "dom.iterable", "ES2022"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/lib/*": ["./src/lib/*"],
      "@/hooks/*": ["./src/hooks/*"],
      "@/types/*": ["./src/types/*"],
      "@/server/*": ["./src/server/*"],
      "@/styles/*": ["./src/styles/*"],
      "@/app/*": ["./src/app/*"],
      "@/config/*": ["./src/config/*"],
      "@/db/*": ["./src/db/*"],
      "@/utils/*": ["./src/utils/*"],
      "@/auth/*": ["./src/auth/*"]
    },
    "baseUrl": "."
  },
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts",
    "prisma/seed.ts"
  ],
  "exclude": ["node_modules"]
}
```

---

## 4. TAILWIND CSS

### 4.1 Design Tokens - Colori

| Token | CSS Variable | Uso |
|-------|--------------|-----|
| `primary` | `--primary` | Colore brand principale |
| `secondary` | `--secondary` | Colore secondario |
| `accent` | `--accent` | Accenti e highlight |
| `destructive` | `--destructive` | Azioni distruttive |
| `success` | `--success` | Feedback positivo |
| `warning` | `--warning` | Avvisi |
| `muted` | `--muted` | Elementi attenuati |
| `background` | `--background` | Background pagina |
| `foreground` | `--foreground` | Testo principale |
| `card` | `--card` | Background card |
| `popover` | `--popover` | Background popover |
| `border` | `--border` | Bordi elementi |
| `input` | `--input` | Bordi input |
| `ring` | `--ring` | Focus ring |

### 4.2 Border Radius

| Token | Valore |
|-------|--------|
| `lg` | `var(--radius)` |
| `md` | `calc(var(--radius) - 2px)` |
| `sm` | `calc(var(--radius) - 4px)` |

### 4.3 Animazioni

| Animation | Keyframes | Durata |
|-----------|-----------|--------|
| `accordion-down` | height: 0 → auto | 0.2s ease-out |
| `accordion-up` | height: auto → 0 | 0.2s ease-out |
| `caret-blink` | opacity blink | 1.25s infinite |

### 4.4 Plugin Attivi

| Plugin | Scopo |
|--------|-------|
| `tailwindcss-animate` | Animazioni utility |
| `@tailwindcss/forms` | Styling form elements |
| `@tailwindcss/typography` | Prose styling |
| `@tailwindcss/aspect-ratio` | Aspect ratio utilities |

### 4.5 File Completo

```typescript
import type { Config } from "tailwindcss";
import { fontFamily } from "tailwindcss/defaultTheme";

const config = {
  darkMode: ["class"],
  content: [
    "./pages/**/*.{ts,tsx}",
    "./components/**/*.{ts,tsx}",
    "./app/**/*.{ts,tsx}",
    "./src/**/*.{ts,tsx}",
  ],
  prefix: "",
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: {
        "2xl": "1400px",
      },
    },
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        success: {
          DEFAULT: "hsl(var(--success))",
          foreground: "hsl(var(--success-foreground))",
        },
        warning: {
          DEFAULT: "hsl(var(--warning))",
          foreground: "hsl(var(--warning-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      fontFamily: {
        sans: ["var(--font-sans)", ...fontFamily.sans],
      },
      keyframes: {
        "accordion-down": {
          from: { height: "0" },
          to: { height: "var(--radix-accordion-content-height)" },
        },
        "accordion-up": {
          from: { height: "var(--radix-accordion-content-height)" },
          to: { height: "0" },
        },
        "caret-blink": {
          "0%,70%,100%": { opacity: "1" },
          "20%,50%": { opacity: "0" },
        },
      },
      animation: {
        "accordion-down": "accordion-down 0.2s ease-out",
        "accordion-up": "accordion-up 0.2s ease-out",
        "caret-blink": "caret-blink 1.25s ease-out infinite",
      },
    },
  },
  plugins: [
    require("tailwindcss-animate"),
    require("@tailwindcss/forms"),
    require("@tailwindcss/typography"),
    require("@tailwindcss/aspect-ratio"),
  ],
} satisfies Config;

export default config;
```

---

## 5. ESLINT & PRETTIER

### 5.1 ESLint Extends

| Config | Scopo |
|--------|-------|
| `eslint:recommended` | Regole base ESLint |
| `plugin:@typescript-eslint/recommended` | TypeScript rules |
| `plugin:react/recommended` | React rules |
| `plugin:react-hooks/recommended` | Hooks rules |
| `plugin:jsx-a11y/recommended` | Accessibility |
| `plugin:import/recommended` | Import ordering |
| `plugin:tailwindcss/recommended` | Tailwind rules |
| `plugin:prettier/recommended` | Prettier integration |
| `next/core-web-vitals` | Next.js rules |

### 5.2 Plugin Attivi

| Plugin | Scopo |
|--------|-------|
| `@typescript-eslint` | TypeScript support |
| `react` | React rules |
| `react-hooks` | Hooks rules |
| `jsx-a11y` | Accessibility |
| `import` | Import validation |
| `simple-import-sort` | Auto sort imports |
| `unused-imports` | Remove unused imports |
| `tailwindcss` | Tailwind validation |
| `vitest` | Test rules |
| `testing-library` | Testing rules |

### 5.3 Import Sort Groups

| Gruppo | Pattern |
|--------|---------|
| 1 | `^react`, `^next`, `^@?\\w` (external) |
| 2 | `@/components`, `@/lib`, `@/hooks`, etc. (internal) |
| 3 | `^\\.\\.` (parent imports) |
| 4 | `^\\.` (sibling imports) |
| 5 | `.css` files |

### 5.4 ESLint Config

```json
{
  "root": true,
  "env": {
    "browser": true,
    "es2022": true,
    "node": true
  },
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module",
    "project": ["./tsconfig.json"]
  },
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "plugin:jsx-a11y/recommended",
    "plugin:import/recommended",
    "plugin:import/typescript",
    "plugin:tailwindcss/recommended",
    "plugin:prettier/recommended",
    "next/core-web-vitals"
  ],
  "plugins": [
    "@typescript-eslint",
    "react",
    "react-hooks",
    "jsx-a11y",
    "import",
    "simple-import-sort",
    "unused-imports",
    "tailwindcss",
    "prettier",
    "vitest",
    "testing-library"
  ],
  "rules": {
    "prettier/prettier": ["error", { "endOfLine": "auto" }],
    "react/react-in-jsx-scope": "off",
    "react/prop-types": "off",
    "react/self-closing-comp": ["error", { "component": true, "html": true }],
    "@typescript-eslint/no-unused-vars": ["warn", { "argsIgnorePattern": "^_" }],
    "unused-imports/no-unused-imports": "error",
    "simple-import-sort/imports": "error",
    "simple-import-sort/exports": "error",
    "tailwindcss/no-custom-classname": "off",
    "tailwindcss/classnames-order": "warn",
    "vitest/no-disabled-tests": "warn",
    "vitest/no-focused-tests": "error"
  }
}
```

### 5.5 Prettier Config

```json
{
  "semi": true,
  "singleQuote": false,
  "tabWidth": 2,
  "useTabs": false,
  "trailingComma": "all",
  "printWidth": 100,
  "arrowParens": "always",
  "endOfLine": "auto",
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

---

## 6. NEXT.JS CONFIG

### 6.1 Opzioni Principali

| Opzione | Valore | Descrizione |
|---------|--------|-------------|
| `reactStrictMode` | true | Strict mode React |
| `swcMinify` | true | SWC minification |
| `output` | standalone | Per Docker deployments |

### 6.2 Allowed Image Hosts

| Host | Servizio |
|------|----------|
| `utfs.io` | Uploadthing |
| `lh3.googleusercontent.com` | Google OAuth |
| `avatars.githubusercontent.com` | GitHub OAuth |

### 6.3 Security Headers

| Header | Valore | Scopo |
|--------|--------|-------|
| `X-DNS-Prefetch-Control` | on | DNS prefetch |
| `Strict-Transport-Security` | max-age=63072000 | HSTS |
| `X-XSS-Protection` | 1; mode=block | XSS protection |
| `X-Frame-Options` | SAMEORIGIN | Clickjacking protection |
| `X-Content-Type-Options` | nosniff | MIME sniffing |
| `Referrer-Policy` | origin-when-cross-origin | Referrer control |
| `Permissions-Policy` | camera=(), microphone=() | Feature policy |

### 6.4 File Completo

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  output: "standalone",
  images: {
    remotePatterns: [
      { protocol: "https", hostname: "utfs.io" },
      { protocol: "https", hostname: "lh3.googleusercontent.com" },
      { protocol: "https", hostname: "avatars.githubusercontent.com" },
    ],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
  async headers() {
    return [
      {
        source: "/(.*)",
        headers: [
          { key: "X-DNS-Prefetch-Control", value: "on" },
          { key: "Strict-Transport-Security", value: "max-age=63072000; includeSubDomains; preload" },
          { key: "X-XSS-Protection", value: "1; mode=block" },
          { key: "X-Frame-Options", value: "SAMEORIGIN" },
          { key: "Permissions-Policy", value: "camera=(), microphone=(), geolocation=()" },
          { key: "X-Content-Type-Options", value: "nosniff" },
          { key: "Referrer-Policy", value: "origin-when-cross-origin" },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

---

## 7. VITEST

### 7.1 Configurazione Test

| Opzione | Valore | Descrizione |
|---------|--------|-------------|
| `environment` | jsdom | DOM simulation |
| `globals` | true | Global test functions |
| `css` | false | Skip CSS processing |
| `setupFiles` | `./vitest.setup.ts` | Setup file |

### 7.2 Coverage Exclude

| Path | Motivo |
|------|--------|
| `node_modules/` | External packages |
| `src/types/` | Type definitions |
| `src/lib/utils.ts` | Simple utilities |
| `src/app/api/` | Integration tested |
| `src/middleware.ts` | Middleware |
| `src/server/` | tRPC server |
| `prisma/` | Database schema |

### 7.3 File Completo

```typescript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    setupFiles: ["./vitest.setup.ts"],
    globals: true,
    css: false,
    include: ["./src/**/*.{test,spec}.{ts,tsx}"],
    exclude: ["./node_modules", "./dist", "./.next", "./cypress", "./playwright"],
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"],
      exclude: [
        "node_modules/",
        "src/types/",
        "src/lib/utils.ts",
        "src/app/api/",
        "src/middleware.ts",
        "src/auth.ts",
        "src/server/",
        "src/db/",
        "prisma/",
      ],
    },
  },
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
      "@/components": path.resolve(__dirname, "./src/components"),
      "@/lib": path.resolve(__dirname, "./src/lib"),
      "@/hooks": path.resolve(__dirname, "./src/hooks"),
      "@/types": path.resolve(__dirname, "./src/types"),
      "@/server": path.resolve(__dirname, "./src/server"),
      "@/styles": path.resolve(__dirname, "./src/styles"),
      "@/app": path.resolve(__dirname, "./src/app"),
      "@/config": path.resolve(__dirname, "./src/config"),
      "@/db": path.resolve(__dirname, "./src/db"),
      "@/utils": path.resolve(__dirname, "./src/utils"),
      "@/auth": path.resolve(__dirname, "./src/auth"),
    },
  },
});
```

---

## 8. ENVIRONMENT

### 8.1 Variabili Richieste

| Variabile | Tipo | Descrizione |
|-----------|------|-------------|
| `NODE_ENV` | string | development/production |
| `NEXT_PUBLIC_APP_URL` | URL | URL applicazione |
| `DATABASE_URL` | URL | Connection string PostgreSQL |
| `DIRECT_URL` | URL | Direct DB connection (migrations) |
| `NEXTAUTH_URL` | URL | URL per NextAuth |
| `NEXTAUTH_SECRET` | string | Secret 32+ chars |

### 8.2 OAuth Providers

| Provider | Client ID | Client Secret |
|----------|-----------|---------------|
| Google | `GOOGLE_CLIENT_ID` | `GOOGLE_CLIENT_SECRET` |
| GitHub | `GITHUB_CLIENT_ID` | `GITHUB_CLIENT_SECRET` |

### 8.3 Servizi Esterni

| Servizio | Variabili |
|----------|-----------|
| Resend | `RESEND_API_KEY` |
| Stripe | `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` |
| Uploadthing | `UPLOADTHING_SECRET`, `UPLOADTHING_APP_ID` |

### 8.4 File .env.example

```bash
# Application Environment
NODE_ENV=development
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_VERCEL_ENV=development

# Database
DATABASE_URL="postgresql://user:password@localhost:5432/platform_template?schema=public"
DIRECT_URL="postgresql://user:password@localhost:5432/platform_template?schema=public"

# NextAuth.js
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=YOUR_NEXTAUTH_SECRET_HERE # Generate with `openssl rand -base64 32`

# OAuth Providers
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GITHUB_CLIENT_ID=
GITHUB_CLIENT_SECRET=

# Email Service (Resend)
RESEND_API_KEY=

# Stripe
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=

# Upload Service (Uploadthing)
UPLOADTHING_SECRET=
UPLOADTHING_APP_ID=
```

---

## 9. GIT & DOCKER

### 9.1 Gitignore Categories

| Categoria | Pattern |
|-----------|---------|
| Next.js | `.next/`, `out/`, `.vercel/` |
| Node | `node_modules/`, `*.log` |
| Environment | `.env`, `.env.local`, `.env.*.local` |
| Prisma | `prisma/migrations/` |
| IDE | `.vscode/`, `.idea/` |
| Testing | `coverage/`, `.vitest-cache/` |
| OS | `.DS_Store`, `Thumbs.db` |

### 9.2 Docker Services

| Service | Image | Port | Descrizione |
|---------|-------|------|-------------|
| `db` | postgres:16-alpine | 5432 | PostgreSQL database |
| `redis` | redis:7-alpine | 6379 | Cache/sessions |
| `adminer` | adminer | 8080 | DB management UI |

### 9.3 Docker Compose

```yaml
version: '3.8'

services:
  db:
    image: postgres:16-alpine
    restart: always
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: platform_template
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d platform_template"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    restart: always
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5

  adminer:
    image: adminer
    restart: always
    ports:
      - "8080:8080"
    depends_on:
      - db

volumes:
  db_data:
  redis_data:
```

---

## CHECKLIST SETUP PROGETTO

| Step | Comando/Azione | Status |
|------|----------------|--------|
| 1 | Copiare tutti i file di config | ⬜ |
| 2 | `npm install` | ⬜ |
| 3 | Copiare `.env.example` → `.env` | ⬜ |
| 4 | Configurare variabili ambiente | ⬜ |
| 5 | `docker-compose up -d` (DB) | ⬜ |
| 6 | `npm run db:push` | ⬜ |
| 7 | `npm run db:seed` (opzionale) | ⬜ |
| 8 | `npm run dev` | ⬜ |
| 9 | Verificare lint: `npm run lint` | ⬜ |
| 10 | Verificare types: `npm run type-check` | ⬜ |
| 11 | Eseguire test: `npm run test` | ⬜ |

---

_Integrato da: 03-OUTPUT-BOILERPLATE-CONFIG.md_
_Data integrazione: 2026-01-29 14:51_
_Generato con: Gemini 2.5 Flash / DeepSeek R1_
