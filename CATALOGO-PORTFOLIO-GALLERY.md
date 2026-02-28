# CATALOGO-PORTFOLIO-GALLERY

Portfolio Platform per Next.js 14 - Catalogo Completo
§ PORTFOLIO DATA MODEL
Prisma Schema
prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Project {
  id              String     @id @default(cuid())
  slug            String     @unique
  title           String
  description     String
  excerpt         String?
  content         Json?      // Rich text content for case studies
  year            Int
  client          String?
  clientUrl       String?
  location        String?
  duration        String?
  featured        Boolean    @default(false)
  published       Boolean    @default(true)
  order           Int        @default(0)
  
  // Relations
  categories      Category[] @relation("ProjectCategories")
  skills          Skill[]    @relation("ProjectSkills")
  media           Media[]
  testimonials    Testimonial[]
  
  // SEO
  metaTitle       String?
  metaDescription String?
  
  // Timestamps
  createdAt       DateTime   @default(now())
  updatedAt       DateTime   @updatedAt
  
  @@index([featured])
  @@index([year])
  @@index([published])
}

model Category {
  id          String    @id @default(cuid())
  slug        String    @unique
  name        String
  description String?
  color       String?   @default("#3B82F6")
  icon        String?
  projects    Project[] @relation("ProjectCategories")
  
  @@index([slug])
}

model Skill {
  id          String    @id @default(cuid())
  slug        String    @unique
  name        String
  type        SkillType @default(TECHNICAL) // TECHNICAL, DESIGN, SOFT
  proficiency Int       @default(50) // 0-100
  icon        String?
  projects    Project[] @relation("ProjectSkills")
  
  @@index([slug])
  @@index([type])
}

enum SkillType {
  TECHNICAL
  DESIGN
  SOFT
}

model Media {
  id          String   @id @default(cuid())
  type        MediaType @default(IMAGE)
  url         String
  alt         String?
  caption     String?
  width       Int?
  height      Int?
  order       Int      @default(0)
  projectId   String
  project     Project  @relation(fields: [projectId], references: [id], onDelete: Cascade)
  
  // For videos
  thumbnail   String?
  duration    Int?     // in seconds
  
  @@index([projectId])
  @@index([order])
}

enum MediaType {
  IMAGE
  VIDEO
  EMBED
}

model Testimonial {
  id        String   @id @default(cuid())
  name      String
  role      String
  company   String?
  avatar    String?
  content   String
  rating    Int?     @default(5) // 1-5
  projectId String?
  project   Project? @relation(fields: [projectId], references: [id], onDelete: Cascade)
  featured  Boolean  @default(false)
  
  @@index([projectId])
  @@index([featured])
}
TypeScript Types
typescript
// types/portfolio.ts
export type Project = {
  id: string;
  slug: string;
  title: string;
  description: string;
  excerpt?: string;
  content?: any;
  year: number;
  client?: string;
  clientUrl?: string;
  location?: string;
  duration?: string;
  featured: boolean;
  published: boolean;
  order: number;
  categories: Category[];
  skills: Skill[];
  media: Media[];
  testimonials: Testimonial[];
  metaTitle?: string;
  metaDescription?: string;
  createdAt: Date;
  updatedAt: Date;
};

export type Category = {
  id: string;
  slug: string;
  name: string;
  description?: string;
  color?: string;
  icon?: string;
  projectCount?: number;
};

export type Skill = {
  id: string;
  slug: string;
  name: string;
  type: 'TECHNICAL' | 'DESIGN' | 'SOFT';
  proficiency: number;
  icon?: string;
  projectCount?: number;
};

export type Media = {
  id: string;
  type: 'IMAGE' | 'VIDEO' | 'EMBED';
  url: string;
  alt?: string;
  caption?: string;
  width?: number;
  height?: number;
  order: number;
  thumbnail?: string;
  duration?: number;
};

export type Testimonial = {
  id: string;
  name: string;
  role: string;
  company?: string;
  avatar?: string;
  content: string;
  rating?: number;
  projectId?: string;
  featured: boolean;
};

