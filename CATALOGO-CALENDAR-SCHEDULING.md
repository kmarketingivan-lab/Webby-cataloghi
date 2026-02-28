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

## § ADVANCED PATTERNS: CALENDAR SCHEDULING

### Server Actions con Validazione

```typescript
// app/actions/calendar-scheduling.ts
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


### CALENDAR SCHEDULING - Utility Helper #317

```typescript
// lib/utils/calendar-scheduling-helper-317.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CALENDAR SCHEDULING - Utility Helper #164

```typescript
// lib/utils/calendar-scheduling-helper-164.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CALENDAR SCHEDULING - Utility Helper #208

```typescript
// lib/utils/calendar-scheduling-helper-208.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CALENDAR SCHEDULING - Utility Helper #484

```typescript
// lib/utils/calendar-scheduling-helper-484.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CALENDAR SCHEDULING - Utility Helper #229

```typescript
// lib/utils/calendar-scheduling-helper-229.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CALENDAR SCHEDULING - Utility Helper #382

```typescript
// lib/utils/calendar-scheduling-helper-382.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CALENDAR SCHEDULING - Utility Helper #487

```typescript
// lib/utils/calendar-scheduling-helper-487.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CALENDAR SCHEDULING - Utility Helper #730

```typescript
// lib/utils/calendar-scheduling-helper-730.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CALENDAR SCHEDULING - Utility Helper #564

```typescript
// lib/utils/calendar-scheduling-helper-564.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CALENDAR SCHEDULING - Utility Helper #857

```typescript
// lib/utils/calendar-scheduling-helper-857.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CALENDAR SCHEDULING - Utility Helper #335

```typescript
// lib/utils/calendar-scheduling-helper-335.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CALENDAR SCHEDULING - Utility Helper #26

```typescript
// lib/utils/calendar-scheduling-helper-26.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CALENDAR SCHEDULING - Utility Helper #438

```typescript
// lib/utils/calendar-scheduling-helper-438.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CALENDAR SCHEDULING - Utility Helper #907

```typescript
// lib/utils/calendar-scheduling-helper-907.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CALENDAR SCHEDULING - Utility Helper #635

```typescript
// lib/utils/calendar-scheduling-helper-635.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CALENDAR SCHEDULING - Utility Helper #64

```typescript
// lib/utils/calendar-scheduling-helper-64.ts
import { z } from "zod";

interface CALENDARSCHEDULINGConfig {
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

export class CALENDARSCHEDULINGProcessor<TInput, TOutput> {
  private config: CALENDARSCHEDULINGConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CALENDARSCHEDULINGConfig> = {}) {
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

  getConfig(): Readonly<CALENDARSCHEDULINGConfig> {
    return Object.freeze({ ...this.config });
  }
}
```
