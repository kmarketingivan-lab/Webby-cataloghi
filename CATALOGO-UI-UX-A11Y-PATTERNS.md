# CATALOGO-UI-UX-A11Y-PATTERNS

Catalogo UI/UX/A11Y Patterns per Next.js 14 - Espansione Significativa
§ SCREEN READER SUPPORT
ARIA Roles Catalog Completo
tsx
// Next.js Component with ARIA Roles
export const ARIAComponent = () => {
  return (
    <div role="application" aria-label="Calcolatrice">
      <div role="banner">Intestazione</div>
      <nav role="navigation" aria-label="Principale">
        {/* Menu */}
      </nav>
      <main role="main">
        <div role="region" aria-label="Risultato">
          <output role="status">0</output>
        </div>
        <div role="group" aria-label="Tastiera numerica">
          <button role="button" aria-label="Uno">1</button>
        </div>
      </main>
      <aside role="complementary" aria-label="Storico">
        {/* Contenuto complementare */}
      </aside>
      <footer role="contentinfo">
        {/* Footer */}
      </footer>
    </div>
  );
};
ARIA States e Properties
tsx
export const AriaStateComponent = () => {
  const [expanded, setExpanded] = useState(false);
  const [pressed, setPressed] = useState(false);
  const [selected, setSelected] = useState(false);
  const [hidden, setHidden] = useState(false);

  return (
    <div>
      <button
        aria-expanded={expanded}
        aria-controls="expandable-content"
        onClick={() => setExpanded(!expanded)}
      >
        {expanded ? 'Comprimi' : 'Espandi'}
      </button>
      
      <div 
        id="expandable-content"
        aria-hidden={!expanded}
        role="region"
      >
        Contenuto espandibile
      </div>

      <button
        aria-pressed={pressed}
        onClick={() => setPressed(!pressed)}
        role="switch"
        aria-checked={pressed}
      >
        Interruttore: {pressed ? 'ON' : 'OFF'}
      </button>

      <div
        role="tablist"
        aria-label="Schede documento"
      >
        <button
          role="tab"
          aria-selected={selected}
          aria-controls="panel-1"
          onClick={() => setSelected(true)}
        >
          Scheda 1
        </button>
      </div>
    </div>
  );
};
Live Regions
tsx
export const LiveRegionComponent = () => {
  const [message, setMessage] = useState('');
  const [urgentMessage, setUrgentMessage] = useState('');
  
  const showPoliteMessage = () => {
    setMessage('Dati caricati con successo ' + new Date().toLocaleTimeString());
  };

  const showAssertiveMessage = () => {
    setUrgentMessage('Errore di connessione! Riprova.');
  };

  return (
    <div>
      <button onClick={showPoliteMessage}>
        Carica dati (notifica gentile)
      </button>
      
      <button onClick={showAssertiveMessage}>
        Simula errore (notifica urgente)
      </button>

      {/* Live Region Polita */}
      <div
        aria-live="polite"
        aria-atomic="true"
        className="sr-only"
        role="status"
      >
        {message}
      </div>

      {/* Live Region Assertiva */}
      <div
        aria-live="assertive"
        aria-atomic="true"
        aria-relevant="additions text"
        className="sr-only"
        role="alert"
      >
        {urgentMessage}
      </div>

      {/* Timer Live Region */}
      <div
        aria-live="polite"
        aria-atomic="false"
        role="timer"
        aria-label="Contatore secondi"
      >
        {/* Contenuto aggiornato dinamicamente */}
      </div>
    </div>
  );
};
Landmark Roles
tsx
export const LandmarkLayout = () => {
  return (
    <div role="document">
      <header role="banner">
        <h1>Mio Sito</h1>
        <nav 
          role="navigation" 
          aria-label="Navigazione principale"
        >
          {/* Menu principale */}
        </nav>
      </header>

      <main role="main" id="main-content">
        <article role="article">
          <header>
            <h2>Titolo Articolo</h2>
          </header>
          <section aria-labelledby="section1">
            <h3 id="section1">Sezione 1</h3>
            {/* Contenuto */}
          </section>
        </article>

        <aside role="complementary" aria-label="Informazioni correlate">
          {/* Contenuto correlato */}
        </aside>
      </main>

      <section 
        role="region" 
        aria-label="Statistiche"
        aria-describedby="stats-desc"
      >
        <p id="stats-desc">Statistiche aggiornate in tempo reale</p>
        {/* Statistiche */}
      </section>

      <form role="search" aria-label="Cerca nel sito">
        {/* Form di ricerca */}
      </form>

      <footer role="contentinfo">
        <nav role="navigation" aria-label="Footer">
          {/* Link footer */}
        </nav>
      </footer>
    </div>
  );
};
Reading Order
tsx
export const ReadingOrderComponent = () => {
  return (
    <div 
      role="document"
      aria-label="Pagina prodotti"
      style={{ display: 'flex' }}
    >
      {/* Contenuto in ordine di lettura logico */}
      <div 
        role="region"
        aria-labelledby="main-heading"
        style={{ order: 2 }}
      >
        <h1 id="main-heading">Prodotti in Evidenza</h1>
        {/* Contenuto principale */}
      </div>
      
      {/* Sidebar con tabindex per ordine personalizzato */}
      <aside 
        role="complementary"
        aria-label="Filtri"
        style={{ order: 1 }}
        tabIndex={-1}
      >
        <h2>Filtri</h2>
        {/* Filtri sidebar */}
      </aside>

      {/* Skip link per navigazione rapida */}
      <a 
        href="#main-heading" 
        className="skip-link"
        aria-label="Salta al contenuto principale"
      >
        Vai al contenuto principale
      </a>

      {/* Uso appropriato di tabindex per ordine personalizzato */}
      <div>
        <button tabIndex={1}>Primo elemento focus</button>
        <button tabIndex={2}>Secondo elemento focus</button>
        <div tabIndex={0} role="region">Area focusabile</div>
      </div>
    </div>
  );
};
§ KEYBOARD NAVIGATION
Focus Management
tsx
import { useEffect, useRef } from 'react';

