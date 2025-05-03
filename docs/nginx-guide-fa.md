# آموزش Nginx

## مقدمه
Nginx یک وب سرور بسیار سبک برای سرویس دادن به درخواست‌ها می‌باشد. این نرم‌افزار برای نصب به‌عنوان reverse proxy و load balancing استفاده می‌شود که از محبوب‌ترین‌ها می‌باشد (دریافت درخواست‌های کاربران، مدیریت آنها و پاسخ دادن به سرور).

## تفاوت‌های Apache و Nginx
- Apache قدیمی‌تر از Nginx است و می‌تواند از XML استفاده کند.
- Apache برای پیکربندی از فایل‌های htaccess استفاده می‌کند.
- Nginx قابلیت‌های زیادی برای پیکربندی دارد و سرعت بالاتری نسبت به Apache دارد.
- Apache به صورت پیش‌فرض فایل‌های index.html را نمایش می‌دهد ولی Nginx باید به صورت دستی پیکربندی شود.

## نصب و راه‌اندازی

### نصب Nginx
برای نصب Nginx در Ubuntu دستور زیر را می‌نویسیم:
```bash
apt-get install nginx
```

### بررسی وضعیت سرور
برای اینکه ببینیم سرور فعال است یا نه از دستور زیر استفاده می‌کنیم:
```bash
ps aux | grep nginx
```

### نکات مهم
- IP های داخلی در کانتینر private هستند (مثل: 172.17.x.x) یعنی خارج از دسترسی جهانی (مثلاً اینترنت) هستند.
- برای اینکه از کانتینر به سیستم لوکال وصل شویم در پورت ۸۰ برای تست Nginx در لوکال خودمان باید:
```bash
docker run ... -p 8080:80 ...
```
- فایل پیکربندی Nginx در داخل مسیر `/etc/nginx/` قرار دارد.
- برای استارت کردن Nginx کافی است در ترمینال دستور `nginx` را اجرا کنیم.

## مدیریت سرویس با systemd

systemd سرویسی است که برنامه‌ها را به صورت خودکار اجرا می‌کند. برای ایجاد یک سرویس Nginx در systemd:

### ایجاد فایل سرویس
فایل config را در مسیر `/etc/systemd/system/nginx.service` قرار می‌دهیم:

```ini
[Unit]
# توضیح کلی سرویس و اینکه این سرویس بعد از چه چیزی باید شروع شود

[Service]
# چطور سرویس را start، stop و reload کنیم
PIDFile = /var/run/nginx.pid        # فایلی که process ID را ذخیره می‌کند
ExecStartPre = /usr/bin/nginx -t    # مسیر باینری nginx و تست کانفیگ
ExecStart = /usr/bin/nginx          # مسیر باینری nginx برای اجرا

[Install]
# برای چه هدفی این سرویس فعال می‌شود
```

### کار با سرویس
```bash
# اجرای سرویس
systemctl start nginx

# بررسی وضعیت سرویس
systemctl status nginx

# فعال کردن اجرای خودکار در هنگام راه‌اندازی سیستم
systemctl enable nginx

# غیرفعال کردن اجرای خودکار
systemctl disable nginx
```

### کار با Nginx بدون systemd
```bash
# شروع سرویس
nginx

# توقف سرویس
nginx -s stop

# بارگذاری مجدد تنظیمات
nginx -s reload
```

## تنظیمات Nginx

### فایل اصلی تنظیمات
فایل تنظیمات Nginx در مسیر `/etc/nginx/nginx.conf` قرار دارد و شامل سه بخش اصلی است:

1. **event**: دستورات اتصالات مثل تعداد کاربران همزمان
2. **http**: مربوط به سرویس HTTP (تنظیمات مانند mimetype، کش و فشرده‌سازی)
3. **main**: تنظیمات عمومی سرویس Nginx مانند worker process

### Server Block
سرور بلاک یا virtual host درون بخش http قرار می‌گیرد و می‌تواند چندین server بلاک داشته باشد:

```nginx
server {
    listen 80;              # پورت ۸۰ مخصوص HTTP (بدون رمزنگاری)، پورت ۴۴۳ برای HTTPS
    server_name example.com;    # نام دامنه یا آدرس IP
    root /sites/demo;       # مسیر ریشه فایل‌های وب‌سایت
}
```

برای بررسی خطاهای syntax در تنظیمات می‌توان از دستور `nginx -t` استفاده کرد.

### تنظیمات MIME Type
برای تعریف انواع محتوا می‌توان از بلاک types استفاده کرد:

```nginx
types {
    text/html html;
    text/css css;
}
```

یا به سادگی از فایل آماده استاندارد استفاده کرد:

