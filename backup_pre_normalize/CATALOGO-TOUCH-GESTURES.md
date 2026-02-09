# CATALOGO-TOUCH-GESTURES

Catalogo Touch Gestures per Next.js 14 + TypeScript
§ GESTURE FUNDAMENTALS
Touch events vs Pointer events
typescript
// Touch Events (legacy)
interface TouchEventHandlers {
  onTouchStart: (e: React.TouchEvent) => void;
  onTouchMove: (e: React.TouchEvent) => void;
  onTouchEnd: (e: React.TouchEvent) => void;
  onTouchCancel: (e: React.TouchEvent) => void;
}

// Pointer Events (modern - unifica mouse, touch, pen)
interface PointerEventHandlers {
  onPointerDown: (e: React.PointerEvent) => void;
  onPointerMove: (e: React.PointerEvent) => void;
  onPointerUp: (e: React.PointerEvent) => void;
  onPointerCancel: (e: React.PointerEvent) => void;
  onPointerLeave: (e: React.PointerEvent) => void;
}

// Hook consigliato per gestione unificata
import { useCallback, useEffect, useRef } from 'react';

export const useUnifiedGesture = (
  elementRef: React.RefObject<HTMLElement>,
  handlers: {
    onStart?: (e: PointerEvent | TouchEvent) => void;
    onMove?: (e: PointerEvent | TouchEvent) => void;
    onEnd?: (e: PointerEvent | TouchEvent) => void;
  }
) => {
  useEffect(() => {
    const element = elementRef.current;
    if (!element) return;

    const handleStart = (e: PointerEvent | TouchEvent) => {
      handlers.onStart?.(e);
    };

    const handleMove = (e: PointerEvent | TouchEvent) => {
      handlers.onMove?.(e);
    };

    const handleEnd = (e: PointerEvent | TouchEvent) => {
      handlers.onEnd?.(e);
    };

    // Usa Pointer Events quando disponibili
    if ('PointerEvent' in window) {
      element.addEventListener('pointerdown', handleStart as EventListener);
      element.addEventListener('pointermove', handleMove as EventListener);
      element.addEventListener('pointerup', handleEnd as EventListener);
      element.addEventListener('pointercancel', handleEnd as EventListener);
    } else {
      element.addEventListener('touchstart', handleStart as EventListener);
      element.addEventListener('touchmove', handleMove as EventListener);
      element.addEventListener('touchend', handleEnd as EventListener);
      element.addEventListener('touchcancel', handleEnd as EventListener);
    }

    return () => {
      if ('PointerEvent' in window) {
        element.removeEventListener('pointerdown', handleStart as EventListener);
        element.removeEventListener('pointermove', handleMove as EventListener);
        element.removeEventListener('pointerup', handleEnd as EventListener);
        element.removeEventListener('pointercancel', handleEnd as EventListener);
      } else {
        element.removeEventListener('touchstart', handleStart as EventListener);
        element.removeEventListener('touchmove', handleMove as EventListener);
        element.removeEventListener('touchend', handleEnd as EventListener);
        element.removeEventListener('touchcancel', handleEnd as EventListener);
      }
    };
  }, [elementRef, handlers]);
};
Event properties
typescript
export interface TouchPoint {
  clientX: number;
  clientY: number;
  identifier: number;
  pageX: number;
  pageY: number;
  screenX: number;
  screenY: number;
  radiusX?: number;
  radiusY?: number;
  rotationAngle?: number;
  force?: number;
}

export class TouchAnalyzer {
  static getTouchPoints(event: TouchEvent | PointerEvent): TouchPoint[] {
    if ('touches' in event) {
      return Array.from(event.touches).map(touch => ({
        clientX: touch.clientX,
        clientY: touch.clientY,
        identifier: touch.identifier,
        pageX: touch.pageX,
        pageY: touch.pageY,
        screenX: touch.screenX,
        screenY: touch.screenY,
        radiusX: touch.radiusX,
        radiusY: touch.radiusY,
        rotationAngle: touch.rotationAngle,
        force: touch.force
      }));
    } else {
      return [{
        clientX: event.clientX,
        clientY: event.clientY,
        identifier: event.pointerId,
        pageX: event.pageX,
        pageY: event.pageY,
        screenX: event.screenX,
        screenY: event.screenY
      }];
    }
  }

