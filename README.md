# [Sing-Box DNS Configurator](https://kira361323.github.io/sing-box-generator/)

Веб-конфигуратор для генерации конфигураций **sing-box** с управлением DNS-маршрутизацией по приложениям, доменам и удалённому `Rule-Set`.

Проект представляет собой статический сайт и может размещаться на **GitHub Pages**.

## Архитектура DNS

Конфигуратор рассчитан на следующую схему:

| Запрос | DNS |
|---|---|
| Исключённые приложения | DNS системы / провайдера / мобильного оператора |
| Исключённые домены | DNS системы / провайдера / мобильного оператора |
| Выбранные приложения | Redirect DNS |
| Выбранные домены | Redirect DNS |
| Домены из `domains.srs` | Redirect DNS |
| Все остальные запросы | Direct DNS |

Логика:

```text
DNS-запрос
    │
    ├── Исключённое приложение ──────► DNS системы/провайдера
    │
    ├── Исключённый домен ───────────► DNS системы/провайдера
    │
    ├── Выбранное приложение ────────► Redirect DNS
    │
    ├── Выбранный домен ─────────────► Redirect DNS
    │
    ├── domains.srs ─────────────────► Redirect DNS
    │
    └── Остальное ───────────────────► Direct DNS
```

> Важно: DNS-маршрутизация и маршрутизация самого TCP/UDP-трафика — разные уровни. Этот конфигуратор управляет именно DNS-запросами.

## Rule-Set `domains.srs`

По умолчанию используется:

```text
https://github.com/Kira361323/Dns-rules/releases/latest/download/domains.srs
```

Rule-Set подключается как удалённый binary Rule-Set:

```json
{
  "type": "remote",
  "tag": "domains",
  "format": "binary",
  "url": "https://github.com/Kira361323/Dns-rules/releases/latest/download/domains.srs",
  "update_interval": "24h"
}
```

Домены, совпавшие с Rule-Set, направляются в `Redirect DNS`.

## Кэш

Конфигурация включает механизм кэширования sing-box:

```json
{
  "experimental": {
    "cache_file": {
      "enabled": true,
      "path": "cache.db"
    }
  }
}
```

Путь к кэшу задаётся через интерфейс.

Важно: `cache_file` — это встроенный механизм кэширования sing-box. Он не означает, что `domains.srs` автоматически превращается в обычный локальный файл, доступный по произвольному пути.

Поэтому не следует вручную добавлять в конфигурацию:

```text
/storage/emulated/0/Android/data/.../domains.srs
```

если этот файл физически не существует.

Используется именно удалённый Rule-Set:

```json
{
  "type": "remote",
  "format": "binary",
  "url": "..."
}
```

## Порядок правил

Порядок правил имеет критическое значение.

Рекомендуемая логика:

```text
1. Исключённые приложения
2. Исключённые домены
3. Пользовательские приложения → Redirect DNS
4. Пользовательские домены → Redirect DNS
5. domains.srs → Redirect DNS
6. Остальные запросы → Direct DNS
```

Более специфичные правила должны обрабатываться раньше общих.

## Redirect DNS

В конфигуратор включены:

- Xbox DNS
- Comss DNS
- Malw Link
- Astracat
- Mafioznik
- Geohide

Все серверы используются через DoH.

## Direct DNS

В конфигуратор включены:

- AliDNS
- BebasID Standard
- BebasID OISD
- BebasID HaGeZi
- Cloudflare
- DNS.SB
- DNSPub
- Google DNS
- Mullvad Adblock
- OpenBLD Ada
- Quad9 Standard
- Quad9 ECS
- Applied Privacy
- Digitale Gesellschaft
- JoinDNS4 NoAds
- LibreDNS Standard
- LibreDNS Ads
- DNSForge
- HaGeZi Root
- HaGeZi Wurzn
- HaGeZi Juuri
- Tiarap
- TiarApp

## Обработка DoH URL

Конфигуратор разбирает URL DoH и формирует отдельные поля sing-box.

Например:

```text
https://dns.alidns.com/dns-query
```

преобразуется в:

```json
{
  "type": "https",
  "tag": "direct-dns",
  "server": "dns.alidns.com",
  "server_port": 443,
  "path": "/dns-query"
}
```

Для нестандартного порта:

```text
https://dns.geohide.ru:444/dns-query
```

генерируется:

```json
{
  "type": "https",
  "tag": "redirect-dns",
  "server": "dns.geohide.ru",
  "server_port": 444,
  "path": "/dns-query"
}
```

Это предотвращает ошибку, когда полный URL ошибочно передаётся в поле `server`.

Некорректная структура:

```json
{
  "server": "https://https://dns.alidns.com/dns-query"
}
```

может привести к ошибкам вида:

```text
invalid port
```

## Bootstrap DNS

