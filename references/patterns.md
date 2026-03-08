# Common React Patterns

Code patterns for React projects. Examples use plain JSX — substitute your UI library (Ant Design, MUI, shadcn/ui, etc.) as needed.

## App Entry Pattern

```tsx
// App.tsx — Routes + Providers only (≤80 lines)
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import MainLayout from './layouts/MainLayout';
import AuthGuard from './components/AuthGuard';

const Login = lazy(() => import('./pages/login'));
const Dashboard = lazy(() => import('./pages/dashboard'));
const Users = lazy(() => import('./pages/users'));
const Settings = lazy(() => import('./pages/settings'));

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

All pages lazy-loaded. App.tsx is pure routing — no data fetching, no state logic.

## API Service Pattern

```ts
// services/api.ts — HTTP client instance (≤100 lines)
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

```ts
// services/userApi.ts — Domain-specific (≤200 lines)
import api from './api';

export const getUsers = (params) => api.get('/users', { params });
export const getUser = (id) => api.get(`/users/${id}`);
export const createUser = (data) => api.post('/users', data);
export const updateUser = (id, data) => api.put(`/users/${id}`, data);
export const deleteUser = (id) => api.delete(`/users/${id}`);
```

One file per domain. If `userApi.ts` grows past 200 lines, split further by sub-domain.

## Custom Hook Pattern

Extract state logic from components into co-located hooks:

```ts
// pages/users/useUsers.ts — Page-specific hook (≤150 lines)
import { useState, useEffect, useCallback } from 'react';
import { getUsers, deleteUser } from '../../services/userApi';

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
    fetchUsers(pagination.current);
  }, [fetchUsers, pagination.current]);

  useEffect(() => { fetchUsers(); }, [fetchUsers]);

  return { users, loading, pagination, fetchUsers, handleDelete };
}
```

```tsx
// pages/users/index.tsx — Clean page component
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

## State Management Pattern

The pattern is: **one store file per domain**. Don't put everything in one giant store.

### Zustand example

```ts
// store/authStore.ts
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

### Redux Toolkit example

```ts
// store/authSlice.ts
import { createSlice } from '@reduxjs/toolkit';

const authSlice = createSlice({
  name: 'auth',
  initialState: { user: null, token: localStorage.getItem('token') },
  reducers: {
    setAuth: (state, action) => {
      state.user = action.payload.user;
      state.token = action.payload.token;
      localStorage.setItem('token', action.payload.token);
    },
    logout: (state) => {
      state.user = null;
      state.token = null;
      localStorage.removeItem('token');
    },
  },
});

export const { setAuth, logout } = authSlice.actions;
export default authSlice.reducer;
```

Either library works. The architecture rule is the same: **one slice/store per domain**.
