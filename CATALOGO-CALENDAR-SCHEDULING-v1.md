# CATALOGO CALENDAR-SCHEDULING v1

§ §1. CALENDAR LIBRARY COMPARISON

| Library | Bundle | Views | Drag & Drop | Events | Best For |
|---------|--------|-------|-------------|--------|----------|
| FullCalendar | 80kB+ | Month/Week/Day/List | ✅ | ✅ | Full-featured |
| react-big-calendar | 40kB | Month/Week/Day | ✅ | ✅ | Simpler needs |
| @schedule-x | 30kB | Month/Week/Day | ✅ | ✅ | Modern, lightweight |
| react-calendar | 15kB | Month only | ❌ | ❌ | Date picker |
| Custom | Variable | Custom | DIY | DIY | Specific needs |

**Raccomandazione:** react-big-calendar per la maggior parte dei casi. FullCalendar per funzionalità avanzate. @schedule-x per bundle leggero con features moderne.

§ §2. EVENT DATA MODEL

prisma
// prisma/schema.prisma
model Calendar {
  id          String    @id @default(cuid())
  name        String
  description String?
  color       String    @default("#3b82f6")
  isDefault   Boolean   @default(false)
  userId      String
  user        User      @relation(fields: [userId], references: [id])
  events      Event[]
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  
  @@unique([userId, name])
}

model Event {
  id           String        @id @default(cuid())
  title        String
  description  String?
  
  // Timing
  startTime    DateTime
  endTime      DateTime
  allDay       Boolean       @default(false)
  timezone     String        @default("UTC")
  
  // Recurrence
  recurrence   Json?         // RRULE format
  recurrenceId String?       // For recurring instances
  exdate       String[]      // Excluded dates (ISO string array)
  
  // Location
  location     String?
  isOnline     Boolean       @default(false)
  meetingUrl   String?
  
  // Participants
  organizerId  String
  organizer    User          @relation(fields: [organizerId], references: [id])
  attendees    EventAttendee[]
  
  // Categorization
  calendarId   String
  calendar     Calendar      @relation(fields: [calendarId], references: [id])
  color        String?
  
  // Status
  status       EventStatus   @default(CONFIRMED)
  
  // Metadata
  metadata     Json          @default("{}")
  
  createdAt    DateTime      @default(now())
  updatedAt    DateTime      @updatedAt
  deletedAt    DateTime?
  
  @@index([calendarId])
  @@index([organizerId])
  @@index([startTime])
  @@index([endTime])
  @@index([recurrenceId])
}

model EventAttendee {
  id         String        @id @default(cuid())
  eventId    String
  event      Event         @relation(fields: [eventId], references: [id], onDelete: Cascade)
  userId     String
  user       User          @relation(fields: [userId], references: [id], onDelete: Cascade)
  status     AttendeeStatus @default(PENDING)
  role       AttendeeRole   @default(ATTENDEE)
  email      String        // Cache email for invites
  
  createdAt  DateTime      @default(now())
  updatedAt  DateTime      @updatedAt
  
  @@unique([eventId, userId])
  @@index([userId])
}

enum EventStatus {
  TENTATIVE
  CONFIRMED
  CANCELLED
}

enum AttendeeStatus {
  PENDING
  ACCEPTED
  DECLINED
  TENTATIVE
}

enum AttendeeRole {
  ORGANIZER
  REQUIRED
  OPTIONAL
}

// For booking/availability systems
model Availability {
  id           String    @id @default(cuid())
  userId       String
  user         User      @relation(fields: [userId], references: [id])
  name         String    @default("Default")
  timezone     String    @default("UTC")
  isDefault    Boolean   @default(false)
  
  // Weekly schedule
  monday       Json      @default("[]") // Array of { start: "09:00", end: "17:00" }
  tuesday      Json      @default("[]")
  wednesday    Json      @default("[]")
  thursday     Json      @default("[]")
  friday       Json      @default("[]")
  saturday     Json      @default("[]")
  sunday       Json      @default("[]")
  
  // Override for specific dates
  overrides    AvailabilityOverride[]
  
  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt
  
  @@unique([userId, name])
}

model AvailabilityOverride {
  id            String    @id @default(cuid())
  availabilityId String
  availability  Availability @relation(fields: [availabilityId], references: [id], onDelete: Cascade)
  date          DateTime  // Date only (time will be 00:00:00)
  type          OverrideType @default(UNAVAILABLE)
  slots         Json      @default("[]") // Array of { start: "HH:mm", end: "HH:mm" }
  note          String?
  
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  @@unique([availabilityId, date])
}

enum OverrideType {
  UNAVAILABLE
  CUSTOM
}

model Booking {
  id           String    @id @default(cuid())
  eventId      String    @unique
  event        Event     @relation(fields: [eventId], references: [id])
  serviceId    String?
  service      Service?
  
  // Customer info
  customerId   String?
  customer     User?     @relation(fields: [customerId], references: [id])
  customerName String
  customerEmail String
  customerPhone String?
  
  // Booking details
  status       BookingStatus @default(CONFIRMED)
  notes        String?
  
  // Payment
  amount       Decimal?
  currency     String    @default("USD")
  paidAt       DateTime?
  
  // Cancellation
  cancelledAt  DateTime?
  cancellationReason String?
  
  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt
  
  @@index([customerId])
  @@index([status])
}

model Service {
  id            String    @id @default(cuid())
  userId        String
  user          User      @relation(fields: [userId], references: [id])
  name          String
  description   String?
  duration      Int       // In minutes
  price         Decimal?
  currency      String    @default("USD")
  
  // Availability
  bufferBefore  Int       @default(0) // Minutes before
  bufferAfter   Int       @default(0) // Minutes after
  
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  @@unique([userId, name])
}

enum BookingStatus {
  PENDING
  CONFIRMED
  CANCELLED
  NO_SHOW
  COMPLETED
}

§ §3. CALENDAR COMPONENT

§ 3.1 CALENDAR VIEWS SETUP

typescript
// lib/calendar/config.ts
import { Calendar as BigCalendar, View, ViewNames } from 'react-big-calendar';

export type CalendarView = 'month' | 'week' | 'day' | 'agenda';

export interface CalendarEvent {
  id: string;
  title: string;
  description?: string;
  start: Date;
  end: Date;
  allDay?: boolean;
  color?: string;
  calendarId: string;
  calendarName?: string;
  status: 'TENTATIVE' | 'CONFIRMED' | 'CANCELLED';
  location?: string;
  isOnline?: boolean;
  meetingUrl?: string;
  attendees?: Array<{
    id: string;
    name: string;
    email: string;
    status: 'PENDING' | 'ACCEPTED' | 'DECLINED' | 'TENTATIVE';
  }>;
  organizer?: {
    id: string;
    name: string;
    email: string;
  };
  resource?: any; // For custom data
}

export const calendarConfig = {
  views: ['month', 'week', 'day', 'agenda'] as CalendarView[],
  defaultView: 'month' as CalendarView,
  step: 30, // minutes
  timeslots: 2,
  min: new Date(0, 0, 0, 8, 0, 0), // 8:00 AM
  max: new Date(0, 0, 0, 22, 0, 0), // 10:00 PM
  formats: {
    dayFormat: 'EEE dd',
    dayRangeHeaderFormat: ({ start, end }: { start: Date; end: Date }) => {
      return `${start.toLocaleDateString()} — ${end.toLocaleDateString()}`;
    },
    eventTimeRangeFormat: () => '',
    timeGutterFormat: (date: Date) => {
      return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    },
  },
  messages: {
    today: 'Today',
    previous: 'Back',
    next: 'Next',
    month: 'Month',
    week: 'Week',
    day: 'Day',
    agenda: 'Agenda',
    date: 'Date',
    time: 'Time',
    event: 'Event',
    allDay: 'All Day',
    noEventsInRange: 'No events in this range',
    showMore: (count: number) => `+${count} more`,
  },
} as const;