  static getTargetTouches(event: TouchEvent): TouchPoint[] {
    return Array.from(event.targetTouches).map(touch => ({
      clientX: touch.clientX,
      clientY: touch.clientY,
      identifier: touch.identifier,
      pageX: touch.pageX,
      pageY: touch.pageY,
      screenX: touch.screenX,
      screenY: touch.screenY
    }));
  }

  static getChangedTouches(event: TouchEvent): TouchPoint[] {
    return Array.from(event.changedTouches).map(touch => ({
      clientX: touch.clientX,
      clientY: touch.clientY,
      identifier: touch.identifier,
      pageX: touch.pageX,
      pageY: touch.pageY,
      screenX: touch.screenX,
      screenY: touch.screenY
    }));
  }
}
Passive event listeners
typescript
export const usePassiveEventListener = (
  eventName: string,
  handler: (e: Event) => void,
  element?: HTMLElement | Window
) => {
  useEffect(() => {
    const target = element || window;
    const options: AddEventListenerOptions = {
      passive: true, // Migliora le performance
      capture: false
    };

    target.addEventListener(eventName, handler, options);
    
    return () => {
      target.removeEventListener(eventName, handler, options);
    };
  }, [eventName, handler, element]);
};

// Hook per touch scroll con passive listener
export const useTouchScroll = (
  elementRef: React.RefObject<HTMLElement>,
  enabled: boolean = true
) => {
  useEffect(() => {
    const element = elementRef.current;
    if (!element || !enabled) return;

    const handleTouchMove = (e: TouchEvent) => {
      // Permette lo scroll naturale
    };

    element.addEventListener('touchmove', handleTouchMove, { passive: true });
    
    return () => {
      element.removeEventListener('touchmove', handleTouchMove);
    };
  }, [elementRef, enabled]);
};
Preventing default behavior
typescript
export const usePreventDefault = (
  elementRef: React.RefObject<HTMLElement>,
  eventNames: string[],
  condition: () => boolean = () => true
) => {
  useEffect(() => {
    const element = elementRef.current;
    if (!element) return;

    const preventDefault = (e: Event) => {
      if (condition()) {
        e.preventDefault();
      }
    };

    eventNames.forEach(eventName => {
      element.addEventListener(eventName, preventDefault, { passive: false });
    });

    return () => {
      eventNames.forEach(eventName => {
        element.removeEventListener(eventName, preventDefault);
      });
    };
  }, [elementRef, eventNames, condition]);
};

// Hook specializzato per prevenire lo scroll durante il drag
export const usePreventScrollDuringDrag = (
  isDragging: boolean,
  elementRef?: React.RefObject<HTMLElement>
) => {
  useEffect(() => {
    if (!isDragging) return;

    const preventDefault = (e: TouchEvent) => {
      e.preventDefault();
    };

    const target = elementRef?.current || document;
    
    // Blocca lo scroll verticale/horizzontale durante il drag
    target.addEventListener('touchmove', preventDefault, { passive: false });
    target.addEventListener('wheel', preventDefault, { passive: false });

    return () => {
      target.removeEventListener('touchmove', preventDefault);
      target.removeEventListener('wheel', preventDefault);
    };
  }, [isDragging, elementRef]);
};
§ SWIPE GESTURES
Swipe detection hook
typescript
import { useState, useCallback, useRef, TouchEvent, PointerEvent } from 'react';

export interface SwipeConfig {
  threshold?: number; // Distanza minima in pixel
  velocityThreshold?: number; // Velocità minima in pixel/ms
  maxDuration?: number; // Durata massima in ms
  direction?: 'horizontal' | 'vertical' | 'both';
}

export interface SwipeEvent {
  direction: 'left' | 'right' | 'up' | 'down';
  distance: number;
  velocity: number;
  duration: number;
  startX: number;
  startY: number;
  endX: number;
  endY: number;
}

