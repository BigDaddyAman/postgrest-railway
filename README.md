# PostgreSQL REST API (PostgREST)

This repository provides a simple Railway-ready service that exposes a PostgreSQL database as a REST API using **PostgREST**.

It can be used to deploy a REST API on top of PostgreSQL without writing backend code.

---

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/postgresql-rest-api-postgrest?referralCode=nIQTyp&utm_medium=integration&utm_source=template&utm_campaign=generic)

## What this does

- Runs PostgREST in a Docker container
- Connects to an existing Railway PostgreSQL service
- Exposes tables (or views) as REST endpoints
- Uses PostgreSQL roles and permissions for security

No database is included in this repository.  
You must deploy PostgreSQL separately (Railway official Postgres).

---

## Architecture

```
HTTP Client
  |
  v
PostgREST (this service)
  |
  v
PostgreSQL (Railway)
```

---

## Deployment (Railway)

### 1. Create a Railway project
- Create a new project in Railway
- Add a **PostgreSQL** service using Railway's official template

### 2. Deploy this repository
- Add a new service
- Deploy from this GitHub repository
- Railway will detect the Dockerfile automatically

### 3. Set environment variables

In the **PostgREST service**, set the following variables:

```
PGRST_DB_URI=${{Postgres.DATABASE_URL}}
PGRST_DB_SCHEMA=public
PGRST_DB_ANON_ROLE=postgres
PORT=3000
```

By default, the `postgres` role is used so the API works immediately after deployment.

### 4. Add a public domain (optional)

Adding a public domain is optional.

If the API is only used by other services within the same Railway project,
you can omit the public domain and access the service via Railway’s internal
networking.

A public domain is required for access from outside Railway.

---

## API usage

Each table or view becomes an endpoint.

Examples:

```
GET /test_items
GET /test_items?id=eq.1
GET /test_items?name=ilike.*hello*
GET /test_items?limit=10&offset=0
```

Example request:

```bash
curl https://your-postgrest-domain.up.railway.app/test_items
```

---

## Switching to a restricted database role (recommended)

By default, this template uses the `postgres` role so the API works immediately
after deployment.

For production use, it is strongly recommended to switch to a restricted
read-only role.

### 1. Create a read-only role

Run the following in PostgreSQL:

```sql
CREATE ROLE api_user NOLOGIN;

GRANT CONNECT ON DATABASE railway TO api_user;
GRANT USAGE ON SCHEMA public TO api_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO api_user;

ALTER DEFAULT PRIVILEGES
IN SCHEMA public
GRANT SELECT ON TABLES TO api_user;
```

### 2. Grant access to existing tables or views

```sql
GRANT SELECT ON table_name TO api_user;
```

### 3. Reload the PostgREST schema cache

```sql
NOTIFY pgrst, 'reload schema';
```

Or restart the PostgREST service.

### 4. Update the environment variable

Change the PostgREST service variable to:

```
PGRST_DB_ANON_ROLE=api_user
```

---

## Security notes

- Do not expose the postgres role publicly
- Prefer views instead of tables for public APIs
- Remove the public domain if the API is internal-only
- Rotate credentials if exposed during testing

---

## Files in this repository

```
.
├── Dockerfile
└── README.md
```

---

## Powered by

- PostgREST
- PostgreSQL
- Railway