---
name: frontend-page-chrome
description: Golden guideline for the full page chrome — sidebar, sticky header, content area padding, background color layering, and vertical spacing rhythm. Use when building any new page, screen, or route in a CRM or admin dashboard application so that every page feels visually consistent with the rest of the app.
---

# Frontend Page Chrome Design Guideline

The "page chrome" is the shell that wraps every page: sidebar + header + content area. Its background colors, padding, and spacing tokens create visual depth and consistency across the entire app.

## App Shell Structure

The outermost layout uses shadcn's `SidebarProvider` + `SidebarInset`:

```tsx
<SidebarProvider>
  <AppSidebar />            {/* dark sidebar */}
  <SidebarInset>
    <AppHeader breadcrumbs={breadcrumbs} />   {/* sticky header */}
    <main className="flex-1 overflow-auto">
      {children}             {/* page content */}
    </main>
  </SidebarInset>
</SidebarProvider>
```

Never render page content directly inside `SidebarProvider` — always go through `SidebarInset` so the layout handles the inset offset automatically.

## Visual Depth Layering

The app uses a three-layer color system to create visual depth without shadows:

| Layer | Background | Usage |
|---|---|---|
| Sidebar | `bg-sidebar` (dark, ~oklch 0.22) | Navigation rail |
| Page content area | `bg-background` (near-white / dark-mode dark) | The main scrollable canvas |
| Cards | `bg-card` (pure white / slightly lighter dark) | Content blocks inside pages |

Cards sit visually "above" the background because `--card` is slightly lighter than `--background` in both light and dark mode. This means you never need a box-shadow to lift a card — the background color contrast does the work.

```
Page bg (--background) → Card bg (--card) → Bordered rows inside card
```

Never place a Card directly on the sidebar — only on `bg-background`.

## Sticky Header

The app header is a thin, sticky bar that always sits at the top of the content area:

```tsx
<header className="sticky top-0 z-40 flex h-14 items-center gap-4 border-b
  bg-background md:bg-background                  {/* light bg on md+ */}
  bg-sidebar text-sidebar-foreground              {/* sidebar-colored on mobile (acts as nav bar) */}
  md:text-foreground
  border-sidebar-border md:border-border
  px-4 md:px-6">
```

Key values:
- Height: `h-14` (56px) — fixed; never varies
- z-index: `z-40` — stays above page content when scrolled
- Left padding: `px-4` on mobile, `px-6` on desktop
- On **mobile** it takes the sidebar's dark background (acting as a top nav bar)
- On **desktop** it uses `bg-background` and blends with the content area

The header contains only breadcrumb navigation — no search bars, filters, or action buttons. Those live inside the page content.

## Breadcrumb Pattern

Breadcrumbs use the app name as the root link, then chevron-separated page labels:

```tsx
<Breadcrumb>
  <BreadcrumbList>
    <BreadcrumbLink href="/" className="font-semibold text-sidebar-primary md:text-primary">
      AppName
    </BreadcrumbLink>
    <BreadcrumbSeparator />
    <BreadcrumbPage>Current Page</BreadcrumbPage>
  </BreadcrumbList>
</Breadcrumb>
```

- Root app link: `font-semibold` + `text-primary` color — makes it act as a subtle brand anchor
- Current page: `BreadcrumbPage` (non-interactive)
- Intermediate links (deep pages): `BreadcrumbLink` with `href`

Pass breadcrumbs as an array into the layout component:
```tsx
<AppLayout breadcrumbs={[{ label: 'Owners', href: '/owners' }, { label: owner.name }]}>
```

## Content Wrapper

Every page content component wraps its content in this exact div:

```tsx
<div className="p-4 md:p-6 space-y-6">
  {/* page header */}
  {/* cards / sections */}
  {/* dialogs (rendered but invisible until opened) */}
</div>
```

- `p-4` on mobile, `p-6` on desktop — consistent inset from all edges
- `space-y-6` — 24px vertical gap between every top-level section
- Never use `mt-` or `mb-` inside this wrapper; `space-y-6` handles all vertical rhythm

On dashboard pages where density is higher:
```tsx
<div className="p-3 md:p-6 space-y-4 md:space-y-6">
```

Use `p-3` / `space-y-4` on mobile for tighter density; `p-6` / `space-y-6` on desktop.

## Page Header Section

