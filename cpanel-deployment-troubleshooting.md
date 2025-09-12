# SlavkoScore™ Enterprise MVP - cPanel Deployment Troubleshooting Guide

This guide provides solutions for common issues that may occur during or after the deployment of SlavkoScore™ Enterprise MVP to a cPanel hosting environment.

## Table of Contents

1. [Database Issues](#1-database-issues)
2. [Backend Application Issues](#2-backend-application-issues)
3. [Frontend Application Issues](#4-frontend-application-issues)
4. [Domain and SSL Issues](#4-domain-and-ssl-issues)
5. [Integration Issues](#5-integration-issues)
6. [Performance Issues](#6-performance-issues)
7. [Maintenance Issues](#7-maintenance-issues)

## 1. Database Issues

### 1.1 Connection Errors

**Issue**: Backend cannot connect to the database

**Symptoms**:
- Error logs show database connection failures
- API endpoints return 500 errors
- Error messages mention "connection refused" or "authentication failed"

**Solutions**:
1. **Verify Database Credentials**:
   ```bash
   # Check the DATABASE_URL in .env file
   cat ~/slavkoscore/.env | grep DATABASE_URL
   
   # Test connection manually
   psql -U slavkoscore_user -d slavkoscore_db -h localhost
   # Or if prefixed:
   psql -U formatdi_slavkoscore_user -d formatdi_slavkoscore_db -h localhost
   ```

2. **Check Database Existence**:
   ```bash
   # List PostgreSQL databases
   psql -l
   ```

3. **Verify User Permissions**:
   ```bash
   # Connect to database
   psql -d slavkoscore_db
   
   # Check user privileges
   \du slavkoscore_user
   # Or if prefixed:
   \du formatdi_slavkoscore_user
   ```

4. **Restart PostgreSQL** (if you have server access):
   ```bash
   # This may require contacting your hosting provider
   ```

### 1.2 Migration/Initialization Errors

**Issue**: Database tables not created or initialized properly

**Symptoms**:
- Error logs show SQL errors
- Application fails to start
- Error messages mention missing tables or columns

**Solutions**:
1. **Run Database Initialization Script**:
   ```bash
   cd ~/slavkoscore
   source venv/bin/activate
   python init_cpanel_db.py
   ```

2. **Check Database Schema**:
   ```bash
   # Connect to database
   psql -d slavkoscore_db
   
   # List tables
   \dt
   
   # Describe specific table
   \d users
   ```

3. **Manual Table Creation** (if needed):
   ```bash
   # Connect to database
   psql -d slavkoscore_db
   
   # Run SQL commands from models
   # Example:
   CREATE TABLE users (
     id SERIAL PRIMARY KEY,
     email VARCHAR(255) UNIQUE NOT NULL,
     ...
   );
   ```

## 2. Backend Application Issues

### 2.1 Application Won't Start

**Issue**: Python FastAPI application fails to start

**Symptoms**:
- Accessing API endpoints returns 503 Service Unavailable
- Error logs show Python exceptions
- Passenger error messages

**Solutions**:
1. **Check Python Version**:
   ```bash
   python3 --version
   # Should be 3.11 or higher
   ```

2. **Verify Virtual Environment**:
   ```bash
   cd ~/slavkoscore
   ls -la venv
   # If missing or incomplete:
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```

3. **Check Passenger Configuration**:
   ```bash
   # Verify passenger_wsgi.py exists
   ls -la ~/slavkoscore/passenger_wsgi.py
   
   # Check its content
   cat ~/slavkoscore/passenger_wsgi.py
   ```

4. **Restart Passenger**:
   ```bash
   mkdir -p ~/slavkoscore/tmp
   touch ~/slavkoscore/tmp/restart.txt
   ```

5. **Check Error Logs**:
   ```bash
   tail -n 100 ~/logs/error_log
   ```

### 2.2 Missing Dependencies

**Issue**: Python dependencies not installed or incompatible

**Symptoms**:
- Error logs show ImportError or ModuleNotFoundError
- Application crashes on startup

**Solutions**:
1. **Reinstall Dependencies**:
   ```bash
   cd ~/slavkoscore
   source venv/bin/activate
   pip install --upgrade -r requirements.txt
   ```

2. **Check for Conflicting Packages**:
   ```bash
   pip list
   ```

3. **Install Missing Package Manually**:
   ```bash
   pip install package_name
   ```

### 2.3 Environment Variables Issues

**Issue**: Missing or incorrect environment variables

**Symptoms**:
- Application crashes on startup
- Specific features fail
- Error logs mention missing configuration

**Solutions**:
1. **Check Environment File**:
   ```bash
   cat ~/slavkoscore/.env
   ```

2. **Verify Required Variables**:
   - DATABASE_URL
   - SECRET_KEY
   - API_URL
   - CORS_ORIGINS

3. **Update Environment File**:
   ```bash
   # Edit .env file
   nano ~/slavkoscore/.env
   
   # Add or correct variables
   SECRET_KEY=your_secure_key
   ```

## 3. Frontend Application Issues

### 3.1 Next.js Build Issues

**Issue**: Frontend application doesn't load or shows errors

**Symptoms**:
- Blank page or error page
- Console errors in browser
- Missing assets or components

**Solutions**:
1. **Check Build Files**:
   ```bash
   ls -la ~/public_html/.next
   # Should contain build, static directories
   ```

2. **Verify Environment Variables**:
   ```bash
   cat ~/public_html/.env.production
   # Check NEXT_PUBLIC_API_URL is correct
   ```

3. **Rebuild Frontend** (if possible):
   ```bash
   cd ~/public_html
   npm run build
   ```

### 3.2 Static Asset Issues

**Issue**: CSS, JavaScript, or images not loading

**Symptoms**:
- Unstyled pages
- Missing functionality
- Broken images

**Solutions**:
1. **Check File Permissions**:
   ```bash
   find ~/public_html/.next -type f -exec ls -la {} \; | head
   # Files should be 644
   
   # Fix permissions if needed
   find ~/public_html/.next -type f -exec chmod 644 {} \;
   ```

2. **Verify .htaccess Configuration**:
   ```bash
   cat ~/public_html/.htaccess
   # Should contain proper rewrite rules
   ```

3. **Check Browser Console** for specific errors

### 3.3 API Connection Issues

**Issue**: Frontend cannot connect to backend API

**Symptoms**:
- Console shows CORS errors
- API requests fail
- Features dependent on API don't work

**Solutions**:
1. **Verify API URL**:
   ```bash
   cat ~/public_html/.env.production | grep NEXT_PUBLIC_API_URL
   # Should be https://api.formatdisc.hr
   ```

2. **Check CORS Configuration**:
   ```bash
   cat ~/slavkoscore/.env | grep CORS_ORIGINS
   # Should include https://formatdisc.hr
   ```

3. **Test API Directly**:
   ```bash
   curl https://api.formatdisc.hr/health
   # Should return {"status":"ok"}
   ```

## 4. Domain and SSL Issues

### 4.1 Domain Not Resolving

**Issue**: Domain doesn't point to the correct server

**Symptoms**:
- Website doesn't load
- DNS errors

**Solutions**:
1. **Check DNS Configuration**:
   ```bash
   dig formatdisc.hr
   # Should point to 178.63.25.160
   ```

2. **Verify Domain in cPanel**:
   - Go to Domains section in cPanel
   - Check that formatdisc.hr points to public_html

3. **Wait for DNS Propagation**:
   - DNS changes can take up to 48 hours to propagate
   - Use the IP address directly in the meantime

### 4.2 SSL Certificate Issues

**Issue**: SSL certificates not working or showing errors

**Symptoms**:
- Browser shows "Not Secure" warning
- Certificate errors
- Mixed content warnings

**Solutions**:
1. **Check Certificate Status**:
   - Go to SSL/TLS section in cPanel
   - Verify certificates are issued and not expired

2. **Regenerate Certificates**:
   - Use Let's Encrypt in cPanel to regenerate certificates
   - Ensure both domains are included

3. **Force HTTPS Redirection**:
   ```bash
   # Check .htaccess for redirection rules
   cat ~/public_html/.htaccess
   ```

4. **Test SSL Configuration**:
   - Use [SSL Labs](https://www.ssllabs.com/ssltest/) to test SSL setup
   - Fix any issues identified

## 5. Integration Issues

### 5.1 Authentication Problems

**Issue**: Users cannot log in or authentication fails

**Symptoms**:
- Login attempts fail
- JWT token errors
- Unauthorized errors when accessing protected resources

**Solutions**:
1. **Check Authentication Configuration**:
   ```bash
   # Backend
   cat ~/slavkoscore/.env | grep -E 'SECRET_KEY|ALGORITHM|ACCESS_TOKEN_EXPIRE'
   
   # Frontend
   cat ~/public_html/.env.production | grep -E 'NEXTAUTH_SECRET|NEXTAUTH_URL'
   ```

2. **Verify Admin User**:
   ```bash
   # Connect to database
   psql -d slavkoscore_db
   
   # Check users table
   SELECT email FROM users;
   ```

3. **Reset Admin Password** (if needed):
   ```python
   # Create a script to reset password
   from app.core.security import get_password_hash
   from app.db.session import SessionLocal
   from app.models.user import User

   db = SessionLocal()
   admin = db.query(User).filter(User.email == "admin@formatdisc.hr").first()
   if admin:
       admin.hashed_password = get_password_hash("new_password")
       db.commit()
       print("Password reset successfully")
   else:
       print("Admin user not found")
   db.close()
   ```

### 5.2 Data Submission Issues

**Issue**: Data not being saved or retrieved correctly

**Symptoms**:
- Form submissions fail
- Data doesn't appear in lists or reports
- Database errors

**Solutions**:
1. **Check API Endpoints**:
   ```bash
   # Test API endpoint
   curl -X POST https://api.formatdisc.hr/api/evaluations \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer YOUR_TOKEN" \
     -d '{"name":"Test","description":"Test"}'
   ```

2. **Verify Database Operations**:
   ```bash
   # Connect to database
   psql -d slavkoscore_db
   
   # Check data in tables
   SELECT * FROM evaluations LIMIT 5;
   ```

3. **Check Backend Logs** for specific errors

## 6. Performance Issues

### 6.1 Slow Response Times

**Issue**: Application responds slowly

**Symptoms**:
- Pages take long to load
- API requests are slow
- Timeouts occur

**Solutions**:
1. **Check Server Resources**:
   ```bash
   # CPU and memory usage
   top
   
   # Disk space
   df -h
   ```

2. **Optimize Database Queries**:
   ```bash
   # Connect to database
   psql -d slavkoscore_db
   
   # Enable query logging (if possible)
   # Identify slow queries
   ```

3. **Enable Caching** (if applicable)

4. **Optimize Frontend Assets**:
   - Enable compression in .htaccess
   - Set appropriate cache headers

### 6.2 Memory Issues

**Issue**: Application crashes due to memory limits

**Symptoms**:
- 503 errors
- Application restarts frequently
- Out of memory errors in logs

**Solutions**:
1. **Check Memory Limits**:
   ```bash
   # Python memory usage
   ps aux | grep python
   ```

2. **Optimize Code** to reduce memory usage

3. **Contact Hosting Provider** to increase memory limits if needed

## 7. Maintenance Issues

### 7.1 Backup and Restore

**Issue**: Need to backup or restore data

**Solutions**:
1. **Backup Database**:
   ```bash
   # Backup PostgreSQL database
   pg_dump -U slavkoscore_user -d slavkoscore_db > backup.sql
   
   # Compress backup
   gzip backup.sql
   ```

2. **Backup Files**:
   ```bash
   # Backup application files
   tar -czf slavkoscore_backup.tar.gz ~/slavkoscore ~/public_html
   ```

3. **Restore Database**:
   ```bash
   # Restore PostgreSQL database
   psql -U slavkoscore_user -d slavkoscore_db < backup.sql
   ```

### 7.2 Updating the Application

**Issue**: Need to update application code

**Solutions**:
1. **Backup Before Updating**:
   ```bash
   # Backup current files
   tar -czf slavkoscore_backup_$(date +%Y%m%d).tar.gz ~/slavkoscore ~/public_html
   ```

2. **Update Backend**:
   ```bash
   # Upload new files
   # Restart application
   touch ~/slavkoscore/tmp/restart.txt
   ```

3. **Update Frontend**:
   ```bash
   # Upload new build files
   # Clear cache if needed
   ```

## Getting Help

If you encounter issues that aren't covered in this guide:

1. **Check Application Logs**:
   ```bash
   tail -n 100 ~/logs/error_log
   ```

2. **Contact Technical Support**:
   - Email: support@formatdisc.hr
   - Include detailed error messages and steps to reproduce

3. **Documentation Resources**:
   - Refer to the complete deployment guide
   - Check the SlavkoScore™ documentation