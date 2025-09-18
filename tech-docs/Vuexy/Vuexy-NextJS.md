# Vuexy NextJS Admin Template - Complete Technical Documentation

## Table of Contents
1. [Overview](#overview)
2. [Installation & Setup](#installation--setup)
3. [Project Structure](#project-structure)
4. [Configuration](#configuration)
5. [Authentication & Security](#authentication--security)
6. [Components & UI](#components--ui)
7. [Menu & Navigation](#menu--navigation)
8. [Development Best Practices](#development-best-practices)
9. [Deployment & Production](#deployment--production)
10. [Integration Guide](#integration-guide)
11. [FAQ & Troubleshooting](#faq--troubleshooting)

---

## Overview

**Vuexy NextJS Admin Template** is a production-ready, extensively customizable MUI-based admin dashboard template built with Next.js 14+ App Router, Material-UI (MUI), and Tailwind CSS.

### Key Features
- **Next.js 14+ App Router** - Modern routing with app directory
- **TypeScript & JavaScript** - Full support for both languages
- **Material-UI (MUI)** - Complete component library
- **Tailwind CSS** - Utility-first CSS framework
- **NextAuth.js** - Authentication system
- **Prisma ORM** - Database management
- **React Hook Form** - Form validation
- **Internationalization** - Multi-language support
- **Responsive Design** - Mobile-first approach
- **Multiple Layouts** - Vertical, Horizontal, Collapsed
- **Dark/Light Mode** - Theme switching
- **Redux Toolkit** - State management

### Technical Stack
```json
{
  "framework": "Next.js 14+",
  "language": "TypeScript/JavaScript",
  "ui": "Material-UI (MUI)",
  "styling": "Tailwind CSS",
  "auth": "NextAuth.js",
  "database": "Prisma ORM",
  "forms": "React Hook Form",
  "state": "Redux Toolkit",
  "charts": "ApexCharts/Recharts",
  "deployment": "Vercel/Netlify compatible"
}
```

---

## Installation & Setup

### Prerequisites
- **Node.js** (LTS version recommended)
- **Package Manager**: pnpm (recommended), yarn, or npm
- **Database**: SQLite (default), PostgreSQL, MySQL, or MongoDB

### Step 1: Project Setup
```bash
# Download and extract Vuexy template
# Choose: typescript-version or javascript-version
# Choose: full-version or starter-kit

# Navigate to project directory
cd vuexy-nextjs-admin-template
```

### Step 2: Environment Configuration
```bash
# Copy environment file
cp .env.example .env

# Edit .env file with your configurations
```

**Required Environment Variables:**
```env
# Base Configuration
BASEPATH=
API_URL=http://localhost:3000${BASEPATH}/api
NEXT_PUBLIC_APP_URL=http://localhost:3000${BASEPATH}

# NextAuth Configuration
NEXTAUTH_BASEPATH=${BASEPATH}/api/auth
NEXTAUTH_URL=http://localhost:3000${BASEPATH}/api/auth
NEXTAUTH_SECRET=your-secret-key-here

# Google OAuth (optional)
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret

# Database Configuration
DATABASE_URL=file:./dev.db
```

### Step 3: Installation
```bash
# Install dependencies (pnpm recommended)
pnpm install

# Alternative package managers
yarn install
npm install
```

### Step 4: Launch Project
```bash
# Start development server
pnpm dev

# Alternative commands
yarn dev
npm run dev

# Custom port
pnpm dev --port 3001
```

Visit `http://localhost:3000` to see your application.

---

## Project Structure

### Core Directory Structure
```
.
├── public/                     # Static assets
├── src/
│   ├── @core/                  # 🚫 Core template files (do not modify)
│   ├── @layouts/               # 🚫 Layout components (do not modify)
│   ├── @menu/                  # 🚫 Menu components (do not modify)
│   ├── app/                    # App router pages
│   ├── assets/                 # Static assets (SVG, images)
│   ├── components/             # ✅ Your custom components
│   ├── configs/                # Configuration files
│   ├── contexts/               # React contexts
│   ├── data/                   # Static data files
│   ├── fake-db/                # Mock database (development)
│   ├── hocs/                   # Higher Order Components
│   ├── hooks/                  # Custom React hooks
│   ├── libs/                   # Third-party libraries
│   ├── prisma/                 # Database schema & migrations
│   ├── redux-store/            # Redux store configuration
│   ├── types/                  # TypeScript type definitions
│   ├── utils/                  # Utility functions
│   └── views/                  # Page components
├── .env.example                # Environment variables template
├── next.config.mjs             # Next.js configuration
├── package.json                # Dependencies and scripts
├── tailwind.config.ts          # Tailwind CSS configuration
└── tsconfig.json               # TypeScript configuration
```

### Key Folders Explained

#### 🚫 Core Folders (Do Not Modify)
- **`src/@core/`** - Template's core functionality
- **`src/@layouts/`** - Layout components (Vertical, Horizontal, Blank)
- **`src/@menu/`** - Menu components and logic

#### ✅ Customizable Folders
- **`src/components/`** - Your custom components
- **`src/app/`** - Your application pages
- **`src/views/`** - Page view components
- **`src/hooks/`** - Custom React hooks
- **`src/utils/`** - Utility functions
- **`src/contexts/`** - React contexts

#### App Router Structure
```
src/app/
├── [lang]/                     # Language-specific routes
│   ├── (blank-layout-pages)/   # Pages with blank layout
│   ├── (dashboard)/            # Main dashboard pages
│   ├── [...not-found]/         # 404 handling
│   └── layout.tsx              # Main layout wrapper
├── api/                        # API routes
│   ├── auth/                   # Authentication endpoints
│   └── login/                  # Login API
├── globals.css                 # Global styles
└── favicon.ico                 # App icon
```

---

## Configuration

### Theme Configuration (`src/configs/themeConfig.ts`)

```typescript
export const themeConfig = {
  // Template Settings
  templateName: 'Vuexy',
  homePageUrl: '/',
  settingsCookieName: 'vuexy-next-settings',
  
  // Appearance
  mode: 'light' | 'dark' | 'system',
  skin: 'default' | 'bordered',
  semiDark: false,
  
  // Layout
  layout: 'vertical' | 'horizontal' | 'collapsed',
  layoutPadding: 24,
  compactContentWidth: 1280,
  
  // Navigation
  navbar: {
    type: 'fixed' | 'static',
    contentWidth: 'compact' | 'wide',
    floating: true,
    detached: true,
    blur: true
  },
  
  // Content
  contentWidth: 'compact' | 'wide',
  
  // Footer
  footer: {
    type: 'fixed' | 'static',
    contentWidth: 'compact' | 'wide',
    detached: true
  },
  
  // UI Settings
  disableRipple: false,
  toastPosition: 'top-right'
}
```

### Next.js Configuration (`next.config.mjs`)

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Base path for deployment
  basePath: process.env.BASEPATH || '',
  
  // Enable experimental features
  experimental: {
    serverComponentsExternalPackages: ['@prisma/client']
  },
  
  // Image optimization
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '**',
      },
    ],
  },
  
  // TypeScript configuration
  typescript: {
    ignoreBuildErrors: false,
  },
  
  // ESLint configuration
  eslint: {
    ignoreDuringBuilds: false,
  },
}

export default nextConfig
```

### Tailwind Configuration (`tailwind.config.ts`)

```typescript
import type { Config } from 'tailwindcss'

const config: Config = {
  content: [
    './src/pages/**/*.{js,ts,jsx,tsx,mdx}',
    './src/components/**/*.{js,ts,jsx,tsx,mdx}',
    './src/app/**/*.{js,ts,jsx,tsx,mdx}',
    './src/@core/**/*.{js,ts,jsx,tsx,mdx}',
    './src/@layouts/**/*.{js,ts,jsx,tsx,mdx}',
    './src/@menu/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  darkMode: 'class',
  theme: {
    extend: {
      // Custom colors, spacing, etc.
    },
  },
  plugins: [
    // Tailwind plugins
  ],
}

export default config
```

---

## Authentication & Security

### NextAuth.js Setup (`src/libs/auth.ts`)

```typescript
import { NextAuthOptions } from 'next-auth'
import GoogleProvider from 'next-auth/providers/google'
import CredentialsProvider from 'next-auth/providers/credentials'
import { PrismaAdapter } from '@next-auth/prisma-adapter'
import { prisma } from '@/libs/prisma'

export const authOptions: NextAuthOptions = {
  adapter: PrismaAdapter(prisma),
  providers: [
    // Google OAuth
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
    
    // Credentials Provider
    CredentialsProvider({
      name: 'credentials',
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' }
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) {
          return null
        }
        
        // Implement your authentication logic
        const user = await authenticateUser(credentials.email, credentials.password)
        
        if (user) {
          return {
            id: user.id,
            email: user.email,
            name: user.name,
            role: user.role
          }
        }
        
        return null
      }
    })
  ],
  
  session: {
    strategy: 'jwt',
    maxAge: 30 * 24 * 60 * 60, // 30 days
  },
  
  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.role = user.role
      }
      return token
    },
    
    async session({ session, token }) {
      if (token) {
        session.user.id = token.sub
        session.user.role = token.role
      }
      return session
    }
  },
  
  pages: {
    signIn: '/login',
    error: '/login',
  },
  
  secret: process.env.NEXTAUTH_SECRET,
}
```

### Route Protection Middleware (`middleware.ts`)

```typescript
import { withAuth } from 'next-auth/middleware'

export default withAuth(
  function middleware(req) {
    // Add custom middleware logic here
  },
  {
    callbacks: {
      authorized: ({ token, req }) => {
        // Define authorization logic
        const { pathname } = req.nextUrl
        
        // Public routes
        if (pathname.startsWith('/login') || pathname.startsWith('/register')) {
          return true
        }
        
        // Protected routes require authentication
        return !!token
      },
    },
  }
)

export const config = {
  matcher: [
    // Match all routes except static files
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
}
```

### Database Schema (`prisma/schema.prisma`)

```prisma
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model User {
  id            String    @id @default(cuid())
  name          String?
  email         String    @unique
  emailVerified DateTime?
  image         String?
  role          String    @default("user")
  accounts      Account[]
  sessions      Session[]
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
}
```

---

## Components & UI

### Custom Component Structure

```typescript
// src/components/ui/CustomButton.tsx
import React from 'react'
import { Button, ButtonProps } from '@mui/material'
import { styled } from '@mui/material/styles'

interface CustomButtonProps extends ButtonProps {
  gradient?: boolean
  rounded?: boolean
}

const StyledButton = styled(Button)<CustomButtonProps>(({ theme, gradient, rounded }) => ({
  ...(gradient && {
    background: 'linear-gradient(45deg, #FE6B8B 30%, #FF8E53 90%)',
    '&:hover': {
      background: 'linear-gradient(45deg, #FE6B8B 60%, #FF8E53 100%)',
    },
  }),
  ...(rounded && {
    borderRadius: theme.shape.borderRadius * 3,
  }),
}))

const CustomButton: React.FC<CustomButtonProps> = ({ 
  children, 
  gradient = false, 
  rounded = false, 
  ...props 
}) => {
  return (
    <StyledButton 
      gradient={gradient} 
      rounded={rounded} 
      {...props}
    >
      {children}
    </StyledButton>
  )
}

export default CustomButton
```

### Form Component Example

```typescript
// src/components/forms/LoginForm.tsx
import React from 'react'
import { useForm, Controller } from 'react-hook-form'
import { yupResolver } from '@hookform/resolvers/yup'
import * as yup from 'yup'
import {
  Box,
  TextField,
  Button,
  Typography,
  Alert,
  IconButton,
  InputAdornment
} from '@mui/material'
import { Visibility, VisibilityOff } from '@mui/icons-material'

interface LoginFormData {
  email: string
  password: string
}

const schema = yup.object().shape({
  email: yup.string().email('Invalid email').required('Email is required'),
  password: yup.string().min(6, 'Password must be at least 6 characters').required('Password is required'),
})

interface LoginFormProps {
  onSubmit: (data: LoginFormData) => void
  loading?: boolean
  error?: string
}

const LoginForm: React.FC<LoginFormProps> = ({ onSubmit, loading, error }) => {
  const [showPassword, setShowPassword] = React.useState(false)
  
  const {
    control,
    handleSubmit,
    formState: { errors, isValid }
  } = useForm<LoginFormData>({
    resolver: yupResolver(schema),
    mode: 'onBlur'
  })

  const handleClickShowPassword = () => {
    setShowPassword(!showPassword)
  }

  return (
    <Box component="form" onSubmit={handleSubmit(onSubmit)} sx={{ width: '100%' }}>
      <Typography variant="h4" component="h1" gutterBottom>
        Login
      </Typography>
      
      {error && (
        <Alert severity="error" sx={{ mb: 2 }}>
          {error}
        </Alert>
      )}
      
      <Controller
        name="email"
        control={control}
        render={({ field }) => (
          <TextField
            {...field}
            label="Email"
            type="email"
            fullWidth
            margin="normal"
            error={!!errors.email}
            helperText={errors.email?.message}
          />
        )}
      />
      
      <Controller
        name="password"
        control={control}
        render={({ field }) => (
          <TextField
            {...field}
            label="Password"
            type={showPassword ? 'text' : 'password'}
            fullWidth
            margin="normal"
            error={!!errors.password}
            helperText={errors.password?.message}
            InputProps={{
              endAdornment: (
                <InputAdornment position="end">
                  <IconButton onClick={handleClickShowPassword} edge="end">
                    {showPassword ? <VisibilityOff /> : <Visibility />}
                  </IconButton>
                </InputAdornment>
              ),
            }}
          />
        )}
      />
      
      <Button
        type="submit"
        fullWidth
        variant="contained"
        disabled={!isValid || loading}
        sx={{ mt: 3, mb: 2 }}
      >
        {loading ? 'Logging in...' : 'Login'}
      </Button>
    </Box>
  )
}

export default LoginForm
```

### Custom Hooks Example

```typescript
// src/hooks/useLocalStorage.ts
import { useState, useEffect } from 'react'

export function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') {
      return initialValue
    }
    
    try {
      const item = window.localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error)
      return initialValue
    }
  })

  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value
      setStoredValue(valueToStore)
      
      if (typeof window !== 'undefined') {
        window.localStorage.setItem(key, JSON.stringify(valueToStore))
      }
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error)
    }
  }

  return [storedValue, setValue] as const
}
```

---

## Menu & Navigation

### Static Menu Configuration (`src/data/navigation/verticalMenuData.ts`)

```typescript
import { VerticalNavItemsType } from '@/@menu/types'

