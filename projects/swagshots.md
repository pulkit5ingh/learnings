### swagshots (swagshots.com)

**How I'd explain it to a recruiter**

"SwagShots is a short-video social platform, live at swagshots.com. I led the architecture of the backend, which runs as 20–30 microservices built with Node.js, NestJS and TypeScript. Users upload and watch short videos, go live with video and audio, like, comment, share, chat, send gifts and get push notifications, and all of it works in real time."

**What the platform does**

- Short-video feed with upload, playback and CDN-optimized delivery.
- Video streaming: videos are processed with FFmpeg into HLS and stored and served from Cloudflare R2.
- Live video and audio using LiveKit.
- Social engagement: likes, comments, shares, chats and push notifications.
- Nudity detection with NudeNet, so unsafe content is caught during moderation.
- Login with Google and WhatsApp OTP.
- User gift and coin system: users buy coins through a payment gateway and send gifts.
- Camera filters, including skin smoothening.
- Multiple microservices connected through BullMQ and Redis, so work moves between services as queued jobs.

**How the video pipeline works**

- A user uploads a video, and a job is queued in BullMQ.
- FFmpeg processes it into HLS streams, and the output is stored on R2 and delivered through the CDN.
- Nudity detection with NudeNet runs on the content as part of moderation.
- The pipeline handled nearly 1,000 videos a day with near real-time turnaround.

**Live streaming**

- Live video and audio run on LiveKit.
- We replaced our legacy streaming infrastructure with LiveKit, which cut infrastructure costs by 90% and improved scalability and latency.

**What I built and the tech behind it**

- 20–30 production microservices in Node.js, NestJS and TypeScript, and I drove the system design decisions.
- Multiple services connected with BullMQ and Redis: one service adds a job to a queue and another service picks it up, so services stay loosely coupled and can scale on their own.
- Video processing with FFmpeg and BullMQ, with storage and delivery on Cloudflare R2 and a CDN.
- Live video and audio with LiveKit.
- Real-time engagement: likes, comments, shares and chats.
- Push notifications.
- NudeNet for nudity detection.
- Authentication with Google sign-in and WhatsApp OTP.
- Gift and coin system for user-to-user gifting.
- Payment gateway integration so users can buy coins.
- Camera filters and skin smoothening.
- PostgreSQL for wallet, coin transactions and gifting; MongoDB for video/post metadata, comments, chat and notifications.

**Database design (PostgreSQL vs MongoDB)**

- PostgreSQL for money and consistency: coin wallet balances, coin purchase transactions from the payment gateway, and gift transactions (debit sender, credit receiver, atomically). Also user accounts, auth and roles, where relational integrity and constraints matter.
- MongoDB for content and high write volume: video and post metadata (caption, tags, filters used, processing status), comments, chat messages, notifications, and like/view counters, all read as documents with no joins needed.
- Rule of thumb: PostgreSQL where correctness of money and transactions across tables matters; MongoDB where the data shape varies or write volume is high and a single document read is enough.

**Impact**

- Nearly 1,000 videos processed a day with near real-time turnaround.
- 90% lower streaming infrastructure cost after moving to LiveKit.
- 70–80% faster API responses through caching, query optimization and async queue-based processing.
- 100% production uptime across the services.

**Likely follow-up questions**

- Why microservices for this product? Video processing, live streaming, chat, notifications and payments have very different load and scaling needs, so each one scales on its own.
- How does the video pipeline work? Upload, queue in BullMQ, FFmpeg to HLS, store on R2, deliver through the CDN.
- Why LiveKit? It replaced our legacy streaming setup, cut cost by 90% and gave us better scalability and latency.
- How does nudity detection fit in? NudeNet checks uploaded content during moderation so unsafe videos are flagged.
- How did your services talk to each other? Through BullMQ queues backed by Redis: a service publishes a job and the service that owns that work consumes it, so a slow or failing service doesn't block the others.
- How did you build the coin and gift system? Coins are a balance per user, and a gift is a transaction that moves coins from the sender to the receiver.
- How do users buy coins? They pay through the payment gateway, and coins are credited to their balance only after the payment is confirmed.
- How does WhatsApp OTP login work? The user gets a one-time code on WhatsApp, and we verify it before issuing a session.
- Why two databases instead of one? PostgreSQL gives ACID transactions for money (wallet, gifts, coin purchases), while MongoDB fits content that's high-volume and flexible in shape (videos, comments, chat, notifications) without forcing a rigid schema on it.
