# CATALOGO-ANIMATIONS-MOTION

CATALOGO ANIMATIONS-MOTION per Next.js 14 + Framer Motion
§ ANIMATION PRINCIPLES
12 Principles of Animation
tsx
// 1. Squash and Stretch
<motion.div
  animate={{
    scaleY: [1, 0.8, 1.2, 1],
    scaleX: [1, 1.2, 0.8, 1]
  }}
  transition={{ duration: 0.5 }}
/>

// 2. Anticipation
const anticipationVariants = {
  initial: { scale: 1, x: 0 },
  anticipate: { scale: 0.9, x: -10 },
  action: { scale: 1.2, x: 100 }
};

// 3. Staging & 4. Straight Ahead vs Pose to Pose
// Use variants for clean, staged animations
const stagedVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: {
      staggerChildren: 0.1,
      when: "beforeChildren"
    }
  }
};

// 5. Follow Through & 6. Slow In Slow Out
<motion.div
  animate={{ x: 100 }}
  transition={{
    type: "spring",
    stiffness: 100,
    damping: 10
  }}
/>

// 7. Arcs
<motion.div
  animate={{
    x: [0, 100, 100, 0],
    y: [0, 50, 150, 200]
  }}
  transition={{
    duration: 2,
    times: [0, 0.3, 0.7, 1]
  }}
/>

// 8. Secondary Action
const primaryVariants = {
  hover: { scale: 1.1 }
};
const secondaryVariants = {
  hover: { rotate: 5 }
};

// 9. Timing
const timingMap = {
  quick: { duration: 0.2 },
  normal: { duration: 0.5 },
  slow: { duration: 1 }
};

// 10. Exaggeration
<motion.div
  whileHover={{ scale: 1.3, rotate: 10 }}
/>

// 11. Solid Drawing & 12. Appeal
// Use 3D transforms and appealing easings
<motion.div
  style={{ transformPerspective: 1000 }}
  animate={{ rotateY: 180 }}
  transition={{ type: "spring" }}
/>
Easing Functions Explained
tsx
// Linear (no easing)
const linear = { ease: "linear" };

// Cubic bezier curves
const easings = {
  easeInOut: { ease: [0.42, 0, 0.58, 1] },
  easeOut: { ease: [0.22, 1, 0.36, 1] },
  easeIn: { ease: [0.36, 0, 0.66, -0.56] },
  bounce: { ease: [0.68, -0.55, 0.27, 1.55] }
};

// Spring physics
const springConfigs = {
  gentle: { type: "spring", stiffness: 100, damping: 20 },
  bouncy: { type: "spring", stiffness: 200, damping: 10 },
  stiff: { type: "spring", stiffness: 300, damping: 30 }
};

// Practical implementation
<motion.div
  animate={{ x: 100 }}
  transition={{
    duration: 0.5,
    ease: [0.43, 0.13, 0.23, 0.96] // Custom cubic bezier
  }}
/>
Duration Guidelines
tsx
// Duration scale for consistent timing
const durationScale = {
  ultrafast: 0.1,    // Button tap, microfeedback
  fast: 0.2,         // Hover states, toggles
  normal: 0.3,       // Modal enter/exit, card interactions
  slow: 0.5,         // Page transitions, major movements
  slower: 0.8,       // Background transitions, hero animations
  slowest: 1.2       // Loading sequences, dramatic reveals
};

// Context-based durations
const contextDurations = {
  microinteractions: {
    hover: 0.15,
    tap: 0.1,
    focus: 0.2
  },
  transitions: {
    route: 0.3,
    modal: 0.25,
    card: 0.4
  },
  feedback: {
    success: 0.5,
    error: 0.6,
    loading: 1
  }
};

// Implementation
<motion.div
  initial={false}
  animate={isActive ? "active" : "inactive"}
  variants={{
    active: { x: 0 },
    inactive: { x: -100 }
  }}
  transition={{ duration: contextDurations.transitions.card }}
/>
Performance Considerations
tsx
// GPU-accelerated properties (use these)
const gpuProperties = {
  transform: ["translateX", "translateY", "translateZ", "scale", "rotate"],
  opacity: true,
  filter: ["blur"]
};

