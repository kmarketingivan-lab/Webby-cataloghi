# CATALOGO-LMS-LEARNING

LMS Learning Platform - Next.js 14 + Prisma Complete Catalog
§ COURSE ARCHITECTURE
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

// Enums
enum ContentType {
  VIDEO
  TEXT
  QUIZ
  ASSIGNMENT
  LIVE_SESSION
  RESOURCE
}

enum CourseStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

enum LessonStatus {
  DRAFT
  PUBLISHED
  HIDDEN
}

enum DifficultyLevel {
  BEGINNER
  INTERMEDIATE
  ADVANCED
}

enum QuizQuestionType {
  MULTIPLE_CHOICE
  TRUE_FALSE
  MATCHING
  FILL_BLANK
  ESSAY
}

// Core Models
model User {
  id                    String    @id @default(cuid())
  email                 String    @unique
  name                  String?
  role                  UserRole  @default(STUDENT)
  image                 String?
  
  // Relationships
  enrollments          Enrollment[]
  createdCourses       Course[]   @relation("InstructorCourses")
  courseProgress       CourseProgress[]
  lessonProgress       LessonProgress[]
  quizAttempts         QuizAttempt[]
  assignments          AssignmentSubmission[]
  certificates         Certificate[]
  userBadges           UserBadge[]
  points               PointTransaction[]
  forumPosts           ForumPost[]
  forumReplies         ForumReply[]
  
  createdAt            DateTime   @default(now())
  updatedAt            DateTime   @updatedAt
}

enum UserRole {
  STUDENT
  INSTRUCTOR
  ADMIN
}

model Course {
  id                  String     @id @default(cuid())
  title               String
  slug                String     @unique
  description         String?
  shortDescription    String?
  thumbnail           String?
  previewVideo        String?
  
  status              CourseStatus @default(DRAFT)
  version             Int        @default(1)
  previousVersionId   String?
  
  difficulty          DifficultyLevel @default(BEGINNER)
  category            String?
  tags                String[]
  language            String     @default("en")
  estimatedHours      Float      @default(0)
  
  // Pricing
  price               Decimal?   @db.Decimal(10, 2)
  isFree              Boolean    @default(false)
  discountPrice       Decimal?   @db.Decimal(10, 2)
  currency            String     @default("USD")
  
  // Metadata
  rating              Float?     @default(0)
  totalRatings        Int        @default(0)
  totalStudents       Int        @default(0)
  
  // Relationships
  instructorId        String
  instructor          User       @relation("InstructorCourses", fields: [instructorId], references: [id])
  
  sections            Section[]
  enrollments         Enrollment[]
  courseProgress      CourseProgress[]
  certificates        Certificate[]
  discussions         CourseDiscussion[]
  resources           CourseResource[]
  
  publishedAt         DateTime?
  createdAt           DateTime   @default(now())
  updatedAt           DateTime   @updatedAt
}

model Section {
  id                  String     @id @default(cuid())
  title               String
  description         String?
  order               Int
  isPreview           Boolean    @default(false)
  
  courseId            String
  course              Course     @relation(fields: [courseId], references: [id])
  
  lessons             Lesson[]
  
  createdAt           DateTime   @default(now())
  updatedAt           DateTime   @updatedAt
  
  @@unique([courseId, order])
}

model Lesson {
  id                  String     @id @default(cuid())
  title               String
  slug                String
  description         String?
  order               Int
  contentType         ContentType
  
  // Content based on type
  videoUrl            String?
  videoDuration       Int?       // in seconds
  videoCaptions       Json?      // JSON array of caption tracks
  content             String?    // Markdown content
  quizId              String?    // Reference to Quiz if contentType is QUIZ
  assignmentId        String?    // Reference to Assignment if contentType is ASSIGNMENT
  liveSessionId       String?    // Reference to LiveSession if contentType is LIVE_SESSION
  resourceId          String?    // Reference to Resource if contentType is RESOURCE
  
  // Metadata
  status              LessonStatus @default(DRAFT)
  isPreview           Boolean    @default(false)
  estimatedMinutes    Int        @default(0)
  
  // Relationships
  sectionId           String
  section             Section    @relation(fields: [sectionId], references: [id])
  
  progress            LessonProgress[]
  forumPosts          ForumPost[]
  bookmarks           Bookmark[]
  notes               Note[]
  
  createdAt           DateTime   @default(now())
  updatedAt           DateTime   @updatedAt
  
  @@unique([sectionId, order])
  @@unique([sectionId, slug])
}

