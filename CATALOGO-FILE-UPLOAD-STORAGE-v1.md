# CATALOGO FILE UPLOAD & STORAGE v1

> **Versione**: 1.0
> **Data**: 2026-01-27
> **Ambito**: File upload, S3/R2 storage, multipart upload, image processing, security patterns

---

§ 1. PRESIGNED URL UPLOAD (STANDARD)

§ 1.1 S3 PRESIGNED URL GENERATION

typescript
// src/lib/s3.ts
import { S3Client, PutObjectCommand, GetObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import { v4 as uuid } from 'uuid';

const s3 = new S3Client({
  region: process.env.AWS_REGION || 'auto',
  endpoint: process.env.S3_ENDPOINT,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});

const BUCKET = process.env.S3_BUCKET!;

export async function getUploadPresignedUrl(
  filename: string,
  contentType: string,
  folder = 'uploads',
  expiresIn = 900 // 15 minutes
) {
  const extension = filename.split('.').pop();
  const key = `${folder}/${uuid()}.${extension}`;

  const command = new PutObjectCommand({
    Bucket: BUCKET,
    Key: key,
    ContentType: contentType,
    Metadata: { 'original-filename': encodeURIComponent(filename) },
  });

  const url = await getSignedUrl(s3, command, { expiresIn });
  return { url, key };
}

export async function getDownloadPresignedUrl(key: string, expiresIn = 3600) {
  const command = new GetObjectCommand({ Bucket: BUCKET, Key: key });
  return getSignedUrl(s3, command, { expiresIn });
}

§ 1.2 API ROUTE (NEXT.JS)

typescript
// src/app/api/upload/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getUploadPresignedUrl } from '@/lib/s3';
import { z } from 'zod';

const uploadSchema = z.object({
  filename: z.string().min(1).max(255),
  contentType: z.string().regex(/^(image|video|application)\//),
  size: z.number().max(50 * 1024 * 1024), // 50MB max
});

const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/webp', 'application/pdf'];

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { filename, contentType, size } = uploadSchema.parse(body);

    if (!ALLOWED_TYPES.includes(contentType)) {
      return NextResponse.json({ error: 'File type not allowed' }, { status: 400 });
    }

    const { url, key } = await getUploadPresignedUrl(filename, contentType);
    return NextResponse.json({ uploadUrl: url, key });
  } catch (error) {
    return NextResponse.json({ error: 'Invalid request' }, { status: 400 });
  }
}

§ 1.3 CLIENT UPLOAD FUNCTION

typescript
// src/lib/upload.ts
interface UploadOptions {
  file: File;
  onProgress?: (progress: number) => void;
}

export async function uploadFile({ file, onProgress }: UploadOptions) {
  // 1. Get presigned URL from API
  const res = await fetch('/api/upload', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      filename: file.name,
      contentType: file.type,
      size: file.size,
    }),
  });

  if (!res.ok) throw new Error('Failed to get upload URL');
  const { uploadUrl, key } = await res.json();

  // 2. Upload directly to S3
  await uploadWithProgress(uploadUrl, file, file.type, onProgress);

  // 3. Return the file key
  return { key, publicUrl: `${process.env.NEXT_PUBLIC_CDN_URL}/${key}` };
}

function uploadWithProgress(
  url: string,
  file: File,
  contentType: string,
  onProgress?: (progress: number) => void
) {
  return new Promise<void>((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.upload.onprogress = (e) => {
      if (e.lengthComputable && onProgress) {
        onProgress(Math.round((e.loaded / e.total) * 100));
      }
    };
    xhr.onload = () => {
      if (xhr.status >= 200 && xhr.status < 300) resolve();
      else reject(new Error(`Upload failed: ${xhr.status}`));
    };
    xhr.onerror = () => reject(new Error('Upload error'));
    xhr.open('PUT', url);
    xhr.setRequestHeader('Content-Type', contentType);
    xhr.send(file);
  });
}

---

§ 2. CHUNKED / MULTIPART UPLOAD (LARGE FILES)

typescript
// src/lib/s3-multipart.ts
import { 
  S3Client, 
  CreateMultipartUploadCommand, 
  UploadPartCommand, 
  CompleteMultipartUploadCommand, 
  AbortMultipartUploadCommand 
} from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import { v4 as uuid } from 'uuid';

const s3 = new S3Client({
  region: process.env.AWS_REGION || 'auto',
  endpoint: process.env.S3_ENDPOINT,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});

const BUCKET = process.env.S3_BUCKET!;
const CHUNK_SIZE = 5 * 1024 * 1024; // 5MB per part (S3 minimum)

interface MultipartSession {
  uploadId: string;
  key: string;
  parts: { PartNumber: number; ETag?: string }[];
}

export async function createMultipartSession(
  filename: string, 
  contentType: string, 
  folder = 'uploads'
): Promise<MultipartSession> {
  const extension = filename.split('.').pop();
  const key = `${folder}/${uuid()}.${extension}`;

  const command = new CreateMultipartUploadCommand({
    Bucket: BUCKET,
    Key: key,
    ContentType: contentType,
    Metadata: { 'original-filename': encodeURIComponent(filename) },
  });

  const response = await s3.send(command);
  if (!response.UploadId) throw new Error('Failed to initiate multipart upload');

  return { uploadId: response.UploadId, key, parts: [] };
}

export async function getPresignedPartUrl(
  key: string, 
  uploadId: string, 
  partNumber: number, 
  expiresIn = 3600
) {
  const command = new UploadPartCommand({ 
    Bucket: BUCKET, 
    Key: key, 
    PartNumber: partNumber, 
    UploadId: uploadId 
  });
  return getSignedUrl(s3, command, { expiresIn });
}

export async function completeMultipartUpload(session: MultipartSession) {
  const command = new CompleteMultipartUploadCommand({
    Bucket: BUCKET,
    Key: session.key,
    UploadId: session.uploadId,
    MultipartUpload: { 
      Parts: session.parts.map(p => ({ 
        PartNumber: p.PartNumber, 
        ETag: p.ETag! 
      })) 
    },
  });
  await s3.send(command);
}

export async function abortMultipartUpload(session: MultipartSession) {
  const command = new AbortMultipartUploadCommand({ 
    Bucket: BUCKET, 
    Key: session.key, 
    UploadId: session.uploadId 
  });
  await s3.send(command);
}

---

§ 3. IMAGE PROCESSING (RESPONSIVE / WEBP / AVIF)

typescript
// src/lib/image.ts
import sharp from 'sharp';
import path from 'path';

interface ProcessOptions {
  formats?: ('webp' | 'avif' | 'jpeg')[];
  widths?: number[];
  quality?: number;
}

export async function processImage(
  inputPath: string,
  outputFolder: string,
  options: ProcessOptions = {}
) {
  const {
    formats = ['webp', 'avif'],
    widths = [320, 640, 1280],
    quality = 80,
  } = options;

  const filename = path.basename(inputPath, path.extname(inputPath));
  const results: string[] = [];

  await Promise.all(
    formats.flatMap(format =>
      widths.map(async width => {
        const outputPath = path.join(outputFolder, `${filename}-${width}.${format}`);
        await sharp(inputPath)
          .resize(width)
          .toFormat(format, { quality })
          .toFile(outputPath);
        results.push(outputPath);
      })
    )
  );

  return results;
}

// Generate srcset for responsive images
export function generateSrcSet(
  baseUrl: string,
  filename: string,
  widths: number[] = [320, 640, 1280],
  format = 'webp'
) {
  return widths
    .map(w => `${baseUrl}/${filename}-${w}.${format} ${w}w`)
    .join(', ');
}

---

§ 4. SECURITY PATTERNS

| Pattern              | Implementation                                                               |
| -------------------- | ---------------------------------------------------------------------------- |
| File type validation | Validate `contentType` against allowed types before generating presigned URL |
| File size validation | Reject files above `MAX_FILE_SIZE` in API route                              |
| Signed URLs          | Use presigned URLs with short expiration (5–15 min)                          |
| Metadata tracking    | Store `original-filename`, `userId`, `timestamp` in S3 metadata              |
| Virus scanning       | Integrate ClamAV or Lambda function to scan after upload                     |
| Bucket policies      | Block public access, use CDN or signed URLs for delivery                     |
| HTTPS only           | Always serve presigned URLs via HTTPS / CDN                                  |
| Rate limiting        | Protect upload endpoints from abuse                                          |
| Multipart cleanup    | Abort incomplete multipart uploads after timeout                             |
| User isolation       | Separate uploads by user ID folder (`users/{userId}/...`)                    |

§ 4.1 FILE VALIDATION SCHEMA

typescript
import { z } from 'zod';

const ALLOWED_IMAGE_TYPES = ['image/jpeg', 'image/png', 'image/webp', 'image/gif'];
const ALLOWED_DOC_TYPES = ['application/pdf'];
const MAX_IMAGE_SIZE = 10 * 1024 * 1024; // 10MB
const MAX_DOC_SIZE = 50 * 1024 * 1024; // 50MB

export const imageUploadSchema = z.object({
  filename: z.string().min(1).max(255),
  contentType: z.enum(ALLOWED_IMAGE_TYPES as [string, ...string[]]),
  size: z.number().max(MAX_IMAGE_SIZE),
});

export const documentUploadSchema = z.object({
  filename: z.string().min(1).max(255),
  contentType: z.enum(ALLOWED_DOC_TYPES as [string, ...string[]]),
  size: z.number().max(MAX_DOC_SIZE),
});

---

§ 5. REACT COMPONENTS

§ 5.1 DROPZONE + UPLOAD PROGRESS

tsx
// src/components/FileUploader.tsx
'use client';
import { useState, useCallback } from 'react';
import { uploadFile } from '@/lib/upload';

interface UploadFile {
  file: File;
  progress: number;
  url?: string;
  error?: string;
}

export default function FileUploader() {
  const [files, setFiles] = useState<UploadFile[]>([]);
  const [isDragging, setIsDragging] = useState(false);

  const handleFiles = useCallback((selected: FileList | File[]) => {
    const fileArray = Array.from(selected).map(f => ({ file: f, progress: 0 }));
    setFiles(prev => [...prev, ...fileArray]);

    fileArray.forEach(async (f) => {
      try {
        const result = await uploadFile({
          file: f.file,
          onProgress: (p) => {
            setFiles(prev =>
              prev.map(item =>
                item.file === f.file ? { ...item, progress: p } : item
              )
            );
          },
        });
        setFiles(prev =>
          prev.map(item =>
            item.file === f.file ? { ...item, url: result.publicUrl } : item
          )
        );
      } catch (error: any) {
        setFiles(prev =>
          prev.map(item =>
            item.file === f.file ? { ...item, error: error.message } : item
          )
        );
      }
    });
  }, []);

  const handleDrop = useCallback((e: React.DragEvent) => {
    e.preventDefault();
    setIsDragging(false);
    if (e.dataTransfer.files) handleFiles(e.dataTransfer.files);
  }, [handleFiles]);

  return (
    <div
      onDragOver={(e) => { e.preventDefault(); setIsDragging(true); }}
      onDragLeave={() => setIsDragging(false)}
      onDrop={handleDrop}
      className={`border-2 border-dashed p-8 rounded-lg ${
        isDragging ? 'border-blue-500 bg-blue-50' : 'border-gray-300'
      }`}
    >
      <input
        type="file"
        multiple
        onChange={(e) => e.target.files && handleFiles(e.target.files)}
        className="hidden"
        id="file-input"
      />
      <label htmlFor="file-input" className="cursor-pointer">
        <p>Drag & drop files here or click to browse</p>
      </label>

      <ul className="mt-4 space-y-2">
        {files.map((f, idx) => (
          <li key={idx} className="flex items-center gap-2">
            <span>{f.file.name}</span>
            <span className="text-sm text-gray-500">{f.progress}%</span>
            {f.url && (
              <a href={f.url} target="_blank" className="text-blue-500">
                View
              </a>
            )}
            {f.error && <span className="text-red-500">{f.error}</span>}
          </li>
        ))}
      </ul>
    </div>
  );
}

