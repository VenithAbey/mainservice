# Railway Environment Variables Reference

Copy these environment variables to your Railway service's "Variables" tab.

## Required Variables

```bash
# Activate production profile
SPRING_PROFILES_ACTIVE=production

# Server port (Railway sets PORT automatically, but you can override)
PORT=8080

# JWT Configuration (REQUIRED - Generate a secure secret!)
JWT_SECRET=your-super-secret-jwt-key-minimum-256-bits-long
JWT_EXPIRATION=86400000

# CORS Configuration (Update with your Vercel frontend URL after deployment)
CORS_ALLOWED_ORIGINS=https://your-frontend.vercel.app,http://localhost:5173,http://localhost:3000
```

## Database Variables (Auto-provided by Railway)

These are **automatically available** when you add a MySQL database service to your Railway project:

- `MYSQLHOST` - Database host
- `MYSQLPORT` - Database port (usually 3306)
- `MYSQLDATABASE` - Database name
- `MYSQLUSER` - Database username
- `MYSQLPASSWORD` - Database password

**You don't need to manually set these** - the application will use them automatically via `application-production.properties`.

## Optional: Manual Database Configuration

If for some reason you need to set them explicitly:

```bash
SPRING_DATASOURCE_URL=jdbc:mysql://${MYSQLHOST}:${MYSQLPORT}/${MYSQLDATABASE}?useSSL=true&requireSSL=true&serverTimezone=UTC
SPRING_DATASOURCE_USERNAME=${MYSQLUSER}
SPRING_DATASOURCE_PASSWORD=${MYSQLPASSWORD}
```

## Generating a Secure JWT Secret

You can generate a secure JWT secret using:

**Linux/Mac:**
```bash
openssl rand -base64 32
```

**PowerShell (Windows):**
```powershell
-join ((48..57) + (65..90) + (97..122) | Get-Random -Count 32 | ForEach-Object {[char]$_})
```

**Online:**
- Use a secure random string generator (minimum 32 characters, 256 bits)
