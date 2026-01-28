# CATALOGO DRAG-DROP v1

## §1. DRAG & DROP LIBRARY COMPARISON

| Library | Bundle | Touch | A11y | Tree Shaking | Status |
|---------|--------|-------|------|--------------|--------|
| dnd-kit | 15kB | ✅ | ✅ | ✅ | Active ✅ |
| react-beautiful-dnd | 30kB | ✅ | ✅ | ❌ | Deprecated ⚠️ |
| react-dnd | 20kB | ⚠️ | ⚠️ | ✅ | Active |
| @hello-pangea/dnd | 30kB | ✅ | ✅ | ❌ | rbd fork ✅ |
| Native HTML5 | 0 | ❌ | ❌ | N/A | Limited |

**Raccomandazione:** dnd-kit (moderno, performante, accessibile)

## §2. DND-KIT SETUP

### 2.1 Installation & Providers

```bash
# Install dnd-kit core and utilities
npm install @dnd-kit/core @dnd-kit/sortable @dnd-kit/utilities
```

```typescript
// lib/dnd/config.ts
import type { Active, Over, UniqueIdentifier } from '@dnd-kit/core';
import {
  KeyboardSensor,
  PointerSensor,
  useSensor,
  useSensors,
  DndContextProps,
} from '@dnd-kit/core';
import {
  sortableKeyboardCoordinates,
  horizontalListSortingStrategy,
  verticalListSortingStrategy,
} from '@dnd-kit/sortable';

export interface DragData {
  type: string;
  [key: string]: any;
}

export interface DropData {
  accepts: string[];
  [key: string]: any;
}

// Sensor configuration
export const useDragAndDropSensors = () => {
  const sensors = useSensors(
    // Pointer sensor for mouse/touch
    useSensor(PointerSensor, {
      activationConstraint: {
        distance: 8, // 8px movement required before drag starts
      },
    }),
    // Keyboard sensor for accessibility
    useSensor(KeyboardSensor, {
      coordinateGetter: sortableKeyboardCoordinates,
    })
  );

  return sensors;
};

// DndContext default props
export const defaultDndContextProps: Partial<DndContextProps> = {
  autoScroll: true,
  collisionDetection: closestCorners,
  measuring: {
    droppable: {
      strategy: MeasuringStrategy.Always,
    },
  },
};

// Announcements for screen readers
export const getScreenReaderInstructions = () => ({
  draggable: `
    To pick up a draggable item, press space or enter.
    While dragging, use the arrow keys to move the item.
    Press space or enter again to drop the item in its new position, or press escape to cancel.
  `,
});

export const getScreenReaderAnnouncement = (
  active: Active | null,
  over: Over | null,
  activeId: UniqueIdentifier | null,
  overId: UniqueIdentifier | null
): string => {
  if (!active) return '';

  const action = active.data.current?.type === 'sortable' ? 'sorting' : 'dragging';
  const activeItem = activeId ? `Item ${activeId}` : 'an item';
  const overItem = overId ? `over item ${overId}` : '';

  if (!over) {
    return `${activeItem} is being ${action}.`;
  }

  if (over.data.current?.type === 'droppable') {
    return `${activeItem} was dropped into a drop zone.`;
  }

  return `${activeItem} was moved ${overItem}.`;
};
```

```typescript
// providers/DndProvider.tsx
'use client';

import React, { createContext, useContext, useCallback } from 'react';
import {
  DndContext,
  DragEndEvent,
  DragStartEvent,
  DragOverlay,
  defaultDropAnimation,
  DropAnimation,
  Active,
  Over,
  UniqueIdentifier,
  pointerWithin,
  rectIntersection,
} from '@dnd-kit/core';
import {
  SortableContext,
  rectSortingStrategy,
  arrayMove,
} from '@dnd-kit/sortable';

import { useDragAndDropSensors } from '@/lib/dnd/config';

interface DndProviderContextValue {
  activeId: UniqueIdentifier | null;
  overId: UniqueIdentifier | null;
  active: Active | null;
  over: Over | null;
  handleDragStart: (event: DragStartEvent) => void;
  handleDragEnd: (event: DragEndEvent) => void;
  handleDragCancel: () => void;
}

const DndProviderContext = createContext<DndProviderContextValue | undefined>(undefined);

interface DndProviderProps {
  children: React.ReactNode;
  onDragStart?: (event: DragStartEvent) => void;
  onDragEnd?: (event: DragEndEvent) => void;
  onDragCancel?: () => void;
}

export function DndProvider({
  children,
  onDragStart,
  onDragEnd,
  onDragCancel,
}: DndProviderProps) {
  const [activeId, setActiveId] = React.useState<UniqueIdentifier | null>(null);
  const [overId, setOverId] = React.useState<UniqueIdentifier | null>(null);
  const [active, setActive] = React.useState<Active | null>(null);
  const [over, setOver] = React.useState<Over | null>(null);

  const sensors = useDragAndDropSensors();

  const handleDragStart = useCallback((event: DragStartEvent) => {
    setActiveId(event.active.id);
    setActive(event.active);
    onDragStart?.(event);
  }, [onDragStart]);

  const handleDragEnd = useCallback((event: DragEndEvent) => {
    const { active, over } = event;

    setActiveId(null);
    setOverId(null);
    setActive(null);
    setOver(null);

    if (over && active.id !== over.id) {
      onDragEnd?.(event);
    }
  }, [onDragEnd]);

  const handleDragCancel = useCallback(() => {
    setActiveId(null);
    setOverId(null);
    setActive(null);
    setOver(null);
    onDragCancel?.();
  }, [onDragCancel]);

  const dropAnimation: DropAnimation = {
    ...defaultDropAnimation,
    dragSourceOpacity: 0.5,
  };

  const value: DndProviderContextValue = {
    activeId,
    overId,
    active,
    over,
    handleDragStart,
    handleDragEnd,
    handleDragCancel,
  };

  return (
    <DndProviderContext.Provider value={value}>
      <DndContext
        sensors={sensors}
        onDragStart={handleDragStart}
        onDragEnd={handleDragEnd}
        onDragCancel={handleDragCancel}
        collisionDetection={pointerWithin}
        autoScroll={{ enabled: true, threshold: { x: 0.1, y: 0.1 } }}
      >
        {children}
        <DragOverlay dropAnimation={dropAnimation}>
          {active ? (
            <div className="opacity-80 rotate-3">
              {active.data.current?.renderDragOverlay?.(active) || (
                <div className="bg-white border rounded-lg shadow-lg p-4">
                  {active.data.current?.title || 'Dragging...'}
                </div>
              )}
            </div>
          ) : null}
        </DragOverlay>
      </DndContext>
    </DndProviderContext.Provider>
  );
}

export function useDndContext() {
  const context = useContext(DndProviderContext);
  if (context === undefined) {
    throw new Error('useDndContext must be used within a DndProvider');
  }
  return context;
}
```

### 2.2 Basic Draggable

