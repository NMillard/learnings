# ASP.NET Authentication

## Identity

Is a library provided by Microsoft  
The aim of ASP.NET Core Identity is to be provid user-store and user-management capabilities

The Core Identity library is structured in layers:

- Data Access Layer -> Stores: IUserStore and IRoleStore
- Business Logic -> Managers: UserManager and RoleManager
- Extensions -> Abstracted Managers: SignInManager, SecurityStamp, etc.
- Entities -> Classes: User, Role <- used by the Stores and Managers

It contains helper functionality for:  
password hashing  
user and password validation  
password and email confirmation  
User lockout  
Multi-Factor Authentication  
External Providers

## Identity data

- Identification
  Username, password, two-factor keys
- Personal information
  Name, gender
- What you are
  Roles, etc.

Identity does not equal permissions.  
Identity is transient of applications whereas permissions are application specific

## IdentityUser class

Designed to take full advantage of the every feature of ASP.NET Core Identity
You should create a custom user class that inherts this class to allow for extensibility

## IdentityUserClaim class

Designed to store arbitrary claims for a user - i.e. for instance application specific claims
Has a ToClaim() method that returns the claim as a System Security Claim to be used with Claims principals and identities

## IdentityUserLogin class

Models an external identity that links back to a local user entity

## IdentityUserToken class

Allows us to keep track of tokens returned from external identity providers

## IdentityRole class

Model for identity roles - a role can the thought of as a group that can be assigned claims

## IdentityDbContext class

DbContext class that has been configured to handle Identity and will create all the required tables on migration
You should create a DbContext that inherts from this class

## UserClaimsPrincipalFactory class

Create claims principal based on an Identity object  
Provide own implementation of UserClaimsPrincipalFactory to add custom claims to the principal when created  
Note: the ClaimsIdentity AuthenticationType is hardcoded to Identity.Application

## ClaimsIdentity and ClaimsPrincipal

A claims principal can have multiple claims identities. e.g. a principal may have an identity on Facebook, Google, Microsoft, etc.  
The authentication type MUST be set in order for IsAuthenticated() to return true - reason for this is, the HttpContext
may have a ClaimsPrincipal, but if the AuthenticationType is null, it denotes that the claims principal hasn't been authenticated
through a trusted provider, and may just be an anonymous user

## SignInManager class

Used to easily implement login and two-factor authentication. But hides a lot of the implementation and is not as flexible. Should only be used if
not having time to do authentication yourself

# ASP.NET Core Sign in

In ASP.NET, users are logged in by setting the HttpContext.User object
This is done by first creating a ClaimsIdentity with an AuthenticationType and a set of claims. Next, a ClaimsPrincipal is created using the ClaimsIdentity
User is signed in by calling SignInAsync("Schema", ClaimsPrincipal) on the HttpContext - "Schema" is the schema that is used for authentication
The authentication of a schema is configured in the Startup.cs AddAuthentication() or AddIdentity()

## Cookie authentication

## Json Web Token authentication

## External Identity authentication

Data returned from external identity providers should be stored temporarily. In ASP.NET Core, the loginProvider is the authentication scheme to be used

# Security

## Password reset link

Must adhere to four criteria

1. Only avaiable for a specific user
2. Time sensitive - link will only be accessible for a short period of time
3. The link must contain some randomness to be "un-guessable"
4. One time use only
   ^ A token fulfilling these criteria can be generated using the UserManager's GeneratePasswordResetTokenAsync() method

## Email confirmation link

Must adhere to same criteria as the password reset link  
Token can be generated using the UserManager's GenerateEmailConfirmationTokenAsync() method

## UserManager tokens

Token lifetime is dictated by the DataProtectionTokenProvider, and can be set by configuring the DataProtectionTokenProviderOptions  
To have different lifetimes for token types, you need to implement your own DataProtectionTokenProvider and Options, and register this using
AddTokenProvider<TTokenProvider>("token_provider_name") method on AddIdentity() and in the AddIdentity options.Tokens.\*

## Passwords

Passwords must be hashed so they are non-reversible without the original plaintext  
Passwords should have added a random salt so no hash is the same, even if the plaintext password is the same  
The default password hasher in ASP.NET Core uses PBKDF2 with HMAC-SHA256 algorithm, 128-bit salt, 256-bit subkey

## Password Hasher

The default password hasher in ASP.NET Core can be switched out with a stronger hasher that can resist GPU-based attacks better
To switch the hasher, in the Startup.cs, add services.AddScoped<IPasswordHasher<TUser>, NewPasswordHasher<TUser>>()  
The switch should be made after the configuration of Identity

## Password validator

Custom password validators can be created in ASP.NET Core by implementing your own IPasswordValidator<TUser> and registering it on
the AddIdentity() using AddPasswordValidator<TPasswordValidator>() method

## Multi Step/Factor Authentication

One-step verification - Sign in using just password  
Two-step verification - Sign in using password and e.g. a passphrase sent to another device  
Two-Factor Authentication - Sign in using password and an other means of authenticating who you are, e.g. finger-print, iris scan, etc.  
Should use an Authenticator App from e.g. google or microsoft

## Two-Step Verification

ASP.NET Core has two 2SV token providers, EmailTokenProvider and PhoneNumberTokenProvider. Both share a base implementation of TotpSecruityStampBasedTokenProvider  
TOTP stands for Time-based One Time Password  
Generated tokens last for 3 minutes and uses the user's email or phone number and the user's current security stamp

## Two-Factor Authentication

After successfully logging in, users can add a trusted device used for two-factor authentication  
A Authenticator Key must be generated and used on the Authenticator app, like Microsoft Authenticator or Google's  
When logging in using 2FA, both the TOTP from the Authenticator app in combination with the Authenticator Key must be provided

UserManager provides two handy methods for this: GetAuthenticatorKeyAsync(user) to get the Authenticator Key and VerifyTwoFactorTokenAsync(user, tokenProvider, token) to verify the token
