# Getting Started with Supabase: From Database to Auth in a Modern Web App

Supabase is a Postgres development platform offering all the building blocks you need to create modern, scalable applications. It exposes a powerful dashboard and a versatile JavaScript client, enabling rapid development without sacrificing flexibility or control.

In this guide, you'll learn how to:
- Set up a Supabase project and database
- Connect a React/Vite web app to Supabase
- Interact with your database securely
- Integrate authentication (sign-up, login)
- Implement fine-grained authorization with Postgres Row Level Security (RLS)

---

## Prerequisites

- Node.js and npm installed
- Basic familiarity with React and Vite
- A Supabase account ([sign up for free](https://supabase.com/))
- (Optional) An email account for testing auth confirmation

---

## Overview

You'll create a simple blog-style app with:
- A `posts` table (with content and publish status)
- User authentication (sign up, sign in, sign out)
- Row-level security ensuring users only see and insert their own posts

---

## 1. Create a Supabase Project

### Steps

1. Visit [https://database.new](https://database.new).
2. Enter your project details:
   - **Name:** Choose any project name.
   - **Database Password:** Pick a secure password.
   - **Region:** Select one geographically close to your users.
   - **Enable Data API:** Ensure this is checked.
3. Click **Create**.

---

## 2. Define Your First Table

### Steps

1. In the Supabase dashboard, go to the **Table Editor**.
2. Click **New Table** and enter:
   - **Name:** `posts`
   - **Columns:**
     - `content` (type: `text`, not nullable)
     - `is_published` (type: `boolean`, default: `false`)
3. Save the table.
4. Insert a test row with some text in `content`; `is_published` can be left as is (it defaults to `false`).

---

## 3. Querying Your Data

Supabase offers two primary ways to interact with your database:

- **Table Editor:** Visual CRUD.
- **SQL Editor:** Raw SQL commands.

For example, you can run:
```sql
SELECT * FROM posts;
```

---

## 4. Connecting Supabase to Your React/Vite App

### Steps

1. In the Supabase dashboard, find the **Connect** button.
2. Select **React** (with Vite).
3. Create the following files in your project root:

   - **`.env`** (environment variables):
     ```env
     VITE_SUPABASE_URL=https://<project-ref>.supabase.co
     VITE_SUPABASE_ANON_KEY=<anon-key-here>
     ```
     Replace `<project-ref>` and `<anon-key-here>` with values from your dashboard.

   - **`supabase.ts`** (client initialization):
     ```ts
     import { createClient } from '@supabase/supabase-js'

     const supabaseUrl = import.meta.env.VITE_SUPABASE_URL as string;
     const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY as string;

     export const supabase = createClient(supabaseUrl, supabaseAnonKey);
     ```

4. Restart your dev server to pick up the new `.env` variables.
5. Install the Supabase JavaScript SDK:
   ```
   npm install @supabase/supabase-js
   ```

---

## 5. Fetching and Inserting Data

Update your event handlers to interact with the `posts` table.

- **Fetching posts:**
  ```ts
  const { data, error } = await supabase
    .from('posts')
    .select('*');
  ```

- **Inserting a new post:**
  ```ts
  const { data, error } = await supabase
    .from('posts')
    .insert([{ content: 'your post content' }]);
  ```

---

## 6. Understanding Row Level Security (RLS)

By default, Supabase enabling RLS means your table actions are **blocked** unless you define explicit policies.

> **Note:**  
> While you may *temporarily* disable RLS during local development for convenience, **RLS should always be enabled in production** and you must write appropriate access policies.

**To disable RLS temporarily:**
1. Go to the Table Editor, select your table.
2. Click **Add Policies**, then **Disable RLS**.

When RLS is disabled, all actions are allowed.

---

## 7. Re-enabling RLS & Adding Auth

Once ready to secure your app:
1. Re-enable RLS on your table.
2. Add authentication to your frontend:
   - Fetch the initial session with:
     ```ts
     const { data, error } = await supabase.auth.getSession();
     ```
   - Listen for auth state changes:
     ```ts
     const { data: { subscription } } = supabase.auth.onAuthStateChange((event, session) => {
       // update your local session state here
     });

     // Optionally, clean up:
     return () => {
       subscription.unsubscribe();
     };
     ```

- **Sign in:**
  ```ts
  await supabase.auth.signInWithPassword({ email, password });
  ```
- **Sign up:**
  ```ts
  await supabase.auth.signUp({ email, password });
  ```
- **Sign out:**
  ```ts
  await supabase.auth.signOut();
  ```

> **Gotcha:**  
> Supabase will email a confirmation link on sign-up. To ensure users are redirected back to your app after confirmation, set the correct **Site URL** in your Supabase project's Auth > URL Configuration.
> 1. Copy your dev site URL.
> 2. Paste it into the **Site URL** field.
> 3. Save changes.

You can verify users in the dashboard under the **Users** tab.

---

## 8. Adding User-Based Row Level Authorization

To ensure users can **only access their own posts**, tie each post to a user:

### Steps

1. Add a new column to `posts`:
   - **Name:** `user_id`
   - **Type:** `uuid`
   - **Default Value:** `auth.uid()`
2. Set a foreign key relationship to `auth.users.id`.
3. For policy-based access:  
   - **Select Policy:** Only allow `auth.uid() = user_id`.
   - **Insert Policy:** Only allow insert if `user_id = auth.uid()`.
   - Use Supabase's policy templates for convenience.

**Typical select policy SQL:**
```sql
auth.uid() = user_id
```

Your users now only see or insert their own rows.

---

## Conclusion & Next Steps

You've built a secure, authenticated web app on Supabase using React and Vite:

- Project and database setup
- Frontend connection
- Auth integration
- Fine-grained RLS policy

Supabase offers much more—object storage, real-time subscriptions, edge functions. Dive deeper as you need more features!

**Curious about advanced topics?** Check the [Supabase docs](https://supabase.com/docs) or let us know what you'd like to learn next.

---