```nginx
include mime.types;
```

## دستور Location

Location برای تطبیق URL‌های درخواست شده و تعیین مسیر برای آنها استفاده می‌شود:

```nginx
location [modifier] path {
    # تنظیمات
}
```

این بلاک درون بلاک server نوشته می‌شود و انواع مختلفی دارد:

1. **Prefix**: هر آدرسی که با مسیر مشخص شده شروع شود
   ```nginx
   location /greet {
       # هر آدرسی که با /greet شروع شود مثل /greeting و...
   }
   ```

2. **Exact Match**: هر آدرسی که دقیقاً برابر مسیر مشخص شده باشد
   ```nginx
   location = /greet {
       # فقط آدرس دقیق /greet
   }
   ```

3. **Regular Expression Match**: برای عبارات منظم
   ```nginx
   location ~ /greet[0-9] {
       # حساس به حروف بزرگ و کوچک
   }
   
   location ~* /greet[0-9] {
       # غیرحساس به حروف بزرگ و کوچک
   }
   ```

4. **Preferential Prefix**: اولویت بالاتر نسبت به regular expression
   ```nginx
   location ^~ /prefix {
       # اگر مسیری مانند prefix/js/name را هم بدهیم به این مسیر می‌رود
   }
   ```

## متغیرها و شرط‌ها

### متغیرها
Nginx شبیه زبان‌های برنامه‌نویسی متغیرهایی دارد:

1. **متغیرهای پیش‌فرض**: با $ شروع می‌شوند
   ```
   $host: نام هاست (دامنه)
   $uri: مسیر درخواستی
   $args: تمام query string ها
   ```

2. **متغیرهای کاربر**: با set تعریف می‌شوند
   ```nginx
   set $weekend "no";
   ```

### دستورات شرطی
```nginx
if ($arg_api_key != "1234") {
    # عملیات
}

if ($date_local ~ "Saturday|Sunday") {
    set $weekend "yes";
}
```

## بازنویسی و هدایت مسیر

### rewrite
برای تغییر مسیر داخلی درخواست در سرور (در URL تغییری نمی‌دهد):

```nginx
rewrite ^/user/(\w+)$ /greet/$1 last;
```

### return
برای برگرداندن سریع پاسخ به کلاینت:

```nginx
return 301 https://example.com$request_uri;
```

### مقایسه rewrite و return

| دستور | rewrite | return |
|-------|---------|--------|
| نوع متغیر | Internal rewrite | redirect |
| سادگی | انعطاف‌پذیر اما پیچیده | ساده و مستقیم |
| تغییر URL | خیر | بله |
| مناسب برای | هدایت داخلی بدون اطلاع کاربر | هدایت به URL جدید |
| پشتیبانی regex | بله | خیر |

### فلگ‌های rewrite
- **last**: بازنویسی انجام شده و به location جدید می‌رود
- **break**: بازنویسی انجام می‌شود اما location جدید بررسی نمی‌شود
- **redirect**: باعث ریدایرکت موقتی می‌شود
- **permanent**: ریدایرکت دائمی (301)

### capture groups
بخشی از آدرس که با regex مشخص می‌شود:
```nginx
rewrite ^/user/(\w+)$ /greet/$1 last;
```

### try_files
مسیرهای مختلف را به ترتیب بررسی می‌کند:
```nginx
try_files $uri $uri/ /index.php?$args;
```

## سیستم لاگ‌گیری

Nginx دو نوع اصلی لاگ دارد:

1. **Access Log**: تمام درخواست‌هایی که به سرور ارسال می‌شوند (آدرس‌ها، IP کاربر و...)
2. **Error Log**: خطاهای رخ داده در سیستم (مشکلات کانفیگ، دسترسی و...)

هر دو فایل لاگ به صورت پیش‌فرض فعال هستند و در مسیر `/var/log/nginx` قرار دارند.

### کاربردها
- بررسی خطاها
- ردیابی درخواست‌ها
- شناسایی کاربران مشکوک
- بهینه‌سازی منابع

### تنظیمات لاگ
```nginx
# غیرفعال کردن لاگ‌گیری
access_log off;

# تنظیم مسیر سفارشی برای لاگ
location /secure {
    return 200 "Welcome to secure area";
    access_log /var/log/nginx/secure-access.log;
}
```

## ارث‌بری در تنظیمات

در Nginx مانند زبان‌های برنامه‌نویسی، context‌های داخلی تنظیمات خود را از context‌های بالاتر به ارث می‌برند:

```
main 
    |
     ----- http
              |
              ---- server
                        |
                        ----- location
```

