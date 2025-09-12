# SlavkoScore™ Enterprise MVP - Complete cPanel Deployment Guide

This comprehensive guide provides detailed instructions for deploying the SlavkoScore™ Enterprise MVP to a cPanel hosting environment. It covers all aspects of the deployment process, from initial setup to post-deployment verification.

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Access Information](#2-access-information)
3. [Database Setup](#3-database-setup)
4. [Deployment Execution](#4-deployment-execution)
5. [Domain and SSL Configuration](#5-domain-and-ssl-configuration)
6. [Application Configuration](#6-application-configuration)
7. [Post-Deployment Verification](#7-post-deployment-verification)
8. [Maintenance and Support](#8-maintenance-and-support)
9. [Troubleshooting](#9-troubleshooting)

## 1. Prerequisites

Before beginning the deployment, ensure you have:

- cPanel access credentials
- SSH access to the server (optional but recommended)
- The SlavkoScore™ deployment package (`slavkoscore_deployment_package.zip`)
- Basic understanding of Linux commands, cPanel operations, and web application deployment

## 2. Access Information

### cPanel Access

- **Temporary URL**: http://178.63.25.160/cpanel
- **After domain propagation**: https://formatdisc.hr/cpanel
- **Username**: formatdi
- **Password**: S1Sx]rSEtd9)79

### SSH Access

- **Hostname**: 178.63.25.160 (or formatdisc.hr after domain propagation)
- **Username**: formatdi
- **Password**: S1Sx]rSEtd9)79
- **SSH Key**: 
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCUrHzpYixMzGfSwew0nRoz13ik9T2oslaeDisdSrdzg80v8ZJZqkQRKzuTDhuiHYYNiz4hw8wuTfA5L6x24JwHbZd3LlpPsdIeI1I4eQYSaA8bCXYWXZJYh9U9r3rYXl18I2hXBS8T55kcoR7Zz58Aum26eHB1YeYfBFmY4pjOkBR5PEYWLLIrFKNB/vgUjxGCXgYOEaZRiW7gpToUeXykmXLSHZ0fZHV/lnw1yOIp9UBpUdOrH/poslllROOV3aF1r2yopCJuspbQ3QLeb4vUTuLlRf/zlMvolwq4dW5mIOKTeaDFG0uJ6mT7/Ib+yHNE1Vod0zBa5kJ9DouoqA3l
```

### Server Information

- **Server Name**: srv5
- **IP Address**: 178.63.25.160
- **Nameservers**:
  - ns1.kuhada.net
  - ns2.kuhada.net

## 3. Database Setup

### 3.1 Access PostgreSQL Databases in cPanel

1. Log in to cPanel using the credentials provided above
2. Navigate to "PostgreSQL Databases" in the Databases section

### 3.2 Create a New Database

1. In the "Create New Database" section:
   - Enter "slavkoscore_db" in the database name field
   - Click "Create Database"
2. Note: cPanel may prefix the database name with your username (e.g., formatdi_slavkoscore_db)

### 3.3 Create a New Database User

1. In the "Add New User" section:
   - Enter "slavkoscore_user" in the username field
   - Enter a secure password (e.g., "SlavkoScore2025!") in the password field
   - Confirm the password
   - Click "Create User"
2. Note: cPanel may prefix the username with your username (e.g., formatdi_slavkoscore_user)

### 3.4 Add User to Database

1. In the "Add User To Database" section:
   - Select "slavkoscore_user" (or the prefixed version) from the user dropdown
   - Select "slavkoscore_db" (or the prefixed version) from the database dropdown
   - Click "Add"
2. On the privileges page:
   - Select "ALL PRIVILEGES"
   - Click "Make Changes"

### 3.5 Update Database Connection String

After creating the database and user, update the database connection string in the backend environment file:

1. If the database name and username are prefixed:
   ```
   DATABASE_URL=postgresql://formatdi_slavkoscore_user:YOUR_PASSWORD@localhost:5432/formatdi_slavkoscore_db
   ```

2. If the database name and username are not prefixed:
   ```
   DATABASE_URL=postgresql://slavkoscore_user:YOUR_PASSWORD@localhost:5432/slavkoscore_db
   ```

3. Replace `YOUR_PASSWORD` with the actual password you created for the database user.

### 3.6 Verify Database Connection

To verify that the database connection is working correctly:

1. SSH into the server (if available):
   ```bash
   ssh formatdi@formatdisc.hr
   ```

2. Test the connection using psql:
   ```bash
   # If prefixed
   psql -U formatdi_slavkoscore_user -d formatdi_slavkoscore_db -h localhost
   
   # If not prefixed
   psql -U slavkoscore_user -d slavkoscore_db -h localhost
   ```

3. When prompted, enter the password you created for the database user.

4. If the connection is successful, you should see the psql prompt:
   ```
   psql (x.x.x)
   Type "help" for help.
   
   slavkoscore_db=>
   ```

5. Exit psql by typing:
   ```
   \q
   ```

## 4. Deployment Execution

### 4.1 Extract Deployment Package

1. Download the `slavkoscore_deployment_package.zip` file to your local machine
2. Extract the contents of the zip file

### 4.2 Upload Files

#### 4.2.1 Using the Deployment Script

The easiest way to deploy is using the provided deployment script:

1. Make the script executable:
   ```bash
   chmod +x scripts/cpanel_deploy.sh
   ```

2. Run the deployment script:
   ```bash
   ./scripts/cpanel_deploy.sh formatdi formatdisc.hr ./frontend ./backend
   ```

#### 4.2.2 Manual Upload (Alternative)

If you prefer to upload files manually:

1. **Frontend Files**:
   - Using File Manager in cPanel or FTP, upload the following to `/home/formatdi/public_html`:
     - `.next/` directory
     - `public/` directory
     - `package.json`
     - `next.config.ts`
     - `.htaccess`
     - `.env.production`

2. **Backend Files**:
   - Using File Manager in cPanel or FTP, upload the following to `/home/formatdi/slavkoscore`:
     - `app/` directory
     - `requirements.txt`
     - `passenger_wsgi.py`
     - `run.py`
     - `.env`

### 4.3 Set File Permissions

Set the correct permissions for files and directories:

```bash
# For frontend
find /home/formatdi/public_html -type d -exec chmod 755 {} \;
find /home/formatdi/public_html -type f -exec chmod 644 {} \;

# For backend
find /home/formatdi/slavkoscore -type d -exec chmod 755 {} \;
find /home/formatdi/slavkoscore -type f -exec chmod 644 {} \;
chmod 755 /home/formatdi/slavkoscore/run.py
```

### 4.4 Install Dependencies

#### 4.4.1 Backend Dependencies

```bash
cd /home/formatdi/slavkoscore
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

#### 4.4.2 Frontend Dependencies

```bash
cd /home/formatdi/public_html
npm install --production
```

### 4.5 Initialize Database

After setting up the database and installing dependencies, initialize the database:

```bash
cd /home/formatdi/slavkoscore
source venv/bin/activate
python init_cpanel_db.py
```

## 5. Domain and SSL Configuration

### 5.1 Main Domain Configuration

1. In cPanel, navigate to "Domains" in the Domains section
2. Verify that the main domain (`formatdisc.hr`) is pointing to the `public_html` directory
3. If not, click on "Manage" next to the domain and set the Document Root to `/home/formatdi/public_html`

### 5.2 API Subdomain Configuration

1. In cPanel, navigate to "Subdomains" in the Domains section
2. Create a new subdomain:
   - Subdomain: api
   - Domain: formatdisc.hr
   - Document Root: `/home/formatdi/slavkoscore`
   - Click "Create"

### 5.3 SSL Certificate Configuration

1. In cPanel, navigate to "SSL/TLS" in the Security section
2. Click on "Manage SSL Sites"
3. Use the "Let's Encrypt" option to generate free SSL certificates:
   - Select both `formatdisc.hr` and `api.formatdisc.hr` domains
   - Click "Issue" to generate the certificates
   - Wait for the certificates to be issued (this may take a few minutes)

### 5.4 Enable HTTPS Redirection

1. In cPanel, navigate to "SSL/TLS Status" in the Security section
2. Ensure that both `formatdisc.hr` and `api.formatdisc.hr` show as "Enabled" for SSL
3. Navigate to "Redirects" in the Domains section
4. Create redirects for HTTP to HTTPS for both domains (select "Permanent (301)" redirect type)

## 6. Application Configuration

### 6.1 Python Application Configuration

1. In cPanel, navigate to "Setup Python App" in the Software section
2. Click on "Create Application"
3. Configure the Python application:
   - Python version: 3.11 (or the highest available)
   - Application root: `/home/formatdi/slavkoscore`
   - Application URL: `https://api.formatdisc.hr`
   - Application startup file: `passenger_wsgi.py`
   - Application Entry point: `application`
4. Click "Create" to set up the Python application

### 6.2 Node.js Configuration (Optional)

If you need to configure Node.js for the frontend:

1. In cPanel, navigate to "Setup Node.js App" in the Software section
2. Click on "Create Application"
3. Configure the Node.js application:
   - Node.js version: 20.x (or the highest available)
   - Application mode: Production
   - Application root: `/home/formatdi/public_html`
   - Application URL: `https://formatdisc.hr`
   - Application startup file: `node_modules/next/dist/bin/next start`
4. Click "Create" to set up the Node.js application

### 6.3 Restart Application Services

1. Create a tmp directory for Passenger restart:
   ```bash
   mkdir -p /home/formatdi/slavkoscore/tmp
   ```

2. Restart Passenger WSGI:
   ```bash
   touch /home/formatdi/slavkoscore/tmp/restart.txt
   ```

3. If using Node.js, restart the Node.js application from cPanel.

## 7. Post-Deployment Verification

### 7.1 Backend API Verification

#### 7.1.1 Health Check Endpoint

1. Navigate to `https://api.formatdisc.hr/health`
2. Verify that the response is:
   ```json
   {"status":"ok"}
   ```
3. Check that the HTTP status code is 200

#### 7.1.2 API Documentation

1. Navigate to `https://api.formatdisc.hr/docs`
2. Verify that the Swagger documentation loads correctly
3. Check that all API endpoints are listed
4. Test the "Try it out" functionality for a simple endpoint (e.g., `/health`)

#### 7.1.3 Authentication Endpoints

1. In the Swagger documentation, locate the authentication endpoints
2. Test the `/auth/token` endpoint with the admin credentials:
   - Username: `admin@formatdisc.hr`
   - Password: [the password set in backend.env]
3. Verify that a valid JWT token is returned

#### 7.1.4 Protected Endpoints

1. Obtain a JWT token using the authentication endpoint
2. Test a protected endpoint using the "Authorize" button in Swagger
3. Verify that the endpoint returns the expected response

#### 7.1.5 Backend Logs

Check the backend logs for any errors:
```bash
ssh formatdi@formatdisc.hr "tail -n 100 /home/formatdi/logs/error_log"
```

### 7.2 Frontend Verification

#### 7.2.1 Website Loading

1. Visit `https://formatdisc.hr`
2. Verify that the website loads correctly
3. Check that all static assets (CSS, JavaScript, images) load without errors
4. Verify that the page is served over HTTPS (look for the padlock icon)

#### 7.2.2 Responsive Design

1. Test the website on different screen sizes:
   - Desktop (1920x1080)
   - Tablet (768x1024)
   - Mobile (375x667)
2. Verify that the layout adapts correctly to each screen size
3. Check that all elements are properly visible and usable

#### 7.2.3 Authentication

1. Test user login functionality:
   - Navigate to the login page
   - Enter valid credentials
   - Verify that login is successful
   - Check that you are redirected to the appropriate page

2. Test logout functionality:
   - Click on the logout button/link
   - Verify that you are logged out
   - Check that protected pages are no longer accessible

#### 7.2.4 Core Features

1. Test the evaluation submission process:
   - Navigate to the evaluation form
   - Fill out all required fields
   - Submit the form
   - Verify that the submission is successful

2. Verify that evaluations are properly stored:
   - Check that submitted evaluations appear in the history/list
   - Verify that the data is correctly displayed

3. Test the evaluation results display:
   - Navigate to the results page for a completed evaluation
   - Verify that all results are correctly displayed
   - Check that charts and visualizations render properly

### 7.3 Integration Tests

#### 7.3.1 Frontend-Backend Communication

1. Open the browser developer tools (F12)
2. Navigate to the Network tab
3. Perform actions that trigger API calls
4. Verify that API calls are made to the correct endpoints
5. Check that responses are correctly handled by the frontend

#### 7.3.2 Error Handling

1. Test error scenarios:
   - Submit invalid data
   - Try to access protected resources without authentication
   - Attempt to perform unauthorized actions
2. Verify that appropriate error messages are displayed
3. Check that the application recovers gracefully from errors

#### 7.3.3 Database Integration

Test database operations:
```bash
ssh formatdi@formatdisc.hr "cd /home/formatdi/slavkoscore && source venv/bin/activate && python -c &quot;from app.db.session import SessionLocal; from app.models.evaluation import Evaluation; db = SessionLocal(); print(db.query(Evaluation).all()); db.close()&quot;"
```

### 7.4 Security Verification

#### 7.4.1 SSL/TLS

1. Verify that SSL certificates are valid for both domains
2. Check that HTTPS is enforced for all connections
3. Verify SSL configuration using [SSL Labs](https://www.ssllabs.com/ssltest/)
   - Enter your domain and run the test
   - Aim for at least an "A" rating

#### 7.4.2 Authentication Security

1. Verify that passwords are properly hashed in the database
2. Check that authentication tokens have appropriate expiration
3. Test that unauthorized access is properly prevented
4. Verify that session management is secure

#### 7.4.3 CORS Configuration

Check that CORS is properly configured:
```bash
curl -I -H "Origin: https://formatdisc.hr" https://api.formatdisc.hr/health
```

Verify that the response includes the appropriate CORS headers:
```
Access-Control-Allow-Origin: https://formatdisc.hr
```

### 7.5 Verification Checklist

| Category        | Item                           | Status |
| --------------- | ------------------------------ | ------ |
| **Backend**     | Health endpoint returns 200    | □      |
|                 | API documentation loads        | □      |
|                 | Authentication works           | □      |
|                 | Protected endpoints secure     | □      |
|                 | No critical errors in logs     | □      |
| **Frontend**    | Website loads correctly        | □      |
|                 | Static assets load             | □      |
|                 | Responsive design works        | □      |
|                 | Authentication works           | □      |
|                 | Core features function         | □      |
| **Integration** | Frontend-backend communication | □      |
|                 | Error handling works           | □      |
|                 | Database operations work       | □      |
| **Security**    | SSL certificates valid         | □      |
|                 | HTTPS enforced                 | □      |
|                 | Authentication secure          | □      |
|                 | CORS configured correctly      | □      |
| **Usability**   | Mobile responsive              | □      |
|                 | Browser compatible             | □      |
|                 | Content correct                | □      |
|                 | Accessibility features         | □      |

## 8. Maintenance and Support

### 8.1 Regular Maintenance Tasks

1. **Monitor Logs**:
   - Check error logs regularly for any issues
   - Set up log rotation to prevent logs from growing too large

2. **Database Backups**:
   - Configure regular database backups
   - Store backups in a secure location
   - Test backup restoration periodically

3. **File Backups**:
   - Back up application files regularly
   - Include configuration files and environment variables

4. **Update Dependencies**:
   - Keep Python and Node.js dependencies up to date
   - Test updates in a staging environment before applying to production

5. **SSL Certificate Renewal**:
   - Monitor SSL certificate expiration dates
   - Renew certificates before they expire

### 8.2 User Management

1. **Create Admin User**:
   - Use the default admin credentials to log in
   - Create additional admin users as needed
   - Update the default admin password

2. **User Permissions**:
   - Set up appropriate user roles and permissions
   - Regularly review user access

### 8.3 Performance Monitoring

1. **Server Resources**:
   - Monitor CPU, memory, and disk usage
   - Set up alerts for resource thresholds

2. **Application Performance**:
   - Monitor response times for key endpoints
   - Track page load times
   - Optimize database queries if needed

## 9. Troubleshooting

### 9.1 Common Issues and Solutions

#### 9.1.1 Database Connection Issues

**Issue**: Application cannot connect to the database
**Solutions**:
- Verify database credentials in the `.env` file
- Check that the database user has the correct permissions
- Ensure the PostgreSQL service is running
- Check for firewall rules blocking database connections

#### 9.1.2 SSL Certificate Issues

**Issue**: SSL certificates not working or showing errors
**Solutions**:
- Verify domain DNS is correctly configured
- Check that certificates are issued for the correct domains
- Ensure certificates are not expired
- Regenerate certificates if needed

#### 9.1.3 Application Startup Issues

**Issue**: Python or Node.js application fails to start
**Solutions**:
- Check application logs for specific errors
- Verify all dependencies are installed
- Ensure environment variables are correctly set
- Check file permissions
- Restart the application

#### 9.1.4 CORS Issues

**Issue**: Frontend cannot communicate with backend due to CORS errors
**Solutions**:
- Verify CORS_ORIGINS in the backend `.env` file includes the frontend domain
- Check that the frontend is using the correct API URL
- Ensure SSL is properly configured for both domains

### 9.2 Accessing Logs

#### 9.2.1 Backend Logs

```bash
# View error logs
tail -n 100 /home/formatdi/logs/error_log

# View application-specific logs (if configured)
tail -n 100 /home/formatdi/slavkoscore/app.log
```

#### 9.2.2 Frontend Logs

For frontend issues, check the browser console (F12) for JavaScript errors.

### 9.3 Getting Help

If you encounter issues that you cannot resolve:

1. Check the documentation in the deployment package
2. Review error logs for specific error messages
3. Search for solutions online based on the error messages
4. Contact technical support at support@formatdisc.hr

---

This guide provides comprehensive instructions for deploying the SlavkoScore™ Enterprise MVP to a cPanel hosting environment. Follow each step carefully to ensure a successful deployment.