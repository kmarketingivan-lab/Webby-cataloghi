# CATALOGO-BOILERPLATE-CONFIG-v1

> **Dominio**: Configurazione progetto Next.js 14 production-ready
> **Stack**: Next.js 14, TypeScript, Prisma, tRPC, Tailwind, Vitest
> **Versione**: 1.0
> **Data**: 2026-01-29

---

§ 1. INDICE

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

§ 1. PANORAMICA

§ 1.1 OBIETTIVO
File di configurazione completi e production-ready per un progetto Next.js 14 con stack moderno.

§ 1.2 STACK TECNOLOGICO

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

§ 1.3 MAPPA FILE CONFIGURAZIONE

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

§ 2. PACKAGE.JSON

§ 2.1 SCRIPT DISPONIBILI

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

§ 2.2 DIPENDENZE CORE

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

§ 2.3 RADIX UI COMPONENTS

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

§ 2.4 FILE COMPLETO

json
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

---

§ 3. TYPESCRIPT

§ 3.1 OPZIONI COMPILER

| Opzione | Valore | Descrizione |
|---------|--------|-------------|
| `target` | ES2022 | Target ECMAScript |
| `strict` | true | Strict mode completo |
| `strictNullChecks` | true | Null check rigorosi |
| `noImplicitAny` | true | No implicit any |
| `noUnusedLocals` | true | Errore su variabili non usate |
| `noUnusedParameters` | true | Errore su parametri non usati |
| `moduleResolution` | bundler | Risoluzione moduli |

§ 3.2 PATH ALIASES

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

§ 3.3 FILE COMPLETO

json
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

---

§ 4. TAILWIND CSS

§ 4.1 DESIGN TOKENS - COLORI

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

§ 4.2 BORDER RADIUS

| Token | Valore |
|-------|--------|
| `lg` | `var(--radius)` |
| `md` | `calc(var(--radius) - 2px)` |
| `sm` | `calc(var(--radius) - 4px)` |

§ 4.3 ANIMAZIONI

| Animation | Keyframes | Durata |
|-----------|-----------|--------|
| `accordion-down` | height: 0 → auto | 0.2s ease-out |
| `accordion-up` | height: auto → 0 | 0.2s ease-out |
| `caret-blink` | opacity blink | 1.25s infinite |

§ 4.4 PLUGIN ATTIVI

| Plugin | Scopo |
|--------|-------|
| `tailwindcss-animate` | Animazioni utility |
| `@tailwindcss/forms` | Styling form elements |
| `@tailwindcss/typography` | Prose styling |
| `@tailwindcss/aspect-ratio` | Aspect ratio utilities |

§ 4.5 FILE COMPLETO

typescript
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

---

§ 5. ESLINT & PRETTIER

§ 5.1 ESLINT EXTENDS

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

§ 5.2 PLUGIN ATTIVI

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

§ 5.3 IMPORT SORT GROUPS

| Gruppo | Pattern |
|--------|---------|
| 1 | `^react`, `^next`, `^@?\\w` (external) |
| 2 | `@/components`, `@/lib`, `@/hooks`, etc. (internal) |
| 3 | `^\\.\\.` (parent imports) |
| 4 | `^\\.` (sibling imports) |
| 5 | `.css` files |

§ 5.4 ESLINT CONFIG

json
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

§ 5.5 PRETTIER CONFIG

json
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

---

§ 6. NEXT.JS CONFIG

§ 6.1 OPZIONI PRINCIPALI

| Opzione | Valore | Descrizione |
|---------|--------|-------------|
| `reactStrictMode` | true | Strict mode React |
| `swcMinify` | true | SWC minification |
| `output` | standalone | Per Docker deployments |

§ 6.2 ALLOWED IMAGE HOSTS

| Host | Servizio |
|------|----------|
| `utfs.io` | Uploadthing |
| `lh3.googleusercontent.com` | Google OAuth |
| `avatars.githubusercontent.com` | GitHub OAuth |

§ 6.3 SECURITY HEADERS

| Header | Valore | Scopo |
|--------|--------|-------|
| `X-DNS-Prefetch-Control` | on | DNS prefetch |
| `Strict-Transport-Security` | max-age=63072000 | HSTS |
| `X-XSS-Protection` | 1; mode=block | XSS protection |
| `X-Frame-Options` | SAMEORIGIN | Clickjacking protection |
| `X-Content-Type-Options` | nosniff | MIME sniffing |
| `Referrer-Policy` | origin-when-cross-origin | Referrer control |
| `Permissions-Policy` | camera=(), microphone=() | Feature policy |

§ 6.4 FILE COMPLETO

javascript
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

---

§ 7. VITEST

§ 7.1 CONFIGURAZIONE TEST

| Opzione | Valore | Descrizione |
|---------|--------|-------------|
| `environment` | jsdom | DOM simulation |
| `globals` | true | Global test functions |
| `css` | false | Skip CSS processing |
| `setupFiles` | `./vitest.setup.ts` | Setup file |

§ 7.2 COVERAGE EXCLUDE

| Path | Motivo |
|------|--------|
| `node_modules/` | External packages |
| `src/types/` | Type definitions |
| `src/lib/utils.ts` | Simple utilities |
| `src/app/api/` | Integration tested |
| `src/middleware.ts` | Middleware |
| `src/server/` | tRPC server |
| `prisma/` | Database schema |

§ 7.3 FILE COMPLETO

typescript
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

---

§ 8. ENVIRONMENT

§ 8.1 VARIABILI RICHIESTE

