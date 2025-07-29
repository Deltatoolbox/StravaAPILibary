---
layout: default
title: Getting Started
description: Quick start guide for StravaAPILibary
---

# Getting Started with StravaAPILibary

Welcome to the StravaAPILibary! This guide will help you get up and running quickly with our .NET library for Strava API integration.

## 📦 Installation

### Clone from GitHub Repository

The recommended way to use StravaAPILibary is to clone it directly from our GitHub repository:

```bash
git clone https://github.com/Deltatoolbox/StravaAPILibary.git
cd StravaAPILibary
```

### Add to Your Project

After cloning, you can add the library to your project by referencing the project files:

#### Option 1: Add as Project Reference

```xml
<!-- In your .csproj file -->
<ItemGroup>
  <ProjectReference Include="path/to/StravaAPILibary/StravaAPILibary.csproj" />
</ItemGroup>
```

#### Option 2: Copy Source Files

You can also copy the source files directly into your project:

1. Copy the `API/` folder to your project
2. Copy the `Authentication/` folder to your project  
3. Copy the `Models/` folder to your project
4. Add the necessary using statements to your code

### Build the Library

```bash
# Build the library
dotnet build StravaAPILibary.csproj

# Or build in Release mode
dotnet build StravaAPILibary.csproj --configuration Release
```

## 🚀 Quick Start

### 1. Set Up Your Strava App

First, create a Strava application at [Strava API Settings](https://www.strava.com/settings/api):

1. Go to your Strava account settings
2. Navigate to "API" section
3. Create a new application
4. Note your **Client ID** and **Client Secret**

### 2. Basic Authentication

```csharp
using StravaAPILibary.Authentication;
using StravaAPILibary.API;

// Initialize credentials
var credentials = new Credentials(
    clientId: "your_client_id",
    clientSecret: "your_client_secret",
    scopes: "read,activity:read_all"
);

// Create authentication instance
var userAuth = new UserAuthentication(
    credentials: credentials,
    redirectUri: "http://localhost:8080/callback",
    scopes: "read,activity:read_all"
);

// Start the authorization process
userAuth.StartAuthorization();
```

### 3. Handle the Authorization Code

After the user authorizes your app, Strava will redirect to your callback URL with an authorization code:

```csharp
// Exchange the authorization code for an access token
bool success = await userAuth.ExchangeCodeForTokenAsync("your_auth_code");

if (success)
{
    string accessToken = credentials.AccessToken;
    Console.WriteLine($"Access Token: {accessToken}");
}
```

### 4. Make API Calls

```csharp
// Get athlete profile
var athlete = await Athletes.GetAuthenticatedAthleteProfileAsync(accessToken);
Console.WriteLine($"Athlete: {athlete.FirstName} {athlete.LastName}");

// Get recent activities
var activities = await Activities.GetAthletesActivitiesAsync(accessToken);
foreach (var activity in activities)
{
    Console.WriteLine($"Activity: {activity.Name} - {activity.Distance}m");
}
```

## 🔐 Authentication Flow

The library supports the OAuth 2.0 authorization code flow:

1. **Authorization Request**: Redirect user to Strava for authorization
2. **Callback Handling**: Receive authorization code from Strava
3. **Token Exchange**: Exchange code for access token
4. **API Calls**: Use access token for API requests

## 📋 Prerequisites

- **.NET 6.0** or later
- **Git** for cloning the repository
- **Strava Account** with API access
- **Strava Application** (Client ID and Secret)

## 🛠️ Configuration

### Environment Variables

For production, store your credentials securely:

```bash
# .env file
STRAVA_CLIENT_ID=your_client_id
STRAVA_CLIENT_SECRET=your_client_secret
STRAVA_REDIRECT_URI=http://localhost:8080/callback
```

### Configuration Class

```csharp
public class StravaConfig
{
    public string ClientId { get; set; }
    public string ClientSecret { get; set; }
    public string RedirectUri { get; set; }
    public string Scopes { get; set; } = "read,activity:read_all";
}
```

## 🔧 Error Handling

The library provides comprehensive error handling:

```csharp
try
{
    var activities = await Activities.GetAthletesActivitiesAsync(accessToken);
}
catch (StravaApiException ex)
{
    Console.WriteLine($"API Error: {ex.Message}");
    Console.WriteLine($"Status Code: {ex.StatusCode}");
}
catch (Exception ex)
{
    Console.WriteLine($"Unexpected error: {ex.Message}");
}
```

## 📚 Next Steps

- **[Authentication Guide]({{ '/articles/authentication/' | relative_url }})** - Detailed OAuth flow
- **[Examples]({{ '/articles/examples/' | relative_url }})** - Real-world usage examples
- **[API Reference]({{ '/api/' | relative_url }})** - Complete API documentation

## 🆘 Need Help?

- Check the **[API Reference]({{ '/api/' | relative_url }})** for detailed method documentation
- Review **[Examples]({{ '/articles/examples/' | relative_url }})** for common use cases
- Visit the **[GitHub Repository](https://github.com/Deltatoolbox/StravaAPILibary)** for issues and contributions

## 🤝 Contributing

Since you're using the source code directly, you can easily contribute to the project:

1. Fork the repository
2. Make your changes
3. Submit a pull request
4. Help improve the library for everyone!

---

**Ready to build amazing Strava applications? Let's get started! 🚀** 