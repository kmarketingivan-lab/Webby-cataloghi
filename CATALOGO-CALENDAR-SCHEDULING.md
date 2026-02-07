# CATALOGO-CALENDAR-SCHEDULING

CATALOGO-CALENDAR-SCHEDULING ESPANSO
Next.js 14 + TypeScript
§ FULLCALENDAR INTEGRATION
Installazione dipendenze
bash
npm install @fullcalendar/core @fullcalendar/react @fullcalendar/daygrid @fullcalendar/timegrid @fullcalendar/interaction @fullcalendar/resource-timeline
npm install @fullcalendar/list @fullcalendar/multimonth @fullcalendar/rrule
npm install @fullcalendar/google-calendar
Configurazione FullCalendar React
typescript
// components/calendar/FullCalendarWrapper.tsx
'use client';

import React, { useState, useCallback, useRef } from 'react';
import FullCalendar from '@fullcalendar/react';
import dayGridPlugin from '@fullcalendar/daygrid';
import timeGridPlugin from '@fullcalendar/timegrid';
import interactionPlugin, { DateClickArg, EventDragStopArg, EventResizeDoneArg } from '@fullcalendar/interaction';
import resourceTimelinePlugin from '@fullcalendar/resource-timeline';
import listPlugin from '@fullcalendar/list';
import rrulePlugin from '@fullcalendar/rrule';
import { EventInput, EventApi, DateSelectArg, EventClickArg, EventChangeArg } from '@fullcalendar/core';
import { EventModal } from './EventModal';
import { Resource } from '@prisma/client';

interface FullCalendarWrapperProps {
  initialView?: string;
  resources?: Resource[];
  events?: EventInput[];
  editable?: boolean;
  selectable?: boolean;
}

export const FullCalendarWrapper: React.FC<FullCalendarWrapperProps> = ({
  initialView = 'dayGridMonth',
  resources = [],
  events = [],
  editable = true,
  selectable = true,
}) => {
  const [calendarEvents, setCalendarEvents] = useState<EventInput[]>(events);
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [selectedEvent, setSelectedEvent] = useState<EventInput | null>(null);
  const [modalMode, setModalMode] = useState<'create' | 'edit'>('create');
  const calendarRef = useRef<FullCalendar>(null);

  // Handle date selection for new event
  const handleDateSelect = useCallback((selectInfo: DateSelectArg) => {
    setSelectedEvent({
      start: selectInfo.startStr,
      end: selectInfo.endStr,
      allDay: selectInfo.allDay,
    });
    setModalMode('create');
    setIsModalOpen(true);
    selectInfo.view.calendar.unselect();
  }, []);

  // Handle event click for editing
  const handleEventClick = useCallback((clickInfo: EventClickArg) => {
    setSelectedEvent({
      id: clickInfo.event.id,
      title: clickInfo.event.title,
      start: clickInfo.event.startStr,
      end: clickInfo.event.endStr,
      allDay: clickInfo.event.allDay,
      extendedProps: clickInfo.event.extendedProps,
      resourceId: clickInfo.event.resourceId,
    });
    setModalMode('edit');
    setIsModalOpen(true);
  }, []);

  // Handle event drop (drag and drop)
  const handleEventDrop = useCallback((dropInfo: EventChangeArg) => {
    const updatedEvents = calendarEvents.map(event => {
      if (event.id === dropInfo.event.id) {
        return {
          ...event,
          start: dropInfo.event.startStr,
          end: dropInfo.event.endStr,
        };
      }
      return event;
    });
    setCalendarEvents(updatedEvents);
    // In produzione: chiamata API per aggiornare evento
  }, [calendarEvents]);

  // Handle event resize
  const handleEventResize = useCallback((resizeInfo: EventResizeDoneArg) => {
    const updatedEvents = calendarEvents.map(event => {
      if (event.id === resizeInfo.event.id) {
        return {
          ...event,
          start: resizeInfo.event.startStr,
          end: resizeInfo.event.endStr,
        };
      }
      return event;
    });
    setCalendarEvents(updatedEvents);
  }, [calendarEvents]);

  // Save event (create or update)
  const handleSaveEvent = useCallback((eventData: EventInput) => {
    if (modalMode === 'create') {
      const newEvent = {
        ...eventData,
        id: `event_${Date.now()}`,
      };
      setCalendarEvents([...calendarEvents, newEvent]);
    } else {
      const updatedEvents = calendarEvents.map(event =>
        event.id === eventData.id ? eventData : event
      );
      setCalendarEvents(updatedEvents);
    }
    setIsModalOpen(false);
  }, [calendarEvents, modalMode]);

  // Delete event
  const handleDeleteEvent = useCallback((eventId: string) => {
    setCalendarEvents(calendarEvents.filter(event => event.id !== eventId));
    setIsModalOpen(false);
  }, [calendarEvents]);

  return (
    <div className="fullcalendar-wrapper">
      <FullCalendar
        ref={calendarRef}
        plugins={[
          dayGridPlugin,
          timeGridPlugin,
          interactionPlugin,
          resourceTimelinePlugin,
          listPlugin,
          rrulePlugin,
        ]}
        headerToolbar={{
          left: 'prev,next today',
          center: 'title',
          right: 'dayGridMonth,timeGridWeek,timeGridDay,listWeek,resourceTimelineWeek',
        }}
        initialView={initialView}
        editable={editable}
        selectable={selectable}
        select={handleDateSelect}
        eventClick={handleEventClick}
        eventDrop={handleEventDrop}
        eventResize={handleEventResize}
        events={calendarEvents}
        resources={resources.map(resource => ({
          id: resource.id,
          title: resource.name,
          ...resource,
        }))}
        resourceAreaWidth="200px"
        resourceAreaHeaderContent="Resources"
        height="auto"
        slotMinTime="08:00:00"
        slotMaxTime="20:00:00"
        businessHours={{
          daysOfWeek: [1, 2, 3, 4, 5], // Lunedì-Venerdì
          startTime: '09:00',
          endTime: '18:00',
        }}
        nowIndicator={true}
        droppable={true}
        eventResizableFromStart={true}
        eventDurationEditable={true}
        eventResourceEditable={true}
        allDaySlot={false}
      />

      <EventModal
        isOpen={isModalOpen}
        onClose={() => setIsModalOpen(false)}
        onSave={handleSaveEvent}
        onDelete={handleDeleteEvent}
        event={selectedEvent}
        mode={modalMode}
        resources={resources}
      />
    </div>
  );
};
Modal per CRUD Eventi
typescript
// components/calendar/EventModal.tsx
'use client';