| Variabile | Tipo | Descrizione |
|-----------|------|-------------|
| `NODE_ENV` | string | development/production |
| `NEXT_PUBLIC_APP_URL` | URL | URL applicazione |
| `DATABASE_URL` | URL | Connection string PostgreSQL |
| `DIRECT_URL` | URL | Direct DB connection (migrations) |
| `NEXTAUTH_URL` | URL | URL per NextAuth |
| `NEXTAUTH_SECRET` | string | Secret 32+ chars |

§ 8.2 OAUTH PROVIDERS

| Provider | Client ID | Client Secret |
|----------|-----------|---------------|
| Google | `GOOGLE_CLIENT_ID` | `GOOGLE_CLIENT_SECRET` |
| GitHub | `GITHUB_CLIENT_ID` | `GITHUB_CLIENT_SECRET` |

§ 8.3 SERVIZI ESTERNI

| Servizio | Variabili |
|----------|-----------|
| Resend | `RESEND_API_KEY` |
| Stripe | `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` |
| Uploadthing | `UPLOADTHING_SECRET`, `UPLOADTHING_APP_ID` |

§ 8.4 FILE .ENV.EXAMPLE

bash
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

---

§ 9. GIT & DOCKER

§ 9.1 GITIGNORE CATEGORIES

| Categoria | Pattern |
|-----------|---------|
| Next.js | `.next/`, `out/`, `.vercel/` |
| Node | `node_modules/`, `*.log` |
| Environment | `.env`, `.env.local`, `.env.*.local` |
| Prisma | `prisma/migrations/` |
| IDE | `.vscode/`, `.idea/` |
| Testing | `coverage/`, `.vitest-cache/` |
| OS | `.DS_Store`, `Thumbs.db` |

§ 9.2 DOCKER SERVICES

| Service | Image | Port | Descrizione |
|---------|-------|------|-------------|
| `db` | postgres:16-alpine | 5432 | PostgreSQL database |
| `redis` | redis:7-alpine | 6379 | Cache/sessions |
| `adminer` | adminer | 8080 | DB management UI |

§ 9.3 DOCKER COMPOSE

yaml
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

---

§ 11. CHECKLIST SETUP PROGETTO

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

§ 10. STRATEGY/DECISION TABLES

§ 10.1 DECISION TABLE PER SCELTA DATABASE

| Database | Caratteristiche | Scelta |
|-----------|----------------|--------|
| Prisma    | Tipo: Relazionale, Documentale, Key-Value <br> Scalabilità: Alta <br> Supporto: TypeScript | ✅ Per progetti con requisiti di scalabilità e supporto a più tipi di dati |
| MongoDB   | Tipo: Documentale <br> Scalabilità: Alta <br> Supporto: TypeScript | ✅ Per progetti con dati non strutturati o semi-strutturati |
| PostgreSQL | Tipo: Relazionale <br> Scalabilità: Alta <br> Supporto: TypeScript | ✅ Per progetti con requisiti di atomicità e consistenza dei dati |

§ 10.2 DECISION TABLE PER SCELTA DI AUTH

| Auth | Caratteristiche | Scelta |
|------|----------------|--------|
| NextAuth | Tipo: Autenticazione <br> Scalabilità: Alta <br> Supporto: TypeScript | ✅ Per progetti con requisiti di autenticazione e autorizzazione |
| Auth0    | Tipo: Autenticazione <br> Scalabilità: Alta <br> Supporto: TypeScript | ✅ Per progetti con requisiti di autenticazione e autorizzazione esterni |

§ 11. BEST PRACTICES

§ 11.1 ✅ DO / ❌ DON'T PER CODICE PULITO

| Best Practice | Descrizione |
|---------------|-------------|
| ✅ Utilizza nomi di variabili descrittivi | Utilizza nomi di variabili che descrivono il contenuto della variabile |
| ❌ Non utilizzare abbreviazioni | Evita di utilizzare abbreviazioni per i nomi di variabili e funzioni |
| ✅ Utilizza commenti per spiegare il codice | Utilizza commenti per spiegare il codice e renderlo più leggibile |
| ❌ Non utilizzare codice duplicato | Evita di utilizzare codice duplicato e cerca di riusare il codice esistente |

§ 11.2 ✅ DO / ❌ DON'T PER PERFORMANCE

| Best Practice | Descrizione |
|---------------|-------------|
| ✅ Utilizza caching per migliorare la performance | Utilizza caching per memorizzare i dati e ridurre il carico del server |
| ❌ Non utilizzare query SQL complesse | Evita di utilizzare query SQL complesse e cerca di ottimizzare le query esistenti |
| ✅ Utilizza tecnologie di ottimizzazione della performance | Utilizza tecnologie come Next.js e Prisma per ottimizzare la performance del progetto |

§ 12. PERFORMANCE CONSIDERATIONS

§ 12.1 OTTIMIZZAZIONE DELLA PERFORMANCE DEL DATABASE

typescript
// Esempio di ottimizzazione della performance del database con Prisma
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// Utilizza il caching per memorizzare i dati
prisma.$use(async (params, next) => {
  const cache = await prisma.cache.findUnique({
    where: { key: params.model },
  });

  if (cache) {
    return cache.data;
  }

  const result = await next(params);
  await prisma.cache.create({
    data: { key: params.model, data: result },
  });

  return result;
});

§ 12.2 OTTIMIZZAZIONE DELLA PERFORMANCE DEL SERVER

typescript
// Esempio di ottimizzazione della performance del server con Next.js
import { NextApiRequest, NextApiResponse } from 'next';

