---
name: frontend-dialog-design
description: Golden guideline for the visual and interaction design of all dialogs and modals — form dialogs, confirmation dialogs, and info dialogs. Use whenever you are building any modal overlay, popup, or dialog in a CRM or admin dashboard. Covers sizing, header copy, backdrop treatment, footer buttons, and when to use Dialog vs AlertDialog.
---

# Frontend Dialog & Modal Design Guideline

All modal overlays in the app come in two types: `Dialog` (for forms and informational content) and `AlertDialog` (for destructive confirmations only). Never use a sheet, drawer, or custom overlay for these purposes.

## Two Dialog Types

| Type | Use for | Component |
|---|---|---|
| `Dialog` | Creating records, editing records, viewing info, settings panels | `Dialog` + `DialogContent` |
| `AlertDialog` | Irreversible destructive actions (delete, clear, revoke) | `AlertDialog` + `AlertDialogContent` |

The choice is not about layout — it is about semantics and keyboard behavior. `AlertDialog` traps focus more aggressively and cannot be closed by clicking the backdrop, which is appropriate when a user must actively choose between "Cancel" and "Confirm Delete."

## Dialog Visual Treatment

```tsx
<DialogContent className="sm:max-w-[500px] max-h-[90vh] overflow-y-auto backdrop-blur-sm">
```

Every Dialog `DialogContent` uses these exact classes:
- `sm:max-w-[500px]` — capped at 500px on sm+ screens; full-width on mobile
- `max-h-[90vh]` — prevents the dialog from exceeding viewport height
- `overflow-y-auto` — allows internal scrolling for long forms
- `backdrop-blur-sm` — applies a subtle background blur, giving the overlay a frosted glass feel that visually separates it from the page without a harsh black overlay

Never override the default shadcn overlay color. The blur handles depth; the overlay provides the dim.

## Dialog Size Guide

| Width | When to use |
|---|---|
| `sm:max-w-[500px]` | Default — all create/edit forms with 1–2 column fields |
| `sm:max-w-[700px]` | Forms with 3+ logical sections or dense side-by-side content |
| `sm:max-w-[400px]` | Simple info-only dialogs with no form fields |

Never go wider than `700px`. If the content requires more space, use a full page instead.

## Dialog Header Copy

`DialogHeader` always contains both `DialogTitle` and `DialogDescription`:

```tsx
<DialogHeader>
  <DialogTitle>{isEditing ? 'Edit Client' : 'Add New Client'}</DialogTitle>
  <DialogDescription>
    {isEditing
      ? 'Update the client information below.'
      : 'Enter the details for the new client.'}
  </DialogDescription>
</DialogHeader>
```

- **Title**: Action verb + entity name — "Edit Client", "Add New Pet", "Schedule Appointment"
- **Description**: One sentence. In edit mode: "Update the [entity] information below." In create mode: "Enter the details for the new [entity]."
- Never leave `DialogDescription` empty — it is read by screen readers and gives context to the title

## Dialog Footer Buttons

The footer always has exactly two buttons: Cancel (outline) on the left, primary action (filled) on the right:

```tsx
<DialogFooter>
  <Button type="button" variant="outline" onClick={() => onOpenChange(false)}>
    Cancel
  </Button>
  <Button type="submit">
    {isEditing ? 'Save Changes' : 'Add Client'}
  </Button>
</DialogFooter>
```

Button label rules:
- Create mode: `"Add [EntityName]"` — e.g., "Add Client", "Add Pet", "Schedule"
- Edit mode: `"Save Changes"` — always, not "Update" or "Edit"
- Cancel: always "Cancel" — never "Close", "Dismiss", or "No"

## AlertDialog for Destructive Confirmation

Use `AlertDialog` for delete and other irreversible actions. Drive it with a single `deletingItem` state:

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
      <AlertDialogTitle>Delete {EntityName}</AlertDialogTitle>
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

Critical: The `AlertDialogAction` must explicitly set `className="bg-destructive text-destructive-foreground hover:bg-destructive/90"`. It does not inherit destructive colors automatically from shadcn.

AlertDialog copy rules:
- Title: "Delete [EntityName]" — always "Delete", not "Remove" or "Are you sure?"
- Body: "Are you sure you want to delete {specific.name}? This action cannot be undone." — always name the specific record so the user knows exactly what they are deleting
- Cancel: "Cancel"
- Confirm: "Delete" — same verb as the title, not "Yes" or "Confirm"

## State Management for Dialogs

Every dialog has two state variables on the page component:

```tsx
// Form dialog (create/edit)
const [dialogOpen, setDialogOpen] = useState(false)
const [editingItem, setEditingItem] = useState<Entity | null>(null)

// Delete confirmation dialog
const [deletingItem, setDeletingItem] = useState<Entity | null>(null)

// Handlers
const handleEdit = (item: Entity) => {
  setEditingItem(item)
  setDialogOpen(true)
}

const handleDialogClose = () => {
  setDialogOpen(false)
  setEditingItem(null)  // always clear editingItem on close
}
```

The `editingItem` must be cleared when the dialog closes — otherwise reopening for "Add New" will briefly flash the old record's data.

## Dialog Placement in JSX

All dialogs are rendered at the **bottom** of the component's JSX, after all visible page content. They are invisible until their `open` state is true:

```tsx
return (
  <div className="p-4 md:p-6 space-y-6">
    {/* page header */}
    {/* cards, tables, lists */}

    {/* dialogs — always last */}
    <EntityFormDialog open={dialogOpen} onOpenChange={handleDialogClose} entity={editingItem} />
    <AlertDialog open={!!deletingItem} onOpenChange={() => setDeletingItem(null)}>
      ...
    </AlertDialog>
  </div>
)
```

Never conditionally render dialogs — always render them and let the `open` prop control visibility. Conditional rendering loses mount/unmount animations and causes layout shifts.

## Long Forms: Scroll Behavior

For forms that may overflow the viewport (many fields, selects, textareas):

```tsx
<DialogContent className="sm:max-w-[500px] max-h-[90vh] overflow-y-auto backdrop-blur-sm">
```

`max-h-[90vh]` + `overflow-y-auto` gives exactly 5% viewport breathing room above and below, and the dialog scrolls internally without the page scrolling underneath it. The `DialogFooter` sticks at the bottom of the dialog's scroll container.

## Rules

- Never use custom modal overlays — always use shadcn `Dialog` or `AlertDialog`
- Never put a form inside an `AlertDialog` — `AlertDialog` is confirmation-only
- Never close a dialog on successful submit before calling the data mutation — submit first, close second
- `backdrop-blur-sm` is mandatory on every `DialogContent`
- The destructive button in `AlertDialog` must manually set `bg-destructive` classes
- Dialog title is always a verb phrase ("Edit X", "Add X", "Delete X") — never a question ("Are you sure?")
- `max-h-[90vh] overflow-y-auto` is required on all `DialogContent` — even short forms, as a future-proofing safeguard
- Never render more than two buttons in a `DialogFooter` (Cancel + primary action)
```