// CPU-intensive properties (use sparingly)
const cpuProperties = {
  avoid: ["width", "height", "top", "left", "margin", "padding"],
  alternatives: {
    width: "scaleX",
    height: "scaleY",
    top: "translateY",
    left: "translateX"
  }
};

// Optimized animation component
const OptimizedAnimation = () => (
  <motion.div
    // ✅ GPU accelerated
    animate={{ 
      x: 100, 
      scale: 1.5,
      opacity: 0.5 
    }}
    // ❌ Avoid layout shifts
    // animate={{ width: "100px" }} 
    
    // Use will-change strategically
    style={{ willChange: "transform, opacity" }}
    
    // Reduce motion support
    transition={{ 
      duration: 0.3,
      ...(prefersReducedMotion && { duration: 0 })
    }}
  />
);

// Batch updates with React.memo
const MemoizedMotion = React.memo(({ animate }) => (
  <motion.div animate={animate} />
));

// Use transform templates
<motion.div
  style={{ x, y }}
  transformTemplate={({ x, y }) => 
    `translate3d(${x}px, ${y}px, 0)`
  }
/>
§ FRAMER MOTION FUNDAMENTALS
Motion Components
tsx
// Basic motion components
import { 
  motion, 
  m, // For reduced bundle size
  motion as m 
} from 'framer-motion';

// HTML elements with motion
const MotionElements = () => (
  <>
    <motion.div />
    <motion.span />
    <motion.h1 />
    <motion.p />
    <motion.a />
    <motion.button />
    <motion.img />
    <motion.svg />
    <motion.path />
    <motion.circle />
  </>
);

// Custom component with motion
const MotionCard = motion(MyCardComponent);

// With TypeScript
interface MotionButtonProps {
  isActive: boolean;
}

const MotionButton = motion.button<MotionButtonProps>();

// Functional component pattern
const AnimatedComponent: React.FC = () => {
  return (
    <motion.div
      initial={{ opacity: 0 }}
      animate={{ opacity: 1 }}
      exit={{ opacity: 0 }}
    />
  );
};
Animate Prop
tsx
// Direct animation values
<motion.div
  animate={{
    opacity: 1,
    x: 0,
    rotate: 360,
    scale: [1, 1.2, 1], // Keyframes
    backgroundColor: "#f00"
  }}
  transition={{
    duration: 0.5,
    times: [0, 0.5, 1], // For keyframes
    repeat: Infinity,
    repeatType: "reverse"
  }}
/>

// Dynamic animation based on state
const [isActive, setIsActive] = useState(false);

<motion.button
  animate={isActive ? "active" : "inactive"}
  variants={{
    active: { backgroundColor: "#3b82f6" },
    inactive: { backgroundColor: "#ef4444" }
  }}
/>

// Animation controls
const controls = useAnimation();

useEffect(() => {
  controls.start({
    x: 100,
    transition: { duration: 0.5 }
  });
}, []);

<motion.div animate={controls} />

// Keyframes with different value types
<motion.div
  animate={{
    x: [0, 100, 50, 200], // Numbers
    color: ["#000", "#f00", "#0f0", "#00f"], // Colors
    borderRadius: ["0%", "50%", "25%", "0%"] // Percentages
  }}
/>

// Conditional animations
<motion.div
  animate={isVisible ? "visible" : "hidden"}
  variants={{
    visible: { opacity: 1 },
    hidden: { opacity: 0 }
  }}
/>
Variants
tsx
// Basic variants
const variants = {
  hidden: { opacity: 0, y: 20 },
  visible: { 
    opacity: 1, 
    y: 0,
    transition: {
      duration: 0.5,
      ease: "easeOut"
    }
  },
  exit: {
    opacity: 0,
    y: -20,
    transition: { duration: 0.3 }
  }
};

// Nested variants with orchestration
const containerVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: {
      staggerChildren: 0.1,
      delayChildren: 0.2,
      when: "beforeChildren"
    }
  }
};