const handler = async (req: NextApiRequest, res: NextApiResponse) => {
  // Utilizza il caching per memorizzare i dati
  const cache = await req.cache.get('data');

  if (cache) {
    return res.json(cache);
  }

  const data = await fetchData();
  await req.cache.set('data', data);

  return res.json(data);
};

§ 13. TESTING PATTERNS

§ 13.1 TESTING DEI COMPONENTI PRINCIPALI CON VITEST

typescript
// Esempio di testing dei componenti principali con Vitest
import { describe, expect, it } from 'vitest';
import { render } from '@testing-library/react';
import { MyComponent } from './MyComponent';

describe('MyComponent', () => {
  it('rende il componente correttamente', () => {
    const { getByText } = render(<MyComponent />);
    expect(getByText('Hello World')).toBeInTheDocument();
  });
});

§ 13.2 TESTING DELLE API CON VITEST

typescript
// Esempio di testing delle API con Vitest
import { describe, expect, it } from 'vitest';
import { api } from './api';

describe('API', () => {
  it('ritorna i dati correttamente', async () => {
    const response = await api.get('/data');
    expect(response.status).toBe(200);
    expect(response.data).toEqual({ message: 'Hello World' });
  });
});

§ 14. COMMON PITFALLS & TROUBLESHOOTING

§ 14.1 ERRORI COMUNI DI CONFIGURAZIONE DEL DATABASE

| Errore | Soluzione |
|--------|----------|
| Errore di connessione al database | Verifica la configurazione del database e assicurati di avere i permessi necessari |
| Errore di query SQL | Verifica la sintassi della query e assicurati di avere i dati necessari |

§ 14.2 ERRORI COMUNI DI CONFIGURAZIONE DEL SERVER

| Errore | Soluzione |
|--------|----------|
| Errore di avvio del server | Verifica la configurazione del server e assicurati di avere i permessi necessari |
| Errore di routing | Verifica la configurazione del routing e assicurati di avere le rotte necessarie |

§ 15. MIGRATION/UPGRADE PATTERNS

§ 15.1 MIGRATION DA UNA VERSIONE PRECEDENTE DI NEXT.JS

typescript
// Esempio di migration da una versione precedente di Next.js
import { NextApiRequest, NextApiResponse } from 'next';

const handler = async (req: NextApiRequest, res: NextApiResponse) => {
  // Utilizza il nuovo routing di Next.js
  const { pathname } = req.url;
  const route = pathname.split('/').pop();

  if (route === 'old-route') {
    return res.redirect(301, '/new-route');
  }

  return res.json({ message: 'Hello World' });
};

§ 15.2 UPGRADE DA UNA VERSIONE PRECEDENTE DI PRISMA

typescript
// Esempio di upgrade da una versione precedente di Prisma
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// Utilizza il nuovo schema di Prisma
prisma.$use(async (params, next) => {
  const schema = await prisma.schema.findUnique({
    where: { name: 'my-schema' },
  });

  if (schema) {
    return schema.data;
  }

  const result = await next(params);
  await prisma.schema.create({
    data: { name: 'my-schema', data: result },
  });

  return result;
});

Catalogo Boilerplate Config - Next.js 14 Configuration Suite
1. Prisma Configuration
1.1 Schema Prisma Completo per Progetto SaaS
prisma
Copia
Scarica
// schema.prisma
generator client {
  provider = "prisma-client-js"
  output   = "../node_modules/.prisma/client"
}

datasource db {
  provider = ["postgresql", "mysql"]
  url      = env("DATABASE_URL")
  relationMode = "prisma"
}

enum UserRole {
  USER
  ADMIN
  MODERATOR
}

enum UserStatus {
  ACTIVE
  INACTIVE
  SUSPENDED
  PENDING
}

model User {
  id                    String    @id @default(cuid())
  email                 String    @unique
  emailVerified         DateTime?
  name                  String?
  image                 String?
  role                  UserRole  @default(USER)
  status                UserStatus @default(PENDING)
  accounts              Account[]
  sessions              Session[]
  createdAt             DateTime  @default(now())
  updatedAt             DateTime  @updatedAt
  
  // Relazioni personalizzate
  projects              Project[]
  subscriptions         Subscription[]
  apiKeys               ApiKey[]
  auditLogs             AuditLog[]
  
  @@map("users")
  @@index([email])
  @@index([status])
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String  // "oauth", "email", "credentials"
  provider          String
  providerAccountId String
  refresh_token     String? @db.Text
  access_token      String? @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String? @db.Text
  session_state     String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
  @@map("accounts")
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("sessions")
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
  @@map("verification_tokens")
}

// Modelli business per progetto SaaS
model Project {
  id          String   @id @default(cuid())
  name        String
  slug        String   @unique
  description String?
  ownerId     String
  owner       User     @relation(fields: [ownerId], references: [id])
  members     User[]   @relation("ProjectMembers")
  settings    Json     @default("{}")
  isActive    Boolean  @default(true)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  // Relazioni
  apiKeys     ApiKey[]
  auditLogs   AuditLog[]
  
  @@map("projects")
  @@index([slug])
  @@index([ownerId])
}

model ApiKey {
  id        String   @id @default(cuid())
  name      String
  key       String   @unique
  prefix    String
  projectId String
  project   Project  @relation(fields: [projectId], references: [id])
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  lastUsed  DateTime?
  expiresAt DateTime?
  isActive  Boolean  @default(true)
  createdAt DateTime @default(now())
  
  @@map("api_keys")
  @@index([key])
  @@index([projectId])
}

