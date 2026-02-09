# ============================================================================
# CATALOGO UI PATTERN PRIMITIVI v1.0
# ============================================================================
# TIPO: SPECIFICA DETERMINISTICA
# TARGET: Generazione automatica UI per qualsiasi piattaforma
# AFFIDABILITÀ TARGET: 90%
# DATA: Gennaio 2026
# ============================================================================

"""
ISTRUZIONI PER IL MODELLO AI:

1. Questi sono pattern UI PRIMITIVI - componenti fondamentali
2. Si COMBINANO per creare UI per qualsiasi categoria di piattaforma
3. OGNI pattern ha una struttura JSX/HTML definita
4. USA i token dal CATALOGO-DESIGN-TOKEN-SYSTEM-v1.md
5. RISPETTA le varianti e gli stati definiti
6. SEGUI le linee guida di accessibilità (WCAG 2.1 AA)
7. I pattern sono COMPONIBILI tra loro
"""

# ============================================================================
# INDICE CATEGORIE PATTERN
# ============================================================================

"""
1. LAYOUT PATTERNS      - Struttura della pagina
2. NAVIGATION PATTERNS  - Come ci si muove nell'app
3. CONTENT PATTERNS     - Come si presenta il contenuto
4. INPUT PATTERNS       - Come si inseriscono dati
5. FEEDBACK PATTERNS    - Come si comunica con l'utente
6. OVERLAY PATTERNS     - Contenuto sovrapposto
7. DATA DISPLAY PATTERNS - Visualizzazione dati strutturati
8. ACTION PATTERNS      - Come si compiono azioni
"""

# ============================================================================
# SEZIONE 1: LAYOUT PATTERNS
# ============================================================================

## 1.1 APP SHELL

### Descrizione:
Struttura base dell'applicazione che contiene header, sidebar, content area e footer.

### Quando usarlo:
- Applicazioni web complete (dashboard, admin panel, SaaS)
- Qualsiasi app che richiede navigazione persistente

### Varianti:
| Variante | Descrizione |
|----------|-------------|
| `sidebar-left` | Sidebar a sinistra (default) |
| `sidebar-right` | Sidebar a destra |
| `sidebar-both` | Sidebar su entrambi i lati |
| `no-sidebar` | Solo header + content + footer |
| `minimal` | Solo content area |

### Struttura JSX:

```jsx
// APP SHELL - Variante sidebar-left
<div className="app-shell min-h-screen flex flex-col">
  {/* Header - Fixed top */}
  <header className="h-16 border-b border-border-default bg-surface-default 
                     flex items-center px-4 sticky top-0 z-sticky">
    {/* Logo */}
    <div className="flex items-center gap-3">
      <img src={logo} alt="Logo" className="h-8 w-8" />
      <span className="font-semibold text-text-default">{appName}</span>
    </div>
    
    {/* Header Actions */}
    <div className="ml-auto flex items-center gap-2">
      {headerActions}
    </div>
  </header>
  
  {/* Main Container */}
  <div className="flex flex-1">
    {/* Sidebar */}
    <aside className="w-64 border-r border-border-default bg-surface-muted 
                      flex flex-col shrink-0">
      <nav className="flex-1 p-4">
        {sidebarContent}
      </nav>
      <div className="p-4 border-t border-border-default">
        {sidebarFooter}
      </div>
    </aside>
    
    {/* Content Area */}
    <main className="flex-1 bg-surface-subtle overflow-auto">
      <div className="p-6 max-w-7xl mx-auto">
        {children}
      </div>
    </main>
  </div>
  
  {/* Footer (opzionale) */}
  <footer className="h-12 border-t border-border-default bg-surface-default 
                     flex items-center justify-center px-4">
    {footerContent}
  </footer>
</div>
```

### Token Utilizzati:
```
- surface-default, surface-muted, surface-subtle
- border-default
- text-default
- spacing: h-16 (64px header), w-64 (256px sidebar), p-4, p-6, gap-2, gap-3
- z-index: z-sticky
```

### Responsive Behavior:
```jsx
// Mobile: Sidebar diventa drawer
// Tablet: Sidebar collassabile
// Desktop: Sidebar sempre visibile

// Breakpoints:
// < 768px (md): Sidebar hidden, hamburger menu
// 768px - 1024px (lg): Sidebar collapsible (icons only)
// > 1024px: Sidebar expanded
```

---

## 1.2 SIDEBAR LAYOUT

### Descrizione:
Layout con sidebar navigazionale. La sidebar può essere collassabile o fissa.

### Varianti:
| Variante | Larghezza | Comportamento |
|----------|-----------|---------------|
| `expanded` | 256px (w-64) | Sempre espansa |
| `collapsed` | 64px (w-16) | Solo icone |
| `collapsible` | Toggle | Espande/collassa |
| `floating` | 256px | Overlay sul content |
| `rail` | 72px | Stile Material rail |

### Struttura JSX:

```jsx
// SIDEBAR COLLAPSIBLE
const [isCollapsed, setIsCollapsed] = useState(false);

<aside className={cn(
  "h-screen flex flex-col border-r border-border-default bg-surface-default",
  "transition-all duration-normal ease-out",
  isCollapsed ? "w-16" : "w-64"
)}>
  {/* Sidebar Header */}
  <div className="h-16 flex items-center justify-between px-4 border-b border-border-muted">
    {!isCollapsed && <span className="font-semibold">{title}</span>}
    <button 
      onClick={() => setIsCollapsed(!isCollapsed)}
      className="p-2 rounded-md hover:bg-interactive-hover"
      aria-label={isCollapsed ? "Expand sidebar" : "Collapse sidebar"}
    >
      <ChevronIcon className={cn(
        "w-5 h-5 transition-transform",
        isCollapsed && "rotate-180"
      )} />
    </button>
  </div>
  
  {/* Navigation */}
  <nav className="flex-1 overflow-y-auto p-2">
    <ul className="space-y-1">
      {navItems.map(item => (
        <li key={item.id}>
          <a 
            href={item.href}
            className={cn(
              "flex items-center gap-3 px-3 py-2 rounded-md",
              "text-text-muted hover:text-text-default",
              "hover:bg-interactive-hover",
              "transition-colors duration-fast",
              item.isActive && "bg-interactive-selected text-text-brand"
            )}
          >
            <item.icon className="w-5 h-5 shrink-0" />
            {!isCollapsed && <span>{item.label}</span>}
          </a>
        </li>
      ))}
    </ul>
  </nav>
  
  {/* Sidebar Footer */}
  <div className="p-4 border-t border-border-muted">
    {!isCollapsed ? (
      <UserProfile user={user} />
    ) : (
      <Avatar user={user} size="sm" />
    )}
  </div>
</aside>
```

---

## 1.3 SPLIT VIEW

### Descrizione:
Layout diviso in due pannelli (sinistra/destra o sopra/sotto) con divisore ridimensionabile.

### Varianti:
| Variante | Direzione | Uso Tipico |
|----------|-----------|------------|
| `horizontal` | Sinistra/Destra | Master-detail, email client |
| `vertical` | Sopra/Sotto | Code editor, terminal |
| `three-column` | 3 colonne | Email (folders, list, preview) |

### Struttura JSX:

```jsx
// SPLIT VIEW - Horizontal
<div className="flex h-full">
  {/* Left Panel (Master) */}
  <div 
    className="flex flex-col border-r border-border-default"
    style={{ width: `${leftWidth}px` }}
  >
    <div className="flex-1 overflow-auto">
      {leftContent}
    </div>
  </div>
  
  {/* Resizer */}
  <div 
    className="w-1 bg-border-default hover:bg-border-brand cursor-col-resize
               transition-colors"
    onMouseDown={handleResizeStart}
  />
  
  {/* Right Panel (Detail) */}
  <div className="flex-1 flex flex-col overflow-auto">
    {rightContent}
  </div>
</div>
```

---

## 1.4 GRID LAYOUT

### Descrizione:
Sistema di griglia responsive per organizzare contenuti.

### Varianti:
| Variante | Colonne | Uso |
|----------|---------|-----|
| `12-col` | 12 colonne | Layout complessi |
| `auto-fit` | Auto | Card grids responsive |
| `fixed` | N colonne fisse | Gallery, product grid |
| `masonry` | Altezze variabili | Pinterest-style |

### Struttura JSX:

```jsx
// GRID - Auto-fit responsive
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
  {items.map(item => (
    <Card key={item.id}>{item.content}</Card>
  ))}
</div>

// GRID - 12 colonne
<div className="grid grid-cols-12 gap-4">
  <div className="col-span-12 md:col-span-8">{mainContent}</div>
  <div className="col-span-12 md:col-span-4">{sidebar}</div>
</div>

// GRID - Masonry (CSS columns)
<div className="columns-1 sm:columns-2 lg:columns-3 gap-4 space-y-4">
  {items.map(item => (
    <div key={item.id} className="break-inside-avoid">
      <Card>{item.content}</Card>
    </div>
  ))}
</div>
```

---

## 1.5 STACK LAYOUT

### Descrizione:
Elementi impilati verticalmente o orizzontalmente con spacing consistente.

### Struttura JSX:

```jsx
// VERTICAL STACK
<div className="flex flex-col gap-4">
  {children}
</div>

// HORIZONTAL STACK
<div className="flex flex-row items-center gap-3">
  {children}
</div>

// STACK con dividers
<div className="flex flex-col divide-y divide-border-default">
  {children}
</div>
```

---

## 1.6 PAGE CONTAINER

### Descrizione:
Contenitore per il contenuto principale della pagina con max-width e padding.

### Varianti:
| Variante | Max Width | Uso |
|----------|-----------|-----|
| `sm` | 640px | Form singolo |
| `md` | 768px | Articolo |
| `lg` | 1024px | Dashboard |
| `xl` | 1280px | Wide content |
| `full` | 100% | No max-width |

### Struttura JSX:

```jsx
// PAGE CONTAINER
<div className={cn(
  "mx-auto px-4 sm:px-6 lg:px-8",
  {
    "max-w-screen-sm": size === "sm",
    "max-w-screen-md": size === "md",
    "max-w-screen-lg": size === "lg",
    "max-w-screen-xl": size === "xl",
    "max-w-full": size === "full"
  }
)}>
  {children}
</div>
```


# ============================================================================
# SEZIONE 2: NAVIGATION PATTERNS
# ============================================================================

## 2.1 TOP NAVIGATION BAR (NAVBAR)

### Descrizione:
Barra di navigazione orizzontale in cima alla pagina.

### Varianti:
| Variante | Descrizione |
|----------|-------------|
| `simple` | Logo + Nav links + Actions |
| `with-search` | Include search bar |
| `mega-menu` | Dropdown con mega menu |
| `transparent` | Background trasparente (hero) |
| `sticky` | Resta fisso durante scroll |

### Struttura JSX:

```jsx
// TOP NAVBAR - Standard
<nav className="h-16 bg-surface-default border-b border-border-default 
                sticky top-0 z-sticky">
  <div className="max-w-7xl mx-auto px-4 h-full flex items-center justify-between">
    {/* Logo */}
    <a href="/" className="flex items-center gap-2">
      <Logo className="h-8 w-8" />
      <span className="font-semibold text-lg text-text-default">{appName}</span>
    </a>
    
    {/* Nav Links - Desktop */}
    <div className="hidden md:flex items-center gap-1">
      {navItems.map(item => (
        <a
          key={item.id}
          href={item.href}
          className={cn(
            "px-3 py-2 rounded-md text-sm font-medium",
            "text-text-muted hover:text-text-default",
            "hover:bg-interactive-hover transition-colors",
            item.isActive && "text-text-brand bg-interactive-selected"
          )}
        >
          {item.label}
        </a>
      ))}
    </div>
    
    {/* Actions */}
    <div className="flex items-center gap-2">
      <Button variant="ghost" size="sm">Sign In</Button>
      <Button variant="primary" size="sm">Get Started</Button>
      
      {/* Mobile Menu Button */}
      <button 
        className="md:hidden p-2 rounded-md hover:bg-interactive-hover"
        onClick={toggleMobileMenu}
        aria-label="Toggle menu"
      >
        <MenuIcon className="w-5 h-5" />
      </button>
    </div>
  </div>
</nav>
```

---

## 2.2 BOTTOM NAVIGATION BAR

### Descrizione:
Navigazione mobile in basso allo schermo (3-5 items).

### Struttura JSX:

```jsx
// BOTTOM NAV - Mobile
<nav className="fixed bottom-0 left-0 right-0 h-16 bg-surface-default 
                border-t border-border-default z-sticky
                flex items-center justify-around px-4
                md:hidden">
  {navItems.slice(0, 5).map(item => (
    <a
      key={item.id}
      href={item.href}
      className={cn(
        "flex flex-col items-center gap-1 py-2 px-3 rounded-lg",
        "transition-colors min-w-[64px]",
        item.isActive 
          ? "text-text-brand" 
          : "text-text-muted hover:text-text-default"
      )}
    >
      <item.icon className="w-6 h-6" />
      <span className="text-xs font-medium">{item.label}</span>
    </a>
  ))}
</nav>
```

---

## 2.3 TABS

### Descrizione:
Navigazione tra sezioni di contenuto correlato.

### Varianti:
| Variante | Descrizione |
|----------|-------------|
| `line` | Underline indicator (default) |
| `pills` | Pill/button style |
| `enclosed` | Box/enclosed style |
| `vertical` | Tabs verticali |
| `scrollable` | Scroll orizzontale mobile |

### Struttura JSX:

```jsx
// TABS - Line variant
<div className="border-b border-border-default">
  <nav className="flex gap-0 -mb-px" role="tablist">
    {tabs.map(tab => (
      <button
        key={tab.id}
        role="tab"
        aria-selected={activeTab === tab.id}
        aria-controls={`panel-${tab.id}`}
        onClick={() => setActiveTab(tab.id)}
        className={cn(
          "px-4 py-3 text-sm font-medium border-b-2 transition-colors",
          activeTab === tab.id
            ? "border-border-brand text-text-brand"
            : "border-transparent text-text-muted hover:text-text-default hover:border-border-emphasis"
        )}
      >
        {tab.icon && <tab.icon className="w-4 h-4 mr-2 inline" />}
        {tab.label}
        {tab.count && (
          <Badge variant="secondary" size="sm" className="ml-2">
            {tab.count}
          </Badge>
        )}
      </button>
    ))}
  </nav>
</div>

{/* Tab Panels */}
{tabs.map(tab => (
  <div
    key={tab.id}
    id={`panel-${tab.id}`}
    role="tabpanel"
    aria-labelledby={tab.id}
    hidden={activeTab !== tab.id}
    className="py-4"
  >
    {tab.content}
  </div>
))}
```

```jsx
// TABS - Pills variant
<div className="flex gap-1 p-1 bg-surface-muted rounded-lg">
  {tabs.map(tab => (
    <button
      key={tab.id}
      role="tab"
      className={cn(
        "px-4 py-2 text-sm font-medium rounded-md transition-all",
        activeTab === tab.id
          ? "bg-surface-default text-text-default shadow-sm"
          : "text-text-muted hover:text-text-default"
      )}
    >
      {tab.label}
    </button>
  ))}
</div>
```

---

## 2.4 BREADCRUMBS

### Descrizione:
Mostra il percorso di navigazione corrente.

### Struttura JSX:

```jsx
// BREADCRUMBS
<nav aria-label="Breadcrumb">
  <ol className="flex items-center gap-2 text-sm">
    {items.map((item, index) => (
      <li key={item.id} className="flex items-center gap-2">
        {index > 0 && (
          <ChevronRightIcon className="w-4 h-4 text-text-subtle" />
        )}
        {index === items.length - 1 ? (
          <span className="text-text-default font-medium" aria-current="page">
            {item.label}
          </span>
        ) : (
          <a 
            href={item.href}
            className="text-text-muted hover:text-text-default transition-colors"
          >
            {item.label}
          </a>
        )}
      </li>
    ))}
  </ol>
</nav>
```

---

## 2.5 PAGINATION

### Descrizione:
Navigazione tra pagine di contenuti.

### Varianti:
| Variante | Descrizione |
|----------|-------------|
| `numbers` | Numeri di pagina cliccabili |
| `simple` | Solo Prev/Next |
| `load-more` | Bottone "Load more" |
| `infinite` | Scroll infinito |

### Struttura JSX:

```jsx
// PAGINATION - Numbers
<nav aria-label="Pagination" className="flex items-center justify-center gap-1">
  {/* Previous */}
  <button
    onClick={() => setPage(page - 1)}
    disabled={page === 1}
    className={cn(
      "p-2 rounded-md transition-colors",
      page === 1
        ? "text-text-subtle cursor-not-allowed"
        : "text-text-muted hover:bg-interactive-hover hover:text-text-default"
    )}
    aria-label="Previous page"
  >
    <ChevronLeftIcon className="w-5 h-5" />
  </button>
  
  {/* Page Numbers */}
  {pageNumbers.map(pageNum => (
    <button
      key={pageNum}
      onClick={() => setPage(pageNum)}
      className={cn(
        "w-10 h-10 rounded-md text-sm font-medium transition-colors",
        page === pageNum
          ? "bg-surface-brand text-text-inverse"
          : "text-text-muted hover:bg-interactive-hover hover:text-text-default"
      )}
      aria-current={page === pageNum ? "page" : undefined}
    >
      {pageNum}
    </button>
  ))}
  
  {/* Next */}
  <button
    onClick={() => setPage(page + 1)}
    disabled={page === totalPages}
    className={cn(
      "p-2 rounded-md transition-colors",
      page === totalPages
        ? "text-text-subtle cursor-not-allowed"
        : "text-text-muted hover:bg-interactive-hover hover:text-text-default"
    )}
    aria-label="Next page"
  >
    <ChevronRightIcon className="w-5 h-5" />
  </button>
</nav>
```

---

## 2.6 STEPPER / WIZARD

### Descrizione:
Navigazione multi-step per processi sequenziali.

### Varianti:
| Variante | Descrizione |
|----------|-------------|
| `horizontal` | Steps orizzontali |
| `vertical` | Steps verticali |
| `dots` | Solo indicatori dot |
| `numbers` | Con numeri |

### Struttura JSX:

```jsx
// STEPPER - Horizontal
<nav aria-label="Progress">
  <ol className="flex items-center">
    {steps.map((step, index) => (
      <li key={step.id} className="flex items-center">
        {/* Step Indicator */}
        <div className="flex flex-col items-center">
          <div className={cn(
            "w-10 h-10 rounded-full flex items-center justify-center",
            "text-sm font-medium transition-colors",
            index < currentStep
              ? "bg-feedback-success text-text-inverse"
              : index === currentStep
                ? "bg-surface-brand text-text-inverse"
                : "bg-surface-muted text-text-muted border-2 border-border-default"
          )}>
            {index < currentStep ? (
              <CheckIcon className="w-5 h-5" />
            ) : (
              index + 1
            )}
          </div>
          <span className={cn(
            "mt-2 text-xs font-medium",
            index <= currentStep ? "text-text-default" : "text-text-muted"
          )}>
            {step.label}
          </span>
        </div>
        
        {/* Connector Line */}
        {index < steps.length - 1 && (
          <div className={cn(
            "w-24 h-0.5 mx-2",
            index < currentStep ? "bg-feedback-success" : "bg-border-default"
          )} />
        )}
      </li>
    ))}
  </ol>
</nav>
```

---

## 2.7 SIDEBAR NAVIGATION

### Descrizione:
Menu di navigazione verticale per sidebar.

### Struttura JSX:

```jsx
// SIDEBAR NAV
<nav className="space-y-1">
  {navGroups.map(group => (
    <div key={group.id} className="py-2">
      {/* Group Header */}
      {group.label && (
        <h3 className="px-3 mb-2 text-xs font-semibold text-text-subtle uppercase tracking-wider">
          {group.label}
        </h3>
      )}
      
      {/* Nav Items */}
      <ul className="space-y-1">
        {group.items.map(item => (
          <li key={item.id}>
            {item.children ? (
              // Collapsible Group
              <Disclosure>
                {({ open }) => (
                  <>
                    <Disclosure.Button className={cn(
                      "w-full flex items-center justify-between px-3 py-2 rounded-md",
                      "text-text-muted hover:text-text-default hover:bg-interactive-hover",
                      "transition-colors"
                    )}>
                      <span className="flex items-center gap-3">
                        <item.icon className="w-5 h-5" />
                        {item.label}
                      </span>
                      <ChevronDownIcon className={cn(
                        "w-4 h-4 transition-transform",
                        open && "rotate-180"
                      )} />
                    </Disclosure.Button>
                    <Disclosure.Panel className="ml-8 mt-1 space-y-1">
                      {item.children.map(child => (
                        <a
                          key={child.id}
                          href={child.href}
                          className={cn(
                            "block px-3 py-2 rounded-md text-sm",
                            "text-text-muted hover:text-text-default hover:bg-interactive-hover",
                            child.isActive && "text-text-brand bg-interactive-selected"
                          )}
                        >
                          {child.label}
                        </a>
                      ))}
                    </Disclosure.Panel>
                  </>
                )}
              </Disclosure>
            ) : (
              // Single Item
              <a
                href={item.href}
                className={cn(
                  "flex items-center gap-3 px-3 py-2 rounded-md",
                  "text-text-muted hover:text-text-default hover:bg-interactive-hover",
                  "transition-colors",
                  item.isActive && "text-text-brand bg-interactive-selected"
                )}
              >
                <item.icon className="w-5 h-5" />
                {item.label}
                {item.badge && (
                  <Badge variant="primary" size="sm" className="ml-auto">
                    {item.badge}
                  </Badge>
                )}
              </a>
            )}
          </li>
        ))}
      </ul>
    </div>
  ))}
</nav>
```


# ============================================================================
# SEZIONE 3: CONTENT PATTERNS
# ============================================================================

## 3.1 CARD

### Descrizione:
Contenitore per raggruppare informazioni correlate con bordo, sfondo e padding.

### Varianti:
| Variante | Descrizione |
|----------|-------------|
| `default` | Card standard con bordo |
| `elevated` | Card con shadow |
| `outlined` | Solo bordo, no background |
| `interactive` | Hover/click states |
| `image-top` | Immagine in cima |
| `horizontal` | Layout orizzontale |

### Struttura JSX:

```jsx
// CARD - Default
<article className="bg-surface-default border border-border-default rounded-lg 
                    overflow-hidden">
  {/* Card Image (opzionale) */}
  {image && (
    <div className="aspect-video bg-surface-muted">
      <img 
        src={image} 
        alt={imageAlt}
        className="w-full h-full object-cover"
      />
    </div>
  )}
  
  {/* Card Content */}
  <div className="p-4 space-y-3">
    {/* Header */}
    <div className="space-y-1">
      {eyebrow && (
        <span className="text-xs font-medium text-text-muted uppercase tracking-wide">
          {eyebrow}
        </span>
      )}
      <h3 className="text-lg font-semibold text-text-default line-clamp-2">
        {title}
      </h3>
      {subtitle && (
        <p className="text-sm text-text-muted">{subtitle}</p>
      )}
    </div>
    
    {/* Body */}
    {description && (
      <p className="text-sm text-text-muted line-clamp-3">
        {description}
      </p>
    )}
    
    {/* Footer */}
    {footer && (
      <div className="pt-3 border-t border-border-muted flex items-center justify-between">
        {footer}
      </div>
    )}
  </div>
</article>
```

```jsx
// CARD - Interactive (Clickable)
<article className={cn(
  "bg-surface-default border border-border-default rounded-lg",
  "overflow-hidden cursor-pointer",
  "transition-all duration-fast",
  "hover:border-border-emphasis hover:shadow-md",
  "focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring-focus"
)}>
  {/* ...contenuto card... */}
</article>
```

---

## 3.2 LIST ITEM

### Descrizione:
Elemento singolo di una lista con layout flessibile.

### Varianti:
| Variante | Descrizione |
|----------|-------------|
| `simple` | Solo testo |
| `with-icon` | Icona a sinistra |
| `with-avatar` | Avatar a sinistra |
| `with-description` | Titolo + descrizione |
| `with-action` | Azione a destra |
| `selectable` | Checkbox/radio |

### Struttura JSX:

```jsx
// LIST ITEM - With description and action
<li className="flex items-start gap-3 p-3 hover:bg-surface-muted 
               rounded-md transition-colors">
  {/* Leading Element */}
  {avatar && (
    <Avatar src={avatar} alt={name} size="md" />
  )}
  {icon && !avatar && (
    <div className="p-2 rounded-md bg-surface-muted">
      <Icon className="w-5 h-5 text-text-muted" />
    </div>
  )}
  
  {/* Content */}
  <div className="flex-1 min-w-0">
    <div className="flex items-center gap-2">
      <span className="font-medium text-text-default truncate">
        {title}
      </span>
      {badge && <Badge variant="secondary" size="sm">{badge}</Badge>}
    </div>
    {description && (
      <p className="text-sm text-text-muted truncate">
        {description}
      </p>
    )}
    {meta && (
      <span className="text-xs text-text-subtle">{meta}</span>
    )}
  </div>
  
  {/* Trailing Element */}
  {action && (
    <div className="shrink-0">
      {action}
    </div>
  )}
</li>
```

---

## 3.3 EMPTY STATE

### Descrizione:
Placeholder quando non ci sono dati da mostrare.

### Struttura JSX:

```jsx
// EMPTY STATE
<div className="flex flex-col items-center justify-center py-12 px-4 text-center">
  {/* Illustration/Icon */}
  <div className="w-16 h-16 mb-4 rounded-full bg-surface-muted 
                  flex items-center justify-center">
    <EmptyIcon className="w-8 h-8 text-text-subtle" />
  </div>
  
  {/* Text */}
  <h3 className="text-lg font-semibold text-text-default mb-1">
    {title}
  </h3>
  <p className="text-sm text-text-muted max-w-sm mb-6">
    {description}
  </p>
  
  {/* Action */}
  {action && (
    <Button variant="primary" onClick={action.onClick}>
      {action.icon && <action.icon className="w-4 h-4 mr-2" />}
      {action.label}
    </Button>
  )}
</div>
```

---

## 3.4 HERO SECTION

### Descrizione:
Sezione prominente in cima alla pagina per landing page e marketing.

### Varianti:
| Variante | Descrizione |
|----------|-------------|
| `centered` | Testo centrato |
| `split` | Testo a sinistra, immagine a destra |
| `with-image-bg` | Immagine di sfondo |
| `with-video` | Video di sfondo |

### Struttura JSX:

```jsx
// HERO - Centered
<section className="relative py-20 lg:py-32 bg-surface-default">
  <div className="max-w-4xl mx-auto px-4 text-center">
    {/* Badge (opzionale) */}
    {badge && (
      <Badge variant="secondary" size="lg" className="mb-4">
        {badge}
      </Badge>
    )}
    
    {/* Headline */}
    <h1 className="text-4xl lg:text-6xl font-bold text-text-default 
                   tracking-tight mb-6">
      {headline}
    </h1>
    
    {/* Subheadline */}
    <p className="text-lg lg:text-xl text-text-muted max-w-2xl mx-auto mb-8">
      {subheadline}
    </p>
    
    {/* CTA Buttons */}
    <div className="flex flex-col sm:flex-row items-center justify-center gap-4">
      <Button variant="primary" size="lg">
        {primaryCTA}
      </Button>
      {secondaryCTA && (
        <Button variant="outline" size="lg">
          {secondaryCTA}
        </Button>
      )}
    </div>
    
    {/* Social Proof (opzionale) */}
    {socialProof && (
      <div className="mt-12 flex items-center justify-center gap-6 
                      text-sm text-text-muted">
        {socialProof}
      </div>
    )}
  </div>
</section>
```

---

## 3.5 SECTION HEADER

### Descrizione:
Header per introdurre una sezione di contenuto.

### Struttura JSX:

```jsx
// SECTION HEADER
<div className="flex items-center justify-between mb-6">
  <div>
    <h2 className="text-xl font-semibold text-text-default">
      {title}
    </h2>
    {description && (
      <p className="text-sm text-text-muted mt-1">
        {description}
      </p>
    )}
  </div>
  
  {action && (
    <Button variant="ghost" size="sm">
      {action.label}
      <ChevronRightIcon className="w-4 h-4 ml-1" />
    </Button>
  )}
</div>
```

---

## 3.6 MEDIA OBJECT

### Descrizione:
Pattern classico con media a sinistra e contenuto a destra.

### Struttura JSX:

```jsx
// MEDIA OBJECT
<div className="flex gap-4">
  {/* Media */}
  <div className="shrink-0">
    <img 
      src={mediaSrc} 
      alt={mediaAlt}
      className="w-16 h-16 rounded-lg object-cover"
    />
  </div>
  
  {/* Content */}
  <div className="flex-1 min-w-0">
    <h4 className="font-medium text-text-default">{title}</h4>
    <p className="text-sm text-text-muted mt-1">{content}</p>
  </div>
</div>
```

# ============================================================================
# SEZIONE 4: INPUT PATTERNS
# ============================================================================

## 4.1 TEXT INPUT

### Descrizione:
Campo di input testuale con label, placeholder, helper text e stati.

### Varianti:
| Variante | Descrizione |
|----------|-------------|
| `default` | Input standard |
| `with-icon` | Con icona leading/trailing |
| `with-addon` | Con addon (es. @, $, .com) |
| `textarea` | Multi-line |
| `password` | Con toggle visibility |

### Stati:
| Stato | Descrizione |
|-------|-------------|
| `default` | Stato normale |
| `focus` | In focus |
| `disabled` | Disabilitato |
| `error` | Con errore |
| `success` | Validato con successo |

### Struttura JSX:

