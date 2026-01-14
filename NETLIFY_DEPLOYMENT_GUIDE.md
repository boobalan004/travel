# Netlify Deployment Guide - Rohil Travel App

## Problem
When deploying to Netlify, the frontend tries to connect to `http://localhost:5000`, which doesn't exist in production, causing a network error.

## Solution
Set the `REACT_APP_API_URL` environment variable in Netlify to point to your backend URL.

---

## Step 1: Get Your Backend URL

Before deploying, you need a live backend URL. Options:

### Option A: Heroku (Free tier available)
1. Deploy your backend (`d:\company\backend`) to Heroku
2. Get URL like: `https://your-app-name.herokuapp.com`

### Option B: Render (Free tier available)
1. Deploy to Render.com
2. Get URL like: `https://your-app-name.onrender.com`

### Option C: Other hosting
- AWS, Azure, DigitalOcean, etc.

---

## Step 2: Add Environment Variable to Netlify

### Via Netlify Dashboard:

1. Go to **Netlify** → Your Site → **Site Settings**
2. Navigate to **Build & Deploy** → **Environment**
3. Click **Edit variables**
4. Add new variable:
   - **Key:** `REACT_APP_API_URL`
   - **Value:** `https://your-backend-url.com` (e.g., `https://my-api.herokuapp.com`)
5. Click **Save**

### Via netlify.toml (in project root):

Create or update `d:\company\netlify.toml`:

```toml
[build]
  command = "cd myapp && npm run build"
  publish = "myapp/build"

[build.environment]
  REACT_APP_API_URL = "https://your-backend-url.com"
```

---

## Step 3: Redeploy on Netlify

1. After setting the environment variable, trigger a new deploy:
   - Go to **Deploys** → **Trigger deploy** → **Deploy site**
   - Or push to your GitHub branch (if connected)

2. Wait for build to complete ✅

---

## Step 4: Test the Connection

1. Go to your Netlify site URL
2. Try to register an account
3. Check browser console (F12) for errors
4. Should see successful connection to `https://your-backend-url.com/api/auth/signup`

---

## Configuration Overview

### Local Development (`.env.local`)
```
REACT_APP_API_URL=http://localhost:5000
```
- Uses local backend
- Default fallback if no env var set

### Production (Netlify)
```
REACT_APP_API_URL=https://your-backend-url.com
```
- Uses live backend URL
- Set in Netlify dashboard

### Files Involved
- `myapp/src/config/apiConfig.js` - Reads `REACT_APP_API_URL`
- `myapp/.env.local` - Local development (Git ignored)
- `myapp/.env.example` - Template (committed to Git)
- `netlify.toml` - Build configuration (optional, can use dashboard instead)

---

## Troubleshooting

### Still getting "Network Error"?

1. **Verify backend is running:**
   - Test backend URL in browser: `https://your-backend-url.com/api/auth` (should return 404 or error, not "connection refused")

2. **Check CORS settings on backend:**
   - Backend must allow requests from your Netlify URL
   - Example (Node.js/Express):
   ```javascript
   const cors = require('cors');
   app.use(cors({
     origin: ['http://localhost:3000', 'https://your-netlify-url.netlify.app'],
     credentials: true
   }));
   ```

3. **Clear Netlify cache:**
   - Go to **Site Settings** → **Clear cache and redeploy**

4. **Check environment variable:**
   - Build logs should show the variable being used
   - Go to **Deploys** → Latest deploy → **Deploy log** → Search for `REACT_APP_API_URL`

---

## Backend CORS Configuration Example

**File:** `d:\company\backend\server.js`

```javascript
const cors = require('cors');

app.use(cors({
  origin: [
    'http://localhost:3000',           // Local dev
    'https://rohil-travel.netlify.app' // Your Netlify URL
  ],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

---

## Summary

| Environment | API URL | Set In |
|---|---|---|
| Local Dev | `http://localhost:5000` | `.env.local` (auto) |
| Netlify Prod | `https://your-backend.com` | Netlify Dashboard |

**Next:** Deploy your backend, set the Netlify env var, and redeploy the frontend! ✅
