# tmp-tst-abc


ps-> 19713a9c6fD$aa#

```csharp
using System;
using RestSharp;

public class OktaApiHelper
{
    private readonly string baseUrl;
    private readonly string clientId;
    private readonly string clientSecret;

    public OktaApiHelper(string baseUrl, string clientId, string clientSecret)
    {
        this.baseUrl = baseUrl;
        this.clientId = clientId;
        this.clientSecret = clientSecret;
    }

    private RestClient CreateRestClient()
    {
        return new RestClient(baseUrl);
    }

    public string GetAuthorizationHeader()
    {
        var base64ClientIdSecret = Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes($"{clientId}:{clientSecret}"));
        return $"Basic {base64ClientIdSecret}";
    }

    public string GetAccessToken(string username, string password, string scope)
    {
        var client = CreateRestClient();
        var request = new RestRequest("/token", Method.POST);
        request.AddHeader("Authorization", GetAuthorizationHeader());
        request.AddParameter("grant_type", "password");
        request.AddParameter("username", username);
        request.AddParameter("password", password);
        request.AddParameter("scope", scope);

        var response = client.Execute(request);
        if (response.IsSuccessful)
        {
            dynamic jsonResponse = Newtonsoft.Json.JsonConvert.DeserializeObject(response.Content);
            return jsonResponse.access_token;
        }
        else
        {
            throw new Exception($"Failed to get access token: {response.Content}");
        }
    }

    public void RevokeAccessToken(string accessToken)
    {
        var client = CreateRestClient();
        var request = new RestRequest("/revoke", Method.POST);
        request.AddParameter("token", accessToken);
        request.AddParameter("token_type_hint", "access_token");

        client.Execute(request);
    }

    public void RevokeRefreshToken(string refreshToken)
    {
        var client = CreateRestClient();
        var request = new RestRequest("/revoke", Method.POST);
        request.AddParameter("token", refreshToken);
        request.AddParameter("token_type_hint", "refresh_token");

        client.Execute(request);
    }
}

/****




****/


var oktaHelper = new OktaApiHelper("https://your-okta-domain/oauth2/default", "your-client-id", "your-client-secret");
var accessToken = oktaHelper.GetAccessToken("username", "password", "openid profile");
// Use the access token as needed
oktaHelper.RevokeAccessToken(accessToken);
```

**Authentication Endpoints**:
    
    *   `/authn`: Initiates authentication flows.
    *   `/sessions`: Manages user sessions.
    *   `/users`: Manages user authentication and credentials.
    *   `/groups`: Manages groups and group membership.
*   **Authorization Endpoints**:
    
    *   `/authorize`: Initiates OAuth 2.0 authorization flows.
    *   `/token`: Issues OAuth 2.0 tokens (access token, refresh token).
    *   `/introspect`: Introspects an OAuth 2.0 token.
    *   `/revoke`: Revokes OAuth 2.0 tokens.
    *   `/logout`: Logs out users and ends sessions.
    *   `/scopes`: Manages OAuth 2.0 scopes.
*   **Multi-Factor Authentication (MFA) Endpoints**:
    
    *   `/factors`: Manages MFA factors for users.
    *   `/verify`: Verifies MFA factors during authentication.
*   **Password Policy Endpoints**:
    
    *   `/password`: Manages password-related operations (reset, change, etc.).
*   **Social Authentication Endpoints**:
    
    *   `/authorizationServers`: Manages authorization servers for social authentication.
    *   `/identityProviders`: Manages identity providers for social authentication.
*   **Device and Session Management**:
    
    *   `/sessions`: Manages user sessions and session cookies.
    *   `/sessions/me`: Retrieves information about the current user's session.
    *   `/devices`: Manages trusted devices for MFA.
*   **User Consent and Authorization**:
    
    *   `/consent`: Handles user consent for OAuth 2.0 applications.
    *   `/authorizations`: Manages application authorizations and permissions.
*   **Security Events**:
    
    *   `/logs`: Retrieves security logs and audit trails.
    *   `/events`: Manages event hooks for security events.
*   **User Management and Profiles**:
    
    *   `/users`: Manages user accounts, profiles, and attributes.
    *   `/groups`: Manages groups, roles, and group memberships.
*   **Token Management**:
    
    *   `/tokens`: Manages OAuth 2.0 tokens, token lifetimes, and policies.



Install-Package Microsoft.IdentityModel.JsonWebTokens


```
using System;
using System.IdentityModel.Tokens.Jwt;
using System.Net.Http;
using System.Threading.Tasks;
using Microsoft.IdentityModel.JsonWebTokens;
using Microsoft.IdentityModel.Protocols;
using Microsoft.IdentityModel.Protocols.OpenIdConnect;
using Microsoft.IdentityModel.Tokens;

public class OktaJwtVerifier
{
    private readonly string issuer;
    private readonly ConfigurationManager<OpenIdConnectConfiguration> configurationManager;

    public OktaJwtVerifier(string issuer, string jwksUri)
    {
        this.issuer = issuer;
        this.configurationManager = new ConfigurationManager<OpenIdConnectConfiguration>(
            jwksUri,
            new OpenIdConnectConfigurationRetriever());
    }

    public async Task<SecurityToken> VerifyJwtTokenAsync(string jwtToken)
    {
        var openIdConfig = await configurationManager.GetConfigurationAsync();

        var validationParameters = new TokenValidationParameters
        {
            ValidIssuer = issuer,
            ValidAudience = openIdConfig.TokenEndpoint,
            IssuerSigningKeys = openIdConfig.SigningKeys,
            ValidateIssuerSigningKey = true,
            ValidateIssuer = true,
            ValidateAudience = true,
            RequireSignedTokens = true,
            RequireExpirationTime = true
        };

        var tokenHandler = new JwtSecurityTokenHandler();
        SecurityToken validatedToken;

        try
        {
            tokenHandler.ValidateToken(jwtToken, validationParameters, out validatedToken);
            return validatedToken;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Failed to validate token: {ex.Message}");
            throw;
        }
    }
}



var oktaVerifier = new OktaJwtVerifier("https://your-okta-domain/oauth2/default", "https://your-okta-domain/oauth2/default/v1/keys");
string jwtToken = "your_jwt_token_here";

try
{
    var validatedToken = await oktaVerifier.VerifyJwtTokenAsync(jwtToken);
    Console.WriteLine("Token is valid!");
    // Optionally, you can access token claims from validatedToken.Claims
}
catch (Exception ex)
{
    Console.WriteLine($"Token validation failed: {ex.Message}");
}

```




