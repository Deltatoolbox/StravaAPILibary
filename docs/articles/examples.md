---
layout: default
title: Examples
description: Practical examples and real-world scenarios for StravaAPILibary
---

# Examples

This page contains practical examples and real-world scenarios showing how to use StravaAPILibary effectively.

## 🚀 Basic Examples

### Get Athlete Profile

```csharp
using StravaAPILibary.API;

// Get the authenticated athlete's profile
var athlete = await Athletes.GetAuthenticatedAthleteProfileAsync(accessToken);

Console.WriteLine($"Name: {athlete.FirstName} {athlete.LastName}");
Console.WriteLine($"Location: {athlete.City}, {athlete.Country}");
Console.WriteLine($"Followers: {athlete.FollowerCount}");
Console.WriteLine($"Following: {athlete.FriendCount}");
```

### Get Recent Activities

```csharp
// Get the last 10 activities
var activities = await Activities.GetAthletesActivitiesAsync(accessToken, perPage: 10);

foreach (var activity in activities)
{
    Console.WriteLine($"Activity: {activity.Name}");
    Console.WriteLine($"Type: {activity.Type}");
    Console.WriteLine($"Distance: {activity.Distance:F0}m");
    Console.WriteLine($"Duration: {TimeSpan.FromSeconds(activity.MovingTime)}");
    Console.WriteLine($"Date: {activity.StartDate}");
    Console.WriteLine("---");
}
```

### Get Activity Details

```csharp
// Get detailed information about a specific activity
long activityId = 123456789;
var activity = await Activities.GetActivityByIdAsync(accessToken, activityId);

Console.WriteLine($"Activity: {activity.Name}");
Console.WriteLine($"Description: {activity.Description}");
Console.WriteLine($"Distance: {activity.Distance:F0}m");
Console.WriteLine($"Elevation Gain: {activity.TotalElevationGain:F0}m");
Console.WriteLine($"Average Speed: {activity.AverageSpeed:F2}m/s");
Console.WriteLine($"Max Speed: {activity.MaxSpeed:F2}m/s");
```

## 📊 Data Analysis Examples

### Calculate Weekly Distance

```csharp
public async Task<double> GetWeeklyDistanceAsync(string accessToken)
{
    // Get activities from the last 7 days
    int after = (int)DateTimeOffset.UtcNow.AddDays(-7).ToUnixTimeSeconds();
    var activities = await Activities.GetAthletesActivitiesAsync(accessToken, after: after);
    
    double totalDistance = activities.Sum(a => a.Distance);
    return totalDistance; // Returns distance in meters
}

// Usage
double weeklyDistance = await GetWeeklyDistanceAsync(accessToken);
Console.WriteLine($"Weekly Distance: {weeklyDistance / 1000:F1}km");
```

### Find Personal Records

```csharp
public async Task<List<SummaryActivity>> FindPersonalRecordsAsync(string accessToken)
{
    var activities = await Activities.GetAthletesActivitiesAsync(accessToken, perPage: 200);
    
    var personalRecords = new List<SummaryActivity>();
    
    // Find fastest 5k
    var fastest5k = activities
        .Where(a => a.Distance >= 5000 && a.Distance <= 5100)
        .OrderBy(a => a.MovingTime)
        .FirstOrDefault();
    
    if (fastest5k != null)
    {
        personalRecords.Add(fastest5k);
    }
    
    // Find longest distance
    var longestDistance = activities
        .OrderByDescending(a => a.Distance)
        .FirstOrDefault();
    
    if (longestDistance != null)
    {
        personalRecords.Add(longestDistance);
    }
    
    return personalRecords;
}
```

### Activity Statistics

```csharp
public class ActivityStats
{
    public int TotalActivities { get; set; }
    public double TotalDistance { get; set; }
    public TimeSpan TotalTime { get; set; }
    public double AverageDistance { get; set; }
    public Dictionary<string, int> ActivityTypes { get; set; } = new();
}

public async Task<ActivityStats> GetActivityStatsAsync(string accessToken, int days = 30)
{
    int after = (int)DateTimeOffset.UtcNow.AddDays(-days).ToUnixTimeSeconds();
    var activities = await Activities.GetAthletesActivitiesAsync(accessToken, after: after);
    
    var stats = new ActivityStats
    {
        TotalActivities = activities.Count,
        TotalDistance = activities.Sum(a => a.Distance),
        TotalTime = TimeSpan.FromSeconds(activities.Sum(a => a.MovingTime))
    };
    
    stats.AverageDistance = stats.TotalDistance / stats.TotalActivities;
    
    // Count activity types
    foreach (var activity in activities)
    {
        if (stats.ActivityTypes.ContainsKey(activity.Type))
        {
            stats.ActivityTypes[activity.Type]++;
        }
        else
        {
            stats.ActivityTypes[activity.Type] = 1;
        }
    }
    
    return stats;
}
```

