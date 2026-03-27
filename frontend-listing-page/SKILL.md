---
name: frontend-listing-page
description: Golden guideline for designing CRM listing/index pages with tables, search, and filter chips. Use when building any list view, index page, data table, or record browser in a CRM, admin panel, or dashboard application.
---

# Frontend Listing Page Design Guideline

A listing page presents a searchable, filterable table of records with inline actions. It is the most common page type in CRM apps.

## Page Wrapper Structure

Every listing page follows this exact shell:

```tsx
<div className="p-4 md:p-6 space-y-6">
  {/* 1. Page header (title + primary CTA) */}
  {/* 2. Card with search/filters + table */}
  {/* 3. Dialogs (form, confirm delete) */}
</div>
```

## Page Header

Title on the left, primary action button on the right. On mobile, stack them vertically; on sm+ align them horizontally.

```tsx
<div className="flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
  <div>
    <h1 className="text-2xl font-bold tracking-tight">Pets</h1>
    <p className="text-muted-foreground">Manage your patient records</p>
  </div>
  <Button onClick={() => setDialogOpen(true)}>
    <Plus className="size-4 mr-2" />
    Add Pet
  </Button>
</div>
```

The subtitle under the H1 is always `text-muted-foreground` and describes what this section manages in 4-6 words.

## Card with Filters

Wrap the entire table (including search/filters) in a single Card. The CardHeader contains the controls; the CardContent contains the table.

```tsx
<Card>
  <CardHeader className="pb-4">
    <div className="flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
      <div>
        <CardTitle>Patient Records</CardTitle>
        <CardDescription>{filteredItems.length} records</CardDescription>  {/* live count */}
      </div>
      <div className="flex flex-col gap-2 sm:flex-row sm:items-center">
        {/* Search input + filter pills */}
      </div>
    </div>
  </CardHeader>
  <CardContent>
    {/* Table or empty state */}
  </CardContent>
</Card>
```

The `CardDescription` shows a **live filtered count** — always update it as filters change, never show the total static count.

## TableCard Wrapper Pattern (Management Pages)

For management pages implemented in Material UI, use `TableCard` as the listing shell and split it into: (1) section header, (2) tabs, (3) content.

```tsx
<TableCard sx={{ p: { xs: 1.5, md: 2 } }}>
  <Box
    sx={{
      py: { xs: 1.5, md: 2 },
      display: "flex",
      alignItems: { xs: "flex-start", sm: "center" },
      justifyContent: "space-between",
      flexDirection: { xs: "column", sm: "row" },
      gap: 1.5,
      borderBottom: "1px solid",
      borderColor: "divider",
    }}
  >
    <Box>
      <Typography variant="subtitle2" sx={{ fontWeight: 700 }}>
        Task Records
      </Typography>
      <Typography variant="body2" color="text.secondary">
        {visibleCountLabel}
      </Typography>
    </Box>
    <SearchField />
  </Box>

  <Tabs value={selectedTab} onChange={handleTabChange}>
    <Tab label="Active tasks" value="active" />
    <Tab label="Recycling Bin" value="recycleBin" />
  </Tabs>

  {/* table or empty state for selected tab */}
</TableCard>
```

Use this pattern when a single entity type needs grouped lifecycle views (active, archived, deleted) in one page.

## Responsive Table

Wrap the table in `overflow-x-auto`. Use progressive column hiding to adapt to screen width:

- Always visible: name/title column + action column
- `hidden sm:table-cell`: secondary identifier (email, type)
- `hidden md:table-cell`: tertiary detail (phone, breed)
- `hidden lg:table-cell`: supplemental (weight, timestamps)

```tsx
<div className="overflow-x-auto">
  <Table>
    <TableHeader>
      <TableRow>
        <TableHead>Name</TableHead>
        <TableHead className="hidden sm:table-cell">Email</TableHead>
        <TableHead className="hidden md:table-cell">Phone</TableHead>
        <TableHead className="w-10"></TableHead>  {/* actions — no label */}
      </TableRow>
    </TableHeader>
    <TableBody>
      {filteredItems.map((item) => (
        <TableRow key={item.id}>
          <TableCell>
            <Link href={`/entity/${item.id}`} className="font-medium hover:text-primary transition-colors">
              {item.name}
            </Link>
          </TableCell>
          <TableCell className="hidden sm:table-cell">{item.email}</TableCell>
          <TableCell className="hidden md:table-cell">{item.phone}</TableCell>
          <TableCell>
            {/* MoreHorizontal dropdown */}
          </TableCell>
        </TableRow>
      ))}
    </TableBody>
  </Table>
</div>
```

## Card-List Layout

For rich records (title, metadata, short description, type icon), prefer card-list rows over a dense table. This follows `frontend-dashboard/table-sample.tsx`.

