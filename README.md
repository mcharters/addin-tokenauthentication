# SKY Add-in token validation
This library provides developers building SKY Add-ins in .NET with a mechanism to validate user identity tokens provided by host applications. 

# Installation

This library is distributed as a NuGet package named [Blackbaud.Addin.TokenAuthentication](https://www.nuget.org/packages/Blackbaud.Addin.TokenAuthentication).

# SKY Add-ins
SKY Add-ins support a single-sign-on (SSO) mechanism that can be used to correlate the Blackbaud user with a user in the add-in's native system.

Within the [SKY Add-in Client JavaScript library](https://github.com/blackbaud/sky-addin-client), the `AddinClient` class provides a `getUserIdentityToken` function for getting a short-lived "user identity token" from the host page. This token is a signed value that is issued to the developer's application and represents the Blackbaud user's identity.

The general flow is that when an add-in is instantiated, it can request a user identity token from the host page using the `getUserIdentityToken` function. The host will in turn request a user identity token from the SKY API OAuth 2.0 service. The token (a JWT) will be addressed to the developer's registered application, and will contain the user's unique identifier (BBID). The OAuth service will return the token to the host, and the host will pass the token to the add-in's iframe. The add-in can then pass the token to its own backend, where it can be validated and used to look up a user in the add-in's native system. If a user mapping exists, then the add-in can present content to the user. If no user mapping exists, the add-in can prompt the user to login. Once the user's identity in the native system is known, the add-in can persist the user mapping so that on subsequent loads the user doesn't have to log in again (even across devices).

Note that the user identity token is a JWT that is signed by the SKY API OAuth 2.0 service, but it cannot be used to make calls to the SKY API. In order to make SKY API calls, a proper SKY API access token must be obtained using one of the supported OAuth flows.

This flow is illustrated below:

![flow](https://sky.blackbaudcdn.net/skyuxapps/uiextensibility-docs/assets/add-in-sso.dadfd9a6ab3e64c24ba984751efd43aef73ee67c.png)

Add-ins can make the following request upon initialization to obtain a user identity token (typically handled within the `init` callback):

```js
var client = new AddinClient({...});
client.getUserIdentityToken().then((token) => {
  var userIdentityToken = token;
  . . .
});
```

# Validating the user identity token 
Before looking for a user mapping, add-in developers should first validate the signature of the user identity token against the OpenIDConnect endpoint within SKY API OAuth 2.0 service. This prevents certain types of attack vectors and provides a mechanism for the add-in to securely convey the Blackbaud user's identity to its own backend.

[SKY API OpenIDConnect configuration](https://oauth2.sky.blackbaud.com/.well-known/openid-configuration)

# Example validation code

```csharp
// this represents the user identity token returned from getUserIdentityToken()
var rawToken = "(raw token value)";

// this is the ID of the developer's application, obtained from the SKY API developer portal
var applicationId = "(some application ID)";

// create and validate the user identity token
UserIdentityToken uit;
try
{
    uit = await UserIdentityToken.ParseAsync(rawToken, applicationId);

    // if valid, the UserId property contains the Blackbaud user's ID
    var userId = uit.UserId;
    
    // if valid, the Email property contains the Blackbaud user's email address
    var email = uit.Email;
    
    // if valid, the FamilyName property contains the Blackbaud user's last name
    var familyName = uit.FamilyName;
    
    // if valid, the GivenName property contains the Blackbaud user's first name
    var givenName = uit.GivenName;

    //if valid, the EnvironmentId property contains the environment ID
    var envId = uit.EnvironmentId;
}
catch (TokenValidationException ex)
{
    // process the exception
}
```

Once the token has been validated, the add-in's backend will know the Blackbaud user's ID and can determine if a mapping exists for the user in the add-in's native system. If a mapping exists, then the add-in's backend can immediately present the content for the add-in. If no user mapping exists, the add-in can prompt the user to login.

For more information on creating SKY Add-ins, please see https://developer.blackbaud.com/skyapi/docs/addins.
