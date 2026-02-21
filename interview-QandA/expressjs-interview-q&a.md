# ExpressJS Interview Questions & Answers

## 1. What is Express.js?

**Answer:** Express.js is a minimal and flexible Node.js web application framework that provides a robust set of features for building web and mobile applications. It's the most popular Node.js framework and is often called the "de facto standard" for Node.js web applications.

**Key Features:**
- Minimal and unopinionated
- Robust routing
- High performance
- Middleware support
- Content negotiation
- Template engine support
- Easy integration with databases

---

## 2. What are the main features of Express.js?

**Answer:**

1. **Fast Development:** Minimal setup required
2. **Routing:** Powerful routing mechanism
3. **Middleware:** Chain of functions for request processing
4. **Template Engines:** Support for Pug, EJS, Handlebars, etc.
5. **HTTP Methods:** Support for GET, POST, PUT, DELETE, etc.
6. **Static Files:** Serve static files easily
7. **Error Handling:** Centralized error handling
8. **Database Integration:** Works with any database
9. **RESTful API:** Easy to build REST APIs
10. **Large Community:** Extensive ecosystem

---

## 3. How do you create a basic Express application?

**Answer:**

```javascript
// Install: npm install express

const express = require('express');
const app = express();
const port = 3000;

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.get('/api/users', (req, res) => {
  res.json([{ id: 1, name: 'John' }]);
});

// Start server
app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

---

## 4. What is Middleware in Express?

**Answer:** Middleware are functions that have access to the request object (req), response object (res), and the next middleware function in the application's request-response cycle.

**Types of Middleware:**
- Application-level middleware
- Router-level middleware
- Error-handling middleware
- Built-in middleware
- Third-party middleware

**Example:**
```javascript
// Application-level middleware
app.use((req, res, next) => {
  console.log('Time:', Date.now());
  next();
});

// Route-specific middleware
app.get('/user/:id', (req, res, next) => {
  console.log('Request Type:', req.method);
  next();
}, (req, res) => {
  res.send('User Info');
});

// Error-handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

---

## 5. What is the difference between app.use() and app.all()?

**Answer:**

| app.use() | app.all() |
|-----------|-----------|
| Mounts middleware | Mounts route handler |
| Matches path prefix | Matches exact path or pattern |
| Can't specify HTTP method | Works for all HTTP methods |
| Called for any HTTP method | More specific routing |

**Example:**
```javascript
// app.use() - matches /api, /api/users, /api/anything
app.use('/api', (req, res, next) => {
  console.log('API called');
  next();
});

// app.all() - matches exactly /api/users for all methods
app.all('/api/users', (req, res) => {
  res.send('All HTTP methods');
});
```

---

## 6. What are the different HTTP methods in Express?

**Answer:**

```javascript
// GET - Retrieve data
app.get('/users', (req, res) => {
  res.json(users);
});

// POST - Create new resource
app.post('/users', (req, res) => {
  const user = req.body;
  users.push(user);
  res.status(201).json(user);
});

// PUT - Update entire resource
app.put('/users/:id', (req, res) => {
  const { id } = req.params;
  users[id] = req.body;
  res.json(users[id]);
});

// PATCH - Partial update
app.patch('/users/:id', (req, res) => {
  const { id } = req.params;
  users[id] = { ...users[id], ...req.body };
  res.json(users[id]);
});

// DELETE - Remove resource
app.delete('/users/:id', (req, res) => {
  const { id } = req.params;
  users.splice(id, 1);
  res.status(204).send();
});

// HEAD - Same as GET but no response body
app.head('/users', (req, res) => {
  res.status(200).end();
});

// OPTIONS - Describe communication options
app.options('/users', (req, res) => {
  res.set('Allow', 'GET, POST, PUT, DELETE');
  res.send();
});
```

---

## 7. How do you handle routing in Express?

**Answer:**

```javascript
// Basic routing
app.get('/users', (req, res) => {
  res.send('Get all users');
});

// Route parameters
app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  res.send(`User ${userId}`);
});

// Multiple route parameters
app.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  res.json({ userId, postId });
});

// Query parameters
app.get('/search', (req, res) => {
  const { q, limit, page } = req.query;
  res.json({ query: q, limit, page });
});

// Route with regex
app.get(/.*fly$/, (req, res) => {
  res.send('Matches butterfly, dragonfly');
});

// Router
const router = express.Router();
router.get('/users', (req, res) => res.send('Users'));
router.get('/posts', (req, res) => res.send('Posts'));
app.use('/api', router);
```

---

## 8. What is the Express Router?

**Answer:** Express Router is a mini express application that can have its own middleware and routes. It's used to create modular, mountable route handlers.

