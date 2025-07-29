---
layout: default
title: API Reference
description: Complete API documentation for StravaAPILibary
---

# StravaAPILibary API Reference

Welcome to the comprehensive API documentation for **StravaAPILibary** - a powerful .NET library for seamless Strava API integration.

## 🚀 Quick Start

```csharp
// Install via NuGet
Install-Package StravaAPILibary

// Basic usage
using StravaAPILibary.API;
using StravaAPILibary.Authentication;

// Authenticate and get athlete profile
var credentials = new Credentials(clientId, clientSecret);
var auth = new UserAuthentication(credentials);
var accessToken = await auth.GetAccessTokenAsync(authorizationCode);
var athlete = await Athletes.GetAuthenticatedAthleteProfileAsync(accessToken);
```

## 📚 API Overview

StravaAPILibary provides a comprehensive .NET wrapper for the Strava API, organized into the following main areas:

### 🔐 Authentication & Security

**OAuth 2.0 Implementation**
- **[Credentials](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/Authentication/credentials.cs)** - Secure credential management
- **[UserAuthentication](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/Authentication/UserAuthentication.cs)** - Complete OAuth 2.0 flow

**Key Features:**
- Automatic token refresh
- Secure credential storage
- Multiple authentication flows
- Error handling and retry logic

```csharp
// Initialize authentication
var credentials = new Credentials
{
    ClientId = "your_client_id",
    ClientSecret = "your_client_secret"
};

var auth = new UserAuthentication(credentials);

// Get authorization URL
var authUrl = auth.GetAuthorizationUrl("read,activity:read_all");

// Exchange code for token
var accessToken = await auth.GetAccessTokenAsync(authorizationCode);
```

### 🏃‍♂️ Activities Management

**Core Activity Operations**
- **[Activities](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Activities.cs)** - Complete activity management
- **[DetailedActivity](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/Models/Activities/DetailedActivity.cs)** - Full activity data
- **[SummaryActivity](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/Models/Activities/SummaryActivity.cs)** - Activity summaries
- **[UpdatableActivity](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/Models/Activities/UpdatableActivity.cs)** - Activity updates

**Available Operations:**
- Get athlete activities
- Get activity by ID
- Create new activities
- Update activity details
- Delete activities
- Get activity comments
- Get activity kudos

```csharp
// Get recent activities
var activities = await Activities.GetAthletesActivitiesAsync(accessToken, perPage: 20);

// Get specific activity
var activity = await Activities.GetActivityByIdAsync(accessToken, activityId);

// Update activity
var updateData = new UpdatableActivity
{
    Name = "Updated Activity Name",
    Description = "Updated description",
    Type = ActivityType.Run
};
await Activities.UpdateActivityByIdAsync(accessToken, activityId, updateData);

// Get activity streams
var streams = await Streams.GetActivityStreamsAsync(accessToken, activityId, 
    new[] { "time", "distance", "latlng", "altitude", "velocity_smooth" });
```

### 👤 Athlete Profiles

**Athlete Management**
- **[Athletes](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Athletes.cs)** - Athlete operations
- **[DetailedAthlete](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/Models/Athletes/DetailedAthlete.cs)** - Complete athlete data
- **[SummaryAthlete](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/Models/Athletes/SummaryAthlete.cs)** - Athlete summaries

**Features:**
- Get authenticated athlete profile
- Get athlete statistics
- Get athlete zones
- Get athlete activities
- Get athlete followers/following

```csharp
// Get authenticated athlete
var athlete = await Athletes.GetAuthenticatedAthleteProfileAsync(accessToken);

// Get athlete stats
var stats = await Athletes.GetAthleteStatsAsync(accessToken, athleteId);

// Get athlete zones
var zones = await Athletes.GetAthleteZonesAsync(accessToken);
```

### 🏢 Clubs & Communities

**Club Management**
- **[Clubs](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Clubs.cs)** - Club operations
- **[DetailedClub](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/Models/Clubs/DetailedClub.cs)** - Club information
- **[ClubActivity](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/Models/Clubs/ClubActivity.cs)** - Club activities

