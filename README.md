# Лабораторная: Продакшн-образы Docker

**Цель:** собрать production-образ для мини-сервиса на Flask так, чтобы он был:
- маленький (**< 150 MB**),
- безопасный (**не root**),
- с **HEALTHCHECK**,
- корректно завершался по **SIGTERM**,
- умел работать при `--read-only`,
- принимал конфиг через переменные окружения **без пересборки**.

Эта лаба **персонифицирована**: каждый получает свой `seed-<login>.json`.  
Вы обязаны проставить **LABEL** с вашим логином/токеном и опубликовать образ в **Docker Hub** под своим аккаунтом.

---

## Необходимые инструменты

1. **Docker Desktop** (Windows/macOS/Linux) — иконка кита должна быть активна.  
2. **Git** и **jq** (на Windows удобно через Git Bash; `jq` можно поставить через `choco install jq`).

---

## Структура репозитория

```
app/                # мини-сервис Flask с / и /health и /proof (уже готов)
  app.py
requirements.txt
.dockerignore
README.md           
```

> Файла `Dockerfile` здесь **нет**: вы создаете его сами, по заданию ниже.

---

## Задание (что именно сделать)

1. Написать **один** `Dockerfile` со сборкой production-образа (multi-stage).
2. Требования к собранному **prod**-образу:
   - Размер образа **< 150 MB**.
   - Есть **HEALTHCHECK** (проверяет `GET /health`).
   - Запуск под **не-root** пользователем (`USER`).
   - Корректное завершение по **SIGTERM**.
   - Контейнер можно запускать на **`--read-only`**.
   - Переменная окружения **`ROCKET_SIZE`** влияет на ответ `/` **без пересборки**.
   - В **LABEL** должны быть:
     - `org.lab.login=<ваш_dockerhub_login>`
     - `org.lab.token=<token>` (из вашего seed-файла)

3. Публично опубликовать образ в Docker Hub по имени:
   ```
   <ваш_dockerhub_login>/image-lab:<prefix>
   ```
   где `prefix` — это первые 8 символов `token` из вашего seed-файла.

---

## Как получить ваш seed и как им пользоваться

Вам будет выдан вам файл `seed-<login>.json`(для его получения нужно написать в личные сообщения свой dockerhub_login), где:
```json
{
  "login":  "<ваш_dockerhub_login>",
  "token":  "<секретный_токен>",
  "prefix": "<первые_8_символов_token>"
}
```

Сохраните этот файл в корень репозитория (не коммитьте в публичный форк).

---

## Шаги выполнения

### 1) Локальная сборка prod-образа

Откройте терминал в корне репозитория и выполните:

```bash
# Прочитаем поля из seed-файла
SEED_JSON=seed-<login>.json
LOGIN=$(jq -r .login  "$SEED_JSON")
TOKEN=$(jq -r .token  "$SEED_JSON")
PREFIX=$(jq -r .prefix "$SEED_JSON")

# Собираем образ. ОБЯЗАТЕЛЬНО прокидываем build-args и ставим LABEL:
docker build   --build-arg LAB_LOGIN="$LOGIN"   --build-arg LAB_TOKEN="$TOKEN"   -t "$LOGIN/image-lab:$PREFIX" .
```

### 2) Локальная проверка (быстро)

Запуск на read-only

```bash
docker run -d --rm --read-only \
  -e ROCKET_SIZE=Big \
  -p 8080:8000 "$LOGIN/image-lab:$PREFIX"

```
Проверка ответа (или открыть в браузере):

```bash
curl -s http://localhost:8080/health
curl -s http://localhost:8080/

```
Остановка контейнера возможна через Docker Desktop (кнопка Stop) или командой:

### 3) Публикация в Docker Hub

```bash
docker login
docker push "$LOGIN/image-lab:$PREFIX"
```

---

## Что сдавать

Что отправляется на проверку:

1. **Ссылку на образ** в Docker Hub:  
   `https://hub.docker.com/r/<login>/image-lab/tags` (или конкретный тег `<prefix>`).
2. Ваш `seed-<login>.json`.
3. Ссылку на ваш репозиторий с `Dockerfile` и коротким отчетом:
   - вывод `docker history <login>/image-lab:<prefix>`;
   - 5–8 предложений, чем ваш образ лучше «наивной» сборки (размер, безопасность, кэш и т.д.).