Для разрешения доменных имён DoH-серверов используется Bootstrap DNS.

По умолчанию:

```text
8.8.8.8
```

Он нужен для первоначального разрешения имён:

```text
dns.alidns.com
dns.cloudflare.com
dns.google
```

и других DNS-серверов, заданных доменными именами.

## TUN

Для перехвата DNS-запросов приложений используется TUN inbound.

Пример:

```json
{
  "type": "tun",
  "tag": "tun-in",
  "interface_name": "tun0",
  "address": [
    "172.19.0.1/30"
  ],
  "auto_route": true,
  "strict_route": true,
  "stack": "gvisor",
  "mtu": 1400
}
```

DNS перехватывается правилом:

```json
{
  "protocol": "dns",
  "action": "hijack-dns"
}
```

## Важное ограничение

Этот проект управляет DNS-маршрутизацией.

Он не является прокси и не определяет автоматически маршрут последующего TCP/UDP-соединения.

Например:

```text
ChatGPT
    │
    ├── DNS-запрос ─────► Redirect DNS
    │
    └── HTTPS-трафик ───► определяется правилами route/outbound
```

Поэтому изменение DNS-сервера не означает автоматического изменения маршрута всего трафика приложения.

## Установка через GitHub Pages

1. Создайте GitHub-репозиторий.
2. Добавьте в корень:

```text
index.html
README.md
```

3. Откройте:

```text
Settings → Pages
```

4. Выберите:

```text
Deploy from a branch
```

5. Укажите:

```text
main
/
```

После публикации сайт будет доступен по адресу:

```text
https://USERNAME.github.io/REPOSITORY/
```

## Локальный запуск

Можно открыть `index.html` непосредственно в браузере.

Либо запустить локальный HTTP-сервер:

```bash
python -m http.server 8080
```

Затем открыть:

```text
http://localhost:8080
```

## Использование

1. Выберите `Redirect DNS`.
2. Выберите `Direct DNS`.
3. Проверьте URL `domains.srs`.
4. Выберите интервал обновления Rule-Set.
5. Укажите путь к `cache.db`.
6. Добавьте приложения для Redirect DNS.
7. Добавьте домены для Redirect DNS.
8. Добавьте исключённые приложения.
9. Добавьте исключённые домены.
10. Нажмите **«Скачать конфигурацию Sing-Box (.json)»**.
11. Импортируйте полученный JSON в совместимый клиент sing-box.

## Пример настройки

```text
Redirect DNS:
Xbox DNS

Direct DNS:
DNSForge
```

Приложения:

```text
com.openai.chatgpt
com.anthropic.claude
com.deepseek.chat
```

Домены:

```text
openai.com
anthropic.com
deepseek.com
```

Исключения:

```text
sberbank.ru
gosuslugi.ru
```

Результат:

```text
OpenAI / Claude / DeepSeek
        │
        ▼
   Redirect DNS

domains.srs
        │
        ▼
   Redirect DNS

sberbank.ru / gosuslugi.ru
        │
        ▼
DNS системы / провайдера

Всё остальное
        │
        ▼
    Direct DNS
```

## Структура репозитория

Минимальный вариант:

```text
.
├── index.html
└── README.md
```

Если Rule-Set хранится в отдельном репозитории:

```text
DNS Configurator
├── index.html
└── README.md

DNS Rules
└── domains.srs
```

## Совместимость

Проект ориентирован на версии sing-box, поддерживающие используемые возможности:

- TUN inbound;
- DNS server type `https`;
- `domain_resolver`;
- DNS rules;
- `package_name`;
- remote binary Rule-Set;
- `.srs`;
- `experimental.cache_file`.

Перед использованием рекомендуется проверить версию ядра sing-box в конкретном клиенте.

Клиенты на базе sing-box могут использовать разные версии ядра и иметь собственные ограничения.

## Безопасность

Конфигуратор является статическим HTML-приложением.

Он:

- не требует собственного backend;
- не требует базы данных;
- генерирует JSON непосредственно в браузере;
- не требует регистрации;
- не хранит конфигурацию на сервере проекта.

Не добавляйте в поля конфигуратора пароли, приватные ключи, токены или другие секретные данные.

## Лицензия

Если отдельная лицензия не указана, автор сохраняет права на исходный код.

Для публикации проекта с открытым исходным кодом рекомендуется добавить файл `LICENSE`, например с лицензией MIT.

## Disclaimer

Проект является инструментом генерации конфигураций sing-box.

Совместимость зависит от версии sing-box и конкретного клиента.

Перед использованием рекомендуется проверить:

1. версию sing-box;
2. поддержку TUN;
3. поддержку DNS `package_name`;
4. поддержку remote Rule-Set;
5. поддержку binary `.srs`;
6. поддержку `experimental.cache_file`;
7. особенности реализации конкретного Android-клиента.