```jsx
// TEXT INPUT - Complete
<div className="space-y-1.5">
  {/* Label */}
  <label 
    htmlFor={id}
    className="block text-sm font-medium text-text-default"
  >
    {label}
    {required && <span className="text-feedback-error ml-0.5">*</span>}
  </label>
  
  {/* Input Container */}
  <div className="relative">
    {/* Leading Icon */}
    {leadingIcon && (
      <div className="absolute left-3 top-1/2 -translate-y-1/2 pointer-events-none">
        <LeadingIcon className="w-5 h-5 text-text-muted" />
      </div>
    )}
    
    {/* Input */}
    <input
      id={id}
      type={type}
      value={value}
      onChange={onChange}
      placeholder={placeholder}
      disabled={disabled}
      aria-invalid={!!error}
      aria-describedby={error ? `${id}-error` : helperText ? `${id}-helper` : undefined}
      className={cn(
        "w-full h-10 px-3 rounded-md border bg-surface-default",
        "text-sm text-text-default placeholder:text-text-subtle",
        "transition-colors duration-fast",
        "focus:outline-none focus:ring-2 focus:ring-ring-focus focus:border-border-brand",
        leadingIcon && "pl-10",
        trailingIcon && "pr-10",
        error 
          ? "border-feedback-error focus:ring-feedback-error/20"
          : "border-border-default hover:border-border-emphasis",
        disabled && "bg-surface-muted cursor-not-allowed opacity-60"
      )}
    />
    
    {/* Trailing Icon */}
    {trailingIcon && (
      <div className="absolute right-3 top-1/2 -translate-y-1/2">
        <TrailingIcon className="w-5 h-5 text-text-muted" />
      </div>
    )}
  </div>
  
  {/* Helper Text / Error */}
  {(helperText || error) && (
    <p 
      id={error ? `${id}-error` : `${id}-helper`}
      className={cn(
        "text-xs",
        error ? "text-feedback-error" : "text-text-muted"
      )}
    >
      {error || helperText}
    </p>
  )}
</div>
```

---

## 4.2 SELECT / DROPDOWN

### Descrizione:
Menu a tendina per selezione singola o multipla.

### Varianti:
| Variante | Descrizione |
|----------|-------------|
| `native` | Select nativo HTML |
| `custom` | Custom dropdown (Headless UI) |
| `searchable` | Con ricerca (Combobox) |
| `multi` | Selezione multipla |

### Struttura JSX:

```jsx
// SELECT - Custom (Headless UI style)
<Listbox value={selected} onChange={setSelected}>
  <div className="relative">
    <Listbox.Label className="block text-sm font-medium text-text-default mb-1.5">
      {label}
    </Listbox.Label>
    
    <Listbox.Button className={cn(
      "relative w-full h-10 pl-3 pr-10 text-left",
      "bg-surface-default border border-border-default rounded-md",
      "text-sm text-text-default",
      "focus:outline-none focus:ring-2 focus:ring-ring-focus focus:border-border-brand"
    )}>
      <span className="block truncate">{selected?.label || placeholder}</span>
      <span className="absolute right-3 top-1/2 -translate-y-1/2">
        <ChevronDownIcon className="w-5 h-5 text-text-muted" />
      </span>
    </Listbox.Button>
    
    <Transition
      enter="transition duration-fast ease-out"
      enterFrom="opacity-0 -translate-y-1"
      enterTo="opacity-100 translate-y-0"
      leave="transition duration-fast ease-in"
      leaveFrom="opacity-100 translate-y-0"
      leaveTo="opacity-0 -translate-y-1"
    >
      <Listbox.Options className={cn(
        "absolute z-dropdown w-full mt-1 py-1",
        "bg-surface-default border border-border-default rounded-md shadow-lg",
        "max-h-60 overflow-auto focus:outline-none"
      )}>
        {options.map(option => (
          <Listbox.Option
            key={option.value}
            value={option}
            className={({ active, selected }) => cn(
              "relative px-3 py-2 cursor-pointer select-none",
              "text-sm",
              active && "bg-interactive-hover",
              selected && "text-text-brand font-medium"
            )}
          >
            {({ selected }) => (
              <>
                <span className="block truncate">{option.label}</span>
                {selected && (
                  <span className="absolute right-3 top-1/2 -translate-y-1/2">
                    <CheckIcon className="w-4 h-4 text-text-brand" />
                  </span>
                )}
              </>
            )}
          </Listbox.Option>
        ))}
      </Listbox.Options>
    </Transition>
  </div>
</Listbox>
```

---

## 4.3 CHECKBOX

### Struttura JSX:

```jsx
// CHECKBOX
<label className="flex items-start gap-3 cursor-pointer group">
  <div className="relative flex items-center justify-center mt-0.5">
    <input
      type="checkbox"
      checked={checked}
      onChange={onChange}
      disabled={disabled}
      className={cn(
        "peer w-5 h-5 rounded border-2 appearance-none cursor-pointer",
        "border-border-default bg-surface-default",
        "checked:bg-surface-brand checked:border-surface-brand",
        "focus:ring-2 focus:ring-ring-focus focus:ring-offset-2",
        "disabled:opacity-50 disabled:cursor-not-allowed",
        "transition-colors"
      )}
    />
    <CheckIcon className={cn(
      "absolute w-3 h-3 text-text-inverse pointer-events-none",
      "opacity-0 peer-checked:opacity-100 transition-opacity"
    )} />
  </div>
  
  <div className="flex-1">
    <span className="text-sm font-medium text-text-default">
      {label}
    </span>
    {description && (
      <p className="text-xs text-text-muted mt-0.5">{description}</p>
    )}
  </div>
</label>
```

---

## 4.4 RADIO GROUP

### Struttura JSX:

```jsx
// RADIO GROUP
<fieldset>
  <legend className="text-sm font-medium text-text-default mb-3">
    {legend}
  </legend>
  
  <div className="space-y-2">
    {options.map(option => (
      <label 
        key={option.value}
        className="flex items-start gap-3 cursor-pointer"
      >
        <div className="relative flex items-center justify-center mt-0.5">
          <input
            type="radio"
            name={name}
            value={option.value}
            checked={selected === option.value}
            onChange={() => setSelected(option.value)}
            className={cn(
              "peer w-5 h-5 rounded-full border-2 appearance-none cursor-pointer",
              "border-border-default bg-surface-default",
              "checked:border-surface-brand",
              "focus:ring-2 focus:ring-ring-focus focus:ring-offset-2",
              "transition-colors"
            )}
          />
          <div className={cn(
            "absolute w-2.5 h-2.5 rounded-full bg-surface-brand",
            "scale-0 peer-checked:scale-100 transition-transform"
          )} />
        </div>
        
        <div>
          <span className="text-sm font-medium text-text-default">
            {option.label}
          </span>
          {option.description && (
            <p className="text-xs text-text-muted mt-0.5">
              {option.description}
            </p>
          )}
        </div>
      </label>
    ))}
  </div>
</fieldset>
```

---

## 4.5 TOGGLE / SWITCH

### Struttura JSX:

```jsx
// TOGGLE SWITCH
<Switch.Group>
  <div className="flex items-center justify-between">
    <Switch.Label className="text-sm font-medium text-text-default">
      {label}
    </Switch.Label>
    
    <Switch
      checked={enabled}
      onChange={setEnabled}
      className={cn(
        "relative inline-flex h-6 w-11 items-center rounded-full",
        "transition-colors duration-fast",
        "focus:outline-none focus:ring-2 focus:ring-ring-focus focus:ring-offset-2",
        enabled ? "bg-surface-brand" : "bg-surface-muted"
      )}
    >
      <span className={cn(
        "inline-block h-4 w-4 rounded-full bg-white shadow-sm",
        "transform transition-transform duration-fast",
        enabled ? "translate-x-6" : "translate-x-1"
      )} />
    </Switch>
  </div>
</Switch.Group>
```

---

## 4.6 SEARCH INPUT

### Struttura JSX:

```jsx
// SEARCH INPUT
<div className="relative">
  <SearchIcon className="absolute left-3 top-1/2 -translate-y-1/2 
                          w-5 h-5 text-text-muted pointer-events-none" />
  <input
    type="search"
    value={query}
    onChange={(e) => setQuery(e.target.value)}
    placeholder={placeholder || "Search..."}
    className={cn(
      "w-full h-10 pl-10 pr-4 rounded-md",
      "bg-surface-muted border-0",
      "text-sm text-text-default placeholder:text-text-subtle",
      "focus:outline-none focus:ring-2 focus:ring-ring-focus focus:bg-surface-default"
    )}
  />
  {query && (
    <button
      onClick={() => setQuery("")}
      className="absolute right-3 top-1/2 -translate-y-1/2 
                 p-1 rounded hover:bg-interactive-hover"
    >
      <XIcon className="w-4 h-4 text-text-muted" />
    </button>
  )}
</div>
```

---

## 4.7 DATE PICKER

### Descrizione:
Selezione data con calendario popup.

### Struttura JSX:

```jsx
// DATE PICKER - Trigger
<Popover>
  <Popover.Button className={cn(
    "w-full h-10 px-3 flex items-center justify-between",
    "bg-surface-default border border-border-default rounded-md",
    "text-sm text-left",
    "focus:outline-none focus:ring-2 focus:ring-ring-focus"
  )}>
    <span className={date ? "text-text-default" : "text-text-subtle"}>
      {date ? format(date, "PPP") : "Pick a date"}
    </span>
    <CalendarIcon className="w-5 h-5 text-text-muted" />
  </Popover.Button>
  
  <Popover.Panel className={cn(
    "absolute z-dropdown mt-1 p-3",
    "bg-surface-default border border-border-default rounded-lg shadow-lg"
  )}>
    <Calendar
      selected={date}
      onSelect={setDate}
      className="rounded-md"
    />
  </Popover.Panel>
</Popover>
```

---

## 4.8 FILE UPLOAD

### Struttura JSX:

```jsx
// FILE UPLOAD - Dropzone
<div
  onDragOver={handleDragOver}
  onDragLeave={handleDragLeave}
  onDrop={handleDrop}
  className={cn(
    "border-2 border-dashed rounded-lg p-8",
    "flex flex-col items-center justify-center",
    "transition-colors cursor-pointer",
    isDragging
      ? "border-border-brand bg-surface-brand/5"
      : "border-border-default hover:border-border-emphasis"
  )}
>
  <input
    type="file"
    accept={accept}
    multiple={multiple}
    onChange={handleFileChange}
    className="sr-only"
    id={id}
  />
  
  <UploadCloudIcon className="w-10 h-10 text-text-muted mb-3" />
  
  <label htmlFor={id} className="cursor-pointer">
    <span className="text-sm font-medium text-text-brand hover:underline">
      Click to upload
    </span>
    <span className="text-sm text-text-muted"> or drag and drop</span>
  </label>
  
  <p className="text-xs text-text-subtle mt-1">
    {hint || "PNG, JPG up to 10MB"}
  </p>
</div>
```


# ============================================================================
# SEZIONE 5: FEEDBACK PATTERNS
# ============================================================================

## 5.1 ALERT / BANNER

### Descrizione:
Messaggio inline per comunicare informazioni importanti all'utente.

### Varianti:
| Variante | Colore | Uso |
|----------|--------|-----|
| `info` | Blue | Informazioni generali |
| `success` | Green | Operazione completata |
| `warning` | Amber | Attenzione richiesta |
| `error` | Red | Errore o problema |

### Struttura JSX:

```jsx
// ALERT
<div 
  role="alert"
  className={cn(
    "flex gap-3 p-4 rounded-lg border",
    {
      "bg-feedback-info/10 border-feedback-info/20 text-feedback-info": variant === "info",
      "bg-feedback-success/10 border-feedback-success/20 text-feedback-success": variant === "success",
      "bg-feedback-warning/10 border-feedback-warning/20 text-feedback-warning": variant === "warning",
      "bg-feedback-error/10 border-feedback-error/20 text-feedback-error": variant === "error"
    }
  )}
>
  {/* Icon */}
  <div className="shrink-0">
    <AlertIcon className="w-5 h-5" />
  </div>
  
  {/* Content */}
  <div className="flex-1">
    {title && (
      <h4 className="font-medium mb-1">{title}</h4>
    )}
    <p className="text-sm opacity-90">{message}</p>
    
    {/* Actions (opzionale) */}
    {actions && (
      <div className="flex gap-2 mt-3">
        {actions}
      </div>
    )}
  </div>
  
  {/* Dismiss */}
  {dismissible && (
    <button 
      onClick={onDismiss}
      className="shrink-0 p-1 rounded hover:bg-black/5"
      aria-label="Dismiss"
    >
      <XIcon className="w-4 h-4" />
    </button>
  )}
</div>
```

---

## 5.2 TOAST / NOTIFICATION

### Descrizione:
Notifica temporanea che appare e scompare automaticamente.

### Struttura JSX:

```jsx
// TOAST
<div className={cn(
  "fixed bottom-4 right-4 z-toast",
  "w-full max-w-sm",
  "animate-slide-in-right"
)}>
  <div className={cn(
    "flex items-start gap-3 p-4",
    "bg-surface-default border border-border-default rounded-lg shadow-lg"
  )}>
    {/* Icon */}
    <div className={cn(
      "shrink-0 p-1 rounded-full",
      {
        "bg-feedback-success/10 text-feedback-success": type === "success",
        "bg-feedback-error/10 text-feedback-error": type === "error",
        "bg-feedback-info/10 text-feedback-info": type === "info"
      }
    )}>
      <ToastIcon className="w-4 h-4" />
    </div>
    
    {/* Content */}
    <div className="flex-1 pt-0.5">
      <p className="text-sm font-medium text-text-default">{title}</p>
      {description && (
        <p className="text-sm text-text-muted mt-1">{description}</p>
      )}
    </div>
    
    {/* Close */}
    <button 
      onClick={onClose}
      className="shrink-0 p-1 rounded hover:bg-interactive-hover"
    >
      <XIcon className="w-4 h-4 text-text-muted" />
    </button>
  </div>
</div>
```

---

## 5.3 PROGRESS BAR

### Varianti:
| Variante | Descrizione |
|----------|-------------|
| `determinate` | Percentuale nota |
| `indeterminate` | Loading infinito |
| `segmented` | Passaggi discreti |

### Struttura JSX:

```jsx
// PROGRESS BAR - Determinate
<div className="space-y-2">
  {/* Label */}
  <div className="flex justify-between text-sm">
    <span className="text-text-muted">{label}</span>
    <span className="font-medium text-text-default">{value}%</span>
  </div>
  
  {/* Bar */}
  <div className="h-2 bg-surface-muted rounded-full overflow-hidden">
    <div 
      className="h-full bg-surface-brand rounded-full transition-all duration-normal"
      style={{ width: `${value}%` }}
      role="progressbar"
      aria-valuenow={value}
      aria-valuemin={0}
      aria-valuemax={100}
    />
  </div>
</div>

// PROGRESS BAR - Indeterminate
<div className="h-2 bg-surface-muted rounded-full overflow-hidden">
  <div className="h-full w-1/3 bg-surface-brand rounded-full animate-progress-indeterminate" />
</div>
```

---

## 5.4 SPINNER / LOADER

### Struttura JSX:

```jsx
// SPINNER
<svg 
  className={cn(
    "animate-spin",
    {
      "w-4 h-4": size === "sm",
      "w-6 h-6": size === "md",
      "w-8 h-8": size === "lg"
    }
  )}
  viewBox="0 0 24 24"
  fill="none"
>
  <circle
    className="opacity-25"
    cx="12"
    cy="12"
    r="10"
    stroke="currentColor"
    strokeWidth="4"
  />
  <path
    className="opacity-75"
    fill="currentColor"
    d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
  />
</svg>

// SKELETON
<div className="animate-pulse space-y-3">
  <div className="h-4 bg-surface-muted rounded w-3/4" />
  <div className="h-4 bg-surface-muted rounded w-1/2" />
  <div className="h-4 bg-surface-muted rounded w-5/6" />
</div>
```

---

## 5.5 BADGE

### Varianti:
| Variante | Uso |
|----------|-----|
| `primary` | Principale |
| `secondary` | Neutro |
| `success` | Positivo |
| `warning` | Attenzione |
| `error` | Negativo |
| `outline` | Solo bordo |

### Struttura JSX:

```jsx
// BADGE
<span className={cn(
  "inline-flex items-center gap-1 px-2 py-0.5 rounded-full",
  "text-xs font-medium",
  {
    "bg-surface-brand text-text-inverse": variant === "primary",
    "bg-surface-muted text-text-muted": variant === "secondary",
    "bg-feedback-success/10 text-feedback-success": variant === "success",
    "bg-feedback-warning/10 text-feedback-warning": variant === "warning",
    "bg-feedback-error/10 text-feedback-error": variant === "error",
    "bg-transparent border border-border-default text-text-muted": variant === "outline"
  }
)}>
  {dot && (
    <span className={cn(
      "w-1.5 h-1.5 rounded-full",
      {
        "bg-current": true
      }
    )} />
  )}
  {children}
</span>
```

---

## 5.6 TOOLTIP

### Struttura JSX:

```jsx
// TOOLTIP
<Tooltip.Provider>
  <Tooltip.Root>
    <Tooltip.Trigger asChild>
      {trigger}
    </Tooltip.Trigger>
    
    <Tooltip.Portal>
      <Tooltip.Content
        side={side}
        sideOffset={5}
        className={cn(
          "z-tooltip px-3 py-1.5 rounded-md",
          "bg-surface-inverse text-text-inverse",
          "text-xs font-medium",
          "animate-fade-in"
        )}
      >
        {content}
        <Tooltip.Arrow className="fill-surface-inverse" />
      </Tooltip.Content>
    </Tooltip.Portal>
  </Tooltip.Root>
</Tooltip.Provider>
```

# ============================================================================
# SEZIONE 6: OVERLAY PATTERNS
# ============================================================================

## 6.1 MODAL / DIALOG

### Descrizione:
Finestra modale per interazioni importanti che richiedono focus.

### Varianti:
| Variante | Larghezza | Uso |
|----------|-----------|-----|
| `sm` | 400px | Conferme semplici |
| `md` | 500px | Form brevi |
| `lg` | 600px | Form complessi |
| `xl` | 800px | Contenuti ricchi |
| `full` | Full screen | Mobile o editor |

### Struttura JSX:

```jsx
// MODAL
<Dialog open={isOpen} onClose={onClose}>
  {/* Backdrop */}
  <div className="fixed inset-0 z-modal-backdrop bg-black/50 backdrop-blur-sm" 
       aria-hidden="true" />
  
  {/* Modal Container */}
  <div className="fixed inset-0 z-modal flex items-center justify-center p-4">
    <Dialog.Panel className={cn(
      "w-full bg-surface-default rounded-xl shadow-2xl",
      "animate-modal-in",
      {
        "max-w-sm": size === "sm",
        "max-w-md": size === "md",
        "max-w-lg": size === "lg",
        "max-w-3xl": size === "xl"
      }
    )}>
      {/* Header */}
      <div className="flex items-center justify-between p-4 border-b border-border-default">
        <Dialog.Title className="text-lg font-semibold text-text-default">
          {title}
        </Dialog.Title>
        <button 
          onClick={onClose}
          className="p-2 rounded-md hover:bg-interactive-hover transition-colors"
        >
          <XIcon className="w-5 h-5 text-text-muted" />
        </button>
      </div>
      
      {/* Body */}
      <div className="p-4 max-h-[60vh] overflow-y-auto">
        {children}
      </div>
      
      {/* Footer */}
      {footer && (
        <div className="flex items-center justify-end gap-2 p-4 
                        border-t border-border-default bg-surface-muted/50">
          {footer}
        </div>
      )}
    </Dialog.Panel>
  </div>
</Dialog>
```

---

## 6.2 DRAWER / SHEET

### Descrizione:
Pannello che scorre dal bordo dello schermo.

### Varianti:
| Variante | Descrizione |
|----------|-------------|
| `left` | Da sinistra |
| `right` | Da destra |
| `bottom` | Dal basso (mobile sheet) |
| `top` | Dall'alto |

### Struttura JSX:

```jsx
// DRAWER - Right
<Transition show={isOpen}>
  {/* Backdrop */}
  <Transition.Child
    enter="transition-opacity duration-normal"
    enterFrom="opacity-0"
    enterTo="opacity-100"
    leave="transition-opacity duration-fast"
    leaveFrom="opacity-100"
    leaveTo="opacity-0"
  >
    <div 
      className="fixed inset-0 z-drawer-backdrop bg-black/50"
      onClick={onClose}
    />
  </Transition.Child>
  
  {/* Drawer */}
  <Transition.Child
    enter="transition-transform duration-normal ease-out"
    enterFrom="translate-x-full"
    enterTo="translate-x-0"
    leave="transition-transform duration-fast ease-in"
    leaveFrom="translate-x-0"
    leaveTo="translate-x-full"
  >
    <div className={cn(
      "fixed right-0 top-0 h-full z-drawer",
      "w-full max-w-md",
      "bg-surface-default shadow-2xl",
      "flex flex-col"
    )}>
      {/* Header */}
      <div className="flex items-center justify-between p-4 border-b border-border-default">
        <h2 className="text-lg font-semibold text-text-default">{title}</h2>
        <button onClick={onClose} className="p-2 rounded-md hover:bg-interactive-hover">
          <XIcon className="w-5 h-5" />
        </button>
      </div>
      
      {/* Content */}
      <div className="flex-1 overflow-y-auto p-4">
        {children}
      </div>
      
      {/* Footer */}
      {footer && (
        <div className="p-4 border-t border-border-default">
          {footer}
        </div>
      )}
    </div>
  </Transition.Child>
</Transition>
```

---

## 6.3 POPOVER / DROPDOWN MENU

### Struttura JSX:

```jsx
// POPOVER
<Popover className="relative">
  <Popover.Button className={triggerClassName}>
    {trigger}
  </Popover.Button>
  
  <Transition
    enter="transition duration-fast ease-out"
    enterFrom="opacity-0 scale-95"
    enterTo="opacity-100 scale-100"
    leave="transition duration-fast ease-in"
    leaveFrom="opacity-100 scale-100"
    leaveTo="opacity-0 scale-95"
  >
    <Popover.Panel className={cn(
      "absolute z-popover mt-2",
      "w-64 p-4",
      "bg-surface-default border border-border-default rounded-lg shadow-lg",
      {
        "left-0": align === "start",
        "right-0": align === "end",
        "left-1/2 -translate-x-1/2": align === "center"
      }
    )}>
      {content}
    </Popover.Panel>
  </Transition>
</Popover>

// DROPDOWN MENU
<Menu as="div" className="relative">
  <Menu.Button className={triggerClassName}>
    {trigger}
  </Menu.Button>
  
  <Menu.Items className={cn(
    "absolute right-0 z-dropdown mt-2",
    "w-56 py-1",
    "bg-surface-default border border-border-default rounded-lg shadow-lg",
    "focus:outline-none"
  )}>
    {items.map(item => (
      <Menu.Item key={item.id}>
        {({ active }) => (
          <button
            onClick={item.onClick}
            className={cn(
              "w-full flex items-center gap-2 px-3 py-2 text-sm text-left",
              active && "bg-interactive-hover",
              item.destructive && "text-feedback-error"
            )}
          >
            {item.icon && <item.icon className="w-4 h-4" />}
            {item.label}
          </button>
        )}
      </Menu.Item>
    ))}
  </Menu.Items>
</Menu>
```

---

## 6.4 COMMAND PALETTE

### Descrizione:
Ricerca globale con comandi (⌘K / Ctrl+K pattern).

### Struttura JSX:

```jsx
// COMMAND PALETTE
<Dialog open={isOpen} onClose={onClose}>
  <div className="fixed inset-0 z-command bg-black/50 backdrop-blur-sm" />
  
  <div className="fixed inset-0 z-command flex items-start justify-center pt-[20vh] p-4">
    <Dialog.Panel className="w-full max-w-xl bg-surface-default rounded-xl shadow-2xl overflow-hidden">
      {/* Search Input */}
      <div className="flex items-center gap-3 px-4 border-b border-border-default">
        <SearchIcon className="w-5 h-5 text-text-muted" />
        <input
          type="text"
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder="Type a command or search..."
          className="flex-1 h-14 bg-transparent text-text-default 
                     placeholder:text-text-subtle focus:outline-none"
          autoFocus
        />
        <kbd className="px-2 py-1 text-xs bg-surface-muted rounded">ESC</kbd>
      </div>
      
      {/* Results */}
      <div className="max-h-80 overflow-y-auto py-2">
        {results.map((group, groupIndex) => (
          <div key={groupIndex}>
            <div className="px-4 py-2 text-xs font-semibold text-text-subtle uppercase">
              {group.title}
            </div>
            {group.items.map((item, itemIndex) => (
              <button
                key={itemIndex}
                onClick={() => handleSelect(item)}
                className={cn(
                  "w-full flex items-center gap-3 px-4 py-2",
                  "hover:bg-interactive-hover",
                  selectedIndex === itemIndex && "bg-interactive-selected"
                )}
              >
                <item.icon className="w-5 h-5 text-text-muted" />
                <span className="flex-1 text-sm text-text-default text-left">
                  {item.label}
                </span>
                {item.shortcut && (
                  <kbd className="text-xs text-text-subtle">{item.shortcut}</kbd>
                )}
              </button>
            ))}
          </div>
        ))}
      </div>
    </Dialog.Panel>
  </div>
</Dialog>
```


# ============================================================================
# SEZIONE 7: DATA DISPLAY PATTERNS
# ============================================================================

## 7.1 DATA TABLE

### Descrizione:
Tabella per visualizzare dati strutturati con sorting, filtering, pagination.

### Struttura JSX:

```jsx
// DATA TABLE
<div className="border border-border-default rounded-lg overflow-hidden">
  {/* Table Header Actions */}
  <div className="flex items-center justify-between p-4 border-b border-border-default bg-surface-muted/50">
    <div className="flex items-center gap-2">
      <SearchInput 
        value={searchQuery}
        onChange={setSearchQuery}
        placeholder="Search..."
      />
      <FilterDropdown filters={filters} onFilterChange={setFilters} />
    </div>
    <div className="flex items-center gap-2">
      <Button variant="outline" size="sm">Export</Button>
      <Button variant="primary" size="sm">Add New</Button>
    </div>
  </div>
  
  {/* Table */}
  <div className="overflow-x-auto">
    <table className="w-full">
      <thead>
        <tr className="bg-surface-muted/50">
          {/* Select All */}
          <th className="w-12 px-4 py-3">
            <Checkbox 
              checked={allSelected}
              onChange={toggleSelectAll}
            />
          </th>
          
          {/* Column Headers */}
          {columns.map(column => (
            <th 
              key={column.id}
              className={cn(
                "px-4 py-3 text-left text-xs font-semibold text-text-muted uppercase tracking-wider",
                column.sortable && "cursor-pointer hover:text-text-default"
              )}
              onClick={() => column.sortable && handleSort(column.id)}
            >
              <div className="flex items-center gap-1">
                {column.label}
                {column.sortable && sortColumn === column.id && (
                  <SortIcon direction={sortDirection} className="w-4 h-4" />
                )}
              </div>
            </th>
          ))}
          
          {/* Actions Column */}
          <th className="w-12 px-4 py-3" />
        </tr>
      </thead>
      
      <tbody className="divide-y divide-border-default">
        {rows.map(row => (
          <tr 
            key={row.id}
            className="hover:bg-surface-muted/30 transition-colors"
          >
            {/* Row Checkbox */}
            <td className="px-4 py-3">
              <Checkbox 
                checked={selectedRows.includes(row.id)}
                onChange={() => toggleRow(row.id)}
              />
            </td>
            
            {/* Data Cells */}
            {columns.map(column => (
              <td key={column.id} className="px-4 py-3 text-sm text-text-default">
                {column.render ? column.render(row) : row[column.id]}
              </td>
            ))}
            
            {/* Row Actions */}
            <td className="px-4 py-3">
              <DropdownMenu>
                <DropdownMenu.Trigger asChild>
                  <button className="p-1 rounded hover:bg-interactive-hover">
                    <MoreHorizontalIcon className="w-4 h-4 text-text-muted" />
                  </button>
                </DropdownMenu.Trigger>
                <DropdownMenu.Content>
                  <DropdownMenu.Item onClick={() => onEdit(row)}>Edit</DropdownMenu.Item>
                  <DropdownMenu.Item onClick={() => onDelete(row)} destructive>Delete</DropdownMenu.Item>
                </DropdownMenu.Content>
              </DropdownMenu>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  </div>
  
  {/* Table Footer - Pagination */}
  <div className="flex items-center justify-between p-4 border-t border-border-default bg-surface-muted/50">
    <span className="text-sm text-text-muted">
      Showing {startIndex + 1} to {endIndex} of {totalCount} results
    </span>
    <Pagination 
      currentPage={page}
      totalPages={totalPages}
      onPageChange={setPage}
    />
  </div>
</div>
```

---

## 7.2 STATS CARD / METRIC

### Descrizione:
Visualizzazione di una singola metrica con trend.

### Struttura JSX:

```jsx
// STAT CARD
<div className="bg-surface-default border border-border-default rounded-lg p-4">
  <div className="flex items-start justify-between">
    {/* Stat Info */}
    <div>
      <p className="text-sm text-text-muted">{label}</p>
      <p className="text-2xl font-bold text-text-default mt-1">{value}</p>
      
      {/* Trend */}
      {trend && (
        <div className={cn(
          "flex items-center gap-1 mt-2 text-sm",
          trend.direction === "up" ? "text-feedback-success" : "text-feedback-error"
        )}>
          {trend.direction === "up" ? (
            <TrendingUpIcon className="w-4 h-4" />
          ) : (
            <TrendingDownIcon className="w-4 h-4" />
          )}
          <span>{trend.value}%</span>
          <span className="text-text-muted">vs last period</span>
        </div>
      )}
    </div>
    
    {/* Icon */}
    <div className={cn(
      "p-2 rounded-lg",
      iconBg || "bg-surface-brand/10"
    )}>
      <Icon className="w-5 h-5 text-text-brand" />
    </div>
  </div>
  
  {/* Sparkline (opzionale) */}
  {sparklineData && (
    <div className="mt-4 h-12">
      <Sparkline data={sparklineData} />
    </div>
  )}
</div>
```

---

## 7.3 CHART CONTAINER

### Descrizione:
Wrapper per grafici con header, legenda e controlli.

### Struttura JSX:

