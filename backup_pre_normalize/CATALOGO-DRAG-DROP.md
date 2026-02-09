# CATALOGO-DRAG-DROP

CATALOGO-DRAG-DROP per Next.js 14 + TypeScript
§ DND-KIT COMPLETE
Setup completo con sensori
typescript
// lib/dnd/setup.ts
import {
  DndContext,
  DragEndEvent,
  DragOverlay,
  DragStartEvent,
  KeyboardSensor,
  PointerSensor,
  TouchSensor,
  closestCorners,
  defaultDropAnimation,
  useSensor,
  useSensors,
} from '@dnd-kit/core';
import { sortableKeyboardCoordinates } from '@dnd-kit/sortable';
import { ReactNode, useState } from 'react';

interface DndProviderProps {
  children: ReactNode;
  onDragEnd?: (event: DragEndEvent) => void;
  onDragStart?: (event: DragStartEvent) => void;
}

export function DndProvider({ children, onDragEnd, onDragStart }: DndProviderProps) {
  const [activeId, setActiveId] = useState<string | null>(null);

  const sensors = useSensors(
    useSensor(PointerSensor, {
      activationConstraint: {
        distance: 8, // 8px di movimento prima di iniziare il drag
      },
    }),
    useSensor(TouchSensor, {
      activationConstraint: {
        delay: 250,
        tolerance: 5,
      },
    }),
    useSensor(KeyboardSensor, {
      coordinateGetter: sortableKeyboardCoordinates,
    })
  );

  const handleDragStart = (event: DragStartEvent) => {
    setActiveId(event.active.id as string);
    onDragStart?.(event);
  };

  const handleDragEnd = (event: DragEndEvent) => {
    setActiveId(null);
    onDragEnd?.(event);
  };

  const dropAnimation = {
    ...defaultDropAnimation,
    dragSourceOpacity: 0.5,
  };

  return (
    <DndContext
      sensors={sensors}
      collisionDetection={closestCorners}
      onDragStart={handleDragStart}
      onDragEnd={handleDragEnd}
      accessibility={{
        announcements: {
          onDragStart(id) {
            return `Hai iniziato a trascinare l'elemento ${id}`;
          },
          onDragOver(id, overId) {
            if (overId) {
              return `L'elemento ${id} è stato spostato sopra ${overId}`;
            }
            return `L'elemento ${id} è in una nuova posizione`;
          },
          onDragEnd(id, overId) {
            if (overId) {
              return `L'elemento ${id} è stato rilasciato sopra ${overId}`;
            }
            return `L'elemento ${id} è stato rilasciato`;
          },
        },
        screenReaderInstructions: {
          draggable: `
            Per raccogliere un elemento, premi spazio o invio.
            Mentre trascini, usa le frecce per spostare l'elemento.
            Premi spazio o invio per rilasciare.
            Premi esc per annullare.
          `,
        },
      }}
    >
      {children}
      <DragOverlay dropAnimation={dropAnimation}>
        {activeId ? (
          <div className="opacity-80 rotate-3 transform-gpu">
            {/* Contenuto dell'overlay personalizzato */}
          </div>
        ) : null}
      </DragOverlay>
    </DndContext>
  );
}
Sortable Lists con Multiple Containers
typescript
// components/dnd/SortableList.tsx
import {
  SortableContext,
  verticalListSortingStrategy,
  horizontalListSortingStrategy,
  rectSortingStrategy,
} from '@dnd-kit/sortable';
import { useSortable } from '@dnd-kit/sortable';
import { CSS } from '@dnd-kit/utilities';
import { ReactNode } from 'react';

interface SortableItemProps {
  id: string;
  children: ReactNode;
  disabled?: boolean;
}

