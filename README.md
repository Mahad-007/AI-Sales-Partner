# AI Sales Partner for Freelancers

An AI-powered Chrome extension that acts as your intelligent sales copilot throughout the complete freelance deal lifecycle -- from identifying opportunities to closing deals and maintaining long-term client relationships.

The AI Sales Partner behaves like a **20-year experienced freelancer**, helping you craft better proposals, prepare for client meetings, analyze clients, and continuously improve your sales strategy. It integrates freelancer profiles, client history, proposal data, and sales insights to ensure every proposal and interaction is aligned with your positioning, skills, and past work.

Built as a Chrome side panel that lives in your browser, it automatically detects what Upwork page you're on and provides contextual AI-powered tools.

## Objectives

- Improve proposal quality and personalization
- Increase freelance deal conversion rates
- Maintain a centralized knowledge base of clients, deals, and strategies
- Reduce time spent writing proposals and preparing meetings
- Align proposals with freelancer profiles, experience, and positioning
- Achieve a 25-35% increase in sales performance over time

## Key Features

### 1. Freelancer Profile Synchronization

Maintains structured information about freelancers including skills, portfolio projects, past clients, positioning, and platform profiles. Ensures proposals and communication are always aligned with your real profile and capabilities.

- **GitHub Integration** -- Fetches your GitHub profile, repos (sorted by stars), languages, and README content
- **Website Scraping** -- Extracts text from your portfolio website
- **AI Profile Suggestions** -- Generates optimized Upwork titles, bio, skills (with proficiency levels), and rate recommendations
- **Completeness Assessment** -- Scores your portfolio data with strengths, gaps, and recommendations

### 2. Proposal Writing Assistant (V1 Priority)

Generates full freelance proposals based on job descriptions, client profile analysis, freelancer portfolio, and project requirements. Requests missing context when needed to ensure higher-quality applications.

- **AI-Generated Cover Letters** -- Creates personalized, Upwork-optimized proposals tailored to each job posting
- **Portfolio-Aware** -- Automatically references your GitHub projects and portfolio highlights
- **Edit & Copy** -- Inline editing with one-click copy to clipboard
- Follows Upwork best practices: hook opening, specific experience, proposed approach, and call to action

### 3. Client Research & Context Analysis

Analyzes client hiring history, budget patterns, project goals, and industry context for better proposal positioning and strategic responses.

- **Smart Job Scoring** -- Automatically scores jobs 0-100 based on how well they match your skills, rate, and experience
- **Sort by Fit** -- Reorder job listings by match score to focus on the best opportunities first
- **Auto-Score** -- Optionally score jobs automatically as soon as a search page loads
- Displays budget, job type, experience level, client country, rating, and proposal count

### 4. Meeting Preparation Assistant

Before client meetings, generates client briefing reports, meeting strategies, recommended questions, potential objections with responses, and deal-closing suggestions.

- **Talking Points** -- AI-generated checklist of key points to cover in client calls
- **Likely Questions** -- Predicts what the client will ask based on the job and conversation history
- **Preparation Tips** -- Actionable advice to make a great impression
- **Manual Context** -- Add context manually if no active conversation is detected

### 5. Profile Analysis & Improvement

Scans and scores freelancer profiles, then provides job-targeted optimization recommendations.

- **Profile Scanner** -- Scores your profile across Title, Bio, Skills, Portfolio, Rate, and Completeness
- **Actionable Suggestions** -- Section-by-section improvement recommendations
- **Job-Targeted Optimization** -- Compares your profile against a specific job posting and suggests targeted improvements
- **Keyword & Skill Gaps** -- Identifies missing keywords and skills to add

### 6. Chat Assistant & Smart Replies

Reads conversation threads and generates contextually relevant replies in multiple tones.

- **Smart Reply Suggestions** -- Generates 3 reply options: Brief, Detailed, and Friendly
- **Conversation-Aware** -- Reads the last messages in the thread for context
- **One-Click Copy** -- Copy any suggestion directly to your clipboard

### Settings

- **Multi-Model Support** -- Choose from Claude Opus 4.6, Sonnet 4.6/4.5, Haiku 4.5, GPT-4o, GPT-4o Mini, or Gemini 2.0 Flash
- **API Key Validation** -- Test your OpenRouter key before saving
- **Auto-Score Toggle** -- Enable/disable automatic job scoring on page load
- **Auto-Switch Tabs** -- Automatically switch to the relevant tab based on the current Upwork page

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **Framework** | React 18 with TypeScript |
| **State Management** | Zustand |
| **Styling** | Tailwind CSS 3 (dark mode, custom neo-brutalist design system) |
| **Build Tool** | Vite 5 with multi-entry rollup config |
| **AI Backend** | OpenRouter API (multi-model router) |
| **Extension API** | Chrome Manifest V3 (service worker, side panel, content scripts) |
| **Data Fetching** | GitHub REST API v3, raw HTML scraping |

## Architecture

