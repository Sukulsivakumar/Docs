### Document: Step-by-Step Guide to Creating the Backend (Server) in a MERN Stack Project

---

### **Step 1: Initialize the Node.js Project**

1. **Create the Project Directory**
   - Create a new directory for your backend project and navigate into it:
     ```bash
     mkdir server
     cd server
     ```

2. **Initialize the Node.js Project**
   - Run the following command to initialize a `package.json` file:
     ```bash
     npm init -y
     ```
   - This will create a default `package.json` file with necessary metadata for the project.
   - for custom initialize run the following command:
     ```bash
     npm init
     ```

3. **Install Required Packages**
   - Install the necessary dependencies for the backend:
     ```bash
     npm install express mongoose dotenv cors bcryptjs jsonwebtoken
     npm install --save-dev nodemon
     ```
   - **Package Descriptions**:
     - `express`: A web framework for building REST APIs.
     - `mongoose`: An ODM (Object Data Modeling) library for MongoDB.
     - `dotenv`: For managing environment variables.
     - `cors`: Enables Cross-Origin Resource Sharing.
     - `bcryptjs`: Handles password hashing securely.
     - `jsonwebtoken`: Used for JWT (JSON Web Token) authentication.
     - `nodemon`: Restarts the server automatically when files are changed during development.

4. **Set Up Scripts**
   - Update the `scripts` section in `package.json` to use `nodemon` during development:
     ```json
     "scripts": {
       "start": "node server.js",
       "dev": "nodemon server.js"
     }
     ```
---

### **Step 2: Create the Main Entry Point (server.js)**

1. **File Purpose**:
   - The `server.js` file serves as the entry point of the application.
   - It initializes the Express server and sets up middleware, routes, and error handling.

2. **Code Implementation**:
   ```javascript
   // server.js
   import express from 'express';
   import dotenv from 'dotenv';
   import cors from 'cors';
   import morgan from 'morgan';
   import connectDB from './src/config/db.js'; // Database connection

   // Load environment variables
   dotenv.config();

   const app = express();

   // Middleware
   app.use(cors()); // Enable Cross-Origin Resource Sharing
   app.use(express.json()); // Parse incoming JSON requests
   app.use(morgan('dev')); // Log HTTP requests

   // Connect to MongoDB
   connectDB();

   // Routes
   import authRoutes from './src/routes/authRoutes.js';
   import userRoutes from './src/routes/userRoutes.js';
   app.use('/api/auth', authRoutes); // Authentication routes
   app.use('/api/users', userRoutes); // User-related routes

   // Error handling middleware
   import { errorHandler } from './src/middlewares/errorMiddleware.js';
   app.use(errorHandler);

   // Start the server
   const PORT = process.env.PORT || 5000;
   app.listen(PORT, () => {
     console.log(`Server running on port ${PORT}`);
   });
   ```

---

### **Step 3: Configure MongoDB Connection**

1. **File Purpose**:
   - The `db.js` file contains the logic to connect to the MongoDB database.

2. **Code Implementation**:
   ```javascript
   // src/config/db.js
   import mongoose from 'mongoose';

   const connectDB = async () => {
     try {
       const conn = await mongoose.connect(process.env.MONGO_URI, {
         useNewUrlParser: true,
         useUnifiedTopology: true,
       });
       console.log(`MongoDB Connected: ${conn.connection.host}`);
     } catch (error) {
       console.error(`Error: ${error.message}`);
       process.exit(1); // Exit process on failure
     }
   };

   export default connectDB;
   ```

3. **Setup Environment Variable**:
   - Create a `.env` file and define the `MONGO_URI` variable:
     ```
     MONGO_URI=your_mongodb_connection_string
     ```

---

### **Step 4: Define Models (Database Schema)**

1. **File Purpose**:
   - Models define the structure of data stored in MongoDB.

2. **Code Implementation**:
   ```javascript
   // src/models/User.js
   import mongoose from 'mongoose';
   import bcrypt from 'bcryptjs';

   const userSchema = mongoose.Schema({
     name: {
       type: String,
       required: true,
     },
     email: {
       type: String,
       required: true,
       unique: true,
     },
     password: {
       type: String,
       required: true,
     },
   }, { timestamps: true });

   // Encrypt password before saving
   userSchema.pre('save', async function (next) {
     if (!this.isModified('password')) {
       next();
     }
     const salt = await bcrypt.genSalt(10);
     this.password = await bcrypt.hash(this.password, salt);
   });

   // Compare entered password with hashed password
   userSchema.methods.matchPassword = async function (enteredPassword) {
     return await bcrypt.compare(enteredPassword, this.password);
   };

   export default mongoose.model('User', userSchema);
   ```

---

### **Step 5: Create Routes (API Endpoints)**

1. **File Purpose**:
   - Define the API endpoints for different functionalities.

2. **Code Implementation**:
   ```javascript
   // src/routes/authRoutes.js
   import express from 'express';
   import { login, register } from '../controllers/authController.js';
   const router = express.Router();

   router.post('/login', login); // Login endpoint
   router.post('/register', register); // Register endpoint

   export default router;
   ```

---

### **Step 6: Implement Controllers (Business Logic)**

1. **File Purpose**:
   - Controllers handle the business logic for each route.

2. **Code Implementation**:
   ```javascript
   // src/controllers/authController.js
   import User from '../models/User.js';
   import jwt from 'jsonwebtoken';

   // Register a new user
   export const register = async (req, res) => {
     const { name, email, password } = req.body;
     try {
       const userExists = await User.findOne({ email });
       if (userExists) {
         return res.status(400).json({ message: 'User already exists' });
       }
       const user = await User.create({ name, email, password });
       res.status(201).json({ message: 'User registered successfully', user });
     } catch (error) {
       res.status(500).json({ message: 'Server error', error });
     }
   };

   // Login user
   export const login = async (req, res) => {
     const { email, password } = req.body;
     try {
       const user = await User.findOne({ email });
       if (user && (await user.matchPassword(password))) {
         const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, { expiresIn: '30d' });
         res.json({ token, user });
       } else {
         res.status(401).json({ message: 'Invalid email or password' });
       }
     } catch (error) {
       res.status(500).json({ message: 'Server error', error });
     }
   };
   ```

---

### **Step 7: Add Middleware**

1. **File Purpose**:
   - Middleware is used for tasks such as authentication and error handling.

2. **Code Implementation**:
   ```javascript
   // src/middlewares/errorMiddleware.js
   export const errorHandler = (err, req, res, next) => {
     const statusCode = res.statusCode === 200 ? 500 : res.statusCode;
     res.status(statusCode).json({
       message: err.message,
       stack: process.env.NODE_ENV === 'production' ? null : err.stack,
     });
   };
   ```

---

### **Why This Approach Works for MNCs**
1. **Scalability**: Modular structure allows for scaling the application.
2. **Maintainability**: Separation of concerns makes the codebase easier to manage.
3. **Reusability**: Middleware and utility functions can be reused.
4. **Error Handling**: Centralized error handling ensures better debugging.

---

This is a complete guide for setting up the backend in a MERN stack project. Each section focuses on modular design principles followed by MNCs to ensure code quality and maintainability.

