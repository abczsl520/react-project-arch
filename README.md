# react-project-arch

**React project architecture standards for AI-assisted development.**

Part of the [project-arch](https://github.com/abczsl520/project-arch-core) family. Keeps React codebases modular enough for AI agents to read and edit without blowing the context window.

## Core Rules

- **Component ≤ 250 lines** / Page ≤ 400 / Hook ≤ 150 / API service ≤ 200
- **App.jsx ≤ 80 lines** — routes + providers only
- **Feature-based organization** — `pages/[feature]/` with co-located hooks and sub-components
- **API calls in `services/`** — one file per domain, not one giant api.js
- **State in `store/`** — one file per slice (Zustand recommended)
- **Lazy load all pages** — `React.lazy` + `Suspense`

## Supported Project Types

| Type | Reference |
|------|-----------|
| SaaS / Admin Dashboard | [references/saas.md](references/saas.md) |
| Content / Marketing Site | [references/content.md](references/content.md) |
| Data-Heavy App | [references/saas.md](references/saas.md) |

## Typical Structure

```
client/src/
  App.jsx              ← Routes + Providers (≤80 lines)
  layouts/             ← MainLayout, AuthLayout
  pages/               ← One folder per route
    dashboard/
      index.jsx        ← Page component
      useDashboard.js  ← Page hook
      DashboardChart.jsx
  components/          ← Shared reusable components
  hooks/               ← Global custom hooks
  services/            ← API calls by domain
  store/               ← State management slices
  utils/               ← Pure helpers
  styles/              ← Global CSS
```

## Token Savings

| Scenario | Before | After | Savings |
|----------|--------|-------|---------|
| Read a feature page | ~10.4K (800-line monolith) | ~3.3K (250-line component) | 68% |
| Edit page + hook + sub-component | 1 read of 10.4K | 3 reads of ~2.5K each = 7.5K | 28% fewer tokens, 3x better edit accuracy |

## Installation

```bash
git clone https://github.com/abczsl520/react-project-arch.git
cp -r react-project-arch ~/.agents/skills/
```

## 🧬 Part of the project-arch Family

| Skill | Language | Status |
|-------|----------|--------|
| [project-arch-core](https://github.com/abczsl520/project-arch-core) | Universal principles | ✅ Foundation |
| [nodejs-project-arch](https://github.com/abczsl520/nodejs-project-arch) | Node.js / Express | ✅ Available |
| **react-project-arch** (this) | React / Vite | ✅ Available |
| python-project-arch | Python | 🔲 Community welcome |
| go-project-arch | Go | 🔲 Community welcome |

## License

MIT
