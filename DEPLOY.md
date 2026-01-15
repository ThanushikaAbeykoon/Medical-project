# How to Host Your Web App for Free with Firebase

Since you are already using Firebase for your database and authentication, **Firebase Hosting** is the best and easiest way to host your website. It is free (generous tier), fast, and secure (HTTPS).

## Prerequisites
You need **Node.js** installed on your computer. If you don't have it, download it from [nodejs.org](https://nodejs.org/).

## Step 1: Install Firebase Tools
Open your terminal (PowerShell or Command Prompt) and run:
```bash
npm install -g firebase-tools
```

## Step 2: Login to Firebase
Run this command and follow the browser prompt to log in with the Google account you used for your Firebase project:
```bash
firebase login
```

## Step 3: Initialize Project
1. Run this command in your project folder (`Medical-project`):
   ```bash
   firebase init hosting
   ```
2. **Select your project**: Choose `Use an existing project` and select `ecg-01-4efa0` (your current project).
3. **Public directory**: Type `frontend` and press Enter. (This is where your HTML files are).
4. **Configure as a single-page app?**: Type `No` (since you have multiple HTML files like login.html, dashboard.html).
5. **Set up automatic builds and deploys with GitHub?**: Type `No` (for now).
6. **File overrides**: If it asks to overwrite `index.html` or `404.html`, type `No` (N) so you don't lose your work.

## Step 4: Deploy!
Run this command to publish your site to the internet:
```bash
firebase deploy
```

## Done!
Firebase will give you a URL (e.g., `https://ecg-01-4efa0.web.app`). You can share this link with anyone!

### Why Host?
- **Fixes Issues**: The "CORS" errors and "Camera/Microphone" permission issues that happen when opening files locally (`file://`) will disappear.
- **Accessible**: You can open the link on your phone to test the mobile view.
- **Secure**: You get a secure "lock" icon (https) automatically.
