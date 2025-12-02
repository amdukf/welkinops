# ERR_TOO_MANY_REDIRECTS Issue with Cloudflare and How to Fix It

## 1. Problem Description

When the domain record `vikunja.amirkheirandish.ir` is behind **Cloudflare Proxy (Proxied, 🔶)**, the following error may appear in the browser:

```
This page isn’t working
vikunja.amirkheirandish.ir redirected you too many times.
ERR_TOO_MANY_REDIRECTS
```

This issue usually occurs due to an **infinite redirect loop between HTTP and HTTPS**.

---

## 2. Why This Happens

### 2.1 Request Flow Before Fix

1. User opens the browser and visits `https://vikunja.amirkheirandish.ir`
2. The request goes to Cloudflare Proxy (HTTPS)
3. Cloudflare in **Flexible SSL mode** sends the request to your server via **HTTP**
4. Your server (Nginx) sees the HTTP request and redirects to HTTPS
5. Cloudflare sends the same request again as HTTP
6. The server redirects again

> Result: The browser is caught in an **infinite redirect loop** and receives no page content.

### 2.2 Key Point

* A single HTTP → HTTPS redirect is fine.
* But when there is a **continuous loop between Cloudflare and the server**, the browser shows the `ERR_TOO_MANY_REDIRECTS` error and the page does not load.

---

## 3. Root Cause

1. **Cloudflare Flexible SSL was active**

   * HTTPS between the user and Cloudflare was working
   * Cloudflare sent HTTP to the server

2. **Server had permanent HTTP → HTTPS redirect**

   * Nginx always redirected HTTP to HTTPS

3. **Redirect loop was created**

   * The browser had no final path to reach the page

---

## 4. Solution

### 4.1 Configure Cloudflare

* Go to Cloudflare → SSL/TLS → choose **Full (Strict)**
* Reason: Cloudflare connects directly to your server's real HTTPS and validates your certificate
* Result: The server no longer performs unnecessary redirects → loop is eliminated

### 4.2 Configure Nginx

* Add the following headers so the server knows the actual protocol:

```nginx
proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
proxy_set_header X-Scheme $http_x_forwarded_proto;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header Host $host;
```

* This ensures Nginx detects HTTPS requests correctly and **does not create extra redirects**.

### 4.3 Check Ports

* Cloudflare proxies only **standard HTTP/HTTPS ports (80/443)**
* If the backend service runs on a custom port like 3456 → a **reverse proxy (Nginx/Traefik)** on port 443 is required

---

## 5. Request Flow After Fix

1. Browser → Cloudflare (Proxied, Full/Strict)
2. Cloudflare → Your server via real HTTPS
3. Server detects HTTPS → no extra redirect
4. Browser receives the page content

---

## 6. Summary

* The `ERR_TOO_MANY_REDIRECTS` issue was caused by **mismatch between the server’s HTTPS configuration and Cloudflare Flexible SSL**
* Solution:

  1. Use **Full (Strict) SSL** in Cloudflare
  2. Correctly configure Nginx headers (`X-Forwarded-Proto`)
  3. Ensure the reverse proxy runs on standard port 443

> After these changes, the page loads correctly without a
