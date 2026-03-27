---
name: frontend-dashboard
description: Golden guideline for designing dashboard pages. Includes stat-card overview dashboards and Material UI management dashboards with grouped content, tabs, and tables. Use when building a home page, overview, summary page, or data-management interface in a CRM, admin panel, or operations dashboard.
---

# Frontend Dashboard Design Guideline

The dashboard communicates system state and provides fast paths to common actions. This skill covers two primary patterns: stat-card overviews and management dashboards.

## Pattern 1: Stat-Card Overview Dashboard

The stat-card dashboard is the first page users see in metrics-heavy applications. It immediately communicates the state of the system with visual stat cards, activity feeds, and quick actions.

### Mobile-First Layout Strategy

The dashboard renders two entirely separate layouts — one for mobile, one for desktop. Both are built in the same component, using `md:hidden` and `hidden md:block` to toggle them.

```tsx
<div className="p-3 md:p-6 space-y-4 md:space-y-6">
  {/* Mobile stat grid */}
  <div className="grid grid-cols-4 gap-2 md:hidden">{...}</div>

  {/* Mobile quick action buttons */}
  <div className="flex gap-2 md:hidden">{...}</div>

  {/* Desktop header with actions */}
  <div className="hidden md:block">{...}</div>

  {/* Desktop stat cards */}
  <div className="hidden md:grid gap-4 sm:grid-cols-2 lg:grid-cols-4">{...}</div>

  {/* Activity cards — shared across both */}
  <div className="grid gap-4 md:gap-6 lg:grid-cols-2">{...}</div>
</div>
```

### Mobile Stat Grid (4-column icon tiles)

On mobile, stats are compact icon tiles in a 4-column grid. Each tile shows icon, label, and a bold number:

```tsx
<div className="grid grid-cols-4 gap-2 md:hidden">
  {stats.map((stat) => (
    <Link key={stat.title} href={stat.href}>
      <div className="flex flex-col items-center gap-1.5 p-3 rounded-xl bg-card border transition-all active:scale-95">
        <div className={`flex size-10 items-center justify-center rounded-full ${stat.bg}`}>
          <stat.icon className={`size-5 ${stat.color}`} />
        </div>
        <span className="text-[11px] font-medium text-muted-foreground">{stat.title}</span>
        <span className={`text-xl font-bold ${stat.color}`}>{stat.value}</span>
      </div>
    </Link>
  ))}
</div>
```

The `active:scale-95` provides a tactile press effect on touch devices.

### Mobile Quick Actions (full-width buttons)

Two buttons below the stat grid for the most important primary actions:

```tsx
<div className="flex gap-2 md:hidden">
  <Button asChild className="flex-1 h-12">
    <Link href="/action">
      <Plus className="size-4 mr-2" />
      Primary Action
    </Link>
  </Button>
  <Button asChild variant="secondary" className="flex-1 h-12">
    <Link href="/secondary">
      <Search className="size-4 mr-2" />
      Secondary Action
    </Link>
  </Button>
</div>
```

Both buttons are `flex-1` (equal width) and `h-12` (tall enough for thumb tap).

### Desktop Page Header

The desktop header has a welcome text left, and action buttons right:

```tsx
<div className="hidden md:block">
  <div className="flex items-center justify-between">
    <div>
      <h1 className="text-2xl font-bold tracking-tight text-balance">Dashboard</h1>
      <p className="text-muted-foreground">Welcome back! Here's your overview.</p>
    </div>
    <div className="flex gap-2">
      <Button asChild>
        <Link href="/primary-action"><Plus className="size-4 mr-2" />Primary</Link>
      </Button>
      <Button asChild variant="outline" className="bg-transparent">
        <Link href="/secondary-action"><Search className="size-4 mr-2" />Secondary</Link>
      </Button>
    </div>
  </div>
</div>
```

### Desktop Stat Cards (hover-interactive)

Each stat card is fully clickable and features a hover animation on its icon:

```tsx
<Link href={stat.href}>
  <Card className="transition-all hover:shadow-md cursor-pointer group overflow-hidden h-full">
    <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
      <CardTitle className="text-sm font-medium text-muted-foreground">{stat.title}</CardTitle>
      <div className={`flex size-8 items-center justify-center rounded-lg ${stat.bg} transition-transform group-hover:scale-110`}>
        <stat.icon className={`size-4 ${stat.color}`} />
      </div>
    </CardHeader>
    <CardContent className="space-y-2">
      <div className={`text-3xl font-bold ${stat.color}`}>{stat.value}</div>
      <div className="flex items-center text-xs text-muted-foreground group-hover:text-foreground transition-colors">
        <span>{stat.action}</span>
        <ChevronRight className="size-3 ml-1 transition-transform group-hover:translate-x-0.5" />
      </div>
    </CardContent>
  </Card>
</Link>
```

The `group` class enables `group-hover:` effects on children. The icon scales up, the action text darkens, and the chevron nudges right — all on hover.

### Stat Color System

Define stats with semantic color classes. Use brand-specific colors for variety, not arbitrary ones:

```tsx
const stats = [
  { title: 'Records', color: 'text-primary', bg: 'bg-primary/10', ... },
  { title: 'Clients', color: 'text-blue-600 dark:text-blue-400', bg: 'bg-blue-500/10', ... },
  { title: 'Today', color: 'text-amber-600 dark:text-amber-400', bg: 'bg-amber-500/10', ... },
  { title: 'Pending', color: 'text-rose-600 dark:text-rose-400', bg: 'bg-rose-500/10', ... },
]
```