**Example:**
```javascript
// users.routes.js
const express = require('express');
const router = express.Router();

// Middleware specific to this router
router.use((req, res, next) => {
  console.log('Time: ', Date.now());
  next();
});

// Define routes
router.get('/', (req, res) => {
  res.json(users);
});

router.get('/:id', (req, res) => {
  res.json(users[req.params.id]);
});

router.post('/', (req, res) => {
  users.push(req.body);
  res.status(201).json(req.body);
});

module.exports = router;

// app.js
const usersRouter = require('./users.routes');
app.use('/api/users', usersRouter);
```

---

## 9. How do you handle errors in Express?

**Answer:**

```javascript
// Synchronous errors
app.get('/sync-error', (req, res) => {
  throw new Error('Synchronous error');
});

// Asynchronous errors
app.get('/async-error', async (req, res, next) => {
  try {
    const data = await fetchData();
    res.json(data);
  } catch (error) {
    next(error);
  }
});

// Custom error class
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

// Error handling middleware (must be last)
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const message = err.message || 'Internal Server Error';

  res.status(statusCode).json({
    success: false,
    status: statusCode,
    message: message,
    stack: process.env.NODE_ENV === 'development' ? err.stack : undefined
  });
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({
    success: false,
    message: 'Route not found'
  });
});
```

---

## 10. What is the difference between req.params, req.query, and req.body?

**Answer:**

| Property | Description | Example |
|----------|-------------|---------|
| req.params | Route parameters | /users/:id → req.params.id |
| req.query | Query string parameters | /search?q=node → req.query.q |
| req.body | Request body data | POST data → req.body |

**Example:**
```javascript
// URL: /users/123/posts/456?sort=date&order=asc
// Body: { "title": "New Post", "content": "..." }

app.post('/users/:userId/posts/:postId', (req, res) => {
  console.log(req.params);  // { userId: '123', postId: '456' }
  console.log(req.query);   // { sort: 'date', order: 'asc' }
  console.log(req.body);    // { title: 'New Post', content: '...' }

  res.json({
    params: req.params,
    query: req.query,
    body: req.body
  });
});
```

---

## 11. How do you serve static files in Express?

**Answer:**

```javascript
// Serve files from 'public' directory
app.use(express.static('public'));
// Access: http://localhost:3000/images/logo.png

// Multiple static directories
app.use(express.static('public'));
app.use(express.static('files'));

// Virtual path prefix
app.use('/static', express.static('public'));
// Access: http://localhost:3000/static/images/logo.png

// Absolute path
const path = require('path');
app.use('/static', express.static(path.join(__dirname, 'public')));

// With options
app.use(express.static('public', {
  maxAge: '1d',
  dotfiles: 'ignore',
  index: 'index.html'
}));
```

---

## 12. What are template engines and how do you use them in Express?

**Answer:** Template engines allow you to use static template files and replace variables with actual values at runtime.

**Popular Template Engines:**
- Pug (Jade)
- EJS
- Handlebars
- Mustache

**Example with EJS:**
```javascript
// Install: npm install ejs

// Set view engine
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// Render view
app.get('/profile', (req, res) => {
  res.render('profile', {
    username: 'John',
    age: 30,
    hobbies: ['reading', 'gaming']
  });
});

// views/profile.ejs
// <h1>Welcome <%= username %></h1>
// <p>Age: <%= age %></p>
// <ul>
//   <% hobbies.forEach(hobby => { %>
//     <li><%= hobby %></li>
//   <% }); %>
// </ul>
```

---

## 13. How do you implement CORS in Express?

**Answer:**

```javascript
// Install: npm install cors

const cors = require('cors');

// Enable all CORS requests
app.use(cors());

// Configure CORS
app.use(cors({
  origin: 'http://localhost:3000',
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,
  maxAge: 3600
}));

// Multiple origins
const allowedOrigins = ['http://localhost:3000', 'http://example.com'];
app.use(cors({
  origin: function(origin, callback) {
    if (!origin || allowedOrigins.indexOf(origin) !== -1) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  }
}));

// Enable CORS for specific route
app.get('/api/data', cors(), (req, res) => {
  res.json({ message: 'CORS enabled' });
});
```

---

## 14. How do you implement authentication in Express?

**Answer:**

```javascript
// Install: npm install jsonwebtoken bcryptjs

const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');

// Register
app.post('/register', async (req, res) => {
  const { username, password } = req.body;

  // Hash password
  const hashedPassword = await bcrypt.hash(password, 10);

  // Save user to database
  const user = { username, password: hashedPassword };
  users.push(user);

  res.status(201).json({ message: 'User created' });
});

// Login
app.post('/login', async (req, res) => {
  const { username, password } = req.body;

  // Find user
  const user = users.find(u => u.username === username);
  if (!user) {
    return res.status(401).json({ message: 'Invalid credentials' });
  }

  // Verify password
  const isValid = await bcrypt.compare(password, user.password);
  if (!isValid) {
    return res.status(401).json({ message: 'Invalid credentials' });
  }

  // Generate token
  const token = jwt.sign(
    { userId: user.id, username: user.username },
    'secret_key',
    { expiresIn: '1h' }
  );

  res.json({ token });
});

// Auth middleware
const authMiddleware = (req, res, next) => {
  const token = req.headers['authorization']?.split(' ')[1];

  if (!token) {
    return res.status(401).json({ message: 'No token provided' });
  }

  try {
    const decoded = jwt.verify(token, 'secret_key');
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ message: 'Invalid token' });
  }
};

// Protected route
app.get('/profile', authMiddleware, (req, res) => {
  res.json({ user: req.user });
});
```

