# JoonWeb Node.js SDK

Official Node.js SDK for building apps on the JoonWeb platform. This SDK provides a Shopify-compatible interface for easy integration.

## 🚀 Features

- **REST API** - Familiar interface for developers
- **OAuth 2.0 Authentication** - Secure app installation flow
- **Complete Resource Coverage** - Products, Orders, Customers, Webhooks, etc.
- **TypeScript-ready** - Full type definitions included
- **Embedded App Support** - Works with JoonWeb App Bridge
- **Webhook Validation** - Secure webhook processing

## 📦 Installation

```bash
npm install joonweb-node-sdk
# or
yarn add joonweb-node-sdk
```

## 🔧 Quick Start

# 1. Initialize the SDK
```javascript
const { JoonWebAPI, Context } = require('joonweb-node-sdk');

// Initialize with your app credentials
Context.init({
    api_key: process.env.JOONWEB_CLIENT_ID,
    api_secret: process.env.JOONWEB_CLIENT_SECRET,
    api_version: '26.0'
});
```

# 2. OAuth Installation Flow
```javascript
const { OAuth } = require('joonweb-node-sdk');

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
```

# 3. Use the API
```javascript
const api = new JoonWebAPI('access_token', 'yoursite.myjoonweb.com');

// Get products
const products = await api.product.all({ limit: 10 });

// Create order
const order = await api.order.create({
    line_items: [{
        variant_id: 123456,
        quantity: 1
    }]
});

// Get site info
const site = await api.site.get();
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
JOONWEB_CLIENT_ID=your_client_id
JOONWEB_CLIENT_SECRET=your_client_secret
JOONWEB_REDIRECT_URI=https://yourapp.com/auth/callback
JOONWEB_API_VERSION=26.0
APP_URL=https://yourapp.com
```