export type FilterState = {
  categories: string[];
  skills: string[];
  year: number | null;
  featured: boolean | null;
};
Project Repository
typescript
// lib/repositories/project.repository.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export class ProjectRepository {
  static async getProjects(filters?: {
    category?: string;
    skill?: string;
    year?: number;
    featured?: boolean;
    published?: boolean;
    take?: number;
    skip?: number;
  }) {
    const where = {
      published: filters?.published ?? true,
      ...(filters?.category && {
        categories: { some: { slug: filters.category } }
      }),
      ...(filters?.skill && {
        skills: { some: { slug: filters.skill } }
      }),
      ...(filters?.year && { year: filters.year }),
      ...(filters?.featured !== undefined && { featured: filters.featured })
    };

    const [projects, total] = await Promise.all([
      prisma.project.findMany({
        where,
        include: {
          categories: true,
          skills: true,
          media: {
            where: { order: 0 },
            take: 1
          },
          _count: {
            select: { media: true }
          }
        },
        orderBy: [
          { featured: 'desc' },
          { order: 'asc' },
          { year: 'desc' }
        ],
        take: filters?.take,
        skip: filters?.skip
      }),
      prisma.project.count({ where })
    ]);

    return { projects, total };
  }

  static async getProjectBySlug(slug: string) {
    return prisma.project.findUnique({
      where: { slug },
      include: {
        categories: true,
        skills: true,
        media: { orderBy: { order: 'asc' } },
        testimonials: true
      }
    });
  }

  static async getFeaturedProjects(take: number = 3) {
    return prisma.project.findMany({
      where: { featured: true, published: true },
      include: {
        categories: true,
        media: {
          where: { order: 0 },
          take: 1
        }
      },
      orderBy: { order: 'asc' },
      take
    });
  }

  static async getProjectFilters() {
    const [categories, skills, years] = await Promise.all([
      prisma.category.findMany({
        include: {
          _count: {
            select: { projects: true }
          }
        },
        orderBy: { name: 'asc' }
      }),
      prisma.skill.findMany({
        where: { projects: { some: {} } },
        include: {
          _count: {
            select: { projects: true }
          }
        },
        orderBy: { name: 'asc' }
      }),
      prisma.project.findMany({
        where: { published: true },
        select: { year: true },
        distinct: ['year'],
        orderBy: { year: 'desc' }
      })
    ]);

    return {
      categories: categories.map(c => ({
        ...c,
        projectCount: c._count.projects
      })),
      skills: skills.map(s => ({
        ...s,
        projectCount: s._count.projects
      })),
      years: years.map(y => y.year)
    };
  }
}
§ PROJECT SHOWCASE
Project Grid Component
typescript
// components/projects/ProjectGrid.tsx
'use client';

import { useState, useEffect } from 'react';
import { Project } from '@/types/portfolio';
import ProjectCard from './ProjectCard';
import { useRouter, useSearchParams } from 'next/navigation';

type LayoutType = 'masonry' | 'grid' | 'list';

interface ProjectGridProps {
  projects: Project[];
  initialLayout?: LayoutType;
  showFilters?: boolean;
}

