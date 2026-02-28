# CATALOGO-CONTENT-TAXONOMY

markdown
# Next.js 14 + Prisma Taxonomy System
## Comprehensive Catalog for Categories, Tags, and Faceted Navigation

## § TAXONOMY FUNDAMENTALS

### Hierarchical vs Flat
- **Hierarchical**: Categories (parent-child relationships)
- **Flat**: Tags (non-hierarchical, equal weight)
- **Mixed**: Combination for complex content organization

§ CATEGORIES VS TAGS
| Aspect | Categories | Tags |
|--------|------------|------|
| Structure | Hierarchical | Flat |
| Cardinality | Usually one primary | Multiple |
| Purpose | Content structure | Content attributes |
| URL | Nested paths | Flat lists |
| SEO | High importance | Medium importance |

§ TAXONOMY DESIGN PRINCIPLES
1. **Consistency**: Uniform naming across content types
2. **Specificity**: Avoid overly broad terms
3. **Mutual Exclusivity**: Categories should not overlap
4. **Completeness**: Cover all content aspects
5. **Scalability**: Design for future growth

§ NAMING CONVENTIONS
typescript
// Recommended naming patterns
const conventions = {
  slug: 'lowercase-with-dashes',
  label: 'Title Case Display',
  description: 'Sentence case with period.',
  apiName: 'camelCaseForCode',
  tableName: 'PascalCaseSingular'
};
§ DATA MODEL
Prisma Schema
prisma
// schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Core taxonomy models
model Taxonomy {
  id          String    @id @default(cuid())
  name        String    @unique
  description String?
  type        TaxonomyType @default(CATEGORY)
  isPublic    Boolean   @default(true)
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  
  // Relationships
  categories  Category[]
  tags        Tag[]
  
  @@index([type, isPublic])
}

enum TaxonomyType {
  CATEGORY
  TAG
  COLLECTION
  ATTRIBUTE
}

model Category {
  id          String    @id @default(cuid())
  slug        String    @unique
  name        String
  description String?
  image       String?
  order       Int       @default(0)
  depth       Int       @default(0)
  path        String    // Materialized path
  lft         Int       // Nested set left
  rgt         Int       // Nested set right
  isActive    Boolean   @default(true)
  isFeatured  Boolean   @default(false)
  metadata    Json?
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  
  // Self-referential hierarchy
  parentId    String?
  parent      Category? @relation("CategoryHierarchy", fields: [parentId], references: [id])
  children    Category[] @relation("CategoryHierarchy")
  
  // Taxonomy relationship
  taxonomyId  String?
  taxonomy    Taxonomy? @relation(fields: [taxonomyId], references: [id])
  
  // Content relationships (polymorphic)
  posts       PostCategory[]
  products    ProductCategory[]
  
  // Indexes
  @@index([parentId])
  @@index([taxonomyId])
  @@index([slug, isActive])
  @@index([lft, rgt])
  @@index([path])
  
  // Composite index for tree queries
  @@index([parentId, order])
}

model Tag {
  id          String    @id @default(cuid())
  slug        String    @unique
  name        String
  description String?
  count       Int       @default(0) // Denormalized count
  isActive    Boolean   @default(true)
  metadata    Json?
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  
  // Taxonomy relationship
  taxonomyId  String?
  taxonomy    Taxonomy? @relation(fields: [taxonomyId], references: [id])
  
  // Content relationships (many-to-many)
  posts       PostTag[]
  products    ProductTag[]
  
  @@index([taxonomyId])
  @@index([slug, isActive])
  @@index([count])
}

// Polymorphic tagging junction tables
model PostCategory {
  id          String   @id @default(cuid())
  postId      String
  categoryId  String
  isPrimary   Boolean  @default(false)
  order       Int      @default(0)
  createdAt   DateTime @default(now())
  
  post        Post     @relation(fields: [postId], references: [id])
  category    Category @relation(fields: [categoryId], references: [id])
  
  @@unique([postId, categoryId])
  @@index([categoryId, isPrimary])
}

model PostTag {
  id         String   @id @default(cuid())
  postId     String
  tagId      String
  createdAt  DateTime @default(now())
  
  post       Post     @relation(fields: [postId], references: [id])
  tag        Tag      @relation(fields: [tagId], references: [id])
  
  @@unique([postId, tagId])
  @@index([tagId])
}

