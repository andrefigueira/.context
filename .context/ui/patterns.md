# UI: Implementation Patterns

This document defines reusable UI patterns, interaction behaviors, and component composition strategies.

## Data Display Patterns

### Data Table Specification

```yaml
component: DataTable
category: patterns
status: stable

features:
  sorting:
    type: single | multi
    default: single
    indicator: chevron icon in header
  pagination:
    type: cursor | offset
    default: offset
    options: [10, 25, 50, 100]
  selection:
    type: none | single | multi
    default: none
  filtering:
    types: [text, select, date-range, number-range]

columns:
  definition:
    - id: string (required, unique)
    - header: string | ReactNode
    - accessor: string | function
    - sortable: boolean
    - width: number | "auto"
    - align: "left" | "center" | "right"

states:
  loading:
    display: skeleton rows
    rowCount: matches pageSize
  empty:
    display: centered message + optional action
  error:
    display: error message + retry button
```

### Implementation

```tsx
// components/patterns/DataTable/DataTable.tsx
interface Column<T> {
  id: string;
  header: string | ReactNode;
  accessor: keyof T | ((row: T) => ReactNode);
  sortable?: boolean;
  width?: number | 'auto';
  align?: 'left' | 'center' | 'right';
}

interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  loading?: boolean;
  error?: Error | null;
  pagination?: {
    page: number;
    pageSize: number;
    total: number;
    onPageChange: (page: number) => void;
    onPageSizeChange: (size: number) => void;
  };
  sorting?: {
    sortBy: string | null;
    sortOrder: 'asc' | 'desc';
    onSort: (columnId: string) => void;
  };
  selection?: {
    selected: string[];
    onSelect: (ids: string[]) => void;
    getRowId: (row: T) => string;
  };
  emptyState?: {
    title: string;
    description?: string;
    action?: ReactNode;
  };
}

export function DataTable<T>({
  data,
  columns,
  loading,
  error,
  pagination,
  sorting,
  selection,
  emptyState,
}: DataTableProps<T>) {
  if (loading) {
    return <TableSkeleton columns={columns.length} rows={pagination?.pageSize ?? 10} />;
  }

  if (error) {
    return (
      <TableError
        message={error.message}
        onRetry={() => window.location.reload()}
      />
    );
  }

  if (data.length === 0) {
    return (
      <TableEmpty
        title={emptyState?.title ?? 'No data'}
        description={emptyState?.description}
        action={emptyState?.action}
      />
    );
  }

  return (
    <div className="overflow-x-auto">
      <table className="w-full">
        <TableHeader
          columns={columns}
          sorting={sorting}
          selection={selection}
          allSelected={selection?.selected.length === data.length}
        />
        <TableBody
          data={data}
          columns={columns}
          selection={selection}
        />
      </table>
      {pagination && <TablePagination {...pagination} />}
    </div>
  );
}
```

## Form Patterns

### Form Field Specification

```yaml
component: FormField
category: compounds

structure:
  - label (optional)
  - description (optional)
  - input slot
  - error message (conditional)
  - character count (optional)

layout:
  vertical:
    spacing: 4px between elements
    labelPosition: above input
  horizontal:
    labelWidth: 120px | 1/3
    labelPosition: left of input

validation:
  timing:
    - onBlur: validate after user leaves field
    - onChange: validate during typing (debounced 300ms)
    - onSubmit: validate all fields before submission
  display:
    error: red border + error text below
    success: green border (optional)
    warning: yellow border + warning text
```

### Multi-Step Form Pattern

