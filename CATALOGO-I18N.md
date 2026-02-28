# CATALOGO-I18N

Catalogo I18N Espanso per Next.js 14 App Router
§ NEXT-INTL COMPLETE
App Router Setup Completo
typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  i18n: {
    locales: ['en', 'it', 'fr', 'ar', 'ja'],
    defaultLocale: 'en',
    localeDetection: true,
  },
  // Per SSG con i18n
  output: 'standalone',
}

module.exports = nextConfig
typescript
// middleware.ts
import createMiddleware from 'next-intl/middleware'
import { routing } from './i18n/routing'

export default createMiddleware(routing)

export const config = {
  // Match all paths except:
  // 1. /api routes
  // 2. /_next (Next.js internals)
  // 3. /_vercel (Vercel internals)
  // 4. Static files (e.g., /favicon.ico)
  matcher: [
    '/((?!api|_next|_vercel|.*\\..*).*)',
    '/(it|fr|ar|ja)/:path*',
  ]
}
typescript
// i18n/routing.ts
import { defineRouting } from 'next-intl/routing'
import { createNavigation } from 'next-intl/navigation'

export const routing = defineRouting({
  // Lista di tutte le lingue supportate
  locales: ['en', 'it', 'fr', 'ar', 'ja'],
  
  // Lingua predefinita
  defaultLocale: 'en',
  
  // Formato delle URL
  localePrefix: 'as-needed', // 'always', 'as-needed', o 'never'
  
  // Dominio specifico per lingua (opzionale)
  domains: [
    {
      domain: 'example.com',
      defaultLocale: 'en',
    },
    {
      domain: 'example.it',
      defaultLocale: 'it',
    },
    {
      domain: 'example.fr',
      defaultLocale: 'fr',
    },
  ],
  
  // Percorsi senza localizzazione
  localeDetection: true,
})

// Esporta le API di navigazione
export const { Link, redirect, usePathname, useRouter } = createNavigation(routing)
typescript
// i18n/request.ts
import { getRequestConfig } from 'next-intl/server'
import { routing } from './routing'

export default getRequestConfig(async ({ requestLocale }) => {
  // Questo viene eseguito su ogni request
  let locale = await requestLocale
  
  // Validazione della lingua
  if (!locale || !routing.locales.includes(locale as any)) {
    locale = routing.defaultLocale
  }
  
  // Caricamento dinamico delle traduzioni
  const messages = (await import(`../locales/${locale}.json`)).default
  
  return {
    locale,
    messages,
    timeZone: 'Europe/Rome',
    now: new Date(),
    
    // Configurazioni specifiche per lingua
    formats: {
      dateTime: {
        short: {
          day: 'numeric',
          month: 'short',
          year: 'numeric'
        }
      },
      number: {
        currency: {
          style: 'currency',
          currency: locale === 'en' ? 'USD' : locale === 'it' ? 'EUR' : 'USD'
        }
      }
    }
  }
})
typescript
// app/[locale]/layout.tsx
import { NextIntlClientProvider } from 'next-intl'
import { getMessages } from 'next-intl/server'
import { routing } from '../../i18n/routing'
import { notFound } from 'next/navigation'
import { ReactNode } from 'react'

type Props = {
  children: ReactNode
  params: { locale: string }
}

export async function generateStaticParams() {
  return routing.locales.map((locale) => ({ locale }))
}

export default async function LocaleLayout({
  children,
  params
}: Props) {
  // Validazione del parametro locale
  const { locale } = await params
  if (!routing.locales.includes(locale as any)) {
    notFound()
  }

  // Caricamento asincrono dei messaggi
  const messages = await getMessages()
  
  // Configurazione RTL
  const direction = locale === 'ar' ? 'rtl' : 'ltr'
  
  return (
    <html lang={locale} dir={direction}>
      <body>
        <NextIntlClientProvider
          locale={locale}
          messages={messages}
          timeZone="Europe/Rome"
          now={new Date()}
        >
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  )
}
Static vs Dynamic Rendering
typescript
// app/[locale]/products/[id]/page.tsx
import { getTranslations } from 'next-intl/server'
import { unstable_setRequestLocale } from 'next-intl/server'
import { generateStaticParams } from '../../layout'