// Product taxonomies (example of multiple content types)
model ProductCategory {
  id          String   @id @default(cuid())
  productId   String
  categoryId  String
  isPrimary   Boolean  @default(false)
  createdAt   DateTime @default(now())
  
  product     Product  @relation(fields: [productId], references: [id])
  category    Category @relation(fields: [categoryId], references: [id])
  
  @@unique([productId, categoryId])
}

model ProductTag {
  id        String   @id @default(cuid())
  productId String
  tagId     String
  createdAt DateTime @default(now())
  
  product   Product  @relation(fields: [productId], references: [id])
  tag       Tag      @relation(fields: [tagId], references: [id])
  
  @@unique([productId, tagId])
}

// Content models (simplified)
model Post {
  id          String   @id @default(cuid())
  title       String
  // ... other fields
  
  categories  PostCategory[]
  tags        PostTag[]
}

model Product {
  id          String   @id @default(cuid())
  name        String
  // ... other fields
  
  categories  ProductCategory[]
  tags        ProductTag[]
}
Polymorphic Tagging Extension
prisma
// Advanced polymorphic tagging for multiple entity types
model TaggedEntity {
  id         String   @id @default(cuid())
  tagId      String
  entityType String   // 'post', 'product', 'page', etc.
  entityId   String
  createdAt  DateTime @default(now())
  
  tag        Tag      @relation(fields: [tagId], references: [id])
  
  @@unique([tagId, entityType, entityId])
  @@index([entityType, entityId])
  @@index([tagId, entityType])
}
§ HIERARCHICAL CATEGORIES
Tree Management Utilities
typescript
// lib/taxonomy/tree-utils.ts
import { PrismaClient, Category } from '@prisma/client';

const prisma = new PrismaClient();

export interface TreeCategory extends Category {
  children: TreeCategory[];
  parent?: TreeCategory | null;
}

export class CategoryTree {
  // Build nested tree from flat list
  static buildTree(categories: Category[]): TreeCategory[] {
    const map = new Map<string, TreeCategory>();
    const roots: TreeCategory[] = [];

    // Initialize all categories with children array
    categories.forEach(category => {
      map.set(category.id, { ...category, children: [] });
    });

    // Build tree structure
    categories.forEach(category => {
      const node = map.get(category.id)!;
      
      if (category.parentId) {
        const parent = map.get(category.parentId);
        if (parent) {
          parent.children.push(node);
          node.parent = parent;
        }
      } else {
        roots.push(node);
      }
    });

    // Sort children by order
    roots.forEach(root => this.sortTree(root));
    return roots.sort((a, b) => a.order - b.order);
  }

  private static sortTree(node: TreeCategory): void {
    node.children.sort((a, b) => a.order - b.order);
    node.children.forEach(child => this.sortTree(child));
  }

  // Get all ancestors of a category
  static async getAncestors(categoryId: string): Promise<Category[]> {
    const category = await prisma.category.findUnique({
      where: { id: categoryId },
      include: { parent: true }
    });

    if (!category) return [];

    const ancestors: Category[] = [];
    let current = category.parent;

    while (current) {
      ancestors.unshift(current);
      const parent = await prisma.category.findUnique({
        where: { id: current.id },
        include: { parent: true }
      });
      current = parent?.parent || null;
    }

    return ancestors;
  }

  // Get all descendants of a category (using materialized path)
  static async getDescendants(categoryId: string): Promise<Category[]> {
    const category = await prisma.category.findUnique({
      where: { id: categoryId }
    });

    if (!category) return [];

    // Using materialized path pattern matching
    return await prisma.category.findMany({
      where: {
        path: {
          startsWith: `${category.path}${category.id}.`
        },
        isActive: true
      },
      orderBy: [{ depth: 'asc' }, { order: 'asc' }]
    });
  }

  // Update materialized path for a category and its descendants
  static async updatePath(categoryId: string): Promise<void> {
    const category = await prisma.category.findUnique({
      where: { id: categoryId },
      include: { parent: true }
    });

    if (!category) return;

    const newPath = category.parentId 
      ? `${category.parent?.path || ''}${category.parentId}.`
      : '';

    // Update this category's path and depth
    await prisma.category.update({
      where: { id: categoryId },
      data: {
        path: newPath,
        depth: newPath.split('.').filter(Boolean).length
      }
    });

    // Update all descendants
    const descendants = await prisma.category.findMany({
      where: {
        path: {
          startsWith: `${category.path}${categoryId}.`
        }
      }
    });

    for (const desc of descendants) {
      const oldPrefix = `${category.path}${categoryId}.`;
      const newPrefix = `${newPath}${categoryId}.`;
      const newDescPath = desc.path.replace(oldPrefix, newPrefix);
      
      await prisma.category.update({
        where: { id: desc.id },
        data: {
          path: newDescPath,
          depth: newDescPath.split('.').filter(Boolean).length
        }
      });
    }
  }