```tsx
// components/patterns/MultiStepForm/MultiStepForm.tsx
interface Step {
  id: string;
  title: string;
  description?: string;
  validate?: () => Promise<boolean>;
  component: ReactNode;
}

interface MultiStepFormProps {
  steps: Step[];
  onComplete: (data: Record<string, unknown>) => void;
  onCancel?: () => void;
}

export function MultiStepForm({ steps, onComplete, onCancel }: MultiStepFormProps) {
  const [currentStep, setCurrentStep] = useState(0);
  const [formData, setFormData] = useState<Record<string, unknown>>({});
  const [completedSteps, setCompletedSteps] = useState<Set<number>>(new Set());

  const goToNext = async () => {
    const step = steps[currentStep];
    if (step.validate) {
      const isValid = await step.validate();
      if (!isValid) return;
    }
    setCompletedSteps(prev => new Set([...prev, currentStep]));
    if (currentStep < steps.length - 1) {
      setCurrentStep(prev => prev + 1);
    } else {
      onComplete(formData);
    }
  };

  const goToPrevious = () => {
    if (currentStep > 0) {
      setCurrentStep(prev => prev - 1);
    }
  };

  return (
    <div className="space-y-6">
      <StepIndicator
        steps={steps}
        currentStep={currentStep}
        completedSteps={completedSteps}
        onStepClick={(index) => {
          if (completedSteps.has(index) || index < currentStep) {
            setCurrentStep(index);
          }
        }}
      />

      <FormDataContext.Provider value={{ formData, setFormData }}>
        <div className="min-h-[300px]">
          {steps[currentStep].component}
        </div>
      </FormDataContext.Provider>

      <div className="flex justify-between">
        <Button
          variant="ghost"
          onClick={currentStep === 0 ? onCancel : goToPrevious}
        >
          {currentStep === 0 ? 'Cancel' : 'Back'}
        </Button>
        <Button onClick={goToNext}>
          {currentStep === steps.length - 1 ? 'Complete' : 'Continue'}
        </Button>
      </div>
    </div>
  );
}
```

## Modal & Dialog Patterns

### Modal Specification

```yaml
component: Modal
category: compounds

behavior:
  open:
    - fade in backdrop (150ms)
    - scale + fade in content (200ms)
    - trap focus inside modal
    - prevent body scroll
  close:
    triggers:
      - close button click
      - backdrop click (configurable)
      - Escape key
    animation:
      - fade out content (150ms)
      - fade out backdrop (150ms)
    cleanup:
      - restore focus to trigger element
      - restore body scroll

sizing:
  sm: "max-w-md"   # 448px
  md: "max-w-lg"   # 512px
  lg: "max-w-2xl"  # 672px
  xl: "max-w-4xl"  # 896px
  full: "max-w-full mx-4"

accessibility:
  role: dialog
  aria-modal: true
  aria-labelledby: "references title element"
  aria-describedby: "references description element"
```

### Confirmation Dialog Pattern

```tsx
// hooks/useConfirmation.tsx
interface ConfirmationOptions {
  title: string;
  description: string;
  confirmText?: string;
  cancelText?: string;
  variant?: 'default' | 'destructive';
}

const ConfirmationContext = createContext<{
  confirm: (options: ConfirmationOptions) => Promise<boolean>;
} | null>(null);

export function ConfirmationProvider({ children }: { children: ReactNode }) {
  const [state, setState] = useState<{
    open: boolean;
    options: ConfirmationOptions | null;
    resolve: ((value: boolean) => void) | null;
  }>({ open: false, options: null, resolve: null });

  const confirm = useCallback((options: ConfirmationOptions) => {
    return new Promise<boolean>((resolve) => {
      setState({ open: true, options, resolve });
    });
  }, []);

  const handleConfirm = () => {
    state.resolve?.(true);
    setState({ open: false, options: null, resolve: null });
  };

  const handleCancel = () => {
    state.resolve?.(false);
    setState({ open: false, options: null, resolve: null });
  };

  return (
    <ConfirmationContext.Provider value={{ confirm }}>
      {children}
      <AlertDialog open={state.open} onOpenChange={handleCancel}>
        <AlertDialogContent>
          <AlertDialogHeader>
            <AlertDialogTitle>{state.options?.title}</AlertDialogTitle>
            <AlertDialogDescription>
              {state.options?.description}
            </AlertDialogDescription>
          </AlertDialogHeader>
          <AlertDialogFooter>
            <AlertDialogCancel onClick={handleCancel}>
              {state.options?.cancelText ?? 'Cancel'}
            </AlertDialogCancel>
            <AlertDialogAction
              onClick={handleConfirm}
              className={state.options?.variant === 'destructive' ? 'bg-red-500' : ''}
            >
              {state.options?.confirmText ?? 'Confirm'}
            </AlertDialogAction>
          </AlertDialogFooter>
        </AlertDialogContent>
      </AlertDialog>
    </ConfirmationContext.Provider>
  );
}

// Usage
function DeleteUserButton({ userId }: { userId: string }) {
  const { confirm } = useConfirmation();
  const deleteUser = useDeleteUser();

  const handleDelete = async () => {
    const confirmed = await confirm({
      title: 'Delete User',
      description: 'This action cannot be undone. All user data will be permanently deleted.',
      confirmText: 'Delete',
      variant: 'destructive',
    });

    if (confirmed) {
      await deleteUser.mutateAsync(userId);
    }
  };

  return <Button variant="destructive" onClick={handleDelete}>Delete</Button>;
}
```

