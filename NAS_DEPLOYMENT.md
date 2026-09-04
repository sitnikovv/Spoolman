# Запуск версии с OpenSpool QR на NAS

Файл `compose.nas.yaml` собирает новый интерфейс из ветки этого форка и
накладывает его на официальный серверный образ Spoolman 0.26.1. База данных и
старый интерфейс при этом не изменяются.

## Перед переключением контейнера

1. В Synology Container Manager откройте действующий контейнер Spoolman.
2. Запишите его переменные `PUID`, `PGID`, `TZ` и внешний порт.
3. Найдите подключение каталога к
   `/home/app/.local/share/spoolman`. Нужен путь на NAS из левой части этого
   подключения.
4. Сделайте резервную копию каталога. Для SQLite достаточно копии файла
   `spoolman.db`, созданной после остановки контейнера.

Новый compose-файл намеренно требует явный `SPOOLMAN_DATA_DIR`: это защищает от
случайного запуска с пустой базой вместо существующей.

## Сборка из консоли NAS

```bash
git clone --branch deploy/openspool-qr-nas https://github.com/sitnikovv/Spoolman.git
cd Spoolman
cp .env.nas.example .env.nas
```

Отредактируйте `.env.nas`, подставив параметры действующего контейнера, затем
соберите образ, не останавливая работающий Spoolman:

```bash
docker compose --env-file .env.nas -f compose.nas.yaml build
```

После успешной сборки остановите старый контейнер в Container Manager и
запустите новый:

```bash
docker compose --env-file .env.nas -f compose.nas.yaml up -d
docker compose --env-file .env.nas -f compose.nas.yaml logs --tail=100
```

Откройте `http://192.168.7.250:7912`, выберите катушку и нажмите
`OpenSpool QR` рядом с кнопкой печати этикеток.

## Откат

Если новый контейнер не запускается, выполните:

```bash
docker compose --env-file .env.nas -f compose.nas.yaml down
```

После этого снова запустите прежний контейнер. Пока оба контейнера используют
один каталог данных, нельзя запускать их одновременно.
