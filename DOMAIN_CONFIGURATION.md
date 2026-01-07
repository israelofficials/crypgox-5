# Domain Configuration Guide

This document explains where to configure multiple domains for your application to work across different domains (e.g., crypgox.com, www.crypgox.com, crypgox.cloud, etc.).

## ✅ Yes, you MUST run the migration first!

**Before creating admins, you MUST run:**
```sql
-- Run this migration first to create the admins table
backend/migrations/010_create_admins.sql
```

## Domain Configuration Locations

### 1. Backend Environment Variables (`.env` file in `backend/`)

These control CORS (Cross-Origin Resource Sharing) and cookie domains:

```env
# Primary frontend domain (used as default)
FRONTEND_ORIGIN=https://crypgox.com

# All allowed frontend domains (comma-separated)
# Include all variations: www, non-www, different TLDs, etc.
FRONTEND_ORIGINS=https://crypgox.com,https://www.crypgox.com,https://crypgox.cloud,https://www.crypgox.cloud

# Cookie domain (for sharing cookies across subdomains)
# Use .crypgox.com to share cookies across crypgox.com and www.crypgox.com
# Leave empty or don't set if domains are completely different
COOKIE_DOMAIN=.crypgox.com
```

**Important Notes:**
- `FRONTEND_ORIGIN`: Used as the default/primary domain
- `FRONTEND_ORIGINS`: All domains that can make API requests (CORS whitelist)
- `COOKIE_DOMAIN`: Only set if you want to share cookies across subdomains (e.g., `.crypgox.com` works for both `crypgox.com` and `www.crypgox.com`)
- If your domains are completely different (e.g., `crypgox.com` and `crypgox.cloud`), leave `COOKIE_DOMAIN` empty

### 2. Frontend Environment Variables (`.env.local` or `.env` in root)

These are used for API calls and fallback URLs:

```env
# Backend API URL
NEXT_PUBLIC_BACKEND_URL=https://api.crypgox.cloud

# Frontend URL (used as fallback for SSR and when window.location is not available)
NEXT_PUBLIC_FRONTEND_URL=https://www.crypgox.cloud
```

**Important Notes:**
- `NEXT_PUBLIC_*` variables are exposed to the browser
- These are used as fallbacks when `window.location` is not available (server-side rendering)
- The referral links now use `window.location.origin` dynamically, so they automatically work for any domain

### 3. Code Locations (Already Updated)

The following files have been updated to use the current domain dynamically:

- ✅ `src/app/exchange/invite/page.tsx` - Uses `window.location.origin`
- ✅ `src/app/me/referrals/page.tsx` - Uses `window.location.origin`
- ✅ `middleware.ts` - Uses `req.nextUrl.origin` (Next.js middleware)
- ✅ `src/app/login/page.tsx` - Uses `window.location.href` for redirects

## How It Works

### Referral Links
- **Client-side**: Uses `window.location.origin` - automatically uses the current domain
- **Server-side**: Falls back to `NEXT_PUBLIC_FRONTEND_URL` or hardcoded default

### Login Redirects
- Uses `window.location.href` which keeps users on the same domain
- No cross-domain redirects happen

### CORS
- Backend checks `FRONTEND_ORIGINS` to allow API requests
- All your domains must be listed here

### Cookies
- If `COOKIE_DOMAIN` is set (e.g., `.crypgox.com`), cookies are shared across subdomains
- If domains are completely different, leave it empty and cookies will be domain-specific

## Example Configuration for Multiple Domains

### Scenario: You have crypgox.com, www.crypgox.com, and crypgox.cloud

**Backend `.env`:**
```env
FRONTEND_ORIGIN=https://crypgox.com
FRONTEND_ORIGINS=https://crypgox.com,https://www.crypgox.com,https://crypgox.cloud,https://www.crypgox.cloud
COOKIE_DOMAIN=  # Leave empty since domains are different
```

**Frontend `.env.local`:**
```env
NEXT_PUBLIC_BACKEND_URL=https://api.crypgox.cloud
NEXT_PUBLIC_FRONTEND_URL=https://crypgox.com  # Used as fallback only
```

**Result:**
- Users on `crypgox.com` see referral links with `crypgox.com`
- Users on `www.crypgox.com` see referral links with `www.crypgox.com`
- Users on `crypgox.cloud` see referral links with `crypgox.cloud`
- All domains can make API requests (CORS allows them)
- Cookies are domain-specific (no sharing between .com and .cloud)

## Creating Admin Users

### Method 1: Using SQL (Recommended)

1. **Generate bcrypt hash** using one of these methods:

   **Option A: Online tool**
   - Go to https://bcrypt-generator.com/
   - Enter your password (e.g., `mypassword`)
   - Set rounds to 10
   - Copy the generated hash

   **Option B: Node.js one-liner**
   ```bash
   node -e "const bcrypt=require('bcrypt');bcrypt.hash('mypassword',10).then(h=>console.log(h))"
   ```

2. **Run the SQL** (see `backend/migrations/create_admin_simple.sql`):
   ```sql
   INSERT INTO admins (username, password_hash)
   VALUES (
     'admin',
     '$2b$10$YOUR_GENERATED_HASH_HERE'
   )
   ON CONFLICT (username) DO NOTHING;
   ```

### Method 2: Using the Script

```bash
node backend/scripts/create-admin.js admin mypassword
```

Then copy and run the generated SQL in your database.

## Summary Checklist

- [ ] Run `backend/migrations/010_create_admins.sql` migration
- [ ] Configure `FRONTEND_ORIGINS` in backend `.env` with all your domains
- [ ] Configure `NEXT_PUBLIC_FRONTEND_URL` in frontend `.env` (fallback only)
- [ ] Create admin user using SQL or script
- [ ] Test referral links on each domain
- [ ] Test login/logout on each domain
