# Catalogo Search per Next.js 14+ e Node.js

§ 1. SEARCH ENGINE COMPARISON

| Engine | Type | Hosting | Latency | Full-text | Facets | Typo Tolerance | Multi-Lang | Vector Search | Best For |
|--------|------|---------|---------|-----------|--------|----------------|------------|---------------|----------|
| Meilisearch | Dedicated | Self/Cloud | <50ms | ✅ | ✅ | ✅ Advanced | ✅ 50+ | ✅ Hybrid | General purpose, developer experience |
| Typesense | Dedicated | Self/Cloud | <50ms | ✅ | ✅ | ✅ Configurable | ✅ 40+ | ✅ Native | Speed-focused, production scaling |
| Algolia | SaaS | Cloud | <20ms | ✅ | ✅ | ✅ AI-powered | ✅ 80+ | ✅ (Premium) | Enterprise, managed service |
| Elasticsearch | Dedicated | Self/Cloud | 50-200ms | ✅ | ✅ | ❌ Requires plugins | ✅ | ✅ with plugins | Complex queries, big data |
| PostgreSQL FTS | Built-in | DB | 50-500ms | ✅ | ❌ Manual | ❌ | ⚠️ Limited | Simple search, budget constraints |
| SQLite FTS5 | Built-in | DB | <10ms | ✅ | ❌ Manual | ❌ | ❌ | Small datasets, local apps |
| Orama | In-browser | Client | <5ms | ✅ | ✅ | ✅ | ✅ | ✅ (soon) | Static sites, edge functions |

§ PRICING COMPARISON

| Engine | Free Tier | Paid Starting | Self-hosted | OSS License | Best For Budget |
|--------|-----------|---------------|-------------|-------------|-----------------|
| Meilisearch | Unlimited docs (self) | $30/mo (cloud) | ✅ MIT | ✅ Apache 2.0 | Startups, scale-to-pay |
| Typesense | Unlimited (self) | $0.03/1k search | ✅ GPLv3 | ✅ Commercial | High-volume, predictable costs |
| Algolia | 10k searches/mo | $1/1k searches | ❌ Proprietary | ❌ | Enterprise, managed service |
| Elasticsearch | Unlimited (self) | $95/mo (cloud) | ✅ Elastic | ⚠️ SSPL | Complex needs, self-hosting |
| PostgreSQL | Free with DB | Included | ✅ PostgreSQL | ✅ PostgreSQL | Already using Postgres |

§ FEATURE MATRIX

| Feature | Meilisearch | Typesense | Algolia | PostgreSQL FTS |
|---------|-------------|-----------|---------|----------------|
| Typo tolerance | ✅ (advanced) | ✅ (configurable) | ✅ (AI) | ❌ |
| Synonyms | ✅ | ✅ | ✅ | ⚠️ Manual |
| Faceted search | ✅ | ✅ | ✅ | ❌ |
| Geo search | ✅ | ✅ | ✅ | ⚠️ With PostGIS |
| Multi-tenancy | ✅ (index level) | ✅ (API keys) | ✅ | ⚠️ Manual |
| Real-time updates | ✅ | ✅ | ✅ | ❌ |
| SDK quality | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| Admin UI | ✅ Meilisearch UI | ✅ Typesense UI | ✅ Algolia Dashboard | ❌ |
| Vector search | ✅ Hybrid | ✅ Native | ✅ (Premium) | ⚠️ pgvector |

---

§ 2. MEILISEARCH IMPLEMENTATION

§ 2.1 SETUP & CONFIGURATION
typescript
// lib/search/meilisearch.ts
import { MeiliSearch } from 'meilisearch';
import { env } from '@/env';

// Singleton client instance
class SearchClient {
  private static instance: MeiliSearch;
  private static indexes = new Map<string, ReturnType<MeiliSearch['index']>>();

  private constructor() {}

  static getInstance(): MeiliSearch {
    if (!SearchClient.instance) {
      if (!env.MEILISEARCH_HOST || !env.MEILISEARCH_API_KEY) {
        throw new Error('Meilisearch environment variables not configured');
      }

      SearchClient.instance = new MeiliSearch({
        host: env.MEILISEARCH_HOST,
        apiKey: env.MEILISEARCH_API_KEY,
        
        // Client configuration
        requestConfig: {
          timeout: env.NODE_ENV === 'production' ? 5000 : 10000,
          headers: {
            'User-Agent': `my-app/${env.NEXT_PUBLIC_APP_VERSION}`,
          },
        },
      });
    }

    return SearchClient.instance;
  }

  static getIndex<T = any>(indexName: string) {
    if (!SearchClient.indexes.has(indexName)) {
      const client = SearchClient.getInstance();
      SearchClient.indexes.set(indexName, client.index<T>(indexName));
    }
    return SearchClient.indexes.get(indexName)!;
  }

  // Health check
  static async healthCheck(): Promise<boolean> {
    try {
      const client = SearchClient.getInstance();
      const health = await client.health();
      return health.status === 'available';
    } catch (error) {
      console.error('Meilisearch health check failed:', error);
      return false;
    }
  }

  // Initialize indexes with settings
  static async initializeIndexes() {
    const client = SearchClient.getInstance();
    
    const indexes = [
      {
        name: 'products',
        primaryKey: 'id',
        settings: {
          searchableAttributes: [
            'name',
            'description',
            'category',
            'tags',
            'sku',
          ],
          filterableAttributes: [
            'category',
            'brand',
            'price',
            'inStock',
            'rating',
            'createdAt',
            'updatedAt',
          ],
          sortableAttributes: [
            'price',
            'rating',
            'createdAt',
            'popularity',
          ],
          distinctAttribute: 'id',
          rankingRules: [
            'words',
            'typo',
            'proximity',
            'attribute',
            'sort',
            'exactness',
            // Custom ranking
            'desc(popularity)',
            'desc(rating)',
          ],
          stopWords: ['the', 'a', 'an', 'and', 'or', 'in', 'on', 'at', 'to'],
          synonyms: {
            'phone': ['mobile', 'cellphone', 'smartphone'],
            'tv': ['television', 'smart tv'],
            'laptop': ['notebook', 'macbook'],
          },
          typoTolerance: {
            enabled: true,
            minWordSizeForTypos: {
              oneTypo: 5,
              twoTypos: 9,
            },
            disableOnWords: ['iphone', 'samsung', 'sony'],
            disableOnAttributes: ['sku'],
          },
          faceting: {
            maxValuesPerFacet: 100,
          },
          pagination: {
            maxTotalHits: 10000,
          },
        },
      },
      {
        name: 'articles',
        primaryKey: 'id',
        settings: {
          searchableAttributes: [
            'title',
            'content',
            'author',
            'tags',
            'excerpt',
          ],
          filterableAttributes: [
            'category',
            'author',
            'published',
            'createdAt',
            'readingTime',
          ],
          sortableAttributes: ['createdAt', 'readingTime', 'views'],
          rankingRules: [
            'words',
            'typo',
            'proximity',
            'attribute',
            'sort',
            'exactness',
            'desc(views)',
            'desc(createdAt)',
          ],
        },
      },
    ];

    for (const indexConfig of indexes) {
      try {
        const index = client.index(indexConfig.name);
        
        // Check if index exists
        try {
          await index.getStats();
          console.log(`Index ${indexConfig.name} already exists`);
        } catch {
          // Create index if it doesn't exist
          await client.createIndex(indexConfig.name, {
            primaryKey: indexConfig.primaryKey,
          });
          console.log(`Created index ${indexConfig.name}`);
        }

        // Update settings
        await index.updateSettings(indexConfig.settings);
        console.log(`Updated settings for ${indexConfig.name}`);
        
      } catch (error) {
        console.error(`Failed to initialize index ${indexConfig.name}:`, error);
        // Don't throw to allow other indexes to initialize
      }
    }
  }
}

export { SearchClient };

§ 2.2 INDEX CONFIGURATION TABLE

| Setting | Purpose | Example Value | Best Practice |
|---------|---------|---------------|---------------|
| `searchableAttributes` | Fields to search | `["title", "description", "tags"]` | List most important first |
| `filterableAttributes` | Fields for filtering | `["category", "price", "inStock"]` | Only fields you'll filter by |
| `sortableAttributes` | Fields for sorting | `["createdAt", "price", "popularity"]` | Limit to needed fields |
| `distinctAttribute` | Deduplication field | `"productId"` | Unique identifier |
| `rankingRules` | Result ordering | Custom array | Business logic first |
| `stopWords` | Ignored words | Language-specific | Use common stop words |
| `synonyms` | Word equivalents | Object mapping | Domain-specific terms |
| `typoTolerance` | Typo handling | Config object | Enable with min word size |
| `faceting.maxValuesPerFacet` | Facet limit | 100 | Adjust based on data |
| `pagination.maxTotalHits` | Max results | 10000 | Performance optimization |

§ 2.3 DOCUMENT INDEXING
typescript
// lib/search/indexing.ts
import { SearchClient } from './meilisearch';
import { prisma } from '@/lib/prisma';
import { env } from '@/env';
import { EventEmitter } from 'events';

export interface IndexableDocument {
  id: string;
  [key: string]: any;
}

export class SearchIndexer {
  private static readonly BATCH_SIZE = 1000;
  private static readonly RETRY_ATTEMPTS = 3;
  private static readonly RETRY_DELAY = 1000; // ms
  
  private static events = new EventEmitter();

  // Subscribe to indexing events
  static on(event: 'start' | 'complete' | 'error' | 'progress', handler: (data: any) => void) {
    this.events.on(event, handler);
  }

  // Add single document
  static async addDocument<T extends IndexableDocument>(
    indexName: string,
    document: T,
    options?: { retry?: boolean }
  ): Promise<void> {
    const index = SearchClient.getIndex<T>(indexName);
    
    const attemptIndexing = async (attempt = 1): Promise<void> => {
      try {
        await index.addDocuments([document]);
        this.events.emit('progress', { indexName, count: 1 });
      } catch (error) {
        if (options?.retry && attempt < this.RETRY_ATTEMPTS) {
          await new Promise(resolve => setTimeout(resolve, this.RETRY_DELAY * attempt));
          return attemptIndexing(attempt + 1);
        }
        throw error;
      }
    };

    return attemptIndexing();
  }

  // Update single document
  static async updateDocument<T extends IndexableDocument>(
    indexName: string,
    document: T
  ): Promise<void> {
    const index = SearchClient.getIndex<T>(indexName);
    await index.updateDocuments([document]);
  }

  // Delete document
  static async deleteDocument(
    indexName: string,
    documentId: string
  ): Promise<void> {
    const index = SearchClient.getIndex(indexName);
    await index.deleteDocument(documentId);
  }

  // Batch operations
  static async batchIndex<T extends IndexableDocument>(
    indexName: string,
    documents: T[],
    options?: { chunkSize?: number }
  ): Promise<{ success: number; failed: number }> {
    const chunkSize = options?.chunkSize || this.BATCH_SIZE;
    const index = SearchClient.getIndex<T>(indexName);
    
    let success = 0;
    let failed = 0;

    this.events.emit('start', { indexName, total: documents.length });

    for (let i = 0; i < documents.length; i += chunkSize) {
      const chunk = documents.slice(i, i + chunkSize);
      
      try {
        await index.addDocuments(chunk);
        success += chunk.length;
        
        this.events.emit('progress', {
          indexName,
          processed: i + chunk.length,
          total: documents.length,
          success,
          failed,
        });
      } catch (error) {
        console.error(`Failed to index chunk ${i}-${i + chunk.length}:`, error);
        failed += chunk.length;
        
        // Individual retry for failed documents
        for (const doc of chunk) {
          try {
            await this.addDocument(indexName, doc, { retry: true });
            success++;
            failed--;
          } catch {
            // Keep as failed
          }
        }
      }
    }

    this.events.emit('complete', { indexName, success, failed });
    return { success, failed };
  }

  // Full reindex from database
  static async reindexFromDatabase(indexName: string) {
    this.events.emit('start', { indexName, type: 'reindex' });

    switch (indexName) {
      case 'products':
        return this.reindexProducts();
      case 'articles':
        return this.reindexArticles();
      default:
        throw new Error(`Unknown index: ${indexName}`);
    }
  }

  private static async reindexProducts() {
    const batchSize = 500;
    let cursor: string | undefined;
    let totalProcessed = 0;

    try {
      // First, get total count for progress tracking
      const totalCount = await prisma.product.count();
      this.events.emit('progress', { indexName: 'products', processed: 0, total: totalCount });

      do {
        const products = await prisma.product.findMany({
          take: batchSize,
          ...(cursor && {
            skip: 1,
            cursor: { id: cursor },
          }),
          orderBy: { id: 'asc' },
          select: {
            id: true,
            name: true,
            description: true,
            sku: true,
            price: true,
            category: true,
            brand: true,
            tags: true,
            inStock: true,
            rating: true,
            popularity: true,
            createdAt: true,
            updatedAt: true,
            // Add any computed fields
            _search: {
              select: {
                reviewsCount: true,
                averageRating: true,
              },
            },
          },
        });

        if (products.length === 0) break;

        // Transform data for indexing
        const documents = products.map(product => ({
          ...product,
          // Flatten computed fields
          reviewsCount: product._search?.reviewsCount || 0,
          averageRating: product._search?.averageRating || 0,
          // Remove internal fields
          _search: undefined,
        }));

        // Index batch
        await this.batchIndex('products', documents, { chunkSize: 100 });
        
        totalProcessed += products.length;
        cursor = products[products.length - 1].id;
        
        this.events.emit('progress', {
          indexName: 'products',
          processed: totalProcessed,
          total: totalCount,
        });

        // Add small delay to avoid overwhelming the database
        if (products.length === batchSize) {
          await new Promise(resolve => setTimeout(resolve, 100));
        }
      } while (true);

      this.events.emit('complete', { indexName: 'products', success: totalProcessed, failed: 0 });
    } catch (error) {
      this.events.emit('error', { indexName: 'products', error });
      throw error;
    }
  }

  private static async reindexArticles() {
    // Similar implementation for articles
    // ...
  }

  // Incremental sync (for real-time updates)
  static async syncChanges<T extends IndexableDocument>(
    indexName: string,
    changes: Array<{
      type: 'create' | 'update' | 'delete';
      id: string;
      data?: T;
    }>
  ): Promise<void> {
    const index = SearchClient.getIndex<T>(indexName);
    
    const createsAndUpdates = changes
      .filter(change => change.type !== 'delete')
      .map(change => change.data!);
    
    const deletes = changes
      .filter(change => change.type === 'delete')
      .map(change => change.id);

    if (createsAndUpdates.length > 0) {
      await index.addDocuments(createsAndUpdates);
    }

    if (deletes.length > 0) {
      await index.deleteDocuments(deletes);
    }
  }
}

// Export singleton instance
export const searchIndexer = SearchIndexer;

§ 2.4 SYNC STRATEGY

| Strategy | Latency | Consistency | Complexity | Use Case |
|----------|---------|-------------|------------|----------|
| **Real-time (on write)** | <1s | Strong | Low | E-commerce, collaboration |
| **Webhook-triggered** | 1-5s | Eventual | Medium | CMS, external data |
| **Scheduled batch** | Minutes | Eventual | Low | Analytics, reporting |
| **Change Data Capture** | <1s | Strong | High | Financial, audit trails |
| **Hybrid** | Variable | Configurable | High | Complex systems |

typescript
// prisma/middleware/search-sync.ts
import { prisma } from '@/lib/prisma';
import { searchIndexer } from '@/lib/search/indexing';

export function createSearchSyncMiddleware() {
  return prisma.$use(async (params, next) => {
    const result = await next(params);
    
    // Only sync on write operations
    if (['create', 'update', 'delete', 'createMany', 'updateMany', 'deleteMany'].includes(params.action)) {
      // Defer sync to avoid blocking response
      setImmediate(async () => {
        try {
          await handleModelSync(params.model, params.action, result, params.args);
        } catch (error) {
          console.error('Search sync failed:', error);
          // Log but don't throw - search sync shouldn't break primary operation
        }
      });
    }
    
    return result;
  });
}

async function handleModelSync(model: string, action: string, result: any, args: any) {
  switch (model) {
    case 'Product':
      await syncProducts(model, action, result, args);
      break;
    case 'Article':
      await syncArticles(model, action, result, args);
      break;
    // Add other models as needed
  }
}

async function syncProducts(model: string, action: string, result: any, args: any) {
  const indexName = 'products';
  
  switch (action) {
    case 'create':
      // Handle single create
      if (result) {
        const document = await prepareProductDocument(result.id);
        if (document) {
          await searchIndexer.addDocument(indexName, document, { retry: true });
        }
      }
      break;
      
    case 'createMany':
      // Handle batch create
      const createdIds = result?.count ? await getCreatedProductIds(args) : [];
      for (const id of createdIds) {
        const document = await prepareProductDocument(id);
        if (document) {
          await searchIndexer.addDocument(indexName, document, { retry: true });
        }
      }
      break;
      
    case 'update':
      // Handle single update
      if (result) {
        const document = await prepareProductDocument(result.id);
        if (document) {
          await searchIndexer.updateDocument(indexName, document);
        }
      }
      break;
      
    case 'delete':
      // Handle single delete
      if (args?.where?.id) {
        await searchIndexer.deleteDocument(indexName, args.where.id);
      }
      break;
      
    case 'deleteMany':
      // Handle batch delete
      if (args?.where?.id?.in) {
        for (const id of args.where.id.in) {
          await searchIndexer.deleteDocument(indexName, id);
        }
      }
      break;
  }
}

async function prepareProductDocument(id: string): Promise<any> {
  const product = await prisma.product.findUnique({
    where: { id },
    include: {
      // Include all needed relations
      category: true,
      brand: true,
      _search: {
        select: {
          reviewsCount: true,
          averageRating: true,
        },
      },
    },
  });
  
  if (!product) return null;
  
  return {
    id: product.id,
    name: product.name,
    description: product.description,
    sku: product.sku,
    price: product.price,
    category: product.category?.name,
    brand: product.brand?.name,
    tags: product.tags,
    inStock: product.inStock,
    rating: product._search?.averageRating || 0,
    reviewsCount: product._search?.reviewsCount || 0,
    popularity: product.popularity,
    createdAt: product.createdAt.toISOString(),
    updatedAt: product.updatedAt.toISOString(),
  };
}

async function getCreatedProductIds(args: any): Promise<string[]> {
  // This is a simplified example
  // In reality, you'd need to query the created products
  return [];
}

