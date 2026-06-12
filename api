// api/supabase.js - Serverless function to proxy Supabase requests securely

export default async function handler(req, res) {
  // Only allow GET requests
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    // Get parameters from query string
    const { table, select = '*', limit = '50000' } = req.query;

    // Validate table name (prevent SQL injection)
    if (!table || typeof table !== 'string' || !/^[a-zA-Z_][a-zA-Z0-9_]*$/.test(table)) {
      return res.status(400).json({ error: 'Invalid table name' });
    }

    // Get credentials from environment variables (hidden from frontend)
    const supabaseUrl = process.env.SUPABASE_URL;
    const supabaseKey = process.env.SUPABASE_KEY;

    if (!supabaseUrl || !supabaseKey) {
      return res.status(500).json({ error: 'Server configuration error' });
    }

    // Build the API request to Supabase
    const url = `${supabaseUrl}/rest/v1/${table}?select=${encodeURIComponent(select)}&limit=${limit}`;

    const response = await fetch(url, {
      method: 'GET',
      headers: {
        'apikey': supabaseKey,
        'Authorization': `Bearer ${supabaseKey}`,
        'Content-Type': 'application/json',
      },
    });

    // Check for errors
    if (!response.ok) {
      console.error(`Supabase error: ${response.status} ${response.statusText}`);
      return res.status(response.status).json({ 
        error: 'Failed to fetch data from database' 
      });
    }

    // Parse and return data
    const data = await response.json();
    
    // Set CORS headers to allow requests from metropolitan.wetu.com.ar
    res.setHeader('Access-Control-Allow-Origin', 'https://metropolitan.wetu.com.ar');
    res.setHeader('Access-Control-Allow-Methods', 'GET');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type');
    res.setHeader('Cache-Control', 'public, max-age=60'); // Cache for 1 minute

    return res.status(200).json(data);

  } catch (error) {
    console.error('Proxy error:', error);
    return res.status(500).json({ error: 'Internal server error' });
  }
}