model AuditLog {
  id          String   @id @default(cuid())
  action      String
  entityType  String
  entityId    String
  userId      String
  user        User     @relation(fields: [userId], references: [id])
  projectId   String?
  project     Project? @relation(fields: [projectId], references: [id])
  details     Json?
  ipAddress   String?
  userAgent   String?
  createdAt   DateTime @default(now())
  
  @@map("audit_logs")
  @@index([userId])
  @@index([entityType, entityId])
  @@index([createdAt])
}
1.2 Configurazione Prisma Client e Seed Script
typescript
Copia
Scarica
// prisma/seed.ts
import { PrismaClient } from '@prisma/client';
import { faker } from '@faker-js/faker';
import bcrypt from 'bcryptjs';

const prisma = new PrismaClient();

async function main() {
  console.log('🧪 Starting seed process...');

  // Clean database
  await prisma.$transaction([
    prisma.auditLog.deleteMany(),
    prisma.apiKey.deleteMany(),
    prisma.project.deleteMany(),
    prisma.verificationToken.deleteMany(),
    prisma.session.deleteMany(),
    prisma.account.deleteMany(),
    prisma.user.deleteMany(),
  ]);

  // Create admin user
  const adminUser = await prisma.user.create({
    data: {
      email: 'admin@example.com',
      name: 'Admin User',
      role: 'ADMIN',
      status: 'ACTIVE',
      emailVerified: new Date(),
    },
  });

  console.log(`✅ Created admin user: ${adminUser.email}`);

  // Create regular users
  const users = [];
  for (let i = 0; i < 20; i++) {
    const user = await prisma.user.create({
      data: {
        email: faker.internet.email(),
        name: faker.person.fullName(),
        role: 'USER',
        status: Math.random() > 0.2 ? 'ACTIVE' : 'INACTIVE',
        emailVerified: Math.random() > 0.3 ? new Date() : null,
      },
    });
    users.push(user);
  }

  console.log(`✅ Created ${users.length} regular users`);

  // Create projects
  const projects = [];
  for (let i = 0; i < 10; i++) {
    const project = await prisma.project.create({
      data: {
        name: faker.company.name(),
        slug: faker.helpers.slugify(faker.company.name()).toLowerCase(),
        description: faker.company.catchPhrase(),
        ownerId: users[i % users.length].id,
        isActive: Math.random() > 0.1,
        settings: {
          theme: faker.helpers.arrayElement(['light', 'dark', 'auto']),
          notifications: {
            email: Math.random() > 0.5,
            push: Math.random() > 0.5,
          },
        },
      },
    });
    projects.push(project);
  }

  console.log(`✅ Created ${projects.length} projects`);

  // Create API keys
  for (const project of projects) {
    await prisma.apiKey.create({
      data: {
        name: 'Production Key',
        key: `sk_${faker.string.alphanumeric(32)}`,
        prefix: 'sk',
        projectId: project.id,
        userId: project.ownerId,
        expiresAt: faker.date.future(),
        isActive: true,
      },
    });
  }

  console.log(`✅ Created API keys for all projects`);

  // Create audit logs
  const actions = ['CREATE', 'UPDATE', 'DELETE', 'LOGIN', 'LOGOUT'];
  for (let i = 0; i < 50; i++) {
    await prisma.auditLog.create({
      data: {
        action: faker.helpers.arrayElement(actions),
        entityType: faker.helpers.arrayElement(['USER', 'PROJECT', 'API_KEY']),
        entityId: faker.string.alphanumeric(10),
        userId: faker.helpers.arrayElement([adminUser.id, ...users.map(u => u.id)]),
        projectId: Math.random() > 0.5 ? faker.helpers.arrayElement(projects.map(p => p.id)) : null,
        details: {
          ip: faker.internet.ip(),
          userAgent: faker.internet.userAgent(),
          metadata: {
            timestamp: new Date().toISOString(),
          },
        },
        ipAddress: faker.internet.ip(),
        userAgent: faker.internet.userAgent(),
      },
    });
  }

  console.log(`✅ Created 50 audit log entries`);
}

