---
name: frontend-row-actions
description: Golden guideline for designing row-level actions in data tables — dropdown menus, status transitions, and destructive delete confirmation dialogs. Use when any table row needs edit, delete, status change, or other contextual actions.
---

# Frontend Row Actions Design Guideline

Row actions are exposed via a `MoreHorizontal` icon button that triggers a `DropdownMenu`. Destructive actions (Delete) always require a confirmation `AlertDialog`.

## The Actions Column

The last column of every table is the actions column. It has no header label and a fixed narrow width:

```tsx
<TableHead className="w-10"></TableHead>

<TableCell>
  <DropdownMenu>
    <DropdownMenuTrigger asChild>
      <Button variant="ghost" size="icon" className="size-8">
        <MoreHorizontal className="size-4" />
        <span className="sr-only">Actions</span>
      </Button>
    </DropdownMenuTrigger>
    <DropdownMenuContent align="end">
      {/* Action items */}
    </DropdownMenuContent>
  </DropdownMenu>
</TableCell>
```

Always set `align="end"` on `DropdownMenuContent` so it doesn't overflow the right edge of the screen.

## Card-List Row Actions

When a listing uses card-style rows (not a `<Table>`), keep row actions in the top-right cluster beside the type badge. This is the `frontend-dashboard/table-sample.tsx` pattern and should be treated as canonical for content-heavy rows.

```tsx
<div className="flex gap-4 rounded-lg border p-4">
  <div className="flex size-12 shrink-0 items-center justify-center rounded-full bg-primary/10 text-primary">
    <EntityIcon className="size-6" />
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

      <div className="flex items-center gap-2 shrink-0">
        <Badge variant="outline">{item.typeLabel}</Badge>
        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="ghost" size="icon" className="size-8">
              <MoreHorizontal className="size-4" />
              <span className="sr-only">Actions</span>
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end">
            {/* Action items */}
          </DropdownMenuContent>
        </DropdownMenu>
      </div>
    </div>

    <p className="mt-2 text-sm text-muted-foreground line-clamp-2">{item.description}</p>
  </div>
</div>
```

This cluster makes the row easy to scan: semantic type context (badge) first, contextual actions (menu) second.

## Standard Action Items

The standard set of actions for a record: View, Edit, Delete. Order matters — View first, Edit second, Delete last (and separated).

```tsx
<DropdownMenuItem asChild>
  <Link href={`/entity/${item.id}`}>
    <Eye className="size-4 mr-2" />
    View Details
  </Link>
</DropdownMenuItem>

<DropdownMenuItem onClick={() => handleEdit(item)}>
  <Edit className="size-4 mr-2" />
  Edit
</DropdownMenuItem>

<DropdownMenuItem
  onClick={() => handleDelete(item)}
  className="text-destructive"
>
  <Trash2 className="size-4 mr-2" />
  Delete
</DropdownMenuItem>
```

Icon size inside dropdown items: always `size-4 mr-2`. Destructive items: `className="text-destructive"`.

## Status Transition Actions

When records have a lifecycle (e.g., Scheduled → Confirmed → In Progress → Completed), expose status transitions as dropdown items. Group them with a separator, and disable transitions that don't apply to the current state.

```tsx
<DropdownMenuSeparator />

<DropdownMenuItem
  onClick={() => handleStatusChange(item.id, 'confirmed')}
  disabled={item.status === 'confirmed'}
>
  <Check className="size-4 mr-2" />
  Confirm
</DropdownMenuItem>

<DropdownMenuItem
  onClick={() => handleStatusChange(item.id, 'in-progress')}
  disabled={item.status === 'in-progress'}
>
  <Play className="size-4 mr-2" />
  Start
</DropdownMenuItem>

<DropdownMenuItem
  onClick={() => handleStatusChange(item.id, 'completed')}
  disabled={item.status === 'completed'}
>
  <Clock className="size-4 mr-2" />
  Complete
</DropdownMenuItem>

<DropdownMenuSeparator />

<DropdownMenuItem onClick={() => setDeletingItem(item)} className="text-destructive">
  <Trash2 className="size-4 mr-2" />
  Delete
</DropdownMenuItem>
```

## Delete Confirmation Dialog

Never delete inline. Always use `AlertDialog` to confirm. The `deletingItem` state drives both the dialog's open state and the confirmation message.

```tsx
const [deletingItem, setDeletingItem] = useState<Entity | null>(null)

const confirmDelete = () => {
  if (deletingItem) {
    deleteEntity(deletingItem.id)
    setDeletingItem(null)
  }
}

<AlertDialog open={!!deletingItem} onOpenChange={() => setDeletingItem(null)}>
  <AlertDialogContent>
    <AlertDialogHeader>
      <AlertDialogTitle>Delete {entityName}</AlertDialogTitle>
      <AlertDialogDescription>
        Are you sure you want to delete {deletingItem?.name}? This action cannot be undone.
      </AlertDialogDescription>
    </AlertDialogHeader>
    <AlertDialogFooter>
      <AlertDialogCancel>Cancel</AlertDialogCancel>
      <AlertDialogAction
        onClick={confirmDelete}
        className="bg-destructive text-destructive-foreground hover:bg-destructive/90"
      >
        Delete
      </AlertDialogAction>
    </AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>
```

The `AlertDialogAction` must explicitly set destructive colors — it does not inherit them automatically.

## Inline Status Handler

Status changes should be immediate (optimistic), not require a form:

```tsx
const handleStatusChange = (id: string, status: Entity['status']) => {
  updateEntity(id, { status })
}
```

## Rules

- Always include `<span className="sr-only">Actions</span>` inside the trigger button for accessibility.
- The MoreHorizontal button is `size-8` (32px), `variant="ghost"`, `size="icon"`.
- Never put more than 7 items in a dropdown — split into sub-menus if needed.
- Delete is always the last item, always separated by a `DropdownMenuSeparator`.
- Disabled items remain visible but non-interactive — never hide them conditionally.
- Status transitions are in the middle of the dropdown, between CRUD actions and Delete.
- The confirmation dialog message always names the specific record being deleted.

## Table vs Card-List: Pattern Selection

- Use a table when records are compact and column-comparable.
- Use card-list rows when records include title + metadata + description and need better readability.
- In card-list rows, keep `Badge` + `MoreHorizontal` in one top-right `flex items-center gap-2` cluster.
- In both patterns, actions are always menu-based and destructive actions always require `AlertDialog`.

