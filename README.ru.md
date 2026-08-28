<p align="center">
  <img src="source/images/pixel-cat.svg" alt="PixelCat" width="160" />
</p>

# PixelCat Blog

[中文](README.md) | **Русский** | [فارسی](README.fa.md)

PixelCat Blog — трёхъязычный сайт PixelCat ICU на Hexo 7. Интерфейс и руководство по PixelCat Proxy доступны на китайском, русском и персидском языках.

## Содержимое

- Китайское руководство: `source/_posts/pixelcat-proxy-install-guide.md`
- Русское руководство: `source/_posts/pixelcat-proxy-install-guide.ru.md`
- Персидское руководство: `source/_posts/pixelcat-proxy-install-guide.fa.md`
- Словарь интерфейса: `source/_data/i18n.json`
- Русские страницы: `source/ru/`
- Персидские страницы: `source/fa/`
- Тема: `themes/pixel-cactus/`

## Технологии

- Hexo 7.3
- EJS
- Пользовательская тема `pixel-cactus`
- Локальный поиск, Atom feed, sitemap и robots.txt
- SEO: canonical URL, `hreflang`, Open Graph, Twitter Card и JSON-LD `inLanguage`
- Поддержка RTL для персидского языка

## Локальная разработка

```bash
npm install
npm run server
```

Сайт будет доступен по адресу:

```text
http://localhost:4000/
```

Статическая сборка:

```bash
npm run clean
npm run build
```

## Языковые маршруты

```text
/                                         главная на китайском
/ru/                                      главная на русском
/fa/                                      главная на персидском, RTL
/posts/pixelcat-proxy-install-guide/      руководство на китайском
/ru/posts/pixelcat-proxy-install-guide/   руководство на русском
/fa/posts/pixelcat-proxy-install-guide/   руководство на персидском
```

В front matter переведённых страниц используются поля `lang`, `translation_key`, `permalink` и `translations`. Тема создаёт переключатель языков и SEO-ссылки `hreflang`.

## Основные каталоги

```text
source/_posts/                 статьи
source/_data/i18n.json         переводы интерфейса
source/ru/                     русские страницы
source/fa/                     персидские страницы
source/images/                 изображения
themes/pixel-cactus/layout/    шаблоны темы
themes/pixel-cactus/source/    стили темы
public/                        результат сборки Hexo
```

## Cloudflare Pages

- Framework preset: `Hexo`
- Build command: `npm run build`
- Build output directory: `public`
- Node.js: `20` или `22`

Подключите репозиторий в Cloudflare Pages, используйте указанные параметры и привяжите домен после успешной сборки.

## Ссылки

- Сайт: https://pixelcat.icu
- YouTube: https://www.youtube.com/@PixelCatICU
- GitHub: https://github.com/PixelCatICU
- X: https://x.com/PixelCatICU
- RSS: `/atom.xml`

Материалы предназначены для законного обучения, исследований, удалённой работы и доступа к открытой информации. Соблюдайте местные законы и правила провайдера.
