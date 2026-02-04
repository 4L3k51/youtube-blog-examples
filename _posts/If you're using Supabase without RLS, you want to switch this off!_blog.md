### Disabling the Superbase Data API When Not Using RLS

If you're using Superbase **without** Row-Level Security (RLS), be aware of a significant security risk:

1. **Data API Exposes Your Database**  
   - Superbase's Data API makes your Postgres database accessible via HTTP endpoints.
   - With RLS off, anyone can access and modify your data if they have the service role key, secret key, or certain connection details.

2. **Vulnerability Example**  
   - Without RLS, an external attacker can run queries like:
     ```http
     DELETE FROM users;
     ```
   - This can wipe out your entire `users` table.

3. **How to Disable the Data API**  
   - Go to your Superbase project.
   - Navigate to **Project Settings** > **Data API**.
   - Turn off the Data API and click **Save**.

After disabling, you can handle authorization your own way without exposing your database to the world.