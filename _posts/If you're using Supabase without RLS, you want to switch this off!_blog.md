# Secure Your Supabase Database: Disable the Data API When Not Using RLS

Supabase is a popular open-source Firebase alternative that makes building secure, scalable applications on top of PostgreSQL a breeze. With features like authentication, database access, and instant APIs, it’s a powerful toolkit for developers. But with great power comes great responsibility—especially when it comes to database security.

If you choose not to use Row-Level Security (RLS) for your tables in Supabase, it’s critical to understand one potentially dangerous setting you must review: the Supabase Data API. This guide explains why, demonstrates the risks, and shows you exactly how to disable the Data API to keep your data safe.

---

## Why Disabling RLS Opens a Security Hole

Row-Level Security (RLS) is a PostgreSQL feature that lets you define precise, per-row access controls on your data. By default, Supabase encourages you to use RLS on every table to prevent unauthorized access by outside users. However, some teams disable RLS because they prefer to manage permissions elsewhere, or want to allow broader access internally.

If you’ve turned off RLS—or never enabled it—Supabase’s Data API exposes a large attack surface. Without any row-level restrictions, anyone with access can make arbitrary requests to your database via convenient HTTP endpoints, including modifying or deleting critical data.

---

## The Real-World Risk: Unprotected Data APIs

With RLS disabled, your database is exposed to the internet via the Supabase Data API. Here’s what that exposure can look like in practice.

**Example:**  
Suppose you have a `users` table with RLS disabled. The Supabase Data API allows external HTTP requests, including powerful destructive operations.

Here’s an example HTTP DELETE request that could be used to wipe your `users` table:

```http
DELETE https://<project-ref>.supabase.co/rest/v1/users
apikey: <your-service-role-key>
Authorization: Bearer <your-service-role-key>
```

**What does this do?**  
If issued, this request would delete _every row_ in your `users` table—without any further checks.

> **⚠️ Warning:**  
> If RLS is disabled on a table, and the Data API is enabled, anyone with the appropriate key (or if your environment is misconfigured and leaks a key) can directly read, modify, or delete your entire dataset via HTTP requests.  
>  
> This is especially risky if you accidentally expose your Service Role key or other privileged credentials in public code repositories, client-side code, or environment variables.

---

## How to Disable the Supabase Data API

If you’re intentionally _not_ using RLS (perhaps because you’re handling all access in your application code or via direct Postgres connections), you should immediately disable the Data API for your Supabase project.

Follow these steps to do so:

1. **Open Your Supabase Project Settings**
2. **Navigate to the “Data API” Section**
    - In the left sidebar, locate **“API”** or **“Settings”**.
    - Find and select the **“Data API”** tab or section.
3. **Disable the Data API**
    - Find the toggle or switch labeled **“Enable Data API”** or similar.
    - Set this to the **Off** position.
4. **Save Your Changes**
    - Click **Save** to apply the new security configuration.

Once the Data API is disabled, the HTTP endpoints for your tables will no longer be available, meaning outside users cannot make direct requests to your data through Supabase’s auto-generated REST interface.

---

## Next Steps: Secure Your Data

Turning off the Data API means you’ll need to handle database access another way. Here are some alternative approaches:

- **Use Direct Postgres Connections**  
  Handle queries on your server by connecting directly to the database with the Service Role key or another tightly-controlled credential.
- **Implement Server-Side Authorization**  
  Build a secure backend API, and only expose endpoints after verifying the identity and permissions of authenticated users.
- **Consider Enabling RLS When Possible**  
  If you want to use the Data API or Supabase’s auto-generated APIs, enabling RLS with appropriate policies is the recommended best practice.

---

## Conclusion

Supabase’s Data API is incredibly powerful—perhaps too powerful if left unguarded. If you opt out of RLS, do not forget to disable the Data API, otherwise your entire database could be exposed to accidental or intentional misuse via HTTP.

**Stay safe:**
- Always use RLS if exposing auto-generated APIs to users.
- If you don’t want RLS and plan to use only server-side/database connections, disable the Data API immediately.
- Regularly audit your Supabase keys and project settings for least-privilege access.

By following these precautions, you can enjoy Supabase’s developer-friendly features without putting your critical data at risk.

---