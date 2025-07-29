---
layout: default
title: API Reference
description: Complete API documentation for StravaAPILibary
---

# API Reference

Welcome to the complete API documentation for **StravaAPILibary**. This reference covers all classes, methods, and types available in the library.

## 📚 API Overview

StravaAPILibary provides a comprehensive .NET wrapper for the Strava API, organized into the following main areas:

### 🔐 Authentication
- **[Credentials]({{ '/api/StravaAPILibary.Authentication.Credentials.html' | relative_url }})** - OAuth 2.0 credential management
- **[UserAuthentication]({{ '/api/StravaAPILibary.Authentication.UserAuthentication.html' | relative_url }})** - Authentication flow handling

### 🏃‍♂️ Activities
- **[Activities]({{ '/api/StravaAPILibary.API.Activities.html' | relative_url }})** - Activity management and retrieval
- **[DetailedActivity]({{ '/api/StravaAPILibary.Models.Activities.DetailedActivity.html' | relative_url }})** - Complete activity information
- **[SummaryActivity]({{ '/api/StravaAPILibary.Models.Activities.SummaryActivity.html' | relative_url }})** - Activity summaries
- **[UpdatableActivity]({{ '/api/StravaAPILibary.Models.Activities.UpdatableActivity.html' | relative_url }})** - Activity update operations

### 👤 Athletes
- **[Athletes]({{ '/api/StravaAPILibary.API.Athletes.html' | relative_url }})** - Athlete profile management
- **[DetailedAthlete]({{ '/api/StravaAPILibary.Models.Athletes.DetailedAthlete.html' | relative_url }})** - Complete athlete information
- **[SummaryAthlete]({{ '/api/StravaAPILibary.Models.Athletes.SummaryAthlete.html' | relative_url }})** - Athlete summaries

### 🏢 Clubs
- **[Clubs]({{ '/api/StravaAPILibary.API.Clubs.html' | relative_url }})** - Club management
- **[DetailedClub]({{ '/api/StravaAPILibary.Models.Clubs.DetailedClub.html' | relative_url }})** - Club information
- **[ClubActivity]({{ '/api/StravaAPILibary.Models.Clubs.ClubActivity.html' | relative_url }})** - Club activities

### 🏁 Segments
- **[Segments]({{ '/api/StravaAPILibary.API.Segments.html' | relative_url }})** - Segment operations
- **[DetailedSegment]({{ '/api/StravaAPILibary.Models.Segments.DetailedSegment.html' | relative_url }})** - Segment information
- **[SegmentEfforts]({{ '/api/StravaAPILibary.API.SegmentsEfforts.html' | relative_url }})** - Segment effort tracking

### 🗺️ Routes
- **[Routes]({{ '/api/StravaAPILibary.API.Routes.html' | relative_url }})** - Route management
- **[Route]({{ '/api/StravaAPILibary.Models.Routes.Route.html' | relative_url }})** - Route information

### 📊 Streams
- **[Streams]({{ '/api/StravaAPILibary.API.Streams.html' | relative_url }})** - Activity stream data
- **[StreamSet]({{ '/api/StravaAPILibary.Models.Streams.StreamSet.html' | relative_url }})** - Stream data collections

### 🚴‍♂️ Gears
- **[Gears]({{ '/api/StravaAPILibary.API.Gears.html' | relative_url }})** - Equipment management
- **[DetailedGear]({{ '/api/StravaAPILibary.Models.Gears.DetailedGear.html' | relative_url }})** - Equipment information

### 📤 Uploads
- **[Uploads]({{ '/api/StravaAPILibary.API.Uploads.html' | relative_url }})** - Activity upload functionality
- **[Upload]({{ '/api/StravaAPILibary.Models.Uploads.Upload.html' | relative_url }})** - Upload status and information

## 🔍 Quick Search

<div class="search-container">
  <input type="text" id="api-search" class="search-input" placeholder="Search API documentation...">
  <div id="search-results" class="search-results"></div>
</div>

## 📋 Common Operations

### Get Athlete Profile
```csharp
var athlete = await Athletes.GetAuthenticatedAthleteProfileAsync(accessToken);
```

### Get Recent Activities
```csharp
var activities = await Activities.GetAthletesActivitiesAsync(accessToken, perPage: 10);
```

### Get Activity Details
```csharp
var activity = await Activities.GetActivityByIdAsync(accessToken, activityId);
```

### Get Club Information
```csharp
var club = await Clubs.GetClubByIdAsync(accessToken, clubId);
```

### Get Segment Details
```csharp
var segment = await Segments.GetSegmentByIdAsync(accessToken, segmentId);
```

## 🎯 Getting Started

If you're new to the library, start with:

1. **[Getting Started]({{ '/articles/getting-started/' | relative_url }})** - Basic setup and installation
2. **[Authentication Guide]({{ '/articles/authentication/' | relative_url }})** - OAuth 2.0 setup
3. **[Examples]({{ '/articles/examples/' | relative_url }})** - Practical usage examples

## 🔗 Related Resources

- **[GitHub Repository](https://github.com/Deltatoolbox/StravaAPILibary)** - Source code and issues
- **[NuGet Package](https://www.nuget.org/packages/StravaAPILibary)** - Package installation
- **[Strava API Documentation](https://developers.strava.com/docs/reference/)** - Official Strava API docs

---

**Need help? Check our examples or create an issue on GitHub! 💡**

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
</style>

<script>
document.addEventListener('DOMContentLoaded', function() {
  const searchInput = document.getElementById('api-search');
  const searchResults = document.getElementById('search-results');
  
  searchInput.addEventListener('input', function() {
    const query = this.value.toLowerCase().trim();
    
    if (query.length < 2) {
      searchResults.style.display = 'none';
      return;
    }
    
    // Simple search implementation
    // In a real implementation, you'd want to index the API documentation
    const results = [
      { title: 'Activities API', url: '{{ "/api/StravaAPILibary.API.Activities.html" | relative_url }}' },
      { title: 'Athletes API', url: '{{ "/api/StravaAPILibary.API.Athletes.html" | relative_url }}' },
      { title: 'Authentication', url: '{{ "/api/StravaAPILibary.Authentication.UserAuthentication.html" | relative_url }}' },
      { title: 'Clubs API', url: '{{ "/api/StravaAPILibary.API.Clubs.html" | relative_url }}' },
      { title: 'Segments API', url: '{{ "/api/StravaAPILibary.API.Segments.html" | relative_url }}' },
      { title: 'Routes API', url: '{{ "/api/StravaAPILibary.API.Routes.html" | relative_url }}' },
      { title: 'Streams API', url: '{{ "/api/StravaAPILibary.API.Streams.html" | relative_url }}' },
      { title: 'Uploads API', url: '{{ "/api/StravaAPILibary.API.Uploads.html" | relative_url }}' }
    ].filter(item => 
      item.title.toLowerCase().includes(query)
    );
    
    if (results.length > 0) {
      searchResults.innerHTML = results.map(result => 
        `<div class="search-result-item" onclick="window.location.href='${result.url}'">
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
});
</script> 