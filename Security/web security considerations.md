# Security Considerations

## Main considerations

- Confidentiality
  - What you do is not viewed by untrusted parties
- Integrity
  - What you communicate is not altered during the flow through the commucation pipeline
- Authenticity
  - What you communicate reach the intended audience.

### Aspects

- Authorization
- Authentication
- Claims
- Tokens
- Keys
- Encryption

### Learnings

- Use claim based security due to its ability to allow for granular restrictions.
- JSON Web Tokens are great for securing web APIs.

# Web Security

### HTTPS

# OAuth

OAuth is a standard for dealing with authorization in applications - NOT authentication.
OAuth does not share passwords between services but uses authorization tokens to 
prove an identity between consumers and service providers

OAuth is an authorization protocol that allows you to approve one application interacting 
with another on your behalf

### Roles

- Resource Owner
  The user
- Client
  The application that wants access to e.g. the user's account.
  The user must give the client permission.
- Auth Server
  The server that handles authorization and provides access tokens.
- Resource server
  Server that holds protected information. i.e. the server that the client wants to access.

# OpenID Connect (OIDC)

OIDC sits on top of OAuth and is created to deal with authentication.

### Components

- ID Token (JWT)
  Used for authentication
- UserInfo endpoint
- Standard scopes (permissions)
- Response type
  The type of response that the client app should receive, e.g. token (Access token) id_token (token to authenticate user)

- Identity provider
  The entity that holds the users' information, e.g. Microsoft, Google, Facebook, etc.
- Relying party
  The application that requires user authentication

### Flows

- Implicit Flow
  Used by SPA web applications
  1. The client app directs the browser to the auth server (e.g. Auth0, Microsoft, etc.)
  2. The auth server redirects to the client app at the chosen callback URL along with access and ID tokens in the URI
  3. Client app extracts the tokens from the URI and stores the relevant authentication data
  4. All subsequent calls to the resource server uses the authentication tokens
