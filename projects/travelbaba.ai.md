Hello, my name is Pulkit Singh, and I’m a Senior Backend & AI Integration Engineer with 5+ years of experience building production backend systems and AI-powered applications. I work mainly with Node.js, TypeScript, NestJS, Python and FastAPI, and I hold a Bachelor's degree in Computer Applications (BCA).

### travelbaba.ai (travelbaba.ai)

**How I'd explain it to a recruiter**

"One of the projects I'm most proud of is TravelBaba.ai. It's an AI-powered travel planning platform, live at travelbaba.ai, and it serves 10K+ users. Travel agents use it to build tour itineraries and send quotations to their customers quickly, and we use OpenAI to generate itineraries and give contextual recommendations instead of agents building everything by hand."

**The problem it solves**

- Travel agents usually create itineraries and price quotations manually, which is slow and inconsistent.
- The platform lets an agent go from a customer enquiry to a priced quotation on the go, with AI helping with the itinerary.

**What the platform does**

- Multi-tenant, multi-portal platform with three kinds of users: admin, admin sales and agents.
- Admin and admin sales create tour packages: pictures, night and day count, pickup and drop location, activities, sightseeing, inclusions and exclusions, day-wise itinerary (place with night count) and transportation vehicles.
- Agents create a quotation from a package by adding customer details and their own commission per customer.
- The total quotation price is sent to the customer over the WhatsApp API.
- Agents pay through a wallet and a subscription model (monthly, quarterly, half-yearly, yearly), and payments go through the PhonePe payment gateway.
- Quotations and itineraries are generated as PDFs using Puppeteer, so agents can share a clean, branded document with their customers.
- Flight booking is integrated through Skyscanner, so agents can add flights to a trip alongside hotels and transport.

**The AI part**

- OpenAI APIs generate itineraries.
- Prompt orchestration is built around the package and customer context, so the output is relevant to that trip.
- Contextual recommendation workflows suggest options to the agent.

**What I built and the tech behind it**

- Backend in NestJS, TypeScript and MongoDB.
- Authentication with JSON Web Tokens, plus OTP verification by email using Nodemailer.
- A dynamic RBAC system, so roles and module access can be changed without code changes.
- API caching for frequently requested data such as packages and itineraries, which cuts database load and improves response times.
- Payment gateway integration (PhonePe) for wallet top-ups and subscription payments.
- Twilio API integration for messaging and notifications.
- Nodemailer for transactional emails, including OTP verification.
- Puppeteer for server-side PDF generation of quotations and itineraries.
- Skyscanner integration for flight search and booking.
- OpenAI integration for itinerary generation, prompt orchestration and recommendations.

**Impact**

- 10K+ users on the platform.
- Helped agents move from manual quotation work to a faster, AI-assisted flow.

**Likely follow-up questions**

- Why NestJS and MongoDB? Modular structure suited the many roles and modules, and packages and itineraries are document-shaped data.
- How did you design RBAC? Roles and module permissions are stored as data and checked at runtime, so new roles need no deployment.
- How did you handle AI output quality? Prompt orchestration with the package and customer context as input.
- Why did you cache APIs, and how did you keep the cache correct? Package and itinerary data is read far more often than it changes, so it is cached and invalidated when an admin edits a package.
- How does the payment flow work? The agent pays through PhonePe, and the wallet or subscription is only updated after the payment is confirmed.
- How did you generate PDFs? Puppeteer renders an HTML template of the quotation on the server and exports it as a PDF.
- How did you integrate flight booking? Flights come from the Skyscanner API and are added to the agent's trip and quotation.

### ordercoro

- order coro a hyper local market place.
- why we faced this issue and how we solved it.
- registered merchants
- inventory setup (bulk upload using messaging queue BullMQ)
- tech parks in mumbai Neptune Magnet Mall,
- employee registration using employee id and manually verification
- schedule delivery by merchants

### robofood

- robofood
- clients with multiple branch can create
- inventory and restaurant setup
- table reservation
- digital menu setup
- staff management
- delivery area and management
