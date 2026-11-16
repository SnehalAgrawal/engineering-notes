# Cloud Architecture: NGINX (Web Server & Reverse Proxy)

### 1. Overview
NGINX is a web server most often used as a reverse proxy. It sits in front of your app, handles HTTPS, serves static files, and forwards other requests to the app. This is a working reference for a Linux host. For Kubernetes, see [kubernetes-networking.md](kubernetes-networking.md).

### 2. Key Concepts
*   **Blocks**: Config is nested. `http` holds `server` blocks (one per site), and each `server` holds `location` blocks (one per URL path).
*   **http-only directives**: `map`, `upstream`, `log_format`, and `proxy_cache_path` go in the `http` block, not inside `server`.
*   **Reload vs. restart**: `reload` starts new workers with the new config while old ones finish their requests, so nothing is dropped. `restart` stops everything and cuts open connections.
*   **File layout**: Debian and Ubuntu use `/etc/nginx/sites-available/` plus symlinks in `sites-enabled/`. Other packages use `/etc/nginx/conf.d/*.conf`. Logs are in `/var/log/nginx/`.

### 3. Everyday Commands

```bash
sudo nginx -t                  # check the config; run before every reload
sudo systemctl reload nginx
sudo systemctl restart nginx
sudo systemctl status nginx
sudo nginx -T                  # print the full config, including all includes
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/   # enable a site (Debian/Ubuntu)
sudo rm /etc/nginx/sites-enabled/myapp                                   # disable it
```

### 4. Reverse Proxy for a Frontend and an API
The frontend build is in `/var/www/html`. The API runs on port 3000 and is served at `/api/`.

```nginx
# Goes in the http block. Sends "Connection: upgrade" only for WebSocket requests.
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80;
    server_name example.com;
    root /var/www/html;

    location / {
        try_files $uri $uri/ /index.html;   # let the frontend router handle unknown paths
    }

    location /api/ {
        proxy_pass http://127.0.0.1:3000/;  # trailing slash: /api/users reaches the app as /users
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
    }
}
```

*   **Trailing slash**: With `proxy_pass http://127.0.0.1:3000;` (no slash) the app gets the full path, `/api/users`.
*   **WebSockets**: They need HTTP/1.1 and the `Upgrade` and `Connection` headers. NGINX closes a connection that is silent for `proxy_read_timeout` (60s by default), so raise it or send pings.

### 5. HTTPS with Let's Encrypt

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d example.com -d www.example.com
sudo certbot renew --dry-run   # test automatic renewal
```

*   **Before you start**: DNS must point to the server, and ports 80 and 443 must be open.
*   **What Certbot does**: It proves you own the domain, saves the certificate in `/etc/letsencrypt/live/<domain>/`, adds `listen 443 ssl` and the certificate paths to your server block, adds an HTTP-to-HTTPS redirect if you choose it, and reloads NGINX.

The HTTPS part of the server block then looks roughly like this:

```nginx
server {
    listen 443 ssl;
    server_name example.com www.example.com;
    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    # ... same root and location blocks as before
}
```

*   **Renewal**: The package installs a timer or cron job. Check with `systemctl list-timers | grep certbot`.

### 6. Caching
Browser caching for static files. Long times are safe only when file names change with content (hashed build files).

```nginx
location ~* \.(?:css|js|png|jpg|svg|woff2)$ {
    expires 30d;
    access_log off;
}
```

Caching API responses in NGINX:

```nginx
# http block
proxy_cache_path /var/cache/nginx/api levels=1:2 keys_zone=api_cache:10m max_size=1g inactive=60m;

# inside the server block
location /api/ {
    proxy_pass http://127.0.0.1:3000;
    proxy_cache api_cache;
    proxy_cache_valid 200 10m;
    proxy_cache_valid 404 1m;
    proxy_cache_use_stale error timeout updating;   # serve old copy if the app is down
    add_header X-Cache-Status $upstream_cache_status;
}
```

*   **`keys_zone`**: Shared memory for cache keys and metadata. The responses themselves are on disk.
*   **`levels=1:2`**: Splits cache files into subfolders so no folder gets huge.
*   **Microcaching**: Caching for about one second (`proxy_cache_valid 200 1s;`) turns a burst of identical requests into one call to the app.
*   **Per-user data**: The cache key is `$scheme$proxy_host$request_uri` by default. It ignores cookies and `Authorization`, so do not cache personal responses unless the key includes the user. NGINX already skips responses with `Set-Cookie` or `Cache-Control: private`, `no-cache`, or `no-store`.

### 7. Load Balancing

```nginx
upstream app {
    least_conn;                                     # default is round robin
    server 127.0.0.1:3000 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:3001 max_fails=3 fail_timeout=30s;
}
```

*   **Use it**: `proxy_pass http://app;` in a `location`.
*   **`least_conn`**: Sends each request to the server with the fewest open connections. Good when request times vary.
*   **Health checks**: Open-source NGINX only has passive checks. After `max_fails` errors within `fail_timeout`, it skips that server for `fail_timeout`. Active checks are an NGINX Plus feature.

### 8. Logging and Monitoring
JSON logs are easy to search in Loki, Elasticsearch, or a SaaS tool:

```nginx
# http block
log_format json_combined escape=json
  '{"time":"$time_iso8601","remote_addr":"$remote_addr","request":"$request",'
  '"status":$status,"request_time":$request_time,'
  '"upstream_time":"$upstream_response_time","user_agent":"$http_user_agent"}';

# server block: separate log files per site
access_log /var/log/nginx/myapp_access.json json_combined;
error_log  /var/log/nginx/myapp_error.log warn;
```

*   **Where time goes**: `$request_time` is the total time. `$upstream_response_time` is the time spent in the app.
*   **Metrics**: The `stub_status` module exposes connection counts. The NGINX Prometheus exporter turns them into Prometheus metrics.

### 9. Common Mistakes
*   **Skipping `nginx -t`**: A restart with a broken config leaves the site down.
*   **Always sending `Connection: upgrade`**: Use the `map` above instead.
*   **Losing the client IP**: Without `X-Forwarded-For`, the app sees every request as coming from `127.0.0.1`.
*   **Caching logged-in pages** under a key that does not include the user.

### Further Watching
*   NGINX tutorial video: https://www.youtube.com/watch?v=ZmH1L1QeNHk
*   NGINX tutorial video: https://www.youtube.com/watch?v=7VAI73roXaY
