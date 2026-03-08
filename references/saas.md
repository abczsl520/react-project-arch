# SaaS / Admin Dashboard Architecture

For admin panels, SaaS platforms, and data management apps. Examples use plain JSX — substitute your UI library (Ant Design, MUI, shadcn/ui, etc.) as needed.

## Directory Structure

```
client/src/
  App.tsx                          ← Routes + Providers (≤80 lines)
  main.tsx                         ← Entry point
  layouts/
    MainLayout.tsx                 ← Sidebar + Header + Outlet (≤250 lines)
    AuthLayout.tsx                 ← Login/register wrapper
  pages/
    dashboard/
      index.tsx                    ← Dashboard page (≤400 lines)
      DashboardCards.tsx           ← KPI cards row
      DashboardCharts.tsx          ← Charts section
      useDashboard.ts              ← Data fetching hook
    users/
      index.tsx                    ← User management page
      UserTable.tsx                ← Table component
      UserForm.tsx                 ← Create/edit modal
      useUsers.ts                  ← CRUD hook
    settings/
      index.tsx
      SystemConfig.tsx             ← Config editor (admin.html equivalent)
      useSettings.ts
    [module-name]/                 ← One folder per business module
      index.tsx
      [SubComponent].tsx
      use[Module].ts
  components/
    DataTable.tsx                  ← Reusable table with pagination/filter/sort
    SearchBar.tsx                  ← Global search
    StatusBadge.tsx                ← Status indicator
    PermissionGuard.tsx            ← RBAC wrapper
    AuthGuard.tsx                  ← Login check wrapper
    PageHeader.tsx                 ← Consistent page headers
    ExportButton.tsx               ← CSV/Excel export
  hooks/
    useAuth.ts                     ← Login/logout/token
    usePermission.ts               ← RBAC checks
    useTable.ts                    ← Shared table state (pagination, filters, sort)
    useExport.ts                   ← Export logic
  services/
    api.ts                         ← HTTP client instance (≤100 lines)
    authApi.ts
    userApi.ts
    dashboardApi.ts
    [module]Api.ts                 ← One per business module
  store/
    authStore.ts                   ← User + token + permissions
    appStore.ts                    ← Sidebar collapsed, theme, notifications
  types/
    user.ts
    permission.ts
  utils/
    format.ts                      ← Date/number/currency formatting
    validate.ts                    ← Form validation rules
    permission.ts                  ← Permission constants + helpers
  styles/
    global.css
    variables.css
```

## SaaS-Specific Patterns

### RBAC Permission Guard

```tsx
// components/PermissionGuard.tsx
import { useAuthStore } from '../store/authStore';

export default function PermissionGuard({
  permission,
  children,
  fallback = null,
}: {
  permission?: string;
  children: React.ReactNode;
  fallback?: React.ReactNode;
}) {
  const { user } = useAuthStore();
  const perms = user?.permissions || [];

  if (!permission || perms.includes(permission) || user?.role === 'admin') {
    return <>{children}</>;
  }
  return <>{fallback}</>;
}
```

Usage:
```tsx
<PermissionGuard permission="users:manage">
  <button onClick={handleDelete}>Delete</button>
</PermissionGuard>
```

### Settings Page = Admin Dashboard Equivalent

In Node.js projects we have `admin.html`. In React SaaS, the Settings page serves the same purpose:

```tsx
// pages/settings/SystemConfig.tsx
import { useState, useEffect } from 'react';
import { getConfig, updateConfig } from '../../services/settingsApi';

export default function SystemConfig() {
  const [config, setConfig] = useState({});
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    getConfig().then(setConfig);
  }, []);

  const handleSave = async () => {
    setLoading(true);
    try {
      await updateConfig(config);
      // Show success notification (use your UI library's toast/message)
    } finally {
      setLoading(false);
    }
  };

  // Render form fields from config object
  // Use your UI library's Form component (Ant Design Form, MUI TextField, etc.)
  return (
    <div>
      <h2>System Configuration</h2>
      {Object.entries(config).map(([key, value]) => (
        <div key={key}>
          <label>{key}</label>
          <input
            value={String(value)}
            onChange={e => setConfig(prev => ({ ...prev, [key]: e.target.value }))}
          />
        </div>
      ))}
      <button onClick={handleSave} disabled={loading}>
        {loading ? 'Saving...' : 'Save'}
      </button>
    </div>
  );
}
```

### Shared DataTable Component

```tsx
// components/DataTable.tsx — Reusable across all pages (≤250 lines)
// Wrap your UI library's table (Ant Design Table, MUI DataGrid, TanStack Table, etc.)

interface DataTableProps {
  columns: any[];
  data: any[];
  loading?: boolean;
  pagination?: { current: number; pageSize: number; total: number };
  onPageChange?: (page: number) => void;
  onSearch?: (query: string) => void;
  extra?: React.ReactNode;  // toolbar buttons (export, batch actions)
}

export default function DataTable({
  columns, data, loading, pagination, onPageChange, onSearch, extra,
}: DataTableProps) {
  const [search, setSearch] = useState('');

  return (
    <div>
      <div style={{ marginBottom: 16, display: 'flex', gap: 8 }}>
        <input
          placeholder="Search..."
          value={search}
          onChange={e => setSearch(e.target.value)}
          onKeyDown={e => e.key === 'Enter' && onSearch?.(search)}
        />
        {extra}
      </div>
      {/* Replace with your UI library's Table component */}
      <table>
        <thead>
          <tr>{columns.map(col => <th key={col.key}>{col.title}</th>)}</tr>
        </thead>
        <tbody>
          {loading ? (
            <tr><td colSpan={columns.length}>Loading...</td></tr>
          ) : (
            data.map(row => (
              <tr key={row.id}>
                {columns.map(col => <td key={col.key}>{col.render ? col.render(row) : row[col.key]}</td>)}
              </tr>
            ))
          )}
        </tbody>
      </table>
      {/* Add pagination controls */}
    </div>
  );
}
```

## Module Addition Checklist

When adding a new business module:

1. Create `pages/[module]/index.tsx` — page component (≤400 lines)
2. Create `pages/[module]/use[Module].ts` — data hook (≤150 lines)
3. Create `services/[module]Api.ts` — API calls (≤200 lines)
4. Add lazy route in `App.tsx`
5. Add menu item in `MainLayout.tsx`
6. Add permission constant in `utils/permission.ts`
7. Wrap with `<PermissionGuard>` if needed
