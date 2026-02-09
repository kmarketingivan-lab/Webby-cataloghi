# CATALOGO-DESIGN-REFERENCE

CATALOGO-DESIGN-REFERENCE per Next.js 14 + Tailwind

Documento espanso con nuove sezioni

§ ICONOGRAPHY COMPLETE
Lucide Icons Catalog per Categoria
tsx
// Import pattern consigliato
import { 
  Home, User, Settings, // Navigazione
  Search, Filter, Download, // Azioni
  Check, X, AlertCircle, // Feedback
  Mail, Phone, MapPin, // Contatti
  Eye, EyeOff, Lock, // Sicurezza
  Sun, Moon, // Tema
  ChevronRight, ArrowUp, // Direzione
  Heart, Star, // Valutazione
  BarChart, PieChart // Dati
} from 'lucide-react';

// Esempio di implementazione
<Home className="w-5 h-5" />
Icon Sizing Standards
tsx
// Tailwind classes per dimensioni standard
const iconSizes = {
  xs: 'w-4 h-4',      // 16px - Testo piccolo
  sm: 'w-5 h-5',      // 20px - Testo body
  md: 'w-6 h-6',      // 24px - Intestazioni
  lg: 'w-8 h-8',      // 32px - Card icons
  xl: 'w-10 h-10',    // 40px - Hero sections
  xxl: 'w-16 h-16'    // 64px - Marketing
};

// Implementazione con clsx
import { clsx } from 'clsx';
import { AlertCircle } from 'lucide-react';

const IconAlert = ({ size = 'md' }) => (
  <AlertCircle className={clsx(
    'text-destructive',
    iconSizes[size]
  )} />
);
Icon + Text Alignment
tsx
// Pattern 1: Icona + testo in linea
<div className="flex items-center gap-2">
  <Mail className="w-4 h-4" />
  <span>email@example.com</span>
</div>

// Pattern 2: Button con icona
<button className="flex items-center justify-center gap-2 px-4 py-2">
  <Download className="w-4 h-4" />
  Download
</button>

// Pattern 3: Icona sopra testo
<div className="flex flex-col items-center gap-1">
  <User className="w-6 h-6" />
  <span className="text-sm">Profile</span>
</div>

// Allineamento verticale preciso
<div className="inline-flex items-baseline gap-1">
  <span>€</span>
  <span className="text-2xl font-bold">99</span>
  <span className="text-sm text-muted-foreground">/month</span>
</div>
Icon Buttons Patterns
tsx
// Base icon button
<button className="p-2 rounded-lg hover:bg-accent transition-colors">
  <Settings className="w-5 h-5" />
</button>

// Varianti con Tailwind
const iconButtonVariants = {
  default: "hover:bg-accent",
  destructive: "hover:bg-destructive hover:text-destructive-foreground",
  outline: "border hover:bg-accent",
  ghost: "hover:bg-accent",
  link: "text-primary underline-offset-4 hover:underline"
};

// Toggle icon button
const [isLiked, setIsLiked] = useState(false);
<button onClick={() => setIsLiked(!isLiked)}>
  {isLiked ? (
    <Heart className="w-5 h-5 fill-red-500 text-red-500" />
  ) : (
    <Heart className="w-5 h-5" />
  )}
</button>
Animated Icons (Loading, Success, Error)
tsx
// Loading spinner
const LoadingSpinner = () => (
  <div className="animate-spin rounded-full border-2 border-current border-t-transparent w-5 h-5"></div>
);

// Success animation
const SuccessIcon = () => (
  <div className="relative">
    <div className="w-6 h-6 bg-green-500 rounded-full animate-ping opacity-75"></div>
    <Check className="w-6 h-6 absolute inset-0 text-green-600" />
  </div>
);

// Error pulse
const ErrorIcon = () => (
  <AlertCircle className="w-6 h-6 text-destructive animate-pulse" />
);