  // Nested Set Model Operations
  static async rebuildNestedSet(): Promise<void> {
    const categories = await prisma.category.findMany({
      orderBy: [{ parentId: 'asc' }, { order: 'asc' }]
    });
    
    let counter = 1;
    const tree = this.buildTree(categories);
    
    const traverse = async (node: TreeCategory, parent?: TreeCategory) => {
      const left = counter++;
      
      // Process children
      for (const child of node.children) {
        await traverse(child, node);
      }
      
      const right = counter++;
      
      // Update database
      await prisma.category.update({
        where: { id: node.id },
        data: {
          lft: left,
          rgt: right,
          depth: parent ? (parent.depth + 1) : 0
        }
      });
    };
    
    for (const root of tree) {
      await traverse(root);
    }
  }

  // Get category with full breadcrumb
  static async getCategoryWithBreadcrumb(slug: string): Promise<{
    category: Category;
    breadcrumb: Array<{ id: string; slug: string; name: string }>;
  }> {
    const category = await prisma.category.findUnique({
      where: { slug }
    });

    if (!category) {
      throw new Error('Category not found');
    }

    // Extract ancestor IDs from path
    const ancestorIds = category.path.split('.').filter(Boolean);
    
    const ancestors = await prisma.category.findMany({
      where: {
        id: { in: ancestorIds }
      },
      select: { id: true, slug: true, name: true }
    });

    // Reorder ancestors based on path
    const breadcrumb = ancestorIds
      .map(id => ancestors.find(a => a.id === id))
      .filter(Boolean) as Array<{ id: string; slug: string; name: string }>;

    breadcrumb.push({
      id: category.id,
      slug: category.slug,
      name: category.name
    });

    return { category, breadcrumb };
  }
}
Breadcrumb Generation Component
tsx
// components/breadcrumbs.tsx
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';

interface BreadcrumbItem {
  label: string;
  href: string;
  isCurrent?: boolean;
}

interface BreadcrumbsProps {
  items: BreadcrumbItem[];
  homeLabel?: string;
  separator?: React.ReactNode;
}

export function Breadcrumbs({ 
  items, 
  homeLabel = 'Home', 
  separator = '/' 
}: BreadcrumbsProps) {
  const pathname = usePathname();

  const generateBreadcrumbs = () => {
    const breadcrumbs: BreadcrumbItem[] = [
      { label: homeLabel, href: '/' }
    ];

    // Add category breadcrumbs
    items.forEach((item, index) => {
      breadcrumbs.push({
        ...item,
        isCurrent: index === items.length - 1
      });
    });

    return breadcrumbs;
  };

  const breadcrumbs = generateBreadcrumbs();

  return (
    <nav aria-label="Breadcrumb" className="py-4">
      <ol className="flex items-center space-x-2 text-sm text-gray-600">
        {breadcrumbs.map((item, index) => (
          <li key={item.href} className="flex items-center">
            {index > 0 && (
              <span className="mx-2" aria-hidden="true">
                {separator}
              </span>
            )}
            
            {item.isCurrent ? (
              <span 
                className="font-medium text-gray-900" 
                aria-current="page"
              >
                {item.label}
              </span>
            ) : (
              <Link
                href={item.href}
                className="hover:text-gray-900 transition-colors"
                prefetch={!item.isCurrent}
              >
                {item.label}
              </Link>
            )}
          </li>
        ))}
      </ol>
    </nav>
  );
}