```typescript
// components/dnd/Draggable.tsx
'use client';

import React, { useMemo } from 'react';
import { useDraggable } from '@dnd-kit/core';
import { CSS } from '@dnd-kit/utilities';
import { cn } from '@/lib/utils';
import { GripVertical } from 'lucide-react';

interface DraggableProps {
  id: string;
  children: React.ReactNode;
  data?: Record<string, any>;
  disabled?: boolean;
  className?: string;
  handleClassName?: string;
  showHandle?: boolean;
  as?: keyof JSX.IntrinsicElements;
  style?: React.CSSProperties;
  onDragStart?: () => void;
  onDragEnd?: () => void;
}

export function Draggable({
  id,
  children,
  data = {},
  disabled = false,
  className,
  handleClassName,
  showHandle = true,
  as: Component = 'div',
  style,
  onDragStart,
  onDragEnd,
}: DraggableProps) {
  const {
    attributes,
    listeners,
    setNodeRef,
    transform,
    isDragging,
    setActivatorNodeRef,
  } = useDraggable({
    id,
    data: {
      type: 'draggable',
      ...data,
    },
    disabled,
  });

  const draggableStyle = useMemo(() => {
    const baseStyle = style || {};
    if (transform) {
      return {
        ...baseStyle,
        transform: CSS.Translate.toString(transform),
        zIndex: isDragging ? 1000 : 'auto',
      };
    }
    return baseStyle;
  }, [transform, isDragging, style]);

  const handleRef = useMemo(() => {
    return (node: HTMLElement | null) => {
      setActivatorNodeRef(node);
    };
  }, [setActivatorNodeRef]);

  React.useEffect(() => {
    if (isDragging && onDragStart) {
      onDragStart();
    }
    if (!isDragging && onDragEnd) {
      onDragEnd();
    }
  }, [isDragging, onDragStart, onDragEnd]);

  return (
    <Component
      ref={setNodeRef}
      style={draggableStyle}
      className={cn(
        'relative transition-all duration-200',
        isDragging && 'opacity-50',
        disabled && 'opacity-50 cursor-not-allowed',
        className
      )}
      {...attributes}
    >
      {showHandle && (
        <button
          ref={handleRef}
          type="button"
          className={cn(
            'absolute left-0 top-1/2 -translate-y-1/2 -translate-x-full',
            'p-2 opacity-0 group-hover:opacity-100 transition-opacity',
            'focus:opacity-100 focus:outline-none',
            'cursor-grab active:cursor-grabbing',
            disabled && 'cursor-not-allowed',
            handleClassName
          )}
          {...listeners}
          aria-label={`Drag item ${id}`}
          disabled={disabled}
        >
          <GripVertical className="h-4 w-4 text-gray-400" />
        </button>
      )}
      {children}
    </Component>
  );
}

// Draggable with custom drag overlay
interface DraggableWithOverlayProps extends Omit<DraggableProps, 'children'> {
  renderContent: (props: { isDragging: boolean }) => React.ReactNode;
  renderDragOverlay?: (props: { id: string; data: any }) => React.ReactNode;
}

export function DraggableWithOverlay({
  renderContent,
  renderDragOverlay,
  ...props
}: DraggableWithOverlayProps) {
  const { data, ...restProps } = props;
  
  return (
    <Draggable
      {...restProps}
      data={{
        ...data,
        renderDragOverlay: () => renderDragOverlay?.({
          id: props.id,
          data: data || {},
        }) || null,
      }}
    >
      {renderContent({ isDragging: false })}
    </Draggable>
  );
}
```

### 2.3 Basic Droppable

```typescript
// components/dnd/Droppable.tsx
'use client';

import React, { useMemo } from 'react';
import { useDroppable } from '@dnd-kit/core';
import { cn } from '@/lib/utils';

interface DroppableProps {
  id: string;
  children: React.ReactNode;
  data?: Record<string, any>;
  accepts?: string[];
  className?: string;
  activeClassName?: string;
  overClassName?: string;
  disabled?: boolean;
  as?: keyof JSX.IntrinsicElements;
  style?: React.CSSProperties;
}

export function Droppable({
  id,
  children,
  data = {},
  accepts = ['draggable', 'sortable'],
  className,
  activeClassName = 'bg-blue-50 border-blue-300',
  overClassName = 'bg-green-50 border-green-300 ring-2 ring-green-200',
  disabled = false,
  as: Component = 'div',
  style,
}: DroppableProps) {
  const { isOver, setNodeRef, active } = useDroppable({
    id,
    data: {
      type: 'droppable',
      accepts,
      ...data,
    },
    disabled,
  });

  const isValidDrop = useMemo(() => {
    if (!active || !active.data.current) return false;
    
    const activeType = active.data.current.type;
    const activeData = active.data.current;
    
    // Check if active item type is accepted
    if (accepts.includes(activeType)) {
      // Additional validation if provided
      if (data.validateDrop) {
        return data.validateDrop(activeData);
      }
      return true;
    }
    
    return false;
  }, [active, accepts, data]);

  const droppableStyle = useMemo(() => {
    const baseStyle = style || {};
    return {
      ...baseStyle,
      transition: 'background-color 0.2s ease, border-color 0.2s ease',
    };
  }, [style]);

  return (
    <Component
      ref={setNodeRef}
      style={droppableStyle}
      className={cn(
        'border-2 border-dashed border-gray-300 rounded-lg p-4 transition-all',
        active && isValidDrop && activeClassName,
        isOver && isValidDrop && overClassName,
        disabled && 'opacity-50 cursor-not-allowed',
        className
      )}
      aria-label={`Drop zone ${id}`}
      role="region"
      aria-describedby={isOver && isValidDrop ? `drop-instructions-${id}` : undefined}
    >
      {children}
      
      {isOver && isValidDrop && (
        <div
          id={`drop-instructions-${id}`}
          className="sr-only"
          aria-live="assertive"
        >
          Drop item here
        </div>
      )}
    </Component>
  );
}

// Droppable with placeholder for empty state
interface DroppableWithPlaceholderProps extends DroppableProps {
  placeholder?: React.ReactNode;
  isEmpty?: boolean;
}

export function DroppableWithPlaceholder({
  placeholder,
  isEmpty = false,
  children,
  ...props
}: DroppableWithPlaceholderProps) {
  return (
    <Droppable {...props}>
      {isEmpty && placeholder ? (
        <div className="text-center py-8 text-gray-500">
          {placeholder}
        </div>
      ) : (
        children
      )}
    </Droppable>
  );
}
```

## §3. SORTABLE LIST

### 3.1 Sortable Context

