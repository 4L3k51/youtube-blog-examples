# Building Smarter, Faster Keyword Search in Postgres with Full-Text Search

PostgreSQL's built-in full-text search features far surpass basic usage of the `LIKE` operator for keyword matching. This guide demonstrates how to implement advanced, efficient search across multiple columns, handle ranking for relevance, prioritize certain fields, and scale your search using Postgres' full-text search capabilities.

## Overview

We'll walk through building a scalable, ranked search feature in PostgreSQL. This approach allows for searching across multiple columns (such as `title`, `description`, and `authors`) while optimizing for performance and providing more accurate results than crude pattern matches.

## Prerequisites

- Basic knowledge of SQL and PostgreSQL
- Access to a PostgreSQL database

## Step 1: Set Up Your Data Table

Start by creating a `books` table to serve as the basis for our search.

```sql
CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title TEXT,
    description TEXT,
    authors TEXT
);
```

Insert some dummy data for testing purposes.

```sql
INSERT INTO books (title, description, authors) VALUES
('The Little Prince', 'A young boy explores planets.', 'Antoine de Saint-Exupéry'),
('Little Women', 'The lives of four sisters during the Civil War.', 'Louisa May Alcott'),
('Running Wild', 'Adventures in the wild.', 'Jon Krakauer');
```

## Step 2: Understanding Full-Text Search Basics

### TSVector and TSQuery

- `TSVector` is a Postgres-specific data format optimized for full-text search. It removes "stop words" (common words like "in", "on", "the") and normalizes word forms (e.g., "runs," "running," "ran" become "run").
- `TSQuery` is the query structure for text searching—think of it as your search keywords (like what you’d enter into a search box).

Convert a column to a `TSVector`:

```sql
SELECT to_tsvector('english', title) FROM books;
```

Query with TSVector and TSQuery using the `@@` operator:

```sql
SELECT * FROM books
WHERE to_tsvector('english', title) @@ to_tsquery('english', 'little');
```

## Step 3: Search Across Multiple Columns

To enable searching across more than one field (e.g., `title`, `description`, `authors`), combine columns before converting to `TSVector`:

```sql
SELECT * FROM books
WHERE to_tsvector('english', title || ' ' || description || ' ' || authors)
      @@ to_tsquery('english', 'little');
```

## Step 4: Using Advanced Search Queries

Instead of `to_tsquery`, use `websearch_to_tsquery` for more natural search expressions, similar to Google search:

- Use spaces for logical AND (`apple banana` matches records containing both).
- Use `or` for logical OR (`apple or banana` matches either).
- Use `-` for negation (`apple -banana` matches records with apple but not banana).

```sql
SELECT * FROM books
WHERE to_tsvector('english', title || ' ' || description)
      @@ websearch_to_tsquery('english', 'little or wild');
```

## Step 5: Improve Performance with Generated Columns and Indexes

On-the-fly generation of `TSVector` values is inefficient. For scalable search:

1. **Create a generated TSVector column:**

    ```sql
    ALTER TABLE books ADD COLUMN search_vector TSVECTOR
      GENERATED ALWAYS AS (
        setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
        setweight(to_tsvector('english', coalesce(description, '')), 'B')
      ) STORED;
    ```

2. **Add an index for efficient search queries:**

    ```sql
    CREATE INDEX idx_books_search_vector ON books USING GIN (search_vector);
    ```

> **Note:**  
> Always index your `TSVector` columns for real-world search scalability!

## Step 6: Rank and Prioritize Search Results

To rank results based on relevance, use the `ts_rank` function:

```sql
SELECT *, ts_rank(search_vector, websearch_to_tsquery('english', 'little'))
FROM books
WHERE search_vector @@ websearch_to_tsquery('english', 'little')
ORDER BY ts_rank(search_vector, websearch_to_tsquery('english', 'little')) DESC;
```

### Prioritizing Columns

Use `setweight` to give higher priority to certain fields (e.g., weigh `title` higher than `description`). Weights—`A` (highest) to `D` (lowest):

```sql
ALTER TABLE books
  DROP COLUMN IF EXISTS search_vector,
  ADD COLUMN search_vector TSVECTOR
    GENERATED ALWAYS AS (
      setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
      setweight(to_tsvector('english', coalesce(description, '')), 'B')
    ) STORED;

-- Remember to re-create the index if you drop and re-create the column.
CREATE INDEX idx_books_search_vector ON books USING GIN (search_vector);
```

## Step 7: Use from Client or a Supabase Function

To expose this feature to your application (e.g., from Supabase or your own client), you can create a Postgres function like:

```sql
CREATE OR REPLACE FUNCTION search_books(query TEXT)
RETURNS SETOF books AS $$
BEGIN
  RETURN QUERY
  SELECT *
  FROM books
  WHERE search_vector @@ websearch_to_tsquery('english', query)
  ORDER BY ts_rank(search_vector, websearch_to_tsquery('english', query)) DESC;
END;
$$ LANGUAGE plpgsql;
```

Invoke this function via RPC from your client library.

## Conclusion and Next Steps

You've now implemented a robust, scalable full-text search in PostgreSQL, including multi-column search, ranking, relevance weighting, and performance optimizations. From here, you can:

- Add highlighting for matched terms in your app.
- Experiment with custom dictionaries for different languages.
- Extend to search over additional related tables.

PostgreSQL's full-text search empowers you to deliver sophisticated search experiences well beyond what `LIKE` operator can achieve.

Happy building!