---

§ 3. TYPESENSE IMPLEMENTATION

§ 3.1 SETUP & CONFIGURATION
typescript
// lib/search/typesense.ts
import { Client as TypesenseClient } from 'typesense';
import { CollectionCreateSchema } from 'typesense/lib/Typesense/Collections';
import { env } from '@/env';

export class TypesenseSearchClient {
  private static instance: TypesenseClient;
  private static collections = new Map<string, any>();

  private constructor() {}

  static getInstance(): TypesenseClient {
    if (!TypesenseSearchClient.instance) {
      if (!env.TYPESENSE_NODES || !env.TYPESENSE_API_KEY) {
        throw new Error('Typesense environment variables not configured');
      }

      const nodes = JSON.parse(env.TYPESENSE_NODES);
      
      TypesenseSearchClient.instance = new TypesenseClient({
        nodes,
        apiKey: env.TYPESENSE_API_KEY,
        connectionTimeoutSeconds: 5,
        numRetries: 3,
        retryIntervalSeconds: 0.1,
        healthcheckIntervalSeconds: 300,
        logLevel: env.NODE_ENV === 'development' ? 'debug' : 'error',
      });
    }

    return TypesenseSearchClient.instance;
  }

  static async initializeCollections() {
    const client = TypesenseSearchClient.getInstance();
    
    const collectionSchemas: CollectionCreateSchema[] = [
      {
        name: 'products',
        fields: [
          { name: 'id', type: 'string', facet: false },
          { name: 'name', type: 'string', facet: false },
          { name: 'description', type: 'string', facet: false },
          { name: 'sku', type: 'string', facet: false },
          { name: 'price', type: 'float', facet: true },
          { name: 'category', type: 'string', facet: true },
          { name: 'brand', type: 'string', facet: true },
          { name: 'tags', type: 'string[]', facet: true },
          { name: 'in_stock', type: 'bool', facet: true },
          { name: 'rating', type: 'float', facet: true },
          { name: 'popularity', type: 'int32', facet: true },
          { name: 'created_at', type: 'int64', facet: false },
          { name: 'updated_at', type: 'int64', facet: false },
          // Vector field for semantic search
          { name: 'embedding', type: 'float[]', facet: false, optional: true },
        ],
        default_sorting_field: 'popularity',
        token_separators: ['-', '_'],
        symbols_to_index: ['!', '@', '#', '$', '%', '&', '*'],
      },
      {
        name: 'articles',
        fields: [
          { name: 'id', type: 'string', facet: false },
          { name: 'title', type: 'string', facet: false },
          { name: 'content', type: 'string', facet: false },
          { name: 'author', type: 'string', facet: true },
          { name: 'category', type: 'string', facet: true },
          { name: 'tags', type: 'string[]', facet: true },
          { name: 'published', type: 'bool', facet: true },
          { name: 'reading_time', type: 'int32', facet: true },
          { name: 'views', type: 'int32', facet: true },
          { name: 'created_at', type: 'int64', facet: false },
        ],
        default_sorting_field: 'created_at',
      },
    ];

    for (const schema of collectionSchemas) {
      try {
        await client.collections().create(schema);
        console.log(`Created collection ${schema.name}`);
      } catch (error: any) {
        if (error.httpStatus !== 409) { // 409 = collection already exists
          console.error(`Failed to create collection ${schema.name}:`, error);
        }
      }
    }
  }

  static async searchProducts(params: {
    q: string;
    query_by?: string;
    filter_by?: string;
    sort_by?: string;
    page?: number;
    per_page?: number;
    facet_by?: string;
    max_facet_values?: number;
  }) {
    const client = TypesenseSearchClient.getInstance();
    
    const searchParameters = {
      q: params.q,
      query_by: params.query_by || 'name,description,tags,sku',
      filter_by: params.filter_by || '',
      sort_by: params.sort_by || 'popularity:desc',
      page: params.page || 1,
      per_page: Math.min(params.per_page || 20, 100),
      facet_by: params.facet_by || 'category,brand',
      max_facet_values: params.max_facet_values || 10,
      highlight_full_fields: 'name,description',
      snippet_threshold: 30,
    };

    return client.collections('products').documents().search(searchParameters);
  }

  static async vectorSearch(params: {
    vector: number[];
    per_page?: number;
    filter_by?: string;
  }) {
    const client = TypesenseSearchClient.getInstance();
    
    return client.collections('products').documents().search({
      vector_query: `embedding:([${params.vector.join(',')}])`,
      per_page: params.per_page || 10,
      filter_by: params.filter_by || '',
    });
  }

  static async hybridSearch(params: {
    q: string;
    vector: number[];
    per_page?: number;
    filter_by?: string;
  }) {
    const client = TypesenseSearchClient.getInstance();
    
    // Typesense supports hybrid search natively in newer versions
    return client.collections('products').documents().search({
      q: params.q,
      query_by: 'name,description,tags',
      vector_query: `embedding:([${params.vector.join(',')}])`,
      per_page: params.per_page || 20,
      filter_by: params.filter_by || '',
    });
  }
}

export const typesense = TypesenseSearchClient;

§ 3.2 COLLECTION SCHEMA DEFINITION
typescript
// lib/search/typesense-schema.ts
import { CollectionCreateSchema } from 'typesense/lib/Typesense/Collections';

export const ProductSchema: CollectionCreateSchema = {
  name: 'products',
  fields: [
    // Required fields
    { name: 'id', type: 'string', facet: false, index: true },
    
    // Text search fields
    { name: 'name', type: 'string', facet: false, index: true, infix: true },
    { name: 'description', type: 'string', facet: false, index: true },
    { name: 'sku', type: 'string', facet: false, index: true },
    
    // Facetable fields
    { name: 'category', type: 'string', facet: true, index: true, optional: true },
    { name: 'brand', type: 'string', facet: true, index: true, optional: true },
    { name: 'tags', type: 'string[]', facet: true, index: true, optional: true },
    
    // Numeric fields
    { name: 'price', type: 'float', facet: true, index: true },
    { name: 'rating', type: 'float', facet: true, index: true, optional: true },
    { name: 'popularity', type: 'int32', facet: true, index: true },
    
    // Boolean fields
    { name: 'in_stock', type: 'bool', facet: true, index: true },
    
    // Date fields (store as timestamp)
    { name: 'created_at', type: 'int64', facet: false, index: true },
    { name: 'updated_at', type: 'int64', facet: false, index: true },
    
    // Geo location fields
    { name: 'location', type: 'geopoint', facet: false, index: true, optional: true },
    
    // Vector embedding for semantic search
    { name: 'embedding', type: 'float[]', facet: false, index: true, optional: true },
    
    // Auto-complete/suggestion fields
    { name: 'name_suggest', type: 'string', facet: false, index: true, infix: true },
  ],
  
  // Default settings
  default_sorting_field: 'popularity',
  token_separators: ['-', '_', '.', '@'],
  symbols_to_index: ['!', '@', '#', '$', '%', '&', '*', '(', ')', '+', '='],
  
  // Enable nested documents if needed
  enable_nested_fields: true,
};

§ 3.3 TYPESENSE VS MEILISEARCH DECISION TABLE

| Aspect | Meilisearch | Typesense | Recommendation |
|--------|-------------|-----------|----------------|
| **Schema** | Schemaless (flexible) | Schema required (strict) | Meilisearch for rapid prototyping |
| **Geo search** | ✅ Built-in | ✅ Built-in | Both good |
| **Multi-tenancy** | Via separate indexes | Via API key scoping | Typesense for SaaS |
| **Vector search** | ✅ Hybrid mode | ✅ Native | Typesense for vector-first |
| **Ease of use** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Meilisearch better DX |
| **Scalability** | Horizontal scaling | Horizontal scaling | Both scale well |
| **Analytics** | Basic metrics | Detailed analytics | Typesense for analytics |
| **Pricing** | Predictable | Usage-based | Meilisearch for budget |
| **Community** | Large, growing | Smaller, dedicated | Meilisearch for support |
| **Admin UI** | Meilisearch UI | Typesense UI | Both good |
| **Typo tolerance** | Advanced config | Configurable | Meilisearch more flexible |
| **Synonyms** | ✅ | ✅ | Both good |
| **Language support** | 50+ languages | 40+ languages | Meilisearch wider |

**Decision Framework:**
1. **Choose Meilisearch if**: You want developer-friendly, schemaless, quick setup, good documentation.
2. **Choose Typesense if**: You need strict schema, advanced vector search, detailed analytics, or multi-tenant SaaS.
3. **Choose PostgreSQL FTS if**: You're already using Postgres, have simple needs, budget is tight.
4. **Choose Algolia if**: You need enterprise support, zero-ops, and have budget.

---

§ 4. POSTGRESQL FULL-TEXT SEARCH

§ 4.1 WHEN TO USE POSTGRESQL FTS

| Use Case | PostgreSQL FTS | Dedicated Engine | Recommendation |
|----------|----------------|------------------|----------------|
| **<100k documents** | ✅ Perfect | Overkill | PostgreSQL FTS |
| **Simple keyword search** | ✅ Good | Overkill | PostgreSQL FTS |
| **Need typo tolerance** | ❌ Not supported | ✅ Required | Dedicated engine |
| **Multi-language support** | ⚠️ Basic (stemming) | ✅ Advanced | Dedicated engine |
| **Real-time facets** | ❌ Manual queries | ✅ Built-in | Dedicated engine |
| **Budget constrained** | ✅ Free | $ Cost | PostgreSQL FTS |
| **Already using Postgres** | ✅ Integrated | Additional infra | PostgreSQL FTS |
| **Complex relevance scoring** | ❌ Limited | ✅ Advanced | Dedicated engine |

§ 4.2 IMPLEMENTATION
sql
-- SQL Schema for FTS
-- products table with FTS support

CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  description TEXT,
  sku TEXT UNIQUE,
  price DECIMAL(10, 2),
  category TEXT,
  brand TEXT,
  tags TEXT[],
  in_stock BOOLEAN DEFAULT true,
  rating DECIMAL(3, 2),
  popularity INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Add tsvector column for search
ALTER TABLE products ADD COLUMN search_vector tsvector;

-- Create GIN index for fast searching
CREATE INDEX idx_products_search_vector ON products USING gin(search_vector);

-- Create function to update search vector
CREATE OR REPLACE FUNCTION products_search_vector_update() RETURNS trigger AS $$
BEGIN
  NEW.search_vector :=
    setweight(to_tsvector('english', COALESCE(NEW.name, '')), 'A') ||
    setweight(to_tsvector('english', COALESCE(NEW.description, '')), 'B') ||
    setweight(to_tsvector('english', COALESCE(array_to_string(NEW.tags, ' '), '')), 'C');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger to auto-update search vector
CREATE TRIGGER products_search_vector_trigger
  BEFORE INSERT OR UPDATE ON products
  FOR EACH ROW
  EXECUTE FUNCTION products_search_vector_update();

-- Initialize search vector for existing rows
UPDATE products SET search_vector = NULL;

typescript
// lib/search/postgres-fts.ts
import { prisma } from '@/lib/prisma';
import { Prisma } from '@prisma/client';

export interface PostgresSearchParams {
  query: string;
  filters?: {
    category?: string[];
    brand?: string[];
    minPrice?: number;
    maxPrice?: number;
    inStock?: boolean;
  };
  sort?: 'relevance' | 'price_asc' | 'price_desc' | 'popularity' | 'newest';
  page?: number;
  perPage?: number;
}

export class PostgresFTS {
  static async searchProducts(params: PostgresSearchParams) {
    const page = params.page || 1;
    const perPage = Math.min(params.perPage || 20, 100);
    const offset = (page - 1) * perPage;

    // Build WHERE clause
    const whereConditions: string[] = [];
    const values: any[] = [];
    let paramIndex = 1;

    // Full-text search condition
    if (params.query) {
      whereConditions.push(`search_vector @@ plainto_tsquery('english', $${paramIndex})`);
      values.push(params.query);
      paramIndex++;
    }

    // Filter conditions
    if (params.filters?.category?.length) {
      whereConditions.push(`category = ANY($${paramIndex}::text[])`);
      values.push(params.filters.category);
      paramIndex++;
    }

    if (params.filters?.brand?.length) {
      whereConditions.push(`brand = ANY($${paramIndex}::text[])`);
      values.push(params.filters.brand);
      paramIndex++;
    }

    if (params.filters?.minPrice !== undefined) {
      whereConditions.push(`price >= $${paramIndex}`);
      values.push(params.filters.minPrice);
      paramIndex++;
    }

    if (params.filters?.maxPrice !== undefined) {
      whereConditions.push(`price <= $${paramIndex}`);
      values.push(params.filters.maxPrice);
      paramIndex++;
    }

    if (params.filters?.inStock !== undefined) {
      whereConditions.push(`in_stock = $${paramIndex}`);
      values.push(params.filters.inStock);
      paramIndex++;
    }

    // Build ORDER BY clause
    let orderBy = '';
    switch (params.sort) {
      case 'price_asc':
        orderBy = 'price ASC';
        break;
      case 'price_desc':
        orderBy = 'price DESC';
        break;
      case 'popularity':
        orderBy = 'popularity DESC';
        break;
      case 'newest':
        orderBy = 'created_at DESC';
        break;
      case 'relevance':
      default:
        orderBy = params.query 
          ? 'ts_rank(search_vector, plainto_tsquery(\'english\', $1)) DESC'
          : 'popularity DESC';
        break;
    }

    // Build final SQL query
    const whereClause = whereConditions.length 
      ? `WHERE ${whereConditions.join(' AND ')}` 
      : '';

    const countQuery = `
      SELECT COUNT(*) as total
      FROM products
      ${whereClause}
    `;

    const searchQuery = `
      SELECT 
        id,
        name,
        description,
        sku,
        price,
        category,
        brand,
        tags,
        in_stock as "inStock",
        rating,
        popularity,
        created_at as "createdAt",
        updated_at as "updatedAt",
        ${
          params.query
            ? 'ts_rank(search_vector, plainto_tsquery(\'english\', $1)) as rank'
            : '0 as rank'
        }
      FROM products
      ${whereClause}
      ORDER BY ${orderBy}
      LIMIT $${paramIndex} OFFSET $${paramIndex + 1}
    `;

    // Execute queries
    const [countResult, searchResult] = await Promise.all([
      prisma.$queryRawUnsafe<Array<{ total: bigint }>>(countQuery, ...values),
      prisma.$queryRawUnsafe<any[]>(searchQuery, ...[...values, perPage, offset]),
    ]);

    const total = Number(countResult[0]?.total || 0);
    const totalPages = Math.ceil(total / perPage);

    return {
      hits: searchResult,
      totalHits: total,
      page,
      totalPages,
      hitsPerPage: perPage,
    };
  }

  // For complex relevance scoring
  static async searchWithRelevance(params: {
    query: string;
    boost?: {
      name?: number;
      description?: number;
      tags?: number;
    };
  }) {
    const boost = {
      name: 1.0,
      description: 0.5,
      tags: 0.3,
      ...params.boost,
    };

    const result = await prisma.$queryRaw<any[]>`
      SELECT 
        id,
        name,
        description,
        sku,
        price,
        category,
        brand,
        tags,
        in_stock as "inStock",
        rating,
        popularity,
        created_at as "createdAt",
        updated_at as "updatedAt",
        (
          ${boost.name} * ts_rank(
            setweight(to_tsvector('english', COALESCE(name, '')), 'A'),
            plainto_tsquery('english', ${params.query})
          ) +
          ${boost.description} * ts_rank(
            setweight(to_tsvector('english', COALESCE(description, '')), 'B'),
            plainto_tsquery('english', ${params.query})
          ) +
          ${boost.tags} * ts_rank(
            setweight(to_tsvector('english', COALESCE(array_to_string(tags, ' '), '')), 'C'),
            plainto_tsquery('english', ${params.query})
          )
        ) as relevance_score
      FROM products
      WHERE 
        to_tsvector('english', COALESCE(name, '')) ||
        to_tsvector('english', COALESCE(description, '')) ||
        to_tsvector('english', COALESCE(array_to_string(tags, ' '), ''))
        @@ plainto_tsquery('english', ${params.query})
      ORDER BY relevance_score DESC
      LIMIT 20
    `;

    return result;
  }

  // Get facets (categories, brands, etc.)
  static async getFacets() {
    const [categories, brands, priceRange] = await Promise.all([
      prisma.$queryRaw<Array<{ category: string; count: number }>>`
        SELECT category, COUNT(*) as count
        FROM products
        WHERE category IS NOT NULL
        GROUP BY category
        ORDER BY count DESC
        LIMIT 20
      `,
      prisma.$queryRaw<Array<{ brand: string; count: number }>>`
        SELECT brand, COUNT(*) as count
        FROM products
        WHERE brand IS NOT NULL
        GROUP BY brand
        ORDER BY count DESC
        LIMIT 20
      `,
      prisma.$queryRaw<Array<{ min: number; max: number }>>`
        SELECT MIN(price) as min, MAX(price) as max
        FROM products
        WHERE price IS NOT NULL
      `,
    ]);

    return {
      categories: categories.map(c => ({
        value: c.category,
        count: Number(c.count),
      })),
      brands: brands.map(b => ({
        value: b.brand,
        count: Number(b.count),
      })),
      priceRange: {
        min: Number(priceRange[0]?.min || 0),
        max: Number(priceRange[0]?.max || 0),
      },
    };
  }
}

§ 4.3 SEARCH VECTOR UPDATE TRIGGER
sql
-- Enhanced trigger with better language support
CREATE OR REPLACE FUNCTION products_search_vector_update() RETURNS trigger AS $$
BEGIN
  -- Handle multiple languages (example: English and Spanish)
  NEW.search_vector :=
    -- English content (higher weight for names)
    setweight(to_tsvector('english', COALESCE(NEW.name, '')), 'A') ||
    setweight(to_tsvector('english', COALESCE(NEW.description, '')), 'B') ||
    setweight(to_tsvector('english', COALESCE(array_to_string(NEW.tags, ' '), '')), 'C') ||
    
    -- Spanish content (if you have Spanish data)
    setweight(to_tsvector('spanish', COALESCE(NEW.name_es, '')), 'A') ||
    setweight(to_tsvector('spanish', COALESCE(NEW.description_es, '')), 'B');
    
  -- Update timestamp
  NEW.updated_at = NOW();
  
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Add index for faster filtering
CREATE INDEX idx_products_category ON products(category);
CREATE INDEX idx_products_brand ON products(brand);
CREATE INDEX idx_products_price ON products(price);
CREATE INDEX idx_products_in_stock ON products(in_stock);