```typescript
// components/dnd/SortableList.tsx
'use client';

import React, { useMemo } from 'react';
import {
  SortableContext,
  sortableKeyboardCoordinates,
  verticalListSortingStrategy,
  horizontalListSortingStrategy,
  rectSortingStrategy,
} from '@dnd-kit/sortable';
import { DndContext, DragEndEvent, closestCenter } from '@dnd-kit/core';
import { useDragAndDropSensors } from '@/lib/dnd/config';
import { cn } from '@/lib/utils';

interface SortableListProps<T extends { id: string }> {
  items: T[];
  onReorder: (items: T[]) => void;
  children: (item: T, index: number) => React.ReactNode;
  className?: string;
  itemClassName?: string;
  direction?: 'vertical' | 'horizontal' | 'grid';
  disabled?: boolean;
  renderPlaceholder?: (props: { id: string; index: number }) => React.ReactNode;
  getItemId?: (item: T) => string;
}

export function SortableList<T extends { id: string }>({
  items,
  onReorder,
  children,
  className,
  itemClassName,
  direction = 'vertical',
  disabled = false,
  renderPlaceholder,
  getItemId = (item) => item.id,
}: SortableListProps<T>) {
  const sensors = useDragAndDropSensors();
  const itemIds = useMemo(() => items.map(getItemId), [items, getItemId]);

  const strategy = useMemo(() => {
    switch (direction) {
      case 'horizontal':
        return horizontalListSortingStrategy;
      case 'grid':
        return rectSortingStrategy;
      default:
        return verticalListSortingStrategy;
    }
  }, [direction]);

  const handleDragEnd = (event: DragEndEvent) => {
    const { active, over } = event;

    if (over && active.id !== over.id) {
      const oldIndex = itemIds.indexOf(active.id as string);
      const newIndex = itemIds.indexOf(over.id as string);

      if (oldIndex !== -1 && newIndex !== -1) {
        const newItems = arrayMove(items, oldIndex, newIndex);
        onReorder(newItems);
      }
    }
  };

  return (
    <DndContext
      sensors={sensors}
      collisionDetection={closestCenter}
      onDragEnd={handleDragEnd}
    >
      <SortableContext
        items={itemIds}
        strategy={strategy}
        disabled={disabled}
      >
        <div
          className={cn(
            direction === 'vertical' && 'space-y-2',
            direction === 'horizontal' && 'flex gap-2 overflow-x-auto',
            direction === 'grid' && 'grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4',
            className
          )}
        >
          {items.map((item, index) => (
            <React.Fragment key={getItemId(item)}>
              {children(item, index)}
            </React.Fragment>
          ))}
        </div>
      </SortableContext>
    </DndContext>
  );
}

// Helper function to move items in array
export function arrayMove<T>(array: T[], from: number, to: number): T[] {
  const newArray = [...array];
  const [movedItem] = newArray.splice(from, 1);
  newArray.splice(to, 0, movedItem);
  return newArray;
}

// Sortable list with drag handle
interface SortableListWithHandleProps<T extends { id: string }>
  extends Omit<SortableListProps<T>, 'children'> {
  renderItem: (item: T, handlers: {
    handleRef: (node: HTMLElement | null) => void;
    isDragging: boolean;
  }) => React.ReactNode;
}

export function SortableListWithHandle<T extends { id: string }>({
  renderItem,
  ...props
}: SortableListWithHandleProps<T>) {
  return (
    <SortableList
      {...props}
      children={(item, index) => (
        <SortableItem
          id={props.getItemId?.(item) || item.id}
          index={index}
          disabled={props.disabled}
        >
          {({ handleRef, isDragging }) => renderItem(item, { handleRef, isDragging })}
        </SortableItem>
      )}
    />
  );
}
```

### 3.2 Sortable Item

```typescript
// components/dnd/SortableItem.tsx
'use client';

import React, { useMemo } from 'react';
import { useSortable } from '@dnd-kit/sortable';
import { CSS } from '@dnd-kit/utilities';
import { cn } from '@/lib/utils';
import { GripVertical } from 'lucide-react';

interface SortableItemProps {
  id: string;
  index?: number;
  children: (props: {
    handleRef: (node: HTMLElement | null) => void;
    isDragging: boolean;
    attributes: any;
    listeners: any;
  }) => React.ReactNode;
  data?: Record<string, any>;
  disabled?: boolean;
  className?: string;
  style?: React.CSSProperties;
  handleClassName?: string;
  showHandle?: boolean;
}

export function SortableItem({
  id,
  index,
  children,
  data = {},
  disabled = false,
  className,
  style,
  handleClassName,
  showHandle = true,
}: SortableItemProps) {
  const {
    attributes,
    listeners,
    setNodeRef,
    transform,
    transition,
    isDragging,
    setActivatorNodeRef,
  } = useSortable({
    id,
    data: {
      type: 'sortable',
      index,
      ...data,
    },
    disabled,
  });

  const sortableStyle = useMemo(() => {
    const baseStyle = style || {};
    return {
      ...baseStyle,
      transform: CSS.Transform.toString(transform),
      transition,
      opacity: isDragging ? 0.5 : 1,
      zIndex: isDragging ? 1000 : 'auto',
    };
  }, [transform, transition, isDragging, style]);

  const handleRef = useMemo(() => {
    return (node: HTMLElement | null) => {
      setActivatorNodeRef(node);
    };
  }, [setActivatorNodeRef]);

  return (
    <div
      ref={setNodeRef}
      style={sortableStyle}
      className={cn(
        'relative transition-all duration-200',
        isDragging && 'shadow-lg',
        disabled && 'opacity-50 cursor-not-allowed',
        className
      )}
    >
      {showHandle && (
        <button
          ref={handleRef}
          type="button"
          className={cn(
            'absolute left-0 top-1/2 -translate-y-1/2 -translate-x-full',
            'p-2 opacity-0 group-hover:opacity-100 transition-opacity',
            'focus:opacity-100 focus:outline-none',
            'cursor-grab active:cursor-grabbing',
            disabled && 'cursor-not-allowed',
            handleClassName
          )}
          {...attributes}
          {...listeners}
          aria-label={`Drag to reorder item ${index !== undefined ? index + 1 : ''}`}
          disabled={disabled}
        >
          <GripVertical className="h-4 w-4 text-gray-400" />
        </button>
      )}
      {children({ handleRef, isDragging, attributes, listeners })}
    </div>
  );
}

// Sortable item with card styling
interface SortableCardProps extends Omit<SortableItemProps, 'children'> {
  children: React.ReactNode;
  onClick?: () => void;
}

export function SortableCard({
  children,
  onClick,
  ...props
}: SortableCardProps) {
  return (
    <SortableItem {...props}>
      {({ handleRef, isDragging, attributes, listeners }) => (
        <div
          className={cn(
            'bg-white border rounded-lg p-4',
            'hover:shadow-md transition-shadow',
            isDragging && 'shadow-lg',
            onClick && 'cursor-pointer hover:border-gray-400'
          )}
          onClick={onClick}
          role={onClick ? 'button' : 'listitem'}
          tabIndex={onClick ? 0 : -1}
          onKeyDown={(e) => {
            if (onClick && (e.key === 'Enter' || e.key === ' ')) {
              e.preventDefault();
              onClick();
            }
          }}
        >
          <div className="flex items-start gap-3">
            <button
              ref={handleRef}
              type="button"
              className={cn(
                'p-1 cursor-grab active:cursor-grabbing',
                'opacity-60 hover:opacity-100 transition-opacity',
                props.disabled && 'cursor-not-allowed'
              )}
              {...attributes}
              {...listeners}
              aria-label="Drag to reorder"
              disabled={props.disabled}
            >
              <GripVertical className="h-4 w-4 text-gray-400" />
            </button>
            <div className="flex-1">{children}</div>
          </div>
        </div>
      )}
    </SortableItem>
  );
}
```

