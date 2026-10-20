# Создание демонов

## Описание
Домашнее задание по дисциплине "Операционные системы и среды". Цель работы — научиться создавать собственный демон (фоновую программу-сервис), писать его конфигурационный файл для systemd, запускать его на стороне пользователя и на стороне системы, а также проверять работу через журнал. Работа выполняется в гостевой ОС Debian (VirtualBox), в терминале.

## Что понадобится
- Oracle VirtualBox с импортированным образом ВМ.
- Терминал ВМ, вход как root (или использование `sudo`).
- Word/PDF для оформления отчёта.

## Задача
Создать программу, которая раз в несколько секунд пишет в системный журнал, что она активна. При запуске — запись о старте, при завершении по сигналам SIGTERM и SIGINT — запись об остановке.

## Скрипт демона (active.sh)

```bash
#!/bin/bash

trap 'logger -p local3.info -t "ScriptActive" "Programms work has ended"; exit 0' SIGTERM SIGINT

logger -p local3.info -t "ScriptActive" "Programms work has started"

while true
do
    sleep 3
    logger -p local3.info -t "ScriptActive" "Programm is active"
done
```

Дать файлу права на исполнение: `chmod 755 active.sh`.

## Часть 1. Демон на стороне пользователя

| № | Задание | Команда |
|---|---------|---------|
| 1 | Создать директорию для пользовательских сервисов | `mkdir -p ~/.config/systemd/user/` |
| 2 | Создать файл сервиса | `touch ~/.config/systemd/user/ScriptActive.service` |
| 3 | Заполнить файл сервиса | см. листинг ниже |
| 4 | Применить изменения в systemd | `systemctl --user daemon-reload` |
| 5 | Проверить, распознаётся ли демон | `systemctl --user status ScriptActive` |
| 6 | Запустить демон | `systemctl --user start ScriptActive` |
| 7 | Убедиться, что демон запущен, и прочитать журнал | `systemctl --user status ScriptActive`; `journalctl -t "ScriptActive" -f` |
| 8 | Остановить демон | `systemctl --user stop ScriptActive` |
| 9 | Убедиться, что в журнале есть запись об остановке | `journalctl -t "ScriptActive"` |

### Файл ~/.config/systemd/user/ScriptActive.service

```ini
[Unit]
Description=Example Service Daemon
After=syslog.service

[Service]
ExecStart=/root/active.sh
Restart=no
```

## Часть 2. Демон на стороне системы

| № | Задание | Команда |
|---|---------|---------|
| 1 | Войти в сеанс суперпользователя | (уже root) или `sudo` |
| 2 | Скопировать файл сервиса в системный каталог под новым именем | `cp ~/.config/systemd/user/ScriptActive.service /etc/systemd/system/SystemActive.service` |
| 3 | Отредактировать файл: добавить `User`, `WorkingDirectory`, секцию `[Install]` | `vim SystemActive.service` |
| 4 | Применить изменения | `systemctl daemon-reload` |
| 5 | Проверить, распознаётся ли демон | `systemctl status SystemActive` |
| 6 | Включить автозапуск | `systemctl enable SystemActive` |
| 7 | Перезагрузить систему и проверить автозапуск | `reboot`; `systemctl status SystemActive` |
| 8 | Проверить журнал | `journalctl -t "SystemActive" -f` |
| 9 | Отключить автозапуск | `systemctl disable SystemActive` |

### Файл /etc/systemd/system/SystemActive.service

```ini
[Unit]
Description=Example Service Daemon
After=syslog.service

[Service]
ExecStart=/usr/local/bin/active.sh
Restart=no
User=user
WorkingDirectory=/home/user

[Install]
WantedBy=multi-user.target
```

## Замечания по выполнению
- Демон — это программа, работающая в фоне без участия пользователя; аналог службы (service) в Windows.
- Управление демонами осуществляется через `systemctl`, который управляет главным демоном `systemd`.
- Условия для демона: программа должна быть исполняемым файлом и полностью автономной.
- `logger -p local3.info -t "имя" "сообщение"` — запись в журнал с указанием имени программы и уровня info.
- `trap "команда" SIGTERM` — установка обработчика сигнала; в скрипте обрабатываются и SIGTERM, и SIGINT.
- На Debian 13 (Trixie) файла `/var/log/syslog` может не быть (журналирование через systemd-journald). В этом случае используйте `journalctl -t "имя" -f` для просмотра в реальном времени и `journalctl -t "имя"` для просмотра всей истории.
- Разница сторон:
  - Пользовательский сервис (`~/.config/systemd/user/`) запускается вручную пользователем, `systemctl --user ...`.
  - Системный сервис (`/etc/systemd/system/`) может запускаться автоматически при старте системы, `systemctl ...` (без `--user`), для автозапуска нужно `systemctl enable`.
- Важная ошибка из отчёта: скрипт лежал в `/root/active.sh`, и при автозапуске systemd не мог его выполнить (нет прав/доступа к домашней директории root). Решение — скопировать скрипт в `/usr/local/bin/` и указать в `ExecStart` корректный путь.
- `systemctl enable` создаёт симлинк в `multi-user.target.wants/`, `disable` — удаляет его.
- При правке системного файла сервиса после копирования меняйте имя демона (например, `SystemActive` вместо `ScriptActive`), чтобы в системе не было двух одинаковых сервисов.

## Требования к отчёту
- Оформляется в Word/PDF.
- Каждый шаг фиксируется: команда + скриншот вывода.
- В конце — вывод: какие знания извлечены, какие приёмы создания демонов освоены.

## Файлы
- `ДЗ. Создание демонов.pdf` — методичка
- `Отчет создание демонов.docx` — выполненный отчёт