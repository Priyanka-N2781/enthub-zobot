# EntHub Zobot - Entertainment Chatbot

🎬 **AI-Powered Entertainment Recommendations for Zoho SalesIQ**

## Overview

EntHub Zobot is a sophisticated entertainment chatbot built for Zoho SalesIQ using the Codeless Bot Builder with 6+ Deluge plugs. It provides personalized suggestions for movies, books, games, food, and event bookings through a seamless, privacy-preserving conversational interface.

### Key Features

✨ **Three Main Functions:**
- **Suggest Something** - Get recommendations for Movies, Books, Games, Food
- **Book an Event** - Find and book movies, concerts, talkshows, and more
- **Surprise Me** - AI-powered vibe planner for personalized night plans

🔒 **Privacy-First Architecture:**
- OAuth 2.0 authentication for all third-party integrations
- Privacy-preserving mood tagging (no raw personal data stored)
- Session-based recommendations without data persistence

🚀 **Technology Stack:**
- **Platform:** Zoho SalesIQ Codeless Bot Builder
- **Language:** Deluge (Zoho proprietary scripting language)
- **Integrations:** TMDB, Google Books, RAWG, Spoonacular, Ticketmaster, OpenAI
- **Auth:** OAuth 2.0 Connections

---

## Project Structure

```
enthub-zobot/
├── deluge/
│   ├── suggest_movie_plug.ds           # Movie recommendations via TMDB API
│   ├── suggest_books_plug.ds           # Book recommendations via Google Books API
│   ├── suggest_games_food_plug.ds      # Games (RAWG) & Food (Spoonacular) suggestions
│   ├── event_finder_plug.ds            # Event discovery via Ticketmaster API
│   ├── ai_vibe_planner_plug.ds         # AI-powered personalized night planner
│   └── helper_utilities.ds             # Shared utility functions
├── docs/
│   ├── BOT_FLOW_DIAGRAM.md            # Visual bot conversation flows
│   ├── PLUGS_CONFIGURATION.md         # Plug input/output specifications
│   ├── OAUTH_SETUP_GUIDE.md           # Step-by-step OAuth connection setup
│   ├── API_INTEGRATION_CONTRACTS.md   # API request/response formats
│   └── DELUGE_FUNCTION_REFERENCE.md   # Function documentation
├── frontend/
│   ├── index.html                      # SalesIQ widget integration page
│   ├── styles.css                      # Styling for demo page
│   └── bot-widget-config.js            # Widget configuration
├── README.md                           # This file
├── ARCHITECTURE.md                     # System design & architecture
├── LICENSE                             # MIT License
└── .gitignore                          # Python/Node.js ignores
```

---

## 6+ Deluge Plugs

### 1. **suggest_movie_plug.ds**
**Purpose:** Fetch movie recommendations based on genre, rating, and year
**API:** The Movie Database (TMDB)
**Inputs:** genre, min_rating, year
**Output:** {title, year, rating, overview, poster_url, tmdb_id}

### 2. **suggest_books_plug.ds**
**Purpose:** Find book recommendations by genre and rating
**API:** Google Books API
**Inputs:** genre, rating
**Output:** {title, author, rating, description, published_date, preview_link}

### 3. **suggest_games_food_plug.ds** (Dual Plug)
**Function A - Games:**
- **API:** RAWG Video Games Database
- **Inputs:** platform, genre
- **Output:** {title, rating, playtime_avg, platforms, genres, image}

**Function B - Food:**
- **API:** Spoonacular
- **Inputs:** cuisine, diet  
- **Output:** {title, image, servings, ready_in_minutes, health_score}

### 4. **event_finder_plug.ds**
**Purpose:** Discover events (movies, concerts, talkshows) near user's location
**API:** Ticketmaster Discovery API
**Inputs:** event_type, city, date_range
**Output:** [{name, date, time, venue, city, booking_url}]