// Category-specific breadcrumb component
export function CategoryBreadcrumbs({ 
  categoryId 
}: { 
  categoryId: string 
}) {
  const [breadcrumbs, setBreadcrumbs] = useState<BreadcrumbItem[]>([]);

  useEffect(() => {
    const fetchBreadcrumbs = async () => {
      const response = await fetch(`/api/categories/${categoryId}/breadcrumb`);
      const data = await response.json();
      
      const items = data.breadcrumb.map((item: any) => ({
        label: item.name,
        href: `/categories/${item.slug}`,
        isCurrent: item.id === categoryId
      }));
      
      setBreadcrumbs(items);
    };

    fetchBreadcrumbs();
  }, [categoryId]);

  return <Breadcrumbs items={breadcrumbs} />;
}
§ TAGGING SYSTEM
Tag Management Service
typescript
// lib/taxonomy/tag-service.ts
import { PrismaClient, Tag, Prisma } from '@prisma/client';
import { slugify } from '@/lib/utils/slugify';

const prisma = new PrismaClient();

export class TagService {
  // Create or get existing tag
  static async findOrCreate(
    names: string[], 
    taxonomyId?: string
  ): Promise<Tag[]> {
    const tags: Tag[] = [];
    
    for (const name of names) {
      const slug = slugify(name);
      
      const existingTag = await prisma.tag.findUnique({
        where: { slug }
      });
      
      if (existingTag) {
        tags.push(existingTag);
      } else {
        const newTag = await prisma.tag.create({
          data: {
            name,
            slug,
            taxonomyId,
            count: 0
          }
        });
        tags.push(newTag);
      }
    }
    
    return tags;
  }

  // Merge multiple tags into one
  static async mergeTags(
    sourceTagIds: string[], 
    targetTagId: string
  ): Promise<Tag> {
    const targetTag = await prisma.tag.findUnique({
      where: { id: targetTagId }
    });

    if (!targetTag) {
      throw new Error('Target tag not found');
    }

    // Start transaction
    return await prisma.$transaction(async (tx) => {
      // Get all content relationships for source tags
      const postRelations = await tx.postTag.findMany({
        where: { tagId: { in: sourceTagIds } }
      });

      const productRelations = await tx.productTag.findMany({
        where: { tagId: { in: sourceTagIds } }
      });

      // Update relationships to use target tag
      for (const relation of postRelations) {
        // Check if relation already exists
        const existing = await tx.postTag.findUnique({
          where: {
            postId_tagId: {
              postId: relation.postId,
              tagId: targetTagId
            }
          }
        });

        if (!existing) {
          await tx.postTag.create({
            data: {
              postId: relation.postId,
              tagId: targetTagId
            }
          });
        }

        // Delete old relation
        await tx.postTag.delete({
          where: { id: relation.id }
        });
      }

      // Repeat for product relations...
      for (const relation of productRelations) {
        const existing = await tx.productTag.findUnique({
          where: {
            productId_tagId: {
              productId: relation.productId,
              tagId: targetTagId
            }
          }
        });

        if (!existing) {
          await tx.productTag.create({
            data: {
              productId: relation.productId,
              tagId: targetTagId
            }
          });
        }

        await tx.productTag.delete({
          where: { id: relation.id }
        });
      }

      // Update target tag count
      const newCount = await tx.postTag.count({
        where: { tagId: targetTagId }
      }) + await tx.productTag.count({
        where: { tagId: targetTagId }
      });

      const updatedTag = await tx.tag.update({
        where: { id: targetTagId },
        data: { count: newCount }
      });

      // Delete source tags
      await tx.tag.deleteMany({
        where: { id: { in: sourceTagIds } }
      });

      return updatedTag;
    });
  }

  // Clean up unused tags
  static async cleanupUnusedTags(olderThan?: Date): Promise<number> {
    const whereCondition: Prisma.TagWhereInput = {
      OR: [
        { posts: { none: {} } },
        { products: { none: {} } }
      ],
      isActive: true
    };

    if (olderThan) {
      whereCondition.createdAt = { lt: olderThan };
    }

    const unusedTags = await prisma.tag.findMany({
      where: whereCondition,
      select: { id: true }
    });

    const tagIds = unusedTags.map(tag => tag.id);
    
    if (tagIds.length > 0) {
      await prisma.tag.deleteMany({
        where: { id: { in: tagIds } }
      });
    }

    return tagIds.length;
  }

  // Get popular tags
  static async getPopularTags(
    limit: number = 10,
    taxonomyId?: string
  ): Promise<Tag[]> {
    const whereCondition: Prisma.TagWhereInput = {
      isActive: true,
      count: { gt: 0 }
    };

    if (taxonomyId) {
      whereCondition.taxonomyId = taxonomyId;
    }

    return await prisma.tag.findMany({
      where: whereCondition,
      orderBy: { count: 'desc' },
      take: limit
    });
  }