import React, { useState, useEffect } from 'react';
import { EventInput } from '@fullcalendar/core';
import { Resource } from '@prisma/client';

interface EventModalProps {
  isOpen: boolean;
  onClose: () => void;
  onSave: (event: EventInput) => void;
  onDelete: (eventId: string) => void;
  event: EventInput | null;
  mode: 'create' | 'edit';
  resources: Resource[];
}

export const EventModal: React.FC<EventModalProps> = ({
  isOpen,
  onClose,
  onSave,
  onDelete,
  event,
  mode,
  resources,
}) => {
  const [formData, setFormData] = useState<EventInput>({
    title: '',
    start: '',
    end: '',
    allDay: false,
    resourceId: '',
    extendedProps: {},
  });

  useEffect(() => {
    if (event) {
      setFormData(event);
    } else {
      setFormData({
        title: '',
        start: new Date().toISOString(),
        end: new Date(Date.now() + 60 * 60 * 1000).toISOString(),
        allDay: false,
        resourceId: resources[0]?.id || '',
        extendedProps: {},
      });
    }
  }, [event, resources]);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSave(formData);
  };

  const handleDelete = () => {
    if (formData.id) {
      onDelete(formData.id);
    }
  };

  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
      <div className="bg-white rounded-lg p-6 w-full max-w-md">
        <h2 className="text-2xl font-bold mb-4">
          {mode === 'create' ? 'Nuovo Evento' : 'Modifica Evento'}
        </h2>
        
        <form onSubmit={handleSubmit}>
          <div className="space-y-4">
            <div>
              <label className="block text-sm font-medium mb-1">Titolo</label>
              <input
                type="text"
                value={formData.title || ''}
                onChange={(e) => setFormData({...formData, title: e.target.value})}
                className="w-full p-2 border rounded"
                required
              />
            </div>

            <div>
              <label className="block text-sm font-medium mb-1">Inizio</label>
              <input
                type="datetime-local"
                value={formData.start ? new Date(formData.start as string).toISOString().slice(0, 16) : ''}
                onChange={(e) => setFormData({...formData, start: e.target.value})}
                className="w-full p-2 border rounded"
                required
              />
            </div>

            <div>
              <label className="block text-sm font-medium mb-1">Fine</label>
              <input
                type="datetime-local"
                value={formData.end ? new Date(formData.end as string).toISOString().slice(0, 16) : ''}
                onChange={(e) => setFormData({...formData, end: e.target.value})}
                className="w-full p-2 border rounded"
                required
              />
            </div>

            <div>
              <label className="block text-sm font-medium mb-1">Risorsa</label>
              <select
                value={formData.resourceId as string}
                onChange={(e) => setFormData({...formData, resourceId: e.target.value})}
                className="w-full p-2 border rounded"
              >
                {resources.map(resource => (
                  <option key={resource.id} value={resource.id}>
                    {resource.name}
                  </option>
                ))}
              </select>
            </div>

            <div className="flex items-center">
              <input
                type="checkbox"
                id="allDay"
                checked={formData.allDay as boolean}
                onChange={(e) => setFormData({...formData, allDay: e.target.checked})}
                className="mr-2"
              />
              <label htmlFor="allDay" className="text-sm font-medium">
                Tutto il giorno
              </label>
            </div>

            <div className="flex justify-end space-x-3 pt-4">
              {mode === 'edit' && (
                <button
                  type="button"
                  onClick={handleDelete}
                  className="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
                >
                  Elimina
                </button>
              )}
              <button
                type="button"
                onClick={onClose}
                className="px-4 py-2 border rounded hover:bg-gray-50"
              >
                Annulla
              </button>
              <button
                type="submit"
                className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
              >
                Salva
              </button>
            </div>
          </div>
        </form>
      </div>
    </div>
  );
};
§ DATE-FNS PATTERNS COMPLETI
Installazione dipendenze
bash
npm install date-fns date-fns-tz rrule @date-fns/utc
npm install @types/rrule
Utility per Timezone Handling
typescript
// lib/date-utils/timezone.ts
import { format, formatInTimeZone, toZonedTime, fromZonedTime } from 'date-fns-tz';
import { addDays, isSameDay, parseISO, isValid } from 'date-fns';

