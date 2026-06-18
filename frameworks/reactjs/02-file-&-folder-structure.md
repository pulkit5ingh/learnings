After creating a React app using Vite, the default project structure is clean and minimal.

## Default Vite + React Structure

```bash
my-react-app/
│── node_modules/
│── public/
│   └── vite.svg
│
│── src/
│   ├── assets/
│   │   └── react.svg
│   ├── App.css
│   ├── App.jsx
│   ├── index.css
│   └── main.jsx
│
│── .gitignore
│── eslint.config.js
│── index.html
│── package.json
│── package-lock.json
│── vite.config.js
```

---

## Meaning of Each Folder/File

### `public/`

Static files copied directly to build output.

Examples:

- favicon
- robots.txt
- images

```bash
public/logo.png
```

Use:

```jsx
<img src='/logo.png' />
```

---

### `src/`

Main React source code.

### `src/main.jsx`

Entry point of app.

Usually:

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')).render(<App />);
```

---

### `src/App.jsx`

Main root component.

```jsx
function App() {
  return <h1>Hello React</h1>;
}

export default App;
```

---

### `src/assets/`

Images, icons, fonts imported into components.

```jsx
import logo from './assets/logo.png';
```

---

### `index.html`

Single HTML file where React mounts.

Contains:

```html
<div id="root"></div>
```

---

### `vite.config.js`

Configuration file for Vite

Example:

```js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
});
```

---

# Best Production Structure (Recommended)

After creating app, organize like this:

```bash
src/
│── api/
│── assets/
│── components/
│── pages/
│── layouts/
│── hooks/
│── store/
│── utils/
│── routes/
│── App.jsx
│── main.jsx
```

---

# Example Real Project

```bash
src/
│── components/
│   ├── Navbar.jsx
│   ├── Footer.jsx
│
│── pages/
│   ├── Home.jsx
│   ├── About.jsx
│   ├── Contact.jsx
│
│── App.jsx
│── main.jsx
```

---

# Important Files

### Run Project

```bash
npm run dev
```

### Build Production

```bash
npm run build
```

### Preview Build

```bash
npm run preview
```

---

---

# Enterprise-Level React + Vite Admin Panel Architecture (2026)

For an admin panel with **Users, Orders, Products CRUD**, authentication, forms, validation, and scalable code, use this stack:

- Vite + React
- React Router
- Zustand
- Axios
- TanStack Query
- React Hook Form
- Zod
- Tailwind CSS

This is the kind of structure used for real dashboards and SaaS admin panels.

---

# 1. Create the Project

```bash
npm create vite@latest admin-panel
cd admin-panel
npm install
```

Choose **React** + **JavaScript** (or TypeScript if preferred).

---

# 2. Install Dependencies

```bash
npm install react-router-dom axios zustand @tanstack/react-query react-hook-form zod @hookform/resolvers js-cookie
npm install -D tailwindcss @tailwindcss/vite
```

---

# 3. Enterprise Folder Structure

```bash
src/
│── app/
│   ├── router.jsx
│   ├── providers.jsx
│
│── assets/
│── components/
│   ├── ui/
│   ├── common/
│   ├── tables/
│   └── forms/
│
│── features/
│   ├── auth/
│   │   ├── api.js
│   │   ├── hooks.js
│   │   ├── store.js
│   │   ├── schema.js
│   │   └── pages/
│   │       └── Login.jsx
│   │
│   ├── users/
│   │   ├── api.js
│   │   ├── hooks.js
│   │   ├── schema.js
│   │   └── pages/
│   │
│   ├── products/
│   └── orders/
│
│── hooks/
│   ├── useDebounce.js
│   ├── usePagination.js
│   ├── useModal.js
│   └── useAuthGuard.js
│
│── lib/
│   ├── axios.js
│   ├── queryClient.js
│   └── helpers.js
│
│── layouts/
│   ├── AdminLayout.jsx
│   └── AuthLayout.jsx
│
│── store/
│── styles/
│── main.jsx
```

---

# 4. Why Feature-Based Architecture?

Instead of keeping all API files together, each domain owns itself:

- `features/users`
- `features/orders`
- `features/products`

That means faster scaling, easier maintenance, better team collaboration.

---

# 5. Tailwind Setup

`src/index.css`

```css
@import 'tailwindcss';

body {
  @apply bg-gray-100 text-gray-900;
}
```

---

# 6. Axios Enterprise Setup

`src/lib/axios.js`

```js
import axios from 'axios';
import Cookies from 'js-cookie';

const api = axios.create({
  baseURL: 'https://api.example.com',
  withCredentials: true,
});

api.interceptors.request.use((config) => {
  const token = Cookies.get('token');

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});