---

## 15. How do you handle file uploads in Express?

**Answer:**

```javascript
// Install: npm install multer

const multer = require('multer');
const path = require('path');

// Configure storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    cb(null, Date.now() + path.extname(file.originalname));
  }
});

// File filter
const fileFilter = (req, file, cb) => {
  const allowedTypes = /jpeg|jpg|png|gif/;
  const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
  const mimetype = allowedTypes.test(file.mimetype);

  if (extname && mimetype) {
    cb(null, true);
  } else {
    cb(new Error('Only images allowed'));
  }
};

// Initialize upload
const upload = multer({
  storage: storage,
  limits: { fileSize: 1024 * 1024 * 5 }, // 5MB
  fileFilter: fileFilter
});

// Single file upload
app.post('/upload', upload.single('avatar'), (req, res) => {
  res.json({
    message: 'File uploaded',
    file: req.file
  });
});

// Multiple files
app.post('/uploads', upload.array('photos', 5), (req, res) => {
  res.json({
    message: 'Files uploaded',
    files: req.files
  });
});

// Multiple fields
app.post('/mixed', upload.fields([
  { name: 'avatar', maxCount: 1 },
  { name: 'gallery', maxCount: 5 }
]), (req, res) => {
  res.json({
    avatar: req.files['avatar'],
    gallery: req.files['gallery']
  });
});
```

---

## 16. What is body-parser and do you still need it?

**Answer:** Body-parser is middleware to parse incoming request bodies. As of Express 4.16+, body-parser is built-in.

```javascript
// Old way (Express < 4.16)
const bodyParser = require('body-parser');
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

// New way (Express >= 4.16)
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Options
app.use(express.json({
  limit: '10mb',
  type: 'application/json'
}));

app.use(express.urlencoded({
  extended: true,
  limit: '10mb',
  parameterLimit: 1000
}));
```

---

## 17. How do you implement logging in Express?

**Answer:**

```javascript
// Install: npm install morgan

const morgan = require('morgan');

// Predefined formats
app.use(morgan('dev'));      // Concise colored output
app.use(morgan('combined')); // Standard Apache combined log
app.use(morgan('common'));   // Standard Apache common log
app.use(morgan('short'));    // Shorter than default
app.use(morgan('tiny'));     // Minimal output

// Custom format
app.use(morgan(':method :url :status :response-time ms'));

// Log to file
const fs = require('fs');
const path = require('path');
const accessLogStream = fs.createWriteStream(
  path.join(__dirname, 'access.log'),
  { flags: 'a' }
);
app.use(morgan('combined', { stream: accessLogStream }));

// Custom logger
const logger = (req, res, next) => {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.url}`);
  next();
};
app.use(logger);
```

---

## 18. How do you implement validation in Express?

**Answer:**

```javascript
// Install: npm install express-validator

const { body, validationResult, param, query } = require('express-validator');

// Validation middleware
const validateUser = [
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 6 }),
  body('username').trim().notEmpty(),
  body('age').optional().isInt({ min: 18 })
];

// Route with validation
app.post('/users', validateUser, (req, res) => {
  const errors = validationResult(req);

  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }

  // Process valid data
  res.json({ message: 'User created', user: req.body });
});

// Custom validator
body('email').custom(async (value) => {
  const user = await User.findOne({ email: value });
  if (user) {
    throw new Error('Email already in use');
  }
});

// Sanitization
body('email').isEmail().normalizeEmail();
body('username').trim().escape();

// Param validation
app.get('/users/:id',
  param('id').isInt(),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    res.json({ userId: req.params.id });
  }
);

// Query validation
app.get('/search',
  query('q').notEmpty(),
  query('page').optional().isInt({ min: 1 }),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    res.json({ query: req.query.q });
  }
);
```

---

## 19. What is the difference between res.send(), res.json(), and res.end()?

**Answer:**

| Method | Description | Content-Type |
|--------|-------------|--------------|
| res.send() | Sends various types of responses | Auto-detected |
| res.json() | Sends JSON response | application/json |
| res.end() | Ends response without data | None |

**Example:**
```javascript
// res.send() - Auto detects type
app.get('/send-string', (req, res) => {
  res.send('Hello');  // text/html
});

app.get('/send-object', (req, res) => {
  res.send({ name: 'John' });  // application/json
});