export class TimezoneManager {
  private userTimezone: string;
  private serverTimezone: string = 'UTC';

  constructor(userTimezone?: string) {
    this.userTimezone = userTimezone || Intl.DateTimeFormat().resolvedOptions().timeZone;
  }

  // Converti da UTC a timezone utente
  toUserTimezone(date: Date | string): Date {
    const dateObj = typeof date === 'string' ? parseISO(date) : date;
    return toZonedTime(dateObj, this.userTimezone);
  }

  // Converti da timezone utente a UTC
  toUTC(date: Date | string): Date {
    const dateObj = typeof date === 'string' ? parseISO(date) : date;
    return fromZonedTime(dateObj, this.userTimezone);
  }

  // Formatta con timezone specifica
  formatWithTimezone(
    date: Date | string,
    formatString: string,
    timezone?: string
  ): string {
    const dateObj = typeof date === 'string' ? parseISO(date) : date;
    const targetTimezone = timezone || this.userTimezone;
    return formatInTimeZone(dateObj, targetTimezone, formatString);
  }

  // Compara date ignorando timezone
  isSameDateInTimezone(date1: Date | string, date2: Date | string): boolean {
    const d1 = this.toUserTimezone(date1);
    const d2 = this.toUserTimezone(date2);
    return isSameDay(d1, d2);
  }

  // Ottieni offset timezone corrente
  getTimezoneOffset(): number {
    return new Date().getTimezoneOffset();
  }

  // Lista timezone supportate
  static getAvailableTimezones(): string[] {
    return Intl.supportedValuesOf('timeZone');
  }
}
Gestione Eventi Ricorrenti (RRule)
typescript
// lib/date-utils/recurring-events.ts
import { RRule, RRuleSet, rrulestr } from 'rrule';
import { addDays, isSameDay, parseISO, isValid, eachDayOfInterval } from 'date-fns';
import { TimezoneManager } from './timezone';

