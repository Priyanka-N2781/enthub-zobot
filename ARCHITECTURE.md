# EntHub Zobot - System Architecture

## 🏗️ Architecture Overview

EntHub Zobot is a multi-tier entertainment recommendation system built on Zoho SalesIQ with privacy-first personalization.

```
┌─────────────────────────────────────────────────────────────┐
│                    USER INTERACTION LAYER                    │
│        (Zoho SalesIQ Codeless Bot Widget/Chat)              │
└──────────────────────────┬──────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   [Suggest          [Book an             [Surprise
   Something]        Event]                Me]
        │                  │                  │
        ▼                  ▼                  ▼
┌──────────────────────────────────────────────────────────────┐
│              BOT ORCHESTRATION LAYER                         │
│         (Zoho SalesIQ Codeless Bot Flows)                   │
│  - Question Blocks  - Condition Blocks  - Action/Plug Calls │
└──────────────────────────┬──────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
┌─────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│  PLUG LAYER 1   │ │  PLUG LAYER 2    │ │  PLUG LAYER 3    │
│ (Deluge Plugs)  │ │ (Deluge Plugs)   │ │ (AI Plugs)       │
├─────────────────┤ ├──────────────────┤ ├──────────────────┤
│ suggest_movie   │ │ suggest_books    │ │ ai_vibe_planner  │
│ event_finder    │ │ suggest_games    │ │                  │
│                 │ │ suggest_food     │ │ Privacy Tagging: │
│ Input: user     │ │                  │ │ - Mood→Tag       │
│ prefs + context │ │ Input: genre,    │ │ - No raw data    │
└────────┬────────┘ │ rating, cuisine  │ │ - Transparent    │
         │          └────────┬─────────┘ └────────┬─────────┘
         │                   │                     │
         └───────────────────┼─────────────────────┘
                             │
┌───────────────────────────────────────────────────────────────┐
│          OAUTH 2.0 CONNECTION LAYER                          │
│   (Zoho Connections - Secure API Key/Token Management)      │
├───────────────────────────────────────────────────────────────┤
│ tmdb_connection    google_books_connection  rawg_connection │
│ spoonacular_conn   ticketmaster_connection  openai_connection│
└────┬─────────────┬───────────────────┬─────────┬───────────┬┘
     │             │                   │         │           │
     ▼             ▼                   ▼         ▼           ▼
  ┌──────┐    ┌────────────┐     ┌──────┐  ┌──────────┐  ┌──────┐
  │ TMDB │    │ Google     │     │ RAWG │  │Ticketm.  │  │OpenAI│
  │Movies│    │ Books      │     │Games │  │Events    │  │ LLM  │
  └──────┘    └────────────┘     └──────┘  └──────────┘  └──────┘
     │             │                 │          │            │
     └─────────────────────────────────┬────────────────────┘
                                       │
                    ┌──────────────────┴─────────────┐
                    │                                │
                    ▼                                ▼
          ┌─────────────────┐          ┌──────────────────────┐
          │  CACHE LAYER    │          │  DATA AGGREGATION    │
          │ (1-hour TTL)    │          │  & RESPONSE FORMAT   │
          └────────┬────────┘          └──────────┬───────────┘
                   │                             │
                   └─────────────┬───────────────┘
                                 │
                    ┌────────────────────────┐
                    │ RESPONSE FORMATTING    │
                    │ - Cards/Carousels      │
                    │ - Follow-up Actions    │
                    │ - Privacy Notices      │
                    └────────────┬───────────┘
                                 │
                    ┌────────────────────────┐
                    │  BOT RESPONSE LAYER    │
                    │  (Back to User)        │
                    └────────────────────────┘
```

## 🔌 Plug-Based Microservices Pattern

Each Plug operates as an independent microservice:

**Plug Lifecycle:**
1. Bot flow collects user input
2. Action block calls Plug with parameters
3. Plug validates input
4. Plug invokes third-party API via OAuth Connection
5. Plug processes response
6. Plug returns structured JSON
7. Bot displays response/follow-up actions

## 🔐 OAuth 2.0 Security Model

```
User Input → Bot Flow → Plug Request
                          │
                          ▼
                  Check OAuth Connection
                          │
    ┌───────────┬─────────┴────────┬────────────┐
    │           │                  │            │
   Valid      Token            Refresh      Token
   Token?    Expired?           Token      Revoked?
    │           │                  │            │
   YES          │ Attempt           │           │
    │          │ Refresh            │           │
    │          ├─ Success ─────┐    │           │
    │          │               │    │           │
    └──────────┴──────┬────────┴────┴──────┐    │
                      │                    │    │
                      ▼                    ▼    ▼
              invokeurl(connection)   Re-authenticate
                      │                User Required
                      ▼
           Third-party API Call
                      │
                      ▼
          Response Processing & Caching
```

## 🎭 Privacy-Preserving Personalization