// Course Versioning Table
model CourseVersion {
  id                  String     @id @default(cuid())
  courseId            String
  version             Int
  data                Json       // Full course JSON snapshot
  changelog           String?
  createdBy           String
  
  createdAt           DateTime   @default(now())
  
  @@unique([courseId, version])
}
§ CONTENT TYPES
prisma
// Continuation of schema.prisma

// Quiz Models
model Quiz {
  id                  String     @id @default(cuid())
  title               String
  description         String?
  passingScore        Int        @default(70)
  timeLimit           Int?       // in minutes
  maxAttempts         Int        @default(1)
  shuffleQuestions    Boolean    @default(false)
  showAnswers         Boolean    @default(false)
  
  questions           QuizQuestion[]
  attempts            QuizAttempt[]
  
  createdAt           DateTime   @default(now())
  updatedAt           DateTime   @updatedAt
}

model QuizQuestion {
  id                  String     @id @default(cuid())
  quizId              String
  quiz                Quiz       @relation(fields: [quizId], references: [id])
  
  order               Int
  questionType        QuizQuestionType
  question            String
  explanation         String?
  points              Int        @default(1)
  
  // For multiple choice/true-false
  options             Json?      // { id: string, text: string, isCorrect: boolean }[]
  correctAnswer       String?    // For fill-blank, true/false
  
  // For matching questions
  matchingPairs       Json?      // { from: string, to: string }[]
  
  createdAt           DateTime   @default(now())
  updatedAt           DateTime   @updatedAt
}

// Assignment Models
model Assignment {
  id                  String     @id @default(cuid())
  title               String
  description         String?
  instructions        String
  dueDate             DateTime?
  maxPoints           Int        @default(100)
  allowLateSubmission Boolean    @default(false)
  attachments         String[]   // URLs to assignment resources
  
  submissions         AssignmentSubmission[]
  
  createdAt           DateTime   @default(now())
  updatedAt           DateTime   @updatedAt
}

model AssignmentSubmission {
  id                  String     @id @default(cuid())
  assignmentId        String
  assignment          Assignment @relation(fields: [assignmentId], references: [id])
  
  userId              String
  user                User       @relation(fields: [userId], references: [id])
  
  content             String?
  attachments         String[]
  grade               Int?
  feedback            String?
  submittedAt         DateTime   @default(now())
  gradedAt            DateTime?
  
  createdAt           DateTime   @default(now())
  updatedAt           DateTime   @updatedAt
}

// Live Session Models
model LiveSession {
  id                  String     @id @default(cuid())
  title               String
  description         String?
  startTime           DateTime
  endTime             DateTime
  meetingUrl          String
  recordingUrl        String?
  isRecorded          Boolean    @default(false)
  
  createdAt           DateTime   @default(now())
  updatedAt           DateTime   @updatedAt
}

// Resource Models
model Resource {
  id                  String     @id @default(cuid())
  title               String
  description         String?
  fileUrl             String
  fileType            String
  fileSize            Int
  downloadCount       Int        @default(0)
  
  createdAt           DateTime   @default(now())
  updatedAt           DateTime   @updatedAt
}

model CourseResource {
  id                  String     @id @default(cuid())
  courseId            String
  course              Course     @relation(fields: [courseId], references: [id])
  
  resourceId          String
  resource            Resource   @relation(fields: [resourceId], references: [id])
  
  isPublic            Boolean    @default(false)
  
  createdAt           DateTime   @default(now())
}
§ VIDEO PLAYER
typescript
// components/video/VideoPlayer.tsx
'use client';

import React, { useState, useEffect, useRef } from 'react';
import ReactPlayer from 'react-player';
import { Play, Pause, Volume2, Settings, Captions } from 'lucide-react';

interface VideoPlayerProps {
  url: string;
  duration?: number;
  captions?: Array<{
    language: string;
    url: string;
    label: string;
  }>;
  onProgress?: (progress: number) => void;
  onComplete?: () => void;
  autoPlay?: boolean;
}

interface PlaybackQuality {
  label: string;
  value: string;
}