-- Materialized view for frequently accessed facets
CREATE MATERIALIZED VIEW product_facets AS
SELECT 
  category,
  brand,
  COUNT(*) as count,
  AVG(price) as avg_price,
  MIN(price) as min_price,
  MAX(price) as max_price
FROM products
WHERE category IS NOT NULL AND brand IS NOT NULL
GROUP BY category, brand;

-- Refresh materialized view periodically or on demand
CREATE OR REPLACE FUNCTION refresh_product_facets()
RETURNS TRIGGER AS $$
BEGIN
  REFRESH MATERIALIZED VIEW CONCURRENTLY product_facets;
  RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Trigger to refresh on product changes (optional - can be heavy)
-- CREATE TRIGGER refresh_facets_trigger
-- AFTER INSERT OR UPDATE OR DELETE ON products
-- FOR EACH STATEMENT
-- EXECUTE FUNCTION refresh_product_facets();

---

§ 5. SEARCH API DESIGN

§ 5.1 SEARCH REQUEST SCHEMA
typescript
// lib/schemas/search.ts
import { z } from 'zod';

export const SearchRequestSchema = z.object({
  // Required
  query: z.string().min(1).max(200).optional().or(z.literal('')),
  
  // Pagination
  page: z.coerce.number().int().positive().default(1),
  hitsPerPage: z.coerce.number().int().min(1).max(100).default(20),
  
  // Sorting
  sortBy: z.enum([
    'relevance',
    'price_asc',
    'price_desc',
    'popularity',
    'newest',
    'rating',
  ]).default('relevance'),
  
  // Filters (dynamic)
  filters: z.record(z.any()).optional().default({}),
  
  // Facets to return
  facets: z.array(z.string()).optional().default([]),
  
  // Advanced search options
  searchType: z.enum(['prefix', 'fulltext', 'hybrid', 'vector']).default('fulltext'),
  
  // Field boosting
  fieldBoosts: z.record(z.number()).optional().default({}),
  
  // Analytics
  analytics: z.object({
    sessionId: z.string().optional(),
    userId: z.string().optional(),
    source: z.string().optional(),
  }).optional(),
});

export type SearchRequest = z.infer<typeof SearchRequestSchema>;

// Individual filter schemas
export const CategoryFilterSchema = z.object({
  type: z.literal('category'),
  values: z.array(z.string()),
  operator: z.enum(['or', 'and']).default('or'),
});

export const PriceFilterSchema = z.object({
  type: z.literal('price'),
  min: z.number().optional(),
  max: z.number().optional(),
});

export const BooleanFilterSchema = z.object({
  type: z.literal('boolean'),
  field: z.string(),
  value: z.boolean(),
});

export const RangeFilterSchema = z.object({
  type: z.literal('range'),
  field: z.string(),
  min: z.number().optional(),
  max: z.number().optional(),
});

// Union of all filter types
export const FilterSchema = z.discriminatedUnion('type', [
  CategoryFilterSchema,
  PriceFilterSchema,
  BooleanFilterSchema,
  RangeFilterSchema,
]);

// Enhanced search request with typed filters
export const AdvancedSearchRequestSchema = SearchRequestSchema.extend({
  filters: z.array(FilterSchema).optional().default([]),
  highlight: z.object({
    enabled: z.boolean().default(true),
    preTag: z.string().default('<mark>'),
    postTag: z.string().default('</mark>'),
    maxLength: z.number().default(200),
  }).optional(),
  vectorQuery: z.array(z.number()).optional(), // For vector search
});

export type AdvancedSearchRequest = z.infer<typeof AdvancedSearchRequestSchema>;

§ 5.2 SEARCH RESPONSE SCHEMA
typescript
// lib/schemas/search-response.ts
import { z } from 'zod';

export const FacetValueSchema = z.object({
  value: z.string(),
  count: z.number(),
  highlighted: z.boolean().optional(),
});

export const FacetSchema = z.object({
  name: z.string(),
  values: z.array(FacetValueSchema),
  type: z.enum(['string', 'number', 'boolean', 'range']),
  stats: z.object({
    min: z.number().optional(),
    max: z.number().optional(),
    avg: z.number().optional(),
    sum: z.number().optional(),
  }).optional(),
});

export const SearchHitSchema = z.object({
  id: z.string(),
  _score: z.number().optional(),
  _highlight: z.record(z.array(z.string())).optional(),
  // Document data (type-specific)
  data: z.record(z.any()),
});

export const SearchResponseSchema = z.object({
  // Results
  hits: z.array(SearchHitSchema),
  totalHits: z.number(),
  
  // Pagination
  page: z.number(),
  totalPages: z.number(),
  hitsPerPage: z.number(),
  
  // Performance
  processingTimeMs: z.number(),
  server: z.string().optional(),
  index: z.string().optional(),
  
  // Query information
  query: z.string().optional(),
  parsedQuery: z.string().optional(),
  
  // Facets
  facets: z.record(FacetSchema).optional(),
  
  // Suggestions
  suggestions: z.array(z.string()).optional(),
  
  // Analytics
  searchId: z.string().optional(),
  
  // Metadata
  warnings: z.array(z.string()).optional(),
});

export type SearchResponse = z.infer<typeof SearchResponseSchema>;
export type Facet = z.infer<typeof FacetSchema>;
export type SearchHit = z.infer<typeof SearchHitSchema>;

// Helper function to create response
export function createSearchResponse<T>(
  data: Omit<SearchResponse, 'hits'> & { hits: T[] }
): SearchResponse {
  return {
    ...data,
    hits: data.hits.map((hit, index) => ({
      id: String((hit as any).id || index),
      _score: (hit as any)._score,
      _highlight: (hit as any)._highlight,
      data: hit,
    })),
  };
}

§ 5.3 API ROUTE IMPLEMENTATION
typescript
// app/api/search/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';
import { SearchRequestSchema, AdvancedSearchRequestSchema } from '@/lib/schemas/search';
import { createSearchResponse, SearchResponse } from '@/lib/schemas/search-response';
import { searchIndexer, SearchClient } from '@/lib/search/meilisearch';
import { rateLimit } from '@/lib/rate-limit';
import { trackSearch } from '@/lib/search/analytics';
import { requireAuth } from '@/lib/auth';

// Rate limiter: 100 requests per minute per IP
const limiter = rateLimit({
  interval: 60 * 1000, // 1 minute
  uniqueTokenPerInterval: 500,
});

// Health check endpoint
export async function GET(request: NextRequest) {
  try {
    const healthy = await SearchClient.healthCheck();
    
    if (!healthy) {
      return NextResponse.json(
        { error: 'Search service unavailable' },
        { status: 503 }
      );
    }
    
    return NextResponse.json({ status: 'ok', timestamp: new Date().toISOString() });
  } catch (error) {
    console.error('Health check failed:', error);
    return NextResponse.json(
      { error: 'Search service error' },
      { status: 500 }
    );
  }
}

// Search endpoint
export async function POST(request: NextRequest) {
  const startTime = Date.now();
  
  try {
    // Rate limiting
    const ip = request.headers.get('x-forwarded-for') || 'anonymous';
    const { success, limit, remaining, reset } = await limiter.check(10, ip);
    
    if (!success) {
      return NextResponse.json(
        { 
          error: 'Rate limit exceeded',
          limit,
          remaining,
          reset: new Date(reset).toISOString(),
        },
        { status: 429, headers: {
          'X-RateLimit-Limit': limit.toString(),
          'X-RateLimit-Remaining': remaining.toString(),
          'X-RateLimit-Reset': reset.toString(),
        } }
      );
    }

    // Parse and validate request
    const body = await request.json();
    
    // Check for advanced search
    const useAdvanced = body.filters && Array.isArray(body.filters);
    const schema = useAdvanced ? AdvancedSearchRequestSchema : SearchRequestSchema;
    
    const validation = schema.safeParse(body);
    
    if (!validation.success) {
      return NextResponse.json(
        { 
          error: 'Invalid search request',
          details: validation.error.errors,
        },
        { status: 400 }
      );
    }
    
    const searchRequest = validation.data;
    
    // Optional: Require auth for certain operations
    if (searchRequest.searchType === 'vector') {
      const auth = await requireAuth(request);
      if (!auth) {
        return NextResponse.json(
          { error: 'Authentication required for vector search' },
          { status: 401 }
        );
      }
    }
    
    // Determine search engine based on query
    const engine = await determineSearchEngine(searchRequest);
    
    // Execute search
    let searchResult: any;
    
    switch (engine) {
      case 'meilisearch':
        searchResult = await searchWithMeilisearch(searchRequest);
        break;
      case 'postgres':
        searchResult = await searchWithPostgres(searchRequest);
        break;
      default:
        throw new Error(`Unknown search engine: ${engine}`);
    }
    
    const processingTimeMs = Date.now() - startTime;
    
    // Build response
    const response: SearchResponse = createSearchResponse({
      hits: searchResult.hits,
      totalHits: searchResult.totalHits,
      page: searchRequest.page,
      totalPages: Math.ceil(searchResult.totalHits / searchRequest.hitsPerPage),
      hitsPerPage: searchRequest.hitsPerPage,
      processingTimeMs,
      query: searchRequest.query || '',
      facets: searchResult.facets,
      suggestions: searchResult.suggestions,
      searchId: crypto.randomUUID(),
      server: engine,
    });
    
    // Track analytics (async, don't await)
    trackSearch({
      searchId: response.searchId!,
      query: searchRequest.query || '',
      filters: searchRequest.filters,
      totalHits: response.totalHits,
      processingTimeMs,
      userId: body.analytics?.userId,
      sessionId: body.analytics?.sessionId,
      source: body.analytics?.source,
    }).catch(console.error);
    
    // Add rate limit headers
    const headers = {
      'X-RateLimit-Limit': limit.toString(),
      'X-RateLimit-Remaining': remaining.toString(),
      'X-RateLimit-Reset': reset.toString(),
      'X-Search-Engine': engine,
      'X-Processing-Time': processingTimeMs.toString(),
    };
    
    return NextResponse.json(response, { headers });
    
  } catch (error: any) {
    console.error('Search API error:', error);
    
    // Distinguish between client and server errors
    const status = error.statusCode || 500;
    const message = status === 500 ? 'Internal server error' : error.message;
    
    return NextResponse.json(
      { 
        error: message,
        requestId: crypto.randomUUID(),
        timestamp: new Date().toISOString(),
      },
      { status }
    );
  }
}

// Helper functions
async function determineSearchEngine(request: any): Promise<'meilisearch' | 'postgres'> {
  // Simple heuristic: use Meilisearch for complex queries, Postgres for simple
  if (
    !request.query || 
    (request.filters && Object.keys(request.filters).length === 0) ||
    request.searchType === 'vector'
  ) {
    return 'meilisearch';
  }
  
  // For very simple queries with few documents, Postgres might be fine
  // In practice, usually default to Meilisearch
  return 'meilisearch';
}

async function searchWithMeilisearch(request: any) {
  const index = SearchClient.getIndex('products');
  
  const searchParams: any = {
    q: request.query || '',
    page: request.page,
    hitsPerPage: request.hitsPerPage,
  };
  
  // Add filters if present
  if (request.filters && Object.keys(request.filters).length > 0) {
    if (Array.isArray(request.filters)) {
      // Advanced filters
      const filterStrings = request.filters.map((filter: any) => {
        switch (filter.type) {
          case 'category':
            return `${filter.field} IN [${filter.values.map((v: string) => `"${v}"`).join(', ')}]`;
          case 'price':
            const conditions = [];
            if (filter.min !== undefined) conditions.push(`${filter.field} >= ${filter.min}`);
            if (filter.max !== undefined) conditions.push(`${filter.field} <= ${filter.max}`);
            return conditions.join(' AND ');
          case 'boolean':
            return `${filter.field} = ${filter.value}`;
          default:
            return '';
        }
      }).filter(Boolean);
      
      if (filterStrings.length > 0) {
        searchParams.filter = filterStrings.join(' AND ');
      }
    } else {
      // Simple filters
      const filterStrings = Object.entries(request.filters)
        .map(([key, value]) => {
          if (Array.isArray(value)) {
            return `${key} IN [${value.map(v => `"${v}"`).join(', ')}]`;
          }
          return `${key} = "${value}"`;
        });
      
      if (filterStrings.length > 0) {
        searchParams.filter = filterStrings.join(' AND ');
      }
    }
  }
  
  // Add facets if requested
  if (request.facets && request.facets.length > 0) {
    searchParams.facets = request.facets;
  }
  
  // Execute search
  const result = await index.search(searchParams.q, searchParams);
  
  return {
    hits: result.hits,
    totalHits: result.estimatedTotalHits,
    facets: result.facetDistribution,
    suggestions: [], // Meilisearch doesn't provide suggestions in search results
  };
}

async function searchWithPostgres(request: any) {
  // Implementation using PostgresFTS class
  // ...
  return { hits: [], totalHits: 0, facets: {} };
}

// Batch search endpoint (for multiple queries)
export async function PUT(request: NextRequest) {
  try {
    const body = await request.json();
    const { queries } = z.object({
      queries: z.array(SearchRequestSchema).max(10),
    }).parse(body);
    
    const results = await Promise.all(
      queries.map(query => searchWithMeilisearch(query))
    );
    
    return NextResponse.json({ results });
  } catch (error) {
    return NextResponse.json(
      { error: 'Batch search failed' },
      { status: 400 }
    );
  }
}

---

§ 6. SEARCH UI COMPONENTS

§ 6.1 COMPONENT ARCHITECTURE

| Component | Purpose | Key Props | State | Events |
|-----------|---------|-----------|-------|--------|
| SearchInput | Query entry | placeholder, value, size | internal | onSearch, onChange, onClear |
| SearchResults | Display hits | hits, loading, emptyState | external | onHitClick, onLoadMore |
| Facets | Filter controls | facets, selected, layout | external | onFacetChange, onClearAll |
| Pagination | Page navigation | page, totalPages, size | external | onPageChange |
| SearchHighlight | Highlight matches | text, query, tag | none | none |
| InstantSearch | All-in-one | config, children | internal | onStateChange |
| SearchStats | Result stats | totalHits, time, query | none | none |

§ 6.2 SEARCH INPUT WITH DEBOUNCE
typescript
// components/search/SearchInput.tsx
'use client';

import React, { useState, useEffect, useCallback, useRef } from 'react';
import { Search, X, Command } from 'lucide-react';
import { cn } from '@/lib/utils';
import { useSearch } from '@/hooks/useSearch';
import { useKeyboardShortcut } from '@/hooks/useKeyboardShortcut';

export interface SearchInputProps {
  placeholder?: string;
  defaultValue?: string;
  debounceMs?: number;
  autoFocus?: boolean;
  showShortcut?: boolean;
  size?: 'sm' | 'md' | 'lg';
  className?: string;
  onSearch?: (query: string) => void;
  onChange?: (query: string) => void;
  onClear?: () => void;
  disabled?: boolean;
}

export function SearchInput({
  placeholder = 'Search...',
  defaultValue = '',
  debounceMs = 300,
  autoFocus = false,
  showShortcut = true,
  size = 'md',
  className,
  onSearch,
  onChange,
  onClear,
  disabled = false,
}: SearchInputProps) {
  const [query, setQuery] = useState(defaultValue);
  const [isComposing, setIsComposing] = useState(false);
  const inputRef = useRef<HTMLInputElement>(null);
  
  // Debounced search
  const debouncedSearch = useCallback(
    useDebouncedCallback((value: string) => {
      if (onSearch && value.trim()) {
        onSearch(value);
      }
    }, debounceMs),
    [onSearch, debounceMs]
  );

  // Handle input change
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    setQuery(value);
    
    if (onChange) {
      onChange(value);
    }
    
    if (!isComposing) {
      debouncedSearch(value);
    }
  };

  // Handle composition (for IME inputs)
  const handleCompositionStart = () => setIsComposing(true);
  const handleCompositionEnd = (e: React.CompositionEvent<HTMLInputElement>) => {
    setIsComposing(false);
    const value = e.currentTarget.value;
    debouncedSearch(value);
  };

  // Handle clear
  const handleClear = () => {
    setQuery('');
    if (onClear) onClear();
    if (onSearch) onSearch('');
    inputRef.current?.focus();
  };

  // Handle submit (Enter key)
  const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter' && !e.shiftKey && !isComposing) {
      e.preventDefault();
      if (onSearch) {
        onSearch(query);
      }
    }
    
    // Escape clears the input
    if (e.key === 'Escape' && query) {
      e.preventDefault();
      handleClear();
    }
  };

  // Keyboard shortcut (Cmd+K / Ctrl+K)
  useKeyboardShortcut({
    key: 'k',
    ctrlKey: true,
    metaKey: true,
    callback: () => {
      inputRef.current?.focus();
      inputRef.current?.select();
    },
  });

  // Sync with external value changes
  useEffect(() => {
    if (defaultValue !== query) {
      setQuery(defaultValue);
    }
  }, [defaultValue]);

  // Size classes
  const sizeClasses = {
    sm: 'h-8 text-sm px-3',
    md: 'h-10 text-base px-4',
    lg: 'h-12 text-lg px-6',
  };

  return (
    <div className={cn('relative w-full', className)}>
      <div className="relative flex items-center">
        <Search className={cn(
          'absolute left-3 text-muted-foreground',
          size === 'sm' ? 'h-4 w-4' : 'h-5 w-5'
        )} />
        
        <input
          ref={inputRef}
          type="search"
          value={query}
          onChange={handleChange}
          onKeyDown={handleKeyDown}
          onCompositionStart={handleCompositionStart}
          onCompositionEnd={handleCompositionEnd}
          placeholder={placeholder}
          autoFocus={autoFocus}
          disabled={disabled}
          autoComplete="off"
          autoCorrect="off"
          autoCapitalize="off"
          spellCheck={false}
          aria-label="Search"
          className={cn(
            'w-full pl-10 pr-10 rounded-lg border border-input bg-background',
            'focus:outline-none focus:ring-2 focus:ring-ring focus:border-transparent',
            'disabled:opacity-50 disabled:cursor-not-allowed',
            'placeholder:text-muted-foreground',
            sizeClasses[size],
            className
          )}
        />
        
        <div className="absolute right-3 flex items-center gap-2">
          {query && (
            <button
              type="button"
              onClick={handleClear}
              className="p-0.5 rounded-sm hover:bg-accent"
              aria-label="Clear search"
            >
              <X className="h-4 w-4 text-muted-foreground" />
            </button>
          )}
          
          {showShortcut && !query && (
            <div className="hidden sm:flex items-center gap-1 text-xs text-muted-foreground border rounded px-1.5 py-0.5">
              <Command className="h-3 w-3" />
              <span>K</span>
            </div>
          )}
        </div>
      </div>
    </div>
  );
}