### انواع دستورات
1. **Array Directives**: می‌توان چند بار در یک context استفاده کرد (مثل access_log، error_page، add_header)
2. **Standard Directives**: فقط یکبار در یک context تعریف می‌شوند (مثل root، index، client_max_body_size)
3. **Action Directives**: دستوراتی که باعث اجرای یک عملیات خاص می‌شوند و ارث‌بری ندارند (مثل return، rewrite)

## پردازش در Nginx

Nginx خودش زبان‌های سمت سرور مانند PHP یا Python (Django) را اجرا نمی‌کند. برای این کار باید یک سرویس مستقل اجرا کنیم:

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

### فرآیندهای Nginx
- **Master Process**: فرآیند اصلی که Nginx را مدیریت می‌کند
- **Worker Process**: فرآیندهای کاری که درخواست‌های کاربران را پردازش می‌کنند

```nginx
# تنظیم تعداد worker process (بهتر است برابر با تعداد هسته‌های CPU باشد)
worker_processes auto;

# تنظیم تعداد اتصالات همزمان برای هر worker
worker_connections 1024;
```

## بافر و زمان‌های انتظار

### تنظیمات کلیدی

| دستور | کاربرد | مقدار پیشنهادی | نکته مهم |
|-------|--------|----------------|----------|
| client_body_buffer_size | ذخیره موقت داده‌های POST | 10K | اگر زیاد باشد حافظه هدر می‌رود، اگر کم باشد به دیسک می‌رود (کند) |
| client_max_body_size | حداکثر اندازه مجاز برای POST | 8M | اگر بیشتر باشد خطای 413 برمی‌گرداند |
| client_header_buffer_size | حافظه برای هدرهای درخواست | 1K | برای اکثر برنامه‌ها کافی است |
| client_body_timeout | زمان انتظار برای دریافت کل بدنه | 12s | اگر کلاینت کند باشد، سرور صبر نمی‌کند |
| client_header_timeout | زمان انتظار برای دریافت هدرها | 12s | مشابه بالا اما فقط برای هدرها |
| keepalive_timeout | مدت زمان نگهداری اتصال | 15s | اگر زیاد باشد منابع هدر می‌روند، اگر کم باشد اتصال دائم قطع می‌شود |
| send_timeout | زمان انتظار برای ارسال داده | 10s | جلوگیری از اشغال منابع در کلاینت‌های ناکارآمد |
| sendfile on | ارسال مستقیم فایل‌ها از دیسک | --- | سرعت بالا برای فایل‌های بزرگ (بدون استفاده از RAM) |
| tcp_nopush on | بهینه‌سازی بسته‌های TCP | --- | برای ارسال بهتر فایل‌های بزرگ |

## ماژول‌ها

ماژول‌ها در Nginx اجزایی هستند که قابلیت‌های اضافی به وب سرور اضافه می‌کنند:

### انواع ماژول
1. **Static**: در زمان کامپایل Nginx اضافه می‌شوند و بعداً قابل جداسازی نیستند (مانند ngx_http_ssl_module)
2. **Dynamic**: جداگانه کامپایل می‌شوند و در زمان اجرا بارگذاری می‌شوند (مانند ngx_http_image_filter_module)

### دسته‌بندی ماژول‌ها
- **Core modules**: برای مدیریت اصلی و ساختارهای پایه
- **HTTP**: برای پروتکل HTTP
- **Stream**: برای TCP/UDP
- **Mail**: برای پروتکل‌های ایمیل مثل IMAP یا SMTP

### نصب و فعال‌سازی ماژول دینامیک
1. دریافت گزینه‌های کامپایل قبلی: `nginx -v`
2. نصب پیش‌نیازها
3. دریافت ماژول‌های خارجی
4. کامپایل ماژول (با دستور `make module`)
5. کپی فایل `.so` خروجی
6. اضافه کردن این خط به `nginx.conf` (قبل از بلاک http):
   ```nginx
   load_module module/name_module.so;
   ```

## Expires Headers و کش‌سازی

Expires Headers به مرورگر اطلاع می‌دهند که یک پاسخ خاص را تا چه مدت می‌تواند در حافظه کش نگه دارد:

```nginx
location ~* \.(?:css|js|jpg|jpeg|png)$ {
    expires 30d;                       # کش برای ۳۰ روز آینده
    add_header Cache-Control "public"; # به مرورگر می‌گوید می‌تواند این فایل را کش کند
    add_header Pragma "public";        # برای مرورگرهای قدیمی‌تر
    add_header Vary "Accept-Encoding"; # سازگاری با فشرده‌سازی
}
```