const childVariants = {
  hidden: { opacity: 0, x: -20 },
  visible: { 
    opacity: 1, 
    x: 0,
    transition: { type: "spring" }
  }
};

// Usage
<motion.div
  variants={containerVariants}
  initial="hidden"
  animate="visible"
  exit="hidden"
>
  {[1, 2, 3].map((item) => (
    <motion.div key={item} variants={childVariants}>
      Item {item}
    </motion.div>
  ))}
</motion.div>

// Dynamic variants
const getVariants = (delay: number) => ({
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: { delay }
  }
});

// Variants with custom props
const customVariants = {
  enter: (custom: number) => ({
    x: custom * 100,
    transition: { delay: custom * 0.1 }
  }),
  exit: { x: 0 }
};

<motion.div
  custom={index}
  variants={customVariants}
  initial="exit"
  animate="enter"
/>
AnimatePresence
tsx
import { AnimatePresence, motion } from 'framer-motion';

// Basic exit animations
const Modal = ({ isOpen }) => (
  <AnimatePresence>
    {isOpen && (
      <motion.div
        initial={{ opacity: 0, scale: 0.8 }}
        animate={{ opacity: 1, scale: 1 }}
        exit={{ opacity: 0, scale: 0.8 }}
        transition={{ duration: 0.3 }}
      >
        Modal Content
      </motion.div>
    )}
  </AnimatePresence>
);

// Wait for exit animations
<AnimatePresence mode="wait">
  {page === 1 && (
    <motion.div
      key="page1"
      initial={{ x: 300, opacity: 0 }}
      animate={{ x: 0, opacity: 1 }}
      exit={{ x: -300, opacity: 0 }}
    >
      Page 1
    </motion.div>
  )}
  {page === 2 && (
    <motion.div
      key="page2"
      initial={{ x: 300, opacity: 0 }}
      animate={{ x: 0, opacity: 1 }}
      exit={{ x: -300, opacity: 0 }}
    >
      Page 2
    </motion.div>
  )}
</AnimatePresence>

// Multiple exiting elements
<AnimatePresence>
  {items.map((item) => (
    <motion.div
      key={item.id}
      layout
      initial={{ opacity: 0, height: 0 }}
      animate={{ opacity: 1, height: "auto" }}
      exit={{ opacity: 0, height: 0 }}
      transition={{ duration: 0.3 }}
    >
      {item.name}
    </motion.div>
  ))}
</AnimatePresence>

// Custom exit animation with callback
const handleExitComplete = () => {
  console.log('Exit animation completed');
};

<AnimatePresence onExitComplete={handleExitComplete}>
  {isVisible && <motion.div exit={{ opacity: 0 }} />}
</AnimatePresence>

// Presence with initial={false}
<AnimatePresence initial={false}>
  {isOpen && <motion.div /* ... */ />}
</AnimatePresence>
Layout Animations
tsx
// Automatic layout animations
<motion.div layout>
  {/* Content that changes size */}
</motion.div>

// Layout animations with IDs
<motion.div layoutId="unique-id">
  Content
</motion.div>

// Shared layout transitions (like hero transitions)
const [selectedId, setSelectedId] = useState<string | null>(null);

return (
  <>
    {items.map(item => (
      <motion.div
        key={item.id}
        layoutId={item.id}
        onClick={() => setSelectedId(item.id)}
      >
        <h2>{item.title}</h2>
      </motion.div>
    ))}
    
    <AnimatePresence>
      {selectedId && (
        <motion.div layoutId={selectedId}>
          <motion.button onClick={() => setSelectedId(null)} />
          <motion.h2>Expanded Content</motion.h2>
        </motion.div>
      )}
    </AnimatePresence>
  </>
);

// Layout animations with shared elements
const SharedElementTransition = () => {
  const [expanded, setExpanded] = useState(false);

  return (
    <div>
      <motion.div
        layout
        style={{ borderRadius: expanded ? 20 : 8 }}
        onClick={() => setExpanded(!expanded)}
      >
        <motion.div
          layoutId="shared-image"
          style={{ 
            width: expanded ? 400 : 100,
            height: expanded ? 400 : 100 
          }}
        />
      </motion.div>
    </div>
  );
};