// Custom debounce hook
function useDebouncedCallback<T extends (...args: any[]) => any>(
  callback: T,
  delay: number
): (...args: Parameters<T>) => void {
  const timeoutRef = useRef<NodeJS.Timeout>();

  return useCallback(
    (...args: Parameters<T>) => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }

      timeoutRef.current = setTimeout(() => {
        callback(...args);
      }, delay);
    },
    [callback, delay]
  );
}

// Hook for keyboard shortcuts
export function useKeyboardShortcut(options: {
  key: string;
  ctrlKey?: boolean;
  metaKey?: boolean;
  shiftKey?: boolean;
  altKey?: boolean;
  callback: () => void;
}) {
  const { key, ctrlKey, metaKey, shiftKey, altKey, callback } = options;

  useEffect(() => {
    const handler = (e: KeyboardEvent) => {
      if (
        e.key.toLowerCase() === key.toLowerCase() &&
        !!e.ctrlKey === !!ctrlKey &&
        !!e.metaKey === !!metaKey &&
        !!e.shiftKey === !!shiftKey &&
        !!e.altKey === !!altKey
      ) {
        e.preventDefault();
        callback();
      }
    };

    window.addEventListener('keydown', handler);
    return () => window.removeEventListener('keydown', handler);
  }, [key, ctrlKey, metaKey, shiftKey, altKey, callback]);
}

§ 6.3 SEARCH RESULTS COMPONENT
typescript
// components/search/SearchResults.tsx
'use client';

import React from 'react';
import { cn } from '@/lib/utils';
import { SearchHit } from '@/lib/schemas/search-response';
import { SearchHighlight } from './SearchHighlight';
import { Skeleton } from '@/components/ui/skeleton';
import { AlertCircle, Search } from 'lucide-react';

export interface SearchResultsProps<T = any> {
  hits: SearchHit<T>[];
  loading?: boolean;
  emptyState?: React.ReactNode;
  query?: string;
  className?: string;
  hitClassName?: string;
  renderHit: (hit: SearchHit<T>, index: number) => React.ReactNode;
  onHitClick?: (hit: SearchHit<T>) => void;
  onLoadMore?: () => void;
  hasMore?: boolean;
  loadingMore?: boolean;
}

export function SearchResults<T>({
  hits,
  loading = false,
  emptyState,
  query = '',
  className,
  hitClassName,
  renderHit,
  onHitClick,
  onLoadMore,
  hasMore = false,
  loadingMore = false,
}: SearchResultsProps<T>) {
  // Loading state
  if (loading) {
    return (
      <div className={cn('space-y-4', className)}>
        {Array.from({ length: 5 }).map((_, i) => (
          <div key={i} className={cn('border rounded-lg p-4', hitClassName)}>
            <Skeleton className="h-6 w-3/4 mb-2" />
            <Skeleton className="h-4 w-full mb-1" />
            <Skeleton className="h-4 w-5/6" />
          </div>
        ))}
      </div>
    );
  }

  // Empty state
  if (hits.length === 0 && !loading) {
    return emptyState || (
      <div className={cn('flex flex-col items-center justify-center py-12 text-center', className)}>
        <div className="w-12 h-12 rounded-full bg-muted flex items-center justify-center mb-4">
          <Search className="h-6 w-6 text-muted-foreground" />
        </div>
        <h3 className="text-lg font-medium mb-2">No results found</h3>
        <p className="text-muted-foreground max-w-md">
          {query ? `No results found for "${query}". Try different keywords or check for typos.` : 'Enter a search term to find results.'}
        </p>
      </div>
    );
  }

  // Error state (if hits is null/undefined)
  if (!hits) {
    return (
      <div className={cn('flex flex-col items-center justify-center py-12', className)}>
        <AlertCircle className="h-12 w-12 text-destructive mb-4" />
        <h3 className="text-lg font-medium mb-2">Search Error</h3>
        <p className="text-muted-foreground">Unable to load search results. Please try again.</p>
      </div>
    );
  }

  return (
    <div className={className}>
      {/* Results list */}
      <div className="space-y-3">
        {hits.map((hit, index) => (
          <div
            key={hit.id}
            className={cn(
              'border rounded-lg p-4 hover:border-primary/50 transition-colors cursor-pointer',
              hitClassName
            )}
            onClick={() => onHitClick?.(hit)}
            role="button"
            tabIndex={0}
            onKeyDown={(e) => {
              if (e.key === 'Enter' || e.key === ' ') {
                onHitClick?.(hit);
              }
            }}
          >
            {renderHit(hit, index)}
          </div>
        ))}
      </div>

      {/* Load more button */}
      {hasMore && onLoadMore && (
        <div className="mt-6 text-center">
          <button
            onClick={onLoadMore}
            disabled={loadingMore}
            className="px-4 py-2 border rounded-lg hover:bg-accent transition-colors disabled:opacity-50"
          >
            {loadingMore ? 'Loading...' : 'Load more results'}
          </button>
        </div>
      )}
    </div>
  );
}

// Individual hit component
export function SearchHitCard<T extends { name: string; description?: string; [key: string]: any }>({
  hit,
  query = '',
  fields = ['name', 'description'],
}: {
  hit: SearchHit<T>;
  query: string;
  fields?: string[];
}) {
  return (
    <div className="space-y-2">
      {/* Title with highlighting */}
      {hit.data.name && (
        <h3 className="text-lg font-semibold">
          <SearchHighlight text={hit.data.name} query={query} />
        </h3>
      )}

      {/* Description with highlighting */}
      {hit.data.description && (
        <p className="text-muted-foreground line-clamp-2">
          <SearchHighlight text={hit.data.description} query={query} maxLength={200} />
        </p>
      )}

      {/* Score and metadata */}
      <div className="flex items-center gap-4 text-sm text-muted-foreground pt-2">
        {hit._score !== undefined && (
          <span className="bg-muted px-2 py-0.5 rounded text-xs">
            Score: {hit._score.toFixed(2)}
          </span>
        )}
        
        {/* Custom metadata */}
        {hit.data.category && (
          <span className="px-2 py-0.5 bg-primary/10 text-primary rounded text-xs">
            {hit.data.category}
          </span>
        )}
        
        {hit.data.price && (
          <span className="font-medium">
            ${hit.data.price.toFixed(2)}
          </span>
        )}
      </div>
    </div>
  );
}

// Highlight component
export function SearchHighlight({
  text,
  query,
  tag: Tag = 'mark',
  className,
  maxLength,
}: {
  text: string;
  query: string;
  tag?: string | React.ComponentType<any>;
  className?: string;
  maxLength?: number;
}) {
  if (!query || !text) {
    return <>{text}</>;
  }

  const queryWords = query.toLowerCase().split(/\s+/).filter(Boolean);
  
  // Truncate text if needed
  let displayText = text;
  let prefix = '';
  let suffix = '';
  
  if (maxLength && text.length > maxLength) {
    // Find a position near a match to center the highlight
    const firstMatchIndex = text.toLowerCase().indexOf(queryWords[0]);
    if (firstMatchIndex > -1) {
      const start = Math.max(0, firstMatchIndex - Math.floor(maxLength / 2));
      const end = Math.min(text.length, start + maxLength);
      
      displayText = text.substring(start, end);
      prefix = start > 0 ? '...' : '';
      suffix = end < text.length ? '...' : '';
    } else {
      displayText = text.substring(0, maxLength) + '...';
    }
  }

  // Create highlighted segments
  const segments: Array<{ text: string; highlight: boolean }> = [];
  let lastIndex = 0;
  
  // For each query word, find matches
  for (const word of queryWords) {
    const regex = new RegExp(`(${escapeRegExp(word)})`, 'gi');
    let match;
    
    while ((match = regex.exec(displayText.toLowerCase())) !== null) {
      // Add text before match
      if (match.index > lastIndex) {
        segments.push({
          text: displayText.substring(lastIndex, match.index),
          highlight: false,
        });
      }
      
      // Add matched text
      segments.push({
        text: displayText.substring(match.index, match.index + match[0].length),
        highlight: true,
      });
      
      lastIndex = match.index + match[0].length;
    }
  }
  
  // Add remaining text
  if (lastIndex < displayText.length) {
    segments.push({
      text: displayText.substring(lastIndex),
      highlight: false,
    });
  }

  return (
    <>
      {prefix}
      {segments.map((segment, i) => {
        if (segment.highlight) {
          return React.createElement(Tag, {
            key: i,
            className: cn('bg-yellow-100 dark:bg-yellow-900', className),
          }, segment.text);
        }
        return <span key={i}>{segment.text}</span>;
      })}
      {suffix}
    </>
  );
}

function escapeRegExp(string: string) {
  return string.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
}

// Skeleton loader
export function SearchResultsSkeleton({ count = 5 }: { count?: number }) {
  return (
    <div className="space-y-4">
      {Array.from({ length: count }).map((_, i) => (
        <div key={i} className="border rounded-lg p-4">
          <Skeleton className="h-6 w-3/4 mb-2" />
          <Skeleton className="h-4 w-full mb-1" />
          <Skeleton className="h-4 w-5/6" />
        </div>
      ))}
    </div>
  );
}

§ 6.4 FACETED SEARCH
typescript
// components/search/Facets.tsx
'use client';

import React, { useState } from 'react';
import { cn } from '@/lib/utils';
import { Facet } from '@/lib/schemas/search-response';
import { ChevronDown, ChevronUp, Filter, X } from 'lucide-react';
import { Slider } from '@/components/ui/slider';
import { Checkbox } from '@/components/ui/checkbox';

export interface FacetsProps {
  facets: Record<string, Facet>;
  selected: Record<string, any>;
  onFacetChange: (facetName: string, value: any) => void;
  onClearAll?: () => void;
  className?: string;
  maxFacetValues?: number;
  layout?: 'vertical' | 'horizontal' | 'accordion';
  showCounts?: boolean;
}

export function Facets({
  facets,
  selected,
  onFacetChange,
  onClearAll,
  className,
  maxFacetValues = 10,
  layout = 'vertical',
  showCounts = true,
}: FacetsProps) {
  const [expandedFacets, setExpandedFacets] = useState<Set<string>>(new Set());
  
  const toggleFacet = (facetName: string) => {
    const newExpanded = new Set(expandedFacets);
    if (newExpanded.has(facetName)) {
      newExpanded.delete(facetName);
    } else {
      newExpanded.add(facetName);
    }
    setExpandedFacets(newExpanded);
  };
  
  const handleStringFacetChange = (facetName: string, value: string) => {
    const current = selected[facetName] || [];
    const newValues = current.includes(value)
      ? current.filter((v: string) => v !== value)
      : [...current, value];
    
    onFacetChange(facetName, newValues);
  };
  
  const handleRangeFacetChange = (facetName: string, min: number, max: number) => {
    onFacetChange(facetName, { min, max });
  };
  
  const handleBooleanFacetChange = (facetName: string, value: boolean) => {
    onFacetChange(facetName, value);
  };
  
  const clearFacet = (facetName: string) => {
    onFacetChange(facetName, undefined);
  };
  
  const renderFacet = (facetName: string, facet: Facet) => {
    const isExpanded = expandedFacets.has(facetName) || layout !== 'accordion';
    const selectedValues = selected[facetName];
    
    return (
      <div key={facetName} className="border rounded-lg p-4">
        <div 
          className="flex items-center justify-between cursor-pointer"
          onClick={() => layout === 'accordion' && toggleFacet(facetName)}
        >
          <div className="flex items-center gap-2">
            <Filter className="h-4 w-4 text-muted-foreground" />
            <h3 className="font-medium capitalize">{facetName.replace('_', ' ')}</h3>
            {selectedValues && (
              <span className="text-xs bg-primary text-primary-foreground px-1.5 py-0.5 rounded">
                {Array.isArray(selectedValues) ? selectedValues.length : 1}
              </span>
            )}
          </div>
          
          {layout === 'accordion' && (
            <button className="p-1 hover:bg-accent rounded">
              {isExpanded ? (
                <ChevronUp className="h-4 w-4" />
              ) : (
                <ChevronDown className="h-4 w-4" />
              )}
            </button>
          )}
        </div>
        
        {isExpanded && (
          <div className="mt-3">
            {renderFacetContent(facetName, facet, selectedValues)}
            
            {selectedValues && (
              <button
                onClick={() => clearFacet(facetName)}
                className="mt-2 text-sm text-muted-foreground hover:text-foreground"
              >
                Clear
              </button>
            )}
          </div>
        )}
      </div>
    );
  };
  
  const renderFacetContent = (facetName: string, facet: Facet, selectedValues: any) => {
    switch (facet.type) {
      case 'string':
        return renderStringFacet(facetName, facet, selectedValues);
      case 'number':
      case 'range':
        return renderRangeFacet(facetName, facet, selectedValues);
      case 'boolean':
        return renderBooleanFacet(facetName, facet, selectedValues);
      default:
        return null;
    }
  };
  
  const renderStringFacet = (facetName: string, facet: Facet, selectedValues: string[] = []) => {
    // Sort by count descending
    const sortedValues = [...facet.values].sort((a, b) => b.count - a.count);
    // Take top N values
    const visibleValues = sortedValues.slice(0, maxFacetValues);
    const hasMore = sortedValues.length > maxFacetValues;
    
    return (
      <div className="space-y-2">
        {visibleValues.map((value) => (
          <div key={value.value} className="flex items-center gap-2">
            <Checkbox
              id={`${facetName}-${value.value}`}
              checked={selectedValues.includes(value.value)}
              onCheckedChange={() => handleStringFacetChange(facetName, value.value)}
            />
            <label
              htmlFor={`${facetName}-${value.value}`}
              className="text-sm cursor-pointer flex-1"
            >
              <span>{value.value}</span>
              {showCounts && (
                <span className="text-muted-foreground ml-2">
                  ({value.count.toLocaleString()})
                </span>
              )}
            </label>
          </div>
        ))}
        
        {hasMore && (
          <div className="pt-2 text-sm text-muted-foreground">
            +{sortedValues.length - maxFacetValues} more
          </div>
        )}
      </div>
    );
  };
  
  const renderRangeFacet = (facetName: string, facet: Facet, selectedValues: any) => {
    const stats = facet.stats;
    if (!stats?.min || !stats?.max) return null;
    
    const min = stats.min;
    const max = stats.max;
    const currentMin = selectedValues?.min || min;
    const currentMax = selectedValues?.max || max;
    
    return (
      <div className="space-y-4">
        <Slider
          min={min}
          max={max}
          step={1}
          value={[currentMin, currentMax]}
          onValueChange={([newMin, newMax]) => {
            handleRangeFacetChange(facetName, newMin, newMax);
          }}
          className="my-6"
        />
        
        <div className="flex items-center justify-between gap-4">
          <div className="flex-1">
            <label className="text-xs text-muted-foreground block mb-1">Min</label>
            <input
              type="number"
              min={min}
              max={currentMax}
              value={currentMin}
              onChange={(e) => handleRangeFacetChange(
                facetName,
                Number(e.target.value),
                currentMax
              )}
              className="w-full border rounded px-2 py-1 text-sm"
            />
          </div>
          <div className="flex-1">
            <label className="text-xs text-muted-foreground block mb-1">Max</label>
            <input
              type="number"
              min={currentMin}
              max={max}
              value={currentMax}
              onChange={(e) => handleRangeFacetChange(
                facetName,
                currentMin,
                Number(e.target.value)
              )}
              className="w-full border rounded px-2 py-1 text-sm"
            />
          </div>
        </div>
      </div>
    );
  };
  
  const renderBooleanFacet = (facetName: string, facet: Facet, selectedValue: boolean) => {
    const trueValue = facet.values.find(v => v.value === 'true');
    const falseValue = facet.values.find(v => v.value === 'false');
    
    return (
      <div className="space-y-2">
        {trueValue && (
          <div className="flex items-center gap-2">
            <Checkbox
              id={`${facetName}-true`}
              checked={selectedValue === true}
              onCheckedChange={() => handleBooleanFacetChange(facetName, true)}
            />
            <label htmlFor={`${facetName}-true`} className="text-sm cursor-pointer">
              Yes {showCounts && `(${trueValue.count.toLocaleString()})`}
            </label>
          </div>
        )}
        
        {falseValue && (
          <div className="flex items-center gap-2">
            <Checkbox
              id={`${facetName}-false`}
              checked={selectedValue === false}
              onCheckedChange={() => handleBooleanFacetChange(facetName, false)}
            />
            <label htmlFor={`${facetName}-false`} className="text-sm cursor-pointer">
              No {showCounts && `(${falseValue.count.toLocaleString()})`}
            </label>
          </div>
        )}
      </div>
    );
  };
  
  // Active filters display
  const activeFilters = Object.entries(selected)
    .filter(([_, value]) => value !== undefined && value !== null)
    .map(([facetName, value]) => ({ facetName, value }));
  
  if (Object.keys(facets).length === 0 && activeFilters.length === 0) {
    return null;
  }
  
  return (
    <div className={cn('space-y-4', className)}>
      {/* Active filters */}
      {activeFilters.length > 0 && (
        <div className="border rounded-lg p-4">
          <div className="flex items-center justify-between mb-3">
            <h3 className="font-medium">Active Filters</h3>
            {onClearAll && (
              <button
                onClick={onClearAll}
                className="text-sm text-muted-foreground hover:text-foreground"
              >
                Clear all
              </button>
            )}
          </div>
          
          <div className="flex flex-wrap gap-2">
            {activeFilters.map(({ facetName, value }) => {
              let displayValue = '';
              
              if (Array.isArray(value)) {
                displayValue = value.join(', ');
              } else if (typeof value === 'object' && value !== null) {
                if (value.min !== undefined && value.max !== undefined) {
                  displayValue = `${value.min} - ${value.max}`;
                }
              } else {
                displayValue = String(value);
              }
              
              return (
                <div
                  key={facetName}
                  className="flex items-center gap-1 bg-accent text-accent-foreground px-3 py-1 rounded-full text-sm"
                >
                  <span className="capitalize">{facetName.replace('_', ' ')}:</span>
                  <span className="font-medium">{displayValue}</span>
                  <button
                    onClick={() => clearFacet(facetName)}
                    className="ml-1 hover:bg-accent/50 rounded-full p-0.5"
                  >
                    <X className="h-3 w-3" />
                  </button>
                </div>
              );
            })}
          </div>
        </div>
      )}
      
      {/* Facets */}
      {Object.keys(facets).length > 0 && (
        <div className="space-y-4">
          {Object.entries(facets).map(([facetName, facet]) => 
            renderFacet(facetName, facet)
          )}
        </div>
      )}
    </div>
  );
}