export const useSwipe = (
  onSwipe: (swipe: SwipeEvent) => void,
  config: SwipeConfig = {}
) => {
  const {
    threshold = 50,
    velocityThreshold = 0.3,
    maxDuration = 500,
    direction = 'both'
  } = config;

  const [isSwiping, setIsSwiping] = useState(false);
  const startPos = useRef({ x: 0, y: 0, time: 0 });
  const lastPos = useRef({ x: 0, y: 0, time: 0 });

  const handleStart = useCallback((clientX: number, clientY: number) => {
    const now = Date.now();
    startPos.current = { x: clientX, y: clientY, time: now };
    lastPos.current = { x: clientX, y: clientY, time: now };
    setIsSwiping(true);
  }, []);

  const handleMove = useCallback((clientX: number, clientY: number) => {
    if (!isSwiping) return;
    lastPos.current = { x: clientX, y: clientY, time: Date.now() };
  }, [isSwiping]);

  const handleEnd = useCallback(() => {
    if (!isSwiping) return;
    setIsSwiping(false);

    const { x: startX, y: startY, time: startTime } = startPos.current;
    const { x: endX, y: endY, time: endTime } = lastPos.current;
    
    const deltaX = endX - startX;
    const deltaY = endY - startY;
    const duration = endTime - startTime;
    
    // Calcola distanza e velocità
    const distanceX = Math.abs(deltaX);
    const distanceY = Math.abs(deltaY);
    const velocityX = distanceX / duration;
    const velocityY = distanceY / duration;
    
    // Determina direzione principale
    let swipeDirection: SwipeEvent['direction'] | null = null;
    let distance = 0;
    let velocity = 0;
    
    if (direction !== 'vertical' && distanceX > distanceY && distanceX > threshold) {
      swipeDirection = deltaX > 0 ? 'right' : 'left';
      distance = distanceX;
      velocity = velocityX;
    } else if (direction !== 'horizontal' && distanceY > distanceX && distanceY > threshold) {
      swipeDirection = deltaY > 0 ? 'down' : 'up';
      distance = distanceY;
      velocity = velocityY;
    }
    
    // Verifica condizioni
    if (
      swipeDirection &&
      duration <= maxDuration &&
      velocity >= velocityThreshold
    ) {
      onSwipe({
        direction: swipeDirection,
        distance,
        velocity,
        duration,
        startX,
        startY,
        endX,
        endY
      });
    }
  }, [isSwiping, threshold, maxDuration, velocityThreshold, direction, onSwipe]);

  // React event handlers
  const onPointerDown = useCallback((e: PointerEvent) => {
    if (e.button !== 0) return; // Solo click sinistro
    handleStart(e.clientX, e.clientY);
  }, [handleStart]);

  const onTouchStart = useCallback((e: TouchEvent) => {
    if (e.touches.length !== 1) return; // Solo un dito
    handleStart(e.touches[0].clientX, e.touches[0].clientY);
  }, [handleStart]);

  const onPointerMove = useCallback((e: PointerEvent) => {
    handleMove(e.clientX, e.clientY);
  }, [handleMove]);

  const onTouchMove = useCallback((e: TouchEvent) => {
    if (e.touches.length !== 1) return;
    handleMove(e.touches[0].clientX, e.touches[0].clientY);
  }, [handleMove]);

  const onPointerUp = useCallback(() => {
    handleEnd();
  }, [handleEnd]);

  const onTouchEnd = useCallback(() => {
    handleEnd();
  }, [handleEnd]);

  return {
    isSwiping,
    eventHandlers: {
      onPointerDown,
      onTouchStart,
      onPointerMove,
      onTouchMove,
      onPointerUp,
      onTouchEnd,
      onPointerCancel: onPointerUp,
      onTouchCancel: onTouchEnd
    }
  };
};

