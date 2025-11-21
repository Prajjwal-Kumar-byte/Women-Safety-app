# SafeGuard Deployment Guide

This guide covers deployment of the SafeGuard Women Safety App to production.

## Prerequisites

- Node.js 18+
- Git
- Firebase account (for backend)
- Netlify or Vercel account (for hosting)
- Domain name (optional)

## Local Development Setup

```bash
# Clone repository
git clone <repository-url>
cd safeguard-app

# Install dependencies
pnpm install

# Start dev server
pnpm dev
```

App will be available at `http://localhost:5173`

## Build for Production

```bash
# Build the application
pnpm build

# Output directory: dist/

# Test production build locally
pnpm preview
```

## Deployment Options

### Option 1: Netlify Deployment (Recommended)

#### Via Git Integration

1. **Push code to GitHub/GitLab**
```bash
git add .
git commit -m "Initial commit"
git push origin main
```

2. **Connect to Netlify**
   - Go to https://app.netlify.com
   - Click "New site from Git"
   - Select your repository
   - Configure build settings:
     - Build command: `pnpm build`
     - Publish directory: `dist`

3. **Set environment variables**
   - Go to Site settings → Environment
   - Add variables from `.env.example`

4. **Deploy**
   - Netlify will automatically deploy on push

#### Via CLI

```bash
# Install Netlify CLI
npm install -g netlify-cli

# Login to Netlify
netlify login

# Deploy
netlify deploy --prod

# Or with automatic builds
netlify init
```

### Option 2: Vercel Deployment

```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
vercel --prod

# Follow prompts to connect Git and configure
```

**vercel.json configuration:**
```json
{
  "buildCommand": "pnpm build",
  "outputDirectory": "dist",
  "env": {
    "VITE_FIREBASE_API_KEY": "@firebase_api_key",
    "VITE_FIREBASE_PROJECT_ID": "@firebase_project_id"
  }
}
```

### Option 3: Firebase Hosting

```bash
# Install Firebase CLI
npm install -g firebase-tools

# Initialize Firebase project
firebase init hosting

# Build
pnpm build

# Deploy
firebase deploy --only hosting
```

**firebase.json:**
```json
{
  "hosting": {
    "public": "dist",
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ],
    "headers": [
      {
        "source": "**",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "max-age=0"
          }
        ]
      }
    ]
  }
}
```

### Option 4: Self-Hosted (VPS/Docker)

#### Docker Deployment

1. **Create Dockerfile**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install
COPY . .
RUN pnpm build

