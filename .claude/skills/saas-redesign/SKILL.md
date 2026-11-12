---
name: saas-redesign
description: Redesign a Vue 3 app's UI into a modern SaaS interface with a left vertical sidebar instead of a top nav, design tokens, and consistent spacing. Use when asked to modernize, restyle, or standardize the look of a Vue 3 frontend.
---

# SaaS Redesign

Turn a Vue 3 app with a top navigation bar into a SaaS-style interface: collapsible left sidebar, CSS-variable design tokens, consistent spacing, and a polished neutral look. Work in four phases and stop for approval before any edit.

## Rules of Engagement

- **Approval gate**: Phases 1 and 2 are read-only. Do not edit any file until the user approves the Phase 2 proposal.
- **Delegate Vue work**: when the project requires it (this repo does), any new or significantly changed `.vue` file goes to the `vue-expert` subagent. Give it the approved token set and the exact file scope.
- **Frontend only**: change files under the client directory (`client/src/` here). Never touch the backend or data files.
- **No emojis**: use inline SVG icons (24x24 viewBox, `stroke="currentColor"`, `stroke-width="1.75"`, `aria-hidden="true"`).
- **Preserve behavior**: keep routes, `v-for` keys, loading and error states, i18n usage, emitted events, and API calls unchanged. This is a visual and layout change only.
- **Public repo**: never add credentials, internal hostnames, or private registry URLs.
- **No commits** unless the user asks. Prefer working on a scratch branch.

## Phase 1: Audit (read-only)

Find and report, with file paths and line numbers:

1. **Shell**: the root component (usually `App.vue`), its top nav, and what sits in it (logo, links, language switcher, profile menu, action buttons).
2. **Router**: route list (`main.js` or `router/`). Note views that exist but have no route.
3. **Nav labels**: which use i18n and which are hard-coded strings.
4. **Style architecture**: global vs `<style scoped>`, whether CSS variables exist, where resets live.
5. **Repeated values**: count hex colors, border radii, shadows, padding and gap values.

   ```bash
   grep -rhoE '#[0-9a-fA-F]{6}\b' client/src | sort | uniq -c | sort -rn | head -20
   grep -rhoE 'border-radius: *[0-9.]+(px|rem)' client/src | sort | uniq -c | sort -rn
   grep -rhoE '(padding|gap): *[0-9.]+(px|rem)' client/src | sort | uniq -c | sort -rn | head
   ```
6. **Coupled offsets**: any `top:` or `height:` that hard-codes the nav height (for example a second sticky bar at `top: 70px`).
7. **Responsiveness**: presence of `@media` rules.
8. **Duplicated overrides**: classes like `.page-header`, `.card-header` or `.kpi-card` redefined in individual views.

Output a short table of findings. Do not propose fixes yet.

## Phase 2: Propose, then stop

Present three things and ask the user to approve or adjust them.

### a) Token set

Add to `:root` in the global stylesheet. Collapse near-duplicate values (for example two blues, two reds) into one token each.

```css
:root {
  /* Color: slate neutrals, one primary */
  --color-bg: #f8fafc;
  --color-surface: #ffffff;
  --color-border: #e2e8f0;
  --color-border-strong: #cbd5e1;
  --color-text: #0f172a;
  --color-text-muted: #64748b;
  --color-text-faint: #94a3b8;
  --color-hover: #f1f5f9;
  --color-primary: #2563eb;
  --color-primary-soft: #eff6ff;
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-danger: #dc2626;

  /* Spacing: 4px base */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-5: 1.25rem;
  --space-6: 1.5rem;
  --space-8: 2rem;

  /* Shape */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --shadow-sm: 0 1px 2px rgba(15, 23, 42, 0.06);
  --shadow-md: 0 4px 12px rgba(15, 23, 42, 0.08);
  --shadow-modal: 0 20px 50px rgba(15, 23, 42, 0.15);

  /* Layout */
  --sidebar-width: 240px;
  --sidebar-width-collapsed: 64px;
  --content-max-width: 1600px;
}
```

Keep the project's existing palette. Only substitute the approved hex values; do not invent a new color scheme unless the user asks for one.

### b) Layout wireframe

