# Парсер ScienceDirect — запуск на сервере

Один контейнер, образ с Docker Hub, вывод в `/mnt/nfs_share/sd_parse`.

## Требования

- Docker (Docker Engine)
- NFS смонтирован в `/mnt/nfs_share` (на запись)

## Установка

1. Создайте `.env` в каталоге `sd_out` и укажите логин Docker Hub:

   ```bash
   echo "DOCKERHUB_USER=ваш_логин" > .env
   # или: nano .env
   ```

2. Убедитесь, что NFS смонтирован:

   ```bash
   mount | grep nfs_share
   ls -la /mnt/nfs_share
   ```

   Каталоги `sd_parse/data`, `sd_parse/logs`, `sd_parse/articles/...` создадутся при первом запуске.

## Запуск

Используется **один** контейнер, который работает постоянно. Скрипты запускаются по команде внутри него.

**Поднять контейнер (один раз):**

```bash
cd sd_out
docker compose pull
docker compose up -d
```

**Выполнить нужный скрипт по команде:**

```bash
docker compose exec parser run journals   # парсинг журналов
docker compose exec parser run articles   # парсинг статей
docker compose exec parser run issues     # парсинг выпусков
docker compose exec parser run download   # скачивание PDF
docker compose exec parser run help       # справка
```

**Остановить контейнер:**

```bash
docker compose down
```

### Краткая шпаргалка

| Команда                                   | Описание                                         |
| ----------------------------------------- | ------------------------------------------------ |
| `docker compose exec parser run journals` | Парсинг журналов → `sd_parse/data/journals.csv`  |
| `docker compose exec parser run articles` | Парсинг статей → `sd_parse/data/articles.csv`    |
| `docker compose exec parser run issues`   | Парсинг выпусков (не запускать, зарезервировано) |
| `docker compose exec parser run download` | Скачивание PDF → `sd_parse/articles/downloads/`  |

Порядок:
сначала `journals`,
затем `articles`
Для загрузки PDF — после появления `articles.csv` запускайте `download`.

## Куда пишутся файлы

При `SD_PARSE_OUTPUT=/mnt/nfs_share/sd_parse`:

- **/mnt/nfs_share/sd_parse/data/** — journals.csv, articles.csv, parsing_metadata.json
- **/mnt/nfs_share/sd_parse/logs/** — логи парсеров
- **/mnt/nfs_share/sd_parse/articles/downloads/** — скачанные PDF
- **/mnt/nfs_share/sd_parse/articles/logs/** — логи и метаданные скрипта загрузки

Чтобы писать в другое место, задайте в `.env` переменную `SD_PARSE_OUTPUT` и при необходимости измените volume в `docker-compose.yml` (смонтировать свой каталог в `/mnt/nfs_share` или подставить другой путь в `SD_PARSE_OUTPUT` и добавить соответствующий volume).
