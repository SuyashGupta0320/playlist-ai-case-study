# Case Study: Playlist AI – AI-Powered Spotify Playlist Generator

## 🚀 Project Overview

**Playlist AI** is a full-stack web application that transforms natural language prompts into curated Spotify playlists using Generative AI. It solves the problem of "playlist paralysis" by allowing users to describe a specific vibe (e.g., *"Upbeat 2010s workout music that feels like a summer festival"*) and instantly receive a context-aware playlist.

Unlike standard playlist generators that rely on simple tag matching, Playlist AI uses **Google's Gemini AI** to understand cultural context, sonic cohesion, and lyrical themes. The platform features a unique **"Account Rotation System"** to bypass API limitations, allowing users to export playlists to Spotify without initially connecting their own accounts.

---

## 🛠️ Tech Stack

| Component | Technologies |
|-----------|--------------|
| **Frontend** | Next.js (React), TypeScript, Tailwind CSS, Framer Motion |
| **Backend** | Python (Flask), SQLAlchemy, Spotipy |
| **Database** | SQLite (Dev), PostgreSQL (Prod) |
| **AI Model** | Google Gemini (via Google AI Studio) |
| **Auth** | Google OAuth, Spotify OAuth |
| **Payments** | Stripe (Webhooks, Checkout Sessions) |
| **Infrastructure** | Docker, Nginx, Cloudflare |

---

## 💡 Key Technical Challenges & Solutions

### 1. The "250k MAU" Wall (Spotify Account Rotation)
**The Problem:** Spotify's API restricts apps from creating playlists on a user's account until the app is approved for "Extended Quota Mode," which requires 250,000 Monthly Active Users. This created a "chicken and egg" problem for a new app.

**The Solution:** I engineered a **Spotify Account Rotation System**.
- **Architecture:** I created a pool of 25 pre-authenticated "host" Spotify accounts managed by the backend.
- **Logic:** When a user clicks "Export," the system selects the least-recently-used (LRU) healthy account from the pool.
- **Process:** The backend acts as a proxy, creating the playlist on the host account and returning a public URL to the user.
- **Result:** Users can "export" playlists immediately without logging in to Spotify, and the system scales horizontally by simply adding more host accounts.

### 2. Context-Aware Playlist Generation
**The Problem:** Simple keyword matching (e.g., searching "sad songs") often results in disjointed playlists that lack flow.

**The Solution:** Leveraged **Google's Gemini Pro** with advanced prompt engineering.
- **Prompt Strategy:** The system constructs a complex prompt that instructs the AI to act as a professional DJ. It enforces constraints on genre, era, and tempo while requiring a JSON output structure.
- **Validation:** The backend validates the AI's output against the Spotify API to ensure every track actually exists, automatically handling hallucinations by searching for the closest match or removing invalid tracks.

### 3. Hybrid Monetization & Rate Limiting
**The Problem:** AI API calls and server costs are expensive, requiring a sustainable business model without alienating free users.

**The Solution:** Implemented a multi-tiered credit system using **Flask-Limiter** and **Stripe**.
- **Free Tier:** Users get 3 credits (playlist generations) which regenerate every 6 hours. This is enforced via a custom `check_rate_limits` function that tracks device fingerprints and IP addresses to prevent abuse.
- **Paid Tier:** Integrated Stripe webhooks to handle one-time credit purchases and recurring subscriptions.
- **State Management:** The frontend reacts instantly to credit changes using optimistic UI updates while the backend verifies the transaction ledger.

---

## 📸 Core Features

### 🎵 "Vibe-to-Playlist" Engine
Users input natural language prompts. The system analyzes the sentiment and musical requirements to generate a list of 20-100 songs that flow seamlessly together.

### 🔄 Smart Export
Export to Spotify, YouTube Music (via search links), or JSON. The export process handles token refreshing and API errors transparently.

### 🛡️ Anti-Abuse System
A sophisticated rate-limiting engine that fingerprint devices and IPs to prevent users from creating infinite accounts to bypass credit limits.

---

## 🧠 Key Learnings

1.  **3rd Party API Dependency:** Building on top of Spotify's ecosystem required robust error handling. I learned to implement exponential backoff and "circuit breaker" patterns to handle rate limits and downtime gracefully.
2.  **State Synchronization:** Keeping the user's credit balance synced across the Next.js client (client-side) and Flask database (server-side) taught me the importance of webhooks and optimistic UI updates for a snappy user experience.
3.  **Security First:** Handling payments and auth tokens required strict adherence to security best practices, including encrypting tokens at rest and validating all webhook signatures.

---

*   **Deep Analysis:** Using audio analysis features to visualize the "energy" curve of a playlist.
*   **Mobile App:** Porting the Next.js frontend to React Native.
