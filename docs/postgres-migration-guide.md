# Migration Guide: Moving from SQLite to DigitalOcean Managed PostgreSQL

The platform was intentionally built using Prisma ORM with standard relational types, foreign keys, and indexes to allow straightforward migration from SQLite to PostgreSQL with zero business logic changes.

---

## Step 1: Provision Managed PostgreSQL on DigitalOcean

1. In the DigitalOcean Cloud Console, go to **Databases** -> **Create Database Cluster**.
2. Select **PostgreSQL 15 or 16**.
3. Choose the same region as your application droplet.
4. Copy the connection string:
   `postgresql://doadmin:secret@db-postgresql-fra1-12345-do-user-67890-0.b.db.ondigitalocean.com:25060/defaultdb?sslmode=require`

---

## Step 2: Update Prisma Schema

In `packages/database/prisma/schema.prisma`, update the `datasource` block:

```diff
datasource db {
-  provider = "sqlite"
-  url      = env("DATABASE_URL")
+  provider = "postgresql"
+  url      = env("DATABASE_URL")
}
```

---

## Step 3: Run Prisma Migrations for PostgreSQL

Set your `DATABASE_URL` to the Postgres connection string and generate the migration:

```bash
export DATABASE_URL="postgresql://doadmin:secret@YOUR_POSTGRES_HOST:25060/defaultdb?sslmode=require"

# Generate fresh Prisma Client
npx prisma generate --schema=packages/database/prisma/schema.prisma

# Push or run migrations on the PostgreSQL instance
npx prisma db push --schema=packages/database/prisma/schema.prisma

# Seed admin account and baseline configurations
npm run db:seed
```

---

## Step 4: Optional Data Transfer from SQLite to PostgreSQL

If you wish to transfer existing historical campaigns and participants from SQLite to PostgreSQL, use `pgloader`:

```bash
# Install pgloader
sudo apt-get install -y pgloader

# Run migration script
pgloader sqlite:///app/data/prod.db postgresql://doadmin:secret@YOUR_POSTGRES_HOST:25060/defaultdb?sslmode=require
```

---

## Step 5: Update Application Environment

In your production `.env` or Docker Compose environment:

```env
DATABASE_URL="postgresql://doadmin:secret@YOUR_POSTGRES_HOST:25060/defaultdb?sslmode=require"
```

Restart your containers:
```bash
docker compose down && docker compose up -d
```
The platform will now use PostgreSQL with complete connection pooling, transactional locking, and high availability.
