# LYRICA (LyricApp) — синхронизация текстов к музыке

## Что это
Платформа для синхронизации текста песен с музыкой (word-level, на базе нейросетей). Дипломный проект ИТМО, защита близко.

## Архитектура

Ноут А (api.lyricapp.ru) — FastAPI + PostgreSQL (asyncpg, без ORM)
↑ fire-and-forget POST /process (X-Worker-Secret)
Ноут Б (worker) — Demucs (вокал) → Whisper (тайминги) → пишет результат прямо в БД
↓
Клиент В (браузер) — поллинг результат

БД доступна также по `db.lyricapp.ru`.

## Стек
- **API:** FastAPI, asyncpg, passlib/bcrypt, httpx
- **Worker:** Demucs, Whisper
- **Инфра:** Ubuntu Server 22.04, nginx + Certbot, systemd (`lyrica.service`, restart=always)
- **Домен/DNS:** lyricapp.ru (reg.ru) → CNAME на DuckDNS (DDNS через cron)

## Схема БД

`users, artists, albums, album_artists, tracks, sync_versions` — связи через `artist_id` (FK, не текстом). Обложки — по URL (FastAPI StaticFiles), не base64.

## Статус
Готово: схема БД, nginx+TLS, systemd-сервис, worker-интеграция (fire-and-forget)
В процессе: подготовка презентации к защите (12 слайдов, графики латентности/нагрузки)
После защиты: стабилизация хостинга, легальный контур (royalty-free аудио для демо)

## Ключевые решения (для памяти)
- Worker → сервер: fire-and-forget + поллинг клиента (не callback)
- `upsert_artist` всегда до `upsert_album` (FK)
- DDNS: только DuckDNS+CNAME, прямой API reg.ru — только для реселлеров
- `ssh`, не `sshd`, для перезапуска при правке конфига SSH на Ubuntu