export interface RecurrenceRule {
  freq: RRule.Frequency;
  interval?: number;
  count?: number;
  until?: Date;
  byweekday?: number[];
  bymonth?: number[];
  bysetpos?: number[];
  wkst?: number;
}

export class RecurringEventManager {
  private timezoneManager: TimezoneManager;

  constructor(timezone?: string) {
    this.timezoneManager = new TimezoneManager(timezone);
  }

  // Crea RRule da configurazione
  createRRule(config: RecurrenceRule): RRule {
    return new RRule({
      freq: config.freq,
      interval: config.interval || 1,
      count: config.count,
      until: config.until,
      byweekday: config.byweekday,
      bymonth: config.bymonth,
      bysetpos: config.bysetpos,
      wkst: config.wkst || RRule.MO,
      dtstart: new Date(),
    });
  }

  // Genera occorrenze per un intervallo di date
  generateOccurrences(
    rruleString: string,
    startDate: Date,
    endDate: Date
  ): Date[] {
    try {
      const rule = rrulestr(rruleString);
      return rule.between(startDate, endDate, true);
    } catch (error) {
      console.error('Error generating occurrences:', error);
      return [];
    }
  }

  // Converti date RRule con timezone
  generateOccurrencesWithTimezone(
    rruleString: string,
    startDate: Date,
    endDate: Date,
    timezone: string
  ): Date[] {
    const occurrences = this.generateOccurrences(rruleString, startDate, endDate);
    return occurrences.map(date => 
      this.timezoneManager.toUserTimezone(date)
    );
  }

  // Modifica occorrenza singola (eccezione alla regola)
  createException(
    rruleSet: RRuleSet,
    originalDate: Date,
    newDate?: Date
  ): RRuleSet {
    const newSet = rruleSet.clone();
    
    if (newDate) {
      // Sposta occorrenza
      newSet.rdate(newDate);
    }
    
    // Escludi occorrenza originale
    newSet.exdate(originalDate);
    
    return newSet;
  }

  // Calcola prossima occorrenza disponibile
  getNextAvailableOccurrence(
    rruleString: string,
    fromDate: Date,
    excludedDates: Date[] = []
  ): Date | null {
    const rule = rrulestr(rruleString);
    let nextDate = rule.after(fromDate, true);
    
    while (nextDate) {
      const isExcluded = excludedDates.some(excluded => 
        isSameDay(excluded, nextDate!)
      );
      
      if (!isExcluded) {
        return nextDate;
      }
      
      nextDate = rule.after(addDays(nextDate, 1), true);
    }
    
    return null;
  }
}

// Utility per giorni lavorativi
export class BusinessDayCalculator {
  private holidays: Date[] = [];
  private workHours = { start: 9, end: 18 };
  private workDays = [1, 2, 3, 4, 5]; // Lun-Ven

  constructor(holidays?: Date[]) {
    this.holidays = holidays || [];
  }

  // Verifica se è giorno lavorativo
  isBusinessDay(date: Date): boolean {
    const dayOfWeek = date.getDay();
    const isWeekday = this.workDays.includes(dayOfWeek);
    const isHoliday = this.holidays.some(holiday => 
      isSameDay(holiday, date)
    );
    
    return isWeekday && !isHoliday;
  }

  // Calcola prossimo giorno lavorativo
  getNextBusinessDay(fromDate: Date): Date {
    let nextDate = addDays(fromDate, 1);
    
    while (!this.isBusinessDay(nextDate)) {
      nextDate = addDays(nextDate, 1);
    }
    
    return nextDate;
  }

  // Calcola giorni lavorativi in un intervallo
  countBusinessDays(startDate: Date, endDate: Date): number {
    const days = eachDayOfInterval({ start: startDate, end: endDate });
    return days.filter(day => this.isBusinessDay(day)).length;
  }

  // Calcola orari di lavoro validi
  isValidBusinessTime(date: Date): boolean {
    if (!this.isBusinessDay(date)) return false;
    
    const hours = date.getHours();
    return hours >= this.workHours.start && hours < this.workHours.end;
  }