// Mobile facets drawer
export function MobileFacetsDrawer({
  facets,
  selected,
  onFacetChange,
  onClearAll,
  open,
  onOpenChange,
}: FacetsProps & {
  open: boolean;
  onOpenChange: (open: boolean) => void;
}) {
  return (
    <>
      {/* Trigger button */}
      <button
        onClick={() => onOpenChange(true)}
        className="flex items-center gap-2 px-4 py-2 border rounded-lg hover:bg-accent"
      >
        <Filter className="h-4 w-4" />
        <span>Filters</span>
        {Object.values(selected).filter(v => v !== undefined).length > 0 && (
          <span className="bg-primary text-primary-foreground text-xs px-1.5 py-0.5 rounded">
            {Object.values(selected).filter(v => v !== undefined).length}
          </span>
        )}
      </button>
      
      {/* Drawer overlay */}
      {open && (
        <div className="fixed inset-0 z-50 bg-background/80 backdrop-blur-sm">
          <div className="fixed inset-y-0 right-0 w-full max-w-sm bg-background border-l shadow-lg">
            <div className="h-full flex flex-col">
              {/* Header */}
              <div className="border-b p-4 flex items-center justify-between">
                <h2 className="text-lg font-semibold">Filters</h2>
                <button
                  onClick={() => onOpenChange(false)}
                  className="p-2 hover:bg-accent rounded"
                >
                  <X className="h-5 w-5" />
                </button>
              </div>
              
              {/* Content */}
              <div className="flex-1 overflow-y-auto p-4">
                <Facets
                  facets={facets}
                  selected={selected}
                  onFacetChange={onFacetChange}
                  onClearAll={onClearAll}
                  layout="accordion"
                />
              </div>
              
              {/* Footer */}
              <div className="border-t p-4 flex gap-3">
                <button
                  onClick={onClearAll}
                  className="flex-1 py-2 border rounded-lg hover:bg-accent"
                >
                  Clear all
                </button>
                <button
                  onClick={() => onOpenChange(false)}
                  className="flex-1 py-2 bg-primary text-primary-foreground rounded-lg hover:bg-primary/90"
                >
                  Apply filters
                </button>
              </div>
            </div>
          </div>
        </div>
      )}
    </>
  );
}

§ 6.5 SEARCH COMMAND PALETTE (CMD+K)
typescript
// components/search/CommandPalette.tsx
'use client';

import React, { useState, useEffect, useCallback } from 'react';
import { Dialog } from '@/components/ui/dialog';
import { Search, Clock, TrendingUp, File, User, Command } from 'lucide-react';
import { useRouter } from 'next/navigation';
import { useSearch } from '@/hooks/useSearch';
import { cn } from '@/lib/utils';

export interface CommandPaletteProps {
  open?: boolean;
  onOpenChange?: (open: boolean) => void;
  recentSearches?: string[];
  popularSearches?: Array<{ query: string; count: number }>;
  quickActions?: Array<{
    id: string;
    label: string;
    icon?: React.ReactNode;
    keywords?: string[];
    onSelect?: () => void;
  }>;
  maxRecentSearches?: number;
}

export function CommandPalette({
  open: controlledOpen,
  onOpenChange: controlledOnOpenChange,
  recentSearches: initialRecentSearches = [],
  popularSearches = [],
  quickActions = [],
  maxRecentSearches = 10,
}: CommandPaletteProps) {
  const [uncontrolledOpen, setUncontrolledOpen] = useState(false);
  const [search, setSearch] = useState('');
  const [selectedIndex, setSelectedIndex] = useState(0);
  const [recentSearches, setRecentSearches] = useState<string[]>(initialRecentSearches);
  const router = useRouter();
  
  const open = controlledOpen ?? uncontrolledOpen;
  const onOpenChange = controlledOnOpenChange ?? setUncontrolledOpen;
  
  // Load recent searches from localStorage
  useEffect(() => {
    const saved = localStorage.getItem('recent-searches');
    if (saved) {
      try {
        setRecentSearches(JSON.parse(saved));
      } catch {
        // Ignore parse errors
      }
    }
  }, []);
  
  // Save recent searches
  const saveRecentSearch = useCallback((query: string) => {
    if (!query.trim()) return;
    
    const updated = [
      query,
      ...recentSearches.filter(q => q !== query),
    ].slice(0, maxRecentSearches);
    
    setRecentSearches(updated);
    localStorage.setItem('recent-searches', JSON.stringify(updated));
  }, [recentSearches, maxRecentSearches]);
  
  // Keyboard shortcut (Cmd+K / Ctrl+K)
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      if ((e.metaKey || e.ctrlKey) && e.key === 'k') {
        e.preventDefault();
        onOpenChange(!open);
      }
      
      if (e.key === 'Escape' && open) {
        onOpenChange(false);
      }
    };
    
    window.addEventListener('keydown', handleKeyDown);
    return () => window.removeEventListener('keydown', handleKeyDown);
  }, [open, onOpenChange]);
  
  // Filter items based on search
  const filteredItems = React.useMemo(() => {
    const searchLower = search.toLowerCase();
    const items = [];
    
    // Recent searches
    if (!search || recentSearches.some(q => q.toLowerCase().includes(searchLower))) {
      items.push({
        type: 'section',
        label: 'Recent Searches',
        icon: <Clock className="h-4 w-4" />,
      });
      
      recentSearches
        .filter(q => !search || q.toLowerCase().includes(searchLower))
        .forEach((query, index) => {
          items.push({
            type: 'recent',
            query,
            index: items.length,
          });
        });
    }
    
    // Popular searches
    if (!search || popularSearches.some(p => p.query.toLowerCase().includes(searchLower))) {
      items.push({
        type: 'section',
        label: 'Popular Searches',
        icon: <TrendingUp className="h-4 w-4" />,
      });
      
      popularSearches
        .filter(p => !search || p.query.toLowerCase().includes(searchLower))
        .forEach(({ query, count }, index) => {
          items.push({
            type: 'popular',
            query,
            count,
            index: items.length,
          });
        });
    }
    
    // Quick actions
    if (quickActions.length > 0 && (!search || quickActions.some(a => 
      a.label.toLowerCase().includes(searchLower) || 
      a.keywords?.some(k => k.toLowerCase().includes(searchLower))
    ))) {
      items.push({
        type: 'section',
        label: 'Quick Actions',
        icon: <Command className="h-4 w-4" />,
      });
      
      quickActions
        .filter(a => !search || 
          a.label.toLowerCase().includes(searchLower) ||
          a.keywords?.some(k => k.toLowerCase().includes(searchLower))
        )
        .forEach((action, index) => {
          items.push({
            type: 'action',
            ...action,
            index: items.length,
          });
        });
    }
    
    return items;
  }, [search, recentSearches, popularSearches, quickActions]);
  
  // Handle selection
  const handleSelect = useCallback((item: any) => {
    switch (item.type) {
      case 'recent':
      case 'popular':
        saveRecentSearch(item.query);
        router.push(`/search?q=${encodeURIComponent(item.query)}`);
        onOpenChange(false);
        setSearch('');
        break;
        
      case 'action':
        item.onSelect?.();
        onOpenChange(false);
        setSearch('');
        break;
    }
  }, [router, saveRecentSearch, onOpenChange]);
  
  // Handle keyboard navigation
  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setSelectedIndex(prev => 
          Math.min(prev + 1, filteredItems.length - 1)
        );
        break;
        
      case 'ArrowUp':
        e.preventDefault();
        setSelectedIndex(prev => Math.max(prev - 1, 0));
        break;
        
      case 'Enter':
        e.preventDefault();
        const selectedItem = filteredItems[selectedIndex];
        if (selectedItem) {
          handleSelect(selectedItem);
        }
        break;
    }
  };
  
  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <div className="fixed inset-0 z-50 bg-background/80 backdrop-blur-sm">
        <div className="fixed inset-x-0 top-20 mx-auto max-w-2xl">
          <div className="overflow-hidden rounded-lg border shadow-lg bg-background">
            {/* Search input */}
            <div className="border-b p-4">
              <div className="flex items-center gap-3">
                <Search className="h-5 w-5 text-muted-foreground" />
                <input
                  type="text"
                  value={search}
                  onChange={(e) => setSearch(e.target.value)}
                  onKeyDown={handleKeyDown}
                  placeholder="Search or type a command..."
                  autoFocus
                  className="flex-1 bg-transparent outline-none placeholder:text-muted-foreground"
                />
                <div className="hidden sm:flex items-center gap-1 text-xs text-muted-foreground border rounded px-2 py-1">
                  <Command className="h-3 w-3" />
                  <span>K</span>
                </div>
              </div>
            </div>
            
            {/* Results */}
            <div className="max-h-96 overflow-y-auto">
              {filteredItems.length === 0 ? (
                <div className="py-8 text-center text-muted-foreground">
                  No results found
                </div>
              ) : (
                <div className="py-2">
                  {filteredItems.map((item, index) => {
                    if (item.type === 'section') {
                      return (
                        <div
                          key={`${item.label}-${index}`}
                          className="px-4 py-2 text-xs font-semibold text-muted-foreground uppercase tracking-wider flex items-center gap-2"
                        >
                          {item.icon}
                          {item.label}
                        </div>
                      );
                    }
                    
                    return (
                      <button
                        key={`${item.type}-${item.query || item.id}-${index}`}
                        className={cn(
                          'w-full px-4 py-3 text-left flex items-center gap-3 hover:bg-accent',
                          selectedIndex === index && 'bg-accent'
                        )}
                        onClick={() => handleSelect(item)}
                        onMouseEnter={() => setSelectedIndex(index)}
                      >
                        {/* Icon */}
                        <div className="flex-shrink-0">
                          {item.type === 'recent' && (
                            <Clock className="h-4 w-4 text-muted-foreground" />
                          )}
                          {item.type === 'popular' && (
                            <TrendingUp className="h-4 w-4 text-muted-foreground" />
                          )}
                          {item.type === 'action' && (
                            item.icon || <Command className="h-4 w-4 text-muted-foreground" />
                          )}
                        </div>
                        
                        {/* Content */}
                        <div className="flex-1 min-w-0">
                          <div className="font-medium truncate">
                            {item.query || item.label}
                          </div>
                          {item.type === 'popular' && (
                            <div className="text-xs text-muted-foreground">
                              {item.count.toLocaleString()} searches
                            </div>
                          )}
                        </div>
                        
                        {/* Keyboard hint */}
                        {selectedIndex === index && (
                          <div className="text-xs text-muted-foreground">
                            Press <kbd className="px-1 py-0.5 border rounded">Enter</kbd>
                          </div>
                        )}
                      </button>
                    );
                  })}
                </div>
              )}
            </div>
            
            {/* Footer */}
            <div className="border-t p-3 text-xs text-muted-foreground flex items-center justify-between">
              <div className="flex items-center gap-4">
                <div className="flex items-center gap-1">
                  <span>Navigate:</span>
                  <kbd className="px-1.5 py-0.5 border rounded">↑</kbd>
                  <kbd className="px-1.5 py-0.5 border rounded">↓</kbd>
                </div>
                <div className="flex items-center gap-1">
                  <span>Select:</span>
                  <kbd className="px-1.5 py-0.5 border rounded">Enter</kbd>
                </div>
              </div>
              <button
                onClick={() => onOpenChange(false)}
                className="hover:text-foreground"
              >
                Close
              </button>
            </div>
          </div>
        </div>
      </div>
    </Dialog>
  );
}

// Hook for using command palette
export function useCommandPalette() {
  const [open, setOpen] = useState(false);
  
  const toggle = () => setOpen(prev => !prev);
  const close = () => setOpen(false);
  const openPalette = () => setOpen(true);
  
  // Register global shortcut
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      if ((e.metaKey || e.ctrlKey) && e.key === 'k') {
        e.preventDefault();
        toggle();
      }
    };
    
    window.addEventListener('keydown', handleKeyDown);
    return () => window.removeEventListener('keydown', handleKeyDown);
  }, []);
  
  return {
    open,
    toggle,
    close,
    openPalette,
  };
}

---

§ 7. AUTOCOMPLETE & SUGGESTIONS

§ 7.1 AUTOCOMPLETE TYPES

| Type | Source | Latency | Implementation | Use Case |
|------|--------|---------|----------------|----------|
| **Query suggestions** | Search history + popular | <20ms | Trie/prefix tree | "Did you mean..." |
| **Instant results** | Search engine | <50ms | Debounced API calls | Live search results |
| **Popular searches** | Analytics DB | <10ms | Cached API endpoint | Empty state, trends |
| **Recent searches** | localStorage | <5ms | Client-side storage | Quick access |
| **Category suggestions** | Taxonomy | <10ms | Precomputed tree | Guided search |
| **Product suggestions** | View history | <20ms | Collaborative filtering | Personalization |

§ 7.2 IMPLEMENTATION
typescript
// hooks/useAutocomplete.ts
'use client';

import { useState, useEffect, useCallback, useRef } from 'react';
import { z } from 'zod';

export interface Suggestion {
  id: string;
  text: string;
  type: 'query' | 'category' | 'product' | 'recent' | 'popular';
  icon?: React.ReactNode;
  metadata?: Record<string, any>;
}

export interface UseAutocompleteOptions {
  source: 'local' | 'api';
  debounceMs?: number;
  minQueryLength?: number;
  maxSuggestions?: number;
  onSelect?: (suggestion: Suggestion) => void;
  apiEndpoint?: string;
}

export function useAutocomplete(options: UseAutocompleteOptions) {
  const {
    source,
    debounceMs = 300,
    minQueryLength = 2,
    maxSuggestions = 10,
    onSelect,
    apiEndpoint = '/api/autocomplete',
  } = options;
  
  const [query, setQuery] = useState('');
  const [suggestions, setSuggestions] = useState<Suggestion[]>([]);
  const [loading, setLoading] = useState(false);
  const [selectedIndex, setSelectedIndex] = useState(-1);
  const [isOpen, setIsOpen] = useState(false);
  
  const timeoutRef = useRef<NodeJS.Timeout>();
  const abortControllerRef = useRef<AbortController>();
  
  // Load recent searches from localStorage
  const getRecentSearches = useCallback((): Suggestion[] => {
    try {
      const saved = localStorage.getItem('recent-searches');
      if (!saved) return [];
      
      const recent = JSON.parse(saved) as string[];
      return recent.slice(0, 5).map((text, index) => ({
        id: `recent-${index}`,
        text,
        type: 'recent',
        metadata: { timestamp: Date.now() },
      }));
    } catch {
      return [];
    }
  }, []);
  
  // Save search to recent
  const saveToRecent = useCallback((text: string) => {
    try {
      const saved = localStorage.getItem('recent-searches') || '[]';
      const recent = JSON.parse(saved) as string[];
      
      const updated = [
        text,
        ...recent.filter(q => q !== text),
      ].slice(0, 10);
      
      localStorage.setItem('recent-searches', JSON.stringify(updated));
    } catch {
      // Ignore errors
    }
  }, []);
  
  // Fetch suggestions from API
  const fetchSuggestions = useCallback(async (searchQuery: string) => {
    if (searchQuery.length < minQueryLength) {
      return [];
    }
    
    if (source === 'local') {
      // Local suggestions logic
      const recent = getRecentSearches();
      const filtered = recent.filter(s => 
        s.text.toLowerCase().includes(searchQuery.toLowerCase())
      );
      return filtered.slice(0, maxSuggestions);
    }
    
    // API call
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }
    
    abortControllerRef.current = new AbortController();
    
    try {
      const response = await fetch(`${apiEndpoint}?q=${encodeURIComponent(searchQuery)}`, {
        signal: abortControllerRef.current.signal,
      });
      
      if (!response.ok) throw new Error('Failed to fetch suggestions');
      
      const data = await response.json();
      
      return data.suggestions.map((s: any, index: number) => ({
        id: s.id || `suggestion-${index}`,
        text: s.text,
        type: s.type || 'query',
        metadata: s.metadata,
      })) as Suggestion[];
      
    } catch (error: any) {
      if (error.name === 'AbortError') {
        // Request was cancelled, ignore
        return [];
      }
      console.error('Failed to fetch suggestions:', error);
      return [];
    }
  }, [source, minQueryLength, maxSuggestions, apiEndpoint, getRecentSearches]);
  
  // Debounced search
  const search = useCallback(async (searchQuery: string) => {
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
    
    timeoutRef.current = setTimeout(async () => {
      if (searchQuery.length < minQueryLength) {
        setSuggestions(getRecentSearches());
        setLoading(false);
        return;
      }
      
      setLoading(true);
      
      const results = await fetchSuggestions(searchQuery);
      
      setSuggestions(results);
      setLoading(false);
      setIsOpen(results.length > 0);
      
      // Reset selection
      setSelectedIndex(-1);
      
    }, debounceMs);
  }, [fetchSuggestions, getRecentSearches, minQueryLength, debounceMs]);
  
  // Handle query change
  const handleQueryChange = useCallback((newQuery: string) => {
    setQuery(newQuery);
    setIsOpen(true);
    
    if (newQuery.trim() === '') {
      setSuggestions(getRecentSearches());
      setLoading(false);
    } else {
      search(newQuery);
    }
  }, [search, getRecentSearches]);
  
  // Handle suggestion selection
  const handleSelect = useCallback((suggestion: Suggestion) => {
    setQuery(suggestion.text);
    setIsOpen(false);
    setSelectedIndex(-1);
    
    saveToRecent(suggestion.text);
    onSelect?.(suggestion);
  }, [onSelect, saveToRecent]);
  
  // Keyboard navigation
  const handleKeyDown = useCallback((e: React.KeyboardEvent) => {
    if (!isOpen) return;
    
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setSelectedIndex(prev => 
          prev < suggestions.length - 1 ? prev + 1 : 0
        );
        break;
        
      case 'ArrowUp':
        e.preventDefault();
        setSelectedIndex(prev => 
          prev > 0 ? prev - 1 : suggestions.length - 1
        );
        break;
        
      case 'Enter':
        e.preventDefault();
        if (selectedIndex >= 0 && selectedIndex < suggestions.length) {
          handleSelect(suggestions[selectedIndex]);
        }
        break;
        
      case 'Escape':
        e.preventDefault();
        setIsOpen(false);
        setSelectedIndex(-1);
        break;
    }
  }, [isOpen, suggestions, selectedIndex, handleSelect]);
  
  // Close when clicking outside
  useEffect(() => {
    const handleClickOutside = () => {
      setIsOpen(false);
      setSelectedIndex(-1);
    };
    
    document.addEventListener('click', handleClickOutside);
    return () => document.removeEventListener('click', handleClickOutside);
  }, []);
  
  // Cleanup
  useEffect(() => {
    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, []);
  
  return {
    query,
    suggestions,
    loading,
    selectedIndex,
    isOpen,
    setQuery: handleQueryChange,
    handleKeyDown,
    handleSelect,
    setIsOpen,
  };
}