typescript
// components/calendar/CalendarContainer.tsx
'use client';

import { useState, useMemo, useCallback } from 'react';
import { Calendar as BigCalendar, momentLocalizer, View, SlotInfo } from 'react-big-calendar';
import withDragAndDrop from 'react-big-calendar/lib/addons/dragAndDrop';
import moment from 'moment';
import 'react-big-calendar/lib/css/react-big-calendar.css';
import 'react-big-calendar/lib/addons/dragAndDrop/styles.css';

import { CalendarEvent, CalendarView, calendarConfig } from '@/lib/calendar/config';
import { CalendarToolbar } from './CalendarToolbar';
import { EventDetails } from './EventDetails';
import { EventForm } from './EventForm';
import { cn } from '@/lib/utils';
import { Calendar } from 'lucide-react';

const localizer = momentLocalizer(moment);
const DnDCalendar = withDragAndDrop(BigCalendar);

interface CalendarContainerProps {
  events: CalendarEvent[];
  calendars?: Array<{
    id: string;
    name: string;
    color: string;
    isDefault: boolean;
  }>;
  onEventCreate?: (event: Omit<CalendarEvent, 'id'>) => Promise<void>;
  onEventUpdate?: (eventId: string, updates: Partial<CalendarEvent>) => Promise<void>;
  onEventDelete?: (eventId: string) => Promise<void>;
  onEventDrop?: (eventId: string, start: Date, end: Date) => Promise<void>;
  onEventResize?: (eventId: string, start: Date, end: Date) => Promise<void>;
  className?: string;
  loading?: boolean;
  readOnly?: boolean;
  defaultView?: CalendarView;
}

export function CalendarContainer({
  events,
  calendars = [],
  onEventCreate,
  onEventUpdate,
  onEventDelete,
  onEventDrop,
  onEventResize,
  className,
  loading = false,
  readOnly = false,
  defaultView = calendarConfig.defaultView,
}: CalendarContainerProps) {
  const [view, setView] = useState<CalendarView>(defaultView);
  const [date, setDate] = useState<Date>(new Date());
  const [selectedEvent, setSelectedEvent] = useState<CalendarEvent | null>(null);
  const [showEventForm, setShowEventForm] = useState(false);
  const [formEvent, setFormEvent] = useState<Partial<CalendarEvent> | null>(null);
  const [selectedCalendarIds, setSelectedCalendarIds] = useState<string[]>(
    calendars.filter(c => c.isDefault).map(c => c.id)
  );

  // Filter events by selected calendars
  const filteredEvents = useMemo(() => {
    if (selectedCalendarIds.length === 0 || selectedCalendarIds.length === calendars.length) {
      return events;
    }
    return events.filter(event => selectedCalendarIds.includes(event.calendarId));
  }, [events, selectedCalendarIds, calendars.length]);

  // Calendar event style getter
  const eventStyleGetter = useCallback((event: CalendarEvent) => {
    const color = event.color || '#3b82f6';
    const backgroundColor = event.status === 'CANCELLED' ? '#f3f4f6' : `${color}20`;
    const borderColor = event.status === 'CANCELLED' ? '#9ca3af' : color;
    const textColor = event.status === 'CANCELLED' ? '#6b7280' : '#111827';
    
    return {
      style: {
        backgroundColor,
        border: `1px solid ${borderColor}`,
        borderLeft: `4px solid ${borderColor}`,
        color: textColor,
        borderRadius: '4px',
        opacity: event.status === 'CANCELLED' ? 0.7 : 1,
        padding: '2px 8px',
        fontSize: '13px',
      },
    };
  }, []);

  // Handle view change
  const handleViewChange = (newView: CalendarView) => {
    setView(newView);
  };

  // Handle date change
  const handleNavigate = (newDate: Date) => {
    setDate(newDate);
  };

  // Handle event selection
  const handleSelectEvent = (event: CalendarEvent) => {
    if (readOnly) return;
    setSelectedEvent(event);
  };

  // Handle slot selection (create new event)
  const handleSelectSlot = (slotInfo: SlotInfo) => {
    if (readOnly || !onEventCreate) return;
    
    const defaultCalendarId = calendars.find(c => c.isDefault)?.id || calendars[0]?.id;
    
    setFormEvent({
      title: '',
      start: slotInfo.start,
      end: slotInfo.end,
      allDay: slotInfo.slots.length === 1 && slotInfo.action === 'select',
      calendarId: defaultCalendarId,
      status: 'CONFIRMED' as const,
    });
    setShowEventForm(true);
  };

  // Handle event drop (drag and drop)
  const handleEventDrop = useCallback(async ({ event, start, end }: any) => {
    if (readOnly || !onEventDrop) return;
    
    try {
      await onEventDrop(event.id, start, end);
    } catch (error) {
      console.error('Failed to move event:', error);
    }
  }, [onEventDrop, readOnly]);

  // Handle event resize
  const handleEventResize = useCallback(async ({ event, start, end }: any) => {
    if (readOnly || !onEventResize) return;
    
    try {
      await onEventResize(event.id, start, end);
    } catch (error) {
      console.error('Failed to resize event:', error);
    }
  }, [onEventResize, readOnly]);

  // Handle event form submit
  const handleEventSubmit = async (eventData: Partial<CalendarEvent>) => {
    try {
      if (formEvent?.id && onEventUpdate) {
        await onEventUpdate(formEvent.id, eventData);
      } else if (onEventCreate) {
        await onEventCreate(eventData as Omit<CalendarEvent, 'id'>);
      }
      setShowEventForm(false);
      setFormEvent(null);
    } catch (error) {
      console.error('Failed to save event:', error);
    }
  };

  // Handle event edit
  const handleEditEvent = (event: CalendarEvent) => {
    setFormEvent(event);
    setShowEventForm(true);
    setSelectedEvent(null);
  };

  // Handle event delete
  const handleDeleteEvent = async (eventId: string) => {
    if (!onEventDelete) return;
    
    if (confirm('Are you sure you want to delete this event?')) {
      try {
        await onEventDelete(eventId);
        setSelectedEvent(null);
      } catch (error) {
        console.error('Failed to delete event:', error);
      }
    }
  };

  // Toggle calendar visibility
  const toggleCalendar = (calendarId: string) => {
    setSelectedCalendarIds(prev => {
      if (prev.includes(calendarId)) {
        return prev.filter(id => id !== calendarId);
      } else {
        return [...prev, calendarId];
      }
    });
  };

  if (loading) {
    return (
      <div className={cn('flex items-center justify-center min-h-[600px] border rounded-lg', className)}>
        <div className="flex flex-col items-center gap-3">
          <Calendar className="h-8 w-8 animate-pulse text-muted-foreground" />
          <p className="text-sm text-muted-foreground">Loading calendar...</p>
        </div>
      </div>
    );
  }

  return (
    <div className={cn('flex flex-col h-full', className)}>
      {/* Toolbar */}
      <CalendarToolbar
        view={view}
        date={date}
        onViewChange={handleViewChange}
        onNavigate={handleNavigate}
        calendars={calendars}
        selectedCalendarIds={selectedCalendarIds}
        onToggleCalendar={toggleCalendar}
        onNewEvent={() => {
          setFormEvent({
            title: '',
            start: new Date(),
            end: new Date(Date.now() + 60 * 60 * 1000), // 1 hour later
            status: 'CONFIRMED',
            calendarId: calendars.find(c => c.isDefault)?.id || calendars[0]?.id,
          });
          setShowEventForm(true);
        }}
      />

      {/* Calendar */}
      <div className="flex-1 relative min-h-[600px]">
        <DnDCalendar
          localizer={localizer}
          events={filteredEvents}
          view={view}
          date={date}
          onView={handleViewChange}
          onNavigate={handleNavigate}
          onSelectEvent={readOnly ? undefined : handleSelectEvent}
          onSelectSlot={readOnly ? undefined : handleSelectSlot}
          onEventDrop={readOnly ? undefined : handleEventDrop}
          onEventResize={readOnly ? undefined : handleEventResize}
          eventPropGetter={eventStyleGetter}
          selectable={!readOnly && !!onEventCreate}
          resizable={!readOnly && !!onEventResize}
          popup={true}
          toolbar={false}
          step={calendarConfig.step}
          timeslots={calendarConfig.timeslots}
          min={calendarConfig.min}
          max={calendarConfig.max}
          formats={calendarConfig.formats}
          messages={calendarConfig.messages}
          components={{
            toolbar: () => null, // We use our own toolbar
          }}
          style={{ height: '100%' }}
        />
      </div>

      {/* Event Details Modal */}
      {selectedEvent && (
        <EventDetails
          event={selectedEvent}
          onClose={() => setSelectedEvent(null)}
          onEdit={handleEditEvent}
          onDelete={handleDeleteEvent}
          readOnly={readOnly}
        />
      )}

      {/* Event Form Modal */}
      {showEventForm && formEvent && (
        <EventForm
          event={formEvent}
          calendars={calendars}
          onSubmit={handleEventSubmit}
          onCancel={() => {
            setShowEventForm(false);
            setFormEvent(null);
          }}
        />
      )}
    </div>
  );
}

