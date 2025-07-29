# Examples

This guide provides practical examples of using the StravaAPILibary for common scenarios and real-world applications.

## Basic Examples

### Get Athlete Profile

```csharp
using StravaAPILibary.API;

var profile = await Athletes.GetAuthenticatedAthleteProfileAsync(accessToken);

Console.WriteLine($"Name: {profile["firstname"]} {profile["lastname"]}");
Console.WriteLine($"Location: {profile["city"]}, {profile["state"]}");
Console.WriteLine($"Followers: {profile["follower_count"]}");
Console.WriteLine($"Following: {profile["friend_count"]}");
```

### Get Recent Activities

```csharp
var activities = await Activities.GetAthletesActivitiesAsync(accessToken, page: 1, perPage: 10);

foreach (var activity in activities)
{
    Console.WriteLine($"Activity: {activity["name"]}");
    Console.WriteLine($"  Type: {activity["type"]}");
    Console.WriteLine($"  Distance: {activity["distance"]}m");
    Console.WriteLine($"  Duration: {activity["moving_time"]}s");
    Console.WriteLine($"  Date: {activity["start_date_local"]}");
    Console.WriteLine();
}
```

### Get Activity Details

```csharp
string activityId = "123456789";
var activity = await Activities.GetActivityByIdAsync(accessToken, activityId);

Console.WriteLine($"Activity: {activity["name"]}");
Console.WriteLine($"  Description: {activity["description"]}");
Console.WriteLine($"  Distance: {activity["distance"]}m");
Console.WriteLine($"  Elevation Gain: {activity["total_elevation_gain"]}m");
Console.WriteLine($"  Average Speed: {activity["average_speed"]} m/s");
Console.WriteLine($"  Max Speed: {activity["max_speed"]} m/s");
```

## Advanced Examples

### Activity Analysis

```csharp
public class ActivityAnalyzer
{
    public async Task AnalyzeActivitiesAsync(string accessToken)
    {
        var activities = await Activities.GetAthletesActivitiesAsync(accessToken, page: 1, perPage: 50);
        
        var stats = new
        {
            TotalActivities = activities.Count,
            TotalDistance = activities.Sum(a => (double)a["distance"]),
            TotalTime = activities.Sum(a => (int)a["moving_time"]),
            ActivitiesByType = activities.GroupBy(a => (string)a["type"])
                                      .ToDictionary(g => g.Key, g => g.Count())
        };
        
        Console.WriteLine($"Total Activities: {stats.TotalActivities}");
        Console.WriteLine($"Total Distance: {stats.TotalDistance / 1000:F1}km");
        Console.WriteLine($"Total Time: {TimeSpan.FromSeconds(stats.TotalTime)}");
        
        foreach (var type in stats.ActivitiesByType)
        {
            Console.WriteLine($"  {type.Key}: {type.Value} activities");
        }
    }
}
```

### Segment Explorer

```csharp
public class SegmentExplorer
{
    public async Task ExploreSegmentsAsync(string accessToken, float[] bounds, string activityType = "running")
    {
        var segments = await Segments.GetExploreSegmentsAsync(accessToken, bounds, activityType);
        
        Console.WriteLine($"Found {segments.Count} segments in the area:");
        
        foreach (var segment in segments)
        {
            Console.WriteLine($"  {segment["name"]}");
            Console.WriteLine($"    Distance: {segment["distance"]}m");
            Console.WriteLine($"    Average Grade: {segment["average_grade"]}%");
            Console.WriteLine($"    Elevation Difference: {segment["elevation_high"] - segment["elevation_low"]}m");
            Console.WriteLine();
        }
    }
    
    public async Task GetStarredSegmentsAsync(string accessToken)
    {
        var starredSegments = await Segments.GetStarredSegmentsAsync(accessToken);
        
        Console.WriteLine($"You have {starredSegments.Count} starred segments:");
        
        foreach (var segment in starredSegments)
        {
            Console.WriteLine($"  {segment["name"]} - {segment["distance"]}m");
        }
    }
}
```

### Club Activities