```jsx
// CHART CONTAINER
<div className="bg-surface-default border border-border-default rounded-lg">
  {/* Chart Header */}
  <div className="flex items-center justify-between p-4 border-b border-border-default">
    <div>
      <h3 className="font-semibold text-text-default">{title}</h3>
      {subtitle && (
        <p className="text-sm text-text-muted mt-0.5">{subtitle}</p>
      )}
    </div>
    
    <div className="flex items-center gap-2">
      {/* Time Range Selector */}
      <SegmentedControl
        options={timeRanges}
        value={selectedRange}
        onChange={setSelectedRange}
      />
      
      {/* More Options */}
      <DropdownMenu>
        <DropdownMenu.Trigger asChild>
          <Button variant="ghost" size="sm">
            <MoreVerticalIcon className="w-4 h-4" />
          </Button>
        </DropdownMenu.Trigger>
        <DropdownMenu.Content>
          <DropdownMenu.Item>Download CSV</DropdownMenu.Item>
          <DropdownMenu.Item>Download PNG</DropdownMenu.Item>
        </DropdownMenu.Content>
      </DropdownMenu>
    </div>
  </div>
  
  {/* Chart Area */}
  <div className="p-4">
    <div className="h-64">
      {/* Recharts, Chart.js, etc. */}
      {children}
    </div>
  </div>
  
  {/* Legend */}
  {legend && (
    <div className="flex items-center justify-center gap-6 p-4 border-t border-border-muted">
      {legend.map(item => (
        <div key={item.key} className="flex items-center gap-2">
          <span 
            className="w-3 h-3 rounded-full" 
            style={{ backgroundColor: item.color }}
          />
          <span className="text-sm text-text-muted">{item.label}</span>
        </div>
      ))}
    </div>
  )}
</div>
```

---

## 7.4 TIMELINE

### Descrizione:
Visualizzazione cronologica di eventi.

### Struttura JSX:

```jsx
// TIMELINE
<div className="space-y-0">
  {events.map((event, index) => (
    <div key={event.id} className="relative flex gap-4">
      {/* Line & Dot */}
      <div className="flex flex-col items-center">
        <div className={cn(
          "w-3 h-3 rounded-full border-2 z-10",
          event.status === "completed" 
            ? "bg-feedback-success border-feedback-success"
            : event.status === "current"
              ? "bg-surface-brand border-surface-brand"
              : "bg-surface-default border-border-default"
        )} />
        {index < events.length - 1 && (
          <div className={cn(
            "w-0.5 flex-1 -my-1",
            event.status === "completed" 
              ? "bg-feedback-success" 
              : "bg-border-default"
          )} />
        )}
      </div>
      
      {/* Content */}
      <div className="flex-1 pb-8">
        <div className="flex items-center gap-2">
          <span className="font-medium text-text-default">{event.title}</span>
          <span className="text-xs text-text-subtle">{event.timestamp}</span>
        </div>
        {event.description && (
          <p className="text-sm text-text-muted mt-1">{event.description}</p>
        )}
        {event.content && (
          <div className="mt-2">{event.content}</div>
        )}
      </div>
    </div>
  ))}
</div>
```

---

## 7.5 AVATAR GROUP

### Struttura JSX:

```jsx
// AVATAR GROUP
<div className="flex -space-x-2">
  {users.slice(0, maxVisible).map((user, index) => (
    <Avatar
      key={user.id}
      src={user.avatar}
      alt={user.name}
      size={size}
      className="ring-2 ring-surface-default"
      style={{ zIndex: maxVisible - index }}
    />
  ))}
  
  {users.length > maxVisible && (
    <div className={cn(
      "flex items-center justify-center rounded-full",
      "bg-surface-muted ring-2 ring-surface-default",
      "text-xs font-medium text-text-muted",
      {
        "w-8 h-8": size === "sm",
        "w-10 h-10": size === "md",
        "w-12 h-12": size === "lg"
      }
    )}>
      +{users.length - maxVisible}
    </div>
  )}
</div>
```

# ============================================================================
# SEZIONE 8: ACTION PATTERNS
# ============================================================================

## 8.1 BUTTON

### Varianti:
| Variante | Uso |
|----------|-----|
| `primary` | Azione principale |
| `secondary` | Azione secondaria |
| `outline` | Azione terziaria |
| `ghost` | Azione minimale |
| `destructive` | Azione pericolosa |
| `link` | Come link |

### Sizes:
| Size | Height | Uso |
|------|--------|-----|
| `sm` | 32px | Compact UI |
| `md` | 40px | Default |
| `lg` | 48px | CTAs prominenti |

### Struttura JSX:

```jsx
// BUTTON
<button
  type={type}
  disabled={disabled || loading}
  className={cn(
    "inline-flex items-center justify-center gap-2",
    "font-medium rounded-md transition-colors",
    "focus:outline-none focus:ring-2 focus:ring-ring-focus focus:ring-offset-2",
    "disabled:opacity-50 disabled:cursor-not-allowed",
    
    // Sizes
    {
      "h-8 px-3 text-sm": size === "sm",
      "h-10 px-4 text-sm": size === "md",
      "h-12 px-6 text-base": size === "lg"
    },
    
    // Variants
    {
      "bg-surface-brand text-text-inverse hover:bg-surface-brand-emphasis": variant === "primary",
      "bg-surface-muted text-text-default hover:bg-surface-muted-emphasis": variant === "secondary",
      "border border-border-default bg-transparent text-text-default hover:bg-interactive-hover": variant === "outline",
      "bg-transparent text-text-default hover:bg-interactive-hover": variant === "ghost",
      "bg-feedback-error text-text-inverse hover:bg-feedback-error/90": variant === "destructive",
      "bg-transparent text-text-brand hover:underline p-0 h-auto": variant === "link"
    },
    
    // Full width
    fullWidth && "w-full"
  )}
>
  {loading ? (
    <Spinner size="sm" />
  ) : (
    <>
      {leftIcon && <LeftIcon className="w-4 h-4" />}
      {children}
      {rightIcon && <RightIcon className="w-4 h-4" />}
    </>
  )}
</button>
```

---

## 8.2 BUTTON GROUP

### Struttura JSX:

```jsx
// BUTTON GROUP
<div className="inline-flex rounded-md shadow-sm" role="group">
  {buttons.map((button, index) => (
    <button
      key={button.id}
      onClick={button.onClick}
      className={cn(
        "px-4 py-2 text-sm font-medium",
        "border border-border-default",
        "focus:z-10 focus:ring-2 focus:ring-ring-focus",
        index === 0 && "rounded-l-md",
        index === buttons.length - 1 && "rounded-r-md",
        index > 0 && "-ml-px",
        button.isActive
          ? "bg-surface-brand text-text-inverse border-surface-brand"
          : "bg-surface-default text-text-default hover:bg-interactive-hover"
      )}
    >
      {button.label}
    </button>
  ))}
</div>
```

---

## 8.3 ICON BUTTON

### Struttura JSX:

```jsx
// ICON BUTTON
<button
  aria-label={ariaLabel}
  disabled={disabled}
  className={cn(
    "inline-flex items-center justify-center rounded-md",
    "transition-colors",
    "focus:outline-none focus:ring-2 focus:ring-ring-focus",
    "disabled:opacity-50 disabled:cursor-not-allowed",
    
    // Sizes
    {
      "w-8 h-8": size === "sm",
      "w-10 h-10": size === "md",
      "w-12 h-12": size === "lg"
    },
    
    // Variants
    {
      "bg-surface-brand text-text-inverse hover:bg-surface-brand-emphasis": variant === "primary",
      "bg-transparent text-text-muted hover:bg-interactive-hover hover:text-text-default": variant === "ghost",
      "border border-border-default text-text-muted hover:bg-interactive-hover": variant === "outline"
    }
  )}
>
  <Icon className={cn(
    size === "sm" && "w-4 h-4",
    size === "md" && "w-5 h-5",
    size === "lg" && "w-6 h-6"
  )} />
</button>
```

---

## 8.4 FAB (Floating Action Button)

### Struttura JSX:

```jsx
// FAB
<button
  className={cn(
    "fixed z-fab shadow-lg",
    "rounded-full bg-surface-brand text-text-inverse",
    "hover:bg-surface-brand-emphasis",
    "focus:outline-none focus:ring-2 focus:ring-ring-focus focus:ring-offset-2",
    "transition-transform hover:scale-105",
    
    // Position
    {
      "bottom-6 right-6": position === "bottom-right",
      "bottom-6 left-6": position === "bottom-left"
    },
    
    // Size
    {
      "w-12 h-12": size === "sm",
      "w-14 h-14": size === "md",
      "w-16 h-16": size === "lg"
    }
  )}
>
  <PlusIcon className="w-6 h-6" />
</button>

// FAB Extended
<button className={cn(
  "fixed bottom-6 right-6 z-fab",
  "flex items-center gap-2 px-5 h-14",
  "rounded-full bg-surface-brand text-text-inverse shadow-lg",
  "hover:bg-surface-brand-emphasis",
  "transition-transform hover:scale-105"
)}>
  <PlusIcon className="w-5 h-5" />
  <span className="font-medium">Create New</span>
</button>
```

---

## 8.5 CONFIRMATION DIALOG

### Struttura JSX:

```jsx
// CONFIRMATION DIALOG
<Dialog open={isOpen} onClose={onCancel}>
  <div className="fixed inset-0 z-modal-backdrop bg-black/50" />
  
  <div className="fixed inset-0 z-modal flex items-center justify-center p-4">
    <Dialog.Panel className="w-full max-w-md bg-surface-default rounded-xl shadow-2xl p-6">
      {/* Icon */}
      <div className={cn(
        "w-12 h-12 mx-auto mb-4 rounded-full flex items-center justify-center",
        type === "danger" && "bg-feedback-error/10",
        type === "warning" && "bg-feedback-warning/10",
        type === "info" && "bg-feedback-info/10"
      )}>
        <AlertIcon className={cn(
          "w-6 h-6",
          type === "danger" && "text-feedback-error",
          type === "warning" && "text-feedback-warning",
          type === "info" && "text-feedback-info"
        )} />
      </div>
      
      {/* Content */}
      <Dialog.Title className="text-lg font-semibold text-text-default text-center">
        {title}
      </Dialog.Title>
      <Dialog.Description className="text-sm text-text-muted text-center mt-2">
        {description}
      </Dialog.Description>
      
      {/* Actions */}
      <div className="flex gap-3 mt-6">
        <Button variant="outline" onClick={onCancel} fullWidth>
          {cancelLabel || "Cancel"}
        </Button>
        <Button 
          variant={type === "danger" ? "destructive" : "primary"} 
          onClick={onConfirm}
          fullWidth
        >
          {confirmLabel || "Confirm"}
        </Button>
      </div>
    </Dialog.Panel>
  </div>
</Dialog>
```


# ============================================================================
# PARTE II: PATTERN SPECIFICI PER CATEGORIA DI PIATTAFORMA
# ============================================================================

"""
Questi pattern sono specializzazioni dei primitivi per ogni categoria.
Combinano multiple primitive per creare componenti domain-specific.

CATEGORIE:
1. E-commerce
2. SaaS Multi-tenant
3. Dashboard/Analytics
4. Social/Chat
5. Video Streaming
6. Healthcare (HIPAA)
7. FinTech (PCI-DSS)
8. IoT (Industrial + Consumer)
"""

# ============================================================================
# CATEGORIA 1: E-COMMERCE PATTERNS
# ============================================================================

## EC-1: PRODUCT CARD

### Descrizione:
Card per visualizzare un prodotto in grid o list view.

### Struttura JSX:

```jsx
// PRODUCT CARD
<article className="group bg-surface-default border border-border-default rounded-lg overflow-hidden">
  {/* Image Container */}
  <div className="relative aspect-square bg-surface-muted overflow-hidden">
    <img 
      src={product.image} 
      alt={product.name}
      className="w-full h-full object-cover group-hover:scale-105 transition-transform duration-normal"
    />
    
    {/* Badges */}
    <div className="absolute top-2 left-2 flex flex-col gap-1">
      {product.isNew && (
        <Badge variant="primary">New</Badge>
      )}
      {product.discount && (
        <Badge variant="error">-{product.discount}%</Badge>
      )}
    </div>
    
    {/* Quick Actions */}
    <div className="absolute top-2 right-2 flex flex-col gap-1 
                    opacity-0 group-hover:opacity-100 transition-opacity">
      <IconButton 
        variant="secondary" 
        size="sm"
        icon={HeartIcon}
        aria-label="Add to wishlist"
      />
      <IconButton 
        variant="secondary" 
        size="sm"
        icon={EyeIcon}
        aria-label="Quick view"
      />
    </div>
    
    {/* Out of Stock Overlay */}
    {!product.inStock && (
      <div className="absolute inset-0 bg-black/50 flex items-center justify-center">
        <span className="text-white font-medium">Out of Stock</span>
      </div>
    )}
  </div>
  
  {/* Content */}
  <div className="p-4">
    {/* Category */}
    <span className="text-xs text-text-muted uppercase tracking-wide">
      {product.category}
    </span>
    
    {/* Name */}
    <h3 className="font-medium text-text-default mt-1 line-clamp-2">
      <a href={`/product/${product.slug}`} className="hover:text-text-brand">
        {product.name}
      </a>
    </h3>
    
    {/* Rating */}
    <div className="flex items-center gap-1 mt-2">
      <StarRating value={product.rating} size="sm" />
      <span className="text-xs text-text-muted">({product.reviewCount})</span>
    </div>
    
    {/* Price */}
    <div className="flex items-baseline gap-2 mt-2">
      <span className="text-lg font-bold text-text-default">
        ${product.price.toFixed(2)}
      </span>
      {product.originalPrice && (
        <span className="text-sm text-text-muted line-through">
          ${product.originalPrice.toFixed(2)}
        </span>
      )}
    </div>
    
    {/* Add to Cart */}
    <Button 
      variant="primary" 
      size="sm" 
      fullWidth 
      className="mt-3"
      disabled={!product.inStock}
    >
      <ShoppingCartIcon className="w-4 h-4 mr-2" />
      Add to Cart
    </Button>
  </div>
</article>
```

---

## EC-2: CART ITEM

### Struttura JSX:

```jsx
// CART ITEM
<div className="flex gap-4 py-4 border-b border-border-default">
  {/* Product Image */}
  <div className="w-20 h-20 shrink-0 bg-surface-muted rounded-md overflow-hidden">
    <img 
      src={item.image} 
      alt={item.name}
      className="w-full h-full object-cover"
    />
  </div>
  
  {/* Product Info */}
  <div className="flex-1 min-w-0">
    <div className="flex items-start justify-between">
      <div>
        <h4 className="font-medium text-text-default line-clamp-1">
          {item.name}
        </h4>
        {item.variant && (
          <p className="text-sm text-text-muted mt-0.5">
            {item.variant}
          </p>
        )}
      </div>
      
      {/* Remove Button */}
      <button 
        onClick={() => onRemove(item.id)}
        className="p-1 text-text-muted hover:text-feedback-error"
        aria-label="Remove item"
      >
        <TrashIcon className="w-4 h-4" />
      </button>
    </div>
    
    {/* Price & Quantity */}
    <div className="flex items-center justify-between mt-3">
      <div className="flex items-center border border-border-default rounded-md">
        <button 
          onClick={() => onUpdateQuantity(item.id, item.quantity - 1)}
          disabled={item.quantity <= 1}
          className="w-8 h-8 flex items-center justify-center hover:bg-interactive-hover disabled:opacity-50"
        >
          <MinusIcon className="w-4 h-4" />
        </button>
        <span className="w-10 text-center text-sm font-medium">
          {item.quantity}
        </span>
        <button 
          onClick={() => onUpdateQuantity(item.id, item.quantity + 1)}
          className="w-8 h-8 flex items-center justify-center hover:bg-interactive-hover"
        >
          <PlusIcon className="w-4 h-4" />
        </button>
      </div>
      
      <span className="font-semibold text-text-default">
        ${(item.price * item.quantity).toFixed(2)}
      </span>
    </div>
  </div>
</div>
```

---

## EC-3: CHECKOUT STEPPER

### Struttura JSX:

```jsx
// CHECKOUT STEPS: Cart → Shipping → Payment → Review
const CHECKOUT_STEPS = [
  { id: 'cart', label: 'Cart', icon: ShoppingCartIcon },
  { id: 'shipping', label: 'Shipping', icon: TruckIcon },
  { id: 'payment', label: 'Payment', icon: CreditCardIcon },
  { id: 'review', label: 'Review', icon: CheckCircleIcon }
];

<nav className="flex items-center justify-center">
  {CHECKOUT_STEPS.map((step, index) => {
    const isCompleted = index < currentStepIndex;
    const isCurrent = index === currentStepIndex;
    
    return (
      <React.Fragment key={step.id}>
        {/* Step */}
        <div className={cn(
          "flex items-center gap-2",
          isCompleted || isCurrent ? "text-text-brand" : "text-text-muted"
        )}>
          <div className={cn(
            "w-10 h-10 rounded-full flex items-center justify-center",
            isCompleted && "bg-feedback-success text-white",
            isCurrent && "bg-surface-brand text-white",
            !isCompleted && !isCurrent && "bg-surface-muted"
          )}>
            {isCompleted ? (
              <CheckIcon className="w-5 h-5" />
            ) : (
              <step.icon className="w-5 h-5" />
            )}
          </div>
          <span className={cn(
            "text-sm font-medium hidden sm:block",
            isCurrent && "text-text-default"
          )}>
            {step.label}
          </span>
        </div>
        
        {/* Connector */}
        {index < CHECKOUT_STEPS.length - 1 && (
          <div className={cn(
            "w-12 sm:w-24 h-0.5 mx-2",
            isCompleted ? "bg-feedback-success" : "bg-border-default"
          )} />
        )}
      </React.Fragment>
    );
  })}
</nav>
```

---

## EC-4: ORDER SUMMARY

### Struttura JSX:

```jsx
// ORDER SUMMARY (Sidebar)
<div className="bg-surface-muted rounded-lg p-6 sticky top-4">
  <h3 className="text-lg font-semibold text-text-default mb-4">
    Order Summary
  </h3>
  
  {/* Items Count */}
  <div className="flex justify-between text-sm mb-4">
    <span className="text-text-muted">Items ({cart.itemCount})</span>
    <span className="text-text-default">${cart.subtotal.toFixed(2)}</span>
  </div>
  
  {/* Promo Code */}
  <div className="mb-4">
    <label className="text-sm text-text-muted mb-1.5 block">Promo Code</label>
    <div className="flex gap-2">
      <Input 
        placeholder="Enter code"
        value={promoCode}
        onChange={(e) => setPromoCode(e.target.value)}
      />
      <Button variant="outline" size="sm">Apply</Button>
    </div>
  </div>
  
  {/* Costs Breakdown */}
  <div className="space-y-2 py-4 border-t border-border-default">
    <div className="flex justify-between text-sm">
      <span className="text-text-muted">Subtotal</span>
      <span className="text-text-default">${cart.subtotal.toFixed(2)}</span>
    </div>
    <div className="flex justify-between text-sm">
      <span className="text-text-muted">Shipping</span>
      <span className="text-text-default">
        {cart.shipping === 0 ? 'Free' : `$${cart.shipping.toFixed(2)}`}
      </span>
    </div>
    {cart.discount > 0 && (
      <div className="flex justify-between text-sm text-feedback-success">
        <span>Discount</span>
        <span>-${cart.discount.toFixed(2)}</span>
      </div>
    )}
    <div className="flex justify-between text-sm">
      <span className="text-text-muted">Tax</span>
      <span className="text-text-default">${cart.tax.toFixed(2)}</span>
    </div>
  </div>
  
  {/* Total */}
  <div className="flex justify-between py-4 border-t border-border-default">
    <span className="font-semibold text-text-default">Total</span>
    <span className="text-xl font-bold text-text-default">
      ${cart.total.toFixed(2)}
    </span>
  </div>
  
  {/* Checkout Button */}
  <Button variant="primary" size="lg" fullWidth>
    Proceed to Checkout
  </Button>
  
  {/* Trust Badges */}
  <div className="flex items-center justify-center gap-4 mt-4 pt-4 border-t border-border-default">
    <img src="/badges/secure.svg" alt="Secure checkout" className="h-6" />
    <img src="/badges/visa.svg" alt="Visa" className="h-6" />
    <img src="/badges/mastercard.svg" alt="Mastercard" className="h-6" />
  </div>
</div>
```

---

## EC-5: PRODUCT FILTERS

### Struttura JSX:

```jsx
// PRODUCT FILTERS (Sidebar)
<aside className="w-64 shrink-0">
  <div className="sticky top-4 space-y-6">
    {/* Active Filters */}
    {activeFilters.length > 0 && (
      <div className="space-y-2">
        <div className="flex items-center justify-between">
          <span className="text-sm font-medium text-text-default">Active Filters</span>
          <button 
            onClick={clearAllFilters}
            className="text-xs text-text-brand hover:underline"
          >
            Clear All
          </button>
        </div>
        <div className="flex flex-wrap gap-2">
          {activeFilters.map(filter => (
            <Badge 
              key={filter.id}
              variant="secondary"
              onRemove={() => removeFilter(filter.id)}
            >
              {filter.label}
            </Badge>
          ))}
        </div>
      </div>
    )}
    
    {/* Category Filter */}
    <FilterSection title="Category" defaultOpen>
      <div className="space-y-2">
        {categories.map(category => (
          <label key={category.id} className="flex items-center gap-2 cursor-pointer">
            <Checkbox 
              checked={selectedCategories.includes(category.id)}
              onChange={() => toggleCategory(category.id)}
            />
            <span className="text-sm text-text-default">{category.name}</span>
            <span className="text-xs text-text-muted ml-auto">({category.count})</span>
          </label>
        ))}
      </div>
    </FilterSection>
    
    {/* Price Range */}
    <FilterSection title="Price Range" defaultOpen>
      <div className="space-y-4">
        <RangeSlider
          min={0}
          max={1000}
          value={priceRange}
          onChange={setPriceRange}
        />
        <div className="flex items-center gap-2">
          <Input 
            type="number"
            value={priceRange[0]}
            onChange={(e) => setPriceRange([+e.target.value, priceRange[1]])}
            prefix="$"
            size="sm"
          />
          <span className="text-text-muted">—</span>
          <Input 
            type="number"
            value={priceRange[1]}
            onChange={(e) => setPriceRange([priceRange[0], +e.target.value])}
            prefix="$"
            size="sm"
          />
        </div>
      </div>
    </FilterSection>
    
    {/* Rating Filter */}
    <FilterSection title="Rating">
      <div className="space-y-2">
        {[5, 4, 3, 2, 1].map(rating => (
          <label key={rating} className="flex items-center gap-2 cursor-pointer">
            <Radio 
              checked={selectedRating === rating}
              onChange={() => setSelectedRating(rating)}
            />
            <StarRating value={rating} size="sm" />
            <span className="text-sm text-text-muted">& up</span>
          </label>
        ))}
      </div>
    </FilterSection>
    
    {/* Color Filter */}
    <FilterSection title="Color">
      <div className="flex flex-wrap gap-2">
        {colors.map(color => (
          <button
            key={color.id}
            onClick={() => toggleColor(color.id)}
            className={cn(
              "w-8 h-8 rounded-full border-2 transition-all",
              selectedColors.includes(color.id)
                ? "border-text-default scale-110"
                : "border-transparent hover:scale-105"
            )}
            style={{ backgroundColor: color.hex }}
            title={color.name}
          />
        ))}
      </div>
    </FilterSection>
  </div>
</aside>
```

# ============================================================================
# CATEGORIA 2: SAAS PATTERNS
# ============================================================================

## SAAS-1: PRICING TABLE

### Struttura JSX:

```jsx
// PRICING TABLE
<div className="grid grid-cols-1 md:grid-cols-3 gap-6 lg:gap-8">
  {plans.map(plan => (
    <div 
      key={plan.id}
      className={cn(
        "relative bg-surface-default border rounded-2xl p-6 lg:p-8",
        plan.isPopular 
          ? "border-surface-brand shadow-lg scale-105 z-10" 
          : "border-border-default"
      )}
    >
      {/* Popular Badge */}
      {plan.isPopular && (
        <div className="absolute -top-4 left-1/2 -translate-x-1/2">
          <Badge variant="primary" size="lg">Most Popular</Badge>
        </div>
      )}
      
      {/* Plan Header */}
      <div className="text-center mb-6">
        <h3 className="text-lg font-semibold text-text-default">{plan.name}</h3>
        <p className="text-sm text-text-muted mt-1">{plan.description}</p>
        
        {/* Price */}
        <div className="mt-4">
          <span className="text-4xl font-bold text-text-default">
            ${billingCycle === 'monthly' ? plan.monthlyPrice : plan.yearlyPrice}
          </span>
          <span className="text-text-muted">
            /{billingCycle === 'monthly' ? 'mo' : 'yr'}
          </span>
        </div>
        
        {billingCycle === 'yearly' && plan.yearlyDiscount && (
          <Badge variant="success" size="sm" className="mt-2">
            Save {plan.yearlyDiscount}%
          </Badge>
        )}
      </div>
      
      {/* Features */}
      <ul className="space-y-3 mb-6">
        {plan.features.map((feature, index) => (
          <li key={index} className="flex items-start gap-3">
            <CheckIcon className="w-5 h-5 text-feedback-success shrink-0 mt-0.5" />
            <span className="text-sm text-text-default">{feature}</span>
          </li>
        ))}
        {plan.notIncluded?.map((feature, index) => (
          <li key={index} className="flex items-start gap-3 opacity-50">
            <XIcon className="w-5 h-5 text-text-muted shrink-0 mt-0.5" />
            <span className="text-sm text-text-muted line-through">{feature}</span>
          </li>
        ))}
      </ul>
      
      {/* CTA */}
      <Button 
        variant={plan.isPopular ? "primary" : "outline"} 
        size="lg"
        fullWidth
      >
        {plan.ctaLabel || 'Get Started'}
      </Button>
    </div>
  ))}
</div>
```

---

## SAAS-2: ONBOARDING CHECKLIST

### Struttura JSX:

```jsx
// ONBOARDING CHECKLIST
<div className="bg-surface-default border border-border-default rounded-lg">
  {/* Header */}
  <div className="p-4 border-b border-border-default">
    <div className="flex items-center justify-between">
      <h3 className="font-semibold text-text-default">Getting Started</h3>
      <span className="text-sm text-text-muted">
        {completedSteps}/{totalSteps} completed
      </span>
    </div>
    <Progress value={(completedSteps / totalSteps) * 100} className="mt-2" />
  </div>
  
  {/* Steps */}
  <div className="divide-y divide-border-default">
    {onboardingSteps.map((step, index) => (
      <div 
        key={step.id}
        className={cn(
          "p-4 flex items-start gap-4",
          step.completed && "bg-surface-muted/30"
        )}
      >
        {/* Status */}
        <div className={cn(
          "w-8 h-8 rounded-full flex items-center justify-center shrink-0",
          step.completed 
            ? "bg-feedback-success text-white"
            : "bg-surface-muted text-text-muted"
        )}>
          {step.completed ? (
            <CheckIcon className="w-5 h-5" />
          ) : (
            <span className="text-sm font-medium">{index + 1}</span>
          )}
        </div>
        
        {/* Content */}
        <div className="flex-1">
          <h4 className={cn(
            "font-medium",
            step.completed ? "text-text-muted line-through" : "text-text-default"
          )}>
            {step.title}
          </h4>
          <p className="text-sm text-text-muted mt-0.5">{step.description}</p>
          
          {!step.completed && (
            <Button 
              variant="outline" 
              size="sm" 
              className="mt-2"
              onClick={() => onStepClick(step)}
            >
              {step.ctaLabel}
            </Button>
          )}
        </div>
        
        {/* Time Estimate */}
        {!step.completed && step.timeEstimate && (
          <span className="text-xs text-text-subtle shrink-0">
            ~{step.timeEstimate} min
          </span>
        )}
      </div>
    ))}
  </div>
</div>
```

---

## SAAS-3: USAGE METER

### Struttura JSX:

```jsx
// USAGE METER
<div className="bg-surface-default border border-border-default rounded-lg p-4">
  <div className="flex items-center justify-between mb-2">
    <div className="flex items-center gap-2">
      <span className="font-medium text-text-default">{resource.name}</span>
      <Tooltip content={resource.description}>
        <InfoIcon className="w-4 h-4 text-text-muted" />
      </Tooltip>
    </div>
    <span className="text-sm text-text-muted">
      {resource.used.toLocaleString()} / {resource.limit.toLocaleString()}
    </span>
  </div>
  
  {/* Progress Bar */}
  <div className="h-2 bg-surface-muted rounded-full overflow-hidden">
    <div 
      className={cn(
        "h-full rounded-full transition-all",
        usagePercent > 90 
          ? "bg-feedback-error"
          : usagePercent > 75
            ? "bg-feedback-warning"
            : "bg-surface-brand"
      )}
      style={{ width: `${Math.min(usagePercent, 100)}%` }}
    />
  </div>
  
  {/* Warning */}
  {usagePercent > 80 && (
    <Alert variant="warning" size="sm" className="mt-3">
      {usagePercent >= 100 
        ? "You've reached your limit. Upgrade to continue."
        : `You're at ${usagePercent}% of your ${resource.name.toLowerCase()} limit.`
      }
    </Alert>
  )}
  
  {/* Upgrade CTA */}
  {usagePercent > 75 && (
    <Button variant="outline" size="sm" className="mt-3">
      Upgrade Plan
    </Button>
  )}
