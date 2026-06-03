# Deploying n8n to Render

This guide provides step-by-step instructions for hosting n8n on Render.com.

## Prerequisites

- A Render.com account (https://render.com)
- A GitHub repository with this n8n code
- Node.js 22.x+ and pnpm 10.x+ locally (for testing)

## Deployment Options

### Option 1: Using render.yaml (Recommended)

This method uses Render's Infrastructure as Code approach.

#### Steps:

1. **Push your code to GitHub**
   ```bash
   git add .
   git commit -m "Add Render configuration"
   git push origin main
   ```

2. **Connect to Render**
   - Go to https://dashboard.render.com
   - Click "New +"
   - Select "Blueprint"
   - Connect your GitHub repository
   - Select the branch to deploy from

3. **Configure Environment Variables**
   - Go to your Web Service settings
   - Add the following environment variables:
     - `N8N_ENCRYPTION_KEY`: Generate a random 32-character string (keep this secret!)
     - `N8N_USER_MANAGEMENT_JWT_SECRET`: Generate another random 32-character string
     - Any other custom credentials or API keys your workflows need

4. **Deploy**
   - Click "Deploy"
   - Render will automatically build and start your application

### Option 2: Using Docker

If you prefer manual Docker-based deployment:

1. **Build the Docker image**
   ```bash
   docker build -f Dockerfile.render -t n8n:latest .
   ```

2. **Push to Docker Registry**
   ```bash
   # Using Docker Hub
   docker tag n8n:latest yourusername/n8n:latest
   docker push yourusername/n8n:latest
   ```

3. **Deploy on Render**
   - Go to https://dashboard.render.com
   - Click "New +"
   - Select "Web Service"
   - Connect your Docker image
   - Configure as described below

## Configuration Guide

### Essential Environment Variables

```env
# Node environment
NODE_ENV=production
NODE_OPTIONS=--max-old-space-size=2048

# Security Keys (Generate random strings - KEEP THESE SECRET)
N8N_ENCRYPTION_KEY=your-random-32-char-string-here
N8N_USER_MANAGEMENT_JWT_SECRET=another-random-32-char-string

# Database
DB_TYPE=sqlite
DB_SQLITE_PATH=/data/database.sqlite

# URLs
N8N_EDITOR_BASE_URL=https://your-render-url.onrender.com
WEBHOOK_URL=https://your-render-url.onrender.com

# Security
N8N_SECURE_COOKIE=true

# Performance
N8N_THREADS_COUNT=2
```

### Optional Environment Variables

```env
# Enable user management
N8N_USER_MANAGEMENT_DISABLED=false

# Email configuration (for notifications)
N8N_SMTP_HOST=your-smtp-host
N8N_SMTP_PORT=587
N8N_SMTP_USER=your-email
N8N_SMTP_PASS=your-password

# License
N8N_LICENSE_ACTIVATION_KEY=your-license-key

# Logging
LOG_LEVEL=info
```

## Storage & Persistence

The deployment includes a 10GB persistent disk mounted at `/data`:
- SQLite database: `/data/database.sqlite`
- Uploaded files: `/data/files`
- Logs: `/data/logs`

**Important**: The persistent disk is tied to the service. If you delete and recreate the service, you'll lose all data. Consider backing up your database regularly.

## Scaling Considerations

### For Better Performance:

1. **Increase Build Timeout**
   - First build may take 10-15 minutes
   - Adjust in Render dashboard if needed

2. **Upgrade Plan**
   - Start with Standard plan
   - Upgrade to Pro+ for more resources as needed

3. **Database Migration**
   - For production, consider PostgreSQL:
     ```env
     DB_TYPE=postgresdb
     DB_POSTGRESDB_HOST=your-postgres-host
     DB_POSTGRESDB_USER=your-user
     DB_POSTGRESDB_PASSWORD=your-password
     DB_POSTGRESDB_DATABASE=n8n
     DB_POSTGRESDB_PORT=5432
     ```

## Monitoring & Logs

1. **View Logs**
   - Render Dashboard → Your Service → Logs tab
   - Look for any startup errors or missing credentials

2. **Health Checks**
   - Render automatically monitors `/api/v1/health`
   - Service will restart if health checks fail

3. **Performance Metrics**
   - Monitor CPU and Memory usage in Render dashboard
   - Adjust `NODE_OPTIONS` memory limit if needed

## Troubleshooting

### Build Fails

- Check logs in Render dashboard
- Ensure all dependencies are in `package.json`
- Verify Node.js version compatibility (22.x+)
- Try building locally: `pnpm install && pnpm build:deploy`

### Application Won't Start

- Check environment variables are set
- Verify `N8N_ENCRYPTION_KEY` and `N8N_USER_MANAGEMENT_JWT_SECRET` are set
- Check disk space: `df -h /data`
- Review application logs in Render dashboard

### Workflows Not Running

- Verify webhook URL matches your Render URL
- Check credentials are properly stored
- Review execution logs in n8n UI

### Database Connection Issues

- For SQLite: Ensure `/data` directory exists and is writable
- For PostgreSQL: Verify connection string and firewall rules
- Check database logs

## Backup & Recovery

### Backup Strategy

1. **Database Backup** (SQLite)
   ```bash
   # Via n8n CLI
   n8n export:workflow --backup
   ```

2. **Persistent Data Backup**
   - Enable Render's backup feature in dashboard
   - Or manually download files from `/data` directory

### Recovery

- Use Render's disk snapshot feature to restore previous states
- Or restore individual workflows from exported files

## Auto-Deployment

With `autoDeploy: true` in render.yaml:
- Render automatically redeploys when you push to the connected branch
- Each deployment takes 5-10 minutes

To disable auto-deployment:
```yaml
autoDeploy: false
```

Then manually trigger deployments via Render dashboard.

## Next Steps

1. Generate secure random keys for encryption
2. Set all required environment variables in Render dashboard
3. Deploy using render.yaml or Docker method
4. Test your workflows after deployment
5. Set up regular backups
6. Monitor application performance

## Support

- n8n Documentation: https://docs.n8n.io
- Render Documentation: https://render.com/docs
- n8n Community: https://community.n8n.io