main()
  .catch((e) => {
    console.error('❌ Seed failed:', e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
    console.log('✨ Seed completed successfully');
  });
1.3 Configurazione Package.json per Prisma
json
Copia
Scarica
{
  "scripts": {
    "db:generate": "prisma generate",
    "db:push": "prisma db push",
    "db:studio": "prisma studio",
    "db:seed": "ts-node prisma/seed.ts",
    "db:migrate": "prisma migrate dev",
    "db:migrate:deploy": "prisma migrate deploy",
    "db:reset": "prisma migrate reset",
    "db:validate": "prisma validate",
    "dev": "next dev",
    "build": "prisma generate && next build",
    "start": "next start",
    "lint": "next lint"
  },
  "devDependencies": {
    "@faker-js/faker": "^8.3.1",
    "@types/bcryptjs": "^2.4.6",
    "@types/node": "^20.10.0",
    "prisma": "^5.7.0",
    "ts-node": "^10.9.2",
    "typescript": "^5.3.0"
  },
  "dependencies": {
    "@prisma/client": "^5.7.0",
    "@prisma/adapter-neon": "^5.7.0",
    "@prisma/adapter-pg": "^5.7.0",
    "bcryptjs": "^2.4.3"
  }
}
2. Authentication Config
2.1 NextAuth.js v5 Setup Completo
typescript
Copia
Scarica
// lib/auth/config.ts
import { PrismaAdapter } from "@auth/prisma-adapter";
import { NextAuthConfig } from "next-auth";
import Google from "next-auth/providers/google";
import GitHub from "next-auth/providers/github";
import Credentials from "next-auth/providers/credentials";
import { prisma } from "@/lib/prisma";
import bcrypt from "bcryptjs";
import { loginSchema } from "@/lib/validations/auth";

export const authConfig: NextAuthConfig = {
  adapter: PrismaAdapter(prisma),
  session: {
    strategy: "jwt",
    maxAge: 30 * 24 * 60 * 60, // 30 giorni
  },
  pages: {
    signIn: "/auth/login",
    signOut: "/auth/logout",
    error: "/auth/error",
    verifyRequest: "/auth/verify-request",
    newUser: "/auth/onboarding",
  },
  providers: [
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
      allowDangerousEmailAccountLinking: true,
    }),
    GitHub({
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
      allowDangerousEmailAccountLinking: true,
    }),
    Credentials({
      name: "credentials",
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" },
      },
      async authorize(credentials) {
        try {
          const validatedFields = loginSchema.safeParse(credentials);
          
          if (!validatedFields.success) {
            throw new Error("Invalid credentials format");
          }

          const { email, password } = validatedFields.data;
          
          const user = await prisma.user.findUnique({
            where: { email },
            select: {
              id: true,
              email: true,
              name: true,
              password: true,
              role: true,
              status: true,
              emailVerified: true,
            },
          });

          if (!user || !user.password) {
            throw new Error("Invalid credentials");
          }

          const isValidPassword = await bcrypt.compare(password, user.password);
          
          if (!isValidPassword) {
            throw new Error("Invalid credentials");
          }

          if (user.status !== "ACTIVE") {
            throw new Error("Account is not active");
          }

          return {
            id: user.id,
            email: user.email,
            name: user.name,
            role: user.role,
          };
        } catch (error) {
          console.error("Auth error:", error);
          return null;
        }
      },
    }),
  ],
  callbacks: {
    async jwt({ token, user, account, trigger, session }) {
      if (user) {
        token.id = user.id;
        token.role = user.role;
        token.status = user.status;
      }
      
      if (trigger === "update" && session) {
        token.name = session.name;
        token.picture = session.image;
      }
      
      return token;
    },
    async session({ session, token }) {
      if (session.user) {
        session.user.id = token.id as string;
        session.user.role = token.role as string;
        session.user.status = token.status as string;
      }
      
      return session;
    },
    async signIn({ user, account, profile }) {
      // Controllo email dominio consentito
      if (process.env.ALLOWED_DOMAINS) {
        const allowedDomains = process.env.ALLOWED_DOMAINS.split(",");
        const emailDomain = user.email?.split("@")[1];
        
        if (emailDomain && !allowedDomains.includes(emailDomain)) {
          return `/auth/error?error=DomainNotAllowed&domain=${emailDomain}`;
        }
      }
      
      // Blocco utenti inattivi
      const existingUser = await prisma.user.findUnique({
        where: { email: user.email! },
        select: { status: true },
      });
      
      if (existingUser?.status === "SUSPENDED") {
        return `/auth/error?error=AccountSuspended`;
      }
      
      return true;
    },
    async redirect({ url, baseUrl }) {
      // Redirezione personalizzata
      if (url.startsWith("/")) return `${baseUrl}${url}`;
      else if (new URL(url).origin === baseUrl) return url;
      return baseUrl;
    },
  },
  events: {
    async createUser({ user }) {
      // Invio email di benvenuto
      await sendWelcomeEmail(user.email!, user.name);
      
      // Log evento
      await prisma.auditLog.create({
        data: {
          action: "CREATE",
          entityType: "USER",
          entityId: user.id!,
          userId: user.id!,
          details: { type: "user_created" },
        },
      });
    },
    async linkAccount({ user, account, profile }) {
      await prisma.auditLog.create({
        data: {
          action: "LINK_ACCOUNT",
          entityType: "USER",
          entityId: user.id,
          userId: user.id,
          details: {
            provider: account.provider,
            providerAccountId: account.providerAccountId,
          },
        },
      });
    },
  },
};
2.2 Middleware e Route Handlers
typescript
Copia
Scarica
// middleware.ts
import { NextResponse } from 'next/server';
import { getToken } from 'next-auth/jwt';
import type { NextRequest } from 'next/server';

export async function middleware(request: NextRequest) {
  const token = await getToken({
    req: request,
    secret: process.env.NEXTAUTH_SECRET,
  });

  const { pathname } = request.nextUrl;

  // Route protette
  const protectedRoutes = ['/dashboard', '/settings', '/api/protected'];
  const isProtectedRoute = protectedRoutes.some(route => 
    pathname.startsWith(route)
  );

  // Route API pubbliche
  const publicApiRoutes = [
    '/api/auth',
    '/api/health',
    '/api/webhooks',
  ];
  const isPublicApiRoute = publicApiRoutes.some(route => 
    pathname.startsWith(route)
  );

  // Redirect per login
  if (isProtectedRoute && !token) {
    const redirectUrl = new URL('/auth/login', request.url);
    redirectUrl.searchParams.set('callbackUrl', encodeURI(pathname));
    return NextResponse.redirect(redirectUrl);
  }

  // Controllo ruolo per route admin
  const adminRoutes = ['/admin', '/api/admin'];
  const isAdminRoute = adminRoutes.some(route => 
    pathname.startsWith(route)
  );

  if (isAdminRoute && token?.role !== 'ADMIN') {
    return NextResponse.redirect(new URL('/unauthorized', request.url));
  }

  // Rate limiting per API
  if (pathname.startsWith('/api/') && !isPublicApiRoute) {
    const ip = request.ip ?? '127.0.0.1';
    const rateLimitKey = `rate-limit:${ip}:${pathname}`;
    
    // Implementazione rate limiting
    const rateLimit = await checkRateLimit(rateLimitKey);
    
    if (rateLimit.exceeded) {
      return new NextResponse(
        JSON.stringify({ error: 'Too many requests' }),
        {
          status: 429,
          headers: {
            'X-RateLimit-Limit': rateLimit.limit.toString(),
            'X-RateLimit-Remaining': rateLimit.remaining.toString(),
            'X-RateLimit-Reset': rateLimit.reset.toString(),
          },
        }
      );
    }
  }

  // Aggiunta header di sicurezza
  const response = NextResponse.next();
  
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
  
  if (process.env.NODE_ENV === 'production') {
    response.headers.set(
      'Strict-Transport-Security',
      'max-age=31536000; includeSubDomains'
    );
  }

  return response;
}