// Progress indicator
const ProgressIcon = ({ progress }: { progress: number }) => (
  <svg className="w-8 h-8" viewBox="0 0 36 36">
    <path
      d="M18 2.0845
        a 15.9155 15.9155 0 0 1 0 31.831
        a 15.9155 15.9155 0 0 1 0 -31.831"
      fill="none"
      stroke="#E2E8F0"
      strokeWidth="3"
    />
    <path
      d="M18 2.0845
        a 15.9155 15.9155 0 0 1 0 31.831
        a 15.9155 15.9155 0 0 1 0 -31.831"
      fill="none"
      stroke="#3B82F6"
      strokeWidth="3"
      strokeDasharray={`${progress}, 100`}
      className="transition-all duration-300"
    />
  </svg>
);
Custom Icon Creation Guidelines
tsx
// 1. SVG ottimizzato per React
const CustomIcon = (props: React.SVGProps<SVGSVGElement>) => (
  <svg
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    strokeWidth="2"
    strokeLinecap="round"
    strokeLinejoin="round"
    {...props}
  >
    <path d="..."/>
  </svg>
);

// 2. Stile consistente
// - Usa strokeWidth="2" per consistenza
// - Mantieni viewBox="0 0 24 24"
// - Usa stroke="currentColor" per ereditare colore

// 3. Pattern di esportazione
// icons/index.ts
export { default as Logo } from './Logo';
export { default as CustomMarker } from './CustomMarker';

// 4. Icon wrapper per varianti
const IconWrapper = ({
  icon: Icon,
  size = 'md',
  className = ''
}) => (
  <Icon className={clsx(
    iconSizes[size],
    'text-current',
    className
  )} />
);
§ ILLUSTRATION STYLES
When to Use Illustrations

Onboarding flows: Spiegare concetti complessi

Empty states: Quando non ci sono dati

Error pages: Comunicare problemi in modo amichevole

Success screens: Confermare azioni completate

Loading states: Migliorare l'attesa

Educational content: Supportare l'apprendimento

Consistent Illustration Style
tsx
// Linee di stile da mantenere
const illustrationStyle = {
  strokeWidth: 2,
  colorPalette: ['#3B82F6', '#8B5CF6', '#10B981'],
  lineStyle: 'rounded',
  complexity: 'medium', // Non troppo dettagliate
  perspective: 'isometric' // Consistente
};

// Esempio di implementazione
<Illustration 
  className="max-w-[200px] mx-auto"
  colors={{
    primary: 'var(--color-primary)',
    secondary: 'var(--color-secondary)'
  }}
/>
Empty States Illustrations
tsx
// Componente EmptyState
const EmptyState = ({ type = 'default' }) => {
  const config = {
    default: {
      illustration: <ClipboardList className="w-24 h-24 text-muted" />,
      title: "Nessun dato disponibile",
      description: "Inizia aggiungendo il tuo primo elemento"
    },
    search: {
      illustration: <Search className="w-24 h-24 text-muted" />,
      title: "Nessun risultato trovato",
      description: "Prova con termini di ricerca diversi"
    },
    messages: {
      illustration: <MessageSquare className="w-24 h-24 text-muted" />,
      title: "Nessun messaggio",
      description: "I messaggi appariranno qui"
    }
  };

  return (
    <div className="text-center py-12">
      <div className="mx-auto mb-6">
        {config[type].illustration}
      </div>
      <h3 className="text-lg font-semibold mb-2">
        {config[type].title}
      </h3>
      <p className="text-muted-foreground">
        {config[type].description}
      </p>
    </div>
  );
};
Onboarding Illustrations
tsx
// Step-by-step illustrations
const OnboardingStep = ({ step, title, description }) => (
  <div className="text-center">
    <div className="relative mx-auto mb-8">
      {/* Illustration container */}
      <div className="w-48 h-48 mx-auto">
        <OnboardingIllustration step={step} />
      </div>
      {/* Step indicator */}
      <div className="absolute -top-2 -right-2 w-8 h-8 bg-primary text-primary-foreground rounded-full flex items-center justify-center">
        {step}
      </div>
    </div>
    <h3 className="text-xl font-bold mb-4">{title}</h3>
    <p className="text-muted-foreground">{description}</p>
  </div>
);

// Animated transitions
<AnimatePresence mode="wait">
  <motion.div
    key={currentStep}
    initial={{ opacity: 0, x: 20 }}
    animate={{ opacity: 1, x: 0 }}
    exit={{ opacity: 0, x: -20 }}
  >
    <OnboardingStep {...steps[currentStep]} />
  </motion.div>
