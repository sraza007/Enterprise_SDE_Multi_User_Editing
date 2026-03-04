# Enterprise_SDE_Multi_User_Editing

This guide provides a complete workflow for managing an ESRI Enterprise SDE (PostgreSQL) environment with multiple users and Editor Tracking, without the overhead of versioning.

ArcGIS Enterprise Geodatabase (PostgreSQL) User & Tracking Setup
This repository contains the standardized workflow for creating users, managing permissions, and enabling editor tracking for non-versioned editing in a PostgreSQL SDE environment.

# 1. PostgreSQL User & Permissions Setup
Run these commands as a Superuser (e.g., postgres) to create the user, the group, and set "set-and-forget" permissions.

-- ===========================================================================
-- STEP 1: CREATE GROUP AND USER
-- ===========================================================================

-- Create the Group Role (Permissions are managed here)

CREATE ROLE gis_internal WITH NOLOGIN NOSUPERUSER INHERIT NOCREATEDB NOCREATEROLE NOREPLICATION;

-- Create the Individual User 'sraza'

CREATE ROLE sraza WITH LOGIN PASSWORD 'razagis' NOSUPERUSER INHERIT NOCREATEDB NOCREATEROLE NOREPLICATION;

-- Add the user to the group

GRANT gis_internal TO sraza;

-- ===========================================================================
-- STEP 2: DATABASE & SCHEMA ACCESS
-- ===========================================================================

-- Grant Database & Schema Access (Replace 'production' with your DB name)

GRANT CONNECT ON DATABASE production TO gis_internal;
GRANT USAGE ON SCHEMA sde TO gis_internal;

-- Grant Permissions on ALL CURRENT tables & sequences in 'sde' schema

GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA sde TO gis_internal;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA sde TO gis_internal;

-- ===========================================================================
-- STEP 3: CONFIGURE DEFAULT PRIVILEGES (For Future Tables)
-- ===========================================================================

-- Ensures NEW tables created by the owner are automatically accessible to the group

ALTER DEFAULT PRIVILEGES IN SCHEMA sde 
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO gis_internal;

ALTER DEFAULT PRIVILEGES IN SCHEMA sde 
GRANT USAGE, SELECT ON SEQUENCES TO gis_internal;


# 2. Troubleshooting: "Geodatabase is not accepting connections"
If a standard user gets this error but the admin can connect, the Geodatabase is "locked."

Resolution Steps:

Open ArcGIS Pro and connect as a Superuser (postgres or sde).

Right-click the database connection in the Catalog Pane.

Select Properties.

Navigate to the Connections tab.

Ensure the box "Geodatabase is accepting connections" is checked.

Click OK.

# 3. Editor Tracking Configuration (Non-Versioned)
To track edits without using versioning, tracking must be enabled on each feature class.

How to Enable
In the Catalog pane, right-click your Feature Class or Table.

Select Manage > Enable Editor Tracking.

Use the default field mapping:

created_user: Logs the creator.

created_date: Logs creation timestamp.

last_edited_user: Logs the editor (e.g., sraza).

last_edited_date: Logs the last update timestamp.

Editing Workflow
No Versioning Required: Users edit the "Base" table directly.

Automatic Logging: ArcGIS Pro captures the credentials from the .sde connection file and writes them to the last_edited_user field automatically upon saving.

# 4. Maintenance: Adding New Users
To add a new team member in the future, you only need to run these three lines:

CREATE ROLE [username] WITH LOGIN PASSWORD '[password]' INHERIT;
GRANT gis_internal TO [username];
GRANT CONNECT ON DATABASE production TO [username];

Note: Because of the Default Privileges set in Phase 1, the new user will immediately have access to all tables without further SQL configuration.
