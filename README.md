Start to Start Masterclass Enrollment Platform
1. Project Overview
This document provides a comprehensive overview of the Start to Start Masterclass Enrollment Platform, a serverless web application designed for a gamified, dynamic enrollment experience.

The platform replaces traditional form-based systems with a modern, real-time solution featuring dynamic pricing, One-Time Password (OTP) verification for price locking, and a full-featured admin cockpit with AI-powered marketing and communication tools.

Key Features:

Dynamic Pricing: The first 10 seats for each masterclass are free. Prices then scale dynamically until seat 40, after which a fixed price applies.

Real-Time Counters: The public enrollment page displays live enrollment counts and progress bars.

OTP Price Lock: Users verify their email to lock in the current price for a 10-minute window, creating urgency and encouraging completion.

Administrative Cockpit: A secure, token-protected admin panel to view enrollments, manage seat counts, and export data.

AI-Powered Tools (Gemini): The admin panel includes an "AI Marketing Assistant" to generate promotional content and an "AI Email Personalizer" to draft welcome emails for new enrollees.

2. System Architecture
The application is built on a modern, serverless stack, designed for scalability, security, and near-zero operational cost.

Frontend: A static website built with HTML, Tailwind CSS, and vanilla JavaScript. It is hosted for free on GitHub Pages.

Backend API: A serverless function built as a Cloudflare Worker. It handles all business logic, database interactions, and integrations.

Database: A serverless SQL database, Cloudflare D1, which is based on SQLite.

Email Service: Secure, programmatic email sending for OTPs and admin notifications is handled via the Microsoft Graph API.

AI Service: Generative AI features are powered by the Google Gemini API.

3. Configuration Management
Proper configuration of secrets and variables is critical for the application to function. These are all set within the Cloudflare Worker's settings.

Location: Cloudflare Dashboard → Workers & Pages → st2s-enroll-worker → Settings → Variables

Variable Name

Type

Description

How to Obtain

DB

D1 Binding

Required. Connects the worker code to your D1 database, allowing it to read and write data.

In the worker settings, click "Add Binding," select "D1 Database," and choose your st2s-enroll-db.

ADMIN_TOKEN

Secret

Required. A secret password used to access the admin API endpoints.

Generate a strong, random string using a password manager or online generator.

CLIENT_ID

Secret

Required for email. The Application (client) ID for your Microsoft Entra App Registration.

From your app registration's "Overview" page in the Microsoft Entra admin center.

CLIENT_SECRET

Secret

Required for email. The client secret you generated within your Microsoft Entra App Registration.

From your app registration's "Certificates & secrets" page. Save this immediately as it cannot be viewed again.

TENANT_ID

Secret

Required for email. The Directory (tenant) ID for your Microsoft Entra instance.

From your app registration's "Overview" page in the Microsoft Entra admin center.

SENDER

Secret

Required for email. The email address of the shared mailbox you configured to send OTPs (e.g., enroll@starttostart.com).

The email address you created in the Microsoft 365 / Exchange admin center.



4. Deployment & Setup Guide
This guide covers setting up the entire application from scratch using a browser-based workflow.

Step 1: Set up Microsoft Entra for Email
Create a Shared Mailbox: In the Microsoft 365 Admin Center, create a shared mailbox (e.g., enroll@starttostart.com). This will be your SENDER.

Register an Application: Go to the Microsoft Entra admin center → Identity → Applications → App registrations → New registration.

Grant API Permissions: In your new app registration, go to "API permissions" → "Add a permission" → "Microsoft Graph" → "Application permissions". Search for Mail.Send and add it. You must then click the "Grant admin consent for..." button.

Get Credentials:

On the "Overview" page, copy the Application (client) ID (CLIENT_ID) and Directory (tenant) ID (TENANT_ID).

Go to "Certificates & secrets" → "New client secret". Copy the Value immediately (CLIENT_SECRET).

Step 2: Set up GitHub Pages (Frontend)
Create a new public repository on GitHub named st2s-enroll-site.

Upload the index.html, admin.html, and CNAME files to this repository.

In the repository settings, go to "Pages". Under "Build and deployment," set the Source to "Deploy from a branch" and the branch to main.

In the "Custom domain" section, enter enroll.starttostart.com and save.