Always pair a dark-mode variant: `text-color-600 dark:text-color-400`. Background uses `/10` opacity.

### Activity Feed Cards

The bottom section uses two side-by-side cards (stacked on mobile):

```tsx
<div className="grid gap-4 md:gap-6 lg:grid-cols-2">
  <UpcomingEventsCard />
  <RecentRecordsCard />
</div>
```

Each card has a "View All" link in the header:

```tsx
<CardHeader className="pb-3">
  <div className="flex items-center justify-between">
    <CardTitle className="flex items-center gap-2 text-base md:text-lg">
      <Icon className="size-4 md:size-5 text-primary" />
      Section Title
    </CardTitle>
    <Button asChild variant="ghost" size="sm" className="text-xs">
      <Link href="/section">View All</Link>
    </Button>
  </div>
</CardHeader>
```

### Color-Coded List Items

Use a left-border accent stripe to color-code items by type without overwhelming the card:

```tsx
<Link className="flex items-center gap-3 rounded-lg border p-2.5 relative overflow-hidden
  hover:shadow-sm hover:border-primary/20 transition-all active:bg-muted">
  <div className={`absolute left-0 top-0 bottom-0 w-1 ${typeColor}`} />  {/* color stripe */}
  <div className="pl-1">
    {/* content */}
  </div>
</Link>
```

### Stat Dashboard Rules

- Always show exactly 4 stat cards — not 3, not 5. This fills the 4-column grid perfectly.
- Stat values must be real computed numbers from live data — never hardcoded.
- Activity feed lists cap at 4 items — use "View All" for the rest.
- Desktop CTA buttons in the header: primary action uses default style, secondary uses `variant="outline" className="bg-transparent"`.
- Never use `text-balance` except on the H1 heading.
- The welcome subtitle ("Welcome back! Here's your overview.") humanizes the interface — always include it.

---

## Pattern 2: Management Dashboard (Material UI)

Management dashboards present grouped data sets with filtering, search, and tabbed views for lifecycle states. This pattern uses Material UI components and is appropriate for admin/CRUD interfaces.

### Framework Baseline

Use Material UI primitives: `Box`, `Typography`, `Button`, `Tabs`, `Tab`, `TextField`, `Dialog`, `InputAdornment`.

### Page Composition

```tsx
<PageContainer>
  {/* 1. Page header: title + contextual subtitle + primary CTA */}
  {/* 2. TableCard: section header + search + tabs + list/table content */}
  {/* 3. Dialogs: create/edit/manage/confirm actions */}
</PageContainer>
```

### Page Header (Title + Contextual Subtitle + Primary Action)

Keep title/subtitle on the left and one primary action button on the right:

```tsx
<Box sx={{ display: "flex", alignItems: "center", justifyContent: "space-between", gap: 2 }}>
  <Box>
    <Typography variant="h5" component="h1" sx={{ fontWeight: 700 }}>
      Task Management
    </Typography>
    <Typography variant="body2" color="text.secondary">
      Manage task definitions, assignments, versions, and lifecycle status.
    </Typography>
  </Box>
  <Button variant="contained" color="primary" startIcon={<AddIcon />}>
    New Task
  </Button>
</Box>
```

**Subtitle guideline**: Must be contextual to the page domain (e.g., "Manage task definitions..."), not a fixed greeting.

### Grouped Listing Container (TableCard)

Group all list content into one `TableCard` with clear internal sections:

```tsx
<TableCard sx={{ p: { xs: 1.5, md: 2 } }}>
  {/* section header: entity title + visible count + search */}
  {/* tabs: grouped data views */}
  {/* table/list content area */}
</TableCard>
```

### Tabs for Grouped Record Sets

Use tabs when records share one schema but differ by lifecycle or state:

```tsx
<Tabs value={selectedTab} onChange={handleTabChange}>
  <Tab label="Active tasks" value="active" />
  <Tab label="Recycling Bin" value="recycleBin" />
</Tabs>
```

Recommended behavior:
- Clear search input on tab switch when each tab has separate datasets.
- Show per-tab counts and per-tab empty states.
- Keep tabs inside the listing container, immediately above content.

### Search Input (Material UI)

Use `TextField` with `InputAdornment` for the search icon:

```tsx
<TextField
  size="small"
  placeholder="Search tasks..."
  value={searchDraft}
  onChange={(e) => setSearchDraft(e.target.value)}
  InputProps={{
    startAdornment: (
      <InputAdornment position="start">
        <SearchIcon fontSize="small" sx={{ color: "text.secondary" }} />
      </InputAdornment>
    ),
  }}
  sx={{ minWidth: { xs: "100%", sm: 240 } }}
/>
```

### Action Buttons (Material UI)

- Primary create action: `variant="contained" color="primary"`
- Secondary actions: `variant="outlined"` or text variant
- Use `startIcon` for button icons instead of inline classes

### Management Dashboard Rules

- This pattern is listing-first; stat-card-only layouts belong in Pattern 1.
- Subtitles must be contextual to the page domain, not fixed greeting strings.
- Keep grouped data states in tabs within one container instead of separate page sections.
- Prefer one clear primary CTA in the page header.

---

## Selection Guide

**Use Pattern 1 (Stat-Card Overview)** when:
- Building a dashboard that emphasizes metrics and KPIs
- Users need a high-level overview before diving into details
- You want hero stat cards with activity feeds

**Use Pattern 2 (Management Dashboard)** when:
- Building a CRUD/admin interface for managing records
- Data has lifecycle states (active, archived, deleted) that benefit from tabs
- Users primarily search and filter rather than scan metrics
- Using Material UI as your component library
```
