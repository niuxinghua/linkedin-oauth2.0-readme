# linkedin-oauth2.0-readme
At LinkedIn, we value the integrity and security of our members' data above all else.  In order for your applications to access LinkedIn member data and/or act on their behalf, they must be authenticated.  To make this process as easy as possible, LinkedIn relies on the industry standard OAuth 2.0 protocol for granting access. 

Follow these steps to enable your application to make authenticated API calls to LinkedIn using OAuth 2.0:

Step 1 — Configuring your LinkedIn application
If you have not already done so, create an application. If you have an existing application, select it to modify its settings.

To prevent fraudulent transactions during the authentication process, we will only communicate with URLs that you have identified as trusted endpoints.  Ensure the "OAuth 2.0 Redirect URLs" field for your application contains a valid callback URL to your server that is listening to complete your portion of the authentication workflow.

Please note that:

We strongly recommend using HTTPS whenever possible
URLs must be absolute (e.g. "https://example.com/auth/callback", not "/auth/callback")
URL arguments are ignored (i.e. https://example.com/?id=1 is the same as https://example.com/)
URLs cannot include #'s (i.e. "https://example.com/auth/callback#linkedin" is invalid)
Sample OAuth2.0 callback value
Once you save your configuration, your application will be assigned a unique "Client ID" (otherwise known as Consumer Key or API key) and "Client Secret" value.  Make note of these values — you will need to integrate them into the configuration files or the actual code of your application, for example:
API Key and Secret Key example
Important!

At the risk of your own application's security, DO NOT share your Client Secret value with anyone!  This includes posting it in support forums for help with your application.
Step 2 — Request an Authorization Code
Once your application is properly configured, it's time to request an authorization code.  The authorization code is not the final token that you use to make calls to LinkedIn with.  It is used in the next step of the OAuth 2.0 flow to exchange for an actual access token.  This is an important step because it provides assurance directly from LinkedIn to the user that permission is being granted to the correct application, with the agreed-upon access to the member's LinkedIn profile.

Redirecting the User

To request an authorization code, you must direct the user's browser to LinkedIn's OAuth 2.0 authorization endpoint.  Once the request is made, one of the following two situations will occur:

If the user has not previously accepted the application's permission request, or the grant has expired or been manually revoked by the user, the browser will be redirected to LinkedIn's authorization screen (as seen below).  When the user completes the authorization process, the browser is redirected to the URL provided in the redirect_uri query parameter.
If there is a valid existing permission grant from the user, the authorization screen is by-passed and the user is immediately redirected to the URL provided in the redirect_uri query parameter.
Note that if you ever change the scope permissions that your application requires, your users will have to re-authenticate to ensure that they have explicitly granted your application all of the permissions that it requests on their behalf.
GET
https://www.linkedin.com/oauth/v2/authorization
Parameter	Description	Required
response_type	The value of this field should always be:  code	Yes
client_id	The "API Key" value generated when you registered your application.	Yes
redirect_uri	
The URI your users will be sent back to after authorization.  This value must match one of the defined OAuth 2.0 Redirect URLs in your application configuration.

e.g. https://www.example.com/auth/linkedin
Yes
state	
A unique string value of your choice that is hard to guess. Used to prevent CSRF. 

e.g. state=DCEeFWf45A53sdfKef424
Yes
scope	
A URL-encoded, space delimited list of member permissions your application is requesting on behalf of the user.  If you do not specify a scope in your call, we will fall back to using the default member permissions you defined in your application configuration.

e.g. scope=r_fullprofile%20r_emailaddress%20w_share
 
See Understanding application permissions and the Best practices guide for additional information about scopes.
Optional
sample call
https://www.linkedin.com/oauth/v2/authorization?response_type=code&client_id=123456789&redirect_uri=https%3A%2F%2Fwww.example.com%2Fauth%2Flinkedin&state=987654321&scope=r_basicprofile
The User Experience

Once redirected, the user will be presented with LinkedIn's authentication dialog box.  This identifies your application as well as outlines the particular member permissions that your application has requested.  If desired, the logo and application name can be changed in your application configuration.
Authentication dialog box
Application is approved

By providing valid LinkedIn credentials and clicking on the "Allow Access" button, the user is approving your application's request to access their member data and interact with LinkedIn on their behalf.  This approval instructs LinkedIn to redirect the user back to the callback URL that you defined in your redirect_uri parameter. 

Attached to the redirect_uri will be two important URL arguments that you need to read from the request:
code — The OAuth 2.0 authorization code.
state — A value used to test for possible CSRF attacks.
The code is a value that you will exchange with LinkedIn for an actual OAuth 2.0 access token in the next step of the authentcation process.  For security reasons, the authorization code has a very short lifespan and must be used within moments of receiving it - before it expires and you need to repeat all of the previous steps to request another.
Before you accept the authorization code, your application should ensure that the value returned in the state parameter matches the state value from your original authorization code request. This ensures that you are dealing with the real original user and not a malicious script that has somehow slipped into the middle of your authentication flow.  If the state values do not match, you are likely the victim of a CSRF attack and you should throw an HTTP 401 error code in response.
Application is rejected

If the user choses to cancel, or the request fails for any other reason, their client will be redirected back to your redirect_uri callback URL with the following additional query parameters appended:

error - A code indicating the type of error. Two available strings are:
user_cancelled_login - The user refused to login into LinkedIn account.
user_cancelled_authorize - The user refused to authorize permissions request from your application.
error_description - A URL-encoded textual description that summarizes error.
state - A value passed by your application to prevent CSRF attacks.
Step 3 — Exchange Authorization Code for an Access Token
The final step towards obtaining an Access Token is for your application to ask for one using the Authorization Code it just acquired.  This is done by making the following "x-www-form-urlencoded" HTTP POST request:

POST
https://www.linkedin.com/oauth/v2/accessToken
Parameter	Description	Required
grant_type	The value of this field should always be:  authorization_code
Yes
code	The authorization code you received from Step 2.
Yes
redirect_uri	The same 'redirect_uri' value that you passed in the previous step.
Yes
client_id	The "API Key" value generated Step 1.
Yes
client_secret	
The "Secret Key" value generated in Step 1. 
 
Follow the Best Practices guide for handing your client_secret value.
Yes
sample call (secure approach)
POST /oauth/v2/accessToken HTTP/1.1
Host: www.linkedin.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&code=987654321&redirect_uri=https%3A%2F%2Fwww.myapp.com%2Fauth%2Flinkedin&client_id=123456789&client_secret=shhdonottell
Access Token Response

A successful Access Token request will return a JSON object containing the following fields:

access_token — The access token for the user.  This value must be kept secure, as per your agreement to the API Terms of Use.
expires_in — The number of seconds remaining, from the time it was requested, before the token will expire.  Currently, all access tokens are issued with a 60 day lifespan.
Step 4 — Make authenticated requests
Once you've obtained an Access Token, you can start making authenticated API requests on behalf of the user. This is accomplished by including an "Authorization" header in your HTTP call to LinkedIn's API.  Here is a sample HTTP request including the header value that includes the token:
sample call
GET /v1/people/~ HTTP/1.1
Host: api.linkedin.com
Connection: Keep-Alive
Authorization: Bearer AQXdSP_W41_UPs5ioT_t8HESyODB4FqbkJ8LrV_5mff4gPODzOYR
Invalid Tokens

If you make an API call using an invalid token, you will receive a "401 Unauthorized" response back from the server.  A token could be invalid and in need of regeneration because:

It has expired.
The user has revoked the permission they initially granted to your application.
You have changed the member permissions (scope) your application is requesting.
A subsequent OAuth2 flow that generated a new access token. The previous token will be invalidated.
Since a predictable expiry time is not the only contributing factor to token invalidation, it is very important that you code your applications to properly handle an encounter with a 401 error by redirecting the user back to the start of the authorization workflow.
Step 5 — Refresh your Access Tokens
To protect our member's data, LinkedIn does not generate excessively long-lived access tokens.  You should ensure your application is built to handle refreshing user tokens before they expire, to avoid having to unnecessarily send your users through the authorization process to re-gain access to their LinkedIn profile.

Refreshing a token

To refresh an Access Token, simply go through the authorization process outlined in this document again to fetch a new token.  During the refresh workflow, provided the following conditions are met, the authorization dialog portion of the flow is automatically skipped and the user is redirected back to your callback URL, making acquiring a refreshed access token a seamless behind-the-scenes user experience:
The user is still logged into www.linkedin.com
The user's current access token has not expired
If the user is no longer logged in to www.linkedin.com, or their access token has expired, they will be sent through the normal authorization process outlined at the start of this document.

Note: If your application is currently on OAuth1 and want to migrate to OAuth2, please refer to the legacy URLs for a seamless behind-the-scenes user experience.

Understanding application permissions
All REST API calls require certain permissions to be granted from the LinkedIn member before they can be made.  This system ensures that members are made aware of what an application could possibly access or do on their behalf before approving it. 

The permissions that members are asked to grant are determined based on the permissions you tell your application to ask for during the OAuth 2.0 authentication process.  They can be specified within the LinkedIn application configuration itself, or they can be explicitly requested using the scope argument during the authorization step of the OAuth 2.0 process.

If your application requires multiple permissions to access all the data it requires, your users will be required to accept all of them to proceed.  It is not possible for users to accept only a subset of the requested application permissions.  Ensure that your application requests the fewest necessary permissions, to provide the best experience for the user.