## 🗺️ Route and Segment Examples

### Get Route Information

```csharp
public async Task<Route> GetRouteDetailsAsync(string accessToken, long routeId)
{
    var route = await Routes.GetRouteByIdAsync(accessToken, routeId);
    
    Console.WriteLine($"Route: {route.Name}");
    Console.WriteLine($"Distance: {route.Distance:F0}m");
    Console.WriteLine($"Elevation Gain: {route.ElevationGain:F0}m");
    Console.WriteLine($"Type: {route.Type}");
    Console.WriteLine($"Sub Type: {route.SubType}");
    
    return route;
}
```

### Find Segments Near Location

```csharp
public async Task<List<ExplorerSegment>> FindSegmentsNearbyAsync(
    string accessToken, 
    double latitude, 
    double longitude, 
    double radius = 1000)
{
    var segments = await Segments.ExploreSegmentsAsync(
        accessToken, 
        bounds: $"{latitude},{longitude},{latitude},{longitude}",
        activityType: "running",
        minCat: 0,
        maxCat: 5
    );
    
    return segments.Segments
        .Where(s => s.Distance <= radius)
        .OrderBy(s => s.Distance)
        .ToList();
}
```

## 👥 Social Features

### Get Athlete's Followers

```csharp
public async Task<List<SummaryAthlete>> GetFollowersAsync(string accessToken)
{
    var followers = await Athletes.GetAuthenticatedAthleteFollowersAsync(accessToken);
    
    foreach (var follower in followers)
    {
        Console.WriteLine($"Follower: {follower.FirstName} {follower.LastName}");
        Console.WriteLine($"Username: {follower.Username}");
        Console.WriteLine($"Location: {follower.City}");
        Console.WriteLine("---");
    }
    
    return followers;
}
```

### Get Club Activities

```csharp
public async Task<List<ClubActivity>> GetClubActivitiesAsync(string accessToken, long clubId)
{
    var activities = await Clubs.GetClubActivitiesByIdAsync(accessToken, clubId);
    
    foreach (var activity in activities)
    {
        Console.WriteLine($"Athlete: {activity.Athlete.FirstName} {activity.Athlete.LastName}");
        Console.WriteLine($"Activity: {activity.Name}");
        Console.WriteLine($"Distance: {activity.Distance:F0}m");
        Console.WriteLine($"Type: {activity.Type}");
        Console.WriteLine("---");
    }
    
    return activities;
}
```

## 📈 Performance Tracking

### Track Progress Over Time

```csharp
public class PerformanceTracker
{
    public async Task<Dictionary<string, List<double>>> TrackProgressAsync(
        string accessToken, 
        int weeks = 12)
    {
        var weeklyDistances = new Dictionary<string, List<double>>();
        
        for (int week = 0; week < weeks; week++)
        {
            var startDate = DateTimeOffset.UtcNow.AddDays(-7 * (week + 1));
            var endDate = startDate.AddDays(7);
            
            int after = (int)startDate.ToUnixTimeSeconds();
            int before = (int)endDate.ToUnixTimeSeconds();
            
            var activities = await Activities.GetAthletesActivitiesAsync(
                accessToken, 
                after: after, 
                before: before
            );
            
            var weekDistance = activities.Sum(a => a.Distance) / 1000; // Convert to km
            var weekKey = startDate.ToString("yyyy-MM-dd");
            
            weeklyDistances[weekKey] = new List<double> { weekDistance };
        }
        
        return weeklyDistances;
    }
}
```

### Compare Performance

```csharp
public async Task<PerformanceComparison> ComparePerformanceAsync(
    string accessToken, 
    DateTime startDate, 
    DateTime endDate)
{
    int after = (int)startDate.ToUnixTimeSeconds();
    int before = (int)endDate.ToUnixTimeSeconds();
    
    var activities = await Activities.GetAthletesActivitiesAsync(
        accessToken, 
        after: after, 
        before: before
    );
    
    var comparison = new PerformanceComparison
    {
        TotalActivities = activities.Count,
        TotalDistance = activities.Sum(a => a.Distance),
        TotalTime = TimeSpan.FromSeconds(activities.Sum(a => a.MovingTime)),
        AverageSpeed = activities.Average(a => a.AverageSpeed),
        MaxSpeed = activities.Max(a => a.MaxSpeed),
        TotalElevationGain = activities.Sum(a => a.TotalElevationGain)
    };
    
    return comparison;
}
```

