# CATALOGO-BACKGROUND-JOBS

Espansione CATALOGO-BACKGROUND-JOBS per Next.js 14 + TypeScript
§ BULLMQ COMPLETE SETUP
Redis connection con ioredis
typescript
// lib/redis/connection.ts
import Redis from 'ioredis';
import { RedisOptions } from 'ioredis';

class RedisClient {
  private static instance: Redis;
  private static subscriberInstance: Redis;

  static getInstance(): Redis {
    if (!RedisClient.instance) {
      const options: RedisOptions = {
        host: process.env.REDIS_HOST || 'localhost',
        port: parseInt(process.env.REDIS_PORT || '6379'),
        password: process.env.REDIS_PASSWORD,
        retryStrategy: (times) => {
          const delay = Math.min(times * 50, 2000);
          return delay;
        },
        maxRetriesPerRequest: 3,
        enableReadyCheck: true,
        lazyConnect: true,
      };

      RedisClient.instance = new Redis(options);

      RedisClient.instance.on('error', (error) => {
        console.error('Redis connection error:', error);
      });

      RedisClient.instance.on('connect', () => {
        console.log('Redis connected successfully');
      });
    }

    return RedisClient.instance;
  }

  static getSubscriberInstance(): Redis {
    if (!RedisClient.subscriberInstance) {
      RedisClient.subscriberInstance = RedisClient.getInstance().duplicate();
    }
    return RedisClient.subscriberInstance;
  }

  static async disconnect(): Promise<void> {
    if (RedisClient.instance) {
      await RedisClient.instance.quit();
    }
    if (RedisClient.subscriberInstance) {
      await RedisClient.subscriberInstance.quit();
    }
  }

  static async healthCheck(): Promise<boolean> {
    try {
      await RedisClient.getInstance().ping();
      return true;
    } catch {
      return false;
    }
  }
}

export default RedisClient;
Queue, Worker, QueueScheduler setup
typescript
// lib/queues/base.queue.ts
import { Queue, Worker, QueueScheduler, QueueEvents, Job } from 'bullmq';
import RedisClient from '../redis/connection';

export interface QueueConfig {
  name: string;
  defaultJobOptions?: {
    attempts?: number;
    backoff?: {
      type: 'exponential' | 'fixed';
      delay: number;
    };
    delay?: number;
    removeOnComplete?: boolean | number;
    removeOnFail?: boolean | number;
    priority?: number;
  };
  limiter?: {
    max: number;
    duration: number;
  };
  concurrency?: number;
}

export abstract class BaseQueue<T = any> {
  protected queue: Queue<T>;
  protected worker: Worker<T>;
  protected scheduler: QueueScheduler;
  protected queueEvents: QueueEvents;
  protected config: QueueConfig;

  constructor(config: QueueConfig) {
    this.config = config;

    this.queue = new Queue(config.name, {
      connection: RedisClient.getInstance(),
      defaultJobOptions: {
        attempts: 3,
        backoff: {
          type: 'exponential',
          delay: 1000,
        },
        removeOnComplete: 100,
        removeOnFail: 1000,
        ...config.defaultJobOptions,
      },
      limiter: config.limiter,
    });

    this.scheduler = new QueueScheduler(config.name, {
      connection: RedisClient.getInstance(),
    });

    this.queueEvents = new QueueEvents(config.name, {
      connection: RedisClient.getInstance(),
    });

    this.setupEventListeners();
  }

  protected setupEventListeners(): void {
    this.queueEvents.on('completed', ({ jobId }) => {
      console.log(`Job ${jobId} completed`);
    });

    this.queueEvents.on('failed', ({ jobId, failedReason }) => {
      console.error(`Job ${jobId} failed:`, failedReason);
    });

    this.queueEvents.on('stalled', ({ jobId }) => {
      console.warn(`Job ${jobId} stalled`);
    });
  }

  abstract process(): void;

  async addJob(data: T, options?: {
    jobId?: string;
    delay?: number;
    priority?: number;
    attempts?: number;
  }): Promise<Job<T>> {
    return this.queue.add(this.config.name, data, {
      jobId: options?.jobId,
      delay: options?.delay,
      priority: options?.priority,
      attempts: options?.attempts,
    });
  }

  async getJobCounts(): Promise<{
    waiting: number;
    active: number;
    completed: number;
    failed: number;
    delayed: number;
  }> {
    return this.queue.getJobCounts();
  }