  // Tag autocomplete
  static async autocomplete(
    query: string,
    limit: number = 10
  ): Promise<Array<{ id: string; name: string; slug: string }>> {
    return await prisma.tag.findMany({
      where: {
        OR: [
          { name: { contains: query, mode: 'insensitive' } },
          { slug: { contains: query, mode: 'insensitive' } }
        ],
        isActive: true
      },
      select: {
        id: true,
        name: true,
        slug: true
      },
      orderBy: { count: 'desc' },
      take: limit
    });
  }

  // Update tag counts (denormalized)
  static async updateTagCounts(): Promise<void> {
    // Update counts based on actual relationships
    const tags = await prisma.tag.findMany({
      select: { id: true }
    });

    for (const tag of tags) {
      const postCount = await prisma.postTag.count({
        where: { tagId: tag.id }
      });

      const productCount = await prisma.productTag.count({
        where: { tagId: tag.id }
      });

      const totalCount = postCount + productCount;

      await prisma.tag.update({
        where: { id: tag.id },
        data: { count: totalCount }
      });
    }
  }
}
Tag Input Component with Autocomplete
tsx
// components/tag-input.tsx
'use client';

import { useState, useRef, useEffect } from 'react';
import { X, Tag } from 'lucide-react';

interface TagInputProps {
  initialTags?: string[];
  onTagsChange?: (tags: string[]) => void;
  maxTags?: number;
  placeholder?: string;
  taxonomyId?: string;
}

export function TagInput({
  initialTags = [],
  onTagsChange,
  maxTags = 10,
  placeholder = 'Add tags...',
  taxonomyId
}: TagInputProps) {
  const [tags, setTags] = useState<string[]>(initialTags);
  const [input, setInput] = useState('');
  const [suggestions, setSuggestions] = useState<any[]>([]);
  const [showSuggestions, setShowSuggestions] = useState(false);
  const inputRef = useRef<HTMLInputElement>(null);

  // Fetch suggestions
  useEffect(() => {
    const fetchSuggestions = async () => {
      if (input.length < 2) {
        setSuggestions([]);
        return;
      }

      try {
        const params = new URLSearchParams({
          q: input,
          limit: '5',
          ...(taxonomyId && { taxonomyId })
        });

        const response = await fetch(`/api/tags/autocomplete?${params}`);
        const data = await response.json();
        setSuggestions(data);
      } catch (error) {
        console.error('Error fetching suggestions:', error);
      }
    };

    const timeoutId = setTimeout(fetchSuggestions, 300);
    return () => clearTimeout(timeoutId);
  }, [input, taxonomyId]);

  const addTag = (tag: string) => {
    if (!tag.trim() || tags.includes(tag) || tags.length >= maxTags) return;
    
    const newTags = [...tags, tag.trim()];
    setTags(newTags);
    setInput('');
    setSuggestions([]);
    onTagsChange?.(newTags);
  };

  const removeTag = (tagToRemove: string) => {
    const newTags = tags.filter(tag => tag !== tagToRemove);
    setTags(newTags);
    onTagsChange?.(newTags);
  };

  const handleInputKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter' && input.trim()) {
      e.preventDefault();
      addTag(input);
    } else if (e.key === 'Backspace' && !input && tags.length > 0) {
      removeTag(tags[tags.length - 1]);
    } else if (e.key === 'Escape') {
      setShowSuggestions(false);
    }
  };

  return (
    <div className="space-y-2">
      <div className="flex flex-wrap gap-2 p-2 border rounded-md min-h-[42px]">
        {tags.map((tag) => (
          <span
            key={tag}
            className="inline-flex items-center gap-1 px-2 py-1 text-sm bg-blue-100 text-blue-800 rounded-md"
          >
            <Tag className="w-3 h-3" />
            {tag}
            <button
              type="button"
              onClick={() => removeTag(tag)}
              className="hover:text-blue-900"
            >
              <X className="w-3 h-3" />
            </button>
          </span>
        ))}
        
        {tags.length < maxTags && (
          <div className="relative flex-1 min-w-[120px]">
            <input
              ref={inputRef}
              type="text"
              value={input}
              onChange={(e) => setInput(e.target.value)}
              onKeyDown={handleInputKeyDown}
              onFocus={() => setShowSuggestions(true)}
              onBlur={() => setTimeout(() => setShowSuggestions(false), 200)}
              placeholder={placeholder}
              className="w-full px-2 py-1 text-sm bg-transparent outline-none"
            />
            
            {showSuggestions && suggestions.length > 0 && (
              <div className="absolute top-full left-0 right-0 z-10 mt-1 bg-white border rounded-md shadow-lg">
                {suggestions.map((suggestion) => (
                  <button
                    key={suggestion.id}
                    type="button"
                    onMouseDown={() => addTag(suggestion.name)}
                    className="w-full px-3 py-2 text-left hover:bg-gray-100"
                  >
                    <div className="font-medium">{suggestion.name}</div>
                    {suggestion.count && (
                      <div className="text-xs text-gray-500">
                        Used {suggestion.count} times
                      </div>
                    )}
                  </button>
                ))}
              </div>
            )}
          </div>
        )}
      </div>
      
      <div className="text-xs text-gray-500">
        {tags.length} of {maxTags} tags. Press Enter to add.
      </div>
    </div>
  );
}
§ FACETED NAVIGATION
Faceted Search System
typescript
// lib/taxonomy/faceted-search.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export interface FilterOption {
  id: string;
  label: string;
  slug: string;
  count: number;
}