// Forza il rendering statico per lingue specifiche
export const dynamicParams = false

// Genera percorsi statici per tutte le lingue
export async function generateStaticParams() {
  const locales = ['en', 'it', 'fr']
  const products = await fetchProducts()
  
  return locales.flatMap((locale) =>
    products.map((product) => ({
      locale,
      id: product.id.toString(),
    }))
  )
}

export default async function ProductPage({
  params,
}: {
  params: Promise<{ locale: string; id: string }>
}) {
  const { locale, id } = await params
  // Imposta la lingua per il rendering server-side
  unstable_setRequestLocale(locale)
  
  const t = await getTranslations('Products')
  const product = await fetchProduct(id)
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{t('description')}</p>
    </div>
  )
}

// Alternativa: Dynamic rendering con revalidazione
export const revalidate = 3600 // revalida ogni ora
Navigation APIs
typescript
// components/LanguageSwitcher.tsx
'use client'

import { useRouter, usePathname } from 'next/navigation'
import { useLocale } from 'next-intl'
import { routing } from '../i18n/routing'

export default function LanguageSwitcher() {
  const router = useRouter()
  const pathname = usePathname()
  const locale = useLocale()

  const changeLanguage = (newLocale: string) => {
    // Sostituisci la lingua nel pathname
    const newPathname = pathname.replace(`/${locale}`, `/${newLocale}`)
    router.push(newPathname)
    
    // Aggiorna il cookie della lingua
    document.cookie = `NEXT_LOCALE=${newLocale}; path=/; max-age=31536000`
  }

  return (
    <select
      value={locale}
      onChange={(e) => changeLanguage(e.target.value)}
      className="p-2 border rounded"
    >
      {routing.locales.map((loc) => (
        <option key={loc} value={loc}>
          {getLanguageName(loc)}
        </option>
      ))}
    </select>
  )
}

// Utility per link internazionalizzati
import { Link } from '../i18n/routing'

export function InternalizedLink({ href, children, ...props }) {
  return (
    <Link href={href} {...props}>
      {children}
    </Link>
  )
}
useTranslations in Server/Client
typescript
// Server Component
import { getTranslations } from 'next-intl/server'

export default async function ServerPage() {
  const t = await getTranslations('HomePage')
  
  return (
    <div>
      <h1>{t('title')}</h1>
      {/* Traduzioni annidate */}
      <p>{t('sections.about.title')}</p>
    </div>
  )
}

// Client Component
'use client'

import { useTranslations } from 'next-intl'

export default function ClientComponent() {
  const t = useTranslations('Dashboard')
  
  const handleClick = () => {
    const message = t('actions.confirm')
    alert(message)
  }
  
  return (
    <button onClick={handleClick}>
      {t('actions.submit')}
    </button>
  )
}

// Hook personalizzato per traduzioni tipizzate
import { useTranslations } from 'next-intl'

export function useTypedTranslations<Ns extends string>(
  namespace: Ns
) {
  const t = useTranslations(namespace)
  
  return {
    t,
    // Metodi helper tipizzati
    plural: (key: string, count: number) => 
      t(`${key}.${count === 1 ? 'one' : 'other'}`),
    
    withValues: (key: string, values: Record<string, any>) =>
      t.rich(key, values),
  }
}
§ PLURALIZATION ADVANCED
ICU Message Format
json
{
  "messages": {
    "items": "{count, plural, =0 {No items} =1 {One item} other {# items}}",
    "unread": "You have {unreadCount, plural, =0 {no unread messages} =1 {one unread message} other {# unread messages}}",
    
    "ordinal": "This is your {position, selectordinal, one {#st} two {#nd} few {#rd} other {#th}} visit",
    
    "range": "{start, number}–{end, number}",
    
    "gender": "{gender, select, male {He} female {She} other {They}} is online",
    
    "nested": "{count, plural, =0 {No {type, select, apple {apples} orange {oranges} other {fruits}}} =1 {One {type, select, apple {apple} orange {orange} other {fruit}}} other {# {type, plural, =1 {type} other {types}}}}"
  }
}
typescript
// components/AdvancedPluralization.tsx
'use client'

import { useTranslations, useFormatter } from 'next-intl'