// Hook specializzato per direzioni specifiche
export const useHorizontalSwipe = (
  onSwipeLeft?: () => void,
  onSwipeRight?: () => void,
  config?: Omit<SwipeConfig, 'direction'>
) => {
  const handleSwipe = useCallback((swipe: SwipeEvent) => {
    if (swipe.direction === 'left') {
      onSwipeLeft?.();
    } else if (swipe.direction === 'right') {
      onSwipeRight?.();
    }
  }, [onSwipeLeft, onSwipeRight]);

  return useSwipe(handleSwipe, { ...config, direction: 'horizontal' });
};
Swipe threshold e velocity calculation
typescript
export class SwipeCalculator {
  static calculateVelocity(
    startX: number,
    startY: number,
    endX: number,
    endY: number,
    duration: number
  ): { velocityX: number; velocityY: number; overallVelocity: number } {
    const deltaX = endX - startX;
    const deltaY = endY - startY;
    const distance = Math.sqrt(deltaX * deltaX + deltaY * deltaY);
    
    return {
      velocityX: deltaX / duration,
      velocityY: deltaY / duration,
      overallVelocity: distance / duration
    };
  }

  static isSwipeValid(
    deltaX: number,
    deltaY: number,
    duration: number,
    config: {
      minDistance: number;
      maxDuration: number;
      minVelocity: number;
      directionLockThreshold?: number;
    }
  ): { isValid: boolean; direction: string | null } {
    const distanceX = Math.abs(deltaX);
    const distanceY = Math.abs(deltaY);
    const velocity = Math.sqrt(deltaX * deltaX + deltaY * deltaY) / duration;
    
    // Direzione lock per evitare diagonali accidentali
    if (config.directionLockThreshold) {
      const ratio = distanceX / distanceY;
      if (ratio < 1 / config.directionLockThreshold && 
          ratio > config.directionLockThreshold) {
        return { isValid: false, direction: null };
      }
    }
    
    // Determina direzione
    let direction: string | null = null;
    if (distanceX > distanceY && distanceX > config.minDistance) {
      direction = deltaX > 0 ? 'right' : 'left';
    } else if (distanceY > distanceX && distanceY > config.minDistance) {
      direction = deltaY > 0 ? 'down' : 'up';
    }
    
    const isValid = !!direction && 
                    duration <= config.maxDuration && 
                    velocity >= config.minVelocity;
    
    return { isValid, direction };
  }
}
Swipe to delete
typescript
import { useState, useRef, CSSProperties } from 'react';