## Loading & Error States

### Loading State Patterns

```yaml
loading_patterns:
  skeleton:
    use_when: "Content structure is predictable"
    implementation: "Match exact layout of loaded content"
    animation: "pulse or shimmer"

  spinner:
    use_when: "Unknown content structure or small areas"
    sizes: [sm: 16px, md: 24px, lg: 32px]
    placement: "center of loading area"

  progress:
    use_when: "Known duration or percentage"
    types: [determinate, indeterminate]

  optimistic:
    use_when: "High confidence action will succeed"
    implementation: "Update UI immediately, rollback on error"
```

### Error Boundary Pattern

```tsx
// components/ErrorBoundary.tsx
interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<
  { children: ReactNode; fallback?: ReactNode },
  ErrorBoundaryState
> {
  state: ErrorBoundaryState = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('ErrorBoundary caught:', error, errorInfo);
    // Send to error tracking service
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? (
        <div className="p-8 text-center">
          <h2 className="text-lg font-semibold text-red-600">Something went wrong</h2>
          <p className="mt-2 text-gray-600">{this.state.error?.message}</p>
          <Button
            className="mt-4"
            onClick={() => this.setState({ hasError: false, error: null })}
          >
            Try again
          </Button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

## Animation Specifications

```yaml
animations:
  micro_interactions:
    button_press:
      transform: "scale(0.98)"
      duration: "100ms"
      easing: "ease-out"

    hover_lift:
      transform: "translateY(-1px)"
      shadow: "shadow-md"
      duration: "150ms"

  transitions:
    fade:
      enter: "opacity 0 -> 1, 150ms ease-out"
      exit: "opacity 1 -> 0, 100ms ease-in"

    slide_up:
      enter: "translateY(8px) -> 0, opacity 0 -> 1, 200ms ease-out"
      exit: "translateY(0) -> 8px, opacity 1 -> 0, 150ms ease-in"

    scale:
      enter: "scale(0.95) -> 1, opacity 0 -> 1, 200ms ease-out"
      exit: "scale(1) -> 0.95, opacity 1 -> 0, 150ms ease-in"

  page_transitions:
    default:
      type: "fade"
      duration: "200ms"

  reduced_motion:
    behavior: "Remove transform animations, keep opacity only"
    media_query: "(prefers-reduced-motion: reduce)"
```

## Responsive Behavior

```yaml
responsive_patterns:
  navigation:
    desktop: "Horizontal nav bar with dropdowns"
    tablet: "Horizontal nav with hamburger for overflow"
    mobile: "Full-screen drawer navigation"
    breakpoint: "lg (1024px)"

  data_table:
    desktop: "Full table with all columns"
    tablet: "Horizontal scroll with sticky first column"
    mobile: "Card view with expandable details"
    breakpoint: "md (768px)"

  sidebar:
    desktop: "Fixed sidebar, always visible"
    tablet: "Collapsible sidebar with toggle"
    mobile: "Off-canvas drawer"
    breakpoint: "lg (1024px)"

  forms:
    desktop: "Multi-column layout where appropriate"
    mobile: "Single column, full-width inputs"
    breakpoint: "sm (640px)"
```

## Decision History & Trade-offs

### Animation Library: CSS vs Framer Motion
**Decision**: Use CSS for simple transitions, Framer Motion for complex animations
**Rationale**:
- CSS handles 90% of micro-interactions efficiently
- Framer Motion for gesture-based interactions and complex sequences
- Reduces bundle size for simple use cases

**Trade-offs**:
- Two animation systems to maintain
- Developers need to know when to use which

### State Management for UI: Zustand vs Context
**Decision**: Zustand for global UI state, Context for component trees
**Rationale**:
- Zustand avoids Context re-render issues
- Simpler API than Redux
- Context still useful for scoped state (forms, modals)

**Trade-offs**:
- Additional dependency
- Team must learn Zustand patterns

---

**Next**: Review [../architecture/patterns.md](../architecture/patterns.md) for backend integration patterns.