### نکات مهم
- وقتی فایلی کش می‌شود، مرورگر تا مدت مشخصی آن را مجدداً درخواست نمی‌کند
- برای حل مشکل تغییرات در فایل‌های کش شده از versioning در URL استفاده می‌کنیم (مثلاً `file.css?v=1`)
- کش‌های Nginx معمولاً در دیسک سرور ذخیره می‌شوند:
  ```nginx
  proxy_cache_path /path/to/disk;
  ```

## فشرده‌سازی Gzip

Gzip امکان فشرده‌سازی محتوا قبل از ارسال به مرورگر را فراهم می‌کند:

```nginx
# فعال‌سازی gzip
gzip on;

# سطح فشرده‌سازی (1-9)
gzip_comp_level 3;

# انواع MIME برای فشرده‌سازی
gzip_types text/plain text/css application/javascript;
```

## Micro Cache

کش کردن پاسخ‌های داینامیک برای مدت بسیار کوتاه که بار را از روی سرور برمی‌دارد:

```nginx
# تنظیم مسیر کش
fastcgi_cache_path /tmp/nginx_cache levels=1:2 keys_zone=zone1:100m inactive=60m use_temp_path=off;

# تعریف کلید کش
fastcgi_cache_key "$scheme$request_method$host$request_uri";
```

## HTTP/2

HTTP/2 یک پروتکل دودویی است، در حالی که HTTP/1 متنی است:

### مزایای HTTP/2
- داده‌های دودویی روش فشرده‌تری برای انتقال اطلاعات هستند
- اتصال دائمی
- Multiplexing: همه فایل‌ها در یک اتصال از سرور دریافت می‌شوند
- Server Push: مرورگر می‌تواند همزمان با درخواست اولیه از وجود فایل‌های دیگر مطلع شود

### نکته
HTTP/2 فقط روی ارتباط امن (HTTPS) کار می‌کند و نیاز به گواهی SSL دارد.

### نصب HTTP/2 و گواهی SSL
برای فعال‌سازی HTTP/2 در بلاک server:

```nginx
listen 443 ssl http2;
ssl_certificate /etc/nginx/ssl/self.crt;
ssl_certificate_key /etc/nginx/ssl/self.key;
```

## ریدایرکت HTTP به HTTPS

برای هدایت خودکار کاربران از HTTP به HTTPS:

```nginx
server {
    listen 80;
    return 301 https://$host$request_uri;
}
```

## افزایش امنیت HTTPS

```nginx
# غیرفعال کردن پروتکل‌های قدیمی SSL
ssl_protocols TLSv1.2 TLSv1.3;

# تعیین cipher suites
ssl_ciphers 'HIGH:!aNULL:!MD5';
ssl_prefer_server_ciphers on;

# فعال کردن DH parameters
ssl_dhparam /etc/nginx/ssl/dhparam.pem;

# فعال کردن HSTS
add_header Strict-Transport-Security "max-age=31536000" always;

# کش کردن session های SSL
ssl_session_cache shared:SSL:40m;
ssl_session_timeout 4h;
ssl_session_tickets on;
```

## Rate Limiting

محدود‌سازی نرخ درخواست‌ها برای محافظت از سرور:

```nginx
# تعریف zone
limit_req_zone $request_uri zone=myzone:10m rate=60r/m;

# اعمال محدودیت
limit_req zone=myzone burst=5 nodelay;
```

## Basic Authentication

افزودن لایه ساده امنیتی با نام کاربری و رمز عبور:

```nginx
location /admin/ {
    auth_basic "Secure Area";
    auth_basic_user_file /etc/nginx/.htpasswd;
}
```

## افزایش امنیت Nginx

1. به‌روزرسانی Nginx و کتابخانه‌ها
2. پنهان کردن نسخه Nginx:
   ```nginx
   server_tokens off;
   ```
3. جلوگیری از clickjacking:
   ```nginx
   add_header X-Frame-Options "SAMEORIGIN";
   ```
4. جلوگیری از XSS:
   ```nginx
   add_header X-XSS-Protection "1; mode=block";
   ```
5. حذف ماژول‌های غیرضروری هنگام کامپایل

## Let's Encrypt

سرویس رایگان برای صدور گواهینامه SSL:

```bash
# نصب Certbot
sudo apt-get install certbot python3-certbot-nginx

# دریافت گواهینامه
sudo certbot --nginx
```

## Reverse Proxy و Load Balancing

### Reverse Proxy
درخواست‌های دریافتی از سمت کلاینت را به سرور بک‌اند متصل می‌کند:

```nginx
location / {
    proxy_pass http://localhost:9000;
}
```

### Load Balancing
توزیع درخواست‌های ورودی بین چندین سرور:

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

### روش‌های مختلف Load Balancing
1. Sticky Session (IP Hash)
2. Least Connection 