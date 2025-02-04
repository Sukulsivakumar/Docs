# MERN Client Setup Guide (Vite)

## 1. Initial Setup & Dependencies

### Create Project
```bash
npm create vite@latest client -- --template react
cd client
npm i
```

### Install Core Packages
```bash
npm install react-router-dom axios 
```

### Security Essentials
```bash
npm install dompurify react-hot-toast @sentry/react
```
### **Core Packages**

-   **`react-router-dom`** – Enables navigation and routing in React applications.
-   **`axios`** – A promise-based HTTP client for making API requests in React.

### **Security Essentials**

-   **`dompurify`** – Sanitizes HTML to prevent XSS attacks.
-   **`react-hot-toast`** – Displays lightweight, customizable toast notifications.
-   **`@sentry/react`** – Captures and monitors errors for debugging and performance tracking.
## 2. Project Structure
```
/client
├── public/
├── src/
│   ├── assets/           # Static assets
│   ├── components/       # Reusable components
│   ├── hooks/            # Custom hooks
│   ├── layouts/          # App layouts
│   ├── pages/            # Route pages
│   ├── services/         # API services
│   ├── utils/            # Utilities & validators
│   ├── App.jsx
│   ├── main.jsx
│   └── index.css
├── .env
├── .eslintrc.cjs
├── vite.config.js
└── package.json
```

## 3. Essential Configurations

### Environment Variables (.env)
```env
VITE_API_BASE_URL=http://localhost:5000/api
VITE_SENTRY_DSN=your-sentry-dsn
VITE_APP_ENV=development
```

### Vite Configuration (vite.config.js)
```javascript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    strictPort: true,
    hmr: { overlay: true}, // Show error overlay on HMR failure
    proxy: {
      '/api': {
        target: 'http://localhost:5000',
        changeOrigin: true,
        secure: false,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  },
  build: {
    sourcemap: true // For error tracking
  }
});
```


## **4. Set Up Routing in React**

In a React + Vite app, routes are typically set up in `main.jsx` or a dedicated `Router.jsx` file.

### **Basic Example: Setting Up Routes**

#### **Project Structure**

```
/src
 ├── /components
 │    ├── Home.jsx
 │    ├── About.jsx
 │    ├── Contact.jsx
 ├── App.jsx
 ├── main.jsx

```

#### **Step 1: Create Page Components**

Create `Home.jsx`, `About.jsx`, and `Contact.jsx` inside the `components` folder.

📌 **`Home.jsx`**

```jsx
const Home = () => <h1>Home Page</h1>;
export default Home;

```

📌 **`About.jsx`**

```jsx
const About = () => <h1>About Us</h1>;
export default About;

```

📌 **`Contact.jsx`**

```jsx
const Contact = () => <h1>Contact Page</h1>;
export default Contact;

```

----------

#### **Step 2: Configure Routes in `App.jsx`**

Modify `App.jsx` to include the `Routes` and `Route` components.

📌 **`App.jsx`**

```jsx
import { Routes, Route } from "react-router-dom";
import Home from "./components/Home";
import About from "./components/About";
import Contact from "./components/Contact";

const NotFound = () => <h1>404 - Page Not Found</h1>;

const App = () => {
  return (
     <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
        <Route path="*" element={<NotFound />} />
     </Routes>
  );
};

export default App;

```

----------

#### **Step 3: Add `BrowserRouter` in `main.jsx`**

📌 **`main.jsx`**

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import App from "./App";

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
	 <BrowserRouter>
	   <App />
	 </BrowserRouter>
  </React.StrictMode>
);

```

## 5. API Service Configuration

### Create src/services/api.js
```javascript
import axios from 'axios';
import DOMPurify from 'dompurify';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
    'X-Requested-With': 'XMLHttpRequest'
  }
});

// Request sanitization
api.interceptors.request.use(config => {
  if (config.data) {
    config.data = JSON.parse(DOMPurify.sanitize(JSON.stringify(config.data)));
  }
  return config;
});