const VideoPlayer: React.FC<VideoPlayerProps> = ({
  url,
  duration,
  captions = [],
  onProgress,
  onComplete,
  autoPlay = false
}) => {
  const [playing, setPlaying] = useState(autoPlay);
  const [volume, setVolume] = useState(0.8);
  const [playbackRate, setPlaybackRate] = useState(1);
  const [quality, setQuality] = useState<string>('auto');
  const [selectedCaption, setSelectedCaption] = useState<string>('off');
  const [progress, setProgress] = useState(0);
  const [playedSeconds, setPlayedSeconds] = useState(0);
  const [seeking, setSeeking] = useState(false);
  const playerRef = useRef<ReactPlayer>(null);

  const playbackRates = [0.5, 0.75, 1, 1.25, 1.5, 1.75, 2];
  const qualities: PlaybackQuality[] = [
    { label: 'Auto', value: 'auto' },
    { label: '360p', value: '360p' },
    { label: '480p', value: '480p' },
    { label: '720p', value: '720p' },
    { label: '1080p', value: '1080p' },
  ];

  const handleProgress = (state: { played: number; playedSeconds: number }) => {
    if (!seeking) {
      setProgress(state.played);
      setPlayedSeconds(state.playedSeconds);
      onProgress?.(state.played);
    }
  };

  const handleEnded = () => {
    setPlaying(false);
    onComplete?.();
  };

  const handleSeekChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = parseFloat(e.target.value);
    setProgress(value);
  };

  const handleSeekMouseDown = () => {
    setSeeking(true);
  };

  const handleSeekMouseUp = (e: React.MouseEvent<HTMLInputElement>) => {
    setSeeking(false);
    playerRef.current?.seekTo(parseFloat(e.currentTarget.value));
  };

  const formatTime = (seconds: number) => {
    const date = new Date(seconds * 1000);
    const hh = date.getUTCHours();
    const mm = date.getUTCMinutes();
    const ss = date.getUTCSeconds().toString().padStart(2, '0');
    
    if (hh) {
      return `${hh}:${mm.toString().padStart(2, '0')}:${ss}`;
    }
    return `${mm}:${ss}`;
  };

  return (
    <div className="relative bg-black rounded-lg overflow-hidden">
      <ReactPlayer
        ref={playerRef}
        url={url}
        playing={playing}
        volume={volume}
        playbackRate={playbackRate}
        width="100%"
        height="100%"
        onProgress={handleProgress}
        onEnded={handleEnded}
        config={{
          file: {
            attributes: {
              controlsList: 'nodownload',
            },
            tracks: captions.map(caption => ({
              kind: 'subtitles',
              src: caption.url,
              srcLang: caption.language,
              label: caption.label,
            })),
          },
        }}
      />
      
      {/* Custom Controls */}
      <div className="absolute bottom-0 left-0 right-0 bg-gradient-to-t from-black/90 to-transparent p-4">
        {/* Progress Bar */}
        <div className="mb-3">
          <input
            type="range"
            min={0}
            max={0.999999}
            step="any"
            value={progress}
            onChange={handleSeekChange}
            onMouseDown={handleSeekMouseDown}
            onMouseUp={handleSeekMouseUp}
            className="w-full h-1.5 bg-gray-600 rounded-lg appearance-none cursor-pointer"
          />
        </div>
        
        <div className="flex items-center justify-between">
          <div className="flex items-center space-x-4">
            <button
              onClick={() => setPlaying(!playing)}
              className="p-2 hover:bg-white/10 rounded-full"
            >
              {playing ? <Pause size={20} /> : <Play size={20} />}
            </button>
            
            <div className="flex items-center space-x-2">
              <Volume2 size={18} />
              <input
                type="range"
                min={0}
                max={1}
                step={0.1}
                value={volume}
                onChange={(e) => setVolume(parseFloat(e.target.value))}
                className="w-20 h-1.5 bg-gray-600 rounded-lg appearance-none cursor-pointer"
              />
            </div>
            
            <div className="text-sm text-gray-300">
              {formatTime(playedSeconds)} / {duration ? formatTime(duration) : '--:--'}
            </div>
          </div>
          
          <div className="flex items-center space-x-4">
            {/* Playback Speed */}
            <div className="relative group">
              <button className="p-2 hover:bg-white/10 rounded-full">
                <Settings size={18} />
              </button>
              <div className="absolute bottom-full right-0 mb-2 hidden group-hover:block bg-gray-900 rounded-lg p-2 min-w-[120px]">
                {playbackRates.map(rate => (
                  <button
                    key={rate}
                    onClick={() => setPlaybackRate(rate)}
                    className={`block w-full text-left px-3 py-2 text-sm rounded hover:bg-gray-800 ${
                      playbackRate === rate ? 'bg-blue-600' : ''
                    }`}
                  >
                    {rate}x Speed
                  </button>
                ))}
              </div>
            </div>
            
            {/* Quality Selection */}
            <div className="relative group">
              <button className="px-3 py-1 text-sm bg-gray-800 hover:bg-gray-700 rounded">
                {qualities.find(q => q.value === quality)?.label || 'Auto'}
              </button>
              <div className="absolute bottom-full right-0 mb-2 hidden group-hover:block bg-gray-900 rounded-lg p-2 min-w-[100px]">
                {qualities.map(q => (
                  <button
                    key={q.value}
                    onClick={() => setQuality(q.value)}
                    className={`block w-full text-left px-3 py-2 text-sm rounded hover:bg-gray-800 ${
                      quality === q.value ? 'bg-blue-600' : ''
                    }`}
                  >
                    {q.label}
                  </button>
                ))}
              </div>
            </div>
            
            {/* Captions */}
            {captions.length > 0 && (
              <div className="relative group">
                <button className="p-2 hover:bg-white/10 rounded-full">
                  <Captions size={18} />
                </button>
                <div className="absolute bottom-full right-0 mb-2 hidden group-hover:block bg-gray-900 rounded-lg p-2 min-w-[120px]">
                  <button
                    onClick={() => setSelectedCaption('off')}
                    className={`block w-full text-left px-3 py-2 text-sm rounded hover:bg-gray-800 ${
                      selectedCaption === 'off' ? 'bg-blue-600' : ''
                    }`}
                  >
                    Off
                  </button>
                  {captions.map(caption => (
                    <button
                      key={caption.language}
                      onClick={() => setSelectedCaption(caption.language)}
                      className={`block w-full text-left px-3 py-2 text-sm rounded hover:bg-gray-800 ${
                        selectedCaption === caption.language ? 'bg-blue-600' : ''
                      }`}
                    >
                      {caption.label}
                    </button>
                  ))}
                </div>
              </div>
            )}
          </div>
        </div>
      </div>
    </div>
  );
};

