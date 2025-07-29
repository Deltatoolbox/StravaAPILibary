---
layout: default
title: Home
description: Comprehensive .NET library for Strava API integration
---

# StravaAPILibary Documentation

Welcome to the comprehensive documentation for **StravaAPILibary**, a powerful .NET library for interacting with the Strava API.

## 🚀 Quick Start

Get up and running in minutes:

```csharp
using StravaAPILibary.Authentication;
using StravaAPILibary.API;

// Set up credentials
var credentials = new Credentials("your_client_id", "your_client_secret", "read,activity:read_all");

// Authenticate
var userAuth = new UserAuthentication(credentials, "http://localhost:8080/callback", "read,activity:read_all");
userAuth.StartAuthorization();

// Exchange code for token
bool success = await userAuth.ExchangeCodeForTokenAsync("your_auth_code");

if (success)
{
    string accessToken = credentials.AccessToken;
    
    // Make API calls
    var activities = await Activities.GetAthletesActivitiesAsync(accessToken);
    var profile = await Athletes.GetAuthenticatedAthleteProfileAsync(accessToken);
}
```

## 📚 Documentation Sections

<div class="card">
  <h3><a href="{{ '/articles/getting-started/' | relative_url }}">🚀 Getting Started</a></h3>
  <p>Complete setup guide with step-by-step instructions for installing and configuring the library.</p>
</div>

<div class="card">
  <h3><a href="{{ '/articles/authentication/' | relative_url }}">🔐 Authentication Guide</a></h3>
  <p>Comprehensive guide to OAuth 2.0 flow, token management, and security best practices.</p>
</div>

<div class="card">
  <h3><a href="{{ '/articles/examples/' | relative_url }}">💡 Examples</a></h3>
  <p>Practical examples and real-world scenarios showing how to use the library effectively.</p>
</div>

<div class="card">
  <h3><a href="{{ '/api/' | relative_url }}">📚 API Reference</a></h3>
  <p>Complete API documentation with detailed method descriptions, parameters, and examples.</p>
</div>

## 🔧 Features

- **Complete API Coverage** - All Strava API endpoints
- **OAuth 2.0 Authentication** - Secure token management
- **Strongly Typed** - Full IntelliSense support
- **Error Handling** - Comprehensive exception handling
- **Async/Await** - Modern asynchronous programming patterns

## 📦 Installation

```bash
dotnet add package StravaAPILibary
```

## 🔗 Links

- **[GitHub Repository](https://github.com/your-repo/StravaAPILibary)**
- **[NuGet Package](https://www.nuget.org/packages/StravaAPILibary)**
- **[Strava API Documentation](https://developers.strava.com/docs/reference/)**

---

**Build amazing Strava applications with confidence! 🚀** 