  async clean(grace: number = 5000, limit: number = 100): Promise<void> {
    await this.queue.clean(grace, limit, 'completed');
    await this.queue.clean(grace, limit, 'failed');
  }

  async close(): Promise<void> {
    await this.worker.close();
    await this.queue.close();
    await this.scheduler.close();
    await this.queueEvents.close();
  }
}
typescript
// lib/queues/worker.factory.ts
import { Worker } from 'bullmq';
import RedisClient from '../redis/connection';

export interface WorkerOptions {
  concurrency?: number;
  limiter?: {
    max: number;
    duration: number;
  };
  drainDelay?: number;
  lockDuration?: number;
}

export function createWorker<T>(
  queueName: string,
  processor: (job: any) => Promise<any>,
  options: WorkerOptions = {}
): Worker<T> {
  return new Worker<T>(
    queueName,
    async (job) => {
      try {
        return await processor(job);
      } catch (error) {
        console.error(`Error processing job ${job.id}:`, error);
        throw error;
      }
    },
    {
      connection: RedisClient.getSubscriberInstance(),
      concurrency: options.concurrency || 5,
      limiter: options.limiter,
      drainDelay: options.drainDelay || 5,
      lockDuration: options.lockDuration || 30000,
    }
  );
}
Job options complete
typescript
// lib/queues/job.types.ts
export interface JobOptions {
  // Identificazione
  jobId?: string;
  // Ritentativi
  attempts?: number;
  backoff?: 
    | number
    | {
        type: 'exponential' | 'fixed' | 'custom';
        delay: number;
      };
  // Scheduling
  delay?: number;
  repeat?: {
    every?: number;
    cron?: string;
    limit?: number;
    immediately?: boolean;
  };
  // Priorità (1 = più alta, 10 = più bassa)
  priority?: number;
  // Lifespan
  lifo?: boolean;
  timeout?: number;
  // Rimozione automatica
  removeOnComplete?: boolean | number;
  removeOnFail?: boolean | number;
  // Stack trace
  stackTraceLimit?: number;
  // Rate limiting
  rateLimiterKey?: string;
}

export interface JobResult<T = any> {
  success: boolean;
  data?: T;
  error?: string;
  retryCount?: number;
  duration?: number;
}

export interface FailedJobInfo {
  jobId: string;
  failedReason: string;
  stacktrace: string[];
  timestamp: number;
  attemptsMade: number;
}
Concurrency e rate limiting
typescript
// lib/queues/concurrency.manager.ts
export class ConcurrencyManager {
  private static instances: Map<string, number> = new Map();
  private static limits: Map<string, number> = new Map();
  private static rateLimits: Map<string, { count: number; reset: number }> = new Map();

  static setConcurrencyLimit(queueName: string, limit: number): void {
    this.limits.set(queueName, limit);
  }

  static canProcess(queueName: string): boolean {
    const limit = this.limits.get(queueName) || 5;
    const current = this.instances.get(queueName) || 0;
    return current < limit;
  }

  static startJob(queueName: string): void {
    const current = this.instances.get(queueName) || 0;
    this.instances.set(queueName, current + 1);
  }

  static endJob(queueName: string): void {
    const current = this.instances.get(queueName) || 0;
    if (current > 0) {
      this.instances.set(queueName, current - 1);
    }
  }

  static async rateLimit(
    key: string,
    maxRequests: number,
    windowMs: number
  ): Promise<{ allowed: boolean; remaining: number; reset: number }> {
    const now = Date.now();
    const rateLimit = this.rateLimits.get(key);

    if (!rateLimit || now > rateLimit.reset) {
      this.rateLimits.set(key, {
        count: 1,
        reset: now + windowMs,
      });
      return { allowed: true, remaining: maxRequests - 1, reset: now + windowMs };
    }

    if (rateLimit.count >= maxRequests) {
      return { allowed: false, remaining: 0, reset: rateLimit.reset };
    }

    rateLimit.count++;
    return {
      allowed: true,
      remaining: maxRequests - rateLimit.count,
      reset: rateLimit.reset,
    };
  }
}
Dashboard Bull Board integrato
typescript
// app/api/admin/queues/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { createBullBoard } from '@bull-board/api';
import { BullMQAdapter } from '@bull-board/api/bullMQAdapter';
import { ExpressAdapter } from '@bull-board/express';
import { Queue } from 'bullmq';
import RedisClient from '@/lib/redis/connection';

