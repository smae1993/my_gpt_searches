
# تنظیم DNS به‌صورت دائمی در Ubuntu 24.04 با Netplan

این راهنما مراحل تنظیم دو DNS زیر را روی سیستم Ubuntu 24.04 نشان می‌دهد:

- `217.218.127.127`
- `217.218.155.155`

---

## 1. بررسی فایل‌های Netplan

ابتدا بررسی کن که فایل‌های Netplan در کجا هستند:

```bash
ls /etc/netplan/
```

خروجی معمولاً چیزی مثل `01-netcfg.yaml` است.

---

## 2. ویرایش فایل پیکربندی

فایل مربوطه را با یک ویرایشگر باز کن:

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

### اگر DHCP غیرفعال (IP استاتیک) است:

```yaml
network:
  version: 2
  ethernets:
    eth0:  # به جای eth0 نام اینترفیس خودت رو بذار
      dhcp4: no
      addresses:
        - 192.168.1.10/24  # آدرس IP لوکال
      gateway4: 192.168.1.1  # گیت‌وی پیش‌فرض
      nameservers:
        addresses:
          - 217.218.127.127
          - 217.218.155.155
```

### اگر DHCP فعال است:

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
      nameservers:
        addresses:
          - 217.218.127.127
          - 217.218.155.155
```

> 💡 توجه: برای پیدا کردن نام اینترفیس شبکه از دستور `ip a` استفاده کن.

---

## 3. اعمال تنظیمات

بعد از ذخیره فایل، دستور زیر را اجرا کن:

```bash
sudo netplan apply
```

---

## 4. بررسی تنظیمات

برای اطمینان از اعمال DNS:

```bash
resolvectl status
```

یا:

```bash
systemd-resolve --status | grep 'DNS Servers' -A2
```

---

## در صورت مشکل

اگر مشکلی در اتصال یا تنظیمات داشتی، این موارد را چک کن:

- دستور `ip a` برای بررسی اینترفیس شبکه
- محتوای فایل YAML که ویرایش کردی
- بررسی وجود خطا با دستور `sudo netplan try`

---