export const FocusManagementComponent = () => {
  const mainRef = useRef<HTMLDivElement>(null);
  const firstInputRef = useRef<HTMLInputElement>(null);
  const lastInputRef = useRef<HTMLInputElement>(null);

  // Focus iniziale
  useEffect(() => {
    if (firstInputRef.current) {
      firstInputRef.current.focus();
    }
  }, []);

  const handleKeyDown = (e: React.KeyboardEvent) => {
    // Gestione Tab per contenuto circoscritto
    if (e.key === 'Tab') {
      if (!e.shiftKey && document.activeElement === lastInputRef.current) {
        e.preventDefault();
        firstInputRef.current?.focus();
      } else if (e.shiftKey && document.activeElement === firstInputRef.current) {
        e.preventDefault();
        lastInputRef.current?.focus();
      }
    }

    // Gestione frecce
    if (e.key === 'ArrowDown' || e.key === 'ArrowRight') {
      // Sposta focus verso il basso/destra
      const focusableElements = mainRef.current?.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      // Implementazione logica focus
    }
  };

  return (
    <div 
      ref={mainRef}
      onKeyDown={handleKeyDown}
      role="application"
      aria-label="Gestione focus"
    >
      <button onClick={() => mainRef.current?.focus()}>
        Focus sull'area principale
      </button>

      <input
        ref={firstInputRef}
        aria-label="Primo campo"
        placeholder="Primo campo"
      />

      <input
        placeholder="Campo intermedio"
      />

      <input
        ref={lastInputRef}
        aria-label="Ultimo campo"
        placeholder="Ultimo campo"
      />

      {/* Programmatic focus */}
      <button 
        onClick={() => {
          document.getElementById('target-element')?.focus();
          document.getElementById('target-element')?.scrollIntoView({
            behavior: 'smooth',
            block: 'center'
          });
        }}
      >
        Vai all'elemento specifico
      </button>
    </div>
  );
};
Focus Trap (Modals)
tsx
export const ModalWithFocusTrap = () => {
  const [isOpen, setIsOpen] = useState(false);
  const modalRef = useRef<HTMLDivElement>(null);
  const firstFocusableRef = useRef<HTMLButtonElement>(null);
  const lastFocusableRef = useRef<HTMLButtonElement>(null);
  const closeButtonRef = useRef<HTMLButtonElement>(null);

  // Apri modal
  const openModal = () => {
    setIsOpen(true);
    // Salva elemento attivo prima di aprire la modal
    const previousActiveElement = document.activeElement as HTMLElement;
    previousActiveElement.dataset.previousFocus = 'true';
  };

  // Chiudi modal
  const closeModal = () => {
    setIsOpen(false);
    // Ripristina focus all'elemento precedente
    const previousElement = document.querySelector('[data-previous-focus]') as HTMLElement;
    previousElement?.focus();
    delete previousElement?.dataset.previousFocus;
  };

  // Trap focus nella modal
  useEffect(() => {
    if (!isOpen || !modalRef.current) return;

    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === 'Escape') {
        closeModal();
        return;
      }

      if (e.key === 'Tab') {
        const focusableElements = modalRef.current?.querySelectorAll(
          'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
        );

        if (focusableElements.length === 0) return;

        const firstElement = focusableElements[0] as HTMLElement;
        const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement;

        if (e.shiftKey && document.activeElement === firstElement) {
          e.preventDefault();
          lastElement.focus();
        } else if (!e.shiftKey && document.activeElement === lastElement) {
          e.preventDefault();
          firstElement.focus();
        }
      }
    };

    document.addEventListener('keydown', handleKeyDown);
    
    // Focus iniziale sulla modal
    closeButtonRef.current?.focus();

    return () => {
      document.removeEventListener('keydown', handleKeyDown);
    };
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <>
      {/* Backdrop */}
      <div 
        role="presentation"
        className="modal-backdrop"
        onClick={closeModal}
        aria-hidden="true"
      />
      
      {/* Modal */}
      <div
        ref={modalRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        aria-describedby="modal-description"
        className="modal"
        tabIndex={-1}
      >
        <h2 id="modal-title">Titolo Modal</h2>
        
        <p id="modal-description">
          Descrizione della modal per screen reader
        </p>

        <div className="modal-content">
          {/* Contenuto della modal */}
          <input type="text" aria-label="Campo testo" />
          <select aria-label="Selezione opzioni">
            <option value="1">Opzione 1</option>
            <option value="2">Opzione 2</option>
          </select>
        </div>

        <div className="modal-actions">
          <button 
            ref={firstFocusableRef}
            onClick={closeModal}
          >
            Annulla
          </button>
          
          <button 
            ref={lastFocusableRef}
            onClick={() => {/* Azione */}}
          >
            Conferma
          </button>

          <button
            ref={closeButtonRef}
            onClick={closeModal}
            aria-label="Chiudi modal"
            className="close-button"
          >
            ×
          </button>
        </div>
      </div>
    </>
  );
};
Roving Tabindex
tsx
export const RovingTabindexComponent = () => {
  const [activeIndex, setActiveIndex] = useState(0);
  const itemsRef = useRef<(HTMLButtonElement | null)[]>([]);

  const items = ['Elemento 1', 'Elemento 2', 'Elemento 3', 'Elemento 4'];

  const handleKeyDown = (e: React.KeyboardEvent, index: number) => {
    switch (e.key) {
      case 'ArrowRight':
      case 'ArrowDown':
        e.preventDefault();
        const nextIndex = (index + 1) % items.length;
        setActiveIndex(nextIndex);
        itemsRef.current[nextIndex]?.focus();
        break;
      
      case 'ArrowLeft':
      case 'ArrowUp':
        e.preventDefault();
        const prevIndex = (index - 1 + items.length) % items.length;
        setActiveIndex(prevIndex);
        itemsRef.current[prevIndex]?.focus();
        break;
      
      case 'Home':
        e.preventDefault();
        setActiveIndex(0);
        itemsRef.current[0]?.focus();
        break;
      
      case 'End':
        e.preventDefault();
        setActiveIndex(items.length - 1);
        itemsRef.current[items.length - 1]?.focus();
        break;
      
      case 'Enter':
      case ' ':
        e.preventDefault();
        // Azione sull'elemento
        break;
    }
  };

  return (
    <div 
      role="toolbar" 
      aria-label="Barra strumenti"
      aria-orientation="horizontal"
    >
      {items.map((item, index) => (
        <button
          key={index}
          ref={(el) => (itemsRef.current[index] = el)}
          role={index === 0 ? 'tab' : undefined}
          tabIndex={activeIndex === index ? 0 : -1}
          aria-selected={activeIndex === index}
          onKeyDown={(e) => handleKeyDown(e, index)}
          onClick={() => setActiveIndex(index)}
          aria-label={item}
        >
          {item}
        </button>
      ))}
    </div>
  );
};
Skip Links
tsx
export const SkipLinksComponent = () => {
  return (
    <>
      {/* Skip Links Collection */}
      <nav aria-label="Link di accesso rapido">
        <ul className="skip-links">
          <li>
            <a 
              href="#main-content" 
              className="skip-link"
              aria-label="Salta al contenuto principale"
            >
              Vai al contenuto principale
            </a>
          </li>
          <li>
            <a 
              href="#main-navigation" 
              className="skip-link"
              aria-label="Salta alla navigazione principale"
            >
              Vai alla navigazione
            </a>
          </li>
          <li>
            <a 
              href="#search" 
              className="skip-link"
              aria-label="Salta al campo di ricerca"
            >
              Vai alla ricerca
            </a>
          </li>
          <li>
            <a 
              href="#footer" 
              className="skip-link"
              aria-label="Salta al footer"
            >
              Vai al footer
            </a>
          </li>
        </ul>
      </nav>

      <style jsx>{`
        .skip-links {
          position: absolute;
          top: -40px;
          left: 0;
          right: 0;
          z-index: 1000;
          list-style: none;
          padding: 0;
          margin: 0;
        }

        .skip-link {
          position: absolute;
          top: 0;
          left: 0;
          background: #005a9c;
          color: white;
          padding: 12px 24px;
          text-decoration: none;
          font-weight: bold;
          border-radius: 0 0 4px 0;
          transform: translateY(-100%);
          transition: transform 0.3s ease;
        }

        .skip-link:focus {
          transform: translateY(0);
          outline: 3px solid #ffbf47;
          outline-offset: 2px;
        }

        .skip-link:hover {
          background: #003d73;
        }
      `}</style>
    </>
  );
};
Keyboard Shortcuts
tsx
export const KeyboardShortcutsComponent = () => {
  useEffect(() => {
    const handleGlobalShortcuts = (e: KeyboardEvent) => {
      // Ctrl + / per focus search
      if ((e.ctrlKey || e.metaKey) && e.key === '/') {
        e.preventDefault();
        document.getElementById('global-search')?.focus();
      }

      // ? per help
      if (e.key === '?' && e.shiftKey) {
        e.preventDefault();
        document.getElementById('help-dialog')?.focus();
      }
    };

    document.addEventListener('keydown', handleGlobalShortcuts);
    return () => document.removeEventListener('keydown', handleGlobalShortcuts);
  }, []);

  return (
    <div>
      {/* Search con shortcut */}
      <input
        id="global-search"
        type="search"
        aria-label="Cerca nel sito (Ctrl + /)"
        placeholder="Cerca (Ctrl + /)"
      />

      {/* Help dialog */}
      <dialog id="help-dialog" role="dialog">
        <h2>Shortcuts da tastiera</h2>
        <dl>
          <dt>Ctrl + /</dt>
          <dd>Focus campo di ricerca</dd>
          
          <dt>Shift + ?</dt>
          <dd>Apri help</dd>
          
          <dt>Esc</dt>
          <dd>Chiudi modali</dd>
        </dl>
      </dialog>

      {/* Shortcut personalizzato per componente */}
      <div 
        role="application"
        aria-label="Editor testo con shortcuts"
        onKeyDown={(e) => {
          if ((e.ctrlKey || e.metaKey) && e.key === 'b') {
            e.preventDefault();
            // Attiva/Disattiva grassetto
          }
          if ((e.ctrlKey || e.metaKey) && e.key === 'i') {
            e.preventDefault();
            // Attiva/Disattiva corsivo
          }
        }}
      >
        {/* Editor */}
      </div>

      {/* Annuncio shortcuts */}
      <div 
        role="region" 
        aria-label="Informazioni shortcuts"
        className="sr-only"
      >
        Usa Ctrl + / per cercare, Shift + ? per help
      </div>
    </div>
  );
};
Focus Visible Styles
tsx
export const FocusVisibleComponent = () => {
  return (
    <>
      <div className="container">
        {/* Focus styles per diversi stati */}
        <button className="btn-focus">
          Bottone con focus visibile
        </button>

        <input 
          type="text" 
          className="input-focus"
          aria-label="Campo con focus visibile"
        />

        <a href="#" className="link-focus">
          Link con focus visibile
        </a>

        {/* Custom focus ring */}
        <button className="custom-focus-ring">
          Custom focus ring
        </button>

        {/* Focus per utenti keyboard only */}
        <div className="keyboard-focus">
          <button className="btn-keyboard-focus">
            Solo focus tastiera
          </button>
        </div>
      </div>

      <style jsx>{`
        /* Focus styles base */
        .btn-focus:focus-visible,
        .input-focus:focus-visible,
        .link-focus:focus-visible {
          outline: 3px solid #4d90fe;
          outline-offset: 2px;
          box-shadow: 0 0 0 3px rgba(77, 144, 254, 0.3);
        }

        /* Supporto per browser senza :focus-visible */
        .btn-focus:focus:not(:focus-visible) {
          outline: none;
        }

        /* Custom focus ring */
        .custom-focus-ring {
          position: relative;
        }

        .custom-focus-ring:focus-visible::before {
          content: '';
          position: absolute;
          top: -4px;
          left: -4px;
          right: -4px;
          bottom: -4px;
          border: 2px solid #ff6b6b;
          border-radius: 6px;
          pointer-events: none;
        }

        /* Focus per tastiera vs mouse */
        .keyboard-focus {
          --focus-ring: 0 0 0 3px rgba(0, 123, 255, 0.5);
        }

        .btn-keyboard-focus:focus {
          box-shadow: var(--focus-ring);
        }

        .btn-keyboard-focus:focus:not(:focus-visible) {
          box-shadow: none;
        }

        /* High contrast support */
        @media (prefers-contrast: high) {
          .btn-focus:focus-visible {
            outline: 3px solid transparent;
            outline-offset: 3px;
            box-shadow: 0 0 0 4px CanvasText;
          }
        }

        /* Reduced motion */
        @media (prefers-reduced-motion: reduce) {
          .btn-focus:focus-visible {
            transition: none;
          }
        }
      `}</style>
    </>
  );
};
§ FORMS ACCESSIBILITY
Label Association
tsx
export const FormLabelsComponent = () => {
  return (
    <form aria-label="Modulo di registrazione">
      {/* Metodo 1: Label esplicito */}
      <div className="form-group">
        <label htmlFor="name" className="required">
          Nome completo
        </label>
        <input
          id="name"
          type="text"
          name="name"
          required
          aria-describedby="name-help"
        />
        <span id="name-help" className="help-text">
          Inserisci il tuo nome completo
        </span>
      </div>

      {/* Metodo 2: aria-label */}
      <div className="form-group">
        <input
          type="search"
          aria-label="Cerca prodotti"
          placeholder="Cerca..."
        />
      </div>

      {/* Metodo 3: aria-labelledby */}
      <div className="form-group">
        <span id="email-label">
          Email
          <span className="sr-only"> (obbligatorio)</span>
        </span>
        <input
          id="email"
          type="email"
          aria-labelledby="email-label"
          aria-required="true"
        />
      </div>

      {/* Gruppo di radio buttons */}
      <fieldset>
        <legend>
          Metodo di pagamento preferito
          <span className="sr-only"> (seleziona un'opzione)</span>
        </legend>
        
        <div role="radiogroup" aria-labelledby="payment-label">
          <label>
            <input type="radio" name="payment" value="card" />
            Carta di credito
          </label>
          <label>
            <input type="radio" name="payment" value="paypal" />
            PayPal
          </label>
        </div>
      </fieldset>

      {/* Label per select */}
      <div className="form-group">
        <label htmlFor="country">Paese di residenza</label>
        <select id="country" name="country">
          <option value="">Seleziona un paese</option>
          <option value="IT">Italia</option>
          <option value="FR">Francia</option>
        </select>
      </div>

      {/* Label nascosta per screen reader */}
      <div className="form-group">
        <span id="phone-label" className="sr-only">
          Numero di telefono
        </span>
        <input
          type="tel"
          aria-labelledby="phone-label"
          placeholder="Telefono"
        />
      </div>

      {/* Combobox con label */}
      <div className="form-group">
        <label htmlFor="fruit">Frutta preferita</label>
        <input
          id="fruit"
          type="text"
          role="combobox"
          aria-autocomplete="list"
          aria-controls="fruit-list"
          aria-expanded="false"
        />
        <ul id="fruit-list" role="listbox">
          <li role="option">Mela</li>
          <li role="option">Banana</li>
        </ul>
      </div>
    </form>
  );
};
Error Announcements
tsx
export const FormErrorsComponent = () => {
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [submitAttempted, setSubmitAttempted] = useState(false);

  const validateForm = (formData: FormData) => {
    const newErrors: Record<string, string> = {};

    const name = formData.get('name') as string;
    if (!name) {
      newErrors.name = 'Il nome è obbligatorio';
    } else if (name.length < 2) {
      newErrors.name = 'Il nome deve contenere almeno 2 caratteri';
    }

    const email = formData.get('email') as string;
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!email) {
      newErrors.email = "L'email è obbligatoria";
    } else if (!emailRegex.test(email)) {
      newErrors.email = "Inserisci un'email valida";
    }

    return newErrors;
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    setSubmitAttempted(true);
    
    const formData = new FormData(e.target as HTMLFormElement);
    const validationErrors = validateForm(formData);
    setErrors(validationErrors);

    if (Object.keys(validationErrors).length === 0) {
      // Invio form
      alert('Form valido!');
    } else {
      // Focus sul primo errore
      const firstErrorField = document.querySelector('[aria-invalid="true"]') as HTMLElement;
      firstErrorField?.focus();
    }
  };

  return (
    <form 
      onSubmit={handleSubmit}
      noValidate
      aria-label="Modulo con validazione"
    >
      {/* Live region per errori totali */}
      <div
        role="alert"
        aria-live="assertive"
        aria-atomic="true"
        className="sr-only"
      >
        {submitAttempted && Object.keys(errors).length > 0 && 
          `Sono presenti ${Object.keys(errors).length} errori nel form`}
      </div>

      <div className="form-group">
        <label htmlFor="name">Nome *</label>
        <input
          id="name"
          name="name"
          type="text"
          aria-required="true"
          aria-invalid={!!errors.name}
          aria-describedby={errors.name ? 'name-error' : undefined}
          onChange={() => {
            if (errors.name) {
              setErrors(prev => ({ ...prev, name: '' }));
            }
          }}
        />
        {errors.name && (
          <span 
            id="name-error" 
            role="alert"
            aria-live="polite"
            className="error-message"
          >
            {errors.name}
          </span>
        )}
      </div>

      <div className="form-group">
        <label htmlFor="email">Email *</label>
        <input
          id="email"
          name="email"
          type="email"
          aria-required="true"
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {errors.email && (
          <span 
            id="email-error"
            role="alert"
            aria-live="polite"
            className="error-message"
          >
            {errors.email}
          </span>
        )}
      </div>

      {/* Errori inline con icona */}
      <div className="form-group">
        <label htmlFor="password">Password</label>
        <input
          id="password"
          name="password"
          type="password"
          aria-describedby="password-requirements password-error"
        />
        <span id="password-requirements" className="help-text">
          Minimo 8 caratteri
        </span>
        {errors.password && (
          <div id="password-error" role="alert" className="error-with-icon">
            <span aria-hidden="true">⚠️</span>
            <span>{errors.password}</span>
          </div>
        )}
      </div>

      {/* Summary errori */}
      {submitAttempted && Object.keys(errors).length > 0 && (
        <div 
          role="alert"
          className="error-summary"
          aria-labelledby="error-summary-heading"
        >
          <h3 id="error-summary-heading">
            Correggi i seguenti errori:
          </h3>
          <ul>
            {Object.entries(errors).map(([field, message]) => (
              <li key={field}>
                <a 
                  href={`#${field}`}
                  onClick={(e) => {
                    e.preventDefault();
                    document.getElementById(field)?.focus();
                  }}
                >
                  {message}
                </a>
              </li>
            ))}
          </ul>
        </div>
      )}

      <button type="submit">Invia</button>
    </form>
  );
};
Required Field Indicators
tsx
export const RequiredFieldsComponent = () => {
  return (
    <form aria-label="Modulo con campi obbligatori">
      {/* Istruzioni generali */}
      <p className="required-instructions">
        I campi contrassegnati con <span className="required-asterisk">*</span> sono obbligatori
      </p>

      {/* Metodo 1: Asterisco visibile */}
      <div className="form-group">
        <label htmlFor="first-name">
          Nome <span className="required-asterisk" aria-hidden="true">*</span>
          <span className="sr-only">(obbligatorio)</span>
        </label>
        <input 
          id="first-name" 
          type="text" 
          required 
          aria-required="true"
        />
      </div>

      {/* Metodo 2: Testo obbligatorio visibile */}
      <div className="form-group">
        <label htmlFor="last-name">
          Cognome <span className="required-text">(obbligatorio)</span>
        </label>
        <input id="last-name" type="text" required />
      </div>

      {/* Metodo 3: Solo per screen reader */}
      <div className="form-group">
        <label htmlFor="email">
          Email
          <span className="sr-only"> (obbligatorio)</span>
        </label>
        <input 
          id="email" 
          type="email" 
          required 
          aria-required="true"
        />
      </div>

      {/* Metodo 4: Colore e icona */}
      <div className="form-group required-with-icon">
        <label htmlFor="phone">
          Telefono
          <span className="required-icon" aria-hidden="true">*</span>
          <span className="sr-only">(obbligatorio)</span>
        </label>
        <input id="phone" type="tel" required />
      </div>

      {/* Campo opzionale esplicito */}
      <div className="form-group">
        <label htmlFor="company">
          Azienda <span className="optional-text">(opzionale)</span>
        </label>
        <input id="company" type="text" />
      </div>

      {/* Gruppo di checkbox con required */}
      <fieldset className="required-fieldset">
        <legend>
          Autorizzazioni
          <span className="required-asterisk" aria-hidden="true">*</span>
          <span className="sr-only">(obbligatorio)</span>
        </legend>
        <div role="group" aria-required="true">
          <label>
            <input type="checkbox" name="permissions" value="marketing" required />
            Marketing
          </label>
          <label>
            <input type="checkbox" name="permissions" value="privacy" required />
            Privacy
          </label>
        </div>
        <p className="fieldset-help">
          Seleziona almeno un'opzione
        </p>
      </fieldset>

      <style jsx>{`
        .required-asterisk {
          color: #dc3545;
          font-weight: bold;
          margin-left: 2px;
        }

        .required-text {
          color: #666;
          font-size: 0.9em;
          margin-left: 4px;
        }

        .required-with-icon label {
          position: relative;
          padding-right: 20px;
        }

        .required-icon {
          position: absolute;
          right: 0;
          top: 0;
          color: #dc3545;
          font-size: 1.2em;
        }

        .optional-text {
          color: #6c757d;
          font-size: 0.85em;
          font-style: italic;
        }

        .required-instructions {
          font-size: 0.9em;
          color: #666;
          margin-bottom: 20px;
        }

        .required-fieldset {
          border: 2px solid #0056b3;
          padding: 15px;
          border-radius: 4px;
        }

        .required-fieldset legend {
          font-weight: bold;
          padding: 0 10px;
        }

        input:required {
          border-left: 3px solid #0056b3;
        }

        input:required:invalid {
          border-color: #dc3545;
        }

        .sr-only {
          position: absolute;
          width: 1px;
          height: 1px;
          padding: 0;
          margin: -1px;
          overflow: hidden;
          clip: rect(0, 0, 0, 0);
          white-space: nowrap;
          border: 0;
        }
      `}</style>
    </form>
  );
};
Help Text
tsx
export const FormHelpTextComponent = () => {
  return (
    <form aria-label="Modulo con testo di aiuto">
      {/* Help text associato con aria-describedby */}
      <div className="form-group">
        <label htmlFor="username">Username</label>
        <input
          id="username"
          type="text"
          aria-describedby="username-help"
          placeholder="Scegli un username"
        />
        <div id="username-help" className="help-text">
          <p>Deve contenere 3-20 caratteri</p>
          <p>Può includere lettere, numeri e underscore</p>
          <p>Non può iniziare con un numero</p>
        </div>
      </div>

      {/* Help text inline */}
      <div className="form-group">
        <label htmlFor="password">
          Password
          <span className="help-inline">
            (minimo 8 caratteri)
          </span>
        </label>
        <input
          id="password"
          type="password"
          aria-describedby="password-requirements password-strength"
        />
        <div id="password-requirements" className="help-text">
          <ul>
            <li>Almeno 8 caratteri</li>
            <li>Una lettera maiuscola</li>
            <li>Un numero</li>
            <li>Un carattere speciale</li>
          </ul>
        </div>
        <div id="password-strength" aria-live="polite" className="sr-only">
          Forza password: media
        </div>
      </div>

      {/* Help text con icona */}

## § ADVANCED PATTERNS: UI UX A11Y PATTERNS

### Server Actions con Validazione

```typescript
// app/actions/ui-ux-a11y-patterns.ts
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


### UI UX A11Y PATTERNS - Utility Helper #78

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-78.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #47

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-47.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #221

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-221.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #285

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-285.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #648

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-648.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #658

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-658.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #678

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-678.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #605

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-605.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #407

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-407.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #763

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-763.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #676

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-676.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #2

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-2.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #693

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-693.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #680

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-680.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #951

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-951.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #21

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-21.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<UIUXA11YPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### UI UX A11Y PATTERNS - Utility Helper #949

```typescript
// lib/utils/ui-ux-a11y-patterns-helper-949.ts
import { z } from "zod";

interface UIUXA11YPATTERNSConfig {
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

export class UIUXA11YPATTERNSProcessor<TInput, TOutput> {
  private config: UIUXA11YPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<UIUXA11YPATTERNSConfig> = {}) {
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
        const timeoutId = setTimeout(() => controller.abort(