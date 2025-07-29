---
layout: default
title: Authentication Guide
description: Comprehensive guide to OAuth 2.0 authentication with Strava API
---

# Authentication Guide

This guide covers the complete OAuth 2.0 authentication flow for the Strava API using StravaAPILibary.

## 🔐 OAuth 2.0 Overview

Strava uses OAuth 2.0 for secure API access. The library implements the **Authorization Code Flow**, which is the most secure method for web applications.

### Flow Steps

1. **Authorization Request** - Redirect user to Strava
2. **User Authorization** - User grants permissions
3. **Authorization Code** - Strava returns code to your app
4. **Token Exchange** - Exchange code for access token
5. **API Access** - Use token for API calls

## 🚀 Basic Authentication

### Step 1: Initialize Credentials

```csharp
using StravaAPILibary.Authentication;

var credentials = new Credentials(
    clientId: "your_client_id",
    clientSecret: "your_client_secret",
    scopes: "read,activity:read_all"
);
```

### Step 2: Create Authentication Instance

```csharp
var userAuth = new UserAuthentication(
    credentials: credentials,
    redirectUri: "http://localhost:8080/callback",
    scopes: "read,activity:read_all"
);
```

### Step 3: Start Authorization

```csharp
// This opens the user's browser to Strava authorization page
userAuth.StartAuthorization();
```

### Step 4: Handle the Callback

After user authorization, Strava redirects to your callback URL with an authorization code:

```
http://localhost:8080/callback?state=&code=YOUR_AUTH_CODE
```

### Step 5: Exchange Code for Token

```csharp
// Extract the authorization code from the callback URL
string authCode = "YOUR_AUTH_CODE";

// Exchange the code for an access token
bool success = await userAuth.ExchangeCodeForTokenAsync(authCode);

if (success)
{
    string accessToken = credentials.AccessToken;
    string refreshToken = credentials.RefreshToken;
    
    Console.WriteLine("✅ Authentication successful!");
    Console.WriteLine($"Access Token: {accessToken[..10]}...");
}
```

## 🔑 Token Management

### Access Tokens

- **Lifetime**: 6 hours
- **Usage**: API requests
- **Storage**: Store securely (encrypted)

### Refresh Tokens

- **Lifetime**: Indefinite (until revoked)
- **Usage**: Get new access tokens
- **Storage**: Store securely (encrypted)

### Automatic Token Refresh

```csharp
// The library automatically handles token refresh
if (credentials.IsTokenExpired())
{
    bool refreshed = await userAuth.RefreshTokenAsync();
    if (refreshed)
    {
        // Token automatically updated in credentials
        string newAccessToken = credentials.AccessToken;
    }
}
```

## 📋 Scopes

Request only the scopes your application needs:

| Scope | Description | Access Level |
|-------|-------------|--------------|
| `read` | Basic profile access | Public profile |
| `activity:read_all` | Read all activities | Private activities |
| `activity:write` | Upload activities | Create activities |
| `profile:read_all` | Detailed profile | Private profile data |
| `profile:write` | Update profile | Modify profile |

### Multiple Scopes

```csharp
var scopes = "read,activity:read_all,activity:write";
var userAuth = new UserAuthentication(credentials, redirectUri, scopes);
```

## 🛡️ Security Best Practices

### 1. Secure Storage

```csharp
// Store tokens encrypted
public class SecureTokenStorage
{
    public async Task SaveTokensAsync(string accessToken, string refreshToken)
    {
        var encryptedAccess = await EncryptAsync(accessToken);
        var encryptedRefresh = await EncryptAsync(refreshToken);
        
        await File.WriteAllTextAsync("tokens.json", JsonSerializer.Serialize(new
        {
            AccessToken = encryptedAccess,
            RefreshToken = encryptedRefresh,
            ExpiresAt = DateTime.UtcNow.AddHours(6)
        }));
    }
}
```

### 2. Environment Variables

```bash
# .env file
STRAVA_CLIENT_ID=your_client_id
STRAVA_CLIENT_SECRET=your_client_secret
STRAVA_REDIRECT_URI=http://localhost:8080/callback
```

### 3. HTTPS in Production

Always use HTTPS for production applications:

```csharp
var redirectUri = Environment.GetEnvironmentVariable("ENVIRONMENT") == "Production" 
    ? "https://yourapp.com/callback"
    : "http://localhost:8080/callback";
```