§ 5.2 LARGE FILE UPLOADER (CHUNKED)

tsx
// src/components/LargeFileUploader.tsx
'use client';
import { useState } from 'react';
import {
  createMultipartSession,
  getPresignedPartUrl,
  completeMultipartUpload,
  abortMultipartUpload,
} from '@/lib/s3-multipart';

const CHUNK_SIZE = 5 * 1024 * 1024; // 5MB

interface ChunkFile {
  file: File;
  progress: number;
  url?: string;
  error?: string;
}

export default function LargeFileUploader() {
  const [files, setFiles] = useState<ChunkFile[]>([]);

  const handleFiles = async (selected: FileList) => {
    const fileArray = Array.from(selected).map(f => ({ file: f, progress: 0 }));
    setFiles(prev => [...prev, ...fileArray]);

    for (const f of fileArray) {
      let session;
      try {
        session = await createMultipartSession(f.file.name, f.file.type);
        const totalChunks = Math.ceil(f.file.size / CHUNK_SIZE);

        for (let i = 0; i < totalChunks; i++) {
          const start = i * CHUNK_SIZE;
          const end = Math.min(start + CHUNK_SIZE, f.file.size);
          const blob = f.file.slice(start, end);
          const url = await getPresignedPartUrl(session.key, session.uploadId, i + 1);

          const etag = await uploadChunk(url, blob, (chunkProgress) => {
            setFiles(prev =>
              prev.map(item =>
                item.file === f.file
                  ? { ...item, progress: Math.round(((i + chunkProgress / 100) / totalChunks) * 100) }
                  : item
              )
            );
          });

          session.parts.push({ PartNumber: i + 1, ETag: etag });
        }

        await completeMultipartUpload(session);
        setFiles(prev =>
          prev.map(item =>
            item.file === f.file ? { ...item, url: session.key, progress: 100 } : item
          )
        );
      } catch (error: any) {
        if (session) await abortMultipartUpload(session);
        setFiles(prev =>
          prev.map(item =>
            item.file === f.file ? { ...item, error: error.message } : item
          )
        );
      }
    }
  };

  return (
    <div>
      <input
        type="file"
        multiple
        onChange={(e) => e.target.files && handleFiles(e.target.files)}
      />
      <ul className="mt-4">
        {files.map((f, idx) => (
          <li key={idx}>
            {f.file.name} - {f.progress}%
            {f.url && <span className="text-green-500 ml-2">✓ Uploaded</span>}
            {f.error && <span className="text-red-500 ml-2">{f.error}</span>}
          </li>
        ))}
      </ul>
    </div>
  );
}

async function uploadChunk(
  url: string,
  chunk: Blob,
  onProgress?: (progress: number) => void
): Promise<string> {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.upload.onprogress = (e) => {
      if (e.lengthComputable && onProgress) {
        onProgress(Math.round((e.loaded / e.total) * 100));
      }
    };
    xhr.onload = () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        const etag = xhr.getResponseHeader('ETag') || '';
        resolve(etag);
      } else {
        reject(new Error(`Chunk upload failed: ${xhr.status}`));
      }
    };
    xhr.onerror = () => reject(new Error('Chunk upload error'));
    xhr.open('PUT', url);
    xhr.send(chunk);
  });
}

---

§ 6. CDN INTEGRATION

| Provider   | Setup                                        | Signed URLs | Edge Caching |
| ---------- | -------------------------------------------- | ----------- | ------------ |
| CloudFront | Origin = S3, OAI/OAC for private buckets     | ✅          | ✅           |
| Cloudflare | R2 native or proxy to S3                     | ✅          | ✅           |
| Bunny.net  | Pull zone from S3 origin                     | ✅          | ✅           |
| Vercel     | Edge middleware for auth + rewrite to S3/R2  | ⚠️         | ✅           |

§ 6.1 CLOUDFRONT SIGNED URL

typescript
import { getSignedUrl } from '@aws-sdk/cloudfront-signer';

export function getCloudFrontSignedUrl(key: string, expiresIn = 3600) {
  const url = `${process.env.CLOUDFRONT_URL}/${key}`;
  const dateLessThan = new Date(Date.now() + expiresIn * 1000).toISOString();

  return getSignedUrl({
    url,
    keyPairId: process.env.CLOUDFRONT_KEY_PAIR_ID!,
    privateKey: process.env.CLOUDFRONT_PRIVATE_KEY!,
    dateLessThan,
  });
}

---

§ 7. STORAGE COST COMPARISON

| Provider        | Storage (GB/mo) | PUT (per 1k) | GET (per 1k) | Egress (GB)  |
| --------------- | --------------- | ------------ | ------------ | ------------ |
| AWS S3 Standard | $0.023          | $0.005       | $0.0004      | $0.09        |
| AWS S3 IA       | $0.0125         | $0.01        | $0.001       | $0.09        |
| Cloudflare R2   | $0.015          | $0.0045      | Free         | **Free**     |
| Backblaze B2    | $0.006          | Free         | Free         | $0.01        |
| Supabase        | $0.021          | Included     | Included     | $0.09        |

---

§ 8. FILE UPLOAD CHECKLIST

□ File type validation (whitelist)
□ File size limits enforced
□ Presigned URLs with short expiration
□ HTTPS-only for all URLs
□ Virus scanning for user uploads
□ User isolation (folder per user)
□ Rate limiting on upload endpoints
□ Multipart cleanup job
□ CDN for delivery
□ Responsive image generation
□ Progress tracking UI
□ Error handling & retry logic
□ Lifecycle rules for temp files
□ Logging & audit trail

---

§ 9. ADDITIONAL TIPS

1. Use **Web Workers** for large file processing in browser
2. Generate **responsive image sizes** for each format and serve via `<picture>` tag
3. Combine **chunked uploads** + **presigned URLs** for secure, large file handling
4. Always **clean up incomplete multipart uploads** to avoid storage leaks
5. Enable **S3 lifecycle rules** to automatically expire temp files
6. Use **Next.js edge functions or middleware** for auth on presigned URLs
7. For multi-user apps, separate uploads by **user ID folder** to isolate permissions


§ 2. ADVANCED ERROR HANDLING

§ 2.1 CUSTOM ERROR TYPES

typescript
// src/lib/errors.ts
class UploadError extends Error {
  constructor(message: string, public code: string) {
    super(message);
    this.name = 'UploadError';
  }
}

class InvalidFileTypeError extends UploadError {
  constructor(message: string) {
    super(message, 'INVALID_FILE_TYPE');
  }
}

class FileTooLargeError extends UploadError {
  constructor(message: string) {
    super(message, 'FILE_TOO_LARGE');
  }
}

export { UploadError, InvalidFileTypeError, FileTooLargeError };

§ 2.2 ERROR HANDLING IN API ROUTE

typescript
// src/app/api/upload/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getUploadPresignedUrl } from '@/lib/s3';
import { z } from 'zod';
import { UploadError, InvalidFileTypeError, FileTooLargeError } from '@/lib/errors';

const uploadSchema = z.object({
  filename: z.string().min(1).max(255),
  contentType: z.string().regex(/^(image|video|application)\//),
  size: z.number().max(50 * 1024 * 1024), // 50MB max
});

const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/webp', 'application/pdf'];

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { filename, contentType, size } = uploadSchema.parse(body);

    if (!ALLOWED_TYPES.includes(contentType)) {
      throw new InvalidFileTypeError('File type not allowed');
    }

    if (size > 50 * 1024 * 1024) {
      throw new FileTooLargeError('File too large');
    }

    const { url, key } = await getUploadPresignedUrl(filename, contentType);
    return NextResponse.json({ uploadUrl: url, key });
  } catch (error) {
    if (error instanceof UploadError) {
      return NextResponse.json({ error: error.message }, { status: 400 });
    } else {
      return NextResponse.json({ error: 'Internal Server Error' }, { status: 500 });
    }
  }
}

§ 3. PERFORMANCE CONSIDERATIONS

§ 3.1 OPTIMIZING IMAGE UPLOADS

typescript
// src/lib/image-optimizer.ts
import { Sharp } from 'sharp';

export async function optimizeImage(imageBuffer: Buffer) {
  const sharp = Sharp(imageBuffer);
  const metadata = await sharp.metadata();

  if (metadata.format === 'jpeg' || metadata.format === 'png') {
    const optimizedImage = await sharp.resize(1024, 1024, {
      fit: 'inside',
      withoutEnlargement: true,
    }).jpeg({ quality: 80 }).toBuffer();
    return optimizedImage;
  } else {
    return imageBuffer;
  }
}

§ 3.2 CACHING PRESIGNED URLS

typescript
// src/lib/s3.ts
import { S3Client, PutObjectCommand, GetObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import { v4 as uuid } from 'uuid';
import { cache } from '@/lib/cache';

const s3 = new S3Client({
  region: process.env.AWS_REGION || 'auto',
  endpoint: process.env.S3_ENDPOINT,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});

const BUCKET = process.env.S3_BUCKET!;

export async function getUploadPresignedUrl(
  filename: string,
  contentType: string,
  folder = 'uploads',
  expiresIn = 900 // 15 minutes
) {
  const cacheKey = `presigned-url:${filename}:${contentType}:${folder}`;
  const cachedUrl = await cache.get(cacheKey);

  if (cachedUrl) {
    return cachedUrl;
  }

  const extension = filename.split('.').pop();
  const key = `${folder}/${uuid()}.${extension}`;

  const command = new PutObjectCommand({
    Bucket: BUCKET,
    Key: key,
    ContentType: contentType,
    Metadata: { 'original-filename': encodeURIComponent(filename) },
  });

  const url = await getSignedUrl(s3, command, { expiresIn });
  await cache.set(cacheKey, url, expiresIn);
  return url;
}

§ 4. TESTING PATTERNS (VITEST)

§ 4.1 TESTING UPLOAD API ROUTE

typescript
// src/app/api/upload/route.test.ts
import { describe, expect, it } from 'vitest';
import { NextRequest, NextResponse } from 'next/server';
import { uploadSchema } from '@/app/api/upload/route';
import { getUploadPresignedUrl } from '@/lib/s3';

describe('Upload API Route', () => {
  it('should return presigned URL for valid request', async () => {
    const request = new NextRequest('https://example.com/api/upload', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        filename: 'example.jpg',
        contentType: 'image/jpeg',
        size: 1024,
      }),
    });

    const response = await uploadSchema.parse(request);
    expect(response).toHaveProperty('uploadUrl');
    expect(response).toHaveProperty('key');
  });

  it('should return error for invalid request', async () => {
    const request = new NextRequest('https://example.com/api/upload', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        filename: '',
        contentType: '',
        size: 0,
      }),
    });

    const response = await uploadSchema.parse(request);
    expect(response).toHaveProperty('error');
  });
});