export default function ProjectGrid({ 
  projects, 
  initialLayout = 'grid',
  showFilters = true 
}: ProjectGridProps) {
  const [layout, setLayout] = useState<LayoutType>(initialLayout);
  const [filteredProjects, setFilteredProjects] = useState(projects);
  const searchParams = useSearchParams();
  const router = useRouter();

  useEffect(() => {
    // Sync with URL params
    const category = searchParams.get('category');
    const skill = searchParams.get('skill');
    const year = searchParams.get('year');
    
    let filtered = [...projects];
    
    if (category) {
      filtered = filtered.filter(p => 
        p.categories.some(c => c.slug === category)
      );
    }
    
    if (skill) {
      filtered = filtered.filter(p => 
        p.skills.some(s => s.slug === skill)
      );
    }
    
    if (year) {
      filtered = filtered.filter(p => p.year === parseInt(year));
    }
    
    setFilteredProjects(filtered);
  }, [searchParams, projects]);

  const gridClasses = {
    masonry: 'columns-1 md:columns-2 lg:columns-3 gap-6 space-y-6',
    grid: 'grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6',
    list: 'grid grid-cols-1 gap-8'
  };

  return (
    <div className="space-y-8">
      {showFilters && (
        <div className="flex flex-wrap items-center justify-between gap-4">
          <div className="flex items-center gap-2">
            <button
              onClick={() => setLayout('grid')}
              className={`p-2 rounded-lg ${layout === 'grid' ? 'bg-blue-100 text-blue-600' : 'bg-gray-100 hover:bg-gray-200'}`}
              aria-label="Grid layout"
            >
              <GridIcon />
            </button>
            <button
              onClick={() => setLayout('masonry')}
              className={`p-2 rounded-lg ${layout === 'masonry' ? 'bg-blue-100 text-blue-600' : 'bg-gray-100 hover:bg-gray-200'}`}
              aria-label="Masonry layout"
            >
              <MasonryIcon />
            </button>
            <button
              onClick={() => setLayout('list')}
              className={`p-2 rounded-lg ${layout === 'list' ? 'bg-blue-100 text-blue-600' : 'bg-gray-100 hover:bg-gray-200'}`}
              aria-label="List layout"
            >
              <ListIcon />
            </button>
          </div>
          
          <div className="text-sm text-gray-500">
            {filteredProjects.length} {filteredProjects.length === 1 ? 'project' : 'projects'} found
          </div>
        </div>
      )}
      
      <div className={gridClasses[layout]}>
        {filteredProjects.map((project) => (
          <ProjectCard
            key={project.id}
            project={project}
            layout={layout}
          />
        ))}
      </div>
      
      {filteredProjects.length === 0 && (
        <div className="text-center py-12">
          <h3 className="text-lg font-semibold text-gray-600 mb-2">
            No projects found
          </h3>
          <button
            onClick={() => router.push('/projects')}
            className="text-blue-600 hover:text-blue-700 font-medium"
          >
            Clear all filters
          </button>
        </div>
      )}
    </div>
  );
}

function GridIcon() {
  return (
    <svg className="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
      <path d="M5 3a2 2 0 00-2 2v2a2 2 0 002 2h2a2 2 0 002-2V5a2 2 0 00-2-2H5zM5 11a2 2 0 00-2 2v2a2 2 0 002 2h2a2 2 0 002-2v-2a2 2 0 00-2-2H5zM11 5a2 2 0 012-2h2a2 2 0 012 2v2a2 2 0 01-2 2h-2a2 2 0 01-2-2V5zM11 13a2 2 0 012-2h2a2 2 0 012 2v2a2 2 0 01-2 2h-2a2 2 0 01-2-2v-2z" />
    </svg>
  );
}

function MasonryIcon() {
  return (
    <svg className="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
      <path d="M2 4a2 2 0 012-2h12a2 2 0 012 2v12a2 2 0 01-2 2H4a2 2 0 01-2-2V4z" />
    </svg>
  );
}

function ListIcon() {
  return (
    <svg className="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
      <path fillRule="evenodd" d="M3 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm0 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm0 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm0 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1z" clipRule="evenodd" />
    </svg>
  );
}
Project Card Component
typescript
// components/projects/ProjectCard.tsx
'use client';

import { Project } from '@/types/portfolio';
import Image from 'next/image';
import Link from 'next/link';
import { motion } from 'framer-motion';

interface ProjectCardProps {
  project: Project;
  layout: 'masonry' | 'grid' | 'list';
}

