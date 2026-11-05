# healthconnect-backend
{
  "name": "healthconnect-backend",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "dotenv": "^16.4.5",
    "express": "^4.19.2",
    "pg": "^8.11.5"
  }
}
const express = require('express');
const cors = require('cors');
const { Pool } = require('pg');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json());

// Database connection
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: { rejectUnauthorized: false }
});

// GET Endpoint to test connection
app.get('/api/health', async (req, res) => {
  res.json({ status: 'API operational' });
});

// POST Endpoint: Create Request
app.post('/api/v1/requests', async (req, res) => {
  try {
    const { source, name, location, category, title, details } = req.body;
    const year = new Date().getFullYear();
    const requestId = `HC-${year}-${Math.floor(100000 + Math.random() * 900000)}`;

    const query = `
      INSERT INTO requests (id, source, name, location, category, title, details, status)
      VALUES ($1, $2, $3, $4, $5, $6, $7, 'Pending')
      RETURNING *;
    `;
    const values = [requestId, source, name || 'Community member', location, category, title, details];
    
    const result = await pool.query(query, values);
    res.status(201).json(result.rows[0]);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Database transaction failed' });
  }
});

// GET Endpoint: Fetch Requests
app.get('/api/v1/requests', async (req, res) => {
  try {
    const { rows } = await pool.query('SELECT * FROM requests ORDER BY created_at DESC');
    res.json(rows);
  } catch (err) {
    res.status(500).json({ error: 'Failed to fetch requests' });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
