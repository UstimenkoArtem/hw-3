Нужно запустить Docker Desktop, команды выполняются в PowerShell из папки проекта

Файл .env создаётся из примера командой

Copy-Item .env.example .env

В .env задаётся внешний порт приложения и текст приветствия

APP\_PORT=8081

GREETING=Compose\_Привет\_мы\_твои\_враги!

В compose.yaml настраиваются
сборка приложения по Dockerfile
передача приветствия через GREETING
публикация порта приложения 8000 на выбранном APP\_PORT
хранилище SQLite: STORAGE=sqlite, DATA\_FILE=/data/counter.db
именованный том counter\_data, подключённый к /data для сохранения базы

запуск
docker compose up -d --build

порт в .env должен быть свободен

Для проверки счётчик увеличивается командой
curl.exe -fsS -X POST http://localhost:8081/increment

проверка значения
curl.exe -fsS http://localhost:8081/

Проверка сохранения данных после пересоздания
docker compose down

docker compose up -d

curl.exe -fsS http://localhost:8081/

При проверке до пересоздания счётчик был равен 3, после пересоздания остался равен 3, идентификатор контейнера изменился



Redis
Конфигурация SQLite сохранена в compose.sqlite.yaml
В прошлых командах теперь используется docker compose -f compose.sqlite.yaml вместо docker compose
В compose.yaml настроены app и redis
Приложение получает STORAGE=redis, REDIS\_URL=redis://redis:6379/0
Redis использует образ redis:7.4-alpine, сохраняет данные через AOF и именованный том redis\_data, подключённый к /data.

Healthcheck Redis выполняет redis-cli ping с интервалом, тайм-атуом и количеством попыток указанных в compose, в данном случае интервал 2с, тайм-аут 1 секунда, попыток 42
Приложение ждёт готовность Redis через depends\_on

Остановка варианта с SQLite перед запуском Redis

docker compose -f compose.sqlite.yaml down

запуск
docker compose up -d --build --wait

Команда увеличения 5 раз
curl.exe -fsS -X POST http://localhost:8081/increment

Проверка и пересоздания контейнеров без удаления тома
curl.exe -fsS http://localhost:8081/

docker compose down

docker compose up -d --wait

curl.exe -fsS http://localhost:8081/

до и после пересоздания count = 5, hostname изменился

Остановка redis и проверка
docker compose stop redis

curl.exe -sS -i http://localhost:8081/healthz

curl.exe -sS -i http://localhost:8081/readyz

curl.exe -sS -i http://localhost:8081/

curl.exe -sS -i -X POST http://localhost:8081/increment

healthz вернул HTTP 200 
readyz GET и POST increment вернули HTTP 503

Восстановление redis и проверка
docker compose start redis

docker compose exec redis redis-cli ping
curl.exe -sS -i http://localhost:8081/readyz

curl.exe -fsS http://localhost:8081/

curl.exe -fsS -X POST http://localhost:8081/increment

readyz вернул HTTP 200 GET показал count = 5, POST - count = 6, приложение не перезапускалось



Результаты проверок в logs/08-checks.txt, логи контейнеров в logs/08-compose.log