§ 3.2 CALENDAR TOOLBAR

typescript
// components/calendar/CalendarToolbar.tsx
'use client';

import { Button } from '@/components/ui/button';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { CalendarView } from '@/lib/calendar/config';
import { cn } from '@/lib/utils';
import { 
  ChevronLeft, 
  ChevronRight, 
  CalendarDays, 
  Grid2x2, 
  Calendar, 
  Plus,
  Eye,
  EyeOff,
} from 'lucide-react';
import { format } from 'date-fns';

interface CalendarToolbarProps {
  view: CalendarView;
  date: Date;
  calendars: Array<{
    id: string;
    name: string;
    color: string;
    isDefault: boolean;
  }>;
  selectedCalendarIds: string[];
  onViewChange: (view: CalendarView) => void;
  onNavigate: (date: Date) => void;
  onToggleCalendar: (calendarId: string) => void;
  onNewEvent: () => void;
}

export function CalendarToolbar({
  view,
  date,
  calendars,
  selectedCalendarIds,
  onViewChange,
  onNavigate,
  onToggleCalendar,
  onNewEvent,
}: CalendarToolbarProps) {
  const viewLabels: Record<CalendarView, string> = {
    month: 'Month',
    week: 'Week',
    day: 'Day',
    agenda: 'Agenda',
  };

  const viewIcons: Record<CalendarView, React.ReactNode> = {
    month: <Grid2x2 className="h-4 w-4" />,
    week: <CalendarDays className="h-4 w-4" />,
    day: <Calendar className="h-4 w-4" />,
    agenda: <ChevronRight className="h-4 w-4" />,
  };

  const navigate = (direction: 'prev' | 'next' | 'today') => {
    const newDate = new Date(date);
    
    switch (direction) {
      case 'prev':
        if (view === 'month') {
          newDate.setMonth(newDate.getMonth() - 1);
        } else if (view === 'week') {
          newDate.setDate(newDate.getDate() - 7);
        } else {
          newDate.setDate(newDate.getDate() - 1);
        }
        break;
      case 'next':
        if (view === 'month') {
          newDate.setMonth(newDate.getMonth() + 1);
        } else if (view === 'week') {
          newDate.setDate(newDate.getDate() + 7);
        } else {
          newDate.setDate(newDate.getDate() + 1);
        }
        break;
      case 'today':
        newDate.setTime(Date.now());
        break;
    }
    
    onNavigate(newDate);
  };

  const formatDateRange = () => {
    const start = new Date(date);
    const end = new Date(date);
    
    switch (view) {
      case 'month':
        start.setDate(1);
        end.setMonth(end.getMonth() + 1);
        end.setDate(0);
        return format(start, 'MMM yyyy');
      
      case 'week':
        const dayOfWeek = start.getDay();
        const diff = start.getDate() - dayOfWeek + (dayOfWeek === 0 ? -6 : 1);
        start.setDate(diff);
        end.setDate(start.getDate() + 6);
        return `${format(start, 'MMM d')} - ${format(end, 'MMM d, yyyy')}`;
      
      case 'day':
        return format(start, 'EEEE, MMMM d, yyyy');
      
      case 'agenda':
        return format(start, 'MMM yyyy');
    }
  };

  return (
    <div className="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4 p-4 border-b">
      {/* Left section */}
      <div className="flex items-center gap-2">
        <Button
          variant="outline"
          size="sm"
          onClick={() => navigate('today')}
          className="gap-2"
        >
          Today
        </Button>
        
        <div className="flex items-center">
          <Button
            variant="ghost"
            size="icon"
            onClick={() => navigate('prev')}
            className="h-8 w-8"
          >
            <ChevronLeft className="h-4 w-4" />
          </Button>
          
          <Button
            variant="ghost"
            size="icon"
            onClick={() => navigate('next')}
            className="h-8 w-8"
          >
            <ChevronRight className="h-4 w-4" />
          </Button>
        </div>
        
        <div className="text-lg font-semibold ml-2">
          {formatDateRange()}
        </div>
      </div>

      {/* Center section - Calendar toggles */}
      {calendars.length > 0 && (
        <div className="flex flex-wrap items-center gap-2">
          {calendars.map(calendar => {
            const isSelected = selectedCalendarIds.includes(calendar.id);
            return (
              <Button
                key={calendar.id}
                variant="ghost"
                size="sm"
                onClick={() => onToggleCalendar(calendar.id)}
                className={cn(
                  'gap-2 border',
                  isSelected 
                    ? 'bg-opacity-20' 
                    : 'bg-opacity-10 opacity-60'
                )}
                style={{
                  backgroundColor: isSelected ? `${calendar.color}20` : 'transparent',
                  borderColor: calendar.color,
                }}
              >
                {isSelected ? (
                  <Eye className="h-3 w-3" />
                ) : (
                  <EyeOff className="h-3 w-3" />
                )}
                <span className="truncate max-w-[100px]">{calendar.name}</span>
              </Button>
            );
          })}
        </div>
      )}

      {/* Right section */}
      <div className="flex items-center gap-2">
        <Select value={view} onValueChange={(value) => onViewChange(value as CalendarView)}>
          <SelectTrigger className="w-32">
            <SelectValue>
              <div className="flex items-center gap-2">
                {viewIcons[view]}
                <span>{viewLabels[view]}</span>
              </div>
            </SelectValue>
          </SelectTrigger>
          <SelectContent>
            {Object.entries(viewLabels).map(([viewKey, label]) => (
              <SelectItem key={viewKey} value={viewKey}>
                <div className="flex items-center gap-2">
                  {viewIcons[viewKey as CalendarView]}
                  <span>{label}</span>
                </div>
              </SelectItem>
            ))}
          </SelectContent>
        </Select>
        
        <Button
          variant="default"
          size="sm"
          onClick={onNewEvent}
          className="gap-2"
        >
          <Plus className="h-4 w-4" />
          New Event
        </Button>
      </div>
    </div>
  );
}

§ 3.3 EVENT DETAILS COMPONENT

typescript
// components/calendar/EventDetails.tsx
'use client';

import { CalendarEvent } from '@/lib/calendar/config';
import { Button } from '@/components/ui/button';
import { Dialog, DialogContent, DialogHeader, DialogTitle } from '@/components/ui/dialog';
import { Badge } from '@/components/ui/badge';
import { cn } from '@/lib/utils';
import { 
  X, 
  Edit2, 
  Trash2, 
  Calendar as CalendarIcon,
  Clock, 
  MapPin, 
  Video,
  Users,
  CheckCircle,
  XCircle,
  Clock4,
  User,
} from 'lucide-react';
import { format } from 'date-fns';

