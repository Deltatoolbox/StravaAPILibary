# Best Practices

This guide covers best practices for using the StravaAPILibary effectively, securely, and efficiently.

## 🔐 Security Best Practices

### 1. Secure Credential Management

**✅ Do:**
```csharp
// Use environment variables for sensitive data
string clientId = Environment.GetEnvironmentVariable("STRAVA_CLIENT_ID") 
    ?? throw new InvalidOperationException("STRAVA_CLIENT_ID not set");
string clientSecret = Environment.GetEnvironmentVariable("STRAVA_CLIENT_SECRET") 
    ?? throw new InvalidOperationException("STRAVA_CLIENT_SECRET not set");

var credentials = new Credentials(clientId, clientSecret, "read,activity:read_all");
```

**❌ Don't:**
```csharp
// Never hardcode credentials
var credentials = new Credentials("12345", "my_secret_key", "read");
```

### 2. Token Storage

**✅ Do:**
```csharp
// Use secure storage for tokens
public class SecureTokenStorage
{
    public async Task SaveTokensAsync(Credentials credentials)
    {
        // Use platform-specific secure storage
        await SecureStorage.SaveAsync("strava_access_token", credentials.AccessToken);
        await SecureStorage.SaveAsync("strava_refresh_token", credentials.RefreshToken);
        await SecureStorage.SaveAsync("strava_token_expiry", credentials.TokenExpiration.ToString());
    }
    
    public async Task<Credentials?> LoadTokensAsync(string clientId, string clientSecret)
    {
        var accessToken = await SecureStorage.GetAsync("strava_access_token");
        var refreshToken = await SecureStorage.GetAsync("strava_refresh_token");
        var expiryStr = await SecureStorage.GetAsync("strava_token_expiry");
        
        if (string.IsNullOrEmpty(accessToken) || string.IsNullOrEmpty(refreshToken))
            return null;
            
        var credentials = new Credentials(clientId, clientSecret, "read,activity:read_all")
        {
            AccessToken = accessToken,
            RefreshToken = refreshToken,
            TokenExpiration = DateTime.Parse(expiryStr)
        };
        
        return credentials;
    }
}
```

**❌ Don't:**
```csharp
// Never store tokens in plain text files
File.WriteAllText("tokens.txt", accessToken);
```

### 3. Scope Management

**✅ Do:**
```csharp
// Request minimal scopes
var credentials = new Credentials(clientId, clientSecret, "read,activity:read_all");

// Check if required scope is available
bool hasActivityWrite = credentials.Scope.Contains("activity:write");
if (!hasActivityWrite)
{
    throw new InvalidOperationException("activity:write scope is required for this operation.");
}
```

**❌ Don't:**
```csharp
// Don't request unnecessary scopes
var credentials = new Credentials(clientId, clientSecret, "read,activity:read_all,activity:write,profile:read_all,profile:write");
```

## ⚡ Performance Best Practices

### 1. Efficient API Usage

**✅ Do:**
```csharp
public class EfficientStravaClient
{
    private readonly HttpClient _httpClient;
    private readonly string _accessToken;
    
    public EfficientStravaClient(string accessToken)
    {
        _accessToken = accessToken;
        _httpClient = new HttpClient
        {
            Timeout = TimeSpan.FromSeconds(30),
            DefaultRequestHeaders = 
            {
                Authorization = new AuthenticationHeaderValue("Bearer", accessToken)
            }
        };
    }
    
    public async Task<JsonArray> GetActivitiesAsync(int page = 1, int perPage = 200)
    {
        // Use maximum per_page to reduce API calls
        return await Activities.GetAthletesActivitiesAsync(_accessToken, page: page, perPage: perPage);
    }
}
```

**❌ Don't:**
```csharp
// Don't make many small requests
for (int i = 1; i <= 100; i++)
{
    var activities = await Activities.GetAthletesActivitiesAsync(accessToken, page: i, perPage: 1);
    // Process single activity
}
```

### 2. Caching Strategies

**✅ Do:**
```csharp
public class CachedStravaClient
{
    private readonly IMemoryCache _cache;
    private readonly string _accessToken;
    
    public CachedStravaClient(string accessToken, IMemoryCache cache)
    {
        _accessToken = accessToken;
        _cache = cache;
    }
    
    public async Task<JsonObject> GetAthleteProfileAsync()
    {
        const string cacheKey = "athlete_profile";
        
        if (_cache.TryGetValue(cacheKey, out JsonObject? cachedProfile))
        {
            return cachedProfile!;
        }
        
        var profile = await Athletes.GetAuthenticatedAthleteProfileAsync(_accessToken);
        
        // Cache for 1 hour (profile doesn't change frequently)
        _cache.Set(cacheKey, profile, TimeSpan.FromHours(1));
        
        return profile;
    }
}
```

