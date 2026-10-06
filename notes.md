## Component Dependency Map for YouTube Clone

Here's a visual breakdown of how all the components depend on and talk to each other:

```
┌─────────────────────────────────────────────────────────────────────┐
│                         index.html (DOM Root)                       │
│                      <div id="root"></div>                          │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  src/index.js (React Entry Point)                   │
│         Mounts App to DOM + loads index.css (global styles)         │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      src/App.js (Root Component)                    │
│                  BrowserRouter + Routes Setup                       │
└────────────────────────────┬────────────────────────────────────────┘
                             │
           ┌─────────────────┼─────────────────┐
           │                 │                 │
           ▼                 ▼                 ▼
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│ Navbar (Fixed)   │ │ Route: /         │ │ Route: /video/:id│
└────────┬─────────┘ │    (Feed)        │ │ (VideoDetail)    │
         │           └────────┬─────────┘ └────────┬─────────┘
         │                    │                    │
         ▼                    ▼                    ▼
   ┌──────────────┐  ┌──────────────────┐  ┌──────────────────┐
   │  SearchBar   │  │ Sidebar          │  │ Loader (if data  │
   │              │  │ (Categories)     │  │ still loading)   │
   │ Input + Icon │  │                  │  │                  │
   │ ↓ navigate   │  │ Buttons → state  │  │ ReactPlayer      │
   │ /search/term │  │ update           │  │ (YouTube embed)  │
   └──────────────┘  └────────┬─────────┘  │                  │
                              │            │ Channel info     │
                              ▼            │ View/Like counts │
                       ┌──────────────┐    │                  │
                       │ Feed fetches  │    └────────┬─────────┘
                       │ by category   │             │
                       └────────┬──────┘             │
                              │                    │
              ┌───────────────┴────────────────┬───┘
              │                                │
              ▼                                ▼
       ┌─────────────────┐            ┌──────────────────┐
       │ Videos (generic)│            │ Videos (generic) │
       │ list renderer   │            │ (related videos) │
       └────────┬────────┘            └────────┬─────────┘
                │                              │
         ┌──────┴──────┐                ┌──────┴──────┐
         │             │                │             │
         ▼             ▼                ▼             ▼
    ┌──────────┐  ┌──────────┐    ┌──────────┐  ┌──────────┐
    │VideoCard │  │ChannelCard│    │VideoCard │  │VideoCard │
    │ ↓        │  │ ↓         │    │ ↓        │  │ ↓        │
    │/video/:id│  │/channel/:id│    │/video/:id│  │/video/:id│
    └──────────┘  └──────────┘    └──────────┘  └──────────┘
```

---

## Detailed Dependency Layers

### **Layer 1: Data & API**
```
src/utils/fetchFromAPI.js
    ↓ Uses axios to call YouTube API v3
    ↓ Requires: REACT_APP_RAPID_API_KEY (env var)
    ↓ Exports: fetchFromAPI(url) function
    
Used by:
  • Feed.jsx
  • VideoDetail.jsx
  • ChannelDetail.jsx
  • SearchFeed.jsx
```

### **Layer 2: Constants & Styling**
```
src/utils/constants.js
    ├─ logo (image URL)
    ├─ categories (array of category objects)
    ├─ demoThumbnailUrl
    ├─ demoChannelUrl
    ├─ demoVideoUrl
    ├─ demoChannelTitle
    ├─ demoVideoTitle
    └─ demoProfilePicture

Used by:
  • Sidebar.jsx (categories)
  • VideoCard.jsx (demo URLs as fallbacks)
  • ChannelCard.jsx (demoProfilePicture)
  • Navbar.jsx (logo)

src/index.css (Global styles)
    ├─ .category-btn (Sidebar button styling)
    ├─ .search-bar (SearchBar input styling)
    ├─ .react-player (video player sizing)
    └─ Responsive media queries

Applied to:
  • All components (via global CSS)
```

### **Layer 3: Shared/Reusable Components**
```
Loader.jsx
    └─ Used by: Videos.jsx, VideoDetail.jsx

Videos.jsx
    ├─ Input: videos array, direction (optional)
    ├─ Logic: decides VideoCard vs ChannelCard rendering
    ├─ Uses: VideoCard, ChannelCard, Loader
    └─ Used by: Feed.jsx, VideoDetail.jsx, SearchFeed.jsx, ChannelDetail.jsx

VideoCard.jsx
    ├─ Input: video object with id and snippet
    ├─ Links to: /video/:videoId
    └─ Uses: demoThumbnailUrl, demoVideoUrl, demoChannelUrl, CheckCircleIcon

ChannelCard.jsx
    ├─ Input: channelDetail object with snippet and statistics
    ├─ Links to: /channel/:channelId
    └─ Uses: demoProfilePicture, CheckCircleIcon
```