### 3.3 Persist Order

```typescript
// hooks/useSortableList.ts
import { useState, useCallback, useRef } from 'react';
import { DragEndEvent } from '@dnd-kit/core';
import { arrayMove } from '@/components/dnd/SortableList';

interface UseSortableListOptions<T extends { id: string; order?: number }> {
  initialItems: T[];
  onReorder?: (items: T[]) => Promise<void>;
  optimisticUpdates?: boolean;
  getItemId?: (item: T) => string;
}

export function useSortableList<T extends { id: string; order?: number }>({
  initialItems,
  onReorder,
  optimisticUpdates = true,
  getItemId = (item) => item.id,
}: UseSortableListOptions<T>) {
  const [items, setItems] = useState<T[]>(initialItems);
  const [isReordering, setIsReordering] = useState(false);
  const previousItemsRef = useRef<T[]>(initialItems);

  // Update order numbers for persistence
  const updateItemOrders = useCallback((items: T[]): T[] => {
    return items.map((item, index) => ({
      ...item,
      order: index + 1,
    }));
  }, []);

  const handleDragEnd = useCallback(async (event: DragEndEvent) => {
    const { active, over } = event;

    if (!over || active.id === over.id) {
      return;
    }

    const oldIndex = items.findIndex(item => getItemId(item) === active.id);
    const newIndex = items.findIndex(item => getItemId(item) === over.id);

    if (oldIndex === -1 || newIndex === -1) {
      return;
    }

    setIsReordering(true);
    previousItemsRef.current = items;

    // Create new items array
    let newItems = arrayMove(items, oldIndex, newIndex);
    newItems = updateItemOrders(newItems);

    // Optimistic update
    if (optimisticUpdates) {
      setItems(newItems);
    }

    try {
      // Call onReorder callback if provided
      if (onReorder) {
        await onReorder(newItems);
      }

      // If not optimistic, update after success
      if (!optimisticUpdates) {
        setItems(newItems);
      }
    } catch (error) {
      console.error('Failed to reorder items:', error);
      
      // Revert on error
      if (optimisticUpdates) {
        setItems(previousItemsRef.current);
      }
      
      // Show error to user
      alert('Failed to save new order. Please try again.');
    } finally {
      setIsReordering(false);
    }
  }, [items, onReorder, optimisticUpdates, getItemId, updateItemOrders]);

  const addItem = useCallback((item: T) => {
    const newItems = [...items, { ...item, order: items.length + 1 }];
    setItems(newItems);
    return newItems;
  }, [items]);

  const removeItem = useCallback((id: string) => {
    const newItems = items.filter(item => getItemId(item) !== id);
    const reorderedItems = updateItemOrders(newItems);
    setItems(reorderedItems);
    return reorderedItems;
  }, [items, getItemId, updateItemOrders]);

  const updateItem = useCallback((id: string, updates: Partial<T>) => {
    const newItems = items.map(item => 
      getItemId(item) === id ? { ...item, ...updates } : item
    );
    setItems(newItems);
    return newItems;
  }, [items, getItemId]);

  return {
    items,
    setItems,
    isReordering,
    handleDragEnd,
    addItem,
    removeItem,
    updateItem,
  };
}

// Example usage component
// components/dnd/SortableListExample.tsx
'use client';

import { useState } from 'react';
import { SortableList } from './SortableList';
import { SortableCard } from './SortableItem';
import { useSortableList } from '@/hooks/useSortableList';
import { Button } from '@/components/ui/button';
import { Plus } from 'lucide-react';

interface Task {
  id: string;
  title: string;
  description: string;
  order: number;
}

export function SortableListExample() {
  const initialTasks: Task[] = [
    { id: '1', title: 'Design homepage', description: 'Create wireframes', order: 1 },
    { id: '2', title: 'Implement auth', description: 'Setup NextAuth.js', order: 2 },
    { id: '3', title: 'Database schema', description: 'Design Prisma schema', order: 3 },
  ];

  const {
    items: tasks,
    handleDragEnd,
    addItem,
    removeItem,
    isReordering,
  } = useSortableList<Task>({
    initialItems: initialTasks,
    onReorder: async (newTasks) => {
      // Simulate API call
      console.log('Saving new order:', newTasks);
      await new Promise(resolve => setTimeout(resolve, 500));
    },
  });

  const handleAddTask = () => {
    const newTask: Task = {
      id: Date.now().toString(),
      title: `New Task ${tasks.length + 1}`,
      description: 'Description',
      order: tasks.length + 1,
    };
    addItem(newTask);
  };

  const handleDeleteTask = (id: string) => {
    if (confirm('Are you sure you want to delete this task?')) {
      removeItem(id);
    }
  };

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <h2 className="text-xl font-semibold">Tasks</h2>
        <Button onClick={handleAddTask} disabled={isReordering} className="gap-2">
          <Plus className="h-4 w-4" />
          Add Task
        </Button>
      </div>

      <div className="space-y-2">
        <SortableList
          items={tasks}
          onReorder={(newTasks) => {
            // The useSortableList hook will handle the actual reordering
            // This is just to satisfy the prop type
          }}
          direction="vertical"
          disabled={isReordering}
        >
          {(task) => (
            <SortableCard
              key={task.id}
              id={task.id}
              data={task}
            >
              <div className="flex items-start justify-between">
                <div>
                  <h3 className="font-medium">{task.title}</h3>
                  <p className="text-sm text-gray-600">{task.description}</p>
                  <div className="text-xs text-gray-500 mt-1">Order: {task.order}</div>
                </div>
                <Button
                  variant="ghost"
                  size="sm"
                  onClick={() => handleDeleteTask(task.id)}
                  disabled={isReordering}
                >
                  Delete
                </Button>
              </div>
            </SortableCard>
          )}
        </SortableList>
      </div>

      {isReordering && (
        <div className="text-sm text-gray-500 text-center">
          Saving new order...
        </div>
      )}
    </div>
  );
}
```

## §4. KANBAN BOARD

### 4.1 Kanban Implementation

```typescript
// components/kanban/types.ts
export interface KanbanItem {
  id: string;
  title: string;
  description?: string;
  columnId: string;
  order: number;
  color?: string;
  tags?: string[];
  assignee?: {
    id: string;
    name: string;
    avatar?: string;
  };
  dueDate?: Date;
}

export interface KanbanColumn {
  id: string;
  title: string;
  color: string;
  itemIds: string[]; // Ordered list of item IDs
  order: number;
  maxItems?: number;
}

export interface KanbanBoard {
  columns: KanbanColumn[];
  items: Record<string, KanbanItem>;
}
```