export const useSwipeToDelete = (
  onDelete: () => Promise<void> | void,
  options: {
    deleteThreshold?: number;
    confirmThreshold?: number;
    animationDuration?: number;
    enableHaptic?: boolean;
  } = {}
) => {
  const {
    deleteThreshold = 100,
    confirmThreshold = 200,
    animationDuration = 300,
    enableHaptic = true
  } = options;

  const [state, setState] = useState({
    isSwiping: false,
    translateX: 0,
    isDeleting: false,
    showConfirm: false
  });

  const startX = useRef(0);
  const elementRef = useRef<HTMLDivElement>(null);

  const handleTouchStart = (e: React.TouchEvent | React.PointerEvent) => {
    const clientX = 'touches' in e ? e.touches[0].clientX : e.clientX;
    startX.current = clientX;
    setState(prev => ({ ...prev, isSwiping: true }));
  };

  const handleTouchMove = (e: React.TouchEvent | React.PointerEvent) => {
    if (!state.isSwiping) return;
    
    const clientX = 'touches' in e ? e.touches[0].clientX : e.clientX;
    const deltaX = clientX - startX.current;
    const translateX = Math.max(-confirmThreshold, Math.min(0, deltaX));
    
    setState(prev => ({
      ...prev,
      translateX,
      showConfirm: Math.abs(translateX) > deleteThreshold
    }));

    // Haptic feedback quando supera la soglia
    if (enableHaptic && Math.abs(translateX) > deleteThreshold) {
      navigator.vibrate?.(10);
    }
  };

  const handleTouchEnd = () => {
    if (!state.isSwiping) return;

    const shouldDelete = Math.abs(state.translateX) > confirmThreshold;
    const shouldConfirm = Math.abs(state.translateX) > deleteThreshold;

    if (shouldDelete) {
      setState(prev => ({ ...prev, isDeleting: true, translateX: -window.innerWidth }));
      setTimeout(async () => {
        await onDelete();
        setState({ isSwiping: false, translateX: 0, isDeleting: false, showConfirm: false });
      }, animationDuration);
    } else if (shouldConfirm) {
      // Torna alla posizione di conferma
      setState(prev => ({
        ...prev,
        translateX: -deleteThreshold - 10,
        showConfirm: true
      }));
    } else {
      // Torna alla posizione iniziale
      setState(prev => ({
        ...prev,
        translateX: 0,
        isSwiping: false,
        showConfirm: false
      }));
    }
  };

  const handleConfirmDelete = async () => {
    setState(prev => ({ ...prev, isDeleting: true, translateX: -window.innerWidth }));
    setTimeout(async () => {
      await onDelete();
      setState({ isSwiping: false, translateX: 0, isDeleting: false, showConfirm: false });
    }, animationDuration);
  };

  const handleCancel = () => {
    setState(prev => ({
      ...prev,
      translateX: 0,
      isSwiping: false,
      showConfirm: false
    }));
  };

  const containerStyle: CSSProperties = {
    position: 'relative',
    overflow: 'hidden',
    userSelect: 'none'
  };

  const contentStyle: CSSProperties = {
    transform: `translateX(${state.translateX}px)`,
    transition: state.isSwiping ? 'none' : `transform ${animationDuration}ms ease-out`,
    backgroundColor: state.showConfirm ? '#fee' : 'transparent',
    position: 'relative',
    zIndex: 1
  };

  const actionStyle: CSSProperties = {
    position: 'absolute',
    right: 0,
    top: 0,
    bottom: 0,
    display: 'flex',
    alignItems: 'center',
    padding: '0 20px',
    backgroundColor: '#dc3545',
    color: 'white',
    transform: `translateX(${state.translateX + 100}%)`,
    transition: 'transform 300ms ease-out'
  };

  return {
    state,
    elementRef,
    containerStyle,
    contentStyle,
    actionStyle,
    handlers: {
      onTouchStart: handleTouchStart,
      onTouchMove: handleTouchMove,
      onTouchEnd: handleTouchEnd,
      onTouchCancel: handleTouchEnd,
      onPointerDown: handleTouchStart,
      onPointerMove: handleTouchMove,
      onPointerUp: handleTouchEnd,
      onPointerCancel: handleTouchEnd
    },
    handleConfirmDelete,
    handleCancel
  };
};

// Componente SwipeToDelete
interface SwipeToDeleteProps {
  onDelete: () => Promise<void> | void;
  children: React.ReactNode;
  confirmText?: string;
  deleteText?: string;
}

export const SwipeToDelete: React.FC<SwipeToDeleteProps> = ({
  onDelete,
  children,
  confirmText = "Elimina",
  deleteText = "Rilascia per eliminare"
}) => {
  const {
    state,
    containerStyle,
    contentStyle,
    actionStyle,
    handlers,
    handleConfirmDelete,
    handleCancel
  } = useSwipeToDelete(onDelete);

  return (
    <div style={containerStyle}>
      <div 
        style={contentStyle}
        {...handlers}
      >
        {children}
        {state.showConfirm && !state.isDeleting && (
          <div style={{
            position: 'absolute',
            right: 10,
            top: '50%',
            transform: 'translateY(-50%)',
            color: '#dc3545',
            fontSize: '0.8rem',
            opacity: Math.max(0, (Math.abs(state.translateX) - 80) / 40)
          }}>
            {deleteText}
          </div>
        )}
      </div>
      
      {state.showConfirm && !state.isDeleting && (
        <div style={actionStyle}>
          <button
            onClick={handleConfirmDelete}
            style={{
              background: 'none',
              border: '1px solid white',
              color: 'white',
              padding: '8px 16px',
              borderRadius: '4px',
              cursor: 'pointer'
            }}
          >
            {confirmText}
          </button>
          <button
            onClick={handleCancel}
            style={{
              background: 'none',
              border: '1px solid white',
              color: 'white',
              padding: '8px 16px',
              borderRadius: '4px',
              marginLeft: '10px',
              cursor: 'pointer'
            }}
          >
            Annulla
          </button>
        </div>
      )}
    </div>
  );
};
Swipe to reveal actions
typescript
import { useState, useRef, CSSProperties } from 'react';