### 3. Batch Processing

**✅ Do:**
```csharp
public class BatchActivityProcessor
{
    public async Task ProcessAllActivitiesAsync(string accessToken)
    {
        int page = 1;
        int perPage = 200; // Maximum to reduce API calls
        var allActivities = new List<JsonNode>();
        
        while (true)
        {
            var activities = await Activities.GetAthletesActivitiesAsync(accessToken, page: page, perPage: perPage);
            
            if (activities.Count == 0)
                break;
                
            allActivities.AddRange(activities);
            page++;
        }
        
        // Process all activities at once
        await ProcessActivitiesBatchAsync(allActivities);
    }
    
    private async Task ProcessActivitiesBatchAsync(List<JsonNode> activities)
    {
        // Process activities in batches for efficiency
        const int batchSize = 50;
        
        for (int i = 0; i < activities.Count; i += batchSize)
        {
            var batch = activities.Skip(i).Take(batchSize);
            await ProcessBatchAsync(batch);
        }
    }
}
```

## 🛡️ Error Handling Best Practices

### 1. Comprehensive Exception Handling

**✅ Do:**
```csharp
public class RobustStravaClient
{
    public async Task<JsonArray?> GetActivitiesWithRetryAsync(int maxRetries = 3)
    {
        for (int attempt = 1; attempt <= maxRetries; attempt++)
        {
            try
            {
                return await Activities.GetAthletesActivitiesAsync(_accessToken);
            }
            catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.TooManyRequests)
            {
                if (attempt < maxRetries)
                {
                    var retryAfter = GetRetryAfterSeconds(ex);
                    await Task.Delay(retryAfter * 1000);
                }
                else
                {
                    throw new InvalidOperationException("Rate limit exceeded after multiple retries", ex);
                }
            }
            catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.Unauthorized)
            {
                // Token might be expired, try to refresh
                if (await TryRefreshTokenAsync())
                {
                    continue; // Retry with new token
                }
                throw new InvalidOperationException("Access token is invalid and refresh failed", ex);
            }
            catch (Exception ex)
            {
                if (attempt == maxRetries)
                    throw;
                
                await Task.Delay(1000 * attempt); // Exponential backoff
            }
        }
        
        return null;
    }
}
```

### 2. Token Refresh Logic

**✅ Do:**
```csharp
public class TokenManager
{
    private readonly Credentials _credentials;
    private readonly UserAuthentication _userAuth;
    
    public async Task<string> GetValidAccessTokenAsync()
    {
        // Check if token is expired or will expire soon
        if (_credentials.TokenExpiration <= DateTime.UtcNow.AddMinutes(5))
        {
            bool refreshSuccess = await _userAuth.RefreshAccessTokenAsync();
            if (!refreshSuccess)
            {
                throw new InvalidOperationException("Failed to refresh access token. User needs to re-authenticate.");
            }
        }
        
        return _credentials.AccessToken;
    }
    
    public async Task<bool> TryRefreshTokenAsync()
    {
        try
        {
            return await _userAuth.RefreshAccessTokenAsync();
        }
        catch (Exception)
        {
            return false;
        }
    }
}
```

### 3. Graceful Degradation

**✅ Do:**
```csharp
public class StravaService
{
    public async Task<ActivitySummary> GetActivitySummaryAsync(string accessToken)
    {
        try
        {
            var activities = await Activities.GetAthletesActivitiesAsync(accessToken, page: 1, perPage: 10);
            
            return new ActivitySummary
            {
                TotalActivities = activities.Count,
                TotalDistance = activities.Sum(a => (double)a["distance"]),
                IsComplete = true
            };
        }
        catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.TooManyRequests)
        {
            // Return partial data when rate limited
            return new ActivitySummary
            {
                TotalActivities = 0,
                TotalDistance = 0,
                IsComplete = false,
                ErrorMessage = "Rate limited - showing cached data"
            };
        }
        catch (Exception ex)
        {
            // Log error and return empty result
            _logger.LogError(ex, "Failed to get activity summary");
            return new ActivitySummary
            {
                TotalActivities = 0,
                TotalDistance = 0,
                IsComplete = false,
                ErrorMessage = "Service temporarily unavailable"
            };
        }
    }
}
```

## 📊 Monitoring and Logging

### 1. Structured Logging

**✅ Do:**
```csharp
public class StravaClientWithLogging
{
    private readonly ILogger<StravaClientWithLogging> _logger;
    
    public async Task<JsonArray> GetActivitiesAsync(string accessToken, int page = 1, int perPage = 30)
    {
        using var scope = _logger.BeginScope(new Dictionary<string, object>
        {
            ["page"] = page,
            ["per_page"] = perPage
        });
        
        _logger.LogInformation("Retrieving activities from Strava API");
        
        try
        {
            var activities = await Activities.GetAthletesActivitiesAsync(accessToken, page: page, perPage: perPage);
            
            _logger.LogInformation("Successfully retrieved {Count} activities", activities.Count);
            
            return activities;
        }
        catch (HttpRequestException ex)
        {
            _logger.LogError(ex, "Failed to retrieve activities. Status: {StatusCode}", ex.StatusCode);
            throw;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unexpected error while retrieving activities");
            throw;
        }
    }
}
```

