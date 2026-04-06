## Questions: Can you explain your current role and responsibilities?

change

## Answer:

I work as a Backend Developer primarily using Node.js and NestJS. My responsibilities include designing and building RESTful APIs, integrating third-party services, managing microservices communication, writing unit and integration tests, and deploying services on AWS. I also participate in code reviews, system design discussions, and collaborate closely with frontend and DevOps teams.

**Example:** In my last project, I was responsible for building the ride-tracking service — from designing the database schema in PostgreSQL, writing APIs for driver location updates, to setting up SQS queues for async notification delivery.

---

## Questions: What technologies are you currently working on?

## Answer:

I'm currently working with:

- **Backend:** Node.js, NestJS, TypeScript
- **Databases:** PostgreSQL, Redis (caching), MongoDB
- **Messaging:** AWS SQS, RabbitMQ
- **Cloud:** AWS (EC2, S3, SQS, SNS, Lambda)
- **DevOps:** Docker, Kubernetes, GitHub Actions
- **Monitoring:** CloudWatch, Datadog

---

## Questions: How many services were there in your project?

## Answer:

Our project had around 6–8 microservices. Each service owned a specific domain:

1. **Auth Service** – JWT-based login, registration, token refresh
2. **User Service** – Profile management
3. **Ride Service** – Booking and ride lifecycle
4. **Tracking Service** – Real-time driver location (WebSocket + Redis)
5. **Notification Service** – Push notifications, SMS, emails
6. **Payment Service** – Payment processing and invoicing
7. **Admin Service** – Dashboard, reporting, analytics
8. **API Gateway** – Single entry point, routing, rate limiting

---

## Questions: What features were handled in the admin panel?

## Answer:

The admin panel handled:

- User and driver management (CRUD, ban/unban)
- Ride monitoring — real-time and historical
- Revenue and analytics dashboards
- Dispute and support ticket management
- Notification broadcasts (push, SMS, email)
- Configuration management (pricing, surge rules)
- Role-based access control (RBAC) for admin staff

**Example:** Admins could search rides by date range, export reports as CSV, and manually assign rides if a driver declined.

---

## Questions: How did you manage live tracking?

## Answer:

Live tracking was managed using **WebSockets** (Socket.IO) and **Redis Pub/Sub**:

1. Driver app sends GPS coordinates every 3–5 seconds via WebSocket.
2. Server stores the latest position in Redis (fast read/write).
3. Rider's app subscribes to a room (ride ID) and receives location updates in real time.
4. Historical path is persisted asynchronously to PostgreSQL.

**Example:**

```typescript
// Driver emits location
socket.emit('update-location', { rideId: '123', lat: 28.61, lng: 77.2 });

// Server stores in Redis and broadcasts to ride room
await redisClient.set(`driver:${rideId}`, JSON.stringify({ lat, lng }));
io.to(rideId).emit('driver-location', { lat, lng });
```

---

## Questions: What is latitude and longitude?

## Answer:

- **Latitude** measures how far north or south a point is from the equator. Range: -90° (South Pole) to +90° (North Pole).
- **Longitude** measures how far east or west a point is from the prime meridian (Greenwich). Range: -180° to +180°.

**Example:** New Delhi is approximately at **28.6139° N, 77.2090° E**. Together they uniquely identify any location on Earth.

In code, you store them as `DECIMAL(9,6)` in SQL or as a GeoJSON point in MongoDB.

---

## Questions: How many parallel requests were handled by your application?

## Answer:

Our application handled around **500–1000 concurrent requests** per service instance during peak hours. We scaled horizontally using Kubernetes — running 3–5 pods per service with a load balancer in front. Node.js's non-blocking event loop allowed each instance to handle many simultaneous connections efficiently without spawning a new thread per request.

---

## Questions: Is Node.js single‑threaded or multi‑threaded?

## Answer:

Node.js runs **JavaScript on a single thread** (the main event loop), but it is **not purely single-threaded**. Under the hood:

- The **event loop** runs on one thread and handles JS execution.
- **libuv** (the C++ layer) uses a **thread pool** (default 4 threads) for heavy I/O operations like file system reads, DNS lookups, and crypto.
- `worker_threads` allow true multi-threading for CPU-intensive tasks.

**Example:** When you call `fs.readFile()`, Node hands it off to the libuv thread pool, frees the main thread to handle other requests, and calls your callback when done.

---

## Questions: How do you manage multiple concurrent requests in Node.js?

## Answer:

Node.js manages concurrency through its **non-blocking event loop**:

1. Each incoming request is registered as an event.
2. I/O operations (DB queries, HTTP calls) are offloaded and don't block the main thread.
3. Callbacks/Promises resolve when I/O completes, and the event loop picks them up.

**Example:**

```javascript
// Both DB calls run concurrently, not sequentially
const [users, orders] = await Promise.all([
  db.query('SELECT * FROM users'),
  db.query('SELECT * FROM orders'),
]);
```

Using `Promise.all` cuts wait time roughly in half compared to awaiting them one by one.

---

## Questions: How many microservices were there and what were they?

## Answer:

See the answer to "How many services were there in your project?" above — we had 6–8 services covering Auth, User, Ride, Tracking, Notification, Payment, Admin, and API Gateway. Each service had its own database (database-per-service pattern) and communicated via REST for synchronous calls and SQS/RabbitMQ for async messaging.

---

## Questions: How do you manage messaging between services?

## Answer:

We used two patterns:

1. **Synchronous (REST/HTTP):** For real-time, blocking calls — e.g., Auth Service validates a token for Ride Service.
2. **Asynchronous (Message Queues):** For non-blocking operations — e.g., when a ride ends, Ride Service publishes an event to SQS; Notification Service and Payment Service consume it independently.

**Example with SQS:**

```typescript
// Producer (Ride Service)
await sqs
  .sendMessage({
    QueueUrl: process.env.RIDE_COMPLETED_QUEUE,
    MessageBody: JSON.stringify({ rideId, userId, amount }),
  })
  .promise();

// Consumer (Notification Service)
// Polls the queue and sends push notification
```

---

## Questions: What is Amazon SQS?

## Answer:

**Amazon Simple Queue Service (SQS)** is a fully managed message queuing service that decouples producers and consumers. Key features:

- **Standard Queue:** At-least-once delivery, best-effort ordering.
- **FIFO Queue:** Exactly-once processing, strict ordering.
- **Visibility Timeout:** Message is hidden from other consumers while being processed.
- **Dead Letter Queue (DLQ):** Failed messages are moved here after max retries.

**Example use case:** After a ride completes, Ride Service pushes a message to SQS. Notification Service picks it up and sends a receipt — completely decoupled. If Notification Service is down, the message waits in the queue.