</AnimatePresence>
Error Page Illustrations
tsx
// Error component con illustrazione
const ErrorIllustration = ({ code = 404 }) => {
  const errors = {
    404: {
      title: "Pagina non trovata",
      description: "La pagina che stai cercando non esiste o è stata spostata.",
      action: "Torna alla homepage"
    },
    500: {
      title: "Errore del server",
      description: "Si è verificato un errore interno. Riprova più tardi.",
      action: "Ricarica la pagina"
    },
    offline: {
      title: "Connessione assente",
      description: "Controlla la tua connessione internet e riprova.",
      action: "Riprova"
    }
  };

  return (
    <div className="min-h-[60vh] flex flex-col items-center justify-center p-4">
      <div className="relative mb-8">
        <AlertTriangle className="w-32 h-32 text-destructive" />
        {code && (
          <div className="absolute -bottom-2 -right-2 bg-background px-3 py-1 rounded-full border text-lg font-bold">
            {code}
          </div>
        )}
      </div>
      <h1 className="text-3xl font-bold mb-4">
        {errors[code]?.title || "Errore"}
      </h1>
      <p className="text-muted-foreground text-center max-w-md mb-8">
        {errors[code]?.description}
      </p>
      <Button>
        {errors[code]?.action}
      </Button>
    </div>
  );
};
Sources: unDraw, Humaaans, DrawKit
tsx
// Configurazione per illustrazioni esterne
const illustrationSources = {
  unDraw: {
    baseUrl: 'https://undraw.co/illustrations',
    style: 'modern, colorful',
    usage: 'Free with attribution',
    customization: 'Colori modificabili via URL'
  },
  humaaans: {
    baseUrl: 'https://www.humaaans.com',
    style: 'People-centric, mix-and-match',
    usage: 'Free for commercial use',
    customization: 'Componenti modulari'
  },
  drawKit: {
    baseUrl: 'https://www.drawkit.com',
    style: 'Professional, vector',
    usage: 'Free and paid options',
    customization: 'Alta qualità'
  }
};

// Esempio di implementazione unDraw
const UnDrawIllustration = ({ name, color = '#6B7280' }) => (
  <img
    src={`https://undraw.co/illustrations/${name}?color=${color.slice(1)}`}
    alt="Illustration"
    className="max-w-full h-auto"
  />
);

// Componente locale fallback
<Image
  src="/illustrations/error-404.svg"
  alt="404 Illustration"
  width={400}
  height={300}
  priority
  className="mx-auto"
/>
§ PHOTOGRAPHY GUIDELINES
Image Aspect Ratios Standard
tsx
// Tailwind aspect ratio classes
const aspectRatios = {
  square: 'aspect-square',      // 1:1 - Avatar, prodotti
  video: 'aspect-video',        // 16:9 - Hero, video
  portrait: 'aspect-[3/4]',     // 3:4 - Ritratti
  banner: 'aspect-[21/9]',      // 21:9 - Banner wide
  story: 'aspect-[9/16]',       // 9:16 - Storie mobile
  auto: 'aspect-auto'           // Naturale
};

// Implementazione
<div className={clsx(
  'overflow-hidden rounded-lg',
  aspectRatios.square
)}>
  <Image src={photo} fill alt="Product" />
</div>

// Grid responsivo con aspect ratio
<div className="grid grid-cols-2 md:grid-cols-3 gap-4">
  {photos.map((photo) => (
    <div key={photo.id} className="aspect-square relative">
      <Image
        src={photo.url}
        fill
        alt={photo.alt}
        className="object-cover"
      />
    </div>
  ))}
</div>
Placeholder Images
tsx
// Next.js Image con placeholder
<Image
  src={imageUrl}
  alt="Description"
  width={800}
  height={600}
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAYEBQYFBAYGBQYHBwYIChAKCgkJChQODwwQFxQYGBcUFhYaHSUfGhsjHBYWICwgIyYnKSopGR8tMC0oMCUoKSj/2wBDAQcHBwoIChMKChMoGhYaKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCj/wAARCAAIAAoDASIAAhEBAxEB/8QAFQABAQAAAAAAAAAAAAAAAAAAAAv/xAAUEAEAAAAAAAAAAAAAAAAAAAAA/8QAFQEBAQAAAAAAAAAAAAAAAAAAAAX/xAAUEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwCdABmX/9k="
/>