export interface Facet {
  id: string;
  name: string;
  type: 'category' | 'tag' | 'attribute';
  options: FilterOption[];
}

export interface FilterState {
  categories?: string[];
  tags?: string[];
  attributes?: Record<string, string[]>;
}

export class FacetedSearch {
  // Get available facets with counts
  static async getFacets(
    baseWhere: any,
    currentFilters: FilterState = {}
  ): Promise<Facet[]> {
    const facets: Facet[] = [];

    // Category facet
    const categoryFacet = await this.getCategoryFacet(baseWhere, currentFilters);
    if (categoryFacet.options.length > 0) {
      facets.push(categoryFacet);
    }

    // Tag facet
    const tagFacet = await this.getTagFacet(baseWhere, currentFilters);
    if (tagFacet.options.length > 0) {
      facets.push(tagFacet);
    }

    return facets;
  }

  private static async getCategoryFacet(
    baseWhere: any,
    currentFilters: FilterState
  ): Promise<Facet> {
    // Exclude already selected categories from filter
    const excludeCategories = currentFilters.categories || [];

    const categories = await prisma.category.findMany({
      where: {
        ...baseWhere,
        id: { notIn: excludeCategories },
        isActive: true
      },
      include: {
        _count: {
          select: {
            posts: true,
            products: true
          }
        }
      },
      orderBy: { name: 'asc' }
    });

    const options = categories
      .filter(cat => {
        const totalCount = cat._count.posts + cat._count.products;
        return totalCount > 0;
      })
      .map(cat => ({
        id: cat.id,
        label: cat.name,
        slug: cat.slug,
        count: cat._count.posts + cat._count.products
      }));

    return {
      id: 'categories',
      name: 'Categories',
      type: 'category',
      options
    };
  }

  private static async getTagFacet(
    baseWhere: any,
    currentFilters: FilterState
  ): Promise<Facet> {
    const excludeTags = currentFilters.tags || [];

    // Get tags with counts for the current filtered content
    const tags = await prisma.tag.findMany({
      where: {
        id: { notIn: excludeTags },
        isActive: true,
        OR: [
          {
            posts: {
              some: baseWhere
            }
          },
          {
            products: {
              some: baseWhere
            }
          }
        ]
      },
      select: {
        id: true,
        name: true,
        slug: true,
        _count: {
          select: {
            posts: {
              where: baseWhere
            },
            products: {
              where: baseWhere
            }
          }
        }
      },
      orderBy: { name: 'asc' }
    });

    const options = tags.map(tag => ({
      id: tag.id,
      label: tag.name,
      slug: tag.slug,
      count: tag._count.posts + tag._count.products
    }));

    return {
      id: 'tags',
      name: 'Tags',
      type: 'tag',
      options
    };
  }

  // Apply filters to Prisma query
  static applyFilters(
    query: any,
    filters: FilterState
  ): any {
    const where: any = { ...query.where };

    // Apply category filters
    if (filters.categories && filters.categories.length > 0) {
      where.categories = {
        some: {
          categoryId: { in: filters.categories }
        }
      };
    }

    // Apply tag filters
    if (filters.tags && filters.tags.length > 0) {
      where.tags = {
        some: {
          tagId: { in: filters.tags }
        }
      };
    }

    // Apply attribute filters (dynamic)
    if (filters.attributes) {
      Object.entries(filters.attributes).forEach(([key, values]) => {
        if (values.length > 0) {
          where.metadata = where.metadata || {};
          where.metadata.path = `$.${key}`;
          where.metadata.in = values;
        }
      });
    }

    return { ...query, where };
  }