// Optimizing layout animations
<motion.div
  layout
  layoutRoot // Forces new layout projection
  layoutDependency={dependency} // Re-runs animation when dependency changes
  transition={{
    layout: {
      duration: 0.3,
      ease: "easeInOut"
    }
  }}
/>

// Disabling layout animations for specific properties
<motion.div
  layout
  animate={{
    opacity: [0, 1],
    // This won't animate with layout
    color: "#f00"
  }}
  transition={{
    layout: { duration: 0.3 },
    opacity: { duration: 0.5 }
  }}
/>
§ ENTRANCE ANIMATIONS
Fade In
tsx
// Basic fade in
const fadeIn = {
  initial: { opacity: 0 },
  animate: { opacity: 1 },
  exit: { opacity: 0 }
};

// Fade in with direction
const fadeInUp = {
  initial: { opacity: 0, y: 20 },
  animate: { opacity: 1, y: 0 },
  exit: { opacity: 0, y: -20 }
};

const fadeInDown = {
  initial: { opacity: 0, y: -20 },
  animate: { opacity: 1, y: 0 }
};

const fadeInLeft = {
  initial: { opacity: 0, x: -20 },
  animate: { opacity: 1, x: 0 }
};

const fadeInRight = {
  initial: { opacity: 0, x: 20 },
  animate: { opacity: 1, x: 0 }
};

// Staggered fade in
const container = {
  hidden: { opacity: 0 },
  show: {
    opacity: 1,
    transition: {
      staggerChildren: 0.1
    }
  }
};

const item = {
  hidden: { opacity: 0, y: 20 },
  show: { opacity: 1, y: 0 }
};

// Implementation
<motion.div
  variants={container}
  initial="hidden"
  animate="show"
>
  {items.map((item) => (
    <motion.div key={item.id} variants={item}>
      {item.name}
    </motion.div>
  ))}
</motion.div>

// Fade in on scroll
const FadeInOnScroll = ({ children }: { children: React.ReactNode }) => {
  const ref = useRef(null);
  const isInView = useInView(ref, { once: true });

  return (
    <motion.div
      ref={ref}
      initial={{ opacity: 0, y: 20 }}
      animate={isInView ? { opacity: 1, y: 0 } : {}}
      transition={{ duration: 0.5 }}
    >
      {children}
    </motion.div>
  );
};
Slide In (from any direction)
tsx
// Slide from left
const slideInLeft = {
  hidden: { x: -100, opacity: 0 },
  visible: { 
    x: 0, 
    opacity: 1,
    transition: {
      type: "spring",
      stiffness: 100
    }
  }
};

// Slide from right
const slideInRight = {
  hidden: { x: 100, opacity: 0 },
  visible: { 
    x: 0, 
    opacity: 1,
    transition: { duration: 0.5 }
  }
};

// Slide from bottom
const slideInUp = {
  hidden: { y: 100, opacity: 0 },
  visible: { 
    y: 0, 
    opacity: 1,
    transition: {
      type: "spring",
      damping: 15
    }
  }
};

// Slide from top
const slideInDown = {
  hidden: { y: -100, opacity: 0 },
  visible: { 
    y: 0, 
    opacity: 1,
    transition: { duration: 0.6 }
  }
};

// Diagonal slide
const slideInDiagonal = {
  hidden: { x: -50, y: 50, opacity: 0 },
  visible: { 
    x: 0, 
    y: 0, 
    opacity: 1 
  }
};

// Slide with overlay (like drawers)
const SlideInDrawer = ({ isOpen, direction = "left" }) => {
  const directions = {
    left: { x: "-100%" },
    right: { x: "100%" },
    top: { y: "-100%" },
    bottom: { y: "100%" }
  };

  return (
    <motion.div
      initial={{ opacity: 0 }}
      animate={{ opacity: isOpen ? 1 : 0 }}
    >
      <motion.div
        initial={directions[direction]}
        animate={{ 
          x: direction === "left" || direction === "right" ? 0 : undefined,
          y: direction === "top" || direction === "bottom" ? 0 : undefined
        }}
        exit={directions[direction]}
        transition={{ type: "spring", damping: 25 }}
        style={{
          position: "fixed",
          [direction]: 0,
          width: direction === "left" || direction === "right" ? 300 : "100%",
          height: direction === "top" || direction === "bottom" ? 300 : "100%"
        }}
      >
        Drawer Content
      </motion.div>
    </motion.div>
  );
};