### 2. Metrics Collection

**✅ Do:**
```csharp
public class StravaClientWithMetrics
{
    private readonly IMetrics _metrics;
    
    public async Task<JsonArray> GetActivitiesAsync(string accessToken)
    {
        using var timer = _metrics.CreateTimer("strava.api.activities.duration");
        
        try
        {
            var activities = await Activities.GetAthletesActivitiesAsync(accessToken);
            
            _metrics.Increment("strava.api.activities.success");
            _metrics.RecordGauge("strava.api.activities.count", activities.Count);
            
            return activities;
        }
        catch (Exception ex)
        {
            _metrics.Increment("strava.api.activities.error");
            throw;
        }
    }
}
```

## 🔄 Design Patterns

### 1. Repository Pattern

**✅ Do:**
```csharp
public interface IStravaRepository
{
    Task<JsonArray> GetActivitiesAsync(int page = 1, int perPage = 30);
    Task<JsonObject> GetActivityAsync(string activityId);
    Task<JsonObject> GetAthleteProfileAsync();
    Task<bool> UpdateActivityAsync(long activityId, string name, string description);
}

public class StravaRepository : IStravaRepository
{
    private readonly string _accessToken;
    private readonly TokenManager _tokenManager;
    
    public StravaRepository(string accessToken, TokenManager tokenManager)
    {
        _accessToken = accessToken;
        _tokenManager = tokenManager;
    }
    
    public async Task<JsonArray> GetActivitiesAsync(int page = 1, int perPage = 30)
    {
        string validToken = await _tokenManager.GetValidAccessTokenAsync();
        return await Activities.GetAthletesActivitiesAsync(validToken, page: page, perPage: perPage);
    }
    
    public async Task<JsonObject> GetActivityAsync(string activityId)
    {
        string validToken = await _tokenManager.GetValidAccessTokenAsync();
        return await Activities.GetActivityByIdAsync(validToken, activityId);
    }
    
    public async Task<JsonObject> GetAthleteProfileAsync()
    {
        string validToken = await _tokenManager.GetValidAccessTokenAsync();
        return await Athletes.GetAuthenticatedAthleteProfileAsync(validToken);
    }
    
    public async Task<bool> UpdateActivityAsync(long activityId, string name, string description)
    {
        string validToken = await _tokenManager.GetValidAccessTokenAsync();
        var result = await Activities.UpdateActivityAsync(validToken, activityId, name, description);
        return result != null;
    }
}
```

### 2. Factory Pattern

**✅ Do:**
```csharp
public class StravaClientFactory
{
    private readonly IConfiguration _configuration;
    private readonly ILogger<StravaClientFactory> _logger;
    
    public StravaClientFactory(IConfiguration configuration, ILogger<StravaClientFactory> logger)
    {
        _configuration = configuration;
        _logger = logger;
    }
    
    public async Task<IStravaRepository> CreateClientAsync()
    {
        var clientId = _configuration["Strava:ClientId"];
        var clientSecret = _configuration["Strava:ClientSecret"];
        
        if (string.IsNullOrEmpty(clientId) || string.IsNullOrEmpty(clientSecret))
        {
            throw new InvalidOperationException("Strava credentials not configured");
        }
        
        var credentials = new Credentials(clientId, clientSecret, "read,activity:read_all");
        var tokenManager = new TokenManager(credentials);
        
        // Try to load existing tokens
        var existingTokens = await LoadTokensAsync();
        if (existingTokens != null)
        {
            credentials.AccessToken = existingTokens.AccessToken;
            credentials.RefreshToken = existingTokens.RefreshToken;
            credentials.TokenExpiration = existingTokens.TokenExpiration;
        }
        
        return new StravaRepository(credentials.AccessToken, tokenManager);
    }
}
```

## 🧪 Testing Best Practices

### 1. Unit Testing

