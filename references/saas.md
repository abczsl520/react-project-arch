# SaaS / Admin Dashboard Architecture

## Directory Structure

```
client/src/
  App.jsx                          ← Routes + Providers (≤80 lines)
  main.jsx                         ← Entry point
  layouts/
    MainLayout.jsx                 ← Sidebar + Header + Outlet (≤250 lines)
    AuthLayout.jsx                 ← Login/register wrapper
  pages/
    dashboard/
      index.jsx                    ← Dashboard page (≤400 lines)
      DashboardCards.jsx           ← KPI cards row
      DashboardCharts.jsx          ← Charts section
      useDashboard.js              ← Data fetching hook
    users/
      index.jsx                    ← User management page
      UserTable.jsx                ← Table component
      UserForm.jsx                 ← Create/edit modal
      useUsers.js                  ← CRUD hook
    settings/
      index.jsx
      SystemConfig.jsx             ← Config editor (like admin.html equivalent)
      useSettings.js
    [module-name]/                 ← One folder per business module
      index.jsx
      [SubComponent].jsx
      use[Module].js
  components/
    DataTable.jsx                  ← Reusable table with pagination/filter/sort
    SearchBar.jsx                  ← Global search
    StatusBadge.jsx                ← Status indicator
    PermissionGuard.jsx            ← RBAC wrapper
    AuthGuard.jsx                  ← Login check wrapper
    PageHeader.jsx                 ← Consistent page headers
    ExportButton.jsx               ← CSV/Excel export
  hooks/
    useAuth.js                     ← Login/logout/token
    usePermission.js               ← RBAC checks
    useTable.js                    ← Shared table state (pagination, filters, sort)
    useExport.js                   ← Export logic
  services/
    api.js                         ← Axios instance (≤100 lines)
    authApi.js
    userApi.js
    dashboardApi.js
    [module]Api.js                 ← One per business module
  store/
    authStore.js                   ← User + token + permissions
    appStore.js                    ← Sidebar collapsed, theme, notifications
  utils/
    format.js                      ← Date/number/currency formatting
    validate.js                    ← Form validation rules
    permission.js                  ← Permission constants + helpers
  styles/
    global.css
    variables.css
```

## SaaS-Specific Patterns

### RBAC Permission Guard

```jsx
// components/PermissionGuard.jsx
import { useAuthStore } from '../store/authStore';

export default function PermissionGuard({ permission, children, fallback = null }) {
  const { user } = useAuthStore();
  const perms = user?.permissions || [];

  if (!permission || perms.includes(permission) || user?.role === 'admin') {
    return children;
  }
  return fallback;
}
```

Usage:
```jsx
<PermissionGuard permission="users:manage">
  <Button onClick={handleDelete}>Delete</Button>
</PermissionGuard>
```

### Settings Page = Admin Dashboard Equivalent

In Node.js projects we have `admin.html`. In React SaaS, the Settings page serves the same purpose:

```jsx
// pages/settings/SystemConfig.jsx
import { useState, useEffect } from 'react';
import { Form, Input, Switch, Button, message } from 'antd';
import { getConfig, updateConfig } from '../../services/settingsApi';

export default function SystemConfig() {
  const [form] = Form.useForm();
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    getConfig().then(data => form.setFieldsValue(data));
  }, [form]);

  const handleSave = async (values) => {
    setLoading(true);
    try {
      await updateConfig(values);
      message.success('Config saved (hot-reloaded)');
    } finally {
      setLoading(false);
    }
  };

  return (
    <Form form={form} onFinish={handleSave} layout="vertical">
      <Form.Item name="siteName" label="Site Name"><Input /></Form.Item>
      <Form.Item name="maintenance" label="Maintenance Mode" valuePropName="checked"><Switch /></Form.Item>
      {/* Auto-generate from config schema */}
      <Button type="primary" htmlType="submit" loading={loading}>Save</Button>
    </Form>
  );
}
```

### Shared DataTable Component

```jsx
// components/DataTable.jsx — Reusable across all pages (≤250 lines)
import { Table, Input, Space } from 'antd';
import { useState } from 'react';

export default function DataTable({
  columns, data, loading, pagination,
  onPageChange, onSearch, rowSelection,
  extra,  // extra toolbar buttons (export, batch actions)
}) {
  const [search, setSearch] = useState('');

  return (
    <div>
      <Space style={{ marginBottom: 16 }}>
        <Input.Search
          placeholder="Search..."
          value={search}
          onChange={e => setSearch(e.target.value)}
          onSearch={onSearch}
          allowClear
        />
        {extra}
      </Space>
      <Table
        columns={columns}
        dataSource={data}
        loading={loading}
        pagination={{
          ...pagination,
          onChange: onPageChange,
          showSizeChanger: true,
          showTotal: total => `Total ${total}`,
        }}
        rowSelection={rowSelection}
        rowKey="id"
      />
    </div>
  );
}
```

## Module Addition Checklist

When adding a new business module:

1. Create `pages/[module]/index.jsx` — page component (≤400 lines)
2. Create `pages/[module]/use[Module].js` — data hook (≤150 lines)
3. Create `services/[module]Api.js` — API calls (≤200 lines)
4. Add lazy route in `App.jsx`
5. Add menu item in `MainLayout.jsx`
6. Add permission constant in `utils/permission.js`
7. Wrap with `<PermissionGuard>` if needed
