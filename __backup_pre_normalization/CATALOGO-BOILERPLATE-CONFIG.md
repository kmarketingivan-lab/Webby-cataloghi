# CATALOGO-BOILERPLATE-CONFIG

Catalogo Boilerplate Config Next.js 14 - Espansione
§ NEXT.CONFIG.JS AVANZATO
javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // ========================
  // BASE CONFIGURATION
  // ========================
  reactStrictMode: true,
  swcMinify: true,
  
  // ========================
  // COMPILER OPTIMIZATIONS
  // ========================
  compiler: {
    removeConsole: process.env.NODE_ENV === 'production' ? {
      exclude: ['error', 'warn'],
    } : false,
    styledComponents: {
      displayName: process.env.NODE_ENV !== 'production',
      ssr: true,
      fileName: true,
      pure: true,
      minify: true,
    },
    emotion: {
      sourceMap: process.env.NODE_ENV !== 'production',
      autoLabel: 'dev-only',
      labelFormat: '[local]',
    },
    relay: {
      src: './',
      artifactDirectory: './__generated__',
      language: 'typescript',
    },
  },

  // ========================
  // IMAGE OPTIMIZATIONS
  // ========================
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '**.example.com',
        pathname: '/**',
      },
      {
        protocol: 'https',
        hostname: '**.cloudinary.com',
        pathname: '/**',
      },
      {
        protocol: 'https',
        hostname: '**.amazonaws.com',
        pathname: '/**',
      },
    ],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    formats: ['image/webp'],
    minimumCacheTTL: 60,
    dangerouslyAllowSVG: false,
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",
    unoptimized: process.env.NODE_ENV === 'development',
  },

  // ========================
  // HEADERS SECURITY
  // ========================
  async headers() {
    const securityHeaders = [
      {
        key: 'X-DNS-Prefetch-Control',
        value: 'on',
      },
      {
        key: 'Strict-Transport-Security',
        value: 'max-age=63072000; includeSubDomains; preload',
      },
      {
        key: 'X-XSS-Protection',
        value: '1; mode=block',
      },
      {
        key: 'X-Frame-Options',
        value: 'SAMEORIGIN',
      },
      {
        key: 'Permissions-Policy',
        value: 'camera=(), microphone=(), geolocation=(), interest-cohort=()',
      },
      {
        key: 'X-Content-Type-Options',
        value: 'nosniff',
      },
      {
        key: 'Referrer-Policy',
        value: 'origin-when-cross-origin',
      },
      {
        key: 'Content-Security-Policy',
        value: `
          default-src 'self';
          script-src 'self' 'unsafe-eval' 'unsafe-inline' *.trusted-scripts.com;
          style-src 'self' 'unsafe-inline' *.trusted-styles.com;
          img-src 'self' data: blob: *.trusted-images.com;
          font-src 'self' data:;
          connect-src 'self' *.trusted-api.com;
          frame-src 'self';
          media-src 'self';
          object-src 'none';
          base-uri 'self';
          form-action 'self';
          frame-ancestors 'none';
          block-all-mixed-content;
          upgrade-insecure-requests;
        `.replace(/\s+/g, ' ').trim(),
      },
    ];

    return [
      {
        source: '/:path*',
        headers: securityHeaders,
      },
      {
        source: '/api/:path*',
        headers: [
          { key: 'Access-Control-Allow-Credentials', value: 'true' },
          { key: 'Access-Control-Allow-Origin', value: '*' },
          { key: 'Access-Control-Allow-Methods', value: 'GET,OPTIONS,PATCH,DELETE,POST,PUT' },
          { key: 'Access-Control-Allow-Headers', value: 'X-CSRF-Token, X-Requested-With, Accept, Accept-Version, Content-Length, Content-MD5, Content-Type, Date, X-Api-Version' },
        ],
      },
    ];
  },

  // ========================
  // REDIRECTS AND REWRITES
  // ========================
  async redirects() {
    return [
      {
        source: '/old-blog/:slug',
        destination: '/blog/:slug',
        permanent: true,
      },
      {
        source: '/docs/:version/:slug',
        destination: '/docs/:slug?version=:version',
        permanent: false,
      },
      // Conditional redirects
      ...(process.env.MAINTENANCE_MODE === 'true'
        ? [
            {
              source: '/((?!maintenance).*)',
              destination: '/maintenance',
              permanent: false,
            },
          ]
        : []),
    ];
  },

  async rewrites() {
    return {
      beforeFiles: [
        {
          source: '/admin/:path*',
          destination: '/admin-dashboard/:path*',
        },
      ],
      afterFiles: [
        {
          source: '/api/v1/:path*',
          destination: 'https://api.example.com/v1/:path*',
        },
        {
          source: '/cdn/:path*',
          destination: 'https://cdn.example.com/:path*',
        },
      ],
      fallback: [
        {
          source: '/:path*',
          destination: 'https://old.example.com/:path*',
        },
      ],
    };
  },

  // ========================
  // WEBPACK CUSTOMIZATION
  // ========================
  webpack: (config, { buildId, dev, isServer, webpack, defaultLoaders }) => {
    // Custom aliases
    config.resolve.alias = {
      ...config.resolve.alias,
      '@styles': './src/styles',
      '@lib': './src/lib',
      '@types': './src/types',
      '@stores': './src/stores',
    };

    // Handle specific file extensions
    config.module.rules.push({
      test: /\.svg$/,
      use: [
        {
          loader: '@svgr/webpack',
          options: {
            svgo: true,
            svgoConfig: {
              plugins: [
                { name: 'removeViewBox', active: false },
                { name: 'cleanupIDs', active: true },
              ],
            },
          },
        },
      ],
    });

    // Exclude specific modules from client bundle
    if (!isServer) {
      config.resolve.fallback = {
        ...config.resolve.fallback,
        fs: false,
        net: false,
        tls: false,
        crypto: require.resolve('crypto-browserify'),
        stream: require.resolve('stream-browserify'),
        path: require.resolve('path-browserify'),
      };
    }

    // Bundle analyzer (development only)
    if (process.env.ANALYZE === 'true') {
      const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
      config.plugins.push(
        new BundleAnalyzerPlugin({
          analyzerMode: 'static',
          reportFilename: isServer
            ? '../analyze/server.html'
            : './analyze/client.html',
        })
      );
    }

    // Performance optimizations
    config.performance = {
      ...config.performance,
      maxAssetSize: 300000,
      maxEntrypointSize: 400000,
    };

    return config;
  },

  // ========================
  // TURBOPACK COMPATIBILITY
  // ========================
  experimental: {
    turbo: {
      rules: {
        '*.svg': {
          loaders: ['@svgr/webpack'],
          as: '*.js',
        },
        '*.md': {
          loaders: ['frontmatter-markdown-loader'],
          as: '*.js',
        },
      },
    },
    // Turbopack specific
    turboMode: process.env.TURBOPACK === 'true',
    // Other experiments
    serverComponentsExternalPackages: [
      '@prisma/client',
      '@aws-sdk/client-s3',
      'puppeteer-core',
    ],
    optimizeCss: true,
    scrollRestoration: true,
    workerThreads: true,
    cpus: 4,
    memoryBasedWorkersCount: true,
    legacyBrowsers: false,
    adjustFontFallbacks: true,
    adjustFontFallbacksWithSizeAdjust: true,
    proxyTimeout: 30000,
    instrumentationHook: true,
    serverActions: {
      bodySizeLimit: '2mb',
      allowedOrigins: ['https://*.example.com'],
    },
    // Web Vitals collection
    webVitalsAttribution: ['CLS', 'LCP', 'FCP', 'FID', 'TTFB', 'INP'],
  },

  // ========================
  // I18N & LOCALE
  // ========================
  i18n: {
    locales: ['en', 'it', 'fr', 'de', 'es'],
    defaultLocale: 'en',
    localeDetection: true,
    domains: [
      {
        domain: 'example.com',
        defaultLocale: 'en',
      },
      {
        domain: 'example.it',
        defaultLocale: 'it',
      },
    ],
  },

  // ========================
  // OUTPUT CONFIGURATION
  // ========================
  output: process.env.NEXT_OUTPUT === 'standalone' ? 'standalone' : undefined,
  
  // ========================
  // ASSET PREFIX
  // ========================
  assetPrefix: process.env.ASSET_PREFIX || undefined,

  // ========================
  // CACHING
  // ========================
  generateBuildId: async () => {
    return process.env.GIT_COMMIT_SHA || `build-${Date.now()}`;
  },

  // ========================
  // ENVIRONMENT VARIABLES
  // ========================
  env: {
    APP_VERSION: process.env.npm_package_version,
    BUILD_TIMESTAMP: Date.now().toString(),
    GIT_COMMIT_SHA: process.env.GIT_COMMIT_SHA || '',
  },

  // ========================
  // CUSTOM PAGE EXTENSIONS
  // ========================
  pageExtensions: ['tsx', 'ts', 'jsx', 'js', 'mdx'],

  // ========================
  // MIDDLEWARE
  // ========================
  middleware: {
    location: 'src/middleware',
  },

  // ========================
  // ON DEMAND ENTRY HANDLING
  // ========================
  onDemandEntries: {
    maxInactiveAge: 25 * 1000,
    pagesBufferLength: 5,
  },

  // ========================
  // STATIC OPTIMIZATIONS
  // ========================
  trailingSlash: false,
  skipTrailingSlashRedirect: true,
  skipMiddlewareUrlNormalize: true,
};

