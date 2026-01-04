# Deployment Guide: Railway (Backend) + Vercel (Frontend)

This guide will walk you through deploying your Spring Boot backend to Railway with MySQL, and your frontend to Vercel.

## Quick Checklist

### Backend (Railway) - ALL Features Enabled
- [ ] Push code to GitHub
- [ ] Create Railway account and project
- [ ] Add MySQL database service
- [ ] Deploy Spring Boot service
- [ ] **Configure ALL environment variables:**
  - [ ] `SPRING_PROFILES_ACTIVE=production` (enables production config)
  - [ ] `JWT_SECRET=<secure-random-string>` (enables JWT authentication)
  - [ ] `JWT_EXPIRATION=86400000` (optional, has default)
  - [ ] `CORS_ALLOWED_ORIGINS=<vercel-url>` (enables CORS for frontend)
- [ ] Get Railway backend URL
- [ ] Test JWT authentication (signup/login)
- [ ] Verify CORS works with frontend

### Frontend (Vercel)
- [ ] Create Vercel account
- [ ] Import GitHub repository
- [ ] Set environment variable (API URL pointing to Railway backend)
- [ ] Deploy and get Vercel URL
- [ ] Update backend CORS with Vercel URL

---

## Part 1: Deploy Backend to Railway

### Step 1: Prepare Your Repository

1. **Push your code to GitHub** (if not already done):
   ```bash
   git add .
   git commit -m "Prepare for Railway deployment"
   git push origin main
   ```

### Step 2: Set Up Railway Account

1. Go to [railway.app](https://railway.app)
2. Sign up/login with your GitHub account
3. Click **"New Project"** (or **"Create Project"**)
4. Select **"Deploy from GitHub repo"** (or **"GitHub Repo"**)
5. Choose your repository containing the `mainservice` folder
6. Railway will automatically create a service for your application

**To Deploy a Specific Branch (instead of main/master):**

After connecting your repository, Railway will deploy from the default branch (usually `main` or `master`). To deploy from a different branch:

1. **Go to your service** → **"Settings"** tab
2. Look for the **"Build"** section (separate from "Deploy")
3. Find **"Branch"** or **"Git Branch"** setting in the Build section
4. Click on it and select the branch you want to deploy (e.g., `develop`, `staging`, `feature/xyz`)
5. Railway will automatically redeploy from the selected branch

**Note**: 
- Railway will automatically redeploy whenever you push to the selected branch
- You can change the branch at any time in Settings → Build section
- Each branch deployment is independent

### Step 3: Add MySQL Database

1. In your Railway project dashboard, click **"+ New"**
2. Select **"Database"** → **"Add MySQL"**
3. Railway will automatically create a MySQL database
4. **Important**: Note down the connection details (you'll need them in the next step)

### Step 4: Configure Environment Variables

**How to find your service and add variables:**

1. **On the Railway project dashboard:**
   - You should see your project with one or more "services" (boxes/cards)
   - Each service represents a deployable component (your app, database, etc.)
   - Look for a service card that shows your repository name or "mainservice"
   - **If you just created the project**, the service might still be initializing - wait a few seconds and refresh

2. **Click on the service card** (the one that's not the MySQL database)
   - It might show "Deploying..." or have your repo name
   - This opens the service details page

3. **Alternative: If you can't find the service:**
   - Look for a **"Services"** section in the left sidebar
   - Or click on your **project name** at the top, then you'll see all services listed
   - The service should be the first one created (before the MySQL database)

4. **Once inside the service page:**
   - Look for tabs: **"Deployments"**, **"Metrics"**, **"Variables"**, **"Settings"**
   - Click the **"Variables"** tab
   - **Alternative locations**: Sometimes it's under **"Settings"** → **"Variables"** or in a sidebar menu

5. **Add the following environment variables:**

   **In the Variables tab, click "+ New Variable" or "Add Variable" button, then add each variable one by one:**
   
   **Variable 1:**
   - **Name**: `SPRING_PROFILES_ACTIVE`
   - **Value**: `production`
   - Click "Add" or "Save"
   
   **Variable 2:**
   - **Name**: `PORT`
   - **Value**: `8080`
   - Click "Add" or "Save"

   **For Database Connection**:
   
   Railway automatically provides these environment variables when you add a MySQL database:
   - `MYSQLHOST` - Database host
   - `MYSQLPORT` - Database port (usually 3306)
   - `MYSQLDATABASE` - Database name
   - `MYSQLUSER` - Database username
   - `MYSQLPASSWORD` - Database password
   
   **The application is already configured to use these variables automatically!**
   
   However, if you want to set them explicitly, you can add:
   ```
   SPRING_DATASOURCE_URL=jdbc:mysql://${MYSQLHOST}:${MYSQLPORT}/${MYSQLDATABASE}?useSSL=true&requireSSL=true&serverTimezone=UTC
   SPRING_DATASOURCE_USERNAME=${MYSQLUSER}
   SPRING_DATASOURCE_PASSWORD=${MYSQLPASSWORD}
   ```
   
   **Important**: 
   - After adding the MySQL database service, Railway automatically makes these variables available to your app service
   - Make sure your app service is in the same project as the MySQL database
   - The variables are automatically shared between services in the same Railway project

   **For JWT Authentication (REQUIRED for login/signup to work):**
   
   **Step 1: Generate a JWT Secret**
   - Open PowerShell (Windows) or Terminal (Mac/Linux)
   - **Windows PowerShell**: Run this command:
     ```powershell
     -join ((48..57) + (65..90) + (97..122) | Get-Random -Count 32 | ForEach-Object {[char]$_})
     ```
   - **Mac/Linux**: Run this command:
     ```bash
     openssl rand -base64 32
     ```
   - **Copy the generated string** (you'll need it in the next step)
   
   **Step 2: Add JWT_SECRET to Railway**
   - In Railway Variables tab, click **"+ New Variable"**
   - **Name**: `JWT_SECRET`
   - **Value**: Paste the secret you generated (should be 32+ characters)
   - Click "Add" or "Save"
   
   **Step 3: Add JWT_EXPIRATION (Optional - has default)**
   - Click **"+ New Variable"** again
   - **Name**: `JWT_EXPIRATION`
   - **Value**: `86400000` (this is 24 hours in milliseconds)
   - Click "Add" or "Save"
   
   **⚠️ IMPORTANT**: Without JWT_SECRET, authentication will fail!

   **For CORS (REQUIRED for frontend to communicate with backend):**
   
   **Step 1: Deploy your frontend to Vercel first** (see Part 2 of this guide)
   - You'll get a URL like: `https://your-app-name.vercel.app`
   
   **Step 2: Add CORS_ALLOWED_ORIGINS to Railway**
   - In Railway Variables tab, click **"+ New Variable"**
   - **Name**: `CORS_ALLOWED_ORIGINS`
   - **Value**: `https://your-app-name.vercel.app,http://localhost:5173,http://localhost:3000`
     - Replace `your-app-name.vercel.app` with your actual Vercel URL
     - Keep the localhost URLs for local development
   - Click "Add" or "Save"
   
   **⚠️ IMPORTANT**: 
   - Without correct CORS settings, your frontend will get CORS errors!
   - Make sure there are NO spaces in the value
   - Use commas to separate multiple URLs
   - Use the exact URL (including https://)

### Step 5: Verify Database Connection

**Important**: Railway automatically shares environment variables between services in the same project.

**How to find your services in Railway:**

1. **Method 1: Project Dashboard View**
   - After logging into Railway, you should see your **project** listed
   - Click on your **project name** (not a service, but the project itself)
   - You should now see the project dashboard with service cards/boxes
   - Look for:
     - One service card showing your repository name (this is your Spring Boot app)
     - Another service card showing "MySQL" or database icon (this is your database)
   - **If you don't see service cards**: Try Method 2 or 3 below

2. **Method 2: Left Sidebar**
   - Look at the **left sidebar** on Railway dashboard
   - You might see a **"Services"** menu item or section
   - Click on it to see all services listed
   - You should see at least 2 services:
     - Your app service (named after your repo)
     - MySQL database service

3. **Method 3: Direct Navigation**
   - In the top navigation, look for your **project name**
   - Click on it to open a dropdown or navigate to project view
   - Services should be listed below or in tabs

4. **Method 4: If you only see one service**
   - If you only see your app service but no MySQL:
     - Go back to Step 3 and make sure you added the MySQL database
     - Click **"+ New"** → **"Database"** → **"Add MySQL"**
   - If you only see MySQL but no app service:
     - Your app might still be deploying
     - Wait 1-2 minutes and refresh the page
     - Check if the deployment completed in the "Deployments" section

**Once you find your services:**

1. **Click on your Spring Boot app service** (the one that's NOT the MySQL database)
   - It might be named after your GitHub repository
   - Or it might show "mainservice" or similar

2. **Go to the "Variables" tab**
   - Look for tabs: **Deployments**, **Metrics**, **Variables**, **Settings**
   - Click **"Variables"**

3. **Link MySQL Database to Your App Service**
   
   **If you see a message like "Trying to connect a database? Add Variable in mainservice":**
   
   This is Railway's way of helping you connect the database! Follow these steps:
   
   **Step A: Use Railway's "Add Variable" Feature**
   1. In your **Spring Boot app service** → **"Variables"** tab
   2. Look for a button or link that says **"Add Variable"** or **"Reference Variable"** or **"Connect Database"**
   3. You might see a dropdown or list showing your MySQL database service
   4. **Click on your MySQL database service** from the list
   5. Railway will automatically add all the MySQL connection variables:
      - `MYSQLHOST`
      - `MYSQLPORT`
      - `MYSQLDATABASE`
      - `MYSQLUSER`
      - `MYSQLPASSWORD`
   
   **Alternative Method - Manual Variable Reference:**
   
   If the above doesn't work, you can manually reference the database variables:
   1. In your app service → **"Variables"** tab
   2. Click **"+ New Variable"** or **"Add Variable"**
   3. Instead of typing a value, look for a **"Reference"** or **"Link"** option
   4. Select your MySQL database service
   5. Railway will create references to the database variables
   
   **If you still don't see the MySQL variables after linking:**
   
   **Manual Setup (Last Resort):**
   1. Go to your **MySQL database service** → **"Variables"** or **"Connect"** tab
   2. Copy the connection details:
      - Host
      - Port (usually 3306)
      - Database name
      - Username
      - Password
   3. Go back to your **app service** → **"Variables"** tab
   4. Manually add each variable:
      - `MYSQLHOST` = [host value]
      - `MYSQLPORT` = `3306`
      - `MYSQLDATABASE` = [database name]
      - `MYSQLUSER` = [username]
      - `MYSQLPASSWORD` = [password]

4. **The application will automatically use these variables** thanks to the `application-production.properties` configuration

**Still can't find services?**
- Make sure you're logged into the correct Railway account
- Check if you have multiple projects - you might be in the wrong project
- Try creating a new project and starting fresh
- Railway's UI might look different - look for any clickable items that say "Service", "App", or show your repo name

### Step 6: Configure Build Settings

1. In your service settings, go to **"Settings"** tab
2. You'll see separate sections: **"Build"** and **"Deploy"**

   **In the "Build" section:**
   - **Root Directory**: Leave it **EMPTY** (since `pom.xml` is at the repository root)
     - If you previously set it to `mainservice`, clear it/leave it blank
     - Railway will look for `pom.xml` at the root level
   - Railway will auto-detect it's a Maven/Spring Boot project
   - **Build Command**: Set to: `chmod +x ./mvnw && ./mvnw clean package -DskipTests`
     - This fixes the "Permission denied" error by making mvnw executable
     - **Alternative**: Use `bash ./mvnw clean package -DskipTests` if the above doesn't work

   **In the "Deploy" section:**
   - The start command will be: `java -jar target/mainservice-0.0.1-SNAPSHOT.jar`
   - This is usually auto-detected by Railway

### Step 7: Deploy

1. Railway will automatically start building and deploying
2. Watch the build logs in the **"Deployments"** tab
3. Once deployed, Railway will provide a public URL (e.g., `https://your-app.up.railway.app`)
4. **Copy this URL** - you'll need it for your frontend!

### Step 8: Test Your Backend

1. **Test basic connectivity:**
   - Open in browser: `https://your-app.up.railway.app/api/auth/test`
   - Should return: `"Backend is running!"`
   
   **If you get "Not Found" (404 error) OR Railway's "The train has not arrived at the station" error:**
   
   This error means your service isn't running or hasn't deployed yet. Follow these steps:
   
   **Step 1: Check Deployment Status**
   - Go to Railway → Your service → **"Deployments"** tab
   - Check the latest deployment:
     - **"Building"** = Still deploying, wait a few minutes
     - **"Active"** or **"Success"** = Deployment completed
     - **"Failed"** = Deployment failed, check logs for errors
   - If it's still building, wait 2-5 minutes and refresh
   
   **Step 2: Check Application Logs**
   - Go to Railway → Your service → **"Logs"** tab (or click on the latest deployment)
   - Look for these messages:
     - ✅ **"Started MainserviceApplication"** = App is running successfully
     - ❌ **Error messages** = App failed to start, see common issues below
     - ❌ **Empty/No logs** = App might still be building
   
   **Step 3: Common Startup Errors & Fixes**
   
   **Error: "Database connection failed" or "Cannot connect to MySQL"**
   - **Fix**: Make sure MySQL database service is running
   - **Fix**: Verify MySQL variables are linked (Step 5)
   - **Fix**: Check that `SPRING_PROFILES_ACTIVE=production` is set
   
   **Error: "JWT_SECRET not found" or JWT-related errors**
   - **Fix**: Add `JWT_SECRET` environment variable (Step 4)
   - **Fix**: Make sure it's at least 32 characters long
   
   **Error: "Port already in use" or port binding errors**
   - **Fix**: Make sure `PORT` environment variable is set to `8080` (Railway usually sets this automatically)
   
   **Error: "Application failed to start" or "Bean creation error"**
   - **Fix**: Check all required environment variables are set:
     - `SPRING_PROFILES_ACTIVE=production`
     - `JWT_SECRET=<your-secret>`
     - MySQL variables (should be auto-linked)
   
   **Error: "Build failed" or Maven errors**
   - **Fix**: Check Root Directory is set correctly (empty if pom.xml is at root)
   - **Fix**: Verify Java 21 is available (Railway auto-detects)
   - **Fix**: Check build logs for specific Maven errors
   
   **Step 4: Verify Service is Running**
   - Go to Railway → Your service → **"Metrics"** tab
   - Check if there's any activity/CPU usage
   - If metrics show 0 or nothing, the service isn't running
   
   **Step 5: Restart the Service (if needed)**
   - Go to Railway → Your service → **"Deployments"** tab
   - Click **"Redeploy"** or **"Deploy"** button
   - Wait for deployment to complete
   - Check logs again after redeployment
   
   **Step 6: Check Domain/Networking**
   - Go to Railway → Your service → **"Settings"** → **"Networking"** or **"Generate Domain"**
   - Make sure a public domain is generated
   - Copy the exact domain shown
   - Try accessing that domain

2. **Test JWT Authentication (Signup):**
   ```bash
   curl -X POST https://your-app.up.railway.app/api/auth/signup \
     -H "Content-Type: application/json" \
     -d '{"username":"testuser","email":"test@example.com","password":"testpass123"}'
   ```
   - Should return a JWT token in the response
   - **If it fails**: Check that `JWT_SECRET` is set correctly

3. **Test JWT Authentication (Login):**
   ```bash
   curl -X POST https://your-app.up.railway.app/api/auth/login \
     -H "Content-Type: application/json" \
     -d '{"username":"testuser","password":"testpass123"}'
   ```
   - Should return a JWT token
   - Copy the token for next test

4. **Test Protected Endpoints (with JWT):**
   ```bash
   curl -X GET https://your-app.up.railway.app/api/your-protected-endpoint \
     -H "Authorization: Bearer YOUR_JWT_TOKEN_HERE"
   ```
   - Should work if token is valid
   - Should return 401 if token is missing/invalid

5. **Check the logs in Railway dashboard** for any errors

---

## Part 2: Deploy Frontend to Vercel

### Step 1: Prepare Your Frontend Repository

1. Ensure your frontend code is pushed to GitHub
2. Make sure your frontend has environment variables configured for the API URL

### Step 2: Set Up Vercel Account

1. Go to [vercel.com](https://vercel.com)
2. Sign up/login with your GitHub account
3. Click **"Add New..."** → **"Project"**
4. Import your frontend repository from GitHub

### Step 3: Configure Environment Variables

1. In your Vercel project settings, go to **"Settings"** → **"Environment Variables"**
2. Add your backend API URL:
   ```
   VITE_API_URL=https://your-app.up.railway.app
   ```
   (or `REACT_APP_API_URL` for Create React App, or `NEXT_PUBLIC_API_URL` for Next.js)

### Step 4: Configure Build Settings

1. Vercel will auto-detect most frameworks (React, Next.js, Vue, etc.)
2. If needed, manually set:
   - **Framework Preset**: (e.g., Vite, Create React App, Next.js)
   - **Build Command**: (usually auto-detected)
   - **Output Directory**: (usually `dist` or `build`)

### Step 5: Deploy

1. Click **"Deploy"**
2. Vercel will build and deploy your frontend
3. Once deployed, you'll get a URL like `https://your-frontend.vercel.app`
4. **Copy this URL**

### Step 6: Update Backend CORS

1. Go back to Railway dashboard
2. Navigate to your service → **"Variables"**
3. Update `CORS_ALLOWED_ORIGINS` to include your Vercel URL:
   ```
   CORS_ALLOWED_ORIGINS=https://your-frontend.vercel.app,http://localhost:5173,http://localhost:3000
   ```
4. Railway will automatically redeploy with the new CORS settings

---

## Part 3: Update Frontend API Configuration

1. In your frontend repository, ensure your API calls use the environment variable:
   ```javascript
   const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:8080';
   ```

2. Commit and push the changes:
   ```bash
   git add .
   git commit -m "Update API URL for production"
   git push
   ```

3. Vercel will automatically redeploy

---

## Troubleshooting

### Backend Issues

1. **Can't Find Services in Railway**:
   
   **What you should see:**
   - After creating a project and adding MySQL, you should see 2 services
   - One for your app (Spring Boot)
   - One for MySQL database
   
   **Where to look:**
   - **Main Dashboard**: Click on your project name → Services should appear as cards/boxes
   - **Left Sidebar**: Look for "Services" menu item
   - **Top Navigation**: Click project name dropdown → Services listed there
   
   **If you see nothing:**
   - Wait 30-60 seconds and refresh (Railway might still be initializing)
   - Make sure you completed Step 2 (deploy from GitHub) and Step 3 (add MySQL)
   - Check if you're in the correct project (you might have multiple projects)
   - Try logging out and back into Railway
   
   **If you only see one service:**
   - Only app service? → Go back and add MySQL database (Step 3)
   - Only MySQL? → Your app might still be deploying, wait and refresh
   
   **Alternative method to add variables:**
   - Even if you can't see services clearly, try this:
   - Go to Railway dashboard
   - Look for any clickable item with your repo name or "mainservice"
   - Click it → Look for "Variables" or "Environment" tab
   - Add variables there

2. **"The train has not arrived at the station" Error (Railway-specific)**:
   - **This means**: Your service isn't running or hasn't deployed yet
   - **Solution 1**: Check deployment status in Railway → Deployments tab
     - If status is "Building", wait for it to complete
     - If status is "Failed", check logs for errors
   - **Solution 2**: Check application logs for startup errors
     - Look for "Started MainserviceApplication" = app is running
     - Look for error messages = app failed to start
   - **Solution 3**: Verify all environment variables are set:
     - `SPRING_PROFILES_ACTIVE=production` (REQUIRED)
     - `JWT_SECRET` (REQUIRED for JWT to work)
     - MySQL variables (should be auto-linked)
   - **Solution 4**: Make sure MySQL database service is running and linked
   - **Solution 5**: Try redeploying - Go to Deployments → Click "Redeploy"
   - **Solution 6**: Check that a public domain is generated in Settings → Networking

3. **Database Connection Errors**:
   - Verify all MySQL environment variables are set correctly
   - Check that the database is linked to your service
   - Ensure `SPRING_PROFILES_ACTIVE=production` is set

3. **Root Directory Error**:
   - **If `pom.xml` is at the repository root** (most common):
     - **Solution**: Leave Root Directory **EMPTY/BLANK**
     - Railway will automatically find `pom.xml` at the root
   - **If `pom.xml` is in a subfolder** (e.g., `mainservice/pom.xml`):
     - Set Root Directory to the folder name (e.g., `mainservice`)
     - Make sure the folder name matches exactly (case-sensitive)
   - **How to check**: Look at your GitHub repository - where is `pom.xml` located?
     - Root level: `your-repo/pom.xml` → Leave Root Directory empty
     - In subfolder: `your-repo/mainservice/pom.xml` → Use `mainservice`

4. **Build Failures**:
   - Check build logs in Railway
   - Ensure Java 21 is available (Railway auto-detects)
   - Verify `pom.xml` is correct
   - Check that Root Directory is set correctly (see above)

4. **JWT Authentication Not Working**:
   - Verify `JWT_SECRET` is set and is at least 32 characters long
   - Check that `SPRING_PROFILES_ACTIVE=production` is set
   - Ensure `JWT_EXPIRATION` is set (default: 86400000 = 24 hours)
   - Check Railway logs for JWT-related errors
   - Test with: `curl -X POST https://your-app.up.railway.app/api/auth/login -H "Content-Type: application/json" -d '{"username":"test","password":"test"}'`

5. **CORS Errors**:
   - Ensure `CORS_ALLOWED_ORIGINS` includes your Vercel URL (exact match, no trailing slash)
   - Check browser console for specific CORS error messages
   - Verify `SecurityConfig.java` is reading the environment variable correctly
   - Make sure the frontend URL in CORS matches exactly (including https://)
   - Test CORS with browser DevTools → Network tab → check response headers for `Access-Control-Allow-Origin`

### Frontend Issues

1. **API Connection Errors**:
   - Verify `VITE_API_URL` (or equivalent) is set in Vercel
   - Check browser console for CORS errors
   - Ensure the backend URL is correct (no trailing slash)

2. **Build Failures**:
   - Check Vercel build logs
   - Verify all dependencies are in `package.json`
   - Ensure Node.js version is compatible

---

## Feature Verification Checklist

After deployment, verify that ALL features are working:

### ✅ Database Connection
- [ ] Application starts without database errors
- [ ] Check Railway logs - should see "Started MainserviceApplication"
- [ ] Database tables are created automatically (check MySQL service in Railway)

### ✅ JWT Authentication
- [ ] **Signup works**: `POST /api/auth/signup` returns JWT token
- [ ] **Login works**: `POST /api/auth/login` returns JWT token
- [ ] **Token validation**: Protected endpoints require valid JWT token
- [ ] **Token expiration**: Expired tokens are rejected
- [ ] Check that `JWT_SECRET` environment variable is set

### ✅ CORS Configuration
- [ ] Frontend can make requests to backend (no CORS errors in browser console)
- [ ] Preflight OPTIONS requests work
- [ ] Check browser DevTools → Network → Response headers include `Access-Control-Allow-Origin`
- [ ] Verify `CORS_ALLOWED_ORIGINS` includes your Vercel frontend URL

### ✅ Security Features
- [ ] Unauthenticated requests to protected endpoints return 401
- [ ] `/api/auth/**` endpoints are publicly accessible (no auth required)
- [ ] Passwords are hashed (BCrypt) - check database, passwords should be hashed
- [ ] CSRF is disabled (for stateless JWT auth)
- [ ] Sessions are stateless (no session cookies)

### ✅ API Endpoints
- [ ] `GET /api/auth/test` - Returns "Backend is running!"
- [ ] `POST /api/auth/signup` - Creates user and returns JWT
- [ ] `POST /api/auth/login` - Authenticates and returns JWT
- [ ] Protected endpoints require `Authorization: Bearer <token>` header

### ✅ Production Configuration
- [ ] `SPRING_PROFILES_ACTIVE=production` is set
- [ ] Database uses SSL (`useSSL=true&requireSSL=true`)
- [ ] SQL logging is disabled (`spring.jpa.show-sql=false`)
- [ ] Application logs are at INFO level

---

## Quick Reference

### Railway Environment Variables
```
SPRING_PROFILES_ACTIVE=production
PORT=8080
SPRING_DATASOURCE_URL=${MYSQL_URL}
SPRING_DATASOURCE_USERNAME=${MYSQL_USER}
SPRING_DATASOURCE_PASSWORD=${MYSQL_PASSWORD}
JWT_SECRET=your-secret-key
JWT_EXPIRATION=86400000
CORS_ALLOWED_ORIGINS=https://your-frontend.vercel.app,http://localhost:5173
```

### Vercel Environment Variables
```
VITE_API_URL=https://your-app.up.railway.app
```

---

## Next Steps

1. Set up custom domains (optional) in both Railway and Vercel
2. Configure SSL certificates (automatically handled by both platforms)
3. Set up monitoring and logging
4. Configure database backups in Railway

---

**Need Help?**
- Railway Docs: https://docs.railway.app
- Vercel Docs: https://vercel.com/docs