**Capabilities:**
- Get club information
- Get club members
- Get club activities
- Get club announcements
- Join/leave clubs

```csharp
// Get club details
var club = await Clubs.GetClubByIdAsync(accessToken, clubId);

// Get club members
var members = await Clubs.GetClubMembersByIdAsync(accessToken, clubId);

// Get club activities
var activities = await Clubs.GetClubActivitiesByIdAsync(accessToken, clubId);
```

### 🏁 Segments & Efforts

**Segment Operations**
- **[Segments](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Segments.cs)** - Segment management
- **[DetailedSegment](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/Models/Segments/DetailedSegment.cs)** - Segment data
- **[SegmentEfforts](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/SegmentsEfforts.cs)** - Effort tracking

**Features:**
- Get segment by ID
- Get segment leaderboard
- Get segment efforts
- Star/unstar segments
- Get starred segments

```csharp
// Get segment details
var segment = await Segments.GetSegmentByIdAsync(accessToken, segmentId);

// Get segment leaderboard
var leaderboard = await Segments.GetSegmentLeaderboardAsync(accessToken, segmentId);

// Get segment efforts
var efforts = await Segments.GetSegmentEffortsAsync(accessToken, segmentId);

// Star a segment
await Segments.StarSegmentAsync(accessToken, segmentId, true);
```

### 🗺️ Routes & Navigation

**Route Management**
- **[Routes](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Routes.cs)** - Route operations
- **[Route](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/Models/Routes/Route.cs)** - Route data

**Capabilities:**
- Get route by ID
- Get athlete routes
- Export route GPX
- Get route streams

```csharp
// Get route details
var route = await Routes.GetRouteByIdAsync(accessToken, routeId);

// Get athlete routes
var routes = await Routes.GetAthleteRoutesAsync(accessToken);

// Export route as GPX
var gpxData = await Routes.ExportRouteGPXAsync(accessToken, routeId);
```

### 📊 Streams & Data Analysis

**Activity Streams**
- **[Streams](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Streams.cs)** - Stream operations
- **[StreamSet](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/Models/Streams/StreamSet.cs)** - Stream collections

**Available Stream Types:**
- Time, Distance, LatLng
- Altitude, Velocity, Moving
- Heart Rate, Cadence, Power
- Temperature, Grade, Watts

```csharp
// Get activity streams
var streams = await Streams.GetActivityStreamsAsync(accessToken, activityId, 
    new[] { "time", "distance", "latlng", "altitude", "velocity_smooth" });

// Get segment effort streams
var effortStreams = await Streams.GetSegmentEffortStreamsAsync(accessToken, segmentEffortId,
    new[] { "distance", "altitude", "latlng" });

// Get route streams
var routeStreams = await Streams.GetRouteStreamsAsync(accessToken, routeId);
```

### 🚴‍♂️ Equipment & Gear

**Gear Management**
- **[Gears](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Gears.cs)** - Equipment operations
- **[DetailedGear](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/Models/Gears/DetailedGear.cs)** - Gear information

**Features:**
- Get gear by ID
- Get athlete gear
- Track gear usage
- Gear statistics

```csharp
// Get gear details
var gear = await Gears.GetGearByIdAsync(accessToken, gearId);

// Get athlete gear
var athleteGear = await Gears.GetAthleteGearAsync(accessToken);
```

### 📤 Uploads & Data Import

**Activity Uploads**
- **[Uploads](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Uploads.cs)** - Upload operations
- **[Upload](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/Models/Uploads/Upload.cs)** - Upload status

**Supported Formats:**
- GPX, TCX, FIT files
- Manual activity creation
- Upload status tracking
- Error handling

```csharp
// Upload activity file
var upload = await Uploads.CreateUploadAsync(accessToken, 
    filePath: "activity.gpx",
    activityType: ActivityType.Run,
    name: "Morning Run");

// Check upload status
var status = await Uploads.GetUploadByIdAsync(accessToken, upload.Id);
```

## 🔍 Advanced Features

### Error Handling