export interface SwipeAction {
  label: string;
  color: string;
  icon?: React.ReactNode;
  onAction: () => void;
  width?: number;
}

export const useSwipeActions = (
  actions: SwipeAction[],
  options: {
    threshold?: number;
    snapBackDuration?: number;
    maxSwipeDistance?: number;
  } = {}
) => {
  const {
    threshold = 60,
    snapBackDuration = 300,
    maxSwipeDistance = 200
  } = options;

  const [state, setState] = useState({
    isSwiping: false,
    translateX: 0,
    isActionVisible: false,
    activeActionIndex: -1
  });

  const startX = useRef(0);
  const startY = useRef(0);
  const elementRef = useRef<HTMLDivElement>(null);
  const totalActionsWidth = actions.reduce((sum, action) => sum + (action.width || 80), 0);

  const handleStart = (clientX: number, clientY: number) => {
    startX.current = clientX;
    startY.current = clientY;
    setState(prev => ({ ...prev, isSwiping: true }));
  };

  const handleMove = (clientX: number, clientY: number) => {
    if (!state.isSwiping) return;

    const deltaX = clientX - startX.current;
    const deltaY = clientY - startY.current;
    
    // Ignora movimenti prevalentemente verticali
    if (Math.abs(deltaY) > Math.abs(deltaX) * 2) {
      return;
    }

    // Limita lo swipe sinistro (per rivelare azioni a destra)
    const translateX = Math.max(-totalActionsWidth, Math.min(0, deltaX));
    
    // Determina quale azione è attiva in base alla distanza
    let accumulatedWidth = 0;
    let activeIndex = -1;
    
    for (let i = 0; i < actions.length; i++) {
      accumulatedWidth += actions[i].width || 80;
      if (Math.abs(translateX) <= accumulatedWidth) {
        activeIndex = i;
        break;
      }
    }

    setState(prev => ({
      ...prev,
      translateX,
      activeActionIndex: activeIndex,
      isActionVisible: Math.abs(translateX) > threshold
    }));

    // Haptic feedback quando supera la soglia
    if (Math.abs(translateX) > threshold) {
      navigator.vibrate?.(5);
    }
  };

  const handleEnd = () => {
    if (!state.isSwiping) return;

    const shouldSnapOpen = Math.abs(state.translateX) > threshold;
    const targetX = shouldSnapOpen ? -totalActionsWidth : 0;

    setState(prev => ({
      ...prev,
      translateX: targetX,
      isSwiping: false,
      isActionVisible: shouldSnapOpen
    }));

    // Esegui l'azione se è stata selezionata
    if (shouldSnapOpen && state.activeActionIndex >= 0) {
      setTimeout(() => {
        actions[state.activeActionIndex].onAction();
        // Ritorna alla posizione iniziale dopo l'azione
        setState(prev => ({ ...prev, translateX: 0, isActionVisible: false }));
      }, snapBackDuration);
    }
  };

  const handleActionClick = (index: number) => {
    actions[index].onAction();
    setState(prev => ({ ...prev, translateX: 0, isActionVisible: false }));
  };

  const handlers = {
    onTouchStart: (e: React.TouchEvent) => handleStart(e.touches[0].clientX, e.touches[0].clientY),
    onTouchMove: (e: React.TouchEvent) => handleMove(e.touches[0].clientX, e.touches[0].clientY),
    onTouchEnd: handleEnd,
    onTouchCancel: handleEnd,
    onPointerDown: (e: React.PointerEvent) => {
      if (e.button !== 0) return;
      handleStart(e.clientX, e.clientY);
    },
    onPointerMove: (e: React.PointerEvent) => handleMove(e.clientX, e.clientY),
    onPointerUp: handleEnd,
    onPointerCancel: handleEnd
  };

  const containerStyle: CSSProperties = {
    position: 'relative',
    overflow: 'hidden',
    userSelect: 'none'
  };

  const contentStyle: CSSProperties = {
    transform: `translateX(${state.translateX}px)`,
    transition: state.isSwiping ? 'none' : `transform ${snapBackDuration}ms ease-out`,
    position: 'relative',
    zIndex: 2,
    backgroundColor: 'white'
  };

  const actionsStyle: CSSProperties = {
    position: 'absolute',
    right: 0,
    top: 0,
    bottom: 0,
    display: 'flex',
    transform: `translateX(${totalActionsWidth + state.translateX}px)`,
    transition: state.isSwiping ? 'none' : `transform ${snapBackDuration}ms ease-out`
  };

  return {
    state,
    elementRef,
    containerStyle,
    contentStyle,
    actionsStyle,
    handlers,
    handleActionClick,
    totalActionsWidth
  };
};
§ PULL-TO-REFRESH
Implementation pattern
typescript
import { useState, useRef, useEffect, CSSProperties } from 'react';