§ 5. BEST PRACTICES

§ 5.1 SECURITY BEST PRACTICES

| Best Practice | Description |
| --- | --- |
| ✅ Validate user input | Always validate user input to prevent security vulnerabilities |
| ❌ Use hardcoded credentials | Never use hardcoded credentials in your code |
| ✅ Use secure protocols | Always use secure protocols (HTTPS) for communication |
| ❌ Store sensitive data in plain text | Never store sensitive data in plain text |

§ 5.2 PERFORMANCE BEST PRACTICES

| Best Practice | Description |
| --- | --- |
| ✅ Optimize images | Always optimize images to reduce file size and improve performance |
| ❌ Use large images | Never use large images without optimizing them first |
| ✅ Use caching | Always use caching to reduce the number of requests to your API |
| ❌ Make unnecessary requests | Never make unnecessary requests to your API |

§ 6. COMMON PITFALLS & TROUBLESHOOTING

§ 6.1 COMMON PITFALLS

* Not validating user input
* Not using secure protocols
* Not optimizing images
* Not using caching

§ 6.2 TROUBLESHOOTING

* Check the API logs for errors
* Check the browser console for errors
* Use a debugger to step through the code
* Check the documentation for any known issues

§ 7. MIGRATION/UPGRADE PATTERNS

§ 7.1 UPGRADING TO A NEW VERSION OF A DEPENDENCY

typescript
// src/lib/upgrade.ts
import { exec } from 'child_process';

export async function upgradeDependency(dependency: string) {
  const command = `npm install ${dependency}@latest`;
  await exec(command);
}

§ 7.2 MIGRATING TO A NEW VERSION OF A LIBRARY

typescript
// src/lib/migrate.ts
import { migrate } from 'library-migration-tool';

export async function migrateLibrary(library: string) {
  await migrate(library);
}

10. NEXT.JS 14 FILE UPLOAD ROUTES

✅ Route handler POST /api/upload con streaming
✅ Server Actions per file upload (useFormData)
✅ Middleware per file size/type validation
✅ Edge Runtime vs Node.js Runtime per upload
✅ Codice completo: route handler con validation, S3 upload, response

typescript
Copia
Scarica
// app/api/upload/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { randomBytes } from 'crypto';
import { PutObjectCommand, S3Client } from '@aws-sdk/client-s3';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';
import { z } from 'zod';

// Runtime Configuration
export const runtime = 'nodejs'; // 'edge' for Edge Runtime
export const dynamic = 'force-dynamic';

// Validation Schema
const uploadSchema = z.object({
  file: z.instanceof(File).refine((file) => file.size <= 100 * 1024 * 1024, {
    message: 'File size must be less than 100MB',
  }).refine((file) => {
    const allowedTypes = ['image/jpeg', 'image/png', 'image/webp', 'application/pdf', 'text/plain'];
    return allowedTypes.includes(file.type);
  }, {
    message: 'File type not allowed',
  }),
  folder: z.string().optional().default('uploads'),
});

// Initialize S3 Client
const s3Client = new S3Client({
  region: process.env.AWS_REGION!,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
  // ⚠️ For Cloudflare R2, use:
  // endpoint: `https://${process.env.CLOUDFLARE_ACCOUNT_ID}.r2.cloudflarestorage.com`,
});

export async function POST(request: NextRequest) {
  try {
    // 1. Authentication
    const session = await getServerSession(authOptions);
    if (!session?.user?.id) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    // 2. Parse FormData
    const formData = await request.formData();
    const file = formData.get('file') as File;
    const folder = formData.get('folder') as string || 'uploads';

    // 3. Validate Input
    const validationResult = uploadSchema.safeParse({ file, folder });
    if (!validationResult.success) {
      return NextResponse.json(
        { error: 'Validation failed', details: validationResult.error.errors },
        { status: 400 }
      );
    }

    // 4. Generate Unique Filename
    const fileExtension = file.name.split('.').pop();
    const uniqueSuffix = `${Date.now()}-${randomBytes(8).toString('hex')}`;
    const key = `${session.user.id}/${folder}/${uniqueSuffix}.${fileExtension}`;

    // 5. Convert File to Buffer (Streaming in production)
    const bytes = await file.arrayBuffer();
    const buffer = Buffer.from(bytes);

    // 6. Upload to S3
    const uploadCommand = new PutObjectCommand({
      Bucket: process.env.S3_BUCKET_NAME!,
      Key: key,
      Body: buffer,
      ContentType: file.type,
      ContentLength: file.size,
      Metadata: {
        'uploaded-by': session.user.id,
        'original-filename': encodeURIComponent(file.name),
      },
    });

    await s3Client.send(uploadCommand);

    // 7. Construct Public URL (via CDN)
    const publicUrl = `https://${process.env.CDN_DOMAIN}/${key}`;

    // 8. Return Success Response
    return NextResponse.json({
      success: true,
      data: {
        id: uniqueSuffix,
        key,
        url: publicUrl,
        name: file.name,
        size: file.size,
        type: file.type,
        uploadedAt: new Date().toISOString(),
      },
    }, { status: 201 });

  } catch (error) {
    console.error('Upload error:', error);
    
    if (error instanceof z.ZodError) {
      return NextResponse.json({ error: 'Validation failed', details: error.errors }, { status: 400 });
    }

    if ((error as any).name === 'PayloadTooLargeError') {
      return NextResponse.json({ error: 'File too large' }, { status: 413 });
    }

    return NextResponse.json(
      { error: 'Upload failed', message: (error as Error).message },
      { status: 500 }
    );
  }
}

// GET route for upload status/listing
export async function GET(request: NextRequest) {
  const session = await getServerSession(authOptions);
  if (!session?.user?.id) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const searchParams = request.nextUrl.searchParams;
  const folder = searchParams.get('folder') || 'uploads';
  
  // In production, query database for user's files
  return NextResponse.json({
    files: [],
    folder,
    pagination: { limit: 50, offset: 0 }
  });
}
typescript
Copia
Scarica
// app/api/upload/server-action/route.ts
import { createUploadthing, type FileRouter } from 'uploadthing/next';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';

const f = createUploadthing();

export const uploadRouter = {
  imageUploader: f({
    image: { maxFileSize: '4MB', maxFileCount: 10 },
    pdf: { maxFileSize: '16MB', maxFileCount: 5 },
  })
    .middleware(async () => {
      const session = await getServerSession(authOptions);
      if (!session?.user?.id) throw new Error('Unauthorized');
      return { userId: session.user.id };
    })
    .onUploadComplete(async ({ metadata, file }) => {
      console.log('Upload complete for userId:', metadata.userId);
      console.log('File URL:', file.url);
      return { uploadedBy: metadata.userId };
    }),
} satisfies FileRouter;

export type OurFileRouter = typeof uploadRouter;
typescript
Copia
Scarica
// app/api/upload/middleware/validate.ts
import { NextRequest, NextResponse } from 'next/server';

export async function uploadValidationMiddleware(
  request: NextRequest,
  next: () => Promise<NextResponse>
) {
  // Check content type
  const contentType = request.headers.get('content-type') || '';
  if (!contentType.includes('multipart/form-data')) {
    return NextResponse.json(
      { error: 'Content-Type must be multipart/form-data' },
      { status: 415 }
    );
  }

  // Check content length
  const contentLength = parseInt(request.headers.get('content-length') || '0', 10);
  const MAX_UPLOAD_SIZE = 100 * 1024 * 1024; // 100MB
  if (contentLength > MAX_UPLOAD_SIZE) {
    return NextResponse.json(
      { error: `File too large. Maximum size is ${MAX_UPLOAD_SIZE / 1024 / 1024}MB` },
      { status: 413 }
    );
  }

  // Add custom headers
  const requestHeaders = new Headers(request.headers);
  requestHeaders.set('x-upload-start-time', Date.now().toString());

  return next();
}
typescript
Copia
Scarica
// lib/server-actions/upload.ts
'use server';

import { revalidatePath } from 'next/cache';
import { z } from 'zod';
import { put } from '@vercel/blob';
import { writeFile } from 'fs/promises';
import { join } from 'path';

const uploadSchema = z.object({
  file: z.instanceof(File),
  description: z.string().optional(),
});

export async function uploadFile(formData: FormData) {
  try {
    // Parse and validate
    const file = formData.get('file') as File;
    const description = formData.get('description') as string;
    
    const validated = uploadSchema.parse({ file, description });

    // Option 1: Upload to Vercel Blob
    const blob = await put(validated.file.name, validated.file, {
      access: 'public',
      addRandomSuffix: true,
    });

    // Option 2: Save to local filesystem (for development)
    // const bytes = await validated.file.arrayBuffer();
    // const buffer = Buffer.from(bytes);
    // const path = join(process.cwd(), 'public/uploads', validated.file.name);
    // await writeFile(path, buffer);

    // Store metadata in database (prisma call here)

    revalidatePath('/dashboard/files');
    return { success: true, url: blob.url };
  } catch (error) {
    console.error('Server action upload error:', error);
    return { success: false, error: 'Upload failed' };
  }
}

RUNTIME DECISION TABLE:

Feature	Node.js Runtime ✅	Edge Runtime ✅	Recommendation
File Size	> 4MB	≤ 4MB	Node.js for large files
FormData	Full support	Limited (4MB)	Edge for small files
Stream Processing	Excellent	Basic	Node.js for streaming
Cold Start	Slower	Faster	Edge for frequent small uploads
Memory	3000MB+	256MB	Node.js for memory-intensive ops
Library Support	All npm packages	Limited	Node.js for complex processing
Use Case	Video processing, PDF generation	Image uploads, avatars	Choose based on file size
11. DRAG & DROP UPLOAD UI

✅ React component completo con react-dropzone
✅ Paste from clipboard support
✅ File preview (image thumbnail, PDF first page, generic icon)
✅ Multiple file selection con batch upload
✅ Accessible drag & drop (keyboard, screen reader)
✅ Codice completo: DropZone component 80+ righe con tutti gli stati

typescript
Copia
Scarica
// components/upload/DropZone.tsx
'use client';

