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