
# راه‌اندازی سایت روی Ubuntu 24 با BIND9 و Nginx

این راهنما نحوه راه‌اندازی یک وب‌سایت با استفاده از BIND9 به‌عنوان DNS و Nginx به‌عنوان وب‌سرور را توضیح می‌دهد.

---

## ✅ مرحله ۱: نصب پکیج‌های ضروری

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx bind9 bind9utils bind9-doc dnsutils curl unzip ufw -y
```

---

## ✅ مرحله ۲: تنظیم DNS سرور با BIND9

### 2.1. تعریف zone در named.conf.local

```bash
sudo nano /etc/bind/named.conf.local
```

اضافه کنید:

```bash
zone "example.com" {
    type master;
    file "/etc/bind/zones/db.example.com";
};
```

### 2.2. ساخت پوشه zone و فایل zone

```bash
sudo mkdir -p /etc/bind/zones
sudo nano /etc/bind/zones/db.example.com
```

محتوای فایل zone:

```bind
$TTL 604800
@   IN  SOA ns1.example.com. admin.example.com. (
            3         ; Serial
            604800    ; Refresh
            86400     ; Retry
            2419200   ; Expire
            604800 )  ; Negative Cache TTL

; Name servers
    IN  NS  ns1.example.com.
    IN  NS  ns2.example.com.

; A records
@       IN  A   192.168.1.100
ns1     IN  A   192.168.1.100
ns2     IN  A   192.168.1.101
www     IN  A   192.168.1.100
```

### 2.3. تست و ری‌استارت

```bash
sudo named-checkconf
sudo named-checkzone example.com /etc/bind/zones/db.example.com
sudo systemctl restart bind9
sudo systemctl enable bind9
```

---

## ✅ مرحله ۳: تنظیم NGINX

### 3.1. فایل پیکربندی سایت

```bash
sudo nano /etc/nginx/sites-available/example.com
```

محتوا:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    root /var/www/example.com;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 3.2. فعال‌سازی سایت

```bash
sudo mkdir -p /var/www/example.com
echo "<h1>Hello from example.com</h1>" | sudo tee /var/www/example.com/index.html

sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## ✅ مرحله ۴: تنظیم فایروال

```bash
sudo ufw allow 'Nginx Full'
sudo ufw allow 53
sudo ufw enable
```

---

## ✅ مرحله ۵: تنظیم دامنه در پنل دامین

رکوردهای DNS را در پنل دامنه‌تان به صورت زیر قرار دهید:

| Type | Name     | Value             |
|------|----------|-------------------|
| A    | @        | 192.168.1.100     |
| A    | www      | 192.168.1.100     |
| NS   | @        | ns1.example.com   |
| NS   | @        | ns2.example.com   |
| A    | ns1      | 192.168.1.100     |
| A    | ns2      | 192.168.1.101     |

---

## ✅ مرحله ۶: تست DNS

```bash
dig @127.0.0.1 example.com
dig @127.0.0.1 www.example.com
```

---

## ✅ مرحله ۷: فعال‌سازی HTTPS با Let's Encrypt (اختیاری)

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d example.com -d www.example.com
```

---

## ✅ پایان 🎉

اکنون سایت شما با DNS اختصاصی و وب‌سرور Nginx آماده است.
