```markdown
# Dealsbe - Exclusive Software Deals for Developers and Startups

![Dealsbe Banner](https://via.placeholder.com/1200x600.png?text=Dealsbe+Exclusive+Deals+2025)

> **تخفیف‌های ویژه نرم‌افزار برای توسعه‌دهندگان و استارتاپ‌ها**  
> آخرین به‌روزرسانی: **۵ نوامبر ۲۰۲۵**

---

## آموزش کامل ساخت پروکسی اسپانسری تلگرام با متد جدید MTProto (2025)

> **پروکسی ضدفیلتر + Obfuscation + اسپانسری خودکار**

---

## فهرست مطالب
- [چرا پروکسی MTProto؟](#چرا-پروکسی-mtproto)
- [پیش‌نیازها](#پیشنیازها)
- [مرحله ۱: آماده‌سازی سرور](#مرحله-۱-آمادهسازی-سرور)
- [مرحله ۲: نصب MTProto Proxy](#مرحله-۲-نصب-mtproto-proxy)
- [مرحله ۳: تولید کلید و راه‌اندازی](#مرحله-۳-تولید-کلید-و-راهاندازی)
- [مرحله ۴: ساخت لینک اسپانسری](#مرحله-۴-ساخت-لینک-اسپانسری)
- [مرحله ۵: تبلیغ در کانال‌های رسمی](#مرحله-۵-تبلیغ-در-کانالهای-رسمی)
- [تکنیک Obfuscation (مبهم‌سازی)](#تکنیک-obfuscation-مبهمسازی)
- [نکات ضدفیلتر پیشرفته](#نکات-ضدفیلتر-پیشرفته)
- [رفع مشکلات رایج](#رفع-مشکلات-رایج)
- [شروع سریع با اسکریپت خودکار](#شروع-سریع-با-اسکریپت-خودکار)
- [از من حمایت کنید](#از-من-حمایت-کنید)

---

## چرا پروکسی MTProto؟

پروکسی **MTProto** پروتکل رسمی تلگرام برای دور زدن فیلترینگ است:
- رمزنگاری end-to-end  
- پشتیبانی از **obfuscation** داخلی  
- نمایش خودکار به‌عنوان **پروکسی اسپانسری** در تلگرام  
- مقاوم در برابر DPI و فیلترینگ هوشمند

---

## پیش‌نیازها

| مورد | توضیح |
|------|-------|
| VPS | حداقل ۱ هسته، ۱ گیگ رم، Ubuntu 22.04+ |
| دسترسی root | از طریق SSH |
| دامنه (اختیاری) | برای مخفی‌سازی بهتر IP |

> **توصیه:** سرور در هلند، آلمان یا سنگاپور (پینگ پایین به ایران)

---

## مرحله ۱: آماده‌سازی سرور

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git curl build-essential libssl-dev zlib1g-dev -y
```

---

## مرحله ۲: نصب MTProto Proxy

```bash
git clone https://github.com/TelegramMessenger/MTProxy.git
cd MTProxy
make
```

---

## مرحله ۳: تولید کلید و راه‌اندازی

```bash
# دریافت فایل‌های رسمی تلگرام
curl -s https://core.telegram.org/getProxySecret -o proxy-secret
curl -s https://core.telegram.org/getProxyConfig -o proxy-multi.conf

# تولید secret تصادفی (16 بایت)
SECRET=$(head -c 16 /dev/urandom | xxd -ps)

# راه‌اندازی پروکسی روی پورت 443
./objs/bin/mtproto-proxy -u nobody -p 8888 -H 443 -S $SECRET \
  --aes-pwd proxy-secret proxy-multi.conf \
  --allow-skip-dh --max-special-connections 2000 -M 2
```

> برای اجرای دائمی از `systemd` یا `screen` استفاده کنید.

---

## مرحله ۴: ساخت لینک اسپانسری

```text
tg://proxy?server=YOUR_IP&port=443&secret=dd$SECRET
```

مثال:
```
tg://proxy?server=185.123.45.67&port=443&secret=dd1a2b3c4d5e6f7890abcdef12345678
```

> پیشوند `dd` = **فعال‌سازی obfuscation (مبهم‌سازی)**

