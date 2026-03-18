---
description: Provide design system context — tokens, components, patterns, and usage guidelines
---

# Design System Context

When invoked, gather and present the project's design system information to help
engineers understand available primitives before building new UI.

## Before You Build

**Always check existing primitives first.**
AI-generated interfaces fail when they reinvent what already exists. Before writing
any component code:

1. Search for existing components that match the need
2. Check the design token system for colors, spacing, typography
3. Review the icon set for available icons
4. Look for established patterns in similar screens

If a component almost fits, extend or wrap it rather than creating a new one.

---

## Design Tokens

### Color System

Surface the project's color tokens. Look for:
- Semantic tokens (not raw hex): `--color-primary`, `--color-error`, etc.
- Background/foreground pairs for each surface
- Interactive state colors: hover, active, focus, disabled
- Dark mode variants

**Anti-patterns to flag:**
- Hardcoded hex values outside the token system
- Pure black (`#000`) or pure white (`#fff`) — neutrals should be tinted toward brand
- Gray text on colored backgrounds (insufficient contrast)
- Neon gradients, cyan-on-dark palettes (2024-2025 AI fingerprints)

### Typography Scale

Surface the type system:
- Font families: display, body, mono
- Size scale with semantic names: `text-xs` through `text-4xl`
- Weight scale: regular, medium, semibold, bold
- Line heights and letter spacing per size
- OpenType features if available (tabular nums, ligatures)

**Anti-patterns to flag:**
- Arbitrary font-size values outside the scale
- Overused fonts: Inter, Roboto, Arial (prefer distinctive choices)
- Raw `px` or `rem` values instead of scale tokens
- Missing `font-variant-numeric: tabular-nums` for number comparisons

### Spacing Scale

Surface the spacing system:
- Base unit and scale multipliers
- Semantic spacing: `space-xs` through `space-4xl`
- Component-specific spacing patterns

**Anti-patterns to flag:**
- Arbitrary pixel values outside the scale
- Inconsistent gaps between similar elements
- Missing vertical rhythm

### Other Tokens

- Border radii scale
- Shadow system (layered: ambient + direct light)
- Breakpoints and container widths
- Z-index scale
- Transition/animation timing tokens

---

## Component Inventory

When invoked, scan the codebase for the component library and surface:

### Core Components
List available UI primitives:
- Button (variants, sizes, states)
- Input, Select, Checkbox, Radio, Switch
- Card, Dialog, Sheet, Drawer
- Toast, Tooltip, Popover
- Avatar, Badge, Tag
- Skeleton, Spinner, Progress

For each component, provide:
- Import path
- Available props with types
- Usage example
- Any deprecated variants

### Layout Components
- Container, Grid, Stack, Flex
- Divider, Spacer
- Page layouts and templates

### Data Display
- Table, DataGrid
- List, Tree
- Charts if available

---

## Icon System

Surface the icon approach:
- Icon library in use (Lucide, Heroicons, custom, etc.)
- Import pattern and usage
- Available icon set or link to reference
- Size variants and color handling

**Rules:**
- All icons need accessible labels (aria-label or visually hidden text)
- Icon-only buttons must have aria-label
- Decorative icons get `aria-hidden="true"`

---

## Motion System

Surface motion tokens and patterns:

### Timing
- Duration tokens: `duration-150`, `duration-300`, etc.
- Easing curves: `ease-out` for entrances, `ease-in` for exits
- Never use `linear` for UI (only for spinners/loops)

### Patterns
- Page transitions: fade or slide, `duration-300 ease-out`
- Dialog/modal: scale + fade (`scale-95 opacity-0` → `scale-100 opacity-100`)
- Bottom sheet: slide up (`translateY(100%)` → `0`)
- Tooltips/popovers: fade only
- Hover states: `duration-150 ease-out`

### Performance
- Animate only `transform` and `opacity`
- Never animate `width`, `height`, `top`, `left`, `margin`
- Use `will-change` sparingly

### Reduced Motion
- Every animation needs `prefers-reduced-motion` fallback
- Tailwind: `motion-safe:` / `motion-reduce:`
- Framer Motion: `useReducedMotion()` hook

---

## Interaction Patterns

### States
Every interactive element must define:
- Default → Hover → Focus-visible → Active → Disabled
- Loading state for async actions
- Error state with recovery path

### Touch Targets
- Minimum 44×44px for all interactive elements
- Expand small visual elements via padding or pseudo-element
- Set `touch-action: manipulation` on controls

### Focus Management
- Visible focus rings using `:focus-visible`
- Focus trap in modals, restore on close
- Skip-to-main link on all full pages
- `scroll-margin-top` on anchor targets

### Forms
- Labels associated with inputs (never placeholder-only)
- Validation on blur/submit (not every keystroke)
- Errors adjacent to fields
- Enter submits single-input forms
- `autocomplete` attributes for autofill

---

## Anti-Patterns Registry

Flag these AI-generation fingerprints immediately:

### Typography
- Inter, Roboto, Arial as primary fonts
- Gradient text effects
- Centered body copy

### Color
- Cyan/purple on dark backgrounds
- Pure black/white without tinting
- Neon gradients
- Gray text on colored backgrounds

### Layout
- Identical card grids
- Cards nested in cards
- Everything centered
- Predictable 3-column layouts

### Effects
- Glassmorphism without purpose
- Generic drop shadows
- Bounce/elastic easing
- Decorative sparkles/particles

### Motion
- `transition: all` (always specify properties)
- Animations on layout properties
- Autoplay animations
- Missing reduced-motion fallbacks

---

## Output Format

When this skill is invoked, produce a structured reference:

1. **Quick Reference** — Token values and component imports for copy/paste
2. **Available Components** — Inventory with usage examples
3. **Patterns in Use** — Common patterns from the existing codebase
4. **Watch Out** — Project-specific anti-patterns or gotchas

Keep output scannable. Group by category. Include code snippets.

---

## Connecting to Your Project

To make this skill fully functional, configure:

1. **Token source**: Point to `tokens.json`, `theme.ts`, or CSS variables file
2. **Component index**: Path to component library exports
3. **Icon reference**: Link to icon set documentation
4. **Figma/Storybook**: URLs for visual reference

Add a `.impeccable.md` or project-specific config at the repo root to capture:
- Brand personality and tone
- Target audience
- Design system URLs
- Project-specific constraints