// Conditional exports for different environments
module.exports = (phase, { defaultConfig }) => {
  // Plugins
  const withPlugins = require('next-compose-plugins');
  const withBundleAnalyzer = require('@next/bundle-analyzer')({
    enabled: process.env.ANALYZE === 'true',
  });

  const plugins = [withBundleAnalyzer];

  return withPlugins(plugins, nextConfig);
};
§ TYPESCRIPT CONFIG STRICT
json
{
  "compilerOptions": {
    // ========================
    // TYPE CHECKING
    // ========================
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "useUnknownInCatchVariables": true,
    "alwaysStrict": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "allowUnusedLabels": false,
    "allowUnreachableCode": false,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    
    // ========================
    // MODULES
    // ========================
    "baseUrl": ".",
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "isolatedModules": true,
    
    // ========================
    // EMIT
    // ========================
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "removeComments": true,
    "importHelpers": true,
    "newLine": "lf",
    
    // ========================
    // LIBRARIES
    // ========================
    "lib": ["dom", "dom.iterable", "esnext"],
    "target": "es2022",
    
    // ========================
    // JSX
    // ========================
    "jsx": "preserve",
    
    // ========================
    // PATHS & ALIASES
    // ========================
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@components": ["./src/components/index"],
      "@ui/*": ["./src/components/ui/*"],
      "@layout/*": ["./src/components/layout/*"],
      "@lib/*": ["./src/lib/*"],
      "@lib": ["./src/lib/index"],
      "@hooks/*": ["./src/hooks/*"],
      "@hooks": ["./src/hooks/index"],
      "@store/*": ["./src/store/*"],
      "@store": ["./src/store/index"],
      "@api/*": ["./src/api/*"],
      "@api": ["./src/api/index"],
      "@types/*": ["./src/types/*"],
      "@utils/*": ["./src/utils/*"],
      "@utils": ["./src/utils/index"],
      "@styles/*": ["./src/styles/*"],
      "@config/*": ["./src/config/*"],
      "@config": ["./src/config/index"],
      "@middleware/*": ["./src/middleware/*"],
      "@server/*": ["./src/server/*"],
      "@trpc/*": ["./src/server/api/trpc/*"],
      "@public/*": ["./public/*"]
    },
    
    // ========================
    // COMPOSITE PROJECTS
    // ========================
    "composite": true,
    "tsBuildInfoFile": "./node_modules/.cache/tsbuildinfo.json",
    
    // ========================
    // INTEROP CONSTRAINTS
    // ========================
    "allowImportingTsExtensions": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "verbatimModuleSyntax": true,
    
    // ========================
    // ADVANCED
    // ========================
    "incremental": true,
    "noEmit": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    
    // ========================
    // DECLARATION FILES
    // ========================
    "declarationDir": "./dist/types",
    "emitDeclarationOnly": false
  },
  
  // ========================
  // INCLUDE/EXCLUDE
  // ========================
  "include": [
    "src/**/*.ts",
    "src/**/*.tsx",
    "src/**/*.js",
    "src/**/*.jsx",
    "next-env.d.ts",
    ".next/types/**/*.ts",
    "**/*.d.ts",
    "types/**/*.d.ts"
  ],
  
  "exclude": [
    "node_modules",
    ".next",
    "dist",
    "build",
    "coverage",
    "**/*.test.ts",
    "**/*.spec.ts",
    "**/*.e2e.ts",
    "**/__tests__/**",
    "**/__mocks__/**",
    "cypress",
    "playwright",
    "jest.config.*",
    "**/*.stories.*",
    "**/*.mdx"
  ],
  
  // ========================
  // REFERENCES (MONOREPO)
  // ========================
  "references": [
    {
      "path": "../shared/tsconfig.json"
    },
    {
      "path": "../design-system/tsconfig.json"
    }
  ]
}
§ ESLINT FLAT CONFIG (ESLint 9+)
javascript
import js from '@eslint/js';
import globals from 'globals';
import reactHooks from 'eslint-plugin-react-hooks';
import reactRefresh from 'eslint-plugin-react-refresh';
import typescriptEslint from '@typescript-eslint/eslint-plugin';
import typescriptParser from '@typescript-eslint/parser';
import importPlugin from 'eslint-plugin-import';
import jsxA11y from 'eslint-plugin-jsx-a11y';
import promise from 'eslint-plugin-promise';
import unicorn from 'eslint-plugin-unicorn';
import sonarjs from 'eslint-plugin-sonarjs';
import nextPlugin from '@next/eslint-plugin-next';
import eslintConfigPrettier from 'eslint-config-prettier';