```typescript
// components/kanban/KanbanBoard.tsx
'use client';

import React, { useState, useMemo, useCallback } from 'react';
import { DndContext, DragEndEvent, DragOverEvent, DragStartEvent } from '@dnd-kit/core';
import { SortableContext, horizontalListSortingStrategy } from '@dnd-kit/sortable';
import { useDragAndDropSensors } from '@/lib/dnd/config';
import { KanbanColumn } from './KanbanColumn';
import { KanbanCard } from './KanbanCard';
import { KanbanItem, KanbanColumn as KanbanColumnType } from './types';
import { arrayMove } from '@/components/dnd/SortableList';
import { cn } from '@/lib/utils';
import { Plus } from 'lucide-react';
import { Button } from '@/components/ui/button';

interface KanbanBoardProps {
  initialColumns: KanbanColumnType[];
  initialItems: KanbanItem[];
  onColumnReorder?: (columns: KanbanColumnType[]) => void;
  onItemMove?: (itemId: string, sourceColumnId: string, destinationColumnId: string, newOrder: number) => void;
  onColumnAdd?: (title: string) => void;
  onColumnUpdate?: (columnId: string, updates: Partial<KanbanColumnType>) => void;
  onColumnDelete?: (columnId: string) => void;
  onItemAdd?: (columnId: string, title: string) => void;
  onItemUpdate?: (itemId: string, updates: Partial<KanbanItem>) => void;
  onItemDelete?: (itemId: string) => void;
  className?: string;
  readOnly?: boolean;
}

export function KanbanBoard({
  initialColumns,
  initialItems,
  onColumnReorder,
  onItemMove,
  onColumnAdd,
  onColumnUpdate,
  onColumnDelete,
  onItemAdd,
  onItemUpdate,
  onItemDelete,
  className,
  readOnly = false,
}: KanbanBoardProps) {
  const [columns, setColumns] = useState<KanbanColumnType[]>(initialColumns);
  const [items, setItems] = useState<Record<string, KanbanItem>>(
    initialItems.reduce((acc, item) => ({ ...acc, [item.id]: item }), {})
  );
  const [activeId, setActiveId] = useState<string | null>(null);
  const [activeColumnId, setActiveColumnId] = useState<string | null>(null);

  const sensors = useDragAndDropSensors();

  // Group items by column
  const itemsByColumn = useMemo(() => {
    const grouped: Record<string, KanbanItem[]> = {};
    columns.forEach(column => {
      grouped[column.id] = column.itemIds
        .map(itemId => items[itemId])
        .filter(Boolean)
        .sort((a, b) => a.order - b.order);
    });
    return grouped;
  }, [columns, items]);

  const columnIds = useMemo(() => columns.map(col => col.id), [columns]);

  const handleDragStart = useCallback((event: DragStartEvent) => {
    const { active } = event;
    setActiveId(active.id as string);
    
    // Check if dragging a column
    if (active.data.current?.type === 'column') {
      setActiveColumnId(active.id as string);
    }
  }, []);

  const handleDragOver = useCallback((event: DragOverEvent) => {
    const { active, over } = event;
    if (!over) return;

    const activeId = active.id as string;
    const overId = over.id as string;

    // Find the containers
    const activeContainer = findContainer(activeId);
    const overContainer = findContainer(overId);

    if (!activeContainer || !overContainer || activeContainer === overContainer) {
      return;
    }

    // Moving item between columns
    setColumns(prev => {
      const activeColumn = prev.find(col => col.id === activeContainer);
      const overColumn = prev.find(col => col.id === overContainer);

      if (!activeColumn || !overColumn) return prev;

      // Remove from active column
      const newActiveColumn = {
        ...activeColumn,
        itemIds: activeColumn.itemIds.filter(id => id !== activeId),
      };

      // Add to over column at the end
      const newOverColumn = {
        ...overColumn,
        itemIds: [...overColumn.itemIds, activeId],
      };

      return prev.map(column => {
        if (column.id === activeContainer) return newActiveColumn;
        if (column.id === overContainer) return newOverColumn;
        return column;
      });
    });
  }, [columns]);

  const handleDragEnd = useCallback((event: DragEndEvent) => {
    const { active, over } = event;
    if (!over) {
      setActiveId(null);
      setActiveColumnId(null);
      return;
    }

    const activeId = active.id as string;
    const overId = over.id as string;

    // Handle column reordering
    if (active.data.current?.type === 'column') {
      if (activeId !== overId) {
        const oldIndex = columns.findIndex(col => col.id === activeId);
        const newIndex = columns.findIndex(col => col.id === overId);

        if (oldIndex !== -1 && newIndex !== -1) {
          const newColumns = arrayMove(columns, oldIndex, newIndex);
          setColumns(newColumns);
          onColumnReorder?.(newColumns);
        }
      }
    }
    // Handle item reordering within column
    else if (active.data.current?.type === 'item') {
      const container = findContainer(activeId);
      if (!container) return;

      const column = columns.find(col => col.id === container);
      if (!column) return;

      const oldIndex = column.itemIds.indexOf(activeId);
      const newIndex = column.itemIds.indexOf(overId);

      if (oldIndex !== -1 && newIndex !== -1 && oldIndex !== newIndex) {
        const newItemIds = arrayMove(column.itemIds, oldIndex, newIndex);
        const newColumns = columns.map(col => 
          col.id === container ? { ...col, itemIds: newItemIds } : col
        );
        
        setColumns(newColumns);
        
        // Update item order
        const newItems = { ...items };
        newItemIds.forEach((itemId, index) => {
          if (newItems[itemId]) {
            newItems[itemId] = { ...newItems[itemId], order: index + 1 };
          }
        });
        setItems(newItems);

        // Call onItemMove callback
        onItemMove?.(activeId, container, container, newIndex + 1);
      }
    }

    setActiveId(null);
    setActiveColumnId(null);
  }, [columns, items, onColumnReorder, onItemMove]);

  const findContainer = useCallback((id: string): string | null => {
    // If it's a column
    if (columns.some(col => col.id === id)) {
      return id;
    }

    // Find which column contains this item
    for (const column of columns) {
      if (column.itemIds.includes(id)) {
        return column.id;
      }
    }

    return null;
  }, [columns]);

  const handleAddColumn = useCallback(() => {
    if (readOnly) return;
    
    const title = prompt('Enter column title:');
    if (!title) return;

    const newColumn: KanbanColumnType = {
      id: Date.now().toString(),
      title,
      color: '#3b82f6',
      itemIds: [],
      order: columns.length + 1,
    };

    const newColumns = [...columns, newColumn];
    setColumns(newColumns);
    onColumnAdd?.(title);
  }, [columns, readOnly, onColumnAdd]);

  const handleAddItem = useCallback((columnId: string) => {
    if (readOnly) return;
    
    const title = prompt('Enter item title:');
    if (!title) return;

    const newItem: KanbanItem = {
      id: Date.now().toString(),
      title,
      columnId,
      order: itemsByColumn[columnId]?.length + 1 || 1,
    };

    const newItems = { ...items, [newItem.id]: newItem };
    setItems(newItems);

    const newColumns = columns.map(col => 
      col.id === columnId 
        ? { ...col, itemIds: [...col.itemIds, newItem.id] }
        : col
    );
    setColumns(newColumns);

    onItemAdd?.(columnId, title);
  }, [columns, items, itemsByColumn, readOnly, onItemAdd]);

  const handleUpdateItem = useCallback((itemId: string, updates: Partial<KanbanItem>) => {
    if (readOnly) return;
    
    setItems(prev => ({
      ...prev,
      [itemId]: { ...prev[itemId], ...updates },
    }));
    onItemUpdate?.(itemId, updates);
  }, [readOnly, onItemUpdate]);

  const handleDeleteItem = useCallback((itemId: string, columnId: string) => {
    if (readOnly) return;
    
    if (confirm('Are you sure you want to delete this item?')) {
      const newItems = { ...items };
      delete newItems[itemId];
      setItems(newItems);

      const newColumns = columns.map(col => 
        col.id === columnId
          ? { ...col, itemIds: col.itemIds.filter(id => id !== itemId) }
          : col
      );
      setColumns(newColumns);

      onItemDelete?.(itemId);
    }
  }, [columns, items, readOnly, onItemDelete]);

  return (
    <div className={cn('h-full', className)}>
      <DndContext
        sensors={sensors}
        onDragStart={handleDragStart}
        onDragOver={handleDragOver}
        onDragEnd={handleDragEnd}
        collisionDetection={closestCorners}
      >
        <SortableContext
          items={columnIds}
          strategy={horizontalListSortingStrategy}
        >
          <div className="flex gap-4 pb-4 overflow-x-auto">
            {columns.map(column => (
              <KanbanColumn
                key={column.id}
                column={column}
                items={itemsByColumn[column.id] || []}
                isDragging={activeColumnId === column.id}
                onAddItem={() => handleAddItem(column.id)}
                onUpdateItem={(itemId, updates) => handleUpdateItem(itemId, updates)}
                onDeleteItem={(itemId) => handleDeleteItem(itemId, column.id)}
                onUpdateColumn={(updates) => onColumnUpdate?.(column.id, updates)}
                onDeleteColumn={() => onColumnDelete?.(column.id)}
                readOnly={readOnly}
              />
            ))}
            
            {!readOnly && (
              <div className="min-w-72">
                <Button
                  onClick={handleAddColumn}
                  variant="outline"
                  className="w-full h-full min-h-[200px] border-dashed"
                >
                  <Plus className="h-8 w-8 text-gray-400" />
                  <span className="mt-2">Add Column</span>
                </Button>
              </div>
            )}
          </div>
        </SortableContext>
      </DndContext>
    </div>
  );
}
```

