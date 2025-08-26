**PROBLEM STATEMENT**

A scalable, cloud-native video app with user-controlled, explainable ranking and strict roles (creator vs consumer), deployed on Azure free tiers.

**SCOPE**
MVP Entities: User (role: creator|consumer), Video (blob url + metadata), Interaction (like|comment|rating), FeedScore (computed fields). Keep it lean as fuck.

MVP Features:

1. Creator upload + metadata.

2. Consumer signup/login.

3. Dashboard “Latest” + switchable feed mode.

4. Search by title/genre/age rating.

5. Comments/ratings.

6. “Why this video” explainer per card.

7. Basic moderation toggle (hide 18+ unless opted-in).


Out of scope for MVP: Live streaming, DM, monetization, duet/remix.


**Risk & Mitigation:**

Transcoding cost → start with client-side format hints (.mp4 H.264/AAC), defer server transcoding; add Azure queue + Function later if free budget allows.

COSMOS RU burn → lean indexes, denormalize hot fields, cache reads at the API layer.

Auth complexity → start with Static Web Apps built-in auth; enforce creator role in DB, not in identity provider on day one.




[React/Vite SPA on Azure Static Web Apps (Free, global)] 
        │  (HTTPS, built-in auth)
        
        ▼
        
[Azure Functions API (Node/TypeScript, Consumption)]
   │            │
   │            ├── Read/Write → [Azure Cosmos DB (NoSQL, serverless/free tier)]
   │            └── SAS token mint → [Azure Blob Storage (video + thumb)]
   
        ▼
        
[Client streams videos via Blob Storage public endpoint/CDN edge from SWA]


**Entity Sketch:**

1. **User:** id, email, displayName, role (creator|consumer), createdAt.

2. **Video:** id, ownerId, title, publisher, producer, genre, ageRating, blobUrl, thumbUrl, duration, createdAt, stats {views, likes, avgRating}.

3. **Comment**: id, videoId, userId, text, createdAt.

4. **Rating:** id, videoId, userId, stars (1–5), createdAt.

5. **FeedScore** (materialized or computed): recency, completionRate, likeRate, creatorReliability, finalScore, mode (“trending|personalized”), reasons[].

**Feed Scoring v0 (formula):**
finalScore = w1*Recency + w2*Completion + w3*LikeRate + w4*AvgRating + w5*CreatorReliability
Expose the weights in config so you can show the user the math.
