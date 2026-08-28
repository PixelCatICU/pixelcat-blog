<p align="center">
  <img src="source/images/pixel-cat.svg" alt="PixelCat" width="160" />
</p>

# PixelCat Blog

[中文](README.md) | [Русский](README.ru.md) | **فارسی**

PixelCat Blog وب‌سایت سه‌زبانه PixelCat ICU است که با Hexo 7 ساخته می‌شود. رابط سایت و راهنمای PixelCat Proxy به زبان‌های چینی، روسی و فارسی در دسترس است.

## محتوا

- راهنمای چینی: `source/_posts/pixelcat-proxy-install-guide.md`
- راهنمای روسی: `source/_posts/pixelcat-proxy-install-guide.ru.md`
- راهنمای فارسی: `source/_posts/pixelcat-proxy-install-guide.fa.md`
- واژه‌نامه رابط: `source/_data/i18n.json`
- صفحه‌های روسی: `source/ru/`
- صفحه‌های فارسی: `source/fa/`
- پوسته: `themes/pixel-cactus/`

## فناوری‌ها

- Hexo 7.3
- قالب‌های EJS
- پوسته اختصاصی `pixel-cactus`
- جستجوی محلی، Atom feed، sitemap و robots.txt
- SEO شامل canonical URL، `hreflang`، Open Graph، Twitter Card و JSON-LD `inLanguage`
- پشتیبانی RTL برای زبان فارسی

## توسعه محلی

```bash
npm install
npm run server
```

نشانی پیش‌فرض:

```text
http://localhost:4000/
```

ساخت فایل‌های ایستا:

```bash
npm run clean
npm run build
```

## مسیرهای زبان

```text
/                                         صفحه اصلی چینی
/ru/                                      صفحه اصلی روسی
/fa/                                      صفحه اصلی فارسی با RTL
/posts/pixelcat-proxy-install-guide/      راهنمای چینی
/ru/posts/pixelcat-proxy-install-guide/   راهنمای روسی
/fa/posts/pixelcat-proxy-install-guide/   راهنمای فارسی
```

در front matter صفحه‌های ترجمه‌شده از فیلدهای `lang`، `translation_key`، `permalink` و `translations` استفاده می‌شود. پوسته، کلید تغییر زبان و پیوندهای SEO از نوع `hreflang` را تولید می‌کند.

## پوشه‌های اصلی

```text
source/_posts/                 نوشته‌ها
source/_data/i18n.json         ترجمه رابط
source/ru/                     صفحه‌های روسی
source/fa/                     صفحه‌های فارسی
source/images/                 تصاویر
themes/pixel-cactus/layout/    قالب‌های پوسته
themes/pixel-cactus/source/    استایل‌های پوسته
public/                        خروجی Hexo
```

## Cloudflare Pages

- Framework preset: `Hexo`
- Build command: `npm run build`
- Build output directory: `public`
- Node.js: نسخه `20` یا `22`

مخزن را در Cloudflare Pages متصل کنید، تنظیمات بالا را وارد کنید و پس از ساخت موفق، دامنه را متصل کنید.

## پیوندها

- وب‌سایت: https://pixelcat.icu
- YouTube: https://www.youtube.com/@PixelCatICU
- GitHub: https://github.com/PixelCatICU
- X: https://x.com/PixelCatICU
- RSS: `/atom.xml`

محتوا برای یادگیری، پژوهش، دورکاری و دسترسی قانونی به اطلاعات عمومی تهیه شده است. قوانین محل و شرایط ارائه‌دهنده را رعایت کنید.