// Staggered slide in list
const StaggeredSlideList = ({ items }) => (
  <motion.ul
    initial="hidden"
    animate="visible"
    variants={{
      hidden: { opacity: 0 },
      visible: {
        opacity: 1,
        transition: {
          staggerChildren: 0.1,
          delayChildren: 0.2
        }
      }
    }}
  >
    {items.map((item, index) => (
      <motion.li
        key={item.id}
        variants={{
          hidden: { 
            x: index % 2 === 0 ? -50 : 50, 
            opacity: 0 
          },
          visible: { 
            x: 0, 
            opacity: 1,
            transition: { type: "spring" }
          }
        }}
      >
        {item.text}
      </motion.li>
    ))}
  </motion.ul>
);
Scale In
tsx
// Basic scale in
const scaleIn = {
  initial: { scale: 0, opacity: 0 },
  animate: { 
    scale: 1, 
    opacity: 1,
    transition: {
      type: "spring",
      stiffness: 200,
      damping: 20
    }
  },
  exit: { 
    scale: 0, 
    opacity: 0,
    transition: { duration: 0.2 }
  }
};

// Scale in from center
const scaleInCenter = {
  hidden: { scale: 0, rotate: -180 },
  visible: {
    scale: 1,
    rotate: 0,
    transition: {
      type: "spring",
      stiffness: 260,
      damping: 20
    }
  }
};

// Scale in with bounce
const scaleInBounce = {
  initial: { scale: 0 },
  animate: {
    scale: [0, 1.2, 0.9, 1],
    transition: {
      duration: 0.6,
      times: [0, 0.6, 0.8, 1]
    }
  }
};

// Scale in from specific origin
const ScaleInFromCorner = () => (
  <motion.div
    initial={{ scale: 0, opacity: 0, originX: 0, originY: 0 }}
    animate={{ scale: 1, opacity: 1 }}
    transition={{
      type: "spring",
      stiffness: 150
    }}
    style={{ transformOrigin: "top left" }}
  >
    Content
  </motion.div>
);

// Sequential scale in (pop effect)
const PopInSequence = () => {
  const letters = "POP IN".split("");
  
  return (
    <div style={{ display: "flex", gap: "4px" }}>
      {letters.map((letter, i) => (
        <motion.span
          key={i}
          initial={{ scale: 0, rotate: -180 }}
          animate={{ scale: 1, rotate: 0 }}
          transition={{
            type: "spring",
            stiffness: 300,
            damping: 15,
            delay: i * 0.1
          }}
          style={{ display: "inline-block" }}
        >
          {letter}
        </motion.span>
      ))}
    </div>
  );
};

// Scale in on hover parent
const ScaleInOnHover = () => (
  <motion.div whileHover="hover">
    <motion.div
      variants={{
        hover: { scale: 1.1 }
      }}
      transition={{ type: "spring" }}
    >
      Hover me
    </motion.div>
    
    <motion.div
      initial={{ scale: 0 }}
      variants={{
        hover: { scale: 1 }
      }}
      transition={{ delay: 0.1 }}
    >
      Appears on hover
    </motion.div>
  </motion.div>
);
Blur In
tsx
// Basic blur in
const blurIn = {
  initial: { 
    opacity: 0,
    filter: "blur(10px)"
  },
  animate: { 
    opacity: 1,
    filter: "blur(0px)",
    transition: {
      duration: 0.5,
      ease: "easeOut"
    }
  },
  exit: {
    opacity: 0,
    filter: "blur(10px)",
    transition: { duration: 0.3 }
  }
};

