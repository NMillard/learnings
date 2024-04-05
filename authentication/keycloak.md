# KeyCloak

Open Source identity and access management service.
- Single sign on
- Federated logins
- Multi Factor authentication

The KeyCloak server holds a user's signed-in session and allows easy single-sign on across multiple applications. The applications redirects the users to authenticate against the KeyCloak server, and, if the user is already signed in, then they don't have to type in their username and password again.


## KeyCloak specific terminology

### Realm
Top-level, self-contained security environment, accesses a space where apps, users, and identity providers are managed independently.  

### Client
The applications that you want to use KeyCloak for authentication and authorization with.

### Users
Users that interact with applications and services secured by keycloak. Can be assigned to groups and roles to control access to resources.

### Groups
Container for users. Groups can be assigned roles, so that all users within a group inherit those roles.

### Scopes
The opposite of a role - a scope defines what access a client (application) has to a user's information. Can be used to restrict information a client can access about a user.