export const dynamic = 'force-dynamic';
export const revalidate = 0;

const basePath = '/api/admin/queues';

const queues = [
  new Queue('email', { connection: RedisClient.getInstance() }),
  new Queue('image-processing', { connection: RedisClient.getInstance() }),
  new Queue('report-generation', { connection: RedisClient.getInstance() }),
  new Queue('data-sync', { connection: RedisClient.getInstance() }),
  new Queue('cleanup', { connection: RedisClient.getInstance() }),
];

const serverAdapter = new ExpressAdapter();
serverAdapter.setBasePath(basePath);

createBullBoard({
  queues: queues.map(queue => new BullMQAdapter(queue)),
  serverAdapter: serverAdapter,
});

export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    const action = searchParams.get('action');
    
    if (action === 'stats') {
      const stats = await Promise.all(
        queues.map(async (queue) => {
          const counts = await queue.getJobCounts();
          return {
            name: queue.name,
            ...counts,
          };
        })
      );
      
      return NextResponse.json({ queues: stats });
    }

    // Simulazione della risposta Bull Board
    return NextResponse.json({
      message: 'Bull Board dashboard available at /api/admin/queues/ui',
      endpoints: [
        `${basePath}/ui`,
        `${basePath}/api/queues`,
        `${basePath}/api/queues/:queueName/jobs`,
      ],
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to fetch queue data' },
      { status: 500 }
    );
  }
}

// Middleware per proteggere l'accesso
export function middleware(request: NextRequest) {
  const authHeader = request.headers.get('authorization');
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return new NextResponse('Unauthorized', { status: 401 });
  }
  
  const token = authHeader.substring(7);
  const adminToken = process.env.QUEUE_ADMIN_TOKEN;
  
  if (token !== adminToken) {
    return new NextResponse('Forbidden', { status: 403 });
  }
  
  return NextResponse.next();
}
typescript
// app/api/admin/queues/ui/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  // In un'implementazione reale, servire l'UI di Bull Board
  return NextResponse.json({
    note: 'Bull Board UI would be served here',
    setup: 'Import and configure @bull-board/express in your API route',
    documentation: 'https://github.com/felixmosh/bull-board',
  });
}
§ JOB PATTERNS COMPLETI
Email queue
typescript
// lib/queues/email.queue.ts
import { BaseQueue } from './base.queue';
import { Job } from 'bullmq';
import nodemailer from 'nodemailer';

export interface EmailJobData {
  to: string | string[];
  subject: string;
  template: 'welcome' | 'password-reset' | 'notification' | 'custom';
  data: Record<string, any>;
  attachments?: Array<{
    filename: string;
    content: Buffer | string;
    contentType?: string;
  }>;
}

export class EmailQueue extends BaseQueue<EmailJobData> {
  private transporter: nodemailer.Transporter;

  constructor() {
    super({
      name: 'email',
      defaultJobOptions: {
        attempts: 5,
        backoff: {
          type: 'exponential',
          delay: 2000,
        },
        removeOnComplete: 500,
        removeOnFail: 1000,
      },
      limiter: {
        max: 100,
        duration: 60000, // 100 email/minuto
      },
    });

    this.transporter = nodemailer.createTransport({
      host: process.env.SMTP_HOST,
      port: parseInt(process.env.SMTP_PORT || '587'),
      secure: process.env.SMTP_SECURE === 'true',
      auth: {
        user: process.env.SMTP_USER,
        pass: process.env.SMTP_PASS,
      },
    });

    this.process();
  }

  async process(): Promise<void> {
    const worker = createWorker<EmailJobData>(
      this.config.name,
      async (job: Job<EmailJobData>) => {
        return await this.processEmail(job);
      },
      {
        concurrency: 10,
        limiter: {
          max: 50,
          duration: 60000,
        },
      }
    );

    worker.on('completed', (job) => {
      console.log(`Email sent successfully to ${job.data.to}`);
    });

    worker.on('failed', (job, err) => {
      console.error(`Failed to send email to ${job?.data.to}:`, err);
    });
  }

