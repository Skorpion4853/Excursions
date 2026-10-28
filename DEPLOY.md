# Развёртывание экскурсии на сервере (Docker)

Экскурсия — это статический сайт из папки `excursion/`. Docker-образ собирает её
вместе с веб-сервером nginx. В образ попадает только экскурсия (около 30 МБ панорам),
исходные фото из `photos/` и старый тур `3Ddata/` туда не входят.

## Что нужно

- Сервер на Linux (подойдёт любой VPS, например Ubuntu 22.04/24.04) с 1 ГБ памяти и 1 ГБ свободного места.
- Доступ по SSH с правами `sudo`.
- Открытый порт 80 (и 443, если нужен HTTPS).

## 1. Установить Docker

На Ubuntu/Debian:

```bash
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER   # чтобы запускать docker без sudo
```

После `usermod` выйдите из SSH и зайдите снова. Проверка:

```bash
docker --version
docker compose version
```

## 2. Скачать проект

```bash
sudo apt-get install -y git
git clone https://github.com/Skorpion4853/fuckingStudents.git
cd fuckingStudents
```

Репозиторий весит больше 300 МБ из-за исходных фото. Если место на сервере ограничено,
можно скачать без истории: `git clone --depth 1 ...`.

## 3. Запустить

```bash
docker compose up -d --build
```

Экскурсия откроется на `http://IP-сервера:8080/`.

Чтобы открыть её сразу на 80-м порту (`http://IP-сервера/`):

```bash
PORT=80 docker compose up -d --build
```

Порт можно также записать в файл `.env` рядом с `docker-compose.yml`: строка `PORT=80`.

Контейнер перезапускается сам после перезагрузки сервера (`restart: unless-stopped`).

## 4. Проверить

```bash
docker compose ps          # статус должен быть "healthy"
curl -I http://localhost:8080/
docker compose logs -f     # журнал запросов, выход — Ctrl+C
```

Если сайт не открывается снаружи, проверьте файрвол:

```bash
sudo ufw allow 80/tcp      # или 8080/tcp, смотря какой порт выбран
```

## 5. Обновить экскурсию

После изменений в репозитории (новые названия, другие фото):

```bash
cd fuckingStudents
git pull
docker compose up -d --build
```

Старый контейнер заменится новым, простой займёт несколько секунд.

## Домен и HTTPS (по желанию)

Если у вас есть домен (например `tour.example.ru`), направьте его A-запись на IP сервера.
Самый простой способ получить HTTPS — поставить перед контейнером Caddy, он сам получает
сертификат Let's Encrypt.

1. Запустите экскурсию на внутреннем порту: в `.env` оставьте `PORT=8080`.
2. Установите Caddy: <https://caddyserver.com/docs/install#debian-ubuntu-raspbian>.
3. Замените содержимое `/etc/caddy/Caddyfile` на:

   ```
   tour.example.ru {
       reverse_proxy localhost:8080
   }
   ```

4. `sudo systemctl reload caddy` — через минуту сайт откроется на `https://tour.example.ru/`.

## Сервер без доступа к GitHub

Образ можно собрать на своём компьютере и перенести файлом:

```bash
# на своём компьютере, в папке проекта
docker build -t licey-excursion .
docker save licey-excursion | gzip > licey-excursion.tar.gz
scp licey-excursion.tar.gz user@IP-сервера:~

# на сервере
gunzip -c licey-excursion.tar.gz | docker load
docker run -d --name licey-excursion --restart unless-stopped -p 80:80 licey-excursion
```

## Остановить и удалить

```bash
docker compose down            # остановить контейнер
docker image rm licey-excursion  # удалить образ
```

## Что внутри

| Файл | Назначение |
|---|---|
| `Dockerfile` | образ на `nginx:1.27-alpine`, копирует `excursion/` |
| `docker/nginx.conf` | настройки nginx: сжатие, кэш панорам на 30 дней, `scenes.js` без кэша |
| `docker-compose.yml` | запуск одной командой, порт задаётся переменной `PORT` (по умолчанию 8080) |
| `.dockerignore` | в образ не попадает ничего, кроме экскурсии и конфига |
