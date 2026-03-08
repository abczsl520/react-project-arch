---
name: react-project-arch
description: React project architecture standards for AI-assisted development. Enforces component splitting (<250 lines), feature-based organization, state/API separation, and lazy loading. Use when creating new React/Vite projects, refactoring large component files, or when AI context window is being consumed by oversized JSX. Covers SaaS dashboards, admin panels, data-heavy apps, and content platforms. Part of the project-arch family.
---

# React Project Architecture for AI-Friendly Development

Architecture standards that keep React codebases small enough for AI agents to read and edit without blowing the context window.

Built on [project-arch-core](https://github.com/abczsl520/project-arch-core) principles.

## When to Apply

✅ Apply when:
- Building a React SPA or SaaS dashboard with 5+ pages
- Codebase has components over 300 lines
- AI agent is hitting context limits or making edit errors on large JSX files
- Starting a new React + Vite project

❌ Don't apply when:
- Single-page tool or widget (one component is fine)
- Using Next.js App Router / Remix / Astro — follow the framework's conventions first, apply these principles where the framework is silent
- Prototype / POC phase — split when stabilizing
- Static site with no interactivity

## Core Rules

- **Component file ≤ 250 lines** — components are denser than backend code; 250 lines ≈ 3.3K tokens
- **Page file ≤ 400 lines** — pages can be larger but split hooks/logic out
- **Custom hook ≤ 150 lines** — if longer, it's doing too much
- **API service file ≤ 200 lines** — group by domain, not one giant api.js
- **App entry ≤ 80 lines** — routes + providers only, zero business logic
- **No inline styles over 10 properties** — extract to CSS module or styled component

## TypeScript

For TypeScript projects (recommended for new projects in 2025+):
- Use `.tsx` / `.ts` extensions instead of `.jsx` / `.js`
- Shared type definitions go in `types/` directory
- Co-located types stay in the same file if under 30 lines, otherwise extract to `[module].types.ts`

## Project Type Selection

| Type | Signals | Reference |
|------|---------|-----------|
| **SaaS / Admin Dashboard** | Tables, forms, RBAC, charts | [references/saas.md](references/saas.md) |
| **Content / Marketing** | Blog, CMS, landing pages, SEO | [references/content.md](references/content.md) |
| **Data-Heavy App** | Charts, real-time, large datasets, filters | [references/saas.md](references/saas.md) |

For common code patterns (App entry, API services, hooks, state management), see [references/patterns.md](references/patterns.md).

## Standard Directory Structure (Vite + React)

```
project-name/
  client/                      ← or src/ if no backend
    src/
      App.tsx                  ← Routes + Providers only (≤80 lines)
      main.tsx                 ← createRoot, nothing else
      layouts/
        MainLayout.tsx         ← Shell: sidebar + header + Outlet
      pages/                   ← One folder per route
        dashboard/
          index.tsx            ← Page component (≤400 lines)
          useDashboard.ts      ← Page-specific hook
          DashboardChart.tsx   ← Page-specific sub-component
        users/
          index.tsx
          UserTable.tsx
          useUsers.ts
      components/              ← Shared/reusable components
        DataTable.tsx
        SearchInput.tsx
        StatusBadge.tsx
      hooks/                   ← Global custom hooks
        useAuth.ts
        usePermission.ts
      services/                ← API calls, grouped by domain
        api.ts                 ← HTTP client instance + interceptors (≤100 lines)
        authApi.ts
        userApi.ts
        dashboardApi.ts
      store/                   ← State management (one file per slice)
        authStore.ts
        appStore.ts
      types/                   ← Shared TypeScript types (if using TS)
        user.ts
        api.ts
      utils/                   ← Pure helper functions
        format.ts
        validate.ts
      styles/                  ← Global styles
        global.css
        variables.css
    dist/                      ← Build output
    vite.config.ts
    package.json
  server/                      ← Backend (follow nodejs-project-arch)
    index.js
    routes/
    services/
    config.js
```

## Splitting Rules

| What | Max Lines | Split Into |
|------|-----------|-----------|
| Page with inline table + chart + form | 400 | `index.tsx` (orchestration) + `PageTable.tsx` + `PageChart.tsx` + `PageForm.tsx` |
| Component with complex state logic | 250 | Component (render) + `useComponentName.ts` (hook) |
| Giant `api.ts` with 30+ endpoints | 200/file | `authApi.ts` + `userApi.ts` + `orderApi.ts` + ... |
| Monolithic store | 300 | `authStore.ts` + `appStore.ts` + `dataStore.ts` |
| Page with 500+ lines of JSX | 400 | Extract repeated patterns into `components/` |
| Inline styles object > 50 lines | — | Extract to `.module.css` or `styles/` |

## Why These Limits?

| Scenario | Tokens | Context % (200K) |
|----------|--------|-------------------|
| 800-line page component | ~10.4K | 5.2% |
| 250-line component | ~3.3K | 1.6% |
| 150-line hook | ~2.0K | 1.0% |
| Read page + hook + 2 sub-components | ~10.6K | 5.3% |

Same total tokens as reading one 800-line monolith, but AI can now edit each piece independently with full understanding.
