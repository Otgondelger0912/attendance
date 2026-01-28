# Render Deployment Guide

## Prerequisites
- GitHub account connected to Render
- MongoDB Atlas URI (already configured in .env)
- Render account

## Backend Deployment (Express Server)

### Step 1: Create Backend Service on Render
1. Go to https://dashboard.render.com
2. Click "New +" > "Web Service"
3. Connect your GitHub repository
4. Configure:
   - **Name:** `attendance-backend`
   - **Root Directory:** `attendance/server`
   - **Environment:** `Node`
   - **Build Command:** `npm install`
   - **Start Command:** `npm start`

### Step 2: Set Environment Variables
Add these in Render dashboard (Settings > Environment):
```
NODE_ENV=production
MONGODB_URI=[Your MongoDB Atlas URI]
JWT_SECRET=[Your secure JWT secret - generate a strong one]
PORT=10000  (Render assigns automatically, but good to set)
FRONTEND_URL=https://your-frontend-render-url.onrender.com
```

### Step 3: Deploy
- Render automatically deploys when you push to GitHub
- Backend will be available at: `https://your-service-name.onrender.com`
- Copy this URL for frontend configuration

---

## Frontend Deployment (React App)

### Step 1: Create Frontend Service on Render
1. Click "New +" > "Web Service"
2. Connect your GitHub repository
3. Configure:
   - **Name:** `attendance-frontend`
   - **Root Directory:** `attendance`
   - **Environment:** `Node`
   - **Build Command:** `npm install && npm run build`
   - **Start Command:** `npm start` (uses react-scripts)
   - **Publish Directory:** `build`

### Step 2: Set Environment Variables
```
REACT_APP_API_URL=https://your-backend-render-url.onrender.com/api
NODE_ENV=production
```

### Step 3: Deploy
- Frontend will be at: `https://your-frontend-render-url.onrender.com`
- Update backend's `FRONTEND_URL` with this URL

---

## Alternative: Deploy Everything on One Service

Create a root-level `render.yaml`:
```yaml
services:
  - type: web
    name: attendance-app
    env: node
    buildCommand: |
      npm install && \
      cd attendance && npm install && npm run build && cd .. && \
      cd attendance/server && npm install && cd ../..
    startCommand: |
      cd attendance/server && npm start
```

---

## After Deployment

1. **Test Backend:**
   ```
   https://your-backend-url.onrender.com/api/health
   ```
   Should return: `{ "success": true, "message": "Server is running" }`

2. **Test Frontend:**
   - Access at `https://your-frontend-url.onrender.com`
   - Try registration/login
   - Check browser console for any CORS errors

3. **Monitor Logs:**
   - Render dashboard shows real-time logs
   - Check for MongoDB connection errors
   - Watch for CORS issues

---

## Common Issues & Fixes

### Issue: "Cannot connect to database"
- Verify MongoDB Atlas IP whitelist includes Render IP: `0.0.0.0/0` (or add Render's IP)
- Check `MONGODB_URI` is correctly set

### Issue: CORS errors in frontend
- Verify `FRONTEND_URL` env var is set on backend
- Ensure frontend's `REACT_APP_API_URL` is correct
- Check browser console for exact error

### Issue: 503 Service Unavailable
- Backend still building/restarting
- Check deployment logs
- May need to increase build timeout

### Issue: Static files not loading
- Frontend build directory must be configured
- Ensure `npm run build` completes successfully

---

## Estimated Costs
- **Backend:** ~$7/month (Web Service)
- **Frontend:** ~$7/month (Web Service)  
- **MongoDB:** $0-57/month (depends on usage tier)

**Total:** ~$14/month for basic deployment

---

## Production Checklist
- [ ] Set strong JWT_SECRET (not the placeholder)
- [ ] Enable MongoDB Atlas encryption
- [ ] Set proper FRONTEND_URL and MONGODB_URI
- [ ] Test all authentication flows
- [ ] Check attendance marking functionality
- [ ] Verify file exports work
- [ ] Test on multiple browsers
- [ ] Monitor error logs daily