---

## Questions: How do you enable notifications using SQS?

## Answer:

The typical pattern is **SNS → SQS → Consumer**:

1. **SNS (Simple Notification Service)** is the publisher — it fans out to multiple SQS queues.
2. Each microservice subscribes to its own SQS queue.
3. The consumer polls the queue and triggers the notification (push, email, SMS).

**Example:**

```
Ride Completed Event
       ↓
    SNS Topic
    /       \
SQS (Email) SQS (Push Notification)
    ↓               ↓
Email Lambda   FCM/APNs Push
```

```typescript
// Send to SNS
await sns
  .publish({
    TopicArn: process.env.RIDE_TOPIC_ARN,
    Message: JSON.stringify({ rideId, userId }),
  })
  .promise();
```

---

## Questions: When would you use SQS versus RabbitMQ?

## Answer:

|              | SQS                 | RabbitMQ                       |
| ------------ | ------------------- | ------------------------------ |
| **Hosting**  | Fully managed (AWS) | Self-hosted or CloudAMQP       |
| **Protocol** | HTTP/HTTPS          | AMQP                           |
| **Routing**  | Basic               | Flexible (exchanges, bindings) |
| **Ordering** | FIFO queue only     | Per-queue ordering             |
| **Cost**     | Pay-per-use         | Infrastructure cost            |

**Use SQS when:** You're on AWS, want zero ops overhead, and need simple point-to-point or fan-out messaging.

**Use RabbitMQ when:** You need complex routing (topic/direct/fanout exchanges), low-latency delivery, or you want more control over message flow.

---

## Questions: How do you trace or track requests across the system?

## Answer:

We use **distributed tracing** with a **Correlation ID** (also called Trace ID):

1. API Gateway generates a unique `correlationId` (UUID) for every incoming request.
2. It's passed as a header (`X-Correlation-ID`) to all downstream services.
3. Every service logs this ID with each log entry.
4. Tools like **Datadog APM** or **AWS X-Ray** stitch all spans together visually.

**Example (NestJS middleware):**

```typescript
@Injectable()
export class CorrelationIdMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    req.headers['x-correlation-id'] =
      req.headers['x-correlation-id'] || uuidv4();
    next();
  }
}
```

---

## Questions: What tools are used for monitoring?

## Answer:

- **AWS CloudWatch** — logs, metrics, alarms for AWS resources.
- **Datadog** — APM, distributed tracing, dashboards, alerting.
- **Prometheus + Grafana** — metrics scraping and visualization (self-hosted).
- **Sentry** — real-time error tracking with stack traces.
- **PagerDuty** — on-call alerting for production incidents.

**Example:** We set a CloudWatch alarm to trigger if the SQS queue depth exceeds 500 messages — indicating consumers are falling behind — and alert the on-call engineer via PagerDuty.

---

## Questions: What cloud providers do you know?

## Answer:

- **AWS (primary):** EC2, S3, RDS, SQS, SNS, Lambda, EKS, CloudWatch, Route53, ALB.
- **GCP (familiar):** Cloud Run, Cloud Pub/Sub, BigQuery, GKE.
- **Azure (basic):** App Service, Azure Functions, Azure Service Bus.

Most hands-on experience is with AWS for production workloads.

---

## Questions: Have you worked with containerization?

## Answer:

Yes. I've worked with **Docker** for containerizing Node.js applications and **Kubernetes** for orchestration.

**Example Dockerfile for a NestJS app:**

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY dist/ ./dist/
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

We used multi-stage builds to keep image sizes small and ran containers in EKS (Elastic Kubernetes Service) on AWS.

---

## Questions: What is Kubernetes and why is it used?

## Answer:

**Kubernetes (K8s)** is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications.

**Why use it:**

- Auto-scaling based on CPU/memory load.
- Self-healing — restarts crashed containers.
- Rolling deployments with zero downtime.
- Service discovery and load balancing built in.

**Example:** We defined a Deployment with 3 replicas for the Ride Service. When CPU exceeded 70%, the Horizontal Pod Autoscaler (HPA) automatically scaled to 6 pods, and scaled back down when load dropped.

---

## Questions: What is a Pod in Kubernetes?

## Answer:

A **Pod** is the smallest deployable unit in Kubernetes. It wraps one or more containers that share the same network namespace and storage volumes.

- Typically **one container per pod** (best practice).
- Pods are ephemeral — if they die, a new one is created.
- Pods in the same node communicate via `localhost`.

**Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ride-service
spec:
  containers:
    - name: ride-service
      image: myrepo/ride-service:latest
      ports:
        - containerPort: 3000
```

---

## Questions: How do services communicate with each other in a microservices architecture?

## Answer:

Two main patterns:

1. **Synchronous (HTTP/gRPC):** Service A calls Service B directly and waits for a response. Used for real-time queries (e.g., "get user profile").
2. **Asynchronous (Message Queue):** Service A publishes an event; Service B processes it independently. Used for notifications, billing, analytics.

**Example:** When a ride is booked:

- Ride Service calls Auth Service (HTTP) to validate the token → synchronous.
- Ride Service publishes `ride.created` to SQS → Notification Service sends a push → asynchronous.

---

## Questions: Can you show how one service calls another service?

## Answer:

Using NestJS `HttpService` (wraps Axios):

```typescript
// ride.service.ts
import { HttpService } from '@nestjs/axios';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class RideService {
  constructor(private readonly http: HttpService) {}

  async getUserProfile(userId: string, token: string) {
    const { data } = await firstValueFrom(
      this.http.get(`${process.env.USER_SERVICE_URL}/users/${userId}`, {
        headers: { Authorization: `Bearer ${token}` },
      })
    );
    return data;
  }
}
```

For resilience, we wrapped these calls with a **retry** (via `axios-retry`) and **circuit breaker** (via `opossum`).

---

## Questions: Can you write code to fetch a user's geolocation (latitude and longitude)?

## Answer:

**Browser (Frontend):**

```javascript
navigator.geolocation.getCurrentPosition(
  (position) => {
    const { latitude, longitude } = position.coords;
    console.log(`Lat: ${latitude}, Lng: ${longitude}`);
  },
  (error) => console.error('Location error:', error),
  { enableHighAccuracy: true, timeout: 5000 }
);
```

**Backend (from IP using a third-party API):**

```typescript
import axios from 'axios';