export default function ProjectCard({ project, layout }: ProjectCardProps) {
  const mainImage = project.media[0];
  
  const cardVariants = {
    hidden: { opacity: 0, y: 20 },
    visible: { 
      opacity: 1, 
      y: 0,
      transition: { duration: 0.3 }
    }
  };

  if (layout === 'list') {
    return (
      <motion.article
        variants={cardVariants}
        initial="hidden"
        animate="visible"
        whileHover={{ y: -4 }}
        className="group"
      >
        <Link href={`/projects/${project.slug}`}>
          <div className="flex flex-col md:flex-row gap-6 p-6 rounded-2xl border border-gray-200 hover:border-blue-300 hover:shadow-lg transition-all duration-300">
            {mainImage && (
              <div className="md:w-48 lg:w-64 flex-shrink-0">
                <div className="relative aspect-video md:aspect-square rounded-xl overflow-hidden">
                  <Image
                    src={mainImage.url}
                    alt={mainImage.alt || project.title}
                    fill
                    sizes="(max-width: 768px) 100vw, 256px"
                    className="object-cover group-hover:scale-105 transition-transform duration-500"
                  />
                </div>
              </div>
            )}
            
            <div className="flex-1">
              <div className="flex flex-wrap items-center gap-3 mb-3">
                <span className="text-sm font-medium text-blue-600 bg-blue-50 px-3 py-1 rounded-full">
                  {project.year}
                </span>
                {project.categories.map((category) => (
                  <span
                    key={category.id}
                    className="text-sm font-medium text-gray-600 bg-gray-100 px-3 py-1 rounded-full"
                  >
                    {category.name}
                  </span>
                ))}
              </div>
              
              <h3 className="text-xl font-bold text-gray-900 mb-3 group-hover:text-blue-600 transition-colors">
                {project.title}
              </h3>
              
              <p className="text-gray-600 mb-4 line-clamp-2">
                {project.excerpt || project.description}
              </p>
              
              <div className="flex flex-wrap gap-2">
                {project.skills.slice(0, 3).map((skill) => (
                  <span
                    key={skill.id}
                    className="text-xs font-medium text-gray-500 bg-gray-50 px-2 py-1 rounded"
                  >
                    {skill.name}
                  </span>
                ))}
                {project.skills.length > 3 && (
                  <span className="text-xs text-gray-400">
                    +{project.skills.length - 3} more
                  </span>
                )}
              </div>
            </div>
          </div>
        </Link>
      </motion.article>
    );
  }

  return (
    <motion.article
      variants={cardVariants}
      initial="hidden"
      animate="visible"
      whileHover={{ y: -8 }}
      className="group break-inside-avoid"
    >
      <Link href={`/projects/${project.slug}`}>
        <div className="rounded-2xl overflow-hidden border border-gray-200 hover:shadow-xl transition-all duration-300">
          {mainImage && (
            <div className="relative aspect-[4/3] overflow-hidden bg-gray-100">
              <Image
                src={mainImage.url}
                alt={mainImage.alt || project.title}
                fill
                sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
                className="object-cover group-hover:scale-110 transition-transform duration-700"
              />
              <div className="absolute inset-0 bg-gradient-to-t from-black/40 via-transparent to-transparent opacity-0 group-hover:opacity-100 transition-opacity duration-300" />
              
              {project.featured && (
                <div className="absolute top-4 right-4">
                  <span className="px-3 py-1 bg-yellow-500 text-white text-xs font-bold rounded-full">
                    Featured
                  </span>
                </div>
              )}
            </div>
          )}
          
          <div className="p-6">
            <div className="flex flex-wrap items-center gap-2 mb-3">
              <span className="text-sm font-medium text-blue-600">
                {project.year}
              </span>
              <span className="text-gray-300">•</span>
              {project.categories.slice(0, 2).map((category) => (
                <span
                  key={category.id}
                  className="text-sm font-medium text-gray-500"
                >
                  {category.name}
                </span>
              ))}
            </div>
            
            <h3 className="text-lg font-bold text-gray-900 mb-2 group-hover:text-blue-600 transition-colors">
              {project.title}
            </h3>
            
            <p className="text-gray-600 text-sm mb-4 line-clamp-2">
              {project.excerpt || project.description}
            </p>
            
            <div className="flex items-center justify-between">
              <div className="flex flex-wrap gap-1">
                {project.skills.slice(0, 2).map((skill) => (
                  <span
                    key={skill.id}
                    className="text-xs text-gray-400"
                  >
                    {skill.name}
                  </span>
                ))}
              </div>
              
              <span className="text-blue-600 text-sm font-medium group-hover:translate-x-1 transition-transform">
                View case →
              </span>
            </div>
          </div>
        </div>
      </Link>
    </motion.article>
  );
}
Project Detail Page
typescript
// app/projects/[slug]/page.tsx
import { ProjectRepository } from '@/lib/repositories/project.repository';
import ProjectDetail from '@/components/projects/ProjectDetail';
import { notFound } from 'next/navigation';
import { Metadata } from 'next';