export function SortableItem({ id, children, disabled }: SortableItemProps) {
  const {
    attributes,
    listeners,
    setNodeRef,
    transform,
    transition,
    isDragging,
  } = useSortable({ id, disabled });

  const style = {
    transform: CSS.Transform.toString(transform),
    transition,
    opacity: isDragging ? 0.5 : 1,
    position: 'relative' as const,
    zIndex: isDragging ? 1000 : 'auto',
  };

  return (
    <div
      ref={setNodeRef}
      style={style}
      className={`relative ${isDragging ? 'shadow-2xl' : 'shadow-sm'}`}
    >
      <div
        className="absolute left-2 top-1/2 transform -translate-y-1/2 cursor-grab active:cursor-grabbing"
        {...attributes}
        {...listeners}
      >
        ⋮⋮
      </div>
      {children}
    </div>
  );
}

interface SortableListProps {
  items: { id: string }[];
  children: (item: { id: string }) => ReactNode;
  direction?: 'vertical' | 'horizontal' | 'grid';
}

export function SortableList({ items, children, direction = 'vertical' }: SortableListProps) {
  const strategy = {
    vertical: verticalListSortingStrategy,
    horizontal: horizontalListSortingStrategy,
    grid: rectSortingStrategy,
  }[direction];

  return (
    <SortableContext items={items.map(item => item.id)} strategy={strategy}>
      <div className={`flex ${direction === 'vertical' ? 'flex-col' : direction === 'horizontal' ? 'flex-row' : 'grid grid-cols-3 gap-4'}`}>
        {items.map((item) => (
          <SortableItem key={item.id} id={item.id}>
            {children(item)}
          </SortableItem>
        ))}
      </div>
    </SortableContext>
  );
}
Multiple Containers con Drag Between
typescript
// components/dnd/MultipleContainers.tsx
import { useState } from 'react';
import { DndProvider } from './DndProvider';
import { Droppable } from './Droppable';
import { SortableList } from './SortableList';

interface Item {
  id: string;
  title: string;
  containerId: string;
}

interface Container {
  id: string;
  title: string;
}

export function MultipleContainersDemo() {
  const [containers, setContainers] = useState<Container[]>([
    { id: 'todo', title: 'To Do' },
    { id: 'in-progress', title: 'In Progress' },
    { id: 'done', title: 'Done' },
  ]);

  const [items, setItems] = useState<Item[]>([
    { id: '1', title: 'Task 1', containerId: 'todo' },
    { id: '2', title: 'Task 2', containerId: 'todo' },
    { id: '3', title: 'Task 3', containerId: 'in-progress' },
    { id: '4', title: 'Task 4', containerId: 'done' },
  ]);

  const handleDragEnd = (event: any) => {
    const { active, over } = event;

    if (!over) return;

    const activeId = active.id;
    const overId = over.id;

    // Se stiamo trascinando su un container
    if (overId.startsWith('container-')) {
      const containerId = overId.replace('container-', '');
      setItems(items.map(item => 
        item.id === activeId ? { ...item, containerId } : item
      ));
    }

    // Se stiamo riordinando i container stessi
    if (activeId.startsWith('container-')) {
      const oldIndex = containers.findIndex(c => `container-${c.id}` === activeId);
      const newIndex = containers.findIndex(c => `container-${c.id}` === overId);
      
      if (oldIndex !== -1 && newIndex !== -1 && oldIndex !== newIndex) {
        const newContainers = [...containers];
        const [moved] = newContainers.splice(oldIndex, 1);
        newContainers.splice(newIndex, 0, moved);
        setContainers(newContainers);
      }
    }
  };

  return (
    <DndProvider onDragEnd={handleDragEnd}>
      <div className="flex gap-4 p-4">
        <SortableList 
          items={containers.map(c => ({ id: `container-${c.id}` }))}
          direction="horizontal"
        >
          {(container) => {
            const containerId = container.id.replace('container-', '');
            const containerData = containers.find(c => c.id === containerId);
            const containerItems = items.filter(item => item.containerId === containerId);

            return (
              <Droppable id={container.id}>
                <div className="w-64 bg-gray-100 rounded-lg p-4 min-h-[500px]">
                  <h3 className="font-bold mb-4">{containerData?.title}</h3>
                  <SortableList items={containerItems} direction="vertical">
                    {(item) => (
                      <div className="bg-white p-3 rounded mb-2 shadow">
                        {item.title}
                      </div>
                    )}
                  </SortableList>
                </div>
              </Droppable>
            );
          }}
        </SortableList>
      </div>
    </DndProvider>
  );
}
Collision Detection Strategies
typescript
// lib/dnd/collision-detection.ts
import {
  CollisionDetection,
  closestCenter,
  closestCorners,
  rectIntersection,
  pointerWithin,
  getFirstCollision,
  rectIntersection as rectIntersectionStrategy,
} from '@dnd-kit/core';