  private async processEmail(job: Job<EmailJobData>): Promise<void> {
    const { to, subject, template, data, attachments } = job.data;
    
    let html = '';
    switch (template) {
      case 'welcome':
        html = this.renderWelcomeEmail(data);
        break;
      case 'password-reset':
        html = this.renderPasswordResetEmail(data);
        break;
      case 'notification':
        html = this.renderNotificationEmail(data);
        break;
      default:
        html = data.html || '';
    }

    const mailOptions = {
      from: process.env.EMAIL_FROM || 'noreply@example.com',
      to: Array.isArray(to) ? to.join(', ') : to,
      subject,
      html,
      attachments,
    };

    await this.transporter.sendMail(mailOptions);
  }

  private renderWelcomeEmail(data: any): string {
    return `
      <!DOCTYPE html>
      <html>
        <body>
          <h1>Welcome, ${data.name}!</h1>
          <p>Thank you for joining our platform.</p>
          <a href="${data.verifyLink}">Verify your email</a>
        </body>
      </html>
    `;
  }

  private renderPasswordResetEmail(data: any): string {
    return `
      <!DOCTYPE html>
      <html>
        <body>
          <h1>Password Reset</h1>
          <p>Click the link below to reset your password:</p>
          <a href="${data.resetLink}">Reset Password</a>
          <p>This link will expire in 1 hour.</p>
        </body>
      </html>
    `;
  }

  private renderNotificationEmail(data: any): string {
    return `
      <!DOCTYPE html>
      <html>
        <body>
          <h1>${data.title}</h1>
          <p>${data.message}</p>
          ${data.actionUrl ? `<a href="${data.actionUrl}">View Details</a>` : ''}
        </body>
      </html>
    `;
  }

  async sendWelcomeEmail(user: { email: string; name: string; verifyToken: string }): Promise<void> {
    await this.addJob({
      to: user.email,
      subject: 'Welcome to Our Platform',
      template: 'welcome',
      data: {
        name: user.name,
        verifyLink: `${process.env.APP_URL}/verify-email?token=${user.verifyToken}`,
      },
    });
  }

  async sendPasswordResetEmail(email: string, resetToken: string): Promise<void> {
    await this.addJob({
      to: email,
      subject: 'Password Reset Request',
      template: 'password-reset',
      data: {
        resetLink: `${process.env.APP_URL}/reset-password?token=${resetToken}`,
      },
    });
  }
}
Image processing queue
typescript
// lib/queues/image.queue.ts
import { BaseQueue } from './base.queue';
import { Job } from 'bullmq';
import sharp from 'sharp';
import { promises as fs } from 'fs';
import path from 'path';

export interface ImageJobData {
  inputPath: string;
  outputFormats: Array<'webp' | 'jpg' | 'png' | 'avif'>;
  sizes: Array<{ width: number; height?: number; suffix: string }>;
  quality?: number;
  optimize?: boolean;
  watermark?: {
    path: string;
    position: 'top-left' | 'top-right' | 'bottom-left' | 'bottom-right' | 'center';
    opacity: number;
  };
}

export interface ImageProcessingResult {
  originalSize: number;
  processedFiles: Array<{
    path: string;
    format: string;
    size: number;
    dimensions: { width: number; height: number };
  }>;
  processingTime: number;
}

export class ImageQueue extends BaseQueue<ImageJobData> {
  constructor() {
    super({
      name: 'image-processing',
      defaultJobOptions: {
        attempts: 3,
        backoff: {
          type: 'fixed',
          delay: 5000,
        },
        timeout: 300000, // 5 minuti timeout
        removeOnComplete: true,
        removeOnFail: 100,
      },
      concurrency: 2, // Limitare concurrency per evitare OOM
    });

    this.process();
  }

  async process(): Promise<void> {
    const worker = createWorker<ImageJobData>(
      this.config.name,
      async (job: Job<ImageJobData>) => {
        return await this.processImage(job);
      },
      {
        concurrency: this.config.concurrency,
      }
    );

    worker.on('active', (job) => {
      job.log(`Starting image processing for ${job.data.inputPath}`);
    });

    worker.on('progress', (job, progress) => {
      console.log(`Progress for job ${job.id}: ${progress}%`);
    });
  }

