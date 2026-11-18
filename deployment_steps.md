# Portal Page Deployment Guide

This document outlines the steps to deploy the Central Portal HTML page (`index.html` and `color_themes`) to your server (`192.168.3.135`). 

These steps assume you are running them from your local machine and have SSH access configured for the user `gm`.

## 1. Prepare the Server Directory
First, connect to the server and ensure the destination folder exists. You specified that the central portal files should be located in `/home/gm/portal-prod`.

Connect via SSH:
```bash
ssh gm@192.168.3.135
```

Create the directory (since you are logging in as `gm`, you don't need `sudo` for files in your home directory):
```bash
mkdir -p /home/gm/portal-prod
```

*Exit the SSH session or open a new local terminal for the next step.*

## 2. Upload the Portal Files
From your local machine, use SCP to securely copy `index.html` and the `color_themes/` directory to the server's directory we just created:

```bash
# Ensure you are in D:\PortalPage locally, or use absolute paths as shown:
scp -r D:\PortalPage\index.html D:\PortalPage\color_themes gm@192.168.3.135:/home/gm/portal-prod/
```

## 3. Verify NGINX Configuration
The central portal relies on NGINX serving the root path `/`. We need to ensure the NGINX configuration points to the correct directory `/home/gm/portal-prod`.

SSH back into the server:
```bash
ssh gm@192.168.3.135
```

Open the gateway config file:
```bash
sudo nano /etc/nginx/sites-enabled/apps-gateway
```

Scroll to the bottom and modify the Central Portal block so it points to `/home/gm/portal-prod`:
```nginx
    # ==========================================
    # CENTRAL PORTAL
    # ==========================================
    location / {
        root /home/gm/portal-prod;
        index index.html;
        try_files $uri $uri/ =404;
    }
```

## 4. Test and Restart NGINX
After confirming the configuration, test it to make sure there are no syntax errors, and then restart the NGINX service:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

## 5. Verification
1. Open your browser and navigate to `https://insight.rathinamtechzone.com/`.
2. The Central Portal Page should load successfully with all styles applied.
3. Verify that the routing to other applications (Water, Food, Power, CCTV) is unaffected by clicking their respective links.