```typescript
// components/kanban/KanbanColumn.tsx
'use client';

import React, { useMemo } from 'react';
import { useDroppable, useSortable } from '@dnd-kit/core';
import { SortableContext, verticalListSortingStrategy } from '@dnd-kit/sortable';
import { CSS } from '@dnd-kit/utilities';
import { KanbanCard } from './KanbanCard';
import { KanbanItem, KanbanColumn as KanbanColumnType } from './types';
import { cn } from '@/lib/utils';
import { GripVertical, Plus, MoreVertical, Trash2 } from 'lucide-react';
import { Button } from '@/components/ui/button';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';

interface KanbanColumnProps {
  column: KanbanColumnType;
  items: KanbanItem[];
  isDragging?: boolean;
  onAddItem: () => void;
  onUpdateItem: (itemId: string, updates: Partial<KanbanItem>) => void;
  onDeleteItem: (itemId: string) => void;
  onUpdateColumn: (updates: Partial<KanbanColumnType>) => void;
  onDeleteColumn: () => void;
  readOnly?: boolean;
}

export function KanbanColumn({
  column,
  items,
  isDragging = false,
  onAddItem,
  onUpdateItem,
  onDeleteItem,
  onUpdateColumn,
  onDeleteColumn,
  readOnly = false,
}: KanbanColumnProps) {
  const {
    attributes,
    listeners,
    setNodeRef,
    transform,
    transition,
    isSorting,
  } = useSortable({
    id: column.id,
    data: {
      type: 'column',
      column,
    },
    disabled: readOnly,
  });

  const { setNodeRef: setDroppableRef, isOver } = useDroppable({
    id: column.id,
    data: {
      type: 'droppable',
      accepts: ['item'],
    },
    disabled: readOnly,
  });

  const itemIds = useMemo(() => items.map(item => item.id), [items]);

  const style = useMemo(() => ({
    transform: CSS.Transform.toString(transform),
    transition,
  }), [transform, transition]);

  const handleTitleChange = () => {
    if (readOnly) return;
    const newTitle = prompt('Enter new column title:', column.title);
    if (newTitle && newTitle !== column.title) {
      onUpdateColumn({ title: newTitle });
    }
  };

  const handleColorChange = () => {
    if (readOnly) return;
    const newColor = prompt('Enter hex color:', column.color);
    if (newColor && /^#[0-9A-F]{6}$/i.test(newColor)) {
      onUpdateColumn({ color: newColor });
    }
  };

  return (
    <div
      ref={setNodeRef}
      style={style}
      className={cn(
        'flex-shrink-0 w-72',
        isSorting && 'opacity-50'
      )}
    >
      <div
        ref={setDroppableRef}
        className={cn(
          'bg-gray-50 rounded-lg p-4 h-full flex flex-col',
          isOver && 'ring-2 ring-blue-500 ring-opacity-50',
          isDragging && 'opacity-50'
        )}
      >
        {/* Column Header */}
        <div className="flex items-center justify-between mb-4">
          <div className="flex items-center gap-2 flex-1 min-w-0">
            {!readOnly && (
              <button
                type="button"
                className="cursor-grab active:cursor-grabbing p-1"
                {...attributes}
                {...listeners}
                aria-label="Drag column"
              >
                <GripVertical className="h-4 w-4 text-gray-400" />
              </button>
            )}
            
            <button
              type="button"
              onClick={handleTitleChange}
              disabled={readOnly}
              className={cn(
                'text-lg font-semibold truncate',
                !readOnly && 'hover:text-blue-600 focus:outline-none'
              )}
            >
              {column.title}
            </button>
            
            <div
              className="h-3 w-3 rounded-full flex-shrink-0"
              style={{ backgroundColor: column.color }}
            />
            
            <span className="text-sm text-gray-500 bg-gray-200 px-2 py-1 rounded-full">
              {items.length}
              {column.maxItems && `/${column.maxItems}`}
            </span>
          </div>
          
          {!readOnly && (
            <DropdownMenu>
              <DropdownMenuTrigger asChild>
                <Button variant="ghost" size="icon">
                  <MoreVertical className="h-4 w-4" />
                </Button>
              </DropdownMenuTrigger>
              <DropdownMenuContent align="end">
                <DropdownMenuItem onClick={handleTitleChange}>
                  Rename
                </DropdownMenuItem>
                <DropdownMenuItem onClick={handleColorChange}>
                  Change Color
                </DropdownMenuItem>
                <DropdownMenuItem
                  onClick={onDeleteColumn}
                  className="text-red-600"
                >
                  <Trash2 className="h-4 w-4 mr-2" />
                  Delete Column
                </DropdownMenuItem>
              </DropdownMenuContent>
            </DropdownMenu>
          )}
        </div>

        {/* Column Items */}
        <SortableContext
          items={itemIds}
          strategy={verticalListSortingStrategy}
        >
          <div className="flex-1 space-y-3 overflow-y-auto min-h-0">
            {items.map(item => (
              <KanbanCard
                key={item.id}
                item={item}
                onUpdate={(updates) => onUpdateItem(item.id, updates)}
                onDelete={() => onDeleteItem(item.id)}
                readOnly={readOnly}
              />
            ))}
          </div>
        </SortableContext>

        {/* Add Item Button */}
        {!readOnly && (
          <Button
            variant="ghost"
            onClick={onAddItem}
            className="mt-4 w-full border-dashed"
          >
            <Plus className="h-4 w-4 mr-2" />
            Add Item
          </Button>
        )}
      </div>
    </div>
  );
}
```

