### Bun + Supabase: Fast SQL & S3 Ops in Seconds

Bun comes with SQL and S3 support out of the box—way faster than third-party NPM packages. You can upload to S3 or query your database with just a couple lines.

1. **Spin up Supabase instantly:**  
   Go to [database.new](https://database.new) to create a free Supabase project. You get Postgres and S3-compatible storage right away.

2. **Set environment variables:**  
   Connect Bun to Supabase with env variables like:
   ```env
   SUPABASE_URL=<your-supabase-url>
   SUPABASE_KEY=<your-service-key-or-token>
   ```

3. **Code: Upload + Query**  
   With Bun, you can handle S3 uploads and Postgres queries in seconds:
   ```js
   // SQL query example
   const db = new Bun.SQL({ connectionString: process.env.SUPABASE_URL });
   const results = await db.query('SELECT * FROM your_table');
   
   // S3 upload example
   const s3 = new Bun.S3({ endpoint: process.env.SUPABASE_URL, credentials: { accessKeyId: '<key>', secretAccessKey: '<secret>' }});
   await s3.putObject({ Bucket: '<bucket>', Key: 'file.txt', Body: Buffer.from('hello') });
   ```

You end up with a faster backend and less code to write or maintain.