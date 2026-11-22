(Я не очень осознала что такое отчет о проверках и для заданий на какие баллы он нужен, но я попробую)

На 4 балла:

{"greeting": "hi", "count": 2, "hostname": "11eed8aaea29", "storage": "sqlite"}

Логи:

docker run -d --name co -p 8080:8000 -e GREETING="hi" counter-app

curl.exe -fsS -X POST http://127.0.0.1:8080/increment #(вот таким счетчик увеличиваем)

curl.exe -fsS http://127.0.0.1:8080/ #(вот таким чекаем)



docker logs co

2026-11-22 15:44:59,799 listening port=8000 storage=sqlite

2026-11-22 15:46:57,693 http 172.17.0.1 "GET / HTTP/1.1" 200 -

2026-11-22 15:46:57,782 http 172.17.0.1 "GET /favicon.ico HTTP/1.1" 404 -

2026-11-22 15:48:05,369 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -

2026-11-22 15:48:42,270 http 172.17.0.1 "GET / HTTP/1.1" 200 -

2026-11-22 15:48:47,682 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -

2026-11-22 15:48:54,614 http 172.17.0.1 "GET / HTTP/1.1" 200 -

2026-11-22 15:49:05,828 http 172.17.0.1 "GET / HTTP/1.1" 200 -



На 6 баллов:



curl.exe -fsS -X POST http://127.0.0.1:8080/increment

{"greeting": "hi?", "count": 1, "hostname": "6c551fd635bc", "storage": "sqlite"}

curl.exe -fsS -X POST http://127.0.0.1:8080/increment

{"greeting": "hi?", "count": 2, "hostname": "6c551fd635bc", "storage": "sqlite"}

curl.exe -fsS -X POST http://127.0.0.1:8080/increment

{"greeting": "hi?", "count": 3, "hostname": "6c551fd635bc", "storage": "sqlite"}

docker compose down

\[+] down 2/2

&#x20;✔ Container hw\_3\_template-main-app-1 Removed                                                                                    3.4s ✔ Network hw\_3\_template-main\_default Removed                                                                                    0.2sPS C:\\Users\\er\_el\\Desktop\\hw\_3

docker compose up -d

\[+] up 2/2

&#x20;✔ Network hw\_3\_template-main\_default Created                                                                                    0.1s ✔ Container hw\_3\_template-main-app-1 Started                                                                                    0.6sPS C:\\Users\\er\_el\\Desktop\\hw\_3\_template-curl.exe -fsS http://127.0.0.1:8080/

{"greeting": "hi?", "count": 3, "hostname": "fc207793471b", "storage": "sqlite"}



(команды-выводы, тут увеличили счетчик до 3 и проверка работы volume)



В compose.yaml:

services:

&#x20; app:

&#x20;   build: .

&#x20;   ports:

&#x20;     - "${APP\_PORT}:8000"

&#x20;   environment:

&#x20;     - GREETING=${GREETING}

&#x20;     - DATA\_FILE=/data/counter.db

&#x20;   volumes:

&#x20;     - counter-data:/data



volumes:

&#x20; counter-data:

(порт из созданного .env, передача переменных окружения, volumes)





На 8 баллов:



В compose.yaml:

services:

&#x20; app:

&#x20;   build: .

&#x20;   ports:

&#x20;     - "${APP\_PORT}:8000"

&#x20;   environment:

&#x20;     - GREETING=${GREETING}

&#x20;     - STORAGE=redis

&#x20;     - REDIS\_URL=redis://redis:6379/0

&#x20;   healthcheck:

&#x20;     test: \["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:8000/healthz')"]

&#x20;     interval: 10s

&#x20;     timeout: 10s

&#x20;     retries: 5



&#x20; redis:

&#x20;   image: redis:7.4-alpine

&#x20;   command: redis-server --appendonly yes

&#x20;   volumes:

&#x20;     - redis-data:/data

&#x20;   healthcheck:

&#x20;     test: \["CMD", "redis-cli", "ping"]

&#x20;     interval: 10s

&#x20;     timeout: 10s

&#x20;     retries: 5



volumes:

&#x20; redis-data:





1\)Выполните несколько POST /increment, запишите значение N. После docker compose down и повторного up -d запрос GET / должен вернуть тот же N. Не используйте down -v в этом эксперименте: он удаляет данные.



curl.exe -fsS -X POST http://127.0.0.1:8080/increment

curl.exe -fsS -X POST http://127.0.0.1:8080/increment

(после этого N = 2)

docker compose down

docker compose up -d

curl.exe -fsS http://127.0.0.1:8080/

{"greeting": "hi?", "count": 2, "hostname": "743933970f04", "storage": "redis"} #(получается N все еще 2, ура)



2\)Выполните docker compose stop redis. Приложение продолжает работать: GET /healthz возвращает 200, а GET /readyz и POST /increment — 503.



docker compose stop redis

\[+] stop 1/1

&#x20;✔ Container hw\_3\_template-main-redis-1 Stopped



curl.exe -s -o NUL -w "%{http\_code}" http://127.0.0.1:8080/healthz

200

curl.exe -s -o NUL -w "%{http\_code}" http://127.0.0.1:8080/readyz

503

curl.exe -s -o NUL -w "%{http\_code}" -X POST http://127.0.0.1:8080/increment

503



3)Выполните docker compose start redis, дождитесь готовности. Без ручного перезапуска приложения GET / снова возвращает N, следующий POST — N + 1.



docker compose start redis

\[+] start 1/1

&#x20;✔ Container hw\_3\_template-main-redis-1 Started

curl.exe -fsS http://127.0.0.1:8080/

{"greeting": "hi?", "count": 2, "hostname": "743933970f04", "storage": "redis"} #(ура, N все еще 2)

curl.exe -fsS -X POST http://127.0.0.1:8080/increment

{"greeting": "hi?", "count": 3, "hostname": "743933970f04", "storage": "redis"} #(стало 3 что как раз N + 1)







