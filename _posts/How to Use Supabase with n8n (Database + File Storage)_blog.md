# Automate Social Media Content Generation with n8n, Supabase, and OpenAI

In this guide, you'll learn how to build an automated workflow using [n8n](https://n8n.io/), Supabase, and OpenAI to generate social media content—specifically, helpful Postgres tips for Twitter—with accompanying AI-generated images. This workflow will run daily, leverage Supabase's database and S3-compatible storage, and leave you well-positioned to expand with enhancements like human approval loops or automated posting.

---

## Overview

We'll create an n8n workflow that:

1. **Triggers daily**
2. **Generates an engaging Twitter post (about Postgres) and an image prompt using OpenAI**
3. **Uses OpenAI to generate an image based on the prompt**
4. **Uploads the image to Supabase Storage**
5. **Stores all relevant data in a Supabase database**

---

## Prerequisites

To follow along, you'll need:

- An n8n instance (self-hosted or cloud)
- A Supabase project with database and storage enabled
- OpenAI API access (or similar LLM provider)
- S3 credentials for Supabase Storage (see below)
- (Optional) Familiarity with JSON and SQL

---

## Step 1: Set Up Supabase Storage

1. **Create a Supabase Project**  
   If you don't have one, head to [database.new](https://database.new) and set up a fresh Supabase project.

2. **Create a Storage Bucket**
   - Navigate to the Storage section in the Supabase dashboard.
   - Create a new (public) bucket, e.g., `social-media-images`.

3. **Configure S3 Credentials**
   - Supabase Storage is S3-compatible.
   - Go to your Supabase project's Storage settings > Configuration.
   - Note the endpoint and region.
   - Generate an Access Key and Secret Access Key.
   
   **Keep your Access Key Secret! Do not expose it.**

---

## Step 2: Create the Supabase Table

You'll need a table to store your post data. Here’s an example SQL snippet. Adjust columns as needed for your workflow:

```sql
create table social_media_posts (
  id serial primary key,
  tweet_text text,
  image_prompt text,
  image_url text,
  tip_category text,
  code_snippet text,
  status text default 'pending',
  published_at timestamp,
  created_at timestamp default now()
);
```

Apply this in the Supabase SQL Editor.  

> **Note:**  
> Ensure [Row Level Security (RLS)](https://supabase.com/docs/guides/auth/row-level-security) is enabled on your table to control access.

---

## Step 3: Build the n8n Workflow

### Option 1: Let n8n's AI Agent or External LLM Generate the Workflow

You can ask an AI (such as ChatGPT or Gemini) to generate JSON for an n8n workflow based on your requirements:

- Request:  
  "Generate a JSON format of an n8n workflow that runs daily, uses OpenAI to generate a Tweet and image prompt about Postgres, generates an image, uploads it via S3 to Supabase Storage, and inserts all metadata into a Supabase database."

- Once you receive the JSON:
  1. Copy it.
  2. In n8n, import the workflow via the import feature.
  3. Adjust as needed based on your environment (credentials, field mappings, etc).

### Option 2: Build the Workflow from Scratch

#### 1. Daily Trigger

Set up a trigger node to execute the workflow every 24 hours.

#### 2. Generate Twitter Content & Image Prompt (OpenAI)

- Use the OpenAI node to craft both your social media post and an image prompt.
- Example system prompt:
  
  ```
  You are a Postgres expert and social media content creator. Generate engaging Twitter content about Postgres tips, along with an image prompt for a helpful code snippet. Respond in valid JSON, no markdown.
  ```
- Ensure the output mode is set to **JSON**.

#### 3. Generate Image (OpenAI)

- Feed the image prompt from the previous step into a new OpenAI node.
- Use the image generation model (e.g., GPT-image-1).
- Adjust image dimensions if needed.

#### 4. Generate File Name

- Add a **Data Transformation (Edit Fields)** node before your upload step.
- Generate a timestamped filename, e.g.,

  ```js
  // pseudocode for n8n expression
  {{$now}}.png
  ```

- Store as a new field: `file_name`.

#### 5. Upload Image to Supabase Storage via S3

- **Use the S3 Upload node** (not the Supabase node, which is database-only).
- Configure with your Supabase S3 endpoint, region, Access Key, Secret Access Key, and bucket name.
- Upload using the dynamically generated `file_name`.

  > **Important:**  
  > Enable "Force path style" in your S3 node. This is required for Supabase S3 compatibility.

#### 6. Construct Image URL

- The pattern for Supabase Storage public URLs is:

  ```
  https://<project-ref>.supabase.co/storage/v1/object/public/<bucket-name>/<file-name>
  ```

- Construct this in a new data transformation node, using your project ref and the output file name.

#### 7. Insert Data Into Supabase Database

- Add the Supabase node (database mode).
- Connect using:
  - **Host:** Your project's Supabase URL
  - **Service Role Secret:** From project settings → Data API
- Insert the post data fields, referencing outputs from previous steps.

---

## Gotchas & Tips

> **Supabase Node vs. S3 Node**
>
> The Supabase node in n8n only supports database actions. To upload to Supabase Storage, always use the S3-compatible node.

> **Filename Management**
>
> S3 upload outputs don't include file names. Always generate and reference your desired file name *before* the upload step.

> **Supabase URL Construction**
>
> Double-check your project ref and use the correct pattern for public URLs.

> **Security Warning**
>
> Never expose your S3 Access Key and Secret Access Key in code, logs, or client-side code.

> **Row Level Security**
>
> After creating your table, explicitly enable Row Level Security in Supabase to protect your data.

---

## Next Steps & Expansion Ideas

- **Human-in-the-Loop:** Add steps to allow human review and approval before posts go live.
- **Auto-Posting:** Integrate with Twitter (X) API or other platforms to automate posting.
- **Content Dashboard:** Build a web front-end to manage, review, or schedule your AI-generated content.

---

## Conclusion

Using n8n in tandem with Supabase and OpenAI offers a powerful, flexible way to automate content creation workflows. You've now built a daily bot that creates, stores, and manages engaging social content with minimal manual effort—and you can expand or adapt it as your needs grow.

Have fun experimenting, and let us know what creative automations you build!