  private async processImage(job: Job<ImageJobData>): Promise<ImageProcessingResult> {
    const startTime = Date.now();
    const { inputPath, outputFormats, sizes, quality = 80, optimize = true, watermark } = job.data;
    
    const results: ImageProcessingResult['processedFiles'] = [];
    
    try {
      // Leggi l'immagine originale
      const originalBuffer = await fs.readFile(inputPath);
      const originalStats = await fs.stat(inputPath);
      
      // Processa per ogni dimensione
      for (const size of sizes) {
        let image = sharp(originalBuffer)
          .resize(size.width, size.height, {
            fit: 'cover',
            withoutEnlargement: true,
          });
        
        // Applica watermark se specificato
        if (watermark) {
          const watermarkBuffer = await fs.readFile(watermark.path);
          image = await this.applyWatermark(image, watermarkBuffer, watermark);
        }
        
        // Processa per ogni formato
        for (const format of outputFormats) {
          const outputPath = this.getOutputPath(inputPath, size.suffix, format);
          
          let processedImage = image.clone();
          
          // Applica le ottimizzazioni specifiche per formato
          switch (format) {
            case 'webp':
              processedImage = processedImage.webp({ quality, effort: optimize ? 6 : 2 });
              break;
            case 'jpg':
              processedImage = processedImage.jpeg({ quality, mozjpeg: optimize });
              break;
            case 'png':
              processedImage = processedImage.png({ compressionLevel: optimize ? 9 : 3, quality });
              break;
            case 'avif':
              processedImage = processedImage.avif({ quality, effort: optimize ? 9 : 2 });
              break;
          }
          
          const { data, info } = await processedImage.toBuffer({ resolveWithObject: true });
          
          // Scrivi il file
          await fs.writeFile(outputPath, data);
          
          results.push({
            path: outputPath,
            format,
            size: data.length,
            dimensions: { width: info.width, height: info.height },
          });
          
          job.updateProgress((results.length / (sizes.length * outputFormats.length)) * 100);
        }
      }
      
      const processingTime = Date.now() - startTime;
      
      return {
        originalSize: originalStats.size,
        processedFiles: results,
        processingTime,
      };
      
    } catch (error) {
      // Cleanup in caso di errore
      await this.cleanupFailedProcessing(results);
      throw error;
    }
  }

  private async applyWatermark(
    image: sharp.Sharp,
    watermarkBuffer: Buffer,
    options: ImageJobData['watermark']
  ): Promise<sharp.Sharp> {
    const metadata = await image.metadata();
    const watermark = sharp(watermarkBuffer);
    const watermarkMetadata = await watermark.metadata();
    
    if (!metadata.width || !metadata.height || !watermarkMetadata.width || !watermarkMetadata.height) {
      return image;
    }
    
    // Calcola la posizione
    const positions = {
      'top-left': { left: 10, top: 10 },
      'top-right': { left: metadata.width - watermarkMetadata.width - 10, top: 10 },
      'bottom-left': { left: 10, top: metadata.height - watermarkMetadata.height - 10 },
      'bottom-right': { 
        left: metadata.width - watermarkMetadata.width - 10, 
        top: metadata.height - watermarkMetadata.height - 10 
      },
      'center': { 
        left: Math.floor((metadata.width - watermarkMetadata.width) / 2),
        top: Math.floor((metadata.height - watermarkMetadata.height) / 2)
      },
    };
    
    const position = positions[options?.position || 'bottom-right'];
    
    // Composizione con watermark
    return image.composite([{
      input: await watermark.toBuffer(),
      top: position.top,
      left: position.left,
      blend: 'over',
    }]);
  }

  private getOutputPath(inputPath: string, suffix: string, format: string): string {
    const dir = path.dirname(inputPath);
    const name = path.basename(inputPath, path.extname(inputPath));
    return path.join(dir, `${name}_${suffix}.${format}`);
  }

  private async cleanupFailedProcessing(
    files: ImageProcessingResult['processedFiles']
  ): Promise<void> {
    for (const file of files) {
      try {
        await fs.unlink(file.path);
      } catch (error) {
        console.error(`Failed to cleanup file ${file.path}:`, error);
      }
    }
  }

  async createThumbnails(
    imagePath: string,
    sizes: Array<{ width: number; suffix: string }>
  ): Promise<void> {
    await this.addJob({
      inputPath: imagePath,
      outputFormats: ['webp', 'jpg'],
      sizes: sizes.map(size => ({ ...size, height: size.width })),
      quality: 85,
      optimize: true,
    });
  }