```csharp
public class ClubManager
{
    public async Task GetClubInfoAsync(string accessToken)
    {
        var clubs = await Clubs.GetClubsAsync(accessToken);
        
        foreach (var club in clubs)
        {
            Console.WriteLine($"Club: {club["name"]}");
            Console.WriteLine($"  Members: {club["member_count"]}");
            Console.WriteLine($"  Activities: {club["activity_count"]}");
            Console.WriteLine($"  Description: {club["description"]}");
            Console.WriteLine();
            
            // Get club activities
            var clubId = (long)club["id"];
            var activities = await Clubs.GetClubActivitiesAsync(accessToken, clubId, page: 1, perPage: 5);
            
            Console.WriteLine("  Recent Activities:");
            foreach (var activity in activities)
            {
                Console.WriteLine($"    {activity["athlete"]["firstname"]} {activity["athlete"]["lastname"]}: {activity["name"]}");
            }
            Console.WriteLine();
        }
    }
}
```

### Route Management

```csharp
public class RouteManager
{
    public async Task ExportRouteAsync(string accessToken, long routeId, string outputPath)
    {
        // Get route details
        var route = await Routes.GetRouteByIdAsync(accessToken, routeId);
        Console.WriteLine($"Route: {route["name"]}");
        Console.WriteLine($"  Distance: {route["distance"]}m");
        Console.WriteLine($"  Elevation Gain: {route["elevation_gain"]}m");
        
        // Export as GPX
        var gpxData = await Routes.GetRouteGpxExportAsync(accessToken, routeId);
        await File.WriteAllTextAsync($"{outputPath}.gpx", gpxData);
        
        // Export as TCX
        var tcxData = await Routes.GetRouteTcxExportAsync(accessToken, routeId);
        await File.WriteAllTextAsync($"{outputPath}.tcx", tcxData);
        
        Console.WriteLine($"Route exported to {outputPath}.gpx and {outputPath}.tcx");
    }
}
```

### Activity Upload

```csharp
public class ActivityUploader
{
    public async Task UploadActivityAsync(string accessToken, string filePath, string activityName, string description = "")
    {
        try
        {
            // Determine file type
            string dataType = Path.GetExtension(filePath).ToLower() switch
            {
                ".gpx" => "gpx",
                ".tcx" => "tcx",
                ".fit" => "fit",
                _ => throw new ArgumentException("Unsupported file format. Use GPX, TCX, or FIT files.")
            };
            
            // Upload activity
            var uploadResponse = await Activities.PostActivityAsync(
                accessToken,
                activityName,
                dataType,
                filePath,
                description
            );
            
            Console.WriteLine($"Upload started. Upload ID: {uploadResponse["id"]}");
            
            // Monitor upload status
            await MonitorUploadStatusAsync(accessToken, (long)uploadResponse["id"]);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Upload failed: {ex.Message}");
        }
    }
    
    private async Task MonitorUploadStatusAsync(string accessToken, long uploadId)
    {
        int maxAttempts = 30; // 5 minutes with 10-second intervals
        int attempts = 0;
        
        while (attempts < maxAttempts)
        {
            var uploadStatus = await Uploads.GetUploadAsync(accessToken, uploadId);
            string status = (string)uploadStatus["status"];
            
            Console.WriteLine($"Upload status: {status}");
            
            if (status == "Your activity is ready.")
            {
                Console.WriteLine("✅ Upload completed successfully!");
                Console.WriteLine($"Activity ID: {uploadStatus["activity_id"]}");
                break;
            }
            else if (status == "There was an error processing your activity.")
            {
                Console.WriteLine("❌ Upload failed!");
                Console.WriteLine($"Error: {uploadStatus["error"]}");
                break;
            }
            
            await Task.Delay(10000); // Wait 10 seconds
            attempts++;
        }
        
        if (attempts >= maxAttempts)
        {
            Console.WriteLine("⏰ Upload monitoring timed out.");
        }
    }
}
```

### Stream Data Analysis

```csharp
public class StreamAnalyzer
{
    public async Task AnalyzeActivityStreamsAsync(string accessToken, long activityId)
    {
        var streams = await Streams.GetActivityStreamsAsync(
            accessToken, 
            activityId, 
            "time,distance,latlng,altitude,velocity_smooth,heartrate,cadence,watts,temp,moving,grade_smooth"
        );
        
        if (streams.ContainsKey("latlng"))
        {
            var latlngStream = streams["latlng"] as JsonArray;
            Console.WriteLine($"GPS Points: {latlngStream?.Count ?? 0}");
        }
        
        if (streams.ContainsKey("heartrate"))
        {
            var hrStream = streams["heartrate"] as JsonArray;
            if (hrStream?.Count > 0)
            {
                var hrValues = hrStream.Select(x => (int)x).ToList();
                Console.WriteLine($"Heart Rate - Avg: {hrValues.Average():F0}, Max: {hrValues.Max()}, Min: {hrValues.Min()}");
            }
        }
        
        if (streams.ContainsKey("velocity_smooth"))
        {
            var speedStream = streams["velocity_smooth"] as JsonArray;
            if (speedStream?.Count > 0)
            {
                var speedValues = speedStream.Select(x => (double)x).ToList();
                Console.WriteLine($"Speed - Avg: {speedValues.Average() * 3.6:F1} km/h, Max: {speedValues.Max() * 3.6:F1} km/h");
            }
        }
    }
}
```