</div>
```

---

## SAAS-4: TEAM MEMBER ROW

### Struttura JSX:

```jsx
// TEAM MEMBER ROW
<div className="flex items-center gap-4 py-4 border-b border-border-default">
  {/* Avatar & Info */}
  <div className="flex items-center gap-3 flex-1 min-w-0">
    <Avatar 
      src={member.avatar} 
      alt={member.name}
      size="md"
    />
    <div className="min-w-0">
      <div className="flex items-center gap-2">
        <span className="font-medium text-text-default truncate">
          {member.name}
        </span>
        {member.isCurrentUser && (
          <Badge variant="secondary" size="sm">You</Badge>
        )}
        {member.isPending && (
          <Badge variant="warning" size="sm">Pending</Badge>
        )}
      </div>
      <span className="text-sm text-text-muted truncate block">
        {member.email}
      </span>
    </div>
  </div>
  
  {/* Role */}
  <div className="w-32">
    <Select
      value={member.role}
      onChange={(role) => onRoleChange(member.id, role)}
      disabled={member.isCurrentUser || !canChangeRole}
      options={[
        { value: 'admin', label: 'Admin' },
        { value: 'member', label: 'Member' },
        { value: 'viewer', label: 'Viewer' }
      ]}
    />
  </div>
  
  {/* Actions */}
  <div className="flex items-center gap-2">
    {member.isPending && (
      <Button variant="ghost" size="sm" onClick={() => onResendInvite(member.id)}>
        Resend
      </Button>
    )}
    {!member.isCurrentUser && (
      <DropdownMenu>
        <DropdownMenu.Trigger asChild>
          <IconButton variant="ghost" icon={MoreHorizontalIcon} />
        </DropdownMenu.Trigger>
        <DropdownMenu.Content>
          <DropdownMenu.Item onClick={() => onRemove(member.id)} destructive>
            Remove from team
          </DropdownMenu.Item>
        </DropdownMenu.Content>
      </DropdownMenu>
    )}
  </div>
</div>
```


# ============================================================================
# CATEGORIA 3: DASHBOARD / ANALYTICS PATTERNS
# ============================================================================

## DASH-1: KPI CARD GRID

### Struttura JSX:

```jsx
// KPI CARD GRID
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
  {kpis.map(kpi => (
    <div 
      key={kpi.id}
      className="bg-surface-default border border-border-default rounded-lg p-4"
    >
      <div className="flex items-start justify-between">
        <div className="p-2 rounded-lg bg-surface-muted">
          <kpi.icon className="w-5 h-5 text-text-brand" />
        </div>
        <TrendIndicator 
          direction={kpi.trend.direction}
          value={kpi.trend.value}
        />
      </div>
      
      <div className="mt-4">
        <span className="text-2xl font-bold text-text-default">
          {kpi.formatValue ? kpi.formatValue(kpi.value) : kpi.value}
        </span>
        <p className="text-sm text-text-muted mt-1">{kpi.label}</p>
      </div>
      
      {/* Mini Sparkline */}
      {kpi.sparklineData && (
        <div className="mt-3 h-8">
          <Sparkline 
            data={kpi.sparklineData}
            color={kpi.trend.direction === 'up' ? 'success' : 'error'}
          />
        </div>
      )}
    </div>
  ))}
</div>
```

---

## DASH-2: DATA WIDGET

### Struttura JSX:

```jsx
// DATA WIDGET (Generic Container)
<div className="bg-surface-default border border-border-default rounded-lg flex flex-col">
  {/* Widget Header */}
  <div className="flex items-center justify-between p-4 border-b border-border-default">
    <div>
      <h3 className="font-semibold text-text-default">{title}</h3>
      {subtitle && (
        <p className="text-sm text-text-muted mt-0.5">{subtitle}</p>
      )}
    </div>
    
    <div className="flex items-center gap-2">
      {/* Time Range Tabs */}
      {timeRanges && (
        <SegmentedControl
          options={timeRanges}
          value={selectedRange}
          onChange={setSelectedRange}
          size="sm"
        />
      )}
      
      {/* Refresh */}
      <IconButton 
        variant="ghost" 
        size="sm" 
        icon={RefreshCwIcon}
        onClick={onRefresh}
        loading={isRefreshing}
      />
      
      {/* More Options */}
      <DropdownMenu>
        <DropdownMenu.Trigger asChild>
          <IconButton variant="ghost" size="sm" icon={MoreVerticalIcon} />
        </DropdownMenu.Trigger>
        <DropdownMenu.Content>
          <DropdownMenu.Item>Export CSV</DropdownMenu.Item>
          <DropdownMenu.Item>Export PNG</DropdownMenu.Item>
          <DropdownMenu.Separator />
          <DropdownMenu.Item>Edit Widget</DropdownMenu.Item>
          <DropdownMenu.Item>Remove Widget</DropdownMenu.Item>
        </DropdownMenu.Content>
      </DropdownMenu>
    </div>
  </div>
  
  {/* Widget Body */}
  <div className="flex-1 p-4 min-h-[200px]">
    {isLoading ? (
      <div className="h-full flex items-center justify-center">
        <Spinner size="lg" />
      </div>
    ) : error ? (
      <EmptyState 
        icon={AlertTriangleIcon}
        title="Failed to load data"
        description={error.message}
        action={{ label: 'Retry', onClick: onRefresh }}
      />
    ) : (
      children
    )}
  </div>
  
  {/* Widget Footer (opzionale) */}
  {footer && (
    <div className="p-4 border-t border-border-muted bg-surface-muted/30">
      {footer}
    </div>
  )}
</div>
```

---

## DASH-3: FILTER BAR

### Struttura JSX:

```jsx
// DASHBOARD FILTER BAR
<div className="flex flex-wrap items-center gap-3 p-4 bg-surface-muted/50 rounded-lg">
  {/* Date Range Picker */}
  <div className="flex items-center gap-2">
    <CalendarIcon className="w-4 h-4 text-text-muted" />
    <DateRangePicker
      value={dateRange}
      onChange={setDateRange}
      presets={[
        { label: 'Today', value: 'today' },
        { label: 'Last 7 days', value: '7d' },
        { label: 'Last 30 days', value: '30d' },
        { label: 'Last 90 days', value: '90d' },
        { label: 'Custom', value: 'custom' }
      ]}
    />
  </div>
  
  <div className="w-px h-6 bg-border-default" />
  
  {/* Dimension Filters */}
  {dimensions.map(dim => (
    <Select
      key={dim.id}
      value={filters[dim.id]}
      onChange={(value) => setFilter(dim.id, value)}
      options={dim.options}
      placeholder={dim.label}
      size="sm"
      multiple={dim.multiple}
    />
  ))}
  
  {/* Comparison Toggle */}
  <div className="flex items-center gap-2 ml-auto">
    <span className="text-sm text-text-muted">Compare to</span>
    <Select
      value={comparisonPeriod}
      onChange={setComparisonPeriod}
      options={[
        { value: 'previous', label: 'Previous period' },
        { value: 'year', label: 'Same period last year' },
        { value: 'none', label: 'No comparison' }
      ]}
      size="sm"
    />
  </div>
  
  {/* Clear Filters */}
  {hasActiveFilters && (
    <Button 
      variant="ghost" 
      size="sm"
      onClick={clearFilters}
    >
      Clear All
    </Button>
  )}
</div>
```

---

## DASH-4: ACTIVITY FEED

### Struttura JSX:

```jsx
// ACTIVITY FEED
<div className="bg-surface-default border border-border-default rounded-lg">
  <div className="p-4 border-b border-border-default">
    <h3 className="font-semibold text-text-default">Recent Activity</h3>
  </div>
  
  <div className="divide-y divide-border-default max-h-96 overflow-y-auto">
    {activities.map(activity => (
      <div key={activity.id} className="p-4 hover:bg-surface-muted/30">
        <div className="flex items-start gap-3">
          {/* Actor Avatar or Icon */}
          {activity.actor ? (
            <Avatar 
              src={activity.actor.avatar} 
              alt={activity.actor.name}
              size="sm"
            />
          ) : (
            <div className={cn(
              "w-8 h-8 rounded-full flex items-center justify-center",
              activityTypeStyles[activity.type].bg
            )}>
              <ActivityIcon 
                type={activity.type} 
                className={cn("w-4 h-4", activityTypeStyles[activity.type].text)}
              />
            </div>
          )}
          
          {/* Content */}
          <div className="flex-1 min-w-0">
            <p className="text-sm text-text-default">
              {activity.actor && (
                <span className="font-medium">{activity.actor.name}</span>
              )}
              {' '}{activity.action}{' '}
              {activity.target && (
                <a href={activity.target.href} className="font-medium text-text-brand hover:underline">
                  {activity.target.name}
                </a>
              )}
            </p>
            <span className="text-xs text-text-subtle">
              {formatRelativeTime(activity.timestamp)}
            </span>
          </div>
        </div>
      </div>
    ))}
  </div>
  
  {hasMore && (
    <div className="p-4 border-t border-border-default text-center">
      <Button variant="ghost" size="sm">View All Activity</Button>
    </div>
  )}
</div>
```

# ============================================================================
# CATEGORIA 4: SOCIAL / CHAT PATTERNS
# ============================================================================

## SOCIAL-1: POST CARD (Feed Item)

### Struttura JSX:

```jsx
// POST CARD
<article className="bg-surface-default border border-border-default rounded-lg">
  {/* Post Header */}
  <div className="flex items-center gap-3 p-4">
    <Avatar src={post.author.avatar} alt={post.author.name} size="md" />
    <div className="flex-1 min-w-0">
      <div className="flex items-center gap-2">
        <span className="font-semibold text-text-default truncate">
          {post.author.name}
        </span>
        {post.author.isVerified && (
          <VerifiedBadge className="w-4 h-4 text-text-brand" />
        )}
      </div>
      <span className="text-sm text-text-muted">
        @{post.author.username} · {formatRelativeTime(post.createdAt)}
      </span>
    </div>
    
    <DropdownMenu>
      <DropdownMenu.Trigger asChild>
        <IconButton variant="ghost" icon={MoreHorizontalIcon} />
      </DropdownMenu.Trigger>
      <DropdownMenu.Content>
        <DropdownMenu.Item>Copy Link</DropdownMenu.Item>
        <DropdownMenu.Item>Embed Post</DropdownMenu.Item>
        <DropdownMenu.Separator />
        <DropdownMenu.Item>Report</DropdownMenu.Item>
        {post.isOwn && (
          <DropdownMenu.Item destructive>Delete</DropdownMenu.Item>
        )}
      </DropdownMenu.Content>
    </DropdownMenu>
  </div>
  
  {/* Post Content */}
  <div className="px-4 pb-3">
    <p className="text-text-default whitespace-pre-wrap">{post.content}</p>
  </div>
  
  {/* Media (opzionale) */}
  {post.media && (
    <div className={cn(
      "grid gap-1",
      post.media.length === 1 && "grid-cols-1",
      post.media.length === 2 && "grid-cols-2",
      post.media.length >= 3 && "grid-cols-2 grid-rows-2"
    )}>
      {post.media.slice(0, 4).map((media, index) => (
        <div 
          key={index}
          className={cn(
            "relative aspect-video bg-surface-muted overflow-hidden",
            post.media.length === 3 && index === 0 && "row-span-2"
          )}
        >
          <img 
            src={media.url} 
            alt=""
            className="w-full h-full object-cover"
          />
          {post.media.length > 4 && index === 3 && (
            <div className="absolute inset-0 bg-black/50 flex items-center justify-center">
              <span className="text-white text-xl font-bold">
                +{post.media.length - 4}
              </span>
            </div>
          )}
        </div>
      ))}
    </div>
  )}
  
  {/* Engagement Stats */}
  <div className="flex items-center gap-6 px-4 py-3 text-sm text-text-muted">
    <span>{formatNumber(post.likes)} likes</span>
    <span>{formatNumber(post.comments)} comments</span>
    <span>{formatNumber(post.shares)} shares</span>
  </div>
  
  {/* Action Bar */}
  <div className="flex items-center justify-around border-t border-border-default">
    <button 
      onClick={() => onLike(post.id)}
      className={cn(
        "flex-1 flex items-center justify-center gap-2 py-3",
        "hover:bg-surface-muted transition-colors",
        post.isLiked && "text-feedback-error"
      )}
    >
      <HeartIcon className={cn("w-5 h-5", post.isLiked && "fill-current")} />
      <span className="text-sm font-medium">Like</span>
    </button>
    
    <button className="flex-1 flex items-center justify-center gap-2 py-3 hover:bg-surface-muted">
      <MessageCircleIcon className="w-5 h-5" />
      <span className="text-sm font-medium">Comment</span>
    </button>
    
    <button className="flex-1 flex items-center justify-center gap-2 py-3 hover:bg-surface-muted">
      <ShareIcon className="w-5 h-5" />
      <span className="text-sm font-medium">Share</span>
    </button>
  </div>
</article>
```

---

## SOCIAL-2: CHAT MESSAGE

### Struttura JSX:

```jsx
// CHAT MESSAGE
<div className={cn(
  "flex gap-2 max-w-[80%]",
  isOwn ? "ml-auto flex-row-reverse" : "mr-auto"
)}>
  {/* Avatar (solo per altri) */}
  {!isOwn && (
    <Avatar src={message.sender.avatar} alt={message.sender.name} size="sm" />
  )}
  
  {/* Message Bubble */}
  <div className={cn(
    "rounded-2xl px-4 py-2",
    isOwn 
      ? "bg-surface-brand text-text-inverse rounded-br-sm"
      : "bg-surface-muted text-text-default rounded-bl-sm"
  )}>
    {/* Reply Preview (se presente) */}
    {message.replyTo && (
      <div className={cn(
        "mb-2 pl-2 border-l-2 text-xs opacity-70",
        isOwn ? "border-white/50" : "border-text-muted"
      )}>
        <span className="font-medium">{message.replyTo.sender.name}</span>
        <p className="truncate">{message.replyTo.content}</p>
      </div>
    )}
    
    {/* Content */}
    <p className="text-sm whitespace-pre-wrap break-words">{message.content}</p>
    
    {/* Media Attachment */}
    {message.attachment && (
      <div className="mt-2">
        {message.attachment.type === 'image' ? (
          <img 
            src={message.attachment.url} 
            alt=""
            className="rounded-lg max-w-[200px]"
          />
        ) : (
          <a 
            href={message.attachment.url}
            className="flex items-center gap-2 p-2 bg-white/10 rounded"
          >
            <FileIcon className="w-5 h-5" />
            <span className="text-sm truncate">{message.attachment.name}</span>
          </a>
        )}
      </div>
    )}
    
    {/* Timestamp & Status */}
    <div className={cn(
      "flex items-center gap-1 mt-1 text-xs",
      isOwn ? "text-white/70 justify-end" : "text-text-subtle"
    )}>
      <span>{formatTime(message.timestamp)}</span>
      {isOwn && (
        <MessageStatus status={message.status} />
      )}
    </div>
  </div>
</div>

// MESSAGE STATUS COMPONENT
function MessageStatus({ status }) {
  if (status === 'sending') return <ClockIcon className="w-3 h-3" />;
  if (status === 'sent') return <CheckIcon className="w-3 h-3" />;
  if (status === 'delivered') return <CheckCheckIcon className="w-3 h-3" />;
  if (status === 'read') return <CheckCheckIcon className="w-3 h-3 text-blue-400" />;
  return null;
}
```

---

## SOCIAL-3: CHAT INPUT

### Struttura JSX:

```jsx
// CHAT INPUT BAR
<div className="border-t border-border-default bg-surface-default p-3">
  {/* Reply Preview */}
  {replyingTo && (
    <div className="flex items-center justify-between mb-2 p-2 bg-surface-muted rounded-lg">
      <div className="flex-1 min-w-0">
        <span className="text-xs text-text-muted">
          Replying to {replyingTo.sender.name}
        </span>
        <p className="text-sm text-text-default truncate">{replyingTo.content}</p>
      </div>
      <IconButton 
        variant="ghost" 
        size="sm" 
        icon={XIcon}
        onClick={cancelReply}
      />
    </div>
  )}
  
  {/* Input Area */}
  <div className="flex items-end gap-2">
    {/* Attachment Button */}
    <IconButton 
      variant="ghost"
      icon={PaperclipIcon}
      onClick={openFilePicker}
    />
    
    {/* Text Input */}
    <div className="flex-1 relative">
      <textarea
        ref={inputRef}
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        onKeyDown={handleKeyDown}
        placeholder="Type a message..."
        rows={1}
        className={cn(
          "w-full px-4 py-2 rounded-2xl resize-none",
          "bg-surface-muted border-0",
          "text-sm text-text-default placeholder:text-text-subtle",
          "focus:outline-none focus:ring-2 focus:ring-ring-focus",
          "max-h-32"
        )}
        style={{ height: 'auto' }}
      />
      
      {/* Emoji Picker Trigger */}
      <button 
        className="absolute right-3 top-1/2 -translate-y-1/2"
        onClick={toggleEmojiPicker}
      >
        <SmileIcon className="w-5 h-5 text-text-muted hover:text-text-default" />
      </button>
    </div>
    
    {/* Send Button */}
    <IconButton
      variant="primary"
      icon={message.trim() ? SendIcon : MicIcon}
      onClick={message.trim() ? handleSend : startVoiceMessage}
    />
  </div>
  
  {/* Typing Indicator */}
  {typingUsers.length > 0 && (
    <div className="mt-2 text-xs text-text-muted">
      <TypingIndicator users={typingUsers} />
    </div>
  )}
</div>
```

---

## SOCIAL-4: NOTIFICATION ITEM

### Struttura JSX:

```jsx
// NOTIFICATION ITEM
<a 
  href={notification.href}
  className={cn(
    "flex items-start gap-3 p-4 hover:bg-surface-muted transition-colors",
    !notification.isRead && "bg-surface-brand/5"
  )}
>
  {/* Icon or Avatar */}
  <div className="relative">
    {notification.actor ? (
      <Avatar 
        src={notification.actor.avatar} 
        alt={notification.actor.name}
        size="md"
      />
    ) : (
      <div className={cn(
        "w-10 h-10 rounded-full flex items-center justify-center",
        notificationTypeStyles[notification.type].bg
      )}>
        <NotificationIcon 
          type={notification.type}
          className={cn("w-5 h-5", notificationTypeStyles[notification.type].text)}
        />
      </div>
    )}
    
    {/* Type Badge */}
    {notification.actor && (
      <div className={cn(
        "absolute -bottom-1 -right-1 w-5 h-5 rounded-full",
        "flex items-center justify-center border-2 border-surface-default",
        notificationTypeStyles[notification.type].bg
      )}>
        <NotificationIcon 
          type={notification.type}
          className={cn("w-3 h-3", notificationTypeStyles[notification.type].text)}
        />
      </div>
    )}
  </div>
  
  {/* Content */}
  <div className="flex-1 min-w-0">
    <p className="text-sm text-text-default">
      {notification.actor && (
        <span className="font-semibold">{notification.actor.name}</span>
      )}
      {' '}{notification.message}
    </p>
    <span className="text-xs text-text-subtle">
      {formatRelativeTime(notification.timestamp)}
    </span>
  </div>
  
  {/* Unread Indicator */}
  {!notification.isRead && (
    <div className="w-2 h-2 rounded-full bg-surface-brand shrink-0 mt-2" />
  )}
</a>
```


# ============================================================================
# CATEGORIA 5: VIDEO STREAMING PATTERNS
# ============================================================================

## VIDEO-1: VIDEO PLAYER

### Struttura JSX:

```jsx
// VIDEO PLAYER
<div 
  className="relative aspect-video bg-black rounded-lg overflow-hidden group"
  onMouseMove={showControls}
  onMouseLeave={hideControlsDelayed}
>
  {/* Video Element */}
  <video
    ref={videoRef}
    src={videoUrl}
    poster={posterUrl}
    className="w-full h-full object-contain"
    onClick={togglePlayPause}
  />
  
  {/* Loading Spinner */}
  {isBuffering && (
    <div className="absolute inset-0 flex items-center justify-center bg-black/30">
      <Spinner size="lg" className="text-white" />
    </div>
  )}
  
  {/* Play Button Overlay (paused state) */}
  {!isPlaying && !isBuffering && (
    <button 
      onClick={play}
      className="absolute inset-0 flex items-center justify-center bg-black/30 
                 opacity-0 group-hover:opacity-100 transition-opacity"
    >
      <div className="w-20 h-20 rounded-full bg-white/20 backdrop-blur 
                      flex items-center justify-center">
        <PlayIcon className="w-10 h-10 text-white ml-1" />
      </div>
    </button>
  )}
  
  {/* Controls Overlay */}
  <div className={cn(
    "absolute inset-x-0 bottom-0 bg-gradient-to-t from-black/80 to-transparent",
    "p-4 transition-opacity",
    controlsVisible ? "opacity-100" : "opacity-0"
  )}>
    {/* Progress Bar */}
    <div className="mb-3">
      <VideoProgressBar
        currentTime={currentTime}
        duration={duration}
        buffered={buffered}
        onSeek={seekTo}
        chapters={chapters}
      />
    </div>
    
    {/* Control Buttons */}
    <div className="flex items-center gap-3">
      {/* Play/Pause */}
      <IconButton 
        variant="ghost"
        icon={isPlaying ? PauseIcon : PlayIcon}
        onClick={togglePlayPause}
        className="text-white"
      />
      
      {/* Skip Back/Forward */}
      <IconButton 
        variant="ghost"
        icon={SkipBackIcon}
        onClick={() => skip(-10)}
        className="text-white"
      />
      <IconButton 
        variant="ghost"
        icon={SkipForwardIcon}
        onClick={() => skip(10)}
        className="text-white"
      />
      
      {/* Volume */}
      <div className="flex items-center gap-2 group/volume">
        <IconButton 
          variant="ghost"
          icon={volume === 0 ? VolumeXIcon : volume < 0.5 ? Volume1Icon : Volume2Icon}
          onClick={toggleMute}
          className="text-white"
        />
        <input
          type="range"
          min={0}
          max={1}
          step={0.1}
          value={volume}
          onChange={(e) => setVolume(parseFloat(e.target.value))}
          className="w-0 group-hover/volume:w-20 transition-all"
        />
      </div>
      
      {/* Time Display */}
      <span className="text-sm text-white/80 tabular-nums">
        {formatTime(currentTime)} / {formatTime(duration)}
      </span>
      
      {/* Spacer */}
      <div className="flex-1" />
      
      {/* Subtitles */}
      {subtitleTracks.length > 0 && (
        <Popover>
          <Popover.Button asChild>
            <IconButton variant="ghost" icon={SubtitlesIcon} className="text-white" />
          </Popover.Button>
          <Popover.Panel className="absolute bottom-full mb-2 right-0 bg-black/90 rounded-lg p-2">
            {subtitleTracks.map(track => (
              <button
                key={track.id}
                onClick={() => setSubtitleTrack(track.id)}
                className={cn(
                  "block w-full text-left px-3 py-1.5 rounded text-sm",
                  activeSubtitle === track.id ? "bg-white/20 text-white" : "text-white/70"
                )}
              >
                {track.label}
              </button>
            ))}
          </Popover.Panel>
        </Popover>
      )}
      
      {/* Settings */}
      <Popover>
        <Popover.Button asChild>
          <IconButton variant="ghost" icon={SettingsIcon} className="text-white" />
        </Popover.Button>
        <Popover.Panel className="absolute bottom-full mb-2 right-0 bg-black/90 rounded-lg p-2 min-w-[200px]">
          {/* Quality */}
          <div className="py-2">
            <span className="text-xs text-white/50 uppercase px-3">Quality</span>
            {qualityOptions.map(q => (
              <button
                key={q.value}
                onClick={() => setQuality(q.value)}
                className={cn(
                  "flex items-center justify-between w-full px-3 py-1.5 rounded text-sm",
                  quality === q.value ? "bg-white/20 text-white" : "text-white/70"
                )}
              >
                {q.label}
                {q.isHD && <Badge variant="primary" size="sm">HD</Badge>}
              </button>
            ))}
          </div>
          
          {/* Playback Speed */}
          <div className="py-2 border-t border-white/10">
            <span className="text-xs text-white/50 uppercase px-3">Speed</span>
            {[0.5, 0.75, 1, 1.25, 1.5, 2].map(speed => (
              <button
                key={speed}
                onClick={() => setPlaybackSpeed(speed)}
                className={cn(
                  "block w-full text-left px-3 py-1.5 rounded text-sm",
                  playbackSpeed === speed ? "bg-white/20 text-white" : "text-white/70"
                )}
              >
                {speed}x
              </button>
            ))}
          </div>
        </Popover.Panel>
      </Popover>
      
      {/* Fullscreen */}
      <IconButton 
        variant="ghost"
        icon={isFullscreen ? MinimizeIcon : MaximizeIcon}
        onClick={toggleFullscreen}
        className="text-white"
      />
    </div>
  </div>
</div>
```

---

## VIDEO-2: CONTENT CARD (Netflix-style)

### Struttura JSX:

```jsx
// VIDEO CONTENT CARD
<div className="group relative">
  {/* Thumbnail */}
  <div className="relative aspect-video rounded-md overflow-hidden">
    <img 
      src={content.thumbnail}
      alt={content.title}
      className="w-full h-full object-cover transition-transform duration-300 
                 group-hover:scale-105"
    />
    
    {/* Duration Badge */}
    <div className="absolute bottom-2 right-2 px-1.5 py-0.5 bg-black/80 
                    rounded text-xs text-white">
      {formatDuration(content.duration)}
    </div>
    
    {/* Progress Bar (if watched) */}
    {content.watchProgress > 0 && (
      <div className="absolute bottom-0 left-0 right-0 h-1 bg-white/30">
        <div 
          className="h-full bg-red-600"
          style={{ width: `${(content.watchProgress / content.duration) * 100}%` }}
        />
      </div>
    )}
  </div>
  
  {/* Hover Card (expanded info) */}
  <div className="absolute inset-x-0 top-0 z-10 
                  opacity-0 invisible scale-100 
                  group-hover:opacity-100 group-hover:visible group-hover:scale-110
                  transition-all duration-300 origin-center
                  pointer-events-none group-hover:pointer-events-auto">
    <div className="bg-surface-inverse rounded-lg shadow-2xl overflow-hidden">
      {/* Preview Video/Image */}
      <div className="aspect-video relative">
        <video
          src={content.previewUrl}
          autoPlay
          muted
          loop
          className="w-full h-full object-cover"
        />
      </div>
      
      {/* Info */}
      <div className="p-3">
        {/* Quick Actions */}
        <div className="flex items-center gap-2 mb-2">
          <IconButton variant="secondary" icon={PlayIcon} size="sm" />
          <IconButton variant="outline" icon={PlusIcon} size="sm" />
          <IconButton variant="outline" icon={ThumbsUpIcon} size="sm" />
          <div className="flex-1" />
          <IconButton variant="outline" icon={ChevronDownIcon} size="sm" />
        </div>
        
        {/* Metadata */}
        <div className="flex items-center gap-2 text-xs text-white/70 mb-2">
          <span className="text-green-500 font-medium">{content.matchScore}% Match</span>
          <span className="border border-white/30 px-1">{content.rating}</span>
          <span>{content.year}</span>
          <span>{formatDuration(content.duration)}</span>
        </div>
        
        {/* Tags */}
        <div className="flex flex-wrap gap-1">
          {content.genres.slice(0, 3).map((genre, i) => (
            <span key={genre} className="text-xs text-white/70">
              {genre}{i < 2 && ' •'}
            </span>
          ))}
        </div>
      </div>
    </div>
  </div>
  
  {/* Basic Info (visible when not hovered) */}
  <div className="mt-2">
    <h4 className="text-sm font-medium text-text-default truncate">
      {content.title}
    </h4>
  </div>
</div>
```

---

## VIDEO-3: CONTENT ROW (Horizontal Carousel)

### Struttura JSX:

```jsx
// CONTENT ROW
<section className="py-4">
  {/* Row Header */}
  <div className="flex items-center justify-between mb-3 px-4">
    <h2 className="text-lg font-semibold text-text-default">{title}</h2>
    <Button variant="ghost" size="sm">
      See All <ChevronRightIcon className="w-4 h-4 ml-1" />
    </Button>
  </div>
  
  {/* Carousel */}
  <div className="relative group/row">
    {/* Scroll Buttons */}
    <button
      onClick={() => scroll('left')}
      className={cn(
        "absolute left-0 top-0 bottom-0 w-12 z-10",
        "flex items-center justify-center",
        "bg-gradient-to-r from-surface-default to-transparent",
        "opacity-0 group-hover/row:opacity-100 transition-opacity",
        !canScrollLeft && "hidden"
      )}
    >
      <ChevronLeftIcon className="w-8 h-8 text-text-default" />
    </button>
    
    <button
      onClick={() => scroll('right')}
      className={cn(
        "absolute right-0 top-0 bottom-0 w-12 z-10",
        "flex items-center justify-center",
        "bg-gradient-to-l from-surface-default to-transparent",
        "opacity-0 group-hover/row:opacity-100 transition-opacity",
        !canScrollRight && "hidden"
      )}
    >
      <ChevronRightIcon className="w-8 h-8 text-text-default" />
    </button>
    
    {/* Scrollable Container */}
    <div
      ref={scrollRef}
      className="flex gap-3 overflow-x-auto scrollbar-hide px-4 
                 scroll-smooth snap-x snap-mandatory"
    >
      {items.map(item => (
        <div 
          key={item.id}
          className="flex-none w-[200px] lg:w-[240px] snap-start"
        >
          <VideoContentCard content={item} />
        </div>
      ))}
    </div>
  </div>