  async optimizeImage(imagePath: string): Promise<void> {
    await this.addJob({
      inputPath: imagePath,
      outputFormats: ['webp'],
      sizes: [{ width: 0, suffix: 'optimized' }],
      quality: 90,
      optimize: true,
    });
  }
}
Report generation queue
typescript
// lib/queues/report.queue.ts
import { BaseQueue } from './base.queue';
import { Job } from 'bullmq';
import ExcelJS from 'exceljs';
import PDFDocument from 'pdfkit';
import { createWriteStream, promises as fs } from 'fs';
import path from 'path';

export interface ReportJobData {
  type: 'pdf' | 'excel' | 'csv';
  userId: string;
  filters: {
    dateFrom?: string;
    dateTo?: string;
    category?: string;
    status?: string;
  };
  format: {
    orientation?: 'portrait' | 'landscape';
    includeCharts?: boolean;
    language?: 'en' | 'it' | 'es';
  };
  emailOnComplete?: boolean;
}

export interface ReportResult {
  filePath: string;
  fileSize: number;
  generatedAt: Date;
  recordCount: number;
}

export class ReportQueue extends BaseQueue<ReportJobData> {
  private reportsDir: string;

  constructor() {
    super({
      name: 'report-generation',
      defaultJobOptions: {
        attempts: 2,
        backoff: {
          type: 'fixed',
          delay: 10000,
        },
        timeout: 600000, // 10 minuti
        removeOnComplete: 500,
        removeOnFail: 50,
        priority: 3, // Priorità media
      },
    });

    this.reportsDir = path.join(process.cwd(), 'storage', 'reports');
    this.ensureReportsDirectory();
    
    this.process();
  }

  private async ensureReportsDirectory(): Promise<void> {
    try {
      await fs.mkdir(this.reportsDir, { recursive: true });
    } catch (error) {
      console.error('Failed to create reports directory:', error);
    }
  }

  async process(): Promise<void> {
    const worker = createWorker<ReportJobData>(
      this.config.name,
      async (job: Job<ReportJobData>) => {
        return await this.generateReport(job);
      },
      {
        concurrency: 3,
        limiter: {
          max: 10,
          duration: 60000, // Max 10 report per minuto
        },
      }
    );

    worker.on('progress', (job, progress) => {
      job.log(`Report generation progress: ${progress}%`);
    });
  }

  private async generateReport(job: Job<ReportJobData>): Promise<ReportResult> {
    const { type, userId, filters, format, emailOnComplete } = job.data;
    const timestamp = Date.now();
    const filename = `report_${userId}_${timestamp}.${type}`;
    const filePath = path.join(this.reportsDir, filename);

    job.log(`Starting ${type} report generation for user ${userId}`);
    job.updateProgress(10);

    // Simula il fetch dei dati
    const data = await this.fetchReportData(filters);
    job.updateProgress(40);

    let result: ReportResult;

    switch (type) {
      case 'pdf':
        result = await this.generatePDF(data, filePath, format);
        break;
      case 'excel':
        result = await this.generateExcel(data, filePath, format);
        break;
      case 'csv':
        result = await this.generateCSV(data, filePath);
        break;
      default:
        throw new Error(`Unsupported report type: ${type}`);
    }

    job.updateProgress(90);

    // Invia email se richiesto
    if (emailOnComplete) {
      await this.notifyUser(userId, filename, result.filePath);
    }

    job.updateProgress(100);
    job.log(`Report generated successfully: ${filename}`);

    return result;
  }

  private async fetchReportData(filters: ReportJobData['filters']): Promise<any[]> {
    // Simulazione di fetch dati
    await new Promise(resolve => setTimeout(resolve, 2000));
    
    // In una implementazione reale, qui ci sarebbe una query al database
    return Array.from({ length: 100 }, (_, i) => ({
      id: i + 1,
      date: new Date(Date.now() - i * 86400000).toISOString(),
      category: ['Sales', 'Support', 'Billing'][i % 3],
      amount: Math.random() * 1000,
      status: ['Completed', 'Pending', 'Failed'][i % 3],
    }));
  }