  // Aggiusta data/ora al prossimo orario lavorativo
  adjustToBusinessHours(date: Date): Date {
    const adjusted = new Date(date);
    
    // Se è weekend o festivo, vai al prossimo giorno lavorativo
    if (!this.isBusinessDay(adjusted)) {
      adjusted.setDate(adjusted.getDate() + 1);
      while (!this.isBusinessDay(adjusted)) {
        adjusted.setDate(adjusted.getDate() + 1);
      }
      adjusted.setHours(this.workHours.start, 0, 0, 0);
      return adjusted;
    }
    
    // Aggiusta orario se fuori orario lavorativo
    const hours = adjusted.getHours();
    if (hours < this.workHours.start) {
      adjusted.setHours(this.workHours.start, 0, 0, 0);
    } else if (hours >= this.workHours.end) {
      adjusted.setDate(adjusted.getDate() + 1);
      adjusted.setHours(this.workHours.start, 0, 0, 0);
      // Ricorsivamente verifica se è giorno lavorativo
      return this.adjustToBusinessHours(adjusted);
    }
    
    return adjusted;
  }
}
§ AVAILABILITY SYSTEM
Schema Prisma
prisma
// prisma/schema.prisma (sezione availability)
model Availability {
  id          String   @id @default(cuid())
  userId      String
  resourceId  String?
  title       String
  description String?
  
  // Disponibilità ricorrente settimanale
  recurringRules Json     // Array di WeeklyAvailability
  overrides      Json?    // Array di DateOverride
  
  // Buffer tra appuntamenti
  bufferBefore   Int      @default(0) // minuti
  bufferAfter    Int      @default(0) // minuti
  
  // Slot di disponibilità
  slotDuration   Int      @default(30) // minuti
  maxSlotsPerDay Int?     // Limite slot giornalieri
  
  // Date di validità
  startDate      DateTime
  endDate        DateTime?
  
  // Metadata
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt
  
  // Relazioni
  user           User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  resource       Resource? @relation(fields: [resourceId], references: [id], onDelete: Cascade)
  bookings       Booking[]
  
  @@index([userId])
  @@index([resourceId])
  @@index([startDate, endDate])
}

model WeeklyAvailability {
  id             String   @id @default(cuid())
  availabilityId String
  dayOfWeek      Int      // 0=Sunday, 6=Saturday
  startTime      String   // "09:00"
  endTime        String   // "18:00"
  isAvailable    Boolean  @default(true)
  
  availability   Availability @relation(fields: [availabilityId], references: [id], onDelete: Cascade)
  
  @@unique([availabilityId, dayOfWeek])
}

model DateOverride {
  id             String   @id @default(cuid())
  availabilityId String
  date           DateTime // Solo data (time 00:00)
  startTime      String?  // Override orario inizio
  endTime        String?  // Override orario fine
  isAvailable    Boolean  @default(true)
  reason         String?
  
  availability   Availability @relation(fields: [availabilityId], references: [id], onDelete: Cascade)
  
  @@unique([availabilityId, date])
}

model Booking {
  id             String   @id @default(cuid())
  availabilityId String
  userId         String
  resourceId     String?
  
  startTime      DateTime
  endTime        DateTime
  duration       Int      // in minuti
  
  status         BookingStatus @default(PENDING)
  title          String
  description    String?
  
  // Informazioni partecipante
  attendeeName   String
  attendeeEmail  String
  attendeePhone  String?
  
  // Conferma e reminder
  confirmedAt    DateTime?
  reminderSentAt DateTime?
  
  // Metadata
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt
  
  // Relazioni
  availability   Availability @relation(fields: [availabilityId], references: [id], onDelete: Cascade)
  user           User         @relation(fields: [userId], references: [id], onDelete: Cascade)
  resource       Resource?    @relation(fields: [resourceId], references: [id], onDelete: Cascade)
  
  @@index([availabilityId, startTime])
  @@index([userId])
  @@index([startTime])
}