### 5. **ai_vibe_planner_plug.ds** (CREATIVE BROWNIE POINTS)
**Purpose:** AI-powered personalized night plan generator
**API:** OpenAI ChatGPT / Google Gemini
**Inputs:** mood_text, company, duration
**Features:**
- Privacy-preserving mood tagging (converts raw text to safe tags)
- Structured AI prompts for consistent JSON responses
- No raw personal data storage
**Output:** {plan_type, movie, food, activity, reason, privacy_note}

### 6. **helper_utilities.ds**
**Purpose:** Shared utility functions
**Functions:**
- `logPlugActivity()` - Activity logging
- `handlePlugError()` - Error handling with graceful fallbacks
- `calculateDateRange()` - Date manipulation for event filtering
- `tagMood()` - Privacy-preserving mood classification

---

## Bot Flows

### Flow 1: Suggest Something
```
User: "Suggest something"
↓
Bot: "What would you like? Movies / Books / Games / Food / More"
↓
User selects "Movies"
↓
Bot: "What genre? Action, Comedy, Horror, Drama..."
Bot: "Minimum rating? (0-10)"
Bot: "Preferred year?"
↓
[suggest_movie_plug triggers]
↓
Bot displays movie card with poster, rating, overview
Bot: "Want another? [More] [Book Tickets] [Different Genre]"
```

### Flow 2: Book an Event
```
User: "Book an event"
↓
Bot: "What's your city?"
↓
Bot: "Event type? Movies / Concerts / Talkshows / Find Events / Comedy Shows"
↓
User selects "Concerts"
↓
[event_finder_plug triggers]
↓
Bot displays carousel of events: [Concert 1] [Concert 2] [Concert 3]
Bot: "Click to book or [View More] [Change Date]"
↓
User clicks event
↓
Bot: "Opening booking page..." (External link to Ticketmaster)
```

### Flow 3: Surprise Me (AI Vibe Planner)
```
User: "Surprise me"
↓
Bot: "What's your mood? (free text)"
User: "Stressed and need to relax"
↓
Bot: "Who are you with? Alone / Friends / Family"
↓
Bot: "How much time? 30 mins / 1-2 hours / 3+ hours"
↓
[ai_vibe_planner_plug triggers with mood tagging]
Mood: "Stressed" → Tagged as: "Chill & Relax" (privacy preserved)
↓
Bot displays:
"✨ Tonight's Plan:
🎬 Watch: Inception (2010) - Mind-bending masterpiece
🍽️ Eat: Thai green curry - Comfort & healing
🎮 Try: Stardew Valley - Relaxing farming game
💡 Why: Perfect for unwinding and escapism"
↓
Bot: "Like this? [Change movie] [Different food] [Full Details]"
```

---

## OAuth 2.0 Integration

### Supported Third-Party Services

| Service | Auth Type | Purpose | Connection Name |
|---------|-----------|---------|------------------|
| TMDB | API Key | Movies | `tmdb_connection` |
| Google Books | OAuth 2.0 | Books | `google_books_connection` |
| RAWG | API Key | Games | `rawg_connection` |
| Spoonacular | API Key | Food | `spoonacular_connection` |
| Ticketmaster | API Key | Events | `ticketmaster_connection` |
| OpenAI | OAuth 2.0 | AI | `openai_connection` |

### Setup Steps (See docs/OAUTH_SETUP_GUIDE.md for details)

1. Create API accounts for each service
2. In Zoho SalesIQ: Settings → Developers → Connections
3. Add each service as a new Connection
4. Select authentication method (OAuth 2.0 or API Key)
5. Enter credentials and authorize
6. Reference connection in plugs via `connection: "connection_name"`

---

## Deluge Code Snippets

### Example 1: Invoking API via OAuth Connection
```deluge
response = invokeurl(
  [
    url: "https://api.themoviedb.org/3/discover/movie",
    type: "GET",
    parameters: params,
    connection: "tmdb_connection"  // OAuth handled automatically
  ]
);
```

### Example 2: Privacy-Preserving Mood Tagging
```deluge
function tagMood(moodText) {
  text = moodText.toLowerCase();
  if(text.contains("stress") || text.contains("anxious")) {
    return "Chill & Relax";  // Generic tag, not raw input
  }
  return "Surprise Me";
}
```

