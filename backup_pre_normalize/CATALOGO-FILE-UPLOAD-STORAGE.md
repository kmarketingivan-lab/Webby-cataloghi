# CATALOGO-FILE-UPLOAD-STORAGE

markdown
# ESPANSIONE CATALOGO-FILE-UPLOAD-STORAGE PER NEXT.JS 14

## § UPLOADTHING COMPLETE

### Configurazione Completa Next.js 14

```typescript
// app/api/uploadthing/core.ts
import { createUploadthing, type FileRouter } from "uploadthing/next";
import { UploadThingError } from "uploadthing/server";
import { z } from "zod";
import { getServerSession } from "next-auth";

const f = createUploadthing();

export const ourFileRouter = {
  imageUploader: f({
    image: {
      maxFileSize: "4MB",
      maxFileCount: 10,
    },
    pdf: {
      maxFileSize: "16MB",
    }
  })
    .input(z.object({
      userId: z.string(),
      albumId: z.string().optional(),
    }))
    .middleware(async ({ req, input }) => {
      const session = await getServerSession(req);
      if (!session) throw new UploadThingError("UNAUTHORIZED");
      
      return { userId: input.userId };
    })
    .onUploadComplete(async ({ metadata, file }) => {
      await db.file.create({
        data: {
          key: file.key,
          name: file.name,
          url: file.url,
          userId: metadata.userId,
          size: file.size,
          type: file.type,
        }
      });
      return { uploadedBy: metadata.userId };
    }),

  videoUploader: f({
    video: {
      maxFileSize: "512MB",
      maxFileCount: 1,
    }
  })
    .middleware(async ({ req }) => {
      const session = await getServerSession(req);
      if (!session) throw new UploadThingError("UNAUTHORIZED");
      
      return { userId: session.user.id };
    })
    .onUploadProgress(async ({ progress }) => {
      console.log(`Progress: ${progress}%`);
      await redis.set(`upload:${reqId}:progress`, progress);
    })
    .onUploadComplete(async ({ metadata, file }) => {
      console.log("Upload completed:", file.url);
    }),
} satisfies FileRouter;

export type OurFileRouter = typeof ourFileRouter;
typescript
// app/providers.tsx
"use client";

import { UploadButton } from "@/utils/uploadthing";
import { useState } from "react";
import { Progress } from "@/components/ui/progress";
import { toast } from "sonner";

export function FileUpload() {
  const [progress, setProgress] = useState(0);
  const [files, setFiles] = useState<File[]>([]);

  return (
    <div className="flex flex-col items-center justify-center p-8">
      <UploadButton
        endpoint="imageUploader"
        input={{
          userId: "user_123",
          albumId: "album_456"
        }}
        onUploadProgress={(p) => setProgress(p)}
        onClientUploadComplete={(res) => {
          toast.success("Upload completato!");
          setFiles(res?.map(f => f.file) || []);
        }}
        onUploadError={(error: Error) => {
          toast.error(`ERRORE: ${error.message}`);
        }}
        config={{
          mode: "auto",
          appendOnPaste: true,
        }}
      />
      
      {progress > 0 && progress < 100 && (
        <Progress value={progress} className="w-[300px] mt-4" />
      )}
      
      <div className="mt-4">
        {files.map((file, index) => (
          <div key={index} className="flex items-center space-x-2">
            <FileIcon type={file.type} />
            <span>{file.name}</span>
            <span className="text-sm text-gray-500">
              {(file.size / 1024 / 1024).toFixed(2)} MB
            </span>
          </div>
        ))}
      </div>
    </div>
  );
}
Validazione Estesa dei File
typescript
// utils/file-validation.ts
import { UTApi } from "uploadthing/server";

export const ALLOWED_MIME_TYPES = {
  image: ['image/jpeg', 'image/png', 'image/gif', 'image/webp', 'image/avif'],
  document: ['application/pdf', 'application/msword', 
             'application/vnd.openxmlformats-officedocument.wordprocessingml.document'],
  video: ['video/mp4', 'video/webm', 'video/quicktime'],
  audio: ['audio/mpeg', 'audio/wav', 'audio/ogg']
} as const;

export const MAX_FILE_SIZES = {
  avatar: 2 * 1024 * 1024, // 2MB
  document: 10 * 1024 * 1024, // 10MB
  video: 500 * 1024 * 1024, // 500MB
  archive: 100 * 1024 * 1024, // 100MB
} as const;

export async function validateFile(file: File, category: keyof typeof ALLOWED_MIME_TYPES) {
  // Validazione dimensione
  if (file.size > MAX_FILE_SIZES[category]) {
    throw new Error(`File troppo grande. Massimo: ${MAX_FILE_SIZES[category] / 1024 / 1024}MB`);
  }

  // Validazione MIME type
  if (!ALLOWED_MIME_TYPES[category].includes(file.type)) {
    throw new Error(`Tipo file non supportato. Consentiti: ${ALLOWED_MIME_TYPES[category].join(', ')}`);
  }

  // Validazione magic bytes
  const buffer = await file.slice(0, 4).arrayBuffer();
  const view = new DataView(buffer);
  const magic = view.getUint32(0, false);
  
  const MAGIC_NUMBERS = {
    JPEG: 0xFFD8FFE0,
    PNG: 0x89504E47,
    PDF: 0x25504446,
  };

  if (file.type === 'image/jpeg' && magic !== MAGIC_NUMBERS.JPEG) {
    throw new Error("File JPEG non valido");
  }

  return true;
}

// Utility per cancellazione file
export async function deleteUploadedFile(url: string) {
  const utapi = new UTApi();
  const fileKey = url.split('/').pop();
  if (fileKey) {
    await utapi.deleteFiles(fileKey);
  }
}
§ AWS S3 DIRECT UPLOAD
Presigned URLs e Multipart Upload
typescript
// lib/aws/s3-upload.ts
import { S3Client, PutObjectCommand, CreateMultipartUploadCommand, 
         UploadPartCommand, CompleteMultipartUploadCommand, 
         AbortMultipartUploadCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";
import axios from "axios";

const s3Client = new S3Client({
  region: process.env.AWS_REGION!,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});

export interface PresignedUrlResponse {
  url: string;
  key: string;
  uploadId?: string;
  partUrls?: string[];
}

export class S3UploadService {
  private readonly bucket = process.env.AWS_S3_BUCKET!;
  
  // Genera URL presigned per upload diretto
  async generatePresignedUrl(
    fileName: string, 
    fileType: string,
    userId: string
  ): Promise<PresignedUrlResponse> {
    const key = `uploads/${userId}/${Date.now()}-${fileName}`;
    
    const command = new PutObjectCommand({
      Bucket: this.bucket,
      Key: key,
      ContentType: fileType,
      Metadata: {
        userId,
        originalName: fileName,
      },
    });

    const url = await getSignedUrl(s3Client, command, { expiresIn: 3600 });
    
    return { url, key };
  }

  // Setup multipart upload per file grandi
  async createMultipartUpload(
    fileName: string,
    fileType: string,
    fileSize: number
  ) {
    const key = `large-files/${Date.now()}-${fileName}`;
    
    const command = new CreateMultipartUploadCommand({
      Bucket: this.bucket,
      Key: key,
      ContentType: fileType,
    });

    const response = await s3Client.send(command);
    
    return {
      uploadId: response.UploadId,
      key: response.Key,
    };
  }

  // Genera URL presigned per ogni parte
  async generatePartUrls(
    key: string,
    uploadId: string,
    partCount: number
  ): Promise<string[]> {
    const partUrls: string[] = [];
    
    for (let partNumber = 1; partNumber <= partCount; partNumber++) {
      const command = new UploadPartCommand({
        Bucket: this.bucket,
        Key: key,
        UploadId: uploadId,
        PartNumber: partNumber,
      });
      
      const url = await getSignedUrl(s3Client, command, { expiresIn: 3600 });
      partUrls.push(url);
    }
    
    return partUrls;
  }

  // Completa upload multipart
  async completeMultipartUpload(
    key: string,
    uploadId: string,
    parts: Array<{ ETag: string; PartNumber: number }>
  ) {
    const command = new CompleteMultipartUploadCommand({
      Bucket: this.bucket,
      Key: key,
      UploadId: uploadId,
      MultipartUpload: { Parts: parts },
    });

    return await s3Client.send(command);
  }
}
Upload Progressivo con Axios
typescript
// components/s3-uploader.tsx
"use client";

import { useState, useRef } from "react";
import axios, { AxiosProgressEvent } from "axios";
import { Progress } from "@/components/ui/progress";

interface UploadPart {
  partNumber: number;
  etag?: string;
  progress: number;
}

export function S3DirectUploader() {
  const [progress, setProgress] = useState(0);
  const [parts, setParts] = useState<UploadPart[]>([]);
  const [isUploading, setIsUploading] = useState(false);
  const abortControllerRef = useRef<AbortController | null>(null);

  const uploadToS3 = async (file: File, presignedUrl: string) => {
    setIsUploading(true);
    abortControllerRef.current = new AbortController();

    try {
      await axios.put(presignedUrl, file, {
        headers: {
          "Content-Type": file.type,
        },
        onUploadProgress: (progressEvent: AxiosProgressEvent) => {
          if (progressEvent.total) {
            const percent = Math.round(
              (progressEvent.loaded * 100) / progressEvent.total
            );
            setProgress(percent);
          }
        },
        signal: abortControllerRef.current.signal,
      });

      return true;
    } catch (error) {
      if (axios.isCancel(error)) {
        console.log("Upload cancellato dall'utente");
      }
      throw error;
    } finally {
      setIsUploading(false);
    }
  };

  const uploadMultipart = async (file: File, partUrls: string[]) => {
    const CHUNK_SIZE = 10 * 1024 * 1024; // 10MB chunks
    const totalParts = Math.ceil(file.size / CHUNK_SIZE);
    
    const uploadPromises = [];
    const uploadedParts: Array<{ ETag: string; PartNumber: number }> = [];

    for (let i = 0; i < totalParts; i++) {
      const start = i * CHUNK_SIZE;
      const end = Math.min(start + CHUNK_SIZE, file.size);
      const chunk = file.slice(start, end);

      const partNumber = i + 1;
      setParts(prev => [...prev, { partNumber, progress: 0 }]);

      uploadPromises.push(
        axios.put(partUrls[i], chunk, {
          onUploadProgress: (progressEvent: AxiosProgressEvent) => {
            if (progressEvent.total) {
              const partProgress = Math.round(
                (progressEvent.loaded * 100) / progressEvent.total
              );
              setParts(prev => 
                prev.map(p => 
                  p.partNumber === partNumber 
                    ? { ...p, progress: partProgress }
                    : p
                )
              );
            }
          },
        }).then(response => {
          const etag = response.headers.etag;
          if (etag) {
            uploadedParts.push({ ETag: etag.replace(/"/g, ''), PartNumber: partNumber });
          }
        })
      );
    }

    await Promise.all(uploadPromises);
    return uploadedParts.sort((a, b) => a.PartNumber - b.PartNumber);
  };

  const cancelUpload = () => {
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }
  };

  return (
    <div className="space-y-4">
      <Progress value={progress} />
      <div className="grid grid-cols-4 gap-2">
        {parts.map(part => (
          <div key={part.partNumber} className="text-center">
            <div className="text-sm">Part {part.partNumber}</div>
            <Progress value={part.progress} className="h-2" />
          </div>
        ))}
      </div>
    </div>
  );
}
Configurazione CORS e Bucket Policies
json
// bucket-cors-policy.json
{
  "CORSRules": [
    {
      "AllowedOrigins": ["https://yourdomain.com", "http://localhost:3000"],
      "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
      "AllowedHeaders": ["*"],
      "ExposeHeaders": ["ETag", "x-amz-meta-custom-header"],
      "MaxAgeSeconds": 3000
    }
  ]
}

// bucket-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadForGetBucketObjects",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket/*",
      "Condition": {
        "StringEquals": {
          "aws:UserAgent": "Next.js-Uploader"
        }
      }
    },
    {
      "Sid": "WriteAccessForUploads",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_ID:user/upload-user"
      },
      "Action": [
        "s3:PutObject",
        "s3:PutObjectAcl",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::your-bucket/uploads/*"
    }
  ]
}
§ CLOUDFLARE R2
Configurazione API S3-compatibile
typescript
// lib/cloudflare/r2-client.ts
import { S3Client, PutObjectCommand, GetObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";

export class R2Storage {
  private client: S3Client;
  private readonly bucket: string;
  
  constructor() {
    this.client = new S3Client({
      region: "auto",
      endpoint: `https://${process.env.CLOUDFLARE_ACCOUNT_ID}.r2.cloudflarestorage.com`,
      credentials: {
        accessKeyId: process.env.R2_ACCESS_KEY_ID!,
        secretAccessKey: process.env.R2_SECRET_ACCESS_KEY!,
      },
    });
    this.bucket = process.env.R2_BUCKET_NAME!;
  }

  async uploadFile(
    file: Buffer | Uint8Array,
    fileName: string,
    contentType: string
  ) {
    const key = `uploads/${Date.now()}-${fileName}`;
    
    const command = new PutObjectCommand({
      Bucket: this.bucket,
      Key: key,
      Body: file,
      ContentType: contentType,
      Metadata: {
        uploadedAt: new Date().toISOString(),
      },
    });

    await this.client.send(command);
    
    return {
      key,
      url: await this.getPublicUrl(key),
    };
  }

  async getSignedUrl(key: string, expiresIn: number = 3600) {
    const command = new GetObjectCommand({
      Bucket: this.bucket,
      Key: key,
    });

    return await getSignedUrl(this.client, command, { expiresIn });
  }

  getPublicUrl(key: string): string {
    // Se hai configurato un custom domain
    if (process.env.R2_PUBLIC_DOMAIN) {
      return `https://${process.env.R2_PUBLIC_DOMAIN}/${key}`;
    }
    
    // URL standard R2
    return `https://pub-${process.env.CLOUDFLARE_ACCOUNT_ID}.r2.dev/${key}`;
  }

  async deleteFile(key: string): Promise<void> {
    const command = new DeleteObjectCommand({
      Bucket: this.bucket,
      Key: key,
    });

    await this.client.send(command);
  }
}

