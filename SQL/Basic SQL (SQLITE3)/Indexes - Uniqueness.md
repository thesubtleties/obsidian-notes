

1. UNIQUE on an INDEX vs. UNIQUE on a column:

   When you create a UNIQUE INDEX, it's similar to adding a UNIQUE constraint to the column(s) involved in the index, but with some differences:

   - A UNIQUE INDEX can span multiple columns, enforcing uniqueness across the combination of those columns.
   - It creates an index structure, which can improve query performance.

2. Single column UNIQUE INDEX:

   If you create a UNIQUE INDEX on a single column like this:

   ```sql
   CREATE UNIQUE INDEX idx_cookie_name ON cookies(name);
   ```

   This is functionally equivalent to declaring the column as UNIQUE when creating the table:

   ```sql
   CREATE TABLE cookies (
     name TEXT UNIQUE,
     ...
   );
   ```

   In both cases, you cannot have two cookies with the same name.

3. Multi-column UNIQUE INDEX:

   The power of UNIQUE INDEX becomes more apparent with multiple columns:

   ```sql
   CREATE UNIQUE INDEX idx_cookies_baker_id_type ON cookies(baker_id, type);
   ```

   This allows multiple cookies with the same name, but prevents the same baker from making two cookies of the same type.

4. Partial uniqueness:

   With a multi-column UNIQUE INDEX, individual columns can have duplicate values, but the combination must be unique. For example, with the index above:
   - Multiple bakers can make chocolate chip cookies (same type, different baker_id)
   - One baker can make many types of cookies (same baker_id, different type)
   - But one baker cannot make two chocolate chip cookies (this would violate the uniqueness)

In summary, a UNIQUE INDEX ensures uniqueness based on all the columns included in the index, either individually (for a single-column index) or in combination (for a multi-column index). It's not necessary for each individual column in a multi-column UNIQUE INDEX to be unique on its own.