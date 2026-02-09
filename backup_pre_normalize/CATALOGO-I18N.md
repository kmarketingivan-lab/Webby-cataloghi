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
   