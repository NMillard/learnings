# MS-SQL Server Security

## Basic concepts
**Principal**
Is an identity - it can be an individual person or a role (group of users).

**Permissions**
Permissions can be assigned to individual users, roles, etc.  
Permissions are added up, i.e. if a user has some permissions, and is also assigned a role, the user will then inhert the permissions from that role as well.

**Secureables**
The object that is being secured, such as a Database, schema, tables, views, etcs.  
Each secureable can have permissions.

# Basics


- Login
  Server (instance) level security
- User
  Database level security.
- Schema
  Basically a container for database objects (i.e. tables, views, stored procs, etc.)

One login may have multiple users connected to it, but only one user per database is allowed for one login.

Always assign permissions on a least-privilege basis.
