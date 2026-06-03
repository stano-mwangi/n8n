# Deploying n8n to Render (Free Tier - Web Service)

This guide uses Render's free Web Service tier for cost-free hosting.

## Prerequisites

- A Render.com account (https://render.com) - free tier available
- A GitHub repository with this n8n code
- Node.js 22.x+ and pnpm 10.x+ locally (for testing)

## Free Tier Limitations

- **CPU**: Shared
- **Memory**: 0.5GB (limited)
- **Disk**: 0GB persistent (data lost on redeploy)
- **Auto-sleep**: Service sleeps after 15 min of inactivity
- **Build time**: Limited to 30 minutes

⚠️ **For data persistence on free tier**: Use PostgreSQL (Render offers free tier) instead of SQLite

---

## Deployment Steps

### 1. Update n8n Configuration for Free Tier

The free tier has limited memory, so we need to optimize the build process. Update your `packages/cli/package.json` to handle low memory:

```json
{
  "scripts": {
    "build": "NODE_OPTIONS=--max-old-space-size=512 turbo run build"
  }
}
```

### 2. Create Render Web Service

1. **Go to Render Dashboard**
   - https://dashboard.render.com

2. **Click "New +"**
   - Select **"Web Service"** (not Blueprint)

3. **Connect GitHub Repository**
   - Select "Connect a repository"
   - Choose your n8n fork
   - Click "Connect"

4. **Configure Web Service**

   | Setting | Value |
   |---------|-------|
   | **Name** | n8n |
   | **Environment** | Node |
   | **Build Command** | `pnpm install && NODE_OPTIONS=--max-old-space-size=512 pnpm build:deploy` |
   | **Start Command** | `node packages/cli/bin/n8n` |
   | **Plan** | Free |

5. **Add Environment Variables**
   
   Click "Advanced" → "Add Environment Variable" for each:

   ```
   NODE_ENV = production
   NODE_OPTIONS = --max-old-space-size=512
   DB_TYPE = sqlite
   DB_SQLITE_PATH = /data/database.sqlite
   N8N_SECURE_COOKIE = true
   N8N_ENCRYPTION_KEY = [generate random 32-char string]
   N8N_USER_MANAGEMENT_JWT_SECRET = [generate random 32-char string]
   ```

   **Generate encryption keys:**
   ```bash
   node -e "console.log(require('crypto').randomBytes(16).toString('hex'))"
   ```

6. **Deploy**
   - Click "Create Web Service"
   - Build will start (~10-15 minutes)

---

## Using PostgreSQL for Data Persistence

Free tier doesn't have persistent disk. **Better option**: Use Render's **free PostgreSQL database**

### 1. Create PostgreSQL Database

1. Go to Render Dashboard
2. Click "New +"
3. Select **"PostgreSQL"**
4. Configure:
   - **Name**: n8n-db
   - **Database**: n8n
   - **User**: n8n_user
   - **Region**: Same as your Web Service
   - **Plan**: Free

5. Note the connection details provided

### 2. Update Web Service Environment Variables

Replace SQLite variables with:

```
DB_TYPE = postgresdb
DB_POSTGRESDB_HOST = [your-postgres-host]
DB_POSTGRESDB_USER = n8n_user
DB_POSTGRESDB_PASSWORD = [auto-generated password]
DB_POSTGRESDB_DATABASE = n8n
DB_POSTGRESDB_PORT = 5432
DB_POSTGRESDB_SSL = true
```

The connection string appears in your PostgreSQL service info on the Render dashboard.

---

## Optimization Tips for Free Tier

### 1. Reduce Build Time
- Your first build will be slow (10-15 min)
- Subsequent deploys are faster due to caching
- Keep dependencies minimal

### 2. Handle Memory Constraints
```bash
NODE_OPTIONS=--max-old-space-size=512
```
This limits Node.js to 512MB to avoid OOM kills on free tier.

### 3. Auto-sleep Behavior
- Service sleeps after 15 min inactivity
- First request will take 30-60 seconds to wake up
- Workflows won't run while service is asleep
- Set up a cron job to ping your service if needed

### 4. Reduce Slug Size
- Remove unnecessary files
- Use `.renderignore` to exclude dev dependencies

---

## Limitations You'll Face

| Issue | Solution |
|-------|----------|
| Service sleeps after 15 min | Upgrade to paid plan or accept slower response times |
| Data lost on redeploy | Use PostgreSQL database instead of SQLite |
| Limited memory (0.5GB) | Only run lightweight workflows; upgrade for complex ones |
| No persistent disk | All uploads must go to PostgreSQL or external storage |
| Slow first request | Expected behavior on free tier |

---

## Monitoring

1. **View Logs**
   - Render Dashboard → Your Service → Logs

2. **Check Build Status**
   - Render Dashboard → Your Service → Events

3. **Common Issues**
   - OOM error → Increase `NODE_OPTIONS` memory or upgrade plan
   - Build timeout → Reduce dependencies or upgrade plan
   - Service won't start → Check environment variables

---

## When to Upgrade to Paid

- You need persistent data without external DB
- Workflows need to run without service sleeping
- You have more than a few active workflows
- You need more than 0.5GB RAM for execution

---

## Manual Deploy (if auto-deploy fails)

1. Push to GitHub:
   ```bash
   git add .
   git commit -m "Deploy n8n to Render"
   git push origin main
   ```

2. Manual trigger in Render:
   - Dashboard → Your Service → Deploys
   - Click "Deploy latest commit"

---

## Backup Your Workflows

Since free tier can be unstable, regularly backup:

```bash
# Export all workflows
n8n export:workflow --backup
```

Or use n8n UI to export individual workflows.

---

## Support

- n8n Docs: https://docs.n8n.io
- Render Docs: https://render.com/docs
- n8n Community: https://community.n8n.io
- Free tier details: https://render.com/pricing
