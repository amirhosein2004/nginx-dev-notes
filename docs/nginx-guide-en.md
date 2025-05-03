# Nginx Tutorial

## Introduction
Nginx is a lightweight web server for handling requests. This software is used for installation as a reverse proxy and load balancing, which is among the most popular (receiving user requests, managing them, and responding to the server).

## Differences between Apache and Nginx
- Apache is older than Nginx and can use XML.
- Apache uses htaccess files for configuration.
- Nginx has many configuration capabilities and has higher speed compared to Apache.
- Apache displays index.html files by default, but Nginx must be manually configured.

## Installation and Setup

### Installing Nginx
To install Nginx in Ubuntu, use the following command:
```bash
apt-get install nginx
```

### Checking Server Status
To check if the server is active, use the following command:
```bash
ps aux | grep nginx
```

### Important Notes
- Internal IPs in containers are private (like: 172.17.x.x), meaning they are not accessible globally (e.g., from the internet).
- To connect from a container to the local system on port 80 for testing Nginx locally:
```bash
docker run ... -p 8080:80 ...
```
- The Nginx configuration file is located in the `/etc/nginx/` directory.
- To start Nginx, simply run the `nginx` command in the terminal.

## Service Management with systemd

systemd is a service that automatically runs programs. To create an Nginx service in systemd:

### Creating a Service File
Place the config file in `/etc/systemd/system/nginx.service`:

```ini
[Unit]
# General description of the service and what this service should start after

[Service]
# How to start, stop, and reload the service
PIDFile = /var/run/nginx.pid        # File that stores the process ID
ExecStartPre = /usr/bin/nginx -t    # Path to the nginx binary and config test
ExecStart = /usr/bin/nginx          # Path to the nginx binary for execution

[Install]
# For what purpose this service is activated
```

### Working with the Service
```bash
# Running the service
systemctl start nginx

# Checking service status
systemctl status nginx

# Enabling automatic startup at system boot
systemctl enable nginx

# Disabling automatic startup
systemctl disable nginx
```

### Working with Nginx without systemd
```bash
# Starting the service
nginx

# Stopping the service
nginx -s stop

# Reloading configurations
nginx -s reload
```

## Nginx Configuration

### Main Configuration File
The Nginx configuration file is located at `/etc/nginx/nginx.conf` and includes three main sections:

1. **event**: Connection commands like the number of simultaneous users
2. **http**: Related to the HTTP service (settings such as mimetype, cache, and compression)
3. **main**: General settings for the Nginx service like worker process

### Server Block
The server block or virtual host is within the http section and can have multiple server blocks:

```nginx
server {
    listen 80;              # Port 80 is for HTTP (without encryption), port 443 for HTTPS
    server_name example.com;    # Domain name or IP address
    root /sites/demo;       # Root path for website files
}
```

To check syntax errors in the configuration, you can use the `nginx -t` command.

### MIME Type Settings
To define content types, you can use the types block:

```nginx
types {
    text/html html;
    text/css css;
}
```

Or simply use the ready-made standard file:

```nginx
include mime.types;
```

## Location Directive

Location is used to match requested URLs and determine a path for them:

```nginx
location [modifier] path {
    # Settings
}
```

This block is written within the server block and has different types:

1. **Prefix**: Any address that starts with the specified path
   ```nginx
   location /greet {
       # Any address starting with /greet like /greeting etc.
   }
   ```

2. **Exact Match**: Any address that exactly matches the specified path
   ```nginx
   location = /greet {
       # Only the exact address /greet
   }
   ```

3. **Regular Expression Match**: For regular expressions
   ```nginx
   location ~ /greet[0-9] {
       # Case-sensitive
   }
   
   location ~* /greet[0-9] {
       # Case-insensitive
   }
   ```

4. **Preferential Prefix**: Higher priority than regular expression
   ```nginx
   location ^~ /prefix {
       # If a path like prefix/js/name is given, it will go to this path
   }
   ```

## Variables and Conditions

### Variables
Nginx has variables similar to programming languages:

1. **Default variables**: Start with $
   ```
   $host: Host name (domain)
   $uri: Requested path
   $args: All query strings
   ```

2. **User variables**: Defined with set
   ```nginx
   set $weekend "no";
   ```

### Conditional Statements
```nginx
if ($arg_api_key != "1234") {
    # Operations
}

if ($date_local ~ "Saturday|Sunday") {
    set $weekend "yes";
}
```

## Rewriting and Path Redirection