app.get('/send-buffer', (req, res) => {
  res.send(Buffer.from('Hello'));
});

// res.json() - Always JSON
app.get('/json', (req, res) => {
  res.json({ name: 'John', age: 30 });
});

// res.end() - No body
app.get('/end', (req, res) => {
  res.status(200).end();
});

app.get('/end-with-data', (req, res) => {
  res.end('Done');  // Not recommended, use res.send()
});
```

---

## 20. How do you handle sessions in Express?

**Answer:**

```javascript
// Install: npm install express-session

const session = require('express-session');

// Configure session
app.use(session({
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: false, // Set true if using HTTPS
    httpOnly: true,
    maxAge: 1000 * 60 * 60 * 24 // 24 hours
  }
}));

// Set session data
app.post('/login', (req, res) => {
  req.session.userId = user.id;
  req.session.username = user.username;
  res.json({ message: 'Logged in' });
});

// Access session data
app.get('/profile', (req, res) => {
  if (req.session.userId) {
    res.json({
      userId: req.session.userId,
      username: req.session.username
    });
  } else {
    res.status(401).json({ message: 'Not authenticated' });
  }
});

// Destroy session
app.post('/logout', (req, res) => {
  req.session.destroy((err) => {
    if (err) {
      return res.status(500).json({ message: 'Logout failed' });
    }
    res.json({ message: 'Logged out' });
  });
});

// Session with Redis
const RedisStore = require('connect-redis')(session);
const redis = require('redis');
const redisClient = redis.createClient();

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: false
}));
```

---

## 21. How do you implement rate limiting in Express?

**Answer:**

```javascript
// Install: npm install express-rate-limit

const rateLimit = require('express-rate-limit');

// Basic rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP',
  standardHeaders: true,
  legacyHeaders: false,
});

// Apply to all requests
app.use(limiter);

// Apply to specific routes
app.use('/api/', limiter);

// Different limits for different routes
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  message: 'Too many login attempts'
});

app.post('/login', loginLimiter, (req, res) => {
  // Login logic
});

// Custom key generator
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  keyGenerator: (req) => {
    return req.headers['x-api-key'] || req.ip;
  }
});

// Skip specific conditions
const conditionalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  skip: (req) => req.headers['x-admin-key'] === 'admin-secret'
});
```

---

## 22. What is the difference between app.get() and router.get()?

**Answer:**

```javascript
// app.get() - Application-level routing
app.get('/users', (req, res) => {
  res.json(users);
});

// router.get() - Router-level routing (modular)
const userRouter = express.Router();
userRouter.get('/', (req, res) => {
  res.json(users);
});
app.use('/users', userRouter);

// Both achieve: GET /users
// But router provides better organization

// Example structure
// routes/users.js
const router = express.Router();
router.get('/', getAllUsers);
router.get('/:id', getUser);
router.post('/', createUser);
module.exports = router;

// app.js
const userRoutes = require('./routes/users');
app.use('/api/users', userRoutes);
```

---

## 23. How do you connect Express with MongoDB?

**Answer:**

```javascript
// Install: npm install mongodb mongoose

// Using Mongoose
const mongoose = require('mongoose');

// Connect
mongoose.connect('mongodb://localhost:27017/myapp', {
  useNewUrlParser: true,
  useUnifiedTopology: true
})
.then(() => console.log('MongoDB connected'))
.catch(err => console.error('Connection error:', err));

// Define Schema
const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  age: Number,
  createdAt: { type: Date, default: Date.now }
});

// Create Model
const User = mongoose.model('User', userSchema);

