# 🌉BridgeNode

> **Краткое описание:** 🌉 Самохостируемый сетевой мост, который пропускает трафик Cloudflare Tunnel через Xray-прокси клиент (VLESS/VMess/XTLS). Предназначен для окружений, где прямые исходящие соединения ограничены или контролируются.

## Архитектура

```
Входящий запрос (через Cloudflare Tunnel)
        │
        ▼
┌───────────────────┐    SOCKS5      ┌─────────────────┐
│   cloudflared     │ ─────────────► │   xray-client   │ ──► Интернет
│  (Tunnel клиент)  │  :10808        │  (Прокси клиент)│
└───────────────────┘                └─────────────────┘
        │
   internet_bridge (внешняя Docker-сеть) → проксирует трафик к бэкенд-сервисам
```

## Компоненты

| Сервис        | Образ                    | Роль                          |
| ------------- | ------------------------ | ----------------------------- |
| `xray-client` | `teddysun/xray`          | SOCKS5-прокси через ядро Xray |
| `cloudflared` | `cloudflare/cloudflared` | Демон Cloudflare Tunnel       |

## Требования

- Docker и Docker Compose
- Cloudflare Tunnel (`credentials.json` + `config.yml`)
- Xray-сервер с валидным конфигом (`xray-config.json`)
- Существующая внешняя Docker-сеть `internet_bridge`

## Установка

1. **Клонировать репозиторий**

```bash
   git clone https://github.com/Sh1yden/BridgeNode.git
   cd BridgeNode
```

2. **Добавить конфигурационные файлы** (не включены, в gitignore)

```
   xray-config.json     # Конфигурация Xray-клиента
   config.yml           # Конфигурация cloudflared тоннеля
   credentials.json     # Credentials Cloudflare тоннеля
   .env                 # Переменные окружения
```

3. **Создать внешнюю сеть** (если ещё не создана)

```bash
   docker network create internet_bridge
```

4. **Запустить стек**

```bash
   docker compose up -d
```

## Сетевая схема

| Сеть              | Тип        | Назначение                               |
| ----------------- | ---------- | ---------------------------------------- |
| `egress_internal` | Внутренняя | Приватный канал между Xray и cloudflared |
| `internet_bridge` | Внешняя    | Общая сеть с бэкенд-сервисами            |

Cloudflare Tunnel подключается к бэкенд-контейнерам через `internet_bridge`,
при этом весь исходящий трафик идёт через Xray по внутреннему SOCKS5-прокси
(`socks5://xray_client:10808`).

## Безопасность

- Конфиги с учётными данными исключены из VCS через `.gitignore` и `.dockerignore`
- Xray-прокси не выставлен в хостовую сеть — доступен только внутри `egress_internal`
- `NO_PROXY` выставлен для localhost и имён внутренних сервисов, чтобы
  внутренний трафик не шёл через прокси

## Лицензия

MIT — см. [LICENSE](LICENSE)

## 💜 Вопросы, контакты / FAQ

Для вопросов и предложений создавайте Issues в репозитории или же пишите в телеграмм который указан в профиле.

PS Любой помощи или совету буду очень благодарен.