  // Build URL from filter state
  static buildFilterUrl(
    basePath: string,
    filters: FilterState,
    newFilter?: {
      type: keyof FilterState;
      value: string;
      action: 'add' | 'remove' | 'toggle';
    }
  ): string {
    const params = new URLSearchParams();
    const updatedFilters = { ...filters };

    // Apply filter update if provided
    if (newFilter) {
      const current = (updatedFilters[newFilter.type] as string[]) || [];
      
      if (newFilter.action === 'add' && !current.includes(newFilter.value)) {
        updatedFilters[newFilter.type] = [...current, newFilter.value];
      } else if (newFilter.action === 'remove') {
        updatedFilters[newFilter.type] = current.filter(v => v !== newFilter.value);
      } else if (newFilter.action === 'toggle') {
        if (current.includes(newFilter.value)) {
          updatedFilters[newFilter.type] = current.filter(v => v !== newFilter.value);
        } else {
          updatedFilters[newFilter.type] = [...current, newFilter.value];
        }
      }
    }

    // Add filters to URL params
    Object.entries(updatedFilters).forEach(([key, value]) => {
      if (Array.isArray(value) && value.length > 0) {
        params.set(key, value.join(','));
      } else if (typeof value === 'object' && value !== null) {
        Object.entries(value).forEach(([subKey, subValue]) => {
          if (Array.isArray(subValue) && subValue.length > 0) {
            params.set(`${key}[${subKey}]`, subValue.join(','));
          }
        });
      }
    });

    const queryString = params.toString();
    return queryString ? `${basePath}?${queryString}` : basePath;
  }

  // Parse URL query to filter state
  static parseFilterParams(searchParams: URLSearchParams): FilterState {
    const filters: FilterState = {};

    // Parse categories
    const categoriesParam = searchParams.get('categories');
    if (categoriesParam) {
      filters.categories = categoriesParam.split(',');
    }

    // Parse tags
    const tagsParam = searchParams.get('tags');
    if (tagsParam) {
      filters.tags = tagsParam.split(',');
    }

    // Parse dynamic attributes
    searchParams.forEach((value, key) => {
      const match = key.match(/^attributes\[(.+)\]$/);
      if (match) {
        const attrName = match[1];
        filters.attributes = filters.attributes || {};
        filters.attributes[attrName] = value.split(',');
      }
    });

    return filters;
  }
}
Faceted Navigation Component
tsx
// components/faceted-navigation.tsx
'use client';

import { useState, useEffect } from 'react';
import { useRouter, useSearchParams } from 'next/navigation';
import { Filter, X } from 'lucide-react';
import { Facet, FacetedSearch } from '@/lib/taxonomy/faceted-search';

interface FacetedNavigationProps {
  contentType: 'posts' | 'products';
  basePath: string;
}