interface EventDetailsProps {
  event: CalendarEvent;
  onClose: () => void;
  onEdit: (event: CalendarEvent) => void;
  onDelete: (eventId: string) => void;
  readOnly?: boolean;
}

export function EventDetails({
  event,
  onClose,
  onEdit,
  onDelete,
  readOnly = false,
}: EventDetailsProps) {
  const statusColors = {
    TENTATIVE: 'bg-yellow-100 text-yellow-800 border-yellow-200',
    CONFIRMED: 'bg-green-100 text-green-800 border-green-200',
    CANCELLED: 'bg-gray-100 text-gray-800 border-gray-200',
  };

  const formatDateTime = (date: Date) => {
    if (event.allDay) {
      return format(date, 'EEEE, MMMM d, yyyy');
    }
    return format(date, 'EEEE, MMMM d, yyyy • h:mm a');
  };

  const getAttendeeStatusIcon = (status: string) => {
    switch (status) {
      case 'ACCEPTED':
        return <CheckCircle className="h-4 w-4 text-green-500" />;
      case 'DECLINED':
        return <XCircle className="h-4 w-4 text-red-500" />;
      case 'TENTATIVE':
        return <Clock4 className="h-4 w-4 text-yellow-500" />;
      default:
        return <Clock className="h-4 w-4 text-gray-500" />;
    }
  };

  return (
    <Dialog open={true} onOpenChange={onClose}>
      <DialogContent className="max-w-2xl">
        <DialogHeader>
          <div className="flex items-start justify-between">
            <DialogTitle className="text-2xl flex items-center gap-2">
              <div 
                className="h-3 w-3 rounded-full"
                style={{ backgroundColor: event.color || '#3b82f6' }}
              />
              {event.title}
            </DialogTitle>
            <Button variant="ghost" size="icon" onClick={onClose}>
              <X className="h-4 w-4" />
            </Button>
          </div>
        </DialogHeader>

        <div className="space-y-6">
          {/* Status */}
          <Badge variant="outline" className={cn(statusColors[event.status])}>
            {event.status}
          </Badge>

          {/* Description */}
          {event.description && (
            <div>
              <h4 className="text-sm font-medium text-gray-500 mb-2">Description</h4>
              <p className="text-gray-700 whitespace-pre-wrap">{event.description}</p>
            </div>
          )}

          {/* Time */}
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div>
              <h4 className="text-sm font-medium text-gray-500 mb-2 flex items-center gap-2">
                <Clock className="h-4 w-4" />
                Time
              </h4>
              <div className="space-y-1">
                <p className="text-gray-700">
                  {formatDateTime(event.start)}
                </p>
                {!event.allDay && (
                  <p className="text-gray-700">
                    to {format(event.end, 'h:mm a')}
                  </p>
                )}
                {event.allDay && (
                  <p className="text-sm text-gray-500">All day</p>
                )}
              </div>
            </div>

            {/* Calendar */}
            <div>
              <h4 className="text-sm font-medium text-gray-500 mb-2 flex items-center gap-2">
                <CalendarIcon className="h-4 w-4" />
                Calendar
              </h4>
              <div className="flex items-center gap-2">
                <div 
                  className="h-3 w-3 rounded-full"
                  style={{ backgroundColor: event.color || '#3b82f6' }}
                />
                <span className="text-gray-700">{event.calendarName || 'Calendar'}</span>
              </div>
            </div>
          </div>

          {/* Location */}
          {(event.location || event.isOnline) && (
            <div>
              <h4 className="text-sm font-medium text-gray-500 mb-2 flex items-center gap-2">
                {event.isOnline ? (
                  <Video className="h-4 w-4" />
                ) : (
                  <MapPin className="h-4 w-4" />
                )}
                {event.isOnline ? 'Online Meeting' : 'Location'}
              </h4>
              {event.isOnline && event.meetingUrl ? (
                <a 
                  href={event.meetingUrl}
                  target="_blank"
                  rel="noopener noreferrer"
                  className="text-blue-600 hover:text-blue-800 underline"
                >
                  {event.meetingUrl}
                </a>
              ) : (
                <p className="text-gray-700">{event.location}</p>
              )}
            </div>
          )}

          {/* Organizer */}
          {event.organizer && (
            <div>
              <h4 className="text-sm font-medium text-gray-500 mb-2 flex items-center gap-2">
                <User className="h-4 w-4" />
                Organizer
              </h4>
              <p className="text-gray-700">{event.organizer.name}</p>
              <p className="text-sm text-gray-500">{event.organizer.email}</p>
            </div>
          )}

          {/* Attendees */}
          {event.attendees && event.attendees.length > 0 && (
            <div>
              <h4 className="text-sm font-medium text-gray-500 mb-2 flex items-center gap-2">
                <Users className="h-4 w-4" />
                Attendees ({event.attendees.length})
              </h4>
              <div className="space-y-2">
                {event.attendees.map(attendee => (
                  <div key={attendee.id} className="flex items-center justify-between">
                    <div className="flex items-center gap-3">
                      <div className="flex items-center justify-center h-8 w-8 rounded-full bg-gray-100">
                        <span className="text-sm font-medium">
                          {attendee.name.charAt(0).toUpperCase()}
                        </span>
                      </div>
                      <div>
                        <p className="text-sm font-medium text-gray-700">{attendee.name}</p>
                        <p className="text-xs text-gray-500">{attendee.email}</p>
                      </div>
                    </div>
                    <div className="flex items-center gap-2">
                      {getAttendeeStatusIcon(attendee.status)}
                      <span className="text-xs text-gray-500">{attendee.status}</span>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          )}
        </div>

        {/* Actions */}
        {!readOnly && (
          <div className="flex justify-end gap-2 pt-6 border-t">
            <Button
              variant="destructive"
              size="sm"
              onClick={() => onDelete(event.id)}
              className="gap-2"
            >
              <Trash2 className="h-4 w-4" />
              Delete
            </Button>
            <Button
              variant="default"
              size="sm"
              onClick={() => onEdit(event)}
              className="gap-2"
            >
              <Edit2 className="h-4 w-4" />
              Edit
            </Button>
          </div>
        )}
      </DialogContent>
    </Dialog>
  );
}

§ §4. EVENT CRUD

§ 4.1 EVENT SERVICE

typescript
// lib/services/event-service.ts
import { prisma } from '@/lib/prisma';
import { addMinutes, subMinutes, isWithinInterval, eachDayOfInterval } from 'date-fns';
import { zonedTimeToUtc, utcToZonedTime } from 'date-fns-tz';
import { RRule, RRuleSet, rrulestr } from 'rrule';

export interface CreateEventInput {
  title: string;
  description?: string;
  startTime: Date;
  endTime: Date;
  allDay?: boolean;
  timezone?: string;
  calendarId: string;
  organizerId: string;
  location?: string;
  isOnline?: boolean;
  meetingUrl?: string;
  color?: string;
  status?: 'TENTATIVE' | 'CONFIRMED' | 'CANCELLED';
  recurrence?: {
    freq: 'DAILY' | 'WEEKLY' | 'MONTHLY' | 'YEARLY';
    interval?: number;
    count?: number;
    until?: Date;
    byweekday?: number[];
    bymonthday?: number[];
    timezone: string;
  };
  attendees?: Array<{
    userId?: string;
    email: string;
    name?: string;
    role?: 'REQUIRED' | 'OPTIONAL';
  }>;
}

export interface UpdateEventInput extends Partial<CreateEventInput> {
  updateType?: 'single' | 'future' | 'all';
}

export class EventService {
  async createEvent(input: CreateEventInput) {
    const {
      recurrence,
      attendees = [],
      timezone = 'UTC',
      ...eventData
    } = input;

    return await prisma.$transaction(async (tx) => {
      // Convert times to UTC
      const startTime = zonedTimeToUtc(eventData.startTime, timezone);
      const endTime = zonedTimeToUtc(eventData.endTime, timezone);

      // Check for conflicts
      const conflicts = await this.checkConflicts(
        eventData.calendarId,
        startTime,
        endTime,
        eventData.organizerId
      );

      if (conflicts.length > 0) {
        throw new Error('Event conflicts with existing events');
      }

      // Handle recurrence
      let recurrenceRule = null;
      if (recurrence) {
        const rule = new RRule({
          freq: RRule[recurrence.freq],
          interval: recurrence.interval || 1,
          count: recurrence.count,
          until: recurrence.until,
          byweekday: recurrence.byweekday,
          bymonthday: recurrence.bymonthday,
          dtstart: startTime,
        });
        recurrenceRule = rule.toString();
      }

      // Create the event
      const event = await tx.event.create({
        data: {
          ...eventData,
          startTime,
          endTime,
          timezone,
          recurrence: recurrenceRule,
          attendees: {
            create: [
              // Add organizer as attendee
              {
                userId: eventData.organizerId,
                email: '', // Will be populated from user
                status: 'ACCEPTED',
                role: 'ORGANIZER',
              },
              // Add other attendees
              ...attendees.map(attendee => ({
                userId: attendee.userId,
                email: attendee.email,
                name: attendee.name,
                status: 'PENDING',
                role: attendee.role || 'REQUIRED',
              })),
            ],
          },
        },
        include: {
          calendar: true,
          attendees: {
            include: {
              user: true,
            },
          },
        },
      });

      // Send invitations
      await this.sendInvitations(event, attendees);

      return event;
    });
  }

  async updateEvent(eventId: string, input: UpdateEventInput) {
    const { updateType = 'single', ...updateData } = input;

    return await prisma.$transaction(async (tx) => {
      const existingEvent = await tx.event.findUnique({
        where: { id: eventId },
        include: { attendees: true },
      });

      if (!existingEvent) {
        throw new Error('Event not found');
      }

      // Handle recurrence updates
      if (existingEvent.recurrence && updateType !== 'single') {
        return await this.updateRecurringEvent(
          existingEvent,
          updateData,
          updateType,
          tx
        );
      }

      // Regular event update
      const event = await tx.event.update({
        where: { id: eventId },
        data: {
          ...this.sanitizeEventData(updateData),
          // Update timezone if times are provided
          ...(updateData.startTime && updateData.timezone && {
            startTime: zonedTimeToUtc(updateData.startTime, updateData.timezone),
          }),
          ...(updateData.endTime && updateData.timezone && {
            endTime: zonedTimeToUtc(updateData.endTime, updateData.timezone),
          }),
        },
        include: {
          calendar: true,
          attendees: {
            include: {
              user: true,
            },
          },
        },
      });

      // Handle attendee updates
      if (updateData.attendees) {
        await this.updateAttendees(eventId, updateData.attendees, tx);
      }

      return event;
    });
  }

  async deleteEvent(eventId: string, deleteType: 'single' | 'future' | 'all' = 'single') {
    const event = await prisma.event.findUnique({
      where: { id: eventId },
    });

    if (!event) {
      throw new Error('Event not found');
    }

    if (event.recurrence && deleteType !== 'single') {
      return await this.deleteRecurringEvent(event, deleteType);
    }

    return await prisma.event.update({
      where: { id: eventId },
      data: { deletedAt: new Date() },
    });
  }

  async getEvents(
    calendarId: string,
    start: Date,
    end: Date,
    options?: {
      includeCancelled?: boolean;
      includeTentative?: boolean;
    }
  ) {
    const where: any = {
      calendarId,
      deletedAt: null,
      startTime: { gte: start },
      endTime: { lte: end },
    };

    if (!options?.includeCancelled) {
      where.status = { not: 'CANCELLED' };
    }

    if (!options?.includeTentative) {
      where.OR = [
        { status: { not: 'TENTATIVE' } },
        { status: 'TENTATIVE' },
      ];
    }

    // Get regular events
    const regularEvents = await prisma.event.findMany({
      where,
      include: {
        calendar: true,
        attendees: {
          include: {
            user: true,
          },
        },
        organizer: true,
      },
      orderBy: { startTime: 'asc' },
    });

    // Get recurring events and expand them
    const recurringEvents = await prisma.event.findMany({
      where: {
        calendarId,
        deletedAt: null,
        recurrence: { not: null },
        recurrenceId: null, // Only master events
        status: { not: 'CANCELLED' },
      },
      include: {
        calendar: true,
        attendees: {
          include: {
            user: true,
          },
        },
        organizer: true,
      },
    });

    // Expand recurring events
    const expandedRecurringEvents = [];
    for (const event of recurringEvents) {
      try {
        const instances = await this.expandRecurringEvent(event, start, end);
        expandedRecurringEvents.push(...instances);
      } catch (error) {
        console.error('Error expanding recurring event:', error);
      }
    }

    return [...regularEvents, ...expandedRecurringEvents];
  }

  async checkConflicts(
    calendarId: string,
    startTime: Date,
    endTime: Date,
    excludeEventId?: string
  ) {
    const where: any = {
      calendarId,
      deletedAt: null,
      status: { not: 'CANCELLED' },
      OR: [
        {
          startTime: { lt: endTime },
          endTime: { gt: startTime },
        },
      ],
    };

    if (excludeEventId) {
      where.NOT = { id: excludeEventId };
    }

    return await prisma.event.findMany({
      where,
      include: {
        calendar: true,
      },
    });
  }

  async inviteAttendee(eventId: string, email: string, name?: string) {
    const event = await prisma.event.findUnique({
      where: { id: eventId },
      include: { attendees: true },
    });

    if (!event) {
      throw new Error('Event not found');
    }

    // Check if attendee already exists
    const existingAttendee = event.attendees.find(a => a.email === email);
    if (existingAttendee) {
      throw new Error('Attendee already invited');
    }

    const attendee = await prisma.eventAttendee.create({
      data: {
        eventId,
        email,
        name,
        status: 'PENDING',
        role: 'REQUIRED',
      },
    });

    // Send invitation email
    await this.sendInvitationEmail(event, email, name);

    return attendee;
  }

  async respondToInvite(eventId: string, userId: string, status: 'ACCEPTED' | 'DECLINED' | 'TENTATIVE') {
    const attendee = await prisma.eventAttendee.findFirst({
      where: {
        eventId,
        userId,
      },
    });

    if (!attendee) {
      throw new Error('Invitation not found');
    }

    return await prisma.eventAttendee.update({
      where: { id: attendee.id },
      data: { status },
    });
  }

  private async expandRecurringEvent(
    event: any,
    rangeStart: Date,
    rangeEnd: Date
  ) {
    if (!event.recurrence) return [event];

    const rruleSet = new RRuleSet();
    const rule = rrulestr(event.recurrence.toString(), { forceset: true });
    rruleSet.rrule(rule);

    // Add exclusions
    if (event.exdate && event.exdate.length > 0) {
      for (const exdate of event.exdate) {
        rruleSet.exdate(new Date(exdate));
      }
    }

    // Generate occurrences
    const occurrences = rruleSet.between(rangeStart, rangeEnd);

    return occurrences.map(occurrence => ({
      ...event,
      id: `${event.id}_${occurrence.getTime()}`,
      startTime: occurrence,
      endTime: addMinutes(occurrence, this.getDuration(event)),
      recurrenceId: event.id,
    }));
  }

  private getDuration(event: any): number {
    return (event.endTime.getTime() - event.startTime.getTime()) / (1000 * 60);
  }

  private async updateRecurringEvent(
    masterEvent: any,
    updateData: any,
    updateType: 'future' | 'all',
    tx: any
  ) {
    // Implementation for updating recurring events
    // This is a simplified version - full implementation would handle:
    // 1. Creating new master event for 'future' updates
    // 2. Updating all occurrences for 'all' updates
    // 3. Handling exceptions and new recurrence rules

    throw new Error('Recurring event updates not fully implemented');
  }

  private async deleteRecurringEvent(event: any, deleteType: 'future' | 'all') {
    // Implementation for deleting recurring events
    // Similar complexity to updateRecurringEvent

    throw new Error('Recurring event deletion not fully implemented');
  }

  private async updateAttendees(
    eventId: string,
    attendees: Array<{
      userId?: string;
      email: string;
      name?: string;
      role?: 'REQUIRED' | 'OPTIONAL';
    }>,
    tx: any
  ) {
    const existingAttendees = await tx.eventAttendee.findMany({
      where: { eventId },
    });

    // Remove attendees not in new list
    const emailsToKeep = attendees.map(a => a.email);
    await tx.eventAttendee.deleteMany({
      where: {
        eventId,
        NOT: {
          email: { in: emailsToKeep },
        },
      },
    });

    // Add or update attendees
    for (const attendee of attendees) {
      const existing = existingAttendees.find((a: any) => a.email === attendee.email);
      
      if (existing) {
        await tx.eventAttendee.update({
          where: { id: existing.id },
          data: {
            name: attendee.name,
            role: attendee.role || 'REQUIRED',
          },
        });
      } else {
        await tx.eventAttendee.create({
          data: {
            eventId,
            userId: attendee.userId,
            email: attendee.email,
            name: attendee.name,
            status: 'PENDING',
            role: attendee.role || 'REQUIRED',
          },
        });
      }
    }
  }

  private sanitizeEventData(data: Partial<CreateEventInput>) {
    const sanitized: any = {};
    
    if (data.title !== undefined) sanitized.title = data.title;
    if (data.description !== undefined) sanitized.description = data.description;
    if (data.allDay !== undefined) sanitized.allDay = data.allDay;
    if (data.location !== undefined) sanitized.location = data.location;
    if (data.isOnline !== undefined) sanitized.isOnline = data.isOnline;
    if (data.meetingUrl !== undefined) sanitized.meetingUrl = data.meetingUrl;
    if (data.color !== undefined) sanitized.color = data.color;
    if (data.status !== undefined) sanitized.status = data.status;
    
    return sanitized;
  }

  private async sendInvitations(event: any, attendees: Array<any>) {
    // Implementation for sending invitation emails
    // This would integrate with your email service (Resend, SendGrid, etc.)
    console.log('Sending invitations for event:', event.id);
  }

  private async sendInvitationEmail(event: any, email: string, name?: string) {
    // Implementation for sending individual invitation email
    console.log('Sending invitation to:', email);
  }
}

§ 4.2 EVENT FORM

typescript
// components/calendar/EventForm.tsx
'use client';

import { useState, useEffect } from 'react';
import { CalendarEvent } from '@/lib/calendar/config';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { Label } from '@/components/ui/label';
import { Switch } from '@/components/ui/switch';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter } from '@/components/ui/dialog';
import { DateTimePicker } from '@/components/calendar/DateTimePicker';
import { RecurrenceSelector } from '@/components/calendar/RecurrenceSelector';
import { AttendeeSelector } from '@/components/calendar/AttendeeSelector';
import { cn } from '@/lib/utils';
import { X, Globe, MapPin, Video } from 'lucide-react';
import { format } from 'date-fns';

interface EventFormProps {
  event: Partial<CalendarEvent>;
  calendars: Array<{
    id: string;
    name: string;
    color: string;
  }>;
  onSubmit: (eventData: Partial<CalendarEvent>) => Promise<void>;
  onCancel: () => void;
}

export function EventForm({
  event: initialEvent,
  calendars,
  onSubmit,
  onCancel,
}: EventFormProps) {
  const [event, setEvent] = useState<Partial<CalendarEvent>>(initialEvent);
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [showRecurrence, setShowRecurrence] = useState(false);
  const [showAttendees, setShowAttendees] = useState(false);
  const [isOnline, setIsOnline] = useState(initialEvent.isOnline || false);

  // Default to today if no start date
  useEffect(() => {
    if (!event.start) {
      const now = new Date();
      const start = new Date(now);
      start.setMinutes(0, 0, 0);
      
      const end = new Date(start);
      end.setHours(start.getHours() + 1);
      
      setEvent(prev => ({
        ...prev,
        start,
        end,
      }));
    }
  }, []);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!event.title?.trim() || !event.start || !event.end || !event.calendarId) {
      alert('Please fill in all required fields');
      return;
    }

    setIsSubmitting(true);
    try {
      await onSubmit(event);
    } catch (error) {
      console.error('Failed to save event:', error);
      alert('Failed to save event');
    } finally {
      setIsSubmitting(false);
    }
  };

  const handleDateTimeChange = (type: 'start' | 'end', date: Date) => {
    setEvent(prev => ({ ...prev, [type]: date }));
  };

  const handleAllDayChange = (checked: boolean) => {
    setEvent(prev => ({
      ...prev,
      allDay: checked,
      start: prev.start ? new Date(prev.start.setHours(0, 0, 0, 0)) : undefined,
      end: prev.end ? new Date(prev.end.setHours(23, 59, 59, 999)) : undefined,
    }));
  };

  const handleCalendarChange = (calendarId: string) => {
    const calendar = calendars.find(c => c.id === calendarId);
    setEvent(prev => ({
      ...prev,
      calendarId,
      color: calendar?.color || prev.color,
    }));
  };

  const handleLocationTypeChange = (type: 'in-person' | 'online') => {
    const isOnline = type === 'online';
    setIsOnline(isOnline);
    setEvent(prev => ({
      ...prev,
      isOnline,
      meetingUrl: isOnline ? prev.meetingUrl : undefined,
    }));
  };

  return (
    <Dialog open={true} onOpenChange={onCancel}>
      <DialogContent className="max-w-2xl max-h-[90vh] overflow-y-auto">
        <DialogHeader>
          <DialogTitle className="text-2xl">
            {initialEvent.id ? 'Edit Event' : 'New Event'}
          </DialogTitle>
        </DialogHeader>

        <form onSubmit={handleSubmit} className="space-y-6">
          {/* Title */}
          <div className="space-y-2">
            <Label htmlFor="title">Title *</Label>
            <Input
              id="title"
              value={event.title || ''}
              onChange={(e) => setEvent(prev => ({ ...prev, title: e.target.value }))}
              placeholder="Event title"
              required
            />
          </div>

          {/* Calendar */}
          <div className="space-y-2">
            <Label htmlFor="calendar">Calendar *</Label>
            <Select
              value={event.calendarId || ''}
              onValueChange={handleCalendarChange}
            >
              <SelectTrigger>
                <SelectValue placeholder="Select calendar" />
              </SelectTrigger>
              <SelectContent>
                {calendars.map(calendar => (
                  <SelectItem key={calendar.id} value={calendar.id}>
                    <div className="flex items-center gap-2">
                      <div 
                        className="h-3 w-3 rounded-full"
                        style={{ backgroundColor: calendar.color }}
                      />
                      {calendar.name}
                    </div>
                  </SelectItem>
                ))}
              </SelectContent>
            </Select>
          </div>

          {/* Date & Time */}
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label>Start *</Label>
              <DateTimePicker
                date={event.start || new Date()}
                onChange={(date) => handleDateTimeChange('start', date)}
                showTime={!event.allDay}
              />
            </div>
            
            <div className="space-y-2">
              <Label>End *</Label>
              <DateTimePicker
                date={event.end || new Date()}
                onChange={(date) => handleDateTimeChange('end', date)}
                showTime={!event.allDay}
                minDate={event.start}
              />
            </div>
          </div>

          {/* All day switch */}
          <div className="flex items-center space-x-2">
            <Switch
              id="all-day"
              checked={event.allDay || false}
              onCheckedChange={handleAllDayChange}
            />
            <Label htmlFor="all-day">All day event</Label>
          </div>

          {/* Recurrence */}
          <div className="space-y-2">
            <div className="flex items-center justify-between">
              <Label>Repeat</Label>
              <Button
                type="button"
                variant="ghost"
                size="sm"
                onClick={() => setShowRecurrence(!showRecurrence)}
              >
                {showRecurrence ? 'Hide' : 'Set up repetition'}
              </Button>
            </div>
            {showRecurrence && (
              <RecurrenceSelector
                value={event.recurrence}
                onChange={(recurrence) => setEvent(prev => ({ ...prev, recurrence }))}
              />
            )}
          </div>

          {/* Description */}
          <div className="space-y-2">
            <Label htmlFor="description">Description</Label>
            <Textarea
              id="description"
              value={event.description || ''}
              onChange={(e) => setEvent(prev => ({ ...prev, description: e.target.value }))}
              placeholder="Event description"
              rows={3}
            />
          </div>

          {/* Location */}
          <div className="space-y-3">
            <Label>Location</Label>
            
            {/* Location type toggle */}
            <div className="flex gap-2">
              <Button
                type="button"
                variant={!isOnline ? "default" : "outline"}
                size="sm"
                onClick={() => handleLocationTypeChange('in-person')}
                className="gap-2"
              >
                <MapPin className="h-4 w-4" />
                In-person
              </Button>
              <Button
                type="button"
                variant={isOnline ? "default" : "outline"}
                size="sm"
                onClick={() => handleLocationTypeChange('online')}
                className="gap-2"
              >
                <Video className="h-4 w-4" />
                Online
              </Button>
            </div>

            {/* Location input */}
            {!isOnline ? (
              <Input
                value={event.location || ''}
                onChange={(e) => setEvent(prev => ({ ...prev, location: e.target.value }))}
                placeholder="Enter location"
              />
            ) : (
              <Input
                value={event.meetingUrl || ''}
                onChange={(e) => setEvent(prev => ({ ...prev, meetingUrl: e.target.value }))}
                placeholder="Enter meeting URL"
                type="url"
              />
            )}
          </div>

          {/* Color */}
          <div className="space-y-2">
            <Label>Color</Label>
            <div className="flex gap-2">
              {[
                '#3b82f6', // blue
                '#10b981', // green
                '#f59e0b', // orange
                '#ef4444', // red
                '#8b5cf6', // purple
                '#ec4899', // pink
              ].map(color => (
                <button
                  key={color}
                  type="button"
                  className={cn(
                    'h-8 w-8 rounded-full border-2',
                    event.color === color ? 'border-gray-900' : 'border-transparent'
                  )}
                  style={{ backgroundColor: color }}
                  onClick={() => setEvent(prev => ({ ...prev, color }))}
                />
              ))}
            </div>
          </div>

          {/* Attendees */}
          <div className="space-y-2">
            <div className="flex items-center justify-between">
              <Label>Attendees</Label>
              <Button
                type="button"
                variant="ghost"
                size="sm"
                onClick={() => setShowAttendees(!showAttendees)}
              >
                {showAttendees ? 'Hide' : 'Add attendees'}
              </Button>
            </div>
            {showAttendees && (
              <AttendeeSelector
                attendees={event.attendees || []}
                onChange={(attendees) => setEvent(prev => ({ ...prev, attendees }))}
              />
            )}
          </div>

          {/* Status */}
          <div className="space-y-2">
            <Label htmlFor="status">Status</Label>
            <Select
              value={event.status || 'CONFIRMED'}
              onValueChange={(value: 'TENTATIVE' | 'CONFIRMED' | 'CANCELLED') =>
                setEvent(prev => ({ ...prev, status: value }))
              }
            >
              <SelectTrigger>
                <SelectValue />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="TENTATIVE">Tentative</SelectItem>
                <SelectItem value="CONFIRMED">Confirmed</SelectItem>
                <SelectItem value="CANCELLED">Cancelled</SelectItem>
              </SelectContent>
            </Select>
          </div>

          {/* Dialog Footer */}
          <DialogFooter className="pt-6 border-t">
            <Button
              type="button"
              variant="outline"
              onClick={onCancel}
              disabled={isSubmitting}
            >
              Cancel
            </Button>
            <Button
              type="submit"
              disabled={isSubmitting}
            >
              {isSubmitting ? 'Saving...' : initialEvent.id ? 'Update Event' : 'Create Event'}
            </Button>
          </DialogFooter>
        </form>
      </DialogContent>
    </Dialog>
  );
}