interface PageProps {
  params: Promise<{ slug: string }>;
}

export async function generateMetadata({ params }: PageProps): Promise<Metadata> {
  const { slug } = await params;
  const project = await ProjectRepository.getProjectBySlug(slug);
  
  if (!project) {
    return {
      title: 'Project Not Found',
    };
  }
  
  return {
    title: project.metaTitle || `${project.title} | Portfolio`,
    description: project.metaDescription || project.description,
    openGraph: {
      title: project.metaTitle || project.title,
      description: project.metaDescription || project.description,
      images: project.media.length > 0 ? [project.media[0].url] : [],
      type: 'article',
    },
  };
}

export default async function ProjectPage({ params }: PageProps) {
  const { slug } = await params;
  const project = await ProjectRepository.getProjectBySlug(slug);
  
  if (!project) {
    notFound();
  }
  
  return <ProjectDetail project={project} />;
}
typescript
// components/projects/ProjectDetail.tsx
'use client';

import { Project } from '@/types/portfolio';
import ImageGallery from '@/components/gallery/ImageGallery';
import CaseStudy from '@/components/case-study/CaseStudy';
import { BeforeAfterSlider } from '@/components/ui/BeforeAfterSlider';
import TestimonialsSlider from '@/components/testimonials/TestimonialsSlider';

interface ProjectDetailProps {
  project: Project;
}