export default VideoPlayer;
typescript
// hooks/useVideoProgress.ts
import { useEffect } from 'react';
import { saveProgress, getProgress } from '@/lib/api/progress';

interface UseVideoProgressProps {
  userId: string;
  lessonId: string;
  courseId: string;
}

export const useVideoProgress = ({ userId, lessonId, courseId }: UseVideoProgressProps) => {
  const [progress, setProgress] = useState(0);
  const [lastPosition, setLastPosition] = useState(0);
  const [isCompleted, setIsCompleted] = useState(false);

  useEffect(() => {
    // Load saved progress
    const loadProgress = async () => {
      const saved = await getProgress(userId, lessonId, courseId);
      if (saved) {
        setProgress(saved.progress);
        setLastPosition(saved.lastPosition);
        setIsCompleted(saved.completed);
      }
    };
    loadProgress();
  }, [userId, lessonId, courseId]);

  const saveVideoProgress = async (currentProgress: number, currentTime: number) => {
    const completed = currentProgress >= 0.95; // Mark as completed if watched 95%
    
    await saveProgress({
      userId,
      lessonId,
      courseId,
      progress: currentProgress,
      lastPosition: currentTime,
      completed,
      updatedAt: new Date(),
    });

    if (completed && !isCompleted) {
      setIsCompleted(true);
    }
  };

  const resumeFromLastPosition = () => {
    return lastPosition;
  };

  return {
    progress,
    isCompleted,
    saveVideoProgress,
    resumeFromLastPosition,
  };
};
§ PROGRESS TRACKING
prisma
// Continuation of schema.prisma - Progress Tracking Models

model CourseProgress {
  id                  String     @id @default(cuid())
  
  userId              String
  user                User       @relation(fields: [userId], references: [id])
  
  courseId            String
  course              Course     @relation(fields: [courseId], references: [id])
  
  completed           Boolean    @default(false)
  progressPercentage  Float      @default(0)
  completedLessons    Int        @default(0)
  totalLessons        Int        @default(0)
  timeSpent           Int        @default(0) // in seconds
  
  lastAccessedAt      DateTime?
  startedAt           DateTime   @default(now())
  completedAt         DateTime?
  
  lessonProgress      LessonProgress[]
  quizProgress        QuizProgress[]
  
  createdAt           DateTime   @default(now())
  updatedAt           DateTime   @updatedAt
  
  @@unique([userId, courseId])
}