```csharp
try
{
    var activity = await Activities.GetActivityByIdAsync(accessToken, activityId);
}
catch (StravaApiException ex)
{
    switch (ex.StatusCode)
    {
        case 401:
            // Handle unauthorized - refresh token
            break;
        case 404:
            // Handle not found
            break;
        case 429:
            // Handle rate limiting
            break;
    }
}
```

### Rate Limiting

```csharp
// The library automatically handles rate limiting
// You can also implement custom retry logic
var retryPolicy = Policy
    .Handle<StravaApiException>(ex => ex.StatusCode == 429)
    .WaitAndRetryAsync(3, retryAttempt => 
        TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)));
```

### Batch Operations

```csharp
// Get multiple activities efficiently
var activityIds = new[] { 123456, 123457, 123458 };
var activities = new List<DetailedActivity>();

foreach (var id in activityIds)
{
    var activity = await Activities.GetActivityByIdAsync(accessToken, id);
    activities.Add(activity);
}
```

## 📋 Common Use Cases

### 1. Activity Analysis Dashboard

```csharp
// Get athlete's recent activities with detailed analysis
var activities = await Activities.GetAthletesActivitiesAsync(accessToken, perPage: 50);
var analysis = new List<ActivityAnalysis>();

foreach (var activity in activities)
{
    var streams = await Streams.GetActivityStreamsAsync(accessToken, activity.Id, 
        new[] { "time", "distance", "heartrate", "velocity_smooth" });
    
    analysis.Add(new ActivityAnalysis
    {
        Activity = activity,
        AverageHeartRate = streams.HeartrateStream?.Data?.Average() ?? 0,
        MaxSpeed = streams.VelocitySmoothStream?.Data?.Max() ?? 0,
        TotalDistance = activity.Distance
    });
}
```

### 2. Segment Performance Tracking

```csharp
// Track segment performance over time
var segmentId = 123456;
var efforts = await Segments.GetSegmentEffortsAsync(accessToken, segmentId);

var performance = efforts.Select(effort => new SegmentPerformance
{
    Date = effort.StartDate,
    Time = effort.ElapsedTime,
    AverageWatts = effort.AverageWatts,
    MaxWatts = effort.MaxWatts
}).OrderBy(p => p.Date).ToList();
```

### 3. Club Activity Monitoring

```csharp
// Monitor club activities and engagement
var clubId = 123456;
var clubActivities = await Clubs.GetClubActivitiesByIdAsync(accessToken, clubId);

var clubStats = new ClubStatistics
{
    TotalActivities = clubActivities.Count,
    TotalDistance = clubActivities.Sum(a => a.Distance),
    AverageSpeed = clubActivities.Average(a => a.AverageSpeed),
    MostActiveMember = clubActivities
        .GroupBy(a => a.Athlete.Id)
        .OrderByDescending(g => g.Count())
        .First().Key
};
```

## 🎯 Best Practices

### 1. Token Management
```csharp
// Store tokens securely and refresh when needed
var tokenManager = new TokenManager();
await tokenManager.RefreshTokenIfNeededAsync(accessToken);
```

### 2. Efficient Data Loading
```csharp
// Use pagination for large datasets
var allActivities = new List<SummaryActivity>();
var page = 1;
const int perPage = 200;

do
{
    var activities = await Activities.GetAthletesActivitiesAsync(accessToken, page, perPage);
    allActivities.AddRange(activities);
    page++;
} while (activities.Count == perPage);
```

### 3. Error Recovery
```csharp
// Implement robust error handling
var policy = Policy
    .Handle<StravaApiException>()
    .Or<HttpRequestException>()
    .WaitAndRetryAsync(3, retryAttempt => 
        TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),
    onRetry: (exception, timeSpan, retryCount, context) =>
    {
        // Log retry attempt
        Console.WriteLine($"Retry {retryCount} after {timeSpan.TotalSeconds}s");
    });
```

## 🔗 Resources & Support

### 📖 Documentation
- **[Getting Started Guide](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/docs/articles/getting-started.md)**
- **[Authentication Guide](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/docs/articles/authentication.md)**
- **[Examples & Tutorials](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/docs/articles/examples.md)**