</section>
```

# ============================================================================
# CATEGORIA 6: HEALTHCARE (HIPAA) PATTERNS
# ============================================================================

## HEALTH-1: PATIENT INFO CARD

### Struttura JSX:

```jsx
// PATIENT INFO CARD (HIPAA Compliant)
<div className="bg-surface-default border border-border-default rounded-lg">
  {/* Patient Header */}
  <div className="flex items-center gap-4 p-4 border-b border-border-default">
    <Avatar 
      src={patient.photo} 
      alt={`${patient.firstName} ${patient.lastName}`}
      size="lg"
      fallback={`${patient.firstName[0]}${patient.lastName[0]}`}
    />
    <div className="flex-1">
      <div className="flex items-center gap-2">
        <h3 className="font-semibold text-text-default">
          {patient.lastName}, {patient.firstName}
        </h3>
        {patient.alerts && patient.alerts.length > 0 && (
          <Badge variant="error" size="sm">
            {patient.alerts.length} Alert{patient.alerts.length > 1 ? 's' : ''}
          </Badge>
        )}
      </div>
      <div className="flex items-center gap-4 mt-1 text-sm text-text-muted">
        <span>DOB: {formatDate(patient.dob)}</span>
        <span>MRN: {patient.mrn}</span>
        <span>Age: {calculateAge(patient.dob)}</span>
      </div>
    </div>
    
    {/* Quick Actions */}
    <div className="flex gap-2">
      <Button variant="outline" size="sm">
        <PhoneIcon className="w-4 h-4 mr-1" />
        Call
      </Button>
      <Button variant="outline" size="sm">
        <MessageSquareIcon className="w-4 h-4 mr-1" />
        Message
      </Button>
    </div>
  </div>
  
  {/* Patient Details Grid */}
  <div className="grid grid-cols-2 lg:grid-cols-4 gap-4 p-4">
    <InfoItem label="Primary Care" value={patient.primaryCare || 'Not assigned'} />
    <InfoItem label="Insurance" value={patient.insurance?.provider || 'Self-pay'} />
    <InfoItem label="Phone" value={formatPhone(patient.phone)} />
    <InfoItem label="Preferred Language" value={patient.language || 'English'} />
  </div>
  
  {/* Allergies & Conditions */}
  <div className="grid grid-cols-2 gap-4 p-4 border-t border-border-default bg-surface-muted/30">
    <div>
      <h4 className="text-xs font-semibold text-text-muted uppercase mb-2">
        Allergies
      </h4>
      <div className="flex flex-wrap gap-1">
        {patient.allergies?.length > 0 ? (
          patient.allergies.map(allergy => (
            <Badge 
              key={allergy.id}
              variant={allergy.severity === 'severe' ? 'error' : 'warning'}
              size="sm"
            >
              {allergy.name}
            </Badge>
          ))
        ) : (
          <span className="text-sm text-text-muted">No known allergies</span>
        )}
      </div>
    </div>
    <div>
      <h4 className="text-xs font-semibold text-text-muted uppercase mb-2">
        Active Conditions
      </h4>
      <div className="flex flex-wrap gap-1">
        {patient.conditions?.slice(0, 5).map(condition => (
          <Badge key={condition.id} variant="secondary" size="sm">
            {condition.name}
          </Badge>
        ))}
        {patient.conditions?.length > 5 && (
          <Badge variant="secondary" size="sm">
            +{patient.conditions.length - 5} more
          </Badge>
        )}
      </div>
    </div>
  </div>
</div>
```

---

## HEALTH-2: VITALS DISPLAY

### Struttura JSX:

```jsx
// VITALS DISPLAY
<div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-6 gap-3">
  {vitals.map(vital => {
    const status = getVitalStatus(vital);
    
    return (
      <div 
        key={vital.id}
        className={cn(
          "bg-surface-default border rounded-lg p-3",
          status === 'critical' && "border-feedback-error bg-feedback-error/5",
          status === 'warning' && "border-feedback-warning bg-feedback-warning/5",
          status === 'normal' && "border-border-default"
        )}
      >
        <div className="flex items-center justify-between mb-1">
          <span className="text-xs text-text-muted">{vital.label}</span>
          {status !== 'normal' && (
            <AlertTriangleIcon className={cn(
              "w-4 h-4",
              status === 'critical' ? "text-feedback-error" : "text-feedback-warning"
            )} />
          )}
        </div>
        <div className="flex items-baseline gap-1">
          <span className={cn(
            "text-xl font-bold",
            status === 'critical' && "text-feedback-error",
            status === 'warning' && "text-feedback-warning",
            status === 'normal' && "text-text-default"
          )}>
            {vital.value}
          </span>
          <span className="text-xs text-text-muted">{vital.unit}</span>
        </div>
        <div className="text-xs text-text-subtle mt-1">
          {formatTime(vital.timestamp)}
        </div>
      </div>
    );
  })}
</div>
```

---

## HEALTH-3: APPOINTMENT SCHEDULER

### Struttura JSX:

```jsx
// APPOINTMENT SCHEDULER
<div className="bg-surface-default border border-border-default rounded-lg">
  {/* Calendar Header */}
  <div className="flex items-center justify-between p-4 border-b border-border-default">
    <div className="flex items-center gap-2">
      <Button variant="outline" size="sm" onClick={goToPreviousWeek}>
        <ChevronLeftIcon className="w-4 h-4" />
      </Button>
      <Button variant="outline" size="sm" onClick={goToNextWeek}>
        <ChevronRightIcon className="w-4 h-4" />
      </Button>
      <Button variant="ghost" size="sm" onClick={goToToday}>Today</Button>
    </div>
    <h3 className="font-semibold text-text-default">
      {format(weekStart, 'MMMM yyyy')}
    </h3>
    <div className="flex items-center gap-2">
      <Select
        value={selectedProvider}
        onChange={setSelectedProvider}
        options={providers}
        placeholder="All Providers"
      />
      <Select
        value={selectedType}
        onChange={setSelectedType}
        options={appointmentTypes}
        placeholder="All Types"
      />
    </div>
  </div>
  
  {/* Time Slots Grid */}
  <div className="overflow-x-auto">
    <div className="min-w-[800px]">
      {/* Day Headers */}
      <div className="grid grid-cols-8 border-b border-border-default">
        <div className="p-2 text-center text-sm text-text-muted">Time</div>
        {weekDays.map(day => (
          <div 
            key={day.date}
            className={cn(
              "p-2 text-center",
              isToday(day.date) && "bg-surface-brand/5"
            )}
          >
            <div className="text-xs text-text-muted">{format(day.date, 'EEE')}</div>
            <div className={cn(
              "text-lg font-semibold",
              isToday(day.date) ? "text-text-brand" : "text-text-default"
            )}>
              {format(day.date, 'd')}
            </div>
          </div>
        ))}
      </div>
      
      {/* Time Slots */}
      {timeSlots.map(time => (
        <div key={time} className="grid grid-cols-8 border-b border-border-muted">
          <div className="p-2 text-xs text-text-muted text-right pr-4">
            {time}
          </div>
          {weekDays.map(day => {
            const appointment = getAppointment(day.date, time);
            
            return (
              <div 
                key={`${day.date}-${time}`}
                className={cn(
                  "p-1 min-h-[60px] border-l border-border-muted",
                  isToday(day.date) && "bg-surface-brand/5"
                )}
              >
                {appointment ? (
                  <AppointmentSlot appointment={appointment} />
                ) : (
                  <button
                    onClick={() => openBookingModal(day.date, time)}
                    className="w-full h-full rounded hover:bg-interactive-hover 
                               transition-colors flex items-center justify-center
                               opacity-0 hover:opacity-100"
                  >
                    <PlusIcon className="w-4 h-4 text-text-muted" />
                  </button>
                )}
              </div>
            );
          })}
        </div>
      ))}
    </div>
  </div>
</div>
```

---

## HEALTH-4: SECURE MESSAGE (PHI)

### Struttura JSX:

```jsx
// SECURE MESSAGE (HIPAA-Compliant)
<div className="bg-surface-default border border-border-default rounded-lg">
  {/* Security Banner */}
  <div className="flex items-center gap-2 px-4 py-2 bg-feedback-success/10 border-b border-feedback-success/20">
    <ShieldCheckIcon className="w-4 h-4 text-feedback-success" />
    <span className="text-xs text-feedback-success font-medium">
      HIPAA-Compliant Secure Message
    </span>
    <span className="text-xs text-text-muted">
      · Encrypted end-to-end
    </span>
  </div>
  
  {/* Message Header */}
  <div className="p-4 border-b border-border-default">
    <div className="flex items-center justify-between">
      <div>
        <h3 className="font-semibold text-text-default">{message.subject}</h3>
        <div className="flex items-center gap-2 mt-1 text-sm text-text-muted">
          <span>From: {message.sender.name}</span>
          <span>·</span>
          <span>{formatDateTime(message.timestamp)}</span>
        </div>
      </div>
      <Badge variant={message.priority === 'urgent' ? 'error' : 'secondary'}>
        {message.priority}
      </Badge>
    </div>
    
    {/* Patient Reference */}
    {message.patient && (
      <div className="mt-3 p-2 bg-surface-muted rounded flex items-center gap-2">
        <UserIcon className="w-4 h-4 text-text-muted" />
        <span className="text-sm">
          Re: {message.patient.name} (MRN: {message.patient.mrn})
        </span>
      </div>
    )}
  </div>
  
  {/* Message Body */}
  <div className="p-4">
    <div className="prose prose-sm text-text-default">
      {message.body}
    </div>
    
    {/* Attachments */}
    {message.attachments?.length > 0 && (
      <div className="mt-4 pt-4 border-t border-border-muted">
        <h4 className="text-sm font-medium text-text-muted mb-2">
          Attachments ({message.attachments.length})
        </h4>
        <div className="flex flex-wrap gap-2">
          {message.attachments.map(attachment => (
            <a
              key={attachment.id}
              href={attachment.secureUrl}
              className="flex items-center gap-2 px-3 py-2 bg-surface-muted 
                         rounded-lg hover:bg-interactive-hover transition-colors"
            >
              <FileIcon className="w-4 h-4 text-text-muted" />
              <span className="text-sm text-text-default">{attachment.name}</span>
              <span className="text-xs text-text-subtle">({attachment.size})</span>
            </a>
          ))}
        </div>
      </div>
    )}
  </div>
  
  {/* Reply Section */}
  <div className="p-4 border-t border-border-default bg-surface-muted/30">
    <textarea
      value={reply}
      onChange={(e) => setReply(e.target.value)}
      placeholder="Type your secure reply..."
      className="w-full p-3 rounded-lg border border-border-default bg-surface-default
                 text-sm resize-none focus:ring-2 focus:ring-ring-focus"
      rows={3}
    />
    <div className="flex items-center justify-between mt-3">
      <Button variant="ghost" size="sm">
        <PaperclipIcon className="w-4 h-4 mr-1" />
        Attach File
      </Button>
      <Button variant="primary" disabled={!reply.trim()}>
        <SendIcon className="w-4 h-4 mr-1" />
        Send Secure Message
      </Button>
    </div>
  </div>
</div>
```


# ============================================================================
# CATEGORIA 7: FINTECH (PCI-DSS) PATTERNS
# ============================================================================

## FINTECH-1: ACCOUNT BALANCE CARD

### Struttura JSX:

```jsx
// ACCOUNT BALANCE CARD (PCI-DSS Compliant)
<div className="bg-gradient-to-br from-surface-brand to-surface-brand-emphasis 
                rounded-xl p-6 text-white">
  {/* Account Type & Mask Toggle */}
  <div className="flex items-center justify-between mb-4">
    <span className="text-sm opacity-80">{account.type}</span>
    <button 
      onClick={toggleMask}
      className="p-1 rounded hover:bg-white/10"
      aria-label={isMasked ? "Show balance" : "Hide balance"}
    >
      {isMasked ? <EyeOffIcon className="w-4 h-4" /> : <EyeIcon className="w-4 h-4" />}
    </button>
  </div>
  
  {/* Balance */}
  <div className="mb-6">
    <span className="text-sm opacity-70">Available Balance</span>
    <div className="text-3xl font-bold mt-1">
      {isMasked ? '••••••' : formatCurrency(account.balance, account.currency)}
    </div>
    {account.pendingBalance > 0 && (
      <span className="text-sm opacity-70">
        +{formatCurrency(account.pendingBalance)} pending
      </span>
    )}
  </div>
  
  {/* Account Details (masked) */}
  <div className="flex items-center justify-between">
    <div>
      <span className="text-xs opacity-70 block">Account Number</span>
      <span className="font-mono">•••• {account.last4}</span>
    </div>
    <div className="text-right">
      <span className="text-xs opacity-70 block">Routing</span>
      <span className="font-mono">{account.routingNumber}</span>
    </div>
  </div>
  
  {/* Quick Actions */}
  <div className="flex gap-2 mt-6 pt-4 border-t border-white/20">
    <Button variant="secondary" size="sm" className="flex-1 bg-white/10 hover:bg-white/20 text-white border-0">
      <SendIcon className="w-4 h-4 mr-1" /> Transfer
    </Button>
    <Button variant="secondary" size="sm" className="flex-1 bg-white/10 hover:bg-white/20 text-white border-0">
      <DownloadIcon className="w-4 h-4 mr-1" /> Download
    </Button>
  </div>
</div>
```

---

## FINTECH-2: TRANSACTION LIST

### Struttura JSX:

```jsx
// TRANSACTION LIST (PCI-DSS Compliant)
<div className="bg-surface-default border border-border-default rounded-lg">
  {/* Header */}
  <div className="flex items-center justify-between p-4 border-b border-border-default">
    <h3 className="font-semibold text-text-default">Recent Transactions</h3>
    <div className="flex items-center gap-2">
      <SearchInput
        value={search}
        onChange={setSearch}
        placeholder="Search transactions..."
        size="sm"
      />
      <Select
        value={filter}
        onChange={setFilter}
        options={[
          { value: 'all', label: 'All' },
          { value: 'credit', label: 'Credits' },
          { value: 'debit', label: 'Debits' },
          { value: 'pending', label: 'Pending' }
        ]}
        size="sm"
      />
    </div>
  </div>
  
  {/* Transactions */}
  <div className="divide-y divide-border-default">
    {transactions.map(tx => (
      <div 
        key={tx.id}
        className="flex items-center gap-4 p-4 hover:bg-surface-muted/30 
                   cursor-pointer transition-colors"
        onClick={() => openTransactionDetail(tx)}
      >
        {/* Icon */}
        <div className={cn(
          "w-10 h-10 rounded-full flex items-center justify-center",
          tx.type === 'credit' ? "bg-feedback-success/10" : "bg-surface-muted"
        )}>
          {tx.category === 'transfer' && <ArrowLeftRightIcon className="w-5 h-5" />}
          {tx.category === 'payment' && <CreditCardIcon className="w-5 h-5" />}
          {tx.category === 'deposit' && <ArrowDownIcon className="w-5 h-5 text-feedback-success" />}
          {tx.category === 'withdrawal' && <ArrowUpIcon className="w-5 h-5" />}
        </div>
        
        {/* Details */}
        <div className="flex-1 min-w-0">
          <div className="flex items-center gap-2">
            <span className="font-medium text-text-default truncate">
              {tx.description}
            </span>
            {tx.status === 'pending' && (
              <Badge variant="warning" size="sm">Pending</Badge>
            )}
          </div>
          <span className="text-sm text-text-muted">
            {formatDate(tx.date)} • {tx.category}
          </span>
        </div>
        
        {/* Amount */}
        <div className="text-right">
          <span className={cn(
            "font-semibold",
            tx.type === 'credit' ? "text-feedback-success" : "text-text-default"
          )}>
            {tx.type === 'credit' ? '+' : '-'}
            {formatCurrency(tx.amount, tx.currency)}
          </span>
          {tx.foreignAmount && (
            <span className="text-xs text-text-muted block">
              {formatCurrency(tx.foreignAmount, tx.foreignCurrency)}
            </span>
          )}
        </div>
        
        <ChevronRightIcon className="w-4 h-4 text-text-muted shrink-0" />
      </div>
    ))}
  </div>
  
  {/* Load More */}
  {hasMore && (
    <div className="p-4 text-center border-t border-border-default">
      <Button variant="ghost" onClick={loadMore}>
        Load More Transactions
      </Button>
    </div>
  )}
</div>
```

---

## FINTECH-3: PAYMENT FORM (PCI-DSS)

### Struttura JSX:

```jsx
// PAYMENT FORM (PCI-DSS Compliant - Card data handled by Stripe/similar)
<div className="max-w-md mx-auto">
  {/* Security Badge */}
  <div className="flex items-center justify-center gap-2 mb-6">
    <LockIcon className="w-4 h-4 text-feedback-success" />
    <span className="text-sm text-text-muted">256-bit SSL Encrypted</span>
  </div>
  
  <form onSubmit={handleSubmit} className="space-y-4">
    {/* Amount */}
    <div>
      <label className="block text-sm font-medium text-text-default mb-1.5">
        Amount
      </label>
      <div className="relative">
        <span className="absolute left-3 top-1/2 -translate-y-1/2 text-text-muted">
          $
        </span>
        <input
          type="text"
          inputMode="decimal"
          value={amount}
          onChange={handleAmountChange}
          className="w-full h-12 pl-8 pr-4 rounded-lg border border-border-default
                     text-xl font-semibold focus:ring-2 focus:ring-ring-focus"
          placeholder="0.00"
        />
      </div>
    </div>
    
    {/* Card Element (Stripe Elements / Similar) */}
    <div>
      <label className="block text-sm font-medium text-text-default mb-1.5">
        Card Information
      </label>
      <div className="p-4 rounded-lg border border-border-default bg-surface-default">
        {/* This would be a Stripe CardElement or similar */}
        <div id="card-element" className="min-h-[24px]">
          {/* Stripe injects card input here */}
        </div>
      </div>
      {cardError && (
        <p className="text-sm text-feedback-error mt-1.5">{cardError}</p>
      )}
    </div>
    
    {/* Billing Name */}
    <Input
      label="Name on Card"
      value={billingName}
      onChange={(e) => setBillingName(e.target.value)}
      required
    />
    
    {/* Billing Address */}
    <div className="grid grid-cols-2 gap-4">
      <Input
        label="ZIP / Postal Code"
        value={postalCode}
        onChange={(e) => setPostalCode(e.target.value)}
        required
      />
      <Select
        label="Country"
        value={country}
        onChange={setCountry}
        options={countries}
        required
      />
    </div>
    
    {/* Save Card Option */}
    <Checkbox
      checked={saveCard}
      onChange={(e) => setSaveCard(e.target.checked)}
      label="Save card for future payments"
      description="Your card will be securely stored"
    />
    
    {/* Submit */}
    <Button 
      type="submit" 
      variant="primary" 
      size="lg" 
      fullWidth
      loading={isProcessing}
      disabled={!isValid || isProcessing}
    >
      {isProcessing ? 'Processing...' : `Pay ${formatCurrency(amount)}`}
    </Button>
    
    {/* Trust Badges */}
    <div className="flex items-center justify-center gap-4 pt-4">
      <img src="/badges/pci-dss.svg" alt="PCI DSS Compliant" className="h-8" />
      <img src="/badges/stripe.svg" alt="Powered by Stripe" className="h-6" />
    </div>
  </form>
</div>
```

---

## FINTECH-4: KYC VERIFICATION STEP

### Struttura JSX:

```jsx
// KYC VERIFICATION STEP
<div className="max-w-lg mx-auto">
  {/* Progress */}
  <div className="mb-8">
    <div className="flex items-center justify-between text-sm mb-2">
      <span className="text-text-muted">Identity Verification</span>
      <span className="font-medium text-text-default">Step {currentStep} of 3</span>
    </div>
    <Progress value={(currentStep / 3) * 100} />
  </div>
  
  {/* Step Content */}
  {currentStep === 1 && (
    <div className="space-y-6">
      <div className="text-center">
        <div className="w-16 h-16 mx-auto mb-4 rounded-full bg-surface-brand/10 
                        flex items-center justify-center">
          <UserCheckIcon className="w-8 h-8 text-text-brand" />
        </div>
        <h2 className="text-xl font-semibold text-text-default">
          Verify Your Identity
        </h2>
        <p className="text-text-muted mt-2">
          We need to verify your identity to comply with financial regulations.
        </p>
      </div>
      
      {/* Document Type Selection */}
      <div className="space-y-3">
        <label className="text-sm font-medium text-text-default">
          Select Document Type
        </label>
        {documentTypes.map(doc => (
          <label
            key={doc.id}
            className={cn(
              "flex items-center gap-4 p-4 rounded-lg border cursor-pointer",
              "transition-colors",
              selectedDoc === doc.id 
                ? "border-surface-brand bg-surface-brand/5" 
                : "border-border-default hover:bg-surface-muted"
            )}
          >
            <Radio
              checked={selectedDoc === doc.id}
              onChange={() => setSelectedDoc(doc.id)}
            />
            <doc.icon className="w-6 h-6 text-text-muted" />
            <div>
              <span className="font-medium text-text-default">{doc.label}</span>
              <span className="text-sm text-text-muted block">{doc.description}</span>
            </div>
          </label>
        ))}
      </div>
    </div>
  )}
  
  {currentStep === 2 && (
    <div className="space-y-6">
      <h2 className="text-xl font-semibold text-text-default text-center">
        Upload Your {documentTypes.find(d => d.id === selectedDoc)?.label}
      </h2>
      
      {/* Upload Areas */}
      <div className="grid grid-cols-2 gap-4">
        <FileUploadZone
          label="Front Side"
          accept="image/*"
          value={frontImage}
          onChange={setFrontImage}
          preview
        />
        <FileUploadZone
          label="Back Side"
          accept="image/*"
          value={backImage}
          onChange={setBackImage}
          preview
        />
      </div>
      
      {/* Guidelines */}
      <Alert variant="info">
        <ul className="text-sm space-y-1">
          <li>• Make sure the document is clearly visible</li>
          <li>• All corners of the document should be visible</li>
          <li>• Avoid glare and shadows</li>
        </ul>
      </Alert>
    </div>
  )}
  
  {currentStep === 3 && (
    <div className="space-y-6">
      <h2 className="text-xl font-semibold text-text-default text-center">
        Take a Selfie
      </h2>
      <p className="text-text-muted text-center">
        We'll compare this with your document photo
      </p>
      
      {/* Camera Preview */}
      <div className="aspect-square max-w-sm mx-auto rounded-full overflow-hidden 
                      border-4 border-dashed border-border-default">
        {selfie ? (
          <img src={selfie} alt="Selfie" className="w-full h-full object-cover" />
        ) : (
          <div className="w-full h-full bg-surface-muted flex flex-col items-center justify-center">
            <CameraIcon className="w-12 h-12 text-text-muted mb-2" />
            <span className="text-sm text-text-muted">Camera preview</span>
          </div>
        )}
      </div>
      
      <Button 
        variant="primary" 
        size="lg" 
        fullWidth
        onClick={captureSelfie}
      >
        <CameraIcon className="w-5 h-5 mr-2" />
        {selfie ? 'Retake Photo' : 'Take Photo'}
      </Button>
    </div>
  )}
  
  {/* Navigation */}
  <div className="flex gap-4 mt-8">
    {currentStep > 1 && (
      <Button variant="outline" onClick={goBack} fullWidth>
        Back
      </Button>
    )}
    <Button 
      variant="primary" 
      onClick={goNext}
      disabled={!canProceed}
      fullWidth
    >
      {currentStep === 3 ? 'Submit Verification' : 'Continue'}
    </Button>
  </div>
</div>
```

# ============================================================================
# CATEGORIA 8: IOT PATTERNS
# ============================================================================

## IOT-1: DEVICE CARD

### Struttura JSX:

```jsx
// IOT DEVICE CARD
<div className={cn(
  "bg-surface-default border rounded-lg p-4",
  device.status === 'online' ? "border-border-default" : "border-feedback-warning"
)}>
  {/* Header */}
  <div className="flex items-start justify-between mb-3">
    <div className="flex items-center gap-3">
      <div className={cn(
        "w-10 h-10 rounded-lg flex items-center justify-center",
        device.status === 'online' ? "bg-feedback-success/10" : "bg-feedback-warning/10"
      )}>
        <DeviceIcon 
          type={device.type} 
          className={cn(
            "w-5 h-5",
            device.status === 'online' ? "text-feedback-success" : "text-feedback-warning"
          )}
        />
      </div>
      <div>
        <h4 className="font-medium text-text-default">{device.name}</h4>
        <div className="flex items-center gap-1.5 text-xs text-text-muted">
          <span className={cn(
            "w-1.5 h-1.5 rounded-full",
            device.status === 'online' ? "bg-feedback-success" : "bg-feedback-warning"
          )} />
          {device.status === 'online' ? 'Online' : 'Offline'}
          <span>•</span>
          <span>{device.location}</span>
        </div>
      </div>
    </div>
    
    <DropdownMenu>
      <DropdownMenu.Trigger asChild>
        <IconButton variant="ghost" size="sm" icon={MoreVerticalIcon} />
      </DropdownMenu.Trigger>
      <DropdownMenu.Content>
        <DropdownMenu.Item>Configure</DropdownMenu.Item>
        <DropdownMenu.Item>View Details</DropdownMenu.Item>
        <DropdownMenu.Separator />
        <DropdownMenu.Item>Restart Device</DropdownMenu.Item>
        <DropdownMenu.Item destructive>Remove</DropdownMenu.Item>
      </DropdownMenu.Content>
    </DropdownMenu>
  </div>
  
  {/* Current Reading */}
  <div className="bg-surface-muted rounded-lg p-3 mb-3">
    <div className="flex items-center justify-between">
      <span className="text-sm text-text-muted">{device.primaryMetric.label}</span>
      <span className="text-2xl font-bold text-text-default">
        {device.primaryMetric.value}
        <span className="text-sm font-normal text-text-muted ml-1">
          {device.primaryMetric.unit}
        </span>
      </span>
    </div>
    
    {/* Mini Chart */}
    <div className="h-12 mt-2">
      <Sparkline data={device.recentReadings} />
    </div>
  </div>
  
  {/* Secondary Metrics */}
  <div className="grid grid-cols-2 gap-2 text-sm">
    {device.secondaryMetrics.map(metric => (
      <div key={metric.id} className="flex items-center justify-between">
        <span className="text-text-muted">{metric.label}</span>
        <span className="font-medium text-text-default">
          {metric.value}{metric.unit}
        </span>
      </div>
    ))}
  </div>
  
  {/* Quick Toggle (if applicable) */}
  {device.hasToggle && (
    <div className="flex items-center justify-between mt-4 pt-4 border-t border-border-muted">
      <span className="text-sm text-text-default">{device.toggleLabel}</span>
      <Switch 
        checked={device.toggleState}
        onChange={() => onToggle(device.id)}
      />
    </div>
  )}
</div>
```

---

## IOT-2: SENSOR DASHBOARD

### Struttura JSX:

```jsx
// SENSOR DASHBOARD
<div className="space-y-6">
  {/* Summary Cards */}
  <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
    <SensorSummaryCard
      label="Total Devices"
      value={stats.totalDevices}
      icon={CpuIcon}
    />
    <SensorSummaryCard
      label="Online"
      value={stats.onlineDevices}
      icon={WifiIcon}
      variant="success"
    />
    <SensorSummaryCard
      label="Alerts"
      value={stats.activeAlerts}
      icon={AlertTriangleIcon}
      variant={stats.activeAlerts > 0 ? "error" : "default"}
    />
    <SensorSummaryCard
      label="Data Points (24h)"
      value={formatNumber(stats.dataPoints)}
      icon={DatabaseIcon}
    />
  </div>
  
  {/* Real-time Monitoring */}
  <div className="bg-surface-default border border-border-default rounded-lg">
    <div className="flex items-center justify-between p-4 border-b border-border-default">
      <h3 className="font-semibold text-text-default">Real-time Monitoring</h3>
      <div className="flex items-center gap-2">
        <Badge variant="success" size="sm">
          <span className="w-1.5 h-1.5 rounded-full bg-current animate-pulse mr-1" />
          Live
        </Badge>
        <Select
          value={selectedSensor}
          onChange={setSelectedSensor}
          options={sensors.map(s => ({ value: s.id, label: s.name }))}
          size="sm"
        />
      </div>
    </div>
    
    {/* Real-time Chart */}
    <div className="p-4">
      <div className="h-64">
        <RealTimeChart
          data={realtimeData}
          xKey="timestamp"
          yKey="value"
          color="var(--color-brand)"
        />
      </div>
      
      {/* Current Value */}
      <div className="flex items-center justify-center gap-8 mt-4 pt-4 border-t border-border-muted">
        <div className="text-center">
          <span className="text-sm text-text-muted">Current</span>
          <div className="text-3xl font-bold text-text-default">
            {currentReading.value}
            <span className="text-lg text-text-muted ml-1">{currentReading.unit}</span>
          </div>
        </div>
        <div className="text-center">
          <span className="text-sm text-text-muted">Min (24h)</span>
          <div className="text-xl font-semibold text-text-muted">
            {stats.min}{currentReading.unit}
          </div>
        </div>
        <div className="text-center">
          <span className="text-sm text-text-muted">Max (24h)</span>
          <div className="text-xl font-semibold text-text-muted">
            {stats.max}{currentReading.unit}
          </div>
        </div>
        <div className="text-center">
          <span className="text-sm text-text-muted">Avg (24h)</span>
          <div className="text-xl font-semibold text-text-muted">
            {stats.avg}{currentReading.unit}
          </div>
        </div>
      </div>
    </div>
  </div>
  
  {/* Alert Rules */}
  <div className="bg-surface-default border border-border-default rounded-lg">
    <div className="flex items-center justify-between p-4 border-b border-border-default">
      <h3 className="font-semibold text-text-default">Alert Rules</h3>
      <Button variant="outline" size="sm">
        <PlusIcon className="w-4 h-4 mr-1" />
        Add Rule
      </Button>
    </div>
    
    <div className="divide-y divide-border-default">
      {alertRules.map(rule => (
        <div key={rule.id} className="flex items-center gap-4 p-4">
          <Switch 
            checked={rule.enabled}
            onChange={() => toggleRule(rule.id)}
          />
          <div className="flex-1">
            <span className="font-medium text-text-default">{rule.name}</span>
            <span className="text-sm text-text-muted block">
              When {rule.metric} {rule.operator} {rule.threshold}{rule.unit}
            </span>
          </div>
          <Badge variant={rule.severity === 'critical' ? 'error' : 'warning'}>
            {rule.severity}
          </Badge>
          <IconButton variant="ghost" size="sm" icon={EditIcon} />
        </div>
      ))}
    </div>
  </div>