// API endpoint for autocomplete
// app/api/autocomplete/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { SearchClient } from '@/lib/search/meilisearch';
import { prisma } from '@/lib/prisma';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const query = searchParams.get('q') || '';
  const limit = parseInt(searchParams.get('limit') || '10');
  const types = searchParams.get('types')?.split(',') || ['query', 'product'];
  
  if (!query || query.length < 2) {
    return NextResponse.json({ suggestions: [] });
  }
  
  const suggestions = [];
  
  // Query suggestions from search engine
  if (types.includes('query')) {
    try {
      const index = SearchClient.getIndex('products');
      const result = await index.search(query, {
        limit: 5,
        attributesToRetrieve: [],
        attributesToHighlight: [],
        showMatchesPosition: false,
      });
      
      // Extract unique query suggestions from hits
      const querySuggestions = result.hits.map(hit => ({
        id: `query-${hit.id}`,
        text: query,
        type: 'query' as const,
        metadata: { hitId: hit.id },
      }));
      
      suggestions.push(...querySuggestions);
    } catch (error) {
      console.error('Failed to get query suggestions:', error);
    }
  }
  
  // Product suggestions
  if (types.includes('product') && suggestions.length < limit) {
    try {
      const products = await prisma.product.findMany({
        where: {
          OR: [
            { name: { contains: query, mode: 'insensitive' } },
            { sku: { contains: query, mode: 'insensitive' } },
          ],
        },
        take: limit - suggestions.length,
        select: {
          id: true,
          name: true,
          sku: true,
          category: true,
        },
      });
      
      const productSuggestions = products.map(product => ({
        id: `product-${product.id}`,
        text: product.name,
        type: 'product' as const,
        metadata: {
          productId: product.id,
          sku: product.sku,
          category: product.category,
        },
      }));
      
      suggestions.push(...productSuggestions);
    } catch (error) {
      console.error('Failed to get product suggestions:', error);
    }
  }
  
  // Category suggestions
  if (types.includes('category') && suggestions.length < limit) {
    try {
      const categories = await prisma.category.findMany({
        where: {
          name: { contains: query, mode: 'insensitive' },
        },
        take: limit - suggestions.length,
        select: {
          id: true,
          name: true,
          slug: true,
        },
      });
      
      const categorySuggestions = categories.map(category => ({
        id: `category-${category.id}`,
        text: category.name,
        type: 'category' as const,
        metadata: {
          categoryId: category.id,
          slug: category.slug,
        },
      }));
      
      suggestions.push(...categorySuggestions);
    } catch (error) {
      console.error('Failed to get category suggestions:', error);
    }
  }
  
  // Popular searches (from analytics)
  if (types.includes('popular') && suggestions.length < limit) {
    try {
      // This would query your analytics database
      const popular = await getPopularSearches(query, limit - suggestions.length);
      
      const popularSuggestions = popular.map((item, index) => ({
        id: `popular-${index}`,
        text: item.query,
        type: 'popular' as const,
        metadata: {
          count: item.count,
          trend: item.trend,
        },
      }));
      
      suggestions.push(...popularSuggestions);
    } catch (error) {
      console.error('Failed to get popular suggestions:', error);
    }
  }
  
  return NextResponse.json({ suggestions: suggestions.slice(0, limit) });
}

async function getPopularSearches(query: string, limit: number) {
  // In a real app, this would query your analytics database
  // This is a simplified version
  return [
    { query: 'laptop', count: 1234, trend: 'up' },
    { query: 'phone', count: 987, trend: 'stable' },
    { query: 'tablet', count: 654, trend: 'down' },
  ].filter(item => item.query.includes(query)).slice(0, limit);
}

§ 7.3 AUTOCOMPLETE UI
typescript
// components/search/Autocomplete.tsx
'use client';

import React from 'react';
import { cn } from '@/lib/utils';
import { Search, Clock, TrendingUp, Package, Tag, Loader2 } from 'lucide-react';
import { Suggestion } from '@/hooks/useAutocomplete';

export interface AutocompleteProps {
  suggestions: Suggestion[];
  query: string;
  loading?: boolean;
  selectedIndex?: number;
  isOpen?: boolean;
  className?: string;
  inputClassName?: string;
  dropdownClassName?: string;
  onSelect: (suggestion: Suggestion) => void;
  onQueryChange: (query: string) => void;
  onKeyDown?: (e: React.KeyboardEvent) => void;
}

export function Autocomplete({
  suggestions,
  query,
  loading = false,
  selectedIndex = -1,
  isOpen = false,
  className,
  inputClassName,
  dropdownClassName,
  onSelect,
  onQueryChange,
  onKeyDown,
}: AutocompleteProps) {
  const handleSuggestionClick = (suggestion: Suggestion) => {
    onSelect(suggestion);
  };
  
  const getIcon = (type: Suggestion['type']) => {
    switch (type) {
      case 'recent':
        return <Clock className="h-4 w-4" />;
      case 'popular':
        return <TrendingUp className="h-4 w-4" />;
      case 'product':
        return <Package className="h-4 w-4" />;
      case 'category':
        return <Tag className="h-4 w-4" />;
      default:
        return <Search className="h-4 w-4" />;
    }
  };
  
  const getTypeLabel = (type: Suggestion['type']) => {
    switch (type) {
      case 'recent':
        return 'Recent search';
      case 'popular':
        return 'Popular search';
      case 'product':
        return 'Product';
      case 'category':
        return 'Category';
      default:
        return 'Search';
    }
  };
  
  return (
    <div className={cn('relative', className)}>
      {/* Input */}
      <div className="relative">
        <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-muted-foreground" />
        <input
          type="text"
          value={query}
          onChange={(e) => onQueryChange(e.target.value)}
          onKeyDown={onKeyDown}
          placeholder="Search products, categories..."
          className={cn(
            'w-full pl-10 pr-10 py-2 border rounded-lg bg-background',
            'focus:outline-none focus:ring-2 focus:ring-ring',
            inputClassName
          )}
        />
        {loading && (
          <Loader2 className="absolute right-3 top-1/2 transform -translate-y-1/2 h-4 w-4 animate-spin text-muted-foreground" />
        )}
      </div>
      
      {/* Dropdown */}
      {isOpen && suggestions.length > 0 && (
        <div className={cn(
          'absolute top-full mt-1 w-full bg-background border rounded-lg shadow-lg z-50',
          'max-h-80 overflow-y-auto',
          dropdownClassName
        )}>
          {suggestions.map((suggestion, index) => (
            <button
              key={suggestion.id}
              className={cn(
                'w-full px-4 py-3 text-left flex items-center gap-3 hover:bg-accent',
                'transition-colors',
                selectedIndex === index && 'bg-accent'
              )}
              onClick={() => handleSuggestionClick(suggestion)}
            >
              {/* Icon */}
              <div className="flex-shrink-0 text-muted-foreground">
                {getIcon(suggestion.type)}
              </div>
              
              {/* Content */}
              <div className="flex-1 min-w-0">
                <div className="font-medium truncate">
                  {suggestion.text}
                </div>
                <div className="text-xs text-muted-foreground">
                  {getTypeLabel(suggestion.type)}
                </div>
              </div>
              
              {/* Metadata */}
              {suggestion.type === 'popular' && suggestion.metadata?.count && (
                <div className="text-xs text-muted-foreground">
                  {suggestion.metadata.count.toLocaleString()}
                </div>
              )}
            </button>
          ))}
        </div>
      )}
      
      {/* Loading state */}
      {isOpen && loading && suggestions.length === 0 && (
        <div className={cn(
          'absolute top-full mt-1 w-full bg-background border rounded-lg shadow-lg z-50 p-4',
          dropdownClassName
        )}>
          <div className="flex items-center justify-center gap-2 text-muted-foreground">
            <Loader2 className="h-4 w-4 animate-spin" />
            <span>Loading suggestions...</span>
          </div>
        </div>
      )}
      
      {/* No results */}
      {isOpen && !loading && suggestions.length === 0 && query.length > 0 && (
        <div className={cn(
          'absolute top-full mt-1 w-full bg-background border rounded-lg shadow-lg z-50 p-4',
          dropdownClassName
        )}>
          <div className="text-center text-muted-foreground">
            No suggestions found
          </div>
        </div>
      )}
    </div>
  );
}

// Standalone autocomplete component with hook integration
export function AutocompleteInput(props: Omit<AutocompleteProps, 
  'suggestions' | 'query' | 'loading' | 'selectedIndex' | 'isOpen' | 
  'onSelect' | 'onQueryChange' | 'onKeyDown'
>) {
  const autocomplete = useAutocomplete({
    source: 'api',
    minQueryLength: 2,
    maxSuggestions: 10,
    onSelect: (suggestion) => {
      // Navigate to search results
      window.location.href = `/search?q=${encodeURIComponent(suggestion.text)}`;
    },
  });
  
  return (
    <Autocomplete
      {...props}
      suggestions={autocomplete.suggestions}
      query={autocomplete.query}
      loading={autocomplete.loading}
      selectedIndex={autocomplete.selectedIndex}
      isOpen={autocomplete.isOpen}
      onSelect={autocomplete.handleSelect}
      onQueryChange={autocomplete.setQuery}
      onKeyDown={autocomplete.handleKeyDown}
    />
  );
}

---

§ 8. SEARCH ANALYTICS

§ 8.1 METRICS TO TRACK

| Metric | Description | Importance | Implementation |
|--------|-------------|------------|----------------|
| **Searches/day** | Volume of searches | High | Count search events |
| **Zero results rate** | % of searches with no results | Critical | Track empty result sets |
| **Click-through rate** | % of searches with at least one click | High | Track result clicks |
| **Average position clicked** | Average rank of clicked results | Medium | Position tracking |
| **Search-to-conversion** | % of searches leading to conversion | High | Conversion tracking |
| **Query length** | Average characters per query | Low | Character count |
| **Search duration** | Time from search to click/exit | Medium | Timing events |
| **Filter usage** | Which filters are used | Medium | Filter event tracking |
| **Popular queries** | Most common search terms | High | Term frequency |
| **Failed queries** | Queries with zero results | Critical | Error logging |

§ 8.2 ANALYTICS IMPLEMENTATION
typescript
// lib/search/analytics.ts
import { env } from '@/env';
import { prisma } from '@/lib/prisma';

export interface SearchEvent {
  type: 'search' | 'click' | 'conversion' | 'filter' | 'zero_results';
  searchId?: string;
  query?: string;
  filters?: Record<string, any>;
  results?: {
    total: number;
    processingTimeMs: number;
    zeroResults: boolean;
  };
  click?: {
    documentId: string;
    position: number;
    query: string;
  };
  conversion?: {
    type: string;
    value: number;
    documentId?: string;
  };
  userId?: string;
  sessionId?: string;
  timestamp: Date;
  userAgent?: string;
  referrer?: string;
  source?: string;
}

export class SearchAnalytics {
  private static readonly BATCH_SIZE = 50;
  private static readonly BATCH_INTERVAL = 10000; // 10 seconds
  private static queue: SearchEvent[] = [];
  private static batchTimeout: NodeJS.Timeout | null = null;
  
  // Initialize batch processing
  static init() {
    if (typeof window !== 'undefined') {
      // Flush on page unload
      window.addEventListener('beforeunload', () => this.flush());
      
      // Flush periodically
      setInterval(() => this.flush(), this.BATCH_INTERVAL);
    }
  }
  
  // Track search event
  static trackSearch(params: {
    searchId: string;
    query: string;
    filters?: Record<string, any>;
    totalHits: number;
    processingTimeMs: number;
    userId?: string;
    sessionId?: string;
    source?: string;
  }) {
    const event: SearchEvent = {
      type: 'search',
      searchId: params.searchId,
      query: params.query,
      filters: params.filters,
      results: {
        total: params.totalHits,
        processingTimeMs: params.processingTimeMs,
        zeroResults: params.totalHits === 0,
      },
      userId: params.userId,
      sessionId: params.sessionId,
      timestamp: new Date(),
      source: params.source,
      userAgent: typeof navigator !== 'undefined' ? navigator.userAgent : undefined,
      referrer: typeof document !== 'undefined' ? document.referrer : undefined,
    };
    
    this.addToQueue(event);
    
    // Also track zero results as separate event
    if (params.totalHits === 0) {
      this.trackZeroResults({
        searchId: params.searchId,
        query: params.query,
        userId: params.userId,
        sessionId: params.sessionId,
      });
    }
  }
  
  // Track click event
  static trackClick(params: {
    searchId: string;
    query: string;
    documentId: string;
    position: number;
    userId?: string;
    sessionId?: string;
  }) {
    const event: SearchEvent = {
      type: 'click',
      searchId: params.searchId,
      query: params.query,
      click: {
        documentId: params.documentId,
        position: params.position,
        query: params.query,
      },
      userId: params.userId,
      sessionId: params.sessionId,
      timestamp: new Date(),
    };
    
    this.addToQueue(event);
  }
  
  // Track conversion
  static trackConversion(params: {
    searchId?: string;
    query?: string;
    type: string;
    value: number;
    documentId?: string;
    userId?: string;
    sessionId?: string;
  }) {
    const event: SearchEvent = {
      type: 'conversion',
      searchId: params.searchId,
      query: params.query,
      conversion: {
        type: params.type,
        value: params.value,
        documentId: params.documentId,
      },
      userId: params.userId,
      sessionId: params.sessionId,
      timestamp: new Date(),
    };
    
    this.addToQueue(event);
  }
  
  // Track zero results
  static trackZeroResults(params: {
    searchId: string;
    query: string;
    userId?: string;
    sessionId?: string;
  }) {
    const event: SearchEvent = {
      type: 'zero_results',
      searchId: params.searchId,
      query: params.query,
      userId: params.userId,
      sessionId: params.sessionId,
      timestamp: new Date(),
    };
    
    this.addToQueue(event);
  }
  
  // Track filter usage
  static trackFilter(params: {
    searchId: string;
    query: string;
    filterName: string;
    filterValue: any;
    userId?: string;
    sessionId?: string;
  }) {
    const event: SearchEvent = {
      type: 'filter',
      searchId: params.searchId,
      query: params.query,
      filters: { [params.filterName]: params.filterValue },
      userId: params.userId,
      sessionId: params.sessionId,
      timestamp: new Date(),
    };
    
    this.addToQueue(event);
  }
  
  // Add event to queue
  private static addToQueue(event: SearchEvent) {
    this.queue.push(event);
    
    // Flush if queue is full
    if (this.queue.length >= this.BATCH_SIZE) {
      this.flush();
    }
    
    // Schedule flush if not already scheduled
    if (!this.batchTimeout) {
      this.batchTimeout = setTimeout(() => this.flush(), this.BATCH_INTERVAL);
    }
  }
  
  // Flush queue to database
  static async flush() {
    if (this.queue.length === 0) {
      if (this.batchTimeout) {
        clearTimeout(this.batchTimeout);
        this.batchTimeout = null;
      }
      return;
    }
    
    const events = [...this.queue];
    this.queue = [];
    
    if (this.batchTimeout) {
      clearTimeout(this.batchTimeout);
      this.batchTimeout = null;
    }
    
    try {
      // Store in database
      await this.storeEvents(events);
      
      // Also send to external analytics if configured
      if (env.ANALYTICS_WEBHOOK_URL) {
        this.sendToWebhook(events).catch(console.error);
      }
      
    } catch (error) {
      console.error('Failed to store search analytics:', error);
      
      // Re-add events to queue for retry (except if too old)
      const now = new Date();
      const retryEvents = events.filter(event => 
        now.getTime() - event.timestamp.getTime() < 3600000 // 1 hour
      );
      
      this.queue.unshift(...retryEvents);
    }
  }
  
  // Store events in database
  private static async storeEvents(events: SearchEvent[]) {
    // Using Prisma to store in PostgreSQL
    const dbEvents = events.map(event => ({
      type: event.type,
      search_id: event.searchId,
      query: event.query,
      filters: event.filters,
      results: event.results,
      click_data: event.click,
      conversion_data: event.conversion,
      user_id: event.userId,
      session_id: event.sessionId,
      timestamp: event.timestamp,
      user_agent: event.userAgent,
      referrer: event.referrer,
      source: event.source,
    }));
    
    await prisma.searchEvent.createMany({
      data: dbEvents,
    });
  }
  