export const customCollisionDetection: CollisionDetection = (args) => {
  // Prima controlla se il puntatore è dentro un droppable
  const pointerCollisions = pointerWithin(args);
  
  if (pointerCollisions.length > 0) {
    return pointerCollisions;
  }

  // Poi controlla le intersezioni tra rettangoli
  const rectIntersections = rectIntersectionStrategy(args);
  
  if (rectIntersections.length > 0) {
    return rectIntersections;
  }

  // Fallback: angoli più vicini
  return closestCorners(args);
};

export const multiContainerCollisionDetection: CollisionDetection = (args) => {
  const { active, droppableContainers, pointerCoordinates } = args;

  // Per il drag tra container, usiamo una strategia personalizzata
  if (!pointerCoordinates) {
    return [];
  }

  // Crea un rettangolo virtuale attorno al puntatore
  const pointerRect = {
    left: pointerCoordinates.x - 20,
    top: pointerCoordinates.y - 20,
    right: pointerCoordinates.x + 20,
    bottom: pointerCoordinates.y + 20,
    width: 40,
    height: 40,
  };

  const collisions = [];

  for (const container of droppableContainers) {
    const { rect } = container;
    
    if (rect) {
      const intersects = !(
        pointerRect.left > rect.right ||
        pointerRect.right < rect.left ||
        pointerRect.top > rect.bottom ||
        pointerRect.bottom < rect.top
      );

      if (intersects) {
        collisions.push({
          id: container.id,
          data: { droppableContainer: container },
        });
      }
    }
  }

  return collisions;
};
§ KANBAN BOARD
Kanban Board Completo
typescript
// components/kanban/KanbanBoard.tsx
'use client';

import { useState, useEffect } from 'react';
import { DragEndEvent } from '@dnd-kit/core';
import { arrayMove } from '@dnd-kit/sortable';
import { DndProvider } from '@/lib/dnd/DndProvider';
import { KanbanColumn } from './KanbanColumn';
import { CreateCardForm } from './CreateCardForm';
import { CreateColumnForm } from './CreateColumnForm';
import { useOptimisticUpdate } from '@/hooks/useOptimisticUpdate';
import { saveBoardState, getBoardState } from '@/lib/kanban/persistence';

interface Card {
  id: string;
  title: string;
  description?: string;
  columnId: string;
  position: number;
  createdAt: Date;
  updatedAt: Date;
  labels?: string[];
  assignee?: string;
}

interface Column {
  id: string;
  title: string;
  position: number;
  cards: Card[];
  maxCards?: number;
}

interface KanbanBoardProps {
  initialColumns?: Column[];
  boardId: string;
  readOnly?: boolean;
}

