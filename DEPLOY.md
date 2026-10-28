# Развёртывание экскурсии

Экскурсия — это статический сайт из папки `excursion/` репозитория
[Skorpion4853/Excursions](https://github.com/Skorpion4853/Excursions).
Разместить её можно двумя способами:

- **На GitHub Pages** — бесплатно, сервер не нужен. См. раздел [«Форк и размещение на своём GitHub»](#форк-и-размещение-на-своём-github).
- **На своём сервере в Docker** — см. раздел [«Развёртывание на сервере (Docker)»](#развёртывание-на-сервере-docker).

Готовая экскурсия: <https://skorpion4853.github.io/Excursions/excursion/>

## Форк и размещение на своём GitHub

Форк — это ваша копия репозитория. В ней можно менять названия и фото, не трогая оригинал,
и бесплатно опубликовать экскурсию через GitHub Pages.

### 1. Сделать форк

1. Войдите в свой аккаунт на <https://github.com> (если аккаунта нет — зарегистрируйтесь).
2. Откройте <https://github.com/Skorpion4853/Excursions>.
3. Нажмите **Fork** в правом верхнем углу, затем **Create fork**.
   При желании поменяйте название в поле *Repository name* — тогда в адресах ниже
   вместо `Excursions` будет ваше название.

Через несколько секунд появится копия по адресу `https://github.com/ВАШ_ЛОГИН/Excursions`.

### 2. Включить GitHub Pages

1. В своём форке откройте **Settings → Pages**.
2. В блоке *Build and deployment* выберите **Source: Deploy from a branch**,
   ветку **main** и папку **/ (root)**, нажмите **Save**.
3. Подождите 1–3 минуты. Ход сборки виден на вкладке **Actions**
   (задача *pages build and deployment* должна стать зелёной).

Экскурсия откроется по адресу:

```
https://ВАШ_ЛОГИН.github.io/Excursions/excursion/
```

Логин в адресе пишется маленькими буквами. Если открывается 404 — подождите ещё пару минут
и обновите страницу с очисткой кэша (Ctrl+F5).

Важно: на бесплатном аккаунте GitHub Pages работает только для **публичных** репозиториев.
Форк публичного репозитория публичный по умолчанию.

### 3. Поменять названия и маршрут

Названия помещений, этажи и переходы хранятся в файле `excursion/scenes.js`.
Его можно поправить прямо на сайте GitHub:

1. Откройте в форке `excursion/scenes.js` и нажмите на значок карандаша (**Edit this file**).
2. Измените `title` (название), `floor` (этаж) или `links` (куда можно пройти — номера сцен).
   Порядок строк — порядок шагов экскурсии.
3. Нажмите **Commit changes**. Через 1–3 минуты Pages пересоберёт сайт.

Панорамы лежат в `excursion/panos/` (полноразмерные) и `excursion/thumbs/` (миниатюры);
имя файла — номер сцены (`id` в `scenes.js`).

### 4. Получать обновления из оригинала

Если в оригинальном репозитории появились изменения, на главной странице форка нажмите
**Sync fork → Update branch**. Если вы меняли те же файлы, GitHub предложит разобраться
с конфликтом вручную.

### Клонировать к себе на компьютер (по желанию)

```bash
git clone https://github.com/ВАШ_ЛОГИН/Excursions.git
cd Excursions
python3 -m http.server 8000
```

Экскурсия откроется на <http://localhost:8000/excursion/>. Просто открыть `index.html`
двойным щелчком не получится: браузер не даст странице загрузить панорамы с диска.

# Развёртывание на сервере (Docker)

Docker-образ собирает экскурсию вместе с веб-сервером nginx. В образ попадает только экскурсия
(около 30 МБ панорам), исходные фото из `photos/` и старый тур `3Ddata/` туда не входят.

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
git clone https://github.com/Skorpion4853/Excursions.git
cd Excursions
```

Если вы сделали форк, подставьте свой адрес: `https://github.com/ВАШ_ЛОГИН/Excursions.git`.
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
cd Excursions
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