  // Send to external webhook
  private static async sendToWebhook(events: SearchEvent[]) {
    const response = await fetch(env.ANALYTICS_WEBHOOK_URL!, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        events,
        app: env.NEXT_PUBLIC_APP_NAME,
        environment: env.NODE_ENV,
      }),
    });
    
    if (!response.ok) {
      throw new Error(`Webhook failed: ${response.status}`);
    }
  }
  
  // Analytics queries
  static async getSearchStats(options: {
    startDate: Date;
    endDate: Date;
    userId?: string;
  }) {
    const { startDate, endDate, userId } = options;
    
    const where = {
      timestamp: {
        gte: startDate,
        lte: endDate,
      },
      ...(userId && { user_id: userId }),
    };
    
    const [
      totalSearches,
      zeroResultSearches,
      searchesWithClicks,
      uniqueUsers,
      popularQueries,
      averageQueryLength,
    ] = await Promise.all([
      // Total searches
      prisma.searchEvent.count({
        where: { ...where, type: 'search' },
      }),
      
      // Zero result searches
      prisma.searchEvent.count({
        where: { 
          ...where, 
          type: 'search',
          results: {
            path: ['zeroResults'],
            equals: true,
          },
        },
      }),
      
      // Searches with clicks
      prisma.searchEvent.count({
        where: {
          ...where,
          type: 'click',
        },
        distinct: ['search_id'],
      }),
      
      // Unique users
      prisma.searchEvent.groupBy({
        by: ['user_id'],
        where: {
          ...where,
          user_id: { not: null },
        },
        _count: true,
      }),
      
      // Popular queries
      prisma.$queryRaw<Array<{ query: string; count: bigint }>>`
        SELECT query, COUNT(*) as count
        FROM search_events
        WHERE type = 'search' 
          AND timestamp >= ${startDate}
          AND timestamp <= ${endDate}
          AND query IS NOT NULL
          ${userId ? prisma.sql`AND user_id = ${userId}` : prisma.sql``}
        GROUP BY query
        ORDER BY count DESC
        LIMIT 20
      `,
      
      // Average query length
      prisma.$queryRaw<Array<{ avg_length: number }>>`
        SELECT AVG(CHAR_LENGTH(query)) as avg_length
        FROM search_events
        WHERE type = 'search'
          AND timestamp >= ${startDate}
          AND timestamp <= ${endDate}
          AND query IS NOT NULL
      `,
    ]);
    
    const zeroResultRate = totalSearches > 0 
      ? (zeroResultSearches / totalSearches) * 100 
      : 0;
    
    const clickThroughRate = totalSearches > 0
      ? (searchesWithClicks / totalSearches) * 100
      : 0;
    
    return {
      totalSearches,
      zeroResultSearches,
      zeroResultRate: parseFloat(zeroResultRate.toFixed(2)),
      searchesWithClicks,
      clickThroughRate: parseFloat(clickThroughRate.toFixed(2)),
      uniqueUsers: uniqueUsers.length,
      popularQueries: popularQueries.map(row => ({
        query: row.query,
        count: Number(row.count),
      })),
      averageQueryLength: parseFloat(averageQueryLength[0]?.avg_length?.toFixed(1) || '0'),
    };
  }
  
  // Get search performance metrics
  static async getPerformanceMetrics(options: {
    startDate: Date;
    endDate: Date;
  }) {
    const { startDate, endDate } = options;
    
    const metrics = await prisma.$queryRaw<Array<{
      avg_processing_time: number;
      p95_processing_time: number;
      p99_processing_time: number;
      avg_click_position: number;
    }>>`
      WITH search_stats AS (
        SELECT 
          processing_time_ms,
          PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY processing_time_ms) as p95,
          PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY processing_time_ms) as p99
        FROM search_events,
          LATERAL (SELECT (results->>'processingTimeMs')::int as processing_time_ms) as pt
        WHERE type = 'search'
          AND timestamp >= ${startDate}
          AND timestamp <= ${endDate}
          AND results->>'processingTimeMs' IS NOT NULL
        GROUP BY search_id
      ),
      click_stats AS (
        SELECT 
          AVG((click_data->>'position')::int) as avg_position
        FROM search_events
        WHERE type = 'click'
          AND timestamp >= ${startDate}
          AND timestamp <= ${endDate}
          AND click_data->>'position' IS NOT NULL
      )
      SELECT 
        AVG(processing_time_ms) as avg_processing_time,
        AVG(p95) as p95_processing_time,
        AVG(p99) as p99_processing_time,
        (SELECT avg_position FROM click_stats) as avg_click_position
      FROM search_stats
    `;
    
    return metrics[0] || {
      avg_processing_time: 0,
      p95_processing_time: 0,
      p99_processing_time: 0,
      avg_click_position: 0,
    };
  }
}

// Initialize analytics
if (typeof window !== 'undefined') {
  SearchAnalytics.init();
}

§ 8.3 ZERO RESULTS HANDLING
typescript
// components/search/ZeroResults.tsx
'use client';

import React from 'react';
import { Search, AlertCircle, TrendingUp, HelpCircle } from 'lucide-react';
import { useSearchAnalytics } from '@/hooks/useSearchAnalytics';

export interface ZeroResultsProps {
  query: string;
  searchId?: string;
  userId?: string;
  sessionId?: string;
  onSuggestionClick?: (suggestion: string) => void;
  onReportMissing?: (query: string) => void;
}

export function ZeroResults({
  query,
  searchId,
  userId,
  sessionId,
  onSuggestionClick,
  onReportMissing,
}: ZeroResultsProps) {
  const [popularSearches, setPopularSearches] = React.useState<string[]>([]);
  const [similarQueries, setSimilarQueries] = React.useState<string[]>([]);
  const [reported, setReported] = React.useState(false);
  
  const { trackZeroResults } = useSearchAnalytics();
  
  // Load suggestions on mount
  React.useEffect(() => {
    if (!query) return;
    
    // Track zero results
    if (searchId) {
      trackZeroResults({
        searchId,
        query,
        userId,
        sessionId,
      });
    }
    
    // Load popular searches
    loadPopularSearches();
    
    // Load similar queries (spelling suggestions)
    loadSimilarQueries();
  }, [query, searchId, userId, sessionId, trackZeroResults]);
  
  const loadPopularSearches = async () => {
    try {
      const response = await fetch('/api/analytics/popular-searches');
      const data = await response.json();
      setPopularSearches(data.searches.slice(0, 5));
    } catch (error) {
      console.error('Failed to load popular searches:', error);
    }
  };
  
  const loadSimilarQueries = async () => {
    try {
      const response = await fetch(`/api/search/suggestions?q=${encodeURIComponent(query)}`);
      const data = await response.json();
      setSimilarQueries(data.suggestions.slice(0, 3));
    } catch (error) {
      console.error('Failed to load similar queries:', error);
    }
  };
  
  const handleReportMissing = () => {
    if (onReportMissing) {
      onReportMissing(query);
    }
    setReported(true);
    
    // Send report to backend
    fetch('/api/search/report-missing', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ query, userId, sessionId }),
    }).catch(console.error);
  };
  
  return (
    <div className="max-w-2xl mx-auto py-12 px-4">
      <div className="text-center">
        <div className="w-16 h-16 rounded-full bg-muted flex items-center justify-center mx-auto mb-6">
          <Search className="h-8 w-8 text-muted-foreground" />
        </div>
        
        <h2 className="text-2xl font-semibold mb-3">No results found for "{query}"</h2>
        <p className="text-muted-foreground mb-8">
          We couldn't find any matches for your search. Try different keywords or check for typos.
        </p>
        
        {/* Suggestions */}
        <div className="space-y-6">
          {/* Similar queries */}
          {similarQueries.length > 0 && (
            <div>
              <h3 className="text-sm font-medium text-muted-foreground mb-3 flex items-center justify-center gap-2">
                <AlertCircle className="h-4 w-4" />
                Did you mean?
              </h3>
              <div className="flex flex-wrap gap-2 justify-center">
                {similarQueries.map((suggestion) => (
                  <button
                    key={suggestion}
                    onClick={() => onSuggestionClick?.(suggestion)}
                    className="px-4 py-2 bg-primary text-primary-foreground rounded-lg hover:bg-primary/90 transition-colors"
                  >
                    {suggestion}
                  </button>
                ))}
              </div>
            </div>
          )}
          
          {/* Popular searches */}
          {popularSearches.length > 0 && (
            <div>
              <h3 className="text-sm font-medium text-muted-foreground mb-3 flex items-center justify-center gap-2">
                <TrendingUp className="h-4 w-4" />
                Popular searches
              </h3>
              <div className="flex flex-wrap gap-2 justify-center">
                {popularSearches.map((search) => (
                  <button
                    key={search}
                    onClick={() => onSuggestionClick?.(search)}
                    className="px-4 py-2 border rounded-lg hover:bg-accent transition-colors"
                  >
                    {search}
                  </button>
                ))}
              </div>
            </div>
          )}
          
          {/* Report missing content */}
          <div className="pt-6 border-t">
            <h3 className="text-sm font-medium text-muted-foreground mb-3 flex items-center justify-center gap-2">
              <HelpCircle className="h-4 w-4" />
              Can't find what you're looking for?
            </h3>
            <button
              onClick={handleReportMissing}
              disabled={reported}
              className="px-4 py-2 border rounded-lg hover:bg-accent transition-colors disabled:opacity-50"
            >
              {reported ? 'Thanks for reporting!' : 'Report missing content'}
            </button>
          </div>
        </div>
        
        {/* Search tips */}
        <div className="mt-8 text-sm text-muted-foreground">
          <p className="mb-2">Search tips:</p>
          <ul className="space-y-1">
            <li>• Try different keywords or synonyms</li>
            <li>• Check your spelling</li>
            <li>• Use fewer words for broader results</li>
            <li>• Try removing filters</li>
          </ul>
        </div>
      </div>
    </div>
  );
}

// Hook for search analytics
export function useSearchAnalytics() {
  const trackSearch = React.useCallback((params: {
    searchId: string;
    query: string;
    totalHits: number;
    processingTimeMs: number;
    userId?: string;
    sessionId?: string;
    source?: string;
  }) => {
    if (typeof window !== 'undefined') {
      // Send to analytics service
      window.gtag?.('event', 'search', {
        search_term: params.query,
        search_id: params.searchId,
        results_count: params.totalHits,
        processing_time: params.processingTimeMs,
      });
      
      // Also send to custom analytics
      fetch('/api/analytics/search', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(params),
      }).catch(console.error);
    }
  }, []);
  
  const trackClick = React.useCallback((params: {
    searchId: string;
    query: string;
    documentId: string;
    position: number;
    userId?: string;
    sessionId?: string;
  }) => {
    if (typeof window !== 'undefined') {
      window.gtag?.('event', 'search_click', {
        search_term: params.query,
        search_id: params.searchId,
        document_id: params.documentId,
        position: params.position,
      });
    }
  }, []);
  
  const trackZeroResults = React.useCallback((params: {
    searchId: string;
    query: string;
    userId?: string;
    sessionId?: string;
  }) => {
    if (typeof window !== 'undefined') {
      window.gtag?.('event', 'zero_results', {
        search_term: params.query,
        search_id: params.searchId,
      });
    }
  }, []);
  
  return {
    trackSearch,
    trackClick,
    trackZeroResults,
  };
}

---

§ 9. SEARCH OPTIMIZATION

§ 9.1 RELEVANCE TUNING

| Technique | Effect | Implementation | Example |
|-----------|--------|----------------|---------|
| **Field boosting** | Prioritize certain fields | `searchableAttributes` order | Title > Description > Tags |
| **Custom ranking** | Business logic scoring | `rankingRules` | Popularity, rating, recency |
| **Synonyms** | Expand term matching | `synonyms` config | "phone" = ["mobile", "cell"] |
| **Stop words** | Reduce noise | `stopWords` | ["the", "a", "an"] |
| **Typo tolerance** | Handle mistakes | `typoTolerance` | minWordSizeForTypos |
| **Prefix search** | Match beginnings | `searchableAttributes` order | "app" matches "apple" |
| **Exactness bonus** | Boost exact matches | `rankingRules` | "words" before "typo" |
| **Geo boosting** | Location relevance | Custom ranking | Near user location |

typescript
// lib/search/relevance-tuning.ts
export class RelevanceTuner {
  // Field boosting configuration
  static getFieldBoosts(): Record<string, number> {
    return {
      name: 3.0,      // Highest priority
      sku: 2.5,       // SKU exact matches important
      category: 1.5,  // Category matches
      brand: 1.5,     // Brand matches
      description: 1.0, // Standard weight
      tags: 0.8,      // Tags less important
      content: 0.5,   // General content lowest
    };
  }
  
  // Custom ranking rules for Meilisearch
  static getRankingRules(): string[] {
    return [
      // 1. Text relevance
      'words',
      'typo',
      'proximity',
      
      // 2. Field importance
      'attribute',
      
      // 3. Custom business logic
      'desc(popularity)',
      'desc(rating)',
      'desc(createdAt)',
      
      // 4. Exact matches last
      'exactness',
      'sort', // User-defined sort
    ];
  }
  
  // Synonyms for better matching
  static getSynonyms(): Record<string, string[]> {
    return {
      // Tech products
      'phone': ['mobile', 'cellphone', 'smartphone', 'cellular'],
      'tv': ['television', 'smart tv', 'led tv', 'oled tv'],
      'laptop': ['notebook', 'macbook', 'ultrabook', 'chromebook'],
      
      // Categories
      'electronics': ['gadgets', 'tech', 'devices'],
      'clothing': ['apparel', 'fashion', 'wear'],
      
      // Common misspellings
      'iphone': ['ipone', 'iphne', 'iphpone'],
      'samsung': ['samsumg', 'samsing'],
    };
  }
  
  // Stop words configuration
  static getStopWords(): string[] {
    const common = ['the', 'a', 'an', 'and', 'or', 'but', 'in', 'on', 'at', 'to', 'for', 'of', 'with'];
    const languageSpecific = {
      en: common,
      it: ['il', 'lo', 'la', 'i', 'gli', 'le', 'un', 'uno', 'una', 'e', 'o'],
      de: ['der', 'die', 'das', 'ein', 'eine', 'und', 'oder'],
      fr: ['le', 'la', 'les', 'un', 'une', 'des', 'et', 'ou'],
    };
    
    return common;
  }
  
  // Dynamic boosting based on query
  static calculateDynamicBoost(query: string): Record<string, number> {
    const boosts = { ...this.getFieldBoosts() };
    
    // If query looks like a SKU (alphanumeric, uppercase)
    if (/^[A-Z0-9-]{3,}$/.test(query)) {
      boosts.sku = 5.0; // Boost SKU matches significantly
    }
    
    // If query is short (likely product name)
    if (query.length <= 3) {
      boosts.name = 4.0; // Boost name matches for short queries
    }
    
    // If query contains category words
    const categoryWords = ['shirt', 'pants', 'shoe', 'phone', 'tv', 'laptop'];
    if (categoryWords.some(word => query.toLowerCase().includes(word))) {
      boosts.category = 2.0;
    }
    
    return boosts;
  }
  
  // Generate query with boosts
  static buildBoostedQuery(query: string): string {
    const boosts = this.calculateDynamicBoost(query);
    const terms = query.split(/\s+/).filter(term => term.length > 0);
    
    if (terms.length === 0) return '';
    
    // Simple boosting: duplicate important terms
    const boostedTerms = terms.flatMap(term => {
      const boostMultiplier = this.getTermBoost(term);
      return Array(boostMultiplier).fill(term);
    });
    
    return boostedTerms.join(' ');
  }
  
  private static getTermBoost(term: string): number {
    // Boost numeric terms (likely prices, SKUs)
    if (/^\d+$/.test(term)) return 2;
    
    // Boost short terms (likely brands, acronyms)
    if (term.length <= 3) return 2;
    
    // Boost uppercase terms (likely brands)
    if (term === term.toUpperCase()) return 2;
    
    return 1;
  }
}

§ 9.2 PERFORMANCE OPTIMIZATION

| Technique | Benefit | Implementation | Impact |
|-----------|---------|----------------|--------|
| **Index only needed fields** | Smaller index, faster queries | `searchableAttributes` | High |
| **Pagination vs infinite scroll** | Memory management | Limit `hitsPerPage` | Medium |
| **Request debouncing** | Reduce server load | Client-side debounce | High |
| **Result caching** | Faster repeated queries | Redis/Edge cache | Very High |
| **Facet caching** | Faster filtering | Server-side cache | High |
| **CDN for static content** | Faster assets | CDN configuration | Medium |
| **Connection pooling** | Reduced latency | Client configuration | Medium |
| **Batch indexing** | Efficient updates | Bulk operations | High |
| **Lazy loading facets** | Faster initial load | Load on demand | Medium |

§ 9.3 CACHING STRATEGY
typescript
// lib/search/cache.ts
import { Redis } from '@upstash/redis';
import { env } from '@/env';

export class SearchCache {
  private static redis: Redis | null = null;
  private static localCache = new Map<string, { data: any; expiry: number }>();
  private static readonly LOCAL_TTL = 10000; // 10 seconds
  private static readonly REDIS_TTL = 3600; // 1 hour
  
  static init() {
    if (env.UPSTASH_REDIS_REST_URL && env.UPSTASH_REDIS_REST_TOKEN) {
      this.redis = new Redis({
        url: env.UPSTASH_REDIS_REST_URL,
        token: env.UPSTASH_REDIS_REST_TOKEN,
      });
    }
  }
  
  static async get<T>(key: string): Promise<T | null> {
    // Check local cache first
    const local = this.localCache.get(key);
    if (local && local.expiry > Date.now()) {
      return local.data as T;
    }
    
    // Check Redis if available
    if (this.redis) {
      try {
        const data = await this.redis.get<T>(key);
        if (data) {
          // Store in local cache
          this.localCache.set(key, {
            data,
            expiry: Date.now() + this.LOCAL_TTL,
          });
          return data;
        }
      } catch (error) {
        console.error('Redis cache error:', error);
      }
    }
    
    return null;
  }
  
  static async set(key: string, value: any, ttl?: number): Promise<void> {
    const expiry = ttl || this.REDIS_TTL;
    
    // Store in local cache
    this.localCache.set(key, {
      data: value,
      expiry: Date.now() + this.LOCAL_TTL,
    });
    
    // Store in Redis if available
    if (this.redis) {
      try {
        await this.redis.set(key, value, { ex: expiry });
      } catch (error) {
        console.error('Redis set error:', error);
      }
    }
  }
  
  static async delete(key: string): Promise<void> {
    this.localCache.delete(key);
    
    if (this.redis) {
      try {
        await this.redis.del(key);
      } catch (error) {
        console.error('Redis delete error:', error);
      }
    }
  }
  
  static async clear(pattern?: string): Promise<void> {
    // Clear local cache
    if (pattern) {
      for (const key of this.localCache.keys()) {
        if (key.includes(pattern)) {
          this.localCache.delete(key);
        }
      }
    } else {
      this.localCache.clear();
    }
    
    // Clear Redis cache
    if (this.redis && pattern) {
      try {
        const keys = await this.redis.keys(`${pattern}*`);
        if (keys.length > 0) {
          await this.redis.del(...keys);
        }
      } catch (error) {
        console.error('Redis clear error:', error);
      }
    }
  }
  
  // Cache search results
  static async cacheSearch(query: string, filters: any, page: number, results: any): Promise<void> {
    const cacheKey = this.getSearchCacheKey(query, filters, page);
    await this.set(cacheKey, results, 300); // 5 minutes TTL for search results
  }
  