enum BookingStatus {
  PENDING
  CONFIRMED
  CANCELLED
  COMPLETED
  NO_SHOW
}
Service per Gestione Disponibilità
typescript
// lib/services/availability-service.ts
import { PrismaClient, Availability, Booking, BookingStatus } from '@prisma/client';
import { addMinutes, parseISO, format, isWithinInterval, eachMinuteOfInterval } from 'date-fns';
import { TimezoneManager } from '../date-utils/timezone';
import { BusinessDayCalculator } from '../date-utils/recurring-events';

const prisma = new PrismaClient();

export interface Slot {
  start: Date;
  end: Date;
  available: boolean;
  bookingId?: string;
}

export interface AvailableSlot {
  start: Date;
  end: Date;
  availabilityId: string;
  resourceId?: string;
}

export class AvailabilityService {
  private timezoneManager: TimezoneManager;
  private businessDayCalculator: BusinessDayCalculator;

  constructor(userTimezone?: string) {
    this.timezoneManager = new TimezoneManager(userTimezone);
    this.businessDayCalculator = new BusinessDayCalculator();
  }

  // Trova slot disponibili per un intervallo di date
  async findAvailableSlots(
    availabilityId: string,
    startDate: Date,
    endDate: Date,
    durationMinutes: number = 30
  ): Promise<AvailableSlot[]> {
    const availability = await prisma.availability.findUnique({
      where: { id: availabilityId },
      include: {
        recurringRules: true,
        overrides: true,
        bookings: {
          where: {
            status: {
              in: [BookingStatus.PENDING, BookingStatus.CONFIRMED]
            },
            startTime: {
              gte: startDate,
              lt: endDate
            }
          }
        }
      }
    });

    if (!availability) {
      throw new Error('Availability not found');
    }

    const slots: AvailableSlot[] = [];
    const currentDate = new Date(startDate);

    while (currentDate < endDate) {
      const dateSlots = await this.getSlotsForDate(
        availability,
        currentDate,
        durationMinutes
      );
      slots.push(...dateSlots);
      currentDate.setDate(currentDate.getDate() + 1);
    }

    return slots;
  }

  // Ottieni slot per una data specifica
  private async getSlotsForDate(
    availability: Availability & {
      recurringRules: any[];
      overrides: any[];
      bookings: Booking[];
    },
    date: Date,
    durationMinutes: number
  ): Promise<AvailableSlot[]> {
    const dayOfWeek = date.getDay();
    
    // Controlla override per questa data
    const dateOverride = availability.overrides.find(override =>
      isWithinInterval(date, {
        start: new Date(override.date),
        end: addMinutes(new Date(override.date), 1439) // Fine giornata
      })
    );

    let availableSlots: AvailableSlot[] = [];

    if (dateOverride && !dateOverride.isAvailable) {
      return []; // Giorno non disponibile
    }

    // Ottieni orari disponibili per questo giorno
    const dailySchedule = this.getDailySchedule(availability, date, dayOfWeek, dateOverride);
    
    if (!dailySchedule.available) {
      return [];
    }

    // Genera slot
    const slots = this.generateTimeSlots(
      date,
      dailySchedule.startTime,
      dailySchedule.endTime,
      availability.slotDuration,
      durationMinutes,
      availability.bufferBefore,
      availability.bufferAfter
    );

    // Filtra slot occupati da prenotazioni
    const bookedSlots = availability.bookings
      .filter(booking => isWithinInterval(booking.startTime, { start: date, end: addMinutes(date, 1439) }))
      .map(booking => ({
        start: booking.startTime,
        end: booking.endTime
      }));

    availableSlots = slots.filter(slot =>
      !bookedSlots.some(booked =>
        this.isOverlapping(slot.start, slot.end, booked.start, booked.end)
      )
    ).map(slot => ({
      start: slot.start,
      end: slot.end,
      availabilityId: availability.id,
      resourceId: availability.resourceId || undefined
    }));

    return availableSlots;
  }

  // Controlla conflitti di prenotazione
  async checkBookingConflict(
    availabilityId: string,
    startTime: Date,
    endTime: Date,
    excludeBookingId?: string
  ): Promise<boolean> {
    const conflictingBooking = await prisma.booking.findFirst({
      where: {
        availabilityId,
        status: {
          in: [BookingStatus.PENDING, BookingStatus.CONFIRMED]
        },
        id: excludeBookingId ? { not: excludeBookingId } : undefined,
        OR: [
          // Sovrapposizione parziale
          {
            startTime: { lt: endTime },
            endTime: { gt: startTime }
          }
        ]
      }
    });

    return !!conflictingBooking;
  }

