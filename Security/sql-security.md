# Basics

- Login
  Server (instance) level security
- User
  Database level security.
- Schema
  Basically a container for database objects (i.e. tables, views, stored procs, etc.)

One login may have multiple users connected to it, but only one user per database is allowed for one login.

Always assign permissions on a least-privilege basis.