## 🔄 Token Refresh Strategy

### Automatic Refresh

```csharp
public class TokenManager
{
    private readonly Credentials _credentials;
    private readonly UserAuthentication _userAuth;
    
    public async Task<string> GetValidAccessTokenAsync()
    {
        // Check if token is expired or will expire soon
        if (_credentials.IsTokenExpired() || _credentials.WillExpireSoon())
        {
            bool refreshed = await _userAuth.RefreshTokenAsync();
            if (!refreshed)
            {
                throw new InvalidOperationException("Failed to refresh token");
            }
        }
        
        return _credentials.AccessToken;
    }
}
```

### Manual Refresh

```csharp
// Force token refresh
bool success = await userAuth.RefreshTokenAsync();

if (success)
{
    string newAccessToken = credentials.AccessToken;
    string newRefreshToken = credentials.RefreshToken;
    
    // Update stored tokens
    await UpdateStoredTokensAsync(newAccessToken, newRefreshToken);
}
```

## 🚨 Error Handling

### Common Authentication Errors

```csharp
try
{
    bool success = await userAuth.ExchangeCodeForTokenAsync(authCode);
}
catch (StravaApiException ex)
{
    switch (ex.StatusCode)
    {
        case 400:
            Console.WriteLine("Invalid authorization code or redirect URI");
            break;
        case 401:
            Console.WriteLine("Invalid client credentials");
            break;
        case 403:
            Console.WriteLine("Insufficient scopes");
            break;
        default:
            Console.WriteLine($"API Error: {ex.Message}");
            break;
    }
}
catch (Exception ex)
{
    Console.WriteLine($"Unexpected error: {ex.Message}");
}
```

### Token Validation

```csharp
public async Task<bool> ValidateTokenAsync(string accessToken)
{
    try
    {
        var athlete = await Athletes.GetAuthenticatedAthleteProfileAsync(accessToken);
        return athlete != null;
    }
    catch (StravaApiException ex) when (ex.StatusCode == 401)
    {
        return false; // Token is invalid
    }
}
```

## 🔧 Advanced Configuration

### Custom HTTP Client

```csharp
var httpClient = new HttpClient();
httpClient.Timeout = TimeSpan.FromSeconds(30);

var userAuth = new UserAuthentication(
    credentials: credentials,
    redirectUri: redirectUri,
    scopes: scopes,
    httpClient: httpClient
);
```

### Custom Token Storage

```csharp
public class CustomTokenStorage : ITokenStorage
{
    public async Task SaveTokensAsync(string accessToken, string refreshToken)
    {
        // Custom storage implementation
        await Database.SaveTokensAsync(accessToken, refreshToken);
    }
    
    public async Task<(string AccessToken, string RefreshToken)> LoadTokensAsync()
    {
        // Custom loading implementation
        return await Database.LoadTokensAsync();
    }
}
```

## 📱 Mobile Applications

For mobile apps, use a custom URL scheme:

```csharp
// iOS/Android deep link
var redirectUri = "myapp://strava-callback";

var userAuth = new UserAuthentication(credentials, redirectUri, scopes);
```

## 🔍 Debugging Authentication

### Enable Debug Logging

```csharp
// Set up logging to see detailed authentication flow
var userAuth = new UserAuthentication(credentials, redirectUri, scopes);
userAuth.DebugMode = true; // Enable detailed logging
```

### Common Issues

1. **Invalid Redirect URI**
   - Ensure redirect URI matches Strava app settings
   - Check for trailing slashes

2. **Expired Authorization Code**
   - Codes expire quickly (usually 10 minutes)
   - Request new authorization if code expires

3. **Scope Mismatch**
   - Ensure requested scopes match Strava app settings
   - Check for typos in scope names

## 📚 Related Resources

- **[Getting Started]({{ '/articles/getting-started/' | relative_url }})** - Basic setup guide
- **[Examples]({{ '/articles/examples/' | relative_url }})** - Authentication examples
- **[API Reference]({{ '/api/' | relative_url }})** - Complete API documentation
- **[Strava API Docs](https://developers.strava.com/docs/authentication/)** - Official documentation

---

**Need help with authentication? Check our examples or create an issue on GitHub! 🆘** 