export default function AdvancedPluralization() {
  const t = useTranslations('Messages')
  const format = useFormatter()
  
  // Pluralizzazione base
  const items = 5
  const itemsText = t('items', { count: items })
  
  // Pluralizzazione annidata con selezioni
  const userGender = 'female'
  const messageCount = 3
  const complexMessage = t('gender_plural', {
    gender: userGender,
    count: messageCount,
  })
  
  // Numeri ordinali
  const position = 42
  const ordinalText = format.selectOrdinal(position, {
    one: '#st',
    two: '#nd',
    few: '#rd',
    other: '#th',
  })
  
  // Range
  const start = 1000
  const end = 2000
  const rangeText = t('range', { start, end })
  
  // Pluralizzazione con condizioni complesse
  const fruitType = 'apple'
  const fruitCount = 0
  const fruitText = t('fruits', {
    type: fruitType,
    count: fruitCount,
  })
  
  return (
    <div>
      <h3>Advanced Pluralization Examples</h3>
      <p>Items: {itemsText}</p>
      <p>Position: You are {ordinalText} in line</p>
      <p>Range: {rangeText}</p>
      <p>Fruits: {fruitText}</p>
    </div>
  )
}
Implementazione Type-Safe
typescript
// types/icu.d.ts
declare global {
  namespace Formatjs {
    interface IntlConfig {
      locale: string
    }
    
    interface Message {
      id: string
      defaultMessage: string
    }
  }
}

// utils/icu-utils.ts
export class ICUFormatter {
  private locale: string
  
  constructor(locale: string) {
    this.locale = locale
  }
  
  plural(
    count: number,
    options: {
      zero?: string
      one: string
      two?: string
      few?: string
      many?: string
      other: string
    }
  ): string {
    const rules = new Intl.PluralRules(this.locale)
    const pluralCategory = rules.select(count)
    
    switch (pluralCategory) {
      case 'zero': return options.zero || options.other
      case 'one': return options.one
      case 'two': return options.two || options.other
      case 'few': return options.few || options.other
      case 'many': return options.many || options.other
      default: return options.other
    }
  }
  
  select(
    key: string,
    options: Record<string, string>
  ): string {
    return options[key] || options.other || ''
  }
  
  selectOrdinal(
    count: number,
    options: {
      one: string
      two?: string
      few?: string
      many?: string
      other: string
    }
  ): string {
    const rules = new Intl.PluralRules(this.locale, { type: 'ordinal' })
    const pluralCategory = rules.select(count)
    
    switch (pluralCategory) {
      case 'one': return options.one.replace('#', count.toString())
      case 'two': return (options.two || options.other).replace('#', count.toString())
      case 'few': return (options.few || options.other).replace('#', count.toString())
      case 'many': return (options.many || options.other).replace('#', count.toString())
      default: return options.other.replace('#', count.toString())
    }
  }
}
Gestione Plurali Annidati
typescript
// hooks/useAdvancedPlural.ts
import { useMemo } from 'react'
import { useLocale } from 'next-intl'

export function useAdvancedPlural() {
  const locale = useLocale()
  
  const formatter = useMemo(() => {
    const pluralRules = new Intl.PluralRules(locale)
    const ordinalRules = new Intl.PluralRules(locale, { type: 'ordinal' })
    
    return {
      // Pluralizzazione semplice
      plural: (count: number, forms: Record<string, string>) => {
        const rule = pluralRules.select(count)
        return forms[rule] || forms.other
      },
      
      // Pluralizzazione ordinale
      ordinal: (count: number, forms: Record<string, string>) => {
        const rule = ordinalRules.select(count)
        return (forms[rule] || forms.other).replace('#', count.toString())
      },
      
      // Pluralizzazione con genere
      genderedPlural: (
        count: number,
        gender: 'male' | 'female' | 'other',
        forms: Record<string, Record<string, string>>
      ) => {
        const rule = pluralRules.select(count)
        const genderForms = forms[gender] || forms.other
        return genderForms[rule] || genderForms.other
      },
      
      // Range complesso
      range: (start: number, end: number) => {
        const formatter = new Intl.NumberFormat(locale)
        return `${formatter.format(start)}–${formatter.format(end)}`
      },
    }
  }, [locale])
  
  return formatter
}
§ DATE/TIME/NUMBER
Intl.DateTimeFormat
typescript
// utils/date-formatters.ts
export class DateTimeFormatter {
  private locale: string
  private timeZone: string
  