</div>
```

---

## IOT-3: DEVICE CONTROL PANEL

### Struttura JSX:

```jsx
// DEVICE CONTROL PANEL
<div className="bg-surface-default border border-border-default rounded-lg p-6">
  <div className="flex items-center justify-between mb-6">
    <div>
      <h3 className="text-lg font-semibold text-text-default">{device.name}</h3>
      <span className="text-sm text-text-muted">{device.model}</span>
    </div>
    <Badge variant={device.status === 'online' ? 'success' : 'warning'}>
      {device.status}
    </Badge>
  </div>
  
  {/* Control Sections */}
  <div className="space-y-6">
    {/* Power */}
    <div className="flex items-center justify-between py-4 border-b border-border-muted">
      <div className="flex items-center gap-3">
        <PowerIcon className="w-5 h-5 text-text-muted" />
        <span className="font-medium text-text-default">Power</span>
      </div>
      <Switch 
        checked={device.isPowered}
        onChange={() => togglePower(device.id)}
      />
    </div>
    
    {/* Temperature/Setpoint (if thermostat) */}
    {device.type === 'thermostat' && (
      <div className="py-4 border-b border-border-muted">
        <div className="flex items-center justify-between mb-4">
          <span className="font-medium text-text-default">Temperature</span>
          <span className="text-2xl font-bold text-text-default">
            {device.setpoint}°{device.unit}
          </span>
        </div>
        <Slider
          value={device.setpoint}
          onChange={(value) => setTemperature(device.id, value)}
          min={60}
          max={85}
          step={1}
        />
        <div className="flex justify-between text-xs text-text-muted mt-2">
          <span>60°</span>
          <span>85°</span>
        </div>
      </div>
    )}
    
    {/* Brightness (if light) */}
    {device.type === 'light' && (
      <div className="py-4 border-b border-border-muted">
        <div className="flex items-center justify-between mb-4">
          <span className="font-medium text-text-default">Brightness</span>
          <span className="text-text-default">{device.brightness}%</span>
        </div>
        <Slider
          value={device.brightness}
          onChange={(value) => setBrightness(device.id, value)}
          min={0}
          max={100}
        />
      </div>
    )}
    
    {/* Mode Selection */}
    {device.modes && (
      <div className="py-4 border-b border-border-muted">
        <span className="font-medium text-text-default block mb-3">Mode</span>
        <div className="grid grid-cols-4 gap-2">
          {device.modes.map(mode => (
            <button
              key={mode.id}
              onClick={() => setMode(device.id, mode.id)}
              className={cn(
                "flex flex-col items-center gap-1 p-3 rounded-lg transition-colors",
                device.currentMode === mode.id
                  ? "bg-surface-brand text-white"
                  : "bg-surface-muted text-text-muted hover:bg-interactive-hover"
              )}
            >
              <mode.icon className="w-5 h-5" />
              <span className="text-xs">{mode.label}</span>
            </button>
          ))}
        </div>
      </div>
    )}
    
    {/* Schedule */}
    <div className="py-4">
      <div className="flex items-center justify-between mb-3">
        <span className="font-medium text-text-default">Schedule</span>
        <Button variant="ghost" size="sm">Edit</Button>
      </div>
      <div className="space-y-2">
        {device.schedule.map(entry => (
          <div 
            key={entry.id}
            className="flex items-center justify-between p-2 bg-surface-muted rounded"
          >
            <span className="text-sm text-text-default">{entry.time}</span>
            <span className="text-sm text-text-muted">{entry.action}</span>
          </div>
        ))}
      </div>
    </div>
  </div>
</div>
```


# ============================================================================
# SEZIONE 3: CONTENT PATTERNS
# ============================================================================

## 3.1 CARD

### Descrizione:
Contenitore per raggruppare informazioni correlate con bordo, sfondo e ombra opzionale.

### Varianti:
| Variante | Descrizione |
|----------|-------------|
| `default` | Card standard con bordo |
| `elevated` | Con ombra (shadow-md) |
| `outlined` | Solo bordo, no sfondo |
| `interactive` | Hover effect, cliccabile |
| `compact` | Padding ridotto |

### Struttura JSX:

```jsx
// CARD - Default
<div className={cn(
  "bg-surface-default rounded-lg border border-border-default",
  "overflow-hidden",
  variant === "elevated" && "shadow-md border-0",
  variant === "interactive" && "cursor-pointer hover:shadow-lg transition-shadow"
)}>
  {/* Card Header (opzionale) */}
  {header && (
    <div className="px-6 py-4 border-b border-border-muted">
      <h3 className="text-lg font-semibold text-text-default">{title}</h3>
      {description && (
        <p className="text-sm text-text-muted mt-1">{description}</p>
      )}
    </div>
  )}
  
  {/* Card Media (opzionale) */}
  {media && (
    <div className="aspect-video bg-surface-muted">
      <img src={media} alt="" className="w-full h-full object-cover" />
    </div>
  )}
  
  {/* Card Body */}
  <div className="p-6">
    {children}
  </div>
  
  {/* Card Footer (opzionale) */}
  {footer && (
    <div className="px-6 py-4 border-t border-border-muted bg-surface-subtle">
      {footer}
    </div>
  )}
</div>
```

---

## 3.2 LIST

### Descrizione:
Elenco verticale di elementi correlati.

### Varianti:
| Variante | Descrizione |
|----------|-------------|
| `simple` | Testo semplice |
| `with-icon` | Icon + testo |
| `with-avatar` | Avatar + contenuto |
| `interactive` | Elementi cliccabili |
| `selectable` | Con checkbox/radio |

### Struttura JSX:

```jsx
// LIST - Interactive with icons
<ul className="divide-y divide-border-default" role="list">
  {items.map(item => (
    <li key={item.id}>
      <a
        href={item.href}
        className={cn(
          "flex items-center gap-4 px-4 py-3",
          "hover:bg-interactive-hover transition-colors",
          item.isActive && "bg-interactive-selected"
        )}
      >
        {/* Leading Element */}
        {item.icon && (
          <span className="shrink-0 w-10 h-10 rounded-full bg-surface-muted
                          flex items-center justify-center">
            <item.icon className="w-5 h-5 text-text-muted" />
          </span>
        )}
        
        {/* Content */}
        <div className="flex-1 min-w-0">
          <p className="text-sm font-medium text-text-default truncate">
            {item.title}
          </p>
          {item.subtitle && (
            <p className="text-sm text-text-muted truncate">
              {item.subtitle}
            </p>
          )}
        </div>
        
        {/* Trailing Element */}
        {item.badge && (
          <Badge variant="secondary">{item.badge}</Badge>
        )}
        <ChevronRightIcon className="w-5 h-5 text-text-subtle shrink-0" />
      </a>
    </li>
  ))}
</ul>
```

---

## 3.3 AVATAR

### Descrizione:
Rappresentazione visiva di un utente o entità.

### Varianti:
| Size | Dimensioni |
|------|------------|
| `xs` | 24px (w-6 h-6) |
| `sm` | 32px (w-8 h-8) |
| `md` | 40px (w-10 h-10) |
| `lg` | 48px (w-12 h-12) |
| `xl` | 64px (w-16 h-16) |

### Struttura JSX:

```jsx
// AVATAR
<div className={cn(
  "relative inline-flex items-center justify-center",
  "rounded-full bg-surface-muted overflow-hidden",
  {
    "w-6 h-6 text-xs": size === "xs",
    "w-8 h-8 text-sm": size === "sm",
    "w-10 h-10 text-base": size === "md",
    "w-12 h-12 text-lg": size === "lg",
    "w-16 h-16 text-xl": size === "xl"
  }
)}>
  {src ? (
    <img 
      src={src} 
      alt={name} 
      className="w-full h-full object-cover"
    />
  ) : (
    <span className="font-medium text-text-muted">
      {getInitials(name)}
    </span>
  )}
  
  {/* Status Indicator */}
  {status && (
    <span className={cn(
      "absolute bottom-0 right-0 rounded-full ring-2 ring-surface-default",
      {
        "w-2 h-2": size === "xs" || size === "sm",
        "w-3 h-3": size === "md" || size === "lg",
        "w-4 h-4": size === "xl"
      },
      {
        "bg-feedback-success": status === "online",
        "bg-feedback-warning": status === "away",
        "bg-text-subtle": status === "offline"
      }
    )} />
  )}
</div>

// AVATAR GROUP
<div className="flex -space-x-2">
  {users.slice(0, maxVisible).map(user => (
    <Avatar key={user.id} {...user} className="ring-2 ring-surface-default" />
  ))}
  {users.length > maxVisible && (
    <div className="w-10 h-10 rounded-full bg-surface-muted 
                    flex items-center justify-center
                    text-sm font-medium text-text-muted
                    ring-2 ring-surface-default">
      +{users.length - maxVisible}
    </div>
  )}
</div>
```

---

## 3.4 BADGE

### Descrizione:
Indicatore compatto per stati, conteggi o etichette.

### Varianti:
| Variante | Colore | Uso |
|----------|--------|-----|
| `default` | Grigio | Neutro |
| `primary` | Brand | Evidenziato |
| `success` | Verde | Positivo |
| `warning` | Giallo | Attenzione |
| `error` | Rosso | Errore/critico |
| `info` | Blu | Informativo |

### Struttura JSX:

```jsx
// BADGE
<span className={cn(
  "inline-flex items-center gap-1 px-2 py-0.5 rounded-full",
  "text-xs font-medium",
  {
    "bg-surface-muted text-text-muted": variant === "default",
    "bg-surface-brand text-text-inverse": variant === "primary",
    "bg-feedback-success-subtle text-feedback-success": variant === "success",
    "bg-feedback-warning-subtle text-feedback-warning": variant === "warning",
    "bg-feedback-error-subtle text-feedback-error": variant === "error",
    "bg-feedback-info-subtle text-feedback-info": variant === "info"
  }
)}>
  {dot && <span className="w-1.5 h-1.5 rounded-full bg-current" />}
  {children}
</span>
```

---

## 3.5 EMPTY STATE

### Descrizione:
Placeholder quando non ci sono dati da mostrare.

### Struttura JSX:

```jsx
// EMPTY STATE
<div className="flex flex-col items-center justify-center py-12 px-4 text-center">
  {/* Illustration */}
  <div className="w-24 h-24 mb-6 text-text-subtle">
    {icon || <EmptyIllustration />}
  </div>
  
  {/* Title */}
  <h3 className="text-lg font-semibold text-text-default mb-2">
    {title}
  </h3>
  
  {/* Description */}
  <p className="text-sm text-text-muted max-w-sm mb-6">
    {description}
  </p>
  
  {/* Action */}
  {action && (
    <Button variant="primary" onClick={action.onClick}>
      {action.icon && <action.icon className="w-4 h-4 mr-2" />}
      {action.label}
    </Button>
  )}
</div>
```

---

## 3.6 SKELETON / LOADING STATE

### Descrizione:
Placeholder animato mentre il contenuto viene caricato.

### Struttura JSX:

```jsx
// SKELETON BASE
<div className="animate-pulse bg-surface-muted rounded" />

// SKELETON VARIANTS
// Text line
<div className="h-4 bg-surface-muted rounded w-3/4 animate-pulse" />

// Avatar
<div className="w-10 h-10 bg-surface-muted rounded-full animate-pulse" />

// Card skeleton
<div className="border border-border-default rounded-lg p-4 space-y-4">
  <div className="flex items-center gap-4">
    <div className="w-12 h-12 bg-surface-muted rounded-full animate-pulse" />
    <div className="flex-1 space-y-2">
      <div className="h-4 bg-surface-muted rounded w-1/4 animate-pulse" />
      <div className="h-3 bg-surface-muted rounded w-1/2 animate-pulse" />
    </div>
  </div>
  <div className="space-y-2">
    <div className="h-4 bg-surface-muted rounded animate-pulse" />
    <div className="h-4 bg-surface-muted rounded w-5/6 animate-pulse" />
    <div className="h-4 bg-surface-muted rounded w-4/6 animate-pulse" />
  </div>
</div>
```

# ============================================================================
# SEZIONE 4: INPUT PATTERNS
# ============================================================================

## 4.1 TEXT INPUT

### Descrizione:
Campo per inserimento testo singola riga.

### Varianti & Stati:
| Stato | Descrizione |
|-------|-------------|
| `default` | Stato normale |
| `focus` | In focus |
| `error` | Con errore di validazione |
| `disabled` | Non modificabile |
| `readonly` | Solo lettura |

### Struttura JSX:

```jsx
// TEXT INPUT
<div className="space-y-1.5">
  {/* Label */}
  <label 
    htmlFor={id}
    className="block text-sm font-medium text-text-default"
  >
    {label}
    {required && <span className="text-feedback-error ml-1">*</span>}
  </label>
  
  {/* Input Container */}
  <div className="relative">
    {/* Leading Icon */}
    {leadingIcon && (
      <div className="absolute left-3 top-1/2 -translate-y-1/2 text-text-subtle">
        {leadingIcon}
      </div>
    )}
    
    {/* Input */}
    <input
      id={id}
      type={type}
      value={value}
      onChange={onChange}
      disabled={disabled}
      readOnly={readOnly}
      placeholder={placeholder}
      aria-invalid={!!error}
      aria-describedby={error ? `${id}-error` : helpText ? `${id}-help` : undefined}
      className={cn(
        "w-full px-3 py-2 rounded-md border bg-surface-default",
        "text-sm text-text-default placeholder:text-text-subtle",
        "transition-colors duration-fast",
        "focus:outline-none focus:ring-2 focus:ring-offset-0",
        leadingIcon && "pl-10",
        trailingIcon && "pr-10",
        {
          "border-border-default focus:border-border-brand focus:ring-border-brand/20":
            !error && !disabled,
          "border-feedback-error focus:border-feedback-error focus:ring-feedback-error/20":
            error,
          "bg-surface-subtle border-border-muted text-text-subtle cursor-not-allowed":
            disabled
        }
      )}
    />
    
    {/* Trailing Icon */}
    {trailingIcon && (
      <div className="absolute right-3 top-1/2 -translate-y-1/2 text-text-subtle">
        {trailingIcon}
      </div>
    )}
  </div>
  
  {/* Help Text or Error */}
  {(helpText || error) && (
    <p 
      id={error ? `${id}-error` : `${id}-help`}
      className={cn(
        "text-sm",
        error ? "text-feedback-error" : "text-text-muted"
      )}
    >
      {error || helpText}
    </p>
  )}
</div>
```

---

## 4.2 SELECT / DROPDOWN

### Struttura JSX:

```jsx
// SELECT
<div className="space-y-1.5">
  <label className="block text-sm font-medium text-text-default">
    {label}
  </label>
  
  <div className="relative">
    <select
      value={value}
      onChange={onChange}
      className={cn(
        "w-full px-3 py-2 pr-10 rounded-md border border-border-default",
        "bg-surface-default text-sm text-text-default",
        "appearance-none cursor-pointer",
        "focus:outline-none focus:ring-2 focus:border-border-brand",
        "focus:ring-border-brand/20"
      )}
    >
      {placeholder && (
        <option value="" disabled>
          {placeholder}
        </option>
      )}
      {options.map(option => (
        <option key={option.value} value={option.value}>
          {option.label}
        </option>
      ))}
    </select>
    
    {/* Chevron Icon */}
    <ChevronDownIcon className="absolute right-3 top-1/2 -translate-y-1/2 
                                w-4 h-4 text-text-subtle pointer-events-none" />
  </div>
</div>
```

---

## 4.3 CHECKBOX

### Struttura JSX:

```jsx
// CHECKBOX
<label className="flex items-start gap-3 cursor-pointer group">
  <div className="relative flex items-center justify-center mt-0.5">
    <input
      type="checkbox"
      checked={checked}
      onChange={onChange}
      disabled={disabled}
      className="peer sr-only"
    />
    <div className={cn(
      "w-5 h-5 rounded border-2 transition-colors",
      "flex items-center justify-center",
      "peer-focus-visible:ring-2 peer-focus-visible:ring-offset-2",
      "peer-focus-visible:ring-border-brand",
      checked
        ? "bg-surface-brand border-border-brand"
        : "bg-surface-default border-border-emphasis",
      disabled && "opacity-50 cursor-not-allowed"
    )}>
      {checked && (
        <CheckIcon className="w-3 h-3 text-text-inverse" />
      )}
    </div>
  </div>
  
  <div>
    <span className="text-sm font-medium text-text-default">
      {label}
    </span>
    {description && (
      <p className="text-sm text-text-muted">{description}</p>
    )}
  </div>
</label>
```

---

## 4.4 RADIO GROUP

### Struttura JSX:

```jsx
// RADIO GROUP
<fieldset>
  <legend className="text-sm font-medium text-text-default mb-3">
    {legend}
  </legend>
  
  <div className="space-y-2">
    {options.map(option => (
      <label 
        key={option.value}
        className="flex items-start gap-3 cursor-pointer"
      >
        <div className="relative flex items-center justify-center mt-0.5">
          <input
            type="radio"
            name={name}
            value={option.value}
            checked={value === option.value}
            onChange={() => onChange(option.value)}
            className="peer sr-only"
          />
          <div className={cn(
            "w-5 h-5 rounded-full border-2 transition-colors",
            "flex items-center justify-center",
            "peer-focus-visible:ring-2 peer-focus-visible:ring-offset-2",
            "peer-focus-visible:ring-border-brand",
            value === option.value
              ? "border-border-brand"
              : "border-border-emphasis"
          )}>
            {value === option.value && (
              <div className="w-2.5 h-2.5 rounded-full bg-surface-brand" />
            )}
          </div>
        </div>
        
        <div>
          <span className="text-sm font-medium text-text-default">
            {option.label}
          </span>
          {option.description && (
            <p className="text-sm text-text-muted">{option.description}</p>
          )}
        </div>
      </label>
    ))}
  </div>
</fieldset>
```

---

## 4.5 TOGGLE / SWITCH

### Struttura JSX:

```jsx
// TOGGLE SWITCH
<label className="flex items-center gap-3 cursor-pointer">
  <button
    type="button"
    role="switch"
    aria-checked={checked}
    onClick={() => onChange(!checked)}
    disabled={disabled}
    className={cn(
      "relative inline-flex h-6 w-11 shrink-0 rounded-full",
      "transition-colors duration-normal ease-out",
      "focus-visible:outline-none focus-visible:ring-2",
      "focus-visible:ring-border-brand focus-visible:ring-offset-2",
      checked ? "bg-surface-brand" : "bg-surface-muted",
      disabled && "opacity-50 cursor-not-allowed"
    )}
  >
    <span
      className={cn(
        "pointer-events-none inline-block h-5 w-5 rounded-full",
        "bg-surface-default shadow-sm ring-0",
        "transition-transform duration-normal ease-out",
        checked ? "translate-x-5" : "translate-x-0.5",
        "mt-0.5"
      )}
    />
  </button>
  
  <span className="text-sm font-medium text-text-default">
    {label}
  </span>
</label>
```

---

## 4.6 TEXTAREA

### Struttura JSX:

```jsx
// TEXTAREA
<div className="space-y-1.5">
  <label className="block text-sm font-medium text-text-default">
    {label}
  </label>
  
  <textarea
    value={value}
    onChange={onChange}
    rows={rows || 4}
    placeholder={placeholder}
    className={cn(
      "w-full px-3 py-2 rounded-md border border-border-default",
      "bg-surface-default text-sm text-text-default",
      "placeholder:text-text-subtle resize-y min-h-[100px]",
      "focus:outline-none focus:ring-2 focus:border-border-brand",
      "focus:ring-border-brand/20"
    )}
  />
  
  {maxLength && (
    <p className="text-xs text-text-muted text-right">
      {value.length}/{maxLength}
    </p>
  )}
</div>
```

---

## 4.7 SEARCH INPUT

### Struttura JSX:

```jsx
// SEARCH INPUT
<div className="relative">
  <SearchIcon className="absolute left-3 top-1/2 -translate-y-1/2 
                         w-5 h-5 text-text-subtle" />
  <input
    type="search"
    value={value}
    onChange={onChange}
    placeholder={placeholder || "Search..."}
    className={cn(
      "w-full pl-10 pr-4 py-2 rounded-full",
      "border border-border-default bg-surface-subtle",
      "text-sm text-text-default placeholder:text-text-subtle",
      "focus:outline-none focus:ring-2 focus:bg-surface-default",
      "focus:border-border-brand focus:ring-border-brand/20"
    )}
  />
  
  {/* Clear button */}
  {value && (
    <button
      onClick={onClear}
      className="absolute right-3 top-1/2 -translate-y-1/2 
                 p-1 rounded-full hover:bg-surface-muted"
    >
      <XIcon className="w-4 h-4 text-text-muted" />
    </button>
  )}
</div>
```

---

## 4.8 DATE PICKER

### Struttura JSX:

```jsx
// DATE PICKER (simplified)
<div className="space-y-1.5">
  <label className="block text-sm font-medium text-text-default">
    {label}
  </label>
  
  <div className="relative">
    <input
      type="date"
      value={value}
      onChange={onChange}
      min={minDate}
      max={maxDate}
      className={cn(
        "w-full px-3 py-2 rounded-md border border-border-default",
        "bg-surface-default text-sm text-text-default",
        "focus:outline-none focus:ring-2 focus:border-border-brand",
        "focus:ring-border-brand/20"
      )}
    />
    <CalendarIcon className="absolute right-3 top-1/2 -translate-y-1/2 
                             w-5 h-5 text-text-subtle pointer-events-none" />
  </div>
</div>
```

---

## 4.9 FILE UPLOAD

### Struttura JSX:

```jsx
// FILE UPLOAD - Dropzone
<div
  onDragOver={handleDragOver}
  onDragLeave={handleDragLeave}
  onDrop={handleDrop}
  className={cn(
    "border-2 border-dashed rounded-lg p-8",
    "flex flex-col items-center justify-center text-center",
    "transition-colors cursor-pointer",
    isDragging
      ? "border-border-brand bg-surface-brand/5"
      : "border-border-default hover:border-border-emphasis"
  )}
>
  <input
    type="file"
    accept={accept}
    multiple={multiple}
    onChange={handleFileSelect}
    className="sr-only"
    id={id}
  />
  
  <UploadIcon className="w-10 h-10 text-text-subtle mb-4" />
  
  <label htmlFor={id} className="cursor-pointer">
    <span className="text-sm font-medium text-text-brand hover:underline">
      Click to upload
    </span>
    <span className="text-sm text-text-muted"> or drag and drop</span>
  </label>
  
  <p className="text-xs text-text-muted mt-2">
    {acceptLabel || "PNG, JPG, PDF up to 10MB"}
  </p>
</div>
```


# ============================================================================
# SEZIONE 5: FEEDBACK PATTERNS
# ============================================================================

## 5.1 ALERT / BANNER

### Descrizione:
Messaggio inline per comunicare informazioni, avvisi o errori.

### Varianti:
| Variante | Colore | Icona | Uso |
|----------|--------|-------|-----|
| `info` | Blu | InfoIcon | Informazioni neutrali |
| `success` | Verde | CheckCircleIcon | Operazione completata |
| `warning` | Giallo | AlertTriangleIcon | Attenzione richiesta |
| `error` | Rosso | XCircleIcon | Errore critico |

### Struttura JSX:

```jsx
// ALERT
<div
  role="alert"
  className={cn(
    "flex items-start gap-3 p-4 rounded-lg border",
    {
      "bg-feedback-info/10 border-feedback-info/20 text-feedback-info": 
        variant === "info",
      "bg-feedback-success/10 border-feedback-success/20 text-feedback-success": 
        variant === "success",
      "bg-feedback-warning/10 border-feedback-warning/20 text-feedback-warning": 
        variant === "warning",
      "bg-feedback-error/10 border-feedback-error/20 text-feedback-error": 
        variant === "error"
    }
  )}
>
  {/* Icon */}
  <VariantIcon className="w-5 h-5 shrink-0 mt-0.5" />
  
  {/* Content */}
  <div className="flex-1 min-w-0">
    {title && (
      <h4 className="font-semibold mb-1">{title}</h4>
    )}
    <p className="text-sm opacity-90">{children}</p>
    
    {/* Actions */}
    {actions && (
      <div className="flex gap-3 mt-3">
        {actions}
      </div>
    )}
  </div>
  
  {/* Dismiss Button */}
  {dismissible && (
    <button
      onClick={onDismiss}
      className="shrink-0 p-1 rounded hover:bg-current/10"
      aria-label="Dismiss"
    >
      <XIcon className="w-4 h-4" />
    </button>
  )}
</div>
```

---

## 5.2 TOAST / NOTIFICATION

### Descrizione:
Notifica temporanea non bloccante.

### Struttura JSX:

```jsx
// TOAST CONTAINER (position)
<div 
  className="fixed z-toast pointer-events-none"
  style={{
    top: position.includes('top') ? '1rem' : 'auto',
    bottom: position.includes('bottom') ? '1rem' : 'auto',
    left: position.includes('left') ? '1rem' : 'auto',
    right: position.includes('right') ? '1rem' : 'auto'
  }}
>
  <div className="flex flex-col gap-2">
    {toasts.map(toast => (
      <ToastItem key={toast.id} {...toast} />
    ))}
  </div>
</div>

// TOAST ITEM
<div
  role="status"
  aria-live="polite"
  className={cn(
    "pointer-events-auto max-w-sm w-full",
    "bg-surface-default border border-border-default rounded-lg shadow-lg",
    "p-4 flex items-start gap-3",
    "animate-slide-in"
  )}
>
  {/* Icon */}
  <div className={cn(
    "shrink-0 w-8 h-8 rounded-full flex items-center justify-center",
    {
      "bg-feedback-success/10 text-feedback-success": type === "success",
      "bg-feedback-error/10 text-feedback-error": type === "error",
      "bg-feedback-warning/10 text-feedback-warning": type === "warning",
      "bg-feedback-info/10 text-feedback-info": type === "info"
    }
  )}>
    <TypeIcon className="w-4 h-4" />
  </div>
  
  {/* Content */}
  <div className="flex-1 min-w-0">
    {title && (
      <p className="text-sm font-semibold text-text-default">{title}</p>
    )}
    <p className="text-sm text-text-muted">{message}</p>
    
    {/* Action */}
    {action && (
      <button 
        onClick={action.onClick}
        className="text-sm font-medium text-text-brand mt-2 hover:underline"
      >
        {action.label}
      </button>
    )}
  </div>
  
  {/* Close */}
  <button
    onClick={onClose}
    className="shrink-0 p-1 rounded hover:bg-interactive-hover"
  >
    <XIcon className="w-4 h-4 text-text-muted" />
  </button>
</div>
```

---

## 5.3 PROGRESS INDICATORS

### Struttura JSX:

```jsx
// PROGRESS BAR
<div className="space-y-2">
  {/* Label */}
  {label && (
    <div className="flex items-center justify-between text-sm">
      <span className="text-text-muted">{label}</span>
      {showPercentage && (
        <span className="font-medium text-text-default">{value}%</span>
      )}
    </div>
  )}
  
  {/* Bar */}
  <div className="h-2 bg-surface-muted rounded-full overflow-hidden">
    <div
      role="progressbar"
      aria-valuenow={value}
      aria-valuemin={0}
      aria-valuemax={100}
      className={cn(
        "h-full rounded-full transition-all duration-normal",
        variant === "default" && "bg-surface-brand",
        variant === "success" && "bg-feedback-success",
        variant === "warning" && "bg-feedback-warning",
        variant === "error" && "bg-feedback-error"
      )}
      style={{ width: `${value}%` }}
    />
  </div>
</div>

// SPINNER
<svg
  className={cn(
    "animate-spin",
    {
      "w-4 h-4": size === "sm",
      "w-6 h-6": size === "md",
      "w-8 h-8": size === "lg"
    },
    className
  )}
  viewBox="0 0 24 24"
  fill="none"
>
  <circle
    className="opacity-25"
    cx="12"
    cy="12"
    r="10"
    stroke="currentColor"
    strokeWidth="4"
  />
  <path
    className="opacity-75"
    fill="currentColor"
    d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z"
  />
</svg>
```

---

## 5.4 TOOLTIP

### Struttura JSX:

```jsx
// TOOLTIP (using Headless UI / Radix style)
<Tooltip.Provider>
  <Tooltip.Root>
    <Tooltip.Trigger asChild>
      {children}
    </Tooltip.Trigger>
    
    <Tooltip.Portal>
      <Tooltip.Content
        side={side}
        sideOffset={5}
        className={cn(
          "z-tooltip px-3 py-1.5 rounded-md",
          "bg-surface-inverse text-text-inverse",
          "text-xs font-medium",
          "animate-in fade-in-0 zoom-in-95"
        )}
      >
        {content}
        <Tooltip.Arrow className="fill-surface-inverse" />
      </Tooltip.Content>
    </Tooltip.Portal>
  </Tooltip.Root>
</Tooltip.Provider>
```

---

## 5.5 FORM VALIDATION

### Struttura JSX:

```jsx
// INLINE ERROR
<div className="flex items-center gap-1.5 mt-1.5">
  <AlertCircleIcon className="w-4 h-4 text-feedback-error shrink-0" />
  <span className="text-sm text-feedback-error">{error}</span>
</div>

// SUCCESS INDICATOR
<div className="flex items-center gap-1.5 mt-1.5">
  <CheckCircleIcon className="w-4 h-4 text-feedback-success shrink-0" />
  <span className="text-sm text-feedback-success">{message}</span>
</div>

// FORM VALIDATION SUMMARY
<div className="bg-feedback-error/10 border border-feedback-error/20 
                rounded-lg p-4 mb-6">
  <div className="flex items-start gap-3">
    <AlertCircleIcon className="w-5 h-5 text-feedback-error shrink-0 mt-0.5" />
    <div>
      <h4 className="font-semibold text-feedback-error mb-2">
        {errors.length} error{errors.length > 1 ? 's' : ''} found
      </h4>
      <ul className="list-disc list-inside space-y-1 text-sm text-feedback-error/80">
        {errors.map((error, index) => (
          <li key={index}>{error}</li>
        ))}
      </ul>
    </div>
  </div>
