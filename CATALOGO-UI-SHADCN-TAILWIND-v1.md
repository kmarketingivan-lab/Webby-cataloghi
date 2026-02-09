# CATALOGO-UI-SHADCN-TAILWIND-v1

> **Dominio**: UI Components
> **Stack**: Next.js, React, TypeScript, Tailwind CSS, shadcn/ui
> **Versione**: 1.0
> **Data**: 2026-02-04

---

§ 1. INDICE

| # | Sezione | Path |
|---|---------|------|
| 35 | [src/lib/utils/cn.ts](#35-src-lib-utils-cn-ts) | `src/lib/utils/cn.ts` |
| 1 | [src/components/ui/button.tsx](#1-src-components-ui-button-tsx) | `src/components/ui/button.tsx` |
| 2 | [src/components/ui/input.tsx](#2-src-components-ui-input-tsx) | `src/components/ui/input.tsx` |
| 3 | [src/components/ui/textarea.tsx](#3-src-components-ui-textarea-tsx) | `src/components/ui/textarea.tsx` |
| 4 | [src/components/ui/select.tsx](#4-src-components-ui-select-tsx) | `src/components/ui/select.tsx` |
| 5 | [src/components/ui/checkbox.tsx](#5-src-components-ui-checkbox-tsx) | `src/components/ui/checkbox.tsx` |
| 6 | [src/components/ui/radio-group.tsx](#6-src-components-ui-radio-group-tsx) | `src/components/ui/radio-group.tsx` |
| 7 | [src/components/ui/switch.tsx](#7-src-components-ui-switch-tsx) | `src/components/ui/switch.tsx` |
| 8 | [src/components/ui/slider.tsx](#8-src-components-ui-slider-tsx) | `src/components/ui/slider.tsx` |
| 9 | [src/components/ui/label.tsx](#9-src-components-ui-label-tsx) | `src/components/ui/label.tsx` |
| 10 | [src/components/ui/form.tsx](#10-src-components-ui-form-tsx) | `src/components/ui/form.tsx` |
| 11 | [src/components/ui/card.tsx](#11-src-components-ui-card-tsx) | `src/components/ui/card.tsx` |
| 12 | [src/components/ui/badge.tsx](#12-src-components-ui-badge-tsx) | `src/components/ui/badge.tsx` |
| 13 | [src/components/ui/avatar.tsx](#13-src-components-ui-avatar-tsx) | `src/components/ui/avatar.tsx` |
| 14 | [src/components/ui/alert.tsx](#14-src-components-ui-alert-tsx) | `src/components/ui/alert.tsx` |
| 15 | [src/components/ui/dialog.tsx](#15-src-components-ui-dialog-tsx) | `src/components/ui/dialog.tsx` |
| 16 | [src/components/ui/sheet.tsx](#16-src-components-ui-sheet-tsx) | `src/components/ui/sheet.tsx` |
| 17 | [src/components/ui/dropdown-menu.tsx](#17-src-components-ui-dropdown-menu-tsx) | `src/components/ui/dropdown-menu.tsx` |
| 18 | [src/components/ui/popover.tsx](#18-src-components-ui-popover-tsx) | `src/components/ui/popover.tsx` |
| 19 | [src/components/ui/tooltip.tsx](#19-src-components-ui-tooltip-tsx) | `src/components/ui/tooltip.tsx` |
| 20 | [src/components/ui/tabs.tsx](#20-src-components-ui-tabs-tsx) | `src/components/ui/tabs.tsx` |
| 21 | [src/components/ui/accordion.tsx](#21-src-components-ui-accordion-tsx) | `src/components/ui/accordion.tsx` |
| 22 | [src/components/ui/table.tsx](#22-src-components-ui-table-tsx) | `src/components/ui/table.tsx` |
| 23 | [src/components/ui/pagination.tsx](#23-src-components-ui-pagination-tsx) | `src/components/ui/pagination.tsx` |
| 24 | [src/components/ui/breadcrumb.tsx](#24-src-components-ui-breadcrumb-tsx) | `src/components/ui/breadcrumb.tsx` |
| 25 | [src/components/ui/separator.tsx](#25-src-components-ui-separator-tsx) | `src/components/ui/separator.tsx` |
| 26 | [src/components/ui/skeleton.tsx](#26-src-components-ui-skeleton-tsx) | `src/components/ui/skeleton.tsx` |
| 27 | [src/components/ui/spinner.tsx](#27-src-components-ui-spinner-tsx) | `src/components/ui/spinner.tsx` |

---

§ 35. SRC/LIB/UTILS/CN.TS

typescript
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

/**
 * Utility function to conditionally join Tailwind CSS classes and merge them efficiently.
 * It uses `clsx` for conditional class joining and `tailwind-merge` to resolve conflicts.
 *
 * @param inputs - An array of `ClassValue` (strings, objects, arrays) to be processed.
 * @returns A single string containing the merged and optimized Tailwind CSS classes.
 */
export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

---

§ 1. SRC/COMPONENTS/UI/BUTTON.TSX

typescript
import * as React from "react";
import { Slot } from "@radix-ui/react-slot";
import { cva, type VariantProps } from "class-variance-authority";
import { Loader2 } from "lucide-react"; // Assuming lucide-react for icons

import { cn } from "@/lib/utils/cn";

/**
 * Defines the variants and sizes for the Button component using CVA.
 * @see https://www.npmjs.com/package/class-variance-authority
 */
const buttonVariants = cva(
  "inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive:
          "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        outline:
          "border border-input bg-background hover:bg-accent hover:text-accent-foreground",
        secondary:
          "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);

/**
 * Props for the Button component.
 * Extends standard HTML button attributes and CVA variant props.
 */
export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  /**
   * If true, the button will render its child as a slot, allowing for custom elements.
   * @default false
   */
  asChild?: boolean;
  /**
   * If true, the button will display a loading spinner and be disabled.
   * @default false
   */
  loading?: boolean;
}

/**
 * A versatile button component with various styles, sizes, and states.
 * It supports `asChild` for custom rendering and a `loading` state.
 *
 * @component
 * @example
 * <Button>Click me</Button>
 * <Button variant="destructive" size="lg">Delete</Button>
 * <Button loading>Submitting</Button>
 * <Button asChild><Link to="/dashboard">Dashboard</Link></Button>
 */
const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, loading, disabled, children, ...props }, ref) => {
    const Comp = asChild ? Slot : "button";
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        disabled={disabled || loading}
        {...props}
      >
        {loading && <Loader2 className="mr-2 h-4 w-4 animate-spin" aria-label="Loading..." />}
        {children}
      </Comp>
    );
  }
);
Button.displayName = "Button";

export { Button, buttonVariants };

---

§ 2. SRC/COMPONENTS/UI/INPUT.TSX

typescript
import * as React from "react";
import { cva, type VariantProps } from "class-variance-authority";
import { X, Search } from "lucide-react"; // Assuming lucide-react for icons

import { cn } from "@/lib/utils/cn";
import { Button } from "./button"; // Assuming Button component exists

/**
 * Defines the base styles for the Input component using CVA.
 */
const inputVariants = cva(
  "flex h-10 w-full rounded-md border border-input bg-background px-3 py-2 text-sm ring-offset-background file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50",
  {
    variants: {
      error: {
        true: "border-destructive focus-visible:ring-destructive",
      },
    },
    defaultVariants: {
      error: false,
    },
  }
);

/**
 * Props for the Input component.
 * Extends standard HTML input attributes and CVA variant props.
 */
export interface InputProps
  extends React.InputHTMLAttributes<HTMLInputElement>,
    VariantProps<typeof inputVariants> {
  /**
   * An icon to display on the left side of the input.
   */
  leftIcon?: React.ReactNode;
  /**
   * An icon to display on the right side of the input.
   */
  rightIcon?: React.ReactNode;
  /**
   * If true, a clear button will be displayed when the input has a value.
   * @default false
   */
  clearable?: boolean;
}

/**
 * A customizable input component supporting various types, states, icons, and an optional clear button.
 *
 * @component
 * @example
 * <Input placeholder="Email" type="email" />
 * <Input placeholder="Search..." leftIcon={<Search />} clearable />
 * <Input placeholder="Password" type="password" error />
 */
const Input = React.forwardRef<HTMLInputElement, InputProps>(
  (
    {
      className,
      type,
      error,
      disabled,
      leftIcon,
      rightIcon,
      clearable,
      value: propValue,
      onChange,
      ...props
    },
    ref
  ) => {
    const [internalValue, setInternalValue] = React.useState(propValue || "");
    const isControlled = propValue !== undefined;

    const currentValue = isControlled ? propValue : internalValue;

    const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
      if (!isControlled) {
        setInternalValue(e.target.value);
      }
      onChange?.(e);
    };

    const handleClear = () => {
      if (!isControlled) {
        setInternalValue("");
      }
      // Manually trigger a change event for external listeners if uncontrolled
      if (!isControlled && onChange) {
        const event = new Event('change', { bubbles: true });
        Object.defineProperty(event, 'target', { writable: true, value: ref?.current });
        Object.defineProperty(event.target, 'value', { writable: true, value: '' });
        onChange(event as React.ChangeEvent<HTMLInputElement>);
      }
    };

    const showClearButton = clearable && !disabled && currentValue;

    return (
      <div
        className={cn(
          "relative flex items-center",
          disabled && "cursor-not-allowed opacity-50"
        )}
      >
        {leftIcon && (
          <div className="absolute left-3 text-muted-foreground peer-disabled:cursor-not-allowed">
            {leftIcon}
          </div>
        )}
        <input
          type={type}
          className={cn(
            inputVariants({ error, className }),
            leftIcon && "pl-10",
            (rightIcon || showClearButton) && "pr-10"
          )}
          ref={ref}
          disabled={disabled}
          value={currentValue}
          onChange={handleChange}
          {...props}
        />
        {(rightIcon || showClearButton) && (
          <div className="absolute right-3 text-muted-foreground peer-disabled:cursor-not-allowed">
            {showClearButton ? (
              <Button
                type="button"
                variant="ghost"
                size="icon"
                onClick={handleClear}
                disabled={disabled}
                className="h-6 w-6 text-muted-foreground hover:bg-transparent hover:text-foreground"
                aria-label="Clear input"
              >
                <X className="h-4 w-4" />
              </Button>
            ) : (
              rightIcon
            )}
          </div>
        )}
      </div>
    );
  }
);
Input.displayName = "Input";

export { Input };

---

§ 3. SRC/COMPONENTS/UI/TEXTAREA.TSX

typescript
import * as React from "react";
import { cn } from "@/lib/utils/cn";
import { cva, type VariantProps } from "class-variance-authority";

/**
 * Defines the base styles for the Textarea component using CVA.
 */
const textareaVariants = cva(
  "flex min-h-[80px] w-full rounded-md border border-input bg-background px-3 py-2 text-sm ring-offset-background placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50",
  {
    variants: {
      error: {
        true: "border-destructive focus-visible:ring-destructive",
      },
    },
    defaultVariants: {
      error: false,
    },
  }
);

/**
 * Props for the Textarea component.
 * Extends standard HTML textarea attributes and CVA variant props.
 */
export interface TextareaProps
  extends React.TextareaHTMLAttributes<HTMLTextAreaElement>,
    VariantProps<typeof textareaVariants> {
  /**
   * If true, the textarea will automatically adjust its height to fit the content.
   * @default false
   */
  autoResize?: boolean;
  /**
   * The maximum number of characters allowed in the textarea.
   * If provided, a character counter will be displayed.
   */
  maxLength?: number;
  /**
   * If true, the textarea will display an error state.
   * @default false
   */
  error?: boolean;
}

/**
 * A customizable textarea component with optional auto-resizing, character counter, and error state.
 *
 * @component
 * @example
 * <Textarea placeholder="Your message..." />
 * <Textarea autoResize maxLength={100} />
 * <Textarea error placeholder="This field is required" />
 */
const Textarea = React.forwardRef<HTMLTextAreaElement, TextareaProps>(
  ({ className, autoResize, maxLength, error, value, onChange, ...props }, ref) => {
    const textareaRef = React.useRef<HTMLTextAreaElement>(null);
    const combinedRef = React.useCallback(
      (node: HTMLTextAreaElement) => {
        if (node) {
          textareaRef.current = node;
          if (typeof ref === "function") {
            ref(node);
          } else if (ref) {
            ref.current = node;
          }
        }
      },
      [ref]
    );

    const [currentValue, setCurrentValue] = React.useState(value || "");

    React.useEffect(() => {
      setCurrentValue(value || "");
    }, [value]);

    React.useLayoutEffect(() => {
      if (autoResize && textareaRef.current) {
        textareaRef.current.style.height = "auto";
        textareaRef.current.style.height = `${textareaRef.current.scrollHeight}px`;
      }
    }, [currentValue, autoResize]);

    const handleChange = (e: React.ChangeEvent<HTMLTextAreaElement>) => {
      setCurrentValue(e.target.value);
      onChange?.(e);
    };

    const charCount = String(currentValue).length;
    const showCharCounter = maxLength !== undefined;

    return (
      <div className="relative w-full">
        <textarea
          className={cn(textareaVariants({ error, className }))}
          ref={combinedRef}
          value={currentValue}
          onChange={handleChange}
          maxLength={maxLength}
          {...props}
        />
        {showCharCounter && (
          <div
            className={cn(
              "absolute bottom-2 right-3 text-xs text-muted-foreground",
              charCount > (maxLength || 0) && "text-destructive"
            )}
            aria-live="polite"
          >
            {charCount} / {maxLength}
          </div>
        )}
      </div>
    );
  }
);
Textarea.displayName = "Textarea";

export { Textarea };

---

§ 4. SRC/COMPONENTS/UI/SELECT.TSX

typescript
import * as React from "react";
import * as SelectPrimitive from "@radix-ui/react-select";
import { Check, ChevronDown, Search } from "lucide-react";

import { cn } from "@/lib/utils/cn";
import { Input } from "./input"; // Assuming Input component exists

/**
 * Select component root.
 * @see https://www.radix-ui.com/primitives/docs/components/select
 */
const Select = SelectPrimitive.Root;

/**
 * Select component trigger.
 * Displays the currently selected value and an arrow icon.
 */
const SelectTrigger = React.forwardRef<
  React.ElementRef<typeof SelectPrimitive.Trigger>,
  React.ComponentPropsWithoutRef<typeof SelectPrimitive.Trigger>
>(({ className, children, ...props }, ref) => (
  <SelectPrimitive.Trigger
    ref={ref}
    className={cn(
      "flex h-10 w-full items-center justify-between rounded-md border border-input bg-background px-3 py-2 text-sm ring-offset-background placeholder:text-muted-foreground focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50 [&>span]:line-clamp-1",
      className
    )}
    {...props}
  >
    {children}
    <SelectPrimitive.Icon asChild>
      <ChevronDown className="h-4 w-4 opacity-50" />
    </SelectPrimitive.Icon>
  </SelectPrimitive.Trigger>
));
SelectTrigger.displayName = SelectPrimitive.Trigger.displayName;

/**
 * Select component content.
 * Contains the list of options.
 */
const SelectContent = React.forwardRef<
  React.ElementRef<typeof SelectPrimitive.Content>,
  React.ComponentPropsWithoutRef<typeof SelectPrimitive.Content> & {
    /**
     * If true, a search input will be displayed at the top of the content.
     * @default false
     */
    searchable?: boolean;
    /**
     * Placeholder text for the search input.
     * @default "Search..."
     */
    searchPlaceholder?: string;
  }
>(
  (
    { className, children, position = "popper", searchable, searchPlaceholder = "Search...", ...props },
    ref
  ) => {
    const [searchTerm, setSearchTerm] = React.useState("");

    const filterChildren = (nodes: React.ReactNode, term: string) => {
      if (!term) return nodes;
      const lowerCaseTerm = term.toLowerCase();

      return React.Children.map(nodes, (child) => {
        if (React.isValidElement(child)) {
          if (child.type === SelectPrimitive.Group) {
            const filteredGroupChildren = filterChildren(child.props.children, term);
            return React.Children.count(filteredGroupChildren) > 0
              ? React.cloneElement(child, { children: filteredGroupChildren })
              : null;
          }
          if (child.type === SelectPrimitive.Item) {
            const itemValue = child.props.value?.toString().toLowerCase();
            const itemText = child.props.children?.toString().toLowerCase();
            if (itemValue?.includes(lowerCaseTerm) || itemText?.includes(lowerCaseTerm)) {
              return child;
            }
            return null;
          }
        }
        return null;
      });
    };

    const filteredChildren = filterChildren(children, searchTerm);

    return (
      <SelectPrimitive.Portal>
        <SelectPrimitive.Content
          ref={ref}
          className={cn(
            "relative z-50 max-h-96 min-w-[8rem] overflow-hidden rounded-md border bg-popover text-popover-foreground shadow-md data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2",
            position === "popper" &&
              "data-[side=bottom]:translate-y-1 data-[side=left]:-translate-x-1 data-[side=right]:translate-x-1 data-[side=top]:-translate-y-1",
            className
          )}
          position={position}
          {...props}
        >
          {searchable && (
            <div className="p-1">
              <Input
                placeholder={searchPlaceholder}
                value={searchTerm}
                onChange={(e) => setSearchTerm(e.target.value)}
                leftIcon={<Search className="h-4 w-4" />}
                className="h-8"
                aria-label="Search select options"
              />
            </div>
          )}
          <SelectPrimitive.Viewport
            className={cn(
              "p-1",
              position === "popper" &&
                "h-[var(--radix-select-trigger-height)] w-full min-w-[var(--radix-select-trigger-width)]"
            )}
          >
            {filteredChildren}
          </SelectPrimitive.Viewport>
        </SelectPrimitive.Content>
      </SelectPrimitive.Portal>
    );
  }
);
SelectContent.displayName = SelectPrimitive.Content.displayName;

/**
 * Select component label for groups.
 */
const SelectLabel = React.forwardRef<
  React.ElementRef<typeof SelectPrimitive.Label>,
  React.ComponentPropsWithoutRef<typeof SelectPrimitive.Label>
>(({ className, ...props }, ref) => (
  <SelectPrimitive.Label
    ref={ref}
    className={cn("py-1.5 pl-8 pr-2 text-sm font-semibold", className)}
    {...props}
  />
));
SelectLabel.displayName = SelectPrimitive.Label.displayName;

/**
 * Select component item (option).
 */
const SelectItem = React.forwardRef<
  React.ElementRef<typeof SelectPrimitive.Item>,
  React.ComponentPropsWithoutRef<typeof SelectPrimitive.Item> & {
    /**
     * Custom render function for the item content.
     * Receives the item's value and children as arguments.
     */
    render?: (value: string, children: React.ReactNode) => React.ReactNode;
  }
>(({ className, children, render, ...props }, ref) => (
  <SelectPrimitive.Item
    ref={ref}
    className={cn(
      "relative flex w-full cursor-default select-none items-center rounded-sm py-1.5 pl-8 pr-2 text-sm outline-none focus:bg-accent focus:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50",
      className
    )}
    {...props}
  >
    <span className="absolute left-2 flex h-3.5 w-3.5 items-center justify-center">
      <SelectPrimitive.ItemIndicator>
        <Check className="h-4 w-4" />
      </SelectPrimitive.ItemIndicator>
    </span>

    <SelectPrimitive.ItemText>
      {render ? render(props.value, children) : children}
    </SelectPrimitive.ItemText>
  </SelectPrimitive.Item>
));
SelectItem.displayName = SelectPrimitive.Item.displayName;

/**
 * Select component separator.
 */
const SelectSeparator = React.forwardRef<
  React.ElementRef<typeof SelectPrimitive.Separator>,
  React.ComponentPropsWithoutRef<typeof SelectPrimitive.Separator>
>(({ className, ...props }, ref) => (
  <SelectPrimitive.Separator
    ref={ref}
    className={cn("-mx-1 my-1 h-px bg-muted", className)}
    {...props}
  />
));
SelectSeparator.displayName = SelectPrimitive.Separator.displayName;

/**
 * Select component group.
 */
const SelectGroup = SelectPrimitive.Group;

/**
 * Select component value display.
 */
const SelectValue = SelectPrimitive.Value;

export {
  Select,
  SelectGroup,
  SelectValue,
  SelectTrigger,
  SelectContent,
  SelectLabel,
  SelectItem,
  SelectSeparator,
};

---

§ 5. SRC/COMPONENTS/UI/CHECKBOX.TSX

typescript
import * as React from "react";
import * as CheckboxPrimitive from "@radix-ui/react-checkbox";
import { Check } from "lucide-react";

import { cn } from "@/lib/utils/cn";

/**
 * Props for the Checkbox component.
 * Extends standard Radix Checkbox.Root props.
 */
export interface CheckboxProps
  extends React.ComponentPropsWithoutRef<typeof CheckboxPrimitive.Root> {}

/**
 * A checkbox component built with Radix UI and styled with Tailwind CSS.
 * Supports checked, unchecked, and indeterminate states.
 *
 * @component
 * @example
 * <Checkbox id="terms" />
 * <label htmlFor="terms">Accept terms and conditions</label>
 */
const Checkbox = React.forwardRef<
  React.ElementRef<typeof CheckboxPrimitive.Root>,
  CheckboxProps
>(({ className, ...props }, ref) => (
  <CheckboxPrimitive.Root
    ref={ref}
    className={cn(
      "peer h-4 w-4 shrink-0 rounded-sm border border-primary ring-offset-background focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50 data-[state=checked]:bg-primary data-[state=checked]:text-primary-foreground",
      className
    )}
    {...props}
  >
    <CheckboxPrimitive.Indicator
      className={cn("flex items-center justify-center text-current")}
    >
      <Check className="h-4 w-4" />
    </CheckboxPrimitive.Indicator>
  </CheckboxPrimitive.Root>
));
Checkbox.displayName = CheckboxPrimitive.Root.displayName;

export { Checkbox };

---

§ 6. SRC/COMPONENTS/UI/RADIO-GROUP.TSX

typescript
import * as React from "react";
import * as RadioGroupPrimitive from "@radix-ui/react-radio-group";
import { Circle } from "lucide-react";

import { cn } from "@/lib/utils/cn";

/**
 * Props for the RadioGroup component.
 * Extends standard Radix RadioGroup.Root props.
 */
export interface RadioGroupProps
  extends React.ComponentPropsWithoutRef<typeof RadioGroupPrimitive.Root> {}

/**
 * A group of radio buttons built with Radix UI and styled with Tailwind CSS.
 * Ensures only one radio button in the group can be selected at a time.
 *
 * @component
 * @example
 * <RadioGroup defaultValue="option-one">
 *   <div className="flex items-center space-x-2">
 *     <RadioGroupItem value="option-one" id="option-one" />
 *     <label htmlFor="option-one">Option One</label>
 *   </div>
 *   <div className="flex items-center space-x-2">
 *     <RadioGroupItem value="option-two" id="option-two" />
 *     <label htmlFor="option-two">Option Two</label>
 *   </div>
 * </RadioGroup>
 */
const RadioGroup = React.forwardRef<
  React.ElementRef<typeof RadioGroupPrimitive.Root>,
  RadioGroupProps
>(({ className, ...props }, ref) => {
  return (
    <RadioGroupPrimitive.Root
      className={cn("grid gap-2", className)}
      {...props}
      ref={ref}
    />
  );
});
RadioGroup.displayName = RadioGroupPrimitive.Root.displayName;

/**
 * Props for the RadioGroupItem component.
 * Extends standard Radix RadioGroup.Item props.
 */
export interface RadioGroupItemProps
  extends React.ComponentPropsWithoutRef<typeof RadioGroupPrimitive.Item> {}

/**
 * An individual radio button item within a RadioGroup.
 *
 * @component
 * @example
 * // Used within <RadioGroup>
 * <RadioGroupItem value="option-one" id="option-one" />
 */
const RadioGroupItem = React.forwardRef<
  React.ElementRef<typeof RadioGroupPrimitive.Item>,
  RadioGroupItemProps
>(({ className, ...props }, ref) => {
  return (
    <RadioGroupPrimitive.Item
      ref={ref}
      className={cn(
        "aspect-square h-4 w-4 rounded-full border border-primary text-primary ring-offset-background focus:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50",
        className
      )}
      {...props}
    >
      <RadioGroupPrimitive.Indicator className="flex items-center justify-center">
        <Circle className="h-2.5 w-2.5 fill-current text-current" />
      </RadioGroupPrimitive.Indicator>
    </RadioGroupPrimitive.Item>
  );
});
RadioGroupItem.displayName = RadioGroupPrimitive.Item.displayName;

export { RadioGroup, RadioGroupItem };

---

§ 7. SRC/COMPONENTS/UI/SWITCH.TSX

typescript
import * as React from "react";
import * as SwitchPrimitives from "@radix-ui/react-switch";

import { cn } from "@/lib/utils/cn";

/**
 * Props for the Switch component.
 * Extends standard Radix Switch.Root props.
 */
export interface SwitchProps
  extends React.ComponentPropsWithoutRef<typeof SwitchPrimitives.Root> {}

/**
 * A switch component built with Radix UI and styled with Tailwind CSS.
 * Allows users to toggle between two states.
 *
 * @component
 * @example
 * <Switch id="airplane-mode" />
 * <label htmlFor="airplane-mode">Airplane Mode</label>
 */
const Switch = React.forwardRef<
  React.ElementRef<typeof SwitchPrimitives.Root>,
  SwitchProps
>(({ className, ...props }, ref) => (
  <SwitchPrimitives.Root
    className={cn(
      "peer inline-flex h-6 w-11 shrink-0 cursor-pointer items-center rounded-full border-2 border-transparent transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 focus-visible:ring-offset-background disabled:cursor-not-allowed disabled:opacity-50 data-[state=checked]:bg-primary data-[state=unchecked]:bg-input",
      className
    )}
    {...props}
    ref={ref}
  >
    <SwitchPrimitives.Thumb
      className={cn(
        "pointer-events-none block h-5 w-5 rounded-full bg-background shadow-lg ring-0 transition-transform data-[state=checked]:translate-x-5 data-[state=unchecked]:translate-x-0"
      )}
    />
  </SwitchPrimitives.Root>
));
Switch.displayName = SwitchPrimitives.Root.displayName;

export { Switch };

---

§ 8. SRC/COMPONENTS/UI/SLIDER.TSX

typescript
import * as React from "react";
import * as SliderPrimitive from "@radix-ui/react-slider";

import { cn } from "@/lib/utils/cn";

/**
 * Props for the Slider component.
 * Extends standard Radix Slider.Root props.
 */
export interface SliderProps
  extends React.ComponentPropsWithoutRef<typeof SliderPrimitive.Root> {}

/**
 * A slider component built with Radix UI and styled with Tailwind CSS.
 * Allows users to select a value or a range of values.
 *
 * @component
 * @example
 * <Slider defaultValue={[50]} max={100} step={1} />
 * <Slider defaultValue={[25, 75]} max={100} step={1} />
 */
const Slider = React.forwardRef<
  React.ElementRef<typeof SliderPrimitive.Root>,
  SliderProps
>(({ className, ...props }, ref) => (
  <SliderPrimitive.Root
    ref={ref}
    className={cn(
      "relative flex w-full touch-none select-none items-center",
      className
    )}
    {...props}
  >
    <SliderPrimitive.Track className="relative h-2 w-full grow overflow-hidden rounded-full bg-secondary">
      <SliderPrimitive.Range className="absolute h-full bg-primary" />
    </SliderPrimitive.Track>
    {props.value?.map((_, index) => (
      <SliderPrimitive.Thumb
        key={index}
        className="block h-5 w-5 rounded-full border-2 border-primary bg-background ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50"
      />
    ))}
  </SliderPrimitive.Root>
));
Slider.displayName = SliderPrimitive.Root.displayName;

export { Slider };

---

§ 9. SRC/COMPONENTS/UI/LABEL.TSX

typescript
import * as React from "react";
import * as LabelPrimitive from "@radix-ui/react-label";
import { cva, type VariantProps } from "class-variance-authority";

import { cn } from "@/lib/utils/cn";

/**
 * Defines the base styles for the Label component using CVA.
 */
const labelVariants = cva(
  "text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
);

/**
 * Props for the Label component.
 * Extends standard Radix Label.Root props and CVA variant props.
 */
export interface LabelProps
  extends React.ComponentPropsWithoutRef<typeof LabelPrimitive.Root>,
    VariantProps<typeof labelVariants> {}

/**
 * A label component built with Radix UI and styled with Tailwind CSS.
 * Used to associate text with form controls for improved accessibility.
 *
 * @component
 * @example
 * <Label htmlFor="email">Your Email</Label>
 * <Input id="email" type="email" />
 */
const Label = React.forwardRef<
  React.ElementRef<typeof LabelPrimitive.Root>,
  LabelProps
>(({ className, ...props }, ref) => (
  <LabelPrimitive.Root
    ref={ref}
    className={cn(labelVariants(), className)}
    {...props}
  />
));
Label.displayName = LabelPrimitive.Root.displayName;

export { Label };

---

§ 10. SRC/COMPONENTS/UI/FORM.TSX

typescript
import * as React from "react";
import { Slot } from "@radix-ui/react-slot";
import {
  Controller,
  ControllerProps,
  FieldPath,
  FieldValues,
  useFormContext,
} from "react-hook-form";

import { cn } from "@/lib/utils/cn";
import { Label } from "./label"; // Assuming Label component exists

/**
 * Context for FormField, providing access to form state and field information.
 */
interface FormFieldContextValue<
  TFieldValues extends FieldValues = FieldValues,
  TName extends FieldPath<TFieldValues> = FieldPath<TFieldValues>
> {
  name: TName;
}

const FormFieldContext = React.createContext<FormFieldContextValue>(
  {} as FormFieldContextValue
);

/**
 * FormField component for integrating with react-hook-form.
 * Manages the state and validation for a single form input.
 *
 * @component
 * @example
 * <Form {...form}>
 *   <FormField
 *     control={form.control}
 *     name="username"
 *     render={({ field }) => (
 *       <FormItem>
 *         <FormLabel>Username</FormLabel>
 *         <FormControl>
 *           <Input placeholder="shadcn" {...field} />
 *         </FormControl>
 *         <FormDescription>
 *           This is your public display name.
 *         </FormDescription>
 *         <FormMessage />
 *       </FormItem>
 *     )}
 *   />
 * </Form>
 */
const FormField = <
  TFieldValues extends FieldValues = FieldValues,
  TName extends FieldPath<TFieldValues> = FieldPath<TFieldValues>
>(
  props: ControllerProps<TFieldValues, TName>
) => {
  return (
    <FormFieldContext.Provider value={{ name: props.name }}>
      <Controller {...props} />
    </FormFieldContext.Provider>
  );
};

/**
 * Custom hook to access the current form field context.
 * Throws an error if used outside of a FormField.
 */
const useFormField = () => {
  const fieldContext = React.useContext(FormFieldContext);
  const itemContext = React.useContext(FormItemContext);
  const { getFieldState, formState } = useFormContext();

  const fieldState = getFieldState(fieldContext.name, formState);

  if (!fieldContext.name) {
    throw new Error("useFormField should be used within <FormField>");
  }

  const { id } = itemContext;

  return {
    id,
    name: fieldContext.name,
    formItemId: `${id}-form-item`,
    formDescriptionId: `${id}-form-item-description`,
    formMessageId: `${id}-form-item-message`,
    ...fieldState,
  };
};

/**
 * Context for FormItem, providing a unique ID for accessibility.
 */
interface FormItemContextValue {
  id: string;
}

const FormItemContext = React.createContext<FormItemContextValue>(
  {} as FormItemContextValue
);

/**
 * FormItem component, a wrapper for form elements.
 * Provides a unique ID and manages error states.
 *
 * @component
 * @example
 * // Used within <FormField>
 * <FormItem>
 *   <FormLabel>Email</FormLabel>
 *   <FormControl><Input type="email" /></FormControl>
 *   <FormMessage />
 * </FormItem>
 */
const FormItem = React.forwardRef<
  HTMLDivElement,
  React.HTMLAttributes<HTMLDivElement>
>(({ className, ...props }, ref) => {
  const id = React.useId();

  return (
    <FormItemContext.Provider value={{ id }}>
      <div ref={ref} className={cn("space-y-2", className)} {...props} />
    </FormItemContext.Provider>
  );
});
FormItem.displayName = "FormItem";

/**
 * FormLabel component, a wrapper for the Label component.
 * Automatically links to the form control using the `htmlFor` attribute.
 *
 * @component
 * @example
 * // Used within <FormItem>
 * <FormLabel>Your Name</FormLabel>
 */
const FormLabel = React.forwardRef<
  React.ElementRef<typeof Label>,
  React.ComponentPropsWithoutRef<typeof Label>
>(({ className, ...props }, ref) => {
  const { error, formItemId } = useFormField();

  return (
    <Label
      ref={ref}
      className={cn(error && "text-destructive", className)}
      htmlFor={formItemId}
      {...props}
    />
  );
});
FormLabel.displayName = "FormLabel";

/**
 * FormControl component, a wrapper for the actual form input.
 * Renders its child as a slot, allowing for custom input components.
 *
 * @component
 * @example
 * // Used within <FormItem>
 * <FormControl><Input placeholder="shadcn" /></FormControl>
 */
const FormControl = React.forwardRef<
  React.ElementRef<typeof Slot>,
  React.ComponentPropsWithoutRef<typeof Slot>
>(({ ...props }, ref) => {
  const { error, formItemId, formDescriptionId, formMessageId } = useFormField();

  return (
    <Slot
      ref={ref}
      id={formItemId}
      aria-describedby={
        !error
          ? `${formDescriptionId}`
          : `${formDescriptionId} ${formMessageId}`
      }
      aria-invalid={!!error}
      {...props}
    />
  );
});
FormControl.displayName = "FormControl";

/**
 * FormDescription component, provides additional context for a form field.
 *
 * @component
 * @example
 * // Used within <FormItem>
 * <FormDescription>
 *   This is your public display name.
 * </FormDescription>
 */
const FormDescription = React.forwardRef<
  HTMLParagraphElement,
  React.HTMLAttributes<HTMLParagraphElement>
>(({ className, ...props }, ref) => {
  const { formDescriptionId } = useFormField();

  return (
    <p
      ref={ref}
      id={formDescriptionId}
      className={cn("text-[0.8rem] text-muted-foreground", className)}
      {...props}
    />
  );
});
FormDescription.displayName = "FormDescription";

/**
 * FormMessage component, displays validation error messages for a form field.
 *
 * @component
 * @example
 * // Used within <FormItem>
 * <FormMessage />
 */
const FormMessage = React.forwardRef<
  HTMLParagraphElement,
  React.HTMLAttributes<HTMLParagraphElement>
>(({ className, children, ...props }, ref) => {
  const { error, formMessageId } = useFormField();
  const body = error ? String(error?.message) : children;

  if (!body) {
    return null;
  }

  return (
    <p
      ref={ref}
      id={formMessageId}
      className={cn("text-[0.8rem] font-medium text-destructive", className)}
      {...props}
    >
      {body}
    </p>
  );
});
FormMessage.displayName = "FormMessage";

export {
  useFormField,
  Form,
  FormItem,
  FormLabel,
  FormControl,
  FormDescription,
  FormMessage,
  FormField,
};

---

§ 11. SRC/COMPONENTS/UI/CARD.TSX

typescript
import * as React from "react";
import { cn } from "@/lib/utils/cn";
import { cva, type VariantProps } from "class-variance-authority";

/**
 * Defines the variants for the Card component using CVA.
 */
const cardVariants = cva(
  "rounded-lg border bg-card text-card-foreground shadow-sm",
  {
    variants: {
      variant: {
        default: "border-border",
        outlined: "border-2 border-input",
        elevated: "shadow-md hover:shadow-lg transition-shadow",
      },
    },
    defaultVariants: {
      variant: "default",
    },
  }
);

/**
 * Props for the Card component.
 * Extends standard HTML div attributes and CVA variant props.
 */
export interface CardProps
  extends React.HTMLAttributes<HTMLDivElement>,
    VariantProps<typeof cardVariants> {}

/**
 * A flexible container component for grouping related content.
 * Supports different visual variants like default, outlined, and elevated.
 *
 * @component
 * @example
 * <Card>
 *   <CardHeader>
 *     <CardTitle>Card Title</CardTitle>
 *     <CardDescription>Card Description</CardDescription>
 *   </CardHeader>
 *   <CardContent>
 *     <p>Card content goes here.</p>
 *   </CardContent>
 *   <CardFooter>
 *     <Button>Action</Button>
 *   </CardFooter>
 * </Card>
 */
const Card = React.forwardRef<HTMLDivElement, CardProps>(
  ({ className, variant, ...props }, ref) => (
    <div
      ref={ref}
      className={cn(cardVariants({ variant, className }))}
      {...props}
    />
  )
);
Card.displayName = "Card";

/**
 * Props for the CardHeader component.
 * Extends standard HTML div attributes.
 */
export interface CardHeaderProps extends React.HTMLAttributes<HTMLDivElement> {}

/**
 * Header section of a Card.
 *
 * @component
 * @example
 * // Used within <Card>
 * <CardHeader>
 *   <CardTitle>Title</CardTitle>
 * </CardHeader>
 */
const CardHeader = React.forwardRef<HTMLDivElement, CardHeaderProps>(
  ({ className, ...props }, ref) => (
    <div
      ref={ref}
      className={cn("flex flex-col space-y-1.5 p-6", className)}
      {...props}
    />
  )
);
CardHeader.displayName = "CardHeader";

/**
 * Props for the CardTitle component.
 * Extends standard HTML h3 attributes.
 */
export interface CardTitleProps
  extends React.HTMLAttributes<HTMLHeadingElement> {}

/**
 * Title of a Card.
 *
 * @component
 * @example
 * // Used within <CardHeader>
 * <CardTitle>My Card</CardTitle>
 */
const CardTitle = React.forwardRef<HTMLParagraphElement, CardTitleProps>(
  ({ className, ...props }, ref) => (
    <h3
      ref={ref}
      className={cn(
        "text-2xl font-semibold leading-none tracking-tight",
        className
      )}
      {...props}
    />
  )
);
CardTitle.displayName = "CardTitle";

/**
 * Props for the CardDescription component.
 * Extends standard HTML p attributes.
 */
export interface CardDescriptionProps
  extends React.HTMLAttributes<HTMLParagraphElement> {}

/**
 * Description or subtitle of a Card.
 *
 * @component
 * @example
 * // Used within <CardHeader>
 * <CardDescription>A brief description.</CardDescription>
 */
const CardDescription = React.forwardRef<
  HTMLParagraphElement,
  CardDescriptionProps
>(({ className, ...props }, ref) => (
  <p
    ref={ref}
    className={cn("text-sm text-muted-foreground", className)}
    {...props}
  />
);
CardDescription.displayName = "CardDescription";

/**
 * Props for the CardContent component.
 * Extends standard HTML div attributes.
 */
export interface CardContentProps extends React.HTMLAttributes<HTMLDivElement> {}

/**
 * Main content area of a Card.
 *
 * @component
 * @example
 * // Used within <Card>
 * <CardContent>
 *   <p>Content goes here.</p>
 * </CardContent>
 */
const CardContent = React.forwardRef<HTMLDivElement, CardContentProps>(
  ({ className, ...props }, ref) => (
    <div ref={ref} className={cn("p-6 pt-0", className)} {...props} />
  )
);
CardContent.displayName = "CardContent";

/**
 * Props for the CardFooter component.
 * Extends standard HTML div attributes.
 */
export interface CardFooterProps extends React.HTMLAttributes<HTMLDivElement> {}

/**
 * Footer section of a Card, often used for actions.
 *
 * @component
 * @example
 * // Used within <Card>
 * <CardFooter>
 *   <Button>Save</Button>
 * </CardFooter>
 */
const CardFooter = React.forwardRef<HTMLDivElement, CardFooterProps>(
  ({ className, ...props }, ref) => (
    <div
      ref={ref}
      className={cn("flex items-center p-6 pt-0", className)}
      {...props}
    />
  )
);
CardFooter.displayName = "CardFooter";

export { Card, CardHeader, CardFooter, CardTitle, CardDescription, CardContent };

---

§ 12. SRC/COMPONENTS/UI/BADGE.TSX

typescript
import * as React from "react";
import { cva, type VariantProps } from "class-variance-authority";

import { cn } from "@/lib/utils/cn";

/**
 * Defines the variants and sizes for the Badge component using CVA.
 */
const badgeVariants = cva(
  "inline-flex items-center rounded-full border px-2.5 py-0.5 text-xs font-semibold transition-colors focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2",
  {
    variants: {
      variant: {
        default:
          "border-transparent bg-primary text-primary-foreground hover:bg-primary/80",
        secondary:
          "border-transparent bg-secondary text-secondary-foreground hover:bg-secondary/80",
        destructive:
          "border-transparent bg-destructive text-destructive-foreground hover:bg-destructive/80",
        outline: "text-foreground",
      },
      size: {
        default: "h-5",
        sm: "h-4 px-2 py-0.5 text-[0.6rem]",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);

/**
 * Props for the Badge component.
 * Extends standard HTML div attributes and CVA variant props.
 */
export interface BadgeProps
  extends React.HTMLAttributes<HTMLDivElement>,
    VariantProps<typeof badgeVariants> {}

/**
 * A small, informative component used to highlight status, categories, or counts.
 * Supports various visual variants and sizes.
 *
 * @component
 * @example
 * <Badge>New</Badge>
 * <Badge variant="secondary">Beta</Badge>
 * <Badge variant="destructive" size="sm">Error</Badge>
 */
function Badge({ className, variant, size, ...props }: BadgeProps) {
  return (
    <div className={cn(badgeVariants({ variant, size }), className)} {...props} />
  );
}

export { Badge, badgeVariants };

---

§ 13. SRC/COMPONENTS/UI/AVATAR.TSX

typescript
import * as React from "react";
import * as AvatarPrimitive from "@radix-ui/react-avatar";
import { cva, type VariantProps } from "class-variance-authority";

import { cn } from "@/lib/utils/cn";

/**
 * Defines the sizes for the Avatar component using CVA.
 */
const avatarSizes = cva("relative flex h-10 w-10 shrink-0 overflow-hidden rounded-full", {
  variants: {
    size: {
      xs: "h-6 w-6 text-xs",
      sm: "h-8 w-8 text-sm",
      default: "h-10 w-10 text-base",
      lg: "h-12 w-12 text-lg",
      xl: "h-16 w-16 text-xl",
    },
  },
  defaultVariants: {
    size: "default",
  },
});

/**
 * Props for the Avatar component.
 * Extends standard Radix Avatar.Root props and CVA size props.
 */
export interface AvatarProps
  extends React.ComponentPropsWithoutRef<typeof AvatarPrimitive.Root>,
    VariantProps<typeof avatarSizes> {}

/**
 * A component for displaying user avatars, with support for images and fallbacks.
 * Supports various sizes.
 *
 * @component
 * @example
 * <Avatar>
 *   <AvatarImage src="https://github.com/shadcn.png" alt="@shadcn" />
 *   <AvatarFallback>SC</AvatarFallback>
 * </Avatar>
 * <Avatar size="lg">
 *   <AvatarFallback>JD</AvatarFallback>
 * </Avatar>
 */
const Avatar = React.forwardRef<
  React.ElementRef<typeof AvatarPrimitive.Root>,
  AvatarProps
>(({ className, size, ...props }, ref) => (
  <AvatarPrimitive.Root
    ref={ref}
    className={cn(avatarSizes({ size, className }))}
    {...props}
  />
));
Avatar.displayName = AvatarPrimitive.Root.displayName;

/**
 * Props for the AvatarImage component.
 * Extends standard Radix Avatar.Image props.
 */
export interface AvatarImageProps
  extends React.ComponentPropsWithoutRef<typeof AvatarPrimitive.Image> {}

/**
 * Displays the image within an Avatar.
 *
 * @component
 * @example
 * // Used within <Avatar>
 * <AvatarImage src="https://github.com/shadcn.png" alt="@shadcn" />
 */
const AvatarImage = React.forwardRef<
  React.ElementRef<typeof AvatarPrimitive.Image>,
  AvatarImageProps
>(({ className, ...props }, ref) => (
  <AvatarPrimitive.Image
    ref={ref}
    className={cn("aspect-square h-full w-full", className)}
    {...props}
  />
));
AvatarImage.displayName = AvatarPrimitive.Image.displayName;

/**
 * Props for the AvatarFallback component.
 * Extends standard Radix Avatar.Fallback props.
 */
export interface AvatarFallbackProps
  extends React.ComponentPropsWithoutRef<typeof AvatarPrimitive.Fallback> {}

/**
 * Displays a fallback (e.g., initials) when the avatar image fails to load.
 *
 * @component
 * @example
 * // Used within <Avatar>
 * <AvatarFallback>SC</AvatarFallback>
 */
const AvatarFallback = React.forwardRef<
  React.ElementRef<typeof AvatarPrimitive.Fallback>,
  AvatarFallbackProps
>(({ className, ...props }, ref) => (
  <AvatarPrimitive.Fallback
    ref={ref}
    className={cn(
      "flex h-full w-full items-center justify-center rounded-full bg-muted",
      className
    )}
    {...props}
  />
));
AvatarFallback.displayName = AvatarPrimitive.Fallback.displayName;

export { Avatar, AvatarImage, AvatarFallback };

---

§ 14. SRC/COMPONENTS/UI/ALERT.TSX

typescript
import * as React from "react";
import { cva, type VariantProps } from "class-variance-authority";
import { X } from "lucide-react"; // Assuming lucide-react for icons

import { cn } from "@/lib/utils/cn";
import { Button } from "./button"; // Assuming Button component exists

/**
 * Defines the variants for the Alert component using CVA.
 */
const alertVariants = cva(
  "relative w-full rounded-lg border p-4 [&>svg~*]:pl-7 [&>svg+div]:translate-y-[-3px] [&>svg]:absolute [&>svg]:left-4 [&>svg]:top-4 [&>svg]:text-foreground",
  {
    variants: {
      variant: {
        default: "bg-background text-foreground",
        destructive:
          "border-destructive/50 text-destructive dark:border-destructive [&>svg]:text-destructive",
        warning:
          "border-orange-500/50 text-orange-700 dark:border-orange-400 dark:text-orange-300 [&>svg]:text-orange-500",
        success:
          "border-green-500/50 text-green-700 dark:border-green-400 dark:text-green-300 [&>svg]:text-green-500",
      },
    },
    defaultVariants: {
      variant: "default",
    },
  }
);

/**
 * Props for the Alert component.
 * Extends standard HTML div attributes and CVA variant props.
 */
export interface AlertProps
  extends React.HTMLAttributes<HTMLDivElement>,
    VariantProps<typeof alertVariants> {
  /**
   * If true, a dismiss button will be displayed to close the alert.
   * @default false
   */
  dismissible?: boolean;
  /**
   * Callback function called when the dismiss button is clicked.
   */
  onDismiss?: () => void;
}

/**
 * A component to display important messages to users.
 * Supports different visual variants (default, destructive, warning, success) and an optional dismiss button.
 *
 * @component
 * @example
 * <Alert>
 *   <AlertTitle>Heads up!</AlertTitle>
 *   <AlertDescription>You can add components to your app using the cli.</AlertDescription>
 * </Alert>
 * <Alert variant="destructive" dismissible onDismiss={() => console.log("Dismissed")}>
 *   <AlertTitle>Error</AlertTitle>
 *   <AlertDescription>Something went wrong.</AlertDescription>
 * </Alert>
 */
const Alert = React.forwardRef<HTMLDivElement, AlertProps>(
  ({ className, variant, dismissible, onDismiss, children, ...props }, ref) => {
    const [isVisible, setIsVisible] = React.useState(true);

    const handleDismiss = () => {
      setIsVisible(false);
      onDismiss?.();
    };

    if (!isVisible) return null;

    return (
      <div
        ref={ref}
        role="alert"
        className={cn(alertVariants({ variant }), className)}
        {...props}
      >
        {children}
        {dismissible && (
          <Button
            variant="ghost"
            size="icon"
            onClick={handleDismiss}
            className="absolute right-2 top-2 h-6 w-6 text-muted-foreground hover:bg-transparent hover:text-foreground"
            aria-label="Dismiss alert"
          >
            <X className="h-4 w-4" />
          </Button>
        )}
      </div>
    );
  }
);
Alert.displayName = "Alert";

/**
 * Props for the AlertTitle component.
 * Extends standard HTML h5 attributes.
 */
export interface AlertTitleProps
  extends React.HTMLAttributes<HTMLHeadingElement> {}

/**
 * Title of an Alert.
 *
 * @component
 * @example
 * // Used within <Alert>
 * <AlertTitle>Success!</AlertTitle>
 */
const AlertTitle = React.forwardRef<HTMLParagraphElement, AlertTitleProps>(
  ({ className, ...props }, ref) => (
    <h5
      ref={ref}
      className={cn("mb-1 font-medium leading-none tracking-tight", className)}
      {...props}
    />
  )
);
AlertTitle.displayName = "AlertTitle";

/**
 * Props for the AlertDescription component.
 * Extends standard HTML div attributes.
 */
export interface AlertDescriptionProps
  extends React.HTMLAttributes<HTMLDivElement> {}

/**
 * Description or main content of an Alert.
 *
 * @component
 * @example
 * // Used within <Alert>
 * <AlertDescription>Your action was successful.</AlertDescription>
 */
const AlertDescription = React.forwardRef<
  HTMLDivElement,
  AlertDescriptionProps
>(({ className, ...props }, ref) => (
  <div
    ref={ref}
    className={cn("text-sm [&_p]:leading-relaxed", className)}
    {...props}
  />
);
AlertDescription.displayName = "AlertDescription";

export { Alert, AlertTitle, AlertDescription };

---

§ 15. SRC/COMPONENTS/UI/DIALOG.TSX

typescript
import * as React from "react";
import * as DialogPrimitive from "@radix-ui/react-dialog";
import { X } from "lucide-react";
import { cva, type VariantProps } from "class-variance-authority";

import { cn } from "@/lib/utils/cn";

/**
 * Dialog component root.
 * @see https://www.radix-ui.com/primitives/docs/components/dialog
 */
const Dialog = DialogPrimitive.Root;

/**
 * Dialog component trigger.
 */
const DialogTrigger = DialogPrimitive.Trigger;

/**
 * Dialog component portal.
 */
const DialogPortal = DialogPrimitive.Portal;

/**
 * Dialog component close button.
 */
const DialogClose = DialogPrimitive.Close;

/**
 * Dialog component overlay.
 */
const DialogOverlay = React.forwardRef<
  React.ElementRef<typeof DialogPrimitive.Overlay>,
  React.ComponentPropsWithoutRef<typeof DialogPrimitive.Overlay>
>(({ className, ...props }, ref) => (
  <DialogPrimitive.Overlay
    ref={ref}
    className={cn(
      "fixed inset-0 z-50 bg-background/80 backdrop-blur-sm data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0",
      className
    )}
    {...props}
  />
));
DialogOverlay.displayName = DialogPrimitive.Overlay.displayName;

/**
 * Defines the sizes for the DialogContent component using CVA.
 */
const dialogContentVariants = cva(
  "fixed left-[50%] top-[50%] z-50 grid w-full translate-x-[-50%] translate-y-[-50%] gap-4 border bg-background p-6 shadow-lg duration-200 data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[state=closed]:slide-out-to-left-1/2 data-[state=closed]:slide-out-to-top-[48%] data-[state=open]:slide-in-from-left-1/2 data-[state=open]:slide-in-from-top-[48%] sm:rounded-lg",
  {
    variants: {
      size: {
        sm: "max-w-sm",
        default: "max-w-lg",
        lg: "max-w-2xl",
        xl: "max-w-4xl",
        full: "max-w-full h-full rounded-none",
      },
    },
    defaultVariants: {
      size: "default",
    },
  }
);

/**
 * Props for the DialogContent component.
 * Extends standard Radix Dialog.Content props and CVA size props.
 */
export interface DialogContentProps
  extends React.ComponentPropsWithoutRef<typeof DialogPrimitive.Content>,
    VariantProps<typeof dialogContentVariants> {}

/**
 * Content area of a Dialog.
 * Supports various sizes and includes a close button.
 *
 * @component
 * @example
 * <Dialog>
 *   <DialogTrigger asChild><Button>Open</Button></DialogTrigger>
 *   <DialogContent size="lg">
 *     <DialogHeader>
 *       <DialogTitle>Are you absolutely sure?</DialogTitle>
 *       <DialogDescription>
 *         This action cannot be undone.
 *       </DialogDescription>
 *     </DialogHeader>
 *     <DialogFooter>
 *       <Button type="submit">Save changes</Button>
 *     </DialogFooter>
 *   </DialogContent>
 * </Dialog>
 */
const DialogContent = React.forwardRef<
  React.ElementRef<typeof DialogPrimitive.Content>,
  DialogContentProps
>(({ className, children, size, ...props }, ref) => (
  <DialogPortal>
    <DialogOverlay />
    <DialogPrimitive.Content
      ref={ref}
      className={cn(dialogContentVariants({ size, className }))}
      {...props}
    >
      {children}
      <DialogPrimitive.Close className="absolute right-4 top-4 rounded-sm opacity-70 ring-offset-background transition-opacity hover:opacity-100 focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2 disabled:pointer-events-none data-[state=open]:bg-accent data-[state=open]:text-muted-foreground">
        <X className="h-4 w-4" />
        <span className="sr-only">Close</span>
      </DialogPrimitive.Close>
    </DialogPrimitive.Content>
  </DialogPortal>
));
DialogContent.displayName = DialogPrimitive.Content.displayName;

/**
 * Props for the DialogHeader component.
 * Extends standard HTML div attributes.
 */
export interface DialogHeaderProps extends React.HTMLAttributes<HTMLDivElement> {}

/**
 * Header section of a Dialog, typically containing title and description.
 *
 * @component
 * @example
 * // Used within <DialogContent>
 * <DialogHeader>
 *   <DialogTitle>Edit Profile</DialogTitle>
 *   <DialogDescription>Make changes to your profile here.</DialogDescription>
 * </DialogHeader>
 */
const DialogHeader = ({
  className,
  ...props
}: DialogHeaderProps) => (
  <div
    className={cn(
      "flex flex-col space-y-1.5 text-center sm:text-left",
      className
    )}
    {...props}
  />
);
DialogHeader.displayName = "DialogHeader";

/**
 * Props for the DialogFooter component.
 * Extends standard HTML div attributes.
 */
export interface DialogFooterProps extends React.HTMLAttributes<HTMLDivElement> {}

/**
 * Footer section of a Dialog, typically containing action buttons.
 *
 * @component
 * @example
 * // Used within <DialogContent>
 * <DialogFooter>
 *   <Button variant="outline">Cancel</Button>
 *   <Button>Save</Button>
 * </DialogFooter>
 */
const DialogFooter = ({
  className,
  ...props
}: DialogFooterProps) => (
  <div
    className={cn(
      "flex flex-col-reverse sm:flex-row sm:justify-end sm:space-x-2",
      className
    )}
    {...props}
  />
);
DialogFooter.displayName = "DialogFooter";

/**
 * Props for the DialogTitle component.
 * Extends standard Radix Dialog.Title props.
 */
export interface DialogTitleProps
  extends React.ComponentPropsWithoutRef<typeof DialogPrimitive.Title> {}

/**
 * Title of a Dialog.
 *
 * @component
 * @example
 * // Used within <DialogHeader>
 * <DialogTitle>Confirm Action</DialogTitle>
 */
const DialogTitle = React.forwardRef<
  React.ElementRef<typeof DialogPrimitive.Title>,
  DialogTitleProps
>(({ className, ...props }, ref) => (
  <DialogPrimitive.Title
    ref={ref}
    className={cn(
      "text-lg font-semibold leading-none tracking-tight",
      className
    )}
    {...props}
  />
));
DialogTitle.displayName = DialogPrimitive.Title.displayName;

/**
 * Props for the DialogDescription component.
 * Extends standard Radix Dialog.Description props.
 */
export interface DialogDescriptionProps
  extends React.ComponentPropsWithoutRef<typeof DialogPrimitive.Description> {}

/**
 * Description or subtitle of a Dialog.
 *
 * @component
 * @example
 * // Used within <DialogHeader>
 * <DialogDescription>
 *   This action cannot be undone.
 * </DialogDescription>
 */
const DialogDescription = React.forwardRef<
  React.ElementRef<typeof DialogPrimitive.Description>,
  DialogDescriptionProps
>(({ className, ...props }, ref) => (
  <DialogPrimitive.Description
    ref={ref}
    className={cn("text-sm text-muted-foreground", className)}
    {...props}
  />
));
DialogDescription.displayName = DialogPrimitive.Description.displayName;

export {
  Dialog,
  DialogPortal,
  DialogOverlay,
  DialogTrigger,
  DialogClose,
  DialogContent,
  DialogHeader,
  DialogFooter,
  DialogTitle,
  DialogDescription,
};

---

§ 16. SRC/COMPONENTS/UI/SHEET.TSX

typescript
import * as React from "react";
import * as SheetPrimitive from "@radix-ui/react-dialog";
import { cva, type VariantProps } from "class-variance-authority";
import { X } from "lucide-react";

import { cn } from "@/lib/utils/cn";

/**
 * Sheet component root.
 * @see https://www.radix-ui.com/primitives/docs/components/dialog
 */
const Sheet = SheetPrimitive.Root;

/**
 * Sheet component trigger.
 */
const SheetTrigger = SheetPrimitive.Trigger;

/**
 * Sheet component close button.
 */
const SheetClose = SheetPrimitive.Close;

/**
 * Sheet component portal.
 */
const SheetPortal = SheetPrimitive.Portal;

/**
 * Sheet component overlay.
 */
const SheetOverlay = React.forwardRef<
  React.ElementRef<typeof SheetPrimitive.Overlay>,
  React.ComponentPropsWithoutRef<typeof SheetPrimitive.Overlay>
>(({ className, ...props }, ref) => (
  <SheetPrimitive.Overlay
    className={cn(
      "fixed inset-0 z-50 bg-background/80 backdrop-blur-sm data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0",
      className
    )}
    {...props}
    ref={ref}
  />
));
SheetOverlay.displayName = SheetPrimitive.Overlay.displayName;

/**
 * Defines the variants for the SheetContent component's side.
 */
const sheetVariants = cva(
  "fixed z-50 gap-4 bg-background p-6 shadow-lg transition ease-in-out data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:duration-300 data-[state=open]:duration-500",
  {
    variants: {
      side: {
        top: "inset-x-0 top-0 border-b data-[state=closed]:slide-out-to-top data-[state=open]:slide-in-from-top",
        bottom:
          "inset-x-0 bottom-0 border-t data-[state=closed]:slide-out-to-bottom data-[state=open]:slide-in-from-bottom",
        left: "inset-y-0 left-0 h-full w-3/4 border-r data-[state=closed]:slide-out-to-left data-[state=open]:slide-in-from-left sm:max-w-sm",
        right:
          "inset-y-0 right-0 h-full w-3/4 border-l data-[state=closed]:slide-out-to-right data-[state=open]:slide-in-from-right sm:max-w-sm",
      },
    },
    defaultVariants: {
      side: "right",
    },
  }
);

/**
 * Props for the SheetContent component.
 * Extends standard Radix Sheet.Content props and CVA side props.
 */
export interface SheetContentProps
  extends React.ComponentPropsWithoutRef<typeof SheetPrimitive.Content>,
    VariantProps<typeof sheetVariants> {}

/**
 * Content area of a Sheet (side drawer).
 * Supports different sides (top, bottom, left, right) and includes a close button.
 *
 * @component
 * @example
 * <Sheet>
 *   <SheetTrigger asChild><Button>Open Left</Button></SheetTrigger>
 *   <SheetContent side="left">
 *     <SheetHeader>
 *       <SheetTitle>Edit Profile</SheetTitle>
 *       <SheetDescription>Make changes to your profile here.</SheetDescription>
 *     </SheetHeader>
 *     <div className="py-4">Content goes here.</div>
 *   </SheetContent>
 * </Sheet>
 */
const SheetContent = React.forwardRef<
  React.ElementRef<typeof SheetPrimitive.Content>,
  SheetContentProps
>(({ side = "right", className, children, ...props }, ref) => (
  <SheetPortal>
    <SheetOverlay />
    <SheetPrimitive.Content
      ref={ref}
      className={cn(sheetVariants({ side }), className)}
      {...props}
    >
      {children}
      <SheetPrimitive.Close className="absolute right-4 top-4 rounded-sm opacity-70 ring-offset-background transition-opacity hover:opacity-100 focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2 disabled:pointer-events-none data-[state=open]:bg-secondary">
        <X className="h-4 w-4" />
        <span className="sr-only">Close</span>
      </SheetPrimitive.Close>
    </SheetPrimitive.Content>
  </SheetPortal>
));
SheetContent.displayName = SheetPrimitive.Content.displayName;

/**
 * Props for the SheetHeader component.
 * Extends standard HTML div attributes.
 */
export interface SheetHeaderProps extends React.HTMLAttributes<HTMLDivElement> {}

/**
 * Header section of a Sheet, typically containing title and description.
 *
 * @component
 * @example
 * // Used within <SheetContent>
 * <SheetHeader>
 *   <SheetTitle>Notifications</SheetTitle>
 *   <SheetDescription>Your recent activity.</SheetDescription>
 * </SheetHeader>
 */
const SheetHeader = ({
  className,
  ...props
}: SheetHeaderProps) => (
  <div
    className={cn(
      "flex flex-col space-y-2 text-center sm:text-left",
      className
    )}
    {...props}
  />
);
SheetHeader.displayName = "SheetHeader";

/**
 * Props for the SheetFooter component.
 * Extends standard HTML div attributes.
 */
export interface SheetFooterProps extends React.HTMLAttributes<HTMLDivElement> {}

/**
 * Footer section of a Sheet, typically containing action buttons.
 *
 * @component
 * @example
 * // Used within <SheetContent>
 * <SheetFooter>
 *   <Button variant="outline">Cancel</Button>
 *   <Button>Save</Button>
 * </SheetFooter>
 */
const SheetFooter = ({
  className,
  ...props
}: SheetFooterProps) => (
  <div
    className={cn(
      "flex flex-col-reverse sm:flex-row sm:justify-end sm:space-x-2",
      className
    )}
    {...props}
  />
);
SheetFooter.displayName = "SheetFooter";

/**
 * Props for the SheetTitle component.
 * Extends standard Radix Sheet.Title props.
 */
export interface SheetTitleProps
  extends React.ComponentPropsWithoutRef<typeof SheetPrimitive.Title> {}

/**
 * Title of a Sheet.
 *
 * @component
 * @example
 * // Used within <SheetHeader>
 * <SheetTitle>Settings</SheetTitle>
 */
const SheetTitle = React.forwardRef<
  React.ElementRef<typeof SheetPrimitive.Title>,
  SheetTitleProps
>(({ className, ...props }, ref) => (
  <SheetPrimitive.Title
    ref={ref}
    className={cn("text-lg font-semibold text-foreground", className)}
    {...props}
  />
));
SheetTitle.displayName = SheetPrimitive.Title.displayName;

/**
 * Props for the SheetDescription component.
 * Extends standard Radix Sheet.Description props.
 */
export interface SheetDescriptionProps
  extends React.ComponentPropsWithoutRef<typeof SheetPrimitive.Description> {}

/**
 * Description or subtitle of a Sheet.
 *
 * @component
 * @example
 * // Used within <SheetHeader>
 * <SheetDescription>
 *   Make changes to your account here.
 * </SheetDescription>
 */
const SheetDescription = React.forwardRef<
  React.ElementRef<typeof SheetPrimitive.Description>,
  SheetDescriptionProps
>(({ className, ...props }, ref) => (
  <SheetPrimitive.Description
    ref={ref}
    className={cn("text-sm text-muted-foreground", className)}
    {...props}
  />
));
SheetDescription.displayName = SheetPrimitive.Description.displayName;

export {
  Sheet,
  SheetPortal,
  SheetOverlay,
  SheetTrigger,
  SheetClose,
  SheetContent,
  SheetHeader,
  SheetFooter,
  SheetTitle,
  SheetDescription,
};

---

§ 17. SRC/COMPONENTS/UI/DROPDOWN-MENU.TSX

typescript
import * as React from "react";
import * as DropdownMenuPrimitive from "@radix-ui/react-dropdown-menu";
import { Check, ChevronRight, Circle } from "lucide-react";

import { cn } from "@/lib/utils/cn";

/**
 * DropdownMenu component root.
 * @see https://www.radix-ui.com/primitives/docs/components/dropdown-menu
 */
const DropdownMenu = DropdownMenuPrimitive.Root;

/**
 * DropdownMenu component trigger.
 */
const DropdownMenuTrigger = DropdownMenuPrimitive.Trigger;

/**
 * DropdownMenu component group.
 */
const DropdownMenuGroup = DropdownMenuPrimitive.Group;

/**
 * DropdownMenu component portal.
 */
const DropdownMenuPortal = DropdownMenuPrimitive.Portal;

/**
 * DropdownMenu component sub-menu.
 */
const DropdownMenuSub = DropdownMenuPrimitive.Sub;

/**
 * DropdownMenu component sub-menu trigger.
 */
const DropdownMenuSubTrigger = React.forwardRef<
  React.ElementRef<typeof DropdownMenuPrimitive.SubTrigger>,
  React.ComponentPropsWithoutRef<typeof DropdownMenuPrimitive.SubTrigger> & {
    inset?: boolean;
  }
>(({ className, inset, children, ...props }, ref) => (
  <DropdownMenuPrimitive.SubTrigger
    ref={ref}
    className={cn(
      "flex cursor-default select-none items-center rounded-sm px-2 py-1.5 text-sm outline-none focus:bg-accent data-[state=open]:bg-accent",
      inset && "pl-8",
      className
    )}
    {...props}
  >
    {children}
    <ChevronRight className="ml-auto h-4 w-4" />
  </DropdownMenuPrimitive.SubTrigger>
));
DropdownMenuSubTrigger.displayName =
  DropdownMenuPrimitive.SubTrigger.displayName;

/**
 * DropdownMenu component sub-menu content.
 */
const DropdownMenuSubContent = React.forwardRef<
  React.ElementRef<typeof DropdownMenuPrimitive.SubContent>,
  React.ComponentPropsWithoutRef<typeof DropdownMenuPrimitive.SubContent>
>(({ className, ...props }, ref) => (
  <DropdownMenuPrimitive.SubContent
    ref={ref}
    className={cn(
      "z-50 min-w-[8rem] overflow-hidden rounded-md border bg-popover p-1 text-popover-foreground shadow-lg data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2",
      className
    )}
    {...props}
  />
));
DropdownMenuSubContent.displayName =
  DropdownMenuPrimitive.SubContent.displayName;

/**
 * DropdownMenu component content.
 */
const DropdownMenuContent = React.forwardRef<
  React.ElementRef<typeof DropdownMenuPrimitive.Content>,
  React.ComponentPropsWithoutRef<typeof DropdownMenuPrimitive.Content>
>(({ className, sideOffset = 4, ...props }, ref) => (
  <DropdownMenuPrimitive.Portal>
    <DropdownMenuPrimitive.Content
      ref={ref}
      sideOffset={sideOffset}
      className={cn(
        "z-50 min-w-[8rem] overflow-hidden rounded-md border bg-popover p-1 text-popover-foreground shadow-md data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2",
        className
      )}
      {...props}
    />
  </DropdownMenuPrimitive.Portal>
));
DropdownMenuContent.displayName = DropdownMenuPrimitive.Content.displayName;

/**
 * DropdownMenu component item.
 */
const DropdownMenuItem = React.forwardRef<
  React.ElementRef<typeof DropdownMenuPrimitive.Item>,
  React.ComponentPropsWithoutRef<typeof DropdownMenuPrimitive.Item> & {
    inset?: boolean;
  }
>(({ className, inset, ...props }, ref) => (
  <DropdownMenuPrimitive.Item
    ref={ref}
    className={cn(
      "relative flex cursor-default select-none items-center rounded-sm px-2 py-1.5 text-sm outline-none transition-colors focus:bg-accent focus:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50",
      inset && "pl-8",
      className
    )}
    {...props}
  />
));
DropdownMenuItem.displayName = DropdownMenuPrimitive.Item.displayName;

/**
 * DropdownMenu component checkbox item.
 */
const DropdownMenuCheckboxItem = React.forwardRef<
  React.ElementRef<typeof DropdownMenuPrimitive.CheckboxItem>,
  React.ComponentPropsWithoutRef<typeof DropdownMenuPrimitive.CheckboxItem>
>(({ className, children, checked, ...props }, ref) => (
  <DropdownMenuPrimitive.CheckboxItem
    ref={ref}
    className={cn(
      "relative flex cursor-default select-none items-center rounded-sm py-1.5 pl-8 pr-2 text-sm outline-none transition-colors focus:bg-accent focus:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50",
      className
    )}
    checked={checked}
    {...props}
  >
    <span className="absolute left-2 flex h-3.5 w-3.5 items-center justify-center">
      <DropdownMenuPrimitive.ItemIndicator>
        <Check className="h-4 w-4" />
      </DropdownMenuPrimitive.ItemIndicator>
    </span>
    {children}
  </DropdownMenuPrimitive.CheckboxItem>
));
DropdownMenuCheckboxItem.displayName =
  DropdownMenuPrimitive.CheckboxItem.displayName;

/**
 * DropdownMenu component radio item.
 */
const DropdownMenuRadioItem = React.forwardRef<
  React.ElementRef<typeof DropdownMenuPrimitive.RadioItem>,
  React.ComponentPropsWithoutRef<typeof DropdownMenuPrimitive.RadioItem>
>(({ className, children, ...props }, ref) => (
  <DropdownMenuPrimitive.RadioItem
    ref={ref}
    className={cn(
      "relative flex cursor-default select-none items-center rounded-sm py-1.5 pl-8 pr-2 text-sm outline-none transition-colors focus:bg-accent focus:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50",
      className
    )}
    {...props}
  >
    <span className="absolute left-2 flex h-3.5 w-3.5 items-center justify-center">
      <DropdownMenuPrimitive.ItemIndicator>
        <Circle className="h-2 w-2 fill-current" />
      </DropdownMenuPrimitive.ItemIndicator>
    </span>
    {children}
  </DropdownMenuPrimitive.RadioItem>
));
DropdownMenuRadioItem.displayName = DropdownMenuPrimitive.RadioItem.displayName;

/**
 * DropdownMenu component label.
 */
const DropdownMenuLabel = React.forwardRef<
  React.ElementRef<typeof DropdownMenuPrimitive.Label>,
  React.ComponentPropsWithoutRef<typeof DropdownMenuPrimitive.Label> & {
    inset?: boolean;
  }
>(({ className, inset, ...props }, ref) => (
  <DropdownMenuPrimitive.Label
    ref={ref}
    className={cn(
      "px-2 py-1.5 text-sm font-semibold",
      inset && "pl-8",
      className
    )}
    {...props}
  />
));
DropdownMenuLabel.displayName = DropdownMenuPrimitive.Label.displayName;

/**
 * DropdownMenu component separator.
 */
const DropdownMenuSeparator = React.forwardRef<
  React.ElementRef<typeof DropdownMenuPrimitive.Separator>,
  React.ComponentPropsWithoutRef<typeof DropdownMenuPrimitive.Separator>
>(({ className, ...props }, ref) => (
  <DropdownMenuPrimitive.Separator
    ref={ref}
    className={cn("-mx-1 my-1 h-px bg-muted", className)}
    {...props}
  />
));
DropdownMenuSeparator.displayName = DropdownMenuPrimitive.Separator.displayName;

/**
 * DropdownMenu component keyboard shortcut.
 */
const DropdownMenuShortcut = ({
  className,
  ...props
}: React.HTMLAttributes<HTMLSpanElement>) => {
  return (
    <span
      className={cn("ml-auto text-xs tracking-widest opacity-60", className)}
      {...props}
    />
  );
};
DropdownMenuShortcut.displayName = "DropdownMenuShortcut";

export {
  DropdownMenu,
  DropdownMenuTrigger,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuCheckboxItem,
  DropdownMenuRadioItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuShortcut,
  DropdownMenuGroup,
  DropdownMenuPortal,
  DropdownMenuSub,
  DropdownMenuSubContent,
  DropdownMenuSubTrigger,
};

---

§ 18. SRC/COMPONENTS/UI/POPOVER.TSX

typescript
import * as React from "react";
import * as PopoverPrimitive from "@radix-ui/react-popover";

import { cn } from "@/lib/utils/cn";

/**
 * Popover component root.
 * @see https://www.radix-ui.com/primitives/docs/components/popover
 */
const Popover = PopoverPrimitive.Root;

/**
 * Popover component trigger.
 */
const PopoverTrigger = PopoverPrimitive.Trigger;

/**
 * Popover component content.
 */
const PopoverContent = React.forwardRef<
  React.ElementRef<typeof PopoverPrimitive.Content>,
  React.ComponentPropsWithoutRef<typeof PopoverPrimitive.Content>
>(({ className, align = "center", sideOffset = 4, ...props }, ref) => (
  <PopoverPrimitive.Portal>
    <PopoverPrimitive.Content
      ref={ref}
      align={align}
      sideOffset={sideOffset}
      className={cn(
        "z-50 w-72 rounded-md border bg-popover p-4 text-popover-foreground shadow-md outline-none data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2",
        className
      )}
      {...props}
    />
  </PopoverPrimitive.Portal>
));
PopoverContent.displayName = PopoverPrimitive.Content.displayName;

export { Popover, PopoverTrigger, PopoverContent };

---

§ 19. SRC/COMPONENTS/UI/TOOLTIP.TSX

typescript
import * as React from "react";
import * as TooltipPrimitive from "@radix-ui/react-tooltip";

import { cn } from "@/lib/utils/cn";

/**
 * Tooltip component provider.
 * Should wrap the entire application or a section where tooltips are used.
 */
const TooltipProvider = TooltipPrimitive.Provider;

/**
 * Tooltip component root.
 */
const Tooltip = TooltipPrimitive.Root;

/**
 * Tooltip component trigger.
 */
const TooltipTrigger = TooltipPrimitive.Trigger;

/**
 * Tooltip component content.
 */
const TooltipContent = React.forwardRef<
  React.ElementRef<typeof TooltipPrimitive.Content>,
  React.ComponentPropsWithoutRef<typeof TooltipPrimitive.Content>
>(({ className, sideOffset = 4, ...props }, ref) => (
  <TooltipPrimitive.Content
    ref={ref}
    sideOffset={sideOffset}
    className={cn(
      "z-50 overflow-hidden rounded-md bg-primary px-3 py-1.5 text-xs text-primary-foreground animate-in fade-in-0 zoom-in-95 data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=closed]:zoom-out-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2",
      className
    )}
    {...props}
  />
));
TooltipContent.displayName = TooltipPrimitive.Content.displayName;

export { Tooltip, TooltipTrigger, TooltipContent, TooltipProvider };

---

§ 20. SRC/COMPONENTS/UI/TABS.TSX

typescript
import * as React from "react";
import * as TabsPrimitive from "@radix-ui/react-tabs";
import { cva, type VariantProps } from "class-variance-authority";

import { cn } from "@/lib/utils/cn";

/**
 * Tabs component root.
 * @see https://www.radix-ui.com/primitives/docs/components/tabs
 */
const Tabs = TabsPrimitive.Root;

/**
 * Defines the variants for the TabsList component using CVA.
 */
const tabsListVariants = cva(
  "inline-flex items-center justify-center rounded-md p-1 text-muted-foreground",
  {
    variants: {
      variant: {
        default: "bg-muted",
        pills: "bg-transparent",
        underline: "bg-transparent border-b border-input rounded-none",
      },
    },
    defaultVariants: {
      variant: "default",
    },
  }
);

/**
 * Props for the TabsList component.
 * Extends standard Radix Tabs.List props and CVA variant props.
 */
export interface TabsListProps
  extends React.ComponentPropsWithoutRef<typeof TabsPrimitive.List>,
    VariantProps<typeof tabsListVariants> {}

/**
 * A list of tab triggers.
 * Supports different visual variants (default, pills, underline).
 *
 * @component
 * @example
 * <Tabs defaultValue="account">
 *   <TabsList>
 *     <TabsTrigger value="account">Account</TabsTrigger>
 *     <TabsTrigger value="password">Password</TabsTrigger>
 *   </TabsList>
 *   <TabsContent value="account">Make changes to your account here.</TabsContent>
 *   <TabsContent value="password">Change your password here.</TabsContent>
 * </Tabs>
 */
const TabsList = React.forwardRef<
  React.ElementRef<typeof TabsPrimitive.List>,
  TabsListProps
>(({ className, variant, ...props }, ref) => (
  <TabsPrimitive.List
    ref={ref}
    className={cn(tabsListVariants({ variant, className }))}
    {...props}
  />
));
TabsList.displayName = TabsPrimitive.List.displayName;

/**
 * Props for the TabsTrigger component.
 * Extends standard Radix Tabs.Trigger props.
 */
export interface TabsTriggerProps
  extends React.ComponentPropsWithoutRef<typeof TabsPrimitive.Trigger> {}

/**
 * An individual tab trigger.
 *
 * @component
 * @example
 * // Used within <TabsList>
 * <TabsTrigger value="account">Account</TabsTrigger>
 */
const TabsTrigger = React.forwardRef<
  React.ElementRef<typeof TabsPrimitive.Trigger>,
  TabsTriggerProps
>(({ className, ...props }, ref) => (
  <TabsPrimitive.Trigger
    ref={ref}
    className={cn(
      "inline-flex items-center justify-center whitespace-nowrap rounded-sm px-3 py-1.5 text-sm font-medium ring-offset-background transition-all focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 data-[state=active]:bg-background data-[state=active]:text-foreground data-[state=active]:shadow-sm",
      className
    )}
    {...props}
  />
));
TabsTrigger.displayName = TabsPrimitive.Trigger.displayName;

/**
 * Props for the TabsContent component.
 * Extends standard Radix Tabs.Content props.
 */
export interface TabsContentProps
  extends React.ComponentPropsWithoutRef<typeof TabsPrimitive.Content> {}

/**
 * Content panel associated with a tab.
 *
 * @component
 * @example
 * // Used within <Tabs>
 * <TabsContent value="account">Make changes to your account here.</TabsContent>
 */
const TabsContent = React.forwardRef<
  React.ElementRef<typeof TabsPrimitive.Content>,
  TabsContentProps
>(({ className, ...props }, ref) => (
  <TabsPrimitive.Content
    ref={ref}
    className={cn(
      "mt-2 ring-offset-background focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2",
      className
    )}
    {...props}
  />
));
TabsContent.displayName = TabsPrimitive.Content.displayName;

export { Tabs, TabsList, TabsTrigger, TabsContent };

---

§ 21. SRC/COMPONENTS/UI/ACCORDION.TSX

typescript
import * as React from "react";
import * as AccordionPrimitive from "@radix-ui/react-accordion";
import { ChevronDown } from "lucide-react";

import { cn } from "@/lib/utils/cn";

/**
 * Accordion component root.
 * @see https://www.radix-ui.com/primitives/docs/components/accordion
 */
const Accordion = AccordionPrimitive.Root;

/**
 * Props for the AccordionItem component.
 * Extends standard Radix Accordion.Item props.
 */
export interface AccordionItemProps
  extends React.ComponentPropsWithoutRef<typeof AccordionPrimitive.Item> {}

/**
 * An individual item within an Accordion.
 *
 * @component
 * @example
 * <Accordion type="single" collapsible>
 *   <AccordionItem value="item-1">
 *     <AccordionTrigger>Is it accessible?</AccordionTrigger>
 *     <AccordionContent>Yes. It adheres to the WAI-ARIA design pattern.</AccordionContent>
 *   </AccordionItem>
 * </Accordion>
 */
const AccordionItem = React.forwardRef<
  React.ElementRef<typeof AccordionPrimitive.Item>,
  AccordionItemProps
>(({ className, ...props }, ref) => (
  <AccordionPrimitive.Item
    ref={ref}
    className={cn("border-b", className)}
    {...props}
  />
));
AccordionItem.displayName = "AccordionItem";

/**
 * Props for the AccordionTrigger component.
 * Extends standard Radix Accordion.Trigger props.
 */
export interface AccordionTriggerProps
  extends React.ComponentPropsWithoutRef<typeof AccordionPrimitive.Trigger> {}

/**
 * The trigger button for an Accordion item.
 * Toggles the visibility of the associated content.
 *
 * @component
 * @example
 * // Used within <AccordionItem>
 * <AccordionTrigger>What is an accordion?</AccordionTrigger>
 */
const AccordionTrigger = React.forwardRef<
  React.ElementRef<typeof AccordionPrimitive.Trigger>,
  AccordionTriggerProps
>(({ className, children, ...props }, ref) => (
  <AccordionPrimitive.Header className="flex">
    <AccordionPrimitive.Trigger
      ref={ref}
      className={cn(
        "flex flex-1 items-center justify-between py-4 font-medium transition-all hover:underline [&[data-state=open]>svg]:rotate-180",
        className
      )}
      {...props}
    >
      {children}
      <ChevronDown className="h-4 w-4 shrink-0 transition-transform duration-200" />
    </AccordionPrimitive.Trigger>
  </AccordionPrimitive.Header>
));
AccordionTrigger.displayName = AccordionPrimitive.Trigger.displayName;

/**
 * Props for the AccordionContent component.
 * Extends standard Radix Accordion.Content props.
 */
export interface AccordionContentProps
  extends React.ComponentPropsWithoutRef<typeof AccordionPrimitive.Content> {}

/**
 * The collapsible content area for an Accordion item.
 *
 * @component
 * @example
 * // Used within <AccordionItem>
 * <AccordionContent>
 *   An accordion is a vertically stacked list of interactive headings.
 * </AccordionContent>
 */
const AccordionContent = React.forwardRef<
  React.ElementRef<typeof AccordionPrimitive.Content>,
  AccordionContentProps
>(({ className, children, ...props }, ref) => (
  <AccordionPrimitive.Content
    ref={ref}
    className="overflow-hidden text-sm transition-all data-[state=closed]:animate-accordion-up data-[state=open]:animate-accordion-down"
    {...props}
  >
    <div className={cn("pb-4 pt-0", className)}>{children}</div>
  </AccordionPrimitive.Content>
));
AccordionContent.displayName = AccordionPrimitive.Content.displayName;

export { Accordion, AccordionItem, AccordionTrigger, AccordionContent };

---

§ 22. SRC/COMPONENTS/UI/TABLE.TSX

typescript
import * as React from "react";

import { cn } from "@/lib/utils/cn";

/**
 * Props for the Table component.
 * Extends standard HTML table attributes.
 */
export interface TableProps
  extends React.HTMLAttributes<HTMLTableElement> {}

/**
 * A semantic table component for displaying tabular data.
 *
 * @component
 * @example
 * <Table>
 *   <TableCaption>A list of your recent invoices.</TableCaption>
 *   <TableHeader>
 *     <TableRow>
 *       <TableHead>Invoice</TableHead>
 *       <TableHead>Status</TableHead>
 *       <TableHead>Method</TableHead>
 *       <TableHead className="text-right">Amount</TableHead>
 *     </TableRow>
 *   </TableHeader>
 *   <TableBody>
 *     <TableRow>
 *       <TableCell className="font-medium">INV001</TableCell>
 *       <TableCell>Paid</TableCell>
 *       <TableCell>Credit Card</TableCell>
 *       <TableCell className="text-right">$250.00</TableCell>
 *     </TableRow>
 *   </TableBody>
 * </Table>
 */
const Table = React.forwardRef<HTMLTableElement, TableProps>(
  ({ className, ...props }, ref) => (
    <div className="relative w-full overflow-auto">
      <table
        ref={ref}
        className={cn("w-full caption-bottom text-sm", className)}
        {...props}
      />
    </div>
  )
);
Table.displayName = "Table";

/**
 * Props for the TableHeader component.
 * Extends standard HTML thead attributes.
 */
export interface TableHeaderProps
  extends React.HTMLAttributes<HTMLTableSectionElement> {}

/**
 * Header section of a Table.
 *
 * @component
 * @example
 * // Used within <Table>
 * <TableHeader>
 *   <TableRow>
 *     <TableHead>Name</TableHead>
 *     <TableHead>Email</TableHead>
 *   </TableRow>
 * </TableHeader>
 */
const TableHeader = React.forwardRef<
  HTMLTableSectionElement,
  TableHeaderProps
>(({ className, ...props }, ref) => (
  <thead ref={ref} className={cn("[&_tr]:border-b", className)} {...props} />
));
TableHeader.displayName = "TableHeader";

/**
 * Props for the TableBody component.
 * Extends standard HTML tbody attributes.
 */
export interface TableBodyProps
  extends React.HTMLAttributes<HTMLTableSectionElement> {}

/**
 * Body section of a Table, containing the main data rows.
 *
 * @component
 * @example
 * // Used within <Table>
 * <TableBody>
 *   <TableRow>
 *     <TableCell>John Doe</TableCell>
 *     <TableCell>john@example.com</TableCell>
 *   </TableRow>
 * </TableBody>
 */
const TableBody = React.forwardRef<
  HTMLTableSectionElement,
  TableBodyProps
>(({ className, ...props }, ref) => (
  <tbody
    ref={ref}
    className={cn("[&_tr:last-child]:border-0", className)}
    {...props}
  />
));
TableBody.displayName = "TableBody";

/**
 * Props for the TableFooter component.
 * Extends standard HTML tfoot attributes.
 */
export interface TableFooterProps
  extends React.HTMLAttributes<HTMLTableSectionElement> {}

/**
 * Footer section of a Table, often used for totals or summaries.
 *
 * @component
 * @example
 * // Used within <Table>
 * <TableFooter>
 *   <TableRow>
 *     <TableCell colSpan={3}>Total</TableCell>
 *     <TableCell className="text-right">$500.00</TableCell>
 *   </TableRow>
 * </TableFooter>
 */
const TableFooter = React.forwardRef<
  HTMLTableSectionElement,
  TableFooterProps
>(({ className, ...props }, ref) => (
  <tfoot
    ref={ref}
    className={cn(
      "border-t bg-muted/50 font-medium [&>tr]:last:border-b-0",
      className
    )}
    {...props}
  />
));
TableFooter.displayName = "TableFooter";

/**
 * Props for the TableRow component.
 * Extends standard HTML tr attributes.
 */
export interface TableRowProps extends React.HTMLAttributes<HTMLTableRowElement> {}

/**
 * A row within a Table.
 *
 * @component
 * @example
 * // Used within <TableHeader> or <TableBody> or <TableFooter>
 * <TableRow>
 *   <TableCell>Data 1</TableCell>
 *   <TableCell>Data 2</TableCell>
 * </TableRow>
 */
const TableRow = React.forwardRef<
  HTMLTableRowElement,
  TableRowProps
>(({ className, ...props }, ref) => (
  <tr
    ref={ref}
    className={cn(
      "border-b transition-colors hover:bg-muted/50 data-[state=selected]:bg-muted",
      className
    )}
    {...props}
  />
));
TableRow.displayName = "TableRow";

/**
 * Props for the TableHead component.
 * Extends standard HTML th attributes.
 */
export interface TableHeadProps
  extends React.ThHTMLAttributes<HTMLTableCellElement> {}

/**
 * A header cell within a TableRow in the TableHeader.
 *
 * @component
 * @example
 * // Used within <TableRow> in <TableHeader>
 * <TableHead>Product Name</TableHead>
 */
const TableHead = React.forwardRef<
  HTMLTableCellElement,
  TableHeadProps
>(({ className, ...props }, ref) => (
  <th
    ref={ref}
    className={cn(
      "h-12 px-4 text-left align-middle font-medium text-muted-foreground [&:has([role=checkbox])]:pr-0",
      className
    )}
    {...props}
  />
));
TableHead.displayName = "TableHead";

/**
 * Props for the TableCell component.
 * Extends standard HTML td attributes.
 */
export interface TableCellProps
  extends React.TdHTMLAttributes<HTMLTableCellElement> {}

/**
 * A data cell within a TableRow.
 *
 * @component
 * @example
 * // Used within <TableRow>
 * <TableCell>Item A</TableCell>
 */
const TableCell = React.forwardRef<
  HTMLTableCellElement,
  TableCellProps
>(({ className, ...props }, ref) => (
  <td
    ref={ref}
    className={cn("p-4 align-middle [&:has([role=checkbox])]:pr-0", className)}
    {...props}
  />
));
TableCell.displayName = "TableCell";

/**
 * Props for the TableCaption component.
 * Extends standard HTML caption attributes.
 */
export interface TableCaptionProps
  extends React.HTMLAttributes<HTMLTableCaptionElement> {}

/**
 * A caption for the Table, providing a description.
 *
 * @component
 * @example
 * // Used within <Table>
 * <TableCaption>A list of your recent invoices.</TableCaption>
 */
const TableCaption = React.forwardRef<
  HTMLTableCaptionElement,
  TableCaptionProps
>(({ className, ...props }, ref) => (
  <caption
    ref={ref}
    className={cn("mt-4 text-sm text-muted-foreground", className)}
    {...props}
  />
));
TableCaption.displayName = "TableCaption";

export {
  Table,
  TableHeader,
  TableBody,
  TableFooter,
  TableHead,
  TableRow,
  TableCell,
  TableCaption,
};

---

§ 23. SRC/COMPONENTS/UI/PAGINATION.TSX

typescript
import * as React from "react";
import { ChevronLeft, ChevronRight, MoreHorizontal } from "lucide-react";

import { cn } from "@/lib/utils/cn";
import { ButtonProps, buttonVariants } from "./button"; // Assuming Button component exists

/**
 * Props for the Pagination component.
 * Extends standard HTML nav attributes.
 */
export interface PaginationProps extends React.ComponentProps<"nav"> {}

/**
 * A navigation component for paginating through content.
 *
 * @component
 * @example
 * <Pagination>
 *   <PaginationContent>
 *     <PaginationItem>
 *       <PaginationPrevious href="#" />
 *     </PaginationItem>
 *     <PaginationItem>
 *       <PaginationLink href="#">1</PaginationLink>
 *     </PaginationItem>
 *     <PaginationItem>
 *       <PaginationLink href="#" isActive>2</PaginationLink>
 *     </PaginationItem>
 *     <PaginationItem>
 *       <PaginationLink href="#">3</PaginationLink>
 *     </PaginationItem>
 *     <PaginationItem>
 *       <PaginationEllipsis />
 *     </PaginationItem>
 *     <PaginationItem>
 *       <PaginationNext href="#" />
 *     </PaginationItem>
 *   </PaginationContent>
 * </Pagination>
 */
const Pagination = ({ className, ...props }: PaginationProps) => (
  <nav
    role="navigation"
    aria-label="Pagination"
    className={cn("mx-auto flex w-full justify-center", className)}
    {...props}
  />
);
Pagination.displayName = "Pagination";

/**
 * Props for the PaginationContent component.
 * Extends standard HTML ul attributes.
 */
export interface PaginationContentProps
  extends React.ComponentProps<"ul"> {}

/**
 * Container for pagination items.
 *
 * @component
 * @example
 * // Used within <Pagination>
 * <PaginationContent>...</PaginationContent>
 */
const PaginationContent = React.forwardRef<
  HTMLUListElement,
  PaginationContentProps
>(({ className, ...props }, ref) => (
  <ul
    ref={ref}
    className={cn("flex flex-row items-center gap-1", className)}
    {...props}
  />
));
PaginationContent.displayName = "PaginationContent";

/**
 * Props for the PaginationItem component.
 * Extends standard HTML li attributes.
 */
export interface PaginationItemProps
  extends React.ComponentProps<"li"> {}

/**
 * An individual item within pagination, typically a link or ellipsis.
 *
 * @component
 * @example
 * // Used within <PaginationContent>
 * <PaginationItem><PaginationLink href="#">1</PaginationLink></PaginationItem>
 */
const PaginationItem = React.forwardRef<
  HTMLLIElement,
  PaginationItemProps
>(({ className, ...props }, ref) => (
  <li ref={ref} className={cn("", className)} {...props} />
));
PaginationItem.displayName = "PaginationItem";

/**
 * Props for the PaginationLink component.
 * Extends ButtonProps and standard HTML a attributes.
 */
export interface PaginationLinkProps
  extends Pick<ButtonProps, "size">,
    React.ComponentProps<"a"> {
  /**
   * If true, the link will be styled as the active page.
   * @default false
   */
  isActive?: boolean;
}

/**
 * A link for a specific page in pagination.
 *
 * @component
 * @example
 * // Used within <PaginationItem>
 * <PaginationLink href="#" isActive>2</PaginationLink>
 */
const PaginationLink = ({
  className,
  isActive,
  size = "icon",
  ...props
}: PaginationLinkProps) => (
  <a
    aria-current={isActive ? "page" : undefined}
    className={cn(
      buttonVariants({
        variant: isActive ? "outline" : "ghost",
        size,
      }),
      className
    )}
    {...props}
  />
);
PaginationLink.displayName = "PaginationLink";

/**
 * Props for the PaginationPrevious component.
 * Extends PaginationLinkProps.
 */
export interface PaginationPreviousProps extends PaginationLinkProps {}

/**
 * Link to the previous page in pagination.
 *
 * @component
 * @example
 * // Used within <PaginationItem>
 * <PaginationPrevious href="#" />
 */
const PaginationPrevious = React.forwardRef<
  HTMLAnchorElement,
  PaginationPreviousProps
>(({ className, ...props }, ref) => (
  <PaginationLink
    aria-label="Go to previous page"
    size="default"
    className={cn("gap-1 pl-2.5", className)}
    ref={ref}
    {...props}
  >
    <ChevronLeft className="h-4 w-4" />
    <span>Previous</span>
  </PaginationLink>
));
PaginationPrevious.displayName = "PaginationPrevious";

/**
 * Props for the PaginationNext component.
 * Extends PaginationLinkProps.
 */
export interface PaginationNextProps extends PaginationLinkProps {}

/**
 * Link to the next page in pagination.
 *
 * @component
 * @example
 * // Used within <PaginationItem>
 * <PaginationNext href="#" />
 */
const PaginationNext = React.forwardRef<
  HTMLAnchorElement,
  PaginationNextProps
>(({ className, ...props }, ref) => (
  <PaginationLink
    aria-label="Go to next page"
    size="default"
    className={cn("gap-1 pr-2.5", className)}
    ref={ref}
    {...props}
  >
    <span>Next</span>
    <ChevronRight className="h-4 w-4" />
  </PaginationLink>
));
PaginationNext.displayName = "PaginationNext";

/**
 * Props for the PaginationEllipsis component.
 * Extends standard HTML span attributes.
 */
export interface PaginationEllipsisProps
  extends React.ComponentProps<"span"> {}

/**
 * Represents a skipped range of pages in pagination.
 *
 * @component
 * @example
 * // Used within <PaginationItem>
 * <PaginationEllipsis />
 */
const PaginationEllipsis = ({
  className,
  ...props
}: PaginationEllipsisProps) => (
  <span
    aria-hidden
    className={cn("flex h-9 w-9 items-center justify-center", className)}
    {...props}
  >
    <MoreHorizontal className="h-4 w-4" />
    <span className="sr-only">More pages</span>
  </span>
);
PaginationEllipsis.displayName = "PaginationEllipsis";

export {
  Pagination,
  PaginationContent,
  PaginationEllipsis,
  PaginationItem,
  PaginationLink,
  PaginationNext,
  PaginationPrevious,
};

---

§ 24. SRC/COMPONENTS/UI/BREADCRUMB.TSX

typescript
import * as React from "react";
import { ChevronRight, MoreHorizontal } from "lucide-react";
import { Slot } from "@radix-ui/react-slot";

import { cn } from "@/lib/utils/cn";

/**
 * Props for the Breadcrumb component.
 * Extends standard HTML nav attributes.
 */
export interface BreadcrumbProps
  extends React.ComponentPropsWithoutRef<"nav"> {
  /**
   * The separator element to use between breadcrumb items.
   * @default <ChevronRight />
   */
  separator?: React.ReactNode;
}

/**
 * A navigation component that indicates the current page's location within a hierarchical structure.
 *
 * @component
 * @example
 * <Breadcrumb>
 *   <BreadcrumbList>
 *     <BreadcrumbItem>
 *       <BreadcrumbLink href="/">Home</BreadcrumbLink>
 *     </BreadcrumbItem>
 *     <BreadcrumbSeparator />
 *     <BreadcrumbItem>
 *       <BreadcrumbLink href="/components">Components</BreadcrumbLink>
 *     </BreadcrumbItem>
 *     <BreadcrumbSeparator />
 *     <BreadcrumbItem>
 *       <BreadcrumbPage>Breadcrumb</BreadcrumbPage>
 *     </BreadcrumbItem>
 *   </BreadcrumbList>
 * </Breadcrumb>
 */
const Breadcrumb = React.forwardRef<HTMLElement, BreadcrumbProps>(
  ({ ...props }, ref) => <nav ref={ref} aria-label="breadcrumb" {...props} />
);
Breadcrumb.displayName = "Breadcrumb";

/**
 * Props for the BreadcrumbList component.
 * Extends standard HTML ol attributes.
 */
export interface BreadcrumbListProps
  extends React.ComponentPropsWithoutRef<"ol"> {}

/**
 * An ordered list container for breadcrumb items.
 *
 * @component
 * @example
 * // Used within <Breadcrumb>
 * <BreadcrumbList>...</BreadcrumbList>
 */
const BreadcrumbList = React.forwardRef<
  HTMLOListElement,
  BreadcrumbListProps
>(({ className, ...props }, ref) => (
  <ol
    ref={ref}
    className={cn(
      "flex flex-wrap items-center gap-1.5 break-words text-sm text-muted-foreground sm:gap-2.5",
      className
    )}
    {...props}
  />
));
BreadcrumbList.displayName = "BreadcrumbList";

/**
 * Props for the BreadcrumbItem component.
 * Extends standard HTML li attributes.
 */
export interface BreadcrumbItemProps
  extends React.ComponentPropsWithoutRef<"li"> {}

/**
 * An individual item within the breadcrumb list.
 *
 * @component
 * @example
 * // Used within <BreadcrumbList>
 * <BreadcrumbItem>
 *   <BreadcrumbLink href="/">Home</BreadcrumbLink>
 * </BreadcrumbItem>
 */
const BreadcrumbItem = React.forwardRef<
  HTMLLIElement,
  BreadcrumbItemProps
>(({ className, ...props }, ref) => (
  <li
    ref={ref}
    className={cn("inline-flex items-center", className)}
    {...props}
  />
));
BreadcrumbItem.displayName = "BreadcrumbItem";

/**
 * Props for the BreadcrumbLink component.
 * Extends standard HTML a attributes.
 */
export interface BreadcrumbLinkProps
  extends React.ComponentPropsWithoutRef<"a"> {
  /**
   * If true, the link will be rendered as a slot, allowing for custom elements.
   * @default false
   */
  asChild?: boolean;
}

/**
 * A clickable link within a breadcrumb item.
 *
 * @component
 * @example
 * // Used within <BreadcrumbItem>
 * <BreadcrumbLink href="/">Home</BreadcrumbLink>
 */
const BreadcrumbLink = React.forwardRef<
  HTMLAnchorElement,
  BreadcrumbLinkProps
>(({ asChild, className, ...props }, ref) => {
  const Comp = asChild ? Slot : "a";

  return (
    <Comp
      ref={ref}
      className={cn("transition-colors hover:text-foreground", className)}
      {...props}
    />
  );
});
BreadcrumbLink.displayName = "BreadcrumbLink";

/**
 * Props for the BreadcrumbPage component.
 * Extends standard HTML span attributes.
 */
export interface BreadcrumbPageProps
  extends React.ComponentPropsWithoutRef<"span"> {}

/**
 * Represents the current page in the breadcrumb, typically not a link.
 *
 * @component
 * @example
 * // Used within <BreadcrumbItem>
 * <BreadcrumbPage>Current Page</BreadcrumbPage>
 */
const BreadcrumbPage = React.forwardRef<
  HTMLSpanElement,
  BreadcrumbPageProps
>(({ className, ...props }, ref) => (
  <span
    ref={ref}
    role="link"
    aria-disabled="true"
    aria-current="page"
    className={cn("font-normal text-foreground", className)}
    {...props}
  />
));
BreadcrumbPage.displayName = "BreadcrumbPage";

/**
 * Props for the BreadcrumbSeparator component.
 * Extends standard HTML li attributes.
 */
export interface BreadcrumbSeparatorProps
  extends React.ComponentPropsWithoutRef<"li"> {
  /**
   * If true, the separator will be rendered as a slot, allowing for custom elements.
   * @default false
   */
  asChild?: boolean;
}

/**
 * A visual separator between breadcrumb items.
 *
 * @component
 * @example
 * // Used within <BreadcrumbList>
 * <BreadcrumbSeparator />
 */
const BreadcrumbSeparator = React.forwardRef<
  HTMLLIElement,
  BreadcrumbSeparatorProps
>(({ asChild, className, ...props }, ref) => {
  const Comp = asChild ? Slot : "li";
  return (
    <Comp
      ref={ref}
      role="presentation"
      aria-hidden="true"
      className={cn("[&>svg]:h-3.5 [&>svg]:w-3.5", className)}
      {...props}
    >
      {!asChild && <ChevronRight className="h-3.5 w-3.5" />}
    </Comp>
  );
});
BreadcrumbSeparator.displayName = "BreadcrumbSeparator";

/**
 * Props for the BreadcrumbEllipsis component.
 * Extends standard HTML span attributes.
 */
export interface BreadcrumbEllipsisProps
  extends React.ComponentPropsWithoutRef<"span"> {}

/**
 * Represents a collapsed section of breadcrumb items.
 *
 * @component
 * @example
 * // Used within <BreadcrumbItem>
 * <BreadcrumbEllipsis />
 */
const BreadcrumbEllipsis = React.forwardRef<
  HTMLSpanElement,
  BreadcrumbEllipsisProps
>(({ className, ...props }, ref) => (
  <span
    ref={ref}
    role="presentation"
    aria-hidden="true"
    className={cn("flex h-9 w-9 items-center justify-center", className)}
    {...props}
  >
    <MoreHorizontal className="h-4 w-4" />
    <span className="sr-only">More pages</span>
  </span>
));
BreadcrumbEllipsis.displayName = "BreadcrumbEllipsis";

export {
  Breadcrumb,
  BreadcrumbList,
  BreadcrumbItem,
  BreadcrumbLink,
  BreadcrumbPage,
  BreadcrumbSeparator,
  BreadcrumbEllipsis,
};

---

§ 25. SRC/COMPONENTS/UI/SEPARATOR.TSX

typescript
import * as React from "react";
import * as SeparatorPrimitive from "@radix-ui/react-separator";

import { cn } from "@/lib/utils/cn";

/**
 * Props for the Separator component.
 * Extends standard Radix Separator.Root props.
 */
export interface SeparatorProps
  extends React.ComponentPropsWithoutRef<typeof SeparatorPrimitive.Root> {}

/**
 * A visual separator component built with Radix UI and styled with Tailwind CSS.
 * Used to distinguish content groups.
 *
 * @component
 * @example
 * <div className="space-y-1">
 *   <h4 className="text-sm font-medium leading-none">Radix Primitives</h4>
 *   <p className="text-sm text-muted-foreground">An open-source UI component library.</p>
 * </div>
 * <Separator className="my-4" />
 * <div className="flex h-5 items-center space-x-4 text-sm">
 *   <div>Blog</div>
 *   <Separator orientation="vertical" />
 *   <div>Docs</div>
 *   <Separator orientation="vertical" />
 *   <div>Source</div>
 * </div>
 */
const Separator = React.forwardRef<
  React.ElementRef<typeof SeparatorPrimitive.Root>,
  SeparatorProps
>(
  (
    { className, orientation = "horizontal", decorative = true, ...props },
    ref
  ) => (
    <SeparatorPrimitive.Root
      ref={ref}
      decorative={decorative}
      orientation={orientation}
      className={cn(
        "shrink-0 bg-border",
        orientation === "horizontal" ? "h-[1px] w-full" : "h-full w-[1px]",
        className
      )}
      {...props}
    />
  )
);
Separator.displayName = SeparatorPrimitive.Root.displayName;

export { Separator };

---

§ 26. SRC/COMPONENTS/UI/SKELETON.TSX

typescript
import { cn } from "@/lib/utils/cn";

/**
 * Props for the Skeleton component.
 * Extends standard HTML div attributes.
 */
export interface SkeletonProps
  extends React.HTMLAttributes<HTMLDivElement> {}

/**
 * A skeleton loading component used as a placeholder for content that is still loading.
 * Provides a visual indication of loading state.
 *
 * @component
 * @example
 * <div className="flex items-center space-x-4">
 *   <Skeleton className="h-12 w-12 rounded-full" />
 *   <div className="space-y-2">
 *     <Skeleton className="h-4 w-[250px]" />
 *     <Skeleton className="h-4 w-[200px]" />
 *   </div>
 * </div>
 */
function Skeleton({ className, ...props }: SkeletonProps) {
  return (
    <div
      className={cn("animate-pulse rounded-md bg-muted", className)}
      {...props}
    />
  );
}

export { Skeleton };

---

§ 27. SRC/COMPONENTS/UI/SPINNER.TSX

typescript
import * as React from "react";
import { cn } from "@/lib/utils/cn";

/**
 * Props for the Spinner component.
 * Extends standard HTML SVG attributes.
 */
export interface SpinnerProps extends React.SVGProps<SVGSVGElement> {
  /**
   * The size of the spinner.
   * @default "md"
   */
  size?: "sm" | "md" | "lg";
  /**
   * The color of the spinner.
   * @default "currentColor"
   */
  color?: string;
}

/**
 * A simple SVG spinner component for indicating loading states.
 *
 * @component
 * @example
 * <Spinner />
 * <Spinner size="lg" color="text

---
_Modello: gemini-2.5-flash (Google AI Studio) | Token: 33652_