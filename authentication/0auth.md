# OAuth2

## Aspects
- Protected resource
- Client (requesting application)
- Resource owner (user)
- Authorization server (trusted by all parties involved)

## The authorization flow
Code grant type:  
1. Client app makes an authorization request. This redirects the user to the authorization server.
2. Resource owner authorizes against the authorization server
3. Resource owner receives and authorization grant. The grant is provided to the client app.
4. The client app makes a request to the authorization server using the grant and a valid client id.
5. The authorization server returns an access token.
6. The client app makes a request to the protected resource with the access token.
7. The protected resource server validates the access token against the authorization server. This could be e.g. by validating a signature using a public key.

It's important to note that the authorization server, that generate access tokens, signs the tokens with a private key that only the authorization server knows.

The authorization server publishes a public key anyone can use to verify the acess token's integrity.

Tokens must be validated for every single request.


## Code grant type
`/authorize`: endpoint to authorize a client. Includes a few query parameters such as `?response_type=code`, `&client_id=<someid>`, `&redirect_uri=https://mysite.com/auth-callback`, and optionally `&state=xyz`. The callback endpoint must be registered with the authorization server. The state is echoed back.

`/token`: request an access token using the code returned from the `/authorize` response. POST using form urlencoded content-type. The client needs to authenticate itself using client id and client secret added as basic credentials to the `Authorization` header.

To exchange and authorization code for an access token you need to provide `grant_type=authorization_code`, `&code=some-code`, `&redirect-uri=https://mysite.com/auth-callback` as key-value in the post body.

Authorization to the token endpoint may take the standard basic base64 authentication style, or, the OAuth style that is `base64(urlformencode(client_id):urlformencode(client_secret))`.

The token response is json format, looking like this:

```json
{
    "access_token": "ey....token",
    "token_type": "Bearer",
    "expires_in": 3600,
    "scope": "api.read"
}
```

## Implicit grant type
__Not recommended__, see [CORS+PKCE instead](https://developer.okta.com/blog/2019/08/22/okta-authjs-pkce).  

Is made specifically for client apps that can't keep a secret, such as Single Page Applications served in a browser. This is also known as "public" clients. There's no explicit client authentication here.

The client requests a token directly at the `/authorize` endpoint using `?response_type=token`, `&client_id=<someid>`, and `&redirect_uri=https://mysite.com/auth-callback`. The redirect URI is in this case the main defence against unauthorized requests.

The authorization server responds with the access_token in a URL fragment: `#access_token=xyzabc`


## Authorization with PKCE
[RFC7636](https://oauth.net/2/pkce/)  

Proof Key for Code Exchange (PKCE) links an authorization request to a token request, by using a proof key that only the initial requester would have known.

To kick off the process, the client generates a random value called code_verifier. Then, the `authorize` request is made with a SHA256 hash of the code_verifier called a code_challenge. The subsequent `token` request is made with the original code_verifier value.

## Backend for Frontend
Recommended approach when using browser-based apps.  

Browser app communicates with a backend running on the same domain using SameSite cookies. The backend handles all interactions with the authorization server, initiating the authorization request, host redirect uri, and swap authorization grant for an access token.

## Privte use URI scheme
Native client - used to limit the attack surface.

Create a custom URI scheme for the app, e.g. `com.myapp.webapp:/cb` instead of using HTTPs. Register this domain to open the application when visited with a browser.

This is not supported on all platforms.


# Refresh token
- [Read more](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/)

Long lived token, used by the client application for getting new access tokens when the current one expires. Saves the user from having to repeatedly reauthorize.

Must be kept secret. It's required to ask for consent to get a refresh token.

Triggering the swap access token for a fresh one may be done when receiving a 401 response from the protected resource server. In C#, you'd typically use an HTTP handler that intercepts 401 responses.

To get a new access token using the refresh token, you'll make a request to the `/token` endpoint with the urlencoded body `grant_type=refresh`, `&refresh_token=mytokenxyz`.

The refresh token should never be exposed to the browser.

# Well-known endpoint
`/.well-known/oauth-authorization-server`