api.interceptors.response.use(
  (res) => res,
  (error) => {
    if (error.response?.status === 401) {
      Cookies.remove('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

---

# 7. JWT Auth in Cookie Strategy

## Recommended Secure Flow

Backend sets:

```http
Set-Cookie: token=jwt_here; HttpOnly; Secure; SameSite=Lax
```

Then frontend uses:

```js
withCredentials: true;
```

## If backend expects manual cookie:

```js
Cookies.set('token', token);
```

> Best security is **HttpOnly cookie** from backend.

---

# 8. Zustand Auth Store

`features/auth/store.js`

```js
import { create } from 'zustand';

export const useAuthStore = create((set) => ({
  user: null,
  isAuthenticated: false,

  login: (user) =>
    set({
      user,
      isAuthenticated: true,
    }),

  logout: () =>
    set({
      user: null,
      isAuthenticated: false,
    }),
}));
```

---

# 9. TanStack Query Setup

`lib/queryClient.js`

```js
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 1,
      staleTime: 1000 * 60 * 5,
      refetchOnWindowFocus: false,
    },
  },
});
```

---

# 10. Main Entry

`main.jsx`

```jsx
import ReactDOM from 'react-dom/client';
import { QueryClientProvider } from '@tanstack/react-query';
import { queryClient } from './lib/queryClient';
import { RouterProvider } from 'react-router-dom';
import router from './app/router';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')).render(
  <QueryClientProvider client={queryClient}>
    <RouterProvider router={router} />
  </QueryClientProvider>
);
```

---

# 11. React Router Structure

`app/router.jsx`

```jsx
import { createBrowserRouter } from 'react-router-dom';

import Login from '../features/auth/pages/Login';
import Dashboard from '../pages/Dashboard';
import UsersPage from '../features/users/pages/UsersPage';

const router = createBrowserRouter([
  { path: '/login', element: <Login /> },

  {
    path: '/',
    element: <ProtectedRoute />,
    children: [
      { index: true, element: <Dashboard /> },
      { path: 'users', element: <UsersPage /> },
      { path: 'products', element: <ProductsPage /> },
      { path: 'orders', element: <OrdersPage /> },
    ],
  },
]);

export default router;
```

---

# 12. Protected Route

```jsx
import { Navigate, Outlet } from 'react-router-dom';
import { useAuthStore } from '../features/auth/store';

export default function ProtectedRoute() {
  const isAuthenticated = useAuthStore((state) => state.isAuthenticated);

  return isAuthenticated ? <Outlet /> : <Navigate to='/login' replace />;
}
```

---

# 13. Login Form (React Hook Form + Zod)

`features/auth/schema.js`

```js
import { z } from 'zod';

export const loginSchema = z.object({
  email: z.email(),
  password: z.string().min(6),
});
```

`Login.jsx`

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { loginSchema } from '../schema';

const {
  register,
  handleSubmit,
  formState: { errors },
} = useForm({
  resolver: zodResolver(loginSchema),
});
```

---

# 14. CRUD Pattern for Users

## API

`features/users/api.js`

```js
import api from '../../lib/axios';

export const getUsers = () => api.get('/users');
export const createUser = (data) => api.post('/users', data);
export const updateUser = (id, data) => api.put(`/users/${id}`, data);
export const deleteUser = (id) => api.delete(`/users/${id}`);
```

## Hooks

`features/users/hooks.js`

```js
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

export const useUsers = () =>
  useQuery({
    queryKey: ['users'],
    queryFn: getUsers,
  });

export const useCreateUser = () => {
  const qc = useQueryClient();

  return useMutation({
    mutationFn: createUser,
    onSuccess: () => qc.invalidateQueries(['users']),
  });
};
```

Use same pattern for:

- Products
- Orders

---

# 15. Custom Hooks

## useDebounce.js

```js
import { useEffect, useState } from 'react';

export default function useDebounce(value, delay = 500) {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);

    return () => clearTimeout(id);
  }, [value]);

  return debounced;
}
```

## usePagination.js

Reusable pagination state.

## useModal.js

Reusable open/close modal logic.

---

# 16. Admin Panel Pages

## Sidebar

- Dashboard
- Users
- Products
- Orders
- Settings
- Logout

## Dashboard Widgets

- Total Users
- Total Revenue
- Pending Orders
- New Products

---

# 17. UI Components

`components/ui`

- Button
- Input
- Select
- Modal
- Table
- Badge
- Loader
- ConfirmDialog

---

# 18. Best CRUD UX Practices

### Users

- Search
- Pagination
- Add User
- Edit User
- Delete User
- Activate / Deactivate

### Products

- Name
- SKU
- Price
- Stock
- Image
- Category

### Orders

- Status
- Customer
- Payment
- Shipping
- Total

---

# 19. Query Keys Pattern

```js
['users'][('users', page)][('users', filters)]['products'][('orders', orderId)];
```

---

# 20. Security Best Practices

✅ Store JWT in HttpOnly cookie
✅ Auto logout on 401
✅ Role-based route guards
✅ Input validation with Zod
✅ Sanitize rich text
✅ CSRF protection (backend)

---

# 21. Performance Best Practices

✅ Lazy load routes
✅ Memoized tables
✅ Debounced search
✅ Pagination
✅ Query caching
✅ Skeleton loaders

---

# 22. Suggested Backend API

```http
POST /auth/login
POST /auth/logout
GET /me

GET /users
POST /users
PUT /users/:id
DELETE /users/:id

GET /products
POST /products
PUT /products/:id
DELETE /products/:id

GET /orders
PUT /orders/:id
DELETE /orders/:id
```

---

# 23. Real Enterprise Add-ons

Later add:

- Dark mode
- Role permissions
- Audit logs
- Export CSV
- Bulk delete
- Notifications
- Charts
- Multi-language

---

# 24. Final Recommendation

Use **feature-first architecture**, not file-type architecture.

Best for teams:

```bash
features/users
features/products
features/orders
features/auth
```

---
