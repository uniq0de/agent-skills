---
name: frontend-sidebar
description: Golden guideline for designing CRM/dashboard sidebars. Use when building a navigation sidebar, app shell layout, mobile drawer nav, or any left-rail navigation system. Applies to Next.js, React, and shadcn/ui-based projects.
---

# Frontend Sidebar Design Guideline

Sidebars in CRM/dashboard apps are the backbone of navigation. They must feel stable, accessible, and context-aware at all times.

## Core Structure

A sidebar has exactly three zones: **Header** (brand identity), **Content** (primary nav), **Footer** (utility/settings). Never mix them.

```tsx
<Sidebar>
  <SidebarHeader className="border-b border-sidebar-border px-4 py-4">
    {/* Logo + App name + subtitle */}
  </SidebarHeader>

  <SidebarContent>
    <SidebarGroup>
      <SidebarGroupContent>
        <SidebarMenu>
          {/* Primary nav items */}
        </SidebarMenu>
      </SidebarGroupContent>
    </SidebarGroup>
  </SidebarContent>

  <SidebarFooter className="border-t border-sidebar-border">
    {/* Settings, Help, Profile */}
  </SidebarFooter>
</Sidebar>
```

## Brand Header

The header is a clickable link to "/" that shows the app icon and name. Always pair a filled icon in a rounded container with a two-line identity (app name bold, subtitle muted).

```tsx
<Link href="/" className="flex items-center gap-3">
  <div className="flex size-9 items-center justify-center rounded-lg bg-primary text-primary-foreground">
    <AppIcon className="size-5" />
  </div>
  <div className="flex flex-col">
    <span className="text-lg font-semibold">AppName</span>
    <span className="text-xs text-sidebar-foreground/70">Tagline or Role</span>
  </div>
</Link>
```

## Navigation Items

Define nav items as a typed array, never inline. Separate primary navigation from utility navigation (Settings, Help, etc.). Utility items live in the Footer.

```tsx
const navItems = [
  { title: 'Dashboard', href: '/', icon: LayoutDashboard },
  { title: 'Records',   href: '/records', icon: FileText },
  // ...
]

const bottomNavItems = [
  { title: 'Settings', href: '/settings', icon: Settings },
]
```

## Active State Detection

Always use `usePathname()`. Match exact path for root `/`; use `startsWith` for all other routes so nested pages keep the parent item highlighted.

```tsx
const pathname = usePathname()

const isActive = pathname === item.href ||
  (item.href !== '/' && pathname.startsWith(item.href))

<SidebarMenuButton asChild isActive={isActive}>
  <Link href={item.href}>{...}</Link>
</SidebarMenuButton>
```

## Color Token System

The sidebar uses a **separate token namespace** — never use `bg-background` or `text-foreground` inside the sidebar. This allows the sidebar to have its own dark/light theming independent of the main content area.

```css
:root {
  --sidebar: oklch(0.22 0.02 180);            /* Dark background */
  --sidebar-foreground: oklch(0.95 0 0);       /* Light text */
  --sidebar-primary: oklch(0.637 0.137 175);   /* Brand accent */
  --sidebar-accent: oklch(0.3 0.02 180);       /* Hover state */
  --sidebar-border: oklch(0.35 0.02 180);      /* Borders */
}
```

Use `border-sidebar-border`, `text-sidebar-foreground`, `bg-sidebar-accent` for any sidebar-internal elements.

## Mobile Behavior

On mobile, the sidebar becomes a **Sheet overlay** triggered by a hamburger button in the header. Close the sheet on any nav item click.

```tsx
const { setOpenMobile } = useSidebar()
const handleNavClick = () => setOpenMobile(false)

// Header hamburger (mobile only)
{isMobile && (
  <Button variant="ghost" size="icon" className="md:hidden" onClick={toggleSidebar}>
    <Menu className="size-5" />
  </Button>
)}
```

## App Shell Layout

Wrap everything in `SidebarProvider`. The main content lives inside `SidebarInset`, which automatically adjusts its left margin when the sidebar collapses.

```tsx
<SidebarProvider>
  <AppSidebar />
  <SidebarInset>
    <AppHeader breadcrumbs={breadcrumbs} />
    <main className="flex-1 overflow-auto">
      {children}
    </main>
  </SidebarInset>
</SidebarProvider>
```

## Sticky Header with Breadcrumbs

The header inside `SidebarInset` is sticky and shows contextual breadcrumbs. On mobile, it matches sidebar colors (seamless visual transition). On desktop, it uses standard app colors.

```tsx
<header className="sticky top-0 z-40 flex h-14 items-center gap-4 border-b
  bg-sidebar text-sidebar-foreground border-sidebar-border   /* mobile */
  md:bg-background md:text-foreground md:border-border       /* desktop */
  px-4 md:px-6">
```

## Rules

- Icon size inside nav items: always `size-4` (16px). Never larger.
- Gap between icon and label: `gap-2` or via SidebarMenuButton defaults.
- Never add more than 8 primary nav items. If you have more, group them with `SidebarGroupLabel`.
- Settings and other utility links always go in `SidebarFooter`, never in `SidebarContent`.
- Never use `font-bold` on nav item labels — `font-medium` is the maximum weight.
```