### Mood Tagging Flow (No Raw Data Stored)

```
User Free Text Input:
"Stressed about work, need to relax"
        │
        ▼
┌─ Tag Mood Function ─┐
│ Check keywords:     │
│ - Contains stress?  │
│ - Contains anxious? │
└────────┬────────────┘
         │
         ▼
Generic Privacy-Safe Tag:
"Chill & Relax"
         │
         ▼
Use Tag in AI Prompt
(Original text NEVER logged)
```

### Data Retention Policy

- **Session Data**: Cleared after bot interaction ends
- **Preferences**: Used for current session only (unless user opts-in)
- **Raw Mood Text**: Never persisted; converted to tags immediately
- **API Responses**: Cached for 1 hour (no PII)
- **Error Logs**: Sanitized (no sensitive parameters)

## 📊 Data Flow & Transformation

```
TMDB API Response:
{
  "results": [
    {
      "title": "Inception",
      "release_date": "2010-07-16",
      "vote_average": 8.8,
      "poster_path": "/...",
      ...
    }
  ]
}
        │
        ▼
┌─ Plug Processing ─┐
│ - Filter movies   │
│ - Pick random     │
│ - Extract fields  │
│ - Format for bot  │
└────────┬──────────┘
         │
         ▼
Bot Response:
{
  "movie": {
    "title": "Inception",
    "year": "2010",
    "rating": "8.8",
    "overview": "...",
    "poster_url": "https://..."
  },
  "isSuccessful": true
}
        │
        ▼
Bot Displays:
🎬 **Inception** (2010)
⭐ Rating: 8.8/10
📝 "A mind-bending masterpiece..."
[Watch Trailer] [Book Now]
```

## ⚡ Error Handling & Resilience

```
Plug Execution Flow:
        │
        ▼
    Try Block
        │
   ┌────┴────┐
   │          │
  YES        NO
   │          │
   ▼          ▼
Success   Catch Block
   │          │
   │          ├─ Log error (sanitized)
   │          ├─ Check error type
   │          │
   │          ├─ Rate limit? → Fallback cache
   │          ├─ Invalid input? → User message
   │          ├─ API down? → Suggestion from cache
   │          └─ Other? → Generic "Try later"
   │          │
   └────┬─────┘
        │
        ▼
   Return Response
   (Always structured JSON)
```

## 🔄 Request-Response Cycle Timing

```
User Query → Bot Processing → Plug Execution → API Response
   0ms          50ms              100ms        (varies)
                                   │
                            ┌──────────────┐
                            │ Max 30 secs  │
                            │ timeout      │
                            └──────────────┘
                                   │
                                   ▼
                          Parse & Format
                                   │
                                   ▼
                          Return to User
                               100ms

Total Time: 100-500ms (depending on API response)
```

## 📈 Scalability Considerations

1. **Stateless Plugs**: Each plug call is independent (no session state)
2. **Connection Pooling**: OAuth connections reused via Zoho SalesIQ
3. **Caching**: 1-hour TTL reduces API calls
4. **Rate Limiting**: Respected per third-party API
5. **Async Processing**: Long operations can queue

## 🚀 Deployment Architecture

```
Development
    │
    ▼
[Test Plugs in SalesIQ Sandbox]
    │
    ▼
Production
    │
    ├─ Zoho SalesIQ Account (Enterprise)
    ├─ OAuth Connections Configured
    ├─ Plugs Published & Versioned
    ├─ Bot Flows Deployed
    └─ Widget Deployed to Website
        │
        ▼
    Monitoring:
    - Bot Interaction Metrics
    - Plug Error Rates
    - API Response Times
    - Cache Hit Rates
```

## 🔗 Integration Points

### Third-Party APIs Used

| Service | Purpose | Latency | Auth |
|---------|---------|---------|------|
| TMDB | Movies | 200-500ms | API Key |
| Google Books | Books | 300-800ms | OAuth 2.0 |
| RAWG | Games | 250-600ms | API Key |
| Spoonacular | Food | 300-700ms | API Key |
| Ticketmaster | Events | 400-1000ms | API Key |
| OpenAI | AI Planner | 2-5 secs | OAuth 2.0 |

### Zoho Integration Points

- **SalesIQ Widgets**: Embedded on client websites
- **CRM Integration**: Visitor data available to bot
- **Analytics**: Bot interaction logged
- **API Rate Limits**: Respected per Zoho tier

## 📋 Future Architecture Enhancements

1. **Database Layer**: Store user preferences (with consent)
2. **Message Queue**: Async processing for slow APIs
3. **ML Model**: Personalization based on interaction history
4. **Analytics Pipeline**: Real-time metrics & dashboards
5. **Multi-language Support**: Localization layer
6. **Mobile App**: Native iOS/Android integration

---

**Architecture Version:** 1.0  
**Last Updated:** November 2025  
**Built for:** Zoho Cliqtrix Internship Challenge