The first child inside the content wrapper is always the page header — title + subtitle on the left, primary CTA button on the right:

```tsx
<div className="flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
  <div>
    <h1 className="text-2xl font-bold tracking-tight">Page Title</h1>
    <p className="text-muted-foreground">Short description of what this section manages</p>
  </div>
  <Button onClick={handleOpenCreate}>
    <Plus className="size-4 mr-2" />
    Add Record
  </Button>
</div>
```

Typography:
- H1: `text-2xl font-bold tracking-tight` — never `text-3xl`, never omit `tracking-tight`
- Subtitle: `text-muted-foreground` — 4 to 8 words, describes the scope of the section
- CTA: default filled `Button` with a `size-4` Lucide icon and `mr-2` margin

On mobile, the header stacks (`flex-col gap-4`). On `sm+`, it goes horizontal with `justify-between`.

Pages without a CTA (e.g., Settings overview, detail read-only pages) still have the title/subtitle pair — just omit the button. Never use `justify-between` if there is no right-side element.

## Card Structure Inside Pages

All content lives inside shadcn `Card` components. Cards segment the page into logical blocks:

```tsx
<Card>
  <CardHeader>
    <CardTitle className="flex items-center gap-2">
      <Icon className="size-5" />     {/* always include an icon */}
      Section Title
    </CardTitle>
    <CardDescription>Contextual count or short description</CardDescription>
  </CardHeader>
  <CardContent>
    {/* table, list, form fields, stat rows... */}
  </CardContent>
</Card>
```

Rules:
- `CardTitle` always includes a leading `size-5` Lucide icon in the same `flex items-center gap-2` row
- `CardDescription` is optional but use it when a live count, date range, or brief scope note adds value
- `CardHeader` gets `pb-4` when its content area starts directly with a table (no visual gap needed)
- `CardHeader` gets `pb-3` for activity feed cards to keep the list items closer to the title
- Never pad `CardContent` with `p-0` unless the card contains a settings list that must bleed to the edges

## Sidebar Design

The sidebar uses its own dedicated color token family (`--sidebar-*`) that is independent of the main theme:

```tsx
<Sidebar>
  <SidebarHeader className="border-b border-sidebar-border px-4 py-4">
    {/* App logo + name */}
  </SidebarHeader>
  <SidebarContent>
    {/* Nav items */}
  </SidebarContent>
  <SidebarFooter className="border-t border-sidebar-border">
    {/* Settings + secondary nav */}
  </SidebarFooter>
</Sidebar>
```

- Use `border-sidebar-border` (not `border-border`) for all sidebar internal dividers
- App logo area: icon in `bg-primary text-primary-foreground` rounded-lg `size-9`, app name in `text-lg font-semibold`
- Active nav items use `isActive` prop on `SidebarMenuButton` — never add active styles manually
- Settings and secondary items go in `SidebarFooter`, separated by `border-t`

## Spacing Token Reference

| Token | Value | Use |
|---|---|---|
| `p-4 md:p-6` | 16px / 24px | Content area inset |
| `space-y-6` | 24px | Between top-level sections |
| `gap-4` | 16px | Between cards in a grid |
| `gap-6` | 24px | Between cards in detail page grid |
| `gap-2` | 8px | Between inline elements (icon + text) |
| `gap-3` | 12px | Icon circle + content in contact rows |
| `space-y-4` | 16px | Between rows inside a card list |
| `space-y-2` | 8px | Label + input inside a form field |
| `pb-4` | 16px | CardHeader bottom padding before table |
| `pb-3` | 12px | CardHeader bottom padding before feed list |
| `py-8` | 32px | Empty state vertical padding |
| `py-12` | 48px | Large empty state (listing page) |

## Rules

- Never apply `bg-background` or `bg-card` manually inside page content — those are set by the shell and Cards automatically
- Never use `shadow-*` to distinguish UI layers — background color difference provides the depth
- Never put filter controls, search inputs, or action buttons in the sticky header — they belong inside the page content Cards
- The sidebar is always dark regardless of light/dark mode — `--sidebar` is darker than `--background` in both modes
- `h-14` is the canonical header height — never make it taller or shorter
- Every page must have exactly one `<h1>` — the page title inside the content wrapper's header section
- `tracking-tight` on `h1` headings is mandatory — it tightens letter spacing for large bold text
```

---