  constructor(locale: string, timeZone?: string) {
    this.locale = locale
    this.timeZone = timeZone || 'UTC'
  }
  
  // Formati predefiniti
  formats = {
    short: { dateStyle: 'short' } as Intl.DateTimeFormatOptions,
    medium: { dateStyle: 'medium' } as Intl.DateTimeFormatOptions,
    long: { dateStyle: 'long' } as Intl.DateTimeFormatOptions,
    full: { dateStyle: 'full' } as Intl.DateTimeFormatOptions,
    
    shortTime: { timeStyle: 'short' } as Intl.DateTimeFormatOptions,
    mediumTime: { timeStyle: 'medium' } as Intl.DateTimeFormatOptions,
    
    dateTime: {
      dateStyle: 'medium',
      timeStyle: 'short',
    } as Intl.DateTimeFormatOptions,
  }
  
  format(
    date: Date | string | number,
    options?: Intl.DateTimeFormatOptions
  ): string {
    const dateObj = date instanceof Date ? date : new Date(date)
    return new Intl.DateTimeFormat(this.locale, {
      timeZone: this.timeZone,
      ...options,
    }).format(dateObj)
  }
  
  formatRange(
    start: Date | string | number,
    end: Date | string | number,
    options?: Intl.DateTimeFormatOptions
  ): string {
    const startDate = start instanceof Date ? start : new Date(start)
    const endDate = end instanceof Date ? end : new Date(end)
    
    const formatter = new Intl.DateTimeFormat(this.locale, {
      timeZone: this.timeZone,
      ...options,
    })
    
    // @ts-ignore - formatRange non è ancora in tutti i browser
    if (typeof formatter.formatRange === 'function') {
      // @ts-ignore
      return formatter.formatRange(startDate, endDate)
    }
    
    // Fallback per browser che non supportano formatRange
    return `${this.format(startDate, options)} – ${this.format(endDate, options)}`
  }
  
  getRelativeTime(
    date: Date | string | number,
    options?: {
      unit?: Intl.RelativeTimeFormatUnit
      style?: Intl.RelativeTimeFormatStyle
    }
  ): string {
    const dateObj = date instanceof Date ? date : new Date(date)
    const now = new Date()
    const diffInMs = dateObj.getTime() - now.getTime()
    
    const rtf = new Intl.RelativeTimeFormat(this.locale, {
      numeric: 'auto',
      style: options?.style || 'long',
    })
    
    const units = [
      { unit: 'year', ms: 31536000000 },
      { unit: 'month', ms: 2628000000 },
      { unit: 'week', ms: 604800000 },
      { unit: 'day', ms: 86400000 },
      { unit: 'hour', ms: 3600000 },
      { unit: 'minute', ms: 60000 },
      { unit: 'second', ms: 1000 },
    ] as const
    
    const absDiff = Math.abs(diffInMs)
    
    for (const { unit, ms } of units) {
      if (absDiff >= ms || unit === 'second') {
        return rtf.format(Math.round(diffInMs / ms), unit)
      }
    }
    
    return rtf.format(0, 'second')
  }
  
  // Utility per timezone
  convertTimeZone(
    date: Date,
    targetTimeZone: string
  ): Date {
    return new Date(
      date.toLocaleString('en-US', { timeZone: targetTimeZone })
    )
  }
  
  // Formattazione avanzata con pattern
  formatWithPattern(
    date: Date | string | number,
    pattern: string
  ): string {
    const dateObj = date instanceof Date ? date : new Date(date)
    
    const formatter = new Intl.DateTimeFormat(this.locale, {
      timeZone: this.timeZone,
      year: pattern.includes('yyyy') ? 'numeric' : undefined,
      month: pattern.includes('MMMM') ? 'long' : 
             pattern.includes('MMM') ? 'short' : 
             pattern.includes('MM') ? '2-digit' : undefined,
      day: pattern.includes('dd') ? '2-digit' : undefined,
      weekday: pattern.includes('EEEE') ? 'long' : 
               pattern.includes('EEE') ? 'short' : undefined,
      hour: pattern.includes('hh') || pattern.includes('HH') ? '2-digit' : undefined,
      minute: pattern.includes('mm') ? '2-digit' : undefined,
      second: pattern.includes('ss') ? '2-digit' : undefined,
      hour12: pattern.includes('hh') ? true : 
              pattern.includes('HH') ? false : undefined,
    })
    
    return formatter.format(dateObj)
  }
}
Intl.NumberFormat
typescript
// utils/number-formatters.ts
export class NumberFormatter {
  private locale: string
  