### **Layer 4: Page-Level Components**
```
Navbar.jsx (Fixed header across all pages)
    ├─ Logo (Link to /)
    └─ SearchBar
        └─ onSubmit → navigate(/search/:searchTerm)

Feed.jsx (Route: /)
    ├─ State: selectedCategory, videos
    ├─ Effect: fetchFromAPI on category change
    ├─ Renders: Sidebar, Videos
    └─ Uses: fetchFromAPI

Sidebar.jsx (Category selector)
    ├─ Input: selectedCategory, setSelectedCategory
    ├─ Uses: categories from constants
    └─ Output: updates Feed's selectedCategory on click

VideoDetail.jsx (Route: /video/:id)
    ├─ Params: id (from URL)
    ├─ State: videoDetail, videos
    ├─ Effects: 
    │   ├─ Fetch video details (snippet, statistics)
    │   └─ Fetch related videos
    ├─ Renders: ReactPlayer, Video metadata, Videos (related)
    └─ Uses: fetchFromAPI, Loader

ChannelDetail.jsx (Route: /channel/:id)
    ├─ Params: id (from URL)
    ├─ State: channelDetail, videos
    ├─ Effects: 
    │   ├─ Fetch channel data
    │   └─ Fetch channel's videos
    ├─ Renders: Banner, ChannelCard, Videos
    └─ Uses: fetchFromAPI, ChannelCard, Videos

SearchFeed.jsx (Route: /search/:searchTerm)
    ├─ Params: searchTerm (from URL)
    ├─ State: videos
    ├─ Effect: fetchFromAPI on searchTerm change
    └─ Renders: Videos
        └─ Uses: fetchFromAPI
```

---

## Component Interaction Flow

### **User Action → Component Chain**

**Scenario 1: User clicks a category button**
```
Sidebar Button
    ↓ onClick
Feed (setSelectedCategory)
    ↓ useEffect triggered
fetchFromAPI('search?q=${selectedCategory}')
    ↓ API response
Feed (setVideos)
    ↓
Videos.jsx
    ↓
VideoCard + ChannelCard (render list)
```

**Scenario 2: User searches for a video**
```
SearchBar Input
    ↓ onSubmit
useNavigate('/search/${searchTerm}')
    ↓ URL change
SearchFeed mounts
    ↓ useParams().searchTerm
fetchFromAPI('search?q=${searchTerm}')
    ↓ API response
SearchFeed (setVideos)
    ↓
Videos.jsx
    ↓
VideoCard + ChannelCard (render list)
```

**Scenario 3: User clicks on a video card**
```
VideoCard
    ↓ Link to /video/:videoId
VideoDetail mounts
    ↓ useParams().id
fetchFromAPI('videos?id=${id}')  [Video metadata]
fetchFromAPI('search?relatedToVideoId=${id}')  [Related videos]
    ↓ Parallel API calls complete
VideoDetail (setVideoDetail, setVideos)
    ↓
ReactPlayer (renders video)
Metadata (title, views, likes)
Videos (related videos sidebar)
```

**Scenario 4: User clicks on a channel card**
```
ChannelCard
    ↓ Link to /channel/:channelId
ChannelDetail mounts
    ↓ useParams().id
fetchFromAPI('channels?id=${id}')  [Channel info]
fetchFromAPI('search?channelId=${id}')  [Channel videos]
    ↓ API calls complete
ChannelDetail (setChannelDetail, setVideos)
    ↓
ChannelCard (displays channel banner + avatar)
Videos (displays channel's video uploads)
```

---

## Dependency Tree (Inverted)

```
fetchFromAPI.js (Core API layer)
    ↑
    ├─ Feed.jsx
    ├─ VideoDetail.jsx
    ├─ ChannelDetail.jsx
    └─ SearchFeed.jsx

constants.js (Shared constants)
    ↑
    ├─ Navbar.jsx (logo)
    ├─ Sidebar.jsx (categories)
    ├─ VideoCard.jsx (demo URLs)
    └─ ChannelCard.jsx (demo image)

index.css (Global styling)
    ↑
    Applied to all components

Loader.jsx (Reusable spinner)
    ↑
    ├─ Videos.jsx
    └─ VideoDetail.jsx

VideoCard.jsx (Reusable card)
    ↑
    └─ Videos.jsx

ChannelCard.jsx (Reusable card)
    ↑
    └─ Videos.jsx

Videos.jsx (Generic list renderer)
    ↑
    ├─ Feed.jsx
    ├─ VideoDetail.jsx
    ├─ SearchFeed.jsx
    └─ ChannelDetail.jsx

Sidebar.jsx (Category selector)
    ↑
    └─ Feed.jsx

SearchBar.jsx (Search input)
    ↑
    └─ Navbar.jsx

Navbar.jsx (Fixed header)
    ↑
    └─ App.jsx (rendered on all pages)

Feed.jsx (Home page)
    ↑
    └─ App.jsx (Route: /)

VideoDetail.jsx (Video player page)
    ↑
    └─ App.jsx (Route: /video/:id)

SearchFeed.jsx (Search results page)
    ↑
    └─ App.jsx (Route: /search/:searchTerm)

ChannelDetail.jsx (Channel page)
    ↑
    └─ App.jsx (Route: /channel/:id)

App.jsx (Router setup)
    ↑
    └─ index.js (Entry point)
    
index.js (App start)
    ↑
    └─ index.html (DOM root)
```

---

## Key Observations

**Highly Reusable:**
- `Videos.jsx` is used by 4 different pages (Feed, VideoDetail, SearchFeed, ChannelDetail)
- `VideoCard` and `ChannelCard` are rendered by `Videos.jsx`, not directly by pages
- `Loader.jsx` is a small utility for consistent loading states

**Data Flow:**
- All API calls go through `fetchFromAPI` (single point of API logic)
- Page components control state, pass down to child components
- No props drilling: constants are imported directly where needed

**Routing:**
- `App.jsx` controls all navigation
- Each route mounts a separate page component (Feed, VideoDetail, etc.)
- SearchBar in Navbar uses `useNavigate` to programmatically redirect

**Styling:**
- Global CSS applies to all components
- Material UI components handle layout (Stack, Box, Typography)
- Custom classes (`.category-btn`, `.search-bar`) add page-specific styling