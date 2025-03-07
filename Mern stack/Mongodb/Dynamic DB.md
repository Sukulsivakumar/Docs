To implement databases based on the fiscal year (June 1 to May 31) with dynamic naming like `2025_2026`, follow these steps:

### Step 1: Create Fiscal Year Utility Function
Add a function to determine the current fiscal year.

```javascript
// config/connectDataBase.js
const getCurrentFiscalYear = (date = new Date()) => {
  const year = date.getFullYear();
  const month = date.getMonth(); // 0-11 (Jan-Dec)
  return month >= 5 ? `${year}_${year + 1}` : `${year - 1}_${year}`;
};
```

### Step 2: Set Up Dynamic Yearly Database Connections
Create a base connection for yearly databases and functions to access them.

```javascript
// config/connectDataBase.js
const mongoose = require('mongoose');

const connectDataBase = (dbURI, dbName) => {
  const connection = mongoose.createConnection(dbURI, {
    dbName,
    useNewUrlParser: true,
    useUnifiedTopology: true,
  });

  // Connection event handlers
  connection.on('connected', () => console.log(`Connected to ${dbName} database`));
  connection.on('error', (err) => console.error(`Connection error on ${dbName}:`, err));
  connection.on('disconnected', () => console.log(`Disconnected from ${dbName}`));

  return connection;
};

// Base connection for yearly databases
const yearlyDBBaseConnection = connectDataBase(process.env.DB_YEARLY_URI, 'yearly_base');

// Get current fiscal year's database
const getYearlyDB = () => yearlyDBBaseConnection.useDb(getCurrentFiscalYear());

// Get specific fiscal year's database
const getYearlyDBByYear = (fiscalYear) => yearlyDBBaseConnection.useDb(fiscalYear);

module.exports = {
  // Existing databases
  eGovernanceDB: connectDataBase(process.env.DB_EGOVERNANCE, 'egovernance'),
  campusKtrDB: connectDataBase(process.env.DB_CAMPUS_KTR, 'campus_ktr'),
  // ... other campuses ...
  feedbackDB: connectDataBase(process.env.DB_FEEDBACK, 'feedback'),

  // Yearly databases
  yearlyDBBaseConnection,
  getYearlyDB,
  getYearlyDBByYear,
};
```

### Step 3: Update Server Shutdown Logic
Ensure the yearly database connection is closed gracefully.

```javascript
// server.js
const {
  // Existing connections...
  yearlyDBBaseConnection,
} = require('./config/connectDataBase');

const closeConnections = async () => {
  try {
    await Promise.all([
      // Close existing connections...
      eGovernanceDB.close(),
      campusKtrDB.close(),
      // ... other campuses ...
      yearlyDBBaseConnection.close(),
    ]);
    console.log('All connections closed.');
  } catch (error) {
    console.error('Error closing connections:', error);
  }
};
```

### Step 4: Use Dynamic Databases in Your Application
Access the current or specific fiscal year's database in your models/controllers.

```javascript
// Example usage in a controller
const { getYearlyDB, getYearlyDBByYear } = require('../config/connectDataBase');

// Access current fiscal year's database
const currentYearlyDB = getYearlyDB();
const Report = currentYearlyDB.model('Report', reportSchema);

// Access a specific fiscal year's database
const pastYearlyDB = getYearlyDBByYear('2023_2024');
const PastReport = pastYearlyDB.model('Report', reportSchema);
```

### Step 5: Environment Variables
Add the new MongoDB URI for yearly databases to your environment.

```env
# .env
DB_YEARLY_URI=mongodb://your-mongo-uri/yearly
```

### Explanation:
- **Fiscal Year Calculation**: The `getCurrentFiscalYear` function determines the database name based on the current date.
- **Dynamic Connections**: `yearlyDBBaseConnection` is a base connection reused for all yearly databases. `useDb()` switches the database without creating new connections.
- **Model Registration**: Models are registered on-demand for each fiscal year's database, ensuring indexes and schemas are applied when first accessed.
- **Graceful Shutdown**: The yearly database connection is closed along with others during shutdown.

This setup automatically switches to the new fiscal year's database on June 1 and allows access to historical data by specifying the fiscal year.