const verticalMenuData = (): VerticalNavItemsType => [
  {
    label: 'Dashboard',
    href: '/dashboard',
    icon: 'ri-home-smile-line',
  },
  {
    label: 'Users',
    icon: 'ri-user-line',
    children: [
      {
        label: 'List',
        href: '/users/list',
      },
      {
        label: 'View',
        href: '/users/view',
      },
    ],
  },
  {
    sectionTitle: 'Apps & Pages',
  },
  {
    label: 'Email',
    href: '/apps/email',
    icon: 'ri-mail-line',
  },
  {
    label: 'Chat',
    href: '/apps/chat',
    icon: 'ri-wechat-line',
  },
  {
    label: 'Calendar',
    href: '/apps/calendar',
    icon: 'ri-calendar-line',
  },
  {
    label: 'Invoice',
    icon: 'ri-file-list-line',
    children: [
      {
        label: 'List',
        href: '/apps/invoice/list',
      },
      {
        label: 'Preview',
        href: '/apps/invoice/preview',
      },
      {
        label: 'Edit',
        href: '/apps/invoice/edit',
      },
      {
        label: 'Add',
        href: '/apps/invoice/add',
      },
    ],
  },
]

export default verticalMenuData
```

### Dynamic Menu with API

```typescript
// src/components/menu/DynamicMenu.tsx
import React, { useState, useEffect } from 'react'
import { VerticalNavItemsType } from '@/@menu/types'

