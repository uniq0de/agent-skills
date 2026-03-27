---
name: frontend-detail-page
description: Golden guideline for designing entity detail/profile pages in CRM applications. Use when building any view that shows a single record's full details, related data, and edit capability — such as a pet profile, client details, or order view.
---

# Frontend Detail Page Design Guideline

Detail pages show everything about a single record. They follow a consistent 3-column-grid layout with a back-navigation header, a 2/3 main area, and a 1/3 sidebar for related metadata.

## Page Header

Every detail page starts with a row containing: back button (left), entity identity (center-left), primary action (right).

```tsx
<div className="flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
  <div className="flex items-center gap-4">
    <Button variant="ghost" size="icon" asChild>
      <Link href="/entity-list">
        <ArrowLeft className="size-4" />
      </Link>
    </Button>
    <div className="flex items-center gap-3">
      <div className="flex size-12 items-center justify-center rounded-full bg-primary/10 text-primary">
        <EntityIcon className="size-6" />
      </div>
      <div>
        <h1 className="text-2xl font-bold">{entity.name}</h1>
        <p className="text-muted-foreground">{entity.subtitle}</p>
      </div>
    </div>
  </div>
  <Button onClick={() => setEditDialogOpen(true)}>
    <Edit className="size-4 mr-2" />
    Edit {EntityName}
  </Button>
</div>
```

The avatar circle for the entity is `size-12` (48px) with `bg-primary/10` background and `text-primary` icon color. This is larger than the `size-8` used in table rows.

## Two-Thirds / One-Third Grid

```tsx
<div className="grid gap-6 lg:grid-cols-3">
  {/* Main content: 2/3 width */}
  <div className="lg:col-span-2 space-y-6">
    <PrimaryInfoCard />
    <RelatedDataCard />
  </div>

  {/* Sidebar: 1/3 width */}
  <div className="space-y-6">
    <ContactCard />
    <QuickStatsCard />
    <RelatedLinksCard />
  </div>
</div>
```

On mobile (`< lg`), both columns stack vertically. The main content appears first, sidebar second.

## Primary Info Card

Shows the entity's own fields in a 2-column label/value grid:

```tsx
<Card>
  <CardHeader>
    <CardTitle className="flex items-center gap-2">
      <EntityIcon className="size-5" />
      Entity Information
    </CardTitle>
  </CardHeader>
  <CardContent>
    <div className="grid gap-4 sm:grid-cols-2">
      <div>
        <p className="text-sm text-muted-foreground">Field Label</p>
        <p className="font-medium">Field Value</p>
      </div>
      {/* More field pairs... */}
    </div>
    {entity.notes && (
      <>
        <Separator className="my-4" />
        <div>
          <p className="text-sm text-muted-foreground mb-1">Notes</p>
          <p className="text-sm">{entity.notes}</p>
        </div>
      </>
    )}
  </CardContent>
</Card>
```

Label/value pattern: label is `text-sm text-muted-foreground`, value is `font-medium`. Never use a `<dl>/<dt>/<dd>` structure — the simple `<div>/<p>/<p>` pattern is more consistent with shadcn's Card system.

## Related Data Card (Timeline/List)

Shows related records (medical history, appointments, etc.) as a list of bordered rows with icon + content:

```tsx
<Card>
  <CardHeader>
    <CardTitle className="flex items-center gap-2">
      <RelatedIcon className="size-5" />
      Related Records
    </CardTitle>
    <CardDescription>{records.length} records on file</CardDescription>
  </CardHeader>
  <CardContent>
    <div className="space-y-4">
      {records.slice(0, 5).map((record) => (
        <div key={record.id} className="flex gap-4 rounded-lg border p-4">
          <div className="flex size-10 shrink-0 items-center justify-center rounded-full bg-muted">
            <RecordIcon className="size-5" />
          </div>
          <div className="flex-1">
            <div className="flex items-start justify-between gap-2">
              <div>
                <p className="font-medium">{record.title}</p>
                <p className="text-sm text-muted-foreground">{record.meta}</p>
              </div>
              <Badge variant="outline">{record.type}</Badge>
            </div>
            <p className="mt-2 text-sm text-muted-foreground">{record.description}</p>
          </div>
        </div>
      ))}
      {records.length > 5 && (
        <Button variant="outline" className="w-full bg-transparent" asChild>
          <Link href={`/related?entityId=${entity.id}`}>
            View All Records ({records.length})
          </Link>
        </Button>
      )}
    </div>
  </CardContent>
</Card>
```

Always cap at 5 items and show a "View All" link to the filtered listing page.

## Sidebar Contact Card

For entities with contact details, use icon + label + value rows:

```tsx
<Card>
  <CardHeader>
    <CardTitle className="flex items-center gap-2">
      <User className="size-5" />
      Contact Information
    </CardTitle>
  </CardHeader>
  <CardContent className="space-y-4">
    <div className="flex items-center gap-3">
      <div className="flex size-9 items-center justify-center rounded-full bg-muted">
        <Phone className="size-4" />
      </div>
      <div>
        <p className="text-sm text-muted-foreground">Phone</p>
        <p className="font-medium">{entity.phone}</p>
      </div>
    </div>
    {/* Email, Address rows... */}
  </CardContent>
</Card>
```

The icon container is `size-9` (36px), `rounded-full`, `bg-muted` — smaller than the entity avatar.

## Quick Stats Card

A compact summary of counts related to this entity:

```tsx
<Card>
  <CardHeader><CardTitle>Quick Stats</CardTitle></CardHeader>
  <CardContent className="space-y-4">
    <div className="flex justify-between">
      <span className="text-muted-foreground">Total Pets</span>
      <span className="font-medium">{count}</span>
    </div>
    <div className="flex justify-between">
      <span className="text-muted-foreground">Completed Visits</span>
      <span className="font-medium">{completedCount}</span>
    </div>
  </CardContent>
</Card>
```

## Not Found State

When the entity ID doesn't resolve, show a centered not-found state with navigation back:

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

## Rules

- The back button is always `variant="ghost" size="icon"` — never a text link.
- Card titles always have an icon: `<Icon className="size-5" />` before the text.
- Limit related data lists to 5 items with a "View All" link — never paginate inline.
- The sidebar column never contains editable forms — only read-only data and quick stats.
- Use `Separator` with `my-4` only to separate optional fields (like Notes) from required fields.
- Always handle the not-found case before rendering the main layout.
```