async function getLocationFromIP(ip: string) {
  const { data } = await axios.get(`https://ipapi.co/${ip}/json/`);
  return { lat: data.latitude, lng: data.longitude, city: data.city };
}
```

---

## Questions: Which platform is your application running on—AWS or GCP?

## Answer:

Our application runs primarily on **AWS**. Key services we use:

- **EKS** for Kubernetes orchestration.
- **RDS (PostgreSQL)** for relational data.
- **ElastiCache (Redis)** for caching and Pub/Sub.
- **S3** for file storage.
- **SQS/SNS** for messaging.
- **ALB** (Application Load Balancer) in front of EKS.
- **CloudWatch** for logs and monitoring.

---

## Questions: What kind of optimization or performance improvements have you implemented?

## Answer:

1. **Database indexing** — Added composite indexes on frequently queried columns, reducing query time from 800ms to 50ms.
2. **Redis caching** — Cached user profiles and config data with a 5-minute TTL, cutting DB load by ~60%.
3. **Pagination** — Replaced full-table fetches with cursor-based pagination.
4. **Connection pooling** — Used `pg-pool` with a pool size tuned to the DB's max connections.
5. **Promise.all** — Parallelized independent async calls instead of awaiting sequentially.
6. **Compression** — Enabled gzip via `compression` middleware, reducing response size by ~70%.

**Example:**

```typescript
// Before (sequential) — 600ms
const user = await getUser(id);
const rides = await getRides(id);

// After (parallel) — 300ms
const [user, rides] = await Promise.all([getUser(id), getRides(id)]);
```

---

## Questions: Is Node.js blocking or non‑blocking?

## Answer:

Node.js is **non-blocking by default** for I/O operations. When you make a DB call or HTTP request, Node doesn't wait — it registers a callback/Promise and moves on to the next task. The event loop picks up the result when the operation completes.

**Exception:** CPU-heavy synchronous code (like parsing a huge JSON or complex computation) **does block** the event loop because there's no I/O to offload. For those, use `worker_threads`.

**Example:**

```javascript
// Non-blocking — doesn't freeze the server
fs.readFile('large.csv', 'utf8', (err, data) => {
  console.log('File read done');
});
console.log('This runs immediately, before file is read');
```

---

## Questions: How would you run a Node.js application in a multi‑threaded environment?

## Answer:

Node offers two approaches:

1. **`worker_threads`** — True multi-threading for CPU-intensive tasks. Workers share memory via `SharedArrayBuffer`.
2. **`cluster` module** — Forks multiple Node.js processes (one per CPU core), each with its own event loop. The primary process load-balances connections.

**Example with `cluster`:**

```javascript
import cluster from 'cluster';
import os from 'os';

if (cluster.isPrimary) {
  const numCPUs = os.cpus().length;
  for (let i = 0; i < numCPUs; i++) cluster.fork();
} else {
  // Each worker runs the Express/NestJS app
  app.listen(3000);
}
```

In production we prefer **multiple pods in Kubernetes** over the cluster module — same effect, better isolation.

---

## Questions: Can you explain how Node.js handles concurrency?

## Answer:

Node.js achieves concurrency through the **event loop** — a continuous loop that picks up and executes callbacks when async operations complete.

**Phases of the event loop (simplified):**

1. **Timers** — `setTimeout`, `setInterval` callbacks.
2. **I/O callbacks** — network, file system results.
3. **Poll** — wait for new I/O events.
4. **Check** — `setImmediate` callbacks.
5. **Close callbacks** — `socket.on('close')`.

`process.nextTick` runs before the next phase (highest priority).

**Key insight:** Node doesn't run things "in parallel" on one thread — it interleaves them. While waiting for DB response A, it starts DB call B. Both resolve in roughly the same wall-clock time.

---

## Questions: Can you explain a microservices‑based architecture you have worked on?

## Answer:

We built a ride-hailing platform with the following architecture:

```
Mobile App / Web
       ↓
  API Gateway (NestJS + JWT auth + rate limiting)
  ↙    ↓    ↓    ↘
Auth  User  Ride  Tracking
             ↓
           SQS
          /    \
   Notification  Payment
```

- Each service had its own PostgreSQL database.
- Internal sync communication via HTTP (axios).
- Async events via AWS SQS.
- Redis for session data and real-time location.
- Deployed on AWS EKS with 3 replicas per service.

---

## Questions: How do you communicate between different microservices?

## Answer:

Same as above — two patterns:

- **REST (HTTP):** `GET /users/:id` from Ride Service to User Service for real-time lookups.
- **Message Queue (SQS/RabbitMQ):** `ride.completed` event consumed by Notification and Payment services asynchronously.

For **NestJS microservices**, you can also use the built-in transport layer:

```typescript
// In main.ts of Notification Service
app.connectMicroservice({
  transport: Transport.SQS,
  options: { ... }
});

// Listener
@MessagePattern('ride.completed')
handleRideCompleted(data: RideCompletedDto) {
  this.notificationService.sendReceipt(data);
}
```

---

## Questions: How do you consume a third‑party API that returns a very large response?

## Answer:

Use **streaming** to process data chunk by chunk instead of buffering the entire response in memory.

**Example with Node.js streams:**

```typescript
import axios from 'axios';
import { createWriteStream } from 'fs';

async function downloadLargeFile(url: string, outputPath: string) {
  const response = await axios.get(url, { responseType: 'stream' });
  const writer = createWriteStream(outputPath);
  response.data.pipe(writer);
  return new Promise((resolve, reject) => {
    writer.on('finish', resolve);
    writer.on('error', reject);
  });
}
```

For large JSON arrays, use `stream-json` to parse line by line without loading everything into RAM.

---

## Questions: What considerations are needed when handling large JSON responses?

## Answer:

1. **Memory:** Avoid `JSON.parse` on huge payloads — use streaming parsers (`stream-json`).
2. **Pagination:** Request data in pages (`limit`/`offset` or cursor) instead of all at once.
3. **Timeout:** Set axios timeout to avoid hanging forever.
4. **Rate limits:** Respect `Retry-After` headers from the API.
5. **Error handling:** Wrap in try/catch and handle partial failures.
6. **Data transformation:** Transform/filter data as it streams to avoid holding large objects in memory.

---

## Questions: How would you handle JWT expiry and rate limiting when consuming third‑party APIs?

## Answer:

**JWT Expiry:**

```typescript
async function getToken(): Promise<string> {
  if (this.token && !isExpired(this.token)) return this.token;
  const response = await axios.post('/auth/token', credentials);
  this.token = response.data.access_token;
  return this.token;
}
```

Cache the token and refresh it before it expires (check `exp` claim).

**Rate Limiting:**

```typescript
import Bottleneck from 'bottleneck';

const limiter = new Bottleneck({ minTime: 200, maxConcurrent: 5 }); // 5 req/sec max