FROM node:18-alpine
WORKDIR /app
RUN npm install -g serve
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["serve", "-s", "dist", "-l", "3000"]
```

2. **Build and deploy**
```bash
docker build -t safeguard-app .
docker run -p 3000:3000 safeguard-app
```

#### Traditional Server

1. **SSH into server**
```bash
ssh user@your-server.com
cd /var/www
git clone <repo-url>
cd safeguard-app
```

2. **Install and build**
```bash
curl -fsSL https://get.pnpm.io/install.sh | sh -
pnpm install
pnpm build
```

3. **Configure web server (Nginx)**
```nginx
server {
    listen 80;
    server_name safeguard.example.com;

    location / {
        root /var/www/safeguard-app/dist;
        try_files $uri $uri/ /index.html;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

4. **Setup SSL with Let's Encrypt**
```bash
sudo certbot certonly --nginx -d safeguard.example.com
```

## Firebase Backend Setup

### 1. Create Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com)
2. Click "Create Project"
3. Follow setup wizard
4. Enable the following services:
   - Authentication
   - Firestore Database
   - Cloud Functions
   - Hosting
   - Cloud Storage (for audio files)

### 2. Initialize Firebase

```bash
npm install -g firebase-tools
firebase login
firebase init
```

### 3. Configure Authentication

**Enable sign-in methods:**
- Email/Password
- Google Sign-In
- Optional: Phone Authentication

### 4. Setup Firestore Database

Create collections:
```javascript
// Initialize with default rules (then update)
collection('users') -> store user profiles
collection('contacts') -> store emergency contacts
collection('alerts') -> store alert history
collection('alertLogs') -> store alert events
```

### 5. Security Rules

```javascript
// firestore.rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can read/write their own documents
    match /users/{userId} {
      allow read, write: if request.auth.uid == userId;
    }

    // Emergency contacts - users can manage their own
    match /contacts/{contactId} {
      allow read, write: if request.auth.uid == resource.data.userId;
    }

    // Alerts - users can read their own, police can read all
    match /alerts/{alertId} {
      allow read: if request.auth.uid == resource.data.userId;
      allow create: if request.auth != null;
      allow update: if request.auth.uid == resource.data.userId || 
                       'police' in request.auth.token.claims;
    }

    // Alert logs
    match /alertLogs/{logId} {
      allow write: if request.auth != null;
      allow read: if request.auth.uid == resource.data.userId ||
                     'police' in request.auth.token.claims;
    }
  }
}
```

### 6. Deploy Cloud Functions

```bash
cd functions
npm install

# Write your functions in index.ts
# Example: Alert notification function
firebase deploy --only functions
```

## Environment Configuration

### Production Environment Variables

Create `.env.production`:
```bash
VITE_FIREBASE_API_KEY=your_production_key
VITE_FIREBASE_AUTH_DOMAIN=your_production_domain
VITE_FIREBASE_PROJECT_ID=your_production_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_production_bucket
VITE_FIREBASE_MESSAGING_SENDER_ID=your_production_sender_id
VITE_FIREBASE_APP_ID=your_production_app_id

VITE_POLICE_API_URL=https://api.safeguardapp.com
VITE_TWILIO_FROM_NUMBER=+1234567890
VITE_ENABLE_AUDIO_DETECTION=true
```

### Setting Environment Variables in Hosting

#### Netlify
1. Site settings → Build & deploy → Environment
2. Add environment variables

#### Vercel
1. Project settings → Environment Variables
2. Add for Production

#### Firebase Hosting
```bash
# Via firebase.json
{
  "hosting": {
    "env": [
      "VITE_FIREBASE_API_KEY",
      "VITE_POLICE_API_URL"
    ]
  }
}
```

## Performance Optimization

### 1. Enable Caching

```javascript
// vercel.json or netlify.toml
headers: [
  {
    source: "/api/**",
    headers: [
      { key: "Cache-Control", value: "no-cache" }
    ]
  },
  {
    source: "/assets/**",
    headers: [
      { key: "Cache-Control", value: "public, max-age=31536000" }
    ]
  }
]
```

### 2. Enable GZIP Compression

Most platforms do this automatically. Verify:
```bash
curl -I https://safeguard.example.com
# Should see: Content-Encoding: gzip
```

### 3. Optimize Bundle

```bash
# Analyze bundle size
npm run build -- --analyze

# Remove unused dependencies
pnpm prune
```

### 4. CDN Configuration

- **Netlify**: Automatically uses Netlify Edge
- **Vercel**: Automatically uses Vercel Edge
- **Firebase**: Use Cloud CDN
- **Self-hosted**: Setup Cloudflare

## SSL/HTTPS Setup

### For Netlify/Vercel
- Automatic SSL certificates
- No additional setup needed

### For Firebase Hosting
- Automatic SSL
- Configure domain in Firebase Console

### For Self-Hosted
```bash
# Using Let's Encrypt
sudo certbot certonly --standalone -d safeguard.example.com
sudo certbot renew --dry-run  # Test auto-renewal
```

## Monitoring & Logging

### Firebase Monitoring

```bash
# Setup Google Cloud Monitoring
gcloud services enable monitoring.googleapis.com

# Create alerts for:
# - High error rate
# - High latency
# - Failed authentication
```

### Application Performance Monitoring

Add to your app:
```typescript
// Firebase Performance Monitoring
import { getPerformance } from "firebase/performance";
const perf = getPerformance(app);
```

### Error Tracking (Sentry)

```bash
npm install @sentry/react @sentry/tracing
```

## Backup & Recovery

### Firestore Backup

```bash
# Automatic backups (Firebase Premium)
# Or manual backup:
gcloud firestore export gs://your-bucket/backup-name

# Restore
gcloud firestore import gs://your-bucket/backup-name
```

### Version Control Backups

Always maintain Git history:
```bash
git push origin main
git push origin --tags
```

## Testing Production Deployment

```bash
# Health checks
curl https://safeguard.example.com/health

# Performance test
lighthouse https://safeguard.example.com

# Security audit
npm audit
```

## Rollback Procedure

### Netlify
1. Go to Deploys
2. Click on previous deployment
3. Click "Publish deploy"

### Vercel
1. Go to Deployments
2. Click "Promote to Production"

### Firebase
```bash
firebase hosting:rollback
```

## Monitoring Checklist

- [ ] SSL certificate valid and not expiring
- [ ] Uptime monitoring configured
- [ ] Error tracking enabled
- [ ] Performance metrics collecting
- [ ] Backups scheduled
- [ ] Logs being collected
- [ ] Alerts configured for critical issues
- [ ] CDN caching working
- [ ] Rate limiting enabled
- [ ] DDoS protection active

## Scaling Considerations

### Database Scaling
- Use Firestore auto-scaling
- Monitor query performance
- Add indexes for common queries

### API Scaling
- Use Cloud Functions auto-scaling
- Add rate limiting
- Implement caching

### Frontend Scaling
- Use CDN for static assets
- Enable lazy loading
- Optimize images

## Maintenance

### Regular Tasks
- Update dependencies: `pnpm update`
- Review security alerts: `npm audit`
- Monitor error rates
- Review performance metrics
- Backup data regularly

### Monthly
- Review logs for errors
- Update documentation
- Test disaster recovery
- Review security rules

### Quarterly
- Load testing
- Penetration testing
- Dependency updates
- Cost analysis

## Support & Troubleshooting

### Common Issues

**1. Build fails on deploy**
```bash
# Clear cache
# Redeploy with --force
netlify deploy --prod --force
```

**2. Environment variables not loading**
- Check variable names match `.env.example`
- Restart deployment
- Check variable scope (production vs preview)

**3. Firebase connection errors**
- Verify API key is correct
- Check security rules
- Verify database exists

**4. Performance issues**
- Check bundle size: `pnpm build --analyze`
- Review Lighthouse report
- Enable compression

## Security Checklist

- [ ] HTTPS enabled
- [ ] Security headers configured
- [ ] CORS properly configured
- [ ] Authentication implemented
- [ ] Authorization rules in place
- [ ] Input validation enabled
- [ ] Rate limiting enabled
- [ ] Secrets not in code
- [ ] Dependencies audited
- [ ] Regular security updates

---

For support: support@safeguardapp.com
Documentation: https://docs.safeguardapp.com
