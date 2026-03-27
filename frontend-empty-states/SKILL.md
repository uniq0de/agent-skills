---
name: frontend-empty-states
description: Golden guideline for designing empty states and loading states in CRM applications. Use when a list, table, card, or section has no data to display, or while data is being fetched.
---

# Frontend Empty & Loading States Design Guideline

Empty and loading states are the first impression for new users and an important recovery point when filters return no results. They must be instructive, not just decorative.

## Table / List Empty State

Blueprint reference: `frontend-dashboard/table-sample.tsx`.

Centered vertically and horizontally within `CardContent`, with a muted icon, clear heading, and context-aware message:

```tsx
<Box
  sx={{
    display: "flex",
    flexDirection: "column",
    alignItems: "center",
    justifyContent: "center",
    py: 6,
    px: 2,
  }}
>
  <AssignmentIcon sx={{ fontSize: 60, color: "text.secondary", mb: 2 }} />
  <Typography variant="h6" color="text.secondary" sx={{ mb: 1 }}>
    No active tasks yet
  </Typography>
  <Typography variant="body2" color="text.secondary" sx={{ mb: 3, maxWidth: 300 }}>
    Create your first task to get started managing task definitions and versions.
  </Typography>
  <Button variant="contained" color="primary">
    Create Task
  </Button>
</Box>
```

Key details:
- Use MUI layout and typography primitives for consistency with listing pages.
- Keep icon + heading + body copy vertically centered.
- Support two message types: "no data exists" and "filtered/no matches".
- Include a primary recovery action when the empty state represents a first-run experience.

## Card-List Empty State (Medical Records Pattern)

For card-list pages (rows rendered as bordered divs), switch between empty and populated states at the `CardContent` level:

```tsx
<CardContent>
  {filteredItems.length === 0 ? (
    <div className="flex flex-col items-center justify-center py-12 text-center">
      <EntityIcon className="size-12 text-muted-foreground/50 mb-4" />
      <h3 className="text-lg font-medium">No records found</h3>
      <p className="text-muted-foreground">
        {searchQuery || activeFilter !== 'all'
          ? 'Try adjusting your filters'
          : 'Add records to track history'}
      </p>
    </div>
  ) : (
    <div className="space-y-4">
      {/* card rows */}
    </div>
  )}
</CardContent>
```

The ternary controls the entire content region — never render empty text and rows at the same time.

## Small Card Empty State (no py-12)

For smaller cards (e.g., sidebar cards on detail pages), use less padding:

```tsx
<div className="text-center py-4 text-muted-foreground text-sm">
  No records scheduled
</div>
```

Just a single centered line of muted text. No icon needed for small contexts.

## Loading State

A centered pulsing text. Keep it minimal — do not use skeleton cards or spinners:

```tsx
if (isLoading) {
  return (
    <div className="flex items-center justify-center p-8">
      <div className="animate-pulse text-muted-foreground">Loading records...</div>
    </div>
  )
}
```

`animate-pulse` provides a breathing fade effect. The text uses the entity name: "Loading pets...", "Loading appointments...", not just "Loading...".

## Not Found State (Detail Page)

When navigating to a detail page for a non-existent entity:

```tsx
if (!entity) {
  return (
    <div className="flex flex-col items-center justify-center p-8 text-center">
      <EntityIcon className="size-12 text-muted-foreground/50 mb-4" />
      <h2 className="text-xl font-semibold">Record Not Found</h2>
      <p className="text-muted-foreground mb-4">The requested record could not be found.</p>
      <Button asChild>
        <Link href="/entity-list">
          <ArrowLeft className="size-4 mr-2" />
          Back to List
        </Link>
      </Button>
    </div>
  )
}
```

This state always includes a recovery button to navigate back. The heading uses `text-xl font-semibold` (slightly larger than the list empty state).

## Chat / Assistant Empty State (with suggestions)

For chat interfaces with no messages, show a centered prompt + clickable suggestion chips:

```tsx
<div className="flex flex-col items-center justify-center h-full text-center py-12 px-2">
  <MessageSquare className="size-12 text-muted-foreground/50 mb-4" />
  <h3 className="text-lg font-medium">How can I help you today?</h3>
  <p className="text-sm text-muted-foreground max-w-md mt-1">
    Brief description of what this assistant can do.
  </p>
  <div className="flex flex-wrap justify-center gap-2 mt-6">
    {suggestions.map((suggestion) => (
      <Button
        key={suggestion}
        variant="outline"
        size="sm"
        className="text-xs md:text-sm bg-transparent"
        onClick={() => setInput(suggestion)}
      >
        {suggestion}
      </Button>
    ))}
  </div>
</div>
```

Suggestion chips: `variant="outline" size="sm" className="bg-transparent"`. They set the input field value, not trigger a submit.

## Context-Aware Messages Reference

| Situation | Message |
|---|---|
| No records, no filters | "Get started by adding your first [record]" |
| Filters active, no results | "Try adjusting your filters" |
| Search active, no results | "Try adjusting your search" |
| Both search + filter active | "Try adjusting your filters" |
| Loading | "Loading [records]..." |
| Not found | "The requested [record] could not be found." |
| No appointments | "No appointments scheduled" |
| All clear | "All clear for now" |

## Rules

- The empty state icon is always the entity's own icon, not a generic placeholder. For appointments: `Calendar`. For pets: `PawPrint`. For messages: `MessageSquare`.
- Icon opacity is always `text-muted-foreground/50` — never full opacity, never lower than `/40`.
- Allow a primary CTA button in empty state when first-run onboarding benefits from an immediate action.
- Loading state is shown at the page level (before the Card), not inside the table body.
- Always guard the not-found state before rendering any layout on detail pages.
