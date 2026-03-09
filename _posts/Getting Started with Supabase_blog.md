# Getting Started with Supabase: The Modern Postgres Development Toolkit

Supabase provides a comprehensive Postgres-based platform for rapidly building modern, scalable applications. This guide will walk through setting up a new project, creating your first table, integrating it with a web app (using React + Vite as an example), enabling authentication, and adding basic authorization.

## Overview

We'll cover:

- Creating a Supabase project and your first database table
- Connecting a web app to Supabase
- Fetching and inserting data from your app
- Implementing authentication (sign up, log in, log out)
- Setting up row-level security and authorization policies

By the end, you'll have a basic but secure full-stack app, powered by Supabase and Postgres.

---

## Prerequisites

- [Node.js](https://nodejs.org/) and [npm](https://www.npmjs.com/)
- Basic knowledge of JavaScript and React (examples use React with Vite)
- A [Supabase](https://supabase.com/) account

---

## 1. Create a Supabase Project

1. Go to [database.new](https://database.new/) to start a new Supabase project.
2. Fill in the following details:
   - **Project Name:** Choose something memorable.
   - **Database Password:** Set a secure password for your database.
   - **Region:** Select a region close to your users.
   - **Enable Data API:** Make sure this box is checked.
3. Click **Create** to provision your project.

---

## 2. Create Your First Table

1. Open the **Table Editor** in the Supabase dashboard.
2. Click **New Table** and fill out the details:
   - **Table Name:** `posts`
   - **Columns:**
     - `content` (type: `text`, not nullable) &mdash; to hold post content.
     - `is_published` (type: `boolean`, default: `false`) &mdash; stores published state.
3. Save the table.

To insert your first row:

- Add a value to the `content` column. The `is_published` column will default to `false`.

---

## 3. Interact with the Database

Supabase provides two ways to work with your database:

- **Table Editor:** For visual editing and managing data.
- **SQL Editor:** For executing raw SQL queries.

Example SQL to select all posts:

```sql
SELECT * FROM posts;
```

---

## 4. Connect Supabase to Your Web App

Supabase is built on standard Postgres, so you can connect using Postgres drivers, ORMs, or the Supabase Client SDK. 

This example uses the Supabase Client SDK in a React + Vite app.

### Step 4.1: Set Up Environment Variables

1. In the Supabase dashboard, go to the **Connect** section and select your framework (**React**).
2. Copy the provided `.env` snippet.
3. Create a `.env` file at your project's root and paste the contents.

**Remember to restart your dev server after adding environment variables.**

### Step 4.2: Set Up the Supabase Client

1. Copy the sample `supabase.ts` file from the dashboard.
2. In your project, create `supabase.ts` with the provided content:

```typescript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY;

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

### Step 4.3: Install the Supabase SDK

```bash
npm install @supabase/supabase-js
```

---

## 5. Fetching and Inserting Data

### Step 5.1: Fetch Data from the `posts` Table

Update your `handleSelect` function as follows:

```typescript
// Example using async/await and supabase client
const handleSelect = async () => {
  const { data, error } = await supabase
    .from('posts')
    .select('*');

  if (error) {
    // handle error
  } else {
    // use the data
  }
};
```

### Step 5.2: Insert New Posts

Add an insert function:

```typescript
const handleInsert = async (content: string) => {
  const { data, error } = await supabase
    .from('posts')
    .insert([{ content }]);
  
  if (error) {
    // handle error
  } else {
    // row inserted
  }
};
```

---

## 6. Dealing with Row Level Security (RLS)

By default, Supabase enables Row Level Security (RLS) on new tables, blocking data access until policies are defined.

> **Note:**  
> Turn off RLS **only for local development**. Always enable RLS for production and define appropriate policies.

To temporarily disable RLS:

1. Go to the **Table Editor**.
2. Select your table.
3. Click **Add Policy**.
4. Choose to **disable RLS** for open access.

Test your select and insert functions—data should now be accessible.

---

## 7. Setting Up Authentication

Re-enable RLS before adding authentication for proper security.

### Step 7.1: Enable RLS

1. In the Table Editor, select the table.
2. Enable Row Level Security.

### Step 7.2: Add Authentication to Your App

Implement login, signup, and logout:

#### Fetch the Initial Session

```typescript
// On component mount:
supabase.auth.getSession().then(({ data }) => {
  // Store session or user email in local state
});
```

#### Subscribe to Auth State Change Events

```typescript
import { useEffect } from 'react';

useEffect(() => {
  const { data: authListener } = supabase.auth.onAuthStateChange((event, session) => {
    // Update your app state accordingly
  });

  return () => {
    authListener?.unsubscribe();
  };
}, []);
```

#### Implement Sign In, Sign Up, and Sign Out

```typescript
// Sign in
await supabase.auth.signInWithPassword({ email, password });

// Sign up
await supabase.auth.signUp({ email, password });

// Sign out
await supabase.auth.signOut();
```

> **Note:**  
> When a user signs up, they receive a confirmation email.  
> To ensure users are redirected to your app after confirming, set your app's URL in the Supabase dashboard.

### Step 7.3: Configure Authentication Redirects

1. Copy your development app URL.
2. Go to your Supabase project's dashboard > **Authentication** > **URL Configuration**.
3. Update the **Site URL** with your app's URL and save.

After signing up, users must confirm their email by clicking the confirmation link in their inbox. Once confirmed, you’ll see their status updated in the dashboard.

---

## 8. Authorizing Users with Policies

To restrict data access so users can only see or insert their own posts:

### Step 8.1: Add a `user_id` Column

1. Add a new column to the `posts` table:
   - **Column name:** `user_id`
   - **Type:** `uuid`
   - **Default value:** `auth.uid()` (returns the authenticated user’s ID)
   - **Foreign key:** Reference `auth.users.id`

### Step 8.2: Populate `user_id` In Existing Rows

Edit a row to include a valid user ID as needed.

### Step 8.3: Define Policies

Supabase provides policy templates. For each action (select, insert):

- **Select Policy:** Only allow users to select rows where `user_id` matches their own ID.

  ```sql
  USING (auth.uid() = user_id)
  ```

- **Insert Policy:** Only allow users to insert rows where `user_id` matches their own ID.

  ```sql
  WITH CHECK (auth.uid() = user_id)
  ```

Save your policies.

Now, in the app, users will only be able to read or write their own posts.

---

## Conclusion & Next Steps

You've now:

- Created a secure, connected Supabase project
- Built a database schema and integrated it with a web app
- Implemented authentication and authorization

Supabase also offers features like Storage, Realtime, and Edge Functions.  
For deeper dives into these services and more advanced integrations, check out the [Supabase documentation](https://supabase.com/docs).

Ready to take your app further? Experiment by:

- Building out your UI
- Adding Storage for user-uploaded assets
- Exploring Realtime or Edge Functions for advanced backend logic

Happy building!