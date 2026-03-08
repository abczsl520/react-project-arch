---
name: react-project-arch
description: React project architecture standards for AI-assisted development. Enforces component splitting (<250 lines), feature-based organization, state/API separation, and lazy loading. Use when creating new React/Vite projects, refactoring large component files, or when AI context window is being consumed by oversized JSX. Covers SaaS dashboards, admin panels, data-heavy apps, and content platforms. Part of the project-arch family.
---

# React Project Architecture for AI-Friendly Development

Architecture standards that keep React codebases small enough for AI agents to read and edit without blowing the context window.

Built on [project-arch-core](https://github.com/abczsl520/project-arch-core) principles.

## Core Rules

- **Component file ≤ 250 lines** — components are denser than backend code; 250 lines ≈ 3.3K tokens
- **Page file ≤ 400 lines** — pages can be larger but split hooks/logic out
- **Custom hook ≤ 150 lines** — if longer, it's doing too much
- **API service file ≤ 200 lines** — group by domain, not one giant api.js
- **App.jsx entry ≤ 80 lines** — routes + providers only, zero business logic
- **No inline styles over 10 properties** — extract to CSS/module or styled component

## Project Type Selection

| Type | Signals | Reference |
|------|---------|-----------|
| **SaaS / Admin Dashboard** | Ant Design, tables, forms, RBAC, charts | [references/saas.md](references/saas.md) |
| **Content / Marketing** | Blog, CMS, landing pages, SEO | [references/content.md](references/content.md) |
| **Data-Heavy App** | Charts, real-time, large datasets, filters | [references/saas.md](references/saas.md) |

## Standard Directory Structure (Vite + React)

```
project-name/
  client/                      ← or src/ if no backend
    src/
      App.jsx                  ← Routes + Providers only (≤80 lines)
      main.jsx                 ← ReactDOM.createRoot, nothing else
      layouts/
        MainLayout.jsx         ← Shell: sidebar + header + Outlet
      pages/                   ← One folder per route
        dashboard/
          index.jsx            ← Page component (≤400 lines)
          useDashboard.js      ← Page-specific hook
          DashboardChart.jsx   ← Page-specific sub-component
        users/
          index.jsx
          UserTable.jsx
          useUsers.js
      components/              ← Shared/reusable components
        DataTable.jsx
        SearchInput.jsx
        StatusBadge.jsx
      hooks/                   ← Global custom hooks
        useAuth.js
        usePermission.js
      services/                ← API calls, grouped by domain
        api.js                 ← Axios instance + interceptors (≤100 lines)
        authApi.js             ← Auth endpoints
        userApi.js             ← User endpoints
        dashboardApi.js        ← Dashboard endpoints
      store/                   ← State management
        index.js               ← Zustand/Redux store setup
        authStore.js           ← Auth state slice
        appStore.js            ← App-wide state slice
      utils/                   ← Pure helper functions
        format.js              ← Date/number formatting
        validate.js            ← Form validation rules
      styles/                  ← Global styles (if not using CSS modules)
        global.css
        variables.css
    dist/                      ← Build output
    vite.config.js
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
| Page with inline table + chart + form | 400 | `index.jsx` (orchestration) + `PageTable.jsx` + `PageChart.jsx` + `PageForm.jsx` |
| Component with complex state logic | 250 | Component (render) + `useComponentName.js` (hook) |
| Giant `api.js` with 30+ endpoints | 200/file | `authApi.js` + `userApi.js` + `orderApi.js` + ... |
| Monolithic store | 300 | `authStore.js` + `appStore.js` + `dataStore.js` |
| Page with 500+ lines of JSX | 400 | Extract repeated patterns into `components/` |
| Inline styles object > 50 lines | — | Extract to `.module.css` or `styles/` |

## App.jsx Pattern

```jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import MainLayout from './layouts/MainLayout';
import AuthGuard from './components/AuthGuard';

const Dashboard = lazy(() => import('./pages/dashboard'));
const Users = lazy(() => import('./pages/users'));
const Settings = lazy(() => import('./pages/settings'));
// ... more lazy imports

export default function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/login" element={<Login />} />
          <Route element={<AuthGuard><MainLayout /></AuthGuard>}>
            <Route path="/" element={<Dashboard />} />
            <Route path="/users" element={<Users />} />
            <Route path="/settings" element={<Settings />} />
          </Route>
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

All pages lazy-loaded. App.jsx is pure routing — no data fetching, no state logic.

## API Service Pattern

```js
// services/api.js — Axios instance (≤100 lines)
import axios from 'axios';

const api = axios.create({ baseURL: '/api', timeout: 15000 });

api.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

api.interceptors.response.use(
  res => res.data,
  err => {
    if (err.response?.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(err);
  }
);

export default api;
```

```js
// services/userApi.js — Domain-specific (≤200 lines)
import api from './api';

export const getUsers = (params) => api.get('/users', { params });
export const getUser = (id) => api.get(`/users/${id}`);
export const createUser = (data) => api.post('/users', data);
export const updateUser = (id, data) => api.put(`/users/${id}`, data);
export const deleteUser = (id) => api.delete(`/users/${id}`);
```

## Custom Hook Pattern (Extract Logic from Components)

```jsx
// pages/users/useUsers.js — Page-specific hook
import { useState, useEffect, useCallback } from 'react';
import { getUsers, deleteUser } from '../../services/userApi';
import { message } from 'antd';

export function useUsers() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [pagination, setPagination] = useState({ current: 1, pageSize: 20, total: 0 });

  const fetchUsers = useCallback(async (page = 1) => {
    setLoading(true);
    try {
      const { data, total } = await getUsers({ page, pageSize: pagination.pageSize });
      setUsers(data);
      setPagination(prev => ({ ...prev, current: page, total }));
    } finally {
      setLoading(false);
    }
  }, [pagination.pageSize]);

  const handleDelete = useCallback(async (id) => {
    await deleteUser(id);
    message.success('Deleted');
    fetchUsers(pagination.current);
  }, [fetchUsers, pagination.current]);

  useEffect(() => { fetchUsers(); }, [fetchUsers]);

  return { users, loading, pagination, fetchUsers, handleDelete };
}
```

```jsx
// pages/users/index.jsx — Clean page component
import { useUsers } from './useUsers';
import UserTable from './UserTable';

export default function UsersPage() {
  const { users, loading, pagination, fetchUsers, handleDelete } = useUsers();

  return (
    <div>
      <h2>Users</h2>
      <UserTable
        data={users}
        loading={loading}
        pagination={pagination}
        onPageChange={fetchUsers}
        onDelete={handleDelete}
      />
    </div>
  );
}
```

## State Management (Zustand Recommended)

```js
// store/authStore.js
import { create } from 'zustand';

export const useAuthStore = create((set) => ({
  user: null,
  token: localStorage.getItem('token'),
  setAuth: (user, token) => {
    localStorage.setItem('token', token);
    set({ user, token });
  },
  logout: () => {
    localStorage.removeItem('token');
    set({ user: null, token: null });
  },
}));
```

One store file per domain. Don't put everything in one giant store.

## Why These Limits?

| Scenario | Tokens | Context % (200K) |
|----------|--------|-------------------|
| 800-line page component | ~10.4K | 5.2% |
| 250-line component | ~3.3K | 1.6% |
| 150-line hook | ~2.0K | 1.0% |
| Read page + hook + 2 sub-components | ~10.6K | 5.3% |
| vs. reading one 800-line monolith | ~10.4K | 5.2% |

Same tokens, but AI can now edit each piece independently with full understanding.
