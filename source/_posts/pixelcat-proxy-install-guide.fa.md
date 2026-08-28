---
title: راه‌اندازی PixelCat Proxy با NaiveProxy و Hysteria2
date: 2026-05-12 10:00:00
updated: 2026-08-28 17:10:00
lang: fa
translation_key: pixelcat-proxy-install-guide
permalink: /fa/posts/pixelcat-proxy-install-guide/
translations:
  zh-CN: /posts/pixelcat-proxy-install-guide/
  ru: /ru/posts/pixelcat-proxy-install-guide/
  fa: /fa/posts/pixelcat-proxy-install-guide/
description: راهنمای سه‌زبانه نصب PixelCat Proxy، NaiveProxy و Hysteria2 روی لینوکس؛ شامل دامنه، فایروال، TLS، پرش پورت UDP، systemd و عیب‌یابی.
tags:
  - PixelCat Proxy
  - NaiveProxy
  - Hysteria2
  - Linux
---

PixelCat Proxy یک نصب‌کننده سرور لینوکس با رابط چینی، روسی و فارسی است. این پروژه موارد زیر را راه‌اندازی می‌کند:

- **NaiveProxy**: کدی (Caddy) با افزونه `github.com/klzgrad/forwardproxy@naive`، اتصال TLS، پروکسی HTTP/2 CONNECT، احراز هویت و سایت پوششی.
- **Hysteria2**: سرور رسمی مبتنی بر QUIC/UDP با قابلیت پرش پورت.
- **ابزارهای نگهداری**: فعال‌سازی BBR، بررسی کیفیت IP، دسترسی سرویس‌های پخش و مسیر بازگشت شبکه.

NaiveProxy از `443/tcp` و Hysteria2 از `443/udp` استفاده می‌کند؛ بنابراین هر دو می‌توانند روی یک سرور و یک دامنه اجرا شوند.

> قوانین محل زندگی، شرایط شرکت میزبانی و سیاست شبکه سازمان خود را رعایت کنید.

## اجرای سریع

به سرور متصل شوید و نسخه فارسی نصب‌کننده را اجرا کنید:

```bash
ssh root@SERVER_IP
curl -fsSL https://raw.githubusercontent.com/PixelCatICU/pixelcat-proxy/main/install.sh | bash -s -- --lang fa
```

اگر پروژه از قبل دریافت شده است:

```bash
./deploy.sh --lang fa
```

زبان را می‌توان با متغیر محیطی نیز مشخص کرد:

```bash
PIXELCAT_LANG=fa ./deploy.sh
```

## پیش‌نیازها

- لینوکس همراه systemd؛ مانند Debian، Ubuntu، توزیع‌های خانواده RHEL، Fedora یا Alpine.
- معماری `amd64` یا `arm64`.
- دامنه با رکورد `A` یا `AAAA` که به IP سرور اشاره کند.
- پورت‌های باز:
  - `80/tcp` و `443/tcp` برای NaiveProxy؛
  - `443/udp` برای Hysteria2؛
  - `20000-50000/udp` برای بازه پیش‌فرض پرش پورت.

هم فایروال پنل ابری و هم فایروال سیستم‌عامل را بررسی کنید.

## انتخاب پروتکل

| پروتکل | انتقال | کاربرد پیشنهادی |
| --- | --- | --- |
| NaiveProxy | HTTPS / TCP | شبکه عادی، سازگاری بالا و ترافیک شبیه HTTPS معمولی |
| Hysteria2 | QUIC / UDP | تأخیر زیاد، اتلاف بسته، شبکه موبایل یا مسیر ناپایدار |

راهکار عملی این است که هر دو را نصب کنید و عملکردشان را روی مسیر واقعی خود بسنجید.

## منوی نصب

پس از اجرا زبان فارسی را انتخاب کنید. گزینه‌های اصلی:

```text
1) نصب / به‌روزرسانی PixelCat NaiveProxy
2) نصب / به‌روزرسانی PixelCat Hysteria2
3) حذف PixelCat NaiveProxy
4) حذف PixelCat Hysteria2
5) فعال‌کردن BBR
6) بررسی کیفیت IP
7) بررسی دسترسی سرویس‌های پخش
8) کیفیت شبکه / مسیر بازگشت
0) خروج
```

برای دیدن تمام پارامترها:

```bash
./deploy.sh --lang fa --help
```

## نصب NaiveProxy

گزینه `1` را انتخاب کنید و موارد زیر را وارد کنید:

1. دامنه پراکسی، مانند `proxy.example.com`؛
2. نام کاربری؛
3. گذرواژه قوی؛
4. دامنه سایت پوششی مانند `www.example.com`، بدون `https://`؛
5. ایمیل Let's Encrypt در صورت نیاز؛
6. پورت HTTP برابر `80` و HTTPS برابر `443`.

اجرای غیرتعاملی:

```bash
./deploy.sh --lang fa --install -y \
  --domain proxy.example.com \
  --username your_user \
  --password change_this_strong_password \
  --decoy-domain www.example.com \
  --email admin@example.com
```

واردکردن گذرواژه در خط فرمان ممکن است آن را در تاریخچه shell ذخیره کند. روی سرور عملیاتی، ورود تعاملی گذرواژه امن‌تر است.

## نصب Hysteria2

به منو برگردید و گزینه `2` را انتخاب کنید. اگر NaiveProxy قبلاً پیکربندی شده باشد، نصب‌کننده می‌تواند دامنه، گذرواژه و گواهی کدی را دوباره استفاده کند.

مقادیر پیشنهادی:

- پورت UDP: مقدار `443`؛
- بازه پرش پورت: `20000-50000`؛
- محدودیت آپلود و دانلود: مقدار `0` یعنی نامحدود؛
- URL پوششی: `https://www.example.com` یا یک سایت HTTPS در دسترس.

اجرای غیرتعاملی:

```bash
./deploy.sh --lang fa --install-hysteria2 -y \
  --hy2-domain proxy.example.com \
  --hy2-port 443 \
  --hy2-hop-range 20000-50000 \
  --hy2-up-mbps 0 \
  --hy2-down-mbps 0 \
  --hy2-masquerade https://www.example.com
```

برای غیرفعال‌کردن پرش پورت از `--hy2-hop-range off` استفاده کنید.

## بررسی پس از نصب

```bash
systemctl status pixelcat-naiveproxy --no-pager
systemctl status pixelcat-hysteria2 --no-pager
systemctl status pixelcat-hysteria2-hop --no-pager

journalctl -u pixelcat-naiveproxy -f
journalctl -u pixelcat-hysteria2 -f

ss -lntup
```

وضعیت سرویس‌ها باید `active (running)` باشد. دسترسی بیرونی TCP 443 و UDP 443 را بررسی کنید و اگر پرش پورت فعال است، بازه UDP را نیز باز نگه دارید.

## به‌روزرسانی و حذف

اجرای دوباره فرمان ورودی، نسخه تازه پروژه را دریافت می‌کند. برای حذف از گزینه‌های `3` و `4` یا فرمان‌های زیر استفاده کنید:

```bash
./deploy.sh --lang fa --uninstall
./deploy.sh --lang fa --uninstall-hysteria2
```

فقط زمانی `--purge` را اضافه کنید که قصد حذف پیکربندی محلی و داده‌های گواهی ذکرشده در پیام تأیید را دارید.

پروژه: [github.com/PixelCatICU/pixelcat-proxy](https://github.com/PixelCatICU/pixelcat-proxy)