const result = await limiter.schedule(() => axios.get('/api/data'));
```

Also respect the `X-RateLimit-Remaining` and `Retry-After` response headers.

---

## Questions: How would you ensure no data loss while fetching large volumes of data?

## Answer:

1. **Checkpointing:** Save the last successfully processed page/cursor to DB or Redis. On restart, resume from there.
2. **Idempotency:** Make processing idempotent — if the same record is processed twice, it produces the same result.
3. **Transactions:** Wrap DB writes in transactions so partial writes are rolled back.
4. **Dead Letter Queue:** Failed messages go to DLQ for manual inspection and reprocessing.

**Example (checkpoint pattern):**

```typescript
let cursor = (await redis.get('fetch:cursor')) || '0';
while (true) {
  const page = await api.getRecords({ after: cursor });
  if (!page.data.length) break;
  await processRecords(page.data);
  cursor = page.nextCursor;
  await redis.set('fetch:cursor', cursor);
}
```

---

## Questions: How would you paginate or chunk large API responses?

## Answer:

**Offset-based pagination:**

```typescript
const pageSize = 100;
let offset = 0;
while (true) {
  const data = await api.get(`/records?limit=${pageSize}&offset=${offset}`);
  if (!data.length) break;
  await processRecords(data);
  offset += pageSize;
}
```

**Cursor-based pagination (preferred for large datasets — consistent even if data changes):**

```typescript
let cursor: string | null = null;
do {
  const res = await api.get('/records', { params: { cursor, limit: 100 } });
  await processRecords(res.data);
  cursor = res.nextCursor;
} while (cursor);
```

---

## Questions: How would you limit API calls to comply with rate limits?

## Answer:

1. **`Bottleneck` library** — token bucket / leaky bucket algorithm.
2. **Exponential backoff** on 429 responses.
3. **Queue + delay** — manually control call frequency.

**Example:**

```typescript
import Bottleneck from 'bottleneck';

const limiter = new Bottleneck({
  reservoir: 100,
  reservoirRefreshAmount: 100,
  reservoirRefreshInterval: 60 * 1000,
}); // 100 req/min

const data = await limiter.schedule(() => callApi());
```

---

## Questions: How do you implement delays or throttling in Node.js?

## Answer:

**Simple delay:**

```typescript
const sleep = (ms: number) => new Promise((resolve) => setTimeout(resolve, ms));

for (const item of items) {
  await processItem(item);
  await sleep(200); // 200ms between each call = 5 calls/sec
}
```

**Throttling with `p-limit`:**

```typescript
import pLimit from 'p-limit';
const limit = pLimit(5); // max 5 concurrent

const results = await Promise.all(
  items.map((item) => limit(() => processItem(item)))
);
```

---

## Questions: How would you make a POST API call with a given JSON payload?

## Answer:

**Using Axios:**

```typescript
import axios from 'axios';