export default function ProjectDetail({ project }: ProjectDetailProps) {
  const hasBeforeAfter = project.media.some(m => 
    m.caption?.toLowerCase().includes('before') || 
    m.caption?.toLowerCase().includes('after')
  );

  return (
    <article className="min-h-screen">
      {/* Hero Section */}
      <section className="relative pt-24 pb-16 px-4 sm:px-6 lg:px-8">
        <div className="max-w-7xl mx-auto">
          <div className="max-w-3xl">
            <div className="flex flex-wrap gap-3 mb-6">
              {project.categories.map((category) => (
                <span
                  key={category.id}
                  style={{ backgroundColor: `${category.color}20` }}
                  className="px-4 py-2 rounded-full text-sm font-medium"
                >
                  {category.name}
                </span>
              ))}
            </div>
            
            <h1 className="text-4xl md:text-5xl lg:text-6xl font-bold text-gray-900 mb-6">
              {project.title}
            </h1>
            
            <p className="text-xl text-gray-600 mb-8">
              {project.description}
            </p>
            
            <div className="flex flex-wrap gap-6 text-sm text-gray-500">
              {project.year && (
                <div>
                  <span className="font-medium">Year:</span> {project.year}
                </div>
              )}
              {project.client && (
                <div>
                  <span className="font-medium">Client:</span> {project.client}
                </div>
              )}
              {project.duration && (
                <div>
                  <span className="font-medium">Duration:</span> {project.duration}
                </div>
              )}
              {project.location && (
                <div>
                  <span className="font-medium">Location:</span> {project.location}
                </div>
              )}
            </div>
          </div>
        </div>
      </section>

      {/* Main Image/Video */}
      {project.media.length > 0 && (
        <section className="py-8">
          <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div className="relative aspect-[16/9] rounded-2xl overflow-hidden shadow-2xl">
              {project.media[0].type === 'VIDEO' ? (
                <video
                  src={project.media[0].url}
                  poster={project.media[0].thumbnail}
                  controls
                  className="w-full h-full object-cover"
                  autoPlay
                  muted
                  loop
                />
              ) : (
                <ImageGallery
                  media={project.media}
                  initialIndex={0}
                  showThumbnails
                />
              )}
            </div>
          </div>
        </section>
      )}

      {/* Before/After Slider */}
      {hasBeforeAfter && (
        <section className="py-16 bg-gray-50">
          <div className="max-w-5xl mx-auto px-4 sm:px-6 lg:px-8">
            <h2 className="text-3xl font-bold text-center mb-12">
              Before & After
            </h2>
            <BeforeAfterSlider
              beforeImage={project.media.find(m => 
                m.caption?.toLowerCase().includes('before')
              )}
              afterImage={project.media.find(m => 
                m.caption?.toLowerCase().includes('after')
              )}
            />
          </div>
        </section>
      )}

      {/* Case Study Content */}
      {project.content && (
        <section className="py-16">
          <div className="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8">
            <CaseStudy content={project.content} />
          </div>
        </section>
      )}

      {/* Skills Used */}
      <section className="py-16 bg-gray-50">
        <div className="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8">
          <h2 className="text-3xl font-bold text-center mb-12">
            Technologies & Skills
          </h2>
          <div className="flex flex-wrap justify-center gap-3">
            {project.skills.map((skill) => (
              <div
                key={skill.id}
                className="flex items-center gap-2 bg-white px-6 py-3 rounded-full shadow-md"
              >
                {skill.icon && (
                  <span className="text-xl">{skill.icon}</span>
                )}
                <span className="font-medium text-gray-800">
                  {skill.name}
                </span>
                <div className="w-24 bg-gray-200 rounded-full h-2">
                  <div
                    className="bg-blue-600 h-2 rounded-full"
                    style={{ width: `${skill.proficiency}%` }}
                  />
                </div>
              </div>
            ))}
          </div>
        </div>
      </section>

      {/* Testimonials */}
      {project.testimonials.length > 0 && (
        <section className="py-16">
          <div className="max-w-6xl mx-auto px-4 sm:px-6 lg:px-8">
            <TestimonialsSlider testimonials={project.testimonials} />
          </div>
        </section>
      )}

      {/* Image Gallery */}
      {project.media.length > 1 && (
        <section className="py-16">
          <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <h2 className="text-3xl font-bold text-center mb-12">
              Project Gallery
            </h2>
            <ImageGallery
              media={project.media.slice(1)}
              showThumbnails
              gridLayout="masonry"
            />
          </div>
        </section>
      )}
    </article>
  );
}
Before/After Slider Component
typescript
// components/ui/BeforeAfterSlider.tsx
'use client';

import { useState, useRef, useEffect } from 'react';
import Image from 'next/image';
import { Media } from '@/types/portfolio';

interface BeforeAfterSliderProps {
  beforeImage?: Media;
  afterImage?: Media;
  initialPosition?: number; // 0-100
  orientation?: 'horizontal' | 'vertical';
}