  constructor(locale: string) {
    this.locale = locale
  }
  
  // Formattazione base
  format(
    value: number,
    options?: Intl.NumberFormatOptions
  ): string {
    return new Intl.NumberFormat(this.locale, options).format(value)
  }
  
  // Formattazione valuta
  formatCurrency(
    amount: number,
    currency: string,
    options?: {
      minimumFractionDigits?: number
      maximumFractionDigits?: number
      currencyDisplay?: 'symbol' | 'code' | 'name'
    }
  ): string {
    return new Intl.NumberFormat(this.locale, {
      style: 'currency',
      currency,
      ...options,
    }).format(amount)
  }
  
  // Formattazione percentuale
  formatPercent(
    value: number,
    options?: {
      minimumFractionDigits?: number
      maximumFractionDigits?: number
    }
  ): string {
    return new Intl.NumberFormat(this.locale, {
      style: 'percent',
      ...options,
    }).format(value / 100)
  }
  
  // Formattazione unità di misura
  formatUnit(
    value: number,
    unit: string,
    options?: Intl.NumberFormatOptions
  ): string {
    return new Intl.NumberFormat(this.locale, {
      style: 'unit',
      unit,
      ...options,
    }).format(value)
  }
  
  // Formattazione notazione scientifica
  formatScientific(
    value: number,
    options?: {
      minimumFractionDigits?: number
      maximumFractionDigits?: number
    }
  ): string {
    return new Intl.NumberFormat(this.locale, {
      notation: 'scientific',
      ...options,
    }).format(value)
  }
  
  // Formattazione compatta
  formatCompact(
    value: number,
    options?: {
      minimumFractionDigits?: number
      maximumFractionDigits?: number
      notation?: 'compact' | 'standard'
    }
  ): string {
    return new Intl.NumberFormat(this.locale, {
      notation: 'compact',
      compactDisplay: 'short',
      ...options,
    }).format(value)
  }
  
  // Range di numeri
  formatRange(
    start: number,
    end: number,
    options?: Intl.NumberFormatOptions
  ): string {
    const formatter = new Intl.NumberFormat(this.locale, options)
    
    // @ts-ignore - formatRange sperimentale
    if (typeof formatter.formatRange === 'function') {
      // @ts-ignore
      return formatter.formatRange(start, end)
    }
    
    return `${formatter.format(start)} – ${formatter.format(end)}`
  }
  
  // Parsing inverso
  parse(
    formattedString: string,
    options?: Intl.NumberFormatOptions
  ): number | null {
    const formatter = new Intl.NumberFormat(this.locale, options)
    const parts = formatter.formatToParts(12345.67)
    
    // Estrai separatori dal formato
    const decimalSeparator = parts.find(p => p.type === 'decimal')?.value || '.'
    const groupSeparator = parts.find(p => p.type === 'group')?.value || ','
    
    // Rimuovi separatori di gruppo e sostituisci separatore decimale
    let cleaned = formattedString
      .replace(new RegExp(`\\${groupSeparator}`, 'g'), '')
      .replace(decimalSeparator, '.')
    
    // Rimuovi caratteri non numerici tranne punto e meno
    cleaned = cleaned.replace(/[^\d.-]/g, '')
    
    const result = parseFloat(cleaned)
    return isNaN(result) ? null : result
  }
}
Intl.RelativeTimeFormat
typescript
// utils/relative-time.ts
export class RelativeTimeFormatter {
  private locale: string
  
  constructor(locale: string) {
    this.locale = locale
  }
  
  private rtf = new Intl.RelativeTimeFormat(this.locale, {
    numeric: 'auto',
    style: 'long',
  })
  
  format(
    value: number,
    unit: Intl.RelativeTimeFormatUnit
  ): string {
    return this.rtf.format(value, unit)
  }
  