import React, { useCallback, useState, useRef } from 'react';
import { useDropzone } from 'react-dropzone';
import { Upload, X, FileText, Image as ImageIcon, File, AlertCircle, Check } from 'lucide-react';
import Image from 'next/image';
import { uploadFile } from '@/lib/server-actions/upload';
import { cn } from '@/lib/utils';

interface UploadFile extends File {
  preview?: string;
  id: string;
  progress: number;
  status: 'pending' | 'uploading' | 'success' | 'error';
  error?: string;
}

interface DropZoneProps {
  maxFiles?: number;
  maxSize?: number;
  accept?: Record<string, string[]>;
  onUploadComplete?: (files: UploadFile[]) => void;
  className?: string;
}

export default function DropZone({
  maxFiles = 10,
  maxSize = 100 * 1024 * 1024, // 100MB
  accept = {
    'image/*': ['.jpg', '.jpeg', '.png', '.gif', '.webp'],
    'application/pdf': ['.pdf'],
    'text/*': ['.txt', '.csv'],
    'video/*': ['.mp4', '.mov', '.avi'],
  },
  onUploadComplete,
  className,
}: DropZoneProps) {
  const [files, setFiles] = useState<UploadFile[]>([]);
  const [isDragging, setIsDragging] = useState(false);
  const [isUploading, setIsUploading] = useState(false);
  const fileInputRef = useRef<HTMLInputElement>(null);

  const onDrop = useCallback((acceptedFiles: File[], rejectedFiles: any[]) => {
    // Handle accepted files
    const newFiles: UploadFile[] = acceptedFiles.map(file => ({
      ...file,
      id: `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
      preview: file.type.startsWith('image/') ? URL.createObjectURL(file) : undefined,
      progress: 0,
      status: 'pending',
    }));

    setFiles(prev => {
      const updated = [...prev, ...newFiles].slice(0, maxFiles);
      return updated;
    });

    // Handle rejected files
    if (rejectedFiles.length > 0) {
      rejectedFiles.forEach(({ file, errors }) => {
        console.error(`Rejected ${file.name}:`, errors);
        // Show toast notification
      });
    }
  }, [maxFiles]);

  const {
    getRootProps,
    getInputProps,
    isDragActive,
    open,
  } = useDropzone({
    onDrop,
    maxFiles,
    maxSize,
    accept,
    noClick: true, // We'll handle click via button
    disabled: isUploading || files.length >= maxFiles,
  });

  const handlePaste = useCallback((event: React.ClipboardEvent) => {
    const items = event.clipboardData?.items;
    if (!items) return;

    const imageFiles: File[] = [];
    
    for (const item of items) {
      if (item.type.startsWith('image/')) {
        const file = item.getAsFile();
        if (file) {
          imageFiles.push(file);
        }
      }
    }

    if (imageFiles.length > 0) {
      event.preventDefault();
      onDrop(imageFiles, []);
    }
  }, [onDrop]);

  const removeFile = (id: string) => {
    setFiles(prev => {
      const file = prev.find(f => f.id === id);
      if (file?.preview) {
        URL.revokeObjectURL(file.preview);
      }
      return prev.filter(f => f.id !== id);
    });
  };

  const uploadFiles = async () => {
    if (files.length === 0) return;

    setIsUploading(true);
    const updatedFiles = [...files];

    for (let i = 0; i < updatedFiles.length; i++) {
      const file = updatedFiles[i];
      if (file.status === 'pending') {
        try {
          // Update status
          updatedFiles[i] = { ...file, status: 'uploading', progress: 10 };
          setFiles([...updatedFiles]);

          // Simulate progress
          const interval = setInterval(() => {
            setFiles(prev => {
              const newFiles = [...prev];
              const currentFile = newFiles[i];
              if (currentFile && currentFile.progress < 90) {
                newFiles[i] = { 
                  ...currentFile, 
                  progress: currentFile.progress + 20 
                };
              }
              return newFiles;
            });
          }, 200);

          // Actual upload
          const formData = new FormData();
          formData.append('file', file);

          const result = await uploadFile(formData);

          clearInterval(interval);

          if (result.success) {
            updatedFiles[i] = { 
              ...file, 
              status: 'success', 
              progress: 100 
            };
          } else {
            throw new Error(result.error);
          }
        } catch (error) {
          updatedFiles[i] = { 
            ...file, 
            status: 'error', 
            error: (error as Error).message,
            progress: 0 
          };
        } finally {
          setFiles([...updatedFiles]);
        }
      }
    }

    setIsUploading(false);
    onUploadComplete?.(updatedFiles);
  };

  const clearAll = () => {
    files.forEach(file => {
      if (file.preview) {
        URL.revokeObjectURL(file.preview);
      }
    });
    setFiles([]);
  };

  const getFileIcon = (file: UploadFile) => {
    if (file.type.startsWith('image/')) {
      return <ImageIcon className="h-5 w-5 text-blue-500" />;
    }
    if (file.type === 'application/pdf') {
      return <FileText className="h-5 w-5 text-red-500" />;
    }
    return <File className="h-5 w-5 text-gray-500" />;
  };

  return (
    <div 
      className={cn('w-full', className)}
      onPaste={handlePaste}
      onDragEnter={() => setIsDragging(true)}
      onDragLeave={() => setIsDragging(false)}
    >
      {/* Drop Zone */}
      <div
        {...getRootProps()}
        className={cn(
          'border-2 border-dashed rounded-xl p-8 text-center transition-all duration-200',
          'focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2',
          isDragActive || isDragging
            ? 'border-blue-500 bg-blue-50'
            : 'border-gray-300 hover:border-gray-400',
          (isUploading || files.length >= maxFiles) && 'opacity-50 cursor-not-allowed'
        )}
        role="button"
        tabIndex={0}
        aria-label="Drag and drop area for file upload"
      >
        <input {...getInputProps()} ref={fileInputRef} />
        
        <div className="flex flex-col items-center justify-center space-y-4">
          <div className="p-3 bg-blue-100 rounded-full">
            <Upload className="h-8 w-8 text-blue-600" />
          </div>
          
          <div className="space-y-2">
            <h3 className="text-lg font-semibold text-gray-900">
              {isDragActive ? 'Drop files here' : 'Drag & drop files'}
            </h3>
            <p className="text-sm text-gray-500">
              or{' '}
              <button
                type="button"
                onClick={open}
                disabled={isUploading || files.length >= maxFiles}
                className="text-blue-600 hover:text-blue-800 font-medium disabled:text-gray-400 disabled:cursor-not-allowed"
              >
                browse
              </button>{' '}
              from your computer
            </p>
            <p className="text-xs text-gray-400">
              Supports images, PDFs, documents up to {maxSize / 1024 / 1024}MB each
            </p>
            <p className="text-xs text-gray-400">
              Press Ctrl+V to paste images from clipboard
            </p>
          </div>
        </div>
      </div>

      {/* File List */}
      {files.length > 0 && (
        <div className="mt-6 space-y-4">
          <div className="flex justify-between items-center">
            <h4 className="font-medium text-gray-900">
              Selected Files ({files.length}/{maxFiles})
            </h4>
            <button
              type="button"
              onClick={clearAll}
              className="text-sm text-gray-500 hover:text-gray-700"
            >
              Clear all
            </button>
          </div>

          <div className="space-y-3">
            {files.map((file, index) => (
              <div
                key={file.id}
                className="flex items-center justify-between p-4 bg-gray-50 rounded-lg border"
              >
                <div className="flex items-center space-x-4 flex-1 min-w-0">
                  {/* Preview */}
                  {file.preview ? (
                    <div className="relative h-12 w-12 flex-shrink-0">
                      <Image
                        src={file.preview}
                        alt={file.name}
                        fill
                        className="object-cover rounded"
                        sizes="48px"
                      />
                    </div>
                  ) : (
                    <div className="h-12 w-12 flex items-center justify-center bg-white border rounded">
                      {getFileIcon(file)}
                    </div>
                  )}

                  {/* File Info */}
                  <div className="flex-1 min-w-0">
                    <div className="flex items-center justify-between">
                      <p className="font-medium text-sm truncate" title={file.name}>
                        {file.name}
                      </p>
                      <span className="text-xs text-gray-500">
                        {(file.size / 1024 / 1024).toFixed(2)} MB
                      </span>
                    </div>

                    {/* Progress Bar

### FILE UPLOAD STORAGE - Advanced Implementation Pattern #1

```typescript
// lib/file-upload-storage/pattern-1.ts
import { z } from "zod";

interface ServiceConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  batchSize: number;
  debug: boolean;
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
  batchSize: z.number().min(1).max(1000).default(50),
  debug: z.boolean().default(false),
});

export class FILEUPLOADSTORAGEService1 {
  private config: ServiceConfig;
  private cache: Map<string, { data: unknown; expiresAt: number }> = new Map();
  private metrics = { operations: 0, errors: 0, avgDuration: 0 };

  constructor(config: Partial<ServiceConfig> = {}) {
    this.config = ConfigSchema.parse(config) as ServiceConfig;
  }

  async execute<TInput, TOutput>(
    operation: string,
    input: TInput,
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    this.metrics.operations++;
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);
        const result = await handler(input);
        clearTimeout(timeoutId);

        const duration = Date.now() - startTime;
        this.metrics.avgDuration =
          (this.metrics.avgDuration * (this.metrics.operations - 1) + duration) /
          this.metrics.operations;

        return {
          success: true,
          data: result,
          duration,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        this.metrics.errors++;

        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Operation failed",
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

  async executeBatch<TInput, TOutput>(
    operation: string,
    inputs: TInput[],
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput[]>> {
    const startTime = Date.now();
    const results: TOutput[] = [];
    const errors: string[] = [];

    for (let i = 0; i < inputs.length; i += this.config.batchSize) {
      const batch = inputs.slice(i, i + this.config.batchSize);
      const batchResults = await Promise.allSettled(
        batch.map((input) => this.execute(operation, input, handler))
      );

      for (const result of batchResults) {
        if (result.status === "fulfilled" && result.value.success && result.value.data) {
          results.push(result.value.data);
        } else if (result.status === "rejected") {
          errors.push(result.reason?.message || "Batch item failed");
        }
      }
    }

    return {
      success: errors.length === 0,
      data: results,
      error: errors.length > 0 ? errors.length + " items failed" : undefined,
      duration: Date.now() - startTime,
      retries: 0,
      timestamp: new Date(),
    };
  }

  getCachedOrFetch<T>(key: string, fetcher: () => Promise<T>, ttlMs: number = 60000): Promise<T> {
    const cached = this.cache.get(key);
    if (cached && Date.now() < cached.expiresAt) {
      return Promise.resolve(cached.data as T);
    }
    return fetcher().then((data) => {
      this.cache.set(key, { data, expiresAt: Date.now() + ttlMs });
      return data;
    });
  }

  getMetrics() {
    return {
      ...this.metrics,
      errorRate: this.metrics.operations > 0
        ? ((this.metrics.errors / this.metrics.operations) * 100).toFixed(2) + "%"
        : "0%",
      cacheSize: this.cache.size,
    };
  }
}
```

```typescript
// components/file-upload-storage/Manager1.tsx
"use client";

import { useState, useEffect, useCallback, useMemo } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import { Loader2, Plus, Search, RefreshCw, Trash2, Edit, ChevronLeft, ChevronRight } from "lucide-react";

interface Item {
  id: string;
  name: string;
  status: "active" | "inactive" | "pending";
  category: string;
  createdAt: string;
  updatedAt: string;
}

interface ManagerProps {
  initialItems?: Item[];
  apiEndpoint?: string;
  pageSize?: number;
}

export function Manager1({
  initialItems = [],
  apiEndpoint = "/api/items",
  pageSize = 10,
}: ManagerProps) {
  const [items, setItems] = useState<Item[]>(initialItems);
  const [loading, setLoading] = useState(false);
  const [search, setSearch] = useState("");
  const [statusFilter, setStatusFilter] = useState<string>("all");
  const [page, setPage] = useState(1);

  const fetchItems = useCallback(async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams({ page: String(page), limit: String(pageSize) });
      if (search) params.set("search", search);
      if (statusFilter !== "all") params.set("status", statusFilter);

      const response = await fetch(apiEndpoint + "?" + params);
      if (!response.ok) throw new Error("Failed to fetch");
      const data = await response.json();
      setItems(data.items || data.data || []);
    } catch (error) {
      console.error("Fetch error:", error);
    } finally {
      setLoading(false);
    }
  }, [apiEndpoint, page, pageSize, search, statusFilter]);

  useEffect(() => { fetchItems(); }, [fetchItems]);

  const filteredItems = useMemo(() => {
    return items.filter((item) => {
      const matchesSearch = !search || item.name.toLowerCase().includes(search.toLowerCase());
      const matchesStatus = statusFilter === "all" || item.status === statusFilter;
      return matchesSearch && matchesStatus;
    });
  }, [items, search, statusFilter]);

  const paginatedItems = filteredItems.slice((page - 1) * pageSize, page * pageSize);
  const totalPages = Math.ceil(filteredItems.length / pageSize);

  const handleDelete = useCallback(async (id: string) => {
    try {
      const response = await fetch(apiEndpoint + "/" + id, { method: "DELETE" });
      if (!response.ok) throw new Error("Delete failed");
      setItems((prev) => prev.filter((item) => item.id !== id));
    } catch (error) {
      console.error("Delete error:", error);
    }
  }, [apiEndpoint]);

  const statusColors: Record<string, string> = {
    active: "bg-green-100 text-green-800",
    inactive: "bg-gray-100 text-gray-800",
    pending: "bg-yellow-100 text-yellow-800",
  };

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle>Item Manager</CardTitle>
          <div className="flex items-center gap-2">
            <Button variant="outline" size="sm" onClick={fetchItems} disabled={loading}>
              <RefreshCw className={"h-4 w-4 " + (loading ? "animate-spin" : "")} />
            </Button>
            <Button size="sm"><Plus className="h-4 w-4 mr-1" /> Add New</Button>
          </div>
        </div>
        <div className="flex gap-2 mt-4">
          <div className="relative flex-1">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
            <Input
              placeholder="Search..."
              value={search}
              onChange={(e) => { setSearch(e.target.value); setPage(1); }}
              className="pl-9"
            />
          </div>
          <select
            value={statusFilter}
            onChange={(e) => { setStatusFilter(e.target.value); setPage(1); }}
            className="border rounded px-3 py-2 text-sm"
          >
            <option value="all">All Status</option>
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
            <option value="pending">Pending</option>
          </select>
        </div>
      </CardHeader>
      <CardContent>
        {loading ? (
          <div className="flex items-center justify-center py-12">
            <Loader2 className="h-6 w-6 animate-spin text-muted-foreground" />
          </div>
        ) : paginatedItems.length === 0 ? (
          <p className="text-center py-12 text-muted-foreground">No items found</p>
        ) : (
          <div className="space-y-2">
            {paginatedItems.map((item) => (
              <div key={item.id} className="flex items-center justify-between p-3 border rounded-lg hover:bg-muted/50 transition-colors">
                <div>
                  <p className="font-medium">{item.name}</p>
                  <p className="text-xs text-muted-foreground">
                    {item.category} - {new Date(item.createdAt).toLocaleDateString()}
                  </p>
                </div>
                <div className="flex items-center gap-2">
                  <Badge className={statusColors[item.status] || ""}>{item.status}</Badge>
                  <Button variant="ghost" size="sm"><Edit className="h-4 w-4" /></Button>
                  <Button variant="ghost" size="sm" onClick={() => handleDelete(item.id)}>
                    <Trash2 className="h-4 w-4 text-red-500" />
                  </Button>
                </div>
              </div>
            ))}
          </div>
        )}

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4 pt-4 border-t">
            <p className="text-sm text-muted-foreground">Page {page} of {totalPages}</p>
            <div className="flex gap-1">
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.max(1, p - 1))} disabled={page === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.min(totalPages, p + 1))} disabled={page === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```


### FILE UPLOAD STORAGE - Advanced Implementation Pattern #2

```typescript
// lib/file-upload-storage/pattern-2.ts
import { z } from "zod";

interface ServiceConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  batchSize: number;
  debug: boolean;
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
  batchSize: z.number().min(1).max(1000).default(50),
  debug: z.boolean().default(false),
});

export class FILEUPLOADSTORAGEService2 {
  private config: ServiceConfig;
  private cache: Map<string, { data: unknown; expiresAt: number }> = new Map();
  private metrics = { operations: 0, errors: 0, avgDuration: 0 };

  constructor(config: Partial<ServiceConfig> = {}) {
    this.config = ConfigSchema.parse(config) as ServiceConfig;
  }

  async execute<TInput, TOutput>(
    operation: string,
    input: TInput,
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    this.metrics.operations++;
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);
        const result = await handler(input);
        clearTimeout(timeoutId);

        const duration = Date.now() - startTime;
        this.metrics.avgDuration =
          (this.metrics.avgDuration * (this.metrics.operations - 1) + duration) /
          this.metrics.operations;

        return {
          success: true,
          data: result,
          duration,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        this.metrics.errors++;

        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Operation failed",
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

  async executeBatch<TInput, TOutput>(
    operation: string,
    inputs: TInput[],
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput[]>> {
    const startTime = Date.now();
    const results: TOutput[] = [];
    const errors: string[] = [];

    for (let i = 0; i < inputs.length; i += this.config.batchSize) {
      const batch = inputs.slice(i, i + this.config.batchSize);
      const batchResults = await Promise.allSettled(
        batch.map((input) => this.execute(operation, input, handler))
      );

      for (const result of batchResults) {
        if (result.status === "fulfilled" && result.value.success && result.value.data) {
          results.push(result.value.data);
        } else if (result.status === "rejected") {
          errors.push(result.reason?.message || "Batch item failed");
        }
      }
    }

    return {
      success: errors.length === 0,
      data: results,
      error: errors.length > 0 ? errors.length + " items failed" : undefined,
      duration: Date.now() - startTime,
      retries: 0,
      timestamp: new Date(),
    };
  }

  getCachedOrFetch<T>(key: string, fetcher: () => Promise<T>, ttlMs: number = 60000): Promise<T> {
    const cached = this.cache.get(key);
    if (cached && Date.now() < cached.expiresAt) {
      return Promise.resolve(cached.data as T);
    }
    return fetcher().then((data) => {
      this.cache.set(key, { data, expiresAt: Date.now() + ttlMs });
      return data;
    });
  }

  getMetrics() {
    return {
      ...this.metrics,
      errorRate: this.metrics.operations > 0
        ? ((this.metrics.errors / this.metrics.operations) * 100).toFixed(2) + "%"
        : "0%",
      cacheSize: this.cache.size,
    };
  }
}
```

```typescript
// components/file-upload-storage/Manager2.tsx
"use client";

import { useState, useEffect, useCallback, useMemo } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import { Loader2, Plus, Search, RefreshCw, Trash2, Edit, ChevronLeft, ChevronRight } from "lucide-react";

interface Item {
  id: string;
  name: string;
  status: "active" | "inactive" | "pending";
  category: string;
  createdAt: string;
  updatedAt: string;
}

interface ManagerProps {
  initialItems?: Item[];
  apiEndpoint?: string;
  pageSize?: number;
}

export function Manager2({
  initialItems = [],
  apiEndpoint = "/api/items",
  pageSize = 10,
}: ManagerProps) {
  const [items, setItems] = useState<Item[]>(initialItems);
  const [loading, setLoading] = useState(false);
  const [search, setSearch] = useState("");
  const [statusFilter, setStatusFilter] = useState<string>("all");
  const [page, setPage] = useState(1);

  const fetchItems = useCallback(async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams({ page: String(page), limit: String(pageSize) });
      if (search) params.set("search", search);
      if (statusFilter !== "all") params.set("status", statusFilter);

      const response = await fetch(apiEndpoint + "?" + params);
      if (!response.ok) throw new Error("Failed to fetch");
      const data = await response.json();
      setItems(data.items || data.data || []);
    } catch (error) {
      console.error("Fetch error:", error);
    } finally {
      setLoading(false);
    }
  }, [apiEndpoint, page, pageSize, search, statusFilter]);

  useEffect(() => { fetchItems(); }, [fetchItems]);

  const filteredItems = useMemo(() => {
    return items.filter((item) => {
      const matchesSearch = !search || item.name.toLowerCase().includes(search.toLowerCase());
      const matchesStatus = statusFilter === "all" || item.status === statusFilter;
      return matchesSearch && matchesStatus;
    });
  }, [items, search, statusFilter]);

  const paginatedItems = filteredItems.slice((page - 1) * pageSize, page * pageSize);
  const totalPages = Math.ceil(filteredItems.length / pageSize);

  const handleDelete = useCallback(async (id: string) => {
    try {
      const response = await fetch(apiEndpoint + "/" + id, { method: "DELETE" });
      if (!response.ok) throw new Error("Delete failed");
      setItems((prev) => prev.filter((item) => item.id !== id));
    } catch (error) {
      console.error("Delete error:", error);
    }
  }, [apiEndpoint]);

  const statusColors: Record<string, string> = {
    active: "bg-green-100 text-green-800",
    inactive: "bg-gray-100 text-gray-800",
    pending: "bg-yellow-100 text-yellow-800",
  };

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle>Item Manager</CardTitle>
          <div className="flex items-center gap-2">
            <Button variant="outline" size="sm" onClick={fetchItems} disabled={loading}>
              <RefreshCw className={"h-4 w-4 " + (loading ? "animate-spin" : "")} />
            </Button>
            <Button size="sm"><Plus className="h-4 w-4 mr-1" /> Add New</Button>
          </div>
        </div>
        <div className="flex gap-2 mt-4">
          <div className="relative flex-1">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
            <Input
              placeholder="Search..."
              value={search}
              onChange={(e) => { setSearch(e.target.value); setPage(1); }}
              className="pl-9"
            />
          </div>
          <select
            value={statusFilter}
            onChange={(e) => { setStatusFilter(e.target.value); setPage(1); }}
            className="border rounded px-3 py-2 text-sm"
          >
            <option value="all">All Status</option>
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
            <option value="pending">Pending</option>
          </select>
        </div>
      </CardHeader>
      <CardContent>
        {loading ? (
          <div className="flex items-center justify-center py-12">
            <Loader2 className="h-6 w-6 animate-spin text-muted-foreground" />
          </div>
        ) : paginatedItems.length === 0 ? (
          <p className="text-center py-12 text-muted-foreground">No items found</p>
        ) : (
          <div className="space-y-2">
            {paginatedItems.map((item) => (
              <div key={item.id} className="flex items-center justify-between p-3 border rounded-lg hover:bg-muted/50 transition-colors">
                <div>
                  <p className="font-medium">{item.name}</p>
                  <p className="text-xs text-muted-foreground">
                    {item.category} - {new Date(item.createdAt).toLocaleDateString()}
                  </p>
                </div>
                <div className="flex items-center gap-2">
                  <Badge className={statusColors[item.status] || ""}>{item.status}</Badge>
                  <Button variant="ghost" size="sm"><Edit className="h-4 w-4" /></Button>
                  <Button variant="ghost" size="sm" onClick={() => handleDelete(item.id)}>
                    <Trash2 className="h-4 w-4 text-red-500" />
                  </Button>
                </div>
              </div>
            ))}
          </div>
        )}

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4 pt-4 border-t">
            <p className="text-sm text-muted-foreground">Page {page} of {totalPages}</p>
            <div className="flex gap-1">
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.max(1, p - 1))} disabled={page === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.min(totalPages, p + 1))} disabled={page === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```


### FILE UPLOAD STORAGE - Advanced Implementation Pattern #3

```typescript
// lib/file-upload-storage/pattern-3.ts
import { z } from "zod";

interface ServiceConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  batchSize: number;
  debug: boolean;
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
  batchSize: z.number().min(1).max(1000).default(50),
  debug: z.boolean().default(false),
});