model LessonProgress {
  id                  String     @id @default(cuid())
  
  userId              String
  user                User       @relation(fields: [userId], references: [id])
  
  lessonId            String
  lesson              Lesson     @relation(fields: [lessonId], references: [id])
  
  courseProgressId    String
  courseProgress      CourseProgress @relation(fields: [courseProgressId], references: [id])
  
  completed           Boolean    @default(false)
  progress            Float      @default(0) // 0 to 1
  lastPosition        Int        @default(0) // for videos, in seconds
  timeSpent           Int        @default(0) // in seconds
  
  firstAccessedAt     DateTime?
  lastAccessedAt      DateTime?
  completedAt         DateTime?
  
  notes               Note[]
  bookmarks           Bookmark[]
  
  createdAt           DateTime   @default(now())
  updatedAt           DateTime   @updatedAt
  
  @@unique([userId, lessonId])
}

model QuizProgress {
  id                  String     @id @default(cuid())
  
  userId              String
  user                User       @relation(fields: [userId], references: [id])
  
  quizId              String
  quiz                Quiz       @relation(fields: [quizId], references: [id])
  
  courseProgressId    String
  courseProgress      CourseProgress @relation(fields: [courseProgressId], references: [id])
  
  score               Float?
  passed              Boolean?
  attemptNumber       Int        @default(1)
  timeSpent           Int?       // in seconds
  answers             Json       // User's answers
  
  startedAt           DateTime?
  completedAt         DateTime?
  
  createdAt           DateTime   @default(now())
  updatedAt           DateTime   @updatedAt
  
  @@unique([userId, quizId, attemptNumber])
}

// Certificate Model
model Certificate {
  id                  String     @id @default(cuid())
  certificateNumber   String     @unique
  userId              String
  user                User       @relation(fields: [userId], references: [id])
  
  courseId            String
  course              Course     @relation(fields: [courseId], references: [id])
  
  completionDate      DateTime
  grade               String?
  qrCodeUrl           String?
  pdfUrl              String
  
  issuedAt            DateTime   @default(now())
  expiresAt           DateTime?
  
  createdAt           DateTime   @default(now())
  updatedAt           DateTime   @updatedAt
  
  @@unique([userId, courseId])
}
typescript
// lib/progress/tracking.ts
import { prisma } from '@/lib/prisma';

export interface ProgressUpdate {
  userId: string;
  lessonId: string;
  courseId: string;
  progress: number;
  lastPosition?: number;
  completed?: boolean;
  timeSpent?: number;
}

export class ProgressTracker {
  static async updateLessonProgress(data: ProgressUpdate) {
    const { userId, lessonId, courseId, progress, lastPosition, completed, timeSpent } = data;
    
    // Start transaction
    return await prisma.$transaction(async (tx) => {
      // Update or create lesson progress
      const lessonProgress = await tx.lessonProgress.upsert({
        where: {
          userId_lessonId: {
            userId,
            lessonId,
          },
        },
        update: {
          progress,
          lastPosition,
          completed: completed ?? false,
          timeSpent: {
            increment: timeSpent ?? 0,
          },
          lastAccessedAt: new Date(),
          completedAt: completed ? new Date() : undefined,
        },
        create: {
          userId,
          lessonId,
          progress,
          lastPosition,
          completed: completed ?? false,
          timeSpent: timeSpent ?? 0,
          lastAccessedAt: new Date(),
          firstAccessedAt: new Date(),
          completedAt: completed ? new Date() : undefined,
        },
      });

      // Get or create course progress
      let courseProgress = await tx.courseProgress.findUnique({
        where: {
          userId_courseId: {
            userId,
            courseId,
          },
        },
        include: {
          lessonProgress: {
            where: { userId },
          },
        },
      });

      if (!courseProgress) {
        // Get total lessons count
        const totalLessons = await tx.lesson.count({
          where: {
            section: {
              courseId,
            },
            status: 'PUBLISHED',
          },
        });

        courseProgress = await tx.courseProgress.create({
          data: {
            userId,
            courseId,
            totalLessons,
            startedAt: new Date(),
            lastAccessedAt: new Date(),
          },
          include: {
            lessonProgress: true,
          },
        });
      }

      // Update course progress stats
      const completedLessons = await tx.lessonProgress.count({
        where: {
          userId,
          courseId,
          completed: true,
        },
      });

      const progressPercentage = courseProgress.totalLessons > 0 
        ? (completedLessons / courseProgress.totalLessons) * 100 
        : 0;

      const isCourseCompleted = progressPercentage >= 100;

      // Update course progress
      courseProgress = await tx.courseProgress.update({
        where: {
          id: courseProgress.id,
        },
        data: {
          completedLessons,
          progressPercentage,
          completed: isCourseCompleted,
          lastAccessedAt: new Date(),
          completedAt: isCourseCompleted ? new Date() : undefined,
          timeSpent: {
            increment: timeSpent ?? 0,
          },
        },
      });

      // Generate certificate if course completed
      if (isCourseCompleted && !courseProgress.completedAt) {
        await this.generateCertificate(userId, courseId);
      }

      return {
        lessonProgress,
        courseProgress,
      };
    });
  }