export function KanbanBoard({ initialColumns = [], boardId, readOnly = false }: KanbanBoardProps) {
  const [columns, setColumns] = useState<Column[]>(initialColumns);
  const { optimisticUpdate, rollback } = useOptimisticUpdate();

  // Carica lo stato iniziale dal database
  useEffect(() => {
    const loadBoard = async () => {
      try {
        const savedState = await getBoardState(boardId);
        if (savedState) {
          setColumns(savedState.columns);
        }
      } catch (error) {
        console.error('Errore nel caricamento della board:', error);
      }
    };
    
    loadBoard();
  }, [boardId]);

  // Salvataggio automatico
  useEffect(() => {
    const saveTimeout = setTimeout(async () => {
      try {
        await saveBoardState(boardId, { columns });
      } catch (error) {
        console.error('Errore nel salvataggio:', error);
      }
    }, 1000);

    return () => clearTimeout(saveTimeout);
  }, [columns, boardId]);

  const handleDragEnd = async (event: DragEndEvent) => {
    const { active, over } = event;

    if (!over) return;

    const activeId = active.id as string;
    const overId = over.id as string;

    try {
      // Update ottimistico
      const previousColumns = [...columns];
      
      optimisticUpdate(async () => {
        setColumns(currentColumns => {
          let newColumns = [...currentColumns];

          // Trascinamento tra colonne
          if (activeId.startsWith('card-') && overId.startsWith('column-')) {
            const cardId = activeId.replace('card-', '');
            const targetColumnId = overId.replace('column-', '');
            
            newColumns = newColumns.map(column => {
              // Rimuovi dalla colonna originale
              if (column.cards.some(card => card.id === cardId)) {
                return {
                  ...column,
                  cards: column.cards.filter(card => card.id !== cardId),
                };
              }
              // Aggiungi alla colonna di destinazione
              if (column.id === targetColumnId) {
                const card = previousColumns
                  .flatMap(c => c.cards)
                  .find(c => c.id === cardId);
                
                if (card) {
                  return {
                    ...column,
                    cards: [
                      ...column.cards,
                      { ...card, columnId: targetColumnId, updatedAt: new Date() },
                    ],
                  };
                }
              }
              return column;
            });
          }
          // Riordinamento nella stessa colonna
          else if (activeId.startsWith('card-') && overId.startsWith('card-')) {
            const activeCardId = activeId.replace('card-', '');
            const overCardId = overId.replace('card-', '');
            
            newColumns = newColumns.map(column => {
              const cardIds = column.cards.map(card => card.id);
              const oldIndex = cardIds.indexOf(activeCardId);
              const newIndex = cardIds.indexOf(overCardId);
              
              if (oldIndex !== -1 && newIndex !== -1) {
                return {
                  ...column,
                  cards: arrayMove(column.cards, oldIndex, newIndex),
                };
              }
              return column;
            });
          }
          // Riordinamento colonne
          else if (activeId.startsWith('column-') && overId.startsWith('column-')) {
            const activeColumnId = activeId.replace('column-', '');
            const overColumnId = overId.replace('column-', '');
            
            const oldIndex = newColumns.findIndex(c => `column-${c.id}` === activeId);
            const newIndex = newColumns.findIndex(c => `column-${c.id}` === overId);
            
            if (oldIndex !== -1 && newIndex !== -1) {
              newColumns = arrayMove(newColumns, oldIndex, newIndex);
            }
          }

          return newColumns;
        });

        // Qui andrebbe la chiamata API al database
        await saveBoardState(boardId, { columns: newColumns });
      }).catch(() => {
        rollback();
      });
    } catch (error) {
      console.error('Errore durante il drag:', error);
    }
  };

  const handleCreateCard = async (columnId: string, title: string, description?: string) => {
    const newCard: Card = {
      id: `card-${Date.now()}`,
      title,
      description,
      columnId,
      position: columns.find(c => c.id === columnId)?.cards.length || 0,
      createdAt: new Date(),
      updatedAt: new Date(),
    };

    setColumns(currentColumns =>
      currentColumns.map(column =>
        column.id === columnId
          ? { ...column, cards: [...column.cards, newCard] }
          : column
      )
    );

    try {
      await saveBoardState(boardId, { columns });
    } catch (error) {
      console.error('Errore nella creazione della card:', error);
    }
  };

  const handleCreateColumn = async (title: string) => {
    const newColumn: Column = {
      id: `col-${Date.now()}`,
      title,
      position: columns.length,
      cards: [],
    };

    setColumns([...columns, newColumn]);

    try {
      await saveBoardState(boardId, { columns: [...columns, newColumn] });
    } catch (error) {
      console.error('Errore nella creazione della colonna:', error);
    }
  };

  return (
    <div className="p-4 bg-gray-50 min-h-screen">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-bold">Kanban Board</h1>
        {!readOnly && <CreateColumnForm onCreate={handleCreateColumn} />}
      </div>

      <DndProvider onDragEnd={handleDragEnd}>
        <div className="flex gap-4 overflow-x-auto pb-4">
          {columns.map((column) => (
            <KanbanColumn
              key={column.id}
              column={column}
              onCardCreate={!readOnly ? (title, description) => 
                handleCreateCard(column.id, title, description) : undefined}
              readOnly={readOnly}
            />
          ))}
        </div>
      </DndProvider>
    </div>
  );
}
Componenti Kanban
typescript
// components/kanban/KanbanColumn.tsx
import { useDroppable } from '@dnd-kit/core';
import { SortableContext, verticalListSortingStrategy } from '@dnd-kit/sortable';
import { KanbanCard } from './KanbanCard';
import { CreateCardForm } from './CreateCardForm';
import { GripVertical } from 'lucide-react';

