# Getting Started with Supabase: A Comprehensive Guide with React & Vite

Supabase is a Postgres-powered development platform that provides all the essential services you need to build a modern, scalable application. In this guide, you'll learn how to spin up a new Supabase project, manage your database, integrate Supabase with a React + Vite web app, enable authentication, and implement row-level authorization.

---

## Overview

We'll walk through:

- Spinning up a new Supabase project
- Creating and interacting with database tables
- Connecting a React + Vite app to Supabase
- Adding authentication (sign-up/sign-in/sign-out)
- Securing data with row-level security and authorization policies

---

## Prerequisites

Make sure you have the following ready:

- Node.js and npm/yarn installed
- [Vite](https://vitejs.dev/) + [React](https://react.dev/) project scaffolding (or follow along with your preferred frontend setup)
- A free Supabase account (sign up at [supabase.com](https://supabase.com))
- Familiarity with JavaScript/TypeScript and React basics

---

## 1. Create a Supabase Project

1. Open [database.new](https://database.new), which redirects to the Supabase project creation page.
2. Fill in the following fields:
    - **Project Name:** Choose a descriptive name.
    - **Database Password:** Pick a secure password (save this for later).
    - **Region:** Select a region close to your users for best performance.
    - **Enable Data API:** _Ensure this box is checked._
3. Click **Create**.

---

## 2. Define Your Database Schema

### Create a Posts Table

1. Inside your Supabase dashboard, open the **Table Editor**.
2. Click **New Table** and configure the following columns:
    - **id:** (default primary key, usually auto-generated)
    - **content:** Type `Text`. (Set `Is Nullable` to **false**; content must be present.)
    - **is_published:** Type `Boolean`. (Default value: `false`.)
3. Save the new table.

### Insert an Initial Row

1. With your table selected, choose **Insert Row**.
2. Enter a value for the `content` column.
3. Save the row (other columns like `is_published` will use their default values).

---

## 3. Explore the SQL Editor

Supabase provides a SQL (SEEK) Editor to run raw SQL queries.

- Run a query like the following to see all rows in your table:

    ```sql
    SELECT * FROM posts;
    ```

You can interact with your data via the Table Editor's UI or by writing SQL directly in the SQL Editor.

---

## 4. Connect Your Supabase Project to Your App

Supabase projects are standard PostgreSQL databases; you can connect via traditional Postgres tooling, ORMs, or the official Client libraries (SDK). This guide uses the Supabase JavaScript Client.

### 1. Obtain Connection Details

1. In the Supabase dashboard, click the **Connect** button.
2. Select your frontend framework; here, we choose **React with Vite**.
3. You'll receive necessary config snippets for your project.

### 2. Set Up Environment & Client Files

1. **Create a `.env` file** at the root of your Vite project with the following content (replace with your project’s values):

    ```env
    VITE_SUPABASE_URL=https://<project-ref>.supabase.co
    VITE_SUPABASE_ANON_KEY=your-anon-key-here
    ```

2. Restart your development server to load the new environment variables.

3. **Create `supabase.ts` in your `src/` directory:** 

    ```typescript
    import { createClient } from '@supabase/supabase-js';

    const supabaseUrl = import.meta.env.VITE_SUPABASE_URL as string;
    const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY as string;

    export const supabase = createClient(supabaseUrl, supabaseAnonKey);
    ```

4. **Install the Supabase SDK:**

    ```sh
    npm install @supabase/supabase-js
    # or
    yarn add @supabase/supabase-js
    ```

---

## 5. Reading and Inserting Data from Your App

### Fetch Posts from the Database

1. Update your fetch/select logic:

    ```typescript
    import { supabase } from './supabase';

    async function handleSelect() {
      const { data, error } = await supabase.from('posts').select();
      if (error) {
        // Handle error
        console.error(error);
      } else {
        // Use your data (e.g., set state)
        console.log(data);
      }
    }
    ```

### Insert a New Post

1. Add insertion logic:

    ```typescript
    async function handleInsert(content: string) {
      const { data, error } = await supabase.from('posts').insert([{ content }]);
      if (error) {
        console.error(error);
      } else {
        console.log('Inserted:', data);
      }
    }
    ```

---

## 6. Enable Access: Row-Level Security (RLS)

By default, Supabase enables Row-Level Security (RLS) on new tables, blocking all unauthorized access.

> **Note:** For development only, you can temporarily disable RLS. **Always re-enable RLS before production deployment!**

### Temporarily Disabling RLS

1. In the Table Editor, select your table.
2. Click **Add Policies**.
3. Choose the option to **disable Row-Level Security**.

Now, your API requests (select/insert) will succeed without authorization errors.

---

## 7. Add Authentication to Your App

Re-enable RLS before deploying. Then, integrate authentication into your app.

### 1. Enable RLS Again

- In the Table Editor, re-enable Row-Level Security for your table.

### 2. Add Sign-In, Sign-Up, Sign-Out to React

#### Fetch Initial Auth Session

```typescript
import { supabase } from './supabase';
import { useEffect, useState } from 'react';

function useSupabaseSession() {
  const [email, setEmail] = useState<string | null>(null);

  useEffect(() => {
    let active = true;
    supabase.auth.getSession().then(({ data }) => {
      if (active && data.session?.user?.email) setEmail(data.session.user.email);
    });

    const { data: listener } = supabase.auth.onAuthStateChange((event, session) => {
      setEmail(session?.user?.email ?? null);
    });

    return () => {
      listener.subscription.unsubscribe();
      active = false;
    };
  }, []);

  return email;
}
```

#### Handle Sign-In and Sign-Up

```typescript
// Sign In with email/password
async function handleSignIn(email: string, password: string) {
  const { error } = await supabase.auth.signInWithPassword({ email, password });
  if (error) {
    // Handle error
  }
}

// Sign Up with email/password
async function handleSignUp(email: string, password: string) {
  const { error } = await supabase.auth.signUp({ email, password });
  if (error) {
    // Handle error
  }
}

// Sign out
async function handleSignOut() {
  await supabase.auth.signOut();
}
```

### 3. Configure Confirmation Email Redirect

To ensure users are redirected to your app after confirming their email:

1. Copy your local dev server’s URL (e.g., `http://localhost:5173`).
2. In the Supabase dashboard, go to **Authentication > URL Configuration**.
3. Set the **Site URL** to your app’s URL.
4. Save changes.

When users sign up, they’ll receive a confirmation email with a link directing them back to your application.

---

## 8. Authorization: Restrict Data per User

To ensure users can only access or insert their own data, set up row-level authorization.

### 1. Add a `user_id` Column to Your Table

1. In **Table Editor**, add a new column:
    - **user_id:** Type `UUID`
    - **Default value:** `auth.uid()`
2. Set up a foreign key reference:
    - Reference `auth.users.id`

This column will automatically store the authenticated user’s UUID for each new row.

### 2. Define Select and Insert Policies

1. In **Table Editor**, go to your table’s policies and create a new policy for `SELECT`:
    - Use the template:  
      _"Users can view rows where their user_id matches the row's user_id."_

      ```sql
      auth.uid() = user_id
      ```

2. Create a policy for `INSERT`:
    - Use the template:  
      _"Users can insert rows only if their auth.uid() matches user_id."_

      ```sql
      auth.uid() = user_id
      ```

With these in place, users can only interact with rows that belong to their own user id.

---

> **Note:** The special Postgres function `auth.uid()` is injected by Supabase, pulling the user ID from the JWT token, making it secure. End-users cannot spoof this value.

---

## 9. Confirm Everything Works

- Try signing up, confirming your email, and inserting a new post.
- Only the authenticated user's posts should be accessible.

---

## Conclusion & Next Steps

You've now built a basic full-stack app with Supabase, React, and Vite, featuring a secure database, fine-grained authorization, and authentication.

Supabase offers additional features such as Storage, Realtime APIs, and Edge Functions to further extend your app’s capabilities.

> **What would you like to learn next?** Explore Supabase’s docs or let us know which tutorials you'd like to see!

---