export function FacetedNavigation({
  contentType,
  basePath
}: FacetedNavigationProps) {
  const router = useRouter();
  const searchParams = useSearchParams();
  const [facets, setFacets] = useState<Facet[]>([]);
  const [loading, setLoading] = useState(true);
  const [selectedFilters, setSelectedFilters] = useState<Record<string, string[]>>({});

  // Parse current filters from URL
  useEffect(() => {
    const filters: Record<string, string[]> = {};
    
    searchParams.forEach((value, key) => {
      if (key === 'categories' || key === 'tags') {
        filters[key] = value.split(',');
      }
    });
    
    setSelectedFilters(filters);
  }, [searchParams]);

  // Fetch facets
  useEffect(() => {
    const fetchFacets = async () => {
      setLoading(true);
      
      // Build base where condition based on content type
      const baseWhere = {
        published: true,
        ...(contentType === 'posts' && { status: 'PUBLISHED' })
      };

      const filters = FacetedSearch.parseFilterParams(searchParams);
      const facets = await FacetedSearch.getFacets(baseWhere, filters);
      
      setFacets(facets);
      setLoading(false);
    };

    fetchFacets();
  }, [contentType, searchParams]);

  const handleFilterToggle = (facetId: string, optionId: string) => {
    const current = selectedFilters[facetId] || [];
    const newFilters = { ...selectedFilters };
    
    if (current.includes(optionId)) {
      newFilters[facetId] = current.filter(id => id !== optionId);
    } else {
      newFilters[facetId] = [...current, optionId];
    }
    
    // Clean up empty arrays
    Object.keys(newFilters).forEach(key => {
      if (newFilters[key].length === 0) {
        delete newFilters[key];
      }
    });
    
    // Build new URL
    const params = new URLSearchParams();
    Object.entries(newFilters).forEach(([key, values]) => {
      if (values.length > 0) {
        params.set(key, values.join(','));
      }
    });
    
    const queryString = params.toString();
    const newUrl = queryString ? `${basePath}?${queryString}` : basePath;
    
    router.push(newUrl);
  };

  const clearAllFilters = () => {
   

════════════════════════════════════════════════════════════
FIGMA CATALOG: BATCH2-18-CONTENT-TAXONOMY
Prompt ID: 18 / 19
Parte: 2
Exported: 2026-02-06T17:36:05.771Z
Characters: 845
════════════════════════════════════════════════════════════

URL */}
      <link
        rel="canonical"
        href={`https://example.com/categories/${category.slug}${
          page > 1 ? `?page=${page}` : ''
        }`}
      />

      {/* Pagination links */}
      {page > 1 && (
        <link
          rel="prev"
          href={`https://example.com/categories/${category.slug}?page=${page - 1}`}
        />
      )}
      {page < Math.ceil(total / limit) && (
        <link
          rel="next"
          href={`https://example.com/categories/${category.slug}?page=${page + 1}`}
        />
      )}

      {/* Main content */}
      <div className="container mx-auto px-4 py-8">
        <nav aria-label="Breadcrumb" className="mb-6">
          {/* Breadcrumb component here */}
        </nav>

        <header className="mb-8">
          <h1 className="text-4xl font-bold mb-4">{category.name}</h1

## § ADVANCED PATTERNS: CONTENT TAXONOMY

### Server Actions con Validazione

```typescript
// app/actions/content-taxonomy.ts
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


### CONTENT TAXONOMY - Utility Helper #844

```typescript
// lib/utils/content-taxonomy-helper-844.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CONTENT TAXONOMY - Utility Helper #232

```typescript
// lib/utils/content-taxonomy-helper-232.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CONTENT TAXONOMY - Utility Helper #961

```typescript
// lib/utils/content-taxonomy-helper-961.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CONTENT TAXONOMY - Utility Helper #322

```typescript
// lib/utils/content-taxonomy-helper-322.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CONTENT TAXONOMY - Utility Helper #500

```typescript
// lib/utils/content-taxonomy-helper-500.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CONTENT TAXONOMY - Utility Helper #995

```typescript
// lib/utils/content-taxonomy-helper-995.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CONTENT TAXONOMY - Utility Helper #631

```typescript
// lib/utils/content-taxonomy-helper-631.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CONTENT TAXONOMY - Utility Helper #984

```typescript
// lib/utils/content-taxonomy-helper-984.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CONTENT TAXONOMY - Utility Helper #797

```typescript
// lib/utils/content-taxonomy-helper-797.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CONTENT TAXONOMY - Utility Helper #43

```typescript
// lib/utils/content-taxonomy-helper-43.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CONTENT TAXONOMY - Utility Helper #652

```typescript
// lib/utils/content-taxonomy-helper-652.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CONTENT TAXONOMY - Utility Helper #231

```typescript
// lib/utils/content-taxonomy-helper-231.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CONTENT TAXONOMY - Utility Helper #363

```typescript
// lib/utils/content-taxonomy-helper-363.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CONTENT TAXONOMY - Utility Helper #123

```typescript
// lib/utils/content-taxonomy-helper-123.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CONTENT TAXONOMY - Utility Helper #103

```typescript
// lib/utils/content-taxonomy-helper-103.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CONTENT TAXONOMY - Utility Helper #960

```typescript
// lib/utils/content-taxonomy-helper-960.ts
import { z } from "zod";

interface CONTENTTAXONOMYConfig {
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

export class CONTENTTAXONOMYProcessor<TInput, TOutput> {
  private config: CONTENTTAXONOMYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CONTENTTAXONOMYConfig> = {}) {
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

  getConfig(): Readonly<CONTENTTAXONOMYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```