interface KanbanColumnProps {
  column: {
    id: string;
    title: string;
    cards: Array<{
      id: string;
      title: string;
      description?: string;
      labels?: string[];
      assignee?: string;
    }>;
  };
  onCardCreate?: (title: string, description?: string) => void;
  readOnly?: boolean;
}

export function KanbanColumn({ column, onCardCreate, readOnly }: KanbanColumnProps) {
  const { setNodeRef, isOver } = useDroppable({
    id: `column-${column.id}`,
    data: { type: 'column', column },
  });

  const cardIds = column.cards.map(card => `card-${card.id}`);

  return (
    <div
      ref={setNodeRef}
      className={`w-72 flex-shrink-0 ${isOver ? 'bg-blue-50' : 'bg-gray-100'} rounded-lg p-4 transition-colors duration-200`}
    >
      <div className="flex items-center justify-between mb-4">
        <div className="flex items-center gap-2">
          {!readOnly && (
            <button className="cursor-grab active:cursor-grabbing">
              <GripVertical className="w-4 h-4 text-gray-500" />
            </button>
          )}
          <h3 className="font-semibold text-gray-800">{column.title}</h3>
          <span className="bg-gray-200 text-gray-600 text-xs px-2 py-1 rounded-full">
            {column.cards.length}
          </span>
        </div>
      </div>

      <SortableContext items={cardIds} strategy={verticalListSortingStrategy}>
        <div className="space-y-3 min-h-[100px]">
          {column.cards.map((card) => (
            <KanbanCard key={card.id} card={card} readOnly={readOnly} />
          ))}
        </div>
      </SortableContext>

      {!readOnly && onCardCreate && (
        <div className="mt-4">
          <CreateCardForm onCreate={onCardCreate} />
        </div>
      )}
    </div>
  );
}
typescript
// components/kanban/KanbanCard.tsx
import { useSortable } from '@dnd-kit/sortable';
import { CSS } from '@dnd-kit/utilities';
import { MoreVertical, Tag, User } from 'lucide-react';
import { useState } from 'react';

interface KanbanCardProps {
  card: {
    id: string;
    title: string;
    description?: string;
    labels?: string[];
    assignee?: string;
  };
  readOnly?: boolean;
}