/** @type {import('eslint').Linter.FlatConfig[]} */
export default [
  // ========================
  // IGNORES
  // ========================
  {
    ignores: [
      '**/node_modules/',
      '**/.next/',
      '**/dist/',
      '**/build/',
      '**/coverage/',
      '**/public/',
      '**/*.min.js',
      '**/package-lock.json',
      '**/yarn.lock',
      '**/pnpm-lock.yaml',
      '**/next.config.js',
      '**/*.d.ts',
      '**/__generated__/**',
      '**/.turbo/',
      '**/tmp/',
      '**/temp/',
    ],
  },

  // ========================
  // BASE CONFIGURATION
  // ========================
  {
    files: ['**/*.{js,jsx,mjs,cjs,ts,tsx}'],
    languageOptions: {
      ecmaVersion: 'latest',
      sourceType: 'module',
      globals: {
        ...globals.browser,
        ...globals.node,
        ...globals.es2022,
        React: 'readonly',
        JSX: 'readonly',
      },
      parser: typescriptParser,
      parserOptions: {
        ecmaFeatures: {
          jsx: true,
        },
        project: './tsconfig.json',
        tsconfigRootDir: import.meta.dirname,
      },
    },
    plugins: {
      '@typescript-eslint': typescriptEslint,
      'react-hooks': reactHooks,
      'react-refresh': reactRefresh,
      import: importPlugin,
      'jsx-a11y': jsxA11y,
      promise: promise,
      unicorn: unicorn,
      sonarjs: sonarjs,
      '@next/next': nextPlugin,
    },
  },

  // ========================
  // JAVASCRIPT RULES
  // ========================
  {
    rules: {
      ...js.configs.recommended.rules,
    },
  },

  // ========================
  // TYPESCRIPT RULES
  // ========================
  {
    files: ['**/*.{ts,tsx}'],
    rules: {
      // TypeScript specific rules
      '@typescript-eslint/no-unused-vars': [
        'error',
        {
          argsIgnorePattern: '^_',
          varsIgnorePattern: '^_',
          caughtErrorsIgnorePattern: '^_',
        },
      ],
      '@typescript-eslint/no-explicit-any': 'error',
      '@typescript-eslint/explicit-function-return-type': [
        'error',
        {
          allowExpressions: true,
          allowTypedFunctionExpressions: true,
          allowHigherOrderFunctions: true,
          allowDirectConstAssertionInArrowFunctions: true,
        },
      ],
      '@typescript-eslint/explicit-module-boundary-types': 'error',
      '@typescript-eslint/no-non-null-assertion': 'error',
      '@typescript-eslint/consistent-type-imports': [
        'error',
        {
          prefer: 'type-imports',
          disallowTypeAnnotations: false,
        },
      ],
      '@typescript-eslint/consistent-type-exports': 'error',
      '@typescript-eslint/ban-ts-comment': [
        'error',
        {
          'ts-expect-error': 'allow-with-description',
          'ts-ignore': true,
          'ts-nocheck': true,
          'ts-check': false,
          minimumDescriptionLength: 3,
        },
      ],
      '@typescript-eslint/no-unsafe-assignment': 'error',
      '@typescript-eslint/no-unsafe-return': 'error',
      '@typescript-eslint/no-unsafe-call': 'error',
      '@typescript-eslint/no-unsafe-member-access': 'error',
      '@typescript-eslint/no-unsafe-argument': 'error',
      '@typescript-eslint/require-await': 'error',
      '@typescript-eslint/await-thenable': 'error',
      '@typescript-eslint/no-floating-promises': 'error',
      '@typescript-eslint/no-misused-promises': 'error',
      '@typescript-eslint/prefer-as-const': 'error',
      '@typescript-eslint/no-empty-interface': [
        'error',
        {
          allowSingleExtends: true,
        },
      ],
      '@typescript-eslint/no-unnecessary-type-constraint': 'error',
      '@typescript-eslint/no-unnecessary-condition': 'error',
      '@typescript-eslint/prefer-optional-chain': 'error',
      '@typescript-eslint/prefer-nullish-coalescing': 'error',
      '@typescript-eslint/array-type': ['error', { default: 'array' }],
      '@typescript-eslint/no-duplicate-type-constituents': 'error',
      '@typescript-eslint/no-redundant-type-constituents': 'error',
      '@typescript-eslint/no-import-type-side-effects': 'error',
    },
  },

  // ========================
  // REACT & NEXT.JS RULES
  // ========================
  {
    files: ['**/*.{jsx,tsx}'],
    rules: {
      'react-hooks/rules-of-hooks': 'error',
      'react-hooks/exhaustive-deps': 'warn',
      'react-refresh/only-export-components': [
        'warn',
        { allowConstantExport: true },
      ],
      'jsx-a11y/alt-text': 'error',
      'jsx-a11y/anchor-is-valid': 'error',
      'jsx-a11y/aria-props': 'error',
      'jsx-a11y/heading-has-content': 'error',
      'jsx-a11y/html-has-lang': 'error',
      'jsx-a11y/iframe-has-title': 'error',
      'jsx-a11y/img-redundant-alt': 'error',
      'jsx-a11y/no-access-key': 'error',
      'jsx-a11y/no-autofocus': 'error',
      'jsx-a11y/no-distracting-elements': 'error',
      'jsx-a11y/no-redundant-roles': 'error',
      'jsx-a11y/role-has-required-aria-props': 'error',
      'jsx-a11y/role-supports-aria-props': 'error',
      'jsx-a11y/scope': 'error',
      '@next/next/no-html-link-for-pages': 'error',
      '@next/next/no-img-element': 'error',
      '@next/next/no-sync-scripts': 'error',
      '@next/next/no-css-tags': 'error',
      '@next/next/no-page-custom-font': 'warn',
      '@next/next/no-title-in-document-head': 'warn',
      '@next/next/google-font-display': 'warn',
      '@next/next/google-font-preconnect': 'warn',
      '@next/next/no-document-import-in-page': 'error',
      '@next/next/no-head-import-in-document': 'error',
      '@next/next/no-typos': 'warn',
      '@next/next/no-unwanted-polyfillio': 'warn',
    },
  },

  // ========================
  // IMPORT RULES
  // ========================
  {
    rules: {
      'import/order': [
        'error',
        {
          groups: [
            'builtin',
            'external',
            'internal',
            'parent',
            'sibling',
            'index',
            'object',
            'type',
          ],
          pathGroups: [
            {
              pattern: 'react',
              group: 'external',
              position: 'before',
            },
            {
              pattern: 'next/**',
              group: 'external',
              position: 'before',
            },
            {
              pattern: '@/**',
              group: 'internal',
            },
            {
              pattern: '@components/**',
              group: 'internal',
              position: 'after',
            },
            {
              pattern: './*.scss',
              group: 'index',
              position: 'after',
            },
          ],
          pathGroupsExcludedImportTypes: ['react'],
          'newlines-between': 'always',
          alphabetize: {
            order: 'asc',
            caseInsensitive: true,
          },
        },
      ],
      'import/first': 'error',
      'import/no-duplicates': 'error',
      'import/no-unresolved': 'error',
      'import/named': 'error',
      'import/default': 'error',
      'import/namespace': 'error',
      'import/no-absolute-path': 'error',
      'import/no-dynamic-require': 'error',
      'import/no-webpack-loader-syntax': 'error',
      'import/no-self-import': 'error',
      'import/no-cycle': 'error',
      'import/no-useless-path-segments': 'error',
      'import/no-deprecated': 'warn',
      'import/no-extraneous-dependencies': [
        'error',
        {
          devDependencies: [
            '**/*.test.{js,jsx,ts,tsx}',
            '**/*.spec.{js,jsx,ts,tsx}',
            '**/*.stories.{js,jsx,ts,tsx}',
            '**/__tests__/**',
            '**/__mocks__/**',
            '**/jest.config.*',
            '**/jest.setup.*',
            '**/*.config.{js,ts}',
            '**/*.config.*.{js,ts}',
            '**/cypress/**',
            '**/playwright/**',
            '**/eslint.config.js',
          ],
          optionalDependencies: false,
        },
      ],
    },
  },

  // ========================
  // PROMISE RULES
  // ========================
  {
    rules: {
      'promise/always-return': 'error',
      'promise/no-return-wrap': 'error',
      'promise/param-names': 'error',
      'promise/catch-or-return': 'error',
      'promise/no-native': 'off',
      'promise/no-nesting': 'warn',
      'promise/no-promise-in-callback': 'warn',
      'promise/no-callback-in-promise': 'warn',
      'promise/avoid-new': 'off',
      'promise/no-new-statics': 'error',
      'promise/no-return-in-finally': 'warn',
      'promise/valid-params': 'warn',
    },
  },

  // ========================
  // UNICORN RULES
  // ========================
  {
    rules: {
      'unicorn/better-regex': 'error',
      'unicorn/catch-error-name': 'error',
      'unicorn/consistent-function-scoping': 'error',
      'unicorn/custom-error-definition': 'off',
      'unicorn/error-message': 'error',
      'unicorn/escape-case': 'error',
      'unicorn/expiring-todo-comments': 'error',
      'unicorn/explicit-length-check': 'error',
      'unicorn/filename-case': [
        'error',
        {
          cases: {
            kebabCase: true,
            pascalCase: true,
          },
          ignore: ['^README\\.md$'],
        },
      ],
      'unicorn/import-style': 'error',
      'unicorn/new-for-builtins': 'error',
      'unicorn/no-abusive-eslint-disable': 'error',
      'unicorn/no-array-callback-reference': 'error',
      'unicorn/no-array-for-each': 'error',
      'unicorn/no-array-method-this-argument': 'error',
      'unicorn/no-array-push-push': 'error',
      'unicorn/no-console-spaces': 'error',
      'unicorn/no-for-loop': 'error',
      'unicorn/no-hex-escape': 'error',
      'unicorn/no-instanceof-array': 'error',
      'unicorn/no-keyword-prefix': 'off',
      'unicorn/no-lonely-if': 'error',
      'unicorn/no-nested-ternary': 'error',
      'unicorn/no-new-array': 'error',
      'unicorn/no-new-buffer': 'error',
      'unicorn/no-null': 'off',
      'unicorn/no-object-as-default-parameter': 'error',
      'unicorn/no-process-exit': 'error',
      'unicorn/no-static-only-class': 'error',
      'unicorn/no-thenable': 'error',
      'unicorn/no-this-assignment': 'error',
      'unicorn/no-unreadable-array-destructuring': 'error',
      'unicorn/no-unreadable-iife': 'error',
      'unicorn/no-unsafe-regex': 'error',
      'unicorn/no-unused-properties': 'off',
      'unicorn/no-useless-fallback-in-spread': 'error',
      'unicorn/no-useless-length-check': 'error',
      'unicorn/no-useless-spread': 'error',
      'unicorn/no-useless-undefined': 'error',
      'unicorn/no-zero-fractions': 'error',
      'unicorn/number-literal-case': 'error',
      'unicorn/numeric-separators-style': 'error',
      'unicorn/prefer-add-event-listener': 'error',
      'unicorn/prefer-array-find': 'error',
      'unicorn/prefer-array-flat': 'error',
      'unicorn/prefer-array-flat-map': 'error',
      'unicorn/prefer-array-index-of': 'error',
      'unicorn/prefer-array-some': 'error',
      'unicorn/prefer-at': 'error',
      'unicorn/prefer-code-point': 'error',
      'unicorn/prefer-date-now': 'error',
      'unicorn/prefer-default-parameters': 'error',
      'unicorn/prefer-dom-node-append': 'error',
      'unicorn/prefer-dom-node-dataset': 'error',
      'unicorn/prefer-dom-node-remove': 'error',
      'unicorn/prefer-dom-node-text-content': 'error',
      'unicorn/prefer-export-from': 'error',
      'unicorn/prefer-includes': 'error',
      'unicorn/prefer-keyboard-event-key': 'error',
      'unicorn/prefer-math-trunc': 'error',
      'unicorn/prefer-modern-dom-apis': 'error',
      'unicorn/prefer-modern-math-apis': 'error',
      'unicorn/prefer-negative-index': 'error',
      'unicorn/prefer-node-protocol': 'error',
      'unicorn/prefer-number-properties': 'error',
      'unicorn/prefer-object-from-entries': 'error',
      'unicorn/prefer-optional-catch-binding': 'error',
      'unicorn/prefer-prototype-methods': 'error',
      'unicorn/prefer-query-selector': 'error',
      'unicorn/prefer-reflect-apply': 'error',
      'unicorn/prefer-regexp-test': 'error',
      'unicorn/prefer-set-has': 'error',
      'unicorn/prefer-spread': 'error',
      'unicorn/prefer-string-replace-all': 'error',
      'unicorn/prefer-string-slice': 'error',
      'unicorn/prefer-string-starts-ends-with': 'error',
      'unicorn/prefer-string-trim-start-end': 'error',
      'unicorn/prefer-switch': 'error',
      'unicorn/prefer-ternary': 'error',
      'unicorn/prefer-top-level-await': 'error',
      'unicorn/prefer-type-error': 'error',
      'unicorn/prevent-abbreviations': [
        'error',
        {
          replacements: {
            args: false,
            doc: false,
            docs: false,
            env: false,
            e2e: false,
            err: false,
            fn: false,
            func: {
              fn: false,
              function: true,
            },
            i: false,
            idx: false,
            j: false,
            len: false,
            num: false,
              obj: false,
              param: false,
              params: false,
              prev: false,
              prop: false,
              props: false,
              ref: false,
              res: false,
              ret: false,
              src: false,
              temp: false,
              tmp: false,
              val: false,
            },
          },
        },
      ],
      'unicorn/require-array-join-separator': 'error',
      'unicorn/require-number-to-fixed-digits-argument': 'error',
      'unicorn/string-content': 'off',
      'unicorn/switch-case-braces': 'error',
      'unicorn/template-indent': 'error',
      'unicorn/throw-new-error': 'error',
    },
  },

  // ========================
  // SONARJS RULES
  // ========================
  {
    rules: {
      'sonarjs/no-all-duplicated-branches': 'error',
      'sonarjs/no-element-overwrite': 'error',
      'sonarjs/no-extra-arguments': 'error',
      'sonarjs/no-identical-conditions': 'error',
      'sonarjs/no-identical-expressions': 'error',
      'sonarjs/no-ignored-return': 'error',
      'sonarjs/no-one-iteration-loop': 'error',
      'sonarjs/no-use-of-empty-return-value': 'error',
      'sonarjs/cognitive-complexity': ['error', 15],
      'sonarjs/max-switch-cases': ['error', 10],
      'sonarjs/no-collapsible-if': 'error',
      'sonarjs/no-collection-size-mischeck': 'error',
      'sonarjs/no-duplicate-string': ['error', { threshold: 3 }],
      'sonarjs/no-duplicated-branches': 'error',
      'sonarjs/no-identical-functions': 'error',
      'sonarjs/no-inverted-boolean-check': 'error',
      'sonarjs/no-redundant-boolean': 'error',
      'sonarjs/no-small-switch': 'error',
      'sonarjs/no-unused-collection': 'error',
      'sonarjs/no-useless-catch': 'error',
      'sonarjs/prefer-immediate-return': 'error',
      'sonarjs/prefer-object-literal': 'error',
      'sonarjs/prefer-single-boolean-return': 'error',
      'sonarjs/prefer-while': 'error',
    },
  },

  // ========================
  // CUSTOM PROJECT RULES