export class FILEUPLOADSTORAGEService3 {
  private config: ServiceConfig;
  private cache: Map<string, { data: unknown; expiresAt: number }> = new Map();
  private metrics = { operations: 0, errors: 0, avgDuration: 0 };

  constructor(config: Partial<ServiceConfig> = {}) {
    this.config = ConfigSchema.parse(config) as ServiceConfig;
  }

  async execute<TInput, TOutput>(
    operation: string,
    input: TInput,
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    this.metrics.operations++;
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);
        const result = await handler(input);
        clearTimeout(timeoutId);

        const duration = Date.now() - startTime;
        this.metrics.avgDuration =
          (this.metrics.avgDuration * (this.metrics.operations - 1) + duration) /
          this.metrics.operations;

        return {
          success: true,
          data: result,
          duration,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        this.metrics.errors++;

        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Operation failed",
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

  async executeBatch<TInput, TOutput>(
    operation: string,
    inputs: TInput[],
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput[]>> {
    const startTime = Date.now();
    const results: TOutput[] = [];
    const errors: string[] = [];

    for (let i = 0; i < inputs.length; i += this.config.batchSize) {
      const batch = inputs.slice(i, i + this.config.batchSize);
      const batchResults = await Promise.allSettled(
        batch.map((input) => this.execute(operation, input, handler))
      );

      for (const result of batchResults) {
        if (result.status === "fulfilled" && result.value.success && result.value.data) {
          results.push(result.value.data);
        } else if (result.status === "rejected") {
          errors.push(result.reason?.message || "Batch item failed");
        }
      }
    }

    return {
      success: errors.length === 0,
      data: results,
      error: errors.length > 0 ? errors.length + " items failed" : undefined,
      duration: Date.now() - startTime,
      retries: 0,
      timestamp: new Date(),
    };
  }

  getCachedOrFetch<T>(key: string, fetcher: () => Promise<T>, ttlMs: number = 60000): Promise<T> {
    const cached = this.cache.get(key);
    if (cached && Date.now() < cached.expiresAt) {
      return Promise.resolve(cached.data as T);
    }
    return fetcher().then((data) => {
      this.cache.set(key, { data, expiresAt: Date.now() + ttlMs });
      return data;
    });
  }

  getMetrics() {
    return {
      ...this.metrics,
      errorRate: this.metrics.operations > 0
        ? ((this.metrics.errors / this.metrics.operations) * 100).toFixed(2) + "%"
        : "0%",
      cacheSize: this.cache.size,
    };
  }
}
```

```typescript
// components/file-upload-storage/Manager3.tsx
"use client";

import { useState, useEffect, useCallback, useMemo } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import { Loader2, Plus, Search, RefreshCw, Trash2, Edit, ChevronLeft, ChevronRight } from "lucide-react";

interface Item {
  id: string;
  name: string;
  status: "active" | "inactive" | "pending";
  category: string;
  createdAt: string;
  updatedAt: string;
}

interface ManagerProps {
  initialItems?: Item[];
  apiEndpoint?: string;
  pageSize?: number;
}

export function Manager3({
  initialItems = [],
  apiEndpoint = "/api/items",
  pageSize = 10,
}: ManagerProps) {
  const [items, setItems] = useState<Item[]>(initialItems);
  const [loading, setLoading] = useState(false);
  const [search, setSearch] = useState("");
  const [statusFilter, setStatusFilter] = useState<string>("all");
  const [page, setPage] = useState(1);

  const fetchItems = useCallback(async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams({ page: String(page), limit: String(pageSize) });
      if (search) params.set("search", search);
      if (statusFilter !== "all") params.set("status", statusFilter);

      const response = await fetch(apiEndpoint + "?" + params);
      if (!response.ok) throw new Error("Failed to fetch");
      const data = await response.json();
      setItems(data.items || data.data || []);
    } catch (error) {
      console.error("Fetch error:", error);
    } finally {
      setLoading(false);
    }
  }, [apiEndpoint, page, pageSize, search, statusFilter]);