```tsx
<div className="space-y-4">
  {filteredItems.map((item) => {
    const Icon = entityTypeIcons[item.type] || FileText
    return (
      <div key={item.id} className="flex gap-4 rounded-lg border p-4">
        <div className="flex size-12 shrink-0 items-center justify-center rounded-full bg-primary/10 text-primary">
          <Icon className="size-6" />
        </div>

        <div className="flex-1 min-w-0">
          <div className="flex flex-col gap-2 sm:flex-row sm:items-start sm:justify-between">
            <div>
              <h3 className="font-semibold">{item.title}</h3>
              <div className="flex flex-wrap items-center gap-2 text-sm text-muted-foreground">
                <Link
                  href={`/entity/${item.relatedId}`}
                  className="font-medium text-foreground hover:text-primary transition-colors"
                >
                  {item.relatedName}
                </Link>
                <span>-</span>
                <span>{item.secondaryField}</span>
                <span>-</span>
                <span>{item.formattedDate}</span>
              </div>
            </div>
            <Badge variant="outline">{item.typeLabel}</Badge>
          </div>

          <p className="mt-2 text-sm text-muted-foreground line-clamp-2">{item.description}</p>
        </div>
      </div>
    )
  })}
</div>
```

## Type Icon + Label Maps

Define stable maps for icon rendering and human-readable labels:

```tsx
const entityTypeIcons: Record<string, React.ComponentType<{ className?: string }>> = {
  typeA: IconA,
  typeB: IconB,
  typeC: IconC,
}

const entityTypeLabels: Record<string, string> = {
  typeA: 'Type A',
  typeB: 'Type B',
  typeC: 'Type C',
}
```

Always use a fallback icon (`|| FileText`) so unknown backend values do not break rendering.

## Clean Row UI/UX Blueprint

- Row container: `rounded-lg border p-4` for balanced density and readable grouping.
- Vertical rhythm: `space-y-4` between rows, `mt-2` between metadata and description.
- Type icon container: `size-12`; icon inside `size-6`; keep `bg-primary/10 text-primary`.
- Content container must include `min-w-0` to prevent overflow in flexible layouts.
- Metadata row uses `text-sm text-muted-foreground` and `gap-2` for easy scanning.
- Metadata separators are inline dashes (`<span>-</span>`) for compact, familiar reading.
- Primary related entity text is clickable with `font-medium` and `hover:text-primary`.
- Description is always clamped (`line-clamp-2`) to keep row height predictable.

## Name Cell with Avatar

The first column (name/title) should be clickable and include a small avatar or icon:

```tsx
<Link href={`/entity/${item.id}`} className="flex items-center gap-2 font-medium hover:text-primary transition-colors">
  {item.avatarUrl ? (
    <div className="size-8 rounded-full overflow-hidden shrink-0">
      <Image src={item.avatarUrl} alt={item.name} width={32} height={32} className="size-full object-cover" />
    </div>
  ) : (
    <div className="flex size-8 items-center justify-center rounded-full bg-primary/10 shrink-0">
      <EntityIcon className="size-4 text-primary" />
    </div>
  )}
  {item.name}
</Link>
```

## State Management

Keep all filter/search state local with `useState`. Filter the array in a derived variable, never mutate the source:

```tsx
const [searchQuery, setSearchQuery] = useState('')
const [categoryFilter, setCategoryFilter] = useState<FilterType>('all')

const filteredItems = items.filter((item) => {
  const matchesSearch = item.name.toLowerCase().includes(searchQuery.toLowerCase())
  const matchesCategory = categoryFilter === 'all' || item.category === categoryFilter
  return matchesSearch && matchesCategory
})
```

## Loading State

Show a centered, pulsing message while data loads. Never show the table skeleton or a spinner bar — a simple text pulse is sufficient for in-memory or fast data:

```tsx
if (isLoading) {
  return (
    <div className="flex items-center justify-center p-8">
      <div className="animate-pulse text-muted-foreground">Loading records...</div>
    </div>
  )
}
```

## Rules

- The action column header must be empty (no `<TableHead>` text) with `className="w-10"`.
- Every table row's name cell must be a navigable link to the detail page.
- Never paginate unless the dataset exceeds 500 records — use client-side filter instead.
- Badge variants for status: `secondary` for neutral, `default` for positive/confirmed, `destructive` for failed/cancelled, `outline` for in-progress/pending.
- Sort data before rendering (newest first, or by date/name) — never show unsorted raw order.
- In card-list layouts, row descriptions are always `line-clamp-2`.
- In card-list layouts, use `size-12` icon circles with `size-6` icons and `bg-primary/10 text-primary`.
- In card-list layouts, preserve the top-right context cluster (type badge + actions menu if present).
