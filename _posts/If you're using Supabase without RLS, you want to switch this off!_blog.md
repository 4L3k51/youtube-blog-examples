If you're using Supabase without Row-Level Security (RLS), be careful: Supabase’s Data API exposes your database tables over HTTP endpoints. With RLS disabled, anyone could run destructive requests, like deleting every user in your table, just by sending the right HTTP call.

**Example HTTP delete request (dangerous with RLS off):**
```http
DELETE /rest/v1/users
Authorization: Bearer <service_role_key>
```

This would wipe all rows from the `users` table if RLS is off and the API is enabled.

**How to prevent this:**  
If you’re using the secret key, service role key, or direct Postgres connections and don’t need the Data API, turn it off:

1. Go to your Supabase project.
2. Open **Project Settings**.
3. Find the **Data API** section.
4. Toggle it **off** and click **Save**.

Now your tables won’t be exposed via those API endpoints, and you can manage authorization however you want.