---

## مرحله ۵: تبلیغ در کانال‌های رسمی

1. لینک را در کانال `@proxy` یا کانال‌های پرمخاطب تلگرام ارسال کنید  
2. پس از تأیید، پروکسی شما در بخش **Sponsored Proxies** نمایش داده می‌شود

---

## تکنیک Obfuscation (مبهم‌سازی ترافیک)

| روش | توضیح |
|-----|-------|
| **TLS + پورت 443** | ترافیک شبیه HTTPS به نظر می‌رسد |
| **Padding تصادفی** | الگوهای بسته‌ها را می‌شکند |
| **Protocol Mimicking** | شبیه‌سازی HTTP/2 |
| **چندلایه (Chain)** | MTProto → Cloudflare → کاربر |

---

## نکات ضدفیلتر پیشرفته

- هر ۲۴ ساعت `secret` را تغییر دهید  
- از **Cloudflare Warp** یا **CDN** به‌عنوان لایه میانی استفاده کنید  
- پورت‌های 80، 8080، 8443 را تست کنید  
- اسکریپت خودکار ری‌استارت با `cron`

```bash
# مثال: تغییر secret هر 24 ساعت
0 0 * * * cd /path/to/MTProxy && ./restart_proxy.sh
```

---

## رفع مشکلات رایج

| مشکل | راه‌حل |
|------|--------|
| `make: command not found` | `sudo apt install build-essential` |
| پروکسی وصل نمی‌شود | `ufw allow 443` یا `iptables -A INPUT -p tcp --dport 443 -j ACCEPT` |
| لینک اسپانسری نمایش داده نمی‌شود | حتماً از پیشوند `dd` استفاده کنید |
| سرور کرش می‌کند | `--max-special-connections 1000` را کاهش دهید |

---

## شروع سریع با اسکریپت خودکار

```bash
# کلون کردن پروژه
git clone https://github.com/theesmaeil1/Telegram-MTProto-Sponsored-Proxy.git
cd Telegram-MTProto-Sponsored-Proxy

# اجرای اسکریپت نصب خودکار
sudo bash install.sh
```

> اسکریپت تمام مراحل را انجام می‌دهد:  
> نصب وابستگی‌ها → کامپایل → تولید secret → راه‌اندازی با systemd → فعال‌سازی obfuscation

---

## از من حمایت کنید

اگر این آموزش براتون مفید بود، لطفاً با یکی از روش‌ها از من حمایت کنید:

| روش | لینک |
|-----|------|
| **قهوه بخر برام** | [buymeacoffee.com/theesmaeil1](https://buymeacoffee.com/theesmaeil1) |
| **ارز دیجیتال (USDT-TRC20)** | `TYourWalletAddressHere` |
| **پیام تشکر در تلگرام** | [@theesmaeil1](https://t.me/theesmaeil1) |

> هر حمایت، انگیزه‌ای برای آپدیت مداوم و اضافه کردن امکانات جدیده!

---

## منابع و لینک‌ها

- [مخزن رسمی MTProxy](https://github.com/TelegramMessenger/MTProxy)  
- [مستندات تلگرام](https://core.telegram.org/mtproto)  
- [کانال تلگرام برای آپدیت‌ها](https://t.me/mtproto_theesmaeil1)

---

**نویسنده:** [theesmaeil1](https://github.com/theesmaeil1)  
**تاریخ انتشار:** ۲۶ شهریور ۱۴۰۳  
**آخرین ویرایش:** ۵ نوامبر ۲۰۲۵

---

> **این آموزش به‌صورت کامل، ریسپانسیو و سئو-بهینه برای گیت‌هاب و گوگل طراحی شده است.**  
> **کلمات کلیدی:** `پروکسی اسپانسری تلگرام`, `MTProto جدید`, `ضدفیلتر`, `obfuscation`, `نصب MTProxy`, `VPS تلگرام`

---

## Fresh Recommendations

[Post a Deal](#)

---

**فقط یک فایل: `README.md`**  
**کامل، فارسی، ریسپانسیو، سئو-بهینه، آماده آپلود در گیت‌هاب**
```
