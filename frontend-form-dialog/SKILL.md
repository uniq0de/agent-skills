---
name: frontend-form-dialog
description: Golden guideline for designing create/edit form dialogs in CRM applications. Use when building any modal form for creating or editing a record — users, pets, appointments, contacts, or any entity with multiple fields.
---

# Frontend Form Dialog Design Guideline

CRM apps use a single, reusable dialog component for both creating and editing a record. The dialog detects which mode it's in based on whether an existing record is passed as a prop.

## Component Signature

```tsx
interface EntityFormDialogProps {
  open: boolean
  onOpenChange: (open: boolean) => void
  entity?: Entity | null  // null/undefined = create mode, populated = edit mode
}
```

The `isEditing` flag drives all conditional text:

```tsx
const isEditing = !!entity
```

## Dialog Container

Use `sm:max-w-[500px]` width with vertical scroll for long forms:

```tsx
<Dialog open={open} onOpenChange={onOpenChange}>
  <DialogContent className="sm:max-w-[500px] max-h-[90vh] overflow-y-auto backdrop-blur-sm">
    <DialogHeader>
      <DialogTitle>{isEditing ? 'Edit Record' : 'Add New Record'}</DialogTitle>
      <DialogDescription>
        {isEditing
          ? 'Update the information below.'
          : 'Enter the details for the new record.'}
      </DialogDescription>
    </DialogHeader>

    <form onSubmit={handleSubmit} className="space-y-4">
      {/* Fields */}
      <DialogFooter>
        <Button type="button" variant="outline" onClick={() => onOpenChange(false)}>
          Cancel
        </Button>
        <Button type="submit">{isEditing ? 'Save Changes' : 'Add Record'}</Button>
      </DialogFooter>
    </form>
  </DialogContent>
</Dialog>
```

## Field Layout

Use a 2-column grid for fields that logically pair (name/species, date/time, type/status). Full-width for standalone fields and textareas.

```tsx
<div className="grid gap-4 sm:grid-cols-2">
  <div className="space-y-2">
    <Label htmlFor="name">Name</Label>
    <Input id="name" value={formData.name} onChange={...} placeholder="..." required />
  </div>

  <div className="space-y-2">
    <Label htmlFor="species">Species</Label>
    <Select value={formData.species} onValueChange={...}>
      <SelectTrigger id="species"><SelectValue placeholder="Select..." /></SelectTrigger>
      <SelectContent>
        {options.map(opt => <SelectItem key={opt.value} value={opt.value}>{opt.label}</SelectItem>)}
      </SelectContent>
    </Select>
  </div>
</div>

{/* Full-width textarea at the end */}
<div className="space-y-2">
  <Label htmlFor="notes">Notes</Label>
  <Textarea id="notes" value={formData.notes} onChange={...} rows={3} placeholder="..." />
</div>
```

Always use `space-y-2` inside each field wrapper (Label + Input pair).

## Form State Initialization

Use `useEffect` to populate the form when editing, and reset it when creating:

```tsx
useEffect(() => {
  if (entity) {
    setFormData({
      name: entity.name || '',
      species: entity.species || 'dog',
      // ... all fields with fallbacks
    })
  } else {
    setFormData({
      name: '',
      species: 'dog',
      // ... reset defaults
    })
  }
}, [entity])
```

## Submit Handler

A single `handleSubmit` calls either `add` or `update` based on `isEditing`, then closes the dialog:

```tsx
const handleSubmit = (e: React.FormEvent) => {
  e.preventDefault()
  if (isEditing && entity) {
    updateEntity(entity.id, formData)
  } else {
    addEntity(formData)
  }
  onOpenChange(false)
}
```

## Smart Field Relationships

When one field value should automatically populate another (e.g., selecting a Pet auto-fills its Owner), handle it in a dedicated change handler:

```tsx
const handlePetChange = (petId: string) => {
  const pet = pets.find((p) => p.id === petId)
  setFormData({
    ...formData,
    petId,
    ownerId: pet?.ownerId || formData.ownerId,  // auto-fill related field
  })
}
```

## Footer Button Order

In `DialogFooter`, Cancel always comes before the primary action. Cancel uses `variant="outline"`, the primary action uses the default (filled) style.

```tsx
<DialogFooter>
  <Button type="button" variant="outline" onClick={() => onOpenChange(false)}>Cancel</Button>
  <Button type="submit">{isEditing ? 'Save Changes' : 'Add Record'}</Button>
</DialogFooter>
```

## Rules

- Always use `Label` with a matching `htmlFor` — never skip labels for accessibility.
- Required fields use the HTML `required` attribute — never custom validation on blur.
- `Textarea` fields go at the bottom of the form, after all Input/Select fields.
- Close the dialog on successful submit — never keep it open to show a success message.
- Never use the same state variable for both edit and create form data — use `useEffect` to sync.
- Dialog width: `sm:max-w-[500px]` for 1-2 column forms; `sm:max-w-[700px]` only if the form has 3+ logical sections.
```

