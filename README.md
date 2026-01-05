# JoonWeb Node.js SDK

Official Node.js SDK for building apps on the JoonWeb platform. This SDK provides a user friendly interface for easy integration.

## 🚀 Features

- **REST API** - Familiar interface for developers
- **OAuth 2.0 Authentication** - Secure app installation flow
- **Complete Resource Coverage** - Products, Orders, Customers, Webhooks, etc.
- **TypeScript-ready** - Full type definitions included
- **Embedded App Support** - Works with JoonWeb App Bridge
- **Webhook Validation** - Secure webhook processing

## 📦 Installation

```bash
npm install @joonweb/joonweb-sdk
# or
yarn add @joonweb/joonweb-sdk
```

## 🔧 Quick Start

# 1. Initialize the SDK
```javascript
const { JoonWebAPI, Context } = require('@joonweb/joonweb-sdk'); // Import Context

// Initialize with your app credentials
// Initialize JoonWeb Context BEFORE any routes
Context.init({
    api_key: process.env.JOONWEB_CLIENT_ID,
    api_secret: process.env.JOONWEB_CLIENT_SECRET,
    api_version: process.env.JOONWEB_API_VERSION || '26.0',
    is_embedded: true,
    
    // Add storage configuration here
    session_storage_type: 'memory', // or 'mysql', 'sqlite', 'mongodb', 'redis', 'postgresql', 'memory'
    session_storage_options: {
        // Options specific to the chosen storage type
        // For Mongodb, Redis:
        url:'' //mongodb url,
        dbName:'your-database-name',
        // FOR MySQL, PostgreSQL, MariaDB:
        host:'',
        port:'',
        user:'',
        password:'',
        database:'',
        // SQLLITE:
        dbPath:''
        
    }
});
```

# 2. OAuth Installation Flow
```javascript
const { OAuth } = require('joonweb-sdk');

const oauth = new OAuth({
    clientId: process.env.JOONWEB_CLIENT_ID,
    clientSecret: process.env.JOONWEB_CLIENT_SECRET,
    redirectUri: 'https://yourapp.com/auth/callback',
    scopes: ['read_products', 'write_orders']
});

// Generate installation URL
const auth = oauth.beginAuth('store.myjoonweb.com');
res.redirect(auth.url);

// Handle callback
app.get('/auth/callback', async (req, res) => {
    await oauth.handleCallback(req, res);
});

// OR:

// Callback route - Enhanced to save session to storage
router.get('/callback', async (req, res) => {
    try {
        const sessionManager = req.sessionManager; // Get from middleware
        
    
        const site = req.query.site;
        if (!site) {
            throw new Error('Site not found in session');
        }
        
        // Use SDK to exchange code for token
        const tokenData = await oauth.exchangeCodeForToken(
            req.query.code,
            site
        );
        
        
        // Store session using the pluggable storage
        const session = await sessionManager.startSession(site, {
            access_token: tokenData.access_token,
            scope: tokenData.scope,
            expires_in: tokenData.expires_in,
            associated_user: tokenData.associated_user || null
        });
        
        // Store additional info in HTTP session
        req.session.joonweb_site = site;
        req.session.joonweb_access_token = tokenData.access_token;
        req.session.joonweb_user = tokenData.associated_user;

         // Build embed URL
        const embedUrl = oauth.getEmbedUrl(site, req.query.site_hash, req.query.app_slug);
        // Redirect to embedded app
        res.redirect(embedUrl);
        
        
    } catch (error) {
        console.error('OAuth callback error:', error);
        
        // Clean up on error
        if (req.session.site && req.sessionManager) {
            await req.sessionManager.destroySession(req.session.site);
        }
        
        res.status(400).render('error', { 
            message: 'Installation failed',
            error: error.message 
        });
    }
});
```

# 3. Use the API
```javascript
const api = new JoonWebAPI(accessToken, site);

// Get all products
const products = await api.product.all();

// Create a new product
const newProduct = await api.product.create({
  title: 'My Product',
  price: 19.99
});

// Get site info
const siteInfo = await api.site.get();
```

## 🔐 Authentication
# OAuth Flow

Redirect to Install: GET /auth?site=store.myjoonweb.com

JoonWeb OAuth: User approves permissions

Callback: GET /auth/callback?code=xxx&site=xxx

Exchange Code: SDK exchanges code for access token

Embed App: Redirects to JoonWeb Admin

# Webhook Validation
```javascript
app.post('/webhooks', express.raw({type: 'application/json'}), (req, res) => {
    const isValid = oauth.verifyHmac(req.body, req.headers['x-joonweb-hmac-sha256']);
    if (isValid) {
        // Process webhook
    }
});
```
## 📝 Environment Variables
```env
APP_NAME="YourAPP"
JOONWEB_CLIENT_ID=your_client_id
JOONWEB_CLIENT_SECRET=your_client_secret
JOONWEB_REDIRECT_URI=https://yourapp.com/auth/callback
JOONWEB_API_VERSION=26.0
APP_URL=https://yourapp.com
```







