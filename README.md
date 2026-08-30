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