</div>
```

# ============================================================================
# SEZIONE 6: OVERLAY PATTERNS
# ============================================================================

## 6.1 MODAL / DIALOG

### Descrizione:
Finestra modale per azioni che richiedono attenzione.

### Varianti:
| Size | Max Width | Uso |
|------|-----------|-----|
| `sm` | 400px | Conferme semplici |
| `md` | 500px | Form brevi |
| `lg` | 640px | Form complessi |
| `xl` | 800px | Contenuti estesi |
| `full` | Full screen | Editor, wizard |

### Struttura JSX:

```jsx
// MODAL
<Dialog open={isOpen} onClose={onClose}>
  {/* Backdrop */}
  <div 
    className="fixed inset-0 z-modal-backdrop bg-black/50 backdrop-blur-sm"
    aria-hidden="true"
  />
  
  {/* Modal Container */}
  <div className="fixed inset-0 z-modal overflow-y-auto p-4">
    <div className="flex min-h-full items-center justify-center">
      <Dialog.Panel
        className={cn(
          "w-full bg-surface-default rounded-xl shadow-xl",
          "transform transition-all",
          {
            "max-w-sm": size === "sm",
            "max-w-md": size === "md",
            "max-w-lg": size === "lg",
            "max-w-3xl": size === "xl",
            "max-w-full m-4": size === "full"
          }
        )}
      >
        {/* Header */}
        <div className="flex items-center justify-between p-4 border-b border-border-default">
          <Dialog.Title className="text-lg font-semibold text-text-default">
            {title}
          </Dialog.Title>
          <button
            onClick={onClose}
            className="p-2 rounded-md hover:bg-interactive-hover transition-colors"
            aria-label="Close"
          >
            <XIcon className="w-5 h-5 text-text-muted" />
          </button>
        </div>
        
        {/* Body */}
        <div className="p-4 max-h-[60vh] overflow-y-auto">
          {children}
        </div>
        
        {/* Footer */}
        {footer && (
          <div className="flex items-center justify-end gap-3 p-4 
                          border-t border-border-default bg-surface-subtle">
            {footer}
          </div>
        )}
      </Dialog.Panel>
    </div>
  </div>
</Dialog>
```

---

## 6.2 DRAWER / SIDE PANEL

### Descrizione:
Pannello che scorre dal bordo dello schermo.

### Varianti:
| Position | Descrizione |
|----------|-------------|
| `left` | Da sinistra |
| `right` | Da destra (più comune) |
| `bottom` | Dal basso (bottom sheet) |

### Struttura JSX:

```jsx
// DRAWER
<Transition show={isOpen}>
  {/* Backdrop */}
  <Transition.Child
    enter="transition-opacity duration-normal"
    enterFrom="opacity-0"
    enterTo="opacity-100"
    leave="transition-opacity duration-fast"
    leaveFrom="opacity-100"
    leaveTo="opacity-0"
  >
    <div 
      className="fixed inset-0 z-drawer-backdrop bg-black/50"
      onClick={onClose}
    />
  </Transition.Child>
  
  {/* Drawer Panel */}
  <Transition.Child
    enter="transition-transform duration-normal ease-out"
    enterFrom={position === 'right' ? 'translate-x-full' : '-translate-x-full'}
    enterTo="translate-x-0"
    leave="transition-transform duration-fast ease-in"
    leaveFrom="translate-x-0"
    leaveTo={position === 'right' ? 'translate-x-full' : '-translate-x-full'}
  >
    <div className={cn(
      "fixed top-0 bottom-0 z-drawer",
      "w-full max-w-md bg-surface-default shadow-xl",
      "flex flex-col",
      position === 'right' ? 'right-0' : 'left-0'
    )}>
      {/* Header */}
      <div className="flex items-center justify-between p-4 border-b border-border-default">
        <h2 className="text-lg font-semibold text-text-default">{title}</h2>
        <button
          onClick={onClose}
          className="p-2 rounded-md hover:bg-interactive-hover"
        >
          <XIcon className="w-5 h-5 text-text-muted" />
        </button>
      </div>
      
      {/* Content */}
      <div className="flex-1 overflow-y-auto p-4">
        {children}
      </div>
      
      {/* Footer */}
      {footer && (
        <div className="p-4 border-t border-border-default bg-surface-subtle">
          {footer}
        </div>
      )}
    </div>
  </Transition.Child>
</Transition>
```

---

## 6.3 POPOVER / DROPDOWN

### Struttura JSX:

```jsx
// POPOVER
<Popover className="relative">
  <Popover.Button className={triggerClassName}>
    {trigger}
  </Popover.Button>
  
  <Transition
    enter="transition duration-fast ease-out"
    enterFrom="opacity-0 scale-95"
    enterTo="opacity-100 scale-100"
    leave="transition duration-fast ease-in"
    leaveFrom="opacity-100 scale-100"
    leaveTo="opacity-0 scale-95"
  >
    <Popover.Panel className={cn(
      "absolute z-popover mt-2",
      "bg-surface-default border border-border-default rounded-lg shadow-lg",
      "min-w-[200px] p-4",
      {
        "left-0": align === "start",
        "right-0": align === "end",
        "left-1/2 -translate-x-1/2": align === "center"
      }
    )}>
      {children}
    </Popover.Panel>
  </Transition>
</Popover>

// DROPDOWN MENU
<Menu as="div" className="relative">
  <Menu.Button className={triggerClassName}>
    {trigger}
  </Menu.Button>
  
  <Menu.Items className={cn(
    "absolute right-0 z-dropdown mt-2",
    "w-56 py-1 bg-surface-default",
    "border border-border-default rounded-lg shadow-lg",
    "focus:outline-none"
  )}>
    {items.map(item => (
      <Menu.Item key={item.id}>
        {({ active }) => (
          <button
            onClick={item.onClick}
            disabled={item.disabled}
            className={cn(
              "w-full flex items-center gap-3 px-4 py-2 text-sm text-left",
              active && "bg-interactive-hover",
              item.destructive && "text-feedback-error",
              item.disabled && "opacity-50 cursor-not-allowed"
            )}
          >
            {item.icon && <item.icon className="w-4 h-4" />}
            {item.label}
            {item.shortcut && (
              <kbd className="ml-auto text-xs text-text-subtle">{item.shortcut}</kbd>
            )}
          </button>
        )}
      </Menu.Item>
    ))}
  </Menu.Items>
</Menu>
```

---

## 6.4 COMMAND PALETTE

### Descrizione:
Ricerca globale e navigazione rapida (Cmd+K pattern).

### Struttura JSX:

```jsx
// COMMAND PALETTE
<Dialog open={isOpen} onClose={onClose}>
  <div className="fixed inset-0 z-command bg-black/50 backdrop-blur-sm" />
  
  <div className="fixed inset-0 z-command flex items-start justify-center pt-[20vh] p-4">
    <Dialog.Panel className="w-full max-w-xl bg-surface-default rounded-xl shadow-2xl overflow-hidden">
      {/* Search Input */}
      <div className="flex items-center gap-3 px-4 border-b border-border-default">
        <SearchIcon className="w-5 h-5 text-text-muted shrink-0" />
        <input
          type="text"
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder="Search commands..."
          className="flex-1 h-14 bg-transparent text-text-default 
                     placeholder:text-text-subtle focus:outline-none"
          autoFocus
        />
        <kbd className="px-2 py-1 text-xs bg-surface-muted rounded">ESC</kbd>
      </div>
      
      {/* Results */}
      <div className="max-h-80 overflow-y-auto py-2">
        {groups.map(group => (
          <div key={group.name}>
            <div className="px-4 py-2 text-xs font-semibold text-text-subtle uppercase">
              {group.name}
            </div>
            {group.items.map((item, index) => (
              <button
                key={item.id}
                onClick={() => handleSelect(item)}
                className={cn(
                  "w-full flex items-center gap-3 px-4 py-2.5",
                  "text-sm text-text-default text-left",
                  "hover:bg-interactive-hover",
                  activeIndex === index && "bg-interactive-selected"
                )}
              >
                <item.icon className="w-5 h-5 text-text-muted" />
                <span className="flex-1">{item.label}</span>
                {item.shortcut && (
                  <kbd className="text-xs text-text-subtle">{item.shortcut}</kbd>
                )}
              </button>
            ))}
          </div>
        ))}
      </div>
      
      {/* Footer */}
      <div className="flex items-center gap-4 px-4 py-3 
                      border-t border-border-default bg-surface-subtle text-xs text-text-muted">
        <span><kbd>↑↓</kbd> navigate</span>
        <span><kbd>↵</kbd> select</span>
        <span><kbd>esc</kbd> close</span>
      </div>
    </Dialog.Panel>
  </div>
</Dialog>
```


# ============================================================================
# SEZIONE 7: DATA DISPLAY PATTERNS
# ============================================================================

## 7.1 TABLE

### Descrizione:
Tabella per visualizzare dati strutturati con sorting, filtering e pagination.

### Struttura JSX:

```jsx
// TABLE
<div className="border border-border-default rounded-lg overflow-hidden">
  {/* Table Header (filters, search, actions) */}
  <div className="flex items-center justify-between p-4 
                  border-b border-border-default bg-surface-subtle">
    <div className="flex items-center gap-3">
      <SearchInput value={search} onChange={setSearch} placeholder="Search..." />
      <Select value={filter} onChange={setFilter} options={filterOptions} />
    </div>
    <div className="flex items-center gap-2">
      <Button variant="outline" size="sm">Export</Button>
      <Button variant="primary" size="sm">Add New</Button>
    </div>
  </div>
  
  {/* Table Content */}
  <div className="overflow-x-auto">
    <table className="w-full">
      <thead>
        <tr className="bg-surface-subtle border-b border-border-default">
          {/* Select All */}
          {selectable && (
            <th className="w-12 px-4 py-3">
              <Checkbox checked={allSelected} onChange={toggleAll} />
            </th>
          )}
          
          {columns.map(column => (
            <th
              key={column.id}
              className={cn(
                "px-4 py-3 text-left text-xs font-semibold",
                "text-text-muted uppercase tracking-wider",
                column.sortable && "cursor-pointer hover:text-text-default"
              )}
              onClick={() => column.sortable && handleSort(column.id)}
            >
              <div className="flex items-center gap-1">
                {column.label}
                {column.sortable && sortColumn === column.id && (
                  <SortIcon direction={sortDirection} />
                )}
              </div>
            </th>
          ))}
          
          {/* Actions column */}
          <th className="w-12 px-4 py-3" />
        </tr>
      </thead>
      
      <tbody className="divide-y divide-border-default">
        {rows.map(row => (
          <tr 
            key={row.id}
            className="hover:bg-surface-subtle/50 transition-colors"
          >
            {selectable && (
              <td className="px-4 py-3">
                <Checkbox 
                  checked={selectedRows.includes(row.id)}
                  onChange={() => toggleRow(row.id)}
                />
              </td>
            )}
            
            {columns.map(column => (
              <td key={column.id} className="px-4 py-3 text-sm text-text-default">
                {column.render ? column.render(row) : row[column.accessorKey]}
              </td>
            ))}
            
            <td className="px-4 py-3">
              <DropdownMenu>
                <DropdownMenu.Trigger asChild>
                  <IconButton variant="ghost" size="sm" icon={MoreHorizontalIcon} />
                </DropdownMenu.Trigger>
                <DropdownMenu.Content>
                  <DropdownMenu.Item onClick={() => onEdit(row)}>Edit</DropdownMenu.Item>
                  <DropdownMenu.Item onClick={() => onDelete(row)} destructive>
                    Delete
                  </DropdownMenu.Item>
                </DropdownMenu.Content>
              </DropdownMenu>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  </div>
  
  {/* Pagination */}
  <div className="flex items-center justify-between p-4 
                  border-t border-border-default bg-surface-subtle">
    <span className="text-sm text-text-muted">
      Showing {startIndex + 1} to {endIndex} of {totalCount} results
    </span>
    <Pagination
      currentPage={page}
      totalPages={totalPages}
      onPageChange={setPage}
    />
  </div>
</div>
```

---

## 7.2 STAT CARD / KPI

### Descrizione:
Card per visualizzare metriche chiave con trend.

### Struttura JSX:

```jsx
// STAT CARD
<div className="bg-surface-default border border-border-default rounded-lg p-6">
  <div className="flex items-start justify-between">
    {/* Stat Content */}
    <div>
      <p className="text-sm font-medium text-text-muted">{label}</p>
      <p className="text-3xl font-bold text-text-default mt-2">
        {formatValue(value)}
      </p>
      
      {/* Trend */}
      {trend && (
        <div className={cn(
          "flex items-center gap-1 mt-2 text-sm",
          trend.direction === "up" 
            ? trend.isPositive ? "text-feedback-success" : "text-feedback-error"
            : trend.isPositive ? "text-feedback-error" : "text-feedback-success"
        )}>
          {trend.direction === "up" ? (
            <TrendingUpIcon className="w-4 h-4" />
          ) : (
            <TrendingDownIcon className="w-4 h-4" />
          )}
          <span>{trend.value}%</span>
          <span className="text-text-muted">vs last period</span>
        </div>
      )}
    </div>
    
    {/* Icon */}
    <div className={cn(
      "p-3 rounded-lg",
      iconBgColor || "bg-surface-brand/10"
    )}>
      <StatIcon className="w-6 h-6 text-text-brand" />
    </div>
  </div>
  
  {/* Sparkline (optional) */}
  {sparklineData && (
    <div className="mt-4 h-12">
      <Sparkline data={sparklineData} />
    </div>
  )}
</div>
```

---

## 7.3 CHART WRAPPER

### Descrizione:
Container standardizzato per grafici.

### Struttura JSX:

```jsx
// CHART WRAPPER
<div className="bg-surface-default border border-border-default rounded-lg">
  {/* Header */}
  <div className="flex items-center justify-between p-4 border-b border-border-default">
    <div>
      <h3 className="text-lg font-semibold text-text-default">{title}</h3>
      {subtitle && (
        <p className="text-sm text-text-muted mt-0.5">{subtitle}</p>
      )}
    </div>
    
    <div className="flex items-center gap-2">
      {/* Time Range Selector */}
      {timeRanges && (
        <SegmentedControl
          options={timeRanges}
          value={selectedRange}
          onChange={setSelectedRange}
        />
      )}
      
      {/* Actions */}
      <DropdownMenu>
        <DropdownMenu.Trigger asChild>
          <IconButton variant="ghost" icon={MoreVerticalIcon} />
        </DropdownMenu.Trigger>
        <DropdownMenu.Content>
          <DropdownMenu.Item>Download CSV</DropdownMenu.Item>
          <DropdownMenu.Item>Download PNG</DropdownMenu.Item>
          <DropdownMenu.Item>Full Screen</DropdownMenu.Item>
        </DropdownMenu.Content>
      </DropdownMenu>
    </div>
  </div>
  
  {/* Chart Area */}
  <div className="p-4">
    <div className="h-64 lg:h-80">
      {isLoading ? (
        <div className="h-full flex items-center justify-center">
          <Spinner size="lg" />
        </div>
      ) : (
        children
      )}
    </div>
  </div>
  
  {/* Legend */}
  {legend && (
    <div className="flex items-center justify-center gap-6 p-4 
                    border-t border-border-muted">
      {legend.map(item => (
        <div key={item.id} className="flex items-center gap-2">
          <span 
            className="w-3 h-3 rounded-full"
            style={{ backgroundColor: item.color }}
          />
          <span className="text-sm text-text-muted">{item.label}</span>
        </div>
      ))}
    </div>
  )}
</div>
```

---

## 7.4 TIMELINE

### Struttura JSX:

```jsx
// TIMELINE
<div className="relative">
  {events.map((event, index) => (
    <div key={event.id} className="relative flex gap-4 pb-8 last:pb-0">
      {/* Vertical Line */}
      {index < events.length - 1 && (
        <div className="absolute left-[15px] top-8 bottom-0 w-0.5 bg-border-default" />
      )}
      
      {/* Dot */}
      <div className={cn(
        "relative z-10 w-8 h-8 rounded-full shrink-0",
        "flex items-center justify-center",
        event.status === "completed" && "bg-feedback-success",
        event.status === "current" && "bg-surface-brand",
        event.status === "upcoming" && "bg-surface-muted border-2 border-border-default"
      )}>
        {event.status === "completed" && (
          <CheckIcon className="w-4 h-4 text-white" />
        )}
        {event.icon && event.status !== "completed" && (
          <event.icon className="w-4 h-4 text-white" />
        )}
      </div>
      
      {/* Content */}
      <div className="flex-1 pt-0.5">
        <div className="flex items-center gap-2 mb-1">
          <h4 className="font-semibold text-text-default">{event.title}</h4>
          <time className="text-xs text-text-subtle">{event.timestamp}</time>
        </div>
        {event.description && (
          <p className="text-sm text-text-muted">{event.description}</p>
        )}
        {event.content && (
          <div className="mt-2">{event.content}</div>
        )}
      </div>
    </div>
  ))}
</div>
```

---

## 7.5 DESCRIPTION LIST

### Struttura JSX:

```jsx
// DESCRIPTION LIST
<dl className="grid grid-cols-1 sm:grid-cols-2 gap-x-6 gap-y-4">
  {items.map(item => (
    <div key={item.label} className={cn(
      item.fullWidth && "sm:col-span-2"
    )}>
      <dt className="text-sm font-medium text-text-muted">{item.label}</dt>
      <dd className="text-sm text-text-default mt-1">{item.value}</dd>
    </div>
  ))}
</dl>

// STACKED DESCRIPTION LIST
<dl className="divide-y divide-border-default">
  {items.map(item => (
    <div 
      key={item.label}
      className="flex items-center justify-between py-3"
    >
      <dt className="text-sm text-text-muted">{item.label}</dt>
      <dd className="text-sm font-medium text-text-default">{item.value}</dd>
    </div>
  ))}
</dl>
```

# ============================================================================
# SEZIONE 8: ACTION PATTERNS
# ============================================================================

## 8.1 BUTTON

### Descrizione:
Elemento interattivo per azioni.

### Varianti:
| Variant | Uso |
|---------|-----|
| `primary` | Azione principale |
| `secondary` | Azione secondaria |
| `outline` | Azione terziaria |
| `ghost` | Azione minimale |
| `destructive` | Azione pericolosa |
| `link` | Come link testuale |

### Sizes:
| Size | Height | Uso |
|------|--------|-----|
| `xs` | 28px | Compact UI |
| `sm` | 32px | Dense layout |
| `md` | 40px | Default |
| `lg` | 48px | CTA prominenti |

### Struttura JSX:

```jsx
// BUTTON
<button
  type={type}
  disabled={disabled || loading}
  className={cn(
    "inline-flex items-center justify-center gap-2",
    "font-medium rounded-md transition-colors duration-fast",
    "focus-visible:outline-none focus-visible:ring-2",
    "focus-visible:ring-ring-focus focus-visible:ring-offset-2",
    "disabled:opacity-50 disabled:pointer-events-none",
    
    // Size
    {
      "h-7 px-2 text-xs": size === "xs",
      "h-8 px-3 text-sm": size === "sm",
      "h-10 px-4 text-sm": size === "md",
      "h-12 px-6 text-base": size === "lg"
    },
    
    // Variant
    {
      "bg-surface-brand text-text-inverse hover:bg-surface-brand-emphasis": 
        variant === "primary",
      "bg-surface-muted text-text-default hover:bg-surface-muted-emphasis": 
        variant === "secondary",
      "border border-border-default text-text-default hover:bg-interactive-hover": 
        variant === "outline",
      "text-text-default hover:bg-interactive-hover": 
        variant === "ghost",
      "bg-feedback-error text-text-inverse hover:bg-feedback-error/90": 
        variant === "destructive",
      "text-text-brand underline-offset-4 hover:underline p-0 h-auto": 
        variant === "link"
    },
    
    fullWidth && "w-full"
  )}
>
  {loading && <Spinner size="sm" />}
  {!loading && leftIcon && <span className="shrink-0">{leftIcon}</span>}
  {children}
  {!loading && rightIcon && <span className="shrink-0">{rightIcon}</span>}
</button>
```

---

## 8.2 ICON BUTTON

### Struttura JSX:

```jsx
// ICON BUTTON
<button
  type="button"
  disabled={disabled}
  aria-label={ariaLabel}
  className={cn(
    "inline-flex items-center justify-center rounded-md",
    "transition-colors duration-fast",
    "focus-visible:outline-none focus-visible:ring-2",
    "focus-visible:ring-ring-focus focus-visible:ring-offset-2",
    "disabled:opacity-50 disabled:pointer-events-none",
    
    // Size
    {
      "w-7 h-7": size === "xs",
      "w-8 h-8": size === "sm",
      "w-10 h-10": size === "md",
      "w-12 h-12": size === "lg"
    },
    
    // Variant
    {
      "bg-surface-brand text-text-inverse hover:bg-surface-brand-emphasis": 
        variant === "primary",
      "bg-transparent text-text-muted hover:bg-interactive-hover hover:text-text-default": 
        variant === "ghost",
      "border border-border-default text-text-muted hover:bg-interactive-hover": 
        variant === "outline"
    }
  )}
>
  <Icon className={cn(
    size === "xs" && "w-3.5 h-3.5",
    size === "sm" && "w-4 h-4",
    size === "md" && "w-5 h-5",
    size === "lg" && "w-6 h-6"
  )} />
</button>
```

---

## 8.3 BUTTON GROUP

### Struttura JSX:

```jsx
// BUTTON GROUP
<div className="inline-flex rounded-md shadow-sm" role="group">
  {buttons.map((button, index) => (
    <button
      key={button.id}
      type="button"
      onClick={button.onClick}
      disabled={button.disabled}
      className={cn(
        "px-4 py-2 text-sm font-medium",
        "border border-border-default",
        "focus:z-10 focus:ring-2 focus:ring-ring-focus",
        "disabled:opacity-50",
        
        // Position-based rounding
        index === 0 && "rounded-l-md",
        index === buttons.length - 1 && "rounded-r-md",
        index > 0 && "-ml-px",
        
        // Active state
        button.isActive
          ? "bg-surface-brand text-text-inverse border-surface-brand z-10"
          : "bg-surface-default text-text-default hover:bg-interactive-hover"
      )}
    >
      {button.icon && <button.icon className="w-4 h-4 mr-2" />}
      {button.label}
    </button>
  ))}
</div>
```

---

## 8.4 FLOATING ACTION BUTTON (FAB)

### Struttura JSX:

```jsx
// FAB
<button
  className={cn(
    "fixed z-fab",
    "rounded-full shadow-lg",
    "bg-surface-brand text-text-inverse",
    "hover:bg-surface-brand-emphasis",
    "focus:outline-none focus:ring-2 focus:ring-ring-focus focus:ring-offset-2",
    "transition-transform hover:scale-105",
    
    // Position
    {
      "bottom-6 right-6": position === "bottom-right",
      "bottom-6 left-6": position === "bottom-left"
    },
    
    // Size
    {
      "w-12 h-12": size === "sm",
      "w-14 h-14": size === "md",
      "w-16 h-16": size === "lg"
    }
  )}
>
  <PlusIcon className="w-6 h-6" />
</button>

// EXTENDED FAB
<button className={cn(
  "fixed bottom-6 right-6 z-fab",
  "inline-flex items-center gap-2 px-5 h-14",
  "rounded-full shadow-lg",
  "bg-surface-brand text-text-inverse",
  "hover:bg-surface-brand-emphasis",
  "transition-transform hover:scale-105"
)}>
  <PlusIcon className="w-5 h-5" />
  <span className="font-medium">{label}</span>
</button>
```

---

## 8.5 SEGMENTED CONTROL

### Struttura JSX:

```jsx
// SEGMENTED CONTROL
<div 
  className="inline-flex p-1 bg-surface-muted rounded-lg"
  role="radiogroup"
>
  {options.map(option => (
    <button
      key={option.value}
      type="button"
      role="radio"
      aria-checked={value === option.value}
      onClick={() => onChange(option.value)}
      className={cn(
        "px-4 py-1.5 text-sm font-medium rounded-md",
        "transition-colors duration-fast",
        value === option.value
          ? "bg-surface-default text-text-default shadow-sm"
          : "text-text-muted hover:text-text-default"
      )}
    >
      {option.icon && <option.icon className="w-4 h-4 mr-2" />}
      {option.label}
    </button>
  ))}
</div>
```

---

## 8.6 CONFIRMATION DIALOG

### Struttura JSX:

```jsx
// CONFIRMATION DIALOG
<Dialog open={isOpen} onClose={onCancel}>
  <div className="fixed inset-0 z-modal-backdrop bg-black/50" />
  
  <div className="fixed inset-0 z-modal flex items-center justify-center p-4">
    <Dialog.Panel className="w-full max-w-sm bg-surface-default rounded-xl p-6 shadow-xl">
      {/* Icon */}
      <div className={cn(
        "w-12 h-12 mx-auto mb-4 rounded-full",
        "flex items-center justify-center",
        type === "danger" && "bg-feedback-error/10",
        type === "warning" && "bg-feedback-warning/10",
        type === "info" && "bg-feedback-info/10"
      )}>
        <TypeIcon className={cn(
          "w-6 h-6",
          type === "danger" && "text-feedback-error",
          type === "warning" && "text-feedback-warning",
          type === "info" && "text-feedback-info"
        )} />
      </div>
      
      {/* Content */}
      <Dialog.Title className="text-lg font-semibold text-text-default text-center">
        {title}
      </Dialog.Title>
      <Dialog.Description className="text-sm text-text-muted text-center mt-2">
        {description}
      </Dialog.Description>
      
      {/* Actions */}
      <div className="flex gap-3 mt-6">
        <Button variant="outline" onClick={onCancel} fullWidth>
          {cancelLabel || "Cancel"}
        </Button>
        <Button 
          variant={type === "danger" ? "destructive" : "primary"} 
          onClick={onConfirm}
          loading={isLoading}
          fullWidth
        >
          {confirmLabel || "Confirm"}
        </Button>
      </div>
    </Dialog.Panel>
  </div>
</Dialog>
```


# ============================================================================
# SEZIONE 9: RESPONSIVE PATTERNS
# ============================================================================

## 9.1 RESPONSIVE BREAKPOINTS

### Descrizione:
Breakpoint standard per design responsive (Tailwind defaults).

### Breakpoints:
| Nome | Min Width | Target |
|------|-----------|--------|
| `sm` | 640px | Mobile landscape |
| `md` | 768px | Tablet |
| `lg` | 1024px | Desktop small |
| `xl` | 1280px | Desktop |
| `2xl` | 1536px | Desktop large |

### Pattern di utilizzo:

```jsx
// Mobile-first responsive classes
<div className={cn(
  "px-4",           // Mobile: padding 16px
  "sm:px-6",        // ≥640px: padding 24px
  "lg:px-8",        // ≥1024px: padding 32px
  "xl:px-12"        // ≥1280px: padding 48px
)}>

// Responsive grid
<div className={cn(
  "grid grid-cols-1",      // Mobile: 1 colonna
  "sm:grid-cols-2",        // ≥640px: 2 colonne
  "lg:grid-cols-3",        // ≥1024px: 3 colonne
  "xl:grid-cols-4"         // ≥1280px: 4 colonne
)}>

// Hide/show elements
<div className="hidden lg:block">Desktop only</div>
<div className="lg:hidden">Mobile/tablet only</div>
```

---

## 9.2 CONTAINER

### Struttura JSX:

```jsx
// CONTAINER
<div className={cn(
  "w-full mx-auto px-4",
  "sm:px-6 lg:px-8",
  {
    "max-w-screen-sm": size === "sm",    // 640px
    "max-w-screen-md": size === "md",    // 768px
    "max-w-screen-lg": size === "lg",    // 1024px
    "max-w-screen-xl": size === "xl",    // 1280px
    "max-w-screen-2xl": size === "2xl",  // 1536px
    "max-w-7xl": size === "default"      // 1280px (common)
  }
)}>
  {children}
</div>
```

---

## 9.3 RESPONSIVE NAVIGATION

### Struttura JSX:

```jsx
// RESPONSIVE HEADER
<header className="sticky top-0 z-header bg-surface-default border-b border-border-default">
  <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div className="flex items-center justify-between h-16">
      {/* Logo */}
      <Logo />
      
      {/* Desktop Navigation */}
      <nav className="hidden md:flex items-center gap-6">
        {navItems.map(item => (
          <NavLink key={item.href} {...item} />
        ))}
      </nav>
      
      {/* Desktop Actions */}
      <div className="hidden md:flex items-center gap-4">
        <Button variant="ghost">Sign In</Button>
        <Button variant="primary">Get Started</Button>
      </div>
      
      {/* Mobile Menu Button */}
      <button
        className="md:hidden p-2 rounded-md hover:bg-interactive-hover"
        onClick={() => setMobileMenuOpen(true)}
        aria-label="Open menu"
      >
        <MenuIcon className="w-6 h-6" />
      </button>
    </div>
  </div>
  
  {/* Mobile Menu (Drawer) */}
  <MobileMenuDrawer 
    isOpen={mobileMenuOpen}
    onClose={() => setMobileMenuOpen(false)}
    items={navItems}
  />
</header>
```

---

## 9.4 RESPONSIVE SIDEBAR LAYOUT

### Struttura JSX:

```jsx
// RESPONSIVE SIDEBAR LAYOUT
<div className="min-h-screen flex">
  {/* Sidebar - Desktop */}
  <aside className={cn(
    "hidden lg:flex lg:flex-col lg:w-64 lg:fixed lg:inset-y-0",
    "bg-surface-default border-r border-border-default"
  )}>
    <SidebarContent />
  </aside>
  
  {/* Sidebar - Mobile (Drawer) */}
  <Drawer
    isOpen={sidebarOpen}
    onClose={() => setSidebarOpen(false)}
    position="left"
  >
    <SidebarContent />
  </Drawer>
  
  {/* Main Content */}
  <div className="flex-1 lg:pl-64">
    {/* Mobile Header */}
    <header className="lg:hidden sticky top-0 z-header 
                       bg-surface-default border-b border-border-default">
      <div className="flex items-center justify-between h-16 px-4">
        <button
          onClick={() => setSidebarOpen(true)}
          className="p-2 rounded-md hover:bg-interactive-hover"
        >
          <MenuIcon className="w-6 h-6" />
        </button>
        <Logo />
        <div className="w-10" /> {/* Spacer for centering */}
      </div>
    </header>
    
    {/* Page Content */}
    <main className="p-4 sm:p-6 lg:p-8">
      {children}
    </main>
  </div>
</div>
```

---

## 9.5 RESPONSIVE CARD GRID

### Struttura JSX:

```jsx
// RESPONSIVE CARD GRID
<div className={cn(
  "grid gap-4 sm:gap-6",
  // Columns based on density
  {
    // Compact: more cards
    "grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 xl:grid-cols-6":
      density === "compact",
    // Default
    "grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4":
      density === "default",
    // Comfortable: fewer, larger cards
    "grid-cols-1 md:grid-cols-2 lg:grid-cols-3":
      density === "comfortable"
  }
)}>
  {items.map(item => (
    <Card key={item.id} {...item} />
  ))}
</div>
```

---

## 9.6 RESPONSIVE TABLE → CARD LIST

### Struttura JSX:

```jsx
// RESPONSIVE TABLE (Desktop) → CARD LIST (Mobile)
<div>
  {/* Desktop Table */}
  <div className="hidden md:block">
    <DataTable columns={columns} data={data} />
  </div>
  
  {/* Mobile Card List */}
  <div className="md:hidden space-y-4">
    {data.map(row => (
      <div 
        key={row.id}
        className="bg-surface-default border border-border-default rounded-lg p-4"
      >
        {/* Primary Info */}
        <div className="flex items-center gap-3 mb-3">
          {row.avatar && <Avatar src={row.avatar} size="md" />}
          <div>
            <p className="font-medium text-text-default">{row.name}</p>
            <p className="text-sm text-text-muted">{row.email}</p>
          </div>
        </div>
        
        {/* Secondary Info */}
        <dl className="grid grid-cols-2 gap-2 text-sm">
          {mobileFields.map(field => (
            <div key={field.key}>
              <dt className="text-text-muted">{field.label}</dt>
              <dd className="font-medium text-text-default">{row[field.key]}</dd>
            </div>
          ))}
        </dl>
        
        {/* Actions */}
        <div className="flex gap-2 mt-4 pt-4 border-t border-border-muted">
          <Button variant="outline" size="sm" fullWidth>Edit</Button>
          <Button variant="ghost" size="sm" fullWidth>Delete</Button>
        </div>
      </div>
    ))}
  </div>
</div>
```

# ============================================================================
# SEZIONE 10: ANIMATION PATTERNS
# ============================================================================

## 10.1 TRANSITION DURATIONS

### Token Reference:
| Token | Value | Uso |
|-------|-------|-----|
| `duration-fast` | 150ms | Micro-interactions, hover |
| `duration-normal` | 200ms | Standard transitions |
| `duration-slow` | 300ms | Elementi complessi |
| `duration-slower` | 500ms | Page transitions |

---

## 10.2 EASING FUNCTIONS

### Token Reference:
| Token | Value | Uso |
|-------|-------|-----|
| `ease-default` | cubic-bezier(0.4, 0, 0.2, 1) | Default |
| `ease-in` | cubic-bezier(0.4, 0, 1, 1) | Elementi che escono |
| `ease-out` | cubic-bezier(0, 0, 0.2, 1) | Elementi che entrano |
| `ease-in-out` | cubic-bezier(0.4, 0, 0.2, 1) | Bi-direzionale |

---

## 10.3 COMMON ANIMATIONS

### Keyframes CSS:

```css
/* Fade In */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

/* Fade Out */
@keyframes fadeOut {
  from { opacity: 1; }
  to { opacity: 0; }
}

/* Slide In from Bottom */
@keyframes slideInUp {
  from { 
    opacity: 0;
    transform: translateY(10px);
  }
  to { 
    opacity: 1;
    transform: translateY(0);
  }
}

/* Slide In from Right */
@keyframes slideInRight {
  from { 
    opacity: 0;
    transform: translateX(10px);
  }
  to { 
    opacity: 1;
    transform: translateX(0);
  }
}

/* Scale In */
@keyframes scaleIn {
  from { 
    opacity: 0;
    transform: scale(0.95);
  }
  to { 
    opacity: 1;
    transform: scale(1);
  }
}

/* Spin (for loaders) */
@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* Pulse (for attention) */
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

/* Bounce (for CTAs) */
@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-5px); }
}
```

### Tailwind Classes:

```jsx
// Utilizzo con Tailwind
<div className="animate-fadeIn" />
<div className="animate-spin" />
<div className="animate-pulse" />
<div className="animate-bounce" />