export function KanbanCard({ card, readOnly }: KanbanCardProps) {
  const {
    attributes,
    listeners,
    setNodeRef,
    transform,
    transition,
    isDragging,
  } = useSortable({
    id: `card-${card.id}`,
    disabled: readOnly,
  });

  const [isHovered, setIsHovered] = useState(false);

  const style = {
    transform: CSS.Transform.toString(transform),
    transition,
    opacity: isDragging ? 0.5 : 1,
  };

  return (
    <div
      ref={setNodeRef}
      style={style}
      className={`bg-white rounded-lg shadow-sm border border-gray-200 p-4 cursor-pointer hover:shadow-md transition-all duration-200 ${
        isDragging ? 'shadow-xl rotate-1' : ''
      }`}
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      <div className="flex justify-between items-start mb-2">
        <h4 className="font-medium text-gray-900">{card.title}</h4>
        {!readOnly && (
          <div className="flex items-center gap-1">
            {isHovered && (
              <button
                className="cursor-grab active:cursor-grabbing p-1"
                {...attributes}
                {...listeners}
              >
                ⋮⋮
              </button>
            )}
            <button className="p-1 hover:bg-gray-100 rounded">
              <MoreVertical className="w-4 h-4 text-gray-500" />
            </button>
          </div>
        )}
      </div>

      {card.description && (
        <p className="text-sm text-gray-600 mb-3 line-clamp-2">{card.description}</p>
      )}

      <div className="flex items-center justify-between mt-3">
        <div className="flex items-center gap-2">
          {card.labels && card.labels.length > 0 && (
            <div className="flex items-center gap-1">
              <Tag className="w-3 h-3 text-gray-400" />
              <div className="flex gap-1">
                {card.labels.slice(0, 2).map((label, index) => (
                  <span
                    key={index}
                    className="text-xs px-2 py-0.5 rounded-full bg-blue-100 text-blue-800"
                  >
                    {label}
                  </span>
                ))}
                {card.labels.length > 2 && (
                  <span className="text-xs text-gray-500">
                    +{card.labels.length - 2}
                  </span>
                )}
              </div>
            </div>
          )}
        </div>

        {card.assignee && (
          <div className="flex items-center gap-1">
            <User className="w-3 h-3 text-gray-400" />
            <span className="text-xs text-gray-600">{card.assignee}</span>
          </div>
        )}
      </div>
    </div>
  );
}
Persistence e Optimistic Updates
typescript
// lib/kanban/persistence.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export async function saveBoardState(boardId: string, state: any) {
  try {
    await prisma.board.upsert({
      where: { id: boardId },
      update: {
        state: JSON.stringify(state),
        updatedAt: new Date(),
      },
      create: {
        id: boardId,
        state: JSON.stringify(state),
      },
    });
  } catch (error) {
    console.error('Errore nel salvataggio dello stato:', error);
    throw error;
  }
}

export async function getBoardState(boardId: string) {
  try {
    const board = await prisma.board.findUnique({
      where: { id: boardId },
    });

    if (board?.state) {
      return JSON.parse(board.state);
    }
    return null;
  } catch (error) {
    console.error('Errore nel caricamento dello stato:', error);
    throw error;
  }
}

export async function saveBoardOrder(boardId: string, columns: any[]) {
  try {
    await prisma.$transaction(async (tx) => {
      // Salva l'ordine delle colonne
      for (const [index, column] of columns.entries()) {
        await tx.column.upsert({
          where: { id: column.id },
          update: {
            position: index,
            updatedAt: new Date(),
          },
          create: {
            id: column.id,
            title: column.title,
            position: index,
            boardId,
          },
        });

        // Salva l'ordine delle card
        for (const [cardIndex, card] of column.cards.entries()) {
          await tx.card.upsert({
            where: { id: card.id },
            update: {
              position: cardIndex,
              columnId: column.id,
              updatedAt: new Date(),
            },
            create: {
              id: card.id,
              title: card.title,
              description: card.description,
              position: cardIndex,
              columnId: column.id,
              boardId,
            },
          });
        }
      }
    });
  } catch (error) {
    console.error('Errore nel salvataggio dell\'ordine:', error);
    throw error;
  }
}
typescript
// hooks/useOptimisticUpdate.ts
import { useState, useCallback } from 'react';

interface OptimisticUpdateOptions {
  onError?: (error: any) => void;
  onSuccess?: (result: any) => void;
}