export interface PullToRefreshProps {
  onRefresh: () => Promise<void>;
  pullingContent?: React.ReactNode;
  releasingContent?: React.ReactNode;
  refreshingContent?: React.ReactNode;
  pullThreshold?: number;
  maxPullDistance?: number;
  resistance?: number;
}

export const usePullToRefresh = (
  onRefresh: () => Promise<void>,
  options: {
    pullThreshold?: number;
    maxPullDistance?: number;
    resistance?: number;
    enabled?: boolean;
  } = {}
) => {
  const {
    pullThreshold = 80,
    maxPullDistance = 150,
    resistance = 2.5,
    enabled = true
  } = options;

  const [state, setState] = useState<'idle' | 'pulling' | 'releasing' | 'refreshing'>('idle');
  const [pullDistance, setPullDistance] = useState(0);
  
  const startY = useRef(0);
  const containerRef = useRef<HTMLDivElement>(null);
  const isAtTop = useRef(false);

  const checkIsAtTop = () => {
    if (!containerRef.current) return false;
    return containerRef.current.scrollTop === 0;
  };

  const handleTouchStart = (e: TouchEvent) => {
    if (!enabled || state === 'refreshing') return;
    
    isAtTop.current = checkIsAtTop();
    if (!isAtTop.current) return;
    
    startY.current = e.touches[0].clientY;
    setState('pulling');
  };

  const handleTouchMove = (e: TouchEvent) => {
    if (!enabled || !isAtTop.current || state === 'refreshing') return;
    
    const currentY = e.touches[0].clientY;
    const pullY = (currentY - startY.current) / resistance;
    
    if (pullY > 0) {
      e.preventDefault();
      const distance = Math.min(pullY, maxPullDistance);
      setPullDistance(distance);
      
      if (distance >= pullThreshold) {
        setState('releasing');
      } else {
        setState('pulling');
      }
    }
  };

  const handleTouchEnd = async () => {
    if (!enabled || !isAtTop.current) return;
    
    if (state === 'releasing' && pullDistance >= pullThreshold) {
      setState('refreshing');
      setPullDistance(pullThreshold);
      
      try {
        await onRefresh();
      } finally {
        setTimeout(() => {
          setState('idle');
          setPullDistance(0);
        }, 500);
      }
    } else {
      setState('idle');
      setPullDistance(0);
    }
    
    isAtTop.current = false;
  };

  useEffect(() => {
    const container = containerRef.current;
    if (!container || !enabled) return;

    container.addEventListener('touchstart', handleTouchStart, { passive: true });
    container.addEventListener('touchmove', handleTouchMove, { passive: false });
    container.addEventListener('touchend', handleTouchEnd);

    return () => {
      container.removeEventListener('touchstart', handleTouchStart);
      container.removeEventListener('touchmove', handleTouchMove);
      container.removeEventListener('touchend', handleTouchEnd);
    };
  }, [enabled, state, pullDistance]);

  const progress = Math.min(pullDistance / pullThreshold, 1);
  const showLoader = state === 'refreshing';

  return {
    containerRef,
    state,
    pullDistance,
    progress,
    showLoader,
    isPulling: state === 'pulling',
    isReleasing: state === 'releasing',
    isRefreshing: state === 'refreshing'
  };
};

