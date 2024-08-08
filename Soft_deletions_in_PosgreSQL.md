## Soft deletions in PostgreSQL

### Links

[Soft deletion by Jose Valim twitter](https://x.com/josevalim/status/1821143821649948822)

[Soft deletion PostgreSQL + Ruby article](https://evilmartians.com/chronicles/soft-deletion-with-postgresql-but-with-logic-on-the-database)

### TL;DR

```sql
# Clean the database
DROP TABLE IF EXISTS _orders CASCADE;
DROP TABLE IF EXISTS _users CASCADE;
DROP TABLE IF EXISTS orders CASCADE;
DROP TABLE IF EXISTS users CASCADE;

# Build the database (for hard deletion)
CREATE TABLE users (
  id integer PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  name text NOT NULL
);
CREATE TABLE orders (
  id integer PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  user_id integer NOT NULL,
  number text NOT NULL,
  FOREIGN KEY (user_id) REFERENCES users (id) ON DELETE CASCADE
);

# Populate it with some data
INSERT INTO users (name) VALUES ('Andrew'), ('Vladimir'), ('Sara');
INSERT INTO orders (user_id, number) VALUES (1, 'A1'), (1, 'A2'), (2, 'V1'), (2, 'V2'), (3, 'S3');

# Add columns for soft deletion
ALTER TABLE users ADD COLUMN deleted boolean NOT NULL DEFAULT false;
ALTER TABLE orders ADD COLUMN deleted boolean NOT NULL DEFAULT false;

# Hide tables on behind of views
ALTER TABLE users RENAME TO _users;
CREATE VIEW users AS SELECT * FROM _users WHERE NOT deleted;
ALTER TABLE orders RENAME TO _orders;
CREATE VIEW orders AS SELECT * FROM _orders WHERE NOT deleted;

# Add rewriting rules for views
CREATE RULE _soft_deletion AS ON DELETE TO orders DO INSTEAD (
  UPDATE _orders SET deleted = true WHERE id = old.id
);
CREATE RULE _soft_deletion AS ON DELETE TO users DO INSTEAD (
  UPDATE _users SET deleted = true WHERE id = old.id
);

# Add rewriting rules for associated table
CREATE RULE _delete_orders AS ON UPDATE TO _users
  WHERE NOT old.deleted AND new.deleted
  DO ALSO UPDATE _orders SET deleted = true WHERE user_id = old.id;

# Then check the results
DELETE FROM users WHERE name = 'Andrew';
SELECT * FROM users;
# Should not contain 'Andrew'

SELECT * FROM _users;
# Should have all records, including 'Andrew' marked as deleted

SELECT * FROM orders;
# Should not contain 'A1' and 'A2'

SELECT * FROM _orders;
# Should have all records, including 'A1' and 'A2' marked as deleted
```

Also, we could handle views vs real tables in Ecto:

```elixir
{"_orders", Order}
|> from(as: :order)
|> Repo.all()
```