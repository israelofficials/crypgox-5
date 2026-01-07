# Cookie Behavior Across Multiple Domains

## ✅ Yes, Cookies Will Work on All Domains!

**Important:** You only need **ONE** `COOKIE_DOMAIN` setting: `COOKIE_DOMAIN=.crypgox.cloud`

The code automatically detects which domain the request is coming from and sets cookies appropriately:
- For `.cloud` domains → uses `COOKIE_DOMAIN` for subdomain sharing
- For other domains (`.com`, `.net`, `.online`, `.org`) → automatically domain-specific (no cookieDomain set)

## How It Works

### For `.cloud` domains:
- `COOKIE_DOMAIN=.crypgox.cloud` allows cookies to be **shared** between:
  - ✅ `crypgox.cloud`
  - ✅ `www.crypgox.cloud`
  - ✅ `api.crypgox.cloud` (if needed)

**Result:** If a user logs in on `crypgox.cloud`, they're also logged in on `www.crypgox.cloud` (same session).

### For other TLDs (`.com`, `.net`, `.online`, `.org`):
- Cookies are **domain-specific** (browser security feature)
- Each domain gets its own cookies:
  - `crypgox.com` → separate login session
  - `crypgox.net` → separate login session
  - `crypgox.online` → separate login session
  - `crypgox.org` → separate login session

**Result:** Users need to login separately on each TLD, but **each domain works perfectly**.

## Why This Happens

Browsers **cannot** share cookies across different TLDs (Top-Level Domains) for security reasons:
- `.com` ≠ `.net` ≠ `.online` ≠ `.org` ≠ `.cloud`
- This is a **browser security feature** and cannot be changed

## What This Means for Your Users

✅ **All domains work correctly:**
- Users can login on `crypgox.com` → works perfectly
- Users can login on `crypgox.net` → works perfectly
- Users can login on `crypgox.online` → works perfectly
- Users can login on `crypgox.org` → works perfectly
- Users can login on `crypgox.cloud` → works perfectly

✅ **Subdomain sharing (`.cloud` only):**
- Login on `crypgox.cloud` → also logged in on `www.crypgox.cloud`
- Login on `www.crypgox.cloud` → also logged in on `crypgox.cloud`

⚠️ **Cross-TLD limitation:**
- Login on `crypgox.com` → **NOT** logged in on `crypgox.net` (separate sessions)
- This is **correct and secure** behavior

## How It Works (Technical)

The backend code now **automatically detects** the request origin:

1. **If request comes from `.cloud` domain** (e.g., `crypgox.cloud`, `www.crypgox.cloud`):
   - Sets cookie with `domain=.crypgox.cloud`
   - Enables subdomain sharing ✅

2. **If request comes from other TLDs** (e.g., `crypgox.com`, `crypgox.net`):
   - Does **NOT** set `domain` in cookie options
   - Browser automatically makes it domain-specific ✅

## Summary

**You only need ONE `COOKIE_DOMAIN` setting!**

✅ **Configuration:**
```env
COOKIE_DOMAIN=.crypgox.cloud
```

✅ **Result:**
- `.cloud` domains → cookies shared across subdomains
- `.com`, `.net`, `.online`, `.org` → automatically domain-specific
- All domains work perfectly - no additional configuration needed!

The code handles everything automatically based on the request origin. 🎉