export function BeforeAfterSlider({
  beforeImage,
  afterImage,
  initialPosition = 50,
  orientation = 'horizontal'
}: BeforeAfterSliderProps) {
  const [position, setPosition] = useState(initialPosition);
  const containerRef = useRef<HTMLDivElement>(null);
  const isDragging = useRef(false);

  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      if (!isDragging.current || !containerRef.current) return;
      
      const rect = containerRef.current.getBoundingClientRect();
      let newPosition;
      
      if (orientation === 'horizontal') {
        newPosition = ((e.clientX - rect.left) / rect.width) * 100;
      } else {
        newPosition = ((e.clientY - rect.top) / rect.height) * 100;
      }
      
      setPosition(Math.min(Math.max(newPosition, 0), 100));
    };

    const handleMouseUp = () => {
      isDragging.current = false;
      document.body.style.cursor = 'default';
    };

    document.addEventListener('mousemove', handleMouseMove);
    document.addEventListener('mouseup', handleMouseUp);

    return () => {
      document.removeEventListener('mousemove', handleMouseMove);
      document.removeEventListener('mouseup', handleMouseUp);
    };
  }, [orientation]);

  const handleMouseDown = () => {
    isDragging.current = true;
    document.body.style.cursor = orientation === 'horizontal' ? 'ew-resize' : 'ns-resize';
  };

  if (!beforeImage || !afterImage) {
    return null;
  }

  const sliderStyle = {
    horizontal: {
      container: 'aspect-video',
      before: { clipPath: `inset(0 ${100 - position}% 0 0)` },
      after: { clipPath: `inset(0 0 0 ${position}%)` },
      slider: {
        left: `${position}%`,
        transform: 'translateX(-50%)'
      }
    },
    vertical: {
      container: 'aspect-[4/5]',
      before: { clipPath: `inset(0 0 ${100 - position}% 0)` },
      after: { clipPath: `inset(${position}% 0 0 0)` },
      slider: {
        top: `${position}%`,
        transform: 'translateY(-50%)'
      }
    }
  }[orientation];

  return (
    <div className="relative rounded-2xl overflow-hidden shadow-2xl">
      <div
        ref={containerRef}
        className={`relative ${sliderStyle.container} cursor-${orientation === 'horizontal' ? 'ew' : 'ns'}-resize`}
      >
        {/* Before Image */}
        <div className="absolute inset-0">
          <div className="relative w-full h-full" style={sliderStyle.before}>
            <Image
              src={beforeImage.url}
              alt={beforeImage.alt || 'Before'}
              fill
              className="object-cover"
              sizes="100vw"
            />
          </div>
          <div className="absolute top-4 left-4 bg-black/70 text-white px-4 py-2 rounded-full text-sm font-bold">
            BEFORE
          </div>
        </div>

        {/* After Image */}
        <div className="absolute inset-0">
          <div className="relative w-full h-full" style={sliderStyle.after}>
            <Image
              src={afterImage.url}
              alt={afterImage.alt || 'After'}
              fill
              className="object-cover"
              sizes="100vw"
            />
          </div>
          <div className="absolute top-4 right-4 bg-black/70 text-white px-4 py-2 rounded-full text-sm font-bold">
            AFTER
          </div>
        </div>

        {/* Slider Control */}
        <div
          className={`absolute ${orientation === 'horizontal' ? 'top-0 bottom-0' : 'left-0 right-0'} cursor-${orientation === 'horizontal' ? 'ew' : 'ns'}-resize`}
          style={sliderStyle.slider}
          onMouseDown={handleMouseDown}
        >
          <div className={`
            absolute ${orientation === 'horizontal' ? 'top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2' : 'left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2'}
            w-12 h-12 rounded-full bg-white shadow-2xl flex items-center justify-center
          `}>
            <div className="flex items-center">
              <div className="w-1 h-4 bg-gray-400 mx-0.5" />
              <div className="w-1 h-4 bg-gray-400 mx-0.5" />
              <div className="w-1 h-4 bg-gray-400 mx-0.5" />
            </div>
          </div>
          <div className={`
            absolute ${orientation === 'horizontal' ? 'top-0 bottom-0 w-0.5' : 'left-0 right-0 h-0.5'}
            bg-white shadow-lg
          `} />
        </div>
      </div>
    </div>
  );
}
§ IMAGE GALLERY
Lightbox Component
typescript
// components/gallery/Lightbox.tsx
'use client';

import { useState, useEffect, useCallback } from 'react';
import Image from 'next/image';
import { Media } from '@/types/portfolio';
import { motion, AnimatePresence } from 'framer-motion';
import { ChevronLeft, ChevronRight, X, ZoomIn, ZoomOut, Maximize2 } from 'lucide-react';

interface LightboxProps {
  isOpen: boolean;
  onClose: () => void;
  media: Media[];
  initialIndex?: number;
}