```typescript
// components/kanban/KanbanCard.tsx
'use client';

import React, { useState } from 'react';
import { useSortable } from '@dnd-kit/sortable';
import { CSS } from '@dnd-kit/utilities';
import { KanbanItem } from './types';
import { cn } from '@/lib/utils';
import { GripVertical, Pencil, Trash2, Calendar, User } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { format } from 'date-fns';

interface KanbanCardProps {
  item: KanbanItem;
  onUpdate: (updates: Partial<KanbanItem>) => void;
  onDelete: () => void;
  readOnly?: boolean;
}

export function KanbanCard({
  item,
  onUpdate,
  onDelete,
  readOnly = false,
}: KanbanCardProps) {
  const [isEditing, setIsEditing] = useState(false);
  const [editedTitle, setEditedTitle] = useState(item.title);
  const [editedDescription, setEditedDescription] = useState(item.description || '');

  const {
    attributes,
    listeners,
    setNodeRef,
    transform,
    transition,
    isDragging,
  } = useSortable({
    id: item.id,
    data: {
      type: 'item',
      item,
    },
    disabled: readOnly,
  });

  const style = useMemo(() => ({
    transform: CSS.Transform.toString(transform),
    transition,
  }), [transform, transition]);

  const handleSave = () => {
    onUpdate({
      title: editedTitle,
      description: editedDescription,
    });
    setIsEditing(false);
  };

  const handleCancel = () => {
    setEditedTitle(item.title);
    setEditedDescription(item.description || '');
    setIsEditing(false);
  };

  const handleTagAdd = () => {
    if (readOnly) return;
    const tag = prompt('Enter tag:');
    if (tag) {
      onUpdate({
        tags: [...(item.tags || []), tag],
      });
    }
  };

  const handleTagRemove = (tagToRemove: string) => {
    onUpdate({
      tags: (item.tags || []).filter(tag => tag !== tagToRemove),
    });
  };

  return (
    <div
      ref={setNodeRef}
      style={style}
      className={cn(
        'bg-white border rounded-lg p-4',
        'hover:shadow-md transition-shadow',
        isDragging && 'shadow-lg rotate-3 opacity-50',
        item.color && 'border-l-4',
        readOnly && 'cursor-default'
      )}
      style={item.color ? { borderLeftColor: item.color } : {}}
    >
      <div className="space-y-3">
        {/* Drag handle and actions */}
        <div className="flex items-start justify-between">
          {!readOnly && (
            <button
              type="button"
              className="cursor-grab active:cursor-grabbing p-1 -ml-2"
              {...attributes}
              {...listeners}
              aria-label="Drag item"
            >
              <GripVertical className="h-4 w-4 text-gray-400" />
            </button>
          )}
          
          <div className="flex-1 min-w-0">
            {isEditing ? (
              <div className="space-y-2">
                <input
                  type="text"
                  value={editedTitle}
                  onChange={(e) => setEditedTitle(e.target.value)}
                  className="w-full border rounded px-2 py-1"
                  autoFocus
                />
                <textarea
                  value={editedDescription}
                  onChange={(e) => setEditedDescription(e.target.value)}
                  className="w-full border rounded px-2 py-1 text-sm"
                  rows={2}
                />
                <div className="flex gap-2">
                  <Button size="sm" onClick={handleSave}>
                    Save
                  </Button>
                  <Button size="sm" variant="outline" onClick={handleCancel}>
                    Cancel
                  </Button>
                </div>
              </div>
            ) : (
              <>
                <h3 className="font-medium truncate">{item.title}</h3>
                {item.description && (
                  <p className="text-sm text-gray-600 mt-1">{item.description}</p>
                )}
              </>
            )}
          </div>
          
          {!readOnly && !isEditing && (
            <div className="flex gap-1">
              <Button
                variant="ghost"
                size="icon"
                onClick={() => setIsEditing(true)}
                className="h-6 w-6"
              >
                <Pencil className="h-3 w-3" />
              </Button>
              <Button
                variant="ghost"
                size="icon"
                onClick={onDelete}
                className="h-6 w-6 text-red-600 hover:text-red-700"
              >
                <Trash2 className="h-3 w-3" />
              </Button>
            </div>
          )}
        </div>

        {/* Tags */}
        {(item.tags && item.tags.length > 0) && (
          <div className="flex flex-wrap gap-1">
            {item.tags.map(tag => (
              <Badge
                key={tag}
                variant="secondary"
                className="text-xs"
              >
                {tag}
                {!readOnly && (
                  <button
                    type="button"
                    onClick={() => handleTagRemove(tag)}
                    className="ml-1 text-gray-500 hover:text-gray-700"
                    aria-label={`Remove tag ${tag}`}
                  >
                    ×
                  </button>
                )}
              </Badge>
            ))}
          </div>
        )}

        {/* Metadata */}
        <div className="flex items-center justify-between text-xs text-gray-500">
          <div className="flex items-center gap-3">
            {item.assignee && (
              <div className="flex items-center gap-1">
                <User className="h-3 w-3" />
                <span>{item.assignee.name}</span>
              </div>
            )}
            
            {item.dueDate && (
              <div className="flex items-center gap-1">
                <Calendar className="h-3 w-3" />
                <span>{format(new Date(item.dueDate), 'MMM d')}</span>
              </div>
            )}
          </div>
          
          {!readOnly && !isEditing && (
            <button
              type="button"
              onClick={handleTagAdd}
              className="text-blue-600 hover:text-blue-800"
            >
              + Add tag
            </button>
          )}
        </div>
      </div>
    </div>
  );
}
```

## §5. FILE UPLOAD DROP ZONE

