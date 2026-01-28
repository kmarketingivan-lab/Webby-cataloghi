# CATALOGO FILE UPLOAD & STORAGE v1

> **Versione**: 1.0
> **Data**: 2026-01-27
> **Ambito**: File upload, S3/R2 storage, multipart upload, image processing, security patterns

---

## 1. PRESIGNED URL UPLOAD (STANDARD)

### 1.1 S3 Presigned URL Generation

```typescript
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
```

### 1.2 API Route (Next.js)

```typescript
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
```

### 1.3 Client Upload Function

```typescript
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
```

---

## 2. CHUNKED / MULTIPART UPLOAD (LARGE FILES)

```typescript
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
```

---

## 3. IMAGE PROCESSING (RESPONSIVE / WEBP / AVIF)

```typescript
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
```

---

## 4. SECURITY PATTERNS

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

### 4.1 File Validation Schema

```typescript
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
```

---

## 5. REACT COMPONENTS

### 5.1 Dropzone + Upload Progress

```tsx
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
```

### 5.2 Large File Uploader (Chunked)

```tsx
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
```

---

## 6. CDN INTEGRATION

| Provider   | Setup                                        | Signed URLs | Edge Caching |
| ---------- | -------------------------------------------- | ----------- | ------------ |
| CloudFront | Origin = S3, OAI/OAC for private buckets     | ✅          | ✅           |
| Cloudflare | R2 native or proxy to S3                     | ✅          | ✅           |
| Bunny.net  | Pull zone from S3 origin                     | ✅          | ✅           |
| Vercel     | Edge middleware for auth + rewrite to S3/R2  | ⚠️         | ✅           |

### 6.1 CloudFront Signed URL

```typescript
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
```

---

## 7. STORAGE COST COMPARISON

| Provider        | Storage (GB/mo) | PUT (per 1k) | GET (per 1k) | Egress (GB)  |
| --------------- | --------------- | ------------ | ------------ | ------------ |
| AWS S3 Standard | $0.023          | $0.005       | $0.0004      | $0.09        |
| AWS S3 IA       | $0.0125         | $0.01        | $0.001       | $0.09        |
| Cloudflare R2   | $0.015          | $0.0045      | Free         | **Free**     |
| Backblaze B2    | $0.006          | Free         | Free         | $0.01        |
| Supabase        | $0.021          | Included     | Included     | $0.09        |

---

## 8. FILE UPLOAD CHECKLIST

```
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
```

---

## 9. ADDITIONAL TIPS

1. Use **Web Workers** for large file processing in browser
2. Generate **responsive image sizes** for each format and serve via `<picture>` tag
3. Combine **chunked uploads** + **presigned URLs** for secure, large file handling
4. Always **clean up incomplete multipart uploads** to avoid storage leaks
5. Enable **S3 lifecycle rules** to automatically expire temp files
6. Use **Next.js edge functions or middleware** for auth on presigned URLs
7. For multi-user apps, separate uploads by **user ID folder** to isolate permissions