§ §5. RECURRING EVENTS

§ 5.1 RRULE IMPLEMENTATION

typescript
// lib/calendar/recurrence.ts
import { RRule, RRuleSet, rrulestr, Frequency } from 'rrule';

export type RecurrenceFrequency = 'DAILY' | 'WEEKLY' | 'MONTHLY' | 'YEARLY';

export interface RecurrenceRule {
  freq: RecurrenceFrequency;
  interval?: number;
  count?: number;
  until?: Date;
  byweekday?: number[]; // 0 = Monday, 6 = Sunday
  bymonthday?: number[];
  bymonth?: number[];
  timezone: string;
}

export interface RecurrenceOptions {
  startDate: Date;
  endDate?: Date;
  rule: RecurrenceRule;
  exceptions?: Date[];
}

export class RecurrenceService {
  static createRule(options: RecurrenceOptions): string {
    const { freq, interval = 1, count, until, byweekday, bymonthday, bymonth } = options.rule;
    
    const rrule = new RRule({
      freq: RRule[freq],
      interval,
      count,
      until,
      byweekday: byweekday?.map(day => RRule[['MO', 'TU', 'WE', 'TH', 'FR', 'SA', 'SU'][day]]),
      bymonthday,
      bymonth,
      dtstart: options.startDate,
    });
    
    return rrule.toString();
  }