  useEffect(() => { fetchItems(); }, [fetchItems]);

  const filteredItems = useMemo(() => {
    return items.filter((item) => {
      const matchesSearch = !search || item.name.toLowerCase().includes(search.toLowerCase());
      const matchesStatus = statusFilter === "all" || item.status === statusFilter;
      return matchesSearch && matchesStatus;
    });
  }, [items, search, statusFilter]);

  const paginatedItems = filteredItems.slice((page - 1) * pageSize, page * pageSize);
  const totalPages = Math.ceil(filteredItems.length / pageSize);

  const handleDelete = useCallback(async (id: string) => {
    try {
      const response = await fetch(apiEndpoint + "/" + id, { method: "DELETE" });
      if (!response.ok) throw new Error("Delete failed");
      setItems((prev) => prev.filter((item) => item.id !== id));
    } catch (error) {
      console.error("Delete error:", error);
    }
  }, [apiEndpoint]);

  const statusColors: Record<string, string> = {
    active: "bg-green-100 text-green-800",
    inactive: "bg-gray-100 text-gray-800",
    pending: "bg-yellow-100 text-yellow-800",
  };

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle>Item Manager</CardTitle>
          <div className="flex items-center gap-2">
            <Button variant="outline" size="sm" onClick={fetchItems} disabled={loading}>
              <RefreshCw className={"h-4 w-4 " + (loading ? "animate-spin" : "")} />
            </Button>
            <Button size="sm"><Plus className="h-4 w-4 mr-1" /> Add New</Button>
          </div>
        </div>
        <div className="flex gap-2 mt-4">
          <div className="relative flex-1">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
            <Input
              placeholder="Search..."
              value={search}
              onChange={(e) => { setSearch(e.target.value); setPage(1); }}
              className="pl-9"
            />
          </div>
          <select
            value={statusFilter}
            onChange={(e) => { setStatusFilter(e.target.value); setPage(1); }}
            className="border rounded px-3 py-2 text-sm"
          >
            <option value="all">All Status</option>
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
            <option value="pending">Pending</option>
          </select>
        </div>
      </CardHeader>
      <CardContent>
        {loading ? (
          <div className="flex items-center justify-center py-12">
            <Loader2 className="h-6 w-6 animate-spin text-muted-foreground" />
          </div>
        ) : paginatedItems.length === 0 ? (
          <p className="text-center py-12 text-muted-foreground">No items found</p>
        ) : (
          <div className="space-y-2">
            {paginatedItems.map((item) => (
              <div key={item.id} className="flex items-center justify-between p-3 border rounded-lg hover:bg-muted/50 transition-colors">
                <div>
                  <p className="font-medium">{item.name}</p>
                  <p className="text-xs text-muted-foreground">
                    {item.category} - {new Date(item.createdAt).toLocaleDateString()}
                  </p>
                </div>
                <div className="flex items-center gap-2">
                  <Badge className={statusColors[item.status] || ""}>{item.status}</Badge>
                  <Button variant="ghost" size="sm"><Edit className="h-4 w-4" /></Button>
                  <Button variant="ghost" size="sm" onClick={() => handleDelete(item.id)}>
                    <Trash2 className="h-4 w-4 text-red-500" />
                  </Button>
                </div>
              </div>
            ))}
          </div>
        )}

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4 pt-4 border-t">
            <p className="text-sm text-muted-foreground">Page {page} of {totalPages}</p>
            <div className="flex gap-1">
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.max(1, p - 1))} disabled={page === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.min(totalPages, p + 1))} disabled={page === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```


### FILE UPLOAD STORAGE - Advanced Implementation Pattern #4

```typescript
// lib/file-upload-storage/pattern-4.ts
import { z } from "zod";

interface ServiceConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  batchSize: number;
  debug: boolean;
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
  batchSize: z.number().min(1).max(1000).default(50),
  debug: z.boolean().default(false),
});

export class FILEUPLOADSTORAGEService4 {
  private config: ServiceConfig;
  private cache: Map<string, { data: unknown; expiresAt: number }> = new Map();
  private metrics = { operations: 0, errors: 0, avgDuration: 0 };

  constructor(config: Partial<ServiceConfig> = {}) {
    this.config = ConfigSchema.parse(config) as ServiceConfig;
  }

  async execute<TInput, TOutput>(
    operation: string,
    input: TInput,
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    this.metrics.operations++;
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);
        const result = await handler(input);
        clearTimeout(timeoutId);

        const duration = Date.now() - startTime;
        this.metrics.avgDuration =
          (this.metrics.avgDuration * (this.metrics.operations - 1) + duration) /
          this.metrics.operations;