### rewrite
For changing the internal request path on the server (no change in the URL):

```nginx
rewrite ^/user/(\w+)$ /greet/$1 last;
```

### return
For quickly returning a response to the client:

```nginx
return 301 https://example.com$request_uri;
```

### Comparison of rewrite and return

| Command | rewrite | return |
|---------|---------|--------|
| Variable type | Internal rewrite | redirect |
| Simplicity | Flexible but complex | Simple and direct |
| Change URL | No | Yes |
| Suitable for | Internal redirection without user knowledge | Redirection to a new URL |
| Regex support | Yes | No |

### rewrite flags
- **last**: Rewriting is done and goes to the new location
- **break**: Rewriting is done, but the new location is not checked
- **redirect**: Causes a temporary redirect
- **permanent**: Like redirect but permanent (301)

### capture groups
A part of the address specified with regex:
```nginx
rewrite ^/user/(\w+)$ /greet/$1 last;
```

### try_files
Checks different paths in order:
```nginx
try_files $uri $uri/ /index.php?$args;
```

## Logging System

Nginx has two main types of logs:

1. **Access Log**: All requests sent to the server (addresses, user IP, etc.)
2. **Error Log**: Errors occurring in the system (config issues, permissions, etc.)

Both log files are enabled by default and are located in the `/var/log/nginx` directory.

### Uses
- Error checking
- Request tracking
- Identifying suspicious users
- Resource optimization

### Log Settings
```nginx
# Disabling logging
access_log off;

# Setting a custom path for logs
location /secure {
    return 200 "Welcome to secure area";
    access_log /var/log/nginx/secure-access.log;
}
```

## Inheritance in Settings

In Nginx, like programming languages, internal contexts inherit their settings from higher contexts:

```
main 
    |
     ----- http
              |
              ---- server
                        |
                        ----- location
```

### Types of Directives
1. **Array Directives**: Can be used multiple times in a context (like access_log, error_page, add_header)
2. **Standard Directives**: Can only be defined once in a context (like root, index, client_max_body_size)
3. **Action Directives**: Directives that cause a specific operation and don't inherit (like return, rewrite)

## Processing in Nginx

Nginx itself doesn't run server-side languages like PHP or Python (Django). For this, we need to run a separate service:

```nginx
location /static/ {
    alias /path/to/your/static/;        
}

location / {
    proxy_pass http://127.0.0.1:8000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

### Nginx Processes
- **Master Process**: The main process that manages Nginx
- **Worker Process**: Work processes that handle user requests

```nginx
# Setting the number of worker processes (best to equal the number of CPU cores)
worker_processes auto;

# Setting the number of simultaneous connections for each worker
worker_connections 1024;
```

## Buffers and Timeouts

### Key Settings

| Command | Usage | Suggested Value | Important Note |
|---------|-------|----------------|----------------|
| client_body_buffer_size | Temporary storage for POST data | 10K | If too high, memory is wasted; if too low, it goes to disk (slow) |
| client_max_body_size | Maximum allowed size for POST | 8M | If exceeded, returns a 413 error |
| client_header_buffer_size | Memory for request headers | 1K | Sufficient for most applications |
| client_body_timeout | Wait time to receive the entire body | 12s | If the client is slow, the server won't wait |
| client_header_timeout | Wait time to receive headers | 12s | Similar to above but only for headers |
| keepalive_timeout | Duration of connection maintenance | 15s | If too high, resources are wasted; if too low, the connection is constantly disconnected |
| send_timeout | Wait time for data transmission | 10s | Prevents resource occupation in inefficient clients |
| sendfile on | Send files directly from disk | --- | High speed for large files (without using RAM) |
| tcp_nopush on | TCP packet optimization | --- | For better sending of large files |

## Modules

Modules in Nginx are components that add additional capabilities to the web server:

### Types of Modules
1. **Static**: Added during Nginx compilation and cannot be separated later (like ngx_http_ssl_module)
2. **Dynamic**: Compiled separately and loaded at runtime (like ngx_http_image_filter_module)

### Module Categories
- **Core modules**: For main management and basic structures
- **HTTP**: For the HTTP protocol
- **Stream**: For TCP/UDP
- **Mail**: For email protocols like IMAP or SMTP

### Installing and Activating a Dynamic Module
1. Get previous compilation options: `nginx -v`
2. Install prerequisites
3. Download external modules
4. Compile the module (with the `make module` command)
5. Copy the output `.so` file
6. Add this line to `nginx.conf` (before the http block):
   ```nginx
   load_module module/name_module.so;
   ```

## Expires Headers and Caching

Expires Headers tell the browser how long it can keep a specific response in the cache:

```nginx
location ~* \.(?:css|js|jpg|jpeg|png)$ {
    expires 30d;                       # Cache for 30 days
    add_header Cache-Control "public"; # Tells the browser it can cache this file
    add_header Pragma "public";        # For older browsers
    add_header Vary "Accept-Encoding"; # Compatibility with compression
}
```

### Important Notes
- When a file is cached, the browser doesn't request it again for a specified period
- To solve the problem of changes in cached files, we use versioning in the URL (e.g., `file.css?v=1`)
- Nginx caches are usually stored on the server's disk:
  ```nginx
  proxy_cache_path /path/to/disk;
  ```

## Gzip Compression

Gzip allows compressing content before sending it to the browser:

```nginx
# Enabling gzip
gzip on;