// Confronto costi S3 vs R2
export function compareCosts(fileSizeGB: number, operations: number) {
  const s3Costs = {
    storage: fileSizeGB * 0.023, // $0.023 per GB/mese
    getOperations: (operations * 0.0004) / 1000, // $0.0004 per 1000 GET
    putOperations: (operations * 0.005) / 1000, // $0.005 per 1000 PUT
  };

  const r2Costs = {
    storage: fileSizeGB * 0.015, // $0.015 per GB/mese
    getOperations: 0, // GET gratuiti
    putOperations: (operations * 0.006) / 1000, // $0.006 per 1000 PUT
  };

  return {
    s3: Object.values(s3Costs).reduce((a, b) => a + b, 0),
    r2: Object.values(r2Costs).reduce((a, b) => a + b, 0),
    savings: Object.values(s3Costs).reduce((a, b) => a + b, 0) - 
             Object.values(r2Costs).reduce((a, b) => a + b, 0),
  };
}
Integrazione CDN Cloudflare
typescript
// lib/cloudflare/cdn.ts
export class CloudflareCDN {
  private readonly accountId = process.env.CLOUDFLARE_ACCOUNT_ID!;
  private readonly apiToken = process.env.CLOUDFLARE_API_TOKEN!;

  async purgeCache(urls: string[]) {
    const response = await fetch(
      `https://api.cloudflare.com/client/v4/zones/${process.env.CLOUDFLARE_ZONE_ID}/purge_cache`,
      {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${this.apiToken}`,
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          files: urls,
        }),
      }
    );

    return await response.json();
  }

  async uploadWithCDN(
    file: File,
    optimize: boolean = true
  ) {
    const formData = new FormData();
    formData.append('file', file);

    // Upload diretto a Cloudflare Images
    const response = await fetch(
      `https://api.cloudflare.com/client/v4/accounts/${this.accountId}/images/v1`,
      {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${this.apiToken}`,
        },
        body: formData,
      }
    );

    const data = await response.json();
    
    if (optimize) {
      return {
        original: data.result.variants[0],
        thumbnail: `${data.result.variants[0]}/thumbnail`,
        optimized: `${data.result.variants[0]}?format=avif&quality=80`,
      };
    }

    return data.result;
  }
}
§ IMAGE OPTIMIZATION
Processing Server-side con Sharp
typescript
// lib/image-processing.ts
import sharp from 'sharp';
import { promises as fs } from 'fs';
import path from 'path';

export interface ImageOptimizationOptions {
  width?: number;
  height?: number;
  quality?: number;
  format?: 'webp' | 'avif' | 'jpeg' | 'png';
  fit?: 'cover' | 'contain' | 'fill' | 'inside' | 'outside';
  position?: string;
}

export class ImageProcessor {
  async optimizeImage(
    inputPath: string,
    outputPath: string,
    options: ImageOptimizationOptions = {}
  ) {
    const {
      width = 1200,
      height,
      quality = 80,
      format = 'webp',
      fit = 'inside',
      position = 'center'
    } = options;

    let pipeline = sharp(inputPath)
      .rotate()
      .resize(width, height, {
        fit,
        position,
        withoutEnlargement: true,
      });

    // Applica formattazione specifica
    switch (format) {
      case 'webp':
        pipeline = pipeline.webp({ quality, effort: 4 });
        break;
      case 'avif':
        pipeline = pipeline.avif({ quality, effort: 4 });
        break;
      case 'jpeg':
        pipeline = pipeline.jpeg({ quality, mozjpeg: true });
        break;
      case 'png':
        pipeline = pipeline.png({ compressionLevel: 9 });
        break;
    }

    await pipeline.toFile(outputPath);

    return {
      path: outputPath,
      format,
      size: (await fs.stat(outputPath)).size,
    };
  }

  async generateThumbnails(
    inputPath: string,
    outputDir: string,
    sizes: Array<{ width: number; height?: number; suffix: string }>
  ) {
    const results = [];
    
    for (const size of sizes) {
      const outputPath = path.join(
        outputDir,
        `${path.parse(inputPath).name}-${size.suffix}.webp`
      );

      await this.optimizeImage(inputPath, outputPath, {
        width: size.width,
        height: size.height,
        format: 'webp',
        quality: 75,
      });

      results.push({
        size: size.suffix,
        path: outputPath,
        width: size.width,
      });
    }

    return results;
  }

  async generateBlurPlaceholder(inputPath: string): Promise<string> {
    const image = sharp(inputPath);
    const metadata = await image.metadata();
    
    // Crea placeholder piccolo
    const { data, info } = await image
      .resize(20, 20, { fit: 'inside' })
      .blur(2)
      .toBuffer({ resolveWithObject: true });

    // Converti in base64
    const base64 = `data:image/${info.format};base64,${data.toString('base64')}`;
    
    return base64;
  }

  async extractDominantColor(inputPath: string): Promise<string> {
    const { dominant } = await sharp(inputPath)
      .resize(1, 1)
      .raw()
      .toBuffer({ resolveWithObject: true });

    const [r, g, b] = dominant.slice(0, 3);
    return `rgb(${r}, ${g}, ${b})`;
  }

  async batchProcessImages(
    images: Array<{ input: string; output: string; options?: ImageOptimizationOptions }>
  ) {
    const promises = images.map(img =>
      this.optimizeImage(img.input, img.output, img.options)
    );

    return await Promise.all(promises);
  }
}

// Next.js Image Optimization Middleware
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const url = request.nextUrl.clone();
  
  // Riscrivi URL per immagini ottimizzate
  if (url.pathname.startsWith('/_next/image')) {
    const searchParams = url.searchParams;
    const width = searchParams.get('w');
    const quality = searchParams.get('q') || '75';
    
    // Aggiungi parametri di ottimizzazione
    searchParams.set('q', quality);
    
    if (width && parseInt(width) > 1920) {
      searchParams.set('w', '1920');
    }
  }
  
  return NextResponse.rewrite(url);
}
Generazione LQIP (Low Quality Image Placeholders)
typescript
// components/optimized-image.tsx
"use client";

import { useState, useEffect } from 'react';
import Image from 'next/image';

interface OptimizedImageProps {
  src: string;
  alt: string;
  width: number;
  height: number;
  placeholder?: string;
  sizes?: string;
  priority?: boolean;
}

export function OptimizedImage({
  src,
  alt,
  width,
  height,
  placeholder,
  sizes = "(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw",
  priority = false,
}: OptimizedImageProps) {
  const [isLoaded, setIsLoaded] = useState(false);
  const [placeholderColor, setPlaceholderColor] = useState<string>('#f0f0f0');

  // Genera gradient placeholder basato sul colore dominante
  useEffect(() => {
    if (!placeholder && src) {
      // In una implementazione reale, potresti pre-calcolare questo lato server
      const hash = src.split('').reduce((a, b) => {
        a = (a << 5) - a + b.charCodeAt(0);
        return a & a;
      }, 0);
      const hue = Math.abs(hash % 360);
      setPlaceholderColor(`linear-gradient(135deg, hsl(${hue}, 50%, 85%), hsl(${hue}, 50%, 95%))`);
    }
  }, [src, placeholder]);

  const srcSet = `
    ${src}?width=320 320w,
    ${src}?width=640 640w,
    ${src}?width=768 768w,
    ${src}?width=1024 1024w,
    ${src}?width=1280 1280w,
    ${src}?width=1536 1536w
  `;

  return (
    <div className="relative overflow-hidden" style={{ aspectRatio: `${width}/${height}` }}>
      {/* Placeholder */}
      {!isLoaded && (
        <div 
          className="absolute inset-0 animate-pulse"
          style={{ 
            background: placeholder || placeholderColor,
          }}
        />
      )}

      {/* Immagine ottimizzata */}
      <Image
        src={src}
        alt={alt}
        fill
        sizes={sizes}
        srcSet={srcSet}
        loading={priority ? "eager" : "lazy"}
        quality={75}
        className={`
          transition-opacity duration-500
          ${isLoaded ? 'opacity-100' : 'opacity-0'}
        `}
        onLoad={() => setIsLoaded(true)}
        style={{
          objectFit: 'cover',
        }}
      />
    </div>
  );
}
§ FILE TYPE HANDLING
Anteprima Documenti
typescript
// lib/document-preview.ts
import mammoth from 'mammoth';
import pdfParse from 'pdf-parse';
import { readFile } from 'fs/promises';

export class DocumentPreview {
  async extractTextFromPDF(buffer: Buffer): Promise<string> {
    const data = await pdfParse(buffer);
    return data.text;
  }

  async extractTextFromDocx(buffer: Buffer): Promise<string> {
    const result = await mammoth.extractRawText({ buffer });
    return result.value;
  }

  async generatePDFPreview(
    pdfBuffer: Buffer,
    outputPath: string,
    pageNumber: number = 1
  ): Promise<string> {
    // Utilizzando pdf2json o similar per estrazione
    const { PDFDocument } = await import('pdf-lib');
    
    const pdfDoc = await PDFDocument.load(pdfBuffer);
    const firstPage = pdfDoc.getPages()[pageNumber - 1];
    
    // Estrai dimensioni per creare un'anteprima
    const { width, height } = firstPage.getSize();
    
    // In una implementazione reale, useresti una libreria come pdf2image
    return `data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" width="${width}" height="${height}">
      <rect width="100%" height="100%" fill="#f0f0f0"/>
      <text x="50%" y="50%" text-anchor="middle" fill="#666">PDF Preview</text>
    </svg>`;
  }

  async getDocumentMetadata(buffer: Buffer, mimeType: string) {
    const metadata: Record<string, any> = {};

    switch (mimeType) {
      case 'application/pdf':
        const pdfData = await pdfParse(buffer);
        metadata.pages = pdfData.numpages;
        metadata.info = pdfData.info;
        break;
      
      case 'application/vnd.openxmlformats-officedocument.wordprocessingml.document':
        const docxText = await this.extractTextFromDocx(buffer);
        metadata.wordCount = docxText.split(/\s+/).length;
        metadata.preview = docxText.substring(0, 500) + '...';
        break;
      
      case 'application/vnd.ms-excel':
        const xlsx = await import('xlsx');
        const workbook = xlsx.read(buffer, { type: 'buffer' });
        metadata.sheets = workbook.SheetNames;
        metadata.sheetCount = workbook.SheetNames.length;
        break;
    }

    return metadata;
  }
}
Video Thumbnails e Waveforms
typescript
// lib/media-processing.ts
import ffmpeg from 'fluent-ffmpeg';
import ffmpegInstaller from '@ffmpeg-installer/ffmpeg';
import { Writable } from 'stream';
import path from 'path';

ffmpeg.setFfmpegPath(ffmpegInstaller.path);

export class MediaProcessor {
  async generateVideoThumbnail(
    videoPath: string,
    outputPath: string,
    timestamp: string = '00:00:01'
  ): Promise<string> {
    return new Promise((resolve, reject) => {
      ffmpeg(videoPath)
        .screenshots({
          timestamps: [timestamp],
          filename: path.basename(outputPath),
          folder: path.dirname(outputPath),
          size: '640x360'
        })
        .on('end', () => resolve(outputPath))
        .on('error', reject);
    });
  }

  async extractAudioWaveform(
    audioPath: string,
    samples: number = 100
  ): Promise<number[]> {
    return new Promise((resolve, reject) => {
      const waveform: number[] = [];
      
      ffmpeg(audioPath)
        .audioChannels(1)
        .audioFrequency(44100)
        .format('s16le')
        .pipe(
          new Writable({
            write(chunk, encoding, callback) {
              // Processa chunk audio per estrarre waveform
              for (let i = 0; i < chunk.length; i += 2) {
                if (waveform.length < samples) {
                  const sample = chunk.readInt16LE(i);
                  waveform.push(Math.abs(sample) / 32768);
                }
              }
              callback();
            },
          }),
          { end: true }
        )
        .on('end', () => resolve(waveform))
        .on('error', reject);
    });
  }

  async getVideoMetadata(videoPath: string) {
    return new Promise((resolve, reject) => {
      ffmpeg.ffprobe(videoPath, (err, metadata) => {
        if (err) reject(err);
        
        const videoStream = metadata.streams.find(s => s.codec_type === 'video');
        const audioStream = metadata.streams.find(s => s.codec_type === 'audio');
        
        resolve({
          duration: metadata.format.duration,
          size: metadata.format.size,
          format: metadata.format.format_name,
          video: videoStream ? {
            codec: videoStream.codec_name,
            width: videoStream.width,
            height: videoStream.height,
            fps: eval(videoStream.r_frame_rate || '0'),
            bitrate: videoStream.bit_rate,
          } : null,
          audio: audioStream ? {
            codec: audioStream.codec_name,
            channels: audioStream.channels,
            sampleRate: audioStream.sample_rate,
          } : null,
        });
      });
    });
  }

  async compressVideo(
    inputPath: string,
    outputPath: string,
    options: {
      crf?: number;
      preset?: string;
      maxHeight?: number;
      bitrate?: string;
    } = {}
  ) {
    const { crf = 23, preset = 'medium', maxHeight = 720, bitrate = '1M' } = options;

    return new Promise((resolve, reject) => {
      ffmpeg(inputPath)
        .outputOptions([
          '-c:v libx264',
          `-crf ${crf}`,
          `-preset ${preset}`,
          `-maxrate ${bitrate}`,
          '-bufsize 2M',
          '-c:a aac',
          '-b:a 128k',
        ])
        .size(`?x${maxHeight}`)
        .output(outputPath)
        .on('end', () => resolve(outputPath))
        .on('error', reject)
        .run();
    });
  }
}
§ SECURITY
Validazione File e Virus Scanning
typescript
// lib/security/file-security.ts
import { fileTypeFromBuffer } from 'file-type';
import { spawn } from 'child_process';
import { createHash } from 'crypto';

export class FileSecurity {
  private static readonly MAGIC_NUMBERS = {
    PDF: [0x25, 0x50, 0x44, 0x46],
    JPEG: [0xFF, 0xD8, 0xFF],
    PNG: [0x89, 0x50, 0x4E, 0x47],
    ZIP: [0x50, 0x4B, 0x03, 0x04],
    GIF: [0x47, 0x49, 0x46, 0x38],
  };

  // Validazione magic bytes
  static async validateFileType(buffer: Buffer, expectedMime: string): Promise<boolean> {
    const fileType = await fileTypeFromBuffer(buffer);
    
    if (!fileType) {
      throw new Error('Tipo file non riconoscibile');
    }

    if (fileType.mime !== expectedMime) {
      throw new Error(`MIME type mismatch: expected ${expectedMime}, got ${fileType.mime}`);
    }

    // Validazione aggiuntiva basata sui primi bytes
    const firstBytes = buffer.slice(0, 4);
    const isValid = this.checkMagicBytes(firstBytes, expectedMime);
    
    if (!isValid) {
      throw new Error('File corrupted or invalid signature');
    }

    return true;
  }

  private static checkMagicBytes(bytes: Buffer, mimeType: string): boolean {
    const hexBytes = Array.from(bytes.slice(0, 4));
    
    switch (mimeType) {
      case 'application/pdf':
        return this.compareBytes(hexBytes, this.MAGIC_NUMBERS.PDF);
      case 'image/jpeg':
        return this.compareBytes(hexBytes, this.MAGIC_NUMBERS.JPEG);
      case 'image/png':
        return this.compareBytes(hexBytes, this.MAGIC_NUMBERS.PNG);
      default:
        return true;
    }
  }

  private static compareBytes(a: number[], b: number[]): boolean {
    return a.slice(0, b.length).every((byte, i) => byte === b[i]);
  }

  // Virus scanning con ClamAV
  static async scanForViruses(filePath: string): Promise<boolean> {
    return new Promise((resolve, reject) => {
      const clamscan = spawn('clamscan', ['--no-summary', filePath]);
      
      let output = '';
      clamscan.stdout.on('data', (data) => output +=

════════════════════════════════════════════════════════════
FIGMA CATALOG: BATCH2-05-FILE-UPLOAD-STORAGE
Prompt ID: 5 / 19
Parte: 2
Exported: 2026-02-06T16:59:34.078Z
Characters: 888
════════════════════════════════════════════════════════════

 async uploadChunk(
    sessionId: string,
    chunkIndex: number,
    onProgress?: (progress: number) => void
  ): Promise<boolean> {
    const session = this.uploadSessions.get(sessionId);
    if (!session) throw new Error('Session not found');

    const chunk = session.chunks[chunkIndex];
    if (!chunk) throw new Error('Chunk not found');

    const chunkData = session.file.slice(chunk.start, chunk.end);
    chunk.status = 'uploading';

    try {
      // Calcola hash del chunk per integrità
      const hash = await this.calculateHash(chunkData);
      chunk.hash = hash;

      // Upload chunk
      await this.uploadChunkToServer(sessionId, chunkIndex, chunkData, (progress) => {
        chunk.uploadedBytes = Math.round((progress / 100) * chunk.size);
        onProgress?.(progress);
      });

      chunk.status = 'completed';
      session.uploadedChunks++;
      session