## 🔧 Error Handling Examples

### Robust API Calls

```csharp
public async Task<T?> SafeApiCallAsync<T>(Func<Task<T>> apiCall, int maxRetries = 3)
{
    for (int attempt = 1; attempt <= maxRetries; attempt++)
    {
        try
        {
            return await apiCall();
        }
        catch (StravaApiException ex) when (ex.StatusCode == 429)
        {
            // Rate limit exceeded
            if (attempt < maxRetries)
            {
                int delaySeconds = attempt * 2; // Exponential backoff
                await Task.Delay(delaySeconds * 1000);
                continue;
            }
            throw;
        }
        catch (StravaApiException ex) when (ex.StatusCode == 401)
        {
            // Token expired, try to refresh
            if (attempt < maxRetries)
            {
                bool refreshed = await RefreshTokenAsync();
                if (refreshed)
                {
                    continue;
                }
            }
            throw;
        }
        catch (HttpRequestException ex)
        {
            // Network error
            if (attempt < maxRetries)
            {
                await Task.Delay(1000 * attempt);
                continue;
            }
            throw;
        }
    }
    
    throw new InvalidOperationException("Max retries exceeded");
}

// Usage
var activities = await SafeApiCallAsync(() => 
    Activities.GetAthletesActivitiesAsync(accessToken)
);
```

### Batch Processing

```csharp
public async Task ProcessActivitiesInBatchesAsync(
    string accessToken, 
    Func<SummaryActivity, Task> processor,
    int batchSize = 50)
{
    int page = 1;
    bool hasMoreActivities = true;
    
    while (hasMoreActivities)
    {
        try
        {
            var activities = await Activities.GetAthletesActivitiesAsync(
                accessToken, 
                page: page, 
                perPage: batchSize
            );
            
            if (activities.Count == 0)
            {
                hasMoreActivities = false;
                break;
            }
            
            // Process activities in parallel
            var tasks = activities.Select(processor);
            await Task.WhenAll(tasks);
            
            page++;
        }
        catch (StravaApiException ex)
        {
            Console.WriteLine($"Error processing batch {page}: {ex.Message}");
            page++; // Skip this batch and continue
        }
    }
}

// Usage
await ProcessActivitiesInBatchesAsync(accessToken, async activity =>
{
    Console.WriteLine($"Processing: {activity.Name}");
    // Your processing logic here
    await Task.Delay(100); // Rate limiting
});
```

## 🎯 Real-World Applications

### Fitness Dashboard

```csharp
public class FitnessDashboard
{
    public async Task<DashboardData> GetDashboardDataAsync(string accessToken)
    {
        var dashboard = new DashboardData();
        
        // Get recent activities
        var recentActivities = await Activities.GetAthletesActivitiesAsync(
            accessToken, 
            perPage: 10
        );
        
        dashboard.RecentActivities = recentActivities;
        
        // Calculate weekly stats
        var weeklyStats = await GetActivityStatsAsync(accessToken, 7);
        dashboard.WeeklyStats = weeklyStats;
        
        // Get personal records
        var personalRecords = await FindPersonalRecordsAsync(accessToken);
        dashboard.PersonalRecords = personalRecords;
        
        return dashboard;
    }
}
```

### Training Plan Generator

```csharp
public class TrainingPlanGenerator
{
    public async Task<TrainingPlan> GeneratePlanAsync(
        string accessToken, 
        string goal, 
        int weeks)
    {
        var plan = new TrainingPlan { Goal = goal, DurationWeeks = weeks };
        
        // Analyze current fitness level
        var recentStats = await GetActivityStatsAsync(accessToken, 30);
        
        // Generate weekly targets based on goal and current fitness
        for (int week = 1; week <= weeks; week++)
        {
            var weeklyTarget = new WeeklyTarget
            {
                Week = week,
                TargetDistance = CalculateTargetDistance(recentStats, goal, week),
                TargetTime = CalculateTargetTime(recentStats, goal, week),
                ActivityTypes = GetRecommendedActivities(goal, week)
            };
            
            plan.WeeklyTargets.Add(weeklyTarget);
        }
        
        return plan;
    }
}
```

## 📚 More Examples

- **[API Reference]({{ '/api/' | relative_url }})** - Complete method documentation
- **[Authentication Guide]({{ '/articles/authentication/' | relative_url }})** - OAuth examples
- **[Getting Started]({{ '/articles/getting-started/' | relative_url }})** - Basic setup

---

**Have a specific use case? Check our API reference or create an issue on GitHub! 💡** 