async function createOrder(payload: CreateOrderDto) {
  try {
    const { data } = await axios.post(
      'https://api.example.com/orders',
      payload,
      {
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Bearer ${token}`,
        },
        timeout: 5000,
      }
    );
    return data;
  } catch (error) {
    if (axios.isAxiosError(error)) {
      throw new Error(
        `API error: ${error.response?.status} - ${error.response?.data?.message}`
      );
    }
    throw error;
  }
}
```

---

## Questions: How would you write a production‑ready POST API in Node.js?

## Answer:

```typescript
// NestJS example — production-ready POST API

// dto/create-user.dto.ts
export class CreateUserDto {
  @IsString() @IsNotEmpty() name: string;
  @IsEmail() email: string;
  @MinLength(8) password: string;
}

// users.controller.ts
@Post()
@UseGuards(JwtAuthGuard)
@HttpCode(HttpStatus.CREATED)
async createUser(@Body() dto: CreateUserDto, @Request() req) {
  return this.usersService.create(dto);
}

// users.service.ts
async create(dto: CreateUserDto) {
  const existing = await this.userRepo.findOne({ where: { email: dto.email } });
  if (existing) throw new ConflictException('Email already in use');
  const hashed = await bcrypt.hash(dto.password, 10);
  const user = this.userRepo.create({ ...dto, password: hashed });
  await this.userRepo.save(user);
  const { password, ...result } = user;
  return result;
}
```

Key aspects: validation (class-validator), auth guard, proper HTTP status code, no password in response, conflict handling.

---

## Questions: Which framework would you use to build APIs in Node.js?

## Answer:

**NestJS** for large, production-grade applications — it gives structure (modules, controllers, services), built-in DI, guards, interceptors, pipes, and excellent TypeScript support.

**Express** for lightweight APIs or microservices where minimal overhead is needed.

**Fastify** when raw performance is critical — 2–3x faster than Express.

**My preference:** NestJS for team projects (maintainability), Express/Fastify for internal tools or edge lambdas.

---

## Questions: How do you structure APIs using controllers and routers?

## Answer:

**NestJS structure:**

```
src/
  users/
    users.module.ts       ← wires everything together
    users.controller.ts   ← handles HTTP requests, routes
    users.service.ts      ← business logic
    users.repository.ts   ← data access
    dto/
      create-user.dto.ts
    entities/
      user.entity.ts
```

**Controller example:**

```typescript
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }
}
```

**Express Router:**

```typescript
const router = Router();
router.get('/:id', getUser);
router.post('/', createUser);
app.use('/users', router);
```

---

## Questions: What security practices do you follow while developing Node.js applications?

## Answer:

1. **Input validation** — `class-validator` or `joi` to reject malformed input.
2. **JWT authentication** — short-lived access tokens + refresh tokens.
3. **Password hashing** — `bcrypt` with salt rounds ≥ 10.
4. **Rate limiting** — `express-rate-limit` or API Gateway throttling.
5. **Helmet** — sets secure HTTP headers (XSS protection, HSTS, CSP).
6. **SQL injection prevention** — always use parameterized queries / ORM.
7. **Secrets management** — environment variables via AWS Secrets Manager, never hardcoded.
8. **CORS** — whitelist allowed origins explicitly.
9. **Dependency scanning** — `npm audit` and Snyk in CI pipeline.

---

## Questions: Which AWS services have you worked with?

## Answer:

- **Compute:** EC2, Lambda, EKS (Kubernetes)
- **Storage:** S3, EBS
- **Database:** RDS (PostgreSQL), DynamoDB, ElastiCache (Redis)
- **Messaging:** SQS, SNS
- **Networking:** VPC, ALB, Route 53, API Gateway
- **Monitoring:** CloudWatch, X-Ray
- **Security:** IAM, Secrets Manager, KMS
- **CI/CD:** CodePipeline, CodeBuild

---

## Questions: In what scenarios did you use SQS?

## Answer:

1. **Ride completion events** — Ride Service publishes to SQS; Notification and Payment services consume.
2. **Email/SMS queue** — Bulk notifications queued to avoid overwhelming the email provider.
3. **Report generation** — Heavy PDF/CSV reports processed async; user polls for status.
4. **Retry mechanism** — Failed third-party API calls queued for retry with exponential backoff.

SQS decouples services and ensures reliability — if the consumer is down, messages wait safely in the queue.

---

## Questions: In what scenarios did you use SNS for notifications?

## Answer:

**SNS** is for **fan-out** — one event needs to trigger multiple consumers.

**Example:** When a new driver registers:

- SNS publishes `driver.registered`.
- SQS queue 1 → Email Service sends welcome email.
- SQS queue 2 → Admin Service creates a review task.
- SQS queue 3 → Analytics Service logs the event.

```typescript
await sns
  .publish({
    TopicArn: process.env.DRIVER_EVENTS_TOPIC,
    Message: JSON.stringify({ event: 'driver.registered', driverId }),
    MessageAttributes: {
      eventType: { DataType: 'String', StringValue: 'driver.registered' },
    },
  })
  .promise();
```

---

## Questions: How do you handle alerting and monitoring in a production environment?

## Answer:

1. **Structured logging** — Winston/Pino with JSON format, shipped to CloudWatch Logs.
2. **Metrics** — Custom CloudWatch metrics for queue depth, API latency, error rates.
3. **Alarms** — CloudWatch Alarms trigger when error rate > 1% or P99 latency > 2s.
4. **Distributed tracing** — Datadog APM traces requests across services.
5. **Error tracking** — Sentry captures uncaught exceptions with full stack traces.
6. **On-call** — PagerDuty escalation policies alert the right engineer.

**Example alarm setup:**

- If `5xx` errors exceed 10 in 5 minutes → SNS → PagerDuty alert.

---

## Questions: What is your current role and responsibility?

## Answer:

_(Same as the first question — see top of file.)_
I'm a Backend Developer working with NestJS and Node.js, building scalable microservices, REST APIs, and event-driven systems deployed on AWS.

---

## Questions: How comfortable are you with Node.js (rate yourself out of 10)?

## Answer:

I'd rate myself **7.5–8/10**. I'm confident with the event loop, async patterns, REST API design, microservices, testing, and performance optimization in Node.js. I'm continuously deepening my knowledge of advanced internals like the V8 engine, memory profiling, and stream optimizations. I believe in being honest — there's always more to learn.

---

## Questions: How does Node.js handle concurrency?

## Answer:

_(See earlier answer — "Can you explain how Node.js handles concurrency?")_

The core mechanism is the **non-blocking event loop**. Node offloads I/O to libuv's thread pool, frees the main thread to accept more requests, and processes results when they're ready. This allows thousands of concurrent connections on a single thread — as long as you don't block the event loop with CPU-heavy synchronous code.

---

## Questions: When does Node.js perform poorly?

## Answer:

Node.js performs poorly when:

1. **CPU-intensive tasks** run on the main thread (image processing, video encoding, large data crunching) — they block the event loop and freeze all other requests.
2. **Large synchronous operations** — `JSON.parse` on a huge object, complex regex on a big string.
3. **Memory leaks** — holding large objects, unbounded caches, event listener accumulation.

**Fix:** Offload CPU work to `worker_threads` or a separate service. Use streaming for large data.

---

## Questions: What is the event loop?

## Answer:

The **event loop** is Node.js's mechanism for handling asynchronous operations. It continuously checks if the call stack is empty, and if so, picks the next callback from the queue and executes it.

```
   ┌─────────────────────────────┐
   │           timers            │  ← setTimeout, setInterval
   ├─────────────────────────────┤
   │     pending callbacks       │  ← I/O errors
   ├─────────────────────────────┤
   │       idle, prepare         │  ← internal
   ├─────────────────────────────┤
   │           poll              │  ← wait for I/O
   ├─────────────────────────────┤
   │           check             │  ← setImmediate
   ├─────────────────────────────┤
   │      close callbacks        │  ← socket.on('close')
   └─────────────────────────────┘
```

This allows Node to handle many concurrent connections without spawning threads for each one.

---

## Questions: Can you explain the phases of the event loop?

## Answer:

1. **Timers** — Executes callbacks from `setTimeout` and `setInterval` whose delay has expired.
2. **Pending I/O callbacks** — I/O errors from the previous cycle.
3. **Idle/Prepare** — Internal use only.
4. **Poll** — Retrieves new I/O events and executes their callbacks. If none, waits here.
5. **Check** — Executes `setImmediate` callbacks.
6. **Close callbacks** — `socket.destroy()`, `socket.on('close')`.

**Special:** `process.nextTick` and Promise microtasks run **between every phase**, before the loop continues.

**Example:**

```javascript
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));

// Output: nextTick → promise → timeout → immediate (order of timeout/immediate can vary)
```

---

## Questions: Can you write a function to count character occurrences?

## Answer:

```typescript
function countCharOccurrences(str: string): Record<string, number> {
  return str.split('').reduce(
    (acc, char) => {
      acc[char] = (acc[char] || 0) + 1;
      return acc;
    },
    {} as Record<string, number>
  );
}

console.log(countCharOccurrences('hello'));
// { h: 1, e: 1, l: 2, o: 1 }
```

**One-liner alternative:**

```typescript
const count = (s: string) =>
  [...s].reduce((a, c) => ({ ...a, [c]: (a[c] || 0) + 1 }), {});
```

---

## Questions: How do you handle errors in asynchronous Node.js code?

## Answer:

**1. async/await with try/catch (preferred):**

```typescript
async function getUser(id: string) {
  try {
    const user = await userRepository.findOneOrFail(id);
    return user;
  } catch (error) {
    if (error instanceof EntityNotFoundError) {
      throw new NotFoundException(`User ${id} not found`);
    }
    throw new InternalServerErrorException('Database error');
  }
}
```

**2. Global exception filter in NestJS:**

```typescript
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const status =
      exception instanceof HttpException ? exception.getStatus() : 500;
    ctx
      .getResponse()
      .status(status)
      .json({ statusCode: status, message: 'Something went wrong' });
  }
}
```

**3. Unhandled rejections (safety net):**

```javascript
process.on('unhandledRejection', (reason) => {
  logger.error('Unhandled rejection:', reason);
});
```

---

## Questions: What is the difference between process.nextTick and setImmediate?

## Answer:

|              | `process.nextTick`                             | `setImmediate`            |
| ------------ | ---------------------------------------------- | ------------------------- |
| **Runs**     | Before next event loop phase (microtask queue) | In the **Check** phase    |
| **Priority** | Higher — runs before I/O callbacks             | Lower — runs after I/O    |
| **Use case** | Defer something before I/O                     | Defer something after I/O |

**Example:**

```javascript
setImmediate(() => console.log('setImmediate'));
process.nextTick(() => console.log('nextTick'));
console.log('sync');

// Output: sync → nextTick → setImmediate
```

**Rule of thumb:** Use `process.nextTick` to ensure something runs before any I/O. Use `setImmediate` to yield to I/O then run something.

---

## Questions: What are call, apply, and bind in JavaScript?

## Answer:

All three let you set `this` explicitly for a function.

- **`call`** — invokes immediately, arguments passed one by one.
- **`apply`** — invokes immediately, arguments passed as an array.
- **`bind`** — returns a new function with `this` permanently bound.

**Example:**

```javascript
function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}
const user = { name: 'Pulkit' };

greet.call(user, 'Hello', '!'); // "Hello, Pulkit!"
greet.apply(user, ['Hi', '.']); // "Hi, Pulkit."
const boundGreet = greet.bind(user);
boundGreet('Hey', '?'); // "Hey, Pulkit?"
```

**Interview tip:** `bind` is useful for event handlers and React class component methods where you need to preserve `this`.

---

## Questions: What is a closure? Can you give a real‑time use case?

## Answer:

A **closure** is a function that retains access to variables from its outer (enclosing) scope even after that scope has returned.

**Real-time use case — rate limiter / counter:**

```typescript
function createRateLimiter(maxRequests: number, windowMs: number) {
  let count = 0;
  let windowStart = Date.now();

  return function isAllowed(): boolean {
    const now = Date.now();
    if (now - windowStart > windowMs) {
      count = 0;
      windowStart = now;
    }
    if (count < maxRequests) {
      count++;
      return true;
    }
    return false;
  };
}

const limiter = createRateLimiter(5, 1000); // 5 requests per second
console.log(limiter()); // true
```

`count` and `windowStart` are private — not accessible from outside, but the returned function remembers them. This is closure in action.

---

## Questions: What is the difference between interface and type in TypeScript?

## Answer:

|                         | `interface`                        | `type`             |
| ----------------------- | ---------------------------------- | ------------------ |
| **Extension**           | `extends`                          | `&` (intersection) |
| **Declaration merging** | Yes (can re-declare to add fields) | No                 |
| **Primitives/unions**   | No                                 | Yes                |
| **Computed types**      | No                                 | Yes                |

**Use `interface` for:** defining object shapes, class contracts, public APIs.
**Use `type` for:** unions, intersections, mapped types, utility types.

**Examples:**

```typescript
// Interface
interface User {
  id: string;
  name: string;
}
interface AdminUser extends User {
  role: 'admin';
}

// Type
type Status = 'active' | 'inactive' | 'banned'; // union — only possible with type
type PartialUser = Partial<User>; // mapped type

// Both work for objects — prefer interface for objects, type for everything else
```

---

## Questions: How do you deploy a Node.js application to Azure?

## Answer:

1. **Azure App Service (easiest):** Zip deploy or connect GitHub → CI/CD auto-deploys on push.
2. **Azure Container Registry + AKS:** Build Docker image → push to ACR → deploy to AKS.
3. **Azure Functions:** Serverless — ideal for lightweight APIs.

**Basic App Service deployment via Azure CLI:**

```bash
az webapp create --resource-group myRG --plan myPlan --name myApp --runtime "NODE:20-lts"
az webapp deployment source config-zip --resource-group myRG --name myApp --src build.zip
```

Set environment variables:

```bash
az webapp config appsettings set --name myApp --resource-group myRG --settings NODE_ENV=production DB_URL=...
```

---

## Questions: Have you worked with HTML and CSS?

## Answer:

Yes. I've built responsive UI components and admin dashboards using HTML5, CSS3, and frameworks like Tailwind CSS and Bootstrap. While my primary focus is backend, I'm comfortable writing semantic HTML, Flexbox/Grid layouts, and basic component styling. I've also worked with React where I wrote JSX alongside CSS modules.

---

## Questions: What is the difference between `<div>` and `<section>`?

## Answer:

- **`<div>`** is a generic, **non-semantic** container. It carries no meaning — purely structural.
- **`<section>`** is a **semantic** HTML5 element that represents a distinct section of content, typically with a heading.

**When to use:**

- Use `<section>` when the content has a clear theme or topic (e.g., "Features", "About Us").
- Use `<div>` purely for styling/layout purposes with no semantic meaning.

**Example:**

```html
<section>
  <h2>Our Services</h2>
  <p>We offer...</p>
</section>

<div class="flex-container">
  <!-- just a layout wrapper, no semantic meaning -->
  <div class="card">...</div>
</div>
```

---

## Questions: What is the difference between id and class?

## Answer:

|                     | `id`                      | `class`                     |
| ------------------- | ------------------------- | --------------------------- |
| **Uniqueness**      | Must be unique per page   | Reusable across elements    |
| **CSS specificity** | Higher (100)              | Lower (10)                  |
| **JS targeting**    | `document.getElementById` | `document.querySelectorAll` |
| **Use case**        | Anchors, unique landmarks | Shared styles               |

**Example:**

```html
<header id="main-header">...</header>
<!-- unique landmark -->
<button class="btn btn-primary">Submit</button>
<!-- reusable style -->
<button class="btn btn-secondary">Cancel</button>
```

---

## Questions: What are variables in SASS?

## Answer:

SASS variables store reusable values (colors, fonts, spacing) prefixed with `$`.

**Example:**

```scss
$primary-color: #3498db;
$font-size-base: 16px;
$spacing-md: 1rem;

.button {
  background-color: $primary-color;
  font-size: $font-size-base;
  padding: $spacing-md;
}

.link {
  color: $primary-color; // reuse — change once, updates everywhere
}
```

**Interview tip:** SASS variables are compile-time; CSS custom properties (`--color`) are runtime and can be changed with JS.

---

## Questions: Have you worked with mixins?

## Answer:

Yes. A **mixin** is a reusable block of CSS that can accept arguments — like a function in SCSS.

**Example:**

```scss
@mixin flex-center($direction: row) {
  display: flex;
  flex-direction: $direction;
  justify-content: center;
  align-items: center;
}

.hero {
  @include flex-center(column);
  height: 100vh;
}

.card-row {
  @include flex-center(); // defaults to row
}
```

Another common use: responsive breakpoints:

```scss
@mixin respond-to($breakpoint) {
  @if $breakpoint == 'mobile' {
    @media (max-width: 768px) {
      @content;
    }
  }
}

.container {
  width: 1200px;
  @include respond-to('mobile') {
    width: 100%;
  }
}
```

---

## Questions: Have you optimized CSS performance? If yes, how?

## Answer:

1. **Remove unused CSS** — PurgeCSS / Tailwind's JIT mode removes unused classes at build time.
2. **Minification** — CSS minifiers (cssnano) reduce file size.
3. **Critical CSS** — Inline above-the-fold CSS to avoid render-blocking.
4. **Avoid expensive selectors** — `*`, deeply nested selectors, `:nth-child` on large lists.
5. **Use CSS variables over JS manipulation** — faster than toggling classes.
6. **Reduce repaints/reflows** — use `transform` and `opacity` for animations (compositor-only, no layout recalculation).

**Example:** Changed a `width` animation (triggers layout) to a `transform: scaleX()` animation — eliminated layout thrashing and made it silky smooth.

---

## Questions: How would you implement a secured GET API using JWT?

## Answer:

```typescript
// jwt.strategy.ts
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: process.env.JWT_SECRET,
    });
  }
  async validate(payload: JwtPayload) {
    return { userId: payload.sub, email: payload.email };
  }
}

// users.controller.ts
@Get('profile')
@UseGuards(JwtAuthGuard)
getProfile(@Request() req) {
  return this.usersService.findOne(req.user.userId);
}
```

**Request flow:**

1. Client sends `Authorization: Bearer <token>` header.
2. `JwtAuthGuard` extracts and verifies the token using the secret.
3. `validate` returns the user payload → attached to `req.user`.
4. Controller uses `req.user.userId` to fetch data.

---

## Questions: Which logging mechanism do you use?

## Answer:

**Winston** (most common) or **Pino** (faster, JSON-first) in production.

**Example with Winston in NestJS:**

```typescript
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';

WinstonModule.createLogger({
  transports: [
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json()
      ),
    }),
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
  ],
});
```

**Best practices:**

- Always log with a **correlation ID** (trace ID) for request tracking.
- Use **log levels**: `error`, `warn`, `info`, `debug`.
- Never log sensitive data (passwords, tokens, PII).
- Ship logs to CloudWatch / ELK / Datadog for centralized analysis.

---

## Questions: What caching mechanisms have you used?

## Answer:

1. **Redis** — Most common for distributed caching (session data, rate limiting, live tracking).
2. **In-memory cache** — `node-cache` or a simple `Map` for single-instance, short-lived data.
3. **CDN caching** — CloudFront for static assets.
4. **HTTP caching** — `Cache-Control` headers on API responses.

**Example with Redis in NestJS:**

```typescript
@Injectable()
export class UserService {
  constructor(@InjectRedis() private readonly redis: Redis) {}

  async findOne(id: string) {
    const cached = await this.redis.get(`user:${id}`);
    if (cached) return JSON.parse(cached);

    const user = await this.userRepo.findOneBy({ id });
    await this.redis.setex(`user:${id}`, 300, JSON.stringify(user)); // TTL 5 min
    return user;
  }
}
```

Cache invalidation: delete the Redis key when the user is updated.

---

## Questions: Can you write code for a GET API that returns user details and related data?

## Answer:

```typescript
// users.controller.ts
@Get(':id')
@UseGuards(JwtAuthGuard)
async getUserWithDetails(@Param('id') id: string) {
  return this.usersService.findWithDetails(id);
}

// users.service.ts
async findWithDetails(id: string) {
  const [user, orders, addresses] = await Promise.all([
    this.userRepo.findOneByOrFail({ id }),
    this.orderRepo.find({ where: { userId: id }, order: { createdAt: 'DESC' }, take: 10 }),
    this.addressRepo.find({ where: { userId: id } }),
  ]);

  return { ...user, orders, addresses };
}
```

**Response:**

```json
{
  "id": "123",
  "name": "Pulkit",
  "email": "pulkit@example.com",
  "orders": [...],
  "addresses": [...]
}
```

---

## Questions: Have you heard about Express Router? Why do we use it?

## Answer:

Yes. `express.Router()` creates a **mini-application** — a modular set of routes that can be mounted at a path in the main app.

**Why use it:**

- Organizes routes by feature/domain (users, orders, rides).
- Each router file is isolated — cleaner, easier to test.
- Middleware can be scoped to a specific router.

**Example:**

```typescript
// routes/users.router.ts
const router = Router();
router.use(authMiddleware); // applies only to /users routes
router.get('/', getAllUsers);
router.get('/:id', getUserById);
router.post('/', createUser);
export default router;

// app.ts
app.use('/users', usersRouter);
app.use('/orders', ordersRouter);
```

---

## Questions: Do you write test cases for your services?

## Answer:

Yes. I write:

- **Unit tests** — Test service methods in isolation by mocking dependencies (Jest).
- **Integration tests** — Test the full request/response cycle (Supertest + NestJS testing module).
- **E2E tests** — Test full flows end-to-end.

**Unit test example (NestJS + Jest):**

```typescript
describe('UsersService', () => {
  let service: UsersService;
  let repo: jest.Mocked<UserRepository>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: UserRepository,
          useValue: { findOneBy: jest.fn(), save: jest.fn() },
        },
      ],
    }).compile();
    service = module.get(UsersService);
    repo = module.get(UserRepository);
  });

  it('should return user by id', async () => {
    repo.findOneBy.mockResolvedValue({ id: '1', name: 'Pulkit' } as User);
    const user = await service.findOne('1');
    expect(user.name).toBe('Pulkit');
    expect(repo.findOneBy).toHaveBeenCalledWith({ id: '1' });
  });
});
```

---

## Questions: How comfortable are you with SQL?

## Answer:

I'm comfortable with SQL — I use it daily through TypeORM and also write raw queries when needed. I'm confident with:

- Joins (INNER, LEFT, RIGHT)
- Aggregations (`GROUP BY`, `HAVING`, window functions)
- Subqueries and CTEs
- Indexes (B-tree, partial indexes, composite indexes)
- Query optimization via `EXPLAIN ANALYZE`
- Transactions and isolation levels

**Example (complex query):**

```sql
SELECT
  u.id,
  u.name,
  COUNT(r.id) AS total_rides,
  SUM(r.fare) AS total_spent
FROM users u
LEFT JOIN rides r ON r.user_id = u.id
WHERE r.created_at >= NOW() - INTERVAL '30 days'
GROUP BY u.id, u.name
HAVING COUNT(r.id) > 5
ORDER BY total_spent DESC
LIMIT 10;
```

---

## Questions: Have you worked with circuit breakers?

## Answer:

Yes. A **circuit breaker** prevents cascading failures — if Service B is down, we stop hammering it and return a fallback instead of waiting for timeouts.

**States:** Closed (normal) → Open (failing, reject all calls) → Half-Open (test with a few calls) → Closed again.

**Example with `opossum`:**

```typescript
import CircuitBreaker from 'opossum';

const options = {
  timeout: 3000, // fail if no response in 3s
  errorThresholdPercentage: 50, // open if 50% of calls fail
  resetTimeout: 10000, // try again after 10s
};

const breaker = new CircuitBreaker(callPaymentService, options);
breaker.fallback(() => ({
  status: 'payment service unavailable, retry later',
}));

const result = await breaker.fire(payload);
```

---

## Questions: How do you mock a service while writing tests?

## Answer:

In NestJS/Jest, replace the real service with a mock object:

```typescript
// Inline mock
const mockUserService = {
  findOne: jest.fn().mockResolvedValue({ id: '1', name: 'Pulkit' }),
  create: jest.fn(),
};

const module = await Test.createTestingModule({
  controllers: [UsersController],
  providers: [{ provide: UsersService, useValue: mockUserService }],
}).compile();

// Then in tests:
it('GET /users/:id returns user', async () => {
  mockUserService.findOne.mockResolvedValueOnce({ id: '1', name: 'Test' });
  const result = await controller.findOne('1');
  expect(result.name).toBe('Test');
});
```

For HTTP calls (axios), mock with `jest.spyOn`:

```typescript
jest
  .spyOn(httpService, 'get')
  .mockReturnValue(of({ data: { userId: '1' } } as any));
```

---

## Questions: What does the navigate method do in React?

## Answer:

`navigate` from `react-router-dom` (v6+) programmatically redirects the user to a different route — without a page reload (SPA navigation).

```typescript
import { useNavigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();

  const handleLogin = async () => {
    await authService.login(credentials);
    navigate('/dashboard'); // go forward
    navigate(-1); // go back (like browser back button)
    navigate('/profile', { replace: true }); // replace current history entry
  };
}
```

---

## Questions: How do you pass data while navigating between pages in React?

## Answer:

**1. URL params (simplest, bookmarkable):**

```typescript
navigate(`/users/${userId}`);
// receive: const { userId } = useParams();
```

**2. Query strings:**

```typescript
navigate(`/search?q=rides&status=active`);
// receive: const [params] = useSearchParams(); params.get('q');
```

**3. Location state (temporary, not in URL):**

```typescript
navigate('/order-summary', { state: { orderId: '123', amount: 500 } });
// receive:
const location = useLocation();
const { orderId } = location.state;
```

**Note:** Location state is lost on page refresh — use it for transient data like form results.

---

## Questions: Are microservices stateful or stateless?

## Answer:

Microservices should be **stateless** — each request contains all the information needed to process it. No in-memory session data is stored between requests.

**Why stateless matters:**

- You can spin up any number of replicas and any of them can handle any request.
- Scaling horizontally becomes trivial.
- No sticky sessions needed.

**Where does state live then?**

- User sessions → Redis.
- Business data → Database (PostgreSQL, MongoDB).
- Files → S3.

**Example:** A JWT token carries the user's identity — the service doesn't store "who is logged in." It validates the token on every request.

---

## Questions: Have you worked with microservices?

## Answer:

Yes, extensively. I built and maintained a microservices-based ride-hailing platform with 6–8 services (Auth, User, Ride, Tracking, Notification, Payment, Admin, API Gateway). Each service had its own codebase, database, and deployment pipeline. I handled service-to-service communication (HTTP + SQS), distributed tracing, centralized logging, and container orchestration with Kubernetes on AWS EKS.

---

## Questions: Have you handled high‑concurrency scenarios?

## Answer:

Yes. In the ride-tracking service, we handled thousands of location updates per second during peak hours. Strategies we used:

1. **Redis for hot data** — latest driver positions stored in Redis (O(1) reads/writes) instead of DB.
2. **WebSocket rooms** — riders subscribed only to their ride's room, reducing broadcast overhead.
3. **Batch DB writes** — location history written to PostgreSQL in batches every 5 seconds (not on every GPS ping).
4. **Horizontal scaling** — 5 pods per service with a sticky load balancer for WebSocket connections.
5. **Rate limiting on clients** — driver app sends updates every 3s max.

---

## Questions: Can you explain Kafka architecture (producer, topic, consumer)?

## Answer:

**Apache Kafka** is a distributed event streaming platform.

**Key concepts:**

- **Producer:** Publishes messages to a topic.
- **Topic:** A named log of ordered messages, split into **partitions**.
- **Partition:** Each partition is an ordered, immutable log. Enables parallelism.
- **Consumer:** Reads messages from a topic. Belongs to a **consumer group**.
- **Consumer group:** Each partition is assigned to one consumer — enabling parallel consumption.
- **Broker:** A Kafka server that stores partitions.
- **Offset:** The position of a consumer in a partition. Consumers commit offsets to track progress.

**Example flow:**

```
Ride Service (Producer)
       ↓ publishes "ride.completed"
    Kafka Topic: ride-events
    Partition 0   Partition 1   Partition 2
       ↓               ↓               ↓
Consumer A        Consumer B     Consumer C
(Notification)   (Payment)     (Analytics)
```

---

## Questions: What happens when producers are faster than consumers in Kafka?

## Answer:

Messages **accumulate in the topic partitions** — Kafka retains them according to its retention policy (e.g., 7 days or 100 GB). Unlike a traditional queue, Kafka doesn't drop unread messages.

**Solutions:**

1. **Scale consumers** — Add more consumer instances in the consumer group (up to the number of partitions).
2. **Increase partitions** — More partitions = more consumers can run in parallel.
3. **Optimize consumer processing** — Batch process messages, reduce per-message processing time.
4. **Backpressure** — Slow down producers if consumers are consistently lagging (monitor consumer lag).

**Monitoring:** Track **consumer lag** (difference between latest offset and consumer's committed offset). Alert if it grows unboundedly.

---

## Questions: What is a dead letter queue?

## Answer:

A **Dead Letter Queue (DLQ)** is a queue where messages are sent when they **fail to be processed** after a maximum number of retries. It prevents bad messages from blocking the main queue indefinitely.

**When a message goes to DLQ:**

- Processing throws an unhandled exception repeatedly.
- Message exceeds visibility timeout without being deleted (in SQS).
- Message is malformed/unparseable.

**AWS SQS DLQ example:**

```json
{
  "RedrivePolicy": {
    "deadLetterTargetArn": "arn:aws:sqs:...:ride-events-dlq",
    "maxReceiveCount": 3
  }
}
```

After 3 failed attempts, the message moves to `ride-events-dlq`. The team gets alerted, investigates, fixes the bug, and redrives the messages back to the main queue.

**Interview tip:** Always set up a DLQ in production — it's your safety net for debugging message processing failures.