Step 3: Set up Cloudflare D1 (Database)
In the Cloudflare dashboard, go to Workers & Pages → D1 SQL database → Create database. Name it st2s-enroll-db.

Go to the new database's Console tab.

Copy the entire content of the final schema.sql file and paste it into the console. Click Execute. This will create all the necessary tables.

Step 4: Set up Cloudflare Worker (Backend)
In the Cloudflare dashboard, go to Workers & Pages → Create application → Create Worker. Name it st2s-enroll-worker.

Click "Edit code" and paste the entire content of the final st2s-enroll-worker.js file. Click "Save and Deploy".

Go to your new worker's Settings → Variables.

Add D1 Database Binding: Click "Add Binding", use DB for the variable name, and select your st2s-enroll-db database.

Add Secrets: Add encrypted variables for ADMIN_TOKEN, CLIENT_ID, CLIENT_SECRET, TENANT_ID, and SENDER, pasting the values you obtained earlier.

5. User Guides
Enrollee User Guide (Public Page)
The public enrollment page (enroll.starttostart.com) is the main interface for users.

View Courses: The top section displays the available masterclasses, the current price for the next available seat, and a progress bar showing how many seats have been filled.

Fill Form: The user selects a masterclass and fills in their personal and company details.

Request OTP: Clicking "Request OTP to Lock Price" sends a 6-digit code to their email and starts a 10-minute price-lock timer.

Verify & Enroll: The user enters the OTP. Upon successful verification, their seat is confirmed at the locked price.

Administrator User Guide (Admin Page)
The admin page (enroll.starttostart.com/admin.html) is the control center for managing the platform.

Authentication: Enter the ADMIN_TOKEN and the GEMINI_API_KEY into the respective fields.

Load Data: Click "Load Data". This fetches the current enrollment counts and the list of all enrollees from the database.

Manage Counters: Use the -1 and +1 buttons under each course to manually correct enrollment numbers if needed.

View Enrollments: A real-time table displays all confirmed enrollments.

Export Data: Click "Export as CSV" to download a complete record of all enrollments.

AI Marketing Assistant: Click "Generate Promotional Post" to use Gemini to draft a marketing post for LinkedIn, focusing on the course with the most available seats.

AI Email Personalizer: In the enrollments table, click "Personalize Email ✨" next to any user to have Gemini draft a custom welcome email for them.

6. Database Schema Documentation
The D1 database (st2s-enroll-db) consists of four main tables:

counters
Stores the current number of confirmed enrollments for each masterclass.

mc (TEXT, PRIMARY KEY): The course identifier (e.g., "OPM", "AGP").

value (INTEGER): The total number of enrollments for that course.

seat_lock
Temporarily holds a user's information and locked price while they complete OTP verification.

id (TEXT, PRIMARY KEY): A unique UUID for the lock.

mc (TEXT): The course identifier.

email (TEXT): The user's email address.

seat (INTEGER): The seat number this lock is for (e.g., 26).

price (INTEGER): The price in cents locked for this user.

state (TEXT): The status of the lock ('active', 'confirmed', 'expired').

enrollment_id (TEXT): The ID of the final enrollment if confirmed.

created_at (TIMESTAMP): When the lock was created.

expires_at (INTEGER): The Unix timestamp when the lock expires.

otp
Stores the OTP codes sent to users.

id (TEXT, PRIMARY KEY): A unique UUID for the OTP record.

email (TEXT): The user's email address.

lock_id (TEXT): The seat_lock this OTP corresponds to.

code_hash (TEXT): A SHA-256 hash of the 6-digit code.

created_at (TIMESTAMP): When the OTP was generated.

expires_at (INTEGER): Unix timestamp when the OTP expires.

used_at (INTEGER): Unix timestamp when the OTP was successfully used.

attempts (INTEGER): The number of failed verification attempts.

enrollment
Stores the permanent record of a confirmed enrollment.

id (TEXT, PRIMARY KEY): A unique UUID for the enrollment.

mc (TEXT): The course identifier.

email (TEXT): The enrollee's email.

seat (INTEGER): The final seat number assigned.

price (INTEGER): The final price in cents.

created_at (TIMESTAMP): When the enrollment was confirmed.