# Compression level (1-9)
gzip_comp_level 3;

# MIME types for compression
gzip_types text/plain text/css application/javascript;
```

## Micro Cache

Caching dynamic responses for a very short time, reducing server load:

```nginx
# Setting cache path
fastcgi_cache_path /tmp/nginx_cache levels=1:2 keys_zone=zone1:100m inactive=60m use_temp_path=off;

# Defining the cache key
fastcgi_cache_key "$scheme$request_method$host$request_uri";
```

## HTTP/2

HTTP/2 is a binary protocol, while HTTP/1 is text-based:

### HTTP/2 Advantages
- Binary data is a more compressed method for transferring information
- Persistent connection
- Multiplexing: All files are received from the server in one connection
- Server Push: The browser can be informed of other files simultaneously with the initial request

### Note
HTTP/2 only works on secure connections (HTTPS) and requires an SSL certificate.

### Installing HTTP/2 and SSL Certificate
To enable HTTP/2 in the server block:

```nginx
listen 443 ssl http2;
ssl_certificate /etc/nginx/ssl/self.crt;
ssl_certificate_key /etc/nginx/ssl/self.key;
```

## Redirecting HTTP to HTTPS

For automatically redirecting users from HTTP to HTTPS:

```nginx
server {
    listen 80;
    return 301 https://$host$request_uri;
}
```

## Enhancing HTTPS Security

```nginx
# Disabling old SSL protocols
ssl_protocols TLSv1.2 TLSv1.3;

# Setting cipher suites
ssl_ciphers 'HIGH:!aNULL:!MD5';
ssl_prefer_server_ciphers on;

# Enabling DH parameters
ssl_dhparam /etc/nginx/ssl/dhparam.pem;

# Enabling HSTS
add_header Strict-Transport-Security "max-age=31536000" always;

# Caching SSL sessions
ssl_session_cache shared:SSL:40m;
ssl_session_timeout 4h;
ssl_session_tickets on;
```

## Rate Limiting

Limiting request rates to protect the server:

```nginx
# Defining a zone
limit_req_zone $request_uri zone=myzone:10m rate=60r/m;

# Applying the limit
limit_req zone=myzone burst=5 nodelay;
```

## Basic Authentication

Adding a simple security layer with username and password:

```nginx
location /admin/ {
    auth_basic "Secure Area";
    auth_basic_user_file /etc/nginx/.htpasswd;
}
```

## Increasing Nginx Security

1. Updating Nginx and libraries
2. Hiding the Nginx version:
   ```nginx
   server_tokens off;
   ```
3. Preventing clickjacking:
   ```nginx
   add_header X-Frame-Options "SAMEORIGIN";
   ```
4. Preventing XSS:
   ```nginx
   add_header X-XSS-Protection "1; mode=block";
   ```
5. Removing unnecessary modules during compilation

## Let's Encrypt

A free service for issuing SSL certificates:

```bash
# Installing Certbot
sudo apt-get install certbot python3-certbot-nginx

# Getting a certificate
sudo certbot --nginx
```

## Reverse Proxy and Load Balancing

### Reverse Proxy
Connects incoming requests from the client to the backend server:

```nginx
location / {
    proxy_pass http://localhost:9000;
}
```

### Load Balancing
Distributing incoming requests among multiple servers:

```nginx
http {
    upstream backend_servers {
        server localhost:1001;
        server localhost:1002;
        server localhost:1003;
    }
    
    server {
        location / {
            proxy_pass http://backend_servers;
        }
    }
}
```

### Different Load Balancing Methods
1. Sticky Session (IP Hash)
2. Least Connection 