interface DynamicMenuProps {
  userId: string
}

const DynamicMenu: React.FC<DynamicMenuProps> = ({ userId }) => {
  const [menuItems, setMenuItems] = useState<VerticalNavItemsType>([])
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    const fetchMenuItems = async () => {
      try {
        const response = await fetch(`/api/menu?userId=${userId}`)
        const data = await response.json()
        setMenuItems(data.menuItems)
      } catch (error) {
        console.error('Error fetching menu items:', error)
      } finally {
        setLoading(false)
      }
    }

    fetchMenuItems()
  }, [userId])

  if (loading) {
    return <div>Loading menu...</div>
  }

  return (
    <div>
      {/* Render menu items */}
      {menuItems.map((item, index) => (
        <div key={index}>
          {/* Menu item rendering logic */}
        </div>
      ))}
    </div>
  )
}

export default DynamicMenu
```

### Menu API Route (`src/app/api/menu/route.ts`)

```typescript
import { NextRequest, NextResponse } from 'next/server'
import { getServerSession } from 'next-auth'
import { authOptions } from '@/libs/auth'

export async function GET(request: NextRequest) {
  try {
    const session = await getServerSession(authOptions)
    
    if (!session) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
    }

    const { searchParams } = new URL(request.url)
    const userId = searchParams.get('userId')
    
    // Fetch user-specific menu items from database
    const menuItems = await getUserMenuItems(userId)
    
    return NextResponse.json({ menuItems })
  } catch (error) {
    console.error('Menu API error:', error)
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 })
  }
}

