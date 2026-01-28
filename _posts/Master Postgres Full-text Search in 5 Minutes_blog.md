# Advanced Full-Text Search in Postgres: Beyond the LIKE Operator

Looking to build a robust keyword search in your Postgres-backed app? While you might reach for the `LIKE` operator as a quick fix, Postgres actually offers a built-in full-text search feature that's faster, smarter, and more flexible—able to handle word variations and intelligent ranking out of the box.

This guide covers how to harness Postgres’s full-text search functionality to implement a scalable, ranking-enabled search system across any number of columns. 

---

## Overview

We’ll walk through:
- Setting up a table for searching
- Inserting and verifying data
- Using `to_tsvector` and `to_tsquery` for full-text search
- Searching across multiple columns
- Leveraging `websearch_to_tsquery` for intuitive queries
- Optimizing with generated columns and indexes
- Ranking search results for relevance
- Utility of weighted columns for prioritized ranking
- Wrapping your search in a reusable Postgres function

---

## Prerequisites

- Access to a PostgreSQL 12+ database
- Familiarity with SQL statements
- Superuser or schema modification rights for creating generated columns and indexes

---

## 1. Setting Up: Create the Books Table and Add Data

First, create a table to house the searchable content.

```sql
CREATE TABLE books (
  id SERIAL PRIMARY KEY,
  title TEXT,
  description TEXT,
  authors TEXT
);
```

Insert sample data (replace with your own examples as appropriate).

```sql
INSERT INTO books (title, description, authors) VALUES
  ('The Little Prince', 'A story about a little prince traveling the universe.', 'Antoine de Saint-Exupéry'),
  ('Little Women', 'This novel tells the story of the four March sisters.', 'Louisa May Alcott');
```

---

## 2. Understanding TSVector and TSQuery

Postgres’s full-text search relies on two key concepts:

- **TSVector**: A special index-friendly representation of text, optimized for searching. It removes stop words (like "the", "in", "on") and normalizes word forms (e.g., "runs", "ran", "running" → "run").
- **TSQuery**: Describes the target search terms—including logical operators.

Transform a text column into a `tsvector` with:

```sql
SELECT to_tsvector('english', title) FROM books;
```

Build a query for searching:

```sql
SELECT to_tsquery('english', 'little') AS query;
```

Search with the matching operator (`@@`):

```sql
SELECT * FROM books
WHERE to_tsvector('english', title) @@ to_tsquery('english', 'little');
```

---

## 3. Searching Across Multiple Columns

To make a search truly useful, you often need to look across several fields (e.g., title, description, authors). Combine columns by concatenating their tsvectors:

```sql
SELECT * FROM books
WHERE (to_tsvector('english', title) ||
       to_tsvector('english', description) ||
       to_tsvector('english', authors))
  @@ to_tsquery('english', 'little');
```

> **Pro tip:** For efficiency, consider creating a generated column that materializes this combination.

---

## 4. Using `websearch_to_tsquery` for Natural-Language Queries

Postgres offers `websearch_to_tsquery`, making search more intuitive—just like web search engines:

- Multiple keywords separated by spaces: **AND** search
- Use `OR` between words: **OR** search
- Prefix with `-` for **NOT** search

Example:
```sql
SELECT * FROM books
WHERE (to_tsvector('english', title) ||
       to_tsvector('english', description) ||
       to_tsvector('english', authors))
  @@ websearch_to_tsquery('english', 'little OR prince');
```

---

## 5. Optimizing Search: Generated Columns and Indexing

To keep your search performant as your dataset grows, persist the tsvector in the table and index it:

1. **Add a generated column:**

   ```sql
   ALTER TABLE books
   ADD COLUMN search_tsv tsvector
     GENERATED ALWAYS AS (
       to_tsvector('english', coalesce(title,'') || ' ' || coalesce(description,'') || ' ' || coalesce(authors,''))
     ) STORED;
   ```

2. **Create an index for fast search:**

   ```sql
   CREATE INDEX search_tsv_idx ON books USING GIN (search_tsv);
   ```

3. **Update your query:**

   ```sql
   SELECT * FROM books
   WHERE search_tsv @@ websearch_to_tsquery('english', 'little');
   ```

> **Note:** Use the GIN index for scalable full-text searches.

---

## 6. Ranking Results by Relevance

To surface the most relevant results first, utilize the `ts_rank` function:

```sql
SELECT *, ts_rank(search_tsv, websearch_to_tsquery('english', 'little')) AS rank
FROM books
WHERE search_tsv @@ websearch_to_tsquery('english', 'little')
ORDER BY rank DESC;
```

---

## 7. Prioritizing Certain Fields with Weighting

Sometimes you want matches in the title to rank higher than those in the description. Postgres allows assigning weights (A-D; A is highest):

1. **Define weighted tsvector:**

   ```sql
   ALTER TABLE books
   ADD COLUMN weighted_search_tsv tsvector
     GENERATED ALWAYS AS (
       setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
       setweight(to_tsvector('english', coalesce(description, '')), 'B') ||
       setweight(to_tsvector('english', coalesce(authors, '')), 'C')
     ) STORED;
   ```

2. **Index the weighted column:**

   ```sql
   CREATE INDEX weighted_search_tsv_idx ON books USING GIN (weighted_search_tsv);
   ```

3. **Rank with weights:**

   ```sql
   SELECT *, ts_rank(weighted_search_tsv, query) AS rank
   FROM books, websearch_to_tsquery('english', 'little') as query
   WHERE weighted_search_tsv @@ query
   ORDER BY rank DESC;
   ```

> **Gotcha:** If you change the fields included or their order, drop and re-create the generated column and index.

---

## 8. Wrapping It in a Postgres Function

To make your search easy to use from an API (for example, with Supabase or PostgREST), you can encapsulate the logic in a SQL function:

```sql
CREATE OR REPLACE FUNCTION search_books(search_text TEXT)
RETURNS SETOF books AS $$
  SELECT *
  FROM books
  WHERE weighted_search_tsv @@ websearch_to_tsquery('english', search_text)
  ORDER BY ts_rank(weighted_search_tsv, websearch_to_tsquery('english', search_text)) DESC;
$$ LANGUAGE sql;
```

Now, you can call this function via an RPC from your client.

---

## Conclusion

Postgres full-text search is far more powerful than simple text matching, providing stemming, stop word removal, logical search, ranking, and field prioritization. Set up generated and weighted columns with GIN indexes for true scalability in production workloads.

**Next steps:**
- Integrate this search with your API or ORM
- Experiment with more complex weighting and ranking strategies
- Explore highlighting search terms and returning snippets

Leverage Postgres’s built-in search—your users (and your CPU) will thank you!