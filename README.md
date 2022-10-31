### Первые шаги в Ansible ###

В Vagrantfile описан подъем двух виртульных машин, на одной из которых поднят ansible, а на другой nginx.По окончанию работы Vargant проверяем доступность nginx на 8080 порту:

```
$ curl -I http://192.168.56.150:8080
HTTP/1.1 200 OK
Server: nginx/1.20.1
Date: Mon, 31 Oct 2022 07:25:26 GMT
Content-Type: text/html
Content-Length: 4833
Last-Modified: Fri, 16 May 2014 15:12:48 GMT
Connection: keep-alive
ETag: "53762af0-12e1"
Accept-Ranges: bytes
```
