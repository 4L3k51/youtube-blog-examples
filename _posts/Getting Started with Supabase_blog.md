# Getting Started with Supabase: Build a Modern Web App with Auth and Policies

Supabase is a developer-friendly platform built on top of Postgres, designed to give you all the building blocks needed for modern application development. This guide walks through the process of creating a simple web app that interacts with your Supabase database, manages authentication, and securely controls access with row-level security (RLS) policies.

---

## Overview

You'll learn how to:
- Set up a new Supabase project
- Create and manage tables in your database
- Connect your web application to Supabase using the client SDK
- Implement data fetching and inserting
- Secure your app with authentication
- Configure row-level security (RLS) for fine-grained access control

---

## Prerequisites

Before you begin, make sure you have:
- [Node.js](https://nodejs.org/) and npm installed
- A Supabase account ([sign up here](https://app.supabase.com))
- Basic familiarity with React and Vite (or adapt to your chosen framework)

---

## 1. Create a Supabase Project

1. Open [https://database.new](https://database.new) to create a new Supabase project.
2. Fill in the following:
    - **Project Name**
    - **Project Database Password**
    - **Select a Region** nearest your users
    - Ensure **Enable Data API** is **checked**
3. Click **Create** to set up your project.

---

## 2. Define Your First Table

1. In the **Supabase Dashboard**, open the **Table Editor**.
2. Click **New Table**.
3. Create a table called **posts** with these columns:
    - `content`: `text` (not nullable)
    - `is_published`: `boolean` (default `false`)
4. Save the table.
5. Insert your first row. Only `content` needs to be specified since `is_published` defaults to `false`.

---

## 3. Explore Data Access Methods

You can interact with your database in two main ways:
- **Table Editor:** Visual interface for managing tables and data.
- **SQL Editor:** Run SQL queries directly.

Example to fetch all posts:
```sql
SELECT * FROM posts;
```

---

## 4. Connect Your App to Supabase

You can connect to your Supabase Postgres with:
- The Supabase client SDK
- Standard Postgres connection strings for use with ORMs or other clients

This guide will use the Supabase JavaScript client for a React app.

### Setup Steps

1. In the Supabase Dashboard, click **Connect**.
2. Select your framework (e.g., **React + Vite**).
3. Copy the `.env` file config provided and add it to your app:
    ```
    VITE_SUPABASE_URL=https://<project-ref>.supabase.co
    VITE_SUPABASE_ANON_KEY=your-anon-key-here
    ```
4. Restart your development server to load environment variables.
5. Copy and create `supabase.ts` (or `supabase.js`) that initializes your client:
    ```typescript
    import { createClient } from '@supabase/supabase-js';

    const supabaseUrl = import.meta.env.VITE_SUPABASE_URL as string;
    const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY as string;

    export const supabase = createClient(supabaseUrl, supabaseAnonKey);
    ```
6. Install the Supabase JavaScript SDK:
    ```bash
    npm install @supabase/supabase-js
    ```

---

## 5. Basic Data Operations

Implement `select` and `insert` operations using the Supabase client.

### Selecting Data

```typescript
const { data, error } = await supabase
  .from('posts')
  .select('*');

if (error) {
  // Handle error
}
```

### Inserting Data

```typescript
const { data, error } = await supabase
  .from('posts')
  .insert([{ content: 'Your post here' }]);

if (error) {
  // Handle error
}
```

---

## 6. Row Level Security: Enable & Configure

By default, new Supabase tables have **Row Level Security (RLS)** enabled: all access is denied until policies are defined.

> **Note**  
> RLS is critical for securing your application when in production. Disabling it is only recommended during early development.

### To temporarily disable RLS for testing:

1. Go to the Table Editor.
2. Click **Add Policies**.
3. **Disable Row Level Security** for the table.

You'll now be able to select and insert rows freely.

---

## 7. Implement Authentication

Now secure your app by enabling auth and re-enabling RLS.

1. **Re-enable Row Level Security** for your table in production.
2. Set up authentication in your React app:

### Fetching Initial Session

```typescript
import { supabase } from './supabase';

supabase.auth.getSession().then(({ data: { session } }) => {
  if (session) {
    // Save session (e.g., setUser(session.user.email))
  }
});
```

### Monitoring Auth State

```typescript
import { useEffect } from 'react';
import { supabase } from './supabase';

useEffect(() => {
  const { data: authListener } = supabase.auth.onAuthStateChange((event, session) => {
    // Respond to auth state changes
  });
  return () => {
    authListener?.subscription.unsubscribe();
  };
}, []);
```

### Sign In / Sign Up / Sign Out

```typescript
// Sign In
await supabase.auth.signInWithPassword({ email, password });

// Sign Up
await supabase.auth.signUp({ email, password });

// Sign Out
await supabase.auth.signOut();
```

### Configure Auth Redirect URLs

1. Copy your dev server's URL (e.g., `http://localhost:5173`) 
2. In the Supabase Dashboard, go to **Authentication > URL Configuration**.
3. Override the **Site URL** with your app's URL.
4. Save changes.

> **Note**  
> Users must confirm their email via the confirmation link sent after signing up. Once confirmed, they are able to make authenticated API requests.

---

## 8. Authorization with Row-Based Policies

To enforce per-user data access, add a `user_id` column to your table and bind posts to users.

### Adding a User Reference to Posts

1. In Table Editor, add:
    - `user_id`: `uuid`
    - Default value: `auth.uid()` (a special Supabase function returning the current user's ID)
    - Set up a foreign key constraint pointing to `auth.users.id`

### Defining Authorization Policies

#### Select Policy

Allow users to select only their own posts:

```sql
-- Template: User can select rows where user_id matches their UID
(auth.uid() = user_id)
```

#### Insert Policy

Allow users to insert posts associated with themselves:

```sql
-- Template: User can insert row if user_id matches their UID
(auth.uid() = user_id)
```

Save these policies in the Table Editor under **Policies** for `SELECT` and `INSERT`.

After applying policies, test inserting and selecting posts; users should only be able to access rows associated with their own user ID.

---

## Conclusion & Next Steps

You now have a fully functional Supabase-powered application with secure authentication and fine-grained authorization. This foundation allows you to build out richer features such as file storage, subscriptions, and edge functions offered by Supabase.

Ready to expand your project? Explore:
- [Supabase Storage](https://supabase.com/docs/guides/storage)
- [Real-Time Subscriptions](https://supabase.com/docs/guides/realtime)
- [Edge Functions](https://supabase.com/docs/guides/functions)

Happy building!

---