
# آموزش کامل ساخت پروکسی اختصاصی و اسپانسری تلگرام (MTProto Proxy)

> **هشدار مهم**: این آموزش صرفاً جنبه **آموزشی** دارد و برای دور زدن محدودیت‌های قانونی یا استفاده غیرمجاز توصیه نمی‌شود. استفاده از پروکسی برای فعالیت‌های غیرقانونی، نقض قوانین کشور شما یا سیاست‌های تلگرام ممکن است عواقب قانونی داشته باشد.  
> **مسئولیت کامل استفاده از این آموزش بر عهده خود شماست.**

این مستند یک راهنمای **کامل، گام‌به‌گام و اختصاصی** برای راه‌اندازی یک **پروکسی MTProto اختصاصی تلگرام** (همراه با قابلیت **اسپانسری** برای نمایش تبلیغ در کانال‌های تلگرام) است. این آموزش برای **سرورهای لینوکسی (اوبونتو 22.04/20.04 یا دبیان 11/12)** طراحی شده و تمام جزئیات فنی، امنیتی و بهینه‌سازی را شامل می‌شود.

---

## فهرست مطالب
1. [پیش‌نیازها](#پیشنیازها)
2. [نصب و راه‌اندازی MTProto Proxy](#نصب-و-راهاندازی-mtproto-proxy)
3. [ایجاد کلید مخفی (Secret)](#ایجاد-کلید-مخفی-secret)
4. [راه‌اندازی اسپانسری (Sponsor)](#راهاندازی-اسپانسری-sponsor)
5. [بهینه‌سازی و امنیت](#بهینهسازی-و-امنیت)
6. [اتوماسیون با systemd](#اتوماسیون-با-systemd)
7. [مانیتورینگ و لاگ‌گیری](#مانیتورینگ-و-لاگگیری)
8. [اتصال به پروکسی در تلگرام](#اتصال-به-پروکسی-در-تلگرام)
9. [حل مشکلات رایج](#حل-مشکلات-رایج)
10. [نکات پیشرفته و اختصاصی](#نکات-پیشرفته-و-اختصاصی)

---

## پیش‌نیازها

قبل از شروع، باید موارد زیر را آماده کنید:

| مورد | توضیحات |
|------|---------|
| **سرور VPS** | حداقل 1 هسته CPU، 1 گیگ رم، 10 گیگ فضای دیسک (توصیه: 2 هسته، 2 گیگ رم) |
| **سیستم عامل** | Ubuntu 22.04/20.04 یا Debian 11/12 |
| **دامنه یا IP عمومی** | برای اتصال پایدار (دامنه توصیه می‌شود) |
| **دسترسی root** | برای نصب پکیج‌ها و تنظیمات فایروال |
| **اتصال اینترنت پرسرعت** | حداقل 100 مگابیت (برای پشتیبانی از 100+ کاربر) |

> توصیه: از ارائه‌دهندگانی مثل **Hetzner, DigitalOcean, Vultr** استفاده کنید.

---

## نصب و راه‌اندازی MTProto Proxy

### 1. به‌روزرسانی سیستم
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. نصب وابستگی‌ها
```bash
sudo apt install git curl build-essential libssl-dev zlib1g-dev -y
```

### 3. کلون کردن سورس رسمی تلگرام
```bash
cd /opt
sudo git clone https://github.com/TelegramMessenger/MTProxy.git
cd MTProxy
```

### 4. کامپایل پروژه
```bash
sudo make
```
> پس از اتمام، فایل اجرایی در مسیر `objs/bin/mtproto-proxy` ایجاد می‌شود.

---

## ایجاد کلید مخفی (Secret)

هر پروکسی نیاز به یک **کلید 32 بایتی (hex)** دارد.

```bash
# ایجاد کلید تصادفی
head -c 16 /dev/urandom | xxd -ps
```

مثال خروجی:
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

> این کلید را **ذخیره کنید** — برای اتصال کاربران لازم است.

---

## راه‌اندازی پروکسی (بدون اسپانسری)

```bash
sudo objs/bin/mtproto-proxy -u nobody -p 8888 -H 443 -S <YOUR_SECRET> --aes-pwd proxy-secret proxy-multi.conf -M 1
```

### توضیحات پارامترها:
| پارامتر | توضیح |
|--------|-------|
| `-u nobody` | اجرای پروکسی با کاربر غیرروت |
| `-p 8888` | پورت داخلی (برای Prometheus) |
| `-H 443` | پورت خارجی (443 استاندارد HTTPS) |
| `-S <SECRET>` | کلید مخفی 32 کاراکتری |
| `--aes-pwd` | فایل‌های رمزنگاری (امنیت بیشتر) |
| `-M 1` | تعداد worker (1 بهینه برای سرور کوچک) |

---

## راه‌اندازی اسپانسری (Sponsor)

برای نمایش تبلیغ در کانال‌های تلگرام (مثل @proxy_sponsor_channel)، باید **کانال اسپانسری** بسازید.

### گام ۱: ساخت کانال اسپانسری
1. در تلگرام یک کانال جدید بسازید (مثلاً: `@MyProxySponsor`)
2. آیدی کانال را یادداشت کنید (مثلاً: `sponsor_channel_test`)

### گام ۲: اجرای پروکسی با اسپانسری
```bash
sudo objs/bin/mtproto-proxy \
  -u nobody \
  -p 8888 \
  -H 443 \
  -S <YOUR_SECRET> \
  --aes-pwd proxy-secret proxy-multi.conf \
  -M 1 \
  --sponsor sponsor_channel_test
```

> حالا وقتی کاربری به پروکسی شما وصل شود، کانال شما به عنوان **اسپانسر** نمایش داده می‌شود.

---

## بهینه‌سازی و امنیت

### ۱. تنظیم فایروال (UFW)
```bash
sudo apt install ufw -y
sudo ufw allow 443/tcp
sudo ufw allow 22/tcp
sudo ufw enable
```

### ۲. فعال‌سازی NAT (برای چند IP)
اگر چندین IP دارید:
```bash
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
```

### ۳. استفاده از چند Secret (چند کاربر همزمان)
فایل `proxy-multi.conf` بسازید:
```bash
echo "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6" > proxy-multi.conf
echo "b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6a1" >> proxy-multi.conf
```

سپس از `--aes-pwd proxy-secret proxy-multi.conf` استفاده کنید.

---

## اتوماسیون با systemd

فایل سرویس بسازید:

```bash
sudo nano /etc/systemd/system/mtproxy.service
```

محتوا:
```ini
[Unit]
Description=MTProto Proxy
After=network.target

[Service]
Type=simple
WorkingDirectory=/opt/MTProxy
ExecStart=/opt/MTProxy/objs/bin/mtproto-proxy -u nobody -p 8888 -H 443 -S a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6 --aes-pwd proxy-secret proxy-multi.conf -M 1 --sponsor sponsor_channel_test
Restart=always
RestartSec=5
User=root

[Install]
WantedBy=multi-user.target
```

فعال‌سازی:
```bash
sudo systemctl daemon-reexec
sudo systemctl enable mtproxy.service
sudo systemctl start mtproxy.service
```

بررسی وضعیت:
```bash
sudo systemctl status mtproxy.service
```

---

## مانیتورینگ و لاگ‌گیری

### ۱. آمار کاربران (Prometheus)
وب اینترفیس در پورت 8888:
```
http://YOUR_IP:8888/stats
```

### ۲. لاگ‌گیری
لاگ‌ها در `stderr` نمایش داده می‌شوند. برای ذخیره:
```bash
sudo journalctl -u mtproxy.service -f
```

---

## اتصال به پروکسی در تلگرام

### لینک اتصال (برای اشتراک‌گذاری):
```
tg://proxy?server=YOUR_IP&port=443&secret=a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

> جایگزین کنید `YOUR_IP` با IP سرور و `secret` با کلید خودتان.

### در تلگرام:
1. تنظیمات → داده و حافظه → پروکسی
2. افزودن پروکسی → MTProto
3. وارد کردن IP، پورت (443)، و Secret

---

## حل مشکلات رایج

| مشکل | راه‌حل |
|------|--------|
| پروکسی وصل نمی‌شود | پورت 443 باز باشد، فایروال چک شود، IP عمومی باشد |
| Secret کار نمی‌کند | دقیقاً 32 کاراکتر hex باشد |
| اسپانسر نمایش داده نمی‌شود | کانال عمومی باشد، آیدی درست وارد شود |
| مصرف رم بالا | تعداد worker را کاهش دهید (`-M 1`) |

---

## نکات پیشرفته و اختصاصی

### ۱. استفاده از دامنه + SSL (اختیاری اما حرفه‌ای)
از **Cloudflare** یا **Nginx Reverse Proxy** با گواهی SSL استفاده کنید.

### ۲. پشتیبانی از Fake TLS (مخفی‌سازی ترافیک)
با اضافه کردن `--nat-info` و تنظیم DNS.

### ۳. چند پروکسی روی یک سرور
با پورت‌های مختلف (443, 8443, ...) و Secret جداگانه.

### ۴. بکاپ‌گیری از Secretها
```bash
cp proxy-secret proxy-multi.conf /root/mtproxy-backup/
```

### ۵. اسکریپت نصب خودکار (اختصاصی)
فایل `install.sh` بسازید:
```bash
#!/bin/bash
apt update && apt upgrade -y
apt install git curl build-essential libssl-dev zlib1g-dev ufw -y
cd /opt
git clone https://github.com/TelegramMessenger/MTProxy.git
cd MTProxy
make
head -c 16 /dev/urandom | xxd -ps > /opt/MTProxy/secret.txt
ufw allow 443/tcp
ufw allow 22/tcp
ufw enable
echo "پروکسی آماده است! Secret: $(cat secret.txt)"
```

اجرا:
```bash
chmod +x install.sh
./install.sh
```

---

## نتیجه‌گیری

شما حالا یک **پروکسی MTProto اختصاصی، امن، با اسپانسری و مانیتورینگ** دارید که می‌توانید آن را در گیت‌هاب، کانال تلگرام یا وبسایت خود منتشر کنید.

> **لینک اشتراک‌گذاری نمونه**:
```
tg://proxy?server=1.2.3.4&port=443&secret=a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

---

## منابع
- [مستندات رسمی MTProto Proxy](https://github.com/TelegramMessenger/MTProxy)
- [کانال رسمی پروکسی تلگرام](https://t.me/proxy)

---

**ساخته شده با ❤️ توسط [نام شما] - برای گیت‌هاب**
```