  private async generatePDF(
    data: any[],
    filePath: string,
    format: ReportJobData['format']
  ): Promise<ReportResult> {
    return new Promise((resolve, reject) => {
      const doc = new PDFDocument({
        size: 'A4',
        layout: format?.orientation || 'portrait',
        margin: 50,
      });

      const writeStream = createWriteStream(filePath);
      doc.pipe(writeStream);

      // Header
      doc.fontSize(20).text('Report', { align: 'center' });
      doc.moveDown();
      doc.fontSize(12).text(`Generated: ${new Date().toLocaleString()}`);
      doc.moveDown();

      // Tabella
      const tableTop = doc.y;
      const colWidths = [50, 100, 100, 80, 100];
      const headers = ['ID', 'Date', 'Category', 'Amount', 'Status'];

      // Header della tabella
      doc.font('Helvetica-Bold');
      headers.forEach((header, i) => {
        doc.text(header, 50 + colWidths.slice(0, i).reduce((a, b) => a + b, 0), tableTop, {
          width: colWidths[i],
          align: 'left',
        });
      });

      // Rows
      doc.font('Helvetica');
      data.forEach((row, rowIndex) => {
        const y = tableTop + 30 + rowIndex * 20;
        if (y > doc.page.height - 50) {
          doc.addPage();
        }

        [
          row.id.toString(),
          new Date(row.date).toLocaleDateString(),
          row.category,
          `$${row.amount.toFixed(2)}`,
          row.status,
        ].forEach((cell, i) => {
          doc.text(cell, 50 + colWidths.slice(0, i).reduce((a, b) => a + b, 0), y, {
            width: colWidths[i],
            align: 'left',
          });
        });
      });

      doc.end();

      writeStream.on('finish', async () => {
        const stats = await fs.stat(filePath);
        resolve({
          filePath,
          fileSize: stats.size,
          generatedAt: new Date(),
          recordCount: data.length,
        });
      });

      writeStream.on('error', reject);
    });
  }

  private async generateExcel(
    data: any[],
    filePath: string,
    format: ReportJobData['format']
  ): Promise<ReportResult> {
    const workbook = new ExcelJS.Workbook();
    const worksheet = workbook.addWorksheet('Report');

    // Header
    worksheet.columns = [
      { header: 'ID', key: 'id', width: 10 },
      { header: 'Date', key: 'date', width: 15 },
      { header: 'Category', key: 'category', width: 15 },
      { header: 'Amount', key: 'amount', width: 15 },
      { header: 'Status', key: 'status', width: 15 },
    ];

    // Stile header
    worksheet.getRow(1).font = { bold: true };
    worksheet.getRow(1).fill = {
      type: 'pattern',
      pattern: 'solid',
      fgColor: { argb: 'FFE0E0E0' },
    };

    // Dati
    data.forEach(row => {
      worksheet.addRow({
        id: row.id,
        date: new Date(row.date),
        category: row.category,
        amount: row.amount,
        status: row.status,
      });
    });

    // Formattazione numerica
    worksheet.getColumn('amount').numFmt = '$#,##0.00';

    // Auto filter
    worksheet.autoFilter = {
      from: 'A1',
      to: `E${data.length + 1}`,
    };

    // Salva
    await workbook.xlsx.writeFile(filePath);
    const stats = await fs.stat(filePath);

    return {
      filePath,
      fileSize: stats.size,
      generatedAt: new Date(),
      recordCount: data.length,
    };
  }

  private async generateCSV(data: any[], filePath: string): Promise<ReportResult> {
    const headers = ['ID', 'Date', 'Category', 'Amount', 'Status'];
    const rows = data.map(row => [
      row.id,
      new Date(row.date).toISOString(),
      row.category,
      row.amount,
      row.status,
    ]);

    const csvContent = [
      headers.join(','),
      ...rows.map(row => row.join(',')),
    ].join('\n');

    await fs.writeFile(filePath, csvContent, 'utf-8');
    const stats = await fs.stat(filePath);

    return {
      filePath,
      fileSize: stats.size,
      generatedAt: new Date(),
      recordCount: data.length,
    };
  }

  private async notifyUser(userId: string, filename: string, filePath: string): Promise<void> {
    // Implementazione di notifica email
    console.log(`Notifying user ${userId} about report ${filename}`);
  }