```typescript
// components/dnd/FileDropzone.tsx
'use client';

import React, { useRef, useState, useCallback } from 'react';
import { useDropzone } from 'react-dropzone';
import { cn } from '@/lib/utils';
import { Upload, File, X, CheckCircle, AlertCircle } from 'lucide-react';

interface FileDropzoneProps {
  onDrop: (files: File[]) => Promise<void> | void;
  accept?: Record<string, string[]>;
  maxSize?: number;
  maxFiles?: number;
  disabled?: boolean;
  className?: string;
}

interface FileWithStatus {
  file: File;
  status: 'pending' | 'uploading' | 'success' | 'error';
  progress?: number;
  error?: string;
}

export function FileDropzone({
  onDrop,
  accept = {
    'image/*': ['.png', '.jpg', '.jpeg', '.gif'],
    'application/pdf': ['.pdf'],
    'text/plain': ['.txt'],
  },
  maxSize = 10 * 1024 * 1024, // 10MB
  maxFiles = 10,
  disabled = false,
  className,
}: FileDropzoneProps) {
  const [files, setFiles] = useState<FileWithStatus[]>([]);
  const [isDragging, setIsDragging] = useState(false);

  const onDropCallback = useCallback(async (acceptedFiles: File[]) => {
    if (disabled) return;

    const newFiles: FileWithStatus[] = acceptedFiles.map(file => ({
      file,
      status: 'pending',
    }));

    setFiles(prev => {
      const combined = [...prev, ...newFiles];
      // Limit to maxFiles
      return combined.slice(0, maxFiles);
    });

    try {
      // Update status to uploading
      setFiles(prev => prev.map(f => 
        newFiles.some(nf => nf.file === f.file) 
          ? { ...f, status: 'uploading', progress: 0 }
          : f
      ));

      // Call onDrop callback
      await onDrop(acceptedFiles);

      // Update status to success
      setFiles(prev => prev.map(f => 
        newFiles.some(nf => nf.file === f.file) 
          ? { ...f, status: 'success', progress: 100 }
          : f
      ));
    } catch (error) {
      // Update status to error
      setFiles(prev => prev.map(f => 
        newFiles.some(nf => nf.file === f.file) 
          ? { ...f, status: 'error', error: error instanceof Error ? error.message : 'Upload failed' }
          : f
      ));
    }
  }, [onDrop, disabled, maxFiles]);

  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    onDrop: onDropCallback,
    accept,
    maxSize,
    maxFiles,
    disabled,
    onDragEnter: () => setIsDragging(true),
    onDragLeave: () => setIsDragging(false),
    onDropAccepted: () => setIsDragging(false),
    onDropRejected: () => setIsDragging(false),
  });

  const removeFile = useCallback((index: number) => {
    setFiles(prev => prev.filter((_, i) => i !== index));
  }, []);

  const formatFileSize = (bytes: number) => {
    if (bytes === 0) return '0 Bytes';
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  };

  return (
    <div className={cn('space-y-4', className)}>
      {/* Dropzone */}
      <div
        {...getRootProps()}
        className={cn(
          'border-2 border-dashed rounded-lg p-8 text-center cursor-pointer transition-all',
          'hover:border-blue-400 hover:bg-blue-50',
          isDragActive && 'border-blue-500 bg-blue-100 border-solid',
          disabled && 'opacity-50 cursor-not-allowed'
        )}
      >
        <input {...getInputProps()} />
        
        <Upload className={cn(
          'h-12 w-12 mx-auto mb-4',
          isDragActive ? 'text-blue-500' : 'text-gray-400'
        )} />
        
        <div className="space-y-2">
          <p className="text-lg font-medium">
            {isDragActive ? 'Drop files here' : 'Drag & drop files here'}
          </p>
          <p className="text-sm text-gray-500">
            or click to browse
          </p>
          <div className="text-xs text-gray-400 space-y-1 mt-4">
            <p>Supported: {Object.values(accept).flat().join(', ')}</p>
            <p>Max size: {formatFileSize(maxSize)}</p>
            {maxFiles > 1 && <p>Max files: {maxFiles}</p>}
          </div>
        </div>
      </div>

      {/* File List */}
      {files.length > 0 && (
        <div className="space-y-2">
          <h3 className="text-sm font-medium">Files ({files.length})</h3>
          <div className="space-y-2">
            {files.map((fileStatus, index) => (
              <div
                key={`${fileStatus.file.name}-${index}`}
                className={cn(
                  'flex items-center justify-between p-3 border rounded-lg',
                  fileStatus.status === 'success' && 'border-green-200 bg-green-50',
                  fileStatus.status === 'error' && 'border-red-200 bg-red-50',
                  fileStatus.status === 'uploading' && 'border-blue-200 bg-blue-50',
                )}
              >
                <div className="flex items-center gap-3 flex-1 min-w-0">
                  <div className="flex-shrink-0">
                    {fileStatus.status === 'success' ? (
                      <CheckCircle className="h-5 w-5 text-green-500" />
                    ) : fileStatus.status === 'error' ? (
                      <AlertCircle className="h-5 w-5 text-red-500" />
                    ) : (
                      <File className="h-5 w-5 text-gray-400" />
                    )}
                  </div>
                  
                  <div className="flex-1 min-w-0">
                    <p className="text-sm font-medium truncate">
                      {fileStatus.file.name}
                    </p>
                    <div className="flex items-center gap-3 text-xs text-gray-500">
                      <span>{formatFileSize(fileStatus.file.size)}</span>
                      <span>•</span>
                      <span className="capitalize">{fileStatus.status}</span>
                      {fileStatus.error && (
                        <span className="text-red-600">{fileStatus.error}</span>
                      )}
                    </div>
                    
                    {fileStatus.status === 'uploading' && fileStatus.progress !== undefined && (
                      <div className="mt-2 w-full bg-gray-200 rounded-full h-1">
                        <div
                          className="bg-blue-600 h-1 rounded-full transition-all duration-300"
                          style={{ width: `${fileStatus.progress}%` }}
                        />
                      </div>
                    )}
                  </div>
                </div>
                
                {!disabled && (
                  <button
                    type="button"
                    onClick={() => removeFile(index)}
                    className="p-1 text-gray-400 hover:text-gray-600"
                    aria-label={`Remove ${fileStatus.file.name}`}
                  >
                    <X className="h-4 w-4" />
                  </button>
                )}
              </div>
            ))}
          </div>
        </div>
      )}
    </div>
  );
}
```

[Continua... Le sezioni rimanenti includerebbero:

## §6. TREE/HIERARCHY DRAG
- Sortable tree component con nesting
- Collapse/expand durante il drag
- Parent-child relationships

## §7. ACCESSIBILITY
- Keyboard navigation completa
- Screen reader announcements
- Focus management durante il drag

## §8. TOUCH SUPPORT
- Touch sensor configuration
- Delay per distinguere drag da scroll
- Visual feedback per touch devices

## §9. PERFORMANCE
- Virtualization per liste lunghe
- Memoization con React.memo e useMemo
- CSS transforms invece di layout changes

## §10. COMMON PATTERNS
- Reorderable table rows
- Image gallery reordering
- Priority/ranking list
- Dashboard widget drag & drop

## §11. DRAG & DROP CHECKLIST
- Checklist completo per deployment
- Testing strategies
- Performance optimization
- Mobile e accessibility testing

Il catalogo completo sarebbe di 800-1000 righe con codice production-ready.]