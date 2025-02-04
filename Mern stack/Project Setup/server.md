

# MERN Stack Backend Setup Guide

## 1. Project Initialization

Create a new project directory and initialize the Node.js project.

```bash
mkdir your application name
cd server
npm init -y
```

## 2. Install Dependencies

Install core dependencies and development tools:

```bash
npm install express mongoose dotenv cors helmet bcryptjs jsonwebtoken cookie-parser express-rate-limit express-mongo-sanitize
npm install --save-dev nodemon morgan eslint prettier

```

**Key Packages:**

-   **express**: Web framework.
-   **mongoose**: ODM for MongoDB.
-   **dotenv**: Environment variable management.
-   **cors**: Enable Cross-Origin Resource Sharing.
-   **helmet**: Secure HTTP headers.
-   **bcryptjs**: Password hashing.
-   **jsonwebtoken**: JWT authentication.
-   **cookie-parser**: Parse cookies.
-   **express-rate-limit**: Rate limiting.
-   **express-mongo-sanitize**: Prevent NoSQL injection.
-   **nodemon**: Auto-restart during development.
-   **morgan**: HTTP request logging.


## 3. Project Structure

Organize your project into a modular folder structure for better maintainability and scalability:

```
mern-backend/
├── config/
│   └── db.js             # Database connection logic
├── controllers/          # Route controllers (business logic)
├── middlewares/          # Custom middleware (authentication, error handling)
├── models/               # Mongoose models
├── routes/               # API route definitions
├── utils/                # Utility functions (optional)
├── .env                  # Environment variables
├── .gitignore            # Git ignore file (include .env)
├── server.js             # Main server entry point
└── package.json

```


## 4. Environment Configuration

Create a `.env` file at the root (or in a dedicated config folder) to securely store sensitive data:

```ini
NODE_ENV=development
PORT=5000
MONGO_URI=mongodb+srv://<user>:<password>@cluster0.mongodb.net/mern?retryWrites=true&w=majority
JWT_SECRET=your_jwt_secret_key
JWT_EXPIRE=30d
CLIENT_URL=http://localhost:3000

```

_Remember to add `.env` to your `.gitignore` file to prevent sensitive data from being committed._


## 5. Basic Server Setup (server.js)

Create your main entry point file (`server.js`) to initialize the Express app, set up middleware, connect to the database, define routes, and handle errors.

```javascript
// server.js
import express from 'express';
import dotenv from 'dotenv';
import cors from 'cors';
import helmet from 'helmet';
import cookieParser from 'cookie-parser';
import morgan from 'morgan';
import rateLimit from 'express-rate-limit';
import mongoSanitize from 'express-mongo-sanitize';

// Load environment variables
dotenv.config();

// Create Express app
const app = express();

// Middleware Setup
app.use(helmet());                      
app.use(mongoSanitize());               
app.use(cors({
  origin: process.env.CLIENT_URL,       
  methods: ["GET", "POST", "PUT", "DELETE"],
  credentials:  true
}));
app.use(cookieParser());
app.use(express.json());                
app.use(morgan('dev'));                 

// Rate Limiting
const limiter = rateLimit({
  windowMs: 10 * 60 * 1000,             // 10 minutes
  max: 200                            // Limit each IP to 200 requests per windowMs
});
app.use(limiter);

// Database Connection
import connectDB from './config/db.js';
connectDB();

// Routes
import authRoutes from './routes/authRoutes.js';

app.use('/api/auth', authRoutes);

// Basic Test Route
app.get('/', (req, res) => {
  res.status(200).json({ success: true, message: 'MERN Backend Server' });
});

// Custom Error Handler Middleware
import { errorHandler } from './middlewares/errorMiddleware.js';
app.use(errorHandler);

// Handle unhandled promise rejections
process.on('unhandledRejection', (err, promise) => {
  console.error(`Unhandled Rejection: ${err.message}`);
  // Close server & exit process
  server.close(() => process.exit(1));
});

// Start Server
const PORT = process.env.PORT || 5000;
const server = app.listen(PORT, () => {
  console.log(`Server running in ${process.env.NODE_ENV} mode on port ${PORT}`);
});

export default app; // for testing purposes

```


## 6. Database Connection (config/db.js)

Set up a dedicated file for MongoDB connection logic.

```javascript
// config/db.js
import mongoose from 'mongoose';

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGO_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true
    });
    console.log(`MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    console.error(`MongoDB Connection Error: ${error.message}`);
    process.exit(1);
  }
};

export default connectDB;

```


## 7. Define Models

### Example: User Model (models/User.js)

Define your data schema using Mongoose and include pre-save hooks for password encryption.

```javascript
// models/User.js
import mongoose from 'mongoose';
import bcrypt from 'bcryptjs';

const UserSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Please add a name']
  },
  email: {
    type: String,
    required: [true, 'Please add an email'],
    unique: true,
    match: [
      /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/,
      'Please add a valid email'
    ]
  },
  password: {
    type: String,
    required: [true, 'Please add a password'],
    minlength: 6,
    select: false
  }
}, { timestamps: true });

// Encrypt password before saving
UserSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  const salt = await bcrypt.genSalt(10);
  this.password = await bcrypt.hash(this.password, salt);
  next();
});

// Compare entered password with hashed password
UserSchema.methods.matchPassword = async function(enteredPassword) {
  return await bcrypt.compare(enteredPassword, this.password);
};

export default mongoose.model('User', UserSchema);

```

----------

## 8. Routes and Controllers

### A. Authentication Routes (routes/authRoutes.js)

Define the API endpoints for authentication.

```javascript
// routes/authRoutes.js
import express from 'express';
import { login, register } from '../controllers/authController.js';
const router = express.Router();

router.post('/register', register);
router.post('/login', login);

export default router;

```

### B. Authentication Controllers (controllers/authController.js)

Implement the business logic for user registration and login.

```javascript
// controllers/authController.js
import User from '../models/User.js';
import jwt from 'jsonwebtoken';

// Register a new user
export const register = async (req, res) => {
  // Your Logic
};

// Login user
export const login = async (req, res) => {
  // Your Logic
};

```

## 9. Middleware

### Error Handling Middleware (middlewares/errorMiddleware.js)

Centralize error handling to improve debugging and user feedback.

```javascript
// middlewares/errorMiddleware.js
export const errorHandler = (err, req, res, next) => {
  const statusCode = res.statusCode === 200 ? 500 : res.statusCode;
  res.status(statusCode).json({
    success: false,
    message: err.message,
    stack: process.env.NODE_ENV === 'production' ? null : err.stack,
  });
};

```