// Custom animation con arbitrary values
<div className="animate-[slideInUp_0.3s_ease-out]" />
```

---

## 10.4 TRANSITION PATTERNS

### Struttura JSX:

```jsx
// HOVER TRANSITIONS
<button className={cn(
  "transition-colors duration-fast",
  "hover:bg-interactive-hover"
)}>

// SCALE ON HOVER (Cards, Images)
<div className={cn(
  "transition-transform duration-normal",
  "hover:scale-105"
)}>

// FOCUS RING
<input className={cn(
  "transition-shadow duration-fast",
  "focus:ring-2 focus:ring-ring-focus"
)}>

// OPACITY TRANSITION
<div className={cn(
  "transition-opacity duration-normal",
  isVisible ? "opacity-100" : "opacity-0"
)}>

// COMBINED TRANSITION
<div className={cn(
  "transition-all duration-normal",
  "hover:shadow-lg hover:scale-[1.02]"
)}>
```

---

## 10.5 PAGE TRANSITION (with Framer Motion style)

### Struttura JSX:

```jsx
// PAGE TRANSITION WRAPPER
const pageVariants = {
  initial: { opacity: 0, y: 10 },
  enter: { opacity: 1, y: 0 },
  exit: { opacity: 0, y: -10 }
};

const pageTransition = {
  duration: 0.3,
  ease: [0.4, 0, 0.2, 1]
};

// Con CSS classes (senza Framer Motion)
<div className={cn(
  "animate-[fadeIn_0.3s_ease-out]",
  // oppure con state
  entering && "animate-slideInUp",
  exiting && "animate-fadeOut"
)}>
  {children}
</div>
```

---

## 10.6 SKELETON ANIMATION

### Struttura JSX:

```jsx
// SKELETON con shimmer effect
<div className="relative overflow-hidden bg-surface-muted rounded">
  <div className={cn(
    "absolute inset-0",
    "bg-gradient-to-r from-transparent via-white/20 to-transparent",
    "animate-[shimmer_2s_infinite]"
  )} />
</div>

// Keyframe per shimmer
@keyframes shimmer {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(100%); }
}
```

# ============================================================================
# SEZIONE 11: ACCESSIBILITY PATTERNS
# ============================================================================

## 11.1 FOCUS MANAGEMENT

### Pattern:

```jsx
// FOCUS VISIBLE (keyboard only)
<button className={cn(
  "focus:outline-none",
  "focus-visible:ring-2 focus-visible:ring-ring-focus",
  "focus-visible:ring-offset-2"
)}>

// FOCUS TRAP (per modali)
import { FocusTrap } from '@headlessui/react';

<FocusTrap>
  <Dialog.Panel>
    {/* Focus rimane all'interno */}
  </Dialog.Panel>
</FocusTrap>

// SKIP LINK
<a 
  href="#main-content"
  className={cn(
    "sr-only focus:not-sr-only",
    "focus:absolute focus:top-4 focus:left-4",
    "focus:z-50 focus:px-4 focus:py-2",
    "focus:bg-surface-brand focus:text-white focus:rounded"
  )}
>
  Skip to main content
</a>
```

---

## 11.2 ARIA PATTERNS

### Pattern comuni:

```jsx
// LIVE REGION (per notifiche dinamiche)
<div 
  role="status" 
  aria-live="polite" 
  aria-atomic="true"
>
  {notification}
</div>

// ALERT (per errori)
<div role="alert" aria-live="assertive">
  {errorMessage}
</div>

// DIALOG
<div
  role="dialog"
  aria-modal="true"
  aria-labelledby="dialog-title"
  aria-describedby="dialog-description"
>
  <h2 id="dialog-title">{title}</h2>
  <p id="dialog-description">{description}</p>
</div>

// NAVIGATION
<nav aria-label="Main navigation">
  <ul role="list">
    <li><a href="/" aria-current={isActive ? "page" : undefined}>Home</a></li>
  </ul>
</nav>

// TABS
<div role="tablist" aria-label="Account settings">
  <button 
    role="tab" 
    aria-selected={activeTab === 'profile'}
    aria-controls="panel-profile"
    id="tab-profile"
  >
    Profile
  </button>
</div>
<div 
  role="tabpanel" 
  id="panel-profile"
  aria-labelledby="tab-profile"
  tabIndex={0}
>
  {/* content */}
</div>

// FORM FIELDS
<div>
  <label htmlFor="email">Email</label>
  <input
    id="email"
    type="email"
    aria-required="true"
    aria-invalid={!!error}
    aria-describedby={error ? "email-error" : "email-hint"}
  />
  <p id="email-hint" className="text-sm text-text-muted">
    We'll never share your email.
  </p>
  {error && (
    <p id="email-error" role="alert" className="text-sm text-feedback-error">
      {error}
    </p>
  )}
</div>

// LOADING STATE
<button disabled aria-busy="true">
  <Spinner aria-hidden="true" />
  <span>Loading...</span>
</button>

// EXPAND/COLLAPSE
<button
  aria-expanded={isExpanded}
  aria-controls="content-panel"
>
  {isExpanded ? 'Collapse' : 'Expand'}
</button>
<div id="content-panel" hidden={!isExpanded}>
  {content}
</div>
```

---

## 11.3 SCREEN READER UTILITIES

### Struttura JSX:

```jsx
// VISUALLY HIDDEN (solo screen reader)
<span className="sr-only">
  Opens in a new window
</span>

// ARIA HIDDEN (nascondi da screen reader)
<span aria-hidden="true">•</span>

// ICON CON LABEL ACCESSIBILE
<button aria-label="Close dialog">
  <XIcon aria-hidden="true" className="w-5 h-5" />
</button>

// ICON DECORATIVA
<div className="flex items-center gap-2">
  <CheckIcon aria-hidden="true" className="w-5 h-5 text-feedback-success" />
  <span>Task completed</span>
</div>

// LINK CON CONTESTO
<a href="/settings">
  Settings
  <span className="sr-only">, opens account settings page</span>
</a>
```

---

## 11.4 COLOR CONTRAST

### Requisiti WCAG 2.1:

| Livello | Testo normale | Testo grande | UI Components |
|---------|---------------|--------------|---------------|
| AA | 4.5:1 | 3:1 | 3:1 |
| AAA | 7:1 | 4.5:1 | 4.5:1 |

### Pattern per verificare contrast:

```jsx
// Colori ad alto contrasto per testo
const textColors = {
  default: 'text-text-default',      // Contrasto ≥ 4.5:1
  muted: 'text-text-muted',          // Contrasto ≥ 4.5:1
  subtle: 'text-text-subtle',        // Solo decorativo, non info essenziale
};

// Feedback colors con contrasto sufficiente
const feedbackColors = {
  error: 'text-feedback-error',      // #DC2626 su bianco = 4.5:1 ✓
  success: 'text-feedback-success',  // #16A34A su bianco = 4.5:1 ✓
  warning: 'text-feedback-warning',  // #D97706 su bianco = 3.1:1 (usa con icona)
};
```

---

## 11.5 KEYBOARD NAVIGATION

### Pattern:

```jsx
// TAB ORDER
<div className="flex gap-2">
  <Button tabIndex={0}>First</Button>
  <Button tabIndex={0}>Second</Button>
  <Button tabIndex={-1}>Skip in tab order</Button>
</div>

// ARROW KEY NAVIGATION (per menu, tabs, listbox)
function useArrowNavigation(items, activeIndex, setActiveIndex) {
  const handleKeyDown = (e) => {
    switch (e.key) {
      case 'ArrowDown':
      case 'ArrowRight':
        e.preventDefault();
        setActiveIndex((activeIndex + 1) % items.length);
        break;
      case 'ArrowUp':
      case 'ArrowLeft':
        e.preventDefault();
        setActiveIndex((activeIndex - 1 + items.length) % items.length);
        break;
      case 'Home':
        e.preventDefault();
        setActiveIndex(0);
        break;
      case 'End':
        e.preventDefault();
        setActiveIndex(items.length - 1);
        break;
    }
  };
  
  return { handleKeyDown };
}

// ESCAPE KEY (per chiudere)
useEffect(() => {
  const handleEscape = (e) => {
    if (e.key === 'Escape') {
      onClose();
    }
  };
  
  document.addEventListener('keydown', handleEscape);
  return () => document.removeEventListener('keydown', handleEscape);
}, [onClose]);
```


# ============================================================================
# SEZIONE 12: PATTERN COMPOSITION GUIDE
# ============================================================================

"""
Questa sezione mostra come combinare i pattern primitivi per costruire
componenti complessi per ogni categoria di piattaforma.
"""

## 12.1 COMPOSIZIONE: E-COMMERCE CHECKOUT

### Primitivi utilizzati:
- Layout: PageHeader, TwoColumnLayout
- Navigation: Stepper, Breadcrumb
- Content: Card, List
- Input: TextInput, Select, Checkbox, RadioGroup
- Feedback: Alert, Toast, Progress
- Action: Button, ButtonGroup
- Data: DescriptionList

### Struttura composita:

```jsx
// E-COMMERCE CHECKOUT PAGE
function CheckoutPage() {
  return (
    <div className="min-h-screen bg-surface-subtle">
      {/* Header */}
      <PageHeader 
        title="Checkout"
        breadcrumb={[
          { label: 'Cart', href: '/cart' },
          { label: 'Checkout' }
        ]}
      />
      
      {/* Stepper */}
      <div className="max-w-4xl mx-auto px-4 py-6">
        <CheckoutStepper currentStep={currentStep} />
      </div>
      
      {/* Two Column Layout */}
      <div className="max-w-7xl mx-auto px-4 pb-12">
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
          {/* Main Content (2/3) */}
          <div className="lg:col-span-2 space-y-6">
            {/* Shipping Address Form */}
            <Card>
              <CardHeader title="Shipping Address" />
              <CardBody>
                <form className="grid grid-cols-2 gap-4">
                  <TextInput label="First Name" required />
                  <TextInput label="Last Name" required />
                  <TextInput label="Email" type="email" className="col-span-2" />
                  <TextInput label="Address" className="col-span-2" />
                  <TextInput label="City" />
                  <Select label="State" options={states} />
                  <TextInput label="ZIP Code" />
                  <Select label="Country" options={countries} />
                </form>
              </CardBody>
            </Card>
            
            {/* Shipping Method */}
            <Card>
              <CardHeader title="Shipping Method" />
              <CardBody>
                <RadioGroup
                  value={shippingMethod}
                  onChange={setShippingMethod}
                  options={[
                    { value: 'standard', label: 'Standard (5-7 days)', price: 'Free' },
                    { value: 'express', label: 'Express (2-3 days)', price: '$9.99' },
                    { value: 'overnight', label: 'Overnight', price: '$19.99' }
                  ]}
                />
              </CardBody>
            </Card>
            
            {/* Payment */}
            <Card>
              <CardHeader title="Payment Method" />
              <CardBody>
                {/* Payment form with PCI-compliant fields */}
                <PaymentForm />
              </CardBody>
            </Card>
          </div>
          
          {/* Sidebar (1/3) */}
          <div className="lg:col-span-1">
            <OrderSummary cart={cart} sticky />
          </div>
        </div>
      </div>
    </div>
  );
}
```

---

## 12.2 COMPOSIZIONE: SAAS DASHBOARD

### Primitivi utilizzati:
- Layout: AppShell, SidebarLayout, Grid
- Navigation: Sidebar, Tabs, Breadcrumb
- Content: Card, StatCard, EmptyState
- Data: Table, Chart, Timeline
- Feedback: Alert, Badge, Progress
- Overlay: CommandPalette, Modal
- Action: Button, DropdownMenu

### Struttura composita:

```jsx
// SAAS DASHBOARD
function DashboardPage() {
  return (
    <AppShell>
      {/* Sidebar */}
      <Sidebar>
        <SidebarHeader logo={<Logo />} />
        <SidebarNav items={navItems} activeItem={activeNav} />
        <SidebarFooter>
          <UsageMeter resource={usage} />
        </SidebarFooter>
      </Sidebar>
      
      {/* Main Content */}
      <MainContent>
        {/* Top Bar */}
        <TopBar>
          <Breadcrumb items={breadcrumbs} />
          <div className="flex items-center gap-4">
            <SearchButton onClick={openCommandPalette} />
            <NotificationBell count={notifications.length} />
            <UserMenu user={currentUser} />
          </div>
        </TopBar>
        
        {/* Page Header */}
        <PageHeader
          title="Dashboard"
          description="Overview of your account"
          actions={
            <Button variant="primary">
              <PlusIcon className="w-4 h-4 mr-2" />
              New Project
            </Button>
          }
        />
        
        {/* KPI Grid */}
        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 mb-8">
          <StatCard label="Total Revenue" value="$45,231" trend={{ direction: 'up', value: 12 }} />
          <StatCard label="Active Users" value="2,350" trend={{ direction: 'up', value: 8 }} />
          <StatCard label="Conversion Rate" value="3.2%" trend={{ direction: 'down', value: 2 }} />
          <StatCard label="Avg. Session" value="4m 32s" trend={{ direction: 'up', value: 5 }} />
        </div>
        
        {/* Charts Row */}
        <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
          <ChartCard title="Revenue Over Time">
            <LineChart data={revenueData} />
          </ChartCard>
          <ChartCard title="Traffic Sources">
            <PieChart data={trafficData} />
          </ChartCard>
        </div>
        
        {/* Recent Activity & Quick Actions */}
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
          <div className="lg:col-span-2">
            <Card>
              <CardHeader title="Recent Activity" />
              <ActivityFeed activities={recentActivities} />
            </Card>
          </div>
          <div>
            <Card>
              <CardHeader title="Quick Actions" />
              <QuickActionsList actions={quickActions} />
            </Card>
          </div>
        </div>
      </MainContent>
      
      {/* Command Palette */}
      <CommandPalette isOpen={commandPaletteOpen} onClose={closeCommandPalette} />
    </AppShell>
  );
}
```

---

## 12.3 COMPOSIZIONE: SOCIAL FEED

### Primitivi utilizzati:
- Layout: InfiniteScroll, TwoColumnLayout
- Content: Card, Avatar, MediaGrid
- Input: TextArea, FileUpload
- Feedback: Toast, Badge
- Action: IconButton, DropdownMenu
- Overlay: Modal, Popover

### Struttura composita:

```jsx
// SOCIAL FEED PAGE
function FeedPage() {
  return (
    <div className="min-h-screen bg-surface-subtle">
      {/* Header */}
      <SocialHeader user={currentUser} />
      
      {/* Main Layout */}
      <div className="max-w-6xl mx-auto px-4 py-6">
        <div className="grid grid-cols-1 lg:grid-cols-4 gap-6">
          {/* Left Sidebar (hidden on mobile) */}
          <aside className="hidden lg:block">
            <UserProfileCard user={currentUser} />
            <TrendingTopics topics={trends} className="mt-4" />
          </aside>
          
          {/* Main Feed */}
          <main className="lg:col-span-2 space-y-4">
            {/* Post Composer */}
            <Card>
              <PostComposer 
                user={currentUser}
                onPost={handleNewPost}
              />
            </Card>
            
            {/* Feed with Infinite Scroll */}
            <InfiniteScroll
              loadMore={loadMorePosts}
              hasMore={hasMorePosts}
              loader={<PostSkeleton count={3} />}
            >
              {posts.map(post => (
                <PostCard
                  key={post.id}
                  post={post}
                  onLike={handleLike}
                  onComment={handleComment}
                  onShare={handleShare}
                />
              ))}
            </InfiniteScroll>
          </main>
          
          {/* Right Sidebar */}
          <aside className="hidden lg:block space-y-4">
            <SuggestedFollows users={suggestedUsers} />
            <SponsoredContent />
          </aside>
        </div>
      </div>
    </div>
  );
}
```

---

## 12.4 COMPOSIZIONE: HEALTHCARE PATIENT PORTAL

### Primitivi utilizzati:
- Layout: SecureAppShell, TabLayout
- Navigation: Sidebar, Tabs
- Content: Card, Timeline, DataTable
- Input: Form (HIPAA-compliant), DatePicker
- Feedback: Alert, Badge, Toast
- Data: VitalsDisplay, Chart
- Overlay: SecureModal

### Struttura composita:

```jsx
// HEALTHCARE PATIENT DASHBOARD
function PatientPortal() {
  return (
    <SecureAppShell 
      user={patient}
      sessionTimeout={900} // 15 min HIPAA requirement
      onSessionExpire={handleSessionExpire}
    >
      {/* Security Banner */}
      <SecurityBanner />
      
      {/* Navigation */}
      <HealthcareSidebar activeItem={activeNav} />
      
      {/* Main Content */}
      <MainContent>
        {/* Patient Header */}
        <PatientInfoCard patient={patient} />
        
        {/* Alerts */}
        {healthAlerts.length > 0 && (
          <Alert variant="warning" className="mb-6">
            <AlertTriangleIcon className="w-5 h-5" />
            You have {healthAlerts.length} health alerts requiring attention.
          </Alert>
        )}
        
        {/* Tab Navigation */}
        <Tabs value={activeTab} onChange={setActiveTab}>
          <TabList>
            <Tab value="overview">Overview</Tab>
            <Tab value="appointments">Appointments</Tab>
            <Tab value="medications">Medications</Tab>
            <Tab value="records">Medical Records</Tab>
            <Tab value="messages">Messages</Tab>
          </TabList>
          
          <TabPanels>
            {/* Overview Tab */}
            <TabPanel value="overview">
              <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
                {/* Vitals */}
                <Card>
                  <CardHeader title="Recent Vitals" />
                  <VitalsDisplay vitals={recentVitals} />
                </Card>
                
                {/* Upcoming Appointments */}
                <Card>
                  <CardHeader 
                    title="Upcoming Appointments"
                    action={<Button variant="outline" size="sm">Schedule New</Button>}
                  />
                  <AppointmentList appointments={upcomingAppointments} />
                </Card>
                
                {/* Medications */}
                <Card className="lg:col-span-2">
                  <CardHeader title="Current Medications" />
                  <MedicationList 
                    medications={currentMedications}
                    onRefillRequest={handleRefillRequest}
                  />
                </Card>
              </div>
            </TabPanel>
            
            {/* Other tabs... */}
          </TabPanels>
        </Tabs>
      </MainContent>
    </SecureAppShell>
  );
}
```

---

## 12.5 COMPOSIZIONE: FINTECH BANKING

### Primitivi utilizzati:
- Layout: SecureAppShell, Grid
- Content: Card, AccountCard, TransactionList
- Input: SecureForm, AmountInput
- Feedback: Alert, Toast, ConfirmationDialog
- Data: Chart, Timeline
- Action: Button, ConfirmationDialog
- Overlay: SecureModal

### Struttura composita:

```jsx
// FINTECH BANKING DASHBOARD
function BankingDashboard() {
  return (
    <SecureAppShell
      user={customer}
      mfaRequired
      sessionTimeout={300} // 5 min for banking
      onInactivity={showInactivityWarning}
    >
      {/* Security Header */}
      <BankingHeader 
        user={customer}
        lastLogin={lastLoginTime}
        securityLevel="verified"
      />
      
      {/* Main Content */}
      <div className="max-w-7xl mx-auto px-4 py-8">
        {/* Account Cards */}
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 mb-8">
          {accounts.map(account => (
            <AccountBalanceCard
              key={account.id}
              account={account}
              onTransfer={() => openTransferModal(account)}
              onDetails={() => openAccountDetails(account)}
            />
          ))}
          <AddAccountCard onClick={openAddAccountModal} />
        </div>
        
        {/* Quick Actions */}
        <div className="grid grid-cols-2 md:grid-cols-4 gap-4 mb-8">
          <QuickActionButton icon={SendIcon} label="Send Money" onClick={openSendMoney} />
          <QuickActionButton icon={CreditCardIcon} label="Pay Bills" onClick={openPayBills} />
          <QuickActionButton icon={QrCodeIcon} label="Scan to Pay" onClick={openQrScanner} />
          <QuickActionButton icon={PiggyBankIcon} label="Savings Goals" onClick={openSavingsGoals} />
        </div>
        
        {/* Main Grid */}
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
          {/* Transactions (2/3) */}
          <div className="lg:col-span-2">
            <Card>
              <CardHeader 
                title="Recent Transactions"
                action={<Button variant="ghost" size="sm">View All</Button>}
              />
              <TransactionList 
                transactions={recentTransactions}
                onTransactionClick={openTransactionDetail}
              />
            </Card>
          </div>
          
          {/* Sidebar (1/3) */}
          <div className="space-y-6">
            {/* Spending Chart */}
            <Card>
              <CardHeader title="Spending by Category" />
              <SpendingDonutChart data={spendingByCategory} />
            </Card>
            
            {/* Pending Actions */}
            <Card>
              <CardHeader title="Pending Actions" />
              <PendingActionsList actions={pendingActions} />
            </Card>
          </div>
        </div>
      </div>
      
      {/* Modals */}
      <TransferModal isOpen={transferModalOpen} onClose={closeTransferModal} />
      <ConfirmationDialog 
        isOpen={confirmDialogOpen}
        type="warning"
        title="Confirm Transaction"
        description="Are you sure you want to proceed with this transaction?"
        onConfirm={handleConfirmTransaction}
        onCancel={closeConfirmDialog}
      />
    </SecureAppShell>
  );
}
```

# ============================================================================
# SEZIONE 13: APPENDICE - QUICK REFERENCE
# ============================================================================

## 13.1 DESIGN TOKEN QUICK REFERENCE

### Colors:
```
text-text-default        → Testo principale
text-text-muted          → Testo secondario
text-text-subtle         → Testo terziario
text-text-inverse        → Testo su sfondo scuro
text-text-brand          → Testo brand

bg-surface-default       → Sfondo card/componenti
bg-surface-subtle        → Sfondo pagina
bg-surface-muted         → Sfondo interattivo
bg-surface-brand         → Sfondo brand

border-border-default    → Bordo standard
border-border-muted      → Bordo sottile
border-border-emphasis   → Bordo hover

text-feedback-error      → Rosso errore
text-feedback-success    → Verde successo
text-feedback-warning    → Giallo warning
text-feedback-info       → Blu info
```

### Spacing:
```
p-1   → 4px      gap-1   → 4px
p-2   → 8px      gap-2   → 8px
p-3   → 12px     gap-3   → 12px
p-4   → 16px     gap-4   → 16px
p-5   → 20px     gap-5   → 20px
p-6   → 24px     gap-6   → 24px
p-8   → 32px     gap-8   → 32px
p-10  → 40px     gap-10  → 40px
p-12  → 48px     gap-12  → 48px
```

### Typography:
```
text-xs    → 12px
text-sm    → 14px
text-base  → 16px
text-lg    → 18px
text-xl    → 20px
text-2xl   → 24px
text-3xl   → 30px
text-4xl   → 36px

font-normal   → 400
font-medium   → 500
font-semibold → 600
font-bold     → 700
```

### Border Radius:
```
rounded-sm   → 2px
rounded      → 4px
rounded-md   → 6px
rounded-lg   → 8px
rounded-xl   → 12px
rounded-2xl  → 16px
rounded-full → 9999px
```

### Z-Index:
```
z-dropdown       → 50
z-sticky         → 100
z-header         → 200
z-drawer         → 300
z-modal-backdrop → 400
z-modal          → 500
z-popover        → 600
z-tooltip        → 700
z-toast          → 800
z-command        → 900
```

---

## 13.2 COMPONENT CHECKLIST

### Prima di implementare un componente:

- [ ] **Accessibilità**: ARIA labels, keyboard navigation, focus management
- [ ] **Responsive**: Mobile-first, breakpoint testing
- [ ] **Stati**: Default, hover, focus, active, disabled, loading, error
- [ ] **Dark mode**: Colori semantici (tokens), non hardcoded
- [ ] **Animation**: Transizioni smooth, rispetta `prefers-reduced-motion`
- [ ] **Internazionalizzazione**: RTL support, text direction
- [ ] **Performance**: Lazy loading, virtualization per liste lunghe
- [ ] **Testing**: Unit test, accessibility test, visual regression

---

## 13.3 FILE STRUCTURE CONSIGLIATA

```
src/
├── components/
│   ├── primitives/           # Pattern base
│   │   ├── Button/
│   │   ├── Input/
│   │   ├── Card/
│   │   └── ...
│   ├── composite/            # Pattern composti
│   │   ├── DataTable/
│   │   ├── CommandPalette/
│   │   └── ...
│   └── domain/               # Pattern specifici per categoria
│       ├── ecommerce/
│       ├── saas/
│       ├── healthcare/
│       └── fintech/
├── styles/
│   ├── tokens.css            # Design tokens
│   ├── animations.css        # Keyframes
│   └── utilities.css         # Utility classes
└── lib/
    ├── cn.ts                 # className utility
    └── hooks/                # Custom hooks
```

---

# ============================================================================
# FINE CATALOGO UI PATTERN PRIMITIVI
# ============================================================================

"""
Questo catalogo fornisce una libreria completa di pattern UI primitivi
che possono essere combinati per costruire qualsiasi tipo di applicazione.

Ogni pattern:
1. È autocontenuto e riutilizzabile
2. Utilizza Design Tokens semantici
3. È accessibile (WCAG 2.1 AA)
4. È responsive by default
5. Supporta dark mode

Per aggiornamenti e nuovi pattern, riferirsi alla documentazione
del Design System di riferimento.

Versione: 1.0.0
Ultima modifica: 2025
"""
