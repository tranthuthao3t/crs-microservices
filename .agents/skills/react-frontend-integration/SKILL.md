---
name: react-frontend-integration
description: >-
  Provides guidelines, API client setup, authentication handling, and state management rules
  for building the CRS React/Vite/Next.js frontend application.
---

# React Frontend Integration Guide

Use this skill when building UI components, setting up API interaction with the Gateway, handling JWT authentication, or managing application state in `crs-frontend`.

## API Client Architecture

All HTTP requests MUST pass through an Axios/Fetch client instance configured with base URL pointing to the API Gateway (`http://localhost:8080`).

### Standard Axios Interceptor Setup

```typescript
import axios from 'axios';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:8080/api',
  headers: {
    'Content-Type': 'application/json',
  },
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

## Recommended Folder Structure

```text
src/
├── api/                # API client functions (authApi, courseApi, registrationApi)
├── assets/             # Images, icons, static assets
├── components/         # Reusable UI components (Navbar, Modal, Table, Button)
├── context/            # AuthContext, ThemeContext
├── hooks/              # Custom React hooks (useAuth, useCourses)
├── pages/              # Page views (LoginPage, CourseListPage, RegistrationPage)
├── routes/             # App routing & ProtectedRoute wrappers
├── types/              # TypeScript interfaces/types
└── utils/              # Helper functions, formatters
```

## Security & Protected Routes

1. Store JWT token securely in memory or `localStorage`.
2. Wrap protected routes using a `ProtectedRoute` component that checks user role (`STUDENT`, `ADMIN`).
