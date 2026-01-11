# CRUD Template with Supabase + React + Vite

A modern, full-stack CRUD application template built with TypeScript, React, Vite, Supabase, and Tailwind CSS.

## Features

- **TypeScript** - Type-safe development
- **React 18** - Modern React with hooks
- **Vite** - Lightning fast build tool
- **Supabase** - Backend as a Service with PostgreSQL
- **Authentication** - Email/password authentication with Supabase Auth
- **React Router** - Client-side routing
- **Tailwind CSS** - Utility-first CSS framework
- **Zustand** - Lightweight state management
- **Generic CRUD Hooks** - Reusable hooks for database operations
- **Dark Mode** - Built-in dark mode support

## Project Structure

```
js-crud-template/
├── src/
│   ├── components/
│   │   ├── auth/          # Authentication components
│   │   ├── crud/          # CRUD components
│   │   └── layout/        # Layout components
│   ├── contexts/          # React contexts
│   ├── hooks/             # Custom React hooks
│   ├── lib/               # Library configurations
│   ├── pages/             # Page components
│   ├── stores/            # Zustand stores
│   ├── styles/            # CSS styles
│   └── types/             # TypeScript type definitions
├── public/                # Static assets
└── ...config files
```

## Getting Started

### Prerequisites

- Node.js (v18 or higher)
- npm or yarn
- A Supabase account ([sign up here](https://supabase.com))

### 1. Clone the repository

```bash
git clone <your-repo-url>
cd js-crud-template
```

### 2. Install dependencies

```bash
npm install
```

### 3. Set up Supabase

1. Create a new project in [Supabase](https://app.supabase.com)
2. Go to Project Settings > API
3. Copy your project URL and anon/public key

### 4. Configure environment variables

Create a `.env` file in the root directory:

```bash
cp .env.example .env
```

Update the `.env` file with your Supabase credentials:

```
VITE_SUPABASE_URL=your_supabase_project_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
```

### 5. Set up database tables

Run this SQL in your Supabase SQL Editor:

```sql
-- Create items table
CREATE TABLE items (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc'::text, NOW()) NOT NULL,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc'::text, NOW()) NOT NULL,
  title TEXT NOT NULL,
  description TEXT,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL
);

-- Enable Row Level Security
ALTER TABLE items ENABLE ROW LEVEL SECURITY;

-- Create policies
CREATE POLICY "Users can view their own items"
  ON items FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can create their own items"
  ON items FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update their own items"
  ON items FOR UPDATE
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can delete their own items"
  ON items FOR DELETE
  USING (auth.uid() = user_id);

-- Create updated_at trigger
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_items_updated_at BEFORE UPDATE ON items
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### 6. Run the development server

```bash
npm run dev
```

Open [http://localhost:5173](http://localhost:5173) to view the app.

## Usage

### Authentication

The template includes a complete authentication system:

- Sign up with email/password
- Sign in with existing account
- Sign out
- Protected routes

### CRUD Operations

The `useCrud` hook provides generic CRUD operations:

```typescript
import { useCrud } from '@/hooks/useCrud'

const { data, loading, error, create, update, remove, refetch } = useCrud<YourType>({
  tableName: 'your_table',
  orderBy: 'created_at',
  ascending: false,
  filter: { user_id: user?.id },
})
```

### State Management

Zustand is used for global state management. Example:

```typescript
import { create } from 'zustand'

interface YourState {
  value: string
  setValue: (value: string) => void
}

export const useYourStore = create<YourState>((set) => ({
  value: '',
  setValue: (value) => set({ value }),
}))
```

## Customization

### Adding New Tables

1. Create the table in Supabase SQL Editor
2. Update `src/types/database.ts` with the new table schema
3. Create components and use the `useCrud` hook

### Updating Database Types

After modifying your Supabase schema, update the types in:
- `src/types/database.ts` - Main database schema
- `src/types/index.ts` - Helper types

## Build

Build the app for production:

```bash
npm run build
```

Preview the production build:

```bash
npm run preview
```

## Deployment

This template can be deployed to various platforms:

- **Vercel** - Recommended for React apps
- **Netlify** - Great for static sites
- **Cloudflare Pages** - Fast global deployment

### Deploy to Vercel

```bash
npm install -g vercel
vercel
```

Make sure to add your environment variables in the Vercel dashboard.

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.