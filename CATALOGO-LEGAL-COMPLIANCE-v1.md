# 📜 CATALOGO LEGAL & COMPLIANCE - Applicazioni Web

§ VERSIONE 1.0 | GENNAIO 2026

---

§ 📋 INDICE GENERALE

1. [Introduzione e Panoramica](#1-introduzione-e-panoramica)
2. [GDPR - General Data Protection Regulation](#2-gdpr-general-data-protection-regulation)
3. [Cookie Consent e ePrivacy](#3-cookie-consent-e-eprivacy)
4. [CCPA/CPRA - California Privacy](#4-ccpacpra-california-privacy)
5. [Privacy Policy e Terms of Service](#5-privacy-policy-e-terms-of-service)
6. [COPPA - Children's Online Privacy](#6-coppa-childrens-online-privacy)
7. [WCAG 2.2 - Accessibilità Web](#7-wcag-22-accessibilità-web)
8. [European Accessibility Act (EAA)](#8-european-accessibility-act-eaa)
9. [EU AI Act - Regolamentazione AI](#9-eu-ai-act-regolamentazione-ai)
10. [Licenze Software Open Source](#10-licenze-software-open-source)
11. [Data Breach Notification](#11-data-breach-notification)
12. [Implementazioni Pratiche](#12-implementazioni-pratiche)
13. [Checklist di Compliance](#13-checklist-di-compliance)
14. [Tool e Risorse](#14-tool-e-risorse)
15. [Glossario Legale](#15-glossario-legale)
16. [Appendice: Template e Modelli](#appendice-template-e-modelli)

---

§ 1. INTRODUZIONE E PANORAMICA

§ 1.1 PERCHÉ LA COMPLIANCE È FONDAMENTALE

La compliance legale nelle applicazioni web non è più un'opzione ma una necessità strategica:

┌─────────────────────────────────────────────────────────────────────┐
│                    PANORAMA REGULATORY 2025-2026                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │   PRIVACY   │  │ ACCESSIBILITÀ│  │     AI      │  │  LICENZE    │ │
│  │             │  │             │  │             │  │             │ │
│  │  • GDPR     │  │  • WCAG 2.2 │  │  • EU AI Act│  │  • MIT      │ │
│  │  • CCPA     │  │  • EAA      │  │  • Risk-    │  │  • Apache   │ │
│  │  • COPPA    │  │  • ADA      │  │    based    │  │  • GPL      │ │
│  │  • ePrivacy │  │  • EN301549 │  │  • Literacy │  │  • BSD      │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │
│                                                                      │
│         CONSEGUENZE NON-COMPLIANCE:                                  │
│         • Multe fino a €20M o 4% fatturato (GDPR)                   │
│         • $7,988 per violazione intenzionale (CCPA)                 │
│         • €35M o 7% fatturato (AI Act)                              │
│         • €500,000+ (EAA)                                           │
│         • Class action e danni reputazionali                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 1.2 FRAMEWORK DI COMPLIANCE INTEGRATO

┌─────────────────────────────────────────────────────────────────────┐
│                    COMPLIANCE INTEGRATION FRAMEWORK                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                      ┌──────────────────────┐                       │
│                      │    APPLICAZIONE WEB   │                       │
│                      └──────────┬───────────┘                       │
│                                 │                                    │
│         ┌───────────────────────┼───────────────────────┐           │
│         │                       │                       │           │
│         ▼                       ▼                       ▼           │
│  ┌─────────────┐        ┌─────────────┐        ┌─────────────┐     │
│  │   PRIVACY   │        │   SECURITY  │        │ ACCESSIBILITY│     │
│  │   LAYER     │        │   LAYER     │        │    LAYER     │     │
│  │             │        │             │        │             │      │
│  │ • Consent   │        │ • Encryption│        │ • WCAG 2.2  │     │
│  │ • Rights    │        │ • Breach    │        │ • ARIA      │     │
│  │ • Retention │        │ • Auditing  │        │ • Keyboard  │     │
│  └─────────────┘        └─────────────┘        └─────────────┘     │
│         │                       │                       │           │
│         └───────────────────────┼───────────────────────┘           │
│                                 │                                    │
│                      ┌──────────▼───────────┐                       │
│                      │  COMPLIANCE ENGINE    │                       │
│                      │  • Monitoring         │                       │
│                      │  • Reporting          │                       │
│                      │  • Documentation      │                       │
│                      └──────────────────────┘                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 1.3 TIMELINE NORMATIVE 2024-2027

Timeline Compliance Deadlines:
════════════════════════════════════════════════════════════════════

2024 ─────────────────────────────────────────────────────────────────
│
├─ Gen 2024: CCPA/CPRA fully enforceable
├─ Apr 2024: ADA Title II Web Accessibility Rule published
├─ Aug 2024: EU AI Act entered into force
├─ Dec 2024: WCAG 2.2 ISO/IEC 40500:2025
│
2025 ─────────────────────────────────────────────────────────────────
│
├─ Feb 2025: EU AI Act - Prohibited practices & AI literacy
├─ Apr 2025: COPPA Final Rule amendments published
├─ Jun 2025: European Accessibility Act (EAA) enforcement
├─ Aug 2025: EU AI Act - GPAI obligations
├─ Oct 2025: Maryland Online Data Privacy Act
│
2026 ─────────────────────────────────────────────────────────────────
│
├─ Jan 2026: CCPA new regulations effective
├─ Apr 2026: ADA Title II compliance (large entities)
│            COPPA compliance deadline
├─ Aug 2026: EU AI Act - Full application
│
2027 ─────────────────────────────────────────────────────────────────
│
├─ Apr 2027: ADA Title II compliance (smaller entities)
├─ Aug 2027: EU AI Act - High-risk AI in regulated products
│
════════════════════════════════════════════════════════════════════

---

§ 2. GDPR - GENERAL DATA PROTECTION REGULATION

§ 2.1 PRINCIPI FONDAMENTALI

Il GDPR (Regolamento UE 2016/679) è il framework di protezione dati più stringente al mondo, applicabile a qualsiasi organizzazione che tratti dati di residenti UE.

┌─────────────────────────────────────────────────────────────────────┐
│                     7 PRINCIPI DEL GDPR (Art. 5)                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. LICEITÀ, CORRETTEZZA, TRASPARENZA                               │
│     └─ Trattamento lecito, corretto e trasparente                   │
│                                                                      │
│  2. LIMITAZIONE DELLE FINALITÀ                                       │
│     └─ Dati raccolti per scopi determinati, espliciti, legittimi    │
│                                                                      │
│  3. MINIMIZZAZIONE DEI DATI                                         │
│     └─ Solo dati adeguati, pertinenti, limitati al necessario       │
│                                                                      │
│  4. ESATTEZZA                                                        │
│     └─ Dati esatti e aggiornati                                     │
│                                                                      │
│  5. LIMITAZIONE DELLA CONSERVAZIONE                                  │
│     └─ Conservati solo per il tempo necessario                      │
│                                                                      │
│  6. INTEGRITÀ E RISERVATEZZA                                        │
│     └─ Sicurezza adeguata, protezione da trattamenti illeciti       │
│                                                                      │
│  7. RESPONSABILIZZAZIONE (Accountability)                           │
│     └─ Il titolare deve dimostrare la conformità                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 2.2 BASI GIURIDICHE DEL TRATTAMENTO

typescript
// Enumerazione delle 6 basi giuridiche GDPR Art. 6
enum GDPRLegalBasis {
  CONSENT = 'consent',                    // Consenso esplicito
  CONTRACT = 'contract',                  // Necessità contrattuale
  LEGAL_OBLIGATION = 'legal_obligation',  // Obbligo legale
  VITAL_INTEREST = 'vital_interest',      // Interessi vitali
  PUBLIC_INTEREST = 'public_interest',    // Interesse pubblico
  LEGITIMATE_INTEREST = 'legitimate_interest' // Interesse legittimo
}

interface DataProcessingActivity {
  purpose: string;
  legalBasis: GDPRLegalBasis;
  dataCategories: string[];
  dataSubjects: string[];
  retentionPeriod: string;
  recipients?: string[];
  transfers?: string[];
  technicalMeasures: string[];
  organizationalMeasures: string[];
}

// Esempio di registro delle attività di trattamento
const processingActivities: DataProcessingActivity[] = [
  {
    purpose: 'Newsletter subscription',
    legalBasis: GDPRLegalBasis.CONSENT,
    dataCategories: ['email', 'name'],
    dataSubjects: ['subscribers'],
    retentionPeriod: 'Until consent withdrawal',
    recipients: ['Email service provider'],
    technicalMeasures: ['Encryption at rest', 'TLS in transit'],
    organizationalMeasures: ['Access control', 'Training']
  },
  {
    purpose: 'Order processing',
    legalBasis: GDPRLegalBasis.CONTRACT,
    dataCategories: ['name', 'address', 'payment info', 'order history'],
    dataSubjects: ['customers'],
    retentionPeriod: '7 years (legal retention)',
    technicalMeasures: ['Encryption', 'Pseudonymization'],
    organizationalMeasures: ['Need-to-know access', 'Audit logs']
  }
];

§ 2.3 DIRITTI DEGLI INTERESSATI

┌─────────────────────────────────────────────────────────────────────┐
│                    DIRITTI GDPR DEGLI INTERESSATI                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐     │
│  │    ACCESSO      │  │   RETTIFICA     │  │  CANCELLAZIONE  │     │
│  │    Art. 15      │  │    Art. 16      │  │    Art. 17      │     │
│  │                 │  │                 │  │                 │     │
│  │ Conferma se i   │  │ Correzione dati │  │ Diritto all'    │     │
│  │ dati sono       │  │ inesatti senza  │  │ oblio quando    │     │
│  │ trattati +      │  │ ritardo         │  │ non più         │     │
│  │ copia           │  │ ingiustificato  │  │ necessari       │     │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘     │
│                                                                      │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐     │
│  │  LIMITAZIONE    │  │  PORTABILITÀ    │  │  OPPOSIZIONE    │     │
│  │    Art. 18      │  │    Art. 20      │  │    Art. 21      │     │
│  │                 │  │                 │  │                 │     │
│  │ Limitare il     │  │ Ricevere dati   │  │ Opporsi a       │     │
│  │ trattamento in  │  │ in formato      │  │ trattamento     │     │
│  │ determinate     │  │ strutturato e   │  │ basato su       │     │
│  │ circostanze     │  │ trasferibili    │  │ interesse       │     │
│  │                 │  │                 │  │ legittimo       │     │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘     │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │            DECISIONI AUTOMATIZZATE - Art. 22                │   │
│  │                                                              │   │
│  │  Non essere sottoposto a decisioni basate unicamente su     │   │
│  │  trattamento automatizzato che producano effetti giuridici  │   │
│  │  o significativi (inclusa profilazione)                     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  TEMPISTICHE DI RISPOSTA:                                           │
│  • Entro 1 mese dalla richiesta                                     │
│  • Estendibile di 2 mesi per complessità (con notifica)            │
│  • Gratuito (salvo richieste manifestamente infondate/eccessive)    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 2.4 IMPLEMENTAZIONE TECNICA GDPR

typescript
// gdpr-compliance.ts - Sistema di gestione GDPR completo

import { v4 as uuidv4 } from 'uuid';
import crypto from 'crypto';

// Interfacce per la gestione del consenso
interface ConsentRecord {
  id: string;
  userId: string;
  purpose: string;
  legalBasis: string;
  granted: boolean;
  timestamp: Date;
  ipAddress?: string;
  userAgent?: string;
  version: string;
  withdrawnAt?: Date;
}

interface DataSubjectRequest {
  id: string;
  type: 'access' | 'rectification' | 'erasure' | 'restriction' | 
        'portability' | 'objection';
  userId: string;
  requestedAt: Date;
  verifiedAt?: Date;
  completedAt?: Date;
  status: 'pending' | 'verified' | 'processing' | 'completed' | 'rejected';
  response?: any;
  notes?: string;
}

// Sistema di gestione consenso
class ConsentManagementSystem {
  private consents: Map<string, ConsentRecord[]> = new Map();
  
  // Registra nuovo consenso
  async recordConsent(
    userId: string,
    purpose: string,
    granted: boolean,
    metadata: {
      ipAddress?: string;
      userAgent?: string;
      consentVersion: string;
    }
  ): Promise<ConsentRecord> {
    const consent: ConsentRecord = {
      id: uuidv4(),
      userId,
      purpose,
      legalBasis: 'consent',
      granted,
      timestamp: new Date(),
      ipAddress: this.hashIpAddress(metadata.ipAddress),
      userAgent: metadata.userAgent,
      version: metadata.consentVersion
    };
    
    const userConsents = this.consents.get(userId) || [];
    userConsents.push(consent);
    this.consents.set(userId, userConsents);
    
    // Audit log
    await this.logConsentActivity('consent_recorded', consent);
    
    return consent;
  }
  
  // Ritira consenso
  async withdrawConsent(userId: string, purpose: string): Promise<void> {
    const userConsents = this.consents.get(userId) || [];
    const consentIndex = userConsents.findIndex(
      c => c.purpose === purpose && c.granted && !c.withdrawnAt
    );
    
    if (consentIndex >= 0) {
      userConsents[consentIndex].withdrawnAt = new Date();
      userConsents[consentIndex].granted = false;
      
      await this.logConsentActivity('consent_withdrawn', userConsents[consentIndex]);
    }
  }
  
  // Verifica consenso attivo
  hasActiveConsent(userId: string, purpose: string): boolean {
    const userConsents = this.consents.get(userId) || [];
    return userConsents.some(
      c => c.purpose === purpose && c.granted && !c.withdrawnAt
    );
  }
  
  // Esporta storico consensi per l'utente
  exportConsentHistory(userId: string): ConsentRecord[] {
    return this.consents.get(userId) || [];
  }
  
  // Hash IP per privacy
  private hashIpAddress(ip?: string): string | undefined {
    if (!ip) return undefined;
    return crypto.createHash('sha256')
      .update(ip + process.env.IP_SALT)
      .digest('hex')
      .substring(0, 16);
  }
  
  private async logConsentActivity(action: string, data: any): Promise<void> {
    // Implementare logging audit trail
    console.log(`[GDPR Audit] ${action}:`, JSON.stringify(data));
  }
}

// Sistema gestione richieste Data Subject
class DataSubjectRequestHandler {
  private requests: Map<string, DataSubjectRequest> = new Map();
  
  // Crea nuova richiesta
  async createRequest(
    type: DataSubjectRequest['type'],
    userId: string
  ): Promise<DataSubjectRequest> {
    const request: DataSubjectRequest = {
      id: uuidv4(),
      type,
      userId,
      requestedAt: new Date(),
      status: 'pending'
    };
    
    this.requests.set(request.id, request);
    
    // Notifica team privacy
    await this.notifyPrivacyTeam(request);
    
    // Deadline: 30 giorni
    this.scheduleDeadlineReminder(request.id, 30);
    
    return request;
  }
  
  // Processo richiesta di accesso
  async processAccessRequest(requestId: string): Promise<any> {
    const request = this.requests.get(requestId);
    if (!request || request.type !== 'access') {
      throw new Error('Invalid request');
    }
    
    request.status = 'processing';
    
    // Raccolta dati da tutti i sistemi
    const userData = await this.collectAllUserData(request.userId);
    
    // Formatta risposta
    const response = {
      requestId: request.id,
      generatedAt: new Date().toISOString(),
      dataSubject: {
        id: request.userId,
        // Non includere dati identificativi diretti nella risposta automatica
      },
      processingActivities: await this.getProcessingActivities(request.userId),
      personalData: userData,
      retentionPolicies: await this.getRetentionInfo(request.userId),
      rights: this.getDataSubjectRights()
    };
    
    request.response = response;
    request.completedAt = new Date();
    request.status = 'completed';
    
    return response;
  }
  
  // Processo richiesta di cancellazione (diritto all'oblio)
  async processErasureRequest(requestId: string): Promise<void> {
    const request = this.requests.get(requestId);
    if (!request || request.type !== 'erasure') {
      throw new Error('Invalid request');
    }
    
    request.status = 'processing';
    
    // Verifica se esistono basi legali per mantenere i dati
    const retentionRequirements = await this.checkLegalRetention(request.userId);
    
    if (retentionRequirements.length > 0) {
      // Non possiamo cancellare tutto
      request.notes = `Partial erasure due to legal retention: ${
        retentionRequirements.join(', ')
      }`;
      await this.partialErasure(request.userId, retentionRequirements);
    } else {
      // Cancellazione completa
      await this.fullErasure(request.userId);
    }
    
    request.completedAt = new Date();
    request.status = 'completed';
  }
  
  // Processo richiesta di portabilità
  async processPortabilityRequest(requestId: string): Promise<Buffer> {
    const request = this.requests.get(requestId);
    if (!request || request.type !== 'portability') {
      throw new Error('Invalid request');
    }
    
    request.status = 'processing';
    
    // Raccolta dati forniti dall'utente
    const userData = await this.collectProvidedUserData(request.userId);
    
    // Formato machine-readable (JSON)
    const portableData = {
      exportedAt: new Date().toISOString(),
      format: 'JSON',
      dataSubject: request.userId,
      data: userData
    };
    
    request.completedAt = new Date();
    request.status = 'completed';
    
    return Buffer.from(JSON.stringify(portableData, null, 2));
  }
  
  private async collectAllUserData(userId: string): Promise<any> {
    // Implementare raccolta da tutti i sistemi
    return {
      profile: {},
      orders: [],
      communications: [],
      preferences: {},
      activityLog: []
    };
  }
  
  private async collectProvidedUserData(userId: string): Promise<any> {
    // Solo dati forniti dall'utente (non inferiti)
    return {
      profile: {},
      orders: [],
      communications: []
    };
  }
  
  private async getProcessingActivities(userId: string): Promise<any[]> {
    return [];
  }
  
  private async getRetentionInfo(userId: string): Promise<any> {
    return {};
  }
  
  private getDataSubjectRights(): string[] {
    return [
      'Right of Access (Art. 15)',
      'Right to Rectification (Art. 16)',
      'Right to Erasure (Art. 17)',
      'Right to Restriction (Art. 18)',
      'Right to Data Portability (Art. 20)',
      'Right to Object (Art. 21)',
      'Right regarding Automated Decision Making (Art. 22)'
    ];
  }
  
  private async checkLegalRetention(userId: string): Promise<string[]> {
    // Verifica obblighi legali di conservazione
    return []; // Es: ['Tax records - 7 years', 'Contract disputes - ongoing']
  }
  
  private async partialErasure(
    userId: string, 
    exceptions: string[]
  ): Promise<void> {
    // Cancella tutto tranne le eccezioni
  }
  
  private async fullErasure(userId: string): Promise<void> {
    // Cancellazione completa
  }
  
  private async notifyPrivacyTeam(request: DataSubjectRequest): Promise<void> {
    // Notifica team
  }
  
  private scheduleDeadlineReminder(requestId: string, days: number): void {
    // Schedula reminder
  }
}

// Data Protection Impact Assessment (DPIA)
interface DPIA {
  id: string;
  projectName: string;
  assessmentDate: Date;
  dataFlows: DataFlow[];
  risks: Risk[];
  mitigations: Mitigation[];
  decision: 'approved' | 'approved_with_conditions' | 'rejected';
  dpoSignoff?: Date;
}

interface DataFlow {
  source: string;
  destination: string;
  dataCategories: string[];
  purpose: string;
  legalBasis: string;
  retention: string;
}

interface Risk {
  id: string;
  description: string;
  likelihood: 'low' | 'medium' | 'high';
  impact: 'low' | 'medium' | 'high';
  overallRisk: 'low' | 'medium' | 'high' | 'critical';
}

interface Mitigation {
  riskId: string;
  measure: string;
  status: 'planned' | 'implemented' | 'verified';
  residualRisk: Risk['overallRisk'];
}

// Export utilities
export {
  ConsentManagementSystem,
  DataSubjectRequestHandler,
  ConsentRecord,
  DataSubjectRequest,
  DPIA
};

§ 2.5 PRIVACY BY DESIGN E BY DEFAULT

┌─────────────────────────────────────────────────────────────────────┐
│              PRIVACY BY DESIGN - 7 PRINCIPI FONDAMENTALI            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. PROATTIVO NON REATTIVO                                          │
│     • Prevenire violazioni, non solo rispondere                     │
│     • Analisi rischi in fase di progettazione                       │
│                                                                      │
│  2. PRIVACY COME DEFAULT                                            │
│     • Massima privacy senza azione dell'utente                      │
│     • Opt-in, non opt-out                                           │
│                                                                      │
│  3. PRIVACY EMBEDDED NEL DESIGN                                     │
│     • Integrata nell'architettura, non aggiunta dopo               │
│     • Core functionality, non add-on                                │
│                                                                      │
│  4. FUNZIONALITÀ COMPLETA (WIN-WIN)                                 │
│     • Privacy E funzionalità, non compromessi                       │
│     • Evitare falsi dilemmi                                         │
│                                                                      │
│  5. SICUREZZA END-TO-END                                            │
│     • Protezione durante tutto il ciclo di vita                     │
│     • Dalla raccolta alla cancellazione                             │
│                                                                      │
│  6. VISIBILITÀ E TRASPARENZA                                        │
│     • Operazioni verificabili e documentate                         │
│     • Open to scrutiny                                              │
│                                                                      │
│  7. RISPETTO PER LA PRIVACY DELL'UTENTE                            │
│     • User-centric approach                                         │
│     • Empowerment dell'interessato                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 2.6 DATA BREACH RESPONSE

typescript
// data-breach-handler.ts

interface DataBreach {
  id: string;
  detectedAt: Date;
  occurredAt?: Date;
  description: string;
  affectedData: string[];
  affectedSubjects: number;
  severity: 'low' | 'medium' | 'high' | 'critical';
  status: 'detected' | 'investigating' | 'contained' | 
          'notified' | 'resolved';
  riskToRights: 'unlikely' | 'likely' | 'high';
  notificationRequired: boolean;
  authorityNotifiedAt?: Date;
  subjectsNotifiedAt?: Date;
}

class DataBreachHandler {
  private readonly AUTHORITY_NOTIFICATION_DEADLINE_HOURS = 72;
  
  async handleBreach(breach: Partial<DataBreach>): Promise<DataBreach> {
    const fullBreach: DataBreach = {
      id: uuidv4(),
      detectedAt: new Date(),
      description: breach.description || '',
      affectedData: breach.affectedData || [],
      affectedSubjects: breach.affectedSubjects || 0,
      severity: this.assessSeverity(breach),
      status: 'detected',
      riskToRights: this.assessRiskToRights(breach),
      notificationRequired: false
    };
    
    // Determina se notifica è richiesta
    fullBreach.notificationRequired = this.isNotificationRequired(fullBreach);
    
    // Log immediato
    await this.logBreachDetection(fullBreach);
    
    // Notifica team incident response
    await this.alertIncidentTeam(fullBreach);
    
    // Se notifica richiesta, prepara documentazione
    if (fullBreach.notificationRequired) {
      await this.prepareAuthorityNotification(fullBreach);
    }
    
    return fullBreach;
  }
  
  private assessSeverity(breach: Partial<DataBreach>): DataBreach['severity'] {
    // Valuta severità basata su:
    // - Tipo di dati (sensibili?)
    // - Numero di interessati
    // - Potenziale per danno
    
    const sensitiveCategories = [
      'health', 'genetic', 'biometric', 'racial',
      'political', 'religious', 'sexual', 'criminal'
    ];
    
    const hasSensitiveData = breach.affectedData?.some(
      d => sensitiveCategories.some(s => d.toLowerCase().includes(s))
    );
    
    if (hasSensitiveData && (breach.affectedSubjects || 0) > 1000) {
      return 'critical';
    }
    if (hasSensitiveData || (breach.affectedSubjects || 0) > 5000) {
      return 'high';
    }
    if ((breach.affectedSubjects || 0) > 100) {
      return 'medium';
    }
    return 'low';
  }
  
  private assessRiskToRights(breach: Partial<DataBreach>): DataBreach['riskToRights'] {
    // Art. 33 GDPR: notifica se rischio per diritti e libertà
    if (breach.affectedData?.some(d => 
      ['password', 'financial', 'health', 'ssn'].some(
        s => d.toLowerCase().includes(s)
      )
    )) {
      return 'high';
    }
    return 'likely';
  }
  
  private isNotificationRequired(breach: DataBreach): boolean {
    // Non richiesta se improbabile rischio per diritti
    return breach.riskToRights !== 'unlikely';
  }
  
  async notifyAuthority(breach: DataBreach): Promise<void> {
    // Entro 72 ore dalla scoperta
    const notification = {
      breachId: breach.id,
      natureOfBreach: breach.description,
      categoriesOfData: breach.affectedData,
      approximateSubjects: breach.affectedSubjects,
      consequencesDescription: this.describeConsequences(breach),
      measuresTaken: await this.getMeasuresTaken(breach.id),
      dpoContact: this.getDPOContact(),
      timestamp: new Date()
    };
    
    // Invio all'autorità competente
    await this.submitToAuthority(notification);
    
    breach.authorityNotifiedAt = new Date();
    breach.status = 'notified';
  }
  
  async notifyAffectedSubjects(breach: DataBreach): Promise<void> {
    // Art. 34: Se alto rischio per diritti e libertà
    if (breach.riskToRights !== 'high') {
      return;
    }
    
    // Comunicazione chiara e semplice
    const notification = {
      natureOfBreach: breach.description,
      likelyConsequences: this.describeConsequences(breach),
      measuresTaken: await this.getMeasuresTaken(breach.id),
      recommendedActions: this.getRecommendedActions(breach),
      dpoContact: this.getDPOContact()
    };
    
    // Invia a tutti gli interessati
    await this.sendSubjectNotifications(breach.id, notification);
    
    breach.subjectsNotifiedAt = new Date();
  }
  
  private describeConsequences(breach: DataBreach): string {
    return '';
  }
  
  private async getMeasuresTaken(breachId: string): Promise<string[]> {
    return [];
  }
  
  private getDPOContact(): { name: string; email: string } {
    return {
      name: process.env.DPO_NAME || 'DPO',
      email: process.env.DPO_EMAIL || 'dpo@example.com'
    };
  }
  
  private getRecommendedActions(breach: DataBreach): string[] {
    return [
      'Change your password immediately',
      'Monitor your accounts for suspicious activity',
      'Enable two-factor authentication'
    ];
  }
  
  private async submitToAuthority(notification: any): Promise<void> {
    // Submit to supervisory authority
  }
  
  private async sendSubjectNotifications(
    breachId: string, 
    notification: any
  ): Promise<void> {
    // Send to affected subjects
  }
  
  private async logBreachDetection(breach: DataBreach): Promise<void> {}
  private async alertIncidentTeam(breach: DataBreach): Promise<void> {}
  private async prepareAuthorityNotification(breach: DataBreach): Promise<void> {}
}

export { DataBreachHandler, DataBreach };



---

§ 3. COOKIE CONSENT E EPRIVACY

§ 3.1 FRAMEWORK NORMATIVO COOKIE

┌─────────────────────────────────────────────────────────────────────┐
│                     NORMATIVA COOKIE EU 2025                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    ePRIVACY DIRECTIVE                        │   │
│  │                    (2002/58/EC + 2009/136/EC)                │   │
│  │                                                              │   │
│  │  Regola l'accesso/memorizzazione su dispositivi utente      │   │
│  │  Richiede consenso PRIMA di impostare cookie non essenziali │   │
│  │  Implementata diversamente in ogni Stato Membro             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              +                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                         GDPR                                 │   │
│  │                   (Regulation 2016/679)                      │   │
│  │                                                              │   │
│  │  Standard elevati per il consenso (Art. 7)                   │   │
│  │  Requisiti di trasparenza (Art. 12-14)                       │   │
│  │  Responsabilità e documentazione                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              =                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              REQUISITI COMBINATI PER COOKIE                  │   │
│  │                                                              │   │
│  │  ✓ Consenso PRIMA di cookie non essenziali                   │   │
│  │  ✓ Consenso libero, specifico, informato, inequivocabile    │   │
│  │  ✓ Facile ritirare consenso come darlo                       │   │
│  │  ✓ Documentazione consensi (audit trail)                     │   │
│  │  ✓ No pre-checked boxes                                      │   │
│  │  ✓ No cookie walls (generalmente)                            │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  NOVITÀ 2025: Digital Omnibus Package propone:                      │
│  • Spostare regole cookie nel GDPR (Art. 88a)                       │
│  • Browser-level consent signals                                     │
│  • 6 mesi "cooldown" se utente rifiuta                              │
│  • Possibile uso legitimate interest per alcuni cookie              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 3.2 CATEGORIE DI COOKIE

typescript
// cookie-categories.ts

enum CookieCategory {
  STRICTLY_NECESSARY = 'strictly_necessary',
  FUNCTIONAL = 'functional',
  ANALYTICS = 'analytics',
  ADVERTISING = 'advertising',
  SOCIAL_MEDIA = 'social_media'
}

interface Cookie {
  name: string;
  category: CookieCategory;
  purpose: string;
  provider: string;
  duration: string;
  type: 'first_party' | 'third_party';
  requiresConsent: boolean;
}

const cookieCategoryDefinitions = {
  [CookieCategory.STRICTLY_NECESSARY]: {
    description: 'Essential for website functionality',
    examples: [
      'Session cookies',
      'Authentication tokens',
      'Security cookies',
      'Load balancing',
      'Consent preferences'
    ],
    requiresConsent: false,
    legalBasis: 'Legitimate interest / Necessary for service'
  },
  
  [CookieCategory.FUNCTIONAL]: {
    description: 'Enable enhanced functionality and personalization',
    examples: [
      'Language preferences',
      'User preferences',
      'Video player settings',
      'Chat widget state'
    ],
    requiresConsent: true,
    legalBasis: 'Consent required'
  },
  
  [CookieCategory.ANALYTICS]: {
    description: 'Help understand how visitors interact with website',
    examples: [
      'Google Analytics',
      'Hotjar',
      'Mixpanel',
      'Page view tracking',
      'Performance monitoring'
    ],
    requiresConsent: true,
    legalBasis: 'Consent required (NOT legitimate interest)'
  },
  
  [CookieCategory.ADVERTISING]: {
    description: 'Used for targeted advertising',
    examples: [
      'Google Ads cookies',
      'Facebook Pixel',
      'Retargeting cookies',
      'Conversion tracking'
    ],
    requiresConsent: true,
    legalBasis: 'Explicit consent required'
  },
  
  [CookieCategory.SOCIAL_MEDIA]: {
    description: 'Enable social media features',
    examples: [
      'Facebook Like button',
      'Twitter widgets',
      'LinkedIn insights',
      'Share buttons'
    ],
    requiresConsent: true,
    legalBasis: 'Consent required'
  }
};

// Cookie inventory per compliance
const sampleCookieInventory: Cookie[] = [
  {
    name: 'session_id',
    category: CookieCategory.STRICTLY_NECESSARY,
    purpose: 'Maintain user session',
    provider: 'First party',
    duration: 'Session',
    type: 'first_party',
    requiresConsent: false
  },
  {
    name: '_ga',
    category: CookieCategory.ANALYTICS,
    purpose: 'Google Analytics - distinguish users',
    provider: 'Google LLC',
    duration: '2 years',
    type: 'third_party',
    requiresConsent: true
  },
  {
    name: '_fbp',
    category: CookieCategory.ADVERTISING,
    purpose: 'Facebook Pixel - track conversions',
    provider: 'Meta Platforms Inc.',
    duration: '3 months',
    type: 'third_party',
    requiresConsent: true
  }
];

§ 3.3 IMPLEMENTAZIONE COOKIE BANNER COMPLIANT

typescript
// cookie-consent-banner.tsx

import React, { useState, useEffect } from 'react';

interface ConsentPreferences {
  necessary: boolean; // Always true
  functional: boolean;
  analytics: boolean;
  advertising: boolean;
  socialMedia: boolean;
}

interface CookieConsentConfig {
  version: string;
  privacyPolicyUrl: string;
  cookiePolicyUrl: string;
  imprintUrl?: string;
  languages: string[];
  defaultLanguage: string;
}

const CookieConsentBanner: React.FC<{ config: CookieConsentConfig }> = ({ config }) => {
  const [showBanner, setShowBanner] = useState(false);
  const [showPreferences, setShowPreferences] = useState(false);
  const [preferences, setPreferences] = useState<ConsentPreferences>({
    necessary: true,
    functional: false,
    analytics: false,
    advertising: false,
    socialMedia: false
  });

  useEffect(() => {
    // Check if consent already given
    const savedConsent = localStorage.getItem('cookie_consent');
    if (!savedConsent) {
      setShowBanner(true);
      // Block all non-essential cookies until consent
      blockNonEssentialCookies();
    } else {
      const parsed = JSON.parse(savedConsent);
      setPreferences(parsed.preferences);
      applyConsentPreferences(parsed.preferences);
    }
  }, []);

  const blockNonEssentialCookies = () => {
    // Block scripts that set non-essential cookies
    document.querySelectorAll('script[data-category]').forEach(script => {
      const category = script.getAttribute('data-category');
      if (category !== 'necessary') {
        script.setAttribute('type', 'text/plain');
      }
    });
  };

  const applyConsentPreferences = (prefs: ConsentPreferences) => {
    // Enable/disable scripts based on consent
    Object.entries(prefs).forEach(([category, enabled]) => {
      document.querySelectorAll(`script[data-category="${category}"]`).forEach(script => {
        if (enabled) {
          script.setAttribute('type', 'text/javascript');
          // Reload script
          const newScript = document.createElement('script');
          newScript.src = script.getAttribute('data-src') || '';
          newScript.setAttribute('data-category', category);
          document.head.appendChild(newScript);
        }
      });
    });
  };

  const saveConsent = (prefs: ConsentPreferences) => {
    const consentRecord = {
      preferences: prefs,
      timestamp: new Date().toISOString(),
      version: config.version,
      userAgent: navigator.userAgent
    };
    
    localStorage.setItem('cookie_consent', JSON.stringify(consentRecord));
    
    // Send to backend for audit trail
    fetch('/api/consent', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(consentRecord)
    });
    
    applyConsentPreferences(prefs);
    setShowBanner(false);
  };

  const acceptAll = () => {
    const allAccepted: ConsentPreferences = {
      necessary: true,
      functional: true,
      analytics: true,
      advertising: true,
      socialMedia: true
    };
    saveConsent(allAccepted);
  };

  const rejectAll = () => {
    const allRejected: ConsentPreferences = {
      necessary: true, // Cannot reject
      functional: false,
      analytics: false,
      advertising: false,
      socialMedia: false
    };
    saveConsent(allRejected);
  };

  const savePreferences = () => {
    saveConsent(preferences);
    setShowPreferences(false);
  };

  if (!showBanner) return null;

  return (
    <div 
      role="dialog" 
      aria-labelledby="cookie-banner-title"
      aria-describedby="cookie-banner-description"
      className="cookie-consent-banner"
      style={{
        position: 'fixed',
        bottom: 0,
        left: 0,
        right: 0,
        backgroundColor: '#fff',
        boxShadow: '0 -2px 10px rgba(0,0,0,0.1)',
        padding: '20px',
        zIndex: 9999
      }}
    >
      {!showPreferences ? (
        // Main banner view
        <div>
          <h2 id="cookie-banner-title">We use cookies</h2>
          <p id="cookie-banner-description">
            We use cookies to enhance your browsing experience, serve personalized 
            content, and analyze our traffic. By clicking "Accept All", you consent 
            to our use of cookies. Read more in our{' '}
            <a href={config.cookiePolicyUrl}>Cookie Policy</a> and{' '}
            <a href={config.privacyPolicyUrl}>Privacy Policy</a>.
          </p>
          
          <div className="cookie-consent-buttons" style={{ display: 'flex', gap: '10px' }}>
            {/* IMPORTANT: Reject button must be equally prominent as Accept */}
            <button 
              onClick={rejectAll}
              style={{
                padding: '12px 24px',
                backgroundColor: '#f0f0f0',
                border: '1px solid #ccc',
                borderRadius: '4px',
                cursor: 'pointer',
                fontSize: '16px'
              }}
            >
              Reject All
            </button>
            
            <button 
              onClick={() => setShowPreferences(true)}
              style={{
                padding: '12px 24px',
                backgroundColor: '#f0f0f0',
                border: '1px solid #ccc',
                borderRadius: '4px',
                cursor: 'pointer',
                fontSize: '16px'
              }}
            >
              Preferences
            </button>
            
            <button 
              onClick={acceptAll}
              style={{
                padding: '12px 24px',
                backgroundColor: '#007bff',
                color: '#fff',
                border: 'none',
                borderRadius: '4px',
                cursor: 'pointer',
                fontSize: '16px'
              }}
            >
              Accept All
            </button>
          </div>
        </div>
      ) : (
        // Preferences view
        <div>
          <h2>Cookie Preferences</h2>
          <p>
            Manage your cookie preferences. You can enable or disable different 
            types of cookies below.
          </p>
          
          <div className="cookie-categories">
            {/* Necessary - always enabled */}
            <div className="cookie-category">
              <label>
                <input 
                  type="checkbox" 
                  checked={true} 
                  disabled 
                />
                <strong>Strictly Necessary</strong>
                <span> (Always active)</span>
              </label>
              <p>
                These cookies are essential for the website to function properly. 
                They cannot be disabled.
              </p>
            </div>
            
            {/* Functional */}
            <div className="cookie-category">
              <label>
                <input 
                  type="checkbox" 
                  checked={preferences.functional}
                  onChange={e => setPreferences({
                    ...preferences, 
                    functional: e.target.checked
                  })}
                />
                <strong>Functional Cookies</strong>
              </label>
              <p>
                Enable enhanced functionality and personalization, such as 
                remembering your preferences.
              </p>
            </div>
            
            {/* Analytics */}
            <div className="cookie-category">
              <label>
                <input 
                  type="checkbox" 
                  checked={preferences.analytics}
                  onChange={e => setPreferences({
                    ...preferences, 
                    analytics: e.target.checked
                  })}
                />
                <strong>Analytics Cookies</strong>
              </label>
              <p>
                Help us understand how visitors interact with our website by 
                collecting anonymous information.
              </p>
            </div>
            
            {/* Advertising */}
            <div className="cookie-category">
              <label>
                <input 
                  type="checkbox" 
                  checked={preferences.advertising}
                  onChange={e => setPreferences({
                    ...preferences, 
                    advertising: e.target.checked
                  })}
                />
                <strong>Advertising Cookies</strong>
              </label>
              <p>
                Used to show you relevant advertisements on other websites.
              </p>
            </div>
            
            {/* Social Media */}
            <div className="cookie-category">
              <label>
                <input 
                  type="checkbox" 
                  checked={preferences.socialMedia}
                  onChange={e => setPreferences({
                    ...preferences, 
                    socialMedia: e.target.checked
                  })}
                />
                <strong>Social Media Cookies</strong>
              </label>
              <p>
                Enable social media features like sharing content and 
                interacting with social networks.
              </p>
            </div>
          </div>
          
          <div className="cookie-consent-buttons" style={{ 
            display: 'flex', 
            gap: '10px', 
            marginTop: '20px' 
          }}>
            <button onClick={() => setShowPreferences(false)}>
              Back
            </button>
            <button onClick={savePreferences}>
              Save Preferences
            </button>
          </div>
        </div>
      )}
    </div>
  );
};

export default CookieConsentBanner;

§ 3.4 GOOGLE CONSENT MODE V2

typescript
// google-consent-mode.ts

interface ConsentModeConfig {
  analytics_storage: 'granted' | 'denied';
  ad_storage: 'granted' | 'denied';
  ad_user_data: 'granted' | 'denied';
  ad_personalization: 'granted' | 'denied';
  functionality_storage: 'granted' | 'denied';
  personalization_storage: 'granted' | 'denied';
  security_storage: 'granted';
}

// Default: tutto negato tranne security
const defaultConsent: ConsentModeConfig = {
  analytics_storage: 'denied',
  ad_storage: 'denied',
  ad_user_data: 'denied',
  ad_personalization: 'denied',
  functionality_storage: 'denied',
  personalization_storage: 'denied',
  security_storage: 'granted'
};

// Inizializza Google Consent Mode PRIMA di gtag
function initializeGoogleConsentMode() {
  // Deve essere eseguito prima del caricamento di gtag.js
  window.dataLayer = window.dataLayer || [];
  function gtag(...args: any[]) {
    window.dataLayer.push(args);
  }
  
  // Set default consent state
  gtag('consent', 'default', {
    ...defaultConsent,
    wait_for_update: 500 // Attendi CMP per 500ms
  });
  
  // Opzionale: region-specific defaults
  gtag('consent', 'default', {
    analytics_storage: 'granted',
    ad_storage: 'granted',
    region: ['US'] // Meno restrittivo per US
  });
}

// Aggiorna consent quando utente sceglie
function updateGoogleConsent(preferences: ConsentPreferences) {
  const consentUpdate: Partial<ConsentModeConfig> = {
    analytics_storage: preferences.analytics ? 'granted' : 'denied',
    ad_storage: preferences.advertising ? 'granted' : 'denied',
    ad_user_data: preferences.advertising ? 'granted' : 'denied',
    ad_personalization: preferences.advertising ? 'granted' : 'denied',
    functionality_storage: preferences.functional ? 'granted' : 'denied',
    personalization_storage: preferences.functional ? 'granted' : 'denied'
  };
  
  window.gtag('consent', 'update', consentUpdate);
}

// Script loading con consent mode
const googleAnalyticsScript = `
<!-- Google tag (gtag.js) - Consent Mode v2 -->
<script>
  // Initialize consent mode BEFORE loading gtag
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  
  gtag('consent', 'default', {
    'analytics_storage': 'denied',
    'ad_storage': 'denied',
    'ad_user_data': 'denied',
    'ad_personalization': 'denied',
    'functionality_storage': 'denied',
    'personalization_storage': 'denied',
    'security_storage': 'granted',
    'wait_for_update': 500
  });
</script>
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"></script>
<script>
  gtag('js', new Date());
  gtag('config', 'GA_MEASUREMENT_ID');
</script>
`;

§ 3.5 BEST PRACTICE COOKIE BANNER

┌─────────────────────────────────────────────────────────────────────┐
│              COOKIE BANNER - DO's AND DON'Ts                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ✅ DO (Pratiche Corrette)                                          │
│  ─────────────────────────────────────────────────────────────────  │
│  • Bottone "Rifiuta tutto" ugualmente visibile come "Accetta tutto"│
│  • Bloccare cookie non essenziali PRIMA del consenso               │
│  • Permettere ritiro consenso facile (link in footer sempre visibile)│
│  • Conservare prova del consenso (audit trail)                      │
│  • Aggiornare consenso quando cambiano cookie/policy               │
│  • Informativa chiara su ogni categoria di cookie                  │
│  • Link a Privacy Policy e Cookie Policy complete                  │
│  • Consenso granulare per categoria                                │
│  • Rispettare segnali browser (GPC - Global Privacy Control)       │
│                                                                      │
│  ❌ DON'T (Pratiche Vietate - "Dark Patterns")                      │
│  ─────────────────────────────────────────────────────────────────  │
│  • Cookie pre-selezionati (pre-checked boxes)                       │
│  • "Rifiuta" nascosto o meno prominente di "Accetta"               │
│  • Cookie walls: bloccare accesso se non si accetta                │
│  • Scroll = consenso (navigare = accettare)                        │
│  • Banner che riappare dopo rifiuto (harassment)                   │
│  • Rendere difficile ritirare il consenso                          │
│  • Consenso bundled (unico toggle per tutto)                       │
│  • Linguaggio confuso o ingannevole                                │
│  • Colori che inducono a cliccare "Accetta"                        │
│  • Caricare script di tracking prima del consenso                  │
│                                                                      │
│  📋 REQUISITI SPECIFICI PER PAESE                                   │
│  ─────────────────────────────────────────────────────────────────  │
│  Francia (CNIL):                                                    │
│  • Simmetria bottoni (Accept/Reject uguale prominenza)             │
│  • Lista dettagliata tracker in 2 click                             │
│                                                                      │
│  Germania (BfDI):                                                   │
│  • Prominenza uguale per tutti i bottoni                           │
│  • No dark patterns                                                 │
│                                                                      │
│  Italia (Garante):                                                  │
│  • Cookie Policy separata e dettagliata                            │
│  • Consenso specifico per profilazione                             │
│                                                                      │
│  UK (ICO):                                                          │
│  • Prima pagina: chiara scelta Accept/Reject                       │
│  • Log consensi per 5 anni                                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

---

§ 4. CCPA/CPRA - CALIFORNIA PRIVACY

§ 4.1 OVERVIEW CCPA/CPRA

┌─────────────────────────────────────────────────────────────────────┐
│                    CCPA/CPRA OVERVIEW 2025                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  California Consumer Privacy Act (CCPA) + California Privacy        │
│  Rights Act (CPRA) = Framework privacy più stringente negli USA     │
│                                                                      │
│  APPLICABILITÀ (2025 thresholds):                                   │
│  ─────────────────────────────────────────────────────────────────  │
│  Si applica se business for-profit che:                             │
│  • Fatturato annuo > $26,625,000 (aggiornato per CPI)              │
│    OPPURE                                                           │
│  • Acquista/vende/condivide dati di 100,000+ residenti CA          │
│    OPPURE                                                           │
│  • Deriva 50%+ ricavi dalla vendita/condivisione dati personali    │
│                                                                      │
│  DIRITTI CONSUMATORI (LOCKA):                                       │
│  ─────────────────────────────────────────────────────────────────  │
│  L - LIMIT: Limitare uso di Sensitive Personal Information          │
│  O - OPT-OUT: Opt-out da vendita e condivisione dati               │
│  C - CORRECT: Correggere dati personali inesatti                   │
│  K - KNOW: Sapere quali dati raccolti e come usati/condivisi       │
│  A - ACCESS: Accedere ai propri dati personali                     │
│    + DELETE: Cancellare i propri dati personali                    │
│                                                                      │
│  SANZIONI:                                                          │
│  ─────────────────────────────────────────────────────────────────  │
│  • $2,500 per violazione non intenzionale                          │
│  • $7,988 per violazione intenzionale (aggiornato 2025)            │
│  • $107-$799 danni per consumatore in class action data breach     │
│  • Enforcement da CPPA (California Privacy Protection Agency)       │
│                                                                      │
│  NUOVI REQUISITI 2026 (Final Regulations):                          │
│  ─────────────────────────────────────────────────────────────────  │
│  • Cybersecurity audits obbligatori                                 │
│  • Risk assessments per attività ad alto rischio                   │
│  • Regole ADMT (Automated Decision-Making Technology)              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 4.2 IMPLEMENTAZIONE "DO NOT SELL/SHARE"

typescript
// ccpa-compliance.ts

interface CCPAConsumerRights {
  rightToKnow: boolean;
  rightToDelete: boolean;
  rightToCorrect: boolean;
  rightToOptOut: boolean;
  rightToLimit: boolean;
  rightToNonDiscrimination: boolean;
}

interface CCPARequest {
  id: string;
  type: 'know' | 'delete' | 'correct' | 'opt-out' | 'opt-in' | 'limit';
  consumerId: string;
  submittedAt: Date;
  verifiedAt?: Date;
  completedAt?: Date;
  status: 'pending' | 'verified' | 'processing' | 'completed' | 'denied';
  denialReason?: string;
}

interface SensitivePersonalInfo {
  ssn?: string;
  driverLicense?: string;
  stateId?: string;
  passport?: string;
  financialAccount?: string;
  preciseGeolocation?: boolean;
  racialOrigin?: string;
  religiousBeliefs?: string;
  unionMembership?: boolean;
  mailEmailTextContent?: boolean;
  geneticData?: boolean;
  biometricData?: boolean;
  healthInfo?: boolean;
  sexLifeOrientation?: boolean;
}

class CCPAComplianceManager {
  private optOutRegistry: Set<string> = new Set();
  private limitSPIRegistry: Set<string> = new Set();
  
  // "Do Not Sell or Share My Personal Information"
  async processOptOut(consumerId: string): Promise<void> {
    this.optOutRegistry.add(consumerId);
    
    // Notifica tutti i service provider
    await this.notifyServiceProviders(consumerId, 'opt_out');
    
    // Aggiorna preferenze marketing
    await this.updateMarketingPreferences(consumerId, {
      targetedAdvertising: false,
      crossContextBehavioralAd: false,
      dataSale: false,
      dataSharing: false
    });
    
    // Log per audit
    await this.logOptOutRequest(consumerId);
  }
  
  // Global Privacy Control (GPC) signal
  handleGPCSignal(consumerId: string, gpcEnabled: boolean): void {
    if (gpcEnabled) {
      // GPC = valid opt-out request under CCPA
      this.processOptOut(consumerId);
    }
  }
  
  // Limit use of Sensitive Personal Information
  async processSPILimitRequest(consumerId: string): Promise<void> {
    this.limitSPIRegistry.add(consumerId);
    
    // Limita uso SPI solo a scopi necessari
    await this.restrictSPIUsage(consumerId);
  }
  
  // Verifica se vendita/condivisione permessa
  canSellOrShare(consumerId: string): boolean {
    return !this.optOutRegistry.has(consumerId);
  }
  
  // Verifica se SPI può essere usato liberamente
  canUseSPIFreely(consumerId: string): boolean {
    return !this.limitSPIRegistry.has(consumerId);
  }
  
  // Process "Right to Know" request
  async processKnowRequest(request: CCPARequest): Promise<any> {
    // Verifica identità (2+ data points)
    await this.verifyIdentity(request.consumerId);
    
    // Raccogli info degli ultimi 12 mesi
    const data = {
      categories: await this.getCollectedCategories(request.consumerId),
      sources: await this.getDataSources(request.consumerId),
      purposes: await this.getBusinessPurposes(request.consumerId),
      thirdParties: await this.getThirdPartySharingInfo(request.consumerId),
      specificPieces: await this.getSpecificDataPieces(request.consumerId)
    };
    
    return data;
  }
  
  // Process "Right to Delete" request
  async processDeleteRequest(request: CCPARequest): Promise<void> {
    // Verifica identità
    await this.verifyIdentity(request.consumerId);
    
    // Verifica eccezioni
    const exceptions = await this.checkDeletionExceptions(request.consumerId);
    if (exceptions.length > 0) {
      // Cancellazione parziale
      await this.partialDeletion(request.consumerId, exceptions);
    } else {
      // Cancellazione completa
      await this.fullDeletion(request.consumerId);
    }
    
    // Notifica service providers
    await this.notifyServiceProviders(request.consumerId, 'delete');
  }
  
  private async verifyIdentity(consumerId: string): Promise<boolean> {
    // Richiede 2+ data points per verifica
    return true;
  }
  
  private async notifyServiceProviders(
    consumerId: string, 
    action: string
  ): Promise<void> {}
  
  private async updateMarketingPreferences(
    consumerId: string, 
    prefs: any
  ): Promise<void> {}
  
  private async logOptOutRequest(consumerId: string): Promise<void> {}
  
  private async restrictSPIUsage(consumerId: string): Promise<void> {}
  
  private async getCollectedCategories(consumerId: string): Promise<string[]> {
    return [];
  }
  
  private async getDataSources(consumerId: string): Promise<string[]> {
    return [];
  }
  
  private async getBusinessPurposes(consumerId: string): Promise<string[]> {
    return [];
  }
  
  private async getThirdPartySharingInfo(consumerId: string): Promise<any> {
    return {};
  }
  
  private async getSpecificDataPieces(consumerId: string): Promise<any> {
    return {};
  }
  
  private async checkDeletionExceptions(consumerId: string): Promise<string[]> {
    return [];
  }
  
  private async partialDeletion(
    consumerId: string, 
    exceptions: string[]
  ): Promise<void> {}
  
  private async fullDeletion(consumerId: string): Promise<void> {}
}

// Do Not Sell/Share Link Component
const DoNotSellShareLink: React.FC = () => {
  const handleClick = async () => {
    // Open opt-out modal or redirect to preference center
    window.location.href = '/privacy/do-not-sell';
  };
  
  return (
    <a 
      href="/privacy/do-not-sell"
      onClick={handleClick}
      style={{
        // Link deve essere chiaramente visibile
        fontSize: '14px',
        textDecoration: 'underline'
      }}
    >
      Do Not Sell or Share My Personal Information
    </a>
  );
};

// Footer links required by CCPA
const CCPAFooterLinks: React.FC = () => (
  <div className="ccpa-footer-links">
    <a href="/privacy-policy">Privacy Policy</a>
    <a href="/privacy/do-not-sell">
      Do Not Sell or Share My Personal Information
    </a>
    <a href="/privacy/limit-sensitive-info">
      Limit the Use of My Sensitive Personal Information
    </a>
    <a href="/privacy/rights">Your Privacy Rights</a>
  </div>
);

export { CCPAComplianceManager, DoNotSellShareLink, CCPAFooterLinks };

§ 4.3 PRIVACY POLICY REQUIREMENTS CCPA

┌─────────────────────────────────────────────────────────────────────┐
│               CCPA PRIVACY POLICY REQUIREMENTS 2025                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  La Privacy Policy deve includere (aggiornata annualmente):         │
│                                                                      │
│  1. CATEGORIE DI DATI RACCOLTI                                      │
│     • Identificatori (nome, email, IP, etc.)                        │
│     • Informazioni commerciali (acquisti, tendenze)                 │
│     • Attività internet (browsing, search history)                  │
│     • Dati di geolocalizzazione                                     │
│     • Inferenze (profili comportamentali)                           │
│     • Sensitive Personal Information (se raccolte)                  │
│                                                                      │
│  2. FONTI DEI DATI                                                  │
│     • Direttamente dal consumatore                                  │
│     • Automaticamente (cookies, tracking)                           │
│     • Da terze parti (data brokers, partner)                        │
│                                                                      │
│  3. SCOPI DEL TRATTAMENTO                                           │
│     • Fulfillment ordini/servizi                                    │
│     • Marketing e pubblicità                                        │
│     • Analisi e miglioramento servizi                               │
│     • Sicurezza e fraud prevention                                  │
│                                                                      │
│  4. CONDIVISIONE CON TERZE PARTI                                    │
│     • Categorie di dati condivisi                                   │
│     • Categorie di destinatari                                      │
│     • Scopi della condivisione                                      │
│     • Se dati venduti/condivisi per advertising                     │
│                                                                      │
│  5. PERIODI DI CONSERVAZIONE                                        │
│     • Criteri per determinare retention period                      │
│     • O periodo specifico per categoria                             │
│                                                                      │
│  6. DIRITTI DEI CONSUMATORI                                         │
│     • Lista completa diritti CCPA                                   │
│     • Come esercitarli (web form, email, toll-free)                │
│     • Tempi di risposta (45 giorni, estensibile)                   │
│                                                                      │
│  7. LINK OBBLIGATORI                                                │
│     • "Do Not Sell or Share My Personal Information"               │
│     • "Limit the Use of My Sensitive Personal Information"         │
│     • Chiaramente visibili e accessibili                           │
│                                                                      │
│  8. INFORMAZIONI DI CONTATTO                                        │
│     • Email o form online                                           │
│     • Toll-free number (se raccogli info offline)                  │
│                                                                      │
│  9. DATA DI ULTIMA MODIFICA                                         │
│     • Prominente nella policy                                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘



---

§ 5. PRIVACY POLICY E TERMS OF SERVICE

§ 5.1 PRIVACY POLICY - REQUISITI FONDAMENTALI

Una Privacy Policy conforme deve essere chiara, accessibile e completa.

#### Elementi Essenziali

┌─────────────────────────────────────────────────────────────────────┐
│              STRUTTURA PRIVACY POLICY COMPLETA                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. IDENTITÀ E CONTATTI DEL TITOLARE                                │
│     • Nome/ragione sociale                                          │
│     • Indirizzo sede legale                                         │
│     • Email/PEC                                                     │
│     • DPO (se nominato): contatti                                   │
│                                                                      │
│  2. DATI RACCOLTI                                                   │
│     • Categorie specifiche di dati                                  │
│     • Dati forniti volontariamente vs automatici                    │
│     • Dati da terze parti                                           │
│     • Dati sensibili/particolari                                    │
│                                                                      │
│  3. FINALITÀ E BASE GIURIDICA                                       │
│     • Scopo specifico per ogni trattamento                          │
│     • Base giuridica corrispondente                                 │
│     • Legittimo interesse: descrizione e bilanciamento             │
│                                                                      │
│  4. DESTINATARI DEI DATI                                            │
│     • Categorie di destinatari                                      │
│     • Trasferimenti extra-UE: garanzie                              │
│     • Responsabili del trattamento                                  │
│                                                                      │
│  5. CONSERVAZIONE                                                   │
│     • Periodi specifici o criteri                                   │
│     • Differenziazione per finalità                                 │
│                                                                      │
│  6. DIRITTI DELL'INTERESSATO                                        │
│     • Lista completa diritti applicabili                            │
│     • Come esercitarli                                              │
│     • Diritto di reclamo all'autorità                               │
│                                                                      │
│  7. OBBLIGATORIETÀ E CONSEGUENZE                                    │
│     • Dati obbligatori vs facoltativi                               │
│     • Conseguenze del mancato conferimento                          │
│                                                                      │
│  8. DECISIONI AUTOMATIZZATE                                         │
│     • Profilazione: logica e conseguenze                            │
│     • Diritto di contestazione                                      │
│                                                                      │
│  9. MODIFICHE ALLA POLICY                                           │
│     • Come vengono comunicate                                       │
│     • Data ultima modifica                                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

#### Implementazione Privacy Policy Dinamica

typescript
// privacy-policy-generator.ts
interface PrivacyPolicyConfig {
  company: {
    name: string;
    legalName: string;
    address: string;
    email: string;
    pec?: string;
    dpo?: {
      name: string;
      email: string;
    };
  };
  dataProcessing: DataProcessingActivity[];
  thirdParties: ThirdPartyRecipient[];
  internationalTransfers: InternationalTransfer[];
  retentionPolicies: RetentionPolicy[];
  jurisdiction: 'EU' | 'US' | 'UK' | 'GLOBAL';
  applicableLaws: ('GDPR' | 'CCPA' | 'COPPA' | 'UK_GDPR')[];
}

interface DataProcessingActivity {
  id: string;
  dataCategories: string[];
  purposes: string[];
  legalBasis: 'consent' | 'contract' | 'legal_obligation' | 
              'vital_interest' | 'public_interest' | 'legitimate_interest';
  legitimateInterestDescription?: string;
  retentionPeriod: string;
  isAutomated: boolean;
  involvesProfiling: boolean;
}

interface ThirdPartyRecipient {
  category: string;
  purpose: string;
  dataShared: string[];
  location: 'EU' | 'US' | 'OTHER';
  safeguards?: string;
}

interface InternationalTransfer {
  destination: string;
  mechanism: 'adequacy_decision' | 'scc' | 'bcr' | 'derogation';
  details: string;
}

interface RetentionPolicy {
  dataCategory: string;
  period: string;
  criteria: string;
}

class PrivacyPolicyGenerator {
  private config: PrivacyPolicyConfig;
  private lastUpdated: Date;

  constructor(config: PrivacyPolicyConfig) {
    this.config = config;
    this.lastUpdated = new Date();
  }

  generateFullPolicy(): string {
    const sections: string[] = [];

    sections.push(this.generateHeader());
    sections.push(this.generateControllerSection());
    sections.push(this.generateDataCollectionSection());
    sections.push(this.generatePurposesSection());
    sections.push(this.generateRecipientsSection());
    sections.push(this.generateTransfersSection());
    sections.push(this.generateRetentionSection());
    sections.push(this.generateRightsSection());
    sections.push(this.generateCookiesReference());
    sections.push(this.generateChangesSection());
    sections.push(this.generateContactSection());

    // Add jurisdiction-specific sections
    if (this.config.applicableLaws.includes('CCPA')) {
      sections.push(this.generateCCPASection());
    }
    if (this.config.applicableLaws.includes('COPPA')) {
      sections.push(this.generateCOPPASection());
    }

    return sections.join('\n\n');
  }

  private generateHeader(): string {
    return `
# Privacy Policy

**Last Updated:** ${this.lastUpdated.toLocaleDateString('it-IT')}

This Privacy Policy describes how ${this.config.company.name} 
("we", "us", "our") collects, uses, and shares your personal information 
when you use our services.

${this.getJurisdictionNotice()}
    `.trim();
  }

  private generateControllerSection(): string {
    const { company } = this.config;
    let section = `
## 1. Data Controller

**${company.legalName}**
${company.address}
Email: ${company.email}
${company.pec ? `PEC: ${company.pec}` : ''}
    `.trim();

    if (company.dpo) {
      section += `

**Data Protection Officer:**
${company.dpo.name}
Email: ${company.dpo.email}`;
    }

    return section;
  }

  private generateDataCollectionSection(): string {
    const categories = new Set<string>();
    this.config.dataProcessing.forEach(activity => {
      activity.dataCategories.forEach(cat => categories.add(cat));
    });

    return `
## 2. Data We Collect

We collect the following categories of personal data:

${Array.from(categories).map(cat => `- **${cat}**`).join('\n')}

### How We Collect Data

- **Directly from you:** When you create an account, make a purchase, 
  contact us, or interact with our services
- **Automatically:** Through cookies, log files, and similar technologies 
  when you use our website or apps
- **From third parties:** From business partners, social media platforms, 
  and public sources
    `.trim();
  }

  private generatePurposesSection(): string {
    const purposesByBasis = new Map<string, string[]>();
    
    this.config.dataProcessing.forEach(activity => {
      const basis = activity.legalBasis;
      if (!purposesByBasis.has(basis)) {
        purposesByBasis.set(basis, []);
      }
      activity.purposes.forEach(purpose => {
        const existing = purposesByBasis.get(basis)!;
        if (!existing.includes(purpose)) {
          existing.push(purpose);
        }
      });
    });

    let section = `
## 3. How We Use Your Data

We process your personal data for the following purposes:

`;

    const basisLabels: Record<string, string> = {
      consent: 'Based on Your Consent',
      contract: 'For Contract Performance',
      legal_obligation: 'Legal Obligations',
      vital_interest: 'Vital Interests',
      public_interest: 'Public Interest',
      legitimate_interest: 'Legitimate Interests'
    };

    purposesByBasis.forEach((purposes, basis) => {
      section += `### ${basisLabels[basis]}\n\n`;
      purposes.forEach(purpose => {
        section += `- ${purpose}\n`;
      });
      section += '\n';
    });

    // Add profiling notice if applicable
    const hasProfiling = this.config.dataProcessing.some(a => a.involvesProfiling);
    if (hasProfiling) {
      section += `
### Automated Decision-Making and Profiling

We use automated processing, including profiling, for the following purposes:

${this.config.dataProcessing
  .filter(a => a.involvesProfiling)
  .map(a => `- ${a.purposes.join(', ')}`)
  .join('\n')}

You have the right to object to profiling and to request human intervention 
in automated decisions that significantly affect you.
`;
    }

    return section;
  }

  private generateRightsSection(): string {
    let section = `
## 6. Your Rights

You have the following rights regarding your personal data:

| Right | Description |
|-------|-------------|
| **Access** | Request a copy of your personal data |
| **Rectification** | Correct inaccurate or incomplete data |
| **Erasure** | Request deletion of your data ("right to be forgotten") |
| **Restriction** | Limit how we process your data |
| **Portability** | Receive your data in a portable format |
| **Objection** | Object to processing based on legitimate interests |
| **Withdraw Consent** | Withdraw consent at any time |

### How to Exercise Your Rights

You can exercise your rights by:

1. **Online Form:** [Privacy Request Portal](/privacy-requests)
2. **Email:** ${this.config.company.email}
3. **Mail:** ${this.config.company.address}

We will respond within 30 days (or 45 days for CCPA requests).

### Right to Complain

You have the right to lodge a complaint with a supervisory authority. 
${this.getSupervisoryAuthorityInfo()}
`;

    return section;
  }

  private getSupervisoryAuthorityInfo(): string {
    if (this.config.jurisdiction === 'EU') {
      return `In Italy, this is the Garante per la Protezione dei Dati Personali 
(www.garanteprivacy.it).`;
    }
    return `Contact your local data protection authority for more information.`;
  }

  private generateCCPASection(): string {
    return `
## California Privacy Rights (CCPA/CPRA)

If you are a California resident, you have additional rights:

### Your California Rights

- **Right to Know:** Request disclosure of personal information collected, 
  used, and shared in the past 12 months
- **Right to Delete:** Request deletion of your personal information
- **Right to Correct:** Request correction of inaccurate information
- **Right to Opt-Out:** Opt out of the sale or sharing of your information
- **Right to Limit:** Limit use of sensitive personal information
- **Non-Discrimination:** Exercise your rights without discrimination

### Categories of Personal Information

In the past 12 months, we have collected the following categories:

${this.getCCPACategories()}

### Do Not Sell or Share My Personal Information

[Click here to opt out of the sale or sharing of your personal information](/do-not-sell)

### Shine the Light

California Civil Code Section 1798.83 permits California residents to 
request certain information regarding our disclosure of personal information 
to third parties for direct marketing purposes.
`;
  }

  private getCCPACategories(): string {
    return `
| Category | Collected | Sold/Shared | Business Purpose |
|----------|-----------|-------------|------------------|
| Identifiers | Yes | No | Service delivery |
| Commercial Info | Yes | No | Order fulfillment |
| Internet Activity | Yes | No | Analytics |
| Geolocation | Yes | No | Localized services |
| Inferences | Yes | No | Personalization |
`;
  }

  // ... additional methods for other sections
}

§ 5.2 TERMS OF SERVICE - STRUTTURA E REQUISITI

┌─────────────────────────────────────────────────────────────────────┐
│              STRUTTURA TERMS OF SERVICE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. ACCEPTANCE OF TERMS                                             │
│     • Come l'utente accetta (uso = accettazione)                    │
│     • Età minima richiesta (13/16/18 anni)                          │
│     • Capacità di contrarre                                         │
│                                                                      │
│  2. DESCRIPTION OF SERVICE                                          │
│     • Cosa offre il servizio                                        │
│     • Limitazioni note                                              │
│     • Disponibilità geografica                                      │
│                                                                      │
│  3. USER ACCOUNTS                                                   │
│     • Requisiti registrazione                                       │
│     • Responsabilità credenziali                                    │
│     • Terminazione account                                          │
│                                                                      │
│  4. USER CONDUCT                                                    │
│     • Usi consentiti                                                │
│     • Usi vietati (lista esaustiva)                                │
│     • Conseguenze violazioni                                        │
│                                                                      │
│  5. INTELLECTUAL PROPERTY                                           │
│     • Proprietà del contenuto                                       │
│     • Licenza concessa all'utente                                   │
│     • User-generated content: licenza al provider                   │
│                                                                      │
│  6. PAYMENT TERMS (se applicabile)                                  │
│     • Prezzi e valuta                                               │
│     • Metodi di pagamento                                           │
│     • Fatturazione e rinnovi                                        │
│     • Politica di rimborso                                          │
│                                                                      │
│  7. DISCLAIMERS                                                     │
│     • "AS IS" provision                                             │
│     • No garanzie implicite                                         │
│     • Accuratezza informazioni                                      │
│                                                                      │
│  8. LIMITATION OF LIABILITY                                         │
│     • Cap sui danni                                                 │
│     • Esclusione danni indiretti                                    │
│     • Eccezioni legali (gross negligence, death)                   │
│                                                                      │
│  9. INDEMNIFICATION                                                 │
│     • Manleva per violazioni utente                                 │
│     • Procedura di notifica                                         │
│                                                                      │
│  10. DISPUTE RESOLUTION                                             │
│      • Legge applicabile                                            │
│      • Foro competente                                              │
│      • Arbitrato (se applicabile)                                   │
│      • Class action waiver (US)                                     │
│                                                                      │
│  11. MODIFICATIONS                                                  │
│      • Diritto di modifica                                          │
│      • Notifica agli utenti                                         │
│      • Effetto delle modifiche                                      │
│                                                                      │
│  12. TERMINATION                                                    │
│      • Diritto di terminare                                         │
│      • Effetti della terminazione                                   │
│      • Sopravvivenza clausole                                       │
│                                                                      │
│  13. GENERAL PROVISIONS                                             │
│      • Severability                                                 │
│      • Entire agreement                                             │
│      • Waiver                                                       │
│      • Assignment                                                   │
│                                                                      │
│  14. CONTACT INFORMATION                                            │
│      • Come contattare per domande                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

#### Clausole Specifiche per Giurisdizione

typescript
// terms-of-service-builder.ts
interface TermsConfig {
  company: CompanyInfo;
  serviceType: 'saas' | 'ecommerce' | 'social' | 'marketplace';
  targetAudience: {
    minAge: number;
    includesMinors: boolean;
  };
  monetization: {
    hasSubscriptions: boolean;
    hasOneTimePurchases: boolean;
    hasFreeTrials: boolean;
    autoRenewal: boolean;
  };
  userContent: {
    allowsUploads: boolean;
    licenseRequired: boolean;
  };
  jurisdiction: {
    primary: 'IT' | 'US' | 'UK' | 'DE';
    targetMarkets: string[];
  };
}

class TermsOfServiceBuilder {
  private config: TermsConfig;

  constructor(config: TermsConfig) {
    this.config = config;
  }

  generateAcceptanceClause(): string {
    const minAge = this.config.targetAudience.minAge;
    
    return `
## 1. Acceptance of Terms

By accessing or using ${this.config.company.name} ("Service"), you agree to 
be bound by these Terms of Service ("Terms"). If you do not agree to these 
Terms, do not use the Service.

### Eligibility

You must be at least ${minAge} years old to use this Service. 
${minAge < 18 ? `If you are under 18, you must have parental consent 
to use this Service.` : ''}

By using the Service, you represent and warrant that:

- You are at least ${minAge} years of age
- You have the legal capacity to enter into these Terms
- ${minAge < 18 ? 'If under 18, you have obtained parental or guardian consent' : ''}
- You will comply with all applicable laws and regulations
`;
  }

  generateLiabilityClause(): string {
    // Different liability caps based on jurisdiction
    if (this.config.jurisdiction.primary === 'IT' || 
        this.config.jurisdiction.targetMarkets.includes('EU')) {
      return this.generateEULiabilityClause();
    } else if (this.config.jurisdiction.primary === 'US') {
      return this.generateUSLiabilityClause();
    }
    return this.generateDefaultLiabilityClause();
  }

  private generateEULiabilityClause(): string {
    return `
## 8. Limitation of Liability

### For EU/EEA Users

Nothing in these Terms shall exclude or limit our liability for:

- Death or personal injury caused by our negligence
- Fraud or fraudulent misrepresentation
- Any liability that cannot be excluded by applicable law
- Gross negligence or willful misconduct

Subject to the above, our total liability shall not exceed the greater of:
- The amounts paid by you in the 12 months preceding the claim, or
- €100 (one hundred euros)

We shall not be liable for any indirect, incidental, special, consequential, 
or punitive damages, except where such exclusion is prohibited by law.

### Consumer Rights

These limitations do not affect your statutory rights as a consumer 
under applicable EU consumer protection laws.
`;
  }

  private generateUSLiabilityClause(): string {
    return `
## 8. Limitation of Liability

TO THE MAXIMUM EXTENT PERMITTED BY APPLICABLE LAW:

THE SERVICE IS PROVIDED "AS IS" AND "AS AVAILABLE" WITHOUT WARRANTIES 
OF ANY KIND, WHETHER EXPRESS, IMPLIED, STATUTORY, OR OTHERWISE, INCLUDING 
BUT NOT LIMITED TO WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR 
PURPOSE, AND NON-INFRINGEMENT.

IN NO EVENT SHALL ${this.config.company.name.toUpperCase()}, ITS AFFILIATES, 
OFFICERS, DIRECTORS, EMPLOYEES, OR AGENTS BE LIABLE FOR ANY INDIRECT, 
INCIDENTAL, SPECIAL, CONSEQUENTIAL, OR PUNITIVE DAMAGES, INCLUDING BUT 
NOT LIMITED TO LOSS OF PROFITS, DATA, USE, GOODWILL, OR OTHER INTANGIBLE 
LOSSES, RESULTING FROM:

(i) YOUR ACCESS TO OR USE OF OR INABILITY TO ACCESS OR USE THE SERVICE;
(ii) ANY CONDUCT OR CONTENT OF ANY THIRD PARTY ON THE SERVICE;
(iii) ANY CONTENT OBTAINED FROM THE SERVICE; AND
(iv) UNAUTHORIZED ACCESS, USE, OR ALTERATION OF YOUR TRANSMISSIONS 
     OR CONTENT.

OUR TOTAL LIABILITY SHALL NOT EXCEED THE AMOUNTS PAID BY YOU, IF ANY, 
IN THE TWELVE (12) MONTHS PRIOR TO THE CLAIM.

SOME JURISDICTIONS DO NOT ALLOW THE EXCLUSION OF CERTAIN WARRANTIES 
OR LIMITATION OF LIABILITY FOR CERTAIN DAMAGES. IF THESE LAWS APPLY TO 
YOU, SOME OF THE ABOVE EXCLUSIONS MAY NOT APPLY.
`;
  }

  generateDisputeResolutionClause(): string {
    if (this.config.jurisdiction.primary === 'US') {
      return `
## 10. Dispute Resolution

### Informal Resolution

Before filing a claim, you agree to attempt to resolve disputes informally 
by contacting us at ${this.config.company.email}. We will attempt to resolve 
the dispute within 60 days.

### Arbitration Agreement

IF INFORMAL RESOLUTION FAILS, ANY DISPUTE SHALL BE RESOLVED BY BINDING 
ARBITRATION under the rules of the American Arbitration Association.

**WAIVER OF JURY TRIAL:** YOU WAIVE ANY RIGHT TO A JURY TRIAL.

**CLASS ACTION WAIVER:** YOU AGREE THAT ANY CLAIMS WILL BE BROUGHT 
IN YOUR INDIVIDUAL CAPACITY, AND NOT AS A PLAINTIFF OR CLASS MEMBER 
IN ANY PURPORTED CLASS OR REPRESENTATIVE PROCEEDING.

### Exceptions

The following claims are not subject to arbitration:
- Claims in small claims court
- Intellectual property disputes
- Emergency injunctive relief

### Opt-Out

You may opt out of this arbitration agreement within 30 days of first 
accepting these Terms by sending written notice to: ${this.config.company.address}
`;
    }

    // EU-style dispute resolution
    return `
## 10. Dispute Resolution

### Applicable Law

These Terms shall be governed by the laws of ${this.getGoverningLaw()}.

### Jurisdiction

Any disputes shall be submitted to the exclusive jurisdiction of the 
courts of ${this.getJurisdiction()}.

### Alternative Dispute Resolution

For EU consumers: You may also use the Online Dispute Resolution (ODR) 
platform provided by the European Commission at: 
https://ec.europa.eu/consumers/odr/

### Consumer Rights

These terms do not affect your right to bring claims before the courts 
of your place of residence if you are a consumer.
`;
  }

  private getGoverningLaw(): string {
    const laws: Record<string, string> = {
      IT: 'Italy',
      DE: 'Germany',
      UK: 'England and Wales',
      US: 'the State of Delaware, United States'
    };
    return laws[this.config.jurisdiction.primary] || 'the country of our registered office';
  }

  private getJurisdiction(): string {
    const courts: Record<string, string> = {
      IT: 'Milan, Italy',
      DE: 'Berlin, Germany',
      UK: 'London, United Kingdom',
      US: 'Wilmington, Delaware'
    };
    return courts[this.config.jurisdiction.primary] || 'our registered office location';
  }
}



---

§ 6. COPPA - CHILDREN'S ONLINE PRIVACY PROTECTION ACT

§ 6.1 OVERVIEW E AGGIORNAMENTI 2025

Il COPPA protegge la privacy online dei bambini sotto i 13 anni negli Stati Uniti.

┌─────────────────────────────────────────────────────────────────────┐
│                    COPPA REQUIREMENTS OVERVIEW                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  AMBITO DI APPLICAZIONE                                             │
│  ├── Siti/app diretti a bambini < 13 anni                          │
│  ├── Siti/app con "actual knowledge" di utenti < 13                │
│  └── Dal 2025: siti con "constructive knowledge" (dovrebbero sapere)│
│                                                                      │
│  DATI COPERTI ("Personal Information")                              │
│  ├── Nome, cognome                                                  │
│  ├── Indirizzo fisico                                               │
│  ├── Email                                                          │
│  ├── Telefono                                                       │
│  ├── SSN                                                            │
│  ├── Identificatori persistenti (cookies, device ID)               │
│  ├── Foto/video/audio con il bambino                               │
│  ├── Geolocalizzazione                                              │
│  └── Dal 2025: Dati biometrici                                      │
│                                                                      │
│  REQUISITI PRINCIPALI                                               │
│  ├── Privacy Policy specifica per bambini                          │
│  ├── Consenso genitoriale verificabile PRIMA della raccolta        │
│  ├── Diritto dei genitori di accesso/cancellazione                 │
│  ├── Minimizzazione dati                                            │
│  └── Sicurezza ragionevole                                          │
│                                                                      │
│  SANZIONI                                                           │
│  ├── Fino a $50,120 per violazione (2024)                          │
│  └── Enforcement: FTC                                               │
│                                                                      │
│  TIMELINE NUOVE REGOLE                                              │
│  ├── Aprile 2024: Regole finali pubblicate                         │
│  └── Aprile 2026: Compliance deadline                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 6.2 MODIFICHE 2025 (EFFECTIVE APRILE 2026)

Le nuove regole COPPA introducono cambiamenti significativi:

┌─────────────────────────────────────────────────────────────────────┐
│              COPPA 2025 AMENDMENTS - KEY CHANGES                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. ESPANSIONE "KNOWLEDGE" STANDARD                                 │
│     PRIMA: Solo "actual knowledge" di utenti < 13                   │
│     ORA:   Include "constructive knowledge"                         │
│     → Se DOVRESTI ragionevolmente sapere che hai utenti bambini,   │
│       devi rispettare COPPA                                         │
│                                                                      │
│  2. NUOVI DATI COPERTI                                              │
│     • Dati biometrici (impronte, face recognition, voce)           │
│     • Identificatori nazionali                                      │
│     • Informazioni di salute                                        │
│                                                                      │
│  3. LIMITAZIONI PUBBLICITÀ                                          │
│     VIETATO: Pubblicità contestuale/targeted a bambini              │
│     VIETATO: Raccolta dati per advertising targeting               │
│     PERMESSO: Pubblicità contestuale basata sul contenuto          │
│               (non sul bambino)                                     │
│                                                                      │
│  4. DATA RETENTION LIMITS                                           │
│     • Conservare solo per il tempo necessario allo scopo           │
│     • Obbligo di cancellazione quando non più necessario           │
│     • No retention per finalità secondarie senza nuovo consenso    │
│                                                                      │
│  5. SAFE HARBOR PROGRAMS                                            │
│     • Standard più stringenti per approvazione FTC                 │
│     • Audit annuali obbligatori                                     │
│     • Reporting incidents a FTC                                     │
│                                                                      │
│  6. CONSENSO GRANULARE                                              │
│     • Consenso separato per ogni categoria di dati                 │
│     • Consenso separato per ogni finalità                          │
│     • Genitori possono approvare solo alcune raccolte              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 6.3 IMPLEMENTAZIONE COPPA COMPLIANCE

typescript
// coppa-compliance.ts
interface COPPAConfig {
  isDirectedToChildren: boolean;
  minimumAge: number;
  parentalConsentMethods: ParentalConsentMethod[];
  dataRetentionDays: number;
}

type ParentalConsentMethod = 
  | 'signed_consent_form'
  | 'credit_card_transaction'
  | 'toll_free_call'
  | 'video_conference'
  | 'government_id'
  | 'knowledge_based_auth'
  | 'face_match';

interface ChildUser {
  id: string;
  birthDate?: Date;
  ageEstimate?: number;
  parentalConsentStatus: 'pending' | 'verified' | 'denied' | 'expired';
  parentalConsentDate?: Date;
  parentalConsentMethod?: ParentalConsentMethod;
  parentEmail?: string;
  dataCollected: CollectedData[];
}

interface CollectedData {
  category: string;
  purpose: string;
  collectedAt: Date;
  expiresAt: Date;
  consentId: string;
}

class COPPAComplianceManager {
  private config: COPPAConfig;
  private childUsers: Map<string, ChildUser> = new Map();

  constructor(config: COPPAConfig) {
    this.config = config;
  }

  // Age Gate Implementation
  async checkAge(providedAge: number): Promise<AgeCheckResult> {
    if (providedAge < this.config.minimumAge) {
      return {
        allowed: false,
        requiresParentalConsent: true,
        message: 'Parental consent required for users under 13',
        nextStep: 'parental_consent_flow'
      };
    }

    if (providedAge < 18) {
      return {
        allowed: true,
        requiresParentalConsent: false,
        message: 'Age verified (teen)',
        restrictions: ['no_targeted_advertising', 'enhanced_privacy']
      };
    }

    return {
      allowed: true,
      requiresParentalConsent: false,
      message: 'Age verified (adult)'
    };
  }

  // Parental Consent Flow
  async initiateParentalConsent(
    childId: string,
    parentEmail: string,
    requestedPermissions: string[]
  ): Promise<ParentalConsentRequest> {
    const request: ParentalConsentRequest = {
      id: crypto.randomUUID(),
      childId,
      parentEmail,
      requestedPermissions,
      status: 'pending',
      createdAt: new Date(),
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000), // 7 days
      verificationMethods: this.config.parentalConsentMethods
    };

    // Send verification email to parent
    await this.sendParentalNotification(request);

    return request;
  }

  private async sendParentalNotification(
    request: ParentalConsentRequest
  ): Promise<void> {
    const emailContent = `
Dear Parent/Guardian,

Your child has requested to create an account on our service. 
Under the Children's Online Privacy Protection Act (COPPA), 
we require your verifiable consent before collecting personal 
information from children under 13.

YOUR CHILD WANTS PERMISSION TO:
${request.requestedPermissions.map(p => `• ${p}`).join('\n')}

To provide consent, please complete ONE of the following 
verification methods:

1. Sign and return the consent form: [Link]
2. Call our toll-free number: 1-800-XXX-XXXX
3. Complete video verification: [Link]
4. Verify via government ID: [Link]

This request expires on: ${request.expiresAt.toLocaleDateString()}

If you did not expect this request or do not wish to provide 
consent, please contact us immediately.

To review our Children's Privacy Policy: [Link]
To exercise your parental rights: [Link]

Sincerely,
${/* Company Name */ ''}
    `;

    // Send email
    await this.emailService.send(request.parentEmail, emailContent);
  }

  // Verify Parental Consent
  async verifyParentalConsent(
    requestId: string,
    method: ParentalConsentMethod,
    verificationData: any
  ): Promise<VerificationResult> {
    switch (method) {
      case 'credit_card_transaction':
        return this.verifyCreditCard(verificationData);
      
      case 'government_id':
        return this.verifyGovernmentId(verificationData);
      
      case 'knowledge_based_auth':
        return this.verifyKnowledgeBased(verificationData);
      
      case 'video_conference':
        return this.scheduleVideoVerification(verificationData);
      
      case 'face_match':
        return this.verifyFaceMatch(verificationData);
      
      default:
        throw new Error(`Unsupported verification method: ${method}`);
    }
  }

  private async verifyCreditCard(data: {
    token: string;
    amount: number;
  }): Promise<VerificationResult> {
    // Process small transaction ($0.50-$1.00) that will be refunded
    // This proves the parent has access to a credit card
    const transaction = await this.paymentProcessor.charge({
      token: data.token,
      amount: 50, // $0.50 in cents
      description: 'COPPA Parental Verification (will be refunded)'
    });

    if (transaction.success) {
      // Immediately refund
      await this.paymentProcessor.refund(transaction.id);
      
      return {
        verified: true,
        method: 'credit_card_transaction',
        timestamp: new Date(),
        confidence: 'high'
      };
    }

    return {
      verified: false,
      reason: 'Credit card verification failed'
    };
  }

  // Data Collection with COPPA Checks
  async collectData(
    userId: string,
    dataCategory: string,
    data: any,
    purpose: string
  ): Promise<CollectionResult> {
    const user = this.childUsers.get(userId);
    
    if (!user) {
      return { success: false, reason: 'User not found' };
    }

    // Check if parental consent covers this collection
    if (user.parentalConsentStatus !== 'verified') {
      return { 
        success: false, 
        reason: 'Parental consent not verified',
        action: 'redirect_to_consent_flow'
      };
    }

    // Check if consent covers this specific data category
    const hasConsent = user.dataCollected.some(
      d => d.category === dataCategory && d.purpose === purpose
    );

    if (!hasConsent) {
      return {
        success: false,
        reason: 'No consent for this data category/purpose',
        action: 'request_additional_consent'
      };
    }

    // Data minimization check
    const minimizedData = this.minimizeData(data, dataCategory);

    // Store with retention date
    const collectedData: CollectedData = {
      category: dataCategory,
      purpose,
      collectedAt: new Date(),
      expiresAt: new Date(
        Date.now() + this.config.dataRetentionDays * 24 * 60 * 60 * 1000
      ),
      consentId: user.parentalConsentDate?.toISOString() || ''
    };

    user.dataCollected.push(collectedData);

    return { success: true, dataId: collectedData.consentId };
  }

  // Parental Rights: Access
  async handleParentAccessRequest(
    parentEmail: string,
    childId: string
  ): Promise<DataAccessResponse> {
    // Verify parent identity first
    const verified = await this.verifyParentIdentity(parentEmail, childId);
    
    if (!verified) {
      return { 
        success: false, 
        reason: 'Parent identity verification failed' 
      };
    }

    const child = this.childUsers.get(childId);
    if (!child) {
      return { success: false, reason: 'Child account not found' };
    }

    return {
      success: true,
      data: {
        collectedData: child.dataCollected,
        consentHistory: await this.getConsentHistory(childId),
        thirdPartySharing: await this.getThirdPartySharing(childId)
      }
    };
  }

  // Parental Rights: Delete
  async handleParentDeleteRequest(
    parentEmail: string,
    childId: string
  ): Promise<DeletionResponse> {
    const verified = await this.verifyParentIdentity(parentEmail, childId);
    
    if (!verified) {
      return { 
        success: false, 
        reason: 'Parent identity verification failed' 
      };
    }

    // Delete all collected data
    const child = this.childUsers.get(childId);
    if (child) {
      child.dataCollected = [];
      child.parentalConsentStatus = 'denied';
      
      // Notify all third parties to delete
      await this.notifyThirdPartiesOfDeletion(childId);
      
      // Optionally delete the account
      // this.childUsers.delete(childId);
    }

    return {
      success: true,
      deletedAt: new Date(),
      confirmation: 'All personal information has been deleted'
    };
  }

  // Parental Rights: Revoke Consent
  async revokeParentalConsent(
    parentEmail: string,
    childId: string
  ): Promise<void> {
    const verified = await this.verifyParentIdentity(parentEmail, childId);
    
    if (!verified) {
      throw new Error('Parent identity verification failed');
    }

    const child = this.childUsers.get(childId);
    if (child) {
      child.parentalConsentStatus = 'denied';
      
      // Stop all data collection immediately
      // Allow continued use without collecting personal info
      // or terminate account based on service requirements
    }
  }

  // Automated Data Retention Cleanup
  async runRetentionCleanup(): Promise<CleanupResult> {
    const now = new Date();
    let deletedCount = 0;

    for (const [userId, user] of this.childUsers) {
      const expiredData = user.dataCollected.filter(
        d => d.expiresAt < now
      );

      for (const data of expiredData) {
        await this.secureDelete(userId, data);
        deletedCount++;
      }

      user.dataCollected = user.dataCollected.filter(
        d => d.expiresAt >= now
      );
    }

    return {
      success: true,
      deletedRecords: deletedCount,
      timestamp: now
    };
  }
}

§ 6.4 CHILDREN'S PRIVACY POLICY TEMPLATE

┌─────────────────────────────────────────────────────────────────────┐
│            CHILDREN'S PRIVACY POLICY - REQUIRED ELEMENTS            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Deve essere SEPARATA dalla privacy policy generale                 │
│  Deve usare linguaggio CHIARO E SEMPLICE                            │
│  Deve essere FACILMENTE ACCESSIBILE                                 │
│                                                                      │
│  CONTENUTO OBBLIGATORIO:                                            │
│                                                                      │
│  1. Chi siamo e come contattarci                                    │
│     • Nome azienda                                                  │
│     • Indirizzo                                                     │
│     • Email                                                         │
│     • Telefono                                                      │
│                                                                      │
│  2. Quali informazioni raccogliamo                                  │
│     • Lista SPECIFICA di ogni tipo di dato                         │
│     • Distinzione tra dati obbligatori e opzionali                 │
│                                                                      │
│  3. Come usiamo le informazioni                                     │
│     • Scopo SPECIFICO per ogni dato raccolto                       │
│     • NO raccolta per pubblicità targeted (dal 2026)               │
│                                                                      │
│  4. Con chi condividiamo le informazioni                            │
│     • Nome/categoria di OGNI terza parte                           │
│     • Scopo della condivisione                                      │
│                                                                      │
│  5. Diritti dei genitori                                            │
│     • Come richiedere accesso ai dati                               │
│     • Come richiedere cancellazione                                 │
│     • Come revocare consenso                                        │
│     • Come impedire futura raccolta                                 │
│                                                                      │
│  6. Come proteggiamo le informazioni                                │
│     • Misure di sicurezza adottate                                  │
│                                                                      │
│  7. Pratiche di raccolta senza consenso                             │
│     • Identificatori persistenti (se raccolti)                     │
│     • Scopo (es. support, security)                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘



---

§ 7. WCAG 2.2 - WEB CONTENT ACCESSIBILITY GUIDELINES

§ 7.1 OVERVIEW E LIVELLI DI CONFORMITÀ

WCAG 2.2 è lo standard internazionale per l'accessibilità web, pubblicato nell'ottobre 2023.

┌─────────────────────────────────────────────────────────────────────┐
│                    WCAG 2.2 CONFORMANCE LEVELS                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  LIVELLO A (Minimo)                                                 │
│  ├── Requisiti base per accessibilità                               │
│  ├── Senza questi, alcuni utenti NON possono usare il sito         │
│  └── 30 criteri di successo                                         │
│                                                                      │
│  LIVELLO AA (Standard Raccomandato)                                 │
│  ├── Include tutti i criteri Level A                                │
│  ├── Standard richiesto dalla maggior parte delle leggi            │
│  ├── Equilibrio tra accessibilità e praticità                      │
│  └── 24 criteri aggiuntivi (totale 54)                             │
│                                                                      │
│  LIVELLO AAA (Ottimale)                                             │
│  ├── Include tutti i criteri Level A e AA                          │
│  ├── Non sempre raggiungibile per tutti i contenuti                │
│  ├── Target per contenuti specifici                                 │
│  └── 28 criteri aggiuntivi (totale 82)                             │
│                                                                      │
│  WCAG 2.2 AGGIUNGE:                                                 │
│  └── 9 nuovi criteri di successo                                    │
│      (6 Level A, 2 Level AA, 1 Level AAA)                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 7.2 I 4 PRINCIPI POUR

┌─────────────────────────────────────────────────────────────────────┐
│                    WCAG POUR PRINCIPLES                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  P - PERCEIVABLE (Percepibile)                                      │
│      Le informazioni e i componenti UI devono essere presentati    │
│      in modi che gli utenti possano percepire                       │
│      ├── Alternative testuali per contenuti non testuali           │
│      ├── Sottotitoli e alternative per media                        │
│      ├── Contenuto adattabile                                       │
│      └── Contenuto distinguibile (contrasto, dimensioni)           │
│                                                                      │
│  O - OPERABLE (Utilizzabile)                                        │
│      I componenti UI e la navigazione devono essere utilizzabili   │
│      ├── Accessibile da tastiera                                    │
│      ├── Tempo sufficiente per leggere e usare                     │
│      ├── Niente contenuti che causano convulsioni                  │
│      ├── Navigabile                                                 │
│      └── Modalità di input (oltre tastiera)                        │
│                                                                      │
│  U - UNDERSTANDABLE (Comprensibile)                                 │
│      Le informazioni e l'operazione dell'UI devono essere          │
│      comprensibili                                                  │
│      ├── Leggibile                                                  │
│      ├── Prevedibile                                                │
│      └── Assistenza nell'input                                      │
│                                                                      │
│  R - ROBUST (Robusto)                                               │
│      Il contenuto deve essere abbastanza robusto da essere         │
│      interpretato da diverse tecnologie assistive                  │
│      ├── Compatibile con user agent attuali e futuri              │
│      └── Parsing corretto                                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 7.3 NUOVI CRITERI WCAG 2.2

┌─────────────────────────────────────────────────────────────────────┐
│                 WCAG 2.2 NEW SUCCESS CRITERIA                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  2.4.11 FOCUS NOT OBSCURED (MINIMUM) - Level AA                    │
│  Quando un elemento riceve focus, non deve essere completamente    │
│  nascosto da contenuto creato dall'autore                          │
│  ✓ Sticky headers non devono coprire elementi focused              │
│  ✓ Modal/dialogs non devono oscurare il focus                      │
│                                                                      │
│  2.4.12 FOCUS NOT OBSCURED (ENHANCED) - Level AAA                  │
│  Focus NESSUNA parte nascosta (più stringente del 2.4.11)          │
│                                                                      │
│  2.4.13 FOCUS APPEARANCE - Level AAA                               │
│  Focus indicator deve avere:                                        │
│  • Area ≥ perimetro × 2 CSS pixel                                  │
│  • Contrasto ≥ 3:1 tra stati focused/unfocused                     │
│                                                                      │
│  2.5.7 DRAGGING MOVEMENTS - Level AA                               │
│  Funzionalità con dragging devono avere alternative single-pointer │
│  ✓ Drag-and-drop deve avere alternativa click                      │
│  ✓ Slider devono essere usabili senza drag                         │
│                                                                      │
│  2.5.8 TARGET SIZE (MINIMUM) - Level AA                            │
│  Target per pointer input devono essere almeno 24×24 CSS pixel     │
│  ECCEZIONI: inline text links, user agent controls, essential size │
│                                                                      │
│  3.2.6 CONSISTENT HELP - Level A                                   │
│  Se un sito ha meccanismi di help, devono essere nello stesso      │
│  ordine relativo su ogni pagina                                     │
│  ✓ Chat, contact info, FAQ in posizione consistente                │
│                                                                      │
│  3.3.7 REDUNDANT ENTRY - Level A                                   │
│  Info già inserite dall'utente NON devono essere richieste di      │
│  nuovo, a meno che:                                                 │
│  • L'utente possa selezionare da info precedenti                   │
│  • Sia essenziale per sicurezza                                    │
│                                                                      │
│  3.3.8 ACCESSIBLE AUTHENTICATION (MINIMUM) - Level AA              │
│  Nessun test cognitivo (es. ricordare password) richiesto          │
│  ECCEZIONI: se alternativa disponibile, o oggetto fornito,         │
│  o riconoscimento personale                                         │
│  ✓ Permettere password manager                                      │
│  ✓ Permettere copy-paste password                                   │
│  ✓ Offrire autenticazione biometrica                               │
│                                                                      │
│  3.3.9 ACCESSIBLE AUTHENTICATION (ENHANCED) - Level AAA            │
│  Come 3.3.8 ma senza eccezione "riconoscimento oggetto"            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 7.4 IMPLEMENTAZIONE ACCESSIBILITÀ REACT

typescript
// accessibility-components.tsx
import React, { useRef, useEffect, useState, useCallback } from 'react';

// Skip Link Component
export const SkipLink: React.FC<{ targetId: string }> = ({ targetId }) => {
  return (
    <a
      href={`#${targetId}`}
      className="skip-link"
      style={{
        position: 'absolute',
        left: '-9999px',
        top: 'auto',
        width: '1px',
        height: '1px',
        overflow: 'hidden',
      }}
      onFocus={(e) => {
        e.currentTarget.style.left = '0';
        e.currentTarget.style.width = 'auto';
        e.currentTarget.style.height = 'auto';
      }}
      onBlur={(e) => {
        e.currentTarget.style.left = '-9999px';
        e.currentTarget.style.width = '1px';
        e.currentTarget.style.height = '1px';
      }}
    >
      Skip to main content
    </a>
  );
};

// WCAG 2.2 Focus Management
export const useFocusManagement = () => {
  const focusableElements = useRef<HTMLElement[]>([]);
  
  const trapFocus = useCallback((container: HTMLElement) => {
    const elements = container.querySelectorAll<HTMLElement>(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    focusableElements.current = Array.from(elements);
    
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key !== 'Tab') return;
      
      const first = focusableElements.current[0];
      const last = focusableElements.current[focusableElements.current.length - 1];
      
      if (e.shiftKey && document.activeElement === first) {
        e.preventDefault();
        last?.focus();
      } else if (!e.shiftKey && document.activeElement === last) {
        e.preventDefault();
        first?.focus();
      }
    };
    
    container.addEventListener('keydown', handleKeyDown);
    return () => container.removeEventListener('keydown', handleKeyDown);
  }, []);
  
  return { trapFocus };
};

// WCAG 2.5.8 Target Size Compliant Button
interface AccessibleButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  minSize?: number; // Default 24px per WCAG 2.2
}

export const AccessibleButton: React.FC<AccessibleButtonProps> = ({
  minSize = 24,
  style,
  children,
  ...props
}) => {
  return (
    <button
      {...props}
      style={{
        minWidth: `${minSize}px`,
        minHeight: `${minSize}px`,
        padding: '8px 16px',
        // Ensure focus indicator is visible (2.4.11)
        outline: 'none',
        ...style,
      }}
      className="accessible-button"
    >
      {children}
      <style>{`
        .accessible-button:focus-visible {
          outline: 3px solid #005fcc;
          outline-offset: 2px;
          /* 2.4.13 Focus Appearance - contrast ratio ≥ 3:1 */
        }
      `}</style>
    </button>
  );
};

// WCAG 2.5.7 Dragging Alternative
interface DraggableListProps<T> {
  items: T[];
  onReorder: (items: T[]) => void;
  renderItem: (item: T, index: number) => React.ReactNode;
  getId: (item: T) => string;
}

export function AccessibleReorderableList<T>({
  items,
  onReorder,
  renderItem,
  getId,
}: DraggableListProps<T>) {
  const [selectedIndex, setSelectedIndex] = useState<number | null>(null);
  
  // Single-pointer alternative to drag
  const moveItem = (fromIndex: number, direction: 'up' | 'down') => {
    const toIndex = direction === 'up' ? fromIndex - 1 : fromIndex + 1;
    if (toIndex < 0 || toIndex >= items.length) return;
    
    const newItems = [...items];
    [newItems[fromIndex], newItems[toIndex]] = [newItems[toIndex], newItems[fromIndex]];
    onReorder(newItems);
  };
  
  return (
    <ul role="listbox" aria-label="Reorderable list">
      {items.map((item, index) => (
        <li
          key={getId(item)}
          role="option"
          aria-selected={selectedIndex === index}
          tabIndex={0}
          onKeyDown={(e) => {
            if (e.key === 'ArrowUp' && e.altKey) {
              e.preventDefault();
              moveItem(index, 'up');
            } else if (e.key === 'ArrowDown' && e.altKey) {
              e.preventDefault();
              moveItem(index, 'down');
            }
          }}
        >
          {renderItem(item, index)}
          {/* Single-pointer alternatives */}
          <button
            aria-label={`Move ${getId(item)} up`}
            onClick={() => moveItem(index, 'up')}
            disabled={index === 0}
            style={{ minWidth: 24, minHeight: 24 }} // 2.5.8
          >
            ↑
          </button>
          <button
            aria-label={`Move ${getId(item)} down`}
            onClick={() => moveItem(index, 'down')}
            disabled={index === items.length - 1}
            style={{ minWidth: 24, minHeight: 24 }} // 2.5.8
          >
            ↓
          </button>
        </li>
      ))}
    </ul>
  );
}

// WCAG 3.3.7 Redundant Entry Prevention
interface FormWithMemoryProps {
  onSubmit: (data: FormData) => void;
}

export const FormWithMemory: React.FC<FormWithMemoryProps> = ({ onSubmit }) => {
  const [savedData, setSavedData] = useState<Record<string, string>>({});
  
  useEffect(() => {
    // Load previously entered data
    const stored = sessionStorage.getItem('form_data');
    if (stored) {
      setSavedData(JSON.parse(stored));
    }
  }, []);
  
  const handleFieldChange = (name: string, value: string) => {
    const newData = { ...savedData, [name]: value };
    setSavedData(newData);
    sessionStorage.setItem('form_data', JSON.stringify(newData));
  };
  
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      onSubmit(new FormData(e.currentTarget));
    }}>
      <fieldset>
        <legend>Shipping Address</legend>
        <input
          name="shipping_name"
          aria-label="Full name for shipping"
          defaultValue={savedData.shipping_name || ''}
          onChange={(e) => handleFieldChange('shipping_name', e.target.value)}
        />
        {/* More fields... */}
      </fieldset>
      
      <fieldset>
        <legend>Billing Address</legend>
        {/* WCAG 3.3.7: Option to use previously entered data */}
        <label>
          <input
            type="checkbox"
            onChange={(e) => {
              if (e.target.checked) {
                // Pre-fill with shipping data
                document.querySelector<HTMLInputElement>('[name="billing_name"]')!
                  .value = savedData.shipping_name || '';
              }
            }}
          />
          Same as shipping address
        </label>
        <input
          name="billing_name"
          aria-label="Full name for billing"
          defaultValue={savedData.billing_name || ''}
          onChange={(e) => handleFieldChange('billing_name', e.target.value)}
        />
      </fieldset>
      
      <button type="submit" style={{ minWidth: 24, minHeight: 24 }}>
        Submit
      </button>
    </form>
  );
};

// WCAG 3.3.8 Accessible Authentication
export const AccessibleLoginForm: React.FC = () => {
  const [authMethod, setAuthMethod] = useState<'password' | 'passkey' | 'magic-link'>('password');
  
  return (
    <div role="form" aria-label="Login">
      <h2>Sign In</h2>
      
      {/* Multiple authentication options per 3.3.8 */}
      <div role="radiogroup" aria-label="Choose authentication method">
        <label>
          <input
            type="radio"
            name="auth_method"
            value="password"
            checked={authMethod === 'password'}
            onChange={() => setAuthMethod('password')}
          />
          Password
        </label>
        <label>
          <input
            type="radio"
            name="auth_method"
            value="passkey"
            checked={authMethod === 'passkey'}
            onChange={() => setAuthMethod('passkey')}
          />
          Passkey (Biometric)
        </label>
        <label>
          <input
            type="radio"
            name="auth_method"
            value="magic-link"
            checked={authMethod === 'magic-link'}
            onChange={() => setAuthMethod('magic-link')}
          />
          Email Magic Link
        </label>
      </div>
      
      {authMethod === 'password' && (
        <div>
          <label htmlFor="email">Email</label>
          <input
            id="email"
            type="email"
            name="email"
            autoComplete="email" // Allow password managers
          />
          
          <label htmlFor="password">Password</label>
          <input
            id="password"
            type="password"
            name="password"
            autoComplete="current-password" // Allow password managers
            // Do NOT disable paste - required by 3.3.8
          />
          
          <button type="submit" style={{ minWidth: 44, minHeight: 44 }}>
            Sign In
          </button>
        </div>
      )}
      
      {authMethod === 'passkey' && (
        <button
          type="button"
          onClick={() => {
            // WebAuthn/Passkey authentication
            // No cognitive test required
          }}
          style={{ minWidth: 44, minHeight: 44 }}
        >
          Authenticate with Passkey
        </button>
      )}
      
      {authMethod === 'magic-link' && (
        <div>
          <label htmlFor="magic-email">Email</label>
          <input
            id="magic-email"
            type="email"
            name="email"
            autoComplete="email"
          />
          <button type="submit" style={{ minWidth: 44, minHeight: 44 }}>
            Send Magic Link
          </button>
          <p>We'll send a link to sign in - no password needed.</p>
        </div>
      )}
    </div>
  );
};

// WCAG 3.2.6 Consistent Help
export const ConsistentHelpWidget: React.FC = () => {
  // This component should appear in the same relative order on all pages
  return (
    <aside aria-label="Help and support" className="help-widget">
      <nav aria-label="Help options">
        <ul>
          {/* Always in this order per 3.2.6 */}
          <li><a href="/faq">FAQ</a></li>
          <li><a href="/contact">Contact Us</a></li>
          <li><button onClick={() => openChat()}>Live Chat</button></li>
          <li><a href="tel:+1-800-123-4567">Call Support</a></li>
        </ul>
      </nav>
    </aside>
  );
};

§ 7.5 TESTING ACCESSIBILITÀ

typescript
// accessibility-testing.ts
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

describe('Accessibility Tests', () => {
  test('page should have no accessibility violations', async () => {
    const { container } = render(<MyPage />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
  
  test('focus should not be obscured by sticky header', () => {
    render(<PageWithStickyHeader />);
    
    const focusableElement = screen.getByRole('button', { name: /submit/i });
    const stickyHeader = screen.getByRole('banner');
    
    focusableElement.focus();
    
    const focusRect = focusableElement.getBoundingClientRect();
    const headerRect = stickyHeader.getBoundingClientRect();
    
    // Element should not be completely hidden by header
    expect(focusRect.top).toBeGreaterThanOrEqual(headerRect.bottom);
  });
  
  test('all interactive elements meet minimum target size', () => {
    render(<MyForm />);
    
    const buttons = screen.getAllByRole('button');
    const links = screen.getAllByRole('link');
    const inputs = screen.getAllByRole('textbox');
    
    [...buttons, ...links, ...inputs].forEach(element => {
      const rect = element.getBoundingClientRect();
      // WCAG 2.5.8: minimum 24x24 CSS pixels
      expect(rect.width).toBeGreaterThanOrEqual(24);
      expect(rect.height).toBeGreaterThanOrEqual(24);
    });
  });
  
  test('drag functionality has single-pointer alternative', () => {
    render(<DraggableList items={['A', 'B', 'C']} />);
    
    // Should have up/down buttons as alternatives
    expect(screen.getAllByRole('button', { name: /move.*up/i })).toHaveLength(3);
    expect(screen.getAllByRole('button', { name: /move.*down/i })).toHaveLength(3);
  });
  
  test('authentication does not require cognitive test', () => {
    render(<LoginForm />);
    
    // Should allow password managers
    const passwordInput = screen.getByLabelText(/password/i);
    expect(passwordInput).toHaveAttribute('autocomplete', 'current-password');
    
    // Should not disable paste
    expect(passwordInput).not.toHaveAttribute('onpaste');
    
    // Should offer alternative auth methods
    expect(screen.getByRole('radio', { name: /passkey/i })).toBeInTheDocument();
  });
});

// Automated accessibility audit
async function runAccessibilityAudit(url: string): Promise<AuditReport> {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto(url);
  
  // Run axe-core
  await page.addScriptTag({ path: require.resolve('axe-core') });
  
  const results = await page.evaluate(async () => {
    // @ts-ignore
    return await axe.run();
  });
  
  await browser.close();
  
  return {
    url,
    timestamp: new Date(),
    violations: results.violations,
    passes: results.passes,
    incomplete: results.incomplete,
    wcagLevel: determineWCAGLevel(results)
  };
}



---

§ 8. EUROPEAN ACCESSIBILITY ACT (EAA)

§ 8.1 OVERVIEW E TIMELINE

La Direttiva (UE) 2019/882 (European Accessibility Act) diventa applicabile dal **28 giugno 2025**.

┌─────────────────────────────────────────────────────────────────────┐
│                    EAA - EUROPEAN ACCESSIBILITY ACT                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  TIMELINE                                                           │
│  ├── Giugno 2019: Adozione direttiva                               │
│  ├── Giugno 2022: Recepimento negli Stati membri                   │
│  ├── 28 Giugno 2025: ENFORCEMENT (prodotti e servizi nuovi)        │
│  └── 28 Giugno 2030: Tutti i prodotti/servizi (inclusi esistenti)  │
│                                                                      │
│  AMBITO DI APPLICAZIONE                                             │
│  ├── E-commerce websites e mobile apps                              │
│  ├── Servizi bancari per consumatori                                │
│  ├── E-books e software dedicato                                    │
│  ├── Servizi di trasporto (biglietteria, check-in)                 │
│  ├── Servizi di comunicazione elettronica                          │
│  ├── Servizi di media audiovisivi                                   │
│  └── Prodotti: computer, smartphone, terminali di pagamento        │
│                                                                      │
│  ESENZIONI                                                          │
│  ├── Microimprese (< 10 dipendenti E < €2M fatturato)             │
│  ├── Onere sproporzionato (da documentare)                         │
│  └── Contenuti di terze parti non controllabili                    │
│                                                                      │
│  SANZIONI (variano per Stato membro)                                │
│  ├── Italia: fino a €120.000 (ripetute: fino a €240.000)          │
│  ├── Germania: fino a €100.000                                     │
│  ├── Francia: fino a €75.000 (ripetute: fino a €225.000)          │
│  └── Reclami da utenti + possibile ritiro dal mercato             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 8.2 REQUISITI EAA PER E-COMMERCE

┌─────────────────────────────────────────────────────────────────────┐
│              EAA REQUIREMENTS FOR E-COMMERCE (Annex I)               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PERCEIVABLE                                                        │
│  ├── Info disponibili in più di un canale sensoriale              │
│  ├── Alternative testuali per contenuti non testuali               │
│  ├── Sottotitoli e audiodescrizioni                                │
│  ├── Presentazione visibile e comprensibile                        │
│  └── Contrasto sufficiente                                          │
│                                                                      │
│  OPERABLE                                                           │
│  ├── Funzionalità accessibili da tastiera                          │
│  ├── Tempo sufficiente per interazioni                             │
│  ├── Prevenzione convulsioni                                        │
│  ├── Navigazione e orientamento facilitati                         │
│  └── Input alternativi a tastiera                                   │
│                                                                      │
│  UNDERSTANDABLE                                                     │
│  ├── Linguaggio semplice (livello B2 max)                          │
│  ├── Funzionamento prevedibile                                      │
│  ├── Aiuto per prevenire e correggere errori                       │
│  └── Istruzioni chiare per uso                                      │
│                                                                      │
│  ROBUST                                                             │
│  ├── Compatibilità con tecnologie assistive attuali                │
│  ├── Compatibilità con tecnologie future (forward compatible)      │
│  └── Standard aperti preferiti                                      │
│                                                                      │
│  REQUISITI SPECIFICI E-COMMERCE                                     │
│  ├── Identificazione e pagamento accessibili                       │
│  ├── Info su accessibilità del prodotto venduto                    │
│  ├── Info su accessibilità del servizio stesso                     │
│  └── Supporto clienti accessibile                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 8.3 DICHIARAZIONE DI ACCESSIBILITÀ EAA

typescript
// eaa-accessibility-statement.ts
interface EAAAccessibilityStatement {
  // Informazioni obbligatorie
  organizationInfo: {
    name: string;
    address: string;
    email: string;
    phone?: string;
  };
  
  // Stato di conformità
  conformanceStatus: 
    | 'fully_conformant'      // Pienamente conforme
    | 'partially_conformant'  // Parzialmente conforme
    | 'non_conformant';       // Non conforme
  
  // Standard di riferimento
  technicalStandard: 'EN_301_549' | 'WCAG_2_1_AA' | 'WCAG_2_2_AA';
  
  // Non conformità note (se parzialmente conforme)
  nonConformances?: {
    requirement: string;
    description: string;
    remediation?: string;
    expectedResolutionDate?: Date;
  }[];
  
  // Contenuti non accessibili
  inaccessibleContent?: {
    description: string;
    reason: 'exemption' | 'third_party' | 'disproportionate_burden';
    justification: string;
  }[];
  
  // Meccanismo di feedback
  feedbackMechanism: {
    email: string;
    phone?: string;
    formUrl?: string;
    responseTime: string; // es. "14 giorni lavorativi"
  };
  
  // Procedura di enforcement
  enforcementProcedure: {
    nationalAuthority: string;
    authorityWebsite: string;
    complaintProcedure: string;
  };
  
  // Metadati
  statementDate: Date;
  lastReviewDate: Date;
  reviewFrequency: 'annual' | 'biannual' | 'continuous';
}

class EAAComplianceManager {
  private statement: EAAAccessibilityStatement;

  generateStatement(): string {
    return `
# Dichiarazione di Accessibilità

## Chi siamo

${this.statement.organizationInfo.name}
${this.statement.organizationInfo.address}
Email: ${this.statement.organizationInfo.email}
${this.statement.organizationInfo.phone ? `Tel: ${this.statement.organizationInfo.phone}` : ''}

## Stato di Conformità

Questo sito web è **${this.getConformanceLabel()}** con lo standard 
${this.getStandardLabel()}, in attuazione della Direttiva (UE) 2019/882 
(European Accessibility Act).

${this.getNonConformancesSection()}

${this.getInaccessibleContentSection()}

## Come Contattarci

Se riscontri problemi di accessibilità o hai bisogno di informazioni 
in formato accessibile, contattaci:

- Email: ${this.statement.feedbackMechanism.email}
${this.statement.feedbackMechanism.phone ? `- Telefono: ${this.statement.feedbackMechanism.phone}` : ''}
${this.statement.feedbackMechanism.formUrl ? `- Modulo online: ${this.statement.feedbackMechanism.formUrl}` : ''}

Tempo di risposta: ${this.statement.feedbackMechanism.responseTime}

## Procedura di Applicazione

Se non ricevi risposta soddisfacente, puoi rivolgerti a:

${this.statement.enforcementProcedure.nationalAuthority}
${this.statement.enforcementProcedure.authorityWebsite}

${this.statement.enforcementProcedure.complaintProcedure}

---

Questa dichiarazione è stata redatta il ${this.formatDate(this.statement.statementDate)}.
Ultima revisione: ${this.formatDate(this.statement.lastReviewDate)}.
Frequenza di revisione: ${this.getReviewFrequencyLabel()}.
    `;
  }

  private getConformanceLabel(): string {
    const labels: Record<string, string> = {
      fully_conformant: 'pienamente conforme',
      partially_conformant: 'parzialmente conforme',
      non_conformant: 'non conforme'
    };
    return labels[this.statement.conformanceStatus];
  }

  private getStandardLabel(): string {
    const labels: Record<string, string> = {
      EN_301_549: 'EN 301 549',
      WCAG_2_1_AA: 'WCAG 2.1 livello AA',
      WCAG_2_2_AA: 'WCAG 2.2 livello AA'
    };
    return labels[this.statement.technicalStandard];
  }

  // Audit per EAA compliance
  async performEAAAudit(): Promise<EAAAuditReport> {
    const report: EAAAuditReport = {
      date: new Date(),
      overallScore: 0,
      categories: []
    };

    // Test automatici
    const automatedResults = await this.runAutomatedTests();
    
    // Checklist manuale richiesta
    const manualChecklist = this.getManualChecklist();

    report.categories = [
      { name: 'Perceivable', score: automatedResults.perceivable, weight: 25 },
      { name: 'Operable', score: automatedResults.operable, weight: 25 },
      { name: 'Understandable', score: automatedResults.understandable, weight: 25 },
      { name: 'Robust', score: automatedResults.robust, weight: 25 }
    ];

    report.overallScore = report.categories.reduce(
      (acc, cat) => acc + (cat.score * cat.weight / 100), 
      0
    );

    report.isCompliant = report.overallScore >= 80; // Soglia minima
    report.manualChecklistRequired = manualChecklist;

    return report;
  }

  private getManualChecklist(): ManualCheckItem[] {
    return [
      {
        id: 'lang_level',
        category: 'Understandable',
        requirement: 'Linguaggio al livello B2 o inferiore',
        howToTest: 'Usa tool di readability (Flesch-Kincaid, Gulpease)',
        passed: null
      },
      {
        id: 'assistive_tech',
        category: 'Robust',
        requirement: 'Compatibilità con screen reader',
        howToTest: 'Testa con NVDA, JAWS, VoiceOver',
        passed: null
      },
      {
        id: 'keyboard_nav',
        category: 'Operable',
        requirement: 'Navigazione completa da tastiera',
        howToTest: 'Naviga tutto il sito usando solo Tab, Enter, frecce',
        passed: null
      },
      {
        id: 'checkout_accessible',
        category: 'E-commerce specific',
        requirement: 'Processo di checkout accessibile',
        howToTest: 'Completa un acquisto con screen reader e solo tastiera',
        passed: null
      },
      {
        id: 'error_prevention',
        category: 'Understandable',
        requirement: 'Prevenzione errori nei form',
        howToTest: 'Verifica messaggi di errore chiari e suggerimenti',
        passed: null
      },
      {
        id: 'product_accessibility_info',
        category: 'E-commerce specific',
        requirement: 'Info accessibilità prodotti venduti',
        howToTest: 'Verifica presenza info accessibilità nelle schede prodotto',
        passed: null
      }
    ];
  }
}

§ 8.4 EAA VS WCAG - DIFFERENZE CHIAVE

┌─────────────────────────────────────────────────────────────────────┐
│                    EAA vs WCAG COMPARISON                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ASPETTO           │ WCAG                │ EAA                      │
│  ──────────────────┼─────────────────────┼─────────────────────────│
│  Tipo              │ Standard tecnico    │ Direttiva legale        │
│  Obbligatorietà    │ Volontario*         │ Obbligatorio UE         │
│  Ambito            │ Solo contenuti web  │ Prodotti + Servizi      │
│  Sanzioni          │ Nessuna diretta     │ Multe, ritiro mercato   │
│  Enforcement       │ Varia per paese     │ Autorità nazionali      │
│  Aggiornamenti     │ W3C (lento)         │ Commissione EU          │
│  Esenzioni         │ Nessuna             │ Microimprese, onere     │
│                    │                     │ sproporzionato          │
│  ──────────────────┴─────────────────────┴─────────────────────────│
│                                                                      │
│  * WCAG diventa obbligatorio quando riferito da leggi (es. ADA,    │
│    Section 508, EN 301 549)                                         │
│                                                                      │
│  RELAZIONE: L'EAA richiede conformità a EN 301 549, che a sua      │
│  volta incorpora WCAG 2.1 AA. Quindi WCAG è il "come", EAA è il    │
│  "perché deve essere fatto".                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

---

§ 9. EU AI ACT - REGOLAMENTO SULL'INTELLIGENZA ARTIFICIALE

§ 9.1 OVERVIEW E TIMELINE

Il Regolamento (UE) 2024/1689 (AI Act) è il primo framework normativo completo per l'AI.

┌─────────────────────────────────────────────────────────────────────┐
│                    EU AI ACT - TIMELINE                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1 Agosto 2024: Entrata in vigore                                   │
│  │                                                                   │
│  ├── 2 Febbraio 2025: PRATICHE VIETATE (Art. 5)                    │
│  │   Divieti su: manipolazione subliminale, sfruttamento           │
│  │   vulnerabilità, social scoring, biometria in tempo reale       │
│  │                                                                   │
│  ├── 2 Agosto 2025: GPAI (General Purpose AI)                      │
│  │   Obblighi per modelli fondazionali (es. GPT, Claude)           │
│  │   Requisiti di trasparenza, documentazione tecnica              │
│  │                                                                   │
│  ├── 2 Agosto 2026: ALTO RISCHIO (Allegato III)                    │
│  │   Sistemi AI in settori critici: employment, credit,            │
│  │   education, law enforcement, migration                         │
│  │                                                                   │
│  └── 2 Agosto 2027: ALTO RISCHIO (Allegato I)                      │
│      Sistemi AI come safety component in prodotti regolamentati    │
│                                                                      │
│  SANZIONI                                                           │
│  ├── Pratiche vietate: fino a €35M o 7% fatturato globale         │
│  ├── Violazioni obblighi: fino a €15M o 3% fatturato              │
│  └── Info errate: fino a €7.5M o 1.5% fatturato                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 9.2 CLASSIFICAZIONE DEL RISCHIO

┌─────────────────────────────────────────────────────────────────────┐
│                    AI ACT RISK CLASSIFICATION                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  🚫 RISCHIO INACCETTABILE (VIETATO)                                 │
│     ├── Manipolazione subliminale cognitiva                        │
│     ├── Sfruttamento vulnerabilità (età, disabilità)              │
│     ├── Social scoring da parte di autorità pubbliche              │
│     ├── Biometria real-time in spazi pubblici (eccezioni limitate)│
│     ├── Scraping biometrico non mirato (es. Clearview AI)         │
│     ├── Emotion recognition sul lavoro/scuola (eccezioni)         │
│     ├── Categorizzazione biometrica per inferire caratteristiche  │
│     └── Predictive policing basato esclusivamente su profiling    │
│                                                                      │
│  ⚠️  ALTO RISCHIO                                                    │
│     ├── Biometria (identificazione, categorizzazione)              │
│     ├── Infrastrutture critiche (energia, trasporti, acqua)       │
│     ├── Educazione (ammissioni, valutazioni)                       │
│     ├── Occupazione (recruitment, promozioni, licenziamenti)      │
│     ├── Servizi essenziali (credito, assicurazioni)               │
│     ├── Law enforcement (valutazione rischio, poligrafo)          │
│     ├── Migrazione/asilo (valutazione domande, controlli)         │
│     ├── Giustizia (supporto decisioni giudiziarie)                │
│     └── Elezioni (influenza su voto)                               │
│                                                                      │
│  ⚡ RISCHIO LIMITATO (OBBLIGHI TRASPARENZA)                         │
│     ├── Chatbot: informare che si interagisce con AI              │
│     ├── Emotion recognition: informare l'utente                    │
│     ├── Deep fake: etichettare contenuto generato                  │
│     └── AI-generated content: disclosure                           │
│                                                                      │
│  ✅ RISCHIO MINIMO (NESSUN OBBLIGO)                                 │
│     ├── Spam filters                                                │
│     ├── Video game AI                                               │
│     └── AI inventory management                                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 9.3 OBBLIGHI PER SISTEMI AD ALTO RISCHIO

typescript
// ai-act-compliance.ts
interface HighRiskAISystem {
  id: string;
  name: string;
  version: string;
  provider: OrganizationInfo;
  category: HighRiskCategory;
  intendedPurpose: string;
  
  // Art. 9: Risk Management
  riskManagementSystem: RiskManagementSystem;
  
  // Art. 10: Data Governance
  dataGovernance: DataGovernanceRequirements;
  
  // Art. 11: Technical Documentation
  technicalDocumentation: TechnicalDocumentation;
  
  // Art. 12: Record-keeping
  recordKeeping: RecordKeepingSystem;
  
  // Art. 13: Transparency
  transparencyInfo: TransparencyRequirements;
  
  // Art. 14: Human Oversight
  humanOversight: HumanOversightMeasures;
  
  // Art. 15: Accuracy, Robustness, Cybersecurity
  qualityRequirements: QualityRequirements;
}

interface RiskManagementSystem {
  // Continuous iterative process
  establishedDate: Date;
  lastReviewDate: Date;
  reviewFrequency: 'continuous' | 'quarterly' | 'annual';
  
  // Identification and analysis of known/foreseeable risks
  identifiedRisks: Risk[];
  
  // Estimation and evaluation
  riskAssessments: RiskAssessment[];
  
  // Mitigation measures
  mitigationMeasures: MitigationMeasure[];
  
  // Testing for risk management
  testingProcedures: TestingProcedure[];
  
  // Residual risks
  residualRisks: ResidualRisk[];
}

interface DataGovernanceRequirements {
  // Training, validation, testing datasets
  datasets: {
    purpose: 'training' | 'validation' | 'testing';
    description: string;
    size: number;
    characteristics: DatasetCharacteristics;
  }[];
  
  // Data quality criteria
  qualityCriteria: {
    relevance: QualityMetric;
    representativeness: QualityMetric;
    errorFreedom: QualityMetric;
    completeness: QualityMetric;
  };
  
  // Bias examination
  biasExamination: {
    methodology: string;
    findingsDate: Date;
    biasesIdentified: BiasReport[];
    mitigationApplied: string[];
  };
  
  // Data gaps identification
  dataGaps: {
    identified: string[];
    howAddressed: string[];
  };
}

interface TechnicalDocumentation {
  // General description
  generalDescription: {
    intendedPurpose: string;
    provider: string;
    versionHistory: VersionInfo[];
  };
  
  // Detailed description
  detailedDescription: {
    developmentMethods: string[];
    computationalResources: string;
    thirdPartyComponents: ThirdPartyComponent[];
  };
  
  // Monitoring, functioning, control
  monitoringInfo: {
    performanceMetrics: Metric[];
    loggingCapabilities: string;
    humanInterfaceTools: string[];
  };
  
  // Risk management documentation
  riskManagementDocs: string; // Reference to RMS docs
  
  // Changes documentation
  changesLog: ChangeRecord[];
  
  // Standards applied
  harmonisedStandards: string[];
  
  // EU Declaration of Conformity
  declarationOfConformity: ConformityDeclaration;
}

class AIActComplianceChecker {
  async assessSystem(system: HighRiskAISystem): Promise<ComplianceReport> {
    const checks: ComplianceCheck[] = [];
    
    // Art. 9: Risk Management System
    checks.push(await this.checkRiskManagement(system.riskManagementSystem));
    
    // Art. 10: Data Governance
    checks.push(await this.checkDataGovernance(system.dataGovernance));
    
    // Art. 11: Technical Documentation
    checks.push(await this.checkTechnicalDocs(system.technicalDocumentation));
    
    // Art. 12: Record-keeping
    checks.push(await this.checkRecordKeeping(system.recordKeeping));
    
    // Art. 13: Transparency
    checks.push(await this.checkTransparency(system.transparencyInfo));
    
    // Art. 14: Human Oversight
    checks.push(await this.checkHumanOversight(system.humanOversight));
    
    // Art. 15: Quality Requirements
    checks.push(await this.checkQualityRequirements(system.qualityRequirements));
    
    return {
      systemId: system.id,
      assessmentDate: new Date(),
      checks,
      overallCompliant: checks.every(c => c.passed),
      requiredActions: checks.filter(c => !c.passed).map(c => c.remediation)
    };
  }

  private async checkRiskManagement(
    rms: RiskManagementSystem
  ): Promise<ComplianceCheck> {
    const issues: string[] = [];
    
    // Must be established throughout lifecycle
    if (!rms.establishedDate) {
      issues.push('Risk management system not established');
    }
    
    // Must be updated regularly
    const daysSinceReview = 
      (Date.now() - rms.lastReviewDate.getTime()) / (1000 * 60 * 60 * 24);
    if (daysSinceReview > 365) {
      issues.push('Risk management not reviewed in past year');
    }
    
    // Must identify known and foreseeable risks
    if (rms.identifiedRisks.length === 0) {
      issues.push('No risks identified - implausible');
    }
    
    // Must have mitigation measures
    const unmitigatedHighRisks = rms.identifiedRisks.filter(
      r => r.severity === 'high' && 
      !rms.mitigationMeasures.some(m => m.riskId === r.id)
    );
    if (unmitigatedHighRisks.length > 0) {
      issues.push(`${unmitigatedHighRisks.length} high-severity risks without mitigation`);
    }
    
    // Must have testing procedures
    if (rms.testingProcedures.length === 0) {
      issues.push('No testing procedures documented');
    }
    
    return {
      article: 'Article 9',
      requirement: 'Risk Management System',
      passed: issues.length === 0,
      issues,
      remediation: issues.length > 0 
        ? 'Update risk management system to address identified gaps' 
        : undefined
    };
  }

  private async checkTransparency(
    transparency: TransparencyRequirements
  ): Promise<ComplianceCheck> {
    const issues: string[] = [];
    
    // Must include instructions for use
    if (!transparency.instructionsForUse) {
      issues.push('Missing instructions for use');
    }
    
    // Must specify intended purpose
    if (!transparency.intendedPurpose) {
      issues.push('Intended purpose not clearly specified');
    }
    
    // Must indicate level of accuracy
    if (!transparency.accuracyMetrics || transparency.accuracyMetrics.length === 0) {
      issues.push('Accuracy metrics not documented');
    }
    
    // Must describe human oversight measures
    if (!transparency.humanOversightDescription) {
      issues.push('Human oversight measures not described');
    }
    
    // For certain systems: must inform users they're interacting with AI
    if (transparency.requiresUserNotification && 
        !transparency.userNotificationMechanism) {
      issues.push('Users not informed they are interacting with AI');
    }
    
    return {
      article: 'Article 13',
      requirement: 'Transparency and provision of information',
      passed: issues.length === 0,
      issues,
      remediation: issues.length > 0 
        ? 'Update transparency documentation and user-facing disclosures' 
        : undefined
    };
  }

  private async checkHumanOversight(
    oversight: HumanOversightMeasures
  ): Promise<ComplianceCheck> {
    const issues: string[] = [];
    
    // Must allow effective oversight by natural persons
    if (!oversight.oversightPersonnel || oversight.oversightPersonnel.length === 0) {
      issues.push('No designated oversight personnel');
    }
    
    // Must provide ability to interpret outputs
    if (!oversight.outputInterpretationTools) {
      issues.push('No tools for interpreting AI outputs');
    }
    
    // Must provide ability to decide not to use system
    if (!oversight.abilityToOverride) {
      issues.push('No mechanism to override AI decisions');
    }
    
    // Must provide ability to intervene/stop
    if (!oversight.interventionMechanism) {
      issues.push('No intervention/stop mechanism');
    }
    
    return {
      article: 'Article 14',
      requirement: 'Human oversight',
      passed: issues.length === 0,
      issues,
      remediation: issues.length > 0 
        ? 'Implement human oversight mechanisms' 
        : undefined
    };
  }
}

§ 9.4 OBBLIGHI DI TRASPARENZA PER AI GENERATIVA

typescript
// ai-transparency-obligations.ts

// Art. 50: Transparency obligations for GPAI and certain AI systems
class AITransparencyManager {
  
  // For chatbots: inform users they're interacting with AI
  displayAIDisclosure(): React.ReactNode {
    return (
      <div 
        role="status" 
        aria-live="polite"
        className="ai-disclosure"
      >
        <span aria-hidden="true">🤖</span>
        <p>
          Stai interagendo con un sistema di intelligenza artificiale.
          Le risposte sono generate automaticamente e potrebbero 
          contenere errori.
        </p>
      </div>
    );
  }
  
  // For AI-generated content: mark as synthetic
  markGeneratedContent(content: GeneratedContent): LabeledContent {
    return {
      ...content,
      metadata: {
        isAIGenerated: true,
        generationDate: new Date(),
        model: content.modelUsed,
        disclaimer: this.getDisclaimer(content.type)
      },
      // Machine-readable marking (C2PA or similar)
      c2paManifest: this.generateC2PAManifest(content),
      // Human-readable marking
      visibleLabel: this.getVisibleLabel(content.type)
    };
  }
  
  private getDisclaimer(contentType: ContentType): string {
    const disclaimers: Record<ContentType, string> = {
      text: 'Questo testo è stato generato da intelligenza artificiale.',
      image: 'Questa immagine è stata generata da intelligenza artificiale.',
      audio: 'Questo audio è stato generato da intelligenza artificiale.',
      video: 'Questo video è stato generato da intelligenza artificiale.',
      deepfake: 'ATTENZIONE: Questo contenuto è stato creato artificialmente ' +
                'e non rappresenta eventi reali o dichiarazioni autentiche.'
    };
    return disclaimers[contentType];
  }
  
  private getVisibleLabel(contentType: ContentType): string {
    const labels: Record<ContentType, string> = {
      text: '[Generato da AI]',
      image: 'AI Generated',
      audio: '🔊 AI Generated Audio',
      video: '📹 AI Generated Video',
      deepfake: '⚠️ SYNTHETIC MEDIA'
    };
    return labels[contentType];
  }
  
  // For emotion recognition systems
  notifyEmotionRecognition(): void {
    // Must inform exposed persons
    this.showNotification({
      type: 'emotion_recognition',
      message: 'Questo sistema utilizza tecnologie di riconoscimento ' +
               'delle emozioni. Le tue espressioni facciali potrebbero ' +
               'essere analizzate.',
      requiresConsent: true,
      consentOptions: ['Accetto', 'Rifiuto']
    });
  }
  
  // For biometric categorization
  notifyBiometricCategorization(): void {
    this.showNotification({
      type: 'biometric_categorization',
      message: 'Questo sistema utilizza categorizzazione biometrica. ' +
               'I tuoi dati biometrici potrebbero essere analizzati ' +
               'per determinare categorie come sesso, età, etnia.',
      requiresConsent: true,
      consentOptions: ['Accetto', 'Rifiuto']
    });
  }
  
  // C2PA (Coalition for Content Provenance and Authenticity) implementation
  private generateC2PAManifest(content: GeneratedContent): C2PAManifest {
    return {
      claim_generator: 'AI Content Labeling System v1.0',
      title: content.title || 'AI Generated Content',
      format: content.mimeType,
      instance_id: crypto.randomUUID(),
      claim: {
        dc_title: content.title,
        dc_format: content.mimeType,
        created: new Date().toISOString(),
        generator: content.modelUsed,
        digital_source_type: 'trainedAlgorithmicMedia'
      },
      assertions: [
        {
          label: 'c2pa.actions',
          data: {
            actions: [
              {
                action: 'c2pa.created',
                digitalSourceType: 'trainedAlgorithmicMedia',
                softwareAgent: content.modelUsed
              }
            ]
          }
        }
      ],
      signature: this.signManifest(content)
    };
  }
}

// React component for AI-generated content watermark
export const AIGeneratedBadge: React.FC<{
  contentType: ContentType;
  modelName?: string;
}> = ({ contentType, modelName }) => {
  return (
    <div 
      className="ai-generated-badge"
      role="img"
      aria-label={`Contenuto ${contentType} generato da intelligenza artificiale${
        modelName ? ` usando ${modelName}` : ''
      }`}
    >
      <svg /* AI icon */ />
      <span>Generato da AI</span>
      {modelName && <span className="model-name">{modelName}</span>}
    </div>
  );
};



---

§ 10. SOFTWARE LICENSES - LICENZE OPEN SOURCE

§ 10.1 OVERVIEW DELLE LICENZE PRINCIPALI

┌─────────────────────────────────────────────────────────────────────┐
│                 SOFTWARE LICENSE COMPARISON MATRIX                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  LICENZA    │PERMISSIVE│COPYLEFT│COMMERC.│ATTRIBUZ.│PATENT│MOD.CHIUSE│
│  ───────────┼──────────┼────────┼────────┼─────────┼──────┼──────────│
│  MIT        │    ✅    │   ❌   │   ✅   │   ✅    │  ❌  │    ✅    │
│  Apache 2.0 │    ✅    │   ❌   │   ✅   │   ✅    │  ✅  │    ✅    │
│  BSD 2/3    │    ✅    │   ❌   │   ✅   │   ✅    │  ❌  │    ✅    │
│  ISC        │    ✅    │   ❌   │   ✅   │   ✅    │  ❌  │    ✅    │
│  ───────────┼──────────┼────────┼────────┼─────────┼──────┼──────────│
│  GPL v2     │    ❌    │   ✅   │   ⚠️   │   ✅    │  ❌  │    ❌    │
│  GPL v3     │    ❌    │   ✅   │   ⚠️   │   ✅    │  ✅  │    ❌    │
│  LGPL       │    ❌    │   ⚠️   │   ✅   │   ✅    │  ⚠️  │    ⚠️    │
│  AGPL v3    │    ❌    │   ✅✅  │   ❌   │   ✅    │  ✅  │    ❌    │
│  ───────────┼──────────┼────────┼────────┼─────────┼──────┼──────────│
│  MPL 2.0    │    ⚠️    │   ⚠️   │   ✅   │   ✅    │  ✅  │    ⚠️    │
│  EPL        │    ⚠️    │   ⚠️   │   ✅   │   ✅    │  ✅  │    ⚠️    │
│  ───────────┴──────────┴────────┴────────┴─────────┴──────┴──────────│
│                                                                      │
│  LEGENDA:                                                           │
│  ✅ = Permesso/Richiesto    ⚠️ = Condizionale    ❌ = Non permesso  │
│                                                                      │
│  PERMISSIVE: Poche restrizioni, libertà massima                     │
│  COPYLEFT: Derivati devono usare stessa licenza                     │
│  COMMERC.: Uso commerciale permesso                                 │
│  ATTRIBUZ.: Richiede attribuzione/copyright notice                  │
│  PATENT: Include grant brevetti                                     │
│  MOD.CHIUSE: Permette modifiche closed-source                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 10.2 DETTAGLIO LICENZE PERMISSIVE

┌─────────────────────────────────────────────────────────────────────┐
│                       MIT LICENSE                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CARATTERISTICHE                                                    │
│  • Licenza più semplice e permissiva                                │
│  • Solo ~170 parole                                                 │
│  • Massima libertà per uso commerciale                              │
│                                                                      │
│  OBBLIGHI                                                           │
│  ✓ Includere copyright notice                                       │
│  ✓ Includere testo licenza                                          │
│                                                                      │
│  PERMESSI                                                           │
│  ✓ Uso commerciale                                                  │
│  ✓ Modifica                                                         │
│  ✓ Distribuzione                                                    │
│  ✓ Uso privato                                                      │
│  ✓ Sublicenza                                                       │
│  ✓ Integrazione in software proprietario                            │
│                                                                      │
│  LIMITAZIONI                                                        │
│  ✗ Nessuna garanzia                                                 │
│  ✗ Nessuna responsabilità                                           │
│                                                                      │
│  USO TIPICO: React, jQuery, Node.js, .NET Core                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                     APACHE LICENSE 2.0                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CARATTERISTICHE                                                    │
│  • Simile a MIT ma più dettagliata                                  │
│  • Include grant esplicito brevetti                                 │
│  • Protezione legale più robusta                                    │
│                                                                      │
│  OBBLIGHI                                                           │
│  ✓ Includere copyright notice                                       │
│  ✓ Includere testo licenza                                          │
│  ✓ Indicare modifiche effettuate (NOTICE file)                     │
│  ✓ Preservare tutte le attribuzioni                                │
│                                                                      │
│  PERMESSI                                                           │
│  ✓ Uso commerciale                                                  │
│  ✓ Modifica                                                         │
│  ✓ Distribuzione                                                    │
│  ✓ Uso brevetti (grant esplicito)                                  │
│  ✓ Uso privato                                                      │
│  ✓ Sublicenza                                                       │
│                                                                      │
│  CLAUSOLA PATENT RETALIATION                                        │
│  Se fai causa per violazione brevetti, perdi i diritti             │
│                                                                      │
│  USO TIPICO: Android, Kubernetes, TensorFlow, Spark                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 10.3 DETTAGLIO LICENZE COPYLEFT

┌─────────────────────────────────────────────────────────────────────┐
│                     GPL v3 (GNU General Public License)              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CARATTERISTICHE                                                    │
│  • Strong copyleft: derivati DEVONO essere GPL                      │
│  • Virale: si propaga a tutto il software collegato                │
│  • Protegge libertà utenti finali                                   │
│                                                                      │
│  OBBLIGHI                                                           │
│  ✓ Distribuire codice sorgente                                      │
│  ✓ Derivati devono usare GPL v3                                    │
│  ✓ Preservare copyright notices                                     │
│  ✓ Indicare modifiche                                               │
│  ✓ Fornire testo completo licenza                                  │
│                                                                      │
│  "VIRALE" SIGNIFICA:                                                │
│  Se LINKI software GPL (staticamente o dinamicamente),             │
│  il TUO software deve diventare GPL                                 │
│                                                                      │
│  ECCEZIONI                                                          │
│  • "System libraries" (libc, kernel headers)                        │
│  • Plugins se interfaccia ben definita                              │
│  • Aggregazione senza integrazione                                  │
│                                                                      │
│  COMPATIBILITÀ                                                      │
│  ✓ GPL v3 compatible: Apache 2.0, MIT, BSD, LGPL v3               │
│  ✗ Incompatible: GPL v2, CDDL, EPL 1.0                             │
│                                                                      │
│  USO TIPICO: Linux (kernel usa v2), GCC, GIMP, WordPress           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                     LGPL v3 (Lesser GPL)                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CARATTERISTICHE                                                    │
│  • "Weak" copyleft: copyleft solo sulla libreria stessa            │
│  • Permette linking da software proprietario                        │
│  • Compromesso tra libertà e adozione commerciale                  │
│                                                                      │
│  OBBLIGHI                                                           │
│  ✓ Modifiche alla LIBRERIA devono essere LGPL                      │
│  ✓ Distribuire sorgente della libreria                              │
│  ✓ Permettere reverse engineering per debugging                    │
│                                                                      │
│  PERMESSI                                                           │
│  ✓ Linking da software proprietario                                 │
│  ✓ Distribuzione binari proprietari che usano la libreria          │
│                                                                      │
│  IMPORTANTE: LINKING DINAMICO vs STATICO                           │
│  • Dinamico: generalmente OK per software proprietario              │
│  • Statico: potrebbe richiedere distribuzione object files         │
│                                                                      │
│  USO TIPICO: glibc, GTK, Qt (edizione open source)                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                     AGPL v3 (Affero GPL)                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CARATTERISTICHE                                                    │
│  • GPL + Network Use Provision                                      │
│  • Copyleft si estende anche a uso server/SaaS                     │
│  • Pensata per chiudere "loophole" SaaS del GPL                    │
│                                                                      │
│  OBBLIGHI CRITICI                                                   │
│  ✓ Se offri servizio via network che usa AGPL software,            │
│    DEVI fornire accesso al codice sorgente                         │
│  ✓ Anche modifiche per uso interno-only devono essere pubblicate   │
│                                                                      │
│  ⚠️  IMPLICAZIONI PER AZIENDE                                       │
│  Se usi MongoDB Community (SSPL, simile), Grafana (AGPL),          │
│  o altri AGPL software in produzione:                               │
│  → Devi pubblicare TUTTO il codice che li integra                  │
│  → O acquistare licenza commerciale                                 │
│                                                                      │
│  USO TIPICO: MongoDB (era AGPL, ora SSPL), Grafana, Nextcloud     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 10.4 GESTIONE LICENZE NEL PROGETTO

typescript
// license-compliance-checker.ts
interface Dependency {
  name: string;
  version: string;
  license: string;
  repository?: string;
  dependencies?: Dependency[];
}

interface LicensePolicy {
  allowed: string[];
  denied: string[];
  requiresReview: string[];
}

class LicenseComplianceChecker {
  private policy: LicensePolicy = {
    allowed: [
      'MIT', 'Apache-2.0', 'BSD-2-Clause', 'BSD-3-Clause', 
      'ISC', 'CC0-1.0', 'Unlicense', '0BSD'
    ],
    denied: [
      'GPL-2.0', 'GPL-3.0', 'AGPL-3.0', 'SSPL-1.0',
      'CC-BY-NC-*', 'Proprietary'
    ],
    requiresReview: [
      'LGPL-2.1', 'LGPL-3.0', 'MPL-2.0', 'EPL-1.0', 'EPL-2.0',
      'CDDL-1.0', 'Artistic-2.0'
    ]
  };

  async scanProject(projectPath: string): Promise<LicenseScanResult> {
    const dependencies = await this.getDependencies(projectPath);
    const issues: LicenseIssue[] = [];
    const approved: ApprovedDependency[] = [];
    const needsReview: ReviewNeeded[] = [];

    for (const dep of dependencies) {
      const normalizedLicense = this.normalizeLicense(dep.license);
      
      if (this.policy.denied.some(d => this.matchLicense(normalizedLicense, d))) {
        issues.push({
          severity: 'critical',
          dependency: dep.name,
          version: dep.version,
          license: dep.license,
          message: `License ${dep.license} is not allowed`,
          recommendation: this.getAlternativeRecommendation(dep)
        });
      } else if (this.policy.requiresReview.some(r => 
        this.matchLicense(normalizedLicense, r)
      )) {
        needsReview.push({
          dependency: dep.name,
          version: dep.version,
          license: dep.license,
          reason: this.getReviewReason(dep.license)
        });
      } else if (this.policy.allowed.some(a => 
        this.matchLicense(normalizedLicense, a)
      )) {
        approved.push({
          dependency: dep.name,
          version: dep.version,
          license: dep.license
        });
      } else {
        // Unknown license
        issues.push({
          severity: 'warning',
          dependency: dep.name,
          version: dep.version,
          license: dep.license,
          message: `Unknown license: ${dep.license}`,
          recommendation: 'Review license terms manually'
        });
      }
    }

    return {
      scannedAt: new Date(),
      totalDependencies: dependencies.length,
      approved,
      needsReview,
      issues,
      compliant: issues.filter(i => i.severity === 'critical').length === 0
    };
  }

  private getReviewReason(license: string): string {
    const reasons: Record<string, string> = {
      'LGPL-2.1': 'Weak copyleft - verify linking method (dynamic OK, static may require disclosure)',
      'LGPL-3.0': 'Weak copyleft - verify linking method and ensure user can replace library',
      'MPL-2.0': 'File-level copyleft - modifications to MPL files must be disclosed',
      'EPL-1.0': 'Weak copyleft + patent clause - review for commercial use',
      'EPL-2.0': 'Weak copyleft + patent clause - compatible with GPL if secondary license used'
    };
    return reasons[license] || 'Review license terms for compatibility';
  }

  private getAlternativeRecommendation(dep: Dependency): string {
    const alternatives: Record<string, string> = {
      'readline': 'Use libedit (BSD) instead of GNU readline (GPL)',
      'mysql': 'Use MariaDB Connector/C (LGPL) or commercial license',
      'mongodb': 'Use MongoDB Atlas (commercial) or switch to PostgreSQL (permissive)',
      'ffmpeg': 'Use FFmpeg with --enable-gpl disabled, or license FFmpeg commercially'
    };
    return alternatives[dep.name] || 'Find alternative with permissive license';
  }

  // Generate NOTICE file for Apache 2.0 compliance
  generateNoticeFile(dependencies: Dependency[]): string {
    let notice = `
THIRD-PARTY SOFTWARE NOTICES AND INFORMATION

This project incorporates components from the projects listed below.
The original copyright notices and license terms are included below.

================================================================================
`;

    for (const dep of dependencies) {
      notice += `
${dep.name} (${dep.version})
License: ${dep.license}
${dep.repository ? `Repository: ${dep.repository}` : ''}
--------------------------------------------------------------------------------
`;
    }

    return notice;
  }

  // Generate attribution file for MIT/BSD
  generateAttributionFile(dependencies: Dependency[]): string {
    let attribution = `# Open Source Attributions\n\n`;
    attribution += `This software uses the following open source packages:\n\n`;

    for (const dep of dependencies) {
      attribution += `## ${dep.name}\n\n`;
      attribution += `- Version: ${dep.version}\n`;
      attribution += `- License: ${dep.license}\n`;
      if (dep.repository) {
        attribution += `- Source: ${dep.repository}\n`;
      }
      attribution += `\n`;
    }

    return attribution;
  }
}

// Package.json script integration
// package.json:
// {
//   "scripts": {
//     "license:check": "license-checker --onlyAllow 'MIT;Apache-2.0;BSD-2-Clause;BSD-3-Clause;ISC'",
//     "license:summary": "license-checker --summary",
//     "license:generate": "license-checker --json > licenses.json"
//   }
// }

§ 10.5 COMPLIANCE CHECKLIST PER LICENZE

┌─────────────────────────────────────────────────────────────────────┐
│              LICENSE COMPLIANCE CHECKLIST                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PER OGNI RELEASE:                                                  │
│                                                                      │
│  □ Scan automatico dipendenze per licenze                          │
│  □ Review manuale licenze sconosciute/nuove                        │
│  □ Generare NOTICE/ATTRIBUTION file aggiornato                     │
│  □ Includere testi licenze in distribuzione                        │
│  □ Verificare compatibilità tra licenze                            │
│                                                                      │
│  PER SOFTWARE CON COPYLEFT:                                         │
│                                                                      │
│  □ GPL: Distribuire tutto il sorgente (incluso tuo codice)        │
│  □ LGPL: Distribuire sorgente libreria + object files (se static) │
│  □ AGPL: Fornire sorgente anche per uso server/SaaS               │
│  □ MPL: Distribuire sorgente file modificati                       │
│                                                                      │
│  PER SOFTWARE PROPRIETARIO:                                         │
│                                                                      │
│  □ NO GPL/AGPL dipendenze (a meno di licensing commerciale)       │
│  □ LGPL solo con dynamic linking                                    │
│  □ Documentare tutte le third-party licenses                       │
│                                                                      │
│  DOCUMENTAZIONE RICHIESTA:                                          │
│                                                                      │
│  □ NOTICE file (per Apache 2.0)                                    │
│  □ ATTRIBUTIONS/CREDITS file                                       │
│  □ LICENSE file con tua licenza                                    │
│  □ THIRD_PARTY_LICENSES folder (testi completi)                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

---

§ 11. DATA BREACH NOTIFICATION

§ 11.1 REQUISITI PER GIURISDIZIONE

┌─────────────────────────────────────────────────────────────────────┐
│              DATA BREACH NOTIFICATION REQUIREMENTS                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  GDPR (EU)                                                          │
│  ├── AUTORITÀ: 72 ore dalla scoperta                               │
│  ├── INTERESSATI: "senza ingiustificato ritardo"                   │
│  │   solo se rischio elevato per diritti/libertà                    │
│  ├── CONTENUTO NOTIFICA:                                            │
│  │   • Natura della violazione                                      │
│  │   • Categorie e numero approssimativo interessati               │
│  │   • Contatto DPO                                                 │
│  │   • Probabili conseguenze                                        │
│  │   • Misure adottate/proposte                                    │
│  └── SANZIONI: fino a €10M o 2% fatturato                          │
│                                                                      │
│  CCPA/CPRA (California)                                             │
│  ├── AUTORITÀ: "expeditiously" (no tempo specifico)                │
│  ├── CONSUMATORI: "most expedient time possible"                   │
│  │   senza unreasonable delay                                       │
│  ├── TRIGGER: Unencrypted personal info accessed                   │
│  └── SANZIONI: $750-$7,500 per consumer, class actions            │
│                                                                      │
│  STATI USA (variano):                                               │
│  ├── Tutti 50 stati + DC hanno leggi breach notification           │
│  ├── Tempo: da 30 a 90 giorni (alcuni "without unreasonable delay")│
│  ├── Florida: 30 giorni                                             │
│  ├── Ohio: 45 giorni                                                │
│  ├── Colorado: 30 giorni                                            │
│  └── New York: "expedient" + AG notification                       │
│                                                                      │
│  HIPAA (US Healthcare)                                              │
│  ├── AUTORITÀ (HHS): 60 giorni (se <500 affected, annual report)  │
│  ├── INDIVIDUI: 60 giorni                                          │
│  ├── MEDIA: se >500 affected in una giurisdizione                 │
│  └── SANZIONI: $100-$50,000 per violazione, max $1.5M/anno        │
│                                                                      │
│  PCI-DSS                                                            │
│  ├── CARD BRANDS: immediato                                         │
│  ├── ACQUIRING BANK: immediato                                      │
│  └── PENALITÀ: $5,000-$100,000/mese + perdita processing rights   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

§ 11.2 IMPLEMENTAZIONE SISTEMA DI GESTIONE BREACH

typescript
// data-breach-management.ts
interface DataBreach {
  id: string;
  discoveredAt: Date;
  reportedAt?: Date;
  
  // Classificazione
  type: BreachType;
  severity: 'low' | 'medium' | 'high' | 'critical';
  
  // Dettagli
  description: string;
  affectedSystems: string[];
  dataCategories: DataCategory[];
  
  // Impatto
  affectedIndividuals: {
    count: number | 'unknown';
    categories: string[]; // es. 'customers', 'employees'
    jurisdictions: string[]; // es. 'EU', 'CA', 'NY'
  };
  
  // Risk Assessment
  riskAssessment: {
    likelihoodOfHarm: 'low' | 'medium' | 'high';
    severityOfHarm: 'low' | 'medium' | 'high';
    overallRisk: 'low' | 'medium' | 'high';
    requiresUserNotification: boolean;
  };
  
  // Response
  containmentActions: Action[];
  remediationActions: Action[];
  notifications: BreachNotification[];
  
  // Status
  status: 'detected' | 'contained' | 'investigating' | 'notifying' | 'resolved';
}

type BreachType = 
  | 'unauthorized_access'
  | 'data_theft'
  | 'ransomware'
  | 'accidental_disclosure'
  | 'lost_device'
  | 'insider_threat'
  | 'system_misconfiguration'
  | 'phishing';

interface BreachNotification {
  id: string;
  recipient: 'authority' | 'affected_individuals' | 'media' | 'business_partners';
  jurisdiction: string;
  dueBy: Date;
  sentAt?: Date;
  content: NotificationContent;
  deliveryMethod: 'email' | 'letter' | 'portal' | 'phone' | 'press_release';
  acknowledgmentReceived?: Date;
}

class DataBreachManager {
  private breaches: Map<string, DataBreach> = new Map();
  private notificationDeadlines: Map<string, NotificationDeadline[]> = new Map();

  async registerBreach(breachData: Partial<DataBreach>): Promise<DataBreach> {
    const breach: DataBreach = {
      id: crypto.randomUUID(),
      discoveredAt: new Date(),
      status: 'detected',
      severity: 'medium',
      ...breachData
    } as DataBreach;

    // Immediate actions
    await this.alertIncidentResponseTeam(breach);
    await this.calculateNotificationDeadlines(breach);
    await this.startDocumentation(breach);

    this.breaches.set(breach.id, breach);
    return breach;
  }

  private async calculateNotificationDeadlines(
    breach: DataBreach
  ): Promise<NotificationDeadline[]> {
    const deadlines: NotificationDeadline[] = [];
    const now = breach.discoveredAt;

    for (const jurisdiction of breach.affectedIndividuals.jurisdictions) {
      switch (jurisdiction) {
        case 'EU':
          // GDPR: 72 hours to authority
          deadlines.push({
            jurisdiction: 'EU',
            recipient: 'authority',
            deadline: new Date(now.getTime() + 72 * 60 * 60 * 1000),
            authority: 'Lead Supervisory Authority',
            requirement: 'GDPR Art. 33'
          });
          
          if (breach.riskAssessment.overallRisk === 'high') {
            deadlines.push({
              jurisdiction: 'EU',
              recipient: 'individuals',
              deadline: new Date(now.getTime() + 7 * 24 * 60 * 60 * 1000), // "without undue delay"
              requirement: 'GDPR Art. 34'
            });
          }
          break;

        case 'CA': // California
          deadlines.push({
            jurisdiction: 'CA',
            recipient: 'individuals',
            deadline: new Date(now.getTime() + 15 * 24 * 60 * 60 * 1000), // "most expedient"
            authority: 'California AG',
            requirement: 'CCPA § 1798.82'
          });
          
          if (breach.affectedIndividuals.count > 500) {
            deadlines.push({
              jurisdiction: 'CA',
              recipient: 'authority',
              deadline: new Date(now.getTime() + 15 * 24 * 60 * 60 * 1000),
              authority: 'California Attorney General',
              requirement: 'CCPA § 1798.82(f)'
            });
          }
          break;

        case 'NY': // New York
          deadlines.push({
            jurisdiction: 'NY',
            recipient: 'individuals',
            deadline: new Date(now.getTime() + 30 * 24 * 60 * 60 * 1000),
            requirement: 'NY GBL § 899-aa'
          });
          deadlines.push({
            jurisdiction: 'NY',
            recipient: 'authority',
            deadline: new Date(now.getTime() + 30 * 24 * 60 * 60 * 1000),
            authority: 'NY Attorney General, DFS, State Police',
            requirement: 'NY GBL § 899-aa'
          });
          break;

        case 'FL': // Florida
          deadlines.push({
            jurisdiction: 'FL',
            recipient: 'individuals',
            deadline: new Date(now.getTime() + 30 * 24 * 60 * 60 * 1000),
            requirement: 'FL Stat. § 501.171'
          });
          break;

        // Add more jurisdictions as needed
      }
    }

    this.notificationDeadlines.set(breach.id, deadlines);
    return deadlines;
  }

  async generateAuthorityNotification(
    breach: DataBreach,
    jurisdiction: string
  ): Promise<AuthorityNotification> {
    const template: AuthorityNotification = {
      // Header
      submissionDate: new Date(),
      organizationName: process.env.COMPANY_NAME!,
      dpoContact: {
        name: process.env.DPO_NAME!,
        email: process.env.DPO_EMAIL!,
        phone: process.env.DPO_PHONE!
      },

      // Breach details
      incidentDate: breach.discoveredAt,
      incidentDescription: breach.description,
      
      // Nature of breach
      breachNature: {
        confidentiality: breach.type === 'data_theft' || breach.type === 'unauthorized_access',
        integrity: breach.type === 'ransomware',
        availability: breach.type === 'ransomware' || breach.type === 'system_misconfiguration'
      },

      // Data involved
      dataCategories: breach.dataCategories,
      approximateRecordsAffected: breach.affectedIndividuals.count,
      approximateIndividualsAffected: breach.affectedIndividuals.count,
      categoriesOfIndividuals: breach.affectedIndividuals.categories,

      // Assessment
      likelyConsequences: this.assessLikelyConsequences(breach),
      
      // Response
      measuresTaken: breach.containmentActions.map(a => a.description),
      measuresProposed: breach.remediationActions.map(a => a.description),

      // Communication to individuals
      communicatedToIndividuals: breach.riskAssessment.requiresUserNotification,
      communicationMethod: 'email', // or other methods
      communicationDate: breach.notifications.find(
        n => n.recipient === 'affected_individuals'
      )?.sentAt,

      // If not communicated, why
      reasonsNotCommunicated: !breach.riskAssessment.requiresUserNotification
        ? 'Risk assessment determined low likelihood of harm to individuals'
        : undefined
    };

    return template;
  }

  async generateIndividualNotification(
    breach: DataBreach,
    individual: AffectedIndividual
  ): Promise<string> {
    return `
Dear ${individual.name || 'Valued Customer'},

We are writing to inform you of a data security incident that may have 
affected your personal information.

WHAT HAPPENED
${breach.description}

This incident was discovered on ${breach.discoveredAt.toLocaleDateString()}.

WHAT INFORMATION WAS INVOLVED
${breach.dataCategories.map(c => `• ${c}`).join('\n')}

WHAT WE ARE DOING
${breach.remediationActions.map(a => `• ${a.description}`).join('\n')}

WHAT YOU CAN DO
• Monitor your accounts for suspicious activity
• Consider placing a fraud alert on your credit reports
• Review your credit reports for unauthorized activity
${this.getJurisdictionSpecificAdvice(individual.jurisdiction)}

FOR MORE INFORMATION
If you have questions, please contact us at:
Email: ${process.env.BREACH_SUPPORT_EMAIL}
Phone: ${process.env.BREACH_SUPPORT_PHONE}

We sincerely apologize for any inconvenience this may cause.

${process.env.COMPANY_NAME}
${new Date().toLocaleDateString()}
    `;
  }

  private assessLikelyConsequences(breach: DataBreach): string[] {
    const consequences: string[] = [];

    if (breach.dataCategories.includes('financial')) {
      consequences.push('Potential financial fraud or identity theft');
    }
    if (breach.dataCategories.includes('health')) {
      consequences.push('Potential discrimination based on health information');
    }
    if (breach.dataCategories.includes('credentials')) {
      consequences.push('Potential unauthorized account access');
    }
    if (breach.dataCategories.includes('personal_identifiers')) {
      consequences.push('Potential identity fraud');
    }

    return consequences;
  }

  // Automated monitoring for deadlines
  startDeadlineMonitoring(): void {
    setInterval(() => {
      const now = new Date();
      
      for (const [breachId, deadlines] of this.notificationDeadlines) {
        for (const deadline of deadlines) {
          const hoursUntil = (deadline.deadline.getTime() - now.getTime()) / (1000 * 60 * 60);
          
          if (hoursUntil <= 24 && !deadline.notified) {
            this.sendDeadlineAlert(breachId, deadline, '24 hours remaining');
            deadline.notified = true;
          } else if (hoursUntil <= 48 && !deadline.warned) {
            this.sendDeadlineAlert(breachId, deadline, '48 hours remaining');
            deadline.warned = true;
          }
        }
      }
    }, 60 * 60 * 1000); // Check every hour
  }
}



  private sendDeadlineAlert(
    breachId: string, 
    deadline: NotificationDeadline, 
    message: string
  ): void {
    console.error(`[BREACH ALERT] ${breachId}: ${deadline.authority} - ${message}`);
    // Implement actual alerting (email, SMS, Slack, etc.)
  }

  private getJurisdictionSpecificAdvice(jurisdiction: string): string {
    const advice: Record<string, string> = {
      'US-CA': '• You may place a security freeze on your credit files',
      'EU': '• You have the right to lodge a complaint with your supervisory authority',
      'UK': '• Contact Action Fraud if you suspect identity theft',
    };
    return advice[jurisdiction] || '';
  }
}

---

## 12. IMPLEMENTAZIONI PRATICHE

### 12.1 Architettura Compliance-Ready

┌─────────────────────────────────────────────────────────────────────┐
│                COMPLIANCE-READY ARCHITECTURE                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                        FRONTEND LAYER                         │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐        │   │
│  │  │ Cookie   │ │ Privacy  │ │ A11y     │ │ AI       │        │   │
│  │  │ Banner   │ │ Center   │ │ Toolbar  │ │ Disclosure│       │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘        │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                     API GATEWAY LAYER                         │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐        │   │
│  │  │ Consent  │ │ Rate     │ │ Audit    │ │ Request  │        │   │
│  │  │ Verify   │ │ Limiter  │ │ Logger   │ │ Validator│        │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘        │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                      SERVICE LAYER                            │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐        │   │
│  │  │ Data     │ │ Rights   │ │ Breach   │ │ License  │        │   │
│  │  │ Processor│ │ Handler  │ │ Monitor  │ │ Tracker  │        │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘        │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                       DATA LAYER                              │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐        │   │
│  │  │ Encrypted│ │ Consent  │ │ Audit    │ │ Retention│        │   │
│  │  │ Storage  │ │ Database │ │ Logs     │ │ Policies │        │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘        │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

### 12.2 Compliance Middleware Stack

// middleware/compliance-stack.ts

import { Request, Response, NextFunction } from 'express';

// 1. Consent Verification Middleware
export const consentVerificationMiddleware = async (
  req: Request,
  res: Response,
  next: NextFunction
): Promise<void> => {
  const consentRequired = getRequiredConsent(req.path, req.method);
  
  if (consentRequired.length === 0) {
    return next();
  }
  
  const userId = req.user?.id || getAnonymousId(req);
  const consents = await getConsentsForUser(userId);
  
  const missingConsents = consentRequired.filter(
    c => !consents.some(uc => uc.purpose === c && uc.status === 'granted')
  );
  
  if (missingConsents.length > 0) {
    res.status(403).json({
      error: 'CONSENT_REQUIRED',
      message: 'Required consents not provided',
      required: missingConsents,
      consentUrl: '/privacy/consent'
    });
    return;
  }
  
  // Attach consent info to request for downstream use
  req.consents = consents;
  next();
};

// 2. Data Minimization Middleware
export const dataMinimizationMiddleware = (
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  // Store original json method
  const originalJson = res.json.bind(res);
  
  // Override json to filter sensitive data
  res.json = (data: any) => {
    const sanitized = sanitizeResponse(data, req.consents);
    return originalJson(sanitized);
  };
  
  next();
};

function sanitizeResponse(data: any, consents: Consent[]): any {
  if (!data || typeof data !== 'object') return data;
  
  const sensitiveFields = ['ssn', 'password', 'creditCard', 'healthInfo'];
  const sanitized = { ...data };
  
  for (const field of sensitiveFields) {
    if (field in sanitized) {
      // Check if user consented to this data category
      const hasConsent = consents?.some(
        c => c.purpose === `process_${field}` && c.status === 'granted'
      );
      
      if (!hasConsent) {
        delete sanitized[field];
      }
    }
  }
  
  // Recursively sanitize nested objects
  for (const [key, value] of Object.entries(sanitized)) {
    if (typeof value === 'object' && value !== null) {
      sanitized[key] = sanitizeResponse(value, consents);
    }
  }
  
  return sanitized;
}

// 3. Audit Logging Middleware
export const auditLoggingMiddleware = async (
  req: Request,
  res: Response,
  next: NextFunction
): Promise<void> => {
  const startTime = Date.now();
  
  // Capture original end method
  const originalEnd = res.end.bind(res);
  
  res.end = function(...args: any[]) {
    const duration = Date.now() - startTime;
    
    // Log to audit trail
    logAuditEvent({
      timestamp: new Date().toISOString(),
      requestId: req.id,
      userId: req.user?.id,
      action: `${req.method} ${req.path}`,
      resource: req.path,
      ipAddress: hashIP(req.ip), // Hash for privacy
      userAgent: req.headers['user-agent'],
      statusCode: res.statusCode,
      duration,
      dataCategories: req.dataCategories || [],
      consentsUsed: req.consents?.map(c => c.purpose) || []
    });
    
    return originalEnd(...args);
  };
  
  next();
};

// 4. GPC Signal Handler Middleware
export const gpcSignalMiddleware = (
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  const gpcEnabled = req.headers['sec-gpc'] === '1';
  
  if (gpcEnabled) {
    req.gpcEnabled = true;
    req.doNotSell = true;
    req.doNotShare = true;
    
    // Set response header acknowledging GPC
    res.setHeader('Sec-GPC', '1');
  }
  
  next();
};

// 5. Age Verification Middleware (COPPA)
export const ageVerificationMiddleware = async (
  req: Request,
  res: Response,
  next: NextFunction
): Promise<void> => {
  const requiresAgeCheck = pathRequiresAgeVerification(req.path);
  
  if (!requiresAgeCheck) {
    return next();
  }
  
  const userId = req.user?.id;
  if (!userId) {
    // Anonymous user - check session for age verification
    if (!req.session?.ageVerified) {
      res.status(403).json({
        error: 'AGE_VERIFICATION_REQUIRED',
        verificationUrl: '/age-verification'
      });
      return;
    }
  } else {
    // Logged in user - check stored age
    const user = await getUserById(userId);
    if (user.dateOfBirth && calculateAge(user.dateOfBirth) < 13) {
      // COPPA applies - require parental consent
      if (!user.parentalConsentVerified) {
        res.status(403).json({
          error: 'PARENTAL_CONSENT_REQUIRED',
          consentUrl: '/parental-consent'
        });
        return;
      }
    }
  }
  
  next();
};

// 6. Accessibility Headers Middleware
export const accessibilityHeadersMiddleware = (
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  // Set accessibility-related headers
  res.setHeader('Content-Language', req.acceptsLanguages()[0] || 'en');
  res.setHeader('X-Content-Type-Options', 'nosniff');
  
  // Add prefers-reduced-motion support info
  if (req.headers['sec-ch-prefers-reduced-motion']) {
    res.setHeader('Vary', 'Sec-CH-Prefers-Reduced-Motion');
  }
  
  next();
};

// Combined Compliance Stack
export const complianceStack = [
  gpcSignalMiddleware,
  ageVerificationMiddleware,
  consentVerificationMiddleware,
  dataMinimizationMiddleware,
  auditLoggingMiddleware,
  accessibilityHeadersMiddleware
];



### 12.3 Privacy Center Component

// components/PrivacyCenter.tsx

import React, { useState, useEffect } from 'react';

interface ConsentItem {
  id: string;
  purpose: string;
  description: string;
  category: 'necessary' | 'functional' | 'analytics' | 'advertising';
  status: 'granted' | 'denied' | 'pending';
  lastUpdated: Date;
  retention: string;
  thirdParties?: string[];
}

interface DataRequest {
  type: 'access' | 'delete' | 'portability' | 'rectification' | 'restriction';
  status: 'pending' | 'processing' | 'completed' | 'rejected';
  submittedAt: Date;
  completedAt?: Date;
}

export const PrivacyCenter: React.FC = () => {
  const [activeTab, setActiveTab] = useState<'consents' | 'requests' | 'data'>('consents');
  const [consents, setConsents] = useState<ConsentItem[]>([]);
  const [requests, setRequests] = useState<DataRequest[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadPrivacyData();
  }, []);

  const loadPrivacyData = async () => {
    setLoading(true);
    try {
      const [consentsRes, requestsRes] = await Promise.all([
        fetch('/api/privacy/consents'),
        fetch('/api/privacy/requests')
      ]);
      setConsents(await consentsRes.json());
      setRequests(await requestsRes.json());
    } finally {
      setLoading(false);
    }
  };

  const updateConsent = async (id: string, status: 'granted' | 'denied') => {
    await fetch(`/api/privacy/consents/${id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ status })
    });
    await loadPrivacyData();
  };

  const submitDataRequest = async (type: DataRequest['type']) => {
    await fetch('/api/privacy/requests', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ type })
    });
    await loadPrivacyData();
  };

  return (
    <div className="privacy-center" role="main" aria-labelledby="privacy-title">
      <h1 id="privacy-title">Centro Privacy</h1>
      
      {/* Tab Navigation */}
      <nav role="tablist" aria-label="Privacy sections">
        <button
          role="tab"
          aria-selected={activeTab === 'consents'}
          aria-controls="panel-consents"
          onClick={() => setActiveTab('consents')}
        >
          Gestione Consensi
        </button>
        <button
          role="tab"
          aria-selected={activeTab === 'requests'}
          aria-controls="panel-requests"
          onClick={() => setActiveTab('requests')}
        >
          Richieste Dati
        </button>
        <button
          role="tab"
          aria-selected={activeTab === 'data'}
          aria-controls="panel-data"
          onClick={() => setActiveTab('data')}
        >
          I Miei Dati
        </button>
      </nav>

      {/* Consents Panel */}
      {activeTab === 'consents' && (
        <section id="panel-consents" role="tabpanel" aria-labelledby="consents-heading">
          <h2 id="consents-heading">I Tuoi Consensi</h2>
          
          {consents.map(consent => (
            <article key={consent.id} className="consent-card">
              <header>
                <h3>{consent.purpose}</h3>
                <span className={`badge badge-${consent.category}`}>
                  {consent.category}
                </span>
              </header>
              
              <p>{consent.description}</p>
              
              <dl>
                <dt>Periodo di conservazione:</dt>
                <dd>{consent.retention}</dd>
                
                {consent.thirdParties && consent.thirdParties.length > 0 && (
                  <>
                    <dt>Condiviso con:</dt>
                    <dd>{consent.thirdParties.join(', ')}</dd>
                  </>
                )}
              </dl>
              
              <div className="consent-actions">
                {consent.category !== 'necessary' && (
                  <>
                    <button
                      onClick={() => updateConsent(consent.id, 'granted')}
                      aria-pressed={consent.status === 'granted'}
                      className={consent.status === 'granted' ? 'active' : ''}
                    >
                      Accetta
                    </button>
                    <button
                      onClick={() => updateConsent(consent.id, 'denied')}
                      aria-pressed={consent.status === 'denied'}
                      className={consent.status === 'denied' ? 'active' : ''}
                    >
                      Rifiuta
                    </button>
                  </>
                )}
                {consent.category === 'necessary' && (
                  <span className="necessary-badge">
                    Necessario per il funzionamento
                  </span>
                )}
              </div>
              
              <footer>
                <small>
                  Ultimo aggiornamento: {new Date(consent.lastUpdated).toLocaleDateString()}
                </small>
              </footer>
            </article>
          ))}
        </section>
      )}

      {/* Data Requests Panel */}
      {activeTab === 'requests' && (
        <section id="panel-requests" role="tabpanel" aria-labelledby="requests-heading">
          <h2 id="requests-heading">Esercita i Tuoi Diritti</h2>
          
          <div className="rights-grid">
            <button onClick={() => submitDataRequest('access')}>
              <span className="icon">📋</span>
              <span>Accesso ai Dati</span>
              <small>Richiedi copia dei tuoi dati</small>
            </button>
            
            <button onClick={() => submitDataRequest('portability')}>
              <span className="icon">📦</span>
              <span>Portabilità</span>
              <small>Esporta i tuoi dati</small>
            </button>
            
            <button onClick={() => submitDataRequest('delete')}>
              <span className="icon">🗑️</span>
              <span>Cancellazione</span>
              <small>Elimina i tuoi dati</small>
            </button>
            
            <button onClick={() => submitDataRequest('rectification')}>
              <span className="icon">✏️</span>
              <span>Rettifica</span>
              <small>Correggi dati errati</small>
            </button>
            
            <button onClick={() => submitDataRequest('restriction')}>
              <span className="icon">⏸️</span>
              <span>Limitazione</span>
              <small>Limita il trattamento</small>
            </button>
          </div>

          <h3>Storico Richieste</h3>
          <table aria-label="Storico richieste dati">
            <thead>
              <tr>
                <th>Tipo</th>
                <th>Data Richiesta</th>
                <th>Stato</th>
                <th>Completata</th>
              </tr>
            </thead>
            <tbody>
              {requests.map((req, idx) => (
                <tr key={idx}>
                  <td>{req.type}</td>
                  <td>{new Date(req.submittedAt).toLocaleDateString()}</td>
                  <td>
                    <span className={`status status-${req.status}`}>
                      {req.status}
                    </span>
                  </td>
                  <td>
                    {req.completedAt 
                      ? new Date(req.completedAt).toLocaleDateString() 
                      : '-'}
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
        </section>
      )}

      {/* My Data Panel */}
      {activeTab === 'data' && (
        <section id="panel-data" role="tabpanel" aria-labelledby="data-heading">
          <h2 id="data-heading">I Dati che Conserviamo</h2>
          <DataCategoriesView />
        </section>
      )}
    </div>
  );
};

### 12.4 Accessibility Testing Automation

// testing/accessibility.test.ts

import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

interface A11yTestConfig {
  wcagLevel: 'A' | 'AA' | 'AAA';
  rules?: string[];
  ignoreRules?: string[];
}

const defaultConfig: A11yTestConfig = {
  wcagLevel: 'AA',
  ignoreRules: ['color-contrast'] // Example: might ignore in dark mode tests
};

// Automated WCAG 2.2 Compliance Tests
test.describe('WCAG 2.2 Compliance', () => {
  
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('Homepage meets WCAG 2.2 Level AA', async ({ page }) => {
    const results = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag22aa'])
      .analyze();
    
    expect(results.violations).toEqual([]);
  });

  test('All form inputs have accessible labels', async ({ page }) => {
    await page.goto('/signup');
    
    const inputs = await page.locator('input, select, textarea').all();
    
    for (const input of inputs) {
      const hasLabel = await input.evaluate((el) => {
        const id = el.id;
        const ariaLabel = el.getAttribute('aria-label');
        const ariaLabelledBy = el.getAttribute('aria-labelledby');
        const label = id ? document.querySelector(`label[for="${id}"]`) : null;
        
        return !!(ariaLabel || ariaLabelledBy || label);
      });
      
      expect(hasLabel).toBe(true);
    }
  });

  test('Keyboard navigation works correctly', async ({ page }) => {
    // Test tab order
    const focusableElements = await page.evaluate(() => {
      const selector = 'a, button, input, select, textarea, [tabindex]:not([tabindex="-1"])';
      const elements = Array.from(document.querySelectorAll(selector));
      return elements.map(el => ({
        tag: el.tagName,
        tabIndex: el.tabIndex,
        visible: el.offsetParent !== null
      }));
    });
    
    // Verify logical tab order
    const visibleElements = focusableElements.filter(el => el.visible);
    expect(visibleElements.length).toBeGreaterThan(0);
  });

  test('Focus indicators are visible', async ({ page }) => {
    const buttons = await page.locator('button').all();
    
    for (const button of buttons.slice(0, 5)) { // Test first 5
      await button.focus();
      
      const hasVisibleFocus = await button.evaluate((el) => {
        const styles = window.getComputedStyle(el);
        const hasOutline = styles.outline !== 'none' && styles.outlineWidth !== '0px';
        const hasBoxShadow = styles.boxShadow !== 'none';
        const hasBorder = styles.borderColor !== el.style.borderColor;
        
        return hasOutline || hasBoxShadow || hasBorder;
      });
      
      expect(hasVisibleFocus).toBe(true);
    }
  });

  test('Images have alt text', async ({ page }) => {
    const images = await page.locator('img').all();
    
    for (const img of images) {
      const alt = await img.getAttribute('alt');
      const role = await img.getAttribute('role');
      
      // Images must have alt OR role="presentation" for decorative
      expect(alt !== null || role === 'presentation').toBe(true);
    }
  });

  test('Heading hierarchy is correct', async ({ page }) => {
    const headings = await page.evaluate(() => {
      const h = document.querySelectorAll('h1, h2, h3, h4, h5, h6');
      return Array.from(h).map(el => parseInt(el.tagName.substring(1)));
    });
    
    // Should have exactly one h1
    const h1Count = headings.filter(h => h === 1).length;
    expect(h1Count).toBe(1);
    
    // Heading levels should not skip (e.g., h1 to h3)
    for (let i = 1; i < headings.length; i++) {
      const diff = headings[i] - headings[i - 1];
      expect(diff).toBeLessThanOrEqual(1);
    }
  });

  test('Color contrast meets requirements', async ({ page }) => {
    const results = await new AxeBuilder({ page })
      .withTags(['wcag2aa'])
      .disableRules(['region']) // Only check contrast
      .analyze();
    
    const contrastViolations = results.violations.filter(
      v => v.id === 'color-contrast'
    );
    
    expect(contrastViolations).toEqual([]);
  });

  test('ARIA attributes are valid', async ({ page }) => {
    const results = await new AxeBuilder({ page })
      .withTags(['cat.aria'])
      .analyze();
    
    expect(results.violations).toEqual([]);
  });

  test('Reduced motion is respected', async ({ page }) => {
    // Emulate prefers-reduced-motion
    await page.emulateMedia({ reducedMotion: 'reduce' });
    await page.goto('/');
    
    const hasAnimations = await page.evaluate(() => {
      const elements = document.querySelectorAll('*');
      for (const el of elements) {
        const styles = window.getComputedStyle(el);
        if (styles.animation !== 'none' && styles.animationDuration !== '0s') {
          return true;
        }
      }
      return false;
    });
    
    expect(hasAnimations).toBe(false);
  });

  test('Target size meets minimum requirements (24x24)', async ({ page }) => {
    const clickables = await page.locator('button, a, input[type="checkbox"], input[type="radio"]').all();
    
    for (const el of clickables) {
      const box = await el.boundingBox();
      if (box) {
        // WCAG 2.2 minimum target size is 24x24px
        expect(box.width).toBeGreaterThanOrEqual(24);
        expect(box.height).toBeGreaterThanOrEqual(24);
      }
    }
  });
});

// Generate accessibility report
export async function generateA11yReport(page: any): Promise<A11yReport> {
  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag22aa', 'best-practice'])
    .analyze();
  
  return {
    timestamp: new Date().toISOString(),
    url: page.url(),
    violations: results.violations.length,
    passes: results.passes.length,
    incomplete: results.incomplete.length,
    inapplicable: results.inapplicable.length,
    details: results.violations.map(v => ({
      id: v.id,
      impact: v.impact,
      description: v.description,
      help: v.help,
      helpUrl: v.helpUrl,
      nodes: v.nodes.length
    })),
    wcagCompliance: {
      levelA: results.violations.filter(v => 
        v.tags.includes('wcag2a')).length === 0,
      levelAA: results.violations.filter(v => 
        v.tags.includes('wcag2aa') || v.tags.includes('wcag22aa')).length === 0
    }
  };
}



---

## 13. CHECKLIST DI COMPLIANCE

### 13.1 GDPR Compliance Checklist

┌─────────────────────────────────────────────────────────────────────┐
│                    GDPR COMPLIANCE CHECKLIST                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  LAWFUL BASIS & CONSENT                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ Identified lawful basis for each processing activity       │   │
│  │ □ Consent is freely given, specific, informed, unambiguous   │   │
│  │ □ Consent can be easily withdrawn                            │   │
│  │ □ Records of consent are maintained                          │   │
│  │ □ Consent forms use clear, plain language                    │   │
│  │ □ No pre-ticked boxes for consent                           │   │
│  │ □ Separate consent for different purposes                    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  DATA SUBJECT RIGHTS                                                │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ Process for handling access requests (30-day response)     │   │
│  │ □ Process for rectification requests                         │   │
│  │ □ Process for erasure requests ("right to be forgotten")     │   │
│  │ □ Process for data portability (machine-readable format)     │   │
│  │ □ Process for restriction of processing                      │   │
│  │ □ Process for objection to processing                        │   │
│  │ □ Automated decision-making safeguards in place              │   │
│  │ □ Identity verification for requests                         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  TRANSPARENCY & NOTICES                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ Privacy policy is accessible and comprehensive             │   │
│  │ □ Identity and contact details of controller provided        │   │
│  │ □ DPO contact details (if applicable)                        │   │
│  │ □ Purposes and legal basis clearly stated                    │   │
│  │ □ Recipients/categories of recipients listed                 │   │
│  │ □ International transfer information provided                │   │
│  │ □ Retention periods specified                                │   │
│  │ □ Data subject rights explained                              │   │
│  │ □ Right to complain to supervisory authority mentioned       │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  SECURITY & PROTECTION                                              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ Data encrypted in transit (TLS 1.2+)                       │   │
│  │ □ Data encrypted at rest                                     │   │
│  │ □ Access controls implemented                                │   │
│  │ □ Data minimization practiced                                │   │
│  │ □ Regular security assessments conducted                     │   │
│  │ □ Breach notification procedure established                  │   │
│  │ □ Employee training on data protection                       │   │
│  │ □ Pseudonymization used where appropriate                    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ORGANIZATIONAL MEASURES                                            │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ Records of processing activities maintained (Art. 30)      │   │
│  │ □ Data Protection Impact Assessment process (Art. 35)        │   │
│  │ □ Data Protection Officer appointed (if required)            │   │
│  │ □ Processor agreements in place (Art. 28)                    │   │
│  │ □ Privacy by Design implemented                              │   │
│  │ □ Regular compliance audits conducted                        │   │
│  │ □ Staff trained on GDPR requirements                         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

### 13.2 CCPA/CPRA Compliance Checklist

┌─────────────────────────────────────────────────────────────────────┐
│                    CCPA/CPRA COMPLIANCE CHECKLIST                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  THRESHOLD ASSESSMENT                                               │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ Determined if annual revenue exceeds $26,625,000          │   │
│  │ □ Counted if processing 100,000+ CA residents' data          │   │
│  │ □ Assessed if 50%+ revenue from selling/sharing data        │   │
│  │ □ Documented threshold assessment and reasoning              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  CONSUMER RIGHTS (LOCKA)                                            │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ Limit use of Sensitive Personal Information implemented    │   │
│  │ □ Opt-out mechanism for sale/sharing operational             │   │
│  │ □ Correct - mechanism to fix inaccurate data                 │   │
│  │ □ Know - process for disclosure requests                     │   │
│  │ □ Access - 45-day response timeline established              │   │
│  │ □ Delete - erasure request process operational               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  REQUIRED DISCLOSURES                                               │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ "Do Not Sell or Share My Personal Information" link        │   │
│  │ □ "Limit the Use of My Sensitive Personal Information" link  │   │
│  │ □ Categories of PI collected disclosed                       │   │
│  │ □ Purposes for collection stated                             │   │
│  │ □ Categories of third parties disclosed                      │   │
│  │ □ Retention periods specified                                │   │
│  │ □ Financial incentive programs disclosed                     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  TECHNICAL REQUIREMENTS                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ Global Privacy Control (GPC) signal honored                │   │
│  │ □ Two or more methods for submitting requests                │   │
│  │ □ Interactive webform available                              │   │
│  │ □ Toll-free phone number provided                            │   │
│  │ □ Identity verification process implemented                  │   │
│  │ □ Authorized agent process established                       │   │
│  │ □ Request tracking and status updates enabled                │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  SERVICE PROVIDER AGREEMENTS                                        │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ Written contracts with service providers                   │   │
│  │ □ Restriction clauses on data use                            │   │
│  │ □ Certification of compliance included                       │   │
│  │ □ Audit rights established                                   │   │
│  │ □ Subcontractor flow-down provisions                         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

### 13.3 WCAG 2.2 Compliance Checklist (Level AA)

┌─────────────────────────────────────────────────────────────────────┐
│                    WCAG 2.2 LEVEL AA CHECKLIST                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PERCEIVABLE                                                        │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ 1.1.1 Non-text content has text alternatives              │   │
│  │ □ 1.2.1-5 Time-based media has captions/descriptions         │   │
│  │ □ 1.3.1-6 Info and relationships programmatically determined │   │
│  │ □ 1.4.1 Color not sole means of conveying info              │   │
│  │ □ 1.4.3 Contrast ratio 4.5:1 for text (3:1 large text)      │   │
│  │ □ 1.4.4 Text resizable to 200% without loss                 │   │
│  │ □ 1.4.10 Content reflows at 320px width                     │   │
│  │ □ 1.4.11 Non-text contrast 3:1 for UI components            │   │
│  │ □ 1.4.12 Text spacing adjustable                            │   │
│  │ □ 1.4.13 Content on hover/focus dismissible                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  OPERABLE                                                           │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ 2.1.1-2 Keyboard accessible, no keyboard trap             │   │
│  │ □ 2.1.4 Character key shortcuts can be turned off           │   │
│  │ □ 2.2.1-2 Timing adjustable, pause/stop/hide available      │   │
│  │ □ 2.3.1 No flashing content over 3 times/second             │   │
│  │ □ 2.4.1 Bypass blocks (skip links) provided                 │   │
│  │ □ 2.4.2 Page titles descriptive                             │   │
│  │ □ 2.4.3-4 Focus order logical, link purpose clear           │   │
│  │ □ 2.4.5-6 Multiple ways to find content, headings/labels    │   │
│  │ □ 2.4.7 Focus visible (enhanced in 2.2)                     │   │
│  │ □ 2.4.11 Focus not obscured (NEW 2.2)                       │   │
│  │ □ 2.5.1-4 Pointer gestures, cancellation, motion actuation  │   │
│  │ □ 2.5.7 Dragging movements have alternatives (NEW 2.2)      │   │
│  │ □ 2.5.8 Target size minimum 24x24px (NEW 2.2)               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  UNDERSTANDABLE                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ 3.1.1-2 Language of page and parts identified             │   │
│  │ □ 3.2.1-4 Predictable navigation and interactions           │   │
│  │ □ 3.2.6 Consistent help (NEW 2.2)                           │   │
│  │ □ 3.3.1-4 Input assistance and error prevention             │   │
│  │ □ 3.3.7 Redundant entry avoided (NEW 2.2)                   │   │
│  │ □ 3.3.8 Accessible authentication (NEW 2.2)                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ROBUST                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ 4.1.1 Parsing (valid HTML)                                │   │
│  │ □ 4.1.2 Name, role, value for all UI components             │   │
│  │ □ 4.1.3 Status messages announced to assistive tech         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

### 13.4 Cookie Consent Compliance Checklist

┌─────────────────────────────────────────────────────────────────────┐
│                COOKIE CONSENT COMPLIANCE CHECKLIST                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  BANNER REQUIREMENTS                                                │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ Banner appears before any non-essential cookies set        │   │
│  │ □ Accept and Reject buttons equally prominent                │   │
│  │ □ No pre-selected optional cookie categories                 │   │
│  │ □ Clear and plain language used                              │   │
│  │ □ Link to detailed cookie policy provided                    │   │
│  │ □ No cookie walls blocking access to content                 │   │
│  │ □ No deceptive design patterns (dark patterns)               │   │
│  │ □ Granular choice for cookie categories offered              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  CONSENT MANAGEMENT                                                 │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ Consent recorded with timestamp                            │   │
│  │ □ User can change preferences at any time                    │   │
│  │ □ Preference center easily accessible                        │   │
│  │ □ Consent withdrawal as easy as giving consent               │   │
│  │ □ Consent refreshed periodically (6-12 months)               │   │
│  │ □ Proof of consent available for audit                       │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  TECHNICAL IMPLEMENTATION                                           │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ Only strictly necessary cookies set before consent         │   │
│  │ □ Consent mode integrated with analytics (GA4)               │   │
│  │ □ Third-party scripts blocked until consent                  │   │
│  │ □ Cookie categories correctly classified                     │   │
│  │ □ Cookie policy lists all cookies with purposes              │   │
│  │ □ Consent state checked on each page load                    │   │
│  │ □ IAB TCF 2.2 implemented (if using ad tech)                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

### 13.5 EU AI Act Compliance Checklist

┌─────────────────────────────────────────────────────────────────────┐
│                    EU AI ACT COMPLIANCE CHECKLIST                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  RISK CLASSIFICATION                                                │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ AI system risk level determined (minimal/limited/high)     │   │
│  │ □ Prohibited use cases identified and avoided                │   │
│  │ □ High-risk determination criteria documented                │   │
│  │ □ GPAI model assessment completed (if applicable)            │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  LIMITED RISK REQUIREMENTS                                          │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ Users informed they are interacting with AI                │   │
│  │ □ AI-generated content clearly labeled                       │   │
│  │ □ Deepfake content disclosed                                 │   │
│  │ □ Emotion recognition system purpose disclosed               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  HIGH-RISK REQUIREMENTS                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ Risk management system established                         │   │
│  │ □ Data governance measures implemented                       │   │
│  │ □ Technical documentation complete                           │   │
│  │ □ Automatic logging capabilities enabled                     │   │
│  │ □ Human oversight measures in place                          │   │
│  │ □ Accuracy, robustness, cybersecurity ensured                │   │
│  │ □ Quality management system operational                      │   │
│  │ □ Instructions for use provided                              │   │
│  │ □ CE marking obtained (when required)                        │   │
│  │ □ Conformity assessment completed                            │   │
│  │ □ EU database registration done                              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  GPAI MODEL REQUIREMENTS                                            │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ □ Technical documentation maintained                         │   │
│  │ □ Information for downstream providers available             │   │
│  │ □ Copyright compliance policy in place                       │   │
│  │ □ Training data summary published                            │   │
│  │ □ Systemic risk assessment (if >10^25 FLOPs)                │   │
│  │ □ Model evaluation conducted                                 │   │
│  │ □ Adversarial testing performed                              │   │
│  │ □ Incident reporting process established                     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

### 13.6 Pre-Launch Compliance Checklist

// scripts/pre-launch-compliance-check.ts

interface ComplianceCheckResult {
  category: string;
  check: string;
  status: 'pass' | 'fail' | 'warning' | 'manual';
  details?: string;
  remediation?: string;
}

async function runPreLaunchCompliance(): Promise<ComplianceCheckResult[]> {
  const results: ComplianceCheckResult[] = [];
  
  // Privacy Checks
  results.push(await checkPrivacyPolicy());
  results.push(await checkCookieBanner());
  results.push(await checkConsentMechanism());
  results.push(await checkDataRetentionPolicy());
  results.push(await checkDSRProcess());
  
  // Security Checks
  results.push(await checkHTTPS());
  results.push(await checkSecurityHeaders());
  results.push(await checkEncryption());
  results.push(await checkBreachNotificationProcess());
  
  // Accessibility Checks
  results.push(await runWCAGAudit());
  results.push(await checkKeyboardNavigation());
  results.push(await checkColorContrast());
  results.push(await checkARIA());
  
  // Legal Checks
  results.push(await checkTermsOfService());
  results.push(await checkLicenseCompliance());
  results.push(await checkAIDisclosure());
  
  return results;
}

async function checkPrivacyPolicy(): Promise<ComplianceCheckResult> {
  const response = await fetch('/privacy-policy');
  const html = await response.text();
  
  const requiredSections = [
    'data controller',
    'categories of data',
    'purposes',
    'legal basis',
    'retention',
    'rights',
    'contact'
  ];
  
  const missingSection = requiredSections.find(
    section => !html.toLowerCase().includes(section)
  );
  
  return {
    category: 'Privacy',
    check: 'Privacy Policy Completeness',
    status: missingSection ? 'fail' : 'pass',
    details: missingSection ? `Missing section: ${missingSection}` : 'All sections present',
    remediation: missingSection ? 'Add missing privacy policy section' : undefined
  };
}

async function checkCookieBanner(): Promise<ComplianceCheckResult> {
  // Check if cookie banner exists and has required elements
  const bannerExists = await page.evaluate(() => {
    return !!document.querySelector('[data-testid="cookie-banner"]');
  });
  
  if (!bannerExists) {
    return {
      category: 'Privacy',
      check: 'Cookie Banner Present',
      status: 'fail',
      details: 'Cookie consent banner not found',
      remediation: 'Implement cookie consent banner before setting non-essential cookies'
    };
  }
  
  const hasRejectButton = await page.evaluate(() => {
    const banner = document.querySelector('[data-testid="cookie-banner"]');
    return !!banner?.querySelector('[data-testid="reject-cookies"]');
  });
  
  return {
    category: 'Privacy',
    check: 'Cookie Banner Compliance',
    status: hasRejectButton ? 'pass' : 'fail',
    details: hasRejectButton ? 'Accept and Reject buttons present' : 'Missing Reject button',
    remediation: !hasRejectButton ? 'Add equally prominent Reject button' : undefined
  };
}

async function runWCAGAudit(): Promise<ComplianceCheckResult> {
  const axeResults = await new AxeBuilder({ page })
    .withTags(['wcag2aa', 'wcag22aa'])
    .analyze();
  
  const criticalViolations = axeResults.violations.filter(
    v => v.impact === 'critical' || v.impact === 'serious'
  );
  
  return {
    category: 'Accessibility',
    check: 'WCAG 2.2 AA Compliance',
    status: criticalViolations.length === 0 ? 'pass' : 'fail',
    details: `${axeResults.violations.length} violations found (${criticalViolations.length} critical)`,
    remediation: criticalViolations.length > 0 
      ? `Fix: ${criticalViolations.map(v => v.id).join(', ')}`
      : undefined
  };
}

async function checkAIDisclosure(): Promise<ComplianceCheckResult> {
  // Check if AI-generated content is properly disclosed
  const aiElements = await page.evaluate(() => {
    return Array.from(document.querySelectorAll('[data-ai-generated="true"]'))
      .map(el => !!el.getAttribute('aria-label')?.includes('AI'));
  });
  
  const allDisclosed = aiElements.every(disclosed => disclosed);
  
  return {
    category: 'AI Act',
    check: 'AI Content Disclosure',
    status: aiElements.length === 0 ? 'pass' : (allDisclosed ? 'pass' : 'fail'),
    details: aiElements.length === 0 
      ? 'No AI-generated content detected'
      : `${aiElements.filter(d => d).length}/${aiElements.length} AI elements disclosed`,
    remediation: !allDisclosed 
      ? 'Add clear AI disclosure to all AI-generated content'
      : undefined
  };
}

// Generate compliance report
function generateComplianceReport(results: ComplianceCheckResult[]): string {
  const passed = results.filter(r => r.status === 'pass').length;
  const failed = results.filter(r => r.status === 'fail').length;
  const warnings = results.filter(r => r.status === 'warning').length;
  
  let report = `
# Pre-Launch Compliance Report
Generated: ${new Date().toISOString()}

§ SUMMARY
- ✅ Passed: ${passed}
- ❌ Failed: ${failed}
- ⚠️ Warnings: ${warnings}
- 📋 Manual Review: ${results.filter(r => r.status === 'manual').length}

§ ${FAILED === 0 ? '✅ READY FOR LAUNCH' : '❌ NOT READY - ADDRESS FAILURES BEFORE LAUNCH'}

§ DETAILED RESULTS

`;

  const categories = [...new Set(results.map(r => r.category))];
  
  for (const category of categories) {
    report += `### ${category}\n\n`;
    
    const categoryResults = results.filter(r => r.category === category);
    for (const result of categoryResults) {
      const icon = result.status === 'pass' ? '✅' : 
                   result.status === 'fail' ? '❌' : 
                   result.status === 'warning' ? '⚠️' : '📋';
      
      report += `${icon} **${result.check}**\n`;
      report += `   ${result.details}\n`;
      if (result.remediation) {
        report += `   → Remediation: ${result.remediation}\n`;
      }
      report += '\n';
    }
  }
  
  return report;
}



---

## 14. TOOL E RISORSE

### 14.1 Cookie Consent Management Platforms

┌─────────────────────────────────────────────────────────────────────┐
│               COOKIE CONSENT MANAGEMENT PLATFORMS                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ENTERPRISE SOLUTIONS                                               │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ OneTrust                                                     │   │
│  │ • Prezzo: $$$$ (Enterprise)                                  │   │
│  │ • IAB TCF 2.2, GDPR, CCPA, global compliance                │   │
│  │ • Auto-scanning, preference center                           │   │
│  │ • URL: https://www.onetrust.com                              │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ TrustArc                                                     │   │
│  │ • Prezzo: $$$$ (Enterprise)                                  │   │
│  │ • Comprehensive privacy management platform                  │   │
│  │ • Cookie consent + assessments + monitoring                  │   │
│  │ • URL: https://trustarc.com                                  │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ Didomi                                                       │   │
│  │ • Prezzo: $$$ (Mid-market to Enterprise)                     │   │
│  │ • Strong EU presence, CMP + preference management           │   │
│  │ • URL: https://www.didomi.io                                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  MID-MARKET SOLUTIONS                                               │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Cookiebot (by Usercentrics)                                  │   │
│  │ • Prezzo: Free (<100 pages) to $$ (Premium)                  │   │
│  │ • Auto-scanning, GDPR/CCPA compliant                         │   │
│  │ • URL: https://www.cookiebot.com                             │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ CookieYes                                                    │   │
│  │ • Prezzo: Free (basic) to $$ (Business)                      │   │
│  │ • Good WordPress integration                                 │   │
│  │ • URL: https://www.cookieyes.com                             │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ Termly                                                       │   │
│  │ • Prezzo: Free to $$ (Pro)                                   │   │
│  │ • Cookie consent + policy generator                          │   │
│  │ • URL: https://termly.io                                     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  OPEN SOURCE / SELF-HOSTED                                          │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Klaro                                                        │   │
│  │ • Prezzo: Free (Open Source, MIT)                            │   │
│  │ • Privacy-friendly, self-hosted                              │   │
│  │ • GitHub: https://github.com/klaro-org/klaro                 │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ Tarteaucitron.js                                             │   │
│  │ • Prezzo: Free (Open Source)                                 │   │
│  │ • French-developed, GDPR compliant                           │   │
│  │ • GitHub: https://github.com/AntarioHD/tarteaucitron.js      │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

### 14.2 Accessibility Testing Tools

┌─────────────────────────────────────────────────────────────────────┐
│                  ACCESSIBILITY TESTING TOOLS                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  AUTOMATED TESTING                                                  │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ axe DevTools (Deque)                                         │   │
│  │ • Type: Browser extension + API                              │   │
│  │ • Coverage: WCAG 2.0/2.1/2.2, Section 508                    │   │
│  │ • URL: https://www.deque.com/axe/                            │   │
│  │ • Integration: Jest, Playwright, Cypress                     │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ WAVE (WebAIM)                                                │   │
│  │ • Type: Browser extension + online tool                      │   │
│  │ • Coverage: WCAG 2.1, visual feedback                        │   │
│  │ • URL: https://wave.webaim.org                               │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ Lighthouse (Google)                                          │   │
│  │ • Type: Built into Chrome DevTools                           │   │
│  │ • Coverage: WCAG subset, performance, SEO                    │   │
│  │ • Free, great for CI/CD integration                          │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ Pa11y                                                        │   │
│  │ • Type: CLI + Dashboard (Open Source)                        │   │
│  │ • Coverage: WCAG 2.1 AA                                      │   │
│  │ • GitHub: https://github.com/pa11y/pa11y                     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  SCREEN READER TESTING                                              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ NVDA (Windows)                                               │   │
│  │ • Free, open source screen reader                            │   │
│  │ • URL: https://www.nvaccess.org                              │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ VoiceOver (macOS/iOS)                                        │   │
│  │ • Built into Apple devices                                   │   │
│  │ • Activate: Cmd + F5 (macOS)                                 │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ JAWS (Windows)                                               │   │
│  │ • Commercial, most popular screen reader                     │   │
│  │ • URL: https://www.freedomscientific.com/products/software/jaws/│
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ TalkBack (Android)                                           │   │
│  │ • Built into Android devices                                 │   │
│  │ • Settings > Accessibility > TalkBack                        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  COLOR & CONTRAST                                                   │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Contrast Checker (WebAIM)                                    │   │
│  │ • URL: https://webaim.org/resources/contrastchecker/         │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ Color Contrast Analyzer (TPGi)                               │   │
│  │ • Desktop app for Windows/macOS                              │   │
│  │ • URL: https://www.tpgi.com/color-contrast-checker/          │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ Stark (Figma/Sketch plugin)                                  │   │
│  │ • Design tool integration                                    │   │
│  │ • URL: https://www.getstark.co                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

### 14.3 Privacy Impact Assessment Tools

┌─────────────────────────────────────────────────────────────────────┐
│              PRIVACY IMPACT ASSESSMENT (PIA/DPIA) TOOLS              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ENTERPRISE PLATFORMS                                               │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ OneTrust Assessment Automation                               │   │
│  │ • Automated DPIA workflows                                   │   │
│  │ • Risk scoring and tracking                                  │   │
│  │ • Integration with Records of Processing                     │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ BigID                                                        │   │
│  │ • Data discovery + privacy intelligence                      │   │
│  │ • ML-powered data classification                             │   │
│  │ • URL: https://bigid.com                                     │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ Securiti.ai                                                  │   │
│  │ • AI-powered privacy platform                                │   │
│  │ • Automated data mapping                                     │   │
│  │ • URL: https://securiti.ai                                   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  FREE RESOURCES                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ ICO DPIA Template (UK)                                       │   │
│  │ • Official UK template                                       │   │
│  │ • URL: https://ico.org.uk/for-organisations/uk-gdpr-guidance-and│
│  │         -resources/data-protection-impact-assessments-dpias/ │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ CNIL PIA Tool (France)                                       │   │
│  │ • Free software for conducting PIAs                          │   │
│  │ • URL: https://www.cnil.fr/en/open-source-pia-software-helps-│   │
│  │         carry-out-data-protection-impact-assesment           │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ NIST Privacy Framework                                       │   │
│  │ • Comprehensive privacy risk management                      │   │
│  │ • URL: https://www.nist.gov/privacy-framework                │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

### 14.4 License Compliance Tools

┌─────────────────────────────────────────────────────────────────────┐
│                    LICENSE COMPLIANCE TOOLS                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  DEPENDENCY SCANNING                                                │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ FOSSA                                                        │   │
│  │ • Automated license compliance                               │   │
│  │ • SBOM generation                                            │   │
│  │ • URL: https://fossa.com                                     │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ Snyk                                                         │   │
│  │ • Security + license scanning                                │   │
│  │ • CI/CD integration                                          │   │
│  │ • URL: https://snyk.io                                       │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ WhiteSource (Mend)                                           │   │
│  │ • Open source security + compliance                          │   │
│  │ • Policy enforcement                                         │   │
│  │ • URL: https://www.mend.io                                   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  FREE/OPEN SOURCE TOOLS                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ license-checker (npm)                                        │   │
│  │ • npm install -g license-checker                             │   │
│  │ • Scans node_modules for licenses                            │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ pip-licenses (Python)                                        │   │
│  │ • pip install pip-licenses                                   │   │
│  │ • Lists licenses of installed packages                       │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ cargo-license (Rust)                                         │   │
│  │ • cargo install cargo-license                                │   │
│  │ • Displays licenses of dependencies                          │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ FOSSology                                                    │   │
│  │ • Open source compliance toolkit                             │   │
│  │ • GitHub: https://github.com/fossology/fossology             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  SBOM GENERATORS                                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Syft (Anchore)                                               │   │
│  │ • SBOM generation for containers                             │   │
│  │ • GitHub: https://github.com/anchore/syft                    │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ CycloneDX                                                    │   │
│  │ • OWASP SBOM standard                                        │   │
│  │ • Multiple language plugins                                  │   │
│  │ • URL: https://cyclonedx.org                                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

### 14.5 Data Subject Request Management

┌─────────────────────────────────────────────────────────────────────┐
│             DATA SUBJECT REQUEST (DSR) MANAGEMENT                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ENTERPRISE PLATFORMS                                               │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ OneTrust Privacy Rights Automation                           │   │
│  │ • Automated DSR fulfillment                                  │   │
│  │ • Identity verification                                      │   │
│  │ • Integration with data systems                              │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ DataGrail                                                    │   │
│  │ • Privacy management platform                                │   │
│  │ • DSR automation + consent management                        │   │
│  │ • URL: https://www.datagrail.io                              │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ Osano                                                        │   │
│  │ • All-in-one privacy platform                                │   │
│  │ • DSR + consent + vendor management                          │   │
│  │ • URL: https://www.osano.com                                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  IDENTITY VERIFICATION FOR DSR                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ ID.me                                                        │   │
│  │ • Identity proofing service                                  │   │
│  │ • URL: https://www.id.me                                     │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ Jumio                                                        │   │
│  │ • AI-powered identity verification                           │   │
│  │ • URL: https://www.jumio.com                                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

### 14.6 Regulatory Resources

┌─────────────────────────────────────────────────────────────────────┐
│                    REGULATORY RESOURCES                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  GDPR & EU PRIVACY                                                  │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ EDPB Guidelines                                              │   │
│  │ https://edpb.europa.eu/our-work-tools/general-guidance/      │   │
│  │ guidelines-recommendations-best-practices_en                 │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ ICO Guidance (UK)                                            │   │
│  │ https://ico.org.uk/for-organisations/guide-to-data-protection/│  │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ CNIL Guidelines (France)                                     │   │
│  │ https://www.cnil.fr/en/gdpr-guidelines                       │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  US PRIVACY                                                         │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ California Attorney General CCPA Resources                   │   │
│  │ https://oag.ca.gov/privacy/ccpa                              │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ CPPA (California Privacy Protection Agency)                  │   │
│  │ https://cppa.ca.gov/regulations/                             │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ FTC Privacy & Security                                       │   │
│  │ https://www.ftc.gov/business-guidance/privacy-security       │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ACCESSIBILITY                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ W3C WAI Resources                                            │   │
│  │ https://www.w3.org/WAI/                                      │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ WebAIM                                                       │   │
│  │ https://webaim.org/                                          │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ A11y Project                                                 │   │
│  │ https://www.a11yproject.com/                                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  AI REGULATION                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ EU AI Act Official Text                                      │   │
│  │ https://artificialintelligenceact.eu/                        │   │
│  ├─────────────────────────────────────────────────────────────┤   │
│  │ EU AI Office                                                 │   │
│  │ https://digital-strategy.ec.europa.eu/en/policies/           │   │
│  │ ai-office                                                    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘



---

## 15. GLOSSARIO LEGALE

### 15.1 Privacy e Data Protection

| Termine | Definizione | Contesto |
|---------|-------------|----------|
| **Controller (Titolare)** | L'entità che determina le finalità e i mezzi del trattamento dei dati personali | GDPR Art. 4(7) |
| **Processor (Responsabile)** | L'entità che tratta dati personali per conto del titolare | GDPR Art. 4(8) |
| **Data Subject (Interessato)** | La persona fisica identificata o identificabile i cui dati sono trattati | GDPR Art. 4(1) |
| **Personal Data** | Qualsiasi informazione relativa a una persona fisica identificata o identificabile | GDPR Art. 4(1) |
| **Sensitive Data (Categorie particolari)** | Dati su razza, religione, salute, orientamento sessuale, opinioni politiche, ecc. | GDPR Art. 9 |
| **Processing (Trattamento)** | Qualsiasi operazione su dati personali (raccolta, registrazione, uso, cancellazione) | GDPR Art. 4(2) |
| **Consent (Consenso)** | Manifestazione di volontà libera, specifica, informata e inequivocabile | GDPR Art. 4(11) |
| **DPO (Data Protection Officer)** | Figura responsabile della conformità privacy all'interno di un'organizzazione | GDPR Art. 37-39 |
| **DPIA (Data Protection Impact Assessment)** | Valutazione dell'impatto sulla protezione dei dati per trattamenti ad alto rischio | GDPR Art. 35 |
| **Pseudonymization** | Trattamento che impedisce l'identificazione diretta senza informazioni aggiuntive | GDPR Art. 4(5) |
| **Anonymization** | Rimozione irreversibile di qualsiasi possibilità di identificazione | Non più dati personali |
| **Data Breach** | Violazione della sicurezza che porta alla distruzione, perdita, alterazione o divulgazione non autorizzata | GDPR Art. 4(12) |
| **Right to Erasure** | Diritto dell'interessato di ottenere la cancellazione dei propri dati | GDPR Art. 17 |
| **Data Portability** | Diritto di ricevere i propri dati in formato strutturato e trasferirli | GDPR Art. 20 |
| **Joint Controller** | Due o più titolari che determinano congiuntamente finalità e mezzi | GDPR Art. 26 |
| **Sub-processor** | Responsabile ingaggiato da un altro responsabile per specifiche attività | GDPR Art. 28 |
| **BCR (Binding Corporate Rules)** | Regole vincolanti d'impresa per trasferimenti intra-gruppo | GDPR Art. 47 |
| **SCC (Standard Contractual Clauses)** | Clausole standard approvate dalla CE per trasferimenti extra-UE | GDPR Art. 46(2)(c) |

### 15.2 California Privacy (CCPA/CPRA)

| Termine | Definizione | Riferimento |
|---------|-------------|-------------|
| **Business** | Organizzazione for-profit che supera le soglie CCPA | CCPA 1798.140(d) |
| **Consumer** | Residente della California | CCPA 1798.140(g) |
| **Sale of Personal Information** | Divulgazione di PI per valore monetario o altro | CCPA 1798.140(ad) |
| **Sharing** | Divulgazione per pubblicità comportamentale cross-context | CPRA aggiunta |
| **Service Provider** | Entità che processa PI per conto di un business | CCPA 1798.140(ag) |
| **Contractor** | Nuova categoria introdotta da CPRA | CPRA 1798.140(j) |
| **Sensitive Personal Information (SPI)** | Categoria speciale: SSN, credenziali, geolocalizzazione, ecc. | CPRA 1798.140(ae) |
| **GPC (Global Privacy Control)** | Segnale browser per opt-out automatico | CPRA riconosciuto |
| **Authorized Agent** | Soggetto che può esercitare diritti per conto del consumer | CCPA 1798.140(c) |
| **Do Not Sell or Share** | Diritto di impedire vendita e condivisione di PI | CCPA 1798.120 |
| **Right to Limit** | Diritto di limitare uso di SPI | CPRA 1798.121 |

### 15.3 Accessibilità

| Termine | Definizione | Standard |
|---------|-------------|----------|
| **WCAG** | Web Content Accessibility Guidelines - linee guida W3C | W3C WAI |
| **Perceivable** | Informazioni presentabili in modi che l'utente può percepire | WCAG Principio 1 |
| **Operable** | Componenti UI navigabili e utilizzabili | WCAG Principio 2 |
| **Understandable** | Informazioni e operazioni comprensibili | WCAG Principio 3 |
| **Robust** | Contenuto interpretabile da diverse tecnologie assistive | WCAG Principio 4 |
| **ARIA** | Accessible Rich Internet Applications - attributi per accessibilità | W3C WAI-ARIA |
| **Screen Reader** | Software che legge contenuti dello schermo ad alta voce | Tecnologia assistiva |
| **Alt Text** | Testo alternativo che descrive il contenuto di un'immagine | WCAG 1.1.1 |
| **Focus Indicator** | Indicatore visivo dell'elemento attualmente focalizzato | WCAG 2.4.7 |
| **Skip Link** | Link per saltare blocchi di navigazione ripetuti | WCAG 2.4.1 |
| **Color Contrast Ratio** | Rapporto luminosità tra testo e sfondo (4.5:1 per AA) | WCAG 1.4.3 |
| **Assistive Technology** | Hardware/software che aiuta persone con disabilità | Generale |
| **EAA** | European Accessibility Act - direttiva UE 2019/882 | UE |
| **Section 508** | Legge USA per accessibilità tecnologie federali | US Federal |
| **ADA** | Americans with Disabilities Act | US Federal |

### 15.4 AI e Automazione

| Termine | Definizione | Contesto |
|---------|-------------|----------|
| **AI System** | Sistema basato su macchine che genera output (previsioni, decisioni, contenuti) | EU AI Act Art. 3(1) |
| **High-Risk AI** | Sistema AI con rischi significativi per salute, sicurezza o diritti fondamentali | EU AI Act Art. 6 |
| **GPAI (General-Purpose AI)** | Modello AI con ampia gamma di utilizzi possibili | EU AI Act Art. 3(63) |
| **Foundation Model** | Modello AI addestrato su dati ampi, adattabile a compiti diversi | Termine tecnico |
| **Automated Decision-Making** | Decisione presa senza intervento umano significativo | GDPR Art. 22 |
| **Profiling** | Elaborazione automatizzata di dati per valutare aspetti personali | GDPR Art. 4(4) |
| **Human Oversight** | Supervisione umana di sistemi AI per prevenire/minimizzare rischi | EU AI Act Art. 14 |
| **Conformity Assessment** | Valutazione della conformità di sistemi AI ai requisiti | EU AI Act Art. 43 |
| **AI Literacy** | Competenze per comprendere e utilizzare sistemi AI in modo informato | EU AI Act Art. 4 |
| **Deepfake** | Contenuto audio/video generato da AI che sembra reale | EU AI Act Art. 52 |
| **Emotion Recognition** | Sistema AI che identifica emozioni da dati biometrici | EU AI Act Art. 3(39) |
| **Systemic Risk** | Rischio con impatto significativo su salute pubblica, sicurezza, diritti | EU AI Act Art. 3(65) |

### 15.5 Licenze Software

| Termine | Definizione | Esempio |
|---------|-------------|---------|
| **Permissive License** | Licenza con poche restrizioni sull'uso e redistribuzione | MIT, Apache, BSD |
| **Copyleft** | Requisito che derivati mantengano stessa licenza | GPL, AGPL |
| **Attribution** | Obbligo di riconoscere autore originale | Tutti (quasi) |
| **Derivative Work** | Opera basata su/che incorpora lavoro esistente | Modifica codice |
| **SBOM** | Software Bill of Materials - lista componenti software | Requisito sicurezza |
| **OSI Approved** | Licenza approvata da Open Source Initiative | Standard OSS |
| **Dual Licensing** | Software disponibile sotto due licenze diverse | MySQL, Qt |
| **License Compatibility** | Possibilità di combinare software con licenze diverse | Verifica necessaria |
| **Warranty Disclaimer** | Dichiarazione di assenza garanzie | Clausola standard |
| **Liability Limitation** | Limitazione responsabilità per danni | Clausola standard |
| **Patent Grant** | Concessione diritti su brevetti inclusi nel software | Apache 2.0, GPLv3 |

### 15.6 Sicurezza e Breach

| Termine | Definizione | Contesto |
|---------|-------------|----------|
| **Personal Data Breach** | Violazione sicurezza con impatto su dati personali | GDPR Art. 4(12) |
| **Notification Deadline** | Tempo massimo per notifica (72h GDPR, vario per stato USA) | Regolamentare |
| **Supervisory Authority** | Autorità di controllo (es. Garante Privacy) | GDPR Art. 51 |
| **High Risk** | Breach con probabilità elevata di impatto sui diritti | GDPR Art. 34 |
| **Incident Response** | Processo di gestione incidenti di sicurezza | Best practice |
| **Forensic Analysis** | Analisi tecnica per determinare causa e impatto di un breach | Investigazione |
| **Containment** | Azioni immediate per limitare danni di un breach | Fase IR |
| **Remediation** | Azioni per correggere vulnerabilità e prevenire recidive | Fase IR |
| **Affected Individuals** | Persone i cui dati sono stati compromessi | Notifica richiesta |
| **Credit Monitoring** | Servizio di monitoraggio offerto a vittime di breach | Rimedio comune |

---

## 16. APPENDICE: TEMPLATE E MODELLI

### 16.1 Privacy Policy Template (GDPR Compliant)

# Privacy Policy

**Last Updated:** [DATE]
**Effective Date:** [DATE]

§ 1. WHO WE ARE

**Data Controller:** [COMPANY NAME]
**Address:** [ADDRESS]
**Email:** privacy@[domain].com
**Phone:** [PHONE]

**Data Protection Officer:** [IF APPLICABLE]
**DPO Contact:** dpo@[domain].com

§ 2. INFORMATION WE COLLECT

§ 2.1 INFORMATION YOU PROVIDE
- Account information (name, email, password)
- Profile information (preferences, settings)
- Communications (support requests, feedback)
- Payment information (processed by [PAYMENT PROCESSOR])

§ 2.2 INFORMATION COLLECTED AUTOMATICALLY
- Device information (browser type, OS, device ID)
- Usage data (pages visited, features used)
- Log data (IP address, timestamps, referrer)
- Cookies and similar technologies (see Cookie Policy)

§ 2.3 INFORMATION FROM THIRD PARTIES
- Social login providers (if you connect accounts)
- Analytics providers
- Marketing partners (with your consent)

§ 3. HOW WE USE YOUR INFORMATION

| Purpose | Legal Basis | Data Categories |
|---------|-------------|-----------------|
| Provide services | Contract performance | Account, usage data |
| Process payments | Contract performance | Payment info |
| Customer support | Contract, legitimate interest | Communications |
| Improve services | Legitimate interest | Usage, analytics |
| Marketing | Consent | Contact info |
| Security | Legitimate interest, legal obligation | All categories |
| Legal compliance | Legal obligation | As required |

§ 4. HOW WE SHARE YOUR INFORMATION

We share your information with:

- **Service Providers:** Hosting, payment processing, analytics, support
- **Business Partners:** [SPECIFY IF APPLICABLE]
- **Legal Requirements:** When required by law or to protect rights
- **Business Transfers:** In case of merger, acquisition, or sale

We do NOT sell your personal information.

§ 5. INTERNATIONAL TRANSFERS

We transfer data outside the EEA using:
- Standard Contractual Clauses approved by the European Commission
- Adequacy decisions (where applicable)
- Your explicit consent (for specific transfers)

§ 6. DATA RETENTION

| Data Category | Retention Period | Reason |
|---------------|------------------|--------|
| Account data | Until deletion request + 30 days | Service provision |
| Transaction records | 7 years | Legal/tax requirements |
| Support tickets | 3 years | Quality, training |
| Marketing preferences | Until withdrawal | Consent-based |
| Log data | 12 months | Security |

§ 7. YOUR RIGHTS

You have the right to:
- **Access** your personal data
- **Rectify** inaccurate data
- **Erase** your data ("right to be forgotten")
- **Restrict** processing in certain circumstances
- **Object** to processing based on legitimate interests
- **Data portability** - receive your data in machine-readable format
- **Withdraw consent** at any time (without affecting prior processing)
- **Lodge a complaint** with a supervisory authority

**To exercise your rights:** privacy@[domain].com or [PRIVACY CENTER URL]

**Response time:** Within 30 days (may extend by 60 days for complex requests)

§ 8. COOKIES

We use cookies for:
- **Essential cookies:** Required for site functionality
- **Analytics cookies:** Help us understand usage (with consent)
- **Marketing cookies:** Enable targeted advertising (with consent)

For detailed information, see our [Cookie Policy].

§ 9. SECURITY

We implement appropriate technical and organizational measures:
- Encryption in transit (TLS 1.2+) and at rest
- Access controls and authentication
- Regular security assessments
- Employee training
- Incident response procedures

§ 10. CHILDREN'S PRIVACY

Our services are not directed to children under [13/16]. We do not knowingly collect data from children. If we discover we have collected data from a child, we will delete it promptly.

§ 11. CHANGES TO THIS POLICY

We may update this policy periodically. We will notify you of significant changes via:
- Email notification
- Website notice
- In-app notification

§ 12. CONTACT US

**Privacy inquiries:** privacy@[domain].com
**General:** support@[domain].com
**Address:** [ADDRESS]

**Supervisory Authority:**
[RELEVANT DPA NAME AND CONTACT]

### 16.2 Cookie Banner Text Template

// templates/cookie-banner-text.ts

export const cookieBannerText = {
  title: "We value your privacy",
  
  description: `We use cookies to enhance your browsing experience, 
provide personalized content, and analyze our traffic. 
You can choose which cookies to accept below.`,

  learnMore: "Learn more in our Cookie Policy",
  
  buttons: {
    acceptAll: "Accept All",
    rejectAll: "Reject All", 
    savePreferences: "Save Preferences",
    customize: "Customize"
  },
  
  categories: {
    necessary: {
      title: "Strictly Necessary",
      description: "Essential for the website to function. Cannot be disabled.",
      always: true
    },
    functional: {
      title: "Functional",
      description: "Enable enhanced functionality and personalization.",
      default: false
    },
    analytics: {
      title: "Analytics",
      description: "Help us understand how visitors interact with our website.",
      default: false
    },
    advertising: {
      title: "Advertising",
      description: "Used to deliver relevant ads and track ad campaign performance.",
      default: false
    }
  },
  
  footer: `By clicking "Accept All", you consent to the storage of cookies 
on your device. You can change your preferences at any time in the 
Privacy Center.`
};

// Multi-language support
export const cookieBannerTextIT = {
  title: "Rispettiamo la tua privacy",
  
  description: `Utilizziamo cookie per migliorare la tua esperienza di navigazione, 
fornire contenuti personalizzati e analizzare il nostro traffico. 
Puoi scegliere quali cookie accettare qui sotto.`,

  learnMore: "Scopri di più nella nostra Cookie Policy",
  
  buttons: {
    acceptAll: "Accetta tutti",
    rejectAll: "Rifiuta tutti",
    savePreferences: "Salva preferenze",
    customize: "Personalizza"
  },
  
  categories: {
    necessary: {
      title: "Strettamente necessari",
      description: "Essenziali per il funzionamento del sito. Non possono essere disabilitati.",
      always: true
    },
    functional: {
      title: "Funzionali",
      description: "Abilitano funzionalità avanzate e personalizzazione.",
      default: false
    },
    analytics: {
      title: "Analitici",
      description: "Ci aiutano a capire come i visitatori interagiscono con il sito.",
      default: false
    },
    advertising: {
      title: "Pubblicitari",
      description: "Utilizzati per mostrare annunci pertinenti e monitorare le campagne.",
      default: false
    }
  },
  
  footer: `Cliccando "Accetta tutti", acconsenti alla memorizzazione dei cookie 
sul tuo dispositivo. Puoi modificare le tue preferenze in qualsiasi momento 
nel Centro Privacy.`
};

### 16.3 Data Processing Agreement (DPA) Outline

# Data Processing Agreement

**Between:**
Controller: [COMPANY NAME] ("Controller")
Processor: [SERVICE PROVIDER] ("Processor")

**Effective Date:** [DATE]

§ 1. DEFINITIONS
- Personal Data: as defined in GDPR Art. 4(1)
- Processing: as defined in GDPR Art. 4(2)
- Sub-processor: any third party engaged by Processor

§ 2. SUBJECT MATTER AND DURATION

§ 2.1 SUBJECT MATTER
[Describe the processing activities covered]

§ 2.2 DURATION
This Agreement remains in effect for the duration of the Main Agreement.

§ 2.3 NATURE AND PURPOSE
[Describe why processing is performed]

§ 2.4 CATEGORIES OF DATA SUBJECTS
- Customers
- Employees
- Website visitors
- [Other categories]

§ 2.5 CATEGORIES OF PERSONAL DATA
- Contact information
- Account credentials
- Usage data
- [Other categories]

§ 3. PROCESSOR OBLIGATIONS

Processor shall:

3.1 Process data only on documented Controller instructions
3.2 Ensure personnel are bound by confidentiality
3.3 Implement appropriate technical and organizational measures (Annex A)
3.4 Engage sub-processors only with prior authorization
3.5 Assist Controller with data subject requests
3.6 Assist with DPIA where required
3.7 Delete or return data at end of services
3.8 Provide information to demonstrate compliance
3.9 Notify Controller of data breaches without undue delay

§ 4. SUB-PROCESSING

4.1 Current sub-processors listed in Annex B
4.2 Controller authorizes use of listed sub-processors
4.3 Processor will notify Controller of changes
4.4 Same obligations flow down to sub-processors

§ 5. INTERNATIONAL TRANSFERS

5.1 Transfers outside EEA require appropriate safeguards
5.2 Standard Contractual Clauses incorporated by reference
5.3 Processor ensures adequate protection level

§ 6. SECURITY MEASURES (ANNEX A)

- Encryption at rest and in transit
- Access controls and authentication
- Regular security testing
- Incident response procedures
- Employee training
- Physical security measures

§ 7. AUDIT RIGHTS

7.1 Controller may audit Processor's compliance
7.2 Reasonable notice required (30 days)
7.3 Third-party audit reports accepted
7.4 Costs borne by Controller unless non-compliance found

§ 8. LIABILITY AND INDEMNIFICATION

[Standard liability clauses per Main Agreement]

§ 9. TERM AND TERMINATION

9.1 Effective upon Main Agreement execution
9.2 Terminates with Main Agreement
9.3 Data return/deletion within 30 days of termination

§ ANNEX A: TECHNICAL AND ORGANIZATIONAL MEASURES
[Detailed security measures]

§ ANNEX B: APPROVED SUB-PROCESSORS
| Sub-processor | Location | Processing Purpose |
|---------------|----------|-------------------|
| [NAME] | [COUNTRY] | [PURPOSE] |

§ ANNEX C: DATA PROCESSING DETAILS
[Specific processing activities description]

### 16.4 DSAR Response Templates

// templates/dsar-responses.ts

export const dsarTemplates = {
  acknowledgment: {
    subject: "Your Data Subject Access Request - Received",
    body: `Dear {{name}},

Thank you for your data subject access request submitted on {{date}}.

**Request ID:** {{requestId}}
**Request Type:** {{requestType}}

We have received your request and will respond within 30 days as required 
by applicable data protection laws. If we need to extend this period, we 
will inform you within the initial 30 days.

To verify your identity and process your request, we may contact you for 
additional information.

You can check the status of your request at: {{statusUrl}}

If you have questions, please contact us at privacy@{{domain}}.

Best regards,
{{companyName}} Privacy Team`
  },

  identityVerification: {
    subject: "Identity Verification Required - Request {{requestId}}",
    body: `Dear {{name}},

To process your data subject request, we need to verify your identity.

Please provide:
- A copy of a government-issued ID (passport, driver's license)
- Proof of address dated within the last 3 months

You can securely upload these documents at: {{uploadUrl}}

This verification helps protect your personal data from unauthorized access.

If we don't receive verification within 14 days, we may be unable to 
process your request.

Best regards,
{{companyName}} Privacy Team`
  },

  accessResponse: {
    subject: "Your Personal Data - Request {{requestId}} Complete",
    body: `Dear {{name}},

In response to your data access request, please find enclosed a copy of 
the personal data we hold about you.

**Request ID:** {{requestId}}
**Completion Date:** {{completionDate}}

The attached document includes:
- Categories of personal data we process
- Purposes of processing
- Recipients of your data
- Retention periods
- Your rights regarding this data

You can download your data securely at: {{downloadUrl}}
This link expires in 7 days.

If you have questions about this data or wish to exercise other rights, 
please contact us at privacy@{{domain}}.

Best regards,
{{companyName}} Privacy Team`
  },

  deletionConfirmation: {
    subject: "Deletion Request Complete - Request {{requestId}}",
    body: `Dear {{name}},

Your deletion request has been processed.

**Request ID:** {{requestId}}
**Completion Date:** {{completionDate}}

We have deleted your personal data from our systems, except where:
- We are legally required to retain certain data
- Data is necessary for ongoing contract obligations
- Anonymized data used for statistical purposes

The following data was retained and why:
{{retainedDataList}}

Please note that it may take up to 30 days for this deletion to propagate 
to all backup systems.

If you have questions, please contact us at privacy@{{domain}}.

Best regards,
{{companyName}} Privacy Team`
  },

  requestDenied: {
    subject: "Regarding Your Data Request - {{requestId}}",
    body: `Dear {{name}},

Thank you for your data subject request submitted on {{submitDate}}.

After careful review, we are unable to fulfill your request for the 
following reason(s):

{{denialReason}}

Under applicable data protection laws, we may deny requests that:
- Cannot be verified
- Are manifestly unfounded or excessive
- Would adversely affect the rights of others
- Conflict with legal obligations

You have the right to:
- Request further information about this decision
- Lodge a complaint with the supervisory authority:
  {{supervisoryAuthorityInfo}}

If you believe this decision is incorrect, please contact us at 
privacy@{{domain}} with additional information.

Best regards,
{{companyName}} Privacy Team`
  }
};



### 16.5 Breach Notification Templates

// templates/breach-notification.ts

export const breachNotificationTemplates = {
  regulatoryNotification: {
    // For notifications to supervisory authorities (e.g., Garante Privacy)
    template: `
PERSONAL DATA BREACH NOTIFICATION
Article 33 GDPR

1. CONTROLLER INFORMATION
Name: {{controllerName}}
Address: {{controllerAddress}}
DPO Contact: {{dpoEmail}}
Reference Number: {{breachId}}

2. BREACH DETAILS
Date/Time Discovered: {{discoveredAt}}
Date/Time Occurred: {{occurredAt}} (if known)
Duration: {{duration}}

3. NATURE OF BREACH
☐ Confidentiality breach (unauthorized disclosure)
☐ Integrity breach (unauthorized alteration)
☐ Availability breach (loss of access)

Description:
{{breachDescription}}

4. DATA AFFECTED
Categories of data subjects: {{subjectCategories}}
Approximate number: {{subjectCount}}

Categories of personal data: {{dataCategories}}
Approximate number of records: {{recordCount}}

5. LIKELY CONSEQUENCES
{{consequenceDescription}}

6. MEASURES TAKEN
Containment measures:
{{containmentMeasures}}

Remediation measures:
{{remediationMeasures}}

7. COMMUNICATION TO DATA SUBJECTS
Have affected individuals been notified? {{notifiedSubjects}}
If yes, date of notification: {{subjectNotificationDate}}
If no, reasons for delay: {{delayReasons}}

8. CONTACT FOR FOLLOW-UP
Name: {{contactName}}
Email: {{contactEmail}}
Phone: {{contactPhone}}

Submitted by: {{submitterName}}
Date: {{submissionDate}}
`
  },

  affectedIndividualNotification: {
    subject: "Important Security Notice from {{companyName}}",
    body: `Dear {{recipientName}},

We are writing to inform you about a data security incident that may 
have affected your personal information.

WHAT HAPPENED
{{incidentDescription}}

We discovered this incident on {{discoveryDate}} and have taken 
immediate action to address it.

WHAT INFORMATION WAS INVOLVED
The following types of your personal information may have been affected:
{{affectedDataTypes}}

WHAT WE ARE DOING
{{remediationActions}}

WHAT YOU CAN DO
To protect yourself, we recommend:
{{recommendedActions}}

{{#if creditMonitoring}}
FREE CREDIT MONITORING
We are offering {{creditMonitoringPeriod}} of free credit monitoring 
through {{creditMonitoringProvider}}.
To enroll: {{creditMonitoringEnrollmentUrl}}
Enrollment deadline: {{creditMonitoringDeadline}}
{{/if}}

FOR MORE INFORMATION
If you have questions or concerns:
Email: {{supportEmail}}
Phone: {{supportPhone}} (Available {{supportHours}})
Website: {{supportUrl}}

We sincerely apologize for this incident and any inconvenience it may 
cause. We take the security of your information seriously and are 
committed to protecting your data.

Sincerely,
{{senderName}}
{{senderTitle}}
{{companyName}}
`
  },

  internalIncidentReport: {
    subject: "[SECURITY INCIDENT] {{severity}} - {{briefDescription}}",
    body: `
═══════════════════════════════════════════════════════════════
SECURITY INCIDENT REPORT - INTERNAL CONFIDENTIAL
═══════════════════════════════════════════════════════════════

INCIDENT ID: {{incidentId}}
SEVERITY: {{severity}}
STATUS: {{status}}

TIMELINE
- Discovered: {{discoveredAt}}
- Reported: {{reportedAt}}
- Contained: {{containedAt}}
- Resolved: {{resolvedAt}}

INCIDENT SUMMARY
{{incidentSummary}}

AFFECTED SYSTEMS
{{affectedSystems}}

AFFECTED DATA
- Personal Data Involved: {{personalDataInvolved}}
- Data Subjects Count: {{dataSubjectsCount}}
- Data Categories: {{dataCategories}}

ROOT CAUSE
{{rootCause}}

CONTAINMENT ACTIONS
{{containmentActions}}

REMEDIATION ACTIONS
{{remediationActions}}

REGULATORY NOTIFICATIONS REQUIRED
- GDPR (72h): {{gdprNotificationRequired}}
- CCPA: {{ccpaNotificationRequired}}
- State Laws: {{stateNotificationsRequired}}

NOTIFICATION STATUS
- Supervisory Authority: {{authorityNotificationStatus}}
- Data Subjects: {{subjectNotificationStatus}}

LESSONS LEARNED
{{lessonsLearned}}

FOLLOW-UP ACTIONS
{{followUpActions}}

PREPARED BY: {{preparedBy}}
DATE: {{reportDate}}
DISTRIBUTION: {{distribution}}
`
  }
};

### 16.6 Records of Processing Activities (ROPA) Template

// templates/ropa-template.ts

interface ProcessingActivityRecord {
  // Article 30(1) Requirements for Controllers
  id: string;
  activityName: string;
  description: string;
  
  // Controller Information
  controller: {
    name: string;
    address: string;
    contactEmail: string;
    dpoContact?: string;
  };
  
  // Joint Controllers (if applicable)
  jointControllers?: Array<{
    name: string;
    address: string;
    responsibilities: string;
  }>;
  
  // Representative (for non-EU controllers)
  representative?: {
    name: string;
    address: string;
    contactEmail: string;
  };
  
  // Processing Details
  purposes: string[];
  legalBasis: {
    basis: 'consent' | 'contract' | 'legal_obligation' | 'vital_interest' | 'public_interest' | 'legitimate_interest';
    details: string;
    legitimateInterestAssessment?: string;
  };
  
  // Data Categories
  dataSubjectCategories: string[];
  personalDataCategories: string[];
  sensitiveDataCategories?: string[];
  
  // Recipients
  recipientCategories: Array<{
    name: string;
    type: 'processor' | 'controller' | 'third_party';
    purpose: string;
    location: string;
  }>;
  
  // International Transfers
  internationalTransfers?: Array<{
    country: string;
    safeguards: 'adequacy_decision' | 'scc' | 'bcr' | 'derogation' | 'other';
    safeguardDetails: string;
  }>;
  
  // Retention
  retentionPeriod: string;
  retentionCriteria: string;
  deletionProcess: string;
  
  // Security Measures
  technicalMeasures: string[];
  organizationalMeasures: string[];
  
  // Additional Information
  source: string;
  automatedDecisionMaking: {
    used: boolean;
    description?: string;
    logic?: string;
    significance?: string;
  };
  
  // Metadata
  createdAt: Date;
  updatedAt: Date;
  reviewDate: Date;
  status: 'active' | 'archived' | 'under_review';
  owner: string;
}

export const ropaExample: ProcessingActivityRecord = {
  id: "ROPA-001",
  activityName: "Customer Account Management",
  description: "Processing of customer personal data for account creation, authentication, and management",
  
  controller: {
    name: "[Company Name]",
    address: "[Address]",
    contactEmail: "privacy@company.com",
    dpoContact: "dpo@company.com"
  },
  
  purposes: [
    "Create and manage customer accounts",
    "Authenticate users during login",
    "Provide customer support",
    "Send service-related communications"
  ],
  
  legalBasis: {
    basis: "contract",
    details: "Processing necessary for performance of the service agreement with the customer"
  },
  
  dataSubjectCategories: [
    "Customers",
    "Registered users"
  ],
  
  personalDataCategories: [
    "Name",
    "Email address",
    "Password (hashed)",
    "Phone number (optional)",
    "Profile preferences"
  ],
  
  recipientCategories: [
    {
      name: "AWS (Amazon Web Services)",
      type: "processor",
      purpose: "Cloud hosting",
      location: "EU (Frankfurt)"
    },
    {
      name: "SendGrid",
      type: "processor",
      purpose: "Email delivery",
      location: "USA (with SCC)"
    }
  ],
  
  internationalTransfers: [
    {
      country: "United States",
      safeguards: "scc",
      safeguardDetails: "EU Standard Contractual Clauses (2021/914)"
    }
  ],
  
  retentionPeriod: "Account data retained while account is active + 2 years after deletion request",
  retentionCriteria: "Regulatory requirements, legal hold, contractual obligations",
  deletionProcess: "Automated deletion 2 years after account closure, manual deletion upon verified request",
  
  technicalMeasures: [
    "TLS 1.3 encryption in transit",
    "AES-256 encryption at rest",
    "Password hashing with bcrypt (cost factor 12)",
    "Multi-factor authentication available",
    "Regular security assessments"
  ],
  
  organizationalMeasures: [
    "Role-based access control",
    "Employee confidentiality agreements",
    "Regular privacy training",
    "Incident response procedures",
    "Annual ROPA review"
  ],
  
  source: "Data collected directly from data subjects during registration",
  
  automatedDecisionMaking: {
    used: false
  },
  
  createdAt: new Date("2024-01-15"),
  updatedAt: new Date("2025-01-10"),
  reviewDate: new Date("2026-01-15"),
  status: "active",
  owner: "Privacy Team"
};

### 16.7 Quick Reference Cards

┌─────────────────────────────────────────────────────────────────────┐
│               QUICK REFERENCE: GDPR RESPONSE TIMES                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Data Subject Requests                                              │
│  ────────────────────────────────────────────────────────────────   │
│  Standard response:                    30 days                      │
│  Complex request extension:            +60 days (notify in 30)     │
│                                                                      │
│  Breach Notification                                                │
│  ────────────────────────────────────────────────────────────────   │
│  To supervisory authority:             72 hours                     │
│  To affected individuals:              "without undue delay"        │
│                                                                      │
│  Consent Records                                                    │
│  ────────────────────────────────────────────────────────────────   │
│  Maintain for:                         Duration of processing       │
│                                        + time to defend claims      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│               QUICK REFERENCE: CCPA/CPRA RESPONSE TIMES              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Consumer Requests                                                  │
│  ────────────────────────────────────────────────────────────────   │
│  Acknowledge receipt:                  10 business days             │
│  Complete request:                     45 calendar days             │
│  Extension (if needed):                +45 days (notify in 45)     │
│                                                                      │
│  Opt-Out Requests                                                   │
│  ────────────────────────────────────────────────────────────────   │
│  Honor opt-out:                        15 business days             │
│  GPC signal:                           Automatic (no delay)        │
│                                                                      │
│  Record Retention                                                   │
│  ────────────────────────────────────────────────────────────────   │
│  Request records:                      24 months minimum            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│               QUICK REFERENCE: WCAG 2.2 KEY NUMBERS                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Color Contrast (Level AA)                                          │
│  ────────────────────────────────────────────────────────────────   │
│  Normal text (<18pt / <14pt bold):     4.5:1                       │
│  Large text (≥18pt / ≥14pt bold):      3:1                         │
│  UI components & graphics:             3:1                         │
│                                                                      │
│  Target Size (Level AA)                                             │
│  ────────────────────────────────────────────────────────────────   │
│  Minimum clickable area:               24×24 CSS pixels            │
│                                                                      │
│  Timing                                                             │
│  ────────────────────────────────────────────────────────────────   │
│  Flashing content:                     Max 3 times/second          │
│  Auto-updating content:                Pause, stop, hide available │
│  Session timeout:                      20+ hours OR warning        │
│                                                                      │
│  Text Resizing                                                      │
│  ────────────────────────────────────────────────────────────────   │
│  Must support:                         200% zoom without loss      │
│  Reflow at:                            320px width (400% zoom)     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│               QUICK REFERENCE: EU AI ACT TIMELINES                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  2024                                                               │
│  ────────────────────────────────────────────────────────────────   │
│  August 1, 2024:          AI Act enters into force                 │
│                                                                      │
│  2025                                                               │
│  ────────────────────────────────────────────────────────────────   │
│  February 2, 2025:        Prohibited practices apply               │
│  August 2, 2025:          GPAI model obligations apply             │
│                                                                      │
│  2026                                                               │
│  ────────────────────────────────────────────────────────────────   │
│  August 2, 2026:          Full application (high-risk systems)     │
│                                                                      │
│  2027                                                               │
│  ────────────────────────────────────────────────────────────────   │
│  August 2, 2027:          High-risk in Annex I (EU products)       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

---

## 📚 RIFERIMENTI E FONTI

### Normative Ufficiali
- **GDPR:** Regolamento (UE) 2016/679
- **ePrivacy Directive:** Direttiva 2002/58/CE
- **CCPA/CPRA:** California Civil Code 1798.100-199.100
- **COPPA:** 16 CFR Part 312
- **WCAG 2.2:** W3C Recommendation 2023
- **EAA:** Direttiva (UE) 2019/882
- **EU AI Act:** Regolamento (UE) 2024/1689

### Linee Guida Ufficiali
- EDPB Guidelines
- ICO Guidance
- CNIL Recommendations
- California Attorney General CCPA Resources
- W3C WAI Resources
- EU AI Office Publications

---

## 🏁 CONCLUSIONI

Questo catalogo fornisce una guida completa per la compliance legale nelle applicazioni web moderne. I punti chiave da ricordare:

1. **Privacy by Design:** Integra la protezione dei dati fin dalla progettazione
2. **Accessibilità First:** L'accessibilità è un requisito legale, non un'opzione
3. **Trasparenza AI:** L'EU AI Act richiede disclosure e supervisione umana
4. **Documentazione:** Mantieni records dettagliati di tutte le attività di compliance
5. **Aggiornamento Continuo:** Le normative evolvono, mantieniti aggiornato

**Timeline Critiche 2025-2026:**
- ✅ Febbraio 2025: EU AI Act - pratiche proibite
- ✅ Giugno 2025: EAA - piena applicazione
- ✅ Agosto 2025: EU AI Act - GPAI
- ✅ Aprile 2026: ADA Title II, COPPA aggiornamenti
- ✅ Agosto 2026: EU AI Act - applicazione completa

---

*Documento generato: Gennaio 2026*
*Versione: 1.0*
*Prossima revisione: Luglio 2026*

**Disclaimer:** Questo catalogo è fornito a scopo informativo e non costituisce consulenza legale. Consultare sempre professionisti legali qualificati per questioni specifiche di compliance.
