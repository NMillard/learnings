# MS-SQL Server Security

## Basic concepts
**Principal** is an identity - it can be an individual person or a role (group of users).  
Principals can request SQL server resources. Every principal as a security identrifier (SID).

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
The object that is being secured, such as a Database, schema, tables, views, stored procedures, etcs.  
Each secureable can have permissions.


## Roles
You can assign multiple permissions to a role, and then give users a role.  

### Server roles
#### Fixed server roles
- sysadmin: highest level that can do absolutely anything
- serveradmin: can do most things, such as configure server, work with server processes
- securityadmin: can work with logins, role memberships
- processadmin: work with processes, stop/start service, change users' connectivity
- setupadmin: add/remove features, server functionality
- bulkadmin
- diskadmin: alter resources
- dbcreator: create any database
- public: _everyone_ is a member of public. Any permission given to public, is given to everyone!

#### User-defined server roles
Create custom roles to group permissions.  
1. Create server role
2. Grant permissions to role (GRANT, REVOKE, DENY)
3. Assign to logins (membership)

More on creating server roles: https://docs.microsoft.com/en-us/sql/t-sql/statements/create-server-role-transact-sql?redirectedfrom=MSDN&view=sql-server-ver15

### Database roles
Most security permissions are set at the database level.  
Basically, a database role is a grouping of database permissions, where users are added as member of a role.  

#### Fixed database roles
- dbo: database owner, can do anything within the database
- dbreader: can read any data
- dbwriter: can write any data
- dbddladmin: can create objects in the database
- dbaccessadmin: creates users
- dbsecurityadmin: adds permissions
- public: everyone is a member of public

#### User-defined database roles
Create roles using CREATE ROLE and give permissions using GRANT PERMISSION.  
A role will have a role owner, specifed with `AUTHORIZATION owner_name`. The owner can be either a database user or another role. The owner is able to add or remove members of the role.


#### Adding/Removing members from roles
Traditionally, members were added or removed from roles using the stored procedures sp_addrolemember and sp_droprolemember. But these are now deprecated!  
Use the ALTER ROLE statement  
`ALTER ROLE role_name ADD MEMBER database_principal`  
`ALTER ROLE role_name DROP MEMBER database_principal`  

`DENY` is used to "revoke" permissions from a user if e.g. the user is in a role that has the permission.  
`REVOKE` is used to remove both DENY and GRANT from user. 

## Firewalls
When users connect, they first reach trhe SQL Database Firewall.  
Firstly, the Server-level rules are evaluated. If the connecting IP is in range, then they can connect to the SQL Database, otherwise, the Database-level rules are evaluated.  

Server-level firewall rules: allow access to all databases on the server  
Database-level firewall rules: allow access to a specific database  

For more about SQL firewalls: https://docs.microsoft.com/en-us/azure/sql-database/sql-database-firewall-configure

## Securing securables
Examples:  
`GRANT SELECT ON schema.someTable TO user/role`  
`GRANT EXECUTE ON schema.someStoredProc TO user/role`  

Note that column-level GRANT precedes table-level denies.   

- GRANT: assigns some permission
- WITH GRANT: assigns permission to give the same permission to others
- DENY: denis the permission
- REVOKE: removes the permission or DENY

### Schemas to group secureables
By creating schemas, you can easily manage permissions to everything inside the schema.  
For instance:  
`GRANT EXECUTE ON SCHEMA::SomeSchema TO user/role`
`GRANT SELECT ON SCHEMA::SomeSchema TO user/role`  

### Ownership chains
You can grant SELECT permission on a view to a user, for tables that the user does not have access to, as long as the view and table owner is the same.  
In this manner, you can carefully expose selected data, without granting access to the tables themselves.  

More on ownerships: https://docs.microsoft.com/en-us/previous-versions/sql/sql-server-2008-r2/ms188676(v=sql.105)?redirectedfrom=MSDN

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

## Auditing

SQL Server auditing  
https://docs.microsoft.com/en-us/sql/relational-databases/security/auditing/sql-server-audit-database-engine?redirectedfrom=MSDN&view=sql-server-ver15  
https://docs.microsoft.com/en-gb/azure/sql-database/sql-database-auditing 




### Triggers
 Use can set up auditing using triggers. A trigger is basically an observer you attach to your system, that will 'trigger'  based on actions, such as insert, update, delete. There's no  trigger for SELECT.

Two types of triggers:  
1. After command (default): triggers after an action is executed
2. Instead-Of: triggers instead of the action that should have been executed

```
-- Example of After trigger

CREATE TRIGGER tr_triggerName
ON schemaName.tableName
AFTER DELETE|UPDATE|INSERT
BEGIN
  -- What to do
  -- you have access to the deleted|updated|inserted item using
  -- FROM deleted|updated|inserted
END
```  

More on triggers: https://docs.microsoft.com/en-us/sql/relational-databases/triggers/ddl-triggers?redirectedfrom=MSDN&view=sql-server-ver15

### Audit actions and action groups
Actions can be audited and grouped into action groups.