## Real-World Scenarios

### Fitness Dashboard

```csharp
public class FitnessDashboard
{
    public async Task GenerateDashboardAsync(string accessToken)
    {
        var profile = await Athletes.GetAuthenticatedAthleteProfileAsync(accessToken);
        var activities = await Activities.GetAthletesActivitiesAsync(accessToken, page: 1, perPage: 30);
        
        Console.WriteLine("=== FITNESS DASHBOARD ===");
        Console.WriteLine($"Athlete: {profile["firstname"]} {profile["lastname"]}");
        Console.WriteLine($"Location: {profile["city"]}, {profile["state"]}");
        Console.WriteLine();
        
        // Weekly summary
        var weekAgo = DateTime.UtcNow.AddDays(-7);
        var recentActivities = activities.Where(a => 
            DateTime.Parse((string)a["start_date_local"]) >= weekAgo).ToList();
        
        var weeklyStats = new
        {
            Activities = recentActivities.Count,
            Distance = recentActivities.Sum(a => (double)a["distance"]) / 1000,
            Time = recentActivities.Sum(a => (int)a["moving_time"]),
            Elevation = recentActivities.Sum(a => (double)a["total_elevation_gain"])
        };
        
        Console.WriteLine("This Week:");
        Console.WriteLine($"  Activities: {weeklyStats.Activities}");
        Console.WriteLine($"  Distance: {weeklyStats.Distance:F1} km");
        Console.WriteLine($"  Time: {TimeSpan.FromSeconds(weeklyStats.Time)}");
        Console.WriteLine($"  Elevation: {weeklyStats.Elevation:F0} m");
        Console.WriteLine();
        
        // Activity breakdown
        var activityTypes = recentActivities.GroupBy(a => (string)a["type"])
                                          .OrderByDescending(g => g.Count());
        
        Console.WriteLine("Activity Breakdown:");
        foreach (var type in activityTypes)
        {
            var typeStats = type.ToList();
            var avgDistance = typeStats.Average(a => (double)a["distance"]) / 1000;
            Console.WriteLine($"  {type.Key}: {type.Count()} activities, avg {avgDistance:F1} km");
        }
    }
}
```

### Training Plan Generator

```csharp
public class TrainingPlanGenerator
{
    public async Task GenerateTrainingPlanAsync(string accessToken, string targetEvent, DateTime targetDate)
    {
        var activities = await Activities.GetAthletesActivitiesAsync(accessToken, page: 1, perPage: 100);
        var profile = await Athletes.GetAuthenticatedAthleteProfileAsync(accessToken);
        
        Console.WriteLine($"=== TRAINING PLAN FOR {targetEvent.ToUpper()} ===");
        Console.WriteLine($"Target Date: {targetDate:MMMM dd, yyyy}");
        Console.WriteLine($"Days until event: {(targetDate - DateTime.Now).Days}");
        Console.WriteLine();
        
        // Analyze current fitness level
        var recentRuns = activities.Where(a => 
            (string)a["type"] == "Run" && 
            DateTime.Parse((string)a["start_date_local"]) >= DateTime.Now.AddDays(-30)
        ).ToList();
        
        if (recentRuns.Any())
        {
            var avgPace = recentRuns.Average(a => (double)a["average_speed"]);
            var maxDistance = recentRuns.Max(a => (double)a["distance"]);
            
            Console.WriteLine("Current Fitness Level:");
            Console.WriteLine($"  Average Pace: {60 / (avgPace * 3.6):F1} min/km");
            Console.WriteLine($"  Longest Run: {maxDistance / 1000:F1} km");
            Console.WriteLine();
            
            // Generate training plan
            GenerateWeeklyPlan(targetDate, maxDistance / 1000);
        }
    }
    
    private void GenerateWeeklyPlan(DateTime targetDate, double currentLongestRun)
    {
        var weeksUntilEvent = (int)((targetDate - DateTime.Now).TotalDays / 7);
        
        Console.WriteLine("Recommended Training Plan:");
        for (int week = 1; week <= Math.Min(weeksUntilEvent, 12); week++)
        {
            var weekDate = DateTime.Now.AddDays(week * 7);
            var targetDistance = Math.Min(currentLongestRun + (week * 2), 42.2); // Cap at marathon distance
            
            Console.WriteLine($"Week {week} ({weekDate:MMM dd}):");
            Console.WriteLine($"  Long Run: {targetDistance:F1} km");
            Console.WriteLine($"  Tempo Run: {targetDistance * 0.6:F1} km");
            Console.WriteLine($"  Easy Runs: 2x {targetDistance * 0.4:F1} km");
            Console.WriteLine();
        }
    }
}
```