### Example 3: AI Prompt Construction
```deluge
aiPrompt = "Create a personalized entertainment night plan:\n";
aiPrompt = aiPrompt + "Mood: " + mood_tag + "\n";
aiPrompt = aiPrompt + "Company: " + company + "\n";
aiPrompt = aiPrompt + "Duration: " + duration + "\n";
```

---

## AI & Innovation Features

### 1. Privacy-Preserving Personalization
- Converts free-text mood input to predefined safe tags
- No raw personal data stored in logs
- Session-based recommendations (cleared after interaction)
- Transparent privacy notes in responses

### 2. Multi-API Orchestration
- Single chatbot interface for 5+ external services
- Fallback mechanisms if API fails
- Smart caching for popular requests

### 3. AI Vibe Planner (Brownie Points)
- Uses OpenAI/Gemini for creative recommendations
- Structured JSON responses for reliable parsing
- Considers context (mood + company + time)
- Creates actionable night plans

---

## Installation & Setup

### Prerequisites
- Zoho SalesIQ account (free tier available)
- API keys from third-party services
- Basic understanding of Deluge syntax

### Steps

1. **Clone this repository**
   ```bash
   git clone https://github.com/Priyanka-N2781/enthub-zobot.git
   ```

2. **Set up OAuth Connections** (See `docs/OAUTH_SETUP_GUIDE.md`)
   - Create each API connection in SalesIQ

3. **Copy Deluge Scripts**
   - In SalesIQ: Settings → Developers → Plugs
   - Create new Plugs for each .ds file
   - Copy-paste code from `deluge/` folder

4. **Configure Bot Flows**
   - In SalesIQ: Bots → Codeless Bot
   - Create bot with three main flows:
     - Suggest Something
     - Book an Event
     - Surprise Me
   - Link Plugs to appropriate Action blocks

5. **Test & Deploy**
   - Test flows in SalesIQ chat interface
   - Deploy to your website via widget code
   - Monitor logs for errors

---

## API Usage Examples

### TMDB Movie Search
```
GET https://api.themoviedb.org/3/discover/movie?api_key=KEY&with_genres=28&sort_by=popularity.desc
Response: {results: [{title, release_date, vote_average, poster_path, overview}]}
```

### Ticketmaster Event Search
```
GET https://app.ticketmaster.com/discovery/v2/events?city=NYC&classificationName=music
Response: {_embedded: {events: [{name, dates, _embedded: {venues}}]}}
```

See `docs/API_INTEGRATION_CONTRACTS.md` for complete contract specs.

---

## Error Handling

All plugs include try-catch blocks with:
- User-friendly error messages
- Graceful fallbacks
- Activity logging for debugging
- No sensitive data in error logs

---

## Performance Considerations

- Responses cached for 1 hour
- Rate limiting respected per API
- Max 10 results per recommendation
- Timeout: 30 seconds per API call

---

## Future Enhancements

- [ ] User preference persistence (with consent)
- [ ] Social sharing (share recommendations)
- [ ] Wishlist feature (save for later)
- [ ] Multi-language support
- [ ] Advanced analytics dashboard
- [ ] Mobile app integration
- [ ] Push notifications for new releases

---

## Contributing

This is a Zoho Cliqtrix internship project. For improvements:

1. Fork the repository
2. Create a feature branch
3. Commit changes
4. Push and create a Pull Request

---

## License

MIT License - See LICENSE file for details

---

## Support & Documentation

- 📖 **Full Docs:** See `docs/` folder
- 🔗 **Bot Flows:** `docs/BOT_FLOW_DIAGRAM.md`
- 🔑 **OAuth Setup:** `docs/OAUTH_SETUP_GUIDE.md`
- 🔌 **Plug Config:** `docs/PLUGS_CONFIGURATION.md`
- 🏗️ **Architecture:** `ARCHITECTURE.md`

---

## Contact

**Developer:** Zoho Internship Student  
**Email:** priyanka.n2781@zoho.com  
**GitHub:** Priyanka-N2781  

---

**EntHub Zobot** - Making entertainment personalized, private, and intelligent! 🎉