// Blur in with slide
const blurInUp = {
  hidden: {
    opacity: 0,
    filter: "blur(10px)",
    y: 20
  },
  visible: {
    opacity: 1,
    filter: "blur(0px)",
    y: 0,
    transition: {
      duration: 0.6,
      ease: [0.22, 1, 0.36, 1]
    }
  }
};

// Staggered blur in
const staggeredBlurIn = {
  container: {
    hidden: { opacity: 0 },
    show: {
      opacity: 1,
      transition: {
        staggerChildren: 0.15,
        delayChildren: 0.1
      }
    }
  },
  item: {
    hidden: { 
      opacity: 0,
      filter: "blur(8px)",
      y: 10
    },
    show: {
      opacity: 1,
      filter: "blur(0px)",
      y: 0,
      transition: {
        duration: 0.4
      }
    }
  }
};

// Blur in text
const BlurInText = ({ text }: { text: string }) => {
  const words = text.split(" ");
  
  return (
    <motion.div
      variants={staggeredBlurIn.container}
      initial="hidden"
      animate="show"
      style={{ display: "flex", flexWrap: "wrap", gap: "8px" }}
    >
      {words.map((word, i) => (
        <motion.span
          key={i}
          variants={staggeredBlurIn.item}
          style={{ display: "inline-block" }}
        >
          {word}
        </motion.span>
      ))}
    </motion.div>
  );
};

// Performance optimized blur (use sparingly)
const OptimizedBlurIn = () => (
  <motion.div
    initial={{ 
      opacity: 0,
      // Use opacity for fallback on low-end devices
      filter: "blur(0px)" 
    }}
    animate={{ 
      opacity: 1,
      filter: "blur(10px)" // Animate to blur for performance
    }}
    style={{
      // Will-change hint for browsers
      willChange: "filter, opacity",
      // Fallback for reduced motion
      "@media (prefers-reduced-motion: reduce)": {
        transition: "opacity 0.3s ease"
      }
    }}
  />
);
Stagger Children
tsx
// Basic stagger
const staggerContainer = {
  hidden: { opacity: 0 },
  show: {
    opacity: 1,
    transition: {
      staggerChildren: 0.1,
      delayChildren: 0.3
    }
  }
};

const staggerItem = {
  hidden: { opacity: 0, y: 20 },
  show: { opacity: 1, y: 0 }
};

// Different stagger directions
const StaggerGrid = ({ items }: { items: any[] }) => {
  const container = {
    hidden: { opacity: 0 },
    show: {
      opacity: 1,
      transition: {
        staggerChildren: 0.05,
        staggerDirection: 1 // 1 = forward, -1 = backward
      }
    }
  };

  const item = {
    hidden: { opacity: 0, scale: 0.8 },
    show: { opacity: 1, scale: 1 }
  };

  return (
    <motion.div
      variants={container}
      initial="hidden"
      animate="show"
      style={{ display: "grid", gridTemplateColumns: "repeat(3, 1fr)" }}
    >
      {items.map((item, i) => (
        <motion.div key={i} variants={item}>
          Item {i + 1}
        </motion.div>
      ))}
    </motion.div>
  );
};

// Stagger with different animations
const mixedStagger = {
  container: {
    hidden: { opacity: 0 },
    show: {
      opacity: 1,
      transition: {
        staggerChildren: 0.1,
        when: "beforeChildren"
      }
    }
  },
  fade: {
    hidden: { opacity: 0 },
    show: { opacity: 1 }
  },
  slide: {
    hidden: { x: -20, opacity: 0 },
    show: { x: 0, opacity: 1 }
  },
  scale: {
    hidden: { scale: 0, opacity: 0 },
    show: { scale: 1, opacity: 1 }
  }
};

// Dynamic stagger based on data
const DynamicStaggerList = ({ items }: { items: any[] }) => {
  const container = {
    hidden: { opacity: 0 },
    show: {
      opacity: 1,
      transition: {
        staggerChildren: 0.5 / items.length, // Even distribution over 0.5s
        when: "beforeChildren"
      }
    }
  };

  return (
    <motion.ul variants={container} initial="hidden" animate="show">
      {items.map((item) => (
        <motion.li
          key={item.id}
          variants={staggerItem}
          custom={item.index}
          transition={{
            delay: item.index * 0.05 // Additional per-item delay
          }}
        >
          {item.name}
        </motion.li>
      ))}
    </motion.ul>
  );
};