  // Get cached search results
  static async getCachedSearch(query: string, filters: any, page: number): Promise<any> {
    const cacheKey = this.getSearchCacheKey(query, filters, page);
    return this.get(cacheKey);
  }
  
  // Invalidate search cache for a document
  static async invalidateDocument(documentId: string): Promise<void> {
    // This would need to track which search queries are affected by this document
    // For simplicity, we'll clear all search caches (or use a more sophisticated approach)
    await this.clear('search:');
  }
  
  private static getSearchCacheKey(query: string, filters: any, page: number): string {
    const filterStr = JSON.stringify(filters || {});
    return `search:${query}:${filterStr}:${page}`;
  }
}

// Initialize cache
SearchCache.init();

---

§ 10. MULTI-TENANT SEARCH

§ 10.1 ISOLATION STRATEGIES

| Strategy | Isolation | Cost | Complexity | Best For |
|----------|-----------|------|------------|----------|
| **Index per tenant** | Full | High | Medium | Enterprise, strict compliance |
| **Tenant ID filter** | Logical | Low | Low | Most SaaS applications |
| **API key scoping** | Logical | Low | Low | B2B platforms |
| **Separate instance** | Full | Very High | High | Banks, healthcare |
| **Database per tenant** | Full | High | High | Regulated industries |

§ 10.2 IMPLEMENTATION
typescript
// lib/search/multi-tenant.ts
import { SearchClient } from './meilisearch';
import { prisma } from '@/lib/prisma';
import { env } from '@/env';

export class MultiTenantSearch {
  private static tenantCache = new Map<string, string>();
  
  // Get tenant-specific index name
  static getIndexName(baseIndex: string, tenantId: string): string {
    if (env.IS_MULTI_TENANT !== 'true') {
      return baseIndex;
    }
    
    // Sanitize tenant ID for index name
    const sanitized = tenantId.replace(/[^a-z0-9]/gi, '_').toLowerCase();
    return `${baseIndex}_${sanitized}`;
  }
  
  // Get tenant from request
  static async getTenantFromRequest(request: Request): Promise<string> {
    // Try multiple methods to get tenant ID
    
    // 1. From subdomain
    const host = request.headers.get('host') || '';
    const subdomain = host.split('.')[0];
    if (subdomain && subdomain !== 'www' && subdomain !== 'app') {
      const tenant = await this.getTenantBySubdomain(subdomain);
      if (tenant) return tenant;
    }
    
    // 2. From JWT token
    const authHeader = request.headers.get('authorization');
    if (authHeader?.startsWith('Bearer ')) {
      const token = authHeader.slice(7);
      const tenant = await this.getTenantFromToken(token);
      if (tenant) return tenant;
    }
    
    // 3. From custom header
    const tenantHeader = request.headers.get('x-tenant-id');
    if (tenantHeader) {
      const tenant = await this.getTenantById(tenantHeader);
      if (tenant) return tenant;
    }
    
    // 4. Default tenant
    return 'default';
  }
  
  // Search with tenant context
  static async search<T>(params: {
    index: string;
    query: string;
    tenantId: string;
    filters?: any;
    page?: number;
    hitsPerPage?: number;
  }): Promise<any> {
    const { index, query, tenantId, filters, page = 1, hitsPerPage = 20 } = params;
    
    // Get tenant-specific index
    const tenantIndexName = this.getIndexName(index, tenantId);
    const indexInstance = SearchClient.getIndex<T>(tenantIndexName);
    
    // Add tenant filter to ensure isolation
    const tenantFilter = `tenant_id = "${tenantId}"`;
    const additionalFilters = filters ? ` AND ${this.buildFilterString(filters)}` : '';
    
    const searchParams = {
      q: query,
      page,
      hitsPerPage,
      filter: `${tenantFilter}${additionalFilters}`,
    };
    
    return indexInstance.search(query, searchParams);
  }
  
  // Index document with tenant context
  static async indexDocument<T extends { id: string; tenant_id: string }>(
    index: string,
    document: T
  ): Promise<void> {
    const tenantIndexName = this.getIndexName(index, document.tenant_id);
    const indexInstance = SearchClient.getIndex<T>(tenantIndexName);
    
    await indexInstance.addDocuments([document]);
  }
  
  // Initialize tenant index
  static async initializeTenantIndex(tenantId: string) {
    const baseIndexes = ['products', 'articles', 'customers'];
    
    for (const baseIndex of baseIndexes) {
      const tenantIndexName = this.getIndexName(baseIndex, tenantId);
      const client = SearchClient.getInstance();
      
      try {
        // Check if index exists
        await client.index(tenantIndexName).getStats();
      } catch {
        // Create index if it doesn't exist
        await client.createIndex(tenantIndexName, {
          primaryKey: 'id',
        });
        
        // Copy settings from base index
        const baseSettings = await this.getBaseIndexSettings(baseIndex);
        await client.index(tenantIndexName).updateSettings(baseSettings);
      }
    }
  }
  
  // Delete tenant index
  static async deleteTenantIndex(tenantId: string) {
    const baseIndexes = ['products', 'articles', 'customers'];
    const client = SearchClient.getInstance();
    
    for (const baseIndex of baseIndexes) {
      const tenantIndexName = this.getIndexName(baseIndex, tenantId);
      
      try {
        await client.deleteIndex(tenantIndexName);
      } catch (error) {
        // Index might not exist, ignore
      }
    }
  }
  
  // Search across all tenants (admin only)
  static async searchAllTenants<T>(params: {
    index: string;
    query: string;
    tenantIds?: string[];
    filters?: any;
  }): Promise<Array<{ tenantId: string; results: any }>> {
    const { index, query, tenantIds, filters } = params;
    
    // Get all tenants if not specified
    const tenants = tenantIds || await this.getAllTenantIds();
    
    const searchPromises = tenants.map(async (tenantId) => {
      try {
        const results = await this.search<T>({
          index,
          query,
          tenantId,
          filters,
          hitsPerPage: 10, // Limit per tenant
        });
        
        return {
          tenantId,
          results,
        };
      } catch (error) {
        console.error(`Search failed for tenant ${tenantId}:`, error);
        return {
          tenantId,
          results: { hits: [], estimatedTotalHits: 0 },
        };
      }
    });
    
    return Promise.all(searchPromises);
  }
  
  // Helper methods
  private static async getTenantBySubdomain(subdomain: string): Promise<string | null> {
    if (this.tenantCache.has(subdomain)) {
      return this.tenantCache.get(subdomain)!;
    }
    
    const tenant = await prisma.tenant.findUnique({
      where: { subdomain },
      select: { id: true },
    });
    
    if (tenant) {
      this.tenantCache.set(subdomain, tenant.id);
      return tenant.id;
    }
    
    return null;
  }
  
  private static async getTenantFromToken(token: string): Promise<string | null> {
    // Decode JWT to get tenant ID
    // Implementation depends on your auth system
    return null;
  }
  
  private static async getTenantById(tenantId: string): Promise<string | null> {
    const tenant = await prisma.tenant.findUnique({
      where: { id: tenantId },
      select: { id: true },
    });
    
    return tenant?.id || null;
  }
  
  private static async getAllTenantIds(): Promise<string[]> {
    const tenants = await prisma.tenant.findMany({
      select: { id: true },
    });
    
    return tenants.map(t => t.id);
  }
  
  private static async getBaseIndexSettings(indexName: string): Promise<any> {
    const baseIndex = SearchClient.getIndex(indexName);
    return baseIndex.getSettings();
  }
  
  private static buildFilterString(filters: any): string {
    if (typeof filters === 'string') {
      return filters;
    }
    
    if (Array.isArray(filters)) {
      return filters.join(' AND ');
    }
    
    if (typeof filters === 'object') {
      return Object.entries(filters)
        .map(([key, value]) => {
          if (Array.isArray(value)) {
            return `${key} IN [${value.map(v => `"${v}"`).join(', ')}]`;
          }
          return `${key} = "${value}"`;
        })
        .join(' AND ');
    }
    
    return '';
  }
}

---

§ 11. VECTOR/SEMANTIC SEARCH

§ 11.1 WHEN TO USE VECTOR SEARCH

| Use Case | Traditional Search | Vector Search | Hybrid | Recommendation |
|----------|-------------------|---------------|--------|----------------|
| **Exact keyword match** | ✅ Excellent | ❌ Poor | ✅ Good | Traditional |
| **Semantic similarity** | ❌ Poor | ✅ Excellent | ✅ Good | Vector/Hybrid |
| **Typo tolerance** | ⚠️ Configurable | ✅ Excellent | ✅ Excellent | Vector/Hybrid |
| **Multi-language** | ⚠️ Configurable | ✅ Excellent | ✅ Excellent | Vector/Hybrid |
| **Performance** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | Traditional |
| **Relevance** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Vector/Hybrid |
| **Implementation complexity** | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Traditional |

§ 11.2 HYBRID SEARCH IMPLEMENTATION
typescript
// lib/search/hybrid.ts
import { SearchClient } from './meilisearch';
import { env } from '@/env';
import OpenAI from 'openai';

export class HybridSearch {
  private static openai: OpenAI | null = null;
  
  static init() {
    if (env.OPENAI_API_KEY) {
      this.openai = new OpenAI({
        apiKey: env.OPENAI_API_KEY,
      });
    }
  }
  
  // Generate embedding for text
  static async generateEmbedding(text: string): Promise<number[]> {
    if (!this.openai) {
      throw new Error('OpenAI client not initialized');
    }
    
    const response = await this.openai.embeddings.create({
      model: 'text-embedding-ada-002',
      input: text,
    });
    
    return response.data[0].embedding;
  }
  
  // Hybrid search with Meilisearch
  static async hybridSearch(params: {
    index: string;
    query: string;
    vectorWeight?: number;
    textWeight?: number;
    filters?: any;
    page?: number;
    hitsPerPage?: number;
  }) {
    const { index, query, vectorWeight = 0.5, textWeight = 0.5, filters, page = 1, hitsPerPage = 20 } = params;
    
    // Generate vector embedding for query
    const vector = await this.generateEmbedding(query);
    
    // Perform vector search
    const vectorResults = await this.vectorSearch({
      index,
      vector,
      filters,
      page,
      hitsPerPage,
    });
    
    // Perform text search
    const textResults = await this.textSearch({
      index,
      query,
      filters,
      page,
      hitsPerPage,
    });
    
    // Combine results
    const combined = this.combineResults(
      vectorResults.hits,
      textResults.hits,
      vectorWeight,
      textWeight
    );
    
    // Sort by combined score
    combined.sort((a, b) => b._hybridScore - a._hybridScore);
    
    return {
      hits: combined.slice(0, hitsPerPage),
      totalHits: Math.max(vectorResults.estimatedTotalHits, textResults.estimatedTotalHits),
      processingTimeMs: vectorResults.processingTimeMs + textResults.processingTimeMs,
      vectorResults: vectorResults.estimatedTotalHits,
      textResults: textResults.estimatedTotalHits,
    };
  }
  
  // Vector search using Meilisearch hybrid search
  private static async vectorSearch(params: {
    index: string;
    vector: number[];
    filters?: any;
    page?: number;
    hitsPerPage?: number;
  }) {
    const indexInstance = SearchClient.getIndex(params.index);
    
    const searchParams: any = {
      vector: params.vector,
      hybrid: {
        semanticRatio: 0.8, // Weight for vector search
        embedder: 'openai', // Configured in Meilisearch
      },
      page: params.page,
      hitsPerPage: params.hitsPerPage,
    };
    
    if (params.filters) {
      searchParams.filter = this.buildFilterString(params.filters);
    }
    
    return indexInstance.search('', searchParams);
  }
  
  // Traditional text search
  private static async textSearch(params: {
    index: string;
    query: string;
    filters?: any;
    page?: number;
    hitsPerPage?: number;
  }) {
    const indexInstance = SearchClient.getIndex(params.index);
    
    const searchParams: any = {
      q: params.query,
      page: params.page,
      hitsPerPage: params.hitsPerPage,
    };
    
    if (params.filters) {
      searchParams.filter = this.buildFilterString(params.filters);
    }
    
    return indexInstance.search(params.query, searchParams);
  }
  
  // Combine and rank results
  private static combineResults(
    vectorHits: any[],
    textHits: any[],
    vectorWeight: number,
    textWeight: number
  ) {
    const combinedMap = new Map<string, any>();
    
    // Process vector hits
    vectorHits.forEach((hit, index) => {
      const normalizedScore = 1 - (index / Math.max(vectorHits.length, 1));
      combinedMap.set(hit.id, {
        ...hit,
        _vectorScore: normalizedScore,
        _hybridScore: normalizedScore * vectorWeight,
      });
    });
    
    // Process text hits
    textHits.forEach((hit, index) => {
      const normalizedScore = 1 - (index / Math.max(textHits.length, 1));
      const existing = combinedMap.get(hit.id);
      
      if (existing) {
        // Combine scores
        existing._textScore = normalizedScore;
        existing._hybridScore = existing._hybridScore + (normalizedScore * textWeight);
      } else {
        combinedMap.set(hit.id, {
          ...hit,
          _textScore: normalizedScore,
          _hybridScore: normalizedScore * textWeight,
        });
      }
    });
    
    return Array.from(combinedMap.values());
  }
  
  private static buildFilterString(filters: any): string {
    // Same implementation as before
    if (typeof filters === 'string') return filters;
    
    if (Array.isArray(filters)) {
      return filters.join(' AND ');
    }
    
    if (typeof filters === 'object') {
      return Object.entries(filters)
        .map(([key, value]) => {
          if (Array.isArray(value)) {
            return `${key} IN [${value.map(v => `"${v}"`).join(', ')}]`;
          }
          return `${key} = "${value}"`;
        })
        .join(' AND ');
    }
    
    return '';
  }
}

// Initialize hybrid search
HybridSearch.init();

---

§ 12. SEARCH CHECKLIST

ENGINE SELECTION
☑ Requirements analyzed (scale, features, budget)
☑ Engine selected (Meilisearch recommended)
☑ Hosting decided (self vs cloud)
☑ Backup strategy planned
☑ Monitoring configured (uptime, performance)

INDEXING
☑ Schema/settings configured for all indexes
☑ Searchable attributes defined and prioritized
☑ Filterable attributes defined
☑ Sortable attributes defined
☑ Distinct attribute set for deduplication
☑ Ranking rules configured (business logic first)
☑ Synonyms configured (domain-specific terms)
☑ Stop words configured (language-appropriate)
☑ Typo tolerance configured (balanced settings)
☑ Faceting limits set (performance vs features)

DATA SYNC
☑ Initial index population completed
☑ Real-time sync strategy implemented
☑ Delete handling (soft/hard delete strategy)
☑ Reindex strategy for schema changes
☑ Data validation before indexing
☑ Error handling for failed syncs
☑ Monitoring for sync latency
☑ Backfill capability for historical data

API
☑ Search endpoint implemented (REST/GraphQL)
☑ Input validation (Zod schemas)
☑ Authentication/authorization checks
☑ Rate limiting implemented
☑ Error handling with proper codes
☑ Response caching strategy
☑ API documentation (OpenAPI/Swagger)
☑ Versioning strategy for API changes
☑ Health check endpoint
☑ Metrics endpoint (prometheus)

UI COMPONENTS
☑ Search input with debounce
☑ Search results display (list/grid)
☑ Loading states (skeletons/spinners)
☑ Empty states (zero results)
☑ Error states (failed searches)
☑ Faceted filters (checkboxes, ranges)
☑ Active filters display
☑ Clear filters functionality
☑ Pagination (numbered/infinite scroll)
☑ Sort controls (relevance, price, date)
☑ Search highlighting (query terms)
☑ Responsive design (mobile/desktop)
☑ Accessibility (ARIA labels, keyboard nav)
☑ Command palette (Cmd+K)
☑ Recent searches display
☑ Popular searches suggestions

AUTOCOMPLETE
☑ Query suggestions API implemented
☑ Recent searches (localStorage)
☑ Popular searches (analytics based)
☑ Category suggestions
☑ Product suggestions
☑ Keyboard navigation (arrow keys)
☑ Selection handling (Enter/click)
☑ Debouncing for performance
☑ Loading states
☑ Error handling

ANALYTICS
☑ Search event tracking implemented
☑ Click tracking (position, document)
☑ Zero results tracking
☑ Filter usage tracking
☑ Conversion tracking (search-to-purchase)
☑ Dashboard for search metrics
☑ Alerting for anomalies (zero result spikes)
☑ A/B testing capability
☑ Export functionality (CSV/JSON)

OPTIMIZATION
☑ Relevance tuned (field boosts, ranking)
☑ Caching implemented (query results)
☑ CDN configured for static assets
☑ Performance tested (latency, throughput)
☑ Index size optimized (only needed fields)
☑ Connection pooling configured
☑ Compression enabled (gzip/brotli)
☑ Lazy loading for non-critical features
☑ Bundle size optimized (code splitting)

MULTI-TENANT
☑ Isolation strategy chosen and implemented
☑ Tenant filtering automatic in all queries
☑ Permission checks for cross-tenant access
☑ Tenant-specific indexing
☑ Tenant onboarding/offboarding automation
☑ Performance scaling per tenant
☑ Billing integration (usage-based)

VECTOR SEARCH
☑ Embedding generation configured
☑ Vector search tested and validated
☑ Hybrid search implemented (vector + text)
☑ Embedding model selected and fine-tuned
☑ Vector storage optimized (dimensions, indexing)
☑ Semantic search use cases identified
☑ Performance impact assessed

SECURITY
☑ Input sanitization (XSS prevention)
☑ SQL/NoSQL injection protection
☑ Rate limiting per user/IP
☑ API key authentication
☑ Tenant isolation verified
☑ Data encryption at rest/in transit
☑ Audit logging for sensitive operations
☑ Regular security reviews

TESTING
☑ Unit tests for search logic
☑ Integration tests for API endpoints
☑ E2E tests for search workflows
☑ Performance tests (load, stress)
☑ Relevance tests (quality assessment)
☑ Cross-browser testing
☑ Mobile responsiveness testing
☑ Accessibility testing (screen readers)
☑ Internationalization testing (RTL, languages)

DEPLOYMENT
☑ CI/CD pipeline for search updates
☑ Blue-green deployment capability
☑ Rollback strategy
☑ Environment-specific configurations
☑ Monitoring and alerting
☑ Backup and disaster recovery
☑ Documentation (admin, developer, user)
☑ Training for support team