// Componente PullToRefresh completo
export const PullToRefresh: React.FC<PullToRefreshProps> = ({
  onRefresh,
  pullingContent,
  releasingContent,
  refreshingContent,
  pullThreshold = 80,
  maxPullDistance = 150,
  resistance = 2.5,
  children
}) => {
  const [state, setState] = useState<'idle' | 'pulling' | 'releasing' | 'refreshing'>('idle');
  const [pullDistance, setPullDistance] = useState(0);
  
  const startY = useRef(0);
  const containerRef = useRef<HTMLDivElement>(null);
  const isAtTop = useRef(false);

  const handleTouchStart = (e: React.TouchEvent) => {
    if (state === 'refreshing') return;
    
    if (containerRef.current?.scrollTop === 0) {
      isAtTop.current = true;
      startY.current = e.touches[0].clientY;
      setState('pulling');
    }
  };

  const handleTouchMove = (e: React.TouchEvent) => {
    if (!isAtTop.current || state === 'refreshing') return;
    
    const currentY = e.touches[0].clientY;
    const pullY = (currentY - startY.current) / resistance;
    
    if (pullY > 0) {
      e.preventDefault();
      const distance = Math.min(pullY, maxPullDistance);
      setPullDistance(distance);
      
      if (distance >= pullThreshold) {
        setState('releasing');
      } else {
        setState('pulling');
      }
    }
  };

  const handleTouchEnd = async () => {
    if (!isAtTop.current) return;
    
    if (state === 'releasing' && pullDistance >= pullThreshold) {
      setState('refreshing');
      setPullDistance(pullThreshold);
      
      try {
        await onRefresh();
      } finally {
        setTimeout(() => {
          setState('idle');
          setPullDistance(0);
        }, 500);
      }
    } else {
      setState('idle');
      setPullDistance(0);
    }
    
    isAtTop.current = false;
  };

  const getIndicatorContent = () => {
    if (state === 'refreshing') {
      return refreshingContent || (
        <div style={{ textAlign: 'center', padding: '10px' }}>
          <div className="spinner" style={{
            width: '20px',
            height: '20px',
            border: '2px solid #f3f3f3',
            borderTop: '2px solid #3498db',
            borderRadius: '50%',
            animation: 'spin 1s linear infinite',
            margin: '0 auto'
          }} />
        </div>
      );
    }
    
    if (state === 'releasing') {
      return releasingContent || (
        <div style={{ textAlign: 'center', padding: '10px' }}>
          Rilascia per aggiornare...
        </div>
      );
    }
    
    if (state === 'pulling') {
      return pullingContent || (
        <div style={{ textAlign: 'center', padding: '10px' }}>
          Trascina per aggiornare...
        </div>
      );
    }
    
    return null;
  };

  const indicatorStyle: CSSProperties = {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    height: `${pullThreshold}px`,
    transform: `translateY(${pullDistance - pullThreshold}px)`,
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'center',
    overflow: 'hidden'
  };

  const contentStyle: CSSProperties = {
    transform: state === 'refreshing' ? `translateY(${pullThreshold}px)` : 'none',
    transition: state === 'refreshing' ? 'transform 0.3s ease' : 'none'
  };

  return (
    <div
      ref={containerRef}
      style={{
        position: 'relative',
        height: '100%',
        overflow: 'auto',
        WebkitOverflowScrolling: 'touch'
      }}
      onTouchStart={handleTouchStart}
      onTouchMove={handleTouchMove}
      onTouchEnd={handleTouchEnd}
    >
      <div style={indicatorStyle}>
        {getIndicatorContent()}
      </div>
      
      <div style={contentStyle}>
        {children}
      </div>
      
      <style jsx>{`
        @keyframes spin {
          0% { transform: rotate