// SVG placeholder component
const ImagePlaceholder = ({ type = 'generic' }) => (
  <div className="bg-gradient-to-br from-muted to-muted/50 animate-pulse rounded-lg">
    <div className="flex items-center justify-center h-full">
      {type === 'user' && <User className="w-12 h-12 text-muted-foreground/50" />}
      {type === 'image' && <ImageIcon className="w-12 h-12 text-muted-foreground/50" />}
    </div>
  </div>
);

// Blur effect migliorato
const BlurredImage = ({ src, alt }) => {
  const [isLoading, setIsLoading] = useState(true);
  
  return (
    <div className="relative">
      {isLoading && (
        <div className="absolute inset-0 bg-gradient-to-br from-muted to-muted/50 animate-pulse" />
      )}
      <Image
        src={src}
        alt={alt}
        fill
        onLoadingComplete={() => setIsLoading(false)}
        className={clsx(
          'transition-opacity duration-300',
          isLoading ? 'opacity-0' : 'opacity-100'
        )}
      />
    </div>
  );
};
Image Optimization
tsx
// Next.js Image component ottimizzato
<Image
  src={imageUrl}
  alt="Descriptive alt text"
  width={1200}
  height={630}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  quality={85}
  priority={true} // Solo per LCP images
  loading="lazy"
  className="w-full h-auto"
/>

// Responsive images
const ResponsiveImage = ({ src, alt, priority = false }) => (
  <Image
    src={src}
    alt={alt}
    sizes="100vw"
    quality={80}
    priority={priority}
    width={0}
    height={0}
    className="w-full h-auto"
  />
);

// WebP fallback
<picture>
  <source srcSet="/image.webp" type="image/webp" />
  <source srcSet="/image.jpg" type="image/jpeg" />
  <img src="/image.jpg" alt="Fallback" />
</picture>
Avatar Defaults
tsx
// Componente Avatar
const Avatar = ({ src, alt, size = 'md', fallback }) => {
  const sizes = {
    sm: 'w-8 h-8',
    md: 'w-10 h-10',
    lg: 'w-12 h-12',
    xl: 'w-16 h-16'
  };

  if (!src) {
    return (
      <div className={clsx(
        sizes[size],
        'rounded-full bg-muted flex items-center justify-center'
      )}>
        <span className="text-sm font-medium">
          {fallback || <User className="w-1/2 h-1/2" />}
        </span>
      </div>
    );
  }

  return (
    <div className={clsx(sizes[size], 'relative rounded-full overflow-hidden')}>
      <Image
        src={src}
        alt={alt}
        fill
        className="object-cover"
      />
    </div>
  );
};

// Avatar group
const AvatarGroup = ({ users, max = 3 }) => (
  <div className="flex -space-x-2">
    {users.slice(0, max).map((user) => (
      <Avatar key={user.id} src={user.avatar} alt={user.name} size="sm" />
    ))}
    {users.length > max && (
      <div className="w-8 h-8 rounded-full bg-muted flex items-center justify-center text-xs border-2 border-background">
        +{users.length - max}
      </div>
    )}
  </div>
);
Hero Images
tsx
// Hero con overlay
const HeroSection = ({ image, title, subtitle }) => (
  <div className="relative h-[60vh] min-h-[400px]">
    <Image
      src={image}
      alt="Hero"
      fill
      priority
      className="object-cover"
    />
    <div className="absolute inset-0 bg-gradient-to-t from-background/80 via-background/40 to-transparent" />
    <div className="relative h-full flex items-center">
      <div className="container mx-auto px-4">
        <h1 className="text-4xl md:text-6xl font-bold text-white mb-4">
          {title}
        </h1>
        <p className="text-xl text-white/90 max-w-2xl">
          {subtitle}
        </p>
      </div>
    </div>
  </div>
);

// Hero con contenuto laterale
<div className="grid md:grid-cols-2 gap-8 items-center">
  <div>
    <h1>Title</h1>
    <p>Description</p>
  </div>
  <div className="relative aspect-square">
    <Image
      src="/hero-image.jpg"
      alt="Hero"
      fill
      className="object-contain"
    />
  </div>