  formatFromNow(date: Date | string | number): string {
    const now = new Date()
    const target = date instanceof Date ? date : new Date(date)
    const diffInMs = target.getTime() - now.getTime()
    
    const units = [
      { unit: 'year', ms: 31536000000 },
      { unit: 'month', ms: 2628000000 },
      { unit: 'week', ms: 604800000 },
      { unit: 'day', ms: 86400000 },
      { unit: 'hour', ms: 3600000 },
      { unit: 'minute', ms: 60000 },
      { unit: 'second', ms: 1000 },
    ] as const
    
    const absDiff = Math.abs(diffInMs)
    
    for (const { unit, ms } of units) {
      if (absDiff >= ms || unit === 'second') {
        const value = Math.round(diffInMs / ms)
        return this.format(value, unit)
      }
    }
    
    return this.format(0, 'second')
  }
  
  formatToParts(
    value: number,
    unit: Intl.RelativeTimeFormatUnit
  ): Intl.RelativeTimeFormatPart[] {
    return this.rtf.formatToParts(value, unit)
  }
  
  // Formattazione personalizzata per social media
  formatSocialTime(date: Date | string | number): string {
    const now = new Date()
    const target = date instanceof Date ? date : new Date(date)
    const diffInMs = now.getTime() - target.getTime()
    const diffInSeconds = Math.floor(diffInMs / 1000)
    
    if (diffInSeconds < 60) {
      return this.format(-diffInSeconds, 'second')
    }
    
    const diffInMinutes = Math.floor(diffInSeconds / 60)
    if (diffInMinutes < 60) {
      return this.format(-diffInMinutes, 'minute')
    }
    
    const diffInHours = Math.floor(diffInMinutes / 60)
    if (diffInHours < 24) {
      return this.format(-diffInHours, 'hour')
    }
    
    const diffInDays = Math.floor(diffInHours / 24)
    if (diffInDays < 7) {
      return this.format(-diffInDays, 'day')
    }
    
    const diffInWeeks = Math.floor(diffInDays / 7)
    if (diffInWeeks < 4) {
      return this.format(-diffInWeeks, 'week')
    }
    
    // Per date più vecchie, usa formato data normale
    const formatter = new Intl.DateTimeFormat(this.locale, {
      month: 'short',
      day: 'numeric',
      ...(new Date().getFullYear() !== target.getFullYear() && { year: 'numeric' }),
    })
    
    return formatter.format(target)
  }
}
Integrazione Completa
typescript
// hooks/useIntlFormats.ts
'use client'

import { useLocale, useTimeZone } from 'next-intl'
import { useMemo } from 'react'
import { DateTimeFormatter } from '../utils/date-formatters'
import { NumberFormatter } from '../utils/number-formatters'
import { RelativeTimeFormatter } from '../utils/relative-time'

export function useIntlFormats() {
  const locale = useLocale()
  const timeZone = useTimeZone() || 'UTC'
  
  return useMemo(() => {
    const dateTimeFormatter = new DateTimeFormatter(locale, timeZone)
    const numberFormatter = new NumberFormatter(locale)
    const relativeTimeFormatter = new RelativeTimeFormatter(locale)
    
    return {
      // Date & Time
      formatDate: (
        date: Date | string | number,
        options?: Intl.DateTimeFormatOptions
      ) => dateTimeFormatter.format(date, options),
      
      formatDateTime: (
        date: Date | string | number,
        options?: Intl.DateTimeFormatOptions
      ) => dateTimeFormatter.format(date, {
        dateStyle: 'medium',
        timeStyle: 'short',
        ...options,
      }),
      
      formatTime: (
        date: Date | string | number,
        options?: Intl.DateTimeFormatOptions
      ) => dateTimeFormatter.format(date, {
        timeStyle: 'short',
        ...options,
      }),
      
      formatRelative: (date: Date | string | number) =>
        dateTimeFormatter.getRelativeTime(date),
      
      // Numbers
      formatNumber: (
        value: number,
        options?: Intl.NumberFormatOptions
      ) => numberFormatter.format(value, options),
      
      formatCurrency: (
        amount: number,
        currency: string,
        options?: Intl.NumberFormatOptions
      ) => numberFormatter.formatCurrency(amount, currency, options),
      
      formatPercent: (
        value: number,
        options?: Intl.NumberFormatOptions
      ) => numberFormatter.formatPercent(value, options),
      
      formatCompact: (
        value: number,
        options?: Intl.NumberFormatOptions
      ) => numberFormatter.formatCompact(value, options),
      
      // Relative Time
      relativeTime: (
        value: number,
        unit: Intl.RelativeTimeFormatUnit
      ) => relativeTimeFormatter.format(value, unit),
      
      fromNow: (date: Date | string | number) =>
        relativeTimeFormatter.formatFromNow(date),
      
      socialTime: (date: Date | string | number) =>
        relativeTimeFormatter.formatSocialTime(date),
      
      // Utility
      locale,
      timeZone,
    }
  }, [locale, timeZone])
}
§ RTL SUPPORT COMPLETE
Configurazione CSS Logical Properties
css
/* styles/rtl.css */
:root {
  /* Logical properties for spacing */
  --start: left;
  --end: right;
  --inline-start: left;
  --inline-end: right;
  
  /* Direction-aware shadows */
  --shadow-start: -2px 0 8px rgba(0, 0, 0, 0.1);
  --shadow-end: 2px 0 8px rgba(0, 0, 0, 0.1);
}