### Social Features

```csharp
public class SocialFeatures
{
    public async Task GetSocialFeedAsync(string accessToken)
    {
        var clubs = await Clubs.GetClubsAsync(accessToken);
        
        Console.WriteLine("=== SOCIAL FEED ===");
        
        foreach (var club in clubs)
        {
            Console.WriteLine($"Club: {club["name"]}");
            
            var clubId = (long)club["id"];
            var activities = await Clubs.GetClubActivitiesAsync(accessToken, clubId, page: 1, perPage: 5);
            
            foreach (var activity in activities)
            {
                var athlete = activity["athlete"];
                Console.WriteLine($"  {athlete["firstname"]} {athlete["lastname"]}: {activity["name"]}");
                Console.WriteLine($"    {activity["type"]} - {((double)activity["distance"] / 1000):F1} km");
                Console.WriteLine($"    {activity["start_date_local"]}");
                Console.WriteLine();
            }
        }
    }
    
    public async Task GetActivityKudosAsync(string accessToken, string activityId)
    {
        var kudos = await Activities.GetActivityKudosAsync(accessToken, activityId);
        
        Console.WriteLine($"Kudos for activity {activityId}:");
        foreach (var kudo in kudos)
        {
            Console.WriteLine($"  {kudo["firstname"]} {kudo["lastname"]}");
        }
    }
}
```

## Error Handling Examples

### Robust API Client

```csharp
public class RobustStravaClient
{
    private readonly string _accessToken;
    private readonly HttpClient _httpClient;
    
    public RobustStravaClient(string accessToken)
    {
        _accessToken = accessToken;
        _httpClient = new HttpClient();
        _httpClient.DefaultRequestHeaders.Authorization = 
            new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", accessToken);
    }
    
    public async Task<JsonArray?> GetActivitiesWithRetryAsync(int maxRetries = 3)
    {
        for (int attempt = 1; attempt <= maxRetries; attempt++)
        {
            try
            {
                return await Activities.GetAthletesActivitiesAsync(_accessToken);
            }
            catch (HttpRequestException ex) when (ex.StatusCode == System.Net.HttpStatusCode.TooManyRequests)
            {
                if (attempt < maxRetries)
                {
                    var retryAfter = GetRetryAfterSeconds(ex);
                    Console.WriteLine($"Rate limited. Waiting {retryAfter} seconds...");
                    await Task.Delay(retryAfter * 1000);
                }
                else
                {
                    throw new InvalidOperationException("Rate limit exceeded after multiple retries", ex);
                }
            }
            catch (HttpRequestException ex) when (ex.StatusCode == System.Net.HttpStatusCode.Unauthorized)
            {
                throw new InvalidOperationException("Access token is invalid or expired", ex);
            }
            catch (Exception ex)
            {
                if (attempt == maxRetries)
                    throw;
                
                Console.WriteLine($"Attempt {attempt} failed: {ex.Message}");
                await Task.Delay(1000 * attempt); // Exponential backoff
            }
        }
        
        return null;
    }
    
    private int GetRetryAfterSeconds(HttpRequestException ex)
    {
        // Parse Retry-After header if available
        if (ex.Data.Contains("RetryAfter"))
        {
            return (int)ex.Data["RetryAfter"];
        }
        return 60; // Default 1 minute
    }
}
```

## Next Steps

- **[API Reference](~/docs/api/)** - Complete API documentation
- **[Authentication Guide](authentication.md)** - Authentication patterns
- **[Best Practices](best-practices.md)** - Performance and security tips

---

**Build amazing Strava applications with these examples! 🚀** 