export const config = {
  matcher: [
    /*
     * Match all request paths except:
     * 1. /api/auth (NextAuth routes)
     * 2. /_next/static (static files)
     * 3. /_next/image (image optimization files)
     * 4. /favicon.ico (favicon file)
     * 5. /public (public files)
     */
    '/((?!api/auth|_next/static|_next/image|favicon.ico|public).*)',
  ],
};
3. Email Configuration
3.1 Setup Resend con React Email
typescript
Copia
Scarica
// lib/email/config.ts
import { Resend } from 'resend';

export const resend = new Resend(process.env.RESEND_API_KEY);

export const EMAIL_CONFIG = {
  from: process.env.EMAIL_FROM || 'Acme <noreply@acme.com>',
  replyTo: process.env.EMAIL_REPLY_TO || 'support@acme.com',
  defaultVariables: {
    companyName: 'Acme Inc',
    companyAddress: '123 Main St, City, Country',
    supportEmail: 'support@acme.com',
  },
} as const;

// Template Email
export const EMAIL_TEMPLATES = {
  WELCOME: 'welcome',
  VERIFICATION: 'verification',
  PASSWORD_RESET: 'password-reset',
  INVITATION: 'invitation',
  NOTIFICATION: 'notification',
  INVOICE: 'invoice',
} as const;

// Queue configuration
export const EMAIL_QUEUE_CONFIG = {
  maxRetries: 3,
  retryDelay: 1000 * 60 * 5, // 5 minuti
  batchSize: 50,
  concurrency: 5,
} as const;

// Rate limiting
export const EMAIL_RATE_LIMITS = {
  perUser: {
    max: 10,
    window: 1000 * 60 * 60, // 1 ora
  },
  global: {
    max: 1000,
    window: 1000 * 60 * 60, // 1 ora
  },
} as const;
3.2 Template React Email e Service
typescript
Copia
Scarica
// lib/email/templates/WelcomeEmail.tsx
import {
  Body,
  Container,
  Column,
  Head,
  Heading,
  Html,
  Img,
  Link,
  Preview,
  Row,
  Section,
  Text,
} from '@react-email/components';
import * as React from 'react';

interface WelcomeEmailProps {
  userName: string;
  userEmail: string;
  verifyLink: string;
  onboardingLink: string;
  supportEmail: string;
}

export const WelcomeEmail = ({
  userName = 'Utente',
  userEmail = 'user@example.com',
  verifyLink = 'https://example.com/verify',
  onboardingLink = 'https://example.com/onboarding',
  supportEmail = 'support@example.com',
}: WelcomeEmailProps) => (
  <Html>
    <Head />
    <Preview>Benvenuto nella nostra piattaforma!</Preview>
    <Body style={main}>
      <Container style={container}>
        <Section style={logoSection}>
          <Img
            src="https://example.com/logo.png"
            width="120"
            height="40"
            alt="Logo"
            style={logo}
          />
        </Section>
        
        <Section style={content}>
          <Heading style={h1}>Benvenuto, {userName}! 👋</Heading>
          
          <Text style={paragraph}>
            Grazie per esserti registrato a <strong>Acme Platform</strong>.
            Siamo felici di averti con noi!
          </Text>
          
          <Section style={buttonContainer}>
            <Row>
              <Column align="center">
                <Link style={button} href={verifyLink}>
                  Verifica Email
                </Link>
              </Column>
            </Row>
          </Section>
          
          <Text style={paragraph}>
            Per iniziare, completa la configurazione del tuo account:
          </Text>
          
          <Section style={steps}>
            <Row>
              <Column style={stepColumn}>
                <Text style={stepNumber}>1</Text>
                <Text style={stepText}>Verifica il tuo indirizzo email</Text>
              </Column>
              <Column style={stepColumn}>
                <Text style={stepNumber}>2</Text>
                <Text style={stepText}>Configura il tuo profilo</Text>
              </Column>
              <Column style={stepColumn}>
                <Text style={stepNumber}>3</Text>
                <Text style={stepText}>Inizia a utilizzare la piattaforma</Text>
              </Column>
            </Row>
          </Section>
          
          <Section style={ctaSection}>
            <Link style={ctaButton} href={onboardingLink}>
              Inizia Onboarding
            </Link>
          </Section>
          
          <Text style={paragraph}>
            Se hai bisogno di aiuto, non esitare a contattare il nostro team di supporto:
          </Text>
          
          <Link style={supportLink} href={`mailto:${supportEmail}`}>
            {supportEmail}
          </Link>
          
          <Text style={footer}>
            Questo è un messaggio automatico, per favore non rispondere a questa email.
          </Text>
        </Section>
      </Container>
    </Body>
  </Html>
);

const main = {
  backgroundColor: '#f6f9fc',
  fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
};

