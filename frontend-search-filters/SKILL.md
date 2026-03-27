---
name: frontend-search-filters
description: Golden guideline for designing search inputs and filter pill systems in CRM listing pages. Use when adding search, filtering by category/status/type, or combining multiple filter dimensions in any list or table view.
---

# Frontend Search & Filter Design Guideline

Search and filters appear together inside the CardHeader of a listing page. They share one row on desktop and stack on mobile.

## Search Input

Always use a relative container to position the search icon absolutely, with left-padded input.

```tsx
<div className="relative">
  <Search className="absolute left-3 top-1/2 size-4 -translate-y-1/2 text-muted-foreground" />
  <Input
    placeholder="Search records..."
    value={searchQuery}
    onChange={(e) => setSearchQuery(e.target.value)}
    className="pl-9 w-full sm:w-64"
  />
</div>
```

- Icon: `size-4`, color `text-muted-foreground`
- Input: `pl-9` (leaves room for icon), fixed width `sm:w-64` on desktop, `w-full` on mobile
- Placeholder: "Search [entity]..." always lowercase after "Search"
- Search is always real-time (no submit button)

## Filter Pills

Use pill-shaped toggle buttons for categorical filters. Never use `<select>` or `<DropdownMenu>` for filters — pills are scannable and require fewer clicks.

```tsx
<div className="flex gap-1 flex-wrap">
  {filterOptions.map((option) => (
    <button
      key={option.value}
      type="button"
      onClick={() => setFilter(option.value === filter ? 'all' : option.value)}
      className={`px-3 py-1.5 text-sm rounded-full transition-colors ${
        filter === option.value
          ? 'bg-primary text-primary-foreground'
          : 'bg-muted text-muted-foreground hover:bg-muted/80'
      }`}
    >
      {option.label}
    </button>
  ))}
</div>
```

Key behaviors:
- **Toggle deselect**: clicking the active filter resets it to `'all'` — `option.value === filter ? 'all' : option.value`
- **Always include an "All" option** as the first item in the options array
- `flex-wrap` allows pills to wrap gracefully on narrow screens

## Filter Options Array

Define filter options as a typed array. Always start with `'all'`:

```tsx
const statusOptions: { value: StatusFilter; label: string }[] = [
  { value: 'all', label: 'All Status' },
  { value: 'scheduled', label: 'Scheduled' },
  { value: 'confirmed', label: 'Confirmed' },
  { value: 'completed', label: 'Completed' },
  { value: 'cancelled', label: 'Cancelled' },
]
```

## Multiple Filter Dimensions

When you have multiple filter categories (e.g. Status + Type), place each on its own row:

```tsx
<CardHeader className="pb-4">
  <div className="flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
    <div>{/* title + count */}</div>
    <div className="flex flex-col gap-2 sm:flex-row sm:items-center">
      <SearchInput />
      <StatusPills />  {/* first dimension inline with search */}
    </div>
  </div>
  <TypePills className="mt-2" />  {/* second dimension on its own row */}
</CardHeader>
```

## Search + Filter Pills Layout

Use this exact `CardHeader` pattern for clean, intuitive filtering UX:

```tsx
<CardHeader className="pb-4">
  <div className="flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
    <div>
      <CardTitle>Patient Records</CardTitle>
      <CardDescription>{filteredRecords.length} records on file</CardDescription>
    </div>
    <div className="relative">
      <Search className="absolute left-3 top-1/2 size-4 -translate-y-1/2 text-muted-foreground" />
      <Input
        placeholder="Search records..."
        value={searchQuery}
        onChange={(e) => setSearchQuery(e.target.value)}
        className="pl-9 w-full sm:w-64"
      />
    </div>
  </div>

  <div className="flex gap-1 flex-wrap mt-2">
    {typeOptions.map((option) => (
      <button
        key={option.value}
        type="button"
        onClick={() => setTypeFilter(option.value === typeFilter ? 'all' : option.value)}
        className={`px-3 py-1.5 text-sm rounded-full transition-colors ${
          typeFilter === option.value
            ? 'bg-primary text-primary-foreground'
            : 'bg-muted text-muted-foreground hover:bg-muted/80'
        }`}
      >
        {option.label}
      </button>
    ))}
  </div>
</CardHeader>
```

Spacing and interaction details from the blueprint:
- `CardHeader` uses `pb-4` to avoid oversized whitespace below controls.
- Pills row uses `mt-2` and `gap-1` for compact grouping.
- Pills use `px-3 py-1.5 rounded-full` for a clear "filter chip" silhouette.
- Active state is always `bg-primary text-primary-foreground`.
- Inactive state is always `bg-muted text-muted-foreground hover:bg-muted/80`.

## Context-Aware Empty State Messages

When filters are active and nothing matches, the empty state message must reflect this:

```tsx
<p className="text-muted-foreground">
  {searchQuery || filter !== 'all'
    ? 'Try adjusting your filters'
    : 'Get started by adding your first record'}
</p>
```

Never show "No records found" — always explain *why* and give a next step.

## Rules

- Max 6 pill options per filter dimension. If you have more, use a dropdown instead.
- Search always searches across the most important 2-3 fields (name, email/identifier, related entity name).
- Never add a "Clear filters" button unless there are 3+ active filters simultaneously.
- Filters are always client-side — never trigger a server request per filter change.
- The search input and first filter row share the same horizontal line on sm+ screens.
- Use `CardHeader className="pb-4"` whenever filter pills are present.
- Keep pill spacing tight (`gap-1`) and use canonical padding (`px-3 py-1.5`).