async function getUserMenuItems(userId: string) {
  // Implementation to fetch menu items based on user role/permissions
  // This is a simplified example
  return [
    {
      label: 'Dashboard',
      href: '/dashboard',
      icon: 'ri-home-smile-line',
    },
    {
      label: 'Profile',
      href: '/profile',
      icon: 'ri-user-line',
    },
  ]
}
```

---

## Development Best Practices

### Code Organization
1. **Separate Concerns**: Keep business logic separate from UI components
2. **Custom Hooks**: Extract reusable logic into custom hooks
3. **Type Safety**: Use TypeScript for better development experience
4. **Error Boundaries**: Implement error boundaries for graceful error handling
5. **Performance**: Use React.memo, useMemo, and useCallback appropriately

### File Naming Conventions
```
components/
├── ui/              # Generic UI components
├── forms/           # Form components
├── layout/          # Layout-specific components
└── features/        # Feature-specific components

hooks/
├── api/             # API-related hooks
├── auth/            # Authentication hooks
└── ui/              # UI-related hooks

utils/
├── api.ts           # API utilities
├── auth.ts          # Authentication utilities
├── validation.ts    # Form validation schemas
└── helpers.ts       # General helper functions
```

### Error Handling

```typescript
// src/components/ErrorBoundary.tsx
import React from 'react'
import { Box, Typography, Button } from '@mui/material'

