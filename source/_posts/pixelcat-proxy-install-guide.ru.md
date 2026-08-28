---
title: "PixelCat Proxy: установка NaiveProxy и Hysteria2"
date: 2026-05-12 10:00:00
updated: 2026-08-28 17:10:00
lang: ru
translation_key: pixelcat-proxy-install-guide
permalink: /ru/posts/pixelcat-proxy-install-guide/
translations:
  zh-CN: /posts/pixelcat-proxy-install-guide/
  ru: /ru/posts/pixelcat-proxy-install-guide/
  fa: /fa/posts/pixelcat-proxy-install-guide/
description: "Трёхъязычное руководство по установке PixelCat Proxy, NaiveProxy и Hysteria2 на Linux: домен, firewall, TLS, UDP port hopping, systemd и диагностика."
tags:
  - PixelCat Proxy
  - NaiveProxy
  - Hysteria2
  - Linux
---

PixelCat Proxy — установщик для Linux-серверов с интерфейсом на китайском, русском и персидском языках. Он развёртывает:

- **NaiveProxy**: Caddy с модулем `github.com/klzgrad/forwardproxy@naive`, TLS, HTTP/2 CONNECT, Basic Auth и сайтом-маскировкой.
- **Hysteria2**: официальный сервер QUIC/UDP с поддержкой смены портов.
- **Диагностику**: BBR, качество IP, разблокировку стриминга и обратный маршрут.

NaiveProxy использует `443/tcp`, а Hysteria2 — `443/udp`, поэтому оба сервиса могут работать на одном сервере и одном домене.

> Соблюдайте местное законодательство, условия хостинга и сетевые правила вашей организации.

## Быстрый запуск

Подключитесь к серверу и запустите установщик:

```bash
ssh root@SERVER_IP
curl -fsSL https://raw.githubusercontent.com/PixelCatICU/pixelcat-proxy/main/install.sh | bash -s -- --lang ru
```

Для уже загруженного проекта:

```bash
./deploy.sh --lang ru
```

Параметр языка можно задать и переменной окружения:

```bash
PIXELCAT_LANG=ru ./deploy.sh
```

## Требования

- Linux с systemd: Debian, Ubuntu, RHEL-подобная система, Fedora или Alpine.
- Архитектура `amd64` или `arm64`.
- Домен с записью `A` или `AAAA`, указывающей на сервер.
- Открытые порты:
  - `80/tcp` и `443/tcp` для NaiveProxy;
  - `443/udp` для Hysteria2;
  - `20000-50000/udp` для диапазона смены портов по умолчанию.

Проверьте правила и в облачном firewall, и в firewall самой ОС.

## Выбор протокола

| Протокол | Транспорт | Когда использовать |
| --- | --- | --- |
| NaiveProxy | HTTPS / TCP | Обычная сеть, совместимость, трафик, похожий на стандартный HTTPS |
| Hysteria2 | QUIC / UDP | Высокая задержка, потери пакетов, мобильная или нестабильная сеть |

Практичный вариант — установить оба протокола и сравнить их на своём маршруте.

## Меню

После запуска выберите русский язык. Основные пункты:

```text
1) Установить / обновить PixelCat NaiveProxy
2) Установить / обновить PixelCat Hysteria2
3) Удалить PixelCat NaiveProxy
4) Удалить PixelCat Hysteria2
5) Включить BBR
6) Проверка качества IP
7) Проверка разблокировки стриминга
8) Качество сети / обратный маршрут
0) Выход
```

Справка по всем параметрам:

```bash
./deploy.sh --lang ru --help
```

## Установка NaiveProxy

Выберите пункт `1` и укажите:

1. домен прокси, например `proxy.example.com`;
2. имя пользователя;
3. надёжный пароль;
4. домен сайта-маскировки, например `www.example.com`, без `https://`;
5. email для Let's Encrypt, если нужен;
6. HTTP-порт `80` и HTTPS-порт `443`.

Неинтерактивный запуск:

```bash
./deploy.sh --lang ru --install -y \
  --domain proxy.example.com \
  --username your_user \
  --password change_this_strong_password \
  --decoy-domain www.example.com \
  --email admin@example.com
```

Передача пароля в командной строке может сохранить его в истории shell. Для рабочего сервера безопаснее вводить пароль интерактивно.

## Установка Hysteria2

Вернитесь в меню и выберите пункт `2`. Если NaiveProxy уже настроен, установщик может повторно использовать его домен, пароль и сертификат Caddy.

Рекомендуемые значения:

- UDP-порт: `443`;
- диапазон смены портов: `20000-50000`;
- лимиты входящей и исходящей скорости: `0` — без ограничения;
- URL маскировки: `https://www.example.com` или другой доступный HTTPS-сайт.

Неинтерактивный запуск:

```bash
./deploy.sh --lang ru --install-hysteria2 -y \
  --hy2-domain proxy.example.com \
  --hy2-port 443 \
  --hy2-hop-range 20000-50000 \
  --hy2-up-mbps 0 \
  --hy2-down-mbps 0 \
  --hy2-masquerade https://www.example.com
```

Чтобы отключить смену портов, используйте `--hy2-hop-range off`.

## Проверка после установки

```bash
systemctl status pixelcat-naiveproxy --no-pager
systemctl status pixelcat-hysteria2 --no-pager
systemctl status pixelcat-hysteria2-hop --no-pager

journalctl -u pixelcat-naiveproxy -f
journalctl -u pixelcat-hysteria2 -f

ss -lntup
```

Сервисы должны иметь состояние `active (running)`. Проверьте, что TCP 443 и UDP 443 доступны снаружи, а диапазон UDP открыт, если включена смена портов.

## Обновление и удаление

Повторный запуск входной команды загружает актуальную версию проекта. Удаление доступно через пункты `3` и `4` или параметры:

```bash
./deploy.sh --lang ru --uninstall
./deploy.sh --lang ru --uninstall-hysteria2
```

Добавляйте `--purge` только если хотите удалить локальную конфигурацию и данные сертификатов, указанные в подтверждении.

Проект: [github.com/PixelCatICU/pixelcat-proxy](https://github.com/PixelCatICU/pixelcat-proxy)
