# hw3 docker

## запуск

```bash
cp .env.example .env
docker compose up -d --build
```

переменные в `.env`: `APP_PORT` (по умолчанию 8080), `GREETING`.

## 4 балла

```text
$ docker build -t counter-app .
$ docker run -d --name counter-single -p 8080:8000 -e GREETING='Hello, Docker!' counter-app
$ curl -s [http://127.0.0.1:8080/](http://127.0.0.1:8080/)
{"greeting": "Hello, Docker!", "count": 0, "hostname": "4b30416dbefb", "storage": "sqlite"}
$ curl -s -X POST [http://127.0.0.1:8080/increment](http://127.0.0.1:8080/increment)
{"greeting": "Hello, Docker!", "count": 1, "hostname": "4b30416dbefb", "storage": "sqlite"}
$ curl -s -X POST [http://127.0.0.1:8080/increment](http://127.0.0.1:8080/increment)
{"greeting": "Hello, Docker!", "count": 2, "hostname": "4b30416dbefb", "storage": "sqlite"}
$ docker logs counter-single
2026-11-16 12:15:25,343 listening port=8000 storage=sqlite
2026-11-16 12:15:27,386 http 192.168.65.1 "GET / HTTP/1.1" 200 -
2026-11-16 12:15:27,405 http 192.168.65.1 "POST /increment HTTP/1.1" 200 -
2026-11-16 12:15:27,420 http 192.168.65.1 "POST /increment HTTP/1.1" 200 -
```

## 6 баллов

```text
$ docker compose up -d
$ curl -s -X POST [http://127.0.0.1:8080/increment](http://127.0.0.1:8080/increment)
{"greeting": "Hello from Compose!", "count": 1, "hostname": "5b64fd151f0d", "storage": "sqlite"}
$ curl -s -X POST [http://127.0.0.1:8080/increment](http://127.0.0.1:8080/increment)
{"greeting": "Hello from Compose!", "count": 2, "hostname": "5b64fd151f0d", "storage": "sqlite"}
$ docker compose down
[+] down 2/2
 ✔ Container dockerhw3-app-1 Removed                                                                                3.1s
 ✔ Network dockerhw3_default Removed                                                                                0.1s
$ docker compose up -d
[+] up 2/2
 ✔ Network dockerhw3_default Created                                                                                0.0s
 ✔ Container dockerhw3-app-1 Started                                                                                0.1s
$ curl -s [http://127.0.0.1:8080/](http://127.0.0.1:8080/)
{"greeting": "Hello from Compose!", "count": 2, "hostname": "75265089dde7", "storage": "sqlite"}
```

## 8 баллов

статус:
```text
$ docker compose ps
NAME                IMAGE              COMMAND                  SERVICE   CREATED          STATUS                    PORTS
dockerhw3-app-1     dockerhw3-app      "python app.py"          app       14 seconds ago   Up 8 seconds (healthy)    0.0.0.0:8080->8000/tcp, [::]:8080->8000/tcp
dockerhw3-redis-1   redis:7.4-alpine   "docker-entrypoint.s…"   redis     14 seconds ago   Up 13 seconds (healthy)   6379/tcp
```

перезапуск без `-v` (счетчик 3 сохранился):
```text
$ curl -s -X POST [http://127.0.0.1:8080/increment](http://127.0.0.1:8080/increment)
{"greeting": "Hello from Compose!", "count": 1, "hostname": "cf6bfbf246dd", "storage": "redis"}
$ curl -s -X POST [http://127.0.0.1:8080/increment](http://127.0.0.1:8080/increment)
{"greeting": "Hello from Compose!", "count": 2, "hostname": "cf6bfbf246dd", "storage": "redis"}
$ curl -s -X POST [http://127.0.0.1:8080/increment](http://127.0.0.1:8080/increment)
{"greeting": "Hello from Compose!", "count": 3, "hostname": "cf6bfbf246dd", "storage": "redis"}
$ docker compose down
[+] down 3/3
 ✔ Container dockerhw3-app-1   Removed                                                                              3.1s
 ✔ Container dockerhw3-redis-1 Removed                                                                              0.2s
 ✔ Network dockerhw3_default   Removed                                                                              0.1s
$ docker compose up -d
[+] up 3/3
 ✔ Network dockerhw3_default   Created                                                                              0.0s
 ✔ Container dockerhw3-redis-1 Healthy                                                                              5.6s
 ✔ Container dockerhw3-app-1   Started                                                                              5.6s
$ curl -s [http://127.0.0.1:8080/](http://127.0.0.1:8080/)
{"greeting": "Hello from Compose!", "count": 3, "hostname": "405801fff2a9", "storage": "redis"}
```

остановка redis:
```text
$ docker compose stop redis
[+] stop 1/1
 ✔ Container dockerhw3-redis-1 Stopped                                                                              0.1s
$ curl -i -s [http://127.0.0.1:8080/healthz](http://127.0.0.1:8080/healthz)
HTTP/1.0 200 OK
Server: BaseHTTP/0.6 Python/3.13.18
Date: Mon, 16 Nov 2026 12:55:14 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 19
Cache-Control: no-store

{"status": "alive"}
$ curl -i -s [http://127.0.0.1:8080/readyz](http://127.0.0.1:8080/readyz)
HTTP/1.0 503 Service Unavailable
Server: BaseHTTP/0.6 Python/3.13.18
Date: Mon, 16 Nov 2026 12:55:14 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 32
Cache-Control: no-store

{"error": "storage_unavailable"}
$ curl -i -s -X POST [http://127.0.0.1:8080/increment](http://127.0.0.1:8080/increment)
HTTP/1.0 503 Service Unavailable
Server: BaseHTTP/0.6 Python/3.13.18
Date: Mon, 16 Nov 2026 12:55:14 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 32
Cache-Control: no-store

{"error": "storage_unavailable"}
```

запуск redis обратно:
```text
$ docker compose start redis
[+] start 1/1
 ✔ Container dockerhw3-redis-1 Started                                                                              0.1s
$ curl -s [http://127.0.0.1:8080/](http://127.0.0.1:8080/)
{"greeting": "Hello from Compose!", "count": 3, "hostname": "405801fff2a9", "storage": "redis"}
$ curl -s -X POST [http://127.0.0.1:8080/increment](http://127.0.0.1:8080/increment)
{"greeting": "Hello from Compose!", "count": 4, "hostname": "405801fff2a9", "storage": "redis"}
```