// Routes
app.get('/users', async (req, res) => {
  try {
    const users = await User.find();
    res.json(users);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.post('/users', async (req, res) => {
  try {
    const user = new User(req.body);
    await user.save();
    res.status(201).json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

app.get('/users/:id', async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.put('/users/:id', async (req, res) => {
  try {
    const user = await User.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true }
    );
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

app.delete('/users/:id', async (req, res) => {
  try {
    const user = await User.findByIdAndDelete(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.status(204).send();
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

---

## 24. How do you implement pagination in Express?

**Answer:**

```javascript
// Basic pagination
app.get('/users', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const skip = (page - 1) * limit;

  try {
    const users = await User.find()
      .skip(skip)
      .limit(limit);

    const total = await User.countDocuments();

    res.json({
      data: users,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit)
      }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// With sorting and filtering
app.get('/users', async (req, res) => {
  const {
    page = 1,
    limit = 10,
    sort = 'createdAt',
    order = 'desc',
    search
  } = req.query;

  const skip = (page - 1) * limit;
  const sortOrder = order === 'desc' ? -1 : 1;

  // Build query
  const query = {};
  if (search) {
    query.$or = [
      { name: { $regex: search, $options: 'i' } },
      { email: { $regex: search, $options: 'i' } }
    ];
  }

  try {
    const users = await User.find(query)
      .sort({ [sort]: sortOrder })
      .skip(skip)
      .limit(parseInt(limit));

    const total = await User.countDocuments(query);

    res.json({
      success: true,
      data: users,
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total,
        pages: Math.ceil(total / limit),
        hasMore: skip + users.length < total
      }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

---

## 25. How do you implement environment variables in Express?

**Answer:**

```javascript
// Install: npm install dotenv

// .env file
PORT=3000
NODE_ENV=development
DATABASE_URL=mongodb://localhost:27017/myapp
JWT_SECRET=your-secret-key
API_KEY=your-api-key

// Load environment variables
require('dotenv').config();

// Access variables
const port = process.env.PORT || 3000;
const dbUrl = process.env.DATABASE_URL;
const jwtSecret = process.env.JWT_SECRET;

// config/config.js
module.exports = {
  port: process.env.PORT || 3000,
  env: process.env.NODE_ENV || 'development',
  db: {
    url: process.env.DATABASE_URL,
    options: {
      useNewUrlParser: true,
      useUnifiedTopology: true
    }
  },
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: '1h'
  }
};

// Usage
const config = require('./config/config');
app.listen(config.port, () => {
  console.log(`Server running on port ${config.port}`);
});

// Different environments
// .env.development
// .env.production
// .env.test

// Load based on environment
const envFile = `.env.${process.env.NODE_ENV || 'development'}`;
require('dotenv').config({ path: envFile });
```

---

## 26. What is helmet and why should you use it?

**Answer:** Helmet helps secure Express apps by setting various HTTP headers.

```javascript
// Install: npm install helmet

const helmet = require('helmet');

// Use all default security headers
app.use(helmet());

// Configure specific options
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'", "trusted-cdn.com"]
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  }
}));

// Individual middlewares
app.use(helmet.hidePoweredBy()); // Hide X-Powered-By header
app.use(helmet.frameguard({ action: 'deny' })); // X-Frame-Options
app.use(helmet.noSniff()); // X-Content-Type-Options
app.use(helmet.xssFilter()); // X-XSS-Protection

// Headers set by Helmet:
// - Content-Security-Policy
// - X-DNS-Prefetch-Control
// - X-Frame-Options
// - Strict-Transport-Security
// - X-Download-Options
// - X-Content-Type-Options
// - X-XSS-Protection
```

---

## 27. How do you implement compression in Express?

**Answer:**

```javascript
// Install: npm install compression

const compression = require('compression');

// Use compression
app.use(compression());

// With options
app.use(compression({
  level: 6, // Compression level (0-9)
  threshold: 1024, // Min size to compress (bytes)
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  }
}));

// Compress specific routes
app.get('/data', compression(), (req, res) => {
  res.json(largeData);
});
```

---

## 28. What is the difference between synchronous and asynchronous middleware?

**Answer:**

```javascript
// Synchronous middleware
app.use((req, res, next) => {
  console.log('Sync middleware');
  next();
});

// Asynchronous middleware (callback)
app.use((req, res, next) => {
  fs.readFile('file.txt', (err, data) => {
    if (err) return next(err);
    req.fileData = data;
    next();
  });
});

// Async/await middleware (requires wrapper or error handling)
// Without wrapper - WRONG
app.use(async (req, res, next) => {
  const data = await fetchData(); // If this throws, it won't be caught
  req.data = data;
  next();
});

// With wrapper - CORRECT
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.use(asyncHandler(async (req, res, next) => {
  const data = await fetchData();
  req.data = data;
  next();
}));

// Or use try-catch
app.use(async (req, res, next) => {
  try {
    const data = await fetchData();
    req.data = data;
    next();
  } catch (error) {
    next(error);
  }
});
```

---

## 29. How do you implement WebSockets in Express?

**Answer:**

```javascript
// Install: npm install socket.io

const express = require('express');
const http = require('http');
const socketIo = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = socketIo(server, {
  cors: {
    origin: '*',
    methods: ['GET', 'POST']
  }
});

// Socket.IO connection
io.on('connection', (socket) => {
  console.log('New client connected:', socket.id);

  // Listen for events
  socket.on('message', (data) => {
    console.log('Message received:', data);

    // Emit to sender
    socket.emit('message', data);

    // Emit to all clients
    io.emit('message', data);

    // Emit to all except sender
    socket.broadcast.emit('message', data);
  });

  // Join room
  socket.on('joinRoom', (room) => {
    socket.join(room);
    socket.to(room).emit('userJoined', socket.id);
  });

  // Emit to specific room
  socket.on('roomMessage', ({ room, message }) => {
    io.to(room).emit('message', message);
  });

  // Disconnect
  socket.on('disconnect', () => {
    console.log('Client disconnected:', socket.id);
  });
});

// Express routes can access io
app.post('/notify', (req, res) => {
  io.emit('notification', req.body);
  res.json({ success: true });
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

---

## 30. How do you implement caching in Express?

**Answer:**

```javascript
// Install: npm install node-cache

const NodeCache = require('node-cache');
const cache = new NodeCache({ stdTTL: 100 }); // 100 seconds default

// Cache middleware
const cacheMiddleware = (duration) => {
  return (req, res, next) => {
    const key = req.originalUrl || req.url;
    const cachedResponse = cache.get(key);

    if (cachedResponse) {
      return res.json(cachedResponse);
    }

    res.originalJson = res.json;
    res.json = (body) => {
      cache.set(key, body, duration);
      res.originalJson(body);
    };
    next();
  };
};

// Use cache
app.get('/users', cacheMiddleware(60), async (req, res) => {
  const users = await User.find();
  res.json(users);
});

// Manual caching
app.get('/posts', async (req, res) => {
  const cacheKey = 'all_posts';
  const cached = cache.get(cacheKey);

  if (cached) {
    return res.json({ source: 'cache', data: cached });
  }

  const posts = await Post.find();
  cache.set(cacheKey, posts, 300); // Cache for 5 minutes
  res.json({ source: 'database', data: posts });
});

// Clear cache
app.post('/clear-cache', (req, res) => {
  cache.flushAll();
  res.json({ message: 'Cache cleared' });
});

// Redis caching
const redis = require('redis');
const client = redis.createClient();

const redisCacheMiddleware = (req, res, next) => {
  const key = req.originalUrl;

  client.get(key, (err, data) => {
    if (err) throw err;

    if (data) {
      return res.json(JSON.parse(data));
    }

    res.originalJson = res.json;
    res.json = (body) => {
      client.setex(key, 3600, JSON.stringify(body));
      res.originalJson(body);
    };
    next();
  });
};
```

---

## 31. How do you handle request timeout in Express?

**Answer:**

```javascript
// Install: npm install connect-timeout

const timeout = require('connect-timeout');

// Set timeout for all requests
app.use(timeout('5s'));

// Halt middleware (must be after timeout)
app.use((req, res, next) => {
  if (!req.timedout) next();
});

// Timeout handler
app.use((req, res, next) => {
  if (req.timedout) {
    return res.status(408).json({
      error: 'Request timeout'
    });
  }
  next();
});

// Manual timeout
const requestTimeout = (duration) => {
  return (req, res, next) => {
    req.setTimeout(duration, () => {
      res.status(408).json({ error: 'Request timeout' });
    });
    next();
  };
};

app.get('/slow', requestTimeout(5000), async (req, res) => {
  const data = await slowOperation();
  res.json(data);
});

// AbortController approach
app.get('/users', async (req, res) => {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), 5000);

  try {
    const users = await User.find().signal(controller.signal);
    clearTimeout(timeoutId);
    res.json(users);
  } catch (error) {
    if (error.name === 'AbortError') {
      res.status(408).json({ error: 'Request timeout' });
    } else {
      res.status(500).json({ error: error.message });
    }
  }
});
```

---

## 32. How do you implement API versioning in Express?

**Answer:**

```javascript
// Method 1: URL versioning
app.use('/api/v1/users', require('./routes/v1/users'));
app.use('/api/v2/users', require('./routes/v2/users'));

// Method 2: Header versioning
const versionMiddleware = (req, res, next) => {
  const version = req.headers['api-version'] || 'v1';
  req.apiVersion = version;
  next();
};

app.use(versionMiddleware);

app.get('/api/users', (req, res) => {
  if (req.apiVersion === 'v1') {
    // Version 1 logic
  } else if (req.apiVersion === 'v2') {
    // Version 2 logic
  }
});

// Method 3: Query parameter
app.get('/api/users', (req, res) => {
  const version = req.query.version || 'v1';
  if (version === 'v1') {
    // V1 logic
  } else if (version === 'v2') {
    // V2 logic
  }
});

// Method 4: Accept header
app.get('/api/users', (req, res) => {
  const acceptHeader = req.headers.accept;
  if (acceptHeader.includes('application/vnd.api.v1+json')) {
    // V1 response
  } else if (acceptHeader.includes('application/vnd.api.v2+json')) {
    // V2 response
  }
});

// Structured approach
// routes/api.js
const express = require('express');
const router = express.Router();

router.use('/v1', require('./v1'));
router.use('/v2', require('./v2'));

module.exports = router;

// app.js
app.use('/api', require('./routes/api'));
```

---

## 33. What are some Express best practices?

**Answer:**

**1. Project Structure:**
```
project/
├── src/
│   ├── controllers/
│   ├── models/
│   ├── routes/
│   ├── middleware/
│   ├── services/
│   ├── utils/
│   ├── config/
│   └── app.js
├── tests/
├── .env
├── .gitignore
└── package.json
```

**2. Security Best Practices:**
```javascript
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

// Use Helmet
app.use(helmet());

// Rate limiting
app.use(rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100
}));

// Input validation
// Use express-validator

// Prevent NoSQL injection
// Sanitize user input

// Use HTTPS in production
// Set secure cookies
```

**3. Error Handling:**
```javascript
// Async wrapper
const asyncHandler = fn => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

// Global error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(err.status || 500).json({
    error: {
      message: err.message,
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    }
  });
});
```

**4. Environment Configuration:**
```javascript
require('dotenv').config();
// Use environment variables
// Never commit .env files
```

**5. Logging:**
```javascript
const morgan = require('morgan');
app.use(morgan('combined'));
// Use winston for production
```

**6. Validation:**
```javascript
// Always validate input
// Use express-validator or Joi
```

**7. Database:**
```javascript
// Use connection pooling
// Handle errors properly
// Use indexes
// Implement pagination
```

---

## 34. How do you test Express applications?

**Answer:**

```javascript
// Install: npm install --save-dev jest supertest

// app.js - Export app without listening
const express = require('express');
const app = express();

app.get('/users', (req, res) => {
  res.json([{ id: 1, name: 'John' }]);
});

module.exports = app; // Export app

// server.js - Start server
const app = require('./app');
const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

// tests/app.test.js
const request = require('supertest');
const app = require('../app');

describe('GET /users', () => {
  it('should return all users', async () => {
    const response = await request(app)
      .get('/users')
      .expect(200)
      .expect('Content-Type', /json/);

    expect(response.body).toEqual([{ id: 1, name: 'John' }]);
  });
});

describe('POST /users', () => {
  it('should create a user', async () => {
    const newUser = { name: 'Jane', email: 'jane@example.com' };

    const response = await request(app)
      .post('/users')
      .send(newUser)
      .expect(201)
      .expect('Content-Type', /json/);

    expect(response.body).toHaveProperty('id');
    expect(response.body.name).toBe('Jane');
  });

  it('should return 400 for invalid data', async () => {
    const response = await request(app)
      .post('/users')
      .send({ name: '' })
      .expect(400);

    expect(response.body).toHaveProperty('error');
  });
});

// Mocking database
jest.mock('../models/User');
const User = require('../models/User');

describe('GET /users/:id', () => {
  it('should return a user', async () => {
    User.findById = jest.fn().mockResolvedValue({
      id: 1,
      name: 'John'
    });

    const response = await request(app)
      .get('/users/1')
      .expect(200);

    expect(response.body.name).toBe('John');
  });
});
```

---

## 35. How do you implement graceful shutdown in Express?

**Answer:**

```javascript
const express = require('express');
const app = express();

let server;

// Start server
const startServer = () => {
  server = app.listen(3000, () => {
    console.log('Server running on port 3000');
  });
};

// Graceful shutdown
const gracefulShutdown = (signal) => {
  console.log(`\n${signal} received`);
  console.log('Closing HTTP server...');

  server.close(() => {
    console.log('HTTP server closed');

    // Close database connections
    mongoose.connection.close(false, () => {
      console.log('MongoDB connection closed');
      process.exit(0);
    });
  });

  // Force shutdown after 10 seconds
  setTimeout(() => {
    console.error('Could not close connections in time, forcefully shutting down');
    process.exit(1);
  }, 10000);
};

// Listen for termination signals
process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
process.on('SIGINT', () => gracefulShutdown('SIGINT'));

// Handle uncaught exceptions
process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
  gracefulShutdown('uncaughtException');
});

// Handle unhandled promise rejections
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection at:', promise, 'reason:', reason);
  gracefulShutdown('unhandledRejection');
});

startServer();
```

---

## 36. How do you implement clustering in Express?

**Answer:**

```javascript
// Install: Built-in with Node.js

const cluster = require('cluster');
const os = require('os');
const express = require('express');

if (cluster.isMaster) {
  const numCPUs = os.cpus().length;
  console.log(`Master ${process.pid} is running`);
  console.log(`Forking ${numCPUs} workers`);

  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  // Restart worker if it dies
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    console.log('Starting a new worker');
    cluster.fork();
  });

} else {
  // Worker processes
  const app = express();

  app.get('/', (req, res) => {
    res.send(`Hello from worker ${process.pid}`);
  });

  app.listen(3000, () => {
    console.log(`Worker ${process.pid} started`);
  });
}

// Using PM2 (better alternative)
// Install: npm install -g pm2
// Start: pm2 start app.js -i max
// -i max uses all CPU cores
```

---

## 37. What is the difference between app.use() and app.get()?

**Answer:**

```javascript
// app.use() - Middleware, runs for all HTTP methods
app.use('/api', (req, res, next) => {
  console.log('Works for GET, POST, PUT, DELETE, etc.');
  next();
});

// app.get() - Route handler, only for GET requests
app.get('/api/users', (req, res) => {
  res.json(users);
});

// app.use() without path - runs for all routes
app.use((req, res, next) => {
  console.log('Every request');
  next();
});

// app.use() with path - runs for matching paths
app.use('/admin', (req, res, next) => {
  // Runs for /admin, /admin/users, /admin/anything
  next();
});

// app.get() - specific route only
app.get('/admin', (req, res) => {
  // Only runs for exact GET /admin
});
```

---

## 38. How do you implement request/response logging?

**Answer:**

```javascript
// Simple logger
app.use((req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.originalUrl} ${res.statusCode} ${duration}ms`);
  });

  next();
});

// Using Morgan
const morgan = require('morgan');
const fs = require('fs');
const path = require('path');

// Create write stream
const accessLogStream = fs.createWriteStream(
  path.join(__dirname, 'access.log'),
  { flags: 'a' }
);

// Setup morgan
app.use(morgan('combined', { stream: accessLogStream }));

// Custom token
morgan.token('body', (req) => JSON.stringify(req.body));
app.use(morgan(':method :url :status :body'));

// Using Winston
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

// Middleware
app.use((req, res, next) => {
  logger.info({
    method: req.method,
    url: req.url,
    ip: req.ip,
    body: req.body
  });
  next();
});
```

---

## 39. How do you handle cookies in Express?

**Answer:**

```javascript
// Install: npm install cookie-parser

const cookieParser = require('cookie-parser');

// Use cookie parser
app.use(cookieParser('secret-key')); // Optional secret for signed cookies

// Set cookie
app.get('/set-cookie', (req, res) => {
  res.cookie('username', 'john', {
    maxAge: 900000, // 15 minutes
    httpOnly: true,
    secure: true, // HTTPS only
    sameSite: 'strict'
  });
  res.send('Cookie set');
});

// Read cookie
app.get('/read-cookie', (req, res) => {
  const username = req.cookies.username;
  res.send(`Username: ${username}`);
});

// Signed cookie
app.get('/signed-cookie', (req, res) => {
  res.cookie('user', 'john', { signed: true });
  res.send('Signed cookie set');
});

app.get('/read-signed', (req, res) => {
  const user = req.signedCookies.user;
  res.send(`User: ${user}`);
});

// Delete cookie
app.get('/delete-cookie', (req, res) => {
  res.clearCookie('username');
  res.send('Cookie deleted');
});

// Cookie options
const cookieOptions = {
  httpOnly: true, // Not accessible via JavaScript
  secure: process.env.NODE_ENV === 'production', // HTTPS only
  sameSite: 'lax', // or 'strict' or 'none'
  maxAge: 24 * 60 * 60 * 1000, // 24 hours
  domain: '.example.com', // Cookie domain
  path: '/' // Cookie path
};

res.cookie('token', 'abc123', cookieOptions);
```

---

## 40. What are the differences between Express 4 and Express 5?

**Answer:**

| Feature | Express 4 | Express 5 |
|---------|-----------|-----------|
| Promise Support | Limited | Native |
| Route matching | Less strict | More strict |
| app.del() | Exists | Removed (use app.delete()) |
| req.param() | Exists | Deprecated |
| res.json() | Returns response | Returns promise |
| res.send() | Returns response | Returns promise |
| Error handling | Callback | Better async support |
| Path-to-RegExp | Version 0.x | Version 1.x |
| Query parser | qs 6.x | qs 6.5.x |
| Breaking changes | None | Several |

**Express 5 Changes:**
```javascript
// Express 4
app.del('/user', handler); // Deprecated

// Express 5
app.delete('/user', handler); // Use this

// Removed methods in Express 5:
// - app.param(fn)
// - Plural method names (e.g., app.params)

// Better async support
app.get('/async', async (req, res, next) => {
  try {
    const data = await fetchData();
    res.json(data);
  } catch (err) {
    next(err); // Still need to pass errors
  }
});

// Stricter routing
// In Express 5, these are different:
app.get('/user/:id'); // Matches /user/123
app.get('/user/:id/'); // Does NOT match /user/123 (trailing slash required)
```

---

## Additional Tips for Express Interviews:

1. **Understand middleware execution order**
2. **Know error handling patterns**
3. **Be familiar with security best practices**
4. **Understand async/await error handling**
5. **Know RESTful API principles**
6. **Understand request/response lifecycle**
7. **Be familiar with popular middleware**
8. **Know performance optimization techniques**
9. **Understand testing strategies**
10. **Practice building complete applications**

---

## Common Middleware Libraries:

- **morgan:** HTTP request logger
- **helmet:** Security headers
- **cors:** Enable CORS
- **compression:** Response compression
- **express-validator:** Input validation
- **express-rate-limit:** Rate limiting
- **multer:** File upload
- **cookie-parser:** Cookie parsing
- **express-session:** Session management
- **passport:** Authentication

---

## Resources:

- Official Documentation: https://expressjs.com
- GitHub: https://github.com/expressjs/express
- npm: https://www.npmjs.com/package/express
- Community: Stack Overflow, Reddit