        return {
          success: true,
          data: result,
          duration,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        this.metrics.errors++;

        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Operation failed",
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

  async executeBatch<TInput, TOutput>(
    operation: string,
    inputs: TInput[],
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput[]>> {
    const startTime = Date.now();
    const results: TOutput[] = [];
    const errors: string[] = [];

    for (let i = 0; i < inputs.length; i += this.config.batchSize) {
      const batch = inputs.slice(i, i + this.config.batchSize);
      const batchResults = await Promise.allSettled(
        batch.map((input) => this.execute(operation, input, handler))
      );

      for (const result of batchResults) {
        if (result.status === "fulfilled" && result.value.success && result.value.data) {
          results.push(result.value.data);
        } else if (result.status === "rejected") {
          errors.push(result.reason?.message || "Batch item failed");
        }
      }
    }

    return {
      success: errors.length === 0,
      data: results,
      error: errors.length > 0 ? errors.length + " items failed" : undefined,
      duration: Date.now() - startTime,
      retries: 0,
      timestamp: new Date(),
    };
  }

  getCachedOrFetch<T>(key: string, fetcher: () => Promise<T>, ttlMs: number = 60000): Promise<T> {
    const cached = this.cache.get(key);
    if (cached && Date.now() < cached.expiresAt) {
      return Promise.resolve(cached.data as T);
    }
    return fetcher().then((data) => {
      this.cache.set(key, { data, expiresAt: Date.now() + ttlMs });
      return data;
    });
  }

  getMetrics() {
    return {
      ...this.metrics,
      errorRate: this.metrics.operations > 0
        ? ((this.metrics.errors / this.metrics.operations) * 100).toFixed(2) + "%"
        : "0%",
      cacheSize: this.cache.size,
    };
  }
}
```

```typescript
// components/file-upload-storage/Manager4.tsx
"use client";

import { useState, useEffect, useCallback, useMemo } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import { Loader2, Plus, Search, RefreshCw, Trash2, Edit, ChevronLeft, ChevronRight } from "lucide-react";

interface Item {
  id: string;
  name: string;
  status: "active" | "inactive" | "pending";
  category: string;
  createdAt: string;
  updatedAt: string;
}

interface ManagerProps {
  initialItems?: Item[];
  apiEndpoint?: string;
  pageSize?: number;
}

export function Manager4({
  initialItems = [],
  apiEndpoint = "/api/items",
  pageSize = 10,
}: ManagerProps) {
  const [items, setItems] = useState<Item[]>(initialItems);
  const [loading, setLoading] = useState(false);
  const [search, setSearch] = useState("");
  const [statusFilter, setStatusFilter] = useState<string>("all");
  const [page, setPage] = useState(1);

  const fetchItems = useCallback(async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams({ page: String(page), limit: String(pageSize) });
      if (search) params.set("search", search);
      if (statusFilter !== "all") params.set("status", statusFilter);

      const response = await fetch(apiEndpoint + "?" + params);
      if (!response.ok) throw new Error("Failed to fetch");
      const data = await response.json();
      setItems(data.items || data.data || []);
    } catch (error) {
      console.error("Fetch error:", error);
    } finally {
      setLoading(false);
    }
  }, [apiEndpoint, page, pageSize, search, statusFilter]);

  useEffect(() => { fetchItems(); }, [fetchItems]);

  const filteredItems = useMemo(() => {
    return items.filter((item) => {
      const matchesSearch = !search || item.name.toLowerCase().includes(search.toLowerCase());
      const matchesStatus = statusFilter === "all" || item.status === statusFilter;
      return matchesSearch && matchesStatus;
    });
  }, [items, search, statusFilter]);

  const paginatedItems = filteredItems.slice((page - 1) * pageSize, page * pageSize);
  const totalPages = Math.ceil(filteredItems.length / pageSize);

  const handleDelete = useCallback(async (id: string) => {
    try {
      const response = await fetch(apiEndpoint + "/" + id, { method: "DELETE" });
      if (!response.ok) throw new Error("Delete failed");
      setItems((prev) => prev.filter((item) => item.id !== id));
    } catch (error) {
      console.error("Delete error:", error);
    }
  }, [apiEndpoint]);

  const statusColors: Record<string, string> = {
    active: "bg-green-100 text-green-800",
    inactive: "bg-gray-100 text-gray-800",
    pending: "bg-yellow-100 text-yellow-800",
  };

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle>Item Manager</CardTitle>
          <div className="flex items-center gap-2">
            <Button variant="outline" size="sm" onClick={fetchItems} disabled={loading}>
              <RefreshCw className={"h-4 w-4 " + (loading ? "animate-spin" : "")} />
            </Button>
            <Button size="sm"><Plus className="h-4 w-4 mr-1" /> Add New</Button>
          </div>
        </div>
        <div className="flex gap-2 mt-4">
          <div className="relative flex-1">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
            <Input
              placeholder="Search..."
              value={search}
              onChange={(e) => { setSearch(e.target.value); setPage(1); }}
              className="pl-9"
            />
          </div>
          <select
            value={statusFilter}
            onChange={(e) => { setStatusFilter(e.target.value); setPage(1); }}
            className="border rounded px-3 py-2 text-sm"
          >
            <option value="all">All Status</option>
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
            <option value="pending">Pending</option>
          </select>
        </div>
      </CardHeader>
      <CardContent>
        {loading ? (
          <div className="flex items-center justify-center py-12">
            <Loader2 className="h-6 w-6 animate-spin text-muted-foreground" />
          </div>
        ) : paginatedItems.length === 0 ? (
          <p className="text-center py-12 text-muted-foreground">No items found</p>
        ) : (
          <div className="space-y-2">
            {paginatedItems.map((item) => (
              <div key={item.id} className="flex items-center justify-between p-3 border rounded-lg hover:bg-muted/50 transition-colors">
                <div>
                  <p className="font-medium">{item.name}</p>
                  <p className="text-xs text-muted-foreground">
                    {item.category} - {new Date(item.createdAt).toLocaleDateString()}
                  </p>
                </div>
                <div className="flex items-center gap-2">
                  <Badge className={statusColors[item.status] || ""}>{item.status}</Badge>
                  <Button variant="ghost" size="sm"><Edit className="h-4 w-4" /></Button>
                  <Button variant="ghost" size="sm" onClick={() => handleDelete(item.id)}>
                    <Trash2 className="h-4 w-4 text-red-500" />
                  </Button>
                </div>
              </div>
            ))}
          </div>
        )}

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4 pt-4 border-t">
            <p className="text-sm text-muted-foreground">Page {page} of {totalPages}</p>
            <div className="flex gap-1">
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.max(1, p - 1))} disabled={page === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.min(totalPages, p + 1))} disabled={page === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```


### FILE UPLOAD STORAGE - Advanced Implementation Pattern #5

```typescript
// lib/file-upload-storage/pattern-5.ts
import { z } from "zod";

interface ServiceConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  batchSize: number;
  debug: boolean;
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
  batchSize: z.number().min(1).max(1000).default(50),
  debug: z.boolean().default(false),
});

export class FILEUPLOADSTORAGEService5 {
  private config: ServiceConfig;
  private cache: Map<string, { data: unknown; expiresAt: number }> = new Map();
  private metrics = { operations: 0, errors: 0, avgDuration: 0 };

  constructor(config: Partial<ServiceConfig> = {}) {
    this.config = ConfigSchema.parse(config) as ServiceConfig;
  }

  async execute<TInput, TOutput>(
    operation: string,
    input: TInput,
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    this.metrics.operations++;
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);
        const result = await handler(input);
        clearTimeout(timeoutId);

        const duration = Date.now() - startTime;
        this.metrics.avgDuration =
          (this.metrics.avgDuration * (this.metrics.operations - 1) + duration) /
          this.metrics.operations;

        return {
          success: true,
          data: result,
          duration,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        this.metrics.errors++;

        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Operation failed",
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

  async executeBatch<TInput, TOutput>(
    operation: string,
    inputs: TInput[],
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput[]>> {
    const startTime = Date.now();
    const results: TOutput[] = [];
    const errors: string[] = [];

    for (let i = 0; i < inputs.length; i += this.config.batchSize) {
      const batch = inputs.slice(i, i + this.config.batchSize);
      const batchResults = await Promise.allSettled(
        batch.map((input) => this.execute(operation, input, handler))
      );

      for (const result of batchResults) {
        if (result.status === "fulfilled" && result.value.success && result.value.data) {
          results.push(result.value.data);
        } else if (result.status === "rejected") {
          errors.push(result.reason?.message || "Batch item failed");
        }
      }
    }

    return {
      success: errors.length === 0,
      data: results,
      error: errors.length > 0 ? errors.length + " items failed" : undefined,
      duration: Date.now() - startTime,
      retries: 0,
      timestamp: new Date(),
    };
  }

  getCachedOrFetch<T>(key: string, fetcher: () => Promise<T>, ttlMs: number = 60000): Promise<T> {
    const cached = this.cache.get(key);
    if (cached && Date.now() < cached.expiresAt) {
      return Promise.resolve(cached.data as T);
    }
    return fetcher().then((data) => {
      this.cache.set(key, { data, expiresAt: Date.now() + ttlMs });
      return data;
    });
  }

  getMetrics() {
    return {
      ...this.metrics,
      errorRate: this.metrics.operations > 0
        ? ((this.metrics.errors / this.metrics.operations) * 100).toFixed(2) + "%"
        : "0%",
      cacheSize: this.cache.size,
    };
  }
}
```

```typescript
// components/file-upload-storage/Manager5.tsx
"use client";

import { useState, useEffect, useCallback, useMemo } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import { Loader2, Plus, Search, RefreshCw, Trash2, Edit, ChevronLeft, ChevronRight } from "lucide-react";

interface Item {
  id: string;
  name: string;
  status: "active" | "inactive" | "pending";
  category: string;
  createdAt: string;
  updatedAt: string;
}

interface ManagerProps {
  initialItems?: Item[];
  apiEndpoint?: string;
  pageSize?: number;
}

export function Manager5({
  initialItems = [],
  apiEndpoint = "/api/items",
  pageSize = 10,
}: ManagerProps) {
  const [items, setItems] = useState<Item[]>(initialItems);
  const [loading, setLoading] = useState(false);
  const [search, setSearch] = useState("");
  const [statusFilter, setStatusFilter] = useState<string>("all");
  const [page, setPage] = useState(1);

  const fetchItems = useCallback(async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams({ page: String(page), limit: String(pageSize) });
      if (search) params.set("search", search);
      if (statusFilter !== "all") params.set("status", statusFilter);

      const response = await fetch(apiEndpoint + "?" + params);
      if (!response.ok) throw new Error("Failed to fetch");
      const data = await response.json();
      setItems(data.items || data.data || []);
    } catch (error) {
      console.error("Fetch error:", error);
    } finally {
      setLoading(false);
    }
  }, [apiEndpoint, page, pageSize, search, statusFilter]);

  useEffect(() => { fetchItems(); }, [fetchItems]);

  const filteredItems = useMemo(() => {
    return items.filter((item) => {
      const matchesSearch = !search || item.name.toLowerCase().includes(search.toLowerCase());
      const matchesStatus = statusFilter === "all" || item.status === statusFilter;
      return matchesSearch && matchesStatus;
    });
  }, [items, search, statusFilter]);