export default function Lightbox({ isOpen, onClose, media, initialIndex = 0 }: LightboxProps) {
  const [currentIndex, setCurrentIndex] = useState(initialIndex);
  const [scale, setScale] = useState(1);
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [isDragging, setIsDragging] = useState(false);
  const [startPosition, setStartPosition] = useState({ x: 0, y: 0 });

  const currentMedia = media[currentIndex];

  const handlePrevious = useCallback(() => {
    setCurrentIndex((prev) => (prev === 0 ? media.length - 1 : prev - 1));
    resetZoom();
  }, [media.length]);

  const handleNext = useCallback(() => {
    setCurrentIndex((prev) => (prev === media.length - 1 ? 0 : prev + 1));
    resetZoom();
  }, [media.length]);

  const resetZoom = () => {
    setScale(1);
    setPosition({ x: 0, y: 0 });
  };

  const handleZoomIn = () => {
    setScale((prev) => Math.min(prev * 1.3, 4));
  };

  const handleZoomOut = () => {
    setScale((prev) => Math.max(prev / 1.3, 1));
  };

  const handleFullscreen = () => {
    const element = document.documentElement;
    if (!document.fullscreenElement) {
      element.requestFullscreen().catch(console.error);
    } else {
      document.exitFullscreen();
    }
  };

  const handleWheel = useCallback((e: WheelEvent) => {
    if (e.ctrlKey) {
      e.preventDefault();
      set

════════════════════════════════════════════════════════════
FIGMA CATALOG: BATCH2-12-PORTFOLIO-GALLERY
Prompt ID: 12 / 19
Parte: 2
Exported: 2026-02-06T16:28:25.096Z
Characters: 1266
════════════════════════════════════════════════════════════

=== 0 ? (
                          <>
                            <span>{activity}</span>
                            <div className="w-2 h-2 rounded-full bg-blue-400" />
                          </>
                        ) : (
                          <>
                            <div className="w-2 h-2 rounded-full bg-blue-400" />
                            <span>{activity}</span>
                          </>
                        )}
                      </li>
                    ))}
                  </ul>
                </div>
              </div>
            </div>
          ))}
        </div>
      </motion.section>

      {/* Results Section */}
      <motion.section
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        className="space-y-6"
      >
        <div className="flex items-center gap-3">
          <div className="p-3 rounded-xl bg-blue-100 text-blue-600">
            <BarChart3 className="w-6 h-6" />
          </div>
          <h2 className="text-2xl font-bold">Measurable Results</h2>
        </div>
        
        {/* Metrics */}
        <div className="grid md:grid-cols-3 gap-6">
          {metrics.map((metric, index) => {
            const Icon = metric.icon;
            return (

## § ADVANCED PATTERNS: PORTFOLIO GALLERY

### Server Actions con Validazione

```typescript
// app/actions/portfolio-gallery.ts
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


### PORTFOLIO GALLERY - Utility Helper #716

```typescript
// lib/utils/portfolio-gallery-helper-716.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #492

```typescript
// lib/utils/portfolio-gallery-helper-492.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #676

```typescript
// lib/utils/portfolio-gallery-helper-676.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #901

```typescript
// lib/utils/portfolio-gallery-helper-901.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #17

```typescript
// lib/utils/portfolio-gallery-helper-17.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #603

```typescript
// lib/utils/portfolio-gallery-helper-603.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #620

```typescript
// lib/utils/portfolio-gallery-helper-620.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #604

```typescript
// lib/utils/portfolio-gallery-helper-604.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #406

```typescript
// lib/utils/portfolio-gallery-helper-406.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #786

```typescript
// lib/utils/portfolio-gallery-helper-786.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #771

```typescript
// lib/utils/portfolio-gallery-helper-771.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #61

```typescript
// lib/utils/portfolio-gallery-helper-61.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #290

```typescript
// lib/utils/portfolio-gallery-helper-290.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #643

```typescript
// lib/utils/portfolio-gallery-helper-643.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #482

```typescript
// lib/utils/portfolio-gallery-helper-482.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #168

```typescript
// lib/utils/portfolio-gallery-helper-168.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
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

  getConfig(): Readonly<PORTFOLIOGALLERYConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PORTFOLIO GALLERY - Utility Helper #972

```typescript
// lib/utils/portfolio-gallery-helper-972.ts
import { z } from "zod";

interface PORTFOLIOGALLERYConfig {
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

export class PORTFOLIOGALLERYProcessor<TInput, TOutput> {
  private config: PORTFOLIOGALLERYConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PORTFOLIOGALLERYConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (