# How to Deploy a Specific Branch to Railway

This guide explains how to deploy a specific Git branch (instead of the default `main` branch) to Railway.

## Method 1: During Initial Setup

When first connecting your repository to Railway:

1. Connect your GitHub repository as usual
2. Railway will deploy from the default branch (`main` or `master`)
3. After the service is created, go to **Settings** tab
4. Look for the **"Build"** section (separate from "Deploy")
5. Find the **"Branch"** or **"Git Branch"** dropdown in the Build section
6. Select your desired branch (e.g., `develop`, `staging`, `feature/xyz`)
7. Railway will automatically redeploy from the selected branch

## Method 2: Change Branch for Existing Service

If you already have a service deployed and want to switch branches:

1. Go to your Railway project
2. Click on your **service** (Spring Boot app)
3. Go to **"Settings"** tab
4. Look for the **"Build"** section (you'll see "Build" and "Deploy" as separate sections)
5. Find **"Branch"** or **"Git Branch"** setting in the Build section
6. Click the dropdown and select your desired branch
7. Railway will automatically trigger a new deployment from that branch

## Method 3: Using Railway CLI (Advanced)

If you have Railway CLI installed:

```bash
# Login to Railway
railway login

# Link to your project
railway link

# Set the branch
railway variables set RAILWAY_GIT_BRANCH=your-branch-name

# Or deploy directly from a branch
railway up --branch your-branch-name
```

## Important Notes

### Branch Selection
- **Available branches**: Railway shows all branches from your connected repository
- **Default branch**: Usually `main` or `master` (depends on your repo settings)
- **Feature branches**: You can deploy any branch, including feature branches

### Automatic Deployments
- Railway automatically redeploys when you push to the selected branch
- Each push triggers a new build and deployment
- You can see deployment status in the **"Deployments"** tab

### Multiple Environments
If you want to deploy multiple branches simultaneously:

1. **Option A: Multiple Services**
   - Create separate services for each branch
   - Each service can point to a different branch
   - Useful for staging/production environments

2. **Option B: Single Service, Switch Branches**
   - Use one service and switch branches as needed
   - Only one branch is deployed at a time

### Environment Variables per Branch
- Environment variables are shared across all branch deployments
- If you need different variables per branch, use separate services

## Example: Deploying a `develop` Branch

1. **Push your `develop` branch to GitHub:**
   ```bash
   git checkout develop
   git push origin develop
   ```

2. **In Railway:**
   - Go to your service → Settings → Build & Deploy
   - Change Branch from `main` to `develop`
   - Railway will automatically deploy from `develop`

3. **Verify deployment:**
   - Check the Deployments tab
   - You should see a new deployment from the `develop` branch
   - The logs will show commits from `develop`

## Troubleshooting

### Branch Not Showing?
- Make sure the branch exists in your GitHub repository
- Push the branch to GitHub if it's only local
- Refresh the Railway page
- Check that Railway has access to your repository

### Deployment Fails?
- Check that the branch has the same structure as `main`
- Verify the `mainservice` folder exists in the branch
- Check build logs for errors
- Ensure `pom.xml` is present in the branch

### Want to Deploy Multiple Branches?
- Create separate Railway services for each branch
- Each service can have its own environment variables
- Useful for: `production` (main), `staging` (develop), `testing` (test branch)

## Quick Reference

**To change branch:**
1. Service → Settings → Build section → Branch → Select branch

**To see current branch:**
- Check the Deployments tab - it shows which branch/commit is deployed

**To deploy a new branch:**
1. Push branch to GitHub
2. Go to Railway service settings
3. Select the branch from dropdown
4. Railway auto-deploys