interface ErrorBoundaryProps {
  children: React.ReactNode
}

interface ErrorBoundaryState {
  hasError: boolean
  error?: Error
}

class ErrorBoundary extends React.Component<ErrorBoundaryProps, ErrorBoundaryState> {
  constructor(props: ErrorBoundaryProps) {
    super(props)
    this.state = { hasError: false }
  }

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error }
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('ErrorBoundary caught an error:', error, errorInfo)
  }

  render() {
    if (this.state.hasError) {
      return (
        <Box 
          display="flex" 
          flexDirection="column" 
          alignItems="center" 
          justifyContent="center" 
          minHeight="50vh"
        >
          <Typography variant="h4" gutterBottom>
            Something went wrong
          </Typography>
          <Typography variant="body1" color="text.secondary" gutterBottom>
            {this.state.error?.message || 'An unexpected error occurred'}
          </Typography>
          <Button 
            variant="contained" 
            onClick={() => window.location.reload()}
          >
            Reload Page
          </Button>
        </Box>
      )
    }

    return this.props.children
  }
}

export default ErrorBoundary
```

### API Integration

```typescript
// src/utils/api.ts
import axios, { AxiosInstance, AxiosRequestConfig } from 'axios'
import { getSession } from 'next-auth/react'

class ApiClient {
  private client: AxiosInstance

  constructor() {
    this.client = axios.create({
      baseURL: process.env.NEXT_PUBLIC_API_URL || '/api',
      timeout: 10000,
    })

    // Request interceptor
    this.client.interceptors.request.use(
      async (config) => {
        const session = await getSession()
        if (session?.accessToken) {
          config.headers.Authorization = `Bearer ${session.accessToken}`
        }
        return config
      },
      (error) => Promise.reject(error)
    )

    // Response interceptor
    this.client.interceptors.response.use(
      (response) => response,
      (error) => {
        if (error.response?.status === 401) {
          // Handle unauthorized error
          window.location.href = '/login'
        }
        return Promise.reject(error)
      }
    )
  }