  static expandRule(
    ruleString: string,
    startDate: Date,
    endDate: Date,
    exceptions: Date[] = []
  ): Date[] {
    const rruleSet = new RRuleSet();
    const rule = rrulestr(ruleString, { forceset: true });
    rruleSet.rrule(rule);
    
    // Add exceptions
    exceptions.forEach(date => {
      rruleSet.exdate(date);
    });
    
    // Generate occurrences between dates
    return rruleSet.between(startDate, endDate);
  }

  static parseRule(ruleString: string): RecurrenceRule | null {
    try {
      const rule = rrulestr(ruleString);
      
      return {
        freq: Frequency[rule.options.freq] as RecurrenceFrequency,
        interval: rule.options.interval || 1,
        count: rule.options.count,
        until: rule.options.until,
        byweekday: rule.options.byweekday?.map((wday: any) => {
          if (typeof wday === 'number') return wday;
          return wday.weekday;
        }),
        bymonthday: rule.options.bymonthday,
        bymonth: rule.options.bymonth,
        timezone: 'UTC', // Default, should be stored separately
      };
    } catch (error) {
      console.error('Failed to parse recurrence rule:', error);
      return null;
    }
  }

  static getNextOccurrence(ruleString: string, afterDate: Date = new Date()): Date | null {
    try {
      const rule = rrulestr(ruleString);
      const next = rule.after(afterDate);
      return next;
    } catch (error) {
      console.error('Failed to get next occurrence:', error);
      return null;
    }
  }

