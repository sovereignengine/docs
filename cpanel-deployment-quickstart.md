# SlavkoScore™ Enterprise MVP - cPanel Deployment Quick Start Guide

This quick start guide provides the essential steps for deploying the SlavkoScore™ Enterprise MVP to a cPanel hosting environment. For detailed instructions, refer to the complete deployment guide.

## Access Information

- **cPanel URL**: http://178.63.25.160/cpanel (temporary) or https://formatdisc.hr/cpanel (after domain propagation)
- **Username**: formatdi
- **Password**: S1Sx]rSEtd9)79
- **Server IP**: 178.63.25.160

## 1. Database Setup

```bash
# 1. Access cPanel and navigate to PostgreSQL Databases
# 2. Create database: slavkoscore_db
# 3. Create user: slavkoscore_user with secure password
# 4. Grant ALL PRIVILEGES to user on database
# 5. Update DATABASE_URL in backend .env file
```

## 2. Deployment Execution

### Using the Deployment Script (Recommended)

```bash
# Extract deployment package
unzip slavkoscore_deployment_package.zip
cd slavkoscore_deployment_package

# Run deployment script
chmod +x scripts/cpanel_deploy.sh
./scripts/cpanel_deploy.sh formatdi formatdisc.hr ./frontend ./backend
```

### Manual Deployment (Alternative)

```bash
# Upload frontend files to public_html
# Upload backend files to slavkoscore
# Set permissions: directories 755, files 644
# Install dependencies:
cd ~/slavkoscore
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

cd ~/public_html
npm install --production

# Initialize database:
cd ~/slavkoscore
source venv/bin/activate
python init_cpanel_db.py
```

## 3. Domain and SSL Configuration

```bash
# 1. Configure main domain (formatdisc.hr) to point to public_html
# 2. Create API subdomain (api.formatdisc.hr) pointing to slavkoscore
# 3. Generate SSL certificates using Let's Encrypt
# 4. Enable HTTPS redirection
```

## 4. Application Configuration

```bash
# 1. Configure Python application in cPanel:
#    - Python version: 3.11+
#    - App root: /home/formatdi/slavkoscore
#    - App URL: https://api.formatdisc.hr
#    - Startup file: passenger_wsgi.py

# 2. (Optional) Configure Node.js application:
#    - Node.js version: 20.x+
#    - App root: /home/formatdi/public_html
#    - App URL: https://formatdisc.hr
#    - Startup file: node_modules/next/dist/bin/next start

# 3. Restart services:
mkdir -p ~/slavkoscore/tmp
touch ~/slavkoscore/tmp/restart.txt
```

## 5. Verification

```bash
# Check backend:
curl https://api.formatdisc.hr/health
# Should return: {"status":"ok"}

# Check API docs:
# Visit https://api.formatdisc.hr/docs

# Check frontend:
# Visit https://formatdisc.hr

# Test authentication and core features
```

## 6. Troubleshooting

- **Database Issues**: Check credentials, permissions, and connection string
- **SSL Issues**: Verify domain configuration and certificate status
- **Application Issues**: Check logs at `/home/formatdi/logs/error_log`
- **CORS Issues**: Verify CORS_ORIGINS in backend .env includes frontend domain

## 7. Post-Deployment

- Create admin user (if not automatically created)
- Configure regular backups
- Monitor logs and performance
- Update documentation with deployment details

For detailed instructions, refer to the complete deployment guide.