  // Crea una prenotazione
  async createBooking(
    availabilityId: string,
    bookingData: {
      startTime: Date;
      endTime: Date;
      title: string;
      description?: string;
      attendeeName: string;
      attendeeEmail: string;
      attendeePhone?: string;
      userId: string;
    }
  ): Promise<Booking> {
    // Verifica conflitti
    const hasConflict = await this.checkBookingConflict(
      availabilityId,
      bookingData.startTime,
      bookingData.endTime
    );

    if (hasConflict) {
      throw new Error('Time slot is already booked');
    }

    // Verifica disponibilità
    const availableSlots = await this.findAvailableSlots(
      availabilityId,
      bookingData.startTime,
      bookingData.endTime
    );

    const isSlotAvailable = availableSlots.some(slot =>
      slot.start.getTime() === bookingData.startTime.getTime() &&
      slot.end.getTime() === bookingData.endTime.getTime()
    );

    if (!isSlotAvailable) {
      throw new Error('Slot is not available');
    }

    // Calcola durata
    const duration = Math.round(
      (bookingData.endTime.getTime() - bookingData.startTime.getTime()) / (1000 * 60)
    );

    // Crea prenotazione
    const booking = await prisma.booking.create({
      data: {
        ...bookingData,
        availabilityId,
        duration,
        status: BookingStatus.PENDING
      }
    });

    return booking;
  }

  // Utility: Genera slot temporali
  private generateTimeSlots(
    date: Date,
    startTime: string,
    endTime: string,
    slotDuration: number,
    requiredDuration: number,
    bufferBefore: number,
    bufferAfter: number
  ): Array<{ start: Date; end: Date }> {
    const slots: Array<{ start: Date; end: Date }> = [];
    
    const [startHour, startMinute] = startTime.split(':').map(Number);
    const [endHour, endMinute] = endTime.split(':').map(Number);
    
    const startDateTime = new Date(date);
    startDateTime.setHours(startHour, startMinute, 0, 0);
    
    const endDateTime = new Date(date);
    endDateTime.setHours(endHour, endMinute, 0, 0);
    
    let currentStart = startDateTime;
    
    while (currentStart < endDateTime) {
      const slotEnd = addMinutes(currentStart, slotDuration);
      
      if (slotEnd <= endDateTime) {
        slots.push({
          start: new Date(currentStart),
          end: new Date(slotEnd)
        });
      }
      
      currentStart = addMinutes(currentStart, slotDuration);
    }
    
    // Filtra per durata richiesta e buffer
    return slots.filter((_, index) => {
      if (index + (requiredDuration / slotDuration) > slots.length) {
        return false;
      }
      
      // Applica buffer
      if (bufferBefore > 0 && index > 0) {
        const previousSlot = slots[index - 1];
        const timeBetween = currentStart.getTime() - previousSlot.end.getTime();
        if (timeBetween < bufferBefore * 60 * 1000) {
          return false;
        }
      }
      
      if (bufferAfter > 0 && index < slots.length - 1) {
        const nextSlot = slots[index + 1];
        const timeBetween = nextSlot.start.getTime() - slotEnd.getTime();
        if (timeBetween < bufferAfter * 60 * 1000) {
          return false;
        }
      }
      
      return true;
    });
  }

  // Utility: Ottieni orari giornalieri
  private getDailySchedule(
    availability: Availability,
    date: Date,
    dayOfWeek: number,
    dateOverride?: any
  ): { available: boolean; startTime: string; endTime: string } {
    if (dateOverride) {
      return {
        available: dateOverride.isAvailable,
        startTime: dateOverride.startTime || '09:00',
        endTime: dateOverride.endTime || '18:00'
      };
    }
    
    const weeklyRule = availability.recurringRules.find(
      (rule: any) => rule.dayOfWeek === dayOfWeek
    );
    
    if (weeklyRule) {
      return {
        available: weeklyRule.isAvailable,
        startTime: weeklyRule.startTime,
        endTime: weeklyRule.endTime
      };
    }
    
    // Default: non disponibile
    return {
      available: false,
      startTime: '09:00',
      endTime: '18:00'
    };
  }

