# Enterprise_SDE_Multi_User_Editing

This guide provides a complete workflow for managing an ESRI Enterprise SDE (PostgreSQL) environment with multiple users and Editor Tracking, without the overhead of versioning.

# ArcGIS SDE (PostgreSQL) Multi-User & Editor Tracking Setup

This guide details the administrative workflow for managing multiple editors in an Enterprise Geodatabase without versioning.

---

## 🛠 1. One-Time Foundation Setup
**Run these commands ONCE as a Superuser (postgres/sde)** to establish the security group and automatic permissions.

```sql
-- 1. Create the 'Container' Role (Group)
-- INHERIT allows members to automatically use the group's permissions.
CREATE ROLE gis_internal WITH NOLOGIN NOSUPERUSER INHERIT NOCREATEDB NOCREATEROLE NOREPLICATION;

-- 2. Grant Database & Schema Access to the Group
GRANT CONNECT ON DATABASE production TO gis_internal;
GRANT USAGE ON SCHEMA sde TO gis_internal;

-- 3. Grant Permissions on ALL CURRENT tables & sequences
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA sde TO gis_internal;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA sde TO gis_internal;

-- 4. SET DEFAULT PRIVILEGES (The 'Set and Forget' Step)
-- This ensures NEW tables created by the admin are automatically editable by the group.
ALTER DEFAULT PRIVILEGES IN SCHEMA sde 
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO gis_internal;

ALTER DEFAULT PRIVILEGES IN SCHEMA sde 
GRANT USAGE, SELECT ON SEQUENCES TO gis_internal;e access to all tables without further SQL configuration.
```

## 🛠 2. Recurring Task: Adding a New User
** Run these commands EVERY TIME you hire a new editor. They will automatically inherit all table permissions from gis_internal.
```sql
-- 1. Create the individual account
CREATE ROLE sraza WITH LOGIN PASSWORD 'razagis' NOSUPERUSER INHERIT NOCREATEDB NOCREATEROLE NOREPLICATION;

-- 2. Add them to the group to inherit permissions
GRANT gis_internal TO sraza;

-- 3. Explicitly allow database connection
GRANT CONNECT ON DATABASE production TO sraza;
```

# 3. Troubleshooting: Geodatabase Connections
If a user sees the error: "The geodatabase is not accepting connections", follow these steps:

Connect to the SDE in ArcGIS Pro as a Superuser.

Right-click Connection > Properties.

Go to the Connections tab.

Check "Geodatabase is accepting connections".

Click OK.