</div>
Product Photography
tsx
// Product image gallery
const ProductGallery = ({ images }) => {
  const [selected, setSelected] = useState(0);
  
  return (
    <div className="grid grid-cols-1 lg:grid-cols-4 gap-4">
      {/* Thumbnails */}
      <div className="lg:col-span-1 order-last lg:order-first">
        <div className="flex lg:flex-col gap-2 overflow-x-auto">
          {images.map((img, idx) => (
            <button
              key={idx}
              onClick={() => setSelected(idx)}
              className={clsx(
                'relative aspect-square min-w-[80px] rounded-lg overflow-hidden',
                selected === idx && 'ring-2 ring-primary'
              )}
            >
              <Image
                src={img.thumbnail}
                alt={`Thumbnail ${idx + 1}`}
                fill
                className="object-cover"
              />
            </button>
          ))}
        </div>
      </div>
      
      {/* Main image */}
      <div className="lg:col-span-3 relative aspect-square">
        <Image
          src={images[selected].full}
          alt="Product"
          fill
          className="object-contain"
          priority
        />
      </div>
    </div>
  );
};

// Product image con zoom
const ZoomableImage = ({ src, alt }) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [isZoomed, setIsZoomed] = useState(false);
  
  return (
    <div className="relative overflow-hidden rounded-lg">
      <div
        className="cursor-zoom-in"
        onMouseMove={(e) => {
          const rect = e.currentTarget.getBoundingClientRect();
          const x = ((e.clientX - rect.left) / rect.width) * 100;
          const y = ((e.clientY - rect.top) / rect.height) * 100;
          setPosition({ x, y });
        }}
        onClick={() => setIsZoomed(!isZoomed)}
      >
        <Image
          src={src}
          alt={alt}
          width={800}
          height={800}
          className={clsx(
            'transition-transform duration-300',
            isZoomed && 'scale-150'
          )}
          style={
            isZoomed
              ? { transformOrigin: `${position.x}% ${position.y}%` }
              : {}
          }
        />
      </div>
    </div>
  );
};
§ VISUAL HIERARCHY
Size Hierarchy
tsx
// Scala tipografica gerarchica
const typographyScale = {
  display: 'text-5xl md:text-7xl font-bold',      // 48-72px
  h1: 'text-4xl md:text-5xl font-bold',          // 36-48px
  h2: 'text-3xl md:text-4xl font-semibold',      // 30-36px
  h3: 'text-2xl font-semibold',                  // 24px
  h4: 'text-xl font-semibold',                   // 20px
  h5: 'text-lg font-medium',                     // 18px
  body: 'text-base',                             // 16px
  small: 'text-sm',                              // 14px
  xs: 'text-xs'                                  // 12px
};

// Gerarchia dei componenti
const componentHierarchy = {
  primary: 'py-3 px-6 text-lg font-semibold',    // Primary CTA
  secondary: 'py-2 px-4 font-medium',            // Secondary actions
  tertiary: 'text-sm underline'                  // Tertiary links
};
Color Hierarchy
tsx
// Scala di importanza del colore
const colorHierarchy = {
  // Livello 1: Azioni primarie e testo importante
  primary: {
    text: 'text-primary',
    bg: 'bg-primary',
    border: 'border-primary'
  },
  // Livello 2: Azioni secondarie e testo corpo
  secondary: {
    text: 'text-secondary',
    bg: 'bg-secondary',
    border: 'border-secondary'
  },
  // Livello 3: Testo meno importante
  muted: {
    text: 'text-muted-foreground',
    bg: 'bg-muted',
    border: 'border-muted'
  },
  // Livello 4: Stati di errore/successo
  semantic: {
    success: 'text-green-600',
    error: 'text-destructive',
    warning: 'text-yellow-600',
    info: 'text-blue-600'
  }
};

// Applicazione
const PriorityIndicator = ({ priority = 'low' }) => {
  const colors = {
    high: 'bg-destructive text-destructive-foreground',
    medium: 'bg-yellow-500 text-yellow-950',
    low: 'bg-green-500 text-green-950'
  };
  
  return (
    <span className={clsx(
      'px-2 py-1 rounded-full text-xs font-medium',
      colors[priority]
    )}>
      {priority}
    </span>
  );
};
Weight Hierarchy
tsx
// Gerarchia del peso del font
const fontWeightHierarchy = {
  bold: 'font-bold',        // Titoli principali (700)
  semibold: 'font-semibold', // Sottotitoli (600)
  medium: 'font-medium',    // Enfasi nel testo (500)
  normal: 'font-normal',    // Testo corpo (400)
  light: 'font-light'       // Testo secondario (300)
};