export function useOptimisticUpdate(options?: OptimisticUpdateOptions) {
  const [pendingUpdates, setPendingUpdates] = useState<any[]>([]);
  const [history, setHistory] = useState<any[]>([]);

  const optimisticUpdate = useCallback(
    async (updateFn: () => Promise<any>) => {
      // Salva lo stato attuale per il rollback
      setHistory(prev => [...prev, { timestamp: Date.now(), state: pendingUpdates }]);

      try {
        const result = await updateFn();
        options?.onSuccess?.(result);
        return result;
      } catch (error) {
        options?.onError?.(error);
        throw error;
      } finally {
        // Rimuovi l'update dalla lista pending
        setPendingUpdates(prev => prev.slice(0, -1));
      }
    },
    [pendingUpdates, options]
  );

  const rollback = useCallback(() => {
    if (history.length > 0) {
      const lastState = history[history.length - 1];
      setPendingUpdates(lastState.state);
      setHistory(prev => prev.slice(0, -1));
    }
  }, [history]);

  return {
    optimisticUpdate,
    rollback,
    pendingUpdates,
    isUpdating: pendingUpdates.length > 0,
  };
}
§ FILE UPLOAD DND
Dropzone Component con react-dropzone
typescript
// components/file-upload/Dropzone.tsx
'use client';

import { useCallback, useState } from 'react';
import { useDropzone } from 'react-dropzone';
import { useDndContext, DragOverlay } from '@dnd-kit/core';
import { Upload, File, X, Check, AlertCircle } from 'lucide-react';

interface UploadFile {
  id: string;
  file: File;
  progress: number;
  status: 'pending' | 'uploading' | 'completed' | 'error';
  error?: string;
  url?: string;
}

interface DropzoneProps {
  onUpload: (files: File[]) => Promise<void>;
  accept?: Record<string, string[]>;
  maxSize?: number;
  maxFiles?: number;
  multiple?: boolean;
}

