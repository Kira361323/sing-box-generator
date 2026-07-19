# 🛡️ [Sing-Box DNS Configurator](https://kira361323.github.io/sing-box-generator/)

Генератор DNS-профилей для [sing-box](https://github.com/SagerNet/sing-box) ≥ 1.11.
Работает полностью в браузере — без бэкенда, без сбора данных.

---

## ✨ Возможности

| Функция | Описание |
|---------|----------|
| **Фиксированный профиль** | Один Redirect + один Direct DNS |
| **Fallback-профиль** | Все DNS категории, перебор при таймауте |
| **55 DNS-серверов** | 13 Redirect + 42 Direct (DoH / DoT) |
| **Bootstrap selector** | 8 предустановленных IP + ручной ввод |
| **Кастомный DNS** | Свой DoH / DoT / UDP сервер |
| **Rule-Set** | Удалённый `.srs` с автообновлением |
| **Исключения** | Домены → Direct, приложения → bypass TUN |
| **DNS Strategy** | `ipv4_only` / `prefer_ipv4` / `prefer_ipv6` / `ipv6_only` |
| **Логирование** | От `off` (без секции log) до `trace` |
| **Описания** | Краткое пояснение к каждому полю |

---

## 🚀 Использование

### Онлайн (GitHub Pages)

1. Откройте **[конфигуратор](https://kira361323.github.io/sing-box-generator/)**
2. Выберите **Bootstrap DNS** (проверьте доступность: `dig @IP xbox-dns.ru`)
3. Выберите **Redirect DNS** и **Direct DNS**
4. При необходимости добавьте свои домены / приложения / исключения
5. Нажмите **«Скачать конфигурацию»**
6. Импортируйте `.json` в sing-box

### Локально

```bash
git clone https://github.com/Kira361323/Dns-rules.git
cd Dns-rules
# Откройте index.html в браузере
```

---

## 🏗️ Архитектура конфига

```
┌─────────────────────────────────────────────────────┐
│                    TUN (gvisor)                      │
│              172.19.0.1/30 · MTU 1400               │
│         auto_route · strict_route                   │
│         exclude_package: [com.whatsapp]             │
└──────────────────────┬──────────────────────────────┘
                       │
              ┌────────▼────────┐
              │   Route Rules   │
              │                 │
              │  1. sniff       │
              │  2. dns→hijack  │
              │  3. :853→hijack │
              │  4. :53→hijack  │
              │  5. private→dir │
              └────────┬────────┘
                       │
         ┌─────────────▼─────────────┐
         │        DNS Rules          │
         │                           │
         │  excludeDomains → Direct  │
         │  package_name  → Redirect │
         │  customDomains → Redirect │
         │  rule_set      → Redirect │
         │  final         → Direct   │
         └─────┬───────────────┬─────┘
               │               │
    ┌──────────▼──┐    ┌──────▼──────────┐
    │ Redirect DNS│    │   Direct DNS    │
    │ (DoH :443)  │    │   (DoH :443)   │
    │ xbox-dns.ru │    │ dnsforge.de     │
    └─────────────┘    └─────────────────┘
               │               │
         ┌─────▼───────────────▼─────┐
         │   Bootstrap (UDP :53)     │
         │   77.88.8.8 (Яндекс)      │
         │   Резолвит hostname DNS   │
         └───────────────────────────┘
```

---

## 📋 DNS-серверы

### Redirect (13)

| Сервер | DoT | DoH | Примечание |
|--------|-----|-----|-----------|
| Astracat | `dns.astracat.ru` | ✅ | |
| Comss | `dns.comss.one` | ✅ | |
| Geohide | `dns.geohide.ru` | ✅ :444 | Нестандартный порт |
| Mafioznik | `dns.mafioznik.xyz` | ✅ | |
| Malw Link | `dns.malw.link` | ✅ | |
| Shecan | `free.shecan.ir` | — | Только DoT |
| Xbox DNS | `xbox-dns.ru` | ✅ | |

### Direct (42)

| Сервер | DoT | DoH | Фильтрация |
|--------|-----|-----|-----------|
| AliDNS | `dns.alidns.com` | ✅ | — |
| Applied Privacy | `dot1.applied-privacy.net` | ✅ `/query` | — |
| BebasID | `dns.bebasid.com` | ✅ + OISD + HaGeZi | Ads/Trackers |
| Cloudflare | `one.one.one.one` | ✅ | — |
| Digitale Gesellschaft | `dns.digitale-gesellschaft.ch` | ✅ | — |
| DNS.SB | `dot.sb` | ✅ | — |
| DNSForge | `dnsforge.de` | ✅ | Ads/Trackers/Malware |
| DNSPod / Pub | `dot.pub` | ✅ | — |
| Google | `dns.google` | ✅ | — |
| HaGeZi Juuri | `juuri.hagezi.org` | ✅ | Ultimate |
| HaGeZi Root | `root.hagezi.org` | ✅ | Ultimate |
| HaGeZi Wurzn | `wurzn.hagezi.org` | ✅ | Ultimate |
| JoinDNS4 | `noads.joindns4.eu` | ✅ | Ads |
| LibreDNS | `dot.libredns.gr` | ✅ + `/ads` | Ads |
| Mullvad | `adblock.dns.mullvad.net` | ✅ | Ads/Trackers |
| OpenBLD Ada | `ada.openbld.net` | ✅ | Ads/Malware |
| Quad9 Standard | `dns.quad9.net` | ✅ | Malware |
| Quad9 ECS | `dns11.quad9.net` | ✅ | Malware + ECS |
| Tiarap | `dot.tiar.app` | ✅ | — |

---

## ⚙️ Требования

| Компонент | Версия |
|-----------|--------|
| **sing-box** | ≥ **1.11** (используется `domain_resolver`, `action: "sniff"`, `ip_is_private`) |
| **Android** | sing-box app или SFA/SFI |
| **Платформа** | Android, Linux, Windows, macOS |

> ⚠️ На sing-box **1.10 и ниже** конфиг **не запустится**.

---

## ❓ FAQ

### DoT или DoH?

| | DoT (:853) | DoH (:443) |
|---|---|---|
| **Порт** | 853 (может блокироваться DPI) | 443 (неотличим от HTTPS) |
| **Шифрование** | TLS | TLS + HTTP/2 |
| **Надёжность** | Может блокироваться | Практически не блокируется |
| **Рекомендация** | Если DoT не блокируется | **По умолчанию** |

### Что такое Bootstrap DNS?

Плоский UDP-DNS (порт 53), который резолвит hostname ваших DoH/DoT-серверов в IP. Без него sing-box не знает, куда подключаться. Должен быть доступен в вашей сети.

### Почему `ipv4_only`?

Если провайдер не маршрутизирует IPv6, sing-box будет таймаутить на AAAA-записях. `ipv4_only` исключает эти задержки.

### Что такое Rule-Set?

Бинарный `.srs`-файл со списком доменов. Домены из него маршрутизируются в Redirect DNS. Обновляется автоматически с заданным интервалом.

---
