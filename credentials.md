# Server Credentials

## SSH / SFTP Access
- **Server IP**: 192.168.3.135
- **Username**: gm
- **Password**: gm123
- **Base Project Directory**: `/home/gm` (e.g., Food Tracker is located at `/home/gm/food-tracker-prod/`)

## Database Access (PostgreSQL)
- **Database Host**: 192.168.3.135
- **Username**: GM
- **Password**: RathinamGM@32145

### Hosted Applications & Databases

**1. Food Tracker (Food Department)**
- **Database**: `food_tracker`
- **Local Directory**: `D:\FoodDepartment\`
- **Server Deployment Directory**: `/home/gm/food-tracker-prod/`
- **Volumes**: `/home/gm/food-tracker-prod/bills` (Used for storing receipt images)
- **Ports**: Frontend (8082), Backend (8000)

**2. Water Tracker (Rathinam Water Tracker)**
- **Database**: `water_tracker`
- **Local Directory**: `D:\RathinamWaterTracker\`
- **Server Deployment Directory**: `/home/gm/water-tracker-prod/`
- **Ports**: Frontend (8081), Backend (8080)

**3. Power Tracker**
- **Database**: `power_tracker` (Requires GM permission grant on Postgres)
- **Local Directory**: `D:\PowerTracker\`
- **Server Deployment Directory**: `/home/gm/power-tracker-prod/`
- **Ports**: Frontend (8084), Backend (8004)

**4. CCTV System**
- **Local Directory**: `C:\Users\janar\OneDrive\Desktop\cctv\`
- **Server Deployment Directory**: `/home/gm/cctv-tracker-prod/`
- **Database Routing**: Connects to Postgres via Docker bridge (`172.17.0.1`)
- **Volumes**: `cctv_media_volume`, `cctv_static_volume` (Media served directly via Nginx from `/home/gm/cctv-tracker-prod/media/`)
- **Ports**: Frontend (8083), Backend (5003)

**5. Maintenance Tracker**
- **Database**: `maintenance_tracker`
- **Local Directory**: `D:\Maintenance\`
- **Server Deployment Directory**: `/home/gm/maintenance-tracker-prod/`
- **Ports**: Frontend (8085), Backend (8005)

---

## NGINX Reverse Proxy Configuration (Gateway)

The main domain for all applications is **`https://insight.rathinamtechzone.com`**. 
Traffic is routed to individual Docker containers locally using an NGINX reverse proxy. 
*(Server configuration file is located at `/etc/nginx/sites-enabled/apps-gateway`)*

**Routing Rules:**
- **Water Tracker:** `/water_tracker/` ➔ Port `8081` (Default redirect from `/`)
- **Food Tracker:** `/food_tracker/` ➔ Port `8082`
- **Power Tracker Frontend:** `/power_tracker/` ➔ Port `8084`
- **Power Tracker API:** `/power_tracker/api/` ➔ Port `8004`
- **CCTV Frontend:** `/cctv/` ➔ Port `8083`
- **CCTV API / Admin / Media:** `/cctv/api/` (and admin/static/media) ➔ Port `5003`
- **Maintenance Tracker Frontend:** `/maintenance/` ➔ Port `8085`
- **Maintenance API / Admin:** `/maintenance/api/` (and admin/static) ➔ Port `8005`

---

## Application Architecture & Security Notes

### 1. Cross-App Authentication (LocalStorage Namespacing)
Because all the trackers (`Water`, `Food`, `Power`, `CCTV`) share the exact same top-level domain (`insight.rathinamtechzone.com`), they operate under the browser's **Same-Origin Policy**. This means they share the same `localStorage` database.

To prevent cross-app logouts (where logging into one app overwrites the authentication token of another), a **Key Namespacing** architecture is strictly enforced across all frontends:
* **Water Tracker**: Uses the `wt_` prefix (e.g., `wt_token`, `wt_user`)
* **Power Tracker**: Uses the `pt_` prefix (e.g., `pt_token`, `pt_user`)
* **Food Tracker**: Uses the `ft_` prefix (e.g., `ft_token`, `ft_user`)
* **Maintenance Tracker**: Uses the `mt_` prefix (e.g., `mt_token`, `mt_user`)
* **CCTV System**: Should logically use `cctv_` or similar prefixes to remain isolated.

### 2. Static & Media File Handling
File uploads and static assets require explicit routing through Nginx to bypass Django and improve performance.
* **Food Tracker Bills**: Uploaded receipts are saved directly to the host volume at `/home/gm/food-tracker-prod/bills`. The Nginx gateway explicitly intercepts `/food_tracker/bills/` to serve these directly and applies a `Cache-Control "no-cache"` header to ensure newly uploaded bills aren't hidden by browser caching.
* **CCTV Media**: Served directly from `/home/gm/cctv-tracker-prod/media/`.