```
+-----------+--------------------------------------------+
| Logo      |  Page title                    [actions]   |
|           |  [ Filter bar                            ] |
| Nav item  |--------------------------------------------|
| Nav item  |                                            |
| Nav item  |   <router-view />  (cards, tables, charts) |
| ...       |                                            |
|           |                                            |
| Language  |                                            |
| Profile   |                                            |
+-----------+--------------------------------------------+
```

- Shell is a CSS Grid: `grid-template-columns: var(--sidebar-width) 1fr`, switching to `var(--sidebar-width-collapsed) 1fr` when collapsed.
- Sidebar: `position: sticky; top: 0; height: 100vh; overflow-y: auto`, white surface with a right border.
- Content column: `padding: var(--space-6) var(--space-8)`, inner `max-width: var(--content-max-width)`.

### c) File-by-file change list

List every file you will create or modify, grouped by the apply steps below. Flag anything risky (large views, unscoped components, shared modals).

Then **stop** and ask the user to approve. Do not proceed on silence or on a background notification.

## Phase 3: Apply, in order

Make each step a reviewable unit. After each, tell the user what changed and confirm the app still renders.

1. **Tokens and global styles**. Add `:root` tokens. Rewrite the shared classes (`.card`, `.stat-card`, `.badge`, tables, `.page-header`, `.loading`, `.error`) to use them. Do not change class names.
2. **Sidebar component**. Create `components/SidebarNav.vue`:
   - Logo and subtitle at the top.
   - One `<router-link>` per route with an inline SVG icon and label. Use the router's active class (`router-link-exact-active`) rather than comparing `$route.path` by hand.
   - Active item: `--color-primary` text on `--color-primary-soft`, plus a 3px left accent.
   - Collapse toggle (button with `aria-expanded` and `aria-label`).
   - Footer area for the language switcher and profile menu.
   - Collapsed state persisted per viewer. Wrap storage in try/catch and render correctly without it:

     ```js
     const KEY = 'sidebar-collapsed'
     const read = () => { try { return localStorage.getItem(KEY) === '1' } catch { return false } }
     const write = (v) => { try { localStorage.setItem(KEY, v ? '1' : '0') } catch { /* ignore */ } }
     ```
   - In rail mode hide the text but keep `title` and `aria-label` on each link.
3. **Shell**. In the root component, remove the top nav, wrap the layout in the grid, and mount `SidebarNav`. Keep existing modal components and their events wired exactly as before (for example `show-profile-details`, `show-tasks`). Move hard-coded nav strings to the i18n locale files for every supported language.
4. **Filters**. Move the filter bar into the content column under the page header. Remove its sticky positioning and any offset tied to the old nav height. Make sure it still writes to the shared filter state.
5. **Per-view cleanup**, one view at a time, delegated to the Vue subagent. Replace repeated hex, radius, shadow and spacing values with tokens. Delete local redefinitions of shared classes unless the view genuinely needs to differ. Check each view before moving on.

## Phase 4: Verify

1. Start the app (the project's start skill or command).
2. Using Playwright MCP tools, visit every route. For each, capture the expanded and collapsed sidebar and check the console for errors.
3. Confirm: sidebar is on the left, no top bar remains, active state follows navigation, collapse state survives a reload, filters sit in the page header and still change the data, modals open above the sidebar.
4. Re-run the Phase 1 grep. Repeated raw hex values in views should drop sharply.
5. Run the existing test suite as a sanity check. Backend tests should be unaffected.
6. `git diff --stat` should show only client files. Report anything else as a problem.

Report outcomes plainly, including anything you could not verify.

## Pitfalls

- **Coupled offsets**: sticky elements positioned against the old nav height will overlap or leave gaps. Search for them in Phase 1.
- **Hard-coded labels**: a nav label that bypasses i18n will stay English after a language switch. Fix it when moving links into the sidebar.
- **Semantic colors**: status colors (green, blue, yellow, red) carry meaning. Consolidate shades, never change which hue means what.
- **Stacking**: modals and dropdowns need a `z-index` above the sidebar. Check the profile menu opening from the sidebar footer, since it will open upward or sideways rather than downward.
- **Unscoped or unrouted views**: note them in the audit and ask before restyling them.
- **Component style conventions**: follow the file being edited (`<script setup>` or `export default { setup() }`). Do not convert styles between them as part of this work.
- **Out of scope**: a mobile off-canvas drawer is not part of this skill. If the app has no `@media` rules, mention that the sidebar will not collapse automatically on small screens and offer it as a follow-up.