  static async generateCertificate(userId: string, courseId: string) {
    const course = await prisma.course.findUnique({
      where: { id: courseId },
      include: {
        instructor: true,
      },
    });

    if (!course) throw new Error('Course not found');

    const certificateNumber = `CERT-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
    
    // Generate certificate PDF (implementation depends on PDF generation library)
    const pdfUrl = await this.generateCertificatePDF({
      userId,
      course,
      certificateNumber,
    });

    // Create certificate record
    const certificate = await prisma.certificate.create({
      data: {
        certificateNumber,
        userId,
        courseId,
        completionDate: new Date(),
        pdfUrl,
        qrCodeUrl: await this.generateQRCode(certificateNumber),
        issuedAt: new Date(),
        expiresAt: course.certificateExpiryDays 
          ? new Date(Date.now() + course.certificateExpiryDays * 24 * 60 * 60 * 1000)
          : null,
      },
    });

    return certificate;
  }

  private static async generateCertificatePDF(data: any): Promise<string> {
    // Implement PDF generation using @react-pdf/renderer or similar
    // Return URL to generated PDF
    return `https://cdn.example.com/certificates/${data.certificateNumber}.pdf`;
  }

  private static async generateQRCode(text: string): Promise<string> {
    // Implement QR code generation
    return `https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=${encodeURIComponent(text)}`;
  }
}
§ QUIZZES & ASSESSMENTS
typescript
// components/quiz/QuizComponent.tsx
'use client';

import React, { useState, useEffect } from 'react';
import { Timer, CheckCircle, XCircle } from 'lucide-react';

interface QuizQuestion {
  id: string;
  type: 'multiple_choice' | 'true_false' | 'matching' | 'fill_blank' | 'essay';
  question: string;
  points: number;
  options?: Array<{
    id: string;
    text: string;
    isCorrect: boolean;
  }>;
  correctAnswer?: string;
  matchingPairs?: Array<{
    from: string;
    to: string;
  }>;
  explanation?: string;
}

interface QuizComponentProps {
  quizId: string;
  questions: QuizQuestion[];
  timeLimit?: number; // in minutes
  maxAttempts: number;
  passingScore: number;
  onComplete: (score: number, answers: Record<string, any>) => void;
}

