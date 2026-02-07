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