**✅ Do:**
```csharp
[TestClass]
public class StravaClientTests
{
    private Mock<IHttpClientFactory> _mockHttpClientFactory;
    private StravaClient _client;
    
    [TestInitialize]
    public void Setup()
    {
        _mockHttpClientFactory = new Mock<IHttpClientFactory>();
        _client = new StravaClient("test_token", _mockHttpClientFactory.Object);
    }
    
    [TestMethod]
    public async Task GetActivitiesAsync_ValidToken_ReturnsActivities()
    {
        // Arrange
        var mockHttpClient = new Mock<HttpClient>();
        var mockResponse = new HttpResponseMessage(HttpStatusCode.OK)
        {
            Content = new StringContent("[{\"id\":123,\"name\":\"Test Activity\"}]")
        };
        
        mockHttpClient.Setup(x => x.GetAsync(It.IsAny<string>()))
            .ReturnsAsync(mockResponse);
        
        _mockHttpClientFactory.Setup(x => x.CreateClient(It.IsAny<string>()))
            .Returns(mockHttpClient.Object);
        
        // Act
        var result = await _client.GetActivitiesAsync();
        
        // Assert
        Assert.IsNotNull(result);
        Assert.AreEqual(1, result.Count);
        Assert.AreEqual("Test Activity", result[0]["name"]);
    }
}
```

### 2. Integration Testing

**✅ Do:**
```csharp
[TestClass]
public class StravaIntegrationTests
{
    private string _testAccessToken;
    
    [TestInitialize]
    public async Task Setup()
    {
        // Use test credentials for integration tests
        _testAccessToken = await GetTestAccessTokenAsync();
    }
    
    [TestMethod]
    public async Task GetAthleteProfile_ValidToken_ReturnsProfile()
    {
        // Arrange & Act
        var profile = await Athletes.GetAuthenticatedAthleteProfileAsync(_testAccessToken);
        
        // Assert
        Assert.IsNotNull(profile);
        Assert.IsTrue(profile.ContainsKey("id"));
        Assert.IsTrue(profile.ContainsKey("firstname"));
        Assert.IsTrue(profile.ContainsKey("lastname"));
    }
}
```

## 📚 Documentation Best Practices

### 1. Code Documentation

**✅ Do:**
```csharp
/// <summary>
/// Retrieves the authenticated athlete's activities with optional filtering and pagination.
/// </summary>
/// <param name="accessToken">The OAuth access token for authentication.</param>
/// <param name="page">Page number for pagination. Must be greater than 0.</param>
/// <param name="perPage">Number of activities per page. Must be between 1 and 200.</param>
/// <returns>A <see cref="JsonArray"/> containing the athlete's activities.</returns>
/// <exception cref="ArgumentException">Thrown when access token is invalid.</exception>
/// <exception cref="HttpRequestException">Thrown when the API request fails.</exception>
/// <remarks>
/// This method requires the <c>activity:read_all</c> scope.
/// Activities are returned in reverse chronological order.
/// </remarks>
/// <example>
/// <code>
/// var activities = await GetActivitiesAsync(accessToken, page: 1, perPage: 10);
/// </code>
/// </example>
public async Task<JsonArray> GetActivitiesAsync(string accessToken, int page = 1, int perPage = 30)
{
    // Implementation
}
```

## 🚀 Deployment Best Practices

### 1. Configuration Management

**✅ Do:**
```json
{
  "Strava": {
    "ClientId": "your_client_id",
    "ClientSecret": "your_client_secret",
    "RedirectUri": "https://yourapp.com/callback",
    "DefaultScope": "read,activity:read_all"
  },
  "Logging": {
    "LogLevel": {
      "StravaAPILibary": "Information"
    }
  }
}
```

### 2. Environment-Specific Settings

**✅ Do:**
```csharp
public class StravaConfiguration
{
    public string ClientId { get; set; } = string.Empty;
    public string ClientSecret { get; set; } = string.Empty;
    public string RedirectUri { get; set; } = string.Empty;
    public string DefaultScope { get; set; } = "read,activity:read_all";
    public int RequestTimeoutSeconds { get; set; } = 30;
    public int MaxRetries { get; set; } = 3;
}

// In Startup.cs
services.Configure<StravaConfiguration>(configuration.GetSection("Strava"));
```

## 📈 Performance Monitoring

### 1. Health Checks

**✅ Do:**
```csharp
public class StravaHealthCheck : IHealthCheck
{
    private readonly IStravaRepository _stravaRepository;
    
    public StravaHealthCheck(IStravaRepository stravaRepository)
    {
        _stravaRepository = stravaRepository;
    }
    
    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default)
    {
        try
        {
            var profile = await _stravaRepository.GetAthleteProfileAsync();
            
            if (profile != null && profile.ContainsKey("id"))
            {
                return HealthCheckResult.Healthy("Strava API is accessible");
            }
            
            return HealthCheckResult.Unhealthy("Strava API returned invalid response");
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("Strava API is not accessible", ex);
        }
    }
}
```

## 🔗 Next Steps

- **[API Reference](~/docs/api/)** - Complete API documentation
- **[Authentication Guide](authentication.md)** - OAuth flow and token management
- **[Examples](examples.md)** - Practical usage examples

---

**Follow these best practices to build robust, secure, and efficient Strava applications! 🚀** 