// Stagger with exit animations
const StaggerWithExit = () => {
  const [items, setItems] = useState([1, 2, 3, 4, 5]);

  const container = {
    hidden: { opacity: 0 },
    show: {
      opacity: 1,
      transition: {
        staggerChildren: 0.1
      }
    },
    exit: {
      opacity: 0,
      transition: {
        staggerChildren: 0.05,
        staggerDirection: -1
      }
    }
  };

  const item = {
    hidden: { opacity: 0, x: -20 },
    show: { opacity: 1, x: 0 },
    exit: { opacity: 0, x: 20 }
  };

  return (
    <AnimatePresence>
      <motion.div variants={container} initial="hidden" animate="show" exit="exit">
        {items.map((item) => (
          <motion.div key={item} variants={item}>
            Item {item}
            <button onClick={() => setItems(items.filter(i => i !== item))}>
              Remove
            </button>
          </motion.div>
        ))}
      </motion.div>
    </AnimatePresence>
  );
};
§ EXIT ANIMATIONS
Fade Out
tsx
// Basic fade out
const fadeOut = {
  exit: { 
    opacity: 0,
    transition: { duration: 0.3 }
  }
};

// Fade out with direction
const fadeOutDown = {
  exit: { 
    opacity: 0, 
    y: 20,
    transition: {
      duration: 0.4,
      ease: "easeIn"
    }
  }
};

const fadeOutUp = {
  exit: { 
    opacity: 0, 
    y: -20 
  }
};

const fadeOutLeft = {
  exit: { 
    opacity: 0, 
    x: -20 
  }
};

const fadeOutRight = {
  exit: { 
    opacity: 0, 
    x: 20 
  }
};

// Fade out with scale
const fadeOutScale = {
  exit: { 
    opacity: 0, 
    scale: 0.8,
    transition: { duration: 0.3 }
  }
};

// Usage with AnimatePresence
const FadingComponent = ({ isVisible }: { isVisible: boolean }) => (
  <AnimatePresence>
    {isVisible && (
      <motion.div
        initial={{ opacity: 0 }}
        animate={{ opacity: 1 }}
        exit={fadeOut}
      >
        Content
      </motion.div>
    )}
  </AnimatePresence>
);

// Staggered fade out
const staggeredFadeOut = {
  exit: {
    opacity: 0,
    transition: {
      when: "afterChildren",
      staggerChildren: 0.05,
      staggerDirection: -1
    }
  }
};
Slide Out
tsx
// Slide to left
const slideOutLeft = {
  exit: { 
    x: -100, 
    opacity: 0,
    transition: {
      type: "spring",
      stiffness: 100,
      damping: 20
    }
  }
};

// Slide to right
const slideOutRight = {
  exit: { 
    x: 100, 
    opacity: 0 
  }
};

// Slide down
const slideOutDown = {
  exit: { 
    y: 100, 
    opacity: 0,
    transition: { duration: 0.5 }
  }
};

// Slide up
const slideOutUp = {
  exit: { 
    y: -100, 
    opacity: 0 
  }
};

// Slide out with transform origin
const SlideOutModal = ({ isOpen, onClose }: { isOpen: boolean, onClose: () => void }) => (
  <AnimatePresence>
    {isOpen && (
      <>
        {/* Backdrop */}
        <motion.div
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
          onClick={onClose}
          style={{
            position: "fixed",
            top: 0,
            left: 0,
            right: 0,
            bottom: 0,
            backgroundColor: "rgba(0,0,0,0.5)"
          }}
        />
        
        {/* Modal */}
        <motion.div
          initial={{ y: "100%" }}
          animate={{ y: 0 }}
          exit={{ y: "100%" }}
          transition={{ type: "spring", damping: 25 }}
          style={{
            position: "fixed",
            bottom: 0,
            left: 0,
            right: 0,
            background: "white",
            borderTopLeftRadius: "20px",
            borderTopRightRadius: "20px",
            padding: "20px"
          }}
        >
          Modal Content
        </motion.div>
      </>
    )}
  </AnimatePresence>
);

