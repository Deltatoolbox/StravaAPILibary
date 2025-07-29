# Getting Started with StravaAPILibary

This guide will walk you through setting up and using the StravaAPILibary to interact with the Strava API.

## Prerequisites

Before you begin, ensure you have:

- **.NET 8.0 SDK** or later installed
- **Visual Studio 2022** or **JetBrains Rider** (recommended IDEs)
- **Strava API credentials** (Client ID and Client Secret)
- **Internet connection** for API calls

## Step 1: Install the Library

### Using NuGet Package Manager

1. Right-click on your project in Solution Explorer
2. Select **Manage NuGet Packages**
3. Search for `StravaAPILibary`
4. Click **Install**

### Using Package Manager Console

```powershell
Install-Package StravaAPILibary
```

### Using .NET CLI

```bash
dotnet add package StravaAPILibary
```

### Using PackageReference

Add this to your `.csproj` file:

```xml
<PackageReference Include="StravaAPILibary" Version="1.0.0" />
```

## Step 2: Set Up Strava API Credentials

### Create a Strava Application

1. Visit [Strava API Settings](https://www.strava.com/settings/api)
2. Click **Create Application**
3. Fill in the required information:
   - **Application Name**: Your app name
   - **Category**: Choose appropriate category
   - **Website**: Your website URL
   - **Authorization Callback Domain**: `localhost` (for development)
4. Click **Create**
5. Note your **Client ID** and **Client Secret**

### Configure Redirect URI

For development, you can use:
- `http://localhost:8080/callback`
- `http://localhost:3000/callback`
- `http://localhost:5000/callback`

## Step 3: Basic Authentication Flow

### Create Your First Application

```csharp
using StravaAPILibary.Authentication;
using StravaAPILibary.API;

class Program
{
    static async Task Main(string[] args)
    {
        // 1. Set up your credentials
        var credentials = new Credentials(
            "your_client_id_here",
            "your_client_secret_here", 
            "read,activity:read_all"
        );

        // 2. Create authentication instance
        var userAuth = new UserAuthentication(
            credentials,
            "http://localhost:8080/callback",
            "read,activity:read_all"
        );

        // 3. Start the authorization process
        Console.WriteLine("Starting Strava authorization...");
        userAuth.StartAuthorization();

        // 4. Wait for user to complete authorization
        Console.WriteLine("Please complete the authorization in your browser.");
        Console.WriteLine("After authorization, copy the authorization code from the URL.");
        Console.Write("Enter authorization code: ");
        
        string authCode = Console.ReadLine() ?? string.Empty;

        // 5. Exchange code for access token
        if (!string.IsNullOrWhiteSpace(authCode))
        {
            bool success = await userAuth.ExchangeCodeForTokenAsync(authCode);
            
            if (success)
            {
                string accessToken = credentials.AccessToken;
                Console.WriteLine("✅ Authentication successful!");
                Console.WriteLine($"Access Token: {accessToken[..10]}...");
                
                // 6. Make your first API call
                await MakeFirstApiCall(accessToken);
            }
            else
            {
                Console.WriteLine("❌ Authentication failed!");
            }
        }
    }

    static async Task MakeFirstApiCall(string accessToken)
    {
        try
        {
            // Get athlete profile
            var profile = await Athletes.GetAuthenticatedAthleteProfileAsync(accessToken);
            Console.WriteLine($"Welcome, {profile["firstname"]} {profile["lastname"]}!");

            // Get recent activities
            var activities = await Activities.GetAthletesActivitiesAsync(accessToken, page: 1, perPage: 5);
            Console.WriteLine($"Found {activities.Count} recent activities:");
            
            foreach (var activity in activities)
            {
                Console.WriteLine($"- {activity["name"]} ({activity["type"]}) - {activity["distance"]}m");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error making API call: {ex.Message}");
        }
    }
}
```

## Step 4: Understanding the Authentication Flow

### OAuth 2.0 Flow

The library implements the standard OAuth 2.0 authorization code flow:

1. **Authorization Request**: User is redirected to Strava to authorize your application
2. **Authorization Code**: Strava returns an authorization code to your redirect URI
3. **Token Exchange**: Your application exchanges the code for an access token
4. **API Calls**: Use the access token to make API requests

### Token Management

The library handles token management automatically:

- **Access Token**: Used for API requests (expires in 6 hours)
- **Refresh Token**: Used to get new access tokens (doesn't expire)
- **Token Expiration**: Automatically tracked and handled

### Scopes

Request only the scopes you need:

- `read` - Basic profile access
- `activity:read_all` - Read all activities
- `activity:write` - Upload activities
- `profile:read_all` - Detailed profile access
- `profile:write` - Update profile information

## Step 5: Common Usage Patterns

### Error Handling

```csharp
try
{
    var activities = await Activities.GetAthletesActivitiesAsync(accessToken);
    // Process activities
}
catch (ArgumentException ex)
{
    Console.WriteLine($"Invalid parameter: {ex.Message}");
}
catch (HttpRequestException ex)
{
    Console.WriteLine($"API request failed: {ex.Message}");
}
catch (JsonException ex)
{
    Console.WriteLine($"JSON parsing failed: {ex.Message}");
}
```

### Pagination

```csharp
int page = 1;
int perPage = 30;
bool hasMoreActivities = true;

while (hasMoreActivities)
{
    var activities = await Activities.GetAthletesActivitiesAsync(accessToken, page: page, perPage: perPage);
    
    if (activities.Count == 0)
    {
        hasMoreActivities = false;
    }
    else
    {
        // Process activities
        foreach (var activity in activities)
        {
            Console.WriteLine($"Activity: {activity["name"]}");
        }
        
        page++;
    }
}
```

### Filtering Activities

```csharp
// Get activities from last 30 days
int after = (int)DateTimeOffset.UtcNow.AddDays(-30).ToUnixTimeSeconds();
var recentActivities = await Activities.GetAthletesActivitiesAsync(accessToken, after: after);

// Get activities before a specific date
int before = (int)DateTimeOffset.Parse("2024-01-01").ToUnixTimeSeconds();
var oldActivities = await Activities.GetAthletesActivitiesAsync(accessToken, before: before);
```

## Step 6: Next Steps

Now that you have the basics working, explore:

- **[API Reference](~/docs/api/)** - Complete API documentation
- **[Authentication Guide](authentication.md)** - Advanced authentication topics
- **[Examples](examples.md)** - More usage examples
- **[Best Practices](best-practices.md)** - Recommended patterns

## Troubleshooting

### Common Issues

**"Invalid authorization code"**
- Ensure the authorization code is copied correctly
- Check that your redirect URI matches your Strava app settings
- Verify the code hasn't expired (codes expire quickly)

**"Access token is invalid"**
- The access token may have expired
- Use the refresh token to get a new access token
- Check that you're using the correct scope

**"API request failed"**
- Verify your internet connection
- Check that the Strava API is available
- Ensure your access token is valid

### Getting Help

- **Documentation**: [https://your-docs-site.com](https://your-docs-site.com)
- **GitHub Issues**: [https://github.com/your-repo/issues](https://github.com/your-repo/issues)
- **Strava API Status**: [https://status.strava.com](https://status.strava.com)

---

**Ready to build amazing Strava applications! 🚀** 