### 🛠️ Development
- **[GitHub Repository](https://github.com/Deltatoolbox/StravaAPILibary)** - Source code and issues
- **[NuGet Package](https://www.nuget.org/packages/StravaAPILibary)** - Package installation
- **[Strava API Documentation](https://developers.strava.com/docs/reference/)** - Official Strava API docs

### 💬 Community
- **GitHub Issues** - Report bugs and request features
- **GitHub Discussions** - Ask questions and share experiences
- **Contributing Guide** - Help improve the library

---

**Ready to get started?** Check out our [Getting Started Guide](https://github.com/Deltatoolbox/StravaAPILibary/blob/master/docs/articles/getting-started.md) for your first steps with StravaAPILibary! 🚀

<style>
.search-container {
  margin: 2rem 0;
}

.search-input {
  width: 100%;
  padding: 0.75rem 1rem;
  border: 1px solid var(--border-color);
  border-radius: 6px;
  background-color: var(--code-bg);
  color: var(--text-primary);
  font-size: 1rem;
}

.search-results {
  margin-top: 1rem;
  max-height: 400px;
  overflow-y: auto;
  border: 1px solid var(--border-color);
  border-radius: 6px;
  background-color: var(--secondary-bg);
  display: none;
}

.search-result-item {
  padding: 0.75rem 1rem;
  border-bottom: 1px solid var(--border-color);
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.search-result-item:hover {
  background-color: var(--tertiary-bg);
}

.search-result-item:last-child {
  border-bottom: none;
}

.api-section {
  margin: 2rem 0;
  padding: 1.5rem;
  border: 1px solid var(--border-color);
  border-radius: 8px;
  background-color: var(--secondary-bg);
}

.code-example {
  background-color: var(--code-bg);
  border-radius: 6px;
  padding: 1rem;
  margin: 1rem 0;
  overflow-x: auto;
}

.feature-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1.5rem;
  margin: 2rem 0;
}

.feature-card {
  padding: 1.5rem;
  border: 1px solid var(--border-color);
  border-radius: 8px;
  background-color: var(--secondary-bg);
}

.feature-card h3 {
  margin-top: 0;
  color: var(--text-primary);
}

.feature-card ul {
  margin: 1rem 0;
  padding-left: 1.5rem;
}
</style>

<script>
document.addEventListener('DOMContentLoaded', function() {
  const searchInput = document.getElementById('api-search');
  const searchResults = document.getElementById('search-results');
  
  if (searchInput && searchResults) {
    searchInput.addEventListener('input', function() {
      const query = this.value.toLowerCase().trim();
      
      if (query.length < 2) {
        searchResults.style.display = 'none';
        return;
      }
      
      // Enhanced search implementation
      const results = [
        { title: 'Activities API', url: 'https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Activities.cs' },
        { title: 'Athletes API', url: 'https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Athletes.cs' },
        { title: 'Authentication', url: 'https://github.com/Deltatoolbox/StravaAPILibary/blob/master/Authentication/UserAuthentication.cs' },
        { title: 'Clubs API', url: 'https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Clubs.cs' },
        { title: 'Segments API', url: 'https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Segments.cs' },
        { title: 'Routes API', url: 'https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Routes.cs' },
        { title: 'Streams API', url: 'https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Streams.cs' },
        { title: 'Uploads API', url: 'https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Uploads.cs' },
        { title: 'Gears API', url: 'https://github.com/Deltatoolbox/StravaAPILibary/blob/master/API/Gears.cs' }
      ].filter(item => 
        item.title.toLowerCase().includes(query)
      );
      
      if (results.length > 0) {
        searchResults.innerHTML = results.map(result => 
          `<div class="search-result-item" onclick="window.open('${result.url}', '_blank')">
            ${result.title}
          </div>`
        ).join('');
        searchResults.style.display = 'block';
      } else {
        searchResults.innerHTML = '<div class="search-result-item">No results found</div>';
        searchResults.style.display = 'block';
      }
    });
    
    // Hide results when clicking outside
    document.addEventListener('click', function(e) {
      if (!searchInput.contains(e.target) && !searchResults.contains(e.target)) {
        searchResults.style.display = 'none';
      }
    });
  }
});
</script> 