//example api
export const fetchData = (userid) => api.get(`/api/student/${userid}`, { withCredentials: true });

export default api;
```

## 6. Security Measures

### Content Security Policy (public/index.html)
```html
<meta http-equiv="Content-Security-Policy" 
  content="default-src 'self';
          script-src 'self' 'unsafe-inline';
          style-src 'self' 'unsafe-inline';
          img-src 'self' data:;
          connect-src 'self' http://localhost:5000">
```
## 7. Error Monitoring Setup

### Initialize Sentry (src/main.jsx)
```javascript
import React from "react";
import ReactDOM from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import * as Sentry from '@sentry/react';
import App from "./App";

// Initialize Sentry before rendering
Sentry.init({
  dsn: import.meta.env.VITE_SENTRY_DSN,  // Your Sentry DSN from environment variables
  integrations: [new Sentry.BrowserTracing()],
  tracesSampleRate: 1.0,  // Sample rate for performance data
  environment: import.meta.env.VITE_APP_ENV  // App environment (development, production, etc.)
});

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>
);

```

## 8. Post-Setup Checklist

1. **Security Verification**
   - [ ] Enable HTTPS in production
   - [ ] Configure CORS whitelist on backend
   - [ ] Implement rate limiting
   - [ ] Set up CSRF protection

2. **Environment Setup**
   - [ ] Create `.env.production` and `.env.development`
   - [ ] Add `.env` to `.gitignore`
   - [ ] Initialize Git repository

3. **Development Standards**
   - [ ] Set up ESLint + Prettier
   - [ ] Create documentation (README.md)


### **Recommended Development Scripts:**

```json
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "preview": "vite preview",
  "lint": "eslint src --ext .js,.jsx --fix",
  "audit": "npm audit --omit=dev"
}

```

### **Explanation of Each Script:**

1.  **`dev`**:
    
    -   Command: `vite`
    -   **Purpose**: Starts the Vite development server in watch mode.
    -   **Usage**: Run this in development to spin up a local development server with hot reloading.
    -   Example:
        
        ```bash
        npm run dev
        
        ```
        
2.  **`build`**:
    
    -   Command: `vite build`
    -   **Purpose**: Builds the app for production, minifying and optimizing the files.
    -   **Usage**: This will create an optimized production build that you can deploy. The build will be placed in the `/dist` folder.
    -   Example:
        
        ```bash
        npm run build
        
        ```
        
3.  **`preview`**:
    
    -   Command: `vite preview`
    -   **Purpose**: Previews the production build locally after running `vite build`.
    -   **Usage**: This allows you to test your production build locally, making sure everything looks and functions as expected before deployment.
    -   Example:
        
        ```bash
        npm run preview
        
        ```
        
4.  **`lint`**:
    
    -   Command: `eslint src --ext .js,.jsx --fix`
    -   **Purpose**: Runs ESLint on your source files to check for coding issues, syntax errors, and other potential problems.
    -   **Usage**: This is often run during development or as part of a pre-commit hook to ensure consistent and clean code.
    -   **Note**: The `--fix` flag automatically fixes any fixable issues.
    -   Example:
        
        ```bash
        npm run lint
        
        ```
        
5.  **`audit`**:
    
    -   Command: `npm audit --omit=dev`
    -   **Purpose**: Checks your project’s dependencies for known security vulnerabilities and provides a report.
    -   **Usage**: It's important to regularly run this command to ensure your project is free from vulnerabilities. The `--omit=dev` flag excludes dev dependencies from the audit.
    -   Example:
        
        ```bash
        npm run audit
        
        ```
        

### **Additional Suggestions for Development Scripts**:

    
-   **Prettier Script** (If you're using Prettier for code formatting):
    
    ```json
    "format": "prettier --write ."
    
    ```
    
-   **Start with `eslint` and `prettier`**: You can integrate both **ESLint** and **Prettier** into a pre-commit hook using tools like **Husky** or **lint-staged**. This ensures that your code is automatically linted and formatted before it’s committed.
    

----------