// Slide out list item
const SlideOutListItem = ({ item, onRemove }: { item: any, onRemove: () => void }) => (
  <motion.li
    layout
    initial={{ opacity: 0, x: -50 }}
    animate={{ opacity: 1, x: 0 }}
    exit={{ opacity: 0, x: 100 }}
    transition={{ duration: 0.3 }}
  >
    {item.name}
    <button onClick={onRemove}>×</button>
  </motion.li>
);
Scale Out
tsx
// Scale out to nothing
const scaleOut = {
  exit: { 
    scale: 0, 
    opacity: 0,
    transition: {
      duration: 0.3,
      ease: "easeIn"
    }
  }
};

// Scale out with bounce
const scaleOutBounce = {
  exit: {
    scale: [1, 1.1, 0],
    opacity: [1, 0.8, 0],
    transition: {
      duration: 0.5,
      times: [0, 0.3, 1]
    }
  }
};

// Scale out to point
const ScaleOutToCursor = () => {
  const [isVisible, setIsVisible] = useState(true);
  const [exitPoint, setExitPoint] = useState({ x: 0, y: 0 });

  const handleClick = (e: React.MouseEvent) => {
    const rect = e.currentTarget.getBoundingClientRect();
    setExitPoint({
      x: e.clientX - rect.left,
      y: e.clientY - rect.top
    });
   

════════════════════════════════════════════════════════════
FIGMA CATALOG: BATCH2-15-ANIMATIONS-MOTION
Prompt ID: 15 / 19
Parte: 2
Exported: 2026-02-06T16:59:52.754Z
Characters: 2510
════════════════════════════════════════════════════════════

riants}
            style={{
              background: '#3b82f6',
              borderRadius: 8
            }}
          >
            Item {item}
          </motion.div>
        ))}
      </motion.div>
    </div>
  );
};

// Background morph
const BackgroundMorph = () => {
  const [page, setPage] = useState('home');

  const backgroundVariants = {
    home: {
      background: "linear-gradient(45deg, #3b82f6, #8b5cf6)",
      clipPath: "circle(70% at 50% 50%)",
      transition: { duration: 0.8 }
    },
    about: {
      background: "linear-gradient(45deg, #10b981, #3b82f6)",
      clipPath: "polygon(50% 0%, 0% 100%, 100% 100%)",
      transition: { duration: 0.8 }
    },
    contact: {
      background: "linear-gradient(45deg, #f59e0b, #ef4444)",
      clipPath: "polygon(20% 0%, 80% 0%, 100% 100%, 0% 100%)",
      transition: { duration: 0.8 }
    }
  };

  return (
    <div>
      <div style={{ display: 'flex', gap: '10px', marginBottom: '20px' }}>
        <button onClick={() => setPage('home')}>Home</button>
        <button onClick={() => setPage('about')}>About</button>
        <button onClick={() => setPage('contact')}>Contact</button>
      </div>

      <motion.div
        style={{
          width: '100%',
          height: '400px',
          borderRadius: 8
        }}
        animate={page}
        variants={backgroundVariants}
      />
    </div>
  );
};
Loading Transitions
tsx
// Page load transition
const PageLoadTransition = ({ children }: { children: ReactNode }) => {
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    // Simulate loading
    const timer = setTimeout(() => setIsLoading(false), 1000);
    return () => clearTimeout(timer);
  }, []);

  return (
    <AnimatePresence mode="wait">
      {isLoading ? (
        <motion.div
          key="loader"
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
          transition={{ duration: 0.3 }}
          style={{
            display: 'flex',
            alignItems: 'center',
            justifyContent: 'center',
            height: '100vh'
          }}
        >
          <motion.div
            animate={{ rotate: 360 }}
            transition={{ duration: 1, repeat: Infinity, ease: "linear" }}
            style={{
              width: 50,
              height: 50,
              border: '4px solid #3b82f6',
              borderTopColor: 'transparent',
              borderRadius: '50%'
            }}
          />