// Esempio di applicazione
<article>
  <h1 className="text-3xl font-bold mb-4">Main Title</h1>
  <h2 className="text-xl font-semibold mb-2">Section Title</h2>
  <p className="font-normal mb-2">Main content with <span className="font-medium">important parts</span> emphasized.</p>
  <p className="font-light text-muted-foreground">Supporting or secondary information.</p>
</article>
Whitespace Usage
tsx
// Sistema di spaziatura
const spacingScale = {
  xs: 'space-y-1',      // 4px - Elementi strettamente correlati
  sm: 'space-y-2',      // 8px - Elementi correlati
  md: 'space-y-4',      // 16px - Sezioni interne
  lg: 'space-y-6',      // 24px - Sezioni principali
  xl: 'space-y-8',      // 32px - Gruppi di sezioni
  xxl: 'space-y-12'     // 48px - Separazione major
};

// Layout con gerarchia spaziale
const CardLayout = () => (
  <div className="space-y-6"> {/* Spaziatura tra sezioni */}
    <header className="space-y-2"> {/* Spaziatura interna */}
      <h2 className="text-2xl font-bold">Card Title</h2>
      <p className="text-muted-foreground">Description</p>
    </header>
    
    <div className="space-y-4"> {/* Contenuto */}
      <div className="space-y-1"> {/* Elementi correlati */}
        <label className="text-sm font-medium">Field 1</label>
        <input className="w-full" />
      </div>
    </div>
    
    <footer className="flex gap-2"> {/* Elementi inline */}
      <Button>Action 1</Button>
      <Button variant="outline">Action 2</Button>
    </footer>
  </div>
);
Z-pattern and F-pattern
tsx
// Layout Z-pattern per landing pages
const ZPatternLayout = () => (
  <div className="space-y-16">
    {/* Row 1 */}
    <div className="grid md:grid-cols-2 gap-8 items-center">
      <div>
        <h1 className="text-4xl font-bold mb-4">Primary Headline</h1>
        <p className="text-lg">Supporting information that leads eye to next point.</p>
      </div>
      <div className="relative aspect-video">
        <Image src="/feature1.jpg" alt="Feature 1" fill />
      </div>
    </div>
    
    {/* Row 2 (inverted) */}
    <div className="grid md:grid-cols-2 gap-8 items-center">
      <div className="md:order-2">
        <h2 className="text-3xl font-bold mb-4">Secondary Headline</h2>
        <p className="text-lg">Continuing the natural reading flow.</p>
      </div>
      <div className="relative aspect-video md:order-1">
        <Image src="/feature2.jpg" alt="Feature 2" fill />
      </div>
    </div>
    
    {/* CTA finale */}
    <div className="text-center">
      <h3 className="text-2xl font-bold mb-4">Final Call to Action</h3>
      <Button size="lg">Get Started</Button>
    </div>
  </div>
);

// Layout F-pattern per contenuti testuali
const FPatternLayout = () => (
  <div className="max-w-4xl mx-auto">
    {/* Titolo principale */}
    <h1 className="text-4xl font-bold mb-6">Main Article Title</h1>
    
    {/* Introduzione */}
    <p className="text-xl mb-8">Important introductory paragraph that sets the context.</p>
    
    {/* Sottosezioni */}
    <div className="space-y-8">
      <section>
        <h2 className="text-2xl font-bold mb-4">First Major Point</h2>
        <p className="mb-2">First line of the section draws attention.</p>
        <p className="text-muted-foreground">Supporting details that might be skimmed.</p>
      </section>
      
      <section>
        <h2 className="text-2xl font-bold mb-4">Second Major Point</h2>
        <p className="mb-2">Key information at the start.</p>
        <p className="text-muted-foreground">Additional context and details.</p>
      </section>
      
      {/* Bullet points per scansionabilità */}
      <section>
        <h2 className="text-2xl font-bold mb-4">Key Takeaways</h2>
        <ul className="space-y-2">
          <li className="flex items-start gap-2">
            <Check className="w-5 h-5 mt-0.5 text-green-600" />
            <span>Concise, scannable point</span>
          </li>
          <li className="flex items-start gap-2">
            <Check className="w-5 h-5 mt-0.5 text-green-600" />
            <span>Another important point</span>
          </li>
        </ul>
      </section>
    </div>
  </div>
);
§ BRAND APPLICATION
Logo Usage Guidelines
tsx
// Logo component con varianti
const Logo = ({ variant = 'default', size = 'md' }) => {
  const sizes = {
    sm: 'h-8',
    md: 'h-10',
    lg: 'h-12',
    xl: 'h-16'
  };
  
  const variants = {
    default: '/logo.svg',
    dark: '/logo-dark.svg',
    light: '/logo-light.svg',
    icon: '/logo-icon.svg',
    wordmark: '/logo-wordmark.svg'
  };
  
  return (
    <Image
      src={variants[variant]}
      alt="Company Name"
      width={0}
      height={0}
      className={`${sizes[size]} w-auto`}
      priority
    />
  );
};