const container = {
  backgroundColor: '#ffffff',
  margin: '0 auto',
  padding: '20px 0 48px',
  marginBottom: '64px',
  borderRadius: '8px',
  boxShadow: '0 4px 12px rgba(0, 0, 0, 0.05)',
};

const logoSection = {
  textAlign: 'center' as const,
  padding: '32px 0',
};

const logo = {
  margin: '0 auto',
};

const content = {
  padding: '0 48px',
};

const h1 = {
  color: '#333',
  fontSize: '32px',
  fontWeight: 'bold',
  textAlign: 'center' as const,
  margin: '32px 0',
};

const paragraph = {
  color: '#666',
  fontSize: '16px',
  lineHeight: '26px',
  margin: '24px 0',
};

const buttonContainer = {
  textAlign: 'center' as const,
  margin: '32px 0',
};

const button = {
  backgroundColor: '#0070f3',
  borderRadius: '6px',
  color: '#fff',
  fontSize: '16px',
  fontWeight: 'bold',
  textDecoration: 'none',
  textAlign: 'center' as const,
  display: 'inline-block',
  padding: '14px 32px',
  margin: '0 auto',
};

const steps = {
  margin: '40px 0',
  padding: '32px',
  backgroundColor: '#f8fafc',
  borderRadius: '8px',
};

const stepColumn = {
  textAlign: 'center' as const,
  padding: '0 16px',
};

const stepNumber = {
  backgroundColor: '#0070f3',
  borderRadius: '50%',
  color: '#fff',
  fontSize: '20px',
  fontWeight: 'bold',
  width: '40px',
  height: '40px',
  lineHeight: '40px',
  margin: '0 auto 16px',
  display: 'inline-block',
};

const stepText = {
  color: '#555',
  fontSize: '14px',
  lineHeight: '20px',
  margin: '0',
};

const ctaSection = {
  textAlign: 'center' as const,
  margin: '40px 0',
};

const ctaButton = {
  backgroundColor: '#10b981',
  borderRadius: '6px',
  color: '#fff',
  fontSize: '16px',
  fontWeight: 'bold',
  textDecoration: 'none',
  padding: '16px 40px',
  display: 'inline-block',
};

const supportLink = {
  color: '#0070f3',
  fontSize: '16px',
  textDecoration: 'none',
  display: 'block',
  textAlign: 'center' as const,
  margin: '24px 0',
};

const footer = {
  color: '#888',
  fontSize: '12px',
  lineHeight: '16px',
  marginTop: '48px',
  textAlign: 'center' as const,
};

export default WelcomeEmail;
4. Payment Configuration
4.1 Stripe Integration Setup Completo
typescript
Copia
Scarica
// lib/stripe/config.ts
import Stripe from 'stripe';

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
  typescript: true,
  appInfo: {
    name: 'Acme Platform',
    version: '0.1.0',
  },
});