[dir="rtl"] {
  --start: right;
  --end: left;
  --inline-start: right;
  --inline-end: left;
  --shadow-start: 2px 0 8px rgba(0, 0, 0, 0.1);
  --shadow-end: -2px 0 8px rgba(0, 0, 0, 0.1);
}

/* Utility classes for RTL */
.rtl-flip {
  transform: scaleX(-1);
}

.text-start {
  text-align: start;
}

.text-end {
  text-align: end;
}

.margin-inline-start {
  margin-inline-start: auto;
}

.padding-inline-end {
  padding-inline-end: 1rem;
}

/* Bi-directional text support */
.bidi-isolate {
  unicode-bidi: isolate;
}

.bidi-override {
  unicode-bidi: bidi-override;
}

/* Scrollbar RTL support */
.rtl-scroll {
  direction: rtl;
}

.rtl-scroll > * {
  direction: ltr;
}
Tailwind RTL Plugin
javascript
// tailwind.config.js
const plugin = require('tailwindcss/plugin')

module.exports = {
  content: [
    './app/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  
  plugins: [
    plugin(function({ addUtilities, addVariant }) {
      // Aggiungi utility per logical properties
      addUtilities({
        '.ms-auto': { 'margin-inline-start': 'auto' },
        '.me-auto': { 'margin-inline-end': 'auto' },
        '.ps-4': { 'padding-inline-start': '1rem' },
        '.pe-4': { 'padding-inline-end': '1rem' },
        '.text-start': { 'text-align': 'start' },
        '.text-end': { 'text-align': 'end' },
        '.float-start': { 'float': 'inline-start' },
        '.float-end': { 'float': 'inline-end' },
        '.border-s-2': { 'border-inline-start-width': '2px' },
        '.border-e-2': { 'border-inline-end-width': '2px' },
        '.rounded-s-lg': {
          'border-start-start-radius': '0.5rem',
          'border-end-start-radius': '0.5rem',
        },
        '.rounded-e-lg': {
          'border-start-end-radius': '0.5rem',
          'border-end-end-radius': '0.5rem',
        },
      })
      
      // Aggiungi varianti per RTL/LTR
      addVariant('rtl', '&:where([dir="rtl"], [dir="rtl"] *)')
      addVariant('ltr', '&:where([dir="ltr"], [dir="ltr"] *)')
      
      // Aggiungi varianti per pseudo-elementi RTL
      addVariant('rtl-hover', ['&:where([dir="rtl"], [dir="rtl"] *):hover'])
      addVariant('ltr-hover', ['&:where([dir="ltr"], [dir="ltr"] *):hover'])
    }),
  ],
  
  theme: {
    extend: {
      // Logical properties extension
      inset: ({ theme }) => ({
        ...theme('spacing'),
        'start': 'var(--inset-start)',
        'end': 'var(--inset-end)',
        'inset-inline-start': 'var(--inset-start)',
        'inset-inline-end': 'var(--inset-end)',
      }),
      
      margin: ({ theme }) => ({
        ...theme('spacing'),
        'inline-start': 'var(--margin-inline-start)',
        'inline-end': 'var(--margin-inline-end)',
      }),
      
      padding: ({ theme }) => ({
        ...theme('spacing'),
        'inline-start': 'var(--padding-inline-start)',
        'inline-end': 'var(--padding-inline-end)',
      }),
      
      // RTL-aware box shadows
      boxShadow: {
        'start': 'var(--shadow-start)',
        'end': 'var(--shadow-end)',
        'start-lg': 'var(--shadow-start-lg)',
        'end-lg': 'var(--shadow-end-lg)',
      },
    },
  },
}
Icon Mirroring
typescript
// components/RTLIcon.tsx
'use client'

import { useLocale } from 'next-intl'
import { ReactNode } from 'react'

interface RTLIconProps {
  icon: ReactNode
  shouldFlip?: boolean
  className?: string
}

const flipIcons = [
  'arrow-left',
  'arrow-right',
  'chevron-left',
  'chevron-right',
  'caret-left',
  'caret-right',
  'double-chevron-left',
  'double-chevron-right',
  'back',
  'forward',
  'next',
  'previous',
]

export default function RTLIcon({
  icon,
  shouldFlip = true,
  className = '',
}: RTLIconProps) {
  const locale = useLocale()
  const isRTL = locale === 'ar' || locale === 'he'
  
  const shouldMirror = shouldFlip && isRTL && 
    (typeof icon === 'object' && icon.props && 
     flipIcons.includes(icon.props['data-icon']))
  
  return (
    <span
      className={`inline-block ${shouldMirror ? 'rtl-flip' : ''} ${className}`}
      dir="ltr" // Force LTR for icons to prevent bidirectional issues
    >
      {icon}
    </span>
  )
}

// Hook per determinare quando flipp

════════════════════════════════════════════════════════════
FIGMA CATALOG: BATCH2-07-I18N
Prompt ID: 7 / 19
Parte: 2
Exported: 2026-02-06T17:21:06.373Z
Characters: 693
════════════════════════════════════════════════════════════

eLinks(
  currentLocale: string,
  locales: string[],
  domains?: Array<{ domain: string; defaultLocale: string }>
): Record<string, string> {
  const alternates: Record<string, string> = {}
  
  for (const locale of locales) {
    let url: string
    
    if (domains) {
      const domainConfig = domains.find(d => d.defaultLocale === locale)
      if (domainConfig) {
        url = `https://${domainConfig.domain}`
      } else {
        url = `https://example.com/${locale}`
      }
    } else {
      url = locale === routing.defaultLocale
        ? 'https://example.com'
        : `https://example.com/${locale}`
    }
    
    // Formato hreflang: language-REGION (es: en-US, it-IT)
   

## § ADVANCED PATTERNS: I18N

### Server Actions con Validazione

```typescript
// app/actions/i18n.ts
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


### I18N - Utility Helper #631

```typescript
// lib/utils/i18n-helper-631.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #788

```typescript
// lib/utils/i18n-helper-788.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #938

```typescript
// lib/utils/i18n-helper-938.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #291

```typescript
// lib/utils/i18n-helper-291.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #75

```typescript
// lib/utils/i18n-helper-75.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #118

```typescript
// lib/utils/i18n-helper-118.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #209

```typescript
// lib/utils/i18n-helper-209.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #329

```typescript
// lib/utils/i18n-helper-329.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #283

```typescript
// lib/utils/i18n-helper-283.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #239

```typescript
// lib/utils/i18n-helper-239.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #886

```typescript
// lib/utils/i18n-helper-886.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #970

```typescript
// lib/utils/i18n-helper-970.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #832

```typescript
// lib/utils/i18n-helper-832.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #174

```typescript
// lib/utils/i18n-helper-174.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #802

```typescript
// lib/utils/i18n-helper-802.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #425

```typescript
// lib/utils/i18n-helper-425.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #361

```typescript
// lib/utils/i18n-helper-361.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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

  getConfig(): Readonly<I18NConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### I18N - Utility Helper #428

```typescript
// lib/utils/i18n-helper-428.ts
import { z } from "zod";

interface I18NConfig {
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

export class I18NProcessor<TInput, TOutput> {
  private config: I18NConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<I18NConfig> = {}) {
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