// Area di sicurezza
<div className="p-4 bg-gray-100 rounded-lg">
  <div className="border-2 border-dashed border-gray-300 p-8 flex items-center justify-center">
    <Logo size="lg" />
  </div>
  <p className="text-sm text-center mt-2">Min. clear space: 50% logo height</p>
</div>
Color Application Rules
tsx
// Sistema di colore del brand
const brandColors = {
  primary: {
    50: '#eff6ff',
    100: '#dbeafe',
    // ... scale completa
    900: '#1e3a8a',
    DEFAULT: '#3b82f6' // Usare DEFAULT per Tailwind
  },
  accent: {
    50: '#fdf4ff',
    DEFAULT: '#d946ef'
  },
  neutral: {
    DEFAULT: '#6b7280'
  }
};

// Regole di applicazione
const colorRules = {
  // Testo
  text: {
    primary: 'text-primary-600',
    onPrimary: 'text-white',
    onDark: 'text-gray-100'
  },
  
  // Backgrounds
  bg: {
    primary: 'bg-primary-600',
    primarySubtle: 'bg-primary-50',
    dark: 'bg-gray-900'
  },
  
  // Borders
  border: {
    primary: 'border-primary-300',
    focus: 'border-primary-500 ring-primary-500'
  }
};

// Esempio di implementazione
<button className="bg-primary-600 text-white hover:bg-primary-700 focus:ring-2 focus:ring-primary-500">
  Primary Action
</button>
Typography in Branding
tsx
// Sistema tipografico del brand
const brandTypography = {
  fontFamilies: {
    heading: 'var(--font-heading)',      // Inter, bold
    body: 'var(--font-body)',            // Inter, regular
    mono: 'var(--font-mono)'             // JetBrains Mono
  },
  
  scales: {
    marketing: {
      display: 'text-5xl md:text-7xl font-heading font-bold',
      h1: 'text-4xl md:text-5xl font-heading font-bold',
      // ...
    },
    product: {
      h1: 'text-3xl font-heading font-semibold',
      h2: 'text-2xl font-heading font-semibold',
      body: 'text-base font-body'
    }
  },
  
  // Line heights
  lineHeights: {
    tight: 'leading-tight',      // 1.25
    normal: 'leading-normal',    // 1.5
    relaxed: 'leading-relaxed'   // 1.75
  }
};

// Componente tipografico
const BrandText = ({ type = 'body', children }) => (
  <div className={brandTypography.scales.product[type]}>
    {children}
  </div>
);
Tone Consistency
tsx
// Voice and tone guidelines
const brandVoice = {
  // Livelli di formalità
  formal: {
    greeting: "Dear User,",
    closing: "Sincerely,",
    words: ["please", "thank you", "appreciate"]
  },
  casual: {
    greeting: "Hi there,",
    closing: "Best,",
    words: ["hey", "awesome", "got it"]
  },
  
  // Messaggi standardizzati
  messages: {
    success: "Great! {action} completed successfully.",
    error: "Oops! Something went wrong. {solution}",
    loading: "Hang tight, we're working on it...",
    empty: "Nothing here yet. {suggestion}"
  }
};

// Hook per messaggi consistenti
const useBrandMessage = () => {
  const formatMessage = (key, variables = {}) => {
    let message = brandVoice.messages[key];
    Object.entries(variables).forEach(([k