export const STRIPE_CONFIG = {
  webhookSecret: process.env.STRIPE_WEBHOOK_SECRET!,
  successUrl: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard/billing/success`,
  cancelUrl: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard/billing/cancel`,
  billingPortalUrl: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard/billing/portal`,
  
  // Product & Price IDs
  products: {
    basic: {
      monthly: process.env.STRIPE_PRICE_ID_BASIC_MONTHLY!,
      yearly: process.env.STRIPE_PRICE_ID_BASIC_YEARLY!,
    },
    pro: {
      monthly: process.env.STRIPE_PRICE_ID_PRO_MONTHLY!,
      yearly: process.env.STRIPE_PRICE_ID_PRO_YEARLY!,
    },
    enterprise: {
      monthly: process.env.STRIPE_PRICE_ID_ENTERPRISE_MONTHLY!,
      yearly: process.env.STRIPE_PRICE_ID_ENTERPRISE_YEARLY!,
    },
  },
  
  // Subscription settings
  subscription: {
    trialPeriodDays: 14,
    paymentCollection: 'if_required',
    allowPromotionCodes: true,
  },
  
  // Tax settings
  tax: {
    automaticTax: {
      enabled: true,
    },
    taxBehavior: 'exclusive',
  },
} as const;

// Price configuration per tier
export const PRICING_TIERS = [
  {
    id: 'basic',
    name: 'Basic',
    description: 'Per piccoli progetti',
    monthly: {
      price: 9.99,
      stripePriceId: STRIPE_CONFIG.products.basic.monthly,
    },
    yearly: {
      price: 99.99,
      stripePriceId: STRIPE_CONFIG.products.basic.yearly,
      discount: 17,
    },
    features: [
      'Fino a 3 progetti',
      '10.000 richieste API/mese',
      'Supporto email',
      '7 giorni storico dati',
    ],
    limits: {
      projects: 3,
      apiRequests: 10000,
      teamMembers: 1,
      dataRetention: 7,
    },
  },
  {
    id: 'pro',
    name: 'Professional',
    description: 'Per team e business',
    monthly: {
      price: 29.99,
      stripePriceId: STRIPE_CONFIG.products.pro.monthly,
    },
    yearly: {
      price: 299.99,
      stripePriceId: STRIPE_CONFIG.products.pro.yearly,
      discount: 17,
    },
    features: [
      'Fino a 10 progetti',
      '100.000 richieste API/mese',
      'Supporto prioritario',
      '30 giorni storico dati',
      'Team fino a 5 membri',
    ],
    limits: {
      projects: 10,
      apiRequests: 100000,
      teamMembers: 5,
      dataRetention: 30,
    },
  },
  {
    id: 'enterprise',
    name: 'Enterprise',
    description: 'Soluzione personalizzata',
    monthly: {
      price: 99.99,
      stripePriceId: STRIPE_CONFIG.products.enterprise.monthly,
    },
    yearly: {
      price: 999.99,
      stripePriceId: STRIPE_CONFIG.products.enterprise.yearly,
      discount: 17,
    },
    features: [
      'Progetti illimitati',
      'Richieste API illimitate',
      'Supporto 24/7',
      '365 giorni storico dati',
      'Team illimitati',
      'Onboarding dedicato',
    ],
    limits: {
      projects: -1, // illimitato
      apiRequests: -1,
      teamMembers: -1,
      dataRetention: 365,
    },
  },
];
4.2 Webhook Handler e Subscription Management
typescript
Copia
Scarica
// app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { stripe } from '@/lib/stripe/config';
import { prisma } from '@/lib/prisma';
import { STRIPE_CONFIG } from '@/lib/stripe/config';
import { sendEmail } from '@/lib/email/service';
import { InvoiceEmail } from '@/lib/email/templates/InvoiceEmail';

export async function POST(request: NextRequest) {
  const body = await request.text();
  const signature = request.headers.get('stripe-signature')!;

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      STRIPE_CONFIG.webhookSecret
    );
  } catch (error: any) {
    console.error('Webhook signature verification failed:', error.message);
    return NextResponse.json(
      { error: `Webhook Error: ${error.message}` },
      { status: 400 }
    );
  }

  console.log(`Received event: ${event.type}`);

  try {
    switch (event.type) {
      case 'checkout.session.completed':
        await handleCheckoutSessionCompleted(event.data.object as Stripe.Checkout.Session);
        break;
        
      case 'customer.subscription.created':
        await handleSubscriptionCreated(event.data.object as Stripe.Subscription);
        break;
        
      case 'customer.subscription.updated':
        await handleSubscriptionUpdated(event.data.object as Stripe.Subscription);
        break;
        
      case 'customer.subscription.deleted':
        await handleSubscriptionDeleted(event.data.object as Stripe.Subscription);
        break;
        
      case 'invoice.paid':
        await handleInvoicePaid(event.data.object as Stripe.Invoice);
        break;
        
      case 'invoice.payment_failed':
        await handleInvoicePaymentFailed(event.data.object as Stripe.Invoice);
        break;
        
      case 'payment_intent.succeeded':
        await handlePaymentIntentSucceeded(event.data.object as Stripe.PaymentIntent);
        break;
        
      case 'payment_intent.payment_failed':
        await handlePaymentIntentFailed(event.data.object as Stripe.PaymentIntent);
        break;
        
      default:
        console.log(`Unhandled event type: ${event.type}`);
    }

    return NextResponse.json({ received: true });
  } catch (error) {
    console.error('Error handling webhook:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

async function handleCheckoutSessionCompleted(session: Stripe.Checkout.Session) {
  const customerId = session.customer as string;
  const subscriptionId = session.subscription as string;
  const userId = session.client_reference_id;
  
  if (!userId) {
    throw new Error('Missing user ID in session');
  }

  // Recupera i dettagli della subscription
  const subscription = await stripe.subscriptions.retrieve(subscriptionId, {
    expand: ['items.data.price.product'],
  });

  const priceId = subscription.items.data[0].price.id;
  const tier = getTierFromPriceId(priceId);
  
  // Aggiorna l'utente nel database
  await prisma.$transaction([
    prisma.user.update({
      where: { id: userId },
      data: {
        stripeCustomerId: customerId,
        stripeSubscriptionId: subscriptionId,
        subscriptionStatus: subscription.status,
        subscriptionTier: tier,
        subscriptionCurrentPeriodStart: new Date(subscription.current_period_start * 1000),
        subscriptionCurrentPeriodEnd: new Date(subscription.current_period_end * 1000),
        subscriptionCancelAtPeriodEnd: subscription.cancel_at_period_end,
      },
    }),
    
    prisma.auditLog.create({
      data: {
        action: 'SUBSCRIPTION_CREATED',
        entityType: 'USER',
        entityId: userId,
        userId: userId,
        details: {
          tier,
          priceId,
          subscriptionId,
          customerId,
          amount: subscription.items.data[0].price.unit_amount,
          currency: subscription.items.data[0].price.currency,
        },
      },
    }),
  ]);

  // Invio email di conferma
  const user = await prisma.user.findUnique({
    where: { id: userId },
    select: { email: true, name: true },
  });

  if (user) {
    await sendEmail({
      to: user.email!,
      template: 'subscription-activated',
      data: {
        userName: user.name,
        tierName: tier,
        amount: subscription.items.data[0].price.unit_amount! / 100,
        currency: subscription.items.data[0].price.currency,
        periodEnd: new Date(subscription.current_period_end * 1000),
      },
    });
  }
}

async function handleInvoicePaid(invoice: Stripe.Invoice) {
  const subscriptionId = invoice.subscription as string;
  const customerId = invoice.customer as string;
  
  const user = await prisma.user.findFirst({
    where: {
      OR: [
        { stripeCustomerId: customerId },
        { stripeSubscriptionId: subscriptionId },
      ],
    },
  });

  if (!user) return;

  // Aggiorna lo stato del pagamento
  await prisma.user.update({
    where: { id: user.id },
    data: {
      subscriptionStatus: 'active',
      subscriptionCurrentPeriodEnd: new Date(invoice.period_end * 1000),
    },
  });

  // Crea record di pagamento
  await prisma.payment.create({
    data: {
      userId: user.id,
      stripeInvoice