const QuizComponent: React.FC<QuizComponentProps> = ({
  quizId,
  questions,
  timeLimit,
  maxAttempts,
  passingScore,
  onComplete,
}) => {
  const [currentQuestion, setCurrentQuestion] = useState(0);
  const [answers, setAnswers] = useState<Record<string, any>>({});
  const [timeLeft, setTimeLeft] = useState(timeLimit ? timeLimit * 60 : null);
  const [submitted, setSubmitted] = useState(false);
  const [score, setScore] = useState<number | null>(null);
  const [reviewMode, setReviewMode] = useState(false);

  // Timer effect
  useEffect(() => {
    if (!timeLeft || submitted) return;

    const timer = setInterval(() => {
      setTimeLeft(prev => {
        if (prev === null || prev <= 1) {
          clearInterval(timer);
          handleSubmit();
          return 0;
        }
        return prev - 1;
      });
    }, 1000);

    return () => clearInterval(timer);
  }, [timeLeft, submitted]);

  const handleAnswer = (questionId: string, answer: any) => {
    setAnswers(prev => ({
      ...prev,
      [questionId]: answer,
    }));
  };

  const calculateScore = () => {
    let totalPoints = 0;
    let earnedPoints = 0;

    questions.forEach(question => {
      totalPoints += question.points;
      
      const userAnswer = answers[question.id];
      if (!userAnswer) return;

      switch (question.type) {
        case 'multiple_choice':
          const selectedOption = question.options?.find(opt => opt.id === userAnswer);
          if (selectedOption?.isCorrect) {
            earnedPoints += question.points;
          }
          break;
        
        case 'true_false':
          if (userAnswer === question.correctAnswer) {
            earnedPoints += question.points;
          }
          break;
        
        case 'fill_blank':
          if (userAnswer.trim().toLowerCase() === question.correctAnswer?.toLowerCase()) {
            earnedPoints += question.points;
          }
          break;
        
        case 'matching':
          // Calculate partial points for matching
          const correctMatches = question.matchingPairs || [];
          const userMatches = userAnswer as Record<string, string>;
          
          let correctCount = 0;
          Object.entries(userMatches).forEach(([from, to]) => {
            const isCorrect = correctMatches.some(
              match => match.from === from && match.to === to
            );
            if (isCorrect) correctCount++;
          });
          
          if (correctMatches.length > 0) {
            earnedPoints += (correctCount / correctMatches.length) * question.points;
          }
          break;
        
        case 'essay':
          // Essay questions might need manual grading
          // For auto-grading, you could use text analysis
          break;
      }
    });

    return (earnedPoints / totalPoints) * 100;
  };

  const handleSubmit = () => {
    if (submitted) return;

    const finalScore = calculateScore();
    setScore(finalScore);
    setSubmitted(true);
    onComplete(finalScore, answers);
  };

  const formatTime = (seconds: number) => {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;
    return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
  };

  const renderQuestion = (question: QuizQuestion) => {
    const userAnswer = answers[question.id];

    switch (question.type) {
      case 'multiple_choice':
        return (
          <div className="space-y-3">
            {question.options?.map(option => (
              <label
                key={option.id}
                className={`flex items-center p-3 rounded-lg border cursor-pointer ${
                  reviewMode
                    ? option.isCorrect
                      ? 'bg-green-100 border-green-500'
                      : userAnswer === option.id
                      ? 'bg-red-100 border-red-500'
                      : 'border-gray-300'
                    : userAnswer === option.id
                    ? 'bg-blue-50 border-blue-500'
                    : 'border-gray-300 hover:border-blue-400'
                }`}
              >
                <input
                  type="radio"
                  name={`question-${question.id}`}
                  value={option.id}
                  checked={userAnswer === option.id}
                  onChange={() => handleAnswer(question.id, option.id)}
                  disabled={reviewMode}
                  className="mr-3"
                />
                <span>{option.text}</span>
                {reviewMode && option.isCorrect && (
                  <CheckCircle className="ml-auto text-green-500" size={20} />
                )}
                {reviewMode && userAnswer === option.id && !option.isCorrect && (
                  <XCircle className="ml-auto text-red-500" size={20} />
                )}
              </label>
            ))}
          </div>
        );

      case 'true_false':
        return (
          <div className="flex space-x-4">
            {['true', 'false'].map(value => (
              <label
                key={value}
                className={`flex-1 text-center p-4 rounded-lg border cursor-pointer ${
                  reviewMode
                    ? value === question.correctAnswer
                      ? 'bg-green-100 border-green-500'
                      : userAnswer === value
                      ? 'bg-red-100 border-red-500'
                      : 'border-gray-300'
                    : userAnswer === value
                    ? 'bg-blue-50 border-blue-500'
                    : 'border-gray-300 hover:border-blue-400'
                }`}
              >
                <input
                  type="radio"
                  name={`question-${question.id}`}
                  value={value}
                  checked={userAnswer === value}
                  onChange={() => handleAnswer(question.id, value)}
                  disabled={reviewMode}
                  className="mr-2"
                />
                {value.charAt(0).toUpperCase() + value.slice(1)}
              </label>
            ))}
          </div>
        );

      case 'fill_blank':
        return (
          <div>
            <input
              type="text"
              value={userAnswer || ''}
              onChange={(e) => handleAnswer(question.id, e.target.value)}
              disabled={reviewMode}
              className="w-full p-3 border rounded-lg"
              placeholder="Type your answer here..."
            />
            {reviewMode && question.correctAnswer && (
              <div className="mt-2 p-2 bg-gray-100 rounded">
                <span className="font-semibold">Correct answer:</span>{' '}
                {question.correctAnswer}
              </div>
            )}
          </div>
        );

      case 'matching':
        return (
          <div className="space-y-4">
            {question.matchingPairs?.map((pair, index) => (
              <div key={index} className="flex items-center space-x-4">
                <div className="flex-1 p-3 bg-gray-50 rounded-lg">
                  {pair.from}
                </div>
                <span className="text-gray-500">→</span>
                <select
                  value={userAnswer?.[pair.from] || ''}
                  onChange={(e) => handleAnswer(question.id, {
                    ...userAnswer,
                    [pair.from]: e.target.value,
                  })}
                  disabled={reviewMode}
                  className="flex-1 p-3 border rounded-lg"
                >
                  <option value="">Select match</option>
                  {question.matchingPairs?.map((p, idx) => (
                    <option key={idx} value={p.to}>
                      {p.to}
                    </option>
                  ))}
                </select>
              </div>
            ))}
          </div>
        );

      case 'essay':
        return (
          <textarea
            value={userAnswer || ''}
            onChange={(e) => handleAnswer(question.id, e.target.value)}
            disabled={reviewMode}
            className="w-full h-48 p-3 border rounded-lg"
            placeholder="Write your essay answer here..."
          />
        );
    }
  };

  if (reviewMode) {
    return (
      <div className="space-y-6">
        <div className="bg-white p-6 rounded-lg shadow">
          <div className="flex items-center justify-between mb-6">
            <h2 className="text-2xl font-bold">Quiz Review</h2>
            <div className={`text-lg font-semibold ${(score || 0) >= passingScore ? 'text-green-600' : 'text-red-600'}`}>
              Score: {score?.toFixed(1)}% {

## § ADVANCED PATTERNS: LMS LEARNING

### Server Actions con Validazione

```typescript
// app/actions/lms-learning.ts
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


### LMS LEARNING - Utility Helper #769

```typescript
// lib/utils/lms-learning-helper-769.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

  getConfig(): Readonly<LMSLEARNINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### LMS LEARNING - Utility Helper #479

```typescript
// lib/utils/lms-learning-helper-479.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

  getConfig(): Readonly<LMSLEARNINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### LMS LEARNING - Utility Helper #778

```typescript
// lib/utils/lms-learning-helper-778.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

  getConfig(): Readonly<LMSLEARNINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### LMS LEARNING - Utility Helper #606

```typescript
// lib/utils/lms-learning-helper-606.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

  getConfig(): Readonly<LMSLEARNINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### LMS LEARNING - Utility Helper #6

```typescript
// lib/utils/lms-learning-helper-6.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

  getConfig(): Readonly<LMSLEARNINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### LMS LEARNING - Utility Helper #393

```typescript
// lib/utils/lms-learning-helper-393.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

  getConfig(): Readonly<LMSLEARNINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### LMS LEARNING - Utility Helper #458

```typescript
// lib/utils/lms-learning-helper-458.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

  getConfig(): Readonly<LMSLEARNINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### LMS LEARNING - Utility Helper #321

```typescript
// lib/utils/lms-learning-helper-321.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

  getConfig(): Readonly<LMSLEARNINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### LMS LEARNING - Utility Helper #104

```typescript
// lib/utils/lms-learning-helper-104.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

  getConfig(): Readonly<LMSLEARNINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### LMS LEARNING - Utility Helper #65

```typescript
// lib/utils/lms-learning-helper-65.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

  getConfig(): Readonly<LMSLEARNINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### LMS LEARNING - Utility Helper #325

```typescript
// lib/utils/lms-learning-helper-325.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

  getConfig(): Readonly<LMSLEARNINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### LMS LEARNING - Utility Helper #798

```typescript
// lib/utils/lms-learning-helper-798.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

  getConfig(): Readonly<LMSLEARNINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### LMS LEARNING - Utility Helper #916

```typescript
// lib/utils/lms-learning-helper-916.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

  getConfig(): Readonly<LMSLEARNINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### LMS LEARNING - Utility Helper #224

```typescript
// lib/utils/lms-learning-helper-224.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

  getConfig(): Readonly<LMSLEARNINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### LMS LEARNING - Utility Helper #690

```typescript
// lib/utils/lms-learning-helper-690.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

  getConfig(): Readonly<LMSLEARNINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### LMS LEARNING - Utility Helper #863

```typescript
// lib/utils/lms-learning-helper-863.ts
import { z } from "zod";

interface LMSLEARNINGConfig {
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

export class LMSLEARNINGProcessor<TInput, TOutput> {
  private config: LMSLEARNINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<LMSLEARNINGConfig> = {}) {
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

        clearTimeout(timeou