  async get<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.get<T>(url, config)
    return response.data
  }

  async post<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.post<T>(url, data, config)
    return response.data
  }

  async put<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.put<T>(url, data, config)
    return response.data
  }

  async delete<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.delete<T>(url, config)
    return response.data
  }
}

export const apiClient = new ApiClient()
```

---

## Deployment & Production

### Build Process
```bash
# Build the application
pnpm build

# Start production server
pnpm start

# Export static files (if using static export)
pnpm export
```

### Environment Variables for Production
```env
# Production Environment
NODE_ENV=production
NEXTAUTH_URL=https://yourdomain.com
NEXTAUTH_SECRET=your-production-secret

# Database
DATABASE_URL=your-production-database-url

# OAuth
GOOGLE_CLIENT_ID=your-production-google-client-id
GOOGLE_CLIENT_SECRET=your-production-google-client-secret
```

### Vercel Deployment
```json
// vercel.json
{
  "framework": "nextjs",
  "buildCommand": "pnpm build",
  "devCommand": "pnpm dev",
  "installCommand": "pnpm install",
  "env": {
    "NEXTAUTH_SECRET": "@nextauth_secret",
    "DATABASE_URL": "@database_url"
  }
}
```

### Docker Deployment
```dockerfile
# Dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Install dependencies based on the preferred package manager
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then yarn global add pnpm && pnpm i --frozen-lockfile; \
  else echo "Lockfile not found." && exit 1; \
  fi

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Next.js collects completely anonymous telemetry data about general usage.
ENV NEXT_TELEMETRY_DISABLED 1

RUN yarn build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