  const paginatedItems = filteredItems.slice((page - 1) * pageSize, page * pageSize);
  const totalPages = Math.ceil(filteredItems.length / pageSize);

  const handleDelete = useCallback(async (id: string) => {
    try {
      const response = await fetch(apiEndpoint + "/" + id, { method: "DELETE" });
      if (!response.ok) throw new Error("Delete failed");
      setItems((prev) => prev.filter((item) => item.id !== id));
    } catch (error) {
      console.error("Delete error:", error);
    }
  }, [apiEndpoint]);

  const statusColors: Record<string, string> = {
    active: "bg-green-100 text-green-800",
    inactive: "bg-gray-100 text-gray-800",
    pending: "bg-yellow-100 text-yellow-800",
  };

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle>Item Manager</CardTitle>
          <div className="flex items-center gap-2">
            <Button variant="outline" size="sm" onClick={fetchItems} disabled={loading}>
              <RefreshCw className={"h-4 w-4 " + (loading ? "animate-spin" : "")} />
            </Button>
            <Button size="sm"><Plus className="h-4 w-4 mr-1" /> Add New</Button>
          </div>
        </div>
        <div className="flex gap-2 mt-4">
          <div className="relative flex-1">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
            <Input
              placeholder="Search..."
              value={search}
              onChange={(e) => { setSearch(e.target.value); setPage(1); }}
              className="pl-9"
            />
          </div>
          <select
            value={statusFilter}
            onChange={(e) => { setStatusFilter(e.target.value); setPage(1); }}
            className="border rounded px-3 py-2 text-sm"
          >
            <option value="all">All Status</option>
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
            <option value="pending">Pending</option>
          </select>
        </div>
      </CardHeader>
      <CardContent>
        {loading ? (
          <div className="flex items-center justify-center py-12">
            <Loader2 className="h-6 w-6 animate-spin text-muted-foreground" />
          </div>
        ) : paginatedItems.length === 0 ? (
          <p className="text-center py-12 text-muted-foreground">No items found</p>
        ) : (
          <div className="space-y-2">
            {paginatedItems.map((item) => (
              <div key={item.id} className="flex items-center justify-between p-3 border rounded-lg hover:bg-muted/50 transition-colors">
                <div>
                  <p className="font-medium">{item.name}</p>
                  <p className="text-xs text-muted-foreground">
                    {item.category} - {new Date(item.createdAt).toLocaleDateString()}
                  </p>
                </div>
                <div className="flex items-center gap-2">
                  <Badge className={statusColors[item.status] || ""}>{item.status}</Badge>
                  <Button variant="ghost" size="sm"><Edit className="h-4 w-4" /></Button>
                  <Button variant="ghost" size="sm" onClick={() => handleDelete(item.id)}>
                    <Trash2 className="h-4 w-4 text-red-500" />
                  </Button>
                </div>
              </div>
            ))}
          </div>
        )}

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4 pt-4 border-t">
            <p className="text-sm text-muted-foreground">Page {page} of {totalPages}</p>
            <div className="flex gap-1">
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.max(1, p - 1))} disabled={page === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.min(totalPages, p + 1))} disabled={page === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```


### FILE UPLOAD STORAGE - Advanced Implementation Pattern #6

```typescript
// lib/file-upload-storage/pattern-6.ts
import { z } from "zod";

interface ServiceConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  batchSize: number;
  debug: boolean;
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
  batchSize: z.number().min(1).max(1000).default(50),
  debug: z.boolean().default(false),
});

export class FILEUPLOADSTORAGEService6 {
  private config: ServiceConfig;
  private cache: Map<string, { data: unknown; expiresAt: number }> = new Map();
  private metrics = { operations: 0, errors: 0, avgDuration: 0 };

  constructor(config: Partial<ServiceConfig> = {}) {
    this.config = ConfigSchema.parse(config) as ServiceConfig;
  }

  async execute<TInput, TOutput>(
    operation: string,
    input: TInput,
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    this.metrics.operations++;
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);
        const result = await handler(input);
        clearTimeout(timeoutId);

        const duration = Date.now() - startTime;
        this.metrics.avgDuration =
          (this.metrics.avgDuration * (this.metrics.operations - 1) + duration) /
          this.metrics.operations;

        return {
          success: true,
          data: result,
          duration,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        this.metrics.errors++;

        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Operation failed",
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

  async executeBatch<TInput, TOutput>(
    operation: string,
    inputs: TInput[],
    handler: (input: TInput) => Promise<TOutput>
  ): Promise<ProcessResult<TOutput[]>> {
    const startTime = Date.now();
    const results: TOutput[] = [];
    const errors: string[] = [];

    for (let i = 0; i < inputs.length; i += this.config.batchSize) {
      const batch = inputs.slice(i, i + this.config.batchSize);
      const batchResults = await Promise.allSettled(
        batch.map((input) => this.execute(operation, input, handler))
      );

      for (const result of batchResults) {
        if (result.status === "fulfilled" && result.value.success && result.value.data) {
          results.push(result.value.data);
        } else if (result.status === "rejected") {
          errors.push(result.reason?.message || "Batch item failed");
        }
      }
    }

    return {
      success: errors.length === 0,
      data: results,
      error: errors.length > 0 ? errors.length + " items failed" : undefined,
      duration: Date.now() - startTime,
      retries: 0,
      timestamp: new Date(),
    };
  }

  getCachedOrFetch<T>(key: string, fetcher: () => Promise<T>, ttlMs: number = 60000): Promise<T> {
    const cached = this.cache.get(key);
    if (cached && Date.now() < cached.expiresAt) {
      return Promise.resolve(cached.data as T);
    }
    return fetcher().then((data) => {
      this.cache.set(key, { data, expiresAt: Date.now() + ttlMs });
      return data;
    });
  }

  getMetrics() {
    return {
      ...this.metrics,
      errorRate: this.metrics.operations > 0
        ? ((this.metrics.errors / this.metrics.operations) * 100).toFixed(2) + "%"
        : "0%",
      cacheSize: this.cache.size,
    };
  }
}
```

```typescript
// components/file-upload-storage/Manager6.tsx
"use client";

import { useState, useEffect, useCallback, useMemo } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import { Loader2, Plus, Search, RefreshCw, Trash2, Edit, ChevronLeft, ChevronRight } from "lucide-react";

interface Item {
  id: string;
  name: string;
  status: "active" | "inactive" | "pending";
  category: string;
  createdAt: string;
  updatedAt: string;
}

interface ManagerProps {
  initialItems?: Item[];
  apiEndpoint?: string;
  pageSize?: number;
}

export function Manager6({
  initialItems = [],
  apiEndpoint = "/api/items",
  pageSize = 10,
}: ManagerProps) {
  const [items, setItems] = useState<Item[]>(initialItems);
  const [loading, setLoading] = useState(false);
  const [search, setSearch] = useState("");
  const [statusFilter, setStatusFilter] = useState<string>("all");
  const [page, setPage] = useState(1);

  const fetchItems = useCallback(async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams({ page: String(page), limit: String(pageSize) });
      if (search) params.set("search", search);
      if (statusFilter !== "all") params.set("status", statusFilter);

      const response = await fetch(apiEndpoint + "?" + params);
      if (!response.ok) throw new Error("Failed to fetch");
      const data = await response.json();
      setItems(data.items || data.data || []);
    } catch (error) {
      console.error("Fetch error:", error);
    } finally {
      setLoading(false);
    }
  }, [apiEndpoint, page, pageSize, search, statusFilter]);

  useEffect(() => { fetchItems(); }, [fetchItems]);

  const filteredItems = useMemo(() => {
    return items.filter((item) => {
      const matchesSearch = !search || item.name.toLowerCase().includes(search.toLowerCase());
      const matchesStatus = statusFilter === "all" || item.status === statusFilter;
      return matchesSearch && matchesStatus;
    });
  }, [items, search, statusFilter]);

  const paginatedItems = filteredItems.slice((page - 1) * pageSize, page * pageSize);
  const totalPages = Math.ceil(filteredItems.length / pageSize);

  const handleDelete = useCallback(async (id: string) => {
    try {
      const response = await fetch(apiEndpoint + "/" + id, { method: "DELETE" });
      if (!response.ok) throw new Error("Delete failed");
      setItems((prev) => prev.filter((item) => item.id !== id));
    } catch (error) {
      console.error("Delete error:", error);
    }
  }, [apiEndpoint]);

  const statusColors: Record<string, string> = {
    active: "bg-green-100 text-green-800",
    inactive: "bg-gray-100 text-gray-800",
    pending: "bg-yellow-100 text-yellow-800",
  };

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle>Item Manager</CardTitle>
          <div className="flex items-center gap-2">
            <Button variant="outline" size="sm" onClick={fetchItems} disabled={loading}>
              <RefreshCw className={"h-4 w-4 " + (loading ? "animate-spin" : "")} />
            </Button>
            <Button size="sm"><Plus className="h-4 w-4 mr-1" /> Add New</Button>
          </div>
        </div>
        <div className="flex gap-2 mt-4">
          <div className="relative flex-1">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
            <Input
              placeholder="Search..."
              value={search}
              onChange={(e) => { setSearch(e.target.value); setPage(1); }}
              className="pl-9"
            />
          </div>
          <select
            value={statusFilter}
            onChange={(e) => { setStatusFilter(e.target.value); setPage(1); }}
            className="border rounded px-3 py-2 text-sm"
          >
            <option value="all">All Status</option>
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
            <option value="pending">Pending</option>
          </select>
        </div>
      </CardHeader>
      <CardContent>
        {loading ? (
          <div className="flex items-center justify-center py-12">
            <Loader2 className="h-6 w-6 animate-spin text-muted-foreground" />
          </div>
        ) : paginatedItems.length === 0 ? (
          <p className="text-center py-12 text-muted-foreground">No items found</p>
        ) : (
          <div className="space-y-2">
            {paginatedItems.map((item) => (
              <div key={item.id} className="flex items-center justify-between p-3 border rounded-lg hover:bg-muted/50 transition-colors">
                <div>
                  <p className="font-medium">{item.name}</p>
                  <p className="text-xs text-muted-foreground">
                    {item.category} - {new Date(item.createdAt).toLocaleDateString()}
                  </p>
                </div>
                <div className="flex items-center gap-2">
                  <Badge className={statusColors[item.status] || ""}>{item.status}</Badge>
                  <Button variant="ghost" size="sm"><Edit className="h-4 w-4" /></Button>
                  <Button variant="ghost" size="sm" onClick={() => handleDelete(item.id)}>
                    <Trash2 className="h-4 w-4 text-red-500" />
                  </Button>
                </div>
              </div>
            ))}
          </div>
        )}

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4 pt-4 border-t">
            <p className="text-sm text-muted-foreground">Page {page} of {totalPages}</p>
            <div className="flex gap-1">
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.max(1, p - 1))} disabled={page === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setPage((p) => Math.min(totalPages, p + 1))} disabled={page === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```