  static getPresetRule(freq: RecurrenceFrequency, interval: number = 1): RecurrenceRule {
    return {
      freq,
      interval,
      timezone: 'UTC',
    };
  }

  static getWeekdayNames(weekdays: number[]): string[] {
    const dayNames = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'];
    return weekdays.map(day => dayNames[day]);
  }
}

§ 5.2 RECURRENCE SELECTOR

typescript
// components/calendar/RecurrenceSelector.tsx
'use client';

import { useState } from 'react';
import { Label } from '@/components/ui/label';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { RadioGroup, RadioGroupItem } from '@/components/ui/radio-group';
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';
import { Checkbox } from '@/components/ui/checkbox';
import { RecurrenceFrequency, RecurrenceRule } from '@/lib/calendar/recurrence';
import { cn } from '@/lib/utils';
import { Calendar, RefreshCw } from 'lucide-react';

interface RecurrenceSelectorProps {
  value?: any;
  onChange: (recurrence: any) => void;
}

export function RecurrenceSelector({ value, onChange }: RecurrenceSelectorProps) {
  const [showCustom, setShowCustom] = useState(false);
  const [frequency, setFrequency] = useState<RecurrenceFrequency>('WEEKLY');
  const [interval, setInterval] = useState(1);
  const [endType, setEndType] = useState<'never' | 'after' | 'until'>('never');
  const [occurrenceCount, setOccurrenceCount] = useState(10);
  const [endDate, setEndDate] = useState<string>('');
  const [selectedWeekdays, setSelectedWeekdays] = useState<number[]>([1]); // Tuesday by default
  const [selectedMonthday, setSelectedMonthday] = useState<number>(1);

  const presetOptions = [
    { label: 'Daily', value: 'DAILY' },
    { label: 'Weekly', value: 'WEEKLY' },
    { label: 'Monthly', value: 'MONTHLY' },
    { label: 'Yearly', value: 'YEARLY' },
  ];

  const weekdayOptions = [
    { label: 'Monday', value: 0 },
    { label: 'Tuesday', value: 1 },
    { label: 'Wednesday', value: 2 },
    { label: 'Thursday', value: 3 },
    { label: 'Friday', value: 4 },
    { label: 'Saturday', value: 5 },
    { label: 'Sunday', value: 6 },
  ];

  const handlePresetChange = (freq: RecurrenceFrequency) => {
    setFrequency(freq);
    updateRule();
  };

  const handleWeekdayToggle = (day: number) => {
    const newWeekdays = selectedWeekdays.includes(day)
      ? selectedWeekdays.filter(d => d !== day)
      : [...selectedWeekdays, day];
    setSelectedWeekdays(newWeekdays.sort());
    updateRule();
  };

  const updateRule = () => {
    const rule: RecurrenceRule = {
      freq: frequency,
      interval,
      timezone: Intl.DateTimeFormat().resolvedOptions().timeZone,
    };

    if (frequency === 'WEEKLY' && selectedWeekdays.length > 0) {
      rule.byweekday = selectedWeekdays;
    }

    if (frequency === 'MONTHLY') {
      rule.bymonthday = [selectedMonthday];
    }

    if (endType === 'after') {
      rule.count = occurrenceCount;
    } else if (endType === 'until' && endDate) {
      rule.until = new Date(endDate);
    }

    onChange(rule);
  };

  const renderFrequencyOptions = () => {
    switch (frequency) {
      case 'WEEKLY':
        return (
          <div className="space-y-2">
            <Label>On days</Label>
            <div className="flex flex-wrap gap-2">
              {weekdayOptions.map(day => (
                <Button
                  key={day.value}
                  type="button"
                  variant={selectedWeekdays.includes(day.value) ? "default" : "outline"}
                  size="sm"
                  onClick={() => handleWeekdayToggle(day.value)}
                >
                  {day.label.slice(0, 3)}
                </Button>
              ))}
            </div>
          </div>
        );

      case 'MONTHLY':
        return (
          <div className="space-y-2">
            <Label>On day</Label>
            <Select
              value={selectedMonthday.toString()}
              onValueChange={(value) => {
                setSelectedMonthday(parseInt(value));
                updateRule();
              }}
            >
              <SelectTrigger>
                <SelectValue />
              </SelectTrigger>
              <SelectContent>
                {Array.from({ length: 31 }, (_, i) => i + 1).map(day => (
                  <SelectItem key={day} value={day.toString()}>
                    {day}{day === 1 ? 'st' : day === 2 ? 'nd' : day === 3 ? 'rd' : 'th'}
                  </SelectItem>
                ))}
              </SelectContent>
            </Select>
          </div>
        );

      default:
        return null;
    }
  };

  return (
    <div className="space-y-4 p-4 border rounded-lg">
      <div className="flex items-center gap-2">
        <RefreshCw className="h-4 w-4 text-muted-foreground" />
        <h4 className="font-medium">Repeat</h4>
      </div>

      {/* Frequency selection */}
      <div className="space-y-2">
        <Label>Frequency</Label>
        <div className="flex flex-wrap gap-2">
          {presetOptions.map(option => (
            <Button
              key={option.value}
              type="button"
              variant={frequency === option.value ? "default" : "outline"}
              size="sm"
              onClick={() => handlePresetChange(option.value as RecurrenceFrequency)}
            >
              {option.label}
            </Button>
          ))}
        </div>
      </div>

      {/* Interval */}
      <div className="space-y-2">
        <Label>Every</Label>
        <div className="flex items-center gap-2">
          <Input
            type="number"
            min="1"
            value={interval}
            onChange={(e) => {
              setInterval(parseInt(e.target.value) || 1);
              updateRule();
            }}
            className="w-20"
          />
          <span className="text-sm text-muted-foreground">
            {frequency === 'DAILY' && interval === 1 ? 'day' : 
             frequency === 'DAILY' ? 'days' :
             frequency === 'WEEKLY' && interval === 1 ? 'week' :
             frequency === 'WEEKLY' ? 'weeks' :
             frequency === 'MONTHLY' && interval === 1 ? 'month' :
             frequency === 'MONTHLY' ? 'months' :
             frequency === 'YEARLY' && interval === 1 ? 'year' : 'years'}
          </span>
        </div>
      </div>

      {/* Day/Weekday options */}
      {renderFrequencyOptions()}

      {/* End condition */}
      <div className="space-y-3">
        <Label>Ends</Label>
        <RadioGroup value={endType} onValueChange={(value: any) => setEndType(value)}>
          <div className="flex items-center space-x-2">
            <RadioGroupItem value="never" id="never" />
            <Label htmlFor="never" className="font-normal">Never</Label>
          </div>
          
          <div className="flex items-center space-x-2">
            <RadioGroupItem value="after" id="after" />
            <Label htmlFor="after" className="font-normal">After</Label>
            {endType === 'after' && (
              <div className="flex items-center gap-2 ml-4">
                <Input
                  type="number"
                  min="1"
                  value={occurrenceCount}
                  onChange={(e) => setOccurrenceCount(parseInt(e.target.value) || 1)}
                  className="w-20"
                />
                <span className="text-sm text-muted-foreground">occurrences</span>
              </div>
            )}
          </div>
          
          <div className="flex items-center space-x-2">
            <RadioGroupItem value="until" id="until" />
            <Label htmlFor="until" className="font-normal">On</Label>
            {endType === 'until' && (
              <Input
                type="date"
                value={endDate}
                onChange={(e) => setEndDate(e.target.value)}
                className="ml-4 w-40"
                min={new Date().toISOString().split('T')[0]}
              />
            )}
          </div>
        </RadioGroup>
      </div>

      <div className="pt-4 border-t">
        <Button
          type="button"
          variant="outline"
          size="sm"
          onClick={updateRule}
          className="w-full"
        >
          Update recurrence rule
        </Button>
      </div>
    </div>
  );
}

[Continua... Le sezioni rimanenti includerebbero:

§ §6. AVAILABILITY & BOOKING
- Availability service e model
- Booking flow completo
- Slot picker component
- Conflict detection

§ §7. CALENDAR INTEGRATIONS
- Google Calendar OAuth setup
- iCal export implementation
- Webhook per sincronizzazione

§ §8. REMINDERS & NOTIFICATIONS
- Scheduled jobs con Inngest/CRON
- Email notification templates
- Push notification setup

§ §9. TIME & TIMEZONE
- Timezone conversion utilities
- DateTimePicker component completo
- User timezone preferences

§ §10. CALENDAR CHECKLIST
- Checklist completo per deployment
- Testing strategies
- Performance optimization
- Mobile responsiveness

Il catalogo completo sarebbe di 1200-1500 righe con codice production-ready.]