# Automatically leverage output traces to reduce image size
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```

### Performance Optimization
```typescript
// next.config.mjs
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Enable gzip compression
  compress: true,
  
  // Optimize images
  images: {
    formats: ['image/webp', 'image/avif'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
  
  // Enable experimental features
  experimental: {
    optimizeCss: true,
    optimizeServerReact: true,
  },
  
  // Bundle analyzer
  webpack: (config, { isServer }) => {
    if (!isServer) {
      config.resolve.fallback = {
        ...config.resolve.fallback,
        fs: false,
      }
    }
    return config
  },
}

export default nextConfig
```

---

## Integration Guide

### Multi-Tenant Architecture Integration

```typescript
// src/utils/tenant.ts
import { headers } from 'next/headers'

export interface Tenant {
  id: string
  name: string
  subdomain: string
  customDomain?: string
  settings: {
    theme: string
    logo: string
    primaryColor: string
  }
}

export async function getCurrentTenant(): Promise<Tenant | null> {
  const headersList = headers()
  const host = headersList.get('host')
  
  if (!host) return null
  
  // Extract subdomain or use custom domain
  const subdomain = host.split('.')[0]
  
  // Fetch tenant from database
  const tenant = await fetchTenantBySubdomain(subdomain)
  
  return tenant
}

async function fetchTenantBySubdomain(subdomain: string): Promise<Tenant | null> {
  // Implementation to fetch tenant from database
  // This would typically use your database client (Prisma, etc.)
  return null
}
```

### WhatsApp Integration

```typescript
// src/services/whatsapp.ts
import { apiClient } from '@/utils/api'

export interface WhatsAppMessage {
  to: string
  type: 'text' | 'template' | 'media'
  content: string
  templateName?: string
  mediaUrl?: string
}

export class WhatsAppService {
  private baseUrl = process.env.WHATSAPP_API_URL
  private accessToken = process.env.WHATSAPP_ACCESS_TOKEN

  async sendMessage(message: WhatsAppMessage): Promise<any> {
    try {
      const response = await apiClient.post('/whatsapp/messages', message)
      return response
    } catch (error) {
      console.error('WhatsApp message sending failed:', error)
      throw error
    }
  }

  async getTemplates(): Promise<any> {
    try {
      const response = await apiClient.get('/whatsapp/templates')
      return response
    } catch (error) {
      console.error('Failed to fetch WhatsApp templates:', error)
      throw error
    }
  }

  async handleWebhook(payload: any): Promise<void> {
    // Handle incoming WhatsApp webhooks
    console.log('Received WhatsApp webhook:', payload)
  }
}

export const whatsAppService = new WhatsAppService()
```

### Facebook Business API Integration

```typescript
// src/services/facebook.ts
import { apiClient } from '@/utils/api'

export interface FacebookInsights {
  pageId: string
  metrics: string[]
  period: 'day' | 'week' | 'days_28' | 'month' | 'lifetime'
  since?: string
  until?: string
}

export class FacebookService {
  private accessToken = process.env.FACEBOOK_ACCESS_TOKEN

  async getPageInsights(params: FacebookInsights): Promise<any> {
    try {
      const response = await apiClient.get('/facebook/insights', { params })
      return response
    } catch (error) {
      console.error('Facebook insights fetch failed:', error)
      throw error
    }
  }

  async getCampaigns(adAccountId: string): Promise<any> {
    try {
      const response = await apiClient.get(`/facebook/campaigns/${adAccountId}`)
      return response
    } catch (error) {
      console.error('Facebook campaigns fetch failed:', error)
      throw error
    }
  }
}

export const facebookService = new FacebookService()
```

---

## FAQ & Troubleshooting

### Common Issues

#### 1. Installation Errors
**Problem**: `npm install` fails with permission errors
**Solution**: 
```bash
# Use yarn or pnpm instead
yarn install
# or
pnpm install

# Clear npm cache
npm cache clean --force
```

#### 2. TypeScript Errors
**Problem**: TypeScript compilation errors
**Solution**:
```bash
# Check TypeScript configuration
npx tsc --noEmit

# Update types
npm install --save-dev @types/node @types/react @types/react-dom
```

#### 3. Authentication Issues
**Problem**: NextAuth.js not working
**Solution**:
```bash
# Check environment variables
echo $NEXTAUTH_SECRET
echo $NEXTAUTH_URL

# Generate new secret
openssl rand -base64 32
```

#### 4. Database Connection Issues
**Problem**: Prisma client not working
**Solution**:
```bash
# Generate Prisma client
npx prisma generate

# Push database schema
npx prisma db push

# Reset database
npx prisma db reset
```

#### 5. Build Failures
**Problem**: `next build` fails
**Solution**:
```bash
# Clean .next directory
rm -rf .next

# Clear node_modules
rm -rf node_modules package-lock.json
npm install

# Check for type errors
npm run type-check
```

### Performance Optimization Tips

1. **Code Splitting**: Use dynamic imports for large components
2. **Image Optimization**: Use Next.js Image component
3. **Bundle Analysis**: Use `@next/bundle-analyzer`
4. **Caching**: Implement proper caching strategies
5. **Database Optimization**: Use database indexes and connection pooling

### Security Best Practices

1. **Environment Variables**: Never commit sensitive data
2. **Authentication**: Use secure session management
3. **Input Validation**: Validate all user inputs
4. **HTTPS**: Always use HTTPS in production
5. **Dependencies**: Keep dependencies updated

### Migration Guide

#### From Starter Kit to Full Version
```bash
# Copy required files from full version
cp -r full-version/src/components/search starter-kit/src/components/
cp -r full-version/src/data/searchData.ts starter-kit/src/data/

# Install additional dependencies
npm install cmdk
```

#### From V1 to V2
```bash
# Update package.json
# Update Next.js configuration
# Update authentication setup
# Update database schema
```

---

## Conclusion

This comprehensive guide covers all aspects of Vuexy NextJS admin template integration and development. The template provides a robust foundation for building modern admin dashboards with authentication, responsive design, and extensive customization options.

### Key Takeaways:
- **Modular Architecture**: Separate core files from customizable components
- **Type Safety**: Full TypeScript support for better development experience
- **Authentication**: Built-in NextAuth.js integration with multiple providers
- **Performance**: Optimized for production deployment
- **Scalability**: Supports multi-tenant and enterprise-level applications

### Next Steps:
1. Set up your development environment
2. Configure authentication and database
3. Customize theme and layout
4. Implement your business logic
5. Deploy to production

For additional support and updates, refer to the official Vuexy documentation and community resources.

---

*Last updated: December 2024*
*Version: NextJS 14+ App Router*
*Template: Vuexy Admin Dashboard*
