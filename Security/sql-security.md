# MS-SQL Server Security

## Basic concepts
**Principal** is an identity - it can be an individual person or a role (group of users).

**Permissions** can be assigned to individual users, roles, etc.  
Permissions are added up, i.e. if a user has some permissions, and is also assigned a role, the user will then inhert the permissions from that role as well.  

_Server-level permissions_  
Related to anything that happens on the server, like a permission to do backups.  
Server-level permission can only be given to logins or server roles.  

These permission are hierachical.  
- Control server: top level permission - allows you to do everything
  - Alter any event notification
    - Create DDL event notification
    - Create Trace event notification
  - Alter any database
    - Create any database

_Database-level permissions_  
Permissions to do e.g. INSERT, UPDATE, DELETE.  
Managed using  GRANT, REVOKE, DENY.  

More on permissions here: https://docs.microsoft.com/en-us/sql/relational-databases/security/permissions-database-engine?view=sql-server-ver15

**Secureables**
The object that is being secured, such as a Database, schema, tables, views, etcs.  
Each secureable can have permissions.


## Roles
You can assign multiple permissions to a role, and then give users a role.  

## Firewalls
When users connect, they first reach trhe SQL Database Firewall.  
Firstly, the Server-level rules are evaluated. If the connecting IP is in range, then they can connect to the SQL Database, otherwise, the Database-level rules are evaluated.  

Server-level firewall rules: allow access to all databases on the server  
Database-level firewall rules: allow access to a specific database  

For more about SQL firewalls: https://docs.microsoft.com/en-us/azure/sql-database/sql-database-firewall-configure

## Database access
#### login 
  Login is a Server (instance) level security that can take advantage of external providers like WINDOWS or Active Directory.
  A default database can also be assigned to a login (does not work in Azure SQL).
  When creating logins, you can also enforce policies by adding CHECK_POLICY = on (policy check does not work in Azure SQL)

```
CREATE LOGIN App
WITH PASSWORD = 'some_secret_pw'
```  

Creating logins: https://docs.microsoft.com/en-us/sql/t-sql/statements/create-login-transact-sql?view=sql-server-ver15

#### User  
Database level security.

```
CREATE USER AppUser
FOR LOGIN App
WITH DEFAULT_SCHEMA = app
```

Users that own any objects cannot be dropped.   
Creating users: https://docs.microsoft.com/en-us/sql/relational-databases/security/authentication-access/create-a-database-user?view=sql-server-ver15  

#### Schema  
Basically a container for database objects (i.e. tables, views, stored procs, etc.)

One login may have multiple users connected to it, but only one user per database is allowed for one login.

Always assign permissions on a least-privilege basis.


#### Specials  
_dbo_  
The dbo (database owner) user is mapped to the sa (Server Administrator) login.  
This is a special account setup at MS SQL Server creation time. You can do anything on the SQL Server with this user.  
the dbo. user should NOT be used by applications or any user other than the actual administrator.  

_guest_  
The guest account is disabled by default.  
This is used by any login that has no user accounts mapped to it.

## Connecting mutiple SQL servers
Run stored procedure to link two servers  
```
EXEC sp_addlinkedserver@server = 'RemoteServer',
@srvproduct = ''
@provider = 'SQLOLEDB',
@datrasrc = 'r:\datrasource\RemoteServer'
```

For this to work, the first server needs trusted delegation in Active Directory, and read and write abilities on the service principal name for both those instances.

See this link for more https://docs.microsoft.com/en-us/archive/blogs/askds/understanding-kerberos-double-hop