```
src/
  background/
    service-worker.ts    # Message router, AI orchestration, storage management
    ai-client.ts         # OpenRouter API client with rate limiting
    portfolio-fetcher.ts # GitHub API + website scraper
  content/
    index.ts             # Page detection, scraping dispatcher, SPA observer
    page-observer.ts     # MutationObserver + History API intercepts for SPA nav
    scrapers/
      profile-scraper.ts # Upwork profile data extraction
      job-scraper.ts     # Job search + job detail page extraction
      chat-scraper.ts    # Message thread extraction + live observer
      meeting-scraper.ts # Meeting/call detection from messages
  shared/
    types.ts             # TypeScript interfaces for all data models
    constants.ts         # API URLs, models, rate limits, storage keys
    messages.ts          # Typed message protocol (content <-> service worker <-> panel)
    prompts.ts           # All AI prompt templates
  sidepanel/
    index.tsx            # React entry point
    App.tsx              # Tab router, settings gate, page-aware navigation
    store.ts             # Zustand store (UI state, settings, page data, portfolio)
    styles.css           # Tailwind + custom neo-brutalist component classes
    hooks/
      useAI.ts           # Generic message sender with loading/error state
      usePageData.ts     # Session storage listener for scraped page data
      useStorage.ts      # Chrome storage hooks (local + session)
    components/
      Settings.tsx       # API key, model picker, toggle switches
      ProfileScanner.tsx # Profile analysis UI with section scores
      JobHunter.tsx      # Job list with scoring, sorting, auto-score
      ProposalGenerator.tsx # Proposal generation, editing, copy
      ProfileImprover.tsx   # Job-targeted profile improvement tips
      ChatAssistant.tsx     # Reply suggestions with tone options
      MeetingPrep.tsx       # Meeting preparation checklist
      PortfolioBuilder.tsx  # GitHub/website fetch + AI suggestions + apply
  popup/
    index.html           # Compact popup with status indicator
    popup.ts             # API key status check, side panel opener
  assets/
    icons/               # Extension icons (16, 48, 128px in PNG + SVG)
```

### Data Flow

1. **Content Script** runs on all `upwork.com` pages. It detects the page type (profile, job search, job detail, messages) and scrapes structured data from the DOM.
2. **Page Observer** watches for SPA navigation by intercepting `pushState`/`replaceState` and using `MutationObserver`, triggering re-scrapes on URL changes.
3. **Service Worker** receives scraped data and stores it in `chrome.storage.session`. It also handles all AI requests by building prompts, calling OpenRouter, parsing JSON responses, and caching results.
4. **Side Panel** reads page data from session storage via Zustand, automatically switching tabs based on page type. Components send typed messages to the service worker for AI operations.
5. **Popup** provides a quick status check (API key configured?) and a button to open the side panel.

### Message Protocol

All communication uses Chrome's `runtime.sendMessage` with a typed union:

- **Content -> Service Worker**: `PAGE_DATA`, `PAGE_CHANGED`
- **Panel -> Service Worker**: `AI_SCORE_JOBS`, `AI_GENERATE_PROPOSAL`, `AI_ANALYZE_PROFILE`, `AI_IMPROVE_PROFILE`, `AI_CHAT_REPLIES`, `AI_MEETING_PREP`, `FETCH_PORTFOLIO`, `AI_ANALYZE_PORTFOLIO`, `TEST_API_KEY`, `GET_SETTINGS`, `SAVE_SETTINGS`, `GET_PROFILE`, `SAVE_PROFILE`, etc.
- **Service Worker -> Panel**: `{ success: true, data } | { success: false, error }`

## Getting Started

### Prerequisites

- Node.js 18+
- An [OpenRouter API key](https://openrouter.ai/keys)

### Installation

```bash
# Clone the repository
git clone https://github.com/Mahad-007/AI-Sales-Partner.git
cd AI-Sales-Partner

# Install dependencies
npm install

# Build the extension
npm run build
```

### Load in Chrome

1. Open `chrome://extensions/`
2. Enable **Developer mode** (toggle in top-right)
3. Click **Load unpacked**
4. Select the `dist/` folder
5. Pin the AI Sales Partner extension and click it to open the side panel
6. Enter your OpenRouter API key in Settings

### Development

```bash
# Watch mode -- rebuilds on file changes
npm run dev
```

After rebuilding, go to `chrome://extensions/` and click the refresh icon on the extension card.

## Configuration

| Setting | Description | Default |
|---------|-------------|---------|
| **OpenRouter API Key** | Required. Get one at [openrouter.ai/keys](https://openrouter.ai/keys) | -- |
| **AI Model** | Choose which LLM powers the extension | Claude Opus 4.6 |
| **Auto-Score Jobs** | Automatically score jobs when a search page loads | On |
| **Auto-Switch Tabs** | Switch to the relevant tab based on the current Upwork page | On |

## Rate Limiting

The extension enforces a client-side rate limit of **10 requests per 60 seconds** to avoid hitting OpenRouter's limits. If exceeded, you'll see a "Rate limit exceeded" message -- wait a moment and try again.

## Supported Upwork Pages

| Page | URL Pattern | Features Available |
|------|------------|-------------------|
| Profile | `/freelancers/~...`, `/fl/...` | Profile Scanner, Portfolio Builder |
| Job Search | `/nx/search/jobs`, `/ab/jobs/search` | Job Hunter (scoring + sorting) |
| Job Detail | `/jobs/~...`, `/nx/proposals/job/~...` | Proposal Generator, Profile Improver |
| Messages | `/ab/messages`, `/nx/messages` | Chat Assistant, Meeting Prep |

## Long-Term Vision

Over time, the AI Sales Partner may evolve into:

- An **AI Sales Operating System for Freelancers**
- An **open-source freelancer sales assistant**
- A **SaaS product for freelancers and agencies**

## License

This project is unlicensed. All rights reserved.