  async generateUserReport(
    userId: string,
    type: ReportJobData['type'],
    filters: ReportJobData['filters'] = {}
  ): Promise<string> {
    const job = await this.addJob({
      type,
      userId,
      filters,
      format: {
        orientation: 'landscape',
        includeCharts: type === 'pdf',
        language: 'en',
      },
      emailOnComplete: true,
    });

    return job.id!;
  }
}
Data sync queue
typescript
// lib/queues/sync.queue.ts
import { BaseQueue } from './base.queue';
import { Job } from 'bullmq';

export interface SyncJobData {
  service: 'stripe' | 'github' | 'google' | 'slack' | 'custom';
  action: 'import' | 'export' | 'sync' | 'webhook';
  entity: string;
  data: any;
  options?: {
    fullSync?: boolean;
    incremental?: boolean;
    batchSize?: number;
    webhookId?: string;
  };
}

export interface SyncResult {
  syncedCount: number;
  added: number;
  updated: number;
  deleted: number;
  errors: Array<{ id: string; error: string }>;
  duration: number;
  nextSyncToken?: string;
}

export class SyncQueue extends BaseQueue<SyncJobData> {
  private serviceClients: Map<string, any> = new Map();

  constructor() {
    super({
      name: 'data-sync',
      defaultJobOptions: {
        attempts: 5,
        backoff: {
          type: 'exponential',
          delay: 5000,
        },
        timeout: 1800000, // 30 minuti
        removeOnComplete: 1000,
        removeOnFail: 100,
        priority: 2,
      },
      limiter: {
        max: 20,
        duration: 60000, // Rate limit per servizi esterni
      },
    });

    this.initializeServiceClients();
    this.process();
  }

  private initializeServiceClients(): void {
    // Inizializza client per servizi esterni
    this.serviceClients.set('stripe', {
      apiKey: process.env.STRIPE_API_KEY,
      // ... altre configurazioni
    });

    this.serviceClients.set('github', {
      token: process.env.GITHUB_TOKEN,
      // ... altre configurazioni
    });
  }

  async process(): Promise<void> {
    const worker = createWorker<SyncJobData>(
      this.config.name,
      async (job: Job<SyncJobData>) => {
        return await this.processSync(job);
      },
      {
        concurrency: 5,
        limiter

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-03-BACKGROUND-JOBS
Prompt ID: 3 / 48
Parte: 2
Exported: 2026-02-06T11:47:31.777Z
Characters: 237
════════════════════════════════════════════════════════════

id} (attempt ${payment.retryCount + 1})`);
    
    try {
      // Implementazione retry pagamento
      // await paymentProcessor.retry(payment);
      
      this.logger.info(`Successfully retried payment ${payment.id}`);
    } catch (

## § ADVANCED PATTERNS: BACKGROUND JOBS

### Server Actions con Validazione

```typescript
// app/actions/background-jobs.ts
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


### BACKGROUND JOBS - Utility Helper #62

```typescript
// lib/utils/background-jobs-helper-62.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #189

```typescript
// lib/utils/background-jobs-helper-189.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #399

```typescript
// lib/utils/background-jobs-helper-399.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #199

```typescript
// lib/utils/background-jobs-helper-199.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #551

```typescript
// lib/utils/background-jobs-helper-551.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #183

```typescript
// lib/utils/background-jobs-helper-183.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #982

```typescript
// lib/utils/background-jobs-helper-982.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #501

```typescript
// lib/utils/background-jobs-helper-501.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #247

```typescript
// lib/utils/background-jobs-helper-247.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #904

```typescript
// lib/utils/background-jobs-helper-904.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #828

```typescript
// lib/utils/background-jobs-helper-828.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #3

```typescript
// lib/utils/background-jobs-helper-3.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #465

```typescript
// lib/utils/background-jobs-helper-465.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #986

```typescript
// lib/utils/background-jobs-helper-986.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #476

```typescript
// lib/utils/background-jobs-helper-476.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #897

```typescript
// lib/utils/background-jobs-helper-897.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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

  getConfig(): Readonly<BACKGROUNDJOBSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BACKGROUND JOBS - Utility Helper #504

```typescript
// lib/utils/background-jobs-helper-504.ts
import { z } from "zod";

interface BACKGROUNDJOBSConfig {
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

export class BACKGROUNDJOBSProcessor<TInput, TOutput> {
  private config: BACKGROUNDJOBSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BACKGROUNDJOBSConfig> = {}) {
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
        const timeoutId = setTimeout(() => controller.abort(), this.config.time