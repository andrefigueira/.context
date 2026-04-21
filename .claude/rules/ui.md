# Rule: UI

Rules for frontend components, pages, and user-facing code.

## Applies to
`src/components/**`, `app/**` (non-page files), `*.tsx`, `*.jsx`, `*.vue`, `*.svelte`

For page-level files (`src/pages/**`, `src/app/**/page.*`), both `ui.md` and `seo.md` apply. Load both.

## Hard rules
1. Use existing design tokens. Do not inline hex colors, spacing, or font sizes
2. Use existing components before writing a new one. Check the design system first
3. Every interactive element is keyboard accessible and has a visible focus state
4. Images require `alt` text; decorative images use empty `alt=""`
5. Forms have labels, not just placeholders. Error messages are announced to screen readers
6. No business logic in components. Fetch, transform, and mutate in hooks or services
7. Loading, empty, and error states are first-class. Do not ship a component without them
8. No layout shift on load. Reserve space for async content

## Patterns

Component with the three states (adapt to your framework):

```tsx
export function UserCard({ id }: { id: string }) {
  const { data, error, isLoading } = useUser(id);
  if (isLoading) return <Skeleton />;
  if (error) return <ErrorState error={error} />;
  if (!data) return <EmptyState />;
  return <Card>{/* render */}</Card>;
}
```

## Load on demand
- Component architecture and design tokens → `.context/ui/overview.md`
- UI implementation patterns (forms, modals, tables) → `.context/ui/patterns.md`