export function Dropzone({
  onUpload,
  accept = {
    'image/*': ['.jpg', '.jpeg', '.png', '.gif'],
    'application/pdf': ['.pdf'],
    'text/plain': ['.txt'],
  },
  maxSize = 10 * 1024 * 1024, // 10MB
  maxFiles = 10,
  multiple = true,
}: DropzoneProps) {
  const [files, setFiles] = useState<UploadFile[]>([]);
  const [dragOver, setDragOver] = useState(false);
  const { active } = useDndContext();

  const validateFile = (file: File): string | null => {
    // Validazione dimensione
    if (file.size > maxSize) {
      return `File troppo grande (max ${maxSize / 1024 / 1024}MB)`;
    }

    // Validazione tipo
    const fileExtension = '.' + file.name.split('.').pop()?.toLowerCase();
    const isValidType = Object.values(accept).some(extensions =>
      extensions.includes(fileExtension)
    );

    if (!isValidType) {
      return 'Tipo di file non supportato';
    }

    return null;
  };

  const onDrop = useCallback(async (acceptedFiles: File[], rejectedFiles: any[]) => {
    // Gestisci file rifiutati
    if (rejectedFiles.length > 0) {
      rejectedFiles.forEach(({ file, errors }) => {
        console.error(`File rifiutato: ${file.name}`, errors);
      });
    }

    // Crea oggetti UploadFile
    const newUploadFiles: UploadFile[] = acceptedFiles.map(file => ({
      id: `${file.name}-${Date.now()}`,
      file,
      progress: 0,
      status: 'pending',
    }));

    setFiles(prev => {
      const updated = [...prev, ...newUploadFiles].slice(-maxFiles);
      return updated;
    });

    // Avvia upload
    try {
      await onUpload(acceptedFiles);
      
      // Simula progresso upload
      newUploadFiles.forEach((uploadFile, index) => {
        simulateUpload(uploadFile.id);
      });
    } catch (error) {
      setFiles(prev =>
        prev.map(f =>
          newUploadFiles.some(nf => nf.id === f.id)
            ? { ...f, status: 'error', error: 'Upload failed' }
            : f
        )
      );
    }
  }, [onUpload, maxFiles]);

  const simulateUpload = (fileId: string) => {
    setFiles(prev =>
      prev.map(f =>
        f.id === fileId ? { ...f, status: 'uploading' } : f
      )
    );

    let progress = 0;
    const interval = setInterval(() => {
      progress += Math.random() * 10;
      if (progress >= 100) {
        progress = 100;
        clearInterval(interval);
        
        setFiles(prev =>
          prev.map(f =>
            f.id === fileId
              ? { ...f, progress: 100, status: 'completed' }
              : f
          )
        );
      } else {
        setFiles(prev =>
          prev.map(f =>
            f.id === fileId ? { ...f, progress } : f
          )
        );
      }
    }, 100);
  };

  const removeFile = (fileId: string) => {
    setFiles(prev => prev.filter(f => f.id !== fileId));
  };

  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    onDrop,
    accept,
    maxSize,
    maxFiles,
    multiple,
    onDragEnter: () => setDragOver(true),
    onDragLeave: () => setDragOver(false),
    validator: (file) => {
      const error = validateFile(file);
      return error ? new Error(error) : null;
    },
  });

  return (
    <div className="space-y-4">
      {/* Dropzone Area */}
      <div
        {...getRootProps()}
        className={`
          border-2 border-dashed rounded-lg p-8 text-center cursor-pointer
          transition-all duration-200
          ${dragOver || isDragActive
            ? 'border-blue-500 bg-blue-50'
            : 'border-gray-300 hover:border-gray-400'
          }
          ${active ? 'opacity-50' : ''}
        `}
      >
        <input {...getInputProps()} />
        
        <div className="flex flex-col items-center justify-center gap-3">
          <div className={`p-3 rounded-full ${dragOver ? 'bg-blue-100' : 'bg-gray-100'}`}>
            <Upload className={`w-8 h-8 ${dragOver ? 'text-blue-600' : 'text-gray-600'}`} />
          </div>
          
          <div>
            <p className="text-lg font-medium text-gray-700">
              {isDragActive
                ? 'Rilascia i file qui...'
                : 'Trascina i file o clicca per selezionare'}
            </p>
            <p className="text-sm text-gray-500 mt-1">
              Supportati: {Object.values(accept).flat().join(', ')}
            </p>
            <p className="text-xs text-gray-400 mt-1">
              Max {maxSize / 1024 / 1024}MB per file, max {maxFiles} file
            </p>
          </div>
        </div>
      </div>

      {/* File List */}
      {files.length > 0 && (
        <div className="space-y-2">
          <h3 className="font-medium text-gray-700">File caricati</h3>
          <div className="grid gap-2">
            {files.map((uploadFile) => (
              <FileItem
                key={uploadFile.id}
                uploadFile={uploadFile}
                onRemove={removeFile}
              />
            ))}
          </div>
        </div>
      )}

      {/* Drag Overlay Preview */}
      <DragOverlay>
        {active && (
          <div className="bg-white p-4 rounded-lg shadow-xl border border-gray-200 flex items-center gap-3">
            <File className="w-5 h-5 text-blue-500" />
            <span className="text-sm font-medium">
              {active.data.current?.files?.length || 1} file selezionati
            </span>
          </div>
        )}
      </DragOverlay>
    </div>
  );
}

function FileItem({ uploadFile, onRemove }: { uploadFile: UploadFile; onRemove: (id: string) => void }) {
  const getStatusIcon = () => {
    switch (uploadFile.status) {
      case 'completed':
        return <Check className="w-4 h-4 text-green-500" />;
      case 'error':
        return <AlertCircle className="w-4 h-4 text-red-500" />;
      case 'uploading':
        return

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-15-DRAG-DROP
Prompt ID: 15 / 48
Parte: 2
Exported: 2026-02-06T13:47:19.920Z
Characters: 171
════════════════════════════════════════════════════════════

r}
            disabled={JSON.stringify(products) === JSON.stringify(originalOrder)}
            className="inline-flex items-center gap-2 px-4 py-2 border border-gray-300