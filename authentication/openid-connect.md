# OpenID Connect

Simple identity layer on top of OAuth2.  
OIDC extends and supersedes OAuth2, and is the recommended protocol.  

It's common practice to use a framework or platform service to add authentication and authorization to your application. Don't roll your own for this.

### Identity Token
Use the identity token to sign into a client application. That same application can use an access token to request data from an API.

### Access Token


### Parties
- IdP: the Identity Provider that a verifies the identity of a user.
- Client application: the application that requires to know the user's identity. 