  // Utility: Verifica sovrapposizione
  private isOverlapping(
    start1: Date,
    end1: Date,
    start2: Date,
    end2: Date
  ): boolean {
    return start1 < end2 && end1 > start2;
  }
}
§ BOOKING FLOW
Componente Selezione Slot
typescript
// components/booking/SlotSelector.tsx
'use client';

import React, { useState, useEffect } from 'react';
import { format, addDays, startOfDay, isSameDay } from 'date-fns';
import { it } from 'date-fns/locale';
import { AvailableSlot } from '@/lib/services/availability-service';

interface SlotSelectorProps {
  availabilityId: string;
  duration: number;
  onSlotSelect: (slot: AvailableSlot) => void;
  timezone?: string;
}

export const SlotSelector: React.FC<SlotSelectorProps> = ({
  availabilityId,
  duration,
  onSlotSelect,
  timezone = 'Europe/Rome',
}) => {
  const [selectedDate, setSelectedDate] = useState<Date>(startOfDay(new Date()));
  const [availableSlots, setAvailableSlots] = useState<AvailableSlot[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [selectedSlot, setSelectedSlot] = useState<AvailableSlot | null>(null);

  // Date disponibili per selezione (prossimi 30 giorni)
  const availableDates = Array.from({ length: 30 }, (_, i) =>
    addDays(startOfDay(new Date()), i)
  );

  // Carica slot per data selezionata
  useEffect(() => {
    const loadSlots = async () => {
      if (!selectedDate) return;

      setIsLoading(true);
      try {
        const endOfDay = new Date(selectedDate);
        endOfDay.setHours(23, 59, 59, 999);

        const response = await fetch(
          `/api/availability/${availabilityId}/slots?` +
          new URLSearchParams({
            startDate: selectedDate.toISOString(),
            endDate: endOfDay.toISOString(),
            duration: duration.toString(),
          })
        );

        if (response.ok) {
          const slots = await response.json();
          setAvailableSlots(slots);
        }
      } catch (error) {
        console.error('Error loading slots:', error);
      } finally {
        setIsLoading(false);
      }
    };

    loadSlots();
  }, [selectedDate, availabilityId, duration]);

  const handleDateSelect = (date: Date) => {
    setSelectedDate(date);
    setSelectedSlot(null);
  };

  const handleSlotSelect = (slot: AvailableSlot) => {
    setSelectedSlot(slot);
    onSlotSelect(slot);
  };

  return (
    <div className="slot-selector">
      <div className="mb-8">
        <h3 className="text-lg font-semibold mb-4">Seleziona Data</h3>
        <div className="grid grid-cols-7 gap-2">
          {availableDates.map(date => (
            <button
              key={date.toISOString()}
              onClick={() => handleDateSelect(date)}
              className={`
                p-3 rounded-lg text-center transition-colors
                ${isSameDay(date, selectedDate)
                  ? 'bg-blue-600 text-white'
                  : 'bg-gray-100 hover:bg-gray-200'
                }
                ${isSameDay(date, startOfDay(new Date()))
                  ?

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-08-CALENDAR-SCHEDULING
Prompt ID: 8 / 48
Parte: 2
Exported: 2026-02-06T12:37:56.176Z
Characters: 327
════════════════════════════════════════════════════════════

 type: ResourceType;
  quantity: number;
  features?: Record<string, any>;
  duration?: number;
}

export interface SchedulingOptions {
  startDate: Date;
  endDate: Date;
  duration: number;
  requirements: ResourceRequirement[];
  ignoreConflicts?: string[];
  allowPartial?: boolean;
}

export interface SchedulingResult {
 

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-08-CALENDAR-SCHEDULING
Prompt ID: 8 / 48
Parte: 3
Exported: 2026-02-06T12:41:40.838Z
Characters: 243
════════════════════════════════════════════════════════════

sync getWaitlistPosition(entryId: string): Promise<number> {
    const entry = await prisma.waitlistEntry.findUnique({
      where: { id: entryId },
    });

    if